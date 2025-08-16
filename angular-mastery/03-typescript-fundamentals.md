# บทที่ 3: TypeScript Fundamentals

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจ TypeScript และความแตกต่างจาก JavaScript
- ใช้ Type System ของ TypeScript ได้อย่างถูกต้อง
- เขียน Classes, Interfaces, และ Modules ใน TypeScript
- ใช้ Modern JavaScript Features กับ TypeScript
- เตรียมพร้อมสำหรับ Angular development ด้วย TypeScript

## 📚 TypeScript คืออะไร?

TypeScript เป็น **statically typed superset** ของ JavaScript ที่พัฒนาโดย Microsoft

### 🌟 ข้อดีของ TypeScript

#### **1. Static Type Checking**
```typescript
// JavaScript - Runtime Error
function add(a, b) {
  return a + b;
}

add("hello", "world"); // "helloworld" - อาจไม่ใช่สิ่งที่เราต้องการ
add(1, "2");           // "12" - Type coercion

// TypeScript - Compile-time Error
function add(a: number, b: number): number {
  return a + b;
}

add("hello", "world"); // ❌ Error: Argument of type 'string' is not assignable to parameter of type 'number'
add(1, 2);             // ✅ 3
```

#### **2. Better IDE Support**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

function getUserInfo(user: User) {
  // IDE จะแสดง autocomplete สำหรับ user.
  return `${user.name} (${user.email})`;
}
```

#### **3. Modern JavaScript Features**
```typescript
// ES6+ features พร้อม type safety
const users: User[] = [
  { id: 1, name: "John", email: "john@example.com", isActive: true },
  { id: 2, name: "Jane", email: "jane@example.com", isActive: false }
];

// Arrow functions
const activeUsers = users.filter((user: User) => user.isActive);

// Destructuring
const { name, email } = users[0];

// Template literals
const message = `Welcome ${name}!`;
```

## 🔢 Basic Types

### **Primitive Types**
```typescript
// Boolean
let isCompleted: boolean = false;
let hasError: boolean = true;

// Number
let age: number = 25;
let price: number = 99.99;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// String
let firstName: string = "John";
let lastName: string = 'Doe';
let fullName: string = `${firstName} ${lastName}`;

// Null and Undefined
let nullValue: null = null;
let undefinedValue: undefined = undefined;
```

### **Array Types**
```typescript
// Array of numbers
let numbers: number[] = [1, 2, 3, 4, 5];
let numbers2: Array<number> = [1, 2, 3, 4, 5];

// Array of strings
let names: string[] = ["John", "Jane", "Bob"];
let names2: Array<string> = ["John", "Jane", "Bob"];

// Mixed array (union type)
let mixedArray: (string | number)[] = ["John", 25, "Jane", 30];

// Array of objects
let users: User[] = [
  { id: 1, name: "John", email: "john@example.com", isActive: true }
];
```

### **Tuple Types**
```typescript
// Tuple - fixed length array with specific types
let userInfo: [number, string, boolean] = [1, "John", true];

// Named tuple
type UserTuple = [id: number, name: string, isActive: boolean];
let user: UserTuple = [1, "John", true];

// Optional tuple elements
let coordinate: [number, number, number?] = [10, 20]; // z is optional
```

### **Enum Types**
```typescript
// Numeric enum
enum UserRole {
  Admin,    // 0
  User,     // 1
  Guest     // 2
}

let role: UserRole = UserRole.Admin;
console.log(role); // 0

// String enum
enum Colors {
  Red = "red",
  Green = "green",
  Blue = "blue"
}

let favoriteColor: Colors = Colors.Blue;

// Mixed enum
enum Status {
  Pending = "pending",
  InProgress = 1,
  Completed = 2,
  Failed = "failed"
}
```

### **Object Types**
```typescript
// Object type annotation
let user: { id: number; name: string; email: string } = {
  id: 1,
  name: "John",
  email: "john@example.com"
};

// Optional properties
let config: { host: string; port?: number; ssl?: boolean } = {
  host: "localhost"
  // port and ssl are optional
};

// Readonly properties
let immutableUser: { readonly id: number; name: string } = {
  id: 1,
  name: "John"
};
// immutableUser.id = 2; // ❌ Error: Cannot assign to 'id' because it is a read-only property
```

## 🏗️ Interfaces

### **Basic Interface**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

// Implementing interface
const user: User = {
  id: 1,
  name: "John Doe",
  email: "john@example.com",
  isActive: true
};

function createUser(userData: User): User {
  return { ...userData };
}
```

