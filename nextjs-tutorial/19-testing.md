# บทที่ 19: Testing

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Testing strategies ใน Next.js
- ทำความเข้าใจ Unit, Integration, และ E2E testing
- เรียนรู้ Testing tools และ best practices
- เข้าใจ Test-driven development (TDD)

## 🧪 Testing Overview

### **⚡ Testing Pyramid**

```
     🔺 E2E Tests
    ────────────
   🔺🔺 Integration Tests  
  ────────────────────
 🔺🔺🔺 Unit Tests
────────────────────────
```

## 🔧 Unit Testing

### **⚗️ Jest & React Testing Library Setup**

```bash
# Install testing dependencies
npm install --save-dev jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event

# TypeScript support
npm install --save-dev @types/jest
```

```javascript
// jest.config.js
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  // Provide the path to your Next.js app to load next.config.js and .env files
  dir: './',
});

// Add any custom config to be passed to Jest
const customJestConfig = {
  // Add more setup options before each test is run
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  
  // If using TypeScript with a baseUrl set to the root directory
  moduleDirectories: ['node_modules', '<rootDir>/'],
  
  // Handle module aliases
  moduleNameMapping: {
    '^@/components/(.*)$': '<rootDir>/components/$1',
    '^@/lib/(.*)$': '<rootDir>/lib/$1',
    '^@/pages/(.*)$': '<rootDir>/pages/$1',
    '^@/styles/(.*)$': '<rootDir>/styles/$1',
    '^@/utils/(.*)$': '<rootDir>/utils/$1',
  },
  
  // Coverage settings
  collectCoverageFrom: [
    'components/**/*.{js,jsx,ts,tsx}',
    'lib/**/*.{js,jsx,ts,tsx}',
    'pages/**/*.{js,jsx,ts,tsx}',
    'utils/**/*.{js,jsx,ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
    '!**/.next/**',
  ],
  
  // Test environment
  testEnvironment: 'jest-environment-jsdom',
  
  // Transform settings
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': ['babel-jest', { presets: ['next/babel'] }],
  },
  
  // Ignore patterns
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
  
  // Setup files
  setupFiles: ['<rootDir>/jest.setup.js'],
};

// createJestConfig is exported this way to ensure that next/jest can load the Next.js config
module.exports = createJestConfig(customJestConfig);

// jest.setup.js
import '@testing-library/jest-dom';

// Mock next/router
jest.mock('next/router', () => ({
  useRouter() {
    return {
      route: '/',
      pathname: '/',
      query: '',
      asPath: '/',
      push: jest.fn(),
      pop: jest.fn(),
      reload: jest.fn(),
      back: jest.fn(),
      prefetch: jest.fn().mockResolvedValue(undefined),
      beforePopState: jest.fn(),
      events: {
        on: jest.fn(),
        off: jest.fn(),
        emit: jest.fn(),
      },
      isFallback: false,
    };
  },
}));

// Mock next/navigation
jest.mock('next/navigation', () => ({
  useRouter() {
    return {
      push: jest.fn(),
      replace: jest.fn(),
      refresh: jest.fn(),
      back: jest.fn(),
      forward: jest.fn(),
      prefetch: jest.fn(),
    };
  },
  useSearchParams() {
    return new URLSearchParams();
  },
  usePathname() {
    return '/';
  },
}));

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

### **🧪 Component Testing**

```jsx
// components/Button.js
import { forwardRef } from 'react';

export const Button = forwardRef(({
  children,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  onClick,
  type = 'button',
  className = '',
  ...props
}, ref) => {
  const baseClasses = 'inline-flex items-center justify-center font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-600 text-white hover:bg-gray-700 focus:ring-gray-500',
    outline: 'border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-blue-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
  };
  
  const sizes = {
    sm: 'px-3 py-2 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };
  
  const variantClasses = variants[variant] || variants.primary;
  const sizeClasses = sizes[size] || sizes.md;
  
  const handleClick = (e) => {
    if (disabled || loading) return;
    onClick?.(e);
  };

  return (
    <button
      ref={ref}
      type={type}
      disabled={disabled || loading}
      onClick={handleClick}
      className={`${baseClasses} ${variantClasses} ${sizeClasses} ${
        disabled || loading ? 'opacity-50 cursor-not-allowed' : ''
      } ${className}`}
      {...props}
    >
      {loading && (
        <svg
          className="w-4 h-4 mr-2 animate-spin"
          viewBox="0 0 24 24"
          fill="none"
          data-testid="loading-spinner"
        >
          <circle
            className="opacity-25"
            cx="12"
            cy="12"
            r="10"
            stroke="currentColor"
            strokeWidth="4"
          />
          <path
            className="opacity-75"
            fill="currentColor"
            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
          />
        </svg>
      )}
      {children}
    </button>
  );
});

