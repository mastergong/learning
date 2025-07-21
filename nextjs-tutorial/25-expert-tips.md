# บทที่ 25: Expert Tips and Best Practices

## 🎯 จุดประสงค์ของบทเรียน
- รวมรวม expert tips และ advanced techniques
- เรียนรู้ production-ready best practices
- ทำความเข้าใจ common pitfalls และวิธีหลีกเลี่ยง
- เตรียมพร้อมสำหรับการพัฒนา Next.js ในระดับมืออาชีพ

## 🚀 Advanced Development Patterns

### **🎨 Component Design Patterns**

```tsx
// ✅ Compound Component Pattern
import { createContext, useContext, useState, ReactNode } from 'react';

// Context for component state
interface AccordionContextType {
  activeItems: Set<string>;
  toggleItem: (id: string) => void;
  allowMultiple: boolean;
}

const AccordionContext = createContext<AccordionContextType | null>(null);

// Main Accordion component
interface AccordionProps {
  children: ReactNode;
  allowMultiple?: boolean;
  defaultActiveItems?: string[];
}

export function Accordion({ 
  children, 
  allowMultiple = false, 
  defaultActiveItems = [] 
}: AccordionProps) {
  const [activeItems, setActiveItems] = useState<Set<string>>(
    new Set(defaultActiveItems)
  );

  const toggleItem = (id: string) => {
    setActiveItems(prev => {
      const newSet = new Set(prev);
      
      if (newSet.has(id)) {
        newSet.delete(id);
      } else {
        if (!allowMultiple) {
          newSet.clear();
        }
        newSet.add(id);
      }
      
      return newSet;
    });
  };

  return (
    <AccordionContext.Provider 
      value={{ activeItems, toggleItem, allowMultiple }}
    >
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

// AccordionItem component
interface AccordionItemProps {
  id: string;
  children: ReactNode;
  className?: string;
}

function AccordionItem({ id, children, className }: AccordionItemProps) {
  return (
    <div className={`accordion-item ${className || ''}`} data-item-id={id}>
      {children}
    </div>
  );
}

// AccordionTrigger component
interface AccordionTriggerProps {
  children: ReactNode;
  className?: string;
}

function AccordionTrigger({ children, className }: AccordionTriggerProps) {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('AccordionTrigger must be used within Accordion');
  
  const { toggleItem } = context;
  
  // Get item ID from parent AccordionItem
  const getItemId = (element: HTMLElement): string | null => {
    const item = element.closest('[data-item-id]');
    return item?.getAttribute('data-item-id') || null;
  };
  
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    const itemId = getItemId(event.currentTarget);
    if (itemId) toggleItem(itemId);
  };

  return (
    <button
      className={`accordion-trigger ${className || ''}`}
      onClick={handleClick}
      type="button"
    >
      {children}
    </button>
  );
}

// AccordionContent component
interface AccordionContentProps {
  children: ReactNode;
  className?: string;
}

function AccordionContent({ children, className }: AccordionContentProps) {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('AccordionContent must be used within Accordion');
  
  const { activeItems } = context;
  
  // Get item ID from parent AccordionItem
  const [itemId, setItemId] = useState<string | null>(null);
  
  useEffect(() => {
    const element = document.currentScript?.closest('[data-item-id]');
    const id = element?.getAttribute('data-item-id');
    setItemId(id || null);
  }, []);

  const isActive = itemId ? activeItems.has(itemId) : false;

  return (
    <div 
      className={`accordion-content ${className || ''} ${
        isActive ? 'accordion-content--active' : ''
      }`}
      style={{
        height: isActive ? 'auto' : '0',
        overflow: 'hidden',
        transition: 'height 0.2s ease-in-out'
      }}
    >
      {children}
    </div>
  );
}

// Attach sub-components to main component
Accordion.Item = AccordionItem;
Accordion.Trigger = AccordionTrigger;
Accordion.Content = AccordionContent;

// Usage:
export function AccordionExample() {
  return (
    <Accordion allowMultiple defaultActiveItems={['item1']}>
      <Accordion.Item id="item1">
        <Accordion.Trigger>
          <h3>What is Next.js?</h3>
        </Accordion.Trigger>
        <Accordion.Content>
          <p>Next.js is a React framework for production...</p>
        </Accordion.Content>
      </Accordion.Item>
      
      <Accordion.Item id="item2">
        <Accordion.Trigger>
          <h3>How does SSR work?</h3>
        </Accordion.Trigger>
        <Accordion.Content>
          <p>Server-Side Rendering pre-renders pages...</p>
        </Accordion.Content>
      </Accordion.Item>
    </Accordion>
  );
}

// ✅ Render Props Pattern
interface RenderPropsDataFetcherProps<T> {
  url: string;
  children: (data: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
  dependencies?: any[];
}

export function DataFetcher<T>({ 
  url, 
  children, 
  dependencies = [] 
}: RenderPropsDataFetcherProps<T>) {
  const [state, setState] = useState<{
    data: T | null;
    loading: boolean;
    error: Error | null;
  }>({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ 
        data: null, 
        loading: false, 
        error: error instanceof Error ? error : new Error('Unknown error') 
      });
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData, ...dependencies]);

  return <>{children({ ...state, refetch: fetchData })}</>;
}

// Usage:
export function UserProfile({ userId }: { userId: string }) {
  return (
    <DataFetcher<User> url={`/api/users/${userId}`} dependencies={[userId]}>
      {({ data: user, loading, error, refetch }) => {
        if (loading) return <UserSkeleton />;
        if (error) return <ErrorMessage error={error} onRetry={refetch} />;
        if (!user) return <div>User not found</div>;
        
        return <UserCard user={user} />;
      }}
    </DataFetcher>
  );
}

// ✅ Higher-Order Component for Error Boundaries
interface WithErrorBoundaryOptions {
  fallback?: React.ComponentType<{ error: Error; resetError: () => void }>;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
  resetKeys?: Array<string | number>;
}

export function withErrorBoundary<P extends object>(
  Component: React.ComponentType<P>,
  options: WithErrorBoundaryOptions = {}
) {
  const { 
    fallback: Fallback = DefaultErrorFallback, 
    onError,
    resetKeys = []
  } = options;

  return React.forwardRef<any, P>((props, ref) => {
    return (
      <ErrorBoundary
        FallbackComponent={Fallback}
        onError={onError}
        resetKeys={resetKeys}
      >
        <Component {...props} ref={ref} />
      </ErrorBoundary>
    );
  });
}

// Custom hook for error boundaries
export function useErrorHandler() {
  const [error, setError] = useState<Error | null>(null);

  const resetError = useCallback(() => {
    setError(null);
  }, []);

  const handleError = useCallback((error: Error) => {
    setError(error);
  }, []);

  useEffect(() => {
    if (error) {
      throw error;
    }
  }, [error]);

  return { handleError, resetError };
}

// Usage:
const SafeUserProfile = withErrorBoundary(UserProfile, {
  fallback: ({ error, resetError }) => (
    <div className="error-container">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={resetError}>Try again</button>
    </div>
  ),
  onError: (error, errorInfo) => {
    console.error('User profile error:', error, errorInfo);
    // Send to error tracking service
  }
});
```

