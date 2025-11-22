# System Requirements Documentation
## Patient Alert Notification System MVP (PagerDuty-Style)

---

## 1. Executive Summary

This document outlines the system requirements for a Minimum Viable Product (MVP) patient alert notification system modeled after PagerDuty's core functionality. The system enables caretakers to manage multiple patient profiles, assign observers to each patient, and create alerts that notify patient-specific observers via text message according to configurable schedules and escalation policies.

---

## 2. System Overview

### 2.1 Purpose
The system enables caretakers to monitor multiple patients by creating scheduled alerts that notify designated observers through escalating notification patterns, similar to how PagerDuty manages on-call incidents.

### 2.2 Key Features
- Multi-patient profile management
- Patient-specific observer assignments
- Incident-based alert system with acknowledgment
- PagerDuty-style escalation policies
- iCalendar RRULE scheduling syntax support
- External webhook-based scheduling via API
- Observer invitation and onboarding flow
- Alert acknowledgment and resolution tracking

### 2.3 PagerDuty Core Concepts Applied

| PagerDuty Concept | This System Equivalent |
|-------------------|------------------------|
| Service | Patient Profile |
| Escalation Policy | Patient Escalation Policy |
| On-Call Schedule | Alert Schedule |
| Incident | Alert Incident |
| User | Observer |
| Notification Rules | Observer Preferences |
| Acknowledge | Acknowledge Alert |
| Resolve | Resolve Alert |

---

## 3. Functional Requirements

### 3.1 User Roles

#### 3.1.1 Caretaker (Account Owner)
- Can create and manage multiple patient profiles
- Can invite and manage observers for each patient
- Can create, update, and delete alerts per patient
- Can define escalation policies per patient
- Can acknowledge or resolve active incidents
- Has full administrative access to their account

#### 3.1.2 Observer (Responder)
- Receives invitation to monitor specific patient(s)
- Completes onboarding flow to accept invitation
- Receives text notifications when on-call for their patient(s)
- Can acknowledge receipt of alert incidents
- Can resolve incidents
- Can update their notification preferences

#### 3.1.3 System Roles
**SRS-REQ-ROLE-001:** Each observer SHALL have one of the following roles per patient:
- **Primary Observer:** First line of escalation
- **Secondary Observer:** Second level escalation
- **Backup Observer:** Final escalation level
- **Always Notify:** Receives all alerts immediately (optional)

---

### 3.2 Patient Profile Management

#### 3.2.1 Patient Profile Creation
**SRS-REQ-PATIENT-001:** The system SHALL allow caretakers to create patient profiles with:
- Patient ID (system-generated, unique)
- Patient name/identifier
- Patient description (optional medical context)
- Profile photo (optional)
- Status (Active, Inactive, Archived)
- Created date
- Associated caretaker ID

**SRS-REQ-PATIENT-002:** Caretakers SHALL be able to manage multiple patient profiles under a single account.

**SRS-REQ-PATIENT-003:** Each patient profile SHALL have:
- Dedicated observer list
- Dedicated escalation policy
- Independent alert schedules
- Separate incident history

#### 3.2.2 Patient Profile Operations
**SRS-REQ-PATIENT-004:** Caretakers SHALL be able to:
- Create new patient profiles
- Update patient information
- Archive patient profiles (maintains history)
- Delete patient profiles (cascades to related data)
- View all active incidents per patient

---

### 3.3 Observer Management & Assignment

#### 3.3.1 Observer Invitation
**SRS-REQ-OBS-001:** The system SHALL support inviting observers with:
- Observer email address
- Observer phone number (E.164 format)
- Observer first and last name
- Assigned patient ID(s)
- Assigned role per patient
- Custom invitation message (optional)

**SRS-REQ-OBS-002:** The system SHALL generate unique invitation tokens that:
- Expire after 7 days
- Can be resent by caretaker
- Are single-use only
- Include patient context

**SRS-REQ-OBS-003:** Invitations SHALL be sent via:
- SMS to provided phone number
- Email to provided email address (if available)

#### 3.3.2 Observer Onboarding Flow
**SRS-REQ-ONBOARD-001:** The onboarding flow SHALL include the following steps:

**Step 1: Invitation Landing Page**
- Display patient name(s) observer is being added to
- Display caretaker name
- Show observer role for each patient
- Present terms of service and privacy policy
- Require acceptance to proceed

**SRS-REQ-ONBOARD-002:** **Step 2: Account Creation**
- Collect or confirm phone number
- Collect or confirm email address
- Create password (min 8 characters, complexity rules)
- Verify phone via SMS code
- Verify email via confirmation link

**SRS-REQ-ONBOARD-003:** **Step 3: Notification Preferences**
- Configure notification methods (SMS, Email, Both)
- Set quiet hours (optional)
- Configure high-urgency override settings
- Set acknowledgment timeout preference

**SRS-REQ-ONBOARD-004:** **Step 4: Patient Context**
- Display patient information observer will monitor
- Show escalation policy for each patient
- Preview example alert notification
- Provide emergency contact information