Button.displayName = 'Button';

// __tests__/components/Button.test.js
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '@/components/Button';

describe('Button Component', () => {
  // ✅ Basic rendering tests
  it('renders button with children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('applies correct variant classes', () => {
    const { rerender } = render(<Button variant="primary">Primary</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-blue-600');

    rerender(<Button variant="secondary">Secondary</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-gray-600');

    rerender(<Button variant="danger">Danger</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-red-600');
  });

  it('applies correct size classes', () => {
    const { rerender } = render(<Button size="sm">Small</Button>);
    expect(screen.getByRole('button')).toHaveClass('px-3 py-2 text-sm');

    rerender(<Button size="lg">Large</Button>);
    expect(screen.getByRole('button')).toHaveClass('px-6 py-3 text-lg');
  });

  // ✅ Interaction tests
  it('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('does not call onClick when disabled', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button disabled onClick={handleClick}>Disabled</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).not.toHaveBeenCalled();
  });

  it('does not call onClick when loading', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button loading onClick={handleClick}>Loading</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).not.toHaveBeenCalled();
  });

  // ✅ Loading state tests
  it('shows loading spinner when loading prop is true', () => {
    render(<Button loading>Loading</Button>);
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });

  it('applies disabled styles when loading', () => {
    render(<Button loading>Loading</Button>);
    expect(screen.getByRole('button')).toHaveClass('opacity-50 cursor-not-allowed');
  });

  // ✅ Accessibility tests
  it('has correct button type', () => {
    render(<Button type="submit">Submit</Button>);
    expect(screen.getByRole('button')).toHaveAttribute('type', 'submit');
  });

  it('forwards ref correctly', () => {
    const ref = { current: null };
    render(<Button ref={ref}>Button</Button>);
    expect(ref.current).toBeInstanceOf(HTMLButtonElement);
  });

  // ✅ Custom props tests
  it('accepts custom className', () => {
    render(<Button className="custom-class">Custom</Button>);
    expect(screen.getByRole('button')).toHaveClass('custom-class');
  });

  it('spreads additional props', () => {
    render(<Button data-testid="custom-button" aria-label="Custom label">Test</Button>);
    const button = screen.getByTestId('custom-button');
    expect(button).toHaveAttribute('aria-label', 'Custom label');
  });
});

// components/ProductCard.js
import Link from 'next/link';
import Image from 'next/image';
import { Button } from './Button';

export function ProductCard({ product, onAddToCart }) {
  const { id, name, price, image, slug, inStock } = product;
  
  const formattedPrice = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(price / 100);

  const handleAddToCart = () => {
    onAddToCart?.(product);
  };

  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden" data-testid="product-card">
      <Link href={`/products/${slug}`} className="block">
        <div className="aspect-square relative">
          <Image
            src={image}
            alt={name}
            fill
            className="object-cover hover:scale-105 transition-transform duration-200"
          />
        </div>
      </Link>
      
      <div className="p-4">
        <Link href={`/products/${slug}`}>
          <h3 className="font-semibold text-lg text-gray-900 hover:text-blue-600 transition-colors">
            {name}
          </h3>
        </Link>
        
        <p className="text-xl font-bold text-gray-900 mt-2">
          {formattedPrice}
        </p>
        
        <div className="mt-4">
          {inStock ? (
            <Button 
              onClick={handleAddToCart}
              className="w-full"
              data-testid="add-to-cart-button"
            >
              Add to Cart
            </Button>
          ) : (
            <Button 
              disabled 
              className="w-full"
              data-testid="out-of-stock-button"
            >
              Out of Stock
            </Button>
          )}
        </div>
      </div>
    </div>
  );
}

// __tests__/components/ProductCard.test.js
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ProductCard } from '@/components/ProductCard';

// Mock Next.js components
jest.mock('next/link', () => {
  return function MockLink({ children, href }) {
    return <a href={href}>{children}</a>;
  };
});

