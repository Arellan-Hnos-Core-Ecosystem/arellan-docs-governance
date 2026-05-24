# Technical Debt Register & Lifecycle Management

## 1. Tracked Shortcuts for the MVP Phase
- **Authentication:** Short-term usage of centralized database sessions before migrating to a dedicated decentralized identity provider (e.g., Keycloak or AWS Cognito).
- **Offline Synch:** Biometric terminals assume a resilient fallback connection. Advanced database replication state engines are deferred to Phase 3.
- **Local Logs storage:** Non-critical tracing logs remain locally inside individual containers before routing to the dedicated Grafana Loki / ELK ecosystem.