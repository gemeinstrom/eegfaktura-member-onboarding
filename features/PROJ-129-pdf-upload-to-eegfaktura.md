# PROJ-129: PDFs eines Antrags nach eegFaktura hochladen

## Status: Deployed
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
- **Nachtrag 2026-11-07 (PR 3):** Der automatische Upload wird **pro Dokument** eingestellt, nicht mit
  einem Schalter — drei Pro-Einstellungen „SEPA-Mandat automatisch hochladen", „Beitrittsbestätigung
  automatisch hochladen", „Beitrittserklärung (unsigniert) automatisch hochladen", jeweils Standard aus.
  Beispiel: nur das SEPA-Mandat automatisch, die Beitrittserklärung später signiert von Hand.
  Schema: neue Migration 000094 (drei Spalten, die ungenutzte `auto_upload_documents` entfällt).

## User Stories

- (Pro-Edition, Owner-Entscheidung 2026-11-07)
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
- [ ] AC-8: Jeder Upload-Versuch (manuell oder automatisch, Erfolg oder Fehler), der eegFaktura
  erreicht, erscheint im eigenen Bereich „Dokumente in eegFaktura" des Antrags (Upload-Protokoll,
  Owner-Entscheidung 2026-11-07 — nicht im Status-Log), ohne Statuswechsel.

### Automatischer Upload (pro EEG, Standard aus)
- [x] AC-9: In den EEG-Einstellungen gibt es pro Dokument einen Schalter „SEPA-Mandat automatisch
  hochladen", „Beitrittsbestätigung automatisch hochladen", „Beitrittserklärung (unsigniert) automatisch
  hochladen" (Owner-Entscheidung 2026-11-07), jeweils Standard **aus** — auch für alle bestehenden EEGs.
  Nur Pro-Edition; beim Schalter für die Beitrittserklärung der Hinweis, dass die signierte Fassung in
  eegFaktura von Hand ergänzt wird.
- [x] AC-10: Beim Übergang auf „Aktiviert" werden automatisch die Dokumente hochgeladen, deren Schalter
  an ist **und** die es für den Antrag gibt (gleiche Regeln wie beim Download: SEPA-Mandat nur, wenn der
  Antrag eines hat; Beitrittserklärung nur mit Vorstands-Workflow).
- [x] AC-11: Gilt für alle Aktivierungs-Wege: manuell „Als aktiv markieren", Activation-Check-Batch und
  manuelle Aktivierung ohne Import (PROJ-53), sofern eine Core-Teilnehmer-Referenz vorhanden ist.
  *Präzisiert (Security-Review PR 3):* Referenz = beim Import gespeicherte `target_participant_id`. Bei
  einer Aktivierung ohne Import (nur Mitgliedsnummer) wird **nicht** automatisch hochgeladen — bitte
  manuell mit Bestätigungsdialog.
- [x] AC-12: Ein Fehler beim automatischen Upload bricht die Aktivierung nicht ab und verhindert keine
  Mails; der Antrag zeigt „Upload fehlgeschlagen", der manuelle Upload holt es nach (AC-7).
- [x] AC-13: Bei erneuter Aktivierung nach „Aktivierung zurücksetzen" wird nicht still erneut
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
- **Vorstands-Workflow aktiv + Auto-Upload der Beitrittserklärung an:** die unsignierte
  Beitrittserklärung wird hochgeladen; die signierte Fassung lädt die EEG weiterhin selbst in eegFaktura
  hoch (Hinweis im User-Guide und beim Schalter). Wer nur die signierte Fassung in eegFaktura will, lässt
  diesen Schalter aus.
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

_Stand 2026-11-07. Owner-Entscheidungen zum Design: **nur Pro-Edition**; Upload-Versuche in einem
**eigenen Bereich im Antrag** (nicht im Status-Log)._

### Ergebnis der Klärung (offene Punkte aus der Spec)

| Frage | Ergebnis |
|---|---|
| Anmeldung beim eegFaktura-Filestore | Kein Service-Konto nötig. Alle Wege, die hochladen, starten mit einem angemeldeten Admin — auch die „Batch"-Aktivierungsprüfung ist ein Admin-Klick. Verwendet wird dasselbe Core-Token wie beim Import (`CORE_AUTH_MODE`; in Prod das von eegFaktura geholte Token). Der Filestore prüft selbst, dass das Token zur EEG gehört. |
| Wohin genau | Filestore von eegFaktura, Kategorie „Vertrag" am Mitglied, über dieselbe Upload-Schnittstelle, die eegfaktura-web nutzt. Adresse = `{CORE_BASE_URL}/filestore` (gleiche Faktura-Basis-URL, keine eigene Konfiguration). |
| Mitglied in eegFaktura finden | Nach regulärem Import ist die Core-Teilnehmer-ID gespeichert. Bei der manuellen Aktivierung ohne Import (PROJ-53) gibt es nur die Mitgliedsnummer — dann wird die ID über die vorhandene Teilnehmerliste des Cores gesucht. Nicht gefunden → Upload „fehlgeschlagen" mit Hinweis. |
| Upload-Status speichern | Neue Tabelle (Upload-Protokoll), siehe Datenmodell. Schema-Änderung → Owner-Freigabe. |
| Blockiert der Upload die Aktivierung? | Nein. Automatische Uploads starten erst **nach** erfolgreicher Aktivierung im Hintergrund; ihr Ergebnis landet im Protokoll. Der manuelle Upload läuft direkt, der Admin sieht das Ergebnis sofort. |

### A) Komponenten

```
Antrags-Detailansicht (Admin)
+-- Aktionsleiste
|   +-- „Herunterladen ▾" (unverändert)
|   +-- „Nach eegFaktura hochladen ▾"            NEU — nur Pro, nur nach Import
|       +-- SEPA-Mandat / Beitrittsbestätigung / Beitrittserklärung
|           (gleiche Sichtbarkeitsregeln wie „Herunterladen")
+-- Rückfrage-Dialog „Bereits am … hochgeladen — erneut hochladen?"   NEU
+-- Bereich „Dokumente in eegFaktura"            NEU
    +-- Liste: Dokument · Zeitpunkt · Ergebnis (hochgeladen/fehlgeschlagen) · manuell/automatisch · von wem
    +-- leer: „Noch nichts nach eegFaktura hochgeladen"

EEG-Einstellungen (Admin)
+-- Schalter je Dokument (SEPA-Mandat / Beitrittsbestätigung / Beitrittserklärung)
    „… automatisch hochladen"   NEU — nur Pro (Owner 2026-11-07: pro Dokument, siehe PR 3)
    +-- Hinweis: Bei Vorstands-Workflow wird die unsignierte Beitrittserklärung hochgeladen;
        die signierte Fassung bitte selbst in eegFaktura ergänzen.

Edition-Wechsel-Dialog (Owner)
+-- Downgrade-Block-Liste: neuer Eintrag „Automatischer Upload nach eegFaktura aktiv"
```

