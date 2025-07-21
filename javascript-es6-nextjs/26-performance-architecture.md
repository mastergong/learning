# บทที่ 26: Performance Architecture

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ advanced performance optimization techniques
- ทำความเข้าใจ caching strategies และ memory management
- เรียนรู้ load balancing และ scaling patterns
- สร้าง high-performance applications ด้วย architectural best practices

## 🚀 Advanced Performance Optimization

### **⚡ Performance Monitoring System**

```javascript
// ============= PERFORMANCE MONITORING =============
// lib/performance/performance-monitor.js

export class PerformanceMonitor {
  constructor(options = {}) {
    this.options = {
      sampleRate: 0.1, // 10% sampling
      bufferSize: 1000,
      flushInterval: 5000,
      enableUserTiming: true,
      enableResourceTiming: true,
      enableNavigationTiming: true,
      endpoint: '/api/performance',
      ...options
    };
    
    this.metrics = new Map();
    this.buffer = [];
    this.observers = new Map();
    this.timers = new Map();
    this.marks = new Map();
    
    this.initialize();
  }
  
  initialize() {
    if (typeof window === 'undefined') return;
    
    // Setup performance observers
    this.setupPerformanceObservers();
    
    // Setup periodic flushing
    this.setupPeriodicFlush();
    
    // Setup page lifecycle monitoring
    this.setupPageLifecycle();
    
    // Setup error monitoring
    this.setupErrorMonitoring();
  }
  
  setupPerformanceObservers() {
    if (!window.PerformanceObserver) return;
    
    // Navigation timing
    if (this.options.enableNavigationTiming) {
      this.observeEntryTypes(['navigation']);
    }
    
    // Resource timing
    if (this.options.enableResourceTiming) {
      this.observeEntryTypes(['resource']);
    }
    
    // User timing
    if (this.options.enableUserTiming) {
      this.observeEntryTypes(['measure', 'mark']);
    }
    
    // Paint timing
    this.observeEntryTypes(['paint']);
    
    // Layout shift
    this.observeEntryTypes(['layout-shift']);
    
    // Largest contentful paint
    this.observeEntryTypes(['largest-contentful-paint']);
    
    // First input delay
    this.observeEntryTypes(['first-input']);
  }
  
  observeEntryTypes(entryTypes) {
    try {
      const observer = new PerformanceObserver(list => {
        for (const entry of list.getEntries()) {
          this.processPerformanceEntry(entry);
        }
      });
      
      observer.observe({ entryTypes });
      this.observers.set(entryTypes.join(','), observer);
    } catch (error) {
      console.warn('Failed to observe performance entries:', entryTypes, error);
    }
  }
  
  processPerformanceEntry(entry) {
    const metric = {
      type: entry.entryType,
      name: entry.name,
      startTime: entry.startTime,
      duration: entry.duration,
      timestamp: Date.now(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      connectionType: this.getConnectionType()
    };
    
    // Add type-specific data
    switch (entry.entryType) {
      case 'navigation':
        metric.navigationData = this.extractNavigationData(entry);
        break;
      case 'resource':
        metric.resourceData = this.extractResourceData(entry);
        break;
      case 'largest-contentful-paint':
        metric.lcpData = this.extractLCPData(entry);
        break;
      case 'layout-shift':
        metric.clsData = this.extractCLSData(entry);
        break;
      case 'first-input':
        metric.fidData = this.extractFIDData(entry);
        break;
    }
    
    this.addMetric(metric);
  }
  
  extractNavigationData(entry) {
    return {
      domContentLoaded: entry.domContentLoadedEventEnd - entry.domContentLoadedEventStart,
      loadComplete: entry.loadEventEnd - entry.loadEventStart,
      dns: entry.domainLookupEnd - entry.domainLookupStart,
      tcp: entry.connectEnd - entry.connectStart,
      ssl: entry.secureConnectionStart > 0 ? entry.connectEnd - entry.secureConnectionStart : 0,
      ttfb: entry.responseStart - entry.requestStart,
      download: entry.responseEnd - entry.responseStart,
      dom: entry.domComplete - entry.domLoading,
      redirectCount: entry.redirectCount,
      type: entry.type
    };
  }
  
  extractResourceData(entry) {
    return {
      initiatorType: entry.initiatorType,
      transferSize: entry.transferSize || 0,
      encodedBodySize: entry.encodedBodySize || 0,
      decodedBodySize: entry.decodedBodySize || 0,
      dns: entry.domainLookupEnd - entry.domainLookupStart,
      tcp: entry.connectEnd - entry.connectStart,
      ssl: entry.secureConnectionStart > 0 ? entry.connectEnd - entry.secureConnectionStart : 0,
      ttfb: entry.responseStart - entry.requestStart,
      download: entry.responseEnd - entry.responseStart,
      cached: entry.transferSize === 0 && entry.decodedBodySize > 0
    };
  }
  
  extractLCPData(entry) {
    return {
      renderTime: entry.renderTime,
      loadTime: entry.loadTime,
      size: entry.size,
      id: entry.id,
      url: entry.url
    };
  }
  
  extractCLSData(entry) {
    return {
      value: entry.value,
      hadRecentInput: entry.hadRecentInput,
      sources: entry.sources?.map(source => ({
        node: source.node?.tagName || 'unknown',
        currentRect: source.currentRect,
        previousRect: source.previousRect
      }))
    };
  }
  
  extractFIDData(entry) {
    return {
      processingStart: entry.processingStart,
      processingEnd: entry.processingEnd,
      processingTime: entry.processingEnd - entry.processingStart,
      target: entry.target?.tagName || 'unknown'
    };
  }
  
  // Custom timing API
  mark(name) {
    if (typeof performance !== 'undefined' && performance.mark) {
      performance.mark(name);
    }
    
    this.marks.set(name, Date.now());
    return this;
  }
  
  measure(name, startMark, endMark = null) {
    const startTime = this.marks.get(startMark);
    const endTime = endMark ? this.marks.get(endMark) : Date.now();
    
    if (startTime) {
      const duration = endTime - startTime;
      
      if (typeof performance !== 'undefined' && performance.measure) {
        try {
          performance.measure(name, startMark, endMark);
        } catch (error) {
          // Fallback if marks don't exist in performance API
        }
      }
      
      this.addMetric({
        type: 'custom-measure',
        name,
        startTime,
        duration,
        timestamp: Date.now()
      });
      
      return duration;
    }
    
    return null;
  }
  
  time(name) {
    this.timers.set(name, Date.now());
    return this;
  }
  
  timeEnd(name) {
    const startTime = this.timers.get(name);
    if (startTime) {
      const duration = Date.now() - startTime;
      this.timers.delete(name);
      
      this.addMetric({
        type: 'custom-timer',
        name,
        duration,
        timestamp: Date.now()
      });
      
      return duration;
    }
    return null;
  }
  
  // Memory monitoring
  getMemoryUsage() {
    if (typeof performance !== 'undefined' && performance.memory) {
      return {
        usedJSHeapSize: performance.memory.usedJSHeapSize,
        totalJSHeapSize: performance.memory.totalJSHeapSize,
        jsHeapSizeLimit: performance.memory.jsHeapSizeLimit,
        timestamp: Date.now()
      };
    }
    return null;
  }
  
  monitorMemory(interval = 5000) {
    const monitor = () => {
      const memoryUsage = this.getMemoryUsage();
      if (memoryUsage) {
        this.addMetric({
          type: 'memory',
          name: 'heap-usage',
          data: memoryUsage,
          timestamp: Date.now()
        });
      }
    };
    
    monitor(); // Initial measurement
    const timer = setInterval(monitor, interval);
    
    return () => clearInterval(timer);
  }
  
  addMetric(metric) {
    // Sampling
    if (Math.random() > this.options.sampleRate) {
      return;
    }
    
    this.buffer.push(metric);
    
    // Auto-flush if buffer is full
    if (this.buffer.length >= this.options.bufferSize) {
      this.flush();
    }
  }
  
  flush() {
    if (this.buffer.length === 0) return;
    
    const metrics = [...this.buffer];
    this.buffer = [];
    
    // Send to analytics endpoint
    this.sendMetrics(metrics);
  }
  
  async sendMetrics(metrics) {
    try {
      await fetch(this.options.endpoint, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          metrics,
          session: this.getSessionId(),
          page: window.location.pathname,
          timestamp: Date.now()
        })
      });
    } catch (error) {
      console.warn('Failed to send performance metrics:', error);
    }
  }
  
  setupPeriodicFlush() {
    setInterval(() => {
      this.flush();
    }, this.options.flushInterval);
  }
  
  setupPageLifecycle() {
    if (typeof window === 'undefined') return;
    
    // Page visibility
    document.addEventListener('visibilitychange', () => {
      this.addMetric({
        type: 'page-lifecycle',
        name: 'visibility-change',
        visible: !document.hidden,
        timestamp: Date.now()
      });
      
      if (document.hidden) {
        this.flush(); // Flush before page becomes hidden
      }
    });
    
    // Page unload
    window.addEventListener('beforeunload', () => {
      this.flush();
    });
    
    // Page freeze/resume (experimental)
    if ('onfreeze' in document) {
      document.addEventListener('freeze', () => {
        this.addMetric({
          type: 'page-lifecycle',
          name: 'freeze',
          timestamp: Date.now()
        });
        this.flush();
      });
      
      document.addEventListener('resume', () => {
        this.addMetric({
          type: 'page-lifecycle',
          name: 'resume',
          timestamp: Date.now()
        });
      });
    }
  }
  
  setupErrorMonitoring() {
    if (typeof window === 'undefined') return;
    
    window.addEventListener('error', event => {
      this.addMetric({
        type: 'error',
        name: 'javascript-error',
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack,
        timestamp: Date.now()
      });
    });
    
    window.addEventListener('unhandledrejection', event => {
      this.addMetric({
        type: 'error',
        name: 'unhandled-promise-rejection',
        reason: event.reason?.toString(),
        stack: event.reason?.stack,
        timestamp: Date.now()
      });
    });
  }
  
  getConnectionType() {
    if (typeof navigator !== 'undefined' && navigator.connection) {
      return {
        effectiveType: navigator.connection.effectiveType,
        downlink: navigator.connection.downlink,
        rtt: navigator.connection.rtt,
        saveData: navigator.connection.saveData
      };
    }
    return null;
  }
  
  getSessionId() {
    let sessionId = sessionStorage.getItem('performance-session-id');
    if (!sessionId) {
      sessionId = `session-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
      sessionStorage.setItem('performance-session-id', sessionId);
    }
    return sessionId;
  }
  
  // Core Web Vitals calculation
  getCoreWebVitals() {
    const vitals = {
      lcp: null, // Largest Contentful Paint
      fid: null, // First Input Delay
      cls: null, // Cumulative Layout Shift
      fcp: null, // First Contentful Paint
      ttfb: null // Time to First Byte
    };
    
    // Get from performance entries
    const entries = performance.getEntriesByType('navigation');
    if (entries.length > 0) {
      const nav = entries[0];
      vitals.ttfb = nav.responseStart - nav.requestStart;
    }
    
    const paintEntries = performance.getEntriesByType('paint');
    const fcpEntry = paintEntries.find(entry => entry.name === 'first-contentful-paint');
    if (fcpEntry) {
      vitals.fcp = fcpEntry.startTime;
    }
    
    return vitals;
  }
  
  // Resource analysis
  analyzeResources() {
    const resources = performance.getEntriesByType('resource');
    const analysis = {
      total: resources.length,
      byType: {},
      large: [],
      slow: [],
      totalSize: 0,
      totalTime: 0
    };
    
    resources.forEach(resource => {
      const type = resource.initiatorType || 'other';
      if (!analysis.byType[type]) {
        analysis.byType[type] = { count: 0, size: 0, time: 0 };
      }
      
      analysis.byType[type].count++;
      analysis.byType[type].size += resource.transferSize || 0;
      analysis.byType[type].time += resource.duration;
      
      analysis.totalSize += resource.transferSize || 0;
      analysis.totalTime += resource.duration;
      
      // Flag large resources (>1MB)
      if (resource.transferSize > 1024 * 1024) {
        analysis.large.push({
          name: resource.name,
          size: resource.transferSize,
          type
        });
      }
      
      // Flag slow resources (>2s)
      if (resource.duration > 2000) {
        analysis.slow.push({
          name: resource.name,
          duration: resource.duration,
          type
        });
      }
    });
    
    return analysis;
  }
  
  destroy() {
    // Clean up observers
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
    
    // Final flush
    this.flush();
    
    // Clear data
    this.metrics.clear();
    this.buffer = [];
    this.timers.clear();
    this.marks.clear();
  }
}

