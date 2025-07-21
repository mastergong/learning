# บทที่ 21: Monitoring and Analytics

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Application monitoring และ observability
- ทำความเข้าใจ Performance monitoring และ APM
- เรียนรู้ Analytics tracking และ user behavior
- เข้าใจ Error tracking และ alerting systems

## 📊 Application Performance Monitoring

### **⚡ Next.js Performance Monitoring**

```javascript
// lib/monitoring/performance.js
export class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.observers = new Map();
    this.setupObservers();
  }

  setupObservers() {
    if (typeof window === 'undefined') return;

    // ✅ Core Web Vitals monitoring
    this.observeWebVitals();
    
    // ✅ Navigation timing
    this.observeNavigation();
    
    // ✅ Resource timing
    this.observeResources();
    
    // ✅ Long tasks
    this.observeLongTasks();
  }

  observeWebVitals() {
    if ('PerformanceObserver' in window) {
      // Largest Contentful Paint
      const lcpObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          this.recordMetric('LCP', {
            value: entry.startTime,
            element: entry.element?.tagName,
            url: entry.url,
            timestamp: Date.now()
          });
        }
      });

      try {
        lcpObserver.observe({ entryTypes: ['largest-contentful-paint'] });
        this.observers.set('lcp', lcpObserver);
      } catch (e) {
        console.warn('LCP observer not supported');
      }

      // First Input Delay
      const fidObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          this.recordMetric('FID', {
            value: entry.processingStart - entry.startTime,
            eventType: entry.name,
            timestamp: Date.now()
          });
        }
      });

      try {
        fidObserver.observe({ entryTypes: ['first-input'] });
        this.observers.set('fid', fidObserver);
      } catch (e) {
        console.warn('FID observer not supported');
      }

      // Cumulative Layout Shift
      let clsValue = 0;
      const clsObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (!entry.hadRecentInput) {
            clsValue += entry.value;
            this.recordMetric('CLS', {
              value: clsValue,
              sources: entry.sources?.map(source => ({
                element: source.element?.tagName,
                previousRect: source.previousRect,
                currentRect: source.currentRect
              })),
              timestamp: Date.now()
            });
          }
        }
      });

      try {
        clsObserver.observe({ entryTypes: ['layout-shift'] });
        this.observers.set('cls', clsObserver);
      } catch (e) {
        console.warn('CLS observer not supported');
      }
    }
  }

  observeNavigation() {
    if ('PerformanceObserver' in window) {
      const navObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          this.recordMetric('NAVIGATION', {
            dns: entry.domainLookupEnd - entry.domainLookupStart,
            tcp: entry.connectEnd - entry.connectStart,
            tls: entry.connectEnd - entry.secureConnectionStart,
            ttfb: entry.responseStart - entry.requestStart,
            download: entry.responseEnd - entry.responseStart,
            domContentLoaded: entry.domContentLoadedEventEnd - entry.navigationStart,
            load: entry.loadEventEnd - entry.navigationStart,
            type: entry.type,
            timestamp: Date.now()
          });
        }
      });

      try {
        navObserver.observe({ entryTypes: ['navigation'] });
        this.observers.set('navigation', navObserver);
      } catch (e) {
        console.warn('Navigation observer not supported');
      }
    }
  }

  observeResources() {
    if ('PerformanceObserver' in window) {
      const resourceObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          // Filter out data URIs and non-HTTP resources
          if (entry.name.startsWith('data:') || entry.name.startsWith('blob:')) {
            continue;
          }

          this.recordMetric('RESOURCE', {
            name: entry.name,
            type: entry.initiatorType,
            size: entry.transferSize,
            duration: entry.duration,
            cached: entry.transferSize === 0 && entry.duration > 0,
            timestamp: Date.now()
          });
        }
      });

      try {
        resourceObserver.observe({ entryTypes: ['resource'] });
        this.observers.set('resource', resourceObserver);
      } catch (e) {
        console.warn('Resource observer not supported');
      }
    }
  }

  observeLongTasks() {
    if ('PerformanceObserver' in window) {
      const longTaskObserver = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          this.recordMetric('LONG_TASK', {
            duration: entry.duration,
            startTime: entry.startTime,
            attribution: entry.attribution?.map(attr => ({
              name: attr.name,
              entryType: attr.entryType,
              startTime: attr.startTime,
              duration: attr.duration
            })),
            timestamp: Date.now()
          });
        }
      });

      try {
        longTaskObserver.observe({ entryTypes: ['longtask'] });
        this.observers.set('longtask', longTaskObserver);
      } catch (e) {
        console.warn('Long task observer not supported');
      }
    }
  }

  recordMetric(type, data) {
    const metric = {
      type,
      data,
      url: window.location.href,
      userAgent: navigator.userAgent,
      connection: navigator.connection ? {
        effectiveType: navigator.connection.effectiveType,
        downlink: navigator.connection.downlink,
        rtt: navigator.connection.rtt
      } : null
    };

    // Store locally
    if (!this.metrics.has(type)) {
      this.metrics.set(type, []);
    }
    this.metrics.get(type).push(metric);

    // Send to analytics
    this.sendMetric(metric);
  }

  async sendMetric(metric) {
    try {
      // Use sendBeacon for reliability
      if (navigator.sendBeacon) {
        navigator.sendBeacon('/api/analytics/performance', JSON.stringify(metric));
      } else {
        fetch('/api/analytics/performance', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(metric),
          keepalive: true
        }).catch(console.error);
      }
    } catch (error) {
      console.error('Failed to send metric:', error);
    }
  }

  getMetrics(type) {
    return this.metrics.get(type) || [];
  }

  clear() {
    this.metrics.clear();
  }

  disconnect() {
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
  }
}

// Create global instance
export const performanceMonitor = new PerformanceMonitor();

// Hook for React components
import { useEffect } from 'react';

export function usePerformanceMonitoring() {
  useEffect(() => {
    return () => {
      // Cleanup on unmount
      performanceMonitor.disconnect();
    };
  }, []);

  return {
    recordCustomMetric: (name, value, metadata = {}) => {
      performanceMonitor.recordMetric('CUSTOM', {
        name,
        value,
        metadata,
        timestamp: Date.now()
      });
    },
    getMetrics: performanceMonitor.getMetrics.bind(performanceMonitor)
  };
}

// pages/api/analytics/performance.js - Performance data collection
import { prisma } from '@/lib/prisma';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { type, data, url, userAgent, connection } = req.body;

    // Store performance metric
    await prisma.performanceMetric.create({
      data: {
        type,
        value: JSON.stringify(data),
        url,
        userAgent,
        connection: connection ? JSON.stringify(connection) : null,
        sessionId: req.headers['x-session-id'],
        userId: req.headers['x-user-id'],
        timestamp: new Date()
      }
    });

    // Send to external monitoring service
    if (process.env.MONITORING_WEBHOOK) {
      await fetch(process.env.MONITORING_WEBHOOK, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          type: 'performance',
          metric: type,
          data,
          metadata: { url, userAgent, connection }
        })
      });
    }

    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Performance tracking error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

### **📈 Real User Monitoring (RUM)**

```javascript
// lib/monitoring/rum.js
export class RealUserMonitoring {
  constructor(config = {}) {
    this.config = {
      endpoint: '/api/analytics/rum',
      batchSize: 10,
      flushInterval: 5000,
      enableConsoleCapture: true,
      enableErrorCapture: true,
      enableInteractionCapture: true,
      ...config
    };

    this.queue = [];
    this.sessionId = this.generateSessionId();
    this.pageId = this.generatePageId();
    this.startTime = Date.now();

    this.setupCaptures();
    this.startBatchProcessor();
  }

  generateSessionId() {
    return `sess_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  generatePageId() {
    return `page_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  setupCaptures() {
    if (typeof window === 'undefined') return;

    // ✅ Page load tracking
    this.trackPageLoad();

    // ✅ Route changes (for SPAs)
    this.trackRouteChanges();

    // ✅ User interactions
    if (this.config.enableInteractionCapture) {
      this.trackInteractions();
    }

    // ✅ Console logs
    if (this.config.enableConsoleCapture) {
      this.captureConsoleLogs();
    }

    // ✅ JavaScript errors
    if (this.config.enableErrorCapture) {
      this.captureErrors();
    }

    // ✅ Network requests
    this.trackNetworkRequests();

    // ✅ Visibility changes
    this.trackVisibilityChanges();
  }

  trackPageLoad() {
    window.addEventListener('load', () => {
      this.track('page_load', {
        url: window.location.href,
        referrer: document.referrer,
        loadTime: Date.now() - this.startTime,
        timestamp: Date.now()
      });
    });

    // Track when page becomes visible
    if (document.readyState === 'complete') {
      this.track('page_interactive', {
        url: window.location.href,
        timeToInteractive: Date.now() - this.startTime,
        timestamp: Date.now()
      });
    } else {
      window.addEventListener('load', () => {
        this.track('page_interactive', {
          url: window.location.href,
          timeToInteractive: Date.now() - this.startTime,
          timestamp: Date.now()
        });
      });
    }
  }

  trackRouteChanges() {
    // For Next.js App Router
    if (typeof window !== 'undefined') {
      let currentPath = window.location.pathname;
      
      const observer = new MutationObserver(() => {
        if (window.location.pathname !== currentPath) {
          this.track('route_change', {
            from: currentPath,
            to: window.location.pathname,
            timestamp: Date.now()
          });
          currentPath = window.location.pathname;
          this.pageId = this.generatePageId(); // New page ID for new route
        }
      });

      observer.observe(document.body, {
        childList: true,
        subtree: true
      });
    }
  }

  trackInteractions() {
    // Click tracking
    document.addEventListener('click', (event) => {
      const element = event.target;
      this.track('click', {
        element: element.tagName,
        className: element.className,
        id: element.id,
        text: element.textContent?.slice(0, 100),
        x: event.clientX,
        y: event.clientY,
        timestamp: Date.now()
      });
    });

    // Form submissions
    document.addEventListener('submit', (event) => {
      const form = event.target;
      this.track('form_submit', {
        formId: form.id,
        formName: form.name,
        action: form.action,
        method: form.method,
        fieldCount: form.elements.length,
        timestamp: Date.now()
      });
    });

    // Scroll tracking
    let scrollTimeout;
    window.addEventListener('scroll', () => {
      clearTimeout(scrollTimeout);
      scrollTimeout = setTimeout(() => {
        this.track('scroll', {
          scrollY: window.scrollY,
          scrollPercent: Math.round((window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100),
          timestamp: Date.now()
        });
      }, 100);
    });
  }

  captureConsoleLogs() {
    const originalLog = console.log;
    const originalError = console.error;
    const originalWarn = console.warn;

    console.log = (...args) => {
      this.track('console_log', {
        level: 'log',
        message: args.join(' '),
        timestamp: Date.now()
      });
      originalLog.apply(console, args);
    };

    console.error = (...args) => {
      this.track('console_log', {
        level: 'error',
        message: args.join(' '),
        timestamp: Date.now()
      });
      originalError.apply(console, args);
    };

    console.warn = (...args) => {
      this.track('console_log', {
        level: 'warn',
        message: args.join(' '),
        timestamp: Date.now()
      });
      originalWarn.apply(console, args);
    };
  }

  captureErrors() {
    window.addEventListener('error', (event) => {
      this.track('javascript_error', {
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack,
        timestamp: Date.now()
      });
    });

    window.addEventListener('unhandledrejection', (event) => {
      this.track('unhandled_promise_rejection', {
        reason: event.reason?.toString(),
        stack: event.reason?.stack,
        timestamp: Date.now()
      });
    });
  }

  trackNetworkRequests() {
    // Intercept fetch requests
    const originalFetch = window.fetch;
    window.fetch = async (...args) => {
      const startTime = Date.now();
      const url = args[0];
      
      try {
        const response = await originalFetch(...args);
        
        this.track('network_request', {
          url: url.toString(),
          method: args[1]?.method || 'GET',
          status: response.status,
          duration: Date.now() - startTime,
          success: response.ok,
          timestamp: Date.now()
        });
        
        return response;
      } catch (error) {
        this.track('network_request', {
          url: url.toString(),
          method: args[1]?.method || 'GET',
          duration: Date.now() - startTime,
          success: false,
          error: error.message,
          timestamp: Date.now()
        });
        throw error;
      }
    };
  }

  trackVisibilityChanges() {
    document.addEventListener('visibilitychange', () => {
      this.track('visibility_change', {
        visible: !document.hidden,
        timestamp: Date.now()
      });
    });

    window.addEventListener('beforeunload', () => {
      this.track('page_unload', {
        timeOnPage: Date.now() - this.startTime,
        timestamp: Date.now()
      });
      this.flush(); // Send remaining data
    });
  }

  track(event, data) {
    const entry = {
      sessionId: this.sessionId,
      pageId: this.pageId,
      event,
      data,
      url: window.location.href,
      userAgent: navigator.userAgent,
      timestamp: Date.now()
    };

    this.queue.push(entry);

    // Flush if queue is full
    if (this.queue.length >= this.config.batchSize) {
      this.flush();
    }
  }

  startBatchProcessor() {
    setInterval(() => {
      if (this.queue.length > 0) {
        this.flush();
      }
    }, this.config.flushInterval);
  }

  async flush() {
    if (this.queue.length === 0) return;

    const batch = [...this.queue];
    this.queue = [];

    try {
      if (navigator.sendBeacon) {
        navigator.sendBeacon(this.config.endpoint, JSON.stringify(batch));
      } else {
        await fetch(this.config.endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(batch),
          keepalive: true
        });
      }
    } catch (error) {
      console.error('Failed to send RUM data:', error);
      // Re-queue failed items
      this.queue.unshift(...batch);
    }
  }

  setUser(userId, userInfo = {}) {
    this.track('user_identified', {
      userId,
      userInfo,
      timestamp: Date.now()
    });
  }

  setCustomData(key, value) {
    this.track('custom_data', {
      key,
      value,
      timestamp: Date.now()
    });
  }
}

// Initialize RUM
export const rum = new RealUserMonitoring();

// React hook for RUM
export function useRUM() {
  useEffect(() => {
    return () => {
      rum.flush();
    };
  }, []);

  return {
    track: rum.track.bind(rum),
    setUser: rum.setUser.bind(rum),
    setCustomData: rum.setCustomData.bind(rum)
  };
}
```

## 📊 Business Analytics

### **📈 Event Tracking System**

```javascript
// lib/analytics/events.js
export class EventTracker {
  constructor() {
    this.queue = [];
    this.userId = null;
    this.sessionId = this.generateSessionId();
    this.properties = {};
    this.setupAutoTracking();
  }

