# บทที่ 18: Best Practices

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Best Practices ในการเขียน TypeScript
- เข้าใจ Code Quality และ Maintainability
- เรียนรู้การจัดการ Project Structure
- เข้าใจ Security และ Performance Best Practices

## 📁 Project Structure และ Organization

### 1. **Folder Structure**

```typescript
// ตัวอย่างโครงสร้างโปรเจ็กต์ที่ดี
/*
src/
├── components/           # UI Components
│   ├── common/          # Reusable components
│   ├── forms/           # Form components
│   └── layout/          # Layout components
├── services/            # Business logic services
│   ├── api/            # API clients
│   ├── auth/           # Authentication
│   └── data/           # Data processing
├── types/              # Type definitions
│   ├── api.ts          # API types
│   ├── common.ts       # Common types
│   └── entities.ts     # Domain entities
├── utils/              # Utility functions
│   ├── helpers.ts      # Helper functions
│   ├── constants.ts    # Constants
│   └── validators.ts   # Validation functions
├── hooks/              # Custom hooks (React)
├── stores/             # State management
├── tests/              # Test files
│   ├── unit/           # Unit tests
│   ├── integration/    # Integration tests
│   └── fixtures/       # Test data
└── config/             # Configuration files
    ├── database.ts     # Database config
    ├── environment.ts  # Environment variables
    └── api.ts          # API configuration
*/

// Barrel exports สำหรับ clean imports
// types/index.ts
export * from './api';
export * from './common';
export * from './entities';

// utils/index.ts
export * from './helpers';
export * from './constants';
export * from './validators';

// services/index.ts
export * from './api';
export * from './auth';
export * from './data';

// การใช้งาน
import { ApiResponse, User, Product } from '@/types';
import { formatCurrency, validateEmail } from '@/utils';
import { UserService, ProductService } from '@/services';
```

### 2. **Naming Conventions**

```typescript
// ✅ Good naming conventions
// Files และ Folders: kebab-case
// user-service.ts
// product-details.component.ts
// api-error-handler.ts

// Classes: PascalCase
class UserService {
    private readonly apiClient: ApiClient;
}

class ProductRepository {
    async findById(id: string): Promise<Product | null> {}
}

// Interfaces: PascalCase (อาจมี I prefix หรือไม่ก็ได้)
interface User {
    id: string;
    name: string;
    email: string;
}

interface IUserRepository {
    findById(id: string): Promise<User | null>;
    save(user: User): Promise<void>;
}

// Types: PascalCase
type UserRole = 'admin' | 'user' | 'moderator';
type ApiResponse<T> = {
    data: T;
    status: number;
    message: string;
};

// Functions และ Variables: camelCase
const getUserById = async (id: string): Promise<User | null> => {
    const userRepository = new UserRepository();
    return userRepository.findById(id);
};

// Constants: SCREAMING_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';
const DEFAULT_TIMEOUT = 5000;
const HTTP_STATUS_CODES = {
    OK: 200,
    CREATED: 201,
    BAD_REQUEST: 400,
    UNAUTHORIZED: 401,
    NOT_FOUND: 404
} as const;

// Enums: PascalCase
enum UserStatus {
    Active = 'active',
    Inactive = 'inactive',
    Pending = 'pending',
    Suspended = 'suspended'
}

// Generic type parameters: Single uppercase letters
interface Repository<T, K = string> {
    findById(id: K): Promise<T | null>;
    save(entity: T): Promise<void>;
    delete(id: K): Promise<boolean>;
}

// Boolean variables: start with is, has, can, should
const isLoggedIn = true;
const hasPermission = false;
const canEdit = true;
const shouldUpdate = false;

// Event handlers: handle prefix
const handleUserClick = (user: User) => {
    console.log(`User ${user.name} clicked`);
};

const handleFormSubmit = (data: FormData) => {
    // Handle form submission
};
```

### 3. **Code Organization**

