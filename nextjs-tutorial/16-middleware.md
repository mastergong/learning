# บทที่ 16: Middleware

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Next.js Middleware และการทำงาน
- ทำความเข้าใจ Request/Response interception
- เรียนรู้ Authentication middleware
- เข้าใจ Performance monitoring และ Analytics

## 🔧 Middleware Fundamentals

### **⚡ Basic Middleware Setup**

```javascript
// middleware.js (in root directory)
import { NextResponse } from 'next/server';

// ✅ Basic middleware function
export function middleware(request) {
  console.log('Middleware executed for:', request.url);
  
  // ✅ Continue to the next middleware or page
  return NextResponse.next();
}

// ✅ Configure which paths middleware should run on
export const config = {
  matcher: [
    // Match all request paths except for the ones starting with:
    // - api (API routes)
    // - _next/static (static files)
    // - _next/image (image optimization files)
    // - favicon.ico (favicon file)
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};

// Alternative matcher configurations
export const config = {
  // Match specific paths
  matcher: ['/about/:path*', '/dashboard/:path*'],
  
  // Or use arrays for multiple patterns
  matcher: [
    '/admin/:path*',
    '/api/protected/:path*',
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

### **🔐 Authentication Middleware**

```javascript
// middleware.js - Complete authentication middleware
import { NextResponse } from 'next/server';
import { jwtVerify } from 'jose';

// ✅ Protected routes configuration
const protectedRoutes = [
  '/dashboard',
  '/profile',
  '/admin',
  '/api/protected'
];

const adminRoutes = [
  '/admin'
];

const publicRoutes = [
  '/',
  '/login',
  '/register',
  '/about',
  '/contact'
];

export async function middleware(request) {
  const { pathname } = request.nextUrl;
  const token = request.cookies.get('auth-token')?.value;

  // ✅ Check if route is protected
  const isProtectedRoute = protectedRoutes.some(route => 
    pathname.startsWith(route)
  );
  
  const isAdminRoute = adminRoutes.some(route => 
    pathname.startsWith(route)
  );

  const isPublicRoute = publicRoutes.some(route => 
    pathname === route || pathname.startsWith(route)
  );

  // ✅ Allow public routes
  if (isPublicRoute && !isProtectedRoute) {
    return NextResponse.next();
  }

  // ✅ Redirect to login if no token on protected route
  if (isProtectedRoute && !token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('callbackUrl', pathname);
    return NextResponse.redirect(loginUrl);
  }

  // ✅ Verify JWT token
  if (token) {
    try {
      const secret = new TextEncoder().encode(process.env.JWT_SECRET);
      const { payload } = await jwtVerify(token, secret);
      
      // ✅ Check admin access
      if (isAdminRoute && payload.role !== 'admin') {
        return NextResponse.redirect(new URL('/unauthorized', request.url));
      }

      // ✅ Add user info to request headers
      const response = NextResponse.next();
      response.headers.set('x-user-id', payload.userId);
      response.headers.set('x-user-role', payload.role);
      response.headers.set('x-user-email', payload.email);
      
      return response;
      
    } catch (error) {
      // ✅ Invalid token - redirect to login
      console.error('JWT verification failed:', error);
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('callbackUrl', pathname);
      
      const response = NextResponse.redirect(loginUrl);
      response.cookies.delete('auth-token');
      return response;
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};

// lib/auth-middleware.js - Helper functions
export function createAuthToken(payload) {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('24h')
    .sign(new TextEncoder().encode(process.env.JWT_SECRET));
}

export async function validateAuthToken(token) {
  try {
    const secret = new TextEncoder().encode(process.env.JWT_SECRET);
    const { payload } = await jwtVerify(token, secret);
    return { valid: true, payload };
  } catch (error) {
    return { valid: false, error: error.message };
  }
}

// api/login/route.js - Login API with token creation
import { NextResponse } from 'next/server';
import { createAuthToken } from '@/lib/auth-middleware';
import bcrypt from 'bcrypt';

export async function POST(request) {
  try {
    const { email, password } = await request.json();
    
    // ✅ Validate user credentials
    const user = await prisma.user.findUnique({
      where: { email },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        password: true
      }
    });

    if (!user || !await bcrypt.compare(password, user.password)) {
      return NextResponse.json(
        { error: 'Invalid credentials' },
        { status: 401 }
      );
    }

    // ✅ Create JWT token
    const token = await createAuthToken({
      userId: user.id,
      email: user.email,
      role: user.role,
      name: user.name
    });

    // ✅ Set token as HTTP-only cookie
    const response = NextResponse.json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role
      }
    });

    response.cookies.set('auth-token', token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 60 * 60 * 24 // 24 hours
    });

    return response;

  } catch (error) {
    console.error('Login error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// api/logout/route.js - Logout API
export async function POST() {
  const response = NextResponse.json({ message: 'Logged out successfully' });
  response.cookies.delete('auth-token');
  return response;
}
```

### **📊 Analytics and Monitoring Middleware**

```javascript
// middleware.js - Analytics middleware
import { NextResponse } from 'next/server';
import { headers } from 'next/headers';

export async function middleware(request) {
  const startTime = Date.now();
  const { pathname, search } = request.nextUrl;
  
  // ✅ Get request information
  const userAgent = request.headers.get('user-agent') || '';
  const referer = request.headers.get('referer') || '';
  const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown';
  const country = request.geo?.country || 'unknown';
  const city = request.geo?.city || 'unknown';

  // ✅ Create response
  const response = NextResponse.next();
  
  // ✅ Add custom headers
  response.headers.set('x-request-id', generateRequestId());
  response.headers.set('x-response-time', `${Date.now() - startTime}ms`);

  // ✅ Log analytics data (fire-and-forget)
  if (process.env.NODE_ENV === 'production') {
    // Don't await this - let it run in background
    trackPageView({
      pathname,
      search,
      userAgent,
      referer,
      ip,
      country,
      city,
      timestamp: new Date().toISOString(),
      requestId: response.headers.get('x-request-id')
    }).catch(error => {
      console.error('Analytics tracking failed:', error);
    });
  }

  return response;
}

// ✅ Analytics tracking function
async function trackPageView(data) {
  try {
    // Send to analytics service
    await fetch(process.env.ANALYTICS_ENDPOINT, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        event: 'page_view',
        ...data
      })
    });

    // Also store in database for internal analytics
    await prisma.analytics.create({
      data: {
        eventType: 'page_view',
        pathname: data.pathname,
        search: data.search,
        userAgent: data.userAgent,
        referer: data.referer,
        ip: data.ip,
        country: data.country,
        city: data.city,
        requestId: data.requestId
      }
    });
  } catch (error) {
    // Fail silently for analytics
    console.error('Analytics error:', error);
  }
}

