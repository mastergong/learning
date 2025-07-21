# บทที่ 20: การให้คำแนะนำและเทคนิคขั้นสูง

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้วิธีการให้คำแนะนำ TypeScript ที่มีประสิทธิภาพ
- เข้าใจเทคนิคการแก้ปัญหาที่ซับซ้อน
- เรียนรู้วิธีการ mentor และถ่ายทอดความรู้
- เข้าใจการพัฒนาทีมและการทำงานร่วมกัน

## 🧠 การให้คำแนะนำด้าน Architecture

### 1. **การออกแบบ Type System**

```typescript
// ❌ การออกแบบที่ไม่ดี - ใช้ any มากเกินไป
interface BadApiResponse {
    data: any;
    status: any;
    headers: any;
}

// ✅ การออกแบบที่ดี - Type-safe และ maintainable
interface ApiResponse<T> {
    data: T;
    status: {
        code: number;
        message: string;
        success: boolean;
    };
    headers: Record<string, string>;
    metadata: {
        timestamp: Date;
        requestId: string;
        version: string;
    };
}

// การใช้งาน
interface User {
    id: string;
    name: string;
    email: string;
    roles: string[];
}

type UserResponse = ApiResponse<User>;
type UsersResponse = ApiResponse<User[]>;

// Generic helper สำหรับสร้าง Response types
type PaginatedResponse<T> = ApiResponse<{
    items: T[];
    pagination: {
        page: number;
        pageSize: number;
        total: number;
        hasNext: boolean;
        hasPrev: boolean;
    };
}>;
```

### 2. **Domain-Driven Design กับ TypeScript**

```typescript
// Value Objects
class Email {
    private constructor(private readonly value: string) {}
    
    static create(email: string): Email | Error {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(email)) {
            return new Error("Invalid email format");
        }
        return new Email(email);
    }
    
    toString(): string {
        return this.value;
    }
    
    equals(other: Email): boolean {
        return this.value === other.value;
    }
}

class UserId {
    private constructor(private readonly value: string) {}
    
    static create(id: string): UserId | Error {
        if (!id || id.trim().length === 0) {
            return new Error("User ID cannot be empty");
        }
        return new UserId(id);
    }
    
    toString(): string {
        return this.value;
    }
}

// Entities
class User {
    constructor(
        private readonly id: UserId,
        private name: string,
        private email: Email,
        private readonly createdAt: Date = new Date()
    ) {}
    
    getId(): UserId {
        return this.id;
    }
    
    getName(): string {
        return this.name;
    }
    
    getEmail(): Email {
        return this.email;
    }
    
    changeName(newName: string): void {
        if (!newName || newName.trim().length === 0) {
            throw new Error("Name cannot be empty");
        }
        this.name = newName;
    }
    
    changeEmail(newEmail: Email): void {
        this.email = newEmail;
    }
}

// Repository Pattern
interface UserRepository {
    findById(id: UserId): Promise<User | null>;
    findByEmail(email: Email): Promise<User | null>;
    save(user: User): Promise<void>;
    delete(id: UserId): Promise<void>;
}

// Use Cases
class CreateUserUseCase {
    constructor(private userRepository: UserRepository) {}
    
    async execute(request: {
        name: string;
        email: string;
    }): Promise<{ success: boolean; userId?: string; error?: string }> {
        try {
            // Validate input
            const email = Email.create(request.email);
            if (email instanceof Error) {
                return { success: false, error: email.message };
            }
            
            // Check if user already exists
            const existingUser = await this.userRepository.findByEmail(email);
            if (existingUser) {
                return { success: false, error: "User already exists" };
            }
            
            // Create new user
            const userId = UserId.create(generateId());
            if (userId instanceof Error) {
                return { success: false, error: userId.message };
            }
            
            const user = new User(userId, request.name, email);
            await this.userRepository.save(user);
            
            return { success: true, userId: userId.toString() };
        } catch (error) {
            return { 
                success: false, 
                error: error instanceof Error ? error.message : "Unknown error" 
            };
        }
    }
}

function generateId(): string {
    return Math.random().toString(36).substring(2) + Date.now().toString(36);
}
```

## 🔧 Advanced Debugging Techniques

### 1. **Type-Level Debugging**

