# บทที่ 6: Data Fetching Basics

## 🎯 จุดประสงค์ของบทเรียน
- ทำความเข้าใจ Data Fetching patterns ใน Next.js
- เรียนรู้ getStaticProps, getServerSideProps และ getStaticPaths
- เข้าใจ Client-side Data Fetching
- เรียนรู้ SWR และ React Query integration

## 📊 Data Fetching Overview

### **🎪 Rendering Methods**

```javascript
// ✅ Static Generation (SSG) - Pre-rendered at build time
export async function getStaticProps() {
  // Fetch data at build time
  return {
    props: { data }
  };
}

// ✅ Server-Side Rendering (SSR) - Pre-rendered on each request
export async function getServerSideProps() {
  // Fetch data on each request
  return {
    props: { data }
  };
}

// ✅ Client-Side Rendering (CSR) - Fetch data in browser
export default function Page() {
  useEffect(() => {
    // Fetch data after component mounts
  }, []);
}

// ✅ Incremental Static Regeneration (ISR) - Update static content
export async function getStaticProps() {
  return {
    props: { data },
    revalidate: 60 // Revalidate every 60 seconds
  };
}
```

## 🏗️ Static Generation (SSG)

### **📋 getStaticProps**

```jsx
// pages/blog/index.js
export default function BlogPage({ posts }) {
  return (
    <div className="blog-page">
      <h1>Blog Posts</h1>
      <div className="posts-grid">
        {posts.map((post) => (
          <article key={post.id} className="post-card">
            <img src={post.image} alt={post.title} />
            <h2>{post.title}</h2>
            <p>{post.excerpt}</p>
            <div className="post-meta">
              <span>{post.author}</span>
              <span>{new Date(post.publishedAt).toLocaleDateString()}</span>
            </div>
          </article>
        ))}
      </div>
    </div>
  );
}

export async function getStaticProps() {
  try {
    // ✅ Fetch data from API
    const response = await fetch('https://api.example.com/posts');
    const posts = await response.json();

    // ✅ Fetch data from file system
    const fs = require('fs');
    const path = require('path');
    const postsDirectory = path.join(process.cwd(), 'content/posts');
    const filenames = fs.readdirSync(postsDirectory);
    
    const localPosts = filenames.map((name) => {
      const filePath = path.join(postsDirectory, name);
      const fileContents = fs.readFileSync(filePath, 'utf8');
      return JSON.parse(fileContents);
    });

    // ✅ Combine data sources
    const allPosts = [...posts, ...localPosts];

    return {
      props: {
        posts: allPosts
      },
      // ✅ Incremental Static Regeneration
      revalidate: 3600 // Revalidate every hour
    };
  } catch (error) {
    console.error('Error fetching posts:', error);
    
    return {
      props: {
        posts: []
      },
      revalidate: 60 // Retry more frequently on error
    };
  }
}
```

### **🎯 getStaticPaths สำหรับ Dynamic Routes**

```jsx
// pages/blog/[slug].js
export default function BlogPost({ post, relatedPosts }) {
  if (!post) {
    return <div>Post not found</div>;
  }

  return (
    <article className="blog-post">
      <header className="post-header">
        <img src={post.image} alt={post.title} className="post-image" />
        <h1>{post.title}</h1>
        <div className="post-meta">
          <span>By {post.author}</span>
          <span>{new Date(post.publishedAt).toLocaleDateString()}</span>
          <span>{post.readTime} min read</span>
        </div>
      </header>

      <div className="post-content">
        <div dangerouslySetInnerHTML={{ __html: post.content }} />
      </div>

      <footer className="post-footer">
        <div className="tags">
          {post.tags.map((tag) => (
            <span key={tag} className="tag">
              {tag}
            </span>
          ))}
        </div>

        <div className="related-posts">
          <h3>Related Posts</h3>
          <div className="related-grid">
            {relatedPosts.map((relatedPost) => (
              <a 
                key={relatedPost.slug} 
                href={`/blog/${relatedPost.slug}`}
                className="related-post"
              >
                <img src={relatedPost.image} alt={relatedPost.title} />
                <h4>{relatedPost.title}</h4>
              </a>
            ))}
          </div>
        </div>
      </footer>
    </article>
  );
}

export async function getStaticPaths() {
  try {
    // ✅ Fetch all possible paths
    const response = await fetch('https://api.example.com/posts');
    const posts = await response.json();

    // ✅ Pre-generate popular posts only
    const popularPosts = posts.filter(post => post.views > 1000);
    
    const paths = popularPosts.map((post) => ({
      params: { slug: post.slug }
    }));

    return {
      paths,
      // ✅ fallback strategies
      fallback: 'blocking' // or true, false
    };
  } catch (error) {
    console.error('Error in getStaticPaths:', error);
    
    return {
      paths: [],
      fallback: 'blocking'
    };
  }
}

export async function getStaticProps({ params, preview = false }) {
  try {
    const { slug } = params;

    // ✅ Fetch main post
    const postResponse = await fetch(`https://api.example.com/posts/${slug}`);
    
    if (!postResponse.ok) {
      return {
        notFound: true
      };
    }

    const post = await postResponse.json();

    // ✅ Fetch related posts
    const relatedResponse = await fetch(
      `https://api.example.com/posts/${slug}/related?limit=3`
    );
    const relatedPosts = await relatedResponse.json();

    return {
      props: {
        post,
        relatedPosts,
        preview
      },
      revalidate: 3600 // 1 hour
    };
  } catch (error) {
    console.error('Error fetching post:', error);
    
    return {
      notFound: true
    };
  }
}
```

### **🔄 Fallback Strategies**

```jsx
// pages/products/[id].js
import { useRouter } from 'next/router';

export default function ProductPage({ product }) {
  const router = useRouter();

  // ✅ Show loading state for fallback: true
  if (router.isFallback) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>Loading product...</p>
      </div>
    );
  }

  // ✅ Handle 404 case
  if (!product) {
    return (
      <div className="not-found">
        <h1>Product not found</h1>
        <p>The product you're looking for doesn't exist.</p>
        <button onClick={() => router.push('/products')}>
          View All Products
        </button>
      </div>
    );
  }

  return (
    <div className="product-page">
      <div className="product-gallery">
        <img src={product.image} alt={product.name} />
      </div>
      
      <div className="product-info">
        <h1>{product.name}</h1>
        <p className="price">${product.price}</p>
        <p className="description">{product.description}</p>
        
        <button className="add-to-cart">
          Add to Cart
        </button>
      </div>
    </div>
  );
}

export async function getStaticPaths() {
  // ✅ Pre-generate only featured products
  const response = await fetch('https://api.example.com/products/featured');
  const featuredProducts = await response.json();

  const paths = featuredProducts.map((product) => ({
    params: { id: product.id.toString() }
  }));

  return {
    paths,
    fallback: true // Generate other pages on-demand
  };
}

