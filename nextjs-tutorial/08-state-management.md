# บทที่ 8: State Management

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ State Management patterns ใน Next.js
- ทำความเข้าใจ React Context API
- เรียนรู้ Redux Toolkit integration
- เข้าใจ Zustand และ SWR state management

## ⚛️ React State Basics

### **📊 Local State with useState**

```jsx
// components/TodoList.js
import { useState, useEffect } from 'react';

export default function TodoList() {
  const [todos, setTodos] = useState([]);
  const [newTodo, setNewTodo] = useState('');
  const [filter, setFilter] = useState('all'); // all, active, completed
  const [loading, setLoading] = useState(false);

  // ✅ Load todos on mount
  useEffect(() => {
    loadTodos();
  }, []);

  const loadTodos = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/todos');
      const data = await response.json();
      setTodos(data.todos || []);
    } catch (error) {
      console.error('Failed to load todos:', error);
    } finally {
      setLoading(false);
    }
  };

  const addTodo = async () => {
    if (!newTodo.trim()) return;

    const optimisticTodo = {
      id: Date.now(),
      text: newTodo.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    };

    // ✅ Optimistic update
    setTodos(prev => [...prev, optimisticTodo]);
    setNewTodo('');

    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text: newTodo.trim() })
      });

      if (response.ok) {
        const newTodoFromServer = await response.json();
        setTodos(prev => 
          prev.map(todo => 
            todo.id === optimisticTodo.id ? newTodoFromServer : todo
          )
        );
      } else {
        // ✅ Rollback on error
        setTodos(prev => prev.filter(todo => todo.id !== optimisticTodo.id));
      }
    } catch (error) {
      console.error('Failed to add todo:', error);
      setTodos(prev => prev.filter(todo => todo.id !== optimisticTodo.id));
    }
  };

  const toggleTodo = async (id) => {
    // ✅ Optimistic update
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );

    try {
      const todo = todos.find(t => t.id === id);
      const response = await fetch(`/api/todos/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed: !todo.completed })
      });

      if (!response.ok) {
        // ✅ Rollback on error
        setTodos(prev =>
          prev.map(todo =>
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
          )
        );
      }
    } catch (error) {
      console.error('Failed to toggle todo:', error);
      // Rollback
      setTodos(prev =>
        prev.map(todo =>
          todo.id === id ? { ...todo, completed: !todo.completed } : todo
        )
      );
    }
  };

  const deleteTodo = async (id) => {
    const todoToDelete = todos.find(t => t.id === id);
    
    // ✅ Optimistic update
    setTodos(prev => prev.filter(todo => todo.id !== id));

    try {
      const response = await fetch(`/api/todos/${id}`, {
        method: 'DELETE'
      });

      if (!response.ok) {
        // ✅ Rollback on error
        setTodos(prev => [...prev, todoToDelete].sort((a, b) => a.id - b.id));
      }
    } catch (error) {
      console.error('Failed to delete todo:', error);
      setTodos(prev => [...prev, todoToDelete].sort((a, b) => a.id - b.id));
    }
  };

  // ✅ Filtered todos
  const filteredTodos = todos.filter(todo => {
    switch (filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });

  const stats = {
    total: todos.length,
    active: todos.filter(t => !t.completed).length,
    completed: todos.filter(t => t.completed).length
  };

  if (loading) {
    return <div className="loading">Loading todos...</div>;
  }

  return (
    <div className="todo-list">
      <header className="todo-header">
        <h1>Todo List</h1>
        <div className="todo-stats">
          <span>Total: {stats.total}</span>
          <span>Active: {stats.active}</span>
          <span>Completed: {stats.completed}</span>
        </div>
      </header>

      <div className="todo-input">
        <input
          type="text"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add a new todo..."
        />
        <button onClick={addTodo} disabled={!newTodo.trim()}>
          Add
        </button>
      </div>

      <div className="todo-filters">
        <button 
          className={filter === 'all' ? 'active' : ''}
          onClick={() => setFilter('all')}
        >
          All
        </button>
        <button 
          className={filter === 'active' ? 'active' : ''}
          onClick={() => setFilter('active')}
        >
          Active
        </button>
        <button 
          className={filter === 'completed' ? 'active' : ''}
          onClick={() => setFilter('completed')}
        >
          Completed
        </button>
      </div>

      <ul className="todo-items">
        {filteredTodos.map(todo => (
          <li key={todo.id} className={`todo-item ${todo.completed ? 'completed' : ''}`}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span className="todo-text">{todo.text}</span>
            <button 
              className="delete-btn"
              onClick={() => deleteTodo(todo.id)}
            >
              ×
            </button>
          </li>
        ))}
      </ul>

      {filteredTodos.length === 0 && (
        <div className="empty-state">
          {filter === 'all' ? 'No todos yet' : `No ${filter} todos`}
        </div>
      )}
    </div>
  );
}
```

## 🌐 React Context API

### **🏪 Context Setup**

```jsx
// contexts/AppContext.js
import { createContext, useContext, useReducer, useEffect } from 'react';

