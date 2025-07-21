# บทที่ 7: คลาสและการสืบทอด

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจการสร้างและใช้งาน Classes ใน TypeScript
- เรียนรู้ Access Modifiers (public, private, protected)
- เข้าใจการสืบทอด (Inheritance) และ Abstract Classes
- เรียนรู้ Static Members และ Getters/Setters

## 🏗️ การสร้าง Classes

### 1. **Basic Class**

```typescript
class Person {
    // Properties
    name: string;
    age: number;
    
    // Constructor
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
    
    // Methods
    greet(): string {
        return `Hello, I'm ${this.name} and I'm ${this.age} years old.`;
    }
    
    celebrateBirthday(): void {
        this.age++;
        console.log(`Happy birthday! Now I'm ${this.age} years old.`);
    }
}

// การใช้งาน
const person = new Person("Alice", 30);
console.log(person.greet()); // "Hello, I'm Alice and I'm 30 years old."
person.celebrateBirthday(); // "Happy birthday! Now I'm 31 years old."
```

### 2. **Constructor Shorthand**

```typescript
// แบบปกติ
class User {
    id: number;
    name: string;
    email: string;
    
    constructor(id: number, name: string, email: string) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
}

// แบบ shorthand
class UserShorthand {
    constructor(
        public id: number,
        public name: string,
        public email: string
    ) {}
    
    getInfo(): string {
        return `${this.name} (${this.email})`;
    }
}
```

## 🔒 Access Modifiers

### 1. **Public, Private, Protected**

```typescript
class BankAccount {
    public accountNumber: string;      // เข้าถึงได้จากทุกที่
    private balance: number;           // เข้าถึงได้เฉพาะใน class นี้
    protected ownerName: string;       // เข้าถึงได้ใน class และ subclass
    
    constructor(accountNumber: string, ownerName: string, initialBalance: number = 0) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = initialBalance;
    }
    
    // Public method
    public deposit(amount: number): void {
        if (amount > 0) {
            this.balance += amount;
            console.log(`Deposited ${amount}. New balance: ${this.balance}`);
        }
    }
    
    public withdraw(amount: number): boolean {
        if (amount > 0 && amount <= this.balance) {
            this.balance -= amount;
            console.log(`Withdrew ${amount}. New balance: ${this.balance}`);
            return true;
        }
        console.log("Insufficient funds or invalid amount");
        return false;
    }
    
    // Public getter สำหรับ private property
    public getBalance(): number {
        return this.balance;
    }
    
    // Protected method
    protected validateTransaction(amount: number): boolean {
        return amount > 0 && amount <= this.balance;
    }
    
    // Private method
    private logTransaction(type: string, amount: number): void {
        console.log(`Transaction: ${type} ${amount} at ${new Date()}`);
    }
}

const account = new BankAccount("123-456-789", "John Doe", 1000);
console.log(account.accountNumber); // ✅ OK - public
// console.log(account.balance);    // ❌ Error - private
// console.log(account.ownerName);  // ❌ Error - protected
```

### 2. **Readonly Properties**

```typescript
class ImmutableUser {
    readonly id: string;
    readonly createdAt: Date;
    name: string;
    
    constructor(id: string, name: string) {
        this.id = id;
        this.name = name;
        this.createdAt = new Date();
    }
    
    updateName(newName: string): void {
        this.name = newName; // ✅ OK
        // this.id = "new-id"; // ❌ Error - readonly
    }
}
```

## 🧬 การสืบทอด (Inheritance)

### 1. **Basic Inheritance**

```typescript
// Base class
class Animal {
    protected name: string;
    protected age: number;
    
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
    
    makeSound(): string {
        return "Some generic animal sound";
    }
    
    getInfo(): string {
        return `${this.name} is ${this.age} years old`;
    }
}

// Derived class
class Dog extends Animal {
    private breed: string;
    
    constructor(name: string, age: number, breed: string) {
        super(name, age); // เรียก constructor ของ parent class
        this.breed = breed;
    }
    
    // Override method
    makeSound(): string {
        return "Woof! Woof!";
    }
    
    // Additional method
    fetch(): string {
        return `${this.name} is fetching the ball!`;
    }
    
    // Override และเพิ่มข้อมูล
    getInfo(): string {
        return `${super.getInfo()} and is a ${this.breed}`;
    }
}

class Cat extends Animal {
    private isIndoor: boolean;
    
    constructor(name: string, age: number, isIndoor: boolean = true) {
        super(name, age);
        this.isIndoor = isIndoor;
    }
    
    makeSound(): string {
        return "Meow!";
    }
    
    purr(): string {
        return `${this.name} is purring contentedly`;
    }
}

// การใช้งาน
const dog = new Dog("Buddy", 3, "Golden Retriever");
const cat = new Cat("Whiskers", 2, true);

