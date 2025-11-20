# Domain Configuration & Deployment Guide

This app is now configured to work with custom domains. Follow the steps below based on your hosting platform.

## Environment Configuration

The app uses environment variables to determine the base URL:

- **Development**: `.env.development` → `http://localhost:8080`
- **Production**: `.env.production` → `https://mydomain.com`

Update `.env.production` with your actual domain before deploying.

## Supabase Configuration

### 1. Update Redirect URLs
In your Supabase dashboard, go to **Authentication → URL Configuration**:

- **Site URL**: `https://mydomain.com`
- **Redirect URLs**: Add both:
  - `https://mydomain.com/**`
  - `https://mydomain.com/auth/callback`

### 2. Update OAuth Providers (if using)
For each OAuth provider (Google, GitHub, etc.), update the callback URLs:
- **Authorized redirect URLs**: `https://mydomain.com/auth/callback`

## Deployment Options

### Option 1: Vercel

1. **Connect Repository**:
   ```bash
   # Push your code to GitHub
   git init
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Deploy to Vercel**:
   - Go to [vercel.com](https://vercel.com)
   - Import your GitHub repository
   - Vercel will auto-detect Vite configuration

3. **Add Custom Domain**:
   - Go to Project → Settings → Domains
   - Add `mydomain.com`
   - Follow DNS configuration instructions

4. **Environment Variables**:
   - Go to Project → Settings → Environment Variables
   - Add: `VITE_BASE_URL` = `https://mydomain.com`
   - Redeploy

### Option 2: Netlify

1. **Connect Repository**:
   ```bash
   # Push to GitHub (same as above)
   ```

2. **Deploy to Netlify**:
   - Go to [netlify.com](https://netlify.com)
   - New site from Git → Choose your repository
   - Build command: `npm run build`
   - Publish directory: `dist`

3. **Add Custom Domain**:
   - Go to Site settings → Domain management
   - Add custom domain → `mydomain.com`
   - Configure DNS as instructed

4. **Environment Variables**:
   - Go to Site settings → Environment variables
   - Add: `VITE_BASE_URL` = `https://mydomain.com`
   - Redeploy

### Option 3: VPS (Ubuntu/DigitalOcean/AWS EC2)

1. **Install Node.js**:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

2. **Install Nginx**:
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```

3. **Configure Nginx**:
   Create `/etc/nginx/sites-available/mydomain.com`:
   ```nginx
   server {
       listen 80;
       server_name mydomain.com www.mydomain.com;
       
       location / {
           proxy_pass http://localhost:8080;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

4. **Enable Site & Reload Nginx**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/mydomain.com /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

5. **Install SSL Certificate (Let's Encrypt)**:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   sudo certbot --nginx -d mydomain.com -d www.mydomain.com
   ```

6. **Clone & Build App**:
   ```bash
   cd /var/www
   git clone <your-repo-url> mydomain
   cd mydomain
   npm install
   npm run build
   ```

7. **Run with PM2** (Process Manager):
   ```bash
   sudo npm install -g pm2
   pm2 start npm --name "hauleer-app" -- run preview
   pm2 startup
   pm2 save
   ```

### Option 4: Render

1. **Connect Repository**:
   - Go to [render.com](https://render.com)
   - New → Static Site
   - Connect your GitHub repository

2. **Build Settings**:
   - Build Command: `npm run build`
   - Publish Directory: `dist`

3. **Add Custom Domain**:
   - Go to Settings → Custom Domain
   - Add `mydomain.com`
   - Configure DNS with the provided CNAME

4. **Environment Variables**:
   - Add: `VITE_BASE_URL` = `https://mydomain.com`
   - Redeploy

## DNS Configuration

For all platforms, update your DNS records at your domain registrar:

### For Vercel/Netlify:
- **Type**: CNAME
- **Name**: `www` (or `@` for root domain)
- **Value**: Provided by platform

### For VPS:
- **Type**: A Record
- **Name**: `@`
- **Value**: Your server IP address
- **Type**: A Record
- **Name**: `www`
- **Value**: Your server IP address

## Testing

After deployment:

1. Test authentication flow: `https://mydomain.com/auth`
2. Test OAuth redirects (if configured)
3. Test API endpoints
4. Verify email confirmations redirect correctly

## Troubleshooting

### Issue: "Invalid redirect URL"
- **Fix**: Add the URL to Supabase → Authentication → URL Configuration

### Issue: OAuth callback fails
- **Fix**: Update OAuth provider settings with new callback URL

### Issue: CORS errors
- **Fix**: Ensure Supabase project allows your domain

### Issue: SSL certificate not auto-renewing
- **Fix** (VPS only):
  ```bash
  sudo certbot renew --dry-run
  ```

## Security Checklist

- [ ] Updated `.env.production` with correct domain
- [ ] Configured Supabase redirect URLs
- [ ] SSL certificate installed and auto-renewing
- [ ] Environment variables set on hosting platform
- [ ] CORS configured properly
- [ ] OAuth providers updated (if applicable)
- [ ] Tested all critical user flows

## Support

For hosting-specific issues, refer to:
- [Vercel Docs](https://vercel.com/docs)
- [Netlify Docs](https://docs.netlify.com)
- [Render Docs](https://render.com/docs)
- [DigitalOcean Docs](https://docs.digitalocean.com)
