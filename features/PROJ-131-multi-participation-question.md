# PROJ-131: Abfrage „Mehrfachteilnahme" im Beitrittsformular

## Status: Planned
**Created:** 2026-11-08
**Last Updated:** 2026-11-08

## Dependencies
- Berührt: PROJ-1 (Public Registration), PROJ-11 (Einleitungstext — gleicher Editor und dieselbe Bereinigung), PROJ-13 (Externe API), PROJ-60/99/112 (Daten-Export, Feldkatalog), PROJ-41/127 (Mails), Beitrittsbestätigungs-PDF (PROJ-46), Admin-Bearbeiten (PROJ-24), Config-Export/-Import (PROJ-61), PROJ-121 (Self-Edit-Link)

## Hintergrund

Tester-Wunsch 2026-10-30 (WhatsApp): Viele regionale EEGs kooperieren mit einer Bürgerenergiegemeinschaft (BEG). Neue Mitglieder der EEG sollen beim Beitritt mit **Ja/Nein** angeben können, ob sie zusätzlich an der BEG teilnehmen möchten (Mehrfachteilnahme). Die EEG braucht dafür **viel Platz für Erklärungstext**. Ein Beispiel ist ein Abschnitt mit Überschrift, zwei Absätzen (Link zur BEG, Hinweis zum Teilnahmefaktor) und darunter der Frage mit Ja/Nein-Auswahl.

Ausdrücklich **nicht** nötig: das Mitglied automatisch auch in der zweiten eegFaktura-Gemeinschaft anzulegen. Die EEG kopiert das selbst mit einem eigenen Werkzeug, das die Antwort aus dem Export liest.

## Owner-Entscheidungen (2026-11-08)

| # | Entscheidung |
|---|---|
| O1 | **Ein festes Feld** „Mehrfachteilnahme gewünscht" (Ja/Nein). Die Texte legt jede EEG selbst fest. Es gibt keine beliebigen eigenen Fragen. |
| O2 | **Einmal je Antrag**, bezogen auf das Mitglied, nicht je Zählpunkt. |
| O3 | **Ein/Aus je EEG.** Standard ist aus. Ist die Frage eingeschaltet, muss das Mitglied Ja oder Nein wählen (Pflicht). Ein „optional“ gibt es nicht. |
| O4 | **Ausgabe:** Antrags-Detail und Bearbeiten (Admin), Daten-Export und Datenweiterleitung, Mails an EEG und Mitglied, Beitrittsbestätigung (PDF). |
| O5 | **Erklärungstext** im selben Editor wie der Einleitungstext (Fett, Kursiv, Absätze, Listen, Links), serverseitig gleich bereinigt. |
| O6 | **Neutrale Vorgabetexte.** Überschrift „Teilnahme an einer weiteren Energiegemeinschaft", Frage „Möchtest du zusätzlich an einer weiteren Energiegemeinschaft teilnehmen?", Erklärung leer. Die EEG kann alles anpassen. Speichern geht nur mit einem Frage-Text. |
| O7 | **Externe API** nimmt das Feld optional an und prüft es wie das Formular. Es wird dort nicht erzwungen. |

## User Stories

- Als **EEG-Admin** möchte ich im Beitrittsformular eine Ja/Nein-Frage zur Teilnahme an unserer Partner-BEG einschalten, damit neue Mitglieder gleich beim Beitritt entscheiden.
- Als **EEG-Admin** möchte ich Überschrift, Erklärungstext (mit Absätzen und Links) und Frage selbst formulieren, damit Mitglieder verstehen, worauf sie sich einlassen.
- Als **Mitglied** möchte ich die Erklärung lesen und dann Ja oder Nein wählen, damit meine Entscheidung festgehalten ist.
- Als **EEG-Admin** möchte ich die Antwort im Antrag, in den Mails, im PDF und im Export sehen, damit ich Mitglieder, die zugestimmt haben, in die BEG übernehmen kann.
- Als **EEG-Admin** möchte ich eine falsche Antwort im Antrag korrigieren können, wenn sich das Mitglied umentscheidet.

## Acceptance Criteria

**Einstellungen**
- [ ] AC-1: In den EEG-Einstellungen gibt es den Abschnitt „Mehrfachteilnahme".
  - Er enthält einen Schalter (Standard: aus), die Überschrift (einzeilig), den Erklärungstext (Editor wie beim Einleitungstext) und die Frage (einzeilig).
  - Er ist in der einfachen Ansicht sichtbar, nicht nur unter „Alle Optionen".
- [ ] AC-2: Beim ersten Einschalten sind Überschrift und Frage mit den Vorgabetexten (O6) belegt, die Erklärung ist leer.
  - Ist die Frage eingeschaltet und der Frage-Text leer, lässt sich nicht speichern (Feldfehler).
  - Die Überschrift darf leer sein; dann entfällt sie im Formular.
- [ ] AC-3: Der Erklärungstext wird serverseitig genauso bereinigt wie der Einleitungstext.
  - Es gibt keine Skripte und keine unerlaubten Tags.
  - Links öffnen in einem neuen Tab.
- [ ] AC-4: Längenbegrenzungen:
  - Überschrift höchstens 200 Zeichen.
  - Frage höchstens 300 Zeichen.
  - Erklärungstext im gleichen Rahmen wie der Einleitungstext.
  - Längere Eingaben lehnt der Server mit einem Feldfehler ab.

