# บทที่ 27: Scalability

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ horizontal และ vertical scaling strategies
- ทำความเข้าใจ distributed systems และ microservices architecture
- เรียนรู้ database scaling และ data partitioning techniques
- สร้าง scalable applications ด้วย modern architectural patterns

## 📈 Scaling Strategies

### **🏗️ Horizontal Scaling System**

```javascript
// ============= HORIZONTAL SCALING =============
// lib/scaling/horizontal-scaler.js

export class HorizontalScaler {
  constructor(options = {}) {
    this.options = {
      minInstances: 2,
      maxInstances: 20,
      targetCpuUtilization: 70,
      targetMemoryUtilization: 80,
      scaleUpCooldown: 300000, // 5 minutes
      scaleDownCooldown: 600000, // 10 minutes
      metricsInterval: 30000,
      healthCheckInterval: 60000,
      loadBalancer: null,
      instanceProvider: null,
      ...options
    };
    
    this.instances = new Map();
    this.metrics = new Map();
    this.scalingHistory = [];
    this.lastScaleUp = 0;
    this.lastScaleDown = 0;
    this.isScaling = false;
    
    this.initialize();
  }
  
  initialize() {
    this.startMetricsCollection();
    this.startHealthChecks();
    this.startAutoScaling();
  }
  
  async addInstance(config = {}) {
    if (this.isScaling) {
      throw new Error('Scaling operation in progress');
    }
    
    this.isScaling = true;
    
    try {
      const instance = await this.createInstance(config);
      this.instances.set(instance.id, instance);
      
      // Wait for instance to be ready
      await this.waitForInstanceReady(instance);
      
      // Add to load balancer
      if (this.options.loadBalancer) {
        await this.options.loadBalancer.addServer(instance.id, {
          url: instance.endpoint,
          weight: instance.weight || 1
        });
      }
      
      this.recordScalingEvent('scale-up', {
        instanceId: instance.id,
        totalInstances: this.instances.size
      });
      
      return instance;
    } finally {
      this.isScaling = false;
    }
  }
  
  async removeInstance(instanceId) {
    if (this.isScaling) {
      throw new Error('Scaling operation in progress');
    }
    
    const instance = this.instances.get(instanceId);
    if (!instance) {
      throw new Error(`Instance ${instanceId} not found`);
    }
    
    this.isScaling = true;
    
    try {
      // Remove from load balancer first
      if (this.options.loadBalancer) {
        await this.options.loadBalancer.removeServer(instanceId);
      }
      
      // Graceful shutdown
      await this.gracefulShutdown(instance);
      
      // Destroy instance
      await this.destroyInstance(instance);
      
      this.instances.delete(instanceId);
      this.metrics.delete(instanceId);
      
      this.recordScalingEvent('scale-down', {
        instanceId,
        totalInstances: this.instances.size
      });
      
      return true;
    } finally {
      this.isScaling = false;
    }
  }
  
  async createInstance(config) {
    if (!this.options.instanceProvider) {
      throw new Error('Instance provider not configured');
    }
    
    const instanceConfig = {
      image: config.image || 'default',
      resources: {
        cpu: config.cpu || '1000m',
        memory: config.memory || '1Gi'
      },
      environment: config.environment || {},
      labels: {
        'app': 'scalable-service',
        'created-by': 'horizontal-scaler',
        ...config.labels
      },
      ...config
    };
    
    const instance = await this.options.instanceProvider.create(instanceConfig);
    
    return {
      id: instance.id,
      endpoint: instance.endpoint,
      status: 'starting',
      created: Date.now(),
      config: instanceConfig,
      metrics: {
        cpu: 0,
        memory: 0,
        requests: 0,
        errors: 0
      },
      ...instance
    };
  }
  
  async waitForInstanceReady(instance, timeout = 300000) {
    const startTime = Date.now();
    
    while (Date.now() - startTime < timeout) {
      try {
        const healthCheck = await this.checkInstanceHealth(instance);
        if (healthCheck.healthy) {
          instance.status = 'ready';
          return true;
        }
      } catch (error) {
        // Continue waiting
      }
      
      await this.delay(5000);
    }
    
    throw new Error(`Instance ${instance.id} failed to become ready within timeout`);
  }
  
  async gracefulShutdown(instance, timeout = 60000) {
    instance.status = 'terminating';
    
    try {
      // Send shutdown signal
      await this.sendShutdownSignal(instance);
      
      // Wait for graceful shutdown
      const startTime = Date.now();
      while (Date.now() - startTime < timeout) {
        const activeConnections = await this.getActiveConnections(instance);
        if (activeConnections === 0) {
          break;
        }
        await this.delay(1000);
      }
    } catch (error) {
      console.warn(`Graceful shutdown failed for ${instance.id}:`, error);
    }
  }
  
  async destroyInstance(instance) {
    if (this.options.instanceProvider) {
      await this.options.instanceProvider.destroy(instance.id);
    }
  }
  
  startMetricsCollection() {
    setInterval(async () => {
      await this.collectMetrics();
    }, this.options.metricsInterval);
  }
  
  async collectMetrics() {
    for (const [instanceId, instance] of this.instances) {
      try {
        const metrics = await this.getInstanceMetrics(instance);
        this.metrics.set(instanceId, {
          ...metrics,
          timestamp: Date.now()
        });
      } catch (error) {
        console.warn(`Failed to collect metrics for ${instanceId}:`, error);
      }
    }
  }
  
  async getInstanceMetrics(instance) {
    // This would typically call your monitoring system
    // Placeholder implementation
    return {
      cpu: Math.random() * 100,
      memory: Math.random() * 100,
      requests: Math.floor(Math.random() * 1000),
      errors: Math.floor(Math.random() * 10),
      responseTime: Math.random() * 1000
    };
  }
  
  startHealthChecks() {
    setInterval(async () => {
      await this.performHealthChecks();
    }, this.options.healthCheckInterval);
  }
  
  async performHealthChecks() {
    const healthChecks = Array.from(this.instances.values()).map(
      instance => this.checkInstanceHealth(instance)
    );
    
    const results = await Promise.allSettled(healthChecks);
    
    results.forEach((result, index) => {
      const instance = Array.from(this.instances.values())[index];
      if (result.status === 'fulfilled') {
        instance.healthy = result.value.healthy;
        instance.lastHealthCheck = Date.now();
      } else {
        instance.healthy = false;
        console.warn(`Health check failed for ${instance.id}:`, result.reason);
      }
    });
  }
  
  async checkInstanceHealth(instance) {
    try {
      const response = await fetch(`${instance.endpoint}/health`, {
        timeout: 5000
      });
      
      return {
        healthy: response.ok,
        status: response.status,
        responseTime: Date.now() - Date.now() // Placeholder
      };
    } catch (error) {
      return {
        healthy: false,
        error: error.message
      };
    }
  }
  
  startAutoScaling() {
    setInterval(async () => {
      await this.evaluateScaling();
    }, this.options.metricsInterval);
  }
  
  async evaluateScaling() {
    if (this.isScaling || this.instances.size === 0) {
      return;
    }
    
    const aggregatedMetrics = this.aggregateMetrics();
    const scalingDecision = this.makeScalingDecision(aggregatedMetrics);
    
    if (scalingDecision.action === 'scale-up' && this.canScaleUp()) {
      await this.scaleUp(scalingDecision.count);
    } else if (scalingDecision.action === 'scale-down' && this.canScaleDown()) {
      await this.scaleDown(scalingDecision.count);
    }
  }
  
  aggregateMetrics() {
    const allMetrics = Array.from(this.metrics.values());
    
    if (allMetrics.length === 0) {
      return { cpu: 0, memory: 0, requests: 0, errors: 0 };
    }
    
    const sum = allMetrics.reduce((acc, metrics) => ({
      cpu: acc.cpu + metrics.cpu,
      memory: acc.memory + metrics.memory,
      requests: acc.requests + metrics.requests,
      errors: acc.errors + metrics.errors
    }), { cpu: 0, memory: 0, requests: 0, errors: 0 });
    
    return {
      cpu: sum.cpu / allMetrics.length,
      memory: sum.memory / allMetrics.length,
      requests: sum.requests,
      errors: sum.errors,
      errorRate: sum.requests > 0 ? (sum.errors / sum.requests) * 100 : 0
    };
  }
  
  makeScalingDecision(metrics) {
    const { cpu, memory, errorRate } = metrics;
    
    // Scale up conditions
    if (cpu > this.options.targetCpuUtilization || 
        memory > this.options.targetMemoryUtilization ||
        errorRate > 5) {
      
      const cpuPressure = Math.max(0, cpu - this.options.targetCpuUtilization) / 20;
      const memoryPressure = Math.max(0, memory - this.options.targetMemoryUtilization) / 20;
      const errorPressure = Math.max(0, errorRate - 5) / 10;
      
      const pressure = Math.max(cpuPressure, memoryPressure, errorPressure);
      const scaleCount = Math.ceil(pressure * this.instances.size * 0.5);
      
      return {
        action: 'scale-up',
        count: Math.min(scaleCount, this.options.maxInstances - this.instances.size),
        reason: `High utilization: CPU=${cpu.toFixed(1)}%, Memory=${memory.toFixed(1)}%, ErrorRate=${errorRate.toFixed(1)}%`
      };
    }
    
    // Scale down conditions
    if (cpu < this.options.targetCpuUtilization * 0.5 && 
        memory < this.options.targetMemoryUtilization * 0.5 &&
        errorRate < 1) {
      
      const overCapacity = Math.min(
        (this.options.targetCpuUtilization * 0.5 - cpu) / 20,
        (this.options.targetMemoryUtilization * 0.5 - memory) / 20
      );
      
      const scaleCount = Math.ceil(overCapacity * this.instances.size * 0.3);
      
      return {
        action: 'scale-down',
        count: Math.min(scaleCount, this.instances.size - this.options.minInstances),
        reason: `Low utilization: CPU=${cpu.toFixed(1)}%, Memory=${memory.toFixed(1)}%`
      };
    }
    
    return { action: 'none' };
  }
  
  canScaleUp() {
    return Date.now() - this.lastScaleUp > this.options.scaleUpCooldown &&
           this.instances.size < this.options.maxInstances;
  }
  
  canScaleDown() {
    return Date.now() - this.lastScaleDown > this.options.scaleDownCooldown &&
           this.instances.size > this.options.minInstances;
  }
  
  async scaleUp(count) {
    console.log(`Scaling up by ${count} instances`);
    this.lastScaleUp = Date.now();
    
    const promises = Array.from({ length: count }, () => this.addInstance());
    await Promise.allSettled(promises);
  }
  
  async scaleDown(count) {
    console.log(`Scaling down by ${count} instances`);
    this.lastScaleDown = Date.now();
    
    // Select instances to remove (prefer least utilized)
    const instances = Array.from(this.instances.values())
      .filter(instance => instance.healthy)
      .sort((a, b) => {
        const metricsA = this.metrics.get(a.id) || { cpu: 0, memory: 0 };
        const metricsB = this.metrics.get(b.id) || { cpu: 0, memory: 0 };
        return (metricsA.cpu + metricsA.memory) - (metricsB.cpu + metricsB.memory);
      });
    
    const instancesToRemove = instances.slice(0, count);
    const promises = instancesToRemove.map(instance => this.removeInstance(instance.id));
    
    await Promise.allSettled(promises);
  }
  
  recordScalingEvent(action, details) {
    const event = {
      action,
      timestamp: Date.now(),
      details
    };
    
    this.scalingHistory.push(event);
    
    // Keep only recent history
    if (this.scalingHistory.length > 1000) {
      this.scalingHistory = this.scalingHistory.slice(-500);
    }
    
    console.log('Scaling event:', event);
  }
  
  async sendShutdownSignal(instance) {
    try {
      await fetch(`${instance.endpoint}/shutdown`, {
        method: 'POST',
        timeout: 5000
      });
    } catch (error) {
      // Ignore errors, force shutdown will follow
    }
  }
  
  async getActiveConnections(instance) {
    try {
      const response = await fetch(`${instance.endpoint}/metrics/connections`);
      const data = await response.json();
      return data.active || 0;
    } catch (error) {
      return 0;
    }
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  getStatus() {
    const healthyInstances = Array.from(this.instances.values())
      .filter(instance => instance.healthy);
    
    const metrics = this.aggregateMetrics();
    
    return {
      instances: {
        total: this.instances.size,
        healthy: healthyInstances.length,
        unhealthy: this.instances.size - healthyInstances.length
      },
      metrics,
      scaling: {
        canScaleUp: this.canScaleUp(),
        canScaleDown: this.canScaleDown(),
        isScaling: this.isScaling
      },
      recentEvents: this.scalingHistory.slice(-10)
    };
  }
  
  destroy() {
    // Remove all instances
    const removePromises = Array.from(this.instances.keys())
      .map(instanceId => this.removeInstance(instanceId));
    
    return Promise.allSettled(removePromises);
  }
}

// ============= DISTRIBUTED SYSTEMS =============
// lib/scaling/distributed-coordinator.js

export class DistributedCoordinator {
  constructor(options = {}) {
    this.options = {
      nodeId: this.generateNodeId(),
      heartbeatInterval: 5000,
      electionTimeout: 15000,
      leaderTimeout: 30000,
      consensusTimeout: 10000,
      retryAttempts: 3,
      retryDelay: 1000,
      ...options
    };
    
    this.nodes = new Map();
    this.state = 'follower'; // follower, candidate, leader
    this.currentTerm = 0;
    this.votedFor = null;
    this.log = [];
    this.commitIndex = 0;
    this.lastApplied = 0;
    this.leaderHeartbeat = null;
    this.electionTimer = null;
    
    this.initialize();
  }
  
  initialize() {
    this.startElectionTimer();
    this.startHeartbeatListener();
  }
  
  addNode(nodeId, endpoint) {
    this.nodes.set(nodeId, {
      id: nodeId,
      endpoint,
      lastSeen: Date.now(),
      healthy: true,
      nextIndex: this.log.length,
      matchIndex: 0
    });
  }
  
  removeNode(nodeId) {
    this.nodes.delete(nodeId);
  }
  
  startElectionTimer() {
    if (this.electionTimer) {
      clearTimeout(this.electionTimer);
    }
    
    const timeout = this.options.electionTimeout + 
                   Math.random() * this.options.electionTimeout;
    
    this.electionTimer = setTimeout(() => {
      if (this.state !== 'leader') {
        this.startElection();
      }
    }, timeout);
  }
  
  async startElection() {
    this.state = 'candidate';
    this.currentTerm++;
    this.votedFor = this.options.nodeId;
    
    console.log(`Starting election for term ${this.currentTerm}`);
    
    const votes = await this.requestVotes();
    const majority = Math.floor(this.nodes.size / 2) + 1;
    
    if (votes >= majority) {
      this.becomeLeader();
    } else {
      this.becomeFollower();
    }
  }
  
  async requestVotes() {
    const voteRequests = Array.from(this.nodes.values()).map(node =>
      this.sendVoteRequest(node)
    );
    
    const results = await Promise.allSettled(voteRequests);
    const votes = results.filter(result => 
      result.status === 'fulfilled' && result.value === true
    ).length;
    
    return votes + 1; // Include own vote
  }
  
  async sendVoteRequest(node) {
    try {
      const request = {
        term: this.currentTerm,
        candidateId: this.options.nodeId,
        lastLogIndex: this.log.length - 1,
        lastLogTerm: this.log.length > 0 ? this.log[this.log.length - 1].term : 0
      };
      
      const response = await this.sendRequest(node, '/vote', request);
      return response.voteGranted;
    } catch (error) {
      return false;
    }
  }
  
  becomeLeader() {
    this.state = 'leader';
    console.log(`Became leader for term ${this.currentTerm}`);
    
    // Initialize leader state
    for (const node of this.nodes.values()) {
      node.nextIndex = this.log.length;
      node.matchIndex = 0;
    }
    
    this.startHeartbeat();
  }
  
  becomeFollower() {
    this.state = 'follower';
    this.votedFor = null;
    this.startElectionTimer();
  }
  
  startHeartbeat() {
    if (this.leaderHeartbeat) {
      clearInterval(this.leaderHeartbeat);
    }
    
    this.leaderHeartbeat = setInterval(() => {
      this.sendHeartbeats();
    }, this.options.heartbeatInterval);
  }
  
  async sendHeartbeats() {
    if (this.state !== 'leader') return;
    
    const heartbeats = Array.from(this.nodes.values()).map(node =>
      this.sendAppendEntries(node, [])
    );
    
    await Promise.allSettled(heartbeats);
  }
  
  async sendAppendEntries(node, entries) {
    try {
      const prevLogIndex = node.nextIndex - 1;
      const prevLogTerm = prevLogIndex >= 0 ? this.log[prevLogIndex].term : 0;
      
      const request = {
        term: this.currentTerm,
        leaderId: this.options.nodeId,
        prevLogIndex,
        prevLogTerm,
        entries,
        leaderCommit: this.commitIndex
      };
      
      const response = await this.sendRequest(node, '/append', request);
      
      if (response.success) {
        node.nextIndex = prevLogIndex + entries.length + 1;
        node.matchIndex = prevLogIndex + entries.length;
      } else {
        node.nextIndex = Math.max(1, node.nextIndex - 1);
      }
      
      return response.success;
    } catch (error) {
      return false;
    }
  }
  
  async appendEntry(command) {
    if (this.state !== 'leader') {
      throw new Error('Only leader can append entries');
    }
    
    const entry = {
      term: this.currentTerm,
      index: this.log.length,
      command,
      timestamp: Date.now()
    };
    
    this.log.push(entry);
    
    // Replicate to followers
    const replications = Array.from(this.nodes.values()).map(node =>
      this.sendAppendEntries(node, [entry])
    );
    
    const results = await Promise.allSettled(replications);
    const successCount = results.filter(r => 
      r.status === 'fulfilled' && r.value === true
    ).length;
    
    const majority = Math.floor(this.nodes.size / 2) + 1;
    
    if (successCount + 1 >= majority) {
      this.commitIndex = entry.index;
      await this.applyCommittedEntries();
      return true;
    }
    
    return false;
  }
  
  async applyCommittedEntries() {
    while (this.lastApplied < this.commitIndex) {
      this.lastApplied++;
      const entry = this.log[this.lastApplied];
      await this.applyEntry(entry);
    }
  }
  
  async applyEntry(entry) {
    // Apply the entry to the state machine
    console.log(`Applying entry: ${JSON.stringify(entry)}`);
    
    // This would typically update application state
    // based on the command in the entry
  }
  
  handleVoteRequest(request) {
    const { term, candidateId, lastLogIndex, lastLogTerm } = request;
    
    if (term > this.currentTerm) {
      this.currentTerm = term;
      this.votedFor = null;
      this.becomeFollower();
    }
    
    const voteGranted = term >= this.currentTerm &&
                      (this.votedFor === null || this.votedFor === candidateId) &&
                      this.isLogUpToDate(lastLogIndex, lastLogTerm);
    
    if (voteGranted) {
      this.votedFor = candidateId;
      this.startElectionTimer();
    }
    
    return {
      term: this.currentTerm,
      voteGranted
    };
  }
  
  handleAppendEntries(request) {
    const { term, leaderId, prevLogIndex, prevLogTerm, entries, leaderCommit } = request;
    
    if (term > this.currentTerm) {
      this.currentTerm = term;
      this.becomeFollower();
    }
    
    if (term < this.currentTerm) {
      return { term: this.currentTerm, success: false };
    }
    
    // Reset election timer on valid heartbeat
    this.startElectionTimer();
    
    // Check log consistency
    if (prevLogIndex >= 0 && 
        (this.log.length <= prevLogIndex || 
         this.log[prevLogIndex].term !== prevLogTerm)) {
      return { term: this.currentTerm, success: false };
    }
    
    // Append entries
    if (entries.length > 0) {
      this.log.splice(prevLogIndex + 1);
      this.log.push(...entries);
    }
    
    // Update commit index
    if (leaderCommit > this.commitIndex) {
      this.commitIndex = Math.min(leaderCommit, this.log.length - 1);
      this.applyCommittedEntries();
    }
    
    return { term: this.currentTerm, success: true };
  }
  
  isLogUpToDate(lastLogIndex, lastLogTerm) {
    if (this.log.length === 0) return true;
    
    const myLastLogTerm = this.log[this.log.length - 1].term;
    const myLastLogIndex = this.log.length - 1;
    
    return lastLogTerm > myLastLogTerm ||
           (lastLogTerm === myLastLogTerm && lastLogIndex >= myLastLogIndex);
  }
  
  async sendRequest(node, path, data) {
    const response = await fetch(`${node.endpoint}${path}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
      timeout: 5000
    });
    
    return response.json();
  }
  
  startHeartbeatListener() {
    // This would typically be implemented as HTTP endpoints
    // for receiving vote requests and append entries
  }
  
  generateNodeId() {
    return `node_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  getStatus() {
    return {
      nodeId: this.options.nodeId,
      state: this.state,
      term: this.currentTerm,
      logLength: this.log.length,
      commitIndex: this.commitIndex,
      nodes: Array.from(this.nodes.values()).map(node => ({
        id: node.id,
        endpoint: node.endpoint,
        healthy: node.healthy,
        lastSeen: node.lastSeen
      }))
    };
  }
  
  destroy() {
    if (this.electionTimer) {
      clearTimeout(this.electionTimer);
    }
    if (this.leaderHeartbeat) {
      clearInterval(this.leaderHeartbeat);
    }
  }
}
```

## 🗄️ Database Scaling

### **📊 Database Sharding System**

```javascript
// ============= DATABASE SHARDING =============
// lib/scaling/database-shard.js

export class DatabaseShardManager {
  constructor(options = {}) {
    this.options = {
      shardingStrategy: 'hash', // hash, range, directory
      hashFunction: 'murmur3',
      replicationFactor: 3,
      consistencyLevel: 'quorum', // one, quorum, all
      autoRebalance: true,
      rebalanceThreshold: 0.8,
      migrationBatchSize: 1000,
      ...options
    };
    
    this.shards = new Map();
    this.shardRing = [];
    this.replicationMap = new Map();
    this.migrationTasks = new Map();
    this.stats = new Map();
    
    this.initialize();
  }
  
  initialize() {
    this.startStatsCollection();
    if (this.options.autoRebalance) {
      this.startRebalanceMonitor();
    }
  }
  
  addShard(shardId, config) {
    const shard = {
      id: shardId,
      endpoint: config.endpoint,
      weight: config.weight || 1,
      capacity: config.capacity || 1000000,
      currentSize: 0,
      healthy: true,
      region: config.region || 'default',
      ...config
    };
    
    this.shards.set(shardId, shard);
    this.stats.set(shardId, {
      reads: 0,
      writes: 0,
      errors: 0,
      latency: 0,
      size: 0
    });
    
    this.rebuildShardRing();
    this.updateReplicationMap();
    
    return shard;
  }
  
  removeShard(shardId) {
    if (!this.shards.has(shardId)) {
      throw new Error(`Shard ${shardId} not found`);
    }
    
    // Start migration before removing
    this.migrateShard(shardId);
    
    this.shards.delete(shardId);
    this.stats.delete(shardId);
    
    this.rebuildShardRing();
    this.updateReplicationMap();
  }
  
  rebuildShardRing() {
    this.shardRing = [];
    
    for (const [shardId, shard] of this.shards) {
      // Add multiple points for better distribution
      const points = shard.weight * 150;
      
      for (let i = 0; i < points; i++) {
        const hash = this.hash(`${shardId}:${i}`);
        this.shardRing.push({
          hash,
          shardId,
          point: i
        });
      }
    }
    
    // Sort by hash value
    this.shardRing.sort((a, b) => a.hash - b.hash);
  }
  
  updateReplicationMap() {
    this.replicationMap.clear();
    
    for (const [shardId] of this.shards) {
      const replicas = this.findReplicas(shardId);
      this.replicationMap.set(shardId, replicas);
    }
  }
  
  findReplicas(primaryShardId) {
    const replicas = [];
    const primaryIndex = this.shardRing.findIndex(item => item.shardId === primaryShardId);
    
    if (primaryIndex === -1) return replicas;
    
    const usedShards = new Set([primaryShardId]);
    let currentIndex = primaryIndex;
    
    while (replicas.length < this.options.replicationFactor - 1 && 
           usedShards.size < this.shards.size) {
      currentIndex = (currentIndex + 1) % this.shardRing.length;
      const shardId = this.shardRing[currentIndex].shardId;
      
      if (!usedShards.has(shardId)) {
        replicas.push(shardId);
        usedShards.add(shardId);
      }
    }
    
    return replicas;
  }
  
  async write(key, data, options = {}) {
    const shardId = this.getShardForKey(key);
    const replicas = this.replicationMap.get(shardId) || [];
    const allShards = [shardId, ...replicas];
    
    const writePromises = allShards.map(async shardId => {
      try {
        await this.writeToShard(shardId, key, data, options);
        return { shardId, success: true };
      } catch (error) {
        return { shardId, success: false, error };
      }
    });
    
    const results = await Promise.allSettled(writePromises);
    const successCount = results.filter(r => 
      r.status === 'fulfilled' && r.value.success
    ).length;
    
    const requiredSuccess = this.getRequiredSuccess();
    
    if (successCount >= requiredSuccess) {
      this.updateStats(shardId, 'write', true);
      return true;
    } else {
      this.updateStats(shardId, 'write', false);
      throw new Error(`Write failed: only ${successCount}/${allShards.length} shards succeeded`);
    }
  }
  
  async read(key, options = {}) {
    const shardId = this.getShardForKey(key);
    const replicas = this.replicationMap.get(shardId) || [];
    const allShards = [shardId, ...replicas];
    
    // Try reading from multiple shards based on consistency level
    const readCount = this.getRequiredSuccess();
    const selectedShards = allShards.slice(0, readCount);
    
    const readPromises = selectedShards.map(async shardId => {
      try {
        const data = await this.readFromShard(shardId, key, options);
        return { shardId, success: true, data };
      } catch (error) {
        return { shardId, success: false, error };
      }
    });
    
    const results = await Promise.allSettled(readPromises);
    const successResults = results.filter(r => 
      r.status === 'fulfilled' && r.value.success
    );
    
    if (successResults.length === 0) {
      this.updateStats(shardId, 'read', false);
      throw new Error('Read failed: no shards responded successfully');
    }
    
    this.updateStats(shardId, 'read', true);
    
    // Return the most recent data
    const data = successResults.map(r => r.value.data);
    return this.resolveConflicts(data);
  }
  
  async delete(key, options = {}) {
    const shardId = this.getShardForKey(key);
    const replicas = this.replicationMap.get(shardId) || [];
    const allShards = [shardId, ...replicas];
    
    const deletePromises = allShards.map(async shardId => {
      try {
        await this.deleteFromShard(shardId, key, options);
        return { shardId, success: true };
      } catch (error) {
        return { shardId, success: false, error };
      }
    });
    
    const results = await Promise.allSettled(deletePromises);
    const successCount = results.filter(r => 
      r.status === 'fulfilled' && r.value.success
    ).length;
    
    const requiredSuccess = this.getRequiredSuccess();
    
    if (successCount >= requiredSuccess) {
      return true;
    } else {
      throw new Error(`Delete failed: only ${successCount}/${allShards.length} shards succeeded`);
    }
  }
  
  getShardForKey(key) {
    switch (this.options.shardingStrategy) {
      case 'hash':
        return this.getShardByHash(key);
      case 'range':
        return this.getShardByRange(key);
      case 'directory':
        return this.getShardByDirectory(key);
      default:
        throw new Error(`Unknown sharding strategy: ${this.options.shardingStrategy}`);
    }
  }
  
  getShardByHash(key) {
    const hash = this.hash(key);
    
    // Find the first shard with hash >= key hash
    for (const item of this.shardRing) {
      if (item.hash >= hash) {
        return item.shardId;
      }
    }
    
    // Wrap around to the first shard
    return this.shardRing[0]?.shardId;
  }
  
  getShardByRange(key) {
    // Implement range-based sharding
    // This would typically involve configured ranges
    throw new Error('Range sharding not implemented');
  }
  
  getShardByDirectory(key) {
    // Implement directory-based sharding
    // This would involve a lookup table
    throw new Error('Directory sharding not implemented');
  }
  
  hash(input) {
    // Simple hash function (replace with murmur3 or similar)
    let hash = 0;
    for (let i = 0; i < input.length; i++) {
      const char = input.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }
  
  getRequiredSuccess() {
    switch (this.options.consistencyLevel) {
      case 'one':
        return 1;
      case 'quorum':
        return Math.floor(this.options.replicationFactor / 2) + 1;
      case 'all':
        return this.options.replicationFactor;
      default:
        return 1;
    }
  }
  
  async writeToShard(shardId, key, data, options) {
    const shard = this.shards.get(shardId);
    if (!shard) {
      throw new Error(`Shard ${shardId} not found`);
    }
    
    // Simulate database write
    const response = await fetch(`${shard.endpoint}/write`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ key, data, options })
    });
    
    if (!response.ok) {
      throw new Error(`Write failed: ${response.statusText}`);
    }
    
    return response.json();
  }
  
  async readFromShard(shardId, key, options) {
    const shard = this.shards.get(shardId);
    if (!shard) {
      throw new Error(`Shard ${shardId} not found`);
    }
    
    // Simulate database read
    const response = await fetch(`${shard.endpoint}/read/${encodeURIComponent(key)}`, {
      method: 'GET'
    });
    
    if (!response.ok) {
      throw new Error(`Read failed: ${response.statusText}`);
    }
    
    return response.json();
  }
  
  async deleteFromShard(shardId, key, options) {
    const shard = this.shards.get(shardId);
    if (!shard) {
      throw new Error(`Shard ${shardId} not found`);
    }
    
    // Simulate database delete
    const response = await fetch(`${shard.endpoint}/delete/${encodeURIComponent(key)}`, {
      method: 'DELETE'
    });
    
    if (!response.ok) {
      throw new Error(`Delete failed: ${response.statusText}`);
    }
    
    return response.json();
  }
  
  resolveConflicts(dataArray) {
    if (dataArray.length === 1) {
      return dataArray[0];
    }
    
    // Simple conflict resolution: return most recent
    return dataArray.reduce((latest, current) => {
      if (!latest.timestamp || !current.timestamp) {
        return latest;
      }
      return current.timestamp > latest.timestamp ? current : latest;
    });
  }
  
  async migrateShard(sourceShardId) {
    const targetShards = Array.from(this.shards.keys())
      .filter(id => id !== sourceShardId);
    
    if (targetShards.length === 0) {
      throw new Error('No target shards available for migration');
    }
    
    const migrationId = this.generateMigrationId();
    const migration = {
      id: migrationId,
      sourceShardId,
      targetShards,
      status: 'running',
      progress: 0,
      startTime: Date.now()
    };
    
    this.migrationTasks.set(migrationId, migration);
    
    try {
      await this.performMigration(migration);
      migration.status = 'completed';
    } catch (error) {
      migration.status = 'failed';
      migration.error = error.message;
      throw error;
    }
    
    return migrationId;
  }
  
  async performMigration(migration) {
    // Get all keys from source shard
    const keys = await this.getShardKeys(migration.sourceShardId);
    const batches = this.createBatches(keys, this.options.migrationBatchSize);
    
    for (let i = 0; i < batches.length; i++) {
      const batch = batches[i];
      
      // Migrate each key in the batch
      for (const key of batch) {
        const data = await this.readFromShard(migration.sourceShardId, key);
        const newShardId = this.getShardForKey(key);
        
        if (newShardId !== migration.sourceShardId) {
          await this.writeToShard(newShardId, key, data);
          await this.deleteFromShard(migration.sourceShardId, key);
        }
      }
      
      migration.progress = ((i + 1) / batches.length) * 100;
    }
  }
  
  async getShardKeys(shardId) {
    const shard = this.shards.get(shardId);
    const response = await fetch(`${shard.endpoint}/keys`);
    const data = await response.json();
    return data.keys || [];
  }
  
  createBatches(array, batchSize) {
    const batches = [];
    for (let i = 0; i < array.length; i += batchSize) {
      batches.push(array.slice(i, i + batchSize));
    }
    return batches;
  }
  
  startStatsCollection() {
    setInterval(() => {
      this.collectShardStats();
    }, 30000);
  }
  
  async collectShardStats() {
    for (const [shardId, shard] of this.shards) {
      try {
        const stats = await this.getShardStats(shardId);
        this.stats.set(shardId, {
          ...this.stats.get(shardId),
          ...stats,
          timestamp: Date.now()
        });
      } catch (error) {
        console.warn(`Failed to collect stats for shard ${shardId}:`, error);
      }
    }
  }
  
  async getShardStats(shardId) {
    const shard = this.shards.get(shardId);
    const response = await fetch(`${shard.endpoint}/stats`);
    return response.json();
  }
  
  startRebalanceMonitor() {
    setInterval(() => {
      this.checkRebalanceNeeded();
    }, 300000); // Check every 5 minutes
  }
  
  async checkRebalanceNeeded() {
    const shardSizes = new Map();
    let totalSize = 0;
    
    for (const [shardId, stats] of this.stats) {
      const size = stats.size || 0;
      shardSizes.set(shardId, size);
      totalSize += size;
    }
    
    if (totalSize === 0) return;
    
    const averageSize = totalSize / this.shards.size;
    const threshold = averageSize * this.options.rebalanceThreshold;
    
    for (const [shardId, size] of shardSizes) {
      if (size > averageSize + threshold) {
        console.log(`Shard ${shardId} is overloaded, triggering rebalance`);
        await this.rebalance();
        break;
      }
    }
  }
  
  async rebalance() {
    console.log('Starting cluster rebalance');
    
    // Implement rebalancing logic
    // This would involve redistributing data across shards
    // to achieve better balance
  }
  
  updateStats(shardId, operation, success) {
    const stats = this.stats.get(shardId);
    if (stats) {
      stats[operation === 'read' ? 'reads' : 'writes']++;
      if (!success) {
        stats.errors++;
      }
    }
  }
  
  generateMigrationId() {
    return `migration_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  getClusterStatus() {
    const shardStatus = Array.from(this.shards.entries()).map(([id, shard]) => ({
      id,
      healthy: shard.healthy,
      stats: this.stats.get(id),
      replicas: this.replicationMap.get(id)
    }));
    
    const activeMigrations = Array.from(this.migrationTasks.values())
      .filter(m => m.status === 'running');
    
    return {
      shards: shardStatus,
      totalShards: this.shards.size,
      strategy: this.options.shardingStrategy,
      replicationFactor: this.options.replicationFactor,
      activeMigrations: activeMigrations.length,
      ringSize: this.shardRing.length
    };
  }
}
```

## 🎯 แบบฝึกหัด

### **📈 แบบฝึกหัดที่ 1: Auto-scaling Implementation**
สร้าง comprehensive auto-scaling system:

```javascript
// TODO: สร้าง auto-scaling system