jest.mock('next/image', () => {
  return function MockImage({ src, alt, ...props }) {
    return <img src={src} alt={alt} {...props} />;
  };
});

describe('ProductCard Component', () => {
  const mockProduct = {
    id: 1,
    name: 'Test Product',
    price: 2999, // Price in cents
    image: '/test-image.jpg',
    slug: 'test-product',
    inStock: true
  };

  const mockOnAddToCart = jest.fn();

  beforeEach(() => {
    mockOnAddToCart.mockClear();
  });

  // ✅ Rendering tests
  it('renders product information correctly', () => {
    render(<ProductCard product={mockProduct} onAddToCart={mockOnAddToCart} />);
    
    expect(screen.getByText('Test Product')).toBeInTheDocument();
    expect(screen.getByText('$29.99')).toBeInTheDocument();
    expect(screen.getByAltText('Test Product')).toBeInTheDocument();
  });

  it('renders product image with correct attributes', () => {
    render(<ProductCard product={mockProduct} onAddToCart={mockOnAddToCart} />);
    
    const image = screen.getByAltText('Test Product');
    expect(image).toHaveAttribute('src', '/test-image.jpg');
  });

  it('renders links with correct href', () => {
    render(<ProductCard product={mockProduct} onAddToCart={mockOnAddToCart} />);
    
    const links = screen.getAllByRole('link');
    links.forEach(link => {
      expect(link).toHaveAttribute('href', '/products/test-product');
    });
  });

  // ✅ Interactive behavior tests
  it('calls onAddToCart when add to cart button is clicked', async () => {
    const user = userEvent.setup();
    render(<ProductCard product={mockProduct} onAddToCart={mockOnAddToCart} />);
    
    const addToCartButton = screen.getByTestId('add-to-cart-button');
    await user.click(addToCartButton);
    
    expect(mockOnAddToCart).toHaveBeenCalledTimes(1);
    expect(mockOnAddToCart).toHaveBeenCalledWith(mockProduct);
  });

  it('shows out of stock button when product is not in stock', () => {
    const outOfStockProduct = { ...mockProduct, inStock: false };
    render(<ProductCard product={outOfStockProduct} onAddToCart={mockOnAddToCart} />);
    
    expect(screen.getByTestId('out-of-stock-button')).toBeInTheDocument();
    expect(screen.getByText('Out of Stock')).toBeInTheDocument();
    expect(screen.queryByTestId('add-to-cart-button')).not.toBeInTheDocument();
  });

  it('does not call onAddToCart when out of stock button is clicked', async () => {
    const user = userEvent.setup();
    const outOfStockProduct = { ...mockProduct, inStock: false };
    render(<ProductCard product={outOfStockProduct} onAddToCart={mockOnAddToCart} />);
    
    const outOfStockButton = screen.getByTestId('out-of-stock-button');
    await user.click(outOfStockButton);
    
    expect(mockOnAddToCart).not.toHaveBeenCalled();
  });

  // ✅ Price formatting tests
  it('formats price correctly for different values', () => {
    const { rerender } = render(
      <ProductCard 
        product={{ ...mockProduct, price: 1000 }} 
        onAddToCart={mockOnAddToCart} 
      />
    );
    expect(screen.getByText('$10.00')).toBeInTheDocument();

    rerender(
      <ProductCard 
        product={{ ...mockProduct, price: 99 }} 
        onAddToCart={mockOnAddToCart} 
      />
    );
    expect(screen.getByText('$0.99')).toBeInTheDocument();
  });

  // ✅ Accessibility tests
  it('has proper data-testid for testing', () => {
    render(<ProductCard product={mockProduct} onAddToCart={mockOnAddToCart} />);
    expect(screen.getByTestId('product-card')).toBeInTheDocument();
  });
});
```

## 🔗 Integration Testing

### **🧩 API Route Testing**

```javascript
// __tests__/api/products.test.js
import { createMocks } from 'node-mocks-http';
import handler from '@/pages/api/products';
import { prisma } from '@/lib/prisma';

// Mock Prisma
jest.mock('@/lib/prisma', () => ({
  prisma: {
    product: {
      findMany: jest.fn(),
      create: jest.fn(),
      findUnique: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    },
  },
}));

