# Integrarsi con il catalogo reels-kb

Guida per applicazioni o agenti AI che vogliono leggere e sincronizzarsi con il catalogo dei Reels di `@themattvision`.

Il catalogo è un repository GitHub pubblico: **`lochiameroIutah/reels-kb`**.
Non serve nessun token o autenticazione.

---

## Struttura del catalogo

```
reels-kb/
├── index.json          ← indice leggero di tutti i reel
└── reels/
    ├── 2025-03-15_10-00_tutorial_ai_titolo-breve_DVt89QWjTKs.txt
    ├── 2025-03-12_18-30_news_ai_altro-titolo_DVrZIS8jU_O.txt
    └── ...
```

- **`index.json`** — array con i metadati essenziali di ogni reel (niente trascrizioni)
- **`reels/*.txt`** — file completo per ogni reel (include trascrizione audio, hook, caption)

---

## Come sincronizzarsi una volta al giorno

Il metodo più semplice e affidabile è clonare il repo con `git` (shallow clone, veloce):

```bash
# Prima volta
git clone --depth 1 https://github.com/lochiameroIutah/reels-kb.git /path/to/reels-kb

# Aggiornamenti successivi (ogni giorno)
cd /path/to/reels-kb && git pull
```

Poi leggi `index.json` per avere il catalogo aggiornato:

```python
import json

with open('/path/to/reels-kb/index.json') as f:
    reels = json.load(f)

# reels è una lista di oggetti — vedi struttura sotto
print(f"Totale reel: {len(reels)}")
```

---

## Struttura di `index.json`

Ogni entry nell'array ha questi campi:

```json
{
  "shortcode": "DVt89QWjTKs",
  "permalink": "https://www.instagram.com/reel/DVt89QWjTKs/",
  "date": "2025-03-15",
  "category": "tutorial_ai",
  "topic": "Come usare Claude Projects per organizzare il lavoro",
  "hook_text": "Claude ora ricorda tutto",
  "like_count": 1243,
  "comments_count": 87,
  "duration": 62,
  "tags": ["claude", "ai", "produttività"]
}
```

| Campo | Tipo | Descrizione |
|---|---|---|
| `shortcode` | string | ID univoco del reel (anche nome file) |
| `permalink` | string | URL Instagram |
| `date` | string | Data pubblicazione `YYYY-MM-DD` |
| `category` | string | Una delle 8 categorie (vedi sotto) |
| `topic` | string | Riassunto argomento (1 frase) |
| `hook_text` | string | Testo sovrapposto al video (estratto da vision AI) |
| `like_count` | number | Like al momento dell'ultimo aggiornamento |
| `comments_count` | number | Commenti al momento dell'ultimo aggiornamento |
| `duration` | number | Durata in secondi |
| `tags` | array | Tag semantici assegnati automaticamente |

---

## Leggere un reel completo

Per ogni reel puoi leggere il file `.txt` completo. Il nome file segue il pattern:

```
YYYY-MM-DD_HH-mm_CATEGORIA_titolo-breve_SHORTCODE.txt
```

Esempio per trovare e leggere un reel specifico:

```python
import os, glob

shortcode = "DVt89QWjTKs"
files = glob.glob(f'/path/to/reels-kb/reels/*{shortcode}*.txt')

if files:
    with open(files[0]) as f:
        content = f.read()
    print(content)
```

Il file `.txt` contiene:

```
SHORTCODE: DVt89QWjTKs
PERMALINK: https://www.instagram.com/reel/DVt89QWjTKs/
DATA: 2025-03-15
DURATA: 62s
LIKE: 1243
COMMENTI: 87
CATEGORIA: tutorial_ai
TOPIC: Come usare Claude Projects per organizzare il lavoro
TAGS: claude, ai, produttività
CTA_COMMENTO: sì
CTA_FOLLOW: no
CTA_LINK: no
HOOK_TESTO: Claude ora ricorda tutto
DESCRIZIONE_VISIVA: Schermata di Claude con Projects aperto, testo in sovrimpressione
CAPTION:
[caption completa pubblicata su Instagram]
TRASCRIZIONE:
[trascrizione audio completa del reel]
```

---

## Categorie disponibili

| Categoria | Contenuto |
|---|---|
| `tutorial_ai` | Guide step-by-step su tool AI |
| `confronto_ai` | Confronti tra modelli o tool |
| `risparmio_tech` | Risparmiare tempo/soldi con la tecnologia |
| `news_ai` | Notizie AI |
| `tool_gratuiti` | Tool gratuiti o open source |
| `automazione` | Workflow e produttività |
| `personale` | Contenuto personale non-tech |
| `altro` | Altro |

---

## Aggiornamento automatico del catalogo

Il catalogo viene aggiornato automaticamente:
- **Ogni ora** — Make.com manda tutti i post recenti a Vercel
- **Post nuovi** → pipeline completa (download video, trascrizione Whisper, vision AI, categorizzazione)
- **Post esistenti** → solo aggiornamento metriche (like, commenti, visualizzazioni)

Non serve fare nulla: ogni `git pull` porta l'ultima versione.

---

## Esempio: sincronizzazione giornaliera completa

```python
import subprocess
import json
from pathlib import Path

KB_PATH = Path("/tmp/reels-kb")

def sync_catalog():
    if KB_PATH.exists():
        subprocess.run(["git", "pull"], cwd=KB_PATH, check=True)
    else:
        subprocess.run([
            "git", "clone", "--depth", "1",
            "https://github.com/lochiameroIutah/reels-kb.git",
            str(KB_PATH)
        ], check=True)

def load_index():
    with open(KB_PATH / "index.json") as f:
        return json.load(f)

def load_reel(shortcode: str) -> str | None:
    matches = list((KB_PATH / "reels").glob(f"*{shortcode}*.txt"))
    if not matches:
        return None
    return matches[0].read_text()

# Uso
sync_catalog()
reels = load_index()
print(f"Catalogo aggiornato: {len(reels)} reel")

# Top 5 per engagement
top = sorted(reels, key=lambda r: r["like_count"] + r["comments_count"], reverse=True)[:5]
for r in top:
    print(f"{r['date']} | {r['category']} | {r['like_count']} like | {r['topic']}")
```
