# บทที่ 21: Design Patterns

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Gang of Four Design Patterns และการประยุกต์ใช้
- ทำความเข้าใจ Architectural Patterns สำหรับ scalable applications
- เรียนรู้ React และ Next.js specific patterns
- สร้าง maintainable และ extensible codebase

## 🏗️ Creational Patterns

### **🏭 Factory และ Abstract Factory**

```javascript
// ============= FACTORY PATTERN =============
// lib/patterns/factory.js

// Base Logger interface
class Logger {
  log(message) {
    throw new Error('log method must be implemented');
  }
}

// Console Logger implementation
class ConsoleLogger extends Logger {
  constructor(options = {}) {
    super();
    this.prefix = options.prefix || '[CONSOLE]';
    this.colorize = options.colorize !== false;
  }
  
  log(message, level = 'info') {
    const timestamp = new Date().toISOString();
    const logMessage = `${timestamp} ${this.prefix} [${level.toUpperCase()}] ${message}`;
    
    if (this.colorize && typeof window !== 'undefined') {
      const colors = {
        info: '#2196F3',
        warn: '#FF9800',
        error: '#F44336',
        debug: '#4CAF50'
      };
      
      console.log(
        `%c${logMessage}`,
        `color: ${colors[level] || colors.info}`
      );
    } else {
      console.log(logMessage);
    }
  }
}

// File Logger implementation (for Node.js)
class FileLogger extends Logger {
  constructor(options = {}) {
    super();
    this.filePath = options.filePath || 'app.log';
    this.maxSize = options.maxSize || 10 * 1024 * 1024; // 10MB
    this.format = options.format || 'json';
  }
  
  async log(message, level = 'info') {
    if (typeof window !== 'undefined') {
      throw new Error('FileLogger is not supported in browser environment');
    }
    
    const fs = await import('fs/promises');
    const path = await import('path');
    
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      pid: process.pid
    };
    
    const logLine = this.format === 'json' 
      ? JSON.stringify(logEntry) + '\n'
      : `${logEntry.timestamp} [${logEntry.level.toUpperCase()}] ${logEntry.message}\n`;
    
    try {
      // Check file size and rotate if necessary
      await this.rotateIfNecessary();
      
      await fs.appendFile(this.filePath, logLine);
    } catch (error) {
      console.error('Failed to write to log file:', error);
    }
  }
  
  async rotateIfNecessary() {
    const fs = await import('fs/promises');
    
    try {
      const stats = await fs.stat(this.filePath);
      if (stats.size > this.maxSize) {
        const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
        const rotatedPath = `${this.filePath}.${timestamp}`;
        await fs.rename(this.filePath, rotatedPath);
      }
    } catch (error) {
      // File doesn't exist yet, which is fine
    }
  }
}

// Remote Logger implementation
class RemoteLogger extends Logger {
  constructor(options = {}) {
    super();
    this.endpoint = options.endpoint || '/api/logs';
    this.apiKey = options.apiKey;
    this.batchSize = options.batchSize || 10;
    this.flushInterval = options.flushInterval || 5000;
    this.buffer = [];
    this.flushTimer = null;
    
    this.startAutoFlush();
  }
  
  async log(message, level = 'info') {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      userAgent: typeof navigator !== 'undefined' ? navigator.userAgent : null,
      url: typeof window !== 'undefined' ? window.location.href : null
    };
    
    this.buffer.push(logEntry);
    
    if (this.buffer.length >= this.batchSize) {
      await this.flush();
    }
  }
  
  async flush() {
    if (this.buffer.length === 0) return;
    
    const logs = [...this.buffer];
    this.buffer = [];
    
    try {
      const headers = {
        'Content-Type': 'application/json'
      };
      
      if (this.apiKey) {
        headers['Authorization'] = `Bearer ${this.apiKey}`;
      }
      
      await fetch(this.endpoint, {
        method: 'POST',
        headers,
        body: JSON.stringify({ logs })
      });
    } catch (error) {
      console.error('Failed to send logs to remote server:', error);
      // Put logs back in buffer for retry
      this.buffer.unshift(...logs);
    }
  }
  
  startAutoFlush() {
    this.flushTimer = setInterval(() => {
      this.flush();
    }, this.flushInterval);
  }
  
  stopAutoFlush() {
    if (this.flushTimer) {
      clearInterval(this.flushTimer);
      this.flushTimer = null;
    }
  }
  
  destroy() {
    this.stopAutoFlush();
    return this.flush(); // Final flush
  }
}

// Logger Factory
export class LoggerFactory {
  static loggers = new Map();
  
  static createLogger(type, options = {}) {
    const key = `${type}-${JSON.stringify(options)}`;
    
    if (this.loggers.has(key)) {
      return this.loggers.get(key);
    }
    
    let logger;
    
    switch (type.toLowerCase()) {
      case 'console':
        logger = new ConsoleLogger(options);
        break;
      case 'file':
        logger = new FileLogger(options);
        break;
      case 'remote':
        logger = new RemoteLogger(options);
        break;
      default:
        throw new Error(`Unknown logger type: ${type}`);
    }
    
    this.loggers.set(key, logger);
    return logger;
  }
  
  static createCompositeLogger(configs) {
    const loggers = configs.map(config => 
      this.createLogger(config.type, config.options)
    );
    
    return new CompositeLogger(loggers);
  }
  
  static clearCache() {
    // Clean up resources
    for (const logger of this.loggers.values()) {
      if (logger.destroy) {
        logger.destroy();
      }
    }
    this.loggers.clear();
  }
}

// Composite Logger for multiple outputs
class CompositeLogger extends Logger {
  constructor(loggers) {
    super();
    this.loggers = loggers;
  }
  
  async log(message, level = 'info') {
    const promises = this.loggers.map(logger => 
      logger.log(message, level).catch(error => {
        console.error('Logger error:', error);
      })
    );
    
    await Promise.allSettled(promises);
  }
  
  destroy() {
    const promises = this.loggers
      .filter(logger => logger.destroy)
      .map(logger => logger.destroy());
    
    return Promise.allSettled(promises);
  }
}

// ============= ABSTRACT FACTORY PATTERN =============
// lib/patterns/abstract-factory.js

// Abstract theme factory
export class ThemeFactory {
  createButton() {
    throw new Error('createButton must be implemented');
  }
  
  createCard() {
    throw new Error('createCard must be implemented');
  }
  
  createModal() {
    throw new Error('createModal must be implemented');
  }
}

// Light theme components
class LightButton {
  render(props) {
    return {
      ...props,
      className: `${props.className || ''} bg-white text-gray-900 border border-gray-300 hover:bg-gray-50`,
      style: {
        ...props.style,
        backgroundColor: '#ffffff',
        color: '#111827',
        border: '1px solid #d1d5db'
      }
    };
  }
}

class LightCard {
  render(props) {
    return {
      ...props,
      className: `${props.className || ''} bg-white shadow-md border border-gray-200`,
      style: {
        ...props.style,
        backgroundColor: '#ffffff',
        boxShadow: '0 4px 6px -1px rgba(0, 0, 0, 0.1)',
        border: '1px solid #e5e7eb'
      }
    };
  }
}

class LightModal {
  render(props) {
    return {
      ...props,
      className: `${props.className || ''} bg-white`,
      overlayStyle: {
        backgroundColor: 'rgba(0, 0, 0, 0.5)'
      },
      contentStyle: {
        backgroundColor: '#ffffff',
        color: '#111827'
      }
    };
  }
}

// Dark theme components
class DarkButton {
  render(props) {
    return {
      ...props,
      className: `${props.className || ''} bg-gray-800 text-white border border-gray-600 hover:bg-gray-700`,
      style: {
        ...props.style,
        backgroundColor: '#1f2937',
        color: '#ffffff',
        border: '1px solid #4b5563'
      }
    };
  }
}

class DarkCard {
  render(props) {
    return {
      ...props,
      className: `${props.className || ''} bg-gray-800 shadow-lg border border-gray-700`,
      style: {
        ...props.style,
        backgroundColor: '#1f2937',
        boxShadow: '0 10px 15px -3px rgba(0, 0, 0, 0.3)',
        border: '1px solid #374151'
      }
    };
  }
}

class DarkModal {
  render(props) {
    return {
      ...props,
      className: `${props.className || ''} bg-gray-800`,
      overlayStyle: {
        backgroundColor: 'rgba(0, 0, 0, 0.75)'
      },
      contentStyle: {
        backgroundColor: '#1f2937',
        color: '#ffffff'
      }
    };
  }
}

// Light theme factory
export class LightThemeFactory extends ThemeFactory {
  createButton() {
    return new LightButton();
  }
  
  createCard() {
    return new LightCard();
  }
  
  createModal() {
    return new LightModal();
  }
}

// Dark theme factory
export class DarkThemeFactory extends ThemeFactory {
  createButton() {
    return new DarkButton();
  }
  
  createCard() {
    return new DarkCard();
  }
  
  createModal() {
    return new DarkModal();
  }
}

// Theme factory selector
export class ThemeFactorySelector {
  static getFactory(theme) {
    switch (theme) {
      case 'light':
        return new LightThemeFactory();
      case 'dark':
        return new DarkThemeFactory();
      default:
        throw new Error(`Unknown theme: ${theme}`);
    }
  }
}
```

