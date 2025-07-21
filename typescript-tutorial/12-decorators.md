# บทที่ 12: Decorators

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจแนวคิดและการใช้งาน Decorators
- เรียนรู้ Class, Method, Property, และ Parameter Decorators
- เข้าใจ Decorator Factories และ Metadata
- เรียนรู้การสร้าง Custom Decorators ที่มีประโยชน์

## 🎨 แนะนำ Decorators

### 1. **Decorator คืออะไร?**

```typescript
// Decorator เป็น function ที่ใช้แต่งแต้ง class, method, property, หรือ parameter
// ต้องเปิดใช้งาน experimental decorators ใน tsconfig.json

// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}

// Basic decorator
function LogClass(target: any) {
    console.log(`Class ${target.name} is being decorated`);
}

@LogClass
class ExampleClass {
    constructor(public name: string) {}
}

// เมื่อ TypeScript โหลด class นี้ จะแสดง:
// "Class ExampleClass is being decorated"
```

### 2. **Decorator Execution Order**

```typescript
function ClassDecorator(name: string) {
    console.log(`ClassDecorator ${name} evaluated`);
    return function(target: any) {
        console.log(`ClassDecorator ${name} called`);
    };
}

function MethodDecorator(name: string) {
    console.log(`MethodDecorator ${name} evaluated`);
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log(`MethodDecorator ${name} called`);
    };
}

function PropertyDecorator(name: string) {
    console.log(`PropertyDecorator ${name} evaluated`);
    return function(target: any, propertyKey: string) {
        console.log(`PropertyDecorator ${name} called`);
    };
}

@ClassDecorator("first")
@ClassDecorator("second")
class DecoratorOrderExample {
    @PropertyDecorator("property1")
    property1: string = "";
    
    @MethodDecorator("method1")
    method1() {}
}

// Output order:
// PropertyDecorator property1 evaluated
// PropertyDecorator property1 called
// MethodDecorator method1 evaluated
// MethodDecorator method1 called
// ClassDecorator first evaluated
// ClassDecorator second evaluated
// ClassDecorator second called
// ClassDecorator first called
```

## 🏛️ Class Decorators

### 1. **Basic Class Decorators**

```typescript
// Simple class decorator
function Sealed(target: any) {
    Object.seal(target);
    Object.seal(target.prototype);
}

@Sealed
class SealedClass {
    name: string;
    
    constructor(name: string) {
        this.name = name;
    }
}

// Class decorator with return value
function Component(config: { selector: string; template: string }) {
    return function<T extends { new(...args: any[]): {} }>(constructor: T) {
        return class extends constructor {
            selector = config.selector;
            template = config.template;
            
            render() {
                console.log(`Rendering ${this.selector}: ${this.template}`);
            }
        };
    };
}

@Component({
    selector: "my-component",
    template: "<div>Hello World</div>"
})
class MyComponent {
    constructor(public title: string) {}
}

const component = new MyComponent("Test");
// component.render(); // "Rendering my-component: <div>Hello World</div>"
```

### 2. **Advanced Class Decorators**

```typescript
// Mixin decorator
function Timestamped<T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
        createdAt = new Date();
        updatedAt = new Date();
        
        touch() {
            this.updatedAt = new Date();
        }
    };
}

@Timestamped
class User {
    constructor(public name: string, public email: string) {}
}

const user = new User("John", "john@example.com");
console.log(user.createdAt); // Current date
// user.touch(); // Updates updatedAt

// Singleton decorator
function Singleton<T extends { new(...args: any[]): {} }>(constructor: T) {
    let instance: T;
    
    return class {
        constructor(...args: any[]) {
            if (instance) {
                return instance;
            }
            instance = new constructor(...args) as T;
            return instance;
        }
    } as any;
}

@Singleton
class DatabaseConnection {
    constructor(public connectionString: string) {}
    
    connect() {
        console.log(`Connecting to ${this.connectionString}`);
    }
}

const db1 = new DatabaseConnection("server1");
const db2 = new DatabaseConnection("server2");
console.log(db1 === db2); // true (same instance)
```

