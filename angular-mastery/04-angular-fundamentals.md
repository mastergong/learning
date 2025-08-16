# 04 — Angular Fundamentals

## Standalone Components and Bootstrapping
```ts
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { routes } from './app/routes';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(),
    provideRouter(routes)
  ]
});
```

```ts
// app/app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <h1>{{ title }}</h1>
    <a routerLink="/">Home</a> | <a routerLink="/about">About</a>
    <router-outlet></router-outlet>
  `,
})
export class AppComponent { title = 'Angular Fundamentals'; }
```

## Template Syntax Essentials
- Interpolation: `{{value}}`
- Property binding: `[src]="imgUrl"`
- Event binding: `(click)="onClick()"`
- Two-way binding: `[(ngModel)]="name"` (add `FormsModule` or standalone import)
- Structural directives: `*ngIf`, `*ngFor`

```ts
@Component({
  selector: 'counter-cmp',
  standalone: true,
  template: `
    <button (click)="dec()">-</button>
    <span>{{count}}</span>
    <button (click)="inc()">+</button>
  `
})
export class CounterComponent {
  count = 0;
  inc(){ this.count++; }
  dec(){ this.count--; }
}
```

## Services and Dependency Injection
```ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class Logger {
  log(message: string){ console.log('[LOG]', message); }
}

@Component({
  selector: 'home-cmp',
  standalone: true,
  template: `<button (click)="hello()">Hello</button>`
})
export class HomeComponent {
  constructor(private logger: Logger) {}
  hello(){ this.logger.log('Hi'); }
}
```

## Pipes and Custom Pipes
```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({name: 'ellipsis', standalone: true})
export class EllipsisPipe implements PipeTransform {
  transform(value: string, max = 10){
    return value.length > max ? value.slice(0,max) + '…' : value;
  }
}
```
