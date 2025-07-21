# บทที่ 17: Advanced Functions

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ advanced function patterns และ techniques
- ทำความเข้าใจ higher-order functions และ function composition
- เรียนรู้ currying, partial application, และ memoization
- ประยุกต์ใช้ใน Next.js สำหรับ sophisticated function architectures

## 🔄 Higher-Order Functions

### **🎪 Function Composition Patterns**

```javascript
// ============= FUNCTION COMPOSITION =============
// utils/functional.js

// Basic composition utility
export const compose = (...fns) => (value) =>
  fns.reduceRight((acc, fn) => fn(acc), value);

// Pipe utility (left-to-right composition)
export const pipe = (...fns) => (value) =>
  fns.reduce((acc, fn) => fn(acc), value);

// Async composition
export const composeAsync = (...fns) => (value) =>
  fns.reduceRight(
    (acc, fn) => acc.then(fn),
    Promise.resolve(value)
  );

export const pipeAsync = (...fns) => (value) =>
  fns.reduce(
    (acc, fn) => acc.then(fn),
    Promise.resolve(value)
  );

// ============= ADVANCED COMPOSITION UTILITIES =============
export class FunctionComposer {
  constructor() {
    this.pipeline = [];
    this.errorHandlers = [];
    this.middleware = [];
  }

  // Add function to pipeline
  add(fn, options = {}) {
    const { 
      condition = () => true,
      transform = (x) => x,
      errorHandler = null,
      retry = 0,
      timeout = null 
    } = options;

    this.pipeline.push({
      fn,
      condition,
      transform,
      errorHandler,
      retry,
      timeout,
      name: fn.name || 'anonymous'
    });

    return this;
  }

  // Add middleware
  use(middleware) {
    this.middleware.push(middleware);
    return this;
  }

  // Add global error handler
  catch(handler) {
    this.errorHandlers.push(handler);
    return this;
  }

  // Execute pipeline
  async execute(input) {
    let result = input;
    const context = {
      input,
      steps: [],
      metadata: {}
    };

    // Apply middleware
    for (const middleware of this.middleware) {
      await middleware(context, result);
    }

    // Execute pipeline
    for (const step of this.pipeline) {
      try {
        // Check condition
        if (!step.condition(result, context)) {
          context.steps.push({
            name: step.name,
            skipped: true,
            input: result,
            output: result
          });
          continue;
        }

        // Transform input
        const transformedInput = step.transform(result);

        // Execute with timeout if specified
        const executeWithTimeout = async () => {
          if (step.timeout) {
            return Promise.race([
              step.fn(transformedInput, context),
              new Promise((_, reject) =>
                setTimeout(() => reject(new Error('Timeout')), step.timeout)
              )
            ]);
          }
          return step.fn(transformedInput, context);
        };

        // Execute with retry
        let attempts = 0;
        while (attempts <= step.retry) {
          try {
            const stepResult = await executeWithTimeout();
            
            context.steps.push({
              name: step.name,
              success: true,
              input: transformedInput,
              output: stepResult,
              attempts: attempts + 1
            });

            result = stepResult;
            break;
          } catch (error) {
            attempts++;
            
            if (attempts > step.retry) {
              if (step.errorHandler) {
                result = await step.errorHandler(error, transformedInput, context);
                context.steps.push({
                  name: step.name,
                  error: true,
                  errorHandled: true,
                  input: transformedInput,
                  output: result,
                  attempts
                });
                break;
              }
              throw error;
            }
            
            // Wait before retry
            await new Promise(resolve => 
              setTimeout(resolve, Math.pow(2, attempts - 1) * 1000)
            );
          }
        }
      } catch (error) {
        // Try global error handlers
        for (const handler of this.errorHandlers) {
          try {
            result = await handler(error, result, context);
            context.steps.push({
              name: step.name,
              error: true,
              globalHandled: true,
              input: result,
              output: result
            });
            break;
          } catch (handlerError) {
            continue;
          }
        }

        // If no handler worked, rethrow
        if (!context.steps.some(s => s.globalHandled)) {
          throw error;
        }
      }
    }

    return { result, context };
  }

  // Clone pipeline
  clone() {
    const cloned = new FunctionComposer();
    cloned.pipeline = [...this.pipeline];
    cloned.errorHandlers = [...this.errorHandlers];
    cloned.middleware = [...this.middleware];
    return cloned;
  }

  // Create reusable function
  toFunction() {
    return (input) => this.execute(input);
  }
}

// ============= REAL-WORLD COMPOSITION EXAMPLE =============
// lib/data-processing/pipeline.js
import { FunctionComposer } from '@/utils/functional';

// Data validation functions
const validateRequired = (data) => {
  const required = ['id', 'name', 'email'];
  const missing = required.filter(field => !data[field]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required fields: ${missing.join(', ')}`);
  }
  
  return data;
};

const validateEmail = (data) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!emailRegex.test(data.email)) {
    throw new Error('Invalid email format');
  }
  
  return data;
};

