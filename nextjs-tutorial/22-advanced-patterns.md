# บทที่ 22: Advanced Patterns and Architecture

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Advanced design patterns ใน Next.js
- ทำความเข้าใจ Application architecture patterns
- เรียนรู้ State management patterns
- เข้าใจ Code organization และ scaling strategies

## 🏗️ Application Architecture Patterns

### **🔄 Clean Architecture Pattern**

```typescript
// lib/architecture/entities/User.ts - Domain entities
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  createdAt: Date;
  updatedAt: Date;
}

export interface UserRole {
  id: string;
  name: string;
  permissions: Permission[];
}

export interface Permission {
  id: string;
  resource: string;
  action: string;
}

// Business rules
export class UserEntity {
  constructor(private user: User) {}

  canPerformAction(resource: string, action: string): boolean {
    return this.user.role.permissions.some(
      permission => 
        permission.resource === resource && 
        permission.action === action
    );
  }

  isActive(): boolean {
    return this.user.status === 'active';
  }

  hasRole(roleName: string): boolean {
    return this.user.role.name === roleName;
  }

  updateProfile(updates: Partial<Pick<User, 'name' | 'email'>>): User {
    // Business validation
    if (updates.email && !this.isValidEmail(updates.email)) {
      throw new Error('Invalid email format');
    }

    return {
      ...this.user,
      ...updates,
      updatedAt: new Date()
    };
  }

  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

// lib/architecture/repositories/UserRepository.ts - Data access layer
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(user: Omit<User, 'id' | 'createdAt' | 'updatedAt'>): Promise<User>;
  update(id: string, updates: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;
  findWithPermissions(id: string): Promise<User | null>;
}

export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({
      where: { id },
      include: {
        role: {
          include: {
            permissions: true
          }
        }
      }
    });

    return user ? this.mapToEntity(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({
      where: { email },
      include: {
        role: {
          include: {
            permissions: true
          }
        }
      }
    });

    return user ? this.mapToEntity(user) : null;
  }

  async create(userData: Omit<User, 'id' | 'createdAt' | 'updatedAt'>): Promise<User> {
    const user = await this.prisma.user.create({
      data: {
        ...userData,
        roleId: userData.role.id
      },
      include: {
        role: {
          include: {
            permissions: true
          }
        }
      }
    });

    return this.mapToEntity(user);
  }

  async update(id: string, updates: Partial<User>): Promise<User> {
    const user = await this.prisma.user.update({
      where: { id },
      data: updates,
      include: {
        role: {
          include: {
            permissions: true
          }
        }
      }
    });

    return this.mapToEntity(user);
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({
      where: { id }
    });
  }

  async findWithPermissions(id: string): Promise<User | null> {
    return this.findById(id); // Already includes permissions
  }

  private mapToEntity(prismaUser: any): User {
    return {
      id: prismaUser.id,
      email: prismaUser.email,
      name: prismaUser.name,
      role: {
        id: prismaUser.role.id,
        name: prismaUser.role.name,
        permissions: prismaUser.role.permissions
      },
      createdAt: prismaUser.createdAt,
      updatedAt: prismaUser.updatedAt
    };
  }
}

// lib/architecture/use-cases/UserUseCases.ts - Application layer
export class CreateUserUseCase {
  constructor(
    private userRepository: IUserRepository,
    private emailService: IEmailService,
    private authService: IAuthService
  ) {}

  async execute(request: CreateUserRequest): Promise<CreateUserResponse> {
    // Validation
    if (!request.email || !request.name) {
      throw new Error('Email and name are required');
    }

    // Check if user already exists
    const existingUser = await this.userRepository.findByEmail(request.email);
    if (existingUser) {
      throw new Error('User with this email already exists');
    }

    // Create user entity
    const userData = {
      email: request.email,
      name: request.name,
      role: await this.getDefaultRole(),
      status: 'active' as const
    };

    const user = await this.userRepository.create(userData);
    const userEntity = new UserEntity(user);

    // Send welcome email
    await this.emailService.sendWelcomeEmail(user);

    // Create auth token
    const token = await this.authService.createToken(user);

    return {
      user: user,
      token: token,
      success: true
    };
  }

  private async getDefaultRole(): Promise<UserRole> {
    // Return default user role
    return {
      id: 'default-user',
      name: 'User',
      permissions: [
        { id: '1', resource: 'profile', action: 'read' },
        { id: '2', resource: 'profile', action: 'update' }
      ]
    };
  }
}

// Application service interfaces
export interface CreateUserRequest {
  email: string;
  name: string;
  role?: string;
}

export interface CreateUserResponse {
  user: User;
  token: string;
  success: boolean;
}

export interface IEmailService {
  sendWelcomeEmail(user: User): Promise<void>;
  sendPasswordResetEmail(user: User, resetToken: string): Promise<void>;
}

export interface IAuthService {
  createToken(user: User): Promise<string>;
  verifyToken(token: string): Promise<User | null>;
  hashPassword(password: string): Promise<string>;
  comparePassword(password: string, hash: string): Promise<boolean>;
}

// lib/architecture/container.ts - Dependency injection
export class DIContainer {
  private services = new Map<string, any>();

  register<T>(name: string, factory: () => T): void {
    this.services.set(name, factory);
  }

  resolve<T>(name: string): T {
    const factory = this.services.get(name);
    if (!factory) {
      throw new Error(`Service ${name} not registered`);
    }
    return factory();
  }
}

// Setup DI container
export const container = new DIContainer();

container.register('prisma', () => new PrismaClient());
container.register('userRepository', () => 
  new PrismaUserRepository(container.resolve('prisma'))
);
container.register('emailService', () => new EmailService());
container.register('authService', () => new AuthService());
container.register('createUserUseCase', () => 
  new CreateUserUseCase(
    container.resolve('userRepository'),
    container.resolve('emailService'),
    container.resolve('authService')
  )
);
```

