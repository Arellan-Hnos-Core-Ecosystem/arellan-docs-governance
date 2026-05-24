# ADR 001: Architecture Choice - Decentralized Domain Multi-Repo Framework

## Status
Approved

## Context
We need to design a highly resilient digital ecosystem for an automotive facility transitioning from artisanal analog methods into enterprise professionalization. Furthermore, the codebase must serve as a scalable, white-label Multi-Tenant SaaS platform ("GarageCore OS") for future commercialization by the consulting agency.

## Alternatives Considered
1. **Monolithic Architecture:** A single massive repository containing backend and all frontends.
2. **Pure Microservices Monorepo:** Separate service directories controlled under a tool like Turborepo/Lerna inside a single repository.
3. **Decoupled Domain Multi-Repo Architecture:** 13 highly targeted repositories with isolated boundaries.

## Decision
We select **Alternative 3: Decoupled Domain Multi-Repo Architecture**.


                   ┌───────────────────────┐
                   │  arellan-api-gateway  │
                   └───────────┬───────────┘
                               │ (HTTP/WSS)
                               ▼
                   ┌───────────────────────┐
                   │ arellan-backend-core  │
                   └───────────┬───────────┘
                               │
     ┌─────────────────────────┼─────────────────────────┐
     ▼                         ▼                         ▼
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│arellan-auth-serv│       │arellan-hardware-│       │arellan-data-intel│
└─────────────────┘       └─────────────────┘       └─────────────────┘


## Consequences
- **Positive:**
  - **Absolute Fault Isolation:** If the `arellan-hardware-iot` controller crashes due to serial driver errors connecting to local biometric hardware, the core transactional engine (`arellan-backend-core`) and customer web invoice payment processing remain unaffected online.
  - **SaaS Packaging Agility:** It allows the extraction of core repositories as pristine boilerplate engines for future tenants without trailing client-specific operational metadata.
- **Negative:**
  - Greater setup orchestration cost. Mitigated by continuous contract validation schemas using unified shared data contracts (`arellan-shared-types`).

  