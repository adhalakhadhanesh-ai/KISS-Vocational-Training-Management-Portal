# Authentication API

## Base URL

```
https://api.example.com/v1
```

## Overview

This API handles user authentication and token management for the KISS Vocational Training Management Portal.

## Scope Note

The password-reset endpoints below (`/auth/password-reset-request`, `/auth/password-reset`) are retained in this MVP API surface because they are fully specified here — request/response shapes, error codes, and validation rules are all documented. Per the documentation audit, any endpoint that is only a stub or is outside [Scope](../SRS/03-Scope.md#what-the-system-will-do) is removed; these are neither.

## Endpoints

### POST /auth/login

**Purpose**: Authenticate user and obtain JWT token

**Authentication**: None (public endpoint)

**Request**:
```json
{
  "email": "student@example.com",
  "password": "SecurePassword123!"
}
```

**Response** (200 OK):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 86400,
  "user": {
    "userId": 42,
    "email": "student@example.com",
    "role": "STUDENT",
    "name": "Rahul Sharma"
  }
}
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 400 | INVALID_CREDENTIALS | Email or password incorrect |
| 400 | MISSING_FIELD | Required field missing |
| 403 | ACCOUNT_LOCKED | Account locked after 5 failed attempts |
| 429 | TOO_MANY_REQUESTS | Rate limit exceeded |
| 500 | INTERNAL_ERROR | Server error |

**Validation**:
- Email must be valid format
- Password must not be empty
- Max 5 attempts per 15 minutes

---

### POST /auth/refresh

**Purpose**: Refresh expired JWT token

**Authentication**: Bearer token (optional for refresh)

**Request**:
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response** (200 OK):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 86400
}
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 401 | INVALID_TOKEN | Token invalid or expired |
| 400 | MISSING_FIELD | Refresh token missing |

---

### GET /auth/me

**Purpose**: Get current authenticated user

**Authentication**: Bearer token (required)

**Response** (200 OK):
```json
{
  "userId": 42,
  "email": "student@example.com",
  "role": "STUDENT",
  "name": "Rahul Sharma",
  "createdAt": "2026-01-10T09:30:00Z",
  "lastLogin": "2026-01-15T10:15:00Z"
}
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 401 | UNAUTHORIZED | No token provided or invalid |
| 401 | TOKEN_EXPIRED | Token has expired |

---

### POST /auth/logout

**Purpose**: Invalidate current session

**Authentication**: Bearer token (required)

**Request**:
```json
{}
```

**Response** (200 OK):
```json
{
  "message": "Logout successful"
}
```

---

### POST /auth/password-reset-request

**Purpose**: Request password reset email

**Authentication**: None (public endpoint)

**Request**:
```json
{
  "email": "student@example.com"
}
```

**Response** (200 OK):
```json
{
  "message": "Password reset email sent",
  "expiresIn": 3600
}
```

**Note**: Returns 200 regardless of email exists (security best practice)

---

### POST /auth/password-reset

**Purpose**: Reset password with token

**Authentication**: None (public endpoint)

**Request**:
```json
{
  "resetToken": "abc123xyz789",
  "newPassword": "NewPassword123!"
}
```

**Response** (200 OK):
```json
{
  "message": "Password reset successful"
}
```

**Error Responses**:

| Status | Error | Description |
|--------|-------|-------------|
| 400 | INVALID_TOKEN | Reset token invalid or expired |
| 400 | WEAK_PASSWORD | Password doesn't meet requirements |

**Validation**:
- Password ≥ 8 characters
- Must contain uppercase, lowercase, number, special char
- Cannot reuse last 5 passwords

---

## JWT Token Structure

**Header**:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload**:
```json
{
  "sub": "42",
  "email": "student@example.com",
  "role": "STUDENT",
  "iat": 1674829800,
  "exp": 1674916200,
  "iss": "kiss-portal",
  "aud": "kiss-portal-app"
}
```

**Properties**:
- `sub`: User ID
- `email`: User email
- `role`: User role (STUDENT, TEACHER, ADMIN)
- `iat`: Issued at (Unix timestamp)
- `exp`: Expiration (Unix timestamp, 24 hours)
- `iss`: Issuer
- `aud`: Audience

---

## Authentication Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Client submits credentials (email, password)             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Server validates credentials                             │
│    ├─ Check email exists                                    │
│    ├─ Verify password (bcrypt)                              │
│    └─ Check account not locked                              │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
      VALID                   INVALID
         │                       │
         ▼                       ▼
   ┌──────────────┐      ┌──────────────┐
   │ Fetch user   │      │ Increment    │
   │ role & data  │      │ failed count │
   └──────────────┘      └──────────────┘
         │
         ▼
   ┌──────────────┐
   │ Generate JWT │
   │ token        │
   └──────────────┘
         │
         ▼
   ┌──────────────┐
   │ Return token │
   │ to client    │
   └──────────────┘
         │
         ▼
   ┌──────────────────────────────┐
   │ Client stores in localStorage│
   └──────────────────────────────┘
```

