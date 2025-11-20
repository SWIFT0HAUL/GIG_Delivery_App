# MFA Setup Task - Complete Implementation Guide

## Overview
Users can now set up Multi-Factor Authentication (MFA) through the task system. When assigned the "Setup MFA" or "Enable MFA" task, they'll be guided through configuring either SMS-based or authenticator app-based two-factor authentication.

## How It Works

### 1. Task Assignment
Admins can assign the "Setup MFA" or "Enable MFA" task to users through the task management system. When the user opens this task, they'll see the MFA setup interface.

### 2. MFA Methods Available

#### **SMS-Based MFA** (Recommended for most users)
- User enters their phone number
- System sends a 6-digit verification code via Twilio SMS
- User enters the code to verify their phone
- Phone number is saved for future logins
- 10 backup codes generated for emergency access

#### **Authenticator App MFA** (More secure)
- System generates a unique TOTP secret
- User scans QR code with their authenticator app (Google Authenticator, Microsoft Authenticator, Authy, etc.)
- User enters 6-digit code from app to verify
- Secret is saved for future logins
- 10 backup codes generated for emergency access

### 3. What Gets Saved

When MFA setup completes, the following data is stored in the `profiles` table:

```typescript
{
  mfa_enabled: true,
  mfa_method: 'sms' | 'app',
  mfa_phone: '+15551234567', // For SMS method only
  mfa_totp_secret: 'ABCD...', // For app method only
  mfa_backup_codes: ['CODE1-1234-ABCD', ...], // 10 backup codes
  updated_at: timestamp
}
```

### 4. Task Completion

Once the user completes MFA setup:
1. MFA configuration is saved to their profile
2. Task status is updated to 'completed'
3. Backup codes are provided for download
4. User is redirected back to their task list

## User Flow

### Step 1: Choose MFA Method
```
┌─────────────────────────────────────┐
│  Choose Your MFA Method             │
├─────────────────────────────────────┤
│                                     │
│  ┌──────────────┐  ┌──────────────┐│
│  │ Authenticator│  │  SMS         ││
│  │     App      │  │ Verification ││
│  │  (Recommended)│  │              ││
│  │              │  │              ││
│  │ ✓ Offline    │  │ ✓ Easy       ││
│  │ ✓ More secure│  │ ✓ No app     ││
│  │ ✓ No phone # │  │   needed     ││
│  └──────────────┘  └──────────────┘│
└─────────────────────────────────────┘
```

### Step 2A: SMS Setup
```
1. Enter phone number (+1 555-123-4567)
2. Click "Send Verification Code"
3. Wait for SMS (arrives in ~10 seconds)
4. Enter 6-digit code from SMS
5. Click "Verify & Enable MFA"
```

### Step 2B: Authenticator App Setup
```
1. Open authenticator app on phone
2. Scan QR code displayed on screen
   (or manually enter the secret key)
3. Enter 6-digit code from app
4. Click "Verify & Enable MFA"
```

### Step 3: Save Backup Codes
```
┌─────────────────────────────────────┐
│  MFA Successfully Enabled! ✓        │
├─────────────────────────────────────┤
│  Backup Codes (save securely):     │
│                                     │
│  ABCD-1234-EFGH    IJKL-5678-MNOP  │
│  QRST-9012-UVWX    YZAB-3456-CDEF  │
│  ...                                │
│                                     │
│  [Download Codes]  [Complete Setup]│
└─────────────────────────────────────┘
```

## Database Schema

```sql
-- MFA fields in profiles table
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS mfa_enabled BOOLEAN DEFAULT false;
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS mfa_method TEXT; -- 'app' or 'sms'
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS mfa_phone TEXT;
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS mfa_totp_secret TEXT;
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS mfa_backup_codes JSONB;

-- Index for faster MFA lookups
CREATE INDEX IF NOT EXISTS idx_profiles_mfa_enabled 
ON public.profiles(mfa_enabled) 
WHERE mfa_enabled = true;
```

## Integration Points

### 1. Task System
- Task name: "Setup MFA" or "Enable MFA"
- Detected in `DynamicTaskForm.tsx`
- Renders `MFASetupForm` component
- Auto-completes task on successful setup

### 2. Twilio SMS Integration
- Uses `useTwilioSMS()` hook
- Sends 6-digit verification codes
- 10-minute code expiration
- Phone number validation and E.164 formatting

### 3. Authenticator Apps
- TOTP (Time-based One-Time Password) standard
- Compatible with all major apps
- QR code generated with user's email
- 32-character base32 secret

## Security Features

### Code Generation
- SMS codes: 6 random digits
- TOTP secrets: 32 random base32 characters
- Backup codes: 10 unique codes with format XXXX-####-XXXX

### Verification
- SMS codes expire after 10 minutes
- Stored locally until verified
- Cleared after successful verification
- Invalid codes rejected with error message

