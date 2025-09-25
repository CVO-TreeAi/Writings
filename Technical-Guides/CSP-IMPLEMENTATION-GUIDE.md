# CSP Fix Implementation Guide for TreeShop

## Problem Summary
Your Content Security Policy is blocking:
1. Google Tag Manager resources
2. Google Ads conversion tracking
3. Font loading (data: URIs)
4. Frame embedding for GTM

## Solution

### Option 1: Using Next.js Middleware (Recommended)

Create or update `middleware.ts` in your project root:

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  const cspHeader = `
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval'
      *.googletagmanager.com
      *.google-analytics.com
      *.youtube.com
      *.stripe.com
      *.googleapis.com
      maps.googleapis.com
      *.doubleclick.net
      *.googlesyndication.com
      *.googleadservices.com;
    style-src 'self' 'unsafe-inline'
      fonts.googleapis.com
      *.googletagmanager.com;
    font-src 'self'
      fonts.gstatic.com
      data:;
    img-src 'self' data: blob:
      *.googletagmanager.com
      *.google-analytics.com
      *.doubleclick.net
      *.google.com
      *.googleapis.com;
    connect-src 'self'
      *.google-analytics.com
      *.analytics.google.com
      *.googletagmanager.com
      *.doubleclick.net
      *.google.com;
    frame-src 'self'
      *.youtube.com
      *.youtube-nocookie.com
      *.googletagmanager.com
      *.doubleclick.net;
    frame-ancestors 'self';
    base-uri 'self';
    form-action 'self';
  `.replace(/\s{2,}/g, ' ').trim();

  response.headers.set('Content-Security-Policy', cspHeader);

  return response;
}

export const config = {
  matcher: '/((?!api|_next/static|_next/image|favicon.ico).*)',
};
```

### Option 2: Using next.config.js

Update your `next.config.js`:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: `
              default-src 'self';
              script-src 'self' 'unsafe-inline' 'unsafe-eval' *.googletagmanager.com *.google-analytics.com *.youtube.com *.stripe.com *.googleapis.com maps.googleapis.com *.doubleclick.net *.googlesyndication.com *.googleadservices.com;
              style-src 'self' 'unsafe-inline' fonts.googleapis.com *.googletagmanager.com;
              font-src 'self' fonts.gstatic.com data:;
              img-src 'self' data: blob: *.googletagmanager.com *.google-analytics.com *.doubleclick.net *.google.com *.googleapis.com;
              connect-src 'self' *.google-analytics.com *.analytics.google.com *.googletagmanager.com *.doubleclick.net *.google.com;
              frame-src 'self' *.youtube.com *.youtube-nocookie.com *.googletagmanager.com *.doubleclick.net;
            `.replace(/\s{2,}/g, ' ').trim()
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## Key Changes Made

### 1. Font Sources (`font-src`)
- Added `data:` to allow inline font data URIs
- Kept `fonts.gstatic.com` for Google Fonts

### 2. Script Sources (`script-src`)
- Added `*.doubleclick.net` for Google Ads
- Added `*.googlesyndication.com` for AdSense/Ads
- Added `*.googleadservices.com` for conversion tracking

### 3. Style Sources (`style-src`)
- Added `*.googletagmanager.com` for GTM debug styles

### 4. Frame Sources (`frame-src`)
- Added `*.googletagmanager.com` for GTM debug panel
- Added `*.doubleclick.net` for ad frames

## Testing

1. Deploy the changes
2. Open Chrome DevTools
3. Check Console for CSP violations
4. Test Google Tag Manager Debug Mode
5. Verify conversion tracking fires

## Security Notes

- `unsafe-inline` and `unsafe-eval` are required for Google Tag Manager
- Consider using nonces for better security in production
- Regularly audit and remove unused domains

## Rollback

If issues occur, remove the CSP header entirely as a temporary measure while debugging.