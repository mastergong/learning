# บทที่ 7: Modern Array Methods

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Modern Array Methods ใน ES6+
- ทำความเข้าใจ Functional Programming กับ Arrays
- เรียนรู้ Array Processing และ Data Transformation
- ประยุกต์ใช้ใน Next.js สำหรับ Data Handling

## 📚 Modern Array Methods Overview

### **🔍 Array Search Methods**

```javascript
// ============= FIND METHODS =============
const users = [
  { id: 1, name: 'John', age: 30, role: 'admin', active: true },
  { id: 2, name: 'Jane', age: 25, role: 'user', active: true },
  { id: 3, name: 'Bob', age: 35, role: 'user', active: false },
  { id: 4, name: 'Alice', age: 28, role: 'moderator', active: true }
];

// find() - หาค่าแรกที่ตรงเงื่อนไข
const admin = users.find(user => user.role === 'admin');
console.log(admin); // { id: 1, name: 'John', age: 30, role: 'admin', active: true }

const youngUser = users.find(user => user.age < 26);
console.log(youngUser); // { id: 2, name: 'Jane', age: 25, role: 'user', active: true }

// findIndex() - หา index ของค่าแรกที่ตรงเงื่อนไข
const adminIndex = users.findIndex(user => user.role === 'admin');
console.log(adminIndex); // 0

const inactiveIndex = users.findIndex(user => !user.active);
console.log(inactiveIndex); // 2

// includes() - เช็คว่ามีค่านั้นใน array หรือไม่
const fruits = ['apple', 'banana', 'orange', 'grape'];
console.log(fruits.includes('apple')); // true
console.log(fruits.includes('mango')); // false

// เช็คว่ามีผู้ใช้ active หรือไม่
const activeUserIds = users.filter(user => user.active).map(user => user.id);
console.log(activeUserIds.includes(1)); // true

// some() - เช็คว่ามี element อย่างน้อย 1 ตัวที่ตรงเงื่อนไข
const hasAdmin = users.some(user => user.role === 'admin');
console.log(hasAdmin); // true

const hasTeenager = users.some(user => user.age < 18);
console.log(hasTeenager); // false

// every() - เช็คว่า element ทั้งหมดตรงเงื่อนไข
const allActive = users.every(user => user.active);
console.log(allActive); // false

const allAdults = users.every(user => user.age >= 18);
console.log(allAdults); // true

// ============= ADVANCED SEARCHING =============
const products = [
  { id: 1, name: 'Laptop', category: 'electronics', price: 1000, tags: ['computer', 'work'] },
  { id: 2, name: 'Phone', category: 'electronics', price: 500, tags: ['mobile', 'communication'] },
  { id: 3, name: 'Book', category: 'education', price: 20, tags: ['learning', 'knowledge'] },
  { id: 4, name: 'Chair', category: 'furniture', price: 150, tags: ['office', 'comfort'] }
];

// Advanced search function
const searchProducts = (products, criteria) => {
  return products.filter(product => {
    // Search in name
    if (criteria.name && !product.name.toLowerCase().includes(criteria.name.toLowerCase())) {
      return false;
    }
    
    // Filter by category
    if (criteria.category && product.category !== criteria.category) {
      return false;
    }
    
    // Filter by price range
    if (criteria.minPrice && product.price < criteria.minPrice) {
      return false;
    }
    if (criteria.maxPrice && product.price > criteria.maxPrice) {
      return false;
    }
    
    // Search in tags
    if (criteria.tags && criteria.tags.length > 0) {
      const hasMatchingTag = criteria.tags.some(tag => 
        product.tags.some(productTag => 
          productTag.toLowerCase().includes(tag.toLowerCase())
        )
      );
      if (!hasMatchingTag) {
        return false;
      }
    }
    
    return true;
  });
};

// Usage examples
const laptopResults = searchProducts(products, { name: 'laptop' });
const electronicsUnder600 = searchProducts(products, { 
  category: 'electronics', 
  maxPrice: 600 
});
const workRelated = searchProducts(products, { tags: ['work', 'office'] });

console.log('Laptop results:', laptopResults);
console.log('Electronics under $600:', electronicsUnder600);
console.log('Work-related products:', workRelated);
```

### **🔄 Array Transformation Methods**

```javascript
// ============= MAP - TRANSFORM ELEMENTS =============
const numbers = [1, 2, 3, 4, 5];

// Basic transformation
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

const squared = numbers.map(num => num ** 2);
console.log(squared); // [1, 4, 9, 16, 25]

// Object transformation
const users = [
  { firstName: 'John', lastName: 'Doe', age: 30 },
  { firstName: 'Jane', lastName: 'Smith', age: 25 },
  { firstName: 'Bob', lastName: 'Johnson', age: 35 }
];

// Create full names
const fullNames = users.map(user => `${user.firstName} ${user.lastName}`);
console.log(fullNames); // ['John Doe', 'Jane Smith', 'Bob Johnson']

// Transform to different object structure
const userProfiles = users.map(user => ({
  id: Math.random().toString(36).substr(2, 9),
  displayName: `${user.firstName} ${user.lastName}`,
  age: user.age,
  isAdult: user.age >= 18,
  initials: `${user.firstName[0]}${user.lastName[0]}`
}));

console.log(userProfiles);

// ============= CHAINING TRANSFORMATIONS =============
const salesData = [
  { product: 'Laptop', quantity: 2, price: 1000 },
  { product: 'Phone', quantity: 5, price: 500 },
  { product: 'Tablet', quantity: 3, price: 300 }
];

// Calculate totals and add metadata
const processedSales = salesData
  .map(sale => ({
    ...sale,
    total: sale.quantity * sale.price,
    category: sale.price > 500 ? 'premium' : 'standard'
  }))
  .map(sale => ({
    ...sale,
    formattedTotal: `$${sale.total.toLocaleString()}`,
    percentageOfMaxPrice: ((sale.price / 1000) * 100).toFixed(1) + '%'
  }));

console.log(processedSales);

// ============= ADVANCED MAP PATTERNS =============
// Map with index
const indexedItems = ['apple', 'banana', 'orange'].map((item, index) => ({
  id: index + 1,
  name: item,
  position: index + 1,
  isFirst: index === 0,
  isLast: index === 2
}));

// Conditional mapping
const conditionalMap = numbers.map(num => {
  if (num % 2 === 0) {
    return { value: num, type: 'even', doubled: num * 2 };
  } else {
    return { value: num, type: 'odd', tripled: num * 3 };
  }
});

// Async mapping (be careful with this pattern)
const fetchUserDetails = async (userIds) => {
  // DON'T do this - creates promises array
  // const userDetails = userIds.map(async id => await fetchUser(id));
  
  // DO this instead
  const userPromises = userIds.map(id => fetchUser(id));
  const userDetails = await Promise.all(userPromises);
  
  return userDetails;
};
```

