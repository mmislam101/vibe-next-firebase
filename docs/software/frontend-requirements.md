# Frontend Requirements Documentation

**Framework:** Next.js (App Router)
**UI Framework:** Tailwind CSS with PostCSS 
**Document Status:** Draft

---

## Implementation Status

### Phase 1: Authentication & Basic Layout (🔄 In Progress - November 20, 2025)

**✅ Completed Components (Authentication Core):**
- ✅ Firebase Client Configuration (`app/firebase/config.ts`) - [Implements: SRS-REQ-UI-001]
- ✅ Authentication Context (`app/context/AuthContext.tsx`) - [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-UI-001]
- ✅ Authentication Hook (`app/hooks/useAuth.ts`) - [Implements: SRS-SRD-REQ-SEC-001]
- ✅ Google Sign-In Button Component (`app/components/auth/GoogleSignInButton.tsx`) - [Implements: SRS-REQ-UI-001, SRS-REQ-UI-002]
- ✅ Landing Page with Auth Integration (`app/page.tsx`) - [Implements: SRS-REQ-UI-001, SRS-REQ-UI-002]
- ✅ Dashboard Page with Basic Layout (`app/(authenticated)/dashboard/page.tsx`) - [Implements: SRS-REQ-UI-003, SRS-REQ-UI-006, SRS-REQ-UI-007]
- ✅ Authenticated Layout Wrapper (`app/(authenticated)/layout.tsx`) - [Implements: SRS-REQ-UI-003, SRS-REQ-UI-017]
- ✅ Root Layout with AuthProvider (`app/layout.tsx`) - [Implements: SRS-REQ-UI-003]
- ✅ Route Protection Middleware (`middleware.ts`) - [Implements: SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005, SRS-REQ-UI-001, SRS-REQ-UI-003]
- ✅ Top Navigation Bar Component (`app/components/layout/TopNavBar.tsx`) - [Implements: SRS-REQ-UI-006, SRS-REQ-UI-007, SRS-REQ-UI-010]
- ✅ Left Side Menu Component (`app/components/layout/LeftSideMenu.tsx`) - [Implements: SRS-REQ-UI-004, SRS-REQ-UI-005]
- ✅ Mobile Navigation Menu (`app/components/layout/MobileNavMenu.tsx`) - [Implements: SRS-REQ-UI-010, SRS-REQ-UI-011, SRS-REQ-UI-012]
- ✅ User Dropdown Menu (`app/components/layout/UserDropdownMenu.tsx`) - [Implements: SRS-REQ-UI-008, SRS-REQ-UI-009]
- ✅ Navigation Link Component (`app/components/layout/NavLink.tsx`) - [Implements: SRS-REQ-UI-005]
- ✅ Loading Overlay Component (`app/components/layout/LoadingOverlay.tsx`) - [Implements: SRS-REQ-UI-013, SRS-REQ-UI-014, SRS-REQ-UI-015]
- ✅ Loading Context (`app/context/LoadingContext.tsx`) - [Implements: SRS-REQ-UI-013, SRS-REQ-UI-016]
- ✅ Loading State Hook (`app/hooks/useLoadingState.ts`) - [Implements: SRS-REQ-UI-013, SRS-REQ-UI-016]
- ✅ Mobile Menu Hook (`app/hooks/useMobileMenu.ts`) - [Implements: SRS-REQ-UI-010, SRS-REQ-UI-011]

**Completed Features:**
- ✅ Google OAuth Sign-In with Firebase Authentication [SRS-REQ-UI-001, SRS-SRD-REQ-SEC-001]
- ✅ Automatic user profile creation via API [SRS-SRD-REQ-SEC-001]
- ✅ Protected routes with middleware [SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005]
- ✅ Client-side authentication state management [SRS-REQ-UI-001]
- ✅ Automatic redirect on login/logout [SRS-REQ-UI-001, SRS-REQ-UI-003]
- ✅ Loading states and error handling (basic) [SRS-REQ-UI-001]
- ✅ Token-based authentication for API calls [SRS-SRD-REQ-SEC-001]
- ✅ Basic dashboard layout with user profile display [SRS-REQ-UI-003, SRS-REQ-UI-006, SRS-REQ-UI-007]
- ✅ Responsive design foundation [SRS-REQ-UI-002, SRS-REQ-UI-018]
- ✅ Full navigation layout with sidebar [SRS-REQ-UI-004, SRS-REQ-UI-005]
- ✅ Mobile responsive navigation [SRS-REQ-UI-010, SRS-REQ-UI-011, SRS-REQ-UI-012]
- ✅ User profile dropdown menu [SRS-REQ-UI-008, SRS-REQ-UI-009]
- ✅ Global loading overlay system [SRS-REQ-UI-013, SRS-REQ-UI-014, SRS-REQ-UI-015, SRS-REQ-UI-016]
- ✅ Complete layout consistency [SRS-REQ-UI-017]

**📋 Notes on Implementation:**
- Role-based navigation (Observers menu for caretakers only) has placeholder comments
- User role will be added when user profile API includes role information
- All components follow the design specifications in the SRD
- Dependencies added: `zustand` (state management), `@headlessui/react` (accessible UI components)

---

### 1.1 Design Principles

- **Mobile-Friendly:** Support for mobile devices (not primary use case)
- **Accessibility:** WCAG 2.1 AA compliance
- **Performance:** Smooth interactions
- **Responsive:** Works across all device sizes

### 1.2 Technology Stack

- **Framework:** Next.js (App Router)
- **React:** React 19
- **Styling:** Tailwind CSS with PostCSS
- **Icons:** Heroicons (@heroicons/react)
- **State Management:** React Context API
- **Real-Time:** Firestore real-time listeners
- **Authentication:** Firebase Authentication (Google Sign-In)
- **API Client:** Custom fetch wrapper for Cloud Functions

---

## 2. Application Structure

**[Implements: SRD-REQ-FRONT-001, SRS-REQ-UI-003, SRS-REQ-UI-004, SRS-REQ-UI-017]**

### 2.1 Directory Structure

