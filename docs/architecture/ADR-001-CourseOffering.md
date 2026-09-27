# ADR-001: CourseOffering as a Separate Entity from Course

**Date**: 2026-01-01
**Status**: Accepted
**Author**: Architecture Team

## Context

The system needs to represent both the general catalog of vocational courses (e.g. "Java Programming") and the semester-specific instances of those courses that students actually enroll in (e.g. "Java Programming, Spring 2026", with its own capacity, dates, batches, and grading scheme).

## Problem

Should `Courses` and `CourseOfferings` be a single table, or two separate entities?

### Options Considered

**Option 1: Single `Courses` table with semester fields**
- Add `semester`, `max_capacity`, `start_date`, `end_date` directly to `Courses`
- Pros: Fewer tables, simpler joins
- Cons: A course offered across multiple semesters requires duplicate rows with identical `course_name`/`course_code`, breaking the idea of a stable course catalog; historical offerings become indistinguishable from the catalog definition

**Option 2: Separate `Courses` (catalog) and `CourseOfferings` (semester instance) tables**
- `Courses` holds the stable definition (code, name, description, duration)
- `CourseOfferings` holds the per-semester instance (dates, capacity, enrollment count), with a foreign key to `Courses`
- Pros: Clean separation between "what a course is" and "when/how it runs"; supports the same course running every semester without duplicating its definition; supports historical reporting across semesters
- Cons: One additional join for most queries

## Decision

**We choose Option 2: Separate `Courses` and `CourseOfferings` entities**, as implemented in [Database Design §4.5–4.6](../SDD/02-Database-Design.md#45-courses-table).

## Rationale

### Data Integrity
- A course's identity (code, name, description) is defined once and referenced by every semester it runs, rather than re-entered
- `UNIQUE(course_id, semester)` on `CourseOfferings` prevents duplicate offerings of the same course in the same semester ([Database Design §4.6](../SDD/02-Database-Design.md#46-courseofferings-table))

### Reporting
- Historical reporting ("how many students took Java Programming across all semesters") is a straightforward join, rather than requiring de-duplication logic on a merged table

### Consistency with Downstream Entities
- `EnrollmentWindows`, `Enrollments`, `Batches`, `GradingSchemes`, and `FeedbackForms` all key off `course_offering_id`, not `course_id` — because these are inherently semester-scoped concepts (see [Database Design §4.7–4.9, 4.12, 4.17](../SDD/02-Database-Design.md#4-table-definitions)). A merged table would force these to carry an implicit semester join through the course row instead of a direct foreign key.

## Consequences

### Positive
✓ Course catalog stays stable and simple to browse/manage independent of scheduling
✓ Running the same course in multiple semesters requires no duplicate catalog data
✓ Per-semester capacity, dates, and enrollment counts are cleanly scoped
✓ Historical/cross-semester reporting is straightforward

### Negative
✗ One additional join (`course_offerings.course_id → courses.course_id`) for any query that needs both catalog and semester detail
✗ Two admin screens conceptually (manage the catalog vs. configure an offering), though the Admin Dashboard presents them as one guided flow (see [Admin Dashboard §4–5](../SDD/08-Admin-Dashboard.md#4-semester-management))

## Related Decisions

- [ADR-002: Random Batch Allocation Algorithm](ADR-002-RandomAllocation.md)
- [ADR-004: Gradebook Draft/Publish Lifecycle](ADR-004-Gradebook.md)

## References

- [Database Design — Courses / CourseOfferings](../SDD/02-Database-Design.md#45-courses-table)
- [Admin Dashboard — Semester & Offering Management](../SDD/08-Admin-Dashboard.md#4-semester-management)
- [Functional Requirements — FR-ADM-003](../SRS/05-Functional-Requirements.md#4-admin-modules)
