# บทที่ 29: Enterprise Integration

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ enterprise system integration patterns
- ทำความเข้าใจ API design และ microservices communication
- เรียนรู้ legacy system integration และ data migration
- สร้าง enterprise-grade integration solutions

## 🏢 Enterprise Architecture Patterns

### **🔗 Integration Hub & Message Broker**

```javascript
// ============= ENTERPRISE INTEGRATION HUB =============
// lib/integration/integration-hub.js

export class IntegrationHub {
  constructor(options = {}) {
    this.options = {
      maxConcurrentConnections: 1000,
      messageRetention: 7 * 24 * 60 * 60 * 1000, // 7 days
      deadLetterThreshold: 3,
      circuitBreakerThreshold: 5,
      circuitBreakerTimeout: 60000,
      enableMetrics: true,
      enableAudit: true,
      ...options
    };
    
    this.connectors = new Map();
    this.messageQueues = new Map();
    this.routingTable = new Map();
    this.transformers = new Map();
    this.validators = new Map();
    this.metrics = new Map();
    this.auditLog = [];
    this.circuitBreakers = new Map();
    
    this.initialize();
  }
  
  initialize() {
    this.setupDefaultQueues();
    this.startMetricsCollection();
    this.startHealthMonitoring();
  }
  
  // Connector Management
  registerConnector(id, connector) {
    this.connectors.set(id, {
      id,
      connector,
      status: 'disconnected',
      lastHealthCheck: null,
      messagesSent: 0,
      messagesReceived: 0,
      errors: 0,
      ...connector.getMetadata()
    });
    
    // Initialize circuit breaker
    this.circuitBreakers.set(id, {
      state: 'closed',
      failures: 0,
      lastFailure: null,
      nextAttempt: null
    });
    
    return this.connectors.get(id);
  }
  
  async connectSystem(connectorId, config = {}) {
    const connectorInfo = this.connectors.get(connectorId);
    if (!connectorInfo) {
      throw new Error(`Connector ${connectorId} not found`);
    }
    
    try {
      await connectorInfo.connector.connect(config);
      connectorInfo.status = 'connected';
      connectorInfo.lastConnected = Date.now();
      
      this.auditEvent('connector_connected', {
        connectorId,
        timestamp: Date.now()
      });
      
      return true;
    } catch (error) {
      connectorInfo.status = 'error';
      connectorInfo.lastError = error.message;
      
      this.auditEvent('connector_failed', {
        connectorId,
        error: error.message,
        timestamp: Date.now()
      });
      
      throw error;
    }
  }
  
  async disconnectSystem(connectorId) {
    const connectorInfo = this.connectors.get(connectorId);
    if (!connectorInfo) {
      throw new Error(`Connector ${connectorId} not found`);
    }
    
    try {
      await connectorInfo.connector.disconnect();
      connectorInfo.status = 'disconnected';
      
      this.auditEvent('connector_disconnected', {
        connectorId,
        timestamp: Date.now()
      });
      
      return true;
    } catch (error) {
      throw error;
    }
  }
  
  // Message Routing & Transformation
  addRoute(pattern, config) {
    this.routingTable.set(pattern, {
      pattern,
      destination: config.destination,
      transform: config.transform,
      validate: config.validate,
      retry: config.retry || { attempts: 3, delay: 1000 },
      dlq: config.dlq || 'dead-letter-queue',
      priority: config.priority || 0,
      created: Date.now()
    });
  }
  
  addTransformer(id, transformer) {
    this.transformers.set(id, transformer);
  }
  
  addValidator(id, validator) {
    this.validators.set(id, validator);
  }
  
  async sendMessage(destination, message, options = {}) {
    const messageId = this.generateMessageId();
    
    const envelope = {
      id: messageId,
      destination,
      payload: message,
      headers: {
        timestamp: Date.now(),
        source: options.source || 'integration-hub',
        correlationId: options.correlationId || messageId,
        messageType: options.messageType || 'data',
        priority: options.priority || 0,
        ...options.headers
      },
      metadata: {
        attempts: 0,
        maxAttempts: options.maxAttempts || 3,
        created: Date.now(),
        lastAttempt: null
      }
    };
    
    try {
      // Find matching route
      const route = this.findRoute(destination);
      if (route) {
        await this.processMessage(envelope, route);
      } else {
        await this.directSend(envelope);
      }
      
      this.recordMetric('messages_sent', 1, { destination });
      
      return messageId;
    } catch (error) {
      this.recordMetric('messages_failed', 1, { destination, error: error.name });
      
      this.auditEvent('message_failed', {
        messageId,
        destination,
        error: error.message,
        timestamp: Date.now()
      });
      
      throw error;
    }
  }
  
  findRoute(destination) {
    let bestMatch = null;
    let bestScore = -1;
    
    for (const [pattern, route] of this.routingTable) {
      const score = this.matchRoute(destination, pattern);
      if (score > bestScore) {
        bestScore = score;
        bestMatch = route;
      }
    }
    
    return bestMatch;
  }
  
  matchRoute(destination, pattern) {
    // Simple pattern matching - could be enhanced with regex or glob patterns
    if (pattern === '*') return 1;
    if (pattern === destination) return 100;
    if (destination.startsWith(pattern.replace('*', ''))) return 50;
    return 0;
  }
  
  async processMessage(envelope, route) {
    // Validate message
    if (route.validate) {
      const validator = this.validators.get(route.validate);
      if (validator && !await validator.validate(envelope.payload)) {
        throw new Error(`Message validation failed for ${route.validate}`);
      }
    }
    
    // Transform message
    if (route.transform) {
      const transformer = this.transformers.get(route.transform);
      if (transformer) {
        envelope.payload = await transformer.transform(envelope.payload);
      }
    }
    
    // Send to actual destination
    await this.deliverMessage(envelope, route);
  }
  
  async deliverMessage(envelope, route) {
    const maxAttempts = route.retry.attempts;
    let lastError;
    
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        envelope.metadata.attempts = attempt;
        envelope.metadata.lastAttempt = Date.now();
        
        await this.attemptDelivery(envelope, route.destination);
        
        this.auditEvent('message_delivered', {
          messageId: envelope.id,
          destination: route.destination,
          attempt,
          timestamp: Date.now()
        });
        
        return;
      } catch (error) {
        lastError = error;
        
        if (attempt < maxAttempts) {
          const delay = route.retry.delay * Math.pow(2, attempt - 1); // Exponential backoff
          await this.delay(delay);
        }
      }
    }
    
    // Send to dead letter queue
    await this.sendToDeadLetterQueue(envelope, route.dlq, lastError);
  }
  
  async attemptDelivery(envelope, destination) {
    const connector = this.connectors.get(destination);
    if (!connector) {
      throw new Error(`Destination connector ${destination} not found`);
    }
    
    // Check circuit breaker
    const breaker = this.circuitBreakers.get(destination);
    if (breaker.state === 'open') {
      if (Date.now() < breaker.nextAttempt) {
        throw new Error(`Circuit breaker open for ${destination}`);
      } else {
        breaker.state = 'half-open';
      }
    }
    
    try {
      await connector.connector.send(envelope);
      
      // Reset circuit breaker on success
      if (breaker.state === 'half-open') {
        breaker.state = 'closed';
        breaker.failures = 0;
      }
      
      connector.messagesSent++;
    } catch (error) {
      connector.errors++;
      
      // Update circuit breaker on failure
      breaker.failures++;
      breaker.lastFailure = Date.now();
      
      if (breaker.failures >= this.options.circuitBreakerThreshold) {
        breaker.state = 'open';
        breaker.nextAttempt = Date.now() + this.options.circuitBreakerTimeout;
      }
      
      throw error;
    }
  }
  
  async directSend(envelope) {
    const connector = this.connectors.get(envelope.destination);
    if (!connector) {
      throw new Error(`Connector ${envelope.destination} not found`);
    }
    
    if (connector.status !== 'connected') {
      throw new Error(`Connector ${envelope.destination} is not connected`);
    }
    
    await connector.connector.send(envelope);
    connector.messagesSent++;
  }
  
  async sendToDeadLetterQueue(envelope, dlqName, error) {
    const dlqEnvelope = {
      ...envelope,
      headers: {
        ...envelope.headers,
        originalDestination: envelope.destination,
        failureReason: error.message,
        failureTimestamp: Date.now()
      }
    };
    
    await this.enqueueMessage(dlqName, dlqEnvelope);
    
    this.auditEvent('message_dead_lettered', {
      messageId: envelope.id,
      originalDestination: envelope.destination,
      dlq: dlqName,
      reason: error.message,
      timestamp: Date.now()
    });
  }
  
  // Message Queue Management
  setupDefaultQueues() {
    this.createQueue('dead-letter-queue', {
      persistent: true,
      maxSize: 10000,
      ttl: this.options.messageRetention
    });
    
    this.createQueue('audit-queue', {
      persistent: true,
      maxSize: 50000,
      ttl: this.options.messageRetention
    });
    
    this.createQueue('metrics-queue', {
      persistent: false,
      maxSize: 1000,
      ttl: 3600000 // 1 hour
    });
  }
  
  createQueue(name, options = {}) {
    this.messageQueues.set(name, {
      name,
      messages: [],
      options: {
        persistent: options.persistent || false,
        maxSize: options.maxSize || 1000,
        ttl: options.ttl || 3600000, // 1 hour
        ...options
      },
      stats: {
        created: Date.now(),
        enqueued: 0,
        dequeued: 0,
        size: 0
      }
    });
  }
  
  async enqueueMessage(queueName, message) {
    const queue = this.messageQueues.get(queueName);
    if (!queue) {
      throw new Error(`Queue ${queueName} not found`);
    }
    
    // Check queue size limit
    if (queue.messages.length >= queue.options.maxSize) {
      // Remove oldest message if at capacity
      queue.messages.shift();
    }
    
    const queuedMessage = {
      ...message,
      queuedAt: Date.now(),
      expiresAt: Date.now() + queue.options.ttl
    };
    
    queue.messages.push(queuedMessage);
    queue.stats.enqueued++;
    queue.stats.size = queue.messages.length;
  }
  
  async dequeueMessage(queueName) {
    const queue = this.messageQueues.get(queueName);
    if (!queue) {
      throw new Error(`Queue ${queueName} not found`);
    }
    
    // Clean expired messages
    this.cleanExpiredMessages(queue);
    
    if (queue.messages.length === 0) {
      return null;
    }
    
    const message = queue.messages.shift();
    queue.stats.dequeued++;
    queue.stats.size = queue.messages.length;
    
    return message;
  }
  
  cleanExpiredMessages(queue) {
    const now = Date.now();
    queue.messages = queue.messages.filter(msg => msg.expiresAt > now);
    queue.stats.size = queue.messages.length;
  }
  
  // Data Transformation
  registerDataMapper(sourceSchema, targetSchema, mappingRules) {
    const mapperId = `${sourceSchema}_to_${targetSchema}`;
    
    this.addTransformer(mapperId, {
      transform: async (data) => {
        return this.applyMappingRules(data, mappingRules);
      }
    });
    
    return mapperId;
  }
  
  applyMappingRules(sourceData, mappingRules) {
    const targetData = {};
    
    for (const rule of mappingRules) {
      try {
        const value = this.extractValue(sourceData, rule.source);
        const transformedValue = rule.transform ? rule.transform(value) : value;
        this.setValue(targetData, rule.target, transformedValue);
      } catch (error) {
        if (rule.required) {
          throw new Error(`Required field mapping failed: ${rule.source} -> ${rule.target}`);
        }
        // Use default value if provided
        if (rule.default !== undefined) {
          this.setValue(targetData, rule.target, rule.default);
        }
      }
    }
    
    return targetData;
  }
  
  extractValue(data, path) {
    return path.split('.').reduce((obj, key) => {
      if (obj && typeof obj === 'object' && key in obj) {
        return obj[key];
      }
      throw new Error(`Path ${path} not found in data`);
    }, data);
  }
  
  setValue(data, path, value) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    
    const target = keys.reduce((obj, key) => {
      if (!(key in obj)) {
        obj[key] = {};
      }
      return obj[key];
    }, data);
    
    target[lastKey] = value;
  }
  
  // Monitoring & Metrics
  startMetricsCollection() {
    setInterval(() => {
      this.collectSystemMetrics();
    }, 30000);
  }
  
  collectSystemMetrics() {
    // Connector metrics
    for (const [id, connector] of this.connectors) {
      this.recordMetric('connector_status', connector.status === 'connected' ? 1 : 0, {
        connector: id
      });
      
      this.recordMetric('connector_messages_sent', connector.messagesSent, {
        connector: id
      });
      
      this.recordMetric('connector_errors', connector.errors, {
        connector: id
      });
    }
    
    // Queue metrics
    for (const [name, queue] of this.messageQueues) {
      this.recordMetric('queue_size', queue.stats.size, {
        queue: name
      });
      
      this.recordMetric('queue_enqueued_total', queue.stats.enqueued, {
        queue: name
      });
      
      this.recordMetric('queue_dequeued_total', queue.stats.dequeued, {
        queue: name
      });
    }
  }
  
  startHealthMonitoring() {
    setInterval(() => {
      this.performHealthChecks();
    }, 60000);
  }
  
  async performHealthChecks() {
    for (const [id, connector] of this.connectors) {
      try {
        const healthy = await connector.connector.healthCheck();
        connector.lastHealthCheck = Date.now();
        connector.healthy = healthy;
        
        if (!healthy && connector.status === 'connected') {
          connector.status = 'unhealthy';
          
          this.auditEvent('connector_unhealthy', {
            connectorId: id,
            timestamp: Date.now()
          });
        }
      } catch (error) {
        connector.healthy = false;
        connector.lastError = error.message;
      }
    }
  }
  
  recordMetric(name, value, labels = {}) {
    const metricKey = `${name}_${JSON.stringify(labels)}`;
    
    if (!this.metrics.has(metricKey)) {
      this.metrics.set(metricKey, {
        name,
        labels,
        values: [],
        lastValue: null
      });
    }
    
    const metric = this.metrics.get(metricKey);
    metric.values.push({
      value,
      timestamp: Date.now()
    });
    metric.lastValue = value;
    
    // Keep only recent values
    const cutoff = Date.now() - 3600000; // 1 hour
    metric.values = metric.values.filter(v => v.timestamp > cutoff);
  }
  
  auditEvent(eventType, eventData) {
    if (!this.options.enableAudit) return;
    
    const auditEntry = {
      eventType,
      eventData,
      timestamp: Date.now(),
      id: this.generateEventId()
    };
    
    this.auditLog.push(auditEntry);
    
    // Keep audit log size manageable
    if (this.auditLog.length > 10000) {
      this.auditLog = this.auditLog.slice(-5000);
    }
    
    // Send to audit queue
    this.enqueueMessage('audit-queue', auditEntry);
  }
  
  generateMessageId() {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 8)}`;
  }
  
  generateEventId() {
    return `evt_${Date.now()}_${Math.random().toString(36).substr(2, 8)}`;
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // API Methods
  getSystemStatus() {
    const connectorStatus = Array.from(this.connectors.entries()).map(([id, connector]) => ({
      id,
      status: connector.status,
      healthy: connector.healthy,
      messagesSent: connector.messagesSent,
      errors: connector.errors,
      lastHealthCheck: connector.lastHealthCheck
    }));
    
    const queueStatus = Array.from(this.messageQueues.entries()).map(([name, queue]) => ({
      name,
      size: queue.stats.size,
      enqueued: queue.stats.enqueued,
      dequeued: queue.stats.dequeued
    }));
    
    return {
      connectors: connectorStatus,
      queues: queueStatus,
      routes: this.routingTable.size,
      uptime: Date.now() - (this.startTime || Date.now())
    };
  }
  
  getMetrics() {
    const metrics = {};
    
    for (const [key, metric] of this.metrics) {
      metrics[metric.name] = metrics[metric.name] || [];
      metrics[metric.name].push({
        labels: metric.labels,
        value: metric.lastValue,
        timestamp: metric.values.length > 0 ? 
          metric.values[metric.values.length - 1].timestamp : 
          Date.now()
      });
    }
    
    return metrics;
  }
  
  getAuditLog(limit = 100) {
    return this.auditLog
      .slice(-limit)
      .sort((a, b) => b.timestamp - a.timestamp);
  }
}

