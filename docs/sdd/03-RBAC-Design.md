# RBAC Design

## 1. Overview

This document defines the Role-Based Access Control (RBAC) system for the KISS Vocational Training Management Portal, including roles, permissions, and enforcement mechanisms.

## 2. Roles Definition

### 2.1 Student Role

**Purpose**: Course enrollment and progress tracking

**Capabilities**:
- View own profile
- Enroll in courses
- View course progress and grades
- Submit grievances
- Submit feedback
- Download certificates

**Access Level**: Limited to own data

**Module Access**:
- Dashboard (read-only)
- Profile (read-only)
- Enrollment (read/write)
- Progress (read-only)
- Grievance Center (write)
- Feedback (write)
- Certificates (read)

---

### 2.2 Teacher Role

**Purpose**: Course instruction and assessment

**Capabilities**:
- View assigned courses
- Mark attendance
- Enter and publish grades
- Mark course complete
- View student allocations
- View student feedback

**Access Level**: Limited to assigned courses and students

**Module Access**:
- Dashboard (read-only)
- Assigned Courses (read-only)
- Attendance (read/write)
- Gradebook (read/write)
- Student List (read-only)

---

### 2.3 Admin Role

**Purpose**: System configuration and operations

**Capabilities**:
- Create and configure semesters
- Create course offerings
- Define grading schemes
- Manage enrollment windows
- Run batch allocation
- Approve batch switching
- Generate reports
- Manage user accounts

**Access Level**: Full system access

**Module Access**:
- All modules (read/write)
- Dashboard/Command Center
- User Management
- Configuration
- Reports

---

## 3. Permission Matrix

### 3.1 Module-Level Permissions

| Module | Student | Teacher | Admin |
|--------|---------|---------|-------|
| Dashboard | View | View | View |
| Profile | View Own | View Own | View All |
| Enrollment | Create | None | Manage |
| Progress | View Own | View Assigned | View All |
| Attendance | None | Record | Manage |
| Gradebook | View Own | Record & Publish | Manage |
| Courses | View | View Assigned | Manage |
| Batches | View Assigned | View Assigned | Manage |
| Allocations | View Own | None | Manage |
| Grievances | Create Own | None | View All |
| Feedback | Submit | None | View All |
| Certificates | Download | None | Manage |
| Reports | None | None | Generate |
| System Config | None | None | Manage |

### 3.2 Action-Level Permissions

| Action | Student | Teacher | Admin |
|--------|---------|---------|-------|
| Create User | ✗ | ✗ | ✓ |
| View User List | ✗ | ✗ | ✓ |
| Edit Profile | Self | Self | ✓ All |
| Create Course Offering | ✗ | ✗ | ✓ |
| Enroll in Course | ✓ | ✗ | ✗ |
| Take Attendance | ✗ | ✓ Assigned | ✓ |
| Mark Grades | ✗ | ✓ Assigned | ✓ |
| Publish Grades | ✗ | ✓ Assigned | ✓ |
| Complete Course | ✗ | ✓ Assigned | ✓ |
| Run Allocation | ✗ | ✗ | ✓ |
| Approve Batch Switch | ✗ | ✗ | ✓ |
| Generate Reports | ✗ | ✗ | ✓ |

---

## 4. Access Control Enforcement

### 4.1 Authentication Layer

**Process**:
```
User Login
    ↓
POST /api/auth/login
    ↓
AuthenticationService.validateCredentials(email, password)
    ↓
Password matches (bcrypt validation)
    ↓
Fetch User record
    ↓
Read role from Users table
    ↓
Generate JWT token with role claim
    ↓
Return token to client
    ↓
Client stores in localStorage
```

**JWT Token Structure**:
```json
{
  "sub": "user_id",
  "email": "student@example.com",
  "role": "STUDENT",
  "iat": 1234567890,
  "exp": 1234654290
}
```

### 4.2 Authorization Layer

**Process**:
```
HTTP Request
    ↓
Extract JWT token from Authorization header
    ↓
JWTAuthenticationFilter validates signature
    ↓
Verify token not expired
    ↓
Extract role from token claims
    ↓
AuthorizationFilter checks role for endpoint
    ↓
Role matches required role(s)
    ↓
Spring Security context populated
    ↓
Request proceeds to controller
    ↓
Controller performs resource-level checks
```

### 4.3 Implementation - Spring Security

**Configuration**:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        http
            .authorizeHttpRequests((authz) -> authz
                // Public endpoints
                .requestMatchers("/api/auth/**").permitAll()
                
                // Student endpoints
                .requestMatchers("/api/student/**").hasRole("STUDENT")
                
                // Teacher endpoints
                .requestMatchers("/api/teacher/**").hasRole("TEACHER")
                
                // Admin endpoints
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                
                // Other requests require authentication
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter(), 
                UsernamePasswordAuthenticationFilter.class)
            .csrf().disable();
        return http.build();
    }
}
```

---

## 5. Resource-Level Access Control

### 5.1 Student Module

**Endpoint**: `GET /api/student/progress/{studentId}`

```java
@GetMapping("/progress/{studentId}")
@PreAuthorize("@studentService.isOwnerOrAdmin(#studentId, authentication)")
public ResponseEntity<?> getProgress(@PathVariable Integer studentId) {
    // Only student or admin can view this
}
```

**Logic**:
- If authenticated user is the student themselves: Allow
- If authenticated user is admin: Allow
- Otherwise: Return 403 Forbidden

### 5.2 Teacher Module

**Endpoint**: `POST /api/teacher/attendance`

```java
@PostMapping("/attendance")
@PreAuthorize("@teacherService.isAssignedTeacher(#batchId, authentication)")
public ResponseEntity<?> recordAttendance(
    @RequestParam Integer batchId,
    @RequestBody AttendanceRequest request) {
    // Only assigned teacher can record attendance
}
```

**Logic**:
- If authenticated user is assigned to this batch: Allow
- Otherwise: Return 403 Forbidden

### 5.3 Admin Module

**Endpoint**: `POST /api/admin/allocation/run`

```java
@PostMapping("/allocation/run")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<?> runAllocation(@RequestBody AllocationRequest request) {
    // Only admin can run allocation
}
```

---

## 6. Role Transitions

### 6.1 User Role Change

**Process**:
```
Admin updates user role
    ↓
Admin navigates to User Management
    ↓
Selects user and new role
    ↓
PATCH /api/admin/users/{userId}/role
    ↓
AdminService.updateUserRole(userId, newRole)
    ↓
Update Users table role column
    ↓
Previous JWT tokens still valid until expiration
    ↓
New login with new role generates new token
```

**Note**: Role change takes effect on next login. Existing sessions continue with old role until token expires (24 hours).

---

## 7. Data Visibility Rules

### 7.1 Student Data Visibility

| Data | Student Self | Other Student | Teacher | Admin |
|------|--------------|---------------|---------|-------|
| Own Profile | View | ✗ | ✗ | View |
| Own Grades | View | ✗ | ✗ | View |
| Own Attendance | View | ✗ | ✗ | View |
| Assigned Batch | View | View | View | View |
| Course Progress | View Own | ✗ | ✗ | View All |

### 7.2 Teacher Data Visibility

| Data | Student | Teacher Self | Other Teacher | Admin |
|------|---------|--------------|---------------|-------|
| Profile | View Basic | View | View Basic | View |
| Assigned Courses | ✗ | View | ✗ | View |
| Student Attendance | ✗ | View Own Batch | ✗ | View All |
| Student Grades | View Own | View Own Students | ✗ | View All |

### 7.3 Admin Data Visibility

| Data | Student | Teacher | Admin |
|------|---------|---------|-------|
| All Profiles | ✗ | ✗ | View |
| System Configuration | ✗ | ✗ | Manage |
| Reports | ✗ | ✗ | Generate |
| Audit Logs | ✗ | ✗ | View |

---

## 8. Audit Logging

### 8.1 Logged Actions

Every action that modifies data is logged:

```
Timestamp: 2026-01-15 10:30:45 IST
User ID: 42 (admin@example.com)
Role: ADMIN
Action: UPDATE_USER_ROLE
Resource: Student (student_id: 101)
Change: role = 'STUDENT'
IP Address: 192.168.1.100
Status: SUCCESS
```

### 8.2 Log Retention

- Logs retained for minimum 1 year
- Logs protected from unauthorized access
- Separate read-only audit table
- Regular backup of audit logs

---

## 9. Security Considerations

### 9.1 Principle of Least Privilege
- Each role has minimum necessary permissions
- No role has more permissions than needed
- Students cannot access teacher/admin functions

### 9.2 Separation of Duties
- One person cannot unilaterally approve and execute allocation
- Admin approval required for batch switches
- Multiple admins possible for critical operations

### 9.3 Session Management
- JWT tokens expire after 24 hours
- Logout clears token on client side
- No session state on server (stateless)

### 9.4 Credential Protection
- Passwords hashed with bcrypt (cost ≥ 10)
- No password storage as plain text
- Password reset via email verification
- Account lockout after 5 failed attempts

---

## 10. Custom Annotations

**Purpose**: Simplify authorization logic in controllers

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface StudentOwnerOrAdmin {
}

@Aspect
@Component
public class StudentOwnerOrAdminAspect {
    @Before("@annotation(StudentOwnerOrAdmin)")
    public void checkStudentOwner(JoinPoint jp) {
        // Verify user is student owner or admin
    }
}
```

---

## 11. Implementation Checklist

- [ ] JWT token generation with role claim
- [ ] JWTAuthenticationFilter implementation
- [ ] AuthorizationFilter for role checking
- [ ] @PreAuthorize annotations on all protected endpoints
- [ ] Resource-level access control checks
- [ ] Audit logging for all sensitive operations
- [ ] Login rate limiting (5 attempts in 15 min)
- [ ] Account lockout mechanism
- [ ] Password reset via email
- [ ] Test cases for each role and permission

---

## 12. Testing Strategy

### 12.1 Unit Tests
- AuthenticationService tests
- Authorization check tests
- Role determination tests

### 12.2 Integration Tests
- Login with different roles
- Access control enforcement
- Endpoint permission tests

### 12.3 Security Tests
- Unauthorized access attempts
- Token expiration
- Role escalation attempts

---

## 13. Related Documents

- [System Architecture](01-System-Architecture.md)
- [Database Design](02-Database-Design.md)
- [Functional Requirements](../SRS/05-Functional-Requirements.md)
- [Security Requirements](../SRS/06-Non-Functional-Requirements.md#2-security-requirements)