// 1. Predictive scaling
class PredictiveScaler {
  // Machine learning for load prediction
  // Seasonal pattern recognition
  // Proactive scaling decisions
}

// 2. Multi-dimensional scaling
class MultiDimensionalScaler {
  // CPU, memory, network, disk scaling
  // Custom metrics integration
  // Composite scaling decisions
}

// 3. Cost-optimized scaling
class CostOptimizedScaler {
  // Spot instance management
  // Reserved capacity optimization
  // Multi-cloud cost comparison
}
```

### **🌐 แบบฝึกหัดที่ 2: Distributed System**
สร้าง resilient distributed system:

```javascript
// TODO: สร้าง distributed system

// 1. Consensus algorithm implementation
class ConsensusEngine {
  // Raft consensus implementation
  // Byzantine fault tolerance
  // Leader election optimization
}

// 2. Distributed cache
class DistributedCache {
  // Consistent hashing
  // Cache invalidation strategies
  // Cross-datacenter replication
}

// 3. Event sourcing cluster
class EventSourcingCluster {
  // Distributed event store
  // Projection synchronization
  // Snapshot coordination
}
```

### **🗄️ แบบฝึกหัดที่ 3: Database Architecture**
สร้าง scalable database system:

```javascript
// TODO: สร้าง database architecture

// 1. Multi-master replication
class MultiMasterReplication {
  // Conflict resolution
  // Asynchronous replication
  // Split-brain prevention
}

// 2. CQRS with event store
class CQRSArchitecture {
  // Command/query separation
  // Event replay capabilities
  // Read model optimization
}

// 3. Time-series database
class TimeSeriesDB {
  // Data compression
  // Retention policies
  // Real-time aggregation
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 26: Performance Architecture](./26-performance-architecture.md)

**บทถัดไป**: [บทที่ 28: Production Deployment](./28-production-deployment.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [The Twelve-Factor App](https://12factor.net/)
- [Microservices Patterns](https://microservices.io/patterns/)
- [Distributed Systems Principles](https://web.mit.edu/6.824/www/)
- [Database Sharding Strategies](https://www.mongodb.com/features/database-sharding-explained)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Horizontal Scaling** - auto-scaling systems และ instance management
- **Distributed Systems** - consensus algorithms และ coordination patterns
- **Database Scaling** - sharding strategies และ replication patterns
- **Load Distribution** - intelligent traffic routing และ fault tolerance
- **System Resilience** - handling failures และ maintaining availability

Scalability เป็นกุญแจสำคัญสำหรับ enterprise applications! 🚀
