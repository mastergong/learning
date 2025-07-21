# บทที่ 1: Introduction to Modern JavaScript

## 🎯 จุดประสงค์ของบทเรียน
- ทำความเข้าใจความสำคัญของ ES6+ ใน Next.js
- รู้จักกับ Features หลักของ Modern JavaScript
- เตรียมสภาพแวดล้อมสำหรับการเรียนรู้
- เข้าใจความแตกต่างระหว่าง ES5 และ ES6+

## 🌟 ทำไมต้องเรียน ES6+ สำหรับ Next.js?

### **📈 ความสำคัญของ Modern JavaScript**

```javascript
// ❌ ES5 - วิธีเก่า
function createUser(name, email) {
  return {
    name: name,
    email: email,
    getName: function() {
      return this.name;
    }
  };
}

var users = [];
for (var i = 0; i < data.length; i++) {
  users.push(createUser(data[i].name, data[i].email));
}

// ✅ ES6+ - วิธีใหม่
const createUser = (name, email) => ({
  name,
  email,
  getName: () => name
});

const users = data.map(({ name, email }) => createUser(name, email));
```

### **🎯 ประโยชน์ของ ES6+ ใน Next.js**

1. **โค้ดสั้นและอ่านง่าย**: Syntax ที่กระชับและเข้าใจได้ง่าย
2. **Performance ดีขึ้น**: Features ที่ optimize แล้ว
3. **Type Safety**: ใช้ร่วมกับ TypeScript ได้ดี
4. **Modern Patterns**: รองรับ React Hooks และ Server Components
5. **Better Error Handling**: async/await และ Promise

## 📋 ES6+ Features Overview

### **🔧 Core Features ที่จำเป็นสำหรับ Next.js**

```javascript
// 1. Let & Const
const API_URL = 'https://api.example.com';
let userData = null;

// 2. Arrow Functions
const fetchUser = async (id) => {
  const response = await fetch(`${API_URL}/users/${id}`);
  return response.json();
};

// 3. Template Literals
const message = `Welcome ${user.name}! You have ${user.notifications} notifications.`;

// 4. Destructuring
const { name, email, ...profile } = user;
const [first, second, ...rest] = items;

// 5. Spread Operator
const newUser = { ...user, isActive: true };
const allItems = [...oldItems, ...newItems];

// 6. Default Parameters
const createPost = (title, content = '', published = false) => ({
  title,
  content,
  published,
  createdAt: new Date()
});

// 7. Enhanced Object Literals
const userService = {
  users: [],
  
  // Method shorthand
  addUser(user) {
    this.users.push(user);
  },
  
  // Computed property names
  [Symbol.iterator]() {
    return this.users[Symbol.iterator]();
  }
};

// 8. Classes
class UserManager {
  constructor() {
    this.users = [];
  }
  
  addUser(user) {
    this.users.push(user);
  }
  
  static fromArray(usersArray) {
    const manager = new UserManager();
    manager.users = usersArray;
    return manager;
  }
}

// 9. Modules
export { UserManager };
export default fetchUser;

// 10. Promises & Async/Await
const loadUserData = async () => {
  try {
    const user = await fetchUser(1);
    const posts = await fetchUserPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Failed to load user data:', error);
    throw error;
  }
};
```

### **🏗️ Next.js Specific Usage**

```jsx
// pages/users/[id].js - Next.js Page with ES6+
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';

const UserProfile = ({ initialUser }) => {
  const router = useRouter();
  const [user, setUser] = useState(initialUser);
  const [loading, setLoading] = useState(false);

  // Destructuring with default values
  const { 
    name = 'Unknown User', 
    email = 'No email', 
    avatar,
    ...profile 
  } = user || {};

  // Async function with error handling
  const updateUser = async (updates) => {
    setLoading(true);
    try {
      const response = await fetch(`/api/users/${user.id}`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updates),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const updatedUser = await response.json();
      setUser(prev => ({ ...prev, ...updatedUser }));
    } catch (error) {
      console.error('Update failed:', error);
    } finally {
      setLoading(false);
    }
  };

  // Effect with cleanup
  useEffect(() => {
    const controller = new AbortController();

    const fetchUserDetails = async () => {
      try {
        const response = await fetch(`/api/users/${router.query.id}`, {
          signal: controller.signal
        });
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      }
    };

    if (router.query.id) {
      fetchUserDetails();
    }

    return () => controller.abort();
  }, [router.query.id]);

  return (
    <div className="user-profile">
      <h1>{`Profile: ${name}`}</h1>
      <img src={avatar || '/default-avatar.png'} alt={`${name}'s avatar`} />
      
      {/* Conditional rendering with logical AND */}
      {email && <p>Email: {email}</p>}
      
      {/* Template literals in JSX */}
      <div className={`user-status ${user?.isActive ? 'active' : 'inactive'}`}>
        Status: {user?.isActive ? 'Active' : 'Inactive'}
      </div>

      {/* Spread props */}
      <UserForm 
        {...profile}
        onSubmit={updateUser}
        loading={loading}
      />
    </div>
  );
};

