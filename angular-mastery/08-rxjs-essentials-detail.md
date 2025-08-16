# บทที่ 8: RxJS Essentials สำหรับ Angular

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจหลักการพื้นฐานของ reactive programming
- สร้างและใช้งาน Observables ใน Angular applications
- ใช้ RxJS operators ที่สำคัญในการจัดการกับข้อมูลแบบ asynchronous
- จัดการ error และ completion ใน observable streams
- ป้องกัน memory leaks จากการใช้ observables
- เข้าใจและใช้งาน Subjects ประเภทต่างๆ
- ประยุกต์ใช้ RxJS กับ Angular services และ components

## 📋 RxJS คืออะไร?

RxJS (Reactive Extensions for JavaScript) เป็นไลบรารีสำหรับ **reactive programming** ที่ใช้ **Observables** เพื่อทำงานกับข้อมูลแบบ asynchronous และ event-based

### 🔑 ความสำคัญของ RxJS ใน Angular

- **HTTP Requests** - Angular HttpClient คืน Observables
- **Forms** - `valueChanges` ของ form controls เป็น Observables
- **Router Events** - Router events เป็น Observables
- **State Management** - ใช้ RxJS ในการจัดการ state แบบ reactive

## 🌟 Observable Fundamentals

### สร้าง Observable แบบพื้นฐาน

```typescript
import { Observable } from 'rxjs';

// สร้าง Observable แบบง่าย
const simpleObservable = new Observable<number>(observer => {
  observer.next(1);      // ส่งค่า 1
  observer.next(2);      // ส่งค่า 2
  observer.next(3);      // ส่งค่า 3
  observer.complete();   // จบการทำงาน
});

// Subscribe เพื่อรับข้อมูล
simpleObservable.subscribe({
  next: value => console.log('Value:', value),
  error: err => console.error('Error:', err),
  complete: () => console.log('Observable completed')
});

// Output:
// Value: 1
// Value: 2
// Value: 3
// Observable completed
```

### Observable Creation Functions

RxJS มีฟังก์ชันมากมายสำหรับสร้าง Observables:

```typescript
import { of, from, interval, timer, fromEvent } from 'rxjs';

// of - สร้างจากค่าที่กำหนด
const ofObservable = of(1, 2, 3, 4, 5);
ofObservable.subscribe(val => console.log('of:', val));

// from - สร้างจาก array, promise, iterable
const arrayObservable = from([10, 20, 30]);
arrayObservable.subscribe(val => console.log('from array:', val));

const promiseObservable = from(Promise.resolve('Hello from Promise'));
promiseObservable.subscribe(val => console.log('from promise:', val));

// interval - ปล่อยค่าทุกๆ n มิลลิวินาที
const intervalObservable = interval(1000); // ทุก 1 วินาที
const intervalSubscription = intervalObservable.subscribe(val => console.log('interval:', val));

// หยุด subscription หลังจาก 5 วินาที
setTimeout(() => {
  intervalSubscription.unsubscribe();
  console.log('Unsubscribed from interval');
}, 5000);

// timer - หน่วงเวลาก่อนปล่อยค่า
const timerObservable = timer(2000, 1000); // เริ่มหลัง 2 วินาที, จากนั้นทุกๆ 1 วินาที
timerObservable.subscribe(val => console.log('timer:', val));

// fromEvent - สร้างจาก DOM events
const button = document.querySelector('button');
if (button) {
  const clickObservable = fromEvent(button, 'click');
  clickObservable.subscribe(event => console.log('Button clicked:', event));
}
```

## 🔨 RxJS Operators

Operators คือฟังก์ชันที่ใช้จัดการกับข้อมูลใน observable stream

### Pipeable Operators ที่สำคัญ

