# บทที่ 13: Utility Types

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจและใช้งาน Built-in Utility Types
- เรียนรู้ Conditional Types และ Mapped Types
- สร้าง Custom Utility Types
- เรียนรู้ Type Manipulation Techniques

## 🔧 Built-in Utility Types

### 1. **Partial<T>**

```typescript
// Partial<T> ทำให้ properties ทั้งหมดใน T เป็น optional

interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

// สร้าง type ที่ทุก property เป็น optional
type PartialUser = Partial<User>;
// เหมือนกับ:
// {
//     id?: number;
//     name?: string;
//     email?: string;
//     age?: number;
// }

// ใช้งานจริง
function updateUser(id: number, updates: Partial<User>): User {
    const existingUser = getUserById(id);
    return { ...existingUser, ...updates };
}

// สามารถส่งเฉพาะ property ที่ต้องการ update
updateUser(1, { name: "John Updated" });
updateUser(2, { email: "new@email.com", age: 25 });

// Example: Form state management
interface FormData {
    username: string;
    password: string;
    confirmPassword: string;
    email: string;
}

class FormManager {
    private data: Partial<FormData> = {};
    
    updateField<K extends keyof FormData>(
        field: K, 
        value: FormData[K]
    ): void {
        this.data[field] = value;
    }
    
    isValid(): boolean {
        return !!(this.data.username && 
                 this.data.password && 
                 this.data.email &&
                 this.data.password === this.data.confirmPassword);
    }
    
    getData(): FormData | null {
        if (this.isValid()) {
            return this.data as FormData;
        }
        return null;
    }
}
```

### 2. **Required<T>**

```typescript
// Required<T> ทำให้ properties ทั้งหมดใน T เป็น required

interface Config {
    host?: string;
    port?: number;
    ssl?: boolean;
    timeout?: number;
}

// ทำให้ทุก property เป็น required
type RequiredConfig = Required<Config>;
// เหมือนกับ:
// {
//     host: string;
//     port: number;
//     ssl: boolean;
//     timeout: number;
// }

// ใช้งานจริง
const defaultConfig: Config = {
    host: "localhost",
    port: 3000
};

function createServer(config: Config): void {
    // Merge กับ default แล้วใช้ Required เพื่อให้แน่ใจว่าครบทุก property
    const fullConfig: Required<Config> = {
        host: "localhost",
        port: 3000,
        ssl: false,
        timeout: 5000,
        ...config
    };
    
    console.log(`Starting server on ${fullConfig.host}:${fullConfig.port}`);
    console.log(`SSL: ${fullConfig.ssl}, Timeout: ${fullConfig.timeout}ms`);
}

// Database connection example
interface DatabaseOptions {
    host?: string;
    port?: number;
    database?: string;
    username?: string;
    password?: string;
    ssl?: boolean;
}

class DatabaseConnection {
    private config: Required<DatabaseOptions>;
    
    constructor(options: DatabaseOptions) {
        this.config = {
            host: "localhost",
            port: 5432,
            database: "myapp",
            username: "user",
            password: "",
            ssl: false,
            ...options
        };
    }
    
    connect(): void {
        console.log(`Connecting to ${this.config.host}:${this.config.port}/${this.config.database}`);
    }
}
```

### 3. **Readonly<T>**

```typescript
// Readonly<T> ทำให้ properties ทั้งหมดใน T เป็น readonly

interface Product {
    id: number;
    name: string;
    price: number;
    category: string;
}

type ReadonlyProduct = Readonly<Product>;
// เหมือนกับ:
// {
//     readonly id: number;
//     readonly name: string;
//     readonly price: number;
//     readonly category: string;
// }

// ใช้งานจริง
function processProduct(product: Readonly<Product>): void {
    console.log(`Processing ${product.name}`);
    // product.price = 100; // ❌ Error: Cannot assign to 'price' because it is a read-only property
}

// Immutable state management
class Store<T> {
    private state: Readonly<T>;
    
    constructor(initialState: T) {
        this.state = Object.freeze({ ...initialState });
    }
    
    getState(): Readonly<T> {
        return this.state;
    }
    
    setState(updater: (state: Readonly<T>) => T): void {
        this.state = Object.freeze(updater(this.state));
    }
}

interface AppState {
    user: { name: string; email: string } | null;
    products: Product[];
    cart: { productId: number; quantity: number }[];
}

const store = new Store<AppState>({
    user: null,
    products: [],
    cart: []
});

// การใช้งาน
store.setState(state => ({
    ...state,
    user: { name: "John", email: "john@example.com" }
}));

// Deep readonly
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface NestedConfig {
    database: {
        host: string;
        credentials: {
            username: string;
            password: string;
        };
    };
    features: string[];
}

type DeepReadonlyConfig = DeepReadonly<NestedConfig>;
// ทุก property รวมถึง nested objects จะเป็น readonly
```

### 4. **Pick<T, K>**

```typescript
// Pick<T, K> เลือกเฉพาะ properties ที่ระบุใน K จาก T

interface Employee {
    id: number;
    name: string;
    email: string;
    salary: number;
    department: string;
    startDate: Date;
    benefits: string[];
}

// เลือกเฉพาะข้อมูลที่ใช้แสดงสาธารณะ
type PublicEmployee = Pick<Employee, "id" | "name" | "department">;
// เหมือนกับ:
// {
//     id: number;
//     name: string;
//     department: string;
// }

// เลือกเฉพาะข้อมูลสำหรับ contact
type EmployeeContact = Pick<Employee, "name" | "email">;

// ใช้งานจริง
function getPublicEmployeeInfo(employee: Employee): PublicEmployee {
    return {
        id: employee.id,
        name: employee.name,
        department: employee.department
    };
}

// API Response types
interface APIUser {
    id: string;
    username: string;
    email: string;
    firstName: string;
    lastName: string;
    avatar: string;
    createdAt: string;
    updatedAt: string;
    lastLogin: string;
    permissions: string[];
}

// เลือกเฉพาะข้อมูลสำหรับ login response
type LoginResponse = Pick<APIUser, "id" | "username" | "avatar" | "permissions">;

// เลือกเฉพาะข้อมูลสำหรับ profile display
type UserProfile = Pick<APIUser, "username" | "firstName" | "lastName" | "avatar">;

// เลือกเฉพาะข้อมูลที่อัพเดทได้
type UpdatableUserFields = Pick<APIUser, "firstName" | "lastName" | "avatar">;

// Generic function สำหรับ API
function apiCall<T, K extends keyof T>(
    endpoint: string,
    fields: K[]
): Promise<Pick<T, K>> {
    // Implementation จะ return เฉพาะ fields ที่ต้องการ
    return fetch(`/api${endpoint}?fields=${fields.join(",")}`)
        .then(res => res.json());
}

// ใช้งาน
const userProfile = await apiCall<APIUser, "username" | "firstName" | "lastName">(
    "/users/123", 
    ["username", "firstName", "lastName"]
);
```

### 5. **Omit<T, K>**

```typescript
// Omit<T, K> เอา properties ที่ระบุใน K ออกจาก T

interface Product {
    id: number;
    name: string;
    price: number;
    description: string;
    inStock: boolean;
    createdAt: Date;
    updatedAt: Date;
}

// สร้าง type สำหรับการสร้าง product ใหม่ (ไม่ต้องการ id, timestamps)
type CreateProductRequest = Omit<Product, "id" | "createdAt" | "updatedAt">;
// เหมือนกับ:
// {
//     name: string;
//     price: number;
//     description: string;
//     inStock: boolean;
// }

// สร้าง type สำหรับ update (ไม่ต้องการ id, timestamps และทำให้เป็น partial)
type UpdateProductRequest = Partial<Omit<Product, "id" | "createdAt" | "updatedAt">>;

// ใช้งานจริง
class ProductService {
    async createProduct(productData: CreateProductRequest): Promise<Product> {
        const newProduct: Product = {
            ...productData,
            id: Date.now(), // Generate ID
            createdAt: new Date(),
            updatedAt: new Date()
        };
        
        // Save to database
        return newProduct;
    }
    
    async updateProduct(
        id: number, 
        updates: UpdateProductRequest
    ): Promise<Product> {
        const existingProduct = await this.getProductById(id);
        
        const updatedProduct: Product = {
            ...existingProduct,
            ...updates,
            updatedAt: new Date()
        };
        
        // Save to database
        return updatedProduct;
    }
    
    private async getProductById(id: number): Promise<Product> {
        // Implementation
        throw new Error("Not implemented");
    }
}

// Form handling example
interface UserRegistration {
    username: string;
    email: string;
    password: string;
    confirmPassword: string;
    acceptTerms: boolean;
    marketingEmails: boolean;
}

// Type สำหรับส่งไปยัง server (ไม่ต้องส่ง confirmPassword, acceptTerms)
type RegistrationPayload = Omit<UserRegistration, "confirmPassword" | "acceptTerms">;

function submitRegistration(formData: UserRegistration): RegistrationPayload {
    if (formData.password !== formData.confirmPassword) {
        throw new Error("Passwords do not match");
    }
    
    if (!formData.acceptTerms) {
        throw new Error("You must accept the terms and conditions");
    }
    
    // Return only the data needed for the API
    return {
        username: formData.username,
        email: formData.email,
        password: formData.password,
        marketingEmails: formData.marketingEmails
    };
}
```

### 6. **Record<K, T>**

```typescript
// Record<K, T> สร้าง object type ที่ key เป็น K และ value เป็น T

// Basic usage
type Theme = "light" | "dark" | "auto";
type ThemeConfig = Record<Theme, { background: string; text: string }>;

const themes: ThemeConfig = {
    light: { background: "#ffffff", text: "#000000" },
    dark: { background: "#000000", text: "#ffffff" },
    auto: { background: "inherit", text: "inherit" }
};

// HTTP Status codes
type HttpStatusCode = 200 | 400 | 401 | 403 | 404 | 500;
type StatusMessage = Record<HttpStatusCode, string>;

const statusMessages: StatusMessage = {
    200: "OK",
    400: "Bad Request",
    401: "Unauthorized", 
    403: "Forbidden",
    404: "Not Found",
    500: "Internal Server Error"
};

// Generic cache implementation
class Cache<T> {
    private storage: Record<string, { data: T; expiry: number }> = {};
    
    set(key: string, value: T, ttl: number = 60000): void {
        this.storage[key] = {
            data: value,
            expiry: Date.now() + ttl
        };
    }
    
    get(key: string): T | null {
        const item = this.storage[key];
        
        if (!item) {
            return null;
        }
        
        if (item.expiry < Date.now()) {
            delete this.storage[key];
            return null;
        }
        
        return item.data;
    }
    
    clear(): void {
        this.storage = {};
    }
}

// Permission system
type Permission = "read" | "write" | "delete" | "admin";
type Resource = "posts" | "users" | "comments" | "settings";

type PermissionMatrix = Record<Resource, Record<Permission, boolean>>;

const userPermissions: PermissionMatrix = {
    posts: { read: true, write: true, delete: false, admin: false },
    users: { read: true, write: false, delete: false, admin: false },
    comments: { read: true, write: true, delete: true, admin: false },
    settings: { read: false, write: false, delete: false, admin: false }
};

function hasPermission(
    resource: Resource, 
    permission: Permission, 
    permissions: PermissionMatrix
): boolean {
    return permissions[resource][permission];
}

// Configuration system
type Environment = "development" | "staging" | "production";

interface DatabaseConfig {
    host: string;
    port: number;
    database: string;
}

type EnvironmentConfig = Record<Environment, DatabaseConfig>;

const dbConfigs: EnvironmentConfig = {
    development: {
        host: "localhost",
        port: 5432,
        database: "myapp_dev"
    },
    staging: {
        host: "staging.example.com",
        port: 5432,
        database: "myapp_staging"
    },
    production: {
        host: "prod.example.com",
        port: 5432,
        database: "myapp_prod"
    }
};
```

### 7. **Exclude<T, U> และ Extract<T, U>**

```typescript
// Exclude<T, U> เอา types ที่อยู่ใน U ออกจาก T
// Extract<T, U> เก็บเฉพาะ types ที่อยู่ทั้งใน T และ U

type AllColors = "red" | "blue" | "green" | "yellow" | "purple" | "orange";
type PrimaryColors = "red" | "blue" | "yellow";

// เอาสีหลักออก เหลือแต่สีรอง
type SecondaryColors = Exclude<AllColors, PrimaryColors>;
// "green" | "purple" | "orange"

// เก็บเฉพาะสีหลัก
type ExtractedPrimary = Extract<AllColors, PrimaryColors>;
// "red" | "blue" | "yellow"

// ใช้งานจริง
type Shape = "circle" | "square" | "triangle" | "rectangle";
type PolygonShapes = Exclude<Shape, "circle">; // "square" | "triangle" | "rectangle"

function calculateArea(shape: PolygonShapes, ...dimensions: number[]): number {
    // ใช้งานได้เฉพาะ polygon shapes
    switch (shape) {
        case "square":
            return dimensions[0] * dimensions[0];
        case "triangle":
            return (dimensions[0] * dimensions[1]) / 2;
        case "rectangle":
            return dimensions[0] * dimensions[1];
    }
}

// Event system
type MouseEvent = "click" | "mousedown" | "mouseup" | "mousemove";
type KeyboardEvent = "keydown" | "keyup" | "keypress";
type AllEvents = MouseEvent | KeyboardEvent;

type NonKeyboardEvents = Exclude<AllEvents, KeyboardEvent>; // MouseEvent
type OnlyMouseEvents = Extract<AllEvents, MouseEvent>;      // MouseEvent

// API Response types
type APIResponse<T> = 
    | { status: "success"; data: T }
    | { status: "error"; error: string }
    | { status: "loading" };

type SuccessResponse<T> = Extract<APIResponse<T>, { status: "success" }>;
type ErrorResponse = Extract<APIResponse<any>, { status: "error" }>;
type NonLoadingResponse<T> = Exclude<APIResponse<T>, { status: "loading" }>;

function handleResponse<T>(response: APIResponse<T>): void {
    if (response.status === "success") {
        // TypeScript รู้ว่า response มี data property
        console.log(response.data);
    } else if (response.status === "error") {
        // TypeScript รู้ว่า response มี error property
        console.error(response.error);
    }
}
```

### 8. **NonNullable<T>**

```typescript
// NonNullable<T> เอา null และ undefined ออกจาก T

type MaybeString = string | null | undefined;
type DefinitelyString = NonNullable<MaybeString>; // string

// ใช้งานจริง
function processValue(value: string | null | undefined): string {
    if (value != null) {
        // ใน scope นี้ TypeScript รู้ว่า value เป็น NonNullable<typeof value>
        return value.toUpperCase();
    }
    throw new Error("Value cannot be null or undefined");
}

// Array filtering
function filterNonNull<T>(array: (T | null | undefined)[]): NonNullable<T>[] {
    return array.filter((item): item is NonNullable<T> => item != null);
}

const mixedArray = ["hello", null, "world", undefined, "test"];
const cleanArray = filterNonNull(mixedArray); // string[]

// Optional chaining with NonNullable
interface User {
    profile?: {
        name?: string;
        email?: string;
    } | null;
}

function getUserName(user: User): NonNullable<string> | null {
    return user.profile?.name ?? null;
}

// Safe property access
function getProperty<T, K extends keyof T>(
    obj: T, 
    key: K
): NonNullable<T[K]> | null {
    const value = obj[key];
    return value != null ? value as NonNullable<T[K]> : null;
}

// Database query results
interface QueryResult<T> {
    data: T | null;
    error: string | null;
}

function unwrapQueryResult<T>(result: QueryResult<T>): NonNullable<T> {
    if (result.error) {
        throw new Error(result.error);
    }
    
    if (result.data == null) {
        throw new Error("No data returned");
    }
    
    return result.data;
}
```

## 🎨 Conditional Types

### 1. **Basic Conditional Types**

```typescript
// T extends U ? X : Y

// ตรวจสอบว่า type เป็น array หรือไม่
type IsArray<T> = T extends (infer U)[] ? true : false;

type StringIsArray = IsArray<string>;   // false
type NumberArrayIsArray = IsArray<number[]>; // true

// Flatten array type
type Flatten<T> = T extends (infer U)[] ? U : T;

type FlatString = Flatten<string>;     // string
type FlatNumbers = Flatten<number[]>;  // number

// ใช้งานจริง
function flatten<T>(input: T): Flatten<T> {
    if (Array.isArray(input)) {
        return input[0] as Flatten<T>;
    }
    return input as Flatten<T>;
}

// Return type based on input
type ApiResponse<T> = T extends string 
    ? { message: T } 
    : T extends number 
    ? { count: T }
    : { data: T };

function createResponse<T>(input: T): ApiResponse<T> {
    if (typeof input === "string") {
        return { message: input } as ApiResponse<T>;
    } else if (typeof input === "number") {
        return { count: input } as ApiResponse<T>;
    } else {
        return { data: input } as ApiResponse<T>;
    }
}

const stringResponse = createResponse("hello"); // { message: string }
const numberResponse = createResponse(42);      // { count: number }
const objectResponse = createResponse({ id: 1 }); // { data: { id: number } }
```

### 2. **Distributive Conditional Types**

```typescript
// Conditional types distribute over union types

type ToArray<T> = T extends any ? T[] : never;

type StringOrNumber = string | number;
type ArrayUnion = ToArray<StringOrNumber>; // string[] | number[]

// Non-distributive version
type ToArrayNonDistributive<T> = [T] extends [any] ? T[] : never;
type ArrayNonDistributive = ToArrayNonDistributive<StringOrNumber>; // (string | number)[]

// Filter union types
type Filter<T, U> = T extends U ? T : never;

type Numbers = 1 | 2 | "a" | "b" | true | false;
type OnlyStrings = Filter<Numbers, string>; // "a" | "b"
type OnlyBooleans = Filter<Numbers, boolean>; // true | false

// Exclude specific types from union
type RemoveStrings<T> = T extends string ? never : T;
type WithoutStrings = RemoveStrings<string | number | boolean>; // number | boolean

// Function parameter extraction
type Parameters<T extends (...args: any) => any> = 
    T extends (...args: infer P) => any ? P : never;

type ReturnType<T extends (...args: any) => any> = 
    T extends (...args: any) => infer R ? R : any;

function exampleFunction(a: string, b: number, c: boolean): { result: string } {
    return { result: `${a}-${b}-${c}` };
}

type ExampleParams = Parameters<typeof exampleFunction>; // [string, number, boolean]
type ExampleReturn = ReturnType<typeof exampleFunction>; // { result: string }
```

### 3. **Advanced Conditional Types**

```typescript
// Deep property access
type Get<T, K extends string> = K extends keyof T 
    ? T[K]
    : K extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
        ? Get<T[Key], Rest>
        : never
    : never;

interface DeepObject {
    user: {
        profile: {
            name: string;
            age: number;
        };
        preferences: {
            theme: "light" | "dark";
        };
    };
}

type UserName = Get<DeepObject, "user.profile.name">; // string
type Theme = Get<DeepObject, "user.preferences.theme">; // "light" | "dark"

// Promise unwrapping
type Awaited<T> = T extends PromiseLike<infer U> ? Awaited<U> : T;

type PromiseString = Awaited<Promise<string>>; // string
type NestedPromise = Awaited<Promise<Promise<number>>>; // number

// Function composition type
type Compose<F extends (...args: any[]) => any, G extends (...args: any[]) => any> =
    G extends (...args: infer A) => infer B
        ? F extends (arg: B) => infer C
            ? (...args: A) => C
            : never
        : never;

const add = (x: number) => x + 1;
const toString = (x: number) => x.toString();

type ComposedType = Compose<typeof toString, typeof add>; // (x: number) => string

// Object key transformation
type Capitalize<S extends string> = S extends `${infer F}${infer R}` 
    ? `${Uppercase<F>}${R}` 
    : S;

type CapitalizeKeys<T> = {
    [K in keyof T as Capitalize<string & K>]: T[K];
};

interface LowercaseInterface {
    name: string;
    age: number;
    email: string;
}

type CapitalizedInterface = CapitalizeKeys<LowercaseInterface>;
// {
//     Name: string;
//     Age: number;
//     Email: string;
// }
```

