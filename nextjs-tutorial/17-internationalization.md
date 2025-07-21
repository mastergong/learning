# บทที่ 17: Internationalization (i18n)

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Internationalization ใน Next.js
- ทำความเข้าใจ Multi-language support
- เรียนรู้ Locale routing และ Content management
- เข้าใจ SEO optimization สำหรับหลายภาษา

## 🌍 i18n Fundamentals

### **⚡ App Router i18n Setup**

```javascript
// lib/i18n/config.js
export const i18n = {
  defaultLocale: 'en',
  locales: ['en', 'th', 'ja', 'ko', 'zh'],
  localeDetection: true,
  localePath: './locales',
  namespaces: ['common', 'navigation', 'auth', 'products', 'blog'],
  defaultNamespace: 'common'
};

export const localeNames = {
  en: 'English',
  th: 'ไทย',
  ja: '日本語',
  ko: '한국어',
  zh: '中文'
};

export const localeFlags = {
  en: '🇺🇸',
  th: '🇹🇭',
  ja: '🇯🇵',
  ko: '🇰🇷',
  zh: '🇨🇳'
};

// Direction for RTL languages
export const localeDirections = {
  en: 'ltr',
  th: 'ltr',
  ja: 'ltr',
  ko: 'ltr',
  zh: 'ltr',
  ar: 'rtl', // If you add Arabic later
  he: 'rtl'  // If you add Hebrew later
};

// middleware.js - Enhanced i18n middleware
import { NextResponse } from 'next/server';
import { i18n } from './lib/i18n/config';

function getLocale(request) {
  const { pathname } = request.nextUrl;
  
  // ✅ Check if pathname already has locale
  const pathnameLocale = i18n.locales.find(locale => 
    pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameLocale) return pathnameLocale;

  // ✅ Check locale preference cookie
  const localeCookie = request.cookies.get('locale')?.value;
  if (i18n.locales.includes(localeCookie)) {
    return localeCookie;
  }

  // ✅ Parse Accept-Language header
  const acceptLanguage = request.headers.get('accept-language') || '';
  const preferredLocales = acceptLanguage
    .split(',')
    .map(lang => {
      const [code, quality = '1'] = lang.trim().split(';q=');
      return { code: code.toLowerCase().split('-')[0], quality: parseFloat(quality) };
    })
    .sort((a, b) => b.quality - a.quality)
    .map(({ code }) => code);

  // ✅ Find matching locale
  for (const preferred of preferredLocales) {
    const matchedLocale = i18n.locales.find(locale => 
      locale.toLowerCase().startsWith(preferred) || preferred.startsWith(locale.toLowerCase())
    );
    if (matchedLocale) return matchedLocale;
  }

  return i18n.defaultLocale;
}

export function middleware(request) {
  const { pathname } = request.nextUrl;

  // ✅ Skip middleware for certain paths
  const skipPaths = [
    '/_next',
    '/api',
    '/favicon.ico',
    '/robots.txt',
    '/sitemap.xml',
    '/manifest.json'
  ];

  if (skipPaths.some(path => pathname.startsWith(path))) {
    return NextResponse.next();
  }

  // ✅ Check if pathname has locale
  const pathnameHasLocale = i18n.locales.some(locale => 
    pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );

  if (!pathnameHasLocale) {
    const locale = getLocale(request);
    
    // ✅ Redirect to URL with locale
    const redirectUrl = new URL(`/${locale}${pathname}${request.nextUrl.search}`, request.url);
    const response = NextResponse.redirect(redirectUrl);
    
    // ✅ Set locale cookie for future requests
    response.cookies.set('locale', locale, {
      maxAge: 60 * 60 * 24 * 365, // 1 year
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      path: '/'
    });
    
    return response;
  }

  // ✅ Extract locale and pass to pages
  const locale = pathname.split('/')[1];
  const response = NextResponse.next();
  response.headers.set('x-locale', locale);
  
  return response;
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico|robots.txt|sitemap.xml|manifest.json).*)',
  ],
};
```

### **📖 Dictionary Management**

```javascript
// lib/i18n/dictionaries.js
const dictionaries = {
  en: () => import('./dictionaries/en.json').then(m => m.default),
  th: () => import('./dictionaries/th.json').then(m => m.default),
  ja: () => import('./dictionaries/ja.json').then(m => m.default),
  ko: () => import('./dictionaries/ko.json').then(m => m.default),
  zh: () => import('./dictionaries/zh.json').then(m => m.default),
};

export const getDictionary = async (locale) => {
  if (!dictionaries[locale]) {
    console.warn(`Dictionary for locale "${locale}" not found, falling back to default`);
    return dictionaries[i18n.defaultLocale]();
  }
  return dictionaries[locale]();
};

// lib/i18n/utils.js
import { headers } from 'next/headers';
import { i18n } from './config';

export function getLocaleFromHeaders() {
  const headersList = headers();
  return headersList.get('x-locale') || i18n.defaultLocale;
}

export function interpolate(template, values = {}) {
  return template.replace(/\{(\w+)\}/g, (match, key) => {
    return values[key] !== undefined ? values[key] : match;
  });
}

export function formatCurrency(amount, locale, currency = 'USD') {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency,
    minimumFractionDigits: 2
  }).format(amount);
}

export function formatDate(date, locale, options = {}) {
  const defaultOptions = {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    ...options
  };
  
  return new Intl.DateTimeFormat(locale, defaultOptions).format(new Date(date));
}

export function formatNumber(number, locale, options = {}) {
  return new Intl.NumberFormat(locale, options).format(number);
}

export function getDirection(locale) {
  return localeDirections[locale] || 'ltr';
}

// lib/i18n/dictionaries/en.json
{
  "common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "retry": "Try again",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete",
    "edit": "Edit",
    "close": "Close",
    "search": "Search",
    "submit": "Submit",
    "back": "Back",
    "next": "Next",
    "previous": "Previous",
    "page_of": "Page {current} of {total}",
    "no_results": "No results found",
    "select_option": "Select an option",
    "required_field": "This field is required",
    "invalid_email": "Please enter a valid email address"
  },
  "navigation": {
    "home": "Home",
    "about": "About",
    "products": "Products",
    "blog": "Blog",
    "contact": "Contact",
    "login": "Login",
    "logout": "Logout",
    "register": "Register",
    "dashboard": "Dashboard",
    "profile": "Profile",
    "settings": "Settings",
    "help": "Help"
  },
  "auth": {
    "email": "Email",
    "password": "Password",
    "confirm_password": "Confirm Password",
    "login_title": "Login to your account",
    "register_title": "Create new account",
    "forgot_password": "Forgot Password?",
    "remember_me": "Remember me",
    "invalid_credentials": "Invalid email or password",
    "login_success": "Logged in successfully",
    "logout_success": "Logged out successfully",
    "register_success": "Account created successfully",
    "password_mismatch": "Passwords do not match",
    "weak_password": "Password must be at least 8 characters long"
  },
  "products": {
    "title": "Products",
    "category": "Category",
    "price": "Price",
    "add_to_cart": "Add to Cart",
    "out_of_stock": "Out of Stock",
    "in_stock": "In Stock",
    "product_details": "Product Details",
    "specifications": "Specifications",
    "reviews": "Reviews",
    "related_products": "Related Products",
    "sort_by": "Sort by",
    "filter_by": "Filter by",
    "price_range": "Price Range",
    "brand": "Brand",
    "rating": "Rating",
    "availability": "Availability"
  },
  "blog": {
    "title": "Blog",
    "read_more": "Read more",
    "published_on": "Published on {date}",
    "by_author": "by {author}",
    "categories": "Categories",
    "tags": "Tags",
    "comments": "Comments",
    "leave_comment": "Leave a comment",
    "recent_posts": "Recent Posts",
    "popular_posts": "Popular Posts",
    "search_posts": "Search posts..."
  },
  "contact": {
    "title": "Contact Us",
    "name": "Name",
    "subject": "Subject",
    "message": "Message",
    "send_message": "Send Message",
    "success_message": "Your message has been sent successfully!",
    "error_message": "Failed to send message. Please try again.",
    "office_hours": "Office Hours",
    "address": "Address",
    "phone": "Phone",
    "email": "Email"
  }
}

// lib/i18n/dictionaries/th.json
{
  "common": {
    "loading": "กำลังโหลด...",
    "error": "เกิดข้อผิดพลาด",
    "retry": "ลองใหม่",
    "cancel": "ยกเลิก",
    "save": "บันทึก",
    "delete": "ลบ",
    "edit": "แก้ไข",
    "close": "ปิด",
    "search": "ค้นหา",
    "submit": "ส่ง",
    "back": "กลับ",
    "next": "ถัดไป",
    "previous": "ก่อนหน้า",
    "page_of": "หน้า {current} จาก {total}",
    "no_results": "ไม่พบผลลัพธ์",
    "select_option": "เลือกตัวเลือก",
    "required_field": "กรุณากรอกข้อมูลในช่องนี้",
    "invalid_email": "กรุณากรอกอีเมลที่ถูกต้อง"
  },
  "navigation": {
    "home": "หน้าแรก",
    "about": "เกี่ยวกับ",
    "products": "สินค้า",
    "blog": "บล็อก",
    "contact": "ติดต่อ",
    "login": "เข้าสู่ระบบ",
    "logout": "ออกจากระบบ",
    "register": "สมัครสมาชิก",
    "dashboard": "แดชบอร์ด",
    "profile": "โปรไฟล์",
    "settings": "ตั้งค่า",
    "help": "ช่วยเหลือ"
  },
  "auth": {
    "email": "อีเมล",
    "password": "รหัสผ่าน",
    "confirm_password": "ยืนยันรหัสผ่าน",
    "login_title": "เข้าสู่ระบบ",
    "register_title": "สร้างบัญชีใหม่",
    "forgot_password": "ลืมรหัสผ่าน?",
    "remember_me": "จดจำฉัน",
    "invalid_credentials": "อีเมลหรือรหัสผ่านไม่ถูกต้อง",
    "login_success": "เข้าสู่ระบบสำเร็จ",
    "logout_success": "ออกจากระบบสำเร็จ",
    "register_success": "สร้างบัญชีสำเร็จ",
    "password_mismatch": "รหัสผ่านไม่ตรงกัน",
    "weak_password": "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร"
  },
  "products": {
    "title": "สินค้า",
    "category": "หมวดหมู่",
    "price": "ราคา",
    "add_to_cart": "เพิ่มลงตะกร้า",
    "out_of_stock": "สินค้าหมด",
    "in_stock": "มีสินค้า",
    "product_details": "รายละเอียดสินค้า",
    "specifications": "ข้อมูลจำเพาะ",
    "reviews": "รีวิว",
    "related_products": "สินค้าที่เกี่ยวข้อง",
    "sort_by": "เรียงตาม",
    "filter_by": "กรองตาม",
    "price_range": "ช่วงราคา",
    "brand": "แบรนด์",
    "rating": "คะแนน",
    "availability": "สถานะสินค้า"
  },
  "blog": {
    "title": "บล็อก",
    "read_more": "อ่านต่อ",
    "published_on": "เผยแพร่เมื่อ {date}",
    "by_author": "โดย {author}",
    "categories": "หมวดหมู่",
    "tags": "แท็ก",
    "comments": "ความคิดเห็น",
    "leave_comment": "แสดงความคิดเห็น",
    "recent_posts": "โพสต์ล่าสุด",
    "popular_posts": "โพสต์ยอดนิยม",
    "search_posts": "ค้นหาโพสต์..."
  },
  "contact": {
    "title": "ติดต่อเรา",
    "name": "ชื่อ",
    "subject": "หัวข้อ",
    "message": "ข้อความ",
    "send_message": "ส่งข้อความ",
    "success_message": "ส่งข้อความเรียบร้อยแล้ว!",
    "error_message": "ส่งข้อความไม่สำเร็จ กรุณาลองใหม่",
    "office_hours": "เวลาทำการ",
    "address": "ที่อยู่",
    "phone": "โทรศัพท์",
    "email": "อีเมล"
  }
}
```

## 🎨 React Components for i18n

### **🔄 Language Provider & Hooks**

```jsx
// app/[locale]/layout.js - Root layout with i18n
import { getDictionary } from '@/lib/i18n/dictionaries';
import { getDirection } from '@/lib/i18n/utils';
import { i18n } from '@/lib/i18n/config';
import { LanguageProvider } from '@/components/providers/LanguageProvider';
import { LanguageSwitcher } from '@/components/LanguageSwitcher';

export async function generateStaticParams() {
  return i18n.locales.map(locale => ({ locale }));
}

export async function generateMetadata({ params }) {
  const dict = await getDictionary(params.locale);
  
  return {
    title: dict.navigation.home,
    description: dict.common.description || 'Next.js i18n Application',
    alternates: {
      canonical: `/${params.locale}`,
      languages: Object.fromEntries(
        i18n.locales.map(locale => [locale, `/${locale}`])
      )
    }
  };
}

export default async function LocaleLayout({ children, params }) {
  const dict = await getDictionary(params.locale);
  const direction = getDirection(params.locale);
  
  return (
    <html lang={params.locale} dir={direction}>
      <head>
        {/* Preload alternate language versions */}
        {i18n.locales.map(locale => (
          <link
            key={locale}
            rel="alternate"
            hrefLang={locale}
            href={`/${locale}`}
          />
        ))}
        <link rel="alternate" hrefLang="x-default" href={`/${i18n.defaultLocale}`} />
      </head>
      <body className={direction === 'rtl' ? 'rtl' : 'ltr'}>
        <LanguageProvider locale={params.locale} dictionary={dict}>
          <div className="min-h-screen bg-gray-50">
            <header className="bg-white shadow-sm">
              <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div className="flex justify-between items-center h-16">
                  <div className="flex items-center space-x-4">
                    <h1 className="text-xl font-bold">{dict.navigation.home}</h1>
                  </div>
                  <LanguageSwitcher />
                </div>
              </div>
            </header>
            
            <main>
              {children}
            </main>
          </div>
        </LanguageProvider>
      </body>
    </html>
  );
}

// components/providers/LanguageProvider.js
'use client';

import { createContext, useContext } from 'react';
import { interpolate, formatCurrency, formatDate, formatNumber } from '@/lib/i18n/utils';

const LanguageContext = createContext();

export function LanguageProvider({ children, locale, dictionary }) {
  const t = (key, values = {}) => {
    const keys = key.split('.');
    let value = dictionary;
    
    for (const k of keys) {
      value = value?.[k];
      if (value === undefined) break;
    }
    
    if (typeof value === 'string' && values && Object.keys(values).length > 0) {
      return interpolate(value, values);
    }
    
    return value || key;
  };

  const formatters = {
    currency: (amount, currency = 'USD') => formatCurrency(amount, locale, currency),
    date: (date, options = {}) => formatDate(date, locale, options),
    number: (number, options = {}) => formatNumber(number, locale, options)
  };

  const value = {
    locale,
    dictionary,
    t,
    ...formatters
  };

  return (
    <LanguageContext.Provider value={value}>
      {children}
    </LanguageContext.Provider>
  );
}

export function useLanguage() {
  const context = useContext(LanguageContext);
  if (!context) {
    throw new Error('useLanguage must be used within LanguageProvider');
  }
  return context;
}

// Convenience hook for translation
export function useTranslation() {
  const { t } = useLanguage();
  return t;
}

// components/LanguageSwitcher.js
'use client';

import { useState } from 'react';
import { usePathname, useRouter } from 'next/navigation';
import { useLanguage } from '@/components/providers/LanguageProvider';
import { i18n, localeNames, localeFlags } from '@/lib/i18n/config';

export function LanguageSwitcher() {
  const [isOpen, setIsOpen] = useState(false);
  const { locale } = useLanguage();
  const pathname = usePathname();
  const router = useRouter();

  const switchLanguage = async (newLocale) => {
    if (newLocale === locale) return;

    // ✅ Remove current locale from pathname
    const segments = pathname.split('/');
    if (i18n.locales.includes(segments[1])) {
      segments[1] = newLocale;
    } else {
      segments.unshift('', newLocale);
    }
    
    const newPath = segments.join('/');
    
    // ✅ Set locale cookie
    document.cookie = `locale=${newLocale}; path=/; max-age=${60 * 60 * 24 * 365}; SameSite=Lax`;
    
    // ✅ Navigate to new locale
    router.push(newPath);
    setIsOpen(false);
  };

  const currentLanguage = {
    code: locale,
    name: localeNames[locale],
    flag: localeFlags[locale]
  };

  return (
    <div className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="flex items-center space-x-2 px-3 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-md shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        aria-expanded={isOpen}
        aria-haspopup="true"
      >
        <span className="text-lg">{currentLanguage.flag}</span>
        <span>{currentLanguage.name}</span>
        <svg
          className={`w-4 h-4 transition-transform ${isOpen ? 'rotate-180' : ''}`}
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 9l-7 7-7-7" />
        </svg>
      </button>

      {isOpen && (
        <>
          {/* Backdrop */}
          <div
            className="fixed inset-0 z-10"
            onClick={() => setIsOpen(false)}
          />
          
          {/* Dropdown */}
          <div className="absolute right-0 z-20 mt-2 w-48 bg-white border border-gray-200 rounded-md shadow-lg">
            <div className="py-1">
              {i18n.locales.map(localeCode => (
                <button
                  key={localeCode}
                  onClick={() => switchLanguage(localeCode)}
                  className={`w-full text-left px-4 py-2 text-sm flex items-center space-x-3 hover:bg-gray-100 transition-colors ${
                    locale === localeCode 
                      ? 'bg-blue-50 text-blue-700' 
                      : 'text-gray-700'
                  }`}
                >
                  <span className="text-lg">{localeFlags[localeCode]}</span>
                  <span className="font-medium">{localeNames[localeCode]}</span>
                  {locale === localeCode && (
                    <svg className="w-4 h-4 ml-auto text-blue-600" fill="currentColor" viewBox="0 0 20 20">
                      <path fillRule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clipRule="evenodd" />
                    </svg>
                  )}
                </button>
              ))}
            </div>
          </div>
        </>
      )}
    </div>
  );
}

// components/LocalizedLink.js
'use client';

import Link from 'next/link';
import { useLanguage } from '@/components/providers/LanguageProvider';

export function LocalizedLink({ href, children, ...props }) {
  const { locale } = useLanguage();
  
  // ✅ Ensure href starts with locale
  const localizedHref = href.startsWith('/') 
    ? `/${locale}${href}` 
    : href;

  return (
    <Link href={localizedHref} {...props}>
      {children}
    </Link>
  );
}

// components/LocalizedImage.js
'use client';

import Image from 'next/image';
import { useLanguage } from '@/components/providers/LanguageProvider';

export function LocalizedImage({ src, alt, ...props }) {
  const { locale } = useLanguage();
  
  // ✅ Try to load localized image first
  const localizedSrc = src.replace(/(\.[^.]+)$/, `_${locale}$1`);
  
  return (
    <Image
      src={localizedSrc}
      alt={alt}
      onError={(e) => {
        // ✅ Fallback to default image if localized version doesn't exist
        e.target.src = src;
      }}
      {...props}
    />
  );
}
```

