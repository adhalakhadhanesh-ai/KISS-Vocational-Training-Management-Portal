# Database Design

## 1. Overview

This document defines the complete database schema for the KISS Vocational Training Management Portal, including tables, attributes, relationships, and constraints.

## 2. Database Technology

- **DBMS**: PostgreSQL 12+
- **Provider**: Supabase (Managed PostgreSQL)
- **Encoding**: UTF-8
- **Timezone**: IST (UTC+5:30)

## 3. Entity Relationship Diagram

```
┌─────────────┐
│ Users       │
├─────────────┤
│ PK: user_id │─┐
│ email       │ │
│ password    │ │
│ role        │ │
│ created_at  │ │
│ updated_at  │ │
└─────────────┘ │
                │
    ┌───────────┼───────────┐
    │           │           │
┌───▼───┐  ┌────▼────┐  ┌──▼───┐
│Student│  │ Teacher │  │Admin │
└───┬───┘  └────┬────┘  └──────┘
    │           │
    └─────┬─────┴──────────────┬─────────────┐
          │                    │             │
      ┌───▼────┐         ┌────▼─────┐  ┌───▼──┐   ┌─────────┐
      │Enroll  │         │Allocation│  │Attend│   │Feedback │
      │ment    │         │          │  │ance  │   │Response │
      └────────┘         └────┬─────┘  └──────┘   └─────────┘
          │                   │
          └─────┬─────────────┘
                │
          ┌─────▼──────┐
          │ CourseOff  │
          │ering       │
          └─────┬──────┘
                │
          ┌─────▼──────┐
          │ Batch      │
          └─────┬──────┘
                │
          ┌─────▼──────┐
          │ Course     │
          └────────────┘
```