function generateRequestId() {
  return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
}

// lib/analytics.js - Client-side analytics
'use client';

import { useEffect } from 'react';
import { usePathname, useSearchParams } from 'next/navigation';

export function useAnalytics() {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    // ✅ Track page views
    const url = pathname + (searchParams.toString() ? `?${searchParams.toString()}` : '');
    
    // Track with multiple services
    Promise.all([
      // Google Analytics
      window.gtag?.('config', process.env.NEXT_PUBLIC_GA_ID, {
        page_path: url,
      }),
      
      // Custom analytics
      fetch('/api/analytics/track', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          event: 'client_page_view',
          pathname,
          search: searchParams.toString(),
          timestamp: new Date().toISOString(),
          userAgent: navigator.userAgent,
          referrer: document.referrer
        })
      }).catch(console.error)
    ]);
  }, [pathname, searchParams]);

  // ✅ Track custom events
  const trackEvent = async (eventName, data = {}) => {
    try {
      await fetch('/api/analytics/track', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          event: eventName,
          ...data,
          pathname,
          timestamp: new Date().toISOString()
        })
      });
    } catch (error) {
      console.error('Event tracking failed:', error);
    }
  };

  return { trackEvent };
}

// app/layout.js - Add analytics to root layout
'use client';

import { useAnalytics } from '@/lib/analytics';
import { Suspense } from 'react';

function AnalyticsProvider({ children }) {
  useAnalytics();
  return children;
}

export default function RootLayout({ children }) {
  return (
    <html lang="th">
      <body>
        <Suspense fallback={null}>
          <AnalyticsProvider>
            {children}
          </AnalyticsProvider>
        </Suspense>
      </body>
    </html>
  );
}
```

## 🌐 Internationalization Middleware

### **🗺️ i18n Routing**

```javascript
// middleware.js - Internationalization middleware
import { NextResponse } from 'next/server';

const locales = ['en', 'th', 'ja', 'ko'];
const defaultLocale = 'en';

