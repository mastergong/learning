# 15 — Deployment & CI/CD

## Build for production
```powershell
ng build --configuration production
```

## Static hosting
- Firebase Hosting, Netlify, Vercel, Azure Static Web Apps

## CI with GitHub Actions (example)
```yaml
name: build
on: [push]
jobs:
  web:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run build --workspaces=false
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/ }
```
