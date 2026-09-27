# ADR-004: Gradebook Draft/Publish Lifecycle

**Date**: 2026-01-01
**Status**: Accepted
**Author**: Architecture Team

## Context

Teachers enter marks incrementally over the course of a semester. Students must never see a mark before the teacher considers it final — a partially-entered or miscalculated grade shown prematurely causes confusion and disputes (see [Grievances](../SDD/02-Database-Design.md#420-grievances-table) category `GRADING`).

## Problem

How should the system prevent students from seeing in-progress grading while still letting teachers save work incrementally?

### Options Considered

**Option 1: No intermediate state — grades visible to students as soon as entered**
- Every `grade_entries` write is immediately visible via the student API
- Pros: Simplest possible implementation, no extra state to manage
- Cons: Directly causes the problem above — a teacher entering 3 of 10 components' marks would show an incomplete, misleadingly low grade to the student

**Option 2: Grades visible only after the course's `end_date`**
- Tie visibility to a date field already on `CourseOfferings`
- Pros: No new state needed
- Cons: Couples grade visibility to scheduling, which is the wrong concern — a teacher may finish grading early and want to publish sooner, or need more time after `end_date`; provides no explicit teacher action or audit trail for "grades are final"

**Option 3: Explicit Draft/Published state machine on a `Gradebook` per batch**
- A `gradebooks` row per batch, `state ∈ {DRAFT, PUBLISHED}`; all mark entry happens against `DRAFT`; a single teacher-triggered `publish` action flips the state and is one-way
- Pros: Explicit, auditable teacher action; decouples grading completion from the calendar; trivial to enforce "never show unpublished grades" as one filter condition
- Cons: One additional table and one additional state to manage

## Decision

**We choose Option 3: an explicit Draft/Published state machine**, as implemented in [Database Design §4.15](../SDD/02-Database-Design.md#415-gradebook-table) and [Teacher Workspace §6](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module).

## Rationale

### Correctness
- Student-facing grade queries filter on a single condition (`gradebooks.state = 'PUBLISHED'`), making the "never leak a draft mark" invariant trivial to enforce consistently everywhere it's needed ([Progress Module §4](../SDD/09-Progress-Module.md#4-draft-grade-visibility-rule))

### Auditability
- Publishing is an explicit, logged teacher action with a timestamp (`published_date`), giving a clear record of when grades became official — useful for grievance resolution

### Teacher Control
- Teachers publish when *they* are confident marks are final, independent of the semester calendar, matching how grading actually happens in practice (in bursts, not evenly across the term)

### Immutability After Publish
- Once published, the gradebook is locked (`GRADEBOOK_LOCKED` on further writes — [Error Codes](../API/Error-Codes.md#gradebook-errors)), which protects published grades from silent post-hoc changes; any correction after publish is an explicit, admin-visible exception process, not a normal write path

## Consequences

### Positive
✓ Strong, simple guarantee that students never see incomplete grading
✓ Clear audit trail of when grades became final
✓ Decoupled from course scheduling — teachers control timing

### Negative
✗ A teacher who publishes with an entry error has no simple "unpublish and fix" path in the MVP — this must go through an admin-mediated correction
✗ One additional table (`gradebooks`) and join for every grade read

## Related Decisions

- [ADR-001: CourseOffering as a Separate Entity](ADR-001-CourseOffering.md)
- [ADR-005: Certificate Generation Trigger](ADR-005-CertificateGeneration.md)

## References

- [Database Design — Gradebook Table](../SDD/02-Database-Design.md#415-gradebook-table)
- [Teacher Workspace — Gradebook Sub-Module](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module)
- [Progress Module — Draft-Grade Visibility Rule](../SDD/09-Progress-Module.md#4-draft-grade-visibility-rule)
- [Functional Requirements — FR-TEA-003, FR-TEA-004](../SRS/05-Functional-Requirements.md#3-teacher-modules)
