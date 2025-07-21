# บทที่ 11: Iterators & Generators

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Iterator Protocol และ Iterable Objects
- ทำความเข้าใจ Generator Functions และ yield keyword
- เรียนรู้ Async Generators และ Advanced Patterns
- ประยุกต์ใช้ใน Next.js สำหรับ Data Processing และ UI Patterns

## 🔄 Iterator Protocol

### **🔧 Iterator Basics**

```javascript
// ============= ITERATOR PROTOCOL =============
// Iterator interface
interface Iterator<T> {
  next(): IteratorResult<T>;
}

interface IteratorResult<T> {
  value: T;
  done: boolean;
}

// Custom iterator implementation
class NumberIterator {
  constructor(start, end, step = 1) {
    this.current = start;
    this.end = end;
    this.step = step;
  }

  next() {
    if (this.current <= this.end) {
      const value = this.current;
      this.current += this.step;
      return { value, done: false };
    }
    return { value: undefined, done: true };
  }

  // Make it iterable
  [Symbol.iterator]() {
    return this;
  }
}

// Usage
const numbers = new NumberIterator(1, 5);
for (const num of numbers) {
  console.log(num); // 1, 2, 3, 4, 5
}

// Manual iteration
const iterator = new NumberIterator(10, 12);
console.log(iterator.next()); // { value: 10, done: false }
console.log(iterator.next()); // { value: 11, done: false }
console.log(iterator.next()); // { value: 12, done: false }
console.log(iterator.next()); // { value: undefined, done: true }

// ============= ITERABLE OBJECTS =============
// Array-like iterable
class CustomArray {
  constructor(...items) {
    this.items = items;
    this.length = items.length;
  }

  // Iterator method
  *[Symbol.iterator]() {
    for (let i = 0; i < this.length; i++) {
      yield this.items[i];
    }
  }

  // Additional methods
  map(callback) {
    const result = new CustomArray();
    for (const [index, item] of this.entries()) {
      result.push(callback(item, index, this));
    }
    return result;
  }

  filter(predicate) {
    const result = new CustomArray();
    for (const [index, item] of this.entries()) {
      if (predicate(item, index, this)) {
        result.push(item);
      }
    }
    return result;
  }

  *entries() {
    for (let i = 0; i < this.length; i++) {
      yield [i, this.items[i]];
    }
  }

  *keys() {
    for (let i = 0; i < this.length; i++) {
      yield i;
    }
  }

  *values() {
    yield* this.items;
  }

  push(item) {
    this.items[this.length++] = item;
    return this.length;
  }

  get(index) {
    return this.items[index];
  }

  set(index, value) {
    this.items[index] = value;
  }

  forEach(callback) {
    for (const [index, item] of this.entries()) {
      callback(item, index, this);
    }
  }

  toArray() {
    return [...this.items];
  }
}

// Usage
const customArray = new CustomArray(1, 2, 3, 4, 5);

// Iteration
for (const item of customArray) {
  console.log(item); // 1, 2, 3, 4, 5
}

// Spread operator
console.log([...customArray]); // [1, 2, 3, 4, 5]

// Array.from
console.log(Array.from(customArray)); // [1, 2, 3, 4, 5]

// Destructuring
const [first, second, ...rest] = customArray;
console.log(first, second, rest); // 1, 2, [3, 4, 5]

// ============= BUILT-IN ITERABLES =============
// String iteration
const str = "Hello";
for (const char of str) {
  console.log(char); // H, e, l, l, o
}

// Array iteration
const arr = [1, 2, 3];
for (const value of arr) {
  console.log(value); // 1, 2, 3
}

// Map iteration
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(key, value); // a 1, b 2
}

// Set iteration
const set = new Set([1, 2, 3]);
for (const value of set) {
  console.log(value); // 1, 2, 3
}

// ============= ADVANCED ITERATOR PATTERNS =============
// Infinite iterator
class InfiniteSequence {
  constructor(generator) {
    this.generator = generator;
  }

  *[Symbol.iterator]() {
    let index = 0;
    while (true) {
      yield this.generator(index++);
    }
  }

  take(count) {
    const result = [];
    const iterator = this[Symbol.iterator]();
    
    for (let i = 0; i < count; i++) {
      const { value, done } = iterator.next();
      if (done) break;
      result.push(value);
    }
    
    return result;
  }

  takeWhile(predicate) {
    const result = [];
    const iterator = this[Symbol.iterator]();
    
    while (true) {
      const { value, done } = iterator.next();
      if (done || !predicate(value)) break;
      result.push(value);
    }
    
    return result;
  }
}

// Fibonacci sequence
const fibonacci = new InfiniteSequence((n) => {
  if (n <= 1) return n;
  let a = 0, b = 1;
  for (let i = 2; i <= n; i++) {
    [a, b] = [b, a + b];
  }
  return b;
});

console.log(fibonacci.take(10)); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Even numbers
const evens = new InfiniteSequence(n => n * 2);
console.log(evens.take(5)); // [0, 2, 4, 6, 8]

// ============= CHAINING ITERATORS =============
class IteratorChain {
  constructor(iterable) {
    this.iterable = iterable;
  }

  static from(iterable) {
    return new IteratorChain(iterable);
  }

  *[Symbol.iterator]() {
    yield* this.iterable;
  }

  map(fn) {
    return new IteratorChain(this._map(fn));
  }

  *_map(fn) {
    for (const item of this.iterable) {
      yield fn(item);
    }
  }

  filter(predicate) {
    return new IteratorChain(this._filter(predicate));
  }

  *_filter(predicate) {
    for (const item of this.iterable) {
      if (predicate(item)) {
        yield item;
      }
    }
  }

  take(count) {
    return new IteratorChain(this._take(count));
  }

  *_take(count) {
    let taken = 0;
    for (const item of this.iterable) {
      if (taken >= count) break;
      yield item;
      taken++;
    }
  }

  skip(count) {
    return new IteratorChain(this._skip(count));
  }

  *_skip(count) {
    let skipped = 0;
    for (const item of this.iterable) {
      if (skipped < count) {
        skipped++;
        continue;
      }
      yield item;
    }
  }

  reduce(reducer, initialValue) {
    let accumulator = initialValue;
    for (const item of this.iterable) {
      accumulator = reducer(accumulator, item);
    }
    return accumulator;
  }

  toArray() {
    return [...this.iterable];
  }

  count() {
    let count = 0;
    for (const _ of this.iterable) {
      count++;
    }
    return count;
  }

  first() {
    for (const item of this.iterable) {
      return item;
    }
    return undefined;
  }

  last() {
    let last;
    for (const item of this.iterable) {
      last = item;
    }
    return last;
  }
}

// Usage
const result = IteratorChain
  .from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  .filter(x => x % 2 === 0)
  .map(x => x * x)
  .take(3)
  .toArray();

console.log(result); // [4, 16, 36]
```