Backend (Go):
- **Filestore-Client** (neues kleines Paket) — lädt eine PDF-Datei zu einer EEG und einem Mitglied hoch.
- **Upload-Dienst** — erzeugt das gewünschte PDF mit den **bestehenden** Generatoren (identisch zu
  „Herunterladen"), sucht das Mitglied in eegFaktura, lädt hoch, schreibt das Protokoll.
- **Ein gemeinsamer Aktivierungs-Haken** für alle drei Aktivierungswege (manuell, Batch-Prüfung,
  PROJ-53) — Lehre aus Audit-Fund K1, wo ein Weg eine Nebenwirkung vergessen hatte.
- Zwei neue Admin-Endpunkte: Upload auslösen (mit „erneut hochladen bestätigt"-Kennzeichen) und
  Protokoll eines Antrags lesen. Beide mit derselben EEG-Berechtigungsprüfung wie die übrigen
  Antrags-Endpunkte.

### B) Datenmodell

**Neue Tabelle `member_onboarding.document_upload`** (ein Eintrag pro Upload-Versuch):
- eindeutige ID, Antrag (Fremdschlüssel)
- Dokumentart: SEPA-Mandat / Beitrittsbestätigung / Beitrittserklärung
- Auslöser: manuell / automatisch bei Aktivierung
- Ergebnis: hochgeladen / fehlgeschlagen
- Datei-ID in eegFaktura (nur bei Erfolg), Dateiname
- Fehlerart als feste Kennung (z. B. „Mitglied nicht gefunden", „eegFaktura nicht erreichbar") —
  keine Texte aus fremden Systemen, keine personenbezogenen Daten
- wer (Admin-Kennung), wann
- Zielmitglied in eegFaktura (`participant_id`, seit Fix-Pass PR 3; NULL wenn nicht auflösbar)

Index auf (Antrag, Dokumentart, Zeitpunkt). Die Detailansicht zeigt die Liste; „bereits hochgeladen am"
ist der letzte erfolgreiche Eintrag je Dokumentart. Dokumente selbst werden **nicht** gespeichert.

**Neue EEG-Einstellungen** an `registration_entrypoint`: „automatisch bei Aktivierung hochladen",
je Dokument ja/nein (seit PR 3, Migration 000094; vorher ein Schalter `auto_upload_documents`),
Standard **nein** (auch für alle bestehenden EEGs). Läuft durch die volle Kette einer
Admin-Einstellung (Typ, Formular, Speichern, Go-DTO, Service, Repo) **und** durch Konfigurations-Export/-Import.

Filestore-Adresse = `{CORE_BASE_URL}/filestore` (keine eigene Konfiguration).

`docs/domain-model.md` und `docs/api-spec.md` werden ergänzt (neue Tabelle ist sonst „undokumentiert").

### C) Entscheidungen und Begründung

- **Pro-only, doppelt abgesichert:** UI blendet Aktion und Schalter in Standard aus; der Server lehnt
  Upload und Schalter-Änderung bei Standard-EEGs ab (UI allein reicht nicht). Beim Wechsel auf Standard
  werden die Auto-Schalter ausgeschaltet (`SetEegEditionTx`, Fix-Pass PR 3; vorher war ein Block geplant,
  der nirgends durchgesetzt wurde). Bereits hochgeladene Dokumente
  bleiben in eegFaktura.
- **Admin-Token statt Service-Konto:** Jede Aktion hat einen angemeldeten Admin; dessen eegFaktura-Rechte
  gelten — ein Tenant-Admin kann nur zu seinen EEGs hochladen, der Filestore prüft das zusätzlich. Kein
  neues Geheimnis in Helm.
- **PDF on demand, nicht speichern:** gleiche Erzeugung wie beim Download → hochgeladen wird exakt, was
  der Admin herunterladen würde (Akzeptanzkriterium AC-3); keine Dokumentenverwaltung im Onboarding
  (Non-Goal V1).
- **Eigenes Protokoll statt Status-Log:** Das Status-Log bleibt reine Statusübergänge; Uploads sind
  Mehrfach-Ereignisse je Dokument und brauchen Ergebnis/Datei-ID.
- **Hintergrund nach Aktivierung:** Aktivierung, Mails und Statuswechsel sind fertig, bevor hochgeladen
  wird. Ein Fehler kann sie nicht rückgängig machen; das Protokoll zeigt „fehlgeschlagen", der Admin
  holt es per Klick nach. Muster wie bei den asynchronen Mails: Start-Vermerk, Erfolg- und Fehler-Log.
- **Kein stilles Duplikat:** Manuell fragt der Dialog nach; automatisch wird nur hochgeladen, wenn es
  für diese Dokumentart und dieses Zielmitglied noch keinen erfolgreichen Eintrag gibt (Re-Aktivierung nach „Aktivierung
  zurücksetzen", AC-13).
- **Kein automatischer Wiederholungs-Mechanismus:** einmal versuchen, Ergebnis protokollieren —
  Nachholen ist bewusst ein Admin-Klick.

### D) Abhängigkeiten

Keine neuen Pakete (Go-Standardbibliothek für den Datei-Upload; Frontend nutzt vorhandene
shadcn-Komponenten: DropdownMenu, AlertDialog, Table, Switch).

### E) Vor dem Bau zu verifizieren (Test-Zone)

1. Upload mit dem Admin-Token in beiden `CORE_AUTH_MODE`-Varianten — der Filestore prüft die
   Token-Zielgruppe (`aud`); im Modus „direct" könnte das Onboarding-Token abgelehnt werden.
2. Hochgeladenes Dokument erscheint in eegFaktura beim richtigen Mitglied unter den Dokumenten
   (wie ein manuell hochgeladener Vertrag).
3. ~~Filestore-Adresse je Zone~~ — erledigt: `{CORE_BASE_URL}/filestore` (Owner 2026-11-07).

### F) Gates und Umsetzung

- Schema-Änderung (neue Tabelle + Spalte) → Owner-Freigabe; `/security-review` (neue Core-Anbindung,
  Auth, Tenant-Isolation, Helm); Review 1 + Review 2 je PR.
- Vorschlag in drei PRs: **(1) Backend** — Filestore-Client, Tabelle, Upload-Dienst, Endpunkte,
  Pro-Prüfung; **(2) Frontend** — Aktion, Dialog, Protokoll-Bereich; **(3) Automatik** — EEG-Schalter
  (volle Kette + Config-Export + Downgrade-Block), Aktivierungs-Haken, User-Guide.

## Implementation Notes

### PR 1 — Backend (2026-11-07)

- Migration `000093_document_upload` (up/down): Tabelle `document_upload` (CHECKs für `document_type`, `trigger_source`, `result`, `error_code`, Ergebnis-Form; Index `(application_id, document_type, created_at)`) + Spalte `registration_entrypoint.auto_upload_documents BOOLEAN NOT NULL DEFAULT FALSE` (noch ungelesen). Lokal gegen Postgres 18.6 alle Migrationen 000001–000093 angewendet, 000093 down/up getestet.
- `internal/filestore`: GraphQL-Multipart-Client (Stdlib), gleicher Request wie eegfaktura-web (`addFile`, `contract`, `category_tag=user`, Bearer + `tenant`-Header); Tenant und Teilnehmer-ID als GraphQL-Variablen statt in den Query-String gespleißt. Fehler: `ErrUnreachable` (Netz/Timeout/5xx), `ErrUnauthorized` (401/403/„Tenant not allowed"), `ErrRejected` (sonst). Datei-Part mit `Content-Type: application/pdf` (Filestore lehnt andere Typen ab). Keine Response-Bodies in Fehlern/Logs.
- `internal/application/document_upload_{service,repo}.go`: Pro-Prüfung (`eeg_edition = pro`, sonst 403 `pro_edition_required`), Status `imported`/`ready_for_activation`/`activated`, Beitrittserklärungs-Regel aus dem Download geteilt (`JoiningDeclarationUnavailableReason`, Download-Handler nutzt jetzt denselben Helper), Doppel-Guard (409 `already_uploaded` + `lastUploadedAt`, außer `confirmRepeat`), PDF über die bestehenden Download-Generatoren, Teilnehmer-ID aus `target_participant_id` oder (PROJ-53) per `ListParticipants` über die Mitgliedsnummer (kein/mehrdeutiger Treffer → `participant_not_found`), Protokoll für Erfolg und Fehler.
- Endpunkte `POST`/`GET /api/admin/applications/{id}/eegfaktura-uploads` (`internal/http/admin_document_uploads.go`): `checkTenantAccess`, PROJ-71-Cool-Down-Gate (wie Downloads), Core-Token via `coreBearerToken`. Fehlversuch → 502 mit Code + protokolliertem Datensatz.
- Filestore-Adresse = `{CORE_BASE_URL}/filestore` (Owner 2026-11-07: der Filestore liegt hinter derselben Faktura-Basis-URL, keine eigene Konfiguration; die zuerst eingeführte `FILESTORE_BASE_URL` wurde wieder entfernt). Timeout = `CORE_TIMEOUT_SECONDS`. `filestore.ValidateBaseURL`: https Pflicht, http nur für localhost/Cluster-interne Hosts (Security-Review Low) — sonst bleibt der Upload aus (Warnung im Log, Endpoints 503).

Entscheidungen im Rahmen des Designs:
- Fehlversuch antwortet 502 (nicht 200), damit die UI ohne Body-Inspektion Erfolg/Fehler unterscheidet; der protokollierte Datensatz liegt im Feld `upload`.
- Abweisungen vor dem Upload (Status, Edition, Dokument nicht verfügbar, Doppel-Upload) und interne PDF-Fehler werden nicht protokolliert — das Protokoll zeigt echte Upload-Versuche.
- Spaltenname `trigger_source` statt `trigger` (SQL-Schlüsselwort vermeiden); `actor_user_id` analog `status_log.changed_by_user_id`.
- Upload unterliegt dem PROJ-71-Cool-Down-Gate auch für die Beitrittserklärung (deren Download-Endpoint hat das Gate nicht).
- Kein eigener Filestore-Timeout-Wert; `CORE_TIMEOUT_SECONDS` gilt.

Offen / in der Test-Zone zu verifizieren:
- Filestore akzeptiert das Admin-Token in `CORE_AUTH_MODE` `direct` und `exchange` (`aud`-Prüfung).
- Dokument erscheint in eegFaktura beim richtigen Mitglied.
- Strawberry nimmt `$userId: UUID` als Variable an (Abweichung von eegfaktura-web, das die ID in den Query-String schreibt).
- Kein Schutz gegen zwei gleichzeitige Klicks (UI sperrt den Button, PR 2). Eine veraltete `target_participant_id` erkennt der Filestore nicht (er prüft die Teilnehmer-ID nicht).

### PR 2 — Frontend + Ziel-Mitglied-Endpunkt (2026-11-07)

- Neuer Endpunkt `GET /api/admin/applications/{id}/eegfaktura-uploads/target` (Owner-Freigabe 2026-11-07, Security-Review-Fund „vertippte Mitgliedsnummer"): `{participantId, participantNumber?, displayName?, source: "import"|"member_number"}`. Prüfungen wie der `POST` in derselben Reihenfolge (`checkTenantAccess`, 503 ohne Filestore, PROJ-71-Cool-Down 423, 503 ohne Admin-Sitzung, 403 `pro_edition_required`, 409 `not_imported`). Fehler: 409 `participant_not_found`, 502 `participant_lookup_failed` (deutsche `message`). Schreibt kein Protokoll.
  - Gemeinsamer Code mit dem Upload: `loadEligible` (Pro + Status) und `resolveParticipant` (vorher `resolveParticipantID`); `findParticipantIDByNumber` → `findParticipantByNumber` (liefert den Eintrag statt nur die ID).
  - Bei gespeicherter `target_participant_id` liest der Endpunkt die Teilnehmerliste für Nummer und Name. Fehlt die ID in der gelesenen Liste, ist das Mitglied in eegFaktura gelöscht → 409 `participant_not_found`, der Dialog bietet kein Hochladen an (Fix-Pass 2026-11-07). Ist die Liste nicht lesbar, antwortet er 200 mit der gespeicherten ID und ohne Nummer/Name (`source: "import"`) — der Upload braucht nur die ID; der Dialog zeigt dann „das beim Import angelegte Mitglied". Der Upload selbst liest die Liste in diesem Fall weiterhin nicht (kein zusätzlicher Core-Aufruf).
  - `displayName` = Core-`firstname` + `lastname` (Firmen/Vereine/Gemeinden tragen ihren Namen in `firstname`, siehe `docs/import-mapping.md`).
- Edition in der Oberfläche: Die Detailansicht kannte die Edition bisher nicht (kein Edition-Feld in Antrag oder EEG-Einstellungen für Tenant-Admins). Explizites Signal statt 403-Raten: `GET .../eegfaktura-uploads` liefert zusätzlich `eegEdition`. Das Protokoll selbst bleibt ohne Edition-/Status-Gate lesbar (nach Downgrade sichtbar).
- Frontend: `src/lib/api/document-uploads.ts` (über `adminRequest`), `src/lib/application-documents.ts` (Dokument-Regeln gemeinsam für „Herunterladen" und „Nach eegFaktura hochladen", Fehler-Labels, Ziel-Text; Vitest), `src/components/admin-eegfaktura-uploads.tsx` (Menü + Bestätigungs-/Rückfrage-Dialog, Protokoll-Tabelle), eingebunden in `admin-application-detail.tsx`.
  - Sichtbarkeit: Menü nur bei `eegEdition = pro` und Status `imported`/`ready_for_activation`/`activated` (sonst ausgeblendet, kein deaktivierter Button); Bereich „Dokumente in eegFaktura" bei Pro nach dem Import oder sobald Einträge existieren; bei 503 (Upload nicht konfiguriert) beides ausgeblendet. Ein 503 beim Upload/Ziel wegen fehlender Admin-Sitzung zeigt die Server-Meldung statt die Aktion auszublenden.
  - Bestätigungsdialog lädt das Ziel bei jedem Öffnen neu; bei `source: "member_number"` Hinweis, das Mitglied zu prüfen; bei Fehler Grund, kein Hochladen-Button. 409 `already_uploaded` → zweiter Dialog mit Datum → erneuter `POST` mit `confirmRepeat: true`. Menü-Button gesperrt („Wird hochgeladen…"), solange ein Upload läuft. Erfolg → Toast + Protokoll neu laden; 502 → Server-`message` als Fehler-Toast + Protokoll neu laden.
  - Spalte „Von": Das Protokoll speichert nur die Keycloak-Subject-ID; angezeigt werden die ersten 8 Zeichen (voller Wert im Tooltip). Ein Admin-Name bräuchte eine zusätzliche Spalte (Follow-up, falls gewünscht).
- User-Guide: `04-admin-applications.md` Abschnitt „Dokumente nach eegFaktura hochladen" + Tabellenzeile, `07-emails-and-pdfs.md` Verweis, `changelog.md`. Screenshots fehlen (Generator braucht ein Backend mit Filestore) — Follow-up.

### Bekannte Grenzen Welle 1 (Reviews 2026-11-07, bewusst akzeptiert bzw. Folge-PRs)

- **Doppelte Kopie in eegFaktura ohne Rückfrage** möglich, wenn (a) zwei Requests gleichzeitig laufen
  (Prüfen-dann-Schreiben ohne Sperre), (b) der Filestore gespeichert hat, das Protokoll-Insert aber
  scheitert (500, lauter Fehler-Log mit Datei-ID), oder (c) der Client-Timeout greift, nachdem der
  Filestore schon gespeichert hat (als `filestore_unreachable` protokolliert). Abhilfe (a): PR 2 sperrt
  den Button während des Uploads; in eegFaktura lässt sich eine Kopie löschen.
- **Gelöschtes Mitglied in eegFaktura:** zeigt `target_participant_id` auf ein dort gelöschtes Mitglied,
  meldet der Filestore trotzdem Erfolg (er prüft den Teilnehmer nicht). Seit Welle 2 fängt der
  Bestätigungsdialog das ab (Ziel-Endpunkt → 409 `participant_not_found`, sofern die Teilnehmerliste
  lesbar ist); der `POST` selbst prüft weiterhin nicht.
- ~~**Ziel kann sich zwischen Dialog und Upload ändern**~~ — behoben in PR 3: der `POST` trägt die
  bestätigte `participantId`, bei Abweichung 409 `target_changed`.
- **Teilnehmerliste bei jedem Dialog-Öffnen:** jedes Öffnen des Bestätigungsdialogs liest die komplette
  Core-Teilnehmerliste der EEG (akzeptiert, wie im Import-Dialog).
- **Spalte „Von":** zeigt nur eine gekürzte Keycloak-ID (ersten 8 Zeichen, voller Wert im Tooltip), keinen
  Admin-Namen.
- ~~**Zielmitglied wechselt nach Reset + erneutem Import**~~ — behoben im Fix-Pass PR 3: die
  „bereits hochgeladen"-Prüfung gilt je (Antrag, Dokumentart, Zielmitglied).
- **PROJ-53 mit vertippter Mitgliedsnummer:** existiert die Nummer in eegFaktura, landet das Dokument
  beim falschen Mitglied derselben EEG. Abhilfe in PR 2: Bestätigungsdialog nennt das gefundene Mitglied.
- **Exchange-Modus ohne `X-Core-Authorization`:** fällt (wie der Import) auf das Onboarding-Token zurück;
  der Filestore lehnt es ab → protokolliert als `filestore_unauthorized`.
- **Timeouts:** PROJ-53-Uploads machen bis zu zwei ausgehende Aufrufe à `CORE_TIMEOUT_SECONDS`; dauert
  das länger als `WRITE_TIMEOUT`, geht die Antwort verloren (Protokoll stimmt, Wiederholung → 409).
  Gleiches Muster wie beim Import.
- **Keine Mengenbegrenzung** für „erneut hochladen" (jeder Versuch steht mit Admin im Protokoll).
- **Kein eigener Schalter für die Funktion** (Owner-Entscheidung 2026-11-07): aktiv, sobald deployt und
  `CORE_BASE_URL` https ist — auch bevor in der Test-Zone geprüft ist, ob eegFaktura das Admin-Token
  annimmt. Absicherung: nur Pro-EEGs, manuell auslösen; Fehlversuche landen im Protokoll.

### PR 3 — Automatik bei Aktivierung (2026-11-07)

- **Owner-Entscheidung 2026-11-07:** pro Dokument ein Schalter statt eines gemeinsamen. Migration
  `000094_auto_upload_per_document` (up/down): `auto_upload_sepa_mandate`, `auto_upload_joining_confirmation`,
  `auto_upload_joining_declaration` BOOLEAN NOT NULL DEFAULT FALSE; die nie gelesene Spalte
  `auto_upload_documents` (000093) entfällt (down stellt sie wieder her, damit 000093-down weiter läuft).
  000093 bleibt unverändert.
- **Einstellungs-Kette:** `shared.RegistrationEntrypoint` (3 Felder), Repo `GetByRCNumber` +
  `SaveAutoUploadSettings`, `SaveAllEEGSettingsTx` (Konfig-Import), `GET /settings/eeg` (+ read-only
  `eegEdition`), `PUT /settings/eeg` mit Patch-Semantik je Feld (nil = unverändert; der
  Rechtsdokumente-Editor, der nur seine Felder kennt, setzt sie so nicht zurück). Pro-Regel serverseitig
  (`resolveAutoUploadSettings`): Einschalten bei Standard → 403 `pro_edition_required` vor jedem
  Schreiben; Ausschalten und ein unverändertes TRUE bleiben erlaubt. Frontend: `EEGSettings`/Payload-Typ,
  Editor-State + Snapshot/Dirty/Discard/Save/onSaved, Schalter nur in „Alle Optionen" und nur bei Pro
  (bei Standard ausgeblendet — seit dem Fix-Pass schaltet der Wechsel auf Standard sie aus, ein TRUE bei
  Standard kommt nicht mehr vor), Awareness-Banner (`settings-mode.ts`), Konfig-Export/-Import
  (Pointer/omitempty; Import und Diff übernehmen TRUE nur bei Pro-Ziel-EEG). Der zunächst ergänzte
  Eintrag in der Downgrade-Block-Liste (`edition-switch-dialog.tsx`) ist im Fix-Pass wieder entfernt:
  der Dialog ist nirgends eingebunden, und der Wechsel auf Standard schaltet die Schalter jetzt aus.
- **Aktivierungs-Haken:** `DocumentUploadService.StartActivationUploads(coreToken, actor, ids)` — ein Haken
  für alle Wege. Aufgerufen in der HTTP-Schicht über `AdminHandler.startActivationUploads`, weil nur dort
  das Core-Token des auslösenden Admins vorliegt (`coreBearerToken(r)`): `ChangeStatus` (Ziel
  `activated`), `MarkActivated` (PROJ-53) und `CheckActivation` (Batch, nach `dispatchActivationMail`).
  Damit keine Paket-Abhängigkeit `importing → application`-Service und keine Signaturänderung von
  `ChangeStatus`/`MarkActivatedSkipImport`/`CheckActivations`. Die Massen-Aktion kann nicht aktivieren
  (nur approve/reject/under_review). Der Upload-Dienst ist bereits über `SetDocumentUploadService` in
  `cmd/server/main.go` verdrahtet; ohne ihn (nil) ist der Haken ein No-op.
  - Läuft nach dem Commit (parallel zur Aktivierungs-Mail) in einer Goroutine (Anträge nacheinander,
    1 min je Dokument), mit eigenem Kontext (der Request-Kontext endet mit der Antwort). Antwort, Aktivierung und Mails bleiben
    unverändert.
  - `UploadOnActivation`: lädt Antrag + EEG neu; überspringt ohne Protokoll, wenn der Antrag nicht mehr
    `activated` ist (Vorstands-Mail im Batch gescheitert → Rollback), die EEG nicht Pro ist oder kein
    Schalter an ist. Je eingeschaltetem Dokument `Upload` mit `trigger = activation`, ohne
    `confirmRepeat` (AC-13: bereits hochgeladen → übersprungen) und mit denselben Verfügbarkeitsregeln
    (kein Mandat / Beitrittserklärung ohne Vorstands-Workflow → übersprungen). Fehler werden wie beim
    manuellen Upload protokolliert.
  - **Ohne Core-Token:** nichts versuchen, nichts protokollieren, Warnung `activation upload: skipped, no
    core token` (die Protokoll-Fehlercodes sind eine feste Liste; „kein Token" ist kein Upload-Versuch).
    In der Praxis hat jeder dieser Handler ein Token (Keycloak-Middleware); der Admin kann manuell
    nachladen.
  - `actor_user_id` = Keycloak-Subject des auslösenden Admins (auch beim Batch).
  - Logs ohne PII: `activation upload: dispatch`, je Dokument die Zeilen des Upload-Dienstes
    (`dispatch`/`success`/`failed`) bzw. `activation upload: skipped` mit Refusal-Code.
- **Ziel-Wechsel (Security-Review PR 2):** `POST …/eegfaktura-uploads` nimmt optional `participantId`;
  weicht das jetzt ermittelte Mitglied ab → 409 `target_changed` („Das Zielmitglied hat sich geändert —
  bitte erneut versuchen."), nichts protokolliert. Die Oberfläche sendet immer die im Dialog bestätigte
  ID (auch beim „Erneut hochladen") und lädt bei 409 das Ziel neu und öffnet den Bestätigungsdialog wieder.
- **Tests:** `internal/application/document_upload_activation_test.go` (alle Schalter / nur SEPA /
  Vorstands-Workflow + Erklärung / Erklärung ohne Workflow / Standard / Schalter aus / nicht aktiviert /
  kein Token / bereits hochgeladen / kein SEPA / Fehler protokolliert / Hintergrund-Lauf / nil-Dienst /
  `target_changed`), `internal/http/admin_activation_upload_hook_test.go` (Quelltext-Prüfung per
  `go/ast`, dass `ChangeStatus`, `MarkActivated`, `CheckActivation` den Haken aufrufen; Pro-Regel des
  Speicherns; 409-Mapping), Konfig-Roundtrip + Diff (`configexport_test.go`), Vitest: Render-Test
  `admin-application-detail.uploads.test.tsx` (Stand je Antrag, Liste bleibt bei fehlgeschlagenem
  Neuladen, Fehlerkarte nur in Upload-Status, 503 blendet aus), `target_changed` im Upload-Menü,
  Awareness-Banner, Downgrade-Liste; E2E (HTTP, ohne Filestore): `participantId` wird angenommen, Settings
  liefern die drei Schalter + Edition, Einschalten bei Standard-EEG → 403.
- **Nicht geprüft:** Migration 000094 lokal gegen Postgres (kein Postgres/Docker in dieser Umgebung) —
  läuft in der CI-E2E-Umgebung. Automatischer Upload gegen einen echten Filestore (Test-Zone, wie PR 1).

#### Fix-Pass nach QA, Reviews und Security-Review (2026-11-07)

- **Token (High):** `changeApplicationStatus` und `markActivated` senden `X-Core-Authorization`
  (`coreAuthHeader`) wie Import und Aktivierungs-Prüfung — sonst scheiterte im `CORE_AUTH_MODE=exchange`
  jeder automatische Upload dieser Wege als `filestore_unauthorized`.
- **Panic-Schutz (High):** der Hintergrund-Lauf läuft außerhalb von chis Recoverer; je Antrag
  `recover()` mit Log (Typ + Stack, ohne Panic-Wert) und weiter mit dem nächsten Antrag.
- **Kein Auto-Upload ohne bestätigtes Zielmitglied (Security-Review, Medium):** ohne gespeicherte
  `target_participant_id` (Aktivierung ohne Import, nur Mitgliedsnummer) überspringt der Haken den Antrag
  (`reason=participant_unconfirmed`); eine vertippte Nummer könnte sonst das SEPA-Mandat (IBAN) ohne
  Rückfrage einem anderen Mitglied zuordnen. Der automatische Weg macht damit keine Core-Teilnehmer-Suche.
- **Re-Import an anderes Mitglied (Medium):** neue Spalte `document_upload.participant_id` (in 000094,
  noch nicht gemergt; Erfolgs-Bestand aus `target_participant_id` befüllt), bei jedem protokollierten
  Versuch geschrieben (NULL wenn nicht auflösbar), „bereits hochgeladen" je (Antrag, Dokumentart,
  Zielmitglied) — automatisch und manuell (409 `already_uploaded` nur fürs selbe Mitglied). Folge: das
  Zielmitglied wird vor der Duplikat-Prüfung aufgelöst; ist es nicht auflösbar, wird der Fehlversuch
  protokolliert statt 409 `already_uploaded` (vorher umgekehrt). Protokoll-Feld `participantId`.
- **Vertrag ausgesetzt (PROJ-71, Medium):** der Dienst bekommt dieselbe Vertragsprüfung wie
  `enforceCustomerContract` (`SetContractChecker`, verdrahtet in `main.go`); ausgesetzt → kein Upload,
  Log `reason=contract_suspended`; Prüf-Fehler → weiter (gleicher Fallback wie der HTTP-Guard).
- **Abgelehnter Token in langen Läufen:** der erste `filestore_unauthorized` beendet den Lauf; ein
  Log-Eintrag nennt die Zahl der übersprungenen Anträge. Eine Teilnehmer-Suche (401) gibt es im
  automatischen Weg nicht mehr (siehe oben).
- **Zeitbudget:** 1 min je Dokument (eigener Kontext) statt 2 min für bis zu drei Dokumente.
- **Effizienz:** `UploadOnActivation` lädt Antrag, EEG und Zielmitglied einmal je Antrag und ruft den
  internen Helfer `uploadDocument`; der manuelle `Upload` nutzt denselben Helfer (Verhalten gleich bis auf
  die Duplikat-Reihenfolge oben). `SaveEEGSettings` lädt die EEG einmal statt bis zu dreimal.
- **Edition-Wechsel:** `SetEegEditionTx` (einziger Schreiber von `eeg_edition`) setzt die drei Schalter
  bei Standard in derselben Anweisung auf FALSE; der UI-Zweig „Standard mit übrig gebliebenem TRUE nur
  ausschaltbar" ist damit unerreichbar und entfernt. Berührt den Billing-/Edition-Pfad minimal (eine
  SQL-Anweisung, kein neuer Endpunkt).
- **Quelltext-Prüfung verschärft:** `admin_activation_upload_hook_test.go` sucht per `go/ast` alle
  Nicht-Test-Aufrufer von `ChangeStatus`, `MarkActivatedSkipImport`, `CheckActivations` im Modul und
  verlangt danach einen `startActivationUploads`-Aufruf; Ausnahme `BulkChangeStatus` (Massen-Aktionen
  aktivieren nie — geprüft: kein Aufrufer nennt `StatusActivated`).
- Kleinkram: Kommentar-Platzierung in `configexport/diff.go`.

#### Bekannte Grenzen PR 3

- **Pod-Shutdown:** die Uploads laufen in einer Goroutine ohne Warteschlange; endet der Pod mitten im
  Lauf, gehen die restlichen Uploads verloren, **ohne** Protokoll-Eintrag. Nachholen manuell.
- **Seltener Doppel-Upload:** ein manueller Upload, der gleichzeitig mit dem automatischen läuft, kann
  die „bereits hochgeladen"-Prüfung ebenfalls passieren (keine Sperre) → Kopie in eegFaktura löschen.
- **Token-Ablauf in langen Läufen:** der Lauf nutzt das Core-Token des auslösenden Admins; läuft es ab
  oder passt es nicht zur EEG, endet der Lauf beim ersten `filestore_unauthorized` — die restlichen
  Anträge werden nicht versucht (nur ein Log-Eintrag mit der Anzahl, kein Protokoll). Nachholen manuell.
- **Aktivierung ohne Import:** kein automatischer Upload (bewusst, siehe oben).

### Änderung 2026-11-08: keine Edition-Sperre bis PROJ-130

Owner-Entscheidung 2026-11-08: Die Pro-Beschränkung (Owner-Entscheidung 2026-11-07, „nur Pro") ruht, bis die Editionen mit PROJ-130 eingeführt sind. Grund: In der Test-Zone waren Schalter und Menü bei einer Standard-EEG trotz „Alle Optionen" unsichtbar; eine Ausgrau-Logik gibt es noch nicht.

- Auto-Upload-Schalter: unter „Alle Optionen" immer sichtbar und schaltbar (einfache Ansicht: weiterhin nicht).
- „Nach eegFaktura hochladen" im Antrag: ab dem Import für jede EEG.
- Backend: Pro-Prüfungen entfernt (Settings-PUT, Upload, Ziel-Abfrage, Aktivierungs-Upload, Config-Import); Fehlercode `pro_edition_required` entfällt. Auch der Owner-Wechsel auf Standard lässt die Schalter unverändert (`SetEegEditionTx`).
- AC-Bezug: Die „nur Pro"-Teile von AC, Tech Design und QA-Notizen (inkl. „Wechsel auf Standard schaltet die Schalter aus") gelten bis PROJ-130 nicht; PROJ-130 führt sie als „ausgegraut" neu ein.

## QA Test Results

### Welle 1 — Backend (PR #120), 2026-11-07

**Scope:** Backend-Anteil von AC-2 bis AC-8, AC-14, AC-15. Frontend (PR 2) und Automatik/EEG-Schalter
(PR 3) sind noch nicht gebaut. Kein UI → keine Browser-/Responsive-Tests in dieser Welle.

| AC | Ergebnis | Nachweis |
|---|---|---|
| AC-1 | nicht im Scope (UI, PR 2) | — |
| AC-2 | ✅ Backend | Status-Whitelist imported/ready_for_activation/activated → sonst 409 `not_imported`; Unit-Test |
| AC-3 | ✅ | `generatePDF` ruft dieselben Generatoren wie die Download-Endpunkte; nicht verfügbare Dokumente → 409 mit Download-Begründung |
| AC-4 | ✅ Backend / ⏳ Live | Kategorie `contract`, Tenant = RC des Antrags, Dateinamen `SEPA-Mandat-<Nr>.pdf` usw.; Sichtbarkeit in eegFaktura erst in der Test-Zone prüfbar |
| AC-5 | ✅ Backend | 200 mit Upload-Datensatz; „hochgeladen am" aus dem Protokoll (UI in PR 2) |
| AC-6 | ✅ Backend | 409 `already_uploaded` + `lastUploadedAt` ohne `confirmRepeat`; Unit-Test |
| AC-7 | ✅ Backend | 502 mit fester Fehlerkennung + deutscher Meldung, Fehler im Protokoll; erneuter Versuch möglich |
| AC-8 | ✅ Backend | Protokoll-Tabelle + `GET …/eegfaktura-uploads` (AC-Text an Owner-Entscheidung angepasst) |
| AC-9 bis AC-13 | nicht im Scope (PR 3) | Spalte `auto_upload_documents` angelegt, wird noch nicht gelesen |
| AC-14 | ✅ | `checkTenantAccess` vor jeder Logik; Filestore prüft Tenant zusätzlich; E2E 401/404/400 |
| AC-15 | ✅ | Logs nur Antrags-ID, Dokumentart, Ergebnis, Fehlerkennung; Client-Fehler ohne Antworttext |
| AC-16 | nicht im Scope (User-Guide mit PR 2/3) | — |

**Automatisierte Tests:** `go vet` sauber, `go test ./... -count=1` grün (inkl. neuer Unit-Tests für
Filestore-Client und Upload-Dienst); Migration 000093 up/down/up gegen Postgres 18.6 geprüft (Backend-PR);
`helm lint` ok (kubeconform lokal nicht installiert — Trivy IaC läuft in der CI). Neu:
`tests/PROJ-129-eegfaktura-upload.spec.ts` (401 ohne Auth für POST/GET, 404 unbekannter Antrag
vor Upload-Logik, 400 ungültige ID).

**Regression:** Download der Beitrittserklärung nutzt jetzt den gemeinsamen Helfer
`JoiningDeclarationUnavailableReason` — Meldungen und 404 `not_available` identisch (Diff geprüft).

**Security-Smoke-Test:**

| Severity | Datei | Funktion | Risiko | Exploit-Szenario | Fix-Empfehlung | Confidence |
|---|---|---|---|---|---|---|
| Low | internal/application/document_upload_service.go | Upload | Doppelter Upload bei gleichzeitigen Requests | Zwei Klicks im selben Moment passieren beide die „bereits hochgeladen"-Prüfung → Dokument liegt zweimal in eegFaktura | Button in PR 2 während des Uploads sperren; optional Advisory-Lock je (Antrag, Dokumentart) | High |
| Low | internal/application/document_upload_service.go | resolveParticipantID | Upload an verwaistes Mitglied | `target_participant_id` zeigt auf ein in eegFaktura gelöschtes Mitglied; der Filestore prüft den Teilnehmer nicht → „hochgeladen", aber unsichtbar | Hinnehmen (Reset-Pfade halten die ID aktuell) oder vor dem Upload per Teilnehmerliste prüfen | Medium |
| Info | internal/http/admin_document_uploads.go | — | Swagger-Dateien nicht regeneriert | — | Mit Q-B13 `swag init` | High |
| Info | go.mod | — | excelize v2.11.0: 6 bekannte Lücken (Bestand, nicht aus diesem PR) | — | Eigenes Dependency-Update | High |

Geprüft ohne Befund: Auth/Tenant-Isolation (beide Endpunkte), SQL nur parametrisiert, keine
Server-Requests an Nutzer-URLs (Filestore-Adresse nur aus Konfiguration), keine Secrets/PII in Logs
oder Antworten, `documentType` gegen feste Liste validiert, Dateiname aus festen Präfixen + bereinigter
Mitgliedsnummer (kein Path Traversal), ohne https-`CORE_BASE_URL` = Funktion aus, Pro-Gate serverseitig.

**Offen vor Live-Nutzung (nicht vor Merge):** Test-Zone — akzeptiert der Filestore das Admin-Token
(`aud`) in `CORE_AUTH_MODE` direct/exchange; erscheint das Dokument beim richtigen Mitglied.

**Ergebnis:** 0 Critical, 0 High, 0 Medium, 2 Low, 2 Info. Welle 1 ist **merge-bereit** (Funktion
bleibt ohne https-`CORE_BASE_URL` inaktiv). `/security-review` erforderlich (neue Core-Anbindung, Auth,
Tenant-Isolation, Helm, Migration).

### Welle 2 — Frontend + Ziel-Endpoint (PR #122), 2026-11-07

**QA-Ergebnis:** 0 Critical, 0 High, 0 Medium. AC-1 bis AC-8 und AC-14 bis AC-16 bestanden (mit
Anmerkungen, siehe unten); AC-9 bis AC-13 sind Welle 3 (Automatik, EEG-Schalter). Offen: Browser- und
Responsive-Prüfung (375/768/1440) und die Test-Zone-Prüfungen (Admin-Token beim Filestore, Dokument beim
richtigen Mitglied); Screenshots im User-Guide fehlen (Generator braucht ein Backend mit Filestore).
Neu: `src/components/admin-eegfaktura-uploads.test.tsx` (Dialog mit Ziel-Mitglied, kein Hochladen bei
nicht auflösbarem Ziel, Rückfrage bei 409, Fehler-Toast + Neuladen bei 502, veraltete Ziel-Antwort).

**Reviews / Security:** `/security-review` APPROVED (ein Low: roher Core-Fehlertext im Log). Code- und
Domain-Review sowie QA meldeten u. a. einen High-Fund (Exchange-Modus) und mehrere Robustheits-Punkte.

**Behoben im Fix-Pass:**
- High: Ziel-Abfrage und Upload senden `X-Core-Authorization` (`coreAuthHeader` nach
  `src/lib/api/_internal.ts`) — ohne den Header nutzten sie im `CORE_AUTH_MODE=exchange` das falsche Token.
- Protokoll wird erst mit Sitzung geladen; Zustand je Antrag (kein Durchsickern von Edition/Einträgen nach
  Antragswechsel); ein fehlgeschlagenes Neuladen behält die zuletzt geladene Liste (Menü verschwindet nicht);
  Fehlerkarte nur in Status, in denen ein Upload möglich ist (nie bei Entwürfen).
- Bestätigungsdialog übernimmt nur die Antwort der zuletzt gestarteten Ziel-Abfrage; Dialoge behalten Titel
  und Text während der Schließ-Animation; Lade-Skeleton mit `role="status"` + Screenreader-Text.
- Backend: `List` liefert bei fehlendem Entrypoint das Protokoll mit leerer Edition statt eines Fehlers;
  `Target` meldet 409 `participant_not_found`, wenn die gespeicherte Teilnehmer-ID in der gelesenen
  Core-Liste fehlt; Logs der Mitglieds-Suche nur mit HTTP-Status bzw. Fehlertyp (kein Core-Antworttext).
- Doku: api-spec §6.16a, User-Guide (Rückfrage-Dialog ohne wörtliches Zitat, drei Dropdowns), Bekannte Grenzen.

**Abschluss-Review (Fix-Commit):** behoben — Swagger-Beschreibung des Ziel-Endpoints (409 bei gelöschtem
Mitglied), Log-Helfer redigiert nur Core-HTTP-/Parse-Fehler (Timeout/Token/Netzwerk bleiben lesbar),
Upload-Menü bei abgeschalteter Funktion ausgeblendet. Follow-up: Render-Test für das Status-Modell der
Detailansicht; Upload (POST) prüft das gelöschte Mitglied nicht selbst (nur der Dialog).

### Welle 3 — Automatik (PR #123), 2026-11-07

**QA:** 0 Critical nach Behebung von BUG-1 (Token im Exchange-Modus fehlte beim Statuswechsel und bei
der Aktivierung ohne Import). AC-9 bis AC-13 bestanden (AC-11 präzisiert: Aktivierung ohne Import lädt
nicht automatisch hoch). Migration 000094 läuft erstmals in der CI (lokal kein Postgres/Docker). Keine
Browser-/E2E-Läufe lokal; die Test-Zone-Prüfungen (Admin-Token beim Filestore, Dokument beim richtigen
Mitglied) sind weiter offen. BUG-5 (Downgrade-Sperre behauptet, aber nicht durchgesetzt) im Fix-Pass
behoben (Schalter werden beim Wechsel auf Standard ausgeschaltet, Doku angepasst).

**Reviews / Security:** Review 1 (Korrektheit + Code-Qualität) und Review 2 (Domäne) mit Funden zu
Token, Panic-Schutz, Re-Import, Vertrags-Suspendierung, Token-Ablauf, Zeitbudget, Mehrfach-Laden und
Doku-Aussagen. `/security-review` **bedingt APPROVED** — Bedingung: kein automatischer Upload ohne
bestätigtes Zielmitglied (Aktivierung ohne Import); im Fix-Pass umgesetzt.

**Behoben im Fix-Pass:** siehe „Fix-Pass nach QA, Reviews und Security-Review" unter PR 3 (Token-Header,
Panic-Schutz, `participant_unconfirmed`, `participant_id` + Duplikat je Zielmitglied, Vertragsprüfung,
Abbruch bei abgelehntem Token, Zeitbudget je Dokument, einmaliges Laden, Edition-Wechsel schaltet aus,
verschärfte Quelltext-Prüfung, Doku-Formulierungen und Bekannte Grenzen).

**Abschluss-Review (2026-11-07):** behoben — automatischer Upload vertraut einer gespeicherten Teilnehmer-ID
nur bei abgeschlossenem Import (`imported_at` gesetzt; `import_failed`-Waisen + spätere PROJ-53-Aktivierung
werden übersprungen); ein abgelehntes Token stoppt nur die übrigen Anträge **derselben EEG**, andere EEGs im
selben Batch laufen weiter; Alt-Erfolgseinträge ohne `participant_id` (PR-2-Uploads per Mitgliedsnummer,
nicht vom Backfill erfasst) zählen weiter als „bereits hochgeladen". Follow-up: Hook-Test auf Verhalten statt
Quelltext-Scan umstellen; die zwei Vertragsprüfer-Adapter in `main.go` teilen.

## Deployment

- Live in Prod seit 2026-11-08 (helm Revision 29, Image sha-84c5033, Tag v1.50.0-PROJ-85-127-128-129). Migrationen 000093/000094 angewendet. Upload für alle EEGs ohne Edition-Sperre (#125); Hinweise „nur in der Test-Umgebung“ im User-Guide entfernt.
