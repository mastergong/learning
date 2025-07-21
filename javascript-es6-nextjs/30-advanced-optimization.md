# บทที่ 30: Advanced Optimization

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ cutting-edge optimization techniques
- ทำความเข้าใจ emerging technologies และ future trends
- เรียนรู้ performance tuning และ advanced profiling
- สร้าง future-proof applications ด้วย latest optimization strategies

## ⚡ Performance Optimization Mastery

### **🚀 Advanced Performance Profiler**

```javascript
// ============= ADVANCED PERFORMANCE PROFILER =============
// lib/optimization/performance-profiler.js

export class AdvancedPerformanceProfiler {
  constructor(options = {}) {
    this.options = {
      enableCPUProfiling: true,
      enableMemoryProfiling: true,
      enableNetworkProfiling: true,
      enableRenderingProfiling: true,
      enableUserTimingAPI: true,
      samplingInterval: 1, // ms
      bufferSize: 10000,
      enableHeatMaps: true,
      enableFlameGraphs: true,
      enableSourceMaps: true,
      enableDebugging: false,
      ...options
    };
    
    this.profiles = new Map();
    this.currentSession = null;
    this.samplingTimer = null;
    this.observers = new Map();
    this.heatMaps = new Map();
    this.bottlenecks = [];
    this.recommendations = [];
    
    this.initialize();
  }
  
  initialize() {
    if (typeof window !== 'undefined') {
      this.setupBrowserProfiling();
    } else {
      this.setupNodeProfiling();
    }
  }
  
  setupBrowserProfiling() {
    // Performance Observer for modern browsers
    if ('PerformanceObserver' in window) {
      this.setupPerformanceObservers();
    }
    
    // User Timing API
    if ('performance' in window && 'mark' in performance) {
      this.setupUserTiming();
    }
    
    // Resource timing
    this.setupResourceTiming();
    
    // Long task observer
    this.setupLongTaskObserver();
    
    // Layout shift observer
    this.setupLayoutShiftObserver();
    
    // Intersection observer for visibility tracking
    this.setupVisibilityTracking();
  }
  
  setupNodeProfiling() {
    // V8 CPU profiler
    if (this.options.enableCPUProfiling) {
      this.setupV8Profiler();
    }
    
    // Memory profiling
    if (this.options.enableMemoryProfiling) {
      this.setupMemoryProfiling();
    }
    
    // Event loop monitoring
    this.setupEventLoopMonitoring();
  }
  
  // Profiling Session Management
  startProfiling(sessionName = 'default') {
    if (this.currentSession) {
      this.stopProfiling();
    }
    
    this.currentSession = {
      name: sessionName,
      startTime: performance.now(),
      endTime: null,
      duration: null,
      samples: [],
      metrics: new Map(),
      callStacks: [],
      memorySnapshots: [],
      networkRequests: [],
      renderingEvents: [],
      userInteractions: []
    };
    
    // Start CPU sampling
    if (this.options.enableCPUProfiling) {
      this.startCPUSampling();
    }
    
    // Start memory monitoring
    if (this.options.enableMemoryProfiling) {
      this.startMemoryMonitoring();
    }
    
    console.log(`Performance profiling started: ${sessionName}`);
    
    return this.currentSession;
  }
  
  stopProfiling() {
    if (!this.currentSession) {
      return null;
    }
    
    this.currentSession.endTime = performance.now();
    this.currentSession.duration = this.currentSession.endTime - this.currentSession.startTime;
    
    // Stop sampling
    this.stopCPUSampling();
    this.stopMemoryMonitoring();
    
    // Analyze results
    const analysis = this.analyzeProfile(this.currentSession);
    
    // Store profile
    this.profiles.set(this.currentSession.name, {
      session: this.currentSession,
      analysis
    });
    
    console.log(`Performance profiling stopped: ${this.currentSession.name}`);
    console.log(`Duration: ${this.currentSession.duration.toFixed(2)}ms`);
    
    const session = this.currentSession;
    this.currentSession = null;
    
    return { session, analysis };
  }
  
  // CPU Profiling
  startCPUSampling() {
    if (typeof window !== 'undefined') {
      this.startBrowserCPUSampling();
    } else {
      this.startNodeCPUSampling();
    }
  }
  
  startBrowserCPUSampling() {
    this.samplingTimer = setInterval(() => {
      const sample = this.captureCPUSample();
      this.currentSession.samples.push(sample);
      
      // Limit buffer size
      if (this.currentSession.samples.length > this.options.bufferSize) {
        this.currentSession.samples.shift();
      }
    }, this.options.samplingInterval);
  }
  
  startNodeCPUSampling() {
    try {
      const v8Profiler = require('v8-profiler-next');
      v8Profiler.startProfiling('CPU Profile', true);
    } catch (error) {
      console.warn('V8 profiler not available:', error.message);
    }
  }
  
  captureCPUSample() {
    const timestamp = performance.now();
    const callStack = this.captureCallStack();
    
    return {
      timestamp,
      callStack,
      executionContext: this.getExecutionContext()
    };
  }
  
  captureCallStack() {
    const error = new Error();
    const stack = error.stack?.split('\n').slice(1) || [];
    
    return stack.map(frame => {
      const match = frame.match(/at\s+(.+?)\s+\((.+?):(\d+):(\d+)\)/) ||
                   frame.match(/at\s+(.+?):(\d+):(\d+)/);
      
      if (match) {
        return {
          function: match[1] || 'anonymous',
          file: match[2] || 'unknown',
          line: parseInt(match[3]) || 0,
          column: parseInt(match[4]) || 0
        };
      }
      
      return { function: frame.trim(), file: 'unknown', line: 0, column: 0 };
    });
  }
  
  getExecutionContext() {
    return {
      url: typeof window !== 'undefined' ? window.location.href : process.cwd(),
      userAgent: typeof navigator !== 'undefined' ? navigator.userAgent : process.version,
      timestamp: Date.now(),
      memory: this.getMemoryUsage()
    };
  }
  
  stopCPUSampling() {
    if (this.samplingTimer) {
      clearInterval(this.samplingTimer);
      this.samplingTimer = null;
    }
    
    if (typeof window === 'undefined') {
      try {
        const v8Profiler = require('v8-profiler-next');
        const profile = v8Profiler.stopProfiling('CPU Profile');
        this.currentSession.v8Profile = profile;
      } catch (error) {
        // Ignore if not available
      }
    }
  }
  
  // Memory Profiling
  startMemoryMonitoring() {
    const captureSnapshot = () => {
      const snapshot = this.captureMemorySnapshot();
      this.currentSession.memorySnapshots.push(snapshot);
    };
    
    // Initial snapshot
    captureSnapshot();
    
    // Periodic snapshots
    this.memoryTimer = setInterval(captureSnapshot, 1000);
  }
  
  captureMemorySnapshot() {
    const memory = this.getMemoryUsage();
    const timestamp = performance.now();
    
    return {
      timestamp,
      ...memory,
      objectCounts: this.getObjectCounts(),
      heapSpaces: this.getHeapSpaces()
    };
  }
  
  getMemoryUsage() {
    if (typeof window !== 'undefined' && 'memory' in performance) {
      return {
        usedJSHeapSize: performance.memory.usedJSHeapSize,
        totalJSHeapSize: performance.memory.totalJSHeapSize,
        jsHeapSizeLimit: performance.memory.jsHeapSizeLimit
      };
    }
    
    if (typeof process !== 'undefined') {
      const memUsage = process.memoryUsage();
      return {
        rss: memUsage.rss,
        heapTotal: memUsage.heapTotal,
        heapUsed: memUsage.heapUsed,
        external: memUsage.external,
        arrayBuffers: memUsage.arrayBuffers
      };
    }
    
    return {};
  }
  
  getObjectCounts() {
    // This would require native bindings or dev tools
    // Simplified implementation
    return {
      objects: 0,
      arrays: 0,
      functions: 0,
      strings: 0
    };
  }
  
  getHeapSpaces() {
    if (typeof process !== 'undefined') {
      try {
        const v8 = require('v8');
        return v8.getHeapSpaceStatistics();
      } catch (error) {
        return [];
      }
    }
    return [];
  }
  
  stopMemoryMonitoring() {
    if (this.memoryTimer) {
      clearInterval(this.memoryTimer);
      this.memoryTimer = null;
    }
  }
  
  // Performance Observers
  setupPerformanceObservers() {
    // Navigation timing
    this.observeEntryTypes(['navigation'], (entries) => {
      entries.forEach(entry => {
        this.recordNavigationMetrics(entry);
      });
    });
    
    // Resource timing
    this.observeEntryTypes(['resource'], (entries) => {
      entries.forEach(entry => {
        this.recordResourceMetrics(entry);
      });
    });
    
    // Paint timing
    this.observeEntryTypes(['paint'], (entries) => {
      entries.forEach(entry => {
        this.recordPaintMetrics(entry);
      });
    });
    
    // Largest Contentful Paint
    this.observeEntryTypes(['largest-contentful-paint'], (entries) => {
      entries.forEach(entry => {
        this.recordLCPMetrics(entry);
      });
    });
  }
  
  observeEntryTypes(entryTypes, callback) {
    try {
      const observer = new PerformanceObserver((list) => {
        callback(list.getEntries());
      });
      
      observer.observe({ entryTypes });
      this.observers.set(entryTypes.join(','), observer);
    } catch (error) {
      console.warn('Failed to observe performance entries:', entryTypes, error);
    }
  }
  
  setupLongTaskObserver() {
    if ('PerformanceObserver' in window) {
      try {
        const observer = new PerformanceObserver((list) => {
          list.getEntries().forEach(entry => {
            this.recordLongTask(entry);
          });
        });
        
        observer.observe({ entryTypes: ['longtask'] });
        this.observers.set('longtask', observer);
      } catch (error) {
        console.warn('Long task observer not supported:', error);
      }
    }
  }
  
  setupLayoutShiftObserver() {
    if ('PerformanceObserver' in window) {
      try {
        const observer = new PerformanceObserver((list) => {
          list.getEntries().forEach(entry => {
            this.recordLayoutShift(entry);
          });
        });
        
        observer.observe({ entryTypes: ['layout-shift'] });
        this.observers.set('layout-shift', observer);
      } catch (error) {
        console.warn('Layout shift observer not supported:', error);
      }
    }
  }
  
  // Analysis and Recommendations
  analyzeProfile(session) {
    const analysis = {
      summary: this.generateSummary(session),
      bottlenecks: this.identifyBottlenecks(session),
      recommendations: this.generateRecommendations(session),
      heatMap: this.generateHeatMap(session),
      flameGraph: this.generateFlameGraph(session),
      metrics: this.calculateMetrics(session)
    };
    
    return analysis;
  }
  
  generateSummary(session) {
    return {
      duration: session.duration,
      totalSamples: session.samples.length,
      memoryGrowth: this.calculateMemoryGrowth(session),
      avgCPUUsage: this.calculateAverageCPUUsage(session),
      mainThreadBlocking: this.calculateMainThreadBlocking(session),
      renderingPerformance: this.calculateRenderingPerformance(session)
    };
  }
  
  identifyBottlenecks(session) {
    const bottlenecks = [];
    
    // CPU bottlenecks
    const cpuHotspots = this.findCPUHotspots(session);
    bottlenecks.push(...cpuHotspots);
    
    // Memory bottlenecks
    const memoryLeaks = this.findMemoryLeaks(session);
    bottlenecks.push(...memoryLeaks);
    
    // Network bottlenecks
    const networkIssues = this.findNetworkBottlenecks(session);
    bottlenecks.push(...networkIssues);
    
    // Rendering bottlenecks
    const renderingIssues = this.findRenderingBottlenecks(session);
    bottlenecks.push(...renderingIssues);
    
    return bottlenecks.sort((a, b) => b.severity - a.severity);
  }
  
  findCPUHotspots(session) {
    const functionCounts = new Map();
    
    session.samples.forEach(sample => {
      sample.callStack.forEach(frame => {
        const key = `${frame.function}@${frame.file}:${frame.line}`;
        functionCounts.set(key, (functionCounts.get(key) || 0) + 1);
      });
    });
    
    const hotspots = [];
    const threshold = session.samples.length * 0.05; // 5% threshold
    
    for (const [key, count] of functionCounts) {
      if (count > threshold) {
        const [func, location] = key.split('@');
        hotspots.push({
          type: 'cpu-hotspot',
          function: func,
          location,
          sampleCount: count,
          percentage: (count / session.samples.length) * 100,
          severity: Math.min((count / threshold) * 3, 10)
        });
      }
    }
    
    return hotspots;
  }
  
  findMemoryLeaks(session) {
    const leaks = [];
    
    if (session.memorySnapshots.length < 2) {
      return leaks;
    }
    
    const firstSnapshot = session.memorySnapshots[0];
    const lastSnapshot = session.memorySnapshots[session.memorySnapshots.length - 1];
    
    const memoryGrowth = lastSnapshot.usedJSHeapSize - firstSnapshot.usedJSHeapSize;
    const growthRate = memoryGrowth / session.duration; // bytes per ms
    
    if (growthRate > 1000) { // 1KB per ms
      leaks.push({
        type: 'memory-leak',
        growthRate: growthRate,
        totalGrowth: memoryGrowth,
        severity: Math.min(growthRate / 1000, 10)
      });
    }
    
    return leaks;
  }
  
  findNetworkBottlenecks(session) {
    const bottlenecks = [];
    
    session.networkRequests.forEach(request => {
      if (request.duration > 2000) { // 2 seconds
        bottlenecks.push({
          type: 'slow-network',
          url: request.url,
          duration: request.duration,
          size: request.size,
          severity: Math.min(request.duration / 1000, 10)
        });
      }
    });
    
    return bottlenecks;
  }
  
  findRenderingBottlenecks(session) {
    const bottlenecks = [];
    
    session.renderingEvents.forEach(event => {
      if (event.type === 'layout' && event.duration > 16) { // >16ms layout
        bottlenecks.push({
          type: 'layout-thrash',
          duration: event.duration,
          elements: event.affectedElements,
          severity: Math.min(event.duration / 16, 10)
        });
      }
    });
    
    return bottlenecks;
  }
  
  generateRecommendations(session) {
    const recommendations = [];
    const bottlenecks = this.identifyBottlenecks(session);
    
    bottlenecks.forEach(bottleneck => {
      switch (bottleneck.type) {
        case 'cpu-hotspot':
          recommendations.push({
            category: 'Performance',
            severity: 'high',
            title: 'CPU Hotspot Detected',
            description: `Function ${bottleneck.function} is consuming ${bottleneck.percentage.toFixed(1)}% of CPU time`,
            actions: [
              'Consider optimizing the algorithm',
              'Use memoization for expensive calculations',
              'Move heavy computations to Web Workers',
              'Implement lazy loading or virtualization'
            ]
          });
          break;
          
        case 'memory-leak':
          recommendations.push({
            category: 'Memory',
            severity: 'critical',
            title: 'Memory Leak Detected',
            description: `Memory is growing at ${(bottleneck.growthRate / 1024).toFixed(2)} KB/ms`,
            actions: [
              'Check for unreleased event listeners',
              'Verify proper cleanup of intervals/timeouts',
              'Review closure usage and references',
              'Use WeakMap/WeakSet for caching'
            ]
          });
          break;
          
        case 'slow-network':
          recommendations.push({
            category: 'Network',
            severity: 'medium',
            title: 'Slow Network Request',
            description: `Request to ${bottleneck.url} took ${bottleneck.duration}ms`,
            actions: [
              'Implement request caching',
              'Use compression (gzip/brotli)',
              'Optimize payload size',
              'Consider using a CDN'
            ]
          });
          break;
          
        case 'layout-thrash':
          recommendations.push({
            category: 'Rendering',
            severity: 'high',
            title: 'Layout Thrashing Detected',
            description: `Layout operation took ${bottleneck.duration}ms`,
            actions: [
              'Batch DOM reads and writes',
              'Use CSS transforms instead of changing layout properties',
              'Implement virtual scrolling for long lists',
              'Minimize forced synchronous layouts'
            ]
          });
          break;
      }
    });
    
    return recommendations;
  }
  
  generateHeatMap(session) {
    const heatMap = {
      cpu: new Map(),
      memory: new Map(),
      network: new Map(),
      rendering: new Map()
    };
    
    // CPU heatmap
    session.samples.forEach(sample => {
      sample.callStack.forEach(frame => {
        const key = `${frame.file}:${frame.line}`;
        heatMap.cpu.set(key, (heatMap.cpu.get(key) || 0) + 1);
      });
    });
    
    // Memory heatmap
    session.memorySnapshots.forEach(snapshot => {
      const timestamp = Math.floor(snapshot.timestamp / 1000) * 1000;
      heatMap.memory.set(timestamp, snapshot.usedJSHeapSize);
    });
    
    return heatMap;
  }
  
  generateFlameGraph(session) {
    const flameGraph = {
      name: 'root',
      value: session.duration,
      children: []
    };
    
    const functionMap = new Map();
    
    session.samples.forEach(sample => {
      let currentNode = flameGraph;
      
      sample.callStack.forEach(frame => {
        const key = `${frame.function}@${frame.file}:${frame.line}`;
        
        let child = currentNode.children.find(c => c.name === key);
        if (!child) {
          child = {
            name: key,
            value: 0,
            children: []
          };
          currentNode.children.push(child);
        }
        
        child.value += 1;
        currentNode = child;
      });
    });
    
    return flameGraph;
  }
  
  calculateMetrics(session) {
    return {
      samplesPerSecond: session.samples.length / (session.duration / 1000),
      memoryEfficiency: this.calculateMemoryEfficiency(session),
      cpuEfficiency: this.calculateCPUEfficiency(session),
      networkEfficiency: this.calculateNetworkEfficiency(session),
      renderingEfficiency: this.calculateRenderingEfficiency(session)
    };
  }
  
  calculateMemoryEfficiency(session) {
    if (session.memorySnapshots.length < 2) return 100;
    
    const growth = this.calculateMemoryGrowth(session);
    const efficiency = Math.max(0, 100 - (growth / 1024 / 1024)); // Penalize growth
    
    return Math.min(100, efficiency);
  }
  
  calculateCPUEfficiency(session) {
    const hotspots = this.findCPUHotspots(session);
    const inefficiency = hotspots.reduce((sum, hotspot) => sum + hotspot.percentage, 0);
    
    return Math.max(0, 100 - inefficiency);
  }
  
  calculateNetworkEfficiency(session) {
    const slowRequests = session.networkRequests.filter(req => req.duration > 1000);
    const efficiency = ((session.networkRequests.length - slowRequests.length) / 
                       Math.max(1, session.networkRequests.length)) * 100;
    
    return efficiency;
  }
  
  calculateRenderingEfficiency(session) {
    const longTasks = session.renderingEvents.filter(event => event.duration > 16);
    const efficiency = ((session.renderingEvents.length - longTasks.length) / 
                       Math.max(1, session.renderingEvents.length)) * 100;
    
    return efficiency;
  }
  
  calculateMemoryGrowth(session) {
    if (session.memorySnapshots.length < 2) return 0;
    
    const first = session.memorySnapshots[0];
    const last = session.memorySnapshots[session.memorySnapshots.length - 1];
    
    return last.usedJSHeapSize - first.usedJSHeapSize;
  }
  
  calculateAverageCPUUsage(session) {
    // Simplified calculation based on sampling frequency
    return Math.min(100, (session.samples.length / session.duration) * 100);
  }
  
  calculateMainThreadBlocking(session) {
    const longTasks = session.samples.filter(sample => 
      sample.callStack.some(frame => frame.function.includes('Promise') || 
                           frame.function.includes('async'))
    );
    
    return (longTasks.length / session.samples.length) * 100;
  }
  
  calculateRenderingPerformance(session) {
    const fastFrames = session.renderingEvents.filter(event => event.duration <= 16);
    return (fastFrames.length / Math.max(1, session.renderingEvents.length)) * 100;
  }
  
  // Utility methods
  recordNavigationMetrics(entry) {
    if (this.currentSession) {
      this.currentSession.metrics.set('navigation', {
        dns: entry.domainLookupEnd - entry.domainLookupStart,
        tcp: entry.connectEnd - entry.connectStart,
        request: entry.responseStart - entry.requestStart,
        response: entry.responseEnd - entry.responseStart,
        processing: entry.domComplete - entry.domLoading,
        load: entry.loadEventEnd - entry.loadEventStart
      });
    }
  }
  
  recordResourceMetrics(entry) {
    if (this.currentSession) {
      this.currentSession.networkRequests.push({
        url: entry.name,
        duration: entry.duration,
        size: entry.transferSize || 0,
        type: entry.initiatorType,
        timestamp: entry.startTime
      });
    }
  }
  
  recordPaintMetrics(entry) {
    if (this.currentSession) {
      this.currentSession.renderingEvents.push({
        type: 'paint',
        name: entry.name,
        startTime: entry.startTime,
        duration: entry.duration || 0
      });
    }
  }
  
  recordLCPMetrics(entry) {
    if (this.currentSession) {
      this.currentSession.metrics.set('lcp', {
        renderTime: entry.renderTime,
        loadTime: entry.loadTime,
        size: entry.size,
        element: entry.element?.tagName || 'unknown'
      });
    }
  }
  
  recordLongTask(entry) {
    if (this.currentSession) {
      this.currentSession.renderingEvents.push({
        type: 'longtask',
        startTime: entry.startTime,
        duration: entry.duration,
        attribution: entry.attribution
      });
    }
  }
  
  recordLayoutShift(entry) {
    if (this.currentSession) {
      this.currentSession.renderingEvents.push({
        type: 'layout-shift',
        value: entry.value,
        hadRecentInput: entry.hadRecentInput,
        sources: entry.sources
      });
    }
  }
  
  // Export and Import
  exportProfile(sessionName) {
    const profile = this.profiles.get(sessionName);
    if (!profile) {
      throw new Error(`Profile ${sessionName} not found`);
    }
    
    return JSON.stringify(profile, null, 2);
  }
  
  importProfile(profileData) {
    const profile = JSON.parse(profileData);
    this.profiles.set(profile.session.name, profile);
    return profile;
  }
  
  // Cleanup
  destroy() {
    // Stop current profiling
    if (this.currentSession) {
      this.stopProfiling();
    }
    
    // Disconnect observers
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
    
    // Clear data
    this.profiles.clear();
    this.heatMaps.clear();
  }
}

// ============= FUTURE-PROOF OPTIMIZATION =============
// lib/optimization/future-optimizer.js

export class FutureProofOptimizer {
  constructor(options = {}) {
    this.options = {
      enableWebAssembly: true,
      enableServiceWorkers: true,
      enableWebGPU: false, // Experimental
      enableOffscreenCanvas: true,
      enableSharedArrayBuffer: false, // Security considerations
      enableStreams: true,
      enableModuleWorkers: true,
      adaptiveOptimization: true,
      ...options
    };
    
    this.capabilities = new Map();
    this.optimizations = new Map();
    this.adaptiveStrategies = new Map();
    this.performanceBaseline = null;
    
    this.initialize();
  }
  
  async initialize() {
    await this.detectCapabilities();
    this.setupAdaptiveOptimization();
    this.initializeOptimizations();
  }
  
  async detectCapabilities() {
    const capabilities = {
      webAssembly: await this.detectWebAssembly(),
      serviceWorker: await this.detectServiceWorker(),
      webGPU: await this.detectWebGPU(),
      offscreenCanvas: await this.detectOffscreenCanvas(),
      sharedArrayBuffer: await this.detectSharedArrayBuffer(),
      streams: await this.detectStreams(),
      moduleWorkers: await this.detectModuleWorkers(),
      bigInt: typeof BigInt !== 'undefined',
      weakRefs: typeof WeakRef !== 'undefined',
      finalizationRegistry: typeof FinalizationRegistry !== 'undefined'
    };
    
    this.capabilities.set('browser', capabilities);
    
    console.log('Detected capabilities:', capabilities);
    
    return capabilities;
  }
  
  async detectWebAssembly() {
    try {
      const wasmSupported = typeof WebAssembly === 'object' &&
                           typeof WebAssembly.instantiate === 'function';
      
      if (wasmSupported) {
        // Test instantiation
        const module = new WebAssembly.Module(
          new Uint8Array([0, 97, 115, 109, 1, 0, 0, 0])
        );
        return true;
      }
      
      return false;
    } catch (error) {
      return false;
    }
  }
  
  async detectServiceWorker() {
    return 'serviceWorker' in navigator;
  }
  
  async detectWebGPU() {
    return 'gpu' in navigator;
  }
  
  async detectOffscreenCanvas() {
    return typeof OffscreenCanvas !== 'undefined';
  }
  
  async detectSharedArrayBuffer() {
    return typeof SharedArrayBuffer !== 'undefined';
  }
  
  async detectStreams() {
    return typeof ReadableStream !== 'undefined' &&
           typeof WritableStream !== 'undefined';
  }
  
  async detectModuleWorkers() {
    try {
      const worker = new Worker('data:application/javascript,', { type: 'module' });
      worker.terminate();
      return true;
    } catch (error) {
      return false;
    }
  }
  
  // Optimization Strategies
  async optimizeWithWebAssembly(computationFunction, data) {
    const capabilities = this.capabilities.get('browser');
    
    if (!capabilities.webAssembly || !this.options.enableWebAssembly) {
      return computationFunction(data);
    }
    
    try {
      const wasmModule = await this.getOrCreateWasmModule(computationFunction);
      return await this.executeWasmFunction(wasmModule, data);
    } catch (error) {
      console.warn('WebAssembly optimization failed, falling back:', error);
      return computationFunction(data);
    }
  }
  
  async getOrCreateWasmModule(computationFunction) {
    const functionCode = computationFunction.toString();
    const cacheKey = this.hashFunction(functionCode);
    
    if (this.optimizations.has(cacheKey)) {
      return this.optimizations.get(cacheKey);
    }
    
    // In a real implementation, this would compile JS to WASM
    // For now, we'll create a simple WASM module
    const wasmModule = await this.createWasmModule();
    this.optimizations.set(cacheKey, wasmModule);
    
    return wasmModule;
  }
  
  async createWasmModule() {
    // Simple WASM module for demonstration
    const wasmCode = new Uint8Array([
      0x00, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00,
      0x01, 0x05, 0x01, 0x60, 0x00, 0x01, 0x7f,
      0x03, 0x02, 0x01, 0x00,
      0x07, 0x08, 0x01, 0x04, 0x6d, 0x61, 0x69, 0x6e, 0x00, 0x00,
      0x0a, 0x06, 0x01, 0x04, 0x00, 0x41, 0x2a, 0x0b
    ]);
    
    const module = await WebAssembly.instantiate(wasmCode);
    return module.instance;
  }
  
  async executeWasmFunction(wasmModule, data) {
    // Execute WASM function with data
    return wasmModule.exports.main();
  }
  
  // Service Worker Optimization
  async optimizeWithServiceWorker(cacheStrategy) {
    const capabilities = this.capabilities.get('browser');
    
    if (!capabilities.serviceWorker || !this.options.enableServiceWorkers) {
      return false;
    }
    
    try {
      const registration = await navigator.serviceWorker.register('/sw.js');
      
      // Configure caching strategy
      await this.configureCachingStrategy(registration, cacheStrategy);
      
      return true;
    } catch (error) {
      console.warn('Service Worker optimization failed:', error);
      return false;
    }
  }
  
  async configureCachingStrategy(registration, strategy) {
    // Send strategy to service worker
    if (registration.active) {
      registration.active.postMessage({
        type: 'CONFIGURE_CACHE',
        strategy
      });
    }
  }
  
  // Web GPU Optimization
  async optimizeWithWebGPU(computeShader, data) {
    const capabilities = this.capabilities.get('browser');
    
    if (!capabilities.webGPU || !this.options.enableWebGPU) {
      return null;
    }
    
    try {
      const adapter = await navigator.gpu.requestAdapter();
      const device = await adapter.requestDevice();
      
      const result = await this.executeGPUCompute(device, computeShader, data);
      return result;
    } catch (error) {
      console.warn('WebGPU optimization failed:', error);
      return null;
    }
  }
  
  async executeGPUCompute(device, shaderCode, data) {
    // Create compute pipeline
    const computePipeline = device.createComputePipeline({
      compute: {
        module: device.createShaderModule({ code: shaderCode }),
        entryPoint: 'main'
      }
    });
    
    // Create buffers
    const inputBuffer = device.createBuffer({
      size: data.byteLength,
      usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_DST
    });
    
    const outputBuffer = device.createBuffer({
      size: data.byteLength,
      usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_SRC
    });
    
    // Write data to input buffer
    device.queue.writeBuffer(inputBuffer, 0, data);
    
    // Create bind group
    const bindGroup = device.createBindGroup({
      layout: computePipeline.getBindGroupLayout(0),
      entries: [
        { binding: 0, resource: { buffer: inputBuffer } },
        { binding: 1, resource: { buffer: outputBuffer } }
      ]
    });
    
    // Dispatch compute
    const commandEncoder = device.createCommandEncoder();
    const passEncoder = commandEncoder.beginComputePass();
    
    passEncoder.setPipeline(computePipeline);
    passEncoder.setBindGroup(0, bindGroup);
    passEncoder.dispatch(Math.ceil(data.length / 64));
    passEncoder.endPass();
    
    device.queue.submit([commandEncoder.finish()]);
    
    // Read result
    const resultBuffer = device.createBuffer({
      size: data.byteLength,
      usage: GPUBufferUsage.COPY_DST | GPUBufferUsage.MAP_READ
    });
    
    const copyEncoder = device.createCommandEncoder();
    copyEncoder.copyBufferToBuffer(outputBuffer, 0, resultBuffer, 0, data.byteLength);
    device.queue.submit([copyEncoder.finish()]);
    
    await resultBuffer.mapAsync(GPUMapMode.READ);
    const result = new Float32Array(resultBuffer.getMappedRange());
    
    return Array.from(result);
  }
  
  // Offscreen Canvas Optimization
  async optimizeWithOffscreenCanvas(renderFunction, width, height) {
    const capabilities = this.capabilities.get('browser');
    
    if (!capabilities.offscreenCanvas || !this.options.enableOffscreenCanvas) {
      return null;
    }
    
    try {
      const offscreen = new OffscreenCanvas(width, height);
      const ctx = offscreen.getContext('2d');
      
      // Execute rendering in worker
      const worker = new Worker('/render-worker.js');
      
      const canvas = offscreen.transferControlToOffscreen
        ? offscreen.transferControlToOffscreen()
        : offscreen;
      
      worker.postMessage({
        type: 'RENDER',
        canvas,
        renderFunction: renderFunction.toString(),
        width,
        height
      }, [canvas]);
      
      return new Promise((resolve) => {
        worker.onmessage = (event) => {
          if (event.data.type === 'RENDER_COMPLETE') {
            resolve(event.data.imageData);
            worker.terminate();
          }
        };
      });
    } catch (error) {
      console.warn('OffscreenCanvas optimization failed:', error);
      return null;
    }
  }
  
  // Adaptive Optimization
  setupAdaptiveOptimization() {
    if (!this.options.adaptiveOptimization) return;
    
    // Monitor performance and adapt strategies
    setInterval(() => {
      this.evaluatePerformance();
      this.adaptOptimizationStrategies();
    }, 30000); // Every 30 seconds
  }
  
  evaluatePerformance() {
    const metrics = {
      memory: this.getMemoryUsage(),
      cpu: this.getCPUUsage(),
      network: this.getNetworkUsage(),
      rendering: this.getRenderingPerformance()
    };
    
    // Compare with baseline
    if (this.performanceBaseline) {
      const degradation = this.calculatePerformanceDegradation(metrics);
      
      if (degradation > 20) { // 20% degradation
        this.triggerOptimizationReview();
      }
    } else {
      this.performanceBaseline = metrics;
    }
  }
  
  adaptOptimizationStrategies() {
    const currentMetrics = this.getCurrentMetrics();
    
    // Adapt based on resource constraints
    if (currentMetrics.memory > 0.8) { // 80% memory usage
      this.enableMemoryOptimizations();
    }
    
    if (currentMetrics.cpu > 0.7) { // 70% CPU usage
      this.enableCPUOptimizations();
    }
    
    if (currentMetrics.network > 1000) { // High latency
      this.enableNetworkOptimizations();
    }
  }
  
  enableMemoryOptimizations() {
    // Enable aggressive garbage collection
    this.optimizations.set('memory', {
      aggressiveGC: true,
      objectPooling: true,
      weakReferences: true
    });
  }
  
  enableCPUOptimizations() {
    // Enable WebAssembly and Web Workers
    this.optimizations.set('cpu', {
      webAssembly: true,
      webWorkers: true,
      taskScheduling: true
    });
  }
  
  enableNetworkOptimizations() {
    // Enable aggressive caching and compression
    this.optimizations.set('network', {
      aggressiveCaching: true,
      compression: true,
      prefetching: true
    });
  }
  
  // Utility methods
  hashFunction(input) {
    let hash = 0;
    for (let i = 0; i < input.length; i++) {
      const char = input.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return hash.toString(36);
  }
  
  getMemoryUsage() {
    if (typeof performance !== 'undefined' && 'memory' in performance) {
      return performance.memory.usedJSHeapSize / performance.memory.jsHeapSizeLimit;
    }
    return 0;
  }
  
  getCPUUsage() {
    // Simplified CPU usage estimation
    return Math.random() * 0.5; // Placeholder
  }
  
  getNetworkUsage() {
    // Network latency estimation
    return Math.random() * 1000; // Placeholder
  }
  
  getRenderingPerformance() {
    // Rendering performance score
    return Math.random() * 100; // Placeholder
  }
  
  getCurrentMetrics() {
    return {
      memory: this.getMemoryUsage(),
      cpu: this.getCPUUsage(),
      network: this.getNetworkUsage(),
      rendering: this.getRenderingPerformance()
    };
  }
  
  calculatePerformanceDegradation(current) {
    if (!this.performanceBaseline) return 0;
    
    const memoryDegradation = ((current.memory - this.performanceBaseline.memory) / 
                              this.performanceBaseline.memory) * 100;
    
    const cpuDegradation = ((current.cpu - this.performanceBaseline.cpu) / 
                           this.performanceBaseline.cpu) * 100;
    
    return Math.max(memoryDegradation, cpuDegradation);
  }
  
  triggerOptimizationReview() {
    console.log('Performance degradation detected, reviewing optimization strategies...');
    this.adaptOptimizationStrategies();
  }
  
  // API methods
  getOptimizationStatus() {
    return {
      capabilities: Object.fromEntries(this.capabilities),
      optimizations: Object.fromEntries(this.optimizations),
      performanceBaseline: this.performanceBaseline,
      adaptiveOptimization: this.options.adaptiveOptimization
    };
  }
}
```

