# Assumptions

## Overview

This document lists all assumptions made during the requirements gathering and system design phases of the KISS Vocational Training Management Portal.

## Organizational Assumptions

### AS-ORG-001: Stable User Base
**Assumption**: Student and teacher roles will remain relatively stable within a semester.

**Basis**: Typical academic calendar structure

**Impact**: Affects capacity planning and resource allocation

**Mitigation**: System allows adding users mid-semester (admin function)

---

### AS-ORG-002: Semester Structure
**Assumption**: Vocational training is organized into distinct semesters with defined start and end dates.

**Basis**: KIIT operates on semester system

**Impact**: CourseOffering tied to specific semester

**Mitigation**: System supports continuous enrollment if needed

---

### AS-ORG-003: Single Mentor per Student
**Assumption**: Each student is assigned a single mentor for guidance.

**Basis**: Current KISS program structure

**Impact**: Profile includes single Mentor Name and Phone

**Mitigation**: Can be extended to multiple mentors in future

---

### AS-ORG-004: One Enrollment per Semester
**Assumption**: Students enroll in only one course per semester.

**Basis**: Vocational training program design

**Impact**: Simplifies enrollment and allocation logic

**Mitigation**: Can be extended to multiple enrollments in future

---

## Technical Assumptions

### AS-TECH-001: Cloud Hosting Infrastructure
**Assumption**: System will be deployed on cloud infrastructure (Supabase/PostgreSQL).

**Basis**: Requirement specification mandates cloud deployment

**Impact**: Affects scalability and availability design

**Mitigation**: Infrastructure designed for cloud-native deployment

---

### AS-TECH-002: Reliable Internet Connectivity
**Assumption**: Users have consistent internet access during operational hours.

**Basis**: System is web-based, requires internet

**Impact**: No offline mode required

**Mitigation**: System gracefully handles temporary disconnections

---

### AS-TECH-003: Modern Browser Support
**Assumption**: Users access system via modern web browsers (Chrome, Firefox, Safari, Edge).

**Basis**: Requirement for web-based system

**Impact**: Affects UI technology choices (React)

**Mitigation**: Responsive design supports multiple browsers and devices

---

### AS-TECH-004: Database Stability
**Assumption**: Supabase/PostgreSQL is stable, reliable, and meets performance requirements.

**Basis**: Specified in technology stack

**Impact**: Affects backup and recovery strategy

**Mitigation**: Regular backup and disaster recovery testing

---

### AS-TECH-005: Email Service Availability
**Assumption**: Email server is available for sending notifications.

**Basis**: System requires sending confirmation, notification emails

**Impact**: Affects communication workflow

**Mitigation**: Fallback to in-app notifications if email fails

---

### AS-TECH-006: JWT Token Security
**Assumption**: JWT token implementation is secure and follows industry best practices.

**Basis**: Specified in authentication design

**Impact**: Session management relies on JWT

**Mitigation**: Regular security audits of JWT implementation

---

## Data Assumptions

### AS-DATA-001: Data Accuracy
**Assumption**: Admin-provided data (student profiles, course details) is accurate and complete.

**Basis**: Data entered by trusted admin personnel

**Impact**: System does not validate student roll numbers against external source

**Mitigation**: Admin interface includes data validation; audit trail maintained

---

### AS-DATA-002: Unique Email Addresses
**Assumption**: Each user has a unique email address; emails are used as unique identifier.

**Basis**: Standard practice in web systems

**Impact**: Affects login mechanism and user identification

**Mitigation**: System enforces email uniqueness constraint at database level

---

### AS-DATA-003: Consistent Roll Number Format
**Assumption**: Student roll numbers follow consistent format (e.g., numeric with leading zeros).

**Basis**: KIIT institutional standard

**Impact**: Used for staggered enrollment windows (roll-number ranges)

**Mitigation**: Admin interface validates roll number format during user creation

---

### AS-DATA-004: Historical Data Retention
**Assumption**: System maintains historical data for auditing; data is not deleted (only archived).

**Basis**: Compliance and reporting requirements

**Impact**: Database grows over time; archival strategy needed

**Mitigation**: Regular data archival and cleanup of very old data

---

## Process Assumptions

### AS-PROC-001: Defined Enrollment Windows
**Assumption**: Admin explicitly defines and activates enrollment windows; no automatic enrollment.

**Basis**: System requires admin to manage process

**Impact**: Manual process, not automatic

