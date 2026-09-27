# Student API

## Base URL

```
https://api.example.com/v1
```

## Overview

All 17 student-facing endpoints. Every endpoint requires a valid JWT with `role: STUDENT` (see [Authentication API](Authentication.md)) and is scoped to the authenticated student's own records unless noted. Full module design: [Student Dashboard](../SDD/07-Student-Dashboard.md), [Progress Module](../SDD/09-Progress-Module.md), [Enrollment Design](../SDD/04-Enrollment-Design.md).

## Endpoint Summary

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/student/dashboard` | Dashboard data |
| GET | `/student/profile` | Get own profile |
| PATCH | `/student/profile` | Update contact fields |
| GET | `/student/available-courses` | List courses open for enrollment |
| POST | `/student/enroll` | Submit enrollment |
| GET | `/student/enrollments` | Get all enrollments |
| DELETE | `/student/enrollments/{enrollmentId}` | Drop course |
| GET | `/student/progress/{courseId}` | View progress |
| GET | `/student/attendance/{courseId}` | View attendance |
| GET | `/student/grades/{courseId}` | View grades |
| POST | `/student/allocation/{allocationId}/switch` | Request batch switch |
| POST | `/student/grievance` | Submit grievance |
| GET | `/student/grievances` | List own grievances |
| GET | `/student/feedback/{courseId}` | Get feedback form |
| POST | `/student/feedback/{courseId}` | Submit feedback |
| GET | `/student/certificates` | List certificates |
| GET | `/student/certificates/{id}/download` | Download certificate PDF |

---

## GET /student/dashboard

**Purpose**: Aggregate dashboard data (FR-STU-001).

**Response** (200 OK):
```json
{
  "enrolledCourse": { "courseOfferingId": 5, "courseName": "Java Programming", "batchName": "Batch B" },
  "progressSummary": { "attendancePercentage": 92.5, "overallGrade": 88.3, "gradebookState": "PUBLISHED" },
  "announcements": [
    { "message": "Grades published for Java Programming", "timestamp": "2026-01-20T09:00:00Z" }
  ],
  "recentlyPublishedGrades": true
}
```

---

## GET /student/profile

**Purpose**: View own profile (FR-STU-002).

**Response** (200 OK):
```json
{
  "userId": 42,
  "name": "Rahul Sharma",
  "rollNumber": "042",
  "section": "A",
  "email": "student@example.com",
  "phoneNumber": "+91-9876543210",
  "address": "123 Main St, Pune",
  "mentorName": "Dr. Anita Rao",
  "mentorPhone": "+91-9812345678"
}
```

---

## PATCH /student/profile

**Purpose**: Update own contact fields only (FR-STU-002). Name, roll number, section, and mentor fields are admin-managed (see [Admin API](Admin.md)) and are rejected here.

**Request**:
```json
{
  "phoneNumber": "+91-9876543210",
  "address": "123 Main St, Pune"
}
```

**Response** (200 OK): updated profile object (same shape as `GET /student/profile`).

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 422 | READ_ONLY_FIELD | Attempted to modify name, rollNumber, or mentor fields |

---

## GET /student/available-courses

**Purpose**: List courses open to this student's roll-number range (FR-STU-003).

**Response** (200 OK): see [Enrollment Design §7.3](../SDD/04-Enrollment-Design.md#73-get-active-courses-for-enrollment).

---

## POST /student/enroll

**Purpose**: Submit enrollment (FR-STU-003).

**Request/Response**: see [Enrollment Design §7.1](../SDD/04-Enrollment-Design.md#71-submit-enrollment).

**Error Responses**: `ENR_001`–`ENR_007` — see [Error Codes](Error-Codes.md#enrollment-errors).

---

## GET /student/enrollments

**Purpose**: List own enrollment history (FR-STU-003).

**Response**: see [Enrollment Design §7.2](../SDD/04-Enrollment-Design.md#72-get-student-enrollments).

---

## DELETE /student/enrollments/{enrollmentId}

**Purpose**: Drop a course before allocation runs (see [Enrollment Design §11](../SDD/04-Enrollment-Design.md#11-enrollment-cancellation)).

**Response** (200 OK):
```json
{ "message": "Enrollment dropped", "enrollmentId": 42 }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 409 | ENR_ALREADY_ALLOCATED | Cannot drop after batch allocation has run |
| 404 | ENR_005 | Invalid or unowned enrollment |

