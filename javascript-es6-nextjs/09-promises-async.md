# บทที่ 9: Promises & Async/Await

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Promises และ Asynchronous Programming
- ทำความเข้าใจ async/await syntax
- เรียนรู้ Error Handling ใน Async Code
- ประยุกต์ใช้ใน Next.js สำหรับ Data Fetching

## 🔄 Understanding Promises

### **📝 Promise Basics**

```javascript
// ============= CREATING PROMISES =============
// Basic Promise constructor
const basicPromise = new Promise((resolve, reject) => {
  const success = Math.random() > 0.5;
  
  setTimeout(() => {
    if (success) {
      resolve({ message: 'Operation successful!', data: { id: 1, name: 'John' } });
    } else {
      reject(new Error('Operation failed!'));
    }
  }, 1000);
});

// Promise states demonstration
const demonstratePromiseStates = () => {
  console.log('Creating promise...');
  
  const promise = new Promise((resolve, reject) => {
    console.log('Promise state: pending');
    
    setTimeout(() => {
      const shouldResolve = Math.random() > 0.3;
      
      if (shouldResolve) {
        console.log('Promise state: fulfilled');
        resolve('Success!');
      } else {
        console.log('Promise state: rejected');
        reject(new Error('Failed!'));
      }
    }, 2000);
  });
  
  return promise;
};

// ============= PROMISE METHODS =============
// Using .then() and .catch()
basicPromise
  .then(result => {
    console.log('Success:', result);
    return result.data; // Return value for next .then()
  })
  .then(data => {
    console.log('Extracted data:', data);
    return `Processed: ${data.name}`;
  })
  .then(processed => {
    console.log('Final result:', processed);
  })
  .catch(error => {
    console.error('Error:', error.message);
  })
  .finally(() => {
    console.log('Promise chain completed');
  });

// ============= PROMISE CHAINING =============
const fetchUserData = (userId) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (userId > 0) {
        resolve({ id: userId, name: `User ${userId}`, email: `user${userId}@example.com` });
      } else {
        reject(new Error('Invalid user ID'));
      }
    }, 500);
  });
};

const fetchUserPosts = (userId) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve([
        { id: 1, title: 'First Post', content: 'Content 1' },
        { id: 2, title: 'Second Post', content: 'Content 2' }
      ]);
    }, 300);
  });
};

const fetchUserComments = (userId) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve([
        { id: 1, text: 'Great post!', postId: 1 },
        { id: 2, text: 'Thanks for sharing', postId: 2 }
      ]);
    }, 200);
  });
};

// Sequential promise chaining
const loadUserProfile = (userId) => {
  let userData;
  
  return fetchUserData(userId)
    .then(user => {
      userData = user;
      console.log('Fetched user:', user);
      return fetchUserPosts(user.id);
    })
    .then(posts => {
      userData.posts = posts;
      console.log('Fetched posts:', posts);
      return fetchUserComments(userData.id);
    })
    .then(comments => {
      userData.comments = comments;
      console.log('Fetched comments:', comments);
      return userData;
    })
    .catch(error => {
      console.error('Error loading user profile:', error);
      throw error;
    });
};

// Usage
loadUserProfile(1)
  .then(completeProfile => {
    console.log('Complete profile:', completeProfile);
  })
  .catch(error => {
    console.error('Failed to load profile:', error);
  });

// ============= PROMISE COMBINATORS =============
// Promise.all - Wait for all promises to resolve
const loadAllUserData = async (userIds) => {
  try {
    console.log('Loading data for all users...');
    
    const userPromises = userIds.map(id => fetchUserData(id));
    const users = await Promise.all(userPromises);
    
    console.log('All users loaded:', users);
    return users;
  } catch (error) {
    console.error('Failed to load all users:', error);
    throw error;
  }
};

// Promise.allSettled - Wait for all promises to settle (resolve or reject)
const loadAllUserDataSafe = async (userIds) => {
  console.log('Loading data for all users (safe mode)...');
  
  const userPromises = userIds.map(id => fetchUserData(id));
  const results = await Promise.allSettled(userPromises);
  
  const successful = results
    .filter(result => result.status === 'fulfilled')
    .map(result => result.value);
    
  const failed = results
    .filter(result => result.status === 'rejected')
    .map(result => result.reason);
  
  console.log('Successful loads:', successful);
  console.log('Failed loads:', failed);
  
  return { successful, failed };
};

// Promise.race - Return first promise to settle
const timeoutWrapper = (promise, timeoutMs) => {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Operation timed out after ${timeoutMs}ms`));
    }, timeoutMs);
  });
  
  return Promise.race([promise, timeout]);
};

