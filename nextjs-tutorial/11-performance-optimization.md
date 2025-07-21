# บทที่ 11: Performance Optimization

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Performance optimization techniques
- ทำความเข้าใจ Caching strategies
- เรียนรู้ Image optimization
- เข้าใจ Code splitting และ Bundle optimization

## ⚡ Performance Overview

### **📊 Performance Metrics**

```javascript
// ✅ Core Web Vitals Monitoring
export function useWebVitals() {
  useEffect(() => {
    function sendToAnalytics(metric) {
      const body = JSON.stringify(metric);
      
      // ✅ Use navigator.sendBeacon() if available
      if (navigator.sendBeacon) {
        navigator.sendBeacon('/api/analytics', body);
      } else {
        // ✅ Fallback to fetch()
        fetch('/api/analytics', {
          body,
          method: 'POST',
          keepalive: true,
        });
      }
    }

    // ✅ Cumulative Layout Shift (CLS)
    function onCLS(metric) {
      console.log('CLS:', metric);
      sendToAnalytics({
        name: 'CLS',
        value: metric.value,
        id: metric.id,
        url: window.location.href,
        timestamp: Date.now()
      });
    }

    // ✅ First Input Delay (FID)
    function onFID(metric) {
      console.log('FID:', metric);
      sendToAnalytics({
        name: 'FID',
        value: metric.value,
        id: metric.id,
        url: window.location.href,
        timestamp: Date.now()
      });
    }

    // ✅ Largest Contentful Paint (LCP)
    function onLCP(metric) {
      console.log('LCP:', metric);
      sendToAnalytics({
        name: 'LCP',
        value: metric.value,
        id: metric.id,
        url: window.location.href,
        timestamp: Date.now()
      });
    }

    // ✅ First Contentful Paint (FCP)
    function onFCP(metric) {
      console.log('FCP:', metric);
      sendToAnalytics({
        name: 'FCP',
        value: metric.value,
        id: metric.id,
        url: window.location.href,
        timestamp: Date.now()
      });
    }

    // ✅ Time to First Byte (TTFB)
    function onTTFB(metric) {
      console.log('TTFB:', metric);
      sendToAnalytics({
        name: 'TTFB',
        value: metric.value,
        id: metric.id,
        url: window.location.href,
        timestamp: Date.now()
      });
    }

    // ✅ Load web-vitals library dynamically
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(onCLS);
      getFID(onFID);
      getFCP(onFCP);
      getLCP(onLCP);
      getTTFB(onTTFB);
    });
  }, []);
}

// pages/_app.js
import { useWebVitals } from '@/hooks/useWebVitals';

export default function MyApp({ Component, pageProps }) {
  useWebVitals();

  return <Component {...pageProps} />;
}

// ✅ Custom performance monitoring
export function reportWebVitals(metric) {
  console.log(metric);
  
  // ✅ Send metrics to analytics service
  if (process.env.NODE_ENV === 'production') {
    // Google Analytics 4
    if (window.gtag) {
      window.gtag('event', metric.name, {
        custom_parameter_1: metric.value,
        custom_parameter_2: metric.id,
        custom_parameter_3: metric.label,
      });
    }

    // ✅ Custom analytics endpoint
    fetch('/api/analytics/web-vitals', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        metric: metric.name,
        value: metric.value,
        id: metric.id,
        label: metric.label,
        url: window.location.href,
        timestamp: Date.now(),
        userAgent: navigator.userAgent,
        connection: navigator.connection?.effectiveType,
      }),
    });
  }
}
```

## 🗄️ Caching Strategies

### **💾 Server-Side Caching**

