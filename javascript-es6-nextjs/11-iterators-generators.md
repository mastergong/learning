# บทที่ 11: Iterators & Generators

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Iterators และ Iterator Protocol
- ทำความเข้าใจ Generators และ Generator Functions
- เรียนรู้ Advanced Iteration Patterns
- ประยุกต์ใช้ใน Next.js สำหรับ Data Processing

## 🔄 Iterator Protocol

### **🔍 Understanding Iterators**

```javascript
// ============= BASIC ITERATOR IMPLEMENTATION =============
// Custom iterator for range of numbers
function createRangeIterator(start, end, step = 1) {
  let current = start;
  
  return {
    // Iterator protocol requires next() method
    next() {
      if (current < end) {
        const value = current;
        current += step;
        return { value, done: false };
      } else {
        return { done: true };
      }
    },
    
    // Make it iterable (can be used in for...of)
    [Symbol.iterator]() {
      return this;
    }
  };
}

// Usage
const range = createRangeIterator(1, 6);
console.log(range.next()); // { value: 1, done: false }
console.log(range.next()); // { value: 2, done: false }

// Use in for...of loop
for (const num of createRangeIterator(1, 4)) {
  console.log(num); // 1, 2, 3
}

// ============= ITERABLE OBJECTS =============
class NumberSequence {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }
  
  // Implement Symbol.iterator to make object iterable
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    const step = this.step;
    
    return {
      next() {
        if (current < end) {
          const value = current;
          current += step;
          return { value, done: false };
        }
        return { done: true };
      }
    };
  }
  
  // Additional methods
  toArray() {
    return [...this];
  }
  
  map(fn) {
    const result = [];
    for (const value of this) {
      result.push(fn(value));
    }
    return result;
  }
  
  filter(predicate) {
    const result = [];
    for (const value of this) {
      if (predicate(value)) {
        result.push(value);
      }
    }
    return result;
  }
  
  reduce(reducer, initialValue) {
    let accumulator = initialValue;
    let first = true;
    
    for (const value of this) {
      if (first && accumulator === undefined) {
        accumulator = value;
        first = false;
      } else {
        accumulator = reducer(accumulator, value);
      }
    }
    
    return accumulator;
  }
}

// Usage
const sequence = new NumberSequence(1, 10, 2);
console.log(sequence.toArray()); // [1, 3, 5, 7, 9]

const doubled = sequence.map(x => x * 2);
console.log(doubled); // [2, 6, 10, 14, 18]

const evenSum = sequence.filter(x => x % 2 === 0).reduce((a, b) => a + b, 0);

// ============= ADVANCED ITERATOR PATTERNS =============
class ChainableIterator {
  constructor(iterable) {
    this.iterable = iterable;
  }
  
  [Symbol.iterator]() {
    return this.iterable[Symbol.iterator]();
  }
  
  map(fn) {
    return new ChainableIterator(this._mapGenerator(fn));
  }
  
  *_mapGenerator(fn) {
    for (const value of this.iterable) {
      yield fn(value);
    }
  }
  
  filter(predicate) {
    return new ChainableIterator(this._filterGenerator(predicate));
  }
  
  *_filterGenerator(predicate) {
    for (const value of this.iterable) {
      if (predicate(value)) {
        yield value;
      }
    }
  }
  
  take(count) {
    return new ChainableIterator(this._takeGenerator(count));
  }
  
  *_takeGenerator(count) {
    let taken = 0;
    for (const value of this.iterable) {
      if (taken >= count) break;
      yield value;
      taken++;
    }
  }
  
  skip(count) {
    return new ChainableIterator(this._skipGenerator(count));
  }
  
  *_skipGenerator(count) {
    let skipped = 0;
    for (const value of this.iterable) {
      if (skipped < count) {
        skipped++;
        continue;
      }
      yield value;
    }
  }
  
  toArray() {
    return [...this.iterable];
  }
  
  reduce(reducer, initialValue) {
    let accumulator = initialValue;
    let first = true;
    
    for (const value of this.iterable) {
      if (first && accumulator === undefined) {
        accumulator = value;
        first = false;
      } else {
        accumulator = reducer(accumulator, value);
      }
    }
    
    return accumulator;
  }
  
  forEach(fn) {
    for (const value of this.iterable) {
      fn(value);
    }
  }
  
  find(predicate) {
    for (const value of this.iterable) {
      if (predicate(value)) {
        return value;
      }
    }
    return undefined;
  }
  
  some(predicate) {
    for (const value of this.iterable) {
      if (predicate(value)) {
        return true;
      }
    }
    return false;
  }
  
  every(predicate) {
    for (const value of this.iterable) {
      if (!predicate(value)) {
        return false;
      }
    }
    return true;
  }
}

// Usage - chainable operations
const numbers = new ChainableIterator([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

const result = numbers
  .filter(x => x % 2 === 0)  // [2, 4, 6, 8, 10]
  .map(x => x * x)           // [4, 16, 36, 64, 100]
  .take(3)                   // [4, 16, 36]
  .toArray();

console.log(result); // [4, 16, 36]

// ============= LAZY EVALUATION ITERATOR =============
class LazyIterator {
  constructor(source) {
    this.source = source;
    this.operations = [];
  }
  
  map(fn) {
    this.operations.push({ type: 'map', fn });
    return this;
  }
  
  filter(predicate) {
    this.operations.push({ type: 'filter', predicate });
    return this;
  }
  
  take(count) {
    this.operations.push({ type: 'take', count });
    return this;
  }
  
  skip(count) {
    this.operations.push({ type: 'skip', count });
    return this;
  }
  
  [Symbol.iterator]() {
    return this._executeOperations();
  }
  
  *_executeOperations() {
    let iterator = this.source[Symbol.iterator]();
    let taken = 0;
    let skipped = 0;
    
    // Find take and skip operations
    const takeOp = this.operations.find(op => op.type === 'take');
    const skipOp = this.operations.find(op => op.type === 'skip');
    const takeCount = takeOp ? takeOp.count : Infinity;
    const skipCount = skipOp ? skipOp.count : 0;
    
    for (const value of this.source) {
      // Apply skip
      if (skipped < skipCount) {
        skipped++;
        continue;
      }
      
      // Apply take
      if (taken >= takeCount) {
        break;
      }
      
      // Apply transformations
      let currentValue = value;
      let shouldYield = true;
      
      for (const operation of this.operations) {
        switch (operation.type) {
          case 'map':
            currentValue = operation.fn(currentValue);
            break;
          case 'filter':
            if (!operation.predicate(currentValue)) {
              shouldYield = false;
            }
            break;
        }
        
        if (!shouldYield) break;
      }
      
      if (shouldYield) {
        yield currentValue;
        taken++;
      }
    }
  }
  
  toArray() {
    return [...this];
  }
  
  first() {
    for (const value of this) {
      return value;
    }
    return undefined;
  }
  
  count() {
    let count = 0;
    for (const _ of this) {
      count++;
    }
    return count;
  }
}

// Usage - operations are not executed until iteration
const lazy = new LazyIterator([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

const lazyResult = lazy
  .filter(x => {
    console.log(`Filtering ${x}`); // Only logs when actually iterating
    return x > 3;
  })
  .map(x => {
    console.log(`Mapping ${x}`);   // Only logs when actually iterating
    return x * 2;
  })
  .take(3);

console.log('Starting iteration...');
console.log(lazyResult.toArray()); // Now the operations execute
```

## ⚡ Generator Functions

### **🔧 Basic Generators**

```javascript
// ============= GENERATOR FUNCTION BASICS =============
// Simple generator function
function* simpleGenerator() {
  console.log('Generator started');
  yield 1;
  console.log('After first yield');
  yield 2;
  console.log('After second yield');
  yield 3;
  console.log('Generator finished');
}

const gen = simpleGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Generator with parameters and return value
function* parameterizedGenerator(start = 0) {
  let current = start;
  
  while (true) {
    const increment = yield current;
    current += increment || 1;
  }
}

const paramGen = parameterizedGenerator(10);
console.log(paramGen.next());      // { value: 10, done: false }
console.log(paramGen.next(5));     // { value: 15, done: false }
console.log(paramGen.next(2));     // { value: 17, done: false }

// ============= GENERATOR FOR SEQUENCES =============
// Fibonacci sequence generator
function* fibonacciGenerator() {
  let [a, b] = [0, 1];
  
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Take first 10 Fibonacci numbers
function takeFirst(generator, count) {
  const result = [];
  const iterator = generator();
  
  for (let i = 0; i < count; i++) {
    const { value, done } = iterator.next();
    if (done) break;
    result.push(value);
  }
  
  return result;
}

const firstTenFib = takeFirst(fibonacciGenerator, 10);
console.log(firstTenFib); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Prime numbers generator
function* primeGenerator() {
  yield 2; // First prime
  
  const primes = [2];
  let candidate = 3;
  
  while (true) {
    let isPrime = true;
    
    for (const prime of primes) {
      if (prime * prime > candidate) break;
      if (candidate % prime === 0) {
        isPrime = false;
        break;
      }
    }
    
    if (isPrime) {
      primes.push(candidate);
      yield candidate;
    }
    
    candidate += 2; // Skip even numbers
  }
}

const firstTenPrimes = takeFirst(primeGenerator, 10);
console.log(firstTenPrimes); // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

// ============= GENERATOR FOR DATA PROCESSING =============
function* mapGenerator(iterable, mapper) {
  for (const item of iterable) {
    yield mapper(item);
  }
}

function* filterGenerator(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) {
      yield item;
    }
  }
}

function* flatMapGenerator(iterable, mapper) {
  for (const item of iterable) {
    const mapped = mapper(item);
    if (Symbol.iterator in mapped) {
      yield* mapped; // Delegate to another iterable
    } else {
      yield mapped;
    }
  }
}

function* zipGenerator(...iterables) {
  const iterators = iterables.map(iterable => iterable[Symbol.iterator]());
  
  while (true) {
    const results = iterators.map(iterator => iterator.next());
    
    if (results.some(result => result.done)) {
      return;
    }
    
    yield results.map(result => result.value);
  }
}

// Usage examples
const numbers = [1, 2, 3, 4, 5];
const letters = ['a', 'b', 'c'];

// Map example
const doubled = [...mapGenerator(numbers, x => x * 2)];
console.log(doubled); // [2, 4, 6, 8, 10]

// Filter example
const evens = [...filterGenerator(numbers, x => x % 2 === 0)];
console.log(evens); // [2, 4]

// FlatMap example
const repeated = [...flatMapGenerator(letters, x => [x, x])];
console.log(repeated); // ['a', 'a', 'b', 'b', 'c', 'c']

// Zip example
const zipped = [...zipGenerator(numbers, letters)];
console.log(zipped); // [[1, 'a'], [2, 'b'], [3, 'c']]

// ============= ADVANCED GENERATOR PATTERNS =============
// Generator with error handling
function* errorHandlingGenerator() {
  try {
    yield 1;
    yield 2;
    const value = yield 3;
    
    if (value === 'error') {
      throw new Error('Intentional error');
    }
    
    yield value;
  } catch (error) {
    console.log('Caught error:', error.message);
    yield 'error-handled';
  } finally {
    console.log('Generator cleanup');
    yield 'cleanup';
  }
}

const errorGen = errorHandlingGenerator();
console.log(errorGen.next());        // { value: 1, done: false }
console.log(errorGen.next());        // { value: 2, done: false }
console.log(errorGen.next());        // { value: 3, done: false }
console.log(errorGen.next('error')); // Catches error, yields 'error-handled'
console.log(errorGen.next());        // { value: 'cleanup', done: false }

// Async generator for data streaming
async function* asyncDataGenerator(urls) {
  for (const url of urls) {
    try {
      console.log(`Fetching ${url}...`);
      
      // Simulate async operation
      await new Promise(resolve => setTimeout(resolve, 100));
      
      // Simulate data
      const data = { url, data: `Data from ${url}`, timestamp: Date.now() };
      yield data;
    } catch (error) {
      yield { url, error: error.message };
    }
  }
}

// Usage of async generator
async function processAsyncData() {
  const urls = ['api/users', 'api/posts', 'api/comments'];
  
  for await (const result of asyncDataGenerator(urls)) {
    if (result.error) {
      console.error('Error:', result.error);
    } else {
      console.log('Received:', result.data);
    }
  }
}

// processAsyncData();

// ============= GENERATOR-BASED STATE MACHINE =============
function* stateMachine() {
  let state = 'idle';
  let data = null;
  
  while (true) {
    const action = yield { state, data };
    
    switch (state) {
      case 'idle':
        if (action?.type === 'START') {
          state = 'loading';
          data = null;
        }
        break;
        
      case 'loading':
        if (action?.type === 'SUCCESS') {
          state = 'loaded';
          data = action.payload;
        } else if (action?.type === 'ERROR') {
          state = 'error';
          data = action.error;
        }
        break;
        
      case 'loaded':
        if (action?.type === 'RESET') {
          state = 'idle';
          data = null;
        } else if (action?.type === 'UPDATE') {
          data = { ...data, ...action.payload };
        }
        break;
        
      case 'error':
        if (action?.type === 'RETRY') {
          state = 'loading';
          data = null;
        } else if (action?.type === 'RESET') {
          state = 'idle';
          data = null;
        }
        break;
    }
  }
}

// Usage
const machine = stateMachine();
console.log(machine.next());                                    // { state: 'idle', data: null }
console.log(machine.next({ type: 'START' }));                  // { state: 'loading', data: null }
console.log(machine.next({ type: 'SUCCESS', payload: {id: 1} })); // { state: 'loaded', data: {id: 1} }
console.log(machine.next({ type: 'UPDATE', payload: {name: 'John'} })); // { state: 'loaded', data: {id: 1, name: 'John'} }
```

### **🔄 Advanced Generator Patterns**

```javascript
// ============= GENERATOR COMPOSITION =============
function* baseGenerator() {
  yield 1;
  yield 2;
}

function* extendedGenerator() {
  yield* baseGenerator(); // Delegate to another generator
  yield 3;
  yield 4;
}

function* compositeGenerator() {
  yield 'start';
  yield* extendedGenerator();
  yield 'end';
}

console.log([...compositeGenerator()]); // ['start', 1, 2, 3, 4, 'end']

// ============= GENERATOR PIPELINE =============
class GeneratorPipeline {
  constructor(source) {
    this.source = source;
    this.operations = [];
  }
  
  map(fn) {
    this.operations.push({
      type: 'map',
      execute: function*(iterable) {
        for (const item of iterable) {
          yield fn(item);
        }
      }
    });
    return this;
  }
  
  filter(predicate) {
    this.operations.push({
      type: 'filter',
      execute: function*(iterable) {
        for (const item of iterable) {
          if (predicate(item)) {
            yield item;
          }
        }
      }
    });
    return this;
  }
  
  take(count) {
    this.operations.push({
      type: 'take',
      execute: function*(iterable) {
        let taken = 0;
        for (const item of iterable) {
          if (taken >= count) break;
          yield item;
          taken++;
        }
      }
    });
    return this;
  }
  
  skip(count) {
    this.operations.push({
      type: 'skip',
      execute: function*(iterable) {
        let skipped = 0;
        for (const item of iterable) {
          if (skipped < count) {
            skipped++;
            continue;
          }
          yield item;
        }
      }
    });
    return this;
  }
  
  batch(size) {
    this.operations.push({
      type: 'batch',
      execute: function*(iterable) {
        let batch = [];
        for (const item of iterable) {
          batch.push(item);
          if (batch.length === size) {
            yield [...batch];
            batch = [];
          }
        }
        if (batch.length > 0) {
          yield batch;
        }
      }
    });
    return this;
  }
  
  *execute() {
    let current = this.source;
    
    for (const operation of this.operations) {
      current = operation.execute(current);
    }
    
    yield* current;
  }
  
  toArray() {
    return [...this.execute()];
  }
  
  forEach(fn) {
    for (const item of this.execute()) {
      fn(item);
    }
  }
  
  reduce(reducer, initialValue) {
    let accumulator = initialValue;
    let first = true;
    
    for (const item of this.execute()) {
      if (first && accumulator === undefined) {
        accumulator = item;
        first = false;
      } else {
        accumulator = reducer(accumulator, item);
      }
    }
    
    return accumulator;
  }
}

// Usage
const pipeline = new GeneratorPipeline([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

const result = pipeline
  .filter(x => x % 2 === 0)
  .map(x => x * x)
  .take(3)
  .toArray();

console.log(result); // [4, 16, 36]

// Batch processing
const batched = new GeneratorPipeline([1, 2, 3, 4, 5, 6, 7, 8, 9])
  .batch(3)
  .toArray();

console.log(batched); // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// ============= INFINITE SEQUENCE GENERATORS =============
function* infiniteSequence(fn, seed) {
  let current = seed;
  while (true) {
    yield current;
    current = fn(current);
  }
}

function* infiniteCounter(start = 0, step = 1) {
  let current = start;
  while (true) {
    yield current;
    current += step;
  }
}

function* infiniteRandom(min = 0, max = 1) {
  while (true) {
    yield Math.random() * (max - min) + min;
  }
}

// Usage
const doubleSequence = infiniteSequence(x => x * 2, 1);
const firstFiveDoubles = takeFirst(() => doubleSequence, 5);
console.log(firstFiveDoubles); // [1, 2, 4, 8, 16]

const counter = infiniteCounter(100, 5);
const firstFiveCount = takeFirst(() => counter, 5);
console.log(firstFiveCount); // [100, 105, 110, 115, 120]

// ============= GENERATOR-BASED OBSERVERS =============
function* observableGenerator() {
  const observers = [];
  let value = null;
  
  try {
    while (true) {
      const action = yield value;
      
      if (action?.type === 'SUBSCRIBE') {
        observers.push(action.observer);
      } else if (action?.type === 'UNSUBSCRIBE') {
        const index = observers.indexOf(action.observer);
        if (index > -1) {
          observers.splice(index, 1);
        }
      } else if (action?.type === 'EMIT') {
        value = action.value;
        observers.forEach(observer => {
          try {
            observer(value);
          } catch (error) {
            console.error('Observer error:', error);
          }
        });
      }
    }
  } finally {
    console.log('Observable cleaned up');
  }
}

// Usage
const observable = observableGenerator();
observable.next(); // Initialize

const observer1 = (value) => console.log('Observer 1:', value);
const observer2 = (value) => console.log('Observer 2:', value);

observable.next({ type: 'SUBSCRIBE', observer: observer1 });
observable.next({ type: 'SUBSCRIBE', observer: observer2 });

observable.next({ type: 'EMIT', value: 'Hello' });
// Observer 1: Hello
// Observer 2: Hello

observable.next({ type: 'UNSUBSCRIBE', observer: observer1 });
observable.next({ type: 'EMIT', value: 'World' });
// Observer 2: World

// ============= ASYNC GENERATOR FOR REAL-TIME DATA =============
async function* realTimeDataGenerator(source, interval = 1000) {
  while (true) {
    try {
      // Simulate fetching real-time data
      const data = await fetchDataFromSource(source);
      yield {
        timestamp: Date.now(),
        source,
        data,
        status: 'success'
      };
    } catch (error) {
      yield {
        timestamp: Date.now(),
        source,
        error: error.message,
        status: 'error'
      };
    }
    
    // Wait for next interval
    await new Promise(resolve => setTimeout(resolve, interval));
  }
}

async function fetchDataFromSource(source) {
  // Simulate async data fetching
  await new Promise(resolve => setTimeout(resolve, Math.random() * 200));
  
  if (Math.random() < 0.1) {
    throw new Error('Network error');
  }
  
  return {
    value: Math.floor(Math.random() * 100),
    source
  };
}

// Usage
async function monitorRealTimeData() {
  const dataStream = realTimeDataGenerator('sensor-1', 500);
  
  // Monitor for 5 seconds
  const endTime = Date.now() + 5000;
  
  for await (const dataPoint of dataStream) {
    if (Date.now() > endTime) break;
    
    if (dataPoint.status === 'success') {
      console.log(`[${new Date(dataPoint.timestamp).toLocaleTimeString()}] 
                  Data: ${dataPoint.data.value}`);
    } else {
      console.error(`[${new Date(dataPoint.timestamp).toLocaleTimeString()}] 
                    Error: ${dataPoint.error}`);
    }
  }
}

// monitorRealTimeData();
```