// Promise.any - Return first promise to resolve (ignores rejections)
const tryMultipleEndpoints = (endpoints) => {
  const requests = endpoints.map(endpoint => 
    fetch(endpoint).then(response => {
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return response.json();
    })
  );
  
  return Promise.any(requests);
};

// ============= ERROR HANDLING PATTERNS =============
class PromiseErrorHandler {
  static withRetry(promiseFactory, maxRetries = 3, delayMs = 1000) {
    return new Promise(async (resolve, reject) => {
      let lastError;
      
      for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
          const result = await promiseFactory();
          resolve(result);
          return;
        } catch (error) {
          lastError = error;
          console.log(`Attempt ${attempt} failed:`, error.message);
          
          if (attempt < maxRetries) {
            await new Promise(resolve => setTimeout(resolve, delayMs * attempt));
          }
        }
      }
      
      reject(new Error(`All ${maxRetries} attempts failed. Last error: ${lastError.message}`));
    });
  }
  
  static withCircuitBreaker(promiseFactory, failureThreshold = 5, resetTimeoutMs = 60000) {
    let failures = 0;
    let lastFailTime = null;
    let state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    
    return () => {
      return new Promise(async (resolve, reject) => {
        const now = Date.now();
        
        // Check if circuit should reset
        if (state === 'OPEN' && now - lastFailTime >= resetTimeoutMs) {
          state = 'HALF_OPEN';
          failures = 0;
        }
        
        // Reject immediately if circuit is open
        if (state === 'OPEN') {
          reject(new Error('Circuit breaker is OPEN'));
          return;
        }
        
        try {
          const result = await promiseFactory();
          
          // Reset on success
          if (state === 'HALF_OPEN') {
            state = 'CLOSED';
          }
          failures = 0;
          
          resolve(result);
        } catch (error) {
          failures++;
          lastFailTime = now;
          
          if (failures >= failureThreshold) {
            state = 'OPEN';
          }
          
          reject(error);
        }
      });
    };
  }
}

// Usage examples
const unreliableOperation = () => {
  return new Promise((resolve, reject) => {
    if (Math.random() > 0.7) {
      resolve('Success!');
    } else {
      reject(new Error('Random failure'));
    }
  });
};

// With retry
PromiseErrorHandler.withRetry(unreliableOperation, 3, 500)
  .then(result => console.log('Retry result:', result))
  .catch(error => console.error('Retry failed:', error));

// With circuit breaker
const protectedOperation = PromiseErrorHandler.withCircuitBreaker(unreliableOperation, 3, 5000);

// Multiple calls will trigger circuit breaker
for (let i = 0; i < 10; i++) {
  setTimeout(() => {
    protectedOperation()
      .then(result => console.log(`Call ${i + 1} result:`, result))
      .catch(error => console.error(`Call ${i + 1} error:`, error.message));
  }, i * 200);
}
```

## ⚡ Async/Await

### **🔧 Async/Await Basics**

```javascript
// ============= CONVERTING PROMISES TO ASYNC/AWAIT =============
// Promise-based version
function loadUserProfilePromises(userId) {
  return fetchUserData(userId)
    .then(user => {
      return Promise.all([
        Promise.resolve(user),
        fetchUserPosts(user.id),
        fetchUserComments(user.id)
      ]);
    })
    .then(([user, posts, comments]) => {
      return {
        ...user,
        posts,
        comments,
        totalPosts: posts.length,
        totalComments: comments.length
      };
    })
    .catch(error => {
      console.error('Error loading profile:', error);
      throw error;
    });
}