### **🎯 Advanced Hooks Patterns**

```tsx
// ✅ Custom Hook for Optimistic Updates
interface UseOptimisticOptions<T> {
  mutationFn: (variables: any) => Promise<T>;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  rollbackOnError?: boolean;
}

export function useOptimistic<T>(
  queryKey: string,
  options: UseOptimisticOptions<T>
) {
  const [optimisticData, setOptimisticData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const mutate = useCallback(async (
    variables: any,
    optimisticUpdate?: (currentData: T | null) => T
  ) => {
    setIsLoading(true);
    setError(null);

    // Apply optimistic update
    let previousData: T | null = null;
    if (optimisticUpdate) {
      previousData = optimisticData;
      setOptimisticData(optimisticUpdate(optimisticData));
    }

    try {
      const result = await options.mutationFn(variables);
      setOptimisticData(result);
      options.onSuccess?.(result);
      return result;
    } catch (err) {
      const error = err instanceof Error ? err : new Error('Unknown error');
      setError(error);
      
      // Rollback optimistic update on error
      if (options.rollbackOnError && optimisticUpdate) {
        setOptimisticData(previousData);
      }
      
      options.onError?.(error);
      throw error;
    } finally {
      setIsLoading(false);
    }
  }, [optimisticData, options]);

  return {
    data: optimisticData,
    isLoading,
    error,
    mutate
  };
}

// Usage:
export function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const { mutate: addTodo, isLoading } = useOptimistic('todos', {
    mutationFn: async (newTodo: Omit<Todo, 'id'>) => {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newTodo)
      });
      return response.json();
    },
    rollbackOnError: true
  });

  const handleAddTodo = async (text: string) => {
    const tempId = `temp-${Date.now()}`;
    
    await addTodo(
      { text, completed: false },
      (currentTodos) => [
        ...(currentTodos || []),
        { id: tempId, text, completed: false }
      ]
    );
  };

  return (
    <div>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
      <TodoForm onSubmit={handleAddTodo} disabled={isLoading} />
    </div>
  );
}

// ✅ Advanced State Management Hook
interface UseStateMachineOptions<TState, TEvent> {
  initialState: TState;
  states: Record<string, {
    on: Record<string, TState | ((context: any) => TState)>;
    entry?: (context: any) => void;
    exit?: (context: any) => void;
  }>;
  context?: any;
}

export function useStateMachine<TState extends string, TEvent extends string>(
  options: UseStateMachineOptions<TState, TEvent>
) {
  const [state, setState] = useState(options.initialState);
  const [context, setContext] = useState(options.context || {});

  const send = useCallback((event: TEvent, payload?: any) => {
    const currentStateConfig = options.states[state];
    const nextState = currentStateConfig?.on[event];

    if (!nextState) {
      console.warn(`No transition defined for event "${event}" in state "${state}"`);
      return;
    }

    // Execute exit action
    currentStateConfig.exit?.(context);

    // Update state
    const newState = typeof nextState === 'function' 
      ? nextState({ ...context, ...payload })
      : nextState;

    setState(newState);

    // Update context
    if (payload) {
      setContext(prev => ({ ...prev, ...payload }));
    }

    // Execute entry action
    const newStateConfig = options.states[newState];
    newStateConfig?.entry?.({ ...context, ...payload });
  }, [state, context, options.states]);

  const can = useCallback((event: TEvent): boolean => {
    const currentStateConfig = options.states[state];
    return currentStateConfig?.on[event] !== undefined;
  }, [state, options.states]);

  return {
    state,
    context,
    send,
    can
  };
}

// Usage:
type FormState = 'idle' | 'validating' | 'submitting' | 'success' | 'error';
type FormEvent = 'VALIDATE' | 'SUBMIT' | 'SUCCESS' | 'ERROR' | 'RESET';

export function useFormStateMachine() {
  return useStateMachine<FormState, FormEvent>({
    initialState: 'idle',
    states: {
      idle: {
        on: {
          VALIDATE: 'validating',
          SUBMIT: 'submitting'
        }
      },
      validating: {
        on: {
          SUBMIT: 'submitting',
          ERROR: 'idle'
        },
        entry: (context) => {
          console.log('Validating form...');
        }
      },
      submitting: {
        on: {
          SUCCESS: 'success',
          ERROR: 'error'
        },
        entry: (context) => {
          console.log('Submitting form...');
        }
      },
      success: {
        on: {
          RESET: 'idle'
        },
        entry: (context) => {
          console.log('Form submitted successfully!');
        }
      },
      error: {
        on: {
          RESET: 'idle',
          SUBMIT: 'submitting'
        }
      }
    }
  });
}

// ✅ Intersection Observer Hook with Advanced Options
interface UseIntersectionObserverOptions {
  threshold?: number | number[];
  rootMargin?: string;
  triggerOnce?: boolean;
  skip?: boolean;
  onChange?: (entry: IntersectionObserverEntry) => void;
}

export function useIntersectionObserver(
  options: UseIntersectionObserverOptions = {}
) {
  const {
    threshold = 0,
    rootMargin = '0px',
    triggerOnce = true,
    skip = false,
    onChange
  } = options;

  const [entry, setEntry] = useState<IntersectionObserverEntry | null>(null);
  const elementRef = useRef<HTMLElement | null>(null);
  const observerRef = useRef<IntersectionObserver | null>(null);

  const frozen = triggerOnce && entry?.isIntersecting;

  useEffect(() => {
    const element = elementRef.current;
    
    if (!element || skip || frozen) return;

    if (!observerRef.current) {
      observerRef.current = new IntersectionObserver(
        (entries) => {
          const [entry] = entries;
          setEntry(entry);
          onChange?.(entry);
        },
        { threshold, rootMargin }
      );
    }

    observerRef.current.observe(element);

    return () => {
      if (observerRef.current && element) {
        observerRef.current.unobserve(element);
      }
    };
  }, [threshold, rootMargin, skip, frozen, onChange]);

  // Cleanup observer on unmount
  useEffect(() => {
    return () => {
      if (observerRef.current) {
        observerRef.current.disconnect();
      }
    };
  }, []);

  return {
    ref: elementRef,
    entry,
    isIntersecting: !!entry?.isIntersecting,
    intersectionRatio: entry?.intersectionRatio ?? 0
  };
}

// Advanced usage with multiple elements
export function useMultipleIntersectionObserver(
  options: UseIntersectionObserverOptions = {}
) {
  const [entries, setEntries] = useState<Map<Element, IntersectionObserverEntry>>(new Map());
  const observerRef = useRef<IntersectionObserver | null>(null);
  const elementsRef = useRef<Set<Element>>(new Set());

  const observe = useCallback((element: Element) => {
    if (!observerRef.current) {
      observerRef.current = new IntersectionObserver(
        (observerEntries) => {
          setEntries(prev => {
            const newEntries = new Map(prev);
            observerEntries.forEach(entry => {
              newEntries.set(entry.target, entry);
              options.onChange?.(entry);
            });
            return newEntries;
          });
        },
        {
          threshold: options.threshold,
          rootMargin: options.rootMargin
        }
      );
    }

    observerRef.current.observe(element);
    elementsRef.current.add(element);
  }, [options]);

  const unobserve = useCallback((element: Element) => {
    if (observerRef.current) {
      observerRef.current.unobserve(element);
    }
    elementsRef.current.delete(element);
    setEntries(prev => {
      const newEntries = new Map(prev);
      newEntries.delete(element);
      return newEntries;
    });
  }, []);

  useEffect(() => {
    return () => {
      if (observerRef.current) {
        observerRef.current.disconnect();
      }
    };
  }, []);

  return {
    observe,
    unobserve,
    entries
  };
}
```

