# System Architecture

## 1. Overview

This document describes the overall system architecture of the KISS Vocational Training Management Portal, including high-level design decisions, system components, and data flow.

## 2. Architectural Principles

### 2.1 Separation of Concerns
- **Frontend**: React SPA (Single Page Application)
- **Backend**: Spring Boot REST API
- **Database**: PostgreSQL (managed by Supabase)
- Clean separation between layers

### 2.2 Stateless Design
- Backend is stateless
- All state stored in database
- JWT tokens for session management
- Enables horizontal scaling

### 2.3 Role-Based Access Control
- Three distinct roles: Student, Teacher, Admin
- Access control at API layer
- Role determined from JWT token

### 2.4 Security First
- TLS encryption in transit
- Password hashing with bcrypt
- SQL injection prevention
- CSRF protection

## 3. System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interface Layer                     │
│  ┌────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  Student   │  │   Teacher    │  │    Admin     │        │
│  │  Dashboard │  │  Workspace   │  │  Dashboard   │        │
│  └────────────┘  └──────────────┘  └──────────────┘        │
│         React Single Page Application (SPA)                 │
└─────────────────────────────────────────────────────────────┘
                            │
                    HTTP/HTTPS (TLS 1.2+)
                            │
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway / Load Balancer              │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │      Spring Boot REST API (Multiple Instances)       │  │
│  │                                                       │  │
│  │  ┌────────────┐  ┌──────────┐  ┌─────────────────┐  │  │
│  │  │   Auth     │  │  Student │  │  Teacher Ctrl   │  │  │
│  │  │ Controller │  │Controller│  │                 │  │  │
│  │  └────────────┘  └──────────┘  └─────────────────┘  │  │
│  │                                                       │  │
│  │  ┌────────────┐  ┌──────────┐  ┌─────────────────┐  │  │
│  │  │   Admin    │  │  Course  │  │  Allocation     │  │  │
│  │  │ Controller │  │Controller│  │ Controller      │  │  │
│  │  └────────────┘  └──────────┘  └─────────────────┘  │  │
│  │                                                       │  │
│  │ ┌─────────────────────────────────────────────────┐ │  │
│  │ │  Service Layer                                  │ │  │
│  │ │  (Business Logic, Validation, Authorization)   │ │  │
│  │ └─────────────────────────────────────────────────┘ │  │
│  │                                                       │  │
│  │ ┌─────────────────────────────────────────────────┐ │  │
│  │ │  Repository Layer (Spring Data JPA)             │ │  │
│  │ │  (Database Access, ORM)                         │ │  │
│  │ └─────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│ ┌──────────────────────────────────────────────────────┐   │
│ │  Cross-Cutting Concerns                             │   │
│ │  ├─ JWT Authentication Filter                       │   │
│ │  ├─ Authorization Filter (RBAC)                     │   │
│ │  ├─ Exception Handling                              │   │
│ │  ├─ Logging & Audit Trail                           │   │
│ │  └─ Request/Response Serialization                  │   │
│ └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                   PostgreSQL JDBC Driver
                            │