---

## Using Tokens in Requests

### With cURL

```bash
curl -X GET https://api.example.com/v1/student/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### With JavaScript (Fetch)

```javascript
const token = localStorage.getItem('accessToken');

fetch('https://api.example.com/v1/student/profile', {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error(error));
```

### With JavaScript (Axios)

```javascript
import axios from 'axios';

const token = localStorage.getItem('accessToken');
const api = axios.create({
  baseURL: 'https://api.example.com/v1',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

api.get('/student/profile')
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

---

## Error Handling

### Error Response Format

```json
{
  "status": 400,
  "error": "INVALID_CREDENTIALS",
  "message": "Email or password incorrect",
  "timestamp": "2026-01-15T10:30:00Z",
  "path": "/api/auth/login"
}
```

### Common Errors

| Error Code | HTTP Status | Meaning |
|------------|------------|---------|
| INVALID_CREDENTIALS | 400 | Email/password incorrect |
| ACCOUNT_LOCKED | 403 | Account locked |
| UNAUTHORIZED | 401 | No token or invalid token |
| TOKEN_EXPIRED | 401 | Token has expired |
| WEAK_PASSWORD | 400 | Password doesn't meet requirements |
| INVALID_TOKEN | 400 | Reset/refresh token invalid |
| TOO_MANY_REQUESTS | 429 | Rate limit exceeded |

---

## Security Best Practices

### Client Side

1. **Store tokens securely**:
   - Use localStorage (not sessionStorage for persistence)
   - Clear tokens on logout
   - Never expose token in URL

2. **Handle token expiration**:
   - Catch 401 errors
   - Attempt to refresh token
   - Redirect to login if refresh fails

3. **Send tokens correctly**:
   - Use `Authorization: Bearer <token>` header
   - Never send token in request body or URL

### Server Side (Already Implemented)

✓ Passwords hashed with bcrypt (cost ≥ 10)
✓ Rate limiting on login (5 attempts/15 min)
✓ Account lockout after failed attempts
✓ JWT signature verification
✓ Token expiration checks
✓ HTTPS/TLS encryption

---

## Example: Complete Login Flow

### Frontend (React)

```javascript
import { useState } from 'react';
import axios from 'axios';

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleLogin = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('https://api.example.com/v1/auth/login', {
        email,
        password
      });

      // Store token
      localStorage.setItem('accessToken', response.data.accessToken);
      localStorage.setItem('user', JSON.stringify(response.data.user));

      // Redirect based on role
      const { role } = response.data.user;
      if (role === 'STUDENT') window.location.href = '/student/dashboard';
      else if (role === 'TEACHER') window.location.href = '/teacher/dashboard';
      else window.location.href = '/admin/dashboard';

    } catch (err) {
      setError(err.response?.data?.message || 'Login failed');
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      {error && <p className="error">{error}</p>}
      <button type="submit">Login</button>
    </form>
  );
}

export default LoginForm;
```

---

## Rate Limiting

**Login Endpoint** (`POST /auth/login`):
- Max 5 attempts per 15 minutes per IP
- Max 10 attempts per 1 hour per email
- Returns 429 Too Many Requests when exceeded

**Other Endpoints**:
- Max 100 requests per minute per user
- Returns 429 Too Many Requests when exceeded

---

## Related Documentation

- [RBAC Design](../SDD/03-RBAC-Design.md)
- [System Architecture](../SDD/01-System-Architecture.md#authentication--authorization-flow)
- [Non-Functional Requirements](../SRS/06-Non-Functional-Requirements.md#2-security-requirements)
