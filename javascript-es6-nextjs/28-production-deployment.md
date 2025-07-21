# บทที่ 28: Production Deployment

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ CI/CD pipelines และ deployment automation
- ทำความเข้าใจ containerization และ orchestration
- เรียนรู้ monitoring และ observability in production
- สร้าง production-ready deployment strategies

## 🚀 CI/CD Pipeline

### **🔄 Continuous Integration/Deployment**

```javascript
// ============= CI/CD PIPELINE =============
// lib/deployment/ci-cd-pipeline.js

export class CICDPipeline {
  constructor(options = {}) {
    this.options = {
      repository: null,
      branch: 'main',
      buildTimeout: 1800000, // 30 minutes
      testTimeout: 600000, // 10 minutes
      deployTimeout: 900000, // 15 minutes
      parallelJobs: 4,
      retryAttempts: 3,
      enableNotifications: true,
      enableArtifacts: true,
      environments: ['staging', 'production'],
      ...options
    };
    
    this.stages = new Map();
    this.artifacts = new Map();
    this.notifications = [];
    this.currentBuild = null;
    this.buildHistory = [];
    
    this.initializeStages();
  }
  
  initializeStages() {
    // Default stages
    this.addStage('checkout', {
      name: 'Checkout Code',
      script: this.checkoutCode.bind(this),
      parallel: false,
      required: true
    });
    
    this.addStage('install', {
      name: 'Install Dependencies',
      script: this.installDependencies.bind(this),
      parallel: false,
      required: true,
      cache: true
    });
    
    this.addStage('lint', {
      name: 'Code Linting',
      script: this.runLinting.bind(this),
      parallel: true,
      required: true
    });
    
    this.addStage('test', {
      name: 'Run Tests',
      script: this.runTests.bind(this),
      parallel: true,
      required: true,
      artifacts: ['coverage-report']
    });
    
    this.addStage('build', {
      name: 'Build Application',
      script: this.buildApplication.bind(this),
      parallel: false,
      required: true,
      artifacts: ['build-output', 'docker-image']
    });
    
    this.addStage('security-scan', {
      name: 'Security Scanning',
      script: this.securityScan.bind(this),
      parallel: true,
      required: false
    });
    
    this.addStage('deploy-staging', {
      name: 'Deploy to Staging',
      script: this.deployToStaging.bind(this),
      parallel: false,
      required: false,
      environment: 'staging'
    });
    
    this.addStage('integration-test', {
      name: 'Integration Tests',
      script: this.runIntegrationTests.bind(this),
      parallel: false,
      required: false,
      dependsOn: ['deploy-staging']
    });
    
    this.addStage('deploy-production', {
      name: 'Deploy to Production',
      script: this.deployToProduction.bind(this),
      parallel: false,
      required: false,
      environment: 'production',
      manual: true,
      dependsOn: ['integration-test']
    });
  }
  
  addStage(id, config) {
    this.stages.set(id, {
      id,
      status: 'pending',
      startTime: null,
      endTime: null,
      logs: [],
      artifacts: [],
      ...config
    });
  }
  
  async trigger(options = {}) {
    const buildId = this.generateBuildId();
    
    this.currentBuild = {
      id: buildId,
      trigger: options.trigger || 'manual',
      branch: options.branch || this.options.branch,
      commit: options.commit,
      status: 'running',
      startTime: Date.now(),
      endTime: null,
      stages: new Map(),
      environment: options.environment || 'staging'
    };
    
    this.buildHistory.unshift(this.currentBuild);
    
    try {
      await this.sendNotification('build-started', {
        buildId,
        branch: this.currentBuild.branch,
        commit: this.currentBuild.commit
      });
      
      await this.runPipeline();
      
      this.currentBuild.status = 'success';
      this.currentBuild.endTime = Date.now();
      
      await this.sendNotification('build-success', {
        buildId,
        duration: this.currentBuild.endTime - this.currentBuild.startTime
      });
      
      return this.currentBuild;
    } catch (error) {
      this.currentBuild.status = 'failed';
      this.currentBuild.endTime = Date.now();
      this.currentBuild.error = error.message;
      
      await this.sendNotification('build-failed', {
        buildId,
        error: error.message
      });
      
      throw error;
    }
  }
  
  async runPipeline() {
    const stageGroups = this.groupStagesByDependencies();
    
    for (const group of stageGroups) {
      const parallelStages = group.filter(stage => stage.parallel);
      const sequentialStages = group.filter(stage => !stage.parallel);
      
      // Run parallel stages concurrently
      if (parallelStages.length > 0) {
        await this.runStagesInParallel(parallelStages);
      }
      
      // Run sequential stages one by one
      for (const stage of sequentialStages) {
        await this.runStage(stage);
      }
    }
  }
  
  groupStagesByDependencies() {
    const groups = [];
    const processed = new Set();
    const stages = Array.from(this.stages.values());
    
    while (processed.size < stages.length) {
      const currentGroup = [];
      
      for (const stage of stages) {
        if (processed.has(stage.id)) continue;
        
        // Check if all dependencies are satisfied
        const dependencies = stage.dependsOn || [];
        const dependenciesSatisfied = dependencies.every(dep => processed.has(dep));
        
        if (dependenciesSatisfied) {
          currentGroup.push(stage);
          processed.add(stage.id);
        }
      }
      
      if (currentGroup.length === 0) {
        throw new Error('Circular dependency detected in pipeline stages');
      }
      
      groups.push(currentGroup);
    }
    
    return groups;
  }
  
  async runStagesInParallel(stages) {
    const promises = stages.map(stage => this.runStage(stage));
    await Promise.all(promises);
  }
  
  async runStage(stage) {
    if (stage.manual && !this.isManuallyTriggered(stage)) {
      stage.status = 'skipped';
      return;
    }
    
    const stageExecution = {
      id: stage.id,
      name: stage.name,
      status: 'running',
      startTime: Date.now(),
      endTime: null,
      logs: [],
      artifacts: []
    };
    
    this.currentBuild.stages.set(stage.id, stageExecution);
    
    try {
      await this.sendNotification('stage-started', {
        buildId: this.currentBuild.id,
        stageName: stage.name
      });
      
      // Check cache if enabled
      if (stage.cache && this.hasCachedResult(stage)) {
        const cachedResult = this.getCachedResult(stage);
        stageExecution.status = 'cached';
        stageExecution.endTime = Date.now();
        stageExecution.artifacts = cachedResult.artifacts || [];
        return;
      }
      
      // Run the stage script
      const result = await this.executeStageScript(stage, stageExecution);
      
      stageExecution.status = 'success';
      stageExecution.endTime = Date.now();
      
      // Handle artifacts
      if (stage.artifacts && result.artifacts) {
        stageExecution.artifacts = result.artifacts;
        await this.storeArtifacts(stage.id, result.artifacts);
      }
      
      // Cache result if caching is enabled
      if (stage.cache) {
        this.cacheResult(stage, {
          status: 'success',
          artifacts: stageExecution.artifacts
        });
      }
      
      await this.sendNotification('stage-success', {
        buildId: this.currentBuild.id,
        stageName: stage.name,
        duration: stageExecution.endTime - stageExecution.startTime
      });
      
    } catch (error) {
      stageExecution.status = 'failed';
      stageExecution.endTime = Date.now();
      stageExecution.error = error.message;
      
      await this.sendNotification('stage-failed', {
        buildId: this.currentBuild.id,
        stageName: stage.name,
        error: error.message
      });
      
      if (stage.required) {
        throw error;
      }
    }
  }
  
  async executeStageScript(stage, stageExecution) {
    const timeout = stage.timeout || this.options.buildTimeout;
    
    return new Promise((resolve, reject) => {
      const timer = setTimeout(() => {
        reject(new Error(`Stage ${stage.name} timed out after ${timeout}ms`));
      }, timeout);
      
      // Create execution context
      const context = {
        buildId: this.currentBuild.id,
        branch: this.currentBuild.branch,
        commit: this.currentBuild.commit,
        environment: this.currentBuild.environment,
        log: (message) => {
          stageExecution.logs.push({
            timestamp: Date.now(),
            message
          });
        },
        artifacts: new Map()
      };
      
      // Execute the stage script
      Promise.resolve(stage.script(context))
        .then(result => {
          clearTimeout(timer);
          resolve(result || {});
        })
        .catch(error => {
          clearTimeout(timer);
          reject(error);
        });
    });
  }
  
  // Built-in stage implementations
  async checkoutCode(context) {
    context.log('Checking out code from repository');
    
    // Simulate git checkout
    await this.delay(2000);
    
    context.log(`Checked out branch: ${context.branch}`);
    context.log(`Commit: ${context.commit || 'latest'}`);
    
    return { status: 'success' };
  }
  
  async installDependencies(context) {
    context.log('Installing dependencies');
    
    // Simulate npm install
    await this.delay(10000);
    
    context.log('Dependencies installed successfully');
    
    return { status: 'success' };
  }
  
  async runLinting(context) {
    context.log('Running code linting');
    
    // Simulate linting
    await this.delay(5000);
    
    context.log('Linting completed successfully');
    
    return { status: 'success' };
  }
  
  async runTests(context) {
    context.log('Running unit tests');
    
    // Simulate test execution
    await this.delay(15000);
    
    const coverageReport = {
      statements: 85.2,
      branches: 78.9,
      functions: 92.1,
      lines: 83.7
    };
    
    context.log(`Test coverage: ${coverageReport.statements}%`);
    
    return {
      status: 'success',
      artifacts: {
        'coverage-report': coverageReport
      }
    };
  }
  
  async buildApplication(context) {
    context.log('Building application');
    
    // Simulate build process
    await this.delay(20000);
    
    const buildOutput = {
      size: '2.3MB',
      files: ['index.js', 'styles.css', 'assets/'],
      hash: this.generateHash()
    };
    
    context.log(`Build completed. Output size: ${buildOutput.size}`);
    
    // Build Docker image
    const dockerImage = await this.buildDockerImage(context, buildOutput);
    
    return {
      status: 'success',
      artifacts: {
        'build-output': buildOutput,
        'docker-image': dockerImage
      }
    };
  }
  
  async buildDockerImage(context, buildOutput) {
    context.log('Building Docker image');
    
    const imageTag = `${this.options.repository}:${context.buildId}`;
    
    // Simulate docker build
    await this.delay(10000);
    
    context.log(`Docker image built: ${imageTag}`);
    
    return {
      tag: imageTag,
      size: '156MB',
      layers: 8
    };
  }
  
  async securityScan(context) {
    context.log('Running security scan');
    
    // Simulate security scanning
    await this.delay(30000);
    
    const vulnerabilities = Math.floor(Math.random() * 3);
    
    if (vulnerabilities > 0) {
      context.log(`Found ${vulnerabilities} vulnerabilities`);
      throw new Error(`Security scan failed: ${vulnerabilities} vulnerabilities found`);
    }
    
    context.log('No security vulnerabilities found');
    
    return { status: 'success' };
  }
  
  async deployToStaging(context) {
    context.log('Deploying to staging environment');
    
    const artifact = this.artifacts.get('docker-image');
    if (!artifact) {
      throw new Error('Docker image artifact not found');
    }
    
    // Simulate deployment
    await this.delay(15000);
    
    context.log('Deployment to staging completed');
    
    return {
      status: 'success',
      url: 'https://staging.example.com'
    };
  }
  
  async runIntegrationTests(context) {
    context.log('Running integration tests');
    
    // Simulate integration tests
    await this.delay(20000);
    
    context.log('Integration tests passed');
    
    return { status: 'success' };
  }
  
  async deployToProduction(context) {
    context.log('Deploying to production environment');
    
    // Simulate production deployment with blue-green
    await this.blueGreenDeployment(context);
    
    context.log('Production deployment completed');
    
    return {
      status: 'success',
      url: 'https://example.com'
    };
  }
  
  async blueGreenDeployment(context) {
    context.log('Starting blue-green deployment');
    
    // Deploy to green environment
    context.log('Deploying to green environment');
    await this.delay(10000);
    
    // Health check
    context.log('Running health checks on green environment');
    await this.delay(5000);
    
    // Switch traffic
    context.log('Switching traffic to green environment');
    await this.delay(2000);
    
    // Cleanup blue environment
    context.log('Cleaning up blue environment');
    await this.delay(3000);
  }
  
  async storeArtifacts(stageId, artifacts) {
    for (const [name, data] of Object.entries(artifacts)) {
      const artifactKey = `${this.currentBuild.id}:${stageId}:${name}`;
      this.artifacts.set(artifactKey, data);
    }
  }
  
  hasCachedResult(stage) {
    // Simple cache check (in real implementation, check file hashes, dependencies, etc.)
    return false;
  }
  
  getCachedResult(stage) {
    // Return cached result
    return null;
  }
  
  cacheResult(stage, result) {
    // Cache the result for future builds
  }
  
  isManuallyTriggered(stage) {
    // Check if manual stage has been triggered
    return false;
  }
  
  async sendNotification(type, data) {
    if (!this.options.enableNotifications) return;
    
    const notification = {
      type,
      timestamp: Date.now(),
      data
    };
    
    this.notifications.push(notification);
    
    // Send to external notification systems
    console.log(`Notification [${type}]:`, data);
  }
  
  generateBuildId() {
    return `build-${Date.now()}-${Math.random().toString(36).substr(2, 8)}`;
  }
  
  generateHash() {
    return Math.random().toString(36).substr(2, 16);
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  getBuildStatus(buildId = null) {
    const build = buildId ? 
      this.buildHistory.find(b => b.id === buildId) : 
      this.currentBuild;
    
    if (!build) return null;
    
    return {
      id: build.id,
      status: build.status,
      branch: build.branch,
      commit: build.commit,
      startTime: build.startTime,
      endTime: build.endTime,
      duration: build.endTime ? build.endTime - build.startTime : Date.now() - build.startTime,
      stages: Array.from(build.stages.values()),
      environment: build.environment
    };
  }
  
  getBuildHistory(limit = 50) {
    return this.buildHistory.slice(0, limit).map(build => ({
      id: build.id,
      status: build.status,
      branch: build.branch,
      startTime: build.startTime,
      endTime: build.endTime,
      duration: build.endTime ? build.endTime - build.startTime : null
    }));
  }
}

// ============= CONTAINERIZATION =============
// lib/deployment/docker-manager.js

export class DockerManager {
  constructor(options = {}) {
    this.options = {
      registry: 'docker.io',
      namespace: 'myapp',
      dockerHost: 'unix:///var/run/docker.sock',
      buildContext: '.',
      buildArgs: {},
      labels: {},
      healthCheckInterval: 30000,
      ...options
    };
    
    this.containers = new Map();
    this.images = new Map();
    this.networks = new Map();
    this.volumes = new Map();
  }
  
  async buildImage(config) {
    const {
      name,
      tag = 'latest',
      dockerfile = 'Dockerfile',
      context = this.options.buildContext,
      args = {},
      labels = {}
    } = config;
    
    const imageTag = `${this.options.registry}/${this.options.namespace}/${name}:${tag}`;
    
    const buildOptions = {
      dockerfile,
      context,
      buildArgs: { ...this.options.buildArgs, ...args },
      labels: { ...this.options.labels, ...labels },
      nocache: config.nocache || false,
      pull: config.pull || false
    };
    
    console.log(`Building Docker image: ${imageTag}`);
    
    // Simulate docker build
    const buildResult = await this.executeBuild(imageTag, buildOptions);
    
    this.images.set(imageTag, {
      name,
      tag: imageTag,
      size: buildResult.size,
      created: Date.now(),
      layers: buildResult.layers
    });
    
    return imageTag;
  }
  
  async executeBuild(imageTag, options) {
    // In real implementation, this would execute docker build command
    // or use Docker API
    
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          size: '156MB',
          layers: 8,
          id: this.generateImageId()
        });
      }, 10000);
    });
  }
  
  async pushImage(imageTag) {
    if (!this.images.has(imageTag)) {
      throw new Error(`Image ${imageTag} not found`);
    }
    
    console.log(`Pushing image: ${imageTag}`);
    
    // Simulate docker push
    await this.delay(15000);
    
    console.log(`Successfully pushed: ${imageTag}`);
    
    return {
      digest: this.generateDigest(),
      size: this.images.get(imageTag).size
    };
  }
  
  async pullImage(imageTag) {
    console.log(`Pulling image: ${imageTag}`);
    
    // Simulate docker pull
    await this.delay(10000);
    
    this.images.set(imageTag, {
      tag: imageTag,
      size: '156MB',
      created: Date.now(),
      layers: 8
    });
    
    console.log(`Successfully pulled: ${imageTag}`);
    
    return imageTag;
  }
  
  async runContainer(config) {
    const {
      name,
      image,
      ports = {},
      environment = {},
      volumes = {},
      networks = [],
      healthCheck = null,
      restartPolicy = 'unless-stopped',
      resources = {}
    } = config;
    
    const containerId = this.generateContainerId();
    
    const containerConfig = {
      id: containerId,
      name,
      image,
      ports,
      environment,
      volumes,
      networks,
      healthCheck,
      restartPolicy,
      resources,
      status: 'running',
      created: Date.now(),
      started: Date.now()
    };
    
    console.log(`Starting container: ${name} (${containerId})`);
    
    // Simulate container start
    await this.delay(3000);
    
    this.containers.set(containerId, containerConfig);
    
    // Start health checks if configured
    if (healthCheck) {
      this.startHealthCheck(containerId);
    }
    
    return containerId;
  }
  
  async stopContainer(containerId, timeout = 10) {
    const container = this.containers.get(containerId);
    if (!container) {
      throw new Error(`Container ${containerId} not found`);
    }
    
    console.log(`Stopping container: ${container.name} (${containerId})`);
    
    container.status = 'stopping';
    
    // Simulate graceful stop
    await this.delay(timeout * 1000);
    
    container.status = 'stopped';
    container.stopped = Date.now();
    
    return true;
  }
  
  async removeContainer(containerId, force = false) {
    const container = this.containers.get(containerId);
    if (!container) {
      throw new Error(`Container ${containerId} not found`);
    }
    
    if (container.status === 'running' && !force) {
      throw new Error(`Container ${containerId} is running. Stop it first or use force=true`);
    }
    
    console.log(`Removing container: ${container.name} (${containerId})`);
    
    this.containers.delete(containerId);
    
    return true;
  }
  
  async createNetwork(config) {
    const {
      name,
      driver = 'bridge',
      options = {},
      labels = {}
    } = config;
    
    const networkId = this.generateNetworkId();
    
    const networkConfig = {
      id: networkId,
      name,
      driver,
      options,
      labels,
      created: Date.now()
    };
    
    console.log(`Creating network: ${name} (${networkId})`);
    
    this.networks.set(networkId, networkConfig);
    
    return networkId;
  }
  
  async createVolume(config) {
    const {
      name,
      driver = 'local',
      options = {},
      labels = {}
    } = config;
    
    const volumeId = this.generateVolumeId();
    
    const volumeConfig = {
      id: volumeId,
      name,
      driver,
      options,
      labels,
      created: Date.now()
    };
    
    console.log(`Creating volume: ${name} (${volumeId})`);
    
    this.volumes.set(volumeId, volumeConfig);
    
    return volumeId;
  }
  
  async deployStack(stackConfig) {
    const {
      name,
      services,
      networks = {},
      volumes = {}
    } = stackConfig;
    
    console.log(`Deploying stack: ${name}`);
    
    // Create networks
    const createdNetworks = {};
    for (const [networkName, networkConfig] of Object.entries(networks)) {
      const networkId = await this.createNetwork({
        name: `${name}_${networkName}`,
        ...networkConfig
      });
      createdNetworks[networkName] = networkId;
    }
    
    // Create volumes
    const createdVolumes = {};
    for (const [volumeName, volumeConfig] of Object.entries(volumes)) {
      const volumeId = await this.createVolume({
        name: `${name}_${volumeName}`,
        ...volumeConfig
      });
      createdVolumes[volumeName] = volumeId;
    }
    
    // Deploy services
    const deployedServices = {};
    for (const [serviceName, serviceConfig] of Object.entries(services)) {
      const containerId = await this.runContainer({
        name: `${name}_${serviceName}`,
        ...serviceConfig,
        networks: serviceConfig.networks?.map(net => createdNetworks[net]) || [],
        volumes: this.mapVolumeBindings(serviceConfig.volumes, createdVolumes)
      });
      deployedServices[serviceName] = containerId;
    }
    
    return {
      stack: name,
      services: deployedServices,
      networks: createdNetworks,
      volumes: createdVolumes
    };
  }
  
  mapVolumeBindings(volumes, createdVolumes) {
    if (!volumes) return {};
    
    const mappedVolumes = {};
    for (const [mountPath, volumeSpec] of Object.entries(volumes)) {
      if (typeof volumeSpec === 'string') {
        // Named volume
        mappedVolumes[mountPath] = createdVolumes[volumeSpec];
      } else {
        // Bind mount or other config
        mappedVolumes[mountPath] = volumeSpec;
      }
    }
    return mappedVolumes;
  }
  
  startHealthCheck(containerId) {
    const container = this.containers.get(containerId);
    if (!container || !container.healthCheck) return;
    
    const interval = setInterval(async () => {
      try {
        const healthy = await this.performHealthCheck(container);
        container.healthy = healthy;
        container.lastHealthCheck = Date.now();
        
        if (!healthy) {
          console.warn(`Health check failed for container: ${container.name}`);
        }
      } catch (error) {
        console.error(`Health check error for ${container.name}:`, error);
        container.healthy = false;
      }
    }, this.options.healthCheckInterval);
    
    container.healthCheckInterval = interval;
  }
  
  async performHealthCheck(container) {
    const { healthCheck } = container;
    
    if (healthCheck.type === 'http') {
      try {
        const response = await fetch(healthCheck.url, {
          timeout: healthCheck.timeout || 5000
        });
        return response.ok;
      } catch (error) {
        return false;
      }
    }
    
    if (healthCheck.type === 'command') {
      // Execute health check command
      // In real implementation, this would exec into the container
      return Math.random() > 0.1; // 90% success rate
    }
    
    return true;
  }
  
  getContainerLogs(containerId, options = {}) {
    const container = this.containers.get(containerId);
    if (!container) {
      throw new Error(`Container ${containerId} not found`);
    }
    
    // Simulate getting logs
    const logs = [
      '2023-01-01T12:00:00Z Starting application...',
      '2023-01-01T12:00:01Z Server listening on port 3000',
      '2023-01-01T12:00:02Z Database connected successfully'
    ];
    
    return options.tail ? logs.slice(-options.tail) : logs;
  }
  
  getContainerStats(containerId) {
    const container = this.containers.get(containerId);
    if (!container) {
      throw new Error(`Container ${containerId} not found`);
    }
    
    // Simulate stats
    return {
      cpu: Math.random() * 100,
      memory: Math.random() * 1024,
      network: {
        rx: Math.random() * 1000000,
        tx: Math.random() * 1000000
      },
      disk: {
        read: Math.random() * 1000000,
        write: Math.random() * 1000000
      }
    };
  }
  
  listContainers(filters = {}) {
    let containers = Array.from(this.containers.values());
    
    if (filters.status) {
      containers = containers.filter(c => c.status === filters.status);
    }
    
    if (filters.name) {
      containers = containers.filter(c => c.name.includes(filters.name));
    }
    
    return containers.map(c => ({
      id: c.id,
      name: c.name,
      image: c.image,
      status: c.status,
      created: c.created,
      healthy: c.healthy
    }));
  }
  
  listImages() {
    return Array.from(this.images.values()).map(img => ({
      tag: img.tag,
      size: img.size,
      created: img.created
    }));
  }
  
  generateContainerId() {
    return `container_${Date.now()}_${Math.random().toString(36).substr(2, 8)}`;
  }
  
  generateImageId() {
    return `sha256:${Math.random().toString(36).substr(2, 64)}`;
  }
  
  generateNetworkId() {
    return `network_${Date.now()}_${Math.random().toString(36).substr(2, 8)}`;
  }
  
  generateVolumeId() {
    return `volume_${Date.now()}_${Math.random().toString(36).substr(2, 8)}`;
  }
  
  generateDigest() {
    return `sha256:${Math.random().toString(36).substr(2, 64)}`;
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## 📊 Monitoring & Observability

### **📈 Production Monitoring System**

```javascript
// ============= MONITORING SYSTEM =============
// lib/deployment/monitoring-system.js

