# บทที่ 17: Design Patterns

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Design Patterns ที่สำคัญใน TypeScript
- เข้าใจ Creational, Structural, และ Behavioral Patterns
- เรียนรู้การนำ Patterns มาใช้ในโปรเจ็กต์จริง
- เข้าใจข้อดีข้อเสียของแต่ละ Pattern

## 🏭 Creational Patterns

### 1. **Singleton Pattern**

```typescript
// Thread-safe Singleton implementation
class DatabaseConnection {
    private static instance: DatabaseConnection | null = null;
    private static isCreating = false;
    private connectionString: string;
    private isConnected = false;
    
    private constructor(connectionString: string) {
        this.connectionString = connectionString;
    }
    
    static async getInstance(connectionString?: string): Promise<DatabaseConnection> {
        if (DatabaseConnection.instance) {
            return DatabaseConnection.instance;
        }
        
        // Prevent multiple instances during async creation
        if (DatabaseConnection.isCreating) {
            return new Promise((resolve) => {
                const checkInstance = () => {
                    if (DatabaseConnection.instance) {
                        resolve(DatabaseConnection.instance);
                    } else {
                        setTimeout(checkInstance, 10);
                    }
                };
                checkInstance();
            });
        }
        
        DatabaseConnection.isCreating = true;
        
        try {
            const instance = new DatabaseConnection(connectionString || 'default');
            await instance.connect();
            DatabaseConnection.instance = instance;
            return instance;
        } finally {
            DatabaseConnection.isCreating = false;
        }
    }
    
    private async connect(): Promise<void> {
        // Simulate connection
        await new Promise(resolve => setTimeout(resolve, 100));
        this.isConnected = true;
        console.log(`Connected to database: ${this.connectionString}`);
    }
    
    async query(sql: string): Promise<any[]> {
        if (!this.isConnected) {
            throw new Error('Database not connected');
        }
        
        console.log(`Executing query: ${sql}`);
        return []; // Simulate query result
    }
    
    static reset(): void {
        DatabaseConnection.instance = null;
        DatabaseConnection.isCreating = false;
    }
}

// Enum-based Singleton (สำหรับ configuration)
enum AppConfig {
    Instance
}

namespace AppConfig {
    let _config: {
        apiUrl: string;
        timeout: number;
        retries: number;
    } | null = null;
    
    export function getInstance() {
        if (!_config) {
            _config = {
                apiUrl: process.env.API_URL || 'http://localhost:3000',
                timeout: parseInt(process.env.TIMEOUT || '5000'),
                retries: parseInt(process.env.RETRIES || '3')
            };
        }
        return _config;
    }
    
    export function updateConfig(updates: Partial<typeof _config>) {
        if (_config) {
            Object.assign(_config, updates);
        }
    }
}

// การใช้งาน Singleton
async function useSingleton() {
    const db1 = await DatabaseConnection.getInstance('postgres://localhost/app');
    const db2 = await DatabaseConnection.getInstance('mysql://localhost/app');
    
    console.log(db1 === db2); // true - same instance
    
    const config = AppConfig.getInstance();
    console.log(config.apiUrl);
}
```

### 2. **Factory Pattern**

```typescript
// Abstract Factory Pattern
interface Vehicle {
    start(): void;
    stop(): void;
    getInfo(): string;
}

interface VehicleFactory {
    createCar(): Vehicle;
    createMotorcycle(): Vehicle;
    createTruck(): Vehicle;
}

// Electric Vehicle implementations
class ElectricCar implements Vehicle {
    start(): void {
        console.log('Electric car started silently');
    }
    
    stop(): void {
        console.log('Electric car stopped');
    }
    
    getInfo(): string {
        return 'Electric Car - Zero emissions';
    }
}

class ElectricMotorcycle implements Vehicle {
    start(): void {
        console.log('Electric motorcycle started');
    }
    
    stop(): void {
        console.log('Electric motorcycle stopped');
    }
    
    getInfo(): string {
        return 'Electric Motorcycle - Eco-friendly';
    }
}

class ElectricTruck implements Vehicle {
    start(): void {
        console.log('Electric truck started');
    }
    
    stop(): void {
        console.log('Electric truck stopped');
    }
    
    getInfo(): string {
        return 'Electric Truck - Heavy duty, zero emissions';
    }
}

// Gasoline Vehicle implementations
class GasolineCar implements Vehicle {
    start(): void {
        console.log('Gasoline car engine started');
    }
    
    stop(): void {
        console.log('Gasoline car engine stopped');
    }
    
    getInfo(): string {
        return 'Gasoline Car - Traditional engine';
    }
}

class GasolineMotorcycle implements Vehicle {
    start(): void {
        console.log('Gasoline motorcycle engine started');
    }
    
    stop(): void {
        console.log('Gasoline motorcycle engine stopped');
    }
    
    getInfo(): string {
        return 'Gasoline Motorcycle - High performance';
    }
}

class GasolineTruck implements Vehicle {
    start(): void {
        console.log('Gasoline truck engine started');
    }
    
    stop(): void {
        console.log('Gasoline truck engine stopped');
    }
    
    getInfo(): string {
        return 'Gasoline Truck - Heavy duty transport';
    }
}

// Factory implementations
class ElectricVehicleFactory implements VehicleFactory {
    createCar(): Vehicle {
        return new ElectricCar();
    }
    
    createMotorcycle(): Vehicle {
        return new ElectricMotorcycle();
    }
    
    createTruck(): Vehicle {
        return new ElectricTruck();
    }
}

class GasolineVehicleFactory implements VehicleFactory {
    createCar(): Vehicle {
        return new GasolineCar();
    }
    
    createMotorcycle(): Vehicle {
        return new GasolineMotorcycle();
    }
    
    createTruck(): Vehicle {
        return new GasolineTruck();
    }
}

// Factory selector
class VehicleFactorySelector {
    static getFactory(type: 'electric' | 'gasoline'): VehicleFactory {
        switch (type) {
            case 'electric':
                return new ElectricVehicleFactory();
            case 'gasoline':
                return new GasolineVehicleFactory();
            default:
                throw new Error(`Unknown vehicle type: ${type}`);
        }
    }
}

// Simple Factory Pattern สำหรับ Logger
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface Logger {
    log(level: LogLevel, message: string): void;
}

class ConsoleLogger implements Logger {
    log(level: LogLevel, message: string): void {
        console.log(`[${level.toUpperCase()}] ${new Date().toISOString()}: ${message}`);
    }
}

class FileLogger implements Logger {
    constructor(private filePath: string) {}
    
    log(level: LogLevel, message: string): void {
        const logEntry = `[${level.toUpperCase()}] ${new Date().toISOString()}: ${message}\n`;
        // In real implementation, write to file
        console.log(`Writing to ${this.filePath}: ${logEntry}`);
    }
}

class RemoteLogger implements Logger {
    constructor(private endpoint: string) {}
    
    log(level: LogLevel, message: string): void {
        const logData = {
            level,
            message,
            timestamp: new Date().toISOString()
        };
        // In real implementation, send to remote endpoint
        console.log(`Sending to ${this.endpoint}:`, logData);
    }
}

class LoggerFactory {
    static createLogger(type: 'console' | 'file' | 'remote', config?: any): Logger {
        switch (type) {
            case 'console':
                return new ConsoleLogger();
            case 'file':
                return new FileLogger(config?.filePath || './app.log');
            case 'remote':
                return new RemoteLogger(config?.endpoint || 'http://localhost:8080/logs');
            default:
                throw new Error(`Unknown logger type: ${type}`);
        }
    }
}

// การใช้งาน Factory Pattern
function useFactoryPattern() {
    // Abstract Factory
    const electricFactory = VehicleFactorySelector.getFactory('electric');
    const electricCar = electricFactory.createCar();
    electricCar.start();
    console.log(electricCar.getInfo());
    
    // Simple Factory
    const logger = LoggerFactory.createLogger('file', { filePath: './app.log' });
    logger.log('info', 'Application started');
}
```

