# บทที่ 3: Arrow Functions & this Binding

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Arrow Functions syntax และการใช้งาน
- ทำความเข้าใจ this binding ใน Arrow Functions
- เปรียบเทียบกับ Regular Functions
- ประยุกต์ใช้ใน React และ Next.js

## 🏹 Arrow Functions Syntax

### **📝 Basic Syntax**

```javascript
// ============= REGULAR FUNCTION =============
function add(a, b) {
  return a + b;
}

// ============= ARROW FUNCTION =============
const add = (a, b) => {
  return a + b;
};

// ============= ARROW FUNCTION (CONCISE) =============
const add = (a, b) => a + b;

// ============= SINGLE PARAMETER =============
// Regular function
function square(x) {
  return x * x;
}

// Arrow function (parentheses optional for single param)
const square = x => x * x;
const squareWithParens = (x) => x * x;

// ============= NO PARAMETERS =============
// Regular function
function getCurrentTime() {
  return new Date().toISOString();
}

// Arrow function (parentheses required)
const getCurrentTime = () => new Date().toISOString();

// ============= MULTIPLE STATEMENTS =============
const processUser = (user) => {
  const processedUser = {
    ...user,
    fullName: `${user.firstName} ${user.lastName}`,
    isActive: true,
    processedAt: new Date().toISOString()
  };
  
  console.log(`Processing user: ${processedUser.fullName}`);
  return processedUser;
};

// ============= RETURNING OBJECTS =============
// ❌ This won't work (interpreted as block)
// const createUser = (name, email) => { name, email };

// ✅ Wrap in parentheses
const createUser = (name, email) => ({ name, email });

// ✅ Or use explicit return
const createUserExplicit = (name, email) => {
  return { name, email };
};
```

### **🔄 Advanced Arrow Function Patterns**

```javascript
// ============= HIGHER-ORDER FUNCTIONS =============
const numbers = [1, 2, 3, 4, 5];

// Traditional way
const doubled = numbers.map(function(num) {
  return num * 2;
});

// Arrow function way
const doubledArrow = numbers.map(num => num * 2);

// Multiple operations
const processed = numbers
  .filter(num => num > 2)
  .map(num => num * 2)
  .reduce((sum, num) => sum + num, 0);

// ============= FUNCTION COMPOSITION =============
const pipe = (...functions) => (value) => 
  functions.reduce((acc, fn) => fn(acc), value);

const addOne = x => x + 1;
const multiplyByTwo = x => x * 2;
const square = x => x * x;

const transform = pipe(addOne, multiplyByTwo, square);
const result = transform(3); // ((3 + 1) * 2)² = 64

// ============= CURRYING =============
const multiply = (a) => (b) => a * b;
const multiplyByThree = multiply(3);
const result = multiplyByThree(4); // 12

// ============= ASYNC ARROW FUNCTIONS =============
const fetchUser = async (id) => {
  try {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error;
  }
};

// ============= IMMEDIATELY INVOKED ARROW FUNCTIONS =============
const result = ((x, y) => x + y)(10, 20); // 30

// ============= CONDITIONAL ARROW FUNCTIONS =============
const getGreeting = (timeOfDay) => 
  timeOfDay === 'morning' ? 'Good morning!' :
  timeOfDay === 'afternoon' ? 'Good afternoon!' :
  timeOfDay === 'evening' ? 'Good evening!' :
  'Hello!';

// ============= ARRAY METHODS WITH ARROW FUNCTIONS =============
const users = [
  { id: 1, name: 'John', age: 30, active: true },
  { id: 2, name: 'Jane', age: 25, active: false },
  { id: 3, name: 'Bob', age: 35, active: true }
];

// Filtering and mapping
const activeUserNames = users
  .filter(user => user.active)
  .map(user => user.name);

// Finding
const userById = (id) => users.find(user => user.id === id);

// Sorting
const sortedByAge = users.sort((a, b) => a.age - b.age);

// Grouping
const groupByStatus = users.reduce((groups, user) => {
  const status = user.active ? 'active' : 'inactive';
  return {
    ...groups,
    [status]: [...(groups[status] || []), user]
  };
}, {});
```

## 🎯 this Binding

### **🔍 this ใน Regular Functions vs Arrow Functions**