// ============= ADVANCED CACHING =============
// lib/performance/cache-manager.js

export class CacheManager {
  constructor(options = {}) {
    this.options = {
      defaultTTL: 5 * 60 * 1000, // 5 minutes
      maxSize: 100,
      enableLRU: true,
      enableCompression: false,
      storageBackend: 'memory', // memory, localStorage, indexedDB
      enableMetrics: true,
      ...options
    };
    
    this.cache = new Map();
    this.accessTimes = new Map();
    this.metadata = new Map();
    this.metrics = {
      hits: 0,
      misses: 0,
      evictions: 0,
      size: 0
    };
    
    this.storage = this.createStorageBackend();
    this.initialize();
  }
  
  createStorageBackend() {
    switch (this.options.storageBackend) {
      case 'localStorage':
        return new LocalStorageBackend();
      case 'indexedDB':
        return new IndexedDBBackend();
      default:
        return new MemoryBackend();
    }
  }
  
  async initialize() {
    await this.storage.initialize();
    await this.loadFromStorage();
  }
  
  async get(key) {
    // Check memory cache first
    if (this.cache.has(key)) {
      const entry = this.cache.get(key);
      
      // Check if expired
      if (this.isExpired(entry)) {
        this.delete(key);
        this.metrics.misses++;
        return null;
      }
      
      // Update access time for LRU
      if (this.options.enableLRU) {
        this.accessTimes.set(key, Date.now());
      }
      
      this.metrics.hits++;
      return this.deserializeValue(entry.value);
    }
    
    // Check storage backend
    try {
      const stored = await this.storage.get(key);
      if (stored && !this.isExpired(stored)) {
        // Load into memory cache
        this.cache.set(key, stored);
        if (this.options.enableLRU) {
          this.accessTimes.set(key, Date.now());
        }
        
        this.metrics.hits++;
        return this.deserializeValue(stored.value);
      }
    } catch (error) {
      console.warn('Cache storage error:', error);
    }
    
    this.metrics.misses++;
    return null;
  }
  