### 3. **Builder Pattern**

```typescript
// Fluent Builder Pattern
interface DatabaseConfig {
    host: string;
    port: number;
    database: string;
    username: string;
    password: string;
    ssl: boolean;
    poolSize: number;
    timeout: number;
    retries: number;
}

class DatabaseConfigBuilder {
    private config: Partial<DatabaseConfig> = {};
    
    host(host: string): this {
        this.config.host = host;
        return this;
    }
    
    port(port: number): this {
        this.config.port = port;
        return this;
    }
    
    database(database: string): this {
        this.config.database = database;
        return this;
    }
    
    credentials(username: string, password: string): this {
        this.config.username = username;
        this.config.password = password;
        return this;
    }
    
    enableSSL(ssl: boolean = true): this {
        this.config.ssl = ssl;
        return this;
    }
    
    poolSize(size: number): this {
        this.config.poolSize = size;
        return this;
    }
    
    timeout(timeout: number): this {
        this.config.timeout = timeout;
        return this;
    }
    
    retries(retries: number): this {
        this.config.retries = retries;
        return this;
    }
    
    build(): DatabaseConfig {
        // Validate required fields
        const required: (keyof DatabaseConfig)[] = ['host', 'port', 'database', 'username', 'password'];
        for (const field of required) {
            if (this.config[field] === undefined) {
                throw new Error(`Missing required field: ${field}`);
            }
        }
        
        // Set defaults
        return {
            host: this.config.host!,
            port: this.config.port!,
            database: this.config.database!,
            username: this.config.username!,
            password: this.config.password!,
            ssl: this.config.ssl ?? false,
            poolSize: this.config.poolSize ?? 10,
            timeout: this.config.timeout ?? 5000,
            retries: this.config.retries ?? 3
        };
    }
    
    static create(): DatabaseConfigBuilder {
        return new DatabaseConfigBuilder();
    }
}

// Complex Builder Pattern สำหรับ HTTP Request
interface HttpRequestConfig {
    url: string;
    method: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
    headers: Record<string, string>;
    params: Record<string, string>;
    body?: any;
    timeout: number;
    retries: number;
    cache: boolean;
}

class HttpRequestBuilder {
    private config: Partial<HttpRequestConfig> = {
        method: 'GET',
        headers: {},
        params: {},
        timeout: 5000,
        retries: 0,
        cache: false
    };
    
    url(url: string): this {
        this.config.url = url;
        return this;
    }
    
    method(method: HttpRequestConfig['method']): this {
        this.config.method = method;
        return this;
    }
    
    header(key: string, value: string): this {
        this.config.headers![key] = value;
        return this;
    }
    
    headers(headers: Record<string, string>): this {
        this.config.headers = { ...this.config.headers, ...headers };
        return this;
    }
    
    param(key: string, value: string): this {
        this.config.params![key] = value;
        return this;
    }
    
    params(params: Record<string, string>): this {
        this.config.params = { ...this.config.params, ...params };
        return this;
    }
    
    body(body: any): this {
        this.config.body = body;
        return this;
    }
    
    json(data: any): this {
        this.config.body = JSON.stringify(data);
        this.config.headers!['Content-Type'] = 'application/json';
        return this;
    }
    
    timeout(timeout: number): this {
        this.config.timeout = timeout;
        return this;
    }
    
    retries(retries: number): this {
        this.config.retries = retries;
        return this;
    }
    
    cache(cache: boolean = true): this {
        this.config.cache = cache;
        return this;
    }
    
    auth(token: string): this {
        this.config.headers!['Authorization'] = `Bearer ${token}`;
        return this;
    }
    
    build(): HttpRequestConfig {
        if (!this.config.url) {
            throw new Error('URL is required');
        }
        
        return this.config as HttpRequestConfig;
    }
    
    async execute(): Promise<Response> {
        const config = this.build();
        const url = new URL(config.url);
        
        // Add query parameters
        Object.entries(config.params).forEach(([key, value]) => {
            url.searchParams.append(key, value);
        });
        
        const fetchConfig: RequestInit = {
            method: config.method,
            headers: config.headers,
            body: config.body
        };
        
        return fetch(url.toString(), fetchConfig);
    }
}

// การใช้งาน Builder Pattern
async function useBuilderPattern() {
    // Database config
    const dbConfig = DatabaseConfigBuilder.create()
        .host('localhost')
        .port(5432)
        .database('myapp')
        .credentials('user', 'password')
        .enableSSL(true)
        .poolSize(20)
        .timeout(10000)
        .retries(3)
        .build();
    
    console.log('Database config:', dbConfig);
    
    // HTTP request
    const response = await new HttpRequestBuilder()
        .url('https://api.example.com/users')
        .method('POST')
        .json({ name: 'John Doe', email: 'john@example.com' })
        .auth('your-jwt-token')
        .timeout(10000)
        .retries(2)
        .cache(false)
        .execute();
    
    console.log('Response status:', response.status);
}
```