### **⚡ Performance Best Practices**

```typescript
// ✅ Advanced Memoization Patterns
import { useMemo, useCallback, memo, forwardRef } from 'react';

// Memoization with deep comparison
function useDeepMemo<T>(factory: () => T, deps: any[]): T {
  const ref = useRef<{ deps: any[]; value: T }>();

  if (!ref.current || !areEqual(ref.current.deps, deps)) {
    ref.current = { deps, value: factory() };
  }

  return ref.current.value;
}

function areEqual(a: any[], b: any[]): boolean {
  if (a.length !== b.length) return false;
  
  for (let i = 0; i < a.length; i++) {
    if (!deepEqual(a[i], b[i])) return false;
  }
  
  return true;
}

function deepEqual(a: any, b: any): boolean {
  if (a === b) return true;
  
  if (a == null || b == null) return false;
  if (typeof a !== typeof b) return false;
  
  if (typeof a === 'object') {
    const keysA = Object.keys(a);
    const keysB = Object.keys(b);
    
    if (keysA.length !== keysB.length) return false;
    
    for (const key of keysA) {
      if (!keysB.includes(key) || !deepEqual(a[key], b[key])) {
        return false;
      }
    }
    
    return true;
  }
  
  return false;
}

// Advanced React.memo with custom comparison
interface ExpensiveComponentProps {
  data: ComplexData;
  config: ComponentConfig;
  onAction: (action: Action) => void;
}

const ExpensiveComponent = memo<ExpensiveComponentProps>(
  ({ data, config, onAction }) => {
    // Expensive computations
    const processedData = useMemo(() => {
      return processComplexData(data, config);
    }, [data, config]);

    const handleAction = useCallback((action: Action) => {
      // Add additional logic if needed
      console.log('Action triggered:', action);
      onAction(action);
    }, [onAction]);

    return (
      <div>
        {processedData.map(item => (
          <Item 
            key={item.id} 
            item={item} 
            onAction={handleAction}
          />
        ))}
      </div>
    );
  },
  // Custom comparison function
  (prevProps, nextProps) => {
    // Only re-render if data or config actually changed
    return (
      deepEqual(prevProps.data, nextProps.data) &&
      deepEqual(prevProps.config, nextProps.config) &&
      prevProps.onAction === nextProps.onAction
    );
  }
);

// ✅ Virtualization for Large Lists
interface VirtualizedListProps<T> {
  items: T[];
  itemHeight: number;
  containerHeight: number;
  renderItem: (item: T, index: number) => React.ReactNode;
  overscan?: number;
}

export function VirtualizedList<T>({
  items,
  itemHeight,
  containerHeight,
  renderItem,
  overscan = 5
}: VirtualizedListProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef<HTMLDivElement>(null);

  const visibleCount = Math.ceil(containerHeight / itemHeight);
  const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
  const endIndex = Math.min(
    items.length - 1,
    startIndex + visibleCount + overscan * 2
  );

  const visibleItems = useMemo(() => {
    return items.slice(startIndex, endIndex + 1).map((item, index) => ({
      item,
      index: startIndex + index
    }));
  }, [items, startIndex, endIndex]);

  const totalHeight = items.length * itemHeight;
  const offsetY = startIndex * itemHeight;

  const handleScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  }, []);

  return (
    <div
      ref={containerRef}
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map(({ item, index }) => (
            <div
              key={index}
              style={{ height: itemHeight }}
            >
              {renderItem(item, index)}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// ✅ Debounced State Hook
export function useDebouncedState<T>(
  initialValue: T,
  delay: number = 300
): [T, T, (value: T) => void] {
  const [value, setValue] = useState<T>(initialValue);
  const [debouncedValue, setDebouncedValue] = useState<T>(initialValue);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return [value, debouncedValue, setValue];
}

// Usage in search component
export function SearchComponent() {
  const [query, debouncedQuery, setQuery] = useDebouncedState('', 500);
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!debouncedQuery) {
      setResults([]);
      return;
    }

    setLoading(true);
    searchAPI(debouncedQuery)
      .then(setResults)
      .finally(() => setLoading(false));
  }, [debouncedQuery]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {loading && <div>Searching...</div>}
      <SearchResults results={results} />
    </div>
  );
}

// ✅ Resource Preloading Hook
export function useResourcePreloader() {
  const preloadedResources = useRef<Set<string>>(new Set());

  const preloadImage = useCallback((src: string): Promise<void> => {
    if (preloadedResources.current.has(src)) {
      return Promise.resolve();
    }

    return new Promise((resolve, reject) => {
      const img = new Image();
      img.onload = () => {
        preloadedResources.current.add(src);
        resolve();
      };
      img.onerror = reject;
      img.src = src;
    });
  }, []);

  const preloadScript = useCallback((src: string): Promise<void> => {
    if (preloadedResources.current.has(src)) {
      return Promise.resolve();
    }

    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.onload = () => {
        preloadedResources.current.add(src);
        resolve();
      };
      script.onerror = reject;
      script.src = src;
      document.head.appendChild(script);
    });
  }, []);

  const preloadCSS = useCallback((href: string): Promise<void> => {
    if (preloadedResources.current.has(href)) {
      return Promise.resolve();
    }

    return new Promise((resolve, reject) => {
      const link = document.createElement('link');
      link.rel = 'stylesheet';
      link.onload = () => {
        preloadedResources.current.add(href);
        resolve();
      };
      link.onerror = reject;
      link.href = href;
      document.head.appendChild(link);
    });
  }, []);

  return {
    preloadImage,
    preloadScript,
    preloadCSS,
    isPreloaded: (resource: string) => preloadedResources.current.has(resource)
  };
}
```

