# บทที่ 10: ES6 Modules

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ ES6 Module System (import/export)
- ทำความเข้าใจ Module Patterns และ Best Practices
- เรียนรู้ Dynamic Imports และ Code Splitting
- ประยุกต์ใช้ใน Next.js สำหรับ Module Organization

## 📦 Module Basics

### **📤 Export Patterns**

```javascript
// ============= NAMED EXPORTS =============
// math.js - Multiple named exports
export const PI = 3.14159;
export const E = 2.71828;

export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

export class Calculator {
  constructor() {
    this.history = [];
  }
  
  add(a, b) {
    const result = a + b;
    this.history.push({ operation: 'add', operands: [a, b], result });
    return result;
  }
  
  getHistory() {
    return [...this.history];
  }
}

// Alternative export syntax
const subtract = (a, b) => a - b;
const divide = (a, b) => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};

export { subtract, divide };

// Export with alias
const power = (base, exponent) => Math.pow(base, exponent);
export { power as pow };

// ============= DEFAULT EXPORTS =============
// logger.js - Single default export
class Logger {
  constructor(name) {
    this.name = name;
    this.logs = [];
  }
  
  log(level, message) {
    const timestamp = new Date().toISOString();
    const logEntry = { timestamp, level, message, source: this.name };
    
    this.logs.push(logEntry);
    console.log(`[${timestamp}] ${level.toUpperCase()}: ${message}`);
  }
  
  info(message) {
    this.log('info', message);
  }
  
  warn(message) {
    this.log('warn', message);
  }
  
  error(message) {
    this.log('error', message);
  }
  
  getLogs(level = null) {
    if (level) {
      return this.logs.filter(log => log.level === level);
    }
    return [...this.logs];
  }
}

export default Logger;

// ============= MIXED EXPORTS =============
// utils.js - Both named and default exports
export const VERSION = '1.0.0';

export function formatDate(date) {
  return date.toLocaleDateString();
}

export function formatCurrency(amount, currency = 'USD') {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount);
}

// Default utility object
const Utils = {
  formatDate,
  formatCurrency,
  VERSION,
  
  // Additional utility methods
  debounce(func, delay) {
    let timeoutId;
    return function (...args) {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
  },
  
  throttle(func, limit) {
    let inThrottle;
    return function (...args) {
      if (!inThrottle) {
        func.apply(this, args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  },
  
  deepClone(obj) {
    if (obj === null || typeof obj !== 'object') return obj;
    if (obj instanceof Date) return new Date(obj.getTime());
    if (obj instanceof Array) return obj.map(item => this.deepClone(item));
    
    const cloned = {};
    for (const key in obj) {
      if (obj.hasOwnProperty(key)) {
        cloned[key] = this.deepClone(obj[key]);
      }
    }
    return cloned;
  }
};

export default Utils;

// ============= RE-EXPORTS =============
// index.js - Barrel exports
export { default as Logger } from './logger.js';
export { add, multiply, Calculator } from './math.js';
export { default as Utils, formatDate, formatCurrency } from './utils.js';

// Re-export with alias
export { default as MathUtils } from './math.js';

// Re-export all named exports
export * from './constants.js';

// Re-export default as named
export { default as ApiClient } from './api-client.js';
```

### **📥 Import Patterns**

