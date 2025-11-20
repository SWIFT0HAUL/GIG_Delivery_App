# System Access Validation

Comprehensive access control and validation system for the platform.

## Overview

This system provides multi-layered security validation for:
- Role-based access control (RBAC)
- Permission-based access control
- Resource-level access control
- Task and job-specific access validation
- KYC requirement validation
- Admin panel access control

## Components

### Hooks

#### `useSystemAccessValidation`
Central hook for all access validation logic.

```tsx
import { useSystemAccessValidation } from '@/hooks/useSystemAccessValidation';

const {
  validateRouteAccess,
  validateTaskAccess,
  validateJobAccess,
  validateResourceAccess,
  validateAdminPanelAccess,
  validateKYCRequirement,
  isSuperAdmin,
  isAdmin,
  isApproved,
  hasAdminAccess,
  userRole,
} = useSystemAccessValidation();
```

### Guards

#### `SystemAccessGuard`
General-purpose access guard with multiple validation options.

```tsx
import { SystemAccessGuard } from '@/components/access';

<SystemAccessGuard
  resource="user"
  action="create"
  requireKYC={true}
  requireApproval={true}
  requireAdmin={false}
  requireSuperAdmin={false}
  showAlert={true}
  redirectOnFail="/dashboard"
>
  <ProtectedContent />
</SystemAccessGuard>
```

#### `TaskAccessGuard`
Validates task-specific access.

```tsx
import { TaskAccessGuard } from '@/components/access';

<TaskAccessGuard
  taskId="task-uuid"
  action="submit"
  showAlert={true}
>
  <TaskForm />
</TaskAccessGuard>
```

#### `JobAccessGuard`
Validates job-specific access.

```tsx
import { JobAccessGuard } from '@/components/access';

<JobAccessGuard
  jobId="job-uuid"
  action="edit"
  showAlert={true}
>
  <JobEditor />
</JobAccessGuard>
```

## Database Functions

### `user_has_permission(user_id, permission_name)`
Checks if a user has a specific permission.

### `user_can_access_task(user_id, task_id, action)`
Validates task access for actions: view, submit, approve, edit, delete.

### `user_can_access_job(user_id, job_id, action)`
Validates job access for actions: view, edit, delete, assign, claim.

### `log_access_violation(resource_type, resource_id, action, reason)`
Logs unauthorized access attempts for security monitoring.

## Access Levels

### Super Admin
- Full access to all features
- Bypasses all permission checks
- Can manage other admins
- Access to audit logs

### Admin
- Access to admin panels
- User management
- Job management
- Task approval
- Reporting

### Role-Based
- Driver: Job claiming, task submissions
- Carrier: Fleet management, job assignments
- Shipper: Job creation, tracking
- Broker: Load management, carrier coordination
- Vendor: Inventory, orders, deliveries

## Security Features

1. **Multi-layer validation**: Client-side guards + server-side RLS policies
2. **Audit logging**: All access violations are logged
3. **KYC enforcement**: Critical features require KYC approval
4. **Session validation**: Real-time session monitoring
5. **Permission caching**: Efficient permission checking

## Usage Examples

### Protecting a Route
```tsx
import { SystemAccessGuard } from '@/components/access';

export default function AdminPage() {
  return (
    <SystemAccessGuard requireAdmin requireApproval>
      <AdminContent />
    </SystemAccessGuard>
  );
}
```

### Protecting a Button
```tsx
import { ProtectedButton } from '@/components/rbac';

<ProtectedButton
  permission="job:delete"
  onClick={handleDelete}
>
  Delete Job
</ProtectedButton>
```

### Validating Task Submission
```tsx
const { validateTaskAccess } = useSystemAccessValidation();

const handleSubmit = async () => {
  const access = await validateTaskAccess(taskId, 'submit');
  if (!access.hasAccess) {
    toast.error(access.reason);
    return;
  }
  // Proceed with submission
};
```

## Best Practices

1. **Always validate on both client and server**: Use guards for UI and RLS policies for database
2. **Super admins bypass checks**: Design features with this in mind
3. **Log access violations**: Use for security monitoring and user support
4. **Provide clear feedback**: Show users why they don't have access
5. **Cache permission checks**: Avoid repeated database queries
6. **Validate before mutations**: Check access before allowing data changes

## Error Handling

All validation functions return an `AccessValidationResult`:

```typescript
interface AccessValidationResult {
  hasAccess: boolean;
  reason?: string;
  requiredPermission?: string;
  userRole?: string;
}
```

Handle access denials gracefully with informative messages.