## 🛡️ Security Best Practices

### **🔒 Advanced Security Measures**

```typescript
// ✅ Content Security Policy Helper
export function generateCSP(config: {
  defaultSrc?: string[];
  scriptSrc?: string[];
  styleSrc?: string[];
  imgSrc?: string[];
  connectSrc?: string[];
  fontSrc?: string[];
  objectSrc?: string[];
  mediaSrc?: string[];
  frameSrc?: string[];
}) {
  const directives = Object.entries(config)
    .map(([key, sources]) => {
      const directive = key.replace(/([A-Z])/g, '-$1').toLowerCase();
      return `${directive} ${sources.join(' ')}`;
    })
    .join('; ');

  return directives;
}

// Usage in Next.js
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: generateCSP({
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", "'unsafe-eval'", "https://cdn.example.com"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    })
  },
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  {
    key: 'X-XSS-Protection',
    value: '1; mode=block'
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'origin-when-cross-origin'
  }
];

// ✅ Input Sanitization and Validation
import DOMPurify from 'isomorphic-dompurify';
import { z } from 'zod';

// Comprehensive validation schemas
export const schemas = {
  email: z.string().email('Invalid email format'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[0-9]/, 'Password must contain number')
    .regex(/[^A-Za-z0-9]/, 'Password must contain special character'),
  
  url: z.string().url('Invalid URL format'),
  
  safeString: z.string()
    .max(1000, 'String too long')
    .refine(
      (val) => !/<script|javascript:|data:|vbscript:/i.test(val),
      'Potentially dangerous content detected'
    ),
  
  html: z.string()
    .max(50000, 'HTML content too long')
    .transform((val) => DOMPurify.sanitize(val, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'ol', 'ul', 'li', 'h1', 'h2', 'h3'],
      ALLOWED_ATTR: ['class', 'style']
    })),

  userId: z.string().uuid('Invalid user ID format'),
  
  pagination: z.object({
    page: z.number().min(1).max(1000),
    limit: z.number().min(1).max(100)
  })
};

// Advanced input sanitization
export class InputSanitizer {
  static sanitizeString(input: string): string {
    return input
      .trim()
      .replace(/[\0\x08\x09\x1a\n\r"'\\\%]/g, (char) => {
        switch (char) {
          case '\0': return '\\0';
          case '\x08': return '\\b';
          case '\x09': return '\\t';
          case '\x1a': return '\\z';
          case '\n': return '\\n';
          case '\r': return '\\r';
          case '"':
          case "'":
          case '\\':
          case '%': return '\\' + char;
          default: return char;
        }
      });
  }

  static sanitizeHtml(html: string, options: {
    allowedTags?: string[];
    allowedAttributes?: string[];
    stripUnknown?: boolean;
  } = {}): string {
    const {
      allowedTags = ['p', 'br', 'strong', 'em', 'u'],
      allowedAttributes = ['class'],
      stripUnknown = true
    } = options;

    return DOMPurify.sanitize(html, {
      ALLOWED_TAGS: allowedTags,
      ALLOWED_ATTR: allowedAttributes,
      KEEP_CONTENT: !stripUnknown
    });
  }

  static validateAndSanitize<T>(
    data: any,
    schema: z.ZodSchema<T>
  ): { success: true; data: T } | { success: false; errors: string[] } {
    try {
      const validatedData = schema.parse(data);
      return { success: true, data: validatedData };
    } catch (error) {
      if (error instanceof z.ZodError) {
        return {
          success: false,
          errors: error.errors.map(err => `${err.path.join('.')}: ${err.message}`)
        };
      }
      return { success: false, errors: ['Validation failed'] };
    }
  }
}

// ✅ Rate Limiting Implementation
interface RateLimitConfig {
  windowMs: number;
  maxRequests: number;
  keyGenerator?: (req: any) => string;
  skipIf?: (req: any) => boolean;
  onLimitReached?: (req: any) => void;
}

export class RateLimiter {
  private windows = new Map<string, { count: number; resetTime: number }>();
  private config: RateLimitConfig;

  constructor(config: RateLimitConfig) {
    this.config = config;
    
    // Cleanup expired windows every minute
    setInterval(() => this.cleanup(), 60000);
  }

  checkLimit(req: any): { allowed: boolean; remaining: number; resetTime: number } {
    if (this.config.skipIf?.(req)) {
      return { allowed: true, remaining: this.config.maxRequests, resetTime: 0 };
    }

    const key = this.config.keyGenerator?.(req) || this.getDefaultKey(req);
    const now = Date.now();
    const window = this.windows.get(key);

    if (!window || now > window.resetTime) {
      // New window
      const newWindow = {
        count: 1,
        resetTime: now + this.config.windowMs
      };
      this.windows.set(key, newWindow);
      
      return {
        allowed: true,
        remaining: this.config.maxRequests - 1,
        resetTime: newWindow.resetTime
      };
    }

    if (window.count >= this.config.maxRequests) {
      this.config.onLimitReached?.(req);
      return {
        allowed: false,
        remaining: 0,
        resetTime: window.resetTime
      };
    }

    window.count++;
    this.windows.set(key, window);

    return {
      allowed: true,
      remaining: this.config.maxRequests - window.count,
      resetTime: window.resetTime
    };
  }

  private getDefaultKey(req: any): string {
    return req.ip || req.connection?.remoteAddress || 'unknown';
  }

  private cleanup(): void {
    const now = Date.now();
    for (const [key, window] of this.windows.entries()) {
      if (now > window.resetTime) {
        this.windows.delete(key);
      }
    }
  }
}

// Middleware for API routes
export function withRateLimit(config: RateLimitConfig) {
  const rateLimiter = new RateLimiter(config);

  return function middleware(req: NextApiRequest, res: NextApiResponse, next: Function) {
    const result = rateLimiter.checkLimit(req);

    res.setHeader('X-RateLimit-Limit', config.maxRequests);
    res.setHeader('X-RateLimit-Remaining', result.remaining);
    res.setHeader('X-RateLimit-Reset', result.resetTime);

    if (!result.allowed) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil((result.resetTime - Date.now()) / 1000)
      });
    }

    next();
  };
}

// Usage:
export default withRateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  maxRequests: 100,
  keyGenerator: (req) => req.headers['x-user-id'] || req.ip,
  skipIf: (req) => req.headers.authorization?.includes('admin-token')
})(async function handler(req: NextApiRequest, res: NextApiResponse) {
  // Your API logic here
});
```

