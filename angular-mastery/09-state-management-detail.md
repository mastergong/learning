# บทที่ 9: State Management ใน Angular

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจรูปแบบต่างๆ ของ state management ใน Angular
- พัฒนา state management solution แบบ service-based
- เข้าใจและใช้งาน signals สำหรับจัดการ state
- ประยุกต์ใช้ NgRx เพื่อจัดการ state ขนาดใหญ่
- เลือกใช้วิธีจัดการ state ที่เหมาะสมกับโปรเจกต์
- จัดการกับ side effects และ asynchronous operations

## 🧩 State Management คืออะไร?

State management คือการจัดการข้อมูลแบบมีโครงสร้าง เพื่อควบคุมสถานะของแอปพลิเคชันในแต่ละช่วงเวลา

### ประเภทของ State ใน Angular Application

1. **Local Component State** - สถานะที่จัดการภายใน component
2. **Shared State** - สถานะที่ใช้ร่วมกันระหว่าง components
3. **Application State** - สถานะระดับแอปพลิเคชัน (เช่น ข้อมูลผู้ใช้, ธีม, การตั้งค่า)
4. **Router State** - สถานะที่เกี่ยวข้องกับ navigation
5. **Form State** - สถานะของฟอร์มและข้อมูล input

## 💡 Service-Based State Management

เป็นวิธีจัดการ state ที่ง่ายและใช้ได้กับโปรเจกต์ขนาดเล็กถึงขนาดกลาง โดยใช้ Angular services และ RxJS

### 1. Service with RxJS BehaviorSubject

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { map } from 'rxjs/operators';

interface TodoState {
  todos: Todo[];
  loading: boolean;
  error: string | null;
}

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  // Private state
  private stateSubject = new BehaviorSubject<TodoState>({
    todos: [],
    loading: false,
    error: null
  });
  
  // Expose state as observable (read-only)
  private state$ = this.stateSubject.asObservable();
  
  // Selectors
  todos$ = this.state$.pipe(map(state => state.todos));
  loading$ = this.state$.pipe(map(state => state.loading));
  error$ = this.state$.pipe(map(state => state.error));
  completedTodos$ = this.todos$.pipe(
    map(todos => todos.filter(todo => todo.completed))
  );
  incompleteTodos$ = this.todos$.pipe(
    map(todos => todos.filter(todo => !todo.completed))
  );
  
  // Helper to get current state snapshot
  private get state(): TodoState {
    return this.stateSubject.value;
  }
  
  // State manipulation methods
  addTodo(text: string): void {
    this.stateSubject.next({
      ...this.state,
      todos: [
        ...this.state.todos,
        { id: Date.now(), text, completed: false }
      ]
    });
  }
  
  toggleTodo(id: number): void {
    this.stateSubject.next({
      ...this.state,
      todos: this.state.todos.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    });
  }
  
  removeTodo(id: number): void {
    this.stateSubject.next({
      ...this.state,
      todos: this.state.todos.filter(todo => todo.id !== id)
    });
  }
  
  loadTodos(): void {
    // Set loading state
    this.stateSubject.next({
      ...this.state,
      loading: true,
      error: null
    });
    
    // Simulate API call
    setTimeout(() => {
      try {
        const todos = [
          { id: 1, text: 'Learn Angular', completed: true },
          { id: 2, text: 'Learn RxJS', completed: false },
          { id: 3, text: 'Build an app', completed: false }
        ];
        
        this.stateSubject.next({
          todos,
          loading: false,
          error: null
        });
      } catch (err) {
        this.stateSubject.next({
          ...this.state,
          loading: false,
          error: 'Failed to load todos'
        });
      }
    }, 1000);
  }
}
```

### 2. การใช้งาน Service-Based State ใน Component

```typescript
import { Component, OnInit } from '@angular/core';
import { Observable } from 'rxjs';
import { TodoService } from './todo.service';

