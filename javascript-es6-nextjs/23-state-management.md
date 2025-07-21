# บทที่ 23: State Management

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ advanced state management patterns และ strategies
- ทำความเข้าใจ Redux Toolkit, Zustand, และ modern state solutions
- เรียนรู้ state synchronization และ data flow optimization
- สร้าง scalable state architecture สำหรับ complex applications

## 🏪 Advanced Redux Patterns

### **⚙️ Redux Toolkit Advanced**

```javascript
// ============= ADVANCED REDUX TOOLKIT =============
// store/slices/user-slice.js
import { createSlice, createAsyncThunk, createSelector } from '@reduxjs/toolkit';
import { userAPI } from '@/lib/api/user-api';

// Async thunks with advanced error handling
export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId, { rejectWithValue, getState, signal }) => {
    try {
      // Check if already loading
      const state = getState();
      if (state.user.loading) {
        return rejectWithValue('Already loading');
      }
      
      const response = await userAPI.getUser(userId, { signal });
      return response.data;
    } catch (error) {
      if (error.name === 'AbortError') {
        return rejectWithValue('Request cancelled');
      }
      return rejectWithValue(error.message);
    }
  }
);

export const updateUser = createAsyncThunk(
  'user/updateUser',
  async ({ userId, data }, { rejectWithValue, dispatch }) => {
    try {
      const response = await userAPI.updateUser(userId, data);
      
      // Optimistic update for immediate UI feedback
      dispatch(userSlice.actions.updateUserOptimistic(data));
      
      return response.data;
    } catch (error) {
      // Revert optimistic update on error
      dispatch(userSlice.actions.revertOptimisticUpdate());
      return rejectWithValue(error.message);
    }
  }
);

export const fetchUsers = createAsyncThunk(
  'user/fetchUsers',
  async (params, { rejectWithValue }) => {
    try {
      const response = await userAPI.getUsers(params);
      return {
        users: response.data,
        total: response.total,
        page: params.page || 1
      };
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    // Current user
    currentUser: null,
    
    // Users list with pagination
    users: [],
    usersById: {},
    pagination: {
      page: 1,
      limit: 10,
      total: 0,
      hasNext: false,
      hasPrev: false
    },
    
    // Loading states
    loading: false,
    listLoading: false,
    updating: false,
    
    // Error states
    error: null,
    listError: null,
    
    // UI states
    selectedUsers: [],
    filters: {
      search: '',
      status: 'all',
      role: 'all'
    },
    
    // Cache management
    lastFetch: null,
    cacheExpiry: 5 * 60 * 1000, // 5 minutes
    
    // Optimistic updates
    optimisticUpdates: {}
  },
  
  reducers: {
    // UI actions
    setFilters: (state, action) => {
      state.filters = { ...state.filters, ...action.payload };
      state.pagination.page = 1; // Reset pagination
    },
    
    clearFilters: (state) => {
      state.filters = {
        search: '',
        status: 'all',
        role: 'all'
      };
      state.pagination.page = 1;
    },
    
    selectUser: (state, action) => {
      const userId = action.payload;
      if (!state.selectedUsers.includes(userId)) {
        state.selectedUsers.push(userId);
      }
    },
    
    deselectUser: (state, action) => {
      const userId = action.payload;
      state.selectedUsers = state.selectedUsers.filter(id => id !== userId);
    },
    
    selectAllUsers: (state) => {
      state.selectedUsers = state.users.map(user => user.id);
    },
    
    clearSelection: (state) => {
      state.selectedUsers = [];
    },
    
    // Optimistic updates
    updateUserOptimistic: (state, action) => {
      const { id, ...updates } = action.payload;
      
      // Store original data for potential revert
      if (state.usersById[id]) {
        state.optimisticUpdates[id] = { ...state.usersById[id] };
        
        // Apply optimistic update
        state.usersById[id] = { ...state.usersById[id], ...updates };
        
        // Update in users array
        const userIndex = state.users.findIndex(user => user.id === id);
        if (userIndex !== -1) {
          state.users[userIndex] = state.usersById[id];
        }
        
        // Update current user if it's the same
        if (state.currentUser?.id === id) {
          state.currentUser = { ...state.currentUser, ...updates };
        }
      }
    },
    
    revertOptimisticUpdate: (state, action) => {
      const userId = action.payload;
      if (state.optimisticUpdates[userId]) {
        const original = state.optimisticUpdates[userId];
        
        // Revert in usersById
        state.usersById[userId] = original;
        
        // Revert in users array
        const userIndex = state.users.findIndex(user => user.id === userId);
        if (userIndex !== -1) {
          state.users[userIndex] = original;
        }
        
        // Revert current user
        if (state.currentUser?.id === userId) {
          state.currentUser = original;
        }
        
        // Clean up optimistic update
        delete state.optimisticUpdates[userId];
      }
    },
    
    // Cache management
    invalidateCache: (state) => {
      state.lastFetch = null;
    },
    
    // Real-time updates
    userUpdated: (state, action) => {
      const user = action.payload;
      state.usersById[user.id] = user;
      
      const userIndex = state.users.findIndex(u => u.id === user.id);
      if (userIndex !== -1) {
        state.users[userIndex] = user;
      }
      
      if (state.currentUser?.id === user.id) {
        state.currentUser = user;
      }
    },
    
    userDeleted: (state, action) => {
      const userId = action.payload;
      delete state.usersById[userId];
      state.users = state.users.filter(user => user.id !== userId);
      state.selectedUsers = state.selectedUsers.filter(id => id !== userId);
      
      if (state.currentUser?.id === userId) {
        state.currentUser = null;
      }
    }
  },
  
  extraReducers: (builder) => {
    builder
      // Fetch user
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.currentUser = action.payload;
        state.usersById[action.payload.id] = action.payload;
        state.lastFetch = Date.now();
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      
      // Fetch users
      .addCase(fetchUsers.pending, (state) => {
        state.listLoading = true;
        state.listError = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.listLoading = false;
        state.users = action.payload.users;
        
        // Update usersById for easy lookup
        action.payload.users.forEach(user => {
          state.usersById[user.id] = user;
        });
        
        // Update pagination
        state.pagination = {
          ...state.pagination,
          page: action.payload.page,
          total: action.payload.total,
          hasNext: action.payload.page * state.pagination.limit < action.payload.total,
          hasPrev: action.payload.page > 1
        };
        
        state.lastFetch = Date.now();
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.listLoading = false;
        state.listError = action.payload;
      })
      
      // Update user
      .addCase(updateUser.pending, (state) => {
        state.updating = true;
      })
      .addCase(updateUser.fulfilled, (state, action) => {
        state.updating = false;
        const user = action.payload;
        
        // Clear optimistic update
        delete state.optimisticUpdates[user.id];
        
        // Apply final update
        state.usersById[user.id] = user;
        
        const userIndex = state.users.findIndex(u => u.id === user.id);
        if (userIndex !== -1) {
          state.users[userIndex] = user;
        }
        
        if (state.currentUser?.id === user.id) {
          state.currentUser = user;
        }
      })
      .addCase(updateUser.rejected, (state) => {
        state.updating = false;
      });
  }
});

// Export actions
export const {
  setFilters,
  clearFilters,
  selectUser,
  deselectUser,
  selectAllUsers,
  clearSelection,
  updateUserOptimistic,
  revertOptimisticUpdate,
  invalidateCache,
  userUpdated,
  userDeleted
} = userSlice.actions;

// Selectors with memoization
export const selectCurrentUser = (state) => state.user.currentUser;
export const selectUsers = (state) => state.user.users;
export const selectUsersById = (state) => state.user.usersById;
export const selectSelectedUsers = (state) => state.user.selectedUsers;
export const selectFilters = (state) => state.user.filters;
export const selectPagination = (state) => state.user.pagination;

// Memoized selectors
export const selectFilteredUsers = createSelector(
  [selectUsers, selectFilters],
  (users, filters) => {
    return users.filter(user => {
      const matchesSearch = !filters.search || 
        user.name.toLowerCase().includes(filters.search.toLowerCase()) ||
        user.email.toLowerCase().includes(filters.search.toLowerCase());
      
      const matchesStatus = filters.status === 'all' || user.status === filters.status;
      const matchesRole = filters.role === 'all' || user.role === filters.role;
      
      return matchesSearch && matchesStatus && matchesRole;
    });
  }
);

export const selectSelectedUsersData = createSelector(
  [selectSelectedUsers, selectUsersById],
  (selectedIds, usersById) => {
    return selectedIds.map(id => usersById[id]).filter(Boolean);
  }
);

export const selectUserStats = createSelector(
  [selectUsers],
  (users) => {
    const total = users.length;
    const active = users.filter(user => user.status === 'active').length;
    const inactive = total - active;
    
    const roleStats = users.reduce((acc, user) => {
      acc[user.role] = (acc[user.role] || 0) + 1;
      return acc;
    }, {});
    
    return {
      total,
      active,
      inactive,
      roleStats
    };
  }
);

// Cache validation selector
export const selectIsCacheValid = createSelector(
  [state => state.user.lastFetch, state => state.user.cacheExpiry],
  (lastFetch, cacheExpiry) => {
    if (!lastFetch) return false;
    return Date.now() - lastFetch < cacheExpiry;
  }
);

export default userSlice.reducer;

// ============= MIDDLEWARE =============
// store/middleware/api-middleware.js
import { createListenerMiddleware } from '@reduxjs/toolkit';
import { fetchUsers, setFilters } from '../slices/user-slice';

export const apiMiddleware = createListenerMiddleware();

// Auto-fetch when filters change
apiMiddleware.startListening({
  actionCreator: setFilters,
  effect: async (action, listenerApi) => {
    // Debounce the API call
    listenerApi.cancelActiveListeners();
    
    await listenerApi.delay(500);
    
    const state = listenerApi.getState();
    const { filters, pagination } = state.user;
    
    listenerApi.dispatch(fetchUsers({
      ...filters,
      page: pagination.page,
      limit: pagination.limit
    }));
  }
});

// ============= REAL-TIME MIDDLEWARE =============
// store/middleware/realtime-middleware.js
export const realtimeMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // Handle real-time events
  if (action.type === 'websocket/message') {
    const { type, payload } = action.payload;
    
    switch (type) {
      case 'user_updated':
        store.dispatch(userUpdated(payload));
        break;
      case 'user_deleted':
        store.dispatch(userDeleted(payload.id));
        break;
    }
  }
  
  return result;
};
```

