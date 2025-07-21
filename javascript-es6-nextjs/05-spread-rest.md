# บทที่ 5: Spread & Rest Operators

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้การใช้งาน Spread Operator (`...`)
- ทำความเข้าใจ Rest Parameters
- เรียนรู้การใช้งานใน Arrays และ Objects
- ประยุกต์ใช้ใน Next.js และ React

## 🌟 Spread Operator (`...`)

### **📦 การใช้งานกับ Arrays**

```javascript
// ============= BASIC ARRAY SPREADING =============
const fruits = ['apple', 'banana'];
const vegetables = ['carrot', 'spinach'];

// รวม arrays
const food = [...fruits, ...vegetables];
// Result: ['apple', 'banana', 'carrot', 'spinach']

// เพิ่มสมาชิกใหม่
const moreFruits = [...fruits, 'orange', 'grape'];
// Result: ['apple', 'banana', 'orange', 'grape']

// Copy array (shallow copy)
const fruitsCopy = [...fruits];

// ============= ARRAY MANIPULATION =============
const numbers = [1, 2, 3];

// เพิ่มที่จุดเริ่มต้น
const withZero = [0, ...numbers];
// Result: [0, 1, 2, 3]

// เพิ่มตรงกลาง
const withMiddle = [...numbers.slice(0, 2), 2.5, ...numbers.slice(2)];
// Result: [1, 2, 2.5, 3]

// แปลง NodeList หรือ arguments เป็น Array
const divElements = [...document.querySelectorAll('div')];

// ============= FUNCTION ARGUMENTS =============
const maxNumber = Math.max(...numbers);
// แทนที่ Math.max.apply(null, numbers)

const minNumber = Math.min(...numbers);

// Push multiple elements
const originalArray = [1, 2];
const newElements = [3, 4, 5];
originalArray.push(...newElements);
// originalArray: [1, 2, 3, 4, 5]
```

### **🏗️ การใช้งานกับ Objects**

```javascript
// ============= BASIC OBJECT SPREADING =============
const person = {
  name: 'John',
  age: 30
};

const address = {
  city: 'Bangkok',
  country: 'Thailand'
};

// รวม objects
const fullProfile = {
  ...person,
  ...address,
  occupation: 'Developer'
};
// Result: { name: 'John', age: 30, city: 'Bangkok', country: 'Thailand', occupation: 'Developer' }

// ============= OBJECT CLONING =============
// Shallow clone
const personCopy = { ...person };

// Deep clone (simple objects only)
const deepClone = { ...person, hobbies: [...person.hobbies] };

// ============= PROPERTY OVERRIDE =============
const updatedPerson = {
  ...person,
  age: 31, // Override existing property
  status: 'active' // Add new property
};

// ============= CONDITIONAL SPREADING =============
const user = {
  name: 'Alice',
  email: 'alice@example.com'
};

const isAdmin = true;
const userWithRole = {
  ...user,
  ...(isAdmin && { role: 'admin', permissions: ['read', 'write'] })
};

// ============= REMOVING PROPERTIES =============
const { password, ...userWithoutPassword } = user;
// userWithoutPassword contains all properties except password

// ============= NESTED OBJECT UPDATES =============
const state = {
  user: {
    profile: {
      name: 'John',
      email: 'john@example.com'
    },
    preferences: {
      theme: 'dark'
    }
  }
};

// Update nested property
const newState = {
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      name: 'John Doe'
    }
  }
};
```

## 🔧 Rest Parameters

### **📝 Function Parameters**