### **🔀 CQRS Pattern Implementation**

```typescript
// lib/cqrs/Command.ts - Command side
export interface ICommand {
  readonly timestamp: Date;
  readonly correlationId: string;
}

export interface ICommandHandler<T extends ICommand> {
  handle(command: T): Promise<void>;
}

export interface ICommandBus {
  send<T extends ICommand>(command: T): Promise<void>;
}

// Commands
export class CreateProductCommand implements ICommand {
  readonly timestamp = new Date();
  readonly correlationId = crypto.randomUUID();

  constructor(
    public readonly name: string,
    public readonly price: number,
    public readonly description: string,
    public readonly categoryId: string,
    public readonly userId: string
  ) {}
}

export class UpdateProductCommand implements ICommand {
  readonly timestamp = new Date();
  readonly correlationId = crypto.randomUUID();

  constructor(
    public readonly id: string,
    public readonly updates: Partial<{
      name: string;
      price: number;
      description: string;
    }>,
    public readonly userId: string
  ) {}
}

// Command handlers
export class CreateProductCommandHandler implements ICommandHandler<CreateProductCommand> {
  constructor(
    private eventStore: IEventStore,
    private productRepository: IProductRepository,
    private eventBus: IEventBus
  ) {}

  async handle(command: CreateProductCommand): Promise<void> {
    // Validation
    if (command.price <= 0) {
      throw new Error('Product price must be positive');
    }

    // Create aggregate
    const product = new ProductAggregate();
    product.create(
      command.name,
      command.price,
      command.description,
      command.categoryId,
      command.userId
    );

    // Save to repository
    await this.productRepository.save(product);

    // Publish events
    const events = product.getUncommittedEvents();
    await Promise.all(
      events.map(event => this.eventBus.publish(event))
    );

    product.markEventsAsCommitted();
  }
}

// lib/cqrs/Query.ts - Query side
export interface IQuery {
  readonly timestamp: Date;
}

export interface IQueryHandler<TQuery extends IQuery, TResult> {
  handle(query: TQuery): Promise<TResult>;
}

export interface IQueryBus {
  send<TQuery extends IQuery, TResult>(query: TQuery): Promise<TResult>;
}

// Queries
export class GetProductByIdQuery implements IQuery {
  readonly timestamp = new Date();

  constructor(public readonly id: string) {}
}

export class GetProductsQuery implements IQuery {
  readonly timestamp = new Date();

  constructor(
    public readonly filters: {
      category?: string;
      priceRange?: { min: number; max: number };
      search?: string;
    } = {},
    public readonly pagination: {
      page: number;
      limit: number;
    } = { page: 1, limit: 10 }
  ) {}
}

// Query handlers
export class GetProductByIdQueryHandler implements IQueryHandler<GetProductByIdQuery, Product | null> {
  constructor(private readModel: IProductReadModel) {}

  async handle(query: GetProductByIdQuery): Promise<Product | null> {
    return this.readModel.findById(query.id);
  }
}

export class GetProductsQueryHandler implements IQueryHandler<GetProductsQuery, PaginatedResult<Product>> {
  constructor(private readModel: IProductReadModel) {}

  async handle(query: GetProductsQuery): Promise<PaginatedResult<Product>> {
    const { filters, pagination } = query;
    
    const products = await this.readModel.findMany({
      where: {
        ...(filters.category && { categoryId: filters.category }),
        ...(filters.priceRange && {
          price: {
            gte: filters.priceRange.min,
            lte: filters.priceRange.max
          }
        }),
        ...(filters.search && {
          OR: [
            { name: { contains: filters.search, mode: 'insensitive' } },
            { description: { contains: filters.search, mode: 'insensitive' } }
          ]
        })
      },
      skip: (pagination.page - 1) * pagination.limit,
      take: pagination.limit
    });

    const total = await this.readModel.count({
      where: {
        ...(filters.category && { categoryId: filters.category }),
        ...(filters.priceRange && {
          price: {
            gte: filters.priceRange.min,
            lte: filters.priceRange.max
          }
        })
      }
    });

    return {
      items: products,
      total,
      page: pagination.page,
      limit: pagination.limit,
      totalPages: Math.ceil(total / pagination.limit)
    };
  }
}

// Bus implementations
export class CommandBus implements ICommandBus {
  private handlers = new Map<string, ICommandHandler<any>>();

  register<T extends ICommand>(
    commandType: new (...args: any[]) => T,
    handler: ICommandHandler<T>
  ): void {
    this.handlers.set(commandType.name, handler);
  }

  async send<T extends ICommand>(command: T): Promise<void> {
    const handler = this.handlers.get(command.constructor.name);
    if (!handler) {
      throw new Error(`No handler registered for command ${command.constructor.name}`);
    }

    await handler.handle(command);
  }
}

export class QueryBus implements IQueryBus {
  private handlers = new Map<string, IQueryHandler<any, any>>();

  register<TQuery extends IQuery, TResult>(
    queryType: new (...args: any[]) => TQuery,
    handler: IQueryHandler<TQuery, TResult>
  ): void {
    this.handlers.set(queryType.name, handler);
  }

  async send<TQuery extends IQuery, TResult>(query: TQuery): Promise<TResult> {
    const handler = this.handlers.get(query.constructor.name);
    if (!handler) {
      throw new Error(`No handler registered for query ${query.constructor.name}`);
    }

    return handler.handle(query);
  }
}

// API integration
// pages/api/products/index.ts
import { container } from '@/lib/architecture/container';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const commandBus = container.resolve<CommandBus>('commandBus');
  const queryBus = container.resolve<QueryBus>('queryBus');

  try {
    if (req.method === 'POST') {
      // Command: Create product
      const command = new CreateProductCommand(
        req.body.name,
        req.body.price,
        req.body.description,
        req.body.categoryId,
        req.user.id
      );

      await commandBus.send(command);
      res.status(201).json({ success: true });

    } else if (req.method === 'GET') {
      // Query: Get products
      const query = new GetProductsQuery(
        {
          category: req.query.category as string,
          search: req.query.search as string,
          priceRange: req.query.minPrice ? {
            min: Number(req.query.minPrice),
            max: Number(req.query.maxPrice)
          } : undefined
        },
        {
          page: Number(req.query.page) || 1,
          limit: Number(req.query.limit) || 10
        }
      );

      const result = await queryBus.send(query);
      res.status(200).json(result);
    }
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## 🎭 Advanced State Management Patterns

### **🗄️ Advanced Zustand Patterns**

```typescript
// lib/store/advanced-patterns.ts - Store composition and middleware
import { create } from 'zustand';
import { devtools, persist, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// Store slice interfaces
interface UserSlice {
  user: User | null;
  isAuthenticated: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  updateProfile: (updates: Partial<User>) => Promise<void>;
}

interface ProductSlice {
  products: Product[];
  selectedProduct: Product | null;
  loading: boolean;
  error: string | null;
  fetchProducts: (filters?: ProductFilters) => Promise<void>;
  selectProduct: (id: string) => void;
  addProduct: (product: CreateProductData) => Promise<void>;
  updateProduct: (id: string, updates: Partial<Product>) => Promise<void>;
  deleteProduct: (id: string) => Promise<void>;
}

interface CartSlice {
  items: CartItem[];
  total: number;
  addItem: (product: Product, quantity: number) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  checkout: () => Promise<void>;
}

interface UISlice {
  theme: 'light' | 'dark';
  sidebar: {
    isOpen: boolean;
    selectedMenu: string | null;
  };
  modal: {
    isOpen: boolean;
    type: string | null;
    data: any;
  };
  toggleTheme: () => void;
  toggleSidebar: () => void;
  openModal: (type: string, data?: any) => void;
  closeModal: () => void;
}

// Combined store type
type StoreState = UserSlice & ProductSlice & CartSlice & UISlice;

// Store slices
const createUserSlice = (
  set: any,
  get: any
): UserSlice => ({
  user: null,
  isAuthenticated: false,
  
  login: async (credentials) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });
      
      if (!response.ok) throw new Error('Login failed');
      
      const { user, token } = await response.json();
      
      set((state: StoreState) => {
        state.user = user;
        state.isAuthenticated = true;
      });
      
      // Store token
      localStorage.setItem('token', token);
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  },
  
  logout: () => {
    set((state: StoreState) => {
      state.user = null;
      state.isAuthenticated = false;
    });
    localStorage.removeItem('token');
  },
  
  updateProfile: async (updates) => {
    const currentUser = get().user;
    if (!currentUser) return;
    
    try {
      const response = await fetch(`/api/users/${currentUser.id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates)
      });
      
      if (!response.ok) throw new Error('Update failed');
      
      const updatedUser = await response.json();
      
      set((state: StoreState) => {
        state.user = updatedUser;
      });
    } catch (error) {
      console.error('Profile update error:', error);
      throw error;
    }
  }
});

const createProductSlice = (
  set: any,
  get: any
): ProductSlice => ({
  products: [],
  selectedProduct: null,
  loading: false,
  error: null,
  
  fetchProducts: async (filters) => {
    set((state: StoreState) => {
      state.loading = true;
      state.error = null;
    });
    
    try {
      const queryParams = new URLSearchParams();
      if (filters?.category) queryParams.set('category', filters.category);
      if (filters?.search) queryParams.set('search', filters.search);
      
      const response = await fetch(`/api/products?${queryParams}`);
      if (!response.ok) throw new Error('Failed to fetch products');
      
      const { items } = await response.json();
      
      set((state: StoreState) => {
        state.products = items;
        state.loading = false;
      });
    } catch (error) {
      set((state: StoreState) => {
        state.error = error.message;
        state.loading = false;
      });
    }
  },
  
  selectProduct: (id) => {
    set((state: StoreState) => {
      state.selectedProduct = state.products.find(p => p.id === id) || null;
    });
  },
  
  addProduct: async (productData) => {
    try {
      const response = await fetch('/api/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(productData)
      });
      
      if (!response.ok) throw new Error('Failed to create product');
      
      const newProduct = await response.json();
      
      set((state: StoreState) => {
        state.products.push(newProduct);
      });
    } catch (error) {
      console.error('Add product error:', error);
      throw error;
    }
  },
  
  updateProduct: async (id, updates) => {
    try {
      const response = await fetch(`/api/products/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates)
      });
      
      if (!response.ok) throw new Error('Failed to update product');
      
      const updatedProduct = await response.json();
      
      set((state: StoreState) => {
        const index = state.products.findIndex(p => p.id === id);
        if (index !== -1) {
          state.products[index] = updatedProduct;
        }
        if (state.selectedProduct?.id === id) {
          state.selectedProduct = updatedProduct;
        }
      });
    } catch (error) {
      console.error('Update product error:', error);
      throw error;
    }
  },
  
  deleteProduct: async (id) => {
    try {
      const response = await fetch(`/api/products/${id}`, {
        method: 'DELETE'
      });
      
      if (!response.ok) throw new Error('Failed to delete product');
      
      set((state: StoreState) => {
        state.products = state.products.filter(p => p.id !== id);
        if (state.selectedProduct?.id === id) {
          state.selectedProduct = null;
        }
      });
    } catch (error) {
      console.error('Delete product error:', error);
      throw error;
    }
  }
});