## ⚡ Generator Functions

### **🎮 Generator Basics**

```javascript
// ============= BASIC GENERATORS =============
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

// Generator with parameters
function* parameterizedGenerator(start, end) {
  for (let i = start; i <= end; i++) {
    const input = yield i;
    if (input === 'stop') {
      return 'Stopped by user';
    }
    if (input) {
      console.log('Received input:', input);
    }
  }
  return 'Completed normally';
}

const paramGen = parameterizedGenerator(1, 5);
console.log(paramGen.next());      // { value: 1, done: false }
console.log(paramGen.next('hello')); // { value: 2, done: false }
console.log(paramGen.next());      // { value: 3, done: false }
console.log(paramGen.next('stop')); // { value: 'Stopped by user', done: true }

// ============= GENERATOR PATTERNS =============
// Infinite generators
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function* take(count, iterable) {
  let taken = 0;
  for (const item of iterable) {
    if (taken >= count) break;
    yield item;
    taken++;
  }
}

// Get first 10 fibonacci numbers
const fibSeq = fibonacci();
const first10Fib = [...take(10, fibSeq)];
console.log(first10Fib); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Random number generator
function* randomNumbers(min = 0, max = 100) {
  while (true) {
    yield Math.floor(Math.random() * (max - min + 1)) + min;
  }
}

const randomGen = randomNumbers(1, 10);
const randomArray = [...take(5, randomGen)];
console.log(randomArray); // [3, 7, 1, 9, 4] (example)

// ============= GENERATOR COMPOSITION =============
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}

function* letters() {
  yield 'a';
  yield 'b';
  yield 'c';
}

function* combined() {
  yield* numbers(); // Delegate to numbers generator
  yield* letters(); // Delegate to letters generator
  yield 'final';
}

console.log([...combined()]); // [1, 2, 3, 'a', 'b', 'c', 'final']

// Advanced composition
function* flatten(iterable) {
  for (const item of iterable) {
    if (typeof item[Symbol.iterator] === 'function' && typeof item !== 'string') {
      yield* flatten(item);
    } else {
      yield item;
    }
  }
}

const nested = [1, [2, [3, 4]], 5, [6, 7]];
console.log([...flatten(nested)]); // [1, 2, 3, 4, 5, 6, 7]

// ============= STATE MACHINES WITH GENERATORS =============
function* trafficLight() {
  while (true) {
    console.log('🔴 Red - Stop');
    yield 'red';
    
    console.log('🟡 Yellow - Caution');
    yield 'yellow';
    
    console.log('🟢 Green - Go');
    yield 'green';
  }
}

const traffic = trafficLight();
for (let i = 0; i < 6; i++) {
  const { value } = traffic.next();
  console.log(`Current light: ${value}`);
}

// Complex state machine
function* gameStateMachine() {
  let health = 100;
  let level = 1;
  
  console.log('🎮 Game Started');
  
  while (health > 0) {
    const action = yield {
      state: 'playing',
      health,
      level,
      actions: ['attack', 'defend', 'heal', 'quit']
    };
    
    switch (action) {
      case 'attack':
        const damage = Math.floor(Math.random() * 20) + 1;
        health -= damage;
        console.log(`⚔️ You took ${damage} damage! Health: ${health}`);
        
        if (Math.random() > 0.7) {
          level++;
          console.log(`🎉 Level up! Now level ${level}`);
        }
        break;
        
      case 'defend':
        console.log('🛡️ You defended successfully!');
        break;
        
      case 'heal':
        const healing = Math.floor(Math.random() * 15) + 5;
        health = Math.min(100, health + healing);
        console.log(`💖 You healed ${healing} HP! Health: ${health}`);
        break;
        
      case 'quit':
        return { state: 'quit', health, level };
        
      default:
        console.log('❓ Invalid action');
    }
  }
  
  return { state: 'game_over', health: 0, level };
}

// Play the game
const game = gameStateMachine();
let gameResult = game.next();

while (!gameResult.done) {
  const { state, health, level } = gameResult.value;
  
  if (state === 'playing') {
    // Simulate random actions
    const actions = ['attack', 'defend', 'heal'];
    const randomAction = actions[Math.floor(Math.random() * actions.length)];
    gameResult = game.next(randomAction);
  }
}

console.log('🎮 Game ended:', gameResult.value);

// ============= DATA PROCESSING GENERATORS =============
function* batchProcessor(data, batchSize) {
  for (let i = 0; i < data.length; i += batchSize) {
    const batch = data.slice(i, i + batchSize);
    yield batch;
  }
}

function* csvParser(csvText) {
  const lines = csvText.trim().split('\n');
  const headers = lines[0].split(',').map(h => h.trim());
  
  for (let i = 1; i < lines.length; i++) {
    const values = lines[i].split(',').map(v => v.trim());
    const row = {};
    
    headers.forEach((header, index) => {
      row[header] = values[index];
    });
    
    yield row;
  }
}

// Process large dataset in batches
const largeDataset = Array.from({ length: 1000 }, (_, i) => i + 1);

for (const batch of batchProcessor(largeDataset, 50)) {
  console.log(`Processing batch of ${batch.length} items: ${batch[0]} - ${batch[batch.length - 1]}`);
  // Process batch here
}

// Parse CSV data
const csvData = `
name,age,city
John,25,New York
Jane,30,Los Angeles
Bob,35,Chicago
`;

for (const row of csvParser(csvData)) {
  console.log(`${row.name} is ${row.age} years old and lives in ${row.city}`);
}

// ============= PIPELINE PROCESSING =============
function* map(iterable, transform) {
  for (const item of iterable) {
    yield transform(item);
  }
}

function* filter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) {
      yield item;
    }
  }
}

function* groupBy(iterable, keyFn) {
  const groups = new Map();
  
  for (const item of iterable) {
    const key = keyFn(item);
    if (!groups.has(key)) {
      groups.set(key, []);
    }
    groups.get(key).push(item);
  }
  
  yield* groups;
}

// Data processing pipeline
function* processingPipeline(data) {
  yield* map(
    filter(
      map(data, x => x * 2),
      x => x > 10
    ),
    x => `Result: ${x}`
  );
}

const inputData = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const results = [...processingPipeline(inputData)];
console.log(results); // ['Result: 12', 'Result: 14', 'Result: 16', 'Result: 18', 'Result: 20']
```

