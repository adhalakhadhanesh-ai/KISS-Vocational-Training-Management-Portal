# Admin API

## Base URL

```
https://api.example.com/v1
```

## Overview

All 22 admin-facing endpoints. Every endpoint requires a valid JWT with `role: ADMIN` (see [Authentication API](Authentication.md)); admin access is not further resource-scoped (see [RBAC Design §5.3](../SDD/03-RBAC-Design.md#53-admin-module)). Full module design: [Admin Dashboard](../SDD/08-Admin-Dashboard.md), [Batch Allocation](../SDD/05-Batch-Allocation.md), [Certificate Workflow](../SDD/10-Certificate-Workflow.md).

## Endpoint Summary

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/admin/dashboard` | Command-center trackers |
| POST | `/admin/semesters` | Create semester |
| GET | `/admin/semesters` | List semesters |
| PATCH | `/admin/semesters/{semesterId}` | Update semester |
| POST | `/admin/course-offerings` | Create course offering |
| GET | `/admin/course-offerings` | List offerings |
| POST | `/admin/grading-schemes` | Define grading scheme |
| GET | `/admin/grading-schemes` | List schemes |
| POST | `/admin/enrollment-windows` | Create enrollment window |
| GET | `/admin/enrollment-windows` | List windows |
| PATCH | `/admin/enrollment-windows/{windowId}/activate` | Activate window |
| PATCH | `/admin/enrollment-windows/{windowId}/deactivate` | Deactivate window |
| POST | `/admin/allocation/run` | Run batch allocation |
| GET | `/admin/allocation/{allocationId}` | Get allocation report |
| GET | `/admin/switch-requests` | List switch requests |
| PATCH | `/admin/switch-requests/{requestId}` | Approve/reject switch |
| GET | `/admin/reports` | Generate reports |
| GET | `/admin/users` | List all users |
| POST | `/admin/users` | Create user |
| PATCH | `/admin/users/{userId}` | Update user (incl. student profile fields) |
| PATCH | `/admin/users/{userId}/role` | Change user role |
| DELETE | `/admin/users/{userId}` | Deactivate user |

---

## GET /admin/dashboard

**Purpose**: Command-center trackers (FR-ADM-001). See [Admin Dashboard §3](../SDD/08-Admin-Dashboard.md#3-command-center-trackers).

**Response** (200 OK):
```json
{
  "enrollmentTracker": [ { "courseOfferingId": 5, "fillPercentage": 85.0 } ],
  "allocationTracker": { "allocated": 320, "pending": 12 },
  "feedbackTracker": { "totalResponses": 210, "averageRating": 4.1 },
  "grievanceTracker": { "submitted": 4, "underReview": 2, "resolved": 30 }
}
```

---

## POST /admin/semesters

**Purpose**: Create a semester grouping (FR-ADM-002). See [Admin Dashboard §4](../SDD/08-Admin-Dashboard.md#4-semester-management).

**Request**:
```json
{ "semesterName": "Spring 2026", "startDate": "2026-02-01", "endDate": "2026-05-31" }
```

**Response** (201 Created): `{ "semesterName": "Spring 2026", "startDate": "2026-02-01", "endDate": "2026-05-31" }`

## GET /admin/semesters

List all semesters.

## PATCH /admin/semesters/{semesterId}

Update semester dates. Rejected once any course offering under it has an active allocation.

---

## POST /admin/course-offerings

**Purpose**: Create a course offering (FR-ADM-003). See [Admin Dashboard §5](../SDD/08-Admin-Dashboard.md#5-course-offering-seat--batch-configuration) and the full example in [API Index](API-Index.md#create-resource-example).

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 400 | CAPACITY_MISMATCH | Sum of batch capacities does not equal `max_capacity` |
| 409 | OFFERING_EXISTS | Course already offered this semester (unique `(course_id, semester)`) |

## GET /admin/course-offerings

List offerings, filterable by `?semester=`.

---

## POST /admin/grading-schemes

**Purpose**: Define a weighted grading scheme (FR-ADM-007).

**Request**:
```json
{
  "courseOfferingId": 5,
  "components": [
    { "name": "Attendance", "maxMarks": 10, "weightPercentage": 10 },
    { "name": "Final Assessment", "maxMarks": 60, "weightPercentage": 60 }
  ]
}
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 422 | WEIGHT_SUM_INVALID | Component weights do not sum to 100% |

## GET /admin/grading-schemes

List schemes, filterable by `?courseOfferingId=`.

---

## POST /admin/enrollment-windows

**Purpose**: Define an enrollment window, optionally staggered by roll-number range (FR-ADM-009). See [Enrollment Design §3](../SDD/04-Enrollment-Design.md#3-enrollment-window-management).

**Request**:
```json
{
  "courseOfferingId": 5,
  "windowStart": "2026-01-15",
  "windowEnd": "2026-01-20",
  "rollNumberStart": "001",
  "rollNumberEnd": "100"
}
```

## GET /admin/enrollment-windows

List windows, filterable by `?courseOfferingId=`.

## PATCH /admin/enrollment-windows/{windowId}/activate

Activate a window (sets `is_active = true`).

## PATCH /admin/enrollment-windows/{windowId}/deactivate

Deactivate/close a window (sets `is_active = false`); required before running allocation (FR-ADM-010).

---

## POST /admin/allocation/run

**Purpose**: Run the random batch-allocation algorithm (FR-ADM-010). See [Batch Allocation §4.1](../SDD/05-Batch-Allocation.md#41-run-allocation).

**Error Responses**: see [Batch Allocation §8](../SDD/05-Batch-Allocation.md#8-error-scenarios) (`ALLOC_INSUFFICIENT_CAPACITY`, `ALLOC_NO_ENROLLMENTS`).

## GET /admin/allocation/{allocationId}

Get the allocation report. See [Batch Allocation §4.2](../SDD/05-Batch-Allocation.md#42-get-allocation-report).

---

## GET /admin/switch-requests

List pending/all batch-switch requests (FR-ADM-011). See [Batch Allocation §6.3](../SDD/05-Batch-Allocation.md#63-admin-approval).

## PATCH /admin/switch-requests/{requestId}

Approve or reject a switch request.

**Request**:
```json
{ "decision": "APPROVED", "comment": "Target batch has capacity" }
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 400 | SWITCH_TARGET_FULL | Cannot approve — target batch now at capacity |

---

## GET /admin/reports

**Purpose**: Generate certificate/student/course reports (FR-ADM-012).

**Query Parameters**: `?type=certificate|student|course&format=pdf|csv&date_from=&date_to=`

**Response**: `200 OK` with `Content-Type` matching `format`, or a JSON summary if `format` is omitted.

---

## GET /admin/users

List all users, filterable by `?role=&active=`.

## POST /admin/users

**Purpose**: Create a user account (any role) (FR-ADM-013).

**Request**:
```json
{ "email": "newstudent@example.com", "role": "STUDENT", "name": "Priya Nair", "rollNumber": "199" }
```

## PATCH /admin/users/{userId}

**Purpose**: Update a user's profile, including student-only fields not editable by the student themself (`name`, `rollNumber`, `section`, `mentorName`, `mentorPhone` — see [Database Design §4.2](../SDD/02-Database-Design.md#42-students-table)).

**Request**:
```json
{ "mentorName": "Dr. Anita Rao", "mentorPhone": "+91-9812345678" }
```

## PATCH /admin/users/{userId}/role

Change a user's role. Takes effect on next login (existing JWT remains valid until expiry — see [RBAC Design §6.1](../SDD/03-RBAC-Design.md#61-user-role-change)).

## DELETE /admin/users/{userId}

Deactivate (soft-delete) a user account (`is_active = false`); does not hard-delete history.

---

## Scope Note

Every admin endpoint above maps to an in-scope capability from [Scope §"What the System WILL Do"](../SRS/03-Scope.md#what-the-system-will-do). No endpoints for out-of-scope features (hostel, mess, transport, third-party LMS, video hosting) exist in this API.

## Related Documentation

- [Admin Dashboard](../SDD/08-Admin-Dashboard.md)
- [Batch Allocation](../SDD/05-Batch-Allocation.md)
- [Error Codes](Error-Codes.md)
- [API Index](API-Index.md)
