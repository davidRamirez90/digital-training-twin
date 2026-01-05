# Digital Twin Cycling — Implementation Backlog (Chronological)

> Each task is small, isolated, and independently testable. Checkboxes indicate completion.
>
> Abbreviations: FE = Angular frontend (Cloudflare Pages), BE = FastAPI backend (Cloud Run), DB = Supabase Postgres, R2 = Cloudflare R2.

---

## Phase 0 — Project setup & working agreements

- [x] Create repository structure (mono-repo vs split FE/BE) and document the decision in `docs/architecture/repo-structure.md`.
- [x] Define coding standards (formatters, linting, commit style) and record in `docs/dev-standards.md`.
- [x] Create `docs/glossary.md` (Fitness, Fatigue, Freshness, impulse, TSS/TRIMP, ΔV, ΔI, etc.).
- [x] Create `docs/non-goals.md` for MVP exclusions (Bluetooth control, full auto-planner, multi-user marketplace features).
- [x] Create `docs/mvp-definition.md` listing MVP user-visible outcomes and success criteria.
- [x] Decide and document the canonical impulse metric for MVP (Power-based TSS-like; HR-based fallback) in `docs/model/impulse-metric.md`.
- [x] Decide and document the ground-truth signal used for model fitting (e.g., compliance outcomes + rolling best-power proxy) in `docs/model/ground-truth.md`.
- [x] Create a risk register `docs/risk-register.md` (data access, FIT variability, time-series storage cost).

## Phase 1 — Infrastructure & environments (walking skeleton)

### 1.1 Accounts, environments, and secrets
- [ ] Create Supabase project and record required environment variables for local/dev/prod in `docs/env.md` (no secrets committed).
- [ ] Create Cloudflare R2 bucket and document bucket naming + lifecycle policy in `docs/storage/r2.md`.
- [ ] Create Cloudflare Pages project for FE and document build settings in `docs/deploy/cloudflare-pages.md`.
- [ ] Create Google Cloud project for Cloud Run and document region + service naming in `docs/deploy/cloud-run.md`.
- [ ] Choose BE secret management approach (Cloud Run env vars vs Secret Manager) and document in `docs/security/secrets.md`.

### 1.2 Local development workflow
- [ ] Create `README.md` with local run instructions (FE, BE, DB migrations) and troubleshooting.
- [ ] Add `.env.example` for FE and BE with placeholders.
- [ ] Decide API base URL strategy (local vs prod) and document in `docs/architecture/config.md`.

### 1.3 CI/CD minimal pipeline
- [ ] Set up CI for FE: install deps, lint, run unit tests, build artifact.
- [ ] Set up CI for BE: install deps, lint, run unit tests, verify Docker build.
- [ ] Add a release checklist `docs/release-checklist.md`.

## Phase 2 — Database baseline (Supabase Postgres)

### 2.1 Authentication & user profile
- [ ] Enable Supabase Auth and verify email/password login in test env.
- [ ] Create DB migration: `user_settings` (user_id PK/FK, ftp, units, optional HR fields, created_at, updated_at).
- [ ] Add RLS policy for `user_settings` (users can read/write their own).
- [ ] Apply migration and verify from Supabase SQL editor.
- [ ] Add BE endpoint `GET /settings` returning current user settings (stub ok initially).

### 2.2 Core entities (schemas only)
- [ ] Create DB migration: `activities` (user_id, source, start_time, duration, external_id, content_hash, r2_key, processing_status, processing_error).
- [ ] Create DB migration: `planned_workouts` (user_id, name, structure JSON, planned_date nullable).
- [ ] Create DB migration: `workout_assignments` (user_id, planned_workout_id, activity_id nullable, match_confidence, created_at).
- [ ] Create DB migration: `session_analyses` (user_id, activity_id, planned_workout_id, delta_v, delta_i, label, reasons JSON, segment_stats JSON).
- [ ] Create DB migration: `model_snapshots` (user_id, timestamp, params JSON, model_version, error_metrics JSON).
- [ ] Create DB migration: `daily_impulses` (user_id, date, impulse_value, source_breakdown JSON).
- [ ] Add RLS policies for all tables (read/write only own rows).
- [ ] Add indexes for common queries (user_id + start_time, user_id + date).

