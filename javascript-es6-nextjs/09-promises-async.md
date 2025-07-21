# บทที่ 9: Promises & Async/Await

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Promises และ Asynchronous Programming
- ทำความเข้าใจ Async/Await Syntax
- เรียนรู้ Error Handling ใน Async Code
- ประยุกต์ใช้ใน Next.js สำหรับ Data Fetching

## 🔄 Understanding Promises

### **⚡ Promise Basics**

```javascript
// ============= PROMISE FUNDAMENTALS =============
// Creating a basic Promise
const createPromise = (shouldResolve = true, delay = 1000) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (shouldResolve) {
        resolve(`Promise resolved after ${delay}ms`);
      } else {
        reject(new Error(`Promise rejected after ${delay}ms`));
      }
    }, delay);
  });
};

// Promise states: pending, fulfilled, rejected
const pendingPromise = createPromise(true, 2000);
console.log('Promise state:', pendingPromise); // Promise { <pending> }

// ============= PROMISE CHAINING =============
const fetchUserData = (userId) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        id: userId,
        name: `User ${userId}`,
        email: `user${userId}@example.com`
      });
    }, 500);
  });
};

const fetchUserPosts = (userId) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, title: 'First Post', userId },
        { id: 2, title: 'Second Post', userId }
      ]);
    }, 300);
  });
};

const fetchPostComments = (postId) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, text: 'Great post!', postId },
        { id: 2, text: 'Thanks for sharing', postId }
      ]);
    }, 200);
  });
};

// Chaining with .then()
fetchUserData(1)
  .then(user => {
    console.log('User:', user);
    return fetchUserPosts(user.id);
  })
  .then(posts => {
    console.log('Posts:', posts);
    return fetchPostComments(posts[0].id);
  })
  .then(comments => {
    console.log('Comments:', comments);
  })
  .catch(error => {
    console.error('Error in chain:', error);
  })
  .finally(() => {
    console.log('Promise chain completed');
  });

// ============= PROMISE COMBINATORS =============
// Promise.all - Wait for all promises to resolve
const fetchAllUserData = async (userId) => {
  try {
    const [user, posts] = await Promise.all([
      fetchUserData(userId),
      fetchUserPosts(userId)
    ]);
    
    return {
      user,
      posts,
      combinedAt: new Date()
    };
  } catch (error) {
    throw new Error(`Failed to fetch user data: ${error.message}`);
  }
};

// Promise.allSettled - Wait for all promises to settle
const fetchUserDataSafely = async (userIds) => {
  const promises = userIds.map(id => fetchUserData(id));
  const results = await Promise.allSettled(promises);
  
  const succeeded = results
    .filter(result => result.status === 'fulfilled')
    .map(result => result.value);
  
  const failed = results
    .filter(result => result.status === 'rejected')
    .map(result => result.reason);
  
  return { succeeded, failed };
};

// Promise.race - First promise to settle wins
const fetchWithTimeout = (promise, timeoutMs) => {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Operation timed out')), timeoutMs);
  });
  
  return Promise.race([promise, timeoutPromise]);
};

// Usage
fetchWithTimeout(fetchUserData(1), 1000)
  .then(user => console.log('User fetched in time:', user))
  .catch(error => console.error('Timeout or error:', error));

// Promise.any - First fulfilled promise wins (ES2021)
const fetchFromMultipleSources = async (userId) => {
  const sources = [
    fetch(`/api/users/${userId}`),
    fetch(`/api/backup/users/${userId}`),
    fetch(`/api/cache/users/${userId}`)
  ];
  
  try {
    const firstResponse = await Promise.any(sources);
    return await firstResponse.json();
  } catch (error) {
    throw new Error('All sources failed');
  }
};

// ============= ADVANCED PROMISE PATTERNS =============
class PromiseQueue {
  constructor(concurrency = 3) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  add(promiseFactory) {
    return new Promise((resolve, reject) => {
      this.queue.push({
        promiseFactory,
        resolve,
        reject
      });
      
      this.tryNext();
    });
  }
  
  tryNext() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { promiseFactory, resolve, reject } = this.queue.shift();
    
    promiseFactory()
      .then(resolve)
      .catch(reject)
      .finally(() => {
        this.running--;
        this.tryNext();
      });
  }
}

// Usage
const queue = new PromiseQueue(2);

const fetchOperations = Array.from({ length: 10 }, (_, i) => 
  queue.add(() => fetchUserData(i + 1))
);

Promise.all(fetchOperations)
  .then(users => console.log('All users fetched:', users))
  .catch(error => console.error('Queue error:', error));

// ============= RETRY PATTERN =============
const retryWithBackoff = async (operation, maxRetries = 3, baseDelay = 1000) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries) {
        throw new Error(`Operation failed after ${maxRetries} attempts: ${error.message}`);
      }
      
      const delay = baseDelay * Math.pow(2, attempt - 1); // Exponential backoff
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};

// Usage
const unreliableOperation = () => {
  return new Promise((resolve, reject) => {
    if (Math.random() < 0.7) {
      reject(new Error('Random failure'));
    } else {
      resolve('Success!');
    }
  });
};

retryWithBackoff(unreliableOperation, 5, 500)
  .then(result => console.log('Operation succeeded:', result))
  .catch(error => console.error('Operation failed permanently:', error));
```