```javascript
// ============= REGULAR FUNCTION - DYNAMIC THIS =============
const regularObj = {
  name: 'Regular Object',
  
  regularMethod: function() {
    console.log('Regular method this:', this.name);
    
    // this ใน nested function จะเป็น undefined (strict mode) หรือ global object
    function nestedFunction() {
      console.log('Nested function this:', this); // undefined หรือ window/global
    }
    nestedFunction();
    
    // Workaround ด้วย bind หรือ call
    const boundNested = nestedFunction.bind(this);
    boundNested();
    
    // Workaround ด้วย variable
    const self = this;
    function nestedWithSelf() {
      console.log('Nested with self:', self.name);
    }
    nestedWithSelf();
  }
};

// ============= ARROW FUNCTION - LEXICAL THIS =============
const arrowObj = {
  name: 'Arrow Object',
  
  arrowMethod: function() {
    console.log('Arrow method this:', this.name);
    
    // Arrow function inherits this from enclosing scope
    const nestedArrow = () => {
      console.log('Nested arrow this:', this.name); // 'Arrow Object'
    };
    nestedArrow();
    
    // Multiple levels
    const deepNested = () => {
      const evenDeeper = () => {
        console.log('Deep nested arrow this:', this.name); // 'Arrow Object'
      };
      evenDeeper();
    };
    deepNested();
  },
  
  // ❌ DON'T: Arrow function as object method
  wrongArrowMethod: () => {
    console.log('Wrong arrow method this:', this); // undefined หรือ global object
  }
};

// ============= CLASS METHODS =============
class UserService {
  constructor(apiUrl) {
    this.apiUrl = apiUrl;
    this.cache = new Map();
  }
  
  // Regular method
  fetchUser(id) {
    console.log('Fetching from:', this.apiUrl);
    return fetch(`${this.apiUrl}/users/${id}`);
  }
  
  // Arrow method (property with arrow function)
  fetchUserArrow = (id) => {
    console.log('Arrow fetching from:', this.apiUrl);
    return fetch(`${this.apiUrl}/users/${id}`);
  }
  
  // Method that uses callbacks
  processUsers(users) {
    // ❌ Regular function callback - this binding issue
    users.forEach(function(user) {
      // this.cache.set(user.id, user); // Error: this is undefined
    });
    
    // ✅ Arrow function callback - this preserved
    users.forEach((user) => {
      this.cache.set(user.id, user); // Works correctly
    });
    
    // ✅ Alternative: bind the function
    users.forEach(function(user) {
      this.cache.set(user.id, user);
    }.bind(this));
  }
}

// ============= EVENT HANDLERS =============
class ButtonComponent {
  constructor(element) {
    this.element = element;
    this.clickCount = 0;
    
    // ❌ Regular function - this binding problem
    this.element.addEventListener('click', function() {
      // this.clickCount++; // Error: this refers to the element
      console.log('Regular handler this:', this); // HTMLElement
    });
    
    // ✅ Arrow function - this preserved
    this.element.addEventListener('click', () => {
      this.clickCount++;
      console.log('Arrow handler this:', this); // ButtonComponent instance
      console.log('Click count:', this.clickCount);
    });
    
    // ✅ Bound regular function
    this.element.addEventListener('click', this.handleClick.bind(this));
  }
  
  handleClick() {
    this.clickCount++;
    console.log('Bound method click count:', this.clickCount);
  }
}
```

### **⚛️ React และ this Binding**