```
app/
├── layout.tsx                      # Root layout with providers
├── page.tsx                        # Landing page (unauthenticated)
├── globals.css                     # Global styles + Tailwind
├── components/                     # Reusable components
│   ├── ui/                        # Base UI components
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   ├── Modal.tsx
│   │   ├── Badge.tsx
│   │   ├── LoadingSpinner.tsx
│   │   └── ErrorMessage.tsx
│   ├── layout/                    # Layout components
│   │   ├── TopNavBar.tsx         # Top navigation bar
│   │   ├── LeftSideMenu.tsx      # Left sidebar (desktop/tablet)
│   │   ├── MobileNavMenu.tsx     # Mobile hamburger menu
│   │   ├── UserDropdownMenu.tsx  # User profile dropdown
│   │   ├── LoadingOverlay.tsx    # Global loading indicator
│   │   └── NavLink.tsx           # Navigation link component
│   ├── auth/                      # Authentication components
│   │   └── GoogleSignInButton.tsx # Google OAuth button
│   ├── PatientCard.tsx            # Patient profile card
│   ├── AlertCard.tsx              # Alert schedule card
│   ├── IncidentCard.tsx           # Active incident card
│   ├── EscalationPolicyEditor.tsx  # Escalation policy UI
│   ├── NotificationPreferences.tsx
│   ├── ObserverInviteForm.tsx
│   └── ScheduleEditor.tsx         # RRULE schedule editor
├── context/                        # React Context providers
│   ├── AuthContext.tsx            # Firebase Auth state
│   ├── LoadingContext.tsx         # Global loading state
│   ├── PatientContext.tsx         # Patient data
│   ├── AlertContext.tsx           # Alert management
│   └── IncidentContext.tsx        # Real-time incidents
├── hooks/                          # Custom React hooks
│   ├── useAuth.ts                 # Authentication hook
│   ├── useLoadingState.ts         # Loading state hook
│   └── useMobileMenu.ts           # Mobile menu state hook
├── (authenticated)/               # Route group for protected pages
│   ├── layout.tsx                 # Authenticated layout wrapper
│   ├── dashboard/                 # Caretaker dashboard
│   │   └── page.tsx
│   ├── patients/                  # Patient management
│   │   ├── page.tsx               # Patient list
│   │   ├── [id]/
│   │   │   └── page.tsx          # Patient detail
│   │   └── new/
│   │       └── page.tsx          # Create patient
│   ├── alerts/                    # Alert management
│   │   ├── page.tsx               # Alert list
│   │   ├── [id]/
│   │   │   └── page.tsx          # Alert detail
│   │   └── new/
│   │       └── page.tsx          # Create alert
│   ├── incidents/                 # Incident management
│   │   ├── page.tsx               # Active incidents
│   │   └── [id]/
│   │       └── page.tsx          # Incident detail
│   ├── observers/                 # Observer management (caretaker only)
│   │   ├── page.tsx               # Observer list
│   │   └── invite/
│   │       └── page.tsx          # Invite observer
│   └── settings/                  # Settings pages
│       ├── page.tsx               # Account settings
│       └── notifications/
│           └── page.tsx           # Notification preferences
├── onboarding/                     # Observer onboarding (public)
│   └── [token]/
│       ├── page.tsx              # Onboarding flow
│       └── complete/
│           └── page.tsx          # Completion page
├── ack/                            # Quick acknowledgment (public)
│   └── [token]/
│       └── page.tsx              # Acknowledge page
├── resolve/                        # Quick resolution (public)
│   └── [token]/
│       └── page.tsx              # Resolve page
├── login/                          # Authentication (public)
│   └── page.tsx
├── firebase/                       # Firebase config
│   └── config.ts
├── lib/                            # Utilities
│   ├── api-client.ts              # API client
│   └── utils.ts                   # Helper functions
└── middleware.ts                   # Route protection middleware
```

---

## 3. Application Layout Structure

**[Implements: SRD-REQ-FRONT-001, SRD-REQ-FRONT-010, SRS-REQ-UI-003, SRS-REQ-UI-017]**

### 3.1 Layout Overview

[Implements: SRD-REQ-FRONT-001, SRS-REQ-UI-017] The application SHALL implement a two-state layout system:

1. **Unauthenticated State:** Landing page with Google Sign-In
2. **Authenticated State:** Dashboard layout with navigation

### 3.2 Landing Page Layout (Unauthenticated)

[Implements: SRD-REQ-FRONT-008, SRD-REQ-FRONT-009, SRS-REQ-UI-001, SRS-REQ-UI-002] The landing page (`app/page.tsx`) SHALL implement:

```typescript
// app/page.tsx
'use client';

import { GoogleSignInButton } from '@/components/auth/GoogleSignInButton';
import { useAuth } from '@/hooks/useAuth';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';

export default function LandingPage() {
  const { user } = useAuth();
  const router = useRouter();
  
  // Redirect if already authenticated
  useEffect(() => {
    if (user) {
      router.push('/dashboard');
    }
  }, [user, router]);
  
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100">
      <div className="max-w-md w-full space-y-8 p-8 bg-white rounded-xl shadow-lg">
        {/* Header */}
        <div className="text-center">
          <h1 className="text-4xl font-bold text-gray-900">Family Pager</h1>
          <p className="mt-2 text-lg text-gray-600">
            Patient alert notification system for caretakers and observers
          </p>
        </div>
        
        {/* Google Sign-In Button */}
        <GoogleSignInButton />
        
        {/* Footer Text */}
        <div className="text-center text-sm text-gray-500">
          <p>Sign in with your Google account to get started</p>
        </div>
      </div>
    </div>
  );
}
```

**Design Requirements:**
- Centered layout on all screen sizes
- Responsive from mobile (320px) to desktop (1920px+)
- No navigation elements visible
- Clean, modern healthcare-focused design
- Fast loading (< 1.5s FCP)

### 3.3 Authenticated Layout Structure

[Implements: SRD-REQ-FRONT-010, SRS-REQ-UI-003, SRS-REQ-UI-004, SRS-REQ-UI-017] The authenticated layout SHALL be implemented as a route group:

```typescript
// app/(authenticated)/layout.tsx
'use client';

import { TopNavBar } from '@/components/layout/TopNavBar';
import { LeftSideMenu } from '@/components/layout/LeftSideMenu';
import { MobileNavMenu } from '@/components/layout/MobileNavMenu';
import { LoadingOverlay } from '@/components/layout/LoadingOverlay';
import { useAuth } from '@/hooks/useAuth';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';

export default function AuthenticatedLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const { user, loading } = useAuth();
  const router = useRouter();
  
  // Redirect if not authenticated
  useEffect(() => {
    if (!loading && !user) {
      router.push('/');
    }
  }, [user, loading, router]);
  
  if (loading) {
    return <LoadingOverlay />;
  }
  
  if (!user) {
    return null;
  }
  
  return (
    <div className="min-h-screen flex flex-col">
      {/* Top Navigation Bar - Always visible */}
      <TopNavBar />
      
      <div className="flex flex-1 overflow-hidden">
        {/* Left Sidebar - Desktop/Tablet only (≥768px) */}
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

### 3.4 Top Navigation Bar Component

[Implements: SRD-REQ-FRONT-013, SRD-REQ-FRONT-014, SRS-REQ-UI-006, SRS-REQ-UI-007, SRS-REQ-UI-010] The TopNavBar component SHALL implement:

```typescript
// app/components/layout/TopNavBar.tsx
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { Bars3Icon, ChevronDownIcon } from '@heroicons/react/24/outline';
import { useAuth } from '@/hooks/useAuth';
import { useMobileMenu } from '@/hooks/useMobileMenu';
import { UserDropdownMenu } from './UserDropdownMenu';