// ============= LEGACY SYSTEM INTEGRATION =============
// lib/integration/legacy-adapter.js

export class LegacySystemAdapter {
  constructor(options = {}) {
    this.options = {
      connectionTimeout: 30000,
      requestTimeout: 60000,
      retryAttempts: 3,
      retryDelay: 1000,
      enableCaching: true,
      cacheTimeout: 300000, // 5 minutes
      enableDataValidation: true,
      enableAuditLogging: true,
      ...options
    };
    
    this.connections = new Map();
    this.cache = new Map();
    this.schemas = new Map();
    this.transformers = new Map();
    this.auditLog = [];
  }
  
  // Connection Management
  async addLegacySystem(systemId, config) {
    const connection = {
      id: systemId,
      type: config.type, // soap, rpc, database, file, etc.
      endpoint: config.endpoint,
      credentials: config.credentials,
      schema: config.schema,
      options: config.options || {},
      status: 'disconnected',
      lastConnected: null,
      connector: this.createConnector(config.type, config)
    };
    
    this.connections.set(systemId, connection);
    
    // Register schema if provided
    if (config.schema) {
      this.schemas.set(systemId, config.schema);
    }
    
    return connection;
  }
  
  createConnector(type, config) {
    switch (type) {
      case 'soap':
        return new SOAPConnector(config);
      case 'database':
        return new DatabaseConnector(config);
      case 'file':
        return new FileSystemConnector(config);
      case 'rpc':
        return new RPCConnector(config);
      case 'mainframe':
        return new MainframeConnector(config);
      default:
        throw new Error(`Unsupported connector type: ${type}`);
    }
  }
  
