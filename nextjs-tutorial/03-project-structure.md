# บทที่ 3: Project Structure

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจโครงสร้างโปรเจ็กต์ Next.js
- เรียนรู้ File-based Routing
- ทำความเข้าใจ Pages Router vs App Router
- เรียนรู้การจัดระเบียบไฟล์และโฟลเดอร์

## 📁 โครงสร้างโปรเจ็กต์ Next.js

### **🗂️ โครงสร้างพื้นฐาน**

```
my-next-app/
├── 📁 public/              # Static files
├── 📁 src/                 # Source code
├── 📁 styles/              # CSS files  
├── 📄 package.json         # Dependencies & scripts
├── 📄 next.config.js       # Next.js configuration
├── 📄 tailwind.config.js   # Tailwind CSS config
├── 📄 tsconfig.json        # TypeScript config
├── 📄 .eslintrc.json       # ESLint rules
├── 📄 .gitignore           # Git ignore rules
└── 📄 README.md            # Documentation
```

## 📂 โฟลเดอร์ Public

### **🌐 Static Assets**

```
public/
├── favicon.ico             # Website icon
├── vercel.svg             # Vercel logo
├── next.svg               # Next.js logo
├── 📁 images/             # Images
│   ├── hero.jpg
│   ├── logo.png
│   └── avatars/
├── 📁 icons/              # Icons
│   ├── apple-touch-icon.png
│   └── android-chrome-192x192.png
├── 📁 fonts/              # Custom fonts
│   ├── Inter-Regular.woff2
│   └── Inter-Bold.woff2
├── robots.txt             # SEO robots file
├── sitemap.xml            # SEO sitemap
└── manifest.json          # PWA manifest
```

### **🎯 การใช้ Static Files**

```jsx
// ใน components หรือ pages
export default function HomePage() {
  return (
    <div>
      {/* ❌ ผิด - ไม่ต้องใส่ public/ */}
      <img src="/public/images/hero.jpg" alt="Hero" />
      
      {/* ✅ ถูก - เริ่มจาก root */}
      <img src="/images/hero.jpg" alt="Hero" />
      
      {/* ✅ Next.js Image component */}
      <Image 
        src="/images/hero.jpg" 
        alt="Hero"
        width={800}
        height={400}
      />
      
      {/* ✅ Icons */}
      <img src="/icons/logo.png" alt="Logo" />
      
      {/* ✅ Documents */}
      <a href="/files/brochure.pdf" download>
        Download Brochure
      </a>
    </div>
  );
}
```

### **📄 Special Files ใน Public**

```html
<!-- public/robots.txt -->
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/private/

Sitemap: https://mywebsite.com/sitemap.xml
```

