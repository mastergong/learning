# บทที่ 18: Metaprogramming

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ metaprogramming concepts และ techniques
- ทำความเข้าใจ Proxy, Reflect, และ dynamic code generation
- เรียนรู้ decorators, annotations, และ code transformation
- ประยุกต์ใช้ใน Next.js สำหรับ advanced development patterns

## 🪞 Proxy และ Reflect

### **🔍 Proxy Fundamentals**

```javascript
// ============= BASIC PROXY PATTERNS =============
// utils/proxy-patterns.js

// Property access logging
export const createLoggingProxy = (target, loggerFn = console.log) => {
  return new Proxy(target, {
    get(target, property, receiver) {
      loggerFn(`GET: ${String(property)}`);
      return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
      loggerFn(`SET: ${String(property)} = ${value}`);
      return Reflect.set(target, property, value, receiver);
    },
    
    has(target, property) {
      loggerFn(`HAS: ${String(property)}`);
      return Reflect.has(target, property);
    },
    
    deleteProperty(target, property) {
      loggerFn(`DELETE: ${String(property)}`);
      return Reflect.deleteProperty(target, property);
    }
  });
};

// Property validation proxy
export const createValidationProxy = (target, validators = {}) => {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      const validator = validators[property];
      
      if (validator && !validator(value)) {
        throw new Error(`Invalid value for property ${String(property)}: ${value}`);
      }
      
      return Reflect.set(target, property, value, receiver);
    }
  });
};

// Auto-populating proxy
export const createAutoPopulatingProxy = (factory) => {
  const cache = new Map();
  
  return new Proxy({}, {
    get(target, property, receiver) {
      if (!cache.has(property)) {
        cache.set(property, factory(property));
      }
      return cache.get(property);
    },
    
    has(target, property) {
      return true; // Always return true for auto-population
    },
    
    ownKeys(target) {
      return Array.from(cache.keys());
    },
    
    getOwnPropertyDescriptor(target, property) {
      return {
        enumerable: true,
        configurable: true,
        value: this.get(target, property)
      };
    }
  });
};

// ============= ADVANCED PROXY UTILITIES =============
// lib/metaprogramming/advanced-proxy.js

export class SmartProxy {
  constructor(target, options = {}) {
    this.target = target;
    this.options = {
      immutable: false,
      tracked: false,
      validation: {},
      transforms: {},
      hooks: {},
      ...options
    };
    
    this.history = [];
    this.listeners = new Set();
    
    return this.createProxy();
  }
  
  createProxy() {
    return new Proxy(this.target, {
      get: this.handleGet.bind(this),
      set: this.handleSet.bind(this),
      has: this.handleHas.bind(this),
      deleteProperty: this.handleDelete.bind(this),
      ownKeys: this.handleOwnKeys.bind(this),
      getOwnPropertyDescriptor: this.handleGetOwnPropertyDescriptor.bind(this)
    });
  }
  
  handleGet(target, property, receiver) {
    // Execute beforeGet hook
    this.executeHook('beforeGet', { target, property, receiver });
    
    let value = Reflect.get(target, property, receiver);
    
    // Apply transforms
    const transform = this.options.transforms[property];
    if (transform && typeof transform.get === 'function') {
      value = transform.get(value, property, target);
    }
    
    // Track access if needed
    if (this.options.tracked) {
      this.recordAccess('get', property, value);
    }
    
    // Execute afterGet hook
    this.executeHook('afterGet', { target, property, receiver, value });
    
    return value;
  }
  
  handleSet(target, property, value, receiver) {
    // Check immutability
    if (this.options.immutable) {
      throw new Error(`Cannot set property ${String(property)} on immutable object`);
    }
    
    // Execute beforeSet hook
    this.executeHook('beforeSet', { target, property, value, receiver });
    
    const oldValue = target[property];
    
    // Apply transforms
    const transform = this.options.transforms[property];
    if (transform && typeof transform.set === 'function') {
      value = transform.set(value, property, target);
    }
    
    // Validate value
    const validator = this.options.validation[property];
    if (validator && !validator(value, property, target)) {
      throw new Error(`Validation failed for property ${String(property)}`);
    }
    
    // Set value
    const result = Reflect.set(target, property, value, receiver);
    
    // Track change if needed
    if (this.options.tracked) {
      this.recordChange('set', property, oldValue, value);
    }
    
    // Notify listeners
    this.notifyListeners('set', property, oldValue, value);
    
    // Execute afterSet hook
    this.executeHook('afterSet', { target, property, value, receiver, oldValue });
    
    return result;
  }
  
  handleHas(target, property) {
    this.executeHook('beforeHas', { target, property });
    
    const result = Reflect.has(target, property);
    
    if (this.options.tracked) {
      this.recordAccess('has', property, result);
    }
    
    this.executeHook('afterHas', { target, property, result });
    
    return result;
  }
  
  handleDelete(target, property) {
    if (this.options.immutable) {
      throw new Error(`Cannot delete property ${String(property)} from immutable object`);
    }
    
    this.executeHook('beforeDelete', { target, property });
    
    const oldValue = target[property];
    const result = Reflect.deleteProperty(target, property);
    
    if (result && this.options.tracked) {
      this.recordChange('delete', property, oldValue, undefined);
    }
    
    this.notifyListeners('delete', property, oldValue, undefined);
    this.executeHook('afterDelete', { target, property, oldValue });
    
    return result;
  }
  
  handleOwnKeys(target) {
    this.executeHook('beforeOwnKeys', { target });
    
    const keys = Reflect.ownKeys(target);
    
    if (this.options.tracked) {
      this.recordAccess('ownKeys', null, keys);
    }
    
    this.executeHook('afterOwnKeys', { target, keys });
    
    return keys;
  }
  
  handleGetOwnPropertyDescriptor(target, property) {
    this.executeHook('beforeGetOwnPropertyDescriptor', { target, property });
    
    const descriptor = Reflect.getOwnPropertyDescriptor(target, property);
    
    if (this.options.tracked) {
      this.recordAccess('getOwnPropertyDescriptor', property, descriptor);
    }
    
    this.executeHook('afterGetOwnPropertyDescriptor', { target, property, descriptor });
    
    return descriptor;
  }
  
  executeHook(hookName, context) {
    const hook = this.options.hooks[hookName];
    if (typeof hook === 'function') {
      hook(context);
    }
  }
  
  recordAccess(operation, property, value) {
    this.history.push({
      type: 'access',
      operation,
      property,
      value,
      timestamp: Date.now()
    });
  }
  
  recordChange(operation, property, oldValue, newValue) {
    this.history.push({
      type: 'change',
      operation,
      property,
      oldValue,
      newValue,
      timestamp: Date.now()
    });
  }
  
  notifyListeners(operation, property, oldValue, newValue) {
    const event = { operation, property, oldValue, newValue, timestamp: Date.now() };
    this.listeners.forEach(listener => {
      try {
        listener(event);
      } catch (error) {
        console.error('Listener error:', error);
      }
    });
  }
  
  addListener(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
  
  getHistory() {
    return [...this.history];
  }
  
  clearHistory() {
    this.history.length = 0;
  }
}

// ============= PRACTICAL PROXY APPLICATIONS =============
// lib/data/reactive-model.js

export class ReactiveModel {
  constructor(initialData = {}, schema = {}) {
    this.data = { ...initialData };
    this.schema = schema;
    this.computed = new Map();
    this.watchers = new Map();
    this.errors = new Map();
    
    return this.createReactiveProxy();
  }
  
  createReactiveProxy() {
    return new Proxy(this, {
      get(target, property, receiver) {
        // Handle special methods
        if (property in target && typeof target[property] === 'function') {
          return target[property].bind(target);
        }
        
        // Handle computed properties
        if (target.computed.has(property)) {
          const computedFn = target.computed.get(property);
          return computedFn.call(target);
        }
        
        // Handle data properties
        if (property in target.data) {
          return target.data[property];
        }
        
        // Handle schema properties
        if (property in target.schema) {
          return target.schema[property].default;
        }
        
        return undefined;
      },
      
      set(target, property, value, receiver) {
        // Validate against schema
        if (target.schema[property]) {
          const validation = target.validateProperty(property, value);
          if (!validation.isValid) {
            target.errors.set(property, validation.errors);
            throw new Error(`Validation failed for ${property}: ${validation.errors.join(', ')}`);
          } else {
            target.errors.delete(property);
          }
        }
        
        const oldValue = target.data[property];
        target.data[property] = value;
        
        // Trigger watchers
        target.triggerWatchers(property, oldValue, value);
        
        // Invalidate dependent computed properties
        target.invalidateComputedProperties(property);
        
        return true;
      },
      
      has(target, property) {
        return property in target.data || 
               property in target.schema || 
               target.computed.has(property) ||
               property in target;
      },
      
      ownKeys(target) {
        return [
          ...Object.keys(target.data),
          ...Object.keys(target.schema),
          ...target.computed.keys()
        ];
      }
    });
  }
  
  // Define computed property
  computed(property, computeFn) {
    this.computed.set(property, computeFn);
    return this;
  }
  
  // Watch property changes
  watch(property, watcherFn) {
    if (!this.watchers.has(property)) {
      this.watchers.set(property, new Set());
    }
    this.watchers.get(property).add(watcherFn);
    
    return () => this.watchers.get(property).delete(watcherFn);
  }
  
  // Validate property
  validateProperty(property, value) {
    const fieldSchema = this.schema[property];
    if (!fieldSchema) return { isValid: true, errors: [] };
    
    const errors = [];
    
    // Required validation
    if (fieldSchema.required && (value === undefined || value === null || value === '')) {
      errors.push('This field is required');
    }
    
    // Type validation
    if (value !== undefined && fieldSchema.type) {
      if (typeof value !== fieldSchema.type) {
        errors.push(`Expected type ${fieldSchema.type}, got ${typeof value}`);
      }
    }
    
    // Custom validators
    if (fieldSchema.validators) {
      for (const validator of fieldSchema.validators) {
        const result = validator(value);
        if (typeof result === 'string') {
          errors.push(result);
        } else if (!result) {
          errors.push('Validation failed');
        }
      }
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
  
  // Trigger watchers
  triggerWatchers(property, oldValue, newValue) {
    const watchers = this.watchers.get(property);
    if (watchers) {
      watchers.forEach(watcher => {
        try {
          watcher(newValue, oldValue, property);
        } catch (error) {
          console.error('Watcher error:', error);
        }
      });
    }
  }
  
  // Invalidate computed properties that depend on this property
  invalidateComputedProperties(property) {
    // In a more sophisticated implementation, we would track dependencies
    // For now, we'll just clear all computed property caches
    this.computed.forEach((computeFn, computedProperty) => {
      // Force recomputation on next access
      if (computeFn.cache) {
        delete computeFn.cache;
      }
    });
  }
  
  // Get validation errors
  getErrors() {
    return Object.fromEntries(this.errors);
  }
  
  // Check if model is valid
  isValid() {
    return this.errors.size === 0;
  }
  
  // Serialize to plain object
  toJSON() {
    return { ...this.data };
  }
}

// ============= API PROXY PATTERNS =============
// lib/api/api-proxy.js

export class APIProxy {
  constructor(baseURL, options = {}) {
    this.baseURL = baseURL.replace(/\/$/, '');
    this.options = {
      timeout: 30000,
      retries: 3,
      cache: new Map(),
      ...options
    };
    
    return this.createAPIProxy();
  }
  
  createAPIProxy() {
    return new Proxy(this, {
      get(target, property, receiver) {
        // Handle existing methods
        if (property in target && typeof target[property] === 'function') {
          return target[property].bind(target);
        }
        
        // Handle special properties
        if (property === 'baseURL' || property === 'options') {
          return target[property];
        }
        
        // Create dynamic API endpoint
        return target.createEndpoint(property);
      }
    });
  }
  
  createEndpoint(endpoint) {
    const endpointProxy = new Proxy({}, {
      get: (target, method) => {
        if (typeof method === 'string' && method.toLowerCase() in ['get', 'post', 'put', 'delete', 'patch']) {
          return this.createMethodHandler(endpoint, method.toLowerCase());
        }
        
        // Handle nested endpoints
        return this.createEndpoint(`${endpoint}/${method}`);
      }
    });
    
    // Add direct call capability
    endpointProxy.get = this.createMethodHandler(endpoint, 'get');
    endpointProxy.post = this.createMethodHandler(endpoint, 'post');
    endpointProxy.put = this.createMethodHandler(endpoint, 'put');
    endpointProxy.delete = this.createMethodHandler(endpoint, 'delete');
    endpointProxy.patch = this.createMethodHandler(endpoint, 'patch');
    
    return endpointProxy;
  }
  
  createMethodHandler(endpoint, method) {
    return async (data = null, options = {}) => {
      const url = `${this.baseURL}/${endpoint}`;
      const cacheKey = `${method}:${url}:${JSON.stringify(data)}`;
      
      // Check cache for GET requests
      if (method === 'get' && this.options.cache.has(cacheKey)) {
        const cached = this.options.cache.get(cacheKey);
        if (Date.now() - cached.timestamp < (options.cacheTime || 300000)) {
          return cached.data;
        }
      }
      
      const requestOptions = {
        method: method.toUpperCase(),
        headers: {
          'Content-Type': 'application/json',
          ...this.options.headers,
          ...options.headers
        },
        ...options
      };
      
      if (data && method !== 'get') {
        requestOptions.body = JSON.stringify(data);
      }
      
      // Add timeout
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), this.options.timeout);
      requestOptions.signal = controller.signal;
      
      try {
        let response;
        let attempt = 0;
        
        while (attempt <= this.options.retries) {
          try {
            response = await fetch(url, requestOptions);
            break;
          } catch (error) {
            attempt++;
            if (attempt > this.options.retries) {
              throw error;
            }
            
            // Wait before retry
            await new Promise(resolve => 
              setTimeout(resolve, Math.pow(2, attempt - 1) * 1000)
            );
          }
        }
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const result = await response.json();
        
        // Cache GET requests
        if (method === 'get') {
          this.options.cache.set(cacheKey, {
            data: result,
            timestamp: Date.now()
          });
        }
        
        return result;
      } catch (error) {
        clearTimeout(timeoutId);
        throw error;
      }
    };
  }
  
  clearCache() {
    this.options.cache.clear();
  }
  
  setHeader(name, value) {
    if (!this.options.headers) {
      this.options.headers = {};
    }
    this.options.headers[name] = value;
  }
}

// Usage examples
const api = new APIProxy('https://api.example.com');

// These all work dynamically:
// await api.users.get()
// await api.users.post({ name: 'John' })
// await api.users(123).get()
// await api.posts.comments(456).put({ content: 'Updated' })
```

