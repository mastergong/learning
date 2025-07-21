# บทที่ 25: Advanced Patterns

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ sophisticated programming patterns และ architectural techniques
- ทำความเข้าใจ event-driven architecture และ reactive programming
- เรียนรู้ functional programming patterns และ monads
- สร้าง enterprise-level applications ด้วย advanced design patterns

## 🔄 Event-Driven Architecture

### **📡 Event System Implementation**

```javascript
// ============= ADVANCED EVENT SYSTEM =============
// lib/events/event-bus.js

export class EventBus {
  constructor(options = {}) {
    this.options = {
      maxListeners: 100,
      enableWildcards: true,
      enableNamespaces: true,
      errorHandler: console.error,
      ...options
    };
    
    this.listeners = new Map();
    this.onceListeners = new Map();
    this.wildcardListeners = new Map();
    this.namespaceListeners = new Map();
    this.middleware = [];
    this.eventHistory = [];
    this.stats = {
      emitted: 0,
      handled: 0,
      errors: 0
    };
  }
  
  // Middleware for event processing
  use(middleware) {
    this.middleware.push(middleware);
    return this;
  }
  
  // Subscribe to events
  on(event, listener, options = {}) {
    this.validateListener(listener);
    
    const eventInfo = {
      listener,
      priority: options.priority || 0,
      once: false,
      context: options.context,
      created: Date.now()
    };
    
    if (this.options.enableNamespaces && event.includes(':')) {
      this.addNamespaceListener(event, eventInfo);
    } else if (this.options.enableWildcards && event.includes('*')) {
      this.addWildcardListener(event, eventInfo);
    } else {
      this.addListener(event, eventInfo);
    }
    
    return this.createUnsubscribe(event, listener);
  }
  
  // Subscribe once
  once(event, listener, options = {}) {
    return this.on(event, (...args) => {
      const result = listener.apply(options.context, args);
      this.off(event, listener);
      return result;
    }, options);
  }
  
  // Unsubscribe
  off(event, listener = null) {
    if (listener === null) {
      // Remove all listeners for event
      this.listeners.delete(event);
      this.wildcardListeners.delete(event);
      this.namespaceListeners.delete(event);
    } else {
      // Remove specific listener
      this.removeListener(event, listener);
    }
    return this;
  }
  
  // Emit events
  async emit(event, ...args) {
    this.stats.emitted++;
    
    const eventData = {
      type: event,
      data: args,
      timestamp: Date.now(),
      id: this.generateEventId()
    };
    
    // Add to history
    this.addToHistory(eventData);
    
    try {
      // Apply middleware
      for (const middleware of this.middleware) {
        const result = await middleware(eventData);
        if (result === false) {
          return false; // Stop propagation
        }
      }
      
      // Get all matching listeners
      const listeners = this.getMatchingListeners(event);
      
      // Sort by priority
      listeners.sort((a, b) => b.priority - a.priority);
      
      // Execute listeners
      const results = await Promise.allSettled(
        listeners.map(listenerInfo => 
          this.executeListener(listenerInfo, eventData)
        )
      );
      
      // Count successful executions
      const successful = results.filter(r => r.status === 'fulfilled').length;
      this.stats.handled += successful;
      
      // Handle errors
      const errors = results
        .filter(r => r.status === 'rejected')
        .map(r => r.reason);
      
      if (errors.length > 0) {
        this.stats.errors += errors.length;
        errors.forEach(error => this.options.errorHandler(error));
      }
      
      return successful > 0;
    } catch (error) {
      this.stats.errors++;
      this.options.errorHandler(error);
      return false;
    }
  }
  
  // Emit with delay
  delay(event, delay, ...args) {
    return new Promise(resolve => {
      setTimeout(() => {
        this.emit(event, ...args).then(resolve);
      }, delay);
    });
  }
  
  // Emit periodically
  interval(event, interval, ...args) {
    const timer = setInterval(() => {
      this.emit(event, ...args);
    }, interval);
    
    return {
      stop: () => clearInterval(timer),
      timer
    };
  }
  
  addListener(event, eventInfo) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    
    const listeners = this.listeners.get(event);
    if (listeners.length >= this.options.maxListeners) {
      throw new Error(`Max listeners (${this.options.maxListeners}) exceeded for event: ${event}`);
    }
    
    listeners.push(eventInfo);
  }
  
  addWildcardListener(pattern, eventInfo) {
    if (!this.wildcardListeners.has(pattern)) {
      this.wildcardListeners.set(pattern, []);
    }
    this.wildcardListeners.get(pattern).push(eventInfo);
  }
  
  addNamespaceListener(event, eventInfo) {
    const namespace = event.split(':')[0];
    if (!this.namespaceListeners.has(namespace)) {
      this.namespaceListeners.set(namespace, []);
    }
    this.namespaceListeners.get(namespace).push({
      ...eventInfo,
      fullEvent: event
    });
  }
  
  getMatchingListeners(event) {
    const listeners = [];
    
    // Exact match listeners
    if (this.listeners.has(event)) {
      listeners.push(...this.listeners.get(event));
    }
    
    // Wildcard listeners
    for (const [pattern, patternListeners] of this.wildcardListeners) {
      if (this.matchesWildcard(event, pattern)) {
        listeners.push(...patternListeners);
      }
    }
    
    // Namespace listeners
    if (event.includes(':')) {
      const namespace = event.split(':')[0];
      if (this.namespaceListeners.has(namespace)) {
        const namespaceListeners = this.namespaceListeners.get(namespace)
          .filter(l => !l.fullEvent || l.fullEvent === event);
        listeners.push(...namespaceListeners);
      }
    }
    
    return listeners;
  }
  
  matchesWildcard(event, pattern) {
    const regex = new RegExp(
      '^' + pattern.replace(/\*/g, '.*').replace(/\?/g, '.') + '$'
    );
    return regex.test(event);
  }
  
  async executeListener(listenerInfo, eventData) {
    const { listener, context } = listenerInfo;
    
    try {
      const result = await listener.call(context, ...eventData.data);
      return result;
    } catch (error) {
      throw new Error(`Listener error for event ${eventData.type}: ${error.message}`);
    }
  }
  
  removeListener(event, listener) {
    const removeFromArray = (array) => {
      const index = array.findIndex(info => info.listener === listener);
      if (index !== -1) {
        array.splice(index, 1);
        return true;
      }
      return false;
    };
    
    // Remove from regular listeners
    if (this.listeners.has(event)) {
      removeFromArray(this.listeners.get(event));
    }
    
    // Remove from wildcard listeners
    if (this.wildcardListeners.has(event)) {
      removeFromArray(this.wildcardListeners.get(event));
    }
    
    // Remove from namespace listeners
    const namespace = event.split(':')[0];
    if (this.namespaceListeners.has(namespace)) {
      removeFromArray(this.namespaceListeners.get(namespace));
    }
  }
  
  createUnsubscribe(event, listener) {
    return () => this.off(event, listener);
  }
  
  validateListener(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Listener must be a function');
    }
  }
  
  generateEventId() {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  addToHistory(eventData) {
    this.eventHistory.push(eventData);
    
    // Limit history size
    if (this.eventHistory.length > 1000) {
      this.eventHistory = this.eventHistory.slice(-500);
    }
  }
  
  // Utility methods
  getStats() {
    return { ...this.stats };
  }
  
  getHistory(limit = 100) {
    return this.eventHistory.slice(-limit);
  }
  
  getListenerCount(event = null) {
    if (event) {
      return this.getMatchingListeners(event).length;
    }
    
    let total = 0;
    for (const listeners of this.listeners.values()) {
      total += listeners.length;
    }
    return total;
  }
  
  clear() {
    this.listeners.clear();
    this.wildcardListeners.clear();
    this.namespaceListeners.clear();
    this.eventHistory = [];
    this.stats = { emitted: 0, handled: 0, errors: 0 };
  }
}

// ============= EVENT SOURCING =============
// lib/events/event-store.js

export class EventStore {
  constructor(options = {}) {
    this.options = {
      snapshotInterval: 100,
      maxEvents: 10000,
      enableSnapshots: true,
      storage: 'memory', // memory, file, database
      ...options
    };
    
    this.events = [];
    this.snapshots = new Map();
    this.aggregates = new Map();
    this.eventHandlers = new Map();
    this.projections = new Map();
  }
  
  // Append event to store
  async appendEvent(streamId, eventType, data, metadata = {}) {
    const event = {
      id: this.generateEventId(),
      streamId,
      eventType,
      data,
      metadata: {
        ...metadata,
        timestamp: Date.now(),
        version: this.getStreamVersion(streamId) + 1
      }
    };
    
    // Validate event
    this.validateEvent(event);
    
    // Store event
    await this.storeEvent(event);
    
    // Update projections
    await this.updateProjections(event);
    
    // Process event handlers
    await this.processEventHandlers(event);
    
    // Create snapshot if needed
    if (this.shouldCreateSnapshot(streamId)) {
      await this.createSnapshot(streamId);
    }
    
    return event;
  }
  
  // Get events for stream
  async getEvents(streamId, fromVersion = 0) {
    const streamEvents = this.events.filter(event => 
      event.streamId === streamId && 
      event.metadata.version > fromVersion
    );
    
    return streamEvents.sort((a, b) => a.metadata.version - b.metadata.version);
  }
  
  // Get aggregate from events
  async getAggregate(streamId, AggregateClass) {
    let aggregate = new AggregateClass();
    
    // Try to load from snapshot first
    if (this.options.enableSnapshots) {
      const snapshot = this.snapshots.get(streamId);
      if (snapshot) {
        aggregate = this.deserializeAggregate(snapshot.data, AggregateClass);
        const eventsAfterSnapshot = await this.getEvents(streamId, snapshot.version);
        aggregate = this.replayEvents(aggregate, eventsAfterSnapshot);
        return aggregate;
      }
    }
    
    // Replay all events
    const events = await this.getEvents(streamId);
    return this.replayEvents(aggregate, events);
  }
  
  // Replay events on aggregate
  replayEvents(aggregate, events) {
    for (const event of events) {
      const handlerMethod = `on${event.eventType}`;
      if (typeof aggregate[handlerMethod] === 'function') {
        aggregate[handlerMethod](event.data);
      }
    }
    
    return aggregate;
  }
  
  // Register event handler
  onEvent(eventType, handler) {
    if (!this.eventHandlers.has(eventType)) {
      this.eventHandlers.set(eventType, []);
    }
    this.eventHandlers.get(eventType).push(handler);
  }
  
  // Register projection
  registerProjection(name, projectionHandler) {
    this.projections.set(name, {
      handler: projectionHandler,
      state: {}
    });
  }
  
  // Get projection state
  getProjection(name) {
    const projection = this.projections.get(name);
    return projection ? projection.state : null;
  }
  
  async storeEvent(event) {
    switch (this.options.storage) {
      case 'memory':
        this.events.push(event);
        break;
      case 'file':
        await this.storeEventToFile(event);
        break;
      case 'database':
        await this.storeEventToDatabase(event);
        break;
    }
  }
  
  async storeEventToFile(event) {
    const fs = await import('fs/promises');
    const eventLine = JSON.stringify(event) + '\n';
    await fs.appendFile('events.jsonl', eventLine);
  }
  
  async storeEventToDatabase(event) {
    // Implementation depends on your database
    // Example with Prisma:
    /*
    await prisma.event.create({
      data: {
        id: event.id,
        streamId: event.streamId,
        eventType: event.eventType,
        data: JSON.stringify(event.data),
        metadata: JSON.stringify(event.metadata)
      }
    });
    */
  }
  
  async updateProjections(event) {
    for (const [name, projection] of this.projections) {
      try {
        projection.state = await projection.handler(projection.state, event);
      } catch (error) {
        console.error(`Projection ${name} failed:`, error);
      }
    }
  }
  
  async processEventHandlers(event) {
    const handlers = this.eventHandlers.get(event.eventType) || [];
    
    await Promise.allSettled(
      handlers.map(handler => handler(event))
    );
  }
  
  getStreamVersion(streamId) {
    const streamEvents = this.events.filter(e => e.streamId === streamId);
    return streamEvents.length;
  }
  
  shouldCreateSnapshot(streamId) {
    if (!this.options.enableSnapshots) return false;
    
    const version = this.getStreamVersion(streamId);
    const lastSnapshot = this.snapshots.get(streamId);
    
    if (!lastSnapshot) {
      return version >= this.options.snapshotInterval;
    }
    
    return version - lastSnapshot.version >= this.options.snapshotInterval;
  }
  
  async createSnapshot(streamId) {
    // This would typically involve reconstructing the aggregate
    // and storing its current state
    const aggregate = await this.getAggregate(streamId, Object);
    
    this.snapshots.set(streamId, {
      streamId,
      version: this.getStreamVersion(streamId),
      data: this.serializeAggregate(aggregate),
      timestamp: Date.now()
    });
  }
  
  serializeAggregate(aggregate) {
    return JSON.stringify(aggregate);
  }
  
  deserializeAggregate(data, AggregateClass) {
    const parsedData = JSON.parse(data);
    const aggregate = new AggregateClass();
    Object.assign(aggregate, parsedData);
    return aggregate;
  }
  
  validateEvent(event) {
    if (!event.streamId) throw new Error('StreamId is required');
    if (!event.eventType) throw new Error('EventType is required');
    if (!event.data) throw new Error('Data is required');
  }
  
  generateEventId() {
    return `evt_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