  async connect(systemId) {
    const connection = this.connections.get(systemId);
    if (!connection) {
      throw new Error(`System ${systemId} not found`);
    }
    
    try {
      await connection.connector.connect();
      connection.status = 'connected';
      connection.lastConnected = Date.now();
      
      this.auditLog.push({
        event: 'system_connected',
        systemId,
        timestamp: Date.now()
      });
      
      return true;
    } catch (error) {
      connection.status = 'error';
      connection.lastError = error.message;
      throw error;
    }
  }
  
  async disconnect(systemId) {
    const connection = this.connections.get(systemId);
    if (!connection) {
      throw new Error(`System ${systemId} not found`);
    }
    
    try {
      await connection.connector.disconnect();
      connection.status = 'disconnected';
      
      this.auditLog.push({
        event: 'system_disconnected',
        systemId,
        timestamp: Date.now()
      });
      
      return true;
    } catch (error) {
      throw error;
    }
  }
  
  // Data Operations
  async queryData(systemId, query, options = {}) {
    const connection = this.connections.get(systemId);
    if (!connection) {
      throw new Error(`System ${systemId} not found`);
    }
    
    if (connection.status !== 'connected') {
      await this.connect(systemId);
    }
    
    // Check cache first
    const cacheKey = this.generateCacheKey(systemId, query);
    if (this.options.enableCaching && this.cache.has(cacheKey)) {
      const cached = this.cache.get(cacheKey);
      if (Date.now() - cached.timestamp < this.options.cacheTimeout) {
        return cached.data;
      }
    }
    
    try {
      const rawData = await this.executeQuery(connection, query, options);
      
      // Transform data if transformer is available
      const transformedData = await this.transformData(systemId, rawData, 'query');
      
      // Validate data
      if (this.options.enableDataValidation) {
        this.validateData(systemId, transformedData);
      }
      
      // Cache result
      if (this.options.enableCaching) {
        this.cache.set(cacheKey, {
          data: transformedData,
          timestamp: Date.now()
        });
      }
      
      this.auditLog.push({
        event: 'data_queried',
        systemId,
        query: query.toString(),
        resultCount: Array.isArray(transformedData) ? transformedData.length : 1,
        timestamp: Date.now()
      });
      
      return transformedData;
    } catch (error) {
      this.auditLog.push({
        event: 'query_failed',
        systemId,
        query: query.toString(),
        error: error.message,
        timestamp: Date.now()
      });
      
      throw error;
    }
  }
  
