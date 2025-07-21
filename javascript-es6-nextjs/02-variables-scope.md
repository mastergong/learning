# บทที่ 2: Variables & Scope

## 🎯 จุดประสงค์ของบทเรียน
- ทำความเข้าใจความแตกต่างระหว่าง var, let, และ const
- เรียนรู้ Block Scope และ Function Scope
- เข้าใจ Temporal Dead Zone
- ประยุกต์ใช้ใน Next.js development

## 🔍 Variable Declarations

### **📊 var vs let vs const**

```javascript
// ============= VAR (ES5) =============
var name = 'John';
var age = 30;
var age = 25; // ✅ สามารถ re-declare ได้

// Function scope
function oldFunction() {
  if (true) {
    var message = 'Hello'; // Function scoped
  }
  console.log(message); // ✅ Accessible - 'Hello'
}

// Hoisting behavior
console.log(hoistedVar); // undefined (not error)
var hoistedVar = 'I am hoisted';

// ============= LET (ES6+) =============
let username = 'Alice';
// let username = 'Bob'; // ❌ SyntaxError: Identifier 'username' has already been declared

// Block scope
function modernFunction() {
  if (true) {
    let blockMessage = 'Hello'; // Block scoped
  }
  // console.log(blockMessage); // ❌ ReferenceError
}

// Temporal Dead Zone
// console.log(notYetDeclared); // ❌ ReferenceError
let notYetDeclared = 'Now I exist';

// ============= CONST (ES6+) =============
const API_URL = 'https://api.example.com';
// API_URL = 'https://other-api.com'; // ❌ TypeError

// Objects and arrays are mutable
const user = { name: 'John', age: 30 };
user.age = 31; // ✅ Allowed - modifying property
user.city = 'Bangkok'; // ✅ Allowed - adding property
// user = {}; // ❌ TypeError - reassigning

const fruits = ['apple', 'banana'];
fruits.push('orange'); // ✅ Allowed - modifying array
fruits[0] = 'grape'; // ✅ Allowed - changing element
// fruits = []; // ❌ TypeError - reassigning
```

### **🏗️ Scope Comparison**

```javascript
// ============= FUNCTION SCOPE (VAR) =============
function functionScopeExample() {
  var x = 1;
  
  if (true) {
    var x = 2; // Same variable
    console.log(x); // 2
  }
  
  console.log(x); // 2 (modified)
  
  for (var i = 0; i < 3; i++) {
    var loopVar = i;
  }
  console.log(i); // 3 (accessible outside loop)
  console.log(loopVar); // 2 (accessible outside loop)
}

// ============= BLOCK SCOPE (LET/CONST) =============
function blockScopeExample() {
  let x = 1;
  const y = 1;
  
  if (true) {
    let x = 2; // Different variable
    const y = 2; // Different variable
    console.log(x, y); // 2, 2
  }
  
  console.log(x, y); // 1, 1 (unchanged)
  
  for (let i = 0; i < 3; i++) {
    const loopConst = i;
    // loopConst is only accessible here
  }
  // console.log(i); // ❌ ReferenceError
  // console.log(loopConst); // ❌ ReferenceError
}

// ============= TEMPORAL DEAD ZONE =============
function temporalDeadZoneExample() {
  console.log(varVariable); // undefined (hoisted)
  // console.log(letVariable); // ❌ ReferenceError (TDZ)
  // console.log(constVariable); // ❌ ReferenceError (TDZ)
  
  var varVariable = 'var value';
  let letVariable = 'let value';
  const constVariable = 'const value';
  
  console.log(varVariable); // 'var value'
  console.log(letVariable); // 'let value'
  console.log(constVariable); // 'const value'
}
```

## 🚀 Next.js Applications

### **⚛️ Component-level Scope**