## ⚡ Reactive Programming

### **📊 Observable Patterns**

```javascript
// ============= REACTIVE PROGRAMMING =============
// lib/reactive/observable.js

export class Observable {
  constructor(subscribe) {
    this.subscribe = subscribe;
  }
  
  // Static creators
  static of(...values) {
    return new Observable(observer => {
      values.forEach(value => observer.next(value));
      observer.complete();
    });
  }
  
  static from(iterable) {
    return new Observable(observer => {
      try {
        for (const value of iterable) {
          observer.next(value);
        }
        observer.complete();
      } catch (error) {
        observer.error(error);
      }
    });
  }
  
  static fromEvent(target, eventName) {
    return new Observable(observer => {
      const handler = event => observer.next(event);
      target.addEventListener(eventName, handler);
      
      return () => target.removeEventListener(eventName, handler);
    });
  }
  
  static fromPromise(promise) {
    return new Observable(observer => {
      promise
        .then(value => {
          observer.next(value);
          observer.complete();
        })
        .catch(error => observer.error(error));
    });
  }
  
  static interval(ms) {
    return new Observable(observer => {
      let count = 0;
      const timer = setInterval(() => {
        observer.next(count++);
      }, ms);
      
      return () => clearInterval(timer);
    });
  }
  
  static timer(delay, period = null) {
    return new Observable(observer => {
      let count = 0;
      
      const timeout = setTimeout(() => {
        observer.next(count++);
        
        if (period !== null) {
          const interval = setInterval(() => {
            observer.next(count++);
          }, period);
          
          return () => clearInterval(interval);
        } else {
          observer.complete();
        }
      }, delay);
      
      return () => clearTimeout(timeout);
    });
  }
  
  static merge(...observables) {
    return new Observable(observer => {
      const subscriptions = observables.map(obs =>
        obs.subscribe({
          next: value => observer.next(value),
          error: error => observer.error(error),
          complete: () => {
            // Complete when all observables complete
            if (subscriptions.every(sub => sub.closed)) {
              observer.complete();
            }
          }
        })
      );
      
      return () => subscriptions.forEach(sub => sub.unsubscribe());
    });
  }
  
  static combineLatest(...observables) {
    return new Observable(observer => {
      const values = new Array(observables.length);
      const hasValue = new Array(observables.length).fill(false);
      let completed = 0;
      
      const subscriptions = observables.map((obs, index) =>
        obs.subscribe({
          next: value => {
            values[index] = value;
            hasValue[index] = true;
            
            if (hasValue.every(Boolean)) {
              observer.next([...values]);
            }
          },
          error: error => observer.error(error),
          complete: () => {
            completed++;
            if (completed === observables.length) {
              observer.complete();
            }
          }
        })
      );
      
      return () => subscriptions.forEach(sub => sub.unsubscribe());
    });
  }
  
  // Transformation operators
  map(transform) {
    return new Observable(observer => {
      return this.subscribe({
        next: value => {
          try {
            observer.next(transform(value));
          } catch (error) {
            observer.error(error);
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
    });
  }
  
  filter(predicate) {
    return new Observable(observer => {
      return this.subscribe({
        next: value => {
          try {
            if (predicate(value)) {
              observer.next(value);
            }
          } catch (error) {
            observer.error(error);
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
    });
  }
  
  take(count) {
    return new Observable(observer => {
      let taken = 0;
      
      const subscription = this.subscribe({
        next: value => {
          if (taken < count) {
            observer.next(value);
            taken++;
            if (taken === count) {
              observer.complete();
              subscription.unsubscribe();
            }
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
      
      return subscription;
    });
  }
  
  skip(count) {
    return new Observable(observer => {
      let skipped = 0;
      
      return this.subscribe({
        next: value => {
          if (skipped >= count) {
            observer.next(value);
          } else {
            skipped++;
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
    });
  }
  
  debounce(ms) {
    return new Observable(observer => {
      let timeout;
      
      return this.subscribe({
        next: value => {
          clearTimeout(timeout);
          timeout = setTimeout(() => {
            observer.next(value);
          }, ms);
        },
        error: error => observer.error(error),
        complete: () => {
          clearTimeout(timeout);
          observer.complete();
        }
      });
    });
  }
  
  throttle(ms) {
    return new Observable(observer => {
      let lastEmit = 0;
      
      return this.subscribe({
        next: value => {
          const now = Date.now();
          if (now - lastEmit >= ms) {
            observer.next(value);
            lastEmit = now;
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
    });
  }
  
  distinctUntilChanged(compare = (a, b) => a === b) {
    return new Observable(observer => {
      let hasLast = false;
      let last;
      
      return this.subscribe({
        next: value => {
          if (!hasLast || !compare(last, value)) {
            observer.next(value);
            last = value;
            hasLast = true;
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
    });
  }
  
  switchMap(project) {
    return new Observable(observer => {
      let innerSubscription;
      
      const outerSubscription = this.subscribe({
        next: value => {
          if (innerSubscription) {
            innerSubscription.unsubscribe();
          }
          
          try {
            const innerObservable = project(value);
            innerSubscription = innerObservable.subscribe({
              next: innerValue => observer.next(innerValue),
              error: error => observer.error(error),
              complete: () => {} // Don't complete outer on inner complete
            });
          } catch (error) {
            observer.error(error);
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
      
      return () => {
        outerSubscription.unsubscribe();
        if (innerSubscription) {
          innerSubscription.unsubscribe();
        }
      };
    });
  }
  
  // Utility operators
  tap(sideEffect) {
    return new Observable(observer => {
      return this.subscribe({
        next: value => {
          try {
            sideEffect(value);
            observer.next(value);
          } catch (error) {
            observer.error(error);
          }
        },
        error: error => observer.error(error),
        complete: () => observer.complete()
      });
    });
  }
  
  catch(handler) {
    return new Observable(observer => {
      return this.subscribe({
        next: value => observer.next(value),
        error: error => {
          try {
            const recovery = handler(error);
            if (recovery instanceof Observable) {
              recovery.subscribe(observer);
            } else {
              observer.next(recovery);
              observer.complete();
            }
          } catch (newError) {
            observer.error(newError);
          }
        },
        complete: () => observer.complete()
      });
    });
  }
  
  // Terminal operators
  toPromise() {
    return new Promise((resolve, reject) => {
      let lastValue;
      
      this.subscribe({
        next: value => lastValue = value,
        error: reject,
        complete: () => resolve(lastValue)
      });
    });
  }
  
  toArray() {
    return new Promise((resolve, reject) => {
      const values = [];
      
      this.subscribe({
        next: value => values.push(value),
        error: reject,
        complete: () => resolve(values)
      });
    });
  }
  
  forEach(callback) {
    return new Promise((resolve, reject) => {
      this.subscribe({
        next: callback,
        error: reject,
        complete: resolve
      });
    });
  }
}

// Observer interface
export class Observer {
  constructor(next, error, complete) {
    this.next = next || (() => {});
    this.error = error || (err => { throw err; });
    this.complete = complete || (() => {});
  }
}

// Subscription class
export class Subscription {
  constructor(unsubscribe) {
    this.unsubscribe = unsubscribe || (() => {});
    this.closed = false;
  }
  
  unsubscribe() {
    if (!this.closed) {
      this.unsubscribe();
      this.closed = true;
    }
  }
}

// ============= REACTIVE STATE =============
// lib/reactive/reactive-state.js

export class ReactiveState {
  constructor(initialState = {}) {
    this.state = { ...initialState };
    this.observers = new Set();
    this.middleware = [];
    this.history = [{ ...initialState }];
    this.maxHistorySize = 50;
  }
  
  // Subscribe to state changes
  subscribe(observer) {
    this.observers.add(observer);
    
    return {
      unsubscribe: () => this.observers.delete(observer)
    };
  }
  
  // Get current state
  getState() {
    return { ...this.state };
  }
  
  // Update state
  setState(updates) {
    const prevState = { ...this.state };
    const nextState = typeof updates === 'function' 
      ? updates(prevState) 
      : { ...prevState, ...updates };
    
    // Apply middleware
    const action = { type: 'setState', payload: updates };
    const processedState = this.applyMiddleware(action, prevState, nextState);
    
    if (processedState !== false) {
      this.state = processedState;
      this.addToHistory(this.state);
      this.notifyObservers(prevState, this.state);
    }
  }
  
  // Select specific part of state as observable
  select(selector) {
    return new Observable(observer => {
      let currentValue = selector(this.state);
      observer.next(currentValue);
      
      const subscription = this.subscribe((prevState, nextState) => {
        const newValue = selector(nextState);
        if (newValue !== currentValue) {
          currentValue = newValue;
          observer.next(newValue);
        }
      });
      
      return subscription.unsubscribe;
    });
  }
  
  // Create computed observable
  computed(computeFn) {
    return this.select(computeFn).distinctUntilChanged();
  }
  
  // Add middleware
  use(middleware) {
    this.middleware.push(middleware);
  }
  
  applyMiddleware(action, prevState, nextState) {
    let result = nextState;
    
    for (const middleware of this.middleware) {
      result = middleware(action, prevState, result);
      if (result === false) break;
    }
    
    return result;
  }
  
  notifyObservers(prevState, nextState) {
    this.observers.forEach(observer => {
      try {
        observer(prevState, nextState);
      } catch (error) {
        console.error('Observer error:', error);
      }
    });
  }
  
  addToHistory(state) {
    this.history.push({ ...state });
    if (this.history.length > this.maxHistorySize) {
      this.history = this.history.slice(-this.maxHistorySize);
    }
  }
  
  // Time travel debugging
  getHistory() {
    return [...this.history];
  }
  
  restoreFromHistory(index) {
    if (index >= 0 && index < this.history.length) {
      this.state = { ...this.history[index] };
      this.notifyObservers({}, this.state);
    }
  }
  
  // Reset to initial state
  reset() {
    const initialState = this.history[0] || {};
    this.setState(initialState);
  }
}
```