## Phase 3 — Backend service skeleton (FastAPI on Cloud Run)

### 3.1 Service skeleton
- [ ] Initialize BE project skeleton and implement `GET /health` returning version + uptime.
- [ ] Add structured logging (request_id, endpoint, duration_ms, user_id when available).
- [ ] Add consistent JSON error shape (code, message, details).
- [ ] Configure CORS for FE origins and document in `docs/security/cors.md`.

### 3.2 Auth wiring (Supabase JWT)
- [ ] Decide FE token transport (Authorization: Bearer) and document in `docs/security/auth.md`.
- [ ] Implement JWT validation middleware and attach user_id to request context.
- [ ] Implement `GET /me` returning user id + settings stub to validate auth end-to-end.

### 3.3 Deployment skeleton
- [ ] Create Dockerfile for BE and document build/run commands in `docs/deploy/cloud-run.md`.
- [ ] Deploy BE to Cloud Run and verify `/health` from the public URL (or secured, per decision).
- [ ] Implement `GET /version` returning build/version string.

## Phase 4 — Frontend skeleton (Angular on Cloudflare Pages)

### 4.1 FE baseline
- [ ] Initialize Angular app with routing and a minimal shell (nav + content).
- [ ] Implement environment config (API base URL) and document in `docs/architecture/config.md`.
- [ ] Create an API service wrapper for BE calls with centralized error handling.

### 4.2 Auth UI & session
- [ ] Implement login/signup with Supabase Auth.
- [ ] Add route guard for authenticated routes.
- [ ] Implement settings page stub (FTP + units) with save + load.
- [ ] Add a status widget that calls `GET /health` or `GET /version` and shows connectivity.

### 4.3 Deploy FE
- [ ] Deploy FE to Cloudflare Pages and verify login + BE calls work.

## Phase 5 — FIT ingestion pipeline (Upload → Store → Parse → Persist)

### 5.1 R2 integration (BE)
- [ ] Decide and document R2 object key format in `docs/storage/r2.md` (e.g., `{user_id}/{yyyy-mm}/{activity_id}.fit`).
- [ ] Implement R2 client wrapper with put/get/head/delete.
- [ ] Add unit tests for pure key-generation logic.

### 5.2 Upload endpoint (BE)
- [ ] Implement `POST /activities/upload` (multipart FIT + optional metadata: source, external_id).
- [ ] Compute content hash for dedupe and persist activity row with `processing_status=stored`.
- [ ] Upload raw FIT to R2 and store r2_key.
- [ ] Ensure idempotency: if same user uploads same content_hash/start_time, return existing activity_id.
- [ ] Return response: activity_id + status + next steps (polling URL or inline result).

### 5.3 FIT parsing module (BE)
- [ ] Create parsing service boundary and document in `docs/backend/fit-parsing.md`.
- [ ] Parse activity metadata: start_time, elapsed_time, sport/type; detect missing/invalid timestamps.
- [ ] Extract time series streams: timestamp, power, HR, cadence, speed if present.
- [ ] Define and document pause handling (moving vs elapsed) in `docs/metrics/time-handling.md`.
- [ ] Implement deterministic downsampling for charting and document in `docs/metrics/downsampling.md`.
- [ ] Add parser tests using representative FIT fixtures (at least 2 device types).

### 5.4 Persist parsed output (DB)
- [ ] Extend `activities` to store summary metrics (duration, avg/max power, avg/max HR) and a stable metrics JSON (document schema).
- [ ] Decide time series storage approach (separate table vs JSON) and document in `docs/data/time-series-storage.md`.
- [ ] Implement DB write of summary + downsampled series after parse; set `processing_status=parsed`.
- [ ] Implement failure capture: set `processing_status=failed` and `processing_error` on exceptions.
- [ ] Implement `GET /activities/{id}` returning summary + series availability + status.