### **🔽 Array Filtering Methods**

```javascript
// ============= FILTER - SELECT ELEMENTS =============
const products = [
  { id: 1, name: 'Laptop', category: 'electronics', price: 1000, inStock: true, rating: 4.5 },
  { id: 2, name: 'Phone', category: 'electronics', price: 500, inStock: false, rating: 4.2 },
  { id: 3, name: 'Book', category: 'education', price: 20, inStock: true, rating: 4.8 },
  { id: 4, name: 'Chair', category: 'furniture', price: 150, inStock: true, rating: 3.9 },
  { id: 5, name: 'Tablet', category: 'electronics', price: 300, inStock: true, rating: 4.0 }
];

// Basic filtering
const inStockProducts = products.filter(product => product.inStock);
const expensiveProducts = products.filter(product => product.price > 200);
const highRatedProducts = products.filter(product => product.rating >= 4.5);

// Multiple conditions
const premiumElectronics = products.filter(product => 
  product.category === 'electronics' && 
  product.price > 400 && 
  product.inStock
);

// Complex filtering with helper functions
const isElectronics = product => product.category === 'electronics';
const isInStock = product => product.inStock;
const isPriceInRange = (min, max) => product => product.price >= min && product.price <= max;
const hasHighRating = threshold => product => product.rating >= threshold;

// Compose filters
const premiumInStockElectronics = products
  .filter(isElectronics)
  .filter(isInStock)
  .filter(isPriceInRange(300, 1000))
  .filter(hasHighRating(4.0));

// ============= ADVANCED FILTERING PATTERNS =============
// Dynamic filtering
const createFilter = (criteria) => {
  return (product) => {
    for (const [key, value] of Object.entries(criteria)) {
      if (key === 'priceRange') {
        if (product.price < value.min || product.price > value.max) {
          return false;
        }
      } else if (key === 'categories') {
        if (!value.includes(product.category)) {
          return false;
        }
      } else if (key === 'search') {
        if (!product.name.toLowerCase().includes(value.toLowerCase())) {
          return false;
        }
      } else if (product[key] !== value) {
        return false;
      }
    }
    return true;
  };
};

// Usage
const filterCriteria = {
  inStock: true,
  priceRange: { min: 100, max: 800 },
  categories: ['electronics', 'furniture']
};

const filteredProducts = products.filter(createFilter(filterCriteria));

// ============= FILTERING WITH STATE MANAGEMENT =============
class ProductFilter {
  constructor(products) {
    this.originalProducts = products;
    this.filteredProducts = [...products];
    this.activeFilters = {};
  }

  addFilter(key, condition) {
    this.activeFilters[key] = condition;
    this.applyFilters();
    return this;
  }

  removeFilter(key) {
    delete this.activeFilters[key];
    this.applyFilters();
    return this;
  }

  applyFilters() {
    this.filteredProducts = this.originalProducts.filter(product => {
      return Object.values(this.activeFilters).every(condition => 
        condition(product)
      );
    });
  }

  getResults() {
    return this.filteredProducts;
  }

  getStats() {
    return {
      total: this.originalProducts.length,
      filtered: this.filteredProducts.length,
      activeFilters: Object.keys(this.activeFilters).length
    };
  }
}

// Usage
const productFilter = new ProductFilter(products);

productFilter
  .addFilter('category', product => product.category === 'electronics')
  .addFilter('price', product => product.price < 600)
  .addFilter('rating', product => product.rating >= 4.0);

console.log(productFilter.getResults());
console.log(productFilter.getStats());
```

### **📊 Array Aggregation Methods**