### **🔄 Async Generators**

```javascript
// ============= ASYNC GENERATORS =============
// Basic async generator
async function* asyncNumbers() {
  for (let i = 1; i <= 5; i++) {
    // Simulate async operation
    await new Promise(resolve => setTimeout(resolve, 1000));
    yield i;
  }
}

// Consuming async generator
async function consumeAsyncGenerator() {
  for await (const num of asyncNumbers()) {
    console.log('Received:', num);
  }
}

// Alternative consumption
async function manualAsyncConsumption() {
  const asyncGen = asyncNumbers();
  
  while (true) {
    const { value, done } = await asyncGen.next();
    if (done) break;
    console.log('Value:', value);
  }
}

// ============= ASYNC DATA STREAMING =============
async function* fetchDataStream(urls) {
  for (const url of urls) {
    try {
      console.log(`Fetching: ${url}`);
      const response = await fetch(url);
      const data = await response.json();
      yield { url, data, success: true };
    } catch (error) {
      yield { url, error: error.message, success: false };
    }
  }
}

// Stream processing with backpressure
async function* rateLimit(asyncIterable, delay) {
  for await (const item of asyncIterable) {
    yield item;
    await new Promise(resolve => setTimeout(resolve, delay));
  }
}

// Parallel processing with async generators
async function* parallelProcessor(asyncIterable, concurrency = 3) {
  const buffer = [];
  
  for await (const item of asyncIterable) {
    buffer.push(processItemAsync(item));
    
    if (buffer.length >= concurrency) {
      const result = await Promise.race(buffer);
      const index = buffer.findIndex(promise => promise === result);
      buffer.splice(index, 1);
      yield await result;
    }
  }
  
  // Process remaining items
  while (buffer.length > 0) {
    const result = await Promise.race(buffer);
    const index = buffer.findIndex(promise => promise === result);
    buffer.splice(index, 1);
    yield await result;
  }
}

async function processItemAsync(item) {
  // Simulate async processing
  await new Promise(resolve => setTimeout(resolve, Math.random() * 1000));
  return `Processed: ${item}`;
}

// ============= REAL-TIME DATA PROCESSING =============
class AsyncDataProcessor {
  constructor() {
    this.subscribers = new Set();
    this.processing = false;
  }

  async *processStream(dataSource) {
    this.processing = true;
    
    try {
      for await (const chunk of dataSource) {
        const processed = await this.processChunk(chunk);
        
        // Notify subscribers
        for (const subscriber of this.subscribers) {
          subscriber(processed);
        }
        
        yield processed;
      }
    } finally {
      this.processing = false;
    }
  }

  async processChunk(chunk) {
    // Simulate complex processing
    await new Promise(resolve => setTimeout(resolve, 100));
    
    return {
      timestamp: new Date().toISOString(),
      data: chunk,
      processed: true,
      id: Math.random().toString(36).substr(2, 9)
    };
  }

  subscribe(callback) {
    this.subscribers.add(callback);
    return () => this.subscribers.delete(callback);
  }

  isProcessing() {
    return this.processing;
  }
}

// Usage
const processor = new AsyncDataProcessor();

// Subscribe to updates
const unsubscribe = processor.subscribe((data) => {
  console.log('Subscriber received:', data);
});

// Simulate data source
async function* dataSource() {
  const data = ['chunk1', 'chunk2', 'chunk3', 'chunk4'];
  for (const chunk of data) {
    await new Promise(resolve => setTimeout(resolve, 500));
    yield chunk;
  }
}

// Process the stream
async function runProcessor() {
  for await (const result of processor.processStream(dataSource())) {
    console.log('Main processor result:', result);
  }
}

// ============= ERROR HANDLING IN ASYNC GENERATORS =============
async function* resilientAsyncGenerator(dataSource) {
  let retryCount = 0;
  const maxRetries = 3;
  
  for await (const item of dataSource) {
    while (retryCount < maxRetries) {
      try {
        const result = await processWithPossibleFailure(item);
        retryCount = 0; // Reset on success
        yield result;
        break;
      } catch (error) {
        retryCount++;
        console.log(`Attempt ${retryCount} failed for item ${item}:`, error.message);
        
        if (retryCount >= maxRetries) {
          yield { error: error.message, item, failed: true };
          retryCount = 0;
          break;
        }
        
        // Exponential backoff
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, retryCount) * 1000)
        );
      }
    }
  }
}

async function processWithPossibleFailure(item) {
  if (Math.random() < 0.3) { // 30% chance of failure
    throw new Error(`Processing failed for ${item}`);
  }
  
  await new Promise(resolve => setTimeout(resolve, 100));
  return `Successfully processed: ${item}`;
}

// ============= ASYNC GENERATOR UTILITIES =============
class AsyncIteratorUtils {
  static async *take(count, asyncIterable) {
    let taken = 0;
    for await (const item of asyncIterable) {
      if (taken >= count) break;
      yield item;
      taken++;
    }
  }

  static async *map(asyncIterable, transform) {
    for await (const item of asyncIterable) {
      yield await transform(item);
    }
  }

  static async *filter(asyncIterable, predicate) {
    for await (const item of asyncIterable) {
      if (await predicate(item)) {
        yield item;
      }
    }
  }

  static async *batch(asyncIterable, size) {
    let batch = [];
    
    for await (const item of asyncIterable) {
      batch.push(item);
      
      if (batch.length >= size) {
        yield batch;
        batch = [];
      }
    }
    
    if (batch.length > 0) {
      yield batch;
    }
  }

  static async *merge(...asyncIterables) {
    const iterators = asyncIterables.map(iterable => iterable[Symbol.asyncIterator]());
    const pending = new Map();
    
    // Start all iterators
    for (const [index, iterator] of iterators.entries()) {
      pending.set(index, iterator.next());
    }
    
    while (pending.size > 0) {
      const results = await Promise.allSettled([...pending.values()]);
      
      for (const [promiseIndex, result] of results.entries()) {
        const iteratorIndex = [...pending.keys()][promiseIndex];
        
        if (result.status === 'fulfilled') {
          const { value, done } = result.value;
          
          if (done) {
            pending.delete(iteratorIndex);
          } else {
            yield { value, source: iteratorIndex };
            // Start next iteration
            pending.set(iteratorIndex, iterators[iteratorIndex].next());
          }
        } else {
          console.error(`Iterator ${iteratorIndex} failed:`, result.reason);
          pending.delete(iteratorIndex);
        }
      }
    }
  }

  static async toArray(asyncIterable) {
    const result = [];
    for await (const item of asyncIterable) {
      result.push(item);
    }
    return result;
  }

  static async reduce(asyncIterable, reducer, initialValue) {
    let accumulator = initialValue;
    for await (const item of asyncIterable) {
      accumulator = await reducer(accumulator, item);
    }
    return accumulator;
  }
}

// Usage examples
async function* source1() {
  for (let i = 1; i <= 3; i++) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield `Source1: ${i}`;
  }
}

async function* source2() {
  for (let i = 1; i <= 3; i++) {
    await new Promise(resolve => setTimeout(resolve, 150));
    yield `Source2: ${i}`;
  }
}

// Merge multiple async iterables
async function demonstrateAsyncUtils() {
  console.log('=== Merged Sources ===');
  for await (const { value, source } of AsyncIteratorUtils.merge(source1(), source2())) {
    console.log(`From source ${source}: ${value}`);
  }
  
  console.log('\n=== Batched Processing ===');
  const numbers = AsyncIteratorUtils.map(
    AsyncIteratorUtils.take(10, asyncNumbers()),
    x => x * 2
  );
  
  for await (const batch of AsyncIteratorUtils.batch(numbers, 3)) {
    console.log('Batch:', batch);
  }
}
```

