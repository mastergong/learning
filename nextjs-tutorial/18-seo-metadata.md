# บทที่ 18: SEO and Metadata

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ SEO optimization ใน Next.js
- ทำความเข้าใจ Metadata API และ Open Graph
- เรียนรู้ Structured data และ JSON-LD
- เข้าใจ Performance และ Core Web Vitals

## 🔍 Next.js Metadata API

### **⚡ Static and Dynamic Metadata**

```javascript
// app/layout.js - Root metadata
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  // ✅ Basic metadata
  title: {
    template: '%s | Your App Name',
    default: 'Your App Name - Best Products Online'
  },
  description: 'Discover amazing products with the best shopping experience. Fast delivery, great prices, and excellent customer service.',
  keywords: ['e-commerce', 'shopping', 'products', 'online store', 'delivery'],
  authors: [{ name: 'Your Name', url: 'https://yoursite.com' }],
  creator: 'Your Company',
  publisher: 'Your Company',
  
  // ✅ Open Graph metadata
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://yoursite.com',
    siteName: 'Your App Name',
    title: 'Your App Name - Best Products Online',
    description: 'Discover amazing products with the best shopping experience.',
    images: [
      {
        url: '/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'Your App Name - Best Products Online'
      }
    ]
  },
  
  // ✅ Twitter metadata
  twitter: {
    card: 'summary_large_image',
    site: '@yourhandle',
    creator: '@yourhandle',
    title: 'Your App Name - Best Products Online',
    description: 'Discover amazing products with the best shopping experience.',
    images: ['/twitter-image.jpg']
  },
  
  // ✅ Robot instructions
  robots: {
    index: true,
    follow: true,
    nocache: false,
    googleBot: {
      index: true,
      follow: true,
      noimageindex: false,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1
    }
  },
  
  // ✅ Icons and manifest
  icons: {
    icon: [
      { url: '/favicon-16x16.png', sizes: '16x16', type: 'image/png' },
      { url: '/favicon-32x32.png', sizes: '32x32', type: 'image/png' }
    ],
    apple: [
      { url: '/apple-touch-icon.png', sizes: '180x180', type: 'image/png' }
    ],
    shortcut: '/favicon.ico'
  },
  
  manifest: '/manifest.json',
  
  // ✅ Verification
  verification: {
    google: 'your-google-verification-code',
    yandex: 'your-yandex-verification-code',
    yahoo: 'your-yahoo-verification-code',
    other: {
      'facebook-domain-verification': 'your-facebook-verification-code'
    }
  },
  
  // ✅ Canonical and alternates
  alternates: {
    canonical: 'https://yoursite.com',
    languages: {
      'en-US': 'https://yoursite.com/en',
      'th-TH': 'https://yoursite.com/th',
      'ja-JP': 'https://yoursite.com/ja'
    }
  },
  
  // ✅ Other metadata
  category: 'technology',
  classification: 'business',
  referrer: 'origin-when-cross-origin',
  colorScheme: 'light dark',
  viewport: 'width=device-width, initial-scale=1',
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: '#ffffff' },
    { media: '(prefers-color-scheme: dark)', color: '#000000' }
  ]
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        {children}
      </body>
    </html>
  );
}

// app/products/[slug]/page.js - Dynamic metadata
async function getProduct(slug) {
  const product = await prisma.product.findUnique({
    where: { slug },
    include: {
      category: true,
      images: true,
      reviews: {
        select: { rating: true },
        where: { published: true }
      },
      brand: true
    }
  });
  
  return product;
}

export async function generateMetadata({ params, searchParams }) {
  const product = await getProduct(params.slug);
  
  if (!product) {
    return {
      title: 'Product Not Found',
      description: 'The product you are looking for was not found.',
      robots: { index: false, follow: false }
    };
  }

  // ✅ Calculate average rating
  const averageRating = product.reviews.length > 0
    ? product.reviews.reduce((acc, review) => acc + review.rating, 0) / product.reviews.length
    : 0;

  const formattedPrice = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(product.price / 100);

  return {
    title: product.name,
    description: product.description,
    keywords: [
      product.name,
      product.category.name,
      product.brand?.name,
      ...product.tags?.map(tag => tag.name) || []
    ],
    
    // ✅ Open Graph for products
    openGraph: {
      type: 'product',
      url: `https://yoursite.com/products/${product.slug}`,
      title: product.name,
      description: product.description,
      images: product.images.map(image => ({
        url: image.url,
        width: image.width || 800,
        height: image.height || 600,
        alt: image.alt || product.name
      })),
      siteName: 'Your Store',
      locale: 'en_US'
    },
    
    // ✅ Twitter product card
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description,
      images: [product.images[0]?.url],
      creator: '@yourhandle'
    },
    
    // ✅ Product-specific robots
    robots: {
      index: product.published,
      follow: true,
      nocache: !product.published
    },
    
    // ✅ Canonical URL
    alternates: {
      canonical: `https://yoursite.com/products/${product.slug}`
    }
  };
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.slug);
  
  if (!product) {
    return <div>Product not found</div>;
  }

  const averageRating = product.reviews.length > 0
    ? product.reviews.reduce((acc, review) => acc + review.rating, 0) / product.reviews.length
    : 0;

  // ✅ Structured data for product
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.images.map(img => img.url),
    brand: {
      '@type': 'Brand',
      name: product.brand?.name || 'Unknown'
    },
    category: product.category.name,
    sku: product.sku,
    offers: {
      '@type': 'Offer',
      url: `https://yoursite.com/products/${product.slug}`,
      priceCurrency: 'USD',
      price: (product.price / 100).toFixed(2),
      availability: product.stock > 0 
        ? 'https://schema.org/InStock' 
        : 'https://schema.org/OutOfStock',
      seller: {
        '@type': 'Organization',
        name: 'Your Store'
      }
    },
    aggregateRating: product.reviews.length > 0 ? {
      '@type': 'AggregateRating',
      ratingValue: averageRating.toFixed(1),
      reviewCount: product.reviews.length,
      bestRating: 5,
      worstRating: 1
    } : undefined
  };

  return (
    <>
      {/* Product content */}
      <div className="max-w-7xl mx-auto px-4 py-8">
        {/* Product display components */}
      </div>
      
      {/* ✅ JSON-LD structured data */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(structuredData)
        }}
      />
    </>
  );
}