```javascript
// lib/cache.js
import Redis from 'ioredis';

class CacheService {
  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: process.env.REDIS_PORT || 6379,
      password: process.env.REDIS_PASSWORD,
      db: process.env.REDIS_DB || 0,
      retryDelayOnFailover: 100,
      enableReadyCheck: true,
      lazyConnect: true,
    });

    this.defaultTTL = 3600; // 1 hour
  }

  // ✅ Set cache with TTL
  async set(key, value, ttl = this.defaultTTL) {
    try {
      const serialized = JSON.stringify(value);
      if (ttl) {
        await this.redis.setex(key, ttl, serialized);
      } else {
        await this.redis.set(key, serialized);
      }
      return true;
    } catch (error) {
      console.error('Cache set error:', error);
      return false;
    }
  }

  // ✅ Get cache
  async get(key) {
    try {
      const value = await this.redis.get(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }

  // ✅ Delete cache
  async del(key) {
    try {
      await this.redis.del(key);
      return true;
    } catch (error) {
      console.error('Cache delete error:', error);
      return false;
    }
  }

  // ✅ Get or set pattern
  async getOrSet(key, fetcher, ttl = this.defaultTTL) {
    try {
      // Try to get from cache first
      let value = await this.get(key);
      
      if (value === null) {
        // If not in cache, fetch and set
        value = await fetcher();
        await this.set(key, value, ttl);
      }
      
      return value;
    } catch (error) {
      console.error('Cache getOrSet error:', error);
      // If cache fails, still return the fetched value
      return await fetcher();
    }
  }

  // ✅ Invalidate pattern
  async invalidatePattern(pattern) {
    try {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        await this.redis.del(...keys);
      }
      return true;
    } catch (error) {
      console.error('Cache invalidate pattern error:', error);
      return false;
    }
  }

  // ✅ Multi-level cache with memory backup
  async getWithFallback(key, fetcher, ttl = this.defaultTTL) {
    try {
      // Level 1: Memory cache (if available)
      if (this.memoryCache?.has(key)) {
        return this.memoryCache.get(key);
      }

      // Level 2: Redis cache
      let value = await this.get(key);
      
      if (value === null) {
        // Level 3: Data source
        value = await fetcher();
        
        // Store in both caches
        await this.set(key, value, ttl);
        this.memoryCache?.set(key, value);
      } else {
        // Store in memory cache for faster access
        this.memoryCache?.set(key, value);
      }
      
      return value;
    } catch (error) {
      console.error('Cache fallback error:', error);
      return await fetcher();
    }
  }
}

export const cache = new CacheService();

// ✅ Cache middleware for API routes
export function withCache(options = {}) {
  const {
    ttl = 3600,
    keyGenerator = (req) => `api:${req.url}`,
    skipCache = false,
    revalidate = false
  } = options;

  return function (handler) {
    return async function (req, res) {
      if (skipCache || req.method !== 'GET') {
        return handler(req, res);
      }

      const cacheKey = typeof keyGenerator === 'function' 
        ? keyGenerator(req) 
        : keyGenerator;

      try {
        // ✅ Check cache first
        if (!revalidate) {
          const cached = await cache.get(cacheKey);
          if (cached) {
            res.setHeader('X-Cache', 'HIT');
            return res.status(200).json(cached);
          }
        }

        // ✅ Capture response
        const originalJson = res.json;
        let responseData;

        res.json = function(data) {
          responseData = data;
          return originalJson.call(this, data);
        };

        // ✅ Call original handler
        await handler(req, res);

        // ✅ Cache the response if successful
        if (res.statusCode === 200 && responseData) {
          await cache.set(cacheKey, responseData, ttl);
          res.setHeader('X-Cache', 'MISS');
        }

      } catch (error) {
        console.error('Cache middleware error:', error);
        return handler(req, res);
      }
    };
  };
}

// pages/api/posts/index.js
import { withCache } from '@/lib/cache';
import { prisma } from '@/lib/prisma';

async function handler(req, res) {
  try {
    const posts = await prisma.post.findMany({
      include: {
        author: {
          select: { id: true, name: true, avatar: true }
        },
        _count: {
          select: { comments: true, likes: true }
        }
      },
      orderBy: { createdAt: 'desc' },
      take: 20
    });

    res.status(200).json({ posts });
  } catch (error) {
    res.status(500).json({ message: 'Failed to fetch posts' });
  }
}

export default withCache({
  ttl: 300, // 5 minutes
  keyGenerator: (req) => {
    const { page = 1, limit = 20 } = req.query;
    return `posts:list:${page}:${limit}`;
  }
})(handler);
```

### **🌐 Client-Side Caching**