```javascript
// ============= NAMED IMPORTS =============
// Basic named imports
import { add, multiply, PI } from './math.js';

console.log(add(5, 3)); // 8
console.log(multiply(PI, 2)); // 6.28318

// Import with alias
import { pow as power } from './math.js';
console.log(power(2, 3)); // 8

// Multiple imports
import { 
  add, 
  subtract, 
  multiply as mult, 
  divide as div 
} from './math.js';

// Import all named exports
import * as MathFunctions from './math.js';
console.log(MathFunctions.add(1, 2));
console.log(MathFunctions.PI);

// ============= DEFAULT IMPORTS =============
// Import default export
import Logger from './logger.js';
const logger = new Logger('AppLogger');
logger.info('Application started');

// Import default with custom name
import MyLogger from './logger.js';
import DatabaseLogger from './logger.js';

// ============= MIXED IMPORTS =============
// Import both default and named
import Utils, { formatDate, VERSION } from './utils.js';

console.log(VERSION); // '1.0.0'
console.log(formatDate(new Date()));
console.log(Utils.formatCurrency(1234.56));

// ============= BARREL IMPORTS =============
// Import from index.js (barrel)
import { Logger, Calculator, Utils } from './';

// Or with explicit index
import { Logger, Calculator, Utils } from './index.js';

// ============= SIDE-EFFECT IMPORTS =============
// Import for side effects only (no bindings)
import './polyfills.js';
import './global-styles.css';

// ============= CONDITIONAL IMPORTS =============
// Note: These are not valid ES6 syntax, but show the concept
// Use dynamic imports instead (covered later)

// if (condition) {
//   import { feature } from './feature.js'; // ❌ Invalid
// }

// ============= IMPORT ORGANIZATION PATTERNS =============
// Organize imports by category
// Built-in modules first (if using Node.js)
import fs from 'fs';
import path from 'path';

// Third-party libraries
import React from 'react';
import axios from 'axios';
import lodash from 'lodash';

// Local modules (relative imports)
import './styles.css';
import Logger from './logger.js';
import { Calculator } from './math.js';
import Utils from '../utils/index.js';
import config from '../../config.js';

// ============= MODULE SCOPE AND GLOBALS =============
// Each module has its own scope
let moduleVariable = 'This is private to the module';

function privateFunction() {
  return 'Only accessible within this module';
}

// Only exported items are accessible from outside
export function publicFunction() {
  return privateFunction(); // Can access private items within module
}

// Module runs once when first imported
console.log('Module loaded!'); // This runs when module is first imported

let initializationCount = 0;
export function getInitCount() {
  return ++initializationCount;
}
```

## 🏗️ Advanced Module Patterns

### **🔄 Dynamic Imports**

