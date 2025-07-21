# บทที่ 4: Destructuring Assignment

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Object และ Array Destructuring
- ทำความเข้าใจ Default Values และ Rename
- เรียนรู้ Nested Destructuring และ Rest Patterns
- ประยุกต์ใช้ใน React Props และ Next.js

## 📦 Object Destructuring

### **🔧 Basic Object Destructuring**

```javascript
// ============= TRADITIONAL WAY =============
const user = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
  city: 'Bangkok'
};

// ES5 way
var userId = user.id;
var userName = user.name;
var userEmail = user.email;

// ============= ES6+ DESTRUCTURING =============
const { id, name, email } = user;
console.log(id);    // 1
console.log(name);  // 'John Doe'
console.log(email); // 'john@example.com'

// ============= RENAMING VARIABLES =============
const { id: userId, name: fullName, email: userEmail } = user;
console.log(userId);    // 1
console.log(fullName);  // 'John Doe'
console.log(userEmail); // 'john@example.com'

// ============= DEFAULT VALUES =============
const { id, name, email, phone = 'N/A', country = 'Thailand' } = user;
console.log(phone);   // 'N/A' (default value)
console.log(country); // 'Thailand' (default value)

// ============= COMBINING RENAME AND DEFAULTS =============
const { 
  id: userId, 
  name: fullName = 'Anonymous',
  phone: phoneNumber = 'Not provided',
  address: userAddress = 'No address'
} = user;

// ============= EXTRACTING SPECIFIC PROPERTIES =============
const { name, email, ...otherInfo } = user;
console.log(name);      // 'John Doe'
console.log(email);     // 'john@example.com'
console.log(otherInfo); // { id: 1, age: 30, city: 'Bangkok' }

// ============= NESTED DESTRUCTURING =============
const userProfile = {
  id: 1,
  name: 'John Doe',
  contact: {
    email: 'john@example.com',
    phone: '123-456-7890',
    address: {
      street: '123 Main St',
      city: 'Bangkok',
      country: 'Thailand'
    }
  },
  preferences: {
    theme: 'dark',
    notifications: true
  }
};

// Extract nested properties
const {
  name,
  contact: {
    email,
    phone,
    address: { city, country }
  },
  preferences: { theme, notifications }
} = userProfile;

console.log(city);    // 'Bangkok'
console.log(country); // 'Thailand'
console.log(theme);   // 'dark'

// ============= DESTRUCTURING WITH COMPUTED PROPERTIES =============
const propertyName = 'email';
const { [propertyName]: userEmailDynamic } = user;
console.log(userEmailDynamic); // 'john@example.com'

// ============= FUNCTION PARAMETERS DESTRUCTURING =============
// Traditional way
function createUserProfile(user) {
  const name = user.name;
  const email = user.email;
  const age = user.age || 18;
  
  return `${name} (${email}) - Age: ${age}`;
}

// Destructured parameters
function createUserProfileModern({ name, email, age = 18 }) {
  return `${name} (${email}) - Age: ${age}`;
}

// With default object
function createUserProfileWithDefaults({ 
  name = 'Anonymous', 
  email = 'no-email@example.com', 
  age = 18 
} = {}) {
  return `${name} (${email}) - Age: ${age}`;
}

// Usage
const profile1 = createUserProfileModern(user);
const profile2 = createUserProfileWithDefaults(); // Works with no arguments
const profile3 = createUserProfileWithDefaults({ name: 'Jane' });
```

### **⚡ Advanced Object Destructuring Patterns**