### **Optional Properties**
```typescript
interface CreateUserRequest {
  name: string;
  email: string;
  age?: number;          // Optional
  phone?: string;        // Optional
}

const newUser: CreateUserRequest = {
  name: "Jane",
  email: "jane@example.com"
  // age and phone are optional
};
```

### **Readonly Properties**
```typescript
interface ReadonlyUser {
  readonly id: number;
  readonly createdAt: Date;
  name: string;
  email: string;
}

const user: ReadonlyUser = {
  id: 1,
  createdAt: new Date(),
  name: "John",
  email: "john@example.com"
};

// user.id = 2; // ❌ Error: Cannot assign to 'id' because it is a read-only property
user.name = "Jane"; // ✅ OK
```

### **Method Signatures**
```typescript
interface UserService {
  // Method signatures
  getUser(id: number): User;
  createUser(user: CreateUserRequest): User;
  updateUser(id: number, updates: Partial<User>): User;
  deleteUser(id: number): boolean;
}

// Function type property
interface Calculator {
  add: (a: number, b: number) => number;
  subtract: (a: number, b: number) => number;
}

const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};
```

### **Index Signatures**
```typescript
interface StringDictionary {
  [key: string]: string;
}

const translations: StringDictionary = {
  hello: "สวัสดี",
  goodbye: "ลาก่อน",
  thank_you: "ขอบคุณ"
};

// Number index
interface NumberArray {
  [index: number]: string;
}

const fruits: NumberArray = ["apple", "banana", "orange"];
```

### **Extending Interfaces**
```typescript
interface BasicUser {
  id: number;
  name: string;
  email: string;
}

interface AdminUser extends BasicUser {
  role: "admin";
  permissions: string[];
  lastLogin: Date;
}

interface SuperAdminUser extends AdminUser {
  canDeleteUsers: boolean;
  systemAccess: boolean;
}

const admin: AdminUser = {
  id: 1,
  name: "Admin User",
  email: "admin@example.com",
  role: "admin",
  permissions: ["create", "read", "update", "delete"],
  lastLogin: new Date()
};
```

## 🎭 Classes

### **Basic Class**
```typescript
class User {
  // Properties
  id: number;
  name: string;
  email: string;
  private _password: string;
  
  // Constructor
  constructor(id: number, name: string, email: string, password: string) {
    this.id = id;
    this.name = name;
    this.email = email;
    this._password = password;
  }
  
  // Methods
  getFullInfo(): string {
    return `${this.name} (${this.email})`;
  }
  
  private validatePassword(password: string): boolean {
    return password.length >= 8;
  }
  
  changePassword(newPassword: string): boolean {
    if (this.validatePassword(newPassword)) {
      this._password = newPassword;
      return true;
    }
    return false;
  }
}

const user = new User(1, "John Doe", "john@example.com", "password123");
console.log(user.getFullInfo()); // "John Doe (john@example.com)"
```

### **Access Modifiers**
```typescript
class BankAccount {
  public accountNumber: string;        // Public (default)
  protected balance: number;           // Protected - accessible in subclasses
  private pin: string;                 // Private - only in this class
  readonly createdAt: Date;           // Readonly - cannot be modified
  
  constructor(accountNumber: string, initialBalance: number, pin: string) {
    this.accountNumber = accountNumber;
    this.balance = initialBalance;
    this.pin = pin;
    this.createdAt = new Date();
  }
  
  public getBalance(): number {
    return this.balance;
  }
  
  protected updateBalance(amount: number): void {
    this.balance += amount;
  }
  
  private validatePin(enteredPin: string): boolean {
    return this.pin === enteredPin;
  }
}

class SavingsAccount extends BankAccount {
  private interestRate: number;
  
  constructor(accountNumber: string, initialBalance: number, pin: string, interestRate: number) {
    super(accountNumber, initialBalance, pin);
    this.interestRate = interestRate;
  }
  
  addInterest(): void {
    const interest = this.balance * this.interestRate / 100;
    this.updateBalance(interest); // ✅ Can access protected method
    // this.pin = "new pin"; // ❌ Error: Property 'pin' is private
  }
}
```

### **Parameter Properties**
```typescript
// Shorthand syntax
class Product {
  constructor(
    public id: number,
    public name: string,
    private price: number,
    protected category: string
  ) {
    // Properties are automatically created and assigned
  }
  
  getPrice(): number {
    return this.price;
  }
}

// Equivalent to:
class ProductLongform {
  public id: number;
  public name: string;
  private price: number;
  protected category: string;
  
  constructor(id: number, name: string, price: number, category: string) {
    this.id = id;
    this.name = name;
    this.price = price;
    this.category = category;
  }
}
```

