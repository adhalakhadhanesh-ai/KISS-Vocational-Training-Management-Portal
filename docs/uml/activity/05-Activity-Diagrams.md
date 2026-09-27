# UML — Activity Diagrams

## Overview

Mermaid `flowchart` renderings of the key multi-step workflows described in [Enrollment Design](../SDD/04-Enrollment-Design.md), [Batch Allocation](../SDD/05-Batch-Allocation.md), and [Teacher Workspace — Gradebook](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module).

## 1. Enrollment Activity

```mermaid
flowchart TD
    Start([Student opens Enroll page]) --> CheckWindow{Enrollment window active for roll range?}
    CheckWindow -- No --> ErrWindow[Show ENR_001 / ENR_002]
    CheckWindow -- Yes --> CheckDup{Already enrolled this offering?}
    CheckDup -- Yes --> ErrDup[Show ENR_003]
    CheckDup -- No --> CheckCap{Capacity available?}
    CheckCap -- No --> ErrCap[Show ENR_004]
    CheckCap -- Yes --> CreateEnr[Create enrollment row]
    CreateEnr --> IncCount[Increment current_enrollment]
    IncCount --> Notify[Send confirmation email]
    Notify --> End([Enrollment complete])
    ErrWindow --> EndErr([No enrollment created])
    ErrDup --> EndErr
    ErrCap --> EndErr
```

## 2. Batch Allocation Activity

```mermaid
flowchart TD
    Start([Admin closes enrollment window]) --> Trigger[Admin triggers allocation run]
    Trigger --> Fetch[Fetch all ACTIVE enrollments + batch capacities]
    Fetch --> CheckCap{Total enrollments <= total batch capacity?}
    CheckCap -- No --> ErrCap[Return ALLOC_INSUFFICIENT_CAPACITY,\nno allocation applied]
    CheckCap -- Yes --> CheckEmpty{At least one enrollment?}
    CheckEmpty -- No --> ErrEmpty[Return ALLOC_NO_ENROLLMENTS]
    CheckEmpty -- Yes --> Shuffle[Randomly shuffle student list]
    Shuffle --> Assign[Assign students to batches\nuntil each batch reaches capacity]
    Assign --> Persist[Bulk-insert allocation rows]
    Persist --> Report[Generate allocation report]
    Report --> NotifyAll[Notify all students of batch assignment]
    NotifyAll --> End([Allocation complete])
    ErrCap --> EndErr([Allocation not run])
    ErrEmpty --> EndErr
```

## 3. Gradebook Draft-to-Publish Activity

```mermaid
flowchart TD
    Start([Teacher opens Gradebook]) --> CheckState{Gradebook state?}
    CheckState -- PUBLISHED --> ErrLocked[Reject edits: GRADEBOOK_LOCKED]
    CheckState -- DRAFT --> EnterMarks[Enter/update marks per component]
    EnterMarks --> ValidateMax{Marks <= component max_marks?}
    ValidateMax -- No --> ErrMax[Reject: GRADE_EXCEEDS_MAX]
    ValidateMax -- Yes --> Save[Save grade_entries]
    Save --> MoreEdits{More edits needed?}
    MoreEdits -- Yes --> EnterMarks
    MoreEdits -- No --> ClickPublish[Teacher clicks Publish]
    ClickPublish --> CheckAny{At least one mark entered?}
    CheckAny -- No --> ErrEmpty[Reject: GRADEBOOK_EMPTY]
    CheckAny -- Yes --> Publish[Set state = PUBLISHED, published_date = now]
    Publish --> NotifyStudents[Notify all students in batch]
    NotifyStudents --> End([Gradebook published, immutable])
    ErrLocked --> EndErr([No change made])
    ErrMax --> EnterMarks
    ErrEmpty --> ClickPublish
```

## Related Documents

- [Enrollment Design](../SDD/04-Enrollment-Design.md)
- [Batch Allocation](../SDD/05-Batch-Allocation.md)
- [Teacher Workspace](../SDD/06-Teacher-Workspace.md)
- [Sequence Diagrams](04-Sequence-Diagrams.md)
- [State Diagram](08-State-Diagram.md)
