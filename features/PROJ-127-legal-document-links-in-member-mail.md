# PROJ-127: Rechtsdokument-Links in der Eingangsbestätigung an das Mitglied

## Status: Planned
**Created:** 2026-10-13
**Last Updated:** 2026-10-13

## Dependencies
- Requires: PROJ-9 (EEG-spezifische Rechtsdokumente) — Consent-Snapshot (Titel, URL, Zeitstempel)
- Requires: PROJ-36 (Optionale Rechtsdokumente als Info-Dokumente) — `consent_type` explicit / informational
- Berührt: PROJ-6 (E-Mail-Benachrichtigungen), PROJ-122 (Re-Submit nutzt dieselbe Mitglieds-Mail)

## Hintergrund

Seit 1. 10. 2026 verlangt § 69 Abs 1 Z 2 ElWG, dass Haushaltskunden und Kleinunternehmen
die Allgemeinen Lieferbedingungen (ALB) und ein Informationsblatt **nachweislich vor
Vertragsabschluss** zur Verfügung gestellt bekommen (Nutzer-Rückmeldung 2026-10-13).

Die Erfassung ist bereits möglich: EEG legt ALB und Informationsblatt als Rechtsdokumente
mit „Zustimmung erforderlich" an (PROJ-9) → Pflicht-Checkbox, Snapshot mit Titel, URL und
Zeitstempel. Zwei Lücken in der Eingangsbestätigung an das Mitglied
(`application_submitted_member.html`):

1. Dokumente erscheinen nur mit Titel und Zeitstempel — **ohne Link**. Das Mitglied hat
   die Dokumente danach nicht mehr verlässlich zur Hand; die Übergabe ist im eigenen
   Postfach nicht dokumentiert.
2. Jeder Eintrag wird als **„Zugestimmt am …"** dargestellt — auch Info-Dokumente
   (`consent_type = informational`), bei denen nur Kenntnisnahme protokolliert wurde.

## User Stories

- Als **Mitglied** möchte ich in der Eingangsbestätigung jedes Rechtsdokument mit Link
  finden, damit ich ALB, Informationsblatt & Co. später jederzeit wieder öffnen kann.
- Als **Mitglied** möchte ich die konkrete URL sehen, auch wenn ich die Mail ausdrucke oder
  mein Mail-Programm Links entfernt, damit klar ist, welche Fassung ich erhalten habe.
- Als **Mitglied** möchte ich korrekt sehen, ob ich einem Dokument zugestimmt oder es nur
  zur Kenntnis genommen habe, damit die Bestätigung nicht mehr behauptet als passiert ist.
- Als **EEG-Admin** möchte ich, dass die Übergabe von ALB und Informationsblatt im Postfach
  des Mitglieds dokumentiert ist, damit ich die ElWG-Pflicht zur nachweislichen
  Bereitstellung belegen kann.
- Als **EEG-Admin** möchte ich dafür nichts zusätzlich konfigurieren müssen, damit das
  Verhalten für alle bestehenden Rechtsdokumente sofort greift.

## Acceptance Criteria

- [ ] AC-1: In der Mitglieds-Eingangsbestätigung ist der Titel jedes gespeicherten Rechtsdokuments ein Link auf die gespeicherte Snapshot-URL.
- [ ] AC-2: Unter bzw. neben dem Titel steht die Snapshot-URL zusätzlich als lesbarer Text.
- [ ] AC-3: Einträge mit `consent_type = explicit` zeigen „Zugestimmt am {Datum Uhrzeit}".
- [ ] AC-4: Einträge mit `consent_type = informational` zeigen „Zur Kenntnis genommen am {Datum Uhrzeit}".
- [ ] AC-5: Die zentrale Datenschutzerklärung des Tool-Betreibers wird nach denselben Regeln dargestellt (Link + URL + Wording je `consent_type`).
- [ ] AC-6: Gilt für alle Versandwege dieser Mail: Erst-Submit, erneuter Versand der Bestätigung, Re-Submit nach Rückfrage (PROJ-122).
- [ ] AC-7: Die Text-Alternative der Mail (falls vorhanden) enthält Titel, URL und Wording ebenfalls.
- [ ] AC-8: EEG-Benachrichtigungs-Mail bleibt unverändert.
- [ ] AC-9: Kein Schema-Change, keine neue Konfiguration; bestehende Consent-Snapshots werden unverändert verwendet.

## Edge Cases

- **Antrag ohne Rechtsdokumente / Consents:** Block bleibt leer bzw. entfällt wie bisher; keine leeren Link-Zeilen.
- **Sehr lange URL:** Wird vollständig ausgegeben und darf umbrechen (kein Abschneiden, kein horizontales Scrollen im Mail-Layout).
- **URL mit Sonderzeichen / Query-String:** Wird korrekt HTML-escaped im `href` und im sichtbaren Text; keine Template-Injection.
- **Dokument wurde nach Einreichung in den EEG-Einstellungen geändert oder gelöscht:** Mail nutzt den gespeicherten Snapshot (Titel/URL zum Einreichzeitpunkt), nicht die aktuelle Konfiguration.
- **Alt-Consents vor PROJ-36:** Haben `consent_type = explicit` (DB-Default) → „Zugestimmt am".
- **Nicht-HTTP-URL im Snapshot** (z. B. `mailto:` oder fehlerhafte Eingabe): wird nur als Text dargestellt, nicht als klickbarer Link, falls Schema nicht `http`/`https`.

## Out of Scope

- Speichern der Dokument-Inhalte oder Content-Hash (Dokumentenverwaltung = Non-Goal V1). Empfehlung an EEGs bleibt: versionierte URLs (z. B. `/alb-2026-10.pdf`).
- Checkbox-Formulierung pro Dokument im Formular („zur Kenntnis genommen" statt „stimme zu").
- Information von Bestandsmitgliedern.
- EEG-Benachrichtigungs-Mail, Genehmigungs-PDF, Beitrittsbestätigung.
- Prüfung, ob eine EEG unter die ElWG-Schwellenwerte fällt (Verantwortung der EEG).

## Technical Requirements

- Nur Mail-Template + ggf. Template-Datenaufbereitung; Go `html/template`-Escaping bleibt aktiv.
- Keine PII-Änderung im Logging.

---
<!-- Sections below are added by subsequent skills -->

## Tech Design (Solution Architect)
_To be added by /architecture_

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