// Static generation with async/await
export const getStaticProps = async ({ params }) => {
  try {
    const user = await fetchUser(params.id);
    
    return {
      props: { initialUser: user },
      revalidate: 60 // ISR
    };
  } catch (error) {
    return {
      notFound: true
    };
  }
};

// Dynamic routes
export const getStaticPaths = async () => {
  const users = await fetchAllUsers();
  
  const paths = users.map(({ id }) => ({
    params: { id: id.toString() }
  }));

  return {
    paths,
    fallback: 'blocking'
  };
};

export default UserProfile;
```

## 🛠️ Environment Setup

### **📦 การตั้งค่า Development Environment**

```bash
# 1. ตรวจสอบ Node.js version
node --version  # ต้อง >= 18.0.0
npm --version

# 2. สร้าง Next.js project
npx create-next-app@latest my-es6-app --typescript --tailwind --eslint --app

# 3. เข้าโฟลเดอร์โปรเจกต์
cd my-es6-app

# 4. ติดตั้ง additional packages
npm install @babel/core @babel/preset-env
npm install --save-dev eslint-config-airbnb

# 5. เริ่ม development server
npm run dev
```

### **⚙️ การตั้งค่า Babel (ถ้าจำเป็น)**

```json
// babel.config.js
module.exports = {
  presets: [
    'next/babel',
    [
      '@babel/preset-env',
      {
        targets: {
          browsers: ['> 1%', 'last 2 versions']
        },
        useBuiltIns: 'usage',
        corejs: 3
      }
    ]
  ],
  plugins: [
    ['@babel/plugin-proposal-decorators', { legacy: true }],
    ['@babel/plugin-proposal-class-properties', { loose: true }]
  ]
};
```

### **🔍 ESLint Configuration**

```json
// .eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "airbnb",
    "airbnb/hooks"
  ],
  "rules": {
    "prefer-const": "error",
    "prefer-arrow-callback": "error",
    "arrow-function-prefer": "error",
    "prefer-template": "error",
    "prefer-destructuring": ["error", {
      "array": true,
      "object": true
    }],
    "no-var": "error",
    "object-shorthand": "error"
  },
  "parserOptions": {
    "ecmaVersion": 2023,
    "sourceType": "module"
  }
}
```

## 📊 ES5 vs ES6+ Comparison

### **🔄 การเปรียบเทียบ Syntax**

```javascript
// ============= VARIABLES =============
// ES5
var userName = 'John';
var userAge = 25;

// ES6+
const userName = 'John';
let userAge = 25;

// ============= FUNCTIONS =============
// ES5
function greetUser(name) {
  return 'Hello, ' + name + '!';
}

// ES6+
const greetUser = (name) => `Hello, ${name}!`;

// ============= OBJECTS =============
// ES5
function createUser(name, email) {
  return {
    name: name,
    email: email,
    greet: function() {
      return 'Hi, I am ' + this.name;
    }
  };
}

// ES6+
const createUser = (name, email) => ({
  name,
  email,
  greet() {
    return `Hi, I am ${this.name}`;
  }
});

// ============= ARRAYS =============
// ES5
var numbers = [1, 2, 3, 4, 5];
var doubled = [];
for (var i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}

// ES6+
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);

// ============= ASYNC CODE =============
// ES5 with Callbacks
function fetchUserData(userId, callback) {
  xhr.get('/api/users/' + userId, function(error, user) {
    if (error) {
      callback(error, null);
      return;
    }
    
    xhr.get('/api/users/' + userId + '/posts', function(error, posts) {
      if (error) {
        callback(error, null);
        return;
      }
      
      callback(null, { user: user, posts: posts });
    });
  });
}

