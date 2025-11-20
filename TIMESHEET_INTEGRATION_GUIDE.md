# Timesheet System Integration Guide

## 🎯 Quick Start

### For Developers

1. **Import the API client:**
```typescript
import * as TimesheetAPI from '@/lib/timesheetApi';
```

2. **Use in your components:**
```typescript
// Clock in
const handleClockIn = async () => {
  try {
    const result = await TimesheetAPI.clockIn(userId, optionalTaskId);
    console.log('Clocked in:', result.timesheet);
  } catch (error) {
    console.error('Clock in failed:', error.message);
  }
};

// Clock out
const handleClockOut = async (timesheetId: string) => {
  try {
    const result = await TimesheetAPI.clockOut(timesheetId);
    console.log('Clocked out:', result.timesheet);
  } catch (error) {
    console.error('Clock out failed:', error.message);
  }
};

// Generate report
const getWeeklyReport = async () => {
  const startDate = '2025-10-07';
  const endDate = '2025-10-13';
  
  try {
    const result = await TimesheetAPI.generateReport({ startDate, endDate });
    console.log('Report:', result.report);
  } catch (error) {
    console.error('Report generation failed:', error.message);
  }
};
```

---

## 🔗 Integration Points

### 1. Task Management Integration

Timesheet entries can be linked to tasks from the Task Management system.

**Database Link:**
```sql
-- timesheet_activity_logs table has these columns:
task_id uuid REFERENCES custom_tasks(id)
task_assignment_id uuid REFERENCES task_assignments(id)
```

**Usage in Code:**
```typescript
// When clocking in with a task
await TimesheetAPI.clockIn(userId, taskId);

// This automatically creates an activity log:
// - timesheet_id: new timesheet entry
// - activity_type: 'TASK_START'
// - task_id: linked task
```

**Benefits:**
- Track time spent on specific tasks
- Generate task-level time reports
- Calculate task completion efficiency
- Monitor resource allocation

---

### 2. User Role System Integration

Timesheet access is controlled by the RBAC system.

**Role Permissions:**

```typescript
// src/lib/rbacConfig.ts additions needed:
export const PERMISSIONS = {
  TIMESHEET: {
    VIEW_OWN: 'timesheet.view.own',
    VIEW_ALL: 'timesheet.view.all',
    CREATE: 'timesheet.create',
    APPROVE: 'timesheet.approve',
    EXPORT: 'timesheet.export',
  }
};
```

**Usage:**
```typescript
import { useGranularPermissions } from '@/hooks/useGranularPermissions';

const { hasPermission } = useGranularPermissions();

// Check if user can approve timesheets
if (hasPermission('timesheet.approve')) {
  // Show approval buttons
}
```

**Database Functions:**
```sql
-- has_role() function checks if user can access timesheets
-- Used in RLS policies:
CREATE POLICY "Admins can view own timesheets"
  ON timesheets FOR SELECT
  USING (admin_id = auth.uid() OR has_role(auth.uid(), 'super_admin'));
```

---

### 3. Notification System Integration

Timesheets trigger notifications automatically via database triggers.

**Notification Events:**

| Event | Recipient | Trigger |
|-------|-----------|---------|
| Manual Entry Created | Super Admins | Insert with manual_entry=true |
| Entry Approved | Submitting Admin | Status → approved |
| Entry Rejected | Submitting Admin | Status → rejected |
| Overlapping Entry | Super Admins | Overlap detected |
| Missing Entries | Super Admins | Weekly check |
| Inactivity Warning | Active Admin | 45 min inactive |

**Database Triggers:**
```sql
-- notify_manual_entry_decision
-- notify_overlapping_timesheets
-- log_timesheet_status_change
```

**Frontend Handling:**
```typescript
// Listen for notifications in real-time
useEffect(() => {
  const channel = supabase
    .channel('timesheet-notifications')
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'notifications',
        filter: `user_id=eq.${userId}`
      },
      (payload) => {
        // Show toast notification
        if (payload.new.title.includes('Timesheet')) {
          toast(payload.new.title, {
            description: payload.new.message
          });
        }
      }
    )
    .subscribe();

  return () => {
    supabase.removeChannel(channel);
  };
}, [userId]);
```

---

### 4. Payroll Module Integration (Future)

The timesheet system is prepared for payroll integration with dedicated fields.

**Payroll Fields:**
```typescript
interface PayrollIntegration {
  payroll_exported: boolean;
  payroll_export_date: timestamp;
  payroll_period_start: date;
  payroll_period_end: date;
}
```

**Integration Hooks:**
```typescript
// Mark timesheets as exported for payroll
export const markExportedForPayroll = async (
  timesheetIds: string[],
  periodStart: string,
  periodEnd: string
) => {
  const { error } = await supabase
    .from('timesheets')
    .update({
      payroll_exported: true,
      payroll_export_date: new Date().toISOString(),
      payroll_period_start: periodStart,
      payroll_period_end: periodEnd,
    })
    .in('id', timesheetIds);
  
  if (error) throw error;
  return { success: true };
};

// Get timesheets ready for payroll export
export const getPayrollReadyTimesheets = async (
  periodStart: string,
  periodEnd: string
) => {
  const { data, error } = await supabase
    .from('timesheets')
    .select('*')
    .gte('date', periodStart)
    .lte('date', periodEnd)
    .eq('status', 'approved')
    .eq('payroll_exported', false);
  
  if (error) throw error;
  return data;
};
```

**Future Payroll Features:**
- Export to common payroll formats (CSV, ADP, QuickBooks)
- Automatic pay period calculations
- Overtime rate calculations
- PTO/vacation time tracking
- Benefits calculations

---

## 🔄 Workflow Diagrams