```xml
<!-- public/sitemap.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://mywebsite.com</loc>
    <lastmod>2024-01-01T00:00:00.000Z</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://mywebsite.com/about</loc>
    <lastmod>2024-01-01T00:00:00.000Z</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

```json
// public/manifest.json (PWA)
{
  "name": "My Next.js App",
  "short_name": "NextApp",
  "description": "A awesome Next.js application",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icons/android-chrome-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/android-chrome-512x512.png", 
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

## 🚦 Pages Router (Legacy)

### **📄 โครงสร้าง Pages Router**

```
src/
└── pages/
    ├── _app.js            # App wrapper
    ├── _document.js       # Document wrapper  
    ├── _error.js          # Custom error page
    ├── 404.js             # Custom 404 page
    ├── 500.js             # Custom 500 page
    ├── index.js           # Home page (/)
    ├── about.js           # About page (/about)
    ├── contact.js         # Contact page (/contact)
    ├── 📁 blog/
    │   ├── index.js       # Blog home (/blog)
    │   ├── [slug].js      # Dynamic route (/blog/my-post)
    │   └── [...all].js    # Catch-all (/blog/2024/01/post)
    ├── 📁 products/
    │   ├── index.js       # Products (/products)
    │   ├── [id].js        # Product detail (/products/123)
    │   └── category/
    │       └── [name].js  # Category (/products/category/electronics)
    └── 📁 api/            # API routes
        ├── users.js       # /api/users
        ├── auth/
        │   ├── login.js   # /api/auth/login
        │   └── logout.js  # /api/auth/logout
        └── products/
            ├── index.js   # /api/products
            └── [id].js    # /api/products/123
```

### **🔧 Special Files ใน Pages Router**

#### **_app.js - App Wrapper:**

```jsx
// pages/_app.js
import '@/styles/globals.css';
import Layout from '@/components/Layout';
import { Provider } from 'react-redux';
import { store } from '@/store';

export default function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <Layout>
        <Component {...pageProps} />
      </Layout>
    </Provider>
  );
}

// ✨ Features ที่ใช้ใน _app.js:
// - Global CSS imports
// - Global state providers  
// - Layout components
// - Error boundaries
// - Analytics tracking
```

#### **_document.js - Document Wrapper:**

```jsx
// pages/_document.js
import { Html, Head, Main, NextScript } from 'next/document';

export default function Document() {
  return (
    <Html lang="en">
      <Head>
        {/* Global meta tags */}
        <meta name="application-name" content="My App" />
        <meta name="apple-mobile-web-app-capable" content="yes" />
        
        {/* Custom fonts */}
        <link
          href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
          rel="stylesheet"
        />
        
        {/* Analytics */}
        <script
          async
          src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"
        />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}

// ✨ ใช้สำหรับ:
// - Custom HTML document structure
// - Global head tags
// - Custom fonts
// - Third-party scripts
// - Server-side rendering enhancements
```

### **🛣️ File-based Routing Examples**

```jsx
// pages/index.js → /
export default function HomePage() {
  return <h1>Welcome to Home Page</h1>;
}

// pages/about.js → /about  
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// pages/blog/index.js → /blog
export default function BlogPage() {
  return <h1>Blog Home</h1>;
}

// pages/blog/[slug].js → /blog/my-first-post
export default function BlogPost({ slug }) {
  return <h1>Blog Post: {slug}</h1>;
}

export async function getStaticPaths() {
  return {
    paths: [
      { params: { slug: 'my-first-post' } },
      { params: { slug: 'second-post' } }
    ],
    fallback: false
  };
}

export async function getStaticProps({ params }) {
  return {
    props: {
      slug: params.slug
    }
  };
}

// pages/products/[...all].js → /products/electronics/phones/iphone
export default function ProductPath({ segments }) {
  return (
    <div>
      <h1>Product Path</h1>
      <p>Segments: {segments.join(' / ')}</p>
    </div>
  );
}

export async function getServerSideProps({ params }) {
  return {
    props: {
      segments: params.all
    }
  };
}
```

## 🚀 App Router (Next.js 13+)

### **📁 โครงสร้าง App Router**

```
src/
└── app/
    ├── globals.css        # Global styles
    ├── layout.js          # Root layout
    ├── page.js            # Home page (/)
    ├── loading.js         # Loading UI
    ├── error.js           # Error UI  
    ├── not-found.js       # 404 page
    ├── 📁 about/
    │   ├── page.js        # /about
    │   ├── layout.js      # About layout
    │   └── loading.js     # About loading
    ├── 📁 blog/
    │   ├── page.js        # /blog
    │   ├── layout.js      # Blog layout
    │   ├── loading.js     # Blog loading
    │   └── 📁 [slug]/
    │       ├── page.js    # /blog/my-post
    │       ├── loading.js # Post loading
    │       └── error.js   # Post error
    ├── 📁 products/
    │   ├── page.js        # /products
    │   ├── 📁 [id]/
    │   │   └── page.js    # /products/123
    │   └── 📁 category/
    │       └── 📁 [name]/
    │           └── page.js # /products/category/electronics
    └── 📁 api/            # API routes
        ├── 📁 users/
        │   └── route.js   # /api/users
        └── 📁 products/
            ├── route.js   # /api/products
            └── 📁 [id]/
                └── route.js # /api/products/123
```

### **🎛️ Special Files ใน App Router**

#### **layout.js - Layout Components:**

```jsx
// app/layout.js - Root Layout
import './globals.css';
import { Inter } from 'next/font/google';
import Navbar from '@/components/Navbar';
import Footer from '@/components/Footer';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: 'My Next.js App',
  description: 'Generated by create next app',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Navbar />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}

// app/blog/layout.js - Blog Layout  
export default function BlogLayout({ children }) {
  return (
    <div className="blog-container">
      <aside className="sidebar">
        <h3>Categories</h3>
        <ul>
          <li>Technology</li>
          <li>Design</li>
          <li>Programming</li>
        </ul>
      </aside>
      <div className="content">
        {children}
      </div>
    </div>
  );
}
```

#### **page.js - Page Components:**

```jsx
// app/page.js - Home Page
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to My Website</h1>
      <p>This is the home page</p>
    </div>
  );
}

// app/about/page.js - About Page
export default function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company</p>
    </div>
  );
}

// app/blog/[slug]/page.js - Dynamic Blog Page
export default function BlogPost({ params }) {
  const { slug } = params;
  
  return (
    <article>
      <h1>Blog Post: {slug}</h1>
      <p>Content for {slug}</p>
    </article>
  );
}

// ✨ Generate static paths
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json());
  
  return posts.map(post => ({
    slug: post.slug
  }));
}
```

#### **loading.js - Loading States:**

```jsx
// app/loading.js - Global Loading
export default function Loading() {
  return (
    <div className="loading-container">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}

// app/blog/loading.js - Blog Loading
export default function BlogLoading() {
  return (
    <div className="blog-loading">
      <div className="skeleton-post">
        <div className="skeleton-title"></div>
        <div className="skeleton-content"></div>
      </div>
    </div>
  );
}

// app/products/[id]/loading.js - Product Loading
export default function ProductLoading() {
  return (
    <div className="product-skeleton">
      <div className="skeleton-image"></div>
      <div className="skeleton-details">
        <div className="skeleton-title"></div>
        <div className="skeleton-price"></div>
        <div className="skeleton-description"></div>
      </div>
    </div>
  );
}
```

#### **error.js - Error Handling:**

```jsx
// app/error.js - Global Error
'use client';

export default function Error({ error, reset }) {
  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>
        Try again
      </button>
    </div>
  );
}

// app/blog/error.js - Blog Error
'use client';

export default function BlogError({ error, reset }) {
  return (
    <div className="blog-error">
      <h2>Failed to load blog posts</h2>
      <p>Error: {error.message}</p>
      <button onClick={() => reset()}>
        Retry
      </button>
    </div>
  );
}
```

#### **not-found.js - 404 Pages:**

```jsx
// app/not-found.js - Global 404
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <p>Sorry, the page you are looking for does not exist.</p>
      <Link href="/">
        <button>Go Home</button>
      </Link>
    </div>
  );
}

// app/blog/not-found.js - Blog 404
import Link from 'next/link';

export default function BlogNotFound() {
  return (
    <div className="blog-not-found">
      <h1>Blog Post Not Found</h1>
      <p>The blog post you're looking for doesn't exist.</p>
      <Link href="/blog">
        <button>Back to Blog</button>
      </Link>
    </div>
  );
}
```

## 🎨 Styles Organization

### **📁 CSS Structure**

```
styles/
├── globals.css            # Global styles
├── 📁 components/         # Component styles
│   ├── Navbar.module.css
│   ├── Footer.module.css
│   ├── Button.module.css
│   └── Card.module.css
├── 📁 pages/              # Page-specific styles
│   ├── Home.module.css
│   ├── About.module.css
│   └── Blog.module.css
├── 📁 utilities/          # Utility classes
│   ├── colors.css
│   ├── typography.css
│   └── spacing.css
└── 📁 vendor/             # Third-party styles
    ├── bootstrap.min.css
    └── prism.css
```

### **🎯 CSS Usage Examples**

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom global styles */
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

html,
body {
  max-width: 100vw;
  overflow-x: hidden;
}

body {
  color: rgb(var(--foreground-rgb));
  background: linear-gradient(
      to bottom,
      transparent,
      rgb(var(--background-end-rgb))
    )
    rgb(var(--background-start-rgb));
}
```

```css
/* styles/components/Button.module.css */
.button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.primary {
  background-color: #3b82f6;
  color: white;
}

.primary:hover {
  background-color: #2563eb;
}

.secondary {
  background-color: #6b7280;
  color: white;
}

.large {
  padding: 1rem 2rem;
  font-size: 1.125rem;
}
```

```jsx
// การใช้ CSS Modules
import styles from '@/styles/components/Button.module.css';

export default function Button({ 
  children, 
  variant = 'primary', 
  size = 'medium',
  ...props 
}) {
  const className = `${styles.button} ${styles[variant]} ${styles[size]}`;
  
  return (
    <button className={className} {...props}>
      {children}
    </button>
  );
}
```

## 🗃️ Components Organization

### **📁 Components Structure**

```
src/
├── 📁 components/
│   ├── 📁 ui/             # Basic UI components
│   │   ├── Button.jsx
│   │   ├── Input.jsx
│   │   ├── Modal.jsx
│   │   ├── Card.jsx
│   │   └── index.js       # Barrel export
│   ├── 📁 layout/         # Layout components
│   │   ├── Navbar.jsx
│   │   ├── Footer.jsx
│   │   ├── Sidebar.jsx
│   │   └── Layout.jsx
│   ├── 📁 forms/          # Form components
│   │   ├── ContactForm.jsx
│   │   ├── LoginForm.jsx
│   │   └── RegisterForm.jsx
│   ├── 📁 features/       # Feature-specific
│   │   ├── 📁 auth/
│   │   │   ├── LoginButton.jsx
│   │   │   └── UserProfile.jsx
│   │   ├── 📁 blog/
│   │   │   ├── PostCard.jsx
│   │   │   ├── PostList.jsx
│   │   │   └── CategoryFilter.jsx
│   │   └── 📁 shop/
│   │       ├── ProductCard.jsx
│   │       ├── ShoppingCart.jsx
│   │       └── Checkout.jsx
│   └── 📁 common/         # Common components
│       ├── Loading.jsx
│       ├── ErrorBoundary.jsx
│       └── SEO.jsx
```

### **🎯 Barrel Exports**

```javascript
// components/ui/index.js
export { default as Button } from './Button';
export { default as Input } from './Input';
export { default as Modal } from './Modal';
export { default as Card } from './Card';

// การใช้งาน
import { Button, Input, Modal } from '@/components/ui';
```

## 🛠️ Utils และ Lib Organization

### **📁 Utilities Structure**

```
src/
├── 📁 lib/                # External libraries config
│   ├── auth.js            # Authentication config
│   ├── database.js        # Database connection
│   ├── email.js           # Email service
│   └── stripe.js          # Payment service
├── 📁 utils/              # Utility functions
│   ├── api.js             # API helpers
│   ├── auth.js            # Auth helpers
│   ├── date.js            # Date formatting
│   ├── format.js          # Text formatting
│   ├── validation.js      # Form validation
│   └── constants.js       # Constants
├── 📁 hooks/              # Custom React hooks
│   ├── useAuth.js
│   ├── useLocalStorage.js
│   ├── useFetch.js
│   └── useDebounce.js
└── 📁 types/              # TypeScript types
    ├── auth.ts
    ├── blog.ts
    ├── product.ts
    └── api.ts
```

### **🔧 Example Utilities**

```javascript
// utils/format.js
export const formatCurrency = (amount, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount);
};

