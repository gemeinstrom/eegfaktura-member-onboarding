# PROJ-127: Rechtsdokument-Links in der Eingangsbestätigung an das Mitglied

## Status: In Progress
**Created:** 2026-10-13
**Last Updated:** 2026-11-07

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

- [x] AC-1: In der Mitglieds-Eingangsbestätigung ist der Titel jedes gespeicherten Rechtsdokuments ein Link auf die gespeicherte Snapshot-URL.
- [x] AC-2: Unter bzw. neben dem Titel steht die Snapshot-URL zusätzlich als lesbarer Text.
- [x] AC-3: Einträge mit `consent_type = explicit` zeigen „Zugestimmt am {Datum Uhrzeit}".
- [x] AC-4: Einträge mit `consent_type = informational` zeigen „Zur Kenntnis genommen am {Datum Uhrzeit}".
- [x] AC-5: Die zentrale Datenschutzerklärung des Tool-Betreibers wird nach denselben Regeln dargestellt (Link + URL + Wording je `consent_type`).
- [x] AC-6: Gilt für alle Versandwege dieser Mail: Erst-Submit, erneuter Versand der Bestätigung, Re-Submit nach Rückfrage (PROJ-122).
- [x] AC-7: Die Text-Alternative der Mail (falls vorhanden) enthält Titel, URL und Wording ebenfalls.
- [x] AC-8: EEG-Benachrichtigungs-Mail bleibt unverändert.
- [x] AC-9: Kein Schema-Change, keine neue Konfiguration; bestehende Consent-Snapshots werden unverändert verwendet.

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
_Kein eigener /architecture-Schritt — reine Template-Änderung._

## Implementation Notes (2026-11-07)

- `internal/mail/templates/application_submitted_member.html`: im Block „Erteilte Zustimmungen"
  ist der Titel jedes `DocumentConsent` ein Link auf die Snapshot-URL (nur bei `http`/`https` mit
  Host — Template-Funktion `isWebURL` in `internal/mail/service.go`); darunter in der Wert-Spalte
  steht die URL als Text (`word-break: break-all`, bricht um). Wording per `consent_type`:
  `informational` → „Zur Kenntnis genommen am", sonst „Zugestimmt am" (Alt-Consents = explicit).
- AC-5: Die zentrale Datenschutzerklärung ist selbst ein `DocumentConsent` (`is_central_policy`)
  und läuft durch dieselbe Schleife. Die separate Zeile „Datenschutzerklärung: Akzeptiert (Version …)"
  aus `privacy_accepted` bleibt unverändert.
- AC-6: Erst-Submit, erneuter Versand der Bestätigung (PROJ-31-Resend) und Re-Submit (PROJ-122)
  rendern alle über `buildMemberMailData` → dasselbe Template.
- AC-7: Die Text-Alternative entsteht über `htmlToText`: Link → „Titel (URL): Zugestimmt am …",
  darunter die URL noch einmal als Textzeile (bewusst hingenommene Doppelung im Plaintext).
- Sonderfälle: `mailto:`/`javascript:`/URL ohne Schema → nur Text, kein Link; `html/template`
  escapt `href` und Text (`&` → `&amp;`).
- Tests: `internal/mail/legal_links_test.go` (Links, Wording, Escaping, Nicht-Web-URL, Plaintext,
  `isWebURL`).
- User-Guide: 06-admin-settings (Hinweis bei Rechtsdokumenten + ElWG-Beispiel), changelog.
- Review-Fixes: `<br>` wurde in `htmlToText` nie zum Zeilenumbruch (Regex-Fehler, betraf alle
  Mails) → behoben, URL steht im Plaintext auf eigener Zeile; `isWebURL` nutzt
  `sanitize.LegalDocURL`; Template nutzt `DocumentConsent.IsInformational()` statt String-Vergleich.

## Härtung Submit-Consents (Review 1, HIGH, Owner-Freigabe 2026-11-07)

Befund: `SubmitApplication` speicherte Titel und URL der expliziten Consents ungeprüft aus dem
öffentlichen Submit-Request. Ein Angreifer konnte einen Entwurf mit fremder E-Mail-Adresse anlegen
und einen Consent `{title: "Hier bestätigen", url: "https://evil.example/…"}` mitschicken → mit
PROJ-127 wäre der Link in der Eingangsbestätigung (Absender der EEG) klickbar gewesen; vorher
erschien der fremde Titel als Text.