```typescript
// ✅ Separate concerns properly
// models/user.model.ts
export interface User {
    id: string;
    name: string;
    email: string;
    role: UserRole;
    createdAt: Date;
    updatedAt: Date;
}

export interface CreateUserRequest {
    name: string;
    email: string;
    role: UserRole;
}

export interface UpdateUserRequest {
    name?: string;
    email?: string;
    role?: UserRole;
}

// repositories/user.repository.ts
export interface IUserRepository {
    findById(id: string): Promise<User | null>;
    findByEmail(email: string): Promise<User | null>;
    findAll(): Promise<User[]>;
    create(user: CreateUserRequest): Promise<User>;
    update(id: string, user: UpdateUserRequest): Promise<User>;
    delete(id: string): Promise<boolean>;
}

export class UserRepository implements IUserRepository {
    constructor(private readonly db: Database) {}

    async findById(id: string): Promise<User | null> {
        try {
            const result = await this.db.query(
                'SELECT * FROM users WHERE id = $1',
                [id]
            );
            return result.rows[0] || null;
        } catch (error) {
            throw new RepositoryError(`Failed to find user by id: ${id}`, error);
        }
    }

    async findByEmail(email: string): Promise<User | null> {
        try {
            const result = await this.db.query(
                'SELECT * FROM users WHERE email = $1',
                [email]
            );
            return result.rows[0] || null;
        } catch (error) {
            throw new RepositoryError(`Failed to find user by email: ${email}`, error);
        }
    }

    // ... other methods
}

// services/user.service.ts
export class UserService {
    constructor(
        private readonly userRepository: IUserRepository,
        private readonly emailService: IEmailService,
        private readonly logger: ILogger
    ) {}

    async createUser(userData: CreateUserRequest): Promise<User> {
        this.logger.info('Creating new user', { email: userData.email });

        // Validate input
        await this.validateUserData(userData);

        // Check if user already exists
        const existingUser = await this.userRepository.findByEmail(userData.email);
        if (existingUser) {
            throw new ConflictError('User with this email already exists');
        }

        try {
            // Create user
            const user = await this.userRepository.create(userData);

            // Send welcome email
            await this.emailService.sendWelcomeEmail(user.email, user.name);

            this.logger.info('User created successfully', { userId: user.id });
            return user;
        } catch (error) {
            this.logger.error('Failed to create user', { error, userData });
            throw new ServiceError('Failed to create user', error);
        }
    }

    private async validateUserData(userData: CreateUserRequest): Promise<void> {
        if (!userData.name || userData.name.trim().length < 2) {
            throw new ValidationError('Name must be at least 2 characters long');
        }

        if (!this.isValidEmail(userData.email)) {
            throw new ValidationError('Invalid email format');
        }

        if (!Object.values(UserRole).includes(userData.role)) {
            throw new ValidationError('Invalid user role');
        }
    }

    private isValidEmail(email: string): boolean {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }
}

// controllers/user.controller.ts
export class UserController {
    constructor(
        private readonly userService: UserService,
        private readonly logger: ILogger
    ) {}

    async createUser(req: Request, res: Response): Promise<void> {
        try {
            const userData = req.body as CreateUserRequest;
            const user = await this.userService.createUser(userData);
            
            res.status(201).json({
                success: true,
                data: user,
                message: 'User created successfully'
            });
        } catch (error) {
            this.handleError(error, res);
        }
    }

    private handleError(error: Error, res: Response): void {
        if (error instanceof ValidationError) {
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
        } else {
            this.logger.error('Unexpected error in UserController', { error });
            res.status(500).json({
                success: false,
                error: 'Internal Server Error',
                message: 'An unexpected error occurred'
            });
        }
    }
}
```

## 🛡️ Type Safety Best Practices

### 1. **Strict Type Checking**