// Async/await version - much cleaner!
async function loadUserProfileAsync(userId) {
  try {
    const user = await fetchUserData(userId);
    
    // Parallel execution
    const [posts, comments] = await Promise.all([
      fetchUserPosts(user.id),
      fetchUserComments(user.id)
    ]);
    
    return {
      ...user,
      posts,
      comments,
      totalPosts: posts.length,
      totalComments: comments.length
    };
  } catch (error) {
    console.error('Error loading profile:', error);
    throw error;
  }
}

// ============= ASYNC/AWAIT PATTERNS =============
class AsyncDataProcessor {
  constructor() {
    this.cache = new Map();
    this.pendingRequests = new Map();
  }
  
  // Cached async function
  async getCachedData(key, fetcher, ttlMs = 300000) {
    const now = Date.now();
    const cached = this.cache.get(key);
    
    if (cached && now - cached.timestamp < ttlMs) {
      return cached.data;
    }
    
    // Prevent duplicate requests
    if (this.pendingRequests.has(key)) {
      return this.pendingRequests.get(key);
    }
    
    const promise = fetcher().then(data => {
      this.cache.set(key, { data, timestamp: now });
      this.pendingRequests.delete(key);
      return data;
    }).catch(error => {
      this.pendingRequests.delete(key);
      throw error;
    });
    
    this.pendingRequests.set(key, promise);
    return promise;
  }
  
  // Sequential processing
  async processSequentially(items, processor) {
    const results = [];
    
    for (const item of items) {
      try {
        const result = await processor(item);
        results.push({ success: true, data: result, item });
      } catch (error) {
        results.push({ success: false, error, item });
      }
    }
    
    return results;
  }
  
  // Parallel processing with concurrency limit
  async processWithConcurrency(items, processor, concurrency = 3) {
    const results = [];
    const executing = [];
    
    for (const item of items) {
      const promise = processor(item)
        .then(data => ({ success: true, data, item }))
        .catch(error => ({ success: false, error, item }));
      
      results.push(promise);
      
      if (results.length >= concurrency) {
        executing.push(promise);
        
        if (executing.length >= concurrency) {
          await Promise.race(executing);
          executing.splice(executing.findIndex(p => p.isFulfilled), 1);
        }
      }
    }
    
    return Promise.all(results);
  }
  
  // Async iteration
  async* asyncGenerator(items, processor) {
    for (const item of items) {
      try {
        const result = await processor(item);
        yield { success: true, data: result, item };
      } catch (error) {
        yield { success: false, error, item };
      }
    }
  }
  
  // Async pipeline
  async pipeline(...stages) {
    return async (input) => {
      let result = input;
      
      for (const stage of stages) {
        result = await stage(result);
      }
      
      return result;
    };
  }
}

// ============= REAL-WORLD ASYNC EXAMPLES =============
// File processing with async/await
class FileProcessor {
  async processFiles(files) {
    const results = [];
    
    for (const file of files) {
      try {
        console.log(`Processing ${file.name}...`);
        
        // Simulate file reading
        const content = await this.readFile(file);
        
        // Simulate processing
        const processed = await this.processContent(content);
        
        // Simulate saving
        const saved = await this.saveProcessed(processed, file.name);
        
        results.push({
          file: file.name,
          success: true,
          output: saved
        });
        
        console.log(`✓ Completed ${file.name}`);
      } catch (error) {
        console.error(`✗ Failed ${file.name}:`, error.message);
        results.push({
          file: file.name,
          success: false,
          error: error.message
        });
      }
    }
    
    return results;
  }
  
  async readFile(file) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (file.size > 1000000) {
          reject(new Error('File too large'));
        } else {
          resolve(`Content of ${file.name}`);
        }
      }, Math.random() * 1000);
    });
  }
  
  async processContent(content) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(content.toUpperCase());
      }, Math.random() * 500);
    });
  }
  
  async saveProcessed(content, filename) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (Math.random() > 0.1) {
          resolve(`processed_${filename}`);
        } else {
          reject(new Error('Save failed'));
        }
      }, Math.random() * 300);
    });
  }
}

