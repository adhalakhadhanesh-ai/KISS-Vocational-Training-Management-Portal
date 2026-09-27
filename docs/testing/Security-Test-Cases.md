# Security Test Cases

## Overview

Expands [Test-Plan §8](Test-Plan.md#8-security-testing) into concrete, executable security test cases covering authentication, authorization, injection, and data protection, aligned to [Non-Functional Requirements §2 (Security)](../SRS/06-Non-Functional-Requirements.md#2-security-requirements) and [RBAC Design](../SDD/03-RBAC-Design.md).

## 1. Authentication Security

| ID | Title | Steps | Expected | Priority |
|----|-------|-------|----------|----------|
| TC-SEC-AUTH-001 | Reject invalid credentials | Login with wrong password | `INVALID_CREDENTIALS`, no token issued | Critical |
| TC-SEC-AUTH-002 | Account lockout after 5 failures | 5 failed logins, then a 6th with correct password | `ACCOUNT_LOCKED` even with correct password | Critical |
| TC-SEC-AUTH-003 | Password stored hashed | Inspect `users.password` column after registration | Value is a bcrypt hash (cost ≥ 10), never plaintext | Critical |
| TC-SEC-AUTH-004 | JWT signature tampering rejected | Modify JWT payload, keep original signature | 401 `INVALID_TOKEN` | Critical |
| TC-SEC-AUTH-005 | Expired token rejected | Use a token past its 24-hour `exp` | 401 `TOKEN_EXPIRED` | Critical |
| TC-SEC-AUTH-006 | No user enumeration on password reset | Submit reset request for non-existent email | Same 200 response and message as for an existing email | High |
| TC-SEC-AUTH-007 | Reset token single-use | Use a reset token twice | Second use rejected with `INVALID_TOKEN` | High |
| TC-SEC-AUTH-008 | Reset token expiry | Use a reset token after 1 hour | Rejected with `INVALID_TOKEN` | High |
| TC-SEC-AUTH-009 | Login rate limiting by IP | 6 login attempts within 15 minutes from one IP | 429 `TOO_MANY_REQUESTS` on 6th attempt | High |
| TC-SEC-AUTH-010 | Logout invalidates client-side token | Logout, then reuse the same token from browser storage | Client discards token; server treats subsequent use per token expiry (stateless — see note below) | Medium |

> **Note on TC-SEC-AUTH-010**: Because sessions are stateless JWTs (see [System Architecture §6](../SDD/01-System-Architecture.md#6-authentication--authorization-flow)), logout clears the token client-side only; the token itself remains cryptographically valid until natural expiry. This is a documented design constraint, not a defect — verify it matches [RBAC Design §9.3](../SDD/03-RBAC-Design.md#93-session-management).

## 2. Authorization (RBAC) Security

| ID | Title | Steps | Expected | Priority |
|----|-------|-------|----------|----------|
| TC-SEC-RBAC-001 | Student blocked from teacher endpoints | Authenticated as STUDENT, call `POST /teacher/attendance` | 403 Forbidden | Critical |
| TC-SEC-RBAC-002 | Student blocked from admin endpoints | Authenticated as STUDENT, call `POST /admin/allocation/run` | 403 Forbidden | Critical |
| TC-SEC-RBAC-003 | Teacher blocked from admin endpoints | Authenticated as TEACHER, call `POST /admin/users` | 403 Forbidden | Critical |
| TC-SEC-RBAC-004 | Teacher blocked from unassigned batch | Teacher not assigned to batch X, call `POST /teacher/attendance` for batch X | 403 `NOT_ASSIGNED_TEACHER` | Critical |
| TC-SEC-RBAC-005 | Student cannot view another student's progress | Student A requests `/student/progress/{courseId}` scoped to Student B's allocation | 403 Forbidden (resource-level check per [RBAC Design §5.1](../SDD/03-RBAC-Design.md#51-student-module)) | Critical |
| TC-SEC-RBAC-006 | Student cannot self-elevate role | Student attempts `PATCH /admin/users/{self}/role` | 403 Forbidden (endpoint requires ADMIN role) | Critical |
| TC-SEC-RBAC-007 | Admin has full access | Authenticated as ADMIN, call one endpoint per role | All succeed subject to normal validation | High |
| TC-SEC-RBAC-008 | Unauthenticated access blocked | No `Authorization` header, call any protected endpoint | 401 Unauthorized | Critical |
| TC-SEC-RBAC-009 | Role change takes effect only on next login | Admin changes a user's role mid-session | Existing JWT keeps old role's access until expiry; new login reflects new role | Medium |
| TC-SEC-RBAC-010 | Student cannot edit admin-managed profile fields | Student calls `PATCH /student/profile` with `mentorName` | 422 `READ_ONLY_FIELD` | High |

## 3. Injection & Input Validation

| ID | Title | Steps | Expected | Priority |
|----|-------|-------|----------|----------|
| TC-SEC-INJ-001 | SQL injection in login | Submit `' OR '1'='1` as email | Login rejected, no query error leaked, parameterized query confirmed in logs | Critical |
| TC-SEC-INJ-002 | SQL injection in search/filter params | Submit `1; DROP TABLE users;--` as a filter value | Input treated as literal string; no schema change occurs | Critical |
| TC-SEC-INJ-003 | XSS in grievance description | Submit `<script>alert(1)</script>` as grievance text | Stored and rendered as escaped text, not executed | Critical |
| TC-SEC-INJ-004 | XSS in feedback text response | Submit HTML/script tags in a free-text feedback answer | Escaped on render (admin feedback view) | High |
| TC-SEC-INJ-005 | CSRF token validation | Submit a state-changing request without a valid CSRF/anti-forgery context where applicable | Rejected (see [System Architecture §2.4](../SDD/01-System-Architecture.md#24-security-first)) | High |
| TC-SEC-INJ-006 | Oversized payload rejected | Submit a grievance description far beyond expected length | 422 Validation Error, not a 500 crash | Medium |
| TC-SEC-INJ-007 | Malformed JSON rejected gracefully | Submit invalid JSON body | 400 Bad Request, no stack trace exposed to client | Medium |

## 4. Data Protection

| ID | Title | Steps | Expected | Priority |
|----|-------|-------|----------|----------|
| TC-SEC-DATA-001 | TLS enforced | Attempt plain HTTP request to API host | Connection refused or redirected to HTTPS | Critical |
| TC-SEC-DATA-002 | Sensitive fields absent from logs | Trigger a login and a failed login; inspect application logs | Passwords and full tokens never appear in logs | Critical |
| TC-SEC-DATA-003 | Error responses don't leak internals | Trigger a 500 error | Response contains a generic message and error ID, not a stack trace | High |
| TC-SEC-DATA-004 | Certificate PDF not guessable | Attempt to download a certificate by guessing sequential IDs | `cert_number` is non-sequential (per [Certificate Workflow §4](../SDD/10-Certificate-Workflow.md#4-certificate-id-generation)); guessed IDs fail ownership check | High |
| TC-SEC-DATA-005 | Feedback responses are anonymous | Inspect `feedback_responses` rows | No `student_id` column; only `allocation_id` (see [Database Design §4.19](../SDD/02-Database-Design.md#419-feedbackresponses-table)) | High |
| TC-SEC-DATA-006 | Database encryption at rest | Confirm Supabase-managed encryption setting | Enabled | Medium |
| TC-SEC-DATA-007 | Audit log completeness | Perform a role change, an allocation run, and a grade publish | All three appear in the audit log with timestamp, user, action, affected record (per [RBAC Design §8](../SDD/03-RBAC-Design.md#8-audit-logging)) | High |
| TC-SEC-DATA-008 | Audit log immutability | Attempt to modify or delete an audit log row via the API | No endpoint exists to do so; direct DB access is the only path (documented, not exposed) | Medium |

## 5. Rate Limiting & Abuse Prevention

| ID | Title | Steps | Expected | Priority |
|----|-------|-------|----------|----------|
| TC-SEC-RATE-001 | General endpoint rate limit | Exceed 100 requests/minute as one user | 429 with `X-RateLimit-*` headers | Medium |
| TC-SEC-RATE-002 | Rate limit headers present | Make a normal request | Response includes `X-RateLimit-Limit`, `-Remaining`, `-Reset` | Low |

## Test Environment Notes

Security tests run against the **Staging** environment (see [Test-Plan §9.2](Test-Plan.md#92-test-environment)) with production-equivalent security middleware enabled — never against Production with live student data.

## Related Documents

- [Test Plan](Test-Plan.md)
- [Functional Test Cases](Functional-Test-Cases.md)
- [RBAC Design](../SDD/03-RBAC-Design.md)
- [Non-Functional Requirements](../SRS/06-Non-Functional-Requirements.md)
- [Error Codes](../API/Error-Codes.md)
