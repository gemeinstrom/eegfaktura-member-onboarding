# PROJ-58 — Abweichende Rechnungs-E-Mail für Org-Mitgliedstypen

**Status:** Deployed
**Implementiert:** 2026-05-21
**Erstellt:** 2026-05-21
**Quelle:** Owner-Anforderung
**Abhängigkeiten:** PROJ-8 (konfigurierbare Felder), PROJ-21 (Beitritts-PDF)

---

## Ziel

Bei Mitgliedstypen Unternehmen, Verein und Gemeinde kann optional eine
separate E-Mail-Adresse für den Rechnungsversand angegeben werden. Per
Checkbox in der Bankverbindungs-Sektion aktivierbar.

## Hintergrund

Vorbereitung für das künftige eigene Rechnungsmodul (siehe
`private/vendor-setup/psp-evaluation-2026-05-20.md`). Bei Organisationen
geht die Standard-Korrespondenz oft an einen anderen Empfänger als die
Rechnungs-/Buchhaltungs-Post. Das Feld erlaubt eine eigene
Rechnungs-Adresse, ohne die Haupt-Email zu überschreiben.

## Geklärte Entscheidungen

- **Mitgliedstypen:** company, association, municipality. Nicht
  private, farmer, sole_proprietor.
- **UI:** Checkbox „Abweichende Rechnungs-E-Mail" in der Bankverbindungs-
  Card + ein Email-Feld darunter (sichtbar wenn Checkbox aktiv).
- **field_config:** per-EEG steuerbar (hidden/optional/required),
  Default `hidden`.
- **Fallback:** Wenn Checkbox inaktiv oder Feld leer, geht die Rechnung
  an die Haupt-Email der Org.

## Datenmodell

Zwei neue Spalten auf `application`:

- `has_billing_email` (BOOLEAN NOT NULL DEFAULT FALSE)
- `billing_email` (TEXT NULL)

Service-Layer cleart das billing_email-Feld auf NULL, wenn
has_billing_email=false oder der Mitgliedstyp nicht in der Org-Liste
liegt.

## field_config

Ein Eintrag `billing_email` mit Default `hidden`. Bei optional/required
wird die Checkbox im Public-Formular angezeigt; bei required muss die
Checkbox aktiv UND das Email-Feld befüllt sein.

## Frontend

- Checkbox in der Bankverbindungs-Card, sichtbar nur bei
  Org-Mitgliedstypen UND field_config != hidden
- Wenn aktiv: Email-Input darunter
- Validierung: wenn Checkbox aktiv, Email Pflicht + Format-Check

## Admin-UI

- Detail-View zeigt billing_email wenn gesetzt
- Edit-Form: Checkbox + Email-Feld editierbar

## PDF

In der Bankverbindungs-Sektion des Beitritts-PDFs als zusätzliche Zeile
„Rechnungs-E-Mail:" gerendert, wenn gesetzt.

## Mail

Vorerst nicht relevant — aktuell werden noch keine Rechnungen versendet.
Sobald das Billing-Modul kommt (separater Workstream), wird die
billing_email als Versand-Adresse genutzt.

## Validierung

- `hasBillingEmail=true` ⇒ billing_email Pflicht + Email-Format
- `hasBillingEmail=false` ⇒ Wert serverseitig auf NULL geclearted
- Mitgliedstyp nicht in Org-Liste ⇒ Wert geclearted
- field_config = hidden ⇒ Wert geclearted

## Out of Scope (V1)

