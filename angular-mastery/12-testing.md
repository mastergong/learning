# 12 — Testing Angular

## Unit testing a service
```ts
describe('CounterService', () => {
  it('increments', () => {
    const s = new CounterService();
    s.inc();
    expect(s.value()).toBe(1);
  });
});
```

## Component testing (TestBed)
```ts
beforeEach(async () => {
  await TestBed.configureTestingModule({
    imports: [CounterComponent]
  }).compileComponents();
});

it('renders count', () => {
  const fixture = TestBed.createComponent(CounterComponent);
  fixture.componentInstance.count.set(2);
  fixture.detectChanges();
  expect(fixture.nativeElement.textContent).toContain('2');
});
```

## HTTP testing
- Use HttpClientTestingModule and HttpTestingController