```javascript
// ============= BASIC REST PARAMETERS =============
// รับ parameters ไม่จำกัดจำนวน
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4, 5); // Result: 15

// ============= MIXED PARAMETERS =============
function greetUsers(greeting, ...names) {
  return names.map(name => `${greeting}, ${name}!`);
}

greetUsers('Hello', 'Alice', 'Bob', 'Charlie');
// Result: ['Hello, Alice!', 'Hello, Bob!', 'Hello, Charlie!']

// ============= OBJECT DESTRUCTURING WITH REST =============
function processUser({ name, email, ...otherInfo }) {
  console.log(`Processing user: ${name} (${email})`);
  console.log('Additional info:', otherInfo);
}

processUser({
  name: 'John',
  email: 'john@example.com',
  age: 30,
  city: 'Bangkok',
  occupation: 'Developer'
});

// ============= ARRAY DESTRUCTURING WITH REST =============
function processItems([first, second, ...remaining]) {
  console.log('First item:', first);
  console.log('Second item:', second);
  console.log('Remaining items:', remaining);
}

processItems(['apple', 'banana', 'orange', 'grape', 'kiwi']);

// ============= ADVANCED REST PATTERNS =============
// Rest in object destructuring
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  debug: true,
  cache: false
};

const { apiUrl, timeout, ...otherConfig } = config;

// Rest in function with destructuring
function createApiClient({ baseURL, timeout = 5000, ...options }) {
  return {
    baseURL,
    timeout,
    request: (url, config) => fetch(`${baseURL}${url}`, { 
      timeout, 
      ...options, 
      ...config 
    })
  };
}
```

## 🚀 Next.js และ React Applications

### **⚛️ Component Props Handling**

```jsx
// components/Button.jsx
import { forwardRef } from 'react';

// ใช้ rest parameters รับ props ทั้งหมด
const Button = forwardRef(({ 
  variant = 'primary',
  size = 'medium',
  children,
  className = '',
  ...restProps 
}, ref) => {
  const baseClasses = 'px-4 py-2 rounded font-medium transition-colors';
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-600 text-white hover:bg-red-700'
  };
  
  const sizeClasses = {
    small: 'text-sm px-2 py-1',
    medium: 'text-base px-4 py-2',
    large: 'text-lg px-6 py-3'
  };

  const buttonClasses = [
    baseClasses,
    variantClasses[variant],
    sizeClasses[size],
    className
  ].join(' ');

  return (
    <button
      ref={ref}
      className={buttonClasses}
      {...restProps} // ส่ง props อื่นๆ ต่อไป (onClick, disabled, etc.)
    >
      {children}
    </button>
  );
});

Button.displayName = 'Button';

// การใช้งาน
const App = () => {
  return (
    <div>
      <Button 
        variant="primary"
        size="large"
        onClick={() => console.log('Clicked!')}
        disabled={false}
        data-testid="submit-button"
      >
        Submit
      </Button>
    </div>
  );
};

export default Button;
```

### **🔄 State Management Patterns**

```jsx
// hooks/useLocalStorage.js
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      // ใช้ spread เพื่อรักษา reference equality
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      
      // ถ้าเป็น object ใช้ spread ก่อน stringify
      const serializedValue = typeof valueToStore === 'object' 
        ? JSON.stringify({ ...valueToStore })
        : JSON.stringify(valueToStore);
        
      window.localStorage.setItem(key, serializedValue);
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue];
}

// components/UserProfile.jsx
import { useState } from 'react';
import useLocalStorage from '../hooks/useLocalStorage';

const UserProfile = () => {
  const [profile, setProfile] = useLocalStorage('userProfile', {
    name: '',
    email: '',
    preferences: {
      theme: 'light',
      notifications: true
    }
  });

  // Update ด้วย spread operator
  const updateProfile = (updates) => {
    setProfile(prev => ({ ...prev, ...updates }));
  };

  const updatePreferences = (newPreferences) => {
    setProfile(prev => ({
      ...prev,
      preferences: { ...prev.preferences, ...newPreferences }
    }));
  };

  // Reset to default
  const resetProfile = () => {
    const defaultProfile = {
      name: '',
      email: '',
      preferences: {
        theme: 'light',
        notifications: true
      }
    };
    setProfile({ ...defaultProfile });
  };

  return (
    <div className="user-profile">
      <h1>User Profile</h1>
      
      <form onSubmit={(e) => {
        e.preventDefault();
        const formData = new FormData(e.target);
        const updates = Object.fromEntries(formData.entries());
        updateProfile(updates);
      }}>
        <input
          name="name"
          type="text"
          placeholder="Name"
          value={profile.name}
          onChange={(e) => updateProfile({ name: e.target.value })}
        />
        
        <input
          name="email"
          type="email"
          placeholder="Email"
          value={profile.email}
          onChange={(e) => updateProfile({ email: e.target.value })}
        />
        
        <div className="preferences">
          <h3>Preferences</h3>
          
          <label>
            <input
              type="radio"
              name="theme"
              value="light"
              checked={profile.preferences.theme === 'light'}
              onChange={(e) => updatePreferences({ theme: e.target.value })}
            />
            Light Theme
          </label>
          
          <label>
            <input
              type="radio"
              name="theme"
              value="dark"
              checked={profile.preferences.theme === 'dark'}
              onChange={(e) => updatePreferences({ theme: e.target.value })}
            />
            Dark Theme
          </label>
          
          <label>
            <input
              type="checkbox"
              checked={profile.preferences.notifications}
              onChange={(e) => updatePreferences({ 
                notifications: e.target.checked 
              })}
            />
            Enable Notifications
          </label>
        </div>

        <div className="actions">
          <button type="submit">Save Profile</button>
          <button type="button" onClick={resetProfile}>
            Reset to Default
          </button>
        </div>
      </form>

      <div className="profile-preview">
        <h3>Profile Preview:</h3>
        <pre>{JSON.stringify(profile, null, 2)}</pre>
      </div>
    </div>
  );
};

export default UserProfile;
```