  async set(key, value, ttl = null) {
    const expiry = ttl !== null ? Date.now() + ttl : Date.now() + this.options.defaultTTL;
    const serializedValue = await this.serializeValue(value);
    
    const entry = {
      value: serializedValue,
      expiry,
      created: Date.now(),
      accessed: Date.now(),
      size: this.calculateSize(serializedValue)
    };
    
    // Check if we need to evict
    if (this.cache.size >= this.options.maxSize) {
      await this.evict();
    }
    
    // Store in memory
    this.cache.set(key, entry);
    this.metadata.set(key, {
      hits: 0,
      lastAccess: Date.now()
    });
    
    if (this.options.enableLRU) {
      this.accessTimes.set(key, Date.now());
    }
    
    // Store in backend
    try {
      await this.storage.set(key, entry);
    } catch (error) {
      console.warn('Cache storage error:', error);
    }
    
    this.updateMetrics();
  }
  
  async delete(key) {
    const deleted = this.cache.delete(key);
    this.accessTimes.delete(key);
    this.metadata.delete(key);
    
    try {
      await this.storage.delete(key);
    } catch (error) {
      console.warn('Cache storage error:', error);
    }
    
    this.updateMetrics();
    return deleted;
  }
  
  async clear() {
    this.cache.clear();
    this.accessTimes.clear();
    this.metadata.clear();
    
    try {
      await this.storage.clear();
    } catch (error) {
      console.warn('Cache storage error:', error);
    }
    
    this.updateMetrics();
  }
  