```javascript
// ============= BASIC DYNAMIC IMPORTS =============
// Dynamic import returns a Promise
async function loadMathModule() {
  try {
    const mathModule = await import('./math.js');
    
    console.log(mathModule.add(5, 3));
    console.log(mathModule.PI);
    
    // Access default export
    if (mathModule.default) {
      const MathUtils = mathModule.default;
      // Use MathUtils...
    }
    
  } catch (error) {
    console.error('Failed to load math module:', error);
  }
}

// Using .then() syntax
function loadUtilsModule() {
  import('./utils.js')
    .then(utilsModule => {
      const { formatDate, formatCurrency } = utilsModule;
      console.log(formatDate(new Date()));
      console.log(formatCurrency(1234.56));
    })
    .catch(error => {
      console.error('Failed to load utils module:', error);
    });
}

// ============= CONDITIONAL LOADING =============
async function loadFeatureBasedOnCondition(userRole) {
  if (userRole === 'admin') {
    const { AdminPanel } = await import('./admin-panel.js');
    return AdminPanel;
  } else if (userRole === 'moderator') {
    const { ModeratorTools } = await import('./moderator-tools.js');
    return ModeratorTools;
  } else {
    const { UserDashboard } = await import('./user-dashboard.js');
    return UserDashboard;
  }
}

// ============= LAZY LOADING WITH ERROR HANDLING =============
class FeatureLoader {
  constructor() {
    this.loadedModules = new Map();
    this.loadingPromises = new Map();
  }
  
  async loadFeature(featureName) {
    // Return cached module if already loaded
    if (this.loadedModules.has(featureName)) {
      return this.loadedModules.get(featureName);
    }
    
    // Return existing promise if already loading
    if (this.loadingPromises.has(featureName)) {
      return this.loadingPromises.get(featureName);
    }
    
    // Start loading
    const loadingPromise = this.loadFeatureModule(featureName);
    this.loadingPromises.set(featureName, loadingPromise);
    
    try {
      const module = await loadingPromise;
      this.loadedModules.set(featureName, module);
      this.loadingPromises.delete(featureName);
      return module;
    } catch (error) {
      this.loadingPromises.delete(featureName);
      throw error;
    }
  }
  
  async loadFeatureModule(featureName) {
    const moduleMap = {
      'charts': () => import('./features/charts.js'),
      'analytics': () => import('./features/analytics.js'),
      'reporting': () => import('./features/reporting.js'),
      'export': () => import('./features/export.js')
    };
    
    const loader = moduleMap[featureName];
    if (!loader) {
      throw new Error(`Unknown feature: ${featureName}`);
    }
    
    return loader();
  }
  
  async loadMultipleFeatures(featureNames) {
    const loadPromises = featureNames.map(name => this.loadFeature(name));
    return Promise.allSettled(loadPromises);
  }
  
  preloadFeature(featureName) {
    // Start loading but don't wait for completion
    this.loadFeature(featureName).catch(error => {
      console.warn(`Preload failed for ${featureName}:`, error);
    });
  }
  
  isLoaded(featureName) {
    return this.loadedModules.has(featureName);
  }
  
  isLoading(featureName) {
    return this.loadingPromises.has(featureName);
  }
}

// Usage
const featureLoader = new FeatureLoader();

async function showCharts() {
  try {
    const chartsModule = await featureLoader.loadFeature('charts');
    const { ChartRenderer } = chartsModule;
    
    const chart = new ChartRenderer();
    chart.render(data);
  } catch (error) {
    console.error('Failed to load charts:', error);
    // Show fallback UI
  }
}

// ============= MODULE FEDERATION PATTERN =============
class ModuleRegistry {
  constructor() {
    this.modules = new Map();
    this.aliases = new Map();
  }
  
  register(name, moduleLoader, options = {}) {
    this.modules.set(name, {
      loader: moduleLoader,
      version: options.version || '1.0.0',
      dependencies: options.dependencies || [],
      loaded: null,
      loading: null
    });
    
    // Register aliases
    if (options.aliases) {
      options.aliases.forEach(alias => {
        this.aliases.set(alias, name);
      });
    }
  }
  
  async load(name, version = null) {
    const actualName = this.aliases.get(name) || name;
    const moduleInfo = this.modules.get(actualName);
    
    if (!moduleInfo) {
      throw new Error(`Module not found: ${name}`);
    }
    
    if (version && moduleInfo.version !== version) {
      throw new Error(`Version mismatch for ${name}: expected ${version}, got ${moduleInfo.version}`);
    }
    
    // Return cached module
    if (moduleInfo.loaded) {
      return moduleInfo.loaded;
    }
    
    // Return existing loading promise
    if (moduleInfo.loading) {
      return moduleInfo.loading;
    }
    
    // Load dependencies first
    if (moduleInfo.dependencies.length > 0) {
      await Promise.all(
        moduleInfo.dependencies.map(dep => this.load(dep))
      );
    }
    
    // Load the module
    moduleInfo.loading = moduleInfo.loader();
    
    try {
      moduleInfo.loaded = await moduleInfo.loading;
      moduleInfo.loading = null;
      return moduleInfo.loaded;
    } catch (error) {
      moduleInfo.loading = null;
      throw error;
    }
  }
  
  unload(name) {
    const actualName = this.aliases.get(name) || name;
    const moduleInfo = this.modules.get(actualName);
    
    if (moduleInfo) {
      moduleInfo.loaded = null;
      moduleInfo.loading = null;
    }
  }
  
  getLoadedModules() {
    return Array.from(this.modules.entries())
      .filter(([, info]) => info.loaded)
      .map(([name, info]) => ({ name, version: info.version }));
  }
}

// Usage
const registry = new ModuleRegistry();

registry.register('ui-components', () => import('./ui-components.js'), {
  version: '2.1.0',
  aliases: ['ui', 'components']
});

registry.register('data-processing', () => import('./data-processing.js'), {
  version: '1.5.0',
  dependencies: ['ui-components']
});

// Load modules
async function initializeApp() {
  try {
    const [uiModule, dataModule] = await Promise.all([
      registry.load('ui-components'),
      registry.load('data-processing')
    ]);
    
    console.log('All modules loaded:', registry.getLoadedModules());
  } catch (error) {
    console.error('Module loading failed:', error);
  }
}
```

