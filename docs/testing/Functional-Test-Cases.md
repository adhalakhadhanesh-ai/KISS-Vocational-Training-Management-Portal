# Functional Test Cases

## Overview

This document expands the sample cases in [Test-Plan §10](Test-Plan.md#10-test-cases) into a full functional test suite covering every module named in [Functional Requirements](../SRS/05-Functional-Requirements.md). Case IDs are grouped by module and traceable to their FR.

## Notation

- **Priority**: Critical / High / Medium
- **Precondition**: State the system must be in before the test starts
- **Steps**: User-observable actions
- **Expected**: Observable result

---

## 1. Authentication (FR-AUTH-001 to FR-AUTH-004)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-AUTH-001 | Student login (valid) | Student account exists | Enter valid email/password, click Login | Dashboard displays, JWT stored | Critical |
| TC-AUTH-002 | Login with invalid password | Student account exists | Enter valid email, wrong password | `INVALID_CREDENTIALS`, no token issued | Critical |
| TC-AUTH-003 | Account lockout | 5 prior failed attempts | Attempt 6th login | `ACCOUNT_LOCKED` returned regardless of password correctness | Critical |
| TC-AUTH-004 | Session timeout | Logged in, idle 30+ minutes | Attempt protected action | Redirected to login | High |
| TC-AUTH-005 | Token refresh | Access token near expiry | Call `/auth/refresh` with valid refresh token | New access token issued | High |
| TC-AUTH-006 | Password reset request | Registered email | Submit `/auth/password-reset-request` | 200 OK regardless of whether email exists (no enumeration) | Critical |
| TC-AUTH-007 | Password reset completion | Valid, unexpired reset token | Submit new password meeting policy | Password updated, old password no longer works | Critical |
| TC-AUTH-008 | Weak password rejected | — | Submit password without special character | `WEAK_PASSWORD` | High |

## 2. Student — Dashboard & Profile (FR-STU-001, FR-STU-002)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-STU-001 | Dashboard loads | Student enrolled and allocated | Login as student | Dashboard shows enrolled course, batch, progress snapshot within 2s | Critical |
| TC-STU-002 | Profile is read-only | Logged in as student | Attempt `PATCH /student/profile` with `name` field | `READ_ONLY_FIELD` (422); name unchanged | High |
| TC-STU-003 | Profile contact update | Logged in as student | `PATCH /student/profile` with new phone/address | 200 OK, fields updated | Medium |
| TC-STU-004 | Mentor fields visible, not editable | Student has mentor assigned | View profile | `mentorName`/`mentorPhone` shown; not present in editable form | Medium |

## 3. Student — Enrollment (FR-STU-003)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-ENR-001 | Enroll during active window | Window active for student's roll range | Select course, confirm | Enrollment created, confirmation email sent | Critical |
| TC-ENR-002 | Enroll outside window | Window not active | Attempt enrollment | `ENR_001`, enrollment rejected | Critical |
| TC-ENR-003 | Duplicate enrollment | Already enrolled this semester | Attempt second enrollment | `ENR_003` (409) | Critical |
| TC-ENR-004 | Enroll at full capacity | Course offering at `max_capacity` | Attempt enrollment | `ENR_004` | High |
| TC-ENR-005 | Roll number outside window range | Window scoped to roll 001–100; student is 150 | Attempt enrollment | `ENR_002` | High |
| TC-ENR-006 | Drop before allocation | Enrolled, allocation not yet run | `DELETE /student/enrollments/{id}` | Enrollment removed | Medium |
| TC-ENR-007 | Drop after allocation | Enrolled and allocated | `DELETE /student/enrollments/{id}` | `ENR_ALREADY_ALLOCATED` (409) | Medium |

## 4. Batch Allocation (FR-ADM-010)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-ALLOC-001 | Run allocation, sufficient capacity | Window closed, enrollments ≤ total batch capacity | Admin runs allocation | All students allocated, no batch exceeds capacity | Critical |
| TC-ALLOC-002 | Run allocation, insufficient capacity | Enrollments > total batch capacity | Admin runs allocation | `ALLOC_INSUFFICIENT_CAPACITY`, allocation not partially applied | Critical |
| TC-ALLOC-003 | Run allocation, no enrollments | Zero enrollments for offering | Admin runs allocation | `ALLOC_NO_ENROLLMENTS` | Medium |
| TC-ALLOC-004 | Allocation reproducibility | Same enrollment set, same seed | Re-run algorithm in test harness | Identical batch assignment | Medium |
| TC-ALLOC-005 | Allocation report accuracy | Allocation completed | View allocation report | Student count per batch matches `allocations` table | High |

## 5. Batch Switching (FR-ADM-011)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-SWITCH-001 | Request switch | Student allocated, no prior switch request | Submit switch request with reason | Request created, status `PENDING` | High |
| TC-SWITCH-002 | Second switch request rejected | Prior switch request exists (any status) | Submit another switch request for same course | `SWITCH_ALREADY_REQUESTED` (409) | Medium |
| TC-SWITCH-003 | Approve switch | Pending request, target batch has capacity | Admin approves | Allocation updated to target batch, student notified | High |
| TC-SWITCH-004 | Approve switch, target full | Pending request, target batch at capacity | Admin attempts approval | `SWITCH_TARGET_FULL` (400) | High |
| TC-SWITCH-005 | Reject switch | Pending request | Admin rejects with comment | Student remains in original batch, notified | Medium |

## 6. Teacher — Attendance (FR-TEA-002)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-ATT-001 | Mark attendance, current date | Teacher assigned to batch | Submit attendance for today | Records saved, one row per student | Critical |
| TC-ATT-002 | Mark attendance, future date rejected | — | Submit attendance for tomorrow | `ATT_FUTURE_DATE` (400) | High |
| TC-ATT-003 | Bulk mark present, toggle absents | — | Mark all present, toggle 2 to absent | Only toggled students marked absent | Medium |
| TC-ATT-004 | Edit within 7-day window | Attendance entered 3 days ago | `PATCH` attendance record | Updated successfully | Medium |
| TC-ATT-005 | Edit after 7-day window | Attendance entered 10 days ago | `PATCH` attendance record (non-admin) | `ATT_EDIT_WINDOW_EXPIRED` (403) | Medium |
| TC-ATT-006 | Unassigned teacher blocked | Teacher not assigned to batch | Attempt to mark attendance | `NOT_ASSIGNED_TEACHER` (403) | Critical |

## 7. Teacher — Gradebook (FR-TEA-003, FR-TEA-004)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-GRADE-001 | Enter marks, draft state | Gradebook in `DRAFT` | Submit marks within component max | Saved, state remains `DRAFT` | Critical |
| TC-GRADE-002 | Marks exceed component max | — | Submit marks above `maxMarks` | `GRADE_EXCEEDS_MAX` (422) | Critical |
| TC-GRADE-003 | Publish with at least one mark | ≥1 student has ≥1 mark | Teacher publishes | State → `PUBLISHED`, students notified | Critical |
| TC-GRADE-004 | Publish with no marks entered | 0 marks entered | Teacher publishes | `GRADEBOOK_EMPTY` (422) | High |
| TC-GRADE-005 | Edit after publish | Gradebook `PUBLISHED` | Attempt `POST /teacher/grades` | `GRADEBOOK_LOCKED` (409) | Critical |
| TC-GRADE-006 | Overall grade calculation | Components weighted 10/30/60 | Publish with full marks | Overall grade = weighted sum, matches manual calculation | Critical |
| TC-GRADE-007 | Student cannot see draft marks | Gradebook `DRAFT` | Student requests `/student/grades/{courseId}` | `GRADES_NOT_PUBLISHED` (404), no marks leaked | Critical |

## 8. Teacher — Course Completion (FR-TEA-005)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-COMPLETE-001 | Mark complete after end date | `end_date` passed, gradebook `PUBLISHED` | Teacher marks course complete | Status `COMPLETED`, certificates queued for all students | Critical |
| TC-COMPLETE-002 | Mark complete before end date | `end_date` in future | Attempt to mark complete | `COURSE_NOT_ENDED` (400) | High |
| TC-COMPLETE-003 | Mark complete with unpublished gradebook | `end_date` passed, gradebook `DRAFT` | Attempt to mark complete | `GRADEBOOK_NOT_PUBLISHED` (409) | High |

## 9. Certificates (FR-STU-007, FR-SYS-005)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-CERT-001 | Certificate generated on completion | Course marked complete | Wait for generation job | Certificate appears within 1 hour, unique `cert_number` | Critical |
| TC-CERT-002 | Certificate download | Certificate generated | `GET /student/certificates/{id}/download` | Valid PDF returned with all required fields | High |
| TC-CERT-003 | Public certificate verification | Valid `certNumber` | `GET /certificates/verify/{certNumber}` | Returns issuance details without requiring auth | Medium |
| TC-CERT-004 | Verify invalid certificate number | — | Submit malformed/nonexistent number | `CERT_INVALID_NUMBER` (400) or `CERT_NOT_FOUND` (404) | Medium |

## 10. Grievances & Feedback (FR-STU-005, FR-STU-006)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-GRIEV-001 | Submit grievance | Logged in as student | Submit with category/description/severity | Grievance created, status `SUBMITTED` | High |
| TC-GRIEV-002 | Status change notifies student | Grievance `UNDER_REVIEW` | Admin updates to `RESOLVED` | Student receives email notification | Medium |
| TC-FEED-001 | Submit feedback, anonymous | Enrolled in course | Submit feedback response | Stored without student identifier on the response row | High |
| TC-FEED-002 | Feedback for non-enrolled course rejected | Not enrolled in course | Attempt feedback submission | 403/404 (course not accessible) | Medium |

## 11. Admin — Configuration (FR-ADM-002 to FR-ADM-009)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-ADM-001 | Create course offering, capacity matches batches | — | Create offering + batches summing to `max_capacity` | Created successfully | Critical |
| TC-ADM-002 | Create course offering, capacity mismatch | — | Batches sum ≠ `max_capacity` | `CAPACITY_MISMATCH` (400) | High |
| TC-ADM-003 | Duplicate offering same semester | Offering exists for course+semester | Attempt to create again | `OFFERING_EXISTS` (409) | Medium |
| TC-ADM-004 | Grading scheme weight validation | — | Submit components summing to 90% | `WEIGHT_SUM_INVALID` (422) | Critical |
| TC-ADM-005 | Grading scheme weight valid | — | Submit components summing to 100% | Created successfully | Critical |
| TC-ADM-006 | Staggered enrollment windows | Two windows, non-overlapping roll ranges | Create both windows | Both active independently, correct students see correct window | High |

## 12. Admin — User Management (FR-ADM-013)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-USER-001 | Create user | Admin logged in | `POST /admin/users` with role STUDENT | User created, can log in | High |
| TC-USER-002 | Change user role | User exists | `PATCH /admin/users/{id}/role` | New role active on next login; current session unaffected until expiry | Medium |
| TC-USER-003 | Deactivate user | User exists | `DELETE /admin/users/{id}` | `is_active = false`; login rejected | High |
| TC-USER-004 | Admin edits student mentor fields | Student exists | `PATCH /admin/users/{id}` with `mentorName`/`mentorPhone` | Fields updated, visible on student profile | Medium |

## 13. Reports (FR-ADM-012)

| ID | Title | Precondition | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-REPORT-001 | Certificate report export | Certificates exist | `GET /admin/reports?type=certificate&format=csv` | CSV matches `certificates` table | Medium |
| TC-REPORT-002 | Student report date filter | — | Request with `date_from`/`date_to` | Only records in range returned | Medium |

---

## Coverage Summary

| Module | Test Cases | FR Coverage |
|--------|-----------|-------------|
| Authentication | 8 | FR-AUTH-001 to 004 |
| Student Dashboard/Profile | 4 | FR-STU-001, FR-STU-002 |
| Enrollment | 7 | FR-STU-003 |
| Allocation | 5 | FR-ADM-010 |
| Batch Switching | 5 | FR-ADM-011 |
| Attendance | 6 | FR-TEA-002 |
| Gradebook | 7 | FR-TEA-003, FR-TEA-004 |
| Course Completion | 3 | FR-TEA-005 |
| Certificates | 4 | FR-STU-007, FR-SYS-005 |
| Grievances/Feedback | 4 | FR-STU-005, FR-STU-006 |
| Admin Configuration | 6 | FR-ADM-002 to 009 |
| User Management | 4 | FR-ADM-013 |
| Reports | 2 | FR-ADM-012 |
| **Total** | **65** | 34/34 Functional Requirements |

## Related Documents

- [Test Plan](Test-Plan.md)
- [Security Test Cases](Security-Test-Cases.md)
- [Performance Test Cases](Performance-Test-Cases.md)
- [Functional Requirements](../SRS/05-Functional-Requirements.md)
- [Error Codes](../API/Error-Codes.md)
