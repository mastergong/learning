# บทที่ 10: Modules และ Namespaces

## 🎯 จุดประสงค์ของบทเรียน
- เข้าใจระบบ Module ใน TypeScript
- เรียนรู้การ Import/Export แบบต่างๆ
- เข้าใจ Namespaces และเมื่อไหร่ควรใช้
- เรียนรู้ Module Resolution และ Path Mapping

## 📦 TypeScript Modules

### 1. **ES6 Modules Syntax**

```typescript
// math.ts - Named exports
export function add(a: number, b: number): number {
    return a + b;
}

export function subtract(a: number, b: number): number {
    return a - b;
}

export const PI = 3.14159;

export class Calculator {
    multiply(a: number, b: number): number {
        return a * b;
    }
}

// Default export
export default function divide(a: number, b: number): number {
    if (b === 0) {
        throw new Error("Division by zero");
    }
    return a / b;
}
```

```typescript
// main.ts - Importing
import divide from "./math"; // Default import
import { add, subtract, PI, Calculator } from "./math"; // Named imports
import * as MathUtils from "./math"; // Namespace import
import divide as divideFunction, { add as addNumbers } from "./math"; // Aliases

// การใช้งาน
console.log(add(5, 3));                    // 8
console.log(subtract(10, 4));              // 6
console.log(divide(20, 4));                // 5
console.log(PI);                           // 3.14159

const calc = new Calculator();
console.log(calc.multiply(3, 4));          // 12

// Namespace import
console.log(MathUtils.add(1, 2));          // 3
console.log(MathUtils.default(10, 2));     // 5

// Aliased imports
console.log(addNumbers(7, 8));             // 15
console.log(divideFunction(16, 4));        // 4
```

### 2. **Re-exports**

```typescript
// shapes/circle.ts
export interface Circle {
    radius: number;
}

export function calculateArea(circle: Circle): number {
    return Math.PI * circle.radius * circle.radius;
}

// shapes/rectangle.ts
export interface Rectangle {
    width: number;
    height: number;
}

export function calculateArea(rectangle: Rectangle): number {
    return rectangle.width * rectangle.height;
}

// shapes/index.ts - Barrel exports
export { Circle, calculateArea as calculateCircleArea } from "./circle";
export { Rectangle, calculateArea as calculateRectangleArea } from "./rectangle";

// Or export everything
export * from "./circle";
export * from "./rectangle";

// Or with type-only exports
export type { Circle } from "./circle";
export type { Rectangle } from "./rectangle";
```

```typescript
// main.ts
import { Circle, Rectangle, calculateCircleArea, calculateRectangleArea } from "./shapes";

const circle: Circle = { radius: 5 };
const rectangle: Rectangle = { width: 10, height: 20 };

console.log(calculateCircleArea(circle));     // ~78.54
console.log(calculateRectangleArea(rectangle)); // 200
```

### 3. **Dynamic Imports**

```typescript
// Dynamic imports for code splitting
async function loadMathModule() {
    const mathModule = await import("./math");
    return mathModule;
}

// Conditional imports
async function getCalculator(type: "basic" | "scientific") {
    if (type === "basic") {
        const { Calculator } = await import("./basic-calculator");
        return new Calculator();
    } else {
        const { ScientificCalculator } = await import("./scientific-calculator");
        return new ScientificCalculator();
    }
}

// Import with error handling
async function safeImport<T>(modulePath: string): Promise<T | null> {
    try {
        const module = await import(modulePath);
        return module;
    } catch (error) {
        console.error(`Failed to load module: ${modulePath}`, error);
        return null;
    }
}

// Usage
loadMathModule().then(math => {
    console.log(math.add(1, 2));
});

getCalculator("scientific").then(calc => {
    // Use scientific calculator
});
```

## 🏭 Module Patterns

### 1. **Singleton Module Pattern**

