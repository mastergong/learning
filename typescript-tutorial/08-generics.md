# บทที่ 8: Generics

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจแนวคิดและประโยชน์ของ Generics
- เรียนรู้การสร้าง Generic Functions และ Classes
- เข้าใจ Generic Constraints และ Conditional Types
- เรียนรู้ Advanced Generic Patterns

## 🧬 แนะนำ Generics

### 1. **ปัญหาที่ Generics แก้ไข**

```typescript
// ❌ ปัญหา: ต้องสร้าง function หลายตัวสำหรับ type ต่างๆ
function getFirstString(arr: string[]): string | undefined {
    return arr[0];
}

function getFirstNumber(arr: number[]): number | undefined {
    return arr[0];
}

function getFirstBoolean(arr: boolean[]): boolean | undefined {
    return arr[0];
}

// ❌ หรือใช้ any (สูญเสีย type safety)
function getFirstAny(arr: any[]): any {
    return arr[0];
}

// ✅ แก้ปัญหาด้วย Generics
function getFirst<T>(arr: T[]): T | undefined {
    return arr[0];
}

// การใช้งาน
const firstString = getFirst(["a", "b", "c"]);     // string | undefined
const firstNumber = getFirst([1, 2, 3]);           // number | undefined
const firstBoolean = getFirst([true, false]);      // boolean | undefined
```

### 2. **Generic Syntax**

```typescript
// Generic function
function identity<T>(arg: T): T {
    return arg;
}

// การเรียกใช้แบบต่างๆ
const str1 = identity<string>("hello");    // explicit type
const str2 = identity("hello");            // type inference
const num1 = identity<number>(42);         // explicit type
const num2 = identity(42);                 // type inference

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second];
}

const stringNumberPair = pair("hello", 42);        // [string, number]
const booleanStringPair = pair(true, "world");     // [boolean, string]
```

## 🔧 Generic Functions

### 1. **Advanced Generic Functions**

```typescript
// Generic function กับ array operations
function map<T, U>(array: T[], transform: (item: T) => U): U[] {
    const result: U[] = [];
    for (const item of array) {
        result.push(transform(item));
    }
    return result;
}

// การใช้งาน
const numbers = [1, 2, 3, 4, 5];
const doubled = map(numbers, x => x * 2);           // number[]
const strings = map(numbers, x => x.toString());    // string[]
const objects = map(numbers, x => ({ value: x }));  // { value: number }[]

// Generic function กับ filtering
function filter<T>(array: T[], predicate: (item: T) => boolean): T[] {
    const result: T[] = [];
    for (const item of array) {
        if (predicate(item)) {
            result.push(item);
        }
    }
    return result;
}

const evenNumbers = filter(numbers, x => x % 2 === 0);  // number[]
const longStrings = filter(["a", "hello", "world"], s => s.length > 2);  // string[]

// Generic function กับ reduce
function reduce<T, U>(
    array: T[], 
    reducer: (acc: U, current: T, index: number) => U, 
    initialValue: U
): U {
    let accumulator = initialValue;
    for (let i = 0; i < array.length; i++) {
        accumulator = reducer(accumulator, array[i], i);
    }
    return accumulator;
}

const sum = reduce([1, 2, 3, 4], (acc, curr) => acc + curr, 0);        // number
const joined = reduce(["a", "b", "c"], (acc, curr) => acc + curr, "");  // string
```

### 2. **Generic Function Overloads**

```typescript
// Generic overloads
function process<T>(input: T[]): T[];
function process<T>(input: T): T;
function process<T>(input: T | T[]): T | T[] {
    if (Array.isArray(input)) {
        return input.map(item => item); // process array
    }
    return input; // process single item
}

const processedArray = process([1, 2, 3]);  // number[]
const processedItem = process(42);          // number
```

## 🏗️ Generic Interfaces และ Types

### 1. **Generic Interfaces**

```typescript
// Generic interface
interface Container<T> {
    value: T;
    getValue(): T;
    setValue(value: T): void;
}

// Implementation
class Box<T> implements Container<T> {
    constructor(private _value: T) {}
    
    getValue(): T {
        return this._value;
    }
    
    setValue(value: T): void {
        this._value = value;
    }
    
    get value(): T {
        return this._value;
    }
}

const stringBox = new Box<string>("hello");
const numberBox = new Box<number>(42);

// Generic interface กับ multiple parameters
interface KeyValuePair<K, V> {
    key: K;
    value: V;
}

interface Dictionary<K, V> {
    get(key: K): V | undefined;
    set(key: K, value: V): void;
    has(key: K): boolean;
    delete(key: K): boolean;
    entries(): KeyValuePair<K, V>[];
}

class SimpleDictionary<K, V> implements Dictionary<K, V> {
    private items: KeyValuePair<K, V>[] = [];
    
    get(key: K): V | undefined {
        const item = this.items.find(i => i.key === key);
        return item?.value;
    }
    
    set(key: K, value: V): void {
        const existingIndex = this.items.findIndex(i => i.key === key);
        if (existingIndex >= 0) {
            this.items[existingIndex].value = value;
        } else {
            this.items.push({ key, value });
        }
    }
    
    has(key: K): boolean {
        return this.items.some(i => i.key === key);
    }
    
    delete(key: K): boolean {
        const index = this.items.findIndex(i => i.key === key);
        if (index >= 0) {
            this.items.splice(index, 1);
            return true;
        }
        return false;
    }
    
    entries(): KeyValuePair<K, V>[] {
        return [...this.items];
    }
}
```

### 2. **Generic Types**

```typescript
// Generic type aliases
type Result<T, E = Error> = {
    success: true;
    data: T;
} | {
    success: false;
    error: E;
};

type ApiResponse<T> = {
    status: number;
    data: T;
    message: string;
};

// Usage
type UserResult = Result<User>;
type ProductsResponse = ApiResponse<Product[]>;

// Generic utility types
type Optional<T> = {
    [K in keyof T]?: T[K];
};

type Nullable<T> = {
    [K in keyof T]: T[K] | null;
};

type DeepReadonly<T> = {
    readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

interface User {
    id: number;
    name: string;
    email: string;
}

type OptionalUser = Optional<User>;      // { id?: number; name?: string; email?: string; }
type NullableUser = Nullable<User>;      // { id: number | null; name: string | null; email: string | null; }
type ReadonlyUser = DeepReadonly<User>;  // { readonly id: number; readonly name: string; readonly email: string; }
```

## 🏭 Generic Classes

### 1. **Basic Generic Classes**

```typescript
class Stack<T> {
    private items: T[] = [];
    
    push(item: T): void {
        this.items.push(item);
    }
    
    pop(): T | undefined {
        return this.items.pop();
    }
    
    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }
    
    isEmpty(): boolean {
        return this.items.length === 0;
    }
    
    size(): number {
        return this.items.length;
    }
    
    toArray(): T[] {
        return [...this.items];
    }
}

// การใช้งาน
const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.push(3);

console.log(numberStack.pop()); // 3
console.log(numberStack.peek()); // 2

const stringStack = new Stack<string>();
stringStack.push("hello");
stringStack.push("world");
```

### 2. **Advanced Generic Classes**

```typescript
// Generic class กับ constraints และ default types
class Repository<T extends { id: string | number }, K = string | number> {
    private items: Map<K, T> = new Map();
    
    save(item: T): void {
        this.items.set(item.id as K, item);
    }
    
    findById(id: K): T | undefined {
        return this.items.get(id);
    }
    
    findAll(): T[] {
        return Array.from(this.items.values());
    }
    
    update(id: K, updates: Partial<T>): T | undefined {
        const item = this.items.get(id);
        if (item) {
            const updatedItem = { ...item, ...updates };
            this.items.set(id, updatedItem);
            return updatedItem;
        }
        return undefined;
    }
    
    delete(id: K): boolean {
        return this.items.delete(id);
    }
    
    count(): number {
        return this.items.size;
    }
}

interface User {
    id: string;
    name: string;
    email: string;
}

interface Product {
    id: number;
    name: string;
    price: number;
}

const userRepository = new Repository<User, string>();
const productRepository = new Repository<Product, number>();

userRepository.save({ id: "1", name: "Alice", email: "alice@example.com" });
productRepository.save({ id: 1, name: "Laptop", price: 999 });
```

## 🔒 Generic Constraints

### 1. **Basic Constraints**

```typescript
// Constraint ด้วย extends
interface Lengthwise {
    length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

logLength("hello");        // ✅ string has length
logLength([1, 2, 3]);      // ✅ array has length
logLength({ length: 10 }); // ✅ object has length property
// logLength(123);         // ❌ Error: number doesn't have length

// Constraint ด้วย keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const person = { name: "Alice", age: 30, email: "alice@example.com" };
const name = getProperty(person, "name");    // string
const age = getProperty(person, "age");      // number
// const invalid = getProperty(person, "invalid"); // ❌ Error
```

### 2. **Advanced Constraints**

```typescript
// Multiple constraints
interface Serializable {
    serialize(): string;
}

interface Timestamped {
    timestamp: Date;
}

function processData<T extends Serializable & Timestamped>(data: T): string {
    return `${data.timestamp.toISOString()}: ${data.serialize()}`;
}

// Conditional constraints
type NonNullable<T> = T extends null | undefined ? never : T;

function assertExists<T>(value: T): NonNullable<T> {
    if (value === null || value === undefined) {
        throw new Error("Value must not be null or undefined");
    }
    return value as NonNullable<T>;
}

// Generic constraint with function
function findInArray<T>(
    array: T[], 
    predicate: (item: T) => boolean
): T | undefined {
    return array.find(predicate);
}

const users = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
];

const user = findInArray(users, u => u.id === 1);
```

## 🎭 Conditional Types

### 1. **Basic Conditional Types**

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;  // true
type Test2 = IsString<number>;  // false

// Conditional type กับ union
type ToArray<T> = T extends any ? T[] : never;
type StringOrNumberArray = ToArray<string | number>; // string[] | number[]

// Extract และ Exclude patterns
type ExtractStrings<T> = T extends string ? T : never;
type OnlyStrings = ExtractStrings<string | number | boolean>; // string

type ExcludeStrings<T> = T extends string ? never : T;
type NoStrings = ExcludeStrings<string | number | boolean>; // number | boolean
```

### 2. **Advanced Conditional Types**

```typescript
// Infer keyword
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getString(): string { return "hello"; }
function getNumber(): number { return 42; }

type StringReturn = ReturnType<typeof getString>; // string
type NumberReturn = ReturnType<typeof getNumber>; // number

// Parameters extraction
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function exampleFunction(a: string, b: number, c: boolean): void {}
type ExampleParams = Parameters<typeof exampleFunction>; // [string, number, boolean]

// Nested conditional types
type DeepFlatten<T> = T extends readonly (infer U)[]
    ? DeepFlatten<U>
    : T;

type Nested = string[][][];
type Flattened = DeepFlatten<Nested>; // string
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **Generic API Client**

```typescript
interface ApiEndpoint {
    path: string;
    method: "GET" | "POST" | "PUT" | "DELETE";
}

interface ApiEndpoints {
    getUser: ApiEndpoint;
    createUser: ApiEndpoint;
    updateUser: ApiEndpoint;
    deleteUser: ApiEndpoint;
}

type ApiMethod<T extends keyof ApiEndpoints> = ApiEndpoints[T]["method"];
type ApiPath<T extends keyof ApiEndpoints> = ApiEndpoints[T]["path"];

class ApiClient<TEndpoints extends Record<string, ApiEndpoint>> {
    constructor(private baseUrl: string, private endpoints: TEndpoints) {}
    
    async request<TEndpoint extends keyof TEndpoints, TResponse = any>(
        endpoint: TEndpoint,
        data?: any
    ): Promise<TResponse> {
        const config = this.endpoints[endpoint];
        const url = `${this.baseUrl}${config.path}`;
        
        const response = await fetch(url, {
            method: config.method,
            headers: {
                "Content-Type": "application/json",
            },
            body: data ? JSON.stringify(data) : undefined,
        });
        
        return response.json();
    }
}

// Usage
const endpoints: ApiEndpoints = {
    getUser: { path: "/users/:id", method: "GET" },
    createUser: { path: "/users", method: "POST" },
    updateUser: { path: "/users/:id", method: "PUT" },
    deleteUser: { path: "/users/:id", method: "DELETE" }
};

const apiClient = new ApiClient("https://api.example.com", endpoints);

// Type-safe API calls
interface User {
    id: string;
    name: string;
    email: string;
}

const user = await apiClient.request<"getUser", User>("getUser");
const newUser = await apiClient.request<"createUser", User>("createUser", {
    name: "Alice",
    email: "alice@example.com"
});
```