@Component({
  selector: 'app-todo-list',
  template: `
    <div class="todo-container">
      <h1>Todo List</h1>
      
      <div *ngIf="loading$ | async" class="loading">Loading...</div>
      <div *ngIf="error$ | async as error" class="error">{{error}}</div>
      
      <div class="add-todo">
        <input #todoInput placeholder="Add a new task..." />
        <button (click)="addTodo(todoInput.value); todoInput.value = ''">Add</button>
      </div>
      
      <h2>Tasks ({{(todos$ | async)?.length || 0}})</h2>
      
      <div class="todo-list">
        <div *ngFor="let todo of todos$ | async" class="todo-item">
          <input 
            type="checkbox" 
            [checked]="todo.completed" 
            (change)="toggleTodo(todo.id)" 
          />
          <span [class.completed]="todo.completed">{{todo.text}}</span>
          <button (click)="removeTodo(todo.id)" class="remove-btn">X</button>
        </div>
        
        <p *ngIf="(todos$ | async)?.length === 0">
          No tasks yet. Add one above!
        </p>
      </div>
      
      <div class="summary">
        <p>Completed: {{(completedCount$ | async) || 0}}</p>
        <p>Remaining: {{(incompleteCount$ | async) || 0}}</p>
        <button (click)="loadTodos()">Refresh</button>
      </div>
    </div>
  `,
  styles: [`
    .todo-container {
      max-width: 500px;
      margin: 0 auto;
      padding: 20px;
    }
    
    .add-todo {
      display: flex;
      margin-bottom: 20px;
    }
    
    .add-todo input {
      flex: 1;
      padding: 8px;
      font-size: 16px;
    }
    
    .add-todo button {
      padding: 8px 16px;
      background: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
    }
    
    .todo-item {
      display: flex;
      align-items: center;
      padding: 8px 0;
      border-bottom: 1px solid #eee;
    }
    
    .completed {
      text-decoration: line-through;
      color: #888;
    }
    
    .remove-btn {
      margin-left: auto;
      background: #f44336;
      color: white;
      border: none;
      padding: 2px 6px;
      cursor: pointer;
    }
    
    .loading {
      padding: 10px;
      background: #e3f2fd;
      margin-bottom: 10px;
    }
    
    .error {
      padding: 10px;
      background: #ffebee;
      color: #c62828;
      margin-bottom: 10px;
    }
    
    .summary {
      margin-top: 20px;
      padding-top: 10px;
      border-top: 1px solid #eee;
    }
  `]
})
export class TodoListComponent implements OnInit {
  todos$: Observable<Todo[]>;
  loading$: Observable<boolean>;
  error$: Observable<string | null>;
  completedCount$: Observable<number>;
  incompleteCount$: Observable<number>;
  
  constructor(private todoService: TodoService) {
    this.todos$ = this.todoService.todos$;
    this.loading$ = this.todoService.loading$;
    this.error$ = this.todoService.error$;
    this.completedCount$ = this.todoService.completedTodos$.pipe(
      map(todos => todos.length)
    );
    this.incompleteCount$ = this.todoService.incompleteTodos$.pipe(
      map(todos => todos.length)
    );
  }
  
  ngOnInit(): void {
    this.loadTodos();
  }
  
  addTodo(text: string): void {
    if (text.trim()) {
      this.todoService.addTodo(text);
    }
  }
  
  toggleTodo(id: number): void {
    this.todoService.toggleTodo(id);
  }
  
  removeTodo(id: number): void {
    this.todoService.removeTodo(id);
  }
  
  loadTodos(): void {
    this.todoService.loadTodos();
  }
}
```

### 3. ข้อดีและข้อควรระวังของ Service-Based State Management

#### ข้อดี
- ง่ายต่อการเริ่มต้นใช้งาน - ไม่ต้องติดตั้ง libraries เพิ่มเติม
- ใช้ Angular DI system ที่มีอยู่แล้ว
- เข้ากันได้ดีกับ Angular lifecycles และ patterns
- ไม่มี boilerplate code มากเกินไป

#### ข้อควรระวัง
- อาจมีปัญหาเรื่องความสอดคล้องของ state เมื่อแอปพลิเคชันเติบโต
- ไม่มี devtools สำหรับดู state ย้อนหลัง
- ต้องระวังเรื่อง immutability เมื่อ update state
- ตรวจสอบ memory leaks จาก subscriptions

## ⚡ Angular Signals (Angular 16+)

Signals เป็น primitives สำหรับ reactive programming แบบ fine-grained ที่แนะนำใน Angular 16+

### 1. Signals พื้นฐาน

```typescript
import { Component, signal, computed, effect, Signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h1>Counter: {{ count() }}</h1>
    <p>Doubled: {{ doubled() }}</p>
    <p>Squared: {{ squared() }}</p>
    
    <button (click)="increment()">Increment</button>
    <button (click)="decrement()">Decrement</button>
    <button (click)="reset()">Reset</button>
  `,
})
export class CounterComponent {
  // สร้าง writable signal
  count = signal(0);
  
  // สร้าง computed signals (read-only signals ที่คำนวณจาก signals อื่น)
  doubled = computed(() => this.count() * 2);
  squared = computed(() => this.count() ** 2);
  
  constructor() {
    // สร้าง effect (ทำงานทุกครั้งที่ dependencies เปลี่ยน)
    effect(() => {
      console.log(`Count changed to ${this.count()}`);
      console.log(`Doubled value is ${this.doubled()}`);
    });
  }
  
  increment(): void {
    // Update signal ด้วย update function
    this.count.update(value => value + 1);
  }
  
  decrement(): void {
    // ใช้ update function เพื่อกำหนดค่าใหม่โดยอ้างอิงจากค่าปัจจุบัน
    this.count.update(c => Math.max(0, c - 1));
  }
  
  reset(): void {
    // ตั้งค่าโดยตรงด้วย set
    this.count.set(0);
  }
}
```

### 2. Signal-Based Store

```typescript
import { Injectable, signal, computed } from '@angular/core';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
  error: string | null;
}

@Injectable({
  providedIn: 'root'
})
export class TodoStore {
  // Private state
  #state = signal<TodoState>({
    todos: [],
    filter: 'all',
    loading: false,
    error: null
  });
  
  // Selectors (computed signals)
  todos = computed(() => this.#state().todos);
  filter = computed(() => this.#state().filter);
  loading = computed(() => this.#state().loading);
  error = computed(() => this.#state().error);
  
  filteredTodos = computed(() => {
    const todos = this.todos();
    const filter = this.filter();
    
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  });
  
  todoCount = computed(() => this.todos().length);
  activeCount = computed(() => this.todos().filter(todo => !todo.completed).length);
  completedCount = computed(() => this.todos().filter(todo => todo.completed).length);
  allCompleted = computed(() => this.todos().length > 0 && this.activeCount() === 0);
  
  // Actions
  addTodo(text: string): void {
    const newTodo: Todo = {
      id: Date.now(),
      text: text.trim(),
      completed: false
    };
    
    this.#state.update(state => ({
      ...state,
      todos: [...state.todos, newTodo]
    }));
  }
  
  removeTodo(id: number): void {
    this.#state.update(state => ({
      ...state,
      todos: state.todos.filter(todo => todo.id !== id)
    }));
  }
  
  toggleTodo(id: number): void {
    this.#state.update(state => ({
      ...state,
      todos: state.todos.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    }));
  }
  
  setFilter(filter: 'all' | 'active' | 'completed'): void {
    this.#state.update(state => ({ ...state, filter }));
  }
  
  clearCompleted(): void {
    this.#state.update(state => ({
      ...state,
      todos: state.todos.filter(todo => !todo.completed)
    }));
  }
  
  toggleAll(): void {
    const allCompleted = this.allCompleted();
    
    this.#state.update(state => ({
      ...state,
      todos: state.todos.map(todo => ({
        ...todo,
        completed: !allCompleted
      }))
    }));
  }
  
  loadTodos(): void {
    // Set loading state
    this.#state.update(state => ({
      ...state,
      loading: true,
      error: null
    }));
    
    // Simulate API call
    setTimeout(() => {
      try {
        const todos = [
          { id: 1, text: 'Learn Angular Signals', completed: false },
          { id: 2, text: 'Build signal-based app', completed: false },
          { id: 3, text: 'Share knowledge', completed: false }
        ];
        
        this.#state.update(state => ({
          ...state,
          todos,
          loading: false
        }));
      } catch (error) {
        this.#state.update(state => ({
          ...state,
          loading: false,
          error: 'Failed to load todos'
        }));
      }
    }, 1000);
  }
}
```

### 3. การใช้งาน Signal Store ใน Component

```typescript
import { Component, OnInit } from '@angular/core';
import { TodoStore } from './todo-store.service';

@Component({
  selector: 'app-todo-app',
  template: `
    <div class="todo-app">
      <h1>Todos (Signal-based)</h1>
      
      <div *ngIf="store.loading()" class="loading">Loading...</div>
      <div *ngIf="store.error()" class="error">{{ store.error() }}</div>
      
      <div class="add-todo">
        <input #todoInput placeholder="What needs to be done?" 
               (keyup.enter)="addTodo(todoInput.value); todoInput.value = ''" />
      </div>
      
      <div *ngIf="store.todoCount() > 0">
        <div class="toggle-all">
          <input type="checkbox" id="toggle-all" [checked]="store.allCompleted()" (change)="store.toggleAll()" />
          <label for="toggle-all">Mark all as complete</label>
        </div>
        
        <ul class="todo-list">
          @for (todo of store.filteredTodos(); track todo.id) {
            <li [class.completed]="todo.completed">
              <div class="todo-item">
                <input type="checkbox" [checked]="todo.completed" (change)="store.toggleTodo(todo.id)" />
                <span>{{ todo.text }}</span>
                <button (click)="store.removeTodo(todo.id)" class="delete-btn">×</button>
              </div>
            </li>
          }
        </ul>
        
        <div class="footer">
          <span class="todo-count">
            <strong>{{ store.activeCount() }}</strong> items left
          </span>
          
          <div class="filters">
            <button [class.selected]="store.filter() === 'all'" 
                    (click)="store.setFilter('all')">All</button>
            <button [class.selected]="store.filter() === 'active'" 
                    (click)="store.setFilter('active')">Active</button>
            <button [class.selected]="store.filter() === 'completed'" 
                    (click)="store.setFilter('completed')">Completed</button>
          </div>
          
          <button *ngIf="store.completedCount() > 0" 
                  (click)="store.clearCompleted()" 
                  class="clear-completed">
            Clear completed
          </button>
        </div>
      </div>
    </div>
  `,
  styles: [/* ... */]
})
export class TodoAppComponent implements OnInit {
  constructor(public store: TodoStore) {}
  
  ngOnInit() {
    this.store.loadTodos();
  }
  
  addTodo(text: string) {
    if (text.trim()) {
      this.store.addTodo(text);
    }
  }
}
```

### 4. ข้อดีของ Signals

- **Fine-grained reactivity** - อัพเดทเฉพาะส่วนที่เปลี่ยนแปลง
- **Type safety** - TypeScript integration ที่ดีขึ้น
- **เข้าใจง่าย** - API ที่เรียบง่าย (set/update/compute)
- **ประสิทธิภาพ** - ลด change detection cycles
- **Integration กับ Angular** - ทำงานร่วมกับ Angular template ได้ดี
- **พร้อมสำหรับอนาคต** - เป็นทิศทางของ Angular ในอนาคต

## 🧰 NgRx - State Management ระดับ Enterprise

NgRx เป็น state management library ที่ใช้ Redux pattern สำหรับ Angular applications ขนาดใหญ่

### 1. NgRx Architecture

- **Store** - เก็บ state แบบ immutable ของทั้งแอปพลิเคชัน
- **Actions** - อธิบายเหตุการณ์ที่เกิดขึ้น
- **Reducers** - Pure functions ที่เปลี่ยน state ตาม action
- **Selectors** - ใช้เลือก/query ข้อมูลจาก store
- **Effects** - จัดการ side effects และ async operations
- **Entity** - จัดการ collections ของ entities

### 2. ติดตั้ง NgRx

```bash
# ติดตั้ง NgRx store
ng add @ngrx/store

# ติดตั้ง NgRx DevTools
ng add @ngrx/store-devtools

# ติดตั้ง NgRx effects
ng add @ngrx/effects

# ติดตั้ง NgRx entity
ng add @ngrx/entity
```

### 3. ตัวอย่างการใช้งาน NgRx

#### Actions

```typescript
import { createAction, props } from '@ngrx/store';

export const loadTodos = createAction('[Todo] Load Todos');
export const loadTodosSuccess = createAction(
  '[Todo] Load Todos Success',
  props<{ todos: Todo[] }>()
);
export const loadTodosFailure = createAction(
  '[Todo] Load Todos Failure',
  props<{ error: string }>()
);

export const addTodo = createAction(
  '[Todo] Add Todo',
  props<{ text: string }>()
);
export const addTodoSuccess = createAction(
  '[Todo] Add Todo Success',
  props<{ todo: Todo }>()
);

export const toggleTodo = createAction(
  '[Todo] Toggle Todo',
  props<{ id: number }>()
);

export const deleteTodo = createAction(
  '[Todo] Delete Todo',
  props<{ id: number }>()
);

export const setFilter = createAction(
  '[Todo] Set Filter',
  props<{ filter: 'all' | 'active' | 'completed' }>()
);
```

#### Reducer

```typescript
import { createReducer, on } from '@ngrx/store';
import * as TodoActions from './todo.actions';

export interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
  error: string | null;
}

export const initialState: TodoState = {
  todos: [],
  filter: 'all',
  loading: false,
  error: null
};

export const todoReducer = createReducer(
  initialState,
  
  // Load todos
  on(TodoActions.loadTodos, state => ({
    ...state,
    loading: true,
    error: null
  })),
  
  on(TodoActions.loadTodosSuccess, (state, { todos }) => ({
    ...state,
    todos,
    loading: false
  })),
  
  on(TodoActions.loadTodosFailure, (state, { error }) => ({
    ...state,
    error,
    loading: false
  })),
  
  // Add todo
  on(TodoActions.addTodoSuccess, (state, { todo }) => ({
    ...state,
    todos: [...state.todos, todo]
  })),
  
  // Toggle todo
  on(TodoActions.toggleTodo, (state, { id }) => ({
    ...state,
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),
  
  // Delete todo
  on(TodoActions.deleteTodo, (state, { id }) => ({
    ...state,
    todos: state.todos.filter(todo => todo.id !== id)
  })),
  
  // Set filter
  on(TodoActions.setFilter, (state, { filter }) => ({
    ...state,
    filter
  }))
);
```

#### Selectors

```typescript
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { TodoState } from './todo.reducer';

export const selectTodoState = createFeatureSelector<TodoState>('todos');

export const selectAllTodos = createSelector(
  selectTodoState,
  state => state.todos
);

export const selectActiveFilter = createSelector(
  selectTodoState,
  state => state.filter
);

export const selectLoading = createSelector(
  selectTodoState,
  state => state.loading
);

export const selectError = createSelector(
  selectTodoState,
  state => state.error
);

export const selectFilteredTodos = createSelector(
  selectAllTodos,
  selectActiveFilter,
  (todos, filter) => {
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  }
);

export const selectTodoCount = createSelector(
  selectAllTodos,
  todos => todos.length
);

export const selectActiveCount = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => !todo.completed).length
);

export const selectCompletedCount = createSelector(
  selectAllTodos,
  todos => todos.filter(todo => todo.completed).length
);
```

#### Effects

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { catchError, map, switchMap, mergeMap } from 'rxjs/operators';
import * as TodoActions from './todo.actions';
import { TodoService } from './todo.service';

@Injectable()
export class TodoEffects {
  loadTodos$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.loadTodos),
    switchMap(() => this.todoService.getTodos().pipe(
      map(todos => TodoActions.loadTodosSuccess({ todos })),
      catchError(error => of(TodoActions.loadTodosFailure({ 
        error: error.message || 'Failed to load todos' 
      })))
    ))
  ));
  
  addTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.addTodo),
    mergeMap(({ text }) => this.todoService.addTodo(text).pipe(
      map(todo => TodoActions.addTodoSuccess({ todo })),
      catchError(error => of(TodoActions.loadTodosFailure({ 
        error: error.message || 'Failed to add todo' 
      })))
    ))
  ));
  
  toggleTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.toggleTodo),
    mergeMap(({ id }) => this.todoService.toggleTodo(id).pipe(
      map(() => TodoActions.loadTodos()),
      catchError(error => of(TodoActions.loadTodosFailure({ 
        error: error.message || 'Failed to toggle todo' 
      })))
    ))
  ));
  
  deleteTodo$ = createEffect(() => this.actions$.pipe(
    ofType(TodoActions.deleteTodo),
    mergeMap(({ id }) => this.todoService.deleteTodo(id).pipe(
      map(() => TodoActions.loadTodos()),
      catchError(error => of(TodoActions.loadTodosFailure({ 
        error: error.message || 'Failed to delete todo' 
      })))
    ))
  ));
  
  constructor(
    private actions$: Actions,
    private todoService: TodoService
  ) {}
}
```

#### ใช้งาน Store ใน Component

```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as TodoActions from './store/todo.actions';
import * as fromTodo from './store/todo.selectors';
import { Todo } from './models/todo.model';

