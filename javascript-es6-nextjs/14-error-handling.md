# บทที่ 14: Error Handling & Debugging

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ advanced error handling strategies
- ทำความเข้าใจ debugging techniques และ tools
- เรียนรู้ error monitoring และ logging systems
- ประยุกต์ใช้ใน Next.js สำหรับ production-ready error handling

## 🚨 Advanced Error Handling

### **⚠️ Error Types และ Custom Errors**

```javascript
// ============= CUSTOM ERROR CLASSES =============
class AppError extends Error {
  constructor(message, statusCode = 500, isOperational = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    this.timestamp = new Date().toISOString();
    
    // Capture stack trace
    Error.captureStackTrace(this, this.constructor);
  }
  
  toJSON() {
    return {
      name: this.name,
      message: this.message,
      statusCode: this.statusCode,
      isOperational: this.isOperational,
      timestamp: this.timestamp,
      stack: this.stack
    };
  }
}

class ValidationError extends AppError {
  constructor(field, value, message) {
    super(`Validation failed for field '${field}': ${message}`, 400);
    this.field = field;
    this.value = value;
    this.type = 'VALIDATION_ERROR';
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with id '${id}' not found`, 404);
    this.resource = resource;
    this.resourceId = id;
    this.type = 'NOT_FOUND_ERROR';
  }
}

class AuthenticationError extends AppError {
  constructor(message = 'Authentication failed') {
    super(message, 401);
    this.type = 'AUTH_ERROR';
  }
}

class AuthorizationError extends AppError {
  constructor(action, resource) {
    super(`Not authorized to ${action} ${resource}`, 403);
    this.action = action;
    this.resource = resource;
    this.type = 'AUTHORIZATION_ERROR';
  }
}

class NetworkError extends AppError {
  constructor(url, method, statusCode) {
    super(`Network request failed: ${method} ${url}`, statusCode || 500);
    this.url = url;
    this.method = method;
    this.type = 'NETWORK_ERROR';
  }
}

class DatabaseError extends AppError {
  constructor(operation, table, originalError) {
    super(`Database ${operation} failed on ${table}`, 500);
    this.operation = operation;
    this.table = table;
    this.originalError = originalError;
    this.type = 'DATABASE_ERROR';
  }
}

// ============= ERROR CONTEXT WRAPPER =============
class ErrorContext {
  constructor() {
    this.context = new Map();
  }
  
  set(key, value) {
    this.context.set(key, value);
    return this;
  }
  
  get(key) {
    return this.context.get(key);
  }
  
  addUser(user) {
    return this.set('user', {
      id: user.id,
      email: user.email,
      role: user.role
    });
  }
  
  addRequest(req) {
    return this.set('request', {
      method: req.method,
      url: req.url,
      userAgent: req.headers['user-agent'],
      ip: req.ip || req.connection.remoteAddress
    });
  }
  
  addOperation(operation, data = {}) {
    return this.set('operation', {
      name: operation,
      ...data,
      timestamp: new Date().toISOString()
    });
  }
  
  toObject() {
    return Object.fromEntries(this.context);
  }
  
  attachToError(error) {
    error.context = this.toObject();
    return error;
  }
}

// ============= ERROR FACTORY =============
class ErrorFactory {
  static createFromHttpStatus(statusCode, message, context = {}) {
    const errors = {
      400: () => new ValidationError('request', null, message),
      401: () => new AuthenticationError(message),
      403: () => new AuthorizationError(context.action, context.resource),
      404: () => new NotFoundError(context.resource, context.id),
      500: () => new AppError(message, statusCode)
    };
    
    const ErrorClass = errors[statusCode] || errors[500];
    const error = ErrorClass();
    
    if (context && Object.keys(context).length > 0) {
      error.context = context;
    }
    
    return error;
  }
  
  static createValidationError(field, value, constraints) {
    const messages = [];
    
    if (constraints.required && !value) {
      messages.push(`${field} is required`);
    }
    
    if (constraints.minLength && value && value.length < constraints.minLength) {
      messages.push(`${field} must be at least ${constraints.minLength} characters`);
    }
    
    if (constraints.maxLength && value && value.length > constraints.maxLength) {
      messages.push(`${field} must be no more than ${constraints.maxLength} characters`);
    }
    
    if (constraints.pattern && value && !constraints.pattern.test(value)) {
      messages.push(`${field} format is invalid`);
    }
    
    return new ValidationError(field, value, messages.join(', '));
  }
  
  static createNetworkError(response, request) {
    const error = new NetworkError(request.url, request.method, response.status);
    
    error.context = {
      requestHeaders: request.headers,
      responseHeaders: response.headers,
      responseBody: response.data
    };
    
    return error;
  }
}

// ============= ERROR HANDLING UTILITIES =============
class ErrorHandler {
  constructor(options = {}) {
    this.logErrors = options.logErrors !== false;
    this.notifyErrors = options.notifyErrors || false;
    this.transformErrors = options.transformErrors || false;
    this.errorReporter = options.errorReporter || console.error;
    this.errorNotifier = options.errorNotifier || null;
    this.errorTransformer = options.errorTransformer || (error => error);
  }
  
  // Handle single error
  handle(error, context = null) {
    // Add context if provided
    if (context) {
      error.context = { ...error.context, ...context };
    }
    
    // Transform error if needed
    if (this.transformErrors) {
      error = this.errorTransformer(error);
    }
    
    // Log error
    if (this.logErrors) {
      this.log(error);
    }
    
    // Notify about error
    if (this.notifyErrors && this.errorNotifier) {
      this.notify(error);
    }
    
    return this.sanitizeError(error);
  }
  
  // Handle async operations with automatic error handling
  async handleAsync(operation, context = null) {
    try {
      return await operation();
    } catch (error) {
      throw this.handle(error, context);
    }
  }
  
  // Wrap function with error handling
  wrap(fn, context = null) {
    return async (...args) => {
      try {
        return await fn(...args);
      } catch (error) {
        throw this.handle(error, context);
      }
    };
  }
  
  // Log error with structured data
  log(error) {
    const logData = {
      timestamp: new Date().toISOString(),
      level: 'ERROR',
      message: error.message,
      type: error.type || error.name,
      statusCode: error.statusCode,
      stack: error.stack,
      context: error.context
    };
    
    this.errorReporter(JSON.stringify(logData, null, 2));
  }
  
  // Notify external services about error
  async notify(error) {
    if (this.errorNotifier) {
      try {
        await this.errorNotifier(error);
      } catch (notificationError) {
        console.error('Failed to notify error:', notificationError);
      }
    }
  }
  
  // Sanitize error for client response
  sanitizeError(error) {
    // Don't expose internal errors in production
    if (process.env.NODE_ENV === 'production' && !error.isOperational) {
      return new AppError('Something went wrong', 500);
    }
    
    // Remove sensitive information
    const sanitized = { ...error };
    delete sanitized.stack;
    delete sanitized.context;
    
    return sanitized;
  }
  
  // Create error middleware for Express
  middleware() {
    return (error, req, res, next) => {
      const handledError = this.handle(error, {
        method: req.method,
        url: req.url,
        ip: req.ip,
        userAgent: req.get('User-Agent')
      });
      
      res.status(handledError.statusCode || 500).json({
        error: {
          message: handledError.message,
          type: handledError.type,
          timestamp: handledError.timestamp
        }
      });
    };
  }
}

// ============= RETRY MECHANISM =============
class RetryHandler {
  constructor(options = {}) {
    this.maxRetries = options.maxRetries || 3;
    this.baseDelay = options.baseDelay || 1000;
    this.maxDelay = options.maxDelay || 10000;
    this.exponentialBase = options.exponentialBase || 2;
    this.jitter = options.jitter || false;
    this.retryCondition = options.retryCondition || this.defaultRetryCondition;
  }
  
  defaultRetryCondition(error, attempt) {
    // Retry on network errors, timeouts, and 5xx status codes
    if (error instanceof NetworkError) {
      return error.statusCode >= 500 || error.statusCode === 408;
    }
    
    // Retry on temporary errors
    return error.isRetryable === true;
  }
  
  async execute(operation, context = {}) {
    let lastError;
    
    for (let attempt = 0; attempt < this.maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;
        
        // Check if we should retry
        if (!this.retryCondition(error, attempt)) {
          throw error;
        }
        
        // Don't delay on last attempt
        if (attempt < this.maxRetries - 1) {
          await this.delay(attempt);
        }
        
        // Add retry context
        error.retryAttempt = attempt + 1;
        error.maxRetries = this.maxRetries;
        
        console.warn(`Retry attempt ${attempt + 1}/${this.maxRetries} for operation:`, {
          error: error.message,
          context
        });
      }
    }
    
    // All retries exhausted
    lastError.retriesExhausted = true;
    throw lastError;
  }
  
  async delay(attempt) {
    let delay = this.baseDelay * Math.pow(this.exponentialBase, attempt);
    delay = Math.min(delay, this.maxDelay);
    
    // Add jitter to prevent thundering herd
    if (this.jitter) {
      delay = delay * (0.5 + Math.random() * 0.5);
    }
    
    return new Promise(resolve => setTimeout(resolve, delay));
  }
}

// ============= CIRCUIT BREAKER =============
class CircuitBreaker {
  constructor(operation, options = {}) {
    this.operation = operation;
    this.failureThreshold = options.failureThreshold || 5;
    this.recoveryTimeout = options.recoveryTimeout || 60000;
    this.monitoringPeriod = options.monitoringPeriod || 10000;
    
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.successCount = 0;
    this.totalRequests = 0;
  }
  
  async execute(...args) {
    if (this.state === 'OPEN') {
      if (this.shouldAttemptReset()) {
        this.state = 'HALF_OPEN';
      } else {
        throw new AppError('Circuit breaker is OPEN', 503);
      }
    }
    
    try {
      const result = await this.operation(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.successCount++;
    this.totalRequests++;
    
    if (this.state === 'HALF_OPEN') {
      this.state = 'CLOSED';
    }
  }
  
  onFailure() {
    this.failureCount++;
    this.totalRequests++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
  
  shouldAttemptReset() {
    return Date.now() - this.lastFailureTime >= this.recoveryTimeout;
  }
  
  getStats() {
    return {
      state: this.state,
      failureCount: this.failureCount,
      successCount: this.successCount,
      totalRequests: this.totalRequests,
      failureRate: this.totalRequests > 0 ? this.failureCount / this.totalRequests : 0,
      lastFailureTime: this.lastFailureTime
    };
  }
}

// ============= USAGE EXAMPLES =============
// Error handling example
const errorHandler = new ErrorHandler({
  logErrors: true,
  notifyErrors: true,
  errorNotifier: async (error) => {
    // Send to monitoring service
    console.log('Notifying error service:', error.message);
  }
});

// Retry mechanism example
const retryHandler = new RetryHandler({
  maxRetries: 3,
  baseDelay: 1000,
  jitter: true
});

// Circuit breaker example
const apiCall = async (url) => {
  const response = await fetch(url);
  if (!response.ok) {
    throw new NetworkError(url, 'GET', response.status);
  }
  return response.json();
};

const protectedApiCall = new CircuitBreaker(apiCall, {
  failureThreshold: 3,
  recoveryTimeout: 30000
});

// Combined usage
async function robustApiCall(url) {
  const context = new ErrorContext()
    .addOperation('api_call', { url })
    .set('timestamp', new Date().toISOString());
  
  try {
    return await retryHandler.execute(async () => {
      return await protectedApiCall.execute(url);
    }, { url });
  } catch (error) {
    throw errorHandler.handle(context.attachToError(error));
  }
}

// Usage
(async () => {
  try {
    const data = await robustApiCall('https://api.example.com/data');
    console.log('Success:', data);
  } catch (error) {
    console.error('Failed after all attempts:', error.message);
  }
})();
```

## 🐛 Advanced Debugging

### **🔍 Debugging Tools และ Techniques**

```javascript
// ============= ADVANCED CONSOLE DEBUGGING =============
class AdvancedLogger {
  constructor(namespace = 'App') {
    this.namespace = namespace;
    this.startTime = Date.now();
    this.logs = [];
    this.enabled = process.env.NODE_ENV !== 'production';
  }
  
  // Enhanced logging with context
  log(level, message, data = {}) {
    if (!this.enabled) return;
    
    const timestamp = new Date().toISOString();
    const elapsed = Date.now() - this.startTime;
    
    const logEntry = {
      timestamp,
      elapsed,
      level: level.toUpperCase(),
      namespace: this.namespace,
      message,
      data
    };
    
    this.logs.push(logEntry);
    
    // Console output with colors
    const colors = {
      DEBUG: '\x1b[36m',   // Cyan
      INFO: '\x1b[32m',    // Green
      WARN: '\x1b[33m',    // Yellow
      ERROR: '\x1b[31m',   // Red
      RESET: '\x1b[0m'
    };
    
    const color = colors[level.toUpperCase()] || colors.RESET;
    
    console.log(
      `${color}[${timestamp}] ${this.namespace} ${level.toUpperCase()}${colors.RESET}:`,
      message,
      Object.keys(data).length > 0 ? data : ''
    );
  }
  
  debug(message, data) { this.log('debug', message, data); }
  info(message, data) { this.log('info', message, data); }
  warn(message, data) { this.log('warn', message, data); }
  error(message, data) { this.log('error', message, data); }
  
  // Performance timing
  time(label) {
    if (!this.enabled) return;
    console.time(`${this.namespace}:${label}`);
  }
  
  timeEnd(label) {
    if (!this.enabled) return;
    console.timeEnd(`${this.namespace}:${label}`);
  }
  
  // Memory usage
  memory() {
    if (!this.enabled) return;
    
    const usage = process.memoryUsage();
    this.info('Memory Usage', {
      rss: `${Math.round(usage.rss / 1024 / 1024)} MB`,
      heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)} MB`,
      heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)} MB`,
      external: `${Math.round(usage.external / 1024 / 1024)} MB`
    });
  }
  
  // Trace function calls
  trace(fn, name) {
    if (!this.enabled) return fn;
    
    return (...args) => {
      this.debug(`Calling ${name}`, { args });
      
      try {
        const result = fn(...args);
        
        if (result instanceof Promise) {
          return result
            .then(res => {
              this.debug(`${name} resolved`, { result: res });
              return res;
            })
            .catch(err => {
              this.error(`${name} rejected`, { error: err.message });
              throw err;
            });
        }
        
        this.debug(`${name} returned`, { result });
        return result;
      } catch (error) {
        this.error(`${name} threw error`, { error: error.message });
        throw error;
      }
    };
  }
  
  // Group related logs
  group(label, fn) {
    if (!this.enabled) return fn();
    
    console.group(`${this.namespace}: ${label}`);
    try {
      return fn();
    } finally {
      console.groupEnd();
    }
  }
  
  // Conditional logging
  assert(condition, message, data = {}) {
    if (!condition) {
      this.error(`Assertion failed: ${message}`, data);
      console.trace('Assertion stack trace');
    }
  }
  
  // Export logs
  exportLogs() {
    return JSON.stringify(this.logs, null, 2);
  }
  
  // Clear logs
  clear() {
    this.logs = [];
    console.clear();
  }
}

// ============= PERFORMANCE PROFILER =============
class PerformanceProfiler {
  constructor() {
    this.profiles = new Map();
    this.marks = new Map();
  }
  
  // Mark a point in time
  mark(name) {
    this.marks.set(name, performance.now());
  }
  
  // Measure between two marks
  measure(name, startMark, endMark) {
    const startTime = this.marks.get(startMark);
    const endTime = this.marks.get(endMark);
    
    if (startTime === undefined || endTime === undefined) {
      throw new Error(`Mark not found: ${startMark} or ${endMark}`);
    }
    
    const duration = endTime - startTime;
    
    if (!this.profiles.has(name)) {
      this.profiles.set(name, []);
    }
    
    this.profiles.get(name).push({
      duration,
      timestamp: Date.now(),
      startMark,
      endMark
    });
    
    return duration;
  }
  
  // Profile a function
  profile(fn, name) {
    return (...args) => {
      const startName = `${name}-start-${Date.now()}`;
      const endName = `${name}-end-${Date.now()}`;
      
      this.mark(startName);
      
      try {
        const result = fn(...args);
        
        if (result instanceof Promise) {
          return result.finally(() => {
            this.mark(endName);
            this.measure(name, startName, endName);
          });
        }
        
        this.mark(endName);
        this.measure(name, startName, endName);
        return result;
      } catch (error) {
        this.mark(endName);
        this.measure(name, startName, endName);
        throw error;
      }
    };
  }
  
  // Get statistics for a profile
  getStats(name) {
    const measurements = this.profiles.get(name);
    if (!measurements || measurements.length === 0) {
      return null;
    }
    
    const durations = measurements.map(m => m.duration);
    durations.sort((a, b) => a - b);
    
    return {
      count: durations.length,
      min: durations[0],
      max: durations[durations.length - 1],
      mean: durations.reduce((sum, d) => sum + d, 0) / durations.length,
      median: durations[Math.floor(durations.length / 2)],
      p95: durations[Math.floor(durations.length * 0.95)],
      p99: durations[Math.floor(durations.length * 0.99)]
    };
  }
  
  // Clear all profiles
  clear() {
    this.profiles.clear();
    this.marks.clear();
  }
  
  // Generate report
  report() {
    const report = {};
    
    for (const [name, measurements] of this.profiles) {
      report[name] = this.getStats(name);
    }
    
    return report;
  }
}

// ============= ASYNC DEBUGGING UTILITIES =============
class AsyncDebugger {
  static timeoutPromise(promise, timeout = 5000, name = 'Promise') {
    return Promise.race([
      promise,
      new Promise((_, reject) => {
        setTimeout(() => {
          reject(new Error(`${name} timed out after ${timeout}ms`));
        }, timeout);
      })
    ]);
  }
  
  static logPromise(promise, name = 'Promise') {
    console.log(`${name} started`);
    
    return promise
      .then(result => {
        console.log(`${name} resolved:`, result);
        return result;
      })
      .catch(error => {
        console.error(`${name} rejected:`, error);
        throw error;
      });
  }
  
  static async promiseAllSettledWithLogs(promises, names = []) {
    const results = await Promise.allSettled(promises);
    
    results.forEach((result, index) => {
      const name = names[index] || `Promise ${index}`;
      
      if (result.status === 'fulfilled') {
        console.log(`✅ ${name} fulfilled:`, result.value);
      } else {
        console.error(`❌ ${name} rejected:`, result.reason);
      }
    });
    
    return results;
  }
  
  static createCancellablePromise(executor) {
    let cancelCallback;
    let isCancelled = false;
    
    const promise = new Promise((resolve, reject) => {
      cancelCallback = () => {
        isCancelled = true;
        reject(new Error('Promise was cancelled'));
      };
      
      executor(
        (value) => {
          if (!isCancelled) resolve(value);
        },
        (reason) => {
          if (!isCancelled) reject(reason);
        }
      );
    });
    
    promise.cancel = cancelCallback;
    promise.isCancelled = () => isCancelled;
    
    return promise;
  }
}

// ============= MEMORY LEAK DETECTOR =============
class MemoryLeakDetector {
  constructor(options = {}) {
    this.checkInterval = options.checkInterval || 30000; // 30 seconds
    this.thresholdMB = options.thresholdMB || 100;
    this.samples = [];
    this.maxSamples = options.maxSamples || 10;
    this.running = false;
  }
  
  start() {
    if (this.running) return;
    
    this.running = true;
    this.intervalId = setInterval(() => {
      this.checkMemory();
    }, this.checkInterval);
    
    console.log('Memory leak detector started');
  }
  
  stop() {
    if (!this.running) return;
    
    this.running = false;
    clearInterval(this.intervalId);
    console.log('Memory leak detector stopped');
  }
  
  checkMemory() {
    const usage = process.memoryUsage();
    const heapUsedMB = usage.heapUsed / 1024 / 1024;
    
    this.samples.push({
      timestamp: Date.now(),
      heapUsed: heapUsedMB,
      heapTotal: usage.heapTotal / 1024 / 1024,
      rss: usage.rss / 1024 / 1024
    });
    
    // Keep only recent samples
    if (this.samples.length > this.maxSamples) {
      this.samples.shift();
    }
    
    // Check for potential leak
    if (this.samples.length >= 3) {
      const trend = this.calculateTrend();
      
      if (trend > this.thresholdMB) {
        console.warn('🚨 Potential memory leak detected!', {
          currentUsage: `${heapUsedMB.toFixed(2)} MB`,
          trend: `+${trend.toFixed(2)} MB`,
          samples: this.samples.slice(-3)
        });
      }
    }
  }
  
  calculateTrend() {
    if (this.samples.length < 2) return 0;
    
    const recent = this.samples.slice(-3);
    const first = recent[0].heapUsed;
    const last = recent[recent.length - 1].heapUsed;
    
    return last - first;
  }
  
