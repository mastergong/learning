# บทที่ 3: ชนิดข้อมูลพื้นฐาน

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ชนิดข้อมูลพื้นฐานใน TypeScript
- เข้าใจการประกาศและใช้งาน Type Annotations
- เข้าใจ Type Inference
- เรียนรู้ Union Types และ Literal Types

## 📊 ชนิดข้อมูลพื้นฐาน

### 1. **Boolean**
```typescript
let isDone: boolean = false;
let isActive: boolean = true;

// Type inference - TypeScript รู้ว่า isComplete เป็น boolean
let isComplete = true; // boolean
```

### 2. **Number**
```typescript
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;

// Floating point
let price: number = 99.99;
let discount: number = 0.1;

// Special values
let notANumber: number = NaN;
let infinity: number = Infinity;
```

### 3. **String**
```typescript
let color: string = "blue";
let fullName: string = 'Bob Bobbington';

// Template literals
let age: number = 37;
let sentence: string = `Hello, my name is ${fullName}. I'll be ${age + 1} years old next month.`;

// Multiline strings
let poem: string = `
    Roses are red,
    Violets are blue,
    TypeScript is awesome,
    And so are you!
`;
```

### 4. **Array**
```typescript
// วิธีที่ 1: Type[]
let list: number[] = [1, 2, 3];
let names: string[] = ["Alice", "Bob", "Charlie"];

// วิธีที่ 2: Array<Type>
let scores: Array<number> = [95, 87, 92];
let fruits: Array<string> = ["apple", "banana", "orange"];

// Mixed types ใช้ union types
let mixedArray: (string | number)[] = ["hello", 42, "world", 7];
```

### 5. **Tuple**
```typescript
// Tuple - array ที่มีจำนวนและประเภทที่กำหนดไว้
let person: [string, number] = ["Alice", 30];
let coordinate: [number, number] = [10, 20];

// การเข้าถึงข้อมูล
console.log(person[0]); // "Alice" (string)
console.log(person[1]); // 30 (number)

// Tuple กับ optional elements
let optionalTuple: [string, number?] = ["Bob"]; // อายุเป็น optional

// Tuple กับ rest elements
let restTuple: [string, ...number[]] = ["scores", 95, 87, 92, 88];
```

### 6. **Enum**
```typescript
// Basic enum
enum Color {
    Red,
    Green,
    Blue
}
let c: Color = Color.Green; // 1

// Enum กับ custom values
enum Status {
    Pending = "PENDING",
    Approved = "APPROVED",
    Rejected = "REJECTED"
}

// Numeric enum กับ custom start
enum Direction {
    Up = 1,
    Down,    // 2
    Left,    // 3
    Right    // 4
}

// Const enum (คอมไพล์เป็น inline values)
const enum ResponseStatus {
    Success = 200,
    NotFound = 404,
    ServerError = 500
}
```

### 7. **Any**
```typescript
// Any - ปิดการตรวจสอบ type (ไม่แนะนำให้ใช้บ่อย)
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean

// Array ของ any
let list: any[] = [1, true, "free"];
list[1] = 100;

// เมื่อไหร่ควรใช้ any
// - เมื่อ migrate จาก JavaScript
// - เมื่อทำงานกับ library ที่ไม่มี type definitions
// - เมื่อ type ซับซ้อนเกินไป (แต่ควรหา alternative)
```

### 8. **Unknown**
```typescript
// Unknown - type-safe ยิ่งกว่า any
let userInput: unknown;
let userName: string;

userInput = 5;
userInput = "Max";

// ไม่สามารถ assign unknown ไปยัง type อื่นได้โดยตรง
// userName = userInput; // Error!

// ต้อง type guard ก่อน
if (typeof userInput === "string") {
    userName = userInput; // ✅ OK
}
```

### 9. **Void**
```typescript
// Void - ไม่มีการ return ค่า
function logMessage(message: string): void {
    console.log(message);
    // ไม่มี return statement หรือ return;
}

// ตัวแปรประเภท void สามารถเป็น undefined หรือ null เท่านั้น
let unusable: void = undefined;
```

### 10. **Null และ Undefined**
```typescript
// Null และ undefined เป็น type ของตัวเอง
let n: null = null;
let u: undefined = undefined;

// ปกติแล้ว null และ undefined เป็น subtypes ของ type อื่นๆ
// แต่เมื่อใช้ --strictNullChecks จะแยกออกมา
let name: string = "Alice";
// name = null; // Error ถ้าใช้ strictNullChecks
```

### 11. **Never**
```typescript
// Never - ไม่เคยมีค่า return
function error(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {
        // infinite loop
    }
}

// Never เกิดขึ้นใน type narrowing
function processValue(value: string | number) {
    if (typeof value === "string") {
        // value เป็น string
        console.log(value.toUpperCase());
    } else if (typeof value === "number") {
        // value เป็น number
        console.log(value.toFixed(2));
    } else {
        // value เป็น never เพราะไม่มี case อื่น
        const exhaustiveCheck: never = value;
    }
}
```

### 12. **Object**
```typescript
// Object type
let obj: object = { name: "Alice", age: 30 };

// Better way - ใช้ interface หรือ type literal
let person: { name: string; age: number } = {
    name: "Bob",
    age: 25
};

// Object กับ optional properties
let user: { name: string; age?: number } = {
    name: "Charlie"
    // age เป็น optional
};
```

## 🔧 Type Annotations vs Type Inference

### Type Annotations - การประกาศ type อย่างชัดเจน
```typescript
let message: string = "Hello";
let count: number = 42;
let isActive: boolean = true;
```

### Type Inference - TypeScript คาดเดา type อัตโนมัติ
```typescript
let message = "Hello";        // string
let count = 42;               // number
let isActive = true;          // boolean
let numbers = [1, 2, 3];      // number[]
let mixed = [1, "hello"];     // (string | number)[]
```

## 🎨 Union Types
```typescript
// Union types - ตัวแปรที่สามารถเป็นได้หลาย type
let id: string | number;
id = "ABC123";    // ✅ OK
id = 123;         // ✅ OK
// id = true;     // ❌ Error

