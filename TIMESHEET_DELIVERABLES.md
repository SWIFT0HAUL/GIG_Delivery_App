# Timesheet System - Complete Deliverables

## 📋 Overview
A comprehensive timesheet management system with clock in/out functionality, manual entry approval workflow, automated reporting, and real-time notifications.

---

## 🗄️ Database Tables

### 1. `timesheets`
Primary table for tracking admin work hours.

**Columns:**
- `id` (uuid, primary key)
- `admin_id` (uuid, references profiles)
- `date` (date, auto-populated from clock_in)
- `clock_in` (timestamp with time zone)
- `clock_out` (timestamp with time zone, nullable)
- `total_duration` (numeric, auto-calculated)
- `status` (text: active, completed, pending_approval, approved, rejected, auto_timeout)
- `manual_entry` (boolean)
- `manual_entry_reason` (text, nullable)
- `notes` (text, nullable)
- `approved_by` (uuid, nullable)
- `approved_at` (timestamp, nullable)
- `rejection_reason` (text, nullable)
- `last_activity` (timestamp)
- `activity_count` (integer)
- `payroll_exported` (boolean)
- `payroll_export_date` (timestamp, nullable)
- `payroll_period_start` (date, nullable)
- `payroll_period_end` (date, nullable)

**Security Features:**
- Row Level Security (RLS) enabled
- Admins can only view/edit their own timesheets
- Super Admins can view/manage all timesheets
- Validation triggers prevent overlapping entries and future timestamps
- Manual entries require Super Admin approval

### 2. `timesheet_activity_logs`
Audit trail for all timesheet activities.

**Columns:**
- `id` (uuid, primary key)
- `timesheet_id` (uuid, references timesheets)
- `activity_type` (text)
- `activity_description` (text)
- `task_id` (uuid, references custom_tasks, nullable)
- `task_assignment_id` (uuid, nullable)
- `metadata` (jsonb)
- `created_at` (timestamp)

**Features:**
- Immutable audit trail (no updates/deletes allowed)
- Links to task management system
- Stores detailed activity metadata

### 3. `timesheet_weekly_summaries`
Automated weekly summary reports.

**Columns:**
- `id` (uuid, primary key)
- `admin_id` (uuid)
- `week_start` (date)
- `week_end` (date)
- `total_hours` (numeric)
- `overtime_hours` (numeric)
- `total_activities` (integer)
- `status` (text: draft, generated, submitted)
- `generated_at` (timestamp)

---

## 🔌 Backend API Endpoints

### Base URL
`https://cqoydkxlonzobykwjcin.supabase.co/functions/v1/timesheet-api/`

All endpoints require Bearer token authentication in the `Authorization` header.

### 1. **POST** `/clock-in`
Start a new timesheet session.

**Request Body:**
```json
{
  "admin_id": "uuid",
  "task_id": "uuid (optional)"
}
```

**Response:**
```json
{
  "success": true,
  "timesheet": {
    "id": "uuid",
    "clock_in": "2025-10-11T14:30:00Z",
    "status": "active"
  }
}
```

**Validations:**
- Checks for existing active session
- Prevents multiple concurrent sessions
- Logs task activity if task_id provided

---

### 2. **POST** `/clock-out`
End an active timesheet session.

**Request Body:**
```json
{
  "timesheet_id": "uuid"
}
```

**Response:**
```json
{
  "success": true,
  "timesheet": {
    "id": "uuid",
    "clock_out": "2025-10-11T18:30:00Z",
    "total_duration": 4.0,
    "status": "completed"
  }
}
```

---

### 3. **POST** `/manual-entry`
Create a manual timesheet entry (requires approval).

**Request Body:**
```json
{
  "admin_id": "uuid",
  "clock_in": "2025-10-11T09:00:00Z",
  "clock_out": "2025-10-11T17:00:00Z",
  "reason": "Forgot to clock in",
  "notes": "Additional context",
  "task_id": "uuid (optional)"
}
```

**Response:**
```json
{
  "success": true,
  "timesheet": {
    "id": "uuid",
    "status": "pending_approval",
    "manual_entry": true
  }
}
```

**Validations:**
- No future timestamps allowed
- Clock out must be after clock in
- Checks for overlapping sessions
- Creates activity log if task provided

---

### 4. **POST** `/approve`
Approve or reject a timesheet entry (Super Admin only).

**Request Body:**
```json
{
  "timesheet_id": "uuid",
  "action": "approve" | "reject",
  "rejection_reason": "string (required if action is reject)"
}
```

