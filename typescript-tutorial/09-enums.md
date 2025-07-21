# บทที่ 9: Enums

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจการใช้งาน Enums ใน TypeScript
- เรียนรู้ประเภทของ Enums (Numeric, String, Heterogeneous)
- เข้าใจ Const Enums และ Computed Members
- เรียนรู้ Best Practices สำหรับการใช้ Enums

## 📋 แนะนำ Enums

### 1. **ปัญหาที่ Enums แก้ไข**

```typescript
// ❌ ใช้ string literals โดยตรง - เสี่ยงต่อ typo
function processStatus(status: string) {
    if (status === "pending") {
        // logic for pending
    } else if (status === "approved") {
        // logic for approved
    } else if (status === "rejected") {
        // logic for rejected
    }
}

processStatus("pendign"); // ❌ Typo ไม่ถูกจับ
processStatus("PENDING"); // ❌ Case sensitivity issue

// ✅ ใช้ Enum - Type-safe และป้องกัน typo
enum Status {
    Pending = "pending",
    Approved = "approved",
    Rejected = "rejected"
}

function processStatusSafe(status: Status) {
    switch (status) {
        case Status.Pending:
            // logic for pending
            break;
        case Status.Approved:
            // logic for approved
            break;
        case Status.Rejected:
            // logic for rejected
            break;
    }
}

processStatusSafe(Status.Pending); // ✅ Type-safe
// processStatusSafe("pending"); // ❌ Error - must use enum
```

## 🔢 Numeric Enums

### 1. **Basic Numeric Enums**

```typescript
// Basic numeric enum - เริ่มจาก 0
enum Direction {
    Up,    // 0
    Down,  // 1
    Left,  // 2
    Right  // 3
}

console.log(Direction.Up);    // 0
console.log(Direction.Down);  // 1
console.log(Direction.Left);  // 2
console.log(Direction.Right); // 3

// Reverse mapping
console.log(Direction[0]); // "Up"
console.log(Direction[1]); // "Down"

// การใช้งาน
function move(direction: Direction) {
    switch (direction) {
        case Direction.Up:
            console.log("Moving up");
            break;
        case Direction.Down:
            console.log("Moving down");
            break;
        case Direction.Left:
            console.log("Moving left");
            break;
        case Direction.Right:
            console.log("Moving right");
            break;
    }
}

move(Direction.Up); // "Moving up"
```

### 2. **Custom Starting Values**

```typescript
// เริ่มจากค่าที่กำหนด
enum HttpStatus {
    OK = 200,
    NotFound = 404,
    InternalServerError = 500
}

// เริ่มจาก 1
enum Priority {
    Low = 1,    // 1
    Medium,     // 2
    High,       // 3
    Critical    // 4
}

// Mixed values
enum MixedEnum {
    A,          // 0
    B,          // 1
    C = 10,     // 10
    D,          // 11
    E = 20,     // 20
    F           // 21
}

console.log(Priority.Low);      // 1
console.log(Priority.Medium);   // 2
console.log(MixedEnum.C);       // 10
console.log(MixedEnum.D);       // 11
```

### 3. **Computed Members**

```typescript
// Computed enum members
enum FileAccess {
    // Constant members
    None,
    Read = 1 << 1,        // 2
    Write = 1 << 2,       // 4
    ReadWrite = Read | Write, // 6
    
    // Computed member
    G = "123".length      // 3
}

console.log(FileAccess.None);      // 0
console.log(FileAccess.Read);      // 2
console.log(FileAccess.Write);     // 4
console.log(FileAccess.ReadWrite); // 6
console.log(FileAccess.G);         // 3

// การใช้งาน bitwise operations
function hasPermission(userPermissions: FileAccess, required: FileAccess): boolean {
    return (userPermissions & required) === required;
}

const userPermissions = FileAccess.ReadWrite;
console.log(hasPermission(userPermissions, FileAccess.Read));  // true
console.log(hasPermission(userPermissions, FileAccess.Write)); // true
```

## 🔤 String Enums

### 1. **Basic String Enums**

```typescript
// String enum
enum Theme {
    Light = "light",
    Dark = "dark",
    Auto = "auto"
}

enum LogLevel {
    Error = "ERROR",
    Warn = "WARN",
    Info = "INFO",
    Debug = "DEBUG"
}

// การใช้งาน
function setTheme(theme: Theme) {
    document.body.className = theme; // จะได้ "light", "dark", หรือ "auto"
}

function log(level: LogLevel, message: string) {
    console.log(`[${level}] ${message}`);
}

setTheme(Theme.Dark);
log(LogLevel.Info, "Application started");

// String enums ไม่มี reverse mapping
console.log(Theme.Light);     // "light"
// console.log(Theme["light"]); // ❌ undefined
```

