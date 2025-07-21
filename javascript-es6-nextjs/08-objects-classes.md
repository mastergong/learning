# บทที่ 8: Objects & Classes

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Modern Object Features ใน ES6+
- ทำความเข้าใจ ES6 Classes และ Inheritance
- เรียนรู้ Object Methods และ Property Handling
- ประยุกต์ใช้ใน Next.js สำหรับ Object-Oriented Programming

## 🏗️ Modern Object Features

### **🔧 Object Property Enhancements**

```javascript
// ============= SHORTHAND PROPERTY NAMES =============
const name = 'John';
const age = 30;
const city = 'Bangkok';

// ES5 way
var userOld = {
  name: name,
  age: age,
  city: city
};

// ES6+ shorthand
const user = { name, age, city };

// ============= COMPUTED PROPERTY NAMES =============
const fieldName = 'email';
const fieldValue = 'john@example.com';

// Dynamic property names
const userProfile = {
  [fieldName]: fieldValue,
  [`${fieldName}Verified`]: true,
  [`last${fieldName.charAt(0).toUpperCase() + fieldName.slice(1)}Update`]: new Date()
};

console.log(userProfile);
// { 
//   email: 'john@example.com',
//   emailVerified: true,
//   lastEmailUpdate: 2024-01-01T00:00:00.000Z
// }

// Advanced computed properties
const createPropertyObject = (properties) => {
  return properties.reduce((obj, prop) => {
    obj[prop.key] = prop.value;
    obj[`${prop.key}Type`] = typeof prop.value;
    obj[`${prop.key}Timestamp`] = Date.now();
    return obj;
  }, {});
};

const dynamicObject = createPropertyObject([
  { key: 'username', value: 'john_doe' },
  { key: 'score', value: 95 },
  { key: 'active', value: true }
]);

// ============= METHOD DEFINITIONS =============
const calculator = {
  // Method shorthand
  add(a, b) {
    return a + b;
  },
  
  subtract(a, b) {
    return a - b;
  },
  
  // Computed method names
  [`${'multi'}ply`](a, b) {
    return a * b;
  },
  
  // Generator methods
  *fibonacci(n) {
    let [a, b] = [0, 1];
    for (let i = 0; i < n; i++) {
      yield a;
      [a, b] = [b, a + b];
    }
  },
  
  // Async methods
  async fetchData(url) {
    try {
      const response = await fetch(url);
      return await response.json();
    } catch (error) {
      console.error('Fetch error:', error);
      throw error;
    }
  }
};

// Usage
console.log(calculator.add(5, 3)); // 8
console.log([...calculator.fibonacci(5)]); // [0, 1, 1, 2, 3]

// ============= PROPERTY DESCRIPTORS =============
const secureObject = {};

// Define properties with descriptors
Object.defineProperty(secureObject, 'id', {
  value: Math.random().toString(36).substr(2, 9),
  writable: false,    // Cannot be changed
  enumerable: true,   // Shows in for...in loop
  configurable: false // Cannot be deleted or reconfigured
});

Object.defineProperty(secureObject, 'createdAt', {
  value: new Date(),
  writable: false,
  enumerable: true,
  configurable: false
});

// Define getter/setter
Object.defineProperty(secureObject, 'data', {
  get() {
    return this._data || {};
  },
  set(value) {
    if (typeof value === 'object' && value !== null) {
      this._data = { ...value, lastModified: new Date() };
    } else {
      throw new Error('Data must be an object');
    }
  },
  enumerable: true,
  configurable: true
});

// Multiple properties at once
Object.defineProperties(secureObject, {
  version: {
    value: '1.0.0',
    writable: false,
    enumerable: true
  },
  status: {
    value: 'active',
    writable: true,
    enumerable: true
  }
});

// ============= OBJECT METHODS ES6+ =============
const sourceObj1 = { a: 1, b: 2 };
const sourceObj2 = { c: 3, d: 4 };
const sourceObj3 = { a: 10, e: 5 }; // 'a' will override

// Object.assign - shallow merge
const merged = Object.assign({}, sourceObj1, sourceObj2, sourceObj3);
console.log(merged); // { a: 10, b: 2, c: 3, d: 4, e: 5 }

// Object.keys, values, entries
const sampleObject = { name: 'John', age: 30, city: 'Bangkok' };

console.log(Object.keys(sampleObject));    // ['name', 'age', 'city']
console.log(Object.values(sampleObject));  // ['John', 30, 'Bangkok']
console.log(Object.entries(sampleObject)); // [['name', 'John'], ['age', 30], ['city', 'Bangkok']]

// Object.fromEntries (ES2019)
const entries = [['name', 'Jane'], ['age', 25], ['city', 'Tokyo']];
const objectFromEntries = Object.fromEntries(entries);
console.log(objectFromEntries); // { name: 'Jane', age: 25, city: 'Tokyo' }

// Transform object using entries
const transformObject = (obj, transformer) => {
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => transformer(key, value))
  );
};

const uppercaseKeys = transformObject(sampleObject, (key, value) => [
  key.toUpperCase(),
  value
]);

const doubleNumbers = transformObject(sampleObject, (key, value) => [
  key,
  typeof value === 'number' ? value * 2 : value
]);
```