@Component({
  selector: 'app-todo-list',
  template: `
    <div class="todo-container">
      <h1>NgRx Todo App</h1>
      
      <div *ngIf="loading$ | async" class="loading">Loading...</div>
      <div *ngIf="error$ | async as error" class="error">{{ error }}</div>
      
      <div class="add-todo">
        <input #todoInput placeholder="Add a new task..." 
               (keyup.enter)="addTodo(todoInput.value); todoInput.value = ''" />
      </div>
      
      <div class="filters">
        <button (click)="setFilter('all')" 
                [class.active]="(currentFilter$ | async) === 'all'">All</button>
        <button (click)="setFilter('active')" 
                [class.active]="(currentFilter$ | async) === 'active'">Active</button>
        <button (click)="setFilter('completed')" 
                [class.active]="(currentFilter$ | async) === 'completed'">Completed</button>
      </div>
      
      <ul class="todo-list">
        <li *ngFor="let todo of filteredTodos$ | async" 
            [class.completed]="todo.completed">
          <input type="checkbox" [checked]="todo.completed" 
                 (change)="toggleTodo(todo.id)" />
          <span>{{ todo.text }}</span>
          <button (click)="deleteTodo(todo.id)" class="delete">×</button>
        </li>
      </ul>
      
      <div class="footer">
        <span>
          <strong>{{ activeCount$ | async }}</strong> items left
        </span>
      </div>
    </div>
  `,
  styles: [/* ... */]
})
export class TodoListComponent implements OnInit {
  filteredTodos$: Observable<Todo[]>;
  loading$: Observable<boolean>;
  error$: Observable<string | null>;
  currentFilter$: Observable<string>;
  activeCount$: Observable<number>;
  