**SRS-REQ-ONBOARD-005:** **Step 5: Confirmation**
- Summary of setup
- Test notification option
- Link to dashboard/documentation
- Confirmation of active status

**SRS-REQ-ONBOARD-006:** Upon completion, the system SHALL:
- Activate observer account
- Add observer to patient's notification list
- Send welcome notification
- Log onboarding completion event

#### 3.3.3 Observer-Patient Assignment
**SRS-REQ-OBS-007:** The system SHALL maintain observer assignments with:
- Observer ID
- Patient ID
- Role (Primary, Secondary, Backup, Always Notify)
- Escalation level (1-5)
- Status (Active, Paused, Inactive)
- Assignment date
- Last notification date

**SRS-REQ-OBS-008:** An observer MAY be assigned to multiple patients with different roles.

**SRS-REQ-OBS-009:** Caretakers SHALL be able to:
- Add observers to patients
- Change observer roles
- Remove observers from patients
- Pause observer notifications temporarily
- View observer response history

---

### 3.4 Alert & Incident Management

#### 3.4.1 Alert Creation (Scheduled Check)
**SRS-REQ-ALERT-001:** The system SHALL allow caretakers to create alerts (scheduled checks) with:
- Alert ID (system-generated, unique)
- Alert name (e.g., "Morning Medication Check")
- Patient ID
- Alert message/instructions
- Alert severity (Low, Medium, High, Critical)
- Schedule definition (iCal format)
- Escalation policy reference
- Auto-resolve timeout (optional)

**SRS-REQ-ALERT-002:** The system SHALL validate iCalendar RRULE syntax (RFC 5545) before accepting alert creation.

**SRS-REQ-ALERT-003:** The system SHALL support the following iCalendar RRULE recurrence patterns:
- One-time (specific datetime, no RRULE)
- Daily intervals: `FREQ=DAILY;INTERVAL=n;BYHOUR=h;BYMINUTE=m`
- Weekly intervals: `FREQ=WEEKLY;BYDAY=MO,WE,FR;BYHOUR=h;BYMINUTE=m`
- Hourly intervals: `FREQ=HOURLY;INTERVAL=n`
- Monthly intervals: `FREQ=MONTHLY;BYMONTHDAY=n;BYHOUR=h;BYMINUTE=m`
- Custom RRULE strings per RFC 5545

**SRS-REQ-ALERT-004:** Alerts SHALL support PagerDuty-style urgency levels:
- **Low:** Standard escalation timing
- **High:** Accelerated escalation (reduced timeouts)
- **Critical:** Immediate broadcast to all levels

#### 3.4.2 Incident Lifecycle (PagerDuty Model)
**SRS-REQ-INC-001:** When an alert fires, the system SHALL create an incident with:
- Incident ID (system-generated, unique)
- Alert ID (parent reference)
- Patient ID
- Trigger timestamp
- Current status (Triggered, Acknowledged, Resolved)
- Assigned observer ID (current escalation level)
- Escalation level counter
- Incident key (for deduplication)

**SRS-REQ-INC-002:** Incidents SHALL have the following states:

```
Triggered → Acknowledged → Resolved
     ↓           ↓
  Escalated   Auto-Resolved
     ↓
  Escalated (repeat until max level)
```

**SRS-REQ-INC-003:** **Triggered State:**
- Incident created when alert fires
- First notification sent to primary observer
- Escalation timer starts
- Status visible in dashboard

**SRS-REQ-INC-004:** **Acknowledged State:**
- Observer has confirmed receipt
- Escalation timer pauses
- Other notifications stop
- Incident assigned to acknowledging observer
- Auto-resolve timer starts (if configured)

**SRS-REQ-INC-005:** **Resolved State:**
- Observer or caretaker marks incident complete
- All notifications stop
- Incident closed in system
- Resolution logged with timestamp and resolver

**SRS-REQ-INC-006:** **Auto-Resolution:**
- If acknowledgment timeout expires without response, escalate
- If auto-resolve timeout expires after acknowledgment, auto-resolve
- Caretaker can configure auto-resolve behavior

#### 3.4.3 Incident Actions
**SRS-REQ-INC-007:** Observers SHALL be able to:
- Acknowledge incidents via SMS reply link
- Acknowledge incidents via web dashboard
- Resolve incidents after acknowledgment
- Reassign incidents to another observer
- Add notes to incidents

**SRS-REQ-INC-008:** Caretakers SHALL be able to:
- View all active incidents across patients
- Manually resolve incidents
- Override escalation policies
- Snooze incidents temporarily
- Cancel incident escalation

---

### 3.5 Escalation Policies (PagerDuty Model)

#### 3.5.1 Escalation Policy Structure
**SRS-REQ-ESC-001:** Each patient SHALL have one active escalation policy defining:
- Policy ID (system-generated)
- Policy name
- Patient ID
- Escalation levels (1-5 levels)
- Repeat policy (optional)

**SRS-REQ-ESC-002:** Each escalation level SHALL contain:
- Level number (1-5)
- Target observers (one or more)
- Escalation timeout (minutes until next level)
- Notification method override (optional)