### **🏛️ Object Patterns และ Utilities**

```javascript
// ============= DEEP OBJECT OPERATIONS =============
class ObjectUtils {
  // Deep clone (handles nested objects, arrays, dates)
  static deepClone(obj) {
    if (obj === null || typeof obj !== 'object') {
      return obj;
    }
    
    if (obj instanceof Date) {
      return new Date(obj.getTime());
    }
    
    if (obj instanceof Array) {
      return obj.map(item => this.deepClone(item));
    }
    
    if (typeof obj === 'object') {
      const cloned = {};
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          cloned[key] = this.deepClone(obj[key]);
        }
      }
      return cloned;
    }
  }

  // Deep merge objects
  static deepMerge(...objects) {
    const isObject = (obj) => obj && typeof obj === 'object' && !Array.isArray(obj);
    
    return objects.reduce((merged, obj) => {
      Object.keys(obj).forEach(key => {
        if (isObject(obj[key]) && isObject(merged[key])) {
          merged[key] = this.deepMerge(merged[key], obj[key]);
        } else {
          merged[key] = obj[key];
        }
      });
      return merged;
    }, {});
  }

  // Get nested property safely
  static getNestedProperty(obj, path, defaultValue = undefined) {
    const keys = path.split('.');
    let current = obj;
    
    for (const key of keys) {
      if (current === null || current === undefined || !(key in current)) {
        return defaultValue;
      }
      current = current[key];
    }
    
    return current;
  }

  // Set nested property
  static setNestedProperty(obj, path, value) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    let current = obj;
    
    for (const key of keys) {
      if (!(key in current) || typeof current[key] !== 'object') {
        current[key] = {};
      }
      current = current[key];
    }
    
    current[lastKey] = value;
    return obj;
  }

  // Remove undefined/null properties
  static clean(obj, removeNull = false) {
    const cleaned = {};
    
    for (const [key, value] of Object.entries(obj)) {
      if (value !== undefined && (!removeNull || value !== null)) {
        if (value && typeof value === 'object' && !Array.isArray(value)) {
          const cleanedNested = this.clean(value, removeNull);
          if (Object.keys(cleanedNested).length > 0) {
            cleaned[key] = cleanedNested;
          }
        } else {
          cleaned[key] = value;
        }
      }
    }
    
    return cleaned;
  }

  // Pick specific properties
  static pick(obj, keys) {
    return keys.reduce((picked, key) => {
      if (key in obj) {
        picked[key] = obj[key];
      }
      return picked;
    }, {});
  }

  // Omit specific properties
  static omit(obj, keys) {
    const omitted = { ...obj };
    keys.forEach(key => delete omitted[key]);
    return omitted;
  }

  // Flatten nested object
  static flatten(obj, prefix = '', separator = '.') {
    const flattened = {};
    
    for (const [key, value] of Object.entries(obj)) {
      const newKey = prefix ? `${prefix}${separator}${key}` : key;
      
      if (value && typeof value === 'object' && !Array.isArray(value)) {
        Object.assign(flattened, this.flatten(value, newKey, separator));
      } else {
        flattened[newKey] = value;
      }
    }
    
    return flattened;
  }

  // Unflatten object
  static unflatten(obj, separator = '.') {
    const unflattened = {};
    
    for (const [key, value] of Object.entries(obj)) {
      this.setNestedProperty(unflattened, key.replace(new RegExp(`\\${separator}`, 'g'), '.'), value);
    }
    
    return unflattened;
  }
}

// ============= USAGE EXAMPLES =============
const testObject = {
  name: 'John',
  address: {
    street: '123 Main St',
    city: 'Bangkok',
    coordinates: {
      lat: 13.7563,
      lng: 100.5018
    }
  },
  preferences: {
    theme: 'dark',
    language: 'en'
  },
  metadata: undefined,
  nullValue: null
};

// Deep clone
const cloned = ObjectUtils.deepClone(testObject);
cloned.address.city = 'Tokyo';
console.log('Original city:', testObject.address.city); // Bangkok
console.log('Cloned city:', cloned.address.city); // Tokyo

// Get nested property
const latitude = ObjectUtils.getNestedProperty(testObject, 'address.coordinates.lat');
console.log('Latitude:', latitude); // 13.7563

const nonExistent = ObjectUtils.getNestedProperty(testObject, 'address.zipcode', '00000');
console.log('Zipcode:', nonExistent); // 00000

// Set nested property
ObjectUtils.setNestedProperty(testObject, 'address.zipcode', '10110');
console.log('Updated object:', testObject);

// Clean object
const cleaned = ObjectUtils.clean(testObject, true);
console.log('Cleaned object:', cleaned);

// Pick properties
const addressOnly = ObjectUtils.pick(testObject, ['name', 'address']);
console.log('Address only:', addressOnly);

// Flatten
const flattened = ObjectUtils.flatten(testObject);
console.log('Flattened:', flattened);

// Unflatten
const unflattened = ObjectUtils.unflatten(flattened);
console.log('Unflattened:', unflattened);
```