const createCartSlice = (
  set: any,
  get: any
): CartSlice => ({
  items: [],
  total: 0,
  
  addItem: (product, quantity) => {
    set((state: StoreState) => {
      const existingItem = state.items.find(item => item.product.id === product.id);
      
      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        state.items.push({ product, quantity });
      }
      
      // Recalculate total
      state.total = state.items.reduce(
        (sum, item) => sum + (item.product.price * item.quantity), 
        0
      );
    });
  },
  
  removeItem: (productId) => {
    set((state: StoreState) => {
      state.items = state.items.filter(item => item.product.id !== productId);
      state.total = state.items.reduce(
        (sum, item) => sum + (item.product.price * item.quantity), 
        0
      );
    });
  },
  
  updateQuantity: (productId, quantity) => {
    set((state: StoreState) => {
      const item = state.items.find(item => item.product.id === productId);
      if (item) {
        if (quantity <= 0) {
          state.items = state.items.filter(item => item.product.id !== productId);
        } else {
          item.quantity = quantity;
        }
        
        state.total = state.items.reduce(
          (sum, item) => sum + (item.product.price * item.quantity), 
          0
        );
      }
    });
  },
  
  clearCart: () => {
    set((state: StoreState) => {
      state.items = [];
      state.total = 0;
    });
  },
  
  checkout: async () => {
    const { items, user } = get();
    
    if (!user) throw new Error('User must be logged in');
    if (items.length === 0) throw new Error('Cart is empty');
    
    try {
      const response = await fetch('/api/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ items })
      });
      
      if (!response.ok) throw new Error('Checkout failed');
      
      // Clear cart on successful checkout
      get().clearCart();
    } catch (error) {
      console.error('Checkout error:', error);
      throw error;
    }
  }
});