### **🐻 Zustand Advanced Store**

```javascript
// ============= ZUSTAND ADVANCED STORE =============
// store/use-app-store.js
import { create } from 'zustand';
import { devtools, persist, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// Store slices
const createUserSlice = (set, get) => ({
  users: [],
  currentUser: null,
  loading: false,
  error: null,
  
  // Actions
  setUsers: (users) => set((state) => {
    state.users = users;
  }),
  
  addUser: (user) => set((state) => {
    state.users.push(user);
  }),
  
  updateUser: (id, updates) => set((state) => {
    const index = state.users.findIndex(user => user.id === id);
    if (index !== -1) {
      Object.assign(state.users[index], updates);
    }
    if (state.currentUser?.id === id) {
      Object.assign(state.currentUser, updates);
    }
  }),
  
  removeUser: (id) => set((state) => {
    state.users = state.users.filter(user => user.id !== id);
    if (state.currentUser?.id === id) {
      state.currentUser = null;
    }
  }),
  
  setCurrentUser: (user) => set((state) => {
    state.currentUser = user;
  }),
  
  setLoading: (loading) => set((state) => {
    state.loading = loading;
  }),
  
  setError: (error) => set((state) => {
    state.error = error;
  })
});

const createUISlice = (set, get) => ({
  theme: 'light',
  sidebar: {
    isOpen: true,
    width: 280
  },
  modals: {},
  notifications: [],
  
  // Actions
  setTheme: (theme) => set((state) => {
    state.theme = theme;
  }),
  
  toggleSidebar: () => set((state) => {
    state.sidebar.isOpen = !state.sidebar.isOpen;
  }),
  
  setSidebarWidth: (width) => set((state) => {
    state.sidebar.width = width;
  }),
  
  openModal: (id, props = {}) => set((state) => {
    state.modals[id] = { isOpen: true, props };
  }),
  
  closeModal: (id) => set((state) => {
    if (state.modals[id]) {
      state.modals[id].isOpen = false;
    }
  }),
  
  addNotification: (notification) => set((state) => {
    const id = Date.now().toString();
    state.notifications.push({
      id,
      ...notification,
      timestamp: new Date()
    });
  }),
  
  removeNotification: (id) => set((state) => {
    state.notifications = state.notifications.filter(n => n.id !== id);
  }),
  
  clearNotifications: () => set((state) => {
    state.notifications = [];
  })
});

const createDataSlice = (set, get) => ({
  cache: {},
  pendingRequests: new Set(),
  
  // Cache management
  setCache: (key, data, ttl = 300000) => set((state) => {
    state.cache[key] = {
      data,
      timestamp: Date.now(),
      ttl
    };
  }),
  
  getCache: (key) => {
    const cached = get().cache[key];
    if (!cached) return null;
    
    if (Date.now() - cached.timestamp > cached.ttl) {
      set((state) => {
        delete state.cache[key];
      });
      return null;
    }
    
    return cached.data;
  },
  
  invalidateCache: (pattern) => set((state) => {
    if (typeof pattern === 'string') {
      delete state.cache[pattern];
    } else if (pattern instanceof RegExp) {
      Object.keys(state.cache).forEach(key => {
        if (pattern.test(key)) {
          delete state.cache[key];
        }
      });
    }
  }),
  
  clearCache: () => set((state) => {
    state.cache = {};
  }),
  
  // Request deduplication
  addPendingRequest: (key) => set((state) => {
    state.pendingRequests.add(key);
  }),
  
  removePendingRequest: (key) => set((state) => {
    state.pendingRequests.delete(key);
  }),
  
  isPending: (key) => get().pendingRequests.has(key)
});

// Combined store with middleware
export const useAppStore = create(
  devtools(
    persist(
      subscribeWithSelector(
        immer((...args) => ({
          // Combine all slices
          ...createUserSlice(...args),
          ...createUISlice(...args),
          ...createDataSlice(...args),
          
          // Global actions
          reset: () => args[0]((state) => {
            // Reset to initial state
            Object.assign(state, {
              users: [],
              currentUser: null,
              loading: false,
              error: null,
              notifications: [],
              cache: {},
              pendingRequests: new Set()
            });
          })
        }))
      ),
      {
        name: 'app-store',
        partialize: (state) => ({
          theme: state.theme,
          sidebar: state.sidebar,
          currentUser: state.currentUser
        })
      }
    ),
    { name: 'App Store' }
  )
);

// Computed values (selectors)
export const useUserStats = () => useAppStore((state) => {
  const users = state.users;
  return {
    total: users.length,
    active: users.filter(u => u.status === 'active').length,
    inactive: users.filter(u => u.status === 'inactive').length
  };
});

export const useFilteredUsers = (filters) => useAppStore((state) => {
  return state.users.filter(user => {
    if (filters.search) {
      const search = filters.search.toLowerCase();
      if (!user.name.toLowerCase().includes(search) && 
          !user.email.toLowerCase().includes(search)) {
        return false;
      }
    }
    
    if (filters.status && filters.status !== 'all') {
      if (user.status !== filters.status) return false;
    }
    
    return true;
  });
});

// ============= ASYNC ACTIONS =============
// store/actions/user-actions.js
import { useAppStore } from '../use-app-store';
import { userAPI } from '@/lib/api/user-api';

export const userActions = {
  async fetchUsers(params = {}) {
    const { setUsers, setLoading, setError, getCache, setCache, isPending, addPendingRequest, removePendingRequest } = useAppStore.getState();
    
    const cacheKey = `users:${JSON.stringify(params)}`;
    
    // Check cache first
    const cached = getCache(cacheKey);
    if (cached) {
      setUsers(cached);
      return cached;
    }
    
    // Check if already loading
    if (isPending(cacheKey)) {
      return null;
    }
    
    try {
      setLoading(true);
      setError(null);
      addPendingRequest(cacheKey);
      
      const response = await userAPI.getUsers(params);
      const users = response.data;
      
      setUsers(users);
      setCache(cacheKey, users);
      
      return users;
    } catch (error) {
      setError(error.message);
      throw error;
    } finally {
      setLoading(false);
      removePendingRequest(cacheKey);
    }
  },
  
  async createUser(userData) {
    const { addUser, setError, addNotification } = useAppStore.getState();
    
    try {
      const response = await userAPI.createUser(userData);
      const user = response.data;
      
      addUser(user);
      addNotification({
        type: 'success',
        message: 'User created successfully'
      });
      
      return user;
    } catch (error) {
      setError(error.message);
      addNotification({
        type: 'error',
        message: 'Failed to create user'
      });
      throw error;
    }
  },
  
  async updateUser(id, updates) {
    const { updateUser, setError, addNotification } = useAppStore.getState();
    
    try {
      const response = await userAPI.updateUser(id, updates);
      const user = response.data;
      
      updateUser(id, user);
      addNotification({
        type: 'success',
        message: 'User updated successfully'
      });
      
      return user;
    } catch (error) {
      setError(error.message);
      addNotification({
        type: 'error',
        message: 'Failed to update user'
      });
      throw error;
    }
  },
  
  async deleteUser(id) {
    const { removeUser, setError, addNotification } = useAppStore.getState();
    
    try {
      await userAPI.deleteUser(id);
      removeUser(id);
      addNotification({
        type: 'success',
        message: 'User deleted successfully'
      });
    } catch (error) {
      setError(error.message);
      addNotification({
        type: 'error',
        message: 'Failed to delete user'
      });
      throw error;
    }
  }
};

// ============= SUBSCRIPTIONS =============
// store/subscriptions.js
import { useAppStore } from './use-app-store';
import { userActions } from './actions/user-actions';

// Theme persistence
useAppStore.subscribe(
  (state) => state.theme,
  (theme) => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }
);

// Notification auto-removal
useAppStore.subscribe(
  (state) => state.notifications,
  (notifications) => {
    notifications.forEach(notification => {
      if (!notification.persistent && !notification.timeout) {
        setTimeout(() => {
          useAppStore.getState().removeNotification(notification.id);
        }, 5000);
      }
    });
  }
);

// Real-time user updates
export const setupRealtimeSubscriptions = (websocket) => {
  websocket.onmessage = (event) => {
    const message = JSON.parse(event.data);
    const { updateUser, removeUser, addNotification } = useAppStore.getState();
    
    switch (message.type) {
      case 'user_updated':
        updateUser(message.data.id, message.data);
        addNotification({
          type: 'info',
          message: `User ${message.data.name} was updated`
        });
        break;
        
      case 'user_deleted':
        removeUser(message.data.id);
        addNotification({
          type: 'warning',
          message: 'A user was deleted'
        });
        break;
        
      case 'user_created':
        // Refresh user list
        userActions.fetchUsers();
        addNotification({
          type: 'success',
          message: 'New user was created'
        });
        break;
    }
  };
};
```

