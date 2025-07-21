# บทที่ 4: ตัวแปรและการประกาศ

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจการประกาศตัวแปรด้วย var, let, const
- เรียนรู้ Scope และ Hoisting ใน TypeScript
- เข้าใจ Destructuring กับ Type Annotations
- เรียนรู้ Optional Properties และ Readonly

## 📝 การประกาศตัวแปร

### 1. **var, let, const**

```typescript
// var - Function scoped (ไม่แนะนำให้ใช้)
var oldStyle = "This is old JavaScript style";

// let - Block scoped, สามารถเปลี่ยนค่าได้
let mutableValue: string = "Can be changed";
mutableValue = "Changed!";

// const - Block scoped, ไม่สามารถเปลี่ยนค่าได้
const immutableValue: string = "Cannot be changed";
// immutableValue = "Error!"; // ❌ Error

// const กับ objects (properties สามารถเปลี่ยนได้)
const user = {
    name: "Alice",
    age: 30
};
user.age = 31; // ✅ OK - เปลี่ยน property ได้
// user = {}; // ❌ Error - ไม่สามารถ reassign ได้
```

### 2. **Scope ใน TypeScript**

```typescript
function demonstrateScope() {
    var varVariable = "var scope";
    let letVariable = "let scope";
    const constVariable = "const scope";
    
    if (true) {
        var varInBlock = "var in block";      // Function scoped
        let letInBlock = "let in block";      // Block scoped
        const constInBlock = "const in block"; // Block scoped
        
        console.log(varVariable);    // ✅ Accessible
        console.log(letVariable);    // ✅ Accessible
        console.log(constVariable);  // ✅ Accessible
    }
    
    console.log(varInBlock);    // ✅ Accessible (function scoped)
    // console.log(letInBlock); // ❌ Error (block scoped)
    // console.log(constInBlock); // ❌ Error (block scoped)
}
```

### 3. **Hoisting**

```typescript
// var hoisting
console.log(hoistedVar); // undefined (not error)
var hoistedVar = "I'm hoisted";

// let และ const hoisting
// console.log(notHoisted); // ❌ ReferenceError
let notHoisted = "I'm not hoisted";

// Function hoisting
console.log(hoistedFunction()); // "I'm hoisted!" - works!

function hoistedFunction(): string {
    return "I'm hoisted!";
}
```

## 🎯 Type Annotations กับการประกาศตัวแปร

### 1. **Explicit Type Annotations**

```typescript
// Basic type annotations
let username: string = "john_doe";
let age: number = 25;
let isActive: boolean = true;
let scores: number[] = [95, 87, 92];

// Type annotations กับ const
const API_URL: string = "https://api.example.com";
const MAX_RETRIES: number = 3;
const FEATURES: string[] = ["auth", "payments", "analytics"];
```

### 2. **Type Inference**

```typescript
// TypeScript สามารถเดา type ได้
let inferredString = "Hello"; // string
let inferredNumber = 42;      // number
let inferredBoolean = true;   // boolean
let inferredArray = [1, 2, 3]; // number[]

// เมื่อไหร่ควรใช้ type annotation
let explicitAny: any = getValue(); // เมื่อไม่รู้ type
let explicitUnion: string | number = getId(); // เมื่อต้องการ union type
```

### 3. **Optional Properties**

```typescript
// Optional properties ใน object types
interface User {
    id: number;
    name: string;
    email?: string;    // Optional
    phone?: string;    // Optional
}

let user1: User = {
    id: 1,
    name: "Alice"
    // email และ phone ไม่จำเป็น
};

let user2: User = {
    id: 2,
    name: "Bob",
    email: "bob@example.com"
    // phone ยังไม่จำเป็น
};

// Optional parameters ใน function
function greetUser(name: string, title?: string): string {
    if (title) {
        return `Hello, ${title} ${name}!`;
    }
    return `Hello, ${name}!`;
}

console.log(greetUser("Alice"));           // "Hello, Alice!"
console.log(greetUser("Bob", "Dr."));      // "Hello, Dr. Bob!"
```

### 4. **Readonly Properties**

