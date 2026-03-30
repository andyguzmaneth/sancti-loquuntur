# Sancti Loquuntur — Architecture & Build Plan

*Last updated: 2026-03-30*

## Overview

Free, open-source AI trained on the authentic writings of Catholic saints. RAG-first approach, graduate to fine-tuned model once validated.

## Phase 1: Corpus (1-2 weekends)

### Sources

| Source | Saints | Format | Status |
|--------|--------|--------|--------|
| New Advent (newadvent.org) | Augustine, Aquinas (full Summa), Bernard, Church Fathers | HTML, public domain | Not started |
| CCEL (ccel.org) | Teresa, John of the Cross, Ignatius, Francis de Sales | HTML/plain text | Not started |
| Internet Archive | Joan of Arc trial transcripts (Quicherat edition) | Scanned PDF + OCR | Not started |
| Project Gutenberg | Therese (Story of a Soul), Catherine (Dialogue, Letters) | Plain text | Not started |
| Documenta Catholica Omnia | Latin originals + some translations | PDF/HTML | Not started |

### Pipeline

1. Scrape HTML with Python (requests + BeautifulSoup), one script per source
2. Strip footnotes, chapter headers, translator notes, page numbers
3. Tag each chunk with `{saint, work, chapter, era}`
4. Output clean JSONL: one line per passage (~500-1000 words each)
5. Estimated total: **8-12M words** clean

### Risk

OCR quality on Joan's trial transcripts and older scans. May need manual cleanup on ~50 pages.

## Phase 2: Architecture

### Path A: RAG + Persona Prompts (ship in days) — START HERE

- Vector DB (Chroma or Pinecone) indexes all passages
- Base LLM (Claude or Llama 3) with saint-specific system prompts
- Each query retrieves relevant passages, LLM responds in character
- **Pro:** Fast, cheap, always citable, can use the best base models
- **Con:** Voice is approximated, not learned. Modern cadence bleeds through.

### Path B: LoRA Fine-tune (ship in 1-2 weeks) — GRADUATE TO THIS

- Fine-tune Llama 3 8B or Mistral 7B with LoRA on the full corpus
- Train separate LoRA adapters per saint (or one merged)
- RAG layer on top for citation accuracy
- **Pro:** Authentic voice, the actual "time machine" effect
- **Con:** More compute, needs eval, harder to iterate

### Decision

Start with Path A. Validate that people use it. Then invest in fine-tuning.

## Phase 3: Build (1 week for Path A)

### Project Structure

```
sancti-loquuntur/
  index.html              # Landing page (DONE)
  images/                 # Saint illustrations (IN PROGRESS)
  app/
    api/
      chat/route.ts       # Edge function: question + saint selection
    components/
      ChatInterface.tsx    # Conversation UI
      SaintSelector.tsx    # Pick a saint or "Council" mode
      LectioDivina.tsx     # Interactive Lectio flow
    lib/
      saints.ts            # Saint configs (system prompts, metadata)
      rag.ts               # Vector search against corpus
      prompts.ts           # Prompt templates per mode
  corpus/
    scripts/
      scrape_newadvent.py
      scrape_ccel.py
      scrape_gutenberg.py
      clean_and_chunk.py
      embed_and_index.py
    data/
      raw/                 # Raw scraped HTML/text
      clean/               # Cleaned JSONL per saint
      embeddings/          # Vector embeddings
```

### Stack

| Layer | Choice | Why |
|-------|--------|-----|
| Frontend | Next.js (or plain HTML + vanilla JS) | Already on Vercel |
| LLM (Path A) | Claude API | Best quality for persona work |
| LLM (Path B) | Llama 3 8B via Together/Replicate | Self-hostable, open source |
| Vector DB | Chroma (local/free) or Pinecone (hosted, free tier) | Zero cost to start |
| Embeddings | Voyage or OpenAI ada-002 | Cheap, good quality |
| Deploy | Vercel (already connected) | Auto-deploy from GitHub |

### Modes

1. **Council of Saints** — question goes to 3-4 saints, each responds in character
2. **One-on-One** — single saint, deep conversation
3. **Lectio Divina** — guided 4-step scripture meditation
4. **Ignatian Discernment** — structured decision-making flow
5. **The Examen** — end-of-day review of consciousness

## Phase 4: Cost

| Resource | Free tier? | At scale |
|----------|-----------|----------|
| Vercel hosting | Yes | Free |
| Claude API (Path A) | Pay per token | ~$0.01-0.05/conversation |
| Chroma (self-hosted) | Free | Free |
| Together AI (Path B) | Trial credits | ~$0.20/M tokens |
| LoRA training (Path B) | -- | ~$50-100 one-time on RunPod |
| Domain name | -- | ~$12/year |

### Keeping it free for users

- Eat API costs if traffic is modest (likely < $50/mo early on)
- Or run self-hosted Llama on cheap GPU (~$50/mo on Lambda/Vast.ai)
- Or apply for API credits from Anthropic/Meta given open-source/educational nature

## Phase 5: Eval & Launch

### Voice evaluation

Get 2-3 people who've actually read these saints to judge "does this sound like Augustine?"

### Theological guardrails

System prompt must include: "I am an AI drawing from the writings of [Saint] — I am not the saint, and this is not spiritual direction from the Church."

### Launch channels

- Catholic Twitter/Reddit (large, engaged community)
- Hacker News (the "time machine" angle is compelling)
- Product Hunt
- Catholic media outlets, podcasts

### Open source everything

- Corpus + scraping scripts
- System prompts
- Fine-tune configs and LoRA adapters
- The model itself (if fine-tuned)
- Training methodology documentation

## Saint Roster (v1)

| Saint | Era | Est. Corpus | Voice Profile |
|-------|-----|-------------|---------------|
| Augustine of Hippo | 354-430 | ~5M words | Confessional, searching, passionate |
| Thomas Aquinas | 1225-1274 | ~1.8M words (Summa alone) | Systematic, precise, exhaustive |
| Bernard of Clairvaux | 1090-1153 | ~500K+ words | Mystical, fiery, poetic |
| Joan of Arc | 1412-1431 | ~50K words (trial transcripts) | Blunt, defiant, prophetic |
| Teresa of Avila | 1515-1582 | Large | Practical, warm, mystical |
| John of the Cross | 1542-1591 | Medium | Poetic, dark, luminous |
| Catherine of Siena | 1347-1380 | 400+ letters + Dialogue | Fearless, political, tender |
| Ignatius of Loyola | 1491-1556 | Exercises + letters | Systematic, discerning |
| Francis de Sales | 1567-1622 | Medium | Gentle, accessible, warm |
| Therese of Lisieux | 1873-1897 | Medium | Simple, intimate, modern |

## Current Status

- [x] Landing page live (sancti-loquuntur.vercel.app)
- [x] GitHub repo (github.com/andyguzmaneth/sancti-loquuntur)
- [x] Vercel auto-deploy connected
- [x] Saint illustrations: Augustine hero, Francis de Sales, Therese, fresco bg
- [x] Midjourney prompts for remaining saints
- [x] Vigils dark mode, Lectio Divina, silence timer, typewriter effects
- [ ] Corpus scraping scripts
- [ ] Clean + chunk pipeline
- [ ] Vector embeddings
- [ ] System prompts per saint
- [ ] Chat API endpoint
- [ ] Chat UI
- [ ] LoRA fine-tune (Phase B)