```javascript
// ============= REDUCE - POWERFUL AGGREGATION =============
const orders = [
  { id: 1, customerId: 101, amount: 250, status: 'completed' },
  { id: 2, customerId: 102, amount: 180, status: 'completed' },
  { id: 3, customerId: 101, amount: 320, status: 'pending' },
  { id: 4, customerId: 103, amount: 450, status: 'completed' },
  { id: 5, customerId: 102, amount: 200, status: 'cancelled' }
];

// Simple sum
const totalRevenue = orders
  .filter(order => order.status === 'completed')
  .reduce((sum, order) => sum + order.amount, 0);

console.log('Total Revenue:', totalRevenue); // 880

// Group by customer
const ordersByCustomer = orders.reduce((acc, order) => {
  if (!acc[order.customerId]) {
    acc[order.customerId] = [];
  }
  acc[order.customerId].push(order);
  return acc;
}, {});

console.log('Orders by customer:', ordersByCustomer);

// Calculate statistics
const orderStats = orders.reduce((stats, order) => {
  // Count by status
  stats.byStatus[order.status] = (stats.byStatus[order.status] || 0) + 1;
  
  // Sum amounts by status
  if (!stats.amountByStatus[order.status]) {
    stats.amountByStatus[order.status] = 0;
  }
  stats.amountByStatus[order.status] += order.amount;
  
  // Track min/max amounts
  if (order.amount > stats.maxAmount) {
    stats.maxAmount = order.amount;
    stats.maxAmountOrder = order;
  }
  if (order.amount < stats.minAmount) {
    stats.minAmount = order.amount;
    stats.minAmountOrder = order;
  }
  
  // Calculate total
  stats.totalAmount += order.amount;
  stats.totalOrders += 1;
  
  return stats;
}, {
  byStatus: {},
  amountByStatus: {},
  maxAmount: 0,
  minAmount: Infinity,
  maxAmountOrder: null,
  minAmountOrder: null,
  totalAmount: 0,
  totalOrders: 0
});

// Add average
orderStats.averageAmount = orderStats.totalAmount / orderStats.totalOrders;

console.log('Order Statistics:', orderStats);

// ============= ADVANCED REDUCE PATTERNS =============
// Create lookup tables
const createLookupTable = (array, keyField, valueField) => {
  return array.reduce((lookup, item) => {
    lookup[item[keyField]] = valueField ? item[valueField] : item;
    return lookup;
  }, {});
};

const customers = [
  { id: 101, name: 'John Doe', email: 'john@example.com' },
  { id: 102, name: 'Jane Smith', email: 'jane@example.com' },
  { id: 103, name: 'Bob Johnson', email: 'bob@example.com' }
];

const customerLookup = createLookupTable(customers, 'id');
const customerNameLookup = createLookupTable(customers, 'id', 'name');

// Flatten nested arrays
const categories = [
  { name: 'Electronics', products: ['phone', 'laptop', 'tablet'] },
  { name: 'Books', products: ['fiction', 'non-fiction'] },
  { name: 'Clothing', products: ['shirts', 'pants', 'shoes'] }
];

const allProducts = categories.reduce((acc, category) => {
  return acc.concat(category.products.map(product => ({
    name: product,
    category: category.name
  })));
}, []);

// Alternative using flatMap (ES2019)
const allProductsFlatMap = categories.flatMap(category =>
  category.products.map(product => ({
    name: product,
    category: category.name
  }))
);

// ============= FUNCTIONAL COMPOSITION WITH REDUCE =============
const pipe = (...functions) => (value) => functions.reduce((acc, fn) => fn(acc), value);

const compose = (...functions) => (value) => functions.reduceRight((acc, fn) => fn(acc), value);

// Data processing pipeline
const processOrderData = pipe(
  orders => orders.filter(order => order.status === 'completed'),
  orders => orders.map(order => ({ ...order, tax: order.amount * 0.1 })),
  orders => orders.reduce((summary, order) => ({
    totalOrders: summary.totalOrders + 1,
    totalAmount: summary.totalAmount + order.amount,
    totalTax: summary.totalTax + order.tax
  }), { totalOrders: 0, totalAmount: 0, totalTax: 0 })
);

const summary = processOrderData(orders);
console.log('Processed Summary:', summary);
```

## 🔧 Array Utility Functions

### **🛠️ Custom Array Methods**

```javascript
// ============= ARRAY UTILITY CLASS =============
class ArrayUtils {
  // Chunk array into smaller arrays
  static chunk(array, size) {
    const chunks = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }

  // Remove duplicates
  static unique(array, keyField = null) {
    if (keyField) {
      const seen = new Set();
      return array.filter(item => {
        const key = item[keyField];
        if (seen.has(key)) {
          return false;
        }
        seen.add(key);
        return true;
      });
    }
    return [...new Set(array)];
  }

  // Intersection of arrays
  static intersection(...arrays) {
    if (arrays.length === 0) return [];
    return arrays.reduce((acc, array) => 
      acc.filter(item => array.includes(item))
    );
  }

  // Difference between arrays
  static difference(array1, array2) {
    return array1.filter(item => !array2.includes(item));
  }

  // Group by field
  static groupBy(array, keyField) {
    return array.reduce((groups, item) => {
      const key = typeof keyField === 'function' ? keyField(item) : item[keyField];
      if (!groups[key]) {
        groups[key] = [];
      }
      groups[key].push(item);
      return groups;
    }, {});
  }

  // Sort by multiple fields
  static sortBy(array, ...sortFields) {
    return [...array].sort((a, b) => {
      for (const field of sortFields) {
        let aVal, bVal, direction = 1;
        
        if (typeof field === 'string') {
          aVal = a[field];
          bVal = b[field];
        } else if (typeof field === 'object') {
          aVal = a[field.key];
          bVal = b[field.key];
          direction = field.direction === 'desc' ? -1 : 1;
        } else if (typeof field === 'function') {
          aVal = field(a);
          bVal = field(b);
        }
        
        if (aVal < bVal) return -1 * direction;
        if (aVal > bVal) return 1 * direction;
      }
      return 0;
    });
  }

  // Paginate array
  static paginate(array, page, pageSize) {
    const startIndex = (page - 1) * pageSize;
    const endIndex = startIndex + pageSize;
    
    return {
      data: array.slice(startIndex, endIndex),
      pagination: {
        currentPage: page,
        pageSize: pageSize,
        totalItems: array.length,
        totalPages: Math.ceil(array.length / pageSize),
        hasNext: endIndex < array.length,
        hasPrev: startIndex > 0
      }
    };
  }

  // Sample random items
  static sample(array, count = 1) {
    const shuffled = [...array].sort(() => 0.5 - Math.random());
    return count === 1 ? shuffled[0] : shuffled.slice(0, count);
  }

  // Deep flatten
  static flattenDeep(array) {
    return array.reduce((acc, val) => 
      Array.isArray(val) ? acc.concat(this.flattenDeep(val)) : acc.concat(val), []
    );
  }

  // Count occurrences
  static countBy(array, keyField) {
    return array.reduce((counts, item) => {
      const key = typeof keyField === 'function' ? keyField(item) : item[keyField];
      counts[key] = (counts[key] || 0) + 1;
      return counts;
    }, {});
  }
}

// ============= USAGE EXAMPLES =============
const testData = [
  { id: 1, name: 'John', department: 'IT', salary: 50000 },
  { id: 2, name: 'Jane', department: 'HR', salary: 45000 },
  { id: 3, name: 'Bob', department: 'IT', salary: 55000 },
  { id: 4, name: 'Alice', department: 'Finance', salary: 60000 },
  { id: 5, name: 'Charlie', department: 'IT', salary: 52000 }
];

// Chunk into groups of 2
const chunked = ArrayUtils.chunk(testData, 2);
console.log('Chunked:', chunked);

// Group by department
const grouped = ArrayUtils.groupBy(testData, 'department');
console.log('Grouped by department:', grouped);

// Sort by salary descending, then by name ascending
const sorted = ArrayUtils.sortBy(
  testData,
  { key: 'salary', direction: 'desc' },
  'name'
);
console.log('Sorted:', sorted);

// Paginate
const paginated = ArrayUtils.paginate(testData, 2, 2);
console.log('Page 2:', paginated);

// Count by department
const departmentCounts = ArrayUtils.countBy(testData, 'department');
console.log('Department counts:', departmentCounts);

// Sample 2 random employees
const randomEmployees = ArrayUtils.sample(testData, 2);
console.log('Random sample:', randomEmployees);
```