## 🚀 Next.js Applications

### **📊 Data Processing with Generators**

```jsx
// ============= PAGINATION WITH GENERATORS =============
// lib/pagination/generator.js
export function* paginationGenerator(fetchFunction, pageSize = 20) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    try {
      const response = await fetchFunction(page, pageSize);
      
      if (response.data.length === 0) {
        hasMore = false;
        return;
      }
      
      yield {
        data: response.data,
        page,
        hasMore: response.hasMore ?? response.data.length === pageSize,
        total: response.total
      };
      
      page++;
      hasMore = response.hasMore ?? response.data.length === pageSize;
    } catch (error) {
      yield {
        error: error.message,
        page,
        hasMore: false
      };
      return;
    }
  }
}

// components/InfiniteList.jsx
import { useState, useEffect, useCallback } from 'react';
import { paginationGenerator } from '../lib/pagination/generator';

const InfiniteList = ({ fetchData, renderItem, pageSize = 20 }) => {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [hasMore, setHasMore] = useState(true);
  const [generator, setGenerator] = useState(null);

  const initializeGenerator = useCallback(() => {
    const gen = paginationGenerator(fetchData, pageSize);
    setGenerator(gen);
    return gen;
  }, [fetchData, pageSize]);

  const loadNextPage = useCallback(async () => {
    if (loading || !hasMore) return;

    setLoading(true);
    setError(null);

    try {
      const currentGenerator = generator || initializeGenerator();
      const { value, done } = await currentGenerator.next();

      if (done) {
        setHasMore(false);
        return;
      }

      if (value.error) {
        setError(value.error);
        setHasMore(false);
        return;
      }

      setItems(prev => [...prev, ...value.data]);
      setHasMore(value.hasMore);
    } catch (err) {
      setError(err.message);
      setHasMore(false);
    } finally {
      setLoading(false);
    }
  }, [generator, initializeGenerator, loading, hasMore]);

  // Load first page
  useEffect(() => {
    loadNextPage();
  }, []);

  // Intersection Observer for infinite scroll
  useEffect(() => {
    const observer = new IntersectionObserver(
      entries => {
        if (entries[0].isIntersecting && hasMore && !loading) {
          loadNextPage();
        }
      },
      { threshold: 1.0 }
    );

    const sentinel = document.getElementById('scroll-sentinel');
    if (sentinel) {
      observer.observe(sentinel);
    }

    return () => observer.disconnect();
  }, [loadNextPage, hasMore, loading]);

  const reset = useCallback(() => {
    setItems([]);
    setError(null);
    setHasMore(true);
    setGenerator(null);
  }, []);

  return (
    <div className="infinite-list">
      {items.map((item, index) => (
        <div key={item.id || index} className="list-item">
          {renderItem(item, index)}
        </div>
      ))}

      {loading && (
        <div className="loading-indicator">
          Loading more items...
        </div>
      )}

      {error && (
        <div className="error-indicator">
          <p>Error: {error}</p>
          <button onClick={() => { setError(null); loadNextPage(); }}>
            Retry
          </button>
        </div>
      )}

      {!hasMore && items.length > 0 && (
        <div className="end-indicator">
          No more items to load
        </div>
      )}

      <div id="scroll-sentinel" style={{ height: '1px' }} />

      <div className="list-controls">
        <button onClick={reset}>Reset</button>
        <span>{items.length} items loaded</span>
      </div>
    </div>
  );
};

export default InfiniteList;

// ============= STREAM PROCESSING COMPONENT =============
// components/DataStreamProcessor.jsx
import { useState, useEffect, useRef } from 'react';

const DataStreamProcessor = ({ 
  dataSource, 
  processors = [], 
  batchSize = 10,
  interval = 1000 
}) => {
  const [processedData, setProcessedData] = useState([]);
  const [stats, setStats] = useState({
    processed: 0,
    errors: 0,
    rate: 0
  });
  const [isRunning, setIsRunning] = useState(false);
  const generatorRef = useRef(null);
  const startTimeRef = useRef(null);

  // Create processing generator
  const createProcessor = useCallback(function*() {
    let batch = [];
    let totalProcessed = 0;
    let totalErrors = 0;

    for (const rawData of dataSource) {
      try {
        // Apply all processors to the data
        let processedItem = rawData;
        for (const processor of processors) {
          processedItem = processor(processedItem);
        }

        batch.push({
          id: rawData.id || totalProcessed,
          original: rawData,
          processed: processedItem,
          timestamp: Date.now()
        });

        totalProcessed++;

        // Yield batch when full
        if (batch.length >= batchSize) {
          yield {
            type: 'batch',
            data: [...batch],
            stats: {
              processed: totalProcessed,
              errors: totalErrors,
              rate: calculateRate(totalProcessed)
            }
          };
          batch = [];
        }
      } catch (error) {
        totalErrors++;
        yield {
          type: 'error',
          error: error.message,
          data: rawData,
          stats: {
            processed: totalProcessed,
            errors: totalErrors,
            rate: calculateRate(totalProcessed)
          }
        };
      }
    }

    // Yield remaining items
    if (batch.length > 0) {
      yield {
        type: 'batch',
        data: batch,
        stats: {
          processed: totalProcessed,
          errors: totalErrors,
          rate: calculateRate(totalProcessed)
        }
      };
    }

    yield { type: 'complete' };
  }, [dataSource, processors, batchSize]);

  const calculateRate = (processed) => {
    if (!startTimeRef.current) return 0;
    const elapsed = (Date.now() - startTimeRef.current) / 1000;
    return elapsed > 0 ? Math.round(processed / elapsed) : 0;
  };

  const startProcessing = useCallback(async () => {
    if (isRunning) return;

    setIsRunning(true);
    startTimeRef.current = Date.now();
    generatorRef.current = createProcessor();

    const processNext = async () => {
      if (!generatorRef.current || !isRunning) return;

      try {
        const { value, done } = await generatorRef.current.next();

        if (done) {
          setIsRunning(false);
          return;
        }

        if (value.type === 'batch') {
          setProcessedData(prev => [...prev, ...value.data]);
          setStats(value.stats);
        } else if (value.type === 'error') {
          console.error('Processing error:', value.error);
          setStats(value.stats);
        } else if (value.type === 'complete') {
          setIsRunning(false);
          return;
        }

        // Continue processing after interval
        setTimeout(processNext, interval);
      } catch (error) {
        console.error('Generator error:', error);
        setIsRunning(false);
      }
    };

    processNext();
  }, [createProcessor, interval, isRunning]);

  const stopProcessing = useCallback(() => {
    setIsRunning(false);
    generatorRef.current = null;
  }, []);

  const reset = useCallback(() => {
    stopProcessing();
    setProcessedData([]);
    setStats({ processed: 0, errors: 0, rate: 0 });
    startTimeRef.current = null;
  }, [stopProcessing]);

  useEffect(() => {
    return () => {
      stopProcessing();
    };
  }, [stopProcessing]);

  return (
    <div className="data-stream-processor">
      <div className="controls">
        <button onClick={startProcessing} disabled={isRunning}>
          Start Processing
        </button>
        <button onClick={stopProcessing} disabled={!isRunning}>
          Stop Processing
        </button>
        <button onClick={reset}>Reset</button>
      </div>

      <div className="stats">
        <div className="stat">
          <label>Processed:</label>
          <span>{stats.processed}</span>
        </div>
        <div className="stat">
          <label>Errors:</label>
          <span>{stats.errors}</span>
        </div>
        <div className="stat">
          <label>Rate:</label>
          <span>{stats.rate} items/sec</span>
        </div>
        <div className="stat">
          <label>Status:</label>
          <span>{isRunning ? 'Running' : 'Stopped'}</span>
        </div>
      </div>

      <div className="processed-data">
        <h3>Processed Data ({processedData.length} items)</h3>
        <div className="data-list">
          {processedData.slice(-20).map((item, index) => (
            <div key={item.id} className="data-item">
              <div className="item-header">
                <span>ID: {item.id}</span>
                <span>{new Date(item.timestamp).toLocaleTimeString()}</span>
              </div>
              <div className="item-content">
                <div className="original">
                  <strong>Original:</strong>
                  <pre>{JSON.stringify(item.original, null, 2)}</pre>
                </div>
                <div className="processed">
                  <strong>Processed:</strong>
                  <pre>{JSON.stringify(item.processed, null, 2)}</pre>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

export default DataStreamProcessor;

// ============= USAGE EXAMPLES =============
// pages/data-processing.js
import { useState } from 'react';
import InfiniteList from '../components/InfiniteList';
import DataStreamProcessor from '../components/DataStreamProcessor';

// Mock data source generator
function* mockDataSource() {
  let id = 1;
  while (id <= 1000) {
    yield {
      id: id++,
      value: Math.random() * 100,
      category: ['A', 'B', 'C'][Math.floor(Math.random() * 3)],
      timestamp: Date.now()
    };
  }
}

// Data processors
const processors = [
  (data) => ({ ...data, value: Math.round(data.value) }),
  (data) => ({ ...data, doubled: data.value * 2 }),
  (data) => ({ ...data, category: data.category.toLowerCase() })
];

const DataProcessingPage = () => {
  const [users, setUsers] = useState([]);

  // Mock fetch function for infinite list
  const fetchUsers = async (page, limit) => {
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 500));

    const start = (page - 1) * limit;
    const end = start + limit;
    
    const mockUsers = Array.from({ length: limit }, (_, i) => ({
      id: start + i + 1,
      name: `User ${start + i + 1}`,
      email: `user${start + i + 1}@example.com`,
      avatar: `https://api.dicebear.com/7.x/avataaars/svg?seed=${start + i + 1}`
    }));

    return {
      data: mockUsers,
      hasMore: end < 1000, // Total 1000 users
      total: 1000
    };
  };

  return (
    <div className="data-processing-page">
      <h1>Data Processing with Generators</h1>
      
      <section>
        <h2>Infinite List with Generator-based Pagination</h2>
        <InfiniteList
          fetchData={fetchUsers}
          renderItem={(user) => (
            <div className="user-card">
              <img src={user.avatar} alt={user.name} width="40" height="40" />
              <div>
                <h4>{user.name}</h4>
                <p>{user.email}</p>
              </div>
            </div>
          )}
        />
      </section>

      <section>
        <h2>Stream Processing with Generators</h2>
        <DataStreamProcessor
          dataSource={mockDataSource()}
          processors={processors}
          batchSize={5}
          interval={1000}
        />
      </section>
    </div>
  );
};