### **🔧 Builder Pattern**

```javascript
// ============= BUILDER PATTERN =============
// lib/patterns/builder.js

export class QueryBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.query = {
      select: [],
      from: null,
      joins: [],
      where: [],
      groupBy: [],
      having: [],
      orderBy: [],
      limit: null,
      offset: null
    };
    return this;
  }
  
  select(...fields) {
    if (fields.length === 1 && Array.isArray(fields[0])) {
      this.query.select = [...this.query.select, ...fields[0]];
    } else {
      this.query.select = [...this.query.select, ...fields];
    }
    return this;
  }
  
  from(table) {
    this.query.from = table;
    return this;
  }
  
  join(table, condition, type = 'INNER') {
    this.query.joins.push({
      type: type.toUpperCase(),
      table,
      condition
    });
    return this;
  }
  
  leftJoin(table, condition) {
    return this.join(table, condition, 'LEFT');
  }
  
  rightJoin(table, condition) {
    return this.join(table, condition, 'RIGHT');
  }
  
  where(condition, operator = 'AND') {
    this.query.where.push({
      condition,
      operator: operator.toUpperCase()
    });
    return this;
  }
  
  orWhere(condition) {
    return this.where(condition, 'OR');
  }
  
  whereIn(field, values) {
    const placeholders = values.map(() => '?').join(', ');
    return this.where(`${field} IN (${placeholders})`);
  }
  
  whereBetween(field, min, max) {
    return this.where(`${field} BETWEEN ? AND ?`);
  }
  
  groupBy(...fields) {
    this.query.groupBy = [...this.query.groupBy, ...fields];
    return this;
  }
  
  having(condition) {
    this.query.having.push(condition);
    return this;
  }
  
  orderBy(field, direction = 'ASC') {
    this.query.orderBy.push({
      field,
      direction: direction.toUpperCase()
    });
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
  
  build() {
    return this.buildSQL();
  }
  
  buildSQL() {
    let sql = '';
    
    // SELECT clause
    if (this.query.select.length === 0) {
      sql += 'SELECT *';
    } else {
      sql += `SELECT ${this.query.select.join(', ')}`;
    }
    
    // FROM clause
    if (!this.query.from) {
      throw new Error('FROM clause is required');
    }
    sql += ` FROM ${this.query.from}`;
    
    // JOIN clauses
    for (const join of this.query.joins) {
      sql += ` ${join.type} JOIN ${join.table} ON ${join.condition}`;
    }
    
    // WHERE clause
    if (this.query.where.length > 0) {
      const whereConditions = this.query.where.map((w, index) => {
        if (index === 0) {
          return w.condition;
        }
        return `${w.operator} ${w.condition}`;
      });
      sql += ` WHERE ${whereConditions.join(' ')}`;
    }
    
    // GROUP BY clause
    if (this.query.groupBy.length > 0) {
      sql += ` GROUP BY ${this.query.groupBy.join(', ')}`;
    }
    
    // HAVING clause
    if (this.query.having.length > 0) {
      sql += ` HAVING ${this.query.having.join(' AND ')}`;
    }
    
    // ORDER BY clause
    if (this.query.orderBy.length > 0) {
      const orderFields = this.query.orderBy.map(o => `${o.field} ${o.direction}`);
      sql += ` ORDER BY ${orderFields.join(', ')}`;
    }
    
    // LIMIT clause
    if (this.query.limit !== null) {
      sql += ` LIMIT ${this.query.limit}`;
    }
    
    // OFFSET clause
    if (this.query.offset !== null) {
      sql += ` OFFSET ${this.query.offset}`;
    }
    
    return sql.trim();
  }
  
  // Build for different database types
  buildForMongoDB() {
    const pipeline = [];
    
    // Match stage (WHERE equivalent)
    if (this.query.where.length > 0) {
      const matchConditions = this.buildMongoMatchConditions();
      pipeline.push({ $match: matchConditions });
    }
    
    // Lookup stages (JOIN equivalent)
    for (const join of this.query.joins) {
      pipeline.push(this.buildMongoLookup(join));
    }
    
    // Group stage
    if (this.query.groupBy.length > 0) {
      pipeline.push(this.buildMongoGroup());
    }
    
    // Sort stage (ORDER BY equivalent)
    if (this.query.orderBy.length > 0) {
      const sortObj = {};
      for (const order of this.query.orderBy) {
        sortObj[order.field] = order.direction === 'ASC' ? 1 : -1;
      }
      pipeline.push({ $sort: sortObj });
    }
    
    // Skip stage (OFFSET equivalent)
    if (this.query.offset !== null) {
      pipeline.push({ $skip: this.query.offset });
    }
    
    // Limit stage
    if (this.query.limit !== null) {
      pipeline.push({ $limit: this.query.limit });
    }
    
    // Project stage (SELECT equivalent)
    if (this.query.select.length > 0) {
      const projectObj = {};
      for (const field of this.query.select) {
        projectObj[field] = 1;
      }
      pipeline.push({ $project: projectObj });
    }
    
    return pipeline;
  }
  
  buildMongoMatchConditions() {
    // Simplified MongoDB condition builder
    const conditions = {};
    
    for (const whereClause of this.query.where) {
      // This is a simplified version
      // In a real implementation, you'd parse the SQL condition
      // and convert it to MongoDB query syntax
      const parts = whereClause.condition.split(/\s*(=|!=|>|<|>=|<=)\s*/);
      if (parts.length >= 3) {
        const field = parts[0];
        const operator = parts[1];
        const value = parts[2];
        
        switch (operator) {
          case '=':
            conditions[field] = value;
            break;
          case '!=':
            conditions[field] = { $ne: value };
            break;
          case '>':
            conditions[field] = { $gt: value };
            break;
          case '<':
            conditions[field] = { $lt: value };
            break;
          case '>=':
            conditions[field] = { $gte: value };
            break;
          case '<=':
            conditions[field] = { $lte: value };
            break;
        }
      }
    }
    
    return conditions;
  }
  
  buildMongoLookup(join) {
    // Simplified MongoDB lookup builder
    return {
      $lookup: {
        from: join.table,
        localField: '_id', // Simplified
        foreignField: 'parentId', // Simplified
        as: join.table
      }
    };
  }
  
  buildMongoGroup() {
    const groupObj = {
      _id: {}
    };
    
    for (const field of this.query.groupBy) {
      groupObj._id[field] = `$${field}`;
    }
    
    return { $group: groupObj };
  }
  
  clone() {
    const newBuilder = new QueryBuilder();
    newBuilder.query = JSON.parse(JSON.stringify(this.query));
    return newBuilder;
  }
}

// ============= API REQUEST BUILDER =============
// lib/patterns/api-builder.js

export class APIRequestBuilder {
  constructor(baseURL = '') {
    this.baseURL = baseURL;
    this.reset();
  }
  
  reset() {
    this.config = {
      url: '',
      method: 'GET',
      headers: {},
      params: {},
      data: null,
      timeout: 10000,
      retries: 0,
      retryDelay: 1000,
      cache: false,
      cacheTTL: 300000
    };
    return this;
  }
  
  url(path) {
    this.config.url = this.baseURL + path;
    return this;
  }
  
  method(method) {
    this.config.method = method.toUpperCase();
    return this;
  }
  
  get(path) {
    return this.url(path).method('GET');
  }
  
  post(path) {
    return this.url(path).method('POST');
  }
  
  put(path) {
    return this.url(path).method('PUT');
  }
  
  delete(path) {
    return this.url(path).method('DELETE');
  }
  
  patch(path) {
    return this.url(path).method('PATCH');
  }
  
  headers(headers) {
    this.config.headers = { ...this.config.headers, ...headers };
    return this;
  }
  
  header(key, value) {
    this.config.headers[key] = value;
    return this;
  }
  
  auth(token, type = 'Bearer') {
    return this.header('Authorization', `${type} ${token}`);
  }
  
  contentType(type) {
    return this.header('Content-Type', type);
  }
  
  json() {
    return this.contentType('application/json');
  }
  
  formData() {
    return this.contentType('multipart/form-data');
  }
  
  params(params) {
    this.config.params = { ...this.config.params, ...params };
    return this;
  }
  
  param(key, value) {
    this.config.params[key] = value;
    return this;
  }
  
  data(data) {
    this.config.data = data;
    return this;
  }
  
  timeout(ms) {
    this.config.timeout = ms;
    return this;
  }
  
  retries(count, delay = 1000) {
    this.config.retries = count;
    this.config.retryDelay = delay;
    return this;
  }
  
  cache(enable = true, ttl = 300000) {
    this.config.cache = enable;
    this.config.cacheTTL = ttl;
    return this;
  }
  
  build() {
    // Build URL with query parameters
    const url = new URL(this.config.url);
    for (const [key, value] of Object.entries(this.config.params)) {
      url.searchParams.set(key, value);
    }
    
    // Prepare fetch options
    const options = {
      method: this.config.method,
      headers: this.config.headers,
      signal: AbortSignal.timeout(this.config.timeout)
    };
    
    // Add body for POST/PUT/PATCH
    if (['POST', 'PUT', 'PATCH'].includes(this.config.method) && this.config.data) {
      if (this.config.headers['Content-Type']?.includes('application/json')) {
        options.body = JSON.stringify(this.config.data);
      } else {
        options.body = this.config.data;
      }
    }
    
    return {
      url: url.toString(),
      options,
      config: this.config
    };
  }
  
  async execute() {
    const { url, options, config } = this.build();
    
    // Check cache first
    if (config.cache && config.method === 'GET') {
      const cached = this.getCached(url);
      if (cached) {
        return cached;
      }
    }
    
    let lastError;
    
    for (let attempt = 0; attempt <= config.retries; attempt++) {
      try {
        const response = await fetch(url, options);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const result = await response.json();
        
        // Cache successful GET requests
        if (config.cache && config.method === 'GET') {
          this.setCached(url, result, config.cacheTTL);
        }
        
        return result;
      } catch (error) {
        lastError = error;
        
        if (attempt < config.retries) {
          await this.delay(config.retryDelay * Math.pow(2, attempt));
        }
      }
    }
    
    throw lastError;
  }
  
  getCached(key) {
    const cached = localStorage.getItem(`api_cache_${key}`);
    if (cached) {
      const data = JSON.parse(cached);
      if (Date.now() - data.timestamp < data.ttl) {
        return data.value;
      }
      localStorage.removeItem(`api_cache_${key}`);
    }
    return null;
  }
  
  setCached(key, value, ttl) {
    const data = {
      value,
      timestamp: Date.now(),
      ttl
    };
    localStorage.setItem(`api_cache_${key}`, JSON.stringify(data));
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  clone() {
    const newBuilder = new APIRequestBuilder(this.baseURL);
    newBuilder.config = JSON.parse(JSON.stringify(this.config));
    return newBuilder;
  }
}
```

