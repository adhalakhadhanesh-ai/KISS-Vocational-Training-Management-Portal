# UML — Sequence Diagrams

## Overview

Key end-to-end sequences, reflecting the flows already described in prose in [Authentication API](../API/Authentication.md), [Enrollment Design](../SDD/04-Enrollment-Design.md), [Batch Allocation](../SDD/05-Batch-Allocation.md), [Teacher Workspace](../SDD/06-Teacher-Workspace.md), and [Certificate Workflow](../SDD/10-Certificate-Workflow.md).

## 1. Login

```mermaid
sequenceDiagram
    actor U as User
    participant FE as React SPA
    participant API as Auth Controller
    participant SVC as Auth Service
    participant DB as PostgreSQL

    U->>FE: Enter email + password
    FE->>API: POST /auth/login
    API->>SVC: authenticate(email, password)
    SVC->>DB: SELECT user WHERE email = ?
    DB-->>SVC: user row (hashed password)
    SVC->>SVC: bcrypt.compare(password, hash)
    alt credentials valid
        SVC->>SVC: generate JWT (userId, role, exp=24h)
        SVC-->>API: token + user profile
        API-->>FE: 200 OK { token, user }
        FE-->>U: Redirect to role dashboard
    else credentials invalid
        SVC-->>API: INVALID_CREDENTIALS
        API-->>FE: 400 Error
        FE-->>U: Show error message
    end
```

## 2. Student Enrollment

```mermaid
sequenceDiagram
    actor S as Student
    participant FE as React SPA
    participant API as Student Controller
    participant SVC as Enrollment Service
    participant DB as PostgreSQL

    S->>FE: Select course, click Enroll
    FE->>API: POST /student/enroll
    API->>SVC: submitEnrollment(studentId, courseOfferingId)
    SVC->>DB: Check enrollment_windows.is_active + roll range
    DB-->>SVC: window valid
    SVC->>DB: Check course_offerings.current_enrollment < max_capacity
    DB-->>SVC: capacity available
    SVC->>DB: Check existing enrollment (unique student+offering)
    DB-->>SVC: none found
    SVC->>DB: INSERT INTO enrollments
    SVC->>DB: UPDATE course_offerings SET current_enrollment += 1
    DB-->>SVC: committed
    SVC-->>API: enrollment confirmation
    API-->>FE: 201 Created
    FE-->>S: "Enrollment confirmed" + email queued
```

## 3. Batch Allocation Run

```mermaid
sequenceDiagram
    actor A as Admin
    participant FE as React SPA
    participant API as Allocation Controller
    participant SVC as Allocation Service
    participant DB as PostgreSQL

    A->>FE: Click "Run Allocation" for course offering
    FE->>API: POST /admin/allocation/run
    API->>SVC: runAllocation(courseOfferingId)
    SVC->>DB: Verify enrollment_windows.is_active = false
    DB-->>SVC: confirmed closed
    SVC->>DB: SELECT all ACTIVE enrollments for offering
    DB-->>SVC: enrollment list
    SVC->>DB: SELECT batches + capacities for offering
    DB-->>SVC: batch list
    alt total enrollments <= total batch capacity
        SVC->>SVC: Random shuffle, assign students to batches
        SVC->>DB: BULK INSERT allocations
        DB-->>SVC: committed
        SVC-->>API: allocation report
        API-->>FE: 200 OK { batches, studentCounts }
        FE-->>A: Show allocation report
    else insufficient capacity
        SVC-->>API: ALLOC_INSUFFICIENT_CAPACITY
        API-->>FE: 400 Error
        FE-->>A: Show capacity error, no partial allocation applied
    end
```

## 4. Grade Entry & Publish

```mermaid
sequenceDiagram
    actor T as Teacher
    participant FE as React SPA
    participant API as Teacher Controller
    participant SVC as Gradebook Service
    participant DB as PostgreSQL
    actor St as Student

    T->>FE: Enter marks per component
    FE->>API: POST /teacher/grades
    API->>SVC: saveGrades(batchId, entries)
    SVC->>DB: Check gradebooks.state = 'DRAFT'
    DB-->>SVC: DRAFT confirmed
    SVC->>DB: UPSERT grade_entries (validate against max_marks)
    DB-->>SVC: committed
    SVC-->>API: saved
    API-->>FE: 200 OK

    T->>FE: Click "Publish"
    FE->>API: PATCH /teacher/gradebook/{batchId}/publish
    API->>SVC: publish(batchId)
    SVC->>DB: Verify at least one mark entered
    DB-->>SVC: confirmed
    SVC->>DB: UPDATE gradebooks SET state='PUBLISHED', published_date=NOW()
    DB-->>SVC: committed
    SVC-->>API: published
    API-->>FE: 200 OK
    SVC-->>St: Notification: grades published (async)
    St->>API: GET /student/grades/{courseId}
    API->>DB: SELECT grade_entries WHERE gradebooks.state='PUBLISHED'
    DB-->>API: published grades
    API-->>St: 200 OK { grades }
```

## 5. Course Completion → Certificate Generation

```mermaid
sequenceDiagram
    actor T as Teacher
    participant API as Teacher Controller
    participant CSVC as Completion Service
    participant CERT as Certificate Service
    participant DB as PostgreSQL
    actor St as Student

    T->>API: POST /teacher/course/{batchId}/complete
    API->>CSVC: markComplete(batchId)
    CSVC->>DB: Verify course_offerings.end_date has passed
    DB-->>CSVC: confirmed
    CSVC->>DB: Verify gradebooks.state = 'PUBLISHED'
    DB-->>CSVC: confirmed
    CSVC->>DB: UPDATE allocations SET status='COMPLETED' WHERE batch_id=?
    DB-->>CSVC: committed
    CSVC->>CERT: generateCertificates(batchId)  [async]
    CSVC-->>API: 200 OK { certificatesQueued }
    API-->>T: "Course marked complete"

    loop for each completed allocation
        CERT->>DB: Generate unique cert_number
        CERT->>CERT: Render PDF (student, course, dates)
        CERT->>DB: INSERT INTO certificates
        DB-->>CERT: committed
        CERT-->>St: Email notification (async)
    end
```

## Related Documents

- [Authentication API](../API/Authentication.md)
- [Enrollment Design](../SDD/04-Enrollment-Design.md)
- [Batch Allocation](../SDD/05-Batch-Allocation.md)
- [Teacher Workspace](../SDD/06-Teacher-Workspace.md)
- [Certificate Workflow](../SDD/10-Certificate-Workflow.md)
- [Activity Diagrams](05-Activity-Diagrams.md)
