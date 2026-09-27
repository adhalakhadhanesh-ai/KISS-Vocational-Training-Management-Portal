# Test Plan

## 1. Introduction

This document outlines the comprehensive testing strategy for the KISS Vocational Training Management Portal.

## 2. Testing Scope

### In Scope

✓ Functional testing of all modules
✓ Security testing (authentication, authorization, injection)
✓ Performance testing (response time, throughput)
✓ Database testing (integrity, consistency)
✓ API testing (endpoints, error handling)
✓ Integration testing (component interactions)
✓ User acceptance testing (UAT)

### Out of Scope

✗ Infrastructure testing (cloud provider responsibility)
✗ UI/UX testing (design team responsibility)
✗ Load balancer testing (ops team responsibility)

---

## 3. Testing Strategy

### 3.1 Test Levels

```
Unit Testing
    ↓
Component Testing
    ↓
Integration Testing
    ↓
System Testing
    ↓
UAT
    ↓
Production
```

### 3.2 Test Types

| Type | Coverage | Effort | Duration |
|------|----------|--------|----------|
| Unit | Code logic | 40% | 2 weeks |
| Integration | Component interaction | 30% | 2 weeks |
| System | End-to-end workflows | 20% | 2 weeks |
| Performance | Response time, load | 5% | 1 week |
| Security | Vulnerabilities, access | 5% | 1 week |

---

## 4. Unit Testing

### 4.1 Coverage Target

- **Goal**: ≥ 70% code coverage
- **Tools**: JUnit 5, Mockito
- **Framework**: Spring Boot Test

### 4.2 Test Categories

**Service Layer**:
- EnrollmentService (enrollment validation)
- AllocationService (random shuffle algorithm)
- GradingService (grade calculation)
- CertificateService (PDF generation)

**Validation**:
- Input validation rules
- Business rule enforcement
- Error handling

**Example Test**:
```java
@Test
void testEnrollmentValidation_DuplicateEnrollment() {
    // Given: Student already enrolled
    Enrollment existing = new Enrollment();
    existing.setStudentId(1);
    existing.setCourseOfferingId(5);
    enrollmentRepo.save(existing);
    
    // When: Attempt duplicate enrollment
    EnrollmentException exception = assertThrows(
        EnrollmentException.class,
        () -> enrollmentService.submitEnrollment(1, 5)
    );
    
    // Then
    assertEquals("Already enrolled", exception.getMessage());
}
```

---

## 5. Integration Testing

### 5.1 Test Scenarios

**Enrollment Workflow**:
1. Admin creates semester
2. Admin creates course offering
3. Admin opens enrollment window
4. Student enrolls in course
5. System creates enrollment record
6. Email notification sent

**Allocation Workflow**:
1. Enrollment window closes
2. Admin runs allocation
3. Students assigned to batches
4. Allocations verified
5. Emails sent

**Grading Workflow**:
1. Teacher accesses gradebook
2. Teacher enters marks
3. Teacher publishes grades
4. Students view grades
5. Grades verified

---

## 6. System Testing

### 6.1 End-to-End Scenarios

**Complete Student Journey** (1-2 hours):
1. Login as student
2. View available courses
3. Enroll in course
4. Check enrollment status
5. View progress (before allocation)
6. Receive batch allocation
7. View assigned batch
8. View attendance
9. View published grades
10. Download certificate

**Complete Teacher Journey** (30 minutes):
1. Login as teacher
2. View assigned courses
3. Record attendance for batch
4. Enter marks for students
5. Publish grades
6. Mark course complete
7. Verify certificate generation

---

## 7. Performance Testing

### 7.1 Performance Benchmarks

| Operation | Target | Method |
|-----------|--------|--------|
| Page load | < 2 sec | Browser DevTools |
| API response | < 1 sec | Load testing tool |
| Allocation (1000 students) | < 5 min | Batch job monitoring |
| 200 concurrent users | < 3 sec response | Apache JMeter |

### 7.2 Load Testing Setup

**Tool**: Apache JMeter

**Scenario**:
- 200 concurrent users
- Each user: login → enroll → view progress
- Duration: 10 minutes
- Ramp-up: 2 minutes

**Expected Results**:
- Avg response time < 1 sec
- 95th percentile < 2 sec
- Error rate < 1%

---

## 8. Security Testing

### 8.1 Authentication Testing

**Tests**:
- Invalid credentials rejected
- Account lockout after 5 failures
- Token expiration enforced
- Weak passwords rejected
- Password reset token expires

