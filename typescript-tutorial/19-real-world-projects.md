# บทที่ 19: Real-world Projects

## 🎯 จุดประสงค์ของบทเรียน
- นำความรู้ TypeScript ทั้งหมดมาใช้ในโปรเจ็กต์จริง
- เรียนรู้การสร้างแอปพลิเคชันที่สมบูรณ์
- เข้าใจการทำงานร่วมกันของ Frontend และ Backend
- เรียนรู้การ Deploy และ Maintenance

## 🛍️ โปรเจ็กต์ที่ 1: E-commerce Platform

### 1. **Project Setup และ Architecture**

```typescript
// package.json
{
  "name": "ecommerce-platform",
  "version": "1.0.0",
  "scripts": {
    "dev": "concurrently \"npm run dev:server\" \"npm run dev:client\"",
    "dev:server": "ts-node-dev --respawn --transpile-only src/server/index.ts",
    "dev:client": "vite",
    "build": "npm run build:server && npm run build:client",
    "build:server": "tsc -p tsconfig.server.json",
    "build:client": "vite build",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src --ext .ts,.tsx",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "express": "^4.18.2",
    "prisma": "^5.0.0",
    "@prisma/client": "^5.0.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0",
    "zod": "^3.21.4",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.11.0",
    "react-query": "^3.39.3",
    "axios": "^1.4.0",
    "zustand": "^4.3.8"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0",
    "@types/express": "^4.17.17",
    "@types/bcryptjs": "^2.4.2",
    "@types/jsonwebtoken": "^9.0.2",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "ts-node-dev": "^2.0.0",
    "vite": "^4.3.0",
    "@vitejs/plugin-react": "^4.0.0",
    "jest": "^29.5.0",
    "@types/jest": "^29.5.0",
    "eslint": "^8.42.0",
    "@typescript-eslint/eslint-plugin": "^5.59.0",
    "concurrently": "^8.2.0"
  }
}

// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/server/*": ["./src/server/*"],
      "@/client/*": ["./src/client/*"],
      "@/shared/*": ["./src/shared/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}

// Project structure
/*
src/
├── shared/              # Shared types and utilities
│   ├── types/
│   │   ├── user.ts
│   │   ├── product.ts
│   │   ├── order.ts
│   │   └── api.ts
│   ├── utils/
│   │   ├── validation.ts
│   │   ├── constants.ts
│   │   └── helpers.ts
│   └── schemas/         # Zod schemas
│       ├── user.schema.ts
│       ├── product.schema.ts
│       └── order.schema.ts
├── server/              # Backend (Express + Prisma)
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   ├── routes/
│   ├── database/
│   └── index.ts
└── client/              # Frontend (React + Vite)
    ├── components/
    ├── pages/
    ├── hooks/
    ├── stores/
    ├── services/
    └── main.tsx
*/
```

### 2. **Shared Types และ Schemas**

