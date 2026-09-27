# ADR-002: Random Batch Allocation Algorithm

**Date**: 2026-01-01  
**Status**: Accepted  
**Author**: Architecture Team  

## Context

After students enroll in courses during designated enrollment windows, they must be assigned to specific batches. The allocation must be fair, transparent, and scalable.

## Problem

How should students be allocated to batches after enrollment closes?

### Options Considered

**Option 1: Random Allocation**
- Shuffle all enrollments randomly
- Assign to batches sequentially
- Pros: Fair, transparent, fast, reproducible
- Cons: May not match preferences

**Option 2: Preference-Based Allocation**
- Students rank batches by preference
- Algorithm matches preferences to capacity
- Pros: Student satisfaction, preferences honored
- Cons: Complex, slower, potentially unfair, hard to explain

**Option 3: Merit-Based Allocation**
- Allocate based on academic performance
- Top performers get preferred batches
- Pros: Rewards performance
- Cons: Unfair, creates perception of bias, complex

**Option 4: First-Come-First-Served**
- Early enrollees get best batches
- Pros: Simple, transparent
- Cons: Unfair to later enrollees, doesn't scale

## Decision

**We choose Option 1: Random Allocation**

Random shuffling followed by sequential batch assignment.

## Rationale

### Fairness
- Every enrolled student has equal probability of assignment
- No favoritism based on preferences or performance
- Truly democratic approach

### Transparency
- Algorithm is simple and explainable to students
- Results are verifiable
- Reproducible with stored random seed

### Performance
- O(n) time complexity
- Can allocate 1000 students in < 5 seconds
- Scales well with enrollment growth

### Scalability
- No complex constraint solving required
- No iterative matching needed
- Works for any batch configuration

### Simplicity
- Easy to understand by stakeholders
- Low implementation complexity
- Fewer edge cases

### Equity
- No special treatment based on roll number, enrollment time, etc.
- Batch assignments truly independent of other factors
- Reduces gaming of the system

## Consequences

### Positive
✓ Fast allocation process
✓ Fair to all students
✓ Transparent and understandable
✓ Easy to explain to students
✓ Reproducible results
✓ Scalable

### Negative
✗ Students may not get preferred batches
✗ Schedule conflicts possible (student responsibility)
✗ One-time switch request allows limited flexibility

## Mitigations

### Student Concerns
- Batch switching allowed (one per course)
- Switch requests reviewed by admin
- Capacity-based switching (no overcrowding)

### Transparency
- Random seed logged and published
- Allocation report shows every student's assignment
- Students can verify their own allocation

### Fairness
- Public communication of algorithm before enrollment
- No secret criteria or bias
- Equal treatment for all

## Alternatives Reconsidered

### Preference-Based (Rejected)
Why not used:
- Students rarely agree on preferences
- Preference collection is cumbersome
- Complex matching algorithms hard to explain
- Students who don't specify preferences feel disadvantaged
- Preference satisfaction never 100%

### Merit-Based (Rejected)
Why not used:
- Violates principle of fairness
- Students feel evaluated on allocation
- Difficult to apply fair metrics
- Creates tension and resentment

## Validation

### Stakeholder Feedback
- Students: Prefer random to unknown criteria
- Teachers: Indifferent, any balanced approach works
- Admin: Prefer simple, fast process
- Leadership: Appreciates fairness and simplicity

### Testing Strategy
- Algorithm tested with 1000+ students
- Randomness verified with statistical tests
- Performance validated under load
- Reproducibility confirmed with seeds

## Implementation Details

### Algorithm
```
1. Fetch all enrollments for course
2. Verify sufficient capacity
3. Shuffle enrollments with random seed
4. Assign sequentially to batches (in capacity order)
5. Verify all assigned (catch overage)
6. Save allocations
7. Generate report and notify students
```

### Code Location
- Service: `AllocationService.runAllocation()`
- File: `src/main/java/com/kiss/service/AllocationService.java`

### Testing
- Unit tests: `AllocationServiceTest.java`
- Integration tests: Allocation workflow tests
- Performance tests: 1000+ student allocation

## Related Decisions

- [ADR-001: CourseOffering Entity](ADR-001-CourseOffering.md)
- [ADR-003: RBAC Model](ADR-003-RBAC.md)

## References

- [Batch Allocation Design](../SDD/05-Batch-Allocation.md)
- [Functional Requirements - Allocation](../SRS/05-Functional-Requirements.md#fr-adm-010-run-allocation)
- [System Architecture - Batch Allocation](../SDD/01-System-Architecture.md#batch-processing)