### 2. **Generic State Management**

```typescript
interface Action<T = any> {
    type: string;
    payload?: T;
}

type Reducer<TState, TAction extends Action> = (
    state: TState,
    action: TAction
) => TState;

class Store<TState, TAction extends Action> {
    private state: TState;
    private listeners: ((state: TState) => void)[] = [];
    
    constructor(
        private reducer: Reducer<TState, TAction>,
        initialState: TState
    ) {
        this.state = initialState;
    }
    
    getState(): TState {
        return this.state;
    }
    
    dispatch(action: TAction): void {
        this.state = this.reducer(this.state, action);
        this.listeners.forEach(listener => listener(this.state));
    }
    
    subscribe(listener: (state: TState) => void): () => void {
        this.listeners.push(listener);
        return () => {
            const index = this.listeners.indexOf(listener);
            if (index > -1) {
                this.listeners.splice(index, 1);
            }
        };
    }
}

// Usage example
interface CounterState {
    count: number;
}

type CounterAction = 
    | { type: "INCREMENT" }
    | { type: "DECREMENT" }
    | { type: "SET"; payload: number };

const counterReducer: Reducer<CounterState, CounterAction> = (state, action) => {
    switch (action.type) {
        case "INCREMENT":
            return { count: state.count + 1 };
        case "DECREMENT":
            return { count: state.count - 1 };
        case "SET":
            return { count: action.payload! };
        default:
            return state;
    }
};

const store = new Store(counterReducer, { count: 0 });

store.subscribe(state => console.log("State:", state));
store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "SET", payload: 10 });
```

### 3. **Generic Validation System**

```typescript
type ValidationResult = {
    isValid: boolean;
    errors: string[];
};

type Validator<T> = (value: T) => ValidationResult;

type ValidationSchema<T> = {
    [K in keyof T]?: Validator<T[K]>[];
};

class ValidationEngine<T> {
    constructor(private schema: ValidationSchema<T>) {}
    
    validate(data: T): ValidationResult {
        const errors: string[] = [];
        
        for (const [key, validators] of Object.entries(this.schema)) {
            const value = data[key as keyof T];
            const fieldValidators = validators as Validator<any>[];
            
            for (const validator of fieldValidators) {
                const result = validator(value);
                if (!result.isValid) {
                    errors.push(...result.errors.map(err => `${key}: ${err}`));
                }
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    }
}

// Validators
const required: Validator<any> = (value) => ({
    isValid: value !== null && value !== undefined && value !== "",
    errors: value !== null && value !== undefined && value !== "" ? [] : ["Field is required"]
});

const minLength = (min: number): Validator<string> => (value) => ({
    isValid: value && value.length >= min,
    errors: value && value.length >= min ? [] : [`Minimum length is ${min}`]
});

const email: Validator<string> = (value) => {
    const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
    return {
        isValid,
        errors: isValid ? [] : ["Invalid email format"]
    };
};

// Usage
interface UserForm {
    name: string;
    email: string;
    age: number;
}

const userValidationSchema: ValidationSchema<UserForm> = {
    name: [required, minLength(2)],
    email: [required, email],
    age: [required]
};

const validator = new ValidationEngine(userValidationSchema);

const result = validator.validate({
    name: "A",
    email: "invalid-email",
    age: 0
});

console.log(result); 
// { isValid: false, errors: ["name: Minimum length is 2", "email: Invalid email format"] }
```

## 📝 แบบฝึกหัด

### 1. **Generic Data Structures**
สร้าง generic data structures:
- `Queue<T>` - FIFO queue
- `LinkedList<T>` - Doubly linked list
- `Tree<T>` - Binary tree with traversal methods

### 2. **Type-safe ORM**
ออกแบบ generic ORM system:
- `Entity<T>` base class
- `Repository<T>` with CRUD operations
- Query builder กับ type safety

### 3. **Generic Event System**
สร้าง type-safe event system:
- Event definitions กับ payload types
- Event emitter/listener pattern
- Middleware support

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 7: คลาสและการสืบทอด](./07-classes-inheritance.md)

**บทต่อไป**: [บทที่ 9: Enums](./09-enums.md)

**กลับไปหน้าหลัก**: [README](./README.md)
