# Software Requirements Document
## Patient Alert Notification System - Next.js on Firebase

**Version:** 1.0  
**Date:** November 9, 2025  
**Document Status:** Draft  
**Project Phase:** MVP  
**Technology Stack:** Next.js 15, Firebase Hosting, Firestore, Firebase Cloud Functions, Tailwind CSS

---

## 1. Executive Summary

This document defines the software requirements for implementing the Patient Alert Notification System as a Next.js application hosted on Firebase. The system provides a PagerDuty-style alert management platform for healthcare scenarios, enabling caretakers to manage patient profiles, assign observers, create scheduled alerts, and handle incident escalation workflows.

### 1.1 Technology Stack

- **Frontend Framework:** Next.js 15 (App Router) with React 19
- **Hosting:** Firebase Hosting (static export)
- **Database:** Cloud Firestore
- **API Layer:** Firebase Cloud Functions (TypeScript)
- **Authentication:** Firebase Authentication (Google Sign-In)
- **UI Framework:** Tailwind CSS 4
- **Scheduling Service:** Quirrel (external webhook scheduler)
- **SMS Gateway:** Twilio
- **Email Service:** SendGrid (optional for MVP)

### 1.2 Key Architectural Decisions

- **Static Site Generation:** Next.js static export for optimal Firebase Hosting performance
- **Serverless API:** All business logic in Firebase Cloud Functions
- **Real-time Updates:** Firestore real-time listeners for incident status changes
- **Queue-First Architecture:** Background job processing for notifications and escalations
- **Type-Safe Development:** Full TypeScript implementation across frontend and backend

---

## 2. System Architecture Overview

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Firebase Hosting                          │
│              (Next.js Static Export)                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Dashboard  │  │  Onboarding  │  │  Incidents    │      │
│  │   (Caretaker)│  │   (Observer) │  │   (Observer)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ HTTPS API Calls
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              Firebase Cloud Functions                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Patient API │  │  Alert API    │  │ Incident API │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Observer API │  │ Escalation   │  │ Notification │      │
│  └──────────────┘  │   Engine      │  │   Service    │      │
│                    └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                          │
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Firestore   │  │   Quirrel    │  │    Twilio    │
│   Database   │  │  Scheduler   │  │  SMS Gateway │
└──────────────┘  └──────────────┘  └──────────────┘
```

### 2.2 Core Components

1. **Next.js Frontend Application**
   - Caretaker dashboard for patient and alert management
   - Observer onboarding flow
   - Observer dashboard for incident management
   - Real-time incident status updates

2. **Firebase Cloud Functions (API Layer)**
   - Patient management endpoints
   - Observer invitation and management
   - Alert creation and scheduling
   - Incident lifecycle management
   - Escalation policy engine
   - Notification delivery service
   - Webhook receiver for scheduled triggers

3. **Firestore Database**
   - Patient profiles
   - Observer accounts and assignments
   - Alert definitions and schedules
   - Incident records
   - Notification logs
   - Escalation policies

4. **External Services**
   - Quirrel: Webhook scheduling service
   - Twilio: SMS notification delivery
   - SendGrid: Email notifications (optional)

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
│   ├── auth/                    # Authentication components
│   │   └── GoogleSignInButton.tsx # Google OAuth button
│   ├── PatientCard.tsx          # Patient profile card
│   ├── AlertCard.tsx            # Alert schedule card
│   ├── IncidentCard.tsx         # Active incident card
│   ├── EscalationPolicyEditor.tsx
│   └── NotificationPreferences.tsx
├── context/                      # React context providers
│   ├── AuthContext.tsx          # Firebase Auth state
│   ├── LoadingContext.tsx       # Global loading state
│   ├── PatientContext.tsx       # Patient data management
│   └── IncidentContext.tsx      # Real-time incident updates
├── hooks/                        # Custom React hooks
│   ├── useAuth.ts               # Authentication hook
│   ├── useLoadingState.ts       # Loading state hook
│   └── useMobileMenu.ts         # Mobile menu state hook
├── (authenticated)/             # Route group for authenticated pages
│   ├── layout.tsx               # Authenticated layout wrapper
│   ├── dashboard/               # Caretaker dashboard
│   │   └── page.tsx
│   ├── patients/                # Patient management
│   │   ├── page.tsx             # Patient list
│   │   ├── [id]/page.tsx        # Patient detail
│   │   └── new/page.tsx         # Create patient
│   ├── alerts/                  # Alert management
│   │   ├── page.tsx             # Alert list
│   │   ├── [id]/page.tsx        # Alert detail
│   │   └── new/page.tsx         # Create alert
│   ├── incidents/               # Incident management
│   │   ├── page.tsx             # Active incidents
│   │   └── [id]/page.tsx        # Incident detail
│   ├── observers/               # Observer management (caretaker only)
│   │   ├── page.tsx             # Observer list
│   │   └── invite/page.tsx      # Invite observer
│   └── settings/                # Settings pages
│       ├── page.tsx             # Account settings
│       └── notifications/page.tsx # Notification preferences
├── onboarding/                  # Observer onboarding (public)
│   ├── [token]/page.tsx         # Onboarding flow
│   └── [token]/complete/page.tsx
├── ack/                         # Quick acknowledgment (public)
│   └── [token]/page.tsx         # Acknowledge incident
├── resolve/                     # Quick resolution (public)
│   └── [token]/page.tsx         # Resolve incident
├── login/                       # Authentication (public)
│   └── page.tsx
├── firebase/                     # Firebase configuration
│   └── config.ts
├── lib/                          # Utilities
│   ├── api-client.ts            # API client for Cloud Functions
│   └── utils.ts                 # Helper functions
└── middleware.ts                 # Route protection middleware
```