// ES6+ with Promises
const fetchUserData = (userId) => {
  return fetch(`/api/users/${userId}`)
    .then(response => response.json())
    .then(user => {
      return fetch(`/api/users/${userId}/posts`)
        .then(response => response.json())
        .then(posts => ({ user, posts }));
    });
};

// ES2017+ with Async/Await
const fetchUserData = async (userId) => {
  try {
    const userResponse = await fetch(`/api/users/${userId}`);
    const user = await userResponse.json();
    
    const postsResponse = await fetch(`/api/users/${userId}/posts`);
    const posts = await postsResponse.json();
    
    return { user, posts };
  } catch (error) {
    console.error('Error fetching user data:', error);
    throw error;
  }
};
```

### **📈 Performance Benefits**

```javascript
// ============= MEMORY EFFICIENCY =============
// ES5 - Creates new objects every time
function createHandler(value) {
  return function(event) {
    console.log('Value:', value, 'Event:', event);
  };
}

// ES6+ - More efficient with closures
const createHandler = (value) => (event) => {
  console.log(`Value: ${value}, Event:`, event);
};

// ============= ITERATION PERFORMANCE =============
// ES5 - Traditional loop
var sum = 0;
for (var i = 0; i < items.length; i++) {
  sum += items[i].value;
}

// ES6+ - Optimized methods
const sum = items.reduce((acc, item) => acc + item.value, 0);

// ============= STRING CONCATENATION =============
// ES5 - Slow with many concatenations
var html = '<div class="' + className + '">';
html += '<h1>' + title + '</h1>';
html += '<p>' + content + '</p>';
html += '</div>';

// ES6+ - Faster template literals
const html = `
  <div class="${className}">
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;
```

## 🎯 Next.js Integration Examples

### **🔄 Component State Management**

```jsx
// components/UserCounter.jsx
import { useState, useCallback, useMemo } from 'react';

const UserCounter = ({ initialUsers = [] }) => {
  const [users, setUsers] = useState(initialUsers);
  const [filter, setFilter] = useState('');

  // Memoized filtered users
  const filteredUsers = useMemo(() => {
    return users.filter(user => 
      user.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [users, filter]);

  // Callback with dependencies
  const addUser = useCallback((userData) => {
    setUsers(prev => [...prev, { 
      id: Date.now(), 
      ...userData,
      createdAt: new Date().toISOString()
    }]);
  }, []);

  // Destructuring in event handler
  const updateUser = useCallback((id, updates) => {
    setUsers(prev => prev.map(user => 
      user.id === id 
        ? { ...user, ...updates, updatedAt: new Date().toISOString() }
        : user
    ));
  }, []);

  const removeUser = useCallback((id) => {
    setUsers(prev => prev.filter(user => user.id !== id));
  }, []);

  return (
    <div className="user-counter">
      <h2>Users ({filteredUsers.length})</h2>
      
      <input
        type="text"
        placeholder="Filter users..."
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        className="filter-input"
      />

      <div className="users-list">
        {filteredUsers.map(({ id, name, email, isActive }) => (
          <div key={id} className={`user-card ${isActive ? 'active' : 'inactive'}`}>
            <h3>{name}</h3>
            <p>{email}</p>
            <div className="user-actions">
              <button 
                onClick={() => updateUser(id, { isActive: !isActive })}
                className={`toggle-btn ${isActive ? 'deactivate' : 'activate'}`}
              >
                {isActive ? 'Deactivate' : 'Activate'}
              </button>
              <button 
                onClick={() => removeUser(id)}
                className="remove-btn"
              >
                Remove
              </button>
            </div>
          </div>
        ))}
      </div>

      <UserForm onSubmit={addUser} />
    </div>
  );
};

// Custom Hook with ES6+
const useUserForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    isActive: true
  });

  const updateField = useCallback((field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
  }, []);

  const resetForm = useCallback(() => {
    setFormData({ name: '', email: '', isActive: true });
  }, []);

  const isValid = useMemo(() => {
    const { name, email } = formData;
    return name.trim().length > 0 && email.includes('@');
  }, [formData]);

  return {
    formData,
    updateField,
    resetForm,
    isValid
  };
};

// Form Component using the custom hook
const UserForm = ({ onSubmit }) => {
  const { formData, updateField, resetForm, isValid } = useUserForm();

  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    if (isValid) {
      onSubmit(formData);
      resetForm();
    }
  }, [formData, isValid, onSubmit, resetForm]);

  return (
    <form onSubmit={handleSubmit} className="user-form">
      <input
        type="text"
        placeholder="Name"
        value={formData.name}
        onChange={(e) => updateField('name', e.target.value)}
      />
      <input
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={(e) => updateField('email', e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={formData.isActive}
          onChange={(e) => updateField('isActive', e.target.checked)}
        />
        Active user
      </label>
      <button 
        type="submit" 
        disabled={!isValid}
        className={`submit-btn ${isValid ? 'enabled' : 'disabled'}`}
      >
        Add User
      </button>
    </form>
  );
};

export default UserCounter;
```