## 🔄 Functional Programming Patterns

### **🧮 Monads and Functors**

```javascript
// ============= FUNCTIONAL PATTERNS =============
// lib/functional/maybe.js

export class Maybe {
  constructor(value) {
    this.value = value;
  }
  
  static of(value) {
    return new Maybe(value);
  }
  
  static some(value) {
    return value !== null && value !== undefined ? new Some(value) : new None();
  }
  
  static none() {
    return new None();
  }
  
  isSome() {
    return this instanceof Some;
  }
  
  isNone() {
    return this instanceof None;
  }
  
  map(fn) {
    return this.isSome() ? Maybe.some(fn(this.value)) : this;
  }
  
  flatMap(fn) {
    return this.isSome() ? fn(this.value) : this;
  }
  
  filter(predicate) {
    return this.isSome() && predicate(this.value) ? this : Maybe.none();
  }
  
  getOrElse(defaultValue) {
    return this.isSome() ? this.value : defaultValue;
  }
  
  getOrElseGet(supplier) {
    return this.isSome() ? this.value : supplier();
  }
  
  orElse(alternative) {
    return this.isSome() ? this : alternative;
  }
  
  ifPresent(consumer) {
    if (this.isSome()) {
      consumer(this.value);
    }
    return this;
  }
  
  ifPresentOrElse(consumer, runnable) {
    if (this.isSome()) {
      consumer(this.value);
    } else {
      runnable();
    }
    return this;
  }
}

export class Some extends Maybe {
  constructor(value) {
    if (value === null || value === undefined) {
      throw new Error('Some cannot contain null or undefined');
    }
    super(value);
  }
}

export class None extends Maybe {
  constructor() {
    super(null);
  }
  
  map() { return this; }
  flatMap() { return this; }
  filter() { return this; }
}

// ============= EITHER MONAD =============
// lib/functional/either.js

export class Either {
  constructor(value, isRight = true) {
    this.value = value;
    this.isRight = isRight;
  }
  
  static right(value) {
    return new Right(value);
  }
  
  static left(value) {
    return new Left(value);
  }
  
  static of(value) {
    return Either.right(value);
  }
  
  static try(fn) {
    try {
      return Either.right(fn());
    } catch (error) {
      return Either.left(error);
    }
  }
  
  map(fn) {
    return this.isRight ? Either.right(fn(this.value)) : this;
  }
  
  flatMap(fn) {
    return this.isRight ? fn(this.value) : this;
  }
  
  mapLeft(fn) {
    return this.isRight ? this : Either.left(fn(this.value));
  }
  
  fold(leftFn, rightFn) {
    return this.isRight ? rightFn(this.value) : leftFn(this.value);
  }
  
  getOrElse(defaultValue) {
    return this.isRight ? this.value : defaultValue;
  }
  
  orElse(alternative) {
    return this.isRight ? this : alternative;
  }
  
  swap() {
    return this.isRight ? Either.left(this.value) : Either.right(this.value);
  }
  
  toMaybe() {
    return this.isRight ? Maybe.some(this.value) : Maybe.none();
  }
}

export class Right extends Either {
  constructor(value) {
    super(value, true);
  }
}

export class Left extends Either {
  constructor(value) {
    super(value, false);
  }
}

// ============= IO MONAD =============
// lib/functional/io.js

export class IO {
  constructor(effect) {
    this.effect = effect;
  }
  
  static of(value) {
    return new IO(() => value);
  }
  
  static from(effect) {
    return new IO(effect);
  }
  
  map(fn) {
    return new IO(() => fn(this.effect()));
  }
  
  flatMap(fn) {
    return new IO(() => fn(this.effect()).effect());
  }
  
  ap(ioFn) {
    return new IO(() => {
      const fn = ioFn.effect();
      const value = this.effect();
      return fn(value);
    });
  }
  
  run() {
    return this.effect();
  }
  
  // Utility methods
  tap(sideEffect) {
    return new IO(() => {
      const value = this.effect();
      sideEffect(value);
      return value;
    });
  }
  
  catch(handler) {
    return new IO(() => {
      try {
        return this.effect();
      } catch (error) {
        return handler(error);
      }
    });
  }
  
  delay(ms) {
    return new IO(() => {
      return new Promise(resolve => {
        setTimeout(() => resolve(this.effect()), ms);
      });
    });
  }
  
  timeout(ms, timeoutValue = null) {
    return new IO(() => {
      return Promise.race([
        Promise.resolve(this.effect()),
        new Promise(resolve => setTimeout(() => resolve(timeoutValue), ms))
      ]);
    });
  }
}

// ============= TASK MONAD =============
// lib/functional/task.js

export class Task {
  constructor(computation) {
    this.computation = computation;
  }
  
  static of(value) {
    return new Task(resolve => resolve(value));
  }
  
  static rejected(error) {
    return new Task((resolve, reject) => reject(error));
  }
  
  static from(promise) {
    return new Task((resolve, reject) => {
      promise.then(resolve, reject);
    });
  }
  
  map(fn) {
    return new Task((resolve, reject) => {
      this.computation(
        value => resolve(fn(value)),
        reject
      );
    });
  }
  
  flatMap(fn) {
    return new Task((resolve, reject) => {
      this.computation(
        value => fn(value).computation(resolve, reject),
        reject
      );
    });
  }
  
  catch(handler) {
    return new Task((resolve, reject) => {
      this.computation(
        resolve,
        error => {
          try {
            const recovery = handler(error);
            if (recovery instanceof Task) {
              recovery.computation(resolve, reject);
            } else {
              resolve(recovery);
            }
          } catch (newError) {
            reject(newError);
          }
        }
      );
    });
  }
  
  finally(cleanup) {
    return new Task((resolve, reject) => {
      this.computation(
        value => {
          cleanup();
          resolve(value);
        },
        error => {
          cleanup();
          reject(error);
        }
      );
    });
  }
  
  race(other) {
    return new Task((resolve, reject) => {
      let settled = false;
      
      const settler = (fn) => (...args) => {
        if (!settled) {
          settled = true;
          fn(...args);
        }
      };
      
      this.computation(settler(resolve), settler(reject));
      other.computation(settler(resolve), setter(reject));
    });
  }
  
  timeout(ms, timeoutError = new Error('Task timeout')) {
    return this.race(
      new Task((resolve, reject) => {
        setTimeout(() => reject(timeoutError), ms);
      })
    );
  }
  
  run() {
    return new Promise((resolve, reject) => {
      this.computation(resolve, reject);
    });
  }
  
  // Utility for sequential execution
  static sequence(tasks) {
    return tasks.reduce(
      (acc, task) => acc.flatMap(results => 
        task.map(result => [...results, result])
      ),
      Task.of([])
    );
  }
  
  // Utility for parallel execution
  static parallel(tasks) {
    return new Task((resolve, reject) => {
      const results = new Array(tasks.length);
      let completed = 0;
      let hasError = false;
      
      if (tasks.length === 0) {
        resolve([]);
        return;
      }
      
      tasks.forEach((task, index) => {
        task.computation(
          value => {
            if (!hasError) {
              results[index] = value;
              completed++;
              if (completed === tasks.length) {
                resolve(results);
              }
            }
          },
          error => {
            if (!hasError) {
              hasError = true;
              reject(error);
            }
          }
        );
      });
    });
  }
}
```

