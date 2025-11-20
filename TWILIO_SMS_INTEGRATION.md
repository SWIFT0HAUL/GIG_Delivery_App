# Twilio SMS Integration Guide

## Overview
This platform now includes Twilio SMS integration for sending verification codes, MFA messages, and notifications to users via text message.

## Setup

### 1. Twilio Account Configuration
The following secrets have been configured in Supabase:
- `TWILIO_ACCOUNT_SID` - Your Twilio Account SID
- `TWILIO_AUTH_TOKEN` - Your Twilio Auth Token  
- `TWILIO_PHONE_NUMBER` - Your Twilio phone number (in E.164 format, e.g., +15551234567)

These secrets are automatically available to the edge functions.

### 2. Edge Function
The `send-sms` edge function is deployed and ready to use. It:
- Validates Twilio credentials
- Formats phone numbers to E.164 format
- Sends SMS via Twilio API
- Returns success/error responses
- Includes proper CORS headers
- Requires authentication (JWT token)

## Usage

### Using the React Hook

```typescript
import { useTwilioSMS } from '@/hooks/useTwilioSMS';

function MyComponent() {
  const { sendSMS, sendVerificationCode, sendMFACode, sendNotification, sending } = useTwilioSMS();

  // Send a verification code
  const handleSendVerification = async () => {
    const result = await sendVerificationCode('+15551234567', '123456');
    if (result.success) {
      console.log('Code sent!', result.sid);
    }
  };

  // Send an MFA code
  const handleSendMFA = async () => {
    const result = await sendMFACode('+15551234567', '987654');
    if (result.success) {
      console.log('MFA code sent!');
    }
  };

  // Send a custom notification
  const handleSendNotification = async () => {
    const result = await sendNotification(
      '+15551234567', 
      'Your order has been shipped!'
    );
  };

  // Send a custom SMS
  const handleCustomSMS = async () => {
    const result = await sendSMS({
      to: '+15551234567',
      message: 'Custom message here',
      type: 'notification'
    });
  };

  return (
    <button onClick={handleSendVerification} disabled={sending}>
      {sending ? 'Sending...' : 'Send Code'}
    </button>
  );
}
```

### Using the Utility Functions

```typescript
import { 
  generateVerificationCode, 
  formatPhoneNumber, 
  isValidPhoneNumber,
  storeVerificationCode,
  verifyStoredCode 
} from '@/lib/smsUtils';

// Generate a random verification code
const code = generateVerificationCode(6); // Returns a 6-digit code

// Format phone number
const formatted = formatPhoneNumber('5551234567'); // Returns '+15551234567'
const formattedWithCode = formatPhoneNumber('5551234567', '44'); // UK number

// Validate phone number
const isValid = isValidPhoneNumber('5551234567'); // Returns true/false

// Store verification code (expires in 10 minutes by default)
storeVerificationCode('+15551234567', code, 10);

// Verify code entered by user
const isCorrect = verifyStoredCode('+15551234567', '123456');
```

### Using the PhoneVerification Component

```typescript
import { PhoneVerification } from '@/components/auth/PhoneVerification';

function MyOnboardingPage() {
  const handleVerified = (phoneNumber: string) => {
    console.log('Phone verified:', phoneNumber);
    // Continue with your onboarding flow
  };

  return (
    <PhoneVerification
      onVerified={handleVerified}
      title="Verify Your Phone"
      description="We'll send you a verification code"
    />
  );
}
```

## Integration Examples

### 1. Phone Verification During Onboarding

```typescript
import { PhoneVerification } from '@/components/auth/PhoneVerification';
import { supabase } from '@/integrations/supabase/client';

function OnboardingStep() {
  const handlePhoneVerified = async (phoneNumber: string) => {
    // Update user profile with verified phone
    const { error } = await supabase
      .from('profiles')
      .update({ 
        phone: phoneNumber,
        phone_verified: true 
      })
      .eq('id', user.id);
    
    if (!error) {
      // Continue to next step
      navigate('/onboarding/next-step');
    }
  };

  return <PhoneVerification onVerified={handlePhoneVerified} />;
}
```