### **Abstract Classes**
```typescript
abstract class Animal {
  protected name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  // Concrete method
  move(): void {
    console.log(`${this.name} is moving`);
  }
  
  // Abstract method - must be implemented by subclasses
  abstract makeSound(): void;
  abstract eat(food: string): void;
}

class Dog extends Animal {
  constructor(name: string) {
    super(name);
  }
  
  makeSound(): void {
    console.log(`${this.name} barks: Woof! Woof!`);
  }
  
  eat(food: string): void {
    console.log(`${this.name} is eating ${food}`);
  }
}

class Cat extends Animal {
  constructor(name: string) {
    super(name);
  }
  
  makeSound(): void {
    console.log(`${this.name} meows: Meow!`);
  }
  
  eat(food: string): void {
    console.log(`${this.name} is eating ${food} elegantly`);
  }
}

// const animal = new Animal("Generic"); // ❌ Error: Cannot create an instance of an abstract class
const dog = new Dog("Buddy");
const cat = new Cat("Whiskers");
```

### **Implementing Interfaces**
```typescript
interface Flyable {
  altitude: number;
  fly(): void;
  land(): void;
}

interface Swimmable {
  depth: number;
  swim(): void;
  dive(): void;
}

class Duck implements Flyable, Swimmable {
  altitude: number = 0;
  depth: number = 0;
  
  constructor(public name: string) {}
  
  fly(): void {
    this.altitude = 100;
    console.log(`${this.name} is flying at ${this.altitude} feet`);
  }
  
  land(): void {
    this.altitude = 0;
    console.log(`${this.name} has landed`);
  }
  
  swim(): void {
    this.depth = 5;
    console.log(`${this.name} is swimming at ${this.depth} feet deep`);
  }
  
  dive(): void {
    this.depth = 20;
    console.log(`${this.name} is diving to ${this.depth} feet`);
  }
}
```

## 🔧 Functions และ Methods

### **Function Types**
```typescript
// Function declaration
function add(a: number, b: number): number {
  return a + b;
}

// Function expression
const multiply = function(a: number, b: number): number {
  return a * b;
};

// Arrow function
const divide = (a: number, b: number): number => {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
};

// One-line arrow function
const square = (x: number): number => x * x;
```

### **Optional และ Default Parameters**
```typescript
// Optional parameters
function greet(name: string, greeting?: string): string {
  return `${greeting || "Hello"}, ${name}!`;
}

greet("John");           // "Hello, John!"
greet("Jane", "Hi");     // "Hi, Jane!"

// Default parameters
function createUser(name: string, role: string = "user", isActive: boolean = true): User {
  return {
    id: Math.random(),
    name,
    role,
    isActive
  };
}

createUser("John");                    // role: "user", isActive: true
createUser("Jane", "admin");           // role: "admin", isActive: true
createUser("Bob", "user", false);      // role: "user", isActive: false
```

### **Rest Parameters**
```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3);        // 6
sum(1, 2, 3, 4, 5);  // 15

function logMessages(mainMessage: string, ...additionalMessages: string[]): void {
  console.log(`Main: ${mainMessage}`);
  additionalMessages.forEach((msg, index) => {
    console.log(`${index + 1}: ${msg}`);
  });
}

logMessages("Error occurred", "Check logs", "Contact admin", "Restart server");
```

### **Function Overloading**
```typescript
// Function overload signatures
function parseValue(value: string): string;
function parseValue(value: number): number;
function parseValue(value: boolean): boolean;

// Implementation signature
function parseValue(value: string | number | boolean): string | number | boolean {
  if (typeof value === "string") {
    return value.trim().toLowerCase();
  } else if (typeof value === "number") {
    return Math.round(value);
  } else {
    return value;
  }
}

const str = parseValue("  HELLO  ");  // string
const num = parseValue(3.14159);      // number
const bool = parseValue(true);        // boolean
```

## 🎯 Generic Types

### **Basic Generics**
```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

const numberResult = identity<number>(42);        // number
const stringResult = identity<string>("hello");   // string
const boolResult = identity(true);                // boolean (type inferred)

// Generic array function
function getFirstElement<T>(array: T[]): T | undefined {
  return array.length > 0 ? array[0] : undefined;
}

const firstNumber = getFirstElement([1, 2, 3]);       // number | undefined
const firstName = getFirstElement(["a", "b", "c"]);   // string | undefined
```

