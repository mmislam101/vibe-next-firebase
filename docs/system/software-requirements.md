# Software Requirements Document
## Patient Alert Notification System - Next.js on Firebase

**Technology Stack:** Next.js 15, Firebase Hosting, Firestore, Firebase Cloud Functions, Tailwind CSS

---

## 1. Executive Summary

This document defines the software requirements for implementing the Patient Alert Notification System as a Next.js application hosted on Firebase. Phase 1 focuses on establishing the core infrastructure, authentication, and user interface layout.

### 1.1 Technology Stack

- **Frontend Framework:** Next.js 15 (App Router) with React 19
- **Hosting:** Firebase Hosting (static export)
- **Database:** Cloud Firestore
- **API Layer:** Firebase Cloud Functions (TypeScript)
- **Authentication:** Firebase Authentication (Google Sign-In)
- **UI Framework:** Tailwind CSS 4

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

**SRD-REQ-FRONT-001:** [Implements: SRS-REQ-UI-003, SRS-REQ-UI-004, SRS-REQ-UI-017] The application SHALL follow Next.js 15 App Router structure:

```
app/
├── layout.tsx                    # Root layout with providers
├── page.tsx                      # Landing page (unauthenticated)
├── globals.css                   # Tailwind CSS imports
├── components/                   # Reusable UI components
│   ├── ui/                      # Base UI components (buttons, inputs, etc.)
│   ├── layout/                  # Layout components
│   │   ├── TopNavBar.tsx       # Top navigation bar with profile menu
│   │   ├── LeftSideMenu.tsx    # Left sidebar navigation (desktop/tablet)
│   │   ├── MobileNavMenu.tsx   # Mobile hamburger menu
│   │   ├── UserDropdownMenu.tsx # User profile dropdown
│   │   ├── LoadingOverlay.tsx  # Global loading indicator
│   │   └── NavLink.tsx         # Navigation link component
│   └── auth/                    # Authentication components
│       └── GoogleSignInButton.tsx # Google OAuth button
├── context/                      # React context providers
│   ├── AuthContext.tsx          # Firebase Auth state
│   └── LoadingContext.tsx       # Global loading state
├── hooks/                        # Custom React hooks
│   ├── useAuth.ts               # Authentication hook
│   ├── useLoadingState.ts       # Loading state hook
│   └── useMobileMenu.ts         # Mobile menu state hook
├── (authenticated)/             # Route group for authenticated pages
│   ├── layout.tsx               # Authenticated layout wrapper
│   └── dashboard/               # Dashboard placeholder
│       └── page.tsx
├── lib/                          # Utilities
│   ├── firebase.ts              # Firebase configuration
│   └── utils.ts                 # Helper functions
└── middleware.ts                 # Route protection middleware (development only)
```

### 3.2 UI Framework Requirements

**SRD-REQ-FRONT-002:** [Implements: SRS-REQ-UI-002, SRS-REQ-UI-018] The application SHALL use Tailwind CSS 4 for styling:

- **Design System:** Modern, clean healthcare-focused UI
- **Responsive Design:** Mobile-first approach, optimized for mobile devices
- **Accessibility:** WCAG 2.1 AA compliance
- **Component Library:** Custom components built on Tailwind utilities
- **Dark Mode:** Optional (future enhancement)

**SRD-REQ-FRONT-003:** [Implements: SRS-REQ-UI-003, SRS-REQ-UI-013, SRS-REQ-UI-016] Key UI components SHALL include:

- **Button:** Primary, secondary, and danger variants
- **Input:** Text input with validation states
- **Card:** Container component for content sections
- **Loading Overlay:** Full-screen loading indicator
- **Navigation Components:** NavLink, LeftSideMenu, TopNavBar, MobileNavMenu

### 3.3 State Management

**SRD-REQ-FRONT-004:** [Implements: SRS-REQ-UI-003, SRS-REQ-UI-017] The application SHALL use React Context API for state management:

- **AuthContext:** Firebase Authentication state and user profile
- **LoadingContext:** Global loading state management

### 3.4 Routing and Navigation

**SRD-REQ-FRONT-006:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] The application SHALL implement protected routes:

- **Public Routes:** Landing page (/)
- **Authenticated Routes:** Dashboard (/dashboard)

**SRD-REQ-FRONT-007:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] Route protection SHALL be implemented via middleware:

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

**SRD-REQ-FRONT-008:** [Implements: SRS-REQ-UI-001] The landing page SHALL be implemented as:

```typescript
// app/page.tsx
export default function LandingPage() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100">
      <div className="max-w-md w-full space-y-8 p-8 bg-white rounded-xl shadow-lg">
        <div className="text-center">
          <h1 className="text-4xl font-bold text-gray-900">Family Pager</h1>
          <p className="mt-2 text-lg text-gray-600">
            Patient alert notification system for caretakers
          </p>
        </div>
        
        <GoogleSignInButton />
        
        <div className="text-center text-sm text-gray-500">
          <p>Sign in with your Google account to get started</p>
        </div>
      </div>
    </div>
  );
}
```

**SRD-REQ-FRONT-009:** [Implements: SRS-REQ-UI-001, SRS-REQ-UI-002] The landing page SHALL:
- Display prominently centered on the viewport
- Show application name and value proposition
- Feature a Google Sign-In button as primary call-to-action
- Use Firebase Authentication for Google OAuth
- Be responsive across desktop (1024px+), tablet (768-1023px), and mobile (<768px)
- Not display any authenticated navigation elements

#### 3.5.2 Authenticated Layout Structure

**SRD-REQ-FRONT-010:** [Implements: SRS-REQ-UI-003, SRS-REQ-UI-004, SRS-REQ-UI-006, SRS-REQ-UI-010, SRS-REQ-UI-017] Once authenticated, the application SHALL use a root layout component:

```typescript
// app/(authenticated)/layout.tsx
export default function AuthenticatedLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen flex flex-col">
      {/* Top Navigation Bar */}
      <TopNavBar />
      
      <div className="flex flex-1 overflow-hidden">
        {/* Left Sidebar - Desktop/Tablet only */}
        <LeftSideMenu />
        
        {/* Main Content Area */}
        <main className="flex-1 overflow-y-auto bg-gray-50 p-6">
          {children}
        </main>
      </div>
      
      {/* Mobile Navigation - Hamburger Menu */}
      <MobileNavMenu />
      
      {/* Global Loading Overlay */}
      <LoadingOverlay />
    </div>
  );
}
```

#### 3.5.3 Left-Side Menu Pane (Desktop/Tablet)

**SRD-REQ-FRONT-011:** [Implements: SRS-REQ-UI-004, SRS-REQ-UI-005] The left-side menu SHALL be implemented with:

```typescript
// app/components/layout/LeftSideMenu.tsx
'use client';

export function LeftSideMenu() {
  const pathname = usePathname();
  const { user } = useAuth();
  
  return (
    <aside className="hidden md:flex md:flex-shrink-0 w-64 bg-white border-r border-gray-200">
      <div className="flex flex-col w-full">
        {/* Navigation Links */}
        <nav className="flex-1 px-4 py-6 space-y-2">
          <NavLink 
            href="/dashboard" 
            icon={HomeIcon}
            active={pathname === '/dashboard'}
          >
            Dashboard
          </NavLink>
          
          {/* Future navigation items will be added here in Phase 2+ */}
        </nav>
      </div>
    </aside>
  );
}
```

**SRD-REQ-FRONT-012:** [Implements: SRS-REQ-UI-004, SRS-REQ-UI-005, SRS-REQ-UI-017] The left-side menu SHALL:
- Be visible only on viewports ≥768px width (hidden on mobile)
- Have a fixed width between 200-280px (recommend 256px/64rem)
- Remain fixed/persistent during navigation
- Highlight the currently active navigation item with visual indicator
- Use clear iconography for each menu item (from @heroicons/react)
- Maintain selection state across page navigation
- Use semantic HTML (`<nav>`, `<aside>`)
- Include placeholder structure for future navigation items

#### 3.5.4 Top Navigation Bar

**SRD-REQ-FRONT-013:** [Implements: SRS-REQ-UI-006, SRS-REQ-UI-007, SRS-REQ-UI-010] The top navigation bar SHALL be implemented as:

```typescript
// app/components/layout/TopNavBar.tsx
'use client';

export function TopNavBar() {
  const { user } = useAuth();
  const [showDropdown, setShowDropdown] = useState(false);
  const [showMobileMenu, setShowMobileMenu] = useState(false);
  
  return (
    <header className="sticky top-0 z-50 w-full border-b border-gray-200 bg-white">
      <div className="flex items-center justify-between h-16 px-4">
        {/* Left Section */}
        <div className="flex items-center space-x-4">
          {/* Hamburger Menu - Mobile Only */}
          <button
            onClick={() => setShowMobileMenu(true)}
            className="md:hidden p-2 rounded-md hover:bg-gray-100"
            aria-label="Open menu"
          >
            <Bars3Icon className="h-6 w-6" />
          </button>
          
          {/* Logo/Brand */}
          <Link href="/dashboard" className="flex items-center">
            <span className="text-xl font-bold text-indigo-600">
              Family Pager
            </span>
          </Link>
        </div>
        
        {/* Right Section - User Profile */}
        <div className="flex items-center">
          <div className="relative">
            <button
              onClick={() => setShowDropdown(!showDropdown)}
              className="flex items-center space-x-3 p-2 rounded-lg hover:bg-gray-100"
              aria-label="User menu"
            >
              {/* Google Profile Picture */}
              <img
                src={user?.photoURL || '/default-avatar.png'}
                alt={user?.displayName || 'User'}
                className="h-8 w-8 rounded-full"
              />
              
              {/* User Name - Desktop Only */}
              <span className="hidden md:block text-sm font-medium text-gray-700">
                {user?.displayName}
              </span>
              
              {/* Dropdown Indicator */}
              <ChevronDownIcon className="h-4 w-4 text-gray-500" />
            </button>
            
            {/* Dropdown Menu */}
            {showDropdown && <UserDropdownMenu />}
          </div>
        </div>
      </div>
    </header>
  );
}
```

**SRD-REQ-FRONT-014:** [Implements: SRS-REQ-UI-006, SRS-REQ-UI-007, SRS-REQ-UI-010, SRS-REQ-UI-018] The top navigation bar SHALL:
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

**SRD-REQ-FRONT-015:** [Implements: SRS-REQ-UI-008, SRS-REQ-UI-009] The user profile dropdown SHALL display:

```typescript
// app/components/layout/UserDropdownMenu.tsx
'use client';

export function UserDropdownMenu() {
  const { user, logout } = useAuth();
  
  return (
    <div className="absolute right-0 mt-2 w-56 rounded-md shadow-lg bg-white ring-1 ring-black ring-opacity-5">
      <div className="py-1" role="menu">
        {/* User Info Section */}
        <div className="px-4 py-3 border-b border-gray-200">
          <p className="text-sm font-medium text-gray-900">{user?.displayName}</p>
          <p className="text-sm text-gray-500 truncate">{user?.email}</p>
        </div>
        
        {/* Placeholder for future menu items (Settings, Help, etc.) */}
        
        {/* Divider */}
        <div className="border-t border-gray-200 my-1" />
        
        {/* Logout */}
        <DropdownMenuItem 
          icon={ArrowRightOnRectangleIcon}
          onClick={logout}
          className="text-red-600 hover:bg-red-50"
        >
          Sign Out
        </DropdownMenuItem>
      </div>
    </div>
  );
}
```

**SRD-REQ-FRONT-016:** [Implements: SRS-REQ-UI-008, SRS-REQ-UI-009, SRS-REQ-UI-017] The dropdown menu SHALL:
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

**SRD-REQ-FRONT-017:** [Implements: SRS-REQ-UI-010, SRS-REQ-UI-011, SRS-REQ-UI-012] The mobile navigation menu SHALL be implemented as:

```typescript
// app/components/layout/MobileNavMenu.tsx
'use client';

export function MobileNavMenu() {
  const { isOpen, closeMenu } = useMobileMenu();
  const pathname = usePathname();
  
  return (
    <>
      {/* Overlay */}
      {isOpen && (
        <div
          className="fixed inset-0 bg-black bg-opacity-50 z-40 md:hidden"
          onClick={closeMenu}
          aria-hidden="true"
        />
      )}
      
      {/* Slide-in Menu */}
      <div
        className={`
          fixed top-0 left-0 bottom-0 w-64 bg-white z-50 transform transition-transform duration-300 ease-in-out md:hidden
          ${isOpen ? 'translate-x-0' : '-translate-x-full'}
        `}
      >
        {/* Close Button */}
        <div className="flex items-center justify-between p-4 border-b border-gray-200">
          <span className="text-lg font-bold text-indigo-600">Menu</span>
          <button
            onClick={closeMenu}
            className="p-2 rounded-md hover:bg-gray-100"
            aria-label="Close menu"
          >
            <XMarkIcon className="h-6 w-6" />
          </button>
        </div>
        
        {/* Navigation Links - Same as Desktop */}
        <nav className="flex-1 px-4 py-6 space-y-2">
          {/* Same navigation items as LeftSideMenu */}
        </nav>
      </div>
    </>
  );
}
```