export class MonitoringSystem {
  constructor(options = {}) {
    this.options = {
      metricsEndpoint: '/metrics',
      healthEndpoint: '/health',
      alertEndpoint: '/alerts',
      retentionPeriod: 7 * 24 * 60 * 60 * 1000, // 7 days
      scrapeInterval: 15000, // 15 seconds
      alertEvaluationInterval: 60000, // 1 minute
      enableTracing: true,
      enableProfiling: false,
      ...options
    };
    
    this.metrics = new Map();
    this.alerts = new Map();
    this.traces = new Map();
    this.healthChecks = new Map();
    this.dashboards = new Map();
    this.targets = new Set();
    
    this.initialize();
  }
  
  initialize() {
    this.startMetricsCollection();
    this.startAlertEvaluation();
    this.setupDefaultDashboards();
    this.setupDefaultAlerts();
  }
  
  // Metrics Collection
  recordMetric(name, value, labels = {}, timestamp = Date.now()) {
    const metricKey = this.generateMetricKey(name, labels);
    
    if (!this.metrics.has(metricKey)) {
      this.metrics.set(metricKey, {
        name,
        labels,
        type: 'gauge',
        values: []
      });
    }
    
    const metric = this.metrics.get(metricKey);
    metric.values.push({ value, timestamp });
    
    // Clean old data
    this.cleanOldMetrics(metric);
  }
  