┌─────────────────────────────────────────────────────────────┐
│               Data Layer (PostgreSQL/Supabase)               │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐ │
│  │   User &     │  │   Enrollment │  │   Grades &        │ │
│  │   Profile    │  │   & Batches  │  │   Assessment      │ │
│  │   Tables     │  │   Tables     │  │   Tables          │ │
│  └──────────────┘  └──────────────┘  └───────────────────┘ │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐ │
│  │  Courses &   │  │  Attendance  │  │   Certificates    │ │
│  │  Offerings   │  │   & Feedback │  │   & Grievances    │ │
│  │  Tables      │  │   Tables     │  │   Tables          │ │
│  └──────────────┘  └──────────────┘  └───────────────────┘ │
│                                                              │
│  Indexes for: Users (email), Enrollments (student_id),     │
│  Allocations (batch_id), Grades (student_id), etc.        │
└─────────────────────────────────────────────────────────────┘
```

## 4. Layered Architecture

### 4.1 Presentation Layer (React)
**Responsibility**: User interface, form validation, client-side routing

**Components**:
- Student Dashboard
- Teacher Workspace
- Admin Dashboard
- Authentication UI
- Shared components (navbar, forms, tables)

**Technology**: React, React Router, Axios/Fetch API

---

### 4.2 API Gateway / Load Balancer
**Responsibility**: Route requests, load balancing, SSL/TLS termination

**Features**:
- HTTPS enforcement
- Request routing based on URL path
- Load balancing across backend instances
- Rate limiting

**Technology**: Nginx or cloud provider's load balancer

---

### 4.3 Application Layer (Spring Boot)

#### 4.3.1 Controller Layer
**Responsibility**: HTTP request handling, parameter validation, response formatting

**Controllers**:
- `AuthenticationController`: Login, token refresh
- `StudentController`: Enrollment, progress, certificates
- `TeacherController`: Attendance, grading, course completion
- `AdminController`: Semester config, allocation, reports
- `CourseController`: Course and offering management

**Pattern**: RESTful API design, standardized request/response

---

#### 4.3.2 Service Layer
**Responsibility**: Business logic, validation, orchestration

**Services**:
- `AuthenticationService`: Credential validation, token generation
- `EnrollmentService`: Enrollment validation, capacity checking
- `AllocationService`: Random shuffle, batch filling algorithm
- `GradingService`: Grade calculation, validation
- `CertificateService`: Certificate generation, PDF creation
- `ReportService`: Report generation and export

**Pattern**: Dependency injection, transaction management

---

#### 4.3.3 Repository Layer
**Responsibility**: Data access, ORM mapping

**Technology**: Spring Data JPA with Hibernate

**Features**:
- CRUD operations
- Custom queries using @Query annotations
- Pagination and sorting
- Transaction management

---

### 4.4 Data Layer (PostgreSQL)

**Database**: PostgreSQL 12+ hosted on Supabase

**Key Tables**: Users, Students, Teachers, Courses, CourseOfferings, Enrollments, Batches, Allocations, Grades, Attendance, Certificates, Feedback, Grievances

**Features**:
- Referential integrity constraints
- Indexes on frequently queried columns
- Partitioning for large tables (optional)
- Regular backups

---

## 5. Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | React | UI framework |
| Routing | React Router | Client-side routing |
| HTTP Client | Axios | API communication |
| Backend | Spring Boot | Application framework |
| Security | Spring Security | Authentication & authorization |
| Database | PostgreSQL | Data storage |
| ORM | Hibernate/JPA | Object-relational mapping |
| Build Tool | Maven | Project build and dependency management |
| Deployment | Docker (optional) | Containerization |
| Hosting | Supabase/AWS | Cloud infrastructure |

---

## 6. Authentication & Authorization Flow

```
User Login
    ↓
POST /api/auth/login (email, password)
    ↓
AuthenticationService validates credentials
    ↓
JWT token generated (user_id, role, email, exp)
    ↓
Token returned to frontend
    ↓
Frontend stores token in localStorage
    ↓
Subsequent requests include:
Authorization: Bearer <JWT_token>
    ↓
JWTAuthenticationFilter validates token
    ↓
AuthorizationFilter checks role for endpoint
    ↓
Request processed by controller/service
    ↓
Response returned
```

---

## 7. Data Flow - Enrollment Example

```
User (Student) → React UI (Enrollment Form)
    ↓
POST /api/student/enroll (courseOffering_id)
    ↓
Spring Controller → Service Layer
    ↓
Validation:
├─ Enrollment window active?
├─ Student already enrolled?
├─ Capacity available?
└─ Student eligible?
    ↓
All checks pass → Create Enrollment Record
    ↓
Save to Database (PostgreSQL)
    ↓
Email Service: Send confirmation
    ↓
Response: 201 Created (enrollment_id)
    ↓
Frontend updates UI, shows confirmation
    ↓