console.log(dog.makeSound()); // "Woof! Woof!"
console.log(cat.makeSound()); // "Meow!"
console.log(dog.getInfo());   // "Buddy is 3 years old and is a Golden Retriever"
```

### 2. **Abstract Classes**

```typescript
// Abstract base class
abstract class Shape {
    protected name: string;
    
    constructor(name: string) {
        this.name = name;
    }
    
    // Abstract method - ต้อง implement ใน derived class
    abstract calculateArea(): number;
    abstract calculatePerimeter(): number;
    
    // Concrete method
    displayInfo(): string {
        return `Shape: ${this.name}, Area: ${this.calculateArea()}`;
    }
}

class Rectangle extends Shape {
    constructor(
        private width: number,
        private height: number
    ) {
        super("Rectangle");
    }
    
    calculateArea(): number {
        return this.width * this.height;
    }
    
    calculatePerimeter(): number {
        return 2 * (this.width + this.height);
    }
}

class Circle extends Shape {
    constructor(private radius: number) {
        super("Circle");
    }
    
    calculateArea(): number {
        return Math.PI * this.radius * this.radius;
    }
    
    calculatePerimeter(): number {
        return 2 * Math.PI * this.radius;
    }
}

// การใช้งาน
const rectangle = new Rectangle(5, 3);
const circle = new Circle(4);

console.log(rectangle.displayInfo()); // "Shape: Rectangle, Area: 15"
console.log(circle.displayInfo());    // "Shape: Circle, Area: 50.26548245743669"

// const shape = new Shape("Generic"); // ❌ Error - ไม่สามารถสร้าง instance ของ abstract class
```

## ⚡ Static Members

```typescript
class MathUtils {
    static readonly PI = 3.14159;
    private static instanceCount = 0;
    
    // Static method
    static calculateCircleArea(radius: number): number {
        return this.PI * radius * radius;
    }
    
    static getInstanceCount(): number {
        return this.instanceCount;
    }
    
    // Instance method
    constructor() {
        MathUtils.instanceCount++;
    }
    
    // Static property กับ accessor
    static get version(): string {
        return "1.0.0";
    }
}

// การใช้งาน static members
console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.calculateCircleArea(5)); // 78.53975
console.log(MathUtils.version); // "1.0.0"

const math1 = new MathUtils();
const math2 = new MathUtils();
console.log(MathUtils.getInstanceCount()); // 2
```

## 🎛️ Getters และ Setters

```typescript
class Temperature {
    private _celsius: number = 0;
    
    // Getter
    get celsius(): number {
        return this._celsius;
    }
    
    // Setter
    set celsius(value: number) {
        if (value < -273.15) {
            throw new Error("Temperature cannot be below absolute zero");
        }
        this._celsius = value;
    }
    
    // Computed property
    get fahrenheit(): number {
        return (this._celsius * 9/5) + 32;
    }
    
    set fahrenheit(value: number) {
        this.celsius = (value - 32) * 5/9;
    }
    
    get kelvin(): number {
        return this._celsius + 273.15;
    }
    
    set kelvin(value: number) {
        this.celsius = value - 273.15;
    }
}

// การใช้งาน
const temp = new Temperature();
temp.celsius = 25;
console.log(temp.fahrenheit); // 77
console.log(temp.kelvin);     // 298.15

temp.fahrenheit = 100;
console.log(temp.celsius);    // 37.77777777777778
```

## 🏭 Design Patterns กับ Classes

### 1. **Singleton Pattern**

```typescript
class Database {
    private static instance: Database;
    private connected: boolean = false;
    
    private constructor() {
        // Private constructor ป้องกันการสร้าง instance จากภายนอก
    }
    
    static getInstance(): Database {
        if (!Database.instance) {
            Database.instance = new Database();
        }
        return Database.instance;
    }
    
    connect(): void {
        if (!this.connected) {
            this.connected = true;
            console.log("Connected to database");
        }
    }
    
    disconnect(): void {
        if (this.connected) {
            this.connected = false;
            console.log("Disconnected from database");
        }
    }
    
    isConnected(): boolean {
        return this.connected;
    }
}

// การใช้งาน
const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true - same instance

db1.connect();
console.log(db2.isConnected()); // true
```

### 2. **Factory Pattern**

```typescript
abstract class Vehicle {
    abstract start(): string;
    abstract stop(): string;
}

class Car extends Vehicle {
    start(): string {
        return "Car engine started";
    }
    
    stop(): string {
        return "Car engine stopped";
    }
}

class Motorcycle extends Vehicle {
    start(): string {
        return "Motorcycle engine started";
    }
    
    stop(): string {
        return "Motorcycle engine stopped";
    }
}

class VehicleFactory {
    static createVehicle(type: "car" | "motorcycle"): Vehicle {
        switch (type) {
            case "car":
                return new Car();
            case "motorcycle":
                return new Motorcycle();
            default:
                throw new Error("Unknown vehicle type");
        }
    }
}