  async writeData(systemId, data, options = {}) {
    const connection = this.connections.get(systemId);
    if (!connection) {
      throw new Error(`System ${systemId} not found`);
    }
    
    if (connection.status !== 'connected') {
      await this.connect(systemId);
    }
    
    try {
      // Validate data
      if (this.options.enableDataValidation) {
        this.validateData(systemId, data);
      }
      
      // Transform data for legacy system
      const transformedData = await this.transformData(systemId, data, 'write');
      
      const result = await this.executeWrite(connection, transformedData, options);
      
      // Invalidate cache
      this.invalidateCache(systemId);
      
      this.auditLog.push({
        event: 'data_written',
        systemId,
        recordCount: Array.isArray(data) ? data.length : 1,
        result,
        timestamp: Date.now()
      });
      
      return result;
    } catch (error) {
      this.auditLog.push({
        event: 'write_failed',
        systemId,
        error: error.message,
        timestamp: Date.now()
      });
      
      throw error;
    }
  }
  
  async executeQuery(connection, query, options) {
    const maxAttempts = options.retryAttempts || this.options.retryAttempts;
    let lastError;
    
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await connection.connector.query(query, options);
      } catch (error) {
        lastError = error;
        
        if (attempt < maxAttempts) {
          const delay = this.options.retryDelay * Math.pow(2, attempt - 1);
          await this.delay(delay);
        }
      }
    }
    
    throw lastError;
  }
  
  async executeWrite(connection, data, options) {
    const maxAttempts = options.retryAttempts || this.options.retryAttempts;
    let lastError;
    
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await connection.connector.write(data, options);
      } catch (error) {
        lastError = error;
        
        if (attempt < maxAttempts) {
          const delay = this.options.retryDelay * Math.pow(2, attempt - 1);
          await this.delay(delay);
        }
      }
    }
    
    throw lastError;
  }
  
  // Data Transformation
  addTransformer(systemId, direction, transformer) {
    const key = `${systemId}_${direction}`;
    this.transformers.set(key, transformer);
  }
  
  async transformData(systemId, data, direction) {
    const key = `${systemId}_${direction}`;
    const transformer = this.transformers.get(key);
    
    if (transformer) {
      return await transformer.transform(data);
    }
    
    return data;
  }
  
  validateData(systemId, data) {
    const schema = this.schemas.get(systemId);
    if (!schema) return true;
    
    // Simple validation (could be enhanced with JSON Schema or similar)
    if (schema.required) {
      for (const field of schema.required) {
        if (!(field in data)) {
          throw new Error(`Required field ${field} is missing`);
        }
      }
    }
    
    if (schema.fields) {
      for (const [field, fieldSchema] of Object.entries(schema.fields)) {
        if (field in data) {
          this.validateField(data[field], fieldSchema, field);
        }
      }
    }
    
    return true;
  }
  
  validateField(value, fieldSchema, fieldName) {
    if (fieldSchema.type) {
      const actualType = typeof value;
      if (actualType !== fieldSchema.type) {
        throw new Error(`Field ${fieldName} expected ${fieldSchema.type}, got ${actualType}`);
      }
    }
    
    if (fieldSchema.maxLength && value.length > fieldSchema.maxLength) {
      throw new Error(`Field ${fieldName} exceeds maximum length of ${fieldSchema.maxLength}`);
    }
    
    if (fieldSchema.pattern && !new RegExp(fieldSchema.pattern).test(value)) {
      throw new Error(`Field ${fieldName} does not match pattern ${fieldSchema.pattern}`);
    }
  }
  
  // Cache Management
  generateCacheKey(systemId, query) {
    return `${systemId}_${JSON.stringify(query)}`;
  }
  
  invalidateCache(systemId) {
    for (const key of this.cache.keys()) {
      if (key.startsWith(systemId)) {
        this.cache.delete(key);
      }
    }
  }
  
  clearCache() {
    this.cache.clear();
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Status and Monitoring
  getSystemStatus() {
    const systems = Array.from(this.connections.entries()).map(([id, connection]) => ({
      id,
      type: connection.type,
      status: connection.status,
      lastConnected: connection.lastConnected,
      lastError: connection.lastError
    }));
    
    return {
      systems,
      cacheSize: this.cache.size,
      auditLogSize: this.auditLog.length
    };
  }
  
  getAuditLog(limit = 100) {
    return this.auditLog
      .slice(-limit)
      .sort((a, b) => b.timestamp - a.timestamp);
  }
}