## 🗺️ Mapped Types

### 1. **Basic Mapped Types**

```typescript
// Mapped types สร้าง type ใหม่โดยการ transform type เดิม

// Make all properties optional
type Optional<T> = {
    [K in keyof T]?: T[K];
};

// Make all properties readonly
type ReadOnly<T> = {
    readonly [K in keyof T]: T[K];
};

// Make all properties mutable (remove readonly)
type Mutable<T> = {
    -readonly [K in keyof T]: T[K];
};

// Make all properties required (remove optional)
type Required<T> = {
    [K in keyof T]-?: T[K];
};

interface Example {
    readonly id?: number;
    readonly name?: string;
    age: number;
}

type OptionalExample = Optional<Example>;   // ทุก property เป็น optional
type MutableExample = Mutable<Example>;     // ไม่มี readonly
type RequiredExample = Required<Example>;   // ไม่มี optional
```

### 2. **Key Mapping**

```typescript
// Transform property keys
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Setters<T> = {
    [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

interface User {
    name: string;
    age: number;
    email: string;
}

type UserGetters = Getters<User>;
// {
//     getName: () => string;
//     getAge: () => number;
//     getEmail: () => string;
// }

type UserSetters = Setters<User>;
// {
//     setName: (value: string) => void;
//     setAge: (value: number) => void;
//     setEmail: (value: string) => void;
// }

// Combine getters and setters
type AccessorPattern<T> = Getters<T> & Setters<T>;

// Event handlers
type EventHandlers<T> = {
    [K in keyof T as `on${Capitalize<string & K>}Change`]: (value: T[K]) => void;
};

type UserEventHandlers = EventHandlers<User>;
// {
//     onNameChange: (value: string) => void;
//     onAgeChange: (value: number) => void;
//     onEmailChange: (value: string) => void;
// }
```

### 3. **Conditional Mapping**

```typescript
// Map only certain types
type StringKeys<T> = {
    [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];

type NumberKeys<T> = {
    [K in keyof T]: T[K] extends number ? K : never;
}[keyof T];

interface MixedTypes {
    id: number;
    name: string;
    active: boolean;
    score: number;
    description: string;
}

type StringProperties = StringKeys<MixedTypes>; // "name" | "description"
type NumberProperties = NumberKeys<MixedTypes>; // "id" | "score"

// Pick only string properties
type StringOnly<T> = {
    [K in StringKeys<T>]: T[K];
};

type MixedStrings = StringOnly<MixedTypes>;
// {
//     name: string;
//     description: string;
// }

// Nullable version of properties
type Nullable<T> = {
    [K in keyof T]: T[K] | null;
};

// Partial with specific keys
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type UserWithOptionalEmail = PartialBy<User, "email">;
// {
//     name: string;
//     age: number;
//     email?: string;
// }

// DeepPartial
type DeepPartial<T> = {
    [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

interface NestedConfig {
    server: {
        host: string;
        port: number;
        ssl: {
            enabled: boolean;
            certificate: string;
        };
    };
    database: {
        url: string;
        options: {
            timeout: number;
            retries: number;
        };
    };
}

type PartialConfig = DeepPartial<NestedConfig>;
// ทุก property รวมถึง nested จะเป็น optional
```

## 🛠️ Custom Utility Types

### 1. **Advanced Utility Types**

```typescript
// PickByType - Pick properties by their type
type PickByType<T, U> = {
    [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Employee {
    id: number;
    name: string;
    salary: number;
    isActive: boolean;
    department: string;
    startDate: Date;
}

type StringProperties = PickByType<Employee, string>;
// {
//     name: string;
//     department: string;
// }

type NumberProperties = PickByType<Employee, number>;
// {
//     id: number;
//     salary: number;
// }

// OmitByType - Omit properties by their type
type OmitByType<T, U> = {
    [K in keyof T as T[K] extends U ? never : K]: T[K];
};

type NonStringProperties = OmitByType<Employee, string>;
// {
//     id: number;
//     salary: number;
//     isActive: boolean;
//     startDate: Date;
// }

// Promisify - Convert all methods to return promises
type Promisify<T> = {
    [K in keyof T]: T[K] extends (...args: infer A) => infer R
        ? (...args: A) => Promise<R>
        : T[K];
};

interface SyncAPI {
    getUser(id: string): User;
    updateUser(id: string, data: Partial<User>): User;
    deleteUser(id: string): boolean;
    config: string;
}

type AsyncAPI = Promisify<SyncAPI>;
// {
//     getUser: (id: string) => Promise<User>;
//     updateUser: (id: string, data: Partial<User>) => Promise<User>;
//     deleteUser: (id: string) => Promise<boolean>;
//     config: string; // ไม่ใช่ function จึงไม่เปลี่ยน
// }
```

