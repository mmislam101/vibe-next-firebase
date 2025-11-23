# API Specification Documentation

**API Layer:** Firebase Cloud Functions (TypeScript)  

---

## Implementation Status

### Authentication & User Profile

**Implemented Endpoints:**
- ✅ `GET /api/v1/me` - Get authenticated user profile
- ✅ `PUT /api/v1/me` - Update user profile

**Authentication System:**
- ✅ Firebase Google OAuth authentication
- ✅ Bearer token validation
- ✅ Automatic user profile creation on first sign-in
- ✅ Role-based access control foundation

---

## 1. Overview

This document defines the REST API endpoints implemented as Firebase Cloud Functions. All endpoints require Firebase Authentication unless explicitly marked as public.

### 1.1 Base URL

**[Implements: SRD-REQ-FUNC-001, SRD-REQ-DEV-003, SRS-REQ-API-003, SRS-REQ-API-011]**

- **Development:** `http://localhost:5001/{project-id}/us-central1`
- **Production:** `https://us-central1-{project-id}.cloudfunctions.net`

### 1.2 Authentication

**[Implements: SRD-REQ-FUNC-003, SRS-SRD-REQ-SEC-001, SRS-REQ-API-001, SRS-REQ-API-002]**

All authenticated endpoints require a Firebase ID token from Google Sign-In in the Authorization header:

```
Authorization: Bearer <firebase-id-token>
```

**Note:** Only Google OAuth authentication is supported. The token must be from a Google sign-in provider.

### 1.3 Response Format

**Success Response:**
```json
{
  "data": { ... },
  "message": "Optional success message"
}
```

**Error Response:**
```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "suggestion": "Optional suggestion for resolution"
}
```

### 1.4 HTTP Status Codes

**[Implements: SRS-SRD-REQ-PERF-003]**

- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Rate Limit Exceeded
- `500` - Internal Server Error

### 1.5 Layout and Navigation Support

The API supports the application's layout structure through the following endpoints:

**User Profile (for Top Navigation Bar):**
- `GET /api/v1/me` - Get authenticated user profile (name, email, photo from Google account)
- `PUT /api/v1/me` - Update user profile information

All layout components use these endpoints to provide user information in the navigation bar.

---

## 2. User Profile Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-007]**

### 2.1 Get User Profile

**[Implements: SRS-REQ-UI-001, SRS-SRD-REQ-SEC-001, SRD-REQ-FUNC-002, SRS-REQ-API-007, SRS-REQ-API-008]**

**Endpoint:** `GET /api/v1/me`

**Authentication:** Required

**Implementation Status:** ✅ Completed (November 20, 2025)

**Response:** `200 OK`
```json
{
  "data": {
    "id": "user-uid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Smith",
    "photoUrl": "https://lh3.googleusercontent.com/...",
    "accountStatus": "active",
    "createdAt": "2025-11-20T00:00:00Z",
    "lastLoginAt": "2025-11-23T10:30:00Z"
  }
}
```

---

### 2.2 Update User Profile

**[Implements: SRS-REQ-UI-001, SRS-SRD-REQ-SEC-001, SRD-REQ-FUNC-002, SRS-REQ-API-007]**

**Endpoint:** `PUT /api/v1/me`

**Authentication:** Required

**Implementation Status:** ✅ Completed (November 20, 2025)

**Request Body:**
```json
{
  "firstName": "John",
  "lastName": "Smith"
}
```

**Response:** `200 OK`
```json
{
  "message": "Profile updated",
  "data": {
    "firstName": "John",
    "lastName": "Smith",
    "updatedAt": "2025-11-23T10:30:00Z"
  }
}
```

**Validation:**
- `firstName` is optional, max 100 characters
- `lastName` is optional, max 100 characters
- Email and Google profile photo cannot be changed (managed by Google account)

---

## 3. Error Handling

### 3.1 Error Codes

- `VALIDATION_ERROR` - Request validation failed
- `AUTHENTICATION_ERROR` - Authentication failed or token expired
- `AUTHORIZATION_ERROR` - User doesn't have permission
- `NOT_FOUND` - Resource not found
- `INTERNAL_ERROR` - Internal server error

### 3.2 Common Error Responses

**Authentication Error:**
```json
{
  "error": "Authentication failed",
  "code": "AUTHENTICATION_ERROR",
  "suggestion": "Please sign in again"
}
```

**Validation Error:**
```json
{
  "error": "Invalid request data",
  "code": "VALIDATION_ERROR",
  "details": {
    "firstName": "Maximum length is 100 characters"
  }
}
```

**Not Found Error:**
```json
{
  "error": "User profile not found",
  "code": "NOT_FOUND"
}
```

---

**End of Document**

