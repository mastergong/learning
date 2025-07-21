# บทที่ 19: Web APIs

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ modern Web APIs และการใช้งาน
- ทำความเข้าใจ Fetch API, WebSockets, และ Service Workers
- เรียนรู้ Browser APIs สำหรับ multimedia และ device access
- ประยุกต์ใช้ใน Next.js สำหรับ advanced web applications

## 🌐 Fetch API และ Network

### **📡 Advanced Fetch Patterns**

```javascript
// ============= ADVANCED FETCH UTILITIES =============
// lib/network/fetch-utils.js

export class AdvancedFetch {
  constructor(baseConfig = {}) {
    this.baseConfig = {
      timeout: 30000,
      retries: 3,
      retryDelay: 1000,
      cache: 'default',
      credentials: 'same-origin',
      ...baseConfig
    };
    
    this.interceptors = {
      request: [],
      response: [],
      error: []
    };
  }
  
  // Add request interceptor
  interceptRequest(interceptor) {
    this.interceptors.request.push(interceptor);
    return this;
  }
  
  // Add response interceptor
  interceptResponse(interceptor) {
    this.interceptors.response.push(interceptor);
    return this;
  }
  
  // Add error interceptor
  interceptError(interceptor) {
    this.interceptors.error.push(interceptor);
    return this;
  }
  
  // Execute fetch with interceptors
  async fetch(url, options = {}) {
    let config = { ...this.baseConfig, ...options };
    
    // Apply request interceptors
    for (const interceptor of this.interceptors.request) {
      config = await interceptor(config);
    }
    
    // Setup abort controller for timeout
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), config.timeout);
    
    config.signal = controller.signal;
    
    try {
      let response;
      let attempt = 0;
      
      while (attempt <= config.retries) {
        try {
          response = await fetch(url, config);
          break;
        } catch (error) {
          attempt++;
          if (attempt > config.retries) {
            throw error;
          }
          
          // Wait before retry
          await new Promise(resolve => 
            setTimeout(resolve, config.retryDelay * Math.pow(2, attempt - 1))
          );
        }
      }
      
      clearTimeout(timeoutId);
      
      // Apply response interceptors
      for (const interceptor of this.interceptors.response) {
        response = await interceptor(response);
      }
      
      return response;
    } catch (error) {
      clearTimeout(timeoutId);
      
      // Apply error interceptors
      for (const interceptor of this.interceptors.error) {
        error = await interceptor(error);
      }
      
      throw error;
    }
  }
  
  // Convenience methods
  async get(url, options = {}) {
    return this.fetch(url, { ...options, method: 'GET' });
  }
  
  async post(url, data, options = {}) {
    return this.fetch(url, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data),
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    });
  }
  
  async put(url, data, options = {}) {
    return this.fetch(url, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data),
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    });
  }
  
  async delete(url, options = {}) {
    return this.fetch(url, { ...options, method: 'DELETE' });
  }
  
  // Upload file with progress
  async upload(url, file, options = {}) {
    const formData = new FormData();
    formData.append('file', file);
    
    return new Promise((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      
      // Track upload progress
      xhr.upload.addEventListener('progress', (event) => {
        if (event.lengthComputable) {
          const progress = (event.loaded / event.total) * 100;
          options.onProgress?.(progress);
        }
      });
      
      xhr.addEventListener('load', () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve(xhr.response);
        } else {
          reject(new Error(`Upload failed: ${xhr.status}`));
        }
      });
      
      xhr.addEventListener('error', () => {
        reject(new Error('Upload failed'));
      });
      
      xhr.open('POST', url);
      
      // Add headers
      Object.entries(options.headers || {}).forEach(([key, value]) => {
        xhr.setRequestHeader(key, value);
      });
      
      xhr.send(formData);
    });
  }
  
  // Download with progress
  async download(url, options = {}) {
    const response = await this.fetch(url, options);
    
    if (!response.body) {
      throw new Error('ReadableStream not supported');
    }
    
    const contentLength = response.headers.get('content-length');
    const total = parseInt(contentLength, 10);
    let loaded = 0;
    
    const reader = response.body.getReader();
    const chunks = [];
    
    while (true) {
      const { done, value } = await reader.read();
      
      if (done) break;
      
      chunks.push(value);
      loaded += value.length;
      
      if (options.onProgress && total) {
        options.onProgress((loaded / total) * 100);
      }
    }
    
    return new Uint8Array(chunks.reduce((acc, chunk) => [...acc, ...chunk], []));
  }
}

// ============= REQUEST BATCHING =============
// lib/network/batch-requests.js

export class RequestBatcher {
  constructor(options = {}) {
    this.options = {
      batchSize: 10,
      delay: 100,
      maxWait: 1000,
      ...options
    };
    
    this.pending = [];
    this.timeoutId = null;
    this.startTime = null;
  }
  
  add(url, options = {}) {
    return new Promise((resolve, reject) => {
      this.pending.push({
        url,
        options,
        resolve,
        reject
      });
      
      this.schedule();
    });
  }
  
  schedule() {
    if (!this.startTime) {
      this.startTime = Date.now();
    }
    
    // Clear existing timeout
    if (this.timeoutId) {
      clearTimeout(this.timeoutId);
    }
    
    // Check if should flush immediately
    const shouldFlushSize = this.pending.length >= this.options.batchSize;
    const shouldFlushTime = Date.now() - this.startTime >= this.options.maxWait;
    
    if (shouldFlushSize || shouldFlushTime) {
      this.flush();
    } else {
      this.timeoutId = setTimeout(() => this.flush(), this.options.delay);
    }
  }
  
  async flush() {
    if (this.pending.length === 0) return;
    
    const batch = this.pending.splice(0);
    this.timeoutId = null;
    this.startTime = null;
    
    // Execute all requests in parallel
    const promises = batch.map(async (request) => {
      try {
        const response = await fetch(request.url, request.options);
        request.resolve(response);
      } catch (error) {
        request.reject(error);
      }
    });
    
    await Promise.allSettled(promises);
  }
}

// ============= CACHING STRATEGIES =============
// lib/network/cache-strategies.js

export class CacheManager {
  constructor() {
    this.memory = new Map();
    this.indexedDB = null;
    this.initIndexedDB();
  }
  
  async initIndexedDB() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('CacheDB', 1);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.indexedDB = request.result;
        resolve();
      };
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        if (!db.objectStoreNames.contains('cache')) {
          const store = db.createObjectStore('cache', { keyPath: 'key' });
          store.createIndex('timestamp', 'timestamp');
        }
      };
    });
  }
  
  // Cache-first strategy
  async cacheFirst(key, fetchFn, ttl = 300000) {
    // Try memory first
    const memoryResult = this.memory.get(key);
    if (memoryResult && Date.now() - memoryResult.timestamp < ttl) {
      return memoryResult.data;
    }
    
    // Try IndexedDB
    const dbResult = await this.getFromIndexedDB(key);
    if (dbResult && Date.now() - dbResult.timestamp < ttl) {
      // Update memory cache
      this.memory.set(key, dbResult);
      return dbResult.data;
    }
    
    // Fetch fresh data
    const data = await fetchFn();
    const cacheEntry = {
      key,
      data,
      timestamp: Date.now()
    };
    
    // Cache in memory and IndexedDB
    this.memory.set(key, cacheEntry);
    await this.saveToIndexedDB(cacheEntry);
    
    return data;
  }
  
  // Network-first strategy
  async networkFirst(key, fetchFn, ttl = 300000) {
    try {
      // Try network first
      const data = await fetchFn();
      const cacheEntry = {
        key,
        data,
        timestamp: Date.now()
      };
      
      // Cache the result
      this.memory.set(key, cacheEntry);
      await this.saveToIndexedDB(cacheEntry);
      
      return data;
    } catch (error) {
      // Fallback to cache
      const cached = await this.get(key);
      if (cached) {
        return cached;
      }
      throw error;
    }
  }
  
  // Stale-while-revalidate strategy
  async staleWhileRevalidate(key, fetchFn, ttl = 300000) {
    const cached = await this.get(key);
    
    // Return cached data immediately if available
    if (cached) {
      // If stale, update in background
      if (Date.now() - cached.timestamp >= ttl) {
        this.updateInBackground(key, fetchFn);
      }
      return cached.data;
    }
    
    // No cache, fetch fresh data
    return this.cacheFirst(key, fetchFn, ttl);
  }
  
  async updateInBackground(key, fetchFn) {
    try {
      const data = await fetchFn();
      const cacheEntry = {
        key,
        data,
        timestamp: Date.now()
      };
      
      this.memory.set(key, cacheEntry);
      await this.saveToIndexedDB(cacheEntry);
    } catch (error) {
      console.error('Background update failed:', error);
    }
  }
  
  async get(key) {
    // Try memory first
    const memoryResult = this.memory.get(key);
    if (memoryResult) {
      return memoryResult;
    }
    
    // Try IndexedDB
    return this.getFromIndexedDB(key);
  }
  
  async getFromIndexedDB(key) {
    if (!this.indexedDB) await this.initIndexedDB();
    
    return new Promise((resolve, reject) => {
      const transaction = this.indexedDB.transaction(['cache'], 'readonly');
      const store = transaction.objectStore('cache');
      const request = store.get(key);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async saveToIndexedDB(entry) {
    if (!this.indexedDB) await this.initIndexedDB();
    
    return new Promise((resolve, reject) => {
      const transaction = this.indexedDB.transaction(['cache'], 'readwrite');
      const store = transaction.objectStore('cache');
      const request = store.put(entry);
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  async clear() {
    this.memory.clear();
    
    if (!this.indexedDB) return;
    
    return new Promise((resolve, reject) => {
      const transaction = this.indexedDB.transaction(['cache'], 'readwrite');
      const store = transaction.objectStore('cache');
      const request = store.clear();
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}

// ============= NEXT.JS INTEGRATION =============
// lib/api/api-client.js
import { AdvancedFetch, CacheManager } from '@/lib/network';

export class APIClient extends AdvancedFetch {
  constructor(baseURL, options = {}) {
    super(options);
    
    this.baseURL = baseURL.replace(/\/$/, '');
    this.cache = new CacheManager();
    
    // Add default interceptors
    this.interceptRequest(this.addAuthHeader.bind(this));
    this.interceptResponse(this.handleAPIErrors.bind(this));
    this.interceptError(this.handleNetworkErrors.bind(this));
  }
  
  async addAuthHeader(config) {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers = {
        ...config.headers,
        Authorization: `Bearer ${token}`
      };
    }
    return config;
  }
  
  async handleAPIErrors(response) {
    if (!response.ok) {
      const error = await response.json();
      throw new APIError(error.message, response.status, error);
    }
    return response;
  }
  
  async handleNetworkErrors(error) {
    if (error.name === 'AbortError') {
      throw new Error('Request timeout');
    }
    throw error;
  }
  
  // Cached API calls
  async getCached(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const cacheKey = `${url}:${JSON.stringify(options)}`;
    
    return this.cache.staleWhileRevalidate(
      cacheKey,
      async () => {
        const response = await this.get(url, options);
        return response.json();
      },
      options.ttl
    );
  }
  
  // Resource methods
  users = {
    list: (params) => this.getCached('/users', { ttl: 60000, ...params }),
    get: (id) => this.getCached(`/users/${id}`, { ttl: 300000 }),
    create: (data) => this.post('/users', data),
    update: (id, data) => this.put(`/users/${id}`, data),
    delete: (id) => this.delete(`/users/${id}`)
  };
  
  posts = {
    list: (params) => this.getCached('/posts', { ttl: 30000, ...params }),
    get: (id) => this.getCached(`/posts/${id}`, { ttl: 300000 }),
    create: (data) => this.post('/posts', data),
    update: (id, data) => this.put(`/posts/${id}`, data),
    delete: (id) => this.delete(`/posts/${id}`)
  };
}

class APIError extends Error {
  constructor(message, status, data) {
    super(message);
    this.name = 'APIError';
    this.status = status;
    this.data = data;
  }
}

// Global API client instance
export const apiClient = new APIClient(process.env.NEXT_PUBLIC_API_URL);
```