export const formatDate = (date, options = {}) => {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    ...options,
  }).format(new Date(date));
};

export const slugify = (text) => {
  return text
    .toLowerCase()
    .replace(/[^\w ]+/g, '')
    .replace(/ +/g, '-');
};
```

```javascript
// utils/api.js
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000/api';

export const apiClient = {
  async get(endpoint) {
    const response = await fetch(`${API_BASE_URL}${endpoint}`);
    if (!response.ok) throw new Error('Failed to fetch');
    return response.json();
  },

  async post(endpoint, data) {
    const response = await fetch(`${API_BASE_URL}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    if (!response.ok) throw new Error('Failed to post');
    return response.json();
  },
};
```

```javascript
// hooks/useAuth.js
import { useState, useEffect } from 'react';

export function useAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check authentication status
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const response = await fetch('/api/auth/me');
      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
      }
    } catch (error) {
      console.error('Auth check failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const login = async (credentials) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials),
    });
    
    if (response.ok) {
      const userData = await response.json();
      setUser(userData);
      return userData;
    }
    
    throw new Error('Login failed');
  };

  const logout = async () => {
    await fetch('/api/auth/logout', { method: 'POST' });
    setUser(null);
  };

  return { user, loading, login, logout, checkAuth };
}
```

## 🔧 Configuration Files

### **⚙️ next.config.js Structure**

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // App Router
  experimental: {
    appDir: true,
  },
  
  // Images
  images: {
    domains: ['example.com', 'cdn.example.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  
  // Headers
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          { key: 'Access-Control-Allow-Origin', value: '*' },
          { key: 'Access-Control-Allow-Methods', value: 'GET,POST,PUT,DELETE' },
        ],
      },
    ];
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },
  
  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/proxy/:path*',
        destination: 'https://external-api.com/:path*',
      },
    ];
  },
};