## 🚀 Async/Await

### **🔄 Async Function Syntax**

```javascript
// ============= BASIC ASYNC/AWAIT =============
// Converting Promise chains to async/await
const getUserDataChain = (userId) => {
  return fetchUserData(userId)
    .then(user => {
      return fetchUserPosts(user.id)
        .then(posts => {
          return { user, posts };
        });
    });
};

// Equivalent async/await version
const getUserDataAsync = async (userId) => {
  try {
    const user = await fetchUserData(userId);
    const posts = await fetchUserPosts(user.id);
    return { user, posts };
  } catch (error) {
    throw new Error(`Failed to get user data: ${error.message}`);
  }
};

// ============= PARALLEL EXECUTION =============
// Sequential execution (slower)
const fetchUserDataSequential = async (userId) => {
  const user = await fetchUserData(userId);      // Wait 500ms
  const posts = await fetchUserPosts(userId);    // Wait another 300ms
  const profile = await fetchUserProfile(userId); // Wait another 400ms
  
  return { user, posts, profile }; // Total: ~1200ms
};

// Parallel execution (faster)
const fetchUserDataParallel = async (userId) => {
  const [user, posts, profile] = await Promise.all([
    fetchUserData(userId),      // All start together
    fetchUserPosts(userId),     // All start together
    fetchUserProfile(userId)    // All start together
  ]);
  
  return { user, posts, profile }; // Total: ~500ms (slowest operation)
};

// ============= ERROR HANDLING =============
const robustDataFetcher = async (userId) => {
  try {
    // Try primary source
    const user = await fetchUserData(userId);
    
    try {
      // Try to get additional data
      const posts = await fetchUserPosts(userId);
      return { user, posts, source: 'complete' };
    } catch (postsError) {
      console.warn('Could not fetch posts:', postsError.message);
      return { user, posts: [], source: 'partial' };
    }
    
  } catch (userError) {
    console.error('Could not fetch user:', userError.message);
    
    // Try fallback
    try {
      const cachedUser = await fetchCachedUser(userId);
      return { user: cachedUser, posts: [], source: 'cached' };
    } catch (cacheError) {
      throw new Error('All data sources failed');
    }
  }
};

// ============= ASYNC GENERATORS =============
async function* fetchUsersPaginated(pageSize = 10) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    try {
      const response = await fetch(`/api/users?page=${page}&limit=${pageSize}`);
      const data = await response.json();
      
      if (data.users.length === 0) {
        hasMore = false;
      } else {
        yield data.users;
        page++;
        hasMore = data.hasMore;
      }
    } catch (error) {
      console.error('Error fetching page:', error);
      hasMore = false;
    }
  }
}

// Usage
const processAllUsers = async () => {
  for await (const userBatch of fetchUsersPaginated(5)) {
    console.log(`Processing batch of ${userBatch.length} users`);
    
    // Process each user in the batch
    await Promise.all(userBatch.map(async (user) => {
      // Simulate processing
      await new Promise(resolve => setTimeout(resolve, 100));
      console.log(`Processed user: ${user.name}`);
    }));
  }
};

// ============= ADVANCED ASYNC PATTERNS =============
class AsyncCache {
  constructor(ttl = 60000) { // 1 minute TTL
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  async get(key, fetchFunction) {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.data;
    }
    
    try {
      const data = await fetchFunction();
      this.cache.set(key, {
        data,
        timestamp: Date.now()
      });
      return data;
    } catch (error) {
      // Return stale data if available
      if (cached) {
        console.warn('Using stale data due to fetch error:', error.message);
        return cached.data;
      }
      throw error;
    }
  }
  
  clear() {
    this.cache.clear();
  }
  
  delete(key) {
    this.cache.delete(key);
  }
}

// Usage
const cache = new AsyncCache(30000); // 30 second TTL

const getCachedUserData = async (userId) => {
  return cache.get(`user:${userId}`, () => fetchUserData(userId));
};

// ============= ASYNC ITERATION =============
const processUsersSequentially = async (userIds) => {
  const results = [];
  
  for (const userId of userIds) {
    try {
      const userData = await getCachedUserData(userId);
      const processedData = await processUser(userData);
      results.push(processedData);
    } catch (error) {
      console.error(`Failed to process user ${userId}:`, error);
      results.push({ error: error.message, userId });
    }
  }
  
  return results;
};

const processUsersConcurrently = async (userIds, concurrency = 3) => {
  const results = [];
  const chunks = [];
  
  // Split into chunks
  for (let i = 0; i < userIds.length; i += concurrency) {
    chunks.push(userIds.slice(i, i + concurrency));
  }
  
  // Process each chunk
  for (const chunk of chunks) {
    const chunkResults = await Promise.allSettled(
      chunk.map(async (userId) => {
        const userData = await getCachedUserData(userId);
        return await processUser(userData);
      })
    );
    
    results.push(...chunkResults);
  }
  
  return results;
};
```