## ⚛️ Next.js Integration

### **🎯 Data Fetching Patterns**

```typescript
// ============= NEXT.JS GENERATOR PATTERNS =============
// hooks/useAsyncGenerator.ts
import { useState, useEffect, useRef } from 'react';

interface AsyncGeneratorState<T> {
  data: T[];
  loading: boolean;
  error: Error | null;
  hasMore: boolean;
}

export function useAsyncGenerator<T>(
  asyncGenerator: () => AsyncGenerator<T, void, unknown>,
  deps: React.DependencyList = []
) {
  const [state, setState] = useState<AsyncGeneratorState<T>>({
    data: [],
    loading: false,
    error: null,
    hasMore: true
  });

  const generatorRef = useRef<AsyncGenerator<T, void, unknown> | null>(null);
  const abortControllerRef = useRef<AbortController | null>(null);

  const startGenerator = async () => {
    // Cleanup previous generator
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();
    generatorRef.current = asyncGenerator();

    setState(prev => ({
      ...prev,
      loading: true,
      error: null,
      data: []
    }));

    try {
      for await (const item of generatorRef.current) {
        if (abortControllerRef.current?.signal.aborted) {
          break;
        }

        setState(prev => ({
          ...prev,
          data: [...prev.data, item]
        }));
      }

      setState(prev => ({
        ...prev,
        loading: false,
        hasMore: false
      }));
    } catch (error) {
      if (!abortControllerRef.current?.signal.aborted) {
        setState(prev => ({
          ...prev,
          loading: false,
          error: error as Error
        }));
      }
    }
  };

  const loadNext = async () => {
    if (!generatorRef.current || state.loading) return;

    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const { value, done } = await generatorRef.current.next();
      
      if (done) {
        setState(prev => ({
          ...prev,
          loading: false,
          hasMore: false
        }));
      } else {
        setState(prev => ({
          ...prev,
          loading: false,
          data: [...prev.data, value]
        }));
      }
    } catch (error) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: error as Error
      }));
    }
  };

  const reset = () => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    setState({
      data: [],
      loading: false,
      error: null,
      hasMore: true
    });
  };

  useEffect(() => {
    startGenerator();
    
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, deps);

  return {
    ...state,
    loadNext,
    reset
  };
}

// ============= INFINITE SCROLL WITH GENERATORS =============
// components/InfiniteScrollList.tsx
import React, { useCallback } from 'react';
import { useAsyncGenerator } from '../hooks/useAsyncGenerator';

interface InfiniteScrollListProps<T> {
  dataGenerator: () => AsyncGenerator<T[], void, unknown>;
  renderItem: (item: T, index: number) => React.ReactNode;
  itemKey: (item: T) => string | number;
  loadingComponent?: React.ReactNode;
  errorComponent?: (error: Error, retry: () => void) => React.ReactNode;
  emptyComponent?: React.ReactNode;
}

function InfiniteScrollList<T>({
  dataGenerator,
  renderItem,
  itemKey,
  loadingComponent = <div>Loading...</div>,
  errorComponent = (error, retry) => (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={retry}>Retry</button>
    </div>
  ),
  emptyComponent = <div>No data available</div>
}: InfiniteScrollListProps<T>) {
  // Flatten batches into individual items
  const flatDataGenerator = useCallback(async function* () {
    for await (const batch of dataGenerator()) {
      for (const item of batch) {
        yield item;
      }
    }
  }, [dataGenerator]);

  const { data, loading, error, hasMore, loadNext, reset } = useAsyncGenerator(
    flatDataGenerator,
    [dataGenerator]
  );

  const handleScroll = useCallback(
    (e: React.UIEvent<HTMLDivElement>) => {
      const { scrollTop, scrollHeight, clientHeight } = e.currentTarget;
      
      if (
        scrollHeight - scrollTop <= clientHeight * 1.5 && // Load when 50% from bottom
        hasMore &&
        !loading
      ) {
        loadNext();
      }
    },
    [hasMore, loading, loadNext]
  );

  if (error) {
    return errorComponent(error, reset);
  }

  if (!loading && data.length === 0) {
    return emptyComponent;
  }

  return (
    <div
      className="infinite-scroll-container"
      onScroll={handleScroll}
      style={{
        height: '400px',
        overflowY: 'auto',
        border: '1px solid #ccc',
        padding: '16px'
      }}
    >
      {data.map((item, index) => (
        <div key={itemKey(item)} className="list-item">
          {renderItem(item, index)}
        </div>
      ))}
      
      {loading && (
        <div className="loading-indicator">
          {loadingComponent}
        </div>
      )}
      
      {!hasMore && data.length > 0 && (
        <div className="end-indicator">
          <p>No more items to load</p>
        </div>
      )}
    </div>
  );
}

// ============= REAL-TIME DATA STREAMING =============
// hooks/useRealtimeStream.ts
export function useRealtimeStream<T>(
  streamGenerator: () => AsyncGenerator<T, void, unknown>,
  options: {
    bufferSize?: number;
    autoStart?: boolean;
  } = {}
) {
  const { bufferSize = 100, autoStart = true } = options;
  
  const [stream, setStream] = useState<T[]>([]);
  const [isConnected, setIsConnected] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const generatorRef = useRef<AsyncGenerator<T, void, unknown> | null>(null);
  const isRunningRef = useRef(false);

  const startStream = useCallback(async () => {
    if (isRunningRef.current) return;
    
    isRunningRef.current = true;
    setIsConnected(true);
    setError(null);
    
    try {
      generatorRef.current = streamGenerator();
      
      for await (const data of generatorRef.current) {
        setStream(prev => {
          const newStream = [...prev, data];
          // Keep only last N items
          return newStream.slice(-bufferSize);
        });
      }
    } catch (err) {
      setError(err as Error);
    } finally {
      setIsConnected(false);
      isRunningRef.current = false;
    }
  }, [streamGenerator, bufferSize]);

  const stopStream = useCallback(() => {
    isRunningRef.current = false;
    setIsConnected(false);
    if (generatorRef.current) {
      generatorRef.current.return?.(undefined);
    }
  }, []);

  const clearStream = useCallback(() => {
    setStream([]);
    setError(null);
  }, []);

  useEffect(() => {
    if (autoStart) {
      startStream();
    }
    
    return () => {
      stopStream();
    };
  }, [autoStart, startStream, stopStream]);

  return {
    stream,
    isConnected,
    error,
    startStream,
    stopStream,
    clearStream
  };
}

// ============= DATA PROCESSING PIPELINE =============
// components/DataPipelineVisualizer.tsx
interface ProcessingStep<T, U> {
  name: string;
  processor: (data: T) => Promise<U> | U;
  color?: string;
}

interface DataPipelineVisualizerProps<T> {
  initialData: T[];
  steps: ProcessingStep<any, any>[];
  onComplete?: (result: any[]) => void;
}

function DataPipelineVisualizer<T>({
  initialData,
  steps,
  onComplete
}: DataPipelineVisualizerProps<T>) {
  const [currentStep, setCurrentStep] = useState(0);
  const [processedData, setProcessedData] = useState<any[]>([]);
  const [processing, setProcessing] = useState(false);

  const processData = useCallback(async function* () {
    let data = initialData;
    
    for (const [stepIndex, step] of steps.entries()) {
      setCurrentStep(stepIndex);
      setProcessing(true);
      
      const newData = [];
      
      for (const [itemIndex, item] of data.entries()) {
        try {
          const result = await step.processor(item);
          newData.push(result);
          
          // Yield intermediate results for visualization
          yield {
            step: stepIndex,
            stepName: step.name,
            item: itemIndex,
            total: data.length,
            result,
            intermediate: newData.slice()
          };
          
          // Small delay for visualization
          await new Promise(resolve => setTimeout(resolve, 100));
        } catch (error) {
          console.error(`Error in step ${step.name}:`, error);
          newData.push(null);
        }
      }
      
      data = newData.filter(item => item !== null);
    }
    
    setProcessing(false);
    return data;
  }, [initialData, steps]);

  const { data: pipelineResults } = useAsyncGenerator(processData, [initialData, steps]);

  useEffect(() => {
    if (pipelineResults.length > 0) {
      const latest = pipelineResults[pipelineResults.length - 1];
      setProcessedData(latest.intermediate);
      
      if (latest.step === steps.length - 1) {
        onComplete?.(latest.intermediate);
      }
    }
  }, [pipelineResults, steps.length, onComplete]);

  return (
    <div className="pipeline-visualizer">
      <div className="steps-progress">
        {steps.map((step, index) => (
          <div
            key={index}
            className={`step ${index <= currentStep ? 'active' : ''} ${
              index === currentStep && processing ? 'processing' : ''
            }`}
            style={{ backgroundColor: step.color || '#007acc' }}
          >
            <span>{step.name}</span>
            {index === currentStep && processing && (
              <div className="spinner">⚡</div>
            )}
          </div>
        ))}
      </div>
      
      <div className="data-preview">
        <h3>Current Data ({processedData.length} items)</h3>
        <div className="data-grid">
          {processedData.slice(0, 10).map((item, index) => (
            <div key={index} className="data-item">
              <pre>{JSON.stringify(item, null, 2)}</pre>
            </div>
          ))}
          {processedData.length > 10 && (
            <div className="data-item more">
              +{processedData.length - 10} more items...
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

// Usage example
const ExampleUsage = () => {
  const sampleData = [
    { id: 1, name: 'John', age: 25 },
    { id: 2, name: 'Jane', age: 30 },
    { id: 3, name: 'Bob', age: 35 }
  ];

  const processingSteps = [
    {
      name: 'Validate',
      processor: (item: any) => {
        if (!item.name || !item.age) throw new Error('Invalid item');
        return item;
      },
      color: '#ff6b6b'
    },
    {
      name: 'Transform',
      processor: (item: any) => ({
        ...item,
        nameUpper: item.name.toUpperCase(),
        isAdult: item.age >= 18
      }),
      color: '#4ecdc4'
    },
    {
      name: 'Enrich',
      processor: async (item: any) => {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 200));
        return {
          ...item,
          timestamp: new Date().toISOString(),
          category: item.age < 30 ? 'young' : 'mature'
        };
      },
      color: '#45b7d1'
    }
  ];

  return (
    <DataPipelineVisualizer
      initialData={sampleData}
      steps={processingSteps}
      onComplete={(result) => console.log('Pipeline complete:', result)}
    />
  );
};
```

