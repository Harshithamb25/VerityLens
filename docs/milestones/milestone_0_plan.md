# VerityLens — Milestone 0: Architecture, Compatibility & Setup Baseline

---

## 1. Milestone Objective

Establish the architectural foundations of **VerityLens**, verify platform and dependency compatibility across Node.js and Python runtimes without polluting global packages, and establish baseline documentation and environment templates.

---

## 2. Verified Environment Findings

The following environment baseline was verified through direct inspection on the host machine:

| Component | Target Version | Detected Version | Architecture | Status |
| :--- | :--- | :--- | :--- | :---: |
| **Node.js** | `>= 20.x` | `v24.11.1` | Windows x64 | **Verified** |
| **npm** | `>= 10.x` | `11.6.2` | Windows x64 | **Verified** |
| **Python** | `3.13.x` | `3.13.7` | Windows x64 (`C:\Users\Harshitha\AppData\Local\Programs\Python\Python313`) | **Verified** |
| **pip** | Latest stable | `26.0.1` | Windows x64 | **Verified** |
| **Git** | `>= 2.40` | `2.51.0.windows.1` | Windows x64 | **Verified** |
| **Git Branch** | `milestone/0-setup` | `milestone/0-setup` | Tracking `origin/milestone/0-setup` | **Verified** |
| **Git Baseline Tree** | Clean initial state | `0 untracked, 0 modified` | Initial baseline at commit `8136b60` (`chore: initialize VerityLens project`), prior to Phase C documentation additions | **Verified** |

---

## 3. Dependency & Local Embedding Compatibility Record

### 3.1 Isolated Test Environment
To test runtime compatibility without polluting the global Python environment, an isolated virtual environment was created outside the project workspace and outside OneDrive at:
`%TEMP%\veritylens_compat_test` (`C:\Users\Harshitha\AppData\Local\Temp\veritylens_compat_test`)

### 3.2 PyPI Binary Wheel Resolution
Direct queries against PyPI JSON endpoints confirmed the availability of native Windows x64 wheels for Python 3.13:
- `torch 2.14.1`: Native `torch-2.14.1-cp313-cp313-win_amd64.whl` (118.4 MB CPU build)
- `onnxruntime 1.31.0`: Native `onnxruntime-1.31.0-cp313-cp313-win_amd64.whl` (14.2 MB)
- `sentence-transformers 6.2.0`: Universal `sentence_transformers-6.2.0-py3-none-any.whl` (0.7 MB)
- `fastembed 0.9.0`: Universal `fastembed-0.9.0-py3-none-any.whl` (0.1 MB)

### 3.3 Multi-Stage Verification Results

```text
[Stage 1: Package Installation]
Command: pip install sentence-transformers
Result: Exit Code 0. Successfully installed 42 packages including torch 2.14.1+cpu and sentence-transformers 6.2.0.

[Stage 2: Module Imports]
Command: python -c "import torch, sentence_transformers; print(torch.__version__, sentence_transformers.__version__)"
Output: torch version: 2.14.1+cpu | sentence_transformers version: 6.2.0 | cuda available: False
Result: Exit Code 0. Both modules imported without C-extension errors.

[Stage 3: Model Weight Loading]
Model: sentence-transformers/all-MiniLM-L6-v2
Output: Downloaded from Hugging Face Hub (103/103 weight files) and loaded into memory in 17.42s.
Result: Successful weight caching and model instantiation on Python 3.13 CPU.

[Stage 4: Sample Sentence Inference]
Input: "VerityLens is an evidence-aware document intelligence and research workspace."
Output:
  - Inference time: 0.0321s (32 ms)
  - Output type: numpy.ndarray
  - Vector shape: (384,)
  - Vector dimension: 384
  - Sample values (first 5): [-0.06509, -0.04410, -0.00453, 0.03753, 0.05356]
Result: EXACT 384-DIMENSIONAL REQUIREMENT PROVEN.
```

### 3.4 Distinction Between Verified Results and Outstanding Checks

- **Verified Facts**:
  - Python 3.13.7 on Windows x64 installs and runs `torch` 2.14.1 (CPU) and `sentence-transformers` 6.2.0.
  - `all-MiniLM-L6-v2` executes local CPU inference in ~32ms and produces 384-dimensional dense vectors.
  - Node.js 24 and npm 11 resolve modern Next.js and React packages.
  - No global Python packages or workspace files were modified during compatibility testing.
- **Assumptions & Outstanding Checks (Deferred to Future Milestones)**:
  - Live Supabase project credentials (URL, service-role key, JWKS/signing configuration, anon key) and network latency have not been tested.
  - Real xAI API key access, quota, and exact model identifier availability (e.g., `grok-beta` or other supported models) have not been tested.
  - Full multi-page PDF coordinate extraction and layout parsing under `pdfplumber` will be verified in Milestone 3.

---

## 4. Approved Milestone Implementation Roadmap

Each milestone possesses strict acceptance criteria and must be formally reviewed and approved before the subsequent milestone begins:

- **Milestone 0: Architecture, Environment Compatibility, and Setup Baseline (Current)**
  - Confirm decoupled architecture and component boundaries.
  - Validate Python 3.13 and Node 24 runtime dependencies and test local 384-dimensional embedding inference.
  - Create architecture documentation, milestone plan, and documented `.env.example`.