// ✅ Initial State
const initialState = {
  user: null,
  theme: 'light',
  language: 'en',
  notifications: [],
  cart: {
    items: [],
    total: 0
  },
  ui: {
    sidebarOpen: false,
    loading: false,
    error: null
  }
};

// ✅ Action Types
const actionTypes = {
  SET_USER: 'SET_USER',
  LOGOUT: 'LOGOUT',
  SET_THEME: 'SET_THEME',
  SET_LANGUAGE: 'SET_LANGUAGE',
  ADD_NOTIFICATION: 'ADD_NOTIFICATION',
  REMOVE_NOTIFICATION: 'REMOVE_NOTIFICATION',
  ADD_TO_CART: 'ADD_TO_CART',
  REMOVE_FROM_CART: 'REMOVE_FROM_CART',
  UPDATE_CART_ITEM: 'UPDATE_CART_ITEM',
  CLEAR_CART: 'CLEAR_CART',
  TOGGLE_SIDEBAR: 'TOGGLE_SIDEBAR',
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR'
};

// ✅ Reducer
function appReducer(state, action) {
  switch (action.type) {
    case actionTypes.SET_USER:
      return {
        ...state,
        user: action.payload
      };

    case actionTypes.LOGOUT:
      return {
        ...state,
        user: null,
        cart: { items: [], total: 0 }
      };

    case actionTypes.SET_THEME:
      return {
        ...state,
        theme: action.payload
      };

    case actionTypes.SET_LANGUAGE:
      return {
        ...state,
        language: action.payload
      };

    case actionTypes.ADD_NOTIFICATION:
      return {
        ...state,
        notifications: [
          ...state.notifications,
          {
            id: Date.now(),
            ...action.payload,
            createdAt: new Date()
          }
        ]
      };

    case actionTypes.REMOVE_NOTIFICATION:
      return {
        ...state,
        notifications: state.notifications.filter(
          notification => notification.id !== action.payload
        )
      };

    case actionTypes.ADD_TO_CART:
      const existingItem = state.cart.items.find(
        item => item.id === action.payload.id
      );

      let newItems;
      if (existingItem) {
        newItems = state.cart.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + action.payload.quantity }
            : item
        );
      } else {
        newItems = [...state.cart.items, action.payload];
      }

      const newTotal = newItems.reduce(
        (total, item) => total + (item.price * item.quantity),
        0
      );

      return {
        ...state,
        cart: {
          items: newItems,
          total: newTotal
        }
      };

    case actionTypes.REMOVE_FROM_CART:
      const filteredItems = state.cart.items.filter(
        item => item.id !== action.payload
      );
      
      return {
        ...state,
        cart: {
          items: filteredItems,
          total: filteredItems.reduce(
            (total, item) => total + (item.price * item.quantity),
            0
          )
        }
      };

    case actionTypes.UPDATE_CART_ITEM:
      const updatedItems = state.cart.items.map(item =>
        item.id === action.payload.id
          ? { ...item, quantity: action.payload.quantity }
          : item
      );

      return {
        ...state,
        cart: {
          items: updatedItems,
          total: updatedItems.reduce(
            (total, item) => total + (item.price * item.quantity),
            0
          )
        }
      };

    case actionTypes.CLEAR_CART:
      return {
        ...state,
        cart: { items: [], total: 0 }
      };

    case actionTypes.TOGGLE_SIDEBAR:
      return {
        ...state,
        ui: {
          ...state.ui,
          sidebarOpen: !state.ui.sidebarOpen
        }
      };

    case actionTypes.SET_LOADING:
      return {
        ...state,
        ui: {
          ...state.ui,
          loading: action.payload
        }
      };

    case actionTypes.SET_ERROR:
      return {
        ...state,
        ui: {
          ...state.ui,
          error: action.payload
        }
      };

    default:
      return state;
  }
}

