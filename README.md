# VerityLens

**Evidence-aware document intelligence and research workspace.**

VerityLens is an advanced research workspace engineered to help users analyze ordinary PDF documents, systematically inspect academic research papers, compare findings across papers, and rigorously trace AI-generated assertions back to verifiable source evidence.

Unlike traditional RAG systems that treat generated text as authoritative, VerityLens separates quote existence from actual claim support, enforces a strict Evidence Matrix accuracy and evidence-absence policy, and preserves cell-level evidence auditing down to specific PDF pages and bounding boxes.

---

## Workspace Modes

- **Document Lens**: Question answering on ordinary PDF documents (reports, manuals, whitepapers). Returns answers grounded in retrieved document excerpts, visual bounding-box citation links to specific PDF pages, and explicit reporting when evidence is missing or insufficient.
- **Research Lens**: Systematic academic paper analysis. Extracts methodologies, datasets, evaluation metrics, findings, advantages, limitations, and applications; constructs a multi-paper **Evidence Matrix**; and provides claim-level evidence auditing against source passages.

---

## Approved Architecture

VerityLens follows a decoupled monorepo architecture:

- **Frontend (`frontend/`)**: Next.js 14+ (App Router), React, Strict TypeScript, Tailwind CSS, Lucide icons. Features a split-screen canvas PDF viewer with bounding-box highlights, streaming chat, and an interactive 10-column Evidence Matrix table with validated CSV export. *(Planned for subsequent milestones)*
- **Backend (`backend/`)**: Python 3.13, FastAPI, Pydantic v2, Uvicorn. Coordinates page-aware PDF parsing, local vector embedding computation, hybrid retrieval (pgvector dense vector search + PostgreSQL full-text search and ranking), xAI Grok API integration, and the Citation Validation Engine. *(Planned for subsequent milestones)*
- **Persistence & Security (`supabase/`)**: Supabase (PostgreSQL, `pgvector`, Supabase Auth, Storage). Enforces user isolation through Row Level Security (RLS) on all tables, stores embeddings in an HNSW-indexed 384-dimensional vector column, verifies tokens against the project's supported JWKS/public-key configuration, and protects uploaded documents in private buckets. *(Planned for subsequent milestones)*
- **Semantic Embeddings**: Local `sentence-transformers/all-MiniLM-L6-v2` running on CPU. Generates 384-dimensional dense vectors with zero external API fees. *(Verified compatible in Milestone 0)*
- **Reasoning Engine**: xAI API (Grok models). Communicates strictly through server-side HTTPS using structured JSON outputs. Server secrets remain completely backend-only. *(Planned for subsequent milestones)*

For technical specifications, data models, and the citation validation flow, consult [`docs/architecture.md`](docs/architecture.md).

---

## Verified Environment Baseline (Milestone 0)

The runtime environment was directly verified on Windows x64:

- **Node.js**: `v24.11.1`
- **npm**: `11.6.2`
- **Python**: `3.13.7` (64-bit)
- **pip**: `26.0.1`
- **Git**: `2.51.0.windows.1`
- **PyTorch**: `2.14.1+cpu` (verified via pre-built binary wheel on Python 3.13)
- **Sentence-Transformers**: `6.2.0` (verified module import)
- **Embedding Inference**: `sentence-transformers/all-MiniLM-L6-v2` loaded and tested in an isolated test environment (`%TEMP%\veritylens_compat_test`); verified exact **384-dimensional** vector output with 32ms inference latency.

---

## Milestone Roadmap

Work is executed incrementally with measurable acceptance criteria and review checkpoints at every milestone:

- **Milestone 0: Architecture, Compatibility & Setup Baseline** *(Current Milestone)*
  - Establish approved architecture, verified dependency matrix, milestone plan, and environment templates.
- **Milestone 1: Application Foundation & Health Checks**
  - Scaffold Next.js and FastAPI projects; add configuration validation, liveness/readiness probes, and verify frontend-to-backend communication.
- **Milestone 2: Supabase Schema, Authentication, Private Storage & RLS**
  - Provision database tables, 384-dimension pgvector indexes, Supabase Auth token verification via project-supported signing configurations (e.g., JWKS), private storage buckets, and RLS policies.
- **Milestone 3: PDF Ingestion, Chunking, Embeddings & Retrieval Foundation**
  - Implement page-aware PDF extraction, structural chunking, local embedding generation, and hybrid retrieval combining pgvector dense search with PostgreSQL full-text search and ranking.
- **Milestone 4: Document Lens Grounded Q&A & Citation Validation**
  - Integrate xAI Grok streaming, citation validation engine (quote existence vs claim support), and insufficient evidence handling.
- **Milestone 5: Research Lens, Cross-Paper Analysis & Evidence Matrix**
  - Implement academic paper structured extractor, 10-column Evidence Matrix, strict evidence-absence policy (retrieval miss is not proof of absence; `not_reported` requires sufficient text inspection; incomplete coverage yields `extraction_failure` or `requires_verification`; accuracy percentage is never inferred from other metrics), and validated CSV export.
- **Milestone 6: Persistent Workspace, Library, History & PDF Navigation**
  - Implement split-screen canvas reader, visual page citation jumping, persistent research sessions, and paper library.
- **Milestone 7: Regression, Security, End-to-End Verification & Deployment**
  - End-to-end testing, security audits, and production deployment configuration for Vercel and backend hosting.

Detailed milestone records and acceptance criteria are documented in [`docs/milestones/milestone_0_plan.md`](docs/milestones/milestone_0_plan.md).

---

## Development Status & Prerequisites

> [!NOTE]
> **Implementation Status**: VerityLens is currently at **Milestone 0 (Architecture & Setup Baseline)**. Application code, active database schemas, authentication flows, and live AI connections are planned for subsequent milestones and are not yet operational.

### Outstanding Prerequisites for Next Milestones
1. **Supabase Project**: Requires creating a Supabase project and adding credentials to `backend/.env` and `frontend/.env.local`.
2. **xAI API Access**: Requires an active `XAI_API_KEY` configured in the backend environment.
3. **Configuration Template**: Refer to [`.env.example`](.env.example) for documented variable names and requirements.

---

## Security Principles

- **No Committed Secrets**: Never commit real `.env`, `.env.local`, API keys, tokens, or service-role keys.
- **Server-Only Credentials**: `XAI_API_KEY` and `SUPABASE_SERVICE_ROLE_KEY` remain strictly on the backend and are never exposed to client bundles.
- **User-Scoped Isolation**: Data access is governed by Supabase Auth JWT verification and PostgreSQL Row Level Security. Identity is derived strictly from verified token claims.
- **Private Document Storage**: Uploaded PDFs are stored in private storage buckets accessible only via authenticated signed URLs.
- **No Fabricated Data**: Citations and Evidence Matrix entries require verified grounding in source text; unsupported claims are flagged rather than masked.