```typescript
import { of, from, interval } from 'rxjs';
import { 
  map, filter, tap, take, takeUntil, debounceTime, 
  distinctUntilChanged, switchMap, catchError
} from 'rxjs/operators';

// map - แปลงค่าในแต่ละ emission
of(1, 2, 3).pipe(
  map(x => x * 10)
).subscribe(val => console.log('map:', val));
// Output: map: 10, map: 20, map: 30

// filter - กรองค่าตามเงื่อนไข
from([1, 2, 3, 4, 5, 6]).pipe(
  filter(x => x % 2 === 0)
).subscribe(val => console.log('filter:', val));
// Output: filter: 2, filter: 4, filter: 6

// tap - ทำ side effects โดยไม่เปลี่ยนค่า
of('a', 'b', 'c').pipe(
  tap(val => console.log('Processing:', val)),
  map(val => val.toUpperCase())
).subscribe(val => console.log('Processed:', val));
// Output:
// Processing: a
// Processed: A
// Processing: b
// Processed: B
// Processing: c
// Processed: C

// take - รับค่าตามจำนวนที่กำหนด
interval(1000).pipe(
  take(3)
).subscribe({
  next: val => console.log('take:', val),
  complete: () => console.log('take completed')
});
// Output: (ทุก 1 วินาที)
// take: 0
// take: 1
// take: 2
// take completed
```

### Transformation Operators

```typescript
import { of, from, fromEvent } from 'rxjs';
import { 
  map, mergeMap, switchMap, concatMap, 
  exhaustMap, scan, reduce 
} from 'rxjs/operators';
import { ajax } from 'rxjs/ajax';

// switchMap - สลับไปยัง Observable ใหม่ทุกครั้งที่มี emission
// และยกเลิก subscription ก่อนหน้า
const searchBox = document.getElementById('search') as HTMLInputElement;
const searchInput$ = fromEvent(searchBox, 'input');

searchInput$.pipe(
  map(event => (event.target as HTMLInputElement).value),
  debounceTime(300),  // รอจนกว่าการพิมพ์จะหยุด 300ms
  distinctUntilChanged(),  // ส่งค่าเมื่อมีการเปลี่ยนแปลงเท่านั้น
  switchMap(term => {
    console.log('Searching for:', term);
    // สร้าง HTTP request ใหม่ทุกครั้ง และยกเลิก request เก่า
    return ajax.getJSON(`https://api.github.com/search/repositories?q=${term}`);
  })
).subscribe(response => console.log('Search results:', response));

// mergeMap - รวม Observable หลายตัวและไม่ยกเลิก subscriptions เก่า
from([1, 2, 3]).pipe(
  mergeMap(id => {
    console.log(`Fetching user ${id}`);
    return ajax.getJSON(`https://jsonplaceholder.typicode.com/users/${id}`);
  })
).subscribe(user => console.log('User:', user));
// ทำ request ทั้งหมดพร้อมกัน

// concatMap - เหมือน mergeMap แต่รอให้ Observable ก่อนหน้าเสร็จก่อน
from([1, 2, 3]).pipe(
  concatMap(id => {
    console.log(`Fetching post ${id}`);
    return ajax.getJSON(`https://jsonplaceholder.typicode.com/posts/${id}`);
  })
).subscribe(post => console.log('Post:', post));
// รอให้ request แรกเสร็จก่อนทำ request ถัดไป

// scan และ reduce - สะสมค่า
of(1, 2, 3, 4).pipe(
  scan((acc, val) => acc + val, 0)
).subscribe(sum => console.log('Running sum:', sum));
// Output:
// Running sum: 1
// Running sum: 3
// Running sum: 6
// Running sum: 10

of(1, 2, 3, 4).pipe(
  reduce((acc, val) => acc + val, 0)
).subscribe(sum => console.log('Final sum:', sum));
// Output:
// Final sum: 10
```

### Filtering Operators

```typescript
import { interval, of, fromEvent } from 'rxjs';
import { 
  filter, take, takeUntil, takeWhile, skip, 
  debounceTime, throttleTime, distinctUntilChanged 
} from 'rxjs/operators';

// take, takeUntil, takeWhile
const counter$ = interval(1000);

// take - รับค่า n ครั้งแล้ว complete
counter$.pipe(
  take(5)
).subscribe({
  next: val => console.log('take:', val),
  complete: () => console.log('take completed')
});
// Output: 0, 1, 2, 3, 4, completed

// takeWhile - รับค่าจนกว่าเงื่อนไขจะเป็น false
counter$.pipe(
  takeWhile(val => val < 5)
).subscribe({
  next: val => console.log('takeWhile:', val),
  complete: () => console.log('takeWhile completed')
});
// Output: 0, 1, 2, 3, 4, completed

