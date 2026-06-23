# MLVault

> Query 150+ ML research papers from inside your editor. Get cited, equation-aware answers in seconds.

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![Python](https://img.shields.io/badge/python-3.12-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What is MLVault?

MLVault is a personal AI-powered research assistant for machine learning engineers and students. Instead of manually searching through research papers every time you need to understand an algorithm or find the right approach, MLVault lets you query all of them at once — and get back a precise, cited answer with the relevant section, equation, and diagram from the actual paper.

Built to work **inside your editor** (Cursor or Claude Code) via an MCP server, so you never leave your workflow.

---

## Example Queries

```
"I'm building a transformer — which positional encoding should I use?"
→ RoPE is recommended for length generalization. See Attention Is All You Need, 
  Section 3.5, Equation (4). Also covered in RoFormer (Su et al., 2021), Section 3.

"What are the failure modes of GANs and how do papers fix them?"
→ Mode collapse and training instability are the two main failure modes...
  [Goodfellow et al. 2014, Section 4] [Arjovsky et al. 2017 — WGAN, Section 2]

"Can I use attention inside a CNN? Which paper covers this?"
→ Yes. CBAM (Convolutional Block Attention Module) covers this directly.
  See Figure 2 for the architecture diagram.
```

---

## Architecture

```
PDFs (150+)
  ↓
marker-pdf      →  clean markdown + LaTeX equations preserved
PyMuPDF         →  extract figures as images
pix2tex         →  LaTeX OCR on equation images
Gemini Vision   →  semantic captions for each figure
  ↓
Section-aware chunker  →  paper → section → paragraph (with full metadata)
  ↓
Embed (text-embedding-3-large)  →  Pinecone vector store
BM25 index                      →  keyword search index
  ↓
Query (from Cursor / Claude Code via MCP)
  → Query rewriter
  → Parallel: Pinecone vector search + BM25
  → Merge (~20 chunks)
  → Cohere Rerank  →  top 4–5 chunks
  → LLM synthesis (Gemini 2.0 Pro / AWS Bedrock)
  → Answer + citations + equations + figure references
  ↓
Response streamed back to editor
```

---

## Stack

| Layer | Tool |
|---|---|
| PDF Parsing | `marker-pdf` + `PyMuPDF` |
| Equation OCR | `pix2tex` |
| Figure Captioning | Gemini Vision (Gemini 2.0 Flash) |
| Embeddings | `text-embedding-3-large` |
| Vector Store | Pinecone |
| Keyword Search | BM25 (`rank_bm25`) |
| Reranker | Cohere Rerank (`rerank-english-v3.0`) |
| Synthesis LLM | Gemini 2.0 Pro / AWS Bedrock (configurable) |
| Server | FastAPI + MCP protocol |
| Editor Integration | Cursor, Claude Code |

---

## Project Structure

```
mlvault/
├── src/
│   ├── ingestion/      # PDF parser, chunker, embedder, Pinecone indexer
│   ├── retrieval/      # vector search, BM25, Cohere reranker
│   ├── synthesis/      # LLM call, citation formatter
│   └── server/         # FastAPI MCP server
├── data/
│   ├── raw/            # original PDFs (not committed)
│   └── processed/      # parsed markdown + extracted figures
├── notebooks/          # exploration, eval, debugging
├── .env.example        # API key template
└── README.md
```

---

## Setup

**1. Clone and install**
```bash
git clone https://github.com/dhrumin/mlvault.git
cd mlvault
pip install -r requirements.txt
```

**2. Configure environment**
```bash
cp .env.example .env
# Fill in: PINECONE_API_KEY, COHERE_API_KEY, GEMINI_API_KEY (or AWS keys)
```

**3. Add your PDFs**
```bash
# Drop your PDF papers into data/raw/
```

**4. Run ingestion**
```bash
python src/ingestion/run.py
# Parses all PDFs → chunks → embeds → indexes into Pinecone
```

**5. Start the MCP server**
```bash
uvicorn src.server.main:app --reload --port 8000
```

**6. Connect in Cursor / Claude Code**
Add to your MCP config:
```json
{
  "mcpServers": {
    "mlvault": {
      "url": "http://localhost:8000/mcp"
    }
  }
}
```

---

## MCP Tools (available in editor)

| Tool | What it does |
|---|---|
| `query_vault` | Main query — returns cited answer with sections + equations |
| `search_papers` | Search for papers by title, author, or topic |
| `get_paper` | Get full metadata for a specific paper |
| `search_equations` | Find papers containing a specific equation or formula |

---

## Roadmap

- [x] Project design and architecture
- [ ] Phase 1 — Ingestion pipeline (parser → chunker → embedder → Pinecone)
- [ ] Phase 2 — Retrieval engine (hybrid search + Cohere Rerank)
- [ ] Phase 3 — Synthesis layer (LLM + citation formatter)
- [ ] Phase 4 — FastAPI MCP server
- [ ] Phase 5 — Eval set + retrieval accuracy benchmarks

---

## Why This Exists

ML researchers and students accumulate hundreds of papers but have no good way to search across them. Existing tools either require uploading to a third-party service or don't understand equations and figures. MLVault runs locally (except the LLM call), keeps your papers private, and integrates directly into your coding workflow.

---

Built by [Dhrumin Upadhyay](https://github.com/dhrumin06) · Ravion Lab
