# บทที่ 2: Development Environment Setup

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- ติดตั้ง Node.js และ npm ได้อย่างถูกต้อง
- ติดตั้งและใช้งาน Angular CLI
- Setup Visual Studio Code สำหรับ Angular development
- สร้างโปรเจกต์ Angular แรกและรันได้สำเร็จ
- เข้าใจ project structure และการจัดการ dependencies

## 🛠️ System Requirements

### **ความต้องการของระบบ**

#### **Operating System**
- ✅ Windows 10/11
- ✅ macOS 10.15+
- ✅ Linux (Ubuntu 18.04+, CentOS 7+)

#### **Hardware Requirements**
- **RAM**: 8GB+ (แนะนำ 16GB)
- **Storage**: 10GB+ free space
- **CPU**: Intel i5 หรือ AMD equivalent

#### **Browser Support**
- ✅ Chrome (recommended)
- ✅ Firefox
- ✅ Safari
- ✅ Edge

## 📦 Node.js และ npm Installation

### **Step 1: ติดตั้ง Node.js**

#### **Windows Installation**
```powershell
# ตรวจสอบเวอร์ชันที่ติดตั้งแล้ว (ถ้ามี)
node --version
npm --version

# หากยังไม่มี ให้ download จาก https://nodejs.org
# เลือก LTS version (แนะนำ)
```

#### **macOS Installation**
```bash
# ใช้ Homebrew (แนะนำ)
brew install node

# หรือ download จาก nodejs.org
# ตรวจสอบการติดตั้ง
node --version
npm --version
```

#### **Linux Installation**
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# CentOS/RHEL
curl -fsSL https://rpm.nodesource.com/setup_lts.x | sudo bash -
sudo yum install -y nodejs

# ตรวจสอบการติดตั้ง
node --version
npm --version
```

### **Node Version Management**

#### **NVM (Node Version Manager)**
```bash
# ติดตั้ง NVM (macOS/Linux)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Windows ใช้ nvm-windows
# Download จาก: https://github.com/coreybutler/nvm-windows

# ใช้งาน NVM
nvm install 18.17.0    # ติดตั้ง Node.js version
nvm use 18.17.0        # เปลี่ยนไปใช้ version นี้
nvm list               # ดู versions ที่ติดตั้ง
```

### **npm Configuration**

#### **Global Configuration**
```bash
# ดู npm config
npm config list

# ตั้ง registry (ถ้าอยู่ในองค์กร)
npm config set registry https://registry.npmjs.org/

# ตั้ง proxy (ถ้าจำเป็น)
npm config set proxy http://proxy.company.com:8080
npm config set https-proxy http://proxy.company.com:8080

# ตั้ง cache directory
npm config set cache "C:\npm-cache"  # Windows
npm config set cache "/usr/local/lib/node_modules/.npm"  # macOS/Linux
```

#### **Alternative Package Managers**

**Yarn Installation**
```bash
# ติดตั้ง Yarn
npm install -g yarn

# ตรวจสอบ
yarn --version

# Basic commands
yarn install     # = npm install
yarn add <pkg>   # = npm install <pkg>
yarn global add  # = npm install -g
```

**pnpm Installation**
```bash
# ติดตั้ง pnpm
npm install -g pnpm

# ตรวจสอบ
pnpm --version

# Basic commands
pnpm install     # faster than npm
pnpm add <pkg>
pnpm dlx <pkg>   # = npx <pkg>
```

## 🅰️ Angular CLI Installation

### **Global Installation**
```bash
# ติดตั้ง Angular CLI แบบ global
npm install -g @angular/cli

# ตรวจสอบการติดตั้ง
ng version
# หรือ
ng --version
```

### **Version Output Example**
```bash
Angular CLI: 15.2.8
Node: 18.17.0
Package Manager: npm 9.6.7
OS: darwin x64

Angular: 
... 

Package                      Version
------------------------------------------------------
@angular-devkit/architect    0.1502.8
@angular-devkit/core         15.2.8
@angular-devkit/schematics   15.2.8
@schematics/angular          15.2.8
```

### **CLI Update**
```bash
# Update Angular CLI
npm update -g @angular/cli

# หรือ uninstall แล้ว install ใหม่
npm uninstall -g @angular/cli
npm install -g @angular/cli@latest
```

### **Project-Specific CLI**
```bash
# ใช้ CLI ตาม version ของโปรเจกต์
npx @angular/cli@15 new my-project

# หรือติดตั้งแบบ local
npm install --save-dev @angular/cli
npx ng version
```

## 💻 Visual Studio Code Setup

### **VS Code Installation**
1. Download จาก [https://code.visualstudio.com](https://code.visualstudio.com)
2. ติดตั้งตาม OS ของคุณ
3. เปิด VS Code และติดตั้ง Extensions

### **Essential Extensions**

#### **Angular Extensions**
```json
{
  "recommendations": [
    "angular.ng-template",           // Angular Language Service
    "johnpapa.angular2",             // Angular Snippets
    "cyrilletuzi.angular-schematics", // Angular Schematics
    "mikerhyssmith.angular-material-snippets", // Material Snippets
    "pkief.material-icon-theme"      // Material Icon Theme
  ]
}
```

#### **TypeScript Extensions**
```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next", // TypeScript Importer
    "streetsidesoftware.code-spell-checker", // Spell Checker
    "esbenp.prettier-vscode",        // Prettier Code Formatter
    "dbaeumer.vscode-eslint",        // ESLint
    "bradlc.vscode-tailwindcss"      // Tailwind CSS
  ]
}
```

#### **General Development Extensions**
```json
{
  "recommendations": [
    "eamodio.gitlens",               // GitLens
    "ms-vscode.live-server",         // Live Server
    "formulahendry.auto-rename-tag", // Auto Rename Tag
    "christian-kohler.path-intellisense", // Path Intellisense
    "ms-vscode.bracket-pair-colorizer-2", // Bracket Pair Colorizer
    "oderwat.indent-rainbow",        // Indent Rainbow
    "ms-vscode.theme-monokai-dimmed" // Monokai Dimmed Theme
  ]
}
```

### **VS Code Settings for Angular**

#### **settings.json Configuration**
```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  
  "emmet.includeLanguages": {
    "typescript": "html"
  },
  "emmet.triggerExpansionOnTab": true,
  
  "files.associations": {
    "*.html": "html"
  },
  
  "html.suggest.html5": true,
  "css.validate": true,
  "scss.validate": true,
  
  "explorer.confirmDelete": false,
  "explorer.confirmDragAndDrop": false,
  
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "terminal.integrated.profiles.windows": {
    "PowerShell": {
      "source": "PowerShell",
      "icon": "terminal-powershell"
    }
  }
}
```

#### **Angular-specific Snippets**
```json
// typescript.json (User Snippets)
{
  "Angular Component": {
    "prefix": "ng-component",
    "body": [
      "import { Component } from '@angular/core';",
      "",
      "@Component({",
      "  selector: 'app-${1:component-name}',",
      "  templateUrl: './${1:component-name}.component.html',",
      "  styleUrls: ['./${1:component-name}.component.css']",
      "})",
      "export class ${2:ComponentName}Component {",
      "  ${3:// component code}",
      "}"
    ],
    "description": "Angular Component Template"
  }
}
```

## 🏗️ สร้างโปรเจกต์ Angular แรก

### **สร้างโปรเจกต์ใหม่**
```bash
# สร้างโปรเจกต์ใหม่
ng new my-first-app

# ตัวเลือกระหว่างการสร้าง
? Would you like to add Angular routing? Yes
? Which stylesheet format would you like to use? CSS
```

### **การตั้งค่าขั้นสูง**
```bash
# สร้างพร้อมกำหนดตัวเลือก
ng new advanced-app \
  --routing=true \
  --style=scss \
  --prefix=app \
  --skip-git=false \
  --package-manager=npm \
  --strict=true
```

### **Project Structure Overview**
```
my-first-app/
├── src/                     # Source code
│   ├── app/                 # Application code
│   │   ├── app.component.*  # Root component
│   │   ├── app.module.ts    # Root module
│   │   └── app-routing.module.ts # Routing configuration
│   ├── assets/              # Static assets
│   ├── environments/        # Environment configurations
│   ├── index.html           # Main HTML file
│   ├── main.ts              # Application bootstrap
│   ├── polyfills.ts         # Browser polyfills
│   └── styles.css           # Global styles
├── angular.json             # Angular CLI configuration
├── package.json             # Dependencies
├── tsconfig.json            # TypeScript configuration
├── karma.conf.js            # Testing configuration
└── README.md                # Project documentation
```

### **รันโปรเจกต์**
```bash
# เข้าไปในโฟลเดอร์โปรเจกต์
cd my-first-app