### **🎨 UI Patterns with Generators**

```typescript
// ============= ANIMATION SEQUENCES =============
// hooks/useAnimationSequence.ts
interface AnimationStep {
  duration: number;
  easing?: string;
  properties: Record<string, any>;
}

export function useAnimationSequence(steps: AnimationStep[]) {
  const [currentStep, setCurrentStep] = useState(0);
  const [isPlaying, setIsPlaying] = useState(false);
  const [currentProperties, setCurrentProperties] = useState({});

  const animationGenerator = useCallback(async function* () {
    for (const [index, step] of steps.entries()) {
      setCurrentStep(index);
      setCurrentProperties(step.properties);
      
      yield {
        step: index,
        properties: step.properties,
        duration: step.duration,
        easing: step.easing || 'ease'
      };
      
      await new Promise(resolve => setTimeout(resolve, step.duration));
    }
  }, [steps]);

  const { data: animationFrames, loading } = useAsyncGenerator(
    animationGenerator,
    [steps]
  );

  const play = useCallback(() => {
    setIsPlaying(true);
  }, []);

  const pause = useCallback(() => {
    setIsPlaying(false);
  }, []);

  const reset = useCallback(() => {
    setCurrentStep(0);
    setCurrentProperties(steps[0]?.properties || {});
    setIsPlaying(false);
  }, [steps]);

  return {
    currentStep,
    currentProperties,
    isPlaying,
    play,
    pause,
    reset,
    animationFrames
  };
}

// ============= FORM WIZARD WITH GENERATORS =============
// components/FormWizard.tsx
interface WizardStep {
  id: string;
  title: string;
  component: React.ComponentType<any>;
  validate?: (data: any) => Promise<boolean> | boolean;
  onEnter?: (data: any) => void;
  onExit?: (data: any) => void;
}

interface FormWizardProps {
  steps: WizardStep[];
  onComplete: (data: any) => void;
  onCancel?: () => void;
}

export const FormWizard: React.FC<FormWizardProps> = ({
  steps,
  onComplete,
  onCancel
}) => {
  const [formData, setFormData] = useState({});
  const [currentStepIndex, setCurrentStepIndex] = useState(0);
  const [isValidating, setIsValidating] = useState(false);
  const [errors, setErrors] = useState<Record<string, string>>({});

  const wizardGenerator = useCallback(async function* () {
    for (const [index, step] of steps.entries()) {
      // Enter step
      step.onEnter?.(formData);
      
      yield {
        step,
        index,
        isFirst: index === 0,
        isLast: index === steps.length - 1,
        canGoNext: true,
        canGoPrevious: index > 0
      };
      
      // Wait for user action (this would be controlled externally)
    }
  }, [steps, formData]);

  const validateCurrentStep = async () => {
    const currentStep = steps[currentStepIndex];
    
    if (!currentStep.validate) return true;
    
    setIsValidating(true);
    setErrors({});
    
    try {
      const isValid = await currentStep.validate(formData);
      
      if (!isValid) {
        setErrors({
          [currentStep.id]: 'Validation failed for this step'
        });
      }
      
      return isValid;
    } catch (error) {
      setErrors({
        [currentStep.id]: error instanceof Error ? error.message : 'Validation error'
      });
      return false;
    } finally {
      setIsValidating(false);
    }
  };

  const goToNext = async () => {
    const isValid = await validateCurrentStep();
    
    if (isValid) {
      const currentStep = steps[currentStepIndex];
      currentStep.onExit?.(formData);
      
      if (currentStepIndex < steps.length - 1) {
        setCurrentStepIndex(prev => prev + 1);
      } else {
        onComplete(formData);
      }
    }
  };

  const goToPrevious = () => {
    if (currentStepIndex > 0) {
      const currentStep = steps[currentStepIndex];
      currentStep.onExit?.(formData);
      setCurrentStepIndex(prev => prev - 1);
      setErrors({});
    }
  };

  const updateFormData = (stepData: any) => {
    setFormData(prev => ({
      ...prev,
      [steps[currentStepIndex].id]: stepData
    }));
  };

  const currentStep = steps[currentStepIndex];
  const StepComponent = currentStep.component;

  return (
    <div className="form-wizard">
      <div className="wizard-header">
        <div className="step-indicator">
          {steps.map((step, index) => (
            <div
              key={step.id}
              className={`step-dot ${
                index <= currentStepIndex ? 'completed' : ''
              } ${index === currentStepIndex ? 'current' : ''}`}
            >
              {index + 1}
            </div>
          ))}
        </div>
        <h2>{currentStep.title}</h2>
      </div>

      <div className="wizard-content">
        <StepComponent
          data={formData[currentStep.id] || {}}
          onChange={updateFormData}
          errors={errors}
        />
      </div>

      <div className="wizard-actions">
        <button
          onClick={onCancel}
          disabled={isValidating}
        >
          Cancel
        </button>
        
        <button
          onClick={goToPrevious}
          disabled={currentStepIndex === 0 || isValidating}
        >
          Previous
        </button>
        
        <button
          onClick={goToNext}
          disabled={isValidating}
        >
          {isValidating
            ? 'Validating...'
            : currentStepIndex === steps.length - 1
            ? 'Complete'
            : 'Next'
          }
        </button>
      </div>

      {errors[currentStep.id] && (
        <div className="error-message">
          {errors[currentStep.id]}
        </div>
      )}
    </div>
  );
};

// ============= VIRTUAL SCROLLING WITH GENERATORS =============
// hooks/useVirtualizedList.ts
interface VirtualizedListOptions {
  itemHeight: number;
  containerHeight: number;
  overscan?: number;
}

export function useVirtualizedList<T>(
  items: T[],
  options: VirtualizedListOptions
) {
  const { itemHeight, containerHeight, overscan = 5 } = options;
  
  const [scrollTop, setScrollTop] = useState(0);
  
  const visibleItemsGenerator = useCallback(function* () {
    const visibleCount = Math.ceil(containerHeight / itemHeight);
    const startIndex = Math.floor(scrollTop / itemHeight);
    const endIndex = Math.min(
      startIndex + visibleCount + overscan,
      items.length
    );
    
    const actualStartIndex = Math.max(0, startIndex - overscan);
    
    for (let i = actualStartIndex; i < endIndex; i++) {
      if (items[i]) {
        yield {
          index: i,
          item: items[i],
          top: i * itemHeight,
          isVisible: i >= startIndex && i < startIndex + visibleCount
        };
      }
    }
  }, [items, itemHeight, scrollTop, containerHeight, overscan]);

  const visibleItems = useMemo(() => {
    return Array.from(visibleItemsGenerator());
  }, [visibleItemsGenerator]);

  const totalHeight = items.length * itemHeight;
  
  const handleScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  }, []);

  return {
    visibleItems,
    totalHeight,
    handleScroll,
    scrollTop
  };
}

// components/VirtualizedList.tsx
interface VirtualizedListProps<T> {
  items: T[];
  itemHeight: number;
  height: number;
  renderItem: (item: T, index: number) => React.ReactNode;
  getItemKey: (item: T, index: number) => string | number;
}

export function VirtualizedList<T>({
  items,
  itemHeight,
  height,
  renderItem,
  getItemKey
}: VirtualizedListProps<T>) {
  const { visibleItems, totalHeight, handleScroll } = useVirtualizedList(
    items,
    { itemHeight, containerHeight: height }
  );

  return (
    <div
      className="virtualized-list"
      style={{ height, overflowY: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        {visibleItems.map(({ item, index, top }) => (
          <div
            key={getItemKey(item, index)}
            style={{
              position: 'absolute',
              top,
              left: 0,
              right: 0,
              height: itemHeight
            }}
          >
            {renderItem(item, index)}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 🎯 แบบฝึกหัด

### **🔄 แบบฝึกหัดที่ 1: Custom Iterators**
สร้าง custom iterator system:

```javascript
// TODO: สร้าง custom iterators

