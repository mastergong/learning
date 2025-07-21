# บทที่ 24: Performance Optimization

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้เทคนิค performance optimization ขั้นสูง
- ทำความเข้าใจ bundle optimization และ code splitting
- เรียนรู้ caching strategies และ CDN optimization
- เข้าใจ image และ asset optimization

## ⚡ Core Performance Optimizations

### **🚀 Bundle Optimization**

```javascript
// next.config.js - Advanced bundle configuration
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

const nextConfig = {
  // Experimental features for performance
  experimental: {
    appDir: true,
    serverActions: true,
    serverComponentsExternalPackages: ['@prisma/client'],
    optimizeCss: true,
    optimizePackageImports: ['lucide-react', 'lodash'],
  },

  // Compiler optimizations
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
    reactRemoveProperties: process.env.NODE_ENV === 'production',
  },

  // Bundle optimization
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Bundle splitting optimization
    if (!isServer && !dev) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          // Vendor chunks
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            priority: 10,
            reuseExistingChunk: true,
          },
          // React chunk
          react: {
            test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
            name: 'react',
            priority: 20,
            reuseExistingChunk: true,
          },
          // UI library chunk
          ui: {
            test: /[\\/]node_modules[\\/](@headlessui|@heroicons|lucide-react)[\\/]/,
            name: 'ui',
            priority: 15,
            reuseExistingChunk: true,
          },
          // Common chunk
          common: {
            minChunks: 2,
            priority: 5,
            reuseExistingChunk: true,
          },
        },
      };
    }

    // Tree shaking optimization
    config.optimization.usedExports = true;
    config.optimization.sideEffects = false;

    // Module resolution optimization
    config.resolve.alias = {
      ...config.resolve.alias,
      '@': path.resolve(__dirname, 'src'),
    };

    // Bundle analysis in development
    if (dev && !isServer) {
      config.plugins.push(
        new webpack.DefinePlugin({
          'process.env.BUNDLE_ANALYZE': JSON.stringify(process.env.ANALYZE === 'true'),
        })
      );
    }

    return config;
  },

  // Output optimization
  output: 'standalone',
  
  // Image optimization
  images: {
    domains: ['localhost', 'cdn.example.com'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    formats: ['image/webp', 'image/avif'],
    minimumCacheTTL: 60 * 60 * 24 * 365, // 1 year
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },

  // Headers for caching
  headers: async () => [
    {
      source: '/(.*)',
      headers: [
        {
          key: 'X-DNS-Prefetch-Control',
          value: 'on'
        },
        {
          key: 'X-Frame-Options',
          value: 'DENY'
        }
      ],
    },
    {
      source: '/api/(.*)',
      headers: [
        {
          key: 'Cache-Control',
          value: 'public, max-age=300, s-maxage=300',
        },
      ],
    },
    {
      source: '/images/(.*)',
      headers: [
        {
          key: 'Cache-Control',
          value: 'public, max-age=31536000, immutable',
        },
      ],
    },
  ],

  // Compression
  compress: true,
  
  // PWA configuration
  pwa: {
    dest: 'public',
    register: true,
    skipWaiting: true,
    runtimeCaching: [
      {
        urlPattern: /^https?.*/,
        handler: 'NetworkFirst',
        options: {
          cacheName: 'offlineCache',
          expiration: {
            maxEntries: 200,
            maxAgeSeconds: 24 * 60 * 60, // 24 hours
          },
        },
      },
    ],
  },
};

module.exports = withBundleAnalyzer(nextConfig);

// lib/performance/lazy-loading.tsx - Advanced lazy loading
import { lazy, Suspense, ComponentType } from 'react';
import { ErrorBoundary } from '@/components/ErrorBoundary';

// Higher-order component for lazy loading with error handling
export function withLazyLoading<T extends object>(
  importFunc: () => Promise<{ default: ComponentType<T> }>,
  fallback: React.ReactNode = <div>Loading...</div>,
  errorFallback: React.ReactNode = <div>Error loading component</div>
) {
  const LazyComponent = lazy(importFunc);

  return function LazyWrapper(props: T) {
    return (
      <ErrorBoundary fallback={errorFallback}>
        <Suspense fallback={fallback}>
          <LazyComponent {...props} />
        </Suspense>
      </ErrorBoundary>
    );
  };
}

// Dynamic imports with preloading
export function createDynamicImport<T extends object>(
  importFunc: () => Promise<{ default: ComponentType<T> }>,
  options: {
    ssr?: boolean;
    loading?: React.ComponentType;
    error?: React.ComponentType<{ error: Error; retry: () => void }>;
    preload?: boolean;
  } = {}
) {
  const {
    ssr = true,
    loading: LoadingComponent,
    error: ErrorComponent,
    preload = false
  } = options;

  let componentPromise: Promise<{ default: ComponentType<T> }> | null = null;

  // Preload function
  const preloadComponent = () => {
    if (!componentPromise) {
      componentPromise = importFunc();
    }
    return componentPromise;
  };

  // Auto-preload if enabled
  if (preload && typeof window !== 'undefined') {
    // Preload after initial page load
    setTimeout(preloadComponent, 1000);
  }

  const DynamicComponent = dynamic(importFunc, {
    ssr,
    loading: LoadingComponent ? () => <LoadingComponent /> : undefined,
  });

  const WrappedComponent = (props: T) => {
    const [error, setError] = useState<Error | null>(null);

    if (error && ErrorComponent) {
      return (
        <ErrorComponent
          error={error}
          retry={() => {
            setError(null);
            componentPromise = null;
          }}
        />
      );
    }

    return <DynamicComponent {...props} />;
  };

  // Expose preload function
  WrappedComponent.preload = preloadComponent;

  return WrappedComponent;
}

// Route-based code splitting
export const LazyProductList = createDynamicImport(
  () => import('@/components/product/ProductList'),
  {
    loading: () => <div className="animate-pulse">Loading products...</div>,
    preload: true
  }
);

export const LazyAdminDashboard = createDynamicImport(
  () => import('@/components/admin/Dashboard'),
  {
    ssr: false,
    loading: () => <div>Loading dashboard...</div>
  }
);

export const LazyChart = createDynamicImport(
  () => import('@/components/charts/Chart'),
  {
    ssr: false,
    loading: () => <div className="h-64 bg-gray-100 animate-pulse rounded" />
  }
);

// Intersection Observer for lazy loading
import { useEffect, useRef, useState } from 'react';

export function useLazyLoad<T extends HTMLElement = HTMLDivElement>(
  options: IntersectionObserverInit = {}
) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const ref = useRef<T>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsIntersecting(true);
          observer.disconnect();
        }
      },
      {
        threshold: 0.1,
        rootMargin: '50px',
        ...options
      }
    );

    const element = ref.current;
    if (element) {
      observer.observe(element);
    }

    return () => {
      if (element) {
        observer.unobserve(element);
      }
    };
  }, [options]);

  return { ref, isIntersecting };
}

// Lazy loading component wrapper
export function LazyLoadWrapper({
  children,
  placeholder,
  className,
  threshold = 0.1,
  rootMargin = '50px'
}: {
  children: React.ReactNode;
  placeholder?: React.ReactNode;
  className?: string;
  threshold?: number;
  rootMargin?: string;
}) {
  const { ref, isIntersecting } = useLazyLoad({
    threshold,
    rootMargin
  });

  return (
    <div ref={ref} className={className}>
      {isIntersecting ? children : (placeholder || <div className="h-64 bg-gray-100" />)}
    </div>
  );
}
```