```typescript
// src/shared/types/user.ts
export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export enum UserRole {
  CUSTOMER = 'customer',
  ADMIN = 'admin',
  SELLER = 'seller'
}

export interface CreateUserRequest {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  role?: UserRole;
}

export interface LoginRequest {
  email: string;
  password: string;
}

export interface AuthResponse {
  user: Omit<User, 'password'>;
  token: string;
  refreshToken: string;
}

// src/shared/types/product.ts
export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  images: string[];
  stock: number;
  sellerId: string;
  seller: Pick<User, 'id' | 'firstName' | 'lastName'>;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateProductRequest {
  name: string;
  description: string;
  price: number;
  category: string;
  images: string[];
  stock: number;
}

export interface UpdateProductRequest {
  name?: string;
  description?: string;
  price?: number;
  category?: string;
  images?: string[];
  stock?: number;
  isActive?: boolean;
}

export interface ProductFilters {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  search?: string;
  sellerId?: string;
}

export interface ProductSearchResult {
  products: Product[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

// src/shared/types/order.ts
export interface Order {
  id: string;
  customerId: string;
  customer: Pick<User, 'id' | 'firstName' | 'lastName' | 'email'>;
  items: OrderItem[];
  subtotal: number;
  tax: number;
  shipping: number;
  total: number;
  status: OrderStatus;
  shippingAddress: Address;
  billingAddress: Address;
  paymentMethod: PaymentMethod;
  paymentStatus: PaymentStatus;
  createdAt: Date;
  updatedAt: Date;
}

export interface OrderItem {
  id: string;
  productId: string;
  product: Pick<Product, 'id' | 'name' | 'price' | 'images'>;
  quantity: number;
  price: number;
  total: number;
}

export enum OrderStatus {
  PENDING = 'pending',
  CONFIRMED = 'confirmed',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled'
}

export enum PaymentStatus {
  PENDING = 'pending',
  PAID = 'paid',
  FAILED = 'failed',
  REFUNDED = 'refunded'
}

export interface Address {
  street: string;
  city: string;
  state: string;
  zipCode: string;
  country: string;
}

export interface PaymentMethod {
  type: 'credit_card' | 'paypal' | 'bank_transfer';
  details: Record<string, any>;
}

export interface CreateOrderRequest {
  items: Array<{
    productId: string;
    quantity: number;
  }>;
  shippingAddress: Address;
  billingAddress: Address;
  paymentMethod: PaymentMethod;
}

// src/shared/schemas/user.schema.ts
import { z } from 'zod';
import { UserRole } from '../types/user';

export const createUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
      'Password must contain uppercase, lowercase, number and special character'
    ),
  firstName: z.string().min(1, 'First name is required').max(50),
  lastName: z.string().min(1, 'Last name is required').max(50),
  role: z.nativeEnum(UserRole).optional()
});

export const loginSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(1, 'Password is required')
});

// src/shared/schemas/product.schema.ts
import { z } from 'zod';

export const createProductSchema = z.object({
  name: z.string().min(1, 'Product name is required').max(200),
  description: z.string().min(1, 'Description is required').max(2000),
  price: z.number().positive('Price must be positive'),
  category: z.string().min(1, 'Category is required'),
  images: z.array(z.string().url('Invalid image URL')).min(1, 'At least one image is required'),
  stock: z.number().int().min(0, 'Stock cannot be negative')
});

export const updateProductSchema = createProductSchema.partial().extend({
  isActive: z.boolean().optional()
});

export const productFiltersSchema = z.object({
  category: z.string().optional(),
  minPrice: z.number().positive().optional(),
  maxPrice: z.number().positive().optional(),
  search: z.string().optional(),
  sellerId: z.string().uuid().optional(),
  page: z.number().int().positive().default(1),
  pageSize: z.number().int().positive().max(100).default(20)
});
```

### 3. **Backend Implementation**

