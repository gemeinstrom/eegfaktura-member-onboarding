# PROJ-132: Abweichende Zählpunkt-Adresse im Admin-Bearbeiten

## Status: Approved
**Created:** 2026-11-10
**Last Updated:** 2026-11-10

## Dependencies
- Baut auf: PROJ-39 (abweichende Zählpunkt-Adresse im Beitrittsformular), PROJ-95 (Bearbeiten-Formular reicht Zählpunkt-Felder beim Speichern durch), PROJ-126 (Zählpunkt-Adresse geht beim Import nach eegFaktura), PROJ-24 (Admin-Bearbeiten)

## Hintergrund

Tester-Rückmeldung (WhatsApp 2026-11-10): Ein Mitglied hatte zwei Zählpunkte vergessen. Beim Ergänzen im Bearbeiten-Modus ließ sich keine abweichende Anlagenadresse eingeben. Die Adresse wurde deshalb nach dem Import in eegFaktura von Hand überschrieben.

Heute kann das Bearbeiten-Formular die Zählpunkt-Adresse weder anzeigen noch ändern. Eine im Beitrittsformular erfasste Adresse wird beim Speichern nur unverändert durchgereicht (PROJ-95). Das Backend prüft die Adresse beim Admin-Speichern bereits nach der Regel „alle vier Felder oder keines" (`validateMeteringPointAddresses`).

## Owner-Entscheidung (2026-11-10)

- **Alle Zählpunkte:** Die Eingabe gibt es bei jedem Zählpunkt im Bearbeiten-Formular, bei neuen und bei bestehenden. Nicht nur beim Hinzufügen.

## User Stories

- Als **EEG-Admin** möchte ich beim Hinzufügen eines Zählpunkts eine abweichende Anlagenadresse eingeben, damit ich sie nicht nach dem Import in eegFaktura nachtragen muss.
- Als **EEG-Admin** möchte ich die abweichende Adresse eines bestehenden Zählpunkts sehen und korrigieren, wenn das Mitglied sie falsch angegeben hat.
- Als **EEG-Admin** möchte ich eine abweichende Adresse wieder entfernen können, damit der Zählpunkt die Mitgliedsadresse übernimmt.

## Acceptance Criteria

- [ ] AC-1: Jeder Zählpunkt im Bearbeiten-Formular hat eine Option „Abweichende Adresse" (wie im Beitrittsformular). Ist sie an, erscheinen Straße, Hausnummer, PLZ und Ort.
- [ ] AC-2: Bei einem bestehenden Zählpunkt mit gespeicherter Adresse ist die Option beim Öffnen an und die vier Felder sind befüllt. Ohne gespeicherte Adresse ist sie aus.
- [ ] AC-3: Ein neu hinzugefügter Zählpunkt startet ohne abweichende Adresse. Die Option lässt sich sofort einschalten.
- [ ] AC-4: Ist die Option an, sind alle vier Felder Pflicht. Ein Speicherversuch mit unvollständiger Adresse zeigt eine Fehlermeldung mit der Zählpunktnummer und markiert die leeren Felder. Der Server prüft dieselbe Regel (besteht schon) und lehnt halbe Adressen ab.
- [ ] AC-5: Schaltet der Admin die Option aus, wird die Adresse beim Speichern entfernt. Der Zählpunkt übernimmt die Mitgliedsadresse.
- [ ] AC-6: Speichern anderer Änderungen verliert keine Zählpunkt-Adresse (Regression zu PROJ-95).
- [ ] AC-7: Die gespeicherte Adresse erscheint wie heute in Detailansicht, Mails, PDF und Export und geht beim Import nach eegFaktura (PROJ-126). An diesen Ausgaben ändert sich nichts.
- [ ] AC-8: Das Formular funktioniert bei 375, 768 und 1440 px. Die vier Felder brechen auf schmalen Bildschirmen untereinander um.
- [ ] AC-9: Der User-Guide beschreibt die Eingabe im Bearbeiten-Formular, ohne PROJ-Referenzen, mit dem Hinweis „Derzeit nur in der Test-Umgebung" bis zum Prod-Deploy. Ein Changelog-Eintrag ist vorhanden.

## Edge Cases

- **Antrag nach dem Import:** Bearbeiten ist wie heute nur bis zum Import möglich. Eine spätere Korrektur erfolgt in eegFaktura. Keine Änderung am Bearbeiten-Fenster.
- **Adresse gleich der Mitgliedsadresse:** wird wie eingegeben gespeichert, keine automatische Bereinigung (wie im Beitrittsformular).
- **Leerzeichen-Eingaben:** zählen als leer. Ist die Option an und ein Feld leer, gibt es einen Feldfehler.
- **Zählpunkt entfernt:** Die Adresse verschwindet mit dem Zählpunkt (Bestand).