### **🛠️ Advanced Async Utilities**

```javascript
// ============= ASYNC UTILITY FUNCTIONS =============
class AsyncUtils {
  // Sleep/delay function
  static sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Timeout wrapper
  static timeout(promise, ms, errorMessage = 'Operation timed out') {
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error(errorMessage)), ms);
    });
    
    return Promise.race([promise, timeoutPromise]);
  }
  
  // Retry with exponential backoff
  static async retry(fn, options = {}) {
    const {
      retries = 3,
      delay = 1000,
      backoff = 2,
      jitter = true
    } = options;
    
    for (let attempt = 1; attempt <= retries; attempt++) {
      try {
        return await fn();
      } catch (error) {
        if (attempt === retries) throw error;
        
        let waitTime = delay * Math.pow(backoff, attempt - 1);
        
        // Add jitter to prevent thundering herd
        if (jitter) {
          waitTime += Math.random() * waitTime * 0.1;
        }
        
        await this.sleep(waitTime);
      }
    }
  }
  
  // Map with concurrency control
  static async mapWithConcurrency(items, fn, concurrency = 3) {
    const results = [];
    const executing = [];
    
    for (const item of items) {
      const promise = fn(item).then(result => {
        executing.splice(executing.indexOf(promise), 1);
        return result;
      });
      
      results.push(promise);
      executing.push(promise);
      
      if (executing.length >= concurrency) {
        await Promise.race(executing);
      }
    }
    
    return Promise.all(results);
  }
  
  // Waterfall execution
  static async waterfall(functions, initialValue) {
    let result = initialValue;
    
    for (const fn of functions) {
      result = await fn(result);
    }
    
    return result;
  }
  
  // Parallel execution with results
  static async parallel(tasks) {
    const results = await Promise.allSettled(
      Object.entries(tasks).map(async ([key, task]) => {
        const result = await task();
        return { key, result };
      })
    );
    
    const success = {};
    const errors = {};
    
    results.forEach(({ status, value, reason }) => {
      if (status === 'fulfilled') {
        success[value.key] = value.result;
      } else {
        errors[value?.key || 'unknown'] = reason;
      }
    });
    
    return { success, errors };
  }
  
  // Race with timeout and fallback
  static async raceWithFallback(primary, fallback, timeout = 5000) {
    try {
      return await this.timeout(primary(), timeout);
    } catch (error) {
      console.warn('Primary failed, using fallback:', error.message);
      return await fallback();
    }
  }
}

// ============= USAGE EXAMPLES =============
const demonstrateAsyncUtils = async () => {
  // Retry example
  const unreliableApi = () => {
    if (Math.random() < 0.7) {
      throw new Error('API temporarily unavailable');
    }
    return Promise.resolve('API response');
  };
  
  try {
    const result = await AsyncUtils.retry(unreliableApi, {
      retries: 5,
      delay: 500,
      backoff: 1.5
    });
    console.log('Retry result:', result);
  } catch (error) {
    console.error('Retry failed:', error.message);
  }
  
  // Concurrent mapping
  const userIds = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  
  const processUser = async (userId) => {
    await AsyncUtils.sleep(Math.random() * 1000); // Simulate work
    return { userId, processed: true, timestamp: Date.now() };
  };
  
  const processedUsers = await AsyncUtils.mapWithConcurrency(
    userIds, 
    processUser, 
    3 // Max 3 concurrent operations
  );
  
  console.log('Processed users:', processedUsers);
  
  // Waterfall example
  const waterflowOperations = [
    async (data) => ({ ...data, step1: 'completed' }),
    async (data) => ({ ...data, step2: 'completed' }),
    async (data) => ({ ...data, step3: 'completed' }),
  ];
  
  const waterfallResult = await AsyncUtils.waterfall(
    waterflowOperations, 
    { initial: true }
  );
  
  console.log('Waterfall result:', waterfallResult);
  
  // Parallel tasks
  const parallelTasks = {
    userData: () => fetchUserData(1),
    userPosts: () => fetchUserPosts(1),
    userSettings: () => AsyncUtils.sleep(500).then(() => ({ theme: 'dark' }))
  };
  
  const { success, errors } = await AsyncUtils.parallel(parallelTasks);
  console.log('Parallel success:', success);
  console.log('Parallel errors:', errors);
};
```

