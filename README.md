# KISS VMP
## KISS Vocational Management Portal

An institutional, ERP-style web portal for managing the vocational training lifecycle at KISS, built for three roles — Student, Teacher, and Admin.

---

## Overview

KISS VMP is an institutional ERP-style web portal that manages the vocational training lifecycle end to end: enrollment, batch/section allocation, attendance, grading and progress, mentoring, grievances, leave, feedback, and certificates.

The system serves three roles, each with its own role-specific Home and set of modules:

- **Student** — enrolls in courses, tracks attendance and progress, downloads certificates, raises grievances, submits feedback, and applies for leave.
- **Teacher** — manages assigned courses (attendance and gradebook), mentors assigned students, raises their own grievances, and applies for their own leave.
- **Admin** — configures courses and allocation, manages student/teacher records, oversees all grievances, configures feedback, generates reports, and applies for their own leave.

> **Implementation status**: requirements and UI/UX are defined and frozen for V1. Backend and frontend implementation have not started yet — see [Development Status](#development-status) below.

---

## Objectives

- Replace manual/fragmented tracking of enrollment, attendance, grading, and certification with a single institutional portal.
- Give each role a focused, no-frills workspace — information-dense where it needs to be, without analytics-dashboard clutter.
- Keep V1 scoped to a workable, end-to-end MVP rather than a visually experimental product.
- Keep documentation, UI/UX, and (eventually) implementation in lockstep, with a single source of truth for each concern.

---

## User Roles

| Role | Purpose |
|---|---|
| **Student** | Enrolls in courses, tracks attendance/progress, downloads certificates, raises grievances, gives feedback, applies for leave |
| **Teacher** | Manages assigned courses (attendance, gradebook), mentors assigned students, raises own grievances, applies for own leave |
| **Admin** | Configures courses/allocation, manages student & teacher records, oversees grievances, configures feedback, generates reports, applies for own leave |

---

## Core Features

### Student
- My Information
- Enrollment
- Attendance
- Progress *(Grades, Course Progress)*
- Certificates
- Grievance
- Feedback
- Leave Application

### Teacher
- Manage Course *(select course → batch/section → Attendance or Gradebook)*
- Mentoring
  - My Students
  - Requests
    - Grievances / Complaints
    - Leave Requests
- Raise Grievance *(own)*
- Apply Leave *(own)*

### Admin
- Course Management
- Enrollment
- Allocation
- Students
- Teachers
- Grievance Management
  - Grievances
  - Students
  - Teachers
  - Add Grievance *(own)*
- Feedback
- Reports
- Apply Leave *(own)*

Terminology above matches [`docs/UI-UX/UI_UX_SPEC.md`](docs/UI-UX/UI_UX_SPEC.md) exactly — including that **leave and grievances are always separate flows**, and that Admin's "Grievances / Students / Teachers" are three views of one grievance dataset, not three separate systems.

---

## System Architecture

### Current / Planned Implementation
Nothing has been implemented yet. The frontend (`/frontend`) and backend (`/backend`) directories do not exist in the repository yet — this phase of the project is documentation and architecture presentation only.

### Architectural Target

**Frontend**: React · React Router · Tailwind CSS
**Backend**: Java · Spring Boot · Spring Security · JWT · Maven
**Database**: PostgreSQL · Supabase
**Future**: Docker · CI/CD

See [High-Level Architecture](docs/Architecture/Diagrams/02-High-Level-Architecture.md) for the layering, and [System Context](docs/Architecture/Diagrams/01-System-Context.md) for the role-to-system view.

---

## Core System Flow

```
Login
 ↓
Backend authentication / role detection
 ↓
Role-specific Home
 ↓
Module
 ↓
Sub-module
 ↓
Feature
 ↓
Backend API
 ↓
Database
```

There is **no manual role selection** during login — the role is detected by the backend, and the user lands directly on their role-specific Home. See [Core Navigation Flow](docs/Architecture/Diagrams/04-Core-Navigation-Flow.md).

---

## Major Business Flows

| Flow | Diagram |
|---|---|
| 1. Enrollment → Allocation | [05-Enrollment-Allocation-Flow.md](docs/Architecture/Diagrams/05-Enrollment-Allocation-Flow.md) |
| 2. Academic workflow (Course → Attendance/Gradebook → Progress → Certificate) | [06-Academic-Flow.md](docs/Architecture/Diagrams/06-Academic-Flow.md) |
| 3. Mentoring | [07-Mentoring-Flow.md](docs/Architecture/Diagrams/07-Mentoring-Flow.md) |
| 4. Grievance | [08-Grievance-Flow.md](docs/Architecture/Diagrams/08-Grievance-Flow.md) |
| 5. Leave | [09-Leave-Flow.md](docs/Architecture/Diagrams/09-Leave-Flow.md) |
| 6. Feedback | Student submits via a feedback form; Admin configures forms and reviews records — see [UI/UX Spec §22](docs/UI-UX/UI_UX_SPEC.md#22-admin-workflows) |
| 7. Certificate generation | [06-Academic-Flow.md](docs/Architecture/Diagrams/06-Academic-Flow.md), [ADR-005](docs/Architecture/ADR-005-CertificateGeneration.md) |

Only workflows already established in the repository's documentation are described here — see [Decisions Needed Before Database Design](#decisions-needed-before-database-design) for what's still open.

---

## Technology Stack

| Category | Technology |
|---|---|
| Frontend | React, React Router, Tailwind CSS |
| Backend | Java, Spring Boot |
| Authentication | Spring Security, JWT |
| Database | PostgreSQL (via Supabase) |
| Build | Maven |
| Development | — |
| Future Infrastructure | Docker, CI/CD |

---

## Repository Structure

The structure below reflects the **actual current repository**, not an aspirational target. `frontend/` and `backend/` are planned but do not exist yet.

```
KISS-VMP/
├── docs/
│   ├── SRS/            Software Requirements Specification
│   ├── SDD/             System Design Document
│   ├── API/              API reference
│   ├── Testing/        Test plan & test cases
│   ├── Architecture/    ADRs + system/business-flow diagrams (this task's output)
│   │   └── Diagrams/
│   ├── UML/              Existing schema-level UML/ER diagrams
│   └── UI-UX/
│       ├── README.md       Entry point — prototype vs. production
│       ├── UI_UX_SPEC.md   Source of truth for frontend UI/UX
│       └── screenshots/    Prototype screenshots (to be added)
├── README.md            (this file)
└── DOCUMENTATION-SUMMARY.md
```

**Planned, not yet created**: `frontend/`, `backend/`, `docs/ER/` (a dedicated, finalized database ER diagram folder — produced during the next phase), `LICENSE`, `.gitignore`.

---

## UI/UX Reference

```
Vercel / v0 Prototype       =  high-fidelity interactive UI/UX reference
React Frontend (/frontend)  =  actual production implementation
```

The prototype shows how the portal should look and behave. **It is not the production frontend**, and its code must not be copied into `/frontend`.

- **Prototype**: https://v0.app/adhalakhadhanesh-1826/chat/kiss-vocational-management-portal-gpHIk5jPcee
- **UI/UX entry point**: [`docs/UI-UX/README.md`](docs/UI-UX/README.md)
- **UI/UX source of truth**: [`docs/UI-UX/UI_UX_SPEC.md`](docs/UI-UX/UI_UX_SPEC.md) — where the prototype and the spec disagree, **the spec wins**.

---

## Documentation

| Documentation | Location | Purpose |
|---|---|---|
| SRS | [`docs/SRS/`](docs/SRS/SRS-Index.md) | Functional & non-functional requirements, use cases |
| SDD | [`docs/SDD/`](docs/SDD/SDD-Index.md) | System design, database schema, module design |
| API | [`docs/API/`](docs/API/API-Index.md) | REST endpoint reference |
| Testing | [`docs/Testing/`](docs/Testing/Testing-Index.md) | Test plan and test cases |
| Architecture Decisions | [`docs/Architecture/`](docs/Architecture/) | ADRs — why things were decided a certain way |
| Architecture Diagrams | [`docs/Architecture/Diagrams/`](docs/Architecture/Diagrams/README.md) | System context, business flows, navigation, domain model |
| UML (existing, schema-level) | [`docs/UML/`](docs/UML/) | Earlier use-case/class/ER diagrams tied to the current database design |
| UI/UX | [`docs/UI-UX/`](docs/UI-UX/README.md) | Frontend source of truth and prototype reference |

> **Note on scope**: `docs/SRS/`, `docs/SDD/`, `docs/API/`, `docs/Testing/`, and `docs/UML/` predate the UI/UX specification and reflect an earlier iteration of this project's requirements. They are accurate for the backend concepts they cover (authentication, enrollment, allocation, attendance, grading, certificates) but **do not yet include Mentoring or Leave Applications**, which the UI/UX spec introduces as frozen V1 features. See [Decisions Needed Before Database Design](#decisions-needed-before-database-design).

---

## V1 Scope

Per [`UI_UX_SPEC.md` §24](docs/UI-UX/UI_UX_SPEC.md#24-v1--v2-boundaries), V1 is frozen and includes:

- Role-based login (no manual role selection)
- Student, Teacher, and Admin Home
- Module and sub-module navigation
- Enrollment
- Attendance
- Progress / Grades
- Certificates
- Grievances (Student, Teacher, Admin)
- Leave (Student Leave Application, Teacher/Admin Apply Leave, Teacher mentee Leave Requests)
- Teacher Mentoring
- Feedback
- Teacher attendance management
- Teacher grade management
- Admin management workflows (including Allocation and Reports)
- Pagination
- Loading, empty, error, and success states
- Basic caching
- Desktop-first UI

---

## V2 / Future Scope

Explicitly deferred per the UI/UX spec:

- Search
- Advanced filters
- Advanced table navigation
- Better mobile optimisation
- Additional convenience features
- Other enhancements identified after V1

Anything not listed as V1 or V2 above is **not specified** and needs a team decision before implementation — see the next section.

---

## Decisions Needed Before Database Design

These are open items the UI/UX spec explicitly flags as **Not specified**, plus one structural gap found while writing this README. None of them have been guessed at anywhere in this repository's diagrams or documentation:

1. Student Leave Application — position in Student navigation, form fields, status display
2. Student-facing view of mentoring, if any
3. Teacher Mentoring — actions on grievances and leave requests, statuses, table columns
4. Teacher and Admin Apply Leave — form fields and approval flow
5. Admin actions on grievances (respond, status, close) and grievance statuses
6. Placement of admin configuration (seats, batches, teachers, grading schemes, feedback forms, sections)
7. Where former Teacher modules "Student Details" and "Assigned Sections" now live
8. Field lists and validation for all forms
9. Exact color values, font sizes, table page size, cache lifetime
10. Profile entry in the sidebar per role
11. Confirmation dialogs for teacher request handling and admin actions beyond those already listed
12. **Mentoring and Leave Applications have no corresponding entity in the existing `docs/SDD/02-Database-Design.md`** — these are new V1-frozen features introduced by the UI/UX spec after the backend design docs were written. They need to be added to the data model during the upcoming database design phase (see [Domain Model](docs/Architecture/Diagrams/10-Domain-Model.md)).

---

## Development Status

| Area | Status |
|---|---|
| UI/UX Specification | Frozen (V1) |
| UI/UX Prototype | Reference available (Vercel/v0) |
| Documentation | In progress — this task covers README + architecture diagrams |
| Database Design | Next phase |
| Supabase | Upcoming |
| Spring Boot Backend | Upcoming |
| React Frontend | Upcoming |
| Integration | Upcoming |
| Testing | Upcoming |

---

## Development Roadmap

1. **Requirements / Documentation** *(in progress)*
2. **UI/UX** *(frozen)*
3. **Database Design** *(next)*
4. **Backend**
5. **Frontend**
6. **Integration**
7. **Testing**
8. **Deployment**

---

## Getting Started

Setup instructions are not available yet — `/frontend` and `/backend` have not been created. Commands, environment variables, and credentials will be added here once implementation begins; none are fabricated in this document.

---

## Development Guidelines

- Keep the frontend aligned with [`docs/UI-UX/UI_UX_SPEC.md`](docs/UI-UX/UI_UX_SPEC.md) — where the spec and the prototype disagree, the spec wins.
- Keep backend API contracts documented as they're built.
- Do not invent unspecified requirements — anything marked **Not specified** needs a team decision first.
- Use meaningful Git commits.
- Keep documentation synchronized with architecture changes.
- Do not commit secrets.
- The production frontend (`/frontend`) is implemented independently — the Vercel/v0 prototype is a reference only and its code is never copied in.

---

## License

No license file currently exists in this repository. One will be added here once the team decides on licensing terms — none is assumed or invented in this document.