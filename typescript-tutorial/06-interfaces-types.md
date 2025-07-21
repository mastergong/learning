# บทที่ 6: Interface และ Type

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจการใช้งาน Interface และ Type Aliases
- เรียนรู้ความแตกต่างระหว่าง Interface และ Type
- เข้าใจ Extending Interfaces และ Intersection Types
- เรียนรู้ Index Signatures และ Mapped Types

## 🏗️ Interface

### 1. **การประกาศ Interface**

```typescript
// Basic interface
interface Person {
    name: string;
    age: number;
    email: string;
}

// การใช้งาน interface
const user: Person = {
    name: "Alice",
    age: 30,
    email: "alice@example.com"
};

// Interface กับ optional properties
interface User {
    id: number;
    username: string;
    email: string;
    avatar?: string;        // Optional
    lastLogin?: Date;       // Optional
}

// Interface กับ readonly properties
interface Config {
    readonly apiUrl: string;
    readonly version: string;
    timeout: number;
}

const config: Config = {
    apiUrl: "https://api.example.com",
    version: "1.0.0",
    timeout: 5000
};

// config.apiUrl = "new-url"; // ❌ Error - readonly
config.timeout = 10000; // ✅ OK
```

### 2. **Method Definitions**

```typescript
interface Calculator {
    // Method signature
    add(a: number, b: number): number;
    subtract(a: number, b: number): number;
    
    // Function property
    multiply: (a: number, b: number) => number;
    
    // Optional method
    divide?(a: number, b: number): number;
}

// Implementation
const calculator: Calculator = {
    add(a, b) {
        return a + b;
    },
    subtract(a, b) {
        return a - b;
    },
    multiply: (a, b) => a * b
    // divide is optional, so we can omit it
};

// Interface สำหรับ functions
interface StringProcessor {
    (input: string): string;
}

const toUpperCase: StringProcessor = (input) => input.toUpperCase();
const toLowerCase: StringProcessor = (input) => input.toLowerCase();
```

### 3. **Extending Interfaces**

```typescript
// Base interface
interface Animal {
    name: string;
    age: number;
}

// Extending interface
interface Dog extends Animal {
    breed: string;
    bark(): void;
}

interface Cat extends Animal {
    color: string;
    meow(): void;
}

// Multiple inheritance
interface Pet extends Animal {
    owner: string;
}

interface ServiceDog extends Dog, Pet {
    serviceType: string;
    isWorking: boolean;
}

// Implementation
const myDog: ServiceDog = {
    name: "Buddy",
    age: 3,
    breed: "Golden Retriever",
    owner: "John",
    serviceType: "Guide Dog",
    isWorking: true,
    bark() {
        console.log("Woof!");
    }
};
```

## 🔧 Type Aliases

### 1. **Basic Type Aliases**

```typescript
// Primitive type aliases
type UserID = string;
type Age = number;
type IsActive = boolean;

// Union type aliases
type Status = "pending" | "approved" | "rejected";
type Theme = "light" | "dark" | "auto";

// Object type aliases
type Point = {
    x: number;
    y: number;
};

type Rectangle = {
    width: number;
    height: number;
    position: Point;
};

// Function type aliases
type EventHandler = (event: Event) => void;
type AsyncProcessor<T> = (data: T) => Promise<void>;
```

### 2. **Generic Type Aliases**

```typescript
// Generic type aliases
type Result<T, E = Error> = {
    success: boolean;
    data?: T;
    error?: E;
};

type ApiResponse<T> = {
    status: number;
    message: string;
    data: T;
    timestamp: Date;
};

// Usage
type UserResult = Result<User>;
type ProductResponse = ApiResponse<Product[]>;

// Complex generic types
type EventMap = {
    click: { x: number; y: number };
    keypress: { key: string; ctrlKey: boolean };
    submit: { formData: FormData };
};

type EventListener<K extends keyof EventMap> = (data: EventMap[K]) => void;

// Usage
const clickHandler: EventListener<"click"> = (data) => {
    console.log(`Clicked at ${data.x}, ${data.y}`);
};
```

### 3. **Intersection Types**

```typescript
// Intersection types with type aliases
type Name = {
    firstName: string;
    lastName: string;
};

type Contact = {
    email: string;
    phone: string;
};

type UserProfile = Name & Contact & {
    id: string;
    createdAt: Date;
};

// Usage
const userProfile: UserProfile = {
    id: "123",
    firstName: "John",
    lastName: "Doe",
    email: "john@example.com",
    phone: "+1234567890",
    createdAt: new Date()
};

// Intersection กับ interfaces
interface Timestamped {
    createdAt: Date;
    updatedAt: Date;
}

interface Identifiable {
    id: string;
}

type Entity = Timestamped & Identifiable;

interface User extends Entity {
    name: string;
    email: string;
}
```