// ✅ Context
const AppContext = createContext();

// ✅ Provider Component
export function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);

  // ✅ Load persisted state
  useEffect(() => {
    const persistedTheme = localStorage.getItem('theme');
    const persistedLanguage = localStorage.getItem('language');
    const persistedCart = localStorage.getItem('cart');

    if (persistedTheme) {
      dispatch({ type: actionTypes.SET_THEME, payload: persistedTheme });
    }

    if (persistedLanguage) {
      dispatch({ type: actionTypes.SET_LANGUAGE, payload: persistedLanguage });
    }

    if (persistedCart) {
      try {
        const cart = JSON.parse(persistedCart);
        cart.items.forEach(item => {
          dispatch({ type: actionTypes.ADD_TO_CART, payload: item });
        });
      } catch (error) {
        console.error('Failed to load cart from localStorage:', error);
      }
    }
  }, []);

  // ✅ Persist state changes
  useEffect(() => {
    localStorage.setItem('theme', state.theme);
  }, [state.theme]);

  useEffect(() => {
    localStorage.setItem('language', state.language);
  }, [state.language]);

  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(state.cart));
  }, [state.cart]);

  // ✅ Action Creators
  const actions = {
    setUser: (user) => dispatch({ type: actionTypes.SET_USER, payload: user }),
    
    logout: () => {
      dispatch({ type: actionTypes.LOGOUT });
      localStorage.removeItem('cart');
    },
    
    setTheme: (theme) => dispatch({ type: actionTypes.SET_THEME, payload: theme }),
    
    setLanguage: (language) => dispatch({ type: actionTypes.SET_LANGUAGE, payload: language }),
    
    addNotification: (notification) => {
      dispatch({ type: actionTypes.ADD_NOTIFICATION, payload: notification });
      
      // ✅ Auto-remove after 5 seconds
      setTimeout(() => {
        dispatch({ type: actionTypes.REMOVE_NOTIFICATION, payload: notification.id || Date.now() });
      }, 5000);
    },
    
    removeNotification: (id) => dispatch({ type: actionTypes.REMOVE_NOTIFICATION, payload: id }),
    
    addToCart: (item) => dispatch({ type: actionTypes.ADD_TO_CART, payload: item }),
    
    removeFromCart: (id) => dispatch({ type: actionTypes.REMOVE_FROM_CART, payload: id }),
    
    updateCartItem: (id, quantity) => dispatch({ 
      type: actionTypes.UPDATE_CART_ITEM, 
      payload: { id, quantity } 
    }),
    
    clearCart: () => dispatch({ type: actionTypes.CLEAR_CART }),
    
    toggleSidebar: () => dispatch({ type: actionTypes.TOGGLE_SIDEBAR }),
    
    setLoading: (loading) => dispatch({ type: actionTypes.SET_LOADING, payload: loading }),
    
    setError: (error) => dispatch({ type: actionTypes.SET_ERROR, payload: error })
  };

  return (
    <AppContext.Provider value={{ state, actions }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ Custom Hook
export function useApp() {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
}

// ✅ Specific Hooks
export function useAuth() {
  const { state, actions } = useApp();
  return {
    user: state.user,
    isAuthenticated: !!state.user,
    setUser: actions.setUser,
    logout: actions.logout
  };
}

export function useTheme() {
  const { state, actions } = useApp();
  return {
    theme: state.theme,
    setTheme: actions.setTheme,
    isDark: state.theme === 'dark'
  };
}

export function useCart() {
  const { state, actions } = useApp();
  return {
    cart: state.cart,
    addToCart: actions.addToCart,
    removeFromCart: actions.removeFromCart,
    updateCartItem: actions.updateCartItem,
    clearCart: actions.clearCart,
    itemCount: state.cart.items.reduce((total, item) => total + item.quantity, 0)
  };
}

export function useNotifications() {
  const { state, actions } = useApp();
  return {
    notifications: state.notifications,
    addNotification: actions.addNotification,
    removeNotification: actions.removeNotification
  };
}
```

### **📱 Context Usage**

```jsx
// pages/_app.js
import { AppProvider } from '@/contexts/AppContext';
import Layout from '@/components/Layout';
import '@/styles/globals.css';

export default function MyApp({ Component, pageProps }) {
  return (
    <AppProvider>
      <Layout>
        <Component {...pageProps} />
      </Layout>
    </AppProvider>
  );
}

// components/Header.js
import { useAuth, useTheme, useCart } from '@/contexts/AppContext';
import Link from 'next/link';

export default function Header() {
  const { user, logout, isAuthenticated } = useAuth();
  const { theme, setTheme, isDark } = useTheme();
  const { itemCount } = useCart();

  return (
    <header className="header">
      <div className="header-content">
        <Link href="/" className="logo">
          My App
        </Link>

        <nav className="navigation">
          <Link href="/products">Products</Link>
          <Link href="/about">About</Link>
          <Link href="/contact">Contact</Link>
        </nav>

        <div className="header-actions">
          <button
            onClick={() => setTheme(isDark ? 'light' : 'dark')}
            className="theme-toggle"
          >
            {isDark ? '☀️' : '🌙'}
          </button>

          <Link href="/cart" className="cart-link">
            🛒 ({itemCount})
          </Link>

          {isAuthenticated ? (
            <div className="user-menu">
              <span>Hello, {user.name}</span>
              <button onClick={logout}>Logout</button>
            </div>
          ) : (
            <div className="auth-links">
              <Link href="/login">Login</Link>
              <Link href="/register">Register</Link>
            </div>
          )}
        </div>
      </div>
    </header>
  );
}

// components/ProductCard.js
import { useCart, useNotifications } from '@/contexts/AppContext';

export default function ProductCard({ product }) {
  const { addToCart } = useCart();
  const { addNotification } = useNotifications();

  const handleAddToCart = () => {
    addToCart({
      id: product.id,
      name: product.name,
      price: product.price,
      image: product.image,
      quantity: 1
    });

    addNotification({
      type: 'success',
      message: `${product.name} added to cart!`
    });
  };

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p className="price">${product.price}</p>
      <button onClick={handleAddToCart}>
        Add to Cart
      </button>
    </div>
  );
}
```

## 🔄 Redux Toolkit

### **📦 Redux Setup**

```bash
npm install @reduxjs/toolkit react-redux redux-persist
```

```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/lib/storage';
import { combineReducers } from '@reduxjs/toolkit';

import authSlice from './slices/authSlice';
import cartSlice from './slices/cartSlice';
import productsSlice from './slices/productsSlice';
import uiSlice from './slices/uiSlice';

// ✅ Persist Configuration
const persistConfig = {
  key: 'root',
  storage,
  whitelist: ['auth', 'cart'], // Only persist these slices
  blacklist: ['ui'] // Don't persist UI state
};

const rootReducer = combineReducers({
  auth: authSlice,
  cart: cartSlice,
  products: productsSlice,
  ui: uiSlice
});

const persistedReducer = persistReducer(persistConfig, rootReducer);

// ✅ Store Configuration
export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
      }
    }),
  devTools: process.env.NODE_ENV !== 'production'
});

