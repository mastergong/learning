# บทที่ 4: Pages and Routing

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจระบบ File-based Routing ของ Next.js
- เรียนรู้การสร้าง Static และ Dynamic Routes
- ทำความเข้าใจ Nested Routes และ Catch-all Routes
- เรียนรู้การ Navigate ระหว่างหน้า

## 🛣️ File-based Routing

### **📁 Routing Basics**

```
pages/
├── index.js          → /
├── about.js          → /about
├── contact.js        → /contact
├── pricing.js        → /pricing
└── 📁 blog/
    ├── index.js      → /blog
    ├── [slug].js     → /blog/my-first-post
    └── [...all].js   → /blog/2024/01/post
```

### **🎯 Basic Pages**

```jsx
// pages/index.js - Home Page
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to My Website</h1>
      <p>This is the home page</p>
    </div>
  );
}

// pages/about.js - About Page
export default function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company</p>
    </div>
  );
}

// pages/contact.js - Contact Page  
export default function ContactPage() {
  return (
    <div>
      <h1>Contact Us</h1>
      <form>
        <input type="text" placeholder="Your name" />
        <input type="email" placeholder="Your email" />
        <textarea placeholder="Your message"></textarea>
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

## 🎪 Dynamic Routes

### **📋 Single Dynamic Route**

```jsx
// pages/blog/[slug].js
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;

  return (
    <article>
      <h1>Blog Post: {slug}</h1>
      <p>This is the content for {slug}</p>
    </article>
  );
}

// ✨ URLs ที่จะ match:
// /blog/my-first-post → { slug: 'my-first-post' }
// /blog/hello-world → { slug: 'hello-world' }
// /blog/nextjs-tutorial → { slug: 'nextjs-tutorial' }
```

### **🔢 Multiple Dynamic Routes**

```jsx
// pages/products/[category]/[id].js
import { useRouter } from 'next/router';

export default function ProductPage() {
  const router = useRouter();
  const { category, id } = router.query;

  return (
    <div>
      <h1>Product ID: {id}</h1>
      <p>Category: {category}</p>
      <p>Product details go here...</p>
    </div>
  );
}

// ✨ URLs ที่จะ match:
// /products/electronics/iphone-15 → { category: 'electronics', id: 'iphone-15' }
// /products/books/react-guide → { category: 'books', id: 'react-guide' }
// /products/clothing/t-shirt → { category: 'clothing', id: 't-shirt' }
```

### **🌟 Optional Dynamic Routes**

```jsx
// pages/shop/[[...slug]].js
import { useRouter } from 'next/router';

export default function ShopPage() {
  const router = useRouter();
  const { slug } = router.query;

  // slug เป็น array หรือ undefined
  const path = slug ? slug.join('/') : '';

  return (
    <div>
      <h1>Shop</h1>
      {path ? (
        <p>Category path: {path}</p>
      ) : (
        <p>All products</p>
      )}
    </div>
  );
}

// ✨ URLs ที่จะ match:
// /shop → { slug: undefined }
// /shop/electronics → { slug: ['electronics'] }
// /shop/electronics/phones → { slug: ['electronics', 'phones'] }
// /shop/electronics/phones/iphone → { slug: ['electronics', 'phones', 'iphone'] }
```

## 🗂️ Nested Routes

### **📁 Folder Structure**

```
pages/
├── dashboard/
│   ├── index.js          → /dashboard
│   ├── profile.js        → /dashboard/profile
│   ├── settings.js       → /dashboard/settings
│   └── projects/
│       ├── index.js      → /dashboard/projects
│       ├── new.js        → /dashboard/projects/new
│       └── [id].js       → /dashboard/projects/123
```

### **🎨 Nested Layout Example**

```jsx
// pages/dashboard/index.js
import DashboardLayout from '@/components/DashboardLayout';

export default function DashboardPage() {
  return (
    <DashboardLayout>
      <h1>Dashboard Overview</h1>
      <div className="stats-grid">
        <div className="stat-card">
          <h3>Total Users</h3>
          <p>1,234</p>
        </div>
        <div className="stat-card">
          <h3>Revenue</h3>
          <p>$12,345</p>
        </div>
      </div>
    </DashboardLayout>
  );
}