## 🔧 Method Decorators

### 1. **Basic Method Decorators**

```typescript
// Method execution logging
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with arguments:`, args);
        const result = originalMethod.apply(this, args);
        console.log(`${propertyKey} returned:`, result);
        return result;
    };
}

class Calculator {
    @Log
    add(a: number, b: number): number {
        return a + b;
    }
    
    @Log
    multiply(a: number, b: number): number {
        return a * b;
    }
}

const calc = new Calculator();
calc.add(2, 3); 
// Calling add with arguments: [2, 3]
// add returned: 5
```

### 2. **Performance Monitoring**

```typescript
// Performance measurement decorator
function Measure(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        const start = performance.now();
        const result = originalMethod.apply(this, args);
        const end = performance.now();
        
        console.log(`${propertyKey} took ${end - start} milliseconds`);
        return result;
    };
}

// Async method measurement
function MeasureAsync(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args: any[]) {
        const start = performance.now();
        const result = await originalMethod.apply(this, args);
        const end = performance.now();
        
        console.log(`${propertyKey} took ${end - start} milliseconds`);
        return result;
    };
}

class DataService {
    @Measure
    processData(data: number[]): number[] {
        return data.map(x => x * 2);
    }
    
    @MeasureAsync
    async fetchData(): Promise<any[]> {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 100));
        return [1, 2, 3, 4, 5];
    }
}
```

### 3. **Validation Decorators**

```typescript
// Method parameter validation
function ValidateParams(...validators: ((value: any) => boolean)[]) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = function(...args: any[]) {
            for (let i = 0; i < validators.length; i++) {
                if (validators[i] && !validators[i](args[i])) {
                    throw new Error(`Invalid parameter at index ${i} for method ${propertyKey}`);
                }
            }
            
            return originalMethod.apply(this, args);
        };
    };
}

// Validators
const isPositiveNumber = (value: any) => typeof value === "number" && value > 0;
const isNonEmptyString = (value: any) => typeof value === "string" && value.length > 0;

class UserService {
    @ValidateParams(isNonEmptyString, isNonEmptyString)
    createUser(name: string, email: string): void {
        console.log(`Creating user: ${name} (${email})`);
    }
    
    @ValidateParams(isPositiveNumber)
    deleteUser(id: number): void {
        console.log(`Deleting user with ID: ${id}`);
    }
}

const userService = new UserService();
// userService.createUser("", "test@test.com"); // Error: Invalid parameter
// userService.deleteUser(-1); // Error: Invalid parameter
```

## 🏷️ Property Decorators

### 1. **Property Validation**

```typescript
// Required property decorator
function Required(target: any, propertyKey: string) {
    let value: any;
    
    const getter = () => {
        if (value === undefined) {
            throw new Error(`Property ${propertyKey} is required`);
        }
        return value;
    };
    
    const setter = (newValue: any) => {
        if (newValue === undefined || newValue === null) {
            throw new Error(`Property ${propertyKey} cannot be null or undefined`);
        }
        value = newValue;
    };
    
    Object.defineProperty(target, propertyKey, {
        get: getter,
        set: setter,
        enumerable: true,
        configurable: true
    });
}

// Range validation decorator
function Range(min: number, max: number) {
    return function(target: any, propertyKey: string) {
        let value: number;
        
        const getter = () => value;
        const setter = (newValue: number) => {
            if (newValue < min || newValue > max) {
                throw new Error(`${propertyKey} must be between ${min} and ${max}`);
            }
            value = newValue;
        };
        
        Object.defineProperty(target, propertyKey, {
            get: getter,
            set: setter,
            enumerable: true,
            configurable: true
        });
    };
}

class Product {
    @Required
    name: string;
    
    @Range(0, 100)
    discount: number;
    
