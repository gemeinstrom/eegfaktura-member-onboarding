# PROJ-123 — Aktivierungs-Kriterium Variante B: „Zählpunkt aktiv" statt „Registrierung gestartet"

**Status:** In Review (Code + Tests grün, Security-Review APPROVED — wartet auf `helm upgrade`)

## Verifikation

- `go build ./...` + `go test ./internal/importing/ ./internal/shared/` grün;
  `TestShouldActivate` deckt Mode-B PENDING/APPROVED → skip, ACTIVE → activate ab.
- `npx tsc --noEmit` clean.
- **Security-Review APPROVED (2026-07-12):** reine Verschärfung eines
  bestehenden Batch-Kriteriums (weniger/spätere Auto-Aktivierungen), kein neuer
  Endpoint, keine Auth-/Tenant-/PII-/Schema-Änderung. gosec 0 Issues auf
  `internal/importing`, keine `go.mod`-Änderung → keine neuen CVEs.

**Auslöser:** Tester-Rückmeldung 2026-07-12 — die Auto-Aktivierung feuerte zu
früh; „Aktiviert" (+ Beitrittsbestätigung) soll erst kommen, wenn das Mitglied
wirklich aktiv ist, nicht schon wenn die Netzbetreiber-Registrierung nur
*gestartet* wurde.

## Problem

Das per-EEG-Aktivierungs-Kriterium (PROJ-53, `registration_entrypoint.activation_mode`)
hat zwei Modi für den Übergang `ready_for_activation → activated` (der die
Beitrittsbestätigungs-Mail auslöst):

- **Variante A** (`participant_active`): Core-Teilnehmer-Status `ACTIVE`.
  Dieser wird gesetzt, sobald der EEG-Verwalter den Teilnehmer in eegFaktura
  **bestätigt** — was die Netzbetreiber-/EDA-Registrierung erst *anstößt*.
- **Variante B** (`any_meter_registration_started`): mindestens ein Zählpunkt
  mit `processState ∈ {PENDING, APPROVED, ACTIVE}` — also „Registrierung
  gestartet / Netzbetreiber hat mind. geantwortet".

Beide feuern **vor** Abschluss der Netz-Registrierung. In der Praxis führt das
dazu, dass frisch importierte Mitglieder als „Aktiviert" markiert werden und
die Beitrittsbestätigung erhalten, obwohl die Zählpunkte beim Netzbetreiber
noch nicht aktiv sind.

## Ziel

**Variante B wird umgedeutet:** Sie feuert künftig erst, wenn **mindestens ein
Zählpunkt `processState = ACTIVE`** ist (Netz-Registrierung für diesen
Zählpunkt abgeschlossen). Die Zwischenstadien `PENDING`/`APPROVED` lösen die
Aktivierung **nicht** mehr aus.

**Owner-Entscheidung 2026-07-12:** bestehende Variante B **umdeuten** (nicht
additiv als neue Variante C), weil „Registrierung gestartet" als
Auto-Aktivierungs-Kriterium ohnehin fragwürdig ist (Bestätigung vor Abschluss).

## Umsetzung (bewusst chirurgisch)

- **DB-Wert bleibt** `any_meter_registration_started` → **keine Migration**,
  `activation_mode`-CHECK-Constraint, Configexport und `sanitize`-Whitelist
  unberührt. Der String ist ein Legacy-Identifier; die *Bedeutung* ändert sich.
- **Backend** `internal/importing/import_service.go` `shouldActivate`: Mode-B-
  Zweig triggert nur noch bei `processState == "ACTIVE"` (statt
  PENDING/APPROVED/ACTIVE).
- **Frontend** `admin-eeg-settings-editor.tsx`: Variante-B-Label +
  Beschreibung umformuliert auf „mindestens ein Zählpunkt ist aktiv".
- **Doku:** CLAUDE.md (Status-Modell), `docs/domain-model.md`,
  `docs/import-mapping.md` (Activation-Check-Sektion).
- **Tests:** `internal/importing/activation_mode_test.go` — Mode-B PENDING/
  APPROVED → jetzt `skip`, nur ACTIVE → `activate`.

## Akzeptanzkriterien

- **AC-1** Variante B aktiviert einen Antrag nur, wenn min. ein Core-Zählpunkt
  `processState = ACTIVE` hat. ✔ (`shouldActivate`-Test)
- **AC-2** Variante B aktiviert **nicht** bei `processState ∈ {PENDING,
  APPROVED, NEW, INVALID, INACTIVE}` und nicht ohne Zählpunkte. ✔
- **AC-3** Variante A (`participant_active`) bleibt unverändert. ✔
- **AC-4** Bestehende EEGs mit `activation_mode = any_meter_registration_started`
  behalten den Wert (keine Migration); ihr Verhalten wird strenger (feuert erst
  bei ACTIVE). Bewusst, Owner-bestätigt.
- **AC-5** Der Übergang ist idempotent + die Beitrittsbestätigung geht wie
  bisher erst beim tatsächlichen `→ activated` raus.

## Nicht-Ziele

- Kein neues Kriterium (keine Variante C).
- Keine Änderung an Variante A.
- Kein „all meters active" (bewusst „mindestens einer", Owner-Aussage).
- Keine Schema-Migration.

## Security

Status-Übergangs-Logik berührt → `/security-review` Pflicht. Kein neuer
Endpoint, keine Auth-/Tenant-/PII-Änderung; reine Verschärfung eines
bestehenden Batch-Kriteriums.