// takeUntil - รับค่าจนกว่าจะมีการ emit จาก observable อื่น
const stopButton = document.querySelector('#stop-button');
if (stopButton) {
  const stopClick$ = fromEvent(stopButton, 'click');
  
  counter$.pipe(
    takeUntil(stopClick$)
  ).subscribe({
    next: val => console.log('Counter until stop:', val),
    complete: () => console.log('Stopped by button click')
  });
}

// debounceTime - รอจนไม่มีการ emit ค่าใหม่ในช่วงเวลาที่กำหนด
const searchInput = document.querySelector('#search');
if (searchInput) {
  fromEvent(searchInput, 'input').pipe(
    debounceTime(300),
    map((event: Event) => (event.target as HTMLInputElement).value),
    distinctUntilChanged()
  ).subscribe(value => console.log('Search term:', value));
}

// throttleTime - จำกัดการ emit ให้เว้นช่วงเวลาที่กำหนด
const mouseMoves = fromEvent(document, 'mousemove');
mouseMoves.pipe(
  throttleTime(1000)  // รับค่าแรก แล้วเว้น 1 วินาที
).subscribe(event => console.log('Mouse position:', event));
```

### Combination Operators

```typescript
import { combineLatest, merge, concat, zip, forkJoin, of, interval } from 'rxjs';
import { map, take, delay } from 'rxjs/operators';

// combineLatest - รวมค่าล่าสุดจากหลาย observables
const temperature$ = of(22, 23, 24).pipe(
  map(temp => `${temp}°C`),
  delay(1000)
);
const humidity$ = of(60, 62).pipe(
  map(humidity => `${humidity}%`),
  delay(1500)
);

combineLatest([temperature$, humidity$]).pipe(
  map(([temp, humidity]) => `Weather: ${temp}, Humidity: ${humidity}`)
).subscribe(console.log);
// Output:
// Weather: 22°C, Humidity: 60%
// Weather: 23°C, Humidity: 60%
// Weather: 24°C, Humidity: 60%
// Weather: 24°C, Humidity: 62%

// merge - รวม emissions จากหลาย observables เป็น stream เดียว
const numbers$ = of(1, 2, 3);
const letters$ = of('a', 'b', 'c');
merge(numbers$, letters$).subscribe(console.log);
// Output: 1, 2, 3, a, b, c (อาจมีลำดับต่างกันได้)

// concat - เรียงลำดับ observables ต่อกัน
concat(numbers$, letters$).subscribe(console.log);
// Output: 1, 2, 3, a, b, c (เรียงตามลำดับเสมอ)

// zip - จับคู่ emissions ตามลำดับ
const firstNames = of('John', 'Jane', 'Bob');
const lastNames = of('Smith', 'Doe', 'Johnson');
zip(firstNames, lastNames).pipe(
  map(([first, last]) => `${first} ${last}`)
).subscribe(console.log);
// Output: John Smith, Jane Doe, Bob Johnson

// forkJoin - รอให้ทุก observables complete แล้วรวมค่าสุดท้าย
const api1$ = of({ user: 'john' }).pipe(delay(1000));
const api2$ = of({ posts: [1, 2, 3] }).pipe(delay(2000));
const api3$ = of({ settings: { theme: 'dark' } }).pipe(delay(3000));

console.log('Making API calls...');
forkJoin([api1$, api2$, api3$]).subscribe({
  next: ([user, posts, settings]) => {
    console.log('All API calls completed:', { user, posts, settings });
  },
  complete: () => console.log('forkJoin completed')
});
// Output: (หลังจาก 3 วินาที)
// Making API calls...
// All API calls completed: {user: {user: 'john'}, posts: {posts: [1, 2, 3]}, settings: {settings: {theme: 'dark'}}}
// forkJoin completed
```

## 🎭 Subjects ใน RxJS

Subjects เป็นทั้ง Observable และ Observer ในตัวเดียวกัน - สามารถทั้ง subscribe และ emit ค่า

### ประเภทของ Subject

```typescript
import { Subject, BehaviorSubject, ReplaySubject, AsyncSubject } from 'rxjs';

// 1. Basic Subject
console.log('--- Basic Subject ---');
const subject = new Subject<number>();

// Observers
subject.subscribe(val => console.log('Observer A:', val));

subject.next(1);  // Output: Observer A: 1

// เพิ่ม observer ใหม่
subject.subscribe(val => console.log('Observer B:', val));