---

## GET /student/progress/{courseId}

**Purpose**: Full progress view (FR-STU-004). See [Progress Module §5](../SDD/09-Progress-Module.md#5-progress-api) for the full response schema and the draft-grade visibility rule.

---

## GET /student/attendance/{courseId}

**Purpose**: Attendance-only view of the same data as `/progress/{courseId}`.

**Response** (200 OK):
```json
{ "courseOfferingId": 5, "attendancePercentage": 92.5, "presentDays": 37, "totalDays": 40 }
```

---

## GET /student/grades/{courseId}

**Purpose**: Grades-only view. Returns `404 GRADES_NOT_PUBLISHED` (not zeroed marks) if the gradebook is still in `DRAFT` — see [Progress Module §5](../SDD/09-Progress-Module.md#5-progress-api).

---

## POST /student/allocation/{allocationId}/switch

**Purpose**: Request a one-time batch switch (FR-ADM-011). See [Batch Allocation §6.2](../SDD/05-Batch-Allocation.md#62-batch-switching-api).

**Request**:
```json
{ "targetBatchId": 9, "reason": "Schedule conflict with another commitment" }
```

**Response** (201 Created):
```json
{ "switchRequestId": 7, "status": "PENDING", "requestedAt": "2026-01-22T11:00:00Z" }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 409 | SWITCH_ALREADY_REQUESTED | One switch request per student per course already exists |
| 400 | SWITCH_TARGET_FULL | Target batch at capacity |

---

## POST /student/grievance

**Purpose**: Lodge a grievance (FR-STU-005).

**Request**:
```json
{
  "category": "GRADING",
  "description": "Component mark appears to not match the marked script",
  "severity": "MEDIUM"
}
```

**Response** (201 Created):
```json
{ "grievanceId": 15, "status": "SUBMITTED", "createdAt": "2026-01-22T11:05:00Z" }
```

---

## GET /student/grievances

**Purpose**: List own grievances with status history (FR-STU-005).

**Response** (200 OK): array of grievance objects, same shape as the create response plus `resolutionNotes` when resolved.

---

## GET /student/feedback/{courseId}

**Purpose**: Fetch the feedback form for an enrolled/completed course (FR-STU-006).

**Response** (200 OK):
```json
{
  "formId": 3,
  "formName": "Java Programming Feedback",
  "questions": [
    { "questionId": 11, "text": "Rate the pace of the course", "type": "RATING", "required": true }
  ]
}
```

---

## POST /student/feedback/{courseId}

**Purpose**: Submit feedback (FR-STU-006). Stored without student identification (`allocation_id` only, per [Database Design §4.19](../SDD/02-Database-Design.md#419-feedbackresponses-table)).

**Request**:
```json
{ "responses": [ { "questionId": 11, "rating": 4 } ] }
```

**Response** (201 Created):
```json
{ "message": "Feedback submitted" }
```

---

## GET /student/certificates

**Purpose**: List certificates for completed courses (FR-STU-007).

**Response** (200 OK):
```json
[
  { "certificateId": 8, "certNumber": "KISS-2026-00042", "courseName": "Java Programming", "issueDate": "2026-05-20" }
]
```

---

## GET /student/certificates/{id}/download

**Purpose**: Download certificate PDF (FR-STU-007). See [Certificate Workflow §5.3](../SDD/10-Certificate-Workflow.md#53-download-certificate).

**Response**: `200 OK`, `Content-Type: application/pdf` (binary body).

---

## Related Documentation

- [Student Dashboard](../SDD/07-Student-Dashboard.md)
- [Progress Module](../SDD/09-Progress-Module.md)
- [Enrollment Design](../SDD/04-Enrollment-Design.md)
- [Error Codes](Error-Codes.md)
- [API Index](API-Index.md)