  generateSessionId() {
    return `sess_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  setupAutoTracking() {
    if (typeof window === 'undefined') return;

    // ✅ Page views
    this.trackPageView();

    // ✅ Link clicks
    this.trackLinkClicks();

    // ✅ Form interactions
    this.trackFormInteractions();

    // ✅ Video/Media interactions
    this.trackMediaInteractions();

    // ✅ E-commerce events
    this.trackEcommerceEvents();
  }

  trackPageView() {
    const pageData = {
      page: window.location.pathname,
      title: document.title,
      url: window.location.href,
      referrer: document.referrer,
      search: window.location.search,
      hash: window.location.hash
    };

    this.track('page_view', pageData);
  }

  trackLinkClicks() {
    document.addEventListener('click', (event) => {
      const link = event.target.closest('a');
      if (link) {
        const isExternal = link.hostname !== window.location.hostname;
        const isDownload = link.download || /\.(pdf|doc|docx|xls|xlsx|zip|rar)$/i.test(link.href);

        this.track('link_click', {
          url: link.href,
          text: link.textContent?.trim(),
          external: isExternal,
          download: isDownload,
          target: link.target
        });
      }
    });
  }

  trackFormInteractions() {
    // Form starts
    document.addEventListener('focusin', (event) => {
      const form = event.target.closest('form');
      if (form && event.target.type !== 'submit') {
        this.track('form_start', {
          formId: form.id,
          formName: form.name,
          fieldName: event.target.name,
          fieldType: event.target.type
        });
      }
    });

    // Form submissions
    document.addEventListener('submit', (event) => {
      const form = event.target;
      const formData = new FormData(form);
      const data = {};
      for (const [key, value] of formData.entries()) {
        // Don't send sensitive data
        if (!['password', 'ssn', 'credit_card'].includes(key)) {
          data[key] = typeof value === 'string' ? value.slice(0, 100) : value;
        }
      }

      this.track('form_submit', {
        formId: form.id,
        formName: form.name,
        action: form.action,
        method: form.method,
        fieldCount: form.elements.length,
        data
      });
    });
  }

  trackMediaInteractions() {
    document.addEventListener('play', (event) => {
      if (event.target.tagName === 'VIDEO' || event.target.tagName === 'AUDIO') {
        this.track('media_play', {
          type: event.target.tagName.toLowerCase(),
          src: event.target.src,
          duration: event.target.duration,
          currentTime: event.target.currentTime
        });
      }
    });

    document.addEventListener('pause', (event) => {
      if (event.target.tagName === 'VIDEO' || event.target.tagName === 'AUDIO') {
        this.track('media_pause', {
          type: event.target.tagName.toLowerCase(),
          src: event.target.src,
          duration: event.target.duration,
          currentTime: event.target.currentTime,
          watchTime: event.target.currentTime
        });
      }
    });
  }

  trackEcommerceEvents() {
    // Product views (assuming data attributes)
    document.addEventListener('click', (event) => {
      const productElement = event.target.closest('[data-product-id]');
      if (productElement) {
        this.track('product_view', {
          productId: productElement.dataset.productId,
          productName: productElement.dataset.productName,
          category: productElement.dataset.category,
          price: parseFloat(productElement.dataset.price),
          currency: productElement.dataset.currency || 'USD'
        });
      }
    });

    // Add to cart buttons
    document.addEventListener('click', (event) => {
      if (event.target.matches('[data-action="add-to-cart"]')) {
        const productData = {
          productId: event.target.dataset.productId,
          productName: event.target.dataset.productName,
          price: parseFloat(event.target.dataset.price),
          quantity: parseInt(event.target.dataset.quantity) || 1,
          category: event.target.dataset.category,
          currency: event.target.dataset.currency || 'USD'
        };

        this.track('add_to_cart', productData);
      }
    });
  }

  // Core tracking method
  track(eventName, properties = {}, options = {}) {
    const event = {
      event: eventName,
      properties: {
        ...this.properties,
        ...properties,
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
        url: window.location.href,
        userAgent: navigator.userAgent,
        screen: {
          width: window.screen.width,
          height: window.screen.height
        },
        viewport: {
          width: window.innerWidth,
          height: window.innerHeight
        },
        timezone: Intl.DateTimeFormat().resolvedOptions().timeZone
      },
      options
    };

    this.queue.push(event);
    this.flush();
  }

  // User identification
  identify(userId, traits = {}) {
    this.userId = userId;
    this.track('user_identify', {
      userId,
      traits
    });
  }

  // Set global properties
  setProperties(properties) {
    this.properties = { ...this.properties, ...properties };
  }

  // E-commerce specific methods
  trackPurchase(transactionData) {
    this.track('purchase', {
      transactionId: transactionData.id,
      revenue: transactionData.total,
      currency: transactionData.currency,
      items: transactionData.items.map(item => ({
        productId: item.id,
        productName: item.name,
        category: item.category,
        price: item.price,
        quantity: item.quantity
      })),
      coupon: transactionData.coupon,
      paymentMethod: transactionData.paymentMethod
    });
  }

  trackSearch(query, filters = {}, results = 0) {
    this.track('search', {
      query,
      filters,
      results,
      timestamp: Date.now()
    });
  }

  // Flush events to server
  async flush() {
    if (this.queue.length === 0) return;

    const events = [...this.queue];
    this.queue = [];

    try {
      // Send to multiple analytics providers
      await Promise.allSettled([
        this.sendToAPI(events),
        this.sendToGoogleAnalytics(events),
        this.sendToCustomProvider(events)
      ]);
    } catch (error) {
      console.error('Analytics flush failed:', error);
      // Re-queue events on failure
      this.queue.unshift(...events);
    }
  }

  async sendToAPI(events) {
    await fetch('/api/analytics/events', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ events }),
      keepalive: true
    });
  }

  sendToGoogleAnalytics(events) {
    if (typeof gtag !== 'undefined') {
      events.forEach(event => {
        gtag('event', event.event, {
          ...event.properties,
          send_to: process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID
        });
      });
    }
  }

  async sendToCustomProvider(events) {
    // Send to custom analytics provider (e.g., Mixpanel, Amplitude)
    if (process.env.NEXT_PUBLIC_ANALYTICS_ENDPOINT) {
      await fetch(process.env.NEXT_PUBLIC_ANALYTICS_ENDPOINT, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${process.env.NEXT_PUBLIC_ANALYTICS_TOKEN}`
        },
        body: JSON.stringify({ events })
      });
    }
  }
}