```javascript
// lib/clientCache.js
class ClientCacheService {
  constructor() {
    this.cache = new Map();
    this.timers = new Map();
    this.defaultTTL = 5 * 60 * 1000; // 5 minutes
  }

  // ✅ Set with automatic cleanup
  set(key, value, ttl = this.defaultTTL) {
    // Clear existing timer
    if (this.timers.has(key)) {
      clearTimeout(this.timers.get(key));
    }

    // Set value
    this.cache.set(key, {
      value,
      timestamp: Date.now(),
      ttl
    });

    // Set cleanup timer
    const timer = setTimeout(() => {
      this.delete(key);
    }, ttl);

    this.timers.set(key, timer);
  }

  // ✅ Get with expiry check
  get(key) {
    const cached = this.cache.get(key);
    
    if (!cached) return null;

    const now = Date.now();
    if (now - cached.timestamp > cached.ttl) {
      this.delete(key);
      return null;
    }

    return cached.value;
  }

  // ✅ Delete
  delete(key) {
    this.cache.delete(key);
    
    const timer = this.timers.get(key);
    if (timer) {
      clearTimeout(timer);
      this.timers.delete(key);
    }
  }

  // ✅ Clear all
  clear() {
    this.cache.clear();
    this.timers.forEach(timer => clearTimeout(timer));
    this.timers.clear();
  }

  // ✅ Get cache stats
  getStats() {
    return {
      size: this.cache.size,
      keys: Array.from(this.cache.keys())
    };
  }
}

export const clientCache = new ClientCacheService();

// hooks/useCache.js
import { useState, useEffect, useCallback } from 'react';
import { clientCache } from '@/lib/clientCache';

export function useCache(key, fetcher, options = {}) {
  const {
    ttl = 5 * 60 * 1000, // 5 minutes
    staleWhileRevalidate = false,
    revalidateOnFocus = true,
    enabled = true
  } = options;

  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async (force = false) => {
    if (!enabled) return;

    try {
      setLoading(true);
      setError(null);

      // ✅ Check cache first
      if (!force) {
        const cached = clientCache.get(key);
        if (cached) {
          setData(cached);
          setLoading(false);
          
          // ✅ Return early if not using stale-while-revalidate
          if (!staleWhileRevalidate) {
            return cached;
          }
        }
      }

      // ✅ Fetch fresh data
      const freshData = await fetcher();
      
      // ✅ Update cache and state
      clientCache.set(key, freshData, ttl);
      setData(freshData);
      
      return freshData;
    } catch (err) {
      setError(err);
      console.error('Cache fetch error:', err);
    } finally {
      setLoading(false);
    }
  }, [key, fetcher, enabled, ttl, staleWhileRevalidate]);

  // ✅ Initial fetch
  useEffect(() => {
    fetchData();
  }, [fetchData]);

  // ✅ Revalidate on focus
  useEffect(() => {
    if (!revalidateOnFocus) return;

    const handleFocus = () => {
      fetchData();
    };

    window.addEventListener('focus', handleFocus);
    return () => window.removeEventListener('focus', handleFocus);
  }, [fetchData, revalidateOnFocus]);

  // ✅ Manual revalidate function
  const revalidate = useCallback(() => {
    return fetchData(true);
  }, [fetchData]);

  // ✅ Mutate cache
  const mutate = useCallback((newData) => {
    clientCache.set(key, newData, ttl);
    setData(newData);
  }, [key, ttl]);

  return {
    data,
    loading,
    error,
    revalidate,
    mutate,
    isValidating: loading
  };
}

// ✅ SWR-like hook with cache
export function useSWR(key, fetcher, options = {}) {
  return useCache(key, fetcher, {
    staleWhileRevalidate: true,
    ...options
  });
}
```

## 🖼️ Image Optimization

### **📷 Next.js Image Component**