## 🔌 WebSockets และ Real-time

### **⚡ WebSocket Management**

```javascript
// ============= WEBSOCKET CLIENT =============
// lib/websocket/client.js

export class WebSocketClient extends EventTarget {
  constructor(url, options = {}) {
    super();
    
    this.url = url;
    this.options = {
      reconnectAttempts: 5,
      reconnectDelay: 1000,
      heartbeatInterval: 30000,
      protocols: [],
      ...options
    };
    
    this.ws = null;
    this.reconnectCount = 0;
    this.heartbeatTimer = null;
    this.messageQueue = [];
    this.subscriptions = new Map();
    this.state = 'disconnected';
    
    this.connect();
  }
  
  connect() {
    try {
      this.state = 'connecting';
      this.ws = new WebSocket(this.url, this.options.protocols);
      
      this.ws.onopen = this.handleOpen.bind(this);
      this.ws.onmessage = this.handleMessage.bind(this);
      this.ws.onclose = this.handleClose.bind(this);
      this.ws.onerror = this.handleError.bind(this);
    } catch (error) {
      this.handleError(error);
    }
  }
  
  handleOpen(event) {
    this.state = 'connected';
    this.reconnectCount = 0;
    
    // Send queued messages
    while (this.messageQueue.length > 0) {
      const message = this.messageQueue.shift();
      this.ws.send(message);
    }
    
    // Start heartbeat
    this.startHeartbeat();
    
    this.dispatchEvent(new CustomEvent('open', { detail: event }));
  }
  
  handleMessage(event) {
    try {
      const data = JSON.parse(event.data);
      
      // Handle heartbeat
      if (data.type === 'ping') {
        this.send({ type: 'pong' });
        return;
      }
      
      // Handle subscription messages
      if (data.channel && this.subscriptions.has(data.channel)) {
        const handlers = this.subscriptions.get(data.channel);
        handlers.forEach(handler => handler(data.payload));
      }
      
      this.dispatchEvent(new CustomEvent('message', { detail: data }));
    } catch (error) {
      this.dispatchEvent(new CustomEvent('error', { detail: error }));
    }
  }
  
  handleClose(event) {
    this.state = 'disconnected';
    this.stopHeartbeat();
    
    this.dispatchEvent(new CustomEvent('close', { detail: event }));
    
    // Attempt reconnection if not intentional
    if (!event.wasClean && this.reconnectCount < this.options.reconnectAttempts) {
      this.scheduleReconnect();
    }
  }
  
  handleError(error) {
    this.dispatchEvent(new CustomEvent('error', { detail: error }));
  }
  
  scheduleReconnect() {
    this.reconnectCount++;
    const delay = this.options.reconnectDelay * Math.pow(2, this.reconnectCount - 1);
    
    setTimeout(() => {
      if (this.state === 'disconnected') {
        this.connect();
      }
    }, delay);
  }
  
  startHeartbeat() {
    if (this.options.heartbeatInterval > 0) {
      this.heartbeatTimer = setInterval(() => {
        if (this.state === 'connected') {
          this.send({ type: 'ping' });
        }
      }, this.options.heartbeatInterval);
    }
  }
  
  stopHeartbeat() {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
      this.heartbeatTimer = null;
    }
  }
  
  send(data) {
    const message = JSON.stringify(data);
    
    if (this.state === 'connected') {
      this.ws.send(message);
    } else {
      this.messageQueue.push(message);
    }
  }
  
  subscribe(channel, handler) {
    if (!this.subscriptions.has(channel)) {
      this.subscriptions.set(channel, new Set());
      
      // Send subscription message
      this.send({
        type: 'subscribe',
        channel
      });
    }
    
    this.subscriptions.get(channel).add(handler);
    
    // Return unsubscribe function
    return () => {
      const handlers = this.subscriptions.get(channel);
      if (handlers) {
        handlers.delete(handler);
        
        if (handlers.size === 0) {
          this.subscriptions.delete(channel);
          this.send({
            type: 'unsubscribe',
            channel
          });
        }
      }
    };
  }
  
  close() {
    this.state = 'disconnected';
    this.stopHeartbeat();
    
    if (this.ws) {
      this.ws.close(1000, 'Client disconnect');
    }
  }
  
  getState() {
    return this.state;
  }
}

// ============= REAL-TIME DATA STORE =============
// lib/realtime/store.js

export class RealtimeStore extends EventTarget {
  constructor(wsClient) {
    super();
    
    this.ws = wsClient;
    this.data = new Map();
    this.watchers = new Map();
    
    this.ws.addEventListener('message', this.handleMessage.bind(this));
  }
  
  handleMessage(event) {
    const { type, payload } = event.detail;
    
    switch (type) {
      case 'data_update':
        this.handleDataUpdate(payload);
        break;
      case 'data_delete':
        this.handleDataDelete(payload);
        break;
      case 'batch_update':
        this.handleBatchUpdate(payload);
        break;
    }
  }
  
  handleDataUpdate(payload) {
    const { key, data, timestamp } = payload;
    
    const oldData = this.data.get(key);
    this.data.set(key, { data, timestamp });
    
    // Notify watchers
    const watchers = this.watchers.get(key);
    if (watchers) {
      watchers.forEach(watcher => watcher(data, oldData?.data));
    }
    
    this.dispatchEvent(new CustomEvent('update', {
      detail: { key, data, oldData: oldData?.data }
    }));
  }
  
  handleDataDelete(payload) {
    const { key } = payload;
    
    const oldData = this.data.get(key);
    this.data.delete(key);
    
    // Notify watchers
    const watchers = this.watchers.get(key);
    if (watchers) {
      watchers.forEach(watcher => watcher(null, oldData?.data));
    }
    
    this.dispatchEvent(new CustomEvent('delete', {
      detail: { key, oldData: oldData?.data }
    }));
  }
  
  handleBatchUpdate(payload) {
    const { updates } = payload;
    
    updates.forEach(update => {
      this.handleDataUpdate(update);
    });
  }
  
  get(key) {
    const entry = this.data.get(key);
    return entry ? entry.data : null;
  }
  
  set(key, data) {
    // Send update to server
    this.ws.send({
      type: 'update_data',
      payload: { key, data }
    });
  }
  
  delete(key) {
    // Send delete to server
    this.ws.send({
      type: 'delete_data',
      payload: { key }
    });
  }
  
  watch(key, callback) {
    if (!this.watchers.has(key)) {
      this.watchers.set(key, new Set());
    }
    
    this.watchers.get(key).add(callback);
    
    // Return unwatch function
    return () => {
      const keyWatchers = this.watchers.get(key);
      if (keyWatchers) {
        keyWatchers.delete(callback);
        if (keyWatchers.size === 0) {
          this.watchers.delete(key);
        }
      }
    };
  }
  
  subscribe(pattern) {
    return this.ws.subscribe(`data:${pattern}`, (data) => {
      this.handleDataUpdate(data);
    });
  }
}

// ============= REACT HOOKS =============
// hooks/useWebSocket.js
import { useEffect, useRef, useState } from 'react';
import { WebSocketClient } from '@/lib/websocket/client';

export const useWebSocket = (url, options = {}) => {
  const [state, setState] = useState('disconnected');
  const [lastMessage, setLastMessage] = useState(null);
  const wsRef = useRef(null);
  
  useEffect(() => {
    wsRef.current = new WebSocketClient(url, options);
    
    const handleOpen = () => setState('connected');
    const handleClose = () => setState('disconnected');
    const handleError = () => setState('error');
    const handleMessage = (event) => setLastMessage(event.detail);
    
    wsRef.current.addEventListener('open', handleOpen);
    wsRef.current.addEventListener('close', handleClose);
    wsRef.current.addEventListener('error', handleError);
    wsRef.current.addEventListener('message', handleMessage);
    
    return () => {
      wsRef.current.close();
    };
  }, [url]);
  
  const send = (data) => {
    if (wsRef.current) {
      wsRef.current.send(data);
    }
  };
  
  const subscribe = (channel, handler) => {
    if (wsRef.current) {
      return wsRef.current.subscribe(channel, handler);
    }
  };
  
  return {
    state,
    lastMessage,
    send,
    subscribe
  };
};

// hooks/useRealtimeData.js
export const useRealtimeData = (key, initialData = null) => {
  const [data, setData] = useState(initialData);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    // This would be connected to your realtime store
    const unwatch = window.realtimeStore?.watch(key, (newData, oldData) => {
      setData(newData);
      setLoading(false);
    });
    
    // Initial data fetch
    const currentData = window.realtimeStore?.get(key);
    if (currentData !== null) {
      setData(currentData);
      setLoading(false);
    }
    
    return unwatch;
  }, [key]);
  
  const updateData = (newData) => {
    window.realtimeStore?.set(key, newData);
  };
  
  const deleteData = () => {
    window.realtimeStore?.delete(key);
  };
  
  return {
    data,
    loading,
    error,
    updateData,
    deleteData
  };
};
```