```javascript
// ============= DESTRUCTURING IN DIFFERENT CONTEXTS =============

// 1. Variable assignment
let name, email;
({ name, email } = user); // Note: parentheses required

// 2. For...of loops
const users = [
  { id: 1, name: 'John', status: 'active' },
  { id: 2, name: 'Jane', status: 'inactive' },
  { id: 3, name: 'Bob', status: 'active' }
];

for (const { id, name, status } of users) {
  console.log(`User ${id}: ${name} is ${status}`);
}

// 3. Array methods
const activeUsers = users
  .filter(({ status }) => status === 'active')
  .map(({ id, name }) => ({ id, displayName: name }));

// 4. Mixed destructuring with arrays
const response = {
  data: ['user1', 'user2', 'user3'],
  meta: { total: 3, page: 1 }
};

const { 
  data: [firstUser, secondUser, ...restUsers], 
  meta: { total } 
} = response;

// ============= CONDITIONAL DESTRUCTURING =============
const config = {
  api: {
    url: 'https://api.example.com',
    timeout: 5000
  },
  cache: {
    enabled: true,
    duration: 3600
  }
};

// Safe destructuring with optional chaining
const { 
  api: { url, timeout } = {}, 
  cache: { enabled, duration } = {} 
} = config || {};

// ============= DESTRUCTURING WITH TYPE CHECKING =============
function processUserData(userData) {
  // Type-safe destructuring
  if (typeof userData !== 'object' || userData === null) {
    throw new Error('Invalid user data');
  }
  
  const {
    id,
    name,
    email,
    age = null,
    preferences: {
      theme = 'light',
      notifications = true
    } = {}
  } = userData;
  
  // Validation
  if (!id || !name || !email) {
    throw new Error('Missing required user fields');
  }
  
  return {
    id,
    name,
    email,
    age,
    theme,
    notifications
  };
}
```

## 📋 Array Destructuring

### **🔧 Basic Array Destructuring**

```javascript
// ============= TRADITIONAL WAY =============
const colors = ['red', 'green', 'blue', 'yellow', 'purple'];

// ES5 way
var firstColor = colors[0];
var secondColor = colors[1];
var thirdColor = colors[2];

// ============= ES6+ DESTRUCTURING =============
const [first, second, third] = colors;
console.log(first);  // 'red'
console.log(second); // 'green'
console.log(third);  // 'blue'

// ============= SKIPPING ELEMENTS =============
const [primary, , secondary] = colors; // Skip the second element
console.log(primary);   // 'red'
console.log(secondary); // 'blue'

// ============= REST PATTERN =============
const [firstColor, secondColor, ...remainingColors] = colors;
console.log(firstColor);       // 'red'
console.log(secondColor);      // 'green'
console.log(remainingColors);  // ['blue', 'yellow', 'purple']

// ============= DEFAULT VALUES =============
const [a, b, c, d, e, f = 'orange'] = colors;
console.log(f); // 'orange' (default value since colors[5] is undefined)

// ============= SWAPPING VALUES =============
let x = 1;
let y = 2;

// Traditional way (requires temporary variable)
// let temp = x;
// x = y;
// y = temp;

// ES6+ way
[x, y] = [y, x];
console.log(x); // 2
console.log(y); // 1

// ============= ARRAY DESTRUCTURING IN FUNCTIONS =============
function getCoordinates() {
  return [10, 20];
}

const [x, y] = getCoordinates();

// Function with array parameter destructuring
function calculateDistance([x1, y1], [x2, y2]) {
  return Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2);
}

const point1 = [0, 0];
const point2 = [3, 4];
const distance = calculateDistance(point1, point2); // 5

// ============= NESTED ARRAY DESTRUCTURING =============
const nestedArray = [
  [1, 2],
  [3, 4],
  [5, 6]
];

const [[a1, a2], [b1, b2], [c1, c2]] = nestedArray;
console.log(a1, a2); // 1, 2
console.log(b1, b2); // 3, 4
console.log(c1, c2); // 5, 6

// ============= MIXED ARRAY AND OBJECT DESTRUCTURING =============
const users = [
  { id: 1, name: 'John', scores: [85, 90, 78] },
  { id: 2, name: 'Jane', scores: [92, 88, 95] },
  { id: 3, name: 'Bob', scores: [76, 82, 89] }
];

// Extract first user's data
const [{ name: firstName, scores: [firstScore, secondScore] }] = users;
console.log(firstName);   // 'John'
console.log(firstScore);  // 85
console.log(secondScore); // 90
```

### **⚡ Advanced Array Destructuring Patterns**