## 🏗️ Structural Patterns

### 1. **Adapter Pattern**

```typescript
// Legacy API interface
interface LegacyUser {
    user_id: string;
    user_name: string;
    user_email: string;
    creation_date: string;
}

// Modern API interface
interface ModernUser {
    id: string;
    name: string;
    email: string;
    createdAt: Date;
}

// Legacy API service
class LegacyUserService {
    async getUser(id: string): Promise<LegacyUser> {
        // Simulate API call
        return {
            user_id: id,
            user_name: 'John Doe',
            user_email: 'john@example.com',
            creation_date: '2023-01-01T00:00:00Z'
        };
    }
    
    async getUsers(): Promise<LegacyUser[]> {
        return [
            {
                user_id: '1',
                user_name: 'John Doe',
                user_email: 'john@example.com',
                creation_date: '2023-01-01T00:00:00Z'
            },
            {
                user_id: '2',
                user_name: 'Jane Smith',
                user_email: 'jane@example.com',
                creation_date: '2023-01-02T00:00:00Z'
            }
        ];
    }
}

// Adapter class
class UserServiceAdapter {
    constructor(private legacyService: LegacyUserService) {}
    
    async getUser(id: string): Promise<ModernUser> {
        const legacyUser = await this.legacyService.getUser(id);
        return this.adaptUser(legacyUser);
    }
    
    async getUsers(): Promise<ModernUser[]> {
        const legacyUsers = await this.legacyService.getUsers();
        return legacyUsers.map(user => this.adaptUser(user));
    }
    
    private adaptUser(legacyUser: LegacyUser): ModernUser {
        return {
            id: legacyUser.user_id,
            name: legacyUser.user_name,
            email: legacyUser.user_email,
            createdAt: new Date(legacyUser.creation_date)
        };
    }
}

// Multiple adapters example
interface PaymentGateway {
    processPayment(amount: number, currency: string): Promise<PaymentResult>;
}

interface PaymentResult {
    transactionId: string;
    status: 'success' | 'failed' | 'pending';
    amount: number;
    currency: string;
}

// Stripe API (hypothetical)
class StripeAPI {
    async charge(amountInCents: number, currency: string): Promise<{
        id: string;
        status: 'succeeded' | 'failed' | 'pending';
        amount: number;
        currency: string;
    }> {
        return {
            id: `stripe_${Date.now()}`,
            status: 'succeeded',
            amount: amountInCents,
            currency
        };
    }
}

// PayPal API (hypothetical)
class PayPalAPI {
    async createPayment(value: number, currencyCode: string): Promise<{
        paymentId: string;
        state: 'approved' | 'failed' | 'created';
        transactions: Array<{ amount: { total: string; currency: string } }>;
    }> {
        return {
            paymentId: `paypal_${Date.now()}`,
            state: 'approved',
            transactions: [{
                amount: {
                    total: value.toString(),
                    currency: currencyCode
                }
            }]
        };
    }
}

// Adapters
class StripeAdapter implements PaymentGateway {
    constructor(private stripeAPI: StripeAPI) {}
    
    async processPayment(amount: number, currency: string): Promise<PaymentResult> {
        const amountInCents = Math.round(amount * 100);
        const result = await this.stripeAPI.charge(amountInCents, currency);
        
        return {
            transactionId: result.id,
            status: result.status === 'succeeded' ? 'success' : 
                   result.status === 'failed' ? 'failed' : 'pending',
            amount: result.amount / 100,
            currency: result.currency
        };
    }
}

class PayPalAdapter implements PaymentGateway {
    constructor(private paypalAPI: PayPalAPI) {}
    
    async processPayment(amount: number, currency: string): Promise<PaymentResult> {
        const result = await this.paypalAPI.createPayment(amount, currency);
        
        return {
            transactionId: result.paymentId,
            status: result.state === 'approved' ? 'success' : 
                   result.state === 'failed' ? 'failed' : 'pending',
            amount: parseFloat(result.transactions[0].amount.total),
            currency: result.transactions[0].amount.currency
        };
    }
}

// Payment processor ที่ใช้ adapters
class PaymentProcessor {
    private gateways = new Map<string, PaymentGateway>();
    
    registerGateway(name: string, gateway: PaymentGateway): void {
        this.gateways.set(name, gateway);
    }
    
    async processPayment(
        gatewayName: string, 
        amount: number, 
        currency: string
    ): Promise<PaymentResult> {
        const gateway = this.gateways.get(gatewayName);
        if (!gateway) {
            throw new Error(`Unknown payment gateway: ${gatewayName}`);
        }
        
        return gateway.processPayment(amount, currency);
    }
    
    getAvailableGateways(): string[] {
        return Array.from(this.gateways.keys());
    }
}
```