  constructor(private store: Store) {
    this.filteredTodos$ = this.store.select(fromTodo.selectFilteredTodos);
    this.loading$ = this.store.select(fromTodo.selectLoading);
    this.error$ = this.store.select(fromTodo.selectError);
    this.currentFilter$ = this.store.select(fromTodo.selectActiveFilter);
    this.activeCount$ = this.store.select(fromTodo.selectActiveCount);
  }
  
  ngOnInit(): void {
    this.store.dispatch(TodoActions.loadTodos());
  }
  
  addTodo(text: string): void {
    if (text.trim()) {
      this.store.dispatch(TodoActions.addTodo({ text }));
    }
  }
  
  toggleTodo(id: number): void {
    this.store.dispatch(TodoActions.toggleTodo({ id }));
  }
  
  deleteTodo(id: number): void {
    this.store.dispatch(TodoActions.deleteTodo({ id }));
  }
  
  setFilter(filter: 'all' | 'active' | 'completed'): void {
    this.store.dispatch(TodoActions.setFilter({ filter }));
  }
}
```

### 4. ข้อดีและข้อควรพิจารณาของ NgRx

#### ข้อดี
- **Centralized state** - จัดการ state ทั้งหมดในที่เดียว
- **Time-traveling debugger** - ย้อนดู state history ได้
- **Immutability** - รับประกัน state เป็น immutable
- **Testability** - reducers เป็น pure functions ทดสอบได้ง่าย
- **Middleware** - จัดการ side effects ด้วย Effects

#### ข้อควรพิจารณา
- **Boilerplate** - ต้องเขียนโค้ดเยอะ (Actions, Reducers, Effects, Selectors)
- **Learning curve** - ต้องเรียนรู้เพิ่มเติม
- **Overhead** - อาจมากเกินไปสำหรับแอปพลิเคชันขนาดเล็ก
- **Bundle size** - เพิ่มขนาดของแอปพลิเคชัน

## 🔄 NgRx Component Store

Component Store เป็น state management solution ที่ให้ความสมดุลระหว่าง NgRx และ service-based approach

### 1. ติดตั้ง Component Store

```bash
npm install @ngrx/component-store
```

### 2. ตัวอย่างการใช้งาน Component Store

```typescript
import { Injectable } from '@angular/core';
import { ComponentStore } from '@ngrx/component-store';
import { Observable } from 'rxjs';
import { tap, switchMap, withLatestFrom } from 'rxjs/operators';
import { TodoService } from './todo.service';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodosState {
  todos: Todo[];
  selectedTodoId: number | null;
  loading: boolean;
  error: string | null;
}