### **🔧 Module Configuration Patterns**

```javascript
// ============= CONFIGURATION MODULE =============
// config/index.js
const environment = process.env.NODE_ENV || 'development';

const baseConfig = {
  app: {
    name: 'My Application',
    version: '1.0.0'
  },
  api: {
    timeout: 30000,
    retries: 3
  },
  features: {
    analytics: true,
    debugMode: false
  }
};

const environmentConfigs = {
  development: {
    api: {
      baseUrl: 'http://localhost:3000/api',
      timeout: 10000
    },
    features: {
      debugMode: true
    }
  },
  
  production: {
    api: {
      baseUrl: 'https://api.example.com',
      timeout: 30000
    },
    features: {
      analytics: true,
      debugMode: false
    }
  },
  
  test: {
    api: {
      baseUrl: 'http://localhost:3001/api',
      timeout: 5000
    },
    features: {
      analytics: false,
      debugMode: true
    }
  }
};

function deepMerge(target, source) {
  const result = { ...target };
  
  for (const key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = deepMerge(target[key] || {}, source[key]);
    } else {
      result[key] = source[key];
    }
  }
  
  return result;
}

const config = deepMerge(baseConfig, environmentConfigs[environment] || {});

export default config;

// Named exports for specific config sections
export const { api: apiConfig } = config;
export const { features: featureConfig } = config;
export const { app: appConfig } = config;

// ============= PLUGIN SYSTEM =============
// plugin-manager.js
class PluginManager {
  constructor() {
    this.plugins = new Map();
    this.hooks = new Map();
    this.middleware = [];
  }
  
  async registerPlugin(name, pluginLoader, options = {}) {
    try {
      const pluginModule = await pluginLoader();
      const plugin = pluginModule.default || pluginModule;
      
      if (typeof plugin.install === 'function') {
        await plugin.install(this, options);
      }
      
      this.plugins.set(name, {
        plugin,
        options,
        active: true
      });
      
      console.log(`Plugin "${name}" registered successfully`);
    } catch (error) {
      console.error(`Failed to register plugin "${name}":`, error);
      throw error;
    }
  }
  
  async unregisterPlugin(name) {
    const pluginInfo = this.plugins.get(name);
    
    if (pluginInfo && typeof pluginInfo.plugin.uninstall === 'function') {
      await pluginInfo.plugin.uninstall(this);
    }
    
    this.plugins.delete(name);
  }
  
  addHook(hookName, callback, priority = 10) {
    if (!this.hooks.has(hookName)) {
      this.hooks.set(hookName, []);
    }
    
    this.hooks.get(hookName).push({ callback, priority });
    
    // Sort by priority
    this.hooks.get(hookName).sort((a, b) => a.priority - b.priority);
  }
  
  async executeHook(hookName, ...args) {
    const hooks = this.hooks.get(hookName) || [];
    const results = [];
    
    for (const { callback } of hooks) {
      try {
        const result = await callback(...args);
        results.push(result);
      } catch (error) {
        console.error(`Hook "${hookName}" failed:`, error);
      }
    }
    
    return results;
  }
  
  addMiddleware(middleware) {
    this.middleware.push(middleware);
  }
  
  async applyMiddleware(context) {
    let result = context;
    
    for (const middleware of this.middleware) {
      try {
        result = await middleware(result);
      } catch (error) {
        console.error('Middleware failed:', error);
        throw error;
      }
    }
    
    return result;
  }
  
  getPluginInfo(name) {
    return this.plugins.get(name);
  }
  
  listPlugins() {
    return Array.from(this.plugins.entries()).map(([name, info]) => ({
      name,
      active: info.active,
      options: info.options
    }));
  }
}

// Example plugin
// plugins/logger-plugin.js
export default {
  async install(pluginManager, options = {}) {
    const { level = 'info', prefix = '[LOG]' } = options;
    
    pluginManager.addHook('beforeRequest', (request) => {
      console.log(`${prefix} Request:`, request.url);
    });
    
    pluginManager.addHook('afterResponse', (response) => {
      console.log(`${prefix} Response:`, response.status);
    });
    
    pluginManager.addMiddleware(async (context) => {
      context.timestamp = new Date().toISOString();
      return context;
    });
  },
  
  async uninstall(pluginManager) {
    console.log('Logger plugin uninstalled');
  }
};

// Usage
const pluginManager = new PluginManager();

await pluginManager.registerPlugin('logger', () => import('./plugins/logger-plugin.js'), {
  level: 'debug',
  prefix: '[DEBUG]'
});

// ============= MODULE FACTORY PATTERN =============
// factory/api-client-factory.js
export function createApiClient(config) {
  const { baseUrl, timeout, retries, interceptors = {} } = config;
  
  class ApiClient {
    constructor() {
      this.baseUrl = baseUrl;
      this.timeout = timeout;
      this.retries = retries;
      this.interceptors = interceptors;
    }
    
    async request(endpoint, options = {}) {
      let url = `${this.baseUrl}${endpoint}`;
      let config = {
        timeout: this.timeout,
        ...options
      };
      
      // Apply request interceptors
      if (this.interceptors.request) {
        config = await this.interceptors.request(config);
      }
      
      let attempt = 0;
      while (attempt <= this.retries) {
        try {
          const response = await fetch(url, config);
          
          // Apply response interceptors
          if (this.interceptors.response) {
            return await this.interceptors.response(response);
          }
          
          return response;
        } catch (error) {
          attempt++;
          if (attempt > this.retries) {
            throw error;
          }
          
          // Wait before retry
          await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
        }
      }
    }
  }
  
  return new ApiClient();
}

// Usage
const apiClient = createApiClient({
  baseUrl: 'https://api.example.com',
  timeout: 10000,
  retries: 3,
  interceptors: {
    request: async (config) => {
      config.headers = {
        ...config.headers,
        'Authorization': `Bearer ${getAuthToken()}`
      };
      return config;
    },
    response: async (response) => {
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return response.json();
    }
  }
});
```