  async evict() {
    if (this.cache.size === 0) return;
    
    let keyToEvict;
    
    if (this.options.enableLRU) {
      // Evict least recently used
      let oldestTime = Date.now();
      for (const [key, time] of this.accessTimes) {
        if (time < oldestTime) {
          oldestTime = time;
          keyToEvict = key;
        }
      }
    } else {
      // Evict first item (FIFO)
      keyToEvict = this.cache.keys().next().value;
    }
    
    if (keyToEvict) {
      await this.delete(keyToEvict);
      this.metrics.evictions++;
    }
  }
  
  isExpired(entry) {
    return Date.now() > entry.expiry;
  }
  
  async serializeValue(value) {
    if (this.options.enableCompression && typeof value === 'string') {
      // Simple compression (could use pako or similar)
      return this.compress(value);
    }
    
    return typeof value === 'object' ? JSON.stringify(value) : value;
  }
  
  deserializeValue(serializedValue) {
    if (this.options.enableCompression) {
      try {
        return this.decompress(serializedValue);
      } catch (error) {
        // Fallback to regular deserialization
      }
    }
    
    try {
      return JSON.parse(serializedValue);
    } catch (error) {
      return serializedValue;
    }
  }
  
  compress(str) {
    // Simple RLE compression (replace with better algorithm)
    return str.replace(/(.)\1+/g, (match, char) => `${char}${match.length}`);
  }
  
  decompress(str) {
    // Simple RLE decompression
    return str.replace(/(.)\d+/g, (match, char) => {
      const count = parseInt(match.slice(1));
      return char.repeat(count);
    });
  }
  
  calculateSize(value) {
    return new Blob([value]).size;
  }
  
  updateMetrics() {
    this.metrics.size = this.cache.size;
  }
  
  getMetrics() {
    return {
      ...this.metrics,
      hitRate: this.metrics.hits / (this.metrics.hits + this.metrics.misses) || 0
    };
  }
  
  async loadFromStorage() {
    try {
      const keys = await this.storage.keys();
      for (const key of keys) {
        const entry = await this.storage.get(key);
        if (entry && !this.isExpired(entry)) {
          this.cache.set(key, entry);
          if (this.options.enableLRU) {
            this.accessTimes.set(key, entry.accessed);
          }
        }
      }
    } catch (error) {
      console.warn('Failed to load cache from storage:', error);
    }
  }
  
  // Cache strategies
  async memoize(fn, key, ttl = null) {
    const cached = await this.get(key);
    if (cached !== null) {
      return cached;
    }
    
    const result = await fn();
    await this.set(key, result, ttl);
    return result;
  }
  
  async invalidatePattern(pattern) {
    const regex = new RegExp(pattern);
    const keysToDelete = [];
    
    for (const key of this.cache.keys()) {
      if (regex.test(key)) {
        keysToDelete.push(key);
      }
    }
    
    await Promise.all(keysToDelete.map(key => this.delete(key)));
    return keysToDelete.length;
  }
  
  async refresh(key, fn, ttl = null) {
    const result = await fn();
    await this.set(key, result, ttl);
    return result;
  }
  
  // Batch operations
  async mget(keys) {
    const results = await Promise.all(
      keys.map(async key => ({
        key,
        value: await this.get(key)
      }))
    );
    
    return Object.fromEntries(
      results.map(({ key, value }) => [key, value])
    );
  }
  
  async mset(entries, ttl = null) {
    await Promise.all(
      Object.entries(entries).map(([key, value]) =>
        this.set(key, value, ttl)
      )
    );
  }
}

// Storage backends
class MemoryBackend {
  constructor() {
    this.data = new Map();
  }
  
  async initialize() {}
  
  async get(key) {
    return this.data.get(key);
  }
  
