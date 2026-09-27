# Stakeholders

## Overview

This document identifies all stakeholders involved in the KISS Vocational Training Management Portal project, their roles, interests, and impact levels.

## Stakeholder Categories

### Primary Stakeholders

#### 1. Students
**Role**: End-users of the student portal

**Interests**:
- Easy enrollment in desired courses
- Fair and transparent batch allocation
- Real-time progress tracking
- Timely access to certificates
- Mechanism to lodge grievances

**Impact Level**: Critical

**Involvement**:
- Participate in UAT (User Acceptance Testing)
- Provide feedback on usability
- Report issues encountered

#### 2. Teachers
**Role**: Content deliverers and assessors

**Interests**:
- Efficient attendance and grading workflows
- Ease of publishing grades
- Clear visibility into student performance
- Timely course completion marking

**Impact Level**: Critical

**Involvement**:
- Validate gradebook workflows during design
- Participate in UAT
- Provide feedback on interface design

#### 3. Administrative Staff (KISS Program Management)
**Role**: Operators and operators of the system

**Interests**:
- Centralized command center for monitoring
- Automated allocation and scheduling
- Easy report generation
- Audit trails for compliance
- System reliability and uptime

**Impact Level**: Critical

**Involvement**:
- Define business rules and policies
- Configure system settings
- Validate admin workflows
- Generate and review reports

### Secondary Stakeholders

#### 4. IT Operations Team
**Role**: System maintenance and infrastructure support

**Interests**:
- System stability and performance
- Secure database management
- Regular backups and disaster recovery
- Monitoring and alerting

**Impact Level**: High

**Involvement**:
- Advise on technical architecture
- Manage deployment and hosting
- Handle system monitoring

#### 5. Project Manager
**Role**: Overall project coordination

**Interests**:
- On-time delivery
- Within-budget execution
- Quality assurance
- Risk management

**Impact Level**: High

**Involvement**:
- Coordinate between teams
- Track milestones
- Manage stakeholder communication

#### 6. Quality Assurance & Testing Team
**Role**: Ensure system quality and reliability

**Interests**:
- Clear requirements for test planning
- Comprehensive testing infrastructure
- Bug-free release
- Performance metrics

**Impact Level**: High

**Involvement**:
- Define test plans
- Execute functional and security testing
- Performance testing and optimization

#### 7. Development Team
**Role**: Design and implementation

**Interests**:
- Clear and stable requirements
- Realistic timelines
- Adequate resources
- Technical debt management

**Impact Level**: High

**Involvement**:
- Clarify requirement ambiguities
- Suggest technical solutions
- Estimate effort and complexity

#### 8. Security & Compliance Officer
**Role**: Ensure data protection and regulatory compliance

**Interests**:
- Data encryption and secure transmission
- Role-based access enforcement
- Audit logging and accountability
- Compliance with data privacy regulations

**Impact Level**: High

**Involvement**:
- Define security requirements
- Review system architecture
- Audit implementation

### Tertiary Stakeholders

#### 9. KIIT University Administration
**Role**: Institutional oversight

**Interests**:
- System alignment with university policies
- Institutional reputation
- Long-term sustainability
- Support for institutional goals

**Impact Level**: Medium

**Involvement**:
- Provide policy guidance
- Approve major system decisions
- Monitor progress

#### 10. Future Administrators (System Maintainers)
**Role**: Long-term system support

**Interests**:
- Clear documentation
- Maintainable codebase
- Well-structured database
- Knowledge transfer

**Impact Level**: Medium

**Involvement**:
- Benefit from comprehensive documentation
- Receive training on system operation

## Stakeholder Engagement Plan

### Communication Matrix

| Stakeholder | Frequency | Medium | Owner |
|-------------|-----------|--------|-------|
| Students | Bi-weekly | Email, Dashboard Messages | Admin Team |
| Teachers | Bi-weekly | Email, Training Sessions | Admin Team |
| Admin Staff | Weekly | Meetings, Reports | Project Manager |
| IT Operations | Weekly | Technical Meetings | Tech Lead |
| Development Team | Daily | Standup, Scrum | Dev Lead |
| QA Team | Daily | Bug Reports, Status Reports | QA Lead |
| Project Manager | Daily | Status Updates, Reviews | PM |
| Security Officer | Bi-weekly | Security Reviews | Dev Lead |

### Stakeholder Influence vs. Impact Matrix

| Stakeholder | Influence | Impact | Strategy |
|-------------|-----------|--------|----------|
| Students | Medium | Critical | Inform & Consult |
| Teachers | High | Critical | Collaborate |
| Admin Staff | High | Critical | Collaborate |
| IT Operations | High | High | Collaborate |
| Development Team | High | High | Collaborate |
| QA Team | High | High | Collaborate |
| Project Manager | High | High | Manage |
| Security Officer | Medium | High | Consult |
| University Admin | Low | Medium | Inform |
| Future Maintainers | Low | Medium | Inform |

## Stakeholder Concerns & Mitigation

| Stakeholder | Concern | Mitigation |
|-------------|---------|-----------|
| Students | System complexity, learning curve | Simple interface, training documentation |
| Teachers | Grading workflow errors | Validation rules, draft/publish mechanism |
| Admin | System reliability | 99.5% uptime SLA, comprehensive monitoring |
| IT Ops | Security vulnerabilities | Regular security audits, patch management |
| Dev Team | Unclear requirements | Clear SRS, regular refinement meetings |
| QA | Insufficient test time | Early test plan creation, continuous testing |
| Security | Unauthorized access | JWT-based auth, role-based access control |

## Sign-Off & Approval

### Stakeholder Approval

| Stakeholder | Approval | Date | Signature |
|-------------|----------|------|-----------|
| Program Director | [ ] | [ ] | [ ] |
| Head of IT | [ ] | [ ] | [ ] |
| Student Representative | [ ] | [ ] | [ ] |
| Teacher Representative | [ ] | [ ] | [ ] |
| Project Manager | [ ] | [ ] | [ ] |

## Related Documents

- [Problem Statement](02-Problem-Statement.md)
- [Scope](03-Scope.md)
- [Functional Requirements](05-Functional-Requirements.md)