```typescript
// tsconfig.json configuration
{
  "compilerOptions": {
    "strict": true,                    // Enable all strict type checking
    "noImplicitAny": true,            // Error on expressions with implied 'any'
    "noImplicitReturns": true,        // Error when not all code paths return a value
    "noImplicitThis": true,           // Error on 'this' expressions with implied 'any'
    "noUnusedLocals": true,           // Error on unused local variables
    "noUnusedParameters": true,       // Error on unused parameters
    "exactOptionalPropertyTypes": true, // Exact optional property types
    "noUncheckedIndexedAccess": true   // Add undefined to index signature results
  }
}

// ✅ Use proper type annotations
function processUser(user: User): ProcessedUser {
    return {
        id: user.id,
        displayName: user.name.toUpperCase(),
        isActive: user.status === UserStatus.Active
    };
}

// ❌ Avoid any type
function processData(data: any): any {
    return data.someProperty;
}

// ✅ Use unknown for uncertain types
function processData(data: unknown): string {
    if (typeof data === 'object' && data !== null && 'name' in data) {
        return (data as { name: string }).name;
    }
    throw new Error('Invalid data format');
}

// ✅ Use type guards
function isUser(obj: unknown): obj is User {
    return (
        typeof obj === 'object' &&
        obj !== null &&
        'id' in obj &&
        'name' in obj &&
        'email' in obj &&
        typeof (obj as any).id === 'string' &&
        typeof (obj as any).name === 'string' &&
        typeof (obj as any).email === 'string'
    );
}

// ✅ Use assertion functions
function assertIsUser(obj: unknown): asserts obj is User {
    if (!isUser(obj)) {
        throw new Error('Object is not a valid User');
    }
}

// การใช้งาน
function handleUserData(data: unknown): void {
    assertIsUser(data);
    // TypeScript จะรู้ว่า data เป็น User หลังจากนี้
    console.log(data.name);
}
```

### 2. **Advanced Type Patterns**

```typescript
// ✅ Use branded types for different string types
type UserId = string & { readonly brand: unique symbol };
type Email = string & { readonly brand: unique symbol };
type ProductId = string & { readonly brand: unique symbol };

function createUserId(id: string): UserId {
    if (!id || id.length < 3) {
        throw new Error('Invalid user ID');
    }
    return id as UserId;
}

function createEmail(email: string): Email {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
        throw new Error('Invalid email format');
    }
    return email as Email;
}

// ✅ Use template literal types
type LogLevel = 'debug' | 'info' | 'warn' | 'error';
type LogMessage<T extends LogLevel> = `[${Uppercase<T>}] ${string}`;

function log<T extends LogLevel>(level: T, message: LogMessage<T>): void {
    console.log(message);
}

// การใช้งาน
log('info', '[INFO] User logged in'); // ✅
log('error', '[ERROR] Database connection failed'); // ✅
// log('info', '[ERROR] Wrong prefix'); // ❌ TypeScript error

// ✅ Use conditional types for complex logic
type ApiResponse<T> = T extends string
    ? { message: T }
    : T extends number
    ? { count: T }
    : { data: T };

type StringResponse = ApiResponse<string>;  // { message: string }
type NumberResponse = ApiResponse<number>;  // { count: number }
type UserResponse = ApiResponse<User>;      // { data: User }

// ✅ Use mapped types for transformations
type Optional<T> = {
    [K in keyof T]?: T[K];
};

type Required<T> = {
    [K in keyof T]-?: T[K];
};

type Readonly<T> = {
    readonly [K in keyof T]: T[K];
};

// ✅ Use utility types effectively
interface BaseEntity {
    id: string;
    createdAt: Date;
    updatedAt: Date;
}

interface User extends BaseEntity {
    name: string;
    email: string;
    role: UserRole;
}

type CreateUserInput = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateUserInput = Partial<Pick<User, 'name' | 'email' | 'role'>>;
type UserSummary = Pick<User, 'id' | 'name' | 'email'>;

// ✅ Use const assertions for immutable data
const HTTP_METHODS = ['GET', 'POST', 'PUT', 'DELETE'] as const;
type HttpMethod = typeof HTTP_METHODS[number]; // 'GET' | 'POST' | 'PUT' | 'DELETE'

const API_ENDPOINTS = {
    users: '/api/users',
    products: '/api/products',
    orders: '/api/orders'
} as const;

type ApiEndpoint = typeof API_ENDPOINTS[keyof typeof API_ENDPOINTS];
```

