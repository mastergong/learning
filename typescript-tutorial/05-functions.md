# บทที่ 5: ฟังก์ชัน

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจการประกาศและใช้งานฟังก์ชันใน TypeScript
- เรียนรู้ Function Types และ Parameter Types
- เข้าใจ Optional และ Default Parameters
- เรียนรู้ Rest Parameters และ Overloads

## 🔧 การประกาศฟังก์ชัน

### 1. **Function Declaration**

```typescript
// Basic function declaration
function greet(name: string): string {
    return `Hello, ${name}!`;
}

// Function กับ multiple parameters
function add(a: number, b: number): number {
    return a + b;
}

// Function ที่ไม่ return ค่า
function logMessage(message: string): void {
    console.log(message);
}

// Function กับ optional parameters
function buildName(firstName: string, lastName?: string): string {
    if (lastName) {
        return `${firstName} ${lastName}`;
    }
    return firstName;
}

console.log(buildName("Alice"));           // "Alice"
console.log(buildName("Bob", "Smith"));    // "Bob Smith"
```

### 2. **Function Expression**

```typescript
// Basic function expression
const multiply = function(a: number, b: number): number {
    return a * b;
};

// Arrow function
const divide = (a: number, b: number): number => {
    return a / b;
};

// Arrow function (short form)
const square = (x: number): number => x * x;

// Arrow function กับ void return
const log = (message: string): void => console.log(message);
```

### 3. **Function Types**

```typescript
// Function type annotations
let mathOperation: (x: number, y: number) => number;

mathOperation = function(a: number, b: number): number {
    return a + b;
};

mathOperation = (a, b) => a * b; // Type inference

// Function type กับ interface
interface Calculator {
    add: (a: number, b: number) => number;
    subtract: (a: number, b: number) => number;
    multiply: (a: number, b: number) => number;
    divide: (a: number, b: number) => number;
}

const calculator: Calculator = {
    add: (a, b) => a + b,
    subtract: (a, b) => a - b,
    multiply: (a, b) => a * b,
    divide: (a, b) => a / b
};
```

## 🎛️ Parameters และ Arguments

### 1. **Optional Parameters**

```typescript
// Optional parameters ต้องอยู่หลัง required parameters
function createUser(name: string, age?: number, email?: string): object {
    const user: any = { name };
    
    if (age !== undefined) {
        user.age = age;
    }
    
    if (email) {
        user.email = email;
    }
    
    return user;
}

console.log(createUser("Alice"));                    // { name: "Alice" }
console.log(createUser("Bob", 25));                  // { name: "Bob", age: 25 }
console.log(createUser("Charlie", 30, "c@test.com")); // All properties
```

### 2. **Default Parameters**

```typescript
// Default parameters
function greetWithDefault(name: string, greeting: string = "Hello"): string {
    return `${greeting}, ${name}!`;
}

console.log(greetWithDefault("Alice"));              // "Hello, Alice!"
console.log(greetWithDefault("Bob", "Hi"));          // "Hi, Bob!"

// Default parameters กับ complex types
interface ConnectionOptions {
    host: string;
    port: number;
    ssl: boolean;
}

function connect(
    options: ConnectionOptions = {
        host: "localhost",
        port: 3000,
        ssl: false
    }
): void {
    console.log(`Connecting to ${options.host}:${options.port}`);
    console.log(`SSL: ${options.ssl}`);
}

connect(); // ใช้ default values
connect({ host: "example.com", port: 443, ssl: true });
```

### 3. **Rest Parameters**

```typescript
// Rest parameters
function sum(...numbers: number[]): number {
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// Rest parameters กับ other parameters
function buildMessage(prefix: string, ...parts: string[]): string {
    return prefix + " " + parts.join(" ");
}

console.log(buildMessage("Hello", "world", "from", "TypeScript")); 
// "Hello world from TypeScript"

// Rest parameters กับ destructuring
function processUsers(mainUser: string, ...otherUsers: string[]): void {
    console.log(`Main user: ${mainUser}`);
    console.log(`Other users: ${otherUsers.join(", ")}`);
}

processUsers("Alice", "Bob", "Charlie", "David");
```

## 🎯 Function Overloads

