# บทที่ 16: Performance Optimization

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้เทคนิคการ optimize TypeScript code
- เข้าใจการจัดการ Memory และ GC
- เรียนรู้ Bundle optimization และ Tree shaking
- เข้าใจ Runtime performance tuning

## ⚡ การ Optimize TypeScript Code

### 1. **Type-Level Optimizations**

```typescript
// ❌ ไม่ดี: การใช้ any ทำให้ lose type safety และ performance
function processData(data: any): any {
    return data.map((item: any) => item.value * 2);
}

// ✅ ดี: Type-safe และ optimized
function processDataOptimized<T extends { value: number }>(data: T[]): number[] {
    return data.map(item => item.value * 2);
}

// ❌ ไม่ดี: Union types ที่ใหญ่เกินไป
type BadEventType = 
    | "click" | "hover" | "focus" | "blur" | "keydown" | "keyup" 
    | "mousedown" | "mouseup" | "mousemove" | "scroll" | "resize"
    | "load" | "unload" | "submit" | "change" | "input" | "select";

// ✅ ดี: จัดกลุ่ม types ให้เหมาะสม
type MouseEvents = "click" | "hover" | "mousedown" | "mouseup" | "mousemove";
type KeyboardEvents = "keydown" | "keyup";
type FormEvents = "submit" | "change" | "input" | "select";
type WindowEvents = "load" | "unload" | "resize" | "scroll";

type EventType = MouseEvents | KeyboardEvents | FormEvents | WindowEvents;

// การใช้ const assertions สำหรับ performance
const eventTypes = ["click", "hover", "focus"] as const;
type EventTypes = typeof eventTypes[number]; // "click" | "hover" | "focus"

// Optimized type guards
function isString(value: unknown): value is string {
    return typeof value === "string";
}

function isNumber(value: unknown): value is number {
    return typeof value === "number";
}

// Generic type constraints ที่มีประสิทธิภาพ
interface Serializable {
    toJSON(): string;
}

function serialize<T extends Serializable>(items: T[]): string[] {
    return items.map(item => item.toJSON());
}

// การใช้ mapped types อย่างมีประสิทธิภาพ
type OptionalFields<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface User {
    id: string;
    name: string;
    email: string;
    age: number;
}

// ทำให้เฉพาะ email เป็น optional
type UserWithOptionalEmail = OptionalFields<User, "email">;
```

### 2. **Runtime Performance Optimizations**

```typescript
// การใช้ Object.freeze() สำหรับ immutable data
const CONSTANTS = Object.freeze({
    API_ENDPOINTS: Object.freeze({
        USERS: "/api/users",
        PRODUCTS: "/api/products",
        ORDERS: "/api/orders"
    }),
    HTTP_METHODS: Object.freeze({
        GET: "GET",
        POST: "POST",
        PUT: "PUT",
        DELETE: "DELETE"
    })
} as const);

// Efficient array operations
class OptimizedArray<T> {
    private items: T[] = [];
    
    // ใช้ pre-allocated array สำหรับ performance
    constructor(initialCapacity: number = 10) {
        this.items = new Array(initialCapacity);
    }
    
    // Efficient bulk operations
    addMany(items: T[]): void {
        if (this.items.length + items.length > this.items.length) {
            // Resize array if needed
            const newArray = new Array(Math.max(this.items.length * 2, this.items.length + items.length));
            for (let i = 0; i < this.items.length; i++) {
                newArray[i] = this.items[i];
            }
            this.items = newArray;
        }
        
        // Use optimized copying
        for (let i = 0; i < items.length; i++) {
            this.items[this.items.length + i] = items[i];
        }
    }
    
    // Memory-efficient filtering
    filterInPlace(predicate: (item: T) => boolean): void {
        let writeIndex = 0;
        
        for (let readIndex = 0; readIndex < this.items.length; readIndex++) {
            if (predicate(this.items[readIndex])) {
                if (writeIndex !== readIndex) {
                    this.items[writeIndex] = this.items[readIndex];
                }
                writeIndex++;
            }
        }
        
        // Trim array
        this.items.length = writeIndex;
    }
}

// Memoization สำหรับ expensive computations
function memoize<Args extends any[], Return>(
    fn: (...args: Args) => Return,
    getKey?: (...args: Args) => string
): (...args: Args) => Return {
    const cache = new Map<string, Return>();
    
    return (...args: Args): Return => {
        const key = getKey ? getKey(...args) : JSON.stringify(args);
        
        if (cache.has(key)) {
            return cache.get(key)!;
        }
        
        const result = fn(...args);
        cache.set(key, result);
        return result;
    };
}

// การใช้งาน memoization
const expensiveCalculation = memoize((a: number, b: number): number => {
    console.log(`Computing ${a} + ${b}`);
    // Simulate expensive operation
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
        result += a + b;
    }
    return result;
});

// Object pooling สำหรับ performance
class ObjectPool<T> {
    private pool: T[] = [];
    private createFn: () => T;
    private resetFn: (obj: T) => void;
    
    constructor(createFn: () => T, resetFn: (obj: T) => void, initialSize: number = 10) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        
        // Pre-populate pool
        for (let i = 0; i < initialSize; i++) {
            this.pool.push(createFn());
        }
    }
    
    acquire(): T {
        if (this.pool.length > 0) {
            return this.pool.pop()!;
        }
        return this.createFn();
    }
    
    release(obj: T): void {
        this.resetFn(obj);
        this.pool.push(obj);
    }
}

// การใช้งาน Object Pool
interface Point {
    x: number;
    y: number;
}

const pointPool = new ObjectPool<Point>(
    () => ({ x: 0, y: 0 }),
    (point) => {
        point.x = 0;
        point.y = 0;
    },
    50
);

function processPoints(coordinates: [number, number][]): Point[] {
    const points: Point[] = [];
    
    for (const [x, y] of coordinates) {
        const point = pointPool.acquire();
        point.x = x;
        point.y = y;
        points.push(point);
    }
    
    return points;
}

function releasePoints(points: Point[]): void {
    for (const point of points) {
        pointPool.release(point);
    }
}
```