## Out of Scope

- Bearbeiten nach dem Import.
- Adressprüfung gegen ein Verzeichnis.
- Änderungen an Import, Mails, PDF oder Export.

## Technical Requirements

- Nur Frontend: `src/components/admin-edit-form.tsx` (Zählpunkt-Zeilen). Backend-Prüfung und Speicherung bestehen (PROJ-39/95).
- Gleiche Feldnamen, Längen und Prüfregeln wie im Beitrittsformular. Wo möglich gemeinsamen Code nutzen statt die Regel neu zu schreiben.
- Keine Placeholders, Hinweise über das Popover-Muster, shadcn-Bausteine.

---
<!-- Sections below are added by subsequent skills -->

## Tech Design (Solution Architect)

**Stand:** 2026-11-10, vom Owner freigegeben (Richtungs-Bezeichnungen „Bezug/Einspeisung“ im Admin-Formular bleiben). Nur Frontend; keine Migration, keine neue Schnittstelle, keine neuen Pakete.

### A) Bausteine

```
Admin: Antrag bearbeiten
+-- Zählpunkte
    +-- je Zählpunkt (bestehend oder neu)
    |   +-- Zählpunkt · Richtung · Faktor · Entfernen   (unverändert)
    |   +-- Erzeugung / Prognosen …                      (unverändert)
    |   +-- [neu] Option „Abweichende Adresse für diesen Zählpunkt"
    |       +-- Straße · Hausnummer · PLZ · Ort (nur wenn an)
    +-- + Hinzufügen
+-- Fehlermeldung unten (wie heute) + markierte leere Adressfelder
```

### B) Daten (in Worten)

Nichts Neues. Jeder Zählpunkt hat schon heute vier Adressfelder (Straße, Hausnummer, PLZ, Ort; alle leer = Mitgliedsadresse). Das Formular hält sie bereits im Zustand und schickt sie beim Speichern mit (PROJ-95). Die Option „Abweichende Adresse" wird nicht gespeichert: Sie ist an, sobald eines der Felder befüllt ist, wie im Beitrittsformular.

### C) Entscheidungen

| # | Entscheidung | Warum |
|---|---|---|
| T1 | **Eigener kleiner Baustein „Zählpunkt-Adresse"** für das Bearbeiten-Formular (Option + vier Felder), gesteuert über Werte und Änderungs-Rückmeldung. | Das Bearbeiten-Formular arbeitet ohne Formular-Bibliothek, das Beitrittsformular mit. Der Baustein des Beitrittsformulars hängt daran und lässt sich nicht direkt einsetzen. |
| T2 | **Eine gemeinsame Prüfregel** „alle vier oder keines" für Beitrittsformular und Bearbeiten; sie liefert die fehlenden Felder. | Dieselbe Regel steht sonst zweimal im Frontend (bekannte Drift-Quelle). Der Server prüft sie ohnehin (`validateMeteringPointAddresses`). |
| T3 | **Ausschalten leert die vier Felder.** Beim Speichern gehen sie leer an den Server, der den Zählpunkt dann auf die Mitgliedsadresse zurücksetzt. | Erfüllt AC-5. So verhält sich auch das Beitrittsformular. |
| T4 | **Fehler wie im Rest des Bearbeiten-Formulars**: eine Meldung unten mit Zählpunkt-Nummer, dazu die leeren Pflichtfelder rot markiert. | Das Formular meldet heute alle Fehler so. Eine zweite Fehler-Mechanik nur für diese Felder wäre inkonsistent. |
| T5 | **Gleiche Längen** wie im Beitrittsformular (Straße 255, Nr. 50, PLZ 20, Ort wie dort). | Server und Beitrittsformular nutzen diese Grenzen. |
| T6 | **Nichts an den Ausgaben ändern.** Detail, Mails, PDF, Export und Import lesen die Adresse schon heute. | AC-7; nur die Eingabe fehlte. |

### D) Umsetzung und Gates

