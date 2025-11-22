# API Specification Documentation
## Patient Alert Notification System - Firebase Cloud Functions

**Version:** 1.0  
**Date:** November 9, 2025  
**Last Updated:** November 20, 2025  
**API Layer:** Firebase Cloud Functions (TypeScript)  
**Document Status:** Draft

---

## Implementation Status

### Phase 1: Authentication & User Profile (✅ Completed - November 20, 2025)

**Implemented Endpoints:**
- ✅ `GET /api/v1/me` - Get authenticated user profile [Implements: SRS-REQ-UI-001, SRS-SRD-REQ-SEC-001]
- ✅ `PUT /api/v1/me` - Update user profile (documented, to be implemented)

**Authentication System:**
- ✅ Firebase Google OAuth authentication [Implements: SRS-REQ-UI-001, SRS-SRD-REQ-SEC-001]
- ✅ Bearer token validation [Implements: SRS-SRD-REQ-SEC-001]
- ✅ Automatic user profile creation on first sign-in [Implements: SRS-REQ-UI-001]
- ✅ Role-based access control foundation [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005]

**Pending Endpoints (Future Phases):**
- ⏳ Patient Management (Phase 2)
- ⏳ Observer Management (Phase 2)
- ⏳ Alert Management (Phase 3)
- ⏳ Incident Management (Phase 4)
- ⏳ Escalation Policies (Phase 4)
- ⏳ Notifications (Phase 5)

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
- `PUT /api/v1/me/preferences` - Update notification preferences (accessed from dropdown menu)

**Navigation Data:**
- `GET /api/v1/patients` - Get patient list (for sidebar navigation badge counts)
- `GET /api/v1/incidents` - Get active incidents (for sidebar navigation badge counts)
- `GET /api/v1/me/patients` - Get observer's assigned patients (for role-based navigation)

All navigation elements and layout components use these endpoints to provide real-time data and role-specific menu items.

---

## 2. Patient Management Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-003, SRS-REQ-PATIENT-001]**

### 2.1 Create Patient

**Endpoint:** `POST /api/v1/patients`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "name": "Sarah Johnson",
  "description": "8-year-old, Type 1 Diabetes, requires insulin monitoring",
  "photoUrl": "https://example.com/photo.jpg",
  "status": "active"
}
```

**Response:** `201 Created`
```json
{
  "data": {
    "id": "uuid",
    "caretakerId": "caretaker-uid",
    "name": "Sarah Johnson",
    "description": "8-year-old, Type 1 Diabetes...",
    "photoUrl": "https://example.com/photo.jpg",
    "status": "active",
    "createdAt": "2025-11-10T00:00:00Z",
    "updatedAt": "2025-11-10T00:00:00Z"
  },
  "message": "Patient created successfully"
}
```

**Validation:**
- `name` is required, max 200 characters
- `description` is optional, max 1000 characters
- `photoUrl` must be valid URL
- `status` must be one of: `active`, `inactive`, `archived`

---

### 2.2 List Patients

**Endpoint:** `GET /api/v1/patients`

**Authentication:** Required (Caretaker)

**Query Parameters:**
- `status` (optional): Filter by status (`active`, `inactive`, `archived`)
- `limit` (optional): Number of results (default: 50, max: 100)
- `offset` (optional): Pagination offset (default: 0)

**Response:** `200 OK`
```json
{
  "data": {
    "patients": [
      {
        "id": "uuid",
        "name": "Sarah Johnson",
        "status": "active",
        "activeAlertsCount": 5,
        "activeIncidentsCount": 2,
        "observersCount": 3,
        "createdAt": "2025-11-10T00:00:00Z"
      }
    ],
    "total": 10,
    "limit": 50,
    "offset": 0
  }
}
```

---

### 2.3 Get Patient Details

**Endpoint:** `GET /api/v1/patients/{patientId}`

**Authentication:** Required (Caretaker or assigned Observer)

**Response:** `200 OK`
```json
{
  "data": {
    "id": "uuid",
    "caretakerId": "caretaker-uid",
    "name": "Sarah Johnson",
    "description": "8-year-old, Type 1 Diabetes...",
    "photoUrl": "https://example.com/photo.jpg",
    "status": "active",
    "activeAlertsCount": 5,
    "activeIncidentsCount": 2,
    "observersCount": 3,
    "createdAt": "2025-11-10T00:00:00Z",
    "updatedAt": "2025-11-10T00:00:00Z"
  }
}
```

**Error:** `404 Not Found` if patient doesn't exist or user doesn't have access

---

### 2.4 Update Patient

**Endpoint:** `PUT /api/v1/patients/{patientId}`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "name": "Sarah Johnson",
  "description": "Updated description",
  "photoUrl": "https://example.com/new-photo.jpg",
  "status": "active"
}
```