```typescript
// Type-level utilities สำหรับ debugging
type Debug<T> = T extends infer U ? { [K in keyof U]: U[K] } : never;

type Expect<T extends true> = T;
type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y ? 1 : 2
    ? true
    : false;

// ตัวอย่างการใช้งาน
interface User {
    id: string;
    name: string;
    email: string;
}

type UserKeys = keyof User; // "id" | "name" | "email"
type DebugUserKeys = Debug<UserKeys>; // สำหรับดู type ใน IDE

// Test types
type TestUserKeys = Expect<Equal<UserKeys, "id" | "name" | "email">>;

// Advanced type debugging
type Prettify<T> = {
    [K in keyof T]: T[K];
} & {};

interface ComplexType {
    a: string;
    b: number;
    c: {
        nested: boolean;
        deep: {
            value: string;
        };
    };
}

type PrettifiedComplex = Prettify<ComplexType>;
```

### 2. **Runtime Debugging Utilities**

```typescript
// Type-safe logger
enum LogLevel {
    DEBUG = 0,
    INFO = 1,
    WARN = 2,
    ERROR = 3
}

interface LogContext {
    userId?: string;
    requestId?: string;
    timestamp: Date;
    level: LogLevel;
}

class TypeSafeLogger {
    private currentLevel: LogLevel = LogLevel.INFO;
    
    setLevel(level: LogLevel): void {
        this.currentLevel = level;
    }
    
    private log<T>(level: LogLevel, message: string, data?: T, context?: Partial<LogContext>): void {
        if (level < this.currentLevel) return;
        
        const logEntry = {
            message,
            level: LogLevel[level],
            timestamp: new Date().toISOString(),
            data,
            ...context
        };
        
        console.log(JSON.stringify(logEntry, null, 2));
    }
    
    debug<T>(message: string, data?: T, context?: Partial<LogContext>): void {
        this.log(LogLevel.DEBUG, message, data, context);
    }
    
    info<T>(message: string, data?: T, context?: Partial<LogContext>): void {
        this.log(LogLevel.INFO, message, data, context);
    }
    
    warn<T>(message: string, data?: T, context?: Partial<LogContext>): void {
        this.log(LogLevel.WARN, message, data, context);
    }
    
    error<T>(message: string, data?: T, context?: Partial<LogContext>): void {
        this.log(LogLevel.ERROR, message, data, context);
    }
}

// Performance monitoring
function measurePerformance<T extends (...args: any[]) => any>(
    target: T,
    label: string
): T {
    return ((...args: Parameters<T>) => {
        const start = performance.now();
        const result = target(...args);
        const end = performance.now();
        
        console.log(`${label} took ${end - start} milliseconds`);
        return result;
    }) as T;
}

// Usage
const logger = new TypeSafeLogger();
logger.setLevel(LogLevel.DEBUG);

const slowFunction = measurePerformance(
    (n: number) => {
        let sum = 0;
        for (let i = 0; i < n; i++) {
            sum += i;
        }
        return sum;
    },
    "Sum calculation"
);

logger.info("Starting calculation", { input: 1000000 });
const result = slowFunction(1000000);
logger.info("Calculation complete", { result });
```

## 🎓 การ Mentor และถ่ายทอดความรู้

### 1. **Code Review Best Practices**

