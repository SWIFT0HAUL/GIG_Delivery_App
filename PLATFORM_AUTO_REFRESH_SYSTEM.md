# Platform Auto-Refresh System Documentation

## Overview
A comprehensive, platform-wide auto-refresh system that detects and responds to all meaningful events across authentication, database changes, and cross-tab interactions.

## Architecture

### 1. **Platform Event Bus** (`src/contexts/PlatformEventsContext.tsx`)
- Centralized pub/sub system for all platform events
- 17 event types covering auth, roles, approvals, and notifications
- Components can subscribe to specific events and react accordingly

**Event Types:**
- `AUTH_SIGN_IN` - User signs in
- `AUTH_SIGN_OUT` - User signs out
- `AUTH_TOKEN_REFRESHED` - Session token refreshed
- `AUTH_PASSWORD_CHANGED` - Password updated
- `AUTH_USER_DELETED` - User account deleted
- `AUTH_USER_UPDATED` - User metadata updated
- `AUTH_SESSION_EXPIRED` - Session expired
- `PROFILE_UPDATED` - Profile data changed
- `ROLE_CHANGED` - User role changed
- `PERMISSION_CHANGED` - Permissions updated
- `USER_APPROVED` - User approved by admin
- `USER_ACTIVATED` - Account activated
- `USER_DEACTIVATED` - Account deactivated
- `USER_BANNED` - User banned
- `NOTIFICATION_RECEIVED` - New notification
- `CROSS_TAB_LOGOUT` - Logout from another tab
- `CROSS_TAB_ROLE_CHANGE` - Role change from another tab
- `FORCE_RELOAD` - Force page reload

### 2. **Cross-Tab Sync** (`src/lib/crossTabSync.ts`)
- Uses BroadcastChannel API (with localStorage fallback)
- Syncs critical events across browser tabs
- Events: `LOGOUT`, `ROLE_CHANGE`, `SESSION_UPDATE`, `USER_DELETED`, `FORCE_RELOAD`

### 3. **Realtime Database Listeners** (`src/hooks/useRealtimeSync.tsx`)
- Listens to Supabase realtime changes on:
  - `profiles` - User profile updates
  - `user_roles` - Role assignments
  - `notifications` - New notifications
  - `admin_approvals` - Approval status changes
- Automatically emits platform events when changes affect current user

### 4. **Platform Events Handler** (`src/components/PlatformEventsHandler.tsx`)
- Global event response coordinator
- Handles:
  - User deletion → Force sign out
  - Deactivation → Redirect to status page
  - Activation → Reload dashboard
  - Role changes → Refresh session + reload
  - Approval → Reload
  - Session expiration → Sign out
- Shows toast notifications for all events
- Coordinates cross-tab sync

### 5. **Enhanced AuthContext** (Modified: `src/contexts/AuthContext.tsx`)
- Added comprehensive `onAuthStateChange` logging
- Added cross-tab logout sync
- All existing auth logic preserved

## How It Works

### User Deletion Flow:
1. Admin deletes user in database
2. Realtime listener detects profile deletion
3. Platform Event Bus emits `AUTH_USER_DELETED`
4. PlatformEventsHandler receives event
5. Sends `USER_DELETED` to all tabs via crossTabSync
6. All tabs sign out user immediately
7. Toast notification shown

### Role Change Flow:
1. Admin updates user role
2. Realtime listener detects change in `profiles` or `user_roles`
3. Platform Event Bus emits `ROLE_CHANGED`
4. PlatformEventsHandler receives event
5. Sends `ROLE_CHANGE` to all tabs
6. Refreshes Supabase session (gets new metadata)
7. Reloads page after 1.5s
8. Toast notification shown

### Cross-Tab Logout Flow:
1. User signs out in Tab A
2. AuthContext calls `signOut()`
3. crossTabSync sends `LOGOUT` message
4. Tab B receives message via BroadcastChannel
5. Tab B immediately calls `signOut()`
6. Both tabs redirect to login

## Usage in Components

### Subscribe to Events:
```tsx
import { usePlatformEvents } from '@/contexts/PlatformEventsContext';

function MyComponent() {
  const { subscribe } = usePlatformEvents();

  useEffect(() => {
    const unsubscribe = subscribe('ROLE_CHANGED', (event) => {
      console.log('Role changed:', event.payload);
      // Handle role change
    });

    return unsubscribe;
  }, [subscribe]);
}
```

### Emit Custom Events:
```tsx
import { usePlatformEvents } from '@/contexts/PlatformEventsContext';

function MyComponent() {
  const { emit } = usePlatformEvents();

  const handleAction = () => {
    emit('CUSTOM_EVENT', { data: 'something' });
  };
}
```

## What Was NOT Changed
✅ No UI components modified
✅ No forms modified
✅ No existing business logic altered
✅ All existing auth flows preserved
✅ All existing routing logic intact

## What Was ADDED
✅ 4 new files (event system, cross-tab sync, realtime hook, event handler)
✅ 2 new context providers wrapped in App.tsx
✅ 1 global event handler component
✅ Cross-tab sync on logout
✅ Comprehensive logging for debugging

## Browser Compatibility
- **Modern browsers**: BroadcastChannel API
- **Older browsers**: localStorage fallback
- **All browsers**: Full functionality maintained

## Security Notes
- All RLS policies remain enforced
- No new database permissions required
- Realtime only listens to user's own data
- Cross-tab sync is client-side only (no security implications)

## Testing

### Test User Deletion:
1. Open app in 2 tabs as same user
2. Have admin delete user in Supabase dashboard
3. Both tabs should sign out immediately

### Test Role Change:
1. Open app as user
2. Have admin change user's role in database
3. Page should reload with new role after ~1.5s

### Test Cross-Tab Logout:
1. Open app in 2 tabs as same user
2. Sign out in Tab A
3. Tab B should sign out immediately

## Performance
- Minimal overhead (event bus is in-memory)
- Realtime listeners only active for logged-in users
- Cross-tab sync uses efficient BroadcastChannel
- No polling or intervals
- Events fire only when actual changes occur

## Future Enhancements
- Add more granular permission events
- Add task/approval realtime updates
- Add admin-to-user push notifications
- Add connection status indicator
- Add offline/online detection