```typescript
// Readonly properties
interface ReadonlyUser {
    readonly id: number;
    name: string;
    readonly createdAt: Date;
}

let user: ReadonlyUser = {
    id: 1,
    name: "Alice",
    createdAt: new Date()
};

user.name = "Alice Smith"; // ✅ OK
// user.id = 2;            // ❌ Error - readonly
// user.createdAt = new Date(); // ❌ Error - readonly

// Readonly arrays
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // ❌ Error
// readonlyNumbers[0] = 10; // ❌ Error

// ReadonlyArray utility type
let readonlyArray: ReadonlyArray<string> = ["a", "b", "c"];
```

## 🎁 Destructuring กับ Type Annotations

### 1. **Array Destructuring**

```typescript
// Basic array destructuring
let numbers: number[] = [1, 2, 3, 4, 5];
let [first, second, ...rest]: [number, number, ...number[]] = numbers;

console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]

// Destructuring กับ tuple
let coordinate: [number, number] = [10, 20];
let [x, y]: [number, number] = coordinate;

// Skipping elements
let colors: string[] = ["red", "green", "blue"];
let [primary, , tertiary]: string[] = colors; // skip green
```

### 2. **Object Destructuring**

```typescript
// Basic object destructuring
interface Person {
    name: string;
    age: number;
    email?: string;
}

let person: Person = {
    name: "Alice",
    age: 30,
    email: "alice@example.com"
};

// Destructuring กับ type annotation
let { name, age, email }: Person = person;

// Destructuring กับ rename
let { name: fullName, age: currentAge }: Person = person;

// Destructuring กับ default values
let { name, age, email = "no-email@example.com" }: Person = person;

// Nested destructuring
interface Address {
    street: string;
    city: string;
    country: string;
}

interface PersonWithAddress {
    name: string;
    address: Address;
}

let personWithAddress: PersonWithAddress = {
    name: "Bob",
    address: {
        street: "123 Main St",
        city: "Bangkok",
        country: "Thailand"
    }
};

let { 
    name, 
    address: { city, country } 
}: PersonWithAddress = personWithAddress;
```

### 3. **Function Parameter Destructuring**

```typescript
// Function parameter destructuring
interface ApiOptions {
    method?: "GET" | "POST" | "PUT" | "DELETE";
    headers?: Record<string, string>;
    timeout?: number;
}

function makeApiCall(
    url: string,
    { method = "GET", headers = {}, timeout = 5000 }: ApiOptions = {}
): Promise<any> {
    console.log(`Making ${method} request to ${url}`);
    console.log(`Timeout: ${timeout}ms`);
    console.log("Headers:", headers);
    
    // API call implementation
    return Promise.resolve();
}

// Usage
makeApiCall("https://api.example.com/users");
makeApiCall("https://api.example.com/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    timeout: 10000
});
```

## 🔧 Advanced Variable Declaration Patterns

### 1. **Assertion Functions**

```typescript
// Type assertion functions
function assertIsNumber(value: unknown): asserts value is number {
    if (typeof value !== "number") {
        throw new Error("Expected number");
    }
}

function processValue(input: unknown) {
    assertIsNumber(input);
    // หลังจากนี้ TypeScript รู้ว่า input เป็น number
    console.log(input.toFixed(2));
}
```

### 2. **Discriminated Unions**

```typescript
// Discriminated unions กับ variable declarations
interface LoadingState {
    status: "loading";
}

interface SuccessState {
    status: "success";
    data: any;
}

interface ErrorState {
    status: "error";
    error: string;
}

type AppState = LoadingState | SuccessState | ErrorState;

function handleState(state: AppState) {
    switch (state.status) {
        case "loading":
            console.log("Loading...");
            break;
        case "success":
            console.log("Data:", state.data); // TypeScript รู้ว่ามี data
            break;
        case "error":
            console.log("Error:", state.error); // TypeScript รู้ว่ามี error
            break;
    }
}
```

### 3. **Const Assertions**