- **Milestone 1: Application Foundation & Health Checks**
  - Scaffold Next.js 14+ frontend (`frontend/`) and FastAPI backend (`backend/`).
  - Implement Pydantic configuration validation, CORS middleware, and liveness/readiness health endpoints (`GET /health`).
  - Establish strict TypeScript checking (`tsc --noEmit`), Python linting, and verify local frontend-to-backend communication.
- **Milestone 2: Supabase Schema, Authentication, Private Storage & RLS**
  - Apply database migrations: `projects`, `documents`, `document_chunks` (with HNSW index on `vector(384)`), `research_sessions`, `chat_messages`, `evidence_matrices`, `matrix_entries`, and `citations`.
  - Configure Supabase Auth and token verification in FastAPI using project-supported signing configurations (e.g., JWKS public-key endpoint); derive user identity strictly from verified token claims.
  - Provision private Supabase Storage bucket `user-documents` with signed URL access.
  - Verify Row Level Security (RLS) enforcement against unauthorized cross-user access.
- **Milestone 3: PDF Ingestion, Chunking, Embeddings & Retrieval Foundation**
  - Implement page-aware PDF parser preserving text, character offsets, and bounding-box coordinates.
  - Implement structural chunker respecting page boundaries.
  - Integrate verified `all-MiniLM-L6-v2` embedding pipeline.
  - Implement hybrid retrieval combining pgvector dense cosine similarity with PostgreSQL full-text search and ranking (`tsvector` / `tsquery` with `ts_rank`) fused via Reciprocal Rank Fusion (RRF).
  - Implement explicit handling for scanned or unsupported documents.
- **Milestone 4: Document Lens Grounded Q&A & Citation Validation**
  - Integrate xAI Grok API with structured JSON outputs.
  - Implement streaming Q&A via Server-Sent Events (SSE).
  - Implement the **Citation Validation Engine** with distinct checks for quote existence vs. semantic claim support.
  - Enforce explicit "insufficient evidence" messaging when document context lacks support.
- **Milestone 5: Research Lens, Cross-Paper Analysis & Evidence Matrix**
  - Implement academic paper structured extractor (methodology, datasets, metrics, results, limitations, applications).
  - Implement multi-paper comparative analysis.
  - Implement the 10-column **Evidence Matrix** enforcing the strict evidence absence policy (retrieval miss is not proof of absence; `not_reported` requires sufficient text inspection; incomplete coverage yields `extraction_failure` or `requires_verification`; accuracy percentage is never inferred from other metrics).
  - Implement validated CSV export backed by cell-level audit records.
- **Milestone 6: Persistent Workspace, Library, History & PDF Navigation**
  - Implement split-screen workspace: interactive PDF canvas reader on the left, chat and Evidence Matrix on the right.
  - Implement visual citation links: clicking an inline badge jumps to the exact PDF page with visible highlight overlay.
  - Implement persistent project sessions, document library management, and search history.
- **Milestone 7: Regression, Security, End-to-End Verification & Deployment**
  - Run comprehensive backend test suite (`pytest`) and frontend component checks (`Vitest` / `Playwright`).
  - Conduct security review: ensure zero server secrets exposed to browser and verify RLS boundaries.
  - Configure production deployment to Vercel (frontend) and containerized backend host.

---

## 5. Milestone 0 Acceptance Checklist

- [x] Workspace location confirmed: `c:\Users\Harshitha\OneDrive\Desktop\MBH\Github Projects\VerityLens`.
- [x] Initial commit `8136b60` and branch `milestone/0-setup` confirmed intact; `main` left untouched.
- [x] Node.js (`v24.11.1`), npm (`11.6.2`), Python (`3.13.7`), pip (`26.0.1`), and Git (`2.51.0`) verified.
- [x] Python 3.13 binary wheels confirmed available on PyPI for `torch` and `onnxruntime`.
- [x] Isolated test environment (`%TEMP%\veritylens_compat_test`) installed without modifying global packages.
- [x] `torch` 2.14.1 and `sentence-transformers` 6.2.0 imported cleanly without errors.
- [x] `sentence-transformers/all-MiniLM-L6-v2` loaded, cached, and tested for inference.
- [x] Output vector dimension verified as **384** (inference latency: 32ms).
- [x] Architecture specification completed: [`docs/architecture.md`](file:///c:/Users/Harshitha/OneDrive/Desktop/MBH/Github%20Projects/VerityLens/docs/architecture.md).
- [x] Milestone plan completed: [`docs/milestones/milestone_0_plan.md`](file:///c:/Users/Harshitha/OneDrive/Desktop/MBH/Github%20Projects/VerityLens/docs/milestones/milestone_0_plan.md).
- [x] Documented environment template completed: [`.env.example`](file:///c:/Users/Harshitha/OneDrive/Desktop/MBH/Github%20Projects/VerityLens/.env.example).
- [x] Project `README.md` updated with architecture summary, verified status, and roadmap.
- [x] Proposed documentation artifacts reviewed; `git diff --check` passes cleanly.