```typescript
// database.ts
class Database {
    private static instance: Database;
    private connected = false;
    
    private constructor() {}
    
    static getInstance(): Database {
        if (!Database.instance) {
            Database.instance = new Database();
        }
        return Database.instance;
    }
    
    connect(): void {
        if (!this.connected) {
            console.log("Connecting to database...");
            this.connected = true;
        }
    }
    
    disconnect(): void {
        if (this.connected) {
            console.log("Disconnecting from database...");
            this.connected = false;
        }
    }
    
    isConnected(): boolean {
        return this.connected;
    }
}

// Export singleton instance
export const database = Database.getInstance();

// Or export the class for testing
export { Database };
```

```typescript
// main.ts
import { database } from "./database";

database.connect();
console.log(database.isConnected()); // true
```

### 2. **Factory Module Pattern**

```typescript
// logger-factory.ts
interface Logger {
    log(message: string): void;
    error(message: string): void;
    warn(message: string): void;
}

class ConsoleLogger implements Logger {
    log(message: string): void {
        console.log(`[LOG] ${message}`);
    }
    
    error(message: string): void {
        console.error(`[ERROR] ${message}`);
    }
    
    warn(message: string): void {
        console.warn(`[WARN] ${message}`);
    }
}

class FileLogger implements Logger {
    constructor(private filename: string) {}
    
    log(message: string): void {
        // Write to file
        console.log(`Writing to ${this.filename}: [LOG] ${message}`);
    }
    
    error(message: string): void {
        console.log(`Writing to ${this.filename}: [ERROR] ${message}`);
    }
    
    warn(message: string): void {
        console.log(`Writing to ${this.filename}: [WARN] ${message}`);
    }
}

type LoggerType = "console" | "file";

export function createLogger(type: LoggerType, options?: { filename?: string }): Logger {
    switch (type) {
        case "console":
            return new ConsoleLogger();
        case "file":
            if (!options?.filename) {
                throw new Error("Filename is required for file logger");
            }
            return new FileLogger(options.filename);
        default:
            throw new Error(`Unknown logger type: ${type}`);
    }
}

export type { Logger };
```

### 3. **Configuration Module Pattern**

```typescript
// config.ts
interface AppConfig {
    apiUrl: string;
    apiKey: string;
    timeout: number;
    retries: number;
    debug: boolean;
}

interface DatabaseConfig {
    host: string;
    port: number;
    database: string;
    username: string;
    password: string;
}

interface Config {
    app: AppConfig;
    database: DatabaseConfig;
}

// Default configuration
const defaultConfig: Config = {
    app: {
        apiUrl: "http://localhost:3000",
        apiKey: "",
        timeout: 5000,
        retries: 3,
        debug: false
    },
    database: {
        host: "localhost",
        port: 5432,
        database: "myapp",
        username: "user",
        password: "password"
    }
};

// Configuration manager
class ConfigManager {
    private config: Config;
    
    constructor(initialConfig: Partial<Config> = {}) {
        this.config = this.mergeConfig(defaultConfig, initialConfig);
    }
    
    private mergeConfig(defaultConfig: Config, userConfig: Partial<Config>): Config {
        return {
            app: { ...defaultConfig.app, ...userConfig.app },
            database: { ...defaultConfig.database, ...userConfig.database }
        };
    }
    
    get<T extends keyof Config>(section: T): Config[T] {
        return this.config[section];
    }
    
    set<T extends keyof Config>(section: T, config: Partial<Config[T]>): void {
        this.config[section] = { ...this.config[section], ...config };
    }
    
    getAll(): Config {
        return { ...this.config };
    }
}

// Export singleton instance
export const config = new ConfigManager();

// Export types for other modules
export type { AppConfig, DatabaseConfig, Config };
```

## 🏛️ Namespaces

### 1. **Basic Namespaces**