### 2. **Advanced String Enum Patterns**

```typescript
// API endpoints
enum ApiEndpoint {
    Users = "/api/users",
    Products = "/api/products",
    Orders = "/api/orders",
    Auth = "/api/auth"
}

// Event names
enum EventName {
    UserLogin = "user:login",
    UserLogout = "user:logout",
    OrderCreated = "order:created",
    OrderUpdated = "order:updated"
}

// CSS classes
enum CssClass {
    Container = "container",
    Button = "btn",
    ButtonPrimary = "btn-primary",
    ButtonSecondary = "btn-secondary",
    Alert = "alert",
    AlertSuccess = "alert-success",
    AlertError = "alert-error"
}

// การใช้งาน
function createButton(text: string, type: CssClass.ButtonPrimary | CssClass.ButtonSecondary) {
    const button = document.createElement("button");
    button.textContent = text;
    button.className = `${CssClass.Button} ${type}`;
    return button;
}

const primaryButton = createButton("Save", CssClass.ButtonPrimary);
const secondaryButton = createButton("Cancel", CssClass.ButtonSecondary);
```

## 🔀 Heterogeneous Enums

```typescript
// Heterogeneous enum (ผสม string และ number) - ไม่แนะนำให้ใช้บ่อย
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES"
}

// แต่มักจะใช้แบบนี้ดีกว่า
enum ResponseCode {
    Success = 200,
    NotFound = 404,
    Error = "ERROR"
}

// ตัวอย่างการใช้งาน
function handleResponse(code: ResponseCode) {
    if (typeof code === "number") {
        console.log(`HTTP Status: ${code}`);
    } else {
        console.log(`Error: ${code}`);
    }
}

handleResponse(ResponseCode.Success);  // "HTTP Status: 200"
handleResponse(ResponseCode.Error);    // "Error: ERROR"
```

## ⚡ Const Enums

### 1. **Const Enums vs Regular Enums**

```typescript
// Regular enum
enum RegularEnum {
    A,
    B,
    C
}

// Const enum
const enum ConstEnum {
    A,
    B,
    C
}

// การใช้งาน
const regularValue = RegularEnum.A; // จะสร้าง object ใน runtime
const constValue = ConstEnum.A;     // จะ inline เป็น 0 ใน compile time

// Compiled JavaScript สำหรับ regular enum:
// var RegularEnum;
// (function (RegularEnum) {
//     RegularEnum[RegularEnum["A"] = 0] = "A";
//     RegularEnum[RegularEnum["B"] = 1] = "B";
//     RegularEnum[RegularEnum["C"] = 2] = "C";
// })(RegularEnum || (RegularEnum = {}));
// const regularValue = RegularEnum.A;

// Compiled JavaScript สำหรับ const enum:
// const constValue = 0; /* ConstEnum.A */
```

### 2. **Const Enum Benefits และ Limitations**

```typescript
const enum Colors {
    Red = "#ff0000",
    Green = "#00ff00",
    Blue = "#0000ff"
}

// ✅ Direct access - จะ inline
const red = Colors.Red; // จะกลายเป็น const red = "#ff0000";

// ❌ Dynamic access - ไม่สามารถทำได้กับ const enum
// const colorName = "Red";
// const dynamicColor = Colors[colorName]; // Error

// ❌ Object.values/keys - ไม่สามารถทำได้
// const allColors = Object.values(Colors); // Error

// Use case ที่เหมาะสม
function setBackgroundColor(color: Colors) {
    document.body.style.backgroundColor = color;
}

setBackgroundColor(Colors.Blue); // Efficient - no runtime enum object
```

## 🎯 Enum Best Practices

### 1. **Naming Conventions**

```typescript
// ✅ ดี - ใช้ PascalCase สำหรับ enum name
enum OrderStatus {
    Pending = "pending",
    Processing = "processing",
    Shipped = "shipped",
    Delivered = "delivered",
    Cancelled = "cancelled"
}

// ✅ ดี - ใช้ PascalCase สำหรับ enum members
enum UserRole {
    Admin = "admin",
    Editor = "editor",
    Viewer = "viewer"
}

// ❌ ไม่ดี - camelCase
enum orderStatus {
    pending = "pending"
}

// ❌ ไม่ดี - snake_case
enum ORDER_STATUS {
    PENDING = "pending"
}
```

### 2. **เมื่อไหร่ควรใช้ String vs Numeric Enums**

```typescript
// ✅ ใช้ String Enums เมื่อ:
// - ต้องการ debugging ที่ดี
// - ค่าจะถูกส่งผ่าน API
// - ต้องการให้มนุษย์อ่านได้
enum ApiMethod {
    GET = "GET",
    POST = "POST",
    PUT = "PUT",
    DELETE = "DELETE"
}

// ✅ ใช้ Numeric Enums เมื่อ:
// - ต้องการ performance ที่ดี
// - ใช้ bitwise operations
// - เป็น internal values
enum Permission {
    None = 0,
    Read = 1,
    Write = 2,
    Execute = 4,
    All = Read | Write | Execute
}

// ✅ ใช้ Const Enums เมื่อ:
// - ต้องการ performance สูงสุด
// - ไม่ต้องการ runtime object
// - ไม่ต้องการ dynamic access
const enum HttpStatusCode {
    OK = 200,
    BadRequest = 400,
    Unauthorized = 401,
    NotFound = 404,
    InternalServerError = 500
}
```

### 3. **Enum Utilities**

```typescript
// Utility functions สำหรับ enums
enum Color {
    Red = "red",
    Green = "green",
    Blue = "blue"
}

// ตรวจสอบว่าค่าเป็น valid enum value หรือไม่
function isValidColor(value: string): value is Color {
    return Object.values(Color).includes(value as Color);
}

// ได้ array ของ enum values
function getColorValues(): Color[] {
    return Object.values(Color);
}

// ได้ array ของ enum keys
function getColorKeys(): (keyof typeof Color)[] {
    return Object.keys(Color) as (keyof typeof Color)[];
}

// Convert string to enum safely
function parseColor(value: string): Color | null {
    return isValidColor(value) ? value as Color : null;
}

// การใช้งาน
console.log(isValidColor("red"));    // true
console.log(isValidColor("yellow")); // false
console.log(getColorValues());       // ["red", "green", "blue"]
console.log(getColorKeys());         // ["Red", "Green", "Blue"]
console.log(parseColor("blue"));     // Color.Blue
console.log(parseColor("yellow"));   // null
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **Game Development**

```typescript
// Game state management
enum GameState {
    Menu = "menu",
    Playing = "playing",
    Paused = "paused",
    GameOver = "gameOver",
    Loading = "loading"
}

enum PlayerAction {
    Move = "move",
    Jump = "jump",
    Attack = "attack",
    Defend = "defend",
    UseItem = "useItem"
}

enum ItemType {
    Weapon = "weapon",
    Armor = "armor",
    Consumable = "consumable",
    Quest = "quest"
}

class Game {
    private state: GameState = GameState.Menu;
    
    setState(newState: GameState): void {
        console.log(`Game state changed from ${this.state} to ${newState}`);
        this.state = newState;
    }
    
    handlePlayerAction(action: PlayerAction): void {
        if (this.state !== GameState.Playing) {
            console.log("Action ignored - game not in playing state");
            return;
        }
        
        switch (action) {
            case PlayerAction.Move:
                this.handleMove();
                break;
            case PlayerAction.Jump:
                this.handleJump();
                break;
            case PlayerAction.Attack:
                this.handleAttack();
                break;
            // ... other actions
        }
    }
    
    private handleMove(): void { /* implementation */ }
    private handleJump(): void { /* implementation */ }
    private handleAttack(): void { /* implementation */ }
}
```

### 2. **E-commerce System**

```typescript
// Order management system
enum OrderStatus {
    Pending = "pending",
    Confirmed = "confirmed",
    Processing = "processing",
    Shipped = "shipped",
    Delivered = "delivered",
    Cancelled = "cancelled",
    Returned = "returned"
}

enum PaymentStatus {
    Pending = "pending",
    Authorized = "authorized",
    Captured = "captured",
    Failed = "failed",
    Refunded = "refunded"
}

enum ShippingMethod {
    Standard = "standard",
    Express = "express",
    Overnight = "overnight",
    Pickup = "pickup"
}

const enum Currency {
    USD = "USD",
    EUR = "EUR",
    GBP = "GBP",
    THB = "THB"
}