```jsx
// ============= CLASS COMPONENTS =============
import React, { Component } from 'react';

class OldClassComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      user: null
    };
    
    // ต้อง bind ใน constructor
    this.handleClick = this.handleClick.bind(this);
  }
  
  // Regular method - ต้อง bind
  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }
  
  // Arrow function property - auto-bound
  handleArrowClick = () => {
    this.setState({ count: this.state.count + 1 });
  }
  
  // Async method with proper binding
  fetchUser = async (id) => {
    try {
      const response = await fetch(`/api/users/${id}`);
      const user = await response.json();
      this.setState({ user });
    } catch (error) {
      console.error('Error:', error);
    }
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        
        {/* ❌ Binding ใน render (performance issue) */}
        <button onClick={this.handleClick.bind(this)}>
          Regular Method (Bad)
        </button>
        
        {/* ❌ Arrow function ใน render (performance issue) */}
        <button onClick={() => this.handleClick()}>
          Arrow in Render (Bad)
        </button>
        
        {/* ✅ Pre-bound method */}
        <button onClick={this.handleClick}>
          Pre-bound Method
        </button>
        
        {/* ✅ Arrow function property */}
        <button onClick={this.handleArrowClick}>
          Arrow Function Property
        </button>
      </div>
    );
  }
}

// ============= FUNCTION COMPONENTS (MODERN) =============
import { useState, useCallback } from 'react';

const ModernFunctionComponent = () => {
  const [count, setCount] = useState(0);
  const [user, setUser] = useState(null);
  
  // ✅ Arrow functions ใน function components
  const handleIncrement = () => {
    setCount(prev => prev + 1);
  };
  
  const handleDecrement = () => {
    setCount(prev => prev - 1);
  };
  
  // ✅ Async arrow function
  const fetchUser = async (id) => {
    try {
      const response = await fetch(`/api/users/${id}`);
      const userData = await response.json();
      setUser(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
    }
  };
  
  // ✅ useCallback with arrow function
  const memoizedFetchUser = useCallback(async (id) => {
    await fetchUser(id);
  }, []);
  
  // ✅ Higher-order functions
  const handleInputChange = (field) => (event) => {
    const value = event.target.value;
    setUser(prev => ({ ...prev, [field]: value }));
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={handleDecrement}>Decrement</button>
      
      {/* ✅ Arrow functions work perfectly in function components */}
      <button onClick={() => setCount(0)}>Reset</button>
      
      {/* ✅ Higher-order function usage */}
      <input 
        placeholder="Name"
        onChange={handleInputChange('name')}
      />
      
      <input 
        placeholder="Email"
        onChange={handleInputChange('email')}
      />
      
      <button onClick={() => memoizedFetchUser(1)}>
        Fetch User
      </button>
      
      {user && <div>User: {user.name}</div>}
    </div>
  );
};
```

## 🚀 Next.js Applications

### **📡 API Routes และ Server-side Code**

```javascript
// pages/api/users/index.js
const users = []; // In-memory storage for demo

// ============= API HANDLER WITH ARROW FUNCTIONS =============
export default async (req, res) => {
  const { method } = req;
  
  // ✅ Arrow functions ใน API routes
  const handleGet = async () => {
    const { page = 1, limit = 10 } = req.query;
    
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + parseInt(limit);
    const paginatedUsers = users.slice(startIndex, endIndex);
    
    return res.status(200).json({
      users: paginatedUsers,
      total: users.length,
      page: parseInt(page),
      hasMore: endIndex < users.length
    });
  };
  
  const handlePost = async () => {
    const userData = req.body;
    
    // Validation with arrow functions
    const validateUser = (user) => {
      const errors = [];
      
      if (!user.name?.trim()) errors.push('Name is required');
      if (!user.email?.includes('@')) errors.push('Valid email is required');
      
      return {
        isValid: errors.length === 0,
        errors
      };
    };
    
    const validation = validateUser(userData);
    
    if (!validation.isValid) {
      return res.status(400).json({ errors: validation.errors });
    }
    
    const newUser = {
      id: Date.now(),
      ...userData,
      createdAt: new Date().toISOString()
    };
    
    users.push(newUser);
    res.status(201).json(newUser);
  };
  
  // Method routing with arrow functions
  const methodHandlers = {
    GET: handleGet,
    POST: handlePost
  };
  
  const handler = methodHandlers[method];
  
  if (handler) {
    await handler();
  } else {
    res.setHeader('Allow', Object.keys(methodHandlers));
    res.status(405).json({ error: `Method ${method} not allowed` });
  }
};

// ============= UTILITY FUNCTIONS =============
// utils/apiHelpers.js
export const createApiHandler = (handlers) => async (req, res) => {
  const { method } = req;
  
  try {
    const handler = handlers[method];
    
    if (!handler) {
      res.setHeader('Allow', Object.keys(handlers));
      return res.status(405).json({ error: `Method ${method} not allowed` });
    }
    
    await handler(req, res);
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
};

// Validation helpers
export const validators = {
  required: (value, fieldName) => 
    value?.trim() ? null : `${fieldName} is required`,
    
  email: (value) => 
    value?.includes('@') ? null : 'Valid email is required',
    
  minLength: (min) => (value) =>
    value?.length >= min ? null : `Must be at least ${min} characters`,
    
  custom: (validatorFn, message) => (value) =>
    validatorFn(value) ? null : message
};

export const validateFields = (data, rules) => {
  const errors = {};
  
  Object.entries(rules).forEach(([field, fieldRules]) => {
    const value = data[field];
    const fieldErrors = fieldRules
      .map(rule => rule(value, field))
      .filter(error => error !== null);
      
    if (fieldErrors.length > 0) {
      errors[field] = fieldErrors;
    }
  });
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
};

// การใช้งาน validation
// pages/api/users/create.js
import { createApiHandler, validators, validateFields } from '../../../utils/apiHelpers';

const userValidationRules = {
  name: [validators.required, validators.minLength(2)],
  email: [validators.required, validators.email],
  age: [validators.custom(age => age >= 0 && age <= 150, 'Age must be between 0 and 150')]
};

export default createApiHandler({
  POST: async (req, res) => {
    const validation = validateFields(req.body, userValidationRules);
    
    if (!validation.isValid) {
      return res.status(400).json({ errors: validation.errors });
    }
    
    // Create user logic here
    const newUser = {
      id: Date.now(),
      ...req.body,
      createdAt: new Date().toISOString()
    };
    
    res.status(201).json(newUser);
  }
});
```