    constructor(name: string, discount: number) {
        this.name = name;
        this.discount = discount;
    }
}

// const product = new Product("", 150); // Errors will be thrown
```

### 2. **Property Transformation**

```typescript
// Format decorator
function Format(formatter: (value: any) => any) {
    return function(target: any, propertyKey: string) {
        let value: any;
        
        const getter = () => value;
        const setter = (newValue: any) => {
            value = formatter(newValue);
        };
        
        Object.defineProperty(target, propertyKey, {
            get: getter,
            set: setter,
            enumerable: true,
            configurable: true
        });
    };
}

// Capitalize decorator
const capitalize = (value: string) => 
    value.charAt(0).toUpperCase() + value.slice(1).toLowerCase();

// Trim decorator
const trim = (value: string) => value.trim();

class Person {
    @Format(capitalize)
    firstName: string;
    
    @Format(capitalize)
    lastName: string;
    
    @Format(trim)
    email: string;
    
    constructor(firstName: string, lastName: string, email: string) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }
}

const person = new Person("john", "DOE", "  john@example.com  ");
console.log(person.firstName); // "John"
console.log(person.lastName);  // "Doe"
console.log(person.email);     // "john@example.com"
```

## 📋 Parameter Decorators

### 1. **Parameter Validation**

```typescript
import "reflect-metadata";

const VALIDATION_METADATA_KEY = Symbol("validation");

// Parameter validation decorators
function IsString(target: any, propertyKey: string | symbol, parameterIndex: number) {
    const existingValidators = Reflect.getMetadata(VALIDATION_METADATA_KEY, target, propertyKey) || [];
    existingValidators.push({
        index: parameterIndex,
        validator: (value: any) => typeof value === "string",
        message: "Parameter must be a string"
    });
    Reflect.defineMetadata(VALIDATION_METADATA_KEY, existingValidators, target, propertyKey);
}

function IsPositive(target: any, propertyKey: string | symbol, parameterIndex: number) {
    const existingValidators = Reflect.getMetadata(VALIDATION_METADATA_KEY, target, propertyKey) || [];
    existingValidators.push({
        index: parameterIndex,
        validator: (value: any) => typeof value === "number" && value > 0,
        message: "Parameter must be a positive number"
    });
    Reflect.defineMetadata(VALIDATION_METADATA_KEY, existingValidators, target, propertyKey);
}

// Method decorator that uses parameter metadata
function ValidateParameters(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    const validators = Reflect.getMetadata(VALIDATION_METADATA_KEY, target, propertyKey) || [];
    
    descriptor.value = function(...args: any[]) {
        for (const validator of validators) {
            const value = args[validator.index];
            if (!validator.validator(value)) {
                throw new Error(`${validator.message} at parameter ${validator.index}`);
            }
        }
        
        return originalMethod.apply(this, args);
    };
}

class OrderService {
    @ValidateParameters
    createOrder(
        @IsString productName: string,
        @IsPositive quantity: number,
        @IsPositive price: number
    ): void {
        console.log(`Creating order: ${productName} x ${quantity} @ $${price}`);
    }
}

const orderService = new OrderService();
orderService.createOrder("Laptop", 2, 999.99); // ✅ OK
// orderService.createOrder("", -1, 0); // ❌ Validation errors
```

## 🏭 Decorator Factories

### 1. **Configurable Decorators**

```typescript
// Cache decorator factory
function Cache(ttl: number = 60000) { // TTL in milliseconds
    const cache = new Map<string, { value: any; expiry: number }>();
    
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = function(...args: any[]) {
            const key = `${propertyKey}_${JSON.stringify(args)}`;
            const cached = cache.get(key);
            
            if (cached && cached.expiry > Date.now()) {
                console.log(`Cache hit for ${propertyKey}`);
                return cached.value;
            }
            
            const result = originalMethod.apply(this, args);
            cache.set(key, {
                value: result,
                expiry: Date.now() + ttl
            });
            
            console.log(`Cache miss for ${propertyKey}, cached for ${ttl}ms`);
            return result;
        };
    };
}

