# บทที่ 10: ES6 Modules

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ ES6 Module System (import/export)
- ทำความเข้าใจ Module Patterns และ Best Practices
- เรียนรู้ Dynamic Imports และ Code Splitting
- ประยุกต์ใช้ใน Next.js สำหรับ Module Organization

## 📦 ES6 Module Basics

### **📤 Export Patterns**

```javascript
// ============= NAMED EXPORTS =============
// utils/mathUtils.js
export const PI = 3.14159;
export const E = 2.71828;

export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

export const divide = (a, b) => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};

// Export multiple items at once
export { add as sum, multiply as product };

// ============= DEFAULT EXPORTS =============
// utils/Calculator.js
class Calculator {
  constructor() {
    this.history = [];
  }
  
  add(a, b) {
    const result = a + b;
    this.history.push({ operation: 'add', operands: [a, b], result });
    return result;
  }
  
  subtract(a, b) {
    const result = a - b;
    this.history.push({ operation: 'subtract', operands: [a, b], result });
    return result;
  }
  
  getHistory() {
    return [...this.history];
  }
  
  clearHistory() {
    this.history = [];
  }
}

export default Calculator;

// ============= MIXED EXPORTS =============
// utils/dataUtils.js
export const formatCurrency = (amount, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency
  }).format(amount);
};

export const formatDate = (date, locale = 'en-US') => {
  return new Intl.DateTimeFormat(locale).format(date);
};

export const validateEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Default export
const DataProcessor = {
  processUsers(users) {
    return users.map(user => ({
      ...user,
      displayName: `${user.firstName} ${user.lastName}`,
      formattedJoinDate: formatDate(new Date(user.joinDate))
    }));
  },
  
  aggregateData(data, groupBy) {
    return data.reduce((acc, item) => {
      const key = item[groupBy];
      if (!acc[key]) {
        acc[key] = [];
      }
      acc[key].push(item);
      return acc;
    }, {});
  }
};

export default DataProcessor;

// ============= RE-EXPORTS =============
// utils/index.js - Central export file
export { default as Calculator } from './Calculator.js';
export { default as DataProcessor } from './dataUtils.js';
export * from './mathUtils.js';
export { formatCurrency, formatDate, validateEmail } from './dataUtils.js';

// Re-export with renaming
export { add as addNumbers, multiply as multiplyNumbers } from './mathUtils.js';

// ============= CONDITIONAL EXPORTS =============
// utils/environmentUtils.js
const isDevelopment = process.env.NODE_ENV === 'development';
const isProduction = process.env.NODE_ENV === 'production';

// Different exports based on environment
if (isDevelopment) {
  export const debugLog = (message, data) => {
    console.log(`[DEBUG] ${message}`, data);
  };
  
  export const performance = {
    start: (label) => console.time(label),
    end: (label) => console.timeEnd(label)
  };
} else {
  export const debugLog = () => {}; // No-op in production
  export const performance = {
    start: () => {},
    end: () => {}
  };
}

export const config = {
  apiUrl: isProduction 
    ? 'https://api.production.com' 
    : 'http://localhost:3001',
  enableLogging: isDevelopment,
  cacheTimeout: isProduction ? 300000 : 5000
};
```

### **📥 Import Patterns**

