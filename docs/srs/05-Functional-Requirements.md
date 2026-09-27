# Functional Requirements

## Overview

This document details all functional requirements for the KISS Vocational Training Management Portal organized by user role and module.

## Table of Contents

1. [Authentication & Authorization](#1-authentication--authorization)
2. [Student Modules](#2-student-modules)
3. [Teacher Modules](#3-teacher-modules)
4. [Admin Modules](#4-admin-modules)
5. [System Workflows](#5-system-workflows)

---

## 1. Authentication & Authorization

### FR-AUTH-001: Single Login Portal
**Description**: The system shall provide a unified login page for all user roles.

**Requirements**:
- Single login endpoint accessible to all users
- User email and password authentication
- Session timeout after 30 minutes of inactivity
- "Remember me" option for optional persistent sessions
- User role determination from Users table after successful authentication

**Acceptance Criteria**:
- User can log in with valid credentials
- Session is created with JWT token
- User role is correctly determined and assigned
- Logout clears session and JWT token
- Inactivity timeout logs user out

### FR-AUTH-002: Role-Based Access Control
**Description**: The system shall enforce role-based access control (RBAC) for all modules and features.

**Requirements**:
- Three distinct roles: Student, Teacher, Admin
- Role assignment during user creation by Admin
- Role stored in Users table
- Access to modules controlled by role
- Unauthorized access attempts logged

**Acceptance Criteria**:
- Student can only access student modules
- Teacher can only access teacher modules
- Admin can only access admin modules
- Role changes by admin take effect immediately
- Access violation attempts are logged

### FR-AUTH-003: JWT Token Management
**Description**: The system shall use JWT tokens for stateless session management.

**Requirements**:
- JWT token issued on successful login
- Token includes user ID, role, email
- Token expiration set to 24 hours
- Token refresh mechanism available
- Token validation on each protected API call

**Acceptance Criteria**:
- Token is issued with correct payload
- Token contains all required claims
- Expired token is rejected
- Token can be refreshed before expiration
- Invalid token returns 401 Unauthorized

### FR-AUTH-004: Password Security
**Description**: Passwords shall be securely managed and validated.

**Requirements**:
- Minimum password length: 8 characters
- Require mix of uppercase, lowercase, numbers, special characters
- Password hashing using bcrypt (cost factor ≥ 10)
- Password change functionality available
- Password reset via email verification
- No password history reuse for last 5 passwords

**Acceptance Criteria**:
- Weak passwords are rejected
- Passwords are hashed before storage
- User can reset password via email
- Old password required for password change
- Password reset token expires in 1 hour

---

## 2. Student Modules

### FR-STU-001: Student Dashboard
**Description**: Students shall have a personalized dashboard upon login.

**Requirements**:
- Display enrolled courses with current batch assignment
- Quick-access cards for progress, grievances, certificates
- Upcoming deadlines and announcements
- Course progress percentage indicator
- Recently published grades notification

**Acceptance Criteria**:
- Dashboard loads within 2 seconds
- All widgets display correct data
- Announcements are displayed in reverse chronological order
- Grades are updated in real-time

### FR-STU-002: Student Profile Management
**Description**: Students shall be able to view their profile information.

**Requirements**:
- Display profile fields: Name, Roll Number, Section, Email, Phone, Address
- Display Mentor Name and Mentor Phone Number
- Profile is read-only for students
- Admins can edit student profiles
- Profile history is not displayed

**Acceptance Criteria**:
- Student can view all profile fields
- Student cannot modify profile fields
- Admin can edit fields via admin interface
- Changes by admin appear in student view within 1 minute

### FR-STU-003: Course Enrollment
**Description**: Students shall be able to enroll in courses during enrollment windows.

**Requirements**:
- Enrollment available only during admin-defined enrollment window
- Enrollment window can be staggered by roll-number ranges
- Student can enroll in exactly one course per semester
- Enrollment form displays available courses with capacity
- Enrollment confirmation via email
- Enrollment history is maintained

**Acceptance Criteria**:
- Student can see enrollment window status
- Can only enroll during active window
- Can only select one course
- Enrollment is confirmed with email receipt
- Cannot enroll if course capacity is full (subject to allocation)

### FR-STU-004: Progress Module
**Description**: Students shall view their progress in enrolled courses.

**Requirements**:
- Display attendance percentage
- Display component-wise marks (if published by teacher)
- Display overall grade percentage
- Display course completion status
- Display progress timeline/milestones

**Acceptance Criteria**:
- Attendance percentage is accurate
- Marks shown only if published (not draft)
- Overall grade calculated correctly
- Course status updates within 1 minute of teacher publish

### FR-STU-005: Grievance Center
**Description**: Students shall have a structured grievance management system.

**Requirements**:
- Students can lodge grievances with category, description, severity
- Grievance categories: Academic, Attendance, Grading, Other
- Grievance status tracking: Submitted, Under Review, Resolved, Closed
- Grievance history visible to student
- Email notifications on status changes

**Acceptance Criteria**:
- Student can create grievance with all fields
- Grievance appears in admin dashboard
- Status updates trigger email notification
- Student can view all past grievances

### FR-STU-006: Feedback Submission
**Description**: Students shall provide feedback on courses and teachers.

**Requirements**:
- Feedback available after course enrollment/completion
- Structured feedback form with admin-defined questions
- Rating scales and text responses supported
- Feedback is anonymous (no student name stored)
- Feedback history per course maintained

**Acceptance Criteria**:
- Student can submit feedback only for enrolled courses
- Feedback form displays all questions
- Submission is recorded without student identification
- Admin can view aggregated feedback

### FR-STU-007: Certificate Download
**Description**: Students shall be able to download certificates for completed courses.

**Requirements**:
- Certificate becomes available after teacher marks course as complete
- Certificate PDF displays: Student Name, Roll Number, Course Name, Batch, Teacher Name, Completion Date, Certificate ID
- Certificate ID is unique and non-sequential
- Certificate download as PDF
- Certificate history available

**Acceptance Criteria**:
- Certificate appears within 1 hour of course completion
- PDF contains all required fields
- Certificate ID is unique across system
- Student can download multiple times
- Certificate cannot be modified after generation

---

## 3. Teacher Modules

### FR-TEA-001: Teacher Dashboard
**Description**: Teachers shall have a dashboard showing assigned courses.

**Requirements**:
- Display list of assigned courses for current semester
- Display batch details (number of students, batch times)
- Quick-access cards for attendance, gradebook, course completion
- Course status indicator (ongoing, completed)
- Recent student submissions or pending tasks

**Acceptance Criteria**:
- Dashboard loads within 2 seconds
- Only assigned courses are displayed
- Student count is accurate
- Pending tasks are highlighted

### FR-TEA-002: Attendance Management
**Description**: Teachers shall take and manage course attendance.

**Requirements**:
- Attendance entry by batch
- Date and attendance status (Present/Absent) for each student
- Bulk attendance marking (mark all present, then toggle absents)
- Attendance history view with filter by date
- Attendance percentage calculation
- Attendance cannot be marked in future dates
- Attendance can be edited within 7 days of entry (with admin override possible)

**Acceptance Criteria**:
- Teacher can mark attendance for current date
- Cannot mark attendance for future dates
- Bulk marking works correctly
- Attendance percentage calculated accurately
- Edit history is logged

### FR-TEA-003: Gradebook Management
**Description**: Teachers shall manage student grades with draft and publish workflows.

**Requirements**:
- Gradebook displays all students in batch
- Grade components defined by admin grading scheme
- Teachers can enter marks for each component
- Marks validated against component maximum
- Gradebook lifecycle: Draft → Published → Visible to Students
- Draft state allows editing and saving without publication
- Published state is immutable (teachers cannot edit after publishing)
- Version history maintained

**Acceptance Criteria**:
- Teacher can enter marks for all components
- Component marks cannot exceed max
- Draft state allows multiple saves
- Publish action locks gradebook
- Students cannot see draft grades
- Students see published grades within 1 minute

### FR-TEA-004: Mark Publishing
**Description**: Teachers shall publish grades to make them visible to students.

**Requirements**:
- Publish action converts gradebook from Draft to Published
- Overall grade calculated as per grading scheme
- Validation: At least one student has at least one mark entered
- Publish confirmation dialog with course details
- Email notification sent to all students when grades are published
- Published status cannot be reverted

**Acceptance Criteria**:
- Publish button triggers validation
- Overall grade is calculated correctly
- Students receive email upon publishing
- Grades appear in student progress module

### FR-TEA-005: Course Completion Marking
**Description**: Teachers shall mark courses as complete.

**Requirements**:
- Available only when course end date has passed
- Marks all students in the course as having completed
- Triggers automatic certificate generation for all students
- Course completion date recorded
- Email notification sent to all students

**Acceptance Criteria**:
- Cannot mark complete before end date
- All students marked complete
- Certificates generated within 1 hour
- Students notified via email
- Course status changes to "Completed"

---

## 4. Admin Modules

### FR-ADM-001: Admin Dashboard (Command Center)
**Description**: Admins shall have a centralized dashboard with real-time trackers.

**Requirements**:
- Live Enrollment Tracker: Current enrollment count, total capacity, fill percentage by course
- Allocation Tracker: Allocated batches, pending allocations, allocation status
- Feedback Tracker: Total feedback received, average rating, pending review count
- Grievance Tracker: Total grievances, by status, resolution time metrics
- All widgets display real-time data
- Drill-down capability to view details

**Acceptance Criteria**:
- All trackers update within 1 minute
- Data is accurate and matches source tables
- Drill-down shows detailed list
- Dashboard loads within 3 seconds

### FR-ADM-002: Create Semester
**Description**: Admins shall configure and create new semesters.

**Requirements**:
- Define semester name, start date, end date
- Configure courses, seats, batches, teachers
- Define grading schemes and feedback forms
- Set enrollment window dates and roll-number ranges

**Acceptance Criteria**:
- Semester created with all configuration
- Configuration can be reviewed before saving
- Semester appears in system immediately
- Configuration changes only affect new enrollments

### FR-ADM-003: Course Offering Management
**Description**: Admins shall create and manage course offerings.

**Requirements**:
- Select base course
- Define maximum enrollment capacity
- Define batch sizes and number of batches
- Assign teachers to course offering
- Set enrollment window (date range and roll-number range)
- Course offering linked to specific semester

**Acceptance Criteria**:
- Course offering created with full configuration
- Capacity correctly enforced during allocation
- Multiple batches are created with correct sizes
- Teacher assignments appear in teacher dashboard

### FR-ADM-004: Seat Configuration
**Description**: Admins shall configure seat allocation for courses.

**Requirements**:
- Define total seats per course offering
- Define batch structure (number of batches, seats per batch)
- Batch seats must sum to total seats
- Seat allocation cannot be changed after allocation run

**Acceptance Criteria**:
- Seat configuration saved and validated
- Sum of batch seats equals total seats
- Configuration appears in allocation settings

### FR-ADM-005: Batch Configuration
**Description**: Admins shall create and configure batches.

**Requirements**:
- Create multiple batches for each course offering
- Assign capacity to each batch
- Assign teacher to each batch
- Set batch timing (if applicable)
- Batches created automatically based on seat configuration

**Acceptance Criteria**:
- Batches created with correct capacity
- Teacher assigned to batch
- Batches visible in course offering details

### FR-ADM-006: Teacher Assignment
**Description**: Admins shall assign teachers to courses and batches.

**Requirements**:
- Assign one teacher per batch
- One teacher can teach multiple batches
- Assignment linked to specific semester
- Teacher can view assigned courses in dashboard

**Acceptance Criteria**:
- Assignment completed without errors
- Teacher sees course in their dashboard
- Change in assignment updates immediately

### FR-ADM-007: Grading Scheme Definition
**Description**: Admins shall define grading schemes with weighted components.

**Requirements**:
- Create grading scheme with multiple components
- Each component has: Name, Max Marks, Weight Percentage
- Sum of weights must equal 100%
- Example components: Attendance (10), Activity Record (10), Monthly Project 1-4 (5 each), Final Assessment (60)
- One grading scheme per course offering

**Acceptance Criteria**:
- Scheme created with all components
- Weight sum validated to 100%
- Overall grade formula: Sum of (Component Marks × Weight)
- Teachers use scheme when entering marks

### FR-ADM-008: Feedback Form Creation
**Description**: Admins shall design feedback forms.

**Requirements**:
- Create feedback form with multiple questions
- Question types: Text, Rating (1-5), Multiple Choice
- Required vs. Optional questions
- Form linked to course offering
- Form available to students after enrollment

**Acceptance Criteria**:
- Form created with all question types
- Form appears in student feedback interface
- Responses are collected and stored

### FR-ADM-009: Enrollment Window Management
**Description**: Admins shall manage enrollment windows.

**Requirements**:
- Define enrollment window date range
- Define roll-number ranges for staggered enrollment (e.g., Roll 001-100, 101-200)
- Multiple staggered windows supported
- Enrollment available only during active window
- Window status visible to students

**Acceptance Criteria**:
- Window created with dates and roll ranges
- Enrollment restricted to window period
- Roll-number range correctly enforced
- Students see window status in dashboard

### FR-ADM-010: Run Allocation
**Description**: Admins shall run the batch allocation algorithm.

**Requirements**:
- Run only after enrollment window closure
- Algorithm: Fetch enrollments, shuffle randomly, fill batches respecting capacity, create Allocation records
- Randomization seed logged for reproducibility
- Allocation report generated (students per batch, capacity utilization)
- Allocation failure scenario handled (more students than seats)
- Students notified of allocation via email

**Acceptance Criteria**:
- Allocation completes without errors
- All enrolled students are allocated
- Batch capacities are not exceeded
- Allocation report is accurate
- Students receive allocation email

### FR-ADM-011: Batch Switching Requests
**Description**: Admins shall manage batch switching requests.

**Requirements**:
- Students can request to switch from current batch to another batch of same course
- One switch request allowed per student per course
- Switch requires admin approval
- Approval checks capacity of target batch
- Approved switch updates student allocation
- Rejected switch returns to original batch

**Acceptance Criteria**:
- Student can submit switch request with reason
- Request appears in admin dashboard
- Admin can approve/reject with comment
- Capacity check prevents overbooking
- Student notified of decision via email

### FR-ADM-012: Report Generation
**Description**: Admins shall generate various reports.

**Requirements**:

#### Certificate Reports
- List of all certificates issued
- Student name, roll number, certificate ID, issue date
- Export to PDF/CSV

#### Student Reports
- Student-wise attendance and performance
- Progress toward completion
- Grievances and feedback submitted
- Export to PDF/CSV

#### Course Reports
- Course-wise enrollment, allocation, completion
- Aggregate performance metrics
- Teacher performance (if applicable)
- Export to PDF/CSV

**Acceptance Criteria**:
- Reports generated accurately
- Data matches source tables
- Export functionality works
- Reports can be filtered by date range

### FR-ADM-013: Admin Profile & Settings
**Description**: Admin user management and system settings.

**Requirements**:
- Admins can view and update their own profile
- Super-admin can create/deactivate admin accounts
- System configuration settings (email server, logo, etc.)
- Audit log viewer (system activity tracking)

**Acceptance Criteria**:
- Admin can update profile
- New admin account created successfully
- Settings changes take effect immediately
- Audit log shows all system actions

---

## 5. System Workflows

### FR-SYS-001: Login Workflow
**Description**: Complete login process from user perspective.

**Flow**:
1. User navigates to system
2. Login page displays
3. User enters email and password
4. System validates credentials
5. System determines user role
6. System redirects to role-appropriate dashboard
7. Session created with JWT token

**Acceptance Criteria**:
- Login succeeds with valid credentials
- Invalid credentials rejected
- Role-based redirection works
- Dashboard loads with user data

### FR-SYS-002: Enrollment Workflow
**Description**: Complete student enrollment process.

**Flow**:
1. Admin creates course offering and enrollment window
2. Admin configures seats and batches
3. Enrollment window opens
4. Student logs in and navigates to enrollment
5. Student selects desired course
6. System checks eligibility and capacity
7. Student confirms enrollment
8. Enrollment saved to database
9. Confirmation email sent to student
10. Student sees enrolled course in dashboard

**Acceptance Criteria**:
- Enrollment available during window only
- Capacity checking works
- Enrollment confirmed with email
- Course appears in student profile

### FR-SYS-003: Batch Allocation Workflow
**Description**: Batch allocation process after enrollment closure.

**Flow**:
1. Admin closes enrollment window
2. Admin navigates to Run Allocation
3. Admin confirms allocation run
4. System fetches all enrollments
5. System shuffles enrollments randomly
6. System fills batches respecting capacity
7. System creates Allocation records
8. Allocation report generated
9. Email sent to all students with allocation details
10. Students see batch assignment in Progress module

**Acceptance Criteria**:
- Allocation runs only after window closure
- All students allocated (if enough seats)
- Batch capacities not exceeded
- Report is accurate
- Students notified

### FR-SYS-004: Attendance & Grading Workflow
**Description**: Complete marking and grading process.

**Flow**:
1. Teacher logs in and navigates to Manage Course
2. Teacher takes attendance for batch on specific date
3. Teacher navigates to Gradebook
4. Teacher enters marks for each component in Draft state
5. Teacher saves grades (remains in Draft)
6. Teacher reviews grades for accuracy
7. Teacher publishes gradebook
8. System calculates overall grade
9. Students receive email of grade publication
10. Students view grades in Progress module

**Acceptance Criteria**:
- Attendance and marks linked
- Draft state allows editing
- Publish locks gradebook
- Students see grades after publish
- Email notifications sent

### FR-SYS-005: Certificate Generation Workflow
**Description**: Automatic certificate generation and download.

**Flow**:
1. Teacher marks course as complete
2. System identifies all students in course
3. System generates certificate for each student
4. Certificate includes all dynamic fields
5. Certificate saved with unique Certificate ID
6. Email notification sent to students
7. Student downloads certificate from dashboard
8. Certificate PDF generated on-demand

**Acceptance Criteria**:
- Certificates generated within 1 hour
- All fields populated correctly
- Certificate ID is unique
- PDF downloads without error
- Certificates cannot be modified

---

## Functional Requirements Summary

| Module | FR Count | Critical | High | Medium |
|--------|----------|----------|------|--------|
| Authentication | 4 | 4 | 0 | 0 |
| Student | 7 | 5 | 2 | 0 |
| Teacher | 5 | 4 | 1 | 0 |
| Admin | 13 | 10 | 3 | 0 |
| System Workflows | 5 | 5 | 0 | 0 |
| **TOTAL** | **34** | **28** | **6** | **0** |

## Related Documents

- [Non-Functional Requirements](06-Non-Functional-Requirements.md)
- [Use Cases](07-Use-Cases.md)
- [System Design Document](../SDD/01-System-Architecture.md)