## 🚀 Next.js Applications

### **📡 API Routes with Async/Await**

```javascript
// pages/api/users/[id].js
import { AsyncUtils } from '../../../utils/AsyncUtils';

// Database simulation
const db = {
  async findUser(id) {
    await AsyncUtils.sleep(Math.random() * 500);
    
    if (Math.random() < 0.1) {
      throw new Error('Database connection failed');
    }
    
    return {
      id: parseInt(id),
      name: `User ${id}`,
      email: `user${id}@example.com`,
      createdAt: new Date().toISOString()
    };
  },
  
  async updateUser(id, updates) {
    await AsyncUtils.sleep(300);
    
    return {
      id: parseInt(id),
      ...updates,
      updatedAt: new Date().toISOString()
    };
  },
  
  async deleteUser(id) {
    await AsyncUtils.sleep(200);
    return { success: true, deletedId: parseInt(id) };
  }
};

// Cache layer
const userCache = new AsyncCache(60000); // 1 minute cache

export default async function handler(req, res) {
  const { id } = req.query;
  const userId = parseInt(id);
  
  if (isNaN(userId)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  try {
    switch (req.method) {
      case 'GET':
        await handleGetUser(req, res, userId);
        break;
      case 'PUT':
        await handleUpdateUser(req, res, userId);
        break;
      case 'DELETE':
        await handleDeleteUser(req, res, userId);
        break;
      default:
        res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
        res.status(405).json({ error: 'Method not allowed' });
    }
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}

async function handleGetUser(req, res, userId) {
  try {
    // Try cache first, then database with retry
    const user = await userCache.get(`user:${userId}`, () =>
      AsyncUtils.retry(() => db.findUser(userId), {
        retries: 3,
        delay: 500
      })
    );
    
    res.status(200).json({ user });
  } catch (error) {
    if (error.message.includes('not found')) {
      res.status(404).json({ error: 'User not found' });
    } else {
      throw error;
    }
  }
}

async function handleUpdateUser(req, res, userId) {
  const updates = req.body;
  
  // Validate updates
  const allowedFields = ['name', 'email'];
  const filteredUpdates = Object.keys(updates)
    .filter(key => allowedFields.includes(key))
    .reduce((obj, key) => {
      obj[key] = updates[key];
      return obj;
    }, {});
  
  if (Object.keys(filteredUpdates).length === 0) {
    return res.status(400).json({ error: 'No valid fields to update' });
  }
  
  try {
    const updatedUser = await AsyncUtils.retry(() => 
      db.updateUser(userId, filteredUpdates), {
      retries: 2,
      delay: 300
    });
    
    // Invalidate cache
    userCache.delete(`user:${userId}`);
    
    res.status(200).json({ user: updatedUser });
  } catch (error) {
    throw error;
  }
}

async function handleDeleteUser(req, res, userId) {
  try {
    const result = await AsyncUtils.retry(() => 
      db.deleteUser(userId), {
      retries: 2,
      delay: 300
    });
    
    // Invalidate cache
    userCache.delete(`user:${userId}`);
    
    res.status(200).json(result);
  } catch (error) {
    throw error;
  }
}

// pages/api/users/batch.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    res.setHeader('Allow', ['POST']);
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { userIds, operation } = req.body;
  
  if (!Array.isArray(userIds) || userIds.length === 0) {
    return res.status(400).json({ error: 'userIds must be a non-empty array' });
  }
  
  try {
    let results;
    
    switch (operation) {
      case 'fetch':
        results = await AsyncUtils.mapWithConcurrency(
          userIds,
          async (userId) => {
            try {
              return await userCache.get(`user:${userId}`, () => 
                db.findUser(userId)
              );
            } catch (error) {
              return { error: error.message, userId };
            }
          },
          5 // Max 5 concurrent requests
        );
        break;
        
      case 'delete':
        results = await AsyncUtils.mapWithConcurrency(
          userIds,
          async (userId) => {
            try {
              const result = await db.deleteUser(userId);
              userCache.delete(`user:${userId}`);
              return result;
            } catch (error) {
              return { error: error.message, userId };
            }
          },
          3 // Max 3 concurrent deletions
        );
        break;
        
      default:
        return res.status(400).json({ error: 'Invalid operation' });
    }
    
    res.status(200).json({ results });
  } catch (error) {
    console.error('Batch operation error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

### **🔄 Client-Side Data Fetching**

```jsx
// hooks/useAsync.js
import { useState, useEffect, useCallback, useRef } from 'react';