## 🎯 แบบฝึกหัด

### **🚀 แบบฝึกหัดที่ 1: Performance Profiling Suite**
สร้าง comprehensive profiling system:

```javascript
// TODO: สร้าง profiling suite

// 1. Real-time profiler
class RealTimeProfiler {
  // Live performance monitoring
  // Interactive flame graphs
  // Memory leak detection
}

// 2. Automated optimization
class AutoOptimizer {
  // ML-based optimization suggestions
  // Automatic code splitting
  // Dynamic resource optimization
}

// 3. Performance regression testing
class RegressionTester {
  // Automated performance testing
  // Baseline comparison
  // CI/CD integration
}
```

### **⚡ แบบฝึกหัดที่ 2: Future Technology Integration**
สร้าง cutting-edge optimization system:

```javascript
// TODO: สร้าง future tech integration

// 1. WebAssembly compiler
class WasmCompiler {
  // JS to WASM compilation
  // Performance hotspot optimization
  // Runtime recompilation
}

// 2. AI-powered optimization
class AIOptimizer {
  // Machine learning optimization
  // Predictive prefetching
  // Adaptive algorithms
}

// 3. Edge computing integration
class EdgeOptimizer {
  // Edge caching strategies
  // Distributed computation
  // Latency optimization
}
```