## 🗄️ Memory Management

### 1. **Memory-Efficient Data Structures**

```typescript
// Efficient Set operations
class OptimizedSet<T> {
    private items = new Set<T>();
    private _size = 0;
    
    add(item: T): boolean {
        const sizeBefore = this.items.size;
        this.items.add(item);
        
        if (this.items.size > sizeBefore) {
            this._size++;
            return true;
        }
        return false;
    }
    
    delete(item: T): boolean {
        if (this.items.delete(item)) {
            this._size--;
            return true;
        }
        return false;
    }
    
    has(item: T): boolean {
        return this.items.has(item);
    }
    
    get size(): number {
        return this._size;
    }
    
    clear(): void {
        this.items.clear();
        this._size = 0;
    }
    
    // Memory-efficient iteration
    *values(): IterableIterator<T> {
        yield* this.items;
    }
}

// Weak references สำหรับ cache
class WeakCache<K extends object, V> {
    private cache = new WeakMap<K, V>();
    
    get(key: K): V | undefined {
        return this.cache.get(key);
    }
    
    set(key: K, value: V): void {
        this.cache.set(key, value);
    }
    
    has(key: K): boolean {
        return this.cache.has(key);
    }
    
    delete(key: K): boolean {
        return this.cache.delete(key);
    }
}

// Lazy loading implementation
class LazyValue<T> {
    private _value?: T;
    private _computed = false;
    
    constructor(private computer: () => T) {}
    
    get value(): T {
        if (!this._computed) {
            this._value = this.computer();
            this._computed = true;
        }
        return this._value!;
    }
    
    reset(): void {
        this._value = undefined;
        this._computed = false;
    }
    
    get isComputed(): boolean {
        return this._computed;
    }
}

// การใช้งาน Lazy loading
class DataService {
    private _expensiveData = new LazyValue(() => this.loadExpensiveData());
    private _config = new LazyValue(() => this.loadConfiguration());
    
    get expensiveData() {
        return this._expensiveData.value;
    }
    
    get config() {
        return this._config.value;
    }
    
    private loadExpensiveData(): any[] {
        console.log("Loading expensive data...");
        // Simulate expensive data loading
        return Array.from({ length: 10000 }, (_, i) => ({ id: i, value: Math.random() }));
    }
    
    private loadConfiguration(): any {
        console.log("Loading configuration...");
        return {
            apiUrl: "https://api.example.com",
            timeout: 5000,
            retries: 3
        };
    }
    
    refreshData(): void {
        this._expensiveData.reset();
    }
    
    refreshConfig(): void {
        this._config.reset();
    }
}

// Memory monitoring
class MemoryMonitor {
    private static instance: MemoryMonitor;
    private listeners: Array<(info: MemoryInfo) => void> = [];
    private intervalId?: NodeJS.Timeout;
    
    static getInstance(): MemoryMonitor {
        if (!MemoryMonitor.instance) {
            MemoryMonitor.instance = new MemoryMonitor();
        }
        return MemoryMonitor.instance;
    }
    
    startMonitoring(intervalMs: number = 5000): void {
        if (this.intervalId) {
            clearInterval(this.intervalId);
        }
        
        this.intervalId = setInterval(() => {
            if (typeof window !== "undefined" && "memory" in performance) {
                const memoryInfo = (performance as any).memory;
                this.notifyListeners(memoryInfo);
            }
        }, intervalMs);
    }
    
    stopMonitoring(): void {
        if (this.intervalId) {
            clearInterval(this.intervalId);
            this.intervalId = undefined;
        }
    }
    
    addListener(listener: (info: MemoryInfo) => void): void {
        this.listeners.push(listener);
    }
    
    removeListener(listener: (info: MemoryInfo) => void): void {
        const index = this.listeners.indexOf(listener);
        if (index !== -1) {
            this.listeners.splice(index, 1);
        }
    }
    
    private notifyListeners(info: MemoryInfo): void {
        this.listeners.forEach(listener => listener(info));
    }
}

interface MemoryInfo {
    usedJSHeapSize: number;
    totalJSHeapSize: number;
    jsHeapSizeLimit: number;
}
```