// API client with async/await
class ApiClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.authToken = null;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const headers = {
      'Content-Type': 'application/json',
      ...options.headers
    };
    
    if (this.authToken) {
      headers.Authorization = `Bearer ${this.authToken}`;
    }
    
    try {
      const response = await fetch(url, {
        ...options,
        headers
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const data = await response.json();
      return data;
    } catch (error) {
      console.error(`API request failed: ${endpoint}`, error);
      throw error;
    }
  }
  
  async get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url);
  }
  
  async post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
  
  async put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
  
  async delete(endpoint) {
    return this.request(endpoint, {
      method: 'DELETE'
    });
  }
  
  async authenticate(credentials) {
    try {
      const response = await this.post('/auth/login', credentials);
      this.authToken = response.token;
      return response;
    } catch (error) {
      console.error('Authentication failed:', error);
      throw error;
    }
  }
  
  async refreshToken() {
    try {
      const response = await this.post('/auth/refresh');
      this.authToken = response.token;
      return response;
    } catch (error) {
      console.error('Token refresh failed:', error);
      this.authToken = null;
      throw error;
    }
  }
}

// ============= ASYNC ERROR HANDLING =============
class AsyncErrorHandler {
  static async safeExecute(asyncFn, fallback = null) {
    try {
      return await asyncFn();
    } catch (error) {
      console.error('Async operation failed:', error);
      return fallback;
    }
  }
  
  static async withTimeout(promise, timeoutMs) {
    const timeout = new Promise((_, reject) => {
      setTimeout(() => {
        reject(new Error(`Operation timed out after ${timeoutMs}ms`));
      }, timeoutMs);
    });
    
    return Promise.race([promise, timeout]);
  }
  
  static async retry(asyncFn, maxAttempts = 3, delayMs = 1000) {
    let lastError;
    
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await asyncFn();
      } catch (error) {
        lastError = error;
        console.log(`Attempt ${attempt} failed:`, error.message);
        
        if (attempt < maxAttempts) {
          await new Promise(resolve => setTimeout(resolve, delayMs * attempt));
        }
      }
    }
    
    throw lastError;
  }
  
  static async parallel(asyncFunctions, { failFast = false, maxConcurrency = Infinity } = {}) {
    if (maxConcurrency === Infinity) {
      return failFast ? Promise.all(asyncFunctions) : Promise.allSettled(asyncFunctions);
    }
    
    const results = [];
    const executing = [];
    
    for (let i = 0; i < asyncFunctions.length; i++) {
      const promise = asyncFunctions[i]();
      
      if (executing.length >= maxConcurrency) {
        await Promise.race(executing);
        executing.splice(0, 1);
      }
      
      executing.push(promise);
      results.push(promise);
    }
    
    return failFast ? Promise.all(results) : Promise.allSettled(results);
  }
}

// Usage examples
const processor = new AsyncDataProcessor();
const fileProcessor = new FileProcessor();
const apiClient = new ApiClient('https://api.example.com');

// Example usage
(async () => {
  try {
    // Cached data fetching
    const userData = await processor.getCachedData(
      'user-123',
      () => fetchUserData(123)
    );
    
    // File processing
    const files = [
      { name: 'file1.txt', size: 500 },
      { name: 'file2.txt', size: 1000 },
      { name: 'file3.txt', size: 2000000 } // This will fail
    ];
    
    const fileResults = await fileProcessor.processFiles(files);
    console.log('File processing results:', fileResults);
    
    // API operations with error handling
    const safeApiCall = await AsyncErrorHandler.safeExecute(
      () => apiClient.get('/users'),
      [] // fallback value
    );
    
    console.log('API result:', safeApiCall);
    
  } catch (error) {
    console.error('Main async operation failed:', error);
  }
})();
```

## 🚀 Next.js Applications

### **📊 Data Fetching Patterns**

```jsx
// hooks/useAsyncData.js
import { useState, useEffect, useCallback, useRef } from 'react';