export default DataProcessingPage;
```

## 🎯 แบบฝึกหัด

### **🔧 แบบฝึกหัดที่ 1: Custom Iterator**
สร้าง custom iterator patterns:

```javascript
// TODO: สร้าง custom iterators

// 1. Tree traversal iterator
class TreeNode {
  constructor(value, children = []) {
    this.value = value;
    this.children = children;
  }
  
  // Implement depth-first iterator
  *[Symbol.iterator]() {
    // Implement tree traversal
  }
}

// 2. Matrix iterator
class Matrix {
  constructor(rows, cols, data) {
    // Implement matrix with custom iteration patterns
  }
  
  *rows() {
    // Iterate by rows
  }
  
  *columns() {
    // Iterate by columns
  }
  
  *spiral() {
    // Iterate in spiral pattern
  }
}

// 3. Async data iterator
class AsyncDataIterator {
  constructor(urls) {
    // Implement async iteration over multiple URLs
  }
  
  async *[Symbol.asyncIterator]() {
    // Implement async iteration
  }
}
```

### **⚡ แบบฝึกหัดที่ 2: Generator Applications**
สร้าง practical generator applications:

```javascript
// TODO: สร้าง generator applications

// 1. CSV parser generator
function* parseCSV(csvText, options = {}) {
  // Parse CSV line by line using generator
}