### **🔄 Async Array Processing**

```javascript
// ============= ASYNC ARRAY OPERATIONS =============
class AsyncArrayUtils {
  // Process items sequentially
  static async mapSequential(array, asyncFunction) {
    const results = [];
    for (const item of array) {
      results.push(await asyncFunction(item));
    }
    return results;
  }

  // Process items in parallel
  static async mapParallel(array, asyncFunction) {
    const promises = array.map(asyncFunction);
    return Promise.all(promises);
  }

  // Process items in parallel with concurrency limit
  static async mapConcurrent(array, asyncFunction, concurrency = 3) {
    const results = [];
    const chunks = ArrayUtils.chunk(array, concurrency);
    
    for (const chunk of chunks) {
      const chunkResults = await Promise.all(chunk.map(asyncFunction));
      results.push(...chunkResults);
    }
    
    return results;
  }

  // Filter with async predicate
  static async filter(array, asyncPredicate) {
    const results = await Promise.all(
      array.map(async (item, index) => ({
        item,
        index,
        keep: await asyncPredicate(item, index)
      }))
    );
    
    return results
      .filter(result => result.keep)
      .map(result => result.item);
  }

  // Find with async predicate
  static async find(array, asyncPredicate) {
    for (const item of array) {
      if (await asyncPredicate(item)) {
        return item;
      }
    }
    return undefined;
  }

  // Reduce with async accumulator
  static async reduce(array, asyncReducer, initialValue) {
    let accumulator = initialValue;
    for (const item of array) {
      accumulator = await asyncReducer(accumulator, item);
    }
    return accumulator;
  }
}

// ============= PRACTICAL ASYNC EXAMPLES =============
// Simulate async operations
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

const fetchUserData = async (userId) => {
  await delay(Math.random() * 1000); // Simulate network delay
  return {
    id: userId,
    name: `User ${userId}`,
    posts: Math.floor(Math.random() * 10),
    followers: Math.floor(Math.random() * 1000)
  };
};

const validateUser = async (user) => {
  await delay(100);
  return user.posts > 5 && user.followers > 100;
};

const processUserBatch = async (userIds) => {
  console.time('Processing users');
  
  // Fetch user data in parallel (fast)
  const users = await AsyncArrayUtils.mapParallel(userIds, fetchUserData);
  console.log('Fetched users:', users.length);
  
  // Filter active users (parallel)
  const activeUsers = await AsyncArrayUtils.filter(users, validateUser);
  console.log('Active users:', activeUsers.length);
  
  // Calculate total engagement sequentially
  const totalEngagement = await AsyncArrayUtils.reduce(
    activeUsers,
    async (total, user) => {
      await delay(50); // Simulate calculation
      return total + user.posts + user.followers;
    },
    0
  );
  
  console.timeEnd('Processing users');
  
  return {
    totalUsers: users.length,
    activeUsers: activeUsers.length,
    totalEngagement
  };
};

// Usage
// const userIds = Array.from({ length: 10 }, (_, i) => i + 1);
// processUserBatch(userIds).then(console.log);
```

## 🚀 Next.js Applications

### **📊 Data Processing Components**

