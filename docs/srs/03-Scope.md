# Scope

## System Scope

### What the System WILL Do

#### Core Functionality

1. **User Authentication & Authorization**
   - Single login portal for all users
   - Role-based access control (Student, Teacher, Admin)
   - JWT-based session management

2. **Student Modules**
   - Dashboard with quick-access widgets
   - Profile management (read-only viewing, admin-editable)
   - Course enrollment during designated enrollment windows
   - Progress tracking (attendance, marks, course completion status)
   - Grievance center for lodging concerns
   - Feedback submission on courses and teachers
   - Certificate download after course completion

3. **Teacher Modules**
   - Dashboard showing assigned courses
   - Attendance management for batches
   - Gradebook with draft and publish workflows
   - Grade publishing with validation
   - Course completion marking
   - View student enrollment and allocation data

4. **Admin Modules**
   - Command center dashboard with real-time trackers
   - Semester creation and configuration
   - Course offering management
   - Grading scheme definition
   - Enrollment window management
   - Batch allocation (random shuffle algorithm)
   - Batch switching request approval
   - Report generation (certificates, student, course)
   - Feedback and grievance tracking

5. **Course Management**
   - Separate Course and CourseOffering entities
   - Seat and batch configuration per semester
   - Teacher-to-course allocation
   - Prerequisite and dependency tracking (if applicable)

6. **Enrollment & Allocation**
   - Staggered enrollment windows by roll-number ranges
   - Single course enrollment per student
   - Automated batch allocation using randomization
   - One-time batch switching with admin approval

7. **Grading & Assessment**
   - Admin-defined grading schemes with weighted components
   - Component-wise mark entry
   - Draft gradebook state
   - Publication workflow with student visibility
   - Attendance tracking integrated with grading

8. **Certificate Management**
   - Automatic certificate generation upon course completion
   - Single universal template with dynamic field insertion
   - Certificate download interface for students
   - Certificate tracking and uniqueness guarantee (Certificate ID)

9. **Feedback & Grievance Management**
   - Structured feedback forms per course
   - Grievance categorization and status tracking
   - Admin visibility into all feedback and grievances
   - Historical tracking and archival

10. **Reporting**
    - Student-wise reports (attendance, marks, certificates)
    - Course-wise reports (enrollment, completion, performance)
    - Institutional reports (aggregate statistics)
    - Export functionality (PDF, CSV)

### What the System WILL NOT Do

1. **Out of Scope Features**
   - Resource allocation (classroom, lab scheduling)
   - Hostel management
   - Mess management
   - Transportation management
   - Any modules unrelated to vocational training
   - Integration with external ERP systems (Tally, SAP)
   - Financial/payroll management
   - Library management

2. **Not Included in MVP**
   - Video lecture hosting
   - Live classroom features
   - Discussion forums
   - Mobile app (web-responsive only)
   - Integration with third-party LMS platforms
   - Customizable grading schemes per teacher

3. **UI/UX Not Specified**
   - This SRS does not define wireframes, mockups, or design specifications
   - UI/UX will be designed by the development team
   - Refer to [UI-UX Documentation](../UI-UX/README.md) for approach

## Functional Boundaries

### System Interfaces

| Interface | Scope |
|-----------|-------|
| **User Interface** | Web-based, responsive, React frontend |
| **Database** | PostgreSQL via Supabase |
| **Authentication** | JWT tokens, role-based authorization |
| **External Services** | None in MVP (future: email notifications, SMS) |
| **Batch Processing** | Nightly allocation runs; certificate generation jobs |

### User Roles

| Role | Managed By | Can Enroll | Can Grade | Can Allocate |
|------|-----------|-----------|-----------|--------------|
| **Student** | Admin | Yes (self) | No | No |
| **Teacher** | Admin | No | Yes (own students) | No |
| **Admin** | System Setup | No | No | Yes |

### Data Scope

The system will manage the following data entities:

- User accounts and roles
- Student profiles and enrollment history
- Teacher assignments and course allocations
- Courses and course offerings (per semester)
- Batches and batch allocations
- Attendance records
- Grade entries and gradebooks
- Feedback and grievance records
- Certificates and issuance records

Historical data will be retained indefinitely for audit and reporting purposes.

## Operational Scope

### Deployment Environment

- **Hosting**: Cloud-based (Supabase)
- **Availability**: 24/7 with scheduled maintenance windows
- **Scalability**: Support 1000+ concurrent users within first 2 years
- **Compliance**: Data privacy and security standards applicable to Indian institutions

### Batch Processing

- Enrollment window management: Manual activation by admin
- Batch allocation: Scheduled nightly after enrollment closure
- Certificate generation: Automatic within 1 hour of course completion marking
- Reports: On-demand generation by admin

## Scope Exclusions Summary

The following are explicitly **NOT** part of this project:

1. Hardware provisioning or infrastructure setup
2. Physical security or access control systems
3. Third-party integrations (external ERP, payment gateways)
4. Custom reporting beyond the defined reports
5. Multilingual support beyond English
6. Video streaming or rich media hosting
7. Real-time notifications (initial version)
8. Mobile app development

## Assumptions Related to Scope

See [Assumptions](08-Assumptions.md) for assumptions that affect scope determination.

## Related Documents

- [Problem Statement](02-Problem-Statement.md)
- [Stakeholders](04-Stakeholders.md)
- [Functional Requirements](05-Functional-Requirements.md)
- [Constraints](09-Constraints.md)