subject.next(2);  // Output: Observer A: 2, Observer B: 2
subject.complete();

// 2. BehaviorSubject - เก็บค่าล่าสุดและส่งให้ subscriber ใหม่
console.log('--- BehaviorSubject ---');
const behaviorSubject = new BehaviorSubject<number>(0); // ต้องระบุค่าเริ่มต้น

// Observer จะได้รับค่าล่าสุดทันทีที่ subscribe
behaviorSubject.subscribe(val => console.log('Behavior Observer A:', val));
// Output: Behavior Observer A: 0

behaviorSubject.next(1);
// Output: Behavior Observer A: 1

behaviorSubject.subscribe(val => console.log('Behavior Observer B:', val));
// Observer B ได้รับค่าล่าสุด (1) ทันที
// Output: Behavior Observer B: 1

behaviorSubject.next(2);
// Output:
// Behavior Observer A: 2
// Behavior Observer B: 2

console.log('Current value:', behaviorSubject.value);
// Output: Current value: 2

// 3. ReplaySubject - เก็บค่าตามจำนวนที่กำหนดและส่งให้ subscriber ใหม่
console.log('--- ReplaySubject ---');
const replaySubject = new ReplaySubject<number>(3); // เก็บ 3 ค่าล่าสุด

replaySubject.next(1);
replaySubject.next(2);
replaySubject.next(3);
replaySubject.next(4);

// จะได้รับ 3 ค่าล่าสุด (2, 3, 4)
replaySubject.subscribe(val => console.log('Replay Observer:', val));
// Output:
// Replay Observer: 2
// Replay Observer: 3
// Replay Observer: 4

// 4. AsyncSubject - ส่งเฉพาะค่าสุดท้ายหลัง complete
console.log('--- AsyncSubject ---');
const asyncSubject = new AsyncSubject<number>();

asyncSubject.subscribe(val => console.log('Async Observer A:', val));

asyncSubject.next(1);
asyncSubject.next(2);
asyncSubject.next(3);

// ยังไม่มี output จนกว่าจะ complete
asyncSubject.subscribe(val => console.log('Async Observer B:', val));

asyncSubject.next(4);
asyncSubject.complete(); // ตอนนี้ทั้ง Observer A และ B จะได้รับค่า 4
// Output:
// Async Observer A: 4
// Async Observer B: 4
```

## 🛡️ Error Handling ใน RxJS

```typescript
import { of, throwError, EMPTY, from, interval } from 'rxjs';
import { catchError, retry, retryWhen, delay, take, map, tap } from 'rxjs/operators';
import { ajax } from 'rxjs/ajax';

// catchError - จัดการ error และคืนค่า fallback
ajax.getJSON('/api/data-that-does-not-exist').pipe(
  catchError(error => {
    console.error('API Error:', error.message);
    return of({ fallback: 'data' }); // ส่ง fallback data
  })
).subscribe(data => console.log('Received data:', data));
// Output: Received data: {fallback: 'data'}

// ทำงานต่อได้แม้ observer ก่อนหน้าเกิด error
of(1, 2, 3, 4).pipe(
  map(val => {
    if (val === 3) throw new Error('Error at value 3');
    return val * 10;
  }),
  catchError((err, caught) => {
    console.error('Error caught:', err.message);
    // return EMPTY; // หยุด stream และ complete
    return of(0); // ส่งค่า fallback แล้ว complete
    // return caught; // ลองใหม่ทั้ง stream (อาจทำให้เกิด infinite loop)
  })
).subscribe({
  next: val => console.log('Value:', val),
  error: err => console.error('Uncaught error:', err),
  complete: () => console.log('Stream completed')
});
// Output:
// Value: 10
// Value: 20
// Error caught: Error at value 3
// Value: 0
// Stream completed

// retry - ลองใหม่เมื่อเกิด error
ajax.getJSON('/api/data-that-might-fail').pipe(
  retry(3), // ลองใหม่ 3 ครั้งก่อนปล่อย error
  catchError(error => {
    console.error('Failed after retries:', error);
    return of({ error: 'Service unavailable' });
  })
).subscribe(data => console.log('API response:', data));

