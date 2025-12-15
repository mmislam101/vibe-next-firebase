# Systems Requirements Document

**Technology Stack:** Next.js, Firebase Hosting, Firestore, Firebase Cloud Functions, Tailwind CSS

---

## 1. Executive Summary

This document defines the Systems Requirements for implementing the [Insert Project Name] as a Next.js application hosted on Firebase.

### 1.1 Technology Stack

- **Frontend Framework:** Next.js (App Router) with React
- **Hosting:** Firebase Hosting (static export)
- **Database:** Cloud Firestore
- **API Layer:** Firebase Cloud Functions (TypeScript)
- **Authentication:** Firebase Authentication (Google Sign-In)
- **UI Framework:** Tailwind CSS

### 1.2 Key Architectural Decisions

- **Static Site Generation:** Next.js static export for optimal Firebase Hosting performance
- **Serverless API:** All business logic in Firebase Cloud Functions (basic structure)
- **Type-Safe Development:** Full TypeScript implementation across frontend and backend
- **Component-Based UI:** Reusable React components with Tailwind CSS

---

## 2. System Architecture Overview

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Firebase Hosting                          │
│              (Next.js Static Export)                         │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │Landing Page  │  │  Dashboard    │                         │
│  │(Unauthenti-  │  │ (Authenticated)│                        │
│  │    cated)    │  │               │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ HTTPS API Calls (Future)
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              Firebase Cloud Functions                        │
│  ┌──────────────┐                                            │
│  │  Basic API   │  (Placeholder for future endpoints)        │
│  │  Structure   │                                            │
│  └──────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
                  ┌──────────────┐
                  │  Firestore   │
                  │   Database   │
                  └──────────────┘