```typescript
// src/server/database/prisma.ts
import { PrismaClient } from '@prisma/client';

export const prisma = new PrismaClient({
  log: ['query', 'error', 'warn'],
  errorFormat: 'pretty'
});

export async function connectDatabase(): Promise<void> {
  try {
    await prisma.$connect();
    console.log('Database connected successfully');
  } catch (error) {
    console.error('Database connection failed:', error);
    process.exit(1);
  }
}

export async function disconnectDatabase(): Promise<void> {
  await prisma.$disconnect();
}

// src/server/services/auth.service.ts
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { User, CreateUserRequest, LoginRequest, AuthResponse } from '@/shared/types/user';
import { UserRepository } from '../repositories/user.repository';
import { ConflictError, UnauthorizedError, ValidationError } from '../utils/errors';

export class AuthService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly jwtSecret: string,
    private readonly jwtRefreshSecret: string
  ) {}

  async register(userData: CreateUserRequest): Promise<AuthResponse> {
    // Check if user already exists
    const existingUser = await this.userRepository.findByEmail(userData.email);
    if (existingUser) {
      throw new ConflictError('User with this email already exists');
    }

    // Hash password
    const passwordHash = await bcrypt.hash(userData.password, 12);

    // Create user
    const user = await this.userRepository.create({
      ...userData,
      password: passwordHash
    });

    // Generate tokens
    const { token, refreshToken } = this.generateTokens(user);

    return {
      user: this.sanitizeUser(user),
      token,
      refreshToken
    };
  }

  async login(credentials: LoginRequest): Promise<AuthResponse> {
    // Find user
    const user = await this.userRepository.findByEmail(credentials.email);
    if (!user) {
      throw new UnauthorizedError('Invalid credentials');
    }

    // Verify password
    const isValidPassword = await bcrypt.compare(credentials.password, user.password);
    if (!isValidPassword) {
      throw new UnauthorizedError('Invalid credentials');
    }

    // Check if user is active
    if (!user.isActive) {
      throw new UnauthorizedError('Account is deactivated');
    }

    // Generate tokens
    const { token, refreshToken } = this.generateTokens(user);

    return {
      user: this.sanitizeUser(user),
      token,
      refreshToken
    };
  }

  async refreshToken(refreshToken: string): Promise<{ token: string; refreshToken: string }> {
    try {
      const payload = jwt.verify(refreshToken, this.jwtRefreshSecret) as { userId: string };
      const user = await this.userRepository.findById(payload.userId);
      
      if (!user || !user.isActive) {
        throw new UnauthorizedError('Invalid refresh token');
      }

      return this.generateTokens(user);
    } catch (error) {
      throw new UnauthorizedError('Invalid refresh token');
    }
  }

  async verifyToken(token: string): Promise<User> {
    try {
      const payload = jwt.verify(token, this.jwtSecret) as { userId: string };
      const user = await this.userRepository.findById(payload.userId);
      
      if (!user || !user.isActive) {
        throw new UnauthorizedError('Invalid token');
      }

      return user;
    } catch (error) {
      throw new UnauthorizedError('Invalid token');
    }
  }

  private generateTokens(user: User): { token: string; refreshToken: string } {
    const payload = { userId: user.id, email: user.email, role: user.role };
    
    const token = jwt.sign(payload, this.jwtSecret, { expiresIn: '1h' });
    const refreshToken = jwt.sign({ userId: user.id }, this.jwtRefreshSecret, { expiresIn: '7d' });
    
    return { token, refreshToken };
  }

  private sanitizeUser(user: User): Omit<User, 'password'> {
    const { password, ...sanitizedUser } = user as any;
    return sanitizedUser;
  }
}

// src/server/services/product.service.ts
import { Product, CreateProductRequest, UpdateProductRequest, ProductFilters, ProductSearchResult } from '@/shared/types/product';
import { ProductRepository } from '../repositories/product.repository';
import { NotFoundError, ForbiddenError } from '../utils/errors';

export class ProductService {
  constructor(private readonly productRepository: ProductRepository) {}

  async createProduct(sellerId: string, productData: CreateProductRequest): Promise<Product> {
    return this.productRepository.create(sellerId, productData);
  }

  async getProductById(id: string): Promise<Product> {
    const product = await this.productRepository.findById(id);
    if (!product) {
      throw new NotFoundError('Product not found');
    }
    return product;
  }

  async updateProduct(id: string, sellerId: string, updateData: UpdateProductRequest): Promise<Product> {
    const product = await this.getProductById(id);
    
    if (product.sellerId !== sellerId) {
      throw new ForbiddenError('You can only update your own products');
    }

    return this.productRepository.update(id, updateData);
  }

  async deleteProduct(id: string, sellerId: string): Promise<void> {
    const product = await this.getProductById(id);
    
    if (product.sellerId !== sellerId) {
      throw new ForbiddenError('You can only delete your own products');
    }

    await this.productRepository.delete(id);
  }

  async searchProducts(filters: ProductFilters): Promise<ProductSearchResult> {
    return this.productRepository.search(filters);
  }

  async getSellerProducts(sellerId: string, page: number = 1, pageSize: number = 20): Promise<ProductSearchResult> {
    return this.productRepository.search({
      sellerId,
      page,
      pageSize
    });
  }

  async updateStock(id: string, quantity: number): Promise<void> {
    const product = await this.getProductById(id);
    
    if (product.stock < quantity) {
      throw new ValidationError('Insufficient stock');
    }

    await this.productRepository.updateStock(id, product.stock - quantity);
  }
}

// src/server/services/order.service.ts
import { Order, CreateOrderRequest, OrderStatus, PaymentStatus } from '@/shared/types/order';
import { OrderRepository } from '../repositories/order.repository';
import { ProductService } from './product.service';
import { PaymentService } from './payment.service';
import { NotFoundError, ValidationError } from '../utils/errors';

export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly productService: ProductService,
    private readonly paymentService: PaymentService
  ) {}

  async createOrder(customerId: string, orderData: CreateOrderRequest): Promise<Order> {
    // Validate products and calculate totals
    let subtotal = 0;
    const validatedItems = [];

    for (const item of orderData.items) {
      const product = await this.productService.getProductById(item.productId);
      
      if (!product.isActive) {
        throw new ValidationError(`Product ${product.name} is not available`);
      }
      
      if (product.stock < item.quantity) {
        throw new ValidationError(`Insufficient stock for ${product.name}`);
      }

      const itemTotal = product.price * item.quantity;
      subtotal += itemTotal;

      validatedItems.push({
        productId: product.id,
        quantity: item.quantity,
        price: product.price,
        total: itemTotal
      });
    }

    // Calculate tax and shipping
    const tax = subtotal * 0.08; // 8% tax
    const shipping = subtotal > 100 ? 0 : 10; // Free shipping over $100
    const total = subtotal + tax + shipping;

    // Create order
    const order = await this.orderRepository.create({
      customerId,
      items: validatedItems,
      subtotal,
      tax,
      shipping,
      total,
      status: OrderStatus.PENDING,
      shippingAddress: orderData.shippingAddress,
      billingAddress: orderData.billingAddress,
      paymentMethod: orderData.paymentMethod,
      paymentStatus: PaymentStatus.PENDING
    });

    // Update product stock
    for (const item of orderData.items) {
      await this.productService.updateStock(item.productId, item.quantity);
    }

    // Process payment
    try {
      const paymentResult = await this.paymentService.processPayment(
        order.id,
        total,
        orderData.paymentMethod
      );

      if (paymentResult.success) {
        await this.orderRepository.updatePaymentStatus(order.id, PaymentStatus.PAID);
        await this.orderRepository.updateStatus(order.id, OrderStatus.CONFIRMED);
      } else {
        await this.orderRepository.updatePaymentStatus(order.id, PaymentStatus.FAILED);
        throw new ValidationError('Payment failed: ' + paymentResult.error);
      }
    } catch (error) {
      // Rollback stock if payment fails
      for (const item of orderData.items) {
        const product = await this.productService.getProductById(item.productId);
        await this.productService.updateStock(item.productId, -item.quantity);
      }
      throw error;
    }

    return this.orderRepository.findById(order.id)!;
  }

  async getOrder(id: string, userId: string): Promise<Order> {
    const order = await this.orderRepository.findById(id);
    if (!order) {
      throw new NotFoundError('Order not found');
    }

    if (order.customerId !== userId) {
      throw new ForbiddenError('You can only view your own orders');
    }

    return order;
  }

  async getUserOrders(userId: string, page: number = 1, pageSize: number = 20): Promise<{
    orders: Order[];
    total: number;
    page: number;
    pageSize: number;
    totalPages: number;
  }> {
    return this.orderRepository.findByCustomerId(userId, page, pageSize);
  }

  async updateOrderStatus(id: string, status: OrderStatus): Promise<Order> {
    const order = await this.orderRepository.findById(id);
    if (!order) {
      throw new NotFoundError('Order not found');
    }

    await this.orderRepository.updateStatus(id, status);
    return this.orderRepository.findById(id)!;
  }
}

// src/server/controllers/auth.controller.ts
import { Request, Response } from 'express';
import { AuthService } from '../services/auth.service';
import { createUserSchema, loginSchema } from '@/shared/schemas/user.schema';
import { ValidationError } from '../utils/errors';

export class AuthController {
  constructor(private readonly authService: AuthService) {}

  register = async (req: Request, res: Response): Promise<void> => {
    try {
      const validatedData = createUserSchema.parse(req.body);
      const result = await this.authService.register(validatedData);
      
      res.status(201).json({
        success: true,
        data: result,
        message: 'User registered successfully'
      });
    } catch (error) {
      this.handleError(error, res);
    }
  };

  login = async (req: Request, res: Response): Promise<void> => {
    try {
      const validatedData = loginSchema.parse(req.body);
      const result = await this.authService.login(validatedData);
      
      res.json({
        success: true,
        data: result,
        message: 'Login successful'
      });
    } catch (error) {
      this.handleError(error, res);
    }
  };

  refreshToken = async (req: Request, res: Response): Promise<void> => {
    try {
      const { refreshToken } = req.body;
      
      if (!refreshToken) {
        throw new ValidationError('Refresh token is required');
      }

      const result = await this.authService.refreshToken(refreshToken);
      
      res.json({
        success: true,
        data: result,
        message: 'Token refreshed successfully'
      });
    } catch (error) {
      this.handleError(error, res);
    }
  };

  private handleError(error: any, res: Response): void {
    if (error.name === 'ZodError') {
      res.status(400).json({
        success: false,
        error: 'Validation Error',
        message: 'Invalid input data',
        details: error.errors
      });
    } else if (error instanceof ValidationError) {
      res.status(400).json({
        success: false,
        error: 'Validation Error',
        message: error.message
      });
    } else if (error instanceof ConflictError) {
      res.status(409).json({
        success: false,
        error: 'Conflict Error',
        message: error.message
      });
    } else if (error instanceof UnauthorizedError) {
      res.status(401).json({
        success: false,
        error: 'Unauthorized',
        message: error.message
      });
    } else {
      console.error('Unexpected error:', error);
      res.status(500).json({
        success: false,
        error: 'Internal Server Error',
        message: 'An unexpected error occurred'
      });
    }
  }
}
```