## 🚀 Next.js Module Patterns

### **📁 File Structure Organization**

```typescript
// ============= NEXT.JS MODULE STRUCTURE =============
/*
src/
├── components/
│   ├── ui/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.module.css
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts
│   │   ├── Modal/
│   │   └── index.ts (barrel export)
│   ├── features/
│   │   ├── auth/
│   │   ├── dashboard/
│   │   └── profile/
│   └── layout/
├── hooks/
│   ├── useAuth.ts
│   ├── useApi.ts
│   └── index.ts
├── utils/
│   ├── api.ts
│   ├── formatting.ts
│   └── index.ts
├── types/
│   ├── api.ts
│   ├── user.ts
│   └── index.ts
└── lib/
    ├── auth.ts
    ├── database.ts
    └── index.ts
*/

// components/ui/index.ts - Barrel exports
export { default as Button } from './Button';
export { default as Modal } from './Modal';
export { default as Input } from './Input';
export { default as Card } from './Card';

// Export types
export type { ButtonProps } from './Button';
export type { ModalProps } from './Modal';

// components/ui/Button/index.ts
export { default } from './Button';
export type { ButtonProps } from './Button';

// components/ui/Button/Button.tsx
import React from 'react';
import styles from './Button.module.css';

export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  children,
  onClick
}) => {
  const className = [
    styles.button,
    styles[variant],
    styles[size],
    disabled && styles.disabled,
    loading && styles.loading
  ].filter(Boolean).join(' ');

  return (
    <button
      className={className}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
};

export default Button;

// ============= DYNAMIC IMPORTS IN NEXT.JS =============
// components/DynamicFeatureLoader.tsx
import { useState, useEffect } from 'react';
import dynamic from 'next/dynamic';

// Dynamic import with loading component
const DynamicChart = dynamic(() => import('./Chart'), {
  loading: () => <div>Loading chart...</div>,
  ssr: false // Disable server-side rendering for this component
});

// Dynamic import with named export
const DynamicAnalytics = dynamic(
  () => import('./features/Analytics').then(mod => mod.AnalyticsComponent),
  {
    loading: () => <div>Loading analytics...</div>
  }
);

// Conditional dynamic loading
const ConditionalLoader: React.FC<{ userRole: string }> = ({ userRole }) => {
  const [Component, setComponent] = useState<React.ComponentType | null>(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const loadComponent = async () => {
      setLoading(true);
      
      try {
        let componentModule;
        
        switch (userRole) {
          case 'admin':
            componentModule = await import('./AdminDashboard');
            break;
          case 'manager':
            componentModule = await import('./ManagerDashboard');
            break;
          default:
            componentModule = await import('./UserDashboard');
        }
        
        setComponent(() => componentModule.default);
      } catch (error) {
        console.error('Failed to load component:', error);
      } finally {
        setLoading(false);
      }
    };

    loadComponent();
  }, [userRole]);

  if (loading) {
    return <div>Loading dashboard...</div>;
  }

  if (!Component) {
    return <div>Failed to load dashboard</div>;
  }

  return <Component />;
};

// ============= FEATURE-BASED MODULE ORGANIZATION =============
// features/auth/index.ts
export { default as LoginForm } from './components/LoginForm';
export { default as SignupForm } from './components/SignupForm';
export { default as PasswordReset } from './components/PasswordReset';

export { useAuth } from './hooks/useAuth';
export { useAuthGuard } from './hooks/useAuthGuard';

export { authApi } from './api';
export { authUtils } from './utils';

export type { User, AuthState, LoginCredentials } from './types';

// features/auth/hooks/useAuth.ts
import { useState, useEffect, createContext, useContext } from 'react';
import { authApi } from '../api';
import type { User, AuthState } from '../types';

const AuthContext = createContext<AuthState | null>(null);

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const initAuth = async () => {
      try {
        const userData = await authApi.getCurrentUser();
        setUser(userData);
      } catch (error) {
        console.error('Auth initialization failed:', error);
      } finally {
        setLoading(false);
      }
    };

    initAuth();
  }, []);

  const login = async (credentials: LoginCredentials) => {
    const userData = await authApi.login(credentials);
    setUser(userData);
    return userData;
  };

  const logout = async () => {
    await authApi.logout();
    setUser(null);
  };

  const value: AuthState = {
    user,
    loading,
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

// ============= API MODULE ORGANIZATION =============
// lib/api/base.ts
class ApiClient {
  private baseUrl: string;
  private headers: Record<string, string>;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
    this.headers = {
      'Content-Type': 'application/json'
    };
  }

  setAuthToken(token: string) {
    this.headers.Authorization = `Bearer ${token}`;
  }

  async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    const config: RequestInit = {
      ...options,
      headers: {
        ...this.headers,
        ...options.headers
      }
    };

    const response = await fetch(url, config);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return response.json();
  }

  get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint);
  }

  post<T>(endpoint: string, data: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  put<T>(endpoint: string, data: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'DELETE'
    });
  }
}

export const apiClient = new ApiClient(process.env.NEXT_PUBLIC_API_URL || '/api');

// lib/api/users.ts
import { apiClient } from './base';
import type { User, CreateUserData, UpdateUserData } from '../../types';

export const usersApi = {
  getAll: (): Promise<User[]> => 
    apiClient.get('/users'),

  getById: (id: string): Promise<User> => 
    apiClient.get(`/users/${id}`),

  create: (data: CreateUserData): Promise<User> => 
    apiClient.post('/users', data),

  update: (id: string, data: UpdateUserData): Promise<User> => 
    apiClient.put(`/users/${id}`, data),

  delete: (id: string): Promise<void> => 
    apiClient.delete(`/users/${id}`)
};

// lib/api/index.ts
export { apiClient } from './base';
export { usersApi } from './users';
export { authApi } from './auth';
export { postsApi } from './posts';

// Export types
export type { ApiResponse, ApiError } from './base';
```

