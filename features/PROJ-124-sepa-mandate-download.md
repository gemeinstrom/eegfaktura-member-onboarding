# PROJ-124 — SEPA-Lastschriftmandat als Admin-Download

**Status:** In Progress
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
