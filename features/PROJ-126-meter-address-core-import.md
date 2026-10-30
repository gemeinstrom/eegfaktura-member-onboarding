# PROJ-126 — Abweichende Zählpunkt-Adresse beim Core-Import übernehmen

**Status:** Deployed
**Created:** 2026-10-13
**Auslöser:** Nutzer-Rückmeldung 2026-10-13 — „die Adresse des Zählpunkts wird nicht übernommen, nur die Adresse des Mitglieds."

## Problem

PROJ-39 hat die abweichende Adresse je Zählpunkt eingeführt
(`metering_point.address_street/_street_number/_zip/_city`, all-or-nothing).
Sie wird erfasst, gespeichert, im Admin angezeigt und exportiert, beim
Core-Import aber ignoriert: `importing.BuildPayload` setzte
`meters[].street/streetNumber/zip/city` immer aus der Mitgliedsadresse
(`application.resident_*`). Die Übernahme war in PROJ-39 explizit als
„Out of Scope — nachfolgende Spec" vermerkt; diese Spec wurde nie angelegt.

## Lösung

In `BuildPayload` übernimmt jeder Zählpunkt mit gesetzter eigener Adresse
diese in `meters[].street/streetNumber/zip/city`. Zählpunkte ohne eigene
Adresse erben wie bisher die Mitgliedsadresse. `residentAddress` und
`billingAddress` des Teilnehmers bleiben unverändert die Mitgliedsadresse.

Da die vier Adress-Spalten serverseitig all-or-nothing sind, entscheidet
`address_street` allein.

## Acceptance Criteria

- [x] AC-1: Zählpunkt mit abweichender Adresse → Core-Payload trägt diese Adresse am Zählpunkt.
- [x] AC-2: Zählpunkt ohne abweichende Adresse → Core-Payload trägt die Mitgliedsadresse (Bestandsverhalten).
- [x] AC-3: `residentAddress` / `billingAddress` des Teilnehmers bleiben die Mitgliedsadresse.
- [x] AC-4: `docs/import-mapping.md` beschreibt das neue Mapping.

## Out of Scope

- **Nachbessern bereits importierter Anträge** (Owner-Entscheidung 2026-10-13): bereits in den Core importierte Zählpunkte behalten die dort gesetzte (Mitglieds-)Adresse; keine Daten-Migration, kein Re-Sync.
- Schema-, Endpoint- oder Frontend-Änderungen (keine nötig — der Core kennt die Felder am Zählpunkt bereits).

## Implementation Notes

- `internal/importing/payload.go` — Override nach dem Aufbau des `CoreMeteringPoint`.
- Test `TestBuildPayload_MeterDeviatingAddress` (beide Fälle + Teilnehmer-Adressen unverändert); `TestBuildPayload_MetersUseResidentAddress` bleibt für den Fallback gültig.
- Kein Schema-Change, kein neues Feld, keine neue Abhängigkeit.

## QA / Security

- `go build ./...` + `go test ./...` grün; gosec auf `internal/importing` 0 Findings.
- Security-Bewertung (Import-Logik): keine neue Eingabe-Oberfläche — die Adressfelder sind bereits beim Submit/Admin-Edit serverseitig validiert (Längen, all-or-nothing). Werte gehen nur JSON-serialisiert an den Core (kein SQL/Template-Kontext). Tenant-Isolation, Status-Transitions und Auth unberührt. Keine PII in Logs hinzugefügt.

## Deployment

- Live in Prod seit 2026-10-25 (helm Revision 28, Image sha-efeb910).