## 🔄 State Synchronization

### **🔁 Cross-Tab Synchronization**

```javascript
// ============= CROSS-TAB SYNC =============
// lib/sync/cross-tab-sync.js

export class CrossTabSync {
  constructor(storeName = 'app-sync') {
    this.storeName = storeName;
    this.channel = new BroadcastChannel(storeName);
    this.listeners = new Map();
    this.setupMessageHandling();
  }
  
  setupMessageHandling() {
    this.channel.onmessage = (event) => {
      const { type, payload, timestamp, tabId } = event.data;
      
      // Ignore our own messages
      if (tabId === this.getTabId()) return;
      
      // Call registered listeners
      const listeners = this.listeners.get(type) || [];
      listeners.forEach(listener => {
        try {
          listener(payload, { timestamp, tabId });
        } catch (error) {
          console.error('Cross-tab sync listener error:', error);
        }
      });
    };
  }
  
  // Broadcast message to other tabs
  broadcast(type, payload) {
    this.channel.postMessage({
      type,
      payload,
      timestamp: Date.now(),
      tabId: this.getTabId()
    });
  }
  
  // Listen for specific message types
  on(type, listener) {
    if (!this.listeners.has(type)) {
      this.listeners.set(type, []);
    }
    this.listeners.get(type).push(listener);
    
    // Return unsubscribe function
    return () => {
      const listeners = this.listeners.get(type);
      if (listeners) {
        const index = listeners.indexOf(listener);
        if (index !== -1) {
          listeners.splice(index, 1);
        }
      }
    };
  }
  
  // Remove all listeners for a type
  off(type) {
    this.listeners.delete(type);
  }
  
  // Get unique tab identifier
  getTabId() {
    if (!this.tabId) {
      this.tabId = `tab_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    }
    return this.tabId;
  }
  
  // Cleanup
  destroy() {
    this.channel.close();
    this.listeners.clear();
  }
}