  async set(key, value) {
    this.data.set(key, value);
  }
  
  async delete(key) {
    return this.data.delete(key);
  }
  
  async clear() {
    this.data.clear();
  }
  
  async keys() {
    return Array.from(this.data.keys());
  }
}

class LocalStorageBackend {
  constructor() {
    this.prefix = 'cache:';
  }
  
  async initialize() {}
  
  async get(key) {
    try {
      const item = localStorage.getItem(this.prefix + key);
      return item ? JSON.parse(item) : null;
    } catch (error) {
      return null;
    }
  }
  
  async set(key, value) {
    try {
      localStorage.setItem(this.prefix + key, JSON.stringify(value));
    } catch (error) {
      // Handle quota exceeded
      throw error;
    }
  }
  
  async delete(key) {
    localStorage.removeItem(this.prefix + key);
    return true;
  }
  
  async clear() {
    const keys = Object.keys(localStorage);
    keys.forEach(key => {
      if (key.startsWith(this.prefix)) {
        localStorage.removeItem(key);
      }
    });
  }
  
  async keys() {
    const keys = Object.keys(localStorage);
    return keys
      .filter(key => key.startsWith(this.prefix))
      .map(key => key.slice(this.prefix.length));
  }
}

class IndexedDBBackend {
  constructor() {
    this.dbName = 'CacheDB';
    this.storeName = 'cache';
    this.version = 1;
    this.db = null;
  }
  
  async initialize() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };
      
      request.onupgradeneeded = event => {
        const db = event.target.result;
        if (!db.objectStoreNames.contains(this.storeName)) {
          db.createObjectStore(this.storeName);
        }
      };
    });
  }
  
  async get(key) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readonly');
      const store = transaction.objectStore(this.storeName);
      const request = store.get(key);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
    });
  }
  
  async set(key, value) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.put(value, key);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }
  
  async delete(key) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.delete(key);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(true);
    });
  }
  
  async clear() {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.clear();
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }
  
  async keys() {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readonly');
      const store = transaction.objectStore(this.storeName);
      const request = store.getAllKeys();
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
    });
  }
}
```

## 🏗️ Load Balancing & Scaling

### **⚖️ Load Balancer Implementation**

```javascript
// ============= LOAD BALANCING =============
// lib/performance/load-balancer.js

export class LoadBalancer {
  constructor(options = {}) {
    this.options = {
      algorithm: 'round-robin', // round-robin, least-connections, weighted, ip-hash
      healthCheckInterval: 30000,
      healthCheckTimeout: 5000,
      healthCheckPath: '/health',
      retryAttempts: 3,
      retryDelay: 1000,
      enableCircuitBreaker: true,
      circuitBreakerThreshold: 5,
      circuitBreakerTimeout: 60000,
      ...options
    };
    
    this.servers = new Map();
    this.currentIndex = 0;
    this.stats = new Map();
    this.circuitBreakers = new Map();
    this.healthCheckTimer = null;
    
    this.initialize();
  }
  
  initialize() {
    this.startHealthChecks();
  }
  
  addServer(id, config) {
    const server = {
      id,
      url: config.url,
      weight: config.weight || 1,
      maxConnections: config.maxConnections || 100,
      currentConnections: 0,
      healthy: true,
      lastHealthCheck: null,
      region: config.region || 'default',
      ...config
    };
    
    this.servers.set(id, server);
    this.stats.set(id, {
      requests: 0,
      errors: 0,
      totalResponseTime: 0,
      avgResponseTime: 0,
      lastError: null
    });
    
    if (this.options.enableCircuitBreaker) {
      this.circuitBreakers.set(id, {
        state: 'closed', // closed, open, half-open
        failures: 0,
        lastFailure: null,
        nextAttempt: null
      });
    }
    
    return server;
  }
  
  removeServer(id) {
    this.servers.delete(id);
    this.stats.delete(id);
    this.circuitBreakers.delete(id);
  }
  
  async getServer(requestInfo = {}) {
    const availableServers = this.getAvailableServers(requestInfo);
    
    if (availableServers.length === 0) {
      throw new Error('No available servers');
    }
    
    let selectedServer;
    
    switch (this.options.algorithm) {
      case 'round-robin':
        selectedServer = this.roundRobin(availableServers);
        break;
      case 'least-connections':
        selectedServer = this.leastConnections(availableServers);
        break;
      case 'weighted':
        selectedServer = this.weighted(availableServers);
        break;
      case 'ip-hash':
        selectedServer = this.ipHash(availableServers, requestInfo.clientIp);
        break;
      case 'geographic':
        selectedServer = this.geographic(availableServers, requestInfo.region);
        break;
      default:
        selectedServer = availableServers[0];
    }
    
    return selectedServer;
  }
  
  getAvailableServers(requestInfo) {
    return Array.from(this.servers.values()).filter(server => {
      // Check if server is healthy
      if (!server.healthy) return false;
      
      // Check circuit breaker
      if (this.options.enableCircuitBreaker) {
        const breaker = this.circuitBreakers.get(server.id);
        if (breaker.state === 'open') {
          if (Date.now() < breaker.nextAttempt) {
            return false;
          } else {
            // Try half-open
            breaker.state = 'half-open';
          }
        }
      }
      
      // Check connection limit
      if (server.currentConnections >= server.maxConnections) {
        return false;
      }
      
      return true;
    });
  }
  