const createUISlice = (set: any, get: any): UISlice => ({
  theme: 'light',
  sidebar: {
    isOpen: false,
    selectedMenu: null
  },
  modal: {
    isOpen: false,
    type: null,
    data: null
  },
  
  toggleTheme: () => {
    set((state: StoreState) => {
      state.theme = state.theme === 'light' ? 'dark' : 'light';
    });
  },
  
  toggleSidebar: () => {
    set((state: StoreState) => {
      state.sidebar.isOpen = !state.sidebar.isOpen;
    });
  },
  
  openModal: (type, data) => {
    set((state: StoreState) => {
      state.modal = { isOpen: true, type, data };
    });
  },
  
  closeModal: () => {
    set((state: StoreState) => {
      state.modal = { isOpen: false, type: null, data: null };
    });
  }
});

// Create the store with middleware
export const useStore = create<StoreState>()(
  devtools(
    persist(
      subscribeWithSelector(
        immer((set, get) => ({
          ...createUserSlice(set, get),
          ...createProductSlice(set, get),
          ...createCartSlice(set, get),
          ...createUISlice(set, get)
        }))
      ),
      {
        name: 'app-storage',
        partialize: (state) => ({
          user: state.user,
          isAuthenticated: state.isAuthenticated,
          theme: state.theme,
          cart: {
            items: state.items,
            total: state.total
          }
        })
      }
    ),
    { name: 'App Store' }
  )
);