## 🔍 Interface vs Type - ความแตกต่าง

### 1. **Declaration Merging**

```typescript
// Interface - รองรับ declaration merging
interface User {
    name: string;
}

interface User {
    age: number;
}

// ผลลัพธ์: User มี name และ age
const user: User = {
    name: "Alice",
    age: 30
};

// Type - ไม่รองรับ declaration merging
type Product = {
    name: string;
};

// type Product = {  // ❌ Error: Duplicate identifier
//     price: number;
// };
```

### 2. **Computed Properties**

```typescript
// Type - รองรับ computed properties
const prefix = "api";
type ApiRoutes = {
    [key in `${typeof prefix}_${"users" | "products" | "orders"}`]: string;
};

// Interface - ไม่รองรับ computed properties โดยตรง
// interface ApiRoutes {
//     [key in `api_${"users" | "products"}`]: string; // ❌ Error
// }

// แต่สามารถใช้ index signatures ได้
interface ApiConfig {
    [key: string]: string;
}
```

### 3. **Union Types**

```typescript
// Type - รองรับ union types
type StringOrNumber = string | number;
type Status = "loading" | "success" | "error";

// Interface - ไม่สามารถเป็น union type ได้
// interface StringOrNumber = string | number; // ❌ Error

// แต่สามารถใช้ใน properties ได้
interface Response {
    data: string | number;
    status: "loading" | "success" | "error";
}
```

## 📇 Index Signatures

### 1. **Basic Index Signatures**

```typescript
// String index signature
interface StringDictionary {
    [key: string]: string;
}

const colors: StringDictionary = {
    primary: "#007bff",
    secondary: "#6c757d",
    success: "#28a745"
};

// Number index signature
interface NumberDictionary {
    [index: number]: string;
}

const weekdays: NumberDictionary = {
    0: "Sunday",
    1: "Monday",
    2: "Tuesday"
};

// Mixed index signatures
interface MixedDictionary {
    [key: string]: any;
    [index: number]: string;
    // Number index type ต้องเป็น subtype ของ string index type
}
```

### 2. **Advanced Index Signatures**

```typescript
// Index signature กับ known properties
interface UserSettings {
    theme: "light" | "dark";
    language: string;
    [key: string]: any; // อนุญาตให้มี property อื่นๆ
}

const settings: UserSettings = {
    theme: "dark",
    language: "en",
    customProperty: "value",
    anotherSetting: 123
};

// Conditional index signatures
type ApiEndpoints<T extends string> = {
    [K in T]: {
        method: "GET" | "POST" | "PUT" | "DELETE";
        path: string;
        handler: Function;
    };
};

type UserEndpoints = ApiEndpoints<"getUser" | "createUser" | "updateUser">;
```

## 🗺️ Mapped Types

### 1. **Basic Mapped Types**

```typescript
// Mapped types สร้างจาก existing types
interface User {
    id: string;
    name: string;
    email: string;
    age: number;
}

// ทำให้ทุก property เป็น optional
type PartialUser = {
    [K in keyof User]?: User[K];
};

// ทำให้ทุก property เป็น readonly
type ReadonlyUser = {
    readonly [K in keyof User]: User[K];
};

// ทำให้ทุก property เป็น string
type StringifiedUser = {
    [K in keyof User]: string;
};
```

### 2. **Advanced Mapped Types**

```typescript
// Template literal types กับ mapped types
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Setters<T> = {
    [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// {
//     getName: () => string;
//     getAge: () => number;
// }

type PersonSetters = Setters<Person>;
// {
//     setName: (value: string) => void;
//     setAge: (value: number) => void;
// }

// Filtering properties
type NonFunctionPropertyNames<T> = {
    [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface UserWithMethods {
    id: string;
    name: string;
    save(): void;
    delete(): void;
}

type UserData = NonFunctionProperties<UserWithMethods>;
// { id: string; name: string; }
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **API Client Design**

```typescript
// API response interfaces
interface BaseResponse {
    success: boolean;
    message: string;
    timestamp: Date;
}

interface DataResponse<T> extends BaseResponse {
    data: T;
}

interface ErrorResponse extends BaseResponse {
    error: {
        code: string;
        details?: Record<string, any>;
    };
}