# รัน development server
ng serve

# หรือกำหนด port
ng serve --port 4200

# เปิดบราวเซอร์อัตโนมัติ
ng serve --open
```

### **Build Commands**
```bash
# Build สำหรับ development
ng build

# Build สำหรับ production
ng build --prod
# หรือ
ng build --configuration=production

# Analyze bundle size
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

## 🔧 Project Configuration

### **angular.json Configuration**
```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "my-app": {
      "projectType": "application",
      "schematics": {
        "@schematics/angular:component": {
          "style": "scss",
          "changeDetection": "OnPush"
        }
      },
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": []
          }
        }
      }
    }
  }
}
```

### **TypeScript Configuration**
```json
// tsconfig.json
{
  "compileOnSave": false,
  "compilerOptions": {
    "baseUrl": "./",
    "outDir": "./dist/out-tsc",
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "sourceMap": true,
    "declaration": false,
    "downlevelIteration": true,
    "experimentalDecorators": true,
    "moduleResolution": "node",
    "importHelpers": true,
    "target": "ES2022",
    "module": "ES2022",
    "useDefineForClassFields": false,
    "lib": [
      "ES2022",
      "dom"
    ]
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

### **Package.json Scripts**
```json
{
  "name": "my-app",
  "version": "0.0.0",
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test",
    "test:coverage": "ng test --code-coverage",
    "e2e": "ng e2e",
    "lint": "ng lint",
    "format": "prettier --write \"src/**/*.{ts,html,scss}\"",
    "format:check": "prettier --check \"src/**/*.{ts,html,scss}\"",
    "serve:prod": "ng serve --configuration=production",
    "build:stats": "ng build --stats-json",
    "analyze": "npx webpack-bundle-analyzer dist/my-app/stats.json"
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "^15.2.0",
    "@angular/common": "^15.2.0",
    "@angular/compiler": "^15.2.0",
    "@angular/core": "^15.2.0",
    "@angular/forms": "^15.2.0",
    "@angular/platform-browser": "^15.2.0",
    "@angular/platform-browser-dynamic": "^15.2.0",
    "@angular/router": "^15.2.0",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.12.0"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^15.2.8",
    "@angular/cli": "~15.2.8",
    "@angular/compiler-cli": "^15.2.0",
    "@types/jasmine": "~4.3.0",
    "@types/node": "^18.7.0",
    "jasmine-core": "~4.6.0",
    "karma": "~6.4.0",
    "karma-chrome-headless": "~3.1.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.1.0",
    "typescript": "~4.9.4"
  }
}
```

## 🚀 Development Workflow

### **Angular CLI Commands**

#### **Generation Commands**
```bash
# Generate Component
ng generate component user-profile
ng g c user-profile --skip-tests

# Generate Service
ng generate service services/user
ng g s services/user

# Generate Module
ng generate module features/auth --routing
ng g m features/auth --routing

# Generate Directive
ng generate directive shared/highlight
ng g d shared/highlight

# Generate Pipe
ng generate pipe shared/phone-format
ng g p shared/phone-format

# Generate Guard
ng generate guard guards/auth
ng g g guards/auth

# Generate Interface
ng generate interface models/user
ng g i models/user

# Generate Enum
ng generate enum enums/user-role
ng g e enums/user-role
```

#### **Project Commands**
```bash
# Add dependencies
ng add @angular/material
ng add @angular/pwa
ng add @ngrx/store

# Update dependencies
ng update
ng update @angular/core @angular/cli

# Lint code
ng lint

# Test
ng test
ng test --watch=false --browsers=ChromeHeadless

# E2E testing
ng e2e

# Extract i18n messages
ng extract-i18n
```

### **Development Server Options**
```bash
# Basic serve
ng serve

# Custom port
ng serve --port 3000

# Open browser automatically
ng serve --open