// Selectors
export const useUser = () => useStore(state => state.user);
export const useProducts = () => useStore(state => state.products);
export const useCart = () => useStore(state => ({ 
  items: state.items, 
  total: state.total 
}));
export const useUI = () => useStore(state => ({
  theme: state.theme,
  sidebar: state.sidebar,
  modal: state.modal
}));

// Action selectors
export const useUserActions = () => useStore(state => ({
  login: state.login,
  logout: state.logout,
  updateProfile: state.updateProfile
}));

export const useProductActions = () => useStore(state => ({
  fetchProducts: state.fetchProducts,
  selectProduct: state.selectProduct,
  addProduct: state.addProduct,
  updateProduct: state.updateProduct,
  deleteProduct: state.deleteProduct
}));

export const useCartActions = () => useStore(state => ({
  addItem: state.addItem,
  removeItem: state.removeItem,
  updateQuantity: state.updateQuantity,
  clearCart: state.clearCart,
  checkout: state.checkout
}));
```

### **🔄 Event Sourcing Pattern**

```typescript
// lib/event-sourcing/Event.ts - Event system
export interface IDomainEvent {
  readonly id: string;
  readonly aggregateId: string;
  readonly aggregateType: string;
  readonly eventType: string;
  readonly eventData: any;
  readonly timestamp: Date;
  readonly version: number;
}