### **🔄 Data Fetching Patterns**

```javascript
// lib/api.js
class ApiClient {
  constructor(baseUrl, options = {}) {
    this.baseUrl = baseUrl;
    this.defaultOptions = {
      headers: {
        'Content-Type': 'application/json'
      },
      ...options
    };
  }
  
  // ✅ Arrow function methods preserve 'this'
  request = async (endpoint, options = {}) => {
    const url = `${this.baseUrl}${endpoint}`;
    const config = {
      ...this.defaultOptions,
      ...options,
      headers: {
        ...this.defaultOptions.headers,
        ...options.headers
      }
    };
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
  
  get = async (endpoint, params = {}) => {
    const searchParams = new URLSearchParams(params);
    const url = searchParams.toString() 
      ? `${endpoint}?${searchParams}` 
      : endpoint;
      
    return this.request(url);
  }
  
  post = async (endpoint, data) => {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
  
  put = async (endpoint, data) => {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
  
  delete = async (endpoint) => {
    return this.request(endpoint, {
      method: 'DELETE'
    });
  }
}

// สร้าง instance
const api = new ApiClient('/api');

// ============= DATA SERVICES =============
export const userService = {
  // ✅ Arrow functions ใน object methods
  getAll: async (filters = {}) => {
    return api.get('/users', filters);
  },
  
  getById: async (id) => {
    return api.get(`/users/${id}`);
  },
  
  create: async (userData) => {
    return api.post('/users', userData);
  },
  
  update: async (id, updates) => {
    return api.put(`/users/${id}`, updates);
  },
  
  delete: async (id) => {
    return api.delete(`/users/${id}`);
  },
  
  // Advanced operations
  bulkUpdate: async (updates) => {
    return api.post('/users/bulk-update', { updates });
  },
  
  search: async (query, options = {}) => {
    return api.get('/users/search', { q: query, ...options });
  }
};

// ============= CUSTOM HOOKS =============
// hooks/useUsers.js
import { useState, useEffect, useCallback } from 'react';
import { userService } from '../lib/api';

export const useUsers = (initialFilters = {}) => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [filters, setFilters] = useState(initialFilters);
  
  // ✅ Arrow functions ใน hooks
  const fetchUsers = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const data = await userService.getAll(filters);
      setUsers(data.users || data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [filters]);
  
  const addUser = useCallback(async (userData) => {
    try {
      const newUser = await userService.create(userData);
      setUsers(prev => [...prev, newUser]);
      return newUser;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  }, []);
  
  const updateUser = useCallback(async (id, updates) => {
    try {
      const updatedUser = await userService.update(id, updates);
      setUsers(prev => prev.map(user => 
        user.id === id ? updatedUser : user
      ));
      return updatedUser;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  }, []);
  
  const deleteUser = useCallback(async (id) => {
    try {
      await userService.delete(id);
      setUsers(prev => prev.filter(user => user.id !== id));
    } catch (err) {
      setError(err.message);
      throw err;
    }
  }, []);
  
  const updateFilters = useCallback((newFilters) => {
    setFilters(prev => ({ ...prev, ...newFilters }));
  }, []);
  
  const clearFilters = useCallback(() => {
    setFilters({});
  }, []);
  
  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);
  
  return {
    users,
    loading,
    error,
    filters,
    actions: {
      refresh: fetchUsers,
      add: addUser,
      update: updateUser,
      delete: deleteUser,
      updateFilters,
      clearFilters
    }
  };
};

// การใช้งาน useUsers hook
const UserManagement = () => {
  const { 
    users, 
    loading, 
    error, 
    filters,
    actions 
  } = useUsers({ active: true });
  
  // ✅ Event handlers ด้วย arrow functions
  const handleAddUser = async (userData) => {
    try {
      await actions.add(userData);
      console.log('User added successfully');
    } catch (error) {
      console.error('Failed to add user:', error);
    }
  };
  
  const handleUpdateUser = async (id, updates) => {
    try {
      await actions.update(id, updates);
      console.log('User updated successfully');
    } catch (error) {
      console.error('Failed to update user:', error);
    }
  };
  
  const handleDeleteUser = async (id) => {
    if (confirm('Are you sure you want to delete this user?')) {
      try {
        await actions.delete(id);
        console.log('User deleted successfully');
      } catch (error) {
        console.error('Failed to delete user:', error);
      }
    }
  };
  
  const handleFilterChange = (field, value) => {
    actions.updateFilters({ [field]: value });
  };
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div>
      <h1>User Management</h1>
      
      {/* Filters */}
      <div className="filters">
        <input
          type="text"
          placeholder="Search users..."
          onChange={(e) => handleFilterChange('search', e.target.value)}
        />
        
        <select onChange={(e) => handleFilterChange('status', e.target.value)}>
          <option value="">All Status</option>
          <option value="active">Active</option>
          <option value="inactive">Inactive</option>
        </select>
        
        <button onClick={actions.clearFilters}>Clear Filters</button>
      </div>
      
      {/* User List */}
      <div className="user-list">
        {users.map(user => (
          <div key={user.id} className="user-card">
            <h3>{user.name}</h3>
            <p>{user.email}</p>
            
            <div className="actions">
              <button onClick={() => handleUpdateUser(user.id, { status: 'inactive' })}>
                Deactivate
              </button>
              <button onClick={() => handleDeleteUser(user.id)}>
                Delete
              </button>
            </div>
          </div>
        ))}
      </div>
      
      <button onClick={() => handleAddUser({ 
        name: 'New User', 
        email: 'new@example.com' 
      })}>
        Add Sample User
      </button>
    </div>
  );
};
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Function Conversion**
แปลง regular functions เป็น arrow functions:

```javascript
// แปลงฟังก์ชันเหล่านี้เป็น arrow functions
function calculateTotal(items) {
  return items.reduce(function(sum, item) {
    return sum + (item.price * item.quantity);
  }, 0);
}

