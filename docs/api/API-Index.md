# API Documentation - Index

## Overview

Complete API reference for the KISS Vocational Training Management Portal REST API.

## API Versions

- **Current**: v1
- **Base URL**: `https://api.example.com/v1`
- **Protocol**: HTTPS (TLS 1.2+)
- **Authentication**: JWT Bearer Token

## Authentication

All endpoints except login require JWT token in `Authorization` header:

```
Authorization: Bearer <jwt_token>
```

See [Authentication API](Authentication.md) for login and token management.

## API Endpoints by Role

### Public Endpoints (No Authentication)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/auth/login` | User login |
| POST | `/auth/refresh` | Refresh token |
| POST | `/auth/password-reset-request` | Request password reset |
| POST | `/auth/password-reset` | Reset password |
| GET | `/certificates/verify/{certNumber}` | Verify certificate |

### Student Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/student/dashboard` | Student dashboard data |
| GET | `/student/profile` | Get own profile |
| PATCH | `/student/profile` | Update own profile |
| GET | `/student/available-courses` | List courses for enrollment |
| POST | `/student/enroll` | Submit enrollment |
| GET | `/student/enrollments` | Get all enrollments |
| DELETE | `/student/enrollments/{enrollmentId}` | Drop course |
| GET | `/student/progress/{courseId}` | View progress |
| GET | `/student/attendance/{courseId}` | View attendance |
| GET | `/student/grades/{courseId}` | View grades |
| POST | `/student/grievance` | Submit grievance |
| GET | `/student/grievances` | List grievances |
| GET | `/student/feedback/{courseId}` | Get feedback form |
| POST | `/student/feedback/{courseId}` | Submit feedback |
| GET | `/student/certificates` | List certificates |
| GET | `/student/certificates/{id}/download` | Download certificate |
| POST | `/student/allocation/{allocationId}/switch` | Request batch switch |

**Total Student Endpoints**: 17

### Teacher Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/teacher/dashboard` | Teacher dashboard |
| GET | `/teacher/courses` | List assigned courses |
| GET | `/teacher/batch/{batchId}/students` | List batch students |
| POST | `/teacher/attendance` | Record attendance |
| GET | `/teacher/attendance/{batchId}` | View attendance |
| PATCH | `/teacher/attendance/{attendanceId}` | Edit attendance |
| GET | `/teacher/gradebook/{batchId}` | View gradebook |
| POST | `/teacher/grades` | Enter grades |
| PATCH | `/teacher/grades/{gradeId}` | Edit grades |
| PATCH | `/teacher/gradebook/{batchId}/publish` | Publish grades |
| POST | `/teacher/course/{batchId}/complete` | Mark course complete |
| GET | `/teacher/feedback/{courseId}` | View feedback |

**Total Teacher Endpoints**: 12

### Admin Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/admin/dashboard` | Admin dashboard |
| POST | `/admin/semesters` | Create semester |
| GET | `/admin/semesters` | List semesters |
| PATCH | `/admin/semesters/{semesterId}` | Update semester |
| POST | `/admin/course-offerings` | Create course offering |
| GET | `/admin/course-offerings` | List offerings |
| POST | `/admin/grading-schemes` | Define grading scheme |
| GET | `/admin/grading-schemes` | List schemes |
| POST | `/admin/enrollment-windows` | Create enrollment window |
| GET | `/admin/enrollment-windows` | List windows |
| PATCH | `/admin/enrollment-windows/{windowId}/activate` | Activate window |
| PATCH | `/admin/enrollment-windows/{windowId}/deactivate` | Deactivate window |
| POST | `/admin/allocation/run` | Run batch allocation |
| GET | `/admin/allocation/{allocationId}` | Get allocation report |
| GET | `/admin/switch-requests` | List switch requests |
| PATCH | `/admin/switch-requests/{requestId}` | Approve/reject switch |
| GET | `/admin/reports` | Generate reports |
| GET | `/admin/users` | List all users |
| POST | `/admin/users` | Create user |
| PATCH | `/admin/users/{userId}` | Update user |
| PATCH | `/admin/users/{userId}/role` | Change user role |
| DELETE | `/admin/users/{userId}` | Deactivate user |

**Total Admin Endpoints**: 22

---

## API Documentation by Module

### Authentication Module

Documentation: [Authentication.md](Authentication.md)

**Endpoints**:
- `POST /auth/login` - User authentication
- `POST /auth/refresh` - Token refresh
- `GET /auth/me` - Current user info
- `POST /auth/logout` - User logout
- `POST /auth/password-reset-request` - Reset request
- `POST /auth/password-reset` - Password reset

---

### Student Module

Documentation: [Student.md](Student.md)

**Submodules**:
- Dashboard
- Profile
- Enrollment
- Progress Tracking
- Attendance
- Grades
- Grievances
- Feedback
- Certificates

---

### Teacher Module

Documentation: [Teacher.md](Teacher.md)

**Submodules**:
- Dashboard
- Attendance Management
- Gradebook
- Course Completion

---

### Admin Module

Documentation: [Admin.md](Admin.md)

**Submodules**:
- Dashboard
- Semester Management
- Course Configuration
- Enrollment Management
- Allocation
- Batch Switching
- User Management
- Reports