export const useAsyncData = (asyncFunction, dependencies = [], options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [lastFetch, setLastFetch] = useState(null);
  
  const abortControllerRef = useRef();
  const cacheRef = useRef(new Map());
  
  const {
    cacheKey,
    cacheTtl = 300000, // 5 minutes
    retries = 3,
    retryDelay = 1000,
    timeout = 10000
  } = options;
  
  const fetchData = useCallback(async (force = false) => {
    // Check cache first
    if (cacheKey && !force) {
      const cached = cacheRef.current.get(cacheKey);
      if (cached && Date.now() - cached.timestamp < cacheTtl) {
        setData(cached.data);
        setLastFetch(new Date(cached.timestamp));
        return cached.data;
      }
    }
    
    // Cancel previous request
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    abortControllerRef.current = new AbortController();
    const signal = abortControllerRef.current.signal;
    
    setLoading(true);
    setError(null);
    
    let attempt = 1;
    
    while (attempt <= retries) {
      try {
        // Add timeout wrapper
        const timeoutPromise = new Promise((_, reject) => {
          setTimeout(() => reject(new Error('Request timeout')), timeout);
        });
        
        const dataPromise = asyncFunction(signal);
        const result = await Promise.race([dataPromise, timeoutPromise]);
        
        // Cache the result
        if (cacheKey) {
          cacheRef.current.set(cacheKey, {
            data: result,
            timestamp: Date.now()
          });
        }
        
        setData(result);
        setLastFetch(new Date());
        setLoading(false);
        
        return result;
        
      } catch (err) {
        if (err.name === 'AbortError') {
          break; // Don't retry if aborted
        }
        
        if (attempt === retries) {
          setError(err);
          setLoading(false);
          throw err;
        }
        
        // Wait before retry
        await new Promise(resolve => 
          setTimeout(resolve, retryDelay * attempt)
        );
        attempt++;
      }
    }
  }, [asyncFunction, cacheKey, cacheTtl, retries, retryDelay, timeout, ...dependencies]);
  
  const refetch = useCallback(() => {
    return fetchData(true);
  }, [fetchData]);
  
  const clearCache = useCallback(() => {
    if (cacheKey) {
      cacheRef.current.delete(cacheKey);
    }
  }, [cacheKey]);
  
  useEffect(() => {
    fetchData();
    
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
    lastFetch,
    refetch,
    clearCache
  };
};

// hooks/useAsyncOperation.js
export const useAsyncOperation = () => {
  const [operations, setOperations] = useState(new Map());
  
  const execute = useCallback(async (operationId, asyncFn, options = {}) => {
    const { 
      onSuccess, 
      onError, 
      onStart, 
      onFinally,
      timeout = 30000 
    } = options;
    
    // Update operation state
    setOperations(prev => new Map(prev).set(operationId, {
      loading: true,
      error: null,
      data: null,
      startTime: Date.now()
    }));
    
    if (onStart) onStart();
    
    try {
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Operation timeout')), timeout);
      });
      
      const result = await Promise.race([asyncFn(), timeoutPromise]);
      
      setOperations(prev => new Map(prev).set(operationId, {
        loading: false,
        error: null,
        data: result,
        startTime: prev.get(operationId)?.startTime,
        endTime: Date.now()
      }));
      
      if (onSuccess) onSuccess(result);
      return result;
      
    } catch (error) {
      setOperations(prev => new Map(prev).set(operationId, {
        loading: false,
        error,
        data: null,
        startTime: prev.get(operationId)?.startTime,
        endTime: Date.now()
      }));
      
      if (onError) onError(error);
      throw error;
      
    } finally {
      if (onFinally) onFinally();
    }
  }, []);
  
  const getOperation = useCallback((operationId) => {
    return operations.get(operationId) || {
      loading: false,
      error: null,
      data: null
    };
  }, [operations]);
  
  const clearOperation = useCallback((operationId) => {
    setOperations(prev => {
      const next = new Map(prev);
      next.delete(operationId);
      return next;
    });
  }, []);
  
  return {
    execute,
    getOperation,
    clearOperation,
    operations: Object.fromEntries(operations)
  };
};

// components/AsyncDataComponent.jsx
import { useAsyncData, useAsyncOperation } from '../hooks/useAsyncData';