## 🏛️ Structural Patterns

### **🎭 Adapter และ Facade**

```javascript
// ============= ADAPTER PATTERN =============
// lib/patterns/adapter.js

// Legacy API interface
class LegacyUserAPI {
  getUserData(userId) {
    // Legacy format
    return {
      user_id: userId,
      user_name: 'John Doe',
      user_email: 'john@example.com',
      user_status: 'active',
      registration_date: '2023-01-15T10:30:00Z'
    };
  }
  
  updateUserData(userId, data) {
    return {
      success: true,
      user_id: userId,
      updated_fields: Object.keys(data)
    };
  }
}

// Modern API interface
class ModernUserAPI {
  constructor() {
    this.baseURL = '/api/v2/users';
  }
  
  async getUser(id) {
    const response = await fetch(`${this.baseURL}/${id}`);
    return response.json();
  }
  
  async updateUser(id, data) {
    const response = await fetch(`${this.baseURL}/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

// Adapter to make legacy API work with modern interface
export class UserAPIAdapter {
  constructor(legacyAPI) {
    this.legacyAPI = legacyAPI;
  }
  
  async getUser(id) {
    const legacyData = this.legacyAPI.getUserData(id);
    
    // Transform legacy format to modern format
    return {
      id: legacyData.user_id,
      name: legacyData.user_name,
      email: legacyData.user_email,
      status: legacyData.user_status,
      createdAt: legacyData.registration_date,
      updatedAt: new Date().toISOString()
    };
  }
  
