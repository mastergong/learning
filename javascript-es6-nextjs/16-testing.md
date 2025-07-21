# บทที่ 16: Testing Strategies

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ comprehensive testing strategies
- ทำความเข้าใจ unit, integration, และ end-to-end testing
- เรียนรู้ test-driven development (TDD) และ behavior-driven development (BDD)
- ประยุกต์ใช้ใน Next.js สำหรับ robust testing environment

## 🧪 Testing Fundamentals

### **🎪 Testing Framework Setup**

```javascript
// ============= JEST CONFIGURATION =============
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
  
  // Module name mapping for absolute imports
  moduleNameMapper: {
    '^@/components/(.*)$': '<rootDir>/components/$1',
    '^@/pages/(.*)$': '<rootDir>/pages/$1',
    '^@/lib/(.*)$': '<rootDir>/lib/$1',
    '^@/hooks/(.*)$': '<rootDir>/hooks/$1',
    '^@/utils/(.*)$': '<rootDir>/utils/$1',
    '^@/styles/(.*)$': '<rootDir>/styles/$1',
  },
  
  // Test environment
  testEnvironment: 'jest-environment-jsdom',
  
  // Coverage settings
  collectCoverageFrom: [
    '**/*.{js,jsx,ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
    '!**/.next/**',
    '!**/coverage/**',
    '!jest.config.js',
    '!next.config.js',
  ],
  
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  
  // Test file patterns
  testMatch: [
    '<rootDir>/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/**/?(*.)+(spec|test).{js,jsx,ts,tsx}',
  ],
  
  // Transform files
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest',
  },
  
  // Module file extensions
  moduleFileExtensions: ['js', 'jsx', 'ts', 'tsx', 'json'],
  
  // Ignore patterns
  testPathIgnorePatterns: [
    '<rootDir>/.next/',
    '<rootDir>/node_modules/',
  ],
  
  // Mock modules
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$':
      '<rootDir>/__mocks__/fileMock.js',
  },
};

// CreateJestConfig is exported this way to ensure that next/jest can load the Next.js config which is async
module.exports = createJestConfig(customJestConfig);

// ============= JEST SETUP =============
// jest.setup.js
import '@testing-library/jest-dom';
import 'jest-extended';

// Mock Next.js router
jest.mock('next/router', () => require('next-router-mock'));

// Mock Next.js dynamic imports
jest.mock('next/dynamic', () => () => {
  const DynamicComponent = () => null;
  DynamicComponent.displayName = 'LoadableComponent';
  DynamicComponent.preload = jest.fn();
  return DynamicComponent;
});

// Global test utilities
global.mockFetch = (data) => {
  global.fetch = jest.fn(() =>
    Promise.resolve({
      ok: true,
      status: 200,
      json: () => Promise.resolve(data),
      text: () => Promise.resolve(JSON.stringify(data)),
    })
  );
};

global.mockFetchError = (error) => {
  global.fetch = jest.fn(() => Promise.reject(error));
};

// Performance observer mock
global.PerformanceObserver = class PerformanceObserver {
  constructor(callback) {
    this.callback = callback;
  }
  observe() {}
  disconnect() {}
};

// Intersection observer mock
global.IntersectionObserver = class IntersectionObserver {
  constructor(callback) {
    this.callback = callback;
  }
  observe() {}
  unobserve() {}
  disconnect() {}
};

// Local storage mock
Object.defineProperty(window, 'localStorage', {
  value: {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn(),
  },
  writable: true,
});

// Session storage mock
Object.defineProperty(window, 'sessionStorage', {
  value: {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn(),
  },
  writable: true,
});

// ============= CUSTOM TESTING UTILITIES =============
// __tests__/utils/testing-utils.js
import { render, screen, waitFor, act } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { ThemeProvider } from 'styled-components';

// Create test store
const createTestStore = (initialState = {}) => {
  return configureStore({
    reducer: {
      // Add your reducers here
    },
    preloadedState: initialState,
  });
};

// Create test query client
const createTestQueryClient = () => {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });
};

// Custom render function with providers
export const renderWithProviders = (
  ui,
  {
    initialState = {},
    store = createTestStore(initialState),
    queryClient = createTestQueryClient(),
    theme = {},
    ...renderOptions
  } = {}
) => {
  const Wrapper = ({ children }) => (
    <Provider store={store}>
      <QueryClientProvider client={queryClient}>
        <ThemeProvider theme={theme}>
          {children}
        </ThemeProvider>
      </QueryClientProvider>
    </Provider>
  );

  return {
    store,
    queryClient,
    user: userEvent.setup(),
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
  };
};

// Test data factories
export const createMockUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  role: 'user',
  createdAt: '2023-01-01T00:00:00Z',
  ...overrides,
});

export const createMockPost = (overrides = {}) => ({
  id: 1,
  title: 'Test Post',
  content: 'This is a test post content',
  authorId: 1,
  published: true,
  createdAt: '2023-01-01T00:00:00Z',
  ...overrides,
});

// Async test utilities
export const waitForLoadingToFinish = async () => {
  await waitFor(() => {
    expect(screen.queryByTestId('loading')).not.toBeInTheDocument();
  });
};

export const waitForErrorToShow = async () => {
  await waitFor(() => {
    expect(screen.getByRole('alert')).toBeInTheDocument();
  });
};

// Form testing utilities
export const fillForm = async (user, formData) => {
  for (const [fieldName, value] of Object.entries(formData)) {
    const field = screen.getByLabelText(new RegExp(fieldName, 'i'));
    await user.clear(field);
    await user.type(field, value);
  }
};

export const submitForm = async (user, formSelector = 'form') => {
  const form = screen.getByRole('form') || document.querySelector(formSelector);
  await user.click(screen.getByRole('button', { type: 'submit' }));
};

// API mocking utilities
export const mockApiResponse = (url, response, options = {}) => {
  const { status = 200, delay = 0 } = options;
  
  return jest.fn().mockImplementation((requestUrl) => {
    if (requestUrl.includes(url)) {
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve({
            ok: status >= 200 && status < 300,
            status,
            json: () => Promise.resolve(response),
            text: () => Promise.resolve(JSON.stringify(response)),
          });
        }, delay);
      });
    }
    return Promise.reject(new Error(`Unexpected request to ${requestUrl}`));
  });
};

// Component testing helpers
export const getByTestId = (testId) => screen.getByTestId(testId);
export const queryByTestId = (testId) => screen.queryByTestId(testId);
export const getAllByTestId = (testId) => screen.getAllByTestId(testId);

// Export everything from testing library
export * from '@testing-library/react';
export { userEvent };

// ============= MOCK DATA GENERATORS =============
// __tests__/utils/data-generators.js
import { faker } from '@faker-js/faker';

export class TestDataGenerator {
  static user(overrides = {}) {
    return {
      id: faker.number.int(),
      name: faker.person.fullName(),
      email: faker.internet.email(),
      avatar: faker.image.avatar(),
      role: faker.helpers.arrayElement(['admin', 'user', 'moderator']),
      isActive: faker.datatype.boolean(),
      createdAt: faker.date.past().toISOString(),
      updatedAt: faker.date.recent().toISOString(),
      profile: {
        bio: faker.lorem.paragraph(),
        website: faker.internet.url(),
        location: faker.location.city(),
        birthday: faker.date.birthdate().toISOString(),
      },
      preferences: {
        theme: faker.helpers.arrayElement(['light', 'dark']),
        language: faker.helpers.arrayElement(['en', 'th', 'ja']),
        notifications: faker.datatype.boolean(),
      },
      ...overrides,
    };
  }

  static users(count = 5, overrides = {}) {
    return Array.from({ length: count }, (_, index) => 
      this.user({ id: index + 1, ...overrides })
    );
  }

  static post(overrides = {}) {
    return {
      id: faker.number.int(),
      title: faker.lorem.sentence(),
      slug: faker.lorem.slug(),
      content: faker.lorem.paragraphs(3),
      excerpt: faker.lorem.paragraph(),
      authorId: faker.number.int(),
      categoryId: faker.number.int(),
      tags: faker.lorem.words(3).split(' '),
      published: faker.datatype.boolean(),
      featured: faker.datatype.boolean(),
      viewCount: faker.number.int({ min: 0, max: 10000 }),
      likeCount: faker.number.int({ min: 0, max: 1000 }),
      commentCount: faker.number.int({ min: 0, max: 100 }),
      createdAt: faker.date.past().toISOString(),
      updatedAt: faker.date.recent().toISOString(),
      publishedAt: faker.date.past().toISOString(),
      ...overrides,
    };
  }

  static posts(count = 10, overrides = {}) {
    return Array.from({ length: count }, (_, index) => 
      this.post({ id: index + 1, ...overrides })
    );
  }

  static comment(overrides = {}) {
    return {
      id: faker.number.int(),
      content: faker.lorem.paragraph(),
      authorId: faker.number.int(),
      postId: faker.number.int(),
      parentId: faker.datatype.boolean() ? faker.number.int() : null,
      approved: faker.datatype.boolean(),
      createdAt: faker.date.past().toISOString(),
      updatedAt: faker.date.recent().toISOString(),
      ...overrides,
    };
  }

  static apiResponse(data, overrides = {}) {
    return {
      success: true,
      data,
      message: faker.lorem.sentence(),
      timestamp: new Date().toISOString(),
      meta: {
        page: 1,
        limit: 10,
        total: Array.isArray(data) ? data.length : 1,
        totalPages: 1,
      },
      ...overrides,
    };
  }

  static apiError(overrides = {}) {
    return {
      success: false,
      error: {
        code: faker.helpers.arrayElement(['VALIDATION_ERROR', 'NOT_FOUND', 'UNAUTHORIZED']),
        message: faker.lorem.sentence(),
        details: faker.lorem.paragraph(),
      },
      timestamp: new Date().toISOString(),
      ...overrides,
    };
  }

  static generateFormData(schema) {
    const data = {};
    
    for (const [field, config] of Object.entries(schema)) {
      switch (config.type) {
        case 'string':
          data[field] = faker.lorem.words(config.words || 2);
          break;
        case 'email':
          data[field] = faker.internet.email();
          break;
        case 'number':
          data[field] = faker.number.int(config.range || { min: 1, max: 100 });
          break;
        case 'boolean':
          data[field] = faker.datatype.boolean();
          break;
        case 'date':
          data[field] = faker.date.recent().toISOString();
          break;
        case 'select':
          data[field] = faker.helpers.arrayElement(config.options || ['option1', 'option2']);
          break;
        default:
          data[field] = faker.lorem.word();
      }
    }
    
    return data;
  }
}

// ============= CUSTOM MATCHERS =============
// __tests__/utils/custom-matchers.js
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },

  toHaveBeenCalledWithObjectContaining(received, expectedObject) {
    const pass = received.mock.calls.some(call =>
      call.some(arg =>
        typeof arg === 'object' &&
        Object.keys(expectedObject).every(key =>
          arg[key] === expectedObject[key]
        )
      )
    );

    if (pass) {
      return {
        message: () =>
          `expected function not to have been called with object containing ${JSON.stringify(expectedObject)}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected function to have been called with object containing ${JSON.stringify(expectedObject)}`,
        pass: false,
      };
    }
  },

  toBeValidDate(received) {
    const pass = received instanceof Date && !isNaN(received.getTime());
    if (pass) {
      return {
        message: () => `expected ${received} not to be a valid date`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected ${received} to be a valid date`,
        pass: false,
      };
    }
  },

  toHaveProperty(received, property) {
    const pass = Object.prototype.hasOwnProperty.call(received, property);
    if (pass) {
      return {
        message: () => `expected object not to have property ${property}`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected object to have property ${property}`,
        pass: false,
      };
    }
  },
});
```

## 🧪 Unit Testing

### **🔬 Component Testing**

```jsx
// ============= COMPONENT TESTING EXAMPLES =============
// __tests__/components/Button.test.jsx
import { renderWithProviders, userEvent } from '../utils/testing-utils';
import Button from '@/components/Button';

describe('Button Component', () => {
  it('renders with default props', () => {
    const { getByRole } = renderWithProviders(<Button>Click me</Button>);
    
    const button = getByRole('button');
    expect(button).toBeInTheDocument();
    expect(button).toHaveTextContent('Click me');
    expect(button).toHaveClass('btn-primary');
  });

  it('handles different variants', () => {
    const { getByRole } = renderWithProviders(
      <Button variant="secondary">Secondary</Button>
    );
    
    expect(getByRole('button')).toHaveClass('btn-secondary');
  });

  it('handles disabled state', () => {
    const handleClick = jest.fn();
    const { getByRole, user } = renderWithProviders(
      <Button disabled onClick={handleClick}>
        Disabled
      </Button>
    );
    
    const button = getByRole('button');
    expect(button).toBeDisabled();
    
    // Click should not trigger handler
    user.click(button);
    expect(handleClick).not.toHaveBeenCalled();
  });

  it('handles loading state', () => {
    const { getByRole, getByTestId } = renderWithProviders(
      <Button loading>Loading</Button>
    );
    
    expect(getByRole('button')).toBeDisabled();
    expect(getByTestId('loading-spinner')).toBeInTheDocument();
  });

  it('calls onClick handler', async () => {
    const handleClick = jest.fn();
    const { getByRole, user } = renderWithProviders(
      <Button onClick={handleClick}>Click me</Button>
    );
    
    await user.click(getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('prevents multiple clicks when loading', async () => {
    const handleClick = jest.fn();
    const { getByRole, user } = renderWithProviders(
      <Button loading onClick={handleClick}>
        Loading
      </Button>
    );
    
    const button = getByRole('button');
    await user.click(button);
    await user.click(button);
    
    expect(handleClick).not.toHaveBeenCalled();
  });

  it('supports keyboard navigation', async () => {
    const handleClick = jest.fn();
    const { getByRole, user } = renderWithProviders(
      <Button onClick={handleClick}>Press Enter</Button>
    );
    
    const button = getByRole('button');
    button.focus();
    
    await user.keyboard('{Enter}');
    expect(handleClick).toHaveBeenCalledTimes(1);
    
    await user.keyboard(' ');
    expect(handleClick).toHaveBeenCalledTimes(2);
  });
});

// ============= FORM TESTING =============
// __tests__/components/ContactForm.test.jsx
import { renderWithProviders, fillForm, submitForm, waitFor } from '../utils/testing-utils';
import { TestDataGenerator } from '../utils/data-generators';
import ContactForm from '@/components/ContactForm';

describe('ContactForm Component', () => {
  const mockOnSubmit = jest.fn();
  
  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders all form fields', () => {
    const { getByLabelText } = renderWithProviders(
      <ContactForm onSubmit={mockOnSubmit} />
    );
    
    expect(getByLabelText(/name/i)).toBeInTheDocument();
    expect(getByLabelText(/email/i)).toBeInTheDocument();
    expect(getByLabelText(/message/i)).toBeInTheDocument();
  });

  it('validates required fields', async () => {
    const { getByRole, getByText, user } = renderWithProviders(
      <ContactForm onSubmit={mockOnSubmit} />
    );
    
    await submitForm(user);
    
    await waitFor(() => {
      expect(getByText(/name is required/i)).toBeInTheDocument();
      expect(getByText(/email is required/i)).toBeInTheDocument();
      expect(getByText(/message is required/i)).toBeInTheDocument();
    });
    
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  it('validates email format', async () => {
    const { getByLabelText, getByText, user } = renderWithProviders(
      <ContactForm onSubmit={mockOnSubmit} />
    );
    
    await user.type(getByLabelText(/email/i), 'invalid-email');
    await user.tab(); // Trigger blur event
    
    await waitFor(() => {
      expect(getByText(/invalid email format/i)).toBeInTheDocument();
    });
  });

  it('submits form with valid data', async () => {
    const formData = TestDataGenerator.generateFormData({
      name: { type: 'string', words: 2 },
      email: { type: 'email' },
      message: { type: 'string', words: 10 },
    });
    
    const { user } = renderWithProviders(
      <ContactForm onSubmit={mockOnSubmit} />
    );
    
    await fillForm(user, formData);
    await submitForm(user);
    
    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith(
        expect.objectContaining(formData)
      );
    });
  });

  it('shows loading state during submission', async () => {
    const slowSubmit = jest.fn(() => 
      new Promise(resolve => setTimeout(resolve, 1000))
    );
    
    const { getByRole, getByTestId, user } = renderWithProviders(
      <ContactForm onSubmit={slowSubmit} />
    );
    
    await fillForm(user, {
      name: 'John Doe',
      email: 'john@example.com',
      message: 'Test message',
    });
    
    await submitForm(user);
    
    expect(getByTestId('loading')).toBeInTheDocument();
    expect(getByRole('button', { name: /submit/i })).toBeDisabled();
  });

  it('handles submission errors', async () => {
    const errorSubmit = jest.fn(() => 
      Promise.reject(new Error('Submission failed'))
    );
    
    const { getByText, user } = renderWithProviders(
      <ContactForm onSubmit={errorSubmit} />
    );
    
    await fillForm(user, {
      name: 'John Doe',
      email: 'john@example.com',
      message: 'Test message',
    });
    
    await submitForm(user);
    
    await waitFor(() => {
      expect(getByText(/submission failed/i)).toBeInTheDocument();
    });
  });
});

// ============= HOOK TESTING =============
// __tests__/hooks/useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from '@/hooks/useCounter';

describe('useCounter Hook', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('resets count', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });

  it('sets count to specific value', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.setValue(42);
    });
    
    expect(result.current.count).toBe(42);
  });

  it('respects min and max bounds', () => {
    const { result } = renderHook(() => useCounter(5, { min: 0, max: 10 }));
    
    // Test max bound
    act(() => {
      result.current.setValue(15);
    });
    expect(result.current.count).toBe(10);
    
    // Test min bound
    act(() => {
      result.current.setValue(-5);
    });
    expect(result.current.count).toBe(0);
  });
});

// ============= ASYNC COMPONENT TESTING =============
// __tests__/components/UserList.test.jsx
import { renderWithProviders, waitForLoadingToFinish, mockApiResponse } from '../utils/testing-utils';
import { TestDataGenerator } from '../utils/data-generators';
import UserList from '@/components/UserList';

describe('UserList Component', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  it('renders loading state initially', () => {
    const { getByTestId } = renderWithProviders(<UserList />);
    
    expect(getByTestId('loading')).toBeInTheDocument();
  });

  it('renders user list after loading', async () => {
    const users = TestDataGenerator.users(3);
    const apiResponse = TestDataGenerator.apiResponse(users);
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => apiResponse,
    });
    
    const { getByText } = renderWithProviders(<UserList />);
    
    await waitForLoadingToFinish();
    
    users.forEach(user => {
      expect(getByText(user.name)).toBeInTheDocument();
      expect(getByText(user.email)).toBeInTheDocument();
    });
  });

  it('handles API errors', async () => {
    fetch.mockRejectedValueOnce(new Error('API Error'));
    
    const { getByText } = renderWithProviders(<UserList />);
    
    await waitForLoadingToFinish();
    
    expect(getByText(/error loading users/i)).toBeInTheDocument();
  });

  it('handles empty user list', async () => {
    const apiResponse = TestDataGenerator.apiResponse([]);
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => apiResponse,
    });
    
    const { getByText } = renderWithProviders(<UserList />);
    
    await waitForLoadingToFinish();
    
    expect(getByText(/no users found/i)).toBeInTheDocument();
  });

  it('filters users by search term', async () => {
    const users = [
      TestDataGenerator.user({ name: 'John Doe' }),
      TestDataGenerator.user({ name: 'Jane Smith' }),
      TestDataGenerator.user({ name: 'Bob Johnson' }),
    ];
    const apiResponse = TestDataGenerator.apiResponse(users);
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => apiResponse,
    });
    
    const { getByLabelText, getByText, queryByText, user } = renderWithProviders(
      <UserList />
    );
    
    await waitForLoadingToFinish();
    
    // Search for "John"
    const searchInput = getByLabelText(/search/i);
    await user.type(searchInput, 'John');
    
    expect(getByText('John Doe')).toBeInTheDocument();
    expect(getByText('Bob Johnson')).toBeInTheDocument();
    expect(queryByText('Jane Smith')).not.toBeInTheDocument();
  });
});
```

## 🔗 Integration Testing

### **🌐 API Testing**

```javascript
// ============= API ROUTE TESTING =============
// __tests__/api/users.test.js
import { createMocks } from 'node-mocks-http';
import handler from '@/pages/api/users';
import { TestDataGenerator } from '../utils/data-generators';

describe('/api/users', () => {
  beforeEach(() => {
    // Reset database or mocks
    jest.clearAllMocks();
  });

  describe('GET /api/users', () => {
    it('returns list of users', async () => {
      const { req, res } = createMocks({ method: 'GET' });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(200);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(true);
      expect(Array.isArray(data.data)).toBe(true);
    });

    it('supports pagination', async () => {
      const { req, res } = createMocks({
        method: 'GET',
        query: { page: '2', limit: '5' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(200);
      
      const data = JSON.parse(res._getData());
      expect(data.meta.page).toBe(2);
      expect(data.meta.limit).toBe(5);
    });

    it('supports filtering', async () => {
      const { req, res } = createMocks({
        method: 'GET',
        query: { role: 'admin' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(200);
      
      const data = JSON.parse(res._getData());
      data.data.forEach(user => {
        expect(user.role).toBe('admin');
      });
    });

    it('handles invalid query parameters', async () => {
      const { req, res } = createMocks({
        method: 'GET',
        query: { page: 'invalid' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(400);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(false);
      expect(data.error.code).toBe('INVALID_PARAMETER');
    });
  });

  describe('POST /api/users', () => {
    it('creates new user with valid data', async () => {
      const userData = TestDataGenerator.generateFormData({
        name: { type: 'string' },
        email: { type: 'email' },
        role: { type: 'select', options: ['user', 'admin'] },
      });
      
      const { req, res } = createMocks({
        method: 'POST',
        body: userData,
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(201);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(true);
      expect(data.data).toMatchObject(userData);
    });

    it('validates required fields', async () => {
      const { req, res } = createMocks({
        method: 'POST',
        body: { name: 'John' }, // Missing email
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(400);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(false);
      expect(data.error.code).toBe('VALIDATION_ERROR');
      expect(data.error.details).toContain('email');
    });

    it('prevents duplicate email addresses', async () => {
      const userData = {
        name: 'John Doe',
        email: 'existing@example.com',
        role: 'user',
      };
      
      const { req, res } = createMocks({
        method: 'POST',
        body: userData,
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(409);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(false);
      expect(data.error.code).toBe('DUPLICATE_EMAIL');
    });
  });

  describe('PUT /api/users/[id]', () => {
    it('updates existing user', async () => {
      const { req, res } = createMocks({
        method: 'PUT',
        query: { id: '1' },
        body: { name: 'Updated Name' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(200);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(true);
      expect(data.data.name).toBe('Updated Name');
    });

    it('returns 404 for non-existent user', async () => {
      const { req, res } = createMocks({
        method: 'PUT',
        query: { id: '999' },
        body: { name: 'Updated Name' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(404);
      
      const data = JSON.parse(res._getData());
      expect(data.success).toBe(false);
      expect(data.error.code).toBe('USER_NOT_FOUND');
    });
  });

  describe('DELETE /api/users/[id]', () => {
    it('deletes existing user', async () => {
      const { req, res } = createMocks({
        method: 'DELETE',
        query: { id: '1' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(204);
    });

    it('returns 404 for non-existent user', async () => {
      const { req, res } = createMocks({
        method: 'DELETE',
        query: { id: '999' },
      });
      
      await handler(req, res);
      
      expect(res._getStatusCode()).toBe(404);
    });
  });

  it('returns 405 for unsupported methods', async () => {
    const { req, res } = createMocks({ method: 'PATCH' });
    
    await handler(req, res);
    
    expect(res._getStatusCode()).toBe(405);
    
    const data = JSON.parse(res._getData());
    expect(data.success).toBe(false);
    expect(data.error.code).toBe('METHOD_NOT_ALLOWED');
  });
});

// ============= DATABASE INTEGRATION TESTING =============
// __tests__/integration/database.test.js
import { setupTestDB, teardownTestDB, clearTestDB } from '../utils/test-db';
import { UserRepository } from '@/lib/repositories/UserRepository';
import { TestDataGenerator } from '../utils/data-generators';

describe('Database Integration', () => {
  let userRepo;

  beforeAll(async () => {
    await setupTestDB();
    userRepo = new UserRepository();
  });

  afterAll(async () => {
    await teardownTestDB();
  });

  beforeEach(async () => {
    await clearTestDB();
  });

  describe('UserRepository', () => {
    it('creates and retrieves users', async () => {
      const userData = TestDataGenerator.user();
      
      const createdUser = await userRepo.create(userData);
      expect(createdUser).toMatchObject(userData);
      expect(createdUser.id).toBeDefined();
      
      const retrievedUser = await userRepo.findById(createdUser.id);
      expect(retrievedUser).toMatchObject(createdUser);
    });

    it('updates user data', async () => {
      const user = await userRepo.create(TestDataGenerator.user());
      
      const updates = { name: 'Updated Name', email: 'updated@example.com' };
      const updatedUser = await userRepo.update(user.id, updates);
      
      expect(updatedUser.name).toBe(updates.name);
      expect(updatedUser.email).toBe(updates.email);
      expect(updatedUser.updatedAt).not.toBe(user.updatedAt);
    });

    it('deletes users', async () => {
      const user = await userRepo.create(TestDataGenerator.user());
      
      await userRepo.delete(user.id);
      
      const deletedUser = await userRepo.findById(user.id);
      expect(deletedUser).toBeNull();
    });

    it('finds users by criteria', async () => {
      const adminUser = await userRepo.create(
        TestDataGenerator.user({ role: 'admin' })
      );
      const regularUser = await userRepo.create(
        TestDataGenerator.user({ role: 'user' })
      );
      
      const admins = await userRepo.findBy({ role: 'admin' });
      expect(admins).toHaveLength(1);
      expect(admins[0].id).toBe(adminUser.id);
    });

    it('paginates results', async () => {
      // Create multiple users
      const users = await Promise.all(
        Array.from({ length: 15 }, () => 
          userRepo.create(TestDataGenerator.user())
        )
      );
      
      const page1 = await userRepo.findAll({ page: 1, limit: 5 });
      expect(page1.data).toHaveLength(5);
      expect(page1.meta.page).toBe(1);
      expect(page1.meta.total).toBe(15);
      expect(page1.meta.totalPages).toBe(3);
      
      const page2 = await userRepo.findAll({ page: 2, limit: 5 });
      expect(page2.data).toHaveLength(5);
      expect(page2.meta.page).toBe(2);
    });

    it('handles unique constraints', async () => {
      const userData = TestDataGenerator.user();
      await userRepo.create(userData);
      
      // Try to create another user with same email
      await expect(
        userRepo.create({ ...userData, name: 'Different Name' })
      ).rejects.toThrow('Unique constraint violation');
    });

    it('validates data before saving', async () => {
      const invalidUser = {
        name: '', // Empty name
        email: 'invalid-email', // Invalid email format
      };
      
      await expect(userRepo.create(invalidUser)).rejects.toThrow('Validation error');
    });
  });
});

// ============= AUTHENTICATION INTEGRATION =============
// __tests__/integration/auth.test.js
import { renderWithProviders, waitFor } from '../utils/testing-utils';
import { AuthProvider, useAuth } from '@/context/AuthContext';
import { TestDataGenerator } from '../utils/data-generators';

describe('Authentication Integration', () => {
  const TestComponent = () => {
    const { user, login, logout, loading } = useAuth();
    
    return (
      <div>
        {loading && <div data-testid="auth-loading">Loading...</div>}
        {user ? (
          <div>
            <div data-testid="user-name">{user.name}</div>
            <button onClick={logout}>Logout</button>
          </div>
        ) : (
          <button onClick={() => login('test@example.com', 'password')}>
            Login
          </button>
        )}
      </div>
    );
  };

  it('handles successful login', async () => {
    const user = TestDataGenerator.user();
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({ success: true, data: { user, token: 'fake-token' } }),
    });
    
    const { getByText, getByTestId, user: testUser } = renderWithProviders(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    await testUser.click(getByText('Login'));
    
    await waitFor(() => {
      expect(getByTestId('user-name')).toHaveTextContent(user.name);
    });
    
    expect(localStorage.setItem).toHaveBeenCalledWith('token', 'fake-token');
  });

  it('handles login failure', async () => {
    fetch.mockRejectedValueOnce(new Error('Invalid credentials'));
    
    const { getByText, user: testUser } = renderWithProviders(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    await testUser.click(getByText('Login'));
    
    await waitFor(() => {
      expect(getByText(/login failed/i)).toBeInTheDocument();
    });
  });

  it('handles logout', async () => {
    const user = TestDataGenerator.user();
    
    // Mock initial authenticated state
    localStorage.getItem.mockReturnValue('fake-token');
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({ success: true, data: user }),
    });
    
    const { getByText, queryByTestId, user: testUser } = renderWithProviders(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    // Wait for initial auth check
    await waitFor(() => {
      expect(queryByTestId('auth-loading')).not.toBeInTheDocument();
    });
    
    await testUser.click(getByText('Logout'));
    
    expect(localStorage.removeItem).toHaveBeenCalledWith('token');
    expect(getByText('Login')).toBeInTheDocument();
  });

  it('persists authentication across page reloads', async () => {
    const user = TestDataGenerator.user();
    
    localStorage.getItem.mockReturnValue('fake-token');
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({ success: true, data: user }),
    });
    
    const { getByTestId } = renderWithProviders(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    await waitFor(() => {
      expect(getByTestId('user-name')).toHaveTextContent(user.name);
    });
  });
});
```

## 🎭 End-to-End Testing

### **🌍 E2E with Playwright**

```javascript
// ============= PLAYWRIGHT CONFIGURATION =============
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

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

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});

// ============= E2E TEST UTILITIES =============
// e2e/utils/test-helpers.js
import { expect } from '@playwright/test';

export class PageObjectModel {
  constructor(page) {
    this.page = page;
  }

  async navigateTo(path) {
    await this.page.goto(path);
  }

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  async takeScreenshot(name) {
    await this.page.screenshot({ path: `screenshots/${name}.png` });
  }

  async fillForm(formData) {
    for (const [field, value] of Object.entries(formData)) {
      await this.page.fill(`[name="${field}"]`, value);
    }
  }

  async submitForm(selector = 'form') {
    await this.page.click(`${selector} button[type="submit"]`);
  }

  async waitForAlert() {
    await this.page.waitForSelector('[role="alert"]');
  }

  async getAlertText() {
    return await this.page.textContent('[role="alert"]');
  }
}

export class LoginPage extends PageObjectModel {
  constructor(page) {
    super(page);
    this.emailInput = '[name="email"]';
    this.passwordInput = '[name="password"]';
    this.submitButton = 'button[type="submit"]';
    this.errorMessage = '[data-testid="error-message"]';
  }

  async login(email, password) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.submitButton);
  }

  async expectLoginError(message) {
    await expect(this.page.locator(this.errorMessage)).toContainText(message);
  }
}

export class DashboardPage extends PageObjectModel {
  constructor(page) {
    super(page);
    this.userMenu = '[data-testid="user-menu"]';
    this.logoutButton = '[data-testid="logout-button"]';
    this.welcomeMessage = '[data-testid="welcome-message"]';
  }

  async logout() {
    await this.page.click(this.userMenu);
    await this.page.click(this.logoutButton);
  }

  async expectWelcomeMessage(name) {
    await expect(this.page.locator(this.welcomeMessage)).toContainText(`Welcome, ${name}`);
  }
}

// ============= E2E TEST EXAMPLES =============
// e2e/auth.spec.js
import { test, expect } from '@playwright/test';
import { LoginPage, DashboardPage } from './utils/test-helpers';

test.describe('Authentication Flow', () => {
  test('successful login and logout', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    // Navigate to login page
    await loginPage.navigateTo('/login');
    
    // Perform login
    await loginPage.login('test@example.com', 'password');
    
    // Verify redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    await dashboardPage.expectWelcomeMessage('Test User');
    
    // Perform logout
    await dashboardPage.logout();
    
    // Verify redirect to login page
    await expect(page).toHaveURL('/login');
  });

  test('login with invalid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.navigateTo('/login');
    await loginPage.login('invalid@example.com', 'wrongpassword');
    
    await loginPage.expectLoginError('Invalid credentials');
    await expect(page).toHaveURL('/login');
  });

  test('redirects to login when not authenticated', async ({ page }) => {
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login');
  });

  test('persists authentication across page refreshes', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    // Login
    await loginPage.navigateTo('/login');
    await loginPage.login('test@example.com', 'password');
    await expect(page).toHaveURL('/dashboard');
    
    // Refresh page
    await page.reload();
    
    // Should still be on dashboard
    await expect(page).toHaveURL('/dashboard');
    await dashboardPage.expectWelcomeMessage('Test User');
  });
});

// e2e/form-submission.spec.js
import { test, expect } from '@playwright/test';
import { PageObjectModel } from './utils/test-helpers';

test.describe('Contact Form', () => {
  test('submits form successfully', async ({ page }) => {
    const contactPage = new PageObjectModel(page);

    await contactPage.navigateTo('/contact');
    
    await contactPage.fillForm({
      name: 'John Doe',
      email: 'john@example.com',
      message: 'This is a test message',
    });
    
    await contactPage.submitForm();
    
    await contactPage.waitForAlert();
    const alertText = await contactPage.getAlertText();
    expect(alertText).toContain('Message sent successfully');
  });

  test('validates required fields', async ({ page }) => {
    await page.goto('/contact');
    
    // Submit empty form
    await page.click('button[type="submit"]');
    
    // Check for validation errors
    await expect(page.locator('[data-testid="name-error"]')).toBeVisible();
    await expect(page.locator('[data-testid="email-error"]')).toBeVisible();
    await expect(page.locator('[data-testid="message-error"]')).toBeVisible();
  });

  test('validates email format', async ({ page }) => {
    await page.goto('/contact');
    
    await page.fill('[name="email"]', 'invalid-email');
    await page.blur('[name="email"]');
    
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Invalid email format');
  });
});

// e2e/responsive.spec.js
import { test, expect, devices } from '@playwright/test';

test.describe('Responsive Design', () => {
  test('mobile navigation menu', async ({ browser }) => {
    const context = await browser.newContext({
      ...devices['iPhone 12'],
    });
    const page = await context.newPage();

    await page.goto('/');
    
    // Mobile menu should be collapsed initially
    await expect(page.locator('[data-testid="mobile-menu"]')).not.toBeVisible();
    
    // Click hamburger menu
    await page.click('[data-testid="hamburger-menu"]');
    
    // Mobile menu should be visible
    await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();
    
    // Navigation links should be clickable
    await page.click('[data-testid="mobile-menu"] a[href="/about"]');
    await expect(page).toHaveURL('/about');
    
    await context.close();
  });

  test('desktop navigation', async ({ page }) => {
    await page.setViewportSize({ width: 1280, height: 720 });
    await page.goto('/');
    
    // Desktop navigation should be visible
    await expect(page.locator('[data-testid="desktop-nav"]')).toBeVisible();
    
    // Mobile menu button should not be visible
    await expect(page.locator('[data-testid="hamburger-menu"]')).not.toBeVisible();
  });
});

// e2e/performance.spec.js
import { test, expect } from '@playwright/test';

test.describe('Performance', () => {
  test('page load performance', async ({ page }) => {
    const startTime = Date.now();
    
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    
    const loadTime = Date.now() - startTime;
    console.log(`Page load time: ${loadTime}ms`);
    
    // Assert page loads within reasonable time
    expect(loadTime).toBeLessThan(3000);
  });

  test('core web vitals', async ({ page }) => {
    await page.goto('/');
    
    // Measure FCP, LCP, CLS
    const webVitals = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          const vitals = {};
          
          entries.forEach((entry) => {
            if (entry.name === 'first-contentful-paint') {
              vitals.fcp = entry.startTime;
            }
            if (entry.entryType === 'largest-contentful-paint') {
              vitals.lcp = entry.startTime;
            }
          });
          
          if (vitals.fcp && vitals.lcp) {
            resolve(vitals);
          }
        }).observe({ entryTypes: ['paint', 'largest-contentful-paint'] });
      });
    });
    
    console.log('Web Vitals:', webVitals);
    
    // Assert reasonable performance metrics
    expect(webVitals.fcp).toBeLessThan(1800); // FCP < 1.8s
    expect(webVitals.lcp).toBeLessThan(2500); // LCP < 2.5s
  });

  test('lighthouse audit', async ({ page }) => {
    await page.goto('/');
    
    // This would require playwright-lighthouse plugin
    // or integration with lighthouse CI
    
    // For now, just verify key performance indicators
    const performanceMetrics = await page.evaluate(() => {
      const navigation = performance.getEntriesByType('navigation')[0];
      return {
        domContentLoaded: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
        loadComplete: navigation.loadEventEnd - navigation.loadEventStart,
      };
    });
    
    expect(performanceMetrics.domContentLoaded).toBeLessThan(1000);
    expect(performanceMetrics.loadComplete).toBeLessThan(2000);
  });
});

// e2e/accessibility.spec.js
import { test, expect } from '@playwright/test';

test.describe('Accessibility', () => {
  test('keyboard navigation', async ({ page }) => {
    await page.goto('/');
    
    // Tab through navigation
    await page.keyboard.press('Tab');
    await expect(page.locator('a[href="/"]')).toBeFocused();
    
    await page.keyboard.press('Tab');
    await expect(page.locator('a[href="/about"]')).toBeFocused();
    
    // Enter should activate links
    await page.keyboard.press('Enter');
    await expect(page).toHaveURL('/about');
  });

  test('aria labels and roles', async ({ page }) => {
    await page.goto('/');
    
    // Check for proper ARIA attributes
    await expect(page.locator('[role="navigation"]')).toBeVisible();
    await expect(page.locator('[role="main"]')).toBeVisible();
    await expect(page.locator('[aria-label="Main navigation"]')).toBeVisible();
  });

  test('color contrast', async ({ page }) => {
    await page.goto('/');
    
    // This would require axe-core integration
    // For now, verify text is visible
    const textElements = page.locator('p, h1, h2, h3, h4, h5, h6, span');
    const count = await textElements.count();
    
    for (let i = 0; i < count; i++) {
      await expect(textElements.nth(i)).toBeVisible();
    }
  });

  test('screen reader compatibility', async ({ page }) => {
    await page.goto('/');
    
    // Check for proper heading structure
    const h1Count = await page.locator('h1').count();
    expect(h1Count).toBe(1); // Should have exactly one h1
    
    // Check for alt text on images
    const images = page.locator('img');
    const imageCount = await images.count();
    
    for (let i = 0; i < imageCount; i++) {
      const altText = await images.nth(i).getAttribute('alt');
      expect(altText).toBeTruthy();
    }
  });
});
```

## 🎯 แบบฝึกหัด

### **🧪 แบบฝึกหัดที่ 1: Testing Strategy**
สร้าง comprehensive testing strategy:

```javascript
// TODO: สร้าง testing strategy

// 1. Test pyramid implementation
class TestPyramid {
  // Define unit test coverage
  // Integration test scenarios
  // E2E test critical paths
}

// 2. Test data management
class TestDataManager {
  // Factory pattern for test data
  // Database seeding and cleanup
  // Mock data consistency
}

// 3. Test reporting system
class TestReporter {
  // Coverage reports
  // Performance metrics
  // Flaky test detection
}
```

### **🔧 แบบฝึกหัดที่ 2: Advanced Testing**
สร้าง advanced testing patterns:

```javascript
// TODO: สร้าง advanced testing

// 1. Visual regression testing
class VisualRegressionTester {
  // Screenshot comparison
  // CSS change detection
  // Cross-browser compatibility
}

// 2. Contract testing
class ContractTester {
  // API contract validation
  // Schema testing
  // Backward compatibility
}

// 3. Load testing
class LoadTester {
  // Performance under load
  // Stress testing
  // Resource utilization
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Testing**
สร้าง Next.js specific testing:

```jsx
// TODO: สร้าง Next.js testing

// 1. SSR/SSG testing
const SSRTester = () => {
  // Server-side rendering validation
  // Static generation testing
  // Hydration testing
};

// 2. API route testing
const APITester = () => {
  // Middleware testing
  // Authentication testing
  // Error handling testing
};

// 3. Performance testing
const PerformanceTester = () => {
  // Bundle size testing
  // Core Web Vitals
  // Lighthouse CI integration
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 15: Performance Optimization](./15-performance.md)

**บทถัดไป**: [บทที่ 17: Advanced Functions](./17-advanced-functions.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright](https://playwright.dev/)
- [Testing Best Practices](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Testing Fundamentals** - Jest configuration, custom matchers, และ test utilities
- **Unit Testing** - component testing, hook testing, และ async testing
- **Integration Testing** - API testing, database integration, และ authentication flow
- **E2E Testing** - Playwright setup, page object model, และ performance testing
- **Advanced Testing** - accessibility testing, visual regression, และ load testing

Comprehensive testing strategy เป็นพื้นฐานสำคัญของ reliable software development! 🚀