# Production mode (for testing)
ng serve --configuration=production

# With SSL
ng serve --ssl

# Different host (for network access)
ng serve --host 0.0.0.0

# Disable live reload
ng serve --live-reload=false

# Custom proxy config
ng serve --proxy-config proxy.conf.json
```

### **Proxy Configuration**
```json
// proxy.conf.json
{
  "/api/*": {
    "target": "http://localhost:3000",
    "secure": true,
    "changeOrigin": true,
    "logLevel": "debug"
  },
  "/auth/*": {
    "target": "https://auth.example.com",
    "secure": true,
    "changeOrigin": true,
    "headers": {
      "X-Forwarded-For": "localhost"
    }
  }
}
```

## 🔍 Debugging Setup

### **Chrome DevTools Configuration**
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome against localhost",
      "url": "http://localhost:4200",
      "webRoot": "${workspaceFolder}/src",
      "sourceMaps": true,
      "sourceMapPathOverrides": {
        "webpack:/*": "${webRoot}/*"
      }
    },
    {
      "name": "Attach to Chrome",
      "port": 9222,
      "request": "attach",
      "type": "chrome",
      "webRoot": "${workspaceFolder}/src",
      "sourceMaps": true
    }
  ]
}
```

### **Angular DevTools**
```bash
# ติดตั้ง Angular DevTools browser extension
# Chrome: https://chrome.google.com/webstore/detail/angular-devtools/ienfalfjdbdpebioblfackkekamfmbnh
# Firefox: https://addons.mozilla.org/en-US/firefox/addon/angular-devtools/
```

### **Source Maps Configuration**
```json
// angular.json
"build": {
  "builder": "@angular-devkit/build-angular:browser",
  "options": {
    "sourceMap": {
      "scripts": true,
      "styles": true,
      "vendor": true
    }
  }
}
```

## 🧪 Testing Environment

### **Unit Testing Configuration**
```typescript
// karma.conf.js
module.exports = function (config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine', '@angular-devkit/build-angular'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-headless'),
      require('karma-jasmine-html-reporter'),
      require('karma-coverage'),
      require('@angular-devkit/build-angular/plugins/karma')
    ],
    client: {
      jasmine: {
        random: true,
        seed: '4321',
        oneFailurePerSpec: true,
        failFast: true,
        timeoutInterval: 1000
      },
      clearContext: false
    },
    jasmineHtmlReporter: {
      suppressAll: true
    },
    coverageReporter: {
      dir: require('path').join(__dirname, './coverage/my-app'),
      subdir: '.',
      reporters: [
        { type: 'html' },
        { type: 'text-summary' },
        { type: 'lcov' }
      ],
      check: {
        global: {
          statements: 80,
          branches: 80,
          functions: 80,
          lines: 80
        }
      }
    },
    reporters: ['progress', 'kjhtml'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: true,
    browsers: ['Chrome'],
    singleRun: false,
    restartOnFileChange: true
  });
};
```

### **E2E Testing Setup**
```bash
# ติดตั้ง Cypress (แนะนำแทน Protractor)
ng add @cypress/schematic

# รัน E2E tests
ng e2e

# เปิด Cypress Test Runner
npx cypress open
```

## 📁 Project Structure Best Practices

### **Recommended Folder Structure**
```
src/
├── app/
│   ├── core/                 # Singleton services, guards
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── services/
│   │   └── core.module.ts
│   ├── shared/               # Shared components, pipes, directives
│   │   ├── components/
│   │   ├── directives/
│   │   ├── pipes/
│   │   ├── models/
│   │   └── shared.module.ts
│   ├── features/             # Feature modules
│   │   ├── auth/
│   │   ├── dashboard/
│   │   └── users/
│   ├── layout/               # Layout components
│   │   ├── header/
│   │   ├── footer/
│   │   └── sidebar/
│   ├── app-routing.module.ts
│   ├── app.component.*
│   └── app.module.ts
├── assets/                   # Static files
│   ├── images/
│   ├── icons/
│   ├── fonts/
│   └── i18n/
├── environments/             # Environment configs
│   ├── environment.ts
│   └── environment.prod.ts
└── styles/                   # Global styles
    ├── _variables.scss
    ├── _mixins.scss
    └── main.scss
```

