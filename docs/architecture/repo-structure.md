# Repository Structure Decision

## Decision
Use a single mono-repo that contains both the frontend (Angular) and backend (FastAPI) codebases, along with shared documentation.

## Rationale
- **Single source of truth** for domain docs, schemas, and product decisions.
- **Simplified coordination** between FE/BE changes (API contracts, auth flow).
- **Shared tooling** (lint/format, release checklist) can live in one place.
- **Clear onboarding** for contributors: one repo, one set of docs.

## Proposed Layout
```
/
  docs/                 # Architecture, API, metrics, security, runbooks
  frontend/             # Angular app (Cloudflare Pages)
  backend/              # FastAPI service (Cloud Run)
  infra/                # IaC, deployment configs, scripts
  .github/              # CI workflows
```

## Notes
If the team later needs independent release trains or ownership boundaries,
the FE/BE can be split with minimal disruption because the docs and contracts
are already structured to be portable.