**Response:**
```json
{
  "success": true,
  "timesheet": {
    "id": "uuid",
    "status": "approved",
    "approved_by": "uuid",
    "approved_at": "2025-10-11T19:00:00Z"
  }
}
```

**Authorization:**
- Only Super Admins can approve/reject
- Returns 403 if user lacks permissions

---

### 5. **GET** `/report`
Generate timesheet reports with filters.

**Query Parameters:**
- `start_date` (ISO date, optional)
- `end_date` (ISO date, optional)
- `admin_id` (uuid, optional)

**Example:**
```
GET /report?start_date=2025-10-01&end_date=2025-10-31&admin_id=uuid
```

**Response:**
```json
{
  "success": true,
  "report": {
    "timesheets": [...],
    "summary": {
      "total_hours": 160.5,
      "overtime_hours": 12.5,
      "pending_count": 3,
      "total_entries": 22
    }
  }
}
```

---

## 🎨 Frontend Components

### Component Structure
```
src/components/admin/
├── TimesheetManagement.tsx (Main container)
└── timesheet/
    ├── ClockInOutCard.tsx (Clock in/out UI)
    ├── TimesheetTable.tsx (Entry list with filters)
    ├── SummaryCard.tsx (Statistics display)
    ├── ManualEntryForm.tsx (Manual entry dialog)
    ├── AdminActivityChart.tsx (Visual analytics)
    └── types.ts (TypeScript interfaces)
```

### Key Features
- **Real-time Session Tracking:** Live timer for active sessions
- **Task Integration:** Optional task selection when clocking in
- **Inactivity Detection:** Warns users after 45 min, auto-clocks out after 60 min
- **Filtering:** By date range, status, and admin (Super Admin view)
- **Export:** CSV, Excel, and PDF export capabilities
- **Approval Workflow:** UI for Super Admins to approve/reject manual entries

---

## ⚙️ Automated Processes

### 1. Weekly Report Generation
**Schedule:** Every Monday at 9:00 AM UTC  
**Function:** `timesheet-weekly-tasks`

**What it does:**
- Generates weekly summaries for all admins
- Calculates total hours, overtime, and activity counts
- Sends weekly reminder notifications
- Checks for missing timesheet entries

**Cron Job:**
```sql
SELECT cron.schedule(
  'generate-weekly-timesheet-summaries',
  '0 9 * * 1',
  $$ ... $$
);
```

---

### 2. Late Clock-out Detection
**Schedule:** Every day at 10:00 PM UTC  
**Function:** `check-late-clockouts`

**What it does:**
- Finds active sessions older than 10 hours
- Sends notifications to affected admins
- Logs warnings in system

**Cron Job:**
```sql
SELECT cron.schedule(
  'check-late-clockouts',
  '0 22 * * *',
  $$ ... $$
);
```

---

### 3. Auto-timeout Mechanism
**Trigger:** Real-time (checked every minute when active)

**What it does:**
- Monitors last activity timestamp
- Auto-clocks out sessions inactive for 60+ minutes
- Sets status to 'auto_timeout'
- Sends notification to user

---

## 🔔 Notification & Approval Flow

### Notification Types

#### 1. **Manual Entry Submission**
- **Trigger:** Admin creates manual entry
- **Recipients:** Super Admins
- **Action Required:** Review and approve/reject

#### 2. **Manual Entry Decision**
- **Trigger:** Super Admin approves/rejects entry
- **Recipients:** Submitting admin
- **Content:** Approval status with optional rejection reason

#### 3. **Overlapping Entry Detection**
- **Trigger:** System detects overlapping timesheets
- **Recipients:** Super Admins
- **Action Required:** Review and resolve conflict

#### 4. **Missing Timesheet Entries**
- **Trigger:** Weekly check finds 2+ missing days
- **Recipients:** Super Admins
- **Action Required:** Follow up with admin

#### 5. **Weekly Reminder**
- **Trigger:** Scheduled weekly (Monday)
- **Recipients:** All active admins
- **Content:** Reminder to review and submit timesheets

#### 6. **Inactivity Warning**
- **Trigger:** 45 minutes of inactivity
- **Recipients:** Admin with active session
- **Action Required:** User activity to prevent auto-timeout

---

## 🔒 Security Features

### Database Level
1. **Row Level Security (RLS)**
   - Enforces user can only access their own data
   - Super Admins have unrestricted access
   
2. **Validation Triggers**
   - Prevents overlapping timesheet entries
   - Blocks future timestamps
   - Enforces manual entry approval workflow

3. **Immutable Audit Logs**
   - Activity logs cannot be modified or deleted
   - Complete audit trail for compliance

