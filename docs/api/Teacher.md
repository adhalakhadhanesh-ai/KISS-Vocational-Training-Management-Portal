# Teacher API

## Base URL

```
https://api.example.com/v1
```

## Overview

All 12 teacher-facing endpoints. Every endpoint requires a valid JWT with `role: TEACHER` (see [Authentication API](Authentication.md)) and is scoped to batches assigned to the authenticated teacher unless noted. Full module design: [Teacher Workspace](../SDD/06-Teacher-Workspace.md).

## Endpoint Summary

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/teacher/dashboard` | Teacher dashboard |
| GET | `/teacher/courses` | List assigned courses |
| GET | `/teacher/batch/{batchId}/students` | List batch students |
| POST | `/teacher/attendance` | Record attendance |
| GET | `/teacher/attendance/{batchId}` | View attendance |
| PATCH | `/teacher/attendance/{attendanceId}` | Edit attendance |
| GET | `/teacher/gradebook/{batchId}` | View gradebook |
| POST | `/teacher/grades` | Enter grades |
| PATCH | `/teacher/grades/{gradeId}` | Edit a single grade entry |
| PATCH | `/teacher/gradebook/{batchId}/publish` | Publish grades |
| POST | `/teacher/course/{batchId}/complete` | Mark course complete |
| GET | `/teacher/feedback/{courseId}` | View aggregated feedback |

---

## GET /teacher/dashboard

**Purpose**: Assigned batches, pending tasks (FR-TEA-001). See [Teacher Workspace §4](../SDD/06-Teacher-Workspace.md#4-teacher-dashboard).

**Response** (200 OK):
```json
{
  "assignedBatches": [
    { "batchId": 9, "batchName": "Batch B", "courseName": "Java Programming", "studentCount": 32, "status": "ONGOING" }
  ],
  "pendingTasks": { "attendanceNotMarkedToday": 1, "gradebooksInDraft": 1 }
}
```

---

## GET /teacher/courses

**Purpose**: List courses/batches assigned to this teacher for the current semester (FR-TEA-001).

**Response** (200 OK): array of `{ courseOfferingId, courseName, batchId, batchName, semester }`.

---

## GET /teacher/batch/{batchId}/students

**Purpose**: Roster for a batch (used by both Attendance and Gradebook screens).

**Response** (200 OK):
```json
[ { "allocationId": 101, "studentId": 42, "name": "Rahul Sharma", "rollNumber": "042" } ]
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 403 | NOT_ASSIGNED_TEACHER | Teacher is not assigned to this batch |

---

## POST /teacher/attendance

**Purpose**: Record attendance for a batch/date (FR-TEA-002). See [Teacher Workspace §5](../SDD/06-Teacher-Workspace.md#5-attendance-sub-module).

**Request**:
```json
{
  "batchId": 9,
  "attendanceDate": "2026-01-22",
  "records": [
    { "allocationId": 101, "status": "PRESENT" },
    { "allocationId": 102, "status": "ABSENT" }
  ]
}
```

**Response** (201 Created):
```json
{ "message": "Attendance recorded", "batchId": 9, "attendanceDate": "2026-01-22", "recordCount": 2 }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 400 | ATT_FUTURE_DATE | Cannot mark attendance for a future date |
| 409 | ATT_ALREADY_MARKED | Attendance already recorded for this batch/date (use PATCH to edit) |

---

## GET /teacher/attendance/{batchId}

**Purpose**: View attendance history for a batch, filterable by date.

**Query Parameters**: `?date_from=2026-01-01&date_to=2026-01-31`

**Response** (200 OK): array of `{ attendanceDate, presentCount, absentCount, attendancePercentage }`.

---

## PATCH /teacher/attendance/{attendanceId}

**Purpose**: Edit an attendance record. Allowed within 7 days of entry; later edits require Admin override (audit-logged — see [RBAC Design §8](../SDD/03-RBAC-Design.md#8-audit-logging)).

**Request**:
```json
{ "status": "PRESENT" }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 403 | ATT_EDIT_WINDOW_EXPIRED | Edit window (7 days) has passed; requires admin override |

---

## GET /teacher/gradebook/{batchId}

**Purpose**: View gradebook state and current marks (FR-TEA-003). See [Teacher Workspace §6](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module).

**Response** (200 OK):
```json
{
  "batchId": 9,
  "state": "DRAFT",
  "components": [ { "componentId": 3, "name": "Attendance", "maxMarks": 10, "weight": 10 } ],
  "entries": [ { "allocationId": 101, "componentId": 3, "marksObtained": 9 } ]
}
```

---

## POST /teacher/grades

**Purpose**: Enter marks (component-wise) while the gradebook is in `DRAFT` (FR-TEA-003).

**Request**:
```json
{
  "batchId": 9,
  "entries": [ { "allocationId": 101, "componentId": 3, "marksObtained": 9 } ]
}
```

**Response** (200 OK):
```json
{ "message": "Grades saved", "batchId": 9, "state": "DRAFT" }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 422 | GRADE_EXCEEDS_MAX | `marksObtained` exceeds the component's `maxMarks` |
| 409 | GRADEBOOK_LOCKED | Gradebook already `PUBLISHED`; cannot be edited |

---

## PATCH /teacher/grades/{gradeId}

**Purpose**: Edit a single `grade_entries` row while still in `DRAFT`.

**Error Responses**: same as `POST /teacher/grades`.

---

## PATCH /teacher/gradebook/{batchId}/publish

**Purpose**: Publish the gradebook (FR-TEA-004). One-way transition — see [Teacher Workspace §6](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module).

**Response** (200 OK):
```json
{ "batchId": 9, "state": "PUBLISHED", "publishedDate": "2026-01-25T10:00:00Z" }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 422 | GRADEBOOK_EMPTY | No student has any mark entered; publish rejected |
| 409 | GRADEBOOK_LOCKED | Already published |

---

## POST /teacher/course/{batchId}/complete

**Purpose**: Mark course complete, triggering certificate generation (FR-TEA-005). See [Teacher Workspace §7](../SDD/06-Teacher-Workspace.md#7-course-completion-sub-module) and [Certificate Workflow](../SDD/10-Certificate-Workflow.md).

**Response** (200 OK):
```json
{ "batchId": 9, "status": "COMPLETED", "completionDate": "2026-05-20", "certificatesQueued": 32 }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 400 | COURSE_NOT_ENDED | `course_offerings.end_date` has not yet passed |
| 409 | GRADEBOOK_NOT_PUBLISHED | Gradebook for this batch must be published before the course can be marked complete |

---

## GET /teacher/feedback/{courseId}

**Purpose**: View aggregated (anonymous) feedback for an assigned course.

**Response** (200 OK):
```json
{ "courseOfferingId": 5, "responseCount": 28, "averageRating": 4.2 }
```

---

## Related Documentation

- [Teacher Workspace](../SDD/06-Teacher-Workspace.md)
- [Certificate Workflow](../SDD/10-Certificate-Workflow.md)
- [Error Codes](Error-Codes.md)
- [API Index](API-Index.md)
