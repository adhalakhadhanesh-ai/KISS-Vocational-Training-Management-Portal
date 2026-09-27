# Non-Functional Requirements

## Overview

This document defines non-functional requirements including performance, security, scalability, reliability, and other quality attributes.

## 1. Performance Requirements

### NFR-PER-001: Response Time
**Requirement**: Page load time shall not exceed 2 seconds under normal load.

**Details**:
- Dashboard pages: ≤ 2 seconds
- List pages (courses, students): ≤ 3 seconds
- Form submission: ≤ 1 second
- Report generation: ≤ 5 seconds
- Measured on high-speed internet (10+ Mbps)

### NFR-PER-002: Throughput
**Requirement**: System shall handle at least 200 concurrent users.

**Details**:
- Peak load: 200 simultaneous users
- Allocation run: Complete within 5 minutes for 1000 students
- Certificate generation: 100 certificates per minute
- Measured under normal database load

### NFR-PER-003: Database Query Performance
**Requirement**: Database queries shall complete within acceptable time.

**Details**:
- Simple queries: ≤ 100 ms
- Aggregate queries: ≤ 500 ms
- Complex joins: ≤ 1 second
- Report queries: ≤ 10 seconds
- Indexed properly for common queries

### NFR-PER-004: Batch Processing Performance
**Requirement**: Batch jobs shall complete within defined windows.

**Details**:
- Allocation run: ≤ 5 minutes
- Certificate generation: ≤ 10 minutes per 1000 students
- Report generation: ≤ 5 minutes
- Scheduled during off-peak hours (10 PM - 6 AM)

## 2. Security Requirements

### NFR-SEC-001: Authentication Security
**Requirement**: User authentication shall be secure and robust.

**Details**:
- Passwords hashed using bcrypt (cost factor ≥ 10)
- Minimum password length: 8 characters
- Complexity requirement: uppercase, lowercase, number, special char
- Login attempts limited: Max 5 failed attempts in 15 minutes
- Account locked for 15 minutes after 5 failures

### NFR-SEC-002: Authorization & Access Control
**Requirement**: System shall enforce role-based access control (RBAC).

**Details**:
- Three roles: Student, Teacher, Admin
- Role checked on every protected API call
- Unauthorized access returns 403 Forbidden
- Access control list (ACL) enforced
- Principle of least privilege applied

### NFR-SEC-003: Data Encryption
**Requirement**: Sensitive data shall be encrypted in transit and at rest.

**Details**:
- HTTPS/TLS for all network communication
- Minimum TLS version: 1.2
- Passwords encrypted with bcrypt before storage
- Database encryption at rest (if Supabase supports)
- Sensitive fields: Password, Email, Phone (not displayed in logs)

### NFR-SEC-004: Session Management
**Requirement**: Sessions shall be securely managed.

**Details**:
- JWT tokens with 24-hour expiration
- Token refresh mechanism available
- Logout clears token and session
- Token includes user ID, role, issued time
- Token signature verified on every request

### NFR-SEC-005: Input Validation
**Requirement**: All user inputs shall be validated.

**Details**:
- Server-side validation for all inputs
- SQL injection prevention (parameterized queries)
- XSS prevention (HTML escaping)
- CSRF token protection for state-changing requests
- File upload validation (type, size)

### NFR-SEC-006: Audit Logging
**Requirement**: All sensitive operations shall be logged.

**Details**:
- Log: Login/logout, role changes, data modifications, admin actions
- Include: Timestamp, user ID, action, affected records, IP address
- Logs retained for minimum 1 year
- Logs protected from unauthorized access
- Regular audit log review

### NFR-SEC-007: API Security
**Requirement**: APIs shall be secure against common attacks.

**Details**:
- Rate limiting: Max 100 requests per minute per user
- API key validation (if needed)
- CORS policy configured to allow only trusted domains
- Headers: X-Frame-Options, X-Content-Type-Options, Content-Security-Policy
- API versioning supported

## 3. Reliability Requirements

### NFR-REL-001: Availability
**Requirement**: System shall maintain 99.5% uptime.

**Details**:
- Uptime calculated monthly
- Maximum 3.6 hours downtime per month
- Excludes scheduled maintenance windows
- Measured from system startup

### NFR-REL-002: Data Backup & Recovery
**Requirement**: Data shall be backed up and recoverable.

**Details**:
- Daily automated backups
- Backup retention: 30 days
- Recovery Time Objective (RTO): 4 hours
- Recovery Point Objective (RPO): 1 hour
- Backup tested quarterly

### NFR-REL-003: Error Handling
**Requirement**: System shall handle errors gracefully.

**Details**:
- All exceptions caught and logged
- User-friendly error messages
- No sensitive information in error messages
- Fallback options where applicable
- Graceful degradation under partial failure

### NFR-REL-004: Data Consistency
**Requirement**: System shall maintain data consistency.

**Details**:
- Database transactions for multi-step operations
- ACID compliance for critical operations
- Referential integrity constraints
- No orphaned records in allocation/gradebook
- Concurrency control for simultaneous edits

## 4. Scalability Requirements

### NFR-SCA-001: User Scalability
**Requirement**: System shall scale to support growing user base.