### 4. **Frontend Implementation**

```typescript
// src/client/services/api.service.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';
import { AuthResponse } from '@/shared/types/user';

class ApiService {
  private client: AxiosInstance;
  private token: string | null = null;

  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    this.setupInterceptors();
    this.loadTokenFromStorage();
  }

  private setupInterceptors(): void {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        if (this.token) {
          config.headers.Authorization = `Bearer ${this.token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response,
      async (error) => {
        if (error.response?.status === 401 && this.token) {
          // Try to refresh token
          try {
            await this.refreshToken();
            // Retry original request
            return this.client.request(error.config);
          } catch (refreshError) {
            this.clearToken();
            window.location.href = '/login';
          }
        }
        return Promise.reject(error);
      }
    );
  }

  setToken(token: string): void {
    this.token = token;
    localStorage.setItem('authToken', token);
  }

  clearToken(): void {
    this.token = null;
    localStorage.removeItem('authToken');
    localStorage.removeItem('refreshToken');
  }

  private loadTokenFromStorage(): void {
    const token = localStorage.getItem('authToken');
    if (token) {
      this.token = token;
    }
  }

  private async refreshToken(): Promise<void> {
    const refreshToken = localStorage.getItem('refreshToken');
    if (!refreshToken) {
      throw new Error('No refresh token available');
    }

    const response = await this.client.post('/auth/refresh', { refreshToken });
    const { token, refreshToken: newRefreshToken } = response.data.data;
    
    this.setToken(token);
    localStorage.setItem('refreshToken', newRefreshToken);
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.get(url, config);
    return response.data;
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.post(url, data, config);
    return response.data;
  }

  async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.put(url, data, config);
    return response.data;
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.delete(url, config);
    return response.data;
  }
}