// Function กับ union types
function printId(id: string | number) {
    if (typeof id === "string") {
        console.log(`String ID: ${id.toUpperCase()}`);
    } else {
        console.log(`Number ID: ${id.toFixed(0)}`);
    }
}

// Array ของ union types
let items: (string | number)[] = ["apple", 1, "banana", 2];
```

## 🎯 Literal Types
```typescript
// String literal types
let alignment: "left" | "right" | "center";
alignment = "left";    // ✅ OK
// alignment = "top";  // ❌ Error

// Number literal types
let dice: 1 | 2 | 3 | 4 | 5 | 6;
dice = 3;    // ✅ OK
// dice = 7; // ❌ Error

// Boolean literal types
let success: true = true;
// success = false; // ❌ Error

// Combining with union types
type Theme = "light" | "dark";
type Size = "small" | "medium" | "large";

interface Button {
    theme: Theme;
    size: Size;
    disabled: boolean;
}
```

## 🔍 Type Assertions
```typescript
// Type assertion - บอก TypeScript ว่าเรารู้ type ที่แน่นอน
let someValue: unknown = "this is a string";

// วิธีที่ 1: angle bracket syntax (ไม่ใช้ใน JSX)
let strLength: number = (<string>someValue).length;

// วิธีที่ 2: as syntax (แนะนำ)
let strLength2: number = (someValue as string).length;

// ตัวอย่างการใช้งานจริง
const canvas = document.getElementById("myCanvas") as HTMLCanvasElement;
const context = canvas.getContext("2d");
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. User Profile System
```typescript
interface UserProfile {
    id: string | number;
    username: string;
    email: string;
    age?: number;
    isActive: boolean;
    roles: string[];
    metadata: {
        createdAt: string;
        lastLogin?: string;
    };
}

const user: UserProfile = {
    id: "user_123",
    username: "john_doe",
    email: "john@example.com",
    age: 28,
    isActive: true,
    roles: ["user", "editor"],
    metadata: {
        createdAt: "2023-01-15T10:00:00Z",
        lastLogin: "2023-07-20T14:30:00Z"
    }
};
```

### 2. API Response Handling
```typescript
type ApiStatus = "loading" | "success" | "error";

interface ApiResponse<T> {
    status: ApiStatus;
    data?: T;
    message?: string;
    timestamp: number;
}

// Usage
const userResponse: ApiResponse<UserProfile> = {
    status: "success",
    data: user,
    timestamp: Date.now()
};

const errorResponse: ApiResponse<null> = {
    status: "error",
    message: "User not found",
    timestamp: Date.now()
};
```

### 3. Configuration Object
```typescript
interface DatabaseConfig {
    host: string;
    port: number;
    database: string;
    username: string;
    password: string;
    ssl: boolean;
    timeout?: number;
    retries?: number;
}

const config: DatabaseConfig = {
    host: "localhost",
    port: 5432,
    database: "myapp",
    username: "admin",
    password: "secret123",
    ssl: true,
    timeout: 30000
};
```

## 🚨 ข้อผิดพลาดที่พบบ่อย

### 1. ใช้ any มากเกินไป
```typescript
// ❌ ไม่ดี
let data: any = fetchUserData();

// ✅ ดีกว่า
interface User {
    id: number;
    name: string;
}
let data: User = fetchUserData();
```

### 2. ไม่ระวัง null/undefined
```typescript
// ❌ อาจเกิด runtime error
function getUserName(user: { name?: string }) {
    return user.name.toUpperCase(); // Error ถ้า name เป็น undefined
}

// ✅ ปลอดภัย
function getUserName(user: { name?: string }) {
    return user.name?.toUpperCase() ?? "Unknown";
}
```

### 3. Type assertion ไม่ถูกต้อง
```typescript
// ❌ อันตราย
const user = {} as UserProfile; // user ไม่มี properties ที่จำเป็น

// ✅ ปลอดภัย
const user: UserProfile = {
    id: 1,
    username: "john",
    email: "john@example.com",
    isActive: true,
    roles: [],
    metadata: {
        createdAt: new Date().toISOString()
    }
};
```

## 📝 แบบฝึกหัด

### 1. **Basic Types**
สร้างตัวแปรด้วย type annotations:
- ชื่อสินค้า (string)
- ราคา (number)  
- มีในสต็อก (boolean)
- หมวดหมู่ (array of strings)
- คะแนน (tuple ของ [string, number])

### 2. **Union Types และ Literal Types**
สร้าง type สำหรับ:
- สถานะคำสั่งซื้อ: "pending" | "processing" | "shipped" | "delivered"
- ขนาดเสื้อผ้า: "XS" | "S" | "M" | "L" | "XL"
- ระดับความสำคัญ: 1 | 2 | 3 | 4 | 5

### 3. **Complex Object**
สร้าง interface สำหรับระบบห้องสมุด:
- หนังสือ (id, title, author, year, available)
- สมาชิก (id, name, email, borrowed books)
- การยืม (book id, member id, borrow date, return date?)

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 2: การติดตั้งและการตั้งค่า](./02-installation-setup.md)

**บทต่อไป**: [บทที่ 4: ตัวแปรและการประกาศ](./04-variables-declarations.md)

**กลับไปหน้าหลัก**: [README](./README.md)