export const persistor = persistStore(store);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### **🔐 Auth Slice**

```javascript
// store/slices/authSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// ✅ Async Thunks
export const login = createAsyncThunk(
  'auth/login',
  async (credentials, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });

      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message);
      }

      const data = await response.json();
      return data.user;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const register = createAsyncThunk(
  'auth/register',
  async (userData, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });

      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message);
      }

      const data = await response.json();
      return data.user;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const logout = createAsyncThunk(
  'auth/logout',
  async (_, { rejectWithValue }) => {
    try {
      await fetch('/api/auth/logout', { method: 'POST' });
      return null;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const verifyToken = createAsyncThunk(
  'auth/verifyToken',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/verify');
      
      if (!response.ok) {
        throw new Error('Token verification failed');
      }

      const data = await response.json();
      return data.user;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// ✅ Auth Slice
const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    isAuthenticated: false,
    loading: false,
    error: null,
    initialized: false
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    updateUser: (state, action) => {
      if (state.user) {
        state.user = { ...state.user, ...action.payload };
      }
    }
  },
  extraReducers: (builder) => {
    // Login
    builder
      .addCase(login.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
        state.isAuthenticated = true;
        state.error = null;
      })
      .addCase(login.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
        state.isAuthenticated = false;
        state.user = null;
      });

    // Register
    builder
      .addCase(register.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(register.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
        state.isAuthenticated = true;
        state.error = null;
      })
      .addCase(register.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });

    // Logout
    builder
      .addCase(logout.fulfilled, (state) => {
        state.user = null;
        state.isAuthenticated = false;
        state.error = null;
      });

    // Verify Token
    builder
      .addCase(verifyToken.pending, (state) => {
        state.loading = true;
      })
      .addCase(verifyToken.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
        state.isAuthenticated = true;
        state.initialized = true;
      })
      .addCase(verifyToken.rejected, (state) => {
        state.loading = false;
        state.user = null;
        state.isAuthenticated = false;
        state.initialized = true;
      });
  }
});

export const { clearError, updateUser } = authSlice.actions;
export default authSlice.reducer;
```

