# n8n-Workflows

Die drei Workflows, die den Coerde-Kalender und die Nachrichten der
Stadtteilseite [coerde.de](https://coerde.de) automatisch pflegen. Sie sind
Exporte aus einer selbst gehosteten [n8n](https://n8n.io)-Instanz und stehen
unter der MIT-Lizenz (siehe [`LICENSE-CODE`](../LICENSE-CODE)).

Die Exporte sind bereinigt: Sie enthalten keine Zugangsdaten, keine
Instanz-Kennungen und keine privaten Mailadressen. Nach dem Import sind sie
inaktiv und laufen erst, wenn die Platzhalter ersetzt und die Zugänge
hinterlegt sind.

## Die Workflows

| Datei | Aufgabe | Zeitplan im Export |
|-------|---------|--------------------|
| [`1-coerde-termine-ermitteln.json`](./1-coerde-termine-ermitteln.json) | Ruft alle Terminquellen ab, vereinheitlicht die Termine, schreibt sie in das Tabellenblatt `events_review` und veröffentlicht JSON, CSV, ICS und GeoJSON in diesem Repository | Mo–Fr 20:00 Uhr |
| [`2-coerde-termine-pruefen.json`](./2-coerde-termine-pruefen.json) | Filtert vergangene und gesperrte Termine, entfernt Dubletten, kennzeichnet Serientermine und schreibt das Ergebnis in das Tabellenblatt `events_current`, aus dem der Kalender liest | Mo–Fr 20:15 Uhr |
| [`4-coerde-nachrichten-abrufen.json`](./4-coerde-nachrichten-abrufen.json) | Sucht in Nachrichtenquellen und im Ratsinformationssystem nach Meldungen mit Coerde-Bezug, lässt sie per Sprachmodell bewerten und übergibt sie an die Redaktion der Stadtteilseite. Veröffentlicht wird dort erst nach Freigabe | täglich 00:10, 06:10, 12:10 und 18:10 Uhr |

Ein früherer dritter Workflow (Veröffentlichung auf einer WordPress-Seite) wird
nicht mehr verwendet und ist deshalb nicht enthalten.

### Wie Workflow 1 die Quellen liest

- **Offizielle Schnittstelle:** Der Veranstaltungskalender der Stadt Münster
  wird über dessen GraphQL-Schnittstelle abgefragt (Node „Stadt Münster API
  abrufen (alle Seiten)“).
- **Feste Parser:** Die Kurssuche der FamilienBildung, der Wochenplan der AWO
  und Seiten mit „The Events Calendar“ werden regelbasiert ausgelesen, ohne
  Sprachmodell.
- **Sprachmodell nur dort, wo Struktur fehlt:** Die übrigen Webseiten werden
  mit Firecrawl in Text umgewandelt und per OpenAI in das einheitliche
  Terminformat überführt. Die Quellen stehen im Node „URL-Quellen laden“.
- **Manuelle Termine:** Termine, die in keiner abrufbaren Quelle stehen, pflegt
  der Node „Manuelle termine ergaenzen“.
- **Überwachung:** „Quellen-Gesundheit prüfen“ vergleicht jeden Lauf mit den
  letzten veröffentlichten Ständen und meldet per Mail, wenn eine Quelle
  ausfällt oder einbricht.

## Einrichten

1. In n8n über **Workflows → Import from File** die gewünschte Datei laden.
2. Die Zugänge anlegen und in den Nodes auswählen (siehe unten). Im Export
   steht an jeder Stelle die Kennung `REPLACE_WITH_YOUR_CREDENTIAL_ID`.
3. Die Platzhalter ersetzen (siehe unten).
4. Bei Bedarf unter **Settings → Error Workflow** einen eigenen
   Fehler-Workflow eintragen. Der im Betrieb genutzte ist nicht enthalten.
5. Den Workflow einmal von Hand ausführen, das Ergebnis prüfen und erst dann
   aktivieren.

### Benötigte Zugänge

| Name im Export | Typ in n8n | Wofür | Workflow |
|----------------|------------|-------|----------|
| `Google Sheets account`, `Google Sheets account 2` | Google Sheets OAuth2 | Tabellenblätter `events_review` und `events_current` | 1, 2 |
| `Bearer Auth Firecrawl` | Bearer Auth | Firecrawl-API | 1 |
| `Bearer Auth OpenAI` | Bearer Auth | OpenAI-API (Terminextraktion) | 1 |
| `Bearer Auth Github` | Bearer Auth | GitHub Contents API; fein abgestuftes Token nur für dieses Repository, Berechtigung „Contents: Read and write“ | 1 |
| `SMTP account` | SMTP | Warn- und Redaktionsmails | 1, 4 |
| `OpenAI account` | OpenAI | Bewertung und Kurzfassungen | 4 |
| `Coerde Lovable Token` | Header Auth (`Authorization: Bearer …`) | Schnittstelle der Stadtteilseite und eingehender Webhook | 4 |

### Platzhalter

| Platzhalter | Wo | Ersetzen durch |
|-------------|----|----------------|
| `YOUR_GOOGLE_SHEET_ID` | alle Google-Sheets-Nodes in Workflow 1 und 2 | ID der eigenen Tabelle. Am einfachsten die Tabelle und das Blatt im Node neu auswählen |
| `absender@example.org`, `empfaenger@example.org` | Node „Warnmail Quellen“ (Workflow 1) | eigene Absender- und Empfängeradresse |
| `absender@example.org`, `redaktion@example.org` | Node „Lauf vorbereiten“ (Workflow 4) | eigene Absender- und Empfängeradresse |
| `Coerdeevents/coerde-events` | GitHub-Nodes und „Quellen-Gesundheit prüfen“ (Workflow 1) | eigenes Repository |
| `https://coerde.de` | Node „Lauf vorbereiten“ (Workflow 4) | Adresse der eigenen Seite |

## Hinweise zur Übertragung auf einen anderen Stadtteil

- Die Quellenliste steht in Workflow 1 im Node „URL-Quellen laden“. Die Nodes
  „Termine normalisieren“ und „Letzte Korrekturen“ führen je eine eigene Liste
  mit Standardwerten pro Quelle (`SOURCE_DEFAULTS`). Neue Quellen gehören in
  alle drei.
- Die Ortsfilter (welche Adressen zum Stadtteil gehören) stehen in den Nodes
  „Kurssuche FaBi Coerde“ und „Stadt Münster Events vorbereiten“.
- Workflow 4 erwartet auf der Website eine Gegenstelle mit den Endpunkten
  `/api/public/n8n/quellen`, `/bekannt`, `/import` und `/kurzfassungen`. Der
  Code der Website ist nicht Teil dieses Repositories.
- Die Code-Nodes laufen im Task-Runner von n8n. Dort gibt es weder die
  `URL`-Klasse noch `require()`; HTTP-Abrufe laufen über
  `this.helpers.httpRequest`.
- Die Serienerkennung ist in Workflow 1 („OpenData Serienfelder ergänzen“) und
  Workflow 2 („Serientermine kennzeichnen“) bewusst identisch. Änderungen
  immer in beiden Nodes vornehmen.