```javascript
// ============= DESTRUCTURING WITH ITERABLES =============

// Strings
const [first, second, third] = 'hello';
console.log(first, second, third); // 'h', 'e', 'l'

// Sets
const uniqueNumbers = new Set([1, 2, 3, 4, 5]);
const [firstNum, secondNum, ...restNums] = uniqueNumbers;
console.log(firstNum);  // 1
console.log(secondNum); // 2
console.log(restNums);  // [3, 4, 5]

// Maps
const userMap = new Map([
  ['name', 'John'],
  ['email', 'john@example.com'],
  ['age', 30]
]);

const [[nameKey, nameValue], [emailKey, emailValue]] = userMap;
console.log(nameKey, nameValue);   // 'name', 'John'
console.log(emailKey, emailValue); // 'email', 'john@example.com'

// ============= DESTRUCTURING WITH GENERATORS =============
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const [fib1, fib2, fib3, fib4, fib5] = fibonacci();
console.log(fib1, fib2, fib3, fib4, fib5); // 0, 1, 1, 2, 3

// ============= DESTRUCTURING WITH REGULAR EXPRESSIONS =============
const url = 'https://example.com:8080/api/users/123';
const urlPattern = /^(https?):\/\/([^:\/\s]+):(\d+)\/(.*)$/;
const [, protocol, hostname, port, path] = url.match(urlPattern) || [];

console.log(protocol); // 'https'
console.log(hostname); // 'example.com'
console.log(port);     // '8080'
console.log(path);     // 'api/users/123'

// ============= PRACTICAL UTILITY FUNCTIONS =============
// Chunk array into smaller arrays
function chunk(array, size) {
  const chunks = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9];
const chunked = chunk(numbers, 3);
// [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// Use destructuring to process chunks
chunked.forEach(([first, second, third]) => {
  console.log(`Chunk: ${first}, ${second}, ${third || 'N/A'}`);
});

// Zip arrays together
function zip(...arrays) {
  const length = Math.min(...arrays.map(arr => arr.length));
  return Array.from({ length }, (_, i) => arrays.map(arr => arr[i]));
}

const names = ['John', 'Jane', 'Bob'];
const ages = [30, 25, 35];
const cities = ['Bangkok', 'Tokyo', 'London'];

const zipped = zip(names, ages, cities);
// [['John', 30, 'Bangkok'], ['Jane', 25, 'Tokyo'], ['Bob', 35, 'London']]

// Use destructuring to process zipped data
zipped.forEach(([name, age, city]) => {
  console.log(`${name} is ${age} years old and lives in ${city}`);
});
```

## ⚛️ React และ Next.js Applications

### **🔧 Component Props Destructuring**

```jsx
// ============= FUNCTION COMPONENT PROPS =============
// Traditional way
function UserCard(props) {
  return (
    <div className="user-card">
      <img src={props.avatar} alt={props.name} />
      <h3>{props.name}</h3>
      <p>{props.email}</p>
      <span className={props.isActive ? 'active' : 'inactive'}>
        {props.isActive ? 'Active' : 'Inactive'}
      </span>
    </div>
  );
}

// Destructured props
function UserCard({ avatar, name, email, isActive }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <span className={isActive ? 'active' : 'inactive'}>
        {isActive ? 'Active' : 'Inactive'}
      </span>
    </div>
  );
}

// With default values
function UserCard({ 
  avatar = '/default-avatar.png', 
  name = 'Anonymous', 
  email = 'No email', 
  isActive = false,
  className = '',
  ...otherProps 
}) {
  return (
    <div 
      className={`user-card ${className} ${isActive ? 'active' : 'inactive'}`}
      {...otherProps}
    >
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <span>{isActive ? 'Active' : 'Inactive'}</span>
    </div>
  );
}

// ============= NESTED PROPS DESTRUCTURING =============
function UserProfile({ 
  user: { 
    id, 
    name, 
    email, 
    profile: { 
      avatar, 
      bio, 
      social: { twitter, github } = {} 
    } = {},
    preferences: { 
      theme = 'light', 
      notifications = true 
    } = {}
  },
  onEdit,
  onDelete
}) {
  return (
    <div className={`user-profile ${theme}`}>
      <div className="header">
        <img src={avatar} alt={name} />
        <div>
          <h2>{name}</h2>
          <p>{email}</p>
          {bio && <p className="bio">{bio}</p>}
        </div>
      </div>
      
      <div className="social">
        {twitter && <a href={`https://twitter.com/${twitter}`}>Twitter</a>}
        {github && <a href={`https://github.com/${github}`}>GitHub</a>}
      </div>
      
      <div className="actions">
        <button onClick={() => onEdit(id)}>Edit</button>
        <button onClick={() => onDelete(id)}>Delete</button>
      </div>
      
      <div className="preferences">
        <p>Theme: {theme}</p>
        <p>Notifications: {notifications ? 'On' : 'Off'}</p>
      </div>
    </div>
  );
}