### **💾 Advanced Caching Strategies**

```typescript
// lib/cache/redis-cache.ts - Redis caching implementation
import { Redis } from 'ioredis';

export class RedisCache {
  private redis: Redis;

  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
      password: process.env.REDIS_PASSWORD,
      retryDelayOnFailover: 100,
      enableReadyCheck: false,
      maxRetriesPerRequest: null,
    });
  }

  async get<T>(key: string): Promise<T | null> {
    try {
      const value = await this.redis.get(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('Redis get error:', error);
      return null;
    }
  }

  async set(
    key: string, 
    value: any, 
    ttl: number = 3600
  ): Promise<boolean> {
    try {
      await this.redis.setex(key, ttl, JSON.stringify(value));
      return true;
    } catch (error) {
      console.error('Redis set error:', error);
      return false;
    }
  }

  async delete(key: string): Promise<boolean> {
    try {
      await this.redis.del(key);
      return true;
    } catch (error) {
      console.error('Redis delete error:', error);
      return false;
    }
  }

  async invalidatePattern(pattern: string): Promise<number> {
    try {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        return await this.redis.del(...keys);
      }
      return 0;
    } catch (error) {
      console.error('Redis invalidate pattern error:', error);
      return 0;
    }
  }

  async mget<T>(keys: string[]): Promise<(T | null)[]> {
    try {
      const values = await this.redis.mget(...keys);
      return values.map(value => value ? JSON.parse(value) : null);
    } catch (error) {
      console.error('Redis mget error:', error);
      return new Array(keys.length).fill(null);
    }
  }

  async mset(keyValuePairs: Array<[string, any, number?]>): Promise<boolean> {
    try {
      const pipeline = this.redis.pipeline();
      
      keyValuePairs.forEach(([key, value, ttl = 3600]) => {
        pipeline.setex(key, ttl, JSON.stringify(value));
      });

      await pipeline.exec();
      return true;
    } catch (error) {
      console.error('Redis mset error:', error);
      return false;
    }
  }

  // Cache with tags for easy invalidation
  async setWithTags(
    key: string,
    value: any,
    tags: string[],
    ttl: number = 3600
  ): Promise<boolean> {
    try {
      const pipeline = this.redis.pipeline();
      
      // Set the main key
      pipeline.setex(key, ttl, JSON.stringify(value));
      
      // Add key to tag sets
      tags.forEach(tag => {
        pipeline.sadd(`tag:${tag}`, key);
        pipeline.expire(`tag:${tag}`, ttl + 300); // Tag expires slightly later
      });

      await pipeline.exec();
      return true;
    } catch (error) {
      console.error('Redis set with tags error:', error);
      return false;
    }
  }

  async invalidateByTag(tag: string): Promise<number> {
    try {
      const keys = await this.redis.smembers(`tag:${tag}`);
      if (keys.length > 0) {
        const pipeline = this.redis.pipeline();
        keys.forEach(key => pipeline.del(key));
        pipeline.del(`tag:${tag}`);
        await pipeline.exec();
        return keys.length;
      }
      return 0;
    } catch (error) {
      console.error('Redis invalidate by tag error:', error);
      return 0;
    }
  }

  disconnect(): void {
    this.redis.disconnect();
  }
}

// Cache decorator for functions
export function cached(
  ttl: number = 3600,
  keyGenerator?: (...args: any[]) => string,
  tags?: string[]
) {
  return function (
    target: any,
    propertyName: string,
    descriptor: PropertyDescriptor
  ) {
    const method = descriptor.value;
    const cache = new RedisCache();

    descriptor.value = async function (...args: any[]) {
      const cacheKey = keyGenerator 
        ? keyGenerator(...args)
        : `${target.constructor.name}:${propertyName}:${JSON.stringify(args)}`;

      // Try to get from cache
      const cached = await cache.get(cacheKey);
      if (cached !== null) {
        return cached;
      }

      // Execute original method
      const result = await method.apply(this, args);

      // Store in cache
      if (tags) {
        await cache.setWithTags(cacheKey, result, tags, ttl);
      } else {
        await cache.set(cacheKey, result, ttl);
      }

      return result;
    };

    return descriptor;
  };
}

// lib/cache/memory-cache.ts - In-memory caching with LRU
import { LRUCache } from 'lru-cache';

export class MemoryCache {
  private cache: LRUCache<string, any>;

  constructor(options: {
    max?: number;
    maxAge?: number;
    updateAgeOnGet?: boolean;
  } = {}) {
    this.cache = new LRUCache({
      max: options.max || 1000,
      ttl: options.maxAge || 1000 * 60 * 60, // 1 hour
      updateAgeOnGet: options.updateAgeOnGet ?? true,
    });
  }

  get<T>(key: string): T | undefined {
    return this.cache.get(key);
  }

  set(key: string, value: any, ttl?: number): void {
    this.cache.set(key, value, { ttl });
  }

  delete(key: string): boolean {
    return this.cache.delete(key);
  }

  clear(): void {
    this.cache.clear();
  }

  has(key: string): boolean {
    return this.cache.has(key);
  }

  size(): number {
    return this.cache.size;
  }
}

// Multi-level caching strategy
export class MultiLevelCache {
  private l1Cache: MemoryCache; // Fast in-memory cache
  private l2Cache: RedisCache;  // Distributed cache

  constructor() {
    this.l1Cache = new MemoryCache({ max: 500, maxAge: 5 * 60 * 1000 }); // 5 minutes
    this.l2Cache = new RedisCache();
  }

  async get<T>(key: string): Promise<T | null> {
    // Try L1 cache first
    const l1Result = this.l1Cache.get<T>(key);
    if (l1Result !== undefined) {
      return l1Result;
    }

    // Try L2 cache
    const l2Result = await this.l2Cache.get<T>(key);
    if (l2Result !== null) {
      // Populate L1 cache
      this.l1Cache.set(key, l2Result);
      return l2Result;
    }

    return null;
  }

  async set(key: string, value: any, ttl: number = 3600): Promise<void> {
    // Set in both caches
    this.l1Cache.set(key, value, Math.min(ttl, 300) * 1000); // Max 5 minutes in L1
    await this.l2Cache.set(key, value, ttl);
  }

  async delete(key: string): Promise<void> {
    this.l1Cache.delete(key);
    await this.l2Cache.delete(key);
  }

  async invalidatePattern(pattern: string): Promise<void> {
    // Clear L1 cache completely for simplicity
    this.l1Cache.clear();
    // Invalidate L2 cache with pattern
    await this.l2Cache.invalidatePattern(pattern);
  }
}

// Global cache instance
export const cache = new MultiLevelCache();
```

