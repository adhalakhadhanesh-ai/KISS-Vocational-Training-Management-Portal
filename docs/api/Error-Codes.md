# Error Codes — Complete Reference

## Overview

This is the single consolidated reference for every error code returned by the API. All errors follow the response envelope defined in [API Index](API-Index.md#error-response-format):

```json
{
  "status": 400,
  "error": "ERROR_CODE",
  "message": "Human-readable message",
  "timestamp": "2026-01-15T10:30:00Z",
  "path": "/api/...",
  "details": null
}
```

## HTTP Status Codes

| Status | Meaning |
|--------|---------|
| 400 | Bad Request — malformed input |
| 401 | Unauthorized — missing/invalid/expired token |
| 403 | Forbidden — authenticated but not permitted |
| 404 | Not Found |
| 409 | Conflict — duplicate or state conflict |
| 422 | Unprocessable Entity — validation failure |
| 429 | Too Many Requests — rate limited |
| 500 | Internal Server Error |

## Authentication Errors

| Code | Status | Meaning |
|------|--------|---------|
| INVALID_CREDENTIALS | 400 | Email/password incorrect |
| MISSING_FIELD | 400 | Required field missing |
| ACCOUNT_LOCKED | 403 | Account locked after 5 failed attempts |
| UNAUTHORIZED | 401 | No token provided or invalid |
| TOKEN_EXPIRED | 401 | Token has expired |
| INVALID_TOKEN | 400/401 | Refresh or password-reset token invalid or expired |
| WEAK_PASSWORD | 400 | Password doesn't meet requirements |
| TOO_MANY_REQUESTS | 429 | Rate limit exceeded |

Full detail: [Authentication API](Authentication.md).

## Enrollment Errors

| Code | Status | Meaning |
|------|--------|---------|
| ENR_001 | 400 | Enrollment window not active |
| ENR_002 | 400 | Not eligible for this window (roll number out of range) |
| ENR_003 | 409 | Already enrolled in course |
| ENR_004 | 400 | Course at capacity |
| ENR_005 | 404 | Invalid course offering / enrollment |
| ENR_006 | 404 | Student not found |
| ENR_007 | 500 | Enrollment failed (server error) |
| ENR_ALREADY_ALLOCATED | 409 | Cannot drop — batch allocation has already run |

Full detail: [Enrollment Design §10.1](../SDD/04-Enrollment-Design.md#101-error-codes).

## Allocation & Batch Switching Errors

| Code | Status | Meaning |
|------|--------|---------|
| ALLOC_INSUFFICIENT_CAPACITY | 400 | Enrollments exceed total batch capacity |
| ALLOC_NO_ENROLLMENTS | 400 | No enrollments exist for the course offering |
| SWITCH_ALREADY_REQUESTED | 409 | Student already has an open switch request for this course |
| SWITCH_TARGET_FULL | 400 | Target batch is at capacity |

Full detail: [Batch Allocation §8](../SDD/05-Batch-Allocation.md#8-error-scenarios).

## Attendance Errors

| Code | Status | Meaning |
|------|--------|---------|
| ATT_FUTURE_DATE | 400 | Cannot mark attendance for a future date |
| ATT_ALREADY_MARKED | 409 | Attendance already recorded for this batch/date |
| ATT_EDIT_WINDOW_EXPIRED | 403 | 7-day edit window has passed; requires admin override |

## Gradebook Errors

| Code | Status | Meaning |
|------|--------|---------|
| GRADE_EXCEEDS_MAX | 422 | `marksObtained` exceeds the component's `maxMarks` |
| GRADEBOOK_LOCKED | 409 | Gradebook already `PUBLISHED`; cannot be edited |
| GRADEBOOK_EMPTY | 422 | No student has any mark entered; publish rejected |
| GRADES_NOT_PUBLISHED | 404 | Requested grades before the gradebook was published |

## Course Completion & Certificate Errors

| Code | Status | Meaning |
|------|--------|---------|
| COURSE_NOT_ENDED | 400 | `course_offerings.end_date` has not yet passed |
| GRADEBOOK_NOT_PUBLISHED | 409 | Gradebook must be published before the course can be marked complete |
| CERT_GENERATION_FAILED | 500 | PDF generation failed mid-process (see [Certificate Workflow §9.1](../SDD/10-Certificate-Workflow.md#91-certificate-generation-fails)) |
| CERT_NOT_FOUND | 404 | Certificate ID does not exist or does not belong to the caller |
| CERT_INVALID_NUMBER | 400 | Certificate number failed verification |

## Admin / Configuration Errors

| Code | Status | Meaning |
|------|--------|---------|
| CAPACITY_MISMATCH | 400 | Sum of batch capacities does not equal course offering `max_capacity` |
| OFFERING_EXISTS | 409 | Course already offered this semester (`(course_id, semester)` unique) |
| WEIGHT_SUM_INVALID | 422 | Grading scheme component weights do not sum to 100% |
| READ_ONLY_FIELD | 422 | Student attempted to modify an admin-managed profile field |

## Generic Validation

| Code | Status | Meaning |
|------|--------|---------|
| VALIDATION_ERROR | 422 | One or more request fields failed validation (see `details`) |
| NOT_FOUND | 404 | Requested resource does not exist |
| FORBIDDEN | 403 | Authenticated but not authorized for this resource |
| INTERNAL_ERROR | 500 | Unexpected server error; error ID logged for tracking |

## Related Documentation

- [API Index](API-Index.md)
- [Authentication API](Authentication.md)
- [Enrollment Design](../SDD/04-Enrollment-Design.md)
- [Batch Allocation](../SDD/05-Batch-Allocation.md)
- [Certificate Workflow](../SDD/10-Certificate-Workflow.md)
