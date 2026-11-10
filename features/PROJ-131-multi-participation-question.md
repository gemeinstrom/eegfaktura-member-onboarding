# PROJ-131: Abfrage „Mehrfachteilnahme" im Beitrittsformular

## Status: Deployed
**Created:** 2026-11-08
**Last Updated:** 2026-11-08

## Dependencies
- Berührt: PROJ-1 (Public Registration), PROJ-11 (Einleitungstext — gleicher Editor und dieselbe Bereinigung), PROJ-13 (Externe API), PROJ-60/99/112 (Daten-Export, Feldkatalog), PROJ-41/127 (Mails), Beitrittsbestätigungs-PDF (PROJ-46), Admin-Bearbeiten (PROJ-24), Config-Export/-Import (PROJ-61), PROJ-121 (Self-Edit-Link)

## Hintergrund

Tester-Wunsch 2026-10-30 (WhatsApp): Viele regionale EEGs kooperieren mit einer Bürgerenergiegemeinschaft (BEG). Neue Mitglieder der EEG sollen beim Beitritt mit **Ja/Nein** angeben können, ob sie zusätzlich an der BEG teilnehmen möchten (Mehrfachteilnahme). Die EEG braucht dafür **viel Platz für Erklärungstext**. Ein Beispiel ist ein Abschnitt mit Überschrift, zwei Absätzen (Link zur BEG, Hinweis zum Teilnahmefaktor) und darunter der Frage mit Ja/Nein-Auswahl.

Ausdrücklich **nicht** nötig: das Mitglied automatisch auch in der zweiten eegFaktura-Gemeinschaft anzulegen. Die EEG kopiert das selbst mit einem eigenen Werkzeug, das die Antwort aus dem Export liest.

## Rückmeldung der Tester zur Spezifikation (WhatsApp 2026-11-08)

- **Umfang bestätigt:** Die Doppel-Anlage in der zweiten Gemeinschaft und die Regelung des Teilnahmefaktors sollen
  organisatorisch erfolgen, nicht im Member-Onboarding. Die anfragende EEG hat dafür ein eigenes Werkzeug gebaut, das
  sie auch anderen EEGs zur Verfügung stellen würde. Start mit dieser Anforderungsdefinition. Das bestätigt die
  Owner-Entscheidungen O1–O7 und „Out of Scope“.
- **Nebenbefund zum Dokumenten-Upload (PROJ-129, nicht Teil dieser Spec):** Bei einer EEG kommt die
  Beitrittserklärung erst in eegFaktura, wenn der Vorstand sie unterschrieben hat. Die automatisch hochgeladene,
  unsignierte Beitrittserklärung würde dort später durch die signierte Fassung ersetzt. Der Schalter je Dokument
  (Einstellungen → „Dokumente bei Aktivierung automatisch nach eegFaktura hochladen“) deckt das ab: Die EEG lässt
  „Beitrittserklärung (unsigniert)“ aus. Kein Handlungsbedarf, solange keine weitere EEG das anders braucht.

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

## Visualisierung der geplanten Lösung

Skizzen zur Abstimmung, kein endgültiges Design. Die Beispieltexte sind frei erfunden.

### 1. Einstellungen der EEG (Admin)

```
┌─ Einstellungen ▸ Formular ─────────────────────────────────────────────┐
│                                                                        │
│  Mehrfachteilnahme                                                     │
│  ────────────────────────────────────────────────────────────────────  │
│  [■ an ]  Frage zur Mehrfachteilnahme im Beitrittsformular anzeigen    │
│                                                                        │
│  Überschrift                                                           │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Zusätzliche Energie aus Wind- und Wasserkraft                    │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  Erklärungstext                     [ B ] [ I ] [ • ] [ 1. ] [ 🔗 ]    │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Als Mitglied kannst du gleichzeitig bei der Muster-BEG           │  │
│  │ mitmachen. Sie wird vor allem im Winter aus Windkraft            │  │
│  │ versorgt. Details unter www.muster-beg.at                        │  │
│  │                                                                  │  │
│  │ Den Teilnahmefaktor zwischen beiden Gemeinschaften passen wir    │  │
│  │ regelmäßig an; regionale Energie hat Vorrang.                    │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  Frage *                                                               │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ Möchtest du zusätzlich bei der Muster-BEG teilnehmen?            │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

Beim ersten Einschalten stehen in Überschrift und Frage die neutralen Vorgabetexte. Der Erklärungstext ist leer.

### 2. Beitrittsformular (Mitglied)

Der Abschnitt steht direkt nach den Zählpunkten (vor den „Weiteren Angaben“) und erscheint nur, wenn die EEG die Frage eingeschaltet hat.

```
┌─ Beitrittsformular ────────────────────────────────────────────────────┐
│  … Persönliche Daten · Bankverbindung · Zählpunkte …                   │
│                                                                        │
│  Zusätzliche Energie aus Wind- und Wasserkraft                         │
│  ────────────────────────────────────────────────────────────────────  │
│  Als Mitglied kannst du gleichzeitig bei der Muster-BEG mitmachen.     │
│  Sie wird vor allem im Winter aus Windkraft versorgt.                  │
│  Details unter www.muster-beg.at  ← anklickbarer Link                  │
│                                                                        │
│  Den Teilnahmefaktor zwischen beiden Gemeinschaften passen wir         │
│  regelmäßig an; regionale Energie hat Vorrang.                         │
│                                                                        │
│  Möchtest du zusätzlich bei der Muster-BEG teilnehmen? *               │
│  ┌──────────────────────────────┐┌──────────────────────────────┐      │
│  │              Ja              ││             Nein             │      │
│  └──────────────────────────────┘└──────────────────────────────┘      │
│  (anfangs nichts gewählt; ohne Auswahl kein Absenden)                  │
│                                                                        │
│  … Einwilligungen · Antrag einreichen                                  │
└────────────────────────────────────────────────────────────────────────┘
```

Auf dem Handy (375 px) stehen „Ja" und „Nein" untereinander, jeweils über die volle Breite.

### 3. Antrags-Detail (Admin)

```
┌─ Antrag ONB-… · Max Mustermann ────────────────────────────────────────┐
│  Mitglied                                                              │
│    Name ............... Max Mustermann                                 │
│    E-Mail ............. max@example.org                                │
│    Mehrfachteilnahme .. Ja                       [Bearbeiten ✎]        │
│  …                                                                     │
└────────────────────────────────────────────────────────────────────────┘
```

Im Bearbeiten-Formular lässt sich der Wert auf Ja, Nein oder „keine Angabe" setzen.

### 4. Weg der Antwort

```mermaid
flowchart LR
    S["EEG-Einstellungen<br/>Schalter + Überschrift,<br/>Erklärung, Frage"] --> F["Beitrittsformular<br/>Ja / Nein (Pflicht)"]
    X["Externe API<br/>(optional)"] --> A
    F --> A[("Antrag<br/>Mehrfachteilnahme:<br/>Ja / Nein / keine Angabe")]
    E["Admin: Bearbeiten<br/>Self-Edit-Link (Mitglied)"] --> A
    A --> D["Antrags-Detail"]
    A --> M["Mails<br/>an EEG + Mitglied"]
    A --> P["Beitrittsbestätigung<br/>(PDF)"]
    A --> X2["Daten-Export /<br/>Datenweiterleitung"]
    X2 -. "Kopier-Werkzeug der EEG" .-> B["Mitglied in der BEG<br/>(eegFaktura, manuell)"]
    A -. "nicht übertragen" .-x C["Import nach eegFaktura<br/>(Core)"]