@Injectable()
export class TodosStore extends ComponentStore<TodosState> {
  constructor(private todoService: TodoService) {
    super({
      todos: [],
      selectedTodoId: null,
      loading: false,
      error: null
    });
  }
  
  // Selectors
  readonly todos$ = this.select(state => state.todos);
  readonly selectedTodoId$ = this.select(state => state.selectedTodoId);
  readonly loading$ = this.select(state => state.loading);
  readonly error$ = this.select(state => state.error);
  
  readonly selectedTodo$ = this.select(
    this.todos$,
    this.selectedTodoId$,
    (todos, selectedId) => todos.find(todo => todo.id === selectedId) || null
  );
  
  // Updaters (synchronous)
  readonly setLoading = this.updater((state, loading: boolean) => ({
    ...state,
    loading,
    error: loading ? null : state.error
  }));
  
  readonly setError = this.updater((state, error: string | null) => ({
    ...state,
    error,
    loading: false
  }));
  
  readonly setTodos = this.updater((state, todos: Todo[]) => ({
    ...state,
    todos,
    loading: false,
    error: null
  }));
  
  readonly setSelectedTodoId = this.updater((state, id: number | null) => ({
    ...state,
    selectedTodoId: id
  }));
  
  readonly addTodo = this.updater((state, todo: Todo) => ({
    ...state,
    todos: [...state.todos, todo]
  }));
  
  readonly removeTodo = this.updater((state, id: number) => ({
    ...state,
    todos: state.todos.filter(todo => todo.id !== id)
  }));
  
  readonly toggleTodo = this.updater((state, id: number) => ({
    ...state,
    todos: state.todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  }));
  
  // Effects (asynchronous)
  readonly loadTodos = this.effect((trigger$: Observable<void>) => {
    return trigger$.pipe(
      tap(() => this.setLoading(true)),
      switchMap(() => this.todoService.getTodos().pipe(
        tap({
          next: todos => this.setTodos(todos),
          error: err => this.setError(err.message || 'Failed to load todos')
        })
      ))
    );
  });
  
  readonly createTodo = this.effect((text$: Observable<string>) => {
    return text$.pipe(
      withLatestFrom(this.todos$),
      switchMap(([text, todos]) => {
        // ทำ optimistic update
        const newId = todos.length ? Math.max(...todos.map(t => t.id)) + 1 : 1;
        const newTodo = { id: newId, text, completed: false };
        this.addTodo(newTodo);
        
        return this.todoService.addTodo(text).pipe(
          tap({
            error: err => {
              // Rollback หากเกิด error
              this.removeTodo(newId);
              this.setError(err.message || 'Failed to create todo');
            }
          })
        );
      })
    );
  });
  
  readonly deleteTodo = this.effect((id$: Observable<number>) => {
    return id$.pipe(
      switchMap(id => {
        // optimistic update
        const todoToDelete = this.get().todos.find(t => t.id === id);
        this.removeTodo(id);
        
        return this.todoService.deleteTodo(id).pipe(
          tap({
            error: err => {
              // Rollback หากเกิด error
              if (todoToDelete) {
                this.addTodo(todoToDelete);
              }
              this.setError(err.message || 'Failed to delete todo');
            }
          })
        );
      })
    );
  });
  
