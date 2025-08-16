# 05 — Routing & Navigation

## Basic Routes with Lazy Loading
```ts
// app/routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', loadComponent: () => import('./home.component').then(m => m.HomeComponent) },
  { path: 'about', loadComponent: () => import('./about.component').then(m => m.AboutComponent) },
  { path: '**', redirectTo: '' }
];
```

## Router Links and Parameters
```html
<a [routerLink]="['/product', id]">View</a>
```

```ts
// product.component.ts
import { Component } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({ selector: 'product-page', standalone: true, template: `ID: {{id}}` })
export class ProductComponent {
  id = this.route.snapshot.paramMap.get('id');
  constructor(private route: ActivatedRoute) {}
}
```

## Guards and Resolvers (functional)
```ts
// auth.guard.ts
import { CanActivateFn, Router } from '@angular/router';

export const authGuard: CanActivateFn = (route, state) => {
  const loggedIn = !!localStorage.getItem('token');
  if (!loggedIn) {
    const r = inject(Router);
    r.navigate(['/']);
    return false;
  }
  return true;
};
```

```ts
// data.resolver.ts
import { ResolveFn } from '@angular/router';
import { HttpClient } from '@angular/common/http';

export const dataResolver: ResolveFn<any> = () => {
  const http = inject(HttpClient);
  return http.get('/api/data');
};
```

Attach to routes:
```ts
{ path: 'secure', canActivate: [authGuard], loadComponent: () => import('./secure.component').then(m => m.SecureComponent) }
{ path: 'with-data', resolve: { data: dataResolver }, loadComponent: () => import('./with-data.component').then(m => m.WithDataComponent) }
```