  roundRobin(servers) {
    const server = servers[this.currentIndex % servers.length];
    this.currentIndex = (this.currentIndex + 1) % servers.length;
    return server;
  }
  
  leastConnections(servers) {
    return servers.reduce((least, current) => 
      current.currentConnections < least.currentConnections ? current : least
    );
  }
  
  weighted(servers) {
    const totalWeight = servers.reduce((sum, server) => sum + server.weight, 0);
    let random = Math.random() * totalWeight;
    
    for (const server of servers) {
      random -= server.weight;
      if (random <= 0) {
        return server;
      }
    }
    
    return servers[servers.length - 1];
  }
  
  ipHash(servers, clientIp) {
    if (!clientIp) return this.roundRobin(servers);
    
    let hash = 0;
    for (let i = 0; i < clientIp.length; i++) {
      hash = ((hash << 5) - hash + clientIp.charCodeAt(i)) & 0xffffffff;
    }
    
    const index = Math.abs(hash) % servers.length;
    return servers[index];
  }
  
  geographic(servers, region) {
    if (!region) return this.roundRobin(servers);
    
    // Prefer servers in the same region
    const localServers = servers.filter(server => server.region === region);
    if (localServers.length > 0) {
      return this.roundRobin(localServers);
    }
    
    return this.roundRobin(servers);
  }
  
