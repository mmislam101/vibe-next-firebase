# Family Pager

Patient Alert Notification System - A PagerDuty-style alert management platform for healthcare scenarios.

## Tech Stack

- **Frontend:** Next.js 15 (App Router), React 19, Tailwind CSS 4
- **Backend:** Firebase Cloud Functions (TypeScript)
- **Database:** Cloud Firestore
- **Authentication:** Firebase Authentication (Google Sign-In only)
- **Hosting:** Firebase Hosting
- **Testing:** Jest, ts-jest

## Project Structure

```
family-pager/
├── app/                    # Next.js app directory
├── components/             # React components (to be created)
├── functions/              # Firebase Cloud Functions
│   ├── src/               # TypeScript source
│   │   ├── index.ts       # API entry point
│   │   ├── user/          # User profile endpoints
│   │   └── shared/        # Shared utilities (auth, validators)
│   └── lib/               # Compiled JavaScript
├── docs/                   # Documentation
│   ├── system/            # System requirements
│   └── software/          # Software specs and implementation docs
├── firebase.json           # Firebase configuration
├── firestore.rules         # Firestore security rules
└── firestore.indexes.json  # Firestore indexes
```

## Phase 1: Completed ✅

### Infrastructure Setup
- ✅ Firebase project configuration (Hosting, Firestore, Functions, Authentication)
- ✅ Firestore security rules (Google Auth only)
- ✅ Firestore composite indexes
- ✅ Firebase Functions with TypeScript

### API Implementation (BDD/TDD)
- ✅ Authentication middleware (Google Sign-In only)
- ✅ User profile endpoints (`GET /api/v1/me`, `PUT /api/v1/me`)
- ✅ Comprehensive test coverage (17 test cases)
- ✅ Tests written BEFORE implementation

### Frontend Structure
- ✅ Next.js 15 with App Router
- ✅ Tailwind CSS 4 configuration
- ✅ Basic landing page
- ⏳ Firebase client configuration (pending)
- ⏳ Google Sign-In integration (pending)
- ⏳ Layout components (pending)

## Getting Started

### Prerequisites

- Node.js 20+
- npm or yarn
- Firebase CLI (`npm install -g firebase-tools`)
- Firebase project with Google Authentication enabled

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd family-pager
   ```

2. **Install dependencies**
   ```bash
   # Root dependencies (Next.js) + Functions dependencies
   npm install
   ```

3. **Configure Firebase**
   ```bash
   # Login to Firebase
   firebase login
   
   # Set your Firebase project
   firebase use family-pager
   ```

4. **Set up environment variables**
   ```bash
   # Copy the example file
   cp .env.local.example .env.local
   
   # Edit .env.local with your Firebase project credentials
   # Get these from Firebase Console > Project Settings
   ```

5. **Enable Google Authentication**
   - Go to Firebase Console > Authentication
   - Enable "Google" sign-in provider
   - Add authorized domains for your app

### Development

#### Run Frontend (Next.js)
```bash
npm run dev
# Open http://localhost:3000
```

#### Run Firebase Emulators
```bash
npm run emulators
# Functions: http://localhost:5001
# Firestore: http://localhost:8080
# Auth: http://localhost:9099
# Emulator UI: http://localhost:4000
```

#### Run Functions Locally
```bash
cd functions
npm run serve
# Functions available at http://localhost:5001/family-pager/us-central1/api
```

### Testing

#### Run API Tests
```bash
cd functions
npm test

# Watch mode
npm run test:watch

# With coverage
npm test -- --coverage
```

### Deployment

#### Deploy Everything
```bash
npm run deploy
# Deploys hosting, functions, and firestore rules
```

#### Deploy Functions Only
```bash
cd functions
npm run deploy
```

#### Deploy Firestore Rules & Indexes
```bash
npm run deploy:firestore:config
```

## API Documentation

### Base URL

- **Local:** `http://localhost:5001/family-pager/us-central1/api`
- **Production:** `https://us-central1-family-pager.cloudfunctions.net/api`

### Endpoints

#### GET /api/v1/me
Get authenticated user profile.

**Headers:**
```
Authorization: Bearer <firebase-id-token>
```

**Response:**
```json
{
  "data": {
    "id": "user-id",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "caretaker",
    "photoUrl": "https://...",
    "accountStatus": "active"
  }
}
```

#### PUT /api/v1/me
Update user profile.

**Headers:**
```
Authorization: Bearer <firebase-id-token>
```

**Request Body:**
```json
{
  "firstName": "John Updated",
  "lastName": "Doe Updated"
}
```

See `docs/software/PHASE1_API_IMPLEMENTATION.md` for detailed API documentation.

## Security

- **Authentication:** Google Sign-In only (enforced at multiple layers)
- **Authorization:** Role-based access control via Firestore rules
- **Validation:** Input validation for all user data
- **Encryption:** HTTPS/TLS for all communications

## Development Workflow

### Waterfall Process

1. **Developer updates:** `docs/system/system-requirements.md` (SRS)
2. **Agent generates:** `docs/system/software-requirements.md` (SRD)
3. **Agent generates:** Software specs in `docs/software/`
4. **Agent implements API:** Following BDD principles (tests first)
5. **Developer reviews:** Tests and implementation
6. **Agent implements UI:** Following frontend requirements

### BDD Principles for API

1. ✅ Write tests first
2. ✅ Tests define expected behavior
3. ✅ Implement to make tests pass
4. ✅ Refactor with confidence

## Next Steps

### Phase 2: Patient Management
- Patient CRUD operations
- Patient list and detail pages
- Patient profile management

### Phase 3: Observer Management
- Observer invitation system
- Observer onboarding flow
- Observer assignment to patients

See `docs/system/phases.md` for complete roadmap.

## Documentation

- `docs/system/system-requirements.md` - System Requirements Specification (SRS)
- `docs/system/software-requirements.md` - Software Requirements Document (SRD)
- `docs/software/api-specification.md` - API Specification
- `docs/software/database-schema.md` - Firestore Schema
- `docs/software/frontend-requirements.md` - Frontend Requirements
- `docs/software/PHASE1_API_IMPLEMENTATION.md` - Phase 1 Implementation Details

## License

Private project - All rights reserved.

## Support

For questions or issues, please refer to the documentation in the `docs/` directory.