## 🎥 Media APIs

### **📸 Camera and Microphone**

```javascript
// ============= MEDIA CAPTURE =============
// lib/media/capture.js

export class MediaCapture {
  constructor() {
    this.stream = null;
    this.recorder = null;
    this.chunks = [];
    this.constraints = {
      video: true,
      audio: true
    };
  }
  
  async startCamera(constraints = {}) {
    try {
      this.constraints = { ...this.constraints, ...constraints };
      this.stream = await navigator.mediaDevices.getUserMedia(this.constraints);
      return this.stream;
    } catch (error) {
      throw new Error(`Camera access failed: ${error.message}`);
    }
  }
  
  async startScreenShare(options = {}) {
    try {
      const displayConstraints = {
        video: {
          cursor: 'always',
          ...options.video
        },
        audio: options.audio || false
      };
      
      this.stream = await navigator.mediaDevices.getDisplayMedia(displayConstraints);
      return this.stream;
    } catch (error) {
      throw new Error(`Screen share failed: ${error.message}`);
    }
  }
  
  async getDevices() {
    try {
      const devices = await navigator.mediaDevices.enumerateDevices();
      return {
        videoInputs: devices.filter(device => device.kind === 'videoinput'),
        audioInputs: devices.filter(device => device.kind === 'audioinput'),
        audioOutputs: devices.filter(device => device.kind === 'audiooutput')
      };
    } catch (error) {
      throw new Error(`Device enumeration failed: ${error.message}`);
    }
  }
  
  switchCamera(deviceId) {
    if (!this.stream) {
      throw new Error('No active stream');
    }
    
    return this.startCamera({
      video: { deviceId: { exact: deviceId } },
      audio: this.constraints.audio
    });
  }
  
  toggleMute() {
    if (!this.stream) return false;
    
    const audioTracks = this.stream.getAudioTracks();
    if (audioTracks.length > 0) {
      const track = audioTracks[0];
      track.enabled = !track.enabled;
      return !track.enabled;
    }
    
    return false;
  }
  
  toggleVideo() {
    if (!this.stream) return false;
    
    const videoTracks = this.stream.getVideoTracks();
    if (videoTracks.length > 0) {
      const track = videoTracks[0];
      track.enabled = !track.enabled;
      return !track.enabled;
    }
    
    return false;
  }
  
  startRecording(options = {}) {
    if (!this.stream) {
      throw new Error('No active stream');
    }
    
    this.chunks = [];
    this.recorder = new MediaRecorder(this.stream, {
      mimeType: options.mimeType || 'video/webm',
      videoBitsPerSecond: options.videoBitsPerSecond || 2500000,
      audioBitsPerSecond: options.audioBitsPerSecond || 128000
    });
    
    this.recorder.ondataavailable = (event) => {
      if (event.data.size > 0) {
        this.chunks.push(event.data);
      }
    };
    
    this.recorder.start(options.interval || 1000);
    return this.recorder;
  }
  
  stopRecording() {
    return new Promise((resolve) => {
      if (!this.recorder) {
        resolve(null);
        return;
      }
      
      this.recorder.onstop = () => {
        const blob = new Blob(this.chunks, { 
          type: this.recorder.mimeType 
        });
        resolve(blob);
      };
      
      this.recorder.stop();
    });
  }
  
  capturePhoto(canvas) {
    if (!this.stream) {
      throw new Error('No active stream');
    }
    
    const video = document.createElement('video');
    video.srcObject = this.stream;
    video.play();
    
    return new Promise((resolve) => {
      video.onloadedmetadata = () => {
        const ctx = canvas.getContext('2d');
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        
        ctx.drawImage(video, 0, 0);
        
        canvas.toBlob((blob) => {
          resolve(blob);
        }, 'image/png');
      };
    });
  }
  
  applyFilter(canvas, filterType) {
    const ctx = canvas.getContext('2d');
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    
    switch (filterType) {
      case 'grayscale':
        for (let i = 0; i < data.length; i += 4) {
          const gray = data[i] * 0.299 + data[i + 1] * 0.587 + data[i + 2] * 0.114;
          data[i] = gray;
          data[i + 1] = gray;
          data[i + 2] = gray;
        }
        break;
        
      case 'sepia':
        for (let i = 0; i < data.length; i += 4) {
          const r = data[i];
          const g = data[i + 1];
          const b = data[i + 2];
          
          data[i] = Math.min(255, (r * 0.393) + (g * 0.769) + (b * 0.189));
          data[i + 1] = Math.min(255, (r * 0.349) + (g * 0.686) + (b * 0.168));
          data[i + 2] = Math.min(255, (r * 0.272) + (g * 0.534) + (b * 0.131));
        }
        break;
        
      case 'invert':
        for (let i = 0; i < data.length; i += 4) {
          data[i] = 255 - data[i];
          data[i + 1] = 255 - data[i + 1];
          data[i + 2] = 255 - data[i + 2];
        }
        break;
    }
    
    ctx.putImageData(imageData, 0, 0);
  }
  
  stop() {
    if (this.stream) {
      this.stream.getTracks().forEach(track => track.stop());
      this.stream = null;
    }
    
    if (this.recorder && this.recorder.state === 'recording') {
      this.recorder.stop();
    }
  }
  
  getStats() {
    if (!this.stream) return null;
    
    const videoTracks = this.stream.getVideoTracks();
    const audioTracks = this.stream.getAudioTracks();
    
    return {
      video: videoTracks.length > 0 ? {
        enabled: videoTracks[0].enabled,
        settings: videoTracks[0].getSettings(),
        capabilities: videoTracks[0].getCapabilities()
      } : null,
      audio: audioTracks.length > 0 ? {
        enabled: audioTracks[0].enabled,
        settings: audioTracks[0].getSettings(),
        capabilities: audioTracks[0].getCapabilities()
      } : null
    };
  }
}

// ============= AUDIO PROCESSING =============
// lib/media/audio-processor.js

export class AudioProcessor {
  constructor() {
    this.audioContext = null;
    this.analyser = null;
    this.dataArray = null;
    this.source = null;
  }
  
  async init() {
    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
    this.analyser = this.audioContext.createAnalyser();
    this.analyser.fftSize = 2048;
    
    const bufferLength = this.analyser.frequencyBinCount;
    this.dataArray = new Uint8Array(bufferLength);
  }
  
  connectStream(stream) {
    if (!this.audioContext) {
      throw new Error('AudioProcessor not initialized');
    }
    
    this.source = this.audioContext.createMediaStreamSource(stream);
    this.source.connect(this.analyser);
  }
  
  getFrequencyData() {
    if (!this.analyser) return null;
    
    this.analyser.getByteFrequencyData(this.dataArray);
    return Array.from(this.dataArray);
  }
  
  getTimeDomainData() {
    if (!this.analyser) return null;
    
    this.analyser.getByteTimeDomainData(this.dataArray);
    return Array.from(this.dataArray);
  }
  
  getVolume() {
    const data = this.getTimeDomainData();
    if (!data) return 0;
    
    let sum = 0;
    for (let i = 0; i < data.length; i++) {
      sum += Math.abs(data[i] - 128);
    }
    
    return sum / data.length / 128;
  }
  
  detectSpeech(threshold = 0.01) {
    const volume = this.getVolume();
    return volume > threshold;
  }
  
  createVisualizer(canvas, type = 'bars') {
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;
    
    const draw = () => {
      requestAnimationFrame(draw);
      
      const data = this.getFrequencyData();
      if (!data) return;
      
      ctx.clearRect(0, 0, width, height);
      
      if (type === 'bars') {
        const barWidth = width / data.length * 2.5;
        let x = 0;
        
        for (let i = 0; i < data.length; i++) {
          const barHeight = (data[i] / 255) * height;
          
          const r = barHeight + 25 * (i / data.length);
          const g = 250 * (i / data.length);
          const b = 50;
          
          ctx.fillStyle = `rgb(${r},${g},${b})`;
          ctx.fillRect(x, height - barHeight, barWidth, barHeight);
          
          x += barWidth + 1;
        }
      } else if (type === 'waveform') {
        const timeData = this.getTimeDomainData();
        
        ctx.lineWidth = 2;
        ctx.strokeStyle = 'rgb(0, 200, 0)';
        ctx.beginPath();
        
        const sliceWidth = width / timeData.length;
        let x = 0;
        
        for (let i = 0; i < timeData.length; i++) {
          const v = timeData[i] / 128.0;
          const y = v * height / 2;
          
          if (i === 0) {
            ctx.moveTo(x, y);
          } else {
            ctx.lineTo(x, y);
          }
          
          x += sliceWidth;
        }
        
        ctx.stroke();
      }
    };
    
    draw();
  }
  
  dispose() {
    if (this.source) {
      this.source.disconnect();
    }
    
    if (this.audioContext) {
      this.audioContext.close();
    }
  }
}

// ============= REACT COMPONENTS =============
// components/media/CameraCapture.jsx
import { useRef, useEffect, useState } from 'react';
import { MediaCapture } from '@/lib/media/capture';

const CameraCapture = ({ onCapture, filters = [] }) => {
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const [mediaCapture] = useState(() => new MediaCapture());
  const [isRecording, setIsRecording] = useState(false);
  const [currentFilter, setCurrentFilter] = useState(null);
  const [devices, setDevices] = useState({ videoInputs: [], audioInputs: [] });
  
  useEffect(() => {
    startCamera();
    loadDevices();
    
    return () => {
      mediaCapture.stop();
    };
  }, []);
  
  const startCamera = async () => {
    try {
      const stream = await mediaCapture.startCamera();
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
      }
    } catch (error) {
      console.error('Camera start failed:', error);
    }
  };
  
  const loadDevices = async () => {
    try {
      const deviceList = await mediaCapture.getDevices();
      setDevices(deviceList);
    } catch (error) {
      console.error('Device enumeration failed:', error);
    }
  };
  
  const handleCapture = async () => {
    try {
      const blob = await mediaCapture.capturePhoto(canvasRef.current);
      
      if (currentFilter) {
        mediaCapture.applyFilter(canvasRef.current, currentFilter);
        canvasRef.current.toBlob((filteredBlob) => {
          onCapture?.(filteredBlob);
        });
      } else {
        onCapture?.(blob);
      }
    } catch (error) {
      console.error('Capture failed:', error);
    }
  };
  
  const handleRecord = async () => {
    if (isRecording) {
      const blob = await mediaCapture.stopRecording();
      setIsRecording(false);
      onCapture?.(blob);
    } else {
      mediaCapture.startRecording();
      setIsRecording(true);
    }
  };
  
  const switchCamera = (deviceId) => {
    mediaCapture.switchCamera(deviceId).then((stream) => {
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
      }
    });
  };
  
  return (
    <div className="camera-capture">
      <div className="video-container">
        <video
          ref={videoRef}
          autoPlay
          playsInline
          muted
          className="w-full h-64 bg-black rounded-lg"
        />
        <canvas
          ref={canvasRef}
          className="hidden"
        />
      </div>
      
      <div className="controls mt-4 space-y-2">
        <div className="flex gap-2">
          <button
            onClick={handleCapture}
            className="px-4 py-2 bg-blue-600 text-white rounded-lg"
          >
            Capture Photo
          </button>
          
          <button
            onClick={handleRecord}
            className={`px-4 py-2 rounded-lg ${
              isRecording
                ? 'bg-red-600 text-white'
                : 'bg-gray-600 text-white'
            }`}
          >
            {isRecording ? 'Stop Recording' : 'Start Recording'}
          </button>
        </div>
        
        {filters.length > 0 && (
          <div className="flex gap-2">
            <button
              onClick={() => setCurrentFilter(null)}
              className={`px-3 py-1 rounded ${
                !currentFilter ? 'bg-blue-600 text-white' : 'bg-gray-200'
              }`}
            >
              No Filter
            </button>
            {filters.map((filter) => (
              <button
                key={filter}
                onClick={() => setCurrentFilter(filter)}
                className={`px-3 py-1 rounded capitalize ${
                  currentFilter === filter
                    ? 'bg-blue-600 text-white'
                    : 'bg-gray-200'
                }`}
              >
                {filter}
              </button>
            ))}
          </div>
        )}
        
        {devices.videoInputs.length > 1 && (
          <select
            onChange={(e) => switchCamera(e.target.value)}
            className="w-full p-2 border rounded-lg"
          >
            {devices.videoInputs.map((device) => (
              <option key={device.deviceId} value={device.deviceId}>
                {device.label || `Camera ${device.deviceId}`}
              </option>
            ))}
          </select>
        )}
      </div>
    </div>
  );
};

export default CameraCapture;
```