  incrementCounter(name, labels = {}, increment = 1) {
    const metricKey = this.generateMetricKey(name, labels);
    
    if (!this.metrics.has(metricKey)) {
      this.metrics.set(metricKey, {
        name,
        labels,
        type: 'counter',
        value: 0
      });
    }
    
    const metric = this.metrics.get(metricKey);
    metric.value += increment;
    metric.lastUpdated = Date.now();
  }
  
  recordHistogram(name, value, labels = {}, buckets = [0.1, 0.5, 1, 5, 10]) {
    const metricKey = this.generateMetricKey(name, labels);
    
    if (!this.metrics.has(metricKey)) {
      this.metrics.set(metricKey, {
        name,
        labels,
        type: 'histogram',
        buckets: new Map(buckets.map(b => [b, 0])),
        sum: 0,
        count: 0
      });
    }
    
    const metric = this.metrics.get(metricKey);
    metric.sum += value;
    metric.count++;
    
    // Update buckets
    for (const [bucket, _] of metric.buckets) {
      if (value <= bucket) {
        metric.buckets.set(bucket, metric.buckets.get(bucket) + 1);
      }
    }
    
    metric.lastUpdated = Date.now();
  }
  
  generateMetricKey(name, labels) {
    const sortedLabels = Object.keys(labels).sort().map(key => `${key}="${labels[key]}"`).join(',');
    return `${name}{${sortedLabels}}`;
  }
  
