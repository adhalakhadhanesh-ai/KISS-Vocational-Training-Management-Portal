# Admin Dashboard — Module Design

## 1. Overview

This document describes the design of the **Admin Dashboard** ("Command Center"), the real-time operational view for admins, and how it ties together the configuration modules (semesters, course offerings, grading schemes, enrollment windows). It implements [FR-ADM-001 to FR-ADM-004](../SRS/05-Functional-Requirements.md#4-admin-modules). Allocation, batch switching, reports, and certificates each have deeper coverage elsewhere and are referenced, not repeated, here.

## 2. Module Layout

```
Admin Dashboard
├─ Command Center (trackers)     (FR-ADM-001)
├─ Semester Management            (FR-ADM-002)
├─ Course Offering Management     (FR-ADM-003)
└─ Seat / Batch Configuration     (FR-ADM-004, FR-ADM-005)
```

## 3. Command Center Trackers

Implements FR-ADM-001.

| Tracker | Data source | Refresh |
|---------|-------------|---------|
| Live Enrollment | `enrollments` count vs. `course_offerings.max_capacity` per offering | ≤ 1 min |
| Allocation | `allocations` count vs. `enrollments` count; batches filled vs. total | ≤ 1 min |
| Feedback | `feedback_responses` count, average `rating` | ≤ 1 min |
| Grievance | `grievances` grouped by `status`, average resolution time (`updated_at - created_at` for `RESOLVED`/`CLOSED`) | ≤ 1 min |

Each tracker supports drill-down to the underlying record list. Dashboard load target: < 3 seconds (see [FR-ADM-001 acceptance criteria](../SRS/05-Functional-Requirements.md#4-admin-modules)).

**Query pattern** (enrollment tracker):
```sql
SELECT co.course_offering_id, co.max_capacity, co.current_enrollment,
       ROUND(co.current_enrollment * 100.0 / co.max_capacity, 1) AS fill_pct
FROM course_offerings co
WHERE co.semester = ?;
```

## 4. Semester Management

Implements FR-ADM-002. A semester is not a standalone table in the current schema — it is the `semester` string on `course_offerings` (see [Database Design §4.6](02-Database-Design.md#46-courseofferings-table)) — so "creating a semester" in the UI means setting up its first course offering(s), grading schemes, and enrollment windows against that semester label in one guided flow.

**Flow**:
```
Admin enters semester name (e.g. "Spring 2026")
    → Admin adds one or more course offerings under that semester
    → Admin defines grading scheme per offering
    → Admin defines enrollment window per offering
    → Configuration reviewed and saved
```

## 5. Course Offering, Seat & Batch Configuration

Implements FR-ADM-003, FR-ADM-004, FR-ADM-005, FR-ADM-006.

**Flow**:
```
Select base Course → set max_capacity for the CourseOffering
    → define batch structure (N batches, capacity per batch; batch capacities sum to max_capacity)
    → assign a Teacher to each Batch
    → set the EnrollmentWindow (dates + roll-number range)
```

**Constraint enforced at save time**: `Σ batch.capacity == course_offering.max_capacity` (validated server-side; see [Error Codes](../API/Error-Codes.md) `CAPACITY_MISMATCH`). Seat/batch configuration is locked once an allocation run has been executed for that offering.

**Primary endpoints**: `POST /admin/semesters`, `POST /admin/course-offerings`, `GET /admin/course-offerings`, `POST /admin/grading-schemes`, `POST /admin/enrollment-windows` — see [Admin API](../API/Admin.md).

## 6. Related Modules (Cross-References)

The remaining Admin capabilities are documented in full elsewhere so this document stays focused on the dashboard and configuration flow:

| Capability | Document |
|------------|----------|
| Batch allocation (random shuffle) | [Batch Allocation](05-Batch-Allocation.md) |
| Batch switching approval | [Batch Allocation §6](05-Batch-Allocation.md#6-batch-switching) |
| Report generation | [Admin API](../API/Admin.md) |
| Certificate oversight | [Certificate Workflow](10-Certificate-Workflow.md) |
| User management | [Admin API](../API/Admin.md) |

## 7. Access Control

All endpoints in this module require `hasRole('ADMIN')`; there is no partial/self-scoped access at the admin level (see [RBAC Design §5.3](03-RBAC-Design.md#53-admin-module)).

## 8. Related Documents

- [Functional Requirements — Admin Modules](../SRS/05-Functional-Requirements.md#4-admin-modules)
- [Batch Allocation](05-Batch-Allocation.md)
- [Admin API](../API/Admin.md)
- [Database Design](02-Database-Design.md)
