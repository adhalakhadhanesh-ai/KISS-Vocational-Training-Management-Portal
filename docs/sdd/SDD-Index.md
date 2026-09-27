# System Design Document - Index

## Document Overview

This index provides navigation to all System Design Document (SDD) files for the KISS Vocational Training Management Portal.

## Complete Documentation Set

### Core Design Documents

| Document | Purpose | Status |
|----------|---------|--------|
| [System Architecture](01-System-Architecture.md) | Overall system design, layers, components | Complete |
| [Database Design](02-Database-Design.md) | Complete schema, tables, relationships | Complete |
| [RBAC Design](03-RBAC-Design.md) | Role-based access control, permissions | Complete |

### Module Design Documents

| Document | Purpose | Status |
|----------|---------|--------|
| [Enrollment Design](04-Enrollment-Design.md) | Student enrollment workflow and validation | Complete |
| [Batch Allocation](05-Batch-Allocation.md) | Random allocation algorithm and process | Complete |
| [Teacher Workspace](06-Teacher-Workspace.md) | Attendance, gradebook, and course-completion module design | Complete |
| [Student Dashboard](07-Student-Dashboard.md) | Student dashboard widgets and data sources | Complete |
| [Admin Dashboard](08-Admin-Dashboard.md) | Admin command-center module design | Complete |
| [Progress Module](09-Progress-Module.md) | Attendance/grade aggregation shown to students | Complete |
| [Certificate Workflow](10-Certificate-Workflow.md) | Certificate generation and management | Complete |

---

## Design Document Map

### Recommended Reading Order

**For New Developers**:
1. [System Architecture](01-System-Architecture.md) - Understand overall design
2. [Database Design](02-Database-Design.md) - Learn data model
3. [RBAC Design](03-RBAC-Design.md) - Understand access control
4. [Enrollment Design](04-Enrollment-Design.md) - Learn key workflow
5. [Batch Allocation](05-Batch-Allocation.md) - Understand allocation algorithm
6. [Teacher Workspace](06-Teacher-Workspace.md) - Learn the teacher-side module
7. [Student Dashboard](07-Student-Dashboard.md) - Learn the student-side module
8. [Admin Dashboard](08-Admin-Dashboard.md) - Learn the admin-side module
9. [Progress Module](09-Progress-Module.md) - Understand progress aggregation
10. [Certificate Workflow](10-Certificate-Workflow.md) - Learn certificate system

**Time**: 150 minutes

---