**Example Test**:
```java
@Test
void testLoginSecurity_AccountLockout() {
    // Try 5 failed logins
    for (int i = 0; i < 5; i++) {
        authService.login("user@example.com", "wrongPassword");
    }
    
    // 6th attempt should fail with locked message
    AuthException ex = assertThrows(AuthException.class,
        () -> authService.login("user@example.com", "correctPassword"));
    
    assertEquals("ACCOUNT_LOCKED", ex.getErrorCode());
}
```

### 8.2 Authorization Testing

**Tests**:
- Student cannot access teacher endpoints
- Teacher cannot access admin endpoints
- Admin can access all endpoints
- Resource-level access control enforced

### 8.3 Injection Testing

**Tests**:
- SQL injection prevention (parameterized queries)
- XSS prevention (HTML escaping)
- CSRF token validation

---

## 9. Test Execution Plan

### 9.1 Timeline

| Phase | Duration | Start | End |
|-------|----------|-------|-----|
| Unit Testing | 2 weeks | Week 1 | Week 2 |
| Integration | 2 weeks | Week 3 | Week 4 |
| System | 2 weeks | Week 5 | Week 6 |
| Performance | 1 week | Week 7 | Week 7 |
| Security | 1 week | Week 8 | Week 8 |
| UAT | 2 weeks | Week 9 | Week 10 |

### 9.2 Test Environment

**Development**:
- Local machine
- H2 in-memory database
- Mock external services

**Staging**:
- Cloud environment (AWS/Azure)
- PostgreSQL test database
- Real email server (sandbox)

**Production-like**:
- Same infrastructure as production
- Test data only
- No real user impact

---

## 10. Test Cases

### 10.1 Sample Functional Test Cases

**TC-001: Student Login**
```
Precondition: Student account exists
Steps:
1. Navigate to login page
2. Enter valid email
3. Enter valid password
4. Click Login
Expected: Dashboard displays, token stored
Priority: Critical
```

**TC-002: Course Enrollment**
```
Precondition: Enrollment window active, student not enrolled
Steps:
1. Login as student
2. Go to Enrollment
3. Select course
4. Confirm enrollment
Expected: Enrollment confirmed, email sent
Priority: Critical
```

**TC-003: Batch Allocation**
```
Precondition: Enrollments exist, window closed
Steps:
1. Login as admin
2. Go to Allocation
3. Run allocation
Expected: All students allocated, no errors
Priority: Critical
```

**TC-004: Grade Publication**
```
Precondition: Grades entered, in Draft state
Steps:
1. Login as teacher
2. Go to Gradebook
3. Review marks
4. Click Publish
Expected: Grades locked, students notified
Priority: Critical
```

**TC-005: Certificate Download**
```
Precondition: Course completed, certificate generated
Steps:
1. Login as student
2. Go to Certificates
3. Click Download
Expected: PDF downloads successfully
Priority: High
```

---

## 11. Test Data Management

### 11.1 Test Data Strategy

**Approach**: Seeded test database

**Data Set**:
- 1000 students (multiple batches)
- 10 courses
- 5 semesters
- 50 teachers
- 10 admins

**Reset**:
- Each test cycle: reset database to seed state
- Use SQL scripts for restoration
- Automation recommended

---

## 12. Defect Management

### 12.1 Severity Levels

| Severity | Description | Example |
|----------|-------------|---------|
| Critical | System inoperable | Login broken, allocation fails |
| High | Major feature broken | Can't view grades |
| Medium | Workaround exists | Email not sent (displayed in UI) |
| Low | Minor issue | Formatting incorrect |

### 12.2 Defect Tracking

- **Tool**: Jira/GitHub Issues
- **Fields**: ID, Title, Severity, Status, Assignee
- **Workflow**: New → Assigned → Fixed → Verified → Closed

---

## 13. Test Metrics

### 13.1 Coverage Metrics

- **Code Coverage**: Target ≥ 70%
- **Test Case Coverage**: ≥ 90% of requirements
- **Functional Coverage**: 100% of user stories

### 13.2 Quality Metrics

- **Defect Density**: ≤ 3 defects per 1000 LOC
- **Test Pass Rate**: ≥ 95%
- **Test Execution**: 100% within timeline

---

## 14. Sign-Off

### 14.1 Approval Gates

| Gate | Owner | Criteria |
|------|-------|----------|
| Unit Test | Dev Lead | Coverage ≥ 70% |
| Integration | QA Lead | All tests pass |
| System | QA Lead | No critical defects |
| Performance | Ops Team | Response time ✓ |
| Security | Security Lead | Vulnerabilities ≤ 0 |
| UAT | Program Director | Stakeholder approval |

---

## 15. Related Documents

- [Test Cases](Functional-Test-Cases.md)
- [Security Testing](Security-Test-Cases.md)
- [Performance Testing](Performance-Test-Cases.md)
- [Functional Requirements](../SRS/05-Functional-Requirements.md)
