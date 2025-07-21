# บทที่ 1: Introduction to Next.js

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจว่า Next.js คืออะไรและทำไมถึงได้รับความนิยม
- เรียนรู้ความแตกต่างระหว่าง Next.js กับ React
- ทำความเข้าใจ features หลักของ Next.js
- เข้าใจการใช้งานในโลกจริง

## 🤔 Next.js คืออะไร?

**Next.js** เป็น **React Framework** ที่พัฒนาโดย Vercel (เดิมชื่อ Zeit) เพื่อช่วยให้การสร้างแอปพลิเคชัน React มีประสิทธิภาพและง่ายขึ้น

### 📚 ประวัติความเป็นมา

```timeline
2016 ║ Next.js เปิดตัวครั้งแรก
2017 ║ เพิ่ม Dynamic Imports และ Static Exports
2018 ║ Server-side Rendering (SSR) ที่สมบูรณ์
2019 ║ API Routes และ Static Site Generation (SSG)
2020 ║ Image Optimization และ Incremental Static Regeneration
2021 ║ ปรับปรุงประสิทธิภาพและ Developer Experience
2022 ║ Middleware และ Edge Functions
2023 ║ App Router (React Server Components)
2024 ║ Partial Prerendering และ Server Actions
```

## ⚡ ทำไมต้องใช้ Next.js?

### 🚫 **ปัญหาของ React ธรรมดา**

```jsx
// ❌ ปัญหาต่างๆ ของ React แอป
function TraditionalReactApp() {
  return (
    <div>
      {/* 1. ⚠️ Client-side Rendering เท่านั้น */}
      {/* SEO ไม่ดี, First Paint ช้า */}
      
      {/* 2. ⚠️ ไม่มี Built-in Routing */}
      {/* ต้องติดตั้ง React Router แยก */}
      
      {/* 3. ⚠️ ไม่มี Image Optimization */}
      {/* รูปภาพใหญ่ ทำให้โหลดช้า */}
      
      {/* 4. ⚠️ Bundle Size ใหญ่ */}
      {/* ไม่มี Code Splitting อัตโนมัติ */}
      
      {/* 5. ⚠️ Configuration ซับซ้อน */}
      {/* ต้องตั้งค่า Webpack, Babel เอง */}
    </div>
  );
}
```

### ✅ **Next.js แก้ปัญหาเหล่านี้**

```jsx
// ✅ Next.js แก้ปัญหาให้หมด
function NextJsApp() {
  return (
    <div>
      {/* ✨ Server-side Rendering อัตโนมัติ */}
      {/* SEO ดี, First Paint เร็ว */}
      
      {/* ✨ File-based Routing */}
      {/* สร้างไฟล์ = สร้าง route */}
      
      {/* ✨ Image Optimization ในตัว */}
      {/* Next/Image ปรับขนาดอัตโนมัติ */}
      
      {/* ✨ Code Splitting อัตโนมัติ */}
      {/* แยก bundle ตามหน้า */}
      
      {/* ✨ Zero Configuration */}
      {/* ใช้ได้ทันทีไม่ต้องตั้งค่า */}
    </div>
  );
}
```

## 🔥 Features หลักของ Next.js

### 1. **Server-Side Rendering (SSR)**

