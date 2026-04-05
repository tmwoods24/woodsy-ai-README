# Woodsy

Used by VC analysts and associates across seed-to-Series A funds.
**Live at [woodsy-ai.com](https://woodsy-ai.com)**

**AI-powered due diligence memo generation for venture capital professionals.**

Woodsy is a full-stack SaaS platform that ingests startup pitch materials and automatically produces structured investment memos. It combines document parsing, retrieval-augmented generation (RAG), and LLM-driven analysis to deliver analyst-grade diligence output in a fraction of the time.

---

## Overview

VC analysts spend significant time synthesizing pitch decks, financials, and data room documents into structured memos. Woodsy automates this workflow end-to-end: upload any combination of supported documents, configure your analysis preferences, and receive a fully structured investment memo with citations grounded in the source material.

---

## Core Features

### Document Ingestion & Parsing
- Accepts PDF, PPTX, PPT, DOCX, DOC, XLSX, XLS, TXT, and Markdown files
- Extracts and chunks text for downstream retrieval and analysis
- Supports multi-document jobs (e.g., deck + data room + financials together)

### RAG Pipeline
- Embeddings stored in a persistent ChromaDB vector database
- Hybrid retrieval combining semantic search and metadata filtering
- Configurable retrieval parameters via `rag_config.py`
- Index persistence across sessions

### Memo Generation
- LLM-driven synthesis using Anthropic's Claude models
- Output structured by a configurable section registry (market, team, financials, competition, risks, etc.)
- Specialist prompts per section for depth and consistency
- Token cost estimation before generation

### Template System
- **System templates**: curated defaults covering common investment verticals (SaaS, Marketplace, Fintech, Deep Tech, and more)
- **Custom templates**: users can clone and fully customize any template
- **Template editor**: full-screen GUI editor for configuring sections, analysis framework, and output style
- **Guided route**: streamlined template selection flow with vertical-based defaults
- **Memo analyzer**: upload an existing memo to automatically suggest a matching template configuration; detects novel sections not present in the registry and routes them to admin review

### Diligence Lens System
- Stage-aware analysis profiles (Pre-Seed through Series C+)
- Lens catalog covering dimensions such as product, growth, defensibility, and founder-market fit
- Per-stage lens overrides and framework presets

### Investor Profile Context
- Users can provide a brief profile of their fund's thesis and focus areas
- Profile context is injected into generation to bias analysis toward what matters for that investor
- Confirm flow before generation to surface how the profile will influence output

### Data Room
- Separate data room ingestion and chat interface
- Ask questions directly against a company's data room documents
- Job-scoped isolation per company

### Memo Comparison
- Side-by-side comparison of two memos generated from different configurations
- LLM-as-judge evaluation with configurable scoring dimensions
- Exportable evaluation reports

### Job Management
- Async job queue with status polling
- Per-user job history with filtering
- Job-scoped resource ownership and access control

### Enrichment
- Optional enrichment providers for additional context: LinkedIn (company profiles), SEC filings, USPTO patent data, Gemini web search
- Provider-level enable/disable configuration
- Caching layer to avoid redundant API calls

### Billing & Plans
- Subscription plans with per-plan memo generation limits
- One-time token top-ups for burst usage
- Stripe-integrated checkout and webhook handling

### Email Notifications
- Optional email summaries via Resend
- Admin notifications for novel section review queue

### Export
- Memo export functionality for offline distribution

---

## Technical Architecture

| Layer | Technology |
|---|---|
| Web framework | FastAPI (Python) |
| Database | SQLite via `aiosqlite` (async) and `sqlite3` (sync for background threads) |
| Vector store | ChromaDB |
| LLM provider | Anthropic Claude |
| Templating | Jinja2 |
| Billing | Stripe |
| Email | Resend |
| Deployment | Railway (persistent volume for DB and vector store) |
| Frontend | Vanilla JS + HTML/CSS (no framework) |

### Key Design Decisions

**Async-first API, sync DB writes from threads.** FastAPI routes use `aiosqlite` for non-blocking DB access. Background analysis threads (spawned via `threading.Thread`) use synchronous `sqlite3` helpers to avoid event loop conflicts.

**Section registry as the source of truth.** All memo sections are defined in a central registry. The registry is extended at query time by merging in any admin-approved novel sections detected during memo analysis, keeping the core registry clean while allowing the system to learn from real memos.

**Template compilation as a translation layer.** The template system compiles a `TemplateConfig` (sections, framework, output style) into the prompt structure consumed by the generation pipeline — decoupling user-facing configuration from internal prompt logic.

**RAG configuration is explicit.** Retrieval parameters (chunk size, overlap, top-k, score thresholds) are managed in a dedicated config module rather than scattered across the codebase, making tuning straightforward.

---

## Project Structure (high-level)

```
woodsy/
├── web/                        # FastAPI application
│   ├── main.py                 # App entry point, lifespan, middleware
│   ├── config.py               # Environment variables and paths
│   ├── database.py             # SQLite schema, migrations, CRUD helpers
│   ├── auth.py                 # Session auth helpers
│   ├── routes/
│   │   ├── api.py              # REST API endpoints
│   │   ├── pages.py            # Server-rendered page routes
│   │   └── dataroom.py         # Data room endpoints
│   ├── section_registry.py     # Section definitions and registry
│   ├── template_schema.py      # Pydantic models for template config
│   ├── template_compiler.py    # Compiles TemplateConfig → prompt structure
│   ├── memo_analyzer.py        # Analyzes uploaded memos for template matching
│   ├── vertical_defaults.py    # Vertical-specific template defaults
│   ├── diligence_lens_catalog.py
│   ├── stage_profiles.py
│   ├── framework_presets.py
│   ├── chat.py                 # Chat interface logic
│   ├── dataroom_chat.py        # Data room chat logic
│   ├── email_service.py        # Resend integration
│   ├── comparison.py           # Memo comparison and evaluation
│   ├── jobs.py                 # Async job runner
│   ├── guardrails.py
│   └── templates/              # Jinja2 HTML templates
│       └── static/             # CSS, JS assets
├── enrichment/                 # External data enrichment providers
├── parsers/                    # Document parsing (PDF, etc.)
├── eval/                       # Evaluation framework
├── Export/                     # Memo export utilities
├── extract_memo.py             # Top-level memo extraction entrypoint
├── specialist_prompts.py       # Per-section LLM prompts and base rules
├── schemas.py                  # Shared Pydantic schemas
├── rag_config.py               # RAG retrieval configuration
├── rag_setup.py                # ChromaDB initialization and indexing
├── ingest.py                   # Document ingestion pipeline
└── tests/                      # Test suite
```

---

## Testing

The test suite covers:
- Template schema validation and versioning
- Template compiler correctness
- Template storage CRUD (DB round-trips)
- Section registry merging (system + novel sections)
- Guided route backend logic
- Token estimation
- Framework presets
- Diligence lens defaults
- Memo analyzer pipeline (mocked LLM)
- User isolation and access control
- Phase-specific backend integration tests

Tests use `pytest` with `pytest-asyncio`. No real API calls are made — all LLM and external service dependencies are mocked. The database is isolated per test using `tmp_path`.

---

## Deployment

The application is deployed on Railway with a persistent volume mounted for the SQLite database and ChromaDB vector store. Environment variables cover LLM API keys, Stripe keys, email configuration, CORS origins, and auth secrets.

---

*Codebase is proprietary and not publicly available.*