## 📱 Content Management & SEO

### **🔍 SEO Optimization for Multiple Languages**

```jsx
// app/[locale]/blog/[slug]/page.js - SEO-optimized blog post
import { getDictionary } from '@/lib/i18n/dictionaries';
import { getLocaleFromHeaders } from '@/lib/i18n/utils';
import { i18n } from '@/lib/i18n/config';
import { LocalizedLink } from '@/components/LocalizedLink';

async function getBlogPost(slug, locale) {
  // ✅ Get post in specific language
  const post = await prisma.blogPost.findFirst({
    where: {
      slug,
      locale,
      published: true
    },
    include: {
      author: true,
      categories: true,
      tags: true,
      // Get translations of this post
      translations: {
        where: { published: true },
        select: { locale: true, slug: true }
      }
    }
  });

  return post;
}

export async function generateMetadata({ params }) {
  const { locale, slug } = params;
  const post = await getBlogPost(slug, locale);
  
  if (!post) {
    return {
      title: 'Post Not Found',
      description: 'The requested blog post was not found.'
    };
  }

  return {
    title: post.title,
    description: post.excerpt,
    keywords: post.tags?.map(tag => tag.name).join(', '),
    authors: [{ name: post.author.name }],
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      images: [
        {
          url: post.featuredImage,
          width: 1200,
          height: 630,
          alt: post.title
        }
      ],
      locale: locale,
      alternateLocale: i18n.locales.filter(l => l !== locale)
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.featuredImage]
    },
    alternates: {
      canonical: `/${locale}/blog/${slug}`,
      languages: Object.fromEntries(
        post.translations.map(trans => [trans.locale, `/${trans.locale}/blog/${trans.slug}`])
      )
    }
  };
}

export async function generateStaticParams() {
  const posts = await prisma.blogPost.findMany({
    where: { published: true },
    select: { slug: true, locale: true }
  });

  return posts.map(post => ({
    locale: post.locale,
    slug: post.slug
  }));
}

export default async function BlogPostPage({ params }) {
  const { locale, slug } = params;
  const [post, dict] = await Promise.all([
    getBlogPost(slug, locale),
    getDictionary(locale)
  ]);

  if (!post) {
    return (
      <div className="max-w-4xl mx-auto px-4 py-16 text-center">
        <h1 className="text-2xl font-bold text-gray-900 mb-4">
          {dict.common.error}
        </h1>
        <p className="text-gray-600 mb-8">
          {dict.blog.post_not_found || 'Blog post not found'}
        </p>
        <LocalizedLink 
          href="/blog"
          className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700"
        >
          {dict.common.back}
        </LocalizedLink>
      </div>
    );
  }

  return (
    <article className="max-w-4xl mx-auto px-4 py-8">
      {/* Language switcher for this post */}
      {post.translations.length > 0 && (
        <div className="mb-6 p-4 bg-blue-50 border border-blue-200 rounded-lg">
          <p className="text-sm text-blue-800 mb-2">
            {dict.blog.available_languages || 'This post is available in other languages:'}
          </p>
          <div className="flex flex-wrap gap-2">
            {post.translations.map(translation => (
              <LocalizedLink
                key={translation.locale}
                href={`/blog/${translation.slug}`}
                className="inline-flex items-center px-3 py-1 text-sm bg-blue-100 text-blue-800 rounded-full hover:bg-blue-200"
              >
                <span className="mr-1">{localeFlags[translation.locale]}</span>
                {localeNames[translation.locale]}
              </LocalizedLink>
            ))}
          </div>
        </div>
      )}

      {/* Post header */}
      <header className="mb-8">
        <div className="flex flex-wrap gap-2 mb-4">
          {post.categories?.map(category => (
            <span
              key={category.id}
              className="px-3 py-1 text-sm bg-blue-100 text-blue-800 rounded-full"
            >
              {category.name}
            </span>
          ))}
        </div>
        
        <h1 className="text-4xl font-bold text-gray-900 mb-4">
          {post.title}
        </h1>
        
        <div className="flex items-center text-gray-600 text-sm">
          <span>{dict.blog.by_author.replace('{author}', post.author.name)}</span>
          <span className="mx-2">•</span>
          <time dateTime={post.publishedAt}>
            {dict.blog.published_on.replace('{date}', formatDate(post.publishedAt, locale))}
          </time>
        </div>
      </header>

      {/* Featured image */}
      {post.featuredImage && (
        <div className="mb-8">
          <img
            src={post.featuredImage}
            alt={post.title}
            className="w-full h-64 md:h-96 object-cover rounded-lg"
          />
        </div>
      )}

      {/* Post content */}
      <div 
        className="prose prose-lg max-w-none"
        dangerouslySetInnerHTML={{ __html: post.content }}
      />

      {/* Tags */}
      {post.tags && post.tags.length > 0 && (
        <div className="mt-8 pt-8 border-t border-gray-200">
          <h3 className="text-lg font-semibold text-gray-900 mb-3">
            {dict.blog.tags}
          </h3>
          <div className="flex flex-wrap gap-2">
            {post.tags.map(tag => (
              <LocalizedLink
                key={tag.id}
                href={`/blog/tag/${tag.slug}`}
                className="px-3 py-1 text-sm bg-gray-100 text-gray-700 rounded-full hover:bg-gray-200"
              >
                #{tag.name}
              </LocalizedLink>
            ))}
          </div>
        </div>
      )}

      {/* JSON-LD structured data */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify({
            '@context': 'https://schema.org',
            '@type': 'BlogPosting',
            headline: post.title,
            description: post.excerpt,
            image: post.featuredImage,
            author: {
              '@type': 'Person',
              name: post.author.name
            },
            publisher: {
              '@type': 'Organization',
              name: 'Your Site Name',
              logo: {
                '@type': 'ImageObject',
                url: '/logo.png'
              }
            },
            datePublished: post.publishedAt,
            dateModified: post.updatedAt,
            mainEntityOfPage: {
              '@type': 'WebPage',
              '@id': `/${locale}/blog/${slug}`
            },
            inLanguage: locale,
            translationOfWork: post.translations.map(trans => ({
              '@type': 'BlogPosting',
              '@id': `/${trans.locale}/blog/${trans.slug}`,
              inLanguage: trans.locale
            }))
          })
        }}
      />
    </article>
  );
}

// app/[locale]/sitemap.xml/route.js - Localized sitemap
import { i18n } from '@/lib/i18n/config';

export async function GET(request, { params }) {
  const { locale } = params;
  
  // ✅ Get all pages for this locale
  const [blogPosts, products, staticPages] = await Promise.all([
    prisma.blogPost.findMany({
      where: { locale, published: true },
      select: { slug: true, updatedAt: true }
    }),
    prisma.product.findMany({
      where: { published: true },
      select: { slug: true, updatedAt: true }
    }),
    // Static pages with their priorities
    Promise.resolve([
      { slug: '', priority: 1.0, changefreq: 'daily' },
      { slug: 'about', priority: 0.8, changefreq: 'monthly' },
      { slug: 'contact', priority: 0.6, changefreq: 'monthly' },
      { slug: 'products', priority: 0.9, changefreq: 'weekly' },
      { slug: 'blog', priority: 0.9, changefreq: 'daily' }
    ])
  ]);

  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://yoursite.com';
  
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" 
            xmlns:xhtml="http://www.w3.org/1999/xhtml">
      ${staticPages.map(page => `
        <url>
          <loc>${baseUrl}/${locale}${page.slug ? `/${page.slug}` : ''}</loc>
          <changefreq>${page.changefreq}</changefreq>
          <priority>${page.priority}</priority>
          ${i18n.locales.map(l => `
            <xhtml:link 
              rel="alternate" 
              hreflang="${l}" 
              href="${baseUrl}/${l}${page.slug ? `/${page.slug}` : ''}" />
          `).join('')}
        </url>
      `).join('')}
      
      ${blogPosts.map(post => `
        <url>
          <loc>${baseUrl}/${locale}/blog/${post.slug}</loc>
          <lastmod>${post.updatedAt.toISOString()}</lastmod>
          <changefreq>weekly</changefreq>
          <priority>0.7</priority>
        </url>
      `).join('')}
      
      ${products.map(product => `
        <url>
          <loc>${baseUrl}/${locale}/products/${product.slug}</loc>
          <lastmod>${product.updatedAt.toISOString()}</lastmod>
          <changefreq>weekly</changefreq>
          <priority>0.8</priority>
        </url>
      `).join('')}
    </urlset>`;

  return new Response(sitemap, {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400'
    }
  });
}