module.exports = nextConfig;
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: โครงสร้างพื้นฐาน**
1. สร้างโปรเจ็กต์ Next.js ใหม่
2. จัดระเบียบโฟลเดอร์ตามมาตรฐาน
3. สร้าง components พื้นฐาน (Button, Card, Input)
4. ทดสอบ import/export

### **🎯 แบบฝึกหัดที่ 2: Pages Router**
1. สร้างหน้า about, contact, blog
2. สร้าง dynamic route สำหรับ blog posts
3. สร้าง catch-all route สำหรับ products
4. เพิ่ม custom _app.js และ _document.js

### **🎯 แบบฝึกหัดที่ 3: App Router**
1. แปลง Pages Router เป็น App Router
2. สร้าง layout สำหรับ blog section
3. เพิ่ม loading states
4. เพิ่ม error handling

### **🎯 แบบฝึกหัดที่ 4: Organization**
1. สร้างโครงสร้าง components แบบ feature-based
2. เพิ่ม utility functions
3. สร้าง custom hooks
4. จัดการ CSS ด้วย modules

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 2: Environment Setup](./02-environment-setup.md)

**บทต่อไป**: [บทที่ 4: Pages and Routing](./04-pages-routing.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Project Structure](https://nextjs.org/docs/getting-started/project-structure)
- [App Router Documentation](https://nextjs.org/docs/app)
- [Pages Router Documentation](https://nextjs.org/docs/pages)
