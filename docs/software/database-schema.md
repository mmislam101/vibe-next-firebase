# Database Schema Documentation

**Database:** Cloud Firestore  

---

## 1. Overview

This document defines the Firestore database schema. The schema uses Firestore's document-based NoSQL structure to store user profiles and authentication data.

### 1.1 Design Principles

- **Security:** Structure data to support Firestore security rules
- **Simplicity:** Keep schema minimal and focused on user management
- **Real-time:** Structure for efficient real-time listeners for user data updates

### 1.2 Layout and Navigation Support

The database schema supports the application's layout structure through user profile fields:

**Top Navigation Bar:**
- `caretakers.photoUrl` and `observers.photoUrl` - Google profile pictures displayed in top-right
- `caretakers.firstName/lastName` and `observers.firstName/lastName` - User display names

**User Profile:**
- User information automatically populated from Google account during authentication
- Profile data available for display throughout the application

All layout components can use real-time Firestore listeners on these fields to provide immediate updates when data changes.

---

## 2. User Profile Collections

**[Implements: SRD-REQ-DB-001, SRS-REQ-SEC-001, SRS-REQ-SEC-003]**

### 2.1 Caretakers Collection

**[Implements: SRD-REQ-DB-001, SRS-REQ-SEC-001, SRS-REQ-SEC-003]**

**Collection Path:** `caretakers/{caretakerId}`

**Purpose:** Store caretaker (account owner) user accounts and profile information.

**Document Structure:**
```typescript
interface Caretaker {
  id: string;                    // Document ID (matches Firebase Auth UID)
  email: string;                 // Unique email address (from Google account)
  firstName: string;             // From Google account
  lastName: string;              // From Google account
  googleUid: string;             // Google user ID (matches Firebase Auth UID)
  photoUrl?: string;             // Google profile photo URL
  accountStatus: 'active' | 'suspended' | 'deleted';
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastLoginAt?: Timestamp;
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

**Notes:**
- Email, first name, last name, and photo URL are automatically populated from Google account during sign-in
- Google UID is used to link Firebase Auth with Firestore user profile
- Account status tracks user lifecycle (active, suspended, deleted)


---

## 3. Data Validation

### 3.1 Type Validation

All data SHALL be validated using TypeScript interfaces in Cloud Functions before writing to Firestore.

### 3.2 Business Rule Validation

- User IDs must match Firebase Auth UIDs
- Email addresses must be unique per collection
- Google UID must be present for all users
- Account status must be a valid enum value

---

## 4. Data Migration

### 4.1 Initial Setup

1. Create collections with initial security rules
2. Create composite indexes for observers collection
3. Set up Cloud Functions triggers for automatic user profile creation on first sign-in

### 4.2 User Profile Creation Flow

When a user signs in with Google for the first time:
1. Firebase Authentication creates an authenticated user
2. Cloud Function trigger (onUserCreate) detects new user
3. Function extracts profile data from Google account (email, name, photo)
4. Function creates corresponding document in `caretakers` or `observers` collection
5. User profile is now available in Firestore for application use

---

**End of Document**