  readonly toggleTodoComplete = this.effect((id$: Observable<number>) => {
    return id$.pipe(
      switchMap(id => {
        // Optimistic update
        this.toggleTodo(id);
        
        return this.todoService.toggleTodo(id).pipe(
          tap({
            error: err => {
              // Rollback หากเกิด error
              this.toggleTodo(id);
              this.setError(err.message || 'Failed to update todo');
            }
          })
        );
      })
    );
  });
}
```

### 3. ใช้งาน Component Store ใน Component

```typescript
import { Component, OnInit } from '@angular/core';
import { TodosStore } from './todos.store';

@Component({
  selector: 'app-todo-list',
  template: `
    <div class="todo-container">
      <h1>Component Store Todos</h1>
      
      <div *ngIf="todosStore.loading$ | async" class="loading">Loading...</div>
      <div *ngIf="todosStore.error$ | async as error" class="error">{{ error }}</div>
      
      <div class="add-todo">
        <input #todoInput placeholder="Add a new task..." 
               (keyup.enter)="addTodo(todoInput.value); todoInput.value = ''" />
      </div>
      
      <div class="todos">
        <div *ngFor="let todo of todosStore.todos$ | async" class="todo-item">
          <input type="checkbox" [checked]="todo.completed" 
                 (change)="toggleTodo(todo.id)" />
          <span [class.completed]="todo.completed">{{ todo.text }}</span>
          <button (click)="removeTodo(todo.id)" class="delete-btn">×</button>
        </div>
        
        <div *ngIf="(todosStore.todos$ | async)?.length === 0 && !(todosStore.loading$ | async)">
          No todos yet. Add one above!
        </div>
      </div>
      
      <div *ngIf="todosStore.selectedTodo$ | async as selectedTodo" class="details">
        <h3>Selected Todo Details</h3>
        <p><strong>ID:</strong> {{ selectedTodo.id }}</p>
        <p><strong>Text:</strong> {{ selectedTodo.text }}</p>
        <p><strong>Status:</strong> {{ selectedTodo.completed ? 'Completed' : 'Active' }}</p>
        <button (click)="todosStore.setSelectedTodoId(null)">Close</button>
      </div>
    </div>
  `,
  styles: [/* ... */],
  providers: [TodosStore] // Scoped ที่ component นี้และ children
})
export class TodoListComponent implements OnInit {
  constructor(public todosStore: TodosStore) {}
  
  ngOnInit() {
    this.todosStore.loadTodos();
  }
  
  addTodo(text: string) {
    if (text.trim()) {
      this.todosStore.createTodo(text);
    }
  }
  
  toggleTodo(id: number) {
    this.todosStore.toggleTodoComplete(id);
  }
  
  removeTodo(id: number) {
    this.todosStore.deleteTodo(id);
  }
  
