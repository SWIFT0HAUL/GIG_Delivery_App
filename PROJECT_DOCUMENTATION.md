# Haulee Platform - Comprehensive Project Documentation

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Platform Overview](#platform-overview)
3. [Technology Stack](#technology-stack)
4. [Architecture](#architecture)
5. [User Roles & Access Control](#user-roles--access-control)
6. [Core Features](#core-features)
7. [Database Schema](#database-schema)
8. [Authentication & Security](#authentication--security)
9. [Frontend Structure](#frontend-structure)
10. [Backend & API](#backend--api)
11. [Deployment & Infrastructure](#deployment--infrastructure)
12. [Development Guidelines](#development-guidelines)

---

## Executive Summary

**Haulee** is a professional gig delivery marketplace platform designed to connect shippers, brokers, carriers, drivers, and vendors in the logistics industry. The platform facilitates job postings, load matching, fleet management, real-time tracking, and comprehensive financial management.

### Key Capabilities

- Multi-role support (Shipper, Broker, Carrier, Driver, Vendor, Admin, Super Admin)
- Real-time job matching and assignment
- GPS tracking and live map views
- Financial management and invoicing
- Document management and KYC verification
- Task management and onboarding workflows
- Comprehensive RBAC (Role-Based Access Control) system
- Multi-language support (English, Spanish, Arabic, Chinese, French)

---

## Platform Overview

### Project Information

- **Platform Name**: Haulee
- **Project ID**: 083b4646-0999-4abf-8b63-fba56b8fc586
- **Project URL**: https://lovable.dev/projects/083b4646-0999-4abf-8b63-fba56b8fc586
- **Type**: Full-stack web application
- **Industry**: Logistics & Transportation

### Platform Purpose

Haulee is designed to streamline the logistics delivery ecosystem by providing:

- A central marketplace for shipping jobs
- Efficient job matching between shippers and carriers/drivers
- Real-time tracking and visibility
- Financial management and payment processing
- Document management and compliance tracking
- Multi-tenant support with role-based access

---

## Technology Stack

### Frontend

- **Framework**: React 18.3.1
- **Build Tool**: Vite
- **Language**: TypeScript
- **Styling**: Tailwind CSS with custom design system
- **UI Components**:
  - shadcn/ui (Radix UI primitives)
  - Lucide React (icons)
- **State Management**:
  - React Query (@tanstack/react-query) for server state
  - React Context API for global state
- **Routing**: React Router DOM v6.30.1
- **Forms**: React Hook Form with Zod validation
- **Maps**:
  - Leaflet & React Leaflet
  - Mapbox GL
  - Google Maps API
- **Internationalization**: i18next with browser language detection

### Backend

- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth
- **Real-time**: Supabase Realtime subscriptions
- **Storage**: Supabase Storage
- **Edge Functions**: Supabase Edge Functions (Deno)

### Additional Technologies

- **Mobile**: Capacitor (iOS & Android support)
- **PDF Handling**: pdfjs-dist, react-pdf
- **Charts**: Recharts
- **QR Codes**: react-qr-code
- **Date Handling**: date-fns
- **Excel Export**: xlsx
- **Notifications**: Sonner

### Development Tools

- **Package Manager**: npm/bun
- **Version Control**: Git
- **Code Quality**: TypeScript strict mode
- **Deployment**: Lovable Cloud

---

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│  (React SPA + Capacitor for Mobile)                         │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Shipper  │  │  Driver  │  │ Carrier  │  │  Admin   │   │      Broker
│  │Dashboard │  │Dashboard │  │Dashboard │  │Dashboard │   │     Dashboard
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   AuthContext│  │SettingsContext│  │EventsContext │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │           React Query (Server State Cache)            │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Supabase Layer                          │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Auth       │  │  Realtime    │  │   Storage    │     │
│  │  (JWT)       │  │ Subscriptions│  │  (Files)     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │         PostgreSQL Database (Row Level Security)    │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Application Flow

1. **Authentication Flow**
   - User visits platform → Auth page
   - Signs up/in via Supabase Auth
   - Profile created in `profiles` table
   - Role-based redirection to appropriate dashboard
   - Force password change if required

2. **Onboarding Flow**
   - New users complete role-specific onboarding
   - Document upload and KYC verification
   - Profile completion required before approval
   - Admin approval for carrier/driver/vendor roles
   - Access granted upon approval

3. **Job Lifecycle**

   ```
   Posted → Claimed (Carrier) → Assigned (Driver)/Planned(Driver)(If Job priority is Scheduled)→
   In Progress → Picked Up → In Transit → Completed/Delivered
   ```

4. **Real-time Updates**
   - Database changes trigger Supabase Realtime events
   - Context providers manage subscriptions
   - UI auto-updates on data changes

---

## User Roles & Access Control

### Role Hierarchy

1. **Super Admin**
   - Full system access
   - Manage all users, roles, and permissions
   - Configure platform settings
   - View all financial data
   - Approve/reject users
   - Access all modules

2. **Admin**
   - Manage specific modules based on permissions
   - Approve users
   - View reports and analytics
   - Manage jobs and tasks
   - Limited financial access

3. **Shipper**
   - Create and manage jobs
   - Track deliveries
   - View shipment history
   - Manage payment methods
   - View invoices

4. **Carrier**
   - View and claim jobs
   - Manage fleet (vehicles and drivers)
   - Assign jobs to drivers
   - Track driver locations
   - View earnings and payouts

5. **Driver (Independent)**
   - View available jobs
   - Accept job assignments
   - Update job status
   - Upload proof of delivery
   - Track earnings

6. **Broker**
   - Post jobs on behalf of shippers
   - Manage multiple clients
   - Coordinate deliveries
   - View commission reports

7. **Vendor/Merchant**
   - Request deliveries
   - Manage delivery preferences
   - Track orders
   - View financial data

### Permission System

The platform uses a granular permission system:

**Permission Categories**:

- Jobs Management
- User Management
- Financial Management
- Settings Management
- Reports & Analytics
- Document Management
- Fleet Management

**Permission Types**:

- `create` - Create new resources
- `read` - View resources
- `update` - Edit existing resources
- `delete` - Remove resources
- `approve` - Approve requests
- `export` - Export data

**Implementation**:

- `useGranularPermissions()` hook
- `PermissionGuard` component
- `ResourceGuard` component
- `ProtectedButton` component
- Database-level RLS policies

---

## Core Features

### 1. Job Management

**For Shippers**:

- Create jobs with pickup/delivery details
- Set pricing and vehicle requirements
- Schedule pickups
- Track job status
- View job history
- Edit/cancel jobs

**For Carriers**:

- Browse available jobs
- Claim jobs for fleet
- Assign jobs to drivers
- Monitor job progress
- View completed jobs

**For Drivers**:

- View available jobs (filtered by capacity)
- Accept assignments
- Update job status
- Upload pickup/delivery photos
- Collect recipient signatures
- Submit proof of delivery

**Job Details Include**:

- Title and description
- Pickup location with address and coordinates
- Delivery location with address and coordinates
- Pay amount
- Vehicle type requirements
- Special instructions
- Metadata (photos, signatures, notes)
- Timeline tracking

### 2. Live Tracking & Maps

- Real-time driver location updates
- GPS tracking during active deliveries
- Multiple map provider support (Leaflet, Mapbox, Google Maps)
- Live view dashboard for all active deliveries
- Route optimization
- ETA calculations
- Geofencing capabilities

### 3. Fleet Management (Carriers)

- Vehicle registration and management
- Driver onboarding and management
- Vehicle capacity tracking
- Document management (licenses, insurance)
- Assignment tracking
- Performance metrics

### 4. Financial Management

**For All Roles**:

- Transaction history
- Invoice generation
- Payment tracking
- Earnings reports
- Payout management

**Features**:

- Encrypted financial data storage
- Financial audit logs
- Dispute management
- Ledger system
- Reports and analytics
- Fund management

**Finance Modules**:

- Overview dashboard
- Transactions
- Invoices
- Payouts
- Funds management
- Ledger
- Reports
- Disputes
- Settings

### 5. Document Management

- Upload documents (PDFs, images)
- Document categorization
- Version control
- Approval workflow
- Document viewer (PDF support)
- Secure storage with RLS
- KYC document handling

**Document Types**:

- Driver licenses
- Vehicle registrations
- Insurance certificates
- W9 forms
- KYC documents
- Proof of delivery
- Invoices

### 6. Onboarding System

**Role-Specific Onboarding**:

- Driver: License, vehicle info, background check
- Carrier: Company details, DOT/MC numbers, insurance
- Shipper: Company info, payment setup
- Vendor: Business verification, tax info
- Broker: Licensing, bond information

**Onboarding Steps**:

1. Account creation
2. Role selection
3. Profile completion
4. Document upload
5. KYC verification
6. Admin approval
7. Account activation

### 7. Task & Checklist System

- Admin task templates
- Auto-generated tasks
- Task assignment by role
- Priority levels (low, medium, high, urgent)
- Task categories (onboarding, compliance, operations, financial)
- Task status tracking
- Due date management
- Task workspace with step-by-step completion
- Checklist management

### 8. Notifications

- Real-time platform events
- In-app notification center
- Email notifications
- SMS notifications (Twilio integration)
- Notification preferences
- Read/unread status
- Notification history
- Event-driven architecture

**Notification Events**:

- Job status changes
- Payment updates
- Document approvals
- Task assignments
- System announcements

### 9. Settings & Customization

**Platform Settings**:

- Branding customization
- Theme configuration (light/dark mode)
- Language preferences
- Map provider selection
- Notification preferences
- Display settings

**User Settings**:

- Profile management
- Password change
- Two-factor authentication
- Privacy settings
- Communication preferences

### 10. Reports & Analytics

- KPI dashboard
- Job statistics
- Revenue reports
- User activity
- Performance metrics
- Financial summaries
- Export capabilities (CSV, Excel, PDF)

### 11. Multi-Language Support

**Supported Languages**:

- English (en)
- Spanish (es)
- Arabic (ar)
- Chinese (zh)
- French (fr)

**Features**:

- Automatic language detection
- Manual language switching
- Persistent language preference
- RTL support for Arabic
- Translation management via i18next

### 12. Mobile Support

- Responsive design
- Capacitor integration for native apps
- Camera access for photo capture
- GPS location tracking
- Push notifications
- Offline support (planned)

---

## Database Schema

### Core Tables

#### Profiles & Authentication

- `profiles` - User profiles with role information
- `roles` - Role definitions
- `role_permissions` - Permission assignments
- `permissions` - Available permissions
- `menu_items` - RBAC menu structure
- `role_menu_access` - Menu access control

#### Job Management

- `jobs` - All job listings
- `job_assignments` - Driver job assignments
- `carrier_jobs` - Carrier job records
- `carrier_job_assignments` - Carrier assignments
- `job_categories` - Job categorization
- `job_claim_authorizations` - Claim permissions

#### User-Specific Profiles

- `driver_profiles` - Driver details
- `carrier_profiles` - Carrier company details
- `carrier_company_status` - Carrier status tracking
- `shipper_profiles` - Shipper details
- `broker_profiles` - Broker details
- `vendor_profiles` - Vendor details

#### Fleet Management

- `vehicles` - Vehicle registry
- `driver_vehicles` - Driver-vehicle relationships
- `driver_capacity` - Driver capacity tracking

#### Documents

- `documents` - File storage metadata
- `admin_documents` - Admin documentation
- `driver_onboarding` - Driver onboarding docs

#### Financial

- `transactions` - Payment transactions
- `invoices` - Invoice records
- `payouts` - Payout tracking
- `financial_data_access_logs` - Encrypted data access logs
- `encrypted_financial_data` - Secure financial storage

#### Tasks & Approvals

- `admin_tasks` - Task management
- `admin_task_templates` - Task templates
- `admin_task_logs` - Task history
- `admin_approvals` - Approval workflow
- `task_step_submissions` - Task completion tracking
- `checklist_templates` - Checklist definitions
- `checklists` - Active checklists
- `checklist_items` - Checklist item details

#### Settings & Configuration

- `platform_settings` - Platform configuration
- `platform_settings_activity_log` - Settings audit trail
- `global_customization` - UI customization
- `user_language_preferences` - Language settings
- `timesheets` - Time tracking
- `timesheet_entries` - Time entries

#### Security & Audit

- `audit_logs` - System audit trail
- `access_violation_logs` - Security violations
- `kyc_data` - KYC verification data

#### Integrations

- `integrations` - Third-party integrations
- `api_keys` - API key management

#### Notifications

- `notifications` - Notification records
- `notification_preferences` - User preferences

### Key Database Features

**Row Level Security (RLS)**:

- All tables have RLS enabled
- Policies enforce role-based access
- Users can only access their own data
- Admins have elevated access via policies
- Cross-table policies for related data

**Triggers & Functions**:

- Auto-update timestamps
- Cascade status updates
- Job synchronization between tables
- Profile creation on signup
- Notification generation

**Enums**:

- `job_status` - Job lifecycle states
- `approval_status` - Approval states
- `task_status` - Task states
- `task_priority` - Priority levels
- `task_category` - Task categories
- `task_role` - Task roles
- `doc_category` - Document categories

**Views**:

- `driver_eligible_jobs` - Jobs filtered by driver capacity

---

## Authentication & Security

### Authentication System

**Supabase Auth Integration**:

- Email/password authentication
- OAuth providers ready (Google, etc.)
- JWT token-based sessions
- Automatic token refresh
- Session persistence in localStorage
- Force password change mechanism

**Auth Flow**:

1. User enters credentials
2. Supabase validates and creates session
3. JWT token issued
4. Profile fetched from `profiles` table
5. Role and permissions loaded
6. User redirected to role-specific dashboard

**Protected Routes**:

- `ProtectedRoute` component wraps authenticated routes
- Automatic redirect to `/auth` if not logged in
- Role-based route protection (e.g., admin-only routes)

### Security Features

**Row Level Security (RLS)**:

- PostgreSQL RLS on all tables
- User isolation at database level
- Admin override policies
- Secure joins across tables

**Password Security**:

- Minimum 12 characters
- Must include special character (!@#$%^&\*)
- Force password change on first login
- Password reset via email

**Data Encryption**:

- Financial data encryption at rest
- Sensitive documents encrypted
- API keys stored encrypted
- Audit trail for data access

**Access Control**:

- Granular permissions system
- Resource-level access control
- Operation-level permissions (CRUD)
- Permission guards on UI components

**Audit & Compliance**:

- Comprehensive audit logs
- Access violation tracking
- Financial data access logs
- Settings change history
- Task completion tracking

**Session Management**:

- Real-time session validation
- Profile change subscriptions
- Auto-logout on profile deactivation
- Session awareness hooks

---

## Frontend Structure

### Directory Structure

```
src/
├── assets/              # Static assets (images, fonts)
├── components/          # React components
│   ├── access/         # Access control components
│   ├── admin/          # Admin-specific components
│   ├── broker/         # Broker components
│   ├── carrier/        # Carrier components
│   ├── dashboard/      # Dashboard widgets
│   ├── driver/         # Driver components
│   ├── finance/        # Finance modules
│   ├── map/            # Map components
│   ├── notifications/  # Notification components
│   ├── onboarding/     # Onboarding components
│   ├── rbac/           # RBAC guards and controls
│   ├── shipper/        # Shipper components
│   ├── task/           # Task management components
│   ├── ui/             # shadcn/ui components
│   └── vendor/         # Vendor components
├── contexts/           # React Context providers
│   ├── AuthContext.tsx
│   ├── PlatformEventsContext.tsx
│   ├── PlatformSettingsContext.tsx
│   ├── SafeModeContext.tsx
│   └── DispatchContext.tsx
├── hooks/              # Custom React hooks
│   ├── useUserRole.tsx
│   ├── useGranularPermissions.tsx
│   ├── usePermissions.tsx
│   ├── useDriverLocation.tsx
│   ├── useNotifications.tsx
│   └── ... (30+ custom hooks)
├── i18n/               # Internationalization
│   ├── config.ts
│   └── locales/        # Translation files
├── integrations/       # External integrations
│   └── supabase/       # Supabase client & types
├── lib/                # Utility libraries
│   └── config.ts       # App configuration
├── pages/              # Page components
│   ├── admin/          # Admin pages
│   ├── dashboard/      # Dashboard pages
│   ├── onboarding/     # Onboarding pages
│   └── ... (30+ pages)
├── App.tsx             # Root component
├── main.tsx            # Entry point
├── index.css           # Global styles & design system
└── vite-env.d.ts       # Vite type definitions
```

### Key Components

**Layout Components**:

- `Navigation` - Main navigation bar
- `AdminSidebar` - Admin dashboard sidebar
- `AppSidebar` - Unified dashboard sidebar
- `AdminHeader` - Admin header with user menu

**Guard Components**:

- `ProtectedRoute` - Authentication guard
- `PermissionGuard` - Permission-based rendering
- `ResourceGuard` - Resource-action guard
- `SystemAccessGuard` - System access validation
- `JobAccessGuard` - Job-specific access
- `TaskAccessGuard` - Task access control

**Feature Components**:

- `JobDetailsDialog` - Job information display
- `JobEditDialog` - Job editing interface
- `FileUpload` - Document upload
- `TaskWorkspace` - Task completion UI
- `NotificationBadge` - Unread notification count
- `PlatformEventsHandler` - Event listener

**Dashboard Components**:

- `DriverHome` - Driver dashboard
- `ShipperDashboardHome` - Shipper dashboard
- `CarrierDashboardHome` - Carrier dashboard
- `AdminDashboardOverview` - Admin overview
- `BrokerDashboardHome` - Broker dashboard
- `VendorDashboardHome` - Vendor dashboard

### Context Providers

**AuthContext**:

- User authentication state
- Sign up/in/out methods
- User redirection logic
- Session management

**PlatformEventsContext**:

- Event emission and subscription
- Pub/sub pattern implementation
- Cross-component communication

**PlatformSettingsContext**:

- Platform configuration
- Theme management
- Settings persistence
- Activity logging

**SafeModeContext**:

- Development safety toggle
- Prevents accidental data changes

**DispatchContext**:

- Job dispatch state
- Assignment management

### Custom Hooks

**Role & Permission Hooks**:

- `useUserRole()` - User role information
- `useGranularPermissions()` - Permission checks
- `useJobPermissions()` - Job-specific permissions
- `useSystemAccessValidation()` - System access

**Data Fetching Hooks**:

- `useDriverProfileData()` - Driver profile
- `useCarrierProfileData()` - Carrier profile
- `useShipperProfileData()` - Shipper profile
- `useBrokerProfileData()` - Broker profile
- `useVendorProfileData()` - Vendor profile

**Utility Hooks**:

- `useDriverLocation()` - GPS tracking
- `useNotifications()` - Notification management
- `useOnboardingProgress()` - Onboarding status
- `useKYCStatus()` - KYC verification
- `useLanguageSync()` - Language synchronization
- `useRealtimeSync()` - Real-time updates

**Map Hooks**:

- `useGeocoding()` - Address to coordinates
- `useDirections()` - Route calculation
- `useJobLocationGeocoding()` - Job location mapping

**Financial Hooks**:

- `useEncryptedFinancialData()` - Secure financial data
- `useSecureVendorData()` - Vendor financial data
- `useVendorFinancialData()` - Vendor finances

**Communication Hooks**:

- `useTwilioSMS()` - SMS sending

**Other Hooks**:

- `useIsMobile()` - Mobile detection
- `useAddressStorage()` - Address autocomplete
- `useTaskStepSubmission()` - Task step handling
- `useTimesheet()` - Time tracking
- `useDriverCapacityCheck()` - Capacity validation

### Routing Structure

```
/ (root)                    → Auth page
/auth                       → Authentication
/dashboard/*                → Unified RBAC dashboard
  ├── /dashboard/home
  ├── /dashboard/jobs
  ├── /dashboard/deliveries
  ├── /dashboard/fleet
  ├── /dashboard/finance
  ├── /dashboard/reports
  └── /dashboard/users

/driver-dashboard/*         → Driver-specific routes
/shipper-dashboard/*        → Shipper-specific routes
/carrier-dashboard/*        → Carrier-specific routes
/broker-dashboard           → Broker dashboard
/vendor-dashboard/*         → Vendor dashboard
/admin-dashboard/*          → Admin dashboard

/onboarding                 → Onboarding flow
/status-and-tasks           → Task center
/pending-approval           → Approval waiting
/w9-form                    → W9 form submission
/task-workspace             → Task workspace
/task-form                  → Task creation

/settings                   → User settings
/support                    → Support page
/notifications              → Notification center
/live-view                  → Live tracking map
/sms-test                   → SMS testing (dev)

* (catch-all)               → 404 Not Found
```

### Design System

**Color Palette**:

- Primary: Deep Blue (#1E40AF) - Logistics/Trust
- Accent: Orange (#F97316) - Action/Energy
- Semantic tokens for all colors
- HSL format for all colors
- Light/dark mode support

**Design Tokens** (index.css):

- `--primary` - Main brand color
- `--primary-foreground` - Text on primary
- `--primary-glow` - Lighter primary
- `--secondary` - Secondary surfaces
- `--accent` - Call-to-action color
- `--muted` - Muted backgrounds
- `--destructive` - Error states
- `--border` - Border colors
- `--card` - Card backgrounds

**Gradients**:

- `--gradient-primary` - Blue gradient
- `--gradient-accent` - Orange gradient
- `--gradient-hero` - Blue to orange

**Shadows**:

- `--shadow-glow` - Primary glow effect
- `--shadow-accent` - Accent shadow

**Animations**:

- `fade-in` - Fade and slide in
- `scale-in` - Scale up
- `heartbeat` - Pulse animation
- `accordion-down/up` - Accordion transitions

**Responsive Design**:

- Mobile-first approach
- Breakpoints: sm, md, lg, xl, 2xl
- Mobile navigation optimization
- Touch-friendly UI elements

---

## Backend & API

### Supabase Configuration

**Connection Details**:

- URL: `https://cqoydkxlonzobykwjcin.supabase.co`
- Anonymous Key: (public, safe for client use)
- Auth: Local storage persistence, auto-refresh

**Client Configuration**:

```typescript
import { createClient } from '@supabase/supabase-js';
import type { Database } from './types';

export const supabase = createClient<Database>(
  SUPABASE_URL,
  SUPABASE_PUBLISHABLE_KEY,
  {
    auth: {
      storage: localStorage,
      persistSession: true,
      autoRefreshToken: true,
    }
  }
);
```

### Database Functions (RPC)

**User & Permissions**:

- `get_user_permissions(user_id)` - Fetch user permissions
- `is_super_admin(user_id)` - Check super admin status
- `check_user_permission(user_id, permission_name)` - Permission check

**Job Management**:

- `get_driver_eligible_jobs(driver_id)` - Jobs within driver capacity

**Menu & RBAC**:

- `get_menu_items_for_role(role_id)` - Role-based menu
- `get_permissions_for_role(role_id)` - Role permissions

### Edge Functions

_Note: Edge functions may be used for_:

- Webhook handling
- Third-party API integrations
- Complex business logic
- Scheduled tasks
- Email/SMS notifications

### Real-time Subscriptions

**Profile Changes**:

```typescript
supabase
  .channel('profile-changes')
  .on('postgres_changes',
    { event: '*', schema: 'public', table: 'profiles' },
    handleProfileChange
  )
  .subscribe();
```

**Job Updates**:

- Subscribe to job status changes
- Real-time assignment notifications
- Location updates during delivery

**Notification System**:

- Platform events emitted via `PlatformEventsContext`
- Real-time notification delivery
- Unread count updates

### API Integration Points

**External Services**:

- Twilio (SMS notifications)
- Google Maps API (geocoding, directions)
- Mapbox (map tiles)
- Leaflet (open-source maps)

**Configuration** (src/lib/config.ts):

```typescript
export const config = {
  baseUrl: process.env.VITE_BASE_URL || 'http://localhost:8080',
  supabase: {
    url: process.env.VITE_SUPABASE_URL,
    anonKey: process.env.VITE_SUPABASE_ANON_KEY,
  },
  auth: {
    redirectTo: `${baseUrl}/auth/callback`,
    emailRedirectTo: `${baseUrl}/`,
  },
};
```

---

## Deployment & Infrastructure

### Hosting

- **Platform**: Lovable Cloud
- **Frontend**: Static hosting via Lovable
- **Backend**: Supabase Cloud
- **Domain**: Custom domain support available

### Environment Variables

- `VITE_BASE_URL` - Application base URL
- `VITE_SUPABASE_URL` - Supabase project URL
- `VITE_SUPABASE_ANON_KEY` - Supabase anonymous key

### Build Process

```bash
# Install dependencies
npm install

# Development server
npm run dev

# Production build
npm run build

# Preview production build
npm run preview
```

### Deployment Strategy

1. Code changes committed to Git
2. Lovable automatically detects changes
3. Build triggered automatically
4. Frontend deployed to CDN
5. Database migrations applied via Supabase
6. Zero-downtime deployment

### Database Migrations

- Located in `supabase/migrations/`
- Timestamped SQL files
- Applied sequentially
- Version controlled
- Rollback support

### Monitoring & Logging

- Supabase dashboard for database metrics
- Browser console for client errors
- Audit logs for security events
- Access violation logs
- Financial access tracking

---

## Development Guidelines

### Code Organization

**Component Structure**:

```typescript
// Component definition
export const MyComponent: React.FC<Props> = ({ prop1, prop2 }) => {
  // Hooks first
  const { user } = useAuth();
  const [state, setState] = useState();

  // Effects
  useEffect(() => { ... }, []);

  // Event handlers
  const handleClick = () => { ... };

  // Render
  return <div>...</div>;
};
```

**Custom Hook Pattern**:

```typescript
export const useMyHook = () => {
  const [data, setData] = useState();
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Fetch logic
  useEffect(() => { ... }, []);

  return { data, loading, error };
};
```

### Styling Guidelines

**Use Design System**:

```tsx
// ✅ Good - Using design tokens
<button className="bg-primary text-primary-foreground">
  Click me
</button>

// ❌ Bad - Hard-coded colors
<button className="bg-blue-600 text-white">
  Click me
</button>
```

**Semantic Tokens**:

- Use `hsl(var(--token))` for all colors
- Define custom tokens in `index.css`
- Add new tokens to `tailwind.config.ts`

**Responsive Design**:

```tsx
<div className="flex flex-col md:flex-row gap-4">
  {/* Mobile: column, Desktop: row */}
</div>
```

### Database Access Patterns

**Fetch Data**:

```typescript
const { data, error } = await supabase
  .from('jobs')
  .select('*')
  .eq('created_by', user.id)
  .order('created_at', { ascending: false });
```

**Insert Data**:

```typescript
const { data, error } = await supabase
  .from('jobs')
  .insert({
    title: 'New Job',
    created_by: user.id,
    status: 'posted'
  })
  .select()
  .single();
```

**Update Data**:

```typescript
const { error } = await supabase
  .from('jobs')
  .update({ status: 'completed' })
  .eq('id', jobId);
```

**RLS-Safe Queries**:

- Always filter by user ID when appropriate
- Trust RLS policies for security
- Use `.select()` to specify columns
- Use `.single()` for single row results

### Permission Checking

**Component Level**:

```tsx
<PermissionGuard permission="jobs.create">
  <CreateJobButton />
</PermissionGuard>
```

**Hook Level**:

```typescript
const { hasPermission } = useGranularPermissions();

if (hasPermission('jobs.create')) {
  // Show create button
}
```

### Error Handling

**API Errors**:

```typescript
try {
  const { data, error } = await supabase.from('jobs').select();
  if (error) throw error;
  // Handle data
} catch (error) {
  console.error('Error:', error);
  toast.error('Failed to load jobs');
}
```

**User Feedback**:

```typescript
import { toast } from 'sonner';

// Success
toast.success('Job created successfully');

// Error
toast.error('Failed to create job');

// Info
toast.info('Changes saved');
```

### Testing Approach

**Manual Testing**:

1. Test all user roles
2. Verify permissions work correctly
3. Check responsive design
4. Test real-time updates
5. Verify RLS policies

**Key Test Scenarios**:

- User signup and onboarding
- Job creation and assignment
- Document upload and approval
- Financial transactions
- Real-time tracking
- Multi-language support

### Best Practices

**Security**:

- Never expose sensitive data in client
- Trust RLS, not client-side checks
- Validate all inputs
- Use prepared statements (Supabase handles this)
- Encrypt sensitive data

**Performance**:

- Use React Query for caching
- Lazy load images
- Paginate large lists
- Debounce search inputs
- Optimize database queries

**Accessibility**:

- Semantic HTML
- ARIA labels where needed
- Keyboard navigation
- Focus management
- Screen reader support

**Code Quality**:

- TypeScript strict mode
- Consistent naming conventions
- Component reusability
- Avoid prop drilling (use Context)
- Keep components focused

---

## Key Files Reference

### Configuration Files

- `vite.config.ts` - Vite configuration
- `tailwind.config.ts` - Tailwind CSS config
- `tsconfig.json` - TypeScript config
- `components.json` - shadcn/ui config
- `src/lib/config.ts` - App configuration

### Entry Points

- `index.html` - HTML entry
- `src/main.tsx` - React entry
- `src/App.tsx` - Root component

### Core Context Files

- `src/contexts/AuthContext.tsx` - Authentication
- `src/contexts/PlatformEventsContext.tsx` - Event system
- `src/contexts/PlatformSettingsContext.tsx` - Settings
- `src/contexts/SafeModeContext.tsx` - Safe mode
- `src/contexts/DispatchContext.tsx` - Dispatch

### Core Hook Files

- `src/hooks/useUserRole.tsx` - User role
- `src/hooks/useGranularPermissions.tsx` - Permissions
- `src/hooks/useDriverLocation.tsx` - GPS tracking
- `src/hooks/useNotifications.tsx` - Notifications

### Database

- `src/integrations/supabase/client.ts` - Supabase client
- `src/integrations/supabase/types.ts` - TypeScript types
- `supabase/migrations/` - Database migrations

### Styling

- `src/index.css` - Global styles & design system
- `src/components/ui/` - shadcn/ui components

### Internationalization

- `src/i18n/config.ts` - i18next config
- `src/i18n/locales/` - Translation files

---

## Future Enhancements

### Planned Features

- Mobile app deployment (iOS/Android via Capacitor)
- Offline support with local storage
- Advanced analytics and reporting
- Automated routing optimization
- Integration with more payment providers
- Enhanced document OCR and extraction
- Video chat support for customer service
- Blockchain-based proof of delivery
- AI-powered job matching
- Predictive analytics for demand forecasting

### Technical Improvements

- Comprehensive test coverage
- CI/CD pipeline
- Performance monitoring
- Error tracking (Sentry)
- Load testing
- Database query optimization
- Code splitting and lazy loading
- PWA capabilities

---

## Support & Documentation

### Internal Documentation

- Code comments throughout
- TypeScript types provide self-documentation
- Component prop interfaces
- Hook return value types

### External Resources

- Lovable Documentation: https://docs.lovable.dev/
- Supabase Documentation: https://supabase.com/docs
- React Documentation: https://react.dev/
- Tailwind CSS: https://tailwindcss.com/

### Getting Help

- Check console logs for errors
- Review Supabase logs for database issues
- Use browser DevTools for debugging
- Check network tab for API issues

---

## Conclusion

Haulee is a comprehensive logistics platform built with modern web technologies. It leverages React, TypeScript, Supabase, and Tailwind CSS to provide a robust, scalable, and secure solution for connecting shippers, carriers, drivers, and other stakeholders in the delivery ecosystem.

The platform's architecture emphasizes security through Row Level Security, real-time capabilities via Supabase Realtime, and a flexible RBAC system that scales from individual drivers to large carrier companies. With support for multiple languages, mobile devices, and comprehensive financial management, Haulee is positioned as a complete solution for the gig delivery marketplace.

---

**Document Version**: 1.0  
**Last Updated**: November 11, 2025  
**Platform Version**: Current Development  
**Author**: Haulee Development Team