### 3.2 UI Framework Requirements

**SRD-REQ-FRONT-002:** [Implements: SRS-REQ-UI-002, SRS-REQ-UI-018] The application SHALL use Tailwind CSS 4 for styling:

- **Design System:** Modern, clean healthcare-focused UI
- **Responsive Design:** Mobile-first approach, optimized for mobile devices
- **Accessibility:** WCAG 2.1 AA compliance
- **Component Library:** Custom components built on Tailwind utilities
- **Dark Mode:** Optional (future enhancement)

**SRD-REQ-FRONT-003:** [Implements: SRS-REQ-UI-003, SRS-REQ-UI-013, SRS-REQ-UI-016] Key UI components SHALL include:

- **Patient Profile Card:** Display patient info, status, active alerts count
- **Alert Schedule Card:** Show alert name, schedule, next trigger time
- **Incident Card:** Display incident status, severity, assigned observer
- **Escalation Policy Editor:** Visual editor for escalation levels
- **Notification Preferences:** Observer preference management
- **Real-time Status Indicators:** Visual indicators for incident states

### 3.3 State Management

**SRD-REQ-FRONT-004:** [Implements: SRS-REQ-UI-003, SRS-REQ-UI-017] The application SHALL use React Context API for state management:

- **AuthContext:** Firebase Authentication state
- **PatientContext:** Patient data and CRUD operations
- **AlertContext:** Alert schedules and management
- **IncidentContext:** Real-time incident updates via Firestore listeners

**SRD-REQ-FRONT-005:** [Implements: SRS-REQ-INC-001, SRS-REQ-INC-002, SRS-REQ-INC-003] Real-time updates SHALL be implemented using Firestore listeners:

```typescript
// Example: Real-time incident updates
useEffect(() => {
  const unsubscribe = onSnapshot(
    query(
      collection(db, 'incidents'),
      where('status', '==', 'triggered'),
      where('patientId', 'in', userPatientIds)
    ),
    (snapshot) => {
      const incidents = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      }));
      setActiveIncidents(incidents);
    }
  );
  
  return () => unsubscribe();
}, [userPatientIds]);
```

### 3.4 Routing and Navigation

**SRD-REQ-FRONT-006:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] The application SHALL implement protected routes:

- **Public Routes:** Landing page, login, onboarding, acknowledgment links
- **Caretaker Routes:** Dashboard, patients, alerts, observers (require caretaker role)
- **Observer Routes:** Observer dashboard, assigned incidents (require observer role)
- **Shared Routes:** Profile settings, notification preferences

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
  const { user, userRole } = useAuth();
  
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
          
          <NavLink 
            href="/patients" 
            icon={UserGroupIcon}
            active={pathname.startsWith('/patients')}
          >
            Patients
          </NavLink>
          
          <NavLink 
            href="/alerts" 
            icon={BellIcon}
            active={pathname.startsWith('/alerts')}
          >
            Alerts
          </NavLink>
          
          <NavLink 
            href="/incidents" 
            icon={ExclamationTriangleIcon}
            active={pathname.startsWith('/incidents')}
          >
            Incidents
          </NavLink>
          
          {userRole === 'caretaker' && (
            <NavLink 
              href="/observers" 
              icon={UsersIcon}
              active={pathname.startsWith('/observers')}
            >
              Observers
            </NavLink>
          )}
          
          <NavLink 
            href="/settings" 
            icon={CogIcon}
            active={pathname === '/settings'}
          >
            Settings
          </NavLink>
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
- Show role-specific navigation items (Observers link only for caretakers)
- Maintain selection state across page navigation
- Use semantic HTML (`<nav>`, `<aside>`)

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
  const router = useRouter();
  
  return (
    <div className="absolute right-0 mt-2 w-56 rounded-md shadow-lg bg-white ring-1 ring-black ring-opacity-5">
      <div className="py-1" role="menu">
        {/* User Info Section */}
        <div className="px-4 py-3 border-b border-gray-200">
          <p className="text-sm font-medium text-gray-900">{user?.displayName}</p>
          <p className="text-sm text-gray-500 truncate">{user?.email}</p>
        </div>
        
        {/* Menu Items */}
        <DropdownMenuItem 
          icon={UserCircleIcon}
          onClick={() => router.push('/settings')}
        >
          Account Settings
        </DropdownMenuItem>
        
        <DropdownMenuItem 
          icon={BellAlertIcon}
          onClick={() => router.push('/settings/notifications')}
        >
          Notification Preferences
        </DropdownMenuItem>
        
        <DropdownMenuItem 
          icon={QuestionMarkCircleIcon}
          onClick={() => router.push('/help')}
        >
          Help & Documentation
        </DropdownMenuItem>
        
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
- Include links to Account Settings, Notification Preferences, Help/Documentation
- Show Logout/Sign Out option with clear visual separation (divider line)
- Position Logout as the last item
- Include icons for all menu items for visual clarity
- Close when clicking outside or selecting an option
- Use proper ARIA attributes for accessibility
- Implement click-outside detection or focus trap

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
- Form submissions (create/update patient, alert, observer)
- Data fetching operations (fetching patients, incidents, alerts)
- Authentication operations (login, logout, token refresh)
- File uploads (patient profile photos)
- API calls that take longer than 200ms
- Bulk operations (batch updates, deletes)

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

**SRD-REQ-FUNC-001:** [Implements: SRS-REQ-TECH-001] Cloud Functions SHALL be organized by feature domain:

```
functions/
├── src/
│   ├── index.ts                 # Main exports
│   ├── patients/                # Patient management
│   │   └── index.ts
│   ├── observers/               # Observer management
│   │   └── index.ts
│   ├── alerts/                  # Alert management
│   │   └── index.ts
│   ├── incidents/               # Incident management
│   │   └── index.ts
│   ├── notifications/           # Notification service
│   │   └── index.ts
│   ├── escalation/              # Escalation engine
│   │   └── index.ts
│   ├── webhooks/                # Webhook receivers
│   │   └── index.ts
│   ├── scheduling/              # Schedule management
│   │   └── index.ts
│   ├── services/                # Business logic services
│   │   ├── patient-service.ts
│   │   ├── observer-service.ts
│   │   ├── alert-service.ts
│   │   ├── incident-service.ts
│   │   ├── escalation-service.ts
│   │   ├── notification-service.ts
│   │   └── scheduling-service.ts
│   └── shared/                  # Shared utilities
│       ├── auth.ts              # Authentication helpers
│       ├── firebase-admin.ts    # Firebase Admin SDK
│       ├── quirrel-client.ts    # Quirrel API client
│       ├── twilio-client.ts     # Twilio SMS client
│       └── validators.ts        # Input validation
└── package.json
```

### 4.2 API Endpoints

**SRD-REQ-FUNC-002:** [Implements: SRS-REQ-API-003, SRS-REQ-API-004, SRS-REQ-API-005, SRS-REQ-API-006, SRS-REQ-API-007, SRS-REQ-API-008, SRS-REQ-API-009, SRS-REQ-API-010, SRS-REQ-API-011] The following API endpoints SHALL be implemented:

#### Patient Management (`/api/v1/patients`)
- `POST /api/v1/patients` - Create patient profile
- `GET /api/v1/patients` - List caretaker's patients
- `GET /api/v1/patients/{id}` - Get patient details
- `PUT /api/v1/patients/{id}` - Update patient
- `DELETE /api/v1/patients/{id}` - Archive patient
- `GET /api/v1/patients/{id}/incidents` - Get patient incidents

#### Observer Management (`/api/v1/observers`)
- `POST /api/v1/patients/{patientId}/observers/invite` - Invite observer
- `GET /api/v1/patients/{patientId}/observers` - List patient observers
- `PUT /api/v1/patients/{patientId}/observers/{id}` - Update assignment
- `DELETE /api/v1/patients/{patientId}/observers/{id}` - Remove observer
- `POST /api/v1/invitations/{id}/resend` - Resend invitation
- `DELETE /api/v1/invitations/{id}` - Cancel invitation

#### Alert Management (`/api/v1/alerts`)
- `POST /api/v1/patients/{patientId}/alerts` - Create alert
- `GET /api/v1/patients/{patientId}/alerts` - List patient alerts
- `GET /api/v1/alerts/{id}` - Get alert details
- `PUT /api/v1/alerts/{id}` - Update alert
- `DELETE /api/v1/alerts/{id}` - Cancel alert
- `POST /api/v1/alerts/{id}/pause` - Pause alert
- `POST /api/v1/alerts/{id}/resume` - Resume alert
- `POST /api/v1/alerts/{id}/test` - Test trigger alert

#### Incident Management (`/api/v1/incidents`)
- `GET /api/v1/incidents` - List all incidents (caretaker)
- `GET /api/v1/patients/{patientId}/incidents` - List patient incidents
- `GET /api/v1/incidents/{id}` - Get incident details
- `POST /api/v1/incidents/{id}/resolve` - Resolve incident
- `POST /api/v1/incidents/{id}/reassign` - Reassign incident
- `POST /api/v1/incidents/{id}/notes` - Add note

#### Observer Operations (`/api/v1/me`)
- `GET /api/v1/me` - Get observer profile
- `PUT /api/v1/me` - Update profile
- `PUT /api/v1/me/preferences` - Update notification preferences
- `GET /api/v1/me/patients` - List assigned patients
- `GET /api/v1/me/incidents` - List active incidents
- `POST /api/v1/incidents/{id}/acknowledge` - Acknowledge incident
- `POST /api/v1/incidents/{id}/resolve` - Resolve incident

#### Onboarding (`/api/v1/invitations`)
- `GET /api/v1/invitations/{token}` - Get invitation details
- `POST /api/v1/invitations/{token}/accept` - Accept invitation
- `POST /api/v1/auth/verify-phone` - Verify phone via SMS
- `POST /api/v1/auth/verify-email` - Verify email

#### Public Endpoints (`/ack`, `/resolve`)
- `GET /ack/{token}` - Acknowledgment page
- `POST /ack/{token}` - Submit acknowledgment
- `GET /resolve/{token}` - Resolution page
- `POST /resolve/{token}` - Submit resolution

#### Webhook Endpoints (`/webhooks`)
- `POST /webhooks/alert-trigger` - Receive scheduled alert trigger
- `POST /webhooks/sms-status` - Receive SMS delivery status
- `POST /webhooks/email-status` - Receive email delivery status

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

**SRD-REQ-FUNC-004:** [Implements: SRS-REQ-SEC-005] Role-based access control SHALL be enforced:

- **Caretaker Role:** Full access to their patients, alerts, and incidents
- **Observer Role:** Access only to assigned patients and incidents
- **Public Endpoints:** Token-based access for acknowledgment/resolution links

### 4.4 Escalation Engine

**SRD-REQ-FUNC-005:** [Implements: SRS-REQ-ESC-001, SRS-REQ-ESC-002, SRS-REQ-ESC-003, SRS-REQ-ESC-004, SRS-REQ-ESC-005] The escalation engine SHALL be implemented as a Cloud Function triggered by Firestore:

```typescript
// Firestore trigger for incident escalation
export const onIncidentCreated = functions.firestore
  .document('incidents/{incidentId}')
  .onCreate(async (snap, context) => {
    const incident = snap.data();
    
    // Start Level 1 escalation
    await startEscalationLevel(incident.id, 1);
    
    // Schedule escalation timer
    await scheduleEscalationTimer(incident.id, 1);
  });
```

**SRD-REQ-FUNC-006:** [Implements: SRS-REQ-ESC-004, SRS-REQ-ESC-007, SRS-REQ-ESC-008] Escalation timers SHALL use Cloud Functions scheduled functions:

```typescript
// Scheduled function for escalation checks
export const checkEscalations = functions.pubsub
  .schedule('every 1 minutes')
  .onRun(async (context) => {
    const dueEscalations = await getDueEscalations();
    
    for (const escalation of dueEscalations) {
      await processEscalation(escalation);
    }
  });
```

### 4.5 Notification Service

**SRD-REQ-FUNC-007:** [Implements: SRS-REQ-NOTIF-001, SRS-REQ-NOTIF-003, SRS-REQ-NOTIF-004, SRS-REQ-TECH-001] Notification delivery SHALL be implemented as a queue-based system:

```typescript
// Notification queue processor
export const processNotificationQueue = functions.pubsub
  .schedule('every 30 seconds')
  .onRun(async (context) => {
    const pendingNotifications = await getPendingNotifications();
    
    for (const notification of pendingNotifications) {
      await sendNotification(notification);
    }
  });
```

**SRD-REQ-FUNC-008:** [Implements: SRS-REQ-NOTIF-001, SRS-REQ-NOTIF-002, SRS-REQ-INT-006, SRS-REQ-INT-007, SRS-REQ-INT-008] SMS notifications SHALL use Twilio:

```typescript
// Twilio SMS service
export async function sendSMSNotification(
  phoneNumber: string,
  message: string
): Promise<string> {
  const twilioClient = twilio(
    process.env.TWILIO_ACCOUNT_SID,
    process.env.TWILIO_AUTH_TOKEN
  );
  
  const result = await twilioClient.messages.create({
    body: message,
    to: phoneNumber,
    from: process.env.TWILIO_PHONE_NUMBER,
    statusCallback: `${API_BASE_URL}/webhooks/sms-status`
  });
  
  return result.sid;
}
```

---

## 5. Firestore Database Requirements

### 5.1 Collection Structure

**SRD-REQ-DB-001:** [Implements: SRS-REQ-PATIENT-001, SRS-REQ-OBS-001, SRS-REQ-OBS-007, SRS-REQ-ALERT-001, SRS-REQ-INC-001, SRS-REQ-ESC-001, SRS-REQ-NOTIF-003, SRS-REQ-TECH-001] The following Firestore collections SHALL be implemented:

- `caretakers` - Caretaker user accounts
- `patients` - Patient profiles
- `observers` - Observer user accounts
- `observer_invitations` - Pending observer invitations
- `patient_observers` - Observer-patient assignments
- `escalation_policies` - Patient escalation policies
- `alerts` - Alert definitions
- `alert_schedules` - Alert schedule metadata
- `scheduled_instances` - Individual scheduled occurrences
- `incidents` - Alert incidents
- `notifications` - Notification delivery logs
- `incident_actions` - Incident action audit log
- `acknowledgment_tokens` - Acknowledgment link tokens

### 5.2 Security Rules

**SRD-REQ-DB-002:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005, SRS-REQ-SEC-006] Firestore security rules SHALL enforce:

- Caretakers can only access their own patients and related data
- Observers can only access assigned patients and incidents
- Public acknowledgment tokens have limited read access
- Cloud Functions have full access via Admin SDK

### 5.3 Indexes

**SRD-REQ-DB-003:** [Implements: SRS-REQ-SCALE-002] Composite indexes SHALL be created for:

- `incidents`: `status + triggeredAt`, `patientId + status`, `assignedObserverId + status`
- `alerts`: `patientId + status`, `status + nextTriggerAt`
- `notifications`: `incidentId`, `observerId + createdAt`, `deliveryStatus + sentAt`
- `scheduled_instances`: `scheduledTime + status`, `alertId + scheduledTime`

---

## 6. External Service Integration

### 6.1 Quirrel Scheduling Service

**SRD-REQ-INT-001:** [Implements: SRS-SRD-REQ-INT-001, SRS-SRD-REQ-INT-002, SRS-REQ-SCHED-001, SRS-REQ-SCHED-002, SRS-REQ-SCHED-004] Quirrel integration SHALL handle:

- Creating scheduled webhooks from RRULE patterns
- Updating schedules when alerts are modified
- Cancelling schedules when alerts are deleted
- Receiving and validating webhook callbacks

**SRD-REQ-INT-002:** [Implements: SRS-REQ-ALERT-002, SRS-REQ-ALERT-003, SRS-REQ-SCHED-006, SRS-SRD-REQ-INT-003, SRS-REQ-INT-005] RRULE to Quirrel conversion SHALL support:

- Daily, weekly, monthly, hourly patterns
- Timezone conversion
- Complex RRULE patterns via calculated scheduling

### 6.2 Twilio SMS Service

**SRD-REQ-INT-003:** [Implements: SRS-REQ-INT-006, SRS-REQ-INT-007, SRS-REQ-INT-008] Twilio integration SHALL provide:

- SMS notification delivery
- Delivery status webhooks
- Link shortening for acknowledgment URLs
- Rate limiting compliance

### 6.3 SendGrid Email Service (Optional)

**SRD-REQ-INT-004:** [Implements: SRS-REQ-INT-009, SRS-REQ-OBS-003] Email notifications SHALL be optional for MVP:

- Observer invitations
- Notification delivery (if observer prefers email)
- Delivery status tracking

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

**SRD-REQ-DEV-003:** [Implements: SRS-REQ-TECH-001, SRS-REQ-API-003, SRS-REQ-API-011] Firebase hosting SHALL route API calls to Cloud Functions:

```json
// firebase.json
{
  "hosting": {
    "public": "out",
    "rewrites": [
      {
        "source": "/api/**",
        "function": "apiRouter"
      },
      {
        "source": "/webhooks/**",
        "function": "webhookRouter"
      }
    ]
  }
}
```

### 7.3 Environment Variables

