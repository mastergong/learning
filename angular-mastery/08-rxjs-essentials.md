# 08 — RxJS Essentials for Angular

## Observables & Operators
```ts
const src$ = from([1,2,3]).pipe(map(x => x*2), filter(x => x>2));
src$.subscribe(console.log);
```

## HttpClient returns Observables
- Combine with `switchMap`, `mergeMap`, `concatMap` based on concurrency needs

```ts
searchTerms.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.http.get<User[]>(`/api/users?q=${term}`))
).subscribe()
```

## Subjects & Multicasting
```ts
const bus$ = new Subject<string>();
bus$.subscribe(v => console.log('A', v));
bus$.subscribe(v => console.log('B', v));
bus$.next('Hello');
```

## Error handling
```ts
obs$.pipe(catchError(err => { this.logger.log(err); return EMPTY; })).subscribe();
```

## Memory leaks & takeUntil
```ts
private destroy$ = new Subject<void>();

ngOnInit(){
  this.obs$.pipe(takeUntil(this.destroy$)).subscribe();
}
ngOnDestroy(){ this.destroy$.next(); this.destroy$.complete(); }
```