### **🖼️ Image and Asset Optimization**

```tsx
// components/optimized/OptimizedImage.tsx
import Image, { ImageProps } from 'next/image';
import { useState, useEffect } from 'react';

interface OptimizedImageProps extends Omit<ImageProps, 'src'> {
  src: string;
  lowQualitySrc?: string;
  webpSrc?: string;
  avifSrc?: string;
  loading?: 'lazy' | 'eager';
  quality?: number;
  placeholder?: 'blur' | 'empty';
  blurDataURL?: string;
  sizes?: string;
  priority?: boolean;
  onLoadComplete?: () => void;
}

export function OptimizedImage({
  src,
  lowQualitySrc,
  webpSrc,
  avifSrc,
  loading = 'lazy',
  quality = 75,
  placeholder = 'empty',
  blurDataURL,
  sizes,
  priority = false,
  onLoadComplete,
  className,
  ...props
}: OptimizedImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [imageSrc, setImageSrc] = useState(lowQualitySrc || src);

  // Progressive loading
  useEffect(() => {
    if (lowQualitySrc) {
      const img = new window.Image();
      img.onload = () => {
        setImageSrc(src);
        setIsLoaded(true);
      };
      img.src = src;
    } else {
      setIsLoaded(true);
    }
  }, [src, lowQualitySrc]);

  // Generate responsive sizes if not provided
  const responsiveSizes = sizes || `
    (max-width: 640px) 100vw,
    (max-width: 1024px) 50vw,
    33vw
  `;

  // Generate blur placeholder if not provided
  const generateBlurDataURL = (width: number = 8, height: number = 6) => {
    if (blurDataURL) return blurDataURL;
    
    const canvas = document.createElement('canvas');
    canvas.width = width;
    canvas.height = height;
    const ctx = canvas.getContext('2d');
    
    if (ctx) {
      // Create a simple gradient blur
      const gradient = ctx.createLinearGradient(0, 0, width, height);
      gradient.addColorStop(0, '#f3f4f6');
      gradient.addColorStop(1, '#e5e7eb');
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, width, height);
    }
    
    return canvas.toDataURL();
  };

  const handleLoadComplete = () => {
    setIsLoaded(true);
    onLoadComplete?.();
  };

  return (
    <div className={`relative overflow-hidden ${className}`}>
      {/* Modern browser support with AVIF and WebP */}
      <picture>
        {avifSrc && (
          <source
            srcSet={avifSrc}
            type="image/avif"
            sizes={responsiveSizes}
          />
        )}
        {webpSrc && (
          <source
            srcSet={webpSrc}
            type="image/webp"
            sizes={responsiveSizes}
          />
        )}
        <Image
          {...props}
          src={imageSrc}
          loading={loading}
          quality={quality}
          placeholder={placeholder}
          blurDataURL={placeholder === 'blur' ? generateBlurDataURL() : undefined}
          sizes={responsiveSizes}
          priority={priority}
          onLoadingComplete={handleLoadComplete}
          className={`transition-opacity duration-300 ${
            isLoaded ? 'opacity-100' : 'opacity-0'
          }`}
        />
      </picture>

      {/* Loading placeholder */}
      {!isLoaded && (
        <div className="absolute inset-0 bg-gray-200 animate-pulse flex items-center justify-center">
          <div className="w-8 h-8 border-2 border-gray-300 border-t-blue-500 rounded-full animate-spin" />
        </div>
      )}
    </div>
  );
}

// Advanced image component with intersection observer
export function LazyOptimizedImage({
  threshold = 0.1,
  rootMargin = '50px',
  ...props
}: OptimizedImageProps & {
  threshold?: number;
  rootMargin?: string;
}) {
  const { ref, isIntersecting } = useLazyLoad({
    threshold,
    rootMargin
  });

  return (
    <div ref={ref}>
      {isIntersecting ? (
        <OptimizedImage {...props} />
      ) : (
        <div 
          className={`bg-gray-200 ${props.className}`}
          style={{ 
            aspectRatio: props.width && props.height 
              ? `${props.width} / ${props.height}` 
              : '16 / 9' 
          }}
        />
      )}
    </div>
  );
}

// lib/image/image-optimizer.ts - Server-side image optimization
import sharp from 'sharp';
import { promises as fs } from 'fs';
import path from 'path';

export class ImageOptimizer {
  private cacheDir: string;

  constructor(cacheDir: string = './public/cache/images') {
    this.cacheDir = cacheDir;
    this.ensureCacheDir();
  }

  private async ensureCacheDir(): Promise<void> {
    try {
      await fs.access(this.cacheDir);
    } catch {
      await fs.mkdir(this.cacheDir, { recursive: true });
    }
  }

  async optimizeImage(
    inputPath: string,
    options: {
      width?: number;
      height?: number;
      quality?: number;
      format?: 'jpeg' | 'png' | 'webp' | 'avif';
      progressive?: boolean;
    } = {}
  ): Promise<string> {
    const {
      width,
      height,
      quality = 80,
      format = 'webp',
      progressive = true
    } = options;

    // Generate cache key
    const cacheKey = this.generateCacheKey(inputPath, options);
    const cachePath = path.join(this.cacheDir, `${cacheKey}.${format}`);

    // Check if optimized image exists
    try {
      await fs.access(cachePath);
      return cachePath;
    } catch {
      // Image not cached, need to optimize
    }

    // Read and optimize image
    let pipeline = sharp(inputPath);

    // Resize if dimensions specified
    if (width || height) {
      pipeline = pipeline.resize(width, height, {
        fit: 'cover',
        position: 'center'
      });
    }

    // Set format and quality
    switch (format) {
      case 'jpeg':
        pipeline = pipeline.jpeg({ 
          quality, 
          progressive,
          mozjpeg: true 
        });
        break;
      case 'png':
        pipeline = pipeline.png({ 
          quality,
          progressive,
          compressionLevel: 9
        });
        break;
      case 'webp':
        pipeline = pipeline.webp({ 
          quality,
          effort: 6
        });
        break;
      case 'avif':
        pipeline = pipeline.avif({ 
          quality,
          effort: 9
        });
        break;
    }

    // Save optimized image
    await pipeline.toFile(cachePath);
    
    return cachePath;
  }

  async generateResponsiveImages(
    inputPath: string,
    breakpoints: number[] = [640, 768, 1024, 1280, 1920],
    formats: Array<'jpeg' | 'png' | 'webp' | 'avif'> = ['webp', 'avif']
  ): Promise<{
    srcSet: Record<string, string[]>;
    sizes: string;
  }> {
    const srcSet: Record<string, string[]> = {};

    for (const format of formats) {
      srcSet[format] = [];
      
      for (const width of breakpoints) {
        const optimizedPath = await this.optimizeImage(inputPath, {
          width,
          format,
          quality: 80
        });
        
        srcSet[format].push(`${optimizedPath} ${width}w`);
      }
    }

    const sizes = breakpoints
      .map((bp, index) => {
        if (index === breakpoints.length - 1) {
          return `${bp}px`;
        }
        return `(max-width: ${bp}px) ${bp}px`;
      })
      .join(', ');

    return { srcSet, sizes };
  }

  private generateCacheKey(
    inputPath: string,
    options: Record<string, any>
  ): string {
    const optionsString = JSON.stringify(options);
    const hash = require('crypto')
      .createHash('md5')
      .update(inputPath + optionsString)
      .digest('hex');
    
    return hash;
  }

  async clearCache(): Promise<void> {
    try {
      const files = await fs.readdir(this.cacheDir);
      await Promise.all(
        files.map(file => fs.unlink(path.join(this.cacheDir, file)))
      );
    } catch (error) {
      console.error('Error clearing image cache:', error);
    }
  }
}

// API route for image optimization
// pages/api/images/optimize.ts
import { NextApiRequest, NextApiResponse } from 'next/api';
import { ImageOptimizer } from '@/lib/image/image-optimizer';

const optimizer = new ImageOptimizer();

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { 
    src, 
    width, 
    height, 
    quality = 80, 
    format = 'webp' 
  } = req.query;

  if (!src || typeof src !== 'string') {
    return res.status(400).json({ error: 'Source parameter is required' });
  }

  try {
    const optimizedPath = await optimizer.optimizeImage(src, {
      width: width ? parseInt(width as string) : undefined,
      height: height ? parseInt(height as string) : undefined,
      quality: parseInt(quality as string),
      format: format as 'jpeg' | 'png' | 'webp' | 'avif'
    });

    // Set caching headers
    res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
    res.setHeader('Content-Type', `image/${format}`);
    
    // Stream the optimized image
    const { createReadStream } = require('fs');
    const stream = createReadStream(optimizedPath);
    stream.pipe(res);
    
  } catch (error) {
    console.error('Image optimization error:', error);
    res.status(500).json({ error: 'Image optimization failed' });
  }
}
```