interface Order {
    id: string;
    status: OrderStatus;
    paymentStatus: PaymentStatus;
    shippingMethod: ShippingMethod;
    currency: Currency;
    total: number;
}

class OrderService {
    updateOrderStatus(orderId: string, newStatus: OrderStatus): void {
        // Validate status transition
        const validTransitions: Record<OrderStatus, OrderStatus[]> = {
            [OrderStatus.Pending]: [OrderStatus.Confirmed, OrderStatus.Cancelled],
            [OrderStatus.Confirmed]: [OrderStatus.Processing, OrderStatus.Cancelled],
            [OrderStatus.Processing]: [OrderStatus.Shipped, OrderStatus.Cancelled],
            [OrderStatus.Shipped]: [OrderStatus.Delivered],
            [OrderStatus.Delivered]: [OrderStatus.Returned],
            [OrderStatus.Cancelled]: [],
            [OrderStatus.Returned]: []
        };
        
        // Implementation here...
    }
    
    calculateShippingCost(method: ShippingMethod, weight: number): number {
        switch (method) {
            case ShippingMethod.Standard:
                return weight * 5;
            case ShippingMethod.Express:
                return weight * 10;
            case ShippingMethod.Overnight:
                return weight * 20;
            case ShippingMethod.Pickup:
                return 0;
            default:
                throw new Error(`Unknown shipping method: ${method}`);
        }
    }
}
```

### 3. **Configuration Management**

```typescript
// Application configuration
const enum Environment {
    Development = "development",
    Staging = "staging",
    Production = "production"
}

enum LogLevel {
    Error = 0,
    Warn = 1,
    Info = 2,
    Debug = 3
}

enum FeatureFlag {
    NewUI = "new_ui",
    AdvancedAnalytics = "advanced_analytics",
    BetaFeatures = "beta_features",
    ExperimentalAPI = "experimental_api"
}

interface AppConfig {
    environment: Environment;
    logLevel: LogLevel;
    enabledFeatures: FeatureFlag[];
    apiUrl: string;
}

class ConfigManager {
    private config: AppConfig;
    
    constructor() {
        this.config = this.loadConfig();
    }
    
    private loadConfig(): AppConfig {
        const env = process.env.NODE_ENV as Environment || Environment.Development;
        
        return {
            environment: env,
            logLevel: env === Environment.Production ? LogLevel.Warn : LogLevel.Debug,
            enabledFeatures: this.getEnabledFeatures(env),
            apiUrl: this.getApiUrl(env)
        };
    }
    
    private getEnabledFeatures(env: Environment): FeatureFlag[] {
        switch (env) {
            case Environment.Development:
                return Object.values(FeatureFlag);
            case Environment.Staging:
                return [FeatureFlag.NewUI, FeatureFlag.AdvancedAnalytics];
            case Environment.Production:
                return [FeatureFlag.NewUI];
            default:
                return [];
        }
    }
    
    private getApiUrl(env: Environment): string {
        switch (env) {
            case Environment.Development:
                return "http://localhost:3000/api";
            case Environment.Staging:
                return "https://staging-api.example.com";
            case Environment.Production:
                return "https://api.example.com";
            default:
                throw new Error(`Unknown environment: ${env}`);
        }
    }
    
    isFeatureEnabled(feature: FeatureFlag): boolean {
        return this.config.enabledFeatures.includes(feature);
    }
    
    shouldLog(level: LogLevel): boolean {
        return level <= this.config.logLevel;
    }
}
```

## 📝 แบบฝึกหัด

### 1. **State Machine**
สร้าง enum สำหรับ finite state machine:
- States: Idle, Loading, Success, Error
- Actions/Events: Start, Complete, Fail, Reset
- ระบบ validation สำหรับ state transitions

### 2. **Permission System**
ออกแบบระบบ permission ด้วย bitwise enums:
- ระดับ permissions ต่างๆ
- การตรวจสอบ permissions
- การรวม/แยก permissions

### 3. **API Response Codes**
สร้างระบบจัดการ HTTP status codes:
- Enum สำหรับ status codes
- Helper functions สำหรับตรวจสอบ status types
- Error handling based on status codes

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 8: Generics](./08-generics.md)

**บทต่อไป**: [บทที่ 10: Modules และ Namespaces](./10-modules-namespaces.md)

**กลับไปหน้าหลัก**: [README](./README.md)