```

Die Antwort bleibt im Onboarding. Ins eegFaktura der BEG kommt das Mitglied über das eigene Kopier-Werkzeug der EEG, das den Export liest. Beim Import nach eegFaktura wird die Antwort nicht mitgegeben.

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
  - Der Abschnitt steht direkt nach den Zählpunkten, vor den „Weiteren Angaben“ (Owner-Änderung 2026-11-10; vorher vor den Einwilligungen).
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
- [ ] AC-15: Die externe API nimmt das Feld optional an (Ja/Nein) und lehnt ungültige Werte mit 422 `validation_error` ab, wie jede andere Validierung dieser API. Bei ausgeschalteter Frage wird der Wert verworfen.
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
- **Status nach dem Import (aktiviert):** Der Wert bleibt im Antrag lesbar. Ändern lässt er sich wie alle anderen Antragsfelder nur, solange das Admin-Bearbeiten erlaubt ist (bis zum Import). *(Korrigiert nach QA 2026-11-08, BUG-2: „änderbar wie andere Onboarding-Felder“ war ein Widerspruch, denn die anderen Felder sind nach dem Import ebenfalls gesperrt.)*

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

**Stand:** 2026-11-08, vom Owner freigegeben (T1 eigene Karte, T5 Antwort bleibt erhalten, T8 PDF in beiden Varianten). Das Feature braucht Backend (Speicherung je EEG und je Antrag) und Frontend. Neue Pakete sind nicht nötig.

### A) Bausteine

```
EEG-Einstellungen (Admin, einfache Ansicht)
+-- Karte „Mehrfachteilnahme"            (neu, eigene Speicherung wie der Einleitungstext)
    +-- Schalter an/aus
    +-- Überschrift (einzeilig)
    +-- Erklärungstext  → gemeinsamer Text-Editor (aus dem Einleitungstext herausgelöst)
    +-- Frage (einzeilig, Pflicht wenn an)

Beitrittsformular (öffentlich) und Self-Edit-Link
+-- … bestehende Abschnitte …
+-- Abschnitt „Mehrfachteilnahme"        (neu, nur wenn an; nach den Zählpunkten)
|   +-- Überschrift
|   +-- Erklärungstext  → gemeinsame Anzeige wie beim Einleitungstext
|   +-- Ja/Nein-Auswahl (Pflicht, anfangs leer)
+-- Einwilligungen · Absenden

Antrag (Admin)
+-- Detailansicht: Zeile „Mehrfachteilnahme: Ja / Nein / keine Angabe"
+-- Bearbeiten: Auswahl Ja / Nein / keine Angabe (immer bearbeitbar)