**Response:** `200 OK`
```json
{
  "data": {
    "id": "uuid",
    "name": "Sarah Johnson",
    "description": "Updated description",
    "status": "active",
    "updatedAt": "2025-11-10T01:00:00Z"
  },
  "message": "Patient updated successfully"
}
```

---

### 2.5 Archive Patient

**Endpoint:** `DELETE /api/v1/patients/{patientId}`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Patient archived successfully"
}
```

**Note:** This is a soft delete - sets status to `archived` and cancels all active alerts.

---

### 2.6 Get Patient Incidents

**Endpoint:** `GET /api/v1/patients/{patientId}/incidents`

**Authentication:** Required (Caretaker or assigned Observer)

**Query Parameters:**
- `status` (optional): Filter by status
- `limit` (optional): Number of results (default: 50)
- `offset` (optional): Pagination offset

**Response:** `200 OK`
```json
{
  "data": {
    "incidents": [
      {
        "id": "uuid",
        "incidentNumber": 1234,
        "alertId": "alert-uuid",
        "status": "triggered",
        "severity": "high",
        "triggeredAt": "2025-11-10T08:00:00Z"
      }
    ],
    "total": 5
  }
}
```

---

## 3. Observer Management Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-004, SRS-REQ-OBS-001, SRS-REQ-OBS-007]**

### 3.1 Invite Observer

**Endpoint:** `POST /api/v1/patients/{patientId}/observers/invite`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "email": "observer@example.com",
  "phoneNumber": "+12345678901",
  "firstName": "John",
  "lastName": "Smith",
  "role": "primary",
  "escalationLevel": 1,
  "customMessage": "Hi John, I'd like you to help monitor Sarah's medication schedule."
}
```

**Response:** `201 Created`
```json
{
  "data": {
    "invitationId": "uuid",
    "status": "pending",
    "expiresAt": "2025-11-17T00:00:00Z"
  },
  "message": "Invitation sent successfully"
}
```

**Validation:**
- `email` must be valid email format
- `phoneNumber` must be E.164 format
- `role` must be one of: `primary`, `secondary`, `backup`, `always_notify`
- `escalationLevel` must be 1-5

---

### 3.2 List Patient Observers

**Endpoint:** `GET /api/v1/patients/{patientId}/observers`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "data": {
    "observers": [
      {
        "assignmentId": "uuid",
        "observerId": "observer-uid",
        "firstName": "John",
        "lastName": "Smith",
        "email": "observer@example.com",
        "phoneNumber": "+12345678901",
        "role": "primary",
        "escalationLevel": 1,
        "status": "active",
        "assignedAt": "2025-11-10T00:00:00Z"
      }
    ]
  }
}
```

---

### 3.3 Update Observer Assignment

**Endpoint:** `PUT /api/v1/patients/{patientId}/observers/{assignmentId}`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "role": "secondary",
  "escalationLevel": 2,
  "status": "active"
}
```

**Response:** `200 OK`
```json
{
  "data": {
    "assignmentId": "uuid",
    "role": "secondary",
    "escalationLevel": 2,
    "status": "active",
    "updatedAt": "2025-11-10T01:00:00Z"
  },
  "message": "Observer assignment updated"
}
```

