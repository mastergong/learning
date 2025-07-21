# บทที่ 12: Symbols & Maps/Sets

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Symbols และการใช้งานเป็น unique identifiers
- ทำความเข้าใจ Maps และ Sets data structures
- เรียนรู้ WeakMap และ WeakSet สำหรับ memory management
- ประยุกต์ใช้ใน Next.js สำหรับ Advanced Data Structures

## 🔣 Symbols

### **🆔 Symbol Basics**

```javascript
// ============= SYMBOL FUNDAMENTALS =============
// Creating symbols
const sym1 = Symbol();
const sym2 = Symbol('description');
const sym3 = Symbol('description');

console.log(sym1); // Symbol()
console.log(sym2); // Symbol(description)
console.log(sym3); // Symbol(description)

// Symbols are always unique
console.log(sym2 === sym3); // false
console.log(typeof sym1); // 'symbol'

// Symbol registry - global symbols
const globalSym1 = Symbol.for('app.config');
const globalSym2 = Symbol.for('app.config');
console.log(globalSym1 === globalSym2); // true

// Get symbol key from registry
console.log(Symbol.keyFor(globalSym1)); // 'app.config'
console.log(Symbol.keyFor(sym1)); // undefined (not in global registry)

// ============= SYMBOL AS OBJECT PROPERTIES =============
const id = Symbol('id');
const name = Symbol('name');

const user = {
  [id]: 12345,
  [name]: 'John Doe',
  email: 'john@example.com'
};

console.log(user[id]); // 12345
console.log(user[name]); // 'John Doe'
console.log(user.email); // 'john@example.com'

// Symbol properties are not enumerable
console.log(Object.keys(user)); // ['email']
console.log(Object.getOwnPropertyNames(user)); // ['email']
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id), Symbol(name)]

// Get all properties including symbols
const allKeys = [
  ...Object.getOwnPropertyNames(user),
  ...Object.getOwnPropertySymbols(user)
];
console.log(allKeys); // ['email', Symbol(id), Symbol(name)]

// ============= WELL-KNOWN SYMBOLS =============
// Symbol.iterator - makes object iterable
class NumberRange {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    
    return {
      next() {
        if (current < end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
}

const range = new NumberRange(1, 5);
console.log([...range]); // [1, 2, 3, 4]

// Symbol.toStringTag - customize toString output
class CustomClass {
  get [Symbol.toStringTag]() {
    return 'CustomClass';
  }
}

const instance = new CustomClass();
console.log(instance.toString()); // [object CustomClass]

// Symbol.toPrimitive - customize type conversion
class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }
  
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case 'number':
        return this.celsius;
      case 'string':
        return `${this.celsius}°C`;
      default:
        return this.celsius;
    }
  }
}

const temp = new Temperature(25);
console.log(+temp); // 25 (number conversion)
console.log(`Temperature: ${temp}`); // "Temperature: 25°C" (string conversion)

// ============= PRIVATE PROPERTIES WITH SYMBOLS =============
const privateData = Symbol('privateData');
const privateMethod = Symbol('privateMethod');

class SecureClass {
  constructor(data) {
    this[privateData] = data;
    this.publicData = 'This is public';
  }
  
  [privateMethod]() {
    return 'This is a private method';
  }
  
  getPrivateData() {
    return this[privateData];
  }
  
  callPrivateMethod() {
    return this[privateMethod]();
  }
}

const secure = new SecureClass('secret information');
console.log(secure.publicData); // 'This is public'
console.log(secure.getPrivateData()); // 'secret information'
console.log(secure.callPrivateMethod()); // 'This is a private method'

// Private properties are not accessible directly
console.log(Object.keys(secure)); // ['publicData']

// ============= SYMBOL-BASED METADATA SYSTEM =============
const METADATA = Symbol('metadata');
const VALIDATORS = Symbol('validators');
const TRANSFORMERS = Symbol('transformers');

class MetadataClass {
  constructor() {
    this[METADATA] = new Map();
    this[VALIDATORS] = new Map();
    this[TRANSFORMERS] = new Map();
  }
  
  setMetadata(key, value) {
    this[METADATA].set(key, value);
    return this;
  }
  
  getMetadata(key) {
    return this[METADATA].get(key);
  }
  
  addValidator(field, validator) {
    if (!this[VALIDATORS].has(field)) {
      this[VALIDATORS].set(field, []);
    }
    this[VALIDATORS].get(field).push(validator);
    return this;
  }
  
  addTransformer(field, transformer) {
    if (!this[TRANSFORMERS].has(field)) {
      this[TRANSFORMERS].set(field, []);
    }
    this[TRANSFORMERS].get(field).push(transformer);
    return this;
  }
  
  validate(data) {
    const errors = [];
    
    for (const [field, validators] of this[VALIDATORS]) {
      const value = data[field];
      for (const validator of validators) {
        const result = validator(value);
        if (!result.valid) {
          errors.push({ field, message: result.message });
        }
      }
    }
    
    return { valid: errors.length === 0, errors };
  }
  
  transform(data) {
    const transformed = { ...data };
    
    for (const [field, transformers] of this[TRANSFORMERS]) {
      let value = transformed[field];
      for (const transformer of transformers) {
        value = transformer(value);
      }
      transformed[field] = value;
    }
    
    return transformed;
  }
}

// Usage
const schema = new MetadataClass()
  .setMetadata('version', '1.0')
  .setMetadata('author', 'John Doe')
  .addValidator('email', (value) => ({
    valid: /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    message: 'Invalid email format'
  }))
  .addValidator('age', (value) => ({
    valid: value >= 18,
    message: 'Must be 18 or older'
  }))
  .addTransformer('name', (value) => value.trim())
  .addTransformer('name', (value) => 
    value.charAt(0).toUpperCase() + value.slice(1).toLowerCase()
  );

const userData = {
  name: '  john doe  ',
  email: 'john@example.com',
  age: 25
};

const transformed = schema.transform(userData);
const validation = schema.validate(transformed);

console.log('Transformed:', transformed);
console.log('Validation:', validation);
```