export class DomainEvent implements IDomainEvent {
  readonly id: string;
  readonly timestamp: Date;

  constructor(
    public readonly aggregateId: string,
    public readonly aggregateType: string,
    public readonly eventType: string,
    public readonly eventData: any,
    public readonly version: number
  ) {
    this.id = crypto.randomUUID();
    this.timestamp = new Date();
  }
}

// Product events
export class ProductCreatedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    eventData: {
      name: string;
      price: number;
      description: string;
      categoryId: string;
      createdBy: string;
    },
    version: number
  ) {
    super(aggregateId, 'Product', 'ProductCreated', eventData, version);
  }
}

export class ProductUpdatedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    eventData: {
      updates: Partial<Product>;
      updatedBy: string;
    },
    version: number
  ) {
    super(aggregateId, 'Product', 'ProductUpdated', eventData, version);
  }
}

export class ProductDeletedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    eventData: {
      deletedBy: string;
      reason?: string;
    },
    version: number
  ) {
    super(aggregateId, 'Product', 'ProductDeleted', eventData, version);
  }
}

// lib/event-sourcing/Aggregate.ts - Aggregate root
export abstract class AggregateRoot {
  private _uncommittedEvents: IDomainEvent[] = [];
  private _version = 0;

  protected constructor(public readonly id: string) {}

  get version(): number {
    return this._version;
  }

  getUncommittedEvents(): IDomainEvent[] {
    return [...this._uncommittedEvents];
  }

  markEventsAsCommitted(): void {
    this._uncommittedEvents = [];
  }

  protected addEvent(event: IDomainEvent): void {
    this._uncommittedEvents.push(event);
    this._version++;
  }

  loadFromHistory(events: IDomainEvent[]): void {
    events.forEach(event => {
      this.apply(event);
      this._version = event.version;
    });
  }

  protected abstract apply(event: IDomainEvent): void;
}

export class ProductAggregate extends AggregateRoot {
  private _name: string = '';
  private _price: number = 0;
  private _description: string = '';
  private _categoryId: string = '';
  private _isDeleted: boolean = false;
  private _createdBy: string = '';
  private _createdAt: Date | null = null;