## 🏛️ ES6+ Classes

### **🏗️ Class Basics และ Inheritance**

```javascript
// ============= BASIC CLASS DEFINITION =============
class User {
  // Class fields (ES2022)
  #id = Math.random().toString(36).substr(2, 9); // Private field
  
  static userCount = 0; // Static field
  
  constructor(name, email, role = 'user') {
    this.name = name;
    this.email = email;
    this.role = role;
    this.createdAt = new Date();
    this.isActive = true;
    
    User.userCount++; // Increment static counter
  }
  
  // Instance methods
  getProfile() {
    return {
      id: this.#id,
      name: this.name,
      email: this.email,
      role: this.role,
      createdAt: this.createdAt,
      isActive: this.isActive
    };
  }
  
  updateProfile(updates) {
    const allowedFields = ['name', 'email'];
    
    for (const [key, value] of Object.entries(updates)) {
      if (allowedFields.includes(key)) {
        this[key] = value;
      }
    }
    
    return this; // Method chaining
  }
  
  deactivate() {
    this.isActive = false;
    return this;
  }
  
  activate() {
    this.isActive = true;
    return this;
  }
  
  // Getter
  get displayName() {
    return `${this.name} (${this.role})`;
  }
  
  // Setter
  set emailAddress(email) {
    if (this.validateEmail(email)) {
      this.email = email;
    } else {
      throw new Error('Invalid email address');
    }
  }
  
  // Private method (ES2022)
  #validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  validateEmail(email) {
    return this.#validateEmail(email);
  }
  
  // Static methods
  static createAdmin(name, email) {
    return new User(name, email, 'admin');
  }
  
  static createModerator(name, email) {
    return new User(name, email, 'moderator');
  }
  
  static getTotalUsers() {
    return User.userCount;
  }
  
  static findByEmail(users, email) {
    return users.find(user => user.email === email);
  }
}

// ============= INHERITANCE =============
class AdminUser extends User {
  constructor(name, email) {
    super(name, email, 'admin');
    this.permissions = new Set(['read', 'write', 'delete', 'manage_users']);
    this.lastLogin = null;
  }
  
  // Override parent method
  getProfile() {
    const baseProfile = super.getProfile();
    return {
      ...baseProfile,
      permissions: Array.from(this.permissions),
      lastLogin: this.lastLogin,
      isAdmin: true
    };
  }
  
  // Admin-specific methods
  addPermission(permission) {
    this.permissions.add(permission);
    return this;
  }
  
  removePermission(permission) {
    this.permissions.delete(permission);
    return this;
  }
  
  hasPermission(permission) {
    return this.permissions.has(permission);
  }
  
  manageUser(user, action) {
    if (!this.hasPermission('manage_users')) {
      throw new Error('Insufficient permissions');
    }
    
    switch (action) {
      case 'activate':
        user.activate();
        break;
      case 'deactivate':
        user.deactivate();
        break;
      default:
        throw new Error('Unknown action');
    }
    
    return user;
  }
  
  recordLogin() {
    this.lastLogin = new Date();
  }
}

// ============= MIXINS PATTERN =============
// Mixin for audit functionality
const AuditMixin = {
  initAudit() {
    this.auditLog = [];
  },
  
  logAction(action, details = {}) {
    if (!this.auditLog) this.initAudit();
    
    this.auditLog.push({
      action,
      details,
      timestamp: new Date(),
      user: this.name || 'Unknown'
    });
  },
  
  getAuditLog() {
    return this.auditLog || [];
  },
  
  clearAuditLog() {
    this.auditLog = [];
  }
};

// Mixin for validation
const ValidationMixin = {
  addValidator(field, validator) {
    if (!this.validators) this.validators = {};
    this.validators[field] = validator;
  },
  
  validate(data = this) {
    if (!this.validators) return { isValid: true, errors: [] };
    
    const errors = [];
    
    for (const [field, validator] of Object.entries(this.validators)) {
      const result = validator(data[field], data);
      if (!result.isValid) {
        errors.push({ field, message: result.message });
      }
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
};

// Apply mixins to class
Object.assign(User.prototype, AuditMixin, ValidationMixin);

// ============= ADVANCED CLASS PATTERNS =============
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
    return this;
  }
  
  off(event, listener) {
    if (!this.events[event]) return this;
    
    this.events[event] = this.events[event].filter(l => l !== listener);
    return this;
  }
  
  emit(event, ...args) {
    if (!this.events[event]) return this;
    
    this.events[event].forEach(listener => {
      listener.apply(this, args);
    });
    
    return this;
  }
  
  once(event, listener) {
    const onceListener = (...args) => {
      listener.apply(this, args);
      this.off(event, onceListener);
    };
    
    return this.on(event, onceListener);
  }
}

class Model extends EventEmitter {
  constructor(data = {}) {
    super();
    this.data = { ...data };
    this.changes = {};
    this.isModified = false;
  }
  
  get(key) {
    return this.data[key];
  }
  
  set(key, value) {
    const oldValue = this.data[key];
    
    if (oldValue !== value) {
      this.data[key] = value;
      this.changes[key] = { oldValue, newValue: value };
      this.isModified = true;
      
      this.emit('change', key, value, oldValue);
      this.emit(`change:${key}`, value, oldValue);
    }
    
    return this;
  }
  
  update(updates) {
    Object.entries(updates).forEach(([key, value]) => {
      this.set(key, value);
    });
    return this;
  }
  
  reset() {
    this.data = {};
    this.changes = {};
    this.isModified = false;
    this.emit('reset');
    return this;
  }
  
  save() {
    // Simulate saving
    return new Promise((resolve) => {
      setTimeout(() => {
        this.changes = {};
        this.isModified = false;
        this.emit('save', this.data);
        resolve(this.data);
      }, 100);
    });
  }
  
  toJSON() {
    return { ...this.data };
  }
}

// Usage examples
const user = new User('John Doe', 'john@example.com');
user.addValidator('email', (email) => ({
  isValid: /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
  message: 'Invalid email format'
}));

user.logAction('profile_created', { role: user.role });

const admin = new AdminUser('Admin User', 'admin@example.com');
admin.addPermission('system_access');

console.log('User profile:', user.getProfile());
console.log('Admin profile:', admin.getProfile());
console.log('Total users:', User.getTotalUsers());

// Model usage
const userModel = new Model({ name: 'Jane', age: 25 });

userModel.on('change', (key, newValue, oldValue) => {
  console.log(`${key} changed from ${oldValue} to ${newValue}`);
});

userModel.set('age', 26); // Triggers change event
userModel.update({ name: 'Jane Smith', city: 'Bangkok' });
```