// ============= ZUSTAND CROSS-TAB MIDDLEWARE =============
// store/middleware/cross-tab-middleware.js
import { CrossTabSync } from '@/lib/sync/cross-tab-sync';

export const crossTabMiddleware = (config) => (set, get, api) => {
  const sync = new CrossTabSync();
  
  // Sync state changes to other tabs
  const setState = (partial, replace) => {
    const result = set(partial, replace);
    
    // Broadcast state change
    sync.broadcast('state_change', {
      partial: typeof partial === 'function' ? null : partial,
      replace,
      newState: get()
    });
    
    return result;
  };
  
  // Listen for state changes from other tabs
  sync.on('state_change', (payload) => {
    if (payload.partial && !payload.replace) {
      // Merge partial state
      set(payload.partial, false);
    } else if (payload.newState) {
      // Handle complex state updates
      const currentState = get();
      const mergedState = mergeStates(currentState, payload.newState);
      set(mergedState, true);
    }
  });
  
  // Listen for specific actions
  sync.on('action', (payload) => {
    const { type, data } = payload;
    const state = get();
    
    // Handle specific action types
    switch (type) {
      case 'user_logout':
        // Clear sensitive data
        set((state) => ({
          ...state,
          currentUser: null,
          authToken: null
        }));
        break;
        
      case 'notification_add':
        // Add notification to all tabs
        if (state.addNotification) {
          state.addNotification(data);
        }
        break;
    }
  });
  
  // Create enhanced store
  const store = config(setState, get, api);
  
  // Add sync methods to store
  store.sync = {
    broadcast: (type, data) => sync.broadcast('action', { type, data }),
    on: (type, listener) => sync.on(type, listener),
    off: (type) => sync.off(type)
  };
  
  return store;
};