---

### 3.4 Remove Observer

**Endpoint:** `DELETE /api/v1/patients/{patientId}/observers/{assignmentId}`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Observer removed from patient"
}
```

---

### 3.5 Resend Invitation

**Endpoint:** `POST /api/v1/invitations/{invitationId}/resend`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Invitation resent successfully",
  "data": {
    "expiresAt": "2025-11-17T00:00:00Z"
  }
}
```

---

### 3.6 Cancel Invitation

**Endpoint:** `DELETE /api/v1/invitations/{invitationId}`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Invitation cancelled"
}
```

---

## 4. Alert Management Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-005, SRS-REQ-ALERT-001, SRS-REQ-ALERT-002, SRS-REQ-ALERT-003, SRS-SRD-REQ-INT-002, SRS-SRD-REQ-INT-003, SRS-REQ-SCHED-006]**

### 4.1 Create Alert

**Endpoint:** `POST /api/v1/patients/{patientId}/alerts`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "name": "Morning Medication Check",
  "message": "Please confirm morning insulin dose administered: 10 units Humalog",
  "severity": "high",
  "schedule": {
    "rrule": "FREQ=DAILY;BYHOUR=8;BYMINUTE=0",
    "timezone": "America/New_York"
  },
  "autoResolveTimeoutMinutes": 30
}
```

**Response:** `201 Created`
```json
{
  "data": {
    "id": "uuid",
    "patientId": "patient-uuid",
    "name": "Morning Medication Check",
    "message": "Please confirm morning insulin dose...",
    "severity": "high",
    "scheduleRrule": "FREQ=DAILY;BYHOUR=8;BYMINUTE=0",
    "scheduleTimezone": "America/New_York",
    "status": "active",
    "nextTriggerAt": "2025-11-11T08:00:00-05:00",
    "createdAt": "2025-11-10T00:00:00Z"
  },
  "message": "Alert created and scheduled successfully"
}
```

**Validation:**
- `name` is required, max 200 characters
- `message` is required, max 1000 characters
- `severity` must be one of: `low`, `medium`, `high`, `critical`
- `schedule.rrule` must be valid iCalendar RRULE syntax
- `schedule.timezone` must be valid IANA timezone

---

### 4.2 List Patient Alerts

**Endpoint:** `GET /api/v1/patients/{patientId}/alerts`

**Authentication:** Required (Caretaker or assigned Observer)

**Query Parameters:**
- `status` (optional): Filter by status
- `limit` (optional): Number of results
- `offset` (optional): Pagination offset

**Response:** `200 OK`
```json
{
  "data": {
    "alerts": [
      {
        "id": "uuid",
        "name": "Morning Medication Check",
        "severity": "high",
        "status": "active",
        "nextTriggerAt": "2025-11-11T08:00:00-05:00",
        "createdAt": "2025-11-10T00:00:00Z"
      }
    ],
    "total": 5
  }
}
```

---

### 4.3 Get Alert Details

**Endpoint:** `GET /api/v1/alerts/{alertId}`

**Authentication:** Required (Caretaker or assigned Observer)

**Response:** `200 OK`
```json
{
  "data": {
    "id": "uuid",
    "patientId": "patient-uuid",
    "name": "Morning Medication Check",
    "message": "Please confirm morning insulin dose...",
    "severity": "high",
    "scheduleRrule": "FREQ=DAILY;BYHOUR=8;BYMINUTE=0",
    "scheduleTimezone": "America/New_York",
    "status": "active",
    "nextTriggerAt": "2025-11-11T08:00:00-05:00",
    "autoResolveTimeoutMinutes": 30,
    "createdAt": "2025-11-10T00:00:00Z",
    "updatedAt": "2025-11-10T00:00:00Z"
  }
}
```

---

### 4.4 Update Alert

**Endpoint:** `PUT /api/v1/alerts/{alertId}`

**Authentication:** Required (Caretaker)

**Request Body:** Same as create alert