### **🛒 Cart Slice**

```javascript
// store/slices/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0,
    shipping: 0,
    tax: 0,
    discount: 0
  },
  reducers: {
    addToCart: (state, action) => {
      const existingItem = state.items.find(
        item => item.id === action.payload.id
      );

      if (existingItem) {
        existingItem.quantity += action.payload.quantity || 1;
      } else {
        state.items.push({
          ...action.payload,
          quantity: action.payload.quantity || 1
        });
      }

      cartSlice.caseReducers.calculateTotal(state);
    },

    removeFromCart: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
      cartSlice.caseReducers.calculateTotal(state);
    },

    updateQuantity: (state, action) => {
      const { id, quantity } = action.payload;
      const item = state.items.find(item => item.id === id);
      
      if (item) {
        if (quantity <= 0) {
          state.items = state.items.filter(item => item.id !== id);
        } else {
          item.quantity = quantity;
        }
      }

      cartSlice.caseReducers.calculateTotal(state);
    },

    clearCart: (state) => {
      state.items = [];
      state.total = 0;
      state.shipping = 0;
      state.tax = 0;
      state.discount = 0;
    },

    applyDiscount: (state, action) => {
      state.discount = action.payload;
      cartSlice.caseReducers.calculateTotal(state);
    },

    setShipping: (state, action) => {
      state.shipping = action.payload;
      cartSlice.caseReducers.calculateTotal(state);
    },

    calculateTotal: (state) => {
      const subtotal = state.items.reduce(
        (total, item) => total + (item.price * item.quantity),
        0
      );
      
      state.tax = subtotal * 0.1; // 10% tax
      state.total = subtotal + state.shipping + state.tax - state.discount;
    }
  }
});

export const {
  addToCart,
  removeFromCart,
  updateQuantity,
  clearCart,
  applyDiscount,
  setShipping
} = cartSlice.actions;

// ✅ Selectors
export const selectCartItems = (state) => state.cart.items;
export const selectCartTotal = (state) => state.cart.total;
export const selectCartItemCount = (state) => 
  state.cart.items.reduce((total, item) => total + item.quantity, 0);

export default cartSlice.reducer;
```