### **📦 Asset Optimization**

```typescript
// lib/assets/asset-optimizer.ts
import { promises as fs } from 'fs';
import path from 'path';
import { minify } from 'terser';
import CleanCSS from 'clean-css';

export class AssetOptimizer {
  private outputDir: string;

  constructor(outputDir: string = './public/optimized') {
    this.outputDir = outputDir;
    this.ensureOutputDir();
  }

  private async ensureOutputDir(): Promise<void> {
    try {
      await fs.access(this.outputDir);
    } catch {
      await fs.mkdir(this.outputDir, { recursive: true });
    }
  }

  // JavaScript optimization
  async optimizeJavaScript(
    inputPath: string,
    options: {
      compress?: boolean;
      mangle?: boolean;
      sourceMap?: boolean;
    } = {}
  ): Promise<string> {
    const {
      compress = true,
      mangle = true,
      sourceMap = false
    } = options;

    const code = await fs.readFile(inputPath, 'utf-8');
    
    const result = await minify(code, {
      compress,
      mangle,
      sourceMap,
      format: {
        comments: false
      }
    });

    const outputPath = path.join(
      this.outputDir,
      path.basename(inputPath, '.js') + '.min.js'
    );

    await fs.writeFile(outputPath, result.code || code);

    if (sourceMap && result.map) {
      await fs.writeFile(outputPath + '.map', result.map);
    }

    return outputPath;
  }

  // CSS optimization
  async optimizeCSS(
    inputPath: string,
    options: {
      level?: 1 | 2;
      sourceMap?: boolean;
    } = {}
  ): Promise<string> {
    const {
      level = 2,
      sourceMap = false
    } = options;

    const css = await fs.readFile(inputPath, 'utf-8');
    
    const cleanCSS = new CleanCSS({
      level,
      sourceMap,
      returnPromise: true
    });

    const result = await cleanCSS.minify(css);

    const outputPath = path.join(
      this.outputDir,
      path.basename(inputPath, '.css') + '.min.css'
    );

    await fs.writeFile(outputPath, result.styles);

    if (sourceMap && result.sourceMap) {
      await fs.writeFile(outputPath + '.map', result.sourceMap.toString());
    }

    return outputPath;
  }

  // Bundle critical CSS
  async extractCriticalCSS(
    htmlContent: string,
    cssFiles: string[],
    options: {
      width?: number;
      height?: number;
      timeout?: number;
    } = {}
  ): Promise<string> {
    const {
      width = 1300,
      height = 900,
      timeout = 30000
    } = options;

    // Using puppeteer for critical CSS extraction
    const puppeteer = await import('puppeteer');
    const browser = await puppeteer.launch();
    
    try {
      const page = await browser.newPage();
      await page.setViewport({ width, height });
      
      // Load HTML content
      await page.setContent(htmlContent, { waitUntil: 'networkidle0', timeout });
      
      // Extract critical CSS
      const criticalCSS = await page.evaluate((cssFiles) => {
        const usedRules: string[] = [];
        
        cssFiles.forEach(cssFile => {
          const styleSheet = Array.from(document.styleSheets)
            .find(sheet => sheet.href?.includes(cssFile));
          
          if (styleSheet) {
            try {
              const rules = Array.from(styleSheet.cssRules || []);
              
              rules.forEach(rule => {
                if (rule.type === CSSRule.STYLE_RULE) {
                  const styleRule = rule as CSSStyleRule;
                  try {
                    if (document.querySelector(styleRule.selectorText)) {
                      usedRules.push(rule.cssText);
                    }
                  } catch (e) {
                    // Invalid selector, skip
                  }
                }
              });
            } catch (e) {
              // CORS or other access issues, skip
            }
          }
        });
        
        return usedRules.join('\n');
      }, cssFiles);

      return criticalCSS;
    } finally {
      await browser.close();
    }
  }

  // Font optimization
  async optimizeFonts(
    fontsDir: string,
    options: {
      formats?: Array<'woff' | 'woff2' | 'ttf' | 'eot'>;
      subset?: string;
    } = {}
  ): Promise<void> {
    const {
      formats = ['woff2', 'woff'],
      subset = 'latin'
    } = options;

    const fontFiles = await fs.readdir(fontsDir);
    const ttfFiles = fontFiles.filter(file => file.endsWith('.ttf'));

    for (const ttfFile of ttfFiles) {
      const inputPath = path.join(fontsDir, ttfFile);
      const baseName = path.basename(ttfFile, '.ttf');

      for (const format of formats) {
        if (format === 'woff2') {
          // Convert to WOFF2 using fonttools
          const outputPath = path.join(fontsDir, `${baseName}.woff2`);
          await this.convertFont(inputPath, outputPath, 'woff2', subset);
        } else if (format === 'woff') {
          // Convert to WOFF
          const outputPath = path.join(fontsDir, `${baseName}.woff`);
          await this.convertFont(inputPath, outputPath, 'woff', subset);
        }
      }
    }
  }

  private async convertFont(
    inputPath: string,
    outputPath: string,
    format: string,
    subset: string
  ): Promise<void> {
    // This would typically use a font conversion library like fonttools
    // For now, this is a placeholder implementation
    console.log(`Converting ${inputPath} to ${format} format with ${subset} subset`);
    
    // Example using fonttools (would need to be installed)
    // const { exec } = require('child_process');
    // const command = `pyftsubset ${inputPath} --output-file=${outputPath} --flavor=${format} --unicodes=U+0020-007F`;
    // await new Promise((resolve, reject) => {
    //   exec(command, (error: any) => {
    //     if (error) reject(error);
    //     else resolve(undefined);
    //   });
    // });
  }

  // Service Worker generation for caching
  async generateServiceWorker(
    assets: string[],
    options: {
      cacheFirst?: string[];
      networkFirst?: string[];
      staleWhileRevalidate?: string[];
    } = {}
  ): Promise<string> {
    const {
      cacheFirst = ['/images/', '/fonts/', '/css/', '/js/'],
      networkFirst = ['/api/'],
      staleWhileRevalidate = ['/']
    } = options;

    const swContent = `