```jsx
// components/DataTable.jsx
import { useState, useMemo } from 'react';

const DataTable = ({ data, columns, initialPageSize = 10 }) => {
  const [sortConfig, setSortConfig] = useState({ key: null, direction: 'asc' });
  const [filterConfig, setFilterConfig] = useState({});
  const [currentPage, setCurrentPage] = useState(1);
  const [pageSize, setPageSize] = useState(initialPageSize);
  const [searchTerm, setSearchTerm] = useState('');

  // Process data with all filters and sorting
  const processedData = useMemo(() => {
    let processed = [...data];

    // Apply search filter
    if (searchTerm) {
      processed = processed.filter(item =>
        columns.some(column => {
          const value = item[column.key];
          return value && value.toString().toLowerCase().includes(searchTerm.toLowerCase());
        })
      );
    }

    // Apply column filters
    Object.entries(filterConfig).forEach(([key, value]) => {
      if (value !== '' && value !== null && value !== undefined) {
        processed = processed.filter(item => {
          const itemValue = item[key];
          if (typeof value === 'string') {
            return itemValue.toString().toLowerCase().includes(value.toLowerCase());
          }
          return itemValue === value;
        });
      }
    });

    // Apply sorting
    if (sortConfig.key) {
      processed.sort((a, b) => {
        const aVal = a[sortConfig.key];
        const bVal = b[sortConfig.key];
        
        if (aVal < bVal) return sortConfig.direction === 'asc' ? -1 : 1;
        if (aVal > bVal) return sortConfig.direction === 'asc' ? 1 : -1;
        return 0;
      });
    }

    return processed;
  }, [data, sortConfig, filterConfig, searchTerm, columns]);

  // Paginate processed data
  const paginatedData = useMemo(() => {
    return ArrayUtils.paginate(processedData, currentPage, pageSize);
  }, [processedData, currentPage, pageSize]);

  // Handle sorting
  const handleSort = (key) => {
    setSortConfig(prevConfig => ({
      key,
      direction: prevConfig.key === key && prevConfig.direction === 'asc' ? 'desc' : 'asc'
    }));
  };

  // Handle filtering
  const handleFilter = (key, value) => {
    setFilterConfig(prev => ({ ...prev, [key]: value }));
    setCurrentPage(1); // Reset to first page when filtering
  };

  // Generate table statistics
  const stats = useMemo(() => {
    const total = data.length;
    const filtered = processedData.length;
    const showing = paginatedData.data.length;
    
    return {
      total,
      filtered,
      showing,
      startIndex: (currentPage - 1) * pageSize + 1,
      endIndex: Math.min(currentPage * pageSize, filtered)
    };
  }, [data.length, processedData.length, paginatedData.data.length, currentPage, pageSize]);

  const getSortIcon = (columnKey) => {
    if (sortConfig.key !== columnKey) return '↕️';
    return sortConfig.direction === 'asc' ? '⬆️' : '⬇️';
  };

  return (
    <div className="data-table">
      {/* Search and Controls */}
      <div className="table-controls">
        <div className="search-controls">
          <input
            type="text"
            placeholder="Search all columns..."
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            className="search-input"
          />
        </div>
        
        <div className="page-size-control">
          <label>Show:</label>
          <select value={pageSize} onChange={(e) => setPageSize(Number(e.target.value))}>
            <option value={5}>5</option>
            <option value={10}>10</option>
            <option value={25}>25</option>
            <option value={50}>50</option>
          </select>
          <span>entries</span>
        </div>
      </div>

      {/* Table Stats */}
      <div className="table-stats">
        {stats.filtered < stats.total ? (
          <span>
            Showing {stats.startIndex} to {stats.endIndex} of {stats.filtered} entries 
            (filtered from {stats.total} total entries)
          </span>
        ) : (
          <span>
            Showing {stats.startIndex} to {stats.endIndex} of {stats.total} entries
          </span>
        )}
      </div>

      {/* Table */}
      <table className="table">
        <thead>
          <tr>
            {columns.map(column => (
              <th key={column.key}>
                <div className="column-header">
                  <span
                    onClick={() => handleSort(column.key)}
                    className="sortable"
                  >
                    {column.label} {getSortIcon(column.key)}
                  </span>
                  
                  {column.filterable && (
                    <input
                      type="text"
                      placeholder={`Filter ${column.label}...`}
                      value={filterConfig[column.key] || ''}
                      onChange={(e) => handleFilter(column.key, e.target.value)}
                      className="column-filter"
                    />
                  )}
                </div>
              </th>
            ))}
          </tr>
        </thead>
        
        <tbody>
          {paginatedData.data.map((row, index) => (
            <tr key={index}>
              {columns.map(column => (
                <td key={column.key}>
                  {column.render ? column.render(row[column.key], row) : row[column.key]}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>

      {/* Pagination */}
      {paginatedData.pagination.totalPages > 1 && (
        <div className="pagination">
          <button
            onClick={() => setCurrentPage(1)}
            disabled={!paginatedData.pagination.hasPrev}
          >
            First
          </button>
          
          <button
            onClick={() => setCurrentPage(currentPage - 1)}
            disabled={!paginatedData.pagination.hasPrev}
          >
            Previous
          </button>
          
          <span className="page-info">
            Page {currentPage} of {paginatedData.pagination.totalPages}
          </span>
          
          <button
            onClick={() => setCurrentPage(currentPage + 1)}
            disabled={!paginatedData.pagination.hasNext}
          >
            Next
          </button>
          
          <button
            onClick={() => setCurrentPage(paginatedData.pagination.totalPages)}
            disabled={!paginatedData.pagination.hasNext}
          >
            Last
          </button>
        </div>
      )}
    </div>
  );
};

// Usage Example
const ProductsTable = () => {
  const products = [
    { id: 1, name: 'Laptop', category: 'Electronics', price: 1000, inStock: true, rating: 4.5 },
    { id: 2, name: 'Phone', category: 'Electronics', price: 500, inStock: false, rating: 4.2 },
    { id: 3, name: 'Book', category: 'Education', price: 20, inStock: true, rating: 4.8 },
    // ... more products
  ];

  const columns = [
    { key: 'id', label: 'ID', filterable: true },
    { key: 'name', label: 'Name', filterable: true },
    { key: 'category', label: 'Category', filterable: true },
    { 
      key: 'price', 
      label: 'Price', 
      filterable: true,
      render: (value) => `$${value.toLocaleString()}`
    },
    { 
      key: 'inStock', 
      label: 'Stock Status',
      render: (value) => value ? '✅ In Stock' : '❌ Out of Stock'
    },
    { 
      key: 'rating', 
      label: 'Rating',
      render: (value) => '⭐'.repeat(Math.floor(value)) + ` (${value})`
    }
  ];

  return (
    <div className="products-page">
      <h1>Products</h1>
      <DataTable data={products} columns={columns} />
    </div>
  );
};

export default ProductsTable;
```

### **📈 Analytics Dashboard**