```jsx
// components/UserProfile.jsx
import { useState, useEffect } from 'react';

const UserProfile = ({ userId }) => {
  // ✅ ใช้ const สำหรับ state setters
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // ✅ ใช้ const สำหรับค่าคงที่
  const API_BASE = process.env.NEXT_PUBLIC_API_URL;
  const CACHE_DURATION = 5 * 60 * 1000; // 5 minutes

  // ✅ Block scope ใน useEffect
  useEffect(() => {
    let isCancelled = false; // ใช้ let เพื่อ flag cancellation
    
    const fetchUserData = async () => {
      try {
        setLoading(true);
        setError(null);

        // ✅ Block scope variables
        const response = await fetch(`${API_BASE}/users/${userId}`);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const userData = await response.json();
        
        // ตรวจสอบว่า component ยังอยู่หรือไม่
        if (!isCancelled) {
          setUser(userData);
        }
      } catch (err) {
        if (!isCancelled) {
          setError(err.message);
        }
      } finally {
        if (!isCancelled) {
          setLoading(false);
        }
      }
    };

    if (userId) {
      fetchUserData();
    }

    // Cleanup function
    return () => {
      isCancelled = true; // Modify let variable
    };
  }, [userId, API_BASE]);

  // ✅ Event handlers ใช้ const
  const handleRefresh = () => {
    // Force refresh โดยการเปลี่ยน dependency
    const timestamp = Date.now();
    // Re-trigger effect somehow
  };

  const handleEdit = (field, value) => {
    setUser(prevUser => {
      // ✅ ใช้ const สำหรับ updated object
      const updatedUser = {
        ...prevUser,
        [field]: value,
        lastModified: new Date().toISOString()
      };
      return updatedUser;
    });
  };

  // ✅ Conditional rendering logic
  if (loading) {
    return (
      <div className="loading">
        <p>Loading user profile...</p>
      </div>
    );
  }

  if (error) {
    return (
      <div className="error">
        <p>Error: {error}</p>
        <button onClick={handleRefresh}>Retry</button>
      </div>
    );
  }

  if (!user) {
    return (
      <div className="not-found">
        <p>User not found</p>
      </div>
    );
  }

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      
      {/* ✅ Map ใช้ block scope */}
      <div className="user-details">
        {Object.entries(user).map(([key, value]) => {
          // ✅ const ใน block scope ของ map
          const isPrivateField = ['password', 'token'].includes(key);
          
          if (isPrivateField) {
            return null;
          }

          return (
            <div key={key} className="field">
              <label>{key}:</label>
              <span>{value}</span>
            </div>
          );
        })}
      </div>
    </div>
  );
};

export default UserProfile;
```

### **🔧 Custom Hooks with Proper Scoping**

```javascript
// hooks/useApi.js
import { useState, useEffect, useCallback, useRef } from 'react';

const useApi = (url, options = {}) => {
  // ✅ State variables ใช้ const
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // ✅ Ref สำหรับ mutable values ที่ไม่ trigger re-render
  const abortControllerRef = useRef(null);
  const retryCountRef = useRef(0);

  // ✅ Constants
  const MAX_RETRIES = options.maxRetries || 3;
  const RETRY_DELAY = options.retryDelay || 1000;

  // ✅ Memoized fetch function
  const fetchData = useCallback(async (customUrl = url) => {
    // ✅ ใช้ let สำหรับ retry logic
    let currentRetry = 0;
    
    const attemptFetch = async () => {
      try {
        // Cancel previous request
        if (abortControllerRef.current) {
          abortControllerRef.current.abort();
        }

        // ✅ สร้าง new AbortController
        const controller = new AbortController();
        abortControllerRef.current = controller;

        setLoading(true);
        setError(null);

        // ✅ Fetch options ใช้ const
        const fetchOptions = {
          signal: controller.signal,
          ...options,
          headers: {
            'Content-Type': 'application/json',
            ...options.headers
          }
        };

        const response = await fetch(customUrl, fetchOptions);

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const result = await response.json();
        setData(result);
        retryCountRef.current = 0; // Reset retry count on success

      } catch (err) {
        if (err.name === 'AbortError') {
          return; // Request was cancelled
        }

        // ✅ Retry logic
        if (currentRetry < MAX_RETRIES) {
          currentRetry++;
          retryCountRef.current = currentRetry;
          
          // ✅ Block scope สำหรับ timeout
          setTimeout(() => {
            attemptFetch();
          }, RETRY_DELAY * currentRetry);
        } else {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    await attemptFetch();
  }, [url, options, MAX_RETRIES, RETRY_DELAY]);

  // ✅ Manual refresh function
  const refresh = useCallback(() => {
    fetchData();
  }, [fetchData]);

  // ✅ Effect สำหรับ initial fetch
  useEffect(() => {
    if (url) {
      fetchData();
    }

    // ✅ Cleanup
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [fetchData]);

  return {
    data,
    loading,
    error,
    refresh,
    retryCount: retryCountRef.current
  };
};

export default useApi;

// การใช้งาน useApi hook
const UserList = () => {
  const { data: users, loading, error, refresh } = useApi('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>Users</h1>
      <button onClick={refresh}>Refresh</button>
      
      {users?.map(user => {
        // ✅ Block scope ใน map
        const isActive = user.status === 'active';
        const userClass = `user ${isActive ? 'active' : 'inactive'}`;
        
        return (
          <div key={user.id} className={userClass}>
            <h3>{user.name}</h3>
            <p>{user.email}</p>
          </div>
        );
      })}
    </div>
  );
};
```

### **🌐 API Routes with Proper Scoping**