// 1. สร้าง Tree iterator
class TreeIterator {
  // Traverse tree in different orders
  // Support depth-first, breadth-first
}

// 2. สร้าง Matrix iterator
class MatrixIterator {
  // Iterate through 2D matrix
  // Support different patterns (row, column, diagonal)
}

// 3. สร้าง Pagination iterator
class PaginationIterator {
  // Handle paginated API responses
  // Support async loading
}

// 4. สร้าง Filter chain
class FilterChain {
  // Chain multiple filters
  // Lazy evaluation
}
```

### **⚡ แบบฝึกหัดที่ 2: Generator Applications**
สร้าง practical generator applications:

```javascript
// TODO: สร้าง generator applications

// 1. Data transformation pipeline
function* createPipeline(transformers) {
  // Apply multiple transformations
  // Support async transformers
}

// 2. State machine
function* createStateMachine(states, transitions) {
  // Handle state transitions
  // Support guards and actions
}

// 3. Task scheduler
function* taskScheduler(tasks) {
  // Schedule and execute tasks
  // Support priorities and dependencies
}

// 4. Event emitter
function* eventEmitter() {
  // Emit and handle events
  // Support async handlers
}
```

### **⚛️ แบบฝึกหัดที่ 3: React Integration**
สร้าง React patterns with generators:

```tsx
// TODO: สร้าง React generator patterns