### 5.5 FE upload UX
- [ ] Create FE upload page with file picker and progress indicator.
- [ ] After upload, navigate to activity detail page and show parsed summary.
- [ ] Show friendly error UI for failed parses (include a “reprocess” button later).
- [ ] Create FE activity list page (latest 50) with date, duration, impulse placeholder, and status badge.

## Phase 6 — Metric normalization (Impulse, TSS-like, TRIMP fallback)

### 6.1 Settings prerequisites
- [ ] Require FTP for power-based impulse; FE prompts user to set FTP if missing.
- [ ] Add optional HR inputs for TRIMP (resting/max HR or zone thresholds) and document in `docs/metrics/trimp.md`.

### 6.2 Power-based impulse (TSS-like)
- [ ] Define the exact formula used (NP/IF/TSS-like) in `docs/metrics/tss.md` including edge cases.
- [ ] Implement NP (or chosen proxy) calculation and validate on synthetic data.
- [ ] Compute IF and impulse for each activity with power data.
- [ ] Add unit tests for metric functions (synthetic time series cases).

### 6.3 HR fallback (optional MVP)
- [ ] Implement TRIMP computation per chosen method and unit tests.
- [ ] Define precedence rules (power primary; HR used only if power missing).

### 6.4 Persist impulse & daily rollups
- [ ] Store per-activity impulse in `activities` (or metrics JSON) and backfill on parse.
- [ ] Implement daily aggregation for a date range (sum impulses per day) and write to `daily_impulses`.
- [ ] Implement `POST /metrics/recompute` to recompute metrics + daily impulses for a date range.
- [ ] Implement `GET /impulses/daily?from=&to=` endpoint returning daily series for charts.

## Phase 7 — Planned workout representation & editor

### 7.1 Schema
- [ ] Define planned workout JSON schema in `docs/plans/workout-schema.md` (segments: type, duration_s, target_watts or target_pct_ftp, tolerance).
- [ ] Implement server-side validation for schema on create/update and return pinpointed validation errors.

### 7.2 BE endpoints
- [ ] Implement `POST /planned-workouts` to create.
- [ ] Implement `GET /planned-workouts` (paginated) to list.
- [ ] Implement `GET /planned-workouts/{id}` to fetch.
- [ ] Implement `PATCH /planned-workouts/{id}` to update name/structure/date.
- [ ] Implement `DELETE /planned-workouts/{id}` to delete.

### 7.3 FE editor
- [ ] Implement planned workout list page with create button.
- [ ] Implement workout editor: add/remove/reorder segments.
- [ ] Show %FTP and watts preview using user FTP.
- [ ] Render planned workout as a step chart (client-generated chart points).
- [ ] Allow setting planned date (calendar input).

## Phase 8 — Matching actual activities to planned workouts

### 8.1 Spec
- [ ] Document matching rules and confidence scoring in `docs/analysis/matching.md`.
- [ ] Define when the system auto-links vs flags “needs confirmation”.

### 8.2 BE matching
- [ ] Implement `POST /assignments/auto-match` for a date range.
- [ ] Implement `POST /assignments/{id}/confirm` to manually set activity_id for an assignment.
- [ ] Ensure reruns are idempotent and update confidence, not duplicate rows.

### 8.3 FE matching UI
- [ ] Create “Unmatched” view listing activities and suggested planned matches.
- [ ] Add confirm/override actions and show confidence.
- [ ] Add a simple calendar view: planned workouts by date with completion state.

## Phase 9 — Session Analyzer (ΔV, ΔI, labels, reasons)

### 9.1 Analyzer spec
- [ ] Finalize ΔV definition in `docs/analysis/session-analyzer.md` (moving vs elapsed, tolerance).
- [ ] Finalize ΔI definition (segment weighting, compliance function, capping) and tolerances.
- [ ] Decide whether warmup/cooldown count; document.
- [ ] Define smoothing rules for outdoor data and document in `docs/analysis/smoothing.md`.