export const useAsync = (asyncFunction, dependencies = []) => {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null
  });
  
  const cancelRef = useRef();
  
  const execute = useCallback(async (...args) => {
    // Cancel previous request
    if (cancelRef.current) {
      cancelRef.current();
    }
    
    let cancelled = false;
    cancelRef.current = () => {
      cancelled = true;
    };
    
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const result = await asyncFunction(...args);
      
      if (!cancelled) {
        setState({ data: result, loading: false, error: null });
      }
    } catch (error) {
      if (!cancelled) {
        setState({ data: null, loading: false, error });
      }
    }
  }, dependencies);
  
  useEffect(() => {
    return () => {
      if (cancelRef.current) {
        cancelRef.current();
      }
    };
  }, []);
  
  return { ...state, execute };
};

// hooks/useApiData.js
import { useState, useEffect, useCallback } from 'react';

class ApiClient {
  constructor(baseURL = '/api') {
    this.baseURL = baseURL;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    };
    
    if (config.body && typeof config.body === 'object') {
      config.body = JSON.stringify(config.body);
    }
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new Error(error.error || `HTTP ${response.status}`);
    }
    
    return response.json();
  }
  
  get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url);
  }
  
  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: data
    });
  }
  
  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: data
    });
  }
  
  delete(endpoint) {
    return this.request(endpoint, {
      method: 'DELETE'
    });
  }
}

const apiClient = new ApiClient();

export const useApiData = (endpoint, options = {}) => {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });
  
  const { 
    params = {}, 
    enabled = true, 
    refetchInterval = 0,
    onSuccess,
    onError 
  } = options;
  
  const fetchData = useCallback(async () => {
    if (!enabled) return;
    
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const result = await apiClient.get(endpoint, params);
      setState({ data: result, loading: false, error: null });
      
      if (onSuccess) {
        onSuccess(result);
      }
    } catch (error) {
      setState({ data: null, loading: false, error });
      
      if (onError) {
        onError(error);
      }
    }
  }, [endpoint, JSON.stringify(params), enabled, onSuccess, onError]);
  
  useEffect(() => {
    fetchData();
    
    if (refetchInterval > 0) {
      const interval = setInterval(fetchData, refetchInterval);
      return () => clearInterval(interval);
    }
  }, [fetchData, refetchInterval]);
  
  const mutate = useCallback(async (newData) => {
    setState(prev => ({ ...prev, data: newData }));
  }, []);
  
  const refetch = useCallback(() => {
    fetchData();
  }, [fetchData]);
  
  return { 
    ...state, 
    mutate, 
    refetch,
    apiClient 
  };
};