  constructor(id?: string) {
    super(id || crypto.randomUUID());
  }

  // Getters
  get name(): string { return this._name; }
  get price(): number { return this._price; }
  get description(): string { return this._description; }
  get categoryId(): string { return this._categoryId; }
  get isDeleted(): boolean { return this._isDeleted; }
  get createdBy(): string { return this._createdBy; }
  get createdAt(): Date | null { return this._createdAt; }

  // Commands
  create(
    name: string,
    price: number,
    description: string,
    categoryId: string,
    createdBy: string
  ): void {
    if (this._createdAt) {
      throw new Error('Product already exists');
    }

    if (!name || name.trim() === '') {
      throw new Error('Product name is required');
    }

    if (price <= 0) {
      throw new Error('Product price must be positive');
    }

    const event = new ProductCreatedEvent(
      this.id,
      { name, price, description, categoryId, createdBy },
      this.version + 1
    );

    this.addEvent(event);
    this.apply(event);
  }

  update(
    updates: Partial<{
      name: string;
      price: number;
      description: string;
    }>,
    updatedBy: string
  ): void {
    if (this._isDeleted) {
      throw new Error('Cannot update deleted product');
    }

    if (!this._createdAt) {
      throw new Error('Product does not exist');
    }

    if (updates.price !== undefined && updates.price <= 0) {
      throw new Error('Product price must be positive');
    }

    const event = new ProductUpdatedEvent(
      this.id,
      { updates, updatedBy },
      this.version + 1
    );

    this.addEvent(event);
    this.apply(event);
  }

  delete(deletedBy: string, reason?: string): void {
    if (this._isDeleted) {
      throw new Error('Product is already deleted');
    }

    if (!this._createdAt) {
      throw new Error('Product does not exist');
    }

    const event = new ProductDeletedEvent(
      this.id,
      { deletedBy, reason },
      this.version + 1
    );

    this.addEvent(event);
    this.apply(event);
  }

  // Event application
  protected apply(event: IDomainEvent): void {
    switch (event.eventType) {
      case 'ProductCreated':
        this.applyProductCreated(event as ProductCreatedEvent);
        break;
      case 'ProductUpdated':
        this.applyProductUpdated(event as ProductUpdatedEvent);
        break;
      case 'ProductDeleted':
        this.applyProductDeleted(event as ProductDeletedEvent);
        break;
      default:
        throw new Error(`Unknown event type: ${event.eventType}`);
    }
  }

  private applyProductCreated(event: ProductCreatedEvent): void {
    this._name = event.eventData.name;
    this._price = event.eventData.price;
    this._description = event.eventData.description;
    this._categoryId = event.eventData.categoryId;
    this._createdBy = event.eventData.createdBy;
    this._createdAt = event.timestamp;
  }

  private applyProductUpdated(event: ProductUpdatedEvent): void {
    const { updates } = event.eventData;
    
    if (updates.name !== undefined) {
      this._name = updates.name;
    }
    if (updates.price !== undefined) {
      this._price = updates.price;
    }
    if (updates.description !== undefined) {
      this._description = updates.description;
    }
  }

  private applyProductDeleted(event: ProductDeletedEvent): void {
    this._isDeleted = true;
  }

  toSnapshot(): ProductSnapshot {
    return {
      id: this.id,
      name: this._name,
      price: this._price,
      description: this._description,
      categoryId: this._categoryId,
      isDeleted: this._isDeleted,
      createdBy: this._createdBy,
      createdAt: this._createdAt,
      version: this.version
    };
  }
}

export interface ProductSnapshot {
  id: string;
  name: string;
  price: number;
  description: string;
  categoryId: string;
  isDeleted: boolean;
  createdBy: string;
  createdAt: Date | null;
  version: number;
}

