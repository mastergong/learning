# 01 — Introduction to Angular

Angular is a batteries-included framework for building scalable, maintainable web apps.

## Key ideas
- TypeScript-first
- Components for UI; Services for logic/state
- Dependency Injection
- RxJS for async streams
- Routing for multi-page SPA
- First-class tooling (CLI)

## Modern Angular highlights
- Standalone components (no NgModule required for most apps)
- Lazy loading by feature
- Signals (reactivity primitive) and OnPush change detection to boost performance

## Minimal standalone component
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'hello-app',
  standalone: true,
  template: `<h1>Hello {{name}}</h1>`,
})
export class HelloComponent {
  name = 'Angular';
}
```

## When to choose Angular
- Enterprise UI at scale, strong conventions
- Teams needing DI, routing, forms, testing out of the box
