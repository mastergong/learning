# บทที่ 15: Streaming and Suspense

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Streaming และ Suspense ใน Next.js
- ทำความเข้าใจ Progressive rendering
- เรียนรู้ Loading states และ Error boundaries
- เข้าใจ Performance optimization ด้วย Streaming

## 🌊 Streaming Overview

### **⚡ Streaming Fundamentals**

```jsx
// ✅ Basic Suspense implementation
// app/products/page.js
import { Suspense } from 'react';
import { ProductList } from '@/components/ProductList';
import { CategoryFilters } from '@/components/CategoryFilters';
import { SearchBar } from '@/components/SearchBar';

// Loading components
function ProductListSkeleton() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {[...Array(9)].map((_, i) => (
        <div key={i} className="animate-pulse">
          <div className="bg-gray-200 h-48 rounded-lg mb-4"></div>
          <div className="h-4 bg-gray-200 rounded mb-2"></div>
          <div className="h-4 bg-gray-200 rounded w-2/3 mb-2"></div>
          <div className="h-6 bg-gray-200 rounded w-1/3"></div>
        </div>
      ))}
    </div>
  );
}

function FiltersSkeleton() {
  return (
    <div className="space-y-4">
      <div className="h-6 bg-gray-200 rounded w-1/2 animate-pulse"></div>
      {[...Array(5)].map((_, i) => (
        <div key={i} className="h-4 bg-gray-200 rounded animate-pulse"></div>
      ))}
    </div>
  );
}

export default function ProductsPage({ searchParams }) {
  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-gray-900 mb-4">Products</h1>
        <SearchBar />
      </div>

      <div className="lg:grid lg:grid-cols-4 lg:gap-8">
        {/* Sidebar with filters */}
        <aside className="lg:col-span-1">
          <Suspense fallback={<FiltersSkeleton />}>
            <CategoryFilters />
          </Suspense>
        </aside>

        {/* Main content */}
        <main className="lg:col-span-3">
          <Suspense fallback={<ProductListSkeleton />}>
            <ProductList searchParams={searchParams} />
          </Suspense>
        </main>
      </div>
    </div>
  );
}

// components/ProductList.js - Server Component with data fetching
async function getProducts(searchParams = {}) {
  const { category, price, search, sort, page = 1 } = searchParams;
  const limit = 12;

  // ✅ Simulate network delay for demonstration
  await new Promise(resolve => setTimeout(resolve, 1000));

  const where = {
    ...(category && { categoryId: category }),
    ...(price && {
      price: {
        gte: parseInt(price.split('-')[0]),
        lte: parseInt(price.split('-')[1])
      }
    }),
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } }
      ]
    })
  };

  const orderBy = (() => {
    switch (sort) {
      case 'price-low':
        return { price: 'asc' };
      case 'price-high':
        return { price: 'desc' };
      case 'name':
        return { name: 'asc' };
      case 'newest':
        return { createdAt: 'desc' };
      default:
        return { popularity: 'desc' };
    }
  })();

  const [products, total] = await Promise.all([
    prisma.product.findMany({
      where,
      include: {
        category: true,
        images: true,
        _count: {
          select: { reviews: true }
        }
      },
      orderBy,
      skip: (page - 1) * limit,
      take: limit
    }),
    prisma.product.count({ where })
  ]);

  return { products, total };
}

export async function ProductList({ searchParams }) {
  const { products, total } = await getProducts(searchParams);

  if (products.length === 0) {
    return (
      <div className="text-center py-12">
        <div className="max-w-md mx-auto">
          <svg className="mx-auto h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2M4 13h2m13-8-2-2-2 2m0 0L9 9l-2-2" />
          </svg>
          <h3 className="mt-2 text-sm font-medium text-gray-900">No products found</h3>
          <p className="mt-1 text-sm text-gray-500">
            Try adjusting your search or filter criteria.
          </p>
        </div>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <p className="text-sm text-gray-700">
          Showing {products.length} of {total} products
        </p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>

      {/* Pagination component */}
      <ProductPagination 
        currentPage={parseInt(searchParams.page) || 1}
        totalProducts={total}
        productsPerPage={12}
      />
    </div>
  );
}
```

### **🎨 Advanced Loading States**