### 9.2 Segment overlay (BE)
- [ ] Convert planned segments to absolute time windows (offset start/end seconds).
- [ ] Slice actual time series by window and compute per-segment stats.
- [ ] Produce chart-ready overlay series (planned step points + aligned actual points).
- [ ] Persist per-segment stats in `session_analyses.segment_stats`.

### 9.3 ΔV calculation (BE)
- [ ] Compute planned total duration from segments.
- [ ] Compute actual completed duration per time-handling decision.
- [ ] Calculate ΔV and add unit tests for early stop, extra time, and pauses.

### 9.4 ΔI calculation (BE)
- [ ] Implement ΔI scoring function and weighting.
- [ ] Add unit tests for under/over target, variable power, and missing power.
- [ ] Define fallback behavior when power missing (unknown vs HR proxy) and implement.

### 9.5 Classification (BE)
- [ ] Define thresholds for Compliant/Bailout/Grinder/Crusher/Unstructured in analyzer spec.
- [ ] Implement classification and attach reason codes (failed segments, shortfall %).
- [ ] Persist analysis record and ensure idempotent updates on re-run.

### 9.6 Analysis API (BE)
- [ ] Implement `POST /analysis/run` (activity_id; uses assignment or explicit planned_workout_id).
- [ ] Implement `GET /analysis/{activity_id}` returning stored analysis + overlay data.

### 9.7 FE analysis UI
- [ ] Activity detail: show label + ΔV/ΔI + human-readable explanation.
- [ ] Render planned vs actual overlay chart.
- [ ] Render per-segment breakdown table (target, actual, compliance%).
- [ ] Add a “re-run analysis” button with success/error feedback.

## Phase 10 — Daily impulse series & baseline Fitness/Fatigue curves

### 10.1 Daily impulse lifecycle
- [ ] Update daily impulses when a new activity is parsed (single-day upsert).
- [ ] Implement `GET /impulses/daily` (already planned) to power charts.

### 10.2 Baseline curves (fixed params)
- [ ] Implement baseline impulse-response curves using fixed decay constants (config) to validate end-to-end.
- [ ] Implement `GET /model/curves?from=&to=` returning Fitness/Fatigue/Freshness points.
- [ ] Add unit tests for curve computation invariants (non-negativity, continuity under stable inputs).

### 10.3 FE curves page
- [ ] Implement charts for Fitness/Fatigue/Freshness with date range controls.
- [ ] Add hover tooltips and legend.

## Phase 11 — Digital Twin learning loop (parameter fitting + snapshots)

### 11.1 Parameter and snapshot schema
- [ ] Define model parameters, bounds, and serialization in `docs/model/parameters.md`.
- [ ] Define model_version string and compatibility rules.
- [ ] Implement DB read/write helpers for `model_snapshots`.

### 11.2 Ground-truth observations
- [ ] Implement observation extractor per `docs/model/ground-truth.md`.
- [ ] Implement `GET /model/observations?from=&to=` for debugging.
- [ ] Add unit tests for observation extraction on synthetic inputs.

### 11.3 Batch fitting
- [ ] Implement fitting routine returning best-fit parameters from impulses + observations.
- [ ] Compute fit-quality metrics (documented) and persist in snapshot.
- [ ] Implement `POST /model/refit` for current user and optional date range.

### 11.4 Update policy
- [ ] Document refit trigger policy in `docs/model/update-policy.md`.
- [ ] Implement basic protection against repeated refits (cooldown or idempotency key).

### 11.5 FE model quality
- [ ] Add “Model” page showing latest snapshot, parameters, and fit metrics.
- [ ] Add “Refit model” button with explanation and progress state.

## Phase 12 — What-if simulator (7–14 day block)

### 12.1 Contract
- [ ] Define simulation request/response schema in `docs/sim/sim-request.md`.
- [ ] Decide whether simulation inputs are planned workouts (structure) or explicit impulses; document.

### 12.2 BE simulation
- [ ] Implement `POST /simulate` using latest model parameters + proposed impulses.
- [ ] Implement simple risk heuristics and return warnings list.
- [ ] Add unit tests for deterministic simulation cases.

### 12.3 FE simulation UI
- [ ] Build simulation page with 7/14-day presets and block builder (select workouts or enter impulses).
- [ ] Render simulated curves and display warnings with day-level context.