// Retry decorator factory
function Retry(maxAttempts: number = 3, delay: number = 1000) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = async function(...args: any[]) {
            let lastError: any;
            
            for (let attempt = 1; attempt <= maxAttempts; attempt++) {
                try {
                    return await originalMethod.apply(this, args);
                } catch (error) {
                    lastError = error;
                    console.log(`Attempt ${attempt} failed for ${propertyKey}: ${error.message}`);
                    
                    if (attempt < maxAttempts) {
                        await new Promise(resolve => setTimeout(resolve, delay));
                    }
                }
            }
            
            throw lastError;
        };
    };
}

class ApiService {
    @Cache(30000) // Cache for 30 seconds
    getUserData(userId: string): any {
        console.log(`Fetching user data for ${userId}`);
        return { id: userId, name: "John Doe", email: "john@example.com" };
    }
    
    @Retry(3, 500) // Retry 3 times with 500ms delay
    async saveData(data: any): Promise<void> {
        // Simulate unreliable operation
        if (Math.random() < 0.7) {
            throw new Error("Network error");
        }
        console.log("Data saved successfully");
    }
}
```

### 2. **Role-Based Access Control**

```typescript
// RBAC decorator factory
function RequireRole(...roles: string[]) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = function(...args: any[]) {
            // In real application, get user from context/session
            const currentUser = getCurrentUser(); // Assume this function exists
            
            if (!currentUser) {
                throw new Error("Authentication required");
            }
            
            const hasRequiredRole = roles.some(role => 
                currentUser.roles.includes(role)
            );
            
            if (!hasRequiredRole) {
                throw new Error(`Access denied. Required roles: ${roles.join(", ")}`);
            }
            
            return originalMethod.apply(this, args);
        };
    };
}

// Audit decorator factory
function Audit(action: string) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalMethod = descriptor.value;
        
        descriptor.value = function(...args: any[]) {
            const user = getCurrentUser();
            const timestamp = new Date().toISOString();
            
            console.log(`AUDIT: ${timestamp} - User ${user?.id} performed ${action}`);
            
            try {
                const result = originalMethod.apply(this, args);
                console.log(`AUDIT: ${action} completed successfully`);
                return result;
            } catch (error) {
                console.log(`AUDIT: ${action} failed: ${error.message}`);
                throw error;
            }
        };
    };
}

// Mock function for example
function getCurrentUser() {
    return { id: "user123", roles: ["admin", "user"] };
}

class AdminService {
    @RequireRole("admin")
    @Audit("DELETE_USER")
    deleteUser(userId: string): void {
        console.log(`Deleting user ${userId}`);
        // Delete user logic
    }
    
    @RequireRole("admin", "moderator")
    @Audit("BAN_USER")
    banUser(userId: string, reason: string): void {
        console.log(`Banning user ${userId} for: ${reason}`);
        // Ban user logic
    }
    
    @RequireRole("user")
    @Audit("VIEW_PROFILE")
    viewProfile(userId: string): any {
        console.log(`Viewing profile for ${userId}`);
        return { id: userId, name: "User Profile" };
    }
}
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **Web Framework Decorators**