describe('/api/products', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // ✅ GET request tests
  describe('GET /api/products', () => {
    it('returns products successfully', async () => {
      const mockProducts = [
        { id: 1, name: 'Product 1', price: 1000 },
        { id: 2, name: 'Product 2', price: 2000 },
      ];

      prisma.product.findMany.mockResolvedValue(mockProducts);

      const { req, res } = createMocks({
        method: 'GET',
      });

      await handler(req, res);

      expect(res._getStatusCode()).toBe(200);
      expect(JSON.parse(res._getData())).toEqual({
        products: mockProducts,
        total: mockProducts.length,
      });
    });

    it('handles search query parameter', async () => {
      const mockProducts = [
        { id: 1, name: 'iPhone', price: 99999 },
      ];

      prisma.product.findMany.mockResolvedValue(mockProducts);

      const { req, res } = createMocks({
        method: 'GET',
        query: { search: 'iPhone' },
      });

      await handler(req, res);

      expect(prisma.product.findMany).toHaveBeenCalledWith({
        where: {
          OR: [
            { name: { contains: 'iPhone', mode: 'insensitive' } },
            { description: { contains: 'iPhone', mode: 'insensitive' } },
          ],
        },
        include: expect.any(Object),
      });
    });

    it('handles database errors', async () => {
      prisma.product.findMany.mockRejectedValue(new Error('Database error'));

      const { req, res } = createMocks({
        method: 'GET',
      });

      await handler(req, res);

      expect(res._getStatusCode()).toBe(500);
      expect(JSON.parse(res._getData())).toEqual({
        error: 'Internal server error',
      });
    });
  });

  // ✅ POST request tests
  describe('POST /api/products', () => {
    it('creates product successfully', async () => {
      const newProduct = {
        name: 'New Product',
        description: 'Product description',
        price: 1500,
        categoryId: 1,
      };

      const createdProduct = { id: 1, ...newProduct };
      prisma.product.create.mockResolvedValue(createdProduct);

      const { req, res } = createMocks({
        method: 'POST',
        body: newProduct,
      });

      await handler(req, res);

      expect(res._getStatusCode()).toBe(201);
      expect(JSON.parse(res._getData())).toEqual(createdProduct);
    });

    it('validates required fields', async () => {
      const invalidProduct = {
        description: 'Missing name and price',
      };

      const { req, res } = createMocks({
        method: 'POST',
        body: invalidProduct,
      });

      await handler(req, res);

      expect(res._getStatusCode()).toBe(400);
      expect(JSON.parse(res._getData())).toHaveProperty('error');
    });
  });

  // ✅ Unsupported method test
  it('returns 405 for unsupported methods', async () => {
    const { req, res } = createMocks({
      method: 'DELETE',
    });

    await handler(req, res);

    expect(res._getStatusCode()).toBe(405);
    expect(JSON.parse(res._getData())).toEqual({
      error: 'Method not allowed',
    });
  });
});

// __tests__/api/auth/login.test.js
import { createMocks } from 'node-mocks-http';
import handler from '@/pages/api/auth/login';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

jest.mock('bcrypt');
jest.mock('jsonwebtoken');
jest.mock('@/lib/prisma', () => ({
  prisma: {
    user: {
      findUnique: jest.fn(),
    },
  },
}));

describe('/api/auth/login', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('logs in user with valid credentials', async () => {
    const mockUser = {
      id: 1,
      email: 'test@example.com',
      password: 'hashedpassword',
      name: 'Test User',
    };

    const loginData = {
      email: 'test@example.com',
      password: 'password123',
    };

    prisma.user.findUnique.mockResolvedValue(mockUser);
    bcrypt.compare.mockResolvedValue(true);
    jwt.sign.mockReturnValue('fake-jwt-token');

    const { req, res } = createMocks({
      method: 'POST',
      body: loginData,
    });

    await handler(req, res);

    expect(res._getStatusCode()).toBe(200);
    expect(JSON.parse(res._getData())).toMatchObject({
      user: {
        id: 1,
        email: 'test@example.com',
        name: 'Test User',
      },
    });
  });

  it('returns error for invalid credentials', async () => {
    prisma.user.findUnique.mockResolvedValue(null);

    const { req, res } = createMocks({
      method: 'POST',
      body: {
        email: 'nonexistent@example.com',
        password: 'password123',
      },
    });

    await handler(req, res);

    expect(res._getStatusCode()).toBe(401);
    expect(JSON.parse(res._getData())).toEqual({
      error: 'Invalid credentials',
    });
  });
});
```

### **🔄 Custom Hook Testing**

```javascript
// hooks/useCart.js
import { useState, useCallback, useMemo } from 'react';