```jsx
// components/LoadingSpinner.js
export function LoadingSpinner({ size = 'md', className = '' }) {
  const sizeClasses = {
    sm: 'h-4 w-4',
    md: 'h-8 w-8',
    lg: 'h-12 w-12',
    xl: 'h-16 w-16'
  };

  return (
    <div className={`animate-spin rounded-full border-2 border-gray-300 border-t-blue-600 ${sizeClasses[size]} ${className}`}>
      <span className="sr-only">Loading...</span>
    </div>
  );
}

// components/skeletons/DashboardSkeleton.js
export function DashboardSkeleton() {
  return (
    <div className="space-y-6 animate-pulse">
      {/* Header skeleton */}
      <div className="flex justify-between items-center">
        <div className="h-8 bg-gray-200 rounded w-1/4"></div>
        <div className="h-10 bg-gray-200 rounded w-32"></div>
      </div>

      {/* Stats grid skeleton */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        {[...Array(4)].map((_, i) => (
          <div key={i} className="bg-white p-6 rounded-lg shadow border">
            <div className="flex items-center justify-between">
              <div className="space-y-2">
                <div className="h-4 bg-gray-200 rounded w-20"></div>
                <div className="h-8 bg-gray-200 rounded w-16"></div>
              </div>
              <div className="h-12 w-12 bg-gray-200 rounded"></div>
            </div>
            <div className="mt-4">
              <div className="h-3 bg-gray-200 rounded w-24"></div>
            </div>
          </div>
        ))}
      </div>

      {/* Chart skeleton */}
      <div className="bg-white p-6 rounded-lg shadow border">
        <div className="h-6 bg-gray-200 rounded w-1/3 mb-6"></div>
        <div className="h-64 bg-gray-200 rounded"></div>
      </div>

      {/* Table skeleton */}
      <div className="bg-white rounded-lg shadow border overflow-hidden">
        <div className="px-6 py-4 border-b">
          <div className="h-6 bg-gray-200 rounded w-1/4"></div>
        </div>
        <div className="divide-y divide-gray-200">
          {[...Array(5)].map((_, i) => (
            <div key={i} className="px-6 py-4 flex items-center space-x-4">
              <div className="h-10 w-10 bg-gray-200 rounded-full"></div>
              <div className="flex-1 space-y-2">
                <div className="h-4 bg-gray-200 rounded w-1/2"></div>
                <div className="h-3 bg-gray-200 rounded w-1/3"></div>
              </div>
              <div className="h-4 bg-gray-200 rounded w-16"></div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// components/skeletons/BlogPostSkeleton.js
export function BlogPostSkeleton() {
  return (
    <article className="max-w-4xl mx-auto animate-pulse">
      {/* Header skeleton */}
      <header className="mb-8">
        <div className="flex space-x-2 mb-4">
          <div className="h-6 bg-gray-200 rounded-full w-20"></div>
          <div className="h-6 bg-gray-200 rounded-full w-16"></div>
        </div>
        <div className="space-y-3">
          <div className="h-10 bg-gray-200 rounded w-3/4"></div>
          <div className="h-6 bg-gray-200 rounded w-1/2"></div>
        </div>
        <div className="flex items-center space-x-4 mt-6">
          <div className="h-8 w-8 bg-gray-200 rounded-full"></div>
          <div className="space-y-1">
            <div className="h-4 bg-gray-200 rounded w-24"></div>
            <div className="h-3 bg-gray-200 rounded w-20"></div>
          </div>
        </div>
      </header>

      {/* Featured image skeleton */}
      <div className="h-64 md:h-96 bg-gray-200 rounded-lg mb-8"></div>

      {/* Content skeleton */}
      <div className="space-y-4">
        {[...Array(8)].map((_, i) => (
          <div key={i} className="space-y-2">
            <div className="h-4 bg-gray-200 rounded"></div>
            <div className="h-4 bg-gray-200 rounded w-5/6"></div>
            <div className="h-4 bg-gray-200 rounded w-4/6"></div>
          </div>
        ))}
      </div>

      {/* Tags skeleton */}
      <div className="mt-8 pt-8 border-t border-gray-200">
        <div className="h-5 bg-gray-200 rounded w-16 mb-3"></div>
        <div className="flex space-x-2">
          {[...Array(4)].map((_, i) => (
            <div key={i} className="h-6 bg-gray-200 rounded-full w-16"></div>
          ))}
        </div>
      </div>
    </article>
  );
}

// components/ProgressiveLoader.js
'use client';

import { useState, useEffect } from 'react';

export function ProgressiveLoader({ 
  steps = ['Loading...', 'Almost there...', 'Just a moment...'],
  interval = 2000 
}) {
  const [currentStep, setCurrentStep] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCurrentStep(prev => (prev + 1) % steps.length);
    }, interval);

    return () => clearInterval(timer);
  }, [steps.length, interval]);

  return (
    <div className="flex flex-col items-center justify-center py-12">
      <LoadingSpinner size="lg" className="mb-4" />
      <p className="text-gray-600 text-lg transition-opacity duration-500">
        {steps[currentStep]}
      </p>
      
      {/* Progress dots */}
      <div className="flex space-x-2 mt-4">
        {steps.map((_, index) => (
          <div
            key={index}
            className={`h-2 w-2 rounded-full transition-colors duration-300 ${
              index === currentStep ? 'bg-blue-600' : 'bg-gray-300'
            }`}
          />
        ))}
      </div>
    </div>
  );
}
```