```jsx
// components/OptimizedImage.js
import Image from 'next/image';
import { useState } from 'react';

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  quality = 75,
  placeholder = 'blur',
  blurDataURL,
  sizes,
  fill = false,
  className = '',
  ...props
}) {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  // ✅ Generate blur data URL for placeholder
  const generateBlurDataURL = (w = 8, h = 8) => {
    const canvas = document.createElement('canvas');
    canvas.width = w;
    canvas.height = h;
    const ctx = canvas.getContext('2d');
    
    // Create gradient
    const gradient = ctx.createLinearGradient(0, 0, w, h);
    gradient.addColorStop(0, '#f3f4f6');
    gradient.addColorStop(1, '#e5e7eb');
    
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, w, h);
    
    return canvas.toDataURL();
  };

  const defaultBlurDataURL = "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAIAAoDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAhEAACAQMDBQAAAAAAAAAAAAABAgMABAUGIWEREiMxUf/EABUBAQEAAAAAAAAAAAAAAAAAAAMF/8QAGhEAAgIDAAAAAAAAAAAAAAAAAAECEgMRkf/aAAwDAQACEQMRAD8AltJagyeH0AthI5xdrLcNM91BF5pX2HaH9bcfaSXWGaRmknyJckliyjqTzSlT54b6bk+h0R//2Q==";

  return (
    <div className={`relative overflow-hidden ${className}`}>
      {/* Loading skeleton */}
      {loading && (
        <div className="absolute inset-0 bg-gray-200 animate-pulse flex items-center justify-center">
          <svg className="w-8 h-8 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
            <path fillRule="evenodd" d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z" clipRule="evenodd" />
          </svg>
        </div>
      )}

      {/* Error fallback */}
      {error ? (
        <div className="w-full h-full bg-gray-100 flex items-center justify-center text-gray-500">
          <span>Failed to load image</span>
        </div>
      ) : (
        <Image
          src={src}
          alt={alt}
          width={width}
          height={height}
          fill={fill}
          priority={priority}
          quality={quality}
          placeholder={placeholder}
          blurDataURL={blurDataURL || defaultBlurDataURL}
          sizes={sizes}
          className={`transition-opacity duration-300 ${loading ? 'opacity-0' : 'opacity-100'}`}
          onLoad={() => setLoading(false)}
          onError={() => {
            setLoading(false);
            setError(true);
          }}
          {...props}
        />
      )}
    </div>
  );
}

// ✅ Responsive image with multiple sizes
export function ResponsiveImage({
  src,
  alt,
  aspectRatio = '16/9',
  sizes = '(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw',
  priority = false,
  className = ''
}) {
  return (
    <div 
      className={`relative w-full ${className}`}
      style={{ aspectRatio }}
    >
      <OptimizedImage
        src={src}
        alt={alt}
        fill
        sizes={sizes}
        priority={priority}
        className="object-cover"
      />
    </div>
  );
}

// ✅ Image gallery with lazy loading
export function ImageGallery({ images = [] }) {
  const [visibleImages, setVisibleImages] = useState(new Set());

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const index = parseInt(entry.target.dataset.index);
            setVisibleImages(prev => new Set([...prev, index]));
          }
        });
      },
      { rootMargin: '50px' }
    );

    const imageElements = document.querySelectorAll('[data-image-index]');
    imageElements.forEach(el => observer.observe(el));

    return () => observer.disconnect();
  }, [images]);

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {images.map((image, index) => (
        <div
          key={image.id}
          data-index={index}
          data-image-index
          className="aspect-square relative overflow-hidden rounded-lg"
        >
          {visibleImages.has(index) ? (
            <OptimizedImage
              src={image.src}
              alt={image.alt}
              fill
              sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
              className="object-cover hover:scale-105 transition-transform duration-300"
            />
          ) : (
            <div className="w-full h-full bg-gray-200 animate-pulse" />
          )}
        </div>
      ))}
    </div>
  );
}
```

### **⚡ Advanced Image Optimization**