  async updateUser(id, data) {
    // Transform modern format to legacy format
    const legacyData = {};
    
    if (data.name) legacyData.user_name = data.name;
    if (data.email) legacyData.user_email = data.email;
    if (data.status) legacyData.user_status = data.status;
    
    const result = this.legacyAPI.updateUserData(id, legacyData);
    
    // Transform response back to modern format
    return {
      id: result.user_id,
      success: result.success,
      modifiedFields: result.updated_fields
    };
  }
}

// ============= FACADE PATTERN =============
// lib/patterns/facade.js

// Complex subsystems
class AuthenticationService {
  async login(credentials) {
    // Complex authentication logic
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    
    if (!response.ok) {
      throw new Error('Authentication failed');
    }
    
    return response.json();
  }
  
  async logout() {
    const response = await fetch('/api/auth/logout', {
      method: 'POST'
    });
    
    return response.ok;
  }
  
  async refreshToken(token) {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${token}` }
    });
    
    return response.json();
  }
}

class SessionManager {
  setToken(token) {
    localStorage.setItem('auth_token', token);
    this.scheduleTokenRefresh(token);
  }
  
  getToken() {
    return localStorage.getItem('auth_token');
  }
  
  clearToken() {
    localStorage.removeItem('auth_token');
    this.clearTokenRefresh();
  }
  
  scheduleTokenRefresh(token) {
    // Decode JWT to get expiration
    const payload = JSON.parse(atob(token.split('.')[1]));
    const expirationTime = payload.exp * 1000;
    const refreshTime = expirationTime - Date.now() - 60000; // 1 minute before
    
    if (refreshTime > 0) {
      this.refreshTimer = setTimeout(() => {
        this.refreshToken();
      }, refreshTime);
    }
  }
  
  clearTokenRefresh() {
    if (this.refreshTimer) {
      clearTimeout(this.refreshTimer);
    }
  }
  
  async refreshToken() {
    const authService = new AuthenticationService();
    const currentToken = this.getToken();
    
    try {
      const response = await authService.refreshToken(currentToken);
      this.setToken(response.token);
    } catch (error) {
      this.clearToken();
      window.location.href = '/login';
    }
  }
}

class PermissionManager {
  constructor() {
    this.permissions = new Set();
  }
  
  setPermissions(permissions) {
    this.permissions = new Set(permissions);
  }
  
  hasPermission(permission) {
    return this.permissions.has(permission);
  }
  
  hasAnyPermission(permissions) {
    return permissions.some(permission => this.permissions.has(permission));
  }
  
  hasAllPermissions(permissions) {
    return permissions.every(permission => this.permissions.has(permission));
  }
  
  clearPermissions() {
    this.permissions.clear();
  }
}

class UserProfileManager {
  constructor() {
    this.profile = null;
  }
  
  setProfile(profile) {
    this.profile = profile;
  }
  
  getProfile() {
    return this.profile;
  }
  
  updateProfile(updates) {
    if (this.profile) {
      this.profile = { ...this.profile, ...updates };
    }
  }
  
  clearProfile() {
    this.profile = null;
  }
}

// Facade that simplifies complex authentication system
export class AuthFacade {
  constructor() {
    this.authService = new AuthenticationService();
    this.sessionManager = new SessionManager();
    this.permissionManager = new PermissionManager();
    this.profileManager = new UserProfileManager();
    this.isAuthenticated = false;
  }
  
  async login(credentials) {
    try {
      const response = await this.authService.login(credentials);
      
      // Set up session
      this.sessionManager.setToken(response.token);
      
      // Set up permissions
      this.permissionManager.setPermissions(response.user.permissions);
      
      // Set up profile
      this.profileManager.setProfile(response.user);
      
      this.isAuthenticated = true;
      
      return {
        success: true,
        user: response.user
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
  
  async logout() {
    try {
      await this.authService.logout();
    } catch (error) {
      console.warn('Logout failed on server:', error);
    }
    
    // Clean up local state
    this.sessionManager.clearToken();
    this.permissionManager.clearPermissions();
    this.profileManager.clearProfile();
    this.isAuthenticated = false;
    
    return { success: true };
  }
  
  getAuthState() {
    return {
      isAuthenticated: this.isAuthenticated,
      user: this.profileManager.getProfile(),
      token: this.sessionManager.getToken()
    };
  }
  
  hasPermission(permission) {
    return this.isAuthenticated && this.permissionManager.hasPermission(permission);
  }
  
  canAccess(permissions) {
    if (!this.isAuthenticated) return false;
    
    if (Array.isArray(permissions)) {
      return this.permissionManager.hasAnyPermission(permissions);
    }
    
    return this.permissionManager.hasPermission(permissions);
  }
  
  requiresPermission(permission) {
    if (!this.hasPermission(permission)) {
      throw new Error(`Access denied. Required permission: ${permission}`);
    }
  }
  
  async updateProfile(updates) {
    if (!this.isAuthenticated) {
      throw new Error('User not authenticated');
    }
    
    try {
      const response = await fetch('/api/user/profile', {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.sessionManager.getToken()}`
        },
        body: JSON.stringify(updates)
      });
      
      if (!response.ok) {
        throw new Error('Profile update failed');
      }
      
      const updatedProfile = await response.json();
      this.profileManager.setProfile(updatedProfile);
      
      return { success: true, profile: updatedProfile };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }
  
  // Initialize authentication state on app start
  async initialize() {
    const token = this.sessionManager.getToken();
    
    if (token) {
      try {
        // Verify token and get user data
        const response = await fetch('/api/auth/verify', {
          headers: { 'Authorization': `Bearer ${token}` }
        });
        
        if (response.ok) {
          const userData = await response.json();
          this.permissionManager.setPermissions(userData.permissions);
          this.profileManager.setProfile(userData);
          this.isAuthenticated = true;
        } else {
          this.sessionManager.clearToken();
        }
      } catch (error) {
        this.sessionManager.clearToken();
      }
    }
    
    return this.getAuthState();
  }
}
```

## 🎯 แบบฝึกหัด

### **🏭 แบบฝึกหัดที่ 1: Factory Patterns**
สร้าง comprehensive factory system:

```javascript
// TODO: สร้าง factory system

