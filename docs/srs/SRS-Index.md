# Software Requirements Specification - Index

## Document Overview

This index provides navigation to all Software Requirements Specification (SRS) documents for the KISS Vocational Training Management Portal.

## Table of Contents

### 1. Foundational Documents

| Document | Purpose | Read Time |
|----------|---------|-----------|
| [Introduction](01-Introduction.md) | Project overview, document scope, and structure | 5 min |
| [Problem Statement](02-Problem-Statement.md) | Business need and challenges addressed | 10 min |
| [Scope](03-Scope.md) | What system will and will not do | 10 min |
| [Stakeholders](04-Stakeholders.md) | Identification and engagement of all parties | 8 min |

**Start Here**: Begin with Introduction for document orientation.

---

### 2. Detailed Requirements

| Document | Purpose | Read Time |
|----------|---------|-----------|
| [Functional Requirements](05-Functional-Requirements.md) | Complete list of system features and capabilities | 25 min |
| [Non-Functional Requirements](06-Non-Functional-Requirements.md) | Performance, security, scalability, compliance | 15 min |
| [Use Cases](07-Use-Cases.md) | Actor scenarios and workflows | 15 min |

**For Developers**: Read Functional Requirements first, then Non-Functional Requirements.

---

### 3. Project Assumptions & Constraints

| Document | Purpose | Read Time |
|----------|---------|-----------|
| [Assumptions](08-Assumptions.md) | What we assume to be true | 10 min |
| [Constraints](09-Constraints.md) | Limitations and boundaries | 12 min |

**For Project Planning**: Review Assumptions and Constraints early.

---

## Document Relationships

```
Introduction
    ↓
Problem Statement + Stakeholders
    ↓
Scope
    ↓
Functional Requirements
    ├→ Use Cases
    ├→ Non-Functional Requirements
    └→ Assumptions & Constraints
```

## Reading Recommendations

### For Project Managers
1. Problem Statement
2. Scope
3. Stakeholders
4. Constraints
5. Assumptions

**Time**: ~45 minutes

### For Developers

1. Introduction
2. Functional Requirements
3. Non-Functional Requirements
4. Use Cases
5. System Architecture (in SDD)

**Time**: ~60 minutes

### For QA/Testing Team

1. Scope
2. Functional Requirements
3. Non-Functional Requirements
4. Use Cases
5. Testing Requirements (in SDD)

**Time**: ~50 minutes

### For Architects

1. Problem Statement
2. Scope
3. Functional Requirements
4. Non-Functional Requirements
5. Constraints
6. System Architecture (in SDD)

**Time**: ~70 minutes

---

## Key Concepts

### User Roles

- **Student**: Enrolls in courses, views progress, downloads certificates
- **Teacher**: Marks attendance, manages grades, completes courses
- **Admin**: Configures semesters, runs allocations, manages system

### Core Workflows

1. **Enrollment**: Student → Course → Enrollment Window
2. **Allocation**: Enrollment Closes → Random Shuffle → Batch Assignment
3. **Grading**: Attendance → Mark Entry → Publish → Certificate Generation

### Technology Stack

- **Frontend**: React
- **Backend**: Spring Boot + Spring Security
- **Database**: PostgreSQL (Supabase)
- **Authentication**: JWT
- **Build**: Maven

---

## Requirement Statistics

| Category | Count | Priority |
|----------|-------|----------|
| Functional Requirements | 34 | 28 Critical, 6 High |
| Non-Functional Requirements | 32 | 19 Critical, 13 High |
| Use Cases | 18 | 14 Critical, 4 High |
| Assumptions | 20+ | Tracked in document |
| Constraints | 23 | Tracked in document |

---

## Cross-Reference Guide

### By Module

**Authentication**
- Requirements: [FR-AUTH-001 to FR-AUTH-004](05-Functional-Requirements.md#1-authentication--authorization)
- Use Cases: [UC-STU-001](07-Use-Cases.md#uc-stu-001-log-in-to-system)
- Security: [NFR-SEC-001 to NFR-SEC-005](06-Non-Functional-Requirements.md#2-security-requirements)

**Student Features**
- Requirements: [FR-STU-001 to FR-STU-007](05-Functional-Requirements.md#2-student-modules)
- Use Cases: [UC-STU-002 to UC-STU-006](07-Use-Cases.md#student-use-cases)

**Teacher Features**
- Requirements: [FR-TEA-001 to FR-TEA-005](05-Functional-Requirements.md#3-teacher-modules)
- Use Cases: [UC-TEA-001 to UC-TEA-005](07-Use-Cases.md#teacher-use-cases)

**Admin Features**
- Requirements: [FR-ADM-001 to FR-ADM-013](05-Functional-Requirements.md#4-admin-modules)
- Use Cases: [UC-ADM-001 to UC-ADM-007](07-Use-Cases.md#admin-use-cases)

**System Workflows**
- Workflows: [FR-SYS-001 to FR-SYS-005](05-Functional-Requirements.md#5-system-workflows)

---

## Requirement Traceability

### Critical Requirements

1. **FR-STU-002**: Course Enrollment
   - Scope: [Section on Enrollment](03-Scope.md#functional-boundaries)
   - Use Case: [UC-STU-002](07-Use-Cases.md#uc-stu-002-enroll-in-course)
   - Design: [Enrollment Design](../SDD/04-Enrollment-Design.md)

2. **FR-ADM-005**: Run Batch Allocation
   - Scope: [Section on Allocation](03-Scope.md#what-the-system-will-do)
   - Use Case: [UC-ADM-005](07-Use-Cases.md#uc-adm-005-run-batch-allocation)
   - Design: [Batch Allocation](../SDD/05-Batch-Allocation.md)

3. **FR-TEA-003**: Enter Grades
   - Requirements: [FR-TEA-003](05-Functional-Requirements.md#fr-tea-003-gradebook-management)
   - Use Case: [UC-TEA-003](07-Use-Cases.md#uc-tea-003-enter-grades)
   - Design: [Gradebook Workflow](../SDD/06-Teacher-Workspace.md)

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2026 | Initial SRS creation | Architecture Team |

---

## Document Maintenance

- **Last Updated**: [Date]
- **Next Review**: [Date]
- **Responsible Team**: Requirements Team
- **Approval Status**: [ ] Approved [ ] Pending Review

---

## Quick Navigation

- [Next: Introduction](01-Introduction.md)
- [Previous: Back to Docs](../)
- [Related: System Design Document Index](../SDD/SDD-Index.md)
- [Related: API Documentation Index](../API/API-Index.md)

---

## Glossary of Abbreviations

| Abbreviation | Meaning |
|--------------|---------|
| SRS | Software Requirements Specification |
| SDD | System Design Document |
| RBAC | Role-Based Access Control |
| JWT | JSON Web Tokens |
| ERP | Enterprise Resource Planning |
| LMS | Learning Management System |
| KISS | KIIT Swayam Siksha (Vocational Program) |
| FR | Functional Requirement |
| NFR | Non-Functional Requirement |
| UC | Use Case |

---

## Related Documentation

- [System Design Document (SDD)](../SDD/SDD-Index.md)
- [API Documentation](../API/API-Index.md)
- [Testing Requirements](../Testing/Testing-Index.md)
- [Architecture Decision Records](../Architecture/)
- [UML Diagrams](../UML/)
