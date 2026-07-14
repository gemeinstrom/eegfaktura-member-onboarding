# PROJ-125 — reset-activation landet in „Bereit zur Aktivierung" (Sackgassen-Fix)

**Status:** Approved
**Created:** 2026-07-14
**Owner-Auslöser:** Tester-Rückmeldung Manuel Wurzinger 2026-07-14 — „Mitglieder im Status ‚Importiert' können nicht Aktiv gekennzeichnet werden und werden auch nicht im Core geprüft."

## Problem (Sackgasse)

`POST /reset-activation` (PROJ-100) setzte einen Antrag von `activated` zurück
auf **`imported`**. Das ist eine Sackgasse:

- Der Aktivierungs-Check-Batch scannt **nur** Anträge in `ready_for_activation`
  ([admin_applications_import.go](../internal/http/admin_applications_import.go)) → ein `imported`-Antrag wird im Core **nie geprüft**.
- Die Admin-Transition-Map hat **keinen** Eintrag für `imported` → kein
  Vorwärts-Übergang über das generische `/status`.
- `imported → ready_for_activation` existiert nur als **Auto-Schaltung des
  Import-Service** (läuft einmalig beim Import, nie erneut).

→ Nach `reset-activation` blieb der Antrag in `imported` hängen: weder Batch
noch manuelle Aktivierung kamen weiter. Exakt der Tester-Befund.

Der PROJ-76-interne Rollback (`RollbackActivation`, beim Batch-Board-Mail-Fehler)
machte es bereits richtig — er setzt auf `ready_for_activation` zurück. Nur das
Admin-`reset-activation` (PROJ-100) wich davon ab.

## Lösung (Owner-Entscheidung 2026-07-14: Variante A)

`ResetActivationTx` / `ResetActivation` setzen jetzt auf **`ready_for_activation`**
statt `imported`. Das ist der normale Post-Import-Ruhestatus; von dort greifen
sowohl der Batch (`ready_for_activation → activated`) als auch die manuelle
Aktivierung. Damit der Voll-Cleanup-Recovery-Pfad erhalten bleibt, akzeptiert
`reset-to-review` jetzt **`imported` ODER `ready_for_activation`** als Quelle.

Kein Schema-Change, keine Feld-Änderung — nur der Ziel-/Quell-Status. **Eine
Daten-Migration** (000092) rettet zusätzlich die bereits gestrandeten
Bestands-Anträge (Owner-Entscheidung 2026-07-14): alle Anträge, die per altem
Verhalten in `imported` hängen (mit abgeschlossenem Import), werden einmalig
nach `ready_for_activation` gehoben.

## Acceptance Criteria

- **AC-1** `POST /reset-activation` transitioniert `activated → ready_for_activation`
  (vorher `→ imported`). Geräumte Felder unverändert (`activated_at`,
  `activation_notification_sent_at`, `board_declaration_sent_at`,
  `edition_at_activation`); `member_number`/`target_participant_id`/`imported_at`
  bleiben.
- **AC-2** Nach dem Reset ist der Antrag **manuell aktivierbar**
  (`ready_for_activation → activated`, bestehender Button „Als aktiv markieren").
- **AC-3** Nach dem Reset wird der Antrag vom **Activation-Check-Batch** wieder
  im Core geprüft (Batch scannt `ready_for_activation`).
- **AC-4** `POST /reset-to-review` akzeptiert `imported` **oder**
  `ready_for_activation` als Quelle; 13-Feld-Cleanup unverändert.
- **AC-5** Frontend: `reset-activation`-Toast/Warnung nennen „Bereit zur
  Aktivierung"; „Auf Prüfung zurücksetzen" ist auch im
  `ready_for_activation`-Block verfügbar.
- **AC-6** Drift-Guard-Tests unverändert grün (`activated`/`imported` bleiben
  ohne Eintrag in der `adminTransitions`-Map — die Rollbacks laufen weiter über
  dedizierte Endpoints).

## Edge Cases

- **EC-1** Generisches `ready_for_activation → under_review` (PROJ-46, ohne
  Cleanup) bleibt neben `reset-to-review` (mit Cleanup) bestehen — zwei bewusst
  unterschiedliche Pfade (leichter Re-Review vs. Voll-Redo), visuell durch die
  destruktive Button-Variante getrennt.
- **EC-2** `reset-import` akzeptierte `ready_for_activation` schon vorher als
  Quelle → der „ganz zurück"-Pfad (`ready_for_activation → approved → rejected`)
  bleibt intakt.
- **EC-3** Bestand-Anträge, die aktuell in `imported` hängen (durch das alte
  Verhalten): werden durch **Migration 000092** einmalig nach
  `ready_for_activation` gehoben (Owner-Entscheidung 2026-07-14). Der
  `imported`-Block behält daneben „Auf Prüfung zurücksetzen" + „Import
  zurücksetzen" für den seltenen Fall einer fehlgeschlagenen Auto-Transition.
  (Ein permanenter `imported → ready_for_activation`-Button wurde bewusst NICHT
  ergänzt — Variante B verworfen; die Migration deckt die Bestandsfälle ab, und
  künftig landet reset-activation gar nicht mehr in `imported`.)

## Non-Goals

- Kein neuer „Zur Aktivierung freigeben"-Button aus `imported` (Variante B,
  verworfen — die Migration 000092 deckt die Bestandsfälle ab).