### 2. **Garbage Collection Optimization**

```typescript
// การจัดการ Event Listeners อย่างมีประสิทธิภาพ
class EventManager {
    private listeners = new Map<string, Set<Function>>();
    private abortController = new AbortController();
    
    addEventListener(element: EventTarget, event: string, handler: Function): void {
        const boundHandler = handler.bind(this);
        
        // ใช้ AbortController สำหรับ cleanup
        element.addEventListener(
            event, 
            boundHandler as EventListener, 
            { signal: this.abortController.signal }
        );
        
        // เก็บ reference สำหรับ manual cleanup
        if (!this.listeners.has(event)) {
            this.listeners.set(event, new Set());
        }
        this.listeners.get(event)!.add(boundHandler);
    }
    
    removeEventListener(event: string, handler: Function): void {
        const listeners = this.listeners.get(event);
        if (listeners) {
            listeners.delete(handler);
            if (listeners.size === 0) {
                this.listeners.delete(event);
            }
        }
    }
    
    cleanup(): void {
        // Remove all event listeners at once
        this.abortController.abort();
        this.listeners.clear();
        
        // Create new AbortController for future use
        this.abortController = new AbortController();
    }
}

// การจัดการ DOM references
class DOMManager {
    private elementRefs = new WeakMap<HTMLElement, any>();
    private observers = new Set<MutationObserver | IntersectionObserver | ResizeObserver>();
    
    storeElementData<T>(element: HTMLElement, data: T): void {
        this.elementRefs.set(element, data);
    }
    
    getElementData<T>(element: HTMLElement): T | undefined {
        return this.elementRefs.get(element);
    }
    
    createMutationObserver(callback: MutationCallback, options?: MutationObserverInit): MutationObserver {
        const observer = new MutationObserver(callback);
        this.observers.add(observer);
        return observer;
    }
    
    createIntersectionObserver(callback: IntersectionObserverCallback, options?: IntersectionObserverInit): IntersectionObserver {
        const observer = new IntersectionObserver(callback, options);
        this.observers.add(observer);
        return observer;
    }
    
    cleanup(): void {
        // Disconnect all observers
        this.observers.forEach(observer => {
            observer.disconnect();
        });
        this.observers.clear();
        
        // WeakMap will be garbage collected automatically
    }
}

// Resource cleanup pattern
abstract class ResourceManager {
    private resources = new Set<Disposable>();
    
    protected addResource<T extends Disposable>(resource: T): T {
        this.resources.add(resource);
        return resource;
    }
    
    dispose(): void {
        this.resources.forEach(resource => {
            try {
                resource.dispose();
            } catch (error) {
                console.error("Error disposing resource:", error);
            }
        });
        this.resources.clear();
    }
}

interface Disposable {
    dispose(): void;
}

// การใช้งาน Resource Manager
class DatabaseConnection implements Disposable {
    private isConnected = true;
    
    query(sql: string): Promise<any[]> {
        if (!this.isConnected) {
            throw new Error("Connection is closed");
        }
        // Implementation
        return Promise.resolve([]);
    }
    
    dispose(): void {
        if (this.isConnected) {
            console.log("Closing database connection");
            this.isConnected = false;
        }
    }
}

class FileHandle implements Disposable {
    constructor(private filePath: string) {}
    
    read(): string {
        // Implementation
        return "";
    }
    
    write(content: string): void {
        // Implementation
    }
    
    dispose(): void {
        console.log(`Closing file handle: ${this.filePath}`);
    }
}

class ApplicationService extends ResourceManager {
    private db: DatabaseConnection;
    private fileHandle: FileHandle;
    
    constructor() {
        super();
        this.db = this.addResource(new DatabaseConnection());
        this.fileHandle = this.addResource(new FileHandle("/tmp/app.log"));
    }
    
    async performOperations(): Promise<void> {
        const data = await this.db.query("SELECT * FROM users");
        this.fileHandle.write(`Processed ${data.length} records`);
    }
}

// การใช้งาน
const service = new ApplicationService();
try {
    await service.performOperations();
} finally {
    service.dispose(); // ทำ cleanup ทุก resources
}
```