## 🗺️ Maps

### **📊 Map Data Structure**

```javascript
// ============= MAP BASICS =============
// Creating maps
const map1 = new Map();
const map2 = new Map([
  ['key1', 'value1'],
  ['key2', 'value2'],
  [42, 'number key'],
  [true, 'boolean key']
]);

// Set and get values
map1.set('name', 'John');
map1.set('age', 30);
map1.set(Symbol('id'), 12345);

console.log(map1.get('name')); // 'John'
console.log(map1.get('age')); // 30
console.log(map1.has('name')); // true
console.log(map1.size); // 3

// Objects as keys
const keyObj1 = { type: 'user' };
const keyObj2 = { type: 'admin' };

map1.set(keyObj1, 'User data');
map1.set(keyObj2, 'Admin data');

console.log(map1.get(keyObj1)); // 'User data'

// ============= MAP ITERATION =============
const userMap = new Map([
  ['john', { name: 'John', age: 30 }],
  ['jane', { name: 'Jane', age: 25 }],
  ['bob', { name: 'Bob', age: 35 }]
]);

// Iterate over keys
for (const key of userMap.keys()) {
  console.log('Key:', key);
}

// Iterate over values
for (const value of userMap.values()) {
  console.log('Value:', value);
}

// Iterate over entries
for (const [key, value] of userMap.entries()) {
  console.log(`${key}: ${value.name} (${value.age})`);
}

// Default iteration is entries
for (const [key, value] of userMap) {
  console.log(`${key}: ${value.name}`);
}

// Using forEach
userMap.forEach((value, key, map) => {
  console.log(`${key}: ${value.name}`);
});

// ============= ADVANCED MAP OPERATIONS =============
class AdvancedMap extends Map {
  // Filter entries
  filter(predicate) {
    const filtered = new AdvancedMap();
    for (const [key, value] of this) {
      if (predicate(value, key, this)) {
        filtered.set(key, value);
      }
    }
    return filtered;
  }
  
  // Map values to new map
  map(mapper) {
    const mapped = new AdvancedMap();
    for (const [key, value] of this) {
      mapped.set(key, mapper(value, key, this));
    }
    return mapped;
  }
  
  // Find entry
  find(predicate) {
    for (const [key, value] of this) {
      if (predicate(value, key, this)) {
        return { key, value };
      }
    }
    return undefined;
  }
  
  // Reduce map to single value
  reduce(reducer, initialValue) {
    let accumulator = initialValue;
    for (const [key, value] of this) {
      accumulator = reducer(accumulator, value, key, this);
    }
    return accumulator;
  }
  
  // Convert to object
  toObject() {
    const obj = {};
    for (const [key, value] of this) {
      obj[key] = value;
    }
    return obj;
  }
  
  // Merge with another map
  merge(otherMap, conflictResolver = (existing, incoming) => incoming) {
    const merged = new AdvancedMap(this);
    
    for (const [key, value] of otherMap) {
      if (merged.has(key)) {
        merged.set(key, conflictResolver(merged.get(key), value));
      } else {
        merged.set(key, value);
      }
    }
    
    return merged;
  }
  
  // Group by function
  static groupBy(iterable, keySelector) {
    const grouped = new AdvancedMap();
    
    for (const item of iterable) {
      const key = keySelector(item);
      if (!grouped.has(key)) {
        grouped.set(key, []);
      }
      grouped.get(key).push(item);
    }
    
    return grouped;
  }
}

// Usage examples
const users = [
  { name: 'John', age: 30, department: 'IT' },
  { name: 'Jane', age: 25, department: 'HR' },
  { name: 'Bob', age: 35, department: 'IT' },
  { name: 'Alice', age: 28, department: 'Finance' }
];

const userMap2 = new AdvancedMap(users.map(user => [user.name, user]));

// Filter users over 27
const oldUsers = userMap2.filter(user => user.age > 27);
console.log('Users over 27:', oldUsers.toObject());

// Map to just names and ages
const simpleUsers = userMap2.map(user => ({ name: user.name, age: user.age }));
console.log('Simple users:', simpleUsers.toObject());

// Group by department
const byDepartment = AdvancedMap.groupBy(users, user => user.department);
console.log('By department:', byDepartment.toObject());

// ============= CACHE IMPLEMENTATION WITH MAP =============
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
  }
  
  get(key) {
    if (this.cache.has(key)) {
      const value = this.cache.get(key);
      
      // Move to end (most recently used)
      this.cache.delete(key);
      this.cache.set(key, value);
      
      return value;
    }
    return undefined;
  }
  
  set(key, value) {
    if (this.cache.has(key)) {
      // Update existing key
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove least recently used (first entry)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, value);
  }
  
  has(key) {
    return this.cache.has(key);
  }
  
  delete(key) {
    return this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
  
  get size() {
    return this.cache.size;
  }
  
  keys() {
    return this.cache.keys();
  }
  
  values() {
    return this.cache.values();
  }
  
  entries() {
    return this.cache.entries();
  }
}

// Usage
const cache = new LRUCache(3);
cache.set('a', 1);
cache.set('b', 2);
cache.set('c', 3);
console.log([...cache.keys()]); // ['a', 'b', 'c']

cache.get('a'); // Move 'a' to end
console.log([...cache.keys()]); // ['b', 'c', 'a']

cache.set('d', 4); // Evict 'b'
console.log([...cache.keys()]); // ['c', 'a', 'd']
```

