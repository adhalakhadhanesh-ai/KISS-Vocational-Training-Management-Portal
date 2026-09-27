# UML — Component Diagram

## Overview

Mermaid rendering of the layered component structure described in [System Architecture §3–4](../SDD/01-System-Architecture.md#3-system-architecture-diagram). Reflects the same layers, controllers, and cross-cutting concerns — no new components introduced.

```mermaid
flowchart TB
    subgraph UI["User Interface Layer — React SPA"]
        StudentDash[Student Dashboard]
        TeacherWork[Teacher Workspace]
        AdminDash[Admin Dashboard]
        AuthUI[Authentication UI]
    end

    subgraph API["API Layer — Spring Boot (multiple instances)"]
        direction TB
        subgraph Controllers
            AuthCtrl[Auth Controller]
            StudentCtrl[Student Controller]
            TeacherCtrl[Teacher Controller]
            AdminCtrl[Admin Controller]
            CourseCtrl[Course Controller]
            AllocCtrl[Allocation Controller]
        end
        subgraph CrossCutting["Cross-Cutting Concerns"]
            JWTFilter[JWT Authentication Filter]
            RBACFilter[Authorization Filter — RBAC]
            ExcHandler[Exception Handling]
            AuditLog[Logging & Audit Trail]
        end
        Service["Service Layer\n(Business Logic, Validation, Authorization)"]
        Repo["Repository Layer\n(Spring Data JPA)"]
    end

    subgraph Data["Data Layer — PostgreSQL / Supabase"]
        UserTables[(User & Profile Tables)]
        EnrollTables[(Enrollment & Batch Tables)]
        GradeTables[(Grades & Assessment Tables)]
        CourseTables[(Course & Offering Tables)]
        AttFeedTables[(Attendance & Feedback Tables)]
        CertGrievTables[(Certificates & Grievances Tables)]
    end

    StudentDash --> AuthUI
    TeacherWork --> AuthUI
    AdminDash --> AuthUI

    AuthUI -- HTTPS/TLS 1.2+ --> AuthCtrl
    StudentDash -- HTTPS/TLS 1.2+ --> StudentCtrl
    TeacherWork -- HTTPS/TLS 1.2+ --> TeacherCtrl
    AdminDash -- HTTPS/TLS 1.2+ --> AdminCtrl
    AdminDash -- HTTPS/TLS 1.2+ --> CourseCtrl
    AdminDash -- HTTPS/TLS 1.2+ --> AllocCtrl

    Controllers --> CrossCutting
    Controllers --> Service
    Service --> Repo
    Repo -- PostgreSQL JDBC --> UserTables
    Repo -- PostgreSQL JDBC --> EnrollTables
    Repo -- PostgreSQL JDBC --> GradeTables
    Repo -- PostgreSQL JDBC --> CourseTables
    Repo -- PostgreSQL JDBC --> AttFeedTables
    Repo -- PostgreSQL JDBC --> CertGrievTables
```

## Notes

- The **Cross-Cutting Concerns** block (JWT filter, RBAC filter, exception handling, audit logging) wraps every controller, per [System Architecture §3](../SDD/01-System-Architecture.md#3-system-architecture-diagram) — shown here as a shared dependency rather than duplicated per-controller for legibility.
- The Service and Repository layers are shared across all controllers (standard layered architecture), not one-per-controller.

## Related Documents

- [System Architecture](../SDD/01-System-Architecture.md)
- [Deployment Diagram](07-Deployment-Diagram.md)