const sanitizeData = (data) => ({
  ...data,
  name: data.name.trim(),
  email: data.email.toLowerCase().trim(),
  phone: data.phone?.replace(/\D/g, '') || null
});

const enrichData = async (data) => {
  // Simulate API call to enrich data
  const enrichment = await fetch(`/api/enrich/${data.id}`);
  const enrichmentData = await enrichment.json();
  
  return {
    ...data,
    ...enrichmentData,
    processed_at: new Date().toISOString()
  };
};

const saveToDatabase = async (data) => {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    throw new Error('Failed to save to database');
  }
  
  return response.json();
};

// Create user processing pipeline
export const createUserProcessingPipeline = () => {
  const pipeline = new FunctionComposer();

  // Add validation middleware
  pipeline.use(async (context, data) => {
    context.metadata.startTime = Date.now();
    context.metadata.userId = data.id;
  });

  // Add processing steps
  pipeline
    .add(validateRequired, {
      errorHandler: (error, data) => {
        console.error('Validation failed:', error.message);
        return { ...data, validation_errors: [error.message] };
      }
    })
    .add(validateEmail, {
      condition: (data) => !data.validation_errors,
      errorHandler: (error, data) => ({
        ...data,
        validation_errors: [...(data.validation_errors || []), error.message]
      })
    })
    .add(sanitizeData, {
      condition: (data) => !data.validation_errors
    })
    .add(enrichData, {
      condition: (data) => !data.validation_errors,
      retry: 2,
      timeout: 5000,
      errorHandler: (error, data) => {
        console.warn('Enrichment failed, proceeding without enrichment:', error.message);
        return { ...data, enrichment_failed: true };
      }
    })
    .add(saveToDatabase, {
      condition: (data) => !data.validation_errors,
      retry: 3,
      transform: (data) => {
        // Remove internal fields before saving
        const { validation_errors, enrichment_failed, ...saveData } = data;
        return saveData;
      }
    })
    .catch(async (error, data, context) => {
      console.error('Pipeline failed:', error);
      
      // Log error with context
      await fetch('/api/error-log', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          error: error.message,
          data,
          context: context.steps,
          userId: context.metadata.userId
        })
      });
      
      return { error: true, message: error.message, original_data: data };
    });

  return pipeline;
};

// ============= FUNCTIONAL PROGRAMMING UTILITIES =============
// utils/fp-helpers.js

