# Glossary

## Core Training Terms
- **Fitness**: Long-term positive training adaptation signal (modeled by a slower-decay component).
- **Fatigue**: Short-term negative training load signal (modeled by a faster-decay component).
- **Freshness**: Net readiness signal, typically `Fitness - Fatigue`.
- **Impulse**: A single-session training load value used by the model (TSS-like or TRIMP).
- **TSS (Training Stress Score)**: Power-based impulse metric derived from intensity and duration.
- **TRIMP (Training Impulse)**: Heart-rate-based impulse metric derived from intensity and duration.
- **ΔV (Delta-V)**: Deviation in session volume (planned vs actual duration/volume).
- **ΔI (Delta-I)**: Deviation in session intensity (planned vs actual intensity compliance).
- **Compliance**: Degree to which the actual session followed the planned workout.

## Workout Planning
- **Planned Workout**: A prescribed session structure (segments with target intensity).
- **Segment**: A portion of a workout with a target intensity and duration.
- **Assignment**: Link between a planned workout and an actual activity.

## Data & Storage
- **Activity**: A recorded workout session (e.g., FIT file).
- **Time Series**: Sequence of timestamped measurements (power, HR, cadence).
- **Downsampling**: Reducing time series resolution for charting or storage.