## 🔧 Service Workers

### **⚙️ Service Worker Implementation**

```javascript
// ============= SERVICE WORKER =============
// public/sw.js

const CACHE_NAME = 'app-cache-v1';
const STATIC_CACHE = 'static-cache-v1';
const DYNAMIC_CACHE = 'dynamic-cache-v1';

const STATIC_ASSETS = [
  '/',
  '/manifest.json',
  '/offline.html',
  '/_next/static/css/',
  '/_next/static/js/'
];

// Install event
self.addEventListener('install', (event) => {
  event.waitUntil(
    Promise.all([
      caches.open(STATIC_CACHE).then(cache => {
        return cache.addAll(STATIC_ASSETS);
      }),
      caches.open(DYNAMIC_CACHE)
    ]).then(() => {
      self.skipWaiting();
    })
  );
});

// Activate event
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (![STATIC_CACHE, DYNAMIC_CACHE].includes(cacheName)) {
            return caches.delete(cacheName);
          }
        })
      );
    }).then(() => {
      self.clients.claim();
    })
  );
});

// Fetch event
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Handle different request types
  if (url.origin === location.origin) {
    event.respondWith(handleSameOrigin(request));
  } else if (url.pathname.startsWith('/api/')) {
    event.respondWith(handleAPIRequest(request));
  } else {
    event.respondWith(handleCrossOrigin(request));
  }
});

// Same origin requests
async function handleSameOrigin(request) {
  try {
    // Try cache first for static assets
    if (request.url.includes('/_next/static/')) {
      const cachedResponse = await caches.match(request);
      if (cachedResponse) {
        return cachedResponse;
      }
    }
    
    // Try network first for pages
    const networkResponse = await fetch(request);
    
    // Cache successful responses
    if (networkResponse.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  } catch (error) {
    // Fallback to cache
    const cachedResponse = await caches.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    // Return offline page for navigation requests
    if (request.mode === 'navigate') {
      return caches.match('/offline.html');
    }
    
    throw error;
  }
}

// API requests
async function handleAPIRequest(request) {
  try {
    const networkResponse = await fetch(request);
    
    // Cache GET requests
    if (request.method === 'GET' && networkResponse.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  } catch (error) {
    // Return cached response for GET requests
    if (request.method === 'GET') {
      const cachedResponse = await caches.match(request);
      if (cachedResponse) {
        return cachedResponse;
      }
    }
    
    throw error;
  }
}

// Cross-origin requests
async function handleCrossOrigin(request) {
  try {
    const networkResponse = await fetch(request);
    
    // Cache external resources
    if (networkResponse.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  } catch (error) {
    const cachedResponse = await caches.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    throw error;
  }
}

// Background sync
self.addEventListener('sync', (event) => {
  if (event.tag === 'background-sync') {
    event.waitUntil(doBackgroundSync());
  }
});

async function doBackgroundSync() {
  try {
    // Get pending requests from IndexedDB
    const pendingRequests = await getPendingRequests();
    
    for (const request of pendingRequests) {
      try {
        await fetch(request.url, request.options);
        await removePendingRequest(request.id);
      } catch (error) {
        console.error('Background sync failed:', error);
      }
    }
  } catch (error) {
    console.error('Background sync error:', error);
  }
}

// Push notifications
self.addEventListener('push', (event) => {
  const options = {
    body: event.data?.text() || 'New notification',
    icon: '/icon-192x192.png',
    badge: '/badge-72x72.png',
    vibrate: [200, 100, 200],
    data: {
      dateOfArrival: Date.now(),
      primaryKey: 1
    },
    actions: [
      {
        action: 'explore',
        title: 'View',
        icon: '/icon-view.png'
      },
      {
        action: 'close',
        title: 'Close',
        icon: '/icon-close.png'
      }
    ]
  };
  
  event.waitUntil(
    self.registration.showNotification('App Notification', options)
  );
});

// Notification click
self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  
  if (event.action === 'explore') {
    event.waitUntil(
      clients.openWindow('/')
    );
  }
});

// ============= SERVICE WORKER MANAGER =============
// lib/service-worker/manager.js

export class ServiceWorkerManager {
  constructor() {
    this.registration = null;
    this.isSupported = 'serviceWorker' in navigator;
  }
  
  async register(swPath = '/sw.js', options = {}) {
    if (!this.isSupported) {
      throw new Error('Service Workers not supported');
    }
    
    try {
      this.registration = await navigator.serviceWorker.register(swPath, {
        scope: '/',
        ...options
      });
      
      console.log('Service Worker registered:', this.registration);
      
      // Handle updates
      this.registration.addEventListener('updatefound', () => {
        const newWorker = this.registration.installing;
        
        newWorker.addEventListener('statechange', () => {
          if (newWorker.state === 'installed') {
            if (navigator.serviceWorker.controller) {
              // New update available
              this.onUpdateAvailable?.(newWorker);
            } else {
              // First install
              this.onFirstInstall?.(newWorker);
            }
          }
        });
      });
      
      return this.registration;
    } catch (error) {
      console.error('Service Worker registration failed:', error);
      throw error;
    }
  }
  
  async unregister() {
    if (this.registration) {
      const result = await this.registration.unregister();
      console.log('Service Worker unregistered:', result);
      return result;
    }
    return false;
  }
  
  async update() {
    if (this.registration) {
      await this.registration.update();
    }
  }
  
  async skipWaiting() {
    if (this.registration?.waiting) {
      this.registration.waiting.postMessage({ type: 'SKIP_WAITING' });
    }
  }
  
  // Cache management
  async clearCache(cacheName) {
    if ('caches' in window) {
      await caches.delete(cacheName);
    }
  }
  
  async getCacheInfo() {
    if (!('caches' in window)) return {};
    
    const cacheNames = await caches.keys();
    const cacheInfo = {};
    
    for (const name of cacheNames) {
      const cache = await caches.open(name);
      const keys = await cache.keys();
      cacheInfo[name] = {
        size: keys.length,
        urls: keys.map(request => request.url)
      };
    }
    
    return cacheInfo;
  }
  
  // Background sync
  async registerBackgroundSync(tag = 'background-sync') {
    if (this.registration?.sync) {
      await this.registration.sync.register(tag);
    }
  }
  
  // Push notifications
  async subscribeToPush(vapidKey) {
    if (!this.registration) {
      throw new Error('Service Worker not registered');
    }
    
    const subscription = await this.registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlBase64ToUint8Array(vapidKey)
    });
    
    return subscription;
  }
  
  async unsubscribeFromPush() {
    if (!this.registration) return false;
    
    const subscription = await this.registration.pushManager.getSubscription();
    if (subscription) {
      return subscription.unsubscribe();
    }
    
    return false;
  }
  
  urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, '+')
      .replace(/_/g, '/');
    
    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);
    
    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }
    
    return outputArray;
  }
  
  // Event handlers
  onUpdateAvailable = null;
  onFirstInstall = null;
  onControllerChange = null;
}

// Global service worker manager
export const swManager = new ServiceWorkerManager();

// Initialize in Next.js
export const initServiceWorker = async () => {
  if (typeof window !== 'undefined') {
    try {
      await swManager.register();
      
      swManager.onUpdateAvailable = (worker) => {
        // Show update notification
        if (confirm('New version available. Update now?')) {
          swManager.skipWaiting();
          window.location.reload();
        }
      };
      
    } catch (error) {
      console.error('Service Worker initialization failed:', error);
    }
  }
};
```