### API Level
1. **Authentication Required**
   - All endpoints require valid JWT token
   - User verification on every request

2. **Authorization Checks**
   - Super Admin role validation for approvals
   - User can only modify their own entries

3. **Input Validation**
   - Timestamp validation (no future dates)
   - Overlap detection
   - Required field enforcement

---

## 📊 Performance Optimizations

### Database Indexes
```sql
-- Faster queries by admin and date
CREATE INDEX idx_timesheets_admin_date ON timesheets(admin_id, date DESC);

-- Quick status filtering
CREATE INDEX idx_timesheets_status ON timesheets(status);

-- Chronological ordering
CREATE INDEX idx_timesheets_clock_in ON timesheets(clock_in DESC);

-- Activity log lookups
CREATE INDEX idx_timesheet_activity_logs_timesheet_id ON timesheet_activity_logs(timesheet_id);
CREATE INDEX idx_timesheet_activity_logs_task_id ON timesheet_activity_logs(task_id);
```

---

## 🧪 Testing Checklist

### Functional Testing
- [ ] Clock in creates new timesheet entry
- [ ] Clock out completes session and calculates duration
- [ ] Manual entry requires approval
- [ ] Super Admin can approve/reject entries
- [ ] Overlapping entries are prevented
- [ ] Future timestamps are blocked
- [ ] Auto-timeout works after 60 minutes
- [ ] Weekly reports generate correctly
- [ ] Notifications are sent properly

### Security Testing
- [ ] Non-admins cannot access timesheet endpoints
- [ ] Admins cannot see other admins' timesheets
- [ ] Only Super Admins can approve entries
- [ ] RLS policies enforce data isolation
- [ ] Activity logs are immutable

### Performance Testing
- [ ] Report generation completes in <2 seconds
- [ ] Dashboard loads in <1 second
- [ ] Database queries use indexes effectively
- [ ] Export functions handle large datasets

---

## 📚 API Usage Examples

### JavaScript/TypeScript
```typescript
// Clock in
const clockIn = async (taskId?: string) => {
  const response = await fetch(
    'https://cqoydkxlonzobykwjcin.supabase.co/functions/v1/timesheet-api/clock-in',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${session.access_token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        admin_id: user.id,
        task_id: taskId
      })
    }
  );
  return await response.json();
};

// Generate report
const getReport = async (startDate: string, endDate: string) => {
  const url = new URL('https://cqoydkxlonzobykwjcin.supabase.co/functions/v1/timesheet-api/report');
  url.searchParams.append('start_date', startDate);
  url.searchParams.append('end_date', endDate);
  
  const response = await fetch(url, {
    headers: {
      'Authorization': `Bearer ${session.access_token}`,
    }
  });
  return await response.json();
};
```

---

## 🚀 Deployment Notes

### Edge Functions
All edge functions are automatically deployed when code is pushed:
- `timesheet-api` - Main API endpoint
- `check-late-clockouts` - Daily late clock-out monitor
- `timesheet-weekly-tasks` - Weekly summary generator

### Cron Jobs
Cron jobs are configured via database migration and require:
- `pg_cron` extension enabled
- `pg_net` extension for HTTP calls
- Proper permissions granted to postgres user

### Environment Variables
Required secrets (already configured):
- `SUPABASE_URL`
- `SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`

---

## 📖 User Roles & Permissions

| Action | Admin | Super Admin |
|--------|-------|-------------|
| Clock in/out | ✅ Own | ✅ Own |
| View timesheets | ✅ Own | ✅ All |
| Create manual entry | ✅ | ✅ |
| Approve manual entry | ❌ | ✅ |
| View all timesheets | ❌ | ✅ |
| Generate reports | ✅ Own | ✅ All |
| Export data | ✅ Own | ✅ All |

---

## 🐛 Troubleshooting

### Issue: Clock in fails with "Active session already exists"
**Solution:** Check for existing active session and clock out first.

### Issue: Manual entry rejected with "overlaps"
**Solution:** Verify no existing entries in the same time range.

### Issue: Approval fails with "Unauthorized"
**Solution:** Verify user has Super Admin role and is approved/active.

### Issue: Reports show no data
**Solution:** Check date filters and ensure user has appropriate permissions.

---

## 📞 Support

For issues or questions:
1. Check logs in Supabase Dashboard → Edge Functions
2. Review database policies in Supabase Dashboard → Database → Policies
3. Verify cron jobs in Supabase SQL Editor: `SELECT * FROM cron.job;`

---

**Version:** 1.0  
**Last Updated:** 2025-10-11  
**Maintained by:** Timesheet System Development Team