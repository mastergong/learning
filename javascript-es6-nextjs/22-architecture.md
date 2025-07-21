# บทที่ 22: Architecture

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ software architecture patterns และ principles
- ทำความเข้าใจ microservices และ distributed systems
- เรียนรู้ scalable architecture design สำหรับ Next.js
- สร้าง maintainable และ testable architecture

## 🏗️ Architectural Patterns

### **🎯 Clean Architecture**

```javascript
// ============= CLEAN ARCHITECTURE =============
// Domain Layer - Business Logic
// domain/entities/user.js

export class User {
  constructor(id, name, email, createdAt = new Date()) {
    this.validateInput(id, name, email);
    
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = createdAt;
    this.updatedAt = new Date();
    this.isActive = true;
  }
  
  validateInput(id, name, email) {
    if (!id || typeof id !== 'string') {
      throw new Error('User ID is required and must be a string');
    }
    
    if (!name || name.length < 2) {
      throw new Error('User name is required and must be at least 2 characters');
    }
    
    if (!email || !this.isValidEmail(email)) {
      throw new Error('Valid email is required');
    }
  }
  
  isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  updateName(newName) {
    if (!newName || newName.length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    
    this.name = newName;
    this.updatedAt = new Date();
  }
  
  updateEmail(newEmail) {
    if (!this.isValidEmail(newEmail)) {
      throw new Error('Invalid email format');
    }
    
    this.email = newEmail;
    this.updatedAt = new Date();
  }
  
  deactivate() {
    this.isActive = false;
    this.updatedAt = new Date();
  }
  
  activate() {
    this.isActive = true;
    this.updatedAt = new Date();
  }
  
  toJSON() {
    return {
      id: this.id,
      name: this.name,
      email: this.email,
      createdAt: this.createdAt,
      updatedAt: this.updatedAt,
      isActive: this.isActive
    };
  }
}

// domain/repositories/user-repository.js
export class UserRepository {
  async create(user) {
    throw new Error('create method must be implemented');
  }
  
  async findById(id) {
    throw new Error('findById method must be implemented');
  }
  
  async findByEmail(email) {
    throw new Error('findByEmail method must be implemented');
  }
  
  async update(user) {
    throw new Error('update method must be implemented');
  }
  
  async delete(id) {
    throw new Error('delete method must be implemented');
  }
  
  async findAll(filters = {}) {
    throw new Error('findAll method must be implemented');
  }
}

// Application Layer - Use Cases
// application/use-cases/create-user.js
export class CreateUserUseCase {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }
  
  async execute(userData) {
    try {
      // Check if user already exists
      const existingUser = await this.userRepository.findByEmail(userData.email);
      if (existingUser) {
        throw new Error('User with this email already exists');
      }
      
      // Create new user
      const user = new User(
        this.generateUserId(),
        userData.name,
        userData.email
      );
      
      // Save user
      const savedUser = await this.userRepository.create(user);
      
      // Send welcome email
      await this.emailService.sendWelcomeEmail(savedUser.email, savedUser.name);
      
      return {
        success: true,
        user: savedUser.toJSON()
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
  
  generateUserId() {
    return `user_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// application/use-cases/get-user.js