**SRD-REQ-FRONT-018:** [Implements: SRS-REQ-UI-010, SRS-REQ-UI-011, SRS-REQ-UI-012, SRS-REQ-UI-018] Mobile navigation (<768px) SHALL:
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

**SRD-REQ-FRONT-019:** [Implements: SRS-REQ-UI-013, SRS-REQ-UI-014, SRS-REQ-UI-015] The global loading overlay SHALL be implemented as:

```typescript
// app/components/layout/LoadingOverlay.tsx
'use client';

export function LoadingOverlay() {
  const { isLoading, loadingMessage } = useLoadingState();
  
  if (!isLoading) return null;
  
  return (
    <div
      className="fixed inset-0 z-[9999] flex items-center justify-center"
      style={{
        backgroundColor: 'rgba(0, 0, 0, 0.5)', // Semi-transparent overlay
      }}
      aria-live="assertive"
      aria-busy="true"
    >
      <div className="bg-white rounded-lg shadow-xl p-6 max-w-sm mx-4">
        {/* Spinner */}
        <div className="flex justify-center mb-4">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-indigo-600" />
        </div>
        
        {/* Loading Message */}
        {loadingMessage && (
          <p className="text-center text-gray-700 font-medium">
            {loadingMessage}
          </p>
        )}
      </div>
    </div>
  );
}
```

**SRD-REQ-FRONT-020:** [Implements: SRS-REQ-UI-013, SRS-REQ-UI-014, SRS-REQ-UI-015] The loading indicator SHALL:
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

**SRD-REQ-FRONT-021:** [Implements: SRS-REQ-UI-013, SRS-REQ-UI-015] The loading state management SHALL be centralized:

```typescript
// app/context/LoadingContext.tsx
'use client';

interface LoadingContextType {
  isLoading: boolean;
  loadingMessage: string | null;
  startLoading: (message?: string) => void;
  stopLoading: () => void;
}

export function LoadingProvider({ children }: { children: React.ReactNode }) {
  const [isLoading, setIsLoading] = useState(false);
  const [loadingMessage, setLoadingMessage] = useState<string | null>(null);
  const [minDisplayTime, setMinDisplayTime] = useState<NodeJS.Timeout | null>(null);
  
  const startLoading = useCallback((message?: string) => {
    setIsLoading(true);
    setLoadingMessage(message || null);
    
    // Ensure minimum 300ms display time
    const timer = setTimeout(() => {
      setMinDisplayTime(null);
    }, 300);
    setMinDisplayTime(timer);
  }, []);
  
  const stopLoading = useCallback(() => {
    if (minDisplayTime) {
      // Wait for minimum display time
      setTimeout(() => {
        setIsLoading(false);
        setLoadingMessage(null);
      }, 300);
    } else {
      setIsLoading(false);
      setLoadingMessage(null);
    }
  }, [minDisplayTime]);
  
  return (
    <LoadingContext.Provider value={{ isLoading, loadingMessage, startLoading, stopLoading }}>
      {children}
    </LoadingContext.Provider>
  );
}
```

**SRD-REQ-FRONT-022:** [Implements: SRS-REQ-UI-016] Loading indicators SHALL be displayed for:
- Initial page load and route navigation
- Authentication operations (login, logout, token refresh)
- API calls that take longer than 200ms

**Note:** Additional loading states for form submissions, data fetching, file uploads, and bulk operations will be added in Phase 2+ as features are implemented.

#### 3.5.7 Layout Consistency and Accessibility

**SRD-REQ-FRONT-023:** [Implements: SRS-REQ-UI-017, SRS-REQ-UI-018] The layout SHALL ensure:
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

**SRD-REQ-FRONT-024:** [Implements: SRS-REQ-UI-002, SRS-REQ-UI-018] The layout responsive breakpoints SHALL follow:
- Mobile: < 768px (sm)
- Tablet: 768px - 1023px (md)
- Desktop: ≥ 1024px (lg+)

---

## 4. Firebase Cloud Functions Requirements

### 4.1 Function Organization

**SRD-REQ-FUNC-001:** [Implements: SRS-REQ-TECH-001] Cloud Functions SHALL be organized with basic structure:

```
functions/
├── src/
│   ├── index.ts                 # Main exports
│   ├── api/                     # API endpoints (placeholder)
│   │   └── index.ts
│   └── shared/                  # Shared utilities
│       ├── auth.ts              # Authentication helpers
│       ├── firebase-admin.ts    # Firebase Admin SDK initialization
│       └── validators.ts        # Input validation utilities
└── package.json
```

### 4.2 API Endpoints

**SRD-REQ-FUNC-002:** [Implements: SRS-REQ-API-003] Basic API structure SHALL be established:

#### Basic API Setup
- API routing structure with versioning (`/api/v1/`)
- Health check endpoint for testing: `GET /api/v1/health`
- Authentication middleware framework

**Note:** Specific feature endpoints (patients, observers, alerts, incidents) will be implemented in Phase 2+.

### 4.3 Authentication and Authorization

**SRD-REQ-FUNC-003:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-API-001, SRS-REQ-API-002] All API endpoints SHALL require Firebase Google Authentication:

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

**SRD-REQ-FUNC-004:** [Implements: SRS-REQ-SEC-005] Role-based access control framework SHALL be established:

- Authentication middleware for verifying Firebase tokens
- User role determination (caretaker/observer) - foundation for future use
- Authorization helper functions

**Note:** Specific access control enforcement for patients, alerts, and incidents will be implemented in Phase 2+.

---

## 5. Firestore Database Requirements

### 5.1 Collection Structure

**SRD-REQ-DB-001:** [Implements: SRS-REQ-TECH-001] Basic Firestore collections:

- `users` - User accounts (both caretakers and observers)
  - Fields: `uid`, `email`, `displayName`, `photoURL`, `role`, `createdAt`, `updatedAt`

**Note:** Additional collections (patients, observers, alerts, incidents, etc.) will be added in Phase 2+.

### 5.2 Security Rules

**SRD-REQ-DB-002:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] Firestore security rules SHALL enforce:

- Authenticated users can read/write their own user document
- Cloud Functions have full access via Admin SDK
- Default deny for all other access

**Note:** More granular security rules for patients, observers, alerts, and incidents will be added in Phase 2+.

### 5.3 Indexes

**SRD-REQ-DB-003:** [Implements: SRS-REQ-SCALE-002] Basic indexes:

- Single-field indexes created automatically by Firestore
- Composite indexes will be added as needed in future phases

---

## 6. External Service Integration

**SRD-REQ-INT-001:** [Implements: SRS-REQ-TECH-001] SHALL only integrate:

- **Firebase Authentication:** Google OAuth for user authentication

**Note:** Additional external services (Quirrel, Twilio, SendGrid) will be integrated in Phase 2+.

---

## 7. Development and Build Requirements

### 7.1 Project Structure

**SRD-REQ-DEV-001:** [Implements: SRS-REQ-TECH-001] The project SHALL follow this structure:

```
family-pager/
├── app/                          # Next.js app directory
├── functions/                    # Firebase Cloud Functions
├── public/                       # Static assets
├── docs/                         # Documentation
├── firebase.json                 # Firebase configuration
├── firestore.rules               # Firestore security rules
├── firestore.indexes.json        # Firestore indexes
├── next.config.ts                # Next.js configuration
├── tailwind.config.ts            # Tailwind configuration
├── tsconfig.json                 # TypeScript configuration
└── package.json                  # Root dependencies
```

### 7.2 Build Configuration

**SRD-REQ-DEV-002:** [Implements: SRS-REQ-TECH-001] Next.js SHALL be configured for static export:

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

**SRD-REQ-DEV-003:** [Implements: SRS-REQ-TECH-001, SRS-REQ-API-003] Firebase hosting SHALL be configured for static site deployment:

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

**Note:** API routes will be added in Phase 2+ when backend endpoints are implemented.

### 7.3 Environment Variables

**SRD-REQ-DEV-004:** [Implements: SRS-SRD-REQ-SEC-001] Required environment variables:

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

**Note:** Additional environment variables (Quirrel, Twilio, SendGrid) will be added in Phase 2+.

---

## 8. Performance Requirements

### 8.1 Frontend Performance

**SRD-REQ-PERF-001:** [Implements: SRS-SRD-REQ-PERF-001] The application SHALL meet baseline performance targets:

- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Lighthouse Performance Score: > 85

**SRD-REQ-PERF-002:** [Implements: SRS-SRD-REQ-PERF-001] Optimization strategies:

- Static site generation for landing page
- Code splitting for authenticated routes
- Optimized bundle size with tree shaking
- Minimal initial JavaScript payload

**Note:** Additional performance optimizations (image optimization, lazy loading, caching) will be implemented in Phase 2+ as features are added.

---

## 9. Security Requirements

### 9.1 Authentication

**SRD-REQ-SEC-001:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-API-001] Authentication SHALL use Firebase Google Authentication:

- Google Sign-In OAuth authentication for all users
- JWT token validation framework in Cloud Functions
- Session management via Firebase Authentication
- Automatic user profile creation from Google account
- User document created in Firestore on first sign-in

### 9.2 Authorization

**SRD-REQ-SEC-002:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] Basic authorization framework SHALL be established:

- Firestore security rules for user documents (read/write own document only)
- Cloud Function authentication middleware
- Frontend route protection via middleware (development) and client-side checks (production)

**Note:** Role-based access control for patients, observers, and incidents will be implemented in Phase 2+.

### 9.3 Data Protection

**SRD-REQ-SEC-003:** [Implements: SRS-SRD-REQ-SEC-003] Basic data protection:

- Google OAuth tokens managed by Firebase (not stored locally)
- HTTPS/TLS for all communications
- Secure cookie configuration
- Environment variables for sensitive configuration

**Note:** Additional data protection (phone number encryption, token hashing) will be implemented in Phase 2+ as features require them.

---

## 10. Testing Requirements

### 10.1 Manual Testing

**SRD-REQ-TEST-001:** Manual testing SHALL verify functionality:

- Google Sign-In authentication flow
- Landing page display (unauthenticated state)
- Dashboard access (authenticated state)
- Navigation menu functionality (desktop, tablet, mobile)
- User profile dropdown and logout
- Loading states
- Responsive design across devices

**Note:** Automated testing (unit, integration, E2E) will be established in Phase 2+ as the application grows in complexity.

---

## 11. Deployment Requirements

### 11.1 Build Process

**SRD-REQ-DEPLOY-001:** [Implements: SRS-REQ-TECH-001] Deployment SHALL use:

```bash
# Build Next.js app for static export
npm run build

# Deploy to Firebase Hosting
firebase deploy --only hosting

# Deploy Firestore security rules
firebase deploy --only firestore:rules
```

**Note:** Cloud Functions deployment will be added in Phase 2+ when API endpoints are implemented.

### 11.2 Environment Configuration

**SRD-REQ-DEPLOY-002:** [Implements: SRS-REQ-TECH-001] Environment setup:

- **Development:** Local development server (`npm run dev`)
- **Production:** Firebase Hosting with static site

**Note:** Firebase emulators and staging environments will be added in Phase 2+ as backend complexity increases.

### 11.3 Monitoring

**SRD-REQ-DEPLOY-003:** [Implements: SRS-REQ-REL-004] Basic monitoring:

- Firebase Hosting analytics
- Firebase Authentication logs
- Browser console error tracking

**Note:** Advanced monitoring (Performance Monitoring, Crashlytics, API metrics) will be added in Phase 2+.

---

## 12. Implementation Phases

### Core Infrastructure and User Creation
**Goal:** Establish foundation with authentication and basic UI layout

**Deliverables:**
- Package.json with basic scripts
- Firebase initialization (Hosting, Firestore, Authentication, Functions)
- Firebase Functions setup with basic structure
- Google Sign-In authentication flow (Login/Logout)
- Landing page for unauthenticated users
- Authenticated layout with:
  - Top navigation bar with profile dropdown
  - Left sidebar menu (desktop/tablet)
  - Mobile hamburger menu
  - Global loading overlay
- Basic dashboard page (placeholder)
- Firebase Hosting deployment configuration
- Firestore security rules foundation

**SRS Requirements Implemented:**
- SRS-REQ-UI-001 through SRS-REQ-UI-018

---

## 13. Dependencies

### 13.1 Frontend Dependencies

```json
{
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "firebase": "^11.0.0",
    "tailwindcss": "^4.0.0",
    "@heroicons/react": "^2.2.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0"
  }
}
```

### 13.2 Cloud Functions Dependencies

```json
{
  "dependencies": {
    "firebase-admin": "^13.0.0",
    "firebase-functions": "^6.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0"
  }
}
```

**Note:** Additional dependencies (Twilio, SendGrid, rrule, axios) will be added in Phase 2+.

---

**End of Document**