```typescript
// Function overloads - หลายรูปแบบการเรียกใช้
function parseInput(input: string): string[];
function parseInput(input: number): number;
function parseInput(input: boolean): boolean;

function parseInput(input: string | number | boolean): any {
    if (typeof input === "string") {
        return input.split(" ");
    } else if (typeof input === "number") {
        return input;
    } else {
        return input;
    }
}

// Usage
const stringResult = parseInput("hello world"); // string[]
const numberResult = parseInput(42);             // number
const booleanResult = parseInput(true);          // boolean

// Overload กับ complex parameters
interface Point2D {
    x: number;
    y: number;
}

interface Point3D {
    x: number;
    y: number;
    z: number;
}

function distance(p1: Point2D, p2: Point2D): number;
function distance(p1: Point3D, p2: Point3D): number;
function distance(p1: Point2D | Point3D, p2: Point2D | Point3D): number {
    const dx = p1.x - p2.x;
    const dy = p1.y - p2.y;
    
    if ('z' in p1 && 'z' in p2) {
        const dz = p1.z - p2.z;
        return Math.sqrt(dx * dx + dy * dy + dz * dz);
    }
    
    return Math.sqrt(dx * dx + dy * dy);
}
```

## 🏗️ Higher-Order Functions

### 1. **Functions as Parameters**

```typescript
// Function เป็น parameter
type Predicate<T> = (item: T) => boolean;
type Mapper<T, U> = (item: T) => U;

function filter<T>(array: T[], predicate: Predicate<T>): T[] {
    const result: T[] = [];
    for (const item of array) {
        if (predicate(item)) {
            result.push(item);
        }
    }
    return result;
}

function map<T, U>(array: T[], mapper: Mapper<T, U>): U[] {
    const result: U[] = [];
    for (const item of array) {
        result.push(mapper(item));
    }
    return result;
}

// Usage
const numbers = [1, 2, 3, 4, 5];
const evenNumbers = filter(numbers, (n) => n % 2 === 0);
const doubled = map(numbers, (n) => n * 2);
const strings = map(numbers, (n) => n.toString());
```

### 2. **Functions as Return Values**

```typescript
// Function ที่ return function
function createMultiplier(factor: number): (x: number) => number {
    return (x: number) => x * factor;
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(4)); // 12

// Currying
function curry<T, U, V>(fn: (a: T, b: U) => V): (a: T) => (b: U) => V {
    return (a: T) => (b: U) => fn(a, b);
}

const add = (a: number, b: number): number => a + b;
const curriedAdd = curry(add);
const addFive = curriedAdd(5);

console.log(addFive(3)); // 8
```

### 3. **Closures และ Lexical Scope**

```typescript
// Closures
function createCounter(initialValue: number = 0) {
    let count = initialValue;
    
    return {
        increment: (): number => ++count,
        decrement: (): number => --count,
        getValue: (): number => count,
        reset: (): void => { count = initialValue; }
    };
}

const counter = createCounter(10);
console.log(counter.getValue()); // 10
console.log(counter.increment()); // 11
console.log(counter.increment()); // 12
console.log(counter.decrement()); // 11
counter.reset();
console.log(counter.getValue()); // 10
```

## 🎨 Generic Functions

```typescript
// Generic functions
function identity<T>(arg: T): T {
    return arg;
}

const stringIdentity = identity<string>("hello");
const numberIdentity = identity<number>(42);
const autoInferred = identity("auto"); // TypeScript infers string

// Generic กับ constraints
interface Lengthwise {
    length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

logLength("hello");      // string has length
logLength([1, 2, 3]);    // array has length
// logLength(42);        // Error: number doesn't have length

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second];
}

const stringNumberPair = pair("hello", 42);      // [string, number]
const booleanArrayPair = pair(true, [1, 2, 3]);  // [boolean, number[]]
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **API Client Functions**

```typescript
interface ApiResponse<T> {
    success: boolean;
    data?: T;
    error?: string;
    timestamp: number;
}

interface User {
    id: number;
    name: string;
    email: string;
}

// Generic API function
async function apiCall<T>(
    url: string,
    method: "GET" | "POST" | "PUT" | "DELETE" = "GET",
    data?: any
): Promise<ApiResponse<T>> {
    try {
        const response = await fetch(url, {
            method,
            headers: {
                "Content-Type": "application/json",
            },
            body: data ? JSON.stringify(data) : undefined,
        });
        
        const result = await response.json();
        
        return {
            success: response.ok,
            data: response.ok ? result : undefined,
            error: response.ok ? undefined : result.message,
            timestamp: Date.now()
        };
    } catch (error) {
        return {
            success: false,
            error: error instanceof Error ? error.message : "Unknown error",
            timestamp: Date.now()
        };
    }
}

// Specific API functions
const getUser = (id: number): Promise<ApiResponse<User>> =>
    apiCall<User>(`/api/users/${id}`);

