# Database Schema Documentation
## Patient Alert Notification System - Firestore Collections

**Version:** 1.0  
**Date:** November 9, 2025  
**Database:** Cloud Firestore  
**Document Status:** Draft

---

## 1. Overview

This document defines the Firestore database schema for the Patient Alert Notification System. All collections use Firestore's document-based NoSQL structure with subcollections where appropriate for hierarchical data organization.

### 1.1 Design Principles

- **Denormalization:** Store frequently accessed data together to minimize reads
- **Security:** Structure data to support Firestore security rules
- **Scalability:** Use composite indexes for efficient queries
- **Real-time:** Structure for efficient real-time listeners

### 1.2 Layout and Navigation Support

The database schema supports the application's layout structure through existing fields:

**Top Navigation Bar:**
- `caretakers.photoUrl` and `observers.photoUrl` - Google profile pictures displayed in top-right
- `caretakers.firstName/lastName` and `observers.firstName/lastName` - User display names

**Left-Side Menu Navigation:**
- `patients.activeAlertsCount` - Badge count for Alerts menu item
- `patients.activeIncidentsCount` - Badge count for Incidents menu item
- `patient_observers` collection - Determines role-based menu visibility (Observers menu for caretakers only)

**Settings Pages (accessed from dropdown menu):**
- `caretakers.metadata.preferences` - Caretaker account preferences
- `observers.notificationPreferences` - Observer notification settings (SMS/email, quiet hours, etc.)

**Role-Based Navigation:**
- User role determined by collection membership (caretakers vs observers)
- `patient_observers.role` field used to display observer's role per patient

All layout components use real-time Firestore listeners on these fields to provide immediate updates when data changes.

---

## 2. Core Collections

**[Implements: SRD-REQ-DB-001, SRS-REQ-PATIENT-001, SRS-REQ-OBS-001, SRS-REQ-OBS-007, SRS-REQ-ALERT-001, SRS-REQ-INC-001, SRS-REQ-ESC-001, SRS-REQ-NOTIF-003]**

### 2.1 Caretakers Collection

**[Implements: SRD-REQ-DB-001, SRS-SRD-REQ-SEC-001, SRS-SRD-REQ-SEC-003]**

**Collection Path:** `caretakers/{caretakerId}`

**Purpose:** Store caretaker (account owner) user accounts and profile information.

**Document Structure:**
```typescript
interface Caretaker {
  id: string;                    // Document ID (matches Firebase Auth UID)
  email: string;                 // Unique email address (from Google account)
  phoneNumber?: string;           // E.164 format, encrypted (optional for caretakers)
  firstName: string;             // From Google account
  lastName: string;              // From Google account
  googleUid: string;             // Google user ID (matches Firebase Auth UID)
  photoUrl?: string;             // Google profile photo URL
  accountStatus: 'active' | 'suspended' | 'deleted';
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastLoginAt?: Timestamp;
  metadata?: {
    timezone?: string;
    preferences?: {
      emailNotifications?: boolean;
      smsNotifications?: boolean;
    };
  };
}
```

**Indexes:**
- `email` (unique constraint via security rules)

**Security Rules:**
```javascript
match /caretakers/{caretakerId} {
  allow read, write: if request.auth != null && 
    request.auth.uid == caretakerId;
}
```

---

### 2.2 Patients Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-PATIENT-001, SRD-REQ-DB-002, SRS-REQ-SEC-005]**

**Collection Path:** `patients/{patientId}`

**Purpose:** Store patient profiles managed by caretakers.

**Document Structure:**
```typescript
interface Patient {
  id: string;                    // Document ID (UUID)
  caretakerId: string;            // Reference to caretakers collection
  name: string;                   // Patient name/identifier
  description?: string;            // Optional medical context
  photoUrl?: string;              // Optional profile photo URL
  status: 'active' | 'inactive' | 'archived';
  createdAt: Timestamp;
  updatedAt: Timestamp;
  
  // Denormalized counts for dashboard
  activeAlertsCount?: number;
  activeIncidentsCount?: number;
  observersCount?: number;
}
```

**Indexes:**
- `caretakerId + status` (composite)
- `status + createdAt` (composite)

