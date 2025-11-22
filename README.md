# Vibe coding Template

Template to start your Vibe coding

## Tech Stack

- **Frontend:** Next.js (App Router), React, Tailwind CSS
- **Backend:** Firebase Cloud Functions (TypeScript)
- **Database:** Cloud Firestore
- **Authentication:** Firebase Authentication (Google Sign-In only)
- **Hosting:** Firebase Hosting
- **Testing:** Jest, ts-jest

## Project Structure

```
/
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

## Getting Started

### Prerequisites

- Node.js
- npm
- Firebase CLI (`npm install -g firebase-tools`)
- Firebase project with Google Authentication enabled

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   ```

2. **Install dependencies**
   ```bash
   # Root dependencies (Next.js) + Functions dependencies
   npm run functions:install
   npm install
   ```

3. **Configure Firebase**
   ```bash
   # Login to Firebase
   firebase login
   
   # Set your Firebase project
   firebase init
   ```

   Add Hosting, Firestore, Authentication, Functions

4. **Set up environment variables**
   1. Create `.env.local` with your Firebase project credentials
   2. Get these from Firebase Console > Project Settings
   

5. **Enable Google Authentication**
   - Go to Firebase Console > Authentication
   - Enable "Google" sign-in provider
   - Add authorized domains for your app

### Development

#### Run Backend
```bash
npm run functions
```

#### Run Frontend (Next.js)
```bash
npm run dev
```
Open http://localhost:3000

### Testing

#### Run API Tests
```bash
npm run functions:test

# With coverage
npm test -- --coverage
```

### Deployment

#### Deploy Firestore Rules & Indexes
```bash
npm run deploy:firestore:config
```

#### Deploy App
```bash
npm run functions:deploy # Deploys hosting, functions, and firestore rules
npm run deploy
```

## Documentation

- `docs/system/system-requirements.md` - System Requirements Specification (SRS)
- `docs/system/software-requirements.md` - Software Requirements Document (SRD)
- `docs/software/api-specification.md` - API Specification
- `docs/software/database-schema.md` - Firestore Schema
- `docs/software/frontend-requirements.md` - Frontend Requirements

## License

Private project - All rights reserved.