### Standard Clock In/Out Flow
```
User clicks "Clock In"
  → Validates no active session
  → Creates timesheet entry (status: active)
  → [Optional] Links to task
  → Creates activity log
  → Starts timer in UI

User clicks "Clock Out"
  → Updates timesheet (clock_out, status: completed)
  → Calculates total_duration
  → Ends timer in UI
  → Refreshes timesheet list
```

### Manual Entry Approval Flow
```
Admin creates manual entry
  → Validates timestamps (no future, no overlap)
  → Creates timesheet (status: pending_approval)
  → Sends notification to Super Admins

Super Admin reviews
  → Views entry details
  → Clicks Approve or Reject
  → [If reject] Provides reason
  → Updates status
  → Sends notification to submitting admin

Admin receives notification
  → Views updated status
  → [If rejected] Reviews reason and resubmits
```

### Weekly Report Automation Flow
```
Cron job triggers (Monday 9 AM UTC)
  → Edge function executes
  → Calculates weekly totals for each admin
  → Generates summary records
  → Sends reminder notifications
  → Checks for missing entries
  → Notifies Super Admins of issues
```

---

## 📦 API Response Examples

### Success Response (Clock In)
```json
{
  "success": true,
  "timesheet": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "admin_id": "123e4567-e89b-12d3-a456-426614174001",
    "clock_in": "2025-10-11T14:30:00.000Z",
    "clock_out": null,
    "total_duration": null,
    "status": "active",
    "manual_entry": false,
    "activity_count": 0,
    "created_at": "2025-10-11T14:30:00.000Z"
  }
}
```

### Error Response (Overlapping Entry)
```json
{
  "error": "This timesheet entry overlaps with an existing session"
}
```

### Report Response
```json
{
  "success": true,
  "report": {
    "timesheets": [
      {
        "id": "...",
        "clock_in": "2025-10-07T09:00:00Z",
        "clock_out": "2025-10-07T17:00:00Z",
        "total_duration": 8.0,
        "status": "approved"
      }
    ],
    "summary": {
      "total_hours": 40.0,
      "overtime_hours": 2.5,
      "pending_count": 1,
      "total_entries": 5
    }
  }
}
```

---

## 🧩 Component Integration

### Using TimesheetManagement Component
```typescript
import { TimesheetManagement } from '@/components/admin/TimesheetManagement';

// In your admin dashboard
<Route path="/admin-dashboard">
  <TimesheetManagement />
</Route>
```

### Custom Integration
```typescript
// Use individual components
import { ClockInOutCard } from '@/components/admin/timesheet/ClockInOutCard';
import { TimesheetTable } from '@/components/admin/timesheet/TimesheetTable';
import { SummaryCard } from '@/components/admin/timesheet/SummaryCard';

// Build custom layout
<div className="timesheet-container">
  <ClockInOutCard
    currentSession={currentSession}
    elapsedTime={elapsedTime}
    availableTasks={tasks}
    onClockIn={handleClockIn}
    onClockOut={handleClockOut}
    onManualEntry={() => setManualEntryOpen(true)}
    formatDuration={formatDuration}
  />
  
  <div className="grid grid-cols-5 gap-4">
    <SummaryCard title="Total Hours" value="40h" />
    {/* ... more summary cards */}
  </div>
  
  <TimesheetTable
    timesheets={timesheets}
    isSuperAdmin={isSuperAdmin}
    viewAllTimesheets={viewAll}
    onViewDetails={handleViewDetails}
    onApprove={handleApprove}
    onReject={handleReject}
  />
</div>
```

---

## 🔍 Monitoring & Debugging

### Check Cron Jobs Status
```sql
-- View all scheduled jobs
SELECT * FROM cron.job;

-- View job run history
SELECT * FROM cron.job_run_details ORDER BY start_time DESC LIMIT 20;
```

### View Edge Function Logs
1. Go to Supabase Dashboard
2. Navigate to Edge Functions
3. Select function (timesheet-api, check-late-clockouts, or timesheet-weekly-tasks)
4. Click "Logs" tab

### Database Queries for Debugging
```sql
-- Find active sessions
SELECT * FROM timesheets WHERE clock_out IS NULL;

-- Find pending approvals
SELECT * FROM timesheets WHERE status = 'pending_approval';

-- Check recent activity logs
SELECT * FROM timesheet_activity_logs ORDER BY created_at DESC LIMIT 50;

-- View notifications sent
SELECT * FROM notifications WHERE message LIKE '%timesheet%' ORDER BY created_at DESC;
```

---

## 🛠️ Maintenance Tasks

### Weekly
- Review pending manual entries
- Check for missing timesheet submissions
- Verify automated summaries generated correctly

### Monthly
- Archive old timesheet data if needed
- Review and optimize database queries
- Check cron job execution logs

### As Needed
- Update cron job schedules
- Modify notification templates
- Adjust validation rules
- Update approval workflows

---

## 📝 Changelog

### Version 1.0 (2025-10-11)
- ✅ Backend API endpoints (`/timesheet-api/*`)
- ✅ Database schema with security triggers
- ✅ Frontend dashboard components
- ✅ Weekly report automation
- ✅ Notification and approval flow
- ✅ Task management integration
- ✅ Role-based access control
- ✅ Export functionality (CSV, Excel, PDF)

---

## 🚦 Next Steps

1. **Test the API endpoints** using the provided examples
2. **Verify cron jobs** are running as scheduled
3. **Monitor notifications** to ensure delivery
4. **Review security policies** and adjust as needed
5. **Train users** on the new timesheet workflow
6. **Set up payroll integration** when ready (optional)

---

For questions or support, refer to `TIMESHEET_DELIVERABLES.md` for detailed specifications.