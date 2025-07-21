# บทที่ 11: Advanced Types

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจ Advanced Type System ใน TypeScript
- เรียนรู้ Conditional Types และ Template Literal Types
- เข้าใจ Mapped Types และ Type Manipulation
- เรียนรู้ Brand Types และ Phantom Types

## 🔀 Conditional Types

### 1. **Basic Conditional Types**

```typescript
// Basic conditional type syntax
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;  // true
type Test2 = IsString<number>;  // false
type Test3 = IsString<"hello">; // true

// Conditional types กับ union types
type Filter<T, U> = T extends U ? T : never;

type StringsOnly = Filter<string | number | boolean, string>; // string
type NumbersOnly = Filter<string | number | boolean, number>; // number

// Conditional types ใน function return types
function process<T>(value: T): T extends string ? string[] : T extends number ? number[] : never {
    if (typeof value === "string") {
        return value.split("") as any;
    } else if (typeof value === "number") {
        return [value] as any;
    }
    throw new Error("Unsupported type");
}

const stringResult = process("hello");  // string[]
const numberResult = process(42);       // number[]
```

### 2. **Infer Keyword**

```typescript
// Extracting return types
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getString(): string { return "hello"; }
function getNumber(): number { return 42; }

type StringReturn = ReturnType<typeof getString>; // string
type NumberReturn = ReturnType<typeof getNumber>; // number

// Extracting parameter types
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function exampleFunc(a: string, b: number, c: boolean): void {}
type ExampleParams = Parameters<typeof exampleFunc>; // [string, number, boolean]

// Extracting array element types
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type StringArrayElement = ArrayElement<string[]>; // string
type NumberArrayElement = ArrayElement<number[]>; // number

// Complex inference patterns
type PromiseValue<T> = T extends Promise<infer U> ? U : T;

type Value1 = PromiseValue<Promise<string>>; // string
type Value2 = PromiseValue<number>;          // number

// Nested inference
type DeepArrayElement<T> = T extends (infer U)[][]
    ? U
    : T extends (infer U)[]
    ? U
    : T;

type Deep1 = DeepArrayElement<string[][]>; // string
type Deep2 = DeepArrayElement<number[]>;   // number
type Deep3 = DeepArrayElement<boolean>;    // boolean
```

### 3. **Distributive Conditional Types**

```typescript
// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;

type DistributedArray = ToArray<string | number>; // string[] | number[]

// Non-distributive (using tuple)
type ToArrayNonDistributive<T> = [T] extends [any] ? T[] : never;

type NonDistributedArray = ToArrayNonDistributive<string | number>; // (string | number)[]

// Practical example: Exclude
type MyExclude<T, U> = T extends U ? never : T;

type WithoutStrings = MyExclude<string | number | boolean, string>; // number | boolean

// Extract
type MyExtract<T, U> = T extends U ? T : never;

type OnlyStrings = MyExtract<string | number | boolean, string>; // string
```

## 📝 Template Literal Types

### 1. **Basic Template Literals**

```typescript
// Template literal types
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

// With unions
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"

// Combining multiple unions
type Lang = "en" | "ja" | "pt";
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
// "en_welcome_email_id" | "en_email_heading_id" | ... (12 combinations)
```

### 2. **Template Literal Patterns**

```typescript
// Event naming patterns
type EventNames<T extends string> = `on${Capitalize<T>}`;

type ButtonEvents = EventNames<"click" | "hover" | "focus">;
// "onClick" | "onHover" | "onFocus"

// API endpoint patterns
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = "/users" | "/products" | "/orders";

type APIRoute = `${HTTPMethod} ${Endpoint}`;
// "GET /users" | "POST /users" | ... (12 combinations)

// CSS class patterns
type Size = "sm" | "md" | "lg";
type Color = "red" | "blue" | "green";

type ButtonClass = `btn-${Size}-${Color}`;
// "btn-sm-red" | "btn-sm-blue" | ... (9 combinations)

// Path patterns
type Path<T extends string> = T extends `${infer Start}/${infer Rest}`
    ? Start | Path<Rest>
    : T;

type URLParts = Path<"api/users/123/posts">; // "api" | "users" | "123" | "posts"
```

### 3. **String Manipulation Types**

```typescript
// Built-in string manipulation utilities
type UppercaseGreeting = Uppercase<"hello world">; // "HELLO WORLD"
type LowercaseGreeting = Lowercase<"HELLO WORLD">; // "hello world"
type CapitalizedGreeting = Capitalize<"hello world">; // "Hello world"
type UncapitalizedGreeting = Uncapitalize<"Hello World">; // "hello World"

// Custom string manipulation
type CamelCase<S extends string> = S extends `${infer P1}_${infer P2}${infer P3}`
    ? `${P1}${Uppercase<P2>}${CamelCase<P3>}`
    : S;

type CamelCased = CamelCase<"hello_world_from_typescript">; // "helloWorldFromTypescript"

// Snake case to camel case
type SnakeToCamel<S extends string> =
    S extends `${infer T}_${infer U}` 
        ? `${T}${Capitalize<SnakeToCamel<U>>}` 
        : S;

type ApiResponseKey = SnakeToCamel<"user_profile_data">; // "userProfileData"

// Remove prefix
type RemovePrefix<S extends string, P extends string> = 
    S extends `${P}${infer Rest}` ? Rest : S;

type WithoutAPI = RemovePrefix<"api_get_users", "api_">; // "get_users"
```

## 🗺️ Advanced Mapped Types

### 1. **Key Remapping**

```typescript
// Key remapping with template literals
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
    email: string;
}

type PersonGetters = Getters<Person>;
// {
//     getName: () => string;
//     getAge: () => number;
//     getEmail: () => string;
// }

// Setters
type Setters<T> = {
    [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

type PersonSetters = Setters<Person>;
// {
//     setName: (value: string) => void;
//     setAge: (value: number) => void;
//     setEmail: (value: string) => void;
// }

// Combined getters and setters
type GettersAndSetters<T> = Getters<T> & Setters<T>;

type PersonAccessors = GettersAndSetters<Person>;
```

### 2. **Conditional Mapping**

```typescript
// Remove readonly modifier conditionally
type Mutable<T> = {
    -readonly [K in keyof T]: T[K];
};

interface ReadonlyPerson {
    readonly id: string;
    readonly name: string;
    age: number; // not readonly
}

type MutablePerson = Mutable<ReadonlyPerson>;
// {
//     id: string;
//     name: string;
//     age: number;
// }

// Filter properties by type
type FilterByType<T, U> = {
    [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Mixed {
    name: string;
    age: number;
    isActive: boolean;
    tags: string[];
    count: number;
}

type StringProps = FilterByType<Mixed, string>; // { name: string; }
type NumberProps = FilterByType<Mixed, number>; // { age: number; count: number; }

// Pick nullable properties
type NullableKeys<T> = {
    [K in keyof T]: null extends T[K] ? K : never;
}[keyof T];

type RequiredKeys<T> = {
    [K in keyof T]: null extends T[K] ? never : K;
}[keyof T];

interface UserProfile {
    id: string;
    name: string;
    email: string | null;
    avatar: string | null;
    lastLogin: Date;
}

type NullableUserKeys = NullableKeys<UserProfile>; // "email" | "avatar"
type RequiredUserKeys = RequiredKeys<UserProfile>; // "id" | "name" | "lastLogin"
```

### 3. **Deep Mapping**

```typescript
// Deep readonly
type DeepReadonly<T> = {
    readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

interface NestedObject {
    level1: {
        level2: {
            value: string;
            array: number[];
        };
        otherValue: boolean;
    };
    topLevel: string;
}

type DeepReadonlyNested = DeepReadonly<NestedObject>;
// All properties and nested properties become readonly

// Deep partial
type DeepPartial<T> = {
    [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

type PartialNested = DeepPartial<NestedObject>;
// All properties and nested properties become optional

// Deep required
type DeepRequired<T> = {
    [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};

interface PartialUser {
    id?: string;
    profile?: {
        name?: string;
        settings?: {
            theme?: string;
            notifications?: boolean;
        };
    };
}

type RequiredUser = DeepRequired<PartialUser>;
// All optional properties become required
```

## 🏷️ Brand Types และ Phantom Types

### 1. **Brand Types**

```typescript
// Brand types for type safety
type Brand<T, B> = T & { __brand: B };

type UserId = Brand<string, "UserId">;
type ProductId = Brand<string, "ProductId">;
type Email = Brand<string, "Email">;

// Helper functions
function createUserId(id: string): UserId {
    return id as UserId;
}

function createProductId(id: string): ProductId {
    return id as ProductId;
}

function createEmail(email: string): Email {
    if (!email.includes("@")) {
        throw new Error("Invalid email");
    }
    return email as Email;
}

// Type safety in action
function getUser(id: UserId): Promise<User> {
    // implementation
    return Promise.resolve({} as User);
}

function getProduct(id: ProductId): Promise<Product> {
    // implementation
    return Promise.resolve({} as Product);
}

const userId = createUserId("user-123");
const productId = createProductId("product-456");

getUser(userId);    // ✅ OK
getProduct(productId); // ✅ OK

// getUser(productId); // ❌ Error - ProductId is not assignable to UserId
// getProduct(userId); // ❌ Error - UserId is not assignable to ProductId
```

### 2. **Phantom Types**

```typescript
// Phantom types for compile-time validation
interface Validated<T> {
    readonly _validation: unique symbol;
    readonly value: T;
}

type ValidatedEmail = Validated<string>;
type ValidatedPhoneNumber = Validated<string>;

function validateEmail(email: string): ValidatedEmail | null {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (emailRegex.test(email)) {
        return { value: email } as ValidatedEmail;
    }
    return null;
}

function validatePhoneNumber(phone: string): ValidatedPhoneNumber | null {
    const phoneRegex = /^\+?\d{10,15}$/;
    if (phoneRegex.test(phone)) {
        return { value: phone } as ValidatedPhoneNumber;
    }
    return null;
}

function sendEmail(to: ValidatedEmail, subject: string, body: string): void {
    console.log(`Sending email to ${to.value}: ${subject}`);
}

function sendSMS(to: ValidatedPhoneNumber, message: string): void {
    console.log(`Sending SMS to ${to.value}: ${message}`);
}

// Usage
const validEmail = validateEmail("user@example.com");
const validPhone = validatePhoneNumber("+1234567890");

if (validEmail) {
    sendEmail(validEmail, "Hello", "World"); // ✅ OK
}

if (validPhone) {
    sendSMS(validPhone, "Hello World"); // ✅ OK
}

// sendEmail(validPhone, "Hello", "World"); // ❌ Error - wrong type
```

### 3. **Units of Measure**

```typescript
// Units of measure with phantom types
type Unit<T, U> = T & { __unit: U };

type Meters = Unit<number, "meters">;
type Feet = Unit<number, "feet">;
type Seconds = Unit<number, "seconds">;
type MetersPerSecond = Unit<number, "meters/second">;

function meters(value: number): Meters {
    return value as Meters;
}

function feet(value: number): Feet {
    return value as Feet;
}

function seconds(value: number): Seconds {
    return value as Seconds;
}

// Conversion functions
function feetToMeters(ft: Feet): Meters {
    return meters(ft * 0.3048);
}

function calculateSpeed(distance: Meters, time: Seconds): MetersPerSecond {
    return (distance / time) as MetersPerSecond;
}

// Usage
const distance = meters(100);
const time = seconds(10);
const speed = calculateSpeed(distance, time);

console.log(`Speed: ${speed} m/s`);

// Type safety
const distanceInFeet = feet(328);
// calculateSpeed(distanceInFeet, time); // ❌ Error - must convert first
calculateSpeed(feetToMeters(distanceInFeet), time); // ✅ OK
```

## 🔍 Type Guards และ Assertion Functions

### 1. **Advanced Type Guards**

```typescript
// Generic type guard
function isOfType<T>(
    value: unknown,
    validator: (val: any) => val is T
): value is T {
    return validator(value);
}

// Specific type guards
function isString(value: unknown): value is string {
    return typeof value === "string";
}

function isNumber(value: unknown): value is number {
    return typeof value === "number";
}

function isUser(value: unknown): value is User {
    return (
        typeof value === "object" &&
        value !== null &&
        "id" in value &&
        "name" in value &&
        "email" in value
    );
}

// Compound type guards
function isStringArray(value: unknown): value is string[] {
    return Array.isArray(value) && value.every(isString);
}

function isNumberArray(value: unknown): value is number[] {
    return Array.isArray(value) && value.every(isNumber);
}
```

### 2. **Assertion Functions**

```typescript
// Basic assertion function
function assertIsString(value: unknown): asserts value is string {
    if (typeof value !== "string") {
        throw new Error("Expected string");
    }
}

function assertIsNumber(value: unknown): asserts value is number {
    if (typeof value !== "number") {
        throw new Error("Expected number");
    }
}

// Generic assertion function
function assert<T>(
    condition: unknown,
    message: string
): asserts condition is T {
    if (!condition) {
        throw new Error(message);
    }
}

// Assertion with property checking
function assertHasProperty<T, K extends PropertyKey>(
    obj: T,
    prop: K,
    message?: string
): asserts obj is T & Record<K, unknown> {
    if (!(prop in (obj as any))) {
        throw new Error(message || `Property ${String(prop)} is missing`);
    }
}

// Usage
function processUserData(data: unknown): void {
    assertIsString(data); // Now TypeScript knows data is string
    console.log(data.toUpperCase());
    
    // Or with generic assert
    assert<string>(typeof data === "string", "Data must be string");
    console.log(data.toLowerCase());
}
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **Type-Safe Event System**

```typescript
// Event map definition
interface EventMap {
    "user:created": { id: string; name: string; email: string };
    "user:updated": { id: string; changes: Partial<User> };
    "user:deleted": { id: string };
    "order:placed": { orderId: string; userId: string; total: number };
    "order:shipped": { orderId: string; trackingNumber: string };
}

// Event listener type
type EventListener<T> = (data: T) => void | Promise<void>;

// Event emitter class
class TypedEventEmitter {
    private listeners: {
        [K in keyof EventMap]?: EventListener<EventMap[K]>[];
    } = {};
    
    on<K extends keyof EventMap>(
        event: K,
        listener: EventListener<EventMap[K]>
    ): void {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        this.listeners[event]!.push(listener);
    }
    
    emit<K extends keyof EventMap>(
        event: K,
        data: EventMap[K]
    ): void {
        const eventListeners = this.listeners[event];
        if (eventListeners) {
            eventListeners.forEach(listener => {
                try {
                    listener(data);
                } catch (error) {
                    console.error(`Error in ${event} listener:`, error);
                }
            });
        }
    }
    
    off<K extends keyof EventMap>(
        event: K,
        listener: EventListener<EventMap[K]>
    ): void {
        const eventListeners = this.listeners[event];
        if (eventListeners) {
            const index = eventListeners.indexOf(listener);
            if (index > -1) {
                eventListeners.splice(index, 1);
            }
        }
    }
}

// Usage
const emitter = new TypedEventEmitter();

emitter.on("user:created", (data) => {
    console.log(`New user: ${data.name} (${data.email})`);
});

emitter.on("order:placed", (data) => {
    console.log(`Order ${data.orderId} placed by user ${data.userId}`);
});

emitter.emit("user:created", {
    id: "123",
    name: "John Doe",
    email: "john@example.com"
});
```

### 2. **Type-Safe API Client**

```typescript
// API endpoint definitions
interface APIEndpoints {
    "GET /users": {
        response: User[];
        query?: { page?: number; limit?: number };
    };
    "GET /users/:id": {
        response: User;
        params: { id: string };
    };
    "POST /users": {
        response: User;
        body: Omit<User, "id">;
    };
    "PUT /users/:id": {
        response: User;
        params: { id: string };
        body: Partial<Omit<User, "id">>;
    };
    "DELETE /users/:id": {
        response: void;
        params: { id: string };
    };
}

// Extract method and path
type Method = "GET" | "POST" | "PUT" | "DELETE";
type ExtractMethod<T> = T extends `${infer M} ${string}` ? M : never;
type ExtractPath<T> = T extends `${string} ${infer P}` ? P : never;

// API client class
class TypedAPIClient {
    constructor(private baseURL: string) {}
    
    async request<K extends keyof APIEndpoints>(
        endpoint: K,
        options?: {
            params?: APIEndpoints[K] extends { params: infer P } ? P : never;
            query?: APIEndpoints[K] extends { query: infer Q } ? Q : never;
            body?: APIEndpoints[K] extends { body: infer B } ? B : never;
        }
    ): Promise<APIEndpoints[K]["response"]> {
        const method = this.extractMethod(endpoint);
        let path = this.extractPath(endpoint);
        
        // Replace path parameters
        if (options?.params) {
            Object.entries(options.params).forEach(([key, value]) => {
                path = path.replace(`:${key}`, String(value));
            });
        }
        
        // Add query parameters
        if (options?.query) {
            const queryString = new URLSearchParams(
                Object.entries(options.query)
                    .filter(([_, value]) => value !== undefined)
                    .map(([key, value]) => [key, String(value)])
            ).toString();
            if (queryString) {
                path += `?${queryString}`;
            }
        }
        
        const response = await fetch(`${this.baseURL}${path}`, {
            method,
            headers: {
                "Content-Type": "application/json",
            },
            body: options?.body ? JSON.stringify(options.body) : undefined,
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return method === "DELETE" ? undefined : response.json();
    }
    
    private extractMethod(endpoint: string): Method {
        return endpoint.split(" ")[0] as Method;
    }
    
    private extractPath(endpoint: string): string {
        return endpoint.split(" ")[1];
    }
}

// Usage
const api = new TypedAPIClient("https://api.example.com");

// Type-safe API calls
const users = await api.request("GET /users", {
    query: { page: 1, limit: 10 }
});

const user = await api.request("GET /users/:id", {
    params: { id: "123" }
});

const newUser = await api.request("POST /users", {
    body: {
        name: "John Doe",
        email: "john@example.com"
    }
});
```

## 📝 แบบฝึกหัด

### 1. **Advanced Form Validation**
สร้างระบบ validation ที่ใช้ conditional types:
- Field validation rules
- Error message generation
- Type-safe form state management

### 2. **Database Query Builder**
ออกแบบ type-safe query builder:
- SELECT, WHERE, JOIN clauses
- Type inference จาก schema
- Compile-time query validation

### 3. **Configuration System**
สร้างระบบ configuration ที่ใช้ template literal types:
- Environment-specific configs
- Type-safe config merging
- Path-based config access

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 10: Modules และ Namespaces](./10-modules-namespaces.md)

**บทต่อไป**: [บทที่ 12: Decorators](./12-decorators.md)

**กลับไปหน้าหลัก**: [README](./README.md)