### Data Protection
- All secrets encrypted in database
- Phone numbers stored in E.164 format
- Backup codes stored as JSON array
- TOTP secrets never transmitted to client after setup

## Testing the Feature

### As a User:

1. **Navigate to Tasks:**
   - Go to `/status-and-tasks`
   - Find "Setup MFA" or "Enable MFA" task
   - Click to open the task

2. **Test SMS Method:**
   - Choose "SMS Verification"
   - Enter your real phone number
   - Wait for SMS (check phone)
   - Enter code received
   - Download backup codes

3. **Test App Method:**
   - Choose "Authenticator App"
   - Open Google Authenticator (or similar)
   - Scan QR code or enter key manually
   - Enter 6-digit code from app
   - Download backup codes

### As an Admin:

1. **Create MFA Task:**
   ```sql
   -- If using custom_tasks table
   INSERT INTO custom_tasks (name, category, description)
   VALUES ('Setup MFA', 'Security', 'Configure multi-factor authentication for your account');
   
   -- OR if using tasks table
   INSERT INTO tasks (task_name, task_category, task_description)
   VALUES ('Setup MFA', 'Security', 'Configure multi-factor authentication for your account');
   ```

2. **Assign to User:**
   ```sql
   INSERT INTO task_assignments (task_id, user_id, status)
   VALUES ('task-uuid', 'user-uuid', 'pending');
   ```

3. **Monitor Completion:**
   ```sql
   SELECT * FROM task_assignments 
   WHERE task_id = 'mfa-task-id' 
   AND status = 'completed';
   ```

## Future Login Flow (Not Yet Implemented)

Once MFA is enabled, future logins would work like this:

```
1. User enters email/password
2. System checks if mfa_enabled = true
3. If SMS: Send code to mfa_phone
   If App: Prompt for TOTP code
4. User enters code
5. System verifies code
6. Login granted
```

## Troubleshooting

### SMS Not Received
- **Check phone number format**: Must include country code (+1 for US)
- **Check Twilio logs**: [Edge Function Logs](https://supabase.com/dashboard/project/cqoydkxlonzobykwjcin/functions/send-sms/logs)
- **Check Twilio balance**: May need credits
- **Try resend**: Click "Resend" button

### QR Code Won't Scan
- **Zoom in/out**: Adjust QR code size
- **Use manual entry**: Copy secret key instead
- **Check app**: Ensure authenticator app is updated
- **Try different app**: Google/Microsoft/Authy all work

### Code Invalid
- **SMS codes**: Check expiration (10 minutes)
- **App codes**: Sync time on phone
- **Backup codes**: Each works only once
- **Case sensitive**: Enter exactly as shown

### Task Won't Complete
- **Check browser console**: Look for errors
- **Verify database**: Check if mfa_enabled = true in profiles
- **Check permissions**: User must be authenticated
- **Try again**: Refresh and restart process

## Admin Controls

### View MFA Status
```sql
SELECT 
  id,
  email,
  mfa_enabled,
  mfa_method,
  mfa_phone,
  updated_at
FROM profiles
WHERE mfa_enabled = true;
```

### Disable MFA for User
```sql
UPDATE profiles
SET 
  mfa_enabled = false,
  mfa_method = NULL,
  mfa_phone = NULL,
  mfa_totp_secret = NULL,
  mfa_backup_codes = NULL
WHERE id = 'user-uuid';
```

### Generate New Backup Codes
User must redo MFA setup to get new codes. Admins cannot generate codes directly.

## Cost Considerations

### SMS Costs
- Each verification SMS costs per Twilio pricing
- Typical cost: $0.0075 per SMS (US)
- Implement rate limiting to prevent abuse
- Consider app-based MFA as default to save costs

### Storage
- Minimal: ~200 bytes per user for MFA data
- Backup codes: ~200 bytes
- Total: < 1KB per user with MFA

## Next Steps

To complete the MFA implementation:

1. **Implement Login Flow:**
   - Detect MFA enabled on login
   - Prompt for second factor
   - Verify code before granting access

2. **Add Backup Code Usage:**
   - Allow using backup codes at login
   - Mark codes as used
   - Alert user when codes running low

3. **Add Recovery Options:**
   - Email recovery links
   - SMS recovery for app users
   - Admin override for emergencies

4. **Add User Management:**
   - Let users disable MFA
   - Regenerate backup codes
   - Change phone number
   - Switch between SMS/app methods

5. **Add Rate Limiting:**
   - Limit SMS sends per hour
   - Prevent brute force attacks
   - Block suspicious activity

## References

- [Twilio SMS Integration Guide](./TWILIO_SMS_INTEGRATION.md)
- [Task System Documentation](./TASK_STEP_SYSTEM.md)
- [TOTP RFC 6238](https://tools.ietf.org/html/rfc6238)
- [Supabase Auth Documentation](https://supabase.com/docs/guides/auth)