### 3. **Error Handling Best Practices**

```typescript
// ✅ Create custom error classes
abstract class AppError extends Error {
    abstract readonly statusCode: number;
    abstract readonly isOperational: boolean;

    constructor(message: string, public readonly context?: Record<string, any>) {
        super(message);
        this.name = this.constructor.name;
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    readonly statusCode = 400;
    readonly isOperational = true;

    constructor(message: string, public readonly field?: string) {
        super(message);
    }
}

class NotFoundError extends AppError {
    readonly statusCode = 404;
    readonly isOperational = true;

    constructor(resource: string, id: string) {
        super(`${resource} with id ${id} not found`);
    }
}

class DatabaseError extends AppError {
    readonly statusCode = 500;
    readonly isOperational = false;

    constructor(message: string, public readonly originalError: Error) {
        super(message);
    }
}

// ✅ Use Result pattern for error handling
type Result<T, E = Error> = 
    | { success: true; data: T }
    | { success: false; error: E };

async function safeUserFetch(id: string): Promise<Result<User, NotFoundError | DatabaseError>> {
    try {
        const user = await userRepository.findById(id);
        if (!user) {
            return { success: false, error: new NotFoundError('User', id) };
        }
        return { success: true, data: user };
    } catch (error) {
        return { 
            success: false, 
            error: new DatabaseError('Failed to fetch user', error as Error) 
        };
    }
}

// การใช้งาน Result pattern
async function handleUserRequest(id: string): Promise<void> {
    const result = await safeUserFetch(id);
    
    if (result.success) {
        console.log('User found:', result.data.name);
    } else {
        if (result.error instanceof NotFoundError) {
            console.log('User not found');
        } else {
            console.error('Database error:', result.error.message);
        }
    }
}

// ✅ Use Either pattern for functional error handling
type Either<L, R> = Left<L> | Right<R>;

interface Left<L> {
    readonly _tag: 'Left';
    readonly left: L;
}

interface Right<R> {
    readonly _tag: 'Right';
    readonly right: R;
}

const left = <L>(value: L): Either<L, never> => ({ _tag: 'Left', left: value });
const right = <R>(value: R): Either<never, R> => ({ _tag: 'Right', right: value });

function isLeft<L, R>(either: Either<L, R>): either is Left<L> {
    return either._tag === 'Left';
}

function isRight<L, R>(either: Either<L, R>): either is Right<R> {
    return either._tag === 'Right';
}

// Chain operations with Either
function map<L, R, R2>(
    either: Either<L, R>,
    fn: (value: R) => R2
): Either<L, R2> {
    return isLeft(either) ? either : right(fn(either.right));
}

function flatMap<L, R, R2>(
    either: Either<L, R>,
    fn: (value: R) => Either<L, R2>
): Either<L, R2> {
    return isLeft(either) ? either : fn(either.right);
}

// การใช้งาน Either
async function processUserEither(id: string): Promise<Either<AppError, ProcessedUser>> {
    try {
        const user = await userRepository.findById(id);
        if (!user) {
            return left(new NotFoundError('User', id));
        }

        const processedUser = {
            id: user.id,
            displayName: user.name.toUpperCase(),
            isActive: user.status === UserStatus.Active
        };

        return right(processedUser);
    } catch (error) {
        return left(new DatabaseError('Failed to process user', error as Error));
    }
}
```

## 🚀 Performance Best Practices

### 1. **Efficient Code Patterns**