export function useCart() {
  const [items, setItems] = useState([]);

  const addItem = useCallback((product) => {
    setItems(prevItems => {
      const existingItem = prevItems.find(item => item.id === product.id);
      
      if (existingItem) {
        return prevItems.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      
      return [...prevItems, { ...product, quantity: 1 }];
    });
  }, []);

  const removeItem = useCallback((productId) => {
    setItems(prevItems => prevItems.filter(item => item.id !== productId));
  }, []);

  const updateQuantity = useCallback((productId, quantity) => {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }

    setItems(prevItems =>
      prevItems.map(item =>
        item.id === productId
          ? { ...item, quantity }
          : item
      )
    );
  }, [removeItem]);

  const clearCart = useCallback(() => {
    setItems([]);
  }, []);

  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }, [items]);

  const itemCount = useMemo(() => {
    return items.reduce((count, item) => count + item.quantity, 0);
  }, [items]);

  return {
    items,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    total,
    itemCount,
  };
}

// __tests__/hooks/useCart.test.js
import { renderHook, act } from '@testing-library/react';
import { useCart } from '@/hooks/useCart';

describe('useCart hook', () => {
  const mockProduct = {
    id: 1,
    name: 'Test Product',
    price: 1000,
  };

  // ✅ Initial state tests
  it('initializes with empty cart', () => {
    const { result } = renderHook(() => useCart());
    
    expect(result.current.items).toEqual([]);
    expect(result.current.total).toBe(0);
    expect(result.current.itemCount).toBe(0);
  });

  // ✅ Add item tests
  it('adds new item to cart', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem(mockProduct);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0]).toEqual({
      ...mockProduct,
      quantity: 1,
    });
    expect(result.current.total).toBe(1000);
    expect(result.current.itemCount).toBe(1);
  });

  it('increases quantity when adding existing item', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem(mockProduct);
      result.current.addItem(mockProduct);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(2);
    expect(result.current.total).toBe(2000);
    expect(result.current.itemCount).toBe(2);
  });

  // ✅ Remove item tests
  it('removes item from cart', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem(mockProduct);
    });

    expect(result.current.items).toHaveLength(1);

    act(() => {
      result.current.removeItem(mockProduct.id);
    });

    expect(result.current.items).toHaveLength(0);
    expect(result.current.total).toBe(0);
    expect(result.current.itemCount).toBe(0);
  });

  // ✅ Update quantity tests
  it('updates item quantity', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem(mockProduct);
    });

    act(() => {
      result.current.updateQuantity(mockProduct.id, 5);
    });

    expect(result.current.items[0].quantity).toBe(5);
    expect(result.current.total).toBe(5000);
    expect(result.current.itemCount).toBe(5);
  });

  it('removes item when quantity is set to 0 or negative', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem(mockProduct);
    });

    expect(result.current.items).toHaveLength(1);

    act(() => {
      result.current.updateQuantity(mockProduct.id, 0);
    });

    expect(result.current.items).toHaveLength(0);
  });

  // ✅ Clear cart tests
  it('clears all items from cart', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem(mockProduct);
      result.current.addItem({ ...mockProduct, id: 2 });
    });

    expect(result.current.items).toHaveLength(2);

    act(() => {
      result.current.clearCart();
    });

    expect(result.current.items).toHaveLength(0);
    expect(result.current.total).toBe(0);
    expect(result.current.itemCount).toBe(0);
  });

  // ✅ Multiple items tests
  it('handles multiple different items correctly', () => {
    const { result } = renderHook(() => useCart());
    
    const product2 = { id: 2, name: 'Product 2', price: 2000 };
    
    act(() => {
      result.current.addItem(mockProduct);
      result.current.addItem(product2);
      result.current.addItem(mockProduct); // Add first product again
    });

    expect(result.current.items).toHaveLength(2);
    expect(result.current.items.find(item => item.id === 1).quantity).toBe(2);
    expect(result.current.items.find(item => item.id === 2).quantity).toBe(1);
    expect(result.current.total).toBe(4000); // (1000 * 2) + (2000 * 1)
    expect(result.current.itemCount).toBe(3);
  });
});
```

## 🌐 End-to-End Testing

### **🎭 Playwright Setup**

```bash
# Install Playwright
npm install --save-dev @playwright/test
npx playwright install
```

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  
  // Run tests in files in parallel
  fullyParallel: true,
  
  // Fail the build on CI if you accidentally left test.only in the source code
  forbidOnly: !!process.env.CI,
  
  // Retry on CI only
  retries: process.env.CI ? 2 : 0,
  
  // Opt out of parallel tests on CI
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter to use
  reporter: 'html',
  
  // Shared settings for all tests
  use: {
    // Base URL to use in actions like `await page.goto('/')`
    baseURL: 'http://localhost:3000',
    
    // Collect trace when retrying the failed test
    trace: 'on-first-retry',
    
    // Take screenshot on failure
    screenshot: 'only-on-failure',
    
    // Record video on failure
    video: 'retain-on-failure',
  },

  // Configure projects for major browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Run your local dev server before starting the tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});

// e2e/auth.setup.js - Authentication helper
import { test as setup, expect } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  // Perform authentication steps
  await page.goto('/login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('[type="submit"]');
  
  // Wait until the page receives the cookies
  await page.waitForURL('/dashboard');
  
  // End of authentication steps
  await page.context().storageState({ path: authFile });
});

// e2e/example.spec.js
import { test, expect } from '@playwright/test';

test.describe('Homepage', () => {
  // ✅ Basic page tests
  test('should load homepage', async ({ page }) => {
    await page.goto('/');
    
    await expect(page).toHaveTitle(/Your App Name/);
    await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
  });

  test('should navigate to products page', async ({ page }) => {
    await page.goto('/');
    
    await page.click('[href="/products"]');
    await expect(page).toHaveURL('/products');
    await expect(page.getByRole('heading', { name: 'Products' })).toBeVisible();
  });

  // ✅ Search functionality
  test('should search for products', async ({ page }) => {
    await page.goto('/products');
    
    const searchInput = page.getByPlaceholder('Search products...');
    await searchInput.fill('iPhone');
    await searchInput.press('Enter');
    
    await expect(page).toHaveURL(/\/products\?search=iPhone/);
    await expect(page.getByText('iPhone')).toBeVisible();
  });
});

test.describe('Product Catalog', () => {
  test('should display product grid', async ({ page }) => {
    await page.goto('/products');
    
    // Wait for products to load
    await page.waitForSelector('[data-testid="product-card"]');
    
    const productCards = page.locator('[data-testid="product-card"]');
    await expect(productCards).toHaveCount.greaterThan(0);
    
    // Check first product has required elements
    const firstProduct = productCards.first();
    await expect(firstProduct.getByRole('heading')).toBeVisible();
    await expect(firstProduct.getByText(/\$/)).toBeVisible();
    await expect(firstProduct.getByRole('button')).toBeVisible();
  });

  test('should filter products by category', async ({ page }) => {
    await page.goto('/products');
    
    // Click on category filter
    await page.click('[data-testid="category-electronics"]');
    
    // Check URL contains category parameter
    await expect(page).toHaveURL(/category=electronics/);
    
    // Verify filtered results
    await page.waitForSelector('[data-testid="product-card"]');
    const products = page.locator('[data-testid="product-card"]');
    await expect(products).toHaveCount.greaterThan(0);
  });

  test('should sort products by price', async ({ page }) => {
    await page.goto('/products');
    
    // Select sort option
    await page.selectOption('[data-testid="sort-select"]', 'price-low');
    
    // Wait for products to reload
    await page.waitForSelector('[data-testid="product-card"]');
    
    // Get all price elements
    const prices = await page.locator('[data-testid="product-price"]').allTextContents();
    
    // Convert to numbers and verify sorting
    const numericPrices = prices.map(price => 
      parseFloat(price.replace('$', '').replace(',', ''))
    );
    
    for (let i = 1; i < numericPrices.length; i++) {
      expect(numericPrices[i]).toBeGreaterThanOrEqual(numericPrices[i - 1]);
    }
  });
});

test.describe('Shopping Cart', () => {
  test('should add product to cart', async ({ page }) => {
    await page.goto('/products');
    
    // Wait for products and click first "Add to Cart" button
    await page.waitForSelector('[data-testid="add-to-cart-button"]');
    await page.click('[data-testid="add-to-cart-button"]');
    
    // Check cart counter updated
    const cartCounter = page.locator('[data-testid="cart-counter"]');
    await expect(cartCounter).toHaveText('1');
    
    // Check success message
    await expect(page.getByText('Added to cart')).toBeVisible();
  });

  test('should view cart contents', async ({ page }) => {
    await page.goto('/products');
    
    // Add product to cart
    await page.waitForSelector('[data-testid="add-to-cart-button"]');
    await page.click('[data-testid="add-to-cart-button"]');
    
    // Go to cart
    await page.click('[data-testid="cart-link"]');
    await expect(page).toHaveURL('/cart');
    
    // Verify cart contents
    await expect(page.getByRole('heading', { name: 'Shopping Cart' })).toBeVisible();
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(1);
  });

  test('should update quantity in cart', async ({ page }) => {
    await page.goto('/cart');
    
    // Assuming there's already an item in cart
    const quantityInput = page.locator('[data-testid="quantity-input"]').first();
    await quantityInput.fill('3');
    await quantityInput.blur();
    
    // Wait for update and check total
    await page.waitForTimeout(500); // Wait for debounced update
    const total = page.locator('[data-testid="cart-total"]');
    await expect(total).toContainText('$'); // Verify total updated
  });

  test('should remove item from cart', async ({ page }) => {
    await page.goto('/cart');
    
    // Get initial item count
    const initialItems = await page.locator('[data-testid="cart-item"]').count();
    
    if (initialItems > 0) {
      // Click remove button
      await page.click('[data-testid="remove-item"]');
      
      // Verify item removed
      await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(initialItems - 1);
    }
  });
});

// e2e/checkout.spec.js - Authenticated user tests
import { test, expect } from '@playwright/test';

test.use({ storageState: 'playwright/.auth/user.json' });

test.describe('Checkout Process', () => {
  test.beforeEach(async ({ page }) => {
    // Add items to cart before each test
    await page.goto('/products');
    await page.waitForSelector('[data-testid="add-to-cart-button"]');
    await page.click('[data-testid="add-to-cart-button"]');
  });

  test('should proceed to checkout', async ({ page }) => {
    await page.goto('/cart');
    
    await page.click('[data-testid="checkout-button"]');
    await expect(page).toHaveURL('/checkout');
    await expect(page.getByRole('heading', { name: 'Checkout' })).toBeVisible();
  });

  test('should complete order', async ({ page }) => {
    await page.goto('/checkout');
    
    // Fill shipping information
    await page.fill('[name="firstName"]', 'John');
    await page.fill('[name="lastName"]', 'Doe');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'Anytown');
    await page.fill('[name="zipCode"]', '12345');
    
    // Fill payment information (assuming test mode)
    await page.fill('[name="cardNumber"]', '4242424242424242');
    await page.fill('[name="expiryDate"]', '12/25');
    await page.fill('[name="cvv"]', '123');
    
    // Submit order
    await page.click('[data-testid="place-order-button"]');
    
    // Verify order success
    await expect(page).toHaveURL(/\/order\/confirmation/);
    await expect(page.getByText('Order placed successfully')).toBeVisible();
  });

  test('should validate required fields', async ({ page }) => {
    await page.goto('/checkout');
    
    // Try to submit without filling fields
    await page.click('[data-testid="place-order-button"]');
    
    // Check for validation errors
    await expect(page.getByText('First name is required')).toBeVisible();
    await expect(page.getByText('Address is required')).toBeVisible();
  });
});
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Component Testing**
1. เขียน unit tests สำหรับ components หลัก
2. ทดสอบ user interactions
3. เพิ่ม accessibility tests
4. วัด test coverage

### **🎯 แบบฝึกหัดที่ 2: API Testing**
1. เขียน integration tests สำหรับ API routes
2. ทดสอบ authentication และ authorization
3. เพิ่ม error handling tests
4. ทดสอบ data validation

### **🎯 แบบฝึกหัดที่ 3: E2E Testing**
1. สร้าง user journey tests
2. ทดสอบ cross-browser compatibility
3. เพิ่ม mobile responsive tests
4. ทดสอบ performance

### **🎯 แบบฝึกหัดที่ 4: Test Automation**
1. ตั้งค่า CI/CD pipeline
2. เพิ่ม automated testing
3. สร้าง test reports
4. เพิ่ม quality gates

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 18: SEO and Metadata](./18-seo-metadata.md)

**บทต่อไป**: [บทที่ 20: Deployment and DevOps](./20-deployment.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Next.js Testing](https://nextjs.org/docs/app/building-your-application/testing)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright Documentation](https://playwright.dev/)