Fix (`internal/application/submit_consents.go`, `buildSubmitConsents`):
- Der Request sagt nur noch, **welche** Dokumente angehakt wurden (Abgleich per URL, getrimmt).
  Titel, URL und `is_central_policy` jeder gespeicherten Zeile kommen aus den `legal_document`-Zeilen
  der EEG bzw. der zentralen Datenschutzerklärung (`CENTRAL_POLICY_*`, nur wenn die EEG sie anzeigt).
- Einträge ohne passendes Dokument → Submit wird mit 400 (`consents`: „Die Rechtsdokumente wurden
  zwischenzeitlich geändert. Bitte die Seite neu laden …") abgelehnt (Warn-Log mit Anzahl, ohne
  Inhalte). Typisch: veraltetes Formular, nachdem der Admin ein Dokument geändert hat; sonst würde ein
  angehakter Consent still verloren gehen (ob alle Pflichtdokumente angehakt
  wurden, prüft der Server weiterhin nicht — Follow-up). Die externe API schickt keine Consents und ist
  nicht betroffen.
- Formular (Owner-Entscheidung 2026-11-07): Bei dieser Ablehnung lädt das Formular die aktuellen
  Rechtsdokumente nach, setzt nur die Dokument-Häkchen zurück (die Datenschutz-Checkbox nur, wenn sich
  die zentrale Datenschutzerklärung geändert hat) und bittet um erneute Bestätigung — alle anderen
  Eingaben bleiben erhalten. Schlägt das Nachladen fehl: Hinweis „Seite neu laden".
- URLs werden auf beiden Seiten getrimmt verglichen (Admin-API und `CENTRAL_POLICY_URL` garantieren
  kein Trimmen); gespeichert wird die getrimmte URL. Gemeinsame Regel „zentrale Datenschutzerklärung wird angezeigt" in
  `showsCentralPolicy` für Formular und Submit.
- Re-Submit nach Rückfrage: bereits erfasste URLs werden nicht erneut geschrieben — die erste Zeile
  mit ihrem Zeitstempel bleibt (vorher wurden alle Consents doppelt gespeichert und in der Mail doppelt
  gelistet).
- Kann die Dokumentliste beim Submit nicht geladen werden, schlägt der Submit fehl (vorher: Warnung,
  Info-Consents fehlten still).
- Bewusst **nicht** umgesetzt: Ablehnen, wenn Pflichtdokumente fehlen — die externe API
  (`POST /api/external/v1/applications`) reicht ohne Consents ein und würde sonst brechen. Follow-up.
- Verdrahtung: `NewApplicationService` bekommt Titel/URL der zentralen Datenschutzerklärung
  (wie `NewRegistrationService`).
- Tests: `internal/application/submit_consents_test.go`.
- `docs/api-spec.md` (Submit-Body) und der Kommentar an `ConsentInput` beschreiben das neue Verhalten.

Review-Runde 2 (2026-11-07), bewusst nicht umgesetzt / Follow-up:
- Re-Submit: Ein Dokument, das beim ersten Submit `informational` war und inzwischen Pflicht ist,
  bleibt „Zur Kenntnis genommen" (die neue explizite Zustimmung wird nicht zusätzlich gespeichert).
- Zwei gleichzeitige Submits desselben Antrags können weiterhin doppelte Zeilen erzeugen
  (`UpdateStatusTx` ohne Status-Bedingung — Bestand).
- Prüfung, ob alle Pflichtdokumente angehakt wurden (externe API reicht ohne Consents ein).
- Im Plaintext der Mail steht die URL zweimal („Titel (URL)" + eigene Zeile) — hingenommen.
- Alt-Daten: Consent-Zeilen von vor dem Deploy können noch Titel/URL aus dem Request enthalten und
  werden beim erneuten Versand jetzt verlinkt. Read-only-Prüfabfrage im PR; Bereinigung nur bei Treffern.
- Admin-Detailansicht zeigt den `consent_type` nicht (Bestand).

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