---

## HTTP Status Codes

| Status | Meaning | When Used |
|--------|---------|-----------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | No auth or invalid token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict (e.g., duplicate) |
| 422 | Unprocessable Entity | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

---

## Error Response Format

All error responses follow this format:

```json
{
  "status": 400,
  "error": "INVALID_CREDENTIALS",
  "message": "Email or password incorrect",
  "timestamp": "2026-01-15T10:30:00Z",
  "path": "/api/auth/login",
  "details": null
}
```

### Fields

- `status`: HTTP status code
- `error`: Error code (machine-readable)
- `message`: Error message (human-readable)
- `timestamp`: ISO 8601 timestamp
- `path`: Request path
- `details`: Additional error details (if applicable)

---

## Success Response Format

All success responses follow this format:

```json
{
  "status": 200,
  "data": { ... },
  "timestamp": "2026-01-15T10:30:00Z",
  "message": "Request successful"
}
```

### Fields

- `status`: HTTP status code
- `data`: Response data (object or array)
- `timestamp`: ISO 8601 timestamp
- `message`: Optional success message

---

## Rate Limiting

### Headers

Response includes rate limit info in headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1674916200
```

### Limits

- **Login**: 5 attempts per 15 minutes per IP
- **General**: 100 requests per minute per user
- **Search**: 50 requests per minute per user

When limit exceeded: `429 Too Many Requests`

---

## Pagination

Endpoints returning lists support pagination:

**Query Parameters**:
```
?page=1&size=10&sort=name,asc
```

**Response**:
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "size": 10,
    "totalElements": 87,
    "totalPages": 9,
    "hasNext": true,
    "hasPrevious": false
  }
}
```

---

## Filtering & Sorting

### Query Parameters

```
GET /api/admin/users?role=STUDENT&active=true&sort=name,asc
```

### Common Filters

- `role`: STUDENT, TEACHER, ADMIN
- `active`: true, false
- `status`: ACTIVE, COMPLETED, DROPPED
- `date_from`: ISO 8601 date
- `date_to`: ISO 8601 date

### Sorting

```
?sort=name,asc
?sort=createdAt,desc
?sort=name,asc&sort=email,asc
```

---

## Common Request/Response Examples

### Create Resource Example

**Request**:
```bash
curl -X POST https://api.example.com/v1/admin/course-offerings \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "courseId": 1,
    "semester": "Spring 2026",
    "startDate": "2026-02-01",
    "endDate": "2026-05-31",
    "maxCapacity": 100
  }'
```

**Response** (201 Created):
```json
{
  "status": 201,
  "data": {
    "courseOfferingId": 5,
    "courseId": 1,
    "courseName": "Java Programming",
    "semester": "Spring 2026",
    "startDate": "2026-02-01",
    "endDate": "2026-05-31",
    "maxCapacity": 100,
    "currentEnrollment": 0,
    "createdAt": "2026-01-15T10:30:00Z"
  },
  "timestamp": "2026-01-15T10:30:00Z"
}
```

---

### Update Resource Example

**Request**:
```bash
curl -X PATCH https://api.example.com/v1/student/profile \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+91-9876543210",
    "address": "123 Main St, Pune"
  }'
```

**Response** (200 OK):
```json
{
  "status": 200,
  "data": {
    "userId": 42,
    "name": "Rahul Sharma",
    "email": "student@example.com",
    "rollNumber": "042",
    "phoneNumber": "+91-9876543210",
    "address": "123 Main St, Pune",
    "updatedAt": "2026-01-15T10:30:00Z"
  },
  "timestamp": "2026-01-15T10:30:00Z"
}
```

---

## Testing the API

### Using cURL

```bash
# Login
curl -X POST https://api.example.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "student@example.com", "password": "Password123!"}'

# Get student profile (with token)
curl -X GET https://api.example.com/v1/student/profile \
  -H "Authorization: Bearer <access_token>"
```

### Using Postman

1. Create new collection
2. Set base URL to `https://api.example.com/v1`
3. Create `/auth/login` request to get token
4. Set token in collection variables
5. Use `{{token}}` in Authorization header for other requests

### Using REST Client Extensions

**VS Code REST Client**:
```
### Login
POST https://api.example.com/v1/auth/login
Content-Type: application/json

{
  "email": "student@example.com",
  "password": "Password123!"
}

### Get Profile
GET https://api.example.com/v1/student/profile
Authorization: Bearer {{access_token}}
```

---

## API Documentation Links

- [Authentication API](Authentication.md) - User login, tokens
- [Student API](Student.md) - All 17 student endpoints
- [Teacher API](Teacher.md) - All 12 teacher endpoints
- [Admin API](Admin.md) - All 22 admin endpoints
- [Error Codes](Error-Codes.md) - Complete error reference

---

## Versioning

Current API version: **v1**

Future changes will increment to v2, v3, etc.

**Version in URL**: `https://api.example.com/v1/...`

---

## Related Documentation

- [System Architecture](../SDD/01-System-Architecture.md#43-application-layer-spring-boot)
- [RBAC Design](../SDD/03-RBAC-Design.md)
- [Database Design](../SDD/02-Database-Design.md)
- [Functional Requirements](../SRS/05-Functional-Requirements.md)