### 2. **Path and PathValue Types**

```typescript
// Path - Generate all possible paths to properties
type Path<T> = T extends object
    ? {
        [K in keyof T]: K extends string
            ? T[K] extends object
                ? K | `${K}.${Path<T[K]>}`
                : K
            : never;
    }[keyof T]
    : never;

// PathValue - Get type at a specific path
type PathValue<T, P extends Path<T>> = P extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
        ? Rest extends Path<T[Key]>
            ? PathValue<T[Key], Rest>
            : never
        : never
    : P extends keyof T
    ? T[P]
    : never;

interface NestedData {
    user: {
        profile: {
            name: string;
            age: number;
        };
        settings: {
            theme: "light" | "dark";
            notifications: boolean;
        };
    };
    metadata: {
        version: string;
    };
}

type AllPaths = Path<NestedData>;
// "user" | "metadata" | "user.profile" | "user.settings" | 
// "user.profile.name" | "user.profile.age" | "user.settings.theme" | 
// "user.settings.notifications" | "metadata.version"

type NameType = PathValue<NestedData, "user.profile.name">; // string
type ThemeType = PathValue<NestedData, "user.settings.theme">; // "light" | "dark"

// Safe property getter
function get<T, P extends Path<T>>(obj: T, path: P): PathValue<T, P> {
    const keys = path.split(".");
    let result: any = obj;
    
    for (const key of keys) {
        result = result[key];
        if (result === undefined) break;
    }
    
    return result;
}

const data: NestedData = {
    user: {
        profile: { name: "John", age: 30 },
        settings: { theme: "dark", notifications: true }
    },
    metadata: { version: "1.0.0" }
};

const userName = get(data, "user.profile.name"); // string
const theme = get(data, "user.settings.theme");  // "light" | "dark"
```

### 3. **Function Utility Types**

```typescript
// FunctionKeys - Get keys that are functions
type FunctionKeys<T> = {
    [K in keyof T]: T[K] extends (...args: any[]) => any ? K : never;
}[keyof T];

// NonFunctionKeys - Get keys that are not functions
type NonFunctionKeys<T> = {
    [K in keyof T]: T[K] extends (...args: any[]) => any ? never : K;
}[keyof T];

interface APIClass {
    name: string;
    version: number;
    getData(): string;
    setData(data: string): void;
    processData(input: any): Promise<any>;
}

type Methods = FunctionKeys<APIClass>; // "getData" | "setData" | "processData"
type Properties = NonFunctionKeys<APIClass>; // "name" | "version"

// MethodsOnly - Extract only methods
type MethodsOnly<T> = Pick<T, FunctionKeys<T>>;

// PropertiesOnly - Extract only properties
type PropertiesOnly<T> = Pick<T, NonFunctionKeys<T>>;

type APIClassMethods = MethodsOnly<APIClass>;
// {
//     getData(): string;
//     setData(data: string): void;
//     processData(input: any): Promise<any>;
// }

type APIClassProperties = PropertiesOnly<APIClass>;
// {
//     name: string;
//     version: number;
// }

// Mock type - Convert all methods to jest mock functions
type Mock<T> = {
    [K in keyof T]: T[K] extends (...args: infer A) => infer R
        ? jest.MockedFunction<(...args: A) => R>
        : T[K];
};

// Spy type - Convert all methods to spy functions
type Spy<T> = {
    [K in keyof T]: T[K] extends (...args: any[]) => any
        ? jest.SpyInstance
        : T[K];
};
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **Form Validation System**

```typescript
// Validation rules
type ValidationRule<T> = (value: T) => string | null;