// pages/dashboard/profile.js
import DashboardLayout from '@/components/DashboardLayout';

export default function ProfilePage() {
  return (
    <DashboardLayout>
      <h1>User Profile</h1>
      <form>
        <input type="text" placeholder="Full Name" />
        <input type="email" placeholder="Email" />
        <button type="submit">Update Profile</button>
      </form>
    </DashboardLayout>
  );
}

// components/DashboardLayout.js
import Link from 'next/link';
import { useRouter } from 'next/router';

export default function DashboardLayout({ children }) {
  const router = useRouter();

  const navigation = [
    { name: 'Overview', href: '/dashboard' },
    { name: 'Profile', href: '/dashboard/profile' },
    { name: 'Settings', href: '/dashboard/settings' },
    { name: 'Projects', href: '/dashboard/projects' },
  ];

  return (
    <div className="dashboard-layout">
      <aside className="sidebar">
        <nav>
          {navigation.map((item) => (
            <Link 
              key={item.href} 
              href={item.href}
              className={router.pathname === item.href ? 'active' : ''}
            >
              {item.name}
            </Link>
          ))}
        </nav>
      </aside>
      <main className="content">
        {children}
      </main>
    </div>
  );
}
```

## 🧭 Navigation

### **🔗 Link Component**

```jsx
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav>
      {/* ✅ Basic Link */}
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/contact">Contact</Link>

      {/* ✅ Dynamic Link */}
      <Link href="/blog/my-first-post">
        Read My First Post
      </Link>

      {/* ✅ Link with Query Parameters */}
      <Link href="/search?q=next.js&category=tutorial">
        Search Next.js Tutorials
      </Link>

      {/* ✅ Link Object Syntax */}
      <Link
        href={{
          pathname: '/blog/[slug]',
          query: { slug: 'my-post' }
        }}
      >
        My Post
      </Link>

      {/* ✅ External Link */}
      <Link 
        href="https://nextjs.org" 
        target="_blank" 
        rel="noopener noreferrer"
      >
        Next.js Docs
      </Link>
    </nav>
  );
}
```

### **🎨 Active Link Styling**

```jsx
import Link from 'next/link';
import { useRouter } from 'next/router';

export default function NavItem({ href, children }) {
  const router = useRouter();
  const isActive = router.pathname === href;

  return (
    <Link 
      href={href}
      className={`nav-link ${isActive ? 'active' : ''}`}
    >
      {children}
    </Link>
  );
}

// การใช้งาน
export default function Navbar() {
  return (
    <nav>
      <NavItem href="/">Home</NavItem>
      <NavItem href="/about">About</NavItem>
      <NavItem href="/blog">Blog</NavItem>
      <NavItem href="/contact">Contact</NavItem>
    </nav>
  );
}
```

### **🎯 Programmatic Navigation**

```jsx
import { useRouter } from 'next/router';
import { useState } from 'react';

export default function SearchForm() {
  const router = useRouter();
  const [query, setQuery] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    
    // ✅ Navigate with router.push
    router.push(`/search?q=${encodeURIComponent(query)}`);
    
    // ✅ Navigate with object
    router.push({
      pathname: '/search',
      query: { q: query, category: 'all' }
    });
  };

  const handleBack = () => {
    // ✅ Go back in history
    router.back();
  };

  const handleReplace = () => {
    // ✅ Replace current route (no back history)
    router.replace('/dashboard');
  };

  const handlePrefetch = () => {
    // ✅ Prefetch a route
    router.prefetch('/blog');
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search..."
        />
        <button type="submit">Search</button>
      </form>
      
      <button onClick={handleBack}>Go Back</button>
      <button onClick={handleReplace}>Go to Dashboard</button>
      <button onClick={handlePrefetch}>Prefetch Blog</button>
    </div>
  );
}
```

## 🌐 Query Parameters

### **📥 Reading Query Parameters**

```jsx
// pages/search.js
import { useRouter } from 'next/router';
import { useEffect, useState } from 'react';

