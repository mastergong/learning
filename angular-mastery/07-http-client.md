# 07 — HTTP Client & Interceptors

## HttpClient Basics
```ts
@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}
  getUsers(){ return this.http.get<User[]>('/api/users'); }
  createUser(u: User){ return this.http.post<User>('/api/users', u); }
}
```

## Interceptor (Auth + Logging)
```ts
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler){
    const token = localStorage.getItem('token');
    const authReq = token ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }) : req;
    console.time(req.url);
    return next.handle(authReq).pipe(finalize(() => console.timeEnd(req.url)));
  }
}

bootstrapApplication(AppComponent, {
  providers: [ provideHttpClient(withInterceptors([ (req, next) => new AuthInterceptor().intercept(req, next) ])) ]
});
```

## Error Handling + Retry
```ts
this.api.getUsers().pipe(
  retry(2),
  catchError(err => {
    console.error(err);
    return of([]);
  })
).subscribe()
```