  cleanOldMetrics(metric) {
    if (metric.type !== 'gauge' || !metric.values) return;
    
    const cutoff = Date.now() - this.options.retentionPeriod;
    metric.values = metric.values.filter(v => v.timestamp > cutoff);
  }
  
  // Health Checks
  addHealthCheck(name, checkFunction, interval = 30000) {
    this.healthChecks.set(name, {
      name,
      checkFunction,
      interval,
      lastCheck: null,
      status: 'unknown',
      lastError: null,
      timer: null
    });
    
    this.startHealthCheck(name);
  }
  
  startHealthCheck(name) {
    const healthCheck = this.healthChecks.get(name);
    if (!healthCheck) return;
    
    const performCheck = async () => {
      try {
        const result = await healthCheck.checkFunction();
        healthCheck.status = result.healthy ? 'healthy' : 'unhealthy';
        healthCheck.lastCheck = Date.now();
        healthCheck.lastError = result.error || null;
        
        this.recordMetric('health_check_status', result.healthy ? 1 : 0, {
          check: name
        });
        
        if (result.metrics) {
          Object.entries(result.metrics).forEach(([metricName, value]) => {
            this.recordMetric(`health_check_${metricName}`, value, {
              check: name
            });
          });
        }
      } catch (error) {
        healthCheck.status = 'unhealthy';
        healthCheck.lastCheck = Date.now();
        healthCheck.lastError = error.message;
        
        this.recordMetric('health_check_status', 0, { check: name });
      }
    };
    
    // Initial check
    performCheck();
    
    // Schedule periodic checks
    healthCheck.timer = setInterval(performCheck, healthCheck.interval);
  }
  
  // Alert Management
  addAlert(name, config) {
    this.alerts.set(name, {
      name,
      query: config.query,
      condition: config.condition,
      threshold: config.threshold,
      duration: config.duration || 300000, // 5 minutes
      severity: config.severity || 'warning',
      labels: config.labels || {},
      annotations: config.annotations || {},
      state: 'inactive',
      activeSince: null,
      lastEvaluation: null,
      lastTriggered: null,
      triggers: 0
    });
  }
  
  startAlertEvaluation() {
    setInterval(() => {
      this.evaluateAlerts();
    }, this.options.alertEvaluationInterval);
  }
  
  async evaluateAlerts() {
    for (const [name, alert] of this.alerts) {
      try {
        await this.evaluateAlert(alert);
      } catch (error) {
        console.error(`Error evaluating alert ${name}:`, error);
      }
    }
  }
  
  async evaluateAlert(alert) {
    const result = await this.executeQuery(alert.query);
    alert.lastEvaluation = Date.now();
    
    const shouldTrigger = this.evaluateCondition(result, alert.condition, alert.threshold);
    
    if (shouldTrigger && alert.state === 'inactive') {
      alert.state = 'pending';
      alert.activeSince = Date.now();
    } else if (!shouldTrigger && alert.state !== 'inactive') {
      alert.state = 'inactive';
      alert.activeSince = null;
    } else if (shouldTrigger && alert.state === 'pending') {
      const activeDuration = Date.now() - alert.activeSince;
      if (activeDuration >= alert.duration) {
        alert.state = 'active';
        alert.lastTriggered = Date.now();
        alert.triggers++;
        
        await this.fireAlert(alert);
      }
    }
  }
  