export class GetUserUseCase {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }
  
  async execute(userId) {
    try {
      const user = await this.userRepository.findById(userId);
      
      if (!user) {
        throw new Error('User not found');
      }
      
      return {
        success: true,
        user: user.toJSON()
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
}

// Infrastructure Layer - External Dependencies
// infrastructure/repositories/prisma-user-repository.js
import { PrismaClient } from '@prisma/client';
import { UserRepository } from '../../domain/repositories/user-repository.js';
import { User } from '../../domain/entities/user.js';

export class PrismaUserRepository extends UserRepository {
  constructor() {
    super();
    this.prisma = new PrismaClient();
  }
  
  async create(user) {
    const savedUser = await this.prisma.user.create({
      data: {
        id: user.id,
        name: user.name,
        email: user.email,
        isActive: user.isActive,
        createdAt: user.createdAt,
        updatedAt: user.updatedAt
      }
    });
    
    return this.toDomain(savedUser);
  }
  
  async findById(id) {
    const user = await this.prisma.user.findUnique({
      where: { id }
    });
    
    return user ? this.toDomain(user) : null;
  }
  
  async findByEmail(email) {
    const user = await this.prisma.user.findUnique({
      where: { email }
    });
    
    return user ? this.toDomain(user) : null;
  }
  
  async update(user) {
    const updatedUser = await this.prisma.user.update({
      where: { id: user.id },
      data: {
        name: user.name,
        email: user.email,
        isActive: user.isActive,
        updatedAt: user.updatedAt
      }
    });
    
    return this.toDomain(updatedUser);
  }
  
  async delete(id) {
    await this.prisma.user.delete({
      where: { id }
    });
    
    return true;
  }
  
  async findAll(filters = {}) {
    const users = await this.prisma.user.findMany({
      where: filters,
      orderBy: { createdAt: 'desc' }
    });
    
    return users.map(user => this.toDomain(user));
  }
  
  toDomain(prismaUser) {
    return new User(
      prismaUser.id,
      prismaUser.name,
      prismaUser.email,
      prismaUser.createdAt
    );
  }
}

// infrastructure/services/email-service.js
export class EmailService {
  constructor(emailProvider) {
    this.emailProvider = emailProvider;
  }
  
  async sendWelcomeEmail(email, name) {
    const template = this.getWelcomeTemplate(name);
    
    return this.emailProvider.send({
      to: email,
      subject: 'Welcome to our platform!',
      html: template
    });
  }
  
  getWelcomeTemplate(name) {
    return `
      <html>
        <body>
          <h1>Welcome ${name}!</h1>
          <p>Thank you for joining our platform.</p>
        </body>
      </html>
    `;
  }
}

// Interface Layer - Controllers
// interface/controllers/user-controller.js
import { CreateUserUseCase } from '../../application/use-cases/create-user.js';
import { GetUserUseCase } from '../../application/use-cases/get-user.js';

export class UserController {
  constructor(userRepository, emailService) {
    this.createUserUseCase = new CreateUserUseCase(userRepository, emailService);
    this.getUserUseCase = new GetUserUseCase(userRepository);
  }
  
  async createUser(req, res) {
    const { name, email } = req.body;
    
    if (!name || !email) {
      return res.status(400).json({
        error: 'Name and email are required'
      });
    }
    
    const result = await this.createUserUseCase.execute({ name, email });
    
    if (result.success) {
      return res.status(201).json(result.user);
    } else {
      return res.status(400).json({
        error: result.error
      });
    }
  }
  
  async getUser(req, res) {
    const { id } = req.params;
    
    if (!id) {
      return res.status(400).json({
        error: 'User ID is required'
      });
    }
    
    const result = await this.getUserUseCase.execute(id);
    
    if (result.success) {
      return res.status(200).json(result.user);
    } else {
      return res.status(404).json({
        error: result.error
      });
    }
  }
}

// Dependency Injection Container
// container/container.js
export class Container {
  constructor() {
    this.services = new Map();
    this.singletons = new Map();
  }
  
  register(name, factory, options = {}) {
    this.services.set(name, {
      factory,
      singleton: options.singleton || false
    });
  }
  
  resolve(name) {
    const service = this.services.get(name);
    
    if (!service) {
      throw new Error(`Service ${name} not found`);
    }
    
    if (service.singleton) {
      if (!this.singletons.has(name)) {
        this.singletons.set(name, service.factory(this));
      }
      return this.singletons.get(name);
    }
    
    return service.factory(this);
  }
  
  clear() {
    this.services.clear();
    this.singletons.clear();
  }
}

// Setup dependencies
// setup/container-setup.js
import { Container } from '../container/container.js';
import { PrismaUserRepository } from '../infrastructure/repositories/prisma-user-repository.js';
import { EmailService } from '../infrastructure/services/email-service.js';
import { UserController } from '../interface/controllers/user-controller.js';

export function setupContainer() {
  const container = new Container();
  
  // Register repositories
  container.register('userRepository', () => {
    return new PrismaUserRepository();
  }, { singleton: true });
  
  // Register services
  container.register('emailService', () => {
    const emailProvider = {
      send: async (data) => {
        console.log('Sending email:', data);
        return { success: true };
      }
    };
    return new EmailService(emailProvider);
  }, { singleton: true });
  
  // Register controllers
  container.register('userController', (container) => {
    const userRepository = container.resolve('userRepository');
    const emailService = container.resolve('emailService');
    return new UserController(userRepository, emailService);
  });
  
  return container;
}
```

### **🌐 Microservices Architecture**

```javascript
// ============= MICROSERVICES ARCHITECTURE =============
// lib/microservices/service-registry.js

export class ServiceRegistry {
  constructor() {
    this.services = new Map();
    this.healthChecks = new Map();
    this.loadBalancers = new Map();
  }
  
  register(serviceName, instances, options = {}) {
    this.services.set(serviceName, {
      instances: instances.map(instance => ({
        ...instance,
        healthy: true,
        lastCheck: Date.now()
      })),
      strategy: options.loadBalancingStrategy || 'round-robin',
      healthCheckInterval: options.healthCheckInterval || 30000,
      timeout: options.timeout || 5000
    });
    
    this.setupLoadBalancer(serviceName);
    this.startHealthChecks(serviceName);
  }
  
  setupLoadBalancer(serviceName) {
    const service = this.services.get(serviceName);
    let currentIndex = 0;
    
    const strategies = {
      'round-robin': () => {
        const healthyInstances = service.instances.filter(i => i.healthy);
        if (healthyInstances.length === 0) return null;
        
        const instance = healthyInstances[currentIndex % healthyInstances.length];
        currentIndex++;
        return instance;
      },
      
      'random': () => {
        const healthyInstances = service.instances.filter(i => i.healthy);
        if (healthyInstances.length === 0) return null;
        
        return healthyInstances[Math.floor(Math.random() * healthyInstances.length)];
      },
      
      'least-connections': () => {
        const healthyInstances = service.instances.filter(i => i.healthy);
        if (healthyInstances.length === 0) return null;
        
        return healthyInstances.reduce((min, instance) => 
          (instance.connections || 0) < (min.connections || 0) ? instance : min
        );
      }
    };
    
    this.loadBalancers.set(serviceName, strategies[service.strategy]);
  }
  
  getInstance(serviceName) {
    const loadBalancer = this.loadBalancers.get(serviceName);
    if (!loadBalancer) {
      throw new Error(`Service ${serviceName} not found`);
    }
    
    return loadBalancer();
  }
  
  startHealthChecks(serviceName) {
    const service = this.services.get(serviceName);
    
    const checkHealth = async () => {
      const promises = service.instances.map(async (instance) => {
        try {
          const response = await fetch(`${instance.url}/health`, {
            method: 'GET',
            timeout: service.timeout
          });
          
          instance.healthy = response.ok;
          instance.lastCheck = Date.now();
        } catch (error) {
          instance.healthy = false;
          instance.lastCheck = Date.now();
          console.warn(`Health check failed for ${instance.url}:`, error.message);
        }
      });
      
      await Promise.allSettled(promises);
    };
    
    // Initial health check
    checkHealth();
    
    // Schedule periodic health checks
    const interval = setInterval(checkHealth, service.healthCheckInterval);
    this.healthChecks.set(serviceName, interval);
  }
  
  getServiceHealth(serviceName) {
    const service = this.services.get(serviceName);
    if (!service) return null;
    
    return {
      serviceName,
      totalInstances: service.instances.length,
      healthyInstances: service.instances.filter(i => i.healthy).length,
      instances: service.instances.map(instance => ({
        url: instance.url,
        healthy: instance.healthy,
        lastCheck: instance.lastCheck
      }))
    };
  }
  
  unregister(serviceName) {
    // Stop health checks
    const healthCheck = this.healthChecks.get(serviceName);
    if (healthCheck) {
      clearInterval(healthCheck);
      this.healthChecks.delete(serviceName);
    }
    
    // Remove service
    this.services.delete(serviceName);
    this.loadBalancers.delete(serviceName);
  }
}

// lib/microservices/api-gateway.js
export class APIGateway {
  constructor(serviceRegistry) {
    this.serviceRegistry = serviceRegistry;
    this.routes = new Map();
    this.middleware = [];
    this.rateLimiters = new Map();
  }
  
  addRoute(path, serviceName, options = {}) {
    this.routes.set(path, {
      serviceName,
      rewrite: options.rewrite || ((path) => path),
      middleware: options.middleware || [],
      rateLimit: options.rateLimit,
      cache: options.cache
    });
  }
  
  use(middleware) {
    this.middleware.push(middleware);
  }
  
  async handleRequest(req) {
    try {
      // Apply global middleware
      for (const middleware of this.middleware) {
        const result = await middleware(req);
        if (result?.blocked) {
          return result.response;
        }
      }
      
      // Find matching route
      const route = this.findRoute(req.path);
      if (!route) {
        return {
          status: 404,
          body: { error: 'Route not found' }
        };
      }
      
      // Apply route-specific middleware
      for (const middleware of route.middleware) {
        const result = await middleware(req);
        if (result?.blocked) {
          return result.response;
        }
      }
      
      // Check rate limiting
      if (route.rateLimit) {
        const allowed = await this.checkRateLimit(req, route.rateLimit);
        if (!allowed) {
          return {
            status: 429,
            body: { error: 'Rate limit exceeded' }
          };
        }
      }
      
      // Check cache
      if (route.cache && req.method === 'GET') {
        const cached = await this.getFromCache(req.path);
        if (cached) {
          return cached;
        }
      }
      
      // Get service instance
      const instance = this.serviceRegistry.getInstance(route.serviceName);
      if (!instance) {
        return {
          status: 503,
          body: { error: 'Service unavailable' }
        };
      }
      
      // Proxy request to service
      const response = await this.proxyRequest(req, instance, route);
      
      // Cache response
      if (route.cache && req.method === 'GET' && response.status === 200) {
        await this.setCache(req.path, response, route.cache.ttl);
      }
      
      return response;
    } catch (error) {
      console.error('API Gateway error:', error);
      return {
        status: 500,
        body: { error: 'Internal server error' }
      };
    }
  }
  
  findRoute(path) {
    for (const [routePath, route] of this.routes) {
      if (this.matchPath(path, routePath)) {
        return route;
      }
    }
    return null;
  }
  
  matchPath(requestPath, routePath) {
    // Simple pattern matching
    const routeRegex = routePath
      .replace(/:\w+/g, '([^/]+)')
      .replace(/\*/g, '.*');
    
    const regex = new RegExp(`^${routeRegex}$`);
    return regex.test(requestPath);
  }
  
  async proxyRequest(req, instance, route) {
    const targetPath = route.rewrite(req.path);
    const targetUrl = `${instance.url}${targetPath}`;
    
    try {
      instance.connections = (instance.connections || 0) + 1;
      
      const response = await fetch(targetUrl, {
        method: req.method,
        headers: {
          ...req.headers,
          'X-Forwarded-For': req.ip,
          'X-Original-Path': req.path
        },
        body: req.body
      });
      
      return {
        status: response.status,
        headers: Object.fromEntries(response.headers),
        body: await response.json()
      };
    } finally {
      instance.connections = Math.max(0, (instance.connections || 1) - 1);
    }
  }
  
  async checkRateLimit(req, rateLimit) {
    const key = `${req.ip}:${req.path}`;
    const limiter = this.rateLimiters.get(key) || {
      requests: 0,
      windowStart: Date.now()
    };
    
    const now = Date.now();
    const windowMs = rateLimit.windowMs || 60000; // 1 minute default
    
    // Reset window if expired
    if (now - limiter.windowStart > windowMs) {
      limiter.requests = 0;
      limiter.windowStart = now;
    }
    
    limiter.requests++;
    this.rateLimiters.set(key, limiter);
    
    return limiter.requests <= rateLimit.max;
  }
  
  async getFromCache(key) {
    // Simple in-memory cache (use Redis in production)
    return this.cache?.get(key);
  }
  
  async setCache(key, value, ttl) {
    // Simple in-memory cache (use Redis in production)
    if (!this.cache) {
      this.cache = new Map();
    }
    
    this.cache.set(key, value);
    
    setTimeout(() => {
      this.cache.delete(key);
    }, ttl);
  }
}

// lib/microservices/service-mesh.js
export class ServiceMesh {
  constructor() {
    this.services = new Map();
    this.policies = new Map();
    this.metrics = new Map();
  }
  
  registerService(serviceName, config) {
    this.services.set(serviceName, {
      ...config,
      connections: new Set(),
      metrics: {
        requests: 0,
        errors: 0,
        latency: []
      }
    });
  }
  
  addPolicy(serviceName, policy) {
    if (!this.policies.has(serviceName)) {
      this.policies.set(serviceName, []);
    }
    this.policies.get(serviceName).push(policy);
  }
  
  async intercept(fromService, toService, request) {
    const startTime = Date.now();
    
    try {
      // Apply policies
      const policies = this.policies.get(toService) || [];
      for (const policy of policies) {
        const result = await policy.apply(fromService, toService, request);
        if (result.blocked) {
          this.recordMetrics(fromService, toService, 'blocked', Date.now() - startTime);
          return result;
        }
      }
      
      // Record connection
      this.recordConnection(fromService, toService);
      
      // Process request
      const response = await this.processRequest(toService, request);
      
      // Record metrics
      this.recordMetrics(fromService, toService, 'success', Date.now() - startTime);
      
      return response;
    } catch (error) {
      this.recordMetrics(fromService, toService, 'error', Date.now() - startTime);
      throw error;
    }
  }
  
  recordConnection(fromService, toService) {
    const service = this.services.get(toService);
    if (service) {
      service.connections.add(fromService);
    }
  }
  
  recordMetrics(fromService, toService, status, latency) {
    const service = this.services.get(toService);
    if (service) {
      service.metrics.requests++;
      service.metrics.latency.push(latency);
      
      if (status === 'error') {
        service.metrics.errors++;
      }
      
      // Keep only last 1000 latency measurements
      if (service.metrics.latency.length > 1000) {
        service.metrics.latency = service.metrics.latency.slice(-1000);
      }
    }
  }
  
  async processRequest(serviceName, request) {
    // This would typically forward to the actual service
    // For now, we'll simulate processing
    return {
      status: 200,
      body: { message: `Processed by ${serviceName}` }
    };
  }
  
  getServiceMetrics(serviceName) {
    const service = this.services.get(serviceName);
    if (!service) return null;
    
    const latencies = service.metrics.latency;
    const avgLatency = latencies.length > 0 
      ? latencies.reduce((a, b) => a + b, 0) / latencies.length 
      : 0;
    
    return {
      serviceName,
      requests: service.metrics.requests,
      errors: service.metrics.errors,
      errorRate: service.metrics.requests > 0 
        ? (service.metrics.errors / service.metrics.requests) * 100 
        : 0,
      avgLatency: Math.round(avgLatency),
      connections: Array.from(service.connections)
    };
  }
  
  getTopology() {
    const topology = {
      nodes: [],
      edges: []
    };
    
    for (const [serviceName, service] of this.services) {
      topology.nodes.push({
        id: serviceName,
        type: 'service',
        metrics: this.getServiceMetrics(serviceName)
      });
      
      for (const connectedService of service.connections) {
        topology.edges.push({
          from: connectedService,
          to: serviceName
        });
      }
    }
    
    return topology;
  }
}

// Security policies
export class SecurityPolicy {
  constructor(name, rules) {
    this.name = name;
    this.rules = rules;
  }
  
  async apply(fromService, toService, request) {
    for (const rule of this.rules) {
      const result = await rule(fromService, toService, request);
      if (result.blocked) {
        return result;
      }
    }
    
    return { blocked: false };
  }
}

// Example security rules
export const securityRules = {
  allowedServices: (allowedList) => (fromService, toService, request) => {
    if (!allowedList.includes(fromService)) {
      return {
        blocked: true,
        reason: `Service ${fromService} not allowed to access ${toService}`,
        status: 403
      };
    }
    return { blocked: false };
  },
  
  requireAuth: (fromService, toService, request) => {
    if (!request.headers.authorization) {
      return {
        blocked: true,
        reason: 'Authentication required',
        status: 401
      };
    }
    return { blocked: false };
  },
  
  rateLimitPerService: (maxRequests, windowMs) => {
    const requestCounts = new Map();
    
    return (fromService, toService, request) => {
      const key = `${fromService}:${toService}`;
      const now = Date.now();
      const count = requestCounts.get(key) || { requests: 0, windowStart: now };
      
      if (now - count.windowStart > windowMs) {
        count.requests = 0;
        count.windowStart = now;
      }
      
      count.requests++;
      requestCounts.set(key, count);
      
      if (count.requests > maxRequests) {
        return {
          blocked: true,
          reason: 'Rate limit exceeded',
          status: 429
        };
      }
      
      return { blocked: false };
    };
  }
};
```

## 🎯 แบบฝึกหัด

### **🏗️ แบบฝึกหัดที่ 1: Clean Architecture**
สร้าง complete clean architecture system:

```javascript
// TODO: สร้าง clean architecture system

// 1. Domain layer expansion
class Product {
  // Business logic
  // Validation rules
  // Domain events
}

// 2. Use cases
class ManageInventoryUseCase {
  // Complex business flows
  // External service integration
  // Event publishing
}

// 3. Infrastructure adapters
class ElasticsearchRepository {
  // Full-text search
  // Complex queries
  // Performance optimization
}
```

### **🌐 แบบฝึกหัดที่ 2: Microservices**
สร้าง comprehensive microservices platform:

```javascript
// TODO: สร้าง microservices platform

// 1. Service discovery
class ServiceDiscovery {
  // Auto-registration
  // Health monitoring
  // Load balancing
}

// 2. Circuit breaker
class CircuitBreaker {
  // Failure detection
  // Fallback mechanisms
  // Recovery strategies
}

// 3. Distributed tracing
class DistributedTracing {
  // Request tracking
  // Performance monitoring
  // Error correlation
}
```

### **🔒 แบบฝึกหัดที่ 3: Security Architecture**
สร้าง comprehensive security system:

```javascript
// TODO: สร้าง security architecture

// 1. Authentication service
class AuthenticationService {
  // Multi-factor auth
  // SSO integration
  // Session management
}

// 2. Authorization engine
class AuthorizationEngine {
  // Role-based access
  // Policy engine
  // Dynamic permissions
}

// 3. Security monitoring
class SecurityMonitor {
  // Threat detection
  // Audit logging
  // Incident response
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 21: Design Patterns](./21-design-patterns.md)

**บทถัดไป**: [บทที่ 23: State Management](./23-state-management.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Microservices Patterns](https://microservices.io/patterns/)
- [Building Microservices - Sam Newman](https://samnewman.io/books/building_microservices/)
- [Domain-Driven Design](https://domainlanguage.com/ddd/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Clean Architecture** - separation of concerns และ dependency inversion
- **Microservices** - service registry, API gateway, และ service mesh
- **Architectural Patterns** - scalable และ maintainable design
- **Security Architecture** - comprehensive security strategies
- **Best Practices** - enterprise-level architecture principles

Architecture เป็นพื้นฐานสำคัญสำหรับ scalable applications! 🚀
