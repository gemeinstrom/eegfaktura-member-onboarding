# PROJ-129: PDFs eines Antrags nach eegFaktura hochladen

## Status: Planned
**Created:** 2026-11-07
**Last Updated:** 2026-11-07

## Dependencies
- Requires: PROJ-30 / Import-Pfad — der Antrag muss als Mitglied im Core existieren (`target_participant_id`)
- Requires: PROJ-124 (SEPA-Mandat on demand reproduzieren), PROJ-46 (Beitrittsbestätigung bei Aktivierung), PROJ-76 (Beitrittserklärung mit Vorstands-Signaturblock)
- Berührt: PROJ-53 (manuelle Aktivierung ohne Import), Activation-Check-Batch (PROJ-46/PROJ-123)

## Hintergrund

Rückmeldung aus der WhatsApp-Gruppe (2026-11-06): EEGs laden SEPA-Mandat und Beitrittsdokument
heute manuell in eegFaktura beim Mitglied hoch. Ein direkter Upload aus dem Onboarding spart diesen
Schritt. Einschränkung (Dani): Bei EEGs mit Vorstands-Workflow ist das maßgebliche Beitrittsdokument
erst die **vom Vorstand signierte** Fassung — die lädt die EEG weiterhin selbst hoch und ersetzt bzw.
ergänzt damit die unsignierte.