## 📦 Bundle Optimization

### 1. **Tree Shaking และ Dead Code Elimination**

```typescript
// ❌ ไม่ดี: Import ทั้ง library
import * as _ from 'lodash';

export function processArray(items: any[]): any[] {
    return _.map(items, _.toUpper);
}

// ✅ ดี: Import เฉพาะที่ใช้
import { map } from 'lodash/map';
import { toUpper } from 'lodash/toUpper';

export function processArrayOptimized(items: string[]): string[] {
    return map(items, toUpper);
}

// การสร้าง utility functions ที่ tree-shakable
// utils/array.ts
export const chunk = <T>(array: T[], size: number): T[][] => {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += size) {
        chunks.push(array.slice(i, i + size));
    }
    return chunks;
};

export const unique = <T>(array: T[]): T[] => {
    return Array.from(new Set(array));
};

export const groupBy = <T, K extends keyof T>(array: T[], key: K): Record<string, T[]> => {
    return array.reduce((groups, item) => {
        const groupKey = String(item[key]);
        groups[groupKey] = groups[groupKey] || [];
        groups[groupKey].push(item);
        return groups;
    }, {} as Record<string, T[]>);
};

// utils/string.ts
export const capitalize = (str: string): string => {
    return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
};

export const kebabCase = (str: string): string => {
    return str
        .replace(/([a-z])([A-Z])/g, '$1-$2')
        .replace(/[\s_]+/g, '-')
        .toLowerCase();
};

export const truncate = (str: string, length: number): string => {
    return str.length > length ? str.slice(0, length) + '...' : str;
};

// การใช้งานที่ tree-shakable
import { chunk, unique } from './utils/array';
import { capitalize } from './utils/string';

export function processData(data: string[]): string[][] {
    const uniqueData = unique(data);
    const capitalizedData = uniqueData.map(capitalize);
    return chunk(capitalizedData, 10);
}

// Dynamic imports สำหรับ code splitting
class FeatureLoader {
    static async loadChartingFeature() {
        const { ChartComponent } = await import('./features/charting');
        return ChartComponent;
    }
    
    static async loadDataAnalysisFeature() {
        const { DataAnalysisModule } = await import('./features/data-analysis');
        return DataAnalysisModule;
    }
    
    static async loadReportingFeature() {
        const { ReportGenerator } = await import('./features/reporting');
        return ReportGenerator;
    }
}

// Conditional imports
interface FeatureFlags {
    enableAdvancedAnalytics: boolean;
    enableRealTimeUpdates: boolean;
    enableDataExport: boolean;
}

class ConditionalFeatureLoader {
    constructor(private featureFlags: FeatureFlags) {}
    
    async loadFeatures() {
        const features: any[] = [];
        
        if (this.featureFlags.enableAdvancedAnalytics) {
            const analytics = await import('./features/analytics');
            features.push(analytics.AdvancedAnalytics);
        }
        
        if (this.featureFlags.enableRealTimeUpdates) {
            const realtime = await import('./features/realtime');
            features.push(realtime.RealTimeModule);
        }
        
        if (this.featureFlags.enableDataExport) {
            const exporter = await import('./features/export');
            features.push(exporter.DataExporter);
        }
        
        return features;
    }
}
```

