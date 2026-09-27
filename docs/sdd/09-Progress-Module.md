# Progress Module — Module Design

## 1. Overview

This document describes the **Progress Module**, the student-facing aggregation of attendance, grades, and course-completion status. It implements [FR-STU-004](../SRS/05-Functional-Requirements.md#2-student-modules) and is shared as a widget on the [Student Dashboard](07-Student-Dashboard.md) and as a standalone detail view.

## 2. Scope

Progress is derived data — this module reads from `attendance`, `grade_entries`, `gradebooks`, and `allocations`; it does not own any table of its own.

## 3. Data Model

| Field shown to student | Computed from |
|--------------------------|----------------|
| Attendance % | `COUNT(*) FILTER (WHERE status='PRESENT') * 100.0 / COUNT(*)` over `attendance` for the student's `allocation_id` |
| Component-wise marks | `grade_entries` joined to `grade_components`, **only** where the parent `gradebooks.state = 'PUBLISHED'` |
| Overall grade % | `Σ (marks_obtained × weight_percentage / 100)` per [Database Design §8.1](02-Database-Design.md#81-common-queries) |
| Course completion status | `allocations` / `batches` completion flag set by [Course Completion](06-Teacher-Workspace.md#7-course-completion-sub-module) |
| Progress timeline | Enrollment date → Allocation date → Attendance start → Grade publication → Completion date |

## 4. Draft-Grade Visibility Rule

This is the single most important invariant in this module: **a student must never see a mark before its gradebook is published.**

```sql
-- Correct: only published grades reach the student
SELECT ge.*
FROM grade_entries ge
JOIN allocations a ON ge.allocation_id = a.allocation_id
JOIN gradebooks gb ON gb.batch_id = a.batch_id
WHERE a.student_id = ?
  AND gb.state = 'PUBLISHED';
```

Any query path that returns `grade_entries` for a student without the `gradebooks.state = 'PUBLISHED'` filter is a defect against FR-STU-004 and FR-TEA-003.

## 5. Progress API

**Primary endpoint**: `GET /student/progress/{courseId}` (also exposed as `GET /student/attendance/{courseId}` and `GET /student/grades/{courseId}` for focused views) — see [Student API](../API/Student.md).

**Response shape** (see [Student API](../API/Student.md) for full schema):
```json
{
  "courseOfferingId": 5,
  "batchName": "Batch B",
  "attendancePercentage": 92.5,
  "components": [
    { "name": "Attendance", "marksObtained": 9, "maxMarks": 10, "weight": 10 },
    { "name": "Final Assessment", "marksObtained": 54, "maxMarks": 60, "weight": 60 }
  ],
  "overallGrade": 88.3,
  "gradebookState": "PUBLISHED",
  "completionStatus": "IN_PROGRESS"
}
```

If `gradebookState` is `DRAFT`, the `components` and `overallGrade` fields are omitted (not zeroed — omitted, so the frontend cannot mistake "not yet published" for "zero marks").

## 6. Update Latency

Per FR-STU-004 acceptance criteria, course status/progress must reflect a teacher's publish action within 1 minute. No caching layer sits in front of these queries in the MVP; this is satisfied directly by synchronous reads against PostgreSQL.

## 7. Access Control

Self-only access (`@studentService.isOwnerOrAdmin`), consistent with the rest of the Student module — see [RBAC Design §5.1](03-RBAC-Design.md#51-student-module) and [RBAC Design §7.1](03-RBAC-Design.md#71-student-data-visibility).

## 8. Related Documents

- [Functional Requirements — FR-STU-004](../SRS/05-Functional-Requirements.md#2-student-modules)
- [Student Dashboard](07-Student-Dashboard.md)
- [Teacher Workspace — Gradebook](06-Teacher-Workspace.md#6-gradebook-sub-module)
- [Student API](../API/Student.md)
- [Database Design](02-Database-Design.md)