## Phase 13 — Adaptive suggestion (bounded MVP)

### 13.1 Spec
- [ ] Document suggestion rules per label (Grinder/Bailout/Crusher/Compliant) in `docs/adapt/suggestions.md`.
- [ ] Define suggestion payload format (title, rationale, suggested_change).

### 13.2 BE suggestions
- [ ] Implement suggestion generator based on latest analysis + upcoming planned session.
- [ ] Implement `GET /adapt/suggestion?activity_id=` returning suggestion payload.

### 13.3 FE display
- [ ] Show suggestion panel on activity analysis page using reason codes for explainability.
- [ ] Add “Apply suggestion” flow that opens workout editor with suggested changes prefilled (user confirms save).

## Phase 14 — Observability & reliability

### 14.1 Status & retries
- [ ] Implement `POST /activities/{id}/reprocess` to retry parsing and metrics (idempotent).
- [ ] FE: add “Retry” action for failed activities.

### 14.2 Logging discipline
- [ ] Standardize log fields and document in `docs/observability/logging.md`.
- [ ] Add request timing and log slow requests above threshold.

### 14.3 Self-check tooling
- [ ] Implement `GET /admin/self-check` (auth-protected) to detect missing metrics, orphaned series, missing R2 objects.
- [ ] Document remediation steps in `docs/runbook.md`.

## Phase 15 — Security & privacy

### 15.1 GPS/location handling
- [ ] Decide GPS retention policy and document in `docs/privacy/gps.md`.
- [ ] FE: surface a privacy notice in settings (what is stored where).

### 15.2 RLS verification
- [ ] Write manual RLS test checklist in `docs/security/rls-tests.md`.
- [ ] Add BE integration tests ensuring cross-user data access is denied for key endpoints.

### 15.3 Deletion
- [ ] Implement `DELETE /activities/{id}` to remove DB rows + R2 object.
- [ ] FE: add delete UI with confirmation and success/error states.

## Phase 16 — UX polish & quality

- [ ] Add chart tooltips and consistent legends across pages.
- [ ] Add loading skeletons and empty states (no activities/plans/unmatched).
- [ ] Ensure every analyzer label has a one-sentence explanation and a “details” section referencing segment failures.
- [ ] Add warning when changing FTP impacts metrics; provide “Recompute” button (calls `POST /metrics/recompute`).

## Phase 17 — Documentation & handoff readiness

- [ ] Write `docs/architecture/overview.md` with a diagram of FE/BE/DB/R2 and data flow.
- [ ] Write `docs/api/api-contracts.md` with endpoint list, request/response examples, and error shapes.
- [ ] Write `docs/data/schemas.md` describing tables, key fields, relationships, indexes.
- [ ] Add worked examples to `docs/analysis/session-analyzer.md` (Bailout/Grinder/Crusher/Compliant).
- [ ] Write `docs/model/model.md` explaining impulse-response model, parameters, fitting, and update policy.
- [ ] Write `docs/sim/simulator.md` describing simulation behavior and limitations.
- [ ] Write `docs/runbook.md` (deploy/redeploy, log inspection, reprocess, recompute, refit).

## Phase 18 — Phase 2+ (parked; not MVP)

### 18.1 External sync (Intervals.icu / Strava)
- [ ] Implement OAuth/connect flow and store tokens securely.
- [ ] Implement scheduled sync to pull new activities daily.
- [ ] Implement webhook receiver for near-real-time updates (provider permitting).

### 18.2 Real-time workout execution (Web Bluetooth)
- [ ] FE: proof-of-concept connect to HR monitor via Web Bluetooth.
- [ ] FE: proof-of-concept read smart trainer power and (if supported) control resistance.
- [ ] Define workout execution UX and safety overrides.

### 18.3 Advanced planning
- [ ] Add multi-week plan templates and progression rules.
- [ ] Add deload detection/suggestions based on modeled fatigue + compliance history.
- [ ] Add event goals (A/B/C priorities) and planning toward events.