## 🎯 แบบฝึกหัด

### **📡 แบบฝึกหัดที่ 1: Event-Driven System**
สร้าง comprehensive event system:

```javascript
// TODO: สร้าง event-driven system

// 1. Event sourcing with CQRS
class EventSourcingSystem {
  // Command handling
  // Event storage
  // Read model projections
}

// 2. Saga pattern
class SagaOrchestrator {
  // Long-running processes
  // Compensation actions
  // State management
}

// 3. Event streaming
class EventStream {
  // Real-time processing
  // Backpressure handling
  // Stream transformations
}
```

### **⚡ แบบฝึกหัดที่ 2: Reactive Patterns**
สร้าง reactive programming system:

```javascript
// TODO: สร้าง reactive system

// 1. Custom operators
const customOperators = {
  // Domain-specific operators
  // Performance optimizations
  // Error handling
};

// 2. Reactive UI components
class ReactiveComponent {
  // State binding
  // Automatic updates
  // Memory management
}

// 3. Data flow architecture
class DataFlowArchitecture {
  // Unidirectional flow
  // State synchronization
  // Side effect management
}
```

### **🔄 แบบฝึกหัดที่ 3: Functional Patterns**
สร้าง functional programming utilities:

```javascript
// TODO: สร้าง functional utilities

// 1. Lens and optics
class Lens {
  // Immutable updates
  // Deep property access
  // Composition
}

// 2. Free monads
class Free {
  // DSL creation
  // Interpreter pattern
  // Effect separation
}

// 3. Validation monad
class Validation {
  // Error accumulation
  // Applicative validation
  // Composite validation
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 24: Security](./24-security.md)

**บทถัดไป**: [บทที่ 26: Performance Architecture](./26-performance-architecture.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [Event-Driven Architecture](https://microservices.io/patterns/data/event-driven-architecture.html)
- [Reactive Programming](https://reactivex.io/)
- [Functional Programming in JavaScript](https://mostly-adequate.gitbooks.io/mostly-adequate-guide/)
- [Event Sourcing Pattern](https://martinfowler.com/eaaDev/EventSourcing.html)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Event-Driven Architecture** - event bus, event sourcing, และ CQRS patterns
- **Reactive Programming** - observables, operators, และ reactive state management
- **Functional Programming** - monads, functors, และ functional composition
- **Advanced Patterns** - sophisticated design patterns สำหรับ complex applications
- **Best Practices** - pattern selection และ implementation strategies

Advanced Patterns เป็นเครื่องมือสำคัญสำหรับ enterprise applications! 🚀