export default function SearchPage() {
  const router = useRouter();
  const [results, setResults] = useState([]);
  
  // ✅ Get query parameters
  const { q, category, page, sort } = router.query;

  useEffect(() => {
    if (router.isReady && q) {
      // ✅ Perform search when query changes
      performSearch();
    }
  }, [router.isReady, q, category, page, sort]);

  const performSearch = async () => {
    const params = new URLSearchParams({
      q: q || '',
      category: category || 'all',
      page: page || '1',
      sort: sort || 'relevance'
    });

    const response = await fetch(`/api/search?${params}`);
    const data = await response.json();
    setResults(data.results);
  };

  const updateQuery = (newParams) => {
    router.push({
      pathname: router.pathname,
      query: { ...router.query, ...newParams }
    });
  };

  return (
    <div>
      <h1>Search Results for "{q}"</h1>
      
      {/* ✅ Filters */}
      <div className="filters">
        <select 
          value={category || 'all'}
          onChange={(e) => updateQuery({ category: e.target.value, page: '1' })}
        >
          <option value="all">All Categories</option>
          <option value="blog">Blog Posts</option>
          <option value="products">Products</option>
          <option value="pages">Pages</option>
        </select>

        <select
          value={sort || 'relevance'}
          onChange={(e) => updateQuery({ sort: e.target.value, page: '1' })}
        >
          <option value="relevance">Relevance</option>
          <option value="date">Date</option>
          <option value="title">Title</option>
        </select>
      </div>

      {/* ✅ Results */}
      <div className="results">
        {results.map((result) => (
          <div key={result.id} className="result-item">
            <h3>{result.title}</h3>
            <p>{result.excerpt}</p>
          </div>
        ))}
      </div>

      {/* ✅ Pagination */}
      <div className="pagination">
        <button 
          disabled={!page || page === '1'}
          onClick={() => updateQuery({ page: String(Number(page || 1) - 1) })}
        >
          Previous
        </button>
        
        <span>Page {page || 1}</span>
        
        <button 
          onClick={() => updateQuery({ page: String(Number(page || 1) + 1) })}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

## 🎭 Route Groups (App Router)

### **📁 Route Groups Structure**

```
app/
├── (auth)/               # Route group - ไม่แสดงใน URL
│   ├── login/
│   │   └── page.js       → /login
│   ├── register/
│   │   └── page.js       → /register
│   └── layout.js         # Layout สำหรับ auth pages
├── (dashboard)/          # Route group
│   ├── analytics/
│   │   └── page.js       → /analytics
│   ├── settings/
│   │   └── page.js       → /settings
│   └── layout.js         # Layout สำหรับ dashboard
└── (marketing)/          # Route group
    ├── about/
    │   └── page.js       → /about
    ├── pricing/
    │   └── page.js       → /pricing
    └── layout.js         # Layout สำหรับ marketing
```

### **🎨 Route Group Layouts**

```jsx
// app/(auth)/layout.js
export default function AuthLayout({ children }) {
  return (
    <div className="auth-layout">
      <div className="auth-container">
        <div className="auth-header">
          <img src="/logo.png" alt="Logo" />
          <h1>Welcome Back</h1>
        </div>
        <div className="auth-content">
          {children}
        </div>
        <div className="auth-footer">
          <p>© 2024 My App. All rights reserved.</p>
        </div>
      </div>
    </div>
  );
}

// app/(dashboard)/layout.js
import Sidebar from '@/components/Sidebar';
import Header from '@/components/Header';

export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard-layout">
      <Header />
      <div className="dashboard-main">
        <Sidebar />
        <main className="dashboard-content">
          {children}
        </main>
      </div>
    </div>
  );
}

// app/(marketing)/layout.js
import Navbar from '@/components/Navbar';
import Footer from '@/components/Footer';

export default function MarketingLayout({ children }) {
  return (
    <div className="marketing-layout">
      <Navbar />
      <main className="marketing-content">
        {children}
      </main>
      <Footer />
    </div>
  );
}
```

## 🎪 Parallel Routes (App Router)

### **📁 Parallel Routes Structure**

```
app/
└── dashboard/
    ├── @analytics/       # Named slot
    │   └── page.js
    ├── @notifications/   # Named slot
    │   └── page.js
    ├── @team/           # Named slot
    │   └── page.js
    ├── layout.js        # รับ slots เป็น props
    └── page.js          # Main content
```

### **🎛️ Parallel Routes Usage**

```jsx
// app/dashboard/layout.js
export default function DashboardLayout({
  children,       // หน้าหลัก
  analytics,      // @analytics slot
  notifications,  // @notifications slot
  team           // @team slot
}) {
  return (
    <div className="dashboard">
      <div className="main-content">
        {children}
      </div>
      <div className="sidebar">
        <div className="analytics-section">
          {analytics}
        </div>
        <div className="notifications-section">
          {notifications}
        </div>
        <div className="team-section">
          {team}
        </div>
      </div>
    </div>
  );
}

// app/dashboard/@analytics/page.js
export default function Analytics() {
  return (
    <div className="analytics-widget">
      <h3>Analytics</h3>
      <div className="charts">
        <div>Users: 1,234</div>
        <div>Sessions: 5,678</div>
        <div>Revenue: $12,345</div>
      </div>
    </div>
  );
}

// app/dashboard/@notifications/page.js
export default function Notifications() {
  return (
    <div className="notifications-widget">
      <h3>Notifications</h3>
      <ul>
        <li>New user registered</li>
        <li>Payment received</li>
        <li>Server maintenance scheduled</li>
      </ul>
    </div>
  );
}

// app/dashboard/@team/page.js
export default function Team() {
  return (
    <div className="team-widget">
      <h3>Team Status</h3>
      <ul>
        <li>John - Online</li>
        <li>Sarah - Away</li>
        <li>Mike - Offline</li>
      </ul>
    </div>
  );
}
```

## 🔄 Intercepting Routes (App Router)

### **📁 Intercepting Routes Structure**

```
app/
├── feed/
│   ├── (..)photo/       # Intercept /photo
│   │   └── [id]/
│   │       └── page.js
│   └── page.js
├── photo/
│   └── [id]/
│       └── page.js
└── layout.js
```

### **🎭 Modal Intercept Example**

```jsx
// app/feed/(..)photo/[id]/page.js
import Modal from '@/components/Modal';
import Photo from '@/components/Photo';

export default function InterceptedPhoto({ params }) {
  return (
    <Modal>
      <Photo id={params.id} />
    </Modal>
  );
}

// app/photo/[id]/page.js  
import Photo from '@/components/Photo';

export default function PhotoPage({ params }) {
  return (
    <div className="photo-page">
      <Photo id={params.id} />
    </div>
  );
}

// ✨ เมื่อ navigate จาก /feed ไป /photo/123
// จะแสดง Modal แทนการไปหน้าใหม่
// แต่ถ้า refresh หรือ direct access จะไปหน้าเต็ม
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic Routing**
1. สร้างหน้า home, about, services, contact
2. เพิ่ม navigation menu ที่แสดง active state
3. สร้าง 404 page ที่สวยงาม
4. ทดสอบการ navigate ระหว่างหน้า

### **🎯 แบบฝึกหัดที่ 2: Dynamic Routes**
1. สร้าง blog system ด้วย /blog/[slug]
2. สร้าง product catalog ด้วย /products/[category]/[id]
3. เพิ่ม search page ที่รับ query parameters
4. สร้าง user profile page /users/[username]

### **🎯 แบบฝึกหัดที่ 3: Advanced Routing**
1. สร้าง admin dashboard ด้วย nested routes
2. เพิ่ม catch-all route สำหรับ documentation
3. สร้าง route groups สำหรับ auth pages
4. ทดสอบ parallel routes

### **🎯 แบบฝึกหัดที่ 4: Navigation Patterns**
1. สร้าง breadcrumb navigation
2. เพิ่ม search autocomplete ด้วย programmatic navigation
3. สร้าง infinite scroll pagination
4. เพิ่ม modal routing pattern

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 3: Project Structure](./03-project-structure.md)

**บทต่อไป**: [บทที่ 5: Components and Styling](./05-components-styling.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Routing](https://nextjs.org/docs/routing/introduction)
- [Dynamic Routes](https://nextjs.org/docs/routing/dynamic-routes)
- [App Router](https://nextjs.org/docs/app/building-your-application/routing)