// lib/event-sourcing/EventStore.ts - Event storage
export interface IEventStore {
  saveEvents(aggregateId: string, events: IDomainEvent[], expectedVersion: number): Promise<void>;
  getEvents(aggregateId: string, fromVersion?: number): Promise<IDomainEvent[]>;
  getAllEvents(fromTimestamp?: Date): Promise<IDomainEvent[]>;
}

export class PrismaEventStore implements IEventStore {
  constructor(private prisma: PrismaClient) {}

  async saveEvents(
    aggregateId: string,
    events: IDomainEvent[],
    expectedVersion: number
  ): Promise<void> {
    const transaction = await this.prisma.$transaction(async (tx) => {
      // Check for concurrency conflicts
      const lastEvent = await tx.event.findFirst({
        where: { aggregateId },
        orderBy: { version: 'desc' }
      });

      if (lastEvent && lastEvent.version !== expectedVersion) {
        throw new Error('Concurrency conflict detected');
      }

      // Save events
      for (const event of events) {
        await tx.event.create({
          data: {
            id: event.id,
            aggregateId: event.aggregateId,
            aggregateType: event.aggregateType,
            eventType: event.eventType,
            eventData: JSON.stringify(event.eventData),
            timestamp: event.timestamp,
            version: event.version
          }
        });
      }
    });
  }

  async getEvents(aggregateId: string, fromVersion = 0): Promise<IDomainEvent[]> {
    const events = await this.prisma.event.findMany({
      where: {
        aggregateId,
        version: { gt: fromVersion }
      },
      orderBy: { version: 'asc' }
    });

    return events.map(event => new DomainEvent(
      event.aggregateId,
      event.aggregateType,
      event.eventType,
      JSON.parse(event.eventData),
      event.version
    ));
  }

  async getAllEvents(fromTimestamp?: Date): Promise<IDomainEvent[]> {
    const events = await this.prisma.event.findMany({
      where: fromTimestamp ? {
        timestamp: { gte: fromTimestamp }
      } : {},
      orderBy: { timestamp: 'asc' }
    });

    return events.map(event => new DomainEvent(
      event.aggregateId,
      event.aggregateType,
      event.eventType,
      JSON.parse(event.eventData),
      event.version
    ));
  }
}

// Repository using event sourcing
export class EventSourcedProductRepository {
  constructor(private eventStore: IEventStore) {}

  async findById(id: string): Promise<ProductAggregate | null> {
    const events = await this.eventStore.getEvents(id);
    
    if (events.length === 0) {
      return null;
    }

    const aggregate = new ProductAggregate(id);
    aggregate.loadFromHistory(events);
    
    return aggregate;
  }

  async save(aggregate: ProductAggregate): Promise<void> {
    const events = aggregate.getUncommittedEvents();
    
    if (events.length === 0) {
      return;
    }

    await this.eventStore.saveEvents(
      aggregate.id,
      events,
      aggregate.version - events.length
    );

    aggregate.markEventsAsCommitted();
  }
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Clean Architecture**
1. สร้าง domain entities และ business rules
2. ตั้งค่า repository pattern
3. เพิ่ม use cases และ dependency injection
4. ทดสอบ architecture layers

### **🎯 แบบฝึกหัดที่ 2: CQRS Implementation**
1. สร้าง command และ query handlers
2. ตั้งค่า command และ query buses
3. เพิ่ม read/write model separation
4. ทดสอบ CQRS flow

### **🎯 แบบฝึกหัดที่ 3: Advanced State Management**
1. สร้าง store slices และ composition
2. เพิ่ม middleware และ persistence
3. ตั้งค่า selectors และ actions
4. ทดสอบ complex state interactions

### **🎯 แบบฝึกหัดที่ 4: Event Sourcing**
1. สร้าง domain events และ aggregates
2. ตั้งค่า event store
3. เพิ่ม event replay และ snapshots
4. ทดสอบ event sourcing patterns

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 21: Monitoring and Analytics](./21-monitoring-analytics.md)

**บทต่อไป**: [บทที่ 23: Real-world Projects](./23-real-world-projects.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)
- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Domain-Driven Design](https://domainlanguage.com/ddd/)