## 🎯 แบบฝึกหัด

### **🌐 แบบฝึกหัดที่ 1: Network Layer**
สร้าง comprehensive network layer:

```javascript
// TODO: สร้าง network layer

// 1. Request/Response pipeline
class NetworkPipeline {
  // Middleware system
  // Request transformation
  // Response caching
}

// 2. Offline support
class OfflineManager {
  // Request queuing
  // Sync strategies
  // Conflict resolution
}

// 3. Performance monitoring
class NetworkMonitor {
  // Request timing
  // Bandwidth detection
  // Error tracking
}
```

### **🔌 แบบฝึกหัดที่ 2: Real-time Features**
สร้าง advanced real-time system:

```javascript
// TODO: สร้าง real-time system

// 1. Multi-channel WebSocket
class ChannelManager {
  // Channel subscriptions
  // Message routing
  // Connection pooling
}

// 2. Collaborative features
class CollaborationEngine {
  // Operational transforms
  // Conflict resolution
  // Presence awareness
}

// 3. Real-time analytics
class RealtimeAnalytics {
  // Live metrics
  // Event streaming
  // Dashboard updates
}
```

### **🎥 แบบฝึกหัดที่ 3: Media Processing**
สร้าง sophisticated media system:

```javascript
// TODO: สร้าง media system

// 1. Video conferencing
class VideoConference {
  // WebRTC integration
  // Screen sharing
  // Recording capabilities
}

// 2. Image processing
class ImageProcessor {
  // Filter effects
  // Compression
  // Format conversion
}

// 3. Audio analysis
class AudioAnalyzer {
  // Voice detection
  // Noise reduction
  // Music recognition
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 18: Metaprogramming](./18-metaprogramming.md)

**บทถัดไป**: [บทที่ 20: Networking](./20-networking.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Fetch API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [WebSocket API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [MediaDevices API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices)
- [Service Worker API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Fetch API และ Network** - advanced fetch patterns, request batching, และ caching strategies
- **WebSockets และ Real-time** - WebSocket management, real-time data stores, และ React hooks
- **Media APIs** - camera/microphone access, audio processing, และ recording capabilities
- **Service Workers** - caching strategies, background sync, และ push notifications
- **Next.js Integration** - API clients, real-time components, และ service worker management

Web APIs เป็นเครื่องมือที่ทรงพลังสำหรับการสร้าง modern web applications! 🚀