## 🔢 Sets

### **🎯 Set Data Structure**

```javascript
// ============= SET BASICS =============
// Creating sets
const set1 = new Set();
const set2 = new Set([1, 2, 3, 4, 5]);
const set3 = new Set('hello'); // Set(['h', 'e', 'l', 'o'])

// Add and delete values
set1.add(1);
set1.add(2);
set1.add(2); // Duplicate - ignored
set1.add('hello');

console.log(set1.has(1)); // true
console.log(set1.has(3)); // false
console.log(set1.size); // 3

set1.delete(2);
console.log(set1.has(2)); // false

// ============= SET OPERATIONS =============
class MathSet extends Set {
  // Union: elements in either set
  union(otherSet) {
    const result = new MathSet(this);
    for (const item of otherSet) {
      result.add(item);
    }
    return result;
  }
  
  // Intersection: elements in both sets
  intersection(otherSet) {
    const result = new MathSet();
    for (const item of this) {
      if (otherSet.has(item)) {
        result.add(item);
      }
    }
    return result;
  }
  
  // Difference: elements in this set but not in other
  difference(otherSet) {
    const result = new MathSet();
    for (const item of this) {
      if (!otherSet.has(item)) {
        result.add(item);
      }
    }
    return result;
  }
  
  // Symmetric difference: elements in either set but not both
  symmetricDifference(otherSet) {
    const result = new MathSet();
    
    for (const item of this) {
      if (!otherSet.has(item)) {
        result.add(item);
      }
    }
    
    for (const item of otherSet) {
      if (!this.has(item)) {
        result.add(item);
      }
    }
    
    return result;
  }
  
  // Subset: all elements in this set are in other set
  isSubsetOf(otherSet) {
    for (const item of this) {
      if (!otherSet.has(item)) {
        return false;
      }
    }
    return true;
  }
  
  // Superset: this set contains all elements of other set
  isSupersetOf(otherSet) {
    return otherSet.isSubsetOf(this);
  }
  
  // Filter elements
  filter(predicate) {
    const result = new MathSet();
    for (const item of this) {
      if (predicate(item)) {
        result.add(item);
      }
    }
    return result;
  }
  
  // Map elements to new set
  map(mapper) {
    const result = new MathSet();
    for (const item of this) {
      result.add(mapper(item));
    }
    return result;
  }
  
  // Convert to array
  toArray() {
    return [...this];
  }
}

// Usage examples
const setA = new MathSet([1, 2, 3, 4]);
const setB = new MathSet([3, 4, 5, 6]);

console.log('Set A:', setA.toArray()); // [1, 2, 3, 4]
console.log('Set B:', setB.toArray()); // [3, 4, 5, 6]

console.log('Union:', setA.union(setB).toArray()); // [1, 2, 3, 4, 5, 6]
console.log('Intersection:', setA.intersection(setB).toArray()); // [3, 4]
console.log('Difference A-B:', setA.difference(setB).toArray()); // [1, 2]
console.log('Difference B-A:', setB.difference(setA).toArray()); // [5, 6]
console.log('Symmetric Diff:', setA.symmetricDifference(setB).toArray()); // [1, 2, 5, 6]

// ============= ADVANCED SET APPLICATIONS =============
// Unique elements tracker
class UniqueTracker {
  constructor() {
    this.seen = new Set();
    this.duplicates = new Set();
  }
  
  add(item) {
    if (this.seen.has(item)) {
      this.duplicates.add(item);
      return false; // Was duplicate
    } else {
      this.seen.add(item);
      return true; // Was unique
    }
  }
  
  isUnique(item) {
    return this.seen.has(item) && !this.duplicates.has(item);
  }
  
  isDuplicate(item) {
    return this.duplicates.has(item);
  }
  
  getUnique() {
    return new Set([...this.seen].filter(item => !this.duplicates.has(item)));
  }
  
  getDuplicates() {
    return new Set(this.duplicates);
  }
  
  reset() {
    this.seen.clear();
    this.duplicates.clear();
  }
}

// Usage
const tracker = new UniqueTracker();
const items = [1, 2, 3, 2, 4, 3, 5, 1];

items.forEach(item => {
  const isUnique = tracker.add(item);
  console.log(`${item}: ${isUnique ? 'unique' : 'duplicate'}`);
});

console.log('Unique items:', [...tracker.getUnique()]); // [4, 5]
console.log('Duplicate items:', [...tracker.getDuplicates()]); // [2, 3, 1]

// ============= PERMISSIONS SYSTEM WITH SETS =============
class PermissionManager {
  constructor() {
    this.userPermissions = new Map(); // userId -> Set of permissions
    this.rolePermissions = new Map(); // roleName -> Set of permissions
    this.userRoles = new Map(); // userId -> Set of roles
  }
  
  // Role management
  createRole(roleName, permissions = []) {
    this.rolePermissions.set(roleName, new Set(permissions));
  }
  
  addPermissionToRole(roleName, permission) {
    if (!this.rolePermissions.has(roleName)) {
      this.createRole(roleName);
    }
    this.rolePermissions.get(roleName).add(permission);
  }
  
  removePermissionFromRole(roleName, permission) {
    if (this.rolePermissions.has(roleName)) {
      this.rolePermissions.get(roleName).delete(permission);
    }
  }
  
  // User management
  assignRoleToUser(userId, roleName) {
    if (!this.userRoles.has(userId)) {
      this.userRoles.set(userId, new Set());
    }
    this.userRoles.get(userId).add(roleName);
  }
  
  removeRoleFromUser(userId, roleName) {
    if (this.userRoles.has(userId)) {
      this.userRoles.get(userId).delete(roleName);
    }
  }
  
  addDirectPermission(userId, permission) {
    if (!this.userPermissions.has(userId)) {
      this.userPermissions.set(userId, new Set());
    }
    this.userPermissions.get(userId).add(permission);
  }
  
  removeDirectPermission(userId, permission) {
    if (this.userPermissions.has(userId)) {
      this.userPermissions.get(userId).delete(permission);
    }
  }
  
  // Permission checking
  getUserPermissions(userId) {
    const permissions = new Set();
    
    // Add direct permissions
    if (this.userPermissions.has(userId)) {
      for (const permission of this.userPermissions.get(userId)) {
        permissions.add(permission);
      }
    }
    
    // Add role-based permissions
    if (this.userRoles.has(userId)) {
      for (const roleName of this.userRoles.get(userId)) {
        if (this.rolePermissions.has(roleName)) {
          for (const permission of this.rolePermissions.get(roleName)) {
            permissions.add(permission);
          }
        }
      }
    }
    
    return permissions;
  }
  
  hasPermission(userId, permission) {
    return this.getUserPermissions(userId).has(permission);
  }
  
  hasAllPermissions(userId, permissions) {
    const userPermissions = this.getUserPermissions(userId);
    return permissions.every(permission => userPermissions.has(permission));
  }
  
  hasAnyPermissions(userId, permissions) {
    const userPermissions = this.getUserPermissions(userId);
    return permissions.some(permission => userPermissions.has(permission));
  }
}

// Usage
const pm = new PermissionManager();

// Create roles
pm.createRole('admin', ['read', 'write', 'delete', 'manage_users']);
pm.createRole('editor', ['read', 'write']);
pm.createRole('viewer', ['read']);

// Assign roles
pm.assignRoleToUser('user1', 'admin');
pm.assignRoleToUser('user2', 'editor');
pm.assignRoleToUser('user3', 'viewer');

// Add direct permission
pm.addDirectPermission('user2', 'delete');

// Check permissions
console.log('User1 permissions:', [...pm.getUserPermissions('user1')]);
console.log('User2 can delete:', pm.hasPermission('user2', 'delete'));
console.log('User3 can write:', pm.hasPermission('user3', 'write'));
```