  getReport() {
    return {
      currentUsage: process.memoryUsage(),
      samples: this.samples,
      trend: this.calculateTrend(),
      isRunning: this.running
    };
  }
}

// ============= USAGE EXAMPLES =============
// Logger usage
const logger = new AdvancedLogger('MyApp');

// Profiler usage
const profiler = new PerformanceProfiler();

// Example function with debugging
const profiledFunction = profiler.profile(
  logger.trace(async (data) => {
    logger.debug('Processing data', { data });
    
    // Simulate async work
    await new Promise(resolve => setTimeout(resolve, 100));
    
    if (Math.random() < 0.1) {
      throw new Error('Random error for testing');
    }
    
    return { processed: data, timestamp: Date.now() };
  }, 'processData'),
  'processData'
);

// Memory leak detector
const memoryDetector = new MemoryLeakDetector({
  checkInterval: 10000,
  thresholdMB: 50
});

memoryDetector.start();

// Example usage
async function debugExample() {
  logger.group('Debug Example', async () => {
    try {
      logger.memory();
      
      const result = await AsyncDebugger.timeoutPromise(
        profiledFunction({ test: 'data' }),
        5000,
        'processData'
      );
      
      logger.info('Operation completed', { result });
      
      // Show performance stats
      console.log('Performance Stats:', profiler.getStats('processData'));
      
    } catch (error) {
      logger.error('Operation failed', { error: error.message });
    }
  });
}

// Run example
debugExample();

// Clean up after 60 seconds
setTimeout(() => {
  memoryDetector.stop();
  profiler.clear();
  logger.clear();
}, 60000);
```

## 🚀 Next.js Error Handling

### **⚡ Production Error Handling**

```jsx
// ============= GLOBAL ERROR BOUNDARY =============
// components/ErrorBoundary.jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null, 
      errorInfo: null,
      errorId: null
    };
  }
  
  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { 
      hasError: true,
      errorId: Math.random().toString(36).substr(2, 9)
    };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log error to reporting service
    this.logErrorToService(error, errorInfo);
    
    this.setState({
      error,
      errorInfo
    });
  }
  
  logErrorToService = async (error, errorInfo) => {
    const errorReport = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      errorBoundary: this.constructor.name,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      userId: this.props.userId,
      errorId: this.state.errorId
    };
    
    try {
      // Send to error reporting service
      await fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorReport)
      });
    } catch (reportingError) {
      console.error('Failed to report error:', reportingError);
    }
  };
  
  handleRetry = () => {
    this.setState({ 
      hasError: false, 
      error: null, 
      errorInfo: null,
      errorId: null
    });
  };
  
  render() {
    if (this.state.hasError) {
      const { fallback: Fallback, showDetails } = this.props;
      
      if (Fallback) {
        return (
          <Fallback
            error={this.state.error}
            errorInfo={this.state.errorInfo}
            errorId={this.state.errorId}
            onRetry={this.handleRetry}
          />
        );
      }
      
      return (
        <div className="error-boundary">
          <div className="error-container">
            <h2>🚨 Something went wrong</h2>
            <p>We're sorry, but something unexpected happened.</p>
            
            {this.state.errorId && (
              <p className="error-id">
                Error ID: <code>{this.state.errorId}</code>
              </p>
            )}
            
            <div className="error-actions">
              <button onClick={this.handleRetry} className="retry-btn">
                Try Again
              </button>
              
              <button 
                onClick={() => window.location.reload()} 
                className="reload-btn"
              >
                Reload Page
              </button>
            </div>
            
            {showDetails && process.env.NODE_ENV === 'development' && (
              <details className="error-details">
                <summary>Error Details (Development Only)</summary>
                <pre>
                  <strong>Error:</strong> {this.state.error?.message}
                  {'\n\n'}
                  <strong>Stack:</strong> {this.state.error?.stack}
                  {'\n\n'}
                  <strong>Component Stack:</strong> {this.state.errorInfo?.componentStack}
                </pre>
              </details>
            )}
          </div>
        </div>
      );
    }
    
    return this.props.children;
  }
}

export default ErrorBoundary;

// ============= CUSTOM ERROR PAGES =============
// pages/_error.js
function ErrorPage({ statusCode, hasGetInitialPropsRun, err }) {
  const errorId = React.useMemo(() => 
    Math.random().toString(36).substr(2, 9), []
  );
  
  React.useEffect(() => {
    // Log client-side error
    if (err && !hasGetInitialPropsRun) {
      logError(err, { 
        statusCode, 
        errorId,
        context: 'client-side' 
      });
    }
  }, [err, hasGetInitialPropsRun, statusCode, errorId]);
  
  const getErrorMessage = (statusCode) => {
    const messages = {
      400: 'Bad Request - The request was malformed',
      401: 'Unauthorized - Please log in to access this page',
      403: 'Forbidden - You don\'t have permission to access this resource',
      404: 'Page Not Found - The page you\'re looking for doesn\'t exist',
      500: 'Internal Server Error - Something went wrong on our end',
      502: 'Bad Gateway - The server received an invalid response',
      503: 'Service Unavailable - The server is temporarily unavailable',
      504: 'Gateway Timeout - The server took too long to respond'
    };
    
    return messages[statusCode] || 'An unexpected error occurred';
  };
  
  const getErrorIcon = (statusCode) => {
    if (statusCode === 404) return '🔍';
    if (statusCode >= 500) return '🚨';
    if (statusCode >= 400) return '⚠️';
    return '❓';
  };
  
  return (
    <div className="error-page">
      <div className="error-content">
        <div className="error-icon">
          {getErrorIcon(statusCode)}
        </div>
        
        <h1 className="error-title">
          {statusCode || 'Error'}
        </h1>
        
        <p className="error-message">
          {getErrorMessage(statusCode)}
        </p>
        
        <div className="error-actions">
          <Link href="/">
            <a className="home-btn">Go Home</a>
          </Link>
          
          <button 
            onClick={() => window.history.back()} 
            className="back-btn"
          >
            Go Back
          </button>
        </div>
        
        {process.env.NODE_ENV === 'development' && err && (
          <details className="error-debug">
            <summary>Debug Information</summary>
            <pre>{err.stack}</pre>
          </details>
        )}
        
        <div className="error-id">
          Error ID: {errorId}
        </div>
      </div>
    </div>
  );
}

