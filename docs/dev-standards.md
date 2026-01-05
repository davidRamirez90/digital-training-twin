# Development Standards

## Formatting & Linting
- **Python (backend)**: `ruff` for linting + formatting.
- **TypeScript/Angular (frontend)**: `eslint` + `prettier`.
- **Markdown**: `markdownlint` (basic ruleset).

## Testing
- **Backend**: `pytest` for unit/integration tests.
- **Frontend**: Angular `ng test` (Karma/Jest per project setup).
- CI should run lint + unit tests for both FE/BE.

## Commit Style
Use Conventional Commits:
```
type(scope): short summary
```
Examples:
- `docs(phase0): add project glossary`
- `feat(api): add health endpoint`
- `chore(ci): add backend lint job`

## Branching
- `main` is protected.
- Feature work happens on short-lived branches merged via PR.

## Review Expectations
- All changes should include updated docs/tests where relevant.
- Keep PRs small and focused (prefer <400 LOC changes).
