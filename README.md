# Vibe Coding Template

This template is designed for prototyping an MVP of a web app product by Vibe Coding using Cursor, hosted on Google Firebase.
The template follows the Waterfall Methodology to help manage the relationship between the developer and the AI.

```
                           SRS
                            |
                ___________SRD_________________________________
               /            |              \                   \
 SDD Docs:  API Doc    Database Doc    Frontend Doc    Testing Behavior Doc
               |            |                |                    |
     Code:  Functions   Firestore         Next.js            API Tests
```

## How to use this Template
- AGENTS.md file
   - Contains instructions on how the Agents should implement the Waterfall
- SRS Doc (`docs/system/system-requirements.md`)
   - Product level requirements and specifications
- SRD Doc (`docs/system/software-requirements.md`)
   - Software architecture
   - Software requirements
- SDD Docs (`docs/software`)
   - API Doc (`docs/software/api-specification.md`)
   - Database Doc (`docs/software/database-schema.md`)
   - Frontend Doc (`docs/software/frontend-requirements.md`)
   - Testing Behavior Doc (`docs/software/testing-behaviors.md`)
- PHASES.md
   - Use to help stage changes
   - Already seeded with Auth flow and basic layout
- Agent Roles
   - Use cursor commands, e.g. `/as-api-agent please update...`
   - [Doc Agent](.cursor/commands/as-docs-agent.md)
      - Use for updating the docs
   - [API Agent](.cursor/commands/as-api-agent.md)
      - Use for any backend related changes
   - [Frontend Agent](.cursor/commands/as-frontend-agent.md)
      - Use for any frontend related changes
   - [Debugger Agent](.cursor/commands/as-frontend-agent.md)
      - Use for debugging the app

1. Create the Firebase Account and Project in `https://console.firebase.google.com`
2. Have Agent implement Phase 1 from the `PHASES.md` doc
3. Test locally successfully
4. Deploy to Firebase successfully
5. Fill out the rest of the SRS document with full product details
6. Have AI generate the rest of the documents and code changes following the Waterfall methodology

## Cursor Commands
Use these to help during development
- Run with `/failingtests` when API tests are failing
- Use `/feedback` as a prefix and then provide Github Copilot feedback from PR
- Use `/runninglocally` as a prefix when coming across problems while testing the app locally
- Use `/as-...` when wanting the agent to assume a particular role

**Delete above, below is the Readme**

# [Insert App Name Here]

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
│   ├── system/            # System requirements (SRS, SRD)
│   └── software/          # Software specs (API, DB, Frontend, Testing)
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
- `docs/software/testing-behaviors.md` - Testing Behavior Document

## License

Private project - All rights reserved.
