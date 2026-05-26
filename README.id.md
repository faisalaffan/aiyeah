<p align="center">
  <a href="README.md">English</a> · <a href="README.id.md">Bahasa Indonesia</a>
</p>

<p align="center">
  <img src="assets/01_BANNER_LIGHT.png" alt="AIyeah Banner" width="100%" />
</p>

<p align="center">
  <img src="assets/04_LOGO_HORIZONTAL.png" alt="Logo AIyeah" height="64" />
</p>

<p align="center">
  <strong>"The AI that's always there."</strong>
</p>

<p align="center">
  Platform AI modular. Lapisan integrasi yang memilih tools terbaik per kategori<br/>
  dan mengeksposnya melalui antarmuka seragam — untuk developer dan pengguna non-teknis.
</p>

---

## Kenapa AIyeah

Membangun produk bertenaga AI hari ini berarti mengintegrasikan 10–20+ library secara manual — masing-masing dengan API contract, mekanisme auth, dan model error yang berbeda. Developer membangun ulang boilerplate yang sama di setiap project. Pengguna non-teknis tidak bisa bereksperimen tanpa bantuan engineer.

**AIyeah menyelesaikan ini** dengan menyediakan antarmuka tunggal yang konsisten ke ekosistem AI. Ganti provider, tambahkan kapabilitas, dan lacak setiap panggilan — tanpa menulis ulang lapisan integrasi.

---

## Arsitektur

```
┌─────────────────────────────────────────────────┐
│              Client Layer                         │
│   Next.js Web UI      REST API / SDK              │
└────────────────────┬────────────────────────────┘
                     │ HTTPS
┌────────────────────▼────────────────────────────┐
│            API Gateway (FastAPI)                  │
│   Auth · Rate Limiting · Request Routing         │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│              Capability Layer                     │
│                                                   │
│  Model Gateway   RAG Engine    Agent Runner       │
│  Document AI     Memory        Search/Grounding   │
│  Text-to-SQL     Multimodal    Observability      │
│  Prompt Manager                                   │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│            Integration Layer                      │
│  LiteLLM · LlamaIndex · LangGraph · Instructor   │
│  pgvector · LangFuse · Cohere · Jina             │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│              Provider Layer                       │
│  OpenAI · Anthropic · Gemini · Ollama             │
└─────────────────────────────────────────────────┘
```

---

## Modul Kapabilitas

| Modul | Fungsi | Dibangun dengan |
|-------|--------|-----------------|
| **Model Gateway** | Chat/completion/embedding terpadu lintas provider dengan fallback dan pelacakan biaya | LiteLLM |
| **RAG Engine** | Pipeline retrieval end-to-end: ingest → embed → hybrid search → rerank → generate | LlamaIndex, pgvector, Cohere |
| **Agent Runner** | Agen multi-langkah dengan tool use dan structured output | LangGraph, Instructor |
| **Document AI** | Parsing PDF, DOCX, PPTX, HTML, gambar menjadi teks terstruktur dan ter-chunk | Docling, Unstructured, Surya |
| **Memory System** | Memori persisten (working, episodic, semantic) untuk agen dan sesi | Mem0, pgvector |
| **Search & Grounding** | Pencarian web untuk grounding agen secara real-time | Tavily, SearXNG |
| **Text-to-SQL** | Natural language ke SQL dengan schema awareness dan mode aman | Vanna.ai, SQLCoder |
| **Multimodal** | Suara-ke-teks, teks-ke-suara, pemahaman gambar | Whisper, ElevenLabs, Moondream |
| **Observability** | Lacak otomatis setiap panggilan LLM: token, biaya, latensi, skor retrieval | LangFuse, RAGAS, DeepEval |
| **Prompt Manager** | Prompt dengan version control, A/B testing, dan rollback | Custom + Agenta |

---

## Prinsip Desain

- **Modular** — setiap kapabilitas bisa digunakan standalone. Komposisi sesuai kebutuhan.
- **Provider-agnostic** — ganti model tanpa mengubah logika aplikasi.
- **Observable by default** — setiap panggilan LLM otomatis terlacak.
- **Typed contracts** — semua input dan output menggunakan skema Pydantic.
- **Fail explicitly** — error yang jelas, tidak pernah gagal diam-diam.

---

## Tech Stack

| Lapisan | Teknologi |
|---------|-----------|
| API | FastAPI (Python) — async, type-safe, OpenAPI otomatis |
| Orkestrasi AI | LangGraph, LlamaIndex, Instructor + Pydantic |
| Routing Model | LiteLLM — antarmuka terpadu, fallback otomatis |
| Database | PostgreSQL 16 + pgvector |
| Cache | Redis |
| Penyimpanan | MinIO (kompatibel S3) |
| Observability | LangFuse, Prometheus + Grafana, Loki |
| Frontend | Next.js 15, Tailwind CSS, shadcn/ui |
| Infrastruktur | Docker Compose (dev), Kubernetes (prod) |

---

## Peta Jalan

| Fase | Waktu | Fokus |
|------|-------|-------|
| **1. Fondasi** | Minggu 1–3 | Struktur project FastAPI, LiteLLM multi-provider, auth, LangFuse traces, Docker Compose |
| **2. RAG Engine** | Minggu 4–5 | Ingestion dokumen, chunking, hybrid search, reranking, evaluasi RAGAS |
| **3. Agent Runner** | Minggu 6–8 | Agent LangGraph, tool registry, structured output, memory, SSE streaming |
| **4. Polish** | Minggu 9–10 | Prompt manager, dashboard penggunaan, dokumentasi API, audit keamanan |

→ [PRD Lengkap](docs/PRD.md) untuk requirements dan keputusan arsitektur yang detail.

---

## Memulai

> AIyeah sedang dalam Fase 1 (Fondasi). Dokumentasi dan panduan setup akan hadir bersamaan dengan rilis pertama.

---

## Komunitas

- [Kontribusi](CONTRIBUTING.md)
- [Kode Etik](CODE_OF_CONDUCT.md)
- [Kebijakan Keamanan](SECURITY.md)
- [Dukungan](SUPPORT.md)
- [Changelog](CHANGELOG.md)

---

## Lisensi

MIT © 2026 [Muhammad Faisal Affan](https://github.com/faisalaffan) — lihat [LICENSE](LICENSE).
