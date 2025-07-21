# บทที่ 14: Error Handling

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจการจัดการ Error ใน TypeScript อย่างมีประสิทธิภาพ
- เรียนรู้ Result Pattern และ Either Pattern
- สร้างระบบ Error Handling ที่ Type-safe
- เรียนรู้การใช้งาน Custom Error Classes

## ⚠️ พื้นฐานการจัดการ Error

### 1. **Traditional Error Handling**

```typescript
// Basic try-catch
function parseNumber(input: string): number {
    try {
        const result = parseInt(input, 10);
        if (isNaN(result)) {
            throw new Error(`Invalid number: ${input}`);
        }
        return result;
    } catch (error) {
        console.error("Error parsing number:", error);
        throw error; // Re-throw หรือ handle ตามต้องการ
    }
}

// Function ที่อาจจะ throw error
function divideNumbers(a: number, b: number): number {
    if (b === 0) {
        throw new Error("Division by zero is not allowed");
    }
    return a / b;
}

// การใช้งาน
try {
    const num1 = parseNumber("10");
    const num2 = parseNumber("0");
    const result = divideNumbers(num1, num2);
    console.log("Result:", result);
} catch (error) {
    if (error instanceof Error) {
        console.error("Operation failed:", error.message);
    }
}

// Async error handling
async function fetchUserData(userId: string): Promise<User> {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const userData = await response.json();
        return userData;
    } catch (error) {
        console.error("Failed to fetch user data:", error);
        throw error;
    }
}

// การใช้งาน async function
async function displayUser(userId: string): Promise<void> {
    try {
        const user = await fetchUserData(userId);
        console.log("User:", user);
    } catch (error) {
        console.error("Could not display user:", error);
    }
}
```

### 2. **Custom Error Classes**

```typescript
// Base Error class
abstract class AppError extends Error {
    abstract readonly statusCode: number;
    abstract readonly isOperational: boolean;
    
    constructor(message: string, public readonly context?: Record<string, any>) {
        super(message);
        this.name = this.constructor.name;
        
        // Maintain proper stack trace
        Error.captureStackTrace(this, this.constructor);
    }
}

// Specific error types
class ValidationError extends AppError {
    readonly statusCode = 400;
    readonly isOperational = true;
    
    constructor(
        message: string,
        public readonly field: string,
        context?: Record<string, any>
    ) {
        super(message, context);
    }
}

class NotFoundError extends AppError {
    readonly statusCode = 404;
    readonly isOperational = true;
    
    constructor(resource: string, id?: string) {
        const message = id 
            ? `${resource} with id '${id}' not found`
            : `${resource} not found`;
        super(message, { resource, id });
    }
}

class DatabaseError extends AppError {
    readonly statusCode = 500;
    readonly isOperational = false;
    
    constructor(message: string, public readonly query?: string) {
        super(message, { query });
    }
}

class AuthenticationError extends AppError {
    readonly statusCode = 401;
    readonly isOperational = true;
    
    constructor(message: string = "Authentication required") {
        super(message);
    }
}

class AuthorizationError extends AppError {
    readonly statusCode = 403;
    readonly isOperational = true;
    
    constructor(
        message: string = "Insufficient permissions",
        public readonly requiredRole?: string
    ) {
        super(message, { requiredRole });
    }
}

// Error factory
class ErrorFactory {
    static validation(field: string, message: string): ValidationError {
        return new ValidationError(message, field);
    }
    
    static notFound(resource: string, id?: string): NotFoundError {
        return new NotFoundError(resource, id);
    }
    
    static database(message: string, query?: string): DatabaseError {
        return new DatabaseError(message, query);
    }
    
    static authentication(message?: string): AuthenticationError {
        return new AuthenticationError(message);
    }
    
    static authorization(message?: string, role?: string): AuthorizationError {
        return new AuthorizationError(message, role);
    }
}

// การใช้งาน
function validateUser(userData: any): User {
    if (!userData.email) {
        throw ErrorFactory.validation("email", "Email is required");
    }
    
    if (!userData.email.includes("@")) {
        throw ErrorFactory.validation("email", "Invalid email format");
    }
    
    if (!userData.name || userData.name.length < 2) {
        throw ErrorFactory.validation("name", "Name must be at least 2 characters");
    }
    
    return userData as User;
}

async function getUserById(id: string): Promise<User> {
    try {
        // Simulate database query
        const query = `SELECT * FROM users WHERE id = '${id}'`;
        const result = await executeQuery(query);
        
        if (!result) {
            throw ErrorFactory.notFound("User", id);
        }
        
        return result;
    } catch (error) {
        if (error instanceof AppError) {
            throw error;
        }
        throw ErrorFactory.database("Failed to fetch user", `getUserById(${id})`);
    }
}
```

