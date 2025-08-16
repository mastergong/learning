# 11 — Advanced Performance

- ChangeDetectionStrategy.OnPush for pure, immutable inputs
- trackBy in *ngFor
- Lazy load routes and components
- Avoid heavy work in templates; memoize with pure pipes/computed signals
- Prefer `async` pipe or signals over manual subscribe

```ts
@Component({
  selector: 'perf-list',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let item of items; trackBy: trackId">{{item.name}}</div>
  `
})
export class PerfListComponent {
  items: Item[] = [];
  trackId(_: number, x: Item){ return x.id; }
}
```

Signals overview: prefer signals for local reactive state to reduce subscription overhead.