**Security Rules:**
```javascript
match /patients/{patientId} {
  allow read: if request.auth != null && (
    // Caretaker owns the patient
    resource.data.caretakerId == request.auth.uid ||
    // Observer is assigned to the patient
    exists(/databases/$(database)/documents/patient_observers/$(request.auth.uid + '_' + patientId))
  );
  allow write: if request.auth != null && 
    resource.data.caretakerId == request.auth.uid;
  allow create: if request.auth != null && 
    request.resource.data.caretakerId == request.auth.uid;
}
```

---

### 2.3 Observers Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-OBS-001, SRS-SRD-REQ-SEC-001, SRS-SRD-REQ-SEC-003, SRD-REQ-DB-002]**

**Collection Path:** `observers/{observerId}`

**Purpose:** Store observer (responder) user accounts.

**Document Structure:**
```typescript
interface Observer {
  id: string;                     // Document ID (matches Firebase Auth UID)
  email: string;                  // Unique email address (from Google account)
  phoneNumber: string;            // E.164 format, encrypted, unique (required for SMS)
  firstName: string;              // From Google account
  lastName: string;              // From Google account
  googleUid: string;              // Google user ID (matches Firebase Auth UID)
  photoUrl?: string;              // Google profile photo URL
  accountStatus: 'invited' | 'active' | 'paused' | 'inactive';
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastLoginAt?: Timestamp;
  
  // Notification preferences
  notificationPreferences: {
    primaryMethod: 'sms' | 'email' | 'both';
    quietHours?: {
      enabled: boolean;
      startTime: string;          // HH:mm format
      endTime: string;             // HH:mm format
      timezone: string;
    };
    highUrgencyOverride: boolean;   // Ignore quiet hours for Critical
    acknowledgmentTimeoutMinutes?: number;
  };
}
```

**Indexes:**
- `email` (unique constraint)
- `phoneNumber` (unique constraint)
- `accountStatus + createdAt` (composite)

**Security Rules:**
```javascript
match /observers/{observerId} {
  allow read: if request.auth != null && (
    request.auth.uid == observerId ||
    // Caretakers can read observers assigned to their patients
    exists(/databases/$(database)/documents/patient_observers/$(observerId + '_*'))
  );
  allow write: if request.auth != null && 
    request.auth.uid == observerId;
}
```

---

### 2.4 Observer Invitations Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-OBS-003, SRS-REQ-SEC-006, SRD-REQ-DB-002]**

**Collection Path:** `observer_invitations/{invitationId}`

**Purpose:** Track pending observer invitations before account creation.

**Document Structure:**
```typescript
interface ObserverInvitation {
  id: string;                     // Document ID (UUID)
  caretakerId: string;            // Reference to caretakers collection
  patientId: string;              // Reference to patients collection
  email: string;
  phoneNumber: string;            // E.164 format, encrypted
  firstName: string;
  lastName: string;
  role: 'primary' | 'secondary' | 'backup' | 'always_notify';
  escalationLevel: number;        // 1-5
  customMessage?: string;
  invitationToken: string;        // Hashed token
  status: 'pending' | 'accepted' | 'expired' | 'cancelled';
  invitedAt: Timestamp;
  expiresAt: Timestamp;           // 7 days from invitedAt
  acceptedAt?: Timestamp;
}
```

**Indexes:**
- `invitationToken` (for lookup)
- `status + expiresAt` (composite, for cleanup)
- `caretakerId + status` (composite)
- `patientId + status` (composite)

**Security Rules:**
```javascript
match /observer_invitations/{invitationId} {
  allow read: if request.auth != null && (
    // Caretaker can read their invitations
    resource.data.caretakerId == request.auth.uid ||
    // Public read with valid token (for onboarding)
    request.query.token == resource.data.invitationToken
  );
  allow write: if request.auth != null && 
    resource.data.caretakerId == request.auth.uid;
}
```

---

### 2.5 Patient Observers Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-OBS-007, SRD-REQ-DB-002, SRS-REQ-SEC-005]**

**Collection Path:** `patient_observers/{assignmentId}`

**Purpose:** Track observer-patient assignments (many-to-many relationship).