## 🔄 Result Pattern

### 1. **Basic Result Type**

```typescript
// Result type definition
type Result<T, E = Error> = 
    | { success: true; data: T }
    | { success: false; error: E };

// Helper functions
class ResultHelpers {
    static ok<T>(data: T): Result<T, never> {
        return { success: true, data };
    }
    
    static err<E>(error: E): Result<never, E> {
        return { success: false, error };
    }
    
    static isOk<T, E>(result: Result<T, E>): result is { success: true; data: T } {
        return result.success;
    }
    
    static isErr<T, E>(result: Result<T, E>): result is { success: false; error: E } {
        return !result.success;
    }
}

// Type alias for convenience
type Ok<T> = { success: true; data: T };
type Err<E> = { success: false; error: E };

// การใช้งาน
function parseInteger(input: string): Result<number, string> {
    const num = parseInt(input, 10);
    
    if (isNaN(num)) {
        return ResultHelpers.err(`"${input}" is not a valid integer`);
    }
    
    return ResultHelpers.ok(num);
}

function divide(a: number, b: number): Result<number, string> {
    if (b === 0) {
        return ResultHelpers.err("Division by zero");
    }
    
    return ResultHelpers.ok(a / b);
}

// Chain operations
function calculateExpression(aStr: string, bStr: string): Result<number, string> {
    const aResult = parseInteger(aStr);
    if (ResultHelpers.isErr(aResult)) {
        return aResult;
    }
    
    const bResult = parseInteger(bStr);
    if (ResultHelpers.isErr(bResult)) {
        return bResult;
    }
    
    return divide(aResult.data, bResult.data);
}

// การใช้งาน
const result = calculateExpression("10", "2");

if (ResultHelpers.isOk(result)) {
    console.log("Calculation result:", result.data); // 5
} else {
    console.error("Calculation failed:", result.error);
}
```

### 2. **Advanced Result Pattern**

```typescript
// Result class with chainable methods
class Result<T, E = Error> {
    constructor(
        private readonly _success: boolean,
        private readonly _data?: T,
        private readonly _error?: E
    ) {}
    
    static ok<T>(data: T): Result<T, never> {
        return new Result(true, data);
    }
    
    static err<E>(error: E): Result<never, E> {
        return new Result(false, undefined, error);
    }
    
    isOk(): this is Result<T, never> {
        return this._success;
    }
    
    isErr(): this is Result<never, E> {
        return !this._success;
    }
    
    get data(): T {
        if (!this._success) {
            throw new Error("Cannot access data on error result");
        }
        return this._data!;
    }
    
    get error(): E {
        if (this._success) {
            throw new Error("Cannot access error on success result");
        }
        return this._error!;
    }
    
    // Map operation (transform success value)
    map<U>(fn: (data: T) => U): Result<U, E> {
        if (this.isErr()) {
            return Result.err(this.error);
        }
        
        try {
            return Result.ok(fn(this.data));
        } catch (error) {
            return Result.err(error as E);
        }
    }
    
    // MapError operation (transform error value)
    mapError<F>(fn: (error: E) => F): Result<T, F> {
        if (this.isOk()) {
            return Result.ok(this.data);
        }
        
        return Result.err(fn(this.error));
    }
    
    // FlatMap operation (chain results)
    flatMap<U, F>(fn: (data: T) => Result<U, F>): Result<U, E | F> {
        if (this.isErr()) {
            return Result.err(this.error);
        }
        
        try {
            return fn(this.data);
        } catch (error) {
            return Result.err(error as E);
        }
    }
    
    // Unwrap with default value
    unwrapOr(defaultValue: T): T {
        return this.isOk() ? this.data : defaultValue;
    }
    
    // Unwrap or throw error
    unwrap(): T {
        if (this.isErr()) {
            throw this.error;
        }
        return this.data;
    }
    
    // Match pattern (similar to Rust)
    match<U>(onOk: (data: T) => U, onErr: (error: E) => U): U {
        return this.isOk() ? onOk(this.data) : onErr(this.error);
    }
}

// การใช้งาน advanced Result
function validateEmail(email: string): Result<string, string> {
    if (!email) {
        return Result.err("Email is required");
    }
    
    if (!email.includes("@")) {
        return Result.err("Invalid email format");
    }
    
    return Result.ok(email.toLowerCase());
}

function validateAge(age: string): Result<number, string> {
    const num = parseInt(age, 10);
    
    if (isNaN(num)) {
        return Result.err("Age must be a number");
    }
    
    if (num < 0 || num > 150) {
        return Result.err("Age must be between 0 and 150");
    }
    
    return Result.ok(num);
}

interface ValidatedUser {
    email: string;
    age: number;
}

function validateUser(email: string, age: string): Result<ValidatedUser, string> {
    return validateEmail(email)
        .flatMap(validEmail => 
            validateAge(age)
                .map(validAge => ({ email: validEmail, age: validAge }))
        );
}

// การใช้งาน
const userResult = validateUser("john@example.com", "25");

const message = userResult.match(
    user => `Valid user: ${user.email}, age ${user.age}`,
    error => `Validation failed: ${error}`
);

console.log(message); // "Valid user: john@example.com, age 25"

// Chain operations
const processResult = validateUser("invalid-email", "25")
    .map(user => ({ ...user, id: Date.now() }))
    .map(user => `User ${user.id}: ${user.email}`)
    .unwrapOr("No user to process");

console.log(processResult); // "No user to process"
```