  async makeRequest(requestInfo, options = {}) {
    const maxRetries = options.retries || this.options.retryAttempts;
    let lastError;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const server = await this.getServer(requestInfo);
        return await this.executeRequest(server, requestInfo, options);
      } catch (error) {
        lastError = error;
        
        if (attempt < maxRetries) {
          await this.delay(this.options.retryDelay * Math.pow(2, attempt));
        }
      }
    }
    
    throw lastError;
  }
  
  async executeRequest(server, requestInfo, options) {
    const startTime = Date.now();
    
    try {
      // Increment connection count
      server.currentConnections++;
      
      // Make the actual request
      const response = await this.performRequest(server, requestInfo, options);
      
      // Record success
      this.recordSuccess(server, Date.now() - startTime);
      
      return response;
    } catch (error) {
      // Record failure
      this.recordFailure(server, error);
      throw error;
    } finally {
      // Decrement connection count
      server.currentConnections--;
    }
  }
  
  async performRequest(server, requestInfo, options) {
    const url = `${server.url}${requestInfo.path || ''}`;
    const requestOptions = {
      method: requestInfo.method || 'GET',
      headers: requestInfo.headers || {},
      body: requestInfo.body,
      timeout: options.timeout || 30000,
      ...options.fetchOptions
    };
    
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), requestOptions.timeout);
    
    try {
      const response = await fetch(url, {
        ...requestOptions,
        signal: controller.signal
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return response;
    } finally {
      clearTimeout(timeoutId);
    }
  }
  
  recordSuccess(server, responseTime) {
    const stats = this.stats.get(server.id);
    stats.requests++;
    stats.totalResponseTime += responseTime;
    stats.avgResponseTime = stats.totalResponseTime / stats.requests;
    
    // Reset circuit breaker on success
    if (this.options.enableCircuitBreaker) {
      const breaker = this.circuitBreakers.get(server.id);
      if (breaker.state === 'half-open') {
        breaker.state = 'closed';
        breaker.failures = 0;
      }
    }
  }
  
  recordFailure(server, error) {
    const stats = this.stats.get(server.id);
    stats.errors++;
    stats.lastError = error;
    
    // Update circuit breaker
    if (this.options.enableCircuitBreaker) {
      const breaker = this.circuitBreakers.get(server.id);
      breaker.failures++;
      breaker.lastFailure = Date.now();
      
      if (breaker.failures >= this.options.circuitBreakerThreshold) {
        breaker.state = 'open';
        breaker.nextAttempt = Date.now() + this.options.circuitBreakerTimeout;
      }
    }
  }
  
  startHealthChecks() {
    if (this.healthCheckTimer) {
      clearInterval(this.healthCheckTimer);
    }
    
    this.healthCheckTimer = setInterval(() => {
      this.performHealthChecks();
    }, this.options.healthCheckInterval);
    
    // Initial health check
    this.performHealthChecks();
  }
  
  async performHealthChecks() {
    const healthCheckPromises = Array.from(this.servers.values()).map(
      server => this.checkServerHealth(server)
    );
    
    await Promise.allSettled(healthCheckPromises);
  }
  
  async checkServerHealth(server) {
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(
        () => controller.abort(),
        this.options.healthCheckTimeout
      );
      
      const response = await fetch(`${server.url}${this.options.healthCheckPath}`, {
        method: 'GET',
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      const wasHealthy = server.healthy;
      server.healthy = response.ok;
      server.lastHealthCheck = Date.now();
      
      if (!wasHealthy && server.healthy) {
        console.log(`Server ${server.id} is now healthy`);
      } else if (wasHealthy && !server.healthy) {
        console.log(`Server ${server.id} is now unhealthy`);
      }
    } catch (error) {
      const wasHealthy = server.healthy;
      server.healthy = false;
      server.lastHealthCheck = Date.now();
      
      if (wasHealthy) {
        console.log(`Server ${server.id} health check failed:`, error.message);
      }
    }
  }
  
  getStats() {
    const serverStats = {};
    
    for (const [id, server] of this.servers) {
      const stats = this.stats.get(id);
      const breaker = this.circuitBreakers.get(id);
      
      serverStats[id] = {
        server: {
          url: server.url,
          healthy: server.healthy,
          currentConnections: server.currentConnections,
          lastHealthCheck: server.lastHealthCheck
        },
        stats: { ...stats },
        circuitBreaker: breaker ? { ...breaker } : null
      };
    }
    
    return {
      algorithm: this.options.algorithm,
      totalServers: this.servers.size,
      healthyServers: Array.from(this.servers.values()).filter(s => s.healthy).length,
      servers: serverStats
    };
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  destroy() {
    if (this.healthCheckTimer) {
      clearInterval(this.healthCheckTimer);
    }
  }
}

// ============= CONNECTION POOLING =============
// lib/performance/connection-pool.js

export class ConnectionPool {
  constructor(options = {}) {
    this.options = {
      maxConnections: 10,
      minConnections: 2,
      acquireTimeout: 30000,
      idleTimeout: 300000,
      maxLifetime: 3600000,
      validationInterval: 60000,
      enableMetrics: true,
      factory: null, // Connection factory function
      destroyer: null, // Connection destroyer function
      validator: null, // Connection validator function
      ...options
    };
    
    this.connections = [];
    this.available = [];
    this.pending = [];
    this.metrics = {
      created: 0,
      destroyed: 0,
      acquired: 0,
      released: 0,
      timeouts: 0,
      validationFailures: 0
    };
    
    this.validationTimer = null;
    this.initialize();
  }
  
  async initialize() {
    // Create minimum connections
    for (let i = 0; i < this.options.minConnections; i++) {
      await this.createConnection();
    }
    
    // Start validation timer
    this.startValidation();
  }
  
  async acquire(timeout = null) {
    const acquisitionTimeout = timeout || this.options.acquireTimeout;
    
    return new Promise((resolve, reject) => {
      const timeoutId = setTimeout(() => {
        this.removePendingRequest(resolve);
        this.metrics.timeouts++;
        reject(new Error('Connection acquisition timeout'));
      }, acquisitionTimeout);
      
      const request = {
        resolve: (connection) => {
          clearTimeout(timeoutId);
          resolve(connection);
        },
        reject: (error) => {
          clearTimeout(timeoutId);
          reject(error);
        },
        timestamp: Date.now()
      };
      
      this.pending.push(request);
      this.processPendingRequests();
    });
  }
  
  async processPendingRequests() {
    while (this.pending.length > 0 && this.available.length > 0) {
      const request = this.pending.shift();
      const connection = this.available.shift();
      
      if (await this.validateConnection(connection)) {
        connection.lastUsed = Date.now();
        this.metrics.acquired++;
        request.resolve(connection);
      } else {
        // Connection is invalid, destroy it and create a new one
        await this.destroyConnection(connection);
        if (this.connections.length < this.options.maxConnections) {
          await this.createConnection();
        }
        
        // Retry with next available connection
        this.pending.unshift(request);
      }
    }
    
    // Create new connections if needed and possible
    if (this.pending.length > 0 && this.connections.length < this.options.maxConnections) {
      await this.createConnection();
      this.processPendingRequests();
    }
  }
  
  async release(connection) {
    if (!this.connections.includes(connection)) {
      throw new Error('Connection not from this pool');
    }
    
    connection.lastUsed = Date.now();
    this.available.push(connection);
    this.metrics.released++;
    
    // Process any pending requests
    this.processPendingRequests();
  }
  
  async createConnection() {
    if (!this.options.factory) {
      throw new Error('Connection factory not provided');
    }
    
    try {
      const connection = await this.options.factory();
      
      const pooledConnection = {
        connection,
        created: Date.now(),
        lastUsed: Date.now(),
        id: this.generateId()
      };
      
      this.connections.push(pooledConnection);
      this.available.push(pooledConnection);
      this.metrics.created++;
      
      return pooledConnection;
    } catch (error) {
      throw new Error(`Failed to create connection: ${error.message}`);
    }
  }
  
  async destroyConnection(pooledConnection) {
    // Remove from all arrays
    this.removeFromArray(this.connections, pooledConnection);
    this.removeFromArray(this.available, pooledConnection);
    
    try {
      if (this.options.destroyer) {
        await this.options.destroyer(pooledConnection.connection);
      }
      this.metrics.destroyed++;
    } catch (error) {
      console.warn('Error destroying connection:', error);
    }
  }
  
  async validateConnection(pooledConnection) {
    try {
      // Check age
      const age = Date.now() - pooledConnection.created;
      if (age > this.options.maxLifetime) {
        return false;
      }
      
      // Check idle time
      const idleTime = Date.now() - pooledConnection.lastUsed;
      if (idleTime > this.options.idleTimeout) {
        return false;
      }
      
      // Custom validation
      if (this.options.validator) {
        const isValid = await this.options.validator(pooledConnection.connection);
        if (!isValid) {
          this.metrics.validationFailures++;
          return false;
        }
      }
      
      return true;
    } catch (error) {
      this.metrics.validationFailures++;
      return false;
    }
  }
  
  startValidation() {
    if (this.validationTimer) {
      clearInterval(this.validationTimer);
    }
    
    this.validationTimer = setInterval(() => {
      this.validateAllConnections();
    }, this.options.validationInterval);
  }
  
  async validateAllConnections() {
    const connectionsToDestroy = [];
    
    for (const connection of this.connections) {
      if (!await this.validateConnection(connection)) {
        connectionsToDestroy.push(connection);
      }
    }
    
    // Destroy invalid connections
    for (const connection of connectionsToDestroy) {
      await this.destroyConnection(connection);
    }
    
    // Maintain minimum connections
    while (this.connections.length < this.options.minConnections) {
      await this.createConnection();
    }
  }
  
  removePendingRequest(resolve) {
    const index = this.pending.findIndex(req => req.resolve === resolve);
    if (index !== -1) {
      this.pending.splice(index, 1);
    }
  }
  
  removeFromArray(array, item) {
    const index = array.indexOf(item);
    if (index !== -1) {
      array.splice(index, 1);
    }
  }
  
  generateId() {
    return `conn_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  getStats() {
    return {
      total: this.connections.length,
      available: this.available.length,
      pending: this.pending.length,
      metrics: { ...this.metrics },
      options: {
        maxConnections: this.options.maxConnections,
        minConnections: this.options.minConnections
      }
    };
  }
  
  async drain() {
    // Stop accepting new requests
    clearInterval(this.validationTimer);
    
    // Reject pending requests
    while (this.pending.length > 0) {
      const request = this.pending.shift();
      request.reject(new Error('Pool is draining'));
    }
    
    // Destroy all connections
    while (this.connections.length > 0) {
      await this.destroyConnection(this.connections[0]);
    }
  }
  
  async destroy() {
    await this.drain();
  }
}
```

## 🎯 แบบฝึกหัด

### **🚀 แบบฝึกหัดที่ 1: Performance Optimization**
สร้าง comprehensive performance system:

```javascript
// TODO: สร้าง performance optimization system

// 1. Bundle analyzer และ optimization
class BundleOptimizer {
  // Code splitting strategies
  // Tree shaking optimization
  // Bundle size analysis
}

// 2. Image optimization pipeline
class ImageOptimizer {
  // WebP conversion
  // Responsive images
  // Lazy loading implementation
}

// 3. Service worker for caching
class ServiceWorkerManager {
  // Cache strategies
  // Background sync
  // Push notifications
}
```

### **📊 แบบฝึกหัดที่ 2: Monitoring Dashboard**
สร้าง real-time monitoring system:

```javascript
// TODO: สร้าง monitoring dashboard

// 1. Real-time metrics
class MetricsDashboard {
  // Live performance data
  // Alert systems
  // Trend analysis
}

// 2. User experience monitoring
class UXMonitor {
  // Core Web Vitals tracking
  // User journey analytics
  // Error boundary reporting
}

// 3. Resource utilization
class ResourceMonitor {
  // Memory usage tracking
  // CPU utilization
  // Network performance
}
```

### **⚖️ แบบฝึกหัดที่ 3: Auto-scaling System**
สร้าง automatic scaling solution:

```javascript
// TODO: สร้าง auto-scaling system

// 1. Load prediction
class LoadPredictor {
  // Traffic pattern analysis
  // Predictive scaling
  // Seasonal adjustments
}

// 2. Container orchestration
class ContainerOrchestrator {
  // Docker management
  // Kubernetes integration
  // Service mesh configuration
}

// 3. Database scaling
class DatabaseScaler {
  // Read replica management
  // Sharding strategies
  // Connection pooling
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 25: Advanced Patterns](./25-advanced-patterns.md)

**บทถัดไป**: [บทที่ 27: Scalability](./27-scalability.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Web Performance Working Group](https://w3c.github.io/web-performance/)
- [Core Web Vitals](https://web.dev/vitals/)
- [Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance)
- [Load Balancing Algorithms](https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Performance Monitoring** - comprehensive metrics collection และ analysis
- **Advanced Caching** - multi-level caching strategies และ optimization
- **Load Balancing** - advanced algorithms และ health checking
- **Connection Pooling** - resource management และ optimization
- **Scaling Patterns** - architectural approaches สำหรับ high-performance applications

Performance Architecture เป็นพื้นฐานสำคัญสำหรับ scalable applications! 🚀
