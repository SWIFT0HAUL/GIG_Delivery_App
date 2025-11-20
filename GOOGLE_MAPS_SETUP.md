# Google Maps Strict API Configuration

This application uses a **strict, controlled Google Maps API management system** that ensures secure and predictable API usage.

## ⚡ Quick Start

The Google Maps API key is managed through a centralized `GoogleMapsAPIManager` that:
- ✅ Restricts access to only approved services (geocode, directions, distancematrix, places)
- ✅ Provides a single point of control for API key management
- ✅ Prevents unauthorized API usage
- ✅ Automatically handles script loading and cleanup

## 🔧 Setup Instructions

### 1. Get a Google Maps API Key

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable **ONLY** these APIs:
   - **Places API**
   - **Maps JavaScript API**
   - **Geocoding API** (if needed)
   - **Directions API** (if needed)
   - **Distance Matrix API** (if needed)
4. Go to **APIs & Services** > **Credentials**
5. Click **Create Credentials** > **API Key**
6. Copy your API key

### 2. Restrict Your API Key (CRITICAL for Security)

⚠️ **IMPORTANT**: Always restrict your API keys to prevent unauthorized usage and unexpected charges.

1. In Google Cloud Console, go to **APIs & Services** > **Credentials**
2. Click on your API key
3. Under **Application restrictions**, select **HTTP referrers (web sites)**
4. Add your allowed referrers:
   - For development: `http://localhost:*` or your Lovable preview URL
   - For production: `https://yourdomain.com/*`
5. Under **API restrictions**, select **Restrict key**
6. Choose ONLY:
   - Maps JavaScript API
   - Places API
   - (Optional) Geocoding API
   - (Optional) Directions API
   - (Optional) Distance Matrix API
7. Click **Save**

### 3. Configure API Key in Your Project

#### Option A: Environment Variable (Recommended)
Add to your `.env` file:
```
VITE_GOOGLE_MAPS_API_KEY=your_api_key_here
```

#### Option B: Admin Settings UI
1. Log in as admin
2. Go to **Admin → Settings → Integrations → Google Maps**
3. Enter your API key
4. Save settings

#### Option C: Programmatic Access (Advanced)
You can also set the key programmatically using the global API manager:
```javascript
window.APIKeyManager.set('map', 'your_api_key_here');
```

## 🏗️ Architecture

### Strict API Manager

The application uses `GoogleMapsAPIManager` located in `src/lib/googleMapsApiManager.ts`:

```typescript
// Access the API manager
import { GoogleMapsAPIManager } from '@/lib/googleMapsApiManager';

// Get current API key
const apiKey = GoogleMapsAPIManager.get('map');

// Set/update API key
GoogleMapsAPIManager.set('map', 'new_api_key');

// Load Google Maps asynchronously
await GoogleMapsAPIManager.loadAsync();

// Check if a service is allowed
if (GoogleMapsAPIManager.isServiceAllowed('places')) {
  // Use places service
}

// Get list of allowed services
const services = GoogleMapsAPIManager.getAllowedServices();
// Returns: ['geocode', 'directions', 'distancematrix', 'places']
```

### Global Window Access

The API manager is also available globally:

```javascript
// Set API key
window.APIKeyManager.set('map', 'your_key');

// Get API key
const key = window.APIKeyManager.get('map');

// Load Google Maps with callback
window.GoogleAPI.load(() => {
  console.log('Google Maps loaded!');
});
```

## 🗺️ Usage Examples

### In React Components

```typescript
import { useEffect } from 'react';
import { GoogleMapsAPIManager } from '@/lib/googleMapsApiManager';

function MyMapComponent() {
  useEffect(() => {
    GoogleMapsAPIManager.loadAsync()
      .then(() => {
        // Google Maps is ready
        const map = new google.maps.Map(document.getElementById('map'), {
          center: { lat: 37.7749, lng: -122.4194 },
          zoom: 12
        });
      })
      .catch((error) => {
        console.error('Failed to load Google Maps:', error);
      });
  }, []);

  return <div id="map" style={{ width: '100%', height: '400px' }} />;
}
```

### Address Autocomplete

The `AddressAutocomplete` component automatically uses the API manager:

```tsx
import { AddressAutocomplete } from '@/components/ui/address-autocomplete';

function MyForm() {
  return (
    <AddressAutocomplete
      label="Delivery Address"
      onAddressSelect={(address) => {
        console.log('Selected:', address);
      }}
      required
    />
  );
}
```

## 🔒 Security Features

### Allowed Services Only
The API manager restricts usage to these services ONLY:
- ✅ `geocode` - Convert addresses to coordinates
- ✅ `directions` - Get route directions
- ✅ `distancematrix` - Calculate distances
- ✅ `places` - Address autocomplete and place search

Any attempt to use other services will be blocked at the application level.