- Kein Schema-Change (Migration 000092 ist reine Daten-Korrektur, keine DDL).
- Batch-Scan-Umfang unverändert (weiterhin nur `ready_for_activation`).

## Security

Status-Transition-Änderung → `/security-review` Pflicht. Kein neuer Endpoint,
keine Auth-/Tenant-/PII-/Schema-Änderung; Tenant-Checks der Recovery-Endpoints
unverändert.

## QA / Verifikation (2026-07-14)

- `go build`, `go vet`, `go test ./...` grün. Bestehende Drift-Guard-Tests
  (`TestAdminTransitions_PROJ100_RollbackPathsNotInMap`,
  `TestAdminTransitions_AllReferencedStatusesAreKnown`) unverändert grün — die
  Rollbacks laufen weiter über dedizierte Endpoints, nicht über `/status` (AC-6).
- Frontend: `tsc --noEmit` clean, `vitest run` 258/258, `npm run build` clean.
- Verhalten aus dem Code verifiziert: Batch scannt `ready_for_activation`
  (import.go), manuelle Transition `ready_for_activation → activated` existiert
  in der Map → AC-2/AC-3 erfüllt. Runtime-E2E-Verifikation auf test-Env nach
  helm upgrade (konsistent mit PROJ-100/PROJ-123-Statusfluss-Features).

## Security Review (2026-07-14) — **APPROVED**

**Scope:** `ResetActivationTx` (Ziel-Status), `ResetToReviewTx` + `ResetToReview`
(Quell-Status erweitert), Frontend-Button/-Label, Doku.

| Prüfung | Ergebnis |
|---------|----------|
| Kein neues Capability | reset-activation war schon vorher möglich (Status=activated + Tenant-Check + Pflicht-Grund ≥10). Nur das **Ziel** änderte sich (imported → ready_for_activation, beide valide Post-Import-States). ✓ |
| reset-to-review Quell-Erweiterung | `ready_for_activation` als zusätzliche Quelle: gleicher Tenant-Check, Pflicht-Grund, In-Flight-Guard, 13-Feld-Cleanup. `ready_for_activation` ist ein legitimer Post-Import-State — kein Eskalations-/Cross-Tenant-Pfad. ✓ |
| SQL | Parametrisiert, Status als Go-Konstanten; kein String-Concat. ✓ |
| Transition-Map | `adminTransitions` unverändert; `activated`/`imported` bleiben ohne Eintrag → Drift-Guards grün, generisches `/status` flippt nichts. ✓ |
| Migration 000092 | Reine Daten-Korrektur (`UPDATE ... SET status`), kein DDL. Guard `import_finished_at IS NOT NULL` schließt In-Flight-Importe aus; idempotent; hebt nur `imported`→`ready_for_activation`. Kein PII, kein Cross-Tenant (globaler Status-Fix). ✓ |
| Abgrenzung | Kein neuer Endpoint/Env/Secret/Helm/Schema/PII/Auth. ✓ |

**Scans:** gosec `-severity medium -confidence medium` (internal/application + internal/http) → **0 Issues** (59 Files). govulncheck → nur stdlib-Baseline (crypto/tls, kein go.mod-Change). Trivy/npm-audit n/a.

**Verdikt: APPROVED** — 0 Critical/High/Medium.
