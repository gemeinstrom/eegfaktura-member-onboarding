# PROJ-124 — SEPA-Lastschriftmandat als Admin-Download

**Status:** Approved
**Created:** 2026-07-12
**Owner-Auslöser:** Kundenfrage 2026-07-12 („Die Beitrittsbestätigung kann ich nachträglich herunterladen, aber das SEPA-Formular finde ich nicht.")

## Problem

Das SEPA-Lastschriftmandat-PDF wird heute nur **per E-Mail** an das Mitglied
verschickt (bei Einreichung bzw. beim Import, je nach `SEPAMandateAtImport`).
Es gibt — anders als bei der Beitrittsbestätigung (`approval-pdf`) und der
Beitrittserklärung (`joining-declaration.pdf`) — **keinen On-Demand-Download**
im Admin-UI. Der Admin kann das Mandat also nicht nachträglich abrufen.

## Scope

Symmetrisch zum bestehenden Beitrittsbestätigungs-Download: ein neuer read-only
Download-Endpoint + Eintrag in der „Herunterladen ▾"-Gruppe der Antrags-Detail-
Ansicht, der das SEPA-Mandat-PDF für den Antrag on-demand erzeugt.

- Reproduziert exakt das Mandat, das (per Mail) erzeugt würde — gleiche
  Gating-Logik (`buildSEPAMandateData`), gleicher Renderer (`Generate` für CORE,
  `GenerateCompany` für B2B), gleiche Mandatsreferenz-/Debtor-Anreicherung wie
  der Import-Mail-Pfad.
- **Read-only**: der Download schreibt NICHTS (kein `SetMandateDateIfEmpty`,
  kein `SetMandateReferenceIfEmpty`) — er liest nur die bereits persistierten
  Werte. Persistierung ist ausschließlich Sache des Submit-/Import-Pfads.

## Acceptance Criteria

- **AC-1** Neuer Endpoint `GET /api/admin/applications/{id}/sepa-mandate.pdf`
  liefert das SEPA-Mandat-PDF (`application/pdf`, `Content-Disposition:
  attachment`).
- **AC-2** Tenant-Isolation: `checkTenantAccess(id)` als erste Prüfung; ein
  Tenant-Admin kann nur Anträge seiner RC(s) abrufen. PROJ-71-Cool-Down-Gate
  (`enforceCustomerContractByID`) wie beim Beitrittsbestätigungs-Download.
- **AC-3** Für `einzugsart=b2b` wird `GenerateCompany` (Firmenlastschrift), sonst
  `Generate` (Basislastschrift) gerendert. Dateiname entsprechend
  `sepa-firmenlastschriftmandat-<Ref>.pdf` bzw. `sepa-lastschriftmandat-<Ref>.pdf`.
- **AC-4** Kein Mandat verfügbar (`einzugsart=kein_sepa`/leer ODER EEG-Stammdaten
  unvollständig, d. h. `buildSEPAMandateData` liefert nil) → `409` mit klarer
  Meldung; der Admin sieht sie als Toast.
- **AC-5** Frontend zeigt den Eintrag „SEPA-Mandat" in der „Herunterladen"-Gruppe
  nur bei `einzugsart ∈ {core, b2b}` (kein_sepa → kein Eintrag).
- **AC-6** Mandatsreferenz + B2B-Debtor-Name werden identisch zum Import-Mail-Pfad
  gesetzt (gemeinsamer Helper, keine divergierende Zweit-Implementierung).

## Edge Cases

- **EC-1** Antrag vor Import (CORE, noch keine Mitgliedsnummer): Mandat hat keine
  Referenz → PDF wird trotzdem erzeugt (wie das Submit-Zeit-Mandat, das ebenfalls
  ohne Referenz verschickt wurde). Korrekt.
- **EC-2** EEG-Stammdaten unvollständig (z. B. Creditor-ID fehlt): 409, kein PDF —
  identisch zu „Mail wird ohne Mandat verschickt" heute, nur hier sichtbar.
- **EC-3** B2B-Vorbereitungs-Toggle (PROJ-91, CORE-Antrag mit zusätzlichem
  B2B-Mandat): der Download liefert das **primäre** Mandat passend zur
  `einzugsart` (CORE → Basislastschrift). Das PROJ-91-Zweit-B2B-Dokument ist
  **out of scope** (Nischenfall; das Mitglied hat es per Mail).

## Non-Goals

- Kein Public-/Mitglieder-seitiger Download (bleibt Admin-only, Keycloak-geschützt).
- Kein separater Download des PROJ-91-B2B-Vorbereitungs-Zweitmandats.
- Keine Änderung am Mail-Versand-Pfad (nur read-only Reproduktion).
- Keine DB-Migration, kein Schema-Change.

## Security

PII-Download (IBAN, Name, Adresse) → gleiche Klassifikation wie der
Beitrittsbestätigungs-/Excel-Download. Neuer Admin-Endpoint + Tenant-Iso →
`/security-review` Pflicht.

## QA / Verifikation (2026-07-12)

- `go build ./...`, `go vet ./...`, `go test ./...` grün. Neuer Unit-Test
  `TestApplyMandateReferenceAndDebtor` (5 Fälle: Override-vor-Default, MemberNumber-
  Default, beide leer, B2B-Debtor, CORE behält Kontoinhaber). Bestehende
  `buildSEPAMandateData`- + Import-Pfad-Tests unverändert grün → Helper-Extraktion
  verhaltensgleich (AC-6).
- Frontend: `tsc --noEmit` clean, `vitest run` 258/258, `npm run build` clean.
- AC-1..AC-6 + EC-1..EC-3 code-verifiziert.

## Security Review (2026-07-12) — **APPROVED**

**Scope:** `internal/http/admin_attachments.go` (neuer Handler),
`internal/application/admin_service.go` (`GenerateSEPAMandatePDF`),
`internal/application/application_service.go` (`applyMandateReferenceAndDebtor`),
`cmd/server/main.go` (Route), Frontend-Download.

| Prüfung | Ergebnis |
|---------|----------|
| Tenant-Iso | `checkTenantAccess(id)` ist die **erste** Prüfung im Handler (nach `parseID`), identisch zu `DownloadApprovalPDF`. `GenerateSEPAMandatePDF` lädt App→Entrypoint über `app.RCNumber`; kein Cross-RC-Pfad. ✓ |
| Cool-Down-Gate | `enforceCustomerContractByID` wie beim approval-pdf → suspendierte EEG kann nicht laden. ✓ |
| Read-only | Nur `GetByID`/`GetByRCNumber`/`GetLogo` + reine Helfer + PDF-Renderer; **kein** `Set*`/Insert. Ein Öffnen des Downloads schreibt nichts. ✓ |
| Refactor-Integrität | Import-Mail-Pfad ruft jetzt denselben Helfer; bit-identisch (Tests grün). ✓ |
| Input-Validierung | `parseID` = `uuid.Parse` (malformed → 400). Repos parametrisiert, kein neuer Query-Bau. ✓ |
| PII in Logs | Handler loggt nichts; keine IBAN/Name/E-Mail im Log-Pfad. ✓ |
| Error-Model | `409` mit Klartext-Ursache (kein Mandat / EEG-Stammdaten unvollständig) — kein Stack-Trace/DB-Detail; sonst generischer `handleServiceError`. ✓ |
| Content-Disposition | Dateiname aus system-generierter `ReferenceNumber` (PROJ-35-Muster, kein CRLF/User-Freitext) — wie approval-pdf, keine Header-Injection. ✓ |
| Abgrenzung | Keine neuen Secrets/Env/Helm/Public-Endpoints/Rate-Limits/Status-Transitions/Import-Logik/Schema. ✓ |

**Scans:** gosec `-severity medium -confidence medium` auf `internal/http/` +
`internal/application/` → **0 Issues** (59 Files). govulncheck → 1 stdlib-Baseline
(`GO-2026-5856`, crypto/tls) — **nicht durch PROJ-124 eingeführt** (kein go.mod-Change),
separater Go-Toolchain-Bump. Trivy/npm-audit n/a (keine Helm/Dockerfile/package.json-
Änderung).

**Info (kein Blocker):** Der Excel-PII-Export schreibt ein DSGVO-`pii-export`-
Audit-Log (Metadaten, keine PII-Werte); approval-pdf + dieser SEPA-Download tun
das nicht. Optionaler künftiger Angleich, da IBAN betroffen ist.

**Verdikt: APPROVED** — 0 Critical/High/Medium.
