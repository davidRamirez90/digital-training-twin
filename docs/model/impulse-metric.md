# Canonical Impulse Metric (MVP)

## Decision
Use a **power-based TSS-like impulse** as the canonical metric for MVP.

## Rationale
- Power is the most reliable, device-consistent signal for cycling load.
- TSS-like metrics are well-understood and map cleanly to impulse-response models.
- Provides a clear fallback path when power is unavailable (HR TRIMP).

## Fallback
If power data is missing or invalid, use a **heart-rate-based TRIMP** metric as a fallback. Power remains the primary signal whenever available.

## Notes
Exact formula details and edge cases are documented in `docs/metrics/tss.md` and `docs/metrics/trimp.md` during Phase 6.