## 🔀 Either Pattern

### 1. **Either Type Implementation**

```typescript
// Either type - similar to Result but more generic
abstract class Either<L, R> {
    abstract isLeft(): this is Left<L, R>;
    abstract isRight(): this is Right<L, R>;
    
    abstract map<U>(fn: (right: R) => U): Either<L, U>;
    abstract mapLeft<U>(fn: (left: L) => U): Either<U, R>;
    abstract flatMap<U>(fn: (right: R) => Either<L, U>): Either<L, U>;
    abstract fold<U>(onLeft: (left: L) => U, onRight: (right: R) => U): U;
    
    static left<L, R>(value: L): Either<L, R> {
        return new Left(value);
    }
    
    static right<L, R>(value: R): Either<L, R> {
        return new Right(value);
    }
}

class Left<L, R> extends Either<L, R> {
    constructor(private readonly value: L) {
        super();
    }
    
    isLeft(): this is Left<L, R> {
        return true;
    }
    
    isRight(): this is Right<L, R> {
        return false;
    }
    
    map<U>(_fn: (right: R) => U): Either<L, U> {
        return Either.left(this.value);
    }
    
    mapLeft<U>(fn: (left: L) => U): Either<U, R> {
        return Either.left(fn(this.value));
    }
    
    flatMap<U>(_fn: (right: R) => Either<L, U>): Either<L, U> {
        return Either.left(this.value);
    }
    
    fold<U>(onLeft: (left: L) => U, _onRight: (right: R) => U): U {
        return onLeft(this.value);
    }
    
    get leftValue(): L {
        return this.value;
    }
}

class Right<L, R> extends Either<L, R> {
    constructor(private readonly value: R) {
        super();
    }
    
    isLeft(): this is Left<L, R> {
        return false;
    }
    
    isRight(): this is Right<L, R> {
        return true;
    }
    
    map<U>(fn: (right: R) => U): Either<L, U> {
        try {
            return Either.right(fn(this.value));
        } catch (error) {
            return Either.left(error as L);
        }
    }
    
    mapLeft<U>(_fn: (left: L) => U): Either<U, R> {
        return Either.right(this.value);
    }
    
    flatMap<U>(fn: (right: R) => Either<L, U>): Either<L, U> {
        try {
            return fn(this.value);
        } catch (error) {
            return Either.left(error as L);
        }
    }
    
    fold<U>(_onLeft: (left: L) => U, onRight: (right: R) => U): U {
        return onRight(this.value);
    }
    
    get rightValue(): R {
        return this.value;
    }
}

// การใช้งาน Either
function safeDivide(a: number, b: number): Either<string, number> {
    if (b === 0) {
        return Either.left("Division by zero");
    }
    return Either.right(a / b);
}

function parseNumber(str: string): Either<string, number> {
    const num = parseFloat(str);
    if (isNaN(num)) {
        return Either.left(`"${str}" is not a valid number`);
    }
    return Either.right(num);
}

// Chain operations
function calculate(aStr: string, bStr: string): Either<string, number> {
    return parseNumber(aStr)
        .flatMap(a => 
            parseNumber(bStr)
                .flatMap(b => safeDivide(a, b))
        );
}

// การใช้งาน
const result = calculate("10", "2");

const message = result.fold(
    error => `Error: ${error}`,
    value => `Result: ${value}`
);

console.log(message); // "Result: 5"

// Multiple operations
const complexCalculation = calculate("20", "4")
    .map(x => x * 2)
    .map(x => x + 1)
    .fold(
        error => `Calculation failed: ${error}`,
        result => `Final result: ${result}`
    );

console.log(complexCalculation); // "Final result: 11"
```