### **Generic Interfaces**
```typescript
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

class NumberContainer implements Container<number> {
  constructor(public value: number) {}
  
  getValue(): number {
    return this.value;
  }
  
  setValue(value: number): void {
    this.value = value;
  }
}

class StringContainer implements Container<string> {
  constructor(public value: string) {}
  
  getValue(): string {
    return this.value;
  }
  
  setValue(value: string): void {
    this.value = value;
  }
}
```

### **Generic Classes**
```typescript
class Repository<T> {
  private items: T[] = [];
  
  add(item: T): void {
    this.items.push(item);
  }
  
  getAll(): T[] {
    return [...this.items];
  }
  
  findById(id: number): T | undefined {
    return this.items.find((item: any) => item.id === id);
  }
  
  update(id: number, updates: Partial<T>): T | undefined {
    const index = this.items.findIndex((item: any) => item.id === id);
    if (index !== -1) {
      this.items[index] = { ...this.items[index], ...updates };
      return this.items[index];
    }
    return undefined;
  }
  
  delete(id: number): boolean {
    const index = this.items.findIndex((item: any) => item.id === id);
    if (index !== -1) {
      this.items.splice(index, 1);
      return true;
    }
    return false;
  }
}

// Usage
const userRepository = new Repository<User>();
const productRepository = new Repository<Product>();

userRepository.add({ id: 1, name: "John", email: "john@example.com" });
productRepository.add({ id: 1, name: "iPhone", price: 999 });
```

### **Generic Constraints**
```typescript
// Constraint with extends
interface HasId {
  id: number;
}

function updateItem<T extends HasId>(item: T, updates: Partial<T>): T {
  return { ...item, ...updates };
}

const user = { id: 1, name: "John", email: "john@example.com" };
const updatedUser = updateItem(user, { name: "Jane" }); // ✅ OK

// const invalidItem = { name: "Test" };
// updateItem(invalidItem, { name: "Updated" }); // ❌ Error: missing 'id'

// Multiple constraints
interface HasName {
  name: string;
}

function logNameAndId<T extends HasId & HasName>(item: T): void {
  console.log(`Item: ${item.name} (ID: ${item.id})`);
}
```

### **Utility Types**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  isActive: boolean;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;
const userUpdates: PartialUser = { name: "Jane" };

// Required - makes all properties required
type RequiredUser = Required<User>;

// Pick - select specific properties
type UserSummary = Pick<User, "id" | "name" | "email">;
const summary: UserSummary = { id: 1, name: "John", email: "john@example.com" };

// Omit - exclude specific properties
type CreateUserData = Omit<User, "id">;
const newUserData: CreateUserData = {
  name: "Jane",
  email: "jane@example.com",
  age: 25,
  isActive: true
};

// Record - create type with specific keys and value type
type UserRoles = Record<string, string>;
const roles: UserRoles = {
  admin: "Administrator",
  user: "Regular User",
  guest: "Guest User"
};

// Readonly - makes all properties readonly
type ReadonlyUser = Readonly<User>;
const immutableUser: ReadonlyUser = {
  id: 1,
  name: "John",
  email: "john@example.com",
  age: 30,
  isActive: true
};
// immutableUser.name = "Jane"; // ❌ Error
```

## 🔗 Union และ Intersection Types

### **Union Types**
```typescript
// Union type
type Status = "pending" | "approved" | "rejected";
type ID = string | number;

function processId(id: ID): string {
  if (typeof id === "string") {
    return id.toUpperCase();
  } else {
    return id.toString();
  }
}

// Union with objects
interface Bird {
  type: "bird";
  wingspan: number;
  fly(): void;
}

interface Fish {
  type: "fish";
  finCount: number;
  swim(): void;
}

type Animal = Bird | Fish;

function moveAnimal(animal: Animal): void {
  // Type guard with discriminated union
  if (animal.type === "bird") {
    animal.fly(); // TypeScript knows this is Bird
  } else {
    animal.swim(); // TypeScript knows this is Fish
  }
}
```

### **Intersection Types**
```typescript
interface HasName {
  name: string;
}

interface HasAge {
  age: number;
}

interface HasEmail {
  email: string;
}

// Intersection type
type Person = HasName & HasAge & HasEmail;

const person: Person = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

// Intersection with additional properties
type Employee = Person & {
  employeeId: string;
  department: string;
  salary: number;
};