**For Database Administrators**:
1. [Database Design](02-Database-Design.md)
2. [System Architecture](01-System-Architecture.md#data-layer)
3. Performance sections in module design documents

**Time**: 60 minutes

---

**For Security Team**:
1. [System Architecture](01-System-Architecture.md#security-architecture)
2. [RBAC Design](03-RBAC-Design.md)
3. [Database Design](02-Database-Design.md#data-integrity)
4. [Non-Functional Requirements](../SRS/06-Non-Functional-Requirements.md#2-security-requirements)

**Time**: 90 minutes

---

## Key Concepts Reference

### System Architecture Layers

```
┌─────────────────────────┐
│  Presentation (React)   │
├─────────────────────────┤
│  API Gateway/LB         │
├─────────────────────────┤
│  Application (Spring)   │
├─────────────────────────┤
│  Data (PostgreSQL)      │
└─────────────────────────┘
```

### Core Workflows

| Workflow | Document | Duration |
|----------|----------|----------|
| Enrollment | [Enrollment Design](04-Enrollment-Design.md) | 1-3 days |
| Batch Allocation | [Batch Allocation](05-Batch-Allocation.md) | 5 minutes (batch job) |
| Attendance & Grading | SDD sections | Ongoing |
| Certificate Generation | [Certificate Workflow](10-Certificate-Workflow.md) | Automatic (< 1 hour) |

### Technology Stack

- **Frontend**: React (SPA)
- **Backend**: Spring Boot + Spring Security
- **Database**: PostgreSQL (Supabase)
- **Build**: Maven
- **Authentication**: JWT

---

## Database Schema Overview

### Entity Groups

**User Management**:
- users, students, teachers, admins

**Course Management**:
- courses, course_offerings, enrollment_windows

**Enrollment & Allocation**:
- enrollments, batches, allocations

**Assessment**:
- grading_schemes, grade_components, grade_entries, attendance, gradebooks

**Certificates & Feedback**:
- certificates, feedback_forms, feedback_questions, feedback_responses, grievances

**Total Tables**: 20

---

## API Endpoints Summary

### Student APIs
```
GET    /api/student/dashboard
GET    /api/student/profile
POST   /api/student/enroll
GET    /api/student/enrollments
GET    /api/student/progress/{courseId}
GET    /api/student/certificates
GET    /api/student/certificates/{id}/download
POST   /api/student/grievance
GET    /api/student/feedback/{courseId}
```

### Teacher APIs
```
GET    /api/teacher/dashboard
GET    /api/teacher/courses
POST   /api/teacher/attendance
POST   /api/teacher/grades
PATCH  /api/teacher/grades/publish
POST   /api/teacher/course/complete
```

### Admin APIs
```
GET    /api/admin/dashboard
POST   /api/admin/semesters
POST   /api/admin/course-offerings
POST   /api/admin/grading-schemes
POST   /api/admin/enrollment-windows
POST   /api/admin/allocation/run
PATCH  /api/admin/switch-requests/{id}
GET    /api/admin/reports
```

---

## Component Interaction Flow

### Login
```
User → Login UI → AuthController → AuthService → JWT Token
```

### Enrollment
```
Student → Dashboard → EnrollmentUI → EnrollmentController → EnrollmentService → Database
```

### Batch Allocation
```
Admin → AllocationUI → AllocationController → AllocationService → Random Shuffle → Allocations → Database
```

### Grading
```
Teacher → GradebookUI → GradebookController → GradingService → Database → Student Progress
```

---

## Design Decisions

Key architectural choices documented in:
- [ADR-001: CourseOffering Entity](../Architecture/ADR-001-CourseOffering.md)
- [ADR-002: Random Allocation](../Architecture/ADR-002-RandomAllocation.md)
- [ADR-003: RBAC Model](../Architecture/ADR-003-RBAC.md)
- [ADR-004: Gradebook States](../Architecture/ADR-004-Gradebook.md)
- [ADR-005: Certificate Template](../Architecture/ADR-005-CertificateGeneration.md)

---

## Performance Targets

| Operation | Target | Achievement |
|-----------|--------|-------------|
| Page Load | 2 seconds | ✓ |
| API Response | 1 second | ✓ |
| Allocation Run (1000 students) | 5 minutes | ✓ |
| Certificate Generation | < 1 hour for 500 students | ✓ |
| Concurrent Users | 200 | ✓ |

---

## Security Features

✓ JWT Authentication
✓ Role-Based Access Control
✓ Password Hashing (bcrypt)
✓ HTTPS/TLS Encryption
✓ SQL Injection Prevention
✓ XSS Protection
✓ CSRF Protection
✓ Audit Logging
✓ Rate Limiting
✓ Account Lockout

---

## Database Indexes

**Critical Indexes**:
- users(email) - Login queries
- students(user_id) - User lookup
- enrollments(student_id, course_offering_id) - Enrollment queries
- allocations(batch_id) - Batch composition
- grade_entries(allocation_id) - Student grades
- attendance(allocation_id, attendance_date) - Attendance tracking

---

## Deployment Architecture

```
Frontend (React SPA)
    ↓
API Gateway / Load Balancer
    ↓
Backend Instances (Spring Boot) × N
    ↓
PostgreSQL Database (Supabase)
    ↓
Backup & DR
```

---

## Cross-Reference by Topic

### Enrollment
- [Enrollment Design](04-Enrollment-Design.md) - Complete workflow
- [Database Design](02-Database-Design.md#411-enrollments-table) - Schema
- [System Architecture](01-System-Architecture.md#enrollment-example) - Data flow

### Grading
- [System Architecture](01-System-Architecture.md) - Overview
- [Database Design](02-Database-Design.md#412-grade-entries-table) - Schema
- [Functional Requirements](../SRS/05-Functional-Requirements.md#fr-tea-003-gradebook-management) - Requirements

### Authentication
- [RBAC Design](03-RBAC-Design.md#4-access-control-enforcement) - Implementation
- [System Architecture](01-System-Architecture.md#authentication--authorization-flow) - Flow
- [Database Design](02-Database-Design.md#41-users-table) - Schema

### Allocation
- [Batch Allocation](05-Batch-Allocation.md) - Complete algorithm
- [Batch Allocation](05-Batch-Allocation.md#3-api-design) - API
- [Database Design](02-Database-Design.md#410-allocations-table) - Schema

### Certificates
- [Certificate Workflow](10-Certificate-Workflow.md) - Complete process
- [Database Design](02-Database-Design.md#416-certificates-table) - Schema
- [Functional Requirements](../SRS/05-Functional-Requirements.md#fr-stu-007-certificate-download) - Requirements

---

## Document Statistics

| Category | Count |
|----------|-------|
| SDD Documents | 10 design docs + 1 index |
| Database Tables | 20 |
| API Endpoints | 20+ |
| Indexes | 10+ |
| Constraints | 25+ |

---

## Version Control

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026 | Initial SDD creation |

---

## Quick Navigation

- [Previous: SRS Index](../SRS/SRS-Index.md)
- [Next: API Documentation](../API/API-Index.md)
- [Back: Documentation Home](../)
- [Related: Testing Documentation](../Testing/Testing-Index.md)
- [Related: Architecture Decisions](../Architecture/)

---

## Related Documentation

- [Requirements (SRS)](../SRS/SRS-Index.md)
- [API Documentation](../API/API-Index.md)
- [Testing](../Testing/Testing-Index.md)
- [Architecture Decisions](../Architecture/)
- [UML Diagrams](../UML/)