## 🛡️ Type-Safe Error Handling

### 1. **Union Types for Errors**

```typescript
// Define specific error types
type ValidationError = {
    type: "VALIDATION_ERROR";
    field: string;
    message: string;
};

type NetworkError = {
    type: "NETWORK_ERROR";
    statusCode: number;
    message: string;
};

type DatabaseError = {
    type: "DATABASE_ERROR";
    query: string;
    message: string;
};

type NotFoundError = {
    type: "NOT_FOUND_ERROR";
    resource: string;
    id?: string;
};

// Union type for all possible errors
type AppError = ValidationError | NetworkError | DatabaseError | NotFoundError;

// Error factory functions
const createValidationError = (field: string, message: string): ValidationError => ({
    type: "VALIDATION_ERROR",
    field,
    message
});

const createNetworkError = (statusCode: number, message: string): NetworkError => ({
    type: "NETWORK_ERROR",
    statusCode,
    message
});

const createDatabaseError = (query: string, message: string): DatabaseError => ({
    type: "DATABASE_ERROR",
    query,
    message
});

const createNotFoundError = (resource: string, id?: string): NotFoundError => ({
    type: "NOT_FOUND_ERROR",
    resource,
    id
});

// Type-safe error handling with discriminated unions
function handleError(error: AppError): string {
    switch (error.type) {
        case "VALIDATION_ERROR":
            return `Validation failed for ${error.field}: ${error.message}`;
            
        case "NETWORK_ERROR":
            return `Network error (${error.statusCode}): ${error.message}`;
            
        case "DATABASE_ERROR":
            return `Database error in query "${error.query}": ${error.message}`;
            
        case "NOT_FOUND_ERROR":
            return error.id 
                ? `${error.resource} with ID "${error.id}" not found`
                : `${error.resource} not found`;
                
        default:
            // TypeScript will ensure this is never reached
            const _exhaustive: never = error;
            return _exhaustive;
    }
}

// Service layer with type-safe errors
type ServiceResult<T> = Result<T, AppError>;

class UserService {
    async getUser(id: string): Promise<ServiceResult<User>> {
        try {
            if (!id) {
                return Result.err(createValidationError("id", "User ID is required"));
            }
            
            // Simulate API call
            const response = await fetch(`/api/users/${id}`);
            
            if (response.status === 404) {
                return Result.err(createNotFoundError("User", id));
            }
            
            if (!response.ok) {
                return Result.err(createNetworkError(
                    response.status, 
                    `Failed to fetch user: ${response.statusText}`
                ));
            }
            
            const user = await response.json();
            return Result.ok(user);
            
        } catch (error) {
            return Result.err(createNetworkError(
                500, 
                error instanceof Error ? error.message : "Unknown error"
            ));
        }
    }
    
    async createUser(userData: CreateUserRequest): Promise<ServiceResult<User>> {
        try {
            const validation = this.validateUserData(userData);
            if (validation.isErr()) {
                return validation;
            }
            
            // Simulate database operation
            const result = await this.saveUserToDatabase(userData);
            return result;
            
        } catch (error) {
            return Result.err(createDatabaseError(
                "INSERT INTO users",
                error instanceof Error ? error.message : "Database operation failed"
            ));
        }
    }
    
    private validateUserData(userData: CreateUserRequest): ServiceResult<CreateUserRequest> {
        if (!userData.email) {
            return Result.err(createValidationError("email", "Email is required"));
        }
        
        if (!userData.email.includes("@")) {
            return Result.err(createValidationError("email", "Invalid email format"));
        }
        
        if (!userData.name || userData.name.length < 2) {
            return Result.err(createValidationError("name", "Name must be at least 2 characters"));
        }
        
        return Result.ok(userData);
    }
    
    private async saveUserToDatabase(userData: CreateUserRequest): Promise<ServiceResult<User>> {
        // Simulate database save
        return Result.ok({
            id: Date.now().toString(),
            ...userData,
            createdAt: new Date(),
            updatedAt: new Date()
        });
    }
}

// การใช้งาน
const userService = new UserService();

async function handleUserCreation(userData: CreateUserRequest): Promise<void> {
    const result = await userService.createUser(userData);
    
    result.match(
        user => console.log("User created successfully:", user),
        error => console.error("Failed to create user:", handleError(error))
    );
}
```