### **📊 Data Processing Patterns**

```javascript
// utils/dataProcessing.js

// ============= ARRAY OPERATIONS =============
export const arrayUtils = {
  // Merge multiple arrays
  mergeArrays: (...arrays) => arrays.flat(),
  
  // Unique values from multiple arrays
  uniqueValues: (...arrays) => [...new Set(arrays.flat())],
  
  // Group by property
  groupBy: (array, keySelector) => {
    return array.reduce((groups, item) => {
      const key = typeof keySelector === 'function' 
        ? keySelector(item) 
        : item[keySelector];
      
      return {
        ...groups,
        [key]: [...(groups[key] || []), item]
      };
    }, {});
  },

  // Chunk array into smaller arrays
  chunk: (array, size) => {
    const chunks = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push([...array.slice(i, i + size)]);
    }
    return chunks;
  },

  // Paginate array
  paginate: (array, page, pageSize) => {
    const startIndex = (page - 1) * pageSize;
    const endIndex = startIndex + pageSize;
    
    return {
      data: [...array.slice(startIndex, endIndex)],
      currentPage: page,
      totalPages: Math.ceil(array.length / pageSize),
      totalItems: array.length,
      hasNextPage: endIndex < array.length,
      hasPrevPage: page > 1
    };
  }
};

// ============= OBJECT OPERATIONS =============
export const objectUtils = {
  // Deep merge objects
  deepMerge: (target, ...sources) => {
    if (!sources.length) return target;
    const source = sources.shift();

    if (typeof target === 'object' && typeof source === 'object') {
      Object.keys(source).forEach(key => {
        if (typeof source[key] === 'object' && source[key] !== null) {
          if (!target[key]) target[key] = {};
          target[key] = objectUtils.deepMerge(target[key], source[key]);
        } else {
          target[key] = source[key];
        }
      });
    }

    return objectUtils.deepMerge(target, ...sources);
  },

  // Pick specific properties
  pick: (obj, ...keys) => {
    return keys.flat().reduce((result, key) => {
      if (key in obj) {
        result[key] = obj[key];
      }
      return result;
    }, {});
  },

  // Omit specific properties
  omit: (obj, ...keys) => {
    const keysToOmit = keys.flat();
    return Object.keys(obj).reduce((result, key) => {
      if (!keysToOmit.includes(key)) {
        result[key] = obj[key];
      }
      return result;
    }, {});
  },

  // Transform object values
  mapValues: (obj, transformer) => {
    return Object.keys(obj).reduce((result, key) => ({
      ...result,
      [key]: transformer(obj[key], key, obj)
    }), {});
  },

  // Filter object by predicate
  filterObject: (obj, predicate) => {
    return Object.keys(obj).reduce((result, key) => {
      if (predicate(obj[key], key, obj)) {
        result[key] = obj[key];
      }
      return result;
    }, {});
  }
};

// ============= API UTILITIES =============
export const apiUtils = {
  // Build query string from object
  buildQueryString: (params) => {
    const searchParams = new URLSearchParams();
    
    Object.entries(params).forEach(([key, value]) => {
      if (Array.isArray(value)) {
        value.forEach(item => searchParams.append(key, item));
      } else if (value !== null && value !== undefined) {
        searchParams.append(key, value);
      }
    });

    return searchParams.toString();
  },

  // Merge request options
  mergeRequestOptions: (defaultOptions, ...optionSets) => {
    return optionSets.reduce((merged, options) => ({
      ...merged,
      ...options,
      headers: {
        ...merged.headers,
        ...options.headers
      }
    }), { ...defaultOptions });
  },

  // Create fetch wrapper with defaults
  createFetcher: (baseURL, defaultOptions = {}) => {
    return async (endpoint, options = {}) => {
      const url = new URL(endpoint, baseURL);
      const finalOptions = apiUtils.mergeRequestOptions(
        {
          headers: {
            'Content-Type': 'application/json'
          }
        },
        defaultOptions,
        options
      );

      const response = await fetch(url.toString(), finalOptions);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      return response.json();
    };
  }
};

// การใช้งาน utilities
const users = [
  { id: 1, name: 'John', department: 'IT', age: 30 },
  { id: 2, name: 'Jane', department: 'HR', age: 25 },
  { id: 3, name: 'Bob', department: 'IT', age: 35 }
];

// Group users by department
const usersByDepartment = arrayUtils.groupBy(users, 'department');

// Get unique departments
const departments = arrayUtils.uniqueValues(users.map(u => u.department));

// Create paginated response
const paginatedUsers = arrayUtils.paginate(users, 1, 2);

// Filter and transform user data
const activeUsers = users.map(user => ({
  ...user,
  isActive: true,
  fullProfile: {
    ...objectUtils.pick(user, 'name', 'age'),
    metadata: {
      createdAt: new Date().toISOString(),
      department: user.department
    }
  }
}));
```

