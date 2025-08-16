# 13 — Architecture Best Practices

## Feature-based structure (recommended)
```
src/
  app/
    features/
      products/
        product.routes.ts
        product.page.ts
        product.service.ts
        product.store.ts
      orders/
    shared/
      ui/
      util/
      data-access/
```

- Co-locate feature components/services/state/tests
- Shared for cross-feature utilities and UI
- Avoid deep relative imports; use tsconfig paths

## Smart vs Presentational components
- Smart: fetch/state/route; Presentational: display via @Input/@Output

## API boundaries
- Keep domain logic in services; make components thin

## Reusability
- Use standalone components and re-exports from shared modules