```javascript
// ============= NAMED IMPORTS =============
// main.js
import { add, multiply, PI } from './utils/mathUtils.js';
import { formatCurrency, validateEmail } from './utils/dataUtils.js';

console.log(add(5, 3)); // 8
console.log(multiply(4, 7)); // 28
console.log(PI); // 3.14159

// Import with renaming
import { add as addNumbers, multiply as times } from './utils/mathUtils.js';

// ============= DEFAULT IMPORTS =============
import Calculator from './utils/Calculator.js';
import DataProcessor from './utils/dataUtils.js';

const calc = new Calculator();
console.log(calc.add(10, 5)); // 15

const users = [
  { firstName: 'John', lastName: 'Doe', joinDate: '2023-01-15' },
  { firstName: 'Jane', lastName: 'Smith', joinDate: '2023-02-20' }
];
const processedUsers = DataProcessor.processUsers(users);

// ============= MIXED IMPORTS =============
import Calculator, { formatCurrency, validateEmail } from './utils/mixed.js';

// ============= NAMESPACE IMPORTS =============
import * as MathUtils from './utils/mathUtils.js';
import * as DataUtils from './utils/dataUtils.js';

console.log(MathUtils.add(1, 2));
console.log(MathUtils.PI);
console.log(DataUtils.formatCurrency(1000));

// ============= SIDE EFFECT IMPORTS =============
// polyfills.js
if (!Array.prototype.includes) {
  Array.prototype.includes = function(searchElement) {
    return this.indexOf(searchElement) !== -1;
  };
}

// Import for side effects only
import './polyfills.js';

// ============= DYNAMIC IMPORTS =============
// Lazy loading modules
const loadMathUtils = async () => {
  const module = await import('./utils/mathUtils.js');
  return module;
};

// Conditional loading
const loadCalculator = async (advanced = false) => {
  if (advanced) {
    const { default: AdvancedCalculator } = await import('./utils/AdvancedCalculator.js');
    return AdvancedCalculator;
  } else {
    const { default: Calculator } = await import('./utils/Calculator.js');
    return Calculator;
  }
};

// Dynamic import with error handling
const loadModule = async (moduleName) => {
  try {
    const module = await import(`./modules/${moduleName}.js`);
    return module;
  } catch (error) {
    console.error(`Failed to load module ${moduleName}:`, error);
    // Return fallback or default implementation
    return { default: () => console.log('Module not available') };
  }
};

// Usage
(async () => {
  const mathUtils = await loadMathUtils();
  console.log(mathUtils.add(5, 3));
  
  const CalculatorClass = await loadCalculator(true);
  const calc = new CalculatorClass();
})();
```

### **🏗️ Advanced Module Patterns**

```javascript
// ============= MODULE FACTORY PATTERN =============
// factories/createApiClient.js
export const createApiClient = (config = {}) => {
  const {
    baseURL = '/api',
    timeout = 5000,
    headers = {},
    retries = 3
  } = config;
  
  const defaultHeaders = {
    'Content-Type': 'application/json',
    ...headers
  };
  
  return {
    async request(endpoint, options = {}) {
      const url = `${baseURL}${endpoint}`;
      const requestConfig = {
        timeout,
        headers: { ...defaultHeaders, ...options.headers },
        ...options
      };
      
      let lastError;
      for (let attempt = 1; attempt <= retries; attempt++) {
        try {
          const response = await fetch(url, requestConfig);
          if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
          }
          return await response.json();
        } catch (error) {
          lastError = error;
          if (attempt === retries) break;
          
          // Wait before retry with exponential backoff
          await new Promise(resolve => 
            setTimeout(resolve, Math.pow(2, attempt) * 100)
          );
        }
      }
      throw lastError;
    },
    
    get(endpoint, params = {}) {
      const queryString = new URLSearchParams(params).toString();
      const url = queryString ? `${endpoint}?${queryString}` : endpoint;
      return this.request(url);
    },
    
    post(endpoint, data) {
      return this.request(endpoint, {
        method: 'POST',
        body: JSON.stringify(data)
      });
    },
    
    put(endpoint, data) {
      return this.request(endpoint, {
        method: 'PUT',
        body: JSON.stringify(data)
      });
    },
    
    delete(endpoint) {
      return this.request(endpoint, {
        method: 'DELETE'
      });
    }
  };
};

// ============= SINGLETON MODULE PATTERN =============
// services/ConfigService.js
class ConfigService {
  constructor() {
    if (ConfigService.instance) {
      return ConfigService.instance;
    }
    
    this.config = new Map();
    this.listeners = new Set();
    ConfigService.instance = this;
  }
  
  set(key, value) {
    const oldValue = this.config.get(key);
    this.config.set(key, value);
    
    // Notify listeners
    this.listeners.forEach(listener => {
      listener(key, value, oldValue);
    });
  }
  
  get(key, defaultValue = undefined) {
    return this.config.get(key) ?? defaultValue;
  }
  
  has(key) {
    return this.config.has(key);
  }
  
  delete(key) {
    return this.config.delete(key);
  }
  
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
  
  getAll() {
    return Object.fromEntries(this.config);
  }
  
  load(configObject) {
    Object.entries(configObject).forEach(([key, value]) => {
      this.set(key, value);
    });
  }
}

// Export singleton instance
export const configService = new ConfigService();

// ============= PLUGIN SYSTEM MODULE =============
// core/PluginManager.js
export class PluginManager {
  constructor() {
    this.plugins = new Map();
    this.hooks = new Map();
  }
  
  register(name, plugin) {
    if (this.plugins.has(name)) {
      throw new Error(`Plugin ${name} already registered`);
    }
    
    // Validate plugin interface
    if (typeof plugin.init !== 'function') {
      throw new Error(`Plugin ${name} must have an init method`);
    }
    
    this.plugins.set(name, plugin);
    
    // Initialize plugin
    plugin.init(this);
    
    return this;
  }
  
  unregister(name) {
    const plugin = this.plugins.get(name);
    if (plugin && typeof plugin.destroy === 'function') {
      plugin.destroy();
    }
    
    return this.plugins.delete(name);
  }
  
  addHook(hookName, callback, priority = 0) {
    if (!this.hooks.has(hookName)) {
      this.hooks.set(hookName, []);
    }
    
    this.hooks.get(hookName).push({ callback, priority });
    
    // Sort by priority (higher priority first)
    this.hooks.get(hookName).sort((a, b) => b.priority - a.priority);
  }
  
  async executeHook(hookName, data = {}) {
    const hooks = this.hooks.get(hookName) || [];
    let result = data;
    
    for (const { callback } of hooks) {
      try {
        result = await callback(result) || result;
      } catch (error) {
        console.error(`Hook ${hookName} error:`, error);
      }
    }
    
    return result;
  }
  
  getPlugin(name) {
    return this.plugins.get(name);
  }
  
  getPlugins() {
    return Array.from(this.plugins.keys());
  }
}

// Create global plugin manager
export const pluginManager = new PluginManager();

// ============= LAZY LOADING MODULE =============
// utils/lazyLoader.js
export class LazyLoader {
  constructor() {
    this.cache = new Map();
    this.loading = new Map();
  }
  
  async load(modulePath, exportName = 'default') {
    const cacheKey = `${modulePath}#${exportName}`;
    
    // Return cached module
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }
    
    // Wait if already loading
    if (this.loading.has(cacheKey)) {
      return this.loading.get(cacheKey);
    }
    
    // Start loading
    const loadPromise = this.loadModule(modulePath, exportName);
    this.loading.set(cacheKey, loadPromise);
    
    try {
      const module = await loadPromise;
      this.cache.set(cacheKey, module);
      return module;
    } catch (error) {
      // Remove from loading cache on error
      this.loading.delete(cacheKey);
      throw error;
    } finally {
      this.loading.delete(cacheKey);
    }
  }
  
  async loadModule(modulePath, exportName) {
    try {
      const module = await import(modulePath);
      
      if (exportName === '*') {
        return module;
      }
      
      if (!(exportName in module)) {
        throw new Error(`Export '${exportName}' not found in module '${modulePath}'`);
      }
      
      return module[exportName];
    } catch (error) {
      throw new Error(`Failed to load module '${modulePath}': ${error.message}`);
    }
  }
  
  preload(modulePaths) {
    return Promise.all(
      modulePaths.map(path => 
        this.load(path).catch(error => 
          console.warn(`Failed to preload ${path}:`, error)
        )
      )
    );
  }
  
  clear() {
    this.cache.clear();
    this.loading.clear();
  }
  
  remove(modulePath, exportName = 'default') {
    const cacheKey = `${modulePath}#${exportName}`;
    this.cache.delete(cacheKey);
  }
}

export const lazyLoader = new LazyLoader();

// ============= MODULE REGISTRY PATTERN =============
// core/ModuleRegistry.js
export class ModuleRegistry {
  constructor() {
    this.modules = new Map();
    this.dependencies = new Map();
    this.initialized = new Set();
  }
  
  register(name, moduleFactory, dependencies = []) {
    if (this.modules.has(name)) {
      throw new Error(`Module ${name} already registered`);
    }
    
    this.modules.set(name, moduleFactory);
    this.dependencies.set(name, dependencies);
    
    return this;
  }
  
  async get(name) {
    if (this.initialized.has(name)) {
      return this.modules.get(name);
    }
    
    if (!this.modules.has(name)) {
      throw new Error(`Module ${name} not found`);
    }
    
    // Initialize dependencies first
    const deps = this.dependencies.get(name) || [];
    const resolvedDeps = await Promise.all(
      deps.map(dep => this.get(dep))
    );
    
    // Initialize module
    const moduleFactory = this.modules.get(name);
    const module = await moduleFactory(...resolvedDeps);
    
    // Cache initialized module
    this.modules.set(name, module);
    this.initialized.add(name);
    
    return module;
  }
  
  async initialize(moduleNames = []) {
    if (moduleNames.length === 0) {
      moduleNames = Array.from(this.modules.keys());
    }
    
    return Promise.all(moduleNames.map(name => this.get(name)));
  }
  
  topologicalSort() {
    const visited = new Set();
    const temp = new Set();
    const result = [];
    
    const visit = (name) => {
      if (temp.has(name)) {
        throw new Error(`Circular dependency detected: ${name}`);
      }
      
      if (!visited.has(name)) {
        temp.add(name);
        
        const deps = this.dependencies.get(name) || [];
        deps.forEach(visit);
        
        temp.delete(name);
        visited.add(name);
        result.push(name);
      }
    };
    
    Array.from(this.modules.keys()).forEach(visit);
    return result;
  }
}

export const moduleRegistry = new ModuleRegistry();
```

## 🚀 Next.js Module Applications

### **📁 Project Structure with Modules**

```javascript
// ============= NEXT.JS PROJECT STRUCTURE =============
/*
project/
├── components/
│   ├── ui/
│   │   ├── index.js          // Re-export all UI components
│   │   ├── Button.jsx
│   │   ├── Input.jsx
│   │   └── Modal.jsx
│   ├── layout/
│   │   ├── index.js
│   │   ├── Header.jsx
│   │   ├── Footer.jsx
│   │   └── Sidebar.jsx
│   └── features/
│       ├── auth/
│       │   ├── index.js
│       │   ├── LoginForm.jsx
│       │   └── SignupForm.jsx
│       └── dashboard/
│           ├── index.js
│           ├── DashboardLayout.jsx
│           └── widgets/
├── lib/
│   ├── api/
│   │   ├── index.js
│   │   ├── client.js
│   │   └── endpoints.js
│   ├── utils/
│   │   ├── index.js
│   │   ├── formatters.js
│   │   └── validators.js
│   └── hooks/
│       ├── index.js
│       ├── useApi.js
│       └── useLocalStorage.js
├── services/
│   ├── index.js
│   ├── AuthService.js
│   ├── DataService.js
│   └── CacheService.js
└── config/
    ├── index.js
    ├── database.js
    └── api.js
*/

// ============= COMPONENT MODULE ORGANIZATION =============
// components/ui/index.js
export { default as Button } from './Button.jsx';
export { default as Input } from './Input.jsx';
export { default as Modal } from './Modal.jsx';
export { default as Loading } from './Loading.jsx';

// components/ui/Button.jsx
import { forwardRef } from 'react';
import styles from './Button.module.css';

const Button = forwardRef(({ 
  variant = 'primary', 
  size = 'medium', 
  loading = false,
  disabled = false,
  children,
  onClick,
  ...props 
}, ref) => {
  const handleClick = (e) => {
    if (loading || disabled) return;
    onClick?.(e);
  };

  return (
    <button
      ref={ref}
      className={`
        ${styles.button}
        ${styles[variant]}
        ${styles[size]}
        ${loading ? styles.loading : ''}
        ${disabled ? styles.disabled : ''}
      `}
      disabled={disabled || loading}
      onClick={handleClick}
      {...props}
    >
      {loading ? <span className={styles.spinner} /> : children}
    </button>
  );
});

Button.displayName = 'Button';

export default Button;

// components/features/auth/index.js
export { default as LoginForm } from './LoginForm.jsx';
export { default as SignupForm } from './SignupForm.jsx';
export { default as AuthProvider } from './AuthProvider.jsx';
export { default as ProtectedRoute } from './ProtectedRoute.jsx';

// ============= SERVICE LAYER MODULES =============
// services/AuthService.js
import { createApiClient } from '../lib/api';

class AuthService {
  constructor() {
    this.apiClient = createApiClient({
      baseURL: '/api/auth'
    });
    this.currentUser = null;
    this.listeners = new Set();
  }
  
  async login(credentials) {
    try {
      const response = await this.apiClient.post('/login', credentials);
      this.currentUser = response.user;
      this.notifyListeners('login', response.user);
      
      // Store token
      if (typeof window !== 'undefined') {
        localStorage.setItem('authToken', response.token);
      }
      
      return response;
    } catch (error) {
      this.notifyListeners('loginError', error);
      throw error;
    }
  }
  
  async logout() {
    try {
      await this.apiClient.post('/logout');
    } catch (error) {
      console.warn('Logout request failed:', error);
    } finally {
      this.currentUser = null;
      this.notifyListeners('logout');
      
      if (typeof window !== 'undefined') {
        localStorage.removeItem('authToken');
      }
    }
  }
  
  async getCurrentUser() {
    if (this.currentUser) {
      return this.currentUser;
    }
    
    try {
      const response = await this.apiClient.get('/me');
      this.currentUser = response.user;
      return response.user;
    } catch (error) {
      return null;
    }
  }
  
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
  
  notifyListeners(event, data) {
    this.listeners.forEach(listener => {
      try {
        listener(event, data);
      } catch (error) {
        console.error('Auth listener error:', error);
      }
    });
  }
}

// Export singleton instance
export const authService = new AuthService();
export default AuthService;

// services/index.js - Central service exports
export { authService } from './AuthService.js';
export { dataService } from './DataService.js';
export { cacheService } from './CacheService.js';
export { notificationService } from './NotificationService.js';

