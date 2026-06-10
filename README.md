# Coerde Events – Offener Veranstaltungskalender für Münster-Coerde

Ein automatisch gepflegter, offener Datensatz öffentlicher Veranstaltungen im
Münsteraner Stadtteil **Coerde**. Die Daten werden aus rund 19 lokalen Quellen
(Vereine, Kirchengemeinden, Bildungseinrichtungen, Stadtteilinitiativen)
zusammengeführt, normalisiert und hier als **Open Data** bereitgestellt – ohne
Zugangsdaten, frei nutzbar, maschinenlesbar.

## Was ist das?

Veranstaltungsinformationen in Coerde sind über viele kleine Websites verstreut
und für Bürgerinnen und Bürger schwer auffindbar. Dieses Projekt aggregiert sie
an einer Stelle und stellt sie in offenen Formaten zur Verfügung – als
Datengrundlage für Kalender-Apps, Stadtteilportale, Karten und Weiterverwendung
durch andere Projekte.

Das Projekt ist als **Modellprojekt** angelegt: Der Ansatz ist auf jeden anderen
Stadtteil und jede andere Kommune übertragbar. Quell-Liste austauschen, fertig.

## Verfügbare Daten

Alle Dateien liegen im Ordner [`data/`](./data) und werden automatisch
aktualisiert.

| Datei | Format | Zweck |
|-------|--------|-------|
| [`data/events.json`](./data/events.json) | JSON | Maschinenlesbar, für Anwendungen und APIs |
| [`data/events.csv`](./data/events.csv) | CSV | Tabellenkalkulation, einfache Weiterverarbeitung |
| [`data/events.ics`](./data/events.ics) | iCalendar | Direkt in Kalender-Apps abonnierbar |
| [`data/events.geojson`](./data/events.geojson) | GeoJSON | Darstellung auf Karten (sofern Koordinaten vorhanden) |

### Kalender abonnieren

Die `events.ics` lässt sich direkt in Google Calendar, Apple Kalender, Outlook
oder Thunderbird abonnieren. Abo-URL (nach Veröffentlichung über GitHub anpassen):

```
https://raw.githubusercontent.com/<Coerdeevents>/coerde-events/main/data/events.ics
```

## Datenschema

Jedes Event in `events.json` folgt diesem Schema:

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `event_id` | string | Stabile, eindeutige ID des Termins |
| `title` | string | Titel der Veranstaltung |
| `start_date` | string (`YYYY-MM-DD`) | Startdatum |
| `start_time` | string (`HH:MM`) \| null | Startuhrzeit, falls bekannt |
| `end_date` | string (`YYYY-MM-DD`) \| null | Enddatum, nur bei mehrtägigen Terminen |
| `end_time` | string (`HH:MM`) \| null | Enduhrzeit, falls bekannt |
| `location_name` | string | Veranstaltungsort (Name) |
| `address` | string | Adresse |
| `city` | string | Ort (i.d.R. Münster / Münster-Coerde) |
| `organizer` | string | Veranstalter |
| `category` | string | Themen-/Kategorie-Zuordnung |
| `short_description` | string | Neutrale Kurzbeschreibung (max. 300 Zeichen) |
| `status` | string | `scheduled`, `cancelled`, `postponed`, `unknown` |
| `date_precision` | string | `day` (tagesgenau), `month` (nur Monat bekannt), `unknown` |
| `source_name` | string | Name der Originalquelle |
| `source_url` | string | URL der Originalquelle |
| `last_updated` | string (ISO 8601) | Zeitpunkt der letzten Aktualisierung |

Termine mit `date_precision` ungleich `day` oder `status` ungleich `scheduled`
sind als unsicher gekennzeichnet und sollten von Anwendungen entsprechend
behandelt werden.

## Aktualisierung

Die Daten werden automatisiert über einen [n8n](https://n8n.io)-Workflow gepflegt,
der die Quellen regelmäßig abruft, normalisiert und in dieses Repository schreibt.
Das Feld `generated_at` in `data/events.json` nennt den Stand der letzten
Aktualisierung.

## Quellen

Die Veranstaltungsdaten stammen aus öffentlich zugänglichen Websites lokaler
Akteure in Münster-Coerde. Die jeweilige Originalquelle ist pro Event über
`source_name` und `source_url` ausgewiesen. Die Rechte an den ursprünglichen
Inhalten verbleiben bei den jeweiligen Veranstaltern; dieser Datensatz stellt
eine strukturierte, faktische Zusammenstellung von Termindaten dar.

## Lizenz

Die in diesem Repository bereitgestellten **Daten** stehen unter
[Creative Commons Namensnennung 4.0 International (CC BY 4.0)](./LICENSE).
Du darfst sie frei nutzen, teilen und weiterverarbeiten, solange du die Quelle
nennst. Siehe [`LICENSE`](./LICENSE).

Der **Programmcode** (sofern in diesem Repository enthalten) steht unter der
MIT-Lizenz, siehe [`LICENSE-CODE`](./LICENSE-CODE).

## Mitmachen / Kontakt

Fehlende Quelle, falscher Termin, Idee zur Weiternutzung? Bitte ein
[Issue](../../issues) eröffnen.

---

*Dieses Projekt ist ein ehrenamtlicher Beitrag zur digitalen Stadtteilarbeit in
Münster-Coerde und versteht sich als übertragbares Open-Source-Modell im Sinne
der Smart-City-Strategie der Stadt Münster.*