### 2. **Decorator Pattern**

```typescript
// Component interface
interface Coffee {
    getDescription(): string;
    getCost(): number;
}

// Basic coffee implementation
class BasicCoffee implements Coffee {
    getDescription(): string {
        return 'Basic Coffee';
    }
    
    getCost(): number {
        return 2.00;
    }
}

// Abstract decorator
abstract class CoffeeDecorator implements Coffee {
    constructor(protected coffee: Coffee) {}
    
    getDescription(): string {
        return this.coffee.getDescription();
    }
    
    getCost(): number {
        return this.coffee.getCost();
    }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    getDescription(): string {
        return `${this.coffee.getDescription()}, Milk`;
    }
    
    getCost(): number {
        return this.coffee.getCost() + 0.50;
    }
}

class SugarDecorator extends CoffeeDecorator {
    getDescription(): string {
        return `${this.coffee.getDescription()}, Sugar`;
    }
    
    getCost(): number {
        return this.coffee.getCost() + 0.25;
    }
}

class WhipDecorator extends CoffeeDecorator {
    getDescription(): string {
        return `${this.coffee.getDescription()}, Whipped Cream`;
    }
    
    getCost(): number {
        return this.coffee.getCost() + 0.75;
    }
}

class VanillaDecorator extends CoffeeDecorator {
    getDescription(): string {
        return `${this.coffee.getDescription()}, Vanilla`;
    }
    
    getCost(): number {
        return this.coffee.getCost() + 0.60;
    }
}

// Function-based decorator pattern for API calls
interface APICall {
    execute(): Promise<any>;
}

class BasicAPICall implements APICall {
    constructor(private url: string, private options: RequestInit = {}) {}
    
    async execute(): Promise<any> {
        const response = await fetch(this.url, this.options);
        return response.json();
    }
}

// API call decorators
function withRetry(retries: number = 3, delay: number = 1000) {
    return function<T extends APICall>(apiCall: T): T {
        const originalExecute = apiCall.execute.bind(apiCall);
        
        apiCall.execute = async function(): Promise<any> {
            let lastError: Error;
            
            for (let attempt = 1; attempt <= retries; attempt++) {
                try {
                    return await originalExecute();
                } catch (error) {
                    lastError = error instanceof Error ? error : new Error('Unknown error');
                    
                    if (attempt === retries) {
                        throw lastError;
                    }
                    
                    await new Promise(resolve => setTimeout(resolve, delay * attempt));
                }
            }
            
            throw lastError!;
        };
        
        return apiCall;
    };
}

function withLogging() {
    return function<T extends APICall>(apiCall: T): T {
        const originalExecute = apiCall.execute.bind(apiCall);
        
        apiCall.execute = async function(): Promise<any> {
            console.log('API call started');
            const start = Date.now();
            
            try {
                const result = await originalExecute();
                const duration = Date.now() - start;
                console.log(`API call completed in ${duration}ms`);
                return result;
            } catch (error) {
                const duration = Date.now() - start;
                console.log(`API call failed after ${duration}ms:`, error);
                throw error;
            }
        };
        
        return apiCall;
    };
}

function withCache(ttl: number = 60000) {
    const cache = new Map<string, { data: any; expiry: number }>();
    
    return function<T extends APICall>(apiCall: T): T {
        const originalExecute = apiCall.execute.bind(apiCall);
        
        apiCall.execute = async function(): Promise<any> {
            // Simple cache key - in real app, would be more sophisticated
            const cacheKey = JSON.stringify(arguments);
            const cached = cache.get(cacheKey);
            
            if (cached && cached.expiry > Date.now()) {
                console.log('Cache hit');
                return cached.data;
            }
            
            const result = await originalExecute();
            cache.set(cacheKey, {
                data: result,
                expiry: Date.now() + ttl
            });
            
            return result;
        };
        
        return apiCall;
    };
}

// การใช้งาน Decorator Pattern
function useDecoratorPattern() {
    // Coffee example
    let coffee: Coffee = new BasicCoffee();
    console.log(`${coffee.getDescription()} - $${coffee.getCost()}`);
    
    coffee = new MilkDecorator(coffee);
    coffee = new SugarDecorator(coffee);
    coffee = new WhipDecorator(coffee);
    
    console.log(`${coffee.getDescription()} - $${coffee.getCost()}`);
    
    // API call example
    let apiCall: APICall = new BasicAPICall('https://api.example.com/data');
    
    // Apply decorators
    apiCall = withLogging()(apiCall);
    apiCall = withRetry(3, 1000)(apiCall);
    apiCall = withCache(30000)(apiCall);
    
    // Execute
    apiCall.execute().then(result => {
        console.log('API result:', result);
    });
}
```

### 3. **Facade Pattern**