**Response:** `200 OK`
```json
{
  "data": {
    "id": "uuid",
    "name": "Updated Alert Name",
    "updatedAt": "2025-11-10T01:00:00Z"
  },
  "message": "Alert updated successfully"
}
```

**Note:** Updating schedule will cancel existing Quirrel jobs and create new ones.

---

### 4.5 Cancel Alert

**Endpoint:** `DELETE /api/v1/alerts/{alertId}`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Alert cancelled and all scheduled instances removed"
}
```

---

### 4.6 Pause Alert

**Endpoint:** `POST /api/v1/alerts/{alertId}/pause`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Alert paused",
  "data": {
    "status": "paused"
  }
}
```

---

### 4.7 Resume Alert

**Endpoint:** `POST /api/v1/alerts/{alertId}/resume`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Alert resumed",
  "data": {
    "status": "active",
    "nextTriggerAt": "2025-11-11T08:00:00-05:00"
  }
}
```

---

### 4.8 Test Alert

**Endpoint:** `POST /api/v1/alerts/{alertId}/test`

**Authentication:** Required (Caretaker)

**Response:** `200 OK`
```json
{
  "message": "Test alert triggered",
  "data": {
    "incidentId": "uuid",
    "incidentNumber": 1234
  }
}
```

**Note:** Manually triggers an incident for testing purposes.

---

## 5. Incident Management Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-006, SRS-REQ-INC-001, SRS-REQ-ESC-002, SRS-REQ-ESC-003, SRS-REQ-ESC-004]**

### 5.1 List Incidents (Caretaker)

**Endpoint:** `GET /api/v1/incidents`

**Authentication:** Required (Caretaker)

**Query Parameters:**
- `status` (optional): Filter by status
- `patientId` (optional): Filter by patient
- `limit` (optional): Number of results
- `offset` (optional): Pagination offset

**Response:** `200 OK`
```json
{
  "data": {
    "incidents": [
      {
        "id": "uuid",
        "incidentNumber": 1234,
        "alertId": "alert-uuid",
        "patientId": "patient-uuid",
        "patientName": "Sarah Johnson",
        "status": "triggered",
        "severity": "high",
        "currentEscalationLevel": 1,
        "assignedObserver": {
          "id": "observer-uid",
          "firstName": "John",
          "lastName": "Smith"
        },
        "triggeredAt": "2025-11-10T08:00:00Z",
        "escalationCount": 0
      }
    ],
    "total": 10
  }
}
```

---

### 5.2 Get Incident Details

**Endpoint:** `GET /api/v1/incidents/{incidentId}`

**Authentication:** Required (Caretaker or assigned Observer)

**Response:** `200 OK`
```json
{
  "data": {
    "id": "uuid",
    "incidentNumber": 1234,
    "alert": {
      "id": "alert-uuid",
      "name": "Morning Medication Check",
      "message": "Please confirm morning insulin dose..."
    },
    "patient": {
      "id": "patient-uuid",
      "name": "Sarah Johnson"
    },
    "status": "triggered",
    "severity": "high",
    "currentEscalationLevel": 1,
    "assignedObserver": {
      "id": "observer-uid",
      "firstName": "John",
      "lastName": "Smith"
    },
    "triggeredAt": "2025-11-10T08:00:00Z",
    "acknowledgedAt": null,
    "resolvedAt": null,
    "escalationCount": 0,
    "actions": [
      {
        "action": "triggered",
        "actorType": "system",
        "timestamp": "2025-11-10T08:00:00Z"
      }
    ]
  }
}
```

---

### 5.3 Resolve Incident

**Endpoint:** `POST /api/v1/incidents/{incidentId}/resolve`

**Authentication:** Required (Caretaker or assigned Observer)

**Request Body:**
```json
{
  "note": "Medication confirmed administered at 8:05 AM"
}
```

**Response:** `200 OK`
```json
{
  "message": "Incident resolved",
  "data": {
    "id": "uuid",
    "status": "resolved",
    "resolvedAt": "2025-11-10T08:05:00Z",
    "resolvedBy": "observer-uid"
  }
}
```

---

### 5.4 Reassign Incident

**Endpoint:** `POST /api/v1/incidents/{incidentId}/reassign`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "observerId": "new-observer-uid"
}
```