```typescript
// ✅ Use object freezing for immutable data
const CONFIG = Object.freeze({
    API_URL: 'https://api.example.com',
    TIMEOUT: 5000,
    MAX_RETRIES: 3
} as const);

// ✅ Use Map for frequent lookups
class UserCache {
    private readonly cache = new Map<string, User>();
    private readonly maxSize = 1000;

    set(id: string, user: User): void {
        if (this.cache.size >= this.maxSize) {
            // Remove oldest entry
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        this.cache.set(id, user);
    }

    get(id: string): User | undefined {
        return this.cache.get(id);
    }

    has(id: string): boolean {
        return this.cache.has(id);
    }

    clear(): void {
        this.cache.clear();
    }
}

// ✅ Use WeakMap for object associations
class ComponentMetadata {
    private readonly metadata = new WeakMap<object, Record<string, any>>();

    setMetadata(component: object, key: string, value: any): void {
        let meta = this.metadata.get(component);
        if (!meta) {
            meta = {};
            this.metadata.set(component, meta);
        }
        meta[key] = value;
    }

    getMetadata(component: object, key: string): any {
        const meta = this.metadata.get(component);
        return meta?.[key];
    }
}

// ✅ Use efficient array operations
function processLargeDataset(data: readonly User[]): ProcessedUser[] {
    // Use for...of for better performance than forEach
    const results: ProcessedUser[] = [];
    
    for (const user of data) {
        if (user.status === UserStatus.Active) {
            results.push({
                id: user.id,
                displayName: user.name.toUpperCase(),
                isActive: true
            });
        }
    }
    
    return results;
}

// ✅ Use Set for unique collections
function removeDuplicateUsers(users: User[]): User[] {
    const seen = new Set<string>();
    return users.filter(user => {
        if (seen.has(user.id)) {
            return false;
        }
        seen.add(user.id);
        return true;
    });
}

// ✅ Use lazy loading and caching
class LazyService<T> {
    private instance?: T;
    
    constructor(private readonly factory: () => T) {}
    
    get(): T {
        if (this.instance === undefined) {
            this.instance = this.factory();
        }
        return this.instance;
    }
    
    reset(): void {
        this.instance = undefined;
    }
}

// การใช้งาน
const userService = new LazyService(() => new UserService(
    new UserRepository(database),
    new EmailService(),
    logger
));

// ✅ Use debouncing for expensive operations
function debounce<T extends (...args: any[]) => any>(
    func: T,
    delay: number
): (...args: Parameters<T>) => void {
    let timeoutId: NodeJS.Timeout;
    
    return (...args: Parameters<T>): void => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func(...args), delay);
    };
}

// การใช้งาน
const debouncedSearch = debounce(async (query: string) => {
    const results = await searchService.search(query);
    updateSearchResults(results);
}, 300);
```

### 2. **Memory Management**

```typescript
// ✅ Proper cleanup and resource management
class ResourceManager {
    private readonly resources = new Set<Disposable>();

    addResource(resource: Disposable): void {
        this.resources.add(resource);
    }

    dispose(): void {
        for (const resource of this.resources) {
            try {
                resource.dispose();
            } catch (error) {
                console.error('Error disposing resource:', error);
            }
        }
        this.resources.clear();
    }
}

interface Disposable {
    dispose(): void;
}

class DatabaseConnection implements Disposable {
    private isConnected = false;

    async connect(): Promise<void> {
        this.isConnected = true;
        console.log('Database connected');
    }

    dispose(): void {
        if (this.isConnected) {
            this.isConnected = false;
            console.log('Database connection closed');
        }
    }
}

// ✅ Use AbortController for cancellable operations
class ApiService {
    private readonly abortController = new AbortController();

    async fetchData(url: string): Promise<any> {
        const response = await fetch(url, {
            signal: this.abortController.signal
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return response.json();
    }

    cancel(): void {
        this.abortController.abort();
    }
}

// ✅ Use proper event listener cleanup
class EventManager {
    private readonly listeners = new Map<string, Set<EventListener>>();

    addEventListener(element: Element, event: string, listener: EventListener): () => void {
        element.addEventListener(event, listener);
        
        if (!this.listeners.has(event)) {
            this.listeners.set(event, new Set());
        }
        this.listeners.get(event)!.add(listener);

        // Return cleanup function
        return () => {
            element.removeEventListener(event, listener);
            this.listeners.get(event)?.delete(listener);
        };
    }

    removeAllListeners(): void {
        for (const [event, listeners] of this.listeners) {
            for (const listener of listeners) {
                // Remove from actual elements would require reference
                // This is simplified example
                console.log(`Removing listener for ${event}`);
            }
        }
        this.listeners.clear();
    }
}
```