## 💾 WeakMap and WeakSet

### **🗑️ Weak References**

```javascript
// ============= WEAKMAP BASICS =============
const weakMap = new WeakMap();

// Only objects can be keys in WeakMap
const obj1 = { name: 'Object 1' };
const obj2 = { name: 'Object 2' };

weakMap.set(obj1, 'data for object 1');
weakMap.set(obj2, 'data for object 2');

console.log(weakMap.get(obj1)); // 'data for object 1'
console.log(weakMap.has(obj2)); // true

// Cannot iterate over WeakMap
// console.log(weakMap.keys()); // Error: not iterable

// ============= WEAKMAP FOR PRIVATE DATA =============
const privateData = new WeakMap();

class PrivateExample {
  constructor(secret) {
    privateData.set(this, { secret, created: new Date() });
  }
  
  getSecret() {
    return privateData.get(this).secret;
  }
  
  updateSecret(newSecret) {
    const data = privateData.get(this);
    data.secret = newSecret;
    data.updated = new Date();
  }
  
  getMetadata() {
    return privateData.get(this);
  }
}

const instance1 = new PrivateExample('my secret');
const instance2 = new PrivateExample('another secret');

console.log(instance1.getSecret()); // 'my secret'
console.log(instance2.getSecret()); // 'another secret'

// Private data is automatically cleaned up when object is garbage collected

// ============= WEAKMAP FOR METADATA =============
class MetadataManager {
  constructor() {
    this.metadata = new WeakMap();
  }
  
  addMetadata(obj, meta) {
    if (!this.metadata.has(obj)) {
      this.metadata.set(obj, {});
    }
    Object.assign(this.metadata.get(obj), meta);
  }
  
  getMetadata(obj, key) {
    const meta = this.metadata.get(obj);
    return key ? meta?.[key] : meta;
  }
  
  hasMetadata(obj) {
    return this.metadata.has(obj);
  }
  
  removeMetadata(obj) {
    return this.metadata.delete(obj);
  }
}

// Usage
const metaManager = new MetadataManager();
const userObj = { name: 'John' };

metaManager.addMetadata(userObj, {
  createdAt: new Date(),
  version: 1,
  tags: ['user', 'active']
});

console.log(metaManager.getMetadata(userObj)); 
// { createdAt: Date, version: 1, tags: ['user', 'active'] }

// ============= WEAKSET BASICS =============
const weakSet = new WeakSet();

const element1 = { id: 1 };
const element2 = { id: 2 };

weakSet.add(element1);
weakSet.add(element2);

console.log(weakSet.has(element1)); // true
console.log(weakSet.has(element2)); // true

weakSet.delete(element1);
console.log(weakSet.has(element1)); // false

// ============= WEAKSET FOR OBJECT TRACKING =============
class ObjectTracker {
  constructor() {
    this.tracked = new WeakSet();
    this.processedObjects = new WeakSet();
  }
  
  track(obj) {
    this.tracked.add(obj);
  }
  
  untrack(obj) {
    this.tracked.delete(obj);
    this.processedObjects.delete(obj);
  }
  
  isTracked(obj) {
    return this.tracked.has(obj);
  }
  
  markAsProcessed(obj) {
    if (this.tracked.has(obj)) {
      this.processedObjects.add(obj);
    }
  }
  
  isProcessed(obj) {
    return this.processedObjects.has(obj);
  }
  
  needsProcessing(obj) {
    return this.tracked.has(obj) && !this.processedObjects.has(obj);
  }
}

// Usage
const tracker = new ObjectTracker();
const task1 = { id: 1, name: 'Task 1' };
const task2 = { id: 2, name: 'Task 2' };

tracker.track(task1);
tracker.track(task2);

console.log(tracker.needsProcessing(task1)); // true

tracker.markAsProcessed(task1);
console.log(tracker.needsProcessing(task1)); // false
console.log(tracker.isProcessed(task1)); // true

// ============= MEMORY-EFFICIENT CACHING =============
class WeakCache {
  constructor() {
    this.cache = new WeakMap();
  }
  
  get(key, factory) {
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }
    
    const value = factory();
    this.cache.set(key, value);
    return value;
  }
  
  set(key, value) {
    this.cache.set(key, value);
  }
  
  has(key) {
    return this.cache.has(key);
  }
  
  delete(key) {
    return this.cache.delete(key);
  }
}

// Usage - automatically cleaned up when objects are garbage collected
const weakCache = new WeakCache();

function expensiveComputation(data) {
  return weakCache.get(data, () => {
    console.log('Computing for:', data);
    return data.value * 2 + Math.random();
  });
}

const dataObj = { value: 10 };
console.log(expensiveComputation(dataObj)); // Computes
console.log(expensiveComputation(dataObj)); // Returns cached value
```