**Response:** `200 OK`
```json
{
  "message": "Incident reassigned",
  "data": {
    "assignedObserverId": "new-observer-uid"
  }
}
```

---

### 5.5 Add Note to Incident

**Endpoint:** `POST /api/v1/incidents/{incidentId}/notes`

**Authentication:** Required (Caretaker or assigned Observer)

**Request Body:**
```json
{
  "note": "Patient confirmed medication taken"
}
```

**Response:** `200 OK`
```json
{
  "message": "Note added",
  "data": {
    "actionId": "uuid",
    "note": "Patient confirmed medication taken",
    "createdAt": "2025-11-10T08:10:00Z"
  }
}
```

---

## 6. Observer Operations Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-007]**

### 6.1 Get User Profile

**[Implements: SRS-REQ-UI-001] (SRS - Phase 1)**
**[Implements: SRS-SRD-REQ-SEC-001] (SRS - Security)**
**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-007, SRS-REQ-API-008] (SRD)**

**Endpoint:** `GET /api/v1/me`

**Authentication:** Required (Caretaker or Observer)

**Implementation Status:** ✅ Completed (Phase 1 - November 20, 2025)

**Response:** `200 OK`
```json
{
  "data": {
    "id": "observer-uid",
    "email": "observer@example.com",
    "phoneNumber": "+12345678901",
    "firstName": "John",
    "lastName": "Smith",
    "accountStatus": "active",
    "notificationPreferences": {
      "primaryMethod": "sms",
      "quietHours": {
        "enabled": true,
        "startTime": "22:00",
        "endTime": "07:00",
        "timezone": "America/New_York"
      },
      "highUrgencyOverride": true
    }
  }
}
```

---

### 6.2 Update User Profile

**[Implements: SRS-REQ-UI-001] (SRS - Phase 1)**
**[Implements: SRS-SRD-REQ-SEC-001] (SRS - Security)**
**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-007] (SRD)**

**Endpoint:** `PUT /api/v1/me`

**Authentication:** Required (Caretaker or Observer)

**Implementation Status:** 📋 Documented (To be implemented in Phase 2)