// components/UserManager.jsx
import { useState, useEffect } from 'react';
import { useApiData } from '../hooks/useApiData';

const UserManager = () => {
  const [selectedUsers, setSelectedUsers] = useState([]);
  const [searchQuery, setSearchQuery] = useState('');
  const [page, setPage] = useState(1);
  
  // Fetch users with pagination and search
  const { 
    data: usersData, 
    loading, 
    error, 
    refetch,
    apiClient 
  } = useApiData('/users', {
    params: {
      search: searchQuery,
      page,
      limit: 10
    },
    enabled: true,
    refetchInterval: 30000 // Refetch every 30 seconds
  });
  
  const users = usersData?.users || [];
  const totalPages = usersData?.totalPages || 1;
  
  // Batch operations
  const handleBatchDelete = async () => {
    if (selectedUsers.length === 0) return;
    
    if (!confirm(`Delete ${selectedUsers.length} users?`)) return;
    
    try {
      await apiClient.post('/users/batch', {
        userIds: selectedUsers,
        operation: 'delete'
      });
      
      setSelectedUsers([]);
      refetch(); // Refresh the list
    } catch (error) {
      alert(`Failed to delete users: ${error.message}`);
    }
  };
  
  // Individual user operations
  const handleDeleteUser = async (userId) => {
    if (!confirm('Delete this user?')) return;
    
    try {
      await apiClient.delete(`/users/${userId}`);
      refetch();
    } catch (error) {
      alert(`Failed to delete user: ${error.message}`);
    }
  };
  
  const handleUpdateUser = async (userId, updates) => {
    try {
      await apiClient.put(`/users/${userId}`, updates);
      refetch();
    } catch (error) {
      alert(`Failed to update user: ${error.message}`);
    }
  };
  
  // Search with debounce
  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setPage(1); // Reset to first page on search
    }, 500);
    
    return () => clearTimeout(timeoutId);
  }, [searchQuery]);
  
  const toggleUserSelection = (userId) => {
    setSelectedUsers(prev => 
      prev.includes(userId)
        ? prev.filter(id => id !== userId)
        : [...prev, userId]
    );
  };
  
  const selectAllUsers = () => {
    setSelectedUsers(users.map(user => user.id));
  };
  
  const clearSelection = () => {
    setSelectedUsers([]);
  };
  
  if (loading && !users.length) {
    return <div className="loading">Loading users...</div>;
  }
  
  if (error) {
    return (
      <div className="error">
        <p>Error loading users: {error.message}</p>
        <button onClick={refetch}>Retry</button>
      </div>
    );
  }
  
  return (
    <div className="user-manager">
      <div className="controls">
        <input
          type="text"
          placeholder="Search users..."
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          className="search-input"
        />
        
        <div className="batch-controls">
          <span>{selectedUsers.length} users selected</span>
          <button onClick={selectAllUsers}>Select All</button>
          <button onClick={clearSelection}>Clear</button>
          <button 
            onClick={handleBatchDelete}
            disabled={selectedUsers.length === 0}
            className="danger"
          >
            Delete Selected
          </button>
        </div>
      </div>
      
      <div className="user-list">
        {users.map(user => (
          <UserCard
            key={user.id}
            user={user}
            selected={selectedUsers.includes(user.id)}
            onToggleSelect={() => toggleUserSelection(user.id)}
            onDelete={() => handleDeleteUser(user.id)}
            onUpdate={(updates) => handleUpdateUser(user.id, updates)}
          />
        ))}
      </div>
      
      <div className="pagination">
        <button 
          onClick={() => setPage(p => Math.max(1, p - 1))}
          disabled={page === 1}
        >
          Previous
        </button>
        
        <span>Page {page} of {totalPages}</span>
        
        <button 
          onClick={() => setPage(p => Math.min(totalPages, p + 1))}
          disabled={page === totalPages}
        >
          Next
        </button>
      </div>
      
      {loading && (
        <div className="loading-overlay">Refreshing...</div>
      )}
    </div>
  );
};