## 🎯 Nested Suspense Boundaries

### **📦 Complex Loading Hierarchies**

```jsx
// app/dashboard/page.js
import { Suspense } from 'react';
import { StatsCards } from '@/components/dashboard/StatsCards';
import { RevenueChart } from '@/components/dashboard/RevenueChart';
import { RecentOrders } from '@/components/dashboard/RecentOrders';
import { TopProducts } from '@/components/dashboard/TopProducts';
import { UserActivity } from '@/components/dashboard/UserActivity';

export default function DashboardPage() {
  return (
    <div className="space-y-8">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold text-gray-900">Dashboard</h1>
        <div className="text-sm text-gray-500">
          {new Date().toLocaleDateString()}
        </div>
      </div>

      {/* ✅ Fast-loading stats cards */}
      <Suspense fallback={<StatsCardsSkeleton />}>
        <StatsCards />
      </Suspense>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
        {/* ✅ Revenue chart - slower loading */}
        <div className="lg:col-span-2">
          <Suspense fallback={<ChartSkeleton />}>
            <RevenueChart />
          </Suspense>
        </div>

        {/* ✅ Top products - medium loading */}
        <div>
          <Suspense fallback={<ListSkeleton title="Top Products" />}>
            <TopProducts />
          </Suspense>
        </div>
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
        {/* ✅ Recent orders - medium loading */}
        <Suspense fallback={<TableSkeleton title="Recent Orders" />}>
          <RecentOrders />
        </Suspense>

        {/* ✅ User activity - slower loading */}
        <Suspense fallback={<ActivitySkeleton />}>
          <UserActivity />
        </Suspense>
      </div>
    </div>
  );
}

// components/dashboard/StatsCards.js
async function getStatsData() {
  // ✅ Fast query - basic counts
  const [totalUsers, totalOrders, totalRevenue, avgOrderValue] = await Promise.all([
    prisma.user.count(),
    prisma.order.count({ where: { status: 'completed' } }),
    prisma.order.aggregate({
      where: { status: 'completed' },
      _sum: { total: true }
    }),
    prisma.order.aggregate({
      where: { status: 'completed' },
      _avg: { total: true }
    })
  ]);

  return {
    totalUsers,
    totalOrders,
    totalRevenue: totalRevenue._sum.total || 0,
    avgOrderValue: avgOrderValue._avg.total || 0
  };
}

export async function StatsCards() {
  const stats = await getStatsData();

  const cards = [
    {
      title: 'Total Users',
      value: stats.totalUsers.toLocaleString(),
      change: '+12%',
      trend: 'up',
      icon: '👥'
    },
    {
      title: 'Total Orders',
      value: stats.totalOrders.toLocaleString(),
      change: '+8%',
      trend: 'up',
      icon: '📦'
    },
    {
      title: 'Revenue',
      value: `$${(stats.totalRevenue / 100).toLocaleString()}`,
      change: '+23%',
      trend: 'up',
      icon: '💰'
    },
    {
      title: 'Avg Order Value',
      value: `$${(stats.avgOrderValue / 100).toFixed(2)}`,
      change: '+5%',
      trend: 'up',
      icon: '📊'
    }
  ];

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      {cards.map((card, index) => (
        <div key={index} className="bg-white p-6 rounded-lg shadow border">
          <div className="flex items-center justify-between">
            <div>
              <p className="text-sm font-medium text-gray-600">{card.title}</p>
              <p className="text-3xl font-bold text-gray-900 mt-2">
                {card.value}
              </p>
            </div>
            <div className="text-3xl">{card.icon}</div>
          </div>
          <div className="mt-4 flex items-center">
            <span className={`text-sm font-medium ${
              card.trend === 'up' ? 'text-green-600' : 'text-red-600'
            }`}>
              {card.change}
            </span>
            <span className="text-sm text-gray-500 ml-2">vs last month</span>
          </div>
        </div>
      ))}
    </div>
  );
}

// components/dashboard/RevenueChart.js
async function getRevenueData() {
  // ✅ Slower query - complex aggregation
  await new Promise(resolve => setTimeout(resolve, 2000)); // Simulate slow query

  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

  const revenueData = await prisma.order.groupBy({
    by: ['createdAt'],
    where: {
      status: 'completed',
      createdAt: {
        gte: thirtyDaysAgo
      }
    },
    _sum: {
      total: true
    },
    orderBy: {
      createdAt: 'asc'
    }
  });

  // Process data for chart
  const chartData = revenueData.map(item => ({
    date: item.createdAt.toLocaleDateString(),
    revenue: item._sum.total / 100 // Convert cents to dollars
  }));

  return chartData;
}

export async function RevenueChart() {
  const data = await getRevenueData();

  return (
    <div className="bg-white p-6 rounded-lg shadow border">
      <h3 className="text-lg font-semibold text-gray-900 mb-6">
        Revenue Trend (Last 30 Days)
      </h3>
      
      {/* ✅ Chart component would go here */}
      <div className="h-64 flex items-center justify-center bg-gray-50 rounded">
        <div className="text-center">
          <div className="text-4xl mb-2">📈</div>
          <p className="text-gray-600">Revenue Chart</p>
          <p className="text-sm text-gray-500 mt-1">
            {data.length} data points
          </p>
        </div>
      </div>
    </div>
  );
}

// components/dashboard/RecentOrders.js
async function getRecentOrders() {
  // ✅ Medium complexity query
  await new Promise(resolve => setTimeout(resolve, 1500));

  return prisma.order.findMany({
    include: {
      user: {
        select: { name: true, email: true, avatar: true }
      },
      items: {
        include: {
          product: {
            select: { name: true, image: true }
          }
        }
      }
    },
    orderBy: { createdAt: 'desc' },
    take: 10
  });
}

export async function RecentOrders() {
  const orders = await getRecentOrders();

  return (
    <div className="bg-white rounded-lg shadow border overflow-hidden">
      <div className="px-6 py-4 border-b">
        <h3 className="text-lg font-semibold text-gray-900">Recent Orders</h3>
      </div>
      
      <div className="divide-y divide-gray-200 max-h-96 overflow-y-auto">
        {orders.map((order) => (
          <div key={order.id} className="px-6 py-4">
            <div className="flex items-center justify-between">
              <div className="flex items-center space-x-3">
                <img
                  src={order.user.avatar || '/default-avatar.png'}
                  alt={order.user.name}
                  className="h-8 w-8 rounded-full"
                />
                <div>
                  <p className="text-sm font-medium text-gray-900">
                    {order.user.name}
                  </p>
                  <p className="text-sm text-gray-500">
                    {order.items.length} item(s)
                  </p>
                </div>
              </div>
              <div className="text-right">
                <p className="text-sm font-medium text-gray-900">
                  ${(order.total / 100).toFixed(2)}
                </p>
                <p className="text-sm text-gray-500">
                  {new Date(order.createdAt).toLocaleDateString()}
                </p>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 🚨 Error Boundaries

### **⚠️ Error Handling with Suspense**

```jsx
// app/error.js - Global error boundary
'use client';