```

### 2.2 Core Components

1. **Next.js Frontend Application**
   - Landing page for unauthenticated users
   - Google Sign-In authentication flow
   - Authenticated layout with navigation structure
   - Responsive design (desktop, tablet, mobile)
   - Loading states and UI components

2. **Firebase Cloud Functions (API Layer)**
   - Basic API structure and routing setup
   - Authentication middleware
   - Placeholder for future endpoints

3. **Firestore Database**
   - User accounts (basic storage)
   - Security rules foundation

4. **External Services**
   - Firebase Authentication (Google OAuth)

---

## 3. Frontend Application Requirements

### 3.1 Next.js Application Structure

**SRD-REQ-FRONT-001:** [Implements: PRD-REQ-UI-003, PRD-REQ-UI-004, PRD-REQ-UI-017] The application SHALL follow Next.js 15 App Router structure:

**App Directory Structure:**
- `layout.tsx` - Root layout with providers
- `page.tsx` - Landing page (unauthenticated)
- `globals.css` - Tailwind CSS imports
- `components/` - Reusable UI components
  - `ui/` - Base UI components (buttons, inputs, etc.)
  - `layout/` - Layout components (TopNavBar, LeftSideMenu, MobileNavMenu, UserDropdownMenu, LoadingOverlay, NavLink)
  - `auth/` - Authentication components (GoogleSignInButton)
- `context/` - React context providers (AuthContext, LoadingContext)
- `hooks/` - Custom React hooks (useAuth, useLoadingState, useMobileMenu)
- `(authenticated)/` - Route group for authenticated pages
  - `layout.tsx` - Authenticated layout wrapper
  - `dashboard/page.tsx` - Dashboard placeholder
- `lib/` - Utilities (firebase.ts, utils.ts)
- `middleware.ts` - Route protection middleware (development only)

### 3.2 UI Framework Requirements

**SRD-REQ-FRONT-002:** [Implements: PRD-REQ-UI-002, PRD-REQ-UI-018] The application SHALL use Tailwind CSS for styling:

- **Design System:** Modern, clean healthcare-focused UI
- **Responsive Design:** Mobile-first approach, optimized for mobile devices
- **Accessibility:** WCAG AA compliance
- **Component Library:** Custom components built on Tailwind utilities
- **Dark Mode:** Optional (future enhancement)

**SRD-REQ-FRONT-003:** [Implements: PRD-REQ-UI-003, PRD-REQ-UI-013, PRD-REQ-UI-016] Key UI components SHALL include:

- **Button:** Primary, secondary, and danger variants
- **Input:** Text input with validation states
- **Card:** Container component for content sections
- **Loading Overlay:** Full-screen loading indicator
- **Navigation Components:** NavLink, LeftSideMenu, TopNavBar, MobileNavMenu

### 3.3 State Management

**SRD-REQ-FRONT-004:** [Implements: PRD-REQ-UI-003, PRD-REQ-UI-017] The application SHALL use React Context API for state management:

- **AuthContext:** Firebase Authentication state and user profile
- **LoadingContext:** Global loading state management

### 3.4 Routing and Navigation

**SRD-REQ-FRONT-006:** [Implements: PRD-REQ-SEC-001] The application SHALL implement protected routes:

- **Public Routes:** Landing page (/)
- **Authenticated Routes:** Dashboard (/dashboard)

**SRD-REQ-FRONT-007:** [Implements: PRD-REQ-SEC-001] Route protection SHALL be implemented via middleware:

```typescript
// middleware.ts
export function middleware(request: NextRequest) {
  const token = request.cookies.get('authToken');
  const path = request.nextUrl.pathname;
  
  // Check authentication and role-based access
  if (requiresAuth(path) && !token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}
```

### 3.5 Application Layout Structure

#### 3.5.1 Landing Page (Unauthenticated State)

**SRD-REQ-FRONT-008:** [Implements: PRD-REQ-UI-001] The landing page SHALL be implemented as:

- Centered layout with full viewport height
- Gradient background (blue to indigo)
- Card container with white background and shadow
- Application title: "Family Pager"
- Descriptive tagline: "Patient alert notification system for caretakers"
- Google Sign-In button as primary CTA
- Helper text explaining sign-in requirement

**SRD-REQ-FRONT-009:** [Implements: PRD-REQ-UI-001, PRD-REQ-UI-002] The landing page SHALL:
- Display prominently centered on the viewport
- Show application name and value proposition
- Feature a Google Sign-In button as primary call-to-action
- Use Firebase Authentication for Google OAuth
- Be responsive across desktop (1024px+), tablet (768-1023px), and mobile (<768px)
- Not display any authenticated navigation elements

#### 3.5.2 Authenticated Layout Structure

**SRD-REQ-FRONT-010:** [Implements: PRD-REQ-UI-003, PRD-REQ-UI-004, PRD-REQ-UI-006, PRD-REQ-UI-010, PRD-REQ-UI-017] Once authenticated, the application SHALL use a root layout component:

- Full-height flex layout with column direction
- Top navigation bar spanning full width
- Horizontal flex container for sidebar and main content
- Left sidebar menu (desktop/tablet only)
- Main content area with scrollable overflow and padding
- Mobile navigation menu (slide-in drawer)
- Global loading overlay component

#### 3.5.3 Left-Side Menu Pane (Desktop/Tablet)

**SRD-REQ-FRONT-011:** [Implements: PRD-REQ-UI-004, PRD-REQ-UI-005] The left-side menu SHALL be implemented with:

- Client component with pathname and auth hooks
- Hidden on mobile, visible on tablet/desktop (≥768px)
- Fixed width (256px) sidebar with white background
- Navigation section with vertical spacing
- NavLink components with icons and active state
- Current pathname comparison for active highlighting
- Placeholder for future navigation items

**SRD-REQ-FRONT-012:** [Implements: PRD-REQ-UI-004, PRD-REQ-UI-005, PRD-REQ-UI-017] The left-side menu SHALL:
- Be visible only on viewports ≥768px width (hidden on mobile)
- Have a fixed width between 200-280px (recommend 256px/64rem)
- Remain fixed/persistent during navigation
- Highlight the currently active navigation item with visual indicator
- Use clear iconography for each menu item (from @heroicons/react)
- Maintain selection state across page navigation
- Use semantic HTML (`<nav>`, `<aside>`)
- Include placeholder structure for future navigation items

#### 3.5.4 Top Navigation Bar

**SRD-REQ-FRONT-013:** [Implements: PRD-REQ-UI-006, PRD-REQ-UI-007, PRD-REQ-UI-010] The top navigation bar SHALL be implemented as:

- Client component with auth hook and dropdown state management
- Sticky positioning at top with high z-index
- Full-width header with fixed height (64px)
- Left section with hamburger menu (mobile only) and logo/brand link
- Right section with user profile button
- Profile picture from Google account (circular avatar)
- User display name (visible on desktop/tablet only)
- Dropdown toggle indicator icon
- Conditional rendering of dropdown menu based on state
- Accessibility labels for interactive elements

**SRD-REQ-FRONT-014:** [Implements: PRD-REQ-UI-006, PRD-REQ-UI-007, PRD-REQ-UI-010, PRD-REQ-UI-018] The top navigation bar SHALL:
- Span the full width of the viewport
- Be fixed at the top with `position: sticky` or `fixed`
- Have a minimum height of 56px (recommend 64px/4rem)
- Display application logo/name on the left
- Display hamburger menu icon (☰) on mobile (<768px) in top-left
- Display user profile section on the top-right
- Show Google profile picture as circular avatar (32px/8rem diameter)
- Show user's display name from Google account on larger viewports (≥768px)
- Include visual dropdown indicator (chevron/arrow icon)
- Have adequate contrast for accessibility (WCAG 2.1 AA)

**SRD-REQ-FRONT-015:** [Implements: PRD-REQ-UI-008, PRD-REQ-UI-009] The user profile dropdown SHALL display:

- Client component with auth hook for user data and logout
- Absolute positioning relative to parent (right-aligned)
- Dropdown card with shadow and border
- User info section displaying name and email
- Border separator after user info
- Placeholder section for future menu items
- Divider line before logout
- Logout menu item with icon and distinct styling (red text)
- Menu role for accessibility

**SRD-REQ-FRONT-016:** [Implements: PRD-REQ-UI-008, PRD-REQ-UI-009, PRD-REQ-UI-017] The dropdown menu SHALL:
- Appear when profile icon is clicked
- Display user's full name and email at the top
- Show Logout/Sign Out option with clear visual separation (divider line)
- Position Logout as the last item
- Include icon for Logout menu item
- Close when clicking outside or selecting an option
- Use proper ARIA attributes for accessibility
- Implement click-outside detection or focus trap
- Include placeholder for future menu items

#### 3.5.5 Mobile Responsive Layout

**SRD-REQ-FRONT-017:** [Implements: PRD-REQ-UI-010, PRD-REQ-UI-011, PRD-REQ-UI-012] The mobile navigation menu SHALL be implemented as:

- Client component with mobile menu state hook
- Semi-transparent overlay (black 50% opacity) covering entire screen
- Click overlay to dismiss menu
- Slide-in drawer from left side with fixed positioning
- 256px width drawer with white background
- CSS transform animation (300ms) for smooth slide
- Header with menu title and close button
- Navigation section matching desktop menu items
- Hidden on desktop/tablet (≥768px)
- Higher z-index for drawer (50) than overlay (40)

**SRD-REQ-FRONT-018:** [Implements: PRD-REQ-UI-010, PRD-REQ-UI-011, PRD-REQ-UI-012, PRD-REQ-UI-018] Mobile navigation (<768px) SHALL:
- Hide the left-side menu pane
- Display hamburger menu icon (☰) in top-left of navigation bar
- Maintain user profile icon in top-right
- Slide in menu drawer from the left when hamburger is tapped
- Display same navigation items as desktop menu
- Include close button (✕) in menu header
- Support tap-outside to dismiss (semi-transparent overlay)
- Prevent body scroll when menu is open (using `overflow: hidden` or focus trap)
- Animate smoothly with CSS transitions (300ms duration recommended)
- Close automatically after navigation selection
- Use full-height drawer (0 to 100vh)
- Set menu width to 256px (64rem) with max 80vw

#### 3.5.6 Loading Indicator UX

**SRD-REQ-FRONT-019:** [Implements: PRD-REQ-UI-013, PRD-REQ-UI-014, PRD-REQ-UI-015] The global loading overlay SHALL be implemented as:

- Client component with loading state hook
- Return null when not loading (conditional rendering)
- Full-screen fixed overlay with very high z-index (9999)
- Semi-transparent black background (50% opacity)
- Centered content with flexbox
- White card container with shadow and padding
- Animated spinner with circular border animation
- Optional loading message displayed below spinner
- ARIA attributes for screen reader accessibility

**SRD-REQ-FRONT-020:** [Implements: PRD-REQ-UI-013, PRD-REQ-UI-014, PRD-REQ-UI-015] The loading indicator SHALL:
- Block the entire screen to prevent user interaction
- Use semi-transparent black overlay (rgba(0, 0, 0, 0.5) or similar)
- Display centered spinner/loading animation
- Show loading text message when appropriate (e.g., "Loading patients...", "Saving alert...")
- Prevent clicking through to underlying content (z-index: 9999)
- Appear within 200ms of operation start
- Dismiss immediately when operation completes or fails
- Show for minimum 300ms to avoid flashing on fast operations
- Never block critical error messages or alerts
- Use proper ARIA attributes for screen readers

**SRD-REQ-FRONT-021:** [Implements: PRD-REQ-UI-013, PRD-REQ-UI-015] The loading state management SHALL be centralized:

- TypeScript interface defining loading context methods and state
- Provider component managing loading state
- State for loading flag, message, and minimum display timer
- `startLoading()` function accepting optional message parameter
- Minimum display time enforcement (300ms) to prevent flashing
- `stopLoading()` function respecting minimum display time
- Timer cleanup to avoid premature dismissal
- Context provider wrapping children with loading functionality

**SRD-REQ-FRONT-022:** [Implements: PRD-REQ-UI-016] Loading indicators SHALL be displayed for:
- Initial page load and route navigation
- Authentication operations (login, logout, token refresh)
- API calls that take longer than 200ms

**Note:** Additional loading states for form submissions, data fetching, file uploads, and bulk operations will be added in Phase 2+ as features are implemented.

#### 3.5.7 Layout Consistency and Accessibility

**SRD-REQ-FRONT-023:** [Implements: PRD-REQ-UI-017, PRD-REQ-UI-018] The layout SHALL ensure:
- Consistent structure across all authenticated pages
- Navigation state maintained during page transitions
- Semantic HTML5 elements (`<header>`, `<nav>`, `<main>`, `<aside>`)
- Keyboard navigation support for all interactive elements
- Focus management for modals and overlays
- Proper ARIA labels and roles
- Touch target sizes of at least 44x44px on mobile
- Accessible contrast ratios (WCAG 2.1 AA minimum - 4.5:1 for text)
- Consistent spacing and padding using Tailwind spacing scale
- Unified color scheme (recommend indigo/blue as primary)

**SRD-REQ-FRONT-024:** [Implements: PRD-REQ-UI-002, PRD-REQ-UI-018] The layout responsive breakpoints SHALL follow:
- Mobile: < 768px (sm)
- Tablet: 768px - 1023px (md)
- Desktop: ≥ 1024px (lg+)

---

## 4. Firebase Cloud Functions Requirements

### 4.1 Function Organization

**SRD-REQ-FUNC-001:** [Implements: PRD-REQ-TECH-001] Cloud Functions SHALL be organized with basic structure:

**Functions Directory Structure:**
- `src/index.ts` - Main exports for Cloud Functions
- `src/api/` - API endpoints (placeholder structure)
- `src/shared/` - Shared utilities
  - `auth.ts` - Authentication helpers
  - `firebase-admin.ts` - Firebase Admin SDK initialization
  - `validators.ts` - Input validation utilities
- `package.json` - Dependencies and scripts

### 4.2 API Endpoints

**SRD-REQ-FUNC-002:** [Implements: PRD-REQ-API-003] Basic API structure SHALL be established:

#### Basic API Setup
- API routing structure with versioning (`/api/v1/`)
- Health check endpoint for testing: `GET /api/v1/health`
- Authentication middleware framework

**Note:** Specific feature endpoints (patients, observers, alerts, incidents) will be implemented in Phase 2+.

### 4.3 Authentication and Authorization

**SRD-REQ-FUNC-003:** [Implements: PRD-REQ-SEC-001, PRD-REQ-API-001, PRD-REQ-API-002] All API endpoints SHALL require Firebase Google Authentication:

```typescript
// Example: Authentication middleware
export async function verifyAuth(req: Request): Promise<string> {
  const authHeader = req.headers.get('Authorization');
  if (!authHeader?.startsWith('Bearer ')) {
    throw new Error('Missing authorization header');
  }
  
  const token = authHeader.split('Bearer ')[1];
  const decodedToken = await admin.auth().verifyIdToken(token);
  
  // Verify Google provider
  if (decodedToken.firebase.sign_in_provider !== 'google.com') {
    throw new Error('Only Google authentication is supported');
  }
  
  return decodedToken.uid;
}
```

**SRD-REQ-FUNC-004:** [Implements: PRD-REQ-SEC-001] Role-based access control framework SHALL be established:

- Authentication middleware for verifying Firebase tokens
- User role determination (caretaker/observer) - foundation for future use
- Authorization helper functions

---

## 5. Firestore Database Requirements

### 5.1 Collection Structure

**SRD-REQ-DB-001:** [Implements: PRD-REQ-TECH-001] Basic Firestore collections:

- `users` - User accounts (both caretakers and observers)
  - Fields: `uid`, `email`, `displayName`, `photoURL`, `role`, `createdAt`, `updatedAt`

### 5.2 Security Rules

**SRD-REQ-DB-002:** [Implements: PRD-REQ-SEC-001] Firestore security rules SHALL enforce:

- Authenticated users can read/write their own user document
- Cloud Functions have full access via Admin SDK
- Default deny for all other access

### 5.3 Indexes

**SRD-REQ-DB-003:** Basic indexes:

- Single-field indexes created automatically by Firestore
- Composite indexes will be added as needed

---

## 6. External Service Integration

**SRD-REQ-INT-001:** [Implements: PRD-REQ-TECH-001] SHALL only integrate:

- **Firebase Authentication:** Google OAuth for user authentication

---

## 7. Development and Build Requirements

### 7.1 Project Structure

**SRD-REQ-DEV-001:** [Implements: PRD-REQ-TECH-001] The project SHALL follow this structure:

**Project Root Structure:**
- `app/` - Next.js app directory
- `functions/` - Firebase Cloud Functions
- `public/` - Static assets
- `docs/` - Documentation
- `firebase.json` - Firebase configuration
- `firestore.rules` - Firestore security rules
- `firestore.indexes.json` - Firestore indexes
- `next.config.ts` - Next.js configuration
- `tailwind.config.ts` - Tailwind configuration
- `tsconfig.json` - TypeScript configuration
- `package.json` - Root dependencies

### 7.2 Build Configuration

**SRD-REQ-DEV-002:** [Implements: PRD-REQ-TECH-001] Next.js SHALL be configured for static export:

```typescript
// next.config.ts
const nextConfig: NextConfig = {
  output: 'export', // Static export for Firebase Hosting
  trailingSlash: true,
  images: {
    unoptimized: true, // Required for static export
  },
};
```

**SRD-REQ-DEV-003:** [Implements: PRD-REQ-TECH-001, PRD-REQ-API-003] Firebase hosting SHALL be configured for static site deployment:

```json
// firebase.json
{
  "hosting": {
    "public": "out",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

### 7.3 Environment Variables

**SRD-REQ-DEV-004:** [Implements: PRD-REQ-SEC-001] Required environment variables:

**Frontend (.env.local):**
- `NEXT_PUBLIC_FIREBASE_API_KEY`
- `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`
- `NEXT_PUBLIC_FIREBASE_PROJECT_ID`
- `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`
- `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
- `NEXT_PUBLIC_FIREBASE_APP_ID`

**Note:** Google Sign-In must be enabled in Firebase Console Authentication settings.

**Cloud Functions (.env):**
- No additional environment variables required

---

## 8. Performance Requirements

### 8.1 Frontend Performance

**SRD-REQ-PERF-001:** The application SHALL meet baseline performance targets:

- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Lighthouse Performance Score: > 85

**SRD-REQ-PERF-002:** Optimization strategies:

- Static site generation for landing page
- Code splitting for authenticated routes
- Optimized bundle size with tree shaking
- Minimal initial JavaScript payload

---

## 9. Security Requirements

### 9.1 Authentication

**SRD-REQ-SEC-001:** [Implements: PRD-REQ-SEC-001] Authentication SHALL use Firebase Google Authentication:

- Google Sign-In OAuth authentication for all users
- JWT token validation framework in Cloud Functions
- Session management via Firebase Authentication
- Automatic user profile creation from Google account
- User document created in Firestore on first sign-in

### 9.2 Authorization

**SRD-REQ-SEC-002:** [Implements: PRD-REQ-SEC-001] Basic authorization framework SHALL be established:

- Firestore security rules for user documents (read/write own document only)
- Cloud Function authentication middleware
- Frontend route protection via middleware (development) and client-side checks (production)

### 9.3 Data Protection

**SRD-REQ-SEC-003:** [Implements: PRD-REQ-SEC-003] Basic data protection:

- Google OAuth tokens managed by Firebase (not stored locally)
- HTTPS/TLS for all communications
- Secure cookie configuration
- Environment variables for sensitive configuration

---

## 10. Testing Requirements

### 10.1 API Testing

**SRD-REQ-TEST-001:** [Implements: PRD-REQ-TEST-001] API functions SHALL be tested using Behavior-Driven Development (BDD):

- All Firebase Cloud Functions endpoints SHALL have BDD tests
- Tests SHALL be written before implementation (test-first approach)
- Tests SHALL use a BDD framework (e.g., Jest with BDD syntax or Mocha/Chai)
- Tests SHALL cover:
  - Authentication and authorization
  - Request validation
  - Response formatting
  - Error handling
  - Edge cases and boundary conditions

### 10.2 Manual Testing

**SRD-REQ-TEST-002:** [Implements: PRD-REQ-TEST-001] Frontend functionality SHALL be verified through manual testing:

- Google Sign-In authentication flow
- Landing page display (unauthenticated state)
- Dashboard access (authenticated state)
- Navigation menu functionality (desktop, tablet, mobile)
- User profile dropdown and logout
- Loading states
- Responsive design across devices

**Note:** Automated frontend testing is explicitly excluded per PRD-REQ-TEST-001.

---

## 11. Deployment Requirements

### 11.1 Build Process

**SRD-REQ-DEPLOY-001:** [Implements: PRD-REQ-TECH-001] Deployment SHALL use:

```bash
# Build Next.js app for static export
npm run build

# Deploy to Firebase Hosting
firebase deploy --only hosting

# Deploy Firestore security rules
firebase deploy --only firestore:rules
```

### 11.2 Environment Configuration

**SRD-REQ-DEPLOY-002:** [Implements: PRD-REQ-TECH-001] Environment setup:

- **Development:** Local development server (`npm run dev`)
- **Production:** Firebase Hosting with static site

### 11.3 Monitoring

**SRD-REQ-DEPLOY-003:** [Implements: PRD-REQ-REL-004] Basic monitoring:

- Firebase Hosting analytics
- Firebase Authentication logs
- Browser console error tracking

---

## 12. Dependencies

### 12.1 Frontend Dependencies

```json
{
  "dependencies": {
    "next"
    "react"
    "react-dom"
    "firebase"
    "tailwindcss"
    "@heroicons/react"
  },
  "devDependencies": {
    "typescript"
    "@types/node"
    "@types/react"
    "@types/react-dom"
  }
}
```

### 12.2 Cloud Functions Dependencies

```json
{
  "dependencies": {
    "firebase-admin"
    "firebase-functions"
  },
  "devDependencies": {
    "typescript"
    "@types/node"
  }
}
```

---

**End of Document**

