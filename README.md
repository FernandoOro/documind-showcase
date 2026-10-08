# DocuMind — Enterprise RAG API (showcase)

**Self-hostable, API-first, multi-tenant RAG API.** Companies run it on their own servers, even fully offline, and ask questions about their documents with cited sources.

> This repository is a public showcase of a private project: the architecture, the process and the results. The source code is private; I'm happy to walk through it in a technical interview.

**Stack:** Python · FastAPI · LangGraph · Celery · Redis · PostgreSQL · Qdrant · Docker · Next.js

---

## Highlights

- **Hybrid retrieval:** dense vectors + BM25 sparse vectors, fused with Reciprocal Rank Fusion (RRF) inside Qdrant. Answer accuracy reached **91.4% on a legal golden dataset (+2.9 points)**.
- **Fully local option:** Spanish embeddings (`jina-embeddings-v2-base-es`) and a local LLM behind the same multi-LLM adapter, so it can run with no internet access.
- **Multi-tenant by design:** tenant isolation enforced in every query (including each hybrid prefetch), self-service API keys, RBAC, rate limiting and per-tenant usage.
- **Async ingestion:** Celery workers with observable progress; PDF → Markdown before chunking; chunking by Markdown headers instead of fixed token windows.
- **Production-ready MVP:** non-root Docker images, observability, audit log, smoke and load tests verified; 195 tests at the end of the MVP.
- **Web console (Next.js):** upload documents and ask questions with cited sources.

---

## Architecture

Hexagonal architecture (ports & adapters) with Domain-Driven Design:

```
src/documind/
├── domain/          entities, value objects, business rules
├── application/     use cases + ports (interfaces)
├── infrastructure/  parsers, vector store, LLM providers, database, Celery
└── api/             FastAPI routers, schemas, middleware
```

Adapters are swappable without touching the core:

| Port | Adapters |
|---|---|
| Vector store | Qdrant (production) · ChromaDB (integration tests) |
| LLM | OpenAI · OpenRouter · local LLM |
| Embeddings | OpenAI · local Spanish embeddings (with Redis cache) |

Tests are split into **unit** (no Docker, no network), **integration** (in-memory ChromaDB) and **end-to-end** (all services up).

---

## How it's built: Spec-Driven Development

Every change starts as a spec. Each spec contains:

1. **Goal and scope** (including what is explicitly out of scope)
2. **Acceptance criteria** in Given / When / Then form
3. **Technical design** and the list of files to create or modify
4. **Test plan** mapping every acceptance criterion to a test
5. **Deviations**, filled in during implementation

Then: failing tests first → implementation → verification against each criterion → one PR per spec.

| Spec | Scope |
|---|---|
| SPEC-001 | Core RAG: PDF → query → answer with sources |
| SPEC-002 | Multi-tenancy and authentication |
| SPEC-003 | Async ingestion and multi-LLM |
| SPEC-004 | Production hardening |
| SPEC-005 | Legal golden dataset |
| SPEC-006 | Local embeddings |
| SPEC-007 | Hybrid search (BM25 + RRF) |
| SPEC-008 | Retrieval presets |
| SPEC-009 | Local LLM |
| SPEC-010a | Identity core |
| SPEC-011 | Frontend foundation |

---

## Architecture Decision Records (selection)

| ADR | Decision |
|---|---|
| ADR-002 | ChromaDB in integration tests, Qdrant in production |
| ADR-004 | Chunk by Markdown headers, not by fixed token windows |
| ADR-005 | PDF → Markdown is mandatory before chunking |
| ADR-006 | Embedding cache in Redis |
| ADR-008 | Default similarity threshold recalibrated (0.75 → 0.40) from evaluation data |
| ADR-009 | Golden dataset as versioned JSONL, with deterministic scoring and no LLM judge |
| ADR-010 | Local Spanish embeddings as an optional provider |
| ADR-011 | Hybrid search validated; the default stays dense until the data justifies the switch |
| ADR-012 | Frontend: Next.js + Feature-Sliced Design |

---

## Evaluation

Retrieval and answers are measured against a **versioned golden dataset** (hit@5, answer correctness, and negative questions the system must refuse to answer), with deterministic scoring. Every retrieval change (embedding model, similarity threshold, hybrid mode) is compared against the dataset before it becomes a default.

---

## Status

- **MVP complete:** 4/4 sprints verified (core RAG, multi-tenancy + auth, async + multi-LLM, production).
- **Post-MVP:** local embeddings, hybrid search, retrieval presets, local LLM, identity and web console.

---

## Author

**Fernando Orozco** — AI Engineer · [LinkedIn](https://www.linkedin.com/in/uriel-fernando-orozco-castillo) · [GitHub](https://github.com/FernandoOro)
