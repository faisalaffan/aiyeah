# Product Requirements Document

# AI Yeah — Modular AI Platform

**Version:** 0.1.0-draft  
**Author:** Muhammad Faisal Affan  
**Status:** Draft  
**Last Updated:** 2026-05-26

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Goals & Non-Goals](#3-goals--non-goals)
4. [Target Users](#4-target-users)
5. [Product Overview](#5-product-overview)
6. [Architecture Overview](#6-architecture-overview)
7. [Feature Modules](#7-feature-modules)
   - 7.1 Model Gateway
   - 7.2 Document AI
   - 7.3 RAG Engine
   - 7.4 Agent Runner
   - 7.5 Memory System
   - 7.6 Search & Grounding
   - 7.7 Text-to-SQL
   - 7.8 Multimodal
   - 7.9 Observability & Eval
   - 7.10 Prompt Manager
8. [API Surface](#8-api-surface)
9. [Web UI](#9-web-ui)
10. [Multi-Tenancy & Auth](#10-multi-tenancy--auth)
11. [Tech Stack](#11-tech-stack)
12. [Build Phases & Milestones](#12-build-phases--milestones)
13. [Success Metrics](#13-success-metrics)
14. [Risks & Trade-offs](#14-risks--trade-offs)
15. [Out of Scope](#15-out-of-scope)
16. [Open Questions](#16-open-questions)

---

## 1. Executive Summary

**AI Yeah** adalah modular AI platform yang memungkinkan developer dan non-technical user untuk membangun, menjalankan, dan mengevaluasi AI-powered workflows tanpa perlu mengintegrasikan setiap tools dari scratch.

Platform ini menyediakan dua surface:

- **API / SDK** — untuk developer yang ingin compose capabilities secara programatik
- **Web UI** — untuk non-technical user yang ingin menggunakan AI capabilities via antarmuka visual

AI Yeah bukan framework baru. AI Yeah adalah **integration layer** yang memilih tools terbaik per kategori dan mengeksposnya melalui unified interface yang konsisten.

---

## 2. Problem Statement

### Saat ini

Membangun AI-powered product membutuhkan:

- Integrasi manual ke 10–20+ libraries yang berbeda (LLM providers, vector DBs, agent frameworks, eval tools)
- Masing-masing punya API contract, auth mechanism, dan error model yang berbeda
- Tidak ada standar untuk observability, cost tracking, atau eval lintas tools
- Developer harus rebuild boilerplate yang sama di setiap project

### Akibatnya

- Time-to-first-working-feature tinggi (days, bukan hours)
- Inconsistent behavior antar environment (dev vs prod)
- Non-technical user tidak bisa bereksperimen tanpa engineering support
- Sulit untuk benchmark atau compare output dari model/approach yang berbeda

### Hipotesis

Jika AI Yeah menyediakan unified, modular interface ke AI ecosystem yang relevan, maka:

- Developer dapat build AI features dalam hitungan jam bukan hari
- Non-technical user dapat menjalankan AI workflows secara mandiri
- Tim dapat measure dan improve AI quality secara sistematis

---

## 3. Goals & Non-Goals

### Goals

| #   | Goal                                                                         | Priority |
| --- | ---------------------------------------------------------------------------- | -------- |
| G1  | Unified API untuk multiple LLM providers (OpenAI, Anthropic, Gemini, Ollama) | P0       |
| G2  | RAG pipeline end-to-end: ingest → embed → retrieve → generate                | P0       |
| G3  | Agent runner dengan tool use dan multi-step reasoning                        | P0       |
| G4  | Observability: trace setiap LLM call dengan cost, latency, token usage       | P0       |
| G5  | Web UI untuk non-technical user menjalankan workflows                        | P1       |
| G6  | Prompt versioning dan management                                             | P1       |
| G7  | Eval pipeline: RAGAS untuk RAG, DeepEval untuk agent output                  | P1       |
| G8  | Multi-tenant: isolasi data dan API keys per workspace                        | P1       |
| G9  | Text-to-SQL capability dengan schema awareness                               | P2       |
| G10 | Multimodal: STT, TTS, image understanding                                    | P2       |

### Non-Goals (v1)

- Fine-tuning pipeline (Unsloth, Axolotl) — butuh GPU infrastructure terpisah
- Self-hosted LLM serving (vLLM, TGI) — scope berbeda
- Mobile app
- Real-time collaboration
- Marketplace / plugin ecosystem

---

## 4. Target Users

### Primary: Developer (API Consumer)

**Who:** Fullstack/backend engineer yang membangun AI features untuk product mereka.

**Pain points:**

- Integrasi LLM providers satu per satu memakan waktu
- Tidak ada standar observability
- Switching provider membutuhkan rewrite significant

**Jobs to be done:**

- "Saya mau add RAG ke app saya tanpa build dari scratch"
- "Saya mau compare output GPT-4o vs Claude Sonnet untuk use case saya"
- "Saya mau trace semua LLM calls di production"

**Technical profile:** Familiar dengan REST API, dapat baca dokumentasi, comfort dengan JSON/Python/TypeScript.

---

### Secondary: Non-Technical User (UI Consumer)

**Who:** Analyst, ops, founder, atau content person yang ingin leverage AI tapi tidak bisa code.

**Pain points:**

- Harus minta bantuan engineer untuk setiap AI task
- ChatGPT bagus tapi tidak bisa connect ke data internal
- Tidak ada audit trail untuk AI-generated content

**Jobs to be done:**

- "Saya mau upload dokumen dan tanya-jawab tanpa minta tolong engineer"
- "Saya mau jalankan agent yang search web dan buat ringkasan harian"
- "Saya mau lihat history dan quality dari AI outputs saya"

**Technical profile:** Comfort dengan web apps, tidak perlu tahu API/code.

---

## 5. Product Overview

### Core Concept: Capability Modules

AI Yeah diorganisir sebagai kumpulan **capability modules** yang dapat digunakan secara independent atau di-compose:

```
Developer API          Web UI
     │                   │
     └──────────┬─────────┘
                │
     ┌──────────▼──────────┐
     │   Capability Layer   │
     │                      │
     │  Model Gateway        │
     │  Document AI          │
     │  RAG Engine           │
     │  Agent Runner         │
     │  Memory System        │
     │  Search & Grounding   │
     │  Text-to-SQL          │
     │  Multimodal           │
     │  Observability        │
     │  Prompt Manager       │
     └──────────┬──────────┘
                │
     ┌──────────▼──────────┐
     │  Integration Layer   │
     │  LiteLLM  pgvector   │
     │  LangFuse LlamaIndex │
     │  LangGraph Instructor│
     └──────────┬──────────┘
                │
     ┌──────────▼──────────┐
     │   Provider Layer     │
     │ OpenAI  Anthropic    │
     │ Gemini  Ollama       │
     └─────────────────────┘
```

### Key Design Principles

1. **Modular** — setiap capability bisa digunakan standalone
2. **Provider-agnostic** — model dapat diganti tanpa mengubah logic
3. **Observable by default** — setiap LLM call otomatis ter-trace
4. **Typed contracts** — semua input/output menggunakan Pydantic schema
5. **Fail explicitly** — error yang jelas, bukan silent failure

---

## 6. Architecture Overview

### System Components

```
┌─────────────────────────────────────────────────────┐
│                    Client Layer                      │
│   Next.js Web UI          REST API Consumers         │
└──────────────────────────────┬──────────────────────┘
                               │ HTTPS
┌──────────────────────────────▼──────────────────────┐
│                  API Gateway (FastAPI)                │
│   Auth middleware   Rate limiting   Request routing  │
└──────┬──────────────────────────────────────────────┘
       │
┌──────▼──────────────────────────────────────────────┐
│                  Service Layer                       │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Model    │  │   RAG    │  │  Agent   │           │
│  │ Gateway  │  │  Engine  │  │  Runner  │           │
│  └──────────┘  └──────────┘  └──────────┘           │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Document │  │ Memory   │  │  Prompt  │           │
│  │    AI    │  │  System  │  │ Manager  │           │
│  └──────────┘  └──────────┘  └──────────┘           │
└──────┬──────────────────────────────────────────────┘
       │
┌──────▼──────────────────────────────────────────────┐
│                  Data Layer                          │
│  PostgreSQL + pgvector   Redis (cache/queue)         │
│  Object Storage (S3-compatible)                      │
└─────────────────────────────────────────────────────┘
       │
┌──────▼──────────────────────────────────────────────┐
│               Observability Layer                    │
│   LangFuse (LLM traces)   Prometheus + Grafana       │
└─────────────────────────────────────────────────────┘
```

### Data Flow: RAG Request (Example)

```
User query
    │
    ▼
API Gateway (auth check, rate limit)
    │
    ▼
RAG Engine
    ├── Embed query (LiteLLM → embedding model)
    ├── Hybrid search (pgvector + BM25)
    ├── Rerank (Cohere Rerank)
    ├── Assemble context
    └── Generate (LiteLLM → LLM)
    │
    ▼
LangFuse (trace: tokens, latency, cost, retrieval score)
    │
    ▼
Response → User
```

---

## 7. Feature Modules

### 7.1 Model Gateway

**Purpose:** Unified interface ke semua LLM providers. Provider dapat diganti via config tanpa mengubah calling code.

**Underlying tool:** LiteLLM

**Capabilities:**

- Chat completion (streaming & non-streaming)
- Embeddings
- Provider fallback (jika provider A down, fallback ke B)
- Cost tracking per request
- Caching (semantic cache via Redis)

**API Contract:**

```
POST /v1/chat
{
  "model": "anthropic/claude-sonnet-4-20250514",  // atau "openai/gpt-4o"
  "messages": [...],
  "stream": false,
  "options": {
    "temperature": 0.7,
    "max_tokens": 1000
  }
}
```

**Supported providers (v1):**

- OpenAI (GPT-4o, o1, text-embedding-3)
- Anthropic (Claude Sonnet, Haiku, Opus)
- Google (Gemini 1.5 Pro, Flash)
- Ollama (local models: llama3, mistral, qwen)

**NFR:**

- P99 latency overhead vs direct provider call: < 50ms
- Provider switch: zero code change required

---

### 7.2 Document AI

**Purpose:** Ingest dokumen dari berbagai format dan konversi ke teks/markdown yang dapat diproses LLM.

**Underlying tools:** Docling (IBM), marker, Surya (OCR), Unstructured.io

**Supported formats:**

- PDF (text-based dan scanned)
- DOCX, PPTX, XLSX
- HTML, Markdown
- Images (PNG, JPG) via OCR

**Capabilities:**

- Layout-aware parsing (preserve heading structure, tables)
- Table extraction → JSON/CSV
- Image extraction dari dokumen
- Chunking strategy: by heading, by sentence, by token count
- Metadata extraction (author, date, title)

**API Contract:**

```
POST /v1/documents/ingest
Content-Type: multipart/form-data
{
  "file": <binary>,
  "options": {
    "chunk_strategy": "heading",  // heading | sentence | fixed_token
    "chunk_size": 512,
    "extract_tables": true,
    "ocr": false
  }
}

Response:
{
  "document_id": "doc_xxx",
  "chunks": [...],
  "metadata": {...},
  "tables": [...]
}
```

---

### 7.3 RAG Engine

**Purpose:** End-to-end Retrieval-Augmented Generation pipeline.

**Underlying tools:** LlamaIndex, pgvector, Cohere Rerank, Jina Embeddings

**Pipeline stages:**

```
Document chunks
      │
      ▼
Embedding (Jina / OpenAI text-embedding-3)
      │
      ▼
Index (pgvector)
      │
      ▼ (on query)
Hybrid Search
  ├── Dense: vector similarity (pgvector)
  └── Sparse: BM25 (keyword)
      │
      ▼
Reranking (Cohere Rerank)
      │
      ▼
Context Assembly (top-k chunks + metadata)
      │
      ▼
Generation (Model Gateway)
      │
      ▼
Response + Sources
```

**API Contract:**

```
POST /v1/rag/query
{
  "query": "apa syarat pengajuan kredit?",
  "collection_id": "col_xxx",
  "options": {
    "top_k": 5,
    "hybrid_alpha": 0.7,   // 0=BM25 only, 1=vector only
    "rerank": true,
    "model": "anthropic/claude-haiku-4-5-20251001",
    "citation_mode": true
  }
}

Response:
{
  "answer": "...",
  "sources": [
    { "chunk_id": "...", "document_id": "...", "score": 0.91, "text": "..." }
  ],
  "usage": { "tokens": 1240, "cost_usd": 0.0004 }
}
```

**Eval metrics (auto-computed via RAGAS):**

- Faithfulness
- Answer Relevancy
- Context Precision
- Context Recall

---

### 7.4 Agent Runner

**Purpose:** Jalankan multi-step reasoning agent dengan tool use.

**Underlying tools:** LangGraph, Instructor, Pydantic AI

**Agent types:**

| Type             | Description                  | Use case            |
| ---------------- | ---------------------------- | ------------------- |
| ReAct            | Reason → Act → Observe loop  | General purpose     |
| Plan-and-Execute | Plan upfront → execute steps | Multi-step research |
| Multi-Agent      | Supervisor + worker agents   | Complex workflows   |

**Built-in tools:**

- `web_search` — Tavily / SearXNG
- `rag_query` — query internal knowledge base
- `sql_query` — Text-to-SQL execution
- `document_read` — baca dokumen dari storage
- `http_request` — generic HTTP call
- Custom tools via function registration

**API Contract:**

```
POST /v1/agents/run
{
  "agent_type": "react",
  "system_prompt": "...",
  "user_message": "Cari info terbaru tentang BI-Fast dan buat ringkasan",
  "tools": ["web_search", "rag_query"],
  "model": "openai/gpt-4o",
  "max_steps": 10,
  "stream": true
}
```

**Structured output:**

```
POST /v1/agents/extract
{
  "text": "...",
  "schema": {
    "type": "object",
    "properties": {
      "name": { "type": "string" },
      "amount": { "type": "number" }
    }
  },
  "model": "anthropic/claude-haiku-4-5-20251001"
}
```

---

### 7.5 Memory System

**Purpose:** Persistent memory untuk agent dan conversation sessions.

**Underlying tools:** Mem0, pgvector

**Memory types:**

| Type            | Storage  | TTL          | Use case                    |
| --------------- | -------- | ------------ | --------------------------- |
| Working memory  | Redis    | Session      | In-conversation context     |
| Episodic memory | pgvector | Configurable | Past conversation summaries |
| Semantic memory | pgvector | Permanent    | User facts, preferences     |

**API Contract:**

```
POST /v1/memory/store
{
  "user_id": "usr_xxx",
  "content": "User prefers concise responses in Bahasa Indonesia",
  "memory_type": "semantic"
}

GET /v1/memory/retrieve?user_id=usr_xxx&query=language+preference&top_k=3
```

---

### 7.6 Search & Grounding

**Purpose:** Web search dan real-time information retrieval untuk agent grounding.

**Underlying tools:** Tavily API, SearXNG (self-hosted), Exa

**Providers:**

| Provider | Mode        | Best for                   |
| -------- | ----------- | -------------------------- |
| Tavily   | Managed API | Production, reliable       |
| SearXNG  | Self-hosted | Privacy, no API cost       |
| Exa      | Managed API | Semantic/similarity search |

**API Contract:**

```
POST /v1/search
{
  "query": "BI-Fast payment Indonesia 2025",
  "provider": "tavily",   // tavily | searxng | exa
  "options": {
    "max_results": 5,
    "include_raw_content": true,
    "search_depth": "advanced"
  }
}
```

---

### 7.7 Text-to-SQL

**Purpose:** Konversi natural language ke SQL query dengan schema awareness.

**Underlying tools:** Vanna.ai, SQLCoder (Defog), LlamaIndex SQL

**Approach:**

1. Schema registry: store tabel definitions + sample data + column descriptions
2. RAG over schema: retrieve relevant tables untuk query
3. Generate SQL via fine-tuned model
4. Execute + validate hasil
5. Natural language explanation dari hasil

**API Contract:**

```
POST /v1/sql/query
{
  "question": "berapa total penjualan bulan ini per kategori?",
  "connection_id": "conn_xxx",
  "options": {
    "explain": true,
    "safe_mode": true   // only SELECT, no write
  }
}

Response:
{
  "sql": "SELECT category, SUM(amount) ...",
  "result": [...],
  "explanation": "Query mengambil total penjualan...",
  "confidence": 0.87
}
```

---

### 7.8 Multimodal

**Purpose:** Speech-to-text, text-to-speech, dan image understanding.

**Underlying tools:** Whisper (STT), ElevenLabs / Kokoro (TTS), Moondream (vision)

**Capabilities:**

| Capability    | Tool                      | API                         |
| ------------- | ------------------------- | --------------------------- |
| Speech → Text | Whisper                   | `POST /v1/audio/transcribe` |
| Text → Speech | ElevenLabs / Kokoro       | `POST /v1/audio/synthesize` |
| Image → Text  | Moondream / GPT-4o Vision | `POST /v1/vision/describe`  |
| Document OCR  | Surya                     | via Document AI module      |

---

### 7.9 Observability & Eval

**Purpose:** Trace, monitor, dan evaluate semua AI operations.

**Underlying tools:** LangFuse, RAGAS, DeepEval, Promptfoo

**Auto-captured per request:**

- LLM call: model, prompt, completion, tokens, cost, latency
- RAG: retrieval scores, chunk sources, faithfulness score
- Agent: full step trace, tool calls, intermediate outputs

**Eval capabilities:**

| Eval type         | Tool      | Metric                               |
| ----------------- | --------- | ------------------------------------ |
| RAG quality       | RAGAS     | Faithfulness, relevancy, recall      |
| LLM output        | DeepEval  | Correctness, hallucination, toxicity |
| Prompt regression | Promptfoo | Score diff antar prompt versions     |

**API Contract:**

```
GET /v1/traces?start=2026-05-01&end=2026-05-26
GET /v1/traces/{trace_id}
POST /v1/eval/run
{
  "eval_set": [...],
  "pipeline": "rag",
  "metrics": ["faithfulness", "answer_relevancy"]
}
```

---

### 7.10 Prompt Manager

**Purpose:** Version control, testing, dan deployment untuk prompts.

**Underlying tools:** Custom + Agenta (optional)

**Capabilities:**

- Prompt versioning (semver)
- A/B testing antar prompt versions
- Variable interpolation dengan Jinja2-style templating
- Prompt library dengan tagging dan search
- Rollback ke versi sebelumnya

**API Contract:**

```
POST /v1/prompts
{
  "name": "invoice-extractor",
  "template": "Extract invoice data from: {{ document }}\nReturn JSON with fields: ...",
  "version": "1.0.0",
  "tags": ["extraction", "finance"]
}

GET /v1/prompts/invoice-extractor?version=latest
POST /v1/prompts/invoice-extractor/render
{
  "variables": { "document": "..." }
}
```

---

## 8. API Surface

### Base URL

```
https://api.aiyeah.dev/v1
```

### Authentication

```
Authorization: Bearer <api_key>
X-Workspace-ID: ws_xxx
```

### Standard Response Envelope

```json
{
  "data": { ... },
  "meta": {
    "request_id": "req_xxx",
    "trace_id": "trc_xxx",
    "duration_ms": 342,
    "usage": {
      "tokens_in": 512,
      "tokens_out": 256,
      "cost_usd": 0.0008
    }
  },
  "error": null
}
```

### Error Format

```json
{
  "data": null,
  "error": {
    "code": "PROVIDER_RATE_LIMIT",
    "message": "OpenAI rate limit exceeded. Retrying with fallback provider.",
    "provider": "openai",
    "retry_after": 5
  }
}
```

### Rate Limits (default per workspace)

| Tier       | RPM    | TPM    |
| ---------- | ------ | ------ |
| Free       | 60     | 100K   |
| Pro        | 600    | 1M     |
| Enterprise | Custom | Custom |

---

## 9. Web UI

### Pages / Views

| Page        | User | Description                                   |
| ----------- | ---- | --------------------------------------------- |
| Dashboard   | Both | Usage overview, recent activity, cost summary |
| Playground  | Both | Test model, prompt, RAG queries interactively |
| Documents   | Both | Upload, manage, search documents              |
| Collections | Both | Manage RAG collections                        |
| Agents      | Both | Create, configure, run agents                 |
| Prompts     | Dev  | Prompt library dan version management         |
| Traces      | Both | LLM call history dan analytics                |
| Eval        | Dev  | Run eval suites, view results                 |
| Settings    | Both | API keys, workspace config, billing           |

### UI Stack

- **Framework:** Next.js 15 (App Router)
- **Styling:** Tailwind CSS
- **Components:** shadcn/ui
- **State:** Zustand
- **Real-time:** Server-Sent Events (streaming responses)

---

## 10. Multi-Tenancy & Auth

### Model

- **Workspace** = unit isolasi utama (1 company / 1 user = 1 workspace)
- **User** = member dari workspace (role: owner, admin, member)
- **API Key** = scoped ke workspace, bukan ke user

### Data Isolation

- Row-level: semua tabel memiliki `workspace_id` column
- Vector store: namespace per workspace di pgvector
- Object storage: prefix `/{workspace_id}/` per bucket

### Auth Flow

- Web UI: email/password + JWT (access token 15min, refresh 7d)
- API: static API key (hashed di DB, prefix-based identification)
- Provider keys: encrypted at rest (AES-256), dekripsi hanya saat request

---

## 11. Tech Stack

### Backend

| Layer             | Technology            | Rationale                                  |
| ----------------- | --------------------- | ------------------------------------------ |
| API Framework     | FastAPI (Python)      | Native async, type hints, OpenAPI auto-gen |
| AI Orchestration  | LangGraph, LlamaIndex | Production-grade, active development       |
| Model Routing     | LiteLLM               | Unified interface, fallback support        |
| Structured Output | Instructor + Pydantic | Type-safe, reliable extraction             |
| Observability     | LangFuse              | Open-source, self-hostable                 |

### Data

| Layer          | Technology            | Rationale                              |
| -------------- | --------------------- | -------------------------------------- |
| Primary DB     | PostgreSQL 16         | ACID, pgvector support                 |
| Vector Store   | pgvector              | Co-located dengan data, no extra infra |
| Cache          | Redis                 | Session, semantic cache                |
| Object Storage | MinIO (S3-compatible) | Self-hostable, dokumen storage         |

### Frontend

| Layer         | Technology               |
| ------------- | ------------------------ |
| Framework     | Next.js 15               |
| Styling       | Tailwind CSS + shadcn/ui |
| State         | Zustand                  |
| Data fetching | TanStack Query           |

### Infrastructure

| Layer      | Technology                                       |
| ---------- | ------------------------------------------------ |
| Container  | Docker + Docker Compose (dev), Kubernetes (prod) |
| CI/CD      | GitHub Actions                                   |
| Monitoring | Prometheus + Grafana                             |
| Logging    | Loki                                             |

---

## 12. Build Phases & Milestones

### Phase 1 — Foundation (Weeks 1–3)

**Goal:** Core infrastructure + Model Gateway working end-to-end

- [ ] FastAPI project structure (routing, middleware, error handling)
- [ ] LiteLLM integration (OpenAI + Anthropic + Ollama)
- [ ] PostgreSQL schema + migration setup (Alembic)
- [ ] Auth system (API key + JWT)
- [ ] LangFuse integration (auto-trace semua LLM calls)
- [ ] Basic Next.js UI dengan Playground
- [ ] Docker Compose untuk local dev

**Deliverable:** Developer dapat hit API, kirim chat ke multiple providers, lihat trace di LangFuse.

---

### Phase 2 — RAG Engine (Weeks 4–5)

**Goal:** Full RAG pipeline dari document upload sampai grounded answer

- [ ] Document ingestion (Docling + Unstructured)
- [ ] Chunking strategies (heading, sentence, fixed-token)
- [ ] pgvector setup + embedding pipeline
- [ ] Hybrid search (vector + BM25)
- [ ] Cohere Rerank integration
- [ ] RAGAS eval pipeline
- [ ] UI: Document manager + RAG query UI

**Deliverable:** Upload PDF, tanya-jawab dengan citation, lihat retrieval quality score.

---

### Phase 3 — Agent Runner (Weeks 6–8)

**Goal:** Multi-step agent dengan tool use

- [ ] LangGraph ReAct agent
- [ ] Tool registry (web_search, rag_query, http_request)
- [ ] Instructor structured output
- [ ] Mem0 memory integration
- [ ] Streaming response via SSE
- [ ] UI: Agent builder + step trace viewer

**Deliverable:** Agent yang bisa search web + query knowledge base + return structured output, semua ter-trace.

---

### Phase 4 — Polish & Showcase (Weeks 9–10)

**Goal:** Production-ready demo, clean docs

- [ ] Prompt Manager (versioning + A/B test)
- [ ] Usage dashboard (cost, token, latency charts)
- [ ] API documentation (Swagger + README)
- [ ] Demo scenarios end-to-end
- [ ] Performance optimization (caching, connection pooling)
- [ ] Security audit (input validation, injection prevention)

**Deliverable:** Platform siap demo ke interviewer, GitHub repo dengan README yang komprehensif.

---

## 13. Success Metrics

### Technical Metrics (measurable)

| Metric                          | Target                    |
| ------------------------------- | ------------------------- |
| API P99 latency (non-LLM)       | < 100ms                   |
| RAG faithfulness score (RAGAS)  | > 0.80                    |
| Text-to-SQL accuracy (test set) | > 75%                     |
| LLM cost tracking accuracy      | 100% (semua call ter-log) |
| Uptime                          | > 99% (demo environment)  |

### Portfolio Metrics

| Metric                          | Target                                 |
| ------------------------------- | -------------------------------------- |
| GitHub stars                    | > 50 dalam 3 bulan post-launch         |
| Demo completion rate (visitors) | Orang bisa pakai tanpa onboarding call |
| Interview mentions              | Dibahas di > 80% technical interviews  |

---

## 14. Risks & Trade-offs

### Risk 1: Python vs Go

**Risk:** Semua AI libraries Python-native. Kamu lebih kuat di Go.  
**Mitigation:** Platform ini full Python. Go untuk future performance layer jika perlu.  
**Trade-off:** Short-term learning curve, long-term alignment dengan ecosystem.

### Risk 2: LiteLLM abstraction overhead

**Risk:** LiteLLM menambah latency dan kadang punya quirks antar provider.  
**Mitigation:** Benchmark setiap provider; bypass LiteLLM untuk hot path jika perlu.  
**Trade-off:** Developer experience vs raw performance.

### Risk 3: pgvector vs dedicated vector DB

**Risk:** pgvector performa menurun di skala > 1M vectors.  
**Mitigation:** Untuk portofolio scope ini non-issue. Design collection interface agar pluggable (Qdrant sebagai swap).  
**Trade-off:** Simplicity sekarang vs scalability nanti.

### Risk 4: Scope creep

**Risk:** "Implement semua tools" bisa jadi 6+ bulan project.  
**Mitigation:** Phase 1-3 saja sudah cukup kuat untuk portofolio. Phase 4+ adalah roadmap.  
**Trade-off:** Completeness vs time-to-demo.

### Risk 5: API key management security

**Risk:** Menyimpan OpenAI/Anthropic keys user adalah security responsibility serius.  
**Mitigation:** Encrypt at rest (AES-256), never log raw keys, short-lived decryption.  
**Note:** Untuk demo/portofolio, user bring-their-own-key (BYOK) lebih simple dan aman.

---

## 15. Out of Scope

Items berikut **tidak** akan diimplementasi di v1:

- **Fine-tuning pipeline** — butuh GPU, beda domain concern
- **Self-hosted LLM serving** (vLLM, TGI) — infra complexity tidak sebanding untuk portofolio
- **Browser automation agents** (Skyvern, Stagehand) — niche use case
- **MLflow / W&B experiment tracking** — relevan untuk ML researcher, bukan AI engineer platform
- **Mobile app** — web-first untuk v1
- **Real-time collaboration** — out of scope
- **Billing/payment system** — portofolio demo, bukan commercial product

---

## 16. Open Questions

| #   | Question                                                  | Owner  | Deadline      |
| --- | --------------------------------------------------------- | ------ | ------------- |
| OQ1 | BYOK (bring-your-own-key) atau platform manages API keys? | Faisal | Phase 1 start |
| OQ2 | Self-hosted LangFuse atau cloud LangFuse?                 | Faisal | Phase 1 start |
| OQ3 | Nama domain final platform?                               | Faisal | Phase 4       |
| OQ4 | License: MIT atau source-available?                       | Faisal | Phase 4       |
| OQ5 | Demo data set apa yang paling impressive untuk showcase?  | Faisal | Phase 2 end   |

---

_Document ini adalah living document. Update setiap ada keputusan arsitektur atau perubahan scope._

---

**AI Yeah** — Build AI products, not plumbing.
