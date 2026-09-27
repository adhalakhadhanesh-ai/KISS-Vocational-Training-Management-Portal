# UML — Deployment Diagram

## Overview

Mermaid rendering of the deployment topology in [System Architecture §11](../SDD/01-System-Architecture.md#11-deployment-architecture) and the scaling notes in [System Architecture §10](../SDD/01-System-Architecture.md#10-scalability-considerations).

```mermaid
flowchart TD
    Repo[("GitHub Repository\n(Version Control)")] --> CICD["CI/CD Pipeline\n(GitHub Actions)\nBuild → Test → Deploy"]

    CICD --> Staging
    CICD --> Production

    subgraph Staging["Staging Environment"]
        direction TB
        SFE["Frontend SPA\n(static hosting)"]
        SBE1["Backend API\ninstance"]
        SDB[("PostgreSQL\n(Supabase)")]
        SFE -- HTTPS --> SBE1
        SBE1 -- JDBC --> SDB
    end

    subgraph Production["Production Environment"]
        direction TB
        PFE["Frontend SPA\n(static hosting / CDN)"]
        LB["Load Balancer / API Gateway\n(SSL/TLS termination)"]
        PBE1["Backend API\ninstance 1"]
        PBE2["Backend API\ninstance 2"]
        PBEn["Backend API\ninstance N"]
        PDB[("PostgreSQL — Primary\n(Supabase)")]
        PDBR[("PostgreSQL — Read Replica\n(optional, reporting)")]

        PFE -- HTTPS/TLS 1.2+ --> LB
        LB --> PBE1
        LB --> PBE2
        LB --> PBEn
        PBE1 -- JDBC / HikariCP pool --> PDB
        PBE2 -- JDBC / HikariCP pool --> PDB
        PBEn -- JDBC / HikariCP pool --> PDB
        PDB -. replication .-> PDBR
    end
```

## Notes

- Backend instances are **stateless** (JWT-based sessions, no server-side session store), so the load balancer needs no session affinity — see [System Architecture §2.2](../SDD/01-System-Architecture.md#22-stateless-design) and [§10.1](../SDD/01-System-Architecture.md#101-horizontal-scaling).
- The read replica is explicitly **optional**, used only for reporting workloads per [System Architecture §10.2](../SDD/01-System-Architecture.md#102-database-scaling) — it is not required for the MVP to function.
- Both environments use the same topology at different scale; Staging typically runs a single backend instance.

## Related Documents

- [System Architecture](../SDD/01-System-Architecture.md)
- [Component Diagram](06-Component-Diagram.md)
