# UML — Entity Relationship Diagram

## Overview

Mermaid `erDiagram` rendering of the physical schema in [Database Design](../SDD/02-Database-Design.md). This supersedes the ASCII-art diagram in that document as the authoritative visual reference; the ASCII diagram there has been simplified to a summary sketch that points here. No entity, attribute, or relationship below was added beyond what Database Design already specifies — per the required fix, there is no `Mentor` entity; `mentor_name` and `mentor_phone` are attributes on `STUDENTS`.

```mermaid
erDiagram
    USERS ||--o| STUDENTS : "user_id"
    USERS ||--o| TEACHERS : "user_id"
    USERS ||--o| ADMINS : "user_id"

    COURSES ||--o{ COURSE_OFFERINGS : "course_id"
    COURSE_OFFERINGS ||--o{ ENROLLMENT_WINDOWS : "course_offering_id"
    COURSE_OFFERINGS ||--o{ ENROLLMENTS : "course_offering_id"
    COURSE_OFFERINGS ||--o{ BATCHES : "course_offering_id"
    COURSE_OFFERINGS ||--|| GRADING_SCHEMES : "course_offering_id"
    COURSE_OFFERINGS ||--|| FEEDBACK_FORMS : "course_offering_id"
    COURSE_OFFERINGS ||--o{ CERTIFICATES : "course_offering_id"

    STUDENTS ||--o{ ENROLLMENTS : "student_id"
    STUDENTS ||--o{ ALLOCATIONS : "student_id"
    STUDENTS ||--o{ GRIEVANCES : "student_id"
    STUDENTS ||--o{ CERTIFICATES : "student_id"

    TEACHERS ||--o{ BATCHES : "teacher_id"
    TEACHERS ||--o{ CERTIFICATES : "teacher_id"

    BATCHES ||--o{ ALLOCATIONS : "batch_id"
    BATCHES ||--|| GRADEBOOKS : "batch_id"

    ALLOCATIONS ||--o{ ATTENDANCE : "allocation_id"
    ALLOCATIONS ||--o{ GRADE_ENTRIES : "allocation_id"
    ALLOCATIONS ||--o{ FEEDBACK_RESPONSES : "allocation_id"
    ALLOCATIONS ||--o| CERTIFICATES : "allocation_id"

    GRADING_SCHEMES ||--o{ GRADE_COMPONENTS : "scheme_id"
    GRADE_COMPONENTS ||--o{ GRADE_ENTRIES : "component_id"

    FEEDBACK_FORMS ||--o{ FEEDBACK_QUESTIONS : "form_id"
    FEEDBACK_QUESTIONS ||--o{ FEEDBACK_RESPONSES : "question_id"

    USERS {
        int user_id PK
        string email UK
        string password
        string role
    }
    STUDENTS {
        int student_id PK
        int user_id FK
        string roll_number UK
        string name
        string mentor_name
        string mentor_phone
    }
    TEACHERS {
        int teacher_id PK
        int user_id FK
        string name
        string department
    }
    ADMINS {
        int admin_id PK
        int user_id FK
        string name
    }
    COURSES {
        int course_id PK
        string course_code UK
        string course_name
        int duration_weeks
    }
    COURSE_OFFERINGS {
        int course_offering_id PK
        int course_id FK
        string semester
        int max_capacity
        int current_enrollment
    }
    ENROLLMENT_WINDOWS {
        int window_id PK
        int course_offering_id FK
        date window_start
        date window_end
        bool is_active
    }
    ENROLLMENTS {
        int enrollment_id PK
        int student_id FK
        int course_offering_id FK
        string status
    }
    BATCHES {
        int batch_id PK
        int course_offering_id FK
        int teacher_id FK
        string batch_name
        int batch_capacity
    }
    ALLOCATIONS {
        int allocation_id PK
        int student_id FK
        int batch_id FK
        string status
        int switch_count
    }
    ATTENDANCE {
        int attendance_id PK
        int allocation_id FK
        date attendance_date
        string status
    }
    GRADING_SCHEMES {
        int scheme_id PK
        int course_offering_id FK
        string scheme_name
    }
    GRADE_COMPONENTS {
        int component_id PK
        int scheme_id FK
        string component_name
        int max_marks
        int weight_percentage
    }
    GRADE_ENTRIES {
        int entry_id PK
        int allocation_id FK
        int component_id FK
        int marks_obtained
    }
    GRADEBOOKS {
        int gradebook_id PK
        int batch_id FK
        string state
        datetime published_date
    }
    CERTIFICATES {
        int certificate_id PK
        string cert_number UK
        int student_id FK
        int course_offering_id FK
        int allocation_id FK
        int teacher_id FK
        date issue_date
    }
    FEEDBACK_FORMS {
        int form_id PK
        int course_offering_id FK
        string form_name
    }
    FEEDBACK_QUESTIONS {
        int question_id PK
        int form_id FK
        string question_text
        string question_type
    }
    FEEDBACK_RESPONSES {
        int response_id PK
        int allocation_id FK
        int question_id FK
        int rating
    }
    GRIEVANCES {
        int grievance_id PK
        int student_id FK
        string category
        string severity
        string status
    }
```

## Notes

- `USERS ||--o| STUDENTS` etc. use a zero-or-one notation because a `Users` row can belong to exactly one of Students/Teachers/Admins depending on `role`, never more than one.
- `COURSE_OFFERINGS ||--o| CERTIFICATES` type relationships that fan out from a 1-side entity to `CERTIFICATES` are shown from the three FKs `CERTIFICATES` actually holds (`student_id`, `course_offering_id`, `allocation_id`, `teacher_id`) — all four are real foreign keys on the table per [Database Design §4.16](../SDD/02-Database-Design.md#416-certificates-table).
- See [Class Diagram §Notes](02-Class-Diagram.md#notes) for the same caveat on the undocumented `SwitchRequest` concept — it is not included here for the same reason.

## Related Documents

- [Database Design](../SDD/02-Database-Design.md) — authoritative table definitions, indexes, and constraints
- [Class Diagram](02-Class-Diagram.md)