const AsyncDataComponent = ({ userId }) => {
  const [selectedTab, setSelectedTab] = useState('profile');
  const { execute, getOperation } = useAsyncOperation();
  
  // User profile data
  const {
    data: profile,
    loading: profileLoading,
    error: profileError,
    refetch: refetchProfile
  } = useAsyncData(
    async (signal) => {
      const response = await fetch(`/api/users/${userId}`, { signal });
      if (!response.ok) throw new Error('Failed to fetch profile');
      return response.json();
    },
    [userId],
    {
      cacheKey: `user-${userId}`,
      cacheTtl: 300000,
      retries: 3
    }
  );
  
  // User posts data
  const {
    data: posts,
    loading: postsLoading,
    error: postsError,
    refetch: refetchPosts
  } = useAsyncData(
    async (signal) => {
      const response = await fetch(`/api/users/${userId}/posts`, { signal });
      if (!response.ok) throw new Error('Failed to fetch posts');
      return response.json();
    },
    [userId],
    {
      cacheKey: `user-posts-${userId}`,
      cacheTtl: 180000 // 3 minutes
    }
  );
  
  // Update profile operation
  const handleUpdateProfile = async (updates) => {
    try {
      await execute('updateProfile', async () => {
        const response = await fetch(`/api/users/${userId}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(updates)
        });
        
        if (!response.ok) {
          throw new Error('Failed to update profile');
        }
        
        return response.json();
      }, {
        onSuccess: () => {
          refetchProfile();
          // Show success message
        },
        onError: (error) => {
          // Show error message
          console.error('Update failed:', error);
        }
      });
    } catch (error) {
      // Error is already handled by the operation
    }
  };
  
  // Delete post operation
  const handleDeletePost = async (postId) => {
    if (!confirm('Are you sure you want to delete this post?')) {
      return;
    }
    
    try {
      await execute(`deletePost-${postId}`, async () => {
        const response = await fetch(`/api/posts/${postId}`, {
          method: 'DELETE'
        });
        
        if (!response.ok) {
          throw new Error('Failed to delete post');
        }
      }, {
        onSuccess: () => {
          refetchPosts();
          // Show success message
        }
      });
    } catch (error) {
      // Error handling
    }
  };
  
  const updateOperation = getOperation('updateProfile');
  
  if (profileLoading) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>Loading profile...</p>
      </div>
    );
  }
  
  if (profileError) {
    return (
      <div className="error-container">
        <h3>Error Loading Profile</h3>
        <p>{profileError.message}</p>
        <button onClick={refetchProfile}>Retry</button>
      </div>
    );
  }
  
  return (
    <div className="async-data-component">
      <div className="profile-header">
        <img src={profile?.avatar} alt={profile?.name} />
        <div className="profile-info">
          <h1>{profile?.name}</h1>
          <p>{profile?.email}</p>
          
          <button 
            onClick={() => handleUpdateProfile({ lastActive: new Date() })}
            disabled={updateOperation.loading}
          >
            {updateOperation.loading ? 'Updating...' : 'Update Last Active'}
          </button>
          
          {updateOperation.error && (
            <div className="error">
              Update failed: {updateOperation.error.message}
            </div>
          )}
        </div>
      </div>
      
      <div className="tabs">
        <button 
          className={selectedTab === 'profile' ? 'active' : ''}
          onClick={() => setSelectedTab('profile')}
        >
          Profile
        </button>
        <button 
          className={selectedTab === 'posts' ? 'active' : ''}
          onClick={() => setSelectedTab('posts')}
        >
          Posts {postsLoading && '(Loading...)'}
        </button>
      </div>
      
      <div className="tab-content">
        {selectedTab === 'profile' && (
          <div className="profile-details">
            <h2>Profile Details</h2>
            <pre>{JSON.stringify(profile, null, 2)}</pre>
          </div>
        )}
        
        {selectedTab === 'posts' && (
          <div className="posts-section">
            <div className="posts-header">
              <h2>Posts</h2>
              <button onClick={refetchPosts}>Refresh</button>
            </div>
            
            {postsError ? (
              <div className="error">
                Error loading posts: {postsError.message}
              </div>
            ) : posts ? (
              <div className="posts-list">
                {posts.map(post => (
                  <div key={post.id} className="post-item">
                    <h3>{post.title}</h3>
                    <p>{post.excerpt}</p>
                    <div className="post-actions">
                      <button 
                        onClick={() => handleDeletePost(post.id)}
                        disabled={getOperation(`deletePost-${post.id}`).loading}
                      >
                        {getOperation(`deletePost-${post.id}`).loading ? 'Deleting...' : 'Delete'}
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            ) : (
              <div className="loading">Loading posts...</div>
            )}
          </div>
        )}
      </div>
    </div>
  );
};

export default AsyncDataComponent;
```

### **🔄 Server-Side Async Patterns**

```javascript
// pages/api/users/[id].js
import { AsyncErrorHandler } from '../../../utils/asyncErrorHandler';
import { DatabaseClient } from '../../../utils/database';
import { CacheManager } from '../../../utils/cache';

const db = new DatabaseClient();
const cache = new CacheManager();

export default async function handler(req, res) {
  const { id } = req.query;
  
  try {
    switch (req.method) {
      case 'GET':
        await handleGetUser(req, res, id);
        break;
      case 'PUT':
        await handleUpdateUser(req, res, id);
        break;
      case 'DELETE':
        await handleDeleteUser(req, res, id);
        break;
      default:
        res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
        res.status(405).json({ error: 'Method not allowed' });
    }
  } catch (error) {
    console.error('API error:', error);
    res.status(500).json({ 
      error: 'Internal server error',
      message: process.env.NODE_ENV === 'development' ? error.message : undefined
    });
  }
}

async function handleGetUser(req, res, userId) {
  const cacheKey = `user:${userId}`;
  
  // Try cache first
  const cached = await cache.get(cacheKey);
  if (cached) {
    return res.status(200).json(cached);
  }
  
  // Fetch from database with timeout
  const user = await AsyncErrorHandler.withTimeout(
    db.users.findById(userId),
    5000
  );
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  // Parallel fetch related data
  const [posts, comments, followers] = await Promise.allSettled([
    db.posts.findByUserId(userId),
    db.comments.findByUserId(userId),
    db.follows.getFollowers(userId)
  ]);
  
  const userData = {
    ...user,
    postsCount: posts.status === 'fulfilled' ? posts.value.length : 0,
    commentsCount: comments.status === 'fulfilled' ? comments.value.length : 0,
    followersCount: followers.status === 'fulfilled' ? followers.value.length : 0
  };
  
  // Cache the result
  await cache.set(cacheKey, userData, 300); // 5 minutes
  
  res.status(200).json(userData);
}

async function handleUpdateUser(req, res, userId) {
  const updates = req.body;
  
  // Validate updates
  const validatedUpdates = await validateUserUpdates(updates);
  
  // Update with retry
  const updatedUser = await AsyncErrorHandler.retry(
    () => db.users.update(userId, validatedUpdates),
    3,
    1000
  );
  
  if (!updatedUser) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  // Invalidate cache
  await cache.delete(`user:${userId}`);
  
  // Send notifications asynchronously (don't wait)
  sendUserUpdateNotification(userId, updates).catch(error => {
    console.error('Notification failed:', error);
  });
  
  res.status(200).json(updatedUser);
}

async function handleDeleteUser(req, res, userId) {
  // Start transaction
  const transaction = await db.beginTransaction();
  
  try {
    // Delete related data first
    await Promise.all([
      db.posts.deleteByUserId(userId, transaction),
      db.comments.deleteByUserId(userId, transaction),
      db.follows.deleteByUserId(userId, transaction)
    ]);
    
    // Delete user
    await db.users.delete(userId, transaction);
    
    // Commit transaction
    await db.commitTransaction(transaction);
    
    // Clean up cache
    await cache.deletePattern(`user:${userId}*`);
    
    res.status(204).end();
    
  } catch (error) {
    await db.rollbackTransaction(transaction);
    throw error;
  }
}

// utils/batchProcessor.js
export class BatchProcessor {
  constructor(batchSize = 10, delayMs = 100) {
    this.batchSize = batchSize;
    this.delayMs = delayMs;
    this.queue = [];
    this.processing = false;
  }
  
  async add(item) {
    return new Promise((resolve, reject) => {
      this.queue.push({ item, resolve, reject });
      this.process();
    });
  }
  
  async process() {
    if (this.processing || this.queue.length === 0) {
      return;
    }
    
    this.processing = true;
    
    while (this.queue.length > 0) {
      const batch = this.queue.splice(0, this.batchSize);
      
      try {
        const results = await this.processBatch(batch.map(b => b.item));
        
        batch.forEach((b, index) => {
          b.resolve(results[index]);
        });
        
      } catch (error) {
        batch.forEach(b => {
          b.reject(error);
        });
      }
      
      if (this.queue.length > 0) {
        await new Promise(resolve => setTimeout(resolve, this.delayMs));
      }
    }
    
    this.processing = false;
  }
  
  async processBatch(items) {
    // Override this method in subclasses
    throw new Error('processBatch must be implemented');
  }
}

// Specific batch processor for emails
export class EmailBatchProcessor extends BatchProcessor {
  constructor(emailService, batchSize = 50) {
    super(batchSize, 1000); // 1 second delay between batches
    this.emailService = emailService;
  }
  
  async processBatch(emails) {
    return this.emailService.sendBatch(emails);
  }
}

// pages/api/notifications/send.js
const emailProcessor = new EmailBatchProcessor(emailService);

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { recipients, subject, message } = req.body;
  
  try {
    // Process emails in batches
    const emailPromises = recipients.map(recipient => 
      emailProcessor.add({
        to: recipient.email,
        subject,
        message,
        userId: recipient.id
      })
    );
    
    const results = await Promise.allSettled(emailPromises);
    
    const successful = results.filter(r => r.status === 'fulfilled').length;
    const failed = results.filter(r => r.status === 'rejected').length;
    
    res.status(200).json({
      sent: successful,
      failed,
      total: recipients.length
    });
    
  } catch (error) {
    console.error('Batch email error:', error);
    res.status(500).json({ error: 'Failed to send emails' });
  }
}
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Promise Patterns**
สร้างฟังก์ชันที่ใช้ Promises:

```javascript
// TODO: สร้างฟังก์ชันเหล่านี้

// 1. สร้าง Promise-based cache
class PromiseCache {
  // Implement caching with TTL support
  // Methods: get, set, delete, clear
}

// 2. สร้าง rate limiter
class RateLimiter {
  // Implement rate limiting using promises
  // Should delay execution if rate exceeded
}

// 3. สร้าง promise pool
class PromisePool {
  // Execute promises with concurrency limit
  // Queue additional promises when pool is full
}

// 4. สร้าง data pipeline
function createPipeline(...stages) {
  // Chain multiple async operations
  // Each stage receives output from previous stage
}
```

### **⚡ แบบฝึกหัดที่ 2: Async/Await Applications**
สร้าง async functions สำหรับ real-world scenarios:

```javascript
// TODO: สร้าง async functions

// 1. File upload with progress
async function uploadFileWithProgress(file, onProgress) {
  // Upload file in chunks
  // Report progress via callback
}

// 2. Data synchronization
async function syncData(localData, remoteEndpoint) {
  // Compare local and remote data
  // Upload changes and download updates
}

// 3. Background task manager
class TaskManager {
  // Schedule and execute background tasks
  // Support task priorities and dependencies
}

// 4. Real-time data processor
async function processRealtimeData(dataStream) {
  // Process streaming data
  // Handle backpressure and errors
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง async components และ patterns:

```jsx
// TODO: สร้าง components ที่ใช้ async patterns

// 1. InfiniteScroll component
const InfiniteScroll = ({ fetchMore, hasMore }) => {
  // Implement infinite scrolling with async data loading
  // Handle loading states and errors
};

// 2. AutoSave form
const AutoSaveForm = ({ initialData, saveFunction }) => {
  // Auto-save form data with debouncing
  // Show save status and handle conflicts
};

// 3. RealTimeChat component
const RealTimeChat = ({ roomId }) => {
  // Real-time chat with WebSocket
  // Handle message sending and receiving
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 8: Objects & Classes](./08-objects-classes.md)

**บทถัดไป**: [บทที่ 10: ES6 Modules](./10-modules.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN: async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [JavaScript.info: Async/await](https://javascript.info/async-await)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Promises** และ asynchronous programming patterns
- **Async/await** syntax และ error handling
- **Promise combinators** และ advanced patterns
- **การใช้งานใน Next.js** สำหรับ data fetching
- **Performance optimization** สำหรับ async operations

Promises และ async/await เป็นพื้นฐานสำคัญของ modern JavaScript ที่ช่วยให้การจัดการ asynchronous code เป็นไปอย่างมีประสิทธิภาพ! 🚀