### **🌐 API Routes with ES6+**

```javascript
// pages/api/users/[id].js
import { NextResponse } from 'next/server';

// Mock database
const users = new Map([
  [1, { id: 1, name: 'John Doe', email: 'john@example.com', isActive: true }],
  [2, { id: 2, name: 'Jane Smith', email: 'jane@example.com', isActive: false }]
]);

export async function GET(request, { params }) {
  try {
    const { id } = params;
    const userId = parseInt(id, 10);
    
    if (!users.has(userId)) {
      return NextResponse.json(
        { error: 'User not found' }, 
        { status: 404 }
      );
    }

    const user = users.get(userId);
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function PATCH(request, { params }) {
  try {
    const { id } = params;
    const userId = parseInt(id, 10);
    const updates = await request.json();

    if (!users.has(userId)) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    const currentUser = users.get(userId);
    const updatedUser = {
      ...currentUser,
      ...updates,
      id: userId, // Prevent ID modification
      updatedAt: new Date().toISOString()
    };

    users.set(userId, updatedUser);

    return NextResponse.json(updatedUser);
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid request body' },
      { status: 400 }
    );
  }
}

export async function DELETE(request, { params }) {
  try {
    const { id } = params;
    const userId = parseInt(id, 10);

    if (!users.has(userId)) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    users.delete(userId);

    return NextResponse.json(
      { message: 'User deleted successfully' },
      { status: 200 }
    );
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// Utility functions with ES6+
export const validateUserData = (data) => {
  const { name, email, isActive = true } = data;
  
  const errors = [];
  
  if (!name?.trim()) {
    errors.push('Name is required');
  }
  
  if (!email?.includes('@')) {
    errors.push('Valid email is required');
  }
  
  return {
    isValid: errors.length === 0,
    errors,
    sanitizedData: {
      name: name?.trim(),
      email: email?.toLowerCase(),
      isActive: Boolean(isActive)
    }
  };
};
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Basic ES6+ Conversion**
แปลงโค้ด ES5 ต่อไปนี้เป็น ES6+:

```javascript
// ES5 Code to Convert
function UserManager() {
  this.users = [];
}

UserManager.prototype.addUser = function(name, email) {
  var user = {
    id: Date.now(),
    name: name,
    email: email,
    createdAt: new Date()
  };
  this.users.push(user);
  return user;
};

UserManager.prototype.findUser = function(id) {
  for (var i = 0; i < this.users.length; i++) {
    if (this.users[i].id === id) {
      return this.users[i];
    }
  }
  return null;
};

var manager = new UserManager();
```

### **🎯 แบบฝึกหัดที่ 2: Template Literals Practice**
สร้างฟังก์ชันที่ใช้ template literals สำหรับ:
1. สร้าง HTML template
2. URL ที่มี query parameters
3. Log message ที่มี timestamp

### **🏗️ แบบฝึกหัดที่ 3: Next.js Component**
สร้าง React component ที่ใช้ ES6+ features:
- Destructuring props
- Arrow functions
- Spread operator
- Default parameters
- Template literals

## 🔗 การเชื่อมต่อ

**บทถัดไป**: [บทที่ 2: Variables & Scope](./02-variables-scope.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [ECMAScript 2015+ Features](https://github.com/lukehoban/es6features)
- [Next.js Documentation](https://nextjs.org/docs)
- [Modern JavaScript Tutorial](https://javascript.info/)
- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **ความสำคัญ** ของ ES6+ ใน Next.js development
- **Features หลัก** ที่ใช้บ่อยในการพัฒนา
- **การตั้งค่า environment** สำหรับ modern JavaScript
- **ตัวอย่างการใช้งาน** ใน Next.js applications

ในบทถัดไป เราจะเรียนรู้เรื่อง Variables และ Scope อย่างละเอียด! 🚀