### 2. MFA Setup with SMS

```typescript
import { useTwilioSMS } from '@/hooks/useTwilioSMS';
import { generateVerificationCode, storeVerificationCode } from '@/lib/smsUtils';

function MFASetup() {
  const { sendMFACode } = useTwilioSMS();
  const [phoneNumber, setPhoneNumber] = useState('');

  const handleEnableMFA = async () => {
    // Generate and send MFA code
    const code = generateVerificationCode(6);
    storeVerificationCode(phoneNumber, code, 10);
    
    const result = await sendMFACode(phoneNumber, code);
    
    if (result.success) {
      // Save MFA settings in database
      await supabase
        .from('profiles')
        .update({ 
          mfa_enabled: true,
          mfa_phone: phoneNumber 
        })
        .eq('id', user.id);
    }
  };

  return (
    // Your MFA setup UI
  );
}
```

### 3. Send Job Notifications

```typescript
import { useTwilioSMS } from '@/hooks/useTwilioSMS';

function JobAssignment() {
  const { sendNotification } = useTwilioSMS();

  const notifyDriver = async (driverId: string, jobDetails: any) => {
    // Get driver phone number
    const { data: driver } = await supabase
      .from('profiles')
      .select('phone')
      .eq('id', driverId)
      .single();

    if (driver?.phone) {
      await sendNotification(
        driver.phone,
        `New job assigned! Pickup: ${jobDetails.pickup_location.address}`
      );
    }
  };

  return (
    // Your job assignment UI
  );
}
```

## Phone Number Format

All phone numbers should be in E.164 format:
- Include country code: `+15551234567` (US)
- No spaces, dashes, or parentheses
- The utility functions handle formatting automatically

## Error Handling

The SMS functions return a response object:

```typescript
interface SendSMSResponse {
  success: boolean;
  message?: string;  // Success message
  error?: string;    // Error message
  sid?: string;      // Twilio message SID
  status?: string;   // Twilio message status
}
```

Always check the `success` field:

```typescript
const result = await sendVerificationCode(phone, code);
if (result.success) {
  // Handle success
  console.log('Message SID:', result.sid);
} else {
  // Handle error
  console.error('Error:', result.error);
}
```

## Security Best Practices

1. **Never expose verification codes in logs**
2. **Set reasonable expiration times** (default is 10 minutes)
3. **Implement rate limiting** on the frontend
4. **Validate phone numbers** before sending
5. **Store codes securely** (current implementation uses localStorage with expiration)
6. **Use HTTPS** for all API calls (automatically handled)

## Testing

To test SMS functionality:

1. Use your personal phone number during development
2. Check the Twilio console for message logs
3. Monitor edge function logs for errors

## Cost Considerations

- Each SMS message costs according to Twilio pricing
- Implement rate limiting to prevent abuse
- Consider using SMS sparingly for cost-sensitive applications

## Troubleshooting

### Common Issues:

1. **"Missing Twilio credentials"**
   - Verify secrets are configured in Supabase
   - Check edge function logs

2. **"Invalid phone number"**
   - Ensure E.164 format (+15551234567)
   - Use `formatPhoneNumber()` utility

3. **"Failed to send SMS"**
   - Check Twilio account balance
   - Verify phone number is valid and not blocked
   - Check Twilio console for detailed error messages

4. **Authentication errors**
   - Ensure user is logged in (JWT token required)
   - Check Supabase auth state

## Next Steps

- Integrate phone verification into your onboarding flow
- Add SMS notifications for job updates
- Implement MFA with SMS codes
- Add rate limiting for SMS sending
- Create admin panel to monitor SMS usage

## Resources

- [Twilio Documentation](https://www.twilio.com/docs)
- [Supabase Edge Functions](https://supabase.com/docs/guides/functions)
- [Edge Function Logs](https://supabase.com/dashboard/project/cqoydkxlonzobykwjcin/functions/send-sms/logs)
