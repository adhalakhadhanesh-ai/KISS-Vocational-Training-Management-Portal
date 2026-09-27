# UML — Use Case Diagram

## Overview

Mermaid rendering of the use cases documented in [Use Cases](../SRS/07-Use-Cases.md). Mermaid has no native use-case diagram type, so actors are modeled as nodes and use cases as stadium-shaped nodes inside actor-scoped subgraphs, matching the UC-ID naming from the SRS.

```mermaid
flowchart LR
    Student((Student))
    Teacher((Teacher))
    Admin((Admin))

    subgraph SC[Student Use Cases]
        UC1(["UC-STU-001: Log In to System"])
        UC2(["UC-STU-002: Enroll in Course"])
        UC3(["UC-STU-003: View Progress"])
        UC4(["UC-STU-004: Submit Grievance"])
        UC5(["UC-STU-005: Submit Course Feedback"])
        UC6(["UC-STU-006: Download Certificate"])
    end

    subgraph TC[Teacher Use Cases]
        UC7(["UC-TEA-001: View Assigned Courses"])
        UC8(["UC-TEA-002: Take Attendance"])
        UC9(["UC-TEA-003: Enter Grades"])
        UC10(["UC-TEA-004: Publish Grades"])
        UC11(["UC-TEA-005: Mark Course Complete"])
    end

    subgraph AC[Admin Use Cases]
        UC12(["UC-ADM-001: Create Semester"])
        UC13(["UC-ADM-002: Create Course Offering"])
        UC14(["UC-ADM-003: Define Grading Scheme"])
        UC15(["UC-ADM-004: Open Enrollment Window"])
        UC16(["UC-ADM-005: Run Batch Allocation"])
        UC17(["UC-ADM-006: Approve Batch Switch Request"])
        UC18(["UC-ADM-007: Generate Reports"])
    end

    Student --> UC1
    Student --> UC2
    Student --> UC3
    Student --> UC4
    Student --> UC5
    Student --> UC6

    Teacher --> UC1
    Teacher --> UC7
    Teacher --> UC8
    Teacher --> UC9
    Teacher --> UC10
    Teacher --> UC11

    Admin --> UC1
    Admin --> UC12
    Admin --> UC13
    Admin --> UC14
    Admin --> UC15
    Admin --> UC16
    Admin --> UC17
    Admin --> UC18

    UC2 -. includes .-> UC1
    UC9 -. precedes .-> UC10
    UC16 -. requires closed window from .-> UC15
```

## Notes

- **Log In (UC-STU-001)** is shared by all three roles — a single authentication flow gated by role stored on the JWT (see [RBAC Design](../SDD/03-RBAC-Design.md)).
- **UC-TEA-003 → UC-TEA-004** (`precedes`): grades must be entered in Draft state before they can be published — see [Teacher Workspace §6](../SDD/06-Teacher-Workspace.md#6-gradebook-sub-module).
- **UC-ADM-005 → UC-ADM-004** (`requires closed window from`): batch allocation can only run after its enrollment window is deactivated — see [Batch Allocation](../SDD/05-Batch-Allocation.md).
- Full pre/post-conditions and flows for every use case are in [Use Cases](../SRS/07-Use-Cases.md); this diagram is a visual index, not a replacement for that document.

## Related Documents

- [Use Cases (SRS)](../SRS/07-Use-Cases.md)
- [RBAC Design](../SDD/03-RBAC-Design.md)
