# Setup: Open-Data-Veröffentlichung nach GitHub

## Teil A — GitHub-Repo anlegen (einmalig)

1. Auf GitHub ein **öffentliches** Repo `coerde-events` anlegen.
2. Den Inhalt des Ordners `coerde-events/` (README, LICENSE, LICENSE-CODE,
   .gitignore, data/, docs/) hochladen — entweder per Web-Upload oder lokal:
   ```bash
   cd coerde-events
   git init
   git add .
   git commit -m "init: Open-Data-Repo Coerde Events"
   git branch -M main
   git remote add origin https://github.com/<USER>/coerde-events.git
   git push -u origin main
   ```
3. In README, LICENSE und docs/schema.json `<USER>` bzw. `USER` durch deinen
   GitHub-Namen ersetzen.

## Teil B — GitHub Personal Access Token

1. GitHub → Settings → Developer settings → **Fine-grained tokens** → neues Token.
2. Repository access: nur `coerde-events`.
3. Permissions: **Contents → Read and write** (mehr nicht).
4. Token kopieren.
5. In n8n unter **Credentials** ein neues **"Header Auth"**-Credential anlegen:
   - Name: `Authorization`
   - Value: `Bearer <DEIN_TOKEN>`

   (Damit liegt das Token NUR in n8n, niemals im Code oder im Repo.)

## Teil C — n8n-Nodes verdrahten

Reihenfolge am Ende deines bestehenden Workflows:

```
... Normalize / Final Cleanup
        │
        ▼
[Build GitHub OpenData Payloads]   (Function-Node, der gelieferte Code)
        │   gibt 4 Items aus (json/csv/ics/geojson)
        ▼
[GitHub: get current SHA]          (HTTP Request, GET)  — optional, s.u.
        │
        ▼
[GitHub: PUT file]                 (HTTP Request, PUT)
```

### Node "GitHub: PUT file" (HTTP Request)

- **Method:** `PUT`
- **URL:**
  ```
  https://api.github.com/repos/<USER>/coerde-events/contents/{{ $json.repo_path }}
  ```
- **Authentication:** Generic Credential → das Header-Auth-Credential aus Teil B
- **Headers (zusätzlich):**
  - `Accept: application/vnd.github+json`
  - `User-Agent: coerde-events-bot`  (GitHub verlangt einen User-Agent)
- **Body (JSON):**
  ```json
  {
    "message": "={{ $json.commit_message }}",
    "content": "={{ $json.content_b64 }}",
    "branch":  "={{ $json.branch }}",
    "sha":     "={{ $json.existing_sha }}"
  }
  ```

### Wichtig: der `sha` beim Überschreiben

Die GitHub Contents API verlangt beim **Aktualisieren** einer bereits
existierenden Datei deren aktuellen Blob-`sha`. Zwei Wege:

**Variante 1 (robust, empfohlen):** Vor dem PUT ein GET auf dieselbe URL:
```
GET https://api.github.com/repos/<USER>/coerde-events/contents/{{ $json.repo_path }}?ref=main
```
Den Wert `sha` aus der Antwort in das Feld `existing_sha` mappen (z.B. per Set-Node).
Schlägt der GET mit 404 fehl (Datei existiert noch nicht), `existing_sha` leer lassen
und das `sha`-Feld im PUT-Body weglassen — dann legt GitHub die Datei neu an.
Im HTTP-Request-Node dafür **"Continue On Fail"** für den GET aktivieren.

**Variante 2 (einfacher):** Den nativen **GitHub-Node** von n8n verwenden
(Operation: *File → Edit*). Der holt den SHA automatisch, kann aber nur eine
Datei pro Aufruf — du bräuchtest dann vier GitHub-Nodes statt einem
HTTP-Request-Node im Loop. Da der Function-Node hier vier Items ausgibt, ist
der HTTP-Weg mit Loop kompakter.

## Teil D — GitHub Pages (optional, für vorzeigbare URL im Antrag)

1. Repo → Settings → Pages → Source: `main`, Ordner `/docs` (oder `/root`).
2. Eine kleine `docs/index.html` ergänzen, die `../data/events.json` lädt und
   als Liste/Kalender anzeigt. Dann ist der Datensatz unter
   `https://<USER>.github.io/coerde-events/` ohne Login sichtbar.

## Teil E — Förderantrag / DIGIFARM.MS

- Datensatz zusätzlich bei **DIGIFARM.MS** (Plattform der Stadt Münster für
  Open-Source-Bürgerprojekte) einreichen und auf das GitHub-Repo verweisen.
- Im Antrag die **Übertragbarkeit** betonen: Quell-Liste austauschbar →
  Modell für jeden Stadtteil/jede Kommune.
- Optional: Registrierung des Datensatzes bei **GovData.de** /
  **Open-Data-Portal Münster** für maximale Sichtbarkeit als kommunales Open Data.

## Sicherheits-Checkliste vor dem ersten Push

- [ ] `.gitignore` ist aktiv (schützt .env, *.token, credentials*.json)
- [ ] Kein API-Key/Token in committeten Dateien (`git grep -i "bearer\|api_key\|token"`)
- [ ] GitHub-Token ist fine-grained und nur auf dieses eine Repo beschränkt
- [ ] Repo ist als Datensatz unter CC BY 4.0 lizenziert, Code unter MIT