## 🎨 Decorators and Annotations

### **🎯 Decorator Patterns**

```javascript
// ============= DECORATOR UTILITIES =============
// utils/decorators.js

// Method decorator factory
export const createMethodDecorator = (decoratorFn) => {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args) {
      return decoratorFn.call(this, originalMethod, args, {
        target,
        propertyKey,
        instance: this
      });
    };
    
    return descriptor;
  };
};

// Class decorator factory
export const createClassDecorator = (decoratorFn) => {
  return function(constructor) {
    return decoratorFn(constructor);
  };
};

// Property decorator factory
export const createPropertyDecorator = (decoratorFn) => {
  return function(target, propertyKey) {
    return decoratorFn(target, propertyKey);
  };
};

// ============= COMMON DECORATORS =============

// Timing decorator
export const timed = createMethodDecorator((originalMethod, args, context) => {
  const start = performance.now();
  const result = originalMethod.apply(context.instance, args);
  
  if (result instanceof Promise) {
    return result.then(value => {
      const end = performance.now();
      console.log(`${context.propertyKey} took ${end - start} milliseconds`);
      return value;
    });
  } else {
    const end = performance.now();
    console.log(`${context.propertyKey} took ${end - start} milliseconds`);
    return result;
  }
});

// Memoization decorator
export const memoized = (keyGenerator = (...args) => JSON.stringify(args)) => {
  return createMethodDecorator((originalMethod, args, context) => {
    if (!context.instance._memoCache) {
      context.instance._memoCache = new Map();
    }
    
    const key = `${context.propertyKey}:${keyGenerator(...args)}`;
    
    if (context.instance._memoCache.has(key)) {
      return context.instance._memoCache.get(key);
    }
    
    const result = originalMethod.apply(context.instance, args);
    context.instance._memoCache.set(key, result);
    
    return result;
  });
};

// Validation decorator
export const validated = (...validators) => {
  return createMethodDecorator((originalMethod, args, context) => {
    for (let i = 0; i < validators.length; i++) {
      const validator = validators[i];
      const arg = args[i];
      
      if (validator && !validator(arg)) {
        throw new Error(`Validation failed for argument ${i} in ${context.propertyKey}`);
      }
    }
    
    return originalMethod.apply(context.instance, args);
  });
};

// Retry decorator
export const retry = (attempts = 3, delay = 1000) => {
  return createMethodDecorator(async (originalMethod, args, context) => {
    let lastError;
    
    for (let i = 0; i < attempts; i++) {
      try {
        return await originalMethod.apply(context.instance, args);
      } catch (error) {
        lastError = error;
        
        if (i === attempts - 1) {
          throw error;
        }
        
        await new Promise(resolve => setTimeout(resolve, delay * Math.pow(2, i)));
      }
    }
    
    throw lastError;
  });
};

// Rate limiting decorator
export const rateLimit = (maxCalls = 10, windowMs = 60000) => {
  const callHistory = new Map();
  
  return createMethodDecorator((originalMethod, args, context) => {
    const key = `${context.instance.constructor.name}.${context.propertyKey}`;
    const now = Date.now();
    
    if (!callHistory.has(key)) {
      callHistory.set(key, []);
    }
    
    const calls = callHistory.get(key);
    
    // Remove old calls outside the window
    while (calls.length > 0 && now - calls[0] > windowMs) {
      calls.shift();
    }
    
    if (calls.length >= maxCalls) {
      throw new Error(`Rate limit exceeded for ${key}`);
    }
    
    calls.push(now);
    return originalMethod.apply(context.instance, args);
  });
};

// Access control decorator
export const authorized = (...roles) => {
  return createMethodDecorator((originalMethod, args, context) => {
    const userRoles = context.instance.currentUser?.roles || [];
    
    if (roles.length > 0 && !roles.some(role => userRoles.includes(role))) {
      throw new Error(`Access denied. Required roles: ${roles.join(', ')}`);
    }
    
    return originalMethod.apply(context.instance, args);
  });
};

// Deprecated decorator
export const deprecated = (message = 'This method is deprecated') => {
  return createMethodDecorator((originalMethod, args, context) => {
    console.warn(`DEPRECATED: ${context.propertyKey} - ${message}`);
    return originalMethod.apply(context.instance, args);
  });
};

// ============= ADVANCED DECORATOR PATTERNS =============
// lib/metaprogramming/advanced-decorators.js

// Aspect-oriented programming decorator
export class AspectDecorator {
  constructor() {
    this.beforeAdvice = [];
    this.afterAdvice = [];
    this.aroundAdvice = [];
    this.errorAdvice = [];
  }
  
  before(advice) {
    this.beforeAdvice.push(advice);
    return this;
  }
  
  after(advice) {
    this.afterAdvice.push(advice);
    return this;
  }
  
  around(advice) {
    this.aroundAdvice.push(advice);
    return this;
  }
  
  onError(advice) {
    this.errorAdvice.push(advice);
    return this;
  }
  
  apply() {
    return createMethodDecorator((originalMethod, args, context) => {
      try {
        // Execute before advice
        for (const advice of this.beforeAdvice) {
          advice(args, context);
        }
        
        // Execute around advice or original method
        let result;
        if (this.aroundAdvice.length > 0) {
          let methodChain = originalMethod;
          
          for (const advice of this.aroundAdvice.reverse()) {
            const currentMethod = methodChain;
            methodChain = function(...args) {
              return advice.call(this, currentMethod, args, context);
            };
          }
          
          result = methodChain.apply(context.instance, args);
        } else {
          result = originalMethod.apply(context.instance, args);
        }
        
        // Handle async results
        if (result instanceof Promise) {
          return result
            .then(value => {
              // Execute after advice
              for (const advice of this.afterAdvice) {
                advice(value, args, context);
              }
              return value;
            })
            .catch(error => {
              // Execute error advice
              for (const advice of this.errorAdvice) {
                advice(error, args, context);
              }
              throw error;
            });
        } else {
          // Execute after advice
          for (const advice of this.afterAdvice) {
            advice(result, args, context);
          }
          return result;
        }
      } catch (error) {
        // Execute error advice
        for (const advice of this.errorAdvice) {
          advice(error, args, context);
        }
        throw error;
      }
    });
  }
}

// Composite decorator
export const composite = (...decorators) => {
  return function(target, propertyKey, descriptor) {
    return decorators.reduce((desc, decorator) => {
      return decorator(target, propertyKey, desc) || desc;
    }, descriptor);
  };
};

// Conditional decorator
export const when = (condition) => (decorator) => {
  return function(target, propertyKey, descriptor) {
    if (condition(target, propertyKey, descriptor)) {
      return decorator(target, propertyKey, descriptor);
    }
    return descriptor;
  };
};

// ============= METADATA MANAGEMENT =============
// lib/metaprogramming/metadata.js

export class MetadataManager {
  constructor() {
    this.metadata = new WeakMap();
  }
  
  define(target, key, value) {
    if (!this.metadata.has(target)) {
      this.metadata.set(target, new Map());
    }
    this.metadata.get(target).set(key, value);
  }
  
  get(target, key) {
    const targetMetadata = this.metadata.get(target);
    return targetMetadata ? targetMetadata.get(key) : undefined;
  }
  
  has(target, key) {
    const targetMetadata = this.metadata.get(target);
    return targetMetadata ? targetMetadata.has(key) : false;
  }
  
  delete(target, key) {
    const targetMetadata = this.metadata.get(target);
    return targetMetadata ? targetMetadata.delete(key) : false;
  }
  
  getAll(target) {
    const targetMetadata = this.metadata.get(target);
    return targetMetadata ? Object.fromEntries(targetMetadata) : {};
  }
  
  inherit(child, parent) {
    const parentMetadata = this.metadata.get(parent);
    if (parentMetadata) {
      if (!this.metadata.has(child)) {
        this.metadata.set(child, new Map());
      }
      const childMetadata = this.metadata.get(child);
      
      for (const [key, value] of parentMetadata) {
        if (!childMetadata.has(key)) {
          childMetadata.set(key, value);
        }
      }
    }
  }
}

// Global metadata manager
export const metadata = new MetadataManager();

// Metadata decorators
export const meta = (key, value) => {
  return function(target, propertyKey, descriptor) {
    if (propertyKey) {
      // Method metadata
      metadata.define(descriptor.value, key, value);
    } else {
      // Class metadata
      metadata.define(target, key, value);
    }
    return descriptor;
  };
};

export const getMeta = (target, key) => metadata.get(target, key);
export const hasMeta = (target, key) => metadata.has(target, key);

// ============= PRACTICAL DECORATOR EXAMPLES =============
// lib/services/decorated-service.js

class UserService {
  constructor() {
    this.users = new Map();
    this.currentUser = null;
  }
  
  @timed
  @memoized((id) => `user:${id}`)
  @validated((id) => typeof id === 'string' && id.length > 0)
  async getUser(id) {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 100));
    
    if (!this.users.has(id)) {
      throw new Error(`User ${id} not found`);
    }
    
    return this.users.get(id);
  }
  
  @authorized('admin', 'moderator')
  @retry(3, 1000)
  @rateLimit(5, 60000)
  async createUser(userData) {
    const id = `user_${Date.now()}`;
    const user = {
      id,
      ...userData,
      createdAt: new Date().toISOString()
    };
    
    this.users.set(id, user);
    return user;
  }
  
  @deprecated('Use getUserList instead')
  @timed
  getAllUsers() {
    return Array.from(this.users.values());
  }
  
  @meta('description', 'Returns paginated user list')
  @meta('version', '2.0')
  async getUserList(page = 1, limit = 10) {
    const users = Array.from(this.users.values());
    const start = (page - 1) * limit;
    const end = start + limit;
    
    return {
      data: users.slice(start, end),
      meta: {
        page,
        limit,
        total: users.length,
        totalPages: Math.ceil(users.length / limit)
      }
    };
  }
}

// Advanced aspect example
const loggingAspect = new AspectDecorator()
  .before((args, context) => {
    console.log(`Calling ${context.propertyKey} with args:`, args);
  })
  .after((result, args, context) => {
    console.log(`${context.propertyKey} returned:`, result);
  })
  .onError((error, args, context) => {
    console.error(`${context.propertyKey} threw error:`, error);
  });

class LoggedService {
  @loggingAspect.apply()
  @timed
  async processData(data) {
    // Process data
    await new Promise(resolve => setTimeout(resolve, 500));
    return { processed: true, data };
  }
}
```

## 🔧 Dynamic Code Generation

### **🚀 Code Generation Patterns**

```javascript
// ============= CODE GENERATION UTILITIES =============
// lib/metaprogramming/code-generation.js

export class CodeGenerator {
  constructor() {
    this.templates = new Map();
    this.functions = new Map();
  }
  
  // Register template
  template(name, templateStr) {
    this.templates.set(name, templateStr);
    return this;
  }
  
  // Generate function from template
  generate(templateName, variables = {}) {
    const template = this.templates.get(templateName);
    if (!template) {
      throw new Error(`Template ${templateName} not found`);
    }
    
    let code = template;
    
    // Replace variables
    for (const [key, value] of Object.entries(variables)) {
      const regex = new RegExp(`\\{\\{${key}\\}\\}`, 'g');
      code = code.replace(regex, JSON.stringify(value));
    }
    
    // Generate function
    try {
      const fn = new Function('return (' + code + ')')();
      this.functions.set(`${templateName}_${Date.now()}`, fn);
      return fn;
    } catch (error) {
      throw new Error(`Failed to generate function: ${error.message}`);
    }
  }
  
  // Create factory function
  factory(templateName) {
    return (variables) => this.generate(templateName, variables);
  }
  
  // Create class from template
  createClass(className, methods = {}, properties = {}) {
    const methodCode = Object.entries(methods)
      .map(([name, fn]) => `${name}() { return (${fn.toString()}).call(this); }`)
      .join('\n  ');
    
    const classCode = `
      class ${className} {
        constructor(props = {}) {
          Object.assign(this, ${JSON.stringify(properties)}, props);
        }
        
        ${methodCode}
      }
      ${className}
    `;
    
    return new Function('return (' + classCode + ')')();
  }
  
  // Create optimized function
  optimize(fn, optimizations = []) {
    let code = fn.toString();
    
    for (const optimization of optimizations) {
      code = optimization(code);
    }
    
    return new Function('return (' + code + ')')();
  }
}

// ============= DYNAMIC API GENERATION =============
// lib/api/dynamic-api.js

export class DynamicAPIGenerator {
  constructor(schema) {
    this.schema = schema;
    this.endpoints = new Map();
  }
  
  generateCRUDAPI(entityName, entitySchema) {
    const API = {
      // Create
      [`create${entityName}`]: this.generateCreateMethod(entityName, entitySchema),
      
      // Read
      [`get${entityName}`]: this.generateGetMethod(entityName, entitySchema),
      [`list${entityName}s`]: this.generateListMethod(entityName, entitySchema),
      
      // Update
      [`update${entityName}`]: this.generateUpdateMethod(entityName, entitySchema),
      
      // Delete
      [`delete${entityName}`]: this.generateDeleteMethod(entityName, entitySchema)
    };
    
    return API;
  }
  
  generateCreateMethod(entityName, schema) {
    const validationCode = this.generateValidation(schema);
    
    return new Function('data', `
      // Validate input
      ${validationCode}
      
      // Create entity
      const entity = {
        id: generateId(),
        ...data,
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString()
      };
      
      // Save to storage
      return this.storage.save('${entityName.toLowerCase()}', entity);
    `);
  }
  
  generateGetMethod(entityName, schema) {
    return new Function('id', `
      if (!id) throw new Error('ID is required');
      return this.storage.findById('${entityName.toLowerCase()}', id);
    `);
  }
  
  generateListMethod(entityName, schema) {
    return new Function('options = {}', `
      const { 
        page = 1, 
        limit = 10, 
        sort = 'createdAt', 
        order = 'desc',
        filter = {}
      } = options;
      
      return this.storage.findMany('${entityName.toLowerCase()}', {
        page,
        limit,
        sort: { [sort]: order },
        filter
      });
    `);
  }
  
  generateUpdateMethod(entityName, schema) {
    const validationCode = this.generateValidation(schema, true);
    
    return new Function('id', 'data', `
      if (!id) throw new Error('ID is required');
      
      // Validate updates
      ${validationCode}
      
      // Update entity
      const updates = {
        ...data,
        updatedAt: new Date().toISOString()
      };
      
      return this.storage.update('${entityName.toLowerCase()}', id, updates);
    `);
  }
  
  generateDeleteMethod(entityName, schema) {
    return new Function('id', `
      if (!id) throw new Error('ID is required');
      return this.storage.delete('${entityName.toLowerCase()}', id);
    `);
  }
  
  generateValidation(schema, isUpdate = false) {
    const validations = [];
    
    for (const [field, rules] of Object.entries(schema)) {
      if (!isUpdate && rules.required) {
        validations.push(`
          if (!data.${field}) {
            throw new Error('${field} is required');
          }
        `);
      }
      
      if (rules.type) {
        validations.push(`
          if (data.${field} !== undefined && typeof data.${field} !== '${rules.type}') {
            throw new Error('${field} must be of type ${rules.type}');
          }
        `);
      }
      
      if (rules.minLength) {
        validations.push(`
          if (data.${field} && data.${field}.length < ${rules.minLength}) {
            throw new Error('${field} must be at least ${rules.minLength} characters');
          }
        `);
      }
      
      if (rules.maxLength) {
        validations.push(`
          if (data.${field} && data.${field}.length > ${rules.maxLength}) {
            throw new Error('${field} must be no more than ${rules.maxLength} characters');
          }
        `);
      }
    }
    
    return validations.join('\n');
  }
}

// ============= QUERY BUILDER GENERATION =============
// lib/database/query-builder.js

export class DynamicQueryBuilder {
  constructor(tableName) {
    this.tableName = tableName;
    this.query = {
      type: null,
      select: [],
      where: [],
      joins: [],
      orderBy: [],
      groupBy: [],
      having: [],
      limit: null,
      offset: null
    };
  }
  
  select(...fields) {
    this.query.type = 'SELECT';
    this.query.select.push(...fields);
    return this;
  }
  
  where(field, operator, value) {
    this.query.where.push({ field, operator, value });
    return this;
  }
  
  join(table, on) {
    this.query.joins.push({ type: 'INNER', table, on });
    return this;
  }
  
  leftJoin(table, on) {
    this.query.joins.push({ type: 'LEFT', table, on });
    return this;
  }
  
  orderBy(field, direction = 'ASC') {
    this.query.orderBy.push({ field, direction });
    return this;
  }
  
  limit(count) {
    this.query.limit = count;
    return this;
  }
  
  offset(count) {
    this.query.offset = count;
    return this;
  }
  
  // Generate SQL
  toSQL() {
    const parts = [];
    
    // SELECT clause
    if (this.query.select.length > 0) {
      parts.push(`SELECT ${this.query.select.join(', ')}`);
    } else {
      parts.push('SELECT *');
    }
    
    // FROM clause
    parts.push(`FROM ${this.tableName}`);
    
    // JOIN clauses
    for (const join of this.query.joins) {
      parts.push(`${join.type} JOIN ${join.table} ON ${join.on}`);
    }
    
    // WHERE clause
    if (this.query.where.length > 0) {
      const conditions = this.query.where.map(w => 
        `${w.field} ${w.operator} ${this.formatValue(w.value)}`
      ).join(' AND ');
      parts.push(`WHERE ${conditions}`);
    }
    
    // ORDER BY clause
    if (this.query.orderBy.length > 0) {
      const orderFields = this.query.orderBy.map(o => 
        `${o.field} ${o.direction}`
      ).join(', ');
      parts.push(`ORDER BY ${orderFields}`);
    }
    
    // LIMIT clause
    if (this.query.limit !== null) {
      parts.push(`LIMIT ${this.query.limit}`);
    }
    
    // OFFSET clause
    if (this.query.offset !== null) {
      parts.push(`OFFSET ${this.query.offset}`);
    }
    
    return parts.join(' ');
  }
  
  formatValue(value) {
    if (typeof value === 'string') {
      return `'${value.replace(/'/g, "''")}'`;
    }
    return value;
  }
  
  // Generate function to execute query
  toFunction() {
    const sql = this.toSQL();
    
    return new Function('database', `
      return database.query(${JSON.stringify(sql)});
    `);
  }
}

// ============= NEXT.JS INTEGRATION =============
// lib/metaprogramming/next-integration.js

export class NextJSMetaprogramming {
  // Generate API routes
  static generateAPIRoute(schema) {
    const handlerCode = `
      export default async function handler(req, res) {
        const { method } = req;
        
        switch (method) {
          case 'GET':
            return handleGet(req, res);
          case 'POST':
            return handlePost(req, res);
          case 'PUT':
            return handlePut(req, res);
          case 'DELETE':
            return handleDelete(req, res);
          default:
            res.setHeader('Allow', ['GET', 'POST', 'PUT', 'DELETE']);
            res.status(405).end(\`Method \${method} Not Allowed\`);
        }
      }
      
      async function handleGet(req, res) {
        // Generated GET handler
        ${schema.handlers?.get || 'res.status(200).json({ message: "GET handler" })'}
      }
      
      async function handlePost(req, res) {
        // Generated POST handler
        ${schema.handlers?.post || 'res.status(200).json({ message: "POST handler" })'}
      }
      
      async function handlePut(req, res) {
        // Generated PUT handler
        ${schema.handlers?.put || 'res.status(200).json({ message: "PUT handler" })'}
      }
      
      async function handleDelete(req, res) {
        // Generated DELETE handler
        ${schema.handlers?.delete || 'res.status(200).json({ message: "DELETE handler" })'}
      }
    `;
    
    return handlerCode;
  }
  
  // Generate React components
  static generateComponent(componentSchema) {
    const {
      name,
      props = [],
      state = [],
      methods = [],
      lifecycle = {},
      hooks = []
    } = componentSchema;
    
    const propsDestructure = props.length > 0 
      ? `{ ${props.join(', ')} }`
      : 'props';
    
    const stateDeclarations = state.map(s => 
      `const [${s.name}, set${s.name.charAt(0).toUpperCase() + s.name.slice(1)}] = useState(${JSON.stringify(s.initial)});`
    ).join('\n  ');
    
    const hookDeclarations = hooks.map(h => h.code).join('\n  ');
    
    const methodDeclarations = methods.map(m => `
      const ${m.name} = ${m.async ? 'async ' : ''}(${m.params?.join(', ') || ''}) => {
        ${m.body}
      };
    `).join('\n');
    
    const componentCode = `
      import React, { useState, useEffect } from 'react';
      
      const ${name} = (${propsDestructure}) => {
        ${stateDeclarations}
        ${hookDeclarations}
        
        ${methodDeclarations}
        
        ${lifecycle.useEffect ? `
          useEffect(() => {
            ${lifecycle.useEffect.body}
          }, ${JSON.stringify(lifecycle.useEffect.deps || [])});
        ` : ''}
        
        return (
          ${componentSchema.render || '<div>Generated Component</div>'}
        );
      };
      
      export default ${name};
    `;
    
    return componentCode;
  }
  
  // Generate type definitions
  static generateTypes(schema) {
    const interfaces = Object.entries(schema.entities || {}).map(([name, entity]) => {
      const properties = Object.entries(entity.properties || {}).map(([prop, type]) => {
        const optional = type.required === false ? '?' : '';
        return `  ${prop}${optional}: ${type.type};`;
      }).join('\n');
      
      return `
        export interface ${name} {
        ${properties}
        }
      `;
    }).join('\n');
    
    return interfaces;
  }
}
```

## 🎯 แบบฝึกหัด

### **🧪 แบบฝึกหัดที่ 1: Proxy Patterns**
สร้าง advanced proxy system:

```javascript
// TODO: สร้าง proxy system

// 1. Reactive data store
class ReactiveStore {
  // Deep reactivity
  // Computed properties
  // Change tracking
}

// 2. API mock system
class APIMockProxy {
  // Dynamic endpoints
  // Response simulation
  // Request interception
}

// 3. Object relationship mapper
class ORMProxy {
  // Relationship loading
  // Query building
  // Cache management
}
```

### **🎨 แบบฝึกหัดที่ 2: Decorator System**
สร้าง comprehensive decorator system:

```javascript
// TODO: สร้าง decorator system

// 1. Validation framework
class ValidationDecorator {
  // Schema validation
  // Custom validators
  // Error handling
}