- **Ein Draft-PR**, nur Frontend: Baustein, Einbau in das Bearbeiten-Formular, gemeinsame Prüfregel, Tests, User-Guide (mit Hinweis „nur in Test"), Changelog.
- **Tests:**
  - Prüfregel: alle Fälle.
  - Bearbeiten: bestehende Adresse ist vorbefüllt.
  - Neuer Zählpunkt mit Adresse kommt im Speicher-Auftrag an.
  - Ausschalten leert die Adresse.
  - Halbe Adresse blockiert das Speichern.
  - Speichern ohne Änderung verliert nichts.
- **Gates:** Review 1. Review 2 ist laut CLAUDE.md nicht nötig: Die Änderung bleibt im Admin-Formular, Import und Ausgaben sind unverändert. Danach `/qa`. Ein `/security-review` entfällt: kein öffentlicher Endpoint, keine Schema-, Auth- oder Import-Änderung.

## Implementation Notes

**Stand:** 2026-11-10, Draft-PR (nur Frontend).

- **T1:** neuer Baustein `src/components/admin-meter-address-fields.tsx` (Checkbox „Abweichende Adresse für diesen Zählpunkt" + Straße/Hausnr./PLZ/Ort, `grid-cols-1 sm:grid-cols-4` wie im Beitrittsformular). Gesteuert über `enabled`/`value` + `onEnabledChange`/`onChange`, keine Placeholders, Längen per `maxLength`.
- **Option-Zustand:** `addressEnabled` liegt als reines UI-Feld in der Zählpunkt-Zeile des Bearbeiten-Formulars (`FormMeteringPoint`). Init: an, sobald ein Adressfeld befüllt ist; neue Zeilen: aus. `toMeteringPointRequest` baut das Request-Objekt explizit und übernimmt das Feld nicht.
- **T2:** gemeinsame Regel `src/lib/meter-address.ts` → `missingMeterAddressFields(address, required?)` liefert die leeren Felder (Leerzeichen = leer). Das zod-`superRefine` des Beitrittsformulars nutzt sie ohne `required` (Verhalten und Meldung unverändert); die Längen im Schema kommen aus `METER_ADDRESS_MAX_LENGTH` (T5, Werte unverändert 255/50/20/255).
- **Entscheidung (AC-4):** Im Bearbeiten-Dialog ruft `validate()` die Regel mit `required = addressEnabled` auf. Damit blockiert auch „Option an, alle vier Felder leer" das Speichern (AC-4: „Ist die Option an, sind alle vier Felder Pflicht"). Das Beitrittsformular bleibt wie bisher (Option an + leer = Mitgliedsadresse), weil sein Verhalten nicht geändert werden sollte.
- **T3:** Ausschalten setzt die vier Felder auf `""` → `toMeteringPointRequest` sendet `undefined` → Server setzt auf Mitgliedsadresse zurück.
- **T4:** `validate()` meldet „Zählpunkt {n}: Bei abweichender Adresse sind alle Adressfelder Pflicht."; nach dem ersten fehlgeschlagenen Speichern werden die leeren Pflichtfelder mit `aria-invalid` + `border-destructive` markiert (live, verschwindet beim Ausfüllen).
- **T6:** keine Änderung an Detail, Mails, PDF, Export, Import, Backend.
- **Tests:** `src/lib/meter-address.test.ts` (alle 16 Kombinationen mit/ohne Pflicht, null, Schema-Meldungen und -Längen), `src/components/admin-edit-meter-address.test.tsx` (vorbefüllt + Option an, Speichern ohne Änderung behält Adresse, neuer Zählpunkt mit Adresse im Payload, Ausschalten leert, halbe Adresse blockiert + Markierung, Option an + leer blockiert).
- **Doku:** User-Guide `04-admin-applications.md` (Abschnitt „Abweichende Adresse eines Zählpunkts") und `changelog.md` mit Hinweis „Derzeit nur in der Test-Umgebung"; `CHANGELOG.md`.
- Kein Playwright-Test ergänzt (kein bestehender Admin-Bearbeiten-E2E-Spec).

### Review 1 (2026-11-10)

- Behoben: Rote Markierung nur nach einem Speicherfehler mit Adressbezug (vorher auch nach anderen Fehlern).
- Behoben: Die Fehlermeldung nennt die Zählpunktnummer statt der Zeilennummer (die Zeilen im Dialog sind nicht nummeriert).
- Behoben: Das Beitrittsformular erkennt „Adresse vorhanden" jetzt ebenfalls über `hasMeterAddress`.
- Behoben: Tests prüfen `aria-checked` und das neue `aria-label` „Zählpunktnummer“ statt CSS-Klassen und Radix-Attribute; neuer Test für einen Speicherfehler ohne Adressbezug. AC-4-Wortlaut an T4 angeglichen.
- Bewusst belassen: Ausschalten leert die Felder sofort (wie im Beitrittsformular, T3). Beschriftungen und Raster sind aus dem Beitrittsformular kopiert, weil dieses `react-hook-form` nutzt; gemeinsamer Baustein als möglicher Follow-up.
- Review 2 und `/security-review` entfallen (nur Admin-Formular, keine Auswirkung auf Import, Schema oder öffentliche Endpoints).

## QA Test Results

**Stand:** 2026-11-10, `/qa` auf PR #129, Head `cc35b51`. Ergebnis: **Approved**, keine Critical- oder High-Bugs. Merge erst, wenn der Konflikt mit `main` aufgelöst ist und CI inkl. E2E grün ist (siehe QA-1).

### Akzeptanzkriterien (9/9 bestanden)

| AC | Ergebnis | Nachweis |
|---|---|---|
| AC-1 | Pass | Jede Zeile rendert `AdminMeterAddressFields` (`admin-edit-form.tsx:1079`); Checkbox + 4 Felder nur bei `enabled` (`admin-meter-address-fields.tsx:42-71`). |
| AC-2 | Pass | Init `addressEnabled: hasMeterAddress(mp)` (`admin-edit-form.tsx:256`), Werte aus `MeteringPointDetail` (`:252-255`). Test „zeigt eine gespeicherte Adresse vorbefüllt mit eingeschalteter Option“. |
| AC-3 | Pass | `addRow` setzt `addressEnabled: false` (`:307`). Test „neuer Zählpunkt startet ohne Adresse und schickt eine eingegebene Adresse mit“. |
| AC-4 | Pass | `validate()` mit `required = addressEnabled` (`:374`), Meldung mit Zählpunktnummer; Markierung nur nach Adressfehler (`:391`, `:1084`). Server: `validateMeteringPointAddresses` im Admin-Pfad (`admin_service.go:584`). Tests „halbe Adresse blockiert …“, „eingeschaltete Option ohne Adresse blockiert …“, „ein anderer Speicherfehler markiert … nicht“. |
| AC-5 | Pass | Ausschalten setzt die 4 Felder auf `""` (`:1091-1097`) → `toMeteringPointRequest` sendet `undefined` (`payload.ts:29-32`) → `trimStringPtr` nil → Repo schreibt NULL. Test „ausschalten entfernt die Adresse beim Speichern“. |
| AC-6 | Pass | Test „speichern ohne Änderung behält die Adresse“; 6-Layer-Kette unten. |
| AC-7 | Pass | Kein Diff in `internal/`, `cmd/`, `db/`, `helm/`, `.github/`; Detail, Mails, PDF, Export und Import unverändert. |
| AC-8 | Pass (Code) | `grid-cols-1 sm:grid-cols-4` wie `DeviatingAddressBlock`; Straße `sm:col-span-2`, Ort `sm:col-span-3`. Kein Browser-Lauf (siehe Einschränkungen). |
| AC-9 | Pass | `04-admin-applications.md` + `changelog.md` mit wörtlichem Block „Derzeit nur in der Test-Umgebung“, 0 PROJ-Refs (grep), Beispiel anonymisiert; `CHANGELOG.md`-Eintrag vorhanden. |

### Admin-Full-Replace: 6 Layer für die 4 Adressfelder

| Layer | Fundstelle | Ergebnis |
|---|---|---|
| TS-Typ | `MeteringPointRequest` `_form-types.ts:285-288`; `MeteringPointDetail` `_admin-types.ts:69-72` | ok |
| Form-State | `FormMeteringPoint` Init `admin-edit-form.tsx:252-256`, `patchRow` über `onChange` | ok; `addressEnabled` ist nur UI-Feld |
| Payload | `meteringPoints.map(toMeteringPointRequest)` `admin-edit-form.tsx:402` → `payload.ts:29-32` (explizites Objekt, `addressEnabled` fällt weg) | ok |
| Go-DTO | `CreateMeteringPointRequest` `internal/shared/requests.go:102-105` (`max=255/50/20/255`, entspricht `METER_ADDRESS_MAX_LENGTH`); Admin-Request `:151` | ok |
| Service | `AdminUpdateApplication` → `BuildMeteringPointFromRequest` (`application_service.go:1953-1957`) → `validateMeteringPointAddresses` (`admin_service.go:584`) | ok |
| Repo-SQL | `metering_point_repo.go:34,51` (INSERT), `:68,109-112` (SELECT/Scan) | ok |

### Edge Cases

- Nach dem Import: unverändert, kein Diff am Bearbeiten-Fenster. Pass.
- Adresse gleich Mitgliedsadresse: keine Bereinigung im Code. Pass.
- Leerzeichen: `isFilled` trimmt (`meter-address.ts:26-28`); Option an + nur Leerzeichen blockiert. Unit-Test über 16 Kombinationen. Pass.
- Zählpunkt entfernt: Full-Replace (DELETE+INSERT), Adresse verschwindet mit der Zeile. Pass.
- Zusätzlich geprüft: Aus → An verwirft die vorher eingegebenen Werte (bewusst, T3). Fehler nach Korrektur einer Adresse, dann E-Mail-Fehler → Markierung wird neu berechnet und verschwindet (`:390-392`). Pass.

### Regression

- Beitrittsformular: zod-`superRefine` nutzt `missingMeterAddressFields(mp)` ohne `required` — gleiche Pfade und Meldung; Längen über `METER_ADDRESS_MAX_LENGTH` mit unveränderten Werten. Tests `meteringPointSchema – abweichende Adresse` (3 Fälle). Pass.
- `DeviatingAddressBlock` nutzt jetzt `hasMeterAddress` (trimmt): Nur-Leerzeichen gilt nicht mehr als „befüllt“. Ohne praktische Wirkung, weil der Server Leerzeichen-Werte als NULL speichert und die Option beim Tippen schon an ist (Info, QA-3).
- Admin-Bearbeiten anderer Felder: alle bestehenden Vitest-Suites grün.
- `main` seit Branch-Punkt (PROJ-58 #128) berührt weder `admin-edit-form.tsx` noch die Zählpunkt-Adresse; keine semantische Überschneidung.

### Automatisierte Tests

| Prüfung | Ergebnis |
|---|---|
| `npx tsc --noEmit` | 0 Fehler |
| `npx vitest run` (Node 22.23.2) | 25 Dateien, 351 Tests grün |
| `npm run lint` | 0 Fehler, 6 Warnungen (alle in Bestand-Dateien, keine in geänderten Dateien) |
| `npm run build` | ok |
| `go test ./...` | ok |
| `npm audit --audit-level=high` | 0 high/critical (10 moderate, Bestand) |
| CI (`gh pr checks 129`) | **keine Checks** — PR ist `CONFLICTING`, GitHub startet `pull_request`-Workflows erst nach Konfliktauflösung (QA-1) |

### Security-Smoke-Test

Nur Admin-UI. Keine neuen oder geänderten Endpoints, kein Diff an Auth, Tenant-Isolation, Schema, Status-Übergängen, Import, Helm oder CI. Eingaben gehen über den bestehenden Admin-PUT mit serverseitigen `max=`-Limits und All-or-Nothing-Prüfung; Ausgabe über React-Text (kein `dangerouslySetInnerHTML`); SQL parametrisiert (Bestand). Keine Findings. `/security-review` nicht nötig.

| Severity | Datei | Funktion | Risiko | Exploit-Szenario | Fix-Empfehlung | Confidence |
|---|---|---|---|---|---|---|
| Info | – | – | keine | – | – | High |

### Bugs / Findings

| ID | Severity | Befund | Repro / Ort | Empfehlung |
|---|---|---|---|---|
| QA-1 | Medium (Merge-Blocker, kein Feature-Bug) | PR #129 hat Konflikte mit `main` in `CHANGELOG.md` und `docs/user-guide/changelog.md` (beide mit neuem `2026-11-10`-Block durch PROJ-58 #128). Deshalb laufen weder CI noch E2E. | `git merge-tree --write-tree origin/main HEAD` → CONFLICT in beiden Dateien | Auf `main` rebasen, die Einträge in je einen `## 2026-11-10`-Block zusammenführen, CI inkl. E2E abwarten. |
| QA-2 | Low | In `features/INDEX.md` stand eine Leerzeile vor der PROJ-132-Zeile; die Zeile fiel dadurch aus der Tabelle. | `features/INDEX.md` (aus dem Spec-Commit) | Mit diesem QA-Commit entfernt (Bookkeeping). |
| QA-3 | Info | Beitrittsformular: Nur-Leerzeichen-Adresse schaltet die Option nicht mehr automatisch an. | `metering-point-fields.tsx:873` | Keine Aktion; kein realer Datenzustand betroffen. |

### Einschränkungen

- Kein Browser-Lauf (Chrome/Firefox/Safari, 375/768/1440 px): kein laufendes Backend in dieser Sitzung. AC-8 nur am Code geprüft; der Raster ist identisch zum Beitrittsformular. Ein manueller Blick im Test-Cluster ist empfohlen.
- Kein Playwright-Spec ergänzt; es gibt keinen bestehenden Admin-Bearbeiten-E2E. Die Abdeckung liegt in `src/components/admin-edit-meter-address.test.tsx` (7 Tests) und `src/lib/meter-address.test.ts`.

## Deployment
_To be added by /deploy_