## 🔒 Security Best Practices

### 1. **Input Validation and Sanitization**

```typescript
// ✅ Strong input validation
import validator from 'validator';
import DOMPurify from 'dompurify';

class InputValidator {
    static validateEmail(email: string): boolean {
        return validator.isEmail(email) && email.length <= 254;
    }

    static validatePassword(password: string): { valid: boolean; errors: string[] } {
        const errors: string[] = [];
        
        if (password.length < 8) {
            errors.push('Password must be at least 8 characters long');
        }
        
        if (!/[A-Z]/.test(password)) {
            errors.push('Password must contain at least one uppercase letter');
        }
        
        if (!/[a-z]/.test(password)) {
            errors.push('Password must contain at least one lowercase letter');
        }
        
        if (!/\d/.test(password)) {
            errors.push('Password must contain at least one number');
        }
        
        if (!/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
            errors.push('Password must contain at least one special character');
        }

        return { valid: errors.length === 0, errors };
    }

    static sanitizeHtml(html: string): string {
        return DOMPurify.sanitize(html);
    }

    static validateAndSanitizeInput(input: string, maxLength: number = 1000): string {
        if (typeof input !== 'string') {
            throw new ValidationError('Input must be a string');
        }

        if (input.length > maxLength) {
            throw new ValidationError(`Input exceeds maximum length of ${maxLength}`);
        }

        // Remove potentially dangerous characters
        return input.replace(/[<>\"'&]/g, '');
    }
}

// ✅ SQL injection prevention
class SafeQueryBuilder {
    private query = '';
    private parameters: any[] = [];

    select(columns: string[]): this {
        // Validate column names to prevent injection
        const validColumns = columns.filter(col => /^[a-zA-Z_][a-zA-Z0-9_]*$/.test(col));
        this.query += `SELECT ${validColumns.join(', ')} `;
        return this;
    }

    from(table: string): this {
        // Validate table name
        if (!/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(table)) {
            throw new Error('Invalid table name');
        }
        this.query += `FROM ${table} `;
        return this;
    }

    where(condition: string, value: any): this {
        this.query += `WHERE ${condition} `;
        this.parameters.push(value);
        return this;
    }

    build(): { query: string; parameters: any[] } {
        return { query: this.query.trim(), parameters: this.parameters };
    }
}

// การใช้งาน
const queryBuilder = new SafeQueryBuilder()
    .select(['id', 'name', 'email'])
    .from('users')
    .where('id = $1', userId);

const { query, parameters } = queryBuilder.build();
// query: "SELECT id, name, email FROM users WHERE id = $1"
// parameters: [userId]
```

### 2. **Authentication and Authorization**