// ============= CUSTOM HOOKS MODULE =============
// lib/hooks/useAuth.js
import { useState, useEffect, useContext, createContext } from 'react';
import { authService } from '../../services';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const initAuth = async () => {
      try {
        const currentUser = await authService.getCurrentUser();
        setUser(currentUser);
      } catch (error) {
        setError(error);
      } finally {
        setLoading(false);
      }
    };
    
    initAuth();
    
    // Subscribe to auth changes
    const unsubscribe = authService.subscribe((event, data) => {
      switch (event) {
        case 'login':
          setUser(data);
          setError(null);
          break;
        case 'logout':
          setUser(null);
          setError(null);
          break;
        case 'loginError':
          setError(data);
          break;
      }
    });
    
    return unsubscribe;
  }, []);
  
  const login = async (credentials) => {
    setLoading(true);
    setError(null);
    
    try {
      await authService.login(credentials);
    } catch (error) {
      setError(error);
      throw error;
    } finally {
      setLoading(false);
    }
  };
  
  const logout = async () => {
    setLoading(true);
    
    try {
      await authService.logout();
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const value = {
    user,
    loading,
    error,
    login,
    logout,
    isAuthenticated: !!user
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

// lib/hooks/index.js
export { useAuth, AuthProvider } from './useAuth.js';
export { useApi } from './useApi.js';
export { useLocalStorage } from './useLocalStorage.js';
export { useDebounce } from './useDebounce.js';

// ============= API CLIENT MODULE =============
// lib/api/client.js
import { createApiClient } from './factory.js';

// Create different API clients for different services
export const mainApiClient = createApiClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL || '/api',
  timeout: 10000,
  headers: {
    'X-Client-Version': process.env.NEXT_PUBLIC_APP_VERSION || '1.0.0'
  }
});

export const authApiClient = createApiClient({
  baseURL: `${process.env.NEXT_PUBLIC_API_URL || '/api'}/auth`,
  timeout: 5000
});

export const uploadsApiClient = createApiClient({
  baseURL: `${process.env.NEXT_PUBLIC_API_URL || '/api'}/uploads`,
  timeout: 30000, // Longer timeout for uploads
  headers: {} // No Content-Type header for file uploads
});

// Add request interceptor for authentication
if (typeof window !== 'undefined') {
  mainApiClient.addRequestInterceptor(async (config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
  });
}

// lib/api/endpoints.js
import { mainApiClient, authApiClient } from './client.js';

export const userApi = {
  getAll: (params) => mainApiClient.get('/users', params),
  getById: (id) => mainApiClient.get(`/users/${id}`),
  create: (data) => mainApiClient.post('/users', data),
  update: (id, data) => mainApiClient.put(`/users/${id}`, data),
  delete: (id) => mainApiClient.delete(`/users/${id}`),
  search: (query) => mainApiClient.get('/users/search', { q: query })
};

export const authApi = {
  login: (credentials) => authApiClient.post('/login', credentials),
  logout: () => authApiClient.post('/logout'),
  register: (userData) => authApiClient.post('/register', userData),
  refreshToken: () => authApiClient.post('/refresh'),
  resetPassword: (email) => authApiClient.post('/reset-password', { email }),
  verifyEmail: (token) => authApiClient.post('/verify-email', { token })
};

export const postApi = {
  getAll: (params) => mainApiClient.get('/posts', params),
  getById: (id) => mainApiClient.get(`/posts/${id}`),
  create: (data) => mainApiClient.post('/posts', data),
  update: (id, data) => mainApiClient.put(`/posts/${id}`, data),
  delete: (id) => mainApiClient.delete(`/posts/${id}`),
  publish: (id) => mainApiClient.post(`/posts/${id}/publish`),
  unpublish: (id) => mainApiClient.post(`/posts/${id}/unpublish`)
};

// lib/api/index.js
export { mainApiClient, authApiClient, uploadsApiClient } from './client.js';
export { userApi, authApi, postApi } from './endpoints.js';
export { createApiClient } from './factory.js';
```

### **🔄 Dynamic Module Loading in Next.js**

```jsx
// ============= DYNAMIC COMPONENT LOADING =============
// components/DynamicLoader.jsx
import { useState, useEffect } from 'react';
import dynamic from 'next/dynamic';

// Higher-order component for dynamic loading
export const withDynamicLoading = (importFunction, options = {}) => {
  const {
    loading: LoadingComponent = () => <div>Loading...</div>,
    error: ErrorComponent = ({ error }) => <div>Error: {error.message}</div>,
    retry = true
  } = options;
  
  return dynamic(importFunction, {
    loading: LoadingComponent,
    ssr: false,
    ...options
  });
};

// Dynamic module loader with error handling
export const DynamicModuleLoader = ({ 
  modulePath, 
  children, 
  fallback = null,
  onError = console.error 
}) => {
  const [module, setModule] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    const loadModule = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const loadedModule = await import(modulePath);
        
        if (!cancelled) {
          setModule(loadedModule);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err);
          onError(err);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };
    
    loadModule();
    
    return () => {
      cancelled = true;
    };
  }, [modulePath, onError]);
  
  if (loading) {
    return fallback || <div>Loading module...</div>;
  }
  
  if (error) {
    return <div>Failed to load module: {error.message}</div>;
  }
  
  return children(module);
};