```typescript
// Complex subsystem classes
class EmailService {
    async sendEmail(to: string, subject: string, body: string): Promise<void> {
        console.log(`Sending email to ${to}: ${subject}`);
        // Complex email sending logic
        await new Promise(resolve => setTimeout(resolve, 100));
    }
}

class SMSService {
    async sendSMS(phoneNumber: string, message: string): Promise<void> {
        console.log(`Sending SMS to ${phoneNumber}: ${message}`);
        // Complex SMS sending logic
        await new Promise(resolve => setTimeout(resolve, 100));
    }
}

class PushNotificationService {
    async sendPushNotification(deviceId: string, title: string, body: string): Promise<void> {
        console.log(`Sending push notification to ${deviceId}: ${title}`);
        // Complex push notification logic
        await new Promise(resolve => setTimeout(resolve, 100));
    }
}

class UserPreferenceService {
    async getUserPreferences(userId: string): Promise<{
        email: boolean;
        sms: boolean;
        push: boolean;
        emailAddress?: string;
        phoneNumber?: string;
        deviceId?: string;
    }> {
        // Simulate database lookup
        return {
            email: true,
            sms: false,
            push: true,
            emailAddress: 'user@example.com',
            phoneNumber: '+1234567890',
            deviceId: 'device123'
        };
    }
}

class TemplateService {
    getEmailTemplate(type: string): { subject: string; body: string } {
        return {
            subject: `Welcome ${type}`,
            body: `Welcome to our ${type} service!`
        };
    }
    
    getSMSTemplate(type: string): string {
        return `Welcome to ${type}!`;
    }
    
    getPushTemplate(type: string): { title: string; body: string } {
        return {
            title: `${type} Welcome`,
            body: `You're now part of ${type}!`
        };
    }
}

// Facade class
class NotificationFacade {
    private emailService = new EmailService();
    private smsService = new SMSService();
    private pushService = new PushNotificationService();
    private preferenceService = new UserPreferenceService();
    private templateService = new TemplateService();
    
    async sendWelcomeNotification(userId: string, serviceType: string): Promise<void> {
        try {
            // Get user preferences
            const preferences = await this.preferenceService.getUserPreferences(userId);
            
            // Send email if preferred
            if (preferences.email && preferences.emailAddress) {
                const template = this.templateService.getEmailTemplate(serviceType);
                await this.emailService.sendEmail(
                    preferences.emailAddress,
                    template.subject,
                    template.body
                );
            }
            
            // Send SMS if preferred
            if (preferences.sms && preferences.phoneNumber) {
                const message = this.templateService.getSMSTemplate(serviceType);
                await this.smsService.sendSMS(preferences.phoneNumber, message);
            }
            
            // Send push notification if preferred
            if (preferences.push && preferences.deviceId) {
                const template = this.templateService.getPushTemplate(serviceType);
                await this.pushService.sendPushNotification(
                    preferences.deviceId,
                    template.title,
                    template.body
                );
            }
            
            console.log(`Welcome notification sent for ${serviceType}`);
        } catch (error) {
            console.error('Failed to send welcome notification:', error);
            throw error;
        }
    }
    
    async sendCustomNotification(
        userId: string,
        message: {
            email?: { subject: string; body: string };
            sms?: string;
            push?: { title: string; body: string };
        }
    ): Promise<void> {
        const preferences = await this.preferenceService.getUserPreferences(userId);
        
        const promises: Promise<void>[] = [];
        
        if (message.email && preferences.email && preferences.emailAddress) {
            promises.push(
                this.emailService.sendEmail(
                    preferences.emailAddress,
                    message.email.subject,
                    message.email.body
                )
            );
        }
        
        if (message.sms && preferences.sms && preferences.phoneNumber) {
            promises.push(
                this.smsService.sendSMS(preferences.phoneNumber, message.sms)
            );
        }
        
        if (message.push && preferences.push && preferences.deviceId) {
            promises.push(
                this.pushService.sendPushNotification(
                    preferences.deviceId,
                    message.push.title,
                    message.push.body
                )
            );
        }
        
        await Promise.all(promises);
    }
}

// E-commerce Facade example
class InventoryService {
    async checkStock(productId: string): Promise<number> {
        // Complex inventory checking logic
        return Math.floor(Math.random() * 100);
    }
    
    async reserveStock(productId: string, quantity: number): Promise<string> {
        return `reservation_${Date.now()}`;
    }
}

class PaymentService {
    async processPayment(amount: number, paymentMethod: string): Promise<string> {
        // Complex payment processing
        return `payment_${Date.now()}`;
    }
}

class ShippingService {
    async calculateShipping(address: string, weight: number): Promise<number> {
        // Complex shipping calculation
        return 10.00;
    }
    
    async createShipment(orderId: string, address: string): Promise<string> {
        return `shipment_${Date.now()}`;
    }
}

class OrderService {
    async createOrder(orderData: any): Promise<string> {
        return `order_${Date.now()}`;
    }
}

class ECommerceFacade {
    private inventory = new InventoryService();
    private payment = new PaymentService();
    private shipping = new ShippingService();
    private orders = new OrderService();
    
    async purchaseProduct(
        productId: string,
        quantity: number,
        paymentMethod: string,
        shippingAddress: string,
        productWeight: number
    ): Promise<{
        orderId: string;
        paymentId: string;
        shipmentId: string;
        totalCost: number;
    }> {
        // Check stock availability
        const stock = await this.inventory.checkStock(productId);
        if (stock < quantity) {
            throw new Error('Insufficient stock');
        }
        
        // Reserve stock
        const reservationId = await this.inventory.reserveStock(productId, quantity);
        
        try {
            // Calculate shipping
            const shippingCost = await this.shipping.calculateShipping(shippingAddress, productWeight);
            
            // Process payment
            const productCost = quantity * 25.00; // Mock price
            const totalCost = productCost + shippingCost;
            const paymentId = await this.payment.processPayment(totalCost, paymentMethod);
            
            // Create order
            const orderId = await this.orders.createOrder({
                productId,
                quantity,
                totalCost,
                paymentId,
                reservationId
            });
            
            // Create shipment
            const shipmentId = await this.shipping.createShipment(orderId, shippingAddress);
            
            return {
                orderId,
                paymentId,
                shipmentId,
                totalCost
            };
        } catch (error) {
            // Handle rollback in case of failure
            console.error('Purchase failed, rolling back reservation');
            throw error;
        }
    }
}
```

## 🎭 Behavioral Patterns

### 1. **Observer Pattern**

```typescript
// Subject interface
interface Subject<T> {
    attach(observer: Observer<T>): void;
    detach(observer: Observer<T>): void;
    notify(data: T): void;
}