```typescript
// geometry.ts
namespace Geometry {
    export namespace Shapes {
        export interface Point {
            x: number;
            y: number;
        }
        
        export interface Circle {
            center: Point;
            radius: number;
        }
        
        export interface Rectangle {
            topLeft: Point;
            width: number;
            height: number;
        }
        
        export function calculateCircleArea(circle: Circle): number {
            return Math.PI * circle.radius * circle.radius;
        }
        
        export function calculateRectangleArea(rectangle: Rectangle): number {
            return rectangle.width * rectangle.height;
        }
    }
    
    export namespace Utils {
        export function distance(p1: Shapes.Point, p2: Shapes.Point): number {
            const dx = p2.x - p1.x;
            const dy = p2.y - p1.y;
            return Math.sqrt(dx * dx + dy * dy);
        }
        
        export function isPointInCircle(point: Shapes.Point, circle: Shapes.Circle): boolean {
            const dist = distance(point, circle.center);
            return dist <= circle.radius;
        }
    }
}

// การใช้งาน
const point: Geometry.Shapes.Point = { x: 10, y: 20 };
const circle: Geometry.Shapes.Circle = {
    center: { x: 0, y: 0 },
    radius: 15
};

console.log(Geometry.Shapes.calculateCircleArea(circle));
console.log(Geometry.Utils.isPointInCircle(point, circle));
```

### 2. **Namespace Merging**

```typescript
// math-basic.ts
namespace MathLib {
    export function add(a: number, b: number): number {
        return a + b;
    }
    
    export function subtract(a: number, b: number): number {
        return a - b;
    }
}

// math-advanced.ts
namespace MathLib {
    export function power(base: number, exponent: number): number {
        return Math.pow(base, exponent);
    }
    
    export function factorial(n: number): number {
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    }
}

// math-constants.ts
namespace MathLib {
    export const PI = 3.14159;
    export const E = 2.71828;
}

// main.ts - หลังจาก import ทุกไฟล์แล้ว
console.log(MathLib.add(1, 2));        // 3
console.log(MathLib.power(2, 3));      // 8
console.log(MathLib.PI);               // 3.14159
```

### 3. **Module vs Namespace**

```typescript
// ✅ แนะนำ: ใช้ modules
// user.ts
export interface User {
    id: string;
    name: string;
    email: string;
}

export class UserService {
    async getUser(id: string): Promise<User> {
        // implementation
        throw new Error("Not implemented");
    }
}

// ❌ ไม่แนะนำ: ใช้ namespace สำหรับโค้ดใหม่
namespace UserModule {
    export interface User {
        id: string;
        name: string;
        email: string;
    }
    
    export class UserService {
        async getUser(id: string): Promise<User> {
            throw new Error("Not implemented");
        }
    }
}

// แต่ namespace ยังมีประโยชน์สำหรับ:
// 1. Organizing types
namespace API {
    export interface Request {
        url: string;
        method: string;
    }
    
    export interface Response {
        status: number;
        data: any;
    }
}

// 2. Global augmentation
declare global {
    namespace NodeJS {
        interface ProcessEnv {
            NODE_ENV: "development" | "production" | "test";
            PORT: string;
        }
    }
}
```

## 🔍 Module Resolution

### 1. **Path Mapping**

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"],
      "@services/*": ["services/*"],
      "@types/*": ["types/*"]
    }
  }
}
```

```typescript
// main.ts
import { Button } from "@components/Button";
import { ApiService } from "@services/ApiService";
import { User } from "@types/User";
import { formatDate } from "@utils/dateUtils";

// แทนที่จะเขียนแบบนี้:
// import { Button } from "../../../components/Button";
// import { ApiService } from "../../services/ApiService";
```

### 2. **Triple-Slash Directives**

```typescript
// types.d.ts
/// <reference types="node" />
/// <reference path="./custom-types.d.ts" />

interface CustomGlobal {
    myCustomProperty: string;
}

declare global {
    var myGlobal: CustomGlobal;
}
```

### 3. **Declaration Files**

```typescript
// math-library.d.ts
declare module "math-library" {
    export function add(a: number, b: number): number;
    export function subtract(a: number, b: number): number;
    