> **Note**: `mentor_name` and `mentor_phone` are plain attributes on the `students` table (a student's mentor is recorded as free text, not a linked user account), so no separate `Mentor` entity appears in this diagram.

## 4. Table Definitions

### 4.1 Users Table

**Purpose**: Core user authentication and authorization

```sql
CREATE TABLE users (
  user_id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL, -- bcrypt hashed
  role VARCHAR(50) NOT NULL, -- 'STUDENT', 'TEACHER', 'ADMIN'
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP NULL
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

**Columns**:
- `user_id`: Unique identifier (primary key)
- `email`: Login email (unique, required)
- `password`: Hashed password (bcrypt)
- `role`: User type (STUDENT, TEACHER, ADMIN)
- `is_active`: Account active status
- `created_at`: Account creation timestamp
- `updated_at`: Last modification timestamp
- `last_login`: Last login timestamp

**Constraints**:
- Email must be unique
- Role must be one of three predefined values
- Password must be hashed before storage

---

### 4.2 Students Table

**Purpose**: Student profile information

```sql
CREATE TABLE students (
  student_id SERIAL PRIMARY KEY,
  user_id INT UNIQUE NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  roll_number VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  section VARCHAR(50) NOT NULL,
  email VARCHAR(255) NOT NULL,
  phone_number VARCHAR(20),
  address TEXT,
  mentor_name VARCHAR(255),
  mentor_phone VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_students_roll ON students(roll_number);
CREATE INDEX idx_students_user_id ON students(user_id);
```

**Columns**:
- `student_id`: Unique identifier
- `user_id`: Reference to users table (1:1)
- `roll_number`: Institutional roll number (unique)
- `name`: Student full name
- `section`: Academic section
- `email`: Contact email
- `phone_number`: Contact phone
- `address`: Residential address
- `mentor_name`: Mentor's name (plain text field, not a linked user account — see AS-ORG-003 in [Assumptions](../SRS/08-Assumptions.md))
- `mentor_phone`: Mentor phone number

**Relationships**:
- FK to users: One-to-One (1:1)
- No FK for mentor: `mentor_name`/`mentor_phone` are informational attributes only, not a relationship to another table

---

### 4.3 Teachers Table

**Purpose**: Teacher profile information

```sql
CREATE TABLE teachers (
  teacher_id SERIAL PRIMARY KEY,
  user_id INT UNIQUE NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  phone_number VARCHAR(20),
  department VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_teachers_user_id ON teachers(user_id);
```

**Columns**:
- `teacher_id`: Unique identifier
- `user_id`: Reference to users table (1:1)
- `name`: Teacher full name
- `email`: Contact email
- `phone_number`: Contact phone
- `department`: Department name

**Relationships**:
- FK to users: One-to-One (1:1)

---

### 4.4 Admins Table

**Purpose**: Admin user information

```sql
CREATE TABLE admins (
  admin_id SERIAL PRIMARY KEY,
  user_id INT UNIQUE NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  phone_number VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_admins_user_id ON admins(user_id);
```

---

### 4.5 Courses Table

**Purpose**: Base course definitions

```sql
CREATE TABLE courses (
  course_id SERIAL PRIMARY KEY,
  course_code VARCHAR(50) UNIQUE NOT NULL,
  course_name VARCHAR(255) NOT NULL,
  description TEXT,
  duration_weeks INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_courses_code ON courses(course_code);
```

**Columns**:
- `course_id`: Unique identifier
- `course_code`: Course code (e.g., "VOCA-001")
- `course_name`: Course full name
- `description`: Course description
- `duration_weeks`: Course duration in weeks

---

### 4.6 CourseOfferings Table

**Purpose**: Semester-specific course instances

```sql
CREATE TABLE course_offerings (
  course_offering_id SERIAL PRIMARY KEY,
  course_id INT NOT NULL REFERENCES courses(course_id),
  semester VARCHAR(50) NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  max_capacity INT NOT NULL,
  current_enrollment INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(course_id, semester)
);

CREATE INDEX idx_course_offerings_semester ON course_offerings(semester);
CREATE INDEX idx_course_offerings_course ON course_offerings(course_id);
```

**Columns**:
- `course_offering_id`: Unique identifier
- `course_id`: Reference to courses table
- `semester`: Semester identifier (e.g., "Spring 2026")
- `start_date`: Course start date
- `end_date`: Course end date
- `max_capacity`: Maximum students
- `current_enrollment`: Current enrollment count

**Constraints**:
- One course offering per course per semester (unique constraint)

---

### 4.7 EnrollmentWindows Table

**Purpose**: Manage enrollment period and roll-number ranges

```sql
CREATE TABLE enrollment_windows (
  window_id SERIAL PRIMARY KEY,
  course_offering_id INT NOT NULL REFERENCES course_offerings(course_offering_id),
  window_start DATE NOT NULL,
  window_end DATE NOT NULL,
  roll_number_start VARCHAR(50),
  roll_number_end VARCHAR(50),
  is_active BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_windows_offering ON enrollment_windows(course_offering_id);
CREATE INDEX idx_windows_dates ON enrollment_windows(window_start, window_end);
```

**Columns**:
- `window_id`: Unique identifier
- `course_offering_id`: Reference to course offerings
- `window_start`: Enrollment start date
- `window_end`: Enrollment end date
- `roll_number_start`: Starting roll number range
- `roll_number_end`: Ending roll number range
- `is_active`: Window activation status

---

### 4.8 Enrollments Table

**Purpose**: Student course enrollments

```sql
CREATE TABLE enrollments (
  enrollment_id SERIAL PRIMARY KEY,
  student_id INT NOT NULL REFERENCES students(student_id),
  course_offering_id INT NOT NULL REFERENCES course_offerings(course_offering_id),
  enrollment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status VARCHAR(50) DEFAULT 'ACTIVE', -- ACTIVE, DROPPED, COMPLETED
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(student_id, course_offering_id)
);

CREATE INDEX idx_enrollments_student ON enrollments(student_id);
CREATE INDEX idx_enrollments_offering ON enrollments(course_offering_id);
CREATE INDEX idx_enrollments_status ON enrollments(status);
```

**Columns**:
- `enrollment_id`: Unique identifier
- `student_id`: Reference to students
- `course_offering_id`: Reference to course offerings
- `enrollment_date`: Enrollment timestamp
- `status`: Enrollment status (ACTIVE, DROPPED, COMPLETED)

**Constraints**:
- One enrollment per student per course offering

---

### 4.9 Batches Table

**Purpose**: Course batches for classroom organization

```sql
CREATE TABLE batches (
  batch_id SERIAL PRIMARY KEY,
  course_offering_id INT NOT NULL REFERENCES course_offerings(course_offering_id),
  batch_name VARCHAR(100) NOT NULL,
  batch_capacity INT NOT NULL,
  teacher_id INT NOT NULL REFERENCES teachers(teacher_id),
  batch_timing VARCHAR(100), -- e.g., "10:00 AM - 12:00 PM"
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_batches_offering ON batches(course_offering_id);
CREATE INDEX idx_batches_teacher ON batches(teacher_id);
```

**Columns**:
- `batch_id`: Unique identifier
- `course_offering_id`: Reference to course offerings
- `batch_name`: Batch identifier (e.g., "Batch A")
- `batch_capacity`: Max students in batch
- `teacher_id`: Reference to assigned teacher
- `batch_timing`: Class timing

---

### 4.10 Allocations Table

**Purpose**: Student-to-batch assignments

```sql
CREATE TABLE allocations (
  allocation_id SERIAL PRIMARY KEY,
  student_id INT NOT NULL REFERENCES students(student_id),
  batch_id INT NOT NULL REFERENCES batches(batch_id),
  allocation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status VARCHAR(50) DEFAULT 'ACTIVE', -- ACTIVE, SWITCHED, DROPPED
  switch_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(student_id, batch_id)
);

CREATE INDEX idx_allocations_student ON allocations(student_id);
CREATE INDEX idx_allocations_batch ON allocations(batch_id);
```

**Columns**:
- `allocation_id`: Unique identifier
- `student_id`: Reference to students
- `batch_id`: Reference to batches
- `allocation_date`: Allocation timestamp
- `status`: Allocation status
- `switch_count`: Number of batch switches used

---

### 4.11 Attendance Table

**Purpose**: Student attendance tracking

```sql
CREATE TABLE attendance (
  attendance_id SERIAL PRIMARY KEY,
  allocation_id INT NOT NULL REFERENCES allocations(allocation_id),
  attendance_date DATE NOT NULL,
  status VARCHAR(20) NOT NULL, -- 'PRESENT', 'ABSENT'
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(allocation_id, attendance_date)
);

CREATE INDEX idx_attendance_allocation ON attendance(allocation_id);
CREATE INDEX idx_attendance_date ON attendance(attendance_date);
```

**Columns**:
- `attendance_id`: Unique identifier
- `allocation_id`: Reference to allocations
- `attendance_date`: Date of attendance
- `status`: Present/Absent

---

### 4.12 GradingSchemes Table

**Purpose**: Define grading structure per course

```sql
CREATE TABLE grading_schemes (
  scheme_id SERIAL PRIMARY KEY,
  course_offering_id INT NOT NULL UNIQUE REFERENCES course_offerings(course_offering_id),
  scheme_name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_schemes_offering ON grading_schemes(course_offering_id);
```

---

### 4.13 GradeComponents Table

**Purpose**: Components within a grading scheme

```sql
CREATE TABLE grade_components (
  component_id SERIAL PRIMARY KEY,
  scheme_id INT NOT NULL REFERENCES grading_schemes(scheme_id) ON DELETE CASCADE,
  component_name VARCHAR(100) NOT NULL,
  max_marks INT NOT NULL,
  weight_percentage INT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(scheme_id, component_name)
);

CREATE INDEX idx_components_scheme ON grade_components(scheme_id);
```

**Columns**:
- `component_id`: Unique identifier
- `scheme_id`: Reference to grading scheme
- `component_name`: Component name (e.g., "Attendance")
- `max_marks`: Maximum marks for component
- `weight_percentage`: Weight in overall grade

**Constraint**: Sum of weights must equal 100%

---

### 4.14 GradeEntries Table

**Purpose**: Individual student grades

```sql
CREATE TABLE grade_entries (
  entry_id SERIAL PRIMARY KEY,
  allocation_id INT NOT NULL REFERENCES allocations(allocation_id),
  component_id INT NOT NULL REFERENCES grade_components(component_id),
  marks_obtained INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(allocation_id, component_id)
);

CREATE INDEX idx_grades_allocation ON grade_entries(allocation_id);
CREATE INDEX idx_grades_component ON grade_entries(component_id);
```

**Columns**:
- `entry_id`: Unique identifier
- `allocation_id`: Reference to allocations
- `component_id`: Reference to grade component
- `marks_obtained`: Marks awarded

---

### 4.15 Gradebook Table

**Purpose**: Track gradebook state (Draft/Published)

```sql
CREATE TABLE gradebooks (
  gradebook_id SERIAL PRIMARY KEY,
  batch_id INT NOT NULL UNIQUE REFERENCES batches(batch_id),
  state VARCHAR(50) DEFAULT 'DRAFT', -- DRAFT, PUBLISHED
  published_date TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_gradebooks_batch ON gradebooks(batch_id);
```

---

### 4.16 Certificates Table

**Purpose**: Certificate issuance tracking

```sql
CREATE TABLE certificates (
  certificate_id SERIAL PRIMARY KEY,
  cert_number VARCHAR(50) UNIQUE NOT NULL,
  student_id INT NOT NULL REFERENCES students(student_id),
  course_offering_id INT NOT NULL REFERENCES course_offerings(course_offering_id),
  allocation_id INT NOT NULL REFERENCES allocations(allocation_id),
  teacher_id INT NOT NULL REFERENCES teachers(teacher_id),
  issue_date DATE NOT NULL,
  completion_date DATE NOT NULL,
  pdf_url VARCHAR(500),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_certs_student ON certificates(student_id);
CREATE INDEX idx_certs_offering ON certificates(course_offering_id);
CREATE INDEX idx_certs_number ON certificates(cert_number);
```

---

### 4.17 FeedbackForms Table

**Purpose**: Feedback form definitions

```sql
CREATE TABLE feedback_forms (
  form_id SERIAL PRIMARY KEY,
  course_offering_id INT NOT NULL UNIQUE REFERENCES course_offerings(course_offering_id),
  form_name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_forms_offering ON feedback_forms(course_offering_id);
```

---

### 4.18 FeedbackQuestions Table

**Purpose**: Questions in feedback forms

```sql
CREATE TABLE feedback_questions (
  question_id SERIAL PRIMARY KEY,
  form_id INT NOT NULL REFERENCES feedback_forms(form_id) ON DELETE CASCADE,
  question_text VARCHAR(500) NOT NULL,
  question_type VARCHAR(50) NOT NULL, -- TEXT, RATING, MULTIPLE_CHOICE
  is_required BOOLEAN DEFAULT TRUE,
  sequence_order INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_questions_form ON feedback_questions(form_id);
```

---

### 4.19 FeedbackResponses Table

**Purpose**: Student feedback submissions

```sql
CREATE TABLE feedback_responses (
  response_id SERIAL PRIMARY KEY,
  allocation_id INT NOT NULL REFERENCES allocations(allocation_id),
  question_id INT NOT NULL REFERENCES feedback_questions(question_id),
  response_text TEXT,
  rating INT, -- 1-5 for rating questions
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(allocation_id, question_id)
);

CREATE INDEX idx_responses_allocation ON feedback_responses(allocation_id);
CREATE INDEX idx_responses_question ON feedback_responses(question_id);
```

---

### 4.20 Grievances Table

**Purpose**: Student grievance management

```sql
CREATE TABLE grievances (
  grievance_id SERIAL PRIMARY KEY,
  student_id INT NOT NULL REFERENCES students(student_id),
  category VARCHAR(50) NOT NULL, -- ACADEMIC, ATTENDANCE, GRADING, OTHER
  description TEXT NOT NULL,
  severity VARCHAR(50) NOT NULL, -- LOW, MEDIUM, HIGH
  status VARCHAR(50) DEFAULT 'SUBMITTED', -- SUBMITTED, UNDER_REVIEW, RESOLVED, CLOSED
  resolution_notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_grievances_student ON grievances(student_id);
CREATE INDEX idx_grievances_status ON grievances(status);
CREATE INDEX idx_grievances_category ON grievances(category);
```

---

## 5. Indexes

**Purpose**: Optimize query performance

| Table | Column(s) | Type | Purpose |
|-------|-----------|------|---------|
| users | email | Unique | Login queries |
| students | user_id, roll_number | Unique | Student lookup |
| enrollments | student_id, course_offering_id | Unique | Prevent duplicates |
| allocations | student_id, batch_id | Unique | Batch assignment |
| attendance | allocation_id, attendance_date | Unique | Daily attendance |
| grade_entries | allocation_id, component_id | Unique | Grade entry |

---

## 6. Constraints

### 6.1 Primary Keys
All tables have `PRIMARY KEY` constraint on their `id` field.

### 6.2 Foreign Keys
All relationship fields have `FOREIGN KEY` constraints with `ON DELETE CASCADE` where appropriate.

### 6.3 Unique Constraints
- `users.email`: Unique
- `students.roll_number`: Unique
- `students.user_id`: Unique (1:1 with users)
- `courses.course_code`: Unique
- `enrollments(student_id, course_offering_id)`: Unique
- `allocations(student_id, batch_id)`: Unique
- `attendance(allocation_id, attendance_date)`: Unique
- `certificates.cert_number`: Unique

### 6.4 Check Constraints
- `course_offerings.max_capacity > 0`
- `batches.batch_capacity > 0`
- `grade_components.weight_percentage >= 0 AND <= 100`
- `grievances.severity IN ('LOW', 'MEDIUM', 'HIGH')`

---

## 7. Data Integrity

### 7.1 Referential Integrity
- Cascading delete: Deletion of parent deletes children
- Foreign key constraints enforced at database level

### 7.2 Data Consistency
- Enrollment unique per student per course
- Allocation unique per student per batch
- One gradebook per batch
- Grade weight sum = 100%

---

## 8. Query Performance Considerations

### 8.1 Common Queries
```sql
-- Student login
SELECT * FROM users WHERE email = ? AND is_active = TRUE;

-- Student enrollments
SELECT e.*, co.course_name, co.semester
FROM enrollments e
JOIN course_offerings co ON e.course_offering_id = co.course_offering_id
WHERE e.student_id = ?;

-- Batch allocation with students
SELECT a.*, s.name, b.batch_name
FROM allocations a
JOIN students s ON a.student_id = s.student_id
JOIN batches b ON a.batch_id = b.batch_id
WHERE a.batch_id = ?;

-- Attendance percentage
SELECT COUNT(*) FILTER (WHERE status = 'PRESENT') * 100.0 / COUNT(*) as attendance_pct
FROM attendance
WHERE allocation_id = ?;

-- Overall grade calculation
SELECT
  a.student_id,
  SUM(ge.marks_obtained * gc.weight_percentage / 100.0) as overall_grade
FROM grade_entries ge
JOIN grade_components gc ON ge.component_id = gc.component_id
JOIN allocations a ON ge.allocation_id = a.allocation_id
WHERE a.batch_id = ?
GROUP BY a.student_id;
```

---

## 9. Backup & Recovery

- Daily automated backups (Supabase managed)
- Backup retention: 30 days
- Point-in-time recovery available
- Test recovery quarterly

---

## 10. Related Documents

- [System Architecture](01-System-Architecture.md)
- [RBAC Design](03-RBAC-Design.md)
- [Functional Requirements](../SRS/05-Functional-Requirements.md)