```typescript
// ✅ Secure authentication patterns
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

interface JwtPayload {
    userId: string;
    email: string;
    role: UserRole;
    iat: number;
    exp: number;
}

class AuthService {
    private readonly jwtSecret: string;
    private readonly saltRounds = 12;

    constructor(jwtSecret: string) {
        this.jwtSecret = jwtSecret;
    }

    async hashPassword(password: string): Promise<string> {
        return bcrypt.hash(password, this.saltRounds);
    }

    async verifyPassword(password: string, hash: string): Promise<boolean> {
        return bcrypt.compare(password, hash);
    }

    generateToken(user: User): string {
        const payload: Omit<JwtPayload, 'iat' | 'exp'> = {
            userId: user.id,
            email: user.email,
            role: user.role
        };

        return jwt.sign(payload, this.jwtSecret, {
            expiresIn: '24h',
            issuer: 'your-app-name',
            audience: 'your-app-users'
        });
    }

    verifyToken(token: string): JwtPayload {
        try {
            return jwt.verify(token, this.jwtSecret, {
                issuer: 'your-app-name',
                audience: 'your-app-users'
            }) as JwtPayload;
        } catch (error) {
            throw new AuthenticationError('Invalid or expired token');
        }
    }

    async authenticateUser(email: string, password: string): Promise<{ user: User; token: string }> {
        const user = await userRepository.findByEmail(email);
        if (!user) {
            throw new AuthenticationError('Invalid credentials');
        }

        const isValid = await this.verifyPassword(password, user.passwordHash);
        if (!isValid) {
            throw new AuthenticationError('Invalid credentials');
        }

        const token = this.generateToken(user);
        return { user, token };
    }
}

// ✅ Authorization middleware
class AuthorizationService {
    static hasPermission(userRole: UserRole, requiredPermissions: Permission[]): boolean {
        const rolePermissions = this.getRolePermissions(userRole);
        return requiredPermissions.every(permission => 
            rolePermissions.includes(permission)
        );
    }

    static requirePermissions(permissions: Permission[]) {
        return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {
            const originalMethod = descriptor.value;

            descriptor.value = function(this: any, ...args: any[]) {
                const user = this.getCurrentUser(); // Assume this method exists
                
                if (!user) {
                    throw new AuthenticationError('Authentication required');
                }

                if (!AuthorizationService.hasPermission(user.role, permissions)) {
                    throw new AuthorizationError('Insufficient permissions');
                }

                return originalMethod.apply(this, args);
            };

            return descriptor;
        };
    }

    private static getRolePermissions(role: UserRole): Permission[] {
        const permissions: Record<UserRole, Permission[]> = {
            admin: ['read', 'write', 'delete', 'manage_users'],
            moderator: ['read', 'write', 'delete'],
            user: ['read', 'write_own']
        };

        return permissions[role] || [];
    }
}

// การใช้งาน authorization decorator
class UserController {
    @AuthorizationService.requirePermissions(['manage_users'])
    async deleteUser(id: string): Promise<void> {
        await userService.deleteUser(id);
    }

    @AuthorizationService.requirePermissions(['read'])
    async getUser(id: string): Promise<User> {
        return userService.getUser(id);
    }
}
```

## 📊 Testing Best Practices

### 1. **Unit Testing Patterns**