**Mitigation**: Clear documentation for admin on enrollment window management

---

### AS-PROC-002: Random Allocation Fairness
**Assumption**: Random shuffling algorithm provides fair batch allocation.

**Basis**: No preference-based or merit-based allocation defined

**Impact**: All enrolled students treated equally in allocation

**Mitigation**: Allocation algorithm tested thoroughly; seed logged for reproducibility

---

### AS-PROC-003: Single Batch Switching Allowed
**Assumption**: Each student can request batch switch only once per course.

**Basis**: Business rule in requirements

**Impact**: Switch counter incremented per enrollment

**Mitigation**: System tracks switch count; enforces single switch limit

---

### AS-PROC-004: Teacher Grading Honesty
**Assumption**: Teachers accurately and honestly enter student marks.

**Basis**: System relies on teacher integrity

**Impact**: No algorithmic fraud detection

**Mitigation**: Admin can audit and override grades if needed; grade ranges validated

---

### AS-PROC-005: Course Completion Marking
**Assumption**: Teacher will mark course complete only after all students assessed.

**Basis**: Business requirement for course completion

**Impact**: Certificates generated only after explicit marking

**Mitigation**: Teacher checklist before marking complete; audit trail maintained

---

## Security Assumptions

### AS-SEC-001: Secure Environment Deployment
**Assumption**: System deployed on secure, hardened servers with appropriate security measures.

**Basis**: Requirement for secure system

**Impact**: Assumes infrastructure security is handled separately

**Mitigation**: Security requirements specified for infrastructure setup

---

### AS-SEC-002: No Shared User Accounts
**Assumption**: Each user has unique account; no shared login credentials.

**Basis**: RBAC and audit trail requirements

**Impact**: Affects user account policy

**Mitigation**: Policy enforced during user provisioning

---

### AS-SEC-003: Regular Security Updates
**Assumption**: Application, framework, and library security vulnerabilities patched regularly.

**Basis**: Industry best practice

**Impact**: Requires dedicated DevOps team for patch management

**Mitigation**: Automated dependency scanning and patch management tools

---

### AS-SEC-004: No SQL Injection Vulnerabilities
**Assumption**: All database queries use parameterized queries/ORM to prevent SQL injection.

**Basis**: Security requirement for web applications

**Impact**: Affects development practices

**Mitigation**: Code review process enforces parameterized queries

---

## Business Assumptions

### AS-BUS-001: Sufficient IT Support
**Assumption**: KIIT has dedicated IT support team for system administration and operations.

**Basis**: System requires ongoing maintenance and support

**Impact**: Affects training and support requirements

**Mitigation**: Comprehensive documentation and training provided

---

### AS-BUS-002: Budget Availability
**Assumption**: Sufficient budget allocated for cloud hosting, maintenance, and support.

**Basis**: System requires ongoing operational costs

**Impact**: Affects system sustainability

**Mitigation**: Cost estimates provided to stakeholders

---

### AS-BUS-003: User Adoption
**Assumption**: Students and teachers will adopt and use system regularly.

**Basis**: System success depends on user adoption

**Impact**: Requires training and change management

**Mitigation**: User training sessions and documentation provided

---

### AS-BUS-004: No Major Regulatory Changes
**Assumption**: Relevant regulatory environment remains stable during system deployment.

**Basis**: System design based on current regulations

**Impact**: May require modifications if regulations change

**Mitigation**: Regular compliance review and update process

---

## Assumptions Risk Matrix

| Assumption | Risk Level | Impact | Mitigation |
|-----------|-----------|--------|-----------|
| AS-ORG-001 | Low | Medium | Allow mid-semester additions |
| AS-TECH-001 | Low | High | Cloud-native design |
| AS-TECH-002 | Medium | High | Handle disconnections gracefully |
| AS-TECH-003 | Low | Low | Responsive design |
| AS-DATA-001 | Medium | High | Data validation & audit |
| AS-PROC-001 | Low | Low | Clear documentation |
| AS-PROC-002 | Medium | Medium | Algorithm testing |
| AS-SEC-001 | Medium | High | Security requirements spec |
| AS-BUS-001 | Medium | Medium | Training & documentation |
| AS-BUS-002 | Medium | High | Budget planning |

## Related Documents

- [Scope](03-Scope.md)
- [Constraints](09-Constraints.md)
- [Non-Functional Requirements](06-Non-Functional-Requirements.md)