function mergeStates(current, incoming) {
  // Custom merge logic based on your needs
  return {
    ...current,
    ...incoming,
    // Preserve certain local state
    ui: current.ui, // Keep UI state local
    cache: current.cache // Keep cache local
  };
}

// ============= STORAGE SYNC =============
// lib/sync/storage-sync.js

export class StorageSync {
  constructor(options = {}) {
    this.options = {
      key: 'app-state',
      storage: localStorage,
      debounceMs: 1000,
      ...options
    };
    
    this.debounceTimeout = null;
    this.listeners = [];
    this.setupStorageListener();
  }
  
  setupStorageListener() {
    window.addEventListener('storage', (event) => {
      if (event.key === this.options.key && event.newValue) {
        try {
          const newState = JSON.parse(event.newValue);
          this.notifyListeners(newState);
        } catch (error) {
          console.error('Failed to parse storage state:', error);
        }
      }
    });
  }
  
  save(state) {
    if (this.debounceTimeout) {
      clearTimeout(this.debounceTimeout);
    }
    
    this.debounceTimeout = setTimeout(() => {
      try {
        const serialized = JSON.stringify(state);
        this.options.storage.setItem(this.options.key, serialized);
      } catch (error) {
        console.error('Failed to save state to storage:', error);
      }
    }, this.options.debounceMs);
  }
  