### **🏭 Factory Patterns และ Advanced OOP**

```javascript
// ============= FACTORY PATTERN =============
class UserFactory {
  static userTypes = {
    admin: AdminUser,
    moderator: ModeratorUser,
    user: User
  };
  
  static createUser(type, name, email, options = {}) {
    const UserClass = this.userTypes[type];
    
    if (!UserClass) {
      throw new Error(`Unknown user type: ${type}`);
    }
    
    const user = new UserClass(name, email);
    
    // Apply options
    if (options.permissions) {
      options.permissions.forEach(permission => {
        if (user.addPermission) {
          user.addPermission(permission);
        }
      });
    }
    
    if (options.profile) {
      user.updateProfile(options.profile);
    }
    
    return user;
  }
  
  static createBatch(users) {
    return users.map(userData => 
      this.createUser(userData.type, userData.name, userData.email, userData.options)
    );
  }
}

// ============= BUILDER PATTERN =============
class UserBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.userData = {
      name: '',
      email: '',
      role: 'user',
      permissions: [],
      profile: {}
    };
    return this;
  }
  
  setName(name) {
    this.userData.name = name;
    return this;
  }
  
  setEmail(email) {
    this.userData.email = email;
    return this;
  }
  
  setRole(role) {
    this.userData.role = role;
    return this;
  }
  
  addPermissions(...permissions) {
    this.userData.permissions.push(...permissions);
    return this;
  }
  
  setProfile(profile) {
    this.userData.profile = { ...this.userData.profile, ...profile };
    return this;
  }
  
  build() {
    const user = UserFactory.createUser(
      this.userData.role,
      this.userData.name,
      this.userData.email,
      {
        permissions: this.userData.permissions,
        profile: this.userData.profile
      }
    );
    
    this.reset(); // Reset for next build
    return user;
  }
}

// ============= SINGLETON PATTERN =============
class ConfigManager {
  static instance = null;
  
  constructor() {
    if (ConfigManager.instance) {
      return ConfigManager.instance;
    }
    
    this.config = {};
    this.env = 'development';
    ConfigManager.instance = this;
  }
  
  static getInstance() {
    if (!ConfigManager.instance) {
      ConfigManager.instance = new ConfigManager();
    }
    return ConfigManager.instance;
  }
  
  set(key, value) {
    this.config[key] = value;
  }
  
  get(key, defaultValue = null) {
    return this.config[key] ?? defaultValue;
  }
  
  setEnvironment(env) {
    this.env = env;
  }
  
  getEnvironment() {
    return this.env;
  }
  
  loadConfig(config) {
    this.config = { ...this.config, ...config };
  }
}

// ============= OBSERVER PATTERN =============
class DataStore extends EventEmitter {
  constructor() {
    super();
    this.data = {};
    this.subscribers = new Map();
  }
  
  subscribe(key, callback) {
    if (!this.subscribers.has(key)) {
      this.subscribers.set(key, new Set());
    }
    
    this.subscribers.get(key).add(callback);
    
    // Return unsubscribe function
    return () => {
      const callbacks = this.subscribers.get(key);
      if (callbacks) {
        callbacks.delete(callback);
      }
    };
  }
  
  set(key, value) {
    const oldValue = this.data[key];
    this.data[key] = value;
    
    // Notify specific subscribers
    const callbacks = this.subscribers.get(key);
    if (callbacks) {
      callbacks.forEach(callback => callback(value, oldValue, key));
    }
    
    // Emit general change event
    this.emit('change', { key, value, oldValue });
  }
  
  get(key) {
    return this.data[key];
  }
  
  delete(key) {
    const oldValue = this.data[key];
    delete this.data[key];
    
    const callbacks = this.subscribers.get(key);
    if (callbacks) {
      callbacks.forEach(callback => callback(undefined, oldValue, key));
    }
    
    this.emit('delete', { key, oldValue });
  }
  
  getState() {
    return { ...this.data };
  }
}

// ============= USAGE EXAMPLES =============
// Factory pattern
const users = UserFactory.createBatch([
  { type: 'admin', name: 'Admin User', email: 'admin@example.com' },
  { type: 'user', name: 'Regular User', email: 'user@example.com' },
  { type: 'moderator', name: 'Mod User', email: 'mod@example.com' }
]);

// Builder pattern
const complexUser = new UserBuilder()
  .setName('Complex User')
  .setEmail('complex@example.com')
  .setRole('admin')
  .addPermissions('read', 'write', 'delete')
  .setProfile({ department: 'IT', location: 'Bangkok' })
  .build();

// Singleton pattern
const config1 = new ConfigManager();
const config2 = ConfigManager.getInstance();
console.log(config1 === config2); // true

config1.set('apiUrl', 'https://api.example.com');
console.log(config2.get('apiUrl')); // https://api.example.com

// Observer pattern
const store = new DataStore();

const unsubscribe = store.subscribe('user', (newValue, oldValue) => {
  console.log(`User changed from ${oldValue?.name} to ${newValue?.name}`);
});

store.set('user', { name: 'John', age: 30 });
store.set('user', { name: 'Jane', age: 25 });

unsubscribe(); // Stop listening to changes
```