// app/blog/[slug]/page.js - Blog post metadata
export async function generateMetadata({ params }) {
  const post = await getBlogPost(params.slug);
  
  if (!post) {
    return {
      title: 'Post Not Found',
      robots: { index: false, follow: false }
    };
  }

  return {
    title: post.title,
    description: post.excerpt,
    keywords: post.tags?.map(tag => tag.name),
    authors: [{ name: post.author.name, url: post.author.website }],
    
    openGraph: {
      type: 'article',
      url: `https://yoursite.com/blog/${post.slug}`,
      title: post.title,
      description: post.excerpt,
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
      ]
    },
    
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.featuredImage]
    },
    
    alternates: {
      canonical: `https://yoursite.com/blog/${post.slug}`
    }
  };
}
```

### **🏢 Organization and Website Schema**

```javascript
// components/StructuredData.js
export function OrganizationSchema() {
  const organizationData = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Your Company Name',
    url: 'https://yoursite.com',
    logo: 'https://yoursite.com/logo.png',
    description: 'Your company description',
    contactPoint: [
      {
        '@type': 'ContactPoint',
        telephone: '+1-555-123-4567',
        contactType: 'customer service',
        areaServed: 'US',
        availableLanguage: ['English', 'Thai', 'Japanese']
      }
    ],
    address: {
      '@type': 'PostalAddress',
      streetAddress: '123 Business St',
      addressLocality: 'City',
      addressRegion: 'State',
      postalCode: '12345',
      addressCountry: 'US'
    },
    sameAs: [
      'https://www.facebook.com/yourpage',
      'https://www.twitter.com/yourhandle',
      'https://www.instagram.com/yourhandle',
      'https://www.linkedin.com/company/yourcompany'
    ]
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(organizationData)
      }}
    />
  );
}

export function WebsiteSchema() {
  const websiteData = {
    '@context': 'https://schema.org',
    '@type': 'WebSite',
    name: 'Your Website Name',
    url: 'https://yoursite.com',
    description: 'Your website description',
    publisher: {
      '@type': 'Organization',
      name: 'Your Company Name'
    },
    potentialAction: {
      '@type': 'SearchAction',
      target: {
        '@type': 'EntryPoint',
        urlTemplate: 'https://yoursite.com/search?q={search_term_string}'
      },
      'query-input': 'required name=search_term_string'
    }
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(websiteData)
      }}
    />
  );
}