```javascript
// lib/imageOptimization.js
export class ImageOptimizer {
  static async generateResponsiveImages(src, sizes = [400, 800, 1200, 1600]) {
    const images = [];

    for (const size of sizes) {
      try {
        // ✅ Generate different sizes
        const optimizedSrc = await this.resizeImage(src, size);
        images.push({
          src: optimizedSrc,
          width: size,
          descriptor: `${size}w`
        });
      } catch (error) {
        console.error(`Failed to generate ${size}px version:`, error);
      }
    }

    return images;
  }

  static async resizeImage(src, width, quality = 80) {
    // ✅ Use next/image API or external service
    const params = new URLSearchParams({
      url: src,
      w: width.toString(),
      q: quality.toString()
    });

    return `/api/images/resize?${params.toString()}`;
  }

  static generateSrcSet(images) {
    return images
      .map(img => `${img.src} ${img.descriptor}`)
      .join(', ');
  }

  static generateSizes(breakpoints = {
    sm: '100vw',
    md: '50vw', 
    lg: '33vw'
  }) {
    const entries = Object.entries(breakpoints);
    const sizeQueries = entries
      .slice(0, -1)
      .map(([breakpoint, size]) => `(max-width: ${this.getBreakpointValue(breakpoint)}) ${size}`)
      .join(', ');
    
    const defaultSize = entries[entries.length - 1][1];
    return sizeQueries ? `${sizeQueries}, ${defaultSize}` : defaultSize;
  }

  static getBreakpointValue(breakpoint) {
    const breakpoints = {
      sm: '640px',
      md: '768px',
      lg: '1024px',
      xl: '1280px'
    };
    return breakpoints[breakpoint] || '768px';
  }

  // ✅ Progressive JPEG/WebP conversion
  static async convertToModernFormats(src) {
    const formats = ['webp', 'avif'];
    const conversions = {};

    for (const format of formats) {
      try {
        conversions[format] = await this.convertFormat(src, format);
      } catch (error) {
        console.error(`Failed to convert to ${format}:`, error);
      }
    }

    return conversions;
  }

  static async convertFormat(src, format) {
    const params = new URLSearchParams({
      url: src,
      format,
      q: '80'
    });

    return `/api/images/convert?${params.toString()}`;
  }
}

// pages/api/images/resize.js
import sharp from 'sharp';

export default async function handler(req, res) {
  const { url, w, q = 80 } = req.query;

  if (!url) {
    return res.status(400).json({ error: 'URL is required' });
  }

  try {
    const width = parseInt(w);
    const quality = parseInt(q);

    // ✅ Set cache headers
    res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
    res.setHeader('Content-Type', 'image/jpeg');

    // ✅ Fetch original image
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('Failed to fetch image');
    }

    const buffer = await response.arrayBuffer();

    // ✅ Resize and optimize with Sharp
    const optimized = await sharp(Buffer.from(buffer))
      .resize(width, null, {
        withoutEnlargement: true,
        fit: 'inside'
      })
      .jpeg({ quality, progressive: true })
      .toBuffer();

    res.send(optimized);
  } catch (error) {
    console.error('Image resize error:', error);
    res.status(500).json({ error: 'Failed to resize image' });
  }
}
```

## 📦 Code Splitting & Bundle Optimization

### **⚡ Dynamic Imports**