export function TopNavBar() {
  const { user } = useAuth();
  const { openMenu } = useMobileMenu();
  const [showDropdown, setShowDropdown] = useState(false);
  
  return (
    <header className="sticky top-0 z-50 w-full border-b border-gray-200 bg-white">
      <div className="flex items-center justify-between h-16 px-4">
        {/* Left Section */}
        <div className="flex items-center space-x-4">
          {/* Hamburger Menu - Mobile Only (<768px) */}
          <button
            onClick={openMenu}
            className="md:hidden p-2 rounded-md hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-indigo-500"
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
              className="flex items-center space-x-3 p-2 rounded-lg hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-indigo-500"
              aria-label="User menu"
              aria-expanded={showDropdown}
            >
              {/* Google Profile Picture */}
              <img
                src={user?.photoURL || '/default-avatar.png'}
                alt={user?.displayName || 'User'}
                className="h-8 w-8 rounded-full border-2 border-gray-200"
              />
              
              {/* User Name - Desktop Only (≥768px) */}
              <span className="hidden md:block text-sm font-medium text-gray-700">
                {user?.displayName}
              </span>
              
              {/* Dropdown Indicator */}
              <ChevronDownIcon className="h-4 w-4 text-gray-500" />
            </button>
            
            {/* Dropdown Menu */}
            {showDropdown && (
              <UserDropdownMenu onClose={() => setShowDropdown(false)} />
            )}
          </div>
        </div>
      </div>
    </header>
  );
}
```

**Design Requirements:**
- Fixed at top of viewport (sticky positioning)
- Minimum height: 64px (4rem) for adequate touch targets
- Spans full width of viewport
- Z-index: 50 to stay above content
- Accessible keyboard navigation with focus indicators

### 3.5 Left Side Menu Component

[Implements: SRD-REQ-FRONT-011, SRD-REQ-FRONT-012, SRS-REQ-UI-004, SRS-REQ-UI-005, SRS-REQ-UI-017, SRS-REQ-SEC-005] The LeftSideMenu component SHALL implement:

```typescript
// app/components/layout/LeftSideMenu.tsx
'use client';

import { usePathname } from 'next/navigation';
import { 
  HomeIcon, 
  UserGroupIcon, 
  BellIcon, 
  ExclamationTriangleIcon,
  UsersIcon,
  CogIcon 
} from '@heroicons/react/24/outline';
import { useAuth } from '@/hooks/useAuth';
import { NavLink } from './NavLink';