export const apiService = new ApiService(import.meta.env.VITE_API_URL || 'http://localhost:3000/api');

// src/client/services/auth.service.ts
import { CreateUserRequest, LoginRequest, AuthResponse, User } from '@/shared/types/user';
import { apiService } from './api.service';

export class AuthService {
  async register(userData: CreateUserRequest): Promise<AuthResponse> {
    const response = await apiService.post<{ data: AuthResponse }>('/auth/register', userData);
    this.handleAuthResponse(response.data);
    return response.data;
  }

  async login(credentials: LoginRequest): Promise<AuthResponse> {
    const response = await apiService.post<{ data: AuthResponse }>('/auth/login', credentials);
    this.handleAuthResponse(response.data);
    return response.data;
  }

  async logout(): Promise<void> {
    apiService.clearToken();
  }

  async getCurrentUser(): Promise<User | null> {
    try {
      const response = await apiService.get<{ data: User }>('/auth/me');
      return response.data;
    } catch (error) {
      return null;
    }
  }

  private handleAuthResponse(authResponse: AuthResponse): void {
    apiService.setToken(authResponse.token);
    localStorage.setItem('refreshToken', authResponse.refreshToken);
  }

  isAuthenticated(): boolean {
    return !!localStorage.getItem('authToken');
  }
}

export const authService = new AuthService();

// src/client/stores/auth.store.ts
import { create } from 'zustand';
import { User } from '@/shared/types/user';
import { authService } from '../services/auth.service';

interface AuthState {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  register: (userData: CreateUserRequest) => Promise<void>;
  logout: () => Promise<void>;
  loadUser: () => Promise<void>;
}