// ============= HOOKS DESTRUCTURING =============
import { useState, useEffect } from 'react';

function UserList() {
  // State destructuring
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [filters, setFilters] = useState({
    search: '',
    status: 'all',
    sortBy: 'name'
  });

  // Custom hook destructuring
  const { data, loading: dataLoading, error: dataError } = useApi('/api/users');
  
  // useEffect with destructuring
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('/api/users');
        const { data: usersData, pagination, meta } = await response.json();
        
        setUsers(usersData);
        // Use pagination and meta if needed
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  // Event handlers with destructuring
  const handleFilterChange = ({ target: { name, value } }) => {
    setFilters(prev => ({ ...prev, [name]: value }));
  };

  const handleUserAction = ({ currentTarget: { dataset: { action, userId } } }) => {
    switch (action) {
      case 'edit':
        // Handle edit
        break;
      case 'delete':
        // Handle delete
        break;
      default:
        break;
    }
  };

  // Array methods with destructuring
  const processedUsers = users
    .filter(({ name, email, status }) => {
      const { search, status: filterStatus } = filters;
      const matchesSearch = name.toLowerCase().includes(search.toLowerCase()) ||
                          email.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = filterStatus === 'all' || status === filterStatus;
      return matchesSearch && matchesStatus;
    })
    .sort(({ [filters.sortBy]: a }, { [filters.sortBy]: b }) => {
      return a.localeCompare(b);
    });

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="user-list">
      <div className="filters">
        <input
          name="search"
          type="text"
          placeholder="Search users..."
          value={filters.search}
          onChange={handleFilterChange}
        />
        
        <select
          name="status"
          value={filters.status}
          onChange={handleFilterChange}
        >
          <option value="all">All Status</option>
          <option value="active">Active</option>
          <option value="inactive">Inactive</option>
        </select>
        
        <select
          name="sortBy"
          value={filters.sortBy}
          onChange={handleFilterChange}
        >
          <option value="name">Sort by Name</option>
          <option value="email">Sort by Email</option>
          <option value="createdAt">Sort by Date</option>
        </select>
      </div>

      <div className="users">
        {processedUsers.map((user) => (
          <UserCard 
            key={user.id} 
            {...user}
            onClick={handleUserAction}
            data-action="view"
            data-user-id={user.id}
          />
        ))}
      </div>
    </div>
  );
}
```

### **🌐 API Routes และ Server-side**

```javascript
// pages/api/users/[id].js
export default async function handler(req, res) {
  // Destructure request data
  const { 
    method, 
    query: { id }, 
    body,
    headers: { authorization, 'content-type': contentType }
  } = req;

  try {
    switch (method) {
      case 'GET': {
        // Destructure query parameters
        const { 
          include = '', 
          fields = '', 
          expand = false 
        } = req.query;

        const user = await getUserById(id);
        
        if (!user) {
          return res.status(404).json({ error: 'User not found' });
        }

        // Destructure user data for response
        const {
          password,
          internalNotes,
          ...publicUserData
        } = user;

        // Handle field selection
        if (fields) {
          const requestedFields = fields.split(',');
          const filteredData = requestedFields.reduce((acc, field) => {
            if (field in publicUserData) {
              acc[field] = publicUserData[field];
            }
            return acc;
          }, {});
          
          return res.status(200).json(filteredData);
        }

        return res.status(200).json(publicUserData);
      }

      case 'PUT':
      case 'PATCH': {
        // Destructure and validate request body
        const {
          name,
          email,
          age,
          profile: {
            bio,
            avatar,
            social: { twitter, github, linkedin } = {}
          } = {},
          preferences: {
            theme,
            notifications,
            privacy: { showEmail, showProfile } = {}
          } = {},
          ...otherFields
        } = body || {};

        // Validation
        const errors = [];
        
        if (name !== undefined && (!name || name.trim().length < 2)) {
          errors.push('Name must be at least 2 characters');
        }
        
        if (email !== undefined && !email.includes('@')) {
          errors.push('Valid email is required');
        }
        
        if (age !== undefined && (age < 0 || age > 150)) {
          errors.push('Age must be between 0 and 150');
        }

        if (errors.length > 0) {
          return res.status(400).json({ errors });
        }

        // Prepare update data
        const updateData = {
          ...(name !== undefined && { name: name.trim() }),
          ...(email !== undefined && { email: email.toLowerCase() }),
          ...(age !== undefined && { age }),
          ...(bio !== undefined && { 'profile.bio': bio }),
          ...(avatar !== undefined && { 'profile.avatar': avatar }),
          ...(twitter !== undefined && { 'profile.social.twitter': twitter }),
          ...(github !== undefined && { 'profile.social.github': github }),
          ...(linkedin !== undefined && { 'profile.social.linkedin': linkedin }),
          ...(theme !== undefined && { 'preferences.theme': theme }),
          ...(notifications !== undefined && { 'preferences.notifications': notifications }),
          ...(showEmail !== undefined && { 'preferences.privacy.showEmail': showEmail }),
          ...(showProfile !== undefined && { 'preferences.privacy.showProfile': showProfile }),
          updatedAt: new Date().toISOString()
        };

        const updatedUser = await updateUser(id, updateData);
        
        // Remove sensitive data before sending response
        const { password, internalNotes, ...responseData } = updatedUser;
        
        return res.status(200).json(responseData);
      }

      case 'DELETE': {
        const deletedUser = await deleteUser(id);
        
        if (!deletedUser) {
          return res.status(404).json({ error: 'User not found' });
        }

        // Log deletion (destructure for logging)
        const { id: userId, name, email } = deletedUser;
        console.log(`User deleted: ${userId} - ${name} (${email})`);

        return res.status(200).json({ 
          message: 'User deleted successfully',
          id: userId 
        });
      }

      default:
        res.setHeader('Allow', ['GET', 'PUT', 'PATCH', 'DELETE']);
        return res.status(405).json({ error: `Method ${method} not allowed` });
    }
  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({ error: 'Internal server error' });
  }
}