### **⚡ Code Splitting Strategies**

```typescript
// ============= ROUTE-BASED CODE SPLITTING =============
// pages/admin/index.tsx
import { GetServerSideProps } from 'next';
import dynamic from 'next/dynamic';
import { useAuth } from '../../features/auth';

// Lazy load admin components
const AdminDashboard = dynamic(() => import('../../features/admin/Dashboard'), {
  loading: () => <div>Loading admin dashboard...</div>
});

const UserManagement = dynamic(() => import('../../features/admin/UserManagement'));
const SystemSettings = dynamic(() => import('../../features/admin/SystemSettings'));

const AdminPage = () => {
  const { user, loading } = useAuth();

  if (loading) return <div>Loading...</div>;

  if (!user || user.role !== 'admin') {
    return <div>Access denied</div>;
  }

  return (
    <div>
      <h1>Admin Panel</h1>
      <AdminDashboard />
      <UserManagement />
      <SystemSettings />
    </div>
  );
};

export const getServerSideProps: GetServerSideProps = async (context) => {
  // Server-side auth check
  const token = context.req.cookies.authToken;
  
  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false
      }
    };
  }

  return { props: {} };
};

export default AdminPage;

// ============= COMPONENT-BASED CODE SPLITTING =============
// components/FeatureToggle.tsx
import { useState } from 'react';
import dynamic from 'next/dynamic';

interface FeatureToggleProps {
  features: string[];
  onFeatureLoad?: (feature: string) => void;
}

const FeatureToggle: React.FC<FeatureToggleProps> = ({ features, onFeatureLoad }) => {
  const [loadedFeatures, setLoadedFeatures] = useState<Set<string>>(new Set());
  const [Components, setComponents] = useState<Map<string, React.ComponentType>>(new Map());

  const loadFeature = async (featureName: string) => {
    if (loadedFeatures.has(featureName)) {
      return;
    }

    try {
      let component;
      
      switch (featureName) {
        case 'charts':
          component = await import('../features/Charts').then(m => m.default);
          break;
        case 'analytics':
          component = await import('../features/Analytics').then(m => m.default);
          break;
        case 'reporting':
          component = await import('../features/Reporting').then(m => m.default);
          break;
        default:
          throw new Error(`Unknown feature: ${featureName}`);
      }

      setComponents(prev => new Map(prev).set(featureName, component));
      setLoadedFeatures(prev => new Set(prev).add(featureName));
      onFeatureLoad?.(featureName);

    } catch (error) {
      console.error(`Failed to load feature ${featureName}:`, error);
    }
  };

  return (
    <div>
      <div className="feature-buttons">
        {features.map(feature => (
          <button
            key={feature}
            onClick={() => loadFeature(feature)}
            disabled={loadedFeatures.has(feature)}
          >
            {loadedFeatures.has(feature) ? `${feature} (Loaded)` : `Load ${feature}`}
          </button>
        ))}
      </div>

      <div className="feature-content">
        {Array.from(Components.entries()).map(([featureName, Component]) => (
          <div key={featureName} className="feature-section">
            <h3>{featureName}</h3>
            <Component />
          </div>
        ))}
      </div>
    </div>
  );
};

export default FeatureToggle;

// ============= VENDOR CODE SPLITTING =============
// next.config.js
module.exports = {
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    // Split vendor libraries
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          default: false,
          vendors: false,
          // Vendor chunk
          vendor: {
            name: 'vendor',
            chunks: 'all',
            test: /node_modules/,
            priority: 20
          },
          // Common chunks
          common: {
            name: 'common',
            minChunks: 2,
            chunks: 'all',
            priority: 10,
            reuseExistingChunk: true,
            enforce: true
          },
          // React chunk
          react: {
            name: 'react',
            chunks: 'all',
            test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
            priority: 30
          },
          // UI library chunk
          ui: {
            name: 'ui',
            chunks: 'all',
            test: /[\\/]node_modules[\\/](@mui|antd|chakra-ui)[\\/]/,
            priority: 25
          }
        }
      };
    }

    return config;
  }
};

// ============= PREFETCHING STRATEGIES =============
// hooks/usePrefetch.ts
import { useEffect, useRef } from 'react';
import { useRouter } from 'next/router';

interface PrefetchOptions {
  routes?: string[];
  components?: (() => Promise<any>)[];
  onIdle?: boolean;
  onHover?: boolean;
}

export const usePrefetch = (options: PrefetchOptions = {}) => {
  const router = useRouter();
  const prefetchedRef = useRef(new Set<string>());

  const prefetchRoute = (route: string) => {
    if (!prefetchedRef.current.has(route)) {
      router.prefetch(route);
      prefetchedRef.current.add(route);
    }
  };

  const prefetchComponent = async (componentLoader: () => Promise<any>) => {
    try {
      await componentLoader();
    } catch (error) {
      console.error('Component prefetch failed:', error);
    }
  };

  useEffect(() => {
    if (options.onIdle && 'requestIdleCallback' in window) {
      const prefetchOnIdle = () => {
        options.routes?.forEach(prefetchRoute);
        options.components?.forEach(prefetchComponent);
      };

      requestIdleCallback(prefetchOnIdle);
    } else {
      // Fallback for browsers without requestIdleCallback
      const timer = setTimeout(() => {
        options.routes?.forEach(prefetchRoute);
        options.components?.forEach(prefetchComponent);
      }, 100);

      return () => clearTimeout(timer);
    }
  }, [options.routes, options.components, options.onIdle]);

  const handleMouseEnter = (route: string) => {
    if (options.onHover) {
      prefetchRoute(route);
    }
  };

  return { prefetchRoute, handleMouseEnter };
};

// Usage
const Navigation = () => {
  const { handleMouseEnter } = usePrefetch({
    routes: ['/dashboard', '/profile', '/settings'],
    onIdle: true,
    onHover: true
  });

  return (
    <nav>
      <Link href="/dashboard">
        <a onMouseEnter={() => handleMouseEnter('/dashboard')}>
          Dashboard
        </a>
      </Link>
      <Link href="/profile">
        <a onMouseEnter={() => handleMouseEnter('/profile')}>
          Profile
        </a>
      </Link>
    </nav>
  );
};
```