## 🚀 Next.js Applications

### **🗃️ Advanced Data Management**

```jsx
// ============= SYMBOL-BASED COMPONENT SYSTEM =============
// lib/component-registry.js
const COMPONENT_TYPE = Symbol('componentType');
const COMPONENT_PROPS = Symbol('componentProps');
const COMPONENT_META = Symbol('componentMeta');

export class ComponentRegistry {
  constructor() {
    this.components = new Map();
    this.instances = new WeakMap();
    this.metadata = new WeakMap();
  }
  
  register(name, component, meta = {}) {
    component[COMPONENT_TYPE] = name;
    component[COMPONENT_META] = meta;
    
    this.components.set(name, component);
    return this;
  }
  
  get(name) {
    return this.components.get(name);
  }
  
  has(name) {
    return this.components.has(name);
  }
  
  getComponentType(component) {
    return component[COMPONENT_TYPE];
  }
  
  getComponentMeta(component) {
    return component[COMPONENT_META];
  }
  
  trackInstance(component, instance) {
    this.instances.set(instance, component);
  }
  
  getInstanceComponent(instance) {
    return this.instances.get(instance);
  }
  
  addInstanceMetadata(instance, meta) {
    if (!this.metadata.has(instance)) {
      this.metadata.set(instance, {});
    }
    Object.assign(this.metadata.get(instance), meta);
  }
  
  getInstanceMetadata(instance) {
    return this.metadata.get(instance);
  }
}

export const componentRegistry = new ComponentRegistry();

// ============= STATE MANAGEMENT WITH MAPS =============
// lib/state-manager.js
export class StateManager {
  constructor() {
    this.state = new Map();
    this.subscribers = new Map(); // key -> Set of callbacks
    this.middleware = [];
    this.history = [];
    this.maxHistorySize = 50;
  }
  
  // Subscribe to state changes
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
        if (callbacks.size === 0) {
          this.subscribers.delete(key);
        }
      }
    };
  }
  
  // Set state value
  setState(key, value) {
    const oldValue = this.state.get(key);
    
    // Apply middleware
    let newValue = value;
    for (const mw of this.middleware) {
      newValue = mw(key, newValue, oldValue);
    }
    
    this.state.set(key, newValue);
    
    // Add to history
    this.history.push({
      timestamp: Date.now(),
      key,
      oldValue,
      newValue,
      type: 'SET'
    });
    
    // Limit history size
    if (this.history.length > this.maxHistorySize) {
      this.history.shift();
    }
    
    // Notify subscribers
    this.notifySubscribers(key, newValue, oldValue);
  }
  
  // Get state value
  getState(key) {
    return this.state.get(key);
  }
  
  // Check if state exists
  hasState(key) {
    return this.state.has(key);
  }
  
  // Delete state
  deleteState(key) {
    const oldValue = this.state.get(key);
    const deleted = this.state.delete(key);
    
    if (deleted) {
      this.history.push({
        timestamp: Date.now(),
        key,
        oldValue,
        newValue: undefined,
        type: 'DELETE'
      });
      
      this.notifySubscribers(key, undefined, oldValue);
    }
    
    return deleted;
  }
  
  // Add middleware
  use(middleware) {
    this.middleware.push(middleware);
  }
  
  // Notify subscribers
  notifySubscribers(key, newValue, oldValue) {
    const callbacks = this.subscribers.get(key);
    if (callbacks) {
      callbacks.forEach(callback => {
        try {
          callback(newValue, oldValue, key);
        } catch (error) {
          console.error('State subscriber error:', error);
        }
      });
    }
  }
  
  // Get all state
  getAllState() {
    return new Map(this.state);
  }
  
  // Get history
  getHistory() {
    return [...this.history];
  }
  
  // Clear history
  clearHistory() {
    this.history = [];
  }
}

// Create global state manager
export const globalState = new StateManager();

// Add logging middleware
globalState.use((key, newValue, oldValue) => {
  console.log(`State change: ${key}`, { oldValue, newValue });
  return newValue;
});

// ============= REACT HOOKS FOR STATE MANAGEMENT =============
// hooks/useGlobalState.js
import { useState, useEffect } from 'react';
import { globalState } from '../lib/state-manager';

export const useGlobalState = (key, defaultValue) => {
  const [value, setValue] = useState(() => 
    globalState.hasState(key) ? globalState.getState(key) : defaultValue
  );
  
  useEffect(() => {
    // Set default value if not exists
    if (!globalState.hasState(key) && defaultValue !== undefined) {
      globalState.setState(key, defaultValue);
    }
    
    // Subscribe to changes
    const unsubscribe = globalState.subscribe(key, (newValue) => {
      setValue(newValue);
    });
    
    return unsubscribe;
  }, [key, defaultValue]);
  
  const setGlobalValue = (newValue) => {
    globalState.setState(key, newValue);
  };
  
  return [value, setGlobalValue];
};

// ============= PERMISSION SYSTEM COMPONENT =============
// components/PermissionGate.jsx
import { useEffect, useState } from 'react';

const userPermissions = new Map(); // userId -> Set of permissions
const rolePermissions = new Map(); // roleName -> Set of permissions

const PermissionGate = ({ 
  permissions = [], 
  roles = [], 
  userId, 
  requireAll = false,
  fallback = null,
  children 
}) => {
  const [hasAccess, setHasAccess] = useState(false);
  
  useEffect(() => {
    const checkPermissions = () => {
      if (!userId) {
        setHasAccess(false);
        return;
      }
      
      const userPerms = userPermissions.get(userId) || new Set();
      
      // Check direct permissions
      const hasDirectPermissions = requireAll
        ? permissions.every(perm => userPerms.has(perm))
        : permissions.some(perm => userPerms.has(perm));
      
      // Check role-based permissions
      let hasRolePermissions = false;
      if (roles.length > 0) {
        for (const role of roles) {
          const rolePerms = rolePermissions.get(role) || new Set();
          const roleMatch = requireAll
            ? permissions.every(perm => rolePerms.has(perm))
            : permissions.some(perm => rolePerms.has(perm));
          
          if (roleMatch) {
            hasRolePermissions = true;
            break;
          }
        }
      }
      
      setHasAccess(hasDirectPermissions || hasRolePermissions);
    };
    
    checkPermissions();
  }, [permissions, roles, userId, requireAll]);
  
  if (!hasAccess) {
    return fallback;
  }
  
  return children;
};

// Permission management functions
export const addUserPermission = (userId, permission) => {
  if (!userPermissions.has(userId)) {
    userPermissions.set(userId, new Set());
  }
  userPermissions.get(userId).add(permission);
};

export const addRolePermission = (roleName, permission) => {
  if (!rolePermissions.has(roleName)) {
    rolePermissions.set(roleName, new Set());
  }
  rolePermissions.get(roleName).add(permission);
};

export default PermissionGate;

// ============= CACHING COMPONENT =============
// components/CachedData.jsx
import { useState, useEffect } from 'react';

const cache = new Map();
const loadingStates = new Map();

const CachedData = ({ 
  cacheKey, 
  fetchData, 
  ttl = 300000, // 5 minutes
  children,
  loading = <div>Loading...</div>,
  error: ErrorComponent = ({ error }) => <div>Error: {error.message}</div>
}) => {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const loadData = async () => {
      // Check cache first
      const cached = cache.get(cacheKey);
      if (cached && Date.now() - cached.timestamp < ttl) {
        setData(cached.data);
        return;
      }
      
      // Check if already loading
      if (loadingStates.has(cacheKey)) {
        const existingPromise = loadingStates.get(cacheKey);
        try {
          const result = await existingPromise;
          setData(result);
        } catch (err) {
          setError(err);
        }
        return;
      }
      
      // Start loading
      setIsLoading(true);
      setError(null);
      
      const loadingPromise = fetchData();
      loadingStates.set(cacheKey, loadingPromise);
      
      try {
        const result = await loadingPromise;
        
        // Cache the result
        cache.set(cacheKey, {
          data: result,
          timestamp: Date.now()
        });
        
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setIsLoading(false);
        loadingStates.delete(cacheKey);
      }
    };
    
    loadData();
  }, [cacheKey, fetchData, ttl]);
  
  if (isLoading) {
    return loading;
  }
  
  if (error) {
    return <ErrorComponent error={error} />;
  }
  
  if (!data) {
    return null;
  }
  
  return children(data);
};

// Cache management functions
export const clearCache = (pattern) => {
  if (pattern) {
    for (const key of cache.keys()) {
      if (key.includes(pattern)) {
        cache.delete(key);
      }
    }
  } else {
    cache.clear();
  }
};

export const getCacheStats = () => ({
  size: cache.size,
  keys: [...cache.keys()],
  memory: JSON.stringify([...cache.values()]).length
});

export default CachedData;

// ============= USAGE EXAMPLE =============
// pages/dashboard.js
import { useGlobalState } from '../hooks/useGlobalState';
import PermissionGate from '../components/PermissionGate';
import CachedData from '../components/CachedData';

const Dashboard = () => {
  const [user] = useGlobalState('currentUser');
  const [theme] = useGlobalState('theme', 'light');
  
  const fetchDashboardData = async () => {
    const response = await fetch('/api/dashboard');
    return response.json();
  };
  
  return (
    <div className={`dashboard theme-${theme}`}>
      <h1>Dashboard</h1>
      
      <PermissionGate
        permissions={['read_dashboard']}
        userId={user?.id}
        fallback={<div>Access denied</div>}
      >
        <CachedData
          cacheKey="dashboard-data"
          fetchData={fetchDashboardData}
          ttl={60000} // 1 minute
        >
          {(dashboardData) => (
            <div>
              <h2>Welcome, {user?.name}</h2>
              <pre>{JSON.stringify(dashboardData, null, 2)}</pre>
            </div>
          )}
        </CachedData>
      </PermissionGate>
      
      <PermissionGate
        permissions={['admin']}
        userId={user?.id}
      >
        <div>Admin-only content</div>
      </PermissionGate>
    </div>
  );
};

export default Dashboard;
```

