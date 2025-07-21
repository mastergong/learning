# บทที่ 15: Performance Optimization

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ performance optimization techniques
- ทำความเข้าใจ memory management และ garbage collection
- เรียนรู้ code splitting และ lazy loading
- ประยุกต์ใช้ใน Next.js สำหรับ high-performance applications

## ⚡ JavaScript Performance Fundamentals

### **🏃‍♂️ Code Optimization Techniques**

```javascript
// ============= MEMORY MANAGEMENT =============
class MemoryOptimizer {
  constructor() {
    this.cache = new Map();
    this.weakCache = new WeakMap();
    this.objectPool = [];
    this.maxPoolSize = 1000;
  }
  
  // Object pooling to reduce garbage collection
  getPooledObject() {
    if (this.objectPool.length > 0) {
      return this.objectPool.pop();
    }
    return this.createNewObject();
  }
  
  returnToPool(obj) {
    if (this.objectPool.length < this.maxPoolSize) {
      // Reset object state
      this.resetObject(obj);
      this.objectPool.push(obj);
    }
  }
  
  createNewObject() {
    return {
      data: null,
      processed: false,
      timestamp: null,
      metadata: {}
    };
  }
  
  resetObject(obj) {
    obj.data = null;
    obj.processed = false;
    obj.timestamp = null;
    obj.metadata = {};
  }
  
  // Efficient string operations
  fastStringConcat(parts) {
    // Using array join is faster than repeated concatenation
    return parts.join('');
  }
  
  // Optimized array operations
  fastArrayFilter(arr, predicate) {
    const result = [];
    for (let i = 0, len = arr.length; i < len; i++) {
      if (predicate(arr[i], i, arr)) {
        result.push(arr[i]);
      }
    }
    return result;
  }
  
  fastArrayMap(arr, mapper) {
    const result = new Array(arr.length);
    for (let i = 0, len = arr.length; i < len; i++) {
      result[i] = mapper(arr[i], i, arr);
    }
    return result;
  }
  
  // Memory-efficient data processing
  processLargeDataset(dataset, batchSize = 1000) {
    const results = [];
    
    for (let i = 0; i < dataset.length; i += batchSize) {
      const batch = dataset.slice(i, i + batchSize);
      const batchResult = this.processBatch(batch);
      results.push(...batchResult);
      
      // Allow garbage collection between batches
      if (i % (batchSize * 10) === 0) {
        // Force garbage collection in Node.js (if available)
        if (global.gc) {
          global.gc();
        }
      }
    }
    
    return results;
  }
  
  processBatch(batch) {
    return batch.map(item => {
      const obj = this.getPooledObject();
      obj.data = item;
      obj.processed = true;
      obj.timestamp = Date.now();
      
      // Process the object
      const result = this.processItem(obj);
      
      // Return to pool
      this.returnToPool(obj);
      
      return result;
    });
  }
  
  processItem(obj) {
    // Simulate processing
    return {
      id: obj.data.id,
      result: obj.data.value * 2,
      processed: obj.processed,
      timestamp: obj.timestamp
    };
  }
}

// ============= PERFORMANCE MONITORING =============
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.observers = [];
    this.thresholds = {
      slowFunction: 100, // ms
      memoryUsage: 100 * 1024 * 1024, // 100MB
      heapSize: 200 * 1024 * 1024 // 200MB
    };
  }
  
  // Measure function execution time
  measure(name, fn) {
    return (...args) => {
      const start = performance.now();
      
      try {
        const result = fn(...args);
        
        if (result instanceof Promise) {
          return result.finally(() => {
            const duration = performance.now() - start;
            this.recordMetric(name, duration);
          });
        }
        
        const duration = performance.now() - start;
        this.recordMetric(name, duration);
        return result;
        
      } catch (error) {
        const duration = performance.now() - start;
        this.recordMetric(name, duration, error);
        throw error;
      }
    };
  }
  
  recordMetric(name, duration, error = null) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, {
        calls: 0,
        totalTime: 0,
        minTime: Infinity,
        maxTime: 0,
        errors: 0,
        history: []
      });
    }
    
    const metric = this.metrics.get(name);
    metric.calls++;
    metric.totalTime += duration;
    metric.minTime = Math.min(metric.minTime, duration);
    metric.maxTime = Math.max(metric.maxTime, duration);
    
    if (error) {
      metric.errors++;
    }
    
    metric.history.push({
      duration,
      timestamp: Date.now(),
      error: error ? error.message : null
    });
    
    // Keep only recent history
    if (metric.history.length > 1000) {
      metric.history = metric.history.slice(-500);
    }
    
    // Check thresholds
    if (duration > this.thresholds.slowFunction) {
      this.notifyObservers('slowFunction', {
        name,
        duration,
        threshold: this.thresholds.slowFunction
      });
    }
  }
  
  getStats(name) {
    const metric = this.metrics.get(name);
    if (!metric) return null;
    
    return {
      calls: metric.calls,
      averageTime: metric.totalTime / metric.calls,
      minTime: metric.minTime,
      maxTime: metric.maxTime,
      totalTime: metric.totalTime,
      errorRate: metric.errors / metric.calls,
      errors: metric.errors
    };
  }
  
  getAllStats() {
    const stats = {};
    for (const [name, metric] of this.metrics) {
      stats[name] = this.getStats(name);
    }
    return stats;
  }
  
  // Monitor memory usage
  monitorMemory() {
    if (typeof process !== 'undefined' && process.memoryUsage) {
      const usage = process.memoryUsage();
      
      if (usage.heapUsed > this.thresholds.memoryUsage) {
        this.notifyObservers('highMemoryUsage', {
          heapUsed: usage.heapUsed,
          heapTotal: usage.heapTotal,
          threshold: this.thresholds.memoryUsage
        });
      }
    }
  }
  
  addObserver(callback) {
    this.observers.push(callback);
  }
  
  notifyObservers(event, data) {
    this.observers.forEach(observer => {
      try {
        observer(event, data);
      } catch (error) {
        console.error('Observer error:', error);
      }
    });
  }
  
  startMonitoring(interval = 5000) {
    this.monitoringInterval = setInterval(() => {
      this.monitorMemory();
    }, interval);
  }
  
  stopMonitoring() {
    if (this.monitoringInterval) {
      clearInterval(this.monitoringInterval);
    }
  }
}

// ============= ALGORITHM OPTIMIZATION =============
class AlgorithmOptimizer {
  // Optimized search algorithms
  static binarySearch(arr, target, compareFn = (a, b) => a - b) {
    let left = 0;
    let right = arr.length - 1;
    
    while (left <= right) {
      const mid = Math.floor((left + right) / 2);
      const comparison = compareFn(arr[mid], target);
      
      if (comparison === 0) {
        return mid;
      } else if (comparison < 0) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    
    return -1;
  }
  
  // Efficient sorting for specific use cases
  static quickSort(arr, compareFn = (a, b) => a - b) {
    if (arr.length <= 1) return arr;
    
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = [];
    const right = [];
    const equal = [];
    
    for (const element of arr) {
      const comparison = compareFn(element, pivot);
      if (comparison < 0) {
        left.push(element);
      } else if (comparison > 0) {
        right.push(element);
      } else {
        equal.push(element);
      }
    }
    
    return [
      ...this.quickSort(left, compareFn),
      ...equal,
      ...this.quickSort(right, compareFn)
    ];
  }
  
  // Memoization decorator
  static memoize(fn, getKey = (...args) => JSON.stringify(args)) {
    const cache = new Map();
    
    return (...args) => {
      const key = getKey(...args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = fn(...args);
      cache.set(key, result);
      
      // Limit cache size
      if (cache.size > 1000) {
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey);
      }
      
      return result;
    };
  }
  
  // Debouncing for performance
  static debounce(fn, delay) {
    let timeoutId;
    
    return (...args) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => fn(...args), delay);
    };
  }
  
  // Throttling for performance
  static throttle(fn, limit) {
    let inThrottle;
    
    return (...args) => {
      if (!inThrottle) {
        fn(...args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  }
  
  // Batch processing
  static createBatcher(fn, batchSize = 10, delay = 100) {
    let batch = [];
    let timeoutId;
    
    return (item) => {
      batch.push(item);
      
      if (batch.length >= batchSize) {
        fn(batch);
        batch = [];
        clearTimeout(timeoutId);
      } else {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
          if (batch.length > 0) {
            fn(batch);
            batch = [];
          }
        }, delay);
      }
    };
  }
}

// ============= DATA STRUCTURE OPTIMIZATION =============
class OptimizedDataStructures {
  // Efficient Set implementation for large datasets
  static createBloomFilter(expectedElements = 1000, falsePositiveRate = 0.01) {
    const optimalBitArraySize = Math.ceil(
      -(expectedElements * Math.log(falsePositiveRate)) / (Math.log(2) ** 2)
    );
    const hashFunctions = Math.round((optimalBitArraySize / expectedElements) * Math.log(2));
    
    const bitArray = new Array(optimalBitArraySize).fill(false);
    
    const hash = (str, seed) => {
      let hash = seed;
      for (let i = 0; i < str.length; i++) {
        hash = (hash * 31 + str.charCodeAt(i)) % optimalBitArraySize;
      }
      return Math.abs(hash);
    };
    
    return {
      add(item) {
        const str = String(item);
        for (let i = 0; i < hashFunctions; i++) {
          const index = hash(str, i);
          bitArray[index] = true;
        }
      },
      
      test(item) {
        const str = String(item);
        for (let i = 0; i < hashFunctions; i++) {
          const index = hash(str, i);
          if (!bitArray[index]) {
            return false;
          }
        }
        return true; // Might be a false positive
      },
      
      clear() {
        bitArray.fill(false);
      }
    };
  }
  
  // Efficient Queue implementation
  static createQueue() {
    let head = 0;
    let tail = 0;
    const items = {};
    
    return {
      enqueue(item) {
        items[tail] = item;
        tail++;
      },
      
      dequeue() {
        if (head === tail) return undefined;
        
        const item = items[head];
        delete items[head];
        head++;
        
        // Reset pointers when queue is empty
        if (head === tail) {
          head = 0;
          tail = 0;
        }
        
        return item;
      },
      
      peek() {
        return head === tail ? undefined : items[head];
      },
      
      get size() {
        return tail - head;
      },
      
      get isEmpty() {
        return head === tail;
      }
    };
  }
  
  // Trie for efficient string operations
  static createTrie() {
    const root = { children: {}, isEndOfWord: false };
    
    return {
      insert(word) {
        let node = root;
        for (const char of word) {
          if (!node.children[char]) {
            node.children[char] = { children: {}, isEndOfWord: false };
          }
          node = node.children[char];
        }
        node.isEndOfWord = true;
      },
      
      search(word) {
        let node = root;
        for (const char of word) {
          if (!node.children[char]) {
            return false;
          }
          node = node.children[char];
        }
        return node.isEndOfWord;
      },
      
      startsWith(prefix) {
        let node = root;
        for (const char of prefix) {
          if (!node.children[char]) {
            return false;
          }
          node = node.children[char];
        }
        return true;
      },
      
      getAllWords(prefix = '') {
        const words = [];
        let node = root;
        
        // Navigate to prefix node
        for (const char of prefix) {
          if (!node.children[char]) {
            return words;
          }
          node = node.children[char];
        }
        
        // DFS to find all words
        const dfs = (currentNode, currentWord) => {
          if (currentNode.isEndOfWord) {
            words.push(currentWord);
          }
          
          for (const [char, childNode] of Object.entries(currentNode.children)) {
            dfs(childNode, currentWord + char);
          }
        };
        
        dfs(node, prefix);
        return words;
      }
    };
  }
}

// ============= ASYNC OPTIMIZATION =============
class AsyncOptimizer {
  // Parallel execution with concurrency limit
  static async parallelMap(items, mapper, concurrency = 5) {
    const results = new Array(items.length);
    const executing = [];
    
    for (let i = 0; i < items.length; i++) {
      const promise = mapper(items[i], i).then(result => {
        results[i] = result;
      });
      
      executing.push(promise);
      
      if (executing.length >= concurrency) {
        await Promise.race(executing);
        executing.splice(executing.findIndex(p => p === promise), 1);
      }
    }
    
    await Promise.all(executing);
    return results;
  }
  
  // Optimized async iteration
  static async *asyncGenerator(items, processor, batchSize = 10) {
    for (let i = 0; i < items.length; i += batchSize) {
      const batch = items.slice(i, i + batchSize);
      const results = await Promise.all(
        batch.map(item => processor(item))
      );
      
      for (const result of results) {
        yield result;
      }
    }
  }
  
  // Request deduplication
  static createRequestDeduplicator() {
    const pendingRequests = new Map();
    
    return (key, requestFn) => {
      if (pendingRequests.has(key)) {
        return pendingRequests.get(key);
      }
      
      const promise = requestFn()
        .finally(() => {
          pendingRequests.delete(key);
        });
      
      pendingRequests.set(key, promise);
      return promise;
    };
  }
  
  // Timeout wrapper
  static withTimeout(promise, timeout = 5000) {
    return Promise.race([
      promise,
      new Promise((_, reject) =>
        setTimeout(() => reject(new Error('Operation timed out')), timeout)
      )
    ]);
  }
}

// ============= USAGE EXAMPLES =============
// Performance monitoring setup
const monitor = new PerformanceMonitor();

monitor.addObserver((event, data) => {
  console.warn(`Performance issue detected: ${event}`, data);
});

monitor.startMonitoring();

// Optimize functions
const optimizedCalculation = monitor.measure(
  'calculation',
  AlgorithmOptimizer.memoize((n) => {
    // Expensive calculation
    let result = 0;
    for (let i = 0; i < n; i++) {
      result += Math.sqrt(i);
    }
    return result;
  })
);

// Debounced search
const debouncedSearch = AlgorithmOptimizer.debounce((query) => {
  console.log('Searching for:', query);
  // Perform search
}, 300);

// Batch processing
const batchProcessor = AlgorithmOptimizer.createBatcher((batch) => {
  console.log('Processing batch of', batch.length, 'items');
  // Process batch
}, 5, 1000);

// Memory optimizer
const memoryOptimizer = new MemoryOptimizer();

// Example usage
console.log('Optimized calculation result:', optimizedCalculation(1000000));
console.log('Performance stats:', monitor.getStats('calculation'));

// Async optimization example
(async () => {
  const items = Array.from({ length: 100 }, (_, i) => i);
  
  const results = await AsyncOptimizer.parallelMap(
    items,
    async (item) => {
      // Simulate async work
      await new Promise(resolve => setTimeout(resolve, Math.random() * 100));
      return item * 2;
    },
    10
  );
  
  console.log('Parallel processing completed:', results.length, 'results');
})();

// Data structure examples
const bloomFilter = OptimizedDataStructures.createBloomFilter(1000, 0.01);
bloomFilter.add('test');
console.log('Bloom filter test:', bloomFilter.test('test')); // true (might be false positive)

const trie = OptimizedDataStructures.createTrie();
trie.insert('hello');
trie.insert('world');
console.log('Trie search:', trie.search('hello')); // true
console.log('Trie words with "he":', trie.getAllWords('he')); // ['hello']
```

## 🚀 Next.js Performance Optimization

### **⚡ Production Optimization**

```jsx
// ============= CODE SPLITTING AND LAZY LOADING =============
// components/LazyComponent.jsx
import dynamic from 'next/dynamic';
import { Suspense, memo, useMemo, useCallback } from 'react';

// Dynamic imports with custom loading
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div className="loading-spinner">Loading...</div>,
  ssr: false // Disable SSR for client-only components
});

const ChartComponent = dynamic(() => import('react-chartjs-2'), {
  loading: () => <div>Loading chart...</div>
});

// Conditional loading
const AdminPanel = dynamic(() => import('./AdminPanel'), {
  loading: () => <div>Loading admin panel...</div>
});

// Component with optimized rendering
const OptimizedComponent = memo(({ data, filters, onUpdate }) => {
  // Memoize expensive calculations
  const processedData = useMemo(() => {
    return data
      .filter(item => filters.includes(item.category))
      .map(item => ({
        ...item,
        computed: item.value * 1.2,
        formatted: new Intl.NumberFormat().format(item.value)
      }));
  }, [data, filters]);
  
  // Memoize callbacks to prevent unnecessary re-renders
  const handleItemClick = useCallback((id) => {
    onUpdate(prev => ({
      ...prev,
      selectedId: id
    }));
  }, [onUpdate]);
  
  const handleBatchUpdate = useCallback((items) => {
    // Batch updates for better performance
    onUpdate(prev => ({
      ...prev,
      items: {
        ...prev.items,
        ...items.reduce((acc, item) => {
          acc[item.id] = item;
          return acc;
        }, {})
      }
    }));
  }, [onUpdate]);
  
  return (
    <div className="optimized-component">
      {processedData.map(item => (
        <div
          key={item.id}
          onClick={() => handleItemClick(item.id)}
          className="item"
        >
          <span>{item.name}</span>
          <span>{item.formatted}</span>
        </div>
      ))}
    </div>
  );
}, (prevProps, nextProps) => {
  // Custom comparison for memo
  return (
    prevProps.data === nextProps.data &&
    JSON.stringify(prevProps.filters) === JSON.stringify(nextProps.filters)
  );
});

export default OptimizedComponent;

// ============= IMAGE OPTIMIZATION =============
// components/OptimizedImage.jsx
import Image from 'next/image';
import { useState, useCallback } from 'react';

const OptimizedImage = ({ 
  src, 
  alt, 
  width, 
  height, 
  priority = false,
  quality = 75,
  placeholder = 'blur',
  ...props 
}) => {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(false);
  
  const handleLoad = useCallback(() => {
    setIsLoading(false);
  }, []);
  
  const handleError = useCallback(() => {
    setError(true);
    setIsLoading(false);
  }, []);
  
  if (error) {
    return (
      <div 
        className="image-error"
        style={{ width, height }}
      >
        <span>Failed to load image</span>
      </div>
    );
  }
  
  return (
    <div className={`image-container ${isLoading ? 'loading' : ''}`}>
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        quality={quality}
        placeholder={placeholder}
        onLoad={handleLoad}
        onError={handleError}
        {...props}
      />
      
      {isLoading && (
        <div className="image-skeleton">
          <div className="skeleton-animation"></div>
        </div>
      )}
    </div>
  );
};

export default OptimizedImage;

// ============= VIRTUAL SCROLLING =============
// components/VirtualList.jsx
import { useState, useEffect, useMemo, useRef } from 'react';

const VirtualList = ({ 
  items, 
  itemHeight = 50, 
  containerHeight = 400,
  renderItem,
  overscan = 5 
}) => {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);
  
  const visibleItems = useMemo(() => {
    const startIndex = Math.max(
      0, 
      Math.floor(scrollTop / itemHeight) - overscan
    );
    const endIndex = Math.min(
      items.length - 1,
      Math.ceil((scrollTop + containerHeight) / itemHeight) + overscan
    );
    
    return items.slice(startIndex, endIndex + 1).map((item, index) => ({
      item,
      index: startIndex + index,
      top: (startIndex + index) * itemHeight
    }));
  }, [items, scrollTop, itemHeight, containerHeight, overscan]);
  
  const totalHeight = items.length * itemHeight;
  
  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };
  
  return (
    <div
      ref={containerRef}
      className="virtual-list"
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        {visibleItems.map(({ item, index, top }) => (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: top,
              left: 0,
              right: 0,
              height: itemHeight
            }}
          >
            {renderItem(item, index)}
          </div>
        ))}
      </div>
    </div>
  );
};

export default VirtualList;

// ============= PERFORMANCE CONTEXT =============
// context/PerformanceContext.jsx
import { createContext, useContext, useReducer, useEffect } from 'react';

const PerformanceContext = createContext();

const performanceReducer = (state, action) => {
  switch (action.type) {
    case 'START_OPERATION':
      return {
        ...state,
        operations: {
          ...state.operations,
          [action.payload.name]: {
            startTime: performance.now(),
            status: 'running'
          }
        }
      };
      
    case 'END_OPERATION':
      const operation = state.operations[action.payload.name];
      if (!operation) return state;
      
      const duration = performance.now() - operation.startTime;
      
      return {
        ...state,
        operations: {
          ...state.operations,
          [action.payload.name]: {
            ...operation,
            duration,
            status: 'completed'
          }
        },
        metrics: {
          ...state.metrics,
          [action.payload.name]: {
            count: (state.metrics[action.payload.name]?.count || 0) + 1,
            totalTime: (state.metrics[action.payload.name]?.totalTime || 0) + duration,
            lastDuration: duration
          }
        }
      };
      
    case 'RECORD_METRIC':
      return {
        ...state,
        customMetrics: {
          ...state.customMetrics,
          [action.payload.name]: action.payload.value
        }
      };
      
    default:
      return state;
  }
};

export const PerformanceProvider = ({ children }) => {
  const [state, dispatch] = useReducer(performanceReducer, {
    operations: {},
    metrics: {},
    customMetrics: {}
  });
  
  const startOperation = (name) => {
    dispatch({ type: 'START_OPERATION', payload: { name } });
  };
  
  const endOperation = (name) => {
    dispatch({ type: 'END_OPERATION', payload: { name } });
  };
  
  const recordMetric = (name, value) => {
    dispatch({ type: 'RECORD_METRIC', payload: { name, value } });
  };
  
  const measureOperation = (name, fn) => {
    return async (...args) => {
      startOperation(name);
      try {
        const result = await fn(...args);
        endOperation(name);
        return result;
      } catch (error) {
        endOperation(name);
        throw error;
      }
    };
  };
  
  // Monitor page performance
  useEffect(() => {
    if (typeof window !== 'undefined') {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          recordMetric(entry.name, entry.duration);
        }
      });
      
      observer.observe({ entryTypes: ['measure', 'navigation'] });
      
      return () => observer.disconnect();
    }
  }, []);
  
  const value = {
    ...state,
    startOperation,
    endOperation,
    recordMetric,
    measureOperation
  };
  
  return (
    <PerformanceContext.Provider value={value}>
      {children}
    </PerformanceContext.Provider>
  );
};

export const usePerformance = () => {
  const context = useContext(PerformanceContext);
  if (!context) {
    throw new Error('usePerformance must be used within PerformanceProvider');
  }
  return context;
};

// ============= PERFORMANCE HOOKS =============
// hooks/usePerformanceOptimization.js
import { useRef, useCallback, useEffect, useState } from 'react';

export const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
};

export const useThrottle = (value, limit) => {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRan = useRef(Date.now());
  
  useEffect(() => {
    const handler = setTimeout(() => {
      if (Date.now() - lastRan.current >= limit) {
        setThrottledValue(value);
        lastRan.current = Date.now();
      }
    }, limit - (Date.now() - lastRan.current));
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, limit]);
  
  return throttledValue;
};

export const useIntersectionObserver = (
  ref,
  options = { threshold: 0.1 }
) => {
  const [isIntersecting, setIsIntersecting] = useState(false);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => {
      observer.disconnect();
    };
  }, [ref, options]);
  
  return isIntersecting;
};

export const useMemoCompare = (next, compare) => {
  const previousRef = useRef();
  const previous = previousRef.current;
  
  const isEqual = compare(previous, next);
  
  useEffect(() => {
    if (!isEqual) {
      previousRef.current = next;
    }
  });
  
  return isEqual ? previous : next;
};

export const useAsyncState = (asyncFn, deps = []) => {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });
  
  useEffect(() => {
    let cancelled = false;
    
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    asyncFn()
      .then(data => {
        if (!cancelled) {
          setState({ data, loading: false, error: null });
        }
      })
      .catch(error => {
        if (!cancelled) {
          setState({ data: null, loading: false, error });
        }
      });
    
    return () => {
      cancelled = true;
    };
  }, deps);
  
  return state;
};

// ============= BUNDLER OPTIMIZATION =============
// next.config.js optimization example
const nextConfig = {
  // Enable experimental features
  experimental: {
    optimizeCss: true,
    nextScriptWorkers: true,
    workerThreads: true
  },
  
  // Image optimization
  images: {
    domains: ['example.com'],
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384]
  },
  
  // Webpack optimization
  webpack: (config, { dev, isServer }) => {
    // Production optimizations
    if (!dev) {
      config.optimization = {
        ...config.optimization,
        splitChunks: {
          chunks: 'all',
          cacheGroups: {
            vendor: {
              test: /[\\/]node_modules[\\/]/,
              name: 'vendors',
              chunks: 'all',
            },
            common: {
              name: 'common',
              minChunks: 2,
              chunks: 'all',
              enforce: true
            }
          }
        }
      };
    }
    
    // Bundle analyzer
    if (process.env.ANALYZE === 'true') {
      const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
      config.plugins.push(
        new BundleAnalyzerPlugin({
          analyzerMode: 'static',
          openAnalyzer: false,
        })
      );
    }
    
    return config;
  },
  
  // Headers for caching
  async headers() {
    return [
      {
        source: '/_next/static/(.*)',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable'
          }
        ]
      },
      {
        source: '/api/(.*)',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=60, s-maxage=60'
          }
        ]
      }
    ];
  }
};

module.exports = nextConfig;

// ============= PERFORMANCE DASHBOARD =============
// components/PerformanceDashboard.jsx
import { usePerformance } from '../context/PerformanceContext';
import { useEffect, useState } from 'react';

const PerformanceDashboard = () => {
  const { metrics, customMetrics, operations } = usePerformance();
  const [vitals, setVitals] = useState({});
  
  useEffect(() => {
    // Collect Web Vitals
    const collectVitals = () => {
      if ('performance' in window) {
        const navigation = performance.getEntriesByType('navigation')[0];
        const paint = performance.getEntriesByType('paint');
        
        setVitals({
          domContentLoaded: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
          firstPaint: paint.find(p => p.name === 'first-paint')?.startTime || 0,
          firstContentfulPaint: paint.find(p => p.name === 'first-contentful-paint')?.startTime || 0,
          totalLoadTime: navigation.loadEventEnd - navigation.navigationStart
        });
      }
    };
    
    collectVitals();
  }, []);
  
  return (
    <div className="performance-dashboard">
      <h2>Performance Metrics</h2>
      
      <section>
        <h3>Web Vitals</h3>
        <div className="metrics-grid">
          <div className="metric">
            <span>First Paint</span>
            <span>{vitals.firstPaint?.toFixed(2)}ms</span>
          </div>
          <div className="metric">
            <span>First Contentful Paint</span>
            <span>{vitals.firstContentfulPaint?.toFixed(2)}ms</span>
          </div>
          <div className="metric">
            <span>DOM Content Loaded</span>
            <span>{vitals.domContentLoaded?.toFixed(2)}ms</span>
          </div>
          <div className="metric">
            <span>Total Load Time</span>
            <span>{vitals.totalLoadTime?.toFixed(2)}ms</span>
          </div>
        </div>
      </section>
      
      <section>
        <h3>Operation Metrics</h3>
        <div className="operations-list">
          {Object.entries(metrics).map(([name, metric]) => (
            <div key={name} className="operation-metric">
              <span>{name}</span>
              <div className="metric-details">
                <span>Count: {metric.count}</span>
                <span>Avg: {(metric.totalTime / metric.count).toFixed(2)}ms</span>
                <span>Last: {metric.lastDuration?.toFixed(2)}ms</span>
              </div>
            </div>
          ))}
        </div>
      </section>
      
      <section>
        <h3>Custom Metrics</h3>
        <div className="custom-metrics">
          {Object.entries(customMetrics).map(([name, value]) => (
            <div key={name} className="custom-metric">
              <span>{name}</span>
              <span>{typeof value === 'number' ? value.toFixed(2) : value}</span>
            </div>
          ))}
        </div>
      </section>
    </div>
  );
};

export default PerformanceDashboard;
```

## 🎯 แบบฝึกหัด

### **⚡ แบบฝึกหัดที่ 1: Algorithm Optimization**
สร้าง optimized algorithms:

```javascript
// TODO: สร้าง optimized algorithms

// 1. Advanced caching system
class AdvancedCache {
  // Implement LRU, LFU, and time-based eviction
  // Memory usage monitoring
  // Cache hit rate optimization
}

// 2. Parallel processing system
class ParallelProcessor {
  // Worker thread management
  // Load balancing algorithms
  // Result aggregation
}

// 3. Data structure optimizer
class DataStructureOptimizer {
  // Analyze data access patterns
  // Recommend optimal data structures
  // Automatic conversion between structures
}
```

### **🚀 แบบฝึกหัดที่ 2: Next.js Optimization**
สร้าง production optimizations:

```jsx
// TODO: สร้าง Next.js optimizations

// 1. Advanced code splitting
const SmartCodeSplitter = () => {
  // Route-based splitting
  // Component-based splitting
  // Feature-based splitting
};

// 2. Performance monitoring
const PerformanceMonitor = () => {
  // Real-time metrics collection
  // Performance regression detection
  // Automatic optimization suggestions
};

// 3. Resource optimization
const ResourceOptimizer = () => {
  // Image optimization pipeline
  // Font loading optimization
  // Third-party script management
};
```

### **📊 แบบฝึกหัดที่ 3: Memory Management**
สร้าง memory management system:

```javascript
// TODO: สร้าง memory management

// 1. Memory leak detector
class MemoryLeakDetector {
  // Automatic leak detection
  // Memory usage analysis
  // Cleanup recommendations
}

// 2. Garbage collection optimizer
class GCOptimizer {
  // GC timing analysis
  // Memory allocation patterns
  // Optimization strategies
}

// 3. Object pool manager
class ObjectPoolManager {
  // Dynamic pool sizing
  // Object lifecycle management
  // Performance monitoring
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 14: Error Handling & Debugging](./14-error-handling.md)

**บทถัดไป**: [บทที่ 16: Testing Strategies](./16-testing.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Web Vitals](https://web.dev/vitals/)
- [Next.js Performance](https://nextjs.org/docs/advanced-features/performance)
- [JavaScript Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [React Performance](https://reactjs.org/docs/optimizing-performance.html)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Performance Optimization** - memory management, algorithm optimization, และ async patterns
- **Monitoring Tools** - performance metrics, profiling, และ memory leak detection
- **Next.js Optimization** - code splitting, lazy loading, และ bundle optimization
- **Production Techniques** - caching strategies, resource optimization, และ performance monitoring
- **Advanced Patterns** - virtual scrolling, debouncing/throttling, และ parallel processing

Performance optimization เป็นสิ่งสำคัญสำหรับ user experience และ scalability! 🚀