```typescript
// ✅ Comprehensive unit tests
import { describe, it, expect, beforeEach, afterEach, jest } from '@jest/globals';

// Mock interfaces for testing
interface MockUserRepository {
    findById: jest.MockedFunction<(id: string) => Promise<User | null>>;
    findByEmail: jest.MockedFunction<(email: string) => Promise<User | null>>;
    create: jest.MockedFunction<(user: CreateUserRequest) => Promise<User>>;
}

describe('UserService', () => {
    let userService: UserService;
    let mockUserRepository: MockUserRepository;
    let mockEmailService: jest.Mocked<IEmailService>;
    let mockLogger: jest.Mocked<ILogger>;

    beforeEach(() => {
        mockUserRepository = {
            findById: jest.fn(),
            findByEmail: jest.fn(),
            create: jest.fn()
        };

        mockEmailService = {
            sendWelcomeEmail: jest.fn()
        };

        mockLogger = {
            info: jest.fn(),
            error: jest.fn(),
            warn: jest.fn(),
            debug: jest.fn()
        };

        userService = new UserService(
            mockUserRepository as any,
            mockEmailService,
            mockLogger
        );
    });

    afterEach(() => {
        jest.clearAllMocks();
    });

    describe('createUser', () => {
        const validUserData: CreateUserRequest = {
            name: 'John Doe',
            email: 'john@example.com',
            role: UserRole.User
        };

        it('should create a user successfully', async () => {
            // Arrange
            const expectedUser: User = {
                id: 'user123',
                ...validUserData,
                createdAt: new Date(),
                updatedAt: new Date()
            };

            mockUserRepository.findByEmail.mockResolvedValue(null);
            mockUserRepository.create.mockResolvedValue(expectedUser);
            mockEmailService.sendWelcomeEmail.mockResolvedValue();

            // Act
            const result = await userService.createUser(validUserData);

            // Assert
            expect(result).toEqual(expectedUser);
            expect(mockUserRepository.findByEmail).toHaveBeenCalledWith(validUserData.email);
            expect(mockUserRepository.create).toHaveBeenCalledWith(validUserData);
            expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(
                expectedUser.email,
                expectedUser.name
            );
            expect(mockLogger.info).toHaveBeenCalledWith(
                'Creating new user',
                { email: validUserData.email }
            );
            expect(mockLogger.info).toHaveBeenCalledWith(
                'User created successfully',
                { userId: expectedUser.id }
            );
        });

        it('should throw ConflictError when user already exists', async () => {
            // Arrange
            const existingUser: User = {
                id: 'existing123',
                ...validUserData,
                createdAt: new Date(),
                updatedAt: new Date()
            };

            mockUserRepository.findByEmail.mockResolvedValue(existingUser);

            // Act & Assert
            await expect(userService.createUser(validUserData)).rejects.toThrow(
                ConflictError
            );
            expect(mockUserRepository.create).not.toHaveBeenCalled();
            expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
        });

        it('should throw ValidationError for invalid email', async () => {
            // Arrange
            const invalidUserData = {
                ...validUserData,
                email: 'invalid-email'
            };

            // Act & Assert
            await expect(userService.createUser(invalidUserData)).rejects.toThrow(
                ValidationError
            );
            expect(mockUserRepository.findByEmail).not.toHaveBeenCalled();
        });

        it('should handle repository errors gracefully', async () => {
            // Arrange
            const repositoryError = new Error('Database connection failed');
            mockUserRepository.findByEmail.mockResolvedValue(null);
            mockUserRepository.create.mockRejectedValue(repositoryError);

            // Act & Assert
            await expect(userService.createUser(validUserData)).rejects.toThrow(
                ServiceError
            );
            expect(mockLogger.error).toHaveBeenCalledWith(
                'Failed to create user',
                { error: repositoryError, userData: validUserData }
            );
        });
    });
});

// ✅ Integration tests
describe('UserController Integration', () => {
    let app: Application;
    let server: Server;
    let testDb: TestDatabase;

    beforeAll(async () => {
        testDb = await TestDatabase.create();
        app = createApp(testDb.getConnection());
        server = app.listen(0);
    });

    afterAll(async () => {
        await server.close();
        await testDb.cleanup();
    });

    beforeEach(async () => {
        await testDb.clear();
    });

    it('should create user via API', async () => {
        // Arrange
        const userData = {
            name: 'John Doe',
            email: 'john@example.com',
            role: 'user'
        };

        // Act
        const response = await request(app)
            .post('/api/users')
            .send(userData)
            .expect(201);

        // Assert
        expect(response.body).toMatchObject({
            success: true,
            data: expect.objectContaining({
                id: expect.any(String),
                name: userData.name,
                email: userData.email,
                role: userData.role
            }),
            message: 'User created successfully'
        });

        // Verify in database
        const user = await testDb.findUserByEmail(userData.email);
        expect(user).toBeTruthy();
        expect(user!.name).toBe(userData.name);
    });
});
```

## 📝 แบบฝึกหัด

### 1. **Code Review Checklist**
สร้าง checklist สำหรับ code review:
- Type safety และ error handling
- Performance และ memory management
- Security considerations
- Testing coverage
- Code organization และ naming

### 2. **Refactoring Exercise**
ปรับปรุงโค้ดต่อไปนี้ให้ตาม best practices:
- แยก concerns
- เพิ่ม type safety
- ปรับปรุง error handling
- เพิ่ม tests

### 3. **Performance Optimization**
วิเคราะห์และปรับปรุง performance:
- Memory usage
- Algorithm complexity
- Caching strategies
- Bundle size optimization

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 17: Design Patterns](./17-design-patterns.md)

**บทต่อไป**: [บทที่ 19: Real-world Projects](./19-real-world-projects.md)

**กลับไปหน้าหลัก**: [README](./README.md)
