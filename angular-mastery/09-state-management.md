# 09 — State Management

## Service-based state (simple and effective)
```ts
@Injectable({ providedIn: 'root' })
export class TodosStore {
  private readonly _todos = signal<Todo[]>([]);
  readonly todos = computed(() => this._todos());

  add(todo: Todo){ this._todos.update(xs => [...xs, todo]); }
  toggle(id: number){ this._todos.update(xs => xs.map(t => t.id===id ? {...t, done: !t.done} : t)); }
}
```

## NgRx (brief intro)
- Actions, Reducers, Selectors, Effects
- Use for complex apps with many cross-feature interactions

## Component-store (scoped state)
- Lightweight state per feature