```typescript
// ❌ Code ที่ควรได้รับ feedback
class BadUserService {
    async getUser(id: any): Promise<any> {
        const response = await fetch(`/api/users/${id}`);
        return response.json();
    }
    
    async createUser(data: any): Promise<any> {
        // ไม่มี validation
        const response = await fetch('/api/users', {
            method: 'POST',
            body: JSON.stringify(data)
        });
        return response.json();
    }
}

// ✅ Code หลัง feedback และปรับปรุง
interface User {
    id: string;
    name: string;
    email: string;
    createdAt: Date;
    updatedAt: Date;
}

interface CreateUserRequest {
    name: string;
    email: string;
}

interface ApiError {
    message: string;
    code: string;
    details?: Record<string, unknown>;
}

type Result<T, E = ApiError> = 
    | { success: true; data: T }
    | { success: false; error: E };

class UserService {
    private readonly baseUrl: string;
    
    constructor(baseUrl: string) {
        this.baseUrl = baseUrl;
    }
    
    async getUser(id: string): Promise<Result<User>> {
        try {
            if (!id || id.trim().length === 0) {
                return {
                    success: false,
                    error: { message: "User ID is required", code: "INVALID_INPUT" }
                };
            }
            
            const response = await fetch(`${this.baseUrl}/users/${id}`, {
                headers: {
                    'Accept': 'application/json',
                }
            });
            
            if (!response.ok) {
                const error = await response.json();
                return { success: false, error };
            }
            
            const user = await response.json();
            return { success: true, data: user };
        } catch (error) {
            return {
                success: false,
                error: {
                    message: "Network error",
                    code: "NETWORK_ERROR",
                    details: { originalError: error }
                }
            };
        }
    }
    
    async createUser(request: CreateUserRequest): Promise<Result<User>> {
        try {
            // Validation
            const validationResult = this.validateCreateUserRequest(request);
            if (!validationResult.isValid) {
                return {
                    success: false,
                    error: {
                        message: "Validation failed",
                        code: "VALIDATION_ERROR",
                        details: { errors: validationResult.errors }
                    }
                };
            }
            
            const response = await fetch(`${this.baseUrl}/users`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json',
                },
                body: JSON.stringify(request)
            });
            
            if (!response.ok) {
                const error = await response.json();
                return { success: false, error };
            }
            
            const user = await response.json();
            return { success: true, data: user };
        } catch (error) {
            return {
                success: false,
                error: {
                    message: "Failed to create user",
                    code: "CREATE_USER_ERROR",
                    details: { originalError: error }
                }
            };
        }
    }
    
    private validateCreateUserRequest(request: CreateUserRequest): {
        isValid: boolean;
        errors: string[];
    } {
        const errors: string[] = [];
        
        if (!request.name || request.name.trim().length === 0) {
            errors.push("Name is required");
        }
        
        if (!request.email || request.email.trim().length === 0) {
            errors.push("Email is required");
        } else {
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test(request.email)) {
                errors.push("Invalid email format");
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    }
}
```

### 2. **Progressive Learning Path**

```typescript
// Level 1: Basic TypeScript concepts
interface BasicConcepts {
    // เริ่มต้นด้วย basic types
    name: string;
    age: number;
    isActive: boolean;
}

// Level 2: Intermediate concepts
interface IntermediateConcepts<T> {
    // Generic types
    data: T;
    // Union types
    status: "pending" | "completed" | "failed";
    // Optional properties
    metadata?: Record<string, unknown>;
}

// Level 3: Advanced concepts
type AdvancedConcepts<T extends Record<string, unknown>> = {
    // Mapped types
    [K in keyof T]: T[K] extends string ? `processed_${T[K]}` : T[K];
} & {
    // Conditional types
    processedAt: T extends { timestamp: unknown } ? Date : never;
};

// Level 4: Expert level
type ExpertLevel<T> = T extends infer U 
    ? U extends readonly unknown[]
        ? {
            readonly [K in keyof U]: U[K] extends object 
                ? ExpertLevel<U[K]>
                : U[K]
        }
        : U extends object
            ? {
                readonly [K in keyof U]: ExpertLevel<U[K]>
            }
            : U
    : never;
```

## 🏢 การทำงานเป็นทีม

### 1. **TypeScript Style Guide**

```typescript
// Naming conventions
interface PascalCaseInterface {
    // Interface ใช้ PascalCase
}

type PascalCaseType = string | number; // Type aliases ใช้ PascalCase

class PascalCaseClass {} // Class ใช้ PascalCase

enum PascalCaseEnum { // Enum ใช้ PascalCase
    UPPER_SNAKE_CASE_VALUE = "value" // Enum values ใช้ UPPER_SNAKE_CASE
}

const UPPER_SNAKE_CASE_CONSTANT = "constant"; // Constants ใช้ UPPER_SNAKE_CASE

function camelCaseFunction() {} // Functions ใช้ camelCase

const camelCaseVariable = "variable"; // Variables ใช้ camelCase

// File naming conventions
// interfaces/user-profile.interface.ts
// types/api-response.type.ts
// services/user.service.ts
// utils/string.util.ts

// Import/Export conventions
// Named exports สำหรับ utilities
export const formatCurrency = (amount: number): string => {
    return new Intl.NumberFormat('th-TH', {
        style: 'currency',
        currency: 'THB'
    }).format(amount);
};

// Default exports สำหรับ main classes/components
export default class UserService {
    // implementation
}

// Barrel exports
// index.ts
export { UserService } from './user.service';
export { formatCurrency } from './string.util';
export type { ApiResponse } from './api-response.type';
```