// Currying utilities
export const curry = (fn) => {
  const arity = fn.length;
  
  return function curried(...args) {
    if (args.length >= arity) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
};

// Partial application
export const partial = (fn, ...args1) => 
  (...args2) => fn(...args1, ...args2);

// Function memoization
export const memoize = (fn, keyGenerator = (...args) => JSON.stringify(args)) => {
  const cache = new Map();
  
  const memoized = (...args) => {
    const key = keyGenerator(...args);
    
    if (cache.has(key)) {
      memoized.hits++;
      return cache.get(key);
    }
    
    const result = fn(...args);
    cache.set(key, result);
    memoized.misses++;
    
    return result;
  };
  
  memoized.cache = cache;
  memoized.hits = 0;
  memoized.misses = 0;
  memoized.clear = () => {
    cache.clear();
    memoized.hits = 0;
    memoized.misses = 0;
  };
  
  return memoized;
};

// Advanced memoization with TTL
export const memoizeWithTTL = (fn, ttlMs = 60000, keyGenerator = (...args) => JSON.stringify(args)) => {
  const cache = new Map();
  
  const memoized = (...args) => {
    const key = keyGenerator(...args);
    const now = Date.now();
    
    if (cache.has(key)) {
      const { value, timestamp } = cache.get(key);
      
      if (now - timestamp < ttlMs) {
        memoized.hits++;
        return value;
      } else {
        cache.delete(key);
        memoized.expired++;
      }
    }
    
    const result = fn(...args);
    cache.set(key, { value: result, timestamp: now });
    memoized.misses++;
    
    return result;
  };
  
  memoized.cache = cache;
  memoized.hits = 0;
  memoized.misses = 0;
  memoized.expired = 0;
  memoized.clear = () => {
    cache.clear();
    memoized.hits = 0;
    memoized.misses = 0;
    memoized.expired = 0;
  };
  
  // Cleanup expired entries periodically
  memoized.cleanup = () => {
    const now = Date.now();
    for (const [key, { timestamp }] of cache.entries()) {
      if (now - timestamp >= ttlMs) {
        cache.delete(key);
      }
    }
  };
  
  return memoized;
};

// Function debouncing
export const debounce = (fn, delay) => {
  let timeoutId;
  
  const debounced = (...args) => {
    clearTimeout(timeoutId);
    
    return new Promise((resolve, reject) => {
      timeoutId = setTimeout(async () => {
        try {
          const result = await fn(...args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }, delay);
    });
  };
  
  debounced.cancel = () => {
    clearTimeout(timeoutId);
  };
  
  debounced.flush = (...args) => {
    clearTimeout(timeoutId);
    return fn(...args);
  };
  
  return debounced;
};

// Function throttling
export const throttle = (fn, limit) => {
  let inThrottle;
  let lastFunc;
  let lastRan;
  
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      lastRan = Date.now();
      inThrottle = true;
    } else {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(() => {
        if ((Date.now() - lastRan) >= limit) {
          fn.apply(this, args);
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  };
};

// Function retry utility
export const retry = (fn, options = {}) => {
  const {
    attempts = 3,
    delay = 1000,
    backoff = 'exponential',
    condition = () => true
  } = options;
  
  return async (...args) => {
    let lastError;
    
    for (let i = 0; i < attempts; i++) {
      try {
        const result = await fn(...args);
        return result;
      } catch (error) {
        lastError = error;
        
        if (i === attempts - 1 || !condition(error, i + 1)) {
          throw error;
        }
        
        // Calculate delay
        let waitTime = delay;
        if (backoff === 'exponential') {
          waitTime = delay * Math.pow(2, i);
        } else if (backoff === 'linear') {
          waitTime = delay * (i + 1);
        }
        
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
    
    throw lastError;
  };
};
```

## 🧮 Currying and Partial Application

### **🔗 Advanced Currying Patterns**

```javascript
// ============= ADVANCED CURRYING =============
// utils/currying.js

// Auto-currying decorator
export const autoCurry = (fn) => {
  const curried = (...args) => {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return (...nextArgs) => curried(...args, ...nextArgs);
  };
  
  // Add utility methods
  curried.arity = fn.length;
  curried.name = fn.name;
  curried.uncurried = fn;
  
  return curried;
};

// Placeholder-based currying
export const _ = Symbol('placeholder');

export const curriedWithPlaceholders = (fn) => {
  const curried = (...args) => {
    // Check if we have enough non-placeholder arguments
    const filledArgs = args.filter(arg => arg !== _);
    
    if (filledArgs.length >= fn.length) {
      // Fill placeholders and call function
      const finalArgs = [];
      let filledIndex = 0;
      
      for (let i = 0; i < Math.max(args.length, fn.length); i++) {
        if (args[i] === _) {
          finalArgs[i] = arguments[fn.length + filledIndex];
          filledIndex++;
        } else {
          finalArgs[i] = args[i];
        }
      }
      
      return fn(...finalArgs.slice(0, fn.length));
    }
    
    return (...nextArgs) => {
      // Merge arguments, replacing placeholders
      const mergedArgs = [...args];
      let nextIndex = 0;
      
      for (let i = 0; i < mergedArgs.length; i++) {
        if (mergedArgs[i] === _ && nextIndex < nextArgs.length) {
          mergedArgs[i] = nextArgs[nextIndex++];
        }
      }
      
      // Add remaining arguments
      while (nextIndex < nextArgs.length) {
        mergedArgs.push(nextArgs[nextIndex++]);
      }
      
      return curried(...mergedArgs);
    };
  };
  
  return curried;
};

// ============= CURRIED UTILITY FUNCTIONS =============
// Math utilities
export const add = autoCurry((a, b) => a + b);
export const multiply = autoCurry((a, b) => a * b);
export const subtract = autoCurry((a, b) => a - b);
export const divide = autoCurry((a, b) => a / b);
export const power = autoCurry((base, exponent) => Math.pow(base, exponent));

// String utilities
export const split = autoCurry((separator, str) => str.split(separator));
export const join = autoCurry((separator, arr) => arr.join(separator));
export const replace = autoCurry((search, replacement, str) => 
  str.replace(search, replacement)
);
export const match = autoCurry((pattern, str) => str.match(pattern));

// Array utilities
export const map = autoCurry((fn, arr) => arr.map(fn));
export const filter = autoCurry((predicate, arr) => arr.filter(predicate));
export const reduce = autoCurry((reducer, initial, arr) => 
  arr.reduce(reducer, initial)
);
export const find = autoCurry((predicate, arr) => arr.find(predicate));
export const includes = autoCurry((item, arr) => arr.includes(item));

// Object utilities
export const prop = autoCurry((key, obj) => obj[key]);
export const assoc = autoCurry((key, value, obj) => ({ ...obj, [key]: value }));
export const dissoc = autoCurry((key, obj) => {
  const { [key]: _, ...rest } = obj;
  return rest;
});

// Comparison utilities
export const equals = autoCurry((a, b) => a === b);
export const lt = autoCurry((a, b) => a < b);
export const gt = autoCurry((a, b) => a > b);
export const lte = autoCurry((a, b) => a <= b);
export const gte = autoCurry((a, b) => a >= b);

// ============= PRACTICAL CURRYING EXAMPLES =============
// lib/data-transformers.js
import { 
  map, filter, reduce, prop, assoc, 
  split, join, replace, multiply, add 
} from '@/utils/currying';

// Create specialized functions
export const getNames = map(prop('name'));
export const getActiveUsers = filter(prop('active'));
export const sumAges = reduce(add, 0);
export const doubleNumbers = map(multiply(2));

// Processing pipelines using curried functions
export const processUserNames = pipe(
  getNames,                           // Extract names
  map(split(' ')),                   // Split into first/last
  map(map(replace(/[^a-zA-Z]/g, ''))), // Clean non-letters
  map(join(' ')),                    // Rejoin names
  filter(name => name.length > 0)    // Remove empty names
);

// Data transformation pipeline
export const transformUserData = (users) => {
  const activeUsers = getActiveUsers(users);
  const names = getNames(activeUsers);
  const totalAge = pipe(
    map(prop('age')),
    sumAges
  )(activeUsers);
  
  return {
    names,
    totalUsers: activeUsers.length,
    averageAge: activeUsers.length > 0 ? totalAge / activeUsers.length : 0
  };
};

// API response transformer
export const createApiTransformer = (transformations) => {
  const applyTransformations = pipe(...transformations);
  
  return autoCurry((status, data) => ({
    status,
    data: applyTransformations(data),
    timestamp: new Date().toISOString()
  }));
};

// Usage example
const transformUserResponse = createApiTransformer([
  filter(prop('active')),
  map(user => assoc('fullName', `${user.firstName} ${user.lastName}`, user)),
  map(dissoc('password'))
]);

const successResponse = transformUserResponse('success');
const errorResponse = transformUserResponse('error');

// ============= REACT HOOKS WITH CURRYING =============
// hooks/useCurriedCallbacks.js
import { useCallback } from 'react';
import { autoCurry } from '@/utils/currying';

export const useCurriedCallback = (fn, deps) => {
  return useCallback(autoCurry(fn), deps);
};

// hooks/useFormHandlers.js
export const useFormHandlers = () => {
  const handleFieldChange = useCurriedCallback((field, value, setState) => {
    setState(prevState => ({
      ...prevState,
      [field]: value
    }));
  }, []);

  const handleNestedChange = useCurriedCallback((path, value, setState) => {
    setState(prevState => {
      const newState = { ...prevState };
      const keys = path.split('.');
      let current = newState;
      
      for (let i = 0; i < keys.length - 1; i++) {
        if (!current[keys[i]]) current[keys[i]] = {};
        current = current[keys[i]];
      }
      
      current[keys[keys.length - 1]] = value;
      return newState;
    });
  }, []);

  const handleArrayChange = useCurriedCallback((index, value, field, setState) => {
    setState(prevState => ({
      ...prevState,
      [field]: prevState[field].map((item, i) => 
        i === index ? value : item
      )
    }));
  }, []);

  return {
    handleFieldChange,
    handleNestedChange,
    handleArrayChange
  };
};

// ============= VALIDATION WITH CURRYING =============
// lib/validation/curried-validators.js

// Basic validators
export const required = autoCurry((message, value) => {
  if (!value || (typeof value === 'string' && value.trim() === '')) {
    return message || 'This field is required';
  }
  return null;
});

export const minLength = autoCurry((min, message, value) => {
  if (value && value.length < min) {
    return message || `Minimum length is ${min} characters`;
  }
  return null;
});

export const maxLength = autoCurry((max, message, value) => {
  if (value && value.length > max) {
    return message || `Maximum length is ${max} characters`;
  }
  return null;
});

export const pattern = autoCurry((regex, message, value) => {
  if (value && !regex.test(value)) {
    return message || 'Invalid format';
  }
  return null;
});

export const email = pattern(
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  'Please enter a valid email address'
);

export const phone = pattern(
  /^\+?[\d\s\-\(\)]+$/,
  'Please enter a valid phone number'
);

// Composition validator
export const compose = (...validators) => (value) => {
  for (const validator of validators) {
    const error = validator(value);
    if (error) return error;
  }
  return null;
};

// Field validation builder
export const field = (fieldName) => ({
  required: (message) => [fieldName, required(message)],
  minLength: (min, message) => [fieldName, minLength(min, message)],
  maxLength: (max, message) => [fieldName, maxLength(max, message)],
  pattern: (regex, message) => [fieldName, pattern(regex, message)],
  email: () => [fieldName, email],
  phone: () => [fieldName, phone],
  custom: (validator) => [fieldName, validator],
  compose: (...validators) => [fieldName, compose(...validators)]
});

// Form validation schema builder
export const schema = (...fieldValidations) => {
  const validationMap = new Map(fieldValidations);
  
  return (data) => {
    const errors = {};
    
    for (const [fieldName, validator] of validationMap) {
      const error = validator(data[fieldName]);
      if (error) {
        errors[fieldName] = error;
      }
    }
    
    return Object.keys(errors).length > 0 ? errors : null;
  };
};

// Usage example
const userValidationSchema = schema(
  field('email').compose(
    required('Email is required'),
    email()
  ),
  field('password').compose(
    required('Password is required'),
    minLength(8, 'Password must be at least 8 characters')
  ),
  field('name').required('Name is required')
);
```

## 💾 Memoization Strategies

### **🧠 Advanced Memoization Techniques**

```javascript
// ============= ADVANCED MEMOIZATION =============
// utils/memoization.js

// Weak map based memoization for object keys
export const weakMemoize = (fn) => {
  const cache = new WeakMap();
  
  return (obj, ...args) => {
    if (!cache.has(obj)) {
      cache.set(obj, fn(obj, ...args));
    }
    return cache.get(obj);
  };
};

// LRU (Least Recently Used) memoization
export class LRUMemoize {
  constructor(fn, maxSize = 100) {
    this.fn = fn;
    this.maxSize = maxSize;
    this.cache = new Map();
    this.hits = 0;
    this.misses = 0;
  }

  get(key) {
    if (this.cache.has(key)) {
      // Move to end (most recently used)
      const value = this.cache.get(key);
      this.cache.delete(key);
      this.cache.set(key, value);
      this.hits++;
      return value;
    }

    // Compute and cache
    const value = this.fn(key);
    this.set(key, value);
    this.misses++;
    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove least recently used (first item)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(key, value);
  }

  clear() {
    this.cache.clear();
    this.hits = 0;
    this.misses = 0;
  }

  stats() {
    return {
      size: this.cache.size,
      hits: this.hits,
      misses: this.misses,
      hitRate: this.hits / (this.hits + this.misses) || 0
    };
  }
}

// Hierarchical memoization
export class HierarchicalMemoize {
  constructor() {
    this.cache = new Map();
    this.levels = [];
  }

  addLevel(keyExtractor, ttl = null) {
    this.levels.push({ keyExtractor, ttl });
    return this;
  }

  get(input) {
    let currentCache = this.cache;
    const path = [];

    // Navigate through hierarchy
    for (const { keyExtractor } of this.levels) {
      const key = keyExtractor(input);
      path.push(key);

      if (!currentCache.has(key)) {
        return null;
      }

      const entry = currentCache.get(key);
      
      // Check TTL
      if (entry.ttl && Date.now() > entry.expires) {
        currentCache.delete(key);
        return null;
      }

      if (entry.isValue) {
        return entry.value;
      }

      currentCache = entry.cache;
    }

    return null;
  }

  set(input, value) {
    let currentCache = this.cache;

    for (let i = 0; i < this.levels.length; i++) {
      const { keyExtractor, ttl } = this.levels[i];
      const key = keyExtractor(input);
      const isLastLevel = i === this.levels.length - 1;

      if (!currentCache.has(key)) {
        if (isLastLevel) {
          currentCache.set(key, {
            isValue: true,
            value,
            expires: ttl ? Date.now() + ttl : null
          });
        } else {
          currentCache.set(key, {
            isValue: false,
            cache: new Map(),
            expires: ttl ? Date.now() + ttl : null
          });
        }
      }

      if (isLastLevel) {
        currentCache.get(key).value = value;
        break;
      }

      currentCache = currentCache.get(key).cache;
    }
  }

  clear() {
    this.cache.clear();
  }
}

// ============= SPECIALIZED MEMOIZATION =============
// React component memoization
export const memoizeComponent = (Component, propsComparer = null) => {
  const cache = new Map();

  return (props) => {
    const key = propsComparer ? 
      propsComparer(props) : 
      JSON.stringify(props);

    if (!cache.has(key)) {
      cache.set(key, Component(props));
    }

    return cache.get(key);
  };
};

// Async function memoization
export const memoizeAsync = (asyncFn, keyGen = (...args) => JSON.stringify(args)) => {
  const cache = new Map();
  const pending = new Map();

  return async (...args) => {
    const key = keyGen(...args);

    // Return cached result
    if (cache.has(key)) {
      return cache.get(key);
    }

    // Return pending promise
    if (pending.has(key)) {
      return pending.get(key);
    }

    // Create new promise
    const promise = asyncFn(...args);
    pending.set(key, promise);

    try {
      const result = await promise;
      cache.set(key, result);
      pending.delete(key);
      return result;
    } catch (error) {
      pending.delete(key);
      throw error;
    }
  };
};

// Conditional memoization
export const memoizeIf = (fn, condition, keyGen) => {
  const memoized = memoize(fn, keyGen);
  
  return (...args) => {
    if (condition(...args)) {
      return memoized(...args);
    }
    return fn(...args);
  };
};

// ============= MEMOIZATION IN NEXT.JS =============
// lib/cache/api-memoization.js
import { memoizeWithTTL, memoizeAsync } from '@/utils/memoization';

// API response caching
export const createApiCache = (ttl = 300000) => { // 5 minutes default
  const cache = new Map();
  
  return {
    async get(url, options = {}) {
      const key = `${url}:${JSON.stringify(options)}`;
      
      if (cache.has(key)) {
        const { data, timestamp } = cache.get(key);
        if (Date.now() - timestamp < ttl) {
          return data;
        }
        cache.delete(key);
      }
      
      const response = await fetch(url, options);
      const data = await response.json();
      
      cache.set(key, {
        data,
        timestamp: Date.now()
      });
      
      return data;
    },
    
    clear() {
      cache.clear();
    },
    
    delete(url, options = {}) {
      const key = `${url}:${JSON.stringify(options)}`;
      cache.delete(key);
    }
  };
};

// Database query memoization
export const memoizeQuery = (queryFn, options = {}) => {
  const { 
    ttl = 60000,
    keyExtractor = (...args) => JSON.stringify(args),
    invalidateOn = []
  } = options;
  
  const cache = new Map();
  
  const memoized = async (...args) => {
    const key = keyExtractor(...args);
    
    if (cache.has(key)) {
      const { result, timestamp } = cache.get(key);
      if (Date.now() - timestamp < ttl) {
        return result;
      }
      cache.delete(key);
    }
    
    const result = await queryFn(...args);
    cache.set(key, {
      result,
      timestamp: Date.now()
    });
    
    return result;
  };
  
  // Add invalidation methods
  memoized.invalidate = (pattern = null) => {
    if (!pattern) {
      cache.clear();
      return;
    }
    
    for (const key of cache.keys()) {
      if (pattern.test(key)) {
        cache.delete(key);
      }
    }
  };
  
  memoized.invalidateKey = (...args) => {
    const key = keyExtractor(...args);
    cache.delete(key);
  };
  
  return memoized;
};

// Component computation memoization
export const useMemoizedComputation = (computeFn, deps, options = {}) => {
  const { 
    ttl = 0,
    keyExtractor = (deps) => JSON.stringify(deps)
  } = options;
  
  const cache = useRef(new Map());
  
  return useMemo(() => {
    const key = keyExtractor(deps);
    
    if (cache.current.has(key)) {
      const { result, timestamp } = cache.current.get(key);
      if (ttl === 0 || Date.now() - timestamp < ttl) {
        return result;
      }
      cache.current.delete(key);
    }
    
    const result = computeFn(...deps);
    cache.current.set(key, {
      result,
      timestamp: Date.now()
    });
    
    return result;
  }, deps);
};

// ============= PERFORMANCE MONITORING =============
// utils/memoization-monitor.js
export class MemoizationMonitor {
  constructor() {
    this.functions = new Map();
  }

  register(name, memoizedFn) {
    this.functions.set(name, {
      fn: memoizedFn,
      startTime: Date.now(),
      callCount: 0,
      totalTime: 0
    });

    // Wrap function to track performance
    const originalFn = memoizedFn;
    const monitor = this;

    return function(...args) {
      const start = performance.now();
      monitor.functions.get(name).callCount++;
      
      const result = originalFn.apply(this, args);
      
      const end = performance.now();
      monitor.functions.get(name).totalTime += (end - start);
      
      return result;
    };
  }

  getStats(name) {
    const fn = this.functions.get(name);
    if (!fn) return null;

    return {
      name,
      callCount: fn.callCount,
      totalTime: fn.totalTime,
      averageTime: fn.callCount > 0 ? fn.totalTime / fn.callCount : 0,
      hits: fn.fn.hits || 0,
      misses: fn.fn.misses || 0,
      hitRate: fn.fn.hits && fn.fn.misses ? 
        fn.fn.hits / (fn.fn.hits + fn.fn.misses) : 0
    };
  }

  getAllStats() {
    return Array.from(this.functions.keys()).map(name => this.getStats(name));
  }

  reset(name = null) {
    if (name) {
      const fn = this.functions.get(name);
      if (fn) {
        fn.callCount = 0;
        fn.totalTime = 0;
        if (fn.fn.clear) fn.fn.clear();
      }
    } else {
      for (const [name, fn] of this.functions) {
        fn.callCount = 0;
        fn.totalTime = 0;
        if (fn.fn.clear) fn.fn.clear();
      }
    }
  }
}

// Global monitor instance
export const memoMonitor = new MemoizationMonitor();
```

## 🚀 Next.js Integration

### **⚛️ Advanced Function Patterns in React**

```jsx
// ============= FUNCTION COMPOSITION IN COMPONENTS =============
// components/advanced/DataProcessor.jsx
import { useState, useEffect, useCallback } from 'react';
import { createUserProcessingPipeline } from '@/lib/data-processing/pipeline';
import { useFormHandlers } from '@/hooks/useFormHandlers';

const DataProcessor = () => {
  const [data, setData] = useState('');
  const [result, setResult] = useState(null);
  const [processing, setProcessing] = useState(false);
  const [error, setError] = useState(null);
  
  const { handleFieldChange } = useFormHandlers();
  
  // Create processing pipeline
  const pipeline = useCallback(() => {
    return createUserProcessingPipeline();
  }, []);
  
  const processData = useCallback(async () => {
    if (!data.trim()) return;
    
    setProcessing(true);
    setError(null);
    
    try {
      const parsedData = JSON.parse(data);
      const processor = pipeline();
      const { result, context } = await processor.execute(parsedData);
      
      setResult({
        data: result,
        steps: context.steps,
        metadata: context.metadata
      });
    } catch (err) {
      setError(err.message);
    } finally {
      setProcessing(false);
    }
  }, [data, pipeline]);
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h2 className="text-2xl font-bold mb-6">Data Processing Pipeline</h2>
      
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        {/* Input Section */}
        <div>
          <label className="block text-sm font-medium mb-2">
            Input Data (JSON)
          </label>
          <textarea
            value={data}
            onChange={(e) => handleFieldChange('data', e.target.value, setData)}
            className="w-full h-64 p-3 border rounded-lg font-mono text-sm"
            placeholder="Enter user data JSON..."
          />
          
          <button
            onClick={processData}
            disabled={processing || !data.trim()}
            className="mt-4 px-4 py-2 bg-blue-600 text-white rounded-lg disabled:opacity-50"
          >
            {processing ? 'Processing...' : 'Process Data'}
          </button>
        </div>
        
        {/* Output Section */}
        <div>
          <h3 className="text-lg font-semibold mb-2">Result</h3>
          
          {error && (
            <div className="p-3 bg-red-100 border border-red-300 rounded-lg mb-4">
              <p className="text-red-800">{error}</p>
            </div>
          )}
          
          {result && (
            <div className="space-y-4">
              {/* Processed Data */}
              <div>
                <h4 className="font-medium mb-2">Processed Data:</h4>
                <pre className="p-3 bg-gray-100 rounded-lg text-sm overflow-auto">
                  {JSON.stringify(result.data, null, 2)}
                </pre>
              </div>
              
              {/* Pipeline Steps */}
              <div>
                <h4 className="font-medium mb-2">Pipeline Steps:</h4>
                <div className="space-y-2">
                  {result.steps.map((step, index) => (
                    <div
                      key={index}
                      className={`p-2 rounded-lg text-sm ${
                        step.error
                          ? 'bg-red-100 text-red-800'
                          : step.skipped
                          ? 'bg-yellow-100 text-yellow-800'
                          : 'bg-green-100 text-green-800'
                      }`}
                    >
                      <div className="font-medium">{step.name}</div>
                      {step.attempts > 1 && (
                        <div className="text-xs">Attempts: {step.attempts}</div>
                      )}
                      {step.error && (
                        <div className="text-xs mt-1">
                          {step.errorHandled ? 'Error handled' : 'Error occurred'}
                        </div>
                      )}
                    </div>
                  ))}
                </div>
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

export default DataProcessor;

// ============= HIGHER-ORDER COMPONENT PATTERNS =============
// components/hoc/withMemoization.jsx
import { memo, useMemo } from 'react';
import { memoize } from '@/utils/memoization';

export const withMemoization = (Component, options = {}) => {
  const {
    propsComparer = null,
    computationMemoizer = memoize,
    displayName = `WithMemoization(${Component.displayName || Component.name})`
  } = options;
  
  const MemoizedComponent = memo(Component, propsComparer);
  MemoizedComponent.displayName = displayName;
  
  return (props) => {
    // Memoize expensive computations
    const memoizedProps = useMemo(() => {
      if (props.computeExpensive) {
        const memoizedCompute = computationMemoizer(props.computeExpensive);
        return {
          ...props,
          computeExpensive: memoizedCompute
        };
      }
      return props;
    }, [props]);
    
    return <MemoizedComponent {...memoizedProps} />;
  };
};

// components/hoc/withFunctionComposition.jsx
export const withFunctionComposition = (Component, processors = []) => {
  return (props) => {
    const processedProps = useMemo(() => {
      return processors.reduce((acc, processor) => processor(acc), props);
    }, [props]);
    
    return <Component {...processedProps} />;
  };
};

// ============= ADVANCED HOOKS =============
// hooks/useAdvancedMemo.js
import { useRef, useMemo } from 'react';
import { LRUMemoize, HierarchicalMemoize } from '@/utils/memoization';

export const useLRUMemo = (computeFn, deps, maxSize = 10) => {
  const memoizer = useRef(new LRUMemoize(computeFn, maxSize));
  
  return useMemo(() => {
    const key = JSON.stringify(deps);
    return memoizer.current.get(key);
  }, deps);
};

export const useHierarchicalMemo = (computeFn, levels, deps) => {
  const memoizer = useRef(new HierarchicalMemoize());
  
  // Setup levels
  useMemo(() => {
    memoizer.current = new HierarchicalMemoize();
    levels.forEach(level => memoizer.current.addLevel(level.keyExtractor, level.ttl));
  }, [levels]);
  
  return useMemo(() => {
    const input = { deps, computeFn };
    let result = memoizer.current.get(input);
    
    if (result === null) {
      result = computeFn(...deps);
      memoizer.current.set(input, result);
    }
    
    return result;
  }, deps);
};

// hooks/useCurriedActions.js
export const useCurriedActions = (actions) => {
  return useMemo(() => {
    const curriedActions = {};
    
    for (const [name, action] of Object.entries(actions)) {
      curriedActions[name] = curry(action);
    }
    
    return curriedActions;
  }, [actions]);
};

// ============= FUNCTIONAL COMPONENT LIBRARY =============
// components/functional/FunctionalForm.jsx
import { useState } from 'react';
import { compose } from '@/utils/functional';
import { field, schema } from '@/lib/validation/curried-validators';

const FunctionalForm = ({ onSubmit, validationSchema, initialData = {} }) => {
  const [formData, setFormData] = useState(initialData);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  // Create validation function
  const validate = useMemo(() => {
    return validationSchema || schema();
  }, [validationSchema]);
  
  // Curried field handlers
  const updateField = useCallback(
    curry((field, value) => {
      setFormData(prev => ({ ...prev, [field]: value }));
      
      // Clear field error
      if (errors[field]) {
        setErrors(prev => {
          const { [field]: _, ...rest } = prev;
          return rest;
        });
      }
    }),
    [errors]
  );
  
  const handleSubmit = useCallback(async (e) => {
    e.preventDefault();
    
    // Validate form
    const validationErrors = validate(formData);
    if (validationErrors) {
      setErrors(validationErrors);
      return;
    }
    
    setIsSubmitting(true);
    try {
      await onSubmit(formData);
    } catch (error) {
      setErrors({ _form: error.message });
    } finally {
      setIsSubmitting(false);
    }
  }, [formData, validate, onSubmit]);
  
  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      {errors._form && (
        <div className="p-3 bg-red-100 border border-red-300 rounded-lg">
          <p className="text-red-800">{errors._form}</p>
        </div>
      )}
      
      <div>
        <label className="block text-sm font-medium mb-1">
          Email
        </label>
        <input
          type="email"
          value={formData.email || ''}
          onChange={(e) => updateField('email')(e.target.value)}
          className={`w-full p-2 border rounded-lg ${
            errors.email ? 'border-red-300' : 'border-gray-300'
          }`}
        />
        {errors.email && (
          <p className="text-red-600 text-sm mt-1">{errors.email}</p>
        )}
      </div>
      
      <div>
        <label className="block text-sm font-medium mb-1">
          Password
        </label>
        <input
          type="password"
          value={formData.password || ''}
          onChange={(e) => updateField('password')(e.target.value)}
          className={`w-full p-2 border rounded-lg ${
            errors.password ? 'border-red-300' : 'border-gray-300'
          }`}
        />
        {errors.password && (
          <p className="text-red-600 text-sm mt-1">{errors.password}</p>
        )}
      </div>
      
      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full py-2 px-4 bg-blue-600 text-white rounded-lg disabled:opacity-50"
      >
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
};

export default FunctionalForm;
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Function Composition**
สร้าง advanced function composition system:

```javascript
// TODO: สร้าง function composition

// 1. Pipeline builder
class PipelineBuilder {
  // Add conditional steps
  // Parallel execution
  // Error recovery
}

// 2. Function orchestrator
class FunctionOrchestrator {
  // Dependency management
  // Resource allocation
  // Performance monitoring
}

// 3. Reactive composition
class ReactiveComposition {
  // Observable patterns
  // Event-driven execution
  // State management
}
```

### **🧮 แบบฝึกหัดที่ 2: Advanced Currying**
สร้าง sophisticated currying patterns:

```javascript
// TODO: สร้าง advanced currying

// 1. Type-safe currying
class TypeSafeCurry {
  // Runtime type checking
  // Generic type inference
  // Validation integration
}

// 2. Performance-optimized currying
class OptimizedCurry {
  // Lazy evaluation
  // Memory optimization
  // Caching strategies
}

// 3. Domain-specific language
class DSLBuilder {
  // Fluent interfaces
  // Method chaining
  // Expression building
}
```

### **💾 แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง comprehensive function integration:

```jsx
// TODO: สร้าง Next.js integration

// 1. Server-side function composition
const ServerComposition = () => {
  // SSR optimization
  // Edge function patterns
  // API composition
};

// 2. Client-side optimization
const ClientOptimization = () => {
  // Bundle splitting
  // Lazy loading
  // Progressive enhancement
};

// 3. Full-stack patterns
const FullStackPatterns = () => {
  // Shared utilities
  // Type safety
  // Performance monitoring
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 16: Testing Strategies](./16-testing.md)

**บทถัดไป**: [บทที่ 18: Metaprogramming](./18-metaprogramming.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Functional Programming Concepts](https://en.wikipedia.org/wiki/Functional_programming)
- [Function Composition](https://en.wikipedia.org/wiki/Function_composition)
- [Currying](https://en.wikipedia.org/wiki/Currying)
- [Memoization](https://en.wikipedia.org/wiki/Memoization)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Higher-Order Functions** - composition patterns, pipeline builders, และ advanced utilities
- **Currying and Partial Application** - auto-currying, placeholder patterns, และ practical applications
- **Memoization Strategies** - LRU caching, hierarchical memoization, และ performance monitoring
- **Next.js Integration** - React patterns, functional components, และ full-stack optimization
- **Advanced Patterns** - error handling, async composition, และ domain-specific languages

Advanced function techniques เป็นเครื่องมือที่ทรงพลังสำหรับการสร้าง maintainable และ efficient code! 🚀