## 🚀 Next.js Applications

### **⚛️ Object-Oriented Components**

```jsx
// models/User.js
class User {
  constructor(data = {}) {
    this.id = data.id;
    this.name = data.name;
    this.email = data.email;
    this.avatar = data.avatar;
    this.role = data.role || 'user';
    this.preferences = data.preferences || {};
    this.createdAt = data.createdAt ? new Date(data.createdAt) : new Date();
    this.lastActiveAt = data.lastActiveAt ? new Date(data.lastActiveAt) : null;
  }
  
  get displayName() {
    return this.name || this.email.split('@')[0];
  }
  
  get initials() {
    return this.name
      .split(' ')
      .map(part => part[0])
      .join('')
      .toUpperCase()
      .slice(0, 2);
  }
  
  get isOnline() {
    if (!this.lastActiveAt) return false;
    const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000);
    return this.lastActiveAt > fiveMinutesAgo;
  }
  
  updateLastActive() {
    this.lastActiveAt = new Date();
  }
  
  updatePreferences(newPreferences) {
    this.preferences = { ...this.preferences, ...newPreferences };
  }
  
  hasRole(role) {
    return this.role === role;
  }
  
  toJSON() {
    return {
      id: this.id,
      name: this.name,
      email: this.email,
      avatar: this.avatar,
      role: this.role,
      preferences: this.preferences,
      createdAt: this.createdAt.toISOString(),
      lastActiveAt: this.lastActiveAt?.toISOString()
    };
  }
  
  static fromJSON(data) {
    return new User(data);
  }
}

// models/Post.js
class Post {
  constructor(data = {}) {
    this.id = data.id;
    this.title = data.title;
    this.content = data.content;
    this.author = data.author instanceof User ? data.author : new User(data.author);
    this.tags = data.tags || [];
    this.likes = data.likes || 0;
    this.comments = (data.comments || []).map(comment => new Comment(comment));
    this.createdAt = data.createdAt ? new Date(data.createdAt) : new Date();
    this.updatedAt = data.updatedAt ? new Date(data.updatedAt) : new Date();
    this.isPublished = data.isPublished ?? false;
  }
  
  get excerpt() {
    return this.content.length > 150 
      ? this.content.slice(0, 150) + '...'
      : this.content;
  }
  
  get readingTime() {
    const wordsPerMinute = 200;
    const wordCount = this.content.split(/\s+/).length;
    return Math.ceil(wordCount / wordsPerMinute);
  }
  
  get commentCount() {
    return this.comments.length;
  }
  
  addComment(commentData) {
    const comment = new Comment({
      ...commentData,
      postId: this.id,
      createdAt: new Date()
    });
    this.comments.push(comment);
    return comment;
  }
  
  like() {
    this.likes++;
  }
  
  unlike() {
    this.likes = Math.max(0, this.likes - 1);
  }
  
  publish() {
    this.isPublished = true;
    this.updatedAt = new Date();
  }
  
  unpublish() {
    this.isPublished = false;
    this.updatedAt = new Date();
  }
  
  addTag(tag) {
    if (!this.tags.includes(tag)) {
      this.tags.push(tag);
    }
  }
  
  removeTag(tag) {
    this.tags = this.tags.filter(t => t !== tag);
  }
  
  toJSON() {
    return {
      id: this.id,
      title: this.title,
      content: this.content,
      author: this.author.toJSON(),
      tags: this.tags,
      likes: this.likes,
      comments: this.comments.map(c => c.toJSON()),
      createdAt: this.createdAt.toISOString(),
      updatedAt: this.updatedAt.toISOString(),
      isPublished: this.isPublished
    };
  }
}

// components/UserCard.jsx
import { useState, useEffect } from 'react';

const UserCard = ({ userData, onUserUpdate }) => {
  const [user, setUser] = useState(new User(userData));
  const [isEditing, setIsEditing] = useState(false);
  const [editForm, setEditForm] = useState({});

  useEffect(() => {
    setUser(new User(userData));
  }, [userData]);

  const handleEdit = () => {
    setEditForm({
      name: user.name,
      email: user.email
    });
    setIsEditing(true);
  };

  const handleSave = () => {
    const updatedUser = new User({
      ...user.toJSON(),
      ...editForm
    });
    
    setUser(updatedUser);
    setIsEditing(false);
    
    if (onUserUpdate) {
      onUserUpdate(updatedUser.toJSON());
    }
  };

  const handleCancel = () => {
    setEditForm({});
    setIsEditing(false);
  };

  const handlePreferenceChange = (key, value) => {
    const updatedUser = new User(user.toJSON());
    updatedUser.updatePreferences({ [key]: value });
    setUser(updatedUser);
    
    if (onUserUpdate) {
      onUserUpdate(updatedUser.toJSON());
    }
  };

  return (
    <div className="user-card">
      <div className="user-header">
        <div className="avatar">
          {user.avatar ? (
            <img src={user.avatar} alt={user.displayName} />
          ) : (
            <div className="avatar-placeholder">
              {user.initials}
            </div>
          )}
          <div className={`status-indicator ${user.isOnline ? 'online' : 'offline'}`} />
        </div>

        <div className="user-info">
          {isEditing ? (
            <div className="edit-form">
              <input
                type="text"
                value={editForm.name}
                onChange={(e) => setEditForm({ ...editForm, name: e.target.value })}
                placeholder="Name"
              />
              <input
                type="email"
                value={editForm.email}
                onChange={(e) => setEditForm({ ...editForm, email: e.target.value })}
                placeholder="Email"
              />
              <div className="edit-actions">
                <button onClick={handleSave}>Save</button>
                <button onClick={handleCancel}>Cancel</button>
              </div>
            </div>
          ) : (
            <div className="user-details">
              <h3>{user.displayName}</h3>
              <p>{user.email}</p>
              <span className={`role role-${user.role}`}>{user.role}</span>
              <button onClick={handleEdit} className="edit-btn">Edit</button>
            </div>
          )}
        </div>
      </div>

      <div className="user-meta">
        <div className="meta-item">
          <label>Member since:</label>
          <span>{user.createdAt.toLocaleDateString()}</span>
        </div>
        
        {user.lastActiveAt && (
          <div className="meta-item">
            <label>Last active:</label>
            <span>{user.lastActiveAt.toLocaleString()}</span>
          </div>
        )}
      </div>

      <div className="user-preferences">
        <h4>Preferences</h4>
        
        <div className="preference-item">
          <label>Theme:</label>
          <select
            value={user.preferences.theme || 'light'}
            onChange={(e) => handlePreferenceChange('theme', e.target.value)}
          >
            <option value="light">Light</option>
            <option value="dark">Dark</option>
            <option value="auto">Auto</option>
          </select>
        </div>

        <div className="preference-item">
          <label>Language:</label>
          <select
            value={user.preferences.language || 'en'}
            onChange={(e) => handlePreferenceChange('language', e.target.value)}
          >
            <option value="en">English</option>
            <option value="th">ไทย</option>
            <option value="ja">日本語</option>
          </select>
        </div>

        <div className="preference-item">
          <label>Email notifications:</label>
          <input
            type="checkbox"
            checked={user.preferences.emailNotifications ?? true}
            onChange={(e) => handlePreferenceChange('emailNotifications', e.target.checked)}
          />
        </div>
      </div>
    </div>
  );
};

export default UserCard;
```