## 🔍 Debugging and Monitoring

### **🐛 Advanced Debugging Techniques**

```typescript
// ✅ Advanced Logging System
enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3
}

interface LogEntry {
  timestamp: Date;
  level: LogLevel;
  message: string;
  data?: any;
  userId?: string;
  sessionId?: string;
  requestId?: string;
  stack?: string;
}

export class Logger {
  private logLevel: LogLevel;
  private transports: LogTransport[] = [];

  constructor(logLevel: LogLevel = LogLevel.INFO) {
    this.logLevel = logLevel;
  }

  addTransport(transport: LogTransport): void {
    this.transports.push(transport);
  }

  private log(level: LogLevel, message: string, data?: any): void {
    if (level < this.logLevel) return;

    const entry: LogEntry = {
      timestamp: new Date(),
      level,
      message,
      data,
      userId: this.getCurrentUserId(),
      sessionId: this.getCurrentSessionId(),
      requestId: this.getCurrentRequestId(),
      stack: level === LogLevel.ERROR ? new Error().stack : undefined
    };

    this.transports.forEach(transport => {
      transport.log(entry);
    });
  }

  debug(message: string, data?: any): void {
    this.log(LogLevel.DEBUG, message, data);
  }

  info(message: string, data?: any): void {
    this.log(LogLevel.INFO, message, data);
  }

  warn(message: string, data?: any): void {
    this.log(LogLevel.WARN, message, data);
  }

  error(message: string, error?: Error | any): void {
    this.log(LogLevel.ERROR, message, {
      error: error instanceof Error ? {
        name: error.name,
        message: error.message,
        stack: error.stack
      } : error
    });
  }

  private getCurrentUserId(): string | undefined {
    // Implementation depends on your auth system
    return undefined;
  }

  private getCurrentSessionId(): string | undefined {
    // Implementation depends on your session management
    return undefined;
  }

  private getCurrentRequestId(): string | undefined {
    // Implementation depends on your request tracking
    return undefined;
  }
}

// Log transports
interface LogTransport {
  log(entry: LogEntry): void;
}

export class ConsoleTransport implements LogTransport {
  log(entry: LogEntry): void {
    const { timestamp, level, message, data } = entry;
    const levelName = LogLevel[level];
    const time = timestamp.toISOString();
    
    const logMessage = `[${time}] ${levelName}: ${message}`;
    
    switch (level) {
      case LogLevel.DEBUG:
        console.debug(logMessage, data);
        break;
      case LogLevel.INFO:
        console.info(logMessage, data);
        break;
      case LogLevel.WARN:
        console.warn(logMessage, data);
        break;
      case LogLevel.ERROR:
        console.error(logMessage, data);
        break;
    }
  }
}

export class FileTransport implements LogTransport {
  constructor(private filename: string) {}

  log(entry: LogEntry): void {
    const logLine = JSON.stringify(entry) + '\n';
    
    // In Node.js environment
    if (typeof window === 'undefined') {
      const fs = require('fs').promises;
      fs.appendFile(this.filename, logLine).catch(console.error);
    }
  }
}

export class RemoteTransport implements LogTransport {
  private queue: LogEntry[] = [];
  private isFlushingQueue = false;

  constructor(
    private endpoint: string,
    private batchSize: number = 10,
    private flushInterval: number = 5000
  ) {
    setInterval(() => this.flush(), flushInterval);
  }

  log(entry: LogEntry): void {
    this.queue.push(entry);
    
    if (this.queue.length >= this.batchSize) {
      this.flush();
    }
  }

  private async flush(): Promise<void> {
    if (this.isFlushingQueue || this.queue.length === 0) return;

    this.isFlushingQueue = true;
    const batch = this.queue.splice(0, this.batchSize);

    try {
      await fetch(this.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ logs: batch })
      });
    } catch (error) {
      // Re-queue failed logs
      this.queue.unshift(...batch);
      console.error('Failed to send logs to remote:', error);
    } finally {
      this.isFlushingQueue = false;
    }
  }
}

// Global logger instance
export const logger = new Logger(LogLevel.INFO);
logger.addTransport(new ConsoleTransport());

if (process.env.NODE_ENV === 'production') {
  logger.addTransport(new FileTransport('/var/log/app.log'));
  logger.addTransport(new RemoteTransport('https://api.example.com/logs'));
}

// ✅ Performance Profiler
export class PerformanceProfiler {
  private profiles = new Map<string, {
    startTime: number;
    endTime?: number;
    duration?: number;
    metadata?: any;
  }>();

  start(profileId: string, metadata?: any): void {
    this.profiles.set(profileId, {
      startTime: performance.now(),
      metadata
    });
  }

  end(profileId: string): number | null {
    const profile = this.profiles.get(profileId);
    if (!profile) return null;

    const endTime = performance.now();
    const duration = endTime - profile.startTime;

    profile.endTime = endTime;
    profile.duration = duration;

    logger.debug(`Profile: ${profileId}`, {
      duration: `${duration.toFixed(2)}ms`,
      metadata: profile.metadata
    });

    return duration;
  }

  measure<T>(profileId: string, fn: () => T, metadata?: any): T {
    this.start(profileId, metadata);
    try {
      const result = fn();
      this.end(profileId);
      return result;
    } catch (error) {
      this.end(profileId);
      throw error;
    }
  }

  async measureAsync<T>(
    profileId: string, 
    fn: () => Promise<T>, 
    metadata?: any
  ): Promise<T> {
    this.start(profileId, metadata);
    try {
      const result = await fn();
      this.end(profileId);
      return result;
    } catch (error) {
      this.end(profileId);
      throw error;
    }
  }

  getProfile(profileId: string) {
    return this.profiles.get(profileId);
  }

  getAllProfiles() {
    return Array.from(this.profiles.entries()).map(([id, profile]) => ({
      id,
      ...profile
    }));
  }

  clear(): void {
    this.profiles.clear();
  }
}

export const profiler = new PerformanceProfiler();

// Decorator for automatic profiling
export function profile(profileId?: string) {
  return function (
    target: any,
    propertyName: string,
    descriptor: PropertyDescriptor
  ) {
    const method = descriptor.value;
    const id = profileId || `${target.constructor.name}.${propertyName}`;

    descriptor.value = function (...args: any[]) {
      return profiler.measure(id, () => method.apply(this, args));
    };

    return descriptor;
  };
}

// Usage:
export class UserService {
  @profile('UserService.createUser')
  async createUser(userData: CreateUserData): Promise<User> {
    // Method implementation
    return {} as User;
  }

  @profile()
  async fetchUsers(): Promise<User[]> {
    // Method implementation
    return [];
  }
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Advanced Component Patterns**
1. สร้าง compound component ที่ซับซ้อน
2. เพิ่ม render props pattern
3. ตั้งค่า higher-order components
4. ทดสอบ component composition

### **🎯 แบบฝึกหัดที่ 2: Performance Optimization**
1. ใช้ advanced memoization techniques
2. เพิ่ม virtualization สำหรับ large lists
3. ตั้งค่า resource preloading
4. วัดและปรับปรุง performance metrics

### **🎯 แบบฝึกหัดที่ 3: Security Implementation**
1. ตั้งค่า comprehensive input validation
2. เพิ่ม rate limiting และ security headers
3. สร้าง content security policy
4. ทดสอบ security measures

### **🎯 แบบฝึกหัดที่ 4: Debugging and Monitoring**
1. ตั้งค่า advanced logging system
2. เพิ่ม performance profiling
3. สร้าง error tracking system
4. ทดสอบ monitoring และ alerting

## 🎉 สรุปการเรียนรู้

### **🏆 ความรู้ที่ได้รับ**
1. **Architecture Patterns**: Clean Architecture, CQRS, Event Sourcing
2. **Performance Optimization**: Bundle splitting, caching, image optimization
3. **Security Best Practices**: Input validation, rate limiting, CSP
4. **Testing Strategies**: Unit, integration, E2E testing
5. **Deployment & DevOps**: Docker, CI/CD, monitoring
6. **Real-world Applications**: E-commerce platform, complex state management

### **🛠️ ทักษะที่พัฒนา**
- การออกแบบ scalable applications
- การใช้ advanced React และ Next.js patterns
- การปรับปรุง performance และ user experience
- การจัดการ security และ data protection
- การ deploy และ maintain production applications

### **📈 เส้นทางการพัฒนาต่อ**
1. **Contribute to Open Source**: เข้าร่วม Next.js community
2. **Stay Updated**: ติดตาม Next.js releases และ React updates
3. **Explore New Technologies**: GraphQL, Edge Computing, Web3
4. **Build Complex Projects**: Micro-frontends, Multi-tenant applications
5. **Share Knowledge**: เขียน blog, สร้าง tutorials, พูดในงาน conference

### **🎯 Next Steps**
- สร้างโปรเจกต์ production-ready ของคุณเอง
- เรียนรู้เทคโนโลยีเพิ่มเติม: TypeScript advanced patterns, GraphQL
- ศึกษา modern development practices: Micro-frontends, Serverless
- พัฒนาทักษะ DevOps และ Cloud platforms

---

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 24: Performance Optimization](./24-performance-optimization.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 แหล่งข้อมูลเพิ่มเติม

- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Web.dev Performance](https://web.dev/performance/)
- [MDN Web Docs](https://developer.mozilla.org/)

**ขอบคุณที่เรียนรู้ Next.js ไปกับเรา! 🚀**

คุณพร้อมแล้วที่จะสร้างแอปพลิเคชันระดับมืออาชีพด้วย Next.js!