// Connector implementations (simplified)
class SOAPConnector {
  constructor(config) {
    this.config = config;
    this.client = null;
  }
  
  async connect() {
    // Initialize SOAP client
    this.client = { connected: true }; // Placeholder
  }
  
  async disconnect() {
    this.client = null;
  }
  
  async query(operation, params) {
    // Execute SOAP operation
    return { result: 'soap_data' }; // Placeholder
  }
  
  async write(operation, data) {
    // Execute SOAP write operation
    return { success: true }; // Placeholder
  }
}

class DatabaseConnector {
  constructor(config) {
    this.config = config;
    this.connection = null;
  }
  
  async connect() {
    // Initialize database connection
    this.connection = { connected: true }; // Placeholder
  }
  
  async disconnect() {
    this.connection = null;
  }
  
  async query(sql, params) {
    // Execute SQL query
    return [{ id: 1, data: 'database_data' }]; // Placeholder
  }
  
  async write(sql, params) {
    // Execute SQL write
    return { affectedRows: 1 }; // Placeholder
  }
}

class FileSystemConnector {
  constructor(config) {
    this.config = config;
  }
  
  async connect() {
    // Check file system access
  }
  
  async disconnect() {
    // Cleanup file handles
  }
  
  async query(path, options) {
    // Read file data
    return { data: 'file_data' }; // Placeholder
  }
  
  async write(path, data) {
    // Write file data
    return { written: true }; // Placeholder
  }
}

class RPCConnector {
  constructor(config) {
    this.config = config;
  }
  
  async connect() {
    // Initialize RPC connection
  }
  
  async disconnect() {
    // Close RPC connection
  }
  
  async query(method, params) {
    // Execute RPC call
    return { result: 'rpc_data' }; // Placeholder
  }
  
  async write(method, data) {
    // Execute RPC write
    return { success: true }; // Placeholder
  }
}

class MainframeConnector {
  constructor(config) {
    this.config = config;
  }
  
  async connect() {
    // Connect to mainframe
  }
  
  async disconnect() {
    // Disconnect from mainframe
  }
  
  async query(program, params) {
    // Execute mainframe program
    return { data: 'mainframe_data' }; // Placeholder
  }
  
  async write(program, data) {
    // Execute mainframe write
    return { success: true }; // Placeholder
  }
}
```

## 🔌 API Design & Microservices

### **🌐 Enterprise API Gateway**

```javascript
// ============= API GATEWAY =============
// lib/integration/api-gateway.js

