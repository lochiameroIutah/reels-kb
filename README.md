# reels-kb

Knowledge base automatica dei Reel pubblicati da [@themattvision](https://www.instagram.com/themattvision/).

Ogni Reel viene processato automaticamente dal backend e salvato come file `.txt` nella cartella `reels/`.

---

## Struttura di ogni file .txt

```
POST ID / IG ID / SHORTCODE / PERMALINK / DATA / ORA

--- METRICHE ---
LIKE / COMMENTI / DURATA VIDEO / CARATTERI CAPTION / HASHTAG

--- FLAGS ---
CTA COMMENTO / CTA FOLLOW / LINK IN CAPTION

--- CATEGORIA ---
CATEGORIA / TAG / TOPIC

--- HOOK TESTUALE (dal frame) ---
[testo sovrapposto nel video]

--- DESCRIZIONE VISIVA ---
[descrizione del frame estratto]

--- CAPTION COMPLETA ---
[caption pubblicata]

--- TRASCRIZIONE AUDIO ---
[trascrizione completa del parlato]
```

---

## Come usare la knowledge base con Claude Code

### 1. Clona o scarica la SKILL.md

Scarica la skill dal repo del backend:

```bash
mkdir -p ~/.claude/skills/scrittura-reels
curl -o ~/.claude/skills/scrittura-reels/SKILL.md \
  https://raw.githubusercontent.com/lochiameroIutah/themattvision-cataloger/main/SKILL.md
```

### 2. Aggiungi il repo come contesto in Claude Code

Apri Claude Code e aggiungi questo repo come fonte di contesto nel tuo Progetto Claude, oppure usa `/add-dir` per includere la cartella `reels/` scaricata localmente.

### 3. Usa la skill

In qualsiasi sessione Claude Code, scrivi:

```
/scrittura-reels
```

Claude leggerà i file `.txt` catalogati e ti aiuterà a:
- Scrivere nuovi Reel nel tuo stile
- Analizzare quali contenuti performano meglio
- Generare hook, caption e scalette coerenti con i tuoi post passati

---

## Backend

Il backend che popola automaticamente questa repo è in:
👉 [`lochiameroIutah/themattvision-cataloger`](https://github.com/lochiameroIutah/themattvision-cataloger)

Pipeline: **Make.com** → **Vercel** (download + ffmpeg + Groq Whisper + vision + categorizzazione) → **questo repo**