**SRD-REQ-DEV-004:** [Implements: SRS-SRD-REQ-SEC-001, SRS-SRD-REQ-INT-001, SRS-REQ-INT-006] Required environment variables:

**Frontend (.env.local):**
- `NEXT_PUBLIC_FIREBASE_API_KEY`
- `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`
- `NEXT_PUBLIC_FIREBASE_PROJECT_ID`
- `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`
- `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
- `NEXT_PUBLIC_FIREBASE_APP_ID`

**Note:** Google Sign-In must be enabled in Firebase Console Authentication settings.

**Cloud Functions (.env):**
- `QUIRREL_URL`
- `QUIRREL_TOKEN`
- `TWILIO_ACCOUNT_SID`
- `TWILIO_AUTH_TOKEN`
- `TWILIO_PHONE_NUMBER`
- `SENDGRID_API_KEY` (optional)
- `API_BASE_URL`

---

## 8. Performance Requirements

### 8.1 Frontend Performance

**SRD-REQ-PERF-001:** [Implements: SRS-SRD-REQ-PERF-001, SRS-SRD-REQ-PERF-002, SRS-SRD-REQ-PERF-003] The application SHALL meet:

- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Lighthouse Performance Score: > 90

**SRD-REQ-PERF-002:** [Implements: SRS-SRD-REQ-PERF-001, SRS-SRD-REQ-PERF-002, SRS-SRD-REQ-PERF-003] Optimization strategies:

- Static site generation for public pages
- Code splitting for route-based chunks
- Image optimization (Next.js Image component)
- Lazy loading for non-critical components

### 8.2 API Performance

**SRD-REQ-PERF-003:** [Implements: SRS-SRD-REQ-PERF-001, SRS-SRD-REQ-PERF-002, SRS-SRD-REQ-PERF-003] Cloud Functions SHALL meet:

- API response time: < 2s (p95)
- Webhook processing: < 5s
- Notification queue processing: < 30s

### 8.3 Database Performance

**SRD-REQ-PERF-004:** [Implements: SRS-REQ-SCALE-002, SRS-REQ-SCALE-003] Firestore queries SHALL be optimized:

- Use composite indexes for complex queries
- Limit query results with pagination
- Cache frequently accessed data
- Use real-time listeners efficiently

---

## 9. Security Requirements

### 9.1 Authentication

**SRD-REQ-SEC-001:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-API-001] Authentication SHALL use Firebase Google Authentication:

- Google Sign-In OAuth authentication for all users
- JWT token validation on all API calls
- Session management via secure cookies
- Automatic user profile creation from Google account
- Phone number collection during onboarding (for SMS notifications)

### 9.2 Authorization

**SRD-REQ-SEC-002:** [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] Role-based access control SHALL be enforced:

- Firestore security rules
- Cloud Function authorization checks
- Frontend route protection
- API endpoint authorization

### 9.3 Data Protection

**SRD-REQ-SEC-003:** [Implements: SRS-SRD-REQ-SEC-003, SRS-REQ-SEC-006] Sensitive data SHALL be protected:

- Phone numbers encrypted at rest
- Invitation tokens hashed
- Acknowledgment tokens expire after 24 hours
- Google OAuth tokens managed by Firebase (no local storage)

### 9.4 Webhook Security

**SRD-REQ-SEC-004:** [Implements: SRS-SRD-REQ-SEC-002, SRS-REQ-SEC-006, SRS-REQ-SEC-007, SRS-REQ-SCHED-005] Webhook endpoints SHALL validate:

- HMAC-SHA256 signatures from Quirrel
- Token expiration for acknowledgment links
- Rate limiting (100 requests/minute per IP)
- Request size limits (10KB max)

---

## 10. Testing Requirements

### 10.1 Unit Testing

**SRD-REQ-TEST-001:** [Implements: SRS-REQ-USE-005] Unit tests SHALL cover:

- Business logic services (>80% coverage)
- Utility functions
- Validation logic
- Escalation policy engine

### 10.2 Integration Testing

**SRD-REQ-TEST-002:** [Implements: SRS-REQ-USE-005] Integration tests SHALL verify:

- API endpoint functionality
- Firestore operations
- External service integrations (mocked)
- Authentication flows

### 10.3 End-to-End Testing

**SRD-REQ-TEST-003:** [Implements: SRS-REQ-USE-001, SRS-REQ-USE-002, SRS-REQ-USE-003, SRS-REQ-USE-004] E2E tests SHALL cover:

- Complete user flows (caretaker and observer)
- Onboarding process
- Alert creation and triggering
- Incident escalation workflow
- Notification delivery

---

## 11. Deployment Requirements

### 11.1 Build Process

**SRD-REQ-DEPLOY-001:** [Implements: SRS-REQ-TECH-001] Deployment SHALL use:

```bash
# Build Next.js app
npm run build

