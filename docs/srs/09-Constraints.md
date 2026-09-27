# Constraints

## Overview

This document identifies constraints that limit system design, implementation, and operations.

## Technical Constraints

### CON-TECH-001: Technology Stack
**Constraint**: System must be built using specified technology stack.

**Details**:
- Frontend: React
- Backend: Spring Boot, Spring Security
- Database: PostgreSQL via Supabase
- Build Tool: Maven
- Authentication: JWT

**Rationale**: Aligned with organizational standards and team expertise

**Impact**: No flexibility in technology choices; must use specified stack

**Mitigation**: Team has expertise in all specified technologies

---

### CON-TECH-002: Database Platform
**Constraint**: Database must be PostgreSQL hosted on Supabase.

**Details**:
- No flexibility to use other databases
- Bound to Supabase infrastructure
- PostgreSQL version 12+

**Rationale**: Organizational decision for managed database service

**Impact**: Cannot migrate to other databases without major redesign

**Mitigation**: Portable schema design; no vendor-specific extensions

---

### CON-TECH-003: Deployment Platform
**Constraint**: System deployed on cloud infrastructure only.

**Details**:
- No on-premise deployment
- Cloud hosting required (AWS, Azure, or Supabase)
- No legacy system integration

**Rationale**: Scalability and maintenance requirements

**Impact**: Ongoing cloud hosting costs; dependency on cloud provider

**Mitigation**: Cost budgeting; multi-cloud strategy if needed

---

### CON-TECH-004: No Third-Party Integration (MVP)
**Constraint**: MVP does not include integration with external systems.

**Details**:
- No ERP system integration
- No payment gateway integration
- No external LMS integration
- All data managed within system

**Rationale**: Scope limitation for initial release

**Impact**: Future integrations will require additional development

**Mitigation**: API design supports future integrations

---

### CON-TECH-005: No Video Streaming
**Constraint**: System does not include video hosting or live streaming.

**Details**:
- No lecture video storage
- No live class streaming
- No video conference integration

**Rationale**: MVP scope limitation

**Impact**: Training must use external video platforms

**Mitigation**: System provides links to external video resources

---

## Business Constraints

### CON-BUS-001: Project Timeline
**Constraint**: System must be delivered within agreed timeline.

**Details**:
- MVP delivery: 6 months from project start
- Deployment: By start of next academic year
- No scope creep beyond frozen architecture

**Rationale**: Academic calendar alignment

**Impact**: Scope must be strictly controlled; only critical features in MVP

**Mitigation**: Change control process; clear scope definition

---

### CON-BUS-002: Budget Limitation
**Constraint**: Project budget is fixed and cannot be exceeded.

**Details**:
- Development cost capped at budget
- Cloud hosting cost budgeted annually
- No allocation for scope expansion

**Rationale**: Institutional budget constraints

**Impact**: Resource allocation limited; efficiency required

**Mitigation**: Careful resource planning; prioritization of features

---

### CON-BUS-003: Single Language Support
**Constraint**: Initial system supports English only.

**Details**:
- No multilingual support in MVP
- Future enhancement for regional languages
- Documentation in English

**Rationale**: MVP scope limitation

**Impact**: System not accessible to non-English speakers initially

**Mitigation**: Internationalization framework in place for future

---

### CON-BUS-004: Frozen Architecture
**Constraint**: System architecture is frozen; no redesign allowed.

**Details**:
- Architecture decisions locked
- No major technology changes
- Design documented in SDD
- Implementation must follow design

**Rationale**: Stakeholder agreement; design finalization

**Impact**: No flexibility in system design; must follow locked architecture

**Mitigation**: Comprehensive design review before freezing

---

## Organizational Constraints

### CON-ORG-001: Organizational Structure
**Constraint**: System must fit within KIIT's organizational hierarchy.

**Details**:
- Three-role RBAC: Student, Teacher, Admin
- Reports to Program Director
- No support for other roles (e.g., Department Head)

**Rationale**: Program structure is fixed

**Impact**: Cannot extend roles without redesign

**Mitigation**: Role model designed for extensibility

---

### CON-ORG-002: No Custom Grading Schemes per Teacher
**Constraint**: Grading schemes defined centrally by admin, not customizable per teacher.

**Details**:
- Admin defines scheme once per course offering
- Teachers cannot modify scheme
- All teachers use same scheme

**Rationale**: Grade consistency and fairness

**Impact**: Teachers have no flexibility in grading structure

**Mitigation**: Admin can create multiple schemes; teacher selects from predefined options

---

### CON-ORG-003: No Prerequisite System (MVP)
**Constraint**: MVP does not include course prerequisites.

**Details**:
- All students eligible for all courses
- No validation of prerequisite completion
- Future enhancement possible

**Rationale**: MVP scope limitation

**Impact**: Cannot enforce course prerequisites

**Mitigation**: Can be added in future version

---

## Data Constraints

### CON-DATA-001: Data Retention Policy
**Constraint**: Historical data retained for minimum 5 years for audit purposes.

**Details**:
- No deletion of production data
- Archival of very old data optional
- Audit trail maintained indefinitely