### **📊 Data Management Classes**

```javascript
// utils/ApiClient.js
class ApiClient {
  constructor(baseURL, options = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...options.headers
    };
    this.timeout = options.timeout || 10000;
    this.interceptors = {
      request: [],
      response: []
    };
  }

  addRequestInterceptor(interceptor) {
    this.interceptors.request.push(interceptor);
  }

  addResponseInterceptor(interceptor) {
    this.interceptors.response.push(interceptor);
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      method: 'GET',
      headers: { ...this.defaultHeaders },
      ...options
    };

    // Apply request interceptors
    for (const interceptor of this.interceptors.request) {
      await interceptor(config);
    }

    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), this.timeout);

      const response = await fetch(url, {
        ...config,
        signal: controller.signal
      });

      clearTimeout(timeoutId);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      let data = await response.json();

      // Apply response interceptors
      for (const interceptor of this.interceptors.response) {
        data = await interceptor(data, response);
      }

      return data;

    } catch (error) {
      if (error.name === 'AbortError') {
        throw new Error('Request timeout');
      }
      throw error;
    }
  }

  get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url);
  }

  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  delete(endpoint) {
    return this.request(endpoint, {
      method: 'DELETE'
    });
  }
}

// utils/DataManager.js
class DataManager extends EventEmitter {
  constructor(apiClient) {
    super();
    this.api = apiClient;
    this.cache = new Map();
    this.loading = new Set();
    this.errors = new Map();
  }

  async fetchUsers(params = {}) {
    const cacheKey = `users:${JSON.stringify(params)}`;
    
    if (this.loading.has(cacheKey)) {
      return this.waitForLoad(cacheKey);
    }

    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    this.loading.add(cacheKey);
    this.emit('loadingStart', { type: 'users', params });

    try {
      const data = await this.api.get('/users', params);
      const users = data.map(userData => new User(userData));
      
      this.cache.set(cacheKey, users);
      this.errors.delete(cacheKey);
      this.emit('loadingEnd', { type: 'users', params, success: true });
      
      return users;
    } catch (error) {
      this.errors.set(cacheKey, error);
      this.emit('loadingEnd', { type: 'users', params, success: false, error });
      throw error;
    } finally {
      this.loading.delete(cacheKey);
    }
  }

  async createUser(userData) {
    try {
      const response = await this.api.post('/users', userData);
      const user = new User(response);
      
      // Invalidate users cache
      this.clearCache('users');
      this.emit('userCreated', user);
      
      return user;
    } catch (error) {
      this.emit('error', { type: 'createUser', error });
      throw error;
    }
  }

  async updateUser(userId, updates) {
    try {
      const response = await this.api.put(`/users/${userId}`, updates);
      const user = new User(response);
      
      // Update cache
      this.updateCachedUser(userId, user);
      this.emit('userUpdated', user);
      
      return user;
    } catch (error) {
      this.emit('error', { type: 'updateUser', error });
      throw error;
    }
  }

  async deleteUser(userId) {
    try {
      await this.api.delete(`/users/${userId}`);
      
      // Remove from cache
      this.removeCachedUser(userId);
      this.emit('userDeleted', userId);
      
      return true;
    } catch (error) {
      this.emit('error', { type: 'deleteUser', error });
      throw error;
    }
  }

  updateCachedUser(userId, updatedUser) {
    for (const [key, users] of this.cache.entries()) {
      if (key.startsWith('users:') && Array.isArray(users)) {
        const index = users.findIndex(user => user.id === userId);
        if (index !== -1) {
          users[index] = updatedUser;
        }
      }
    }
  }

  removeCachedUser(userId) {
    for (const [key, users] of this.cache.entries()) {
      if (key.startsWith('users:') && Array.isArray(users)) {
        const index = users.findIndex(user => user.id === userId);
        if (index !== -1) {
          users.splice(index, 1);
        }
      }
    }
  }

  clearCache(prefix = '') {
    if (prefix) {
      for (const key of this.cache.keys()) {
        if (key.startsWith(prefix)) {
          this.cache.delete(key);
        }
      }
    } else {
      this.cache.clear();
    }
  }

  async waitForLoad(cacheKey) {
    return new Promise((resolve, reject) => {
      const checkLoad = () => {
        if (!this.loading.has(cacheKey)) {
          if (this.cache.has(cacheKey)) {
            resolve(this.cache.get(cacheKey));
          } else if (this.errors.has(cacheKey)) {
            reject(this.errors.get(cacheKey));
          }
        } else {
          setTimeout(checkLoad, 100);
        }
      };
      checkLoad();
    });
  }
}

// hooks/useDataManager.js
import { useState, useEffect, useContext, createContext } from 'react';

const DataManagerContext = createContext();

export const DataManagerProvider = ({ children }) => {
  const apiClient = new ApiClient('/api');
  const dataManager = new DataManager(apiClient);

  // Add authentication interceptor
  apiClient.addRequestInterceptor(async (config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
  });

  return (
    <DataManagerContext.Provider value={dataManager}>
      {children}
    </DataManagerContext.Provider>
  );
};

export const useDataManager = () => {
  const dataManager = useContext(DataManagerContext);
  if (!dataManager) {
    throw new Error('useDataManager must be used within DataManagerProvider');
  }
  return dataManager;
};

export const useUsers = (params = {}) => {
  const dataManager = useDataManager();
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const fetchedUsers = await dataManager.fetchUsers(params);
        setUsers(fetchedUsers);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    // Listen for updates
    const handleUserCreated = (user) => {
      setUsers(prev => [...prev, user]);
    };

    const handleUserUpdated = (user) => {
      setUsers(prev => prev.map(u => u.id === user.id ? user : u));
    };

    const handleUserDeleted = (userId) => {
      setUsers(prev => prev.filter(u => u.id !== userId));
    };

    dataManager.on('userCreated', handleUserCreated);
    dataManager.on('userUpdated', handleUserUpdated);
    dataManager.on('userDeleted', handleUserDeleted);

    return () => {
      dataManager.off('userCreated', handleUserCreated);
      dataManager.off('userUpdated', handleUserUpdated);
      dataManager.off('userDeleted', handleUserDeleted);
    };
  }, [dataManager, JSON.stringify(params)]);

  return { users, loading, error };
};
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Object Manipulation**
สร้างฟังก์ชันจัดการ objects:

```javascript
// TODO: สร้างฟังก์ชันเหล่านี้