    export interface Calculator {
        multiply(a: number, b: number): number;
        divide(a: number, b: number): number;
    }
    
    export class ScientificCalculator implements Calculator {
        multiply(a: number, b: number): number;
        divide(a: number, b: number): number;
        power(base: number, exponent: number): number;
        sqrt(value: number): number;
    }
}

// main.ts
import { add, ScientificCalculator } from "math-library";

const calc = new ScientificCalculator();
console.log(add(1, 2));
console.log(calc.power(2, 3));
```

## 📝 ตัวอย่างการใช้งานจริง

### 1. **API Service Architecture**

```typescript
// types/api.ts
export interface ApiResponse<T> {
    data: T;
    success: boolean;
    message?: string;
}

export interface ApiError {
    code: string;
    message: string;
    details?: any;
}

// services/base-api.ts
import type { ApiResponse, ApiError } from "@types/api";

export abstract class BaseApiService {
    protected baseUrl: string;
    
    constructor(baseUrl: string) {
        this.baseUrl = baseUrl;
    }
    
    protected async request<T>(
        endpoint: string,
        options: RequestInit = {}
    ): Promise<ApiResponse<T>> {
        try {
            const response = await fetch(`${this.baseUrl}${endpoint}`, {
                headers: {
                    "Content-Type": "application/json",
                    ...options.headers,
                },
                ...options,
            });
            
            const data = await response.json();
            
            if (!response.ok) {
                throw new Error(data.message || "API request failed");
            }
            
            return data;
        } catch (error) {
            throw new ApiError(
                "API_ERROR",
                error instanceof Error ? error.message : "Unknown error"
            );
        }
    }
}

// services/user-api.ts
import { BaseApiService } from "./base-api";
import type { User } from "@types/user";

export class UserApiService extends BaseApiService {
    constructor() {
        super("/api/users");
    }
    
    async getUser(id: string): Promise<User> {
        const response = await this.request<User>(`/${id}`);
        return response.data;
    }
    
    async createUser(userData: Omit<User, "id">): Promise<User> {
        const response = await this.request<User>("", {
            method: "POST",
            body: JSON.stringify(userData),
        });
        return response.data;
    }
}

// services/index.ts - Barrel export
export { BaseApiService } from "./base-api";
export { UserApiService } from "./user-api";
export type { ApiResponse, ApiError } from "@types/api";
```

### 2. **Plugin Architecture**

```typescript
// plugin-system/types.ts
export interface Plugin {
    name: string;
    version: string;
    initialize(): Promise<void>;
    cleanup(): Promise<void>;
}

export interface PluginContext {
    log(message: string): void;
    config: Record<string, any>;
}

// plugin-system/manager.ts
import type { Plugin, PluginContext } from "./types";

export class PluginManager {
    private plugins = new Map<string, Plugin>();
    private context: PluginContext;
    
    constructor(config: Record<string, any>) {
        this.context = {
            log: (message: string) => console.log(`[PLUGIN] ${message}`),
            config
        };
    }
    
    async loadPlugin(plugin: Plugin): Promise<void> {
        if (this.plugins.has(plugin.name)) {
            throw new Error(`Plugin ${plugin.name} is already loaded`);
        }
        
        this.context.log(`Loading plugin: ${plugin.name} v${plugin.version}`);
        await plugin.initialize();
        this.plugins.set(plugin.name, plugin);
        this.context.log(`Plugin ${plugin.name} loaded successfully`);
    }
    
    async unloadPlugin(name: string): Promise<void> {
        const plugin = this.plugins.get(name);
        if (!plugin) {
            throw new Error(`Plugin ${name} is not loaded`);
        }
        
        this.context.log(`Unloading plugin: ${name}`);
        await plugin.cleanup();
        this.plugins.delete(name);
        this.context.log(`Plugin ${name} unloaded successfully`);
    }
    
