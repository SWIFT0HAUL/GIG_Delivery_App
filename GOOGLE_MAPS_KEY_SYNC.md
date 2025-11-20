# Google Maps API Key Synchronization Guide

## Overview

The system now supports syncing Google Maps API keys from the frontend admin panel to Supabase secrets, ensuring both frontend and backend use the same API key.

## How It Works

### 1. **Frontend Storage** (`.env` files)
The Google Maps API key is stored in environment variables:
```
VITE_GOOGLE_MAPS_API_KEY="AIzaSyBczEjmsmbHFiGNN9eg6FYT-PYmb3TkkVI"
```

This key is used by:
- Address autocomplete components
- Geocoding utilities on the client side
- Any frontend map displays

### 2. **Backend Storage** (Supabase Secrets)
The same key is stored as a Supabase secret: `GOOGLE_MAPS_API_KEY`

This key is used by:
- `autocomplete` edge function
- `place-details` edge function  
- `geocode` edge function
- Any other backend services that need Google Maps API access

### 3. **Synchronization from Admin Panel**

Admins can update the Google Maps API key through the admin interface:

**Path:** Admin Dashboard → Platform Settings → Security Tab → API Keys

**Steps:**
1. Navigate to the Security tab
2. Find "Google Maps API" in the API Keys table
3. Click the Edit (pencil) icon
4. Enter the new API key
5. Click Save (checkmark icon)

**What Happens:**
- The key is validated (must be at least 10 characters)
- The key is sent to the `update-google-maps-key` edge function
- The edge function updates the Supabase secret `GOOGLE_MAPS_API_KEY`
- Edge functions will use the new key (may require redeployment)
- The frontend continues to use the value from `.env` until updated

## Updating Both Frontend and Backend

### Option 1: Update via Admin Panel (Recommended)
1. Update the key in Admin Panel (updates Supabase secrets)
2. Manually update `.env`, `.env.development`, and `.env.production` files
3. Rebuild the frontend application

### Option 2: Manual Update
1. Update `.env` files:
   ```bash
   # .env
   VITE_GOOGLE_MAPS_API_KEY="your-new-key-here"
   ```
2. Add the key to Supabase secrets via Supabase Dashboard:
   - Go to Project Settings → Edge Functions → Secrets
   - Add/Update `GOOGLE_MAPS_API_KEY`

### Option 3: Use Admin Panel + Environment Variable Sync
1. Update in Admin Panel (syncs to Supabase)
2. Copy the new key from the admin panel
3. Update your `.env` files
4. Redeploy your application

## Security Considerations

1. **Admin Access Only**: Only users with `admin` or `super_admin` role can update API keys
2. **Validation**: Keys must be at least 10 characters long
3. **Audit Trail**: All updates are logged (consider adding audit logs)
4. **Key Visibility**: Keys are partially masked in the UI for security

## Required Configuration

### Supabase Access Token
To enable automatic secret updates, you need to configure `SUPABASE_ACCESS_TOKEN`:

1. Generate a Supabase access token from your Supabase dashboard
2. Add it to Supabase secrets:
   ```
   SUPABASE_ACCESS_TOKEN=<your-management-api-token>
   ```

Without this token, the update function will still accept the key but won't be able to update Supabase secrets automatically.

## Edge Function: `update-google-maps-key`

### Purpose
Securely updates the Google Maps API key in Supabase secrets.

### Authentication
- Requires valid Supabase auth token
- Verifies user has admin or super_admin role

### Request
```typescript
POST /functions/v1/update-google-maps-key
Headers:
  Authorization: Bearer <supabase-auth-token>
Body:
  {
    "apiKey": "your-google-maps-api-key"
  }
```

### Response
```typescript
{
  "success": true,
  "message": "Google Maps API key updated successfully in Supabase secrets",
  "note": "Edge functions will use the new key after redeployment"
}
```

## Troubleshooting

### Issue: Edge functions still using old key
**Solution**: Redeploy edge functions after updating the secret

### Issue: "SUPABASE_ACCESS_TOKEN not configured" error
**Solution**: Add the management API token to Supabase secrets

### Issue: Frontend shows old key
**Solution**: Update `.env` files and rebuild the application

### Issue: Autocomplete not working after key update
**Solution**: 
1. Check if edge functions are deployed
2. Verify the key in Supabase secrets
3. Test the edge function directly
4. Check browser console for errors

## Best Practices

1. **Always Update Both**: When changing the API key, update both frontend (.env) and backend (Supabase secrets)
2. **Test After Update**: Verify autocomplete and geocoding work after updating
3. **Keep Keys Secure**: Never commit API keys to version control
4. **Use Restrictions**: Configure API key restrictions in Google Cloud Console
5. **Monitor Usage**: Track API usage in Google Cloud Console to detect issues

## API Key Restrictions

Configure these restrictions in Google Cloud Console for security:

1. **Application Restrictions**:
   - HTTP referrers: Add your domain(s)
   
2. **API Restrictions**:
   - Places API
   - Geocoding API
   - Maps JavaScript API (if using map displays)

## Related Files

- **Edge Functions**:
  - `supabase/functions/update-google-maps-key/index.ts` - Secret update function
  - `supabase/functions/autocomplete/index.ts` - Address autocomplete
  - `supabase/functions/place-details/index.ts` - Place details lookup
  - `supabase/functions/geocode/index.ts` - Geocoding service

- **Frontend**:
  - `src/components/admin/PlatformSettings.tsx` - Admin interface
  - `src/components/ui/address-autocomplete.tsx` - Autocomplete component
  - `src/lib/geocoding.ts` - Geocoding utilities

- **Configuration**:
  - `.env` - Local environment
  - `.env.development` - Development environment
  - `.env.production` - Production environment