### 2. **Module Federation และ Micro-frontends**

```typescript
// shared/types.ts - Shared types across micro-frontends
export interface User {
    id: string;
    name: string;
    email: string;
    role: string;
}

export interface Product {
    id: string;
    name: string;
    price: number;
    category: string;
}

export interface SharedAPI {
    getCurrentUser(): Promise<User>;
    getProducts(): Promise<Product[]>;
    updateUser(user: Partial<User>): Promise<User>;
}

// shared/events.ts - Event system for micro-frontend communication
export class MicroFrontendEventBus {
    private static instance: MicroFrontendEventBus;
    private listeners = new Map<string, Set<Function>>();
    
    static getInstance(): MicroFrontendEventBus {
        if (!MicroFrontendEventBus.instance) {
            MicroFrontendEventBus.instance = new MicroFrontendEventBus();
        }
        return MicroFrontendEventBus.instance;
    }
    
    emit(event: string, data?: any): void {
        const listeners = this.listeners.get(event);
        if (listeners) {
            listeners.forEach(listener => {
                try {
                    listener(data);
                } catch (error) {
                    console.error(`Error in event listener for ${event}:`, error);
                }
            });
        }
    }
    
    on(event: string, listener: Function): () => void {
        if (!this.listeners.has(event)) {
            this.listeners.set(event, new Set());
        }
        
        this.listeners.get(event)!.add(listener);
        
        // Return unsubscribe function
        return () => {
            const listeners = this.listeners.get(event);
            if (listeners) {
                listeners.delete(listener);
                if (listeners.size === 0) {
                    this.listeners.delete(event);
                }
            }
        };
    }
    
    off(event: string, listener?: Function): void {
        if (!listener) {
            this.listeners.delete(event);
            return;
        }
        
        const listeners = this.listeners.get(event);
        if (listeners) {
            listeners.delete(listener);
            if (listeners.size === 0) {
                this.listeners.delete(event);
            }
        }
    }
}

// host/module-loader.ts - Dynamic module loading
interface ModuleDefinition {
    name: string;
    url: string;
    mount: string;
    scope: string;
}

class ModuleLoader {
    private loadedModules = new Map<string, any>();
    private loadingPromises = new Map<string, Promise<any>>();
    
    async loadModule(definition: ModuleDefinition): Promise<any> {
        if (this.loadedModules.has(definition.name)) {
            return this.loadedModules.get(definition.name);
        }
        
        if (this.loadingPromises.has(definition.name)) {
            return this.loadingPromises.get(definition.name);
        }
        
        const loadingPromise = this.loadRemoteModule(definition);
        this.loadingPromises.set(definition.name, loadingPromise);
        
        try {
            const module = await loadingPromise;
            this.loadedModules.set(definition.name, module);
            this.loadingPromises.delete(definition.name);
            return module;
        } catch (error) {
            this.loadingPromises.delete(definition.name);
            throw error;
        }
    }
    
    private async loadRemoteModule(definition: ModuleDefinition): Promise<any> {
        // Load remote entry script
        await this.loadScript(definition.url);
        
        // Initialize shared scope
        await this.initializeSharing();
        
        // Get container
        const container = (window as any)[definition.scope];
        if (!container) {
            throw new Error(`Container ${definition.scope} not found`);
        }
        
        // Initialize container
        await container.init((window as any).__webpack_share_scopes__.default);
        
        // Get factory
        const factory = await container.get(definition.mount);
        
        // Return module
        return factory();
    }
    
    private loadScript(url: string): Promise<void> {
        return new Promise((resolve, reject) => {
            const script = document.createElement('script');
            script.src = url;
            script.onload = () => resolve();
            script.onerror = () => reject(new Error(`Failed to load script: ${url}`));
            document.head.appendChild(script);
        });
    }
    
    private async initializeSharing(): Promise<void> {
        if (!(window as any).__webpack_share_scopes__) {
            (window as any).__webpack_share_scopes__ = { default: {} };
        }
    }
    
    unloadModule(name: string): void {
        this.loadedModules.delete(name);
        this.loadingPromises.delete(name);
    }
    
    getLoadedModules(): string[] {
        return Array.from(this.loadedModules.keys());
    }
}

// Performance-optimized component loader
class ComponentLoader {
    private componentCache = new Map<string, React.ComponentType<any>>();
    private preloadPromises = new Map<string, Promise<any>>();
    
    async loadComponent<T = any>(
        componentPath: string,
        preload: boolean = false
    ): Promise<React.ComponentType<T>> {
        // Check cache first
        if (this.componentCache.has(componentPath)) {
            return this.componentCache.get(componentPath)!;
        }
        
        // Check if already preloading
        if (this.preloadPromises.has(componentPath)) {
            const module = await this.preloadPromises.get(componentPath)!;
            const Component = module.default || module;
            this.componentCache.set(componentPath, Component);
            return Component;
        }
        
        // Load component
        const importPromise = import(componentPath);
        
        if (preload) {
            this.preloadPromises.set(componentPath, importPromise);
        }
        
        const module = await importPromise;
        const Component = module.default || module;
        
        this.componentCache.set(componentPath, Component);
        this.preloadPromises.delete(componentPath);
        
        return Component;
    }
    
    preloadComponent(componentPath: string): void {
        if (!this.componentCache.has(componentPath) && !this.preloadPromises.has(componentPath)) {
            this.preloadPromises.set(componentPath, import(componentPath));
        }
    }
    
    preloadMultiple(componentPaths: string[]): void {
        componentPaths.forEach(path => this.preloadComponent(path));
    }
    
    clearCache(): void {
        this.componentCache.clear();
        this.preloadPromises.clear();
    }
}
```