### 2. **Documentation Standards**

```typescript
/**
 * Represents a user in the system
 * @example
 * ```typescript
 * const user = new User("123", "John Doe", "john@example.com");
 * console.log(user.getDisplayName()); // "John Doe"
 * ```
 */
class User {
    /**
     * Creates a new User instance
     * @param id - Unique identifier for the user
     * @param name - Full name of the user
     * @param email - Email address of the user
     * @throws {Error} When email format is invalid
     */
    constructor(
        private readonly id: string,
        private readonly name: string,
        private readonly email: string
    ) {
        if (!this.isValidEmail(email)) {
            throw new Error(`Invalid email format: ${email}`);
        }
    }
    
    /**
     * Gets the user's display name
     * @returns The formatted display name
     * @since 1.0.0
     */
    getDisplayName(): string {
        return this.name;
    }
    
    /**
     * Validates email format
     * @param email - Email to validate
     * @returns True if email is valid
     * @internal
     */
    private isValidEmail(email: string): boolean {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }
}

/**
 * Configuration options for API calls
 * @public
 */
interface ApiOptions {
    /** Request timeout in milliseconds */
    timeout?: number;
    /** Number of retry attempts */
    retries?: number;
    /** Additional headers to include */
    headers?: Record<string, string>;
}

/**
 * Makes an API call with the specified options
 * @param url - The URL to call
 * @param options - Configuration options
 * @returns Promise that resolves to the API response
 * 
 * @example
 * ```typescript
 * const response = await makeApiCall('https://api.example.com/users', {
 *   timeout: 5000,
 *   retries: 3
 * });
 * ```
 * 
 * @throws {ApiError} When the API call fails
 * @see {@link ApiOptions} for available options
 */
async function makeApiCall<T>(
    url: string, 
    options: ApiOptions = {}
): Promise<T> {
    // implementation
    throw new Error("Not implemented");
}
```

### 3. **Testing Strategy**

```typescript
// Type-safe testing utilities
interface TestUser {
    id: string;
    name: string;
    email: string;
    createdAt: Date;
}

// Test data builders
class TestUserBuilder {
    private user: Partial<TestUser> = {};
    
    withId(id: string): TestUserBuilder {
        this.user.id = id;
        return this;
    }
    
    withName(name: string): TestUserBuilder {
        this.user.name = name;
        return this;
    }
    
    withEmail(email: string): TestUserBuilder {
        this.user.email = email;
        return this;
    }
    
    withCreatedAt(date: Date): TestUserBuilder {
        this.user.createdAt = date;
        return this;
    }
    
    build(): TestUser {
        return {
            id: this.user.id ?? "test-id",
            name: this.user.name ?? "Test User",
            email: this.user.email ?? "test@example.com",
            createdAt: this.user.createdAt ?? new Date()
        };
    }
}

// Mock utilities
type MockFunction<T extends (...args: any[]) => any> = jest.MockedFunction<T>;

interface MockUserService {
    getUser: MockFunction<(id: string) => Promise<Result<User>>>;
    createUser: MockFunction<(request: CreateUserRequest) => Promise<Result<User>>>;
}

function createMockUserService(): MockUserService {
    return {
        getUser: jest.fn(),
        createUser: jest.fn()
    };
}

// Test example
describe("UserService", () => {
    let userService: UserService;
    let mockUserService: MockUserService;
    
    beforeEach(() => {
        mockUserService = createMockUserService();
        userService = new UserService("https://api.test.com");
    });
    
    describe("getUser", () => {
        it("should return user when id is valid", async () => {
            // Arrange
            const expectedUser = new TestUserBuilder()
                .withId("123")
                .withName("John Doe")
                .build();
            
            mockUserService.getUser.mockResolvedValue({
                success: true,
                data: expectedUser
            });
            
            // Act
            const result = await userService.getUser("123");
            
            // Assert
            expect(result.success).toBe(true);
            if (result.success) {
                expect(result.data.id).toBe("123");
                expect(result.data.name).toBe("John Doe");
            }
        });
        
        it("should return error when id is empty", async () => {
            // Act
            const result = await userService.getUser("");
            
            // Assert
            expect(result.success).toBe(false);
            if (!result.success) {
                expect(result.error.code).toBe("INVALID_INPUT");
            }
        });
    });
});
```