// app/robots.txt/route.js - Robots.txt with sitemap references
export async function GET() {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://yoursite.com';
  
  const robots = `User-agent: *
Allow: /

# Sitemaps for all locales
${i18n.locales.map(locale => `Sitemap: ${baseUrl}/${locale}/sitemap.xml`).join('\n')}

# Disallow admin and API routes
Disallow: /admin
Disallow: /api
Disallow: /_next
`;

  return new Response(robots, {
    headers: {
      'Content-Type': 'text/plain',
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400'
    }
  });
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic i18n Setup**
1. ตั้งค่า middleware สำหรับ locale detection
2. สร้าง dictionary files สำหรับ 3 ภาษา
3. เพิ่ม LanguageProvider และ hooks
4. ทดสอบ language switching

### **🎯 แบบฝึกหัดที่ 2: Advanced Content Management**
1. สร้าง localized blog system
2. เพิ่ม translation linking
3. สร้าง SEO metadata สำหรับหลายภาษา
4. ทดสอบ search engine indexing

### **🎯 แบบฝึกหัดที่ 3: E-commerce i18n**
1. เพิ่ม multi-language product catalog
2. สร้าง localized pricing และ currency
3. เพิ่ม translated checkout process
4. ทดสอบ user experience ในแต่ละภาษา

### **🎯 แบบฝึกหัดที่ 4: Performance Optimization**
1. เพิ่ม lazy loading สำหรับ dictionaries
2. สร้าง caching strategies
3. เพิ่ม CDN configuration
4. วัด performance impact

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 16: Middleware](./16-middleware.md)

**บทต่อไป**: [บทที่ 18: SEO and Metadata](./18-seo-metadata.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Internationalization](https://nextjs.org/docs/app/building-your-application/routing/internationalization)
- [App Router i18n](https://nextjs.org/docs/app/building-your-application/routing/internationalization)
- [React Intl](https://formatjs.io/docs/react-intl/)
- [MDN Intl API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)