```javascript
// pages/api/users/index.js
const users = []; // Module-level storage (for demo)

export default async function handler(req, res) {
  // ✅ ใช้ const สำหรับ method
  const { method } = req;

  try {
    switch (method) {
      case 'GET': {
        // ✅ Block scope สำหรับ GET logic
        const { page = 1, limit = 10, search = '' } = req.query;
        
        // ✅ Parse parameters
        const pageNum = parseInt(page, 10);
        const limitNum = parseInt(limit, 10);
        
        // ✅ Filter และ pagination
        let filteredUsers = users;
        
        if (search) {
          const searchLower = search.toLowerCase();
          filteredUsers = users.filter(user =>
            user.name.toLowerCase().includes(searchLower) ||
            user.email.toLowerCase().includes(searchLower)
          );
        }

        const startIndex = (pageNum - 1) * limitNum;
        const endIndex = startIndex + limitNum;
        const paginatedUsers = filteredUsers.slice(startIndex, endIndex);

        // ✅ Response data
        const responseData = {
          users: paginatedUsers,
          pagination: {
            page: pageNum,
            limit: limitNum,
            total: filteredUsers.length,
            totalPages: Math.ceil(filteredUsers.length / limitNum)
          }
        };

        res.status(200).json(responseData);
        break;
      }

      case 'POST': {
        // ✅ Block scope สำหรับ POST logic
        const { name, email, age } = req.body;

        // ✅ Validation
        const errors = [];
        
        if (!name?.trim()) {
          errors.push('Name is required');
        }
        
        if (!email?.includes('@')) {
          errors.push('Valid email is required');
        }
        
        if (age && (age < 0 || age > 150)) {
          errors.push('Age must be between 0 and 150');
        }

        if (errors.length > 0) {
          return res.status(400).json({ errors });
        }

        // ✅ Create new user
        const newUser = {
          id: Date.now(),
          name: name.trim(),
          email: email.toLowerCase(),
          age: age || null,
          createdAt: new Date().toISOString(),
          status: 'active'
        };

        users.push(newUser);
        res.status(201).json(newUser);
        break;
      }

      default: {
        // ✅ Default case
        res.setHeader('Allow', ['GET', 'POST']);
        res.status(405).json({ error: `Method ${method} not allowed` });
        break;
      }
    }
  } catch (error) {
    // ✅ Global error handling
    console.error('API Error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}

// pages/api/users/[id].js
export default async function handler(req, res) {
  const { method, query: { id } } = req;
  
  // ✅ Parse user ID
  const userId = parseInt(id, 10);
  
  if (isNaN(userId)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }

  try {
    switch (method) {
      case 'GET': {
        // ✅ Find user
        const user = users.find(u => u.id === userId);
        
        if (!user) {
          return res.status(404).json({ error: 'User not found' });
        }

        res.status(200).json(user);
        break;
      }

      case 'PUT':
      case 'PATCH': {
        // ✅ Find user index
        const userIndex = users.findIndex(u => u.id === userId);
        
        if (userIndex === -1) {
          return res.status(404).json({ error: 'User not found' });
        }

        // ✅ Update logic
        const updates = req.body;
        const currentUser = users[userIndex];
        
        // ✅ Merge updates (PATCH) หรือ replace (PUT)
        const updatedUser = method === 'PATCH'
          ? { ...currentUser, ...updates, id: userId, updatedAt: new Date().toISOString() }
          : { ...updates, id: userId, updatedAt: new Date().toISOString() };

        users[userIndex] = updatedUser;
        res.status(200).json(updatedUser);
        break;
      }

      case 'DELETE': {
        // ✅ Find and remove user
        const userIndex = users.findIndex(u => u.id === userId);
        
        if (userIndex === -1) {
          return res.status(404).json({ error: 'User not found' });
        }

        // ✅ Remove user
        const deletedUser = users.splice(userIndex, 1)[0];
        res.status(200).json({ 
          message: 'User deleted successfully',
          deletedUser 
        });
        break;
      }

      default: {
        res.setHeader('Allow', ['GET', 'PUT', 'PATCH', 'DELETE']);
        res.status(405).json({ error: `Method ${method} not allowed` });
        break;
      }
    }
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

## ⚠️ Common Pitfalls และ Best Practices

### **🚫 หลีกเลี่ยงปัญหาเหล่านี้**

```javascript
// ❌ VAR ใน LOOPS - สร้างปัญหา closure
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log('var:', i); // จะแสดง 3, 3, 3
  }, 100);
}

// ✅ LET ใน LOOPS - ทำงานถูกต้อง
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log('let:', i); // จะแสดง 0, 1, 2
  }, 100);
}

// ❌ การใช้ VAR ใน IF STATEMENTS
function badExample() {
  if (true) {
    var message = 'Hello';
  }
  console.log(message); // ทำงานได้แต่ไม่ดี
}