# Deploy to Firebase
firebase deploy --only hosting,functions,firestore
```

### 11.2 Environment Configuration

**SRD-REQ-DEPLOY-002:** [Implements: SRS-REQ-TECH-001] Environment-specific configurations:

- **Development:** Local Firebase emulators
- **Staging:** Separate Firebase project
- **Production:** Production Firebase project with monitoring

### 11.3 Monitoring

**SRD-REQ-DEPLOY-003:** [Implements: SRS-REQ-REL-004, SRS-SRD-REQ-PERF-001] Monitoring SHALL include:

- Firebase Performance Monitoring
- Cloud Functions logs
- Error tracking (Firebase Crashlytics)
- API usage metrics
- Notification delivery rates

---

## 12. Implementation Phases

### Phase 1: Foundation (Week 1-2)
- Next.js project setup with Tailwind
- Firebase project configuration
- Google Authentication implementation
- Basic Firestore collections
- Core API structure

### Phase 2: Patient & Observer Management (Week 2-3)
- Patient CRUD operations
- Observer invitation system
- Onboarding flow
- Firestore security rules

### Phase 3: Alert & Scheduling (Week 3-4)
- Alert creation and management
- Quirrel integration
- RRULE parsing and scheduling
- Webhook receiver

### Phase 4: Incident & Escalation (Week 4-5)
- Incident lifecycle management
- Escalation policy engine
- Escalation timer system
- State machine implementation

### Phase 5: Notifications (Week 5-6)
- Twilio SMS integration
- Notification queue system
- Acknowledgment link generation
- Delivery status tracking

### Phase 6: Frontend UI (Week 6-7)
- Caretaker dashboard
- Observer dashboard
- Onboarding UI
- Incident management UI

### Phase 7: Testing & Deployment (Week 7-8)
- Comprehensive testing
- Performance optimization
- Security audit
- Production deployment

---

## 13. Dependencies

### 13.1 Frontend Dependencies

```json
{
  "dependencies": {
    "next": "^15.5.4",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "firebase": "^11.4.0",
    "tailwindcss": "^4.0.0",
    "@heroicons/react": "^2.2.0"
  }
}
```

### 13.2 Cloud Functions Dependencies

```json
{
  "dependencies": {
    "firebase-admin": "^13.4.0",
    "firebase-functions": "^6.3.2",
    "twilio": "^5.0.0",
    "@sendgrid/mail": "^8.0.0",
    "rrule": "^2.8.1",
    "axios": "^1.7.7"
  }
}
```

---

**End of Document**

