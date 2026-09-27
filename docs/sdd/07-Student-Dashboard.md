# Student Dashboard — Module Design

## 1. Overview

This document describes the design of the **Student Dashboard**, the landing module students see after login, and the surrounding student-facing modules (Profile, Enrollment). It implements [FR-STU-001 to FR-STU-003](../SRS/05-Functional-Requirements.md#2-student-modules). Progress, grievances, feedback, and certificates are detailed in their own sections/documents to keep this document focused.

## 2. Module Layout

```
Student Dashboard
├─ Overview widgets        (FR-STU-001)
├─ Profile (read-only)     (FR-STU-002)
└─ Enrollment               (FR-STU-003)
```

## 3. Dashboard Widgets

**Data sources**: `enrollments`, `allocations`, `batches`, `course_offerings`, `grade_entries` (via `gradebooks.state = 'PUBLISHED'`), `enrollment_windows`.

| Widget | Source | Notes |
|--------|--------|-------|
| Enrolled course + batch | `enrollments` → `allocations` → `batches` | Shows "Pending allocation" if not yet allocated |
| Progress snapshot | `attendance`, `grade_entries` (published only) | See [Progress Module](09-Progress-Module.md) for calculation |
| Quick-access cards | Static links | Progress, Grievances, Certificates |
| Announcements | Reverse chronological | Enrollment window open/close, allocation results, grade publication |
| Recently published grades | `gradebooks.published_date` | Notification badge, cleared on view |

**Performance target**: dashboard loads in < 2 seconds (see [Non-Functional Requirements](../SRS/06-Non-Functional-Requirements.md)); achieved via a single aggregate query per widget rather than N+1 lookups.

## 4. Profile Sub-Module

Implements FR-STU-002.

- Fields shown: Name, Roll Number, Section, Email, Phone, Address, Mentor Name, Mentor Phone (`mentor_name`/`mentor_phone` on `students` — see [Database Design §4.2](02-Database-Design.md#42-students-table))
- **Read-only for students.** Only Admin can edit (`PATCH /admin/users/{userId}`); student-facing `PATCH /student/profile` is limited to contact fields only (phone, address) per the API contract in [Student API](../API/Student.md) — name, roll number, and mentor fields are admin-managed.
- Profile history is not tracked (no versioning table), consistent with [Assumptions AS-ORG-003](../SRS/08-Assumptions.md).

## 5. Enrollment Sub-Module

Implements FR-STU-003. Full workflow, validation rules, and API detail live in [Enrollment Design](04-Enrollment-Design.md); this section covers only how the dashboard surfaces it.

**Dashboard states**:

| Enrollment window status | Dashboard shows |
|---------------------------|------------------|
| Not yet open for student's roll range | "Enrollment opens on \<date\>" |
| Open for student's roll range | "Enroll now" with available course list |
| Closed, student enrolled | Enrolled course + "Awaiting batch allocation" |
| Closed, student not enrolled | "Enrollment window closed" (student missed the window) |

**Primary endpoints**: `GET /student/dashboard`, `GET /student/profile`, `PATCH /student/profile`, `GET /student/available-courses`, `POST /student/enroll` — see [Student API](../API/Student.md).

## 6. Access Control

All endpoints in this module enforce **self-only** access (`@studentService.isOwnerOrAdmin`), with Admin able to view/edit on behalf of a student. See [RBAC Design §5.1](03-RBAC-Design.md#51-student-module).

## 7. Related Documents

- [Functional Requirements — Student Modules](../SRS/05-Functional-Requirements.md#2-student-modules)
- [Enrollment Design](04-Enrollment-Design.md)
- [Progress Module](09-Progress-Module.md)
- [Student API](../API/Student.md)
- [Database Design](02-Database-Design.md)
