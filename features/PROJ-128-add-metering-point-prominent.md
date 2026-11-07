# PROJ-128: „Zählpunkt hinzufügen" im Registrierungsformular hervorheben

## Status: In Progress
**Created:** 2026-11-07
**Last Updated:** 2026-11-07

## Dependencies
- Berührt: PROJ-1 (Public Registration), PROJ-106 (Formular-Split)

## Hintergrund

Tester-Rückmeldung 2026-11-06 (WhatsApp-Gruppe): Mitglieder tun sich schwer, mehrere Zählpunkte
einzutragen (z. B. Bezug + PV-Einspeisung). Der Button „Zählpunkt hinzufügen" war klein und nur
umrandet (`variant="outline" size="sm"`) und wurde übersehen. Owner-Entscheidung 2026-11-07: Button
größer und farbig, dazu ein kurzer Hinweis beim ersten Zählpunkt.

## User Stories

- Als **Mitglied** mit mehreren Zählpunkten möchte ich sofort sehen, dass ich weitere Zählpunkte
  hinzufügen kann, damit ich nicht nur den ersten eintrage.
- Als **EEG-Admin** möchte ich weniger Rückfragen wegen fehlender Zählpunkte.

## Acceptance Criteria

- [x] AC-1: Der Button „Zählpunkt hinzufügen" ist ein primärer (farbiger) Button in Standardgröße
  (statt klein und nur umrandet). Bewusst nicht volle Breite, damit er sich mobil vom Button
  „Antrag einreichen" unterscheidet.
- [x] AC-2: Solange genau ein Zählpunkt erfasst ist, steht oberhalb davon der Hinweis
  „Weitere Zählpunkte? Unten hinzufügen." mit Info-Symbol.
- [x] AC-3: Ab zwei Zählpunkten entfällt der Hinweis; bei 10 Zählpunkten entfällt wie bisher der Button.
- [x] AC-4: Keine Änderung an Validierung, Payload oder Backend.

## Edge Cases

- Custom-Brand-Theme (PROJ-103): Der Button nutzt die Primärfarbe des Themes — Kontrast liefert der
  Theme-Gate wie bei allen primären Buttons.
- Mitglied entfernt den zweiten Zählpunkt wieder → Hinweis erscheint wieder.

## Out of Scope

- Umbau des Zählpunkt-Blocks (Karten pro Zählpunkt, Assistent).
- Admin-Bearbeitungsformular.

## Implementation Notes (2026-11-07)

Nur `src/components/metering-point-fields.tsx`: Hinweiszeile (`text-muted-foreground`, `Info`-Icon,
nur bei `fields.length === 1`) oberhalb des ersten Zählpunkts (Owner-Vorgabe „beim ersten
Zählpunkt") und Button `variant="default"` statt `outline size="sm"`.

Review-Fixes: Icon-Vergrößerung `h-5 w-5` wirkte nicht (Button-Basis `[&_svg]:size-4` gewinnt) und
`mr-2` doppelte den `gap-2` der Basis → beides entfernt; `w-full` auf Mobil entfernt (sah aus wie
„Antrag einreichen"). Bewusst gelassen: Hinweis steht oben beim ersten Zählpunkt, nicht direkt über
dem Button (Owner-Vorgabe) — bei vielen Zusatzfeldern ist der Button weiter unten.

Follow-ups: Screenshots im User-Guide (`register-form-metering-points.png`) neu generieren
(Generator braucht Backend); kein automatisierter Test für AC-2/AC-3.

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