## 🎯 แบบฝึกหัด

### **Exercise 1: Environment Setup Verification**
1. ตรวจสอบว่าติดตั้งเครื่องมือครบถ้วน:
   ```bash
   node --version
   npm --version
   ng version
   ```

2. สร้างโปรเจกต์ test:
   ```bash
   ng new exercise-app --routing --style=scss
   cd exercise-app
   ng serve
   ```

3. เปิดเบราว์เซอร์ไปที่ `http://localhost:4200`

### **Exercise 2: VS Code Configuration**
1. ติดตั้ง Extensions ที่จำเป็น
2. สร้าง settings.json สำหรับ Angular
3. ทดสอบ code completion และ error detection

### **Exercise 3: CLI Commands Practice**
```bash
# สร้าง components และ services
ng g c components/header
ng g c components/footer
ng g s services/data
ng g i models/user

# ทดสอบ generation options
ng g c components/nav --skip-tests --inline-style
```

## 🔧 Troubleshooting

### **Common Issues**

#### **Node.js Version Conflicts**
```bash
# ใช้ NVM เพื่อจัดการ versions
nvm install 18.17.0
nvm use 18.17.0
npm install -g @angular/cli
```

#### **Permission Issues (macOS/Linux)**
```bash
# แก้ npm permissions
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}

# หรือใช้ nvm แทน
```

#### **Port Already in Use**
```bash
# หา process ที่ใช้ port
lsof -ti :4200 | xargs kill -9  # macOS/Linux
netstat -ano | findstr :4200    # Windows

# หรือใช้ port อื่น
ng serve --port 4201
```

#### **Angular CLI Not Found**
```bash
# ติดตั้งใหม่
npm uninstall -g @angular/cli
npm install -g @angular/cli@latest

# หรือใช้ npx
npx @angular/cli new my-app
```

### **Performance Optimization**
```bash
# เพิ่ม memory limit
export NODE_OPTIONS="--max-old-space-size=8192"

# ใช้ npm cache
npm config set cache ~/.npm --global

# ใช้ yarn แทน npm (เร็วกว่า)
npm install -g yarn
yarn install
```

## 🧪 Quiz

### **Question 1**
Angular CLI ใช้สำหรับ:
- a) รัน web server เท่านั้น
- b) สร้างและจัดการ Angular projects
- c) เขียน TypeScript เท่านั้น
- d) ติดตั้ง Node.js

### **Question 2**
คำสั่งใดใช้สร้าง Angular component:
- a) `ng create component`
- b) `ng generate component`
- c) `ng new component`
- d) `ng add component`

### **Question 3**
Port default ของ Angular development server คือ:
- a) 3000
- b) 8080
- c) 4200
- d) 5000

**คำตอบ: 1-b, 2-b, 3-c**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🛠️ Development Environment**
1. **Node.js และ npm** - รันไทม์และแพ็กเกจแมเนเจอร์
2. **Angular CLI** - เครื่องมือสำคัญสำหรับ Angular development
3. **VS Code** - การตั้งค่าและ extensions สำหรับ Angular
4. **Project Structure** - โครงสร้างโปรเจกต์มาตรฐาน

### **🚀 Development Workflow**
1. **สร้างโปรเจกต์** ด้วย `ng new`
2. **รัน development server** ด้วย `ng serve`
3. **Generate code** ด้วย `ng generate`
4. **Build และ deploy** ด้วย `ng build`

### **🔧 Tools และ Configuration**
1. **TypeScript configuration** สำหรับ type checking
2. **Angular configuration** ใน angular.json
3. **Testing setup** สำหรับ unit และ E2E tests
4. **Debugging tools** สำหรับ development

### **📚 Next Steps**
ในบทต่อไป เราจะเรียนรู้ TypeScript fundamentals ซึ่งเป็นพื้นฐานสำคัญสำหรับ Angular development

---

**🎉 ยินดีด้วย! ตอนนี้คุณพร้อมสำหรับ Angular development แล้ว**

**[⬅️ บทที่ 1: Angular Introduction](./01-angular-introduction.md) | [➡️ บทที่ 3: TypeScript Fundamentals](./03-typescript-fundamentals.md)**