const createUser = (userData: Omit<User, "id">): Promise<ApiResponse<User>> =>
    apiCall<User>("/api/users", "POST", userData);

const updateUser = (id: number, userData: Partial<User>): Promise<ApiResponse<User>> =>
    apiCall<User>(`/api/users/${id}`, "PUT", userData);
```

### 2. **Validation Functions**

```typescript
// Validation function types
type ValidationResult = {
    isValid: boolean;
    errors: string[];
};

type Validator<T> = (value: T) => ValidationResult;

// Basic validators
const required: Validator<string> = (value) => ({
    isValid: value.trim().length > 0,
    errors: value.trim().length > 0 ? [] : ["Field is required"]
});

const minLength = (min: number): Validator<string> => (value) => ({
    isValid: value.length >= min,
    errors: value.length >= min ? [] : [`Minimum length is ${min}`]
});

const email: Validator<string> = (value) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return {
        isValid: emailRegex.test(value),
        errors: emailRegex.test(value) ? [] : ["Invalid email format"]
    };
};

// Compose validators
function composeValidators<T>(...validators: Validator<T>[]): Validator<T> {
    return (value: T) => {
        const allErrors: string[] = [];
        let isValid = true;
        
        for (const validator of validators) {
            const result = validator(value);
            if (!result.isValid) {
                isValid = false;
                allErrors.push(...result.errors);
            }
        }
        
        return { isValid, errors: allErrors };
    };
}

// Usage
const validateUsername = composeValidators(required, minLength(3));
const validateEmail = composeValidators(required, email);

console.log(validateUsername(""));        // { isValid: false, errors: ["Field is required"] }
console.log(validateUsername("ab"));      // { isValid: false, errors: ["Minimum length is 3"] }
console.log(validateUsername("alice"));   // { isValid: true, errors: [] }
```

### 3. **Event Handler Functions**

```typescript
// Event system
interface EventMap {
    "user:login": { userId: number; timestamp: Date };
    "user:logout": { userId: number; timestamp: Date };
    "order:created": { orderId: string; userId: number; total: number };
    "order:updated": { orderId: string; status: string };
}

type EventHandler<T> = (data: T) => void;

class EventEmitter {
    private handlers: { [K in keyof EventMap]?: EventHandler<EventMap[K]>[] } = {};
    
    on<K extends keyof EventMap>(event: K, handler: EventHandler<EventMap[K]>): void {
        if (!this.handlers[event]) {
            this.handlers[event] = [];
        }
        this.handlers[event]!.push(handler);
    }
    
    emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
        const eventHandlers = this.handlers[event];
        if (eventHandlers) {
            eventHandlers.forEach(handler => handler(data));
        }
    }
    
    off<K extends keyof EventMap>(event: K, handler: EventHandler<EventMap[K]>): void {
        const eventHandlers = this.handlers[event];
        if (eventHandlers) {
            const index = eventHandlers.indexOf(handler);
            if (index > -1) {
                eventHandlers.splice(index, 1);
            }
        }
    }
}

// Usage
const emitter = new EventEmitter();

emitter.on("user:login", (data) => {
    console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.on("order:created", (data) => {
    console.log(`Order ${data.orderId} created for user ${data.userId}, total: $${data.total}`);
});

emitter.emit("user:login", { userId: 123, timestamp: new Date() });
emitter.emit("order:created", { orderId: "ORD-001", userId: 123, total: 99.99 });
```

## 📝 แบบฝึกหัด

### 1. **Basic Functions**
สร้างฟังก์ชันเหล่านี้:
- `calculateArea` - คำนวณพื้นที่สี่เหลี่ยม (กว้าง, ยาว)
- `formatCurrency` - จัดรูปแบบตัวเลขเป็นสกุลเงิน (จำนวน, สกุลเงิน?)
- `isEven` - ตรวจสอบว่าเลขเป็นเลขคู่หรือไม่

### 2. **Advanced Functions**
สร้าง utility functions:
- `debounce` - function ที่ทำ debouncing
- `memoize` - function ที่ cache ผลลัพธ์
- `retry` - function ที่ลองใหม่เมื่อเกิด error

### 3. **Type-safe Functions**
สร้างระบบ CRUD สำหรับจัดการข้อมูล:
- Generic CRUD operations
- Type-safe database queries
- Validation pipeline

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 4: ตัวแปรและการประกาศ](./04-variables-declarations.md)

**บทต่อไป**: [บทที่ 6: Interface และ Type](./06-interfaces-types.md)

**กลับไปหน้าหลัก**: [README](./README.md)