**Document Structure:**
```typescript
interface PatientObserver {
  id: string;                     // Document ID: `${observerId}_${patientId}`
  patientId: string;             // Reference to patients collection
  observerId: string;             // Reference to observers collection
  role: 'primary' | 'secondary' | 'backup' | 'always_notify';
  escalationLevel: number;        // 1-5
  status: 'active' | 'paused' | 'inactive';
  assignedAt: Timestamp;
  updatedAt: Timestamp;
  lastNotificationAt?: Timestamp;
}
```

**Indexes:**
- `patientId + status` (composite)
- `observerId + status` (composite)
- `patientId + escalationLevel` (composite, for escalation queries)

**Security Rules:**
```javascript
match /patient_observers/{assignmentId} {
  allow read: if request.auth != null && (
    // Observer can read their own assignments
    resource.data.observerId == request.auth.uid ||
    // Caretaker can read assignments for their patients
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid
  );
  allow write: if request.auth != null && 
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid;
}
```

---

### 2.6 Escalation Policies Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-ESC-001, SRD-REQ-DB-002]**

**Collection Path:** `escalation_policies/{policyId}`

**Purpose:** Store escalation policy configurations for each patient.

**Document Structure:**
```typescript
interface EscalationPolicy {
  id: string;                     // Document ID (UUID)
  patientId: string;              // Reference to patients collection (unique per patient)
  name: string;                   // Policy name
  policyConfig: {
    levels: Array<{
      level: number;              // 1-5
      targets: Array<{
        observerId: string;
        role: 'primary' | 'secondary' | 'backup' | 'always_notify';
      }>;
      timeoutMinutes: number;
      notifyAll?: boolean;        // Notify all targets simultaneously
    }>;
    repeatPolicy?: {
      enabled: boolean;
      maxRepeats: number;
      restartFrom: 'level-1' | 'level-2';
    };
  };
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes:**
- `patientId` (unique)

**Security Rules:**
```javascript
match /escalation_policies/{policyId} {
  allow read: if request.auth != null && (
    // Caretaker owns the patient
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid ||
    // Observer assigned to patient
    exists(/databases/$(database)/documents/patient_observers/$(request.auth.uid + '_' + resource.data.patientId))
  );
  allow write: if request.auth != null && 
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid;
}
```

---

### 2.7 Alerts Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-ALERT-001, SRD-REQ-DB-002]**

**Collection Path:** `alerts/{alertId}`

**Purpose:** Store alert definitions (scheduled checks) for patients.

**Document Structure:**
```typescript
interface Alert {
  id: string;                     // Document ID (UUID)
  patientId: string;              // Reference to patients collection
  name: string;                    // Alert name (e.g., "Morning Medication Check")
  message: string;                 // Alert message/instructions
  severity: 'low' | 'medium' | 'high' | 'critical';
  scheduleRrule: string;           // iCalendar RRULE format
  scheduleTimezone: string;        // IANA timezone (e.g., "America/New_York")
  escalationPolicyId: string;      // Reference to escalation_policies collection
  autoResolveTimeoutMinutes?: number; // Optional auto-resolve timeout
  status: 'active' | 'paused' | 'cancelled' | 'completed';
  createdAt: Timestamp;
  updatedAt: Timestamp;
  nextTriggerAt?: Timestamp;      // Next scheduled occurrence
}
```

**Indexes:**
- `patientId + status` (composite)
- `status + nextTriggerAt` (composite, for scheduled queries)
- `escalationPolicyId`

**Security Rules:**
```javascript
match /alerts/{alertId} {
  allow read: if request.auth != null && (
    // Caretaker owns the patient
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid ||
    // Observer assigned to patient
    exists(/databases/$(database)/documents/patient_observers/$(request.auth.uid + '_' + resource.data.patientId))
  );
  allow write: if request.auth != null && 
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid;
}
```

---

### 2.8 Alert Schedules Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-ALERT-002, SRS-REQ-SCHED-001, SRS-SRD-REQ-INT-001, SRS-SRD-REQ-INT-002, SRS-REQ-INT-005]**

**Collection Path:** `alert_schedules/{scheduleId}`

**Purpose:** Store schedule metadata and tracking for Quirrel integration.

**Document Structure:**
```typescript
interface AlertSchedule {
  id: string;                     // Document ID (UUID)
  alertId: string;                // Reference to alerts collection (unique per alert)
  rrule: string;                  // iCalendar RRULE format
  timezone: string;                // IANA timezone
  startDate: Timestamp;
  endDate?: Timestamp;            // Optional end date
  lastGeneratedUntil?: Timestamp; // Last occurrence generated (90-day window)
  nextOccurrence?: Timestamp;     // Next scheduled occurrence
  scheduleVersion: number;         // Incremented on updates
  status: 'active' | 'paused' | 'cancelled';
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes:**
- `alertId` (unique)
- `status + nextOccurrence` (composite)
- `status + lastGeneratedUntil` (composite, for rolling window)

**Security Rules:**
```javascript
match /alert_schedules/{scheduleId} {
  allow read: if request.auth != null && 
    exists(/databases/$(database)/documents/alerts/$(resource.data.alertId)) &&
    // Check patient ownership via alert
    exists(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/alerts/$(resource.data.alertId)).data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/alerts/$(resource.data.alertId)).data.patientId)).data.caretakerId == request.auth.uid;
  allow write: if false; // Only Cloud Functions can write
}
```

---

### 2.9 Scheduled Instances Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-ALERT-003, SRS-REQ-SCHED-002, SRS-SRD-REQ-INT-002, SRS-REQ-ESC-004]**

**Collection Path:** `scheduled_instances/{instanceId}`

**Purpose:** Track individual scheduled occurrences (90-day window).

**Document Structure:**
```typescript
interface ScheduledInstance {
  id: string;                     // Document ID (UUID)
  scheduleId: string;              // Reference to alert_schedules collection
  alertId: string;                 // Reference to alerts collection
  scheduledTime: Timestamp;       // When this occurrence should trigger
  quirrelJobId?: string;          // Quirrel job ID
  status: 'scheduled' | 'sent' | 'triggered' | 'failed' | 'cancelled';
  attempts: number;               // Retry attempts
  lastAttempt?: Timestamp;
  error?: string;                  // Error message if failed
  createdAt: Timestamp;
}
```

**Indexes:**
- `alertId + scheduledTime` (composite, unique)
- `scheduledTime + status` (composite, for due queries)
- `scheduleId + scheduledTime` (composite)
- `status + createdAt` (composite, for cleanup)

**Security Rules:**
```javascript
match /scheduled_instances/{instanceId} {
  allow read: if request.auth != null && 
    exists(/databases/$(database)/documents/alerts/$(resource.data.alertId)) &&
    // Check patient ownership via alert
    exists(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/alerts/$(resource.data.alertId)).data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/alerts/$(resource.data.alertId)).data.patientId)).data.caretakerId == request.auth.uid;
  allow write: if false; // Only Cloud Functions can write
}
```

---

### 2.10 Incidents Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-INC-001, SRD-REQ-DB-002, SRS-REQ-SEC-005]**

**Collection Path:** `incidents/{incidentId}`

**Purpose:** Store alert incidents (triggered alerts requiring response).

**Document Structure:**
```typescript
interface Incident {
  id: string;                     // Document ID (UUID)
  incidentNumber: number;         // Auto-incrementing, human-readable number
  alertId: string;                // Reference to alerts collection
  patientId: string;              // Reference to patients collection
  incidentKey: string;            // For deduplication (alertId + scheduledTime)
  status: 'triggered' | 'acknowledged' | 'resolved' | 'auto_resolved';
  severity: 'low' | 'medium' | 'high' | 'critical';
  currentEscalationLevel: number; // Current escalation level (1-5)
  assignedObserverId?: string;   // Reference to observers collection
  triggeredAt: Timestamp;
  acknowledgedAt?: Timestamp;
  acknowledgedBy?: string;        // Observer ID
  resolvedAt?: Timestamp;
  resolvedBy?: string;            // Observer or caretaker ID
  resolverType?: 'observer' | 'caretaker' | 'auto';
  escalationCount: number;       // Number of escalations
  message: string;                // Alert message
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

**Indexes:**
- `patientId + status` (composite)
- `status + triggeredAt` (composite)
- `incidentKey` (unique, for deduplication)
- `assignedObserverId + status` (composite)
- `alertId + triggeredAt` (composite)

**Security Rules:**
```javascript
match /incidents/{incidentId} {
  allow read: if request.auth != null && (
    // Caretaker owns the patient
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid ||
    // Observer is assigned to the patient
    exists(/databases/$(database)/documents/patient_observers/$(request.auth.uid + '_' + resource.data.patientId)) ||
    // Observer is assigned to the incident
    resource.data.assignedObserverId == request.auth.uid
  );
  allow write: if request.auth != null && (
    // Caretaker can update any incident for their patients
    exists(/databases/$(database)/documents/patients/$(resource.data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(resource.data.patientId)).data.caretakerId == request.auth.uid ||
    // Observer can acknowledge/resolve assigned incidents
    resource.data.assignedObserverId == request.auth.uid
  );
}
```

---

### 2.11 Notifications Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-NOTIF-003, SRS-REQ-INT-006, SRS-REQ-INT-007]**

**Collection Path:** `notifications/{notificationId}`

**Purpose:** Track notification delivery logs and status.

**Document Structure:**
```typescript
interface Notification {
  id: string;                     // Document ID (UUID)
  incidentId: string;             // Reference to incidents collection
  observerId: string;             // Reference to observers collection
  escalationLevel: number;       // Escalation level when sent
  notificationType: 'sms' | 'email' | 'push';
  messageContent: string;         // Message text sent
  deliveryStatus: 'queued' | 'sent' | 'delivered' | 'failed' | 'read';
  externalMessageId?: string;      // Twilio/SendGrid message ID
  sentAt?: Timestamp;
  deliveredAt?: Timestamp;
  readAt?: Timestamp;
  retryCount: number;
  errorMessage?: string;
  createdAt: Timestamp;
}
```

**Indexes:**
- `incidentId + createdAt` (composite)
- `observerId + createdAt` (composite)
- `deliveryStatus + sentAt` (composite)
- `incidentId + observerId` (composite)

**Security Rules:**
```javascript
match /notifications/{notificationId} {
  allow read: if request.auth != null && (
    // Observer can read their own notifications
    resource.data.observerId == request.auth.uid ||
    // Caretaker can read notifications for their patient incidents
    exists(/databases/$(database)/documents/incidents/$(resource.data.incidentId)) &&
    exists(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/incidents/$(resource.data.incidentId)).data.patientId)) &&
    get(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/incidents/$(resource.data.incidentId)).data.patientId)).data.caretakerId == request.auth.uid
  );
  allow write: if false; // Only Cloud Functions can write
}
```

---

### 2.12 Incident Actions Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-ESC-002]**

**Collection Path:** `incident_actions/{actionId}`

**Purpose:** Audit log of all incident state changes and actions.

**Document Structure:**
```typescript
interface IncidentAction {
  id: string;                     // Document ID (UUID)
  incidentId: string;              // Reference to incidents collection
  actorId: string;                // Observer or caretaker ID
  actorType: 'observer' | 'caretaker' | 'system';
  action: 'triggered' | 'notified' | 'acknowledged' | 'resolved' | 'escalated' | 'reassigned' | 'noted';
  details: {
    escalationLevel?: number;
    observerId?: string;
    notificationMethod?: string;
    note?: string;
    previousStatus?: string;
    newStatus?: string;
  };
  createdAt: Timestamp;
}
```

**Indexes:**
- `incidentId + createdAt` (composite)
- `actorId + createdAt` (composite)
- `action + createdAt` (composite)

**Security Rules:**
```javascript
match /incident_actions/{actionId} {
  allow read: if request.auth != null && (
    // Can read actions for incidents they have access to
    exists(/databases/$(database)/documents/incidents/$(resource.data.incidentId)) &&
    // Check patient ownership or observer assignment
    (exists(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/incidents/$(resource.data.incidentId)).data.patientId)) &&
     get(/databases/$(database)/documents/patients/$(get(/databases/$(database)/documents/incidents/$(resource.data.incidentId)).data.patientId)).data.caretakerId == request.auth.uid) ||
    exists(/databases/$(database)/documents/patient_observers/$(request.auth.uid + '_' + get(/databases/$(database)/documents/incidents/$(resource.data.incidentId)).data.patientId))
  );
  allow write: if false; // Only Cloud Functions can write
}
```

---

### 2.13 Acknowledgment Tokens Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-SEC-006, SRS-REQ-INT-008]**

**Collection Path:** `acknowledgment_tokens/{tokenId}`

**Purpose:** Store secure tokens for acknowledgment and resolution links.

**Document Structure:**
```typescript
interface AcknowledgmentToken {
  id: string;                     // Document ID (UUID)
  incidentId: string;             // Reference to incidents collection
  observerId: string;              // Reference to observers collection
  token: string;                   // Hashed token
  actionType: 'acknowledge' | 'resolve';
  expiresAt: Timestamp;           // 24 hours from creation
  usedAt?: Timestamp;
  createdAt: Timestamp;
}
```

**Indexes:**
- `token` (for lookup)
- `incidentId + observerId` (composite)
- `expiresAt` (for cleanup)

**Security Rules:**
```javascript
match /acknowledgment_tokens/{tokenId} {
  // Public read for token validation (limited fields only)
  allow read: if request.query.token == resource.data.token;
  allow write: if false; // Only Cloud Functions can write
}
```

---

## 3. Composite Indexes

**[Implements: SRD-REQ-DB-003, SRS-REQ-SCALE-002, SRS-SRD-REQ-PERF-004]**

### 3.1 Required Indexes

The following composite indexes MUST be created in Firestore:

**incidents:**
- `status` (Ascending) + `triggeredAt` (Descending)
- `patientId` (Ascending) + `status` (Ascending)
- `assignedObserverId` (Ascending) + `status` (Ascending)
- `alertId` (Ascending) + `triggeredAt` (Descending)

**alerts:**
- `patientId` (Ascending) + `status` (Ascending)
- `status` (Ascending) + `nextTriggerAt` (Ascending)

**notifications:**
- `incidentId` (Ascending) + `createdAt` (Descending)
- `observerId` (Ascending) + `createdAt` (Descending)
- `deliveryStatus` (Ascending) + `sentAt` (Descending)

**scheduled_instances:**
- `scheduledTime` (Ascending) + `status` (Ascending)
- `alertId` (Ascending) + `scheduledTime` (Ascending)

**patient_observers:**
- `patientId` (Ascending) + `escalationLevel` (Ascending)

**observer_invitations:**
- `status` (Ascending) + `expiresAt` (Ascending)

---

## 4. Data Relationships

### 4.1 Relationship Diagram

```
caretakers (1) ──< (many) patients
patients (1) ──< (many) alerts
patients (1) ──< (1) escalation_policies
patients (1) ──< (many) patient_observers
patient_observers (many) ──> (1) observers
alerts (1) ──< (1) alert_schedules
alert_schedules (1) ──< (many) scheduled_instances
alerts (1) ──< (many) incidents
incidents (1) ──< (many) notifications
incidents (1) ──< (many) incident_actions
incidents (1) ──< (many) acknowledgment_tokens
```

### 4.2 Denormalization Strategy

**Denormalized Fields:**
- `patients.activeAlertsCount` - Updated when alerts are created/deleted
- `patients.activeIncidentsCount` - Updated when incidents change status
- `patients.observersCount` - Updated when observers are assigned/removed

**Update Strategy:**
- Use Cloud Functions triggers to maintain denormalized counts
- Batch updates to minimize write operations

---

## 5. Data Validation

### 5.1 Type Validation

All data SHALL be validated using TypeScript interfaces in Cloud Functions before writing to Firestore.

### 5.2 Business Rule Validation

- Patient IDs must reference valid caretaker accounts
- Observer assignments must reference valid patients and observers
- Escalation policies must have at least one level
- Alert schedules must have valid RRULE syntax
- Incident keys must be unique per alert + scheduled time

---

## 6. Data Migration

### 6.1 Initial Setup

1. Create all collections with initial security rules
2. Create composite indexes
3. Set up Cloud Functions triggers for denormalized fields

### 6.2 Future Migrations

- Use Cloud Functions to migrate existing data
- Maintain backward compatibility during migrations
- Test migrations on staging environment first

---

**End of Document**