const complexObject = {
  user: {
    id: 1,
    profile: {
      name: 'John',
      address: {
        street: '123 Main St',
        city: 'Bangkok'
      }
    },
    preferences: {
      theme: 'dark',
      notifications: true
    }
  },
  posts: [
    { id: 1, title: 'First Post', tags: ['javascript', 'tutorial'] },
    { id: 2, title: 'Second Post', tags: ['react', 'nextjs'] }
  ]
};

// 1. Create deep object diff
function createObjectDiff(obj1, obj2) {
  // Return object showing differences between two objects
}

// 2. Transform object keys
function transformKeys(obj, transformer) {
  // Apply transformer function to all keys recursively
  // transformer could be camelCase, snake_case, etc.
}

// 3. Query object paths
function queryObject(obj, query) {
  // Support queries like 'user.profile.name' or 'posts[0].title'
}

// 4. Validate object schema
function validateSchema(obj, schema) {
  // Validate object against a schema definition
}
```

### **🏛️ แบบฝึกหัดที่ 2: Class Design**
สร้าง class hierarchy:

```javascript
// TODO: สร้าง class system สำหรับ blog platform

// 1. Base Content class
class Content {
  // Common properties and methods for all content types
}

// 2. Article extends Content
class Article extends Content {
  // Article-specific features
}