**Formular (öffentlich)**
- [ ] AC-5: Ist die Frage eingeschaltet, zeigt das Beitrittsformular einen eigenen Abschnitt.
  - Er besteht aus der Überschrift, dem Erklärungstext und der Frage mit zwei deutlich klickbaren Optionen „Ja" und „Nein".
  - Am Anfang ist keine Option gewählt.
  - Der Abschnitt steht vor den Einwilligungen.
- [ ] AC-6: Ohne Auswahl lässt sich der Antrag nicht einreichen (Feldfehler am Abschnitt). Der Server prüft das ebenfalls.
- [ ] AC-7: Ist die Frage ausgeschaltet, erscheint der Abschnitt nicht. Ein mitgeschickter Wert wird verworfen und nicht gespeichert.
- [ ] AC-8: Der Abschnitt funktioniert auf Mobilgeräten (375 px) und mit Tastatur. Die Ja/Nein-Auswahl ist als Gruppe mit Beschriftung zugänglich.

**Antrag (Admin)**
- [ ] AC-9: Die Antrags-Detailansicht zeigt „Mehrfachteilnahme: Ja / Nein / keine Angabe".
  - Steht der Wert auf „keine Angabe" (Altanträge oder Frage ausgeschaltet), wird die Zeile nur angezeigt, wenn die Frage bei der EEG eingeschaltet ist.
- [ ] AC-10: Im Admin-Bearbeiten-Formular lässt sich der Wert auf Ja, Nein oder „keine Angabe" setzen.
  - Das geht auch, wenn die Frage inzwischen ausgeschaltet wurde.
  - Speichern über das Bearbeiten-Formular verliert den Wert nicht.
- [ ] AC-11: Der Self-Edit-Link (Rückfrage an das Mitglied) zeigt die Frage wie das Formular, mit der bisherigen Antwort vorausgewählt.

**Ausgaben**
- [ ] AC-12: Mail an die EEG (neuer Antrag) und Eingangsbestätigung an das Mitglied enthalten die Zeile „Mehrfachteilnahme: Ja/Nein".
  - Das gilt nur, wenn ein Wert gesetzt ist.
  - Beim erneuten Einreichen nach einer Rückfrage und beim erneuten Senden gilt dasselbe, auf allen Sendewegen.
- [ ] AC-13: Die Beitrittsbestätigung (PDF) zeigt die Zeile „Mehrfachteilnahme: Ja/Nein", wenn ein Wert gesetzt ist.
- [ ] AC-14: Daten-Export und Datenweiterleitung haben die Spalte „Mehrfachteilnahme" mit den Werten Ja, Nein oder leer. Der Feldkatalog ist in Backend und Frontend gleich.
- [ ] AC-15: Die externe API nimmt das Feld optional an (Ja/Nein) und lehnt ungültige Werte mit 400 ab. Bei ausgeschalteter Frage wird der Wert verworfen.
- [ ] AC-16: Config-Export und -Import übertragen Schalter und Texte. Der Erklärungstext wird beim Import wieder bereinigt.
- [ ] AC-17: Der Import nach eegFaktura (Core) bleibt unverändert. Die Antwort wird nicht übertragen.

**Doku**
- [ ] AC-18: Der User-Guide beschreibt Einstellungen, Formular und Ausgaben, ohne PROJ-Referenzen. Ein Changelog-Eintrag ist vorhanden.

## Edge Cases

- **EEG schaltet die Frage aus, während Anträge laufen:** Bereits gespeicherte Antworten bleiben erhalten und sind in Antrag, Export, PDF und Mails sichtbar. Neue Anträge erhalten keinen Wert.
- **EEG schaltet die Frage ein:** Altanträge stehen auf „keine Angabe". Es wird nichts nachgefragt.
- **Mitglied ändert die Antwort über den Self-Edit-Link:** Der neue Wert gilt. Die Mail beim erneuten Einreichen zeigt ihn.
- **Erklärungstext mit Links oder HTML aus dem Copy-Paste einer Website:** Die Bereinigung entfernt unerlaubte Tags, erlaubte Formatierung bleibt.
- **Frage-Text gelöscht, Schalter an:** Speichern wird abgelehnt (AC-2), das Formular zeigt nie eine Frage ohne Text.
- **Mehrere RC-Nummern beim Tenant-Admin:** Schalter und Texte gelten je EEG, so wie jede EEG-Einstellung.
- **Status nach dem Import (aktiviert):** Der Wert bleibt im Antrag lesbar und änderbar, wie andere reine Onboarding-Felder.

## Out of Scope

- Automatische Anlage des Mitglieds in einer zweiten eegFaktura-Gemeinschaft (BEG) und jede Übertragung in den Core.
- Mehrere oder frei definierbare Zusatzfragen.
- Angabe je Zählpunkt oder Teilnahmefaktor.
- Auswahl einer bestimmten Partner-Gemeinschaft aus einer Liste.

## Technical Requirements

- Die Speicherung erfolgt als Spalten, kein JSON (Projektregel).
- Die Bereinigung des Erklärungstexts nutzt denselben Helfer wie der Einleitungstext (gemeinsamer Pfad, keine zweite Regelmenge).
- Der Export-Feldkatalog in Backend und Frontend wird gemeinsam gepflegt.
- Gelten die Pro-Regeln (PROJ-130) später auch hier, wird das in PROJ-130 entschieden. Bis dahin ist die Frage in jeder Edition verfügbar.

---
<!-- Sections below are added by subsequent skills -->

## Tech Design (Solution Architect)
_To be added by /architecture_

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