export function BreadcrumbSchema({ items }) {
  const breadcrumbData = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: item.url
    }))
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(breadcrumbData)
      }}
    />
  );
}

// app/layout.js - Add global structured data
import { OrganizationSchema, WebsiteSchema } from '@/components/StructuredData';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <OrganizationSchema />
        <WebsiteSchema />
      </body>
    </html>
  );
}

// components/Breadcrumbs.js - SEO-friendly breadcrumbs
import Link from 'next/link';
import { BreadcrumbSchema } from './StructuredData';

export function Breadcrumbs({ items }) {
  const breadcrumbItems = items.map(item => ({
    name: item.label,
    url: `https://yoursite.com${item.href}`
  }));

  return (
    <>
      <nav aria-label="Breadcrumb" className="mb-6">
        <ol className="flex items-center space-x-2 text-sm text-gray-600">
          {items.map((item, index) => (
            <li key={index} className="flex items-center">
              {index > 0 && (
                <svg className="w-4 h-4 mx-2 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
                  <path fillRule="evenodd" d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z" clipRule="evenodd" />
                </svg>
              )}
              {index === items.length - 1 ? (
                <span className="font-medium text-gray-900">{item.label}</span>
              ) : (
                <Link 
                  href={item.href}
                  className="hover:text-blue-600 transition-colors"
                >
                  {item.label}
                </Link>
              )}
            </li>
          ))}
        </ol>
      </nav>
      
      <BreadcrumbSchema items={breadcrumbItems} />
    </>
  );
}
```

## 🚀 Performance & Core Web Vitals

### **⚡ Image Optimization for SEO**

```jsx
// components/SEOImage.js
import Image from 'next/image';
import { useState } from 'react';

export function SEOImage({ 
  src, 
  alt, 
  width, 
  height, 
  priority = false,
  sizes,
  className,
  ...props 
}) {
  const [imageError, setImageError] = useState(false);

  if (imageError) {
    return (
      <div 
        className={`bg-gray-200 flex items-center justify-center ${className}`}
        style={{ width, height }}
      >
        <span className="text-gray-400 text-sm">Image not available</span>
      </div>
    );
  }

  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      priority={priority}
      sizes={sizes}
      className={className}
      onError={() => setImageError(true)}
      // ✅ SEO optimizations
      loading={priority ? 'eager' : 'lazy'}
      quality={85}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAIAAoDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAhEAACAQMDBQAAAAAAAAAAAAABAgMABAUGIWGRkbHB0f/EABUBAQEAAAAAAAAAAAAAAAAAAAMF/8QAGhEAAgIDAAAAAAAAAAAAAAAAAAECEgMRkf/aAAwDAQACEQMRAD8AltJagyeH0AthI5xdrLcNM91BF5pX2HaH9bcfaSXWGaRmknyJckliyjqTzSlT54b6bk+h0R//2Q=="
      {...props}
    />
  );
}

// components/ResponsiveImage.js
export function ResponsiveImage({ 
  src, 
  alt, 
  aspectRatio = '16/9',
  sizes = '(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw',
  priority = false,
  className = '',
  ...props 
}) {
  return (
    <div className={`relative overflow-hidden ${className}`} style={{ aspectRatio }}>
      <Image
        src={src}
        alt={alt}
        fill
        sizes={sizes}
        priority={priority}
        className="object-cover"
        quality={85}
        {...props}
      />
    </div>
  );
}

// components/ProductImageGallery.js
'use client';

import { useState } from 'react';
import { SEOImage } from './SEOImage';