### 2. **Error Boundary Pattern**

```typescript
// Error boundary for async operations
class AsyncErrorBoundary {
    private static instance: AsyncErrorBoundary;
    private errorHandlers = new Map<string, (error: any) => void>();
    private globalErrorHandler?: (error: any) => void;
    
    static getInstance(): AsyncErrorBoundary {
        if (!AsyncErrorBoundary.instance) {
            AsyncErrorBoundary.instance = new AsyncErrorBoundary();
        }
        return AsyncErrorBoundary.instance;
    }
    
    setGlobalErrorHandler(handler: (error: any) => void): void {
        this.globalErrorHandler = handler;
    }
    
    registerErrorHandler(key: string, handler: (error: any) => void): void {
        this.errorHandlers.set(key, handler);
    }
    
    async execute<T>(
        operation: () => Promise<T>,
        errorKey?: string
    ): Promise<Result<T, any>> {
        try {
            const result = await operation();
            return Result.ok(result);
        } catch (error) {
            this.handleError(error, errorKey);
            return Result.err(error);
        }
    }
    
    executeSync<T>(
        operation: () => T,
        errorKey?: string
    ): Result<T, any> {
        try {
            const result = operation();
            return Result.ok(result);
        } catch (error) {
            this.handleError(error, errorKey);
            return Result.err(error);
        }
    }
    
    private handleError(error: any, errorKey?: string): void {
        if (errorKey && this.errorHandlers.has(errorKey)) {
            this.errorHandlers.get(errorKey)!(error);
        } else if (this.globalErrorHandler) {
            this.globalErrorHandler(error);
        } else {
            console.error("Unhandled error:", error);
        }
    }
}

// การใช้งาน Error Boundary
const errorBoundary = AsyncErrorBoundary.getInstance();

// Set up error handlers
errorBoundary.setGlobalErrorHandler((error) => {
    console.error("Global error handler:", error);
    // Send to logging service
});

errorBoundary.registerErrorHandler("user-service", (error) => {
    console.error("User service error:", error);
    // Handle user service specific errors
});

errorBoundary.registerErrorHandler("payment-service", (error) => {
    console.error("Payment service error:", error);
    // Handle payment service specific errors
    // Maybe rollback transactions
});

// Service implementations
class PaymentService {
    async processPayment(amount: number): Promise<Result<PaymentResult, any>> {
        return errorBoundary.execute(async () => {
            if (amount <= 0) {
                throw new Error("Invalid payment amount");
            }
            
            // Simulate payment processing
            await new Promise(resolve => setTimeout(resolve, 1000));
            
            if (Math.random() < 0.1) { // 10% chance of failure
                throw new Error("Payment gateway error");
            }
            
            return {
                transactionId: `txn_${Date.now()}`,
                amount,
                status: "completed" as const
            };
        }, "payment-service");
    }
}

// Usage with error boundary
async function handlePayment(amount: number): Promise<void> {
    const paymentService = new PaymentService();
    const result = await paymentService.processPayment(amount);
    
    if (result.isOk()) {
        console.log("Payment processed:", result.data);
    } else {
        console.log("Payment failed - error already handled by boundary");
    }
}
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **API Service with Comprehensive Error Handling**

```typescript
// API Error types
type APIError = 
    | { type: "NETWORK_ERROR"; status: number; message: string }
    | { type: "TIMEOUT_ERROR"; timeout: number }
    | { type: "PARSE_ERROR"; response: string }
    | { type: "VALIDATION_ERROR"; errors: Record<string, string[]> };