ErrorPage.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404;
  
  // Log server-side error
  if (err) {
    logError(err, { 
      statusCode, 
      context: 'server-side',
      url: res?.req?.url,
      userAgent: res?.req?.headers?.['user-agent']
    });
  }
  
  return { statusCode, hasGetInitialPropsRun: true };
};

async function logError(error, context) {
  try {
    const errorReport = {
      message: error.message,
      stack: error.stack,
      statusCode: context.statusCode,
      context: context.context,
      timestamp: new Date().toISOString(),
      url: context.url || (typeof window !== 'undefined' ? window.location.href : ''),
      userAgent: context.userAgent || (typeof navigator !== 'undefined' ? navigator.userAgent : ''),
      errorId: context.errorId || Math.random().toString(36).substr(2, 9)
    };
    
    // Send to logging service
    if (typeof fetch !== 'undefined') {
      await fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorReport)
      });
    }
  } catch (loggingError) {
    console.error('Failed to log error:', loggingError);
  }
}

export default ErrorPage;

// ============= API ERROR HANDLING =============
// pages/api/errors.js
import { ErrorHandler, ErrorContext } from '../../lib/error-handling';

const errorHandler = new ErrorHandler({
  logErrors: true,
  notifyErrors: process.env.NODE_ENV === 'production',
  errorNotifier: async (error) => {
    // Integrate with error monitoring service (e.g., Sentry, Bugsnag)
    console.log('Would notify monitoring service:', error);
  }
});

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const errorReport = req.body;
    
    // Validate error report
    if (!errorReport.message || !errorReport.timestamp) {
      throw new ValidationError('errorReport', errorReport, 'Invalid error report format');
    }
    
    // Create error context
    const context = new ErrorContext()
      .addRequest(req)
      .addOperation('error_logging', { errorId: errorReport.errorId });
    
    // Log the error
    console.error('Client Error Report:', {
      ...errorReport,
      context: context.toObject()
    });
    
    // Store in database or send to monitoring service
    // await saveErrorReport(errorReport);
    
    res.status(200).json({ 
      success: true, 
      errorId: errorReport.errorId 
    });
    
  } catch (error) {
    const handledError = errorHandler.handle(error);
    res.status(handledError.statusCode || 500).json({
      error: handledError.message
    });
  }
}

// ============= HOOK FOR ERROR HANDLING =============
// hooks/useErrorHandler.js
import { useState, useCallback } from 'react';

export const useErrorHandler = () => {
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const handleAsync = useCallback(async (asyncFn, options = {}) => {
    const { onError, onSuccess, showLoading = true } = options;
    
    try {
      if (showLoading) setIsLoading(true);
      setError(null);
      
      const result = await asyncFn();
      
      if (onSuccess) onSuccess(result);
      return result;
      
    } catch (err) {
      const errorInfo = {
        message: err.message,
        stack: err.stack,
        timestamp: new Date().toISOString(),
        context: options.context || {}
      };
      
      setError(errorInfo);
      
      if (onError) {
        onError(err);
      } else {
        // Log to error reporting service
        logError(err, errorInfo);
      }
      
      throw err;
    } finally {
      if (showLoading) setIsLoading(false);
    }
  }, []);
  
  const clearError = useCallback(() => {
    setError(null);
  }, []);
  
  const retryLastOperation = useCallback((operation) => {
    if (operation) {
      return handleAsync(operation);
    }
  }, [handleAsync]);
  
  return {
    error,
    isLoading,
    handleAsync,
    clearError,
    retryLastOperation
  };
};

// ============= COMPONENT WITH ERROR HANDLING =============
// components/DataFetcher.jsx
import { useState, useEffect } from 'react';
import { useErrorHandler } from '../hooks/useErrorHandler';