// Observer interface
interface Observer<T> {
    update(data: T): void;
}

// Event data types
interface UserEvent {
    type: 'login' | 'logout' | 'signup' | 'profile_update';
    userId: string;
    timestamp: Date;
    data?: any;
}

interface SystemEvent {
    type: 'error' | 'warning' | 'info';
    message: string;
    source: string;
    timestamp: Date;
}

// Observable class
class Observable<T> implements Subject<T> {
    private observers: Set<Observer<T>> = new Set();
    
    attach(observer: Observer<T>): void {
        this.observers.add(observer);
    }
    
    detach(observer: Observer<T>): void {
        this.observers.delete(observer);
    }
    
    notify(data: T): void {
        this.observers.forEach(observer => {
            try {
                observer.update(data);
            } catch (error) {
                console.error('Error in observer:', error);
            }
        });
    }
    
    getObserverCount(): number {
        return this.observers.size;
    }
}

// Concrete observers
class UserActivityLogger implements Observer<UserEvent> {
    update(event: UserEvent): void {
        console.log(`[USER LOG] ${event.type} - User ${event.userId} at ${event.timestamp.toISOString()}`);
        
        // In real app, would write to log file or database
        const logEntry = {
            timestamp: event.timestamp,
            level: 'info',
            message: `User ${event.userId} performed ${event.type}`,
            metadata: event.data
        };
        
        // Simulate writing to log
        this.writeToLog(logEntry);
    }
    
    private writeToLog(entry: any): void {
        // Implementation would write to actual log system
        console.log('Writing to log:', entry);
    }
}

class EmailNotificationService implements Observer<UserEvent> {
    update(event: UserEvent): void {
        if (event.type === 'signup') {
            this.sendWelcomeEmail(event.userId);
        } else if (event.type === 'login') {
            this.sendLoginAlert(event.userId);
        }
    }
    
    private async sendWelcomeEmail(userId: string): Promise<void> {
        console.log(`Sending welcome email to user ${userId}`);
        // Email sending implementation
    }
    
    private async sendLoginAlert(userId: string): Promise<void> {
        console.log(`Sending login alert to user ${userId}`);
        // Email sending implementation
    }
}

class SecurityMonitor implements Observer<UserEvent> {
    private suspiciousActivities = new Map<string, number>();
    
    update(event: UserEvent): void {
        if (event.type === 'login') {
            this.checkSuspiciousActivity(event.userId);
        }
    }
    
    private checkSuspiciousActivity(userId: string): void {
        const count = this.suspiciousActivities.get(userId) || 0;
        this.suspiciousActivities.set(userId, count + 1);
        
        // Check for suspicious login patterns
        if (count > 5) {
            console.log(`SECURITY ALERT: Suspicious activity detected for user ${userId}`);
            // Trigger security measures
        }
    }
}

class AnalyticsCollector implements Observer<UserEvent> {
    private events: UserEvent[] = [];
    
    update(event: UserEvent): void {
        this.events.push(event);
        
        // Send to analytics service in batches
        if (this.events.length >= 10) {
            this.sendToAnalytics();
        }
    }
    
    private sendToAnalytics(): void {
        console.log(`Sending ${this.events.length} events to analytics service`);
        // Implementation would send to analytics service
        this.events = [];
    }
}

// Event emitter with typed events
class TypedEventEmitter<EventMap extends Record<string, any>> {
    private listeners = new Map<keyof EventMap, Set<Function>>();
    
    on<K extends keyof EventMap>(event: K, listener: (data: EventMap[K]) => void): () => void {
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
    
    emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
        const listeners = this.listeners.get(event);
        if (listeners) {
            listeners.forEach(listener => {
                try {
                    listener(data);
                } catch (error) {
                    console.error(`Error in listener for ${String(event)}:`, error);
                }
            });
        }
    }
    
    off<K extends keyof EventMap>(event: K, listener?: Function): void {
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
    
    once<K extends keyof EventMap>(event: K, listener: (data: EventMap[K]) => void): void {
        const onceWrapper = (data: EventMap[K]) => {
            listener(data);
            this.off(event, onceWrapper);
        };
        
        this.on(event, onceWrapper);
    }
    
    listenerCount<K extends keyof EventMap>(event: K): number {
        const listeners = this.listeners.get(event);
        return listeners ? listeners.size : 0;
    }
}

// การใช้งาน Observer Pattern
function useObserverPattern() {
    // Traditional Observer Pattern
    const userEventSubject = new Observable<UserEvent>();
    
    // Attach observers
    const logger = new UserActivityLogger();
    const emailService = new EmailNotificationService();
    const securityMonitor = new SecurityMonitor();
    const analytics = new AnalyticsCollector();
    
    userEventSubject.attach(logger);
    userEventSubject.attach(emailService);
    userEventSubject.attach(securityMonitor);
    userEventSubject.attach(analytics);
    
    // Emit events
    userEventSubject.notify({
        type: 'signup',
        userId: 'user123',
        timestamp: new Date(),
        data: { email: 'user@example.com' }
    });
    
    userEventSubject.notify({
        type: 'login',
        userId: 'user123',
        timestamp: new Date()
    });
    
    // Typed Event Emitter
    interface AppEvents {
        'user:login': { userId: string; timestamp: Date };
        'user:logout': { userId: string; timestamp: Date };
        'order:created': { orderId: string; userId: string; amount: number };
        'payment:processed': { paymentId: string; orderId: string; amount: number };
    }
    
    const eventEmitter = new TypedEventEmitter<AppEvents>();
    
    // Subscribe to events
    const unsubscribeLogin = eventEmitter.on('user:login', (data) => {
        console.log(`User ${data.userId} logged in at ${data.timestamp}`);
    });
    
    eventEmitter.on('order:created', (data) => {
        console.log(`Order ${data.orderId} created for $${data.amount}`);
    });
    
    // Emit events
    eventEmitter.emit('user:login', { userId: 'user456', timestamp: new Date() });
    eventEmitter.emit('order:created', { orderId: 'order789', userId: 'user456', amount: 99.99 });
    
    // Unsubscribe
    unsubscribeLogin();
}
```

### 2. **Strategy Pattern**

```typescript
// Strategy interface
interface SortingStrategy<T> {
    sort(items: T[]): T[];
}

// Concrete strategies
class BubbleSortStrategy<T> implements SortingStrategy<T> {
    sort(items: T[]): T[] {
        const arr = [...items];
        const n = arr.length;
        
        for (let i = 0; i < n - 1; i++) {
            for (let j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                }
            }
        }
        