class APIService {
    private baseURL: string;
    private timeout: number;
    
    constructor(baseURL: string, timeout: number = 5000) {
        this.baseURL = baseURL;
        this.timeout = timeout;
    }
    
    async get<T>(endpoint: string): Promise<Result<T, APIError>> {
        return this.request<T>("GET", endpoint);
    }
    
    async post<T>(endpoint: string, data?: any): Promise<Result<T, APIError>> {
        return this.request<T>("POST", endpoint, data);
    }
    
    async put<T>(endpoint: string, data?: any): Promise<Result<T, APIError>> {
        return this.request<T>("PUT", endpoint, data);
    }
    
    async delete<T>(endpoint: string): Promise<Result<T, APIError>> {
        return this.request<T>("DELETE", endpoint);
    }
    
    private async request<T>(
        method: string, 
        endpoint: string, 
        data?: any
    ): Promise<Result<T, APIError>> {
        try {
            const controller = new AbortController();
            const timeoutId = setTimeout(() => controller.abort(), this.timeout);
            
            const config: RequestInit = {
                method,
                signal: controller.signal,
                headers: {
                    "Content-Type": "application/json",
                },
            };
            
            if (data) {
                config.body = JSON.stringify(data);
            }
            
            const response = await fetch(`${this.baseURL}${endpoint}`, config);
            clearTimeout(timeoutId);
            
            if (!response.ok) {
                if (response.status === 422) {
                    const errorData = await response.json();
                    return Result.err({
                        type: "VALIDATION_ERROR",
                        errors: errorData.errors
                    });
                }
                
                return Result.err({
                    type: "NETWORK_ERROR",
                    status: response.status,
                    message: response.statusText
                });
            }
            
            const responseData = await response.json();
            return Result.ok(responseData);
            
        } catch (error) {
            if (error instanceof Error) {
                if (error.name === "AbortError") {
                    return Result.err({
                        type: "TIMEOUT_ERROR",
                        timeout: this.timeout
                    });
                }
                
                return Result.err({
                    type: "PARSE_ERROR",
                    response: error.message
                });
            }
            
            return Result.err({
                type: "NETWORK_ERROR",
                status: 0,
                message: "Unknown error"
            });
        }
    }
}

// Usage with comprehensive error handling
class UserRepository {
    private api: APIService;
    
    constructor() {
        this.api = new APIService("/api", 10000);
    }
    
    async getUser(id: string): Promise<Result<User, string>> {
        const result = await this.api.get<User>(`/users/${id}`);
        
        return result.mapError(error => {
            switch (error.type) {
                case "NETWORK_ERROR":
                    if (error.status === 404) {
                        return `User with ID ${id} not found`;
                    }
                    return `Network error: ${error.message} (${error.status})`;
                    
                case "TIMEOUT_ERROR":
                    return `Request timed out after ${error.timeout}ms`;
                    
                case "PARSE_ERROR":
                    return `Failed to parse response: ${error.response}`;
                    
                case "VALIDATION_ERROR":
                    return `Validation error: ${JSON.stringify(error.errors)}`;
                    
                default:
                    const _exhaustive: never = error;
                    return _exhaustive;
            }
        });
    }
    
    async createUser(userData: CreateUserRequest): Promise<Result<User, string[]>> {
        const result = await this.api.post<User>("/users", userData);
        
        return result.mapError(error => {
            switch (error.type) {
                case "VALIDATION_ERROR":
                    return Object.entries(error.errors)
                        .flatMap(([field, errors]) => 
                            errors.map(err => `${field}: ${err}`)
                        );
                        
                case "NETWORK_ERROR":
                    return [`Network error: ${error.message}`];
                    
                case "TIMEOUT_ERROR":
                    return [`Request timed out after ${error.timeout}ms`];
                    
                case "PARSE_ERROR":
                    return [`Failed to parse response: ${error.response}`];
                    
                default:
                    const _exhaustive: never = error;
                    return _exhaustive;
            }
        });
    }
}
```

### 2. **Database Service with Transaction Rollback**

```typescript
// Database operation types
type DatabaseOperation<T> = () => Promise<T>;