**Details**:
- Current: 1000 users
- Year 1: Up to 2000 users
- Year 2: Up to 5000 users
- Response time maintained as user base grows

### NFR-SCA-002: Data Scalability
**Requirement**: System shall handle growing data volumes.

**Details**:
- 5 years of historical data retained
- Estimated 500 MB to 2 GB per year
- Query performance maintained with indexing
- Archival strategy for old data (optional)

### NFR-SCA-003: Horizontal Scalability
**Requirement**: Backend shall support horizontal scaling.

**Details**:
- Stateless application servers
- Load balancing across multiple instances
- No session stickiness required
- Database connection pooling
- Cloud infrastructure support (Supabase/AWS)

## 5. Maintainability Requirements

### NFR-MAN-001: Code Quality
**Requirement**: Codebase shall maintain high quality standards.

**Details**:
- Coding standards documented and enforced
- Code review process mandatory
- Technical debt tracked and managed
- Test coverage ≥ 70%
- Static analysis tools (SonarQube, ESLint)

### NFR-MAN-002: Documentation
**Requirement**: System shall be comprehensively documented.

**Details**:
- API documentation (Swagger/OpenAPI)
- Database schema documentation
- Architecture Decision Records (ADRs)
- Deployment and operations guide
- Troubleshooting guide

### NFR-MAN-003: Logging & Monitoring
**Requirement**: System operations shall be observable.

**Details**:
- Centralized logging
- Application performance monitoring (APM)
- Error tracking and alerting
- Real-time dashboard for ops team
- Alert for critical failures

## 6. Usability Requirements

### NFR-USA-001: User Interface
**Requirement**: UI shall be intuitive and user-friendly.

**Details**:
- Consistent design language
- Responsive design (desktop, tablet, mobile)
- Accessibility compliance (WCAG 2.1 Level AA)
- Clear navigation and information hierarchy
- Help documentation and tooltips

### NFR-USA-002: User Onboarding
**Requirement**: New users shall be able to onboard quickly.

**Details**:
- Tutorial for first-time users
- Role-specific onboarding paths
- Video guides for complex features
- Chat support for immediate help
- Time to productive use: ≤ 5 minutes

### NFR-USA-003: Language & Localization
**Requirement**: System shall support multiple languages (future).

**Details**:
- Current: English only
- Future: Hindi, regional languages
- Date/time formats localized
- Right-to-left language support (if needed)

## 7. Compliance Requirements

### NFR-COM-001: Data Privacy
**Requirement**: System shall comply with data privacy regulations.

**Details**:
- GDPR compliance (if applicable)
- India Data Privacy standards
- User consent for data collection
- Data deletion on request
- Privacy policy displayed

### NFR-COM-002: Regulatory Compliance
**Requirement**: System shall meet applicable regulations.

**Details**:
- Educational institution compliance
- NIST cybersecurity framework (optional)
- Industry best practices for data security

## 8. Compatibility Requirements

### NFR-COM-001: Browser Compatibility
**Requirement**: System shall work on modern browsers.

**Details**:
- Chrome (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Edge (latest 2 versions)
- Mobile browsers (Chrome Android, Safari iOS)

### NFR-COM-002: Database Compatibility
**Requirement**: System shall work with specified database.

**Details**:
- PostgreSQL 12+
- Supabase as database provider
- No vendor-specific SQL extensions
- Portable schema and queries

## 9. Installation & Deployment

### NFR-DEP-001: Deployment Automation
**Requirement**: Deployment shall be automated and reliable.

**Details**:
- CI/CD pipeline (GitHub Actions, Jenkins)
- Automated testing before deployment
- Blue-green or canary deployment strategy
- Rollback capability
- Zero-downtime deployments (if possible)

### NFR-DEP-002: Environment Configuration
**Requirement**: Configuration shall be environment-independent.

**Details**:
- Environment variables for configuration
- Secrets management (not hardcoded)
- Configuration per environment (dev, staging, production)
- Easy configuration for new deployments

## 10. Support & Maintenance

### NFR-SUP-001: Support Levels
**Requirement**: System shall have defined support levels.

**Details**:
- Level 1: User support (help desk)
- Level 2: Technical support (admin/ops team)
- Level 3: Development team (complex issues)
- Response time: Critical ≤ 1 hour, High ≤ 4 hours, Medium ≤ 1 day

### NFR-SUP-002: Updates & Patches
**Requirement**: System shall receive regular updates.

**Details**:
- Security patches: Within 7 days
- Bug fixes: Within 2 weeks
- Feature updates: Monthly or quarterly
- Maintenance windows: Weekends, 10 PM - 6 AM
- Backup before every deployment

## Non-Functional Requirements Summary

| Category | Requirement Count | Priority |
|----------|------------------|----------|
| Performance | 4 | Critical |
| Security | 7 | Critical |
| Reliability | 4 | Critical |
| Scalability | 3 | High |
| Maintainability | 3 | High |
| Usability | 3 | High |
| Compliance | 2 | High |
| Compatibility | 2 | High |
| Deployment | 2 | High |
| Support | 2 | Medium |
| **TOTAL** | **32** | — |

## Related Documents

- [Functional Requirements](05-Functional-Requirements.md)
- [Constraints](09-Constraints.md)
- [System Architecture](../SDD/01-System-Architecture.md)