type ApiResponse<T> = DataResponse<T> | ErrorResponse;

// API endpoints configuration
interface ApiEndpoint {
    method: "GET" | "POST" | "PUT" | "DELETE";
    path: string;
    requiresAuth: boolean;
}

type ApiEndpoints = {
    // User endpoints
    getUser: ApiEndpoint;
    createUser: ApiEndpoint;
    updateUser: ApiEndpoint;
    deleteUser: ApiEndpoint;
    
    // Product endpoints
    getProducts: ApiEndpoint;
    createProduct: ApiEndpoint;
};

const endpoints: ApiEndpoints = {
    getUser: {
        method: "GET",
        path: "/users/:id",
        requiresAuth: true
    },
    createUser: {
        method: "POST",
        path: "/users",
        requiresAuth: false
    },
    updateUser: {
        method: "PUT",
        path: "/users/:id",
        requiresAuth: true
    },
    deleteUser: {
        method: "DELETE",
        path: "/users/:id",
        requiresAuth: true
    },
    getProducts: {
        method: "GET",
        path: "/products",
        requiresAuth: false
    },
    createProduct: {
        method: "POST",
        path: "/products",
        requiresAuth: true
    }
};
```

### 2. **State Management**

```typescript
// Application state interfaces
interface UserState {
    currentUser: User | null;
    isLoading: boolean;
    error: string | null;
}

interface ProductState {
    products: Product[];
    selectedProduct: Product | null;
    filters: ProductFilters;
    isLoading: boolean;
}

interface ProductFilters {
    category?: string;
    priceRange?: {
        min: number;
        max: number;
    };
    inStock?: boolean;
}

// Action types
interface BaseAction {
    type: string;
    timestamp: Date;
}

interface UserLoginAction extends BaseAction {
    type: "USER_LOGIN";
    payload: {
        user: User;
    };
}

interface UserLogoutAction extends BaseAction {
    type: "USER_LOGOUT";
}

interface SetProductsAction extends BaseAction {
    type: "SET_PRODUCTS";
    payload: {
        products: Product[];
    };
}

type UserAction = UserLoginAction | UserLogoutAction;
type ProductAction = SetProductsAction;
type AppAction = UserAction | ProductAction;

// Reducer type
type Reducer<TState, TAction> = (state: TState, action: TAction) => TState;

type UserReducer = Reducer<UserState, UserAction>;
type ProductReducer = Reducer<ProductState, ProductAction>;
```

### 3. **Form Validation**

```typescript
// Form field types
interface FormField<T = any> {
    value: T;
    error?: string;
    touched: boolean;
    required: boolean;
}

// Form state
type FormState<T> = {
    [K in keyof T]: FormField<T[K]>;
};

// Validation rules
type ValidationRule<T> = (value: T) => string | undefined;

type ValidationRules<T> = {
    [K in keyof T]?: ValidationRule<T[K]>[];
};

// Example usage
interface UserRegistration {
    username: string;
    email: string;
    password: string;
    confirmPassword: string;
    age: number;
}

type UserRegistrationForm = FormState<UserRegistration>;

const validationRules: ValidationRules<UserRegistration> = {
    username: [
        (value) => value.length < 3 ? "Username must be at least 3 characters" : undefined,
        (value) => /^[a-zA-Z0-9_]+$/.test(value) ? undefined : "Username can only contain letters, numbers, and underscores"
    ],
    email: [
        (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? undefined : "Invalid email format"
    ],
    password: [
        (value) => value.length < 8 ? "Password must be at least 8 characters" : undefined
    ],
    age: [
        (value) => value >= 13 ? undefined : "Must be at least 13 years old"
    ]
};
```

## 📝 แบบฝึกหัด

### 1. **E-commerce Interfaces**
สร้าง interfaces สำหรับระบบ e-commerce:
```typescript
// Product, Category, Order, Customer
// รวมถึง methods สำหรับ CRUD operations
```

### 2. **Event System**
ออกแบบ type-safe event system:
```typescript
// Event types, Event handlers, Event emitter
// ใช้ mapped types และ template literal types
```

### 3. **Database Models**
สร้าง type system สำหรับ database models:
```typescript
// Base model, Relationships, Queries
// ใช้ generics และ conditional types
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 5: ฟังก์ชัน](./05-functions.md)

**บทต่อไป**: [บทที่ 7: คลาสและการสืบทอด](./07-classes-inheritance.md)

**กลับไปหน้าหลัก**: [README](./README.md)
