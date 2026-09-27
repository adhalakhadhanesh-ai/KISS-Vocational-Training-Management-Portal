# ADR-005: Automatic, Asynchronous Certificate Generation on Course Completion

**Date**: 2026-01-01
**Status**: Accepted
**Author**: Architecture Team

## Context

Every student who completes a course is entitled to a certificate ([Functional Requirements — FR-STU-007](../SRS/05-Functional-Requirements.md#2-student-modules)). Certificate generation involves rendering a PDF per student and must scale to a full batch (up to ~100 students) completing at once, while meeting the throughput target of ≥ 100 certificates/minute ([Non-Functional Requirements — NFR-PER-002](../SRS/06-Non-Functional-Requirements.md#nfr-per-002-throughput)).

## Problem

When and how should certificates be generated: manually by an admin, synchronously at the moment a teacher marks a course complete, or asynchronously in the background?

### Options Considered

**Option 1: Manual generation, admin-triggered per student**
- Admin reviews each completed student and clicks "Generate Certificate"
- Pros: Full manual control, no unwanted certificates
- Cons: Doesn't scale to batches of ~100 students; adds a manual step admins must not forget; delays students receiving their certificate

**Option 2: Automatic, synchronous generation on "Mark Complete"**
- `POST /teacher/course/{batchId}/complete` blocks until every student's PDF is rendered and saved
- Pros: Simple request/response flow, no background job infrastructure
- Cons: A 100-student batch's PDF rendering would make the completion request slow and fragile — a single PDF failure could fail the entire request, and the response-time target for form submission (≤ 1 s, [Non-Functional Requirements — NFR-PER-001](../SRS/06-Non-Functional-Requirements.md#nfr-per-001-response-time)) cannot be met while rendering dozens of PDFs inline

**Option 3: Automatic, asynchronous generation on "Mark Complete"**
- Marking a course complete immediately updates `allocations.status` and returns a fast response; certificate generation for each student is queued and processed in the background (see [Certificate Workflow §2](../SDD/10-Certificate-Workflow.md#2-certificate-generation-flow))
- Pros: Fast, predictable completion request; a single certificate failure doesn't block the rest of the batch; meets the ≥ 100/minute throughput target by processing in parallel/batches
- Cons: Certificates are not instantly available — students must wait (bounded by the ≤ 10 minutes/1000 students target, [Non-Functional Requirements — NFR-PER-004](../SRS/06-Non-Functional-Requirements.md#nfr-per-004-batch-processing-performance))

## Decision

**We choose Option 3: automatic, asynchronous generation triggered by course completion**, as implemented in [Certificate Workflow §2](../SDD/10-Certificate-Workflow.md#2-certificate-generation-flow) and [Teacher Workspace §7](../SDD/06-Teacher-Workspace.md#7-course-completion-sub-module).

## Rationale

### No Manual Step to Forget
- Every completed student automatically receives a certificate; there is no admin action that can be skipped or delayed

### Responsiveness
- The teacher-facing completion request returns immediately (`certificatesQueued` count), independent of how long PDF rendering for the full batch takes

### Fault Isolation
- One student's PDF generation failure (`CERT_GENERATION_FAILED` — [Error Codes](../API/Error-Codes.md#course-completion--certificate-errors)) does not block or roll back certificates for the rest of the batch

### Meets Throughput Targets
- Background processing can be scaled/parallelized independently of the request/response cycle to satisfy the ≥ 100 certificates/minute target

## Consequences

### Positive
✓ Fast, reliable completion action for teachers
✓ Certificates generated for every eligible student with no manual trigger
✓ Scales to large batches without degrading the completion request

### Negative
✗ Students do not receive their certificate the instant the course is marked complete — there is a bounded delay
✗ Requires a background job/worker mechanism (queue or scheduled task) rather than a simple synchronous call, adding operational surface area

## Mitigations

- Students are notified by email once their specific certificate is ready, rather than needing to poll ([Certificate Workflow §2](../SDD/10-Certificate-Workflow.md#2-certificate-generation-flow))
- `GET /student/certificates` reflects generation status so the UI can show "processing" vs. "ready" per certificate
- Course completion validates preconditions (`end_date` passed, gradebook `PUBLISHED`) synchronously before any generation is queued, so students only wait for PDF rendering — not for correctable validation errors

## Related Decisions

- [ADR-004: Gradebook Draft/Publish Lifecycle](ADR-004-Gradebook.md) — the `PUBLISHED` precondition this ADR depends on

## References

- [Certificate Workflow](../SDD/10-Certificate-Workflow.md)
- [Teacher Workspace — Course Completion](../SDD/06-Teacher-Workspace.md#7-course-completion-sub-module)
- [Non-Functional Requirements — Performance](../SRS/06-Non-Functional-Requirements.md#1-performance-requirements)
- [Functional Requirements — FR-STU-007](../SRS/05-Functional-Requirements.md#2-student-modules)