function getLocale(request) {
  // ✅ Check URL pathname for locale
  const pathname = request.nextUrl.pathname;
  const pathnameLocale = locales.find(locale => 
    pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameLocale) return pathnameLocale;

  // ✅ Check cookie
  const localeCookie = request.cookies.get('locale')?.value;
  if (locales.includes(localeCookie)) return localeCookie;

  // ✅ Check Accept-Language header
  const acceptLanguage = request.headers.get('accept-language') || '';
  const preferredLocales = acceptLanguage
    .split(',')
    .map(lang => lang.split(';')[0].trim().toLowerCase())
    .map(lang => {
      // Map language codes to our supported locales
      if (lang.startsWith('th')) return 'th';
      if (lang.startsWith('ja')) return 'ja';
      if (lang.startsWith('ko')) return 'ko';
      if (lang.startsWith('en')) return 'en';
      return null;
    })
    .filter(Boolean);

  return preferredLocales.find(locale => locales.includes(locale)) || defaultLocale;
}

export function middleware(request) {
  const { pathname } = request.nextUrl;
  
  // ✅ Skip middleware for certain paths
  if (
    pathname.startsWith('/_next') ||
    pathname.startsWith('/api') ||
    pathname.includes('/favicon.ico') ||
    pathname.includes('/robots.txt') ||
    pathname.includes('/sitemap.xml')
  ) {
    return NextResponse.next();
  }

  // ✅ Check if pathname already has a locale
  const pathnameHasLocale = locales.some(locale => 
    pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );

  if (!pathnameHasLocale) {
    // ✅ Redirect to URL with locale
    const locale = getLocale(request);
    const newUrl = new URL(`/${locale}${pathname}`, request.url);
    
    const response = NextResponse.redirect(newUrl);
    
    // ✅ Set locale cookie
    response.cookies.set('locale', locale, {
      maxAge: 60 * 60 * 24 * 365, // 1 year
      path: '/',
      sameSite: 'lax'
    });
    
    return response;
  }

  // ✅ Add locale to request headers for server components
  const locale = pathname.split('/')[1];
  const response = NextResponse.next();
  response.headers.set('x-locale', locale);
  
  return response;
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};

// lib/i18n.js - Internationalization utilities
const dictionaries = {
  en: () => import('@/dictionaries/en.json').then(module => module.default),
  th: () => import('@/dictionaries/th.json').then(module => module.default),
  ja: () => import('@/dictionaries/ja.json').then(module => module.default),
  ko: () => import('@/dictionaries/ko.json').then(module => module.default),
};

export const getDictionary = async (locale) => {
  if (!dictionaries[locale]) {
    return dictionaries[defaultLocale]();
  }
  return dictionaries[locale]();
};

export function getLocaleFromHeaders() {
  const headersList = headers();
  return headersList.get('x-locale') || defaultLocale;
}

// dictionaries/en.json
{
  "common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "retry": "Try again",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete",
    "edit": "Edit",
    "close": "Close"
  },
  "navigation": {
    "home": "Home",
    "about": "About",
    "contact": "Contact",
    "login": "Login",
    "logout": "Logout",
    "dashboard": "Dashboard",
    "profile": "Profile"
  },
  "auth": {
    "email": "Email",
    "password": "Password",
    "login": "Login",
    "register": "Register",
    "forgot_password": "Forgot Password?",
    "invalid_credentials": "Invalid email or password",
    "login_success": "Logged in successfully",
    "logout_success": "Logged out successfully"
  }
}

// dictionaries/th.json
{
  "common": {
    "loading": "กำลังโหลด...",
    "error": "เกิดข้อผิดพลาด",
    "retry": "ลองใหม่",
    "cancel": "ยกเลิก",
    "save": "บันทึก",
    "delete": "ลบ",
    "edit": "แก้ไข",
    "close": "ปิด"
  },
  "navigation": {
    "home": "หน้าแรก",
    "about": "เกี่ยวกับ",
    "contact": "ติดต่อ",
    "login": "เข้าสู่ระบบ",
    "logout": "ออกจากระบบ",
    "dashboard": "แดชบอร์ด",
    "profile": "โปรไฟล์"
  },
  "auth": {
    "email": "อีเมล",
    "password": "รหัสผ่าน",
    "login": "เข้าสู่ระบบ",
    "register": "สมัครสมาชิก",
    "forgot_password": "ลืมรหัสผ่าน?",
    "invalid_credentials": "อีเมลหรือรหัสผ่านไม่ถูกต้อง",
    "login_success": "เข้าสู่ระบบสำเร็จ",
    "logout_success": "ออกจากระบบสำเร็จ"
  }
}

// app/[locale]/layout.js - Locale-aware layout
import { getDictionary } from '@/lib/i18n';

export async function generateStaticParams() {
  return locales.map(locale => ({ locale }));
}