**Rationale**: Compliance and audit requirements

**Impact**: Database grows continuously; archival strategy needed

**Mitigation**: Regular archival; index optimization

---

### CON-DATA-002: Data Privacy
**Constraint**: Student data must be protected according to privacy regulations.

**Details**:
- Personally identifiable information (PII) encrypted
- Access logs maintained
- Data deletion on request (if legally possible)

**Rationale**: Data privacy regulations (GDPR, India standards)

**Impact**: Affects data storage and access patterns

**Mitigation**: Encryption, access control, audit logging

---

### CON-DATA-003: No Student Data Export
**Constraint**: Students cannot export their own data in initial version.

**Details**:
- Students view data only through UI
- No bulk data export in CSV/JSON
- Admin can generate reports

**Rationale**: MVP scope; data security

**Impact**: No data portability for students initially

**Mitigation**: Can be added in future version

---

## Operational Constraints

### CON-OPS-001: Maintenance Windows
**Constraint**: System updates only during scheduled maintenance windows.

**Details**:
- Maintenance: Weekends, 10 PM - 6 AM IST
- No emergency patches during business hours
- Advance notice required

**Rationale**: Minimize disruption to users

**Impact**: Urgent bugs may wait until next maintenance window

**Mitigation**: Critical patch can be deployed immediately if necessary

---

### CON-OPS-002: Single Admin Team
**Constraint**: System operated by single admin team (2-3 people).

**Details**:
- No distributed admin capability
- All admin functions centralized
- No delegated admin roles

**Rationale**: Organizational structure

**Impact**: Requires comprehensive admin documentation

**Mitigation**: Automation of routine tasks; clear procedures

---

### CON-OPS-003: No Real-Time Notifications (MVP)
**Constraint**: Initial version uses email and in-app notifications only.

**Details**:
- No push notifications
- No SMS notifications
- No real-time alerts

**Rationale**: MVP scope limitation

**Impact**: Students may not be immediately notified of updates

**Mitigation**: Can be added in future version

---

## Support & Maintenance Constraints

### CON-SUP-001: Support Hours
**Constraint**: Support provided during business hours only.

**Details**:
- Business hours: 9 AM - 6 PM IST, weekdays
- No 24/7 support in MVP
- On-call support for critical issues

**Rationale**: Resource limitation

**Impact**: Users cannot get help outside business hours

**Mitigation**: Comprehensive self-help documentation; FAQ

---

### CON-SUP-002: No On-Premise Deployment
**Constraint**: System cannot be deployed on-premise.

**Details**:
- Cloud deployment only
- No option for institutional servers
- Dependency on cloud provider

**Rationale**: Scalability and maintenance

**Impact**: Ongoing cloud costs; reliance on cloud provider

**Mitigation**: Cost budgeting; SLA with cloud provider

---

## Security Constraints

### CON-SEC-001: No Custom Security Implementation
**Constraint**: Must use industry-standard security practices; no custom crypto.

**Details**:
- bcrypt for password hashing (cost ≥ 10)
- TLS 1.2+ for encryption
- JWT for tokens
- No custom authentication mechanisms

**Rationale**: Security best practices

**Impact**: No flexibility in security implementation

**Mitigation**: Well-tested and industry-standard libraries used

---

### CON-SEC-002: No End-to-End Encryption (MVP)
**Constraint**: Data encrypted in transit and at rest; no E2E encryption between users.

**Details**:
- TLS for transit encryption
- Database encryption for rest
- No message encryption between students/teachers

**Rationale**: MVP scope; use case doesn't require E2E

**Impact**: Admins can read all messages/communications

**Mitigation**: Access control limits who can view data

---

## Compliance Constraints

### CON-COM-001: Educational Data Standards
**Constraint**: System must comply with educational data standards.

**Details**:
- Student data: Name, Roll Number, Contact (standard fields)
- Grade data: Component marks, overall grade (standard format)
- Certificate: Uniform template

**Rationale**: Institutional and regulatory standards

**Impact**: Limited flexibility in data structure

**Mitigation**: Extensible schema for future enhancements

---

## Constraints Summary Table

| Category | Constraint Count | Severity |
|----------|-----------------|----------|
| Technical | 5 | High |
| Business | 4 | High |
| Organizational | 3 | High |
| Data | 3 | Medium |
| Operational | 3 | Medium |
| Support | 2 | Medium |
| Security | 2 | High |
| Compliance | 1 | Medium |
| **TOTAL** | **23** | — |

## Impact Analysis

| Constraint | Risk | Mitigation |
|-----------|------|-----------|
| Frozen Architecture | Medium | Comprehensive design review |
| Budget Limitation | High | Careful planning; prioritization |
| Timeline | High | Scope control; resource allocation |
| Single Admin Team | Medium | Automation; documentation |
| Technology Stack | Low | Team expertise available |
| No Integrations (MVP) | Medium | Design for future extensibility |

## Related Documents

- [Assumptions](08-Assumptions.md)
- [Non-Functional Requirements](06-Non-Functional-Requirements.md)
- [Scope](03-Scope.md)
