# บทที่ 20: Networking

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ advanced networking concepts และ protocols
- ทำความเข้าใจ HTTP/2, WebRTC, และ GraphQL
- เรียนรู้ network optimization และ performance strategies
- ประยุกต์ใช้ใน Next.js สำหรับ high-performance applications

## 🌐 HTTP/2 และ Advanced Protocols

### **⚡ HTTP/2 Optimization**

```javascript
// ============= HTTP/2 CLIENT =============
// lib/networking/http2-client.js

export class HTTP2Client {
  constructor(baseURL, options = {}) {
    this.baseURL = baseURL.replace(/\/$/, '');
    this.options = {
      maxConcurrentStreams: 100,
      enablePush: true,
      enableMultiplexing: true,
      compressionLevel: 6,
      ...options
    };
    
    this.activeStreams = new Map();
    this.streamQueue = [];
    this.pushCache = new Map();
    this.multiplexingEnabled = this.detectHTTP2Support();
  }
  
  detectHTTP2Support() {
    // Check if the browser supports HTTP/2
    return 'serviceWorker' in navigator && 
           'PushManager' in window &&
           window.chrome?.loadTimes;
  }
  
  async request(path, options = {}) {
    const url = `${this.baseURL}${path}`;
    const requestId = this.generateRequestId();
    
    // Check push cache first
    if (options.method === 'GET') {
      const cached = this.pushCache.get(url);
      if (cached && !this.isCacheExpired(cached)) {
        return cached.response.clone();
      }
    }
    
    // Use multiplexing if available
    if (this.multiplexingEnabled && this.canMultiplex()) {
      return this.multiplexRequest(url, options, requestId);
    }
    
    // Fallback to regular fetch
    return this.regularRequest(url, options);
  }
  
  async multiplexRequest(url, options, requestId) {
    return new Promise((resolve, reject) => {
      const stream = {
        id: requestId,
        url,
        options,
        resolve,
        reject,
        priority: options.priority || 5,
        timestamp: Date.now()
      };
      
      if (this.activeStreams.size < this.options.maxConcurrentStreams) {
        this.executeStream(stream);
      } else {
        this.queueStream(stream);
      }
    });
  }
  
  async executeStream(stream) {
    this.activeStreams.set(stream.id, stream);
    
    try {
      // Add HTTP/2 specific headers
      const headers = {
        ':method': stream.options.method || 'GET',
        ':path': new URL(stream.url).pathname,
        ':scheme': 'https',
        ':authority': new URL(stream.url).host,
        ...stream.options.headers
      };
      
      // Create fetch request with HTTP/2 optimizations
      const response = await fetch(stream.url, {
        ...stream.options,
        headers,
        // Enable server push if supported
        cache: this.options.enablePush ? 'force-cache' : 'default'
      });
      
      // Handle server push
      if (response.headers.get('link')) {
        this.handleServerPush(response.headers.get('link'));
      }
      
      stream.resolve(response);
    } catch (error) {
      stream.reject(error);
    } finally {
      this.activeStreams.delete(stream.id);
      this.processQueue();
    }
  }
  
  queueStream(stream) {
    // Insert into queue based on priority
    const insertIndex = this.streamQueue.findIndex(
      queued => queued.priority > stream.priority
    );
    
    if (insertIndex === -1) {
      this.streamQueue.push(stream);
    } else {
      this.streamQueue.splice(insertIndex, 0, stream);
    }
  }
  
  processQueue() {
    while (this.streamQueue.length > 0 && 
           this.activeStreams.size < this.options.maxConcurrentStreams) {
      const stream = this.streamQueue.shift();
      this.executeStream(stream);
    }
  }
  
  handleServerPush(linkHeader) {
    // Parse Link header for push promises
    const links = this.parseLinkHeader(linkHeader);
    
    links.forEach(link => {
      if (link.rel === 'preload') {
        this.cachePushPromise(link.href);
      }
    });
  }
  
  async cachePushPromise(url) {
    try {
      const response = await fetch(url);
      this.pushCache.set(url, {
        response: response.clone(),
        timestamp: Date.now(),
        ttl: 300000 // 5 minutes
      });
    } catch (error) {
      console.warn('Failed to cache push promise:', error);
    }
  }
  
  parseLinkHeader(header) {
    return header.split(',').map(part => {
      const [url, ...params] = part.trim().split(';');
      const link = { href: url.slice(1, -1) }; // Remove < >
      
      params.forEach(param => {
        const [key, value] = param.trim().split('=');
        link[key] = value?.replace(/"/g, '');
      });
      
      return link;
    });
  }
  
  canMultiplex() {
    return this.activeStreams.size < this.options.maxConcurrentStreams;
  }
  
  isCacheExpired(cached) {
    return Date.now() - cached.timestamp > cached.ttl;
  }
  
  generateRequestId() {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  regularRequest(url, options) {
    return fetch(url, options);
  }
  
  // Resource prioritization
  setPriority(priority) {
    return (url, options = {}) => {
      return this.request(url, { ...options, priority });
    };
  }
  
  // High priority for critical resources
  critical = this.setPriority(1);
  
  // Medium priority for important resources
  important = this.setPriority(3);
  
  // Low priority for non-critical resources
  background = this.setPriority(7);
  
  // Statistics
  getStats() {
    return {
      activeStreams: this.activeStreams.size,
      queuedStreams: this.streamQueue.length,
      pushCacheSize: this.pushCache.size,
      multiplexingEnabled: this.multiplexingEnabled
    };
  }
  
  clearPushCache() {
    this.pushCache.clear();
  }
}

// ============= CONNECTION POOLING =============
// lib/networking/connection-pool.js

export class ConnectionPool {
  constructor(options = {}) {
    this.options = {
      maxConnections: 6,
      maxConnectionsPerHost: 2,
      connectionTimeout: 30000,
      idleTimeout: 60000,
      retryAttempts: 3,
      ...options
    };
    
    this.connections = new Map();
    this.hostConnections = new Map();
    this.waitingQueue = [];
    this.stats = {
      created: 0,
      reused: 0,
      destroyed: 0,
      failed: 0
    };
  }
  
  async getConnection(host) {
    const hostKey = this.normalizeHost(host);
    
    // Check for available connection
    const available = this.findAvailableConnection(hostKey);
    if (available) {
      this.stats.reused++;
      return available;
    }
    
    // Check if we can create new connection
    if (this.canCreateConnection(hostKey)) {
      return this.createConnection(hostKey);
    }
    
    // Wait for available connection
    return this.waitForConnection(hostKey);
  }
  
  findAvailableConnection(hostKey) {
    const hostConnections = this.hostConnections.get(hostKey) || [];
    return hostConnections.find(conn => 
      conn.state === 'idle' && 
      !this.isConnectionExpired(conn)
    );
  }
  
  canCreateConnection(hostKey) {
    const totalConnections = this.connections.size;
    const hostConnections = this.hostConnections.get(hostKey)?.length || 0;
    
    return totalConnections < this.options.maxConnections &&
           hostConnections < this.options.maxConnectionsPerHost;
  }
  
  async createConnection(hostKey) {
    const connection = {
      id: this.generateConnectionId(),
      host: hostKey,
      state: 'connecting',
      createdAt: Date.now(),
      lastUsed: Date.now(),
      requests: 0,
      keepAlive: true
    };
    
    try {
      // Simulate connection establishment
      await this.establishConnection(connection);
      
      connection.state = 'idle';
      this.addConnection(connection);
      this.stats.created++;
      
      return connection;
    } catch (error) {
      this.stats.failed++;
      throw error;
    }
  }
  
  async establishConnection(connection) {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => {
        reject(new Error('Connection timeout'));
      }, this.options.connectionTimeout);
      
      // Simulate connection delay
      setTimeout(() => {
        clearTimeout(timeout);
        resolve();
      }, Math.random() * 100);
    });
  }
  
  addConnection(connection) {
    this.connections.set(connection.id, connection);
    
    const hostKey = connection.host;
    if (!this.hostConnections.has(hostKey)) {
      this.hostConnections.set(hostKey, []);
    }
    this.hostConnections.get(hostKey).push(connection);
  }
  
  async waitForConnection(hostKey) {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => {
        reject(new Error('Connection wait timeout'));
      }, this.options.connectionTimeout);
      
      this.waitingQueue.push({
        hostKey,
        resolve: (conn) => {
          clearTimeout(timeout);
          resolve(conn);
        },
        reject: (error) => {
          clearTimeout(timeout);
          reject(error);
        }
      });
    });
  }
  
  releaseConnection(connection) {
    connection.state = 'idle';
    connection.lastUsed = Date.now();
    
    // Process waiting queue
    const waiting = this.waitingQueue.find(w => w.hostKey === connection.host);
    if (waiting) {
      const index = this.waitingQueue.indexOf(waiting);
      this.waitingQueue.splice(index, 1);
      waiting.resolve(connection);
    }
  }
  
  useConnection(connection) {
    connection.state = 'active';
    connection.lastUsed = Date.now();
    connection.requests++;
  }
  
  destroyConnection(connection) {
    connection.state = 'destroyed';
    
    this.connections.delete(connection.id);
    
    const hostConnections = this.hostConnections.get(connection.host);
    if (hostConnections) {
      const index = hostConnections.indexOf(connection);
      if (index !== -1) {
        hostConnections.splice(index, 1);
      }
    }
    
    this.stats.destroyed++;
  }
  
  isConnectionExpired(connection) {
    return Date.now() - connection.lastUsed > this.options.idleTimeout;
  }
  
  normalizeHost(host) {
    try {
      const url = new URL(host);
      return `${url.protocol}//${url.host}`;
    } catch {
      return host;
    }
  }
  
  generateConnectionId() {
    return `conn_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  // Cleanup expired connections
  cleanup() {
    const now = Date.now();
    
    for (const [id, connection] of this.connections) {
      if (connection.state === 'idle' && this.isConnectionExpired(connection)) {
        this.destroyConnection(connection);
      }
    }
  }
  
  // Start periodic cleanup
  startCleanup(interval = 60000) {
    this.cleanupTimer = setInterval(() => {
      this.cleanup();
    }, interval);
  }
  
  stopCleanup() {
    if (this.cleanupTimer) {
      clearInterval(this.cleanupTimer);
    }
  }
  
  getStats() {
    return {
      ...this.stats,
      activeConnections: Array.from(this.connections.values())
        .filter(conn => conn.state === 'active').length,
      idleConnections: Array.from(this.connections.values())
        .filter(conn => conn.state === 'idle').length,
      totalConnections: this.connections.size,
      waitingRequests: this.waitingQueue.length
    };
  }
}
```

## 🔗 WebRTC และ Peer-to-Peer

### **📡 WebRTC Implementation**

```javascript
// ============= WEBRTC CLIENT =============
// lib/webrtc/client.js

export class WebRTCClient extends EventTarget {
  constructor(options = {}) {
    super();
    
    this.options = {
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        { urls: 'stun:stun1.l.google.com:19302' }
      ],
      dataChannelOptions: {
        ordered: true,
        maxRetransmits: 3
      },
      mediaConstraints: {
        video: true,
        audio: true
      },
      ...options
    };
    
    this.peerConnection = null;
    this.dataChannel = null;
    this.localStream = null;
    this.remoteStream = null;
    this.connectionState = 'new';
    this.isInitiator = false;
    this.pendingCandidates = [];
  }
  
  async init(isInitiator = false) {
    this.isInitiator = isInitiator;
    
    // Create peer connection
    this.peerConnection = new RTCPeerConnection({
      iceServers: this.options.iceServers,
      iceCandidatePoolSize: 10
    });
    
    // Set up event handlers
    this.setupPeerConnectionHandlers();
    
    // Create data channel if initiator
    if (this.isInitiator) {
      this.createDataChannel();
    }
    
    this.connectionState = 'initialized';
    this.dispatchEvent(new CustomEvent('initialized'));
  }
  
  setupPeerConnectionHandlers() {
    this.peerConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.dispatchEvent(new CustomEvent('icecandidate', {
          detail: event.candidate
        }));
      }
    };
    
    this.peerConnection.onconnectionstatechange = () => {
      this.connectionState = this.peerConnection.connectionState;
      this.dispatchEvent(new CustomEvent('connectionstatechange', {
        detail: this.connectionState
      }));
    };
    
    this.peerConnection.ontrack = (event) => {
      this.remoteStream = event.streams[0];
      this.dispatchEvent(new CustomEvent('remotestream', {
        detail: this.remoteStream
      }));
    };
    
    this.peerConnection.ondatachannel = (event) => {
      const channel = event.channel;
      this.setupDataChannelHandlers(channel);
    };
    
    this.peerConnection.onicegatheringstatechange = () => {
      this.dispatchEvent(new CustomEvent('icegatheringstatechange', {
        detail: this.peerConnection.iceGatheringState
      }));
    };
  }
  
  createDataChannel(label = 'data') {
    this.dataChannel = this.peerConnection.createDataChannel(
      label,
      this.options.dataChannelOptions
    );
    
    this.setupDataChannelHandlers(this.dataChannel);
    return this.dataChannel;
  }
  
  setupDataChannelHandlers(channel) {
    if (!this.dataChannel) {
      this.dataChannel = channel;
    }
    
    channel.onopen = () => {
      this.dispatchEvent(new CustomEvent('datachannelopen'));
    };
    
    channel.onclose = () => {
      this.dispatchEvent(new CustomEvent('datachannelclose'));
    };
    
    channel.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        this.dispatchEvent(new CustomEvent('datamessage', { detail: data }));
      } catch (error) {
        this.dispatchEvent(new CustomEvent('datamessage', { 
          detail: event.data 
        }));
      }
    };
    
    channel.onerror = (error) => {
      this.dispatchEvent(new CustomEvent('datachannelerror', { 
        detail: error 
      }));
    };
  }
  
  async addLocalMedia(constraints = this.options.mediaConstraints) {
    try {
      this.localStream = await navigator.mediaDevices.getUserMedia(constraints);
      
      // Add tracks to peer connection
      this.localStream.getTracks().forEach(track => {
        this.peerConnection.addTrack(track, this.localStream);
      });
      
      this.dispatchEvent(new CustomEvent('localstream', {
        detail: this.localStream
      }));
      
      return this.localStream;
    } catch (error) {
      throw new Error(`Failed to get local media: ${error.message}`);
    }
  }
  
  async addScreenShare() {
    try {
      const screenStream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
        audio: true
      });
      
      // Replace video track
      const videoTrack = screenStream.getVideoTracks()[0];
      const sender = this.peerConnection.getSenders()
        .find(s => s.track && s.track.kind === 'video');
      
      if (sender) {
        await sender.replaceTrack(videoTrack);
      } else {
        this.peerConnection.addTrack(videoTrack, screenStream);
      }
      
      // Handle screen share end
      videoTrack.onended = () => {
        this.dispatchEvent(new CustomEvent('screenshareended'));
      };
      
      this.dispatchEvent(new CustomEvent('screensharestarted', {
        detail: screenStream
      }));
      
      return screenStream;
    } catch (error) {
      throw new Error(`Failed to start screen share: ${error.message}`);
    }
  }
  
  async createOffer() {
    if (!this.peerConnection) {
      throw new Error('Peer connection not initialized');
    }
    
    const offer = await this.peerConnection.createOffer();
    await this.peerConnection.setLocalDescription(offer);
    
    return offer;
  }
  
  async createAnswer() {
    if (!this.peerConnection) {
      throw new Error('Peer connection not initialized');
    }
    
    const answer = await this.peerConnection.createAnswer();
    await this.peerConnection.setLocalDescription(answer);
    
    return answer;
  }
  
  async setRemoteDescription(description) {
    await this.peerConnection.setRemoteDescription(description);
    
    // Add pending candidates
    for (const candidate of this.pendingCandidates) {
      await this.peerConnection.addIceCandidate(candidate);
    }
    this.pendingCandidates = [];
  }
  
  async addIceCandidate(candidate) {
    if (this.peerConnection.remoteDescription) {
      await this.peerConnection.addIceCandidate(candidate);
    } else {
      this.pendingCandidates.push(candidate);
    }
  }
  
  sendData(data) {
    if (this.dataChannel && this.dataChannel.readyState === 'open') {
      const message = typeof data === 'string' ? data : JSON.stringify(data);
      this.dataChannel.send(message);
      return true;
    }
    return false;
  }
  
  toggleAudio() {
    if (this.localStream) {
      const audioTracks = this.localStream.getAudioTracks();
      if (audioTracks.length > 0) {
        audioTracks[0].enabled = !audioTracks[0].enabled;
        return audioTracks[0].enabled;
      }
    }
    return false;
  }
  
  toggleVideo() {
    if (this.localStream) {
      const videoTracks = this.localStream.getVideoTracks();
      if (videoTracks.length > 0) {
        videoTracks[0].enabled = !videoTracks[0].enabled;
        return videoTracks[0].enabled;
      }
    }
    return false;
  }
  
  getStats() {
    if (this.peerConnection) {
      return this.peerConnection.getStats();
    }
    return null;
  }
  
  close() {
    if (this.localStream) {
      this.localStream.getTracks().forEach(track => track.stop());
    }
    
    if (this.dataChannel) {
      this.dataChannel.close();
    }
    
    if (this.peerConnection) {
      this.peerConnection.close();
    }
    
    this.connectionState = 'closed';
    this.dispatchEvent(new CustomEvent('closed'));
  }
}

// ============= P2P NETWORK =============
// lib/webrtc/p2p-network.js

export class P2PNetwork extends EventTarget {
  constructor(signalServer, options = {}) {
    super();
    
    this.signalServer = signalServer;
    this.options = {
      maxPeers: 8,
      autoConnect: true,
      heartbeatInterval: 10000,
      ...options
    };
    
    this.peers = new Map();
    this.nodeId = this.generateNodeId();
    this.isConnected = false;
    this.heartbeatTimer = null;
    this.messageHandlers = new Map();
  }
  
  async connect() {
    try {
      // Connect to signaling server
      await this.connectToSignalServer();
      
      // Start peer discovery
      this.startPeerDiscovery();
      
      // Start heartbeat
      this.startHeartbeat();
      
      this.isConnected = true;
      this.dispatchEvent(new CustomEvent('connected'));
    } catch (error) {
      throw new Error(`Failed to connect to P2P network: ${error.message}`);
    }
  }
  
  async connectToSignalServer() {
    return new Promise((resolve, reject) => {
      this.signalServer.onopen = () => {
        this.signalServer.send(JSON.stringify({
          type: 'join',
          nodeId: this.nodeId
        }));
        resolve();
      };
      
      this.signalServer.onerror = reject;
      
      this.signalServer.onmessage = (event) => {
        this.handleSignalMessage(JSON.parse(event.data));
      };
    });
  }
  
  handleSignalMessage(message) {
    switch (message.type) {
      case 'peer-list':
        this.handlePeerList(message.peers);
        break;
      case 'peer-joined':
        this.handlePeerJoined(message.peer);
        break;
      case 'peer-left':
        this.handlePeerLeft(message.peer);
        break;
      case 'offer':
        this.handleOffer(message);
        break;
      case 'answer':
        this.handleAnswer(message);
        break;
      case 'ice-candidate':
        this.handleIceCandidate(message);
        break;
    }
  }
  
  async handlePeerList(peers) {
    for (const peer of peers) {
      if (peer.id !== this.nodeId && this.peers.size < this.options.maxPeers) {
        await this.connectToPeer(peer.id, true);
      }
    }
  }
  
  async handlePeerJoined(peer) {
    if (peer.id !== this.nodeId && this.peers.size < this.options.maxPeers) {
      // Only initiator creates the connection
      if (this.nodeId < peer.id) {
        await this.connectToPeer(peer.id, true);
      }
    }
  }
  
  handlePeerLeft(peer) {
    const peerConnection = this.peers.get(peer.id);
    if (peerConnection) {
      peerConnection.close();
      this.peers.delete(peer.id);
      
      this.dispatchEvent(new CustomEvent('peerdisconnected', {
        detail: peer.id
      }));
    }
  }
  
  async connectToPeer(peerId, isInitiator) {
    try {
      const rtcClient = new WebRTCClient();
      await rtcClient.init(isInitiator);
      
      // Set up event handlers
      rtcClient.addEventListener('icecandidate', (event) => {
        this.signalServer.send(JSON.stringify({
          type: 'ice-candidate',
          targetPeer: peerId,
          candidate: event.detail
        }));
      });
      
      rtcClient.addEventListener('datamessage', (event) => {
        this.handlePeerMessage(peerId, event.detail);
      });
      
      rtcClient.addEventListener('connectionstatechange', (event) => {
        if (event.detail === 'connected') {
          this.dispatchEvent(new CustomEvent('peerconnected', {
            detail: peerId
          }));
        }
      });
      
      this.peers.set(peerId, rtcClient);
      
      if (isInitiator) {
        const offer = await rtcClient.createOffer();
        this.signalServer.send(JSON.stringify({
          type: 'offer',
          targetPeer: peerId,
          offer
        }));
      }
      
    } catch (error) {
      console.error(`Failed to connect to peer ${peerId}:`, error);
    }
  }
  
  async handleOffer(message) {
    const peer = this.peers.get(message.fromPeer);
    if (!peer) {
      await this.connectToPeer(message.fromPeer, false);
      return;
    }
    
    await peer.setRemoteDescription(message.offer);
    const answer = await peer.createAnswer();
    
    this.signalServer.send(JSON.stringify({
      type: 'answer',
      targetPeer: message.fromPeer,
      answer
    }));
  }
  
  async handleAnswer(message) {
    const peer = this.peers.get(message.fromPeer);
    if (peer) {
      await peer.setRemoteDescription(message.answer);
    }
  }
  
  async handleIceCandidate(message) {
    const peer = this.peers.get(message.fromPeer);
    if (peer) {
      await peer.addIceCandidate(message.candidate);
    }
  }
  
  handlePeerMessage(peerId, message) {
    if (message.type && this.messageHandlers.has(message.type)) {
      const handler = this.messageHandlers.get(message.type);
      handler(peerId, message.payload);
    }
    
    this.dispatchEvent(new CustomEvent('message', {
      detail: { peerId, message }
    }));
  }
  
  // Message handling
  onMessage(type, handler) {
    this.messageHandlers.set(type, handler);
  }
  
  broadcast(message) {
    const messageData = {
      id: this.generateMessageId(),
      fromPeer: this.nodeId,
      timestamp: Date.now(),
      ...message
    };
    
    this.peers.forEach((peer, peerId) => {
      peer.sendData(messageData);
    });
  }
  
  sendToPeer(peerId, message) {
    const peer = this.peers.get(peerId);
    if (peer) {
      const messageData = {
        id: this.generateMessageId(),
        fromPeer: this.nodeId,
        timestamp: Date.now(),
        ...message
      };
      
      return peer.sendData(messageData);
    }
    return false;
  }
  
  startPeerDiscovery() {
    this.signalServer.send(JSON.stringify({
      type: 'get-peers'
    }));
  }
  
  startHeartbeat() {
    this.heartbeatTimer = setInterval(() => {
      this.broadcast({
        type: 'heartbeat',
        nodeId: this.nodeId
      });
    }, this.options.heartbeatInterval);
  }
  
  stopHeartbeat() {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
    }
  }
  
  disconnect() {
    this.stopHeartbeat();
    
    this.peers.forEach(peer => peer.close());
    this.peers.clear();
    
    if (this.signalServer) {
      this.signalServer.close();
    }
    
    this.isConnected = false;
    this.dispatchEvent(new CustomEvent('disconnected'));
  }
  
  generateNodeId() {
    return `node_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  generateMessageId() {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  getNetworkStats() {
    return {
      nodeId: this.nodeId,
      connected: this.isConnected,
      peerCount: this.peers.size,
      peers: Array.from(this.peers.keys())
    };
  }
}
```

## 📊 GraphQL Integration

### **🔗 GraphQL Client**

```javascript
// ============= GRAPHQL CLIENT =============
// lib/graphql/client.js

export class GraphQLClient {
  constructor(endpoint, options = {}) {
    this.endpoint = endpoint;
    this.options = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      cache: 'default',
      credentials: 'same-origin',
      ...options
    };
    
    this.cache = new Map();
    this.subscriptions = new Map();
    this.middleware = [];
    this.errorHandlers = [];
  }
  
  // Add middleware
  use(middleware) {
    this.middleware.push(middleware);
    return this;
  }
  
  // Add error handler
  onError(handler) {
    this.errorHandlers.push(handler);
    return this;
  }
  
  // Execute query
  async query(query, variables = {}, options = {}) {
    const operationName = this.extractOperationName(query);
    const cacheKey = this.generateCacheKey(query, variables);
    
    // Check cache first
    if (options.cache !== false && this.cache.has(cacheKey)) {
      const cached = this.cache.get(cacheKey);
      if (!this.isCacheExpired(cached, options.ttl)) {
        return cached.data;
      }
    }
    
    // Execute query
    const result = await this.execute({
      query,
      variables,
      operationName
    }, options);
    
    // Cache successful results
    if (!result.errors && options.cache !== false) {
      this.cache.set(cacheKey, {
        data: result,
        timestamp: Date.now()
      });
    }
    
    return result;
  }
  
  // Execute mutation
  async mutate(mutation, variables = {}, options = {}) {
    const operationName = this.extractOperationName(mutation);
    
    const result = await this.execute({
      query: mutation,
      variables,
      operationName
    }, options);
    
    // Invalidate related cache entries
    if (!result.errors && options.invalidate) {
      this.invalidateCache(options.invalidate);
    }
    
    return result;
  }
  
  // Execute subscription
  subscribe(subscription, variables = {}, options = {}) {
    const operationName = this.extractOperationName(subscription);
    const subscriptionId = this.generateSubscriptionId();
    
    // Create WebSocket connection for subscriptions
    const ws = new WebSocket(
      this.endpoint.replace('http', 'ws') + '/graphql',
      'graphql-ws'
    );
    
    const subscriptionData = {
      id: subscriptionId,
      ws,
      options,
      handlers: new Set()
    };
    
    ws.onopen = () => {
      ws.send(JSON.stringify({
        type: 'connection_init'
      }));
    };
    
    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      this.handleSubscriptionMessage(subscriptionId, message);
    };
    
    ws.onopen = () => {
      ws.send(JSON.stringify({
        id: subscriptionId,
        type: 'start',
        payload: {
          query: subscription,
          variables,
          operationName
        }
      }));
    };
    
    this.subscriptions.set(subscriptionId, subscriptionData);
    
    return {
      subscribe: (handler) => {
        subscriptionData.handlers.add(handler);
        return () => subscriptionData.handlers.delete(handler);
      },
      unsubscribe: () => {
        ws.send(JSON.stringify({
          id: subscriptionId,
          type: 'stop'
        }));
        ws.close();
        this.subscriptions.delete(subscriptionId);
      }
    };
  }
  
  // Execute GraphQL operation
  async execute(operation, options = {}) {
    try {
      // Apply middleware
      let request = {
        ...operation,
        headers: { ...this.options.headers, ...options.headers }
      };
      
      for (const middleware of this.middleware) {
        request = await middleware(request);
      }
      
      // Make HTTP request
      const response = await fetch(this.endpoint, {
        method: 'POST',
        headers: request.headers,
        body: JSON.stringify({
          query: request.query,
          variables: request.variables,
          operationName: request.operationName
        }),
        ...this.options,
        ...options
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      
      // Handle GraphQL errors
      if (result.errors) {
        for (const handler of this.errorHandlers) {
          await handler(result.errors, operation);
        }
      }
      
      return result;
    } catch (error) {
      // Handle network errors
      for (const handler of this.errorHandlers) {
        await handler([{ message: error.message }], operation);
      }
      throw error;
    }
  }
  
  handleSubscriptionMessage(subscriptionId, message) {
    const subscription = this.subscriptions.get(subscriptionId);
    if (!subscription) return;
    
    switch (message.type) {
      case 'data':
        subscription.handlers.forEach(handler => {
          handler(message.payload);
        });
        break;
      case 'error':
        subscription.handlers.forEach(handler => {
          handler(null, message.payload);
        });
        break;
      case 'complete':
        subscription.ws.close();
        this.subscriptions.delete(subscriptionId);
        break;
    }
  }
  
  // Batch multiple queries
  async batch(queries) {
    const batchRequest = queries.map(({ query, variables, operationName }) => ({
      query,
      variables,
      operationName: operationName || this.extractOperationName(query)
    }));
    
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: this.options.headers,
      body: JSON.stringify(batchRequest),
      ...this.options
    });
    
    return response.json();
  }
  
  // Cache management
  invalidateCache(pattern) {
    if (typeof pattern === 'string') {
      // Remove specific cache entry
      this.cache.delete(pattern);
    } else if (pattern instanceof RegExp) {
      // Remove entries matching pattern
      for (const key of this.cache.keys()) {
        if (pattern.test(key)) {
          this.cache.delete(key);
        }
      }
    } else if (Array.isArray(pattern)) {
      // Remove specific entries
      pattern.forEach(key => this.cache.delete(key));
    }
  }
  
  clearCache() {
    this.cache.clear();
  }
  
  // Utility methods
  extractOperationName(query) {
    const match = query.match(/(?:query|mutation|subscription)\s+(\w+)/);
    return match ? match[1] : null;
  }
  
  generateCacheKey(query, variables) {
    return `${query}:${JSON.stringify(variables)}`;
  }
  
  generateSubscriptionId() {
    return `sub_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  isCacheExpired(cached, ttl = 300000) {
    return Date.now() - cached.timestamp > ttl;
  }
  
  // Get cache statistics
  getCacheStats() {
    return {
      size: this.cache.size,
      keys: Array.from(this.cache.keys())
    };
  }
}

// ============= REACT HOOKS =============
// hooks/useGraphQL.js
import { useState, useEffect, useCallback } from 'react';
import { GraphQLClient } from '@/lib/graphql/client';

const client = new GraphQLClient(process.env.NEXT_PUBLIC_GRAPHQL_ENDPOINT);

export const useQuery = (query, variables = {}, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  const executeQuery = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const result = await client.query(query, variables, options);
      
      if (result.errors) {
        setError(result.errors);
      } else {
        setData(result.data);
      }
    } catch (err) {
      setError([{ message: err.message }]);
    } finally {
      setLoading(false);
    }
  }, [query, JSON.stringify(variables)]);
  
  useEffect(() => {
    executeQuery();
  }, [executeQuery]);
  
  const refetch = useCallback(() => {
    return executeQuery();
  }, [executeQuery]);
  
  return {
    data,
    loading,
    error,
    refetch
  };
};

export const useMutation = (mutation) => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const execute = useCallback(async (variables = {}, options = {}) => {
    try {
      setLoading(true);
      setError(null);
      
      const result = await client.mutate(mutation, variables, options);
      
      if (result.errors) {
        setError(result.errors);
        return { data: null, errors: result.errors };
      }
      
      return { data: result.data, errors: null };
    } catch (err) {
      const errors = [{ message: err.message }];
      setError(errors);
      return { data: null, errors };
    } finally {
      setLoading(false);
    }
  }, [mutation]);
  
  return [execute, { loading, error }];
};

export const useSubscription = (subscription, variables = {}, options = {}) => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [connected, setConnected] = useState(false);
  
  useEffect(() => {
    const sub = client.subscribe(subscription, variables, options);
    
    const unsubscribe = sub.subscribe((result, error) => {
      if (error) {
        setError(error);
        setConnected(false);
      } else {
        setData(result.data);
        setError(null);
        setConnected(true);
      }
    });
    
    return () => {
      unsubscribe();
      sub.unsubscribe();
    };
  }, [subscription, JSON.stringify(variables)]);
  
  return {
    data,
    error,
    connected
  };
};
```

## 🎯 แบบฝึกหัด

### **⚡ แบบฝึกหัดที่ 1: Protocol Optimization**
สร้าง advanced protocol optimization:

```javascript
// TODO: สร้าง protocol optimization

// 1. Adaptive protocol selection
class ProtocolSelector {
  // Network condition detection
  // Protocol switching
  // Performance optimization
}

// 2. Request prioritization
class RequestPrioritizer {
  // Priority queuing
  // Bandwidth allocation
  // Latency optimization
}

// 3. Connection optimization
class ConnectionOptimizer {
  // Keep-alive management
  // Connection reuse
  // Pool optimization
}
```

### **🔗 แบบฝึกหัดที่ 2: P2P Features**
สร้าง sophisticated P2P system:

```javascript
// TODO: สร้าง P2P system

// 1. Distributed data storage
class DistributedStorage {
  // Data sharding
  // Replication
  // Consistency management
}

// 2. Mesh networking
class MeshNetwork {
  // Route optimization
  // Network resilience
  // Load balancing
}

// 3. Consensus mechanism
class ConsensusEngine {
  // Voting algorithms
  // Conflict resolution
  // State synchronization
}
```

### **📊 แบบฝึกหัดที่ 3: GraphQL Advanced**
สร้าง comprehensive GraphQL system:

```javascript
// TODO: สร้าง GraphQL system

// 1. Schema stitching
class SchemaStitcher {
  // Multiple endpoints
  // Type merging
  // Resolver composition
}

// 2. Real-time subscriptions
class RealtimeSubscriptions {
  // Live updates
  // Conflict resolution
  // Presence tracking
}

// 3. Performance optimization
class GraphQLOptimizer {
  // Query analysis
  // Caching strategies
  // Batch optimization
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 19: Web APIs](./19-web-apis.md)

**บทถัดไป**: [บทที่ 21: Design Patterns](./21-design-patterns.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [HTTP/2 Specification](https://tools.ietf.org/html/rfc7540)
- [WebRTC API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [GraphQL Specification](https://spec.graphql.org/)
- [Network Performance Best Practices](https://web.dev/network/)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **HTTP/2 และ Advanced Protocols** - multiplexing, server push, และ connection optimization
- **WebRTC และ Peer-to-Peer** - real-time communication, P2P networks, และ data channels
- **GraphQL Integration** - client implementation, subscriptions, และ caching strategies
- **Performance Optimization** - connection pooling, request prioritization, และ protocol selection
- **Next.js Integration** - network layers, GraphQL hooks, และ real-time features

Advanced networking เป็นพื้นฐานสำคัญสำหรับ high-performance applications! 🚀