const CACHE_NAME = 'app-cache-v${Date.now()}';
const ASSETS_TO_CACHE = ${JSON.stringify(assets)};

// Install event - cache assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(ASSETS_TO_CACHE))
      .then(() => self.skipWaiting())
  );
});

// Activate event - clean up old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then(cacheNames => {
        return Promise.all(
          cacheNames
            .filter(cacheName => cacheName !== CACHE_NAME)
            .map(cacheName => caches.delete(cacheName))
        );
      })
      .then(() => self.clients.claim())
  );
});

// Fetch event - handle requests
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);

  // Cache first strategy
  if (${JSON.stringify(cacheFirst)}.some(pattern => url.pathname.startsWith(pattern))) {
    event.respondWith(
      caches.match(request)
        .then(response => {
          if (response) return response;
          return fetch(request).then(response => {
            const responseClone = response.clone();
            caches.open(CACHE_NAME)
              .then(cache => cache.put(request, responseClone));
            return response;
          });
        })
    );
    return;
  }

  // Network first strategy
  if (${JSON.stringify(networkFirst)}.some(pattern => url.pathname.startsWith(pattern))) {
    event.respondWith(
      fetch(request)
        .catch(() => caches.match(request))
    );
    return;
  }

  // Stale while revalidate strategy
  if (${JSON.stringify(staleWhileRevalidate)}.some(pattern => url.pathname.startsWith(pattern))) {
    event.respondWith(
      caches.match(request)
        .then(response => {
          const fetchPromise = fetch(request).then(networkResponse => {
            caches.open(CACHE_NAME)
              .then(cache => cache.put(request, networkResponse.clone()));
            return networkResponse;
          });
          return response || fetchPromise;
        })
    );
    return;
  }

  // Default: network only
  event.respondWith(fetch(request));
});
`;

    const swPath = path.join(this.outputDir, 'sw.js');
    await fs.writeFile(swPath, swContent);
    
    return swPath;
  }
}

// Usage example
const optimizer = new AssetOptimizer();

// Optimize assets during build
export async function optimizeBuildAssets(): Promise<void> {
  const assetsToOptimize = [
    './public/js/main.js',
    './public/css/styles.css'
  ];

  for (const asset of assetsToOptimize) {
    if (asset.endsWith('.js')) {
      await optimizer.optimizeJavaScript(asset);
    } else if (asset.endsWith('.css')) {
      await optimizer.optimizeCSS(asset);
    }
  }

  // Generate service worker
  const staticAssets = [
    '/',
    '/css/styles.min.css',
    '/js/main.min.js',
    '/images/logo.webp'
  ];

  await optimizer.generateServiceWorker(staticAssets);
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Bundle Optimization**
1. ตั้งค่า webpack bundle splitting
2. เพิ่ม code splitting และ lazy loading
3. วิเคราะห์ bundle size ด้วย bundle analyzer
4. ทดสอบ performance improvements

### **🎯 แบบฝึกหัดที่ 2: Caching Implementation**
1. ตั้งค่า Redis caching
2. เพิ่ม multi-level caching strategy
3. สร้าง cache invalidation system
4. ทดสอบ cache performance

### **🎯 แบบฝึกหัดที่ 3: Image Optimization**
1. ตั้งค่า responsive images
2. เพิ่ม modern image formats (WebP, AVIF)
3. สร้าง progressive loading
4. ทดสอบ image loading performance

### **🎯 แบบฝึกหัดที่ 4: Asset Optimization**
1. ตั้งค่า CSS และ JS minification
2. เพิ่ม critical CSS extraction
3. สร้าง service worker for caching
4. ทดสอบ overall performance metrics

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 23: Real-world Projects](./23-real-world-projects.md)

**บทต่อไป**: [บทที่ 25: Expert Tips and Best Practices](./25-expert-tips.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Performance](https://nextjs.org/docs/advanced-features/measuring-performance)
- [Web Performance Metrics](https://web.dev/metrics/)
- [Bundle Analyzer](https://www.npmjs.com/package/@next/bundle-analyzer)
- [Sharp Image Processing](https://sharp.pixelplumbing.com/)