// ✅ การใช้ LET/CONST ใน IF STATEMENTS
function goodExample() {
  let message;
  if (true) {
    message = 'Hello';
  }
  console.log(message); // ชัดเจนกว่า
}

// ❌ การ Mutate CONST objects โดยไม่ระวัง
const config = { apiUrl: 'https://api.example.com' };
// ลืมว่า object properties สามารถเปลี่ยนได้
config.apiUrl = 'https://malicious-site.com'; // อันตราย!

// ✅ ใช้ Object.freeze() หรือ immutable patterns
const config = Object.freeze({
  apiUrl: 'https://api.example.com'
});
// config.apiUrl = 'https://malicious-site.com'; // ไม่ทำงาน

// หรือใช้ immutable update pattern
const updateConfig = (newConfig) => ({
  ...config,
  ...newConfig
});
```

### **✅ Best Practices**

```javascript
// ✅ 1. ใช้ const เป็นค่าเริ่มต้น
const userName = 'John';
const users = [];
const API_ENDPOINTS = {
  users: '/api/users',
  posts: '/api/posts'
};

// ✅ 2. ใช้ let เมื่อต้องการ reassign
let currentPage = 1;
let isLoading = false;

function loadNextPage() {
  if (!isLoading) {
    isLoading = true;
    currentPage += 1;
    // ... loading logic
    isLoading = false;
  }
}

// ✅ 3. ใช้ block scope เพื่อจำกัดขอบเขต
function processData(data) {
  const results = [];
  
  // Block scope สำหรับ validation
  {
    const errors = [];
    
    if (!data) errors.push('Data is required');
    if (!Array.isArray(data)) errors.push('Data must be an array');
    
    if (errors.length > 0) {
      throw new Error(errors.join(', '));
    }
  }
  
  // Block scope สำหรับ processing
  {
    const processedItems = data.map(item => ({
      ...item,
      processed: true,
      timestamp: Date.now()
    }));
    
    results.push(...processedItems);
  }
  
  return results;
}

// ✅ 4. ใช้ destructuring กับ const
function handleUserData(userData) {
  const { name, email, age, ...profile } = userData;
  const { street, city, ...address } = profile.address || {};
  
  // Process ตามต้องการ
  return {
    displayName: name,
    contact: email,
    location: `${city}, ${address.country || 'Unknown'}`
  };
}

// ✅ 5. Module-level constants
const CONFIG = {
  API_BASE: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000',
  TIMEOUT: 5000,
  RETRY_ATTEMPTS: 3
};

export { CONFIG };
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Variable Declarations**
แก้ไขโค้ดต่อไปนี้ให้ใช้ let/const อย่างเหมาะสม:

```javascript
// โค้ดที่ต้องแก้ไข
var userName = 'John';
var userAge = 25;
var isLoggedIn = false;

function updateUser() {
  if (isLoggedIn) {
    var message = 'User updated successfully';
    userAge = userAge + 1;
  }
  console.log(message);
}

for (var i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log('Count:', i);
  }, i * 1000);
}
```

### **🏗️ แบบฝึกหัดที่ 2: Scope Management**
สร้าง function ที่จัดการ scope อย่างถูกต้อง:

```javascript
// สร้าง shopping cart manager
function createShoppingCart() {
  // ใช้ proper scoping สำหรับ:
  // - items array
  // - total calculation
  // - tax rate (constant)
  // - discount logic
  
  // ต้องมี methods:
  // - addItem(item)
  // - removeItem(id)
  // - calculateTotal()
  // - applyDiscount(percentage)
}
```

### **⚛️ แบบฝึกหัดที่ 3: React Component**
สร้าง React component ที่ใช้ scoping อย่างถูกต้อง:

```jsx
// สร้าง UserForm component ที่:
// 1. ใช้ proper variable declarations
// 2. จัดการ form state
// 3. มี validation logic
// 4. Handle form submission
const UserForm = () => {
  // Implementation here
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 1: Introduction to Modern JavaScript](./01-introduction.md)

**บทถัดไป**: [บทที่ 3: Arrow Functions & this Binding](./03-arrow-functions.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
- [MDN: const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
- [MDN: var](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)
- [JavaScript.info: Variables](https://javascript.info/variables)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **ความแตกต่าง** ระหว่าง var, let, และ const
- **Block Scope vs Function Scope**
- **Temporal Dead Zone** และการ hoisting
- **Best Practices** สำหรับ variable declarations
- **การใช้งานใน Next.js** applications

การเลือกใช้ variable declaration ที่เหมาะสมจะช่วยให้โค้ดมีความปลอดภัย อ่านง่าย และ maintain ได้ดีขึ้น! 🚀