// Usage example
const DynamicChartComponent = () => {
  return (
    <DynamicModuleLoader
      modulePath="./charts/AdvancedChart"
      fallback={<div>Loading chart...</div>}
    >
      {(chartModule) => {
        const Chart = chartModule.default;
        return <Chart data={chartData} />;
      }}
    </DynamicModuleLoader>
  );
};

// ============= FEATURE-BASED DYNAMIC LOADING =============
// components/FeatureLoader.jsx
import { useState, useEffect } from 'react';
import { lazyLoader } from '../lib/utils/lazyLoader.js';

const featureModules = {
  dashboard: () => import('../features/dashboard'),
  analytics: () => import('../features/analytics'),
  userManagement: () => import('../features/userManagement'),
  settings: () => import('../features/settings')
};

export const FeatureLoader = ({ feature, ...props }) => {
  const [FeatureComponent, setFeatureComponent] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const loadFeature = async () => {
      try {
        setLoading(true);
        setError(null);
        
        if (!featureModules[feature]) {
          throw new Error(`Unknown feature: ${feature}`);
        }
        
        const module = await featureModules[feature]();
        setFeatureComponent(() => module.default);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };
    
    loadFeature();
  }, [feature]);
  
  if (loading) {
    return (
      <div className="feature-loading">
        <div className="spinner" />
        <p>Loading {feature}...</p>
      </div>
    );
  }
  
  if (error) {
    return (
      <div className="feature-error">
        <h3>Failed to load {feature}</h3>
        <p>{error.message}</p>
        <button onClick={() => window.location.reload()}>
          Retry
        </button>
      </div>
    );
  }
  
  if (!FeatureComponent) {
    return <div>Feature not available</div>;
  }
  
  return <FeatureComponent {...props} />;
};

// ============= CODE SPLITTING BY ROUTE =============
// pages/dashboard/index.js
import dynamic from 'next/dynamic';
import { useAuth } from '../../lib/hooks';

// Dynamic imports for dashboard widgets
const DashboardOverview = dynamic(() => 
  import('../../components/features/dashboard/Overview'), {
  loading: () => <div>Loading overview...</div>
});

const DashboardCharts = dynamic(() => 
  import('../../components/features/dashboard/Charts'), {
  loading: () => <div>Loading charts...</div>,
  ssr: false
});

const DashboardTables = dynamic(() => 
  import('../../components/features/dashboard/Tables'), {
  loading: () => <div>Loading tables...</div>
});

const Dashboard = () => {
  const { user, isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    return <div>Please log in to view dashboard</div>;
  }
  
  return (
    <div className="dashboard">
      <h1>Welcome, {user.name}</h1>
      
      <div className="dashboard-grid">
        <section className="overview">
          <DashboardOverview userId={user.id} />
        </section>
        
        <section className="charts">
          <DashboardCharts userId={user.id} />
        </section>
        
        <section className="tables">
          <DashboardTables userId={user.id} />
        </section>
      </div>
    </div>
  );
};

export default Dashboard;

// ============= PLUGIN SYSTEM FOR NEXT.JS =============
// lib/plugins/index.js
import { pluginManager } from '../core/PluginManager.js';