eegFaktura speichert Mitglieds-Dokumente im Filestore als Vertragsdokument („contract") am
Teilnehmer; dort erscheinen sie in eegFaktura beim Mitglied unter den Dokumenten.

Owner-Entscheidungen 2026-11-07:
- Standardmäßig wird **nichts automatisch** hochgeladen. Ob automatisch hochgeladen wird, ist **pro
  EEG einstellbar** (Standard: aus).
- Unabhängig davon gibt es **im Antrag** — neben Herunterladen, erneut Senden usw. — immer die
  Möglichkeit, ein Dokument manuell nach eegFaktura hochzuladen.
- Automatischer Upload passiert **bei Aktivierung** (alle Dokumente zu einem Zeitpunkt).
- Hochladbar: **SEPA-Mandat, Beitrittsbestätigung, Beitrittserklärung (unsigniert)**. Keine eigenen
  Dateien vom Rechner.
- Scheitert der Upload, laufen Import/Aktivierung trotzdem weiter; der Antrag zeigt den Upload-Status
  und der Upload lässt sich nachholen.
- Erneutes Hochladen eines bereits hochgeladenen Dokuments: Hinweis mit Datum + Bestätigung.

## User Stories

- Als **EEG-Admin** möchte ich im Antrag ein Dokument (SEPA-Mandat, Beitrittsbestätigung,
  Beitrittserklärung) mit einem Klick nach eegFaktura hochladen, damit ich es nicht erst
  herunterladen und dort wieder hochladen muss.
- Als **EEG-Admin** möchte ich in den EEG-Einstellungen festlegen, dass die Dokumente bei der
  Aktivierung automatisch hochgeladen werden, damit ich mich gar nicht darum kümmern muss.
- Als **EEG-Admin mit Vorstands-Workflow** möchte ich den automatischen Upload ausgeschaltet lassen
  oder die unsignierte Beitrittserklärung bewusst hochladen können, damit in eegFaktura nicht
  ungewollt eine unsignierte Fassung als maßgebliches Dokument liegt.
- Als **EEG-Admin** möchte ich im Antrag sehen, welches Dokument wann nach eegFaktura hochgeladen
  wurde und ob ein Upload fehlgeschlagen ist, damit ich weiß, ob ich nacharbeiten muss.
- Als **EEG-Admin** möchte ich gewarnt werden, wenn ich ein Dokument ein zweites Mal hochlade, damit
  es in eegFaktura nicht versehentlich doppelt liegt.
- Als **EEG-Admin** möchte ich, dass ein fehlgeschlagener Upload weder Import noch Aktivierung
  blockiert, damit das Mitglied nicht auf einen technischen Nebenschritt warten muss.

## Acceptance Criteria

### Manueller Upload (immer verfügbar)
- [ ] AC-1: In der Antrags-Detailansicht gibt es neben „Herunterladen" eine Aktion „Nach eegFaktura
  hochladen" mit den Einträgen SEPA-Mandat, Beitrittsbestätigung, Beitrittserklärung — jeweils nur,
  wenn das Dokument für diesen Antrag auch heruntergeladen werden kann (gleiche Bedingungen wie im
  Menü „Herunterladen").
- [ ] AC-2: Die Aktion ist nur verfügbar, wenn der Antrag im Core existiert (Status `imported`,
  `ready_for_activation` oder `activated` mit gesetzter Core-Teilnehmer-Referenz). Sonst ist sie
  ausgeblendet oder deaktiviert mit Hinweis „Erst nach dem Import möglich".
- [ ] AC-3: Hochgeladen wird genau das PDF, das „Herunterladen" in diesem Moment liefert (gleiche
  Erzeugung, gleicher Inhalt).
- [ ] AC-4: Das Dokument landet in eegFaktura als Vertragsdokument am richtigen Mitglied der richtigen
  EEG, mit sprechendem Dateinamen (z. B. `SEPA-Mandat-<Mitgliedsnummer>.pdf`,
  `Beitrittsbestaetigung-<Mitgliedsnummer>.pdf`, `Beitrittserklaerung-<Mitgliedsnummer>.pdf`).
- [ ] AC-5: Nach Erfolg zeigt die UI eine Bestätigung; im Antrag ist pro Dokument sichtbar „nach
  eegFaktura hochgeladen am {Datum Uhrzeit}".
- [ ] AC-6: Wurde das Dokument schon hochgeladen, fragt ein Dialog „Bereits am {Datum} nach
  eegFaktura hochgeladen — erneut hochladen?"; erst nach Bestätigung wird erneut hochgeladen
  (in eegFaktura liegt es dann zweimal).
- [ ] AC-7: Schlägt der Upload fehl, zeigt die UI eine verständliche Fehlermeldung (ohne technische
  Details/PII); im Antrag steht „Upload fehlgeschlagen am {Datum}", und die Aktion bleibt zum
  erneuten Versuch verfügbar.
- [ ] AC-8: Jeder Upload-Versuch (manuell oder automatisch, Erfolg oder Fehler) erscheint im
  Verlauf des Antrags (Status-Log bzw. Aktivitäts-Historie), ohne Statuswechsel.

### Automatischer Upload (pro EEG, Standard aus)
- [ ] AC-9: In den EEG-Einstellungen gibt es den Schalter „Dokumente bei Aktivierung automatisch nach
  eegFaktura hochladen", Standard **aus** — auch für alle bestehenden EEGs.
- [ ] AC-10: Ist der Schalter an, werden beim Übergang auf „Aktiviert" automatisch hochgeladen:
  das SEPA-Mandat (falls der Antrag eines hat) und das Beitrittsdokument, das bei dieser Aktivierung
  entsteht — Beitrittsbestätigung, bzw. bei aktivem Vorstands-Workflow die unsignierte
  Beitrittserklärung.
- [ ] AC-11: Gilt für alle Aktivierungs-Wege: manuell „Als aktiv markieren", Activation-Check-Batch und
  manuelle Aktivierung ohne Import (PROJ-53), sofern eine Core-Teilnehmer-Referenz vorhanden ist.
- [ ] AC-12: Ein Fehler beim automatischen Upload bricht die Aktivierung nicht ab und verhindert keine
  Mails; der Antrag zeigt „Upload fehlgeschlagen", der manuelle Upload holt es nach (AC-7).
- [ ] AC-13: Bei erneuter Aktivierung nach „Aktivierung zurücksetzen" wird nicht still erneut
  hochgeladen, wenn das Dokument bereits erfolgreich hochgeladen wurde (kein Duplikat ohne Rückfrage).

### Allgemein
- [ ] AC-14: Ein Tenant-Admin kann nur Dokumente von Anträgen seiner eigenen EEG(s) hochladen; das
  Ziel in eegFaktura ist immer die EEG des Antrags.
- [ ] AC-15: In Logs erscheinen keine personenbezogenen Daten oder Dokumentinhalte (nur
  Antrags-ID, Dokumentart, Ergebnis).
- [ ] AC-16: User-Guide (Antrags-Detail + EEG-Einstellungen + E-Mails-&-PDFs-Übersicht) beschreibt
  Aktion, Schalter und den Hinweis zur signierten Beitrittserklärung.

## Edge Cases

- **Antrag noch nicht importiert** (`approved` oder früher): keine Upload-Aktion (AC-2); automatischer
  Upload greift erst bei Aktivierung.
- **PROJ-53-Aktivierung ohne Import, aber mit eingetragener Mitgliedsnummer:** nur hochladen, wenn eine
  Core-Teilnehmer-Referenz bekannt ist; sonst „nicht möglich — Mitglied in eegFaktura nicht
  verknüpft" (Klärung in /architecture, ob die Referenz über die Mitgliedsnummer ermittelt werden kann).
- **Kein SEPA-Mandat** (Einzugsart „kein SEPA") oder fehlende EEG-Stammdaten (z. B. Gläubiger-ID):
  SEPA-Eintrag fehlt bzw. zeigt denselben Hinweis wie beim Download.
- **Vorstands-Workflow aktiv + Auto-Upload an:** die unsignierte Beitrittserklärung wird hochgeladen;
  die signierte Fassung lädt die EEG weiterhin selbst in eegFaktura hoch (Hinweis im User-Guide und
  beim Schalter).
- **Import zurückgesetzt** (`reset-import`, `reset-to-review`): bereits hochgeladene Dokumente bleiben in
  eegFaktura liegen (Onboarding löscht dort nichts); Upload-Vermerke im Antrag bleiben zur
  Nachvollziehbarkeit erhalten.
- **Mitglied in eegFaktura gelöscht oder Referenz ungültig:** Upload schlägt fehl → AC-7.
- **eegFaktura/Filestore nicht erreichbar oder Zeitüberschreitung:** Fehler, kein Retry-Sturm;
  Nachholen manuell.
- **Mehrere Klicks schnell hintereinander:** höchstens ein Upload pro Klick-Sequenz (Button während des
  Uploads gesperrt).
- **Activation-Check-Batch ohne angemeldeten Admin:** der automatische Upload muss ohne Benutzer-Token
  funktionieren (Klärung in /architecture).

## Out of Scope

- Hochladen eigener Dateien vom Rechner (z. B. signierte Beitrittserklärung) — erledigt die EEG in
  eegFaktura selbst.
- Löschen oder Ersetzen von Dokumenten in eegFaktura aus dem Onboarding.
- Nachträglicher Massen-Upload für bereits aktivierte Bestandsanträge (manuell pro Antrag möglich).
- Weitere Dokumentarten (Rechnungen, Netzbetreiber-Vollmacht als eigenes PDF).
- Upload zu anderen Zielsystemen (EDA-Portal o. ä.).

## Offene Punkte für /architecture

- Authentifizierung gegenüber dem Filestore: Admin-Token (CORE_AUTH_MODE) für den manuellen Upload;
  für den Activation-Check-Batch ein Service-Konto mit Tenant-Berechtigung — Filestore prüft `aud`
  und Tenant.
- Filestore-URL pro Zone (Test/Prod) als neue Konfiguration (Helm-Values).
- Ablage des Upload-Status (pro Antrag und Dokumentart: Zeitpunkt, Ergebnis, Datei-ID in eegFaktura) →
  Schema-Änderung, Owner-Freigabe.
- Ermitteln der Core-Teilnehmer-Referenz bei PROJ-53-Aktivierungen.
- Synchroner Upload im Aktivierungs-Pfad vs. nachgelagert (Aktivierung darf nicht warten/blockieren).

## Technical Requirements

- Kein Schreiben in Core-Tabellen; Upload ausschließlich über die Filestore-Schnittstelle von eegFaktura.
- Dokumente werden für den Upload wie beim Download on demand erzeugt; das Onboarding speichert keine
  PDFs.
- `/security-review` erforderlich (neue Core-Anbindung, Auth, Tenant-Isolation, Helm).

---
<!-- Sections below are added by subsequent skills -->

## Tech Design (Solution Architect)
_To be added by /architecture_

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