import { useEffect } from 'react';

export default function Error({ error, reset }) {
  useEffect(() => {
    // ✅ Log error to monitoring service
    console.error('Global error:', error);
    
    // Send to error tracking service
    if (process.env.NODE_ENV === 'production') {
      // Sentry, LogRocket, etc.
    }
  }, [error]);

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full bg-white rounded-lg shadow-lg p-6 text-center">
        <div className="text-red-500 text-6xl mb-4">⚠️</div>
        <h2 className="text-2xl font-bold text-gray-900 mb-2">
          Something went wrong!
        </h2>
        <p className="text-gray-600 mb-6">
          We're sorry, but something unexpected happened. Please try again.
        </p>
        
        <div className="space-y-3">
          <button
            onClick={reset}
            className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 transition-colors"
          >
            Try Again
          </button>
          
          <button
            onClick={() => window.location.href = '/'}
            className="w-full bg-gray-300 text-gray-700 py-2 px-4 rounded-md hover:bg-gray-400 transition-colors"
          >
            Go Home
          </button>
        </div>

        {process.env.NODE_ENV === 'development' && (
          <details className="mt-6 text-left">
            <summary className="cursor-pointer text-sm text-gray-500">
              Error Details (Development)
            </summary>
            <pre className="mt-2 text-xs bg-gray-100 p-2 rounded overflow-auto">
              {error.message}
            </pre>
          </details>
        )}
      </div>
    </div>
  );
}