```typescript
// Const assertions
let regularArray = ["red", "green", "blue"]; // string[]
let readonlyArray = ["red", "green", "blue"] as const; // readonly ["red", "green", "blue"]

let regularObject = { x: 10, y: 20 }; // { x: number; y: number; }
let readonlyObject = { x: 10, y: 20 } as const; // { readonly x: 10; readonly y: 20; }

// ใช้กับ enum-like objects
const THEMES = {
    LIGHT: "light",
    DARK: "dark",
    AUTO: "auto"
} as const;

type Theme = typeof THEMES[keyof typeof THEMES]; // "light" | "dark" | "auto"
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **Configuration Management**

```typescript
interface DatabaseConfig {
    readonly host: string;
    readonly port: number;
    readonly database: string;
    username: string;
    password: string;
    ssl?: boolean;
    readonly maxConnections: number;
}

// การใช้งาน
const dbConfig: DatabaseConfig = {
    host: "localhost",
    port: 5432,
    database: "myapp",
    username: "admin",
    password: "secret",
    ssl: true,
    maxConnections: 10
};

// ไม่สามารถเปลี่ยน readonly properties
// dbConfig.host = "newhost"; // ❌ Error
dbConfig.username = "newuser"; // ✅ OK
```

### 2. **State Management**

```typescript
interface UserState {
    readonly id: number;
    name: string;
    email: string;
    preferences: {
        theme: "light" | "dark";
        notifications: boolean;
    };
    readonly createdAt: Date;
    lastLoginAt?: Date;
}

let userState: UserState = {
    id: 1,
    name: "Alice Johnson",
    email: "alice@example.com",
    preferences: {
        theme: "light",
        notifications: true
    },
    createdAt: new Date("2023-01-15"),
    lastLoginAt: new Date()
};

// การอัปเดต state (immutable pattern)
function updateUserName(state: UserState, newName: string): UserState {
    return {
        ...state,
        name: newName
    };
}

function updateTheme(state: UserState, theme: "light" | "dark"): UserState {
    return {
        ...state,
        preferences: {
            ...state.preferences,
            theme
        }
    };
}
```

### 3. **API Response Handling**

```typescript
interface ApiResponse<T> {
    readonly success: boolean;
    readonly data?: T;
    readonly error?: string;
    readonly timestamp: number;
}

// Generic function สำหรับ handle API responses
function handleApiResponse<T>(response: ApiResponse<T>) {
    const { success, data, error, timestamp } = response;
    
    if (success && data) {
        console.log("Success at", new Date(timestamp));
        return data;
    } else if (error) {
        console.error("API Error:", error);
        throw new Error(error);
    } else {
        throw new Error("Unknown API response format");
    }
}

// Usage
interface User {
    id: number;
    name: string;
    email: string;
}

const userResponse: ApiResponse<User> = {
    success: true,
    data: { id: 1, name: "Alice", email: "alice@example.com" },
    timestamp: Date.now()
};

const user = handleApiResponse(userResponse);
```

## 📝 แบบฝึกหัด

### 1. **Basic Variable Declaration**
```typescript
// ประกาศตัวแปรด้วย type annotations ที่เหมาะสม:
// - ชื่อผู้ใช้ (ไม่สามารถเปลี่ยนได้)
// - อายุ (สามารถเปลี่ยนได้)
// - อีเมล (optional)
// - สถานะออนไลน์ (boolean)
// - คะแนนสอบ (array ของ numbers)
```

### 2. **Complex Object with Destructuring**
```typescript
// สร้าง interface สำหรับข้อมูลสินค้า:
interface Product {
    // เพิ่ม properties ที่เหมาะสม
}

// สร้าง function ที่ใช้ destructuring รับ Product
// และ return ข้อมูลที่จัดรูปแบบแล้ว
```

### 3. **State Management Pattern**
```typescript
// สร้างระบบจัดการสถานะของ Shopping Cart:
// - readonly id
// - items (array ของสินค้า)
// - total (คำนวณจาก items)
// - discount (optional)
// - functions สำหรับ add/remove items
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 3: ชนิดข้อมูลพื้นฐาน](./03-basic-types.md)

**บทต่อไป**: [บทที่ 5: ฟังก์ชัน](./05-functions.md)

**กลับไปหน้าหลัก**: [README](./README.md)