// retryWhen - ควบคุมการ retry อย่างละเอียด (เช่น exponential backoff)
ajax.getJSON('/api/data').pipe(
  retryWhen(errors => 
    errors.pipe(
      // Log error
      tap(err => console.log('Error occurred, retrying:', err.message)),
      // ดีเลย์ 1 วินาทีก่อนลองใหม่
      delay(1000),
      // ลองแค่ 3 ครั้งแล้ว throw error
      take(3),
      // ถ้าเกิน 3 ครั้ง throw error ออกไป
      tap(() => { throw new Error('Max retries reached'); })
    )
  ),
  catchError(err => {
    console.error('Final error:', err);
    return EMPTY;
  })
).subscribe({
  next: data => console.log('Data:', data),
  complete: () => console.log('Operation completed')
});
```

## 🧹 Memory Management และการป้องกัน Memory Leaks

การจัดการ Subscriptions อย่างเหมาะสมเป็นสิ่งสำคัญเพื่อป้องกัน memory leaks

### การใช้ unsubscribe

```typescript
// ไม่มีการ unsubscribe (อาจเกิด memory leak)
const subscription = interval(1000).subscribe(val => {
  console.log('Interval:', val);
});

// วิธีที่ถูกต้อง - unsubscribe เมื่อไม่ต้องการแล้ว
setTimeout(() => {
  subscription.unsubscribe();
  console.log('Unsubscribed manually');
}, 5000);
```

### การใช้ takeUntil Pattern

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { interval, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-timer',
  template: '{{counter}}'
})
export class TimerComponent implements OnInit, OnDestroy {
  counter = 0;
  private destroy$ = new Subject<void>();

  ngOnInit() {
    interval(1000).pipe(
      // หยุด subscription เมื่อ destroy$ emit ค่า
      takeUntil(this.destroy$)
    ).subscribe(val => {
      this.counter = val;
      console.log('Counter:', val);
    });
  }

  ngOnDestroy() {
    // emit ค่าและ complete subject เมื่อ component ถูกทำลาย
    this.destroy$.next();
    this.destroy$.complete();
    console.log('Component destroyed, resources cleaned up');
  }
}
```

### แนวทางอื่นๆ สำหรับ Cleanup

```typescript
// 1. operators ที่ complete อัตโนมัติ
import { from, interval } from 'rxjs';
import { take, first, last } from 'rxjs/operators';

// take(n) - complete หลังจาก emit n ครั้ง
interval(1000).pipe(take(3)).subscribe({
  next: val => console.log('take(3):', val),
  complete: () => console.log('take(3) completed automatically')
});

// first() - รับค่าแรกแล้ว complete
interval(1000).pipe(first()).subscribe({
  next: val => console.log('first():', val),
  complete: () => console.log('first() completed automatically')
});

// 2. ใช้ async pipe ใน template (Angular)
// Angular จะจัดการ subscription และ unsubscription ให้อัตโนมัติ
/*
Template:
<div *ngIf="data$ | async as data">
  {{data}}
</div>

Component:
data$ = this.http.get('/api/data');
*/
```

## ⚡ การประยุกต์ใช้ RxJS ใน Angular

### 1. HTTP Requests

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError, of } from 'rxjs';
import { catchError, retry, map, shareReplay } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'https://api.example.com/users';
  
  // Cache results
  private users$ = this.http.get<User[]>(this.apiUrl).pipe(
    retry(2),
    catchError(this.handleError),
    shareReplay(1) // Cache ผลลัพธ์สำหรับ subscribers หลายตัว
  );
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.users$;
  }
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }
  
  createUser(user: Partial<User>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user).pipe(
      catchError(this.handleError)
    );
  }
  
  private handleError(error: HttpErrorResponse) {
    if (error.status === 0) {
      // Client-side error (network issue)
      console.error('Client-side error:', error.error);
    } else {
      // Server-side error
      console.error(
        `Backend returned code ${error.status}, body was:`, 
        error.error
      );
    }
    return throwError(() => new Error('Something went wrong; please try again later.'));
  }
}