// ============= DATA PROCESSING UTILITIES =============
// utils/dataProcessing.js

export function processApiResponse(response) {
  // Destructure API response structure
  const {
    data,
    meta: {
      pagination: { page, limit, total, hasMore } = {},
      timing: { queryTime, totalTime } = {},
      ...otherMeta
    } = {},
    errors = [],
    warnings = []
  } = response;

  // Process data with destructuring
  const processedData = data?.map(item => {
    const {
      id,
      type,
      attributes: {
        name,
        email,
        createdAt,
        updatedAt,
        ...otherAttributes
      } = {},
      relationships = {},
      links = {}
    } = item;

    return {
      id,
      type,
      name,
      email,
      createdAt: new Date(createdAt),
      updatedAt: new Date(updatedAt),
      ...otherAttributes,
      relationships,
      links
    };
  }) || [];

  return {
    data: processedData,
    pagination: { page, limit, total, hasMore },
    performance: { queryTime, totalTime },
    meta: otherMeta,
    errors,
    warnings,
    hasErrors: errors.length > 0,
    hasWarnings: warnings.length > 0
  };
}

// Transform form data
export function transformFormData(formData) {
  const {
    // Personal information
    firstName,
    lastName,
    email,
    phone,
    dateOfBirth,
    
    // Address information
    address1,
    address2 = '',
    city,
    state,
    zipCode,
    country = 'Thailand',
    
    // Preferences
    newsletter = false,
    smsNotifications = false,
    emailNotifications = true,
    
    // Social media (optional)
    facebookUrl = '',
    twitterHandle = '',
    linkedinUrl = '',
    
    ...otherFields
  } = formData;

  return {
    personal: {
      name: `${firstName} ${lastName}`.trim(),
      email: email.toLowerCase(),
      phone,
      dateOfBirth: dateOfBirth ? new Date(dateOfBirth) : null
    },
    address: {
      street1: address1,
      street2: address2,
      city,
      state,
      zipCode,
      country
    },
    preferences: {
      marketing: {
        newsletter,
        smsNotifications,
        emailNotifications
      }
    },
    social: {
      ...(facebookUrl && { facebook: facebookUrl }),
      ...(twitterHandle && { twitter: twitterHandle }),
      ...(linkedinUrl && { linkedin: linkedinUrl })
    },
    metadata: {
      ...otherFields,
      createdAt: new Date().toISOString()
    }
  };
}
```

### **📊 Data Fetching Patterns**

```javascript
// hooks/useApi.js
import { useState, useEffect, useCallback } from 'react';