## 📚 การพัฒนาต่อยอด

### 1. **Learning Resources**

```typescript
// Recommended learning path
interface LearningPath {
    beginner: {
        topics: [
            "Basic Types",
            "Interfaces",
            "Functions",
            "Classes"
        ];
        resources: [
            "TypeScript Handbook",
            "TypeScript Playground",
            "Official Documentation"
        ];
        timeframe: "2-4 weeks";
    };
    
    intermediate: {
        topics: [
            "Generics",
            "Advanced Types",
            "Utility Types",
            "Decorators"
        ];
        resources: [
            "Advanced TypeScript Courses",
            "Type Challenges",
            "Real Projects"
        ];
        timeframe: "1-2 months";
    };
    
    advanced: {
        topics: [
            "Template Literal Types",
            "Conditional Types",
            "Mapped Types",
            "Type-level Programming"
        ];
        resources: [
            "TypeScript Source Code",
            "Community Contributions",
            "Conference Talks"
        ];
        timeframe: "3-6 months";
    };
    
    expert: {
        topics: [
            "Compiler API",
            "Type System Internals",
            "Performance Optimization",
            "Teaching Others"
        ];
        resources: [
            "Contributing to TypeScript",
            "Writing Articles",
            "Speaking at Conferences"
        ];
        timeframe: "Ongoing";
    };
}
```

### 2. **Community Contribution**

```typescript
// Open source contribution guidelines
interface ContributionGuidelines {
    codeContribution: {
        steps: [
            "Find good first issues",
            "Read contribution guidelines",
            "Write tests",
            "Submit PR with clear description"
        ];
        bestPractices: [
            "Follow existing code style",
            "Add comprehensive tests",
            "Update documentation",
            "Be responsive to feedback"
        ];
    };
    
    documentation: {
        types: [
            "Fix typos and errors",
            "Add examples",
            "Improve explanations",
            "Translate content"
        ];
        tools: [
            "MDX for documentation",
            "TypeScript for examples",
            "GitHub for collaboration"
        ];
    };
    
    community: {
        activities: [
            "Answer questions on Stack Overflow",
            "Help in Discord/Slack channels",
            "Write blog posts",
            "Create video tutorials"
        ];
        platforms: [
            "GitHub Discussions",
            "TypeScript Discord",
            "Reddit r/typescript",
            "Dev.to"
        ];
    };
}
```

## 📝 แบบฝึกหัด Expert Level

### 1. **Architecture Review**
ออกแบบ type system สำหรับ e-commerce platform ที่มี:
- Product catalog with variants
- Shopping cart with pricing rules
- Order processing workflow
- User permission system

### 2. **Mentoring Simulation**
ให้คำแนะนำสำหรับ code review cases:
- Junior developer's first TypeScript PR
- Refactoring legacy JavaScript to TypeScript
- Performance optimization in large codebase

### 3. **Advanced Problem Solving**
แก้ปัญหา TypeScript ที่ซับซ้อน:
- Type inference issues in complex generics
- Module resolution problems
- Build performance optimization

## 🎓 สรุป: เส้นทางสู่ TypeScript Expert

เมื่อคุณได้เรียนรู้ TypeScript จากพื้นฐานมาจนถึงระดับขั้นสูงแล้ว การเป็น Expert ไม่ได้หยุดแค่การเขียนโค้ดที่ดี แต่รวมถึง:

1. **การถ่ายทอดความรู้** - สอนและ mentor คนอื่น
2. **การพัฒนาชุมชน** - มีส่วนร่วมใน open source
3. **การแก้ปัญหาระดับระบบ** - ออกแบบ architecture ที่ดี
4. **การเรียนรู้ต่อเนื่อง** - ติดตามเทคโนโลยีใหม่ๆ

TypeScript เป็นเครื่องมือที่ทรงพลัง และการเป็น Expert หมายถึงการใช้พลังนั้นเพื่อสร้างสรรค์สิ่งที่ดีให้กับชุมชนนักพัฒนา

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 19: Real-world Projects](./19-real-world-projects.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

**🎉 ยินดีด้วย! คุณได้เรียนจบคอร์ส TypeScript: Zero to Hero แล้ว!** 

ตอนนี้คุณพร้อมที่จะเป็น TypeScript Expert และถ่ายทอดความรู้ให้กับคนอื่นๆ ต่อไป!
