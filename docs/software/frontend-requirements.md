# Frontend Requirements Documentation

**Framework:** Next.js (App Router)
**UI Framework:** Tailwind CSS with PostCSS 

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
│   │   ├── LoadingSpinner.tsx
│   │   └── ErrorMessage.tsx
│   ├── layout/                    # Layout components
│   │   ├── TopNavBar.tsx         # Top navigation bar
│   │   ├── LeftSideMenu.tsx      # Left sidebar (desktop/tablet)
│   │   ├── MobileNavMenu.tsx     # Mobile hamburger menu
│   │   ├── UserDropdownMenu.tsx  # User profile dropdown
│   │   ├── LoadingOverlay.tsx    # Global loading indicator
│   │   └── NavLink.tsx           # Navigation link component
│   └── auth/                      # Authentication components
│       └── GoogleSignInButton.tsx # Google OAuth button
├── context/                        # React Context providers
│   ├── AuthContext.tsx            # Firebase Auth state
│   └── LoadingContext.tsx         # Global loading state
├── hooks/                          # Custom React hooks
│   ├── useAuth.ts                 # Authentication hook
│   ├── useLoadingState.ts         # Loading state hook
│   └── useMobileMenu.ts           # Mobile menu state hook
├── (authenticated)/               # Route group for protected pages
│   ├── layout.tsx                 # Authenticated layout wrapper
│   ├── dashboard/                 # Dashboard
│   │   └── page.tsx
│   └── settings/                  # Settings pages
│       └── page.tsx               # Account settings
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

[Implements: SRD-REQ-FRONT-011, SRD-REQ-FRONT-012, SRS-REQ-UI-004, SRS-REQ-UI-005, SRS-REQ-UI-017] The LeftSideMenu component SHALL implement:

```typescript
// app/components/layout/LeftSideMenu.tsx
'use client';

import { usePathname } from 'next/navigation';
import { 
  HomeIcon, 
  CogIcon 
} from '@heroicons/react/24/outline';
import { useAuth } from '@/hooks/useAuth';
import { NavLink } from './NavLink';

export function LeftSideMenu() {
  const pathname = usePathname();
  const { user } = useAuth();
  
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
- Simple navigation structure for authentication-focused application

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
import { XMarkIcon, HomeIcon, CogIcon } from '@heroicons/react/24/outline';
import { useMobileMenu } from '@/hooks/useMobileMenu';
import { usePathname } from 'next/navigation';
import { NavLink } from './NavLink';
import { useAuth } from '@/hooks/useAuth';

export function MobileNavMenu() {
  const { isOpen, closeMenu } = useMobileMenu();
  const pathname = usePathname();
  const { user } = useAuth();
  
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
  const publicRoutes = ['/', '/login'];
  const isPublicRoute = publicRoutes.some(route => path.startsWith(route));
  
  // Authenticated routes (everything under /(authenticated))
  const isAuthenticatedRoute = path.startsWith('/dashboard') || 
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

### 4.2 Dashboard (`/dashboard`)

[Implements: SRD-REQ-FRONT-005, SRS-REQ-UI-003, SRS-REQ-UI-006, SRS-REQ-UI-007] The dashboard page SHALL be wrapped in the authenticated layout and display:

- **Welcome Section:**
  - Welcome message with user's name
  - User profile information (from Google account)
  - Account status indicator

- **Profile Summary:**
  - Display name
  - Email address
  - Profile photo
  - Account created date
  - Last login timestamp

- **Quick Actions:**
  - "Edit Profile" button (navigates to settings)
  - "Sign Out" button

**Layout:**
- Responsive card layout
- Mobile: Single column
- Desktop: Centered card with max-width

**Example Implementation:**
```typescript
export default function DashboardPage() {
  const { user } = useAuth();
  
  return (
    <div className="max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold text-gray-900 mb-8">
        Welcome, {user?.displayName}!
      </h1>
      
      <div className="bg-white rounded-lg shadow p-6">
        <div className="flex items-center space-x-4 mb-6">
          <img
            src={user?.photoURL}
            alt="Profile"
            className="h-20 w-20 rounded-full"
          />
          <div>
            <h2 className="text-xl font-semibold">{user?.displayName}</h2>
            <p className="text-gray-600">{user?.email}</p>
          </div>
        </div>
        
        <div className="border-t pt-4">
          <p className="text-sm text-gray-500">
            Account Status: <span className="text-green-600">Active</span>
          </p>
        </div>
      </div>
    </div>
  );
}
```

---

### 4.3 Settings Page (`/settings`)

The settings page SHALL provide:

- **Profile Information:**
  - Display current profile (read-only email from Google)
  - Editable first name
  - Editable last name
  - Profile photo (from Google, not editable)

- **Form Fields:**
  - First name input
  - Last name input

- **Form Validation:**
  - Client-side validation
  - Max length constraints
  - Error messages

- **Actions:**
  - "Save Changes" button
  - Success/error notifications

**Example Implementation:**
```typescript
export default function SettingsPage() {
  const { user } = useAuth();
  const [firstName, setFirstName] = useState(user?.firstName || '');
  const [lastName, setLastName] = useState(user?.lastName || '');
  const [saving, setSaving] = useState(false);
  
  const handleSave = async () => {
    setSaving(true);
    try {
      await fetch('/api/v1/me', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ firstName, lastName })
      });
      // Show success message
    } catch (error) {
      // Show error message
    } finally {
      setSaving(false);
    }
  };
  
  return (
    <div className="max-w-2xl mx-auto">
      <h1 className="text-3xl font-bold mb-8">Account Settings</h1>
      
      <div className="bg-white rounded-lg shadow p-6">
        <div className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700">
              Email (from Google account)
            </label>
            <input
              type="email"
              value={user?.email}
              disabled
              className="mt-1 block w-full rounded-md border-gray-300 bg-gray-50"
            />
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700">
              First Name
            </label>
            <input
              type="text"
              value={firstName}
              onChange={(e) => setFirstName(e.target.value)}
              className="mt-1 block w-full rounded-md border-gray-300"
            />
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700">
              Last Name
            </label>
            <input
              type="text"
              value={lastName}
              onChange={(e) => setLastName(e.target.value)}
              className="mt-1 block w-full rounded-md border-gray-300"
            />
          </div>
          
          <button
            onClick={handleSave}
            disabled={saving}
            className="btn-primary"
          >
            {saving ? 'Saving...' : 'Save Changes'}
          </button>
        </div>
      </div>
    </div>
  );
}
```

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

**LoadingSpinner Component:**
```typescript
interface LoadingSpinnerProps {
  size?: 'sm' | 'md' | 'lg';
  color?: string;
}
```

**ErrorMessage Component:**
```typescript
interface ErrorMessageProps {
  message: string;
  onRetry?: () => void;
}
```

---

## 6. State Management

**[Implements: SRD-REQ-FRONT-004, SRS-REQ-UI-003, SRS-REQ-UI-017]**

### 6.1 Context Providers

[Implements: SRD-REQ-FRONT-004, SRS-REQ-UI-003] The following context providers SHALL be implemented:

**AuthContext:**
```typescript
interface AuthContextValue {
  user: User | null;
  loading: boolean;
  signInWithGoogle: () => Promise<void>;
  signOut: () => Promise<void>;
  // User profile automatically created from Google account via API
}
```

**LoadingContext:**
```typescript
interface LoadingContextValue {
  isLoading: boolean;
  loadingMessage: string | null;
  startLoading: (message?: string) => void;
  stopLoading: () => void;
}
```

**Notes:**
- AuthContext manages Firebase Authentication state
- LoadingContext provides global loading state for API calls and long-running operations
- User profile data fetched from `/api/v1/me` after authentication
- All context providers wrapped in root layout for global access

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

**[Implements: SRD-REQ-TEST-001, SRD-REQ-TEST-002]**

### 13.1 Component Testing

Components SHALL be tested for:

- Component rendering
- User interaction (button clicks, form inputs)
- Loading states
- Error states
- Accessibility compliance

### 13.2 Integration Testing

Integration tests SHALL cover:

- Authentication flow (Google Sign-In, logout)
- API integration (`GET /api/v1/me`, `PUT /api/v1/me`)
- Navigation between pages
- Form submissions (settings update)
- Route protection (middleware redirect)

### 13.3 E2E Testing Scenarios

Key user journeys to test:

1. **First-time user sign-in:**
   - Click "Sign in with Google"
   - Complete Google OAuth flow
   - Verify redirect to dashboard
   - Verify user profile displayed

2. **Update profile:**
   - Navigate to settings
   - Update first/last name
   - Save changes
   - Verify success message
   - Verify updated data displayed

3. **Sign out:**
   - Click user dropdown
   - Click sign out
   - Verify redirect to landing page
   - Verify cannot access protected routes

---

**End of Document**