### **🌐 API Integration Examples**

```javascript
// services/userService.js
import { apiUtils } from '../utils/dataProcessing';

const API_BASE = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000/api';

// Create fetcher with default options
const fetcher = apiUtils.createFetcher(API_BASE, {
  headers: {
    'Accept': 'application/json'
  }
});

export const userService = {
  // Get all users with optional filters
  getUsers: async (filters = {}) => {
    const queryString = apiUtils.buildQueryString(filters);
    const endpoint = queryString ? `/users?${queryString}` : '/users';
    return fetcher(endpoint);
  },

  // Get user by ID
  getUser: async (id) => {
    return fetcher(`/users/${id}`);
  },

  // Create new user
  createUser: async (userData) => {
    return fetcher('/users', {
      method: 'POST',
      body: JSON.stringify(userData)
    });
  },

  // Update user (partial update)
  updateUser: async (id, updates) => {
    return fetcher(`/users/${id}`, {
      method: 'PATCH',
      body: JSON.stringify(updates)
    });
  },

  // Bulk update users
  bulkUpdateUsers: async (userUpdates) => {
    return fetcher('/users/bulk', {
      method: 'PATCH',
      body: JSON.stringify({ updates: userUpdates })
    });
  },

  // Delete user
  deleteUser: async (id) => {
    return fetcher(`/users/${id}`, {
      method: 'DELETE'
    });
  },

  // Upload user avatar
  uploadAvatar: async (userId, file) => {
    const formData = new FormData();
    formData.append('avatar', file);
    formData.append('userId', userId);

    return fetch(`${API_BASE}/users/${userId}/avatar`, {
      method: 'POST',
      body: formData // ไม่ต้องใส่ Content-Type header
    }).then(response => response.json());
  }
};

// pages/api/users/bulk.js - Bulk operations API
export default async function handler(req, res) {
  if (req.method !== 'PATCH') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { updates } = req.body;

    // Validate updates array
    if (!Array.isArray(updates)) {
      return res.status(400).json({ error: 'Updates must be an array' });
    }

    // Process each update
    const results = await Promise.allSettled(
      updates.map(async ({ id, ...updateData }) => {
        // Find existing user
        const existingUser = await getUserById(id);
        if (!existingUser) {
          throw new Error(`User with ID ${id} not found`);
        }

        // Merge updates with existing data
        const updatedUser = {
          ...existingUser,
          ...updateData,
          updatedAt: new Date().toISOString()
        };

        // Save updated user
        return saveUser(updatedUser);
      })
    );

    // Separate successful and failed updates
    const successful = [];
    const failed = [];

    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        successful.push(result.value);
      } else {
        failed.push({
          index,
          id: updates[index].id,
          error: result.reason.message
        });
      }
    });

    res.status(200).json({
      message: 'Bulk update completed',
      successful: successful.length,
      failed: failed.length,
      results: { successful, failed }
    });

  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
}

// Mock database functions
async function getUserById(id) {
  // Simulate database query
  return { id, name: 'John Doe', email: 'john@example.com' };
}

async function saveUser(user) {
  // Simulate database save
  return { ...user, saved: true };
}
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Array Operations**
สร้างฟังก์ชันต่อไปนี้โดยใช้ spread operator:

```javascript
// 1. ฟังก์ชันรวม arrays หลายตัว
function combineArrays(...arrays) {
  // Your implementation here
}