        return arr;
    }
}

class QuickSortStrategy<T> implements SortingStrategy<T> {
    sort(items: T[]): T[] {
        if (items.length <= 1) return [...items];
        
        const arr = [...items];
        this.quickSort(arr, 0, arr.length - 1);
        return arr;
    }
    
    private quickSort(arr: T[], low: number, high: number): void {
        if (low < high) {
            const pi = this.partition(arr, low, high);
            this.quickSort(arr, low, pi - 1);
            this.quickSort(arr, pi + 1, high);
        }
    }
    
    private partition(arr: T[], low: number, high: number): number {
        const pivot = arr[high];
        let i = low - 1;
        
        for (let j = low; j < high; j++) {
            if (arr[j] <= pivot) {
                i++;
                [arr[i], arr[j]] = [arr[j], arr[i]];
            }
        }
        
        [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
        return i + 1;
    }
}

class MergeSortStrategy<T> implements SortingStrategy<T> {
    sort(items: T[]): T[] {
        if (items.length <= 1) return [...items];
        
        return this.mergeSort([...items]);
    }
    
    private mergeSort(arr: T[]): T[] {
        if (arr.length <= 1) return arr;
        
        const mid = Math.floor(arr.length / 2);
        const left = this.mergeSort(arr.slice(0, mid));
        const right = this.mergeSort(arr.slice(mid));
        
        return this.merge(left, right);
    }
    
    private merge(left: T[], right: T[]): T[] {
        const result: T[] = [];
        let leftIndex = 0;
        let rightIndex = 0;
        
        while (leftIndex < left.length && rightIndex < right.length) {
            if (left[leftIndex] <= right[rightIndex]) {
                result.push(left[leftIndex]);
                leftIndex++;
            } else {
                result.push(right[rightIndex]);
                rightIndex++;
            }
        }
        
        return result.concat(left.slice(leftIndex)).concat(right.slice(rightIndex));
    }
}

// Context class
class Sorter<T> {
    private strategy: SortingStrategy<T>;
    
    constructor(strategy: SortingStrategy<T>) {
        this.strategy = strategy;
    }
    
    setStrategy(strategy: SortingStrategy<T>): void {
        this.strategy = strategy;
    }
    
    sort(items: T[]): T[] {
        return this.strategy.sort(items);
    }
}

// Payment strategy example
interface PaymentStrategy {
    pay(amount: number): Promise<PaymentResult>;
    validatePayment(paymentData: any): boolean;
}

interface PaymentResult {
    success: boolean;
    transactionId?: string;
    error?: string;
}

class CreditCardPayment implements PaymentStrategy {
    constructor(
        private cardNumber: string,
        private expiryDate: string,
        private cvv: string
    ) {}
    
    validatePayment(paymentData: any): boolean {
        return !!(paymentData.cardNumber && paymentData.expiryDate && paymentData.cvv);
    }
    
    async pay(amount: number): Promise<PaymentResult> {
        if (!this.validatePayment({ 
            cardNumber: this.cardNumber, 
            expiryDate: this.expiryDate, 
            cvv: this.cvv 
        })) {
            return { success: false, error: 'Invalid credit card data' };
        }
        
        // Simulate payment processing
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        return {
            success: true,
            transactionId: `cc_${Date.now()}`
        };
    }
}

class PayPalPayment implements PaymentStrategy {
    constructor(private email: string, private password: string) {}
    
    validatePayment(paymentData: any): boolean {
        return !!(paymentData.email && paymentData.password);
    }
    
    async pay(amount: number): Promise<PaymentResult> {
        if (!this.validatePayment({ email: this.email, password: this.password })) {
            return { success: false, error: 'Invalid PayPal credentials' };
        }
        
        // Simulate PayPal payment
        await new Promise(resolve => setTimeout(resolve, 1500));
        
        return {
            success: true,
            transactionId: `pp_${Date.now()}`
        };
    }
}

class BankTransferPayment implements PaymentStrategy {
    constructor(private accountNumber: string, private routingNumber: string) {}
    
    validatePayment(paymentData: any): boolean {
        return !!(paymentData.accountNumber && paymentData.routingNumber);
    }
    
    async pay(amount: number): Promise<PaymentResult> {
        if (!this.validatePayment({ 
            accountNumber: this.accountNumber, 
            routingNumber: this.routingNumber 
        })) {
            return { success: false, error: 'Invalid bank account data' };
        }
        
        // Simulate bank transfer
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        return {
            success: true,
            transactionId: `bt_${Date.now()}`
        };
    }
}

// Payment processor
class PaymentProcessor {
    private strategy: PaymentStrategy;
    