### **🏪 Redux Provider Setup**

```jsx
// pages/_app.js
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import { store, persistor } from '@/store';
import Layout from '@/components/Layout';

export default function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <PersistGate loading={<div>Loading...</div>} persistor={persistor}>
        <Layout>
          <Component {...pageProps} />
        </Layout>
      </PersistGate>
    </Provider>
  );
}

// hooks/redux.js
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from '@/store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### **📱 Redux Usage**

```jsx
// components/LoginForm.js
import { useState } from 'react';
import { useAppDispatch, useAppSelector } from '@/hooks/redux';
import { login, clearError } from '@/store/slices/authSlice';

export default function LoginForm() {
  const dispatch = useAppDispatch();
  const { loading, error } = useAppSelector(state => state.auth);
  
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (error) {
      dispatch(clearError());
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await dispatch(login(formData));
  };

  return (
    <form onSubmit={handleSubmit} className="login-form">
      <h2>Login</h2>
      
      {error && (
        <div className="error-message">
          {error}
        </div>
      )}
      
      <div className="form-group">
        <label>Email</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          required
        />
      </div>
      
      <div className="form-group">
        <label>Password</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          required
        />
      </div>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

// components/ShoppingCart.js
import { useAppDispatch, useAppSelector } from '@/hooks/redux';
import { 
  selectCartItems, 
  selectCartTotal, 
  selectCartItemCount,
  updateQuantity,
  removeFromCart 
} from '@/store/slices/cartSlice';

export default function ShoppingCart() {
  const dispatch = useAppDispatch();
  const items = useAppSelector(selectCartItems);
  const total = useAppSelector(selectCartTotal);
  const itemCount = useAppSelector(selectCartItemCount);

  const handleQuantityChange = (id, quantity) => {
    dispatch(updateQuantity({ id, quantity: parseInt(quantity) }));
  };

  const handleRemoveItem = (id) => {
    dispatch(removeFromCart(id));
  };

  return (
    <div className="shopping-cart">
      <h2>Shopping Cart ({itemCount} items)</h2>
      
      {items.length === 0 ? (
        <p>Your cart is empty</p>
      ) : (
        <>
          <div className="cart-items">
            {items.map(item => (
              <div key={item.id} className="cart-item">
                <img src={item.image} alt={item.name} />
                <div className="item-details">
                  <h3>{item.name}</h3>
                  <p>${item.price}</p>
                </div>
                <div className="quantity-controls">
                  <select
                    value={item.quantity}
                    onChange={(e) => handleQuantityChange(item.id, e.target.value)}
                  >
                    {[...Array(10)].map((_, i) => (
                      <option key={i + 1} value={i + 1}>
                        {i + 1}
                      </option>
                    ))}
                  </select>
                </div>
                <div className="item-total">
                  ${(item.price * item.quantity).toFixed(2)}
                </div>
                <button onClick={() => handleRemoveItem(item.id)}>
                  Remove
                </button>
              </div>
            ))}
          </div>
          
          <div className="cart-summary">
            <h3>Total: ${total.toFixed(2)}</h3>
            <button className="checkout-btn">
              Proceed to Checkout
            </button>
          </div>
        </>
      )}
    </div>
  );
}
```

## 🐻 Zustand

### **📦 Zustand Setup**

```bash
npm install zustand
```

```javascript
// stores/useAuthStore.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useAuthStore = create(
  persist(
    (set, get) => ({
      // ✅ State
      user: null,
      isAuthenticated: false,
      loading: false,
      error: null,

      // ✅ Actions
      login: async (credentials) => {
        set({ loading: true, error: null });
        
        try {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(credentials)
          });

          if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message);
          }

          const data = await response.json();
          
          set({ 
            user: data.user, 
            isAuthenticated: true, 
            loading: false,
            error: null 
          });
          
          return data.user;
        } catch (error) {
          set({ 
            loading: false, 
            error: error.message,
            isAuthenticated: false,
            user: null 
          });
          throw error;
        }
      },

      register: async (userData) => {
        set({ loading: true, error: null });
        
        try {
          const response = await fetch('/api/auth/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData)
          });

          if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message);
          }

          const data = await response.json();
          
          set({ 
            user: data.user, 
            isAuthenticated: true, 
            loading: false,
            error: null 
          });
          
          return data.user;
        } catch (error) {
          set({ loading: false, error: error.message });
          throw error;
        }
      },

      logout: async () => {
        try {
          await fetch('/api/auth/logout', { method: 'POST' });
        } catch (error) {
          console.error('Logout error:', error);
        } finally {
          set({ 
            user: null, 
            isAuthenticated: false, 
            error: null 
          });
        }
      },

      updateUser: (userData) => {
        const currentUser = get().user;
        if (currentUser) {
          set({ user: { ...currentUser, ...userData } });
        }
      },

      clearError: () => set({ error: null }),

      // ✅ Computed values
      get isAdmin() {
        return get().user?.role === 'admin';
      },

      get userName() {
        return get().user?.name || 'Guest';
      }
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ 
        user: state.user, 
        isAuthenticated: state.isAuthenticated 
      })
    }
  )
);