### Single Point of Control
- All API key access goes through `GoogleMapsAPIManager`
- Centralized script loading prevents duplicate API calls
- Automatic cleanup when keys are removed or changed

### Validation
```typescript
// Check if service is allowed before use
if (!GoogleMapsAPIManager.isServiceAllowed('someService')) {
  console.error('Service not allowed');
  return;
}
```


## 📍 Features & Integration

### Integrated Components

The strict API manager is integrated into:

- **Address Autocomplete**: All address input fields throughout the platform
- **Map Views**: `GoogleMapView` and `LiveViewMap` components
- **Onboarding Forms**: Driver, Carrier, Broker, Shipper, Vendor/Merchant
- **Profile Settings**: User address management
- **Job Creation**: Pickup and delivery location selection

### Automatic Features

- 🔍 **Smart autocomplete**: Start typing and get USA address suggestions
- 📍 **Automatic parsing**: Fills street, city, state, and ZIP code automatically
- ✏️ **Manual entry fallback**: Falls back to text input if API unavailable
- 🇺🇸 **USA-focused**: Restricted to United States addresses
- 🔄 **Real-time updates**: Changes to API key in admin settings apply instantly
- ⚡ **Optimized loading**: Single script load, shared across all components

### Fallback Behavior

If the API key is not configured or there's an error:
- System automatically falls back to regular text input fields
- Users can still enter addresses manually
- Subtle message informs users that autocomplete is unavailable

## 🐛 Troubleshooting

### Autocomplete not working?

1. **Check API Key Configuration**:
   ```javascript
   console.log(window.APIKeyManager.get('map'));
   ```
   Should return your API key (not empty)

2. **Verify API is enabled** in Google Cloud Console:
   - Places API ✓
   - Maps JavaScript API ✓

3. **Check HTTP Referrers**: Your current domain/URL must be in the allowed list

4. **Browser Console**: Press F12 and check for errors
   - Look for "Failed to load Google Maps" messages
   - Check for API key errors or quota exceeded

5. **Service Validation**:
   ```javascript
   console.log(window.APIKeyManager.getAllowedServices());
   ```
   Should show: `['geocode', 'directions', 'distancematrix', 'places']`

### Common Error Messages

| Error | Solution |
|-------|----------|
| "Google Maps API key not configured" | Set API key in admin settings or `.env` |
| "Places service is not allowed" | Service restriction - should not occur with default config |
| "Failed to load Google Maps script" | Check API key validity and network connection |
| "Loading timeout" | Check network or Google Maps API status |

### Force Reload API

If you need to reload the Google Maps script:
```javascript
window.APIKeyManager.delete('map');
window.APIKeyManager.set('map', 'your_new_key');
```

## 💰 Cost & Quotas

### Google Maps Pricing (2024)

- **Free tier**: $200 credit per month
- **Autocomplete requests**: ~$2.83 per 1,000 requests
- **Free autocomplete**: ~28,000 requests with the credit
- **Maps JavaScript API**: $7 per 1,000 map loads

### Recommendations

✅ **DO:**
- Set up billing alerts in Google Cloud Console
- Monitor usage regularly
- Use HTTP referrer restrictions
- Implement rate limiting if needed

❌ **DON'T:**
- Exceed free tier without monitoring
- Leave API key unrestricted
- Use same key across multiple projects

## 🔐 API Key Security Best Practices

### ✅ Essential Security Measures

1. **Always restrict by HTTP referrer** (domain/URL)
2. **Restrict which APIs the key can access** (only what you need)
3. **Monitor usage** in Google Cloud Console
4. **Set up billing alerts** to prevent surprise charges
5. **Use separate keys** for development and production
6. **Rotate keys periodically** (every 90-180 days)

### ❌ Security Don'ts

- ❌ Share API keys publicly or commit to git
- ❌ Use same key for multiple projects
- ❌ Leave keys unrestricted
- ❌ Ignore usage spikes or anomalies
- ❌ Hardcode keys in source code (use env variables)

## 📚 Additional Resources

- [Google Maps Platform Documentation](https://developers.google.com/maps/documentation)
- [Places API Documentation](https://developers.google.com/maps/documentation/places/web-service/overview)
- [API Key Best Practices](https://developers.google.com/maps/api-security-best-practices)
- [Google Maps Pricing](https://mapsplatform.google.com/pricing/)
- [Billing and Usage](https://console.cloud.google.com/google/maps-apis/metrics)

## 🆘 Support

For issues specific to this implementation:
1. Check browser console for error messages
2. Verify API key configuration in admin settings
3. Test with the troubleshooting commands above
4. Review Google Cloud Console for API errors

For Google Maps API issues:
- [Google Maps Support](https://developers.google.com/maps/support)
- [Stack Overflow - Google Maps](https://stackoverflow.com/questions/tagged/google-maps)