**SRS-REQ-ESC-003:** Example escalation policy structure:
```json
{
  "policyId": "uuid",
  "patientId": "uuid",
  "name": "Standard Patient Escalation",
  "levels": [
    {
      "level": 1,
      "targets": [
        {"observerId": "uuid", "role": "Primary"}
      ],
      "timeoutMinutes": 5
    },
    {
      "level": 2,
      "targets": [
        {"observerId": "uuid", "role": "Secondary"}
      ],
      "timeoutMinutes": 10
    },
    {
      "level": 3,
      "targets": [
        {"observerId": "uuid1", "role": "Backup"},
        {"observerId": "uuid2", "role": "Backup"}
      ],
      "timeoutMinutes": 15
    }
  ],
  "repeatPolicy": {
    "enabled": true,
    "maxRepeats": 3,
    "restartFrom": "level-1"
  }
}
```

#### 3.5.2 Escalation Behavior
**SRS-REQ-ESC-004:** The system SHALL execute escalation as follows:
1. Incident triggered → Notify Level 1 observers
2. Start timeout timer for Level 1
3. If no acknowledgment before timeout → Escalate to Level 2
4. Repeat until incident acknowledged or max level reached
5. If max level reached and no acknowledgment → Execute repeat policy or alert caretaker

**SRS-REQ-ESC-005:** The system SHALL support simultaneous notification of multiple observers at the same level.

**SRS-REQ-ESC-006:** The system SHALL support round-robin notification among observers at the same level (optional for MVP).

**SRS-REQ-ESC-007:** Urgency-based timeout modifiers:
- **Low:** Use configured timeout as-is
- **High:** Reduce timeout by 50%
- **Critical:** Set timeout to 1 minute, notify all levels simultaneously

#### 3.5.3 Repeat Policy
**SRS-REQ-ESC-008:** If all escalation levels are exhausted without acknowledgment, the system SHALL:
- Restart from specified level (typically Level 1)
- Increment repeat counter
- Stop after max repeats reached
- Send critical alert to caretaker
- Log escalation failure

---

### 3.6 Notification System

#### 3.6.1 Notification Delivery
**SRS-REQ-NOTIF-001:** The system SHALL send SMS notifications containing:
- Patient name
- Incident ID (short code)
- Alert message
- Severity indicator
- Timestamp
- Quick action links:
  - "Acknowledge" link
  - "Resolve" link (if already acknowledged)
  - "View Details" link

**SRS-REQ-NOTIF-002:** SMS format example:
```
🔴 HIGH: Patient Sarah - Morning Medication Check

Due at 8:00 AM. Please confirm medication administered.

Incident #1234
Ack: https://app.link/ack/xyz
View: https://app.link/inc/1234
```

**SRS-REQ-NOTIF-003:** The system SHALL track notification delivery:
- Notification ID
- Incident ID
- Observer ID
- Notification method (SMS, Email)
- Status (Queued, Sent, Delivered, Failed, Read)
- Sent timestamp
- Delivered timestamp
- Retry count

**SRS-REQ-NOTIF-004:** Failed notifications SHALL be retried:
- Retry 1: After 1 minute
- Retry 2: After 5 minutes
- Retry 3: After 15 minutes
- After 3 failures: Alert caretaker and escalate

#### 3.6.2 Acknowledgment Flow
**SRS-REQ-NOTIF-005:** When observer clicks acknowledgment link:
1. Validate token and incident status
2. Mark incident as acknowledged
3. Record acknowledging observer ID and timestamp
4. Stop escalation timer
5. Cancel pending notifications to other observers
6. Send confirmation SMS to observer
7. Notify caretaker of acknowledgment (optional)

**SRS-REQ-NOTIF-006:** The system SHALL prevent duplicate acknowledgments:
- First valid acknowledgment wins
- Subsequent acknowledgment attempts return "Already acknowledged" status
- Show current incident assignee

#### 3.6.3 Notification Preferences
**SRS-REQ-NOTIF-007:** Observers SHALL be able to configure:
- Primary notification method (SMS, Email, Both)
- Quiet hours (start time, end time, timezone)
- High-urgency override (ignore quiet hours for Critical severity)
- Notification sound/ringtone preference (mobile app, future)
- Language preference

---

### 3.7 Scheduling System

#### 3.7.1 External Webhook Service Integration
**SRS-REQ-SCHED-001:** The system SHALL integrate with external webhook scheduling service to:
- Create scheduled webhook callbacks from iCalendar RRULE expressions
- Update scheduled callbacks when alerts modified
- Delete scheduled callbacks when alerts cancelled
- Receive webhook POST requests on schedule

**SRS-REQ-SCHED-002:** The system SHALL provide webhook endpoint at:
```
POST /webhooks/alert-trigger
```

**SRS-REQ-SCHED-003:** Webhook payload SHALL include:
```json
{
  "scheduleId": "external-service-schedule-id",
  "alertId": "uuid",
  "patientId": "uuid",
  "triggeredAt": "ISO8601-timestamp",
  "signature": "hmac-signature"
}
```