interface User {
  id: number;
  name: string;
  email: string;
}
```

### 2. Form Handling with RxJS

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';
import { debounceTime, distinctUntilChanged, switchMap, filter } from 'rxjs/operators';

@Component({
  selector: 'app-search',
  template: `
    <form [formGroup]="searchForm">
      <input type="text" formControlName="query" placeholder="Search...">
    </form>
    
    <div *ngIf="loading">Loading...</div>
    <div *ngIf="results">
      <div *ngFor="let result of results">{{result.name}}</div>
    </div>
  `
})
export class SearchComponent implements OnInit {
  searchForm: FormGroup;
  loading = false;
  results: any[] = [];
  
  constructor(
    private fb: FormBuilder,
    private searchService: SearchService
  ) {
    this.searchForm = this.fb.group({
      query: ['']
    });
  }
  
  ngOnInit() {
    // Reactive search with typeahead
    this.searchForm.get('query')!.valueChanges.pipe(
      debounceTime(300), // รอให้หยุดพิมพ์
      distinctUntilChanged(), // กรองค่าที่ซ้ำกัน
      filter(query => query.length > 2), // ต้องมีตัวอักษรอย่างน้อย 3 ตัว
      switchMap(query => {
        this.loading = true;
        return this.searchService.search(query).pipe(
          catchError(err => {
            console.error('Search error:', err);
            return of([]);
          })
        );
      })
    ).subscribe(results => {
      this.results = results;
      this.loading = false;
    });
  }
}
```

### 3. State Management with RxJS

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { map } from 'rxjs/operators';

interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  loading: boolean;
  error: string | null;
}

@Injectable({
  providedIn: 'root'
})
export class CartService {
  // private state
  private state = new BehaviorSubject<CartState>({
    items: [],
    loading: false,
    error: null
  });
  
  // public observables (read-only)
  readonly cart$ = this.state.asObservable();
  readonly items$ = this.cart$.pipe(map(state => state.items));
  readonly totalItems$ = this.items$.pipe(
    map(items => items.reduce((total, item) => total + item.quantity, 0))
  );
  readonly totalPrice$ = this.items$.pipe(
    map(items => items.reduce((total, item) => total + item.price * item.quantity, 0))
  );
  readonly loading$ = this.cart$.pipe(map(state => state.loading));
  readonly error$ = this.cart$.pipe(map(state => state.error));
  
  // state mutation methods
  addItem(item: CartItem) {
    const currentState = this.state.value;
    const existingItemIndex = currentState.items.findIndex(i => i.id === item.id);
    
    let newItems: CartItem[];
    
    if (existingItemIndex >= 0) {
      // Update existing item
      newItems = [...currentState.items];
      newItems[existingItemIndex] = {
        ...newItems[existingItemIndex],
        quantity: newItems[existingItemIndex].quantity + 1
      };
    } else {
      // Add new item
      newItems = [...currentState.items, { ...item, quantity: 1 }];
    }
    
    this.state.next({
      ...currentState,
      items: newItems
    });
  }
  
  removeItem(id: number) {
    const currentState = this.state.value;
    this.state.next({
      ...currentState,
      items: currentState.items.filter(item => item.id !== id)
    });
  }
  
  updateQuantity(id: number, quantity: number) {
    const currentState = this.state.value;
    this.state.next({
      ...currentState,
      items: currentState.items.map(item => 
        item.id === id ? { ...item, quantity } : item
      )
    });
  }
  
  clearCart() {
    const currentState = this.state.value;
    this.state.next({
      ...currentState,
      items: []
    });
  }
  
  setLoading(loading: boolean) {
    const currentState = this.state.value;
    this.state.next({
      ...currentState,
      loading,
      error: loading ? null : currentState.error
    });
  }
  