Ausgaben (alle aus demselben gespeicherten Wert)
+-- Mail an EEG + Eingangsbestätigung (alle Sendewege: Einreichen, erneut Einreichen, erneut Senden)
+-- PDF: Beitrittsbestätigung und Beitrittserklärung (Vorstands-Variante teilt die Daten)
+-- Daten-Export + Datenweiterleitung (ein Feldkatalog, Backend und Frontend gleich)
+-- Externe API (optional) · Config-Export/-Import
+-- Import nach eegFaktura: unverändert, Wert wird nicht übertragen
```

### B) Datenmodell (in Worten)

**Je EEG** (Erweiterung des Eintrags in `registration_entrypoint`, vier Spalten, kein JSON):
- Schalter „Frage anzeigen" — Standard aus.
- Überschrift — Text, höchstens 200 Zeichen, darf leer sein.
- Erklärungstext — formatierter Text, serverseitig bereinigt, Längenrahmen wie beim Einleitungstext, darf leer sein.
- Frage — Text, höchstens 300 Zeichen. Pflicht, solange der Schalter an ist.

**Je Antrag** (eine Spalte in `application`):
- Mehrfachteilnahme gewünscht — drei Zustände: Ja, Nein, keine Angabe (leer).

Eine Datenbank-Migration legt die fünf Spalten an. Sie fügt nur hinzu, alle Werte starten aus bzw. leer. Nach dem Deploy ändert sich deshalb für keine EEG etwas, bis sie die Frage einschaltet. Die Migrationsnummer wird beim Bau vergeben, weil PROJ-130 einen eigenen Entwurf hat.

### C) Entscheidungen und Begründung

| # | Entscheidung | Warum |
|---|---|---|
| T1 | **Eigene Einstellungskarte mit eigener Speicherung** (neues Endpoint-Paar „Mehrfachteilnahme lesen/speichern" je RC-Nummer), nicht in der großen EEG-Einstellungs-Speicherung. | Der Erklärungstext ist langer formatierter Text wie der Einleitungstext, und der hat sich als eigene Karte bewährt. Die große Einstellungs-Speicherung bleibt unberührt, damit kein bestehendes Feld beim Speichern verloren gehen kann. Die Prüfung „Frage nötig, wenn an" bleibt an einer Stelle. |
| T2 | **Ein gemeinsamer Text-Editor**: Die Werkzeugleiste und das Eingabefeld des Einleitungstexts werden als eigener Baustein herausgelöst und an beiden Stellen genutzt. Dasselbe gilt für die Anzeige im Formular. | Das ist O5 („gleicher Editor"). Zwei Kopien würden auseinanderlaufen. Das Verhalten des Einleitungstexts ändert sich nicht. |
| T3 | **Bereinigung doppelt wie beim Einleitungstext**: Der Server bereinigt beim Speichern und beim Config-Import mit demselben Helfer. Die Anzeige im Formular bereinigt ein zweites Mal. | Der Text erscheint auf einer öffentlichen Seite. Das ist dasselbe zweistufige Muster wie beim Einleitungstext, ohne eigene Regelmenge (AC-3). |
| T4 | **Eine Prüfstelle für die Antwort** im Antrags-Service, genutzt vom Formular, vom Self-Edit-Link und von der externen API. Ist die Frage an und fehlt die Antwort, lehnt sie beim Einreichen ab. Ausnahme ist die externe API, dort ist die Antwort optional (O7). Ist die Frage aus, verwirft sie einen mitgeschickten Wert. | So kann kein Weg die Regel umgehen. Formular und Server prüfen dasselbe (AC-6, AC-7, AC-15). |
| T5 | **Ausgeschaltete Frage lässt gespeicherte Antworten unangetastet.** Speichert ein öffentlicher Weg bei ausgeschalteter Frage (z. B. Self-Edit), bleibt der alte Wert stehen; er wird weder überschrieben noch gelöscht. Das Admin-Bearbeiten darf ihn immer setzen, auch auf „keine Angabe". | Erfüllt den Edge Case „EEG schaltet aus, Antworten bleiben" und AC-10. Ohne diese Regel würde das Speichern über den Self-Edit-Link den Wert still löschen. |
| T6 | **Admin-Bearbeiten trägt das Feld auf allen Ebenen**: Typ, Formular-Zustand, Speicher-Daten, Server-Anfrage, Service und Datenbank. | Das Bearbeiten ersetzt den ganzen Antrag. Fehlt das Feld auf einer Ebene, geht der Wert beim Speichern verloren (bekannte Fehlerquelle). |
| T7 | **Mails über die gemeinsame Zusatzfeld-Liste**: Die Zeile wird dort ergänzt, wo heute schon Wärmepumpe und ähnliche Felder stehen. Sie erscheint, sobald ein Wert gesetzt ist, unabhängig vom Schalter. | Alle Sendewege nutzen diese Liste. So reicht eine Stelle für EEG-Mail, Eingangsbestätigung, erneutes Einreichen und erneutes Senden (AC-12). |
| T8 | **PDF**: Die Zeile kommt in den Mitglieds-Block. Damit steht sie in Beitrittsbestätigung und Beitrittserklärung (Vorstands-Variante). | Beide Varianten teilen dieselben Daten. Dass der Vorstand die Antwort auf der Erklärung sieht, ist gewollt: Er entscheidet über die Aufnahme mit. |
| T9 | **Export-Feldkatalog**: Die neue Spalte kommt im selben Commit in Backend und Frontend, Werte Ja / Nein / leer. Die Datenweiterleitung nutzt denselben Katalog. | Backend und Frontend sind hier schon mehrfach auseinandergelaufen (AC-14). Das Kopier-Werkzeug der EEG liest genau diese Spalte. |
| T10 | **Öffentliche Formular-Konfiguration** liefert Überschrift, Erklärung und Frage nur aus, wenn die Frage an ist. | Ist sie aus, braucht das Formular nichts, und es wird nichts Unnötiges veröffentlicht. |
| T11 | **Einstellungs-Ansicht**: Die Karte ist in der einfachen Ansicht sichtbar und zählt nicht als „erweiterte" Einstellung. | So verlangt es AC-1. Die Prüfung, ob die Ansicht auf „Alle Optionen" springen muss, bleibt unverändert. |
| T12 | **Mandanten-Trennung** wie bei jeder EEG-Einstellung: Jede Anfrage prüft zuerst die RC-Nummer gegen die Rechte des Admins. | Gleiches Muster wie beim Einleitungstext, keine neue Rechte-Logik. |
| T13 | **Kein Import nach eegFaktura**: Import-Daten und Core-Schnittstelle bleiben unverändert. | Out of Scope (AC-17). |

### D) Schnittstellen (für api-spec.md)

- **Neu:** Einstellungen „Mehrfachteilnahme" lesen und speichern je RC-Nummer (Admin, Keycloak).
  - Beim Speichern prüft der Server die Längen, verlangt eine Frage, wenn der Schalter an ist, und bereinigt den Erklärungstext.
  - Fehler meldet er als Feldfehler (400).
- **Erweitert:**
  - Öffentliche Formular-Konfiguration: neuer optionaler Block mit den drei Texten.
  - Antrag anlegen, ändern und einreichen (öffentlich, Self-Edit): neues optionales Feld Ja/Nein.
  - Admin-Antrag lesen und bearbeiten: dasselbe Feld, dreiwertig.
  - Externe API: dasselbe Feld, optional, ungültiger Wert ergibt 400.
  - Config-Export/-Import: vier neue Einstellungswerte.

### E) Abhängigkeiten

Keine neuen Pakete. Der Text-Editor (tiptap), die Auswahl-Bausteine (shadcn) und der HTML-Bereiniger (bluemonday) sind schon im Projekt.

### F) Umsetzung und Gates

- **Ein Draft-PR** (Backend + Frontend zusammen). Der Export-Katalog muss in Backend und Frontend gleichzeitig kommen. Die Ausgaben greifen nur gemeinsam sinnvoll.
- **Reihenfolge:**
  1. Migration
  2. Einstellungs-Endpoints + Prüfung
  3. Antragsfeld + Prüfstelle (T4/T5)
  4. Mails, PDF, Export, externe API, Config-Export
  5. Frontend: Editor-Baustein, Einstellungskarte, Formularabschnitt, Detail/Bearbeiten
  6. Doku: api-spec, domain-model, User-Guide mit Hinweis „nur in Test" bis zum Prod-Deploy
- **Gates:**
  - Review 1 + Review 2: Migration, öffentliches Formular, Mails/PDF, externe API
  - `/qa`
  - `/security-review`: öffentlicher Endpoint, Schema-Änderung, HTML auf öffentlicher Seite
  - Freigabe der Migration durch den Owner vor dem Anwenden
- **Tests:**
  - Prüfstelle mit allen Kombinationen aus Schalter, Weg und Wert
  - Bereinigung
  - Admin-Bearbeiten verliert den Wert nicht
  - Self-Edit bei ausgeschalteter Frage
  - Export-Katalog Backend und Frontend gleich
  - E2E: Formular mit eingeschalteter Frage, ohne Auswahl kein Absenden

## Implementation Notes

**Stand:** 2026-11-08, Draft-PR (Backend + Frontend + Doku). Migration noch nicht angewendet.

### Gebaut

- **Migration `000095_multi_participation`** (additiv): vier Spalten auf `registration_entrypoint`, eine auf `application`. Down-Migration entfernt sie wieder; lokal Down+Up in einer Transaktion geprüft.
- **Einstellungen (T1, T12):** `internal/http/admin_settings_multi_participation.go`, Routen neben dem Einleitungstext. Prüfung und Bereinigung leben in `application.NormalizeMultiParticipationSettings` und werden vom Endpoint und vom Config-Import gemeinsam genutzt. Grenzen: Überschrift 200 und Frage 300 Zeichen (Runen, nicht Bytes), Erklärung 50 KiB nach der Bereinigung (gleicher Rahmen wie `configexport.MaxIntroTextLength`, weil der Einleitungstext-Endpoint selbst keine Grenze hat).
- **Prüfstelle (T4/T5):** `internal/application/multi_participation.go`. `applyMultiParticipationAnswer` übernimmt den Wert nur bei eingeschalteter Frage (Create, Public-Update/Self-Edit, externe API über Create). `validateMultiParticipationOnSubmit` prüft beim Einreichen; `SubmitApplication` (Formular, Self-Edit) verlangt eine Antwort, die neue Methode `SubmitExternalApplication` (externe API) nicht. Eine eingeschaltete Frage ohne Frage-Text zählt als aus (kann über Endpoint/Import nicht entstehen; Schutz gegen eine Pflichtfrage, die das Formular nicht zeigen könnte).
- **Admin-Bearbeiten (T6):** Der Admin-Update ist im Backend kein echter Full-Replace, sondern Pointer-Sentinel. Damit „keine Angabe" setzbar ist, nutzt das Feld `shared.OptionalBool` (unterscheidet fehlenden Key von explizitem `null`). Das Admin-Formular schickt den Wert bei jedem Speichern mit (TS-Typ: Pflichtfeld `boolean | null`).
- **Detailansicht (AC-9):** Das Backend liefert `multiParticipationEnabled` (aktueller Schalter) im Detail mit, der Entrypoint wird dort ohnehin geladen. Die Zeile erscheint bei gesetztem Wert oder eingeschalteter Frage.
- **Mails (T7):** EEG-Mail über `buildConfigurableFields` (Zeile unabhängig von `field_config`), Eingangsbestätigung über ein eigenes Template-Feld in `buildMemberMailData`, weil das Mitglieder-Template die Zusatzfeld-Liste nicht rendert. Damit decken beide Builder alle Sendewege ab (Einreichen, Bestätigungs-Klick, erneutes Einreichen nach Rückfrage, erneutes Senden). Gemeinsamer Helfer `mail.MultiParticipationLabel` auch für das PDF.
- **PDF (T8):** Zeile im Mitglieder-Block nach der Adresse, gilt für beide Varianten.
- **Export (T9):** `multi_participation_requested` „Mehrfachteilnahme" als Bool in der Kategorie **Stammdaten** (eine Kategorie „Mitglied" gibt es nicht), Backend und Frontend im selben Commit, Test auf beiden Seiten mit denselben Werten.
- **Frontend:** `src/components/rich-text-editor.tsx` (Werkzeugleiste + Editor, aus dem Einleitungstext herausgelöst, Verhalten dort unverändert), `RichTextDisplay` in `intro-text-display.tsx` (gemeinsame DOMPurify-Bereinigung), `admin-multi-participation-editor.tsx`, `registration-form/multi-participation-section.tsx`, `src/lib/multi-participation.ts` (Vorgabetexte, Client-Prüfung, Ja/Nein/keine-Angabe-Mapping).

### Entscheidungen / Abweichungen

- **Eigener Tab statt Karte im Tab „Formular-Felder":** Die Einstellungsseite verfolgt ungespeicherte Änderungen je Tab (Confirm-Dialog, Punkt-Symbol). Ein eigener Tab „Mehrfachteilnahme" fügt sich ohne Sonderlogik ein; er ist wie alle Tabs in beiden Ansichten sichtbar und zählt nicht in `isAdvancedActive` (T11).
- **Kein Auto-Speichern in der Karte:** Nur der Speichern-Button. Ein Auto-Speichern würde beim Tippen der Frage laufend an der Pflicht-Prüfung scheitern. Der Tab-Wechsel-Schutz greift wie beim Einleitungstext.
- **Links im neuen Tab (AC-3):** Der gemeinsame Editor setzt `target="_blank"` bereits. Für eingefügtes oder importiertes HTML ohne `target` setzt `RichTextDisplay` beim Anzeigen `target="_blank" rel="noopener noreferrer"` auf alle Links. Die Server-Policy (`sanitize.HTML`) ist unverändert, damit der Einleitungstext und die Admin-Notiz nicht mitgeändert werden.
- **Externe API, ungültiger Typ:** AC-15 nennt 400. Die externe API meldet einen nicht dekodierbaren Body seit jeher mit `422 validation_error`; das bleibt so (gleicher Fehlercode wie jede andere Validierung dieser API). Dokumentiert in `docs/api-spec.md` 8.1.
- **Swagger:** `swag init` bricht seit PROJ-103 an `json.RawMessage` ab. Nachgezogen wurde nur das Feld im externen Request (`docs/swagger.*`, `docs/docs.go`), die neuen Admin-Endpoints stehen in `docs/api-spec.md`.

### Self-Reviews (2026-11-08)

- **Review 1 (Korrektheit + Qualität):**
  - Behoben: Die E2E-Spec schaltete die Frage an der gemeinsamen Test-EEG ein und konnte so parallel laufende Specs stören. Jetzt ergänzt sie die öffentliche Formular-Konfiguration nur im Browser und prüft, dass das Formular die Antwort mitschickt. Der Admin-Test überspringt sich ohne Test-Auth.
  - Behoben: Schaltet die EEG die Frage ein, während ein Formular schon offen ist, zeigt das Formular jetzt die Server-Meldung mit dem Hinweis „Bitte lade die Seite neu.“
  - Behoben: Der Erklärungstext ist erst nach dem Laden editierbar.
  - Behoben: Die Detailansicht nutzt dieselbe Regel wie das Formular (Schalter an und Frage vorhanden).
  - Vereinfacht: Hilfsfunktion `boolOrFalse` im Config-Import gemeinsam für Auto-Upload und Mehrfachteilnahme; `editorHtml` auch im Einleitungstext; leere Texte einheitlich als `null`; `maxLength` an den Eingaben entfernt (die Prüfung zählt Zeichen, nicht UTF-16-Einheiten); Schema-Block vor den SEPA-Kommentar verschoben.
- **Bewusst belassen:** Ein Config-Bundle von vor dieser Funktion schaltet die Frage beim Import aus und leert die Texte. Das ist dasselbe Muster wie bei den anderen später ergänzten Schaltern; die Vorschau im Import zeigt die Änderung vorher an.
- **Review 2 (Fachlich):** keine blockierenden Funde. Export zeigt Ja/Nein als Text (Test vorhanden).

## QA Test Results

**Stand:** 2026-11-08, /qa auf PR #127 (Head 7c9c044). Code und Tests statisch gegen jedes AC geprüft, lokale Test-Suiten ausgeführt, CI ausgewertet. Keine manuelle Browser-Session (kein lokaler Backend-/Keycloak-Stack); Cross-Browser und 375/768/1440 px nur per Code-Review (Grid `grid-cols-1 sm:grid-cols-2`, Radix-RadioGroup).

**Ergebnis: NICHT bereit** — 1 High (rote E2E-Spec), 1 Medium, 1 Low. Status bleibt **In Review**.

### Automatisierte Tests

| Prüfung | Ergebnis |
|---|---|
| `go build ./...`, `go vet ./...` | ok |
| `go test ./...` | ok (alle Pakete grün) |
| `npx vitest run` (Node 22) | 23 Dateien, 308 Tests grün |
| `npx tsc --noEmit` | ok |
| `npm run lint` | 0 Fehler, 6 Warnungen (React-Compiler `incompatible-library`, Bestand) |
| `npm run build` | ok |
| CI Backend / Frontend / Semgrep / Trivy / Safety | grün (gosec zum Prüfzeitpunkt pending) |
| CI **E2E (Playwright)** | **rot**: `tests/PROJ-131-multi-participation.spec.ts:81` (3 Versuche), siehe BUG-1 |
| `govulncheck` | 6 erreichbare Funde in `excelize/v2@v2.11.0` — Bestand, `go.mod` vom PR nicht berührt |
| `npm audit --audit-level=high` | keine High/Critical |

### Acceptance Criteria

| AC | Ergebnis | Nachweis |
|---|---|---|
| AC-1 Einstellungs-Abschnitt | Pass (Abweichung dokumentiert: eigener Tab statt Karte, in beiden Ansichten) | `src/components/admin-multi-participation-editor.tsx`, `src/app/admin/settings/page.tsx`; Vitest `admin-multi-participation.test.tsx` |
| AC-2 Vorgabetexte, Frage Pflicht, Überschrift optional | Pass | `src/lib/multi-participation.ts` (`applyEnableDefaults`, `validateMultiParticipationSettings`); Server `NormalizeMultiParticipationSettings` (`internal/application/multi_participation.go`); `TestNormalizeMultiParticipationSettings`, `TestSaveMultiParticipation_FieldErrors` |
| AC-3 Bereinigung wie Einleitungstext | Pass | `sanitize.HTML` in `sanitizeToNil`; Client DOMPurify mit gleicher Allow-List (`intro-text-display.tsx` `useSanitizedHtml`), Links `target=_blank rel=noopener noreferrer` in `RichTextDisplay`; Test „explanation sanitized like intro text" |
| AC-4 Längen 200/300/50 KiB, Feldfehler | Pass | Runen-Zählung in `multi_participation.go`; Tests „length limits count characters", „explanation limit" |
| AC-5 Abschnitt, Ja/Nein, nichts vorgewählt, vor Einwilligungen | Pass (Code + Vitest); E2E rot wegen Testfehler (BUG-1) | `registration-form/multi-participation-section.tsx`, `registration-form.tsx` (vor `ConsentSection`), `defaults.ts`; Vitest „startet ohne Auswahl" |
| AC-6 ohne Auswahl kein Einreichen, Server prüft | Pass | Zod-superRefine in `schema.ts`; Server `validateMultiParticipationOnSubmit` in `submitApplication`; `TestMultiParticipation_CheckLocation` |
| AC-7 ausgeschaltet: kein Abschnitt, Wert verworfen | Pass | `buildMultiParticipationConfig` (nil wenn aus), `applyMultiParticipationAnswer`; `payload.ts` sendet nur bei vorhandener Konfiguration |
| AC-8 375 px, Tastatur, Gruppe beschriftet | Pass (Code-Review) | RadioGroup mit `aria-labelledby` + `aria-required`, `grid-cols-1 sm:grid-cols-2`, `min-h-11` |
| AC-9 Detail-Zeile, „keine Angabe" nur bei aktiver Frage | Pass | `admin-application-detail.tsx:777`; `admin_service.go` setzt `MultiParticipationEnabled = multiParticipationActive(ep)` |
| AC-10 Admin-Bearbeiten Ja/Nein/keine Angabe, auch bei ausgeschalteter Frage, kein Verlust | Pass (für bearbeitbare Status, siehe BUG-2) | 6 Layer: TS `_admin-types.ts` (`boolean \| null`, Pflichtfeld) → State `admin-edit-form.tsx` → Payload `fromMultiParticipationChoice` → DTO `OptionalBool` (`requests.go`) → Service `AdminUpdateApplication` (`.Set`) → SQL `UpdateAdminTx` `$36`; `TestOptionalBool_AdminUpdateRequest`, Vitest „verliert den Wert beim Speichern nicht", „sendet keine Angabe als null" |
| AC-11 Self-Edit mit Vorauswahl | Pass | `GetApplicationForEdit` liefert den Wert, `from-application.ts`; Vitest „Self-Edit-Link: bisherige Antwort ist vorausgewählt" |
| AC-12 Mails EEG + Mitglied, alle Sendewege | Pass | `buildConfigurableFields` (EEG-Mail) und `buildMemberMailData` (`SendSubmissionEmails` + `SendMemberConfirmation`); `TestEEGTemplate_RendersMultiParticipation`, `TestMemberTemplate_MultiParticipationRow` |
| AC-13 PDF | Pass | `approval_pdf.go` Mitglieder-Block, `buildApprovalPDFData`; `TestFPDFApprovalGenerator_MultiParticipation_ChangesOutput` |
| AC-14 Export-Spalte, Katalog FE = BE | Pass | `fields.go` + `data-export-fields.ts` mit gleichem Key/Label/Kategorie/Typ; `TestPROJ131_MultiParticipationField`, `data-export-fields.test.ts` |
| AC-15 Externe API optional, ungültig → 400 | Pass mit Abweichung (BUG-3) | `external.go` Feld + `SubmitExternalApplication`; ungültiger Typ → 422 statt 400 |
| AC-16 Config-Export/-Import, erneute Bereinigung | Pass | `exporter.go`, `importer.go` `validateAndSanitize` (gleicher Helfer), `diff.go`; `TestMultiParticipation_ConfigRoundtrip`, `…ImportSanitizesAndValidates`, `…LegacyBundleDefaultsOff` |
| AC-17 Core-Import unverändert | Pass | kein Diff in `internal/importing`, `internal/coreclient` |
| AC-18 User-Guide ohne PROJ-Refs, Changelog, Test-Hinweis | Pass (vorhanden, `grep PROJ- docs/user-guide` leer), aber eine falsche Aussage (BUG-2) | `docs/user-guide/02/04/06/07`, `changelog.md` |

**Summe:** 18/18 AC funktional erfüllt (AC-15 mit dokumentierter Statuscode-Abweichung); ein Edge Case nicht erfüllt (BUG-2); E2E-Nachweis für AC-5/6 fehlt (BUG-1).

### Edge Cases

| Edge Case | Ergebnis |
|---|---|
| EEG schaltet aus, Antworten bleiben | Pass (T5; Mails/PDF/Export unabhängig vom Schalter) |
| EEG schaltet ein, Altanträge „keine Angabe" | Pass. Hinweis: Läuft eine Rückfrage, muss das Mitglied beim erneuten Einreichen über den Self-Edit-Link antworten (Pflicht) — konsistent mit AC-11 |
| Self-Edit ändert Antwort, Mail zeigt neuen Wert | Pass |
| HTML aus Copy-Paste | Pass (bluemonday + DOMPurify) |
| Frage gelöscht, Schalter an | Pass (Server + Client; „an ohne Frage" zählt serverseitig zusätzlich als aus) |
| Mehrere RC-Nummern | Pass (`parseRCAndCheck` je Request) |
| Status nach Import (aktiviert): Wert lesbar, Bearbeiten wie alle Felder bis zum Import | Pass nach Korrektur von Spec und User-Guide (BUG-2) |

### Bugs

| # | Severity | Befund | Repro / Ort | Empfehlung |
|---|---|---|---|---|
| BUG-1 | **High** | Neue E2E-Spec scheitert in CI (3/3 Versuche): `tests/PROJ-131-multi-participation.spec.ts:81` „Formular: ohne Ja/Nein kein Absenden…" findet die Überschrift nicht. Ursache: `enableQuestionInConfig` mockt `GET /api/public/registration/{rc}` per `page.route`, die Seite `src/app/register/[rc_number]/page.tsx:44` lädt die Konfiguration aber serverseitig (SSR) — der Browser-Mock greift nie (derselbe Hinweis steht in `tests/PROJ-11-registration-intro-text.spec.ts:30`). Damit ist die PR nicht mergebar, und AC-5/AC-6 haben keinen E2E-Nachweis. | `gh run view 42514214945 --log-failed` | Testansatz ändern, z. B. Frage an einer eigenen Test-EEG per Admin-API einschalten und danach zurücksetzen, oder Formularabschnitt per Vitest testen und E2E auf den echten Backend-Pfad beschränken. |
| BUG-2 | Medium | Edge Case „Status nach dem Import (aktiviert): Wert bleibt änderbar" ist nicht umgesetzt, der User-Guide behauptet es aber: `docs/user-guide/04-admin-applications.md` „…auch nach Import oder Aktivierung". Admin-Bearbeiten ist nur in `submitted/under_review/needs_info/approved/import_failed` erlaubt (`internal/application/admin_service.go:407-416` → 409; Button in `admin-application-detail.tsx:703-708` ausgeblendet). | Antrag in `activated` öffnen → kein „Bearbeiten"; `PUT /api/admin/applications/{id}` mit `{"multiParticipationRequested":false}` → 409. | Owner-Entscheidung: Edge Case + User-Guide korrigieren (nach Import nicht änderbar, wie die anderen Onboarding-Felder) oder einen schmalen Pfad zum Setzen des Werts nach Import vorsehen. |
| BUG-3 | Low | AC-15 verlangt 400 für einen ungültigen Wert, die externe API liefert 422 `validation_error` (Bestandsverhalten für nicht dekodierbare Bodies; in Implementation Notes und `api-spec.md` 8.1 dokumentiert). | `POST /api/external/v1/applications` mit `"multiParticipationRequested":"ja"` → 422 | AC-15 an 422 anpassen. |

### Security Smoke Test

| Severity | Datei | Funktion | Risiko | Exploit-Szenario | Fix-Empfehlung | Confidence |
|---|---|---|---|---|---|---|
| Info | `internal/http/admin_settings_multi_participation.go` | Get/SaveMultiParticipation | Tenant-Isolation | `parseRCAndCheck` als erste Prüfung (403 für fremde RC, `TestMultiParticipationSettings_ForeignTenantForbidden`); gleiches Muster wie Einleitungstext | keine | High |
| Info | `internal/application/multi_participation.go`, `src/components/intro-text-display.tsx` | Erklärungstext auf öffentlicher Seite | Stored XSS | Server bluemonday (`sanitize.HTML`) bei Save und Config-Import, Client DOMPurify mit fester Allow-List; `openLinksInNewTab` arbeitet auf `<template>` (inert). Überschrift/Frage als React-Text gerendert | keine; im /security-review bestätigen | High |
| Info | `registration_entrypoint_repo.go`, `application_repo.go` | SQL | Injection | alle Queries parametrisiert | keine | High |
| Info | `admin_settings_multi_participation.go` | SaveMultiParticipation | Eingabelängen | Längen nach Bereinigung geprüft (200/300 Zeichen, 50 KiB); Body-Limit über bestehende `MaxBodySize`-Middleware | keine | Medium |
| Info | `internal/application/application_service.go` | Create/Update/Submit | Business-Logik | Server erzwingt Pflicht (außer externe API), verwirft bei ausgeschalteter Frage; keine neuen Statusübergänge | keine | High |
| Info | alle neuen Pfade | Logs | PII | keine neuen Log-Ausgaben mit Personendaten | keine | High |

Keine Critical/High/Medium-Security-Funde.

**Für `/security-review`:** öffentliche Formular-Konfiguration (`GET /api/public/registration/{rc}`, neuer Block nur bei aktiver Frage); Zwei-Schicht-Bereinigung des Erklärungstexts inkl. Config-Import und Link-Umschreibung in `RichTextDisplay`; Tenant-Isolation der zwei neuen Admin-Endpoints (`parseRCAndCheck`, `claims == nil`-Zweig); `OptionalBool`-Dekodierung im Admin-Update; Migration `000095` (rein additiv); externe API (Feld optional).

### Regression

| Bereich | Ergebnis |
|---|---|
| Einleitungstext (PROJ-11) nach Herauslösen des Editors | Pass: `admin-intro-text-editor.tsx` nutzt `useRichTextEditor`/`editorHtml`, Auto-Save/Dirty-Logik äquivalent (`isEmpty ? null : getHTML()` ↔ `editorHtml(e) \|\| null`); `IntroTextDisplay` ohne Link-Umschreibung wie bisher; PROJ-11-E2E in CI grün |
| Admin-Bearbeiten speichern | Pass: fehlender Key lässt Wert stehen, explizites `null` leert; andere Felder unverändert |
| Export-Katalog FE/BE | Pass (gleiche Definition, Tests beidseitig) |
| Config-Export/-Import | Pass; `SaveAllEEGSettingsTx` nur vom Importer genutzt, `PUT /settings/eeg` unberührt; Alt-Bundle schaltet die Frage aus (bewusst, in der Vorschau sichtbar) |
| Entrypoint-Laden | einziger SELECT (`GetByRCNumber`) um die vier Spalten erweitert |

### Production-Ready

**NEIN** — BUG-1 (High, rote E2E) zuerst beheben; BUG-2 per Owner-Entscheidung (Spec/User-Guide oder Code) vor dem Prod-Deploy klären. Danach /qa erneut, dann `/security-review`.

### Nacharbeit nach QA (2026-11-08)

- **BUG-1 behoben:** Die Formular-Tests laufen gegen die eigene Seed-EEG `RC131131` (Frage eingeschaltet, `db/seeds/dev_seed.sql`) statt gegen einen Browser-Mock, der wegen SSR nicht greift. Die gemeinsame Test-EEG `RC123456` bleibt unverändert.
- **BUG-2 behoben (Doku):** Edge Case und User-Guide an das bestehende Bearbeiten-Fenster angeglichen. Nach dem Import ist der Wert lesbar, aber wie alle Antragsfelder nicht mehr änderbar. Kein neuer Bearbeiten-Pfad.
- **Ergebnis:** CI auf 73933a1 komplett grün inkl. E2E (Chromium). Keine offenen Critical/High-Funde → **Approved**. Offen vor dem Merge: `/security-review` und Owner-Freigabe der Migration 000095.
- **BUG-3 behoben (Doku):** AC-15 nennt jetzt 422 `validation_error`, wie die externe API seit jeher antwortet.

## Security Review (2026-11-09)

**Reviewer:** Security Engineer (AI) · **Scope:** PR #127 (Migration 000095, `GET/PUT /api/admin/settings/multi-participation`, öffentliche Formular-Konfiguration, Antragsfeld in öffentlichem/Self-Edit-/Admin-/externem Pfad, Config-Import, `RichTextDisplay`, Seed-EEG `RC131131`).

**Threat Model:** Der Erklärungstext ist von EEG-Admins geschriebenes HTML auf einer öffentlichen Seite (XSS-Risiko). Die neuen Admin-Endpoints könnten fremde EEGs lesen oder ändern (Mandanten-Trennung). Das Antragsfeld könnte über einen Weg die Pflichtregel umgehen.

| Severity | File | Function/Area | Risk | Exploit Scenario | Recommended Fix | Confidence |
|---|---|---|---|---|---|---|
| Low | `internal/sanitize/html.go` | bluemonday-Policy (bestehend) | Erlaubt `target` auf Links, erzwingt aber kein `rel="noopener"` | Gespeicherter Link mit `target="_blank"` ohne `rel` → Reverse-Tabnabbing, falls das HTML einmal ohne Client-Nachbearbeitung gerendert wird | Keine Änderung nötig: `RichTextDisplay` setzt `rel="noopener noreferrer"` auf jeden Link nach DOMPurify. Optional später `RequireNoReferrerOnLinks` in der Policy (betrifft auch Einleitungstext und Admin-Notiz) | High |
| Info | `internal/http/admin.go` | `parseRCAndCheck`, Zweig `claims == nil` | Ohne Claims keine Tenant-Prüfung | Nur erreichbar, wenn die Auth-Middleware fehlt; beide Middlewares (Keycloak, Test-Header) setzen Claims oder lehnen ab. Gleiches Muster wie der Einleitungstext | Keine | High |
| Info | `internal/application/multi_participation.go` | Überschrift/Frage als Klartext | Nicht HTML-bereinigt | Werden nur als React-Text gerendert (escaped), nicht in Mails/PDF | Keine; bei künftiger Nutzung in HTML-Vorlagen über `html/template` escapen | High |
| Info | `db/seeds/dev_seed.sql` | Seed-EEG `RC131131` | Zusätzliche aktive RC-Nummer | Seed läuft nur in CI und lokal, nicht über Helm (Test/Prod) | Keine | High |

**Geprüft ohne Befund:**
- **Mandanten-Trennung:** `parseRCAndCheck` ist die erste Prüfung in GET und PUT. Der Unit-Test deckt 403 bei fremder RC ab.
- **SQL und Eingaben:** alles parametrisiert. Längen, Pflichtfrage und Bereinigung prüft der Server über `NormalizeMultiParticipationSettings`, gemeinsam mit dem Config-Import; `Apply` ruft vorher `validateAndSanitize` auf.
- **XSS, zweistufig:** Server bluemonday, Client DOMPurify mit derselben Allow-List. Das Link-Rewrite arbeitet auf einem inerten `<template>` nach der Bereinigung.
- **Öffentliche Konfiguration:** Der Block wird nur bei aktiver Frage ausgeliefert. Rate-Limits der öffentlichen Routen bleiben unverändert.
- **Externe API:** eigener Einreichweg (`SubmitExternalApplication`), den das öffentliche Formular nicht wählen kann. Bei ausgeschalteter Frage wird der Wert verworfen.
- **`OptionalBool`:** ein falscher Typ führt zum JSON-Fehler und 400. Ohne den Schlüssel bleibt der Wert unverändert.
- **Logging:** keine neuen Logs mit personenbezogenen Daten. Fehlerantworten nutzen das bestehende Fehlermodell.
- **Migration 000095:** fügt nur hinzu und ist umkehrbar; der Down-Pfad löscht nur die fünf neuen Spalten.
- **Core-Import:** unverändert.

**Scans:**
- govulncheck: 6 bekannte Advisories in `excelize/v2@v2.11.0`, schon auf main, kein Release mit Fix; der PR ändert `go.mod` nicht.
- gosec (application, http, configexport, shared): 0 Issues.
- npm audit `--audit-level=high`: keine High/Critical.
- Semgrep/Trivy: in der PR-CI grün.

**Verdict: APPROVED** — keine Critical/High/Medium-Funde.

## Deployment

- Live in Prod seit 2026-11-10 (helm Revision 30, Image sha-70534b7, Tag v1.51.0-PROJ-58-131-132). Migration 000095 angewendet. Test-Hinweise im User-Guide entfernt.