  evaluateCondition(value, condition, threshold) {
    switch (condition) {
      case 'gt': return value > threshold;
      case 'gte': return value >= threshold;
      case 'lt': return value < threshold;
      case 'lte': return value <= threshold;
      case 'eq': return value === threshold;
      case 'ne': return value !== threshold;
      default: return false;
    }
  }
  
  async executeQuery(query) {
    // Simple query execution - in real implementation,
    // this would query a time-series database
    
    if (query.startsWith('avg(')) {
      const metricName = query.match(/avg\(([^)]+)\)/)[1];
      return this.getAverageMetric(metricName);
    }
    
    if (query.startsWith('rate(')) {
      const metricName = query.match(/rate\(([^)]+)\)/)[1];
      return this.getRateMetric(metricName);
    }
    
    return this.getLatestMetric(query);
  }
  
  getAverageMetric(metricName) {
    const metrics = Array.from(this.metrics.values())
      .filter(m => m.name === metricName && m.type === 'gauge');
    
    if (metrics.length === 0) return 0;
    
    const now = Date.now();
    const recent = metrics.flatMap(m => 
      m.values.filter(v => now - v.timestamp < 300000) // Last 5 minutes
    );
    
    if (recent.length === 0) return 0;
    
    return recent.reduce((sum, v) => sum + v.value, 0) / recent.length;
  }
  
  getRateMetric(metricName) {
    const metrics = Array.from(this.metrics.values())
      .filter(m => m.name === metricName && m.type === 'counter');
    
    if (metrics.length === 0) return 0;
    
    // Calculate rate per second
    return metrics.reduce((sum, m) => sum + (m.value || 0), 0) / 60; // Approximate
  }
  
  getLatestMetric(metricName) {
    const metric = Array.from(this.metrics.values())
      .find(m => m.name === metricName);
    
    if (!metric) return 0;
    
    if (metric.type === 'gauge' && metric.values.length > 0) {
      return metric.values[metric.values.length - 1].value;
    }
    
    if (metric.type === 'counter') {
      return metric.value || 0;
    }
    
    return 0;
  }
  
  async fireAlert(alert) {
    console.log(`🚨 ALERT: ${alert.name} (${alert.severity})`);
    
    const alertPayload = {
      name: alert.name,
      severity: alert.severity,
      state: alert.state,
      activeSince: alert.activeSince,
      labels: alert.labels,
      annotations: alert.annotations,
      timestamp: Date.now()
    };
    
    // Send to alert manager or notification system
    await this.sendAlert(alertPayload);
    
    // Record alert metric
    this.incrementCounter('alerts_fired_total', {
      alertname: alert.name,
      severity: alert.severity
    });
  }
  
  async sendAlert(alertPayload) {
    // Send to external alert manager
    try {
      const response = await fetch(this.options.alertEndpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify([alertPayload])
      });
      
      if (!response.ok) {
        throw new Error(`Alert webhook failed: ${response.statusText}`);
      }
    } catch (error) {
      console.error('Failed to send alert:', error);
    }
  }
  
  // Distributed Tracing
  startTrace(operationName, parentSpan = null) {
    const traceId = parentSpan?.traceId || this.generateTraceId();
    const spanId = this.generateSpanId();
    
    const span = {
      traceId,
      spanId,
      parentSpanId: parentSpan?.spanId || null,
      operationName,
      startTime: Date.now(),
      endTime: null,
      duration: null,
      tags: new Map(),
      logs: [],
      status: 'active'
    };
    
    this.traces.set(spanId, span);
    
    return {
      traceId,
      spanId,
      setTag: (key, value) => span.tags.set(key, value),
      log: (message, timestamp = Date.now()) => span.logs.push({ message, timestamp }),
      finish: () => this.finishSpan(spanId),
      child: (childOperationName) => this.startTrace(childOperationName, span)
    };
  }
  
  finishSpan(spanId) {
    const span = this.traces.get(spanId);
    if (span) {
      span.endTime = Date.now();
      span.duration = span.endTime - span.startTime;
      span.status = 'finished';
      
      // Record trace metrics
      this.recordHistogram('trace_duration_seconds', span.duration / 1000, {
        operation: span.operationName
      });
    }
  }
  
  // Dashboard Management
  createDashboard(name, config) {
    this.dashboards.set(name, {
      name,
      title: config.title || name,
      panels: config.panels || [],
      created: Date.now(),
      updated: Date.now()
    });
  }
  
  setupDefaultDashboards() {
    this.createDashboard('application', {
      title: 'Application Overview',
      panels: [
        {
          title: 'Request Rate',
          query: 'rate(http_requests_total)',
          type: 'graph'
        },
        {
          title: 'Response Time',
          query: 'avg(http_request_duration_seconds)',
          type: 'graph'
        },
        {
          title: 'Error Rate',
          query: 'rate(http_requests_total{status=~"5.."})',
          type: 'graph'
        },
        {
          title: 'Active Users',
          query: 'active_users',
          type: 'stat'
        }
      ]
    });
    
    this.createDashboard('infrastructure', {
      title: 'Infrastructure Metrics',
      panels: [
        {
          title: 'CPU Usage',
          query: 'avg(cpu_usage_percent)',
          type: 'graph'
        },
        {
          title: 'Memory Usage',
          query: 'avg(memory_usage_percent)',
          type: 'graph'
        },
        {
          title: 'Disk I/O',
          query: 'rate(disk_io_total)',
          type: 'graph'
        },
        {
          title: 'Network Traffic',
          query: 'rate(network_bytes_total)',
          type: 'graph'
        }
      ]
    });
  }
  
  setupDefaultAlerts() {
    this.addAlert('high_error_rate', {
      query: 'rate(http_requests_total{status=~"5.."})',
      condition: 'gt',
      threshold: 0.05, // 5% error rate
      duration: 300000, // 5 minutes
      severity: 'critical',
      labels: { team: 'backend' },
      annotations: {
        summary: 'High error rate detected',
        description: 'Error rate is above 5% for 5 minutes'
      }
    });
    
    this.addAlert('high_response_time', {
      query: 'avg(http_request_duration_seconds)',
      condition: 'gt',
      threshold: 2.0, // 2 seconds
      duration: 600000, // 10 minutes
      severity: 'warning',
      labels: { team: 'backend' },
      annotations: {
        summary: 'High response time detected',
        description: 'Average response time is above 2 seconds'
      }
    });
    
    this.addAlert('low_disk_space', {
      query: 'disk_usage_percent',
      condition: 'gt',
      threshold: 90, // 90% disk usage
      duration: 0, // Immediate
      severity: 'critical',
      labels: { team: 'infrastructure' },
      annotations: {
        summary: 'Low disk space',
        description: 'Disk usage is above 90%'
      }
    });
  }
  
  startMetricsCollection() {
    setInterval(() => {
      this.collectSystemMetrics();
    }, this.options.scrapeInterval);
  }
  
  collectSystemMetrics() {
    // Simulate system metrics collection
    this.recordMetric('cpu_usage_percent', Math.random() * 100);
    this.recordMetric('memory_usage_percent', Math.random() * 100);
    this.recordMetric('disk_usage_percent', 20 + Math.random() * 70);
    this.recordMetric('active_connections', Math.floor(Math.random() * 1000));
    
    // HTTP metrics
    this.incrementCounter('http_requests_total', { 
      method: 'GET', 
      status: '200' 
    }, Math.floor(Math.random() * 10));
    
    this.recordHistogram('http_request_duration_seconds', 
      Math.random() * 2, 
      { method: 'GET' }
    );
  }
  
  // API endpoints
  getMetrics() {
    const output = [];
    
    for (const [key, metric] of this.metrics) {
      if (metric.type === 'counter') {
        output.push(`${metric.name}{${this.formatLabels(metric.labels)}} ${metric.value || 0}`);
      } else if (metric.type === 'gauge') {
        const latestValue = metric.values.length > 0 
          ? metric.values[metric.values.length - 1].value 
          : 0;
        output.push(`${metric.name}{${this.formatLabels(metric.labels)}} ${latestValue}`);
      } else if (metric.type === 'histogram') {
        const labels = this.formatLabels(metric.labels);
        output.push(`${metric.name}_sum{${labels}} ${metric.sum}`);
        output.push(`${metric.name}_count{${labels}} ${metric.count}`);
        
        for (const [bucket, count] of metric.buckets) {
          output.push(`${metric.name}_bucket{${labels},le="${bucket}"} ${count}`);
        }
      }
    }
    
    return output.join('\n');
  }
  
  formatLabels(labels) {
    return Object.entries(labels)
      .map(([key, value]) => `${key}="${value}"`)
      .join(',');
  }
  
  getHealth() {
    const healthChecks = Array.from(this.healthChecks.values()).map(hc => ({
      name: hc.name,
      status: hc.status,
      lastCheck: hc.lastCheck,
      error: hc.lastError
    }));
    
    const overallStatus = healthChecks.every(hc => hc.status === 'healthy') 
      ? 'healthy' 
      : 'unhealthy';
    
    return {
      status: overallStatus,
      timestamp: Date.now(),
      checks: healthChecks
    };
  }
  
  getAlerts() {
    return Array.from(this.alerts.values()).map(alert => ({
      name: alert.name,
      state: alert.state,
      severity: alert.severity,
      activeSince: alert.activeSince,
      lastTriggered: alert.lastTriggered,
      triggers: alert.triggers
    }));
  }
  
  generateTraceId() {
    return Math.random().toString(16).substring(2, 18);
  }
  
  generateSpanId() {
    return Math.random().toString(16).substring(2, 10);
  }
}
```

## 🎯 แบบฝึกหัด

### **🚀 แบบฝึกหัดที่ 1: Deployment Automation**
สร้าง comprehensive deployment system:

```javascript
// TODO: สร้าง deployment automation