// 1. Infinite loading component
const InfiniteLoader = () => {
  // Use generators for infinite scroll
  // Handle loading states
};

// 2. Form stepper
const FormStepper = () => {
  // Multi-step form with generators
  // Handle validation and navigation
};

// 3. Animation controller
const AnimationController = () => {
  // Control complex animations
  // Support sequences and loops
};

// 4. Data stream visualizer
const StreamVisualizer = () => {
  // Visualize real-time data
  // Handle backpressure
};
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 10: ES6 Modules](./10-modules.md)

**บทถัดไป**: [บทที่ 12: Symbols & Maps/Sets](./12-symbols-maps-sets.md)

**กลับไปหน้าหลัก**: [README](../README.md)

---

## 📚 เอกสารอ้างอิง

- [MDN: Iterators and Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)
- [MDN: Iteration Protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)
- [MDN: async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

---

## 🎉 สรุป

ในบทนี้เราได้เรียนรู้:

- **Iterator Protocol** และการสร้าง custom iterators
- **Generator Functions** และ yield keyword
- **Async Generators** สำหรับ async data processing
- **การใช้งานใน React/Next.js** สำหรับ UI patterns
- **Advanced patterns** เช่น data pipelines และ state machines

Iterators และ Generators เป็นเครื่องมือที่ทรงพลังสำหรับการจัดการ data flow และสร้าง efficient applications! 🚀
