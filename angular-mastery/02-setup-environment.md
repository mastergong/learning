# 02 — Setup Environment (Windows, macOS, Linux)

## Install Node.js
- Download LTS from nodejs.org
- Verify:
```powershell
node -v
npm -v
```

Optional (Windows): nvm-windows to switch Node versions.

## Install Angular CLI
```powershell
npm i -g @angular/cli
ng version
```

## Create a new app (standalone)
```powershell
ng new my-app --standalone --routing --style=scss
cd my-app
ng serve -o
```

## VS Code essentials
- Extensions: Angular Language Service, ESLint, Prettier
- Settings: format on save, ESLint fixes on save

## Troubleshooting
- Clear npm cache, delete node_modules, reinstall
```powershell
npm cache verify
rd /s /q node_modules
npm i
```