const employee: Employee = {
  name: "Jane",
  age: 28,
  email: "jane@example.com",
  employeeId: "EMP001",
  department: "Engineering",
  salary: 75000
};
```

## 🎪 Advanced Types

### **Type Guards**
```typescript
// typeof type guard
function processValue(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // value is string
  } else {
    return value.toString(); // value is number
  }
}

// instanceof type guard
class Dog {
  bark(): void {
    console.log("Woof!");
  }
}

class Cat {
  meow(): void {
    console.log("Meow!");
  }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark(); // animal is Dog
  } else {
    animal.meow(); // animal is Cat
  }
}

// Custom type guard
interface Fish {
  swim(): void;
}

interface Bird {
  fly(): void;
}

// Type predicate
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function movePet(pet: Fish | Bird): void {
  if (isFish(pet)) {
    pet.swim(); // pet is Fish
  } else {
    pet.fly(); // pet is Bird
  }
}
```

### **Mapped Types**
```typescript
// Basic mapped type
type ReadonlyVersion<T> = {
  readonly [K in keyof T]: T[K];
};

type User = {
  id: number;
  name: string;
  email: string;
};

type ReadonlyUser = ReadonlyVersion<User>;
// Result: { readonly id: number; readonly name: string; readonly email: string; }

// Optional mapped type
type OptionalVersion<T> = {
  [K in keyof T]?: T[K];
};

type PartialUser = OptionalVersion<User>;
// Result: { id?: number; name?: string; email?: string; }

// Conditional mapped type
type NonNullable<T> = {
  [K in keyof T]: T[K] extends null | undefined ? never : T[K];
};
```

### **Conditional Types**
```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;  // true
type Test2 = IsString<number>;  // false

// More complex conditional type
type ApiResponse<T> = T extends string
  ? { message: T }
  : T extends number
  ? { code: T }
  : { data: T };

type StringResponse = ApiResponse<string>;  // { message: string }
type NumberResponse = ApiResponse<number>;  // { code: number }
type ObjectResponse = ApiResponse<User>;    // { data: User }

// Conditional type with infer
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type FunctionReturn = ReturnType<() => string>;        // string
type MethodReturn = ReturnType<(x: number) => number>; // number
```

## 🏭 Modules และ Namespaces

### **ES6 Modules**
```typescript
// user.model.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export class UserService {
  private users: User[] = [];
  
  addUser(user: User): void {
    this.users.push(user);
  }
  
  getUsers(): User[] {
    return this.users;
  }
}

export const DEFAULT_USER_ROLE = "user";

// math.utils.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export default function calculate(operation: string, a: number, b: number): number {
  switch (operation) {
    case "add": return add(a, b);
    case "multiply": return multiply(a, b);
    default: throw new Error("Unknown operation");
  }
}
```

### **Importing Modules**
```typescript
// app.ts
import { User, UserService, DEFAULT_USER_ROLE } from "./user.model";
import calculate, { add, multiply } from "./math.utils";
import * as MathUtils from "./math.utils";

// Using imports
const userService = new UserService();
const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com"
};

userService.addUser(user);
console.log(DEFAULT_USER_ROLE);

const result1 = add(5, 3);
const result2 = calculate("multiply", 4, 6);
const result3 = MathUtils.multiply(2, 8);
```

### **Re-exports**
```typescript
// models/index.ts
export { User, UserService } from "./user.model";
export { Product, ProductService } from "./product.model";
export { Order, OrderService } from "./order.model";

// utils/index.ts
export * from "./math.utils";
export * from "./string.utils";
export * from "./date.utils";

// app.ts
import { User, Product, Order } from "./models";
import { add, formatCurrency, formatDate } from "./utils";
```

### **Namespaces (Legacy)**
```typescript
// shapes.ts
namespace Shapes {
  export interface Point {
    x: number;
    y: number;
  }
  
  export class Circle {
    constructor(public center: Point, public radius: number) {}
    
    area(): number {
      return Math.PI * this.radius * this.radius;
    }
  }
  
  export class Rectangle {
    constructor(
      public topLeft: Point,
      public width: number,
      public height: number
    ) {}
    
    area(): number {
      return this.width * this.height;
    }
  }
}

// Usage
const point: Shapes.Point = { x: 0, y: 0 };
const circle = new Shapes.Circle(point, 5);
const rectangle = new Shapes.Rectangle(point, 10, 20);
```

## 🎯 Decorators (Preview)

### **Class Decorators**
```typescript
// Class decorator
function Component(config: { selector: string; template: string }) {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      selector = config.selector;
      template = config.template;
    };
  };
}