export function LeftSideMenu() {
  const pathname = usePathname();
  const { user, userRole } = useAuth();
  
  return (
    <aside className="hidden md:flex md:flex-shrink-0 w-64 bg-white border-r border-gray-200">
      <div className="flex flex-col w-full">
        {/* Navigation Links */}
        <nav className="flex-1 px-4 py-6 space-y-2" aria-label="Main navigation">
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
          
          {/* Caretaker-only navigation */}
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

**Design Requirements:**
- Visible only on viewports ≥768px (md breakpoint)
- Fixed width: 256px (64rem / w-64)
- Full height of viewport below top nav
- Background: white with right border
- Role-based navigation items (Observers for caretakers only)

### 3.6 Navigation Link Component

[Implements: SRD-REQ-FRONT-012, SRS-REQ-UI-005] The NavLink component SHALL implement:

```typescript
// app/components/layout/NavLink.tsx
'use client';

import Link from 'next/link';
import { ReactNode } from 'react';

interface NavLinkProps {
  href: string;
  icon: React.ComponentType<{ className?: string }>;
  active: boolean;
  children: ReactNode;
}

export function NavLink({ href, icon: Icon, active, children }: NavLinkProps) {
  return (
    <Link
      href={href}
      className={`
        flex items-center space-x-3 px-4 py-3 rounded-lg transition-colors
        ${active 
          ? 'bg-indigo-50 text-indigo-600 font-medium' 
          : 'text-gray-700 hover:bg-gray-100'
        }
      `}
      aria-current={active ? 'page' : undefined}
    >
      <Icon className="h-5 w-5 flex-shrink-0" />
      <span>{children}</span>
    </Link>
  );
}
```

**Design Requirements:**
- Active state: indigo background with indigo text
- Hover state: light gray background
- Icon size: 20px (h-5 w-5)
- Touch target size: minimum 44px height
- Proper ARIA attributes for screen readers

### 3.7 Mobile Navigation Menu

[Implements: SRD-REQ-FRONT-017, SRD-REQ-FRONT-018, SRS-REQ-UI-010, SRS-REQ-UI-011, SRS-REQ-UI-012, SRS-REQ-UI-018] The MobileNavMenu component SHALL implement:

```typescript
// app/components/layout/MobileNavMenu.tsx
'use client';

import { Fragment } from 'react';
import { Dialog, Transition } from '@headlessui/react';
import { XMarkIcon } from '@heroicons/react/24/outline';
import { useMobileMenu } from '@/hooks/useMobileMenu';
import { usePathname } from 'next/navigation';
import { NavLink } from './NavLink';
import {
  HomeIcon,
  UserGroupIcon,
  BellIcon,
  ExclamationTriangleIcon,
  UsersIcon,
  CogIcon
} from '@heroicons/react/24/outline';
import { useAuth } from '@/hooks/useAuth';

export function MobileNavMenu() {
  const { isOpen, closeMenu } = useMobileMenu();
  const pathname = usePathname();
  const { userRole } = useAuth();
  
  return (
    <Transition.Root show={isOpen} as={Fragment}>
      <Dialog as="div" className="relative z-50 md:hidden" onClose={closeMenu}>
        {/* Backdrop */}
        <Transition.Child
          as={Fragment}
          enter="transition-opacity ease-linear duration-300"
          enterFrom="opacity-0"
          enterTo="opacity-100"
          leave="transition-opacity ease-linear duration-300"
          leaveFrom="opacity-100"
          leaveTo="opacity-0"
        >
          <div className="fixed inset-0 bg-black bg-opacity-50" />
        </Transition.Child>

        {/* Menu Panel */}
        <div className="fixed inset-0 flex">
          <Transition.Child
            as={Fragment}
            enter="transition ease-in-out duration-300 transform"
            enterFrom="-translate-x-full"
            enterTo="translate-x-0"
            leave="transition ease-in-out duration-300 transform"
            leaveFrom="translate-x-0"
            leaveTo="-translate-x-full"
          >
            <Dialog.Panel className="relative mr-16 flex w-full max-w-xs flex-1">
              <div className="flex grow flex-col gap-y-5 overflow-y-auto bg-white px-6 pb-4">
                {/* Header */}
                <div className="flex h-16 shrink-0 items-center justify-between">
                  <span className="text-lg font-bold text-indigo-600">Menu</span>
                  <button
                    type="button"
                    className="p-2 rounded-md text-gray-700 hover:bg-gray-100"
                    onClick={closeMenu}
                  >
                    <span className="sr-only">Close sidebar</span>
                    <XMarkIcon className="h-6 w-6" aria-hidden="true" />
                  </button>
                </div>
                
                {/* Navigation - Same as desktop */}
                <nav className="flex flex-1 flex-col space-y-2">
                  <div onClick={closeMenu}>
                    <NavLink href="/dashboard" icon={HomeIcon} active={pathname === '/dashboard'}>
                      Dashboard
                    </NavLink>
                  </div>
                  
                  <div onClick={closeMenu}>
                    <NavLink href="/patients" icon={UserGroupIcon} active={pathname.startsWith('/patients')}>
                      Patients
                    </NavLink>
                  </div>
                  
                  <div onClick={closeMenu}>
                    <NavLink href="/alerts" icon={BellIcon} active={pathname.startsWith('/alerts')}>
                      Alerts
                    </NavLink>
                  </div>
                  
                  <div onClick={closeMenu}>
                    <NavLink href="/incidents" icon={ExclamationTriangleIcon} active={pathname.startsWith('/incidents')}>
                      Incidents
                    </NavLink>
                  </div>
                  
                  {userRole === 'caretaker' && (
                    <div onClick={closeMenu}>
                      <NavLink href="/observers" icon={UsersIcon} active={pathname.startsWith('/observers')}>
                        Observers
                      </NavLink>
                    </div>
                  )}
                  
                  <div onClick={closeMenu}>
                    <NavLink href="/settings" icon={CogIcon} active={pathname === '/settings'}>
                      Settings
                    </NavLink>
                  </div>
                </nav>
              </div>
            </Dialog.Panel>
          </Transition.Child>
        </div>
      </Dialog>
    </Transition.Root>
  );
}
```

**Design Requirements:**
- Visible only on mobile (<768px)
- Slide-in animation from left (300ms duration)
- Semi-transparent backdrop (rgba(0, 0, 0, 0.5))
- Menu width: max 320px (max-w-xs), taking 80vw on smaller screens
- Tap outside to close
- Prevent body scroll when open
- Close after navigation selection

### 3.8 User Dropdown Menu

[Implements: SRD-REQ-FRONT-015, SRD-REQ-FRONT-016, SRS-REQ-UI-008, SRS-REQ-UI-009, SRS-REQ-UI-017] The UserDropdownMenu component SHALL implement:

```typescript
// app/components/layout/UserDropdownMenu.tsx
'use client';

import { Fragment } from 'react';
import { Menu, Transition } from '@headlessui/react';
import { useRouter } from 'next/navigation';
import { useAuth } from '@/hooks/useAuth';
import {
  UserCircleIcon,
  BellAlertIcon,
  QuestionMarkCircleIcon,
  ArrowRightOnRectangleIcon
} from '@heroicons/react/24/outline';

interface UserDropdownMenuProps {
  onClose: () => void;
}

export function UserDropdownMenu({ onClose }: UserDropdownMenuProps) {
  const { user, signOut } = useAuth();
  const router = useRouter();
  
  const handleNavigation = (path: string) => {
    router.push(path);
    onClose();
  };
  
  const handleLogout = async () => {
    await signOut();
    onClose();
  };
  
  return (
    <div className="absolute right-0 mt-2 w-56 rounded-md shadow-lg bg-white ring-1 ring-black ring-opacity-5 divide-y divide-gray-200">
      {/* User Info Section */}
      <div className="px-4 py-3">
        <p className="text-sm font-medium text-gray-900">{user?.displayName}</p>
        <p className="text-sm text-gray-500 truncate">{user?.email}</p>
      </div>
      
      {/* Menu Items */}
      <div className="py-1">
        <button
          onClick={() => handleNavigation('/settings')}
          className="flex w-full items-center px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
        >
          <UserCircleIcon className="mr-3 h-5 w-5 text-gray-400" />
          Account Settings
        </button>
        
        <button
          onClick={() => handleNavigation('/settings/notifications')}
          className="flex w-full items-center px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
        >
          <BellAlertIcon className="mr-3 h-5 w-5 text-gray-400" />
          Notification Preferences
        </button>
        
        <button
          onClick={() => handleNavigation('/help')}
          className="flex w-full items-center px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
        >
          <QuestionMarkCircleIcon className="mr-3 h-5 w-5 text-gray-400" />
          Help & Documentation
        </button>
      </div>
      
      {/* Logout Section */}
      <div className="py-1">
        <button
          onClick={handleLogout}
          className="flex w-full items-center px-4 py-2 text-sm text-red-600 hover:bg-red-50"
        >
          <ArrowRightOnRectangleIcon className="mr-3 h-5 w-5 text-red-500" />
          Sign Out
        </button>
      </div>
    </div>
  );
}
```

**Design Requirements:**
- Dropdown positioned below profile icon
- Width: 224px (w-56)
- Sections separated by dividers
- Logout in red with hover state
- Click outside to close (handled by Headless UI)

### 3.9 Loading Overlay Component

[Implements: SRD-REQ-FRONT-019, SRD-REQ-FRONT-020, SRS-REQ-UI-013, SRS-REQ-UI-014, SRS-REQ-UI-015] The LoadingOverlay component SHALL implement:

```typescript
// app/components/layout/LoadingOverlay.tsx
'use client';

import { useLoadingState } from '@/hooks/useLoadingState';
import { useEffect, useState } from 'react';

export function LoadingOverlay() {
  const { isLoading, loadingMessage } = useLoadingState();
  const [shouldDisplay, setShouldDisplay] = useState(false);
  
  useEffect(() => {
    let timer: NodeJS.Timeout;
    
    if (isLoading) {
      // Show after 200ms to avoid flashing on fast operations
      timer = setTimeout(() => {
        setShouldDisplay(true);
      }, 200);
    } else {
      // Hide immediately when loading completes
      setShouldDisplay(false);
    }
    
    return () => {
      if (timer) clearTimeout(timer);
    };
  }, [isLoading]);
  
  if (!shouldDisplay) return null;
  
  return (
    <div
      className="fixed inset-0 z-[9999] flex items-center justify-center"
      style={{
        backgroundColor: 'rgba(0, 0, 0, 0.5)', // Semi-transparent overlay
      }}
      aria-live="assertive"
      aria-busy="true"
      role="status"
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

**Design Requirements:**
- Full-screen overlay blocking all interaction
- Semi-transparent black background (rgba(0, 0, 0, 0.5))
- Z-index: 9999 to be above all content
- Centered spinner and message
- Delay 200ms before showing (avoid flashing)
- Proper ARIA attributes for accessibility
- Display minimum 300ms once shown

### 3.10 Loading Context Provider

[Implements: SRD-REQ-FRONT-021, SRS-REQ-UI-013, SRS-REQ-UI-015] The LoadingContext SHALL manage global loading state:

```typescript
// app/context/LoadingContext.tsx
'use client';

import { createContext, useContext, useState, useCallback, ReactNode } from 'react';

interface LoadingContextType {
  isLoading: boolean;
  loadingMessage: string | null;
  startLoading: (message?: string) => void;
  stopLoading: () => void;
}

const LoadingContext = createContext<LoadingContextType | undefined>(undefined);

export function LoadingProvider({ children }: { children: ReactNode }) {
  const [isLoading, setIsLoading] = useState(false);
  const [loadingMessage, setLoadingMessage] = useState<string | null>(null);
  const [minDisplayTimer, setMinDisplayTimer] = useState<NodeJS.Timeout | null>(null);
  
  const startLoading = useCallback((message?: string) => {
    setIsLoading(true);
    setLoadingMessage(message || null);
    
    // Ensure minimum 300ms display time
    const timer = setTimeout(() => {
      setMinDisplayTimer(null);
    }, 300);
    setMinDisplayTimer(timer);
  }, []);
  
  const stopLoading = useCallback(() => {
    if (minDisplayTimer) {
      // Wait for minimum display time
      setTimeout(() => {
        setIsLoading(false);
        setLoadingMessage(null);
      }, 300);
    } else {
      setIsLoading(false);
      setLoadingMessage(null);
    }
  }, [minDisplayTimer]);
  
  return (
    <LoadingContext.Provider value={{ isLoading, loadingMessage, startLoading, stopLoading }}>
      {children}
    </LoadingContext.Provider>
  );
}

export function useLoadingState() {
  const context = useContext(LoadingContext);
  if (context === undefined) {
    throw new Error('useLoadingState must be used within a LoadingProvider');
  }
  return context;
}
```

### 3.11 Mobile Menu Hook

[Implements: SRD-REQ-FRONT-017] The useMobileMenu hook SHALL manage mobile menu state:

```typescript
// app/hooks/useMobileMenu.ts
'use client';

import { create } from 'zustand';

interface MobileMenuState {
  isOpen: boolean;
  openMenu: () => void;
  closeMenu: () => void;
  toggleMenu: () => void;
}

export const useMobileMenu = create<MobileMenuState>((set) => ({
  isOpen: false,
  openMenu: () => set({ isOpen: true }),
  closeMenu: () => set({ isOpen: false }),
  toggleMenu: () => set((state) => ({ isOpen: !state.isOpen })),
}));
```

### 3.12 Route Protection Middleware

[Implements: SRD-REQ-FRONT-006, SRD-REQ-FRONT-007, SRS-SRD-REQ-SEC-001, SRS-REQ-SEC-005] The middleware SHALL protect authenticated routes:

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const authToken = request.cookies.get('authToken');
  const path = request.nextUrl.pathname;
  
  // Public routes that don't require authentication
  const publicRoutes = ['/', '/login', '/onboarding', '/ack', '/resolve'];
  const isPublicRoute = publicRoutes.some(route => path.startsWith(route));
  
  // Authenticated routes (everything under /(authenticated))
  const isAuthenticatedRoute = path.startsWith('/dashboard') || 
                                path.startsWith('/patients') ||
                                path.startsWith('/alerts') ||
                                path.startsWith('/incidents') ||
                                path.startsWith('/observers') ||
                                path.startsWith('/settings');
  
  // Redirect to landing if trying to access authenticated route without token
  if (isAuthenticatedRoute && !authToken) {
    return NextResponse.redirect(new URL('/', request.url));
  }
  
  // Redirect to dashboard if trying to access landing page while authenticated
  if (path === '/' && authToken) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

---

## 4. Page Requirements

**[Implements: SRD-REQ-FRONT-001, SRS-REQ-UI-003]**

### 4.1 Landing Page (`/`)

[Implements: SRD-REQ-FRONT-008, SRD-REQ-FRONT-009, SRS-REQ-UI-001] The landing page is defined in section 3.2 above. Additional requirements:

**Features:**
- Automatic redirect to `/dashboard` if user is already authenticated
- Google Sign-In button using Firebase Authentication
- Responsive layout across all device sizes
- Loading state during authentication check

**Error Handling:**
- Display authentication errors in user-friendly format
- Handle Google OAuth popup cancellation
- Network error recovery with retry option

**Design:**
- Clean, modern healthcare-focused design
- Centered card layout with gradient background
- Fast loading (< 1.5s FCP)
- No navigation elements (not authenticated)

---

### 4.2 Caretaker Dashboard (`/dashboard`)

[Implements: SRD-REQ-FRONT-005, SRS-REQ-INC-002, SRS-REQ-INC-003] The caretaker dashboard SHALL be wrapped in the authenticated layout and display:

- **Summary Cards:**
  - Total patients count
  - Active alerts count
  - Active incidents count
  - Total observers count

- **Active Incidents Section:**
  - List of active incidents (triggered, acknowledged)
  - Patient name, alert name, severity, status
  - Quick actions (view, resolve)
  - Real-time updates via Firestore listener

- **Recent Patients:**
  - List of recently accessed patients
  - Quick access to patient detail

- **Quick Actions:**
  - "Add Patient" button
  - "Create Alert" button
  - "Invite Observer" button

**Layout:**
- Responsive grid layout
- Mobile: Single column
- Tablet: 2 columns
- Desktop: 3-4 columns

**Real-Time Updates:**
```typescript
// Example: Real-time incident updates
useEffect(() => {
  const unsubscribe = onSnapshot(
    query(
      collection(db, 'incidents'),
      where('status', 'in', ['triggered', 'acknowledged']),
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

---

### 4.3 Patient List (`/patients`)

[Implements: SRS-REQ-PATIENT-001] The patient list page SHALL display:

- **Header:**
  - "Patients" title
  - "Add Patient" button
  - Search/filter input

- **Patient Cards:**
  - Patient name and photo
  - Status badge (active, inactive, archived)
  - Active alerts count
  - Active incidents count
  - Observers count
  - Last updated timestamp
  - Quick actions (view, edit, archive)

- **Empty State:**
  - Message when no patients
  - "Add Your First Patient" CTA

**Features:**
- Search by patient name
- Filter by status
- Pagination (if > 50 patients)
- Click card to navigate to detail

---

### 4.4 Patient Detail (`/patients/[id]`)

The patient detail page SHALL include:

- **Patient Header:**
  - Patient name, photo, status
  - Edit button
  - Archive button

- **Tabs:**
  - Overview
  - Alerts
  - Observers
  - Incidents
  - Escalation Policy

- **Overview Tab:**
  - Patient description
  - Key metrics (alerts, incidents, observers)
  - Recent activity

- **Alerts Tab:**
  - List of all alerts for patient
  - Create alert button
  - Alert status and next trigger time

- **Observers Tab:**
  - List of assigned observers
  - Invite observer button
  - Observer role and status

- **Incidents Tab:**
  - Incident history
  - Filter by status
  - Incident details

- **Escalation Policy Tab:**
  - Current escalation policy
  - Edit policy button
  - Visual policy editor

**Real-Time Updates:**
- Active incidents count
- Alert status changes
- Observer assignments

---

### 4.5 Create Patient (`/patients/new`)

The create patient page SHALL provide:

- **Form Fields:**
  - Patient name (required, max 200 chars)
  - Description (optional, max 1000 chars, textarea)
  - Photo URL (optional, URL validation)
  - Status (default: active)

- **Form Validation:**
  - Client-side validation
  - Error messages below fields
  - Disable submit until valid

- **Actions:**
  - "Cancel" button (navigate back)
  - "Create Patient" button (submit form)

- **Success Flow:**
  - Show success message
  - Redirect to patient detail page

---

### 4.6 Alert List (`/alerts`)

The alert list page SHALL display:

- **Header:**
  - "Alerts" title
  - "Create Alert" button
  - Filter by patient, status

- **Alert Cards:**
  - Alert name
  - Patient name
  - Schedule (human-readable)
  - Next trigger time
  - Severity badge
  - Status badge
  - Quick actions (view, pause, delete)

- **Empty State:**
  - Message when no alerts
  - "Create Your First Alert" CTA

---

### 4.7 Create Alert (`/alerts/new`)

The create alert page SHALL provide:

- **Form Fields:**
  - Patient selection (required, dropdown)
  - Alert name (required, max 200 chars)
  - Alert message (required, max 1000 chars, textarea)
  - Severity (required, radio: low, medium, high, critical)
  - Schedule editor (required)
  - Auto-resolve timeout (optional, minutes)

- **Schedule Editor:**
  - Visual schedule builder
  - RRULE preview
  - Timezone selection
  - Next occurrence preview

- **Form Validation:**
  - Validate RRULE syntax
  - Show schedule preview
  - Validate all required fields

- **Success Flow:**
  - Show success message
  - Redirect to alert detail page

---

### 4.8 Incident List (`/incidents`)

The incident list page SHALL display:

- **Header:**
  - "Active Incidents" title
  - Filter by status, patient, severity

- **Incident Cards:**
  - Incident number
  - Patient name
  - Alert name
  - Severity badge
  - Status badge (triggered, acknowledged, resolved)
  - Current escalation level
  - Assigned observer
  - Triggered timestamp
  - Quick actions (view, acknowledge, resolve)

- **Real-Time Updates:**
  - Live status updates
  - New incidents appear automatically
  - Status changes reflected immediately

---

### 4.9 Incident Detail (`/incidents/[id]`)

The incident detail page SHALL include:

- **Incident Header:**
  - Incident number
  - Patient name
  - Alert name
  - Status badge
  - Severity badge

- **Timeline:**
  - Triggered timestamp
  - Acknowledged timestamp (if applicable)
  - Resolved timestamp (if applicable)
  - Escalation events

- **Actions:**
  - Acknowledge button (if triggered)
  - Resolve button (if acknowledged)
  - Add note button
  - Reassign button (caretaker only)

- **Action History:**
  - List of all actions
  - Actor, action type, timestamp
  - Notes and details

**Real-Time Updates:**
- Status changes
- New actions
- Escalation events

---

### 4.10 Observer Onboarding (`/onboarding/[token]`)

The onboarding flow SHALL include multiple steps:

**Step 1: Invitation Landing**
- Display patient name(s)
- Display caretaker name
- Show observer role(s)
- Terms of service and privacy policy checkboxes
- "Accept and Continue" button

**Step 2: Account Creation**
- "Sign in with Google" button
- Google OAuth authentication flow
- Automatic profile creation from Google account
- Phone number input (E.164 format, required for SMS notifications)
- Phone number verification via SMS code

**Step 3: Phone Verification**
- Phone verification: SMS code input
- "Verify Phone" button
- Email is automatically verified via Google account

**Step 4: Notification Preferences**
- Primary notification method (SMS, Email, Both)
- Quiet hours toggle and time selection
- High-urgency override toggle
- "Continue" button

**Step 5: Patient Context**
- Patient information display
- Escalation policy preview
- Example alert notification preview
- "Continue" button

**Step 6: Confirmation**
- Summary of setup
- Test notification button
- "Complete Setup" button

**Success Flow:**
- Show success message
- Redirect to observer dashboard

---

### 4.11 Quick Acknowledge (`/ack/[token]`)

The acknowledgment page SHALL provide:

- **Incident Information:**
  - Incident number
  - Patient name
  - Alert name
  - Severity
  - Triggered time

- **Actions:**
  - "I'll Handle This" button (acknowledge)
  - "View Details" link
  - "Not My Responsibility" link (if applicable)

- **Success State:**
  - Confirmation message
  - "You're assigned to this incident"
  - "Resolve" button (if applicable)

**Design:**
- Mobile-optimized
- Large, easy-to-tap buttons
- Clear, simple layout

---

### 4.12 Quick Resolve (`/resolve/[token]`)

The resolution page SHALL provide:

- **Incident Information:**
  - Incident number
  - Patient name
  - Alert name
  - Current status

- **Resolution Form:**
  - Note input (optional, textarea)
  - "Mark as Resolved" button

- **Success State:**
  - Confirmation message
  - "Thank you for responding"

---

### 4.13 Observer Dashboard (`/observer/dashboard`)

The observer dashboard SHALL display:

- **My Patients:**
  - List of assigned patients
  - Role for each patient
  - Active incidents count per patient

- **Active Incidents:**
  - List of incidents assigned to observer
  - Patient name, alert name, severity
  - Quick acknowledge/resolve actions

- **Notification Preferences:**
  - Quick access to preferences
  - Current settings summary

**Real-Time Updates:**
- New incident assignments
- Status changes
- Patient updates

---

## 5. Component Requirements

**[Implements: SRD-REQ-FRONT-003, SRS-REQ-UI-003, SRS-REQ-UI-013, SRS-REQ-UI-016]**

### 5.1 Base UI Components

[Implements: SRD-REQ-FRONT-003, SRS-REQ-UI-013, SRS-REQ-UI-016] The following base UI components SHALL be implemented:

**Button Component:**
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger' | 'ghost';
  size: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}
```

**Input Component:**
```typescript
interface InputProps {
  type?: 'text' | 'email' | 'tel' | 'number';
  label?: string;
  error?: string;
  required?: boolean;
  disabled?: boolean;
  value: string;
  onChange: (value: string) => void;
}
```

**Card Component:**
```typescript
interface CardProps {
  title?: string;
  subtitle?: string;
  actions?: React.ReactNode;
  children: React.ReactNode;
  onClick?: () => void;
}
```

**Badge Component:**
```typescript
interface BadgeProps {
  variant: 'success' | 'warning' | 'error' | 'info';
  children: React.ReactNode;
}
```

---

### 5.2 Patient Card Component

[Implements: SRD-REQ-FRONT-003, SRS-REQ-PATIENT-001] The PatientCard component SHALL display:

- Patient photo (or placeholder)
- Patient name
- Status badge
- Active alerts count
- Active incidents count
- Observers count
- Click handler for navigation

**Props:**
```typescript
interface PatientCardProps {
  patient: Patient;
  onClick?: (patientId: string) => void;
}
```

---

### 5.3 Alert Card Component

The AlertCard component SHALL display:

- Alert name
- Patient name
- Schedule (human-readable)
- Next trigger time
- Severity badge
- Status badge
- Quick actions (pause, delete)

**Props:**
```typescript
interface AlertCardProps {
  alert: Alert;
  onPause?: (alertId: string) => void;
  onDelete?: (alertId: string) => void;
  onClick?: (alertId: string) => void;
}
```

---

### 5.4 Incident Card Component

[Implements: SRD-REQ-FRONT-003, SRD-REQ-FRONT-005, SRS-REQ-INC-001, SRS-REQ-ESC-003] The IncidentCard component SHALL display:

- Incident number
- Patient name
- Alert name
- Severity badge
- Status badge
- Current escalation level
- Assigned observer
- Triggered timestamp
- Quick actions (acknowledge, resolve)

**Props:**
```typescript
interface IncidentCardProps {
  incident: Incident;
  onAcknowledge?: (incidentId: string) => void;
  onResolve?: (incidentId: string) => void;
  onClick?: (incidentId: string) => void;
}
```

---

### 5.5 Escalation Policy Editor Component

[Implements: SRD-REQ-FRONT-003, SRS-REQ-ESC-001, SRS-REQ-ESC-005] The EscalationPolicyEditor component SHALL provide:

- **Visual Editor:**
  - Add/remove escalation levels
  - Drag-and-drop level reordering
  - Observer selection per level
  - Timeout configuration per level

- **Level Configuration:**
  - Level number
  - Target observers (multi-select)
  - Timeout in minutes
  - "Notify all" toggle

- **Repeat Policy:**
  - Enable/disable toggle
  - Max repeats input
  - Restart from level selection

**Props:**
```typescript
interface EscalationPolicyEditorProps {
  policy?: EscalationPolicy;
  patientId: string;
  availableObservers: Observer[];
  onSave: (policy: EscalationPolicy) => Promise<void>;
}
```

---

### 5.6 Schedule Editor Component

[Implements: SRS-REQ-ALERT-002, SRS-REQ-ALERT-003, SRS-REQ-SCHED-001, SRS-REQ-SCHED-002, SRS-REQ-SCHED-004, SRS-REQ-SCHED-006] The ScheduleEditor component SHALL provide:

- **Visual Schedule Builder:**
  - Frequency selection (daily, weekly, monthly, hourly)
  - Time selection (hour, minute)
  - Day selection (for weekly)
  - Date selection (for monthly)
  - Interval input (for hourly)

- **RRULE Preview:**
  - Display generated RRULE
  - Human-readable schedule description
  - Next occurrence preview

- **Timezone Selection:**
  - Timezone dropdown
  - Current timezone display

**Props:**
```typescript
interface ScheduleEditorProps {
  value: {
    rrule: string;
    timezone: string;
  };
  onChange: (schedule: Schedule) => void;
  error?: string;
}
```

---

## 6. State Management

**[Implements: SRD-REQ-FRONT-004, SRS-REQ-UI-003, SRS-REQ-UI-017]**

### 6.1 Context Providers

[Implements: SRD-REQ-FRONT-004, SRD-REQ-FRONT-005, SRS-REQ-INC-003] The following context providers SHALL be implemented:

**AuthContext:**
```typescript
interface AuthContextValue {
  user: User | null;
  loading: boolean;
  signInWithGoogle: () => Promise<void>;
  signOut: () => Promise<void>;
  // User profile automatically created from Google account
}
```

**PatientContext:**
```typescript
interface PatientContextValue {
  patients: Patient[];
  loading: boolean;
  createPatient: (data: CreatePatientData) => Promise<Patient>;
  updatePatient: (id: string, data: UpdatePatientData) => Promise<void>;
  deletePatient: (id: string) => Promise<void>;
  refreshPatients: () => Promise<void>;
}
```

**IncidentContext:**
```typescript
interface IncidentContextValue {
  incidents: Incident[];
  loading: boolean;
  acknowledgeIncident: (id: string) => Promise<void>;
  resolveIncident: (id: string, note?: string) => Promise<void>;
  // Real-time updates via Firestore listener
}
```

---

## 7. Google Authentication Implementation

**[Implements: SRS-SRD-REQ-SEC-001, SRD-REQ-FRONT-006, SRD-REQ-FRONT-007]**

### 7.1 Firebase Google Auth Setup

[Implements: SRS-SRD-REQ-SEC-001, SRD-REQ-DEV-004] Google Authentication SHALL be implemented using Firebase Auth:

```typescript
// app/firebase/config.ts
import { initializeApp, getApps, getApp } from 'firebase/app';
import { getAuth, GoogleAuthProvider } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  // ... other config
};

const app = !getApps().length ? initializeApp(firebaseConfig) : getApp();
const auth = getAuth(app);
const googleProvider = new GoogleAuthProvider();

export { auth, googleProvider, db };
```

### 7.2 Sign-In Implementation

[Implements: SRS-SRD-REQ-SEC-001] Google Sign-In SHALL be implemented:

```typescript
// app/context/AuthContext.tsx
import { signInWithPopup, signOut as firebaseSignOut } from 'firebase/auth';
import { auth, googleProvider } from '../firebase/config';

const signInWithGoogle = async () => {
  try {
    const result = await signInWithPopup(auth, googleProvider);
    const user = result.user;
    
    // User profile automatically available from Google account
    // Create/update user profile in Firestore
    await createOrUpdateUserProfile(user);
    
    return user;
  } catch (error) {
    console.error('Google sign-in error:', error);
    throw error;
  }
};
```

### 7.3 User Profile Creation

User profiles SHALL be automatically created from Google account:

- Extract email, name, photo from Google account
- Create caretaker or observer profile in Firestore
- Store Google UID for reference
- Phone number collected separately during onboarding (observers)

---

## 8. API Client

### 8.1 API Client Implementation

**SRS-REQ-API-001:** The API client SHALL provide:

- **Base Configuration:**
  - Base URL from environment
  - Automatic token injection
  - Error handling
  - Request/response logging (dev only)

- **Methods:**
```typescript
class ApiClient {
  async get<T>(endpoint: string, params?: Record<string, string>): Promise<T>;
  async post<T>(endpoint: string, data?: any): Promise<T>;
  async put<T>(endpoint: string, data?: any): Promise<T>;
  async delete<T>(endpoint: string): Promise<T>;
}
```

- **Error Handling:**
  - Network errors
  - Authentication errors (redirect to login)
  - Validation errors (display user-friendly messages)
  - Rate limit errors (show retry suggestion)

---

## 9. Styling Requirements

**[Implements: SRD-REQ-FRONT-002, SRS-REQ-UI-002, SRS-REQ-UI-018]**

### 9.1 Tailwind Configuration

[Implements: SRD-REQ-FRONT-002, SRS-REQ-UI-002] Tailwind CSS SHALL be configured with:

- **Custom Colors:**
  - Primary (healthcare blue)
  - Secondary (gray)
  - Success (green)
  - Warning (yellow)
  - Error (red)
  - Info (blue)

- **Typography:**
  - Font family: Inter (Google Fonts)
  - Font sizes: xs, sm, base, lg, xl, 2xl, 3xl
  - Font weights: normal, medium, semibold, bold

- **Spacing:**
  - Consistent spacing scale
  - Mobile-friendly padding/margins

- **Breakpoints:**
  - sm: 640px
  - md: 768px
  - lg: 1024px
  - xl: 1280px

---

### 9.2 Design System

[Implements: SRD-REQ-FRONT-002, SRD-REQ-FRONT-024, SRS-REQ-UI-002, SRS-REQ-UI-018] The design system SHALL include:

- **Color Palette:**
  - Primary: Blue (#3B82F6)
  - Success: Green (#10B981)
  - Warning: Yellow (#F59E0B)
  - Error: Red (#EF4444)
  - Gray scale for text and backgrounds

- **Component Styles:**
  - Consistent button styles
  - Form input styles
  - Card styles
  - Badge styles

- **Responsive Design:**
  - Mobile-first approach
  - Breakpoint-based layouts
  - Touch-friendly targets (min 44x44px)

---

## 10. Performance Requirements

**[Implements: SRS-SRD-REQ-PERF-001, SRS-SRD-REQ-PERF-002, SRS-SRD-REQ-PERF-003]**

### 10.1 Loading Performance

**SRS-SRD-REQ-PERF-001:** [Implements: SRS-SRD-REQ-PERF-001, SRS-SRD-REQ-PERF-002, SRS-SRD-REQ-PERF-003] The application SHALL meet:

- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Lighthouse Performance Score: > 90

**Optimization Strategies:**
- Static site generation for public pages
- Code splitting per route
- Image optimization
- Lazy loading for non-critical components
- Minimal JavaScript bundle size

---

### 10.2 Real-Time Performance

**SRS-SRD-REQ-PERF-002:** [Implements: SRD-REQ-FRONT-005, SRS-REQ-INC-002] Real-time updates SHALL:

- Update UI within 1 second of Firestore change
- Handle connection loss gracefully
- Reconnect automatically
- Show connection status indicator

---

## 11. Accessibility Requirements

**[Implements: SRD-REQ-FRONT-002, SRD-REQ-FRONT-023, SRS-REQ-UI-018]**

### 11.1 WCAG Compliance

**REQ-A11Y-001:** [Implements: SRD-REQ-FRONT-002, SRD-REQ-FRONT-014, SRD-REQ-FRONT-018, SRD-REQ-FRONT-023, SRS-REQ-UI-018] The application SHALL meet WCAG 2.1 AA:

- **Keyboard Navigation:**
  - All interactive elements keyboard accessible
  - Focus indicators visible
  - Logical tab order

- **Screen Readers:**
  - Semantic HTML
  - ARIA labels where needed
  - Alt text for images

- **Color Contrast:**
  - Minimum 4.5:1 for text
  - Minimum 3:1 for UI components

- **Form Accessibility:**
  - Labels for all inputs
  - Error messages associated with fields
  - Required field indicators

---

## 12. Error Handling

### 12.1 Error States

The application SHALL handle:

- **Network Errors:**
  - Display user-friendly message
  - Retry button
  - Offline indicator

- **Authentication Errors:**
  - Redirect to login
  - Clear error message
  - Preserve intended destination

- **Validation Errors:**
  - Inline field errors
  - Clear error messages
  - Highlight invalid fields

- **404 Errors:**
  - Custom 404 page
  - Navigation back to dashboard

---

## 13. Testing Requirements

**[Implements: SRD-REQ-TEST-001, SRD-REQ-TEST-002, SRD-REQ-TEST-003, SRS-REQ-USE-001, SRS-REQ-USE-002, SRS-REQ-USE-003, SRS-REQ-USE-004, SRS-REQ-USE-005]**

### 13.1 Component Testing

**SRD-REQ-TEST-001:** [Implements: SRD-REQ-TEST-001, SRS-REQ-USE-005] Components SHALL be tested:

- Unit tests for utility functions
- Component rendering tests
- User interaction tests
- Accessibility tests

### 13.2 Integration Testing

**SRD-REQ-TEST-002:** [Implements: SRD-REQ-TEST-002, SRS-REQ-USE-005] Integration tests SHALL cover:

- User flows (login, create patient, create alert)
- API integration
- Real-time updates
- Form submissions

---

**End of Document**