export default async function LocaleLayout({ children, params }) {
  const dict = await getDictionary(params.locale);
  
  return (
    <html lang={params.locale}>
      <body>
        <DictionaryProvider dictionary={dict}>
          <LocaleProvider locale={params.locale}>
            {children}
          </LocaleProvider>
        </DictionaryProvider>
      </body>
    </html>
  );
}

// components/LocaleProvider.js
'use client';

import { createContext, useContext } from 'react';

const LocaleContext = createContext();
const DictionaryContext = createContext();

export function LocaleProvider({ children, locale }) {
  return (
    <LocaleContext.Provider value={locale}>
      {children}
    </LocaleContext.Provider>
  );
}

export function DictionaryProvider({ children, dictionary }) {
  return (
    <DictionaryContext.Provider value={dictionary}>
      {children}
    </DictionaryContext.Provider>
  );
}

export function useLocale() {
  const locale = useContext(LocaleContext);
  if (!locale) throw new Error('useLocale must be used within LocaleProvider');
  return locale;
}

export function useDictionary() {
  const dict = useContext(DictionaryContext);
  if (!dict) throw new Error('useDictionary must be used within DictionaryProvider');
  return dict;
}

// components/LanguageSwitcher.js
'use client';

import { useLocale } from './LocaleProvider';
import { useRouter, usePathname } from 'next/navigation';

const languages = [
  { code: 'en', name: 'English', flag: '🇺🇸' },
  { code: 'th', name: 'ไทย', flag: '🇹🇭' },
  { code: 'ja', name: '日本語', flag: '🇯🇵' },
  { code: 'ko', name: '한국어', flag: '🇰🇷' }
];

export function LanguageSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();

  const switchLanguage = (newLocale) => {
    // Remove current locale from pathname
    const currentLocaleRegex = new RegExp(`^/${locale}`);
    const pathnameWithoutLocale = pathname.replace(currentLocaleRegex, '');
    
    // Add new locale
    const newPathname = `/${newLocale}${pathnameWithoutLocale}`;
    
    // Set locale cookie and navigate
    document.cookie = `locale=${newLocale}; path=/; max-age=${60 * 60 * 24 * 365}`;
    router.push(newPathname);
  };

  return (
    <div className="relative group">
      <button className="flex items-center space-x-2 px-3 py-2 rounded-md hover:bg-gray-100">
        <span>{languages.find(lang => lang.code === locale)?.flag}</span>
        <span>{languages.find(lang => lang.code === locale)?.name}</span>
        <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 9l-7 7-7-7" />
        </svg>
      </button>
      
      <div className="absolute right-0 mt-1 w-48 bg-white rounded-md shadow-lg opacity-0 invisible group-hover:opacity-100 group-hover:visible transition-all duration-200 z-50">
        <div className="py-1">
          {languages.map(language => (
            <button
              key={language.code}
              onClick={() => switchLanguage(language.code)}
              className={`w-full text-left px-4 py-2 text-sm hover:bg-gray-100 flex items-center space-x-2 ${
                locale === language.code ? 'bg-blue-50 text-blue-600' : 'text-gray-700'
              }`}
            >
              <span>{language.flag}</span>
              <span>{language.name}</span>
            </button>
          ))}
        </div>
      </div>
    </div>
  );
}
```

## ⚡ Performance Optimization

### **🚀 Caching and Optimization Middleware**

```javascript
// middleware.js - Performance optimization middleware
import { NextResponse } from 'next/server';

export function middleware(request) {
  const { pathname } = request.nextUrl;
  const response = NextResponse.next();

  // ✅ Add security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');

  // ✅ Add cache headers for static assets
  if (pathname.startsWith('/_next/static/') || pathname.includes('.')) {
    response.headers.set('Cache-Control', 'public, max-age=31536000, immutable');
  }

  // ✅ Add cache headers for API routes
  if (pathname.startsWith('/api/')) {
    // Don't cache API routes by default
    response.headers.set('Cache-Control', 'no-store, max-age=0');
  }

  // ✅ Add CORS headers for API routes
  if (pathname.startsWith('/api/')) {
    response.headers.set('Access-Control-Allow-Origin', process.env.ALLOWED_ORIGINS || '*');
    response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  }

  // ✅ Handle preflight requests
  if (request.method === 'OPTIONS') {
    return new Response(null, { status: 200, headers: response.headers });
  }

  // ✅ Compress responses (if not already handled by CDN)
  if (process.env.ENABLE_COMPRESSION === 'true') {
    const acceptEncoding = request.headers.get('accept-encoding') || '';
    if (acceptEncoding.includes('gzip')) {
      response.headers.set('Content-Encoding', 'gzip');
    }
  }

  // ✅ Add response time header
  const requestStartTime = Date.now();
  return new Promise((resolve) => {
    setTimeout(() => {
      const responseTime = Date.now() - requestStartTime;
      response.headers.set('X-Response-Time', `${responseTime}ms`);
      resolve(response);
    }, 0);
  });
}