// 2. File reader generator
async function* readFileChunks(file, chunkSize = 1024) {
  // Read file in chunks using async generator
}

// 3. Data validation pipeline
function* validateData(data, validators) {
  // Process data through validation pipeline
}

// 4. Mock data generator
function* generateMockData(schema, count) {
  // Generate mock data based on schema
}
```

### **⚛️ แบบฝึกหัดที่ 3: Next.js Integration**
สร้าง Next.js components with generators:

```jsx
// TODO: สร้าง Next.js integration

// 1. Real-time data dashboard
const RealtimeDashboard = () => {
  // Use generators for real-time data streaming
  // Implement charts with live updates
  // Handle multiple data sources
};

// 2. Lazy-loaded content system
const LazyContentSystem = () => {
  // Use generators for progressive content loading
  // Implement content prioritization
  // Handle content caching
};

// 3. Interactive data explorer
const DataExplorer = () => {
  // Use generators for large dataset exploration
  // Implement filtering and sorting
  // Handle performance optimization
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 10: ES6 Modules](./10-modules.md)

**บทถัดไป**: [บทที่ 12: Symbols & Maps/Sets](./12-symbols-maps-sets.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Iterators and Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)
- [MDN: Generator Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
- [JavaScript.info: Generators](https://javascript.info/generators)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Iterator Protocol** และการสร้าง custom iterators
- **Generator Functions** และ advanced patterns
- **Lazy Evaluation** และ infinite sequences
- **Async Generators** สำหรับ asynchronous iteration
- **การใช้งานใน Next.js** สำหรับ data processing และ streaming

Iterators และ Generators เป็นเครื่องมือที่มีประสิทธิภาพสำหรับการจัดการข้อมูลและการประมวลผลแบบ lazy evaluation! 🚀
