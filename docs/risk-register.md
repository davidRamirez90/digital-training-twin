# Risk Register

| Risk | Impact | Likelihood | Mitigation |
| --- | --- | --- | --- |
| FIT file variability across devices | Parsing failures, inconsistent fields | Medium | Use robust parser, add fixtures for multiple devices, capture errors with retries. |
| Time-series storage cost | Increased DB/storage costs | Medium | Downsample for charting, evaluate separate storage table vs JSON, archive raw files in R2. |
| Data access/security | Unauthorized access to user data | Low | Enforce Supabase RLS, JWT validation, audit endpoints. |
| Missing power data | Incomplete impulse calculation | Medium | Implement HR TRIMP fallback, prompt user for HR settings. |
| Inaccurate model fitting | Poor recommendations | Medium | Use compliance + rolling best proxy, validate on synthetic/real data, monitor fit metrics. |
| Dependency on external services | Outages affect uploads or auth | Low | Document failover procedures, provide status messaging in FE. |