**SRS-REQ-SCHED-004:** Upon receiving webhook, the system SHALL:
1. Validate webhook signature
2. Verify alert is still active
3. Create incident for the alert
4. Retrieve patient's escalation policy
5. Begin Level 1 notifications
6. Return 200 OK to webhook service

**SRS-REQ-SCHED-005:** The system SHALL handle webhook failures:
- Return 4xx for invalid requests (don't retry)
- Return 5xx for server errors (allow retry)
- Implement idempotency using incident keys
- Log all webhook attempts

**SRS-REQ-SCHED-006:** The system SHALL parse and convert iCalendar RRULE expressions to the external scheduling service's format, handling:
- Timezone conversions
- RRULE validation before submission
- Error handling for unsupported RRULE features

---

### 3.8 User Interface Layout & Structure

#### 3.8.1 Landing Page (Unauthenticated)

**SRS-REQ-UI-001:** When a user is not logged in, the system SHALL display a Landing Page that:
- Presents the application name and value proposition
- Contains a Google Authentication button for sign-in
- Provides clear call-to-action for authentication
- Does not show any authenticated navigation elements

**SRS-REQ-UI-002:** The Landing Page SHALL be responsive and accessible on:
- Desktop browsers (1024px+ width)
- Tablet devices (768px-1023px width)
- Mobile devices (<768px width)

#### 3.8.2 Authenticated Layout Structure

**SRS-REQ-UI-003:** Once a user is authenticated, the application SHALL display a consistent layout structure consisting of:
- Left-side menu pane (desktop/tablet)
- Top navigation bar
- Main content area
- Responsive mobile layout with hamburger menu

#### 3.8.3 Left-Side Menu Pane (Desktop/Tablet)

**SRS-REQ-UI-004:** On desktop and tablet viewports (≥768px width), the system SHALL display a left-side menu pane that:
- Remains fixed/persistent during navigation
- Contains primary navigation links for:
  - Dashboard/Home
  - Patients (list/management)
  - Alerts (list/management)
  - Incidents (active/history)
  - Observers (if caretaker)
  - My Profile (if observer)
  - Settings
- Highlights the currently active navigation item
- Provides clear visual hierarchy and iconography
- Has a minimum width of 200px and maximum width of 280px

**SRS-REQ-UI-005:** The left-side menu pane SHALL:
- Be collapsible/expandable (optional enhancement)
- Support tooltips for menu items when collapsed
- Maintain selection state across page navigation

#### 3.8.4 Top Navigation Bar

**SRS-REQ-UI-006:** The top navigation bar SHALL:
- Span the full width of the application
- Be fixed at the top of the viewport
- Display the application logo/name on the left
- Display user profile information on the top right
- Have a minimum height of 56px for adequate touch targets

**SRS-REQ-UI-007:** The top-right user profile section SHALL display:
- User's Google profile picture (circular avatar)
- User's display name (on larger viewports)
- Visual indicator (icon/arrow) showing clickable dropdown

**SRS-REQ-UI-008:** When the user profile icon is clicked, the system SHALL display a dropdown menu containing:
- User's full name and email
- "Account Settings" link
- "Notification Preferences" link (observers)
- "Help/Documentation" link
- "Logout" option with clear visual separation

**SRS-REQ-UI-009:** The logout option SHALL:
- Be clearly labeled as "Logout" or "Sign Out"
- Be visually separated from other menu items (e.g., divider line)
- Positioned as the last item in the dropdown
- Include an icon for visual clarity

#### 3.8.5 Mobile Responsive Layout

**SRS-REQ-UI-010:** On mobile viewports (<768px width), the system SHALL:
- Hide the left-side menu pane
- Display a hamburger menu icon (☰) in the top-left of the navigation bar
- Maintain the user profile icon in the top-right

**SRS-REQ-UI-011:** When the hamburger menu icon is tapped, the system SHALL:
- Slide in or overlay the navigation menu from the left
- Display the same navigation items as the desktop menu
- Include a close button (✕) or allow tap-outside to dismiss
- Prevent body scroll when menu is open (trap focus)

**SRS-REQ-UI-012:** The mobile navigation menu SHALL:
- Animate smoothly when opening/closing
- Use full-height overlay or slide-in drawer pattern
- Close automatically after navigation selection
- Support swipe-to-close gesture (optional enhancement)

#### 3.8.6 Common UX Patterns

**SRS-REQ-UI-013:** The system SHALL display a full-screen loading indicator when:
- Making API calls that require user to wait
- Performing authentication operations
- Loading critical application data
- Navigating between major sections

**SRS-REQ-UI-014:** The loading indicator SHALL:
- Block the entire screen to prevent user interaction
- Use a semi-transparent overlay (e.g., rgba(0, 0, 0, 0.5) or similar)
- Display a spinner or loading animation in the center
- Include loading text message when appropriate (e.g., "Loading patients...")
- Prevent clicking through to underlying content

**SRS-REQ-UI-015:** The loading indicator SHALL:
- Appear within 200ms of operation start
- Dismiss immediately when operation completes or fails
- Show for minimum 300ms to avoid flashing on fast operations
- Never block error messages or critical alerts

**SRS-REQ-UI-016:** The system SHALL implement proper loading states for:
- Initial page load
- Form submissions
- Data fetching operations
- File uploads
- Authentication flows
- Navigation between routes

#### 3.8.7 Layout Consistency

**SRS-REQ-UI-017:** The layout structure SHALL:
- Remain consistent across all authenticated pages
- Maintain navigation state during page transitions
- Use semantic HTML5 elements for accessibility
- Support keyboard navigation for all interactive elements

**SRS-REQ-UI-018:** The system SHALL ensure:
- Consistent spacing and padding across all pages
- Unified color scheme and typography
- Accessible contrast ratios (WCAG 2.1 AA minimum)
- Touch target sizes of at least 44x44px on mobile

---

## 4. Non-Functional Requirements

### 4.1 Performance

**SRS-REQ-PERF-001:** The system SHALL process incoming webhooks within 2 seconds.

**SRS-REQ-PERF-002:** Notifications SHALL be queued within 3 seconds of incident creation.

**SRS-REQ-PERF-003:** Acknowledgment links SHALL respond within 1 second.

**SRS-REQ-PERF-004:** The system SHALL support:
- 50 caretaker accounts
- 20 patients per caretaker (1000 total patients)
- 10 observers per patient (10,000 total observers)
- 500 concurrent active incidents
- 1000 scheduled alerts

### 4.2 Reliability

**SRS-REQ-REL-001:** The webhook endpoint SHALL maintain 99.5% uptime.

**SRS-REQ-REL-002:** No incidents SHALL be lost due to system failures (persistent queue).

**SRS-REQ-REL-003:** The system SHALL implement idempotency to prevent duplicate incidents from duplicate webhooks.

**SRS-REQ-REL-004:** All critical state changes SHALL be logged for audit trail.

### 4.3 Security

**SRS-REQ-SEC-001:** All API endpoints SHALL require authentication via:
- Bearer tokens (JWT)
- Session cookies (web dashboard)

**SRS-REQ-SEC-002:** Webhook callbacks SHALL be validated using HMAC-SHA256 signatures.

**SRS-REQ-SEC-003:** Sensitive data SHALL be encrypted:
- Phone numbers (at rest)
- Passwords (bcrypt, minimum 10 rounds)
- Invitation tokens (hashed)

**SRS-REQ-SEC-004:** All API communication SHALL use HTTPS/TLS 1.2+.

**SRS-REQ-SEC-005:** Observers SHALL only access data for their assigned patients.

**SRS-REQ-SEC-006:** Acknowledgment links SHALL:
- Expire after 24 hours
- Be single-use or require authentication for repeated use
- Include CSRF protection

**SRS-REQ-SEC-007:** Rate limiting SHALL be applied:
- 100 requests/minute per caretaker account
- 20 requests/minute per observer
- 1000 webhooks/hour per account

### 4.4 Scalability

**SRS-REQ-SCALE-001:** The notification queue SHALL be horizontally scalable.

**SRS-REQ-SCALE-002:** Database queries SHALL use indexes on:
- Patient ID
- Alert ID
- Incident ID
- Observer ID
- Incident status + patient ID (composite)
- Alert schedule + next trigger time (composite)

**SRS-REQ-SCALE-003:** The system SHALL support database read replicas for queries.

### 4.5 Usability

**SRS-REQ-USE-001:** Caretakers SHALL complete patient profile creation in under 2 minutes.

**SRS-REQ-USE-002:** Observer onboarding SHALL be completable in under 5 minutes.

**SRS-REQ-USE-003:** Observers SHALL be able to acknowledge incidents in 2 taps/clicks.

**SRS-REQ-USE-004:** The system SHALL provide clear error messages with actionable guidance.

**SRS-REQ-USE-005:** The system SHALL provide clear error messages for invalid iCalendar RRULE syntax.

---

## 5. Technical Requirements

### 5.1 System Architecture

#### 5.1.1 Core Components
**SRS-REQ-TECH-001:** The system SHALL consist of:
- REST API (primary interface)
- Webhook receiver endpoint
- Incident management engine
- Escalation policy processor
- Notification queue/worker
- SMS gateway integration
- Email service integration
- Background job processor
- Database (PostgreSQL recommended)
- Cache layer (Redis recommended)
- iCalendar RRULE parser/calculator library (e.g., rrule.js, python-dateutil, or rrule crate)

### 5.2 External Service Integration

#### 5.2.1 Webhook Scheduling Service
**SRS-REQ-INT-001:** The system SHALL integrate with a Quirrel Scheduling service
- Refer to docs/system/quirrel-scheduling-requirements.md for how that system works

**SRS-REQ-INT-002:** The integration SHALL support:
- Creating recurring schedules from iCalendar RRULE expressions or cron syntax
- Updating schedule timing
- Cancelling/deleting schedules
- Receiving webhook callbacks with payload
- Webhook signature verification

**SRS-REQ-INT-003:** The system SHALL include an RRULE-to-Cron converter utility for services that only support cron syntax:
- Convert common RRULE patterns to cron expressions
- Handle timezone conversions
- Validate RRULE compatibility with target service
- Document unsupported RRULE patterns

**SRS-REQ-INT-004:** API calls SHALL include:
```javascript
// Create schedule (example with cron conversion)
POST /schedules
{
  "destination": "https://yourapp.com/webhooks/alert-trigger",
  "schedule": "0 8 * * *",  // Converted from RRULE
  "timezone": "America/New_York",
  "body": {
    "alertId": "uuid",
    "patientId": "uuid"
  }
}

// Response
{
  "scheduleId": "external-id",
  "nextRun": "2025-11-10T08:00:00Z"
}
```

**SRS-REQ-INT-005:** For RRULE patterns that cannot be converted to cron (complex recurrence rules), the system SHALL:
- Store the RRULE internally
- Calculate next trigger times using an RRULE library
- Create one-time scheduled webhooks for each occurrence
- Automatically schedule the next occurrence after each trigger

#### 5.2.2 SMS Gateway
**SRS-REQ-INT-006:** The system SHALL integrate with:
- **Twilio** - Recommended for reliability and features
- Alternative: AWS SNS, MessageBird, Vonage

**SRS-REQ-INT-007:** SMS integration SHALL support:
- Sending individual messages
- Delivery status webhooks
- Message status tracking
- Link shortening (for acknowledgment URLs)
- Rate limiting compliance

**SRS-REQ-INT-008:** SMS API calls SHALL include:
```javascript
// Send SMS
POST /messages
{
  "to": "+1234567890",
  "from": "+1234567890",
  "body": "Alert message with links...",
  "statusCallback": "https://yourapp.com/webhooks/sms-status"
}
```

#### 5.2.3 Email Service (Optional for MVP)
**SRS-REQ-INT-009:** For email notifications, integrate with SendGrid

### 5.3 API Specifications

#### 5.3.1 Authentication
**SRS-REQ-API-001:** All API endpoints SHALL require authentication:
```
Authorization: Bearer <jwt-token>
```

**SRS-REQ-API-002:** JWT tokens SHALL include:
```json
{
  "sub": "user-id",
  "type": "caretaker|observer",
  "exp": "expiration-timestamp",
  "iat": "issued-at-timestamp"
}
```

#### 5.3.2 Caretaker API Endpoints

**SRS-REQ-API-003:** Patient Management:
```
POST   /api/v1/patients                    # Create patient profile
GET    /api/v1/patients                    # List my patients
GET    /api/v1/patients/{id}               # Get patient details
PUT    /api/v1/patients/{id}               # Update patient
DELETE /api/v1/patients/{id}               # Archive/delete patient
GET    /api/v1/patients/{id}/incidents     # Get patient incidents
```

**SRS-REQ-API-004:** Observer Management:
```
POST   /api/v1/patients/{patientId}/observers/invite  # Invite observer
GET    /api/v1/patients/{patientId}/observers         # List patient observers
PUT    /api/v1/patients/{patientId}/observers/{id}    # Update observer assignment
DELETE /api/v1/patients/{patientId}/observers/{id}    # Remove observer
POST   /api/v1/invitations/{id}/resend                # Resend invitation
DELETE /api/v1/invitations/{id}                       # Cancel invitation
GET    /api/v1/invitations                            # List pending invitations
```

**SRS-REQ-API-005:** Escalation Policy Management:
```
POST   /api/v1/patients/{patientId}/escalation-policy    # Create/update policy
GET    /api/v1/patients/{patientId}/escalation-policy    # Get policy
```

**SRS-REQ-API-006:** Alert Management:
```
POST   /api/v1/patients/{patientId}/alerts        # Create alert
GET    /api/v1/patients/{patientId}/alerts        # List patient alerts
GET    /api/v1/alerts/{id}                        # Get alert details
PUT    /api/v1/alerts/{id}                        # Update alert
DELETE /api/v1/alerts/{id}                        # Cancel alert
POST   /api/v1/alerts/{id}/pause                  # Pause alert
POST   /api/v1/alerts/{id}/resume                 # Resume alert
POST   /api/v1/alerts/{id}/test                   # Test alert (manual trigger)
```

**SRS-REQ-API-007:** Incident Management:
```
GET    /api/v1/incidents                          # List all my incidents
GET    /api/v1/patients/{patientId}/incidents     # List patient incidents
GET    /api/v1/incidents/{id}                     # Get incident details
POST   /api/v1/incidents/{id}/resolve             # Resolve incident
POST   /api/v1/incidents/{id}/reassign            # Reassign incident
POST   /api/v1/incidents/{id}/notes               # Add note to incident
```

#### 5.3.3 Observer API Endpoints

**SRS-REQ-API-008:** Onboarding:
```
GET    /api/v1/invitations/{token}                # Get invitation details
POST   /api/v1/invitations/{token}/accept         # Accept invitation & create account
POST   /api/v1/auth/verify-phone                  # Verify phone via SMS code
POST   /api/v1/auth/verify-email                  # Verify email
```

**SRS-REQ-API-009:** Observer Operations:
```
GET    /api/v1/me                                 # Get my profile
PUT    /api/v1/me                                 # Update profile
PUT    /api/v1/me/preferences                     # Update notification preferences
GET    /api/v1/me/patients                        # List my assigned patients
GET    /api/v1/me/incidents                       # List my active incidents
POST   /api/v1/incidents/{id}/acknowledge         # Acknowledge incident
POST   /api/v1/incidents/{id}/resolve             # Resolve incident
GET    /api/v1/incidents/{id}                     # Get incident details
```

#### 5.3.4 Public/Unauthenticated Endpoints

**SRS-REQ-API-010:** Acknowledgment Links (token-based):
```
GET    /ack/{token}                               # Quick acknowledge page
POST   /ack/{token}                               # Submit acknowledgment
GET    /resolve/{token}                           # Quick resolve page
POST   /resolve/{token}                           # Submit resolution
```

#### 5.3.5 Webhook Endpoints

**SRS-REQ-API-011:** Incoming Webhooks:
```
POST   /webhooks/alert-trigger                    # Receive scheduled alert trigger
POST   /webhooks/sms-status                       # Receive SMS delivery status
POST   /webhooks/email-status                     # Receive email delivery status (optional)
```

## 6. System Constraints

### 6.1 MVP Constraints
**SRS-CON-007:** Alert messages limited to 1000 characters.

**SRS-CON-008:** SMS notifications limited to 1600 characters (multiple segments).

---

## 7. User Flows

### 7.1 Caretaker: Create Patient & Add Observer

```
1. Caretaker logs in
2. Navigate to "Add Patient"
3. Fill patient form (name, description, photo)
4. Submit → Patient created
5. Click "Add Observer" for patient
6. Enter observer details (name, email, phone, role)
7. Submit invitation → SMS/Email sent to observer
8. System shows "Invitation pending" status
```

### 7.2 Observer: Complete Onboarding

```
1. Observer receives SMS: "You've been invited to monitor Sarah Johnson..."
2. Click invitation link
3. **Landing page:** See patient name, caretaker name, role
4. Accept terms & privacy policy
5. **Account creation:**
   - Confirm/edit phone number
   - Enter email
   - Create password
6. Verify phone via SMS code
7. Verify email via confirmation link
8. **Notification preferences:**
   - Choose methods (SMS/Email)
   - Set quiet hours
   - Configure urgency overrides
9. **Patient context:** Review patient info & escalation policy
10. Test notification (optional)
11. Complete onboarding → Account active
12. Redirected to dashboard
```

### 7.3 Caretaker: Create Alert for Patient

```
1. Navigate to patient profile
2. Click "Create Alert"
3. Fill alert form:
   - Alert name: "Morning Medication Check"
   - Message: "Confirm 10 units Humalog administered"
   - Severity: High
   - Schedule: "Every day at 8:00 AM" 
     (UI generates RRULE: FREQ=DAILY;BYHOUR=8;BYMINUTE=0)
4. Review escalation policy (uses patient's default)
5. Set auto-resolve: 30 minutes
6. Preview notification
7. Submit → Alert created
8. System converts RRULE to webhook service format and creates schedule
9. Alert shows "Active" status with next trigger time
```

### 7.4 System: Alert Fires & Escalates

```
1. Webhook service triggers at scheduled time (8:00 AM)
2. System receives webhook POST
3. System creates incident (ID #1234)
4. System loads patient escalation policy
5. **Level 1 notification:**
   - Send SMS to Primary Observer (John)
   - Include: patient name, message, acknowledge link
   - Start 5-minute timeout timer
6. [5 minutes pass, no acknowledgment]
7. **Escalate to Level 2:**
   - Log escalation action
   - Send SMS to Secondary Observer (Mary)
   - Start 10-minute timeout timer
8. [Mary clicks acknowledge link]
9. **Incident acknowledged:**
   - Stop escalation timer
   - Cancel pending Level 3 notifications
   - Mark incident assigned to Mary
   - Send confirmation SMS to Mary
   - Notify caretaker (optional)
   - Start 30-minute auto-resolve timer
10. [Mary completes patient care, clicks resolve link]
11. **Incident resolved:**
    - Mark incident resolved by Mary
    - Log resolution timestamp
    - Send confirmation
    - Close incident
```

### 7.5 Observer: Acknowledge Via SMS Link

```
1. Observer receives SMS notification
2. Click "Acknowledge" link
3. Browser opens acknowledgment page:
   - "Incident #1234: Morning Medication Check"
   - Patient: Sarah Johnson
   - "Confirm you are responding to this alert?"
4. Observer clicks "Yes, I'll handle this"
5. System:
   - Validates token
   - Marks incident acknowledged
   - Assigns incident to observer
6. Confirmation page: "You're assigned to this incident"
7. Observer sees "Resolve" button
8. [After completing care, observer clicks Resolve]
9. Incident marked resolved
10. Confirmation: "Incident #1234 resolved"
```

---

## 8. Testing Requirements

No testing

## 12. Dependencies and Assumptions

### 12.1 Assumptions

- Caretakers have valid email and phone numbers for observers
- Observers have smartphones capable of receiving SMS
- Observers consent to receiving notifications
- iCalendar RRULE expressions are validated before submission
- Phone numbers are collected with proper consent
- System deployed in cloud environment (AWS/GCP/Azure)
- SMS gateway has sufficient rate limits for expected load
- Webhook scheduling service supports cron expressions OR system can calculate next occurrences from RRULE

---

## 13. Glossary

**Alert:** A scheduled check for a patient that triggers incidents  
**Incident:** A specific occurrence of an alert requiring response (PagerDuty concept)  
**Caretaker:** User who manages patients and creates alerts  
**Observer:** User who receives notifications and responds to incidents  
**Patient:** Individual being monitored, with dedicated observers and alerts  
**Escalation Policy:** Rules defining who to notify and when (PagerDuty concept)  
**Escalation Level:** Step in escalation policy with specific observers and timeout  
**Acknowledge:** Observer confirms receipt of incident (stops escalation)  
**Resolve:** Observer or caretaker marks incident as complete  
**Webhook:** HTTP callback triggered by external scheduling service  
**iCalendar RRULE:** Recurrence Rule format from RFC 5545 for defining recurring events  
**RRULE:** Short form of "Recurrence Rule" - the standardized syntax for expressing recurring schedules  
**Onboarding:** Process of accepting invitation and setting up observer account  
**Invitation Token:** Unique time-limited code for accepting observer invitation  
**Acknowledgment Token:** Unique link allowing quick incident acknowledgment  
**Quiet Hours:** Time period when observer prefers not to receive notifications (overridable)

---

## 15. Appendices

### Appendix A: PagerDuty Feature Comparison

| PagerDuty Feature | Our System | MVP Status |
|-------------------|------------|------------|
| Services | Patient Profiles | ✅ Included |
| Incidents | Alert Incidents | ✅ Included |
| Escalation Policies | Patient Escalation Policies | ✅ Included |
| On-Call Schedules | Alert Schedules (iCal) | ✅ Included |
| Users | Observers | ✅ Included |
| Acknowledge | Acknowledge Incident | ✅ Included |
| Resolve | Resolve Incident | ✅ Included |
| Notification Rules | Observer Preferences | ✅ Basic |
| Alert Grouping | N/A | ❌ Out of scope |
| Event Rules | N/A | ❌ Out of scope |
| Schedule Overrides | N/A | ❌ Out of scope |
| Team Dashboard | N/A | ❌ API only |
| Analytics | N/A | ❌ Future |
| Status Page | N/A | ❌ Future |
| Slack Integration | N/A | ❌ Future |

### Appendix B: iCalendar RRULE Examples

The system uses iCalendar Recurrence Rule (RRULE) format as defined in RFC 5545. Below are common patterns:

**Note:** RRULE syntax does not include the "RRULE:" prefix when stored in the database or sent via API. The format is just the rule itself (e.g., `FREQ=DAILY;BYHOUR=8;BYMINUTE=0`).

```
Daily at 8am:
FREQ=DAILY;BYHOUR=8;BYMINUTE=0

Every Monday, Wednesday, Friday at 2pm:
FREQ=WEEKLY;BYDAY=MO,WE,FR;BYHOUR=14;BYMINUTE=0

Every 4 hours:
FREQ=HOURLY;INTERVAL=4

First day of every month at 9am:
FREQ=MONTHLY;BYMONTHDAY=1;BYHOUR=9;BYMINUTE=0

Every weekday at 6pm:
FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR;BYHOUR=18;BYMINUTE=0

Every 2 weeks on Monday at 10am:
FREQ=WEEKLY;INTERVAL=2;BYDAY=MO;BYHOUR=10;BYMINUTE=0

Last day of every month at 5pm:
FREQ=MONTHLY;BYMONTHDAY=-1;BYHOUR=17;BYMINUTE=0
```

**RRULE to Cron Conversion Notes:**
- Simple daily/weekly/monthly patterns convert cleanly to cron
- Complex patterns (e.g., "last weekday of month") may require calculated one-time schedules
- System should validate RRULE compatibility with chosen webhook service

### Appendix C: SMS Notification Templates

**Incident Triggered (Level 1):**
```
🔴 HIGH: Patient Sarah Johnson
Morning Medication Check

Confirm 10 units Humalog administered.

Incident #1234 | Level 1
Ack: https://app.link/ack/xyz123
View: https://app.link/i/1234
```

**Escalation (Level 2):**
```
⚠️ ESCALATED: Patient Sarah Johnson
Morning Medication Check

No response from primary observer.
Confirm 10 units Humalog administered.

Incident #1234 | Level 2
Ack: https://app.link/ack/abc456
View: https://app.link/i/1234
```

**Acknowledgment Confirmation:**
```
✅ Incident #1234 Acknowledged

You are now assigned to:
Patient Sarah Johnson - Morning Medication Check

Resolve when complete:
https://app.link/resolve/xyz123
```

**Resolution Confirmation:**
```
✅ Incident #1234 Resolved

Thank you for responding to Sarah Johnson's alert.
```

**Invitation SMS:**
```
You've been invited to monitor patient Sarah Johnson by [Caretaker Name].

Accept invitation:
https://app.link/invite/token123

Expires in 7 days
```

---

**End of Document**