    getPlugin(name: string): Plugin | undefined {
        return this.plugins.get(name);
    }
    
    getAllPlugins(): Plugin[] {
        return Array.from(this.plugins.values());
    }
}

// plugins/logger-plugin.ts
import type { Plugin } from "@/plugin-system/types";

export class LoggerPlugin implements Plugin {
    name = "logger";
    version = "1.0.0";
    
    async initialize(): Promise<void> {
        console.log("Logger plugin initialized");
    }
    
    async cleanup(): Promise<void> {
        console.log("Logger plugin cleaned up");
    }
    
    log(level: "info" | "warn" | "error", message: string): void {
        const timestamp = new Date().toISOString();
        console.log(`[${timestamp}] [${level.toUpperCase()}] ${message}`);
    }
}

// main.ts
import { PluginManager } from "@/plugin-system/manager";
import { LoggerPlugin } from "@/plugins/logger-plugin";

async function main() {
    const pluginManager = new PluginManager({
        environment: "development"
    });
    
    const loggerPlugin = new LoggerPlugin();
    await pluginManager.loadPlugin(loggerPlugin);
    
    // Use plugin
    const logger = pluginManager.getPlugin("logger") as LoggerPlugin;
    logger?.log("info", "Application started");
}

main().catch(console.error);
```

### 3. **Feature Module System**

```typescript
// features/types.ts
export interface FeatureModule {
    name: string;
    dependencies: string[];
    initialize(): Promise<void>;
    routes?: Route[];
    services?: ServiceDefinition[];
}

export interface Route {
    path: string;
    handler: Function;
    method: "GET" | "POST" | "PUT" | "DELETE";
}

export interface ServiceDefinition {
    name: string;
    implementation: any;
}

// features/user/index.ts
import type { FeatureModule } from "../types";
import { UserService } from "./user.service";
import { userRoutes } from "./user.routes";

export const userModule: FeatureModule = {
    name: "user",
    dependencies: ["auth"],
    
    async initialize() {
        console.log("User module initialized");
    },
    
    routes: userRoutes,
    
    services: [
        {
            name: "UserService",
            implementation: UserService
        }
    ]
};

// features/auth/index.ts
import type { FeatureModule } from "../types";
import { AuthService } from "./auth.service";
import { authRoutes } from "./auth.routes";

export const authModule: FeatureModule = {
    name: "auth",
    dependencies: [],
    
    async initialize() {
        console.log("Auth module initialized");
    },
    
    routes: authRoutes,
    
    services: [
        {
            name: "AuthService",
            implementation: AuthService
        }
    ]
};

// features/index.ts
import type { FeatureModule } from "./types";
import { authModule } from "./auth";
import { userModule } from "./user";

export const modules: FeatureModule[] = [authModule, userModule];

export async function initializeModules(modules: FeatureModule[]): Promise<void> {
    // Sort modules by dependencies
    const sortedModules = topologicalSort(modules);
    
    for (const module of sortedModules) {
        await module.initialize();
    }
}

function topologicalSort(modules: FeatureModule[]): FeatureModule[] {
    // Implementation of topological sort for dependency resolution
    return modules; // Simplified
}
```

## 📝 แบบฝึกหัด

### 1. **Library System**
สร้างระบบห้องสมุดด้วย module pattern:
- User management module
- Book management module
- Borrowing system module
- Shared types และ utilities

### 2. **E-commerce Modules**
ออกแบบระบบ e-commerce แบบ modular:
- Product catalog module
- Shopping cart module
- Payment processing module
- Order management module

### 3. **Microservices Communication**
สร้างระบบสำหรับ microservices:
- API client modules
- Service discovery
- Message queue integration
- Error handling และ retry logic

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 9: Enums](./09-enums.md)

**บทต่อไป**: [บทที่ 11: Advanced Types](./11-advanced-types.md)

**กลับไปหน้าหลัก**: [README](./README.md)