export class EnterpriseAPIGateway {
  constructor(options = {}) {
    this.options = {
      port: options.port || 3000,
      enableRateLimit: true,
      enableAuth: true,
      enableCors: true,
      enableCompression: true,
      enableCaching: true,
      enableLogging: true,
      enableMetrics: true,
      rateLimitWindow: 60000, // 1 minute
      rateLimitMax: 1000, // requests per window
      cacheTimeout: 300000, // 5 minutes
      ...options
    };
    
    this.routes = new Map();
    this.middleware = [];
    this.services = new Map();
    this.rateLimiters = new Map();
    this.cache = new Map();
    this.metrics = new Map();
    this.authProviders = new Map();
    
    this.initialize();
  }
  
  initialize() {
    this.setupDefaultMiddleware();
    this.startMetricsCollection();
  }
  
  setupDefaultMiddleware() {
    if (this.options.enableCors) {
      this.use(this.corsMiddleware());
    }
    
    if (this.options.enableAuth) {
      this.use(this.authMiddleware());
    }
    
    if (this.options.enableRateLimit) {
      this.use(this.rateLimitMiddleware());
    }
    
    if (this.options.enableCaching) {
      this.use(this.cacheMiddleware());
    }
    
    if (this.options.enableLogging) {
      this.use(this.loggingMiddleware());
    }
    
    if (this.options.enableMetrics) {
      this.use(this.metricsMiddleware());
    }
  }
  
  // Route Management
  addRoute(pattern, config) {
    this.routes.set(pattern, {
      pattern,
      method: config.method || 'GET',
      service: config.service,
      endpoint: config.endpoint,
      timeout: config.timeout || 30000,
      retries: config.retries || 3,
      circuitBreaker: config.circuitBreaker || false,
      authentication: config.authentication || 'required',
      authorization: config.authorization || [],
      rateLimit: config.rateLimit || {},
      transform: config.transform || null,
      validate: config.validate || null,
      cache: config.cache || false,
      created: Date.now()
    });
  }
  
  addService(serviceId, config) {
    this.services.set(serviceId, {
      id: serviceId,
      baseUrl: config.baseUrl,
      healthEndpoint: config.healthEndpoint || '/health',
      timeout: config.timeout || 30000,
      retries: config.retries || 3,
      loadBalancer: config.loadBalancer || 'round-robin',
      instances: config.instances || [config.baseUrl],
      currentInstance: 0,
      circuitBreaker: {
        state: 'closed',
        failures: 0,
        threshold: config.circuitBreakerThreshold || 5,
        timeout: config.circuitBreakerTimeout || 60000,
        nextAttempt: null
      },
      healthStatus: 'unknown',
      lastHealthCheck: null
    });
  }
  
  // Middleware System
  use(middleware) {
    this.middleware.push(middleware);
  }
  
  corsMiddleware() {
    return async (req, res, next) => {
      res.headers = res.headers || {};
      res.headers['Access-Control-Allow-Origin'] = '*';
      res.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS';
      res.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization, X-API-Key';
      
      if (req.method === 'OPTIONS') {
        res.status = 200;
        return res;
      }
      
      return next();
    };
  }
  
  authMiddleware() {
    return async (req, res, next) => {
      const route = this.findMatchingRoute(req.path, req.method);
      
      if (!route || route.authentication === 'none') {
        return next();
      }
      
      const authHeader = req.headers.authorization;
      if (!authHeader) {
        return this.unauthorizedResponse(res, 'Authorization header missing');
      }
      
      try {
        const token = authHeader.replace('Bearer ', '');
        const user = await this.validateToken(token);
        
        if (route.authorization.length > 0) {
          const hasPermission = await this.checkAuthorization(user, route.authorization);
          if (!hasPermission) {
            return this.forbiddenResponse(res, 'Insufficient permissions');
          }
        }
        
        req.user = user;
        return next();
      } catch (error) {
        return this.unauthorizedResponse(res, 'Invalid token');
      }
    };
  }
  
  rateLimitMiddleware() {
    return async (req, res, next) => {
      const clientId = this.getClientId(req);
      const route = this.findMatchingRoute(req.path, req.method);
      
      const limit = route?.rateLimit?.max || this.options.rateLimitMax;
      const window = route?.rateLimit?.window || this.options.rateLimitWindow;
      
      const limiter = this.getRateLimiter(clientId, limit, window);
      
      if (!limiter.isAllowed()) {
        return this.rateLimitExceededResponse(res, limiter.getResetTime());
      }
      
      limiter.increment();
      
      res.headers = res.headers || {};
      res.headers['X-RateLimit-Limit'] = limit.toString();
      res.headers['X-RateLimit-Remaining'] = limiter.getRemaining().toString();
      res.headers['X-RateLimit-Reset'] = limiter.getResetTime().toString();
      
      return next();
    };
  }
  
  cacheMiddleware() {
    return async (req, res, next) => {
      if (req.method !== 'GET') {
        return next();
      }
      
      const route = this.findMatchingRoute(req.path, req.method);
      if (!route || !route.cache) {
        return next();
      }
      
      const cacheKey = this.generateCacheKey(req);
      const cached = this.cache.get(cacheKey);
      
      if (cached && Date.now() - cached.timestamp < this.options.cacheTimeout) {
        res.status = 200;
        res.data = cached.data;
        res.headers = res.headers || {};
        res.headers['X-Cache'] = 'HIT';
        return res;
      }
      
      // Store original response
      const originalResponse = await next();
      
      if (originalResponse.status === 200) {
        this.cache.set(cacheKey, {
          data: originalResponse.data,
          timestamp: Date.now()
        });
      }
      
      originalResponse.headers = originalResponse.headers || {};
      originalResponse.headers['X-Cache'] = 'MISS';
      
      return originalResponse;
    };
  }
  