export const useAuthStore = create<AuthState>((set, get) => ({
  user: null,
  isLoading: false,
  isAuthenticated: false,

  login: async (email: string, password: string) => {
    set({ isLoading: true });
    try {
      const response = await authService.login({ email, password });
      set({
        user: response.user,
        isAuthenticated: true,
        isLoading: false
      });
    } catch (error) {
      set({ isLoading: false });
      throw error;
    }
  },

  register: async (userData: CreateUserRequest) => {
    set({ isLoading: true });
    try {
      const response = await authService.register(userData);
      set({
        user: response.user,
        isAuthenticated: true,
        isLoading: false
      });
    } catch (error) {
      set({ isLoading: false });
      throw error;
    }
  },

  logout: async () => {
    await authService.logout();
    set({
      user: null,
      isAuthenticated: false
    });
  },

  loadUser: async () => {
    if (!authService.isAuthenticated()) {
      return;
    }

    set({ isLoading: true });
    try {
      const user = await authService.getCurrentUser();
      set({
        user,
        isAuthenticated: !!user,
        isLoading: false
      });
    } catch (error) {
      set({
        user: null,
        isAuthenticated: false,
        isLoading: false
      });
    }
  }
}));

// src/client/components/LoginForm.tsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuthStore } from '../stores/auth.store';

export const LoginForm: React.FC = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [error, setError] = useState<string | null>(null);
  
  const { login, isLoading } = useAuthStore();
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);

    try {
      await login(formData.email, formData.password);
      navigate('/dashboard');
    } catch (error: any) {
      setError(error.response?.data?.message || 'Login failed');
    }
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData({
      ...formData,
      [e.target.name]: e.target.value
    });
  };

  return (
    <div className="max-w-md mx-auto mt-8 p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-6 text-center">Login</h2>
      
      {error && (
        <div className="mb-4 p-3 bg-red-100 border border-red-400 text-red-700 rounded">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit}>
        <div className="mb-4">
          <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-2">
            Email
          </label>
          <input
            type="email"
            id="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            required
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div className="mb-6">
          <label htmlFor="password" className="block text-sm font-medium text-gray-700 mb-2">
            Password
          </label>
          <input
            type="password"
            id="password"
            name="password"
            value={formData.password}
            onChange={handleChange}
            required
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <button
          type="submit"
          disabled={isLoading}
          className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50"
        >
          {isLoading ? 'Logging in...' : 'Login'}
        </button>
      </form>
    </div>
  );
};

// src/client/components/ProductCard.tsx
import React from 'react';
import { Product } from '@/shared/types/product';

interface ProductCardProps {
  product: Product;
  onAddToCart: (productId: string) => void;
}

export const ProductCard: React.FC<ProductCardProps> = ({ product, onAddToCart }) => {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
      <img
        src={product.images[0]}
        alt={product.name}
        className="w-full h-48 object-cover"
      />
      
      <div className="p-4">
        <h3 className="text-lg font-semibold text-gray-900 mb-2">{product.name}</h3>
        <p className="text-gray-600 text-sm mb-3 line-clamp-2">{product.description}</p>
        
        <div className="flex items-center justify-between">
          <span className="text-2xl font-bold text-blue-600">
            ${product.price.toFixed(2)}
          </span>
          
          <div className="flex items-center space-x-2">
            <span className="text-sm text-gray-500">
              Stock: {product.stock}
            </span>
            
            <button
              onClick={() => onAddToCart(product.id)}
              disabled={product.stock === 0}
              className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {product.stock === 0 ? 'Out of Stock' : 'Add to Cart'}
            </button>
          </div>
        </div>
        
        <div className="mt-3 flex items-center justify-between text-sm text-gray-500">
          <span>Category: {product.category}</span>
          <span>By: {product.seller.firstName} {product.seller.lastName}</span>
        </div>
      </div>
    </div>
  );
};
```

## 📝 แบบฝึกหัด

### 1. **เพิ่มฟีเจอร์ Wishlist**
- สร้าง API endpoints สำหรับ wishlist
- เพิ่ม UI components สำหรับจัดการ wishlist
- เพิ่ม state management สำหรับ wishlist

### 2. **สร้างระบบ Reviews และ Ratings**
- ออกแบบ database schema สำหรับ reviews
- สร้าง API สำหรับ CRUD operations
- สร้าง UI สำหรับแสดงและเขียน reviews

### 3. **เพิ่มระบบ Real-time Notifications**
- ใช้ WebSocket สำหรับ real-time updates
- สร้างระบบ notification ใน frontend
- เพิ่มการแจ้งเตือนสำหรับ order status updates

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 18: Best Practices](./18-best-practices.md)

**บทต่อไป**: [บทที่ 20: Expert Advice](./20-expert-advice.md)

**กลับไปหน้าหลัก**: [README](./README.md)