// 1. Component factory
class ComponentFactory {
  // Dynamic component creation
  // Theme-based rendering
  // Lazy loading integration
}

// 2. Service factory
class ServiceFactory {
  // Service registration
  // Dependency injection
  // Lifecycle management
}

// 3. Configuration factory
class ConfigFactory {
  // Environment-based configs
  // Feature flags
  // Dynamic configuration
}
```

### **🔧 แบบฝึกหัดที่ 2: Builder Patterns**
สร้าง advanced builder system:

```javascript
// TODO: สร้าง builder system

// 1. Form builder
class FormBuilder {
  // Dynamic form creation
  // Validation rules
  // Field dependencies
}

// 2. Chart builder
class ChartBuilder {
  // Data visualization
  // Interactive charts
  // Export capabilities
}

// 3. Email builder
class EmailBuilder {
  // Template system
  // Dynamic content
  // Multi-language support
}
```

### **🏛️ แบบฝึกหัดที่ 3: Structural Patterns**
สร้าง comprehensive structural system:

```javascript
// TODO: สร้าง structural system

// 1. Plugin adapter
class PluginAdapter {
  // Plugin integration
  // API normalization
  // Event handling
}

// 2. Data facade
class DataFacade {
  // Multiple data sources
  // Caching layer
  // Transformation pipeline
}

// 3. Component proxy
class ComponentProxy {
  // Lazy loading
  // Performance monitoring
  // Error boundaries
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 20: Networking](./20-networking.md)

**บทถัดไป**: [บทที่ 22: Architecture](./22-architecture.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)
- [JavaScript Design Patterns](https://www.patterns.dev/posts/classic-design-patterns/)
- [React Patterns](https://reactpatterns.com/)
- [Architectural Patterns](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Creational Patterns** - Factory, Abstract Factory, และ Builder patterns
- **Structural Patterns** - Adapter, Facade, และ Proxy patterns
- **Implementation Techniques** - practical applications ใน Next.js projects
- **Code Organization** - maintainable และ extensible architecture
- **Best Practices** - pattern selection และ proper implementation

Design Patterns เป็นเครื่องมือสำคัญสำหรับ scalable applications! 🚀
