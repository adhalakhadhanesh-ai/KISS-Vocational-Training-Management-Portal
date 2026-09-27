# Teacher Workspace — Module Design

## 1. Overview

This document describes the design of the **Teacher Workspace**, the single module through which teachers manage their assigned batches: dashboard, attendance, gradebook (Draft → Publish), and course completion. It implements the functional requirements in [FR-TEA-001 to FR-TEA-005](../SRS/05-Functional-Requirements.md#3-teacher-modules).

## 2. Scope

The Teacher Workspace covers:
- Teacher Dashboard (assigned courses/batches, pending tasks)
- Attendance Management
- Gradebook (Draft/Published lifecycle)
- Course Completion Marking

It does **not** cover student-facing views (see [Student Dashboard](07-Student-Dashboard.md) and [Progress Module](09-Progress-Module.md)) or admin configuration (see [Admin Dashboard](08-Admin-Dashboard.md)).

## 3. Module Layout

```
Teacher Workspace
├─ Dashboard              (FR-TEA-001)
├─ Manage Course
│   ├─ Attendance         (FR-TEA-002)
│   └─ Gradebook          (FR-TEA-003, FR-TEA-004)
└─ Course Completion       (FR-TEA-005)
```

A teacher reaches "Manage Course" for a specific batch and, from there, moves between the Attendance and Gradebook tabs without leaving the batch context.

## 4. Teacher Dashboard

**Data sources**: `teachers`, `batches` (assigned via `teacher_id`), `course_offerings`, `enrollments`/`allocations` (for student counts), `gradebooks` (for pending-publish indicators).

**Widgets**:
- Assigned courses/batches for the current semester
- Batch size and schedule (if applicable)
- Pending tasks: attendance not yet marked today, gradebooks still in Draft
- Course status (Ongoing / Completed)

**Query pattern**:
```sql
SELECT b.batch_id, b.batch_name, co.course_name, co.semester,
       COUNT(a.allocation_id) AS student_count
FROM batches b
JOIN course_offerings co ON b.course_offering_id = co.course_offering_id
LEFT JOIN allocations a ON a.batch_id = b.batch_id
WHERE b.teacher_id = ?
GROUP BY b.batch_id, co.course_offering_id;
```

## 5. Attendance Sub-Module

Implements FR-TEA-002.

**Flow**:
```
Teacher selects batch → selects date (today or past, never future)
    → System loads roster from Allocations
    → Teacher marks Present/Absent per student (bulk "mark all present" + toggle exceptions)
    → Save → Attendance rows upserted (one row per allocation per date)
```

**Constraints**:
- One attendance row per `(allocation_id, attendance_date)` — see `attendance` table's unique index in [Database Design](02-Database-Design.md#5-indexes)
- No future-dated attendance
- Edits allowed within 7 days of entry; edits after that require admin override (audit-logged per [RBAC Design §8](03-RBAC-Design.md#8-audit-logging))

**Primary endpoints**: `POST /teacher/attendance`, `GET /teacher/attendance/{batchId}`, `PATCH /teacher/attendance/{attendanceId}` — see [Teacher API](../API/Teacher.md).

## 6. Gradebook Sub-Module

Implements FR-TEA-003 and FR-TEA-004.

**State machine**:

```
        enter/save marks
DRAFT ───────────────────► DRAFT
  │
  │ publish (validated)
  ▼
PUBLISHED  (immutable; visible to students)
```

**Validation on publish**:
- At least one student has at least one mark entered
- Marks per component do not exceed `grade_components.max_marks`
- Overall grade computed as `Σ (component_mark × weight_percentage / 100)` per [Database Design §8.1](02-Database-Design.md#81-common-queries)

**Publish is one-way**: once a `gradebooks.state` row flips to `PUBLISHED`, the API rejects further `POST /teacher/grades` calls for that batch (see [Error Codes](../API/Error-Codes.md) `GRADEBOOK_LOCKED`).

**Primary endpoints**: `GET /teacher/gradebook/{batchId}`, `POST /teacher/grades`, `PATCH /teacher/grades/{gradeId}`, `PATCH /teacher/gradebook/{batchId}/publish` — see [Teacher API](../API/Teacher.md).

## 7. Course Completion Sub-Module

Implements FR-TEA-005.

**Preconditions**: `course_offerings.end_date` has passed; the batch's gradebook must be `PUBLISHED` (validated server-side — matches the certificate-generation precondition in [Certificate Workflow §2](10-Certificate-Workflow.md#2-certificate-generation-flow)).

**Flow**:
```
Teacher clicks "Mark Complete" on a batch
    → Confirmation dialog
    → POST /teacher/course/{batchId}/complete
    → System marks all allocations in the batch as completed
    → CertificateService triggered asynchronously (see Certificate Workflow)
    → Email notification queued to all students in the batch
```

This hands off to [Certificate Workflow](10-Certificate-Workflow.md) for certificate generation.

## 8. Access Control

Every endpoint in this module is scoped to the **assigned teacher for the batch** (`@teacherService.isAssignedTeacher(batchId, authentication)`), with Admin retaining full override access. See [RBAC Design §5.2](03-RBAC-Design.md#52-teacher-module).

## 9. Related Documents

- [Functional Requirements — Teacher Modules](../SRS/05-Functional-Requirements.md#3-teacher-modules)
- [RBAC Design](03-RBAC-Design.md)
- [Database Design](02-Database-Design.md)
- [Teacher API](../API/Teacher.md)
- [Certificate Workflow](10-Certificate-Workflow.md)