  loggingMiddleware() {
    return async (req, res, next) => {
      const startTime = Date.now();
      
      console.log(`[${new Date().toISOString()}] ${req.method} ${req.path} - Started`);
      
      const response = await next();
      
      const duration = Date.now() - startTime;
      console.log(`[${new Date().toISOString()}] ${req.method} ${req.path} - ${response.status} (${duration}ms)`);
      
      return response;
    };
  }
  
  metricsMiddleware() {
    return async (req, res, next) => {
      const startTime = Date.now();
      
      this.recordMetric('requests_total', 1, {
        method: req.method,
        path: req.path
      });
      
      const response = await next();
      
      const duration = Date.now() - startTime;
      
      this.recordMetric('request_duration_ms', duration, {
        method: req.method,
        path: req.path,
        status: response.status.toString()
      });
      
      this.recordMetric('responses_total', 1, {
        method: req.method,
        path: req.path,
        status: response.status.toString()
      });
      
      return response;
    };
  }
  
  // Request Processing
  async handleRequest(req) {
    const context = {
      request: req,
      response: {
        status: 200,
        headers: {},
        data: null
      },
      user: null,
      startTime: Date.now()
    };
    
    try {
      // Apply middleware chain
      const response = await this.applyMiddleware(context);
      
      if (response) {
        return response;
      }
      
      // Route to service
      const route = this.findMatchingRoute(req.path, req.method);
      if (!route) {
        return this.notFoundResponse();
      }
      
      return await this.proxyToService(context, route);
    } catch (error) {
      return this.errorResponse(error);
    }
  }
  
  async applyMiddleware(context) {
    let index = 0;
    
    const next = async () => {
      if (index >= this.middleware.length) {
        return null;
      }
      
      const middleware = this.middleware[index++];
      return await middleware(context.request, context.response, next);
    };
    
    return await next();
  }
  
  async proxyToService(context, route) {
    const service = this.services.get(route.service);
    if (!service) {
      throw new Error(`Service ${route.service} not found`);
    }
    
    // Check circuit breaker
    if (service.circuitBreaker.state === 'open') {
      if (Date.now() < service.circuitBreaker.nextAttempt) {
        throw new Error('Service circuit breaker is open');
      } else {
        service.circuitBreaker.state = 'half-open';
      }
    }
    
    const instance = this.selectServiceInstance(service);
    const url = `${instance}${route.endpoint}`;
    
    try {
      // Validate request if configured
      if (route.validate) {
        await this.validateRequest(context.request, route.validate);
      }
      
      // Transform request if configured
      let requestBody = context.request.body;
      if (route.transform && route.transform.request) {
        requestBody = await route.transform.request(requestBody);
      }
      
      const response = await this.makeServiceRequest(url, {
        method: context.request.method,
        headers: context.request.headers,
        body: requestBody,
        timeout: route.timeout
      });
      
      // Transform response if configured
      let responseData = response.data;
      if (route.transform && route.transform.response) {
        responseData = await route.transform.response(responseData);
      }
      
      // Reset circuit breaker on success
      if (service.circuitBreaker.state === 'half-open') {
        service.circuitBreaker.state = 'closed';
        service.circuitBreaker.failures = 0;
      }
      
      return {
        status: response.status,
        headers: response.headers,
        data: responseData
      };
    } catch (error) {
      // Update circuit breaker on failure
      service.circuitBreaker.failures++;
      
      if (service.circuitBreaker.failures >= service.circuitBreaker.threshold) {
        service.circuitBreaker.state = 'open';
        service.circuitBreaker.nextAttempt = Date.now() + service.circuitBreaker.timeout;
      }
      
      throw error;
    }
  }
  
  selectServiceInstance(service) {
    if (service.instances.length === 1) {
      return service.instances[0];
    }
    
    switch (service.loadBalancer) {
      case 'round-robin':
        const instance = service.instances[service.currentInstance];
        service.currentInstance = (service.currentInstance + 1) % service.instances.length;
        return instance;
      
      case 'random':
        return service.instances[Math.floor(Math.random() * service.instances.length)];
      
      default:
        return service.instances[0];
    }
  }
  