## 🚀 Runtime Performance

### 1. **Async Performance Optimization**

```typescript
// การ optimize Promise operations
class PromiseUtils {
    // Batch promises with concurrency limit
    static async batchProcess<T, R>(
        items: T[],
        processor: (item: T) => Promise<R>,
        concurrency: number = 5
    ): Promise<R[]> {
        const results: R[] = [];
        
        for (let i = 0; i < items.length; i += concurrency) {
            const batch = items.slice(i, i + concurrency);
            const batchPromises = batch.map(processor);
            const batchResults = await Promise.all(batchPromises);
            results.push(...batchResults);
        }
        
        return results;
    }
    
    // Promise with timeout
    static withTimeout<T>(promise: Promise<T>, timeoutMs: number): Promise<T> {
        return Promise.race([
            promise,
            new Promise<never>((_, reject) => {
                setTimeout(() => reject(new Error('Operation timed out')), timeoutMs);
            })
        ]);
    }
    
    // Retry with exponential backoff
    static async retry<T>(
        operation: () => Promise<T>,
        maxAttempts: number = 3,
        baseDelay: number = 1000
    ): Promise<T> {
        let lastError: Error;
        
        for (let attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                return await operation();
            } catch (error) {
                lastError = error instanceof Error ? error : new Error('Unknown error');
                
                if (attempt === maxAttempts) {
                    throw lastError;
                }
                
                const delay = baseDelay * Math.pow(2, attempt - 1);
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
        
        throw lastError!;
    }
    
    // Promise pool for resource management
    static createPromisePool<T>(
        promiseFactory: () => Promise<T>,
        poolSize: number = 10
    ): PromisePool<T> {
        return new PromisePool(promiseFactory, poolSize);
    }
}

class PromisePool<T> {
    private pool: Promise<T>[] = [];
    private activePromises = 0;
    
    constructor(
        private promiseFactory: () => Promise<T>,
        private maxSize: number
    ) {}
    
    async acquire(): Promise<T> {
        if (this.pool.length > 0) {
            return this.pool.pop()!;
        }
        
        if (this.activePromises < this.maxSize) {
            this.activePromises++;
            try {
                return await this.promiseFactory();
            } finally {
                this.activePromises--;
            }
        }
        
        // Wait for a promise to become available
        return new Promise((resolve) => {
            const checkPool = () => {
                if (this.pool.length > 0) {
                    resolve(this.pool.pop()!);
                } else {
                    setTimeout(checkPool, 10);
                }
            };
            checkPool();
        });
    }
    
    release(promise: Promise<T>): void {
        if (this.pool.length < this.maxSize) {
            this.pool.push(promise);
        }
    }
}

// Efficient data streaming
class DataStream<T> {
    private subscribers = new Set<(data: T) => void>();
    private buffer: T[] = [];
    private bufferSize: number;
    private flushInterval: number;
    private intervalId?: NodeJS.Timeout;
    
    constructor(bufferSize: number = 100, flushInterval: number = 1000) {
        this.bufferSize = bufferSize;
        this.flushInterval = flushInterval;
        this.startFlushing();
    }
    
    push(data: T): void {
        this.buffer.push(data);
        
        if (this.buffer.length >= this.bufferSize) {
            this.flush();
        }
    }
    
    subscribe(callback: (data: T) => void): () => void {
        this.subscribers.add(callback);
        
        return () => {
            this.subscribers.delete(callback);
        };
    }
    
    private flush(): void {
        if (this.buffer.length === 0) return;
        
        const dataToFlush = this.buffer.splice(0);
        
        // Use requestIdleCallback for better performance
        if (typeof window !== 'undefined' && 'requestIdleCallback' in window) {
            requestIdleCallback(() => {
                this.notifySubscribers(dataToFlush);
            });
        } else {
            setTimeout(() => {
                this.notifySubscribers(dataToFlush);
            }, 0);
        }
    }
    
    private notifySubscribers(data: T[]): void {
        data.forEach(item => {
            this.subscribers.forEach(callback => {
                try {
                    callback(item);
                } catch (error) {
                    console.error('Error in stream subscriber:', error);
                }
            });
        });
    }
    
    private startFlushing(): void {
        this.intervalId = setInterval(() => {
            this.flush();
        }, this.flushInterval);
    }
    
    destroy(): void {
        if (this.intervalId) {
            clearInterval(this.intervalId);
        }
        this.flush();
        this.subscribers.clear();
        this.buffer = [];
    }
}

// การใช้งาน optimized data processing
class DataProcessor {
    private stream = new DataStream<any>(500, 100);
    
    constructor() {
        this.stream.subscribe(this.processData.bind(this));
    }
    
    addData(data: any): void {
        this.stream.push(data);
    }
    
    private processData(data: any): void {
        // Process individual data items
        console.log('Processing:', data);
    }
    
    async processLargeDataset(dataset: any[]): Promise<void> {
        const results = await PromiseUtils.batchProcess(
            dataset,
            async (item) => {
                // Simulate processing
                await new Promise(resolve => setTimeout(resolve, 10));
                return { ...item, processed: true };
            },
            10 // Process 10 items concurrently
        );
        
        console.log(`Processed ${results.length} items`);
    }
    
    async processWithRetry(data: any): Promise<any> {
        return PromiseUtils.retry(
            async () => {
                // Simulate unreliable operation
                if (Math.random() < 0.3) {
                    throw new Error('Processing failed');
                }
                return { ...data, processed: true };
            },
            3,
            500
        );
    }
}
```