export function useApi(url, options = {}) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = useCallback(async () => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));
      
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      
      // Destructure different response formats
      const data = result.data || result.items || result;
      const { meta, pagination, errors, warnings } = result;
      
      setState({
        data,
        loading: false,
        error: null,
        meta,
        pagination,
        errors,
        warnings
      });
      
    } catch (error) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: error.message
      }));
    }
  }, [url, options]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return {
    ...state,
    refetch: fetchData
  };
}

// Hook สำหรับ form handling
export function useForm(initialValues = {}, validationRules = {}) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const handleChange = useCallback((event) => {
    const { name, value, type, checked } = event.target;
    const fieldValue = type === 'checkbox' ? checked : value;
    
    setValues(prev => ({ ...prev, [name]: fieldValue }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => {
        const { [name]: removedError, ...rest } = prev;
        return rest;
      });
    }
  }, [errors]);

  const handleBlur = useCallback((event) => {
    const { name } = event.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    // Validate field on blur
    if (validationRules[name] && touched[name]) {
      const fieldErrors = validationRules[name](values[name], values);
      if (fieldErrors) {
        setErrors(prev => ({ ...prev, [name]: fieldErrors }));
      }
    }
  }, [validationRules, values, touched]);

  const validate = useCallback(() => {
    const newErrors = {};
    
    Object.entries(validationRules).forEach(([field, validator]) => {
      const fieldErrors = validator(values[field], values);
      if (fieldErrors) {
        newErrors[field] = fieldErrors;
      }
    });
    
    setErrors(newErrors);
    setTouched(Object.keys(validationRules).reduce((acc, field) => {
      acc[field] = true;
      return acc;
    }, {}));
    
    return Object.keys(newErrors).length === 0;
  }, [values, validationRules]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validate,
    reset,
    isValid: Object.keys(errors).length === 0,
    isDirty: JSON.stringify(values) !== JSON.stringify(initialValues)
  };
}

// การใช้งาน hooks
const ContactForm = () => {
  const validationRules = {
    name: (value) => !value?.trim() ? 'Name is required' : null,
    email: (value) => !value?.includes('@') ? 'Valid email is required' : null,
    message: (value) => !value?.trim() || value.length < 10 ? 'Message must be at least 10 characters' : null
  };

  const {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validate,
    reset,
    isValid
  } = useForm(
    { name: '', email: '', message: '', newsletter: false },
    validationRules
  );

  const { data: submitResponse, loading: submitting, error: submitError } = useApi();

  const handleSubmit = async (event) => {
    event.preventDefault();
    
    if (!validate()) {
      return;
    }

    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values)
      });

      if (response.ok) {
        alert('Message sent successfully!');
        reset();
      } else {
        const { errors: serverErrors } = await response.json();
        console.error('Server errors:', serverErrors);
      }
    } catch (error) {
      console.error('Submit error:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name:</label>
        <input
          name="name"
          type="text"
          value={values.name}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.name && errors.name && (
          <span className="error">{errors.name}</span>
        )}
      </div>

      <div>
        <label>Email:</label>
        <input
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>

      <div>
        <label>Message:</label>
        <textarea
          name="message"
          value={values.message}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.message && errors.message && (
          <span className="error">{errors.message}</span>
        )}
      </div>

      <div>
        <label>
          <input
            name="newsletter"
            type="checkbox"
            checked={values.newsletter}
            onChange={handleChange}
          />
          Subscribe to newsletter
        </label>
      </div>

      <button type="submit" disabled={!isValid || submitting}>
        {submitting ? 'Sending...' : 'Send Message'}
      </button>
      
      {submitError && <div className="error">{submitError}</div>}
    </form>
  );
};
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Object Destructuring**
ฝึกใช้ object destructuring ในสถานการณ์ต่างๆ:

```javascript
const apiResponse = {
  status: 'success',
  data: {
    users: [
      {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
        profile: {
          avatar: 'avatar1.jpg',
          bio: 'Software developer',
          address: {
            street: '123 Main St',
            city: 'Bangkok',
            country: 'Thailand'
          },
          social: {
            twitter: '@johndoe',
            github: 'johndoe'
          }
        },
        preferences: {
          theme: 'dark',
          notifications: {
            email: true,
            push: false,
            sms: false
          }
        }
      }
    ],
    pagination: {
      page: 1,
      limit: 10,
      total: 1,
      hasMore: false
    }
  },
  meta: {
    timestamp: '2024-01-01T00:00:00Z',
    version: 'v1'
  }
};

// TODO: ใช้ destructuring เพื่อ:
// 1. Extract status และ users array
// 2. Extract first user's name, email, และ bio
// 3. Extract address information
// 4. Extract social media links (with defaults)
// 5. Extract notification preferences
// 6. Extract pagination info
```

### **📋 แบบฝึกหัดที่ 2: Array Destructuring**
ฝึกใช้ array destructuring:

```javascript
const csvData = [
  ['Name', 'Age', 'City', 'Occupation', 'Salary'],
  ['John Doe', '30', 'Bangkok', 'Developer', '50000'],
  ['Jane Smith', '25', 'Tokyo', 'Designer', '45000'],
  ['Bob Johnson', '35', 'London', 'Manager', '60000']
];

const coordinates = [
  [0, 0, 5],    // [x, y, z]
  [1, 2, 3],
  [4, 5, 6]
];

const nestedData = [
  ['user1', ['john@example.com', 'admin', true]],
  ['user2', ['jane@example.com', 'user', false]],
  ['user3', ['bob@example.com', 'moderator', true]]
];

// TODO: ใช้ destructuring เพื่อ:
// 1. Extract headers และ data rows จาก csvData
// 2. Process each data row โดย destructure columns
// 3. Extract coordinates และคำนวณ distance
// 4. Process nested user data
// 5. Swap coordinates
```

### **⚛️ แบบฝึกหัดที่ 3: React Component**
สร้าง React component ที่ใช้ destructuring อย่างเต็มที่:

```jsx
// สร้าง ProductCard component ที่รับ props:
const productData = {
  id: 1,
  name: 'Laptop',
  price: 25000,
  discount: {
    percentage: 10,
    validUntil: '2024-12-31'
  },
  specs: {
    cpu: 'Intel i7',
    ram: '16GB',
    storage: '512GB SSD'
  },
  images: ['image1.jpg', 'image2.jpg', 'image3.jpg'],
  rating: {
    average: 4.5,
    count: 127
  },
  availability: {
    inStock: true,
    quantity: 5,
    warehouse: 'Bangkok'
  }
};

// TODO: สร้าง ProductCard component ที่:
// 1. Destructure props อย่างเหมาะสม
// 2. แสดงข้อมูลสินค้า
// 3. คำนวณราคาหลังหักส่วนลด
// 4. แสดง rating และ availability
// 5. Handle image gallery
const ProductCard = (props) => {
  // Implementation here
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 3: Arrow Functions & this Binding](./03-arrow-functions.md)

**บทถัดไป**: [บทที่ 5: Spread & Rest Operators](./05-spread-rest.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Destructuring Assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [JavaScript.info: Destructuring](https://javascript.info/destructuring-assignment)
- [React: Props](https://react.dev/learn/passing-props-to-a-component)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Object Destructuring** และ Array Destructuring
- **Default Values** และ Variable Renaming
- **Nested Destructuring** และ Rest Patterns  
- **การใช้งานใน React** props และ hooks
- **Best Practices** สำหรับ destructuring ใน Next.js

Destructuring ช่วยให้โค้ดสั้น อ่านง่าย และจัดการข้อมูลได้อย่างมีประสิทธิภาพ! 🚀
