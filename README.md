<div align="center">

# 🧠 Recall Engine

### A Retrieval-Augmented, Citation-Grounded Knowledge Copilot for Industrial Teams

*Turn scattered engineering drawings, maintenance logs, safety procedures, and regulatory documents into a single, queryable source of truth — with every answer traceable back to its source.*

[![Python](https://img.shields.io/badge/Python-85.8%25-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Offline First](https://img.shields.io/badge/Offline--First-No%20API%20Keys%20Required-2ea44f)](#)
[![Hybrid RAG](https://img.shields.io/badge/Retrieval-Hybrid%20(Dense%20%2B%20BM25)-purple)](#)
[![Tests](https://img.shields.io/badge/Tests-pytest-yellow?logo=pytest)](#)

</div>

---

## 📖 Overview

**Recall Engine** is a Retrieval-Augmented Generation (RAG) platform purpose-built for industrial environments where knowledge lives across 7–12 disconnected systems — P&IDs, CMMS work orders, safety procedures, inspection reports, operating instructions, project files, and regulatory submissions.

Instead of forcing field technicians and engineers to hunt through folders and legacy systems, Recall Engine unifies all of it into one **conversational, mobile-ready interface** that answers questions in natural language — and never answers without proof.

Every response the system generates is:

- **Grounded** — backed by inline `[n]` citations pointing to the exact source document
- **Scored** — accompanied by a calibrated confidence score, not a black-box guess
- **Actionable** — paired with suggested next steps (raise a work order, confirm a permit, escalate for review)

The system was designed to solve three concrete problems in industrial operations: **knowledge fragmentation**, **unplanned downtime caused by incomplete equipment context**, and the **"knowledge cliff"** created when experienced engineers retire and take undocumented expertise with them.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🔌 **Zero-dependency offline mode** | Runs entirely on-device with a deterministic hashing embedder and extractive synthesizer — no API keys, no internet, works on an air-gapped plant network. |
| ☁️ **Pluggable cloud LLMs** | Drop in OpenAI or Anthropic with a single environment variable for higher-quality generation. Falls back gracefully to offline mode if a key or package is missing. |
| 🔍 **Hybrid retrieval engine** | Combines dense cosine-similarity search with BM25 lexical scoring, so both semantic questions *and* exact equipment-tag lookups (`P-101A`, `FT-2301`) resolve accurately. |
| 📎 **Grounded, cited answers** | Every answer carries inline citations, a confidence score derived from semantic agreement × query-term coverage × corroboration, and direct source links. |
| 🤖 **Agentic workflows** | Purpose-built agents run multi-pass retrieval to assemble briefings a single query can't — maintenance history diagnostics and regulatory compliance gap analysis. |
| 📱 **Mobile-first field UI** | A lightweight, responsive chat interface designed for technicians in the field, with a full desktop experience for engineers. |
| 🏷️ **Automatic document intelligence** | Detects document type and extracts ISA-style equipment tags automatically from filename and content — no manual tagging required. |
| 🖼️ **Structured drawing digitization** | Parses P&ID / engineering drawing JSON sidecars directly into searchable, citable prose. |
| 🧩 **Composable architecture** | Swap the vector store for pgvector or another production DB, or plug in a computer-vision drawing parser, without touching the rest of the pipeline. |

---

## 🏗️ Architecture & Pipeline

Recall Engine follows a classic **ingest → index → retrieve → generate** RAG pipeline, purpose-tuned for heterogeneous industrial data:

```
            INGEST                    INDEX                        ANSWER
 documents ─────────▶ loaders ─────▶ chunker ─────▶ embeddings ─────▶ KnowledgeStore
(pdf / csv / json /   (type +        (sentence-      (offline hashing    (hybrid vector
  md front-matter)     tag infer)     aware, w/        or OpenAI/          + BM25 index)
                                       overlap)         Anthropic)               │
                                                                                  ▼
                                             query ──▶ hybrid search ──▶ Copilot ──▶ answer
                                                        (dense + BM25)     (generate + cite
                                                                            + score + suggest
                                                                             next actions)
```

**Pipeline stages:**

1. **Loaders** — detect document type (drawing, maintenance, safety, inspection, operating, project, regulatory) from filename and content, extract ISA-style equipment tags via regex, and parse structured P&ID JSON sidecars into readable prose.
2. **Chunker** — splits source text sentence-aware with configurable overlap to preserve context across chunk boundaries.
3. **Embedding layer** — offline deterministic hashing embedder by default; transparently swappable for OpenAI or Anthropic embeddings via environment variable.
4. **Knowledge Store** — a persistent hybrid index combining dense vector similarity with BM25 lexical scoring for robust retrieval across both semantic and exact-match queries.
5. **Copilot** — synthesizes a grounded answer from retrieved chunks, attaches inline citations and a calibrated confidence score, and surfaces recommended next actions.
6. **Agents** — orchestrate multi-pass retrieval across the knowledge store to produce composite outputs (e.g., a full maintenance briefing or a compliance gap report) that a single retrieval pass cannot assemble.

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| **API Framework** | FastAPI + Uvicorn (ASGI server) |
| **Data Validation** | Pydantic v2 |
| **Language** | Python 3 (core engine), HTML/JS (mobile-first web UI) |
| **Retrieval** | Custom hybrid retriever — cosine similarity (dense) + BM25 (lexical) |
| **Embeddings** | Deterministic offline hashing embedder (default) · OpenAI · Anthropic (pluggable) |
| **Document Parsing** | `pypdf` for PDF text extraction; native parsers for CSV, JSON, and Markdown/front-matter |
| **Persistence** | Lightweight persistent hybrid vector + lexical store (JSON-backed, swappable for pgvector or another vector DB) |
| **File Uploads** | `python-multipart` |
| **Testing** | `pytest` — 13-test suite covering embeddings, loaders, chunking, storage, persistence, citation/confidence logic, and agents |

---

## 🚀 Quick Start

```bash
# Install dependencies (fully offline-capable out of the box)
pip install -r requirements.txt

# Option A — one command: seed the sample corpus and launch the web app
python run.py
# then open http://127.0.0.1:8000 on your laptop or phone

# Option B — CLI workflow
python scripts/seed.py                     # ingest sample_data/
python -m iki.cli ask "Why does pump P-101A keep tripping?"
python -m iki.cli diagnose P-101A          # maintenance briefing agent
python -m iki.cli compliance "cooling water system"
python -m iki.cli stats
```

No API key is required for any of the above — the offline backend handles it end-to-end.

### Optional: Using a Cloud LLM

```bash
export IKI_AI_PROVIDER=openai
export OPENAI_API_KEY=sk-...
# or: IKI_AI_PROVIDER=anthropic + ANTHROPIC_API_KEY=...

python -m iki.cli ask "Summarise the failure history of P-101A"
```

If the provider, package, or key is missing, the system logs a notice and transparently falls back to the offline backend — it never hard-fails.

---

## 🌐 REST API

Start the server with `python run.py` (or `uvicorn iki.api.app:app`):

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/` | Mobile-first web chat UI |
| `GET` | `/api/health` | Index stats and active embedder info |
| `POST` | `/api/query` | Core RAG query — returns answer, citations, and confidence score |
| `GET` | `/api/documents` | List all ingested documents |
| `GET` | `/api/doc_types` | Available document-type filters |
| `POST` | `/api/ingest/text` | Ingest raw text into the knowledge base |
| `POST` | `/api/ingest/file` | Upload a file (`pdf` / `csv` / `json` / `md` / `txt`) |
| `POST` | `/api/agent/maintenance` | Generate a maintenance briefing for a given equipment tag |
| `POST` | `/api/agent/compliance` | Generate a regulatory compliance gap analysis for a topic |

**Example:**

```bash
curl -s localhost:8000/api/query \
  -H 'content-type: application/json' \
  -d '{"query":"What is the low flow trip setpoint for FT-2301?"}' | python -m json.tool
```

---

## 📄 Supported Document Formats

| Format | Handling |
|---|---|
| `.json` | Structured P&ID / drawing digitization (equipment, lines, instruments) |
| `.csv` | CMMS / work-order exports, flattened into records |
| `.md` / `.txt` | Procedures, instructions, and reports, with optional `--- front-matter ---` metadata |
| `.pdf` | Text extracted via `pypdf`; scanned PDFs are automatically flagged for OCR |

Document type can also be declared explicitly via front-matter:

```yaml
---
title: SOP — Lockout/Tagout for Cooling Water Pumps
doc_type: safety_procedure
---
```

---

## 🧪 Sample Corpus

Included in `sample_data/` is a realistic cooling-water system spanning pumps **P-101A/B**, heat exchanger **HX-12**, and surge vessel **V-201** — modeled across all seven supported document types, including a retiring engineer's *tribal-knowledge* handover note.

This lets you see the "knowledge cliff" problem being solved firsthand: ask *"Why does P-101A keep tripping?"* and the Copilot surfaces the undocumented insight — *check the surge-vessel level first* — complete with a citation back to the original handover note.

---

## ✅ Testing

```bash
pip install pytest
pytest -q
```

The suite covers **13 tests** spanning embeddings, document loaders, chunking, the knowledge store, persistence, Copilot citation/confidence logic, and agent workflows.

---

## 📁 Project Layout

```
iki/
├── ingestion/   # Loaders, chunker, ingestion pipeline
├── ai/          # Embeddings + generation backends (offline | openai | anthropic)
├── store/       # Persistent hybrid vector + lexical knowledge store
├── rag/         # Copilot core + maintenance/compliance agents
├── api/         # FastAPI application
└── web/         # Mobile-first chat UI

sample_data/     # Demo industrial corpus (7 document types)
scripts/seed.py  # Corpus seeding script
tests/           # Test suite
```

---

## 🔧 Extensibility

Recall Engine was designed with clear extension seams:

- **Vector store** — swap the default JSON-backed store for `pgvector` or another production vector database by reimplementing the `KnowledgeStore` interface, which is intentionally small.
- **Drawing digitization** — the structured P&ID loader is the integration point for a computer-vision drawing-parsing pipeline. Emit the same JSON shape and the rest of the pipeline works unchanged.
- **Agentic integrations** — `_ACTION_RULES` inside `copilot.py` is where CMMS/QMS system integrations hook into the suggested-actions engine.

---

## 🎯 Why This Design

- **Runs anywhere.** No model downloads, no API keys, no internet dependency — demo-ready on a laptop or deployable on an air-gapped plant network.
- **Best-of-both retrieval.** Hybrid dense + lexical search means both natural-language questions and precise part-number lookups return accurate results.
- **Trustworthy by construction.** Answers are never presented without a citation trail and a confidence score, with explicit low-confidence warnings surfaced to the user.
- **Goes beyond single-query lookups.** Agentic workflows assemble multi-source briefings — failure history, procedures, inspection status, and regulatory gaps — that a single retrieval pass cannot produce.

---

<div align="center">

**Recall Engine** — because the answer your team needs shouldn't be buried in seven different systems.

</div>