```javascript
// components/LazyComponents.js
import { lazy, Suspense, useState, useEffect } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// ✅ Lazy load components
const DashboardChart = lazy(() => 
  import('./DashboardChart').then(module => ({
    default: module.DashboardChart
  }))
);

const DataTable = lazy(() => 
  import('./DataTable')
);

const VideoPlayer = lazy(() => 
  import('./VideoPlayer').then(module => ({
    default: module.VideoPlayer
  }))
);

// ✅ Loading components
function ChartSkeleton() {
  return (
    <div className="w-full h-64 bg-gray-200 animate-pulse rounded-lg flex items-center justify-center">
      <div className="text-gray-400">Loading chart...</div>
    </div>
  );
}

function TableSkeleton() {
  return (
    <div className="space-y-2">
      {[...Array(5)].map((_, i) => (
        <div key={i} className="h-12 bg-gray-200 animate-pulse rounded" />
      ))}
    </div>
  );
}

// ✅ Error fallback
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div className="p-4 bg-red-50 border border-red-200 rounded-lg">
      <h3 className="text-red-800 font-medium">Something went wrong</h3>
      <p className="text-red-600 text-sm mt-1">{error.message}</p>
      <button 
        onClick={resetErrorBoundary}
        className="mt-2 px-3 py-1 bg-red-600 text-white text-sm rounded hover:bg-red-700"
      >
        Try again
      </button>
    </div>
  );
}

// ✅ Intersection Observer for lazy loading
function LazySection({ children, threshold = 0.1 }) {
  const [isVisible, setIsVisible] = useState(false);
  const [ref, setRef] = useState(null);

  useEffect(() => {
    if (!ref) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold }
    );

    observer.observe(ref);
    return () => observer.disconnect();
  }, [ref, threshold]);

  return (
    <div ref={setRef}>
      {isVisible ? children : <div className="h-64" />}
    </div>
  );
}

// ✅ Main component with lazy loading
export function Dashboard() {
  const [activeTab, setActiveTab] = useState('overview');

  return (
    <div className="space-y-6">
      {/* Tab navigation */}
      <div className="border-b border-gray-200">
        <nav className="-mb-px flex space-x-8">
          {['overview', 'analytics', 'data', 'media'].map((tab) => (
            <button
              key={tab}
              onClick={() => setActiveTab(tab)}
              className={`py-2 px-1 border-b-2 font-medium text-sm ${
                activeTab === tab
                  ? 'border-indigo-500 text-indigo-600'
                  : 'border-transparent text-gray-500 hover:text-gray-700'
              }`}
            >
              {tab.charAt(0).toUpperCase() + tab.slice(1)}
            </button>
          ))}
        </nav>
      </div>

      {/* Tab content with lazy loading */}
      <ErrorBoundary FallbackComponent={ErrorFallback}>
        {activeTab === 'overview' && (
          <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <LazySection>
              <Suspense fallback={<ChartSkeleton />}>
                <DashboardChart type="line" />
              </Suspense>
            </LazySection>
            
            <LazySection>
              <Suspense fallback={<ChartSkeleton />}>
                <DashboardChart type="bar" />
              </Suspense>
            </LazySection>
          </div>
        )}

        {activeTab === 'data' && (
          <LazySection>
            <Suspense fallback={<TableSkeleton />}>
              <DataTable />
            </Suspense>
          </LazySection>
        )}

        {activeTab === 'media' && (
          <LazySection>
            <Suspense fallback={<div className="h-64 bg-gray-200 animate-pulse" />}>
              <VideoPlayer />
            </Suspense>
          </LazySection>
        )}
      </ErrorBoundary>
    </div>
  );
}

// ✅ Dynamic import with retry logic
export async function importWithRetry(importFn, retries = 3) {
  try {
    return await importFn();
  } catch (error) {
    if (retries > 0) {
      console.warn(`Import failed, retrying... (${retries} attempts left)`);
      await new Promise(resolve => setTimeout(resolve, 1000));
      return importWithRetry(importFn, retries - 1);
    }
    throw error;
  }
}

// ✅ Preload critical components
export function preloadComponents() {
  // Preload components that are likely to be needed
  import('./DashboardChart');
  import('./DataTable');
}

// ✅ Route-based code splitting
export function useRoutePreloading() {
  useEffect(() => {
    const links = document.querySelectorAll('a[href]');
    
    const preloadRoute = (href) => {
      // Preload the route component
      if (href.startsWith('/dashboard')) {
        import('../pages/dashboard');
      } else if (href.startsWith('/profile')) {
        import('../pages/profile');
      }
    };

    const handleMouseEnter = (e) => {
      const href = e.target.getAttribute('href');
      if (href && href.startsWith('/')) {
        preloadRoute(href);
      }
    };

    links.forEach(link => {
      link.addEventListener('mouseenter', handleMouseEnter);
    });

    return () => {
      links.forEach(link => {
        link.removeEventListener('mouseenter', handleMouseEnter);
      });
    };
  }, []);
}
```

### **📊 Bundle Analysis**