### 2. **Virtual DOM และ Rendering Optimization**

```typescript
// Virtual scrolling implementation
interface VirtualScrollItem {
    id: string;
    height: number;
    data: any;
}

class VirtualScroller<T extends VirtualScrollItem> {
    private container: HTMLElement;
    private items: T[] = [];
    private visibleItems: T[] = [];
    private scrollTop = 0;
    private containerHeight = 0;
    private itemHeight = 50; // Default item height
    private overscan = 5; // Number of items to render outside visible area
    
    constructor(container: HTMLElement, itemHeight: number = 50) {
        this.container = container;
        this.itemHeight = itemHeight;
        this.setupEventListeners();
        this.updateContainerHeight();
    }
    
    setItems(items: T[]): void {
        this.items = items;
        this.updateVirtualList();
    }
    
    private setupEventListeners(): void {
        this.container.addEventListener('scroll', () => {
            this.scrollTop = this.container.scrollTop;
            this.updateVirtualList();
        });
        
        window.addEventListener('resize', () => {
            this.updateContainerHeight();
            this.updateVirtualList();
        });
    }
    
    private updateContainerHeight(): void {
        this.containerHeight = this.container.clientHeight;
    }
    
    private updateVirtualList(): void {
        if (this.items.length === 0) {
            this.visibleItems = [];
            return;
        }
        
        const startIndex = Math.max(0, Math.floor(this.scrollTop / this.itemHeight) - this.overscan);
        const endIndex = Math.min(
            this.items.length - 1,
            Math.floor((this.scrollTop + this.containerHeight) / this.itemHeight) + this.overscan
        );
        
        this.visibleItems = this.items.slice(startIndex, endIndex + 1);
        
        // Update container content
        this.renderVisibleItems(startIndex);
    }
    
    private renderVisibleItems(startIndex: number): void {
        const totalHeight = this.items.length * this.itemHeight;
        const offsetY = startIndex * this.itemHeight;
        
        // Update container
        this.container.style.height = `${totalHeight}px`;
        this.container.style.paddingTop = `${offsetY}px`;
        
        // Render visible items
        this.container.innerHTML = '';
        this.visibleItems.forEach((item, index) => {
            const element = this.createItemElement(item, startIndex + index);
            this.container.appendChild(element);
        });
    }
    
    private createItemElement(item: T, index: number): HTMLElement {
        const element = document.createElement('div');
        element.style.height = `${this.itemHeight}px`;
        element.style.overflow = 'hidden';
        element.textContent = `Item ${index}: ${JSON.stringify(item.data)}`;
        return element;
    }
    
    scrollToIndex(index: number): void {
        const scrollTop = index * this.itemHeight;
        this.container.scrollTop = scrollTop;
    }
    
    getVisibleRange(): { start: number; end: number } {
        const start = Math.floor(this.scrollTop / this.itemHeight);
        const end = Math.min(
            this.items.length - 1,
            Math.floor((this.scrollTop + this.containerHeight) / this.itemHeight)
        );
        return { start, end };
    }
}

// Optimized DOM manipulation
class DOMOptimizer {
    private documentFragment = document.createDocumentFragment();
    private batchedOperations: Array<() => void> = [];
    private isScheduled = false;
    
    // Batch DOM updates
    batchUpdate(operation: () => void): void {
        this.batchedOperations.push(operation);
        
        if (!this.isScheduled) {
            this.isScheduled = true;
            requestAnimationFrame(() => {
                this.flush();
            });
        }
    }
    
    private flush(): void {
        this.batchedOperations.forEach(operation => operation());
        this.batchedOperations = [];
        this.isScheduled = false;
    }
    
    // Efficient element creation
    createElement<K extends keyof HTMLElementTagNameMap>(
        tagName: K,
        options: {
            className?: string;
            textContent?: string;
            attributes?: Record<string, string>;
            styles?: Partial<CSSStyleDeclaration>;
        } = {}
    ): HTMLElementTagNameMap[K] {
        const element = document.createElement(tagName);
        
        if (options.className) {
            element.className = options.className;
        }
        
        if (options.textContent) {
            element.textContent = options.textContent;
        }
        
        if (options.attributes) {
            Object.entries(options.attributes).forEach(([key, value]) => {
                element.setAttribute(key, value);
            });
        }
        
        if (options.styles) {
            Object.assign(element.style, options.styles);
        }
        
        return element;
    }
    
    // Efficient list rendering
    renderList<T>(
        container: HTMLElement,
        items: T[],
        renderItem: (item: T, index: number) => HTMLElement
    ): void {
        // Clear container
        container.innerHTML = '';
        
        // Create elements in document fragment
        items.forEach((item, index) => {
            const element = renderItem(item, index);
            this.documentFragment.appendChild(element);
        });
        
        // Append all at once
        container.appendChild(this.documentFragment);
    }
    
    // Measure performance
    measureRender<T>(operation: () => T, label: string = 'Render'): T {
        const start = performance.now();
        const result = operation();
        const end = performance.now();
        
        console.log(`${label} took ${end - start} milliseconds`);
        return result;
    }
}
```

## 📝 แบบฝึกหัด

### 1. **Data Processing Optimization**
สร้างระบบประมวลผลข้อมูลที่มีประสิทธิภาพ:
- Implement streaming data processor
- Add memory-efficient caching
- Optimize for large datasets
- Include performance monitoring

### 2. **Component Performance Optimization**
ออกแบบ component system ที่เร็ว:
- Virtual scrolling for large lists
- Lazy loading with preloading
- Efficient state management
- Optimized re-rendering

### 3. **Bundle Optimization Project**
สร้างระบบ bundle optimization:
- Code splitting strategies
- Dynamic imports
- Tree shaking implementation
- Performance budgets

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 15: Testing TypeScript](./15-testing.md)

**บทต่อไป**: [บทที่ 17: Design Patterns](./17-design-patterns.md)

**กลับไปหน้าหลัก**: [README](./README.md)