const DataFetcher = ({ url, children, fallback }) => {
  const [data, setData] = useState(null);
  const { error, isLoading, handleAsync, clearError } = useErrorHandler();
  
  const fetchData = useCallback(async () => {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new Error(`Failed to fetch: ${response.status} ${response.statusText}`);
    }
    
    return response.json();
  }, [url]);
  
  useEffect(() => {
    handleAsync(fetchData, {
      context: { component: 'DataFetcher', url },
      onSuccess: setData
    });
  }, [url, fetchData, handleAsync]);
  
  const handleRetry = () => {
    clearError();
    handleAsync(fetchData, {
      context: { component: 'DataFetcher', url, retry: true },
      onSuccess: setData
    });
  };
  
  if (isLoading) {
    return <div className="loading">Loading...</div>;
  }
  
  if (error) {
    return (
      <div className="error-state">
        <h3>Failed to load data</h3>
        <p>{error.message}</p>
        <button onClick={handleRetry}>Retry</button>
        {fallback}
      </div>
    );
  }
  
  if (!data) {
    return <div className="no-data">No data available</div>;
  }
  
  return children(data);
};

export default DataFetcher;

// ============= APP WRAPPER WITH ERROR HANDLING =============
// pages/_app.js
import ErrorBoundary from '../components/ErrorBoundary';
import { MemoryLeakDetector } from '../lib/debugging';

// Initialize memory leak detector in development
if (process.env.NODE_ENV === 'development' && typeof window !== 'undefined') {
  const memoryDetector = new MemoryLeakDetector({
    checkInterval: 30000,
    thresholdMB: 100
  });
  memoryDetector.start();
}

function MyApp({ Component, pageProps }) {
  return (
    <ErrorBoundary
      fallback={({ error, errorId, onRetry }) => (
        <div className="global-error">
          <h1>Application Error</h1>
          <p>Something went wrong in the application.</p>
          <p>Error ID: {errorId}</p>
          <button onClick={onRetry}>Try Again</button>
        </div>
      )}
      showDetails={process.env.NODE_ENV === 'development'}
    >
      <Component {...pageProps} />
    </ErrorBoundary>
  );
}

export default MyApp;
```

## 🎯 แบบฝึกหัด

### **🚨 แบบฝึกหัดที่ 1: Custom Error System**
สร้าง comprehensive error system:

```javascript
// TODO: สร้าง custom error system

// 1. Error taxonomy
class ErrorTaxonomy {
  // Define error categories and types
  // Create error inheritance hierarchy
  // Implement error serialization
}

// 2. Error aggregation service
class ErrorAggregator {
  // Collect and aggregate errors
  // Generate error reports
  // Detect error patterns
}

// 3. Error recovery system
class ErrorRecovery {
  // Implement automatic recovery strategies
  // State restoration mechanisms
  // Graceful degradation patterns
}
```

### **🔍 แบบฝึกหัดที่ 2: Debugging Tools**
สร้าง advanced debugging tools:

```javascript
// TODO: สร้าง debugging tools

// 1. Call stack analyzer
class CallStackAnalyzer {
  // Analyze call stacks
  // Detect recursive calls
  // Performance bottleneck detection
}

// 2. State mutation tracker
class StateMutationTracker {
  // Track state changes
  // Detect unexpected mutations
  // Generate state change reports
}

// 3. Network request debugger
class NetworkDebugger {
  // Monitor network requests
  // Detect failed requests
  // Performance analysis
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Error Handling**
สร้าง production-ready error handling:

```jsx
// TODO: สร้าง production error handling

// 1. Error monitoring dashboard
const ErrorDashboard = () => {
  // Display error statistics
  // Real-time error tracking
  // Error resolution tools
};

// 2. Automatic error recovery
const ErrorRecoveryProvider = ({ children }) => {
  // Implement automatic recovery
  // Fallback mechanisms
  // Progressive degradation
};

// 3. Error analytics
const ErrorAnalytics = () => {
  // Error trend analysis
  // User impact assessment
  // Performance correlation
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 13: Regular Expressions](./13-regex.md)

**บทถัดไป**: [บทที่ 15: Performance Optimization](./15-performance.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)
- [Node.js Error Handling](https://nodejs.org/en/docs/guides/error-handling)
- [React Error Boundaries](https://reactjs.org/docs/error-boundaries.html)
- [Next.js Error Handling](https://nextjs.org/docs/advanced-features/error-handling)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Advanced Error Handling** - custom errors, error context, และ error factory
- **Debugging Techniques** - logging, profiling, และ memory leak detection
- **Retry Mechanisms** - circuit breaker และ resilience patterns
- **Next.js Error Handling** - error boundaries, custom error pages, และ API error handling
- **Production Monitoring** - error reporting และ analytics

Error handling และ debugging ที่ดีเป็นสิ่งสำคัญสำหรับ production applications! 🚀