export default useAuthStore;

// stores/useCartStore.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useCartStore = create(
  persist(
    (set, get) => ({
      // ✅ State
      items: [],
      isOpen: false,

      // ✅ Actions
      addToCart: (product) => {
        const items = get().items;
        const existingItem = items.find(item => item.id === product.id);

        if (existingItem) {
          set({
            items: items.map(item =>
              item.id === product.id
                ? { ...item, quantity: item.quantity + 1 }
                : item
            )
          });
        } else {
          set({ items: [...items, { ...product, quantity: 1 }] });
        }
      },

      removeFromCart: (productId) => {
        set({
          items: get().items.filter(item => item.id !== productId)
        });
      },

      updateQuantity: (productId, quantity) => {
        if (quantity <= 0) {
          get().removeFromCart(productId);
          return;
        }

        set({
          items: get().items.map(item =>
            item.id === productId ? { ...item, quantity } : item
          )
        });
      },

      clearCart: () => set({ items: [] }),

      toggleCart: () => set({ isOpen: !get().isOpen }),

      openCart: () => set({ isOpen: true }),

      closeCart: () => set({ isOpen: false }),

      // ✅ Computed values
      get total() {
        return get().items.reduce(
          (total, item) => total + item.price * item.quantity,
          0
        );
      },

      get itemCount() {
        return get().items.reduce(
          (count, item) => count + item.quantity,
          0
        );
      },

      get isEmpty() {
        return get().items.length === 0;
      }
    }),
    {
      name: 'cart-storage'
    }
  )
);

export default useCartStore;

// stores/useUIStore.js
import { create } from 'zustand';