```jsx
// components/AnalyticsDashboard.jsx
import { useState, useMemo } from 'react';

const AnalyticsDashboard = ({ salesData, dateRange }) => {
  const [selectedMetric, setSelectedMetric] = useState('revenue');
  const [groupBy, setGroupBy] = useState('month');

  // Process sales data for analytics
  const analytics = useMemo(() => {
    // Filter by date range
    const filteredData = salesData.filter(sale => {
      const saleDate = new Date(sale.date);
      return saleDate >= dateRange.start && saleDate <= dateRange.end;
    });

    // Group data by time period
    const groupedData = ArrayUtils.groupBy(filteredData, sale => {
      const date = new Date(sale.date);
      switch (groupBy) {
        case 'day':
          return date.toISOString().split('T')[0];
        case 'week':
          const weekStart = new Date(date);
          weekStart.setDate(date.getDate() - date.getDay());
          return weekStart.toISOString().split('T')[0];
        case 'month':
          return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`;
        case 'year':
          return date.getFullYear().toString();
        default:
          return 'all';
      }
    });

    // Calculate metrics for each group
    const processedGroups = Object.entries(groupedData).map(([period, sales]) => {
      const revenue = sales.reduce((sum, sale) => sum + sale.amount, 0);
      const orders = sales.length;
      const averageOrderValue = orders > 0 ? revenue / orders : 0;
      const uniqueCustomers = new Set(sales.map(sale => sale.customerId)).size;
      
      // Top products in this period
      const productSales = ArrayUtils.groupBy(sales, 'productId');
      const topProducts = Object.entries(productSales)
        .map(([productId, productSales]) => ({
          productId,
          quantity: productSales.reduce((sum, sale) => sum + sale.quantity, 0),
          revenue: productSales.reduce((sum, sale) => sum + sale.amount, 0),
          orders: productSales.length
        }))
        .sort((a, b) => b.revenue - a.revenue)
        .slice(0, 5);

      return {
        period,
        revenue,
        orders,
        averageOrderValue,
        uniqueCustomers,
        topProducts,
        growthRate: 0 // Will calculate later
      };
    });

    // Sort by period
    processedGroups.sort((a, b) => a.period.localeCompare(b.period));

    // Calculate growth rates
    processedGroups.forEach((group, index) => {
      if (index > 0) {
        const prev = processedGroups[index - 1];
        group.growthRate = prev[selectedMetric] > 0 
          ? ((group[selectedMetric] - prev[selectedMetric]) / prev[selectedMetric]) * 100 
          : 0;
      }
    });

    // Overall statistics
    const totalRevenue = filteredData.reduce((sum, sale) => sum + sale.amount, 0);
    const totalOrders = filteredData.length;
    const averageOrderValue = totalOrders > 0 ? totalRevenue / totalOrders : 0;
    const uniqueCustomers = new Set(filteredData.map(sale => sale.customerId)).size;

    // Top performers
    const topCustomers = Object.entries(ArrayUtils.groupBy(filteredData, 'customerId'))
      .map(([customerId, sales]) => ({
        customerId,
        revenue: sales.reduce((sum, sale) => sum + sale.amount, 0),
        orders: sales.length,
        averageOrderValue: sales.reduce((sum, sale) => sum + sale.amount, 0) / sales.length
      }))
      .sort((a, b) => b.revenue - a.revenue)
      .slice(0, 10);

    const topProducts = Object.entries(ArrayUtils.groupBy(filteredData, 'productId'))
      .map(([productId, sales]) => ({
        productId,
        revenue: sales.reduce((sum, sale) => sum + sale.amount, 0),
        quantity: sales.reduce((sum, sale) => sum + sale.quantity, 0),
        orders: sales.length
      }))
      .sort((a, b) => b.revenue - a.revenue)
      .slice(0, 10);

    return {
      timeSeries: processedGroups,
      totals: {
        revenue: totalRevenue,
        orders: totalOrders,
        averageOrderValue,
        uniqueCustomers
      },
      topCustomers,
      topProducts
    };
  }, [salesData, dateRange, groupBy, selectedMetric]);

  // Format currency
  const formatCurrency = (amount) => `$${amount.toLocaleString(undefined, { minimumFractionDigits: 2 })}`;

  // Format percentage
  const formatPercentage = (value) => `${value > 0 ? '+' : ''}${value.toFixed(1)}%`;

  return (
    <div className="analytics-dashboard">
      <div className="dashboard-header">
        <h1>Sales Analytics</h1>
        
        <div className="dashboard-controls">
          <select value={groupBy} onChange={(e) => setGroupBy(e.target.value)}>
            <option value="day">Daily</option>
            <option value="week">Weekly</option>
            <option value="month">Monthly</option>
            <option value="year">Yearly</option>
          </select>
          
          <select value={selectedMetric} onChange={(e) => setSelectedMetric(e.target.value)}>
            <option value="revenue">Revenue</option>
            <option value="orders">Orders</option>
            <option value="averageOrderValue">Avg. Order Value</option>
            <option value="uniqueCustomers">Unique Customers</option>
          </select>
        </div>
      </div>

      {/* Summary Cards */}
      <div className="summary-cards">
        <div className="summary-card">
          <h3>Total Revenue</h3>
          <p className="metric-value">{formatCurrency(analytics.totals.revenue)}</p>
        </div>
        
        <div className="summary-card">
          <h3>Total Orders</h3>
          <p className="metric-value">{analytics.totals.orders.toLocaleString()}</p>
        </div>
        
        <div className="summary-card">
          <h3>Avg. Order Value</h3>
          <p className="metric-value">{formatCurrency(analytics.totals.averageOrderValue)}</p>
        </div>
        
        <div className="summary-card">
          <h3>Unique Customers</h3>
          <p className="metric-value">{analytics.totals.uniqueCustomers.toLocaleString()}</p>
        </div>
      </div>

      {/* Time Series Chart */}
      <div className="chart-container">
        <h2>Trends Over Time</h2>
        <div className="time-series-data">
          {analytics.timeSeries.map((dataPoint, index) => (
            <div key={dataPoint.period} className="data-point">
              <div className="period">{dataPoint.period}</div>
              <div className="metric-value">
                {selectedMetric === 'revenue' || selectedMetric === 'averageOrderValue' 
                  ? formatCurrency(dataPoint[selectedMetric])
                  : dataPoint[selectedMetric].toLocaleString()
                }
              </div>
              {index > 0 && (
                <div className={`growth-rate ${dataPoint.growthRate >= 0 ? 'positive' : 'negative'}`}>
                  {formatPercentage(dataPoint.growthRate)}
                </div>
              )}
            </div>
          ))}
        </div>
      </div>

      {/* Top Performers */}
      <div className="top-performers">
        <div className="top-customers">
          <h2>Top Customers</h2>
          <table>
            <thead>
              <tr>
                <th>Customer ID</th>
                <th>Revenue</th>
                <th>Orders</th>
                <th>Avg. Order Value</th>
              </tr>
            </thead>
            <tbody>
              {analytics.topCustomers.map(customer => (
                <tr key={customer.customerId}>
                  <td>{customer.customerId}</td>
                  <td>{formatCurrency(customer.revenue)}</td>
                  <td>{customer.orders}</td>
                  <td>{formatCurrency(customer.averageOrderValue)}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>

        <div className="top-products">
          <h2>Top Products</h2>
          <table>
            <thead>
              <tr>
                <th>Product ID</th>
                <th>Revenue</th>
                <th>Quantity Sold</th>
                <th>Orders</th>
              </tr>
            </thead>
            <tbody>
              {analytics.topProducts.map(product => (
                <tr key={product.productId}>
                  <td>{product.productId}</td>
                  <td>{formatCurrency(product.revenue)}</td>
                  <td>{product.quantity.toLocaleString()}</td>
                  <td>{product.orders}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  );
};

export default AnalyticsDashboard;
```

### **🔍 API Data Processing**

```javascript
// pages/api/products/search.js
export default async function handler(req, res) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const {
      q: query = '',
      category,
      minPrice,
      maxPrice,
      inStock,
      sortBy = 'relevance',
      page = 1,
      limit = 20
    } = req.query;

    // Get all products (in real app, this would be from database)
    const allProducts = await getProductsFromDatabase();

    // Apply filters
    let filteredProducts = allProducts;

    // Text search
    if (query) {
      filteredProducts = filteredProducts.filter(product => {
        const searchFields = [
          product.name,
          product.description,
          product.brand,
          ...(product.tags || [])
        ].join(' ').toLowerCase();
        
        return query.toLowerCase().split(' ').every(term =>
          searchFields.includes(term)
        );
      });
    }

    // Category filter
    if (category) {
      filteredProducts = filteredProducts.filter(product => 
        product.category === category
      );
    }

    // Price range filter
    if (minPrice) {
      filteredProducts = filteredProducts.filter(product => 
        product.price >= parseFloat(minPrice)
      );
    }
    if (maxPrice) {
      filteredProducts = filteredProducts.filter(product => 
        product.price <= parseFloat(maxPrice)
      );
    }

    // Stock filter
    if (inStock !== undefined) {
      const stockFilter = inStock === 'true';
      filteredProducts = filteredProducts.filter(product => 
        product.inStock === stockFilter
      );
    }

    // Apply sorting
    const sortedProducts = ArrayUtils.sortBy(filteredProducts, 
      getSortConfig(sortBy, query)
    );

    // Apply pagination
    const paginatedResults = ArrayUtils.paginate(
      sortedProducts, 
      parseInt(page), 
      parseInt(limit)
    );

    // Generate facets for filtering UI
    const facets = generateFacets(allProducts, filteredProducts);

    res.status(200).json({
      results: paginatedResults.data,
      pagination: paginatedResults.pagination,
      facets,
      total: filteredProducts.length,
      query: {
        q: query,
        category,
        minPrice,
        maxPrice,
        inStock,
        sortBy,
        page: parseInt(page),
        limit: parseInt(limit)
      }
    });

  } catch (error) {
    console.error('Search error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}

function getSortConfig(sortBy, query) {
  switch (sortBy) {
    case 'price-asc':
      return { key: 'price', direction: 'asc' };
    case 'price-desc':
      return { key: 'price', direction: 'desc' };
    case 'name':
      return { key: 'name', direction: 'asc' };
    case 'rating':
      return { key: 'rating', direction: 'desc' };
    case 'newest':
      return { key: 'createdAt', direction: 'desc' };
    case 'relevance':
    default:
      // For relevance, we could implement a scoring algorithm
      return query ? (product) => calculateRelevanceScore(product, query) : { key: 'featured', direction: 'desc' };
  }
}

function calculateRelevanceScore(product, query) {
  let score = 0;
  const queryTerms = query.toLowerCase().split(' ');
  
  queryTerms.forEach(term => {
    // Exact match in name gets highest score
    if (product.name.toLowerCase().includes(term)) {
      score += 10;
    }
    
    // Match in description
    if (product.description && product.description.toLowerCase().includes(term)) {
      score += 5;
    }
    
    // Match in tags
    if (product.tags && product.tags.some(tag => tag.toLowerCase().includes(term))) {
      score += 3;
    }
    
    // Match in brand
    if (product.brand && product.brand.toLowerCase().includes(term)) {
      score += 2;
    }
  });
  
  // Boost score based on popularity metrics
  score += (product.rating || 0) * 0.5;
  score += (product.reviewCount || 0) * 0.01;
  
  return score;
}

function generateFacets(allProducts, filteredProducts) {
  return {
    categories: ArrayUtils.countBy(allProducts, 'category'),
    brands: ArrayUtils.countBy(allProducts, 'brand'),
    priceRanges: {
      'under-50': filteredProducts.filter(p => p.price < 50).length,
      '50-100': filteredProducts.filter(p => p.price >= 50 && p.price < 100).length,
      '100-200': filteredProducts.filter(p => p.price >= 100 && p.price < 200).length,
      'over-200': filteredProducts.filter(p => p.price >= 200).length
    },
    availability: {
      inStock: filteredProducts.filter(p => p.inStock).length,
      outOfStock: filteredProducts.filter(p => !p.inStock).length
    }
  };
}

// pages/api/analytics/summary.js
export default async function handler(req, res) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { startDate, endDate, groupBy = 'day' } = req.query;

    const salesData = await getSalesData(startDate, endDate);
    
    // Process sales data using array methods
    const summary = salesData
      .filter(sale => sale.status === 'completed')
      .reduce((acc, sale) => {
        // Group by time period
        const period = getTimePeriod(sale.date, groupBy);
        
        if (!acc[period]) {
          acc[period] = {
            revenue: 0,
            orders: 0,
            customers: new Set(),
            products: new Map()
          };
        }
        
        acc[period].revenue += sale.amount;
        acc[period].orders += 1;
        acc[period].customers.add(sale.customerId);
        
        // Track product sales
        sale.items.forEach(item => {
          const productId = item.productId;
          const existing = acc[period].products.get(productId) || { quantity: 0, revenue: 0 };
          existing.quantity += item.quantity;
          existing.revenue += item.price * item.quantity;
          acc[period].products.set(productId, existing);
        });
        
        return acc;
      }, {});

    // Convert to array and add derived metrics
    const timeSeriesData = Object.entries(summary).map(([period, data]) => ({
      period,
      revenue: data.revenue,
      orders: data.orders,
      uniqueCustomers: data.customers.size,
      averageOrderValue: data.orders > 0 ? data.revenue / data.orders : 0,
      topProducts: Array.from(data.products.entries())
        .map(([productId, stats]) => ({ productId, ...stats }))
        .sort((a, b) => b.revenue - a.revenue)
        .slice(0, 5)
    }));

    // Sort by period
    timeSeriesData.sort((a, b) => a.period.localeCompare(b.period));

    // Calculate growth rates
    timeSeriesData.forEach((data, index) => {
      if (index > 0) {
        const previous = timeSeriesData[index - 1];
        data.revenueGrowth = previous.revenue > 0 
          ? ((data.revenue - previous.revenue) / previous.revenue) * 100 
          : 0;
        data.orderGrowth = previous.orders > 0 
          ? ((data.orders - previous.orders) / previous.orders) * 100 
          : 0;
      }
    });

    res.status(200).json({
      timeSeries: timeSeriesData,
      totals: {
        revenue: timeSeriesData.reduce((sum, data) => sum + data.revenue, 0),
        orders: timeSeriesData.reduce((sum, data) => sum + data.orders, 0),
        uniqueCustomers: new Set(salesData.map(sale => sale.customerId)).size
      }
    });

  } catch (error) {
    console.error('Analytics error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Array Processing**
สร้างฟังก์ชันประมวลผล array:

```javascript
// TODO: สร้างฟังก์ชันเหล่านี้

const orders = [
  { id: 1, customerId: 101, items: [{ productId: 'A', price: 100, quantity: 2 }], date: '2024-01-01' },
  { id: 2, customerId: 102, items: [{ productId: 'B', price: 50, quantity: 1 }], date: '2024-01-02' },
  // ... more orders
];

// 1. หาลูกค้าที่ซื้อสินค้ามากที่สุด
function findTopCustomer(orders) {
  // Return { customerId, totalSpent, orderCount }
}

// 2. คำนวณยอดขายรายเดือน
function calculateMonthlySales(orders) {
  // Return [{ month: '2024-01', revenue: 1000, orderCount: 10 }]
}

// 3. หาสินค้าที่ขายดีที่สุด
function findBestSellingProducts(orders, limit = 5) {
  // Return [{ productId, totalQuantity, totalRevenue }]
}

// 4. สร้าง cohort analysis
function generateCohortAnalysis(orders) {
  // Group customers by first purchase month
  // Track their purchases in subsequent months
}
```

### **🔤 แบบฝึกหัดที่ 2: Data Transformation**
แปลงข้อมูลให้อยู่ในรูปแบบที่ต้องการ:

```javascript
// TODO: แปลงข้อมูลเหล่านี้

const rawData = [
  'John,Doe,30,Engineer,New York',
  'Jane,Smith,25,Designer,Los Angeles',
  'Bob,Johnson,35,Manager,Chicago'
];

// 1. แปลง CSV เป็น objects
function parseCSV(csvData, headers) {
  // Return array of objects
}

// 2. Group และ aggregate data
const salesData = [
  { region: 'North', product: 'A', sales: 100 },
  { region: 'North', product: 'B', sales: 150 },
  { region: 'South', product: 'A', sales: 200 }
];

function createPivotTable(data, rowKey, colKey, valueKey) {
  // Create pivot table structure
}

// 3. Normalize nested data
const nestedData = [
  { 
    user: { id: 1, name: 'John' },
    orders: [
      { id: 101, total: 100 },
      { id: 102, total: 200 }
    ]
  }
];

function flattenUserOrders(data) {
  // Return flat array with user info repeated for each order
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง component ที่ใช้ array methods:

```jsx
// TODO: สร้าง RealtimeMetrics component ที่:
// 1. แสดง metrics แบบ real-time
// 2. ใช้ array methods ในการประมวลผล
// 3. มี filters และ time range selection
// 4. แสดงผลเป็น charts และ tables

const RealtimeMetrics = ({ data, updateInterval = 5000 }) => {
  // Implementation here
  // Use array methods for data processing
  // Include filtering, sorting, grouping
  // Real-time updates with useEffect
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 6: Template Literals & String Methods](./06-template-literals.md)

**บทถัดไป**: [บทที่ 8: Objects & Classes](./08-objects-classes.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Array Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [JavaScript.info: Array Methods](https://javascript.info/array-methods)
- [Functional Programming with Arrays](https://eloquentjavascript.net/05_higher_order.html)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Modern Array Methods** สำหรับการค้นหาและประมวลผล
- **Functional Programming** กับ arrays
- **Data Transformation** และ aggregation
- **การใช้งานใน Next.js** สำหรับ data handling
- **Performance Optimization** สำหรับ array operations

Array methods ใน ES6+ ช่วยให้การประมวลผลข้อมูลเป็นไปอย่างมีประสิทธิภาพและอ่านง่ายมากขึ้น! 🚀