```jsx
// pages/products/[id].js
export default function ProductPage({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <img src={product.image} alt={product.name} />
    </div>
  );
}

// ✨ Server-side Rendering
// HTML สร้างที่ server ส่งมาพร้อมข้อมูล
export async function getServerSideProps({ params }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`);
  
  return {
    props: {
      product: await product.json()
    }
  };
}
```

**ข้อดี SSR:**
- 🏃‍♂️ **First Paint เร็ว**: ผู้ใช้เห็นเนื้อหาทันที
- 🔍 **SEO ดีเยี่ยม**: Search engines อ่านข้อมูลได้
- 📱 **Mobile Friendly**: โหลดเร็วบนมือถือ

### 2. **Static Site Generation (SSG)**

```jsx
// pages/blog/index.js
export default function BlogPage({ posts }) {
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// ✨ Static Generation
// HTML สร้างตอน build time
export async function getStaticProps() {
  const posts = await fetch('https://api.example.com/posts');
  
  return {
    props: {
      posts: await posts.json()
    },
    // 🔄 Revalidate ทุก 3600 วินาที
    revalidate: 3600
  };
}
```

**ข้อดี SSG:**
- ⚡ **เร็วที่สุด**: HTML สำเร็จรูป
- 💰 **ประหยัด**: ใช้ CDN ได้
- 🛡️ **ปลอดภัย**: ไม่มี server-side code

### 3. **File-based Routing**

```bash
# ✨ โครงสร้างไฟล์ = Routes อัตโนมัติ
pages/
├── index.js          → / (Home page)
├── about.js          → /about
├── contact.js        → /contact
├── blog/
│   ├── index.js      → /blog
│   └── [slug].js     → /blog/any-slug
├── products/
│   ├── [id].js       → /products/123
│   └── [...all].js   → /products/any/nested/path
└── api/
    ├── users.js      → /api/users
    └── auth/
        └── login.js  → /api/auth/login
```

### 4. **API Routes**

```jsx
// pages/api/users.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    // ดึงข้อมูล users
    const users = [
      { id: 1, name: 'John Doe' },
      { id: 2, name: 'Jane Smith' }
    ];
    
    res.status(200).json(users);
  } else if (req.method === 'POST') {
    // สร้าง user ใหม่
    const { name, email } = req.body;
    
    // บันทึกลงฐานข้อมูล
    const newUser = { id: Date.now(), name, email };
    
    res.status(201).json(newUser);
  } else {
    res.setHeader('Allow', ['GET', 'POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}

// ✨ ใช้ API ในหน้าอื่น
export async function getServerSideProps() {
  const response = await fetch('http://localhost:3000/api/users');
  const users = await response.json();
  
  return { props: { users } };
}
```

### 5. **Image Optimization**

```jsx
import Image from 'next/image';

export default function ProductCard({ product }) {
  return (
    <div className="product-card">
      {/* ❌ HTML img tag ธรรมดา */}
      <img 
        src={product.image} 
        alt={product.name}
        width="300"
        height="200"
        // ⚠️ ไม่มี optimization
      />
      
      {/* ✅ Next.js Image component */}
      <Image
        src={product.image}
        alt={product.name}
        width={300}
        height={200}
        // ✨ Features ที่ได้ฟรี:
        priority          // โหลดก่อนรูปอื่น
        placeholder="blur" // แสดง blur ขณะโหลด
        quality={90}      // ปรับคุณภาพ
        sizes="(max-width: 768px) 100vw, 50vw" // Responsive
      />
    </div>
  );
}
```

**ข้อดี Next/Image:**
- 🎯 **Lazy Loading**: โหลดเมื่อจำเป็น
- 📱 **Responsive**: ปรับขนาดตามหน้าจอ
- 🗜️ **Format Optimization**: แปลงเป็น WebP อัตโนมัติ
- ⚡ **Performance**: ลดขนาดไฟล์ 50-80%

### 6. **Code Splitting อัตโนมัติ**

```jsx
// pages/dashboard.js
import { useState } from 'react';
import dynamic from 'next/dynamic';

// ✨ Dynamic Import - โหลดเมื่อจำเป็น
const HeavyChart = dynamic(() => import('../components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false // ไม่ render ที่ server
});

const ExpensiveModal = dynamic(() => import('../components/ExpensiveModal'));

export default function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [showModal, setShowModal] = useState(false);

  return (
    <div>
      <h1>Dashboard</h1>
      
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>
      
      {/* ✨ Component โหลดเมื่อ showChart = true */}
      {showChart && <HeavyChart />}
      
      {/* ✨ Modal โหลดเมื่อ showModal = true */}
      {showModal && <ExpensiveModal onClose={() => setShowModal(false)} />}
    </div>
  );
}

// ✨ Route-based Code Splitting
// แต่ละหน้ามี bundle แยก
// /dashboard → dashboard.js
// /profile → profile.js
// /settings → settings.js
```

## 🆚 เปรียบเทียบ Next.js กับเครื่องมืออื่น

### **Next.js vs Create React App**

| คุณสมบัติ | Next.js | Create React App |
|-----------|---------|------------------|
| **Rendering** | SSR + SSG + CSR | CSR เท่านั้น |
| **Routing** | File-based ในตัว | ต้องติดตั้ง React Router |
| **API** | API Routes ในตัว | ต้องสร้าง backend แยก |
| **Image Optimization** | ในตัว | ต้องติดตั้งเพิ่ม |
| **Code Splitting** | อัตโนมัติ | ต้องตั้งค่าเอง |
| **SEO** | ดีมาก | จำกัด |
| **Performance** | เร็วมาก | ปานกลาง |
| **Configuration** | ไม่ต้องตั้งค่า | ต้อง eject หรือ override |

### **Next.js vs Gatsby**

| คุณสมบัติ | Next.js | Gatsby |
|-----------|---------|---------|
| **Target Use Case** | Web Apps, E-commerce | Static Sites, Blogs |
| **Data Sources** | Any | GraphQL เท่านั้น |
| **Build Time** | เร็ว | ช้า (โปรเจ็กต์ใหญ่) |
| **Learning Curve** | ง่าย | ยาก (ต้องเรียน GraphQL) |
| **Flexibility** | สูงมาก | จำกัด |
| **Real-time Data** | ดี | จำกัด |

### **Next.js vs Nuxt.js (Vue)**

| คุณสมบัติ | Next.js | Nuxt.js |
|-----------|---------|---------|
| **Framework Base** | React | Vue |
| **Community** | ใหญ่มาก | ขนาดกลาง |
| **TypeScript** | ดีมาก | ดี |
| **Ecosystem** | กว้างมาก | ขนาดกลาง |
| **Performance** | เยี่ยม | เยี่ยม |
| **Job Market** | มากมาย | น้อยกว่า |

## 🌍 การใช้งานในโลกจริง

### **🏢 บริษัทที่ใช้ Next.js**

```typescript
const companiesUsingNextJS = [
  {
    name: "Netflix",
    useCase: "Landing pages และ marketing sites",
    reason: "SEO ดี, Performance เยี่ยม"
  },
  {
    name: "TikTok",
    useCase: "Web version ของแอป",
    reason: "SSR สำหรับ social sharing"
  },
  {
    name: "Twitch", 
    useCase: "Creator dashboard",
    reason: "Real-time data + SSR"
  },
  {
    name: "Hulu",
    useCase: "Streaming platform",
    reason: "Image optimization + Performance"
  },
  {
    name: "Notion",
    useCase: "Public pages",
    reason: "Static generation + SEO"
  }
];
```

### **📊 ประเภทโปรเจ็กต์ที่เหมาะกับ Next.js**

#### ✅ **เหมาะสำหรับ:**

```typescript
const perfectForNextJS = {
  eCommerce: {
    examples: ["Shopify stores", "Product catalogs"],
    benefits: ["SEO ดี", "Performance", "Image optimization"]
  },
  
  blogs: {
    examples: ["Corporate blogs", "Personal blogs", "News sites"],
    benefits: ["Static generation", "Fast loading", "SEO"]
  },
  
  marketingPages: {
    examples: ["Landing pages", "Company websites"],
    benefits: ["First paint เร็ว", "Conversion ดี"]
  },
  
  dashboards: {
    examples: ["Admin panels", "Analytics dashboards"],
    benefits: ["Authentication", "API routes", "Real-time data"]
  },
  
  socialPlatforms: {
    examples: ["Forums", "Social networks"],
    benefits: ["SSR สำหรับ sharing", "Dynamic content"]
  }
};
```

#### ❌ **อาจไม่เหมาะสำหรับ:**

```typescript
const notIdealForNextJS = {
  mobileApps: {
    reason: "ใช้ React Native ดีกว่า",
    alternative: "Expo, React Native"
  },
  
  gameApplications: {
    reason: "ต้อง Real-time performance สูง",
    alternative: "Unity, Unreal Engine"
  },
  
  desktopApps: {
    reason: "Native performance ดีกว่า",
    alternative: "Electron, Tauri"
  },
  
  simpleStaticSites: {
    reason: "Overkill สำหรับเว็บง่ายๆ",
    alternative: "HTML/CSS, Jekyll"
  }
};
```

## 🎯 ตัวอย่างโค้ดเปรียบเทียบ

### **📄 สร้างหน้าง่ายๆ**

#### **React แบบเดิม:**

```jsx
// ❌ Create React App
// App.js
import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}

// pages/Home.js
import React, { useState, useEffect } from 'react';

function Home() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => {
        setPosts(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>My Blog</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}
```

#### **Next.js:**

```jsx
// ✅ Next.js
// pages/index.js
export default function Home({ posts }) {
  return (
    <div>
      <h1>My Blog</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// ✨ Data fetching ที่ server
export async function getStaticProps() {
  const posts = await fetch('https://api.example.com/posts');
  
  return {
    props: {
      posts: await posts.json()
    },
    revalidate: 3600 // อัปเดตทุกชั่วโมง
  };
}

// pages/about.js - สร้างไฟล์ใหม่ = route ใหม่
export default function About() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Welcome to our amazing website!</p>
    </div>
  );
}
```

### **🔄 การแปลงจาก React ไป Next.js**

```jsx
// ❌ ก่อน: React Component ธรรมดา
import React, { useState, useEffect } from 'react';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchProducts();
  }, []);

  const fetchProducts = async () => {
    try {
      const response = await fetch('/api/products');
      const data = await response.json();
      setProducts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      {products.map(product => (
        <div key={product.id}>
          <img src={product.image} alt={product.name} />
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}

// ✅ หลัง: Next.js Page
import Image from 'next/image';

export default function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <div key={product.id}>
          <Image
            src={product.image}
            alt={product.name}
            width={300}
            height={200}
            priority={product.featured}
          />
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}

// ✨ Server-side data fetching
export async function getServerSideProps() {
  try {
    const response = await fetch('https://api.example.com/products');
    const products = await response.json();
    
    return {
      props: { products }
    };
  } catch (error) {
    return {
      props: { products: [] }
    };
  }
}
```

## 📈 ประสิทธิภาพที่ได้

### **⚡ Performance Metrics**

```typescript
const performanceComparison = {
  traditionalReact: {
    firstContentfulPaint: "2.5s",
    largestContentfulPaint: "4.2s",
    cumulativeLayoutShift: "0.15",
    firstInputDelay: "120ms",
    seoScore: "40/100"
  },
  
  nextJS: {
    firstContentfulPaint: "0.8s",    // 🔥 68% เร็วขึ้น
    largestContentfulPaint: "1.5s",  // 🔥 64% เร็วขึ้น
    cumulativeLayoutShift: "0.05",   // 🔥 67% ดีขึ้น
    firstInputDelay: "45ms",         // 🔥 63% เร็วขึ้น
    seoScore: "95/100"               // 🔥 138% ดีขึ้น
  }
};
```

### **📊 Bundle Size Comparison**

```typescript
const bundleSizeComparison = {
  createReactApp: {
    initialBundle: "250KB",
    vendorBundle: "180KB",
    totalSize: "430KB",
    codesplitting: "Manual"
  },
  
  nextJS: {
    initialBundle: "85KB",     // 🔥 66% เล็กลง
    vendorBundle: "120KB",     // 🔥 33% เล็กลง
    totalSize: "205KB",        // 🔥 52% เล็กลง
    codesplitting: "Automatic" // ✨ อัตโนมัติ
  }
};
```

## 🎯 สรุปข้อดีของ Next.js

### **🚀 Developer Experience**

```typescript
const developerBenefits = {
  productivity: {
    zeroConfig: "ไม่ต้องตั้งค่า Webpack, Babel",
    hotReload: "Fast Refresh ทำงานเร็วมาก",
    fileBasedRouting: "สร้างไฟล์ = สร้าง route",
    builtInCSS: "CSS Modules, Sass ในตัว"
  },
  
  performance: {
    automaticOptimization: "Bundle และ Image optimization อัตโนมัติ",
    codeSplitting: "แยก bundle ตามหน้าอัตโนมัติ",
    prefetching: "Prefetch หน้าที่จะไปต่อ",
    imageOptimization: "WebP conversion อัตโนมัติ"
  },
  
  seo: {
    serverSideRendering: "HTML สำเร็จรูป",
    staticGeneration: "เร็วที่สุดสำหรับ SEO",
    metaData: "จัดการ meta tags ง่าย",
    structuredData: "Schema markup ในตัว"
  }
};
```

### **💼 Business Benefits**

```typescript
const businessBenefits = {
  timeToMarket: {
    development: "พัฒนาเร็วขึ้น 40-60%",
    deployment: "Deploy ง่ายด้วย Vercel",
    maintenance: "ซ่อมบำรุงง่ายกว่า"
  },
  
  cost: {
    hosting: "ใช้ CDN ได้ (ถูกกว่า)",
    development: "ทีมเล็กพัฒนาได้",
    performance: "Server cost ต่ำกว่า"
  },
  
  userExperience: {
    loading: "โหลดเร็วขึ้น 50-70%",
    seo: "Google ranking ดีขึ้น",
    conversion: "Conversion rate สูงขึ้น"
  }
};
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: วิเคราะห์ Use Case**
จากโปรเจ็กต์ต่อไปนี้ ให้วิเคราะห์ว่าควรใช้ Next.js หรือไม่ และเพราะอะไร:

1. **Blog ส่วนตัว** - 50 บทความ, อัปเดตสัปดาห์ละครั้ง
2. **E-commerce Shop** - 10,000 สินค้า, ต้องการ SEO ดี
3. **Admin Dashboard** - จัดการข้อมูลลูกค้า, Real-time updates
4. **Portfolio Website** - แสดงผลงาน, Static content
5. **Social Media App** - Post, Comment, Like, Real-time chat

### **🎯 แบบฝึกหัดที่ 2: เปรียบเทียบ Performance**
ให้เปรียบเทียบ performance ระหว่าง:
- Traditional React App (Client-side only)
- Next.js SSR
- Next.js SSG

สำหรับเว็บไซต์ข่าวที่มี 1000 บทความ

### **🎯 แบบฝึกหัดที่ 3: Migration Planning**
คุณมี React App ที่ใช้:
- Create React App
- React Router
- Axios สำหรับ API calls
- CSS Modules

วางแผนการย้ายไป Next.js โดยระบุ:
1. ขั้นตอนการย้าย
2. ปัญหาที่อาจเจอ
3. ประโยชน์ที่จะได้

## 🔗 การเชื่อมต่อ

**บทต่อไป**: [บทที่ 2: Environment Setup](./02-environment-setup.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Official Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [Vercel Platform](https://vercel.com)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
