# UML — Class Diagram

## Overview

Mermaid class diagram reflecting the 20 tables defined in [Database Design](../SDD/02-Database-Design.md). Attributes are abbreviated to the fields relevant for understanding relationships; see Database Design for full column lists, types, and constraints. This diagram does not introduce any table, column, or relationship not already present in Database Design.

```mermaid
classDiagram
    class Users {
        +int user_id
        +string email
        +string password
        +string role
    }
    class Students {
        +int student_id
        +int user_id
        +string roll_number
        +string name
        +string section
        +string mentor_name
        +string mentor_phone
    }
    class Teachers {
        +int teacher_id
        +int user_id
        +string name
        +string department
    }
    class Admins {
        +int admin_id
        +int user_id
        +string name
    }
    class Courses {
        +int course_id
        +string course_code
        +string course_name
        +int duration_weeks
    }
    class CourseOfferings {
        +int course_offering_id
        +int course_id
        +string semester
        +int max_capacity
        +int current_enrollment
    }
    class EnrollmentWindows {
        +int window_id
        +int course_offering_id
        +date window_start
        +date window_end
        +string roll_number_start
        +string roll_number_end
        +bool is_active
    }
    class Enrollments {
        +int enrollment_id
        +int student_id
        +int course_offering_id
        +string status
    }
    class Batches {
        +int batch_id
        +int course_offering_id
        +string batch_name
        +int batch_capacity
        +int teacher_id
    }
    class Allocations {
        +int allocation_id
        +int student_id
        +int batch_id
        +string status
        +int switch_count
    }
    class Attendance {
        +int attendance_id
        +int allocation_id
        +date attendance_date
        +string status
    }
    class GradingSchemes {
        +int scheme_id
        +int course_offering_id
        +string scheme_name
    }
    class GradeComponents {
        +int component_id
        +int scheme_id
        +string component_name
        +int max_marks
        +int weight_percentage
    }
    class GradeEntries {
        +int entry_id
        +int allocation_id
        +int component_id
        +int marks_obtained
    }
    class Gradebooks {
        +int gradebook_id
        +int batch_id
        +string state
        +datetime published_date
    }
    class Certificates {
        +int certificate_id
        +string cert_number
        +int student_id
        +int course_offering_id
        +int allocation_id
        +int teacher_id
        +date issue_date
        +string pdf_url
    }
    class FeedbackForms {
        +int form_id
        +int course_offering_id
        +string form_name
    }
    class FeedbackQuestions {
        +int question_id
        +int form_id
        +string question_text
        +string question_type
    }
    class FeedbackResponses {
        +int response_id
        +int allocation_id
        +int question_id
        +int rating
    }
    class Grievances {
        +int grievance_id
        +int student_id
        +string category
        +string severity
        +string status
    }

    Users "1" -- "0..1" Students : user_id
    Users "1" -- "0..1" Teachers : user_id
    Users "1" -- "0..1" Admins : user_id

    Courses "1" -- "many" CourseOfferings : course_id
    CourseOfferings "1" -- "many" EnrollmentWindows : course_offering_id
    CourseOfferings "1" -- "many" Enrollments : course_offering_id
    CourseOfferings "1" -- "many" Batches : course_offering_id
    CourseOfferings "1" -- "1" GradingSchemes : course_offering_id
    CourseOfferings "1" -- "1" FeedbackForms : course_offering_id

    Students "1" -- "many" Enrollments : student_id
    Students "1" -- "many" Allocations : student_id
    Students "1" -- "many" Grievances : student_id
    Students "1" -- "many" Certificates : student_id

    Teachers "1" -- "many" Batches : teacher_id
    Teachers "1" -- "many" Certificates : teacher_id

    Batches "1" -- "many" Allocations : batch_id
    Batches "1" -- "1" Gradebooks : batch_id

    Allocations "1" -- "many" Attendance : allocation_id
    Allocations "1" -- "many" GradeEntries : allocation_id
    Allocations "1" -- "many" FeedbackResponses : allocation_id
    Allocations "1" -- "0..1" Certificates : allocation_id

    GradingSchemes "1" -- "many" GradeComponents : scheme_id
    GradeComponents "1" -- "many" GradeEntries : component_id

    FeedbackForms "1" -- "many" FeedbackQuestions : form_id
    FeedbackQuestions "1" -- "many" FeedbackResponses : question_id
```

## Notes

- `Students`, `Teachers`, and `Admins` are **not** subclasses of `Users` in an OOP-inheritance sense — each holds its own `user_id` foreign key back to `Users` (a 1-to-0..1 association), exactly as defined in [Database Design §4.1–4.4](../SDD/02-Database-Design.md#4-table-definitions). The diagram intentionally uses association, not inheritance arrows, to stay accurate to the actual schema.
- `mentor_name` / `mentor_phone` on `Students` are plain attributes with no relationship line — there is no `Mentor` class, consistent with the fix in [Database Design §4.2](../SDD/02-Database-Design.md#42-students-table).
- **Known documentation gap** (not fixed here, as it would require designing new schema rather than documenting existing schema): a `SwitchRequest` concept is referenced by the batch-switching workflow in [Batch Allocation §6](../SDD/05-Batch-Allocation.md#6-batch-switching) and by `POST /student/allocation/{allocationId}/switch` / `GET /admin/switch-requests` in the API docs, but no `switch_requests` table is defined in [Database Design](../SDD/02-Database-Design.md). It is intentionally omitted from this class diagram rather than invented. Flagging this for the engineering team to resolve in Database Design is recommended.

## Related Documents

- [Database Design](../SDD/02-Database-Design.md)
- [ER Diagram](03-ER-Diagram.md)