    constructor(strategy: PaymentStrategy) {
        this.strategy = strategy;
    }
    
    setPaymentStrategy(strategy: PaymentStrategy): void {
        this.strategy = strategy;
    }
    
    async processPayment(amount: number): Promise<PaymentResult> {
        console.log(`Processing payment of $${amount}`);
        return this.strategy.pay(amount);
    }
}

// Discount strategy example
interface DiscountStrategy {
    calculateDiscount(originalPrice: number): number;
    getDescription(): string;
}

class NoDiscountStrategy implements DiscountStrategy {
    calculateDiscount(originalPrice: number): number {
        return originalPrice;
    }
    
    getDescription(): string {
        return 'No discount applied';
    }
}

class PercentageDiscountStrategy implements DiscountStrategy {
    constructor(private percentage: number) {}
    
    calculateDiscount(originalPrice: number): number {
        return originalPrice * (1 - this.percentage / 100);
    }
    
    getDescription(): string {
        return `${this.percentage}% discount applied`;
    }
}

class FixedAmountDiscountStrategy implements DiscountStrategy {
    constructor(private amount: number) {}
    
    calculateDiscount(originalPrice: number): number {
        return Math.max(0, originalPrice - this.amount);
    }
    
    getDescription(): string {
        return `$${this.amount} discount applied`;
    }
}

class BuyOneGetOneStrategy implements DiscountStrategy {
    calculateDiscount(originalPrice: number): number {
        return originalPrice * 0.5; // 50% off for BOGO
    }
    
    getDescription(): string {
        return 'Buy One Get One Free';
    }
}

// Shopping cart with discount strategy
class ShoppingCart {
    private items: Array<{ name: string; price: number; quantity: number }> = [];
    private discountStrategy: DiscountStrategy = new NoDiscountStrategy();
    
    addItem(name: string, price: number, quantity: number = 1): void {
        this.items.push({ name, price, quantity });
    }
    
    setDiscountStrategy(strategy: DiscountStrategy): void {
        this.discountStrategy = strategy;
    }
    
    getTotal(): { originalTotal: number; discountedTotal: number; savings: number; description: string } {
        const originalTotal = this.items.reduce(
            (sum, item) => sum + (item.price * item.quantity), 
            0
        );
        
        const discountedTotal = this.discountStrategy.calculateDiscount(originalTotal);
        const savings = originalTotal - discountedTotal;
        
        return {
            originalTotal,
            discountedTotal,
            savings,
            description: this.discountStrategy.getDescription()
        };
    }
    
    getItems(): Array<{ name: string; price: number; quantity: number }> {
        return [...this.items];
    }
    
    clear(): void {
        this.items = [];
    }
}

// การใช้งาน Strategy Pattern
async function useStrategyPattern() {
    // Sorting strategies
    const numbers = [64, 34, 25, 12, 22, 11, 90];
    
    const sorter = new Sorter(new QuickSortStrategy<number>());
    console.log('Quick sort:', sorter.sort(numbers));
    
    sorter.setStrategy(new MergeSortStrategy<number>());
    console.log('Merge sort:', sorter.sort(numbers));
    
    // Payment strategies
    const paymentProcessor = new PaymentProcessor(
        new CreditCardPayment('1234-5678-9012-3456', '12/25', '123')
    );
    
    let result = await paymentProcessor.processPayment(100);
    console.log('Credit card payment:', result);
    
    paymentProcessor.setPaymentStrategy(
        new PayPalPayment('user@example.com', 'password')
    );
    
    result = await paymentProcessor.processPayment(100);
    console.log('PayPal payment:', result);
    
    // Shopping cart with discount strategies
    const cart = new ShoppingCart();
    cart.addItem('T-Shirt', 25, 2);
    cart.addItem('Jeans', 60, 1);
    
    console.log('No discount:', cart.getTotal());
    
    cart.setDiscountStrategy(new PercentageDiscountStrategy(20));
    console.log('20% discount:', cart.getTotal());
    
    cart.setDiscountStrategy(new FixedAmountDiscountStrategy(15));
    console.log('$15 off:', cart.getTotal());
    
    cart.setDiscountStrategy(new BuyOneGetOneStrategy());
    console.log('BOGO:', cart.getTotal());
}
```

## 📝 แบบฝึกหัด

### 1. **E-commerce System Design**
ออกแบบระบบ e-commerce ด้วย design patterns:
- ใช้ Factory Pattern สำหรับ product creation
- ใช้ Observer Pattern สำหรับ order notifications
- ใช้ Strategy Pattern สำหรับ shipping calculations
- ใช้ Decorator Pattern สำหรับ product add-ons

### 2. **Logging System**
สร้างระบบ logging ที่ยืดหยุ่น:
- ใช้ Builder Pattern สำหรับ log configuration
- ใช้ Adapter Pattern สำหรับ different log destinations
- ใช้ Facade Pattern สำหรับ simplified logging interface
- ใช้ Observer Pattern สำหรับ log event handling

### 3. **Game Development Framework**
ออกแบบ framework สำหรับเกม:
- ใช้ Singleton Pattern สำหรับ game state
- ใช้ Strategy Pattern สำหรับ AI behaviors
- ใช้ Observer Pattern สำหรับ game events
- ใช้ Factory Pattern สำหรับ entity creation

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 16: Performance Optimization](./16-performance.md)

**บทต่อไป**: [บทที่ 18: Best Practices](./18-best-practices.md)

**กลับไปหน้าหลัก**: [README](./README.md)