// 2. ฟังก์ชันเพิ่มสมาชิกที่จุดใดก็ได้ใน array
function insertAt(array, index, ...items) {
  // Your implementation here
}

// 3. ฟังก์ชันลบสมาชิกและใส่สมาชิกใหม่
function replaceItems(array, startIndex, deleteCount, ...newItems) {
  // Your implementation here
}

// Test cases
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const arr3 = [7, 8, 9];

console.log(combineArrays(arr1, arr2, arr3)); // [1,2,3,4,5,6,7,8,9]
console.log(insertAt(arr1, 1, 'a', 'b')); // [1,'a','b',2,3]
console.log(replaceItems(arr1, 1, 1, 'x', 'y')); // [1,'x','y',3]
```

### **🏗️ แบบฝึกหัดที่ 2: Object Manipulation**
สร้าง utility functions สำหรับจัดการ objects:

```javascript
// 1. Merge objects with conflict resolution
function mergeWithConflictResolution(target, source, resolver) {
  // resolver(targetValue, sourceValue, key) => finalValue
  // Your implementation here
}

// 2. Deep clone object
function deepClone(obj) {
  // Your implementation here
}

// 3. Transform object structure
function transformObject(obj, transformer) {
  // transformer(value, key, path) => newValue
  // Your implementation here
}

// Test cases
const obj1 = { a: 1, b: 2, nested: { x: 10 } };
const obj2 = { b: 3, c: 4, nested: { y: 20 } };

const merged = mergeWithConflictResolution(obj1, obj2, (target, source) => target + source);
const cloned = deepClone(obj1);
const transformed = transformObject(obj1, (value, key) => 
  typeof value === 'number' ? value * 2 : value
);
```

### **⚛️ แบบฝึกหัดที่ 3: React Component**
สร้าง flexible React component ที่ใช้ spread operator:

```jsx
// สร้าง Card component ที่รับ props แบบ flexible
const Card = ({ title, children, className, variant = 'default', ...props }) => {
  // Implementation here
  // ต้องรองรับ variants: default, outlined, elevated
  // ต้องส่ง props อื่นๆ ไปยัง DOM element
};

// สร้าง custom hook สำหรับจัดการ form data
const useFormData = (initialData = {}) => {
  // Implementation here
  // Return: { data, updateField, updateFields, reset }
};

// ใช้งาน Card และ useFormData ร่วมกัน
const ContactForm = () => {
  // Implementation here
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 4: Destructuring Assignment](./04-destructuring.md)

**บทถัดไป**: [บทที่ 6: Template Literals & String Methods](./06-template-literals.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Spread Syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
- [MDN: Rest Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
- [React: Passing Props](https://react.dev/learn/passing-props-to-a-component)
- [JavaScript.info: Rest/Spread](https://javascript.info/rest-parameters-spread)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Spread Operator** สำหรับ arrays และ objects
- **Rest Parameters** ในฟังก์ชัน
- **การใช้งานใน React** components และ props
- **Patterns ขั้นสูง** สำหรับ data manipulation
- **API integration** patterns ด้วย spread/rest

Spread และ Rest operators เป็น features ที่มีประโยชน์มากใน Next.js development เพราะช่วยให้โค้ดสั้น อ่านง่าย และมีประสิทธิภาพ! 🚀
