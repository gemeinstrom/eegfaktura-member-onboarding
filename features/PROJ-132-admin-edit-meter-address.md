# PROJ-132: Abweichende Zählpunkt-Adresse im Admin-Bearbeiten

## Status: Architected
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
- [ ] AC-4: Ist die Option an, sind alle vier Felder Pflicht. Das Formular zeigt Feldfehler vor dem Speichern. Der Server prüft dieselbe Regel (besteht schon) und lehnt halbe Adressen ab.
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

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