const useUIStore = create((set, get) => ({
  // ✅ State
  theme: 'light',
  sidebarOpen: false,
  loading: false,
  notifications: [],

  // ✅ Actions
  setTheme: (theme) => set({ theme }),

  toggleTheme: () => {
    const currentTheme = get().theme;
    set({ theme: currentTheme === 'light' ? 'dark' : 'light' });
  },

  toggleSidebar: () => set({ sidebarOpen: !get().sidebarOpen }),

  openSidebar: () => set({ sidebarOpen: true }),

  closeSidebar: () => set({ sidebarOpen: false }),

  setLoading: (loading) => set({ loading }),

  addNotification: (notification) => {
    const id = Date.now();
    const newNotification = { id, ...notification };
    
    set({ 
      notifications: [...get().notifications, newNotification] 
    });

    // Auto-remove after 5 seconds
    setTimeout(() => {
      get().removeNotification(id);
    }, 5000);

    return id;
  },

  removeNotification: (id) => {
    set({
      notifications: get().notifications.filter(n => n.id !== id)
    });
  },

  clearNotifications: () => set({ notifications: [] }),

  // ✅ Computed values
  get isDark() {
    return get().theme === 'dark';
  }
}));

export default useUIStore;
```

### **📱 Zustand Usage**

```jsx
// components/ProductCard.js
import useCartStore from '@/stores/useCartStore';
import useUIStore from '@/stores/useUIStore';

export default function ProductCard({ product }) {
  const addToCart = useCartStore(state => state.addToCart);
  const addNotification = useUIStore(state => state.addNotification);

  const handleAddToCart = () => {
    addToCart(product);
    addNotification({
      type: 'success',
      message: `${product.name} added to cart!`
    });
  };

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p className="price">${product.price}</p>
      <button onClick={handleAddToCart}>
        Add to Cart
      </button>
    </div>
  );
}

// components/Header.js
import useAuthStore from '@/stores/useAuthStore';
import useCartStore from '@/stores/useCartStore';
import useUIStore from '@/stores/useUIStore';

export default function Header() {
  const { user, isAuthenticated, logout } = useAuthStore();
  const { itemCount, openCart } = useCartStore();
  const { theme, toggleTheme, isDark } = useUIStore();

  return (
    <header className="header">
      <div className="header-content">
        <h1>My Store</h1>
        
        <nav>
          <a href="/products">Products</a>
          <a href="/about">About</a>
        </nav>
        
        <div className="header-actions">
          <button onClick={toggleTheme}>
            {isDark ? '☀️' : '🌙'}
          </button>
          
          <button onClick={openCart}>
            🛒 ({itemCount})
          </button>
          
          {isAuthenticated ? (
            <div>
              <span>Hello, {user.name}</span>
              <button onClick={logout}>Logout</button>
            </div>
          ) : (
            <a href="/login">Login</a>
          )}
        </div>
      </div>
    </header>
  );
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Context API**
1. สร้าง Theme Context สำหรับ dark/light mode
2. เพิ่ม Shopping Cart Context
3. สร้าง Notification System
4. เพิ่ม User Authentication Context

### **🎯 แบบฝึกหัดที่ 2: Redux Toolkit**
1. Setup Redux store ด้วย RTK
2. สร้าง async thunks สำหรับ API calls
3. เพิ่ม Redux Persist
4. สร้าง custom selectors

### **🎯 แบบฝึกหัดที่ 3: Zustand**
1. แปลง Context เป็น Zustand stores
2. เพิ่ม persistence middleware
3. สร้าง computed values
4. เพิ่ม store subscriptions

### **🎯 แบบฝึกหัดที่ 4: State Management Comparison**
1. เปรียบเทียบ performance ระหว่าง solutions
2. ทดสอบ bundle size impact
3. เปรียบเทียบ developer experience
4. สร้าง migration guide

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 7: API Routes](./07-api-routes.md)

**บทต่อไป**: [บทที่ 9: Database Integration](./09-database-integration.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [React Context API](https://react.dev/reference/react/useContext)
- [Redux Toolkit](https://redux-toolkit.js.org/)
- [Zustand](https://zustand-demo.pmnd.rs/)
- [Redux Persist](https://github.com/rt2zz/redux-persist)