export async function getStaticProps({ params }) {
  try {
    const response = await fetch(`https://api.example.com/products/${params.id}`);
    
    if (!response.ok) {
      return { notFound: true };
    }

    const product = await response.json();

    return {
      props: { product },
      revalidate: 86400 // 24 hours
    };
  } catch (error) {
    return { notFound: true };
  }
}
```

## 🖥️ Server-Side Rendering (SSR)

### **⚡ getServerSideProps**

```jsx
// pages/dashboard.js
export default function Dashboard({ user, stats, notifications }) {
  return (
    <div className="dashboard">
      <header className="dashboard-header">
        <h1>Welcome back, {user.name}!</h1>
        <div className="user-avatar">
          <img src={user.avatar} alt={user.name} />
        </div>
      </header>

      <div className="dashboard-stats">
        <div className="stat-card">
          <h3>Revenue</h3>
          <p>${stats.revenue.toLocaleString()}</p>
          <span className={`trend ${stats.revenueTrend > 0 ? 'up' : 'down'}`}>
            {stats.revenueTrend > 0 ? '+' : ''}{stats.revenueTrend}%
          </span>
        </div>
        
        <div className="stat-card">
          <h3>Orders</h3>
          <p>{stats.orders.toLocaleString()}</p>
          <span className={`trend ${stats.ordersTrend > 0 ? 'up' : 'down'}`}>
            {stats.ordersTrend > 0 ? '+' : ''}{stats.ordersTrend}%
          </span>
        </div>
        
        <div className="stat-card">
          <h3>Customers</h3>
          <p>{stats.customers.toLocaleString()}</p>
          <span className={`trend ${stats.customersTrend > 0 ? 'up' : 'down'}`}>
            {stats.customersTrend > 0 ? '+' : ''}{stats.customersTrend}%
          </span>
        </div>
      </div>

      <div className="dashboard-notifications">
        <h2>Recent Notifications</h2>
        <ul>
          {notifications.map((notification) => (
            <li key={notification.id} className={`notification ${notification.type}`}>
              <span className="notification-icon">{notification.icon}</span>
              <div className="notification-content">
                <p>{notification.message}</p>
                <span className="notification-time">
                  {new Date(notification.createdAt).toLocaleString()}
                </span>
              </div>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

export async function getServerSideProps({ req, res, query }) {
  try {
    // ✅ Get user from session/cookie
    const cookies = req.headers.cookie || '';
    const sessionToken = cookies
      .split(';')
      .find(c => c.trim().startsWith('session='))
      ?.split('=')[1];

    if (!sessionToken) {
      return {
        redirect: {
          destination: '/login',
          permanent: false
        }
      };
    }

    // ✅ Verify session and get user
    const userResponse = await fetch('https://api.example.com/auth/verify', {
      headers: {
        'Authorization': `Bearer ${sessionToken}`,
        'Cookie': req.headers.cookie
      }
    });

    if (!userResponse.ok) {
      return {
        redirect: {
          destination: '/login',
          permanent: false
        }
      };
    }

    const user = await userResponse.json();

    // ✅ Fetch personalized data
    const [statsResponse, notificationsResponse] = await Promise.all([
      fetch(`https://api.example.com/users/${user.id}/stats`, {
        headers: { 'Authorization': `Bearer ${sessionToken}` }
      }),
      fetch(`https://api.example.com/users/${user.id}/notifications?limit=5`, {
        headers: { 'Authorization': `Bearer ${sessionToken}` }
      })
    ]);

    const stats = await statsResponse.json();
    const notifications = await notificationsResponse.json();

    // ✅ Set cache headers
    res.setHeader(
      'Cache-Control',
      'private, s-maxage=0, max-age=0, must-revalidate'
    );

    return {
      props: {
        user,
        stats,
        notifications
      }
    };
  } catch (error) {
    console.error('Dashboard data fetch error:', error);
    
    return {
      props: {
        user: null,
        stats: null,
        notifications: []
      }
    };
  }
}
```

### **🔐 Authentication ใน SSR**

```jsx
// pages/profile.js
import { withAuth } from '@/lib/auth';

export default function ProfilePage({ user, orders, preferences }) {
  return (
    <div className="profile-page">
      <section className="profile-info">
        <h1>Profile Settings</h1>
        <form>
          <div className="form-group">
            <label>Full Name</label>
            <input 
              type="text" 
              defaultValue={user.name}
              name="name"
            />
          </div>
          
          <div className="form-group">
            <label>Email</label>
            <input 
              type="email" 
              defaultValue={user.email}
              name="email"
            />
          </div>
          
          <div className="form-group">
            <label>Phone</label>
            <input 
              type="tel" 
              defaultValue={user.phone}
              name="phone"
            />
          </div>
          
          <button type="submit">Update Profile</button>
        </form>
      </section>

      <section className="order-history">
        <h2>Order History</h2>
        <div className="orders-list">
          {orders.map((order) => (
            <div key={order.id} className="order-item">
              <h3>Order #{order.number}</h3>
              <p>Total: ${order.total}</p>
              <p>Status: {order.status}</p>
              <p>Date: {new Date(order.createdAt).toLocaleDateString()}</p>
            </div>
          ))}
        </div>
      </section>

      <section className="preferences">
        <h2>Preferences</h2>
        <div className="preferences-form">
          <label>
            <input 
              type="checkbox" 
              defaultChecked={preferences.emailNotifications}
            />
            Email Notifications
          </label>
          
          <label>
            <input 
              type="checkbox" 
              defaultChecked={preferences.smsNotifications}
            />
            SMS Notifications
          </label>
          
          <label>
            <input 
              type="checkbox" 
              defaultChecked={preferences.marketingEmails}
            />
            Marketing Emails
          </label>
        </div>
      </section>
    </div>
  );
}

export const getServerSideProps = withAuth(async ({ req, user }) => {
  try {
    // ✅ User already authenticated by withAuth HOF
    const sessionToken = req.headers.cookie
      ?.split(';')
      ?.find(c => c.trim().startsWith('session='))
      ?.split('=')[1];

    // ✅ Fetch user-specific data
    const [ordersResponse, preferencesResponse] = await Promise.all([
      fetch(`https://api.example.com/users/${user.id}/orders`, {
        headers: { 'Authorization': `Bearer ${sessionToken}` }
      }),
      fetch(`https://api.example.com/users/${user.id}/preferences`, {
        headers: { 'Authorization': `Bearer ${sessionToken}` }
      })
    ]);

    const orders = await ordersResponse.json();
    const preferences = await preferencesResponse.json();

    return {
      props: {
        user,
        orders,
        preferences
      }
    };
  } catch (error) {
    console.error('Profile data fetch error:', error);
    
    return {
      props: {
        user,
        orders: [],
        preferences: {}
      }
    };
  }
});

// lib/auth.js
export function withAuth(gssp) {
  return async (context) => {
    const { req, res } = context;
    
    try {
      const cookies = req.headers.cookie || '';
      const sessionToken = cookies
        .split(';')
        .find(c => c.trim().startsWith('session='))
        ?.split('=')[1];

      if (!sessionToken) {
        return {
          redirect: {
            destination: '/login',
            permanent: false
          }
        };
      }

      const userResponse = await fetch('https://api.example.com/auth/verify', {
        headers: { 'Authorization': `Bearer ${sessionToken}` }
      });

      if (!userResponse.ok) {
        return {
          redirect: {
            destination: '/login',
            permanent: false
          }
        };
      }

      const user = await userResponse.json();
      
      // ✅ Call the original getServerSideProps with user data
      return await gssp({ ...context, user });
    } catch (error) {
      console.error('Auth error:', error);
      
      return {
        redirect: {
          destination: '/login',
          permanent: false
        }
      };
    }
  };
}
```

## 🌐 Client-Side Data Fetching

### **⚛️ useEffect และ useState**

```jsx
// pages/search.js
import { useState, useEffect, useMemo } from 'react';
import { useRouter } from 'next/router';
import { useDebounce } from '@/hooks/useDebounce';

export default function SearchPage() {
  const router = useRouter();
  const [query, setQuery] = useState(router.query.q || '');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [filters, setFilters] = useState({
    category: 'all',
    sortBy: 'relevance',
    dateRange: 'all'
  });

  // ✅ Debounce search query
  const debouncedQuery = useDebounce(query, 300);

  // ✅ Search function
  const searchProducts = async (searchQuery, searchFilters) => {
    if (!searchQuery.trim()) {
      setResults([]);
      return;
    }

    setLoading(true);
    setError(null);

    try {
      const params = new URLSearchParams({
        q: searchQuery,
        category: searchFilters.category,
        sortBy: searchFilters.sortBy,
        dateRange: searchFilters.dateRange
      });

      const response = await fetch(`/api/search?${params}`);
      
      if (!response.ok) {
        throw new Error('Search failed');
      }

      const data = await response.json();
      setResults(data.results || []);
    } catch (err) {
      setError(err.message);
      setResults([]);
    } finally {
      setLoading(false);
    }
  };

  // ✅ Effect for debounced search
  useEffect(() => {
    searchProducts(debouncedQuery, filters);
  }, [debouncedQuery, filters]);

  // ✅ Update URL when query changes
  useEffect(() => {
    if (debouncedQuery !== router.query.q) {
      router.push({
        pathname: router.pathname,
        query: { ...router.query, q: debouncedQuery }
      }, undefined, { shallow: true });
    }
  }, [debouncedQuery]);

  // ✅ Memoized filtered results
  const filteredResults = useMemo(() => {
    return results.filter(result => {
      if (filters.category !== 'all' && result.category !== filters.category) {
        return false;
      }
      return true;
    });
  }, [results, filters]);

  const handleFilterChange = (filterType, value) => {
    setFilters(prev => ({
      ...prev,
      [filterType]: value
    }));
  };

  return (
    <div className="search-page">
      <div className="search-header">
        <h1>Search Products</h1>
        
        <div className="search-input-container">
          <input
            type="text"
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            placeholder="Search for products..."
            className="search-input"
          />
          {loading && <div className="search-spinner">🔄</div>}
        </div>
      </div>

      <div className="search-content">
        <aside className="search-filters">
          <h3>Filters</h3>
          
          <div className="filter-group">
            <label>Category</label>
            <select
              value={filters.category}
              onChange={(e) => handleFilterChange('category', e.target.value)}
            >
              <option value="all">All Categories</option>
              <option value="electronics">Electronics</option>
              <option value="clothing">Clothing</option>
              <option value="books">Books</option>
              <option value="home">Home & Garden</option>
            </select>
          </div>

          <div className="filter-group">
            <label>Sort By</label>
            <select
              value={filters.sortBy}
              onChange={(e) => handleFilterChange('sortBy', e.target.value)}
            >
              <option value="relevance">Relevance</option>
              <option value="price_low">Price: Low to High</option>
              <option value="price_high">Price: High to Low</option>
              <option value="rating">Customer Rating</option>
              <option value="newest">Newest First</option>
            </select>
          </div>

          <div className="filter-group">
            <label>Date Range</label>
            <select
              value={filters.dateRange}
              onChange={(e) => handleFilterChange('dateRange', e.target.value)}
            >
              <option value="all">All Time</option>
              <option value="week">Past Week</option>
              <option value="month">Past Month</option>
              <option value="year">Past Year</option>
            </select>
          </div>
        </aside>

        <main className="search-results">
          {error && (
            <div className="error-message">
              <h3>Search Error</h3>
              <p>{error}</p>
              <button onClick={() => searchProducts(debouncedQuery, filters)}>
                Try Again
              </button>
            </div>
          )}

          {!loading && !error && debouncedQuery && (
            <div className="results-header">
              <h2>
                {filteredResults.length} results for "{debouncedQuery}"
              </h2>
            </div>
          )}

          <div className="results-grid">
            {filteredResults.map((result) => (
              <div key={result.id} className="result-card">
                <img src={result.image} alt={result.title} />
                <div className="result-content">
                  <h3>{result.title}</h3>
                  <p className="result-price">${result.price}</p>
                  <p className="result-description">{result.description}</p>
                  <div className="result-rating">
                    <span className="stars">⭐⭐⭐⭐⭐</span>
                    <span>({result.reviews} reviews)</span>
                  </div>
                </div>
              </div>
            ))}
          </div>

          {!loading && !error && filteredResults.length === 0 && debouncedQuery && (
            <div className="no-results">
              <h3>No results found</h3>
              <p>Try adjusting your search terms or filters.</p>
            </div>
          )}
        </main>
      </div>
    </div>
  );
}

// hooks/useDebounce.js
import { useState, useEffect } from 'react';

export function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

## 🔄 SWR Integration

### **📦 SWR Setup**

```bash
npm install swr
```

```jsx
// hooks/useSWR.js
import useSWR from 'swr';

const fetcher = async (url) => {
  const response = await fetch(url);
  
  if (!response.ok) {
    const error = new Error('An error occurred while fetching the data.');
    error.info = await response.json();
    error.status = response.status;
    throw error;
  }
  
  return response.json();
};

export function useApi(endpoint) {
  const { data, error, mutate } = useSWR(endpoint, fetcher, {
    revalidateOnFocus: false,
    revalidateOnReconnect: true,
    refreshInterval: 0,
    dedupingInterval: 2000
  });

  return {
    data,
    loading: !error && !data,
    error,
    mutate
  };
}

export function useUser() {
  return useApi('/api/user');
}

export function usePosts() {
  return useApi('/api/posts');
}

export function usePost(id) {
  return useApi(id ? `/api/posts/${id}` : null);
}
```

### **🎯 SWR Examples**

```jsx
// pages/posts/index.js
import { useState } from 'react';
import { usePosts } from '@/hooks/useSWR';
import LoadingSpinner from '@/components/LoadingSpinner';
import ErrorBoundary from '@/components/ErrorBoundary';

export default function PostsPage() {
  const { data: posts, loading, error, mutate } = usePosts();
  const [creating, setCreating] = useState(false);

  const handleCreatePost = async (postData) => {
    setCreating(true);
    
    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(postData)
      });

      if (response.ok) {
        // ✅ Revalidate the posts list
        mutate();
      }
    } catch (error) {
      console.error('Create post error:', error);
    } finally {
      setCreating(false);
    }
  };

  const handleDeletePost = async (postId) => {
    try {
      // ✅ Optimistic update
      const optimisticData = posts.filter(post => post.id !== postId);
      mutate(optimisticData, false);

      const response = await fetch(`/api/posts/${postId}`, {
        method: 'DELETE'
      });

      if (!response.ok) {
        // ✅ Revert on failure
        mutate();
      }
    } catch (error) {
      // ✅ Revert on error
      mutate();
    }
  };

  if (error) {
    return (
      <ErrorBoundary>
        <div className="error-state">
          <h2>Failed to load posts</h2>
          <p>{error.message}</p>
          <button onClick={() => mutate()}>Retry</button>
        </div>
      </ErrorBoundary>
    );
  }

  if (loading) {
    return (
      <div className="loading-state">
        <LoadingSpinner />
        <p>Loading posts...</p>
      </div>
    );
  }

  return (
    <div className="posts-page">
      <header className="posts-header">
        <h1>Blog Posts</h1>
        <CreatePostForm 
          onSubmit={handleCreatePost}
          loading={creating}
        />
      </header>

      <div className="posts-grid">
        {posts?.map((post) => (
          <PostCard
            key={post.id}
            post={post}
            onDelete={() => handleDeletePost(post.id)}
          />
        ))}
      </div>

      {posts?.length === 0 && (
        <div className="empty-state">
          <h3>No posts yet</h3>
          <p>Create your first blog post to get started.</p>
        </div>
      )}
    </div>
  );
}
```

### **🔄 SWR Configuration**

```jsx
// pages/_app.js
import { SWRConfig } from 'swr';

function MyApp({ Component, pageProps }) {
  return (
    <SWRConfig
      value={{
        fetcher: (resource, init) => 
          fetch(resource, init).then(res => res.json()),
        onError: (error, key) => {
          console.error('SWR Error:', error, 'Key:', key);
          
          if (error.status === 403 || error.status === 401) {
            // Redirect to login on auth errors
            window.location.href = '/login';
          }
        },
        onErrorRetry: (error, key, config, revalidate, { retryCount }) => {
          // Never retry on 404
          if (error.status === 404) return;

          // Never retry on specific keys
          if (key === '/api/user') return;

          // Only retry up to 10 times
          if (retryCount >= 10) return;

          // Retry after 5 seconds
          setTimeout(() => revalidate({ retryCount }), 5000);
        },
        revalidateOnFocus: false,
        revalidateOnReconnect: true,
        refreshInterval: 30000, // 30 seconds
        dedupingInterval: 2000,
        loadingTimeout: 3000,
        errorRetryInterval: 5000,
        errorRetryCount: 3
      }}
    >
      <Component {...pageProps} />
    </SWRConfig>
  );
}

export default MyApp;
```

## ⚡ React Query Integration

### **📦 React Query Setup**

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

```jsx
// pages/_app.js
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export default function MyApp({ Component, pageProps }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 1000 * 60 * 5, // 5 minutes
        cacheTime: 1000 * 60 * 10, // 10 minutes
        retry: 3,
        retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
        refetchOnWindowFocus: false,
        refetchOnReconnect: true
      },
      mutations: {
        retry: 1
      }
    }
  }));

  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### **🎯 React Query Hooks**

```jsx
// hooks/useQueries.js
import { 
  useQuery, 
  useMutation, 
  useQueryClient,
  useInfiniteQuery 
} from '@tanstack/react-query';

// ✅ User queries
export function useUser() {
  return useQuery({
    queryKey: ['user'],
    queryFn: async () => {
      const response = await fetch('/api/user');
      if (!response.ok) throw new Error('Failed to fetch user');
      return response.json();
    },
    staleTime: 1000 * 60 * 5 // 5 minutes
  });
}

// ✅ Posts queries
export function usePosts(filters = {}) {
  return useQuery({
    queryKey: ['posts', filters],
    queryFn: async () => {
      const params = new URLSearchParams(filters);
      const response = await fetch(`/api/posts?${params}`);
      if (!response.ok) throw new Error('Failed to fetch posts');
      return response.json();
    },
    keepPreviousData: true,
    staleTime: 1000 * 60 * 2 // 2 minutes
  });
}

// ✅ Infinite posts query
export function useInfinitePosts() {
  return useInfiniteQuery({
    queryKey: ['posts', 'infinite'],
    queryFn: async ({ pageParam = 1 }) => {
      const response = await fetch(`/api/posts?page=${pageParam}&limit=10`);
      if (!response.ok) throw new Error('Failed to fetch posts');
      return response.json();
    },
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
    getPreviousPageParam: (firstPage, pages) => {
      return pages.length > 1 ? pages.length - 1 : undefined;
    }
  });
}

// ✅ Single post query
export function usePost(id) {
  return useQuery({
    queryKey: ['posts', id],
    queryFn: async () => {
      const response = await fetch(`/api/posts/${id}`);
      if (!response.ok) throw new Error('Failed to fetch post');
      return response.json();
    },
    enabled: !!id,
    staleTime: 1000 * 60 * 10 // 10 minutes
  });
}

// ✅ Create post mutation
export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (postData) => {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(postData)
      });
      if (!response.ok) throw new Error('Failed to create post');
      return response.json();
    },
    onSuccess: (newPost) => {
      // ✅ Invalidate and refetch posts
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      
      // ✅ Add new post to cache
      queryClient.setQueryData(['posts', newPost.id], newPost);
    },
    onError: (error) => {
      console.error('Create post error:', error);
    }
  });
}