```typescript
// Simple web framework using decorators
const routes: Array<{ method: string; path: string; handler: Function }> = [];

// Route decorators
function Get(path: string) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        routes.push({
            method: "GET",
            path,
            handler: descriptor.value
        });
    };
}

function Post(path: string) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        routes.push({
            method: "POST",
            path,
            handler: descriptor.value
        });
    };
}

// Middleware decorator
function UseMiddleware(middleware: Function) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        const originalHandler = descriptor.value;
        
        descriptor.value = function(req: any, res: any) {
            middleware(req, res, () => {
                return originalHandler.call(this, req, res);
            });
        };
    };
}

// Authentication middleware
function authMiddleware(req: any, res: any, next: Function) {
    if (req.headers.authorization) {
        next();
    } else {
        res.status = 401;
        res.body = "Unauthorized";
    }
}

// Controller
class UserController {
    @Get("/users")
    getUsers(req: any, res: any) {
        res.body = [{ id: 1, name: "John" }, { id: 2, name: "Jane" }];
    }
    
    @Get("/users/:id")
    getUser(req: any, res: any) {
        res.body = { id: req.params.id, name: "John Doe" };
    }
    
    @Post("/users")
    @UseMiddleware(authMiddleware)
    createUser(req: any, res: any) {
        res.body = { id: 3, ...req.body };
    }
}

// Start server (pseudo-code)
function startServer() {
    console.log("Registered routes:");
    routes.forEach(route => {
        console.log(`${route.method} ${route.path}`);
    });
}
```

### 2. **Database ORM Decorators**

```typescript
// ORM decorators
const entityMetadata = new Map<Function, any>();
const columnMetadata = new Map<Function, Map<string, any>>();

function Entity(tableName: string) {
    return function(target: Function) {
        entityMetadata.set(target, { tableName });
    };
}

function Column(options: { type?: string; nullable?: boolean; primaryKey?: boolean } = {}) {
    return function(target: any, propertyKey: string) {
        if (!columnMetadata.has(target.constructor)) {
            columnMetadata.set(target.constructor, new Map());
        }
        
        const columns = columnMetadata.get(target.constructor)!;
        columns.set(propertyKey, {
            type: options.type || "TEXT",
            nullable: options.nullable || false,
            primaryKey: options.primaryKey || false
        });
    };
}

function PrimaryKey(target: any, propertyKey: string) {
    Column({ primaryKey: true })(target, propertyKey);
}

// Repository base class
abstract class Repository<T> {
    constructor(private entityClass: new() => T) {}
    
    generateCreateTableSQL(): string {
        const entity = entityMetadata.get(this.entityClass);
        const columns = columnMetadata.get(this.entityClass);
        
        if (!entity || !columns) {
            throw new Error("Entity metadata not found");
        }
        
        const columnDefinitions: string[] = [];
        columns.forEach((columnDef, columnName) => {
            let def = `${columnName} ${columnDef.type}`;
            if (columnDef.primaryKey) def += " PRIMARY KEY";
            if (!columnDef.nullable) def += " NOT NULL";
            columnDefinitions.push(def);
        });
        
        return `CREATE TABLE ${entity.tableName} (${columnDefinitions.join(", ")})`;
    }
}

// Usage
@Entity("users")
class User {
    @PrimaryKey
    id: string;
    
    @Column({ type: "VARCHAR(255)" })
    name: string;
    
    @Column({ type: "VARCHAR(255)" })
    email: string;
    
    @Column({ type: "TIMESTAMP", nullable: true })
    lastLogin: Date;
}

class UserRepository extends Repository<User> {
    constructor() {
        super(User);
    }
}

const userRepo = new UserRepository();
console.log(userRepo.generateCreateTableSQL());
// CREATE TABLE users (id TEXT PRIMARY KEY NOT NULL, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL, lastLogin TIMESTAMP)
```

## 📝 แบบฝึกหัด

### 1. **Validation Framework**
สร้างระบบ validation ด้วย decorators:
- Property validation decorators
- Method parameter validation
- Custom validation rules
- Error message customization

### 2. **Event-Driven Architecture**
ออกแบบระบบ event handling:
- Event listener decorators
- Event publisher decorators
- Async event processing
- Event middleware

### 3. **API Documentation Generator**
สร้างระบบสำหรับ generate API docs:
- Route documentation decorators
- Parameter description decorators
- Response type decorators
- OpenAPI/Swagger integration

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 11: Advanced Types](./11-advanced-types.md)

**บทต่อไป**: [บทที่ 13: Utility Types](./13-utility-types.md)

**กลับไปหน้าหลัก**: [README](./README.md)