// การใช้งาน
const car = VehicleFactory.createVehicle("car");
const motorcycle = VehicleFactory.createVehicle("motorcycle");

console.log(car.start());        // "Car engine started"
console.log(motorcycle.start()); // "Motorcycle engine started"
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **User Management System**

```typescript
abstract class User {
    protected constructor(
        public readonly id: string,
        public readonly email: string,
        protected _name: string,
        protected _isActive: boolean = true
    ) {}
    
    get name(): string {
        return this._name;
    }
    
    set name(value: string) {
        if (value.trim().length < 2) {
            throw new Error("Name must be at least 2 characters");
        }
        this._name = value.trim();
    }
    
    get isActive(): boolean {
        return this._isActive;
    }
    
    activate(): void {
        this._isActive = true;
    }
    
    deactivate(): void {
        this._isActive = false;
    }
    
    abstract getPermissions(): string[];
    abstract getRole(): string;
}

class AdminUser extends User {
    constructor(id: string, email: string, name: string) {
        super(id, email, name);
    }
    
    getPermissions(): string[] {
        return ["read", "write", "delete", "admin"];
    }
    
    getRole(): string {
        return "Administrator";
    }
    
    deleteUser(userId: string): string {
        return `Admin ${this.name} deleted user ${userId}`;
    }
}

class RegularUser extends User {
    constructor(id: string, email: string, name: string) {
        super(id, email, name);
    }
    
    getPermissions(): string[] {
        return ["read"];
    }
    
    getRole(): string {
        return "User";
    }
}

class UserManager {
    private users: Map<string, User> = new Map();
    
    addUser(user: User): void {
        this.users.set(user.id, user);
    }
    
    getUser(id: string): User | undefined {
        return this.users.get(id);
    }
    
    getAllActiveUsers(): User[] {
        return Array.from(this.users.values()).filter(user => user.isActive);
    }
    
    getUsersByRole(role: string): User[] {
        return Array.from(this.users.values()).filter(user => user.getRole() === role);
    }
}
```

### 2. **E-commerce Product System**

```typescript
interface Discountable {
    applyDiscount(percentage: number): number;
}

abstract class Product implements Discountable {
    protected constructor(
        public readonly id: string,
        public readonly name: string,
        protected _price: number,
        protected _stock: number
    ) {}
    
    get price(): number {
        return this._price;
    }
    
    get stock(): number {
        return this._stock;
    }
    
    abstract getCategory(): string;
    abstract getDescription(): string;
    
    applyDiscount(percentage: number): number {
        if (percentage < 0 || percentage > 100) {
            throw new Error("Invalid discount percentage");
        }
        return this._price * (1 - percentage / 100);
    }
    
    updateStock(quantity: number): void {
        if (this._stock + quantity < 0) {
            throw new Error("Insufficient stock");
        }
        this._stock += quantity;
    }
    
    isInStock(): boolean {
        return this._stock > 0;
    }
}

class DigitalProduct extends Product {
    constructor(
        id: string,
        name: string,
        price: number,
        private downloadUrl: string,
        private fileSize: number
    ) {
        super(id, name, price, Number.MAX_SAFE_INTEGER); // Digital products don't have stock limits
    }
    
    getCategory(): string {
        return "Digital";
    }
    
    getDescription(): string {
        return `Digital product: ${this.name} (${this.fileSize}MB)`;
    }
    
    getDownloadUrl(): string {
        return this.downloadUrl;
    }
}

class PhysicalProduct extends Product {
    constructor(
        id: string,
        name: string,
        price: number,
        stock: number,
        private weight: number,
        private dimensions: { width: number; height: number; depth: number }
    ) {
        super(id, name, price, stock);
    }
    
    getCategory(): string {
        return "Physical";
    }
    
    getDescription(): string {
        return `Physical product: ${this.name} (${this.weight}kg)`;
    }
    
    getShippingCost(): number {
        return this.weight * 10; // 10 บาทต่อกิโลกรัม
    }
}
```

## 📝 แบบฝึกหัด

### 1. **Banking System**
สร้างระบบธนาคารที่มี:
- Abstract class `Account` 
- Derived classes: `SavingsAccount`, `CheckingAccount`
- การคำนวณดอกเบี้ย, ค่าธรรมเนียม
- Transaction history

### 2. **Game Character System**
ออกแบบระบบตัวละครในเกมที่มี:
- Base class `Character`
- Different character types: `Warrior`, `Mage`, `Archer`
- Skills, levels, inventory management
- Combat system

### 3. **Library Management**
สร้างระบบห้องสมุดที่มี:
- Abstract class `LibraryItem`
- Book, Magazine, DVD classes
- Member management
- Borrowing/returning system

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 6: Interface และ Type](./06-interfaces-types.md)

**บทต่อไป**: [บทที่ 8: Generics](./08-generics.md)

**กลับไปหน้าหลัก**: [README](./README.md)
