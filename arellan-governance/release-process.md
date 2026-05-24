# Production Release Protocol

## 1. Release Progression

[develop] ──> [staging (Validation)] ──> [main (Release Tagging)]


## 2. Hotpatch Exceptions
In the event of a total system failure affecting workshop vehicle billing routines, a hotpatch may branch directly from `main`, receive mutual review via verbal verification, deploy immediately, and be reverse-merged into `develop` within 12 hours.