# Ground-Truth Signal for Model Fitting

## Decision
Use **compliance outcomes** augmented by a **rolling best-power proxy** as the ground-truth signal for model fitting.

## Components
1. **Compliance Outcomes**
   - Derived from session analyzer labels (Compliant, Grinder, Bailout, Crusher).
   - Encodes whether the athlete could execute planned intensity/volume.

2. **Rolling Best-Power Proxy**
   - Maintain rolling bests (e.g., 5s/1m/5m/20m) as a proxy for performance trends.
   - Used to anchor fit quality when planned sessions are sparse.

## Justification
- Compliance is directly tied to how the athlete handled prescribed work.
- Rolling bests provide a lightweight performance signal without requiring full testing protocols.

## Notes
Exact extraction logic and aggregation windows are defined in Phase 11 (`docs/model/ground-truth.md` will be updated with implementation details).