  selectTodo(id: number) {
    this.todosStore.setSelectedTodoId(id);
  }
}
```

### 4. ข้อดีของ Component Store

- **Scoped state** - จัดการ state เฉพาะส่วน component tree
- **Less boilerplate** - ไม่ต้องแยก actions, reducers, selectors
- **Effects built-in** - จัดการ side effects ได้ง่าย
- **DI Integration** - ทำงานร่วมกับ Angular DI system
- **ประสิทธิภาพ** - bundle size เล็กกว่า NgRx เต็มรูปแบบ
- **Type safety** - fully typed API

## 🔄 เลือกใช้ State Management Solution ที่เหมาะสม

### 1. เมื่อใดควรใช้แต่ละวิธี

#### Service with RxJS
- แอปพลิเคชันขนาดเล็กถึงขนาดกลาง
- ต้องการ setup ที่ง่าย
- ทีมมีความคุ้นเคยกับ RxJS
- state มีการเปลี่ยนแปลงไม่ซับซ้อน

#### Angular Signals
- แอปพลิเคชันขนาดเล็กถึงขนาดกลาง
- ใช้ Angular 16+
- ต้องการประสิทธิภาพสูงและ fine-grained reactivity
- ต้องการเตรียมพร้อมสำหรับอนาคตของ Angular

#### NgRx
- แอปพลิเคชันขนาดใหญ่หรือซับซ้อน
- มี team ขนาดใหญ่
- ต้องการ strict state management
- ต้องการ DevTools และ time-traveling debugger
- มี side effects ซับซ้อน

#### Component Store
- ต้องการ state management ที่ scoped ใน feature
- ต้องการลด boilerplate ของ NgRx แบบเต็มรูปแบบ
- ต้องการ balance ระหว่างโค้ดที่เป็นระเบียบกับความซับซ้อน

### 2. ตารางเปรียบเทียบ

| Feature | Service + RxJS | Signals | NgRx | Component Store |
|---------|---------------|---------|------|----------------|
| Setup | ง่าย | ง่าย | ซับซ้อน | ปานกลาง |
| Boilerplate | น้อย | น้อยที่สุด | มาก | ปานกลาง |
| DevTools | ไม่มี | ไม่มี (ยังไม่สมบูรณ์) | มี | จำกัด |
| Time-travel debugging | ไม่มี | ไม่มี | มี | ไม่มี |
| Learning curve | ต่ำ (ถ้าคุ้นเคย RxJS) | ต่ำ | สูง | ปานกลาง |
| Bundle size | เล็ก | เล็ก | ใหญ่ | ปานกลาง |
| Scalability | ปานกลาง | ดี | ดีมาก | ดี |
| Side effect management | Manual | Manual | Effects | Effects |
| Testability | ดี | ดี | ดีมาก | ดีมาก |
| TypeScript integration | ดี | ดีมาก | ดี | ดี |

## 🎯 แบบฝึกหัด

### Exercise 1: สร้าง Shopping Cart ด้วย Service-based State

```typescript
// สร้าง Shopping Cart Service ที่สามารถ:
// - เพิ่มสินค้า
// - ลบสินค้า
// - ปรับปรุงจำนวนสินค้า
// - คำนวณราคารวม
// - เคลียร์ตะกร้า
// ใช้ RxJS BehaviorSubject เพื่อให้ components สามารถ subscribe และอัพเดทได้
```

### Exercise 2: สร้าง Counter App ด้วย Signals

```typescript
// สร้าง Counter Component ด้วย Signals ที่มีฟีเจอร์:
// - เพิ่ม/ลด counter
// - Reset counter
// - ตั้งค่า step (เพิ่มครั้งละเท่าไหร่)
// - แสดง computed values (doubled, squared)
// - Log history ของการเปลี่ยนแปลงค่า
```

### Exercise 3: สร้าง Todo App ด้วย NgRx

```typescript
// สร้าง Todo App ด้วย NgRx ที่มี:
// - Actions (load, add, toggle, delete)
// - Reducer
// - Selectors (all, active, completed)
// - Effects (API calls)
// - Component ที่ใช้งาน store
```

## 🧪 Quiz

### Question 1
ข้อใดเป็นข้อดีของการใช้ Service-based state management:
- a) มี DevTools แบบ time-traveling
- b) ไม่ต้องติดตั้ง library เพิ่มเติม
- c) มีโครงสร้างชัดเจนสำหรับ team ขนาดใหญ่
- d) มี built-in effects system

### Question 2
Signals แตกต่างจาก Observable อย่างไร:
- a) Signals ใช้ push-based model ในขณะที่ Observable ใช้ pull-based
- b) Signals มี fine-grained reactivity และตรวจจับการเปลี่ยนแปลงเฉพาะส่วน
- c) Signals ไม่สามารถทำงานกับ asynchronous operations
- d) Signals ทำงานกับ plain JavaScript ในขณะที่ Observable ทำงานกับ TypeScript

### Question 3
NgRx Component Store เหมาะกับกรณีใด:
- a) แอปพลิเคชันที่มี state น้อยมาก
- b) เมื่อต้องการ state management ที่ scope อยู่ในบาง features
- c) เมื่อต้องการ DevTools แบบ time-traveling
- d) เมื่อไม่ต้องการจัดการ side effects

**คำตอบ: 1-b, 2-b, 3-b**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🧩 State Management Patterns**
1. **Service-based state** - ใช้ Angular services และ RxJS
2. **Signals** - reactivity primitive ใหม่ของ Angular
3. **NgRx** - state management แบบ Redux สำหรับแอปพลิเคชันขนาดใหญ่
4. **Component Store** - state management แบบ scoped สำหรับ features

### **🛠️ Implementation Techniques**
1. **Immutability** - การรักษา state ไม่ให้เปลี่ยนแปลงโดยตรง
2. **Selectors** - การเลือก/query ข้อมูลจาก state
3. **Side Effects** - การจัดการกับ async operations
4. **DevTools** - การ debug state

### **🔄 Selection Criteria**
1. **ขนาดของแอปพลิเคชัน** - เลือก solution ตามขนาดและความซับซ้อน
2. **Team Experience** - คำนึงถึงความคุ้นเคยของทีม
3. **Tooling Requirements** - ความต้องการด้าน DevTools และ debugging
4. **Future Proofing** - การเตรียมพร้อมสำหรับอนาคต

ใน**บทถัดไป** เราจะเรียนรู้เกี่ยวกับ Angular Material และ Component Dev Kit (CDK)

---

**[⬅️ บทที่ 8: RxJS Essentials](./08-rxjs-essentials.md) | [➡️ บทที่ 10: Angular Material & CDK](./10-angular-material-cdk.md)**
