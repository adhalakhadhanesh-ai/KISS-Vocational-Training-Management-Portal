# UML — Data Flow Diagram, Level 1

## Overview

Expands the single process in [DFD Level 0](09-DFD-Level0.md) into its major sub-processes and data stores, matching the module boundaries in [SDD Index](../SDD/SDD-Index.md). Notation: circles = processes, rounded rectangles = external entities, open rectangles = data stores (tables from [Database Design](../SDD/02-Database-Design.md)).

```mermaid
flowchart TD
    Student([Student])
    Teacher([Teacher])
    Admin([Admin])
    EmailSvc([Email Notification Service])

    P1((1.0\nAuthenticate\nUser))
    P2((2.0\nManage\nEnrollment))
    P3((3.0\nRun Batch\nAllocation))
    P4((4.0\nManage\nAttendance))
    P5((5.0\nManage\nGradebook))
    P6((6.0\nGenerate\nCertificate))
    P7((7.0\nManage\nGrievance & Feedback))
    P8((8.0\nAdmin\nConfiguration))
    P9((9.0\nGenerate\nReports))

    DS1[(D1: users / students / teachers / admins)]
    DS2[(D2: courses / course_offerings / enrollment_windows)]
    DS3[(D3: enrollments)]
    DS4[(D4: batches / allocations)]
    DS5[(D5: attendance)]
    DS6[(D6: grading_schemes / grade_components / grade_entries / gradebooks)]
    DS7[(D7: certificates)]
    DS8[(D8: feedback_forms / feedback_questions / feedback_responses)]
    DS9[(D9: grievances)]

    Student -- credentials --> P1
    Teacher -- credentials --> P1
    Admin -- credentials --> P1
    P1 <--> DS1
    P1 -- JWT --> Student
    P1 -- JWT --> Teacher
    P1 -- JWT --> Admin

    Student -- course selection --> P2
    P2 <--> DS2
    P2 <--> DS3
    P2 -- confirmation --> Student
    P2 -- enrollment event --> EmailSvc

    Admin -- trigger --> P3
    P3 <--> DS3
    P3 <--> DS4
    P3 -- allocation report --> Admin
    P3 -- allocation event --> EmailSvc

    Teacher -- attendance records --> P4
    P4 <--> DS5
    P4 -- attendance view --> Student

    Teacher -- marks, publish action --> P5
    P5 <--> DS6
    P5 -- published grades --> Student
    P5 -- publish event --> EmailSvc

    Teacher -- completion trigger --> P6
    P6 <--> DS4
    P6 <--> DS7
    P6 -- certificate --> Student
    P6 -- certificate event --> EmailSvc

    Student -- grievance/feedback --> P7
    P7 <--> DS8
    P7 <--> DS9
    Admin -- resolution --> P7
    P7 -- status update --> Student

    Admin -- semester/offering/scheme/window config --> P8
    P8 <--> DS2
    P8 <--> DS6

    Admin -- report request --> P9
    P9 <--> DS3
    P9 <--> DS4
    P9 <--> DS7
    P9 -- report file --> Admin
```

## Notes

- Data stores (D1–D9) group the 20 tables from [Database Design §4](../SDD/02-Database-Design.md#4-table-definitions) by the process that primarily owns them; several tables are read by more than one process (e.g. `allocations` is written by 3.0 and read by 4.0/5.0/6.0), shown via the process-to-store arrows relevant to each.
- Process numbering (1.0–9.0) corresponds one-to-one with the primary module documents: 2.0 → [Enrollment Design](../SDD/04-Enrollment-Design.md), 3.0 → [Batch Allocation](../SDD/05-Batch-Allocation.md), 4.0/5.0/6.0 → [Teacher Workspace](../SDD/06-Teacher-Workspace.md), 8.0 → [Admin Dashboard](../SDD/08-Admin-Dashboard.md).

## Related Documents

- [DFD Level 0](09-DFD-Level0.md)
- [Database Design](../SDD/02-Database-Design.md)
- [SDD Index](../SDD/SDD-Index.md)