// app/blog/error.js - Blog-specific error boundary
'use client';

export default function BlogError({ error, reset }) {
  return (
    <div className="max-w-4xl mx-auto px-4 py-16">
      <div className="text-center">
        <div className="text-red-500 text-5xl mb-4">📚</div>
        <h2 className="text-2xl font-bold text-gray-900 mb-2">
          Blog Temporarily Unavailable
        </h2>
        <p className="text-gray-600 mb-6">
          We're having trouble loading the blog posts right now. Please try again in a moment.
        </p>
        
        <div className="space-x-4">
          <button
            onClick={reset}
            className="bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700"
          >
            Retry
          </button>
          
          <a
            href="/"
            className="bg-gray-300 text-gray-700 py-2 px-4 rounded-md hover:bg-gray-400"
          >
            Go Home
          </a>
        </div>
      </div>
    </div>
  );
}

// components/ErrorBoundary.js - Custom error boundary component
'use client';

import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    // ✅ Update state to render fallback UI
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // ✅ Log error to monitoring service
    console.error('ErrorBoundary caught an error:', error, errorInfo);
    
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  render() {
    if (this.state.hasError) {
      // ✅ Custom fallback UI
      if (this.props.fallback) {
        return this.props.fallback(this.state.error);
      }

      return (
        <div className="p-6 bg-red-50 border border-red-200 rounded-lg">
          <div className="flex items-start">
            <div className="text-red-400 mr-3">
              <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clipRule="evenodd" />
              </svg>
            </div>
            <div>
              <h3 className="text-red-800 font-medium">
                {this.props.title || 'Something went wrong'}
              </h3>
              <p className="text-red-700 mt-1">
                {this.props.message || 'Please try refreshing the page or contact support if the problem persists.'}
              </p>
              
              {this.props.showRetry && (
                <button
                  onClick={() => this.setState({ hasError: false, error: null })}
                  className="mt-3 text-sm bg-red-600 text-white px-3 py-1 rounded hover:bg-red-700"
                >
                  Try Again
                </button>
              )}
            </div>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;

// components/SafeComponent.js - Component with error boundary
import ErrorBoundary from './ErrorBoundary';

export function SafeComponent({ children, title, message, onError }) {
  return (
    <ErrorBoundary
      title={title}
      message={message}
      onError={onError}
      showRetry
      fallback={(error) => (
        <div className="p-4 bg-yellow-50 border border-yellow-200 rounded">
          <h4 className="text-yellow-800 font-medium">Component Error</h4>
          <p className="text-yellow-700 text-sm mt-1">
            This component failed to load. 
          </p>
          {process.env.NODE_ENV === 'development' && (
            <details className="mt-2">
              <summary className="text-xs text-yellow-600 cursor-pointer">
                Debug Info
              </summary>
              <pre className="text-xs bg-yellow-100 p-2 mt-1 rounded">
                {error.message}
              </pre>
            </details>
          )}
        </div>
      )}
    >
      {children}
    </ErrorBoundary>
  );
}
```

## 🔄 Progressive Enhancement

### **⚡ Incremental Loading**

```jsx
// components/InfiniteScroll.js
'use client';

import { useState, useEffect, useRef, useCallback } from 'react';
import { useRouter } from 'next/navigation';

export function InfiniteScroll({ 
  initialData = [],
  loadMore,
  hasMore = true,
  threshold = 100 
}) {
  const [data, setData] = useState(initialData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const observerRef = useRef();
  const router = useRouter();

  const lastElementRef = useCallback((node) => {
    if (loading) return;
    if (observerRef.current) observerRef.current.disconnect();
    
    observerRef.current = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting && hasMore) {
        loadMoreData();
      }
    }, { rootMargin: `${threshold}px` });
    
    if (node) observerRef.current.observe(node);
  }, [loading, hasMore, threshold]);

  const loadMoreData = async () => {
    if (loading || !hasMore) return;

    setLoading(true);
    setError(null);

    try {
      const newData = await loadMore(data.length);
      setData(prev => [...prev, ...newData.items]);
      
      // ✅ Update URL with current page
      const newPage = Math.ceil((data.length + newData.items.length) / 10);
      router.replace(`?page=${newPage}`, { scroll: false });
      
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      {data.map((item, index) => (
        <div
          key={item.id}
          ref={index === data.length - 1 ? lastElementRef : null}
          className="p-4 bg-white rounded-lg shadow border"
        >
          {/* Render item content */}
          <h3 className="font-semibold">{item.title}</h3>
          <p className="text-gray-600 mt-2">{item.description}</p>
        </div>
      ))}

      {/* Loading indicator */}
      {loading && (
        <div className="flex justify-center py-8">
          <LoadingSpinner />
        </div>
      )}

      {/* Error state */}
      {error && (
        <div className="text-center py-8">
          <p className="text-red-600 mb-4">Failed to load more items</p>
          <button
            onClick={loadMoreData}
            className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
          >
            Try Again
          </button>
        </div>
      )}

      {/* End message */}
      {!hasMore && data.length > 0 && (
        <div className="text-center py-8 text-gray-500">
          You've reached the end!
        </div>
      )}
    </div>
  );
}

// app/posts/page.js with infinite scroll
import { Suspense } from 'react';
import { PostsInfiniteList } from '@/components/PostsInfiniteList';

export default function PostsPage({ searchParams }) {
  return (
    <div className="max-w-4xl mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">All Posts</h1>
      
      <Suspense fallback={<div>Loading posts...</div>}>
        <PostsInfiniteList searchParams={searchParams} />
      </Suspense>
    </div>
  );
}

// components/PostsInfiniteList.js
import { InfiniteScroll } from './InfiniteScroll';

async function getInitialPosts(page = 1, limit = 10) {
  const posts = await prisma.post.findMany({
    where: { published: true },
    include: {
      author: { select: { name: true, avatar: true } },
      _count: { select: { comments: true, likes: true } }
    },
    orderBy: { publishedAt: 'desc' },
    skip: (page - 1) * limit,
    take: limit
  });

  const total = await prisma.post.count({ where: { published: true } });
  
  return {
    posts,
    hasMore: page * limit < total
  };
}

export async function PostsInfiniteList({ searchParams }) {
  const page = parseInt(searchParams?.page) || 1;
  const { posts, hasMore } = await getInitialPosts(page);

  const loadMore = async (offset) => {
    'use server';
    
    const limit = 10;
    const page = Math.floor(offset / limit) + 1;
    
    const newPosts = await prisma.post.findMany({
      where: { published: true },
      include: {
        author: { select: { name: true, avatar: true } },
        _count: { select: { comments: true, likes: true } }
      },
      orderBy: { publishedAt: 'desc' },
      skip: offset,
      take: limit
    });

    const total = await prisma.post.count({ where: { published: true } });
    
    return {
      items: newPosts,
      hasMore: offset + limit < total
    };
  };

  return (
    <InfiniteScroll
      initialData={posts}
      loadMore={loadMore}
      hasMore={hasMore}
    />
  );
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic Streaming**
1. สร้าง page พร้อม multiple Suspense boundaries
2. เพิ่ม loading skeletons
3. ทดสอบ streaming behavior
4. วัด performance improvements

### **🎯 แบบฝึกหัดที่ 2: Advanced Loading States**
1. สร้าง custom loading components
2. เพิ่ม progressive loading messages
3. สร้าง skeleton screens
4. ทดสอบ user experience

### **🎯 แบบฝึกหัดที่ 3: Error Boundaries**
1. เพิ่ม error boundaries ต่าง ๆ
2. สร้าง custom error components
3. ทดสอบ error recovery
4. เพิ่ม error logging

### **🎯 แบบฝึกหัดที่ 4: Infinite Scroll**
1. สร้าง infinite scroll component
2. เพิ่ม progressive loading
3. จัดการ URL state
4. ทดสอบ performance

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 14: Server Components](./14-server-components.md)

**บทต่อไป**: [บทที่ 16: Middleware](./16-middleware.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Streaming](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)
- [React Suspense](https://react.dev/reference/react/Suspense)
- [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [Progressive Enhancement](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#streaming-with-suspense)
