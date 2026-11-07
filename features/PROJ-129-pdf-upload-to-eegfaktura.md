# PROJ-129: PDFs eines Antrags nach eegFaktura hochladen

## Status: In Review
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
+-- Schalter „Dokumente bei Aktivierung automatisch nach eegFaktura hochladen"   NEU — nur Pro
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

Index auf (Antrag, Dokumentart, Zeitpunkt). Die Detailansicht zeigt die Liste; „bereits hochgeladen am"
ist der letzte erfolgreiche Eintrag je Dokumentart. Dokumente selbst werden **nicht** gespeichert.

**Neue EEG-Einstellung** an `registration_entrypoint`: „automatisch bei Aktivierung hochladen",
ja/nein, Standard **nein** (auch für alle bestehenden EEGs). Läuft durch die volle Kette einer
Admin-Einstellung (Typ, Formular, Speichern, Go-DTO, Service, Repo) **und** durch Konfigurations-Export/-Import.

Filestore-Adresse = `{CORE_BASE_URL}/filestore` (keine eigene Konfiguration).

`docs/domain-model.md` und `docs/api-spec.md` werden ergänzt (neue Tabelle ist sonst „undokumentiert").

### C) Entscheidungen und Begründung

- **Pro-only, doppelt abgesichert:** UI blendet Aktion und Schalter in Standard aus; der Server lehnt
  Upload und Schalter-Änderung bei Standard-EEGs ab (UI allein reicht nicht). Beim Wechsel auf Standard
  blockiert ein aktiver Auto-Schalter wie die übrigen Pro-Funktionen. Bereits hochgeladene Dokumente
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
  für diese Dokumentart noch keinen erfolgreichen Eintrag gibt (Re-Aktivierung nach „Aktivierung
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
- **Ziel kann sich zwischen Dialog und Upload ändern:** der Dialog zeigt das Ziel, der `POST` ermittelt es
  erneut. Ändern sich Mitgliedsnummer oder Teilnehmer-ID dazwischen, kann der Upload an ein anderes
  Mitglied gehen. Follow-up (PR 3): bestätigte `participantId` mitsenden, bei Abweichung ablehnen.
- **Teilnehmerliste bei jedem Dialog-Öffnen:** jedes Öffnen des Bestätigungsdialogs liest die komplette
  Core-Teilnehmerliste der EEG (akzeptiert, wie im Import-Dialog).
- **Spalte „Von":** zeigt nur eine gekürzte Keycloak-ID (ersten 8 Zeichen, voller Wert im Tooltip), keinen
  Admin-Namen.
- **Zielmitglied wechselt nach Reset + erneutem Import:** die „bereits hochgeladen"-Prüfung kennt nur
  (Antrag, Dokumentart), nicht das Zielmitglied → Rückfrage, obwohl das neue Mitglied das Dokument
  noch nicht hat; „erneut hochladen" bestätigen genügt.
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

## Deployment
_To be added by /deploy_