// middleware.js - Rate limiting middleware
import { NextResponse } from 'next/server';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN,
});

const rateLimitConfig = {
  '/api/auth/login': { requests: 5, window: 900 }, // 5 requests per 15 minutes
  '/api/auth/register': { requests: 3, window: 3600 }, // 3 requests per hour
  '/api/contact': { requests: 3, window: 3600 }, // 3 requests per hour
  '/api/newsletter': { requests: 1, window: 86400 }, // 1 request per day
  default: { requests: 100, window: 3600 } // 100 requests per hour
};

async function isRateLimited(ip, pathname) {
  const config = rateLimitConfig[pathname] || rateLimitConfig.default;
  const key = `rate_limit:${ip}:${pathname}`;
  
  try {
    const current = await redis.get(key);
    
    if (current === null) {
      // First request
      await redis.setex(key, config.window, 1);
      return { limited: false, count: 1, remaining: config.requests - 1 };
    }
    
    if (current >= config.requests) {
      // Rate limit exceeded
      const ttl = await redis.ttl(key);
      return { 
        limited: true, 
        count: current, 
        remaining: 0,
        resetTime: Date.now() + (ttl * 1000)
      };
    }
    
    // Increment counter
    const newCount = await redis.incr(key);
    return { 
      limited: false, 
      count: newCount, 
      remaining: config.requests - newCount 
    };
    
  } catch (error) {
    console.error('Rate limiting error:', error);
    // Allow request if Redis is down
    return { limited: false, count: 0, remaining: 100 };
  }
}

export async function middleware(request) {
  const { pathname } = request.nextUrl;
  const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown';

  // ✅ Apply rate limiting to API routes
  if (pathname.startsWith('/api/')) {
    const rateLimitResult = await isRateLimited(ip, pathname);
    
    if (rateLimitResult.limited) {
      return NextResponse.json(
        { 
          error: 'Too many requests', 
          resetTime: rateLimitResult.resetTime 
        },
        { 
          status: 429,
          headers: {
            'X-RateLimit-Limit': String(rateLimitConfig[pathname]?.requests || rateLimitConfig.default.requests),
            'X-RateLimit-Remaining': '0',
            'X-RateLimit-Reset': String(Math.floor(rateLimitResult.resetTime / 1000)),
            'Retry-After': String(Math.floor((rateLimitResult.resetTime - Date.now()) / 1000))
          }
        }
      );
    }

    // ✅ Add rate limit headers to successful responses
    const response = NextResponse.next();
    response.headers.set('X-RateLimit-Limit', String(rateLimitConfig[pathname]?.requests || rateLimitConfig.default.requests));
    response.headers.set('X-RateLimit-Remaining', String(rateLimitResult.remaining));
    
    return response;
  }

  return NextResponse.next();
}

export const config = {
  matcher: [
    '/api/:path*',
    '/((?!_next/static|_next/image|favicon.ico).*)',
  ],
};
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Authentication Middleware**
1. สร้าง JWT-based authentication middleware
2. เพิ่ม role-based access control
3. ทดสอบ protected routes
4. เพิ่ม token refresh mechanism

### **🎯 แบบฝึกหัดที่ 2: Analytics Middleware**
1. สร้าง analytics tracking middleware
2. เก็บ user behavior data
3. สร้าง analytics dashboard
4. เพิ่ม real-time monitoring

### **🎯 แบบฝึกหัดที่ 3: i18n Middleware**
1. เพิ่ม internationalization support
2. สร้าง language detection logic
3. ทดสอบ locale switching
4. เพิ่ม SEO-friendly URLs

### **🎯 แบบฝึกหัดที่ 4: Performance Middleware**
1. เพิ่ม rate limiting
2. สร้าง caching strategies
3. เพิ่ม security headers
4. ทดสอบ performance impact

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 15: Streaming and Suspense](./15-streaming-suspense.md)

**บทต่อไป**: [บทที่ 17: Internationalization (i18n)](./17-internationalization.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
- [JWT Authentication](https://jwt.io/)
- [Rate Limiting](https://nextjs.org/docs/app/building-your-application/routing/middleware#rate-limiting)
- [Internationalization](https://nextjs.org/docs/app/building-your-application/routing/internationalization)