// ✅ Update post mutation
export function useUpdatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ id, ...updateData }) => {
      const response = await fetch(`/api/posts/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updateData)
      });
      if (!response.ok) throw new Error('Failed to update post');
      return response.json();
    },
    onMutate: async ({ id, ...updateData }) => {
      // ✅ Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['posts', id] });

      // ✅ Snapshot previous value
      const previousPost = queryClient.getQueryData(['posts', id]);

      // ✅ Optimistically update
      queryClient.setQueryData(['posts', id], old => ({
        ...old,
        ...updateData
      }));

      return { previousPost, id };
    },
    onError: (err, variables, context) => {
      // ✅ Rollback on error
      if (context?.previousPost) {
        queryClient.setQueryData(['posts', context.id], context.previousPost);
      }
    },
    onSettled: (data, error, variables) => {
      // ✅ Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['posts', variables.id] });
    }
  });
}

// ✅ Delete post mutation
export function useDeletePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (id) => {
      const response = await fetch(`/api/posts/${id}`, {
        method: 'DELETE'
      });
      if (!response.ok) throw new Error('Failed to delete post');
      return { id };
    },
    onSuccess: ({ id }) => {
      // ✅ Remove from all posts queries
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      
      // ✅ Remove specific post from cache
      queryClient.removeQueries({ queryKey: ['posts', id] });
    }
  });
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Static Generation**
1. สร้างหน้า blog ด้วย getStaticProps
2. เพิ่ม dynamic routes สำหรับ blog posts
3. ใช้ getStaticPaths ด้วย fallback strategies
4. เพิ่ม ISR (Incremental Static Regeneration)

### **🎯 แบบฝึกหัดที่ 2: Server-Side Rendering**
1. สร้างหน้า dashboard ด้วย getServerSideProps
2. เพิ่ม authentication check
3. Fetch personalized data
4. Handle error states

### **🎯 แบบฝึกหัดที่ 3: Client-Side Fetching**
1. สร้าง search page ด้วย useEffect
2. เพิ่ม debounced search
3. Implement infinite scrolling
4. Add loading และ error states

### **🎯 แบบฝึกหัดที่ 4: SWR/React Query**
1. Setup SWR หรือ React Query
2. สร้าง custom hooks
3. Implement optimistic updates
4. Add cache management

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 5: Components and Styling](./05-components-styling.md)

**บทต่อไป**: [บทที่ 7: API Routes](./07-api-routes.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Data Fetching](https://nextjs.org/docs/basic-features/data-fetching)
- [SWR Documentation](https://swr.vercel.app/)
- [React Query Documentation](https://tanstack.com/query/latest)