  load() {
    try {
      const stored = this.options.storage.getItem(this.options.key);
      return stored ? JSON.parse(stored) : null;
    } catch (error) {
      console.error('Failed to load state from storage:', error);
      return null;
    }
  }
  
  clear() {
    this.options.storage.removeItem(this.options.key);
  }
  
  onStateChange(listener) {
    this.listeners.push(listener);
    
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index !== -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  notifyListeners(state) {
    this.listeners.forEach(listener => {
      try {
        listener(state);
      } catch (error) {
        console.error('Storage sync listener error:', error);
      }
    });
  }
  
  destroy() {
    if (this.debounceTimeout) {
      clearTimeout(this.debounceTimeout);
    }
    this.listeners = [];
  }
}
```

## 🎯 แบบฝึกหัด

### **⚙️ แบบฝึกหัดที่ 1: Redux Advanced**
สร้าง comprehensive Redux system:

```javascript
// TODO: สร้าง Redux system

// 1. Entity adapter
const userAdapter = createEntityAdapter({
  // Normalized state
  // CRUD operations
  // Optimistic updates
});

// 2. RTK Query integration
const apiSlice = createApi({
  // Automated caching
  // Background refetching
  // Optimistic updates
});

// 3. Complex state logic
const workflowSlice = createSlice({
  // Multi-step processes
  // State machines
  // Event sourcing
});
```

### **🐻 แบบฝึกหัดที่ 2: Zustand Ecosystem**
สร้าง complete Zustand setup:

```javascript
// TODO: สร้าง Zustand ecosystem

// 1. Store composition
const useComposedStore = create(() => ({
  // Multiple store slices
  // Cross-slice actions
  // Computed values
}));

// 2. Middleware stack
const middleware = pipe(
  // Logging middleware
  // Persistence middleware
  // DevTools middleware
);

// 3. Store subscriptions
const subscriptions = {
  // External API sync
  // WebSocket integration
  // Background tasks
};
```

### **🔄 แบบฝึกหัดที่ 3: State Synchronization**
สร้าง advanced synchronization:

```javascript
// TODO: สร้าง synchronization system

// 1. Offline-first sync
class OfflineSync {
  // Queue management
  // Conflict resolution
  // Retry mechanisms
}

// 2. Real-time collaboration
class CollaborativeState {
  // Operational transforms
  // Presence tracking
  // Concurrent editing
}

// 3. Multi-device sync
class MultiDeviceSync {
  // Cloud synchronization
  // Device management
  // Partial sync
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 22: Architecture](./22-architecture.md)

**บทถัดไป**: [บทที่ 24: Security](./24-security.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Zustand Documentation](https://docs.pmnd.rs/zustand/getting-started/introduction)
- [State Management Patterns](https://www.patterns.dev/posts/state-management/)
- [Real-time State Synchronization](https://blog.logrocket.com/real-time-state-synchronization/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Advanced Redux** - RTK patterns, middleware, และ async thunks
- **Zustand Ecosystem** - store composition, middleware, และ subscriptions
- **State Synchronization** - cross-tab sync, storage sync, และ real-time updates
- **Performance Optimization** - memoization, normalization, และ caching
- **Best Practices** - scalable state architecture และ maintainable patterns

State Management เป็นหัวใจสำคัญของ complex applications! 🚀