function processData(data) {
  return data
    .filter(function(item) {
      return item.active === true;
    })
    .map(function(item) {
      return {
        id: item.id,
        name: item.name.toUpperCase(),
        value: item.value * 2
      };
    })
    .sort(function(a, b) {
      return a.name.localeCompare(b.name);
    });
}

// สร้าง object ที่มี methods เป็น arrow functions
const calculator = {
  // แปลงเป็น arrow functions
  add: function(a, b) {
    return a + b;
  },
  multiply: function(a, b) {
    return a * b;
  }
};
```

### **⚛️ แบบฝึกหัดที่ 2: React Component**
สร้าง React component ที่ใช้ arrow functions อย่างถูกต้อง:

```jsx
// สร้าง TodoList component ที่:
// 1. ใช้ arrow functions สำหรับ event handlers
// 2. จัดการ state ด้วย hooks
// 3. มี functions สำหรับ add, toggle, delete todos
// 4. ใช้ higher-order functions ใน data processing

const TodoList = () => {
  // Implementation here
};
```

### **🌐 แบบฝึกหัดที่ 3: API Service**
สร้าง API service class ที่ใช้ arrow functions:

```javascript
// สร้าง class PostService ที่:
// 1. ใช้ arrow function methods
// 2. มี proper this binding
// 3. Handle async operations
// 4. Include error handling

class PostService {
  constructor(apiUrl) {
    this.apiUrl = apiUrl;
  }
  
  // Implementation here
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 2: Variables & Scope](./02-variables-scope.md)

**บทถัดไป**: [บทที่ 4: Destructuring Assignment](./04-destructuring.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [JavaScript.info: Arrow Functions](https://javascript.info/arrow-functions)
- [React: Handling Events](https://react.dev/learn/responding-to-events)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Arrow Functions syntax** และรูปแบบต่างๆ
- **this binding** และความแตกต่างจาก regular functions
- **การใช้งานใน React** และ event handling
- **Best practices** สำหรับ arrow functions
- **การประยุกต์ใช้ใน Next.js** applications

Arrow functions ช่วยให้โค้ดสั้น อ่านง่าย และแก้ปัญหา this binding ได้อย่างมีประสิทธิภาพ! 🚀