// 2. Security decorators
class SecurityDecorator {
  // Authentication
  // Authorization
  // Rate limiting
}

// 3. Performance decorators
class PerformanceDecorator {
  // Profiling
  // Optimization
  // Monitoring
}
```

### **🔧 แบบฝึกหัดที่ 3: Code Generation**
สร้าง sophisticated code generation:

```javascript
// TODO: สร้าง code generation

// 1. Full-stack generator
class FullStackGenerator {
  // API generation
  // Frontend components
  // Database schemas
}

// 2. Migration system
class MigrationGenerator {
  // Schema changes
  // Data transformations
  // Rollback support
}

// 3. Documentation generator
class DocGenerator {
  // API documentation
  // Component stories
  // Type definitions
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 17: Advanced Functions](./17-advanced-functions.md)

**บทถัดไป**: [บทที่ 19: Web APIs](./19-web-apis.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Proxy - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- [Reflect - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
- [Decorators Proposal](https://github.com/tc39/proposal-decorators)
- [Metaprogramming in JavaScript](https://en.wikipedia.org/wiki/Metaprogramming)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Proxy และ Reflect** - property interception, reactive systems, และ API proxies
- **Decorators and Annotations** - method decorators, class decorators, และ metadata management
- **Dynamic Code Generation** - template systems, API generation, และ query builders
- **Advanced Patterns** - aspect-oriented programming, composite decorators, และ optimization
- **Next.js Integration** - component generation, API route creation, และ type generation

Metaprogramming เป็นเทคนิคขั้นสูงที่ให้ความยืดหยุ่นและประสิทธิภาพสูง! 🚀