### **🔬 แบบฝึกหัดที่ 3: Advanced Analytics**
สร้าง sophisticated analytics platform:

```javascript
// TODO: สร้าง analytics platform

// 1. Performance analytics
class PerformanceAnalytics {
  // Advanced metrics collection
  // User experience analytics
  // Business impact analysis
}

// 2. Predictive modeling
class PredictiveModeler {
  // Performance prediction
  // Capacity planning
  // Anomaly detection
}

// 3. Optimization recommendations
class RecommendationEngine {
  // AI-powered suggestions
  // Cost-benefit analysis
  // Implementation guidance
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 29: Enterprise Integration](./29-enterprise-integration.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Web Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance)
- [WebAssembly](https://webassembly.org/)
- [WebGPU](https://gpuweb.github.io/gpuweb/)
- [Performance Optimization Guide](https://web.dev/performance/)

---

## 🎉 สรุปการเรียนรู้ทั้งหมด

ในบทสุดท้ายนี้เราได้เรียนรู้:

- **Advanced Profiling** - sophisticated performance monitoring และ analysis tools
- **Future Technologies** - WebAssembly, WebGPU, และ emerging web technologies
- **Adaptive Optimization** - intelligent systems ที่ปรับปรุงประสิทธิภาพแบบอัตโนมัติ
- **Cutting-edge Techniques** - latest optimization strategies และ best practices
- **Future-proof Development** - techniques สำหรับ building applications ที่พร้อมสำหรับอนาคต

## 🏁 สรุปการเรียนรู้ทั้งหมด 30 บท

เราได้เดินทางผ่าน **JavaScript ES6+ for Next.js** ทั้ง 30 บท ตั้งแต่พื้นฐานไปจนถึงระดับ expert:

### 🎯 **บทที่ 1-8: พื้นฐาน**
- Modern JavaScript features และ ES6+ syntax
- Functions, Objects, Arrays และ advanced techniques
- Classes, Modules และ Error Handling
- Async Programming และ Event Handling

### 🚀 **บทที่ 9-16: ระดับกลาง**
- Advanced Features และ Next.js fundamentals
- React Hooks และ API integration
- State Management และ UI Components
- Routing และ Styling strategies

### 🏗️ **บทที่ 17-24: ระดับสูง**
- Testing strategies และ Build Optimization
- Database integration และ Authentication
- Real-time features และ PWA development
- Performance optimization และ Security

### 🌟 **บทที่ 25-30: ระดับ Expert**
- Advanced Patterns และ Architecture
- Scalability และ Production Deployment
- Enterprise Integration และ Future Optimization

**คุณได้เรียนรู้ทุกสิ่งที่จำเป็นสำหรับการเป็น JavaScript Developer ระดับ Expert! 🎓**

ขอบคุณที่ร่วมเดินทางไปกับเราในการเรียนรู้ JavaScript ES6+ และ Next.js! 🙏

**Happy Coding! 🚀💻✨**