interface Transaction {
    id: string;
    rollback(): Promise<void>;
    commit(): Promise<void>;
}

class DatabaseService {
    private connection: any; // Database connection
    
    async withTransaction<T>(
        operation: (tx: Transaction) => Promise<Result<T, string>>
    ): Promise<Result<T, string>> {
        const transaction = await this.beginTransaction();
        
        try {
            const result = await operation(transaction);
            
            if (result.isOk()) {
                await transaction.commit();
                return result;
            } else {
                await transaction.rollback();
                return result;
            }
        } catch (error) {
            await transaction.rollback();
            return Result.err(
                error instanceof Error ? error.message : "Transaction failed"
            );
        }
    }
    
    async executeWithRetry<T>(
        operation: DatabaseOperation<T>,
        maxRetries: number = 3,
        delay: number = 1000
    ): Promise<Result<T, string>> {
        let lastError: any;
        
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                const result = await operation();
                return Result.ok(result);
            } catch (error) {
                lastError = error;
                console.warn(`Database operation failed (attempt ${attempt}/${maxRetries}):`, error);
                
                if (attempt < maxRetries) {
                    await new Promise(resolve => setTimeout(resolve, delay * attempt));
                }
            }
        }
        
        return Result.err(
            lastError instanceof Error 
                ? `Operation failed after ${maxRetries} attempts: ${lastError.message}`
                : "Operation failed after multiple attempts"
        );
    }
    
    private async beginTransaction(): Promise<Transaction> {
        // Implementation would create actual database transaction
        const id = `tx_${Date.now()}`;
        
        return {
            id,
            rollback: async () => {
                console.log(`Rolling back transaction ${id}`);
                // Actual rollback implementation
            },
            commit: async () => {
                console.log(`Committing transaction ${id}`);
                // Actual commit implementation
            }
        };
    }
}

// Usage example
class OrderService {
    private db: DatabaseService;
    
    constructor() {
        this.db = new DatabaseService();
    }
    
    async createOrder(orderData: CreateOrderRequest): Promise<Result<Order, string>> {
        return this.db.withTransaction(async (tx) => {
            // Create order
            const orderResult = await this.db.executeWithRetry(async () => {
                // Insert order into database
                return { id: Date.now().toString(), ...orderData };
            });
            
            if (orderResult.isErr()) {
                return orderResult;
            }
            
            // Update inventory
            const inventoryResult = await this.db.executeWithRetry(async () => {
                // Update inventory in database
                return true;
            });
            
            if (inventoryResult.isErr()) {
                return Result.err(`Failed to update inventory: ${inventoryResult.error}`);
            }
            
            // Send notification
            const notificationResult = await this.sendOrderNotification(orderResult.data);
            if (notificationResult.isErr()) {
                // Notification failure shouldn't rollback the transaction
                console.warn("Failed to send notification:", notificationResult.error);
            }
            
            return orderResult;
        });
    }
    
    private async sendOrderNotification(order: Order): Promise<Result<void, string>> {
        try {
            // Send notification logic
            return Result.ok(undefined);
        } catch (error) {
            return Result.err(
                error instanceof Error ? error.message : "Notification failed"
            );
        }
    }
}
```

## 📝 แบบฝึกหัด

### 1. **Email Service with Error Recovery**
สร้าง service สำหรับส่ง email ที่มีการจัดการ error:
- Retry mechanism with exponential backoff
- Fallback to different email providers
- Queue failed emails for later retry
- Comprehensive error reporting

### 2. **File Processing Pipeline**
ออกแบบระบบประมวลผลไฟล์ที่มี error handling:
- File validation
- Processing stages with rollback capability
- Partial success handling
- Progress tracking with error recovery

### 3. **Cache Service with Error Handling**
สร้าง cache service ที่จัดการ error ได้ดี:
- Network failure fallback
- Cache corruption recovery
- TTL management with error scenarios
- Multi-tier caching with failover

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 13: Utility Types](./13-utility-types.md)

**บทต่อไป**: [บทที่ 15: Testing TypeScript](./15-testing.md)

**กลับไปหน้าหลัก**: [README](./README.md)
