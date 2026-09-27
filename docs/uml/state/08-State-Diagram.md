# UML — State Diagrams

## Overview

State machines for every stateful entity documented in [Database Design](../SDD/02-Database-Design.md) (fields with a bounded set of string values) and the module design docs that describe their transition rules.

## 1. Gradebook State

Source: `gradebooks.state` — [Database Design §4.15](../SDD/02-Database-Design.md#415-gradebook-table), [Teacher Workspace §6](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module).

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Batch created
    DRAFT --> DRAFT: Save marks (POST /teacher/grades)
    DRAFT --> PUBLISHED: Publish (≥1 mark entered)
    PUBLISHED --> [*]: Terminal — immutable, visible to students
```

## 2. Enrollment Status

Source: `enrollments.status` — [Database Design §4.8](../SDD/02-Database-Design.md#48-enrollments-table).

```mermaid
stateDiagram-v2
    [*] --> ACTIVE: Student enrolls (POST /student/enroll)
    ACTIVE --> DROPPED: Student drops (DELETE, before allocation)
    ACTIVE --> COMPLETED: Course marked complete by teacher
    DROPPED --> [*]
    COMPLETED --> [*]
```

## 3. Allocation Status

Source: `allocations.status` — [Database Design §4.10](../SDD/02-Database-Design.md#410-allocations-table), [Batch Allocation](../SDD/05-Batch-Allocation.md).

```mermaid
stateDiagram-v2
    [*] --> ACTIVE: Allocation run assigns student to batch
    ACTIVE --> SWITCHED: Admin approves batch switch request
    ACTIVE --> DROPPED: Admin removes student from batch
    SWITCHED --> SWITCHED: Further switch requests blocked\n(SWITCH_ALREADY_REQUESTED)
    DROPPED --> [*]
```

## 4. Grievance Status

Source: `grievances.status` — [Database Design §4.20](../SDD/02-Database-Design.md#420-grievances-table).

```mermaid
stateDiagram-v2
    [*] --> SUBMITTED: Student submits grievance
    SUBMITTED --> UNDER_REVIEW: Admin begins review
    UNDER_REVIEW --> RESOLVED: Admin resolves, adds resolution_notes
    RESOLVED --> CLOSED: Admin closes
    UNDER_REVIEW --> CLOSED: Closed without resolution (e.g. duplicate)
    CLOSED --> [*]
```

## 5. Enrollment Window Activation

Source: `enrollment_windows.is_active` — [Database Design §4.7](../SDD/02-Database-Design.md#47-enrollmentwindows-table), [Admin Dashboard §5](../SDD/08-Admin-Dashboard.md#5-course-offering-seat--batch-configuration).

```mermaid
stateDiagram-v2
    [*] --> INACTIVE: Window created (is_active = false)
    INACTIVE --> ACTIVE: Admin activates (PATCH .../activate)
    ACTIVE --> INACTIVE: Admin deactivates (PATCH .../deactivate)\nor window_end passes
    INACTIVE --> [*]: Allocation may now be run
```

## Related Documents

- [Database Design](../SDD/02-Database-Design.md)
- [Teacher Workspace](../SDD/06-Teacher-Workspace.md)
- [Batch Allocation](../SDD/05-Batch-Allocation.md)
- [Activity Diagrams](05-Activity-Diagrams.md)