// Analytics plugin
const analyticsPlugin = {
  name: 'analytics',
  
  init(manager) {
    console.log('Analytics plugin initialized');
    
    // Add tracking hooks
    manager.addHook('pageView', async (data) => {
      if (typeof window !== 'undefined' && window.gtag) {
        window.gtag('event', 'page_view', {
          page_title: data.title,
          page_location: data.url
        });
      }
      return data;
    });
    
    manager.addHook('userEvent', async (data) => {
      if (typeof window !== 'undefined' && window.gtag) {
        window.gtag('event', data.action, {
          event_category: data.category,
          event_label: data.label,
          value: data.value
        });
      }
      return data;
    });
  },
  
  destroy() {
    console.log('Analytics plugin destroyed');
  }
};

// Theme plugin
const themePlugin = {
  name: 'theme',
  
  init(manager) {
    console.log('Theme plugin initialized');
    
    manager.addHook('themeChange', async (data) => {
      if (typeof window !== 'undefined') {
        document.documentElement.setAttribute('data-theme', data.theme);
        localStorage.setItem('theme', data.theme);
      }
      return data;
    });
  },
  
  destroy() {
    console.log('Theme plugin destroyed');
  }
};

// Register plugins
pluginManager
  .register('analytics', analyticsPlugin)
  .register('theme', themePlugin);

export { pluginManager };

// pages/_app.js
import { useEffect } from 'react';
import { pluginManager } from '../lib/plugins';

function MyApp({ Component, pageProps, router }) {
  useEffect(() => {
    // Track page views
    const handleRouteChange = (url) => {
      pluginManager.executeHook('pageView', {
        url,
        title: document.title
      });
    };
    
    router.events.on('routeChangeComplete', handleRouteChange);
    
    return () => {
      router.events.off('routeChangeComplete', handleRouteChange);
    };
  }, [router]);
  
  return <Component {...pageProps} />;
}

export default MyApp;
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Module Organization**
จัดระเบียบโครงสร้าง modules:

```javascript
// TODO: สร้างระบบ module organization

// 1. Create a feature-based module system
// Structure:
// features/
//   ├── auth/
//   ├── dashboard/
//   ├── users/
//   └── settings/

// 2. Implement module dependency injection
class ModuleDI {
  constructor() {
    // Implement dependency injection for modules
  }
}

// 3. Create module hot-reloading system
class ModuleHotReloader {
  constructor() {
    // Implement hot reloading for development
  }
}
```

### **🔄 แบบฝึกหัดที่ 2: Dynamic Module System**
สร้างระบบ dynamic modules:

```javascript
// TODO: สร้าง dynamic module system

// 1. Implement module marketplace
class ModuleMarketplace {
  async installModule(moduleId) {
    // Download and install module dynamically
  }
  
  async uninstallModule(moduleId) {
    // Remove module and clean up
  }
}

// 2. Create module sandboxing
class ModuleSandbox {
  execute(moduleCode, permissions) {
    // Execute module code in isolated environment
  }
}

// 3. Implement module versioning
class ModuleVersionManager {
  async upgradeModule(moduleId, version) {
    // Handle module upgrades safely
  }
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Module Integration**
สร้าง integrated module system:

```jsx
// TODO: สร้าง Next.js module system ที่:
// 1. Support plugin architecture
// 2. Handle dynamic feature loading
// 3. Implement module permissions
// 4. Support module marketplace

const ModularApp = () => {
  // Implement modular application architecture
  // Load modules based on user permissions
  // Handle module lifecycle
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 9: Promises & Async/Await](./09-promises-async.md)

**บทถัดไป**: [บทที่ 11: Iterators & Generators](./11-iterators-generators.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: ES6 Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [MDN: Dynamic Imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#dynamic_imports)
- [Next.js: Dynamic Imports](https://nextjs.org/docs/advanced-features/dynamic-import)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **ES6 Module System** (import/export)
- **Advanced Module Patterns** และ organization
- **Dynamic Imports** และ code splitting
- **Module Architecture** สำหรับ scalable applications
- **การใช้งานใน Next.js** สำหรับ modular development

ES6 Modules เป็นพื้นฐานสำคัญสำหรับการจัดระเบียบโค้ดใน modern JavaScript applications! 🚀