- Mehrere Rechnungs-E-Mails
- Abweichende Rechnungs-Adresse (Straße/PLZ) — nur Email
- Rolle der Rechnungs-Email (z. B. „Buchhaltung")
- BCC / CC-Logik beim tatsächlichen Versand (kommt mit Billing-Modul)

---

## Deployment

**Deployed:** 2026-05-21 (Implementierung + Helm-Tag-Bump), Bookkeeping post-hoc 2026-05-23
**Chart version:** 1.9.1
**Image SHA:** `sha-dd2376c` (Stand inkl. Docs-Sync für PROJ-56/57/58)
**Helm-Tag-Bump-Commit:** `42d4f34`
**Git tag:** `v1.9.1-PROJ-56-57-58` (bundled)
**Migration:** Schema-Migration `000051_billing_email` lief automatisch über pre-upgrade Hook-Job

Bookkeeping wurde am 2026-05-23 nachgezogen. Feature ist seit 2026-05-21 in Prod.

## Nachtrag 2026-11-10: Übertragung beim Import

Tester-Rückmeldung 2026-11-09: Die Rechnungs-E-Mail kam beim Import nicht nach eegFaktura (V1 hatte keine Übertragung vorgesehen). Der Core hat je Teilnehmer ein E-Mail-Feld (`contact.email`), nimmt dort aber mehrere Adressen getrennt durch `;` an und verschickt Rechnungen und Mitglieds-Mails an jede (`eegfaktura-backend` `model.ValidateEmailList`, Billing `MailTemplateValues`). Owner-Entscheidung: Der Import überträgt `haupt;rechnung` (Haupt-E-Mail zuerst; nur bei Org-Mitgliedstypen; gleiche Adresse nur einmal; ohne Toggle oder leere Rechnungs-E-Mail nur die Haupt-E-Mail). Eine Rechnungs-E-Mail, die die Core-Regel je Adresse nicht erfüllt (z. B. Umlaut), wird weggelassen, damit der Import nicht scheitert. Der Import-Dialog zeigt den vom Backend berechneten Wert (`importContactEmail`). Rückweg: Resync schreibt nur den ersten Teil zurück, die Reconciliation indexiert jeden Teil. Helfer: `shared.CoreContactEmail` / `shared.SplitCoreEmails`. Mapping: `docs/import-mapping.md`. Self-Reviews: Resync-Rückschreiben (Review 1+2), Umlaut-Import-Fehler, Org-Prüfung, Reconciliation, Dialog-Doppelregel — alle behoben.

### Security Review (2026-11-10)

**Scope:** PR #128 (Core-Import `contact.email`, Resync PROJ-70, Reconciliation PROJ-69, Admin-Detail-Feld `importContactEmail`, Import-Dialog).

**Threat Model:** Der Import schreibt in den eegFaktura-Core. Eine falsche E-Mail-Liste könnte Mails an fremde Adressen lenken oder den Import scheitern lassen. Resync und Reconciliation lesen Core-Daten zurück und könnten Onboarding-Daten verfälschen oder falsche Anträge verknüpfen.

| Severity | File | Area | Risk | Scenario | Fix | Confidence |
|---|---|---|---|---|---|---|
| Low | `internal/application/reconciliation_service.go` | `buildMatchIndex` | Doppelte Adresse in der Core-Liste → zwei Kandidaten → „ambiguous“ | Admin trägt in eegFaktura `a@x.at;A@x.at` ein | Behoben: je Teilnehmer jeder Schlüssel nur einmal, Test ergänzt | High |
| Info | `internal/application/reconciliation_service.go` | Match über Rechnungs-Adresse | Antrag passt auf die Rechnungs-Adresse eines anderen Teilnehmers | Braucht zusätzlich dieselbe IBAN; bei zwei Treffern „ambiguous“ (sicher) | Keine | High |
| Info | `internal/shared/core_email.go` | Haupt-E-Mail ohne Core-Regel | Haupt-E-Mail mit Umlaut lässt den Import scheitern | Bestand vor diesem PR, keine Regression | Keine | High |

**Geprüft ohne Befund:**
- Mandanten-Trennung unverändert: Die Reconciliation lädt Teilnehmer je RC-Nummer, das Detail-Endpoint prüft den Tenant.
- `importContactEmail` kombiniert nur `email` und `billingEmail`, die die Admin-Detailansicht ohnehin liefert.
- Resync kann keine `;`-Liste mehr in `application.email` schreiben; `billing_email` wird nicht zurückgelesen.
- Keine neuen Logs mit E-Mail-Adressen, keine neuen Fehlerantworten.
- Keine Schema-Änderung, keine neuen Abhängigkeiten.

**Scans:** gosec (shared, importing, application): 0 Issues. govulncheck und npm audit unverändert, weil `go.mod` und `package.json` nicht geändert sind. Semgrep/Trivy laufen in der PR-CI.

**Verdict: APPROVED** — keine Critical/High/Medium-Funde.

### Deployment Nachtrag

- Live in Prod seit 2026-11-10 (helm Revision 30, Image sha-70534b7, Tag v1.51.0-PROJ-58-131-132).