  setError(error: string | null) {
    const currentState = this.state.value;
    this.state.next({
      ...currentState,
      error
    });
  }
}
```

### 4. Route Params Handling

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, ParamMap, Router } from '@angular/router';
import { Observable, combineLatest } from 'rxjs';
import { switchMap, tap, filter } from 'rxjs/operators';

@Component({
  selector: 'app-product-details',
  template: `
    <div *ngIf="product$ | async as product; else loading">
      <h1>{{product.name}}</h1>
      <p>{{product.description}}</p>
      <p>Price: {{product.price | currency}}</p>
      
      <div class="reviews">
        <h2>Reviews ({{reviews.length}})</h2>
        <div *ngFor="let review of reviews">
          <p>{{review.text}} - {{review.user}}</p>
        </div>
      </div>
    </div>
    
    <ng-template #loading>Loading...</ng-template>
  `
})
export class ProductDetailsComponent implements OnInit {
  product$!: Observable<any>;
  reviews: any[] = [];

  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private productService: ProductService
  ) {}

  ngOnInit() {
    // จัดการกับ params และ queryParams พร้อมกัน
    this.product$ = combineLatest([
      this.route.paramMap,
      this.route.queryParamMap
    ]).pipe(
      tap(([params, queryParams]) => {
        console.log('Route params:', params);
        console.log('Query params:', queryParams);
      }),
      filter(([params]) => params.has('id')), // ตรวจสอบว่ามี id
      switchMap(([params, queryParams]) => {
        const id = Number(params.get('id'));
        const showDetails = queryParams.get('details') === 'true';
        
        // ส่ง parameters ให้ service
        return this.productService.getProduct(id, showDetails);
      }),
      tap(product => {
        if (!product) {
          // ถ้าไม่พบ product ให้ redirect ไปหน้า not-found
          this.router.navigate(['/not-found']);
        }
      })
    );
    
    // แยก subscription สำหรับ reviews
    this.route.paramMap.pipe(
      switchMap(params => {
        const id = Number(params.get('id'));
        return this.productService.getProductReviews(id);
      })
    ).subscribe(reviews => {
      this.reviews = reviews;
    });
  }
}
```

## 🎯 แบบฝึกหัด

### Exercise 1: สร้าง Observable จากหลายแหล่ง

```typescript
/*
1. สร้าง Observable จากการกดปุ่ม "Click me"
2. สร้าง Observable จากการพิมพ์ใน input field
3. รวม Observables ทั้งสองด้วย merge operator
4. แสดงผลทั้งสองใน log
*/
```

### Exercise 2: ทำระบบ Autocomplete Search

```typescript
/*
1. สร้าง form ที่มี input field สำหรับค้นหา
2. ใช้ valueChanges จาก FormControl
3. ใช้ operators: debounceTime, distinctUntilChanged, filter, switchMap
4. เชื่อมต่อกับ API และแสดงผลลัพธ์
*/
```

### Exercise 3: สร้าง Custom Operator

```typescript
/*
1. สร้าง custom operator ชื่อ "logAndContinue"
2. Operator ต้องทำการ log ค่าแต่ละ emission และส่งต่อไปยัง stream
3. นำไปใช้กับ Observable อื่น
*/
```

## 🧪 Quiz

### Question 1
RxJS ใช้สำหรับอะไร:
- a) สร้าง UI components
- b) จัดการ asynchronous streams of data
- c) ติดต่อกับ database โดยตรง
- d) แสดงผล animation

### Question 2
Operator ใดที่ใช้ในการแปลงค่าแต่ละ emission ใน Observable:
- a) subscribe
- b) map
- c) filter
- d) catchError

### Question 3
Subject และ Observable ต่างกันอย่างไร:
- a) Subject สามารถเป็นทั้ง Observable และ Observer
- b) Subject ทำงานแบบ synchronous เท่านั้น
- c) Observable สามารถ multicast ได้เท่านั้น
- d) Subject ไม่สามารถ complete ได้

**คำตอบ: 1-b, 2-b, 3-a**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🌟 RxJS Fundamentals**
1. **Observables** - โครงสร้างพื้นฐานสำหรับจัดการข้อมูลแบบ asynchronous
2. **Operators** - เครื่องมือสำหรับแปลง กรอง และรวม Observable streams
3. **Subjects** - Observable ที่สามารถ multicast ได้และมีหลายประเภท

### **🚀 RxJS ใน Angular**
1. **HTTP Requests** - จัดการ API calls และ error handling
2. **Forms** - reactive forms และ valueChanges
3. **State Management** - จัดการ state ด้วย BehaviorSubject
4. **Router Events** - ตรวจจับและตอบสนองต่อการเปลี่ยนแปลง route

### **🔧 Best Practices**
1. **Memory Management** - ป้องกัน memory leaks ด้วย unsubscribe หรือ takeUntil
2. **Error Handling** - จัดการ errors ด้วย catchError, retry
3. **Combining Streams** - เลือกใช้ combineLatest, merge, forkJoin ตามความเหมาะสม

ใน**บทถัดไป** เราจะเรียนรู้เกี่ยวกับการจัดการ state ใน Angular applications

---

**[⬅️ บทที่ 7: HTTP Client](./07-http-client.md) | [➡️ บทที่ 9: State Management](./09-state-management.md)**