export function ProductImageGallery({ images, productName }) {
  const [selectedImage, setSelectedImage] = useState(0);

  if (!images || images.length === 0) {
    return <div className="aspect-square bg-gray-200" />;
  }

  return (
    <div className="space-y-4">
      {/* Main image */}
      <div className="aspect-square relative overflow-hidden rounded-lg bg-gray-100">
        <SEOImage
          src={images[selectedImage].url}
          alt={`${productName} - Image ${selectedImage + 1}`}
          fill
          sizes="(max-width: 768px) 100vw, 50vw"
          priority={selectedImage === 0}
          className="object-cover"
        />
      </div>

      {/* Thumbnail grid */}
      {images.length > 1 && (
        <div className="grid grid-cols-4 gap-2">
          {images.map((image, index) => (
            <button
              key={index}
              onClick={() => setSelectedImage(index)}
              className={`aspect-square relative overflow-hidden rounded border-2 transition-colors ${
                selectedImage === index 
                  ? 'border-blue-500' 
                  : 'border-gray-200 hover:border-gray-300'
              }`}
            >
              <SEOImage
                src={image.url}
                alt={`${productName} - Thumbnail ${index + 1}`}
                fill
                sizes="150px"
                className="object-cover"
              />
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

### **📊 Core Web Vitals Optimization**

```javascript
// lib/analytics/webVitals.js
export function reportWebVitals(metric) {
  // ✅ Send to analytics
  if (typeof window !== 'undefined') {
    // Google Analytics 4
    window.gtag?.('event', metric.name, {
      event_category: 'Web Vitals',
      event_label: metric.id,
      value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
      non_interaction: true,
      custom_map: {
        metric_id: 'dimension1',
        metric_value: 'metric1',
        metric_delta: 'metric2'
      }
    });

    // Custom analytics
    fetch('/api/analytics/web-vitals', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        metric: metric.name,
        value: metric.value,
        id: metric.id,
        delta: metric.delta,
        navigationType: metric.navigationType,
        url: window.location.href,
        userAgent: navigator.userAgent,
        timestamp: Date.now()
      })
    }).catch(console.error);
  }
}

// app/layout.js - Web Vitals reporting
import { reportWebVitals } from '@/lib/analytics/webVitals';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        
        {/* ✅ Web Vitals reporting */}
        <script
          dangerouslySetInnerHTML={{
            __html: `
              if (typeof window !== 'undefined') {
                import('web-vitals').then(({ onCLS, onFID, onFCP, onLCP, onTTFB }) => {
                  onCLS(${reportWebVitals.toString()});
                  onFID(${reportWebVitals.toString()});
                  onFCP(${reportWebVitals.toString()});
                  onLCP(${reportWebVitals.toString()});
                  onTTFB(${reportWebVitals.toString()});
                });
              }
            `
          }}
        />
      </body>
    </html>
  );
}

// api/analytics/web-vitals/route.js - Store web vitals
export async function POST(request) {
  try {
    const data = await request.json();
    
    // ✅ Store in database
    await prisma.webVital.create({
      data: {
        metric: data.metric,
        value: data.value,
        metricId: data.id,
        delta: data.delta,
        navigationType: data.navigationType,
        url: data.url,
        userAgent: data.userAgent,
        timestamp: new Date(data.timestamp)
      }
    });

    return Response.json({ success: true });
  } catch (error) {
    console.error('Web Vitals error:', error);
    return Response.json({ error: 'Failed to record metric' }, { status: 500 });
  }
}

// components/PerformanceMonitor.js
'use client';

import { useEffect } from 'react';

export function PerformanceMonitor() {
  useEffect(() => {
    // ✅ Monitor Largest Contentful Paint
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.entryType === 'largest-contentful-paint') {
          console.log('LCP:', entry.startTime);
        }
      }
    });

    try {
      observer.observe({ entryTypes: ['largest-contentful-paint'] });
    } catch (error) {
      console.warn('PerformanceObserver not supported');
    }

    // ✅ Monitor Cumulative Layout Shift
    let clsValue = 0;
    const clsObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      }
    });

    try {
      clsObserver.observe({ entryTypes: ['layout-shift'] });
    } catch (error) {
      console.warn('Layout shift observer not supported');
    }

    // ✅ Report on page unload
    const handleBeforeUnload = () => {
      if (clsValue > 0) {
        navigator.sendBeacon('/api/analytics/cls', JSON.stringify({
          value: clsValue,
          url: window.location.href
        }));
      }
    };

    window.addEventListener('beforeunload', handleBeforeUnload);

    return () => {
      observer.disconnect();
      clsObserver.disconnect();
      window.removeEventListener('beforeunload', handleBeforeUnload);
    };
  }, []);

  return null;
}
```

## 🔎 Advanced SEO Features

### **🗺️ Sitemap Generation**

```javascript
// app/sitemap.js - Dynamic sitemap
import { prisma } from '@/lib/prisma';

export default async function sitemap() {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://yoursite.com';
  
  // ✅ Get dynamic content
  const [products, blogPosts, categories] = await Promise.all([
    prisma.product.findMany({
      where: { published: true },
      select: { slug: true, updatedAt: true }
    }),
    prisma.blogPost.findMany({
      where: { published: true },
      select: { slug: true, updatedAt: true, publishedAt: true }
    }),
    prisma.category.findMany({
      select: { slug: true, updatedAt: true }
    })
  ]);

  // ✅ Static pages
  const staticPages = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1
    },
    {
      url: `${baseUrl}/about`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8
    },
    {
      url: `${baseUrl}/contact`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.6
    },
    {
      url: `${baseUrl}/products`,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 0.9
    },
    {
      url: `${baseUrl}/blog`,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 0.9
    }
  ];

  // ✅ Product pages
  const productPages = products.map(product => ({
    url: `${baseUrl}/products/${product.slug}`,
    lastModified: product.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.8
  }));

  // ✅ Blog pages
  const blogPages = blogPosts.map(post => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'monthly',
    priority: 0.7
  }));

  // ✅ Category pages
  const categoryPages = categories.map(category => ({
    url: `${baseUrl}/categories/${category.slug}`,
    lastModified: category.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.6
  }));

  return [...staticPages, ...productPages, ...blogPages, ...categoryPages];
}

// app/robots.txt/route.js - Dynamic robots.txt
export async function GET() {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://yoursite.com';
  
  const robots = `User-agent: *
Allow: /

# Disallow admin and API routes
Disallow: /admin/
Disallow: /api/
Disallow: /_next/
Disallow: /search?

# Allow specific API routes for SEO
Allow: /api/og/

# Sitemap
Sitemap: ${baseUrl}/sitemap.xml

# Crawl delay
Crawl-delay: 1
`;

  return new Response(robots, {
    headers: {
      'Content-Type': 'text/plain'
    }
  });
}

// app/manifest.json/route.js - Dynamic PWA manifest
export async function GET() {
  const manifest = {
    name: 'Your App Name',
    short_name: 'YourApp',
    description: 'Your app description',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    orientation: 'portrait-primary',
    icons: [
      {
        src: '/icon-192x192.png',
        sizes: '192x192',
        type: 'image/png',
        purpose: 'maskable'
      },
      {
        src: '/icon-512x512.png',
        sizes: '512x512',
        type: 'image/png',
        purpose: 'maskable'
      }
    ],
    categories: ['shopping', 'business'],
    screenshots: [
      {
        src: '/screenshot-wide.png',
        sizes: '1280x720',
        type: 'image/png',
        form_factor: 'wide'
      },
      {
        src: '/screenshot-narrow.png',
        sizes: '750x1334',
        type: 'image/png',
        form_factor: 'narrow'
      }
    ]
  };

  return Response.json(manifest);
}
```

### **🎨 Dynamic OG Image Generation**

```javascript
// app/api/og/route.js - Dynamic Open Graph image generation
import { ImageResponse } from 'next/og';

export async function GET(request) {
  try {
    const { searchParams } = new URL(request.url);
    
    const title = searchParams.get('title') || 'Default Title';
    const description = searchParams.get('description') || 'Default Description';
    const image = searchParams.get('image');
    const type = searchParams.get('type') || 'default';

    // ✅ Load custom font
    const interSemiBold = fetch(
      new URL('../../../assets/fonts/Inter-SemiBold.ttf', import.meta.url)
    ).then((res) => res.arrayBuffer());

    return new ImageResponse(
      (
        <div
          style={{
            height: '100%',
            width: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
            backgroundColor: 'white',
            backgroundImage: 'linear-gradient(45deg, #667eea 0%, #764ba2 100%)'
          }}
        >
          {/* Background pattern */}
          <div
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              right: 0,
              bottom: 0,
              opacity: 0.1,
              backgroundImage: 'url("data:image/svg+xml,%3Csvg width="60" height="60" viewBox="0 0 60 60" xmlns="http://www.w3.org/2000/svg"%3E%3Cg fill="none" fill-rule="evenodd"%3E%3Cg fill="%23ffffff" fill-opacity="0.4"%3E%3Ccircle cx="30" cy="30" r="4"/%3E%3C/g%3E%3C/g%3E%3C/svg%3E")'
            }}
          />
          
          <div
            style={{
              display: 'flex',
              flexDirection: 'column',
              alignItems: 'center',
              justifyContent: 'center',
              padding: '40px',
              textAlign: 'center',
              maxWidth: '900px'
            }}
          >
            {/* Logo or brand */}
            <div
              style={{
                fontSize: 32,
                fontWeight: 'bold',
                color: 'white',
                marginBottom: 20,
                opacity: 0.9
              }}
            >
              YourBrand
            </div>

            {/* Title */}
            <div
              style={{
                fontSize: type === 'product' ? 48 : 56,
                fontWeight: 'bold',
                color: 'white',
                lineHeight: 1.2,
                marginBottom: 16,
                textShadow: '0 2px 4px rgba(0,0,0,0.3)'
              }}
            >
              {title}
            </div>

            {/* Description */}
            {description && (
              <div
                style={{
                  fontSize: 24,
                  color: 'rgba(255,255,255,0.9)',
                  lineHeight: 1.4,
                  maxWidth: '600px',
                  textShadow: '0 1px 2px rgba(0,0,0,0.3)'
                }}
              >
                {description}
              </div>
            )}

            {/* Product image if available */}
            {type === 'product' && image && (
              <div
                style={{
                  marginTop: 30,
                  borderRadius: 12,
                  overflow: 'hidden',
                  boxShadow: '0 10px 30px rgba(0,0,0,0.3)'
                }}
              >
                <img
                  src={image}
                  alt="Product"
                  width={200}
                  height={200}
                  style={{ objectFit: 'cover' }}
                />
              </div>
            )}
          </div>
        </div>
      ),
      {
        width: 1200,
        height: 630,
        fonts: [
          {
            name: 'Inter',
            data: await interSemiBold,
            style: 'normal',
            weight: 600
          }
        ]
      }
    );
  } catch (error) {
    console.error('OG Image generation failed:', error);
    return new Response('Failed to generate image', { status: 500 });
  }
}

// Use in metadata
export async function generateMetadata({ params }) {
  const product = await getProduct(params.slug);
  
  const ogImageUrl = new URL('/api/og', process.env.NEXT_PUBLIC_BASE_URL);
  ogImageUrl.searchParams.set('title', product.name);
  ogImageUrl.searchParams.set('description', product.description);
  ogImageUrl.searchParams.set('image', product.images[0]?.url);
  ogImageUrl.searchParams.set('type', 'product');

  return {
    title: product.name,
    openGraph: {
      images: [
        {
          url: ogImageUrl.toString(),
          width: 1200,
          height: 630,
          alt: product.name
        }
      ]
    }
  };
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic SEO Setup**
1. เพิ่ม metadata API ใน layout และ pages
2. สร้าง Open Graph images
3. เพิ่ม structured data
4. ทดสอบด้วย Google Search Console

### **🎯 แบบฝึกหัดที่ 2: Advanced SEO Features**
1. สร้าง dynamic sitemap
2. เพิ่ม breadcrumb navigation
3. สร้าง robots.txt และ manifest.json
4. ทดสอบ rich snippets

### **🎯 แบบฝึกหัดที่ 3: Performance Optimization**
1. เพิ่ม Core Web Vitals monitoring
2. ปรับปรุง image optimization
3. เพิ่ม performance budgets
4. วัด SEO impact

### **🎯 แบบฝึกหัดที่ 4: E-commerce SEO**
1. เพิ่ม product schema markup
2. สร้าง category และ tag pages
3. เพิ่ม review และ rating schema
4. ทดสอบ rich snippets สำหรับ products

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 17: Internationalization (i18n)](./17-internationalization.md)

**บทต่อไป**: [บทที่ 19: Testing](./19-testing.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Metadata API](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
- [Structured Data](https://developers.google.com/search/docs/guides/intro-structured-data)
- [Core Web Vitals](https://web.dev/vitals/)
- [Open Graph Protocol](https://ogp.me/)