// 3. Video extends Content
class Video extends Content {
  // Video-specific features
}

// 4. ContentManager class
class ContentManager {
  // Manage different content types
  // Include search, filtering, publishing
}

// 5. User permission system
class Permission {
  // Handle user permissions and roles
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง object-oriented component system:

```jsx
// TODO: สร้าง reusable component system ที่:
// 1. ใช้ classes สำหรับ data models
// 2. มี data management layer
// 3. Support real-time updates
// 4. Include caching และ error handling

// Example usage:
const BlogPost = ({ postId }) => {
  // Use object-oriented data models
  // Handle loading states
  // Support real-time updates
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 7: Modern Array Methods](./07-array-methods.md)

**บทถัดไป**: [บทที่ 9: Promises & Async/Await](./09-promises-async.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [MDN: Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- [JavaScript.info: Objects](https://javascript.info/object)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Modern Object Features** ใน ES6+
- **ES6+ Classes** และ inheritance
- **Object-oriented Design Patterns**
- **การใช้งานใน Next.js** สำหรับ data modeling
- **Advanced OOP Concepts** และ practical applications

Objects และ Classes ใน ES6+ ช่วยให้การจัดระเบียบโค้ดและการจัดการข้อมูลเป็นไปอย่างมีประสิทธิภาพ! 🚀