## 🎯 แบบฝึกหัด

### **🔣 แบบฝึกหัดที่ 1: Symbol Applications**
สร้าง advanced symbol applications:

```javascript
// TODO: สร้าง symbol-based systems

// 1. Plugin system with symbols
class PluginSystem {
  constructor() {
    // Use symbols for plugin metadata
  }
  
  registerPlugin(name, plugin) {
    // Implement plugin registration with symbol-based metadata
  }
}

// 2. Event system with symbol-based types
class SymbolEventEmitter {
  constructor() {
    // Use symbols for private event storage
  }
  
  createEventType(name) {
    // Create unique symbol for event type
  }
}

// 3. Serialization system with symbols
class SymbolSerializer {
  // Handle serialization of objects with symbol properties
}
```

### **🗺️ แบบฝึกหัดที่ 2: Advanced Map/Set Usage**
สร้าง complex data structures:

```javascript
// TODO: สร้าง advanced data structures

// 1. Graph data structure
class Graph {
  constructor() {
    this.vertices = new Set();
    this.edges = new Map(); // vertex -> Set of connected vertices
  }
  
  addVertex(vertex) {
    // Add vertex to graph
  }
  
  addEdge(from, to) {
    // Add edge between vertices
  }
  
  findPath(start, end) {
    // Find path between vertices
  }
}

// 2. Multi-level cache system
class MultiLevelCache {
  constructor() {
    this.l1Cache = new Map(); // Fast cache
    this.l2Cache = new Map(); // Slower cache
    this.weakCache = new WeakMap(); // Object-based cache
  }
  
  get(key) {
    // Implement multi-level cache lookup
  }
  
  set(key, value) {
    // Implement multi-level cache storage
  }
}

// 3. Index system
class IndexManager {
  constructor() {
    this.indexes = new Map(); // fieldName -> Map(value -> Set of records)
  }
  
  createIndex(fieldName) {
    // Create index on field
  }
  
  query(conditions) {
    // Query using indexes
  }
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง integrated system:

```jsx
// TODO: สร้าง Next.js integration

// 1. Advanced state management
const StateProvider = ({ children }) => {
  // Implement Map/Set-based state management
  // Support middleware, persistence, time travel
};

// 2. Permission-based routing
const PermissionRouter = () => {
  // Implement routing based on user permissions
  // Use Sets for permission management
};

// 3. Real-time collaboration
const CollaborationManager = () => {
  // Use WeakMap for user sessions
  // Use Sets for tracking active users
  // Implement conflict resolution
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 11: Iterators & Generators](./11-iterators-generators.md)

**บทถัดไป**: [บทที่ 13: Regular Expressions](./13-regex.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
- [MDN: Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [MDN: Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [MDN: WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Symbols** สำหรับ unique identifiers และ private properties
- **Maps** สำหรับ advanced key-value storage
- **Sets** สำหรับ unique value collections และ mathematical operations
- **WeakMap/WeakSet** สำหรับ memory-efficient data structures
- **การใช้งานใน Next.js** สำหรับ state management และ advanced features

Symbols, Maps, และ Sets เป็น data structures ที่มีประสิทธิภาพสำหรับการจัดการข้อมูลใน modern JavaScript! 🚀