```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

const nextConfig = {
  // ✅ Webpack optimization
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // ✅ Bundle optimization
    if (!dev) {
      config.optimization = {
        ...config.optimization,
        splitChunks: {
          chunks: 'all',
          cacheGroups: {
            // ✅ Vendor chunk
            vendor: {
              test: /[\\/]node_modules[\\/]/,
              name: 'vendors',
              chunks: 'all',
              priority: 10,
            },
            // ✅ Common chunk
            common: {
              minChunks: 2,
              name: 'common',
              chunks: 'all',
              priority: 5,
              reuseExistingChunk: true,
            },
            // ✅ React chunk
            react: {
              test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
              name: 'react',
              chunks: 'all',
              priority: 20,
            },
            // ✅ UI library chunk
            ui: {
              test: /[\\/]node_modules[\\/](@headlessui|@heroicons|framer-motion)[\\/]/,
              name: 'ui',
              chunks: 'all',
              priority: 15,
            }
          }
        }
      };
    }

    // ✅ Tree shaking for specific libraries
    config.resolve.alias = {
      ...config.resolve.alias,
      'lodash': 'lodash-es',
    };

    // ✅ Ignore unused imports
    config.plugins.push(
      new webpack.IgnorePlugin({
        resourceRegExp: /^\.\/locale$/,
        contextRegExp: /moment$/,
      })
    );

    return config;
  },

  // ✅ Experimental features
  experimental: {
    optimizeCss: true,
    swcMinify: true,
    legacyBrowsers: false,
    browsersListForSwc: true,
  },

  // ✅ Compression
  compress: true,

  // ✅ Static optimization
  staticPageGenerationTimeout: 1000,

  // ✅ Image optimization
  images: {
    formats: ['image/avif', 'image/webp'],
    minimumCacheTTL: 60 * 60 * 24 * 30, // 30 days
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
};

module.exports = withBundleAnalyzer(nextConfig);

// ✅ Bundle analysis script
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true npm run build",
    "bundle-stats": "npx webpack-bundle-analyzer .next/static/chunks/",
    "lighthouse": "lighthouse http://localhost:3000 --output html --output-path ./lighthouse-report.html",
    "speed-test": "npm run build && npm run start & sleep 5 && npm run lighthouse; kill %1"
  }
}

// ✅ Performance monitoring
// lib/performance.js
export class PerformanceMonitor {
  static measureComponentRender(Component) {
    return function MeasuredComponent(props) {
      const renderStart = performance.now();
      
      useEffect(() => {
        const renderEnd = performance.now();
        const renderTime = renderEnd - renderStart;
        
        console.log(`${Component.name || 'Component'} render time: ${renderTime.toFixed(2)}ms`);
        
        // ✅ Send to analytics
        if (process.env.NODE_ENV === 'production' && window.gtag) {
          window.gtag('event', 'component_render_time', {
            component_name: Component.name,
            render_time: renderTime,
          });
        }
      });

      return <Component {...props} />;
    };
  }

  static measureAsyncOperation(operationName, asyncFn) {
    return async function(...args) {
      const start = performance.now();
      
      try {
        const result = await asyncFn(...args);
        const end = performance.now();
        
        console.log(`${operationName} completed in ${(end - start).toFixed(2)}ms`);
        
        return result;
      } catch (error) {
        const end = performance.now();
        console.error(`${operationName} failed after ${(end - start).toFixed(2)}ms:`, error);
        throw error;
      }
    };
  }

  static trackResourceLoading() {
    if (typeof window === 'undefined') return;

    window.addEventListener('load', () => {
      // ✅ Get navigation timing
      const navigation = performance.getEntriesByType('navigation')[0];
      
      console.log('Page Load Performance:', {
        DNS: navigation.domainLookupEnd - navigation.domainLookupStart,
        TCP: navigation.connectEnd - navigation.connectStart,
        Request: navigation.responseStart - navigation.requestStart,
        Response: navigation.responseEnd - navigation.responseStart,
        DOM: navigation.domContentLoadedEventEnd - navigation.responseEnd,
        Load: navigation.loadEventEnd - navigation.loadEventStart,
        Total: navigation.loadEventEnd - navigation.navigationStart
      });

      // ✅ Get resource timing
      const resources = performance.getEntriesByType('resource');
      const resourceStats = resources.reduce((acc, resource) => {
        const type = resource.initiatorType;
        if (!acc[type]) acc[type] = { count: 0, totalSize: 0, totalTime: 0 };
        
        acc[type].count++;
        acc[type].totalSize += resource.transferSize || 0;
        acc[type].totalTime += resource.duration;
        
        return acc;
      }, {});

      console.log('Resource Loading Stats:', resourceStats);
    });
  }
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Caching Implementation**
1. ติดตั้งและ configure Redis caching
2. เพิ่ม API route caching
3. สร้าง client-side cache system
4. ทดสอบ cache performance

### **🎯 แบบฝึกหัดที่ 2: Image Optimization**
1. เพิ่ม responsive image component
2. ติดตั้ง modern image formats
3. สร้าง lazy loading gallery
4. วัด image loading performance

### **🎯 แบบฝึกหัดที่ 3: Code Splitting**
1. เพิ่ม dynamic imports
2. สร้าง lazy-loaded components
3. ปรับปรุง bundle splitting
4. วิเคราะห์ bundle size

### **🎯 แบบฝึกหัดที่ 4: Performance Monitoring**
1. เพิ่ม Web Vitals tracking
2. สร้าง performance dashboard
3. ติดตั้ง lighthouse CI
4. ปรับปรุง performance scores

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 10: Authentication](./10-authentication.md)

**บทต่อไป**: [บทที่ 12: Forms and Validation](./12-forms-validation.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Performance](https://nextjs.org/docs/advanced-features/measuring-performance)
- [Web Vitals](https://web.dev/vitals/)
- [Image Optimization](https://nextjs.org/docs/basic-features/image-optimization)
- [Bundle Analyzer](https://www.npmjs.com/package/@next/bundle-analyzer)