type FormValidation<T> = {
    [K in keyof T]?: ValidationRule<T[K]>[];
};

type FormErrors<T> = {
    [K in keyof T]?: string;
};

type FormState<T> = {
    values: T;
    errors: FormErrors<T>;
    touched: Partial<Record<keyof T, boolean>>;
    isValid: boolean;
};

// Form manager class
class FormManager<T extends Record<string, any>> {
    private state: FormState<T>;
    private validationRules: FormValidation<T>;
    
    constructor(initialValues: T, validation: FormValidation<T> = {}) {
        this.state = {
            values: initialValues,
            errors: {},
            touched: {},
            isValid: true
        };
        this.validationRules = validation;
    }
    
    setValue<K extends keyof T>(field: K, value: T[K]): void {
        this.state.values[field] = value;
        this.state.touched[field] = true;
        this.validateField(field);
    }
    
    private validateField<K extends keyof T>(field: K): void {
        const rules = this.validationRules[field];
        if (!rules) return;
        
        const value = this.state.values[field];
        for (const rule of rules) {
            const error = rule(value);
            if (error) {
                this.state.errors[field] = error;
                this.updateValidState();
                return;
            }
        }
        
        delete this.state.errors[field];
        this.updateValidState();
    }
    
    private updateValidState(): void {
        this.state.isValid = Object.keys(this.state.errors).length === 0;
    }
    
    getState(): Readonly<FormState<T>> {
        return this.state;
    }
}

// Usage
interface UserForm {
    username: string;
    email: string;
    age: number;
}

const required = <T>(value: T): string | null => 
    value ? null : "This field is required";

const minLength = (min: number) => (value: string): string | null =>
    value.length >= min ? null : `Minimum length is ${min}`;

const email = (value: string): string | null =>
    /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? null : "Invalid email format";

const form = new FormManager<UserForm>(
    { username: "", email: "", age: 0 },
    {
        username: [required, minLength(3)],
        email: [required, email],
        age: [required]
    }
);
```

### 2. **Type-Safe Event System**

```typescript
// Event system with type safety
type EventMap = Record<string, any>;

type EventListener<T> = (data: T) => void;

class TypedEventEmitter<T extends EventMap> {
    private listeners: {
        [K in keyof T]?: EventListener<T[K]>[];
    } = {};
    
    on<K extends keyof T>(event: K, listener: EventListener<T[K]>): void {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        this.listeners[event]!.push(listener);
    }
    
    off<K extends keyof T>(event: K, listener: EventListener<T[K]>): void {
        const listeners = this.listeners[event];
        if (listeners) {
            const index = listeners.indexOf(listener);
            if (index !== -1) {
                listeners.splice(index, 1);
            }
        }
    }
    
    emit<K extends keyof T>(event: K, data: T[K]): void {
        const listeners = this.listeners[event];
        if (listeners) {
            listeners.forEach(listener => listener(data));
        }
    }
}

// Define your events
interface AppEvents {
    "user:login": { userId: string; timestamp: Date };
    "user:logout": { userId: string };
    "product:created": { product: Product; author: string };
    "error": { message: string; code: number };
}

const eventEmitter = new TypedEventEmitter<AppEvents>();

// Type-safe usage
eventEmitter.on("user:login", (data) => {
    console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

eventEmitter.emit("user:login", {
    userId: "123",
    timestamp: new Date()
});
```

## 📝 แบบฝึกหัด

### 1. **Advanced API Response Handler**
สร้างระบบจัดการ API response ที่มี type safety:
- Success, Error, Loading states
- Data transformation utilities
- Cache management

### 2. **Database Query Builder**
สร้าง type-safe query builder:
- SELECT with specific columns
- WHERE conditions with proper types
- JOIN operations
- Type-safe result mapping

### 3. **Configuration Management System**
ออกแบบระบบจัดการ config:
- Environment-specific configurations
- Type validation
- Deep merging with type safety
- Default value handling

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 12: Decorators](./12-decorators.md)

**บทต่อไป**: [บทที่ 14: Error Handling](./14-error-handling.md)

**กลับไปหน้าหลัก**: [README](./README.md)
