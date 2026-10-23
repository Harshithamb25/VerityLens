# VerityLens

**Evidence-aware document intelligence and research workspace.**

VerityLens is being developed to help users explore ordinary PDF documents, analyze research papers, compare findings across papers, and trace AI-generated claims back to source evidence.

## Planned capabilities

- **Document Lens:** PDF question answering with source-grounded responses.
- **Research Lens:** research-paper analysis and cross-paper comparisons.
- **Evidence Matrix:** structured paper findings with explicit reporting of accuracy and other metrics.
- **Evidence auditing:** source passages, page references, and citation validation.
- **Persistent workspace:** projects, documents, and conversation history.

## Planned technology

- Frontend: Next.js, React, TypeScript.
- Backend: Python, FastAPI, Pydantic.
- AI generation: xAI API, configured server-side.
- Persistence and authentication: Supabase.
- Semantic retrieval: local open-source embeddings, subject to compatibility testing.

## Development status

Initial repository setup. Features will be implemented and tested incrementally. Planned capabilities are not yet claimed to be implemented.

## Security

- Never commit API keys, passwords, service-role keys, or real `.env` files.
- Do not commit private documents or user-uploaded files.
- Keep server credentials on the backend.
- Enforce user-scoped access to stored data.

## Development approach

Work is organized into milestones. Each milestone should have documented changes, test results, and a review checkpoint before the next milestone begins.