## 🎯 แบบฝึกหัด

### **📦 แบบฝึกหัดที่ 1: Module Organization**
สร้าง module system:

```javascript
// TODO: สร้าง module organization

// 1. สร้าง plugin system
class PluginManager {
  // Implement plugin loading, unloading
  // Support hooks and middleware
}

// 2. สร้าง module registry
class ModuleRegistry {
  // Register modules with dependencies
  // Support version management
}

// 3. สร้าง lazy loading system
class LazyLoader {
  // Load modules on demand
  // Support preloading and caching
}

// 4. สร้าง barrel exports
// Organize exports efficiently
```

### **⚡ แบบฝึกหัดที่ 2: Dynamic Loading**
สร้าง dynamic import patterns:

```javascript
// TODO: สร้าง dynamic loading patterns

// 1. Feature flag system
class FeatureFlags {
  // Load features based on flags
  // Support A/B testing
}

// 2. Route-based splitting
// Split code by routes
// Preload critical routes

// 3. Component lazy loading
// Load components on interaction
// Support fallback UI

// 4. Vendor code optimization
// Split vendor libraries
// Optimize bundle sizes
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง Next.js module system:

```tsx
// TODO: สร้าง Next.js module system

// 1. Feature-based organization
// Organize by features not file types
// Include components, hooks, types

// 2. Smart code splitting
// Implement intelligent splitting
// Monitor and optimize bundles

// 3. Module federation
// Share modules between apps
// Support micro-frontends

// 4. Performance monitoring
// Track module load times
// Optimize critical paths
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 9: Promises & Async/Await](./09-promises-async.md)

**บทถัดไป**: [บทที่ 11: Iterators & Generators](./11-iterators-generators.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: ES6 Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [Next.js: Dynamic Imports](https://nextjs.org/docs/advanced-features/dynamic-import)
- [Webpack: Code Splitting](https://webpack.js.org/guides/code-splitting/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **ES6 Module System** และ import/export patterns
- **Dynamic Imports** และ code splitting
- **Module organization** และ architecture patterns
- **การใช้งานใน Next.js** สำหรับ optimal loading
- **Performance optimization** ด้วย module strategies

ES6 Modules เป็นพื้นฐานสำคัญของ modern JavaScript ที่ช่วยในการจัดระเบียบโค้ดและ performance optimization! 🚀