Student sees enrolled course in dashboard
```

---

## 8. Batch Processing

### 8.1 Allocation Run Process

```
Admin initiates allocation
    ↓
Fetch all enrollments for course offering
    ↓
Randomize enrollment list (shuffle)
    ↓
For each batch (by capacity):
  ├─ Fill batch with shuffled students
  ├─ Create Allocation records
  └─ Move to next batch
    ↓
Generate allocation report
    ↓
Send email notifications to students
    ↓
Update student dashboard
```

### 8.2 Certificate Generation

```
Teacher marks course complete
    ↓
Service fetches all enrolled students
    ↓
For each student:
  ├─ Generate PDF with merged fields
  ├─ Assign unique Certificate ID
  ├─ Save to database
  └─ Send email notification
    ↓
Student sees certificate in dashboard
```

---

## 9. Security Architecture

### 9.1 Authentication
- JWT tokens with 24-hour expiration
- Password hashing: bcrypt (cost factor ≥ 10)
- Login rate limiting: Max 5 attempts in 15 minutes

### 9.2 Authorization
- Role-based access control (Student, Teacher, Admin)
- Access control checked on every protected endpoint
- Unauthorized access returns 403 Forbidden

### 9.3 Data Protection
- HTTPS/TLS for all network communication
- Database encryption at rest (Supabase feature)
- Sensitive fields not logged (passwords, emails)

### 9.4 Input Validation
- Server-side validation for all inputs
- Parameterized queries prevent SQL injection
- HTML escaping prevents XSS

### 9.5 Audit Logging
- All sensitive operations logged
- Includes: timestamp, user, action, affected records
- Logs retained for minimum 1 year

---

## 10. Scalability Considerations

### 10.1 Horizontal Scaling
- Stateless backend (no session affinity needed)
- Load balancer distributes requests
- Multiple Spring Boot instances

### 10.2 Database Scaling
- Connection pooling (HikariCP)
- Indexed queries for performance
- Prepared statements reduce parsing
- Optional read replicas for reporting

### 10.3 Performance Optimization
- Lazy loading in ORM to prevent N+1 queries
- Caching for frequently accessed data (optional)
- Batch processing for large operations

---

## 11. Deployment Architecture

```
┌─────────────────────────────────────────┐
│      GitHub Repository (Version Control)│
└────────────────────┬────────────────────┘
                     │
           ┌─────────▼────────┐
           │  CI/CD Pipeline  │
           │  (GitHub Actions)│
           │  ├─ Build       │
           │  ├─ Test        │
           │  └─ Deploy      │
           └─────────┬────────┘
                     │
        ┌────────────┴──────────────┐
        │                           │
┌───────▼──────────┐    ┌──────────▼────────┐
│  Staging Env     │    │  Production Env   │
│  ├─ Frontend SPA │    │  ├─ Frontend SPA  │
│  ├─ Backend API  │    │  ├─ Backend API   │
│  └─ Database     │    │  └─ Database      │
└──────────────────┘    └───────────────────┘
```

---

## 12. Error Handling Strategy

### 12.1 Application Errors
```
┌─ Bad Request (400)
├─ Unauthorized (401)
├─ Forbidden (403)
├─ Not Found (404)
├─ Conflict (409)
├─ Validation Error (422)
└─ Internal Server Error (500)
```

### 12.2 Error Response Format
```json
{
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "Enrollment window is not active",
  "timestamp": "2026-01-15T10:30:00Z",
  "path": "/api/student/enroll"
}
```

### 12.3 Logging
- All exceptions logged with stack trace
- Error ID returned for tracking
- User-friendly error messages without technical details

---

## 13. Related Documents

- [Database Design](02-Database-Design.md)
- [RBAC Design](03-RBAC-Design.md)
- [Enrollment Design](04-Enrollment-Design.md)
- [Batch Allocation](05-Batch-Allocation.md)
- [Functional Requirements](../SRS/05-Functional-Requirements.md)

---

## 14. Architecture Decision Records

See [Architecture Decision Records](../Architecture/) for detailed justification of architectural choices.