**Request Body:**
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "phoneNumber": "+12345678901"
}
```

**Response:** `200 OK`
```json
{
  "message": "Profile updated",
  "data": {
    "updatedAt": "2025-11-10T01:00:00Z"
  }
}
```

---

### 6.3 Update Notification Preferences

**Endpoint:** `PUT /api/v1/me/preferences`

**Authentication:** Required (Observer)

**Request Body:**
```json
{
  "primaryMethod": "both",
  "quietHours": {
    "enabled": true,
    "startTime": "22:00",
    "endTime": "07:00",
    "timezone": "America/New_York"
  },
  "highUrgencyOverride": true,
  "acknowledgmentTimeoutMinutes": 5
}
```

**Response:** `200 OK`
```json
{
  "message": "Preferences updated"
}
```

---

### 6.4 List Assigned Patients

**Endpoint:** `GET /api/v1/me/patients`

**Authentication:** Required (Observer)

**Response:** `200 OK`
```json
{
  "data": {
    "patients": [
      {
        "patientId": "patient-uuid",
        "patientName": "Sarah Johnson",
        "role": "primary",
        "escalationLevel": 1,
        "status": "active"
      }
    ]
  }
}
```

---

### 6.5 List Active Incidents

**Endpoint:** `GET /api/v1/me/incidents`

**Authentication:** Required (Observer)

**Query Parameters:**
- `status` (optional): Filter by status
- `limit` (optional): Number of results

**Response:** `200 OK`
```json
{
  "data": {
    "incidents": [
      {
        "id": "uuid",
        "incidentNumber": 1234,
        "patientName": "Sarah Johnson",
        "alertName": "Morning Medication Check",
        "status": "triggered",
        "severity": "high",
        "triggeredAt": "2025-11-10T08:00:00Z"
      }
    ]
  }
}
```

---

### 6.6 Acknowledge Incident

**Endpoint:** `POST /api/v1/incidents/{incidentId}/acknowledge`

**Authentication:** Required (Observer)

**Response:** `200 OK`
```json
{
  "message": "Incident acknowledged",
  "data": {
    "id": "uuid",
    "status": "acknowledged",
    "acknowledgedAt": "2025-11-10T08:02:00Z",
    "assignedObserverId": "observer-uid"
  }
}
```

---

## 7. Onboarding Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-008, SRS-REQ-OBS-003, SRS-REQ-INT-009]**

### 7.1 Get Invitation Details

**Endpoint:** `GET /api/v1/invitations/{token}`

**Authentication:** Public (token-based)

**Response:** `200 OK`
```json
{
  "data": {
    "invitationId": "uuid",
    "patientName": "Sarah Johnson",
    "caretakerName": "Jane Doe",
    "role": "primary",
    "escalationLevel": 1,
    "expiresAt": "2025-11-17T00:00:00Z",
    "status": "pending"
  }
}
```

**Error:** `404 Not Found` if token is invalid or expired

---

### 7.2 Accept Invitation

**Endpoint:** `POST /api/v1/invitations/{token}/accept`

**Authentication:** Public (token-based)

**Request Body:**
```json
{
  "idToken": "firebase-google-id-token",
  "phoneNumber": "+12345678901",
  "notificationPreferences": {
    "primaryMethod": "sms",
    "highUrgencyOverride": true
  }
}
```

**Response:** `201 Created`
```json
{
  "message": "Account created and invitation accepted",
  "data": {
    "observerId": "observer-uid",
    "accountStatus": "active",
    "email": "observer@gmail.com",
    "firstName": "John",
    "lastName": "Smith"
  }
}
```

**Validation:**
- ID token must be valid Firebase Google authentication token
- Phone number must be E.164 format (required for SMS notifications)
- User profile (email, name) automatically extracted from Google account

---

### 7.3 Verify Phone

**Endpoint:** `POST /api/v1/auth/verify-phone`

**Authentication:** Required (Observer)

**Request Body:**
```json
{
  "phoneNumber": "+12345678901",
  "code": "123456"
}
```

**Response:** `200 OK`
```json
{
  "message": "Phone number verified"
}
```

**Note:** Email is automatically verified via Google account during authentication. Only phone verification is required for SMS notifications.

---

## 8. Public Endpoints (Token-Based)

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-009, SRS-SRD-REQ-SEC-003, SRS-REQ-INT-008]**

### 8.1 Acknowledge Incident (Public)

**Endpoint:** `GET /ack/{token}`

**Authentication:** Public (token-based)

**Response:** `200 OK` (HTML page for acknowledgment)

**Endpoint:** `POST /ack/{token}`

**Authentication:** Public (token-based)

**Response:** `200 OK`
```json
{
  "message": "Incident acknowledged",
  "data": {
    "incidentId": "uuid",
    "incidentNumber": 1234,
    "status": "acknowledged"
  }
}
```

---

### 8.2 Resolve Incident (Public)

**Endpoint:** `GET /resolve/{token}`

**Authentication:** Public (token-based)

**Response:** `200 OK` (HTML page for resolution)

**Endpoint:** `POST /resolve/{token}`

**Authentication:** Public (token-based)

**Request Body:**
```json
{
  "note": "Medication confirmed"
}
```

**Response:** `200 OK`
```json
{
  "message": "Incident resolved",
  "data": {
    "incidentId": "uuid",
    "status": "resolved"
  }
}
```

---

## 9. Webhook Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-010, SRD-REQ-FUNC-007, SRD-REQ-FUNC-008, SRS-REQ-NOTIF-001, SRS-REQ-NOTIF-002, SRS-SRD-REQ-INT-001, SRS-SRD-REQ-INT-002, SRS-REQ-INT-006, SRS-REQ-INT-007, SRS-REQ-SCHED-001, SRS-REQ-SCHED-002, SRS-REQ-SCHED-004, SRS-SRD-REQ-SEC-004]**

### 9.1 Alert Trigger Webhook

**[Implements: SRS-SRD-REQ-INT-001, SRS-REQ-SCHED-001, SRS-REQ-SCHED-005, SRS-SRD-REQ-SEC-002, SRS-SRD-REQ-SEC-004, SRS-REQ-SEC-007]**

**Endpoint:** `POST /webhooks/alert-trigger`

**Authentication:** HMAC signature validation

**Request Headers:**
```
X-Quirrel-Signature: <hmac-sha256-signature>
```

**Request Body:**
```json
{
  "scheduleId": "quirrel-schedule-id",
  "alertId": "alert-uuid",
  "patientId": "patient-uuid",
  "scheduledTime": "2025-11-10T08:00:00Z",
  "occurrenceId": "occurrence-uuid",
  "scheduleVersion": 1
}
```

**Response:** `200 OK`
```json
{
  "received": true,
  "incidentId": "incident-uuid"
}
```

**Error:** `400 Bad Request` if signature invalid or payload invalid

---

### 9.2 SMS Status Webhook

**[Implements: SRD-REQ-FUNC-008, SRS-REQ-NOTIF-002, SRS-REQ-INT-006, SRS-REQ-INT-007]**

**Endpoint:** `POST /webhooks/sms-status`

**Authentication:** Twilio signature validation

**Request Body:** (Twilio webhook format)

**Response:** `200 OK`

---

## 10. Escalation Policy Endpoints

**[Implements: SRD-REQ-FUNC-002, SRS-REQ-API-011, SRD-REQ-FUNC-005, SRS-REQ-ESC-001, SRS-REQ-ESC-005]**

### 10.1 Create/Update Escalation Policy

**Endpoint:** `POST /api/v1/patients/{patientId}/escalation-policy`

**Authentication:** Required (Caretaker)

**Request Body:**
```json
{
  "name": "Standard Patient Escalation",
  "levels": [
    {
      "level": 1,
      "targets": [
        {
          "observerId": "observer-uuid",
          "role": "primary"
        }
      ],
      "timeoutMinutes": 5
    },
    {
      "level": 2,
      "targets": [
        {
          "observerId": "observer-uuid-2",
          "role": "secondary"
        }
      ],
      "timeoutMinutes": 10
    }
  ],
  "repeatPolicy": {
    "enabled": true,
    "maxRepeats": 3,
    "restartFrom": "level-1"
  }
}
```

**Response:** `200 OK`
```json
{
  "message": "Escalation policy updated",
  "data": {
    "policyId": "uuid",
    "patientId": "patient-uuid"
  }
}
```

---

### 10.2 Get Escalation Policy

**Endpoint:** `GET /api/v1/patients/{patientId}/escalation-policy`

**Authentication:** Required (Caretaker or assigned Observer)

**Response:** `200 OK`
```json
{
  "data": {
    "id": "uuid",
    "patientId": "patient-uuid",
    "name": "Standard Patient Escalation",
    "policyConfig": {
      "levels": [ ... ],
      "repeatPolicy": { ... }
    }
  }
}
```

---

## 11. Error Handling

### 11.1 Error Codes

- `VALIDATION_ERROR` - Request validation failed
- `AUTHENTICATION_ERROR` - Authentication failed
- `AUTHORIZATION_ERROR` - User doesn't have permission
- `NOT_FOUND` - Resource not found
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `EXTERNAL_SERVICE_ERROR` - External service failure
- `INTERNAL_ERROR` - Internal server error

### 11.2 Rate Limiting

- **Caretaker:** 100 requests/minute
- **Observer:** 20 requests/minute
- **Webhooks:** 1000 requests/hour per account

---

**End of Document**