  async makeServiceRequest(url, options) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), options.timeout);
    
    try {
      const response = await fetch(url, {
        method: options.method,
        headers: options.headers,
        body: options.body ? JSON.stringify(options.body) : undefined,
        signal: controller.signal
      });
      
      const data = await response.json();
      
      return {
        status: response.status,
        headers: Object.fromEntries(response.headers.entries()),
        data
      };
    } finally {
      clearTimeout(timeoutId);
    }
  }
  
  // Helper Methods
  findMatchingRoute(path, method) {
    for (const [pattern, route] of this.routes) {
      if (route.method === method && this.matchPath(path, pattern)) {
        return route;
      }
    }
    return null;
  }
  
  matchPath(path, pattern) {
    // Simple pattern matching - could be enhanced with regex
    if (pattern === path) return true;
    if (pattern.includes('*')) {
      const regex = new RegExp('^' + pattern.replace(/\*/g, '.*') + '$');
      return regex.test(path);
    }
    return false;
  }
  
  async validateToken(token) {
    // JWT validation or external auth service call
    return { id: 'user123', roles: ['user'] }; // Placeholder
  }
  
  async checkAuthorization(user, requiredRoles) {
    return requiredRoles.some(role => user.roles.includes(role));
  }
  
  getClientId(req) {
    return req.headers['x-api-key'] || req.ip || 'anonymous';
  }
  
  getRateLimiter(clientId, limit, window) {
    const key = `${clientId}_${Math.floor(Date.now() / window)}`;
    
    if (!this.rateLimiters.has(key)) {
      this.rateLimiters.set(key, {
        count: 0,
        limit,
        window,
        resetTime: Math.floor(Date.now() / window) * window + window,
        isAllowed: function() { return this.count < this.limit; },
        increment: function() { this.count++; },
        getRemaining: function() { return Math.max(0, this.limit - this.count); },
        getResetTime: function() { return this.resetTime; }
      });
    }
    
    return this.rateLimiters.get(key);
  }
  
  generateCacheKey(req) {
    return `${req.method}_${req.path}_${JSON.stringify(req.query || {})}`;
  }
  
  async validateRequest(req, schema) {
    // Request validation logic
    return true; // Placeholder
  }
  
  recordMetric(name, value, labels = {}) {
    const key = `${name}_${JSON.stringify(labels)}`;
    
    if (!this.metrics.has(key)) {
      this.metrics.set(key, {
        name,
        labels,
        count: 0,
        sum: 0,
        avg: 0
      });
    }
    
    const metric = this.metrics.get(key);
    metric.count++;
    metric.sum += value;
    metric.avg = metric.sum / metric.count;
  }
  
  startMetricsCollection() {
    setInterval(() => {
      this.cleanupExpiredEntries();
    }, 60000); // Cleanup every minute
  }
  
  cleanupExpiredEntries() {
    const now = Date.now();
    
    // Cleanup rate limiters
    for (const [key, limiter] of this.rateLimiters) {
      if (now > limiter.resetTime) {
        this.rateLimiters.delete(key);
      }
    }
    
    // Cleanup cache
    for (const [key, entry] of this.cache) {
      if (now - entry.timestamp > this.options.cacheTimeout) {
        this.cache.delete(key);
      }
    }
  }
  
  // Response helpers
  unauthorizedResponse(res, message) {
    return {
      status: 401,
      headers: { 'Content-Type': 'application/json' },
      data: { error: 'Unauthorized', message }
    };
  }
  
  forbiddenResponse(res, message) {
    return {
      status: 403,
      headers: { 'Content-Type': 'application/json' },
      data: { error: 'Forbidden', message }
    };
  }
  
  rateLimitExceededResponse(res, resetTime) {
    return {
      status: 429,
      headers: {
        'Content-Type': 'application/json',
        'X-RateLimit-Reset': resetTime.toString()
      },
      data: { error: 'Rate limit exceeded' }
    };
  }
  
  notFoundResponse() {
    return {
      status: 404,
      headers: { 'Content-Type': 'application/json' },
      data: { error: 'Not found' }
    };
  }
  
  errorResponse(error) {
    return {
      status: 500,
      headers: { 'Content-Type': 'application/json' },
      data: { error: 'Internal server error', message: error.message }
    };
  }
}
```

## 🎯 แบบฝึกหัด

### **🏢 แบบฝึกหัดที่ 1: Integration Platform**
สร้าง comprehensive integration platform:

```javascript
// TODO: สร้าง integration platform

// 1. Event-driven integration
class EventDrivenIntegration {
  // Event sourcing across systems
  // Saga pattern implementation
  // Event replay capabilities
}

// 2. Data synchronization
class DataSynchronizer {
  // Real-time sync
  // Conflict resolution
  // Delta synchronization
}

// 3. Workflow orchestration
class WorkflowOrchestrator {
  // Business process automation
  // Human task integration
  // Process monitoring
}
```

### **🔌 แบบฝึกหัดที่ 2: API Management**
สร้าง enterprise API management system:

```javascript
// TODO: สร้าง API management system

// 1. API versioning
class APIVersionManager {
  // Multiple version support
  // Deprecation policies
  // Migration tools
}

// 2. Developer portal
class DeveloperPortal {
  // API documentation
  // SDK generation
  // Testing tools
}

// 3. Analytics platform
class APIAnalytics {
  // Usage metrics
  // Performance monitoring
  // Business insights
}
```

### **📊 แบบฝึกหัดที่ 3: Enterprise Data Platform**
สร้าง data integration platform:

```javascript
// TODO: สร้าง data platform

// 1. Data lake integration
class DataLakeIntegrator {
  // Multi-format support
  // Schema evolution
  // Data lineage tracking
}

// 2. Real-time streaming
class StreamProcessor {
  // Event streaming
  // Complex event processing
  // Stream analytics
}

// 3. Master data management
class MasterDataManager {
  // Golden record creation
  // Data quality rules
  // Reference data management
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 28: Production Deployment](./28-production-deployment.md)

**บทถัดไป**: [บทที่ 30: Advanced Optimization](./30-advanced-optimization.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)
- [API Gateway Pattern](https://microservices.io/patterns/apigateway.html)
- [Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
- [Legacy System Integration](https://martinfowler.com/bliki/LegacySystem.html)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Integration Hub** - centralized message routing และ transformation
- **Legacy System Integration** - connecting with existing enterprise systems
- **API Gateway** - comprehensive API management และ routing
- **Enterprise Patterns** - proven patterns สำหรับ enterprise integration
- **Data Integration** - synchronization และ transformation strategies

Enterprise Integration เป็นกุญแจสำคัญสำหรับ digital transformation! 🚀
