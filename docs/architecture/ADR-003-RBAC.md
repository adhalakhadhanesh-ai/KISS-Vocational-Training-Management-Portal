# ADR-003: Stateless JWT-Based RBAC

**Date**: 2026-01-01
**Status**: Accepted
**Author**: Architecture Team

## Context

The system has exactly three roles — Student, Teacher, Admin — each with a distinct set of accessible modules and, within a module, resource-level restrictions (a teacher may only touch their assigned batches; a student may only touch their own records). See [RBAC Design](../SDD/03-RBAC-Design.md) for the full permission matrix.

## Problem

How should role and permission checks be implemented across a stateless, horizontally-scaled backend ([System Architecture §2.2](../SDD/01-System-Architecture.md#22-stateless-design))?

### Options Considered

**Option 1: Server-side session store with role lookup per request**
- Session ID in a cookie, role fetched from a session table (Redis/DB) on every request
- Pros: Role changes take effect immediately; smaller token
- Cons: Requires a shared session store, reintroducing state into a backend designed to be stateless; extra lookup on every request; complicates horizontal scaling

**Option 2: JWT carrying the role claim, verified statelessly**
- Role embedded in the signed JWT at login; every request verifies the signature and reads the role directly from the token — no database lookup for authorization itself
- Pros: No shared state needed; scales horizontally with zero coordination between instances; fast (no extra DB round-trip per request)
- Cons: A role change doesn't take effect until the user's next login (existing token keeps the old role until it expires)

**Option 3: Database-backed permission check on every request (role always re-fetched)**
- Token only proves identity; role is looked up fresh from `users.role` on every request
- Pros: Role changes take effect immediately
- Cons: Adds a database round-trip to every single API call, undermining the performance targets in [Non-Functional Requirements §1](../SRS/06-Non-Functional-Requirements.md#1-performance-requirements); partially reintroduces the coupling Option 1 has

## Decision

**We choose Option 2: JWT carrying the role claim, verified statelessly**, as implemented in [RBAC Design](../SDD/03-RBAC-Design.md) and [System Architecture §2.2–2.3](../SDD/01-System-Architecture.md#22-stateless-design).

## Rationale

### Scalability
- No shared session state between backend instances — any instance can serve any request, satisfying the horizontal-scaling goal in [System Architecture §10.1](../SDD/01-System-Architecture.md#101-horizontal-scaling)

### Performance
- Role and identity are available immediately after signature verification, with no extra database round-trip — supports the ≤ 100 ms simple-query and ≤ 2 s page-load targets in [Non-Functional Requirements §1](../SRS/06-Non-Functional-Requirements.md#1-performance-requirements)

### Security
- Token signature prevents tampering with the embedded role
- Combined with resource-level checks in the Service layer (e.g. `@teacherService.isAssignedTeacher`, `@studentService.isOwnerOrAdmin` — [RBAC Design §5](../SDD/03-RBAC-Design.md#5-module-level-access-control)), role alone is never sufficient for access; every request is checked against both role and resource ownership

## Consequences

### Positive
✓ No shared session infrastructure to operate or scale
✓ Authorization check adds negligible latency
✓ Simple to reason about: role is a claim on a verified token, not a mutable server-side value

### Negative
✗ A role change made by an Admin does not apply to an already-issued token until it expires or the user logs in again ([RBAC Design §6.1](../SDD/03-RBAC-Design.md#61-user-role-change))
✗ There is no way to forcibly invalidate a single token before its natural 24-hour expiry (no server-side revocation list in the MVP)

## Mitigations

- Token lifetime is capped at 24 hours ([Non-Functional Requirements — NFR-SEC-004](../SRS/06-Non-Functional-Requirements.md#nfr-sec-004-session-management)), bounding the exposure window of a stale role claim
- Admins are informed in the UI that a role change takes effect on the affected user's next login, not immediately
- Sensitive role-elevation actions (e.g. promoting to Admin) are themselves Admin-only and audit-logged ([RBAC Design §8](../SDD/03-RBAC-Design.md#8-audit-logging))

## Related Decisions

- [ADR-002: Random Batch Allocation Algorithm](ADR-002-RandomAllocation.md)

## References

- [RBAC Design](../SDD/03-RBAC-Design.md)
- [System Architecture — Stateless Design](../SDD/01-System-Architecture.md#22-stateless-design)
- [Non-Functional Requirements — Security](../SRS/06-Non-Functional-Requirements.md#2-security-requirements)