@Component({
  selector: "app-user",
  template: "<div>User Component</div>"
})
class UserComponent {
  name: string = "User";
}
```

### **Method Decorators**
```typescript
// Method decorator
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function (...args: any[]) {
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
```

### **Property Decorators**
```typescript
// Property decorator
function Required(target: any, propertyKey: string) {
  let value: any;
  
  const getter = () => {
    return value;
  };
  
  const setter = (newValue: any) => {
    if (newValue === null || newValue === undefined) {
      throw new Error(`${propertyKey} is required`);
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

class User {
  @Required
  public name: string;
  
  @Required
  public email: string;
  
  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }
}
```

## 🧪 แบบฝึกหัด

### **Exercise 1: Basic Types**
```typescript
// สร้าง types สำหรับระบบ E-commerce
interface Product {
  // TODO: Define product interface
}

interface Customer {
  // TODO: Define customer interface
}

interface Order {
  // TODO: Define order interface
}

// TODO: Create enum for order status
enum OrderStatus {
  // Define status values
}

// TODO: Create functions to handle orders
function createOrder(customer: Customer, products: Product[]): Order {
  // Implementation
}

function calculateTotal(order: Order): number {
  // Implementation
}
```

### **Exercise 2: Classes และ Inheritance**
```typescript
// สร้าง class hierarchy สำหรับพนักงาน
abstract class Employee {
  // TODO: Define abstract employee class
}

class FullTimeEmployee extends Employee {
  // TODO: Implement full-time employee
}

class ContractEmployee extends Employee {
  // TODO: Implement contract employee
}

interface Payable {
  // TODO: Define payable interface
}
```

### **Exercise 3: Generics**
```typescript
// สร้าง Generic Repository pattern
interface Repository<T> {
  // TODO: Define repository interface
}

class InMemoryRepository<T> implements Repository<T> {
  // TODO: Implement in-memory repository
}

// TODO: Create specific repositories
class UserRepository extends InMemoryRepository<User> {
  // Additional user-specific methods
}
```

## 🧪 Quiz

### **Question 1**
TypeScript คือ:
- a) เป็น JavaScript framework
- b) เป็น superset ของ JavaScript ที่เพิ่ม static typing
- c) เป็นภาษาโปรแกรมใหม่ที่แยกจาก JavaScript
- d) เป็น runtime environment สำหรับ JavaScript

### **Question 2**
Interface ใน TypeScript ใช้สำหรับ:
- a) สร้าง object instances
- b) กำหนด structure ของ object types
- c) เขียน business logic
- d) จัดการ dependencies

### **Question 3**
Generic ใน TypeScript ช่วยให้:
- a) เขียน code ที่ type-safe และ reusable
- b) ทำให้ code รันเร็วขึ้น
- c) ลดขนาดของ bundle
- d) เขียน CSS ได้

**คำตอบ: 1-b, 2-b, 3-a**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🔤 Type System**
1. **Basic Types** - primitive types, arrays, tuples, enums
2. **Object Types** - interfaces, optional properties, readonly
3. **Union และ Intersection** - combining types
4. **Generic Types** - reusable type-safe code

### **🏗️ Object-Oriented Programming**
1. **Classes** - properties, methods, constructors
2. **Inheritance** - extending classes และ implementing interfaces
3. **Access Modifiers** - public, private, protected
4. **Abstract Classes** - base classes with abstract methods

### **🔧 Advanced Features**
1. **Type Guards** - runtime type checking
2. **Mapped Types** - transforming types
3. **Conditional Types** - type-level conditionals
4. **Decorators** - metadata และ code modification

### **📦 Module System**
1. **ES6 Modules** - import/export
2. **Re-exports** - organizing exports
3. **Namespaces** - grouping related code

### **🎯 Angular Preparation**
ความรู้ TypeScript นี้จะเป็นพื้นฐานสำคัญสำหรับ:
- Angular Components และ Services
- Dependency Injection
- Type-safe HTTP clients
- Reactive programming กับ RxJS

---

**🎉 ยินดีด้วย! ตอนนี้คุณพร้อมที่จะเริ่มสร้าง Angular applications แล้ว**

**[⬅️ บทที่ 2: Development Environment Setup](./02-development-environment-setup.md) | [➡️ บทที่ 4: First Angular Application](./04-first-angular-application.md)**