// Create global instance
export const analytics = new EventTracker();

// React hooks
export function useAnalytics() {
  return {
    track: analytics.track.bind(analytics),
    identify: analytics.identify.bind(analytics),
    setProperties: analytics.setProperties.bind(analytics),
    trackPurchase: analytics.trackPurchase.bind(analytics),
    trackSearch: analytics.trackSearch.bind(analytics)
  };
}

// Higher-order component for automatic tracking
export function withAnalytics(WrappedComponent, eventName) {
  return function AnalyticsWrapper(props) {
    const { track } = useAnalytics();

    useEffect(() => {
      if (eventName) {
        track(eventName, { component: WrappedComponent.name });
      }
    }, [track]);

    return <WrappedComponent {...props} />;
  };
}
```

### **📊 Analytics Dashboard**

```jsx
// pages/admin/analytics.js - Analytics dashboard
import { useState, useEffect } from 'react';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/Card';
import { LineChart, BarChart, PieChart } from '@/components/charts';

export default function AnalyticsDashboard() {
  const [metrics, setMetrics] = useState({
    pageViews: [],
    userSessions: [],
    topPages: [],
    conversionRates: [],
    realTimeUsers: 0,
    performanceMetrics: []
  });

  const [timeRange, setTimeRange] = useState('7d');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadAnalyticsData();
  }, [timeRange]);

  const loadAnalyticsData = async () => {
    setLoading(true);
    try {
      const response = await fetch(`/api/admin/analytics?range=${timeRange}`);
      const data = await response.json();
      setMetrics(data);
    } catch (error) {
      console.error('Failed to load analytics:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6 space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold">Analytics Dashboard</h1>
        <select 
          value={timeRange} 
          onChange={(e) => setTimeRange(e.target.value)}
          className="px-4 py-2 border rounded-md"
        >
          <option value="1d">Last 24 hours</option>
          <option value="7d">Last 7 days</option>
          <option value="30d">Last 30 days</option>
          <option value="90d">Last 90 days</option>
        </select>
      </div>

      {/* Key Metrics */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
        <MetricCard
          title="Page Views"
          value={metrics.pageViews.reduce((sum, item) => sum + item.value, 0)}
          change="+12.5%"
          positive={true}
        />
        <MetricCard
          title="Unique Visitors"
          value={metrics.userSessions.length}
          change="+8.2%"
          positive={true}
        />
        <MetricCard
          title="Bounce Rate"
          value="34.2%"
          change="-2.1%"
          positive={true}
        />
        <MetricCard
          title="Avg. Session Duration"
          value="4m 32s"
          change="+15.3%"
          positive={true}
        />
      </div>

      {/* Charts */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <Card>
          <CardHeader>
            <CardTitle>Page Views Over Time</CardTitle>
          </CardHeader>
          <CardContent>
            <LineChart 
              data={metrics.pageViews}
              xKey="date"
              yKey="value"
              height={300}
            />
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Top Pages</CardTitle>
          </CardHeader>
          <CardContent>
            <BarChart 
              data={metrics.topPages}
              xKey="page"
              yKey="views"
              height={300}
            />
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>User Sessions</CardTitle>
          </CardHeader>
          <CardContent>
            <LineChart 
              data={metrics.userSessions}
              xKey="hour"
              yKey="sessions"
              height={300}
            />
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Conversion Funnel</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-4">
              {metrics.conversionRates.map((step, index) => (
                <div key={index} className="flex items-center justify-between">
                  <span className="font-medium">{step.name}</span>
                  <div className="flex items-center space-x-2">
                    <div className="w-32 bg-gray-200 rounded-full h-2">
                      <div 
                        className="bg-blue-600 h-2 rounded-full"
                        style={{ width: `${step.rate}%` }}
                      />
                    </div>
                    <span className="text-sm text-gray-600">{step.rate}%</span>
                  </div>
                </div>
              ))}
            </div>
          </CardContent>
        </Card>
      </div>

      {/* Real-time metrics */}
      <Card>
        <CardHeader>
          <CardTitle>Real-time Activity</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            <div className="text-center">
              <div className="text-3xl font-bold text-green-600">
                {metrics.realTimeUsers}
              </div>
              <div className="text-sm text-gray-600">Active Users</div>
            </div>
            <div className="text-center">
              <div className="text-3xl font-bold text-blue-600">
                {metrics.realTimePageViews || 0}
              </div>
              <div className="text-sm text-gray-600">Page Views (last hour)</div>
            </div>
            <div className="text-center">
              <div className="text-3xl font-bold text-purple-600">
                {metrics.realTimeEvents || 0}
              </div>
              <div className="text-sm text-gray-600">Events (last hour)</div>
            </div>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}

function MetricCard({ title, value, change, positive }) {
  return (
    <Card>
      <CardContent className="p-6">
        <div className="flex items-center justify-between">
          <div>
            <p className="text-sm font-medium text-gray-600">{title}</p>
            <p className="text-2xl font-bold">{value}</p>
          </div>
          <div className={`text-sm font-medium ${
            positive ? 'text-green-600' : 'text-red-600'
          }`}>
            {change}
          </div>
        </div>
      </CardContent>
    </Card>
  );
}

// API endpoint for analytics data
// pages/api/admin/analytics.js
import { prisma } from '@/lib/prisma';

export default async function handler(req, res) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { range = '7d' } = req.query;
    const days = parseInt(range.replace('d', ''));
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - days);

    // Page views
    const pageViews = await prisma.analyticsEvent.groupBy({
      by: ['createdAt'],
      where: {
        event: 'page_view',
        createdAt: { gte: startDate }
      },
      _count: { id: true },
      orderBy: { createdAt: 'asc' }
    });

    // Top pages
    const topPages = await prisma.analyticsEvent.groupBy({
      by: ['properties'],
      where: {
        event: 'page_view',
        createdAt: { gte: startDate }
      },
      _count: { id: true },
      orderBy: { _count: { id: 'desc' } },
      take: 10
    });

    // User sessions
    const userSessions = await prisma.analyticsEvent.groupBy({
      by: ['sessionId'],
      where: {
        createdAt: { gte: startDate }
      },
      _count: { id: true }
    });

    // Performance metrics
    const performanceMetrics = await prisma.performanceMetric.findMany({
      where: {
        timestamp: { gte: startDate }
      },
      orderBy: { timestamp: 'desc' },
      take: 1000
    });

    res.status(200).json({
      pageViews: pageViews.map(pv => ({
        date: pv.createdAt.toISOString().split('T')[0],
        value: pv._count.id
      })),
      topPages: topPages.map(tp => ({
        page: JSON.parse(tp.properties).page || 'Unknown',
        views: tp._count.id
      })),
      userSessions: userSessions.map(us => ({
        session: us.sessionId,
        events: us._count.id
      })),
      conversionRates: [
        { name: 'Page View', rate: 100 },
        { name: 'Product View', rate: 45 },
        { name: 'Add to Cart', rate: 12 },
        { name: 'Checkout', rate: 8 },
        { name: 'Purchase', rate: 6 }
      ],
      realTimeUsers: await getRealTimeUsers(),
      performanceMetrics
    });
  } catch (error) {
    console.error('Analytics API error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}

async function getRealTimeUsers() {
  const fifteenMinutesAgo = new Date(Date.now() - 15 * 60 * 1000);
  
  const activeUsers = await prisma.analyticsEvent.findMany({
    where: {
      createdAt: { gte: fifteenMinutesAgo }
    },
    distinct: ['sessionId']
  });

  return activeUsers.length;
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Performance Monitoring**
1. ตั้งค่า Core Web Vitals monitoring
2. เพิ่ม custom performance metrics
3. สร้าง performance dashboard
4. เพิ่ม performance budgets

### **🎯 แบบฝึกหัดที่ 2: Error Tracking**
1. ตั้งค่า error tracking system
2. เพิ่ม error boundaries
3. สร้าง error reporting dashboard
4. เพิ่ม alerting rules

### **🎯 แบบฝึกหัดที่ 3: User Analytics**
1. สร้าง event tracking system
2. เพิ่ม user behavior analysis
3. สร้าง conversion funnel
4. ทดสอบ A/B testing

### **🎯 แบบฝึกหัดที่ 4: Real-time Monitoring**
1. ตั้งค่า real-time dashboards
2. เพิ่ม real-time alerts
3. สร้าง health check systems
4. ทดสอบ monitoring integration

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 20: Deployment and DevOps](./20-deployment.md)

**บทต่อไป**: [บทที่ 22: Advanced Patterns and Architecture](./22-advanced-patterns.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Web Vitals](https://web.dev/vitals/)
- [Google Analytics](https://developers.google.com/analytics)
- [Performance Observer API](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver)
- [Real User Monitoring](https://developer.mozilla.org/en-US/docs/Web/Performance)