// 1. GitOps deployment
class GitOpsDeployer {
  // Git-based deployment automation
  // Rollback capabilities
  // Environment promotion
}

// 2. Canary deployment
class CanaryDeployer {
  // Traffic splitting
  // Automated rollback
  // Performance monitoring
}

// 3. Infrastructure as Code
class InfrastructureManager {
  // Terraform integration
  // Cloud resource management
  // Cost optimization
}
```

### **📊 แบบฝึกหัดที่ 2: Observability Platform**
สร้าง comprehensive monitoring system:

```javascript
// TODO: สร้าง observability platform

// 1. Distributed tracing
class DistributedTracing {
  // OpenTelemetry integration
  // Trace sampling
  // Performance analysis
}

// 2. Log aggregation
class LogAggregator {
  // Centralized logging
  // Log parsing and indexing
  // Alert correlation
}

// 3. Custom dashboards
class DashboardBuilder {
  // Dynamic dashboard creation
  // Custom visualizations
  // Real-time updates
}
```

### **🔒 แบบฝึกหัดที่ 3: Production Reliability**
สร้าง production reliability system:

```javascript
// TODO: สร้าง reliability system

// 1. Chaos engineering
class ChaosEngineer {
  // Fault injection
  // Resilience testing
  // Failure simulation
}

// 2. SRE automation
class SREAutomation {
  // Incident response
  // Runbook automation
  // Capacity planning
}

// 3. Disaster recovery
class DisasterRecovery {
  // Backup strategies
  // Recovery procedures
  // Business continuity
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 27: Scalability](./27-scalability.md)

**บทถัดไป**: [บทที่ 29: Enterprise Integration](./29-enterprise-integration.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [GitOps Principles](https://www.gitops.tech/)
- [Docker Best Practices](https://docs.docker.com/develop/best-practices/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Prometheus Monitoring](https://prometheus.io/docs/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **CI/CD Pipelines** - automated build, test, และ deployment processes
- **Containerization** - Docker และ container orchestration
- **Monitoring & Observability** - comprehensive monitoring systems
- **Production Deployment** - strategies สำหรับ reliable deployments
- **DevOps Practices** - automation และ best practices สำหรับ production

Production Deployment เป็นขั้นตอนสำคัญสำหรับ successful software delivery! 🚀