const UserCard = ({ user, selected, onToggleSelect, onDelete, onUpdate }) => {
  const [editing, setEditing] = useState(false);
  const [editData, setEditData] = useState({ name: user.name, email: user.email });
  const [updating, setUpdating] = useState(false);
  
  const handleSave = async () => {
    setUpdating(true);
    try {
      await onUpdate(editData);
      setEditing(false);
    } catch (error) {
      // Error handled by parent
    } finally {
      setUpdating(false);
    }
  };
  
  const handleCancel = () => {
    setEditData({ name: user.name, email: user.email });
    setEditing(false);
  };
  
  return (
    <div className={`user-card ${selected ? 'selected' : ''}`}>
      <input
        type="checkbox"
        checked={selected}
        onChange={onToggleSelect}
        className="select-checkbox"
      />
      
      {editing ? (
        <div className="edit-form">
          <input
            type="text"
            value={editData.name}
            onChange={(e) => setEditData(prev => ({ ...prev, name: e.target.value }))}
            placeholder="Name"
          />
          <input
            type="email"
            value={editData.email}
            onChange={(e) => setEditData(prev => ({ ...prev, email: e.target.value }))}
            placeholder="Email"
          />
          <div className="edit-actions">
            <button onClick={handleSave} disabled={updating}>
              {updating ? 'Saving...' : 'Save'}
            </button>
            <button onClick={handleCancel} disabled={updating}>
              Cancel
            </button>
          </div>
        </div>
      ) : (
        <div className="user-info">
          <h3>{user.name}</h3>
          <p>{user.email}</p>
          <div className="actions">
            <button onClick={() => setEditing(true)}>Edit</button>
            <button onClick={onDelete} className="danger">Delete</button>
          </div>
        </div>
      )}
    </div>
  );
};

export default UserManager;
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Promise Management**
สร้างระบบจัดการ Promise:

```javascript
// TODO: สร้างฟังก์ชันเหล่านี้

// 1. Create a Promise pool with priority queue
class PromisePriorityQueue {
  constructor(maxConcurrency = 3) {
    // Implement priority-based promise execution
  }
  
  add(promiseFactory, priority = 0) {
    // Add promise with priority (higher number = higher priority)
  }
}

// 2. Implement circuit breaker pattern
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    // Implement circuit breaker for failing operations
  }
  
  async execute(operation) {
    // Execute with circuit breaker logic
  }
}

// 3. Create batch processor with retry
async function processBatch(items, processor, options = {}) {
  // Process items in batches with retry logic
  // options: { batchSize, maxRetries, concurrency }
}
```

### **🔄 แบบฝึกหัดที่ 2: Async Data Pipeline**
สร้าง data processing pipeline:

```javascript
// TODO: สร้าง async data pipeline

class DataPipeline {
  constructor() {
    this.steps = [];
  }
  
  addStep(name, processor) {
    // Add processing step to pipeline
  }
  
  async execute(data) {
    // Execute all steps sequentially with error handling
  }
}

// Example usage:
const pipeline = new DataPipeline();
pipeline.addStep('validate', validateData);
pipeline.addStep('transform', transformData);
pipeline.addStep('save', saveData);

const result = await pipeline.execute(inputData);
```

### **⚛️ แบบฝึกหัดที่ 3: Real-time Data Sync**
สร้าง real-time synchronization system:

```jsx
// TODO: สร้าง component ที่:
// 1. Sync data แบบ real-time
// 2. Handle offline/online states
// 3. Implement conflict resolution
// 4. Cache data locally

const RealtimeDataSync = ({ endpoint, children }) => {
  // Implement real-time data synchronization
  // Use WebSocket or polling
  // Handle network interruptions
  // Provide data to children via context
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 8: Objects & Classes](./08-objects-classes.md)

**บทถัดไป**: [บทที่ 10: ES6 Modules](./10-modules.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN: Async/Await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [JavaScript.info: Promises](https://javascript.info/promise-basics)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Promises** และการจัดการ asynchronous operations
- **Async/Await** syntax สำหรับโค้ดที่อ่านง่าย
- **Error Handling** ใน async code
- **Advanced Promise Patterns** และ utilities
- **การใช้งานใน Next.js** สำหรับ API routes และ data fetching

Promises และ Async/Await เป็นพื้นฐานสำคัญของ modern JavaScript ที่ช่วยให้การจัดการ asynchronous operations เป็นไปอย่างมีประสิทธิภาพ! 🚀
