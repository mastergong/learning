# บทที่ 2: การติดตั้งและการตั้งค่า

## 🎯 จุดประสงค์ของบทเรียน
- ติดตั้ง Node.js และ TypeScript
- ตั้งค่า development environment
- เขียนและรัน TypeScript program แรก
- เข้าใจไฟล์ tsconfig.json

## 📋 ข้อกำหนดระบบ

- **Node.js** เวอร์ชัน 14.x หรือใหม่กว่า
- **Code Editor** (แนะนำ VS Code)
- **Terminal/Command Prompt**

## 🔧 ขั้นตอนการติดตั้ง

### 1. ติดตั้ง Node.js

ดาวน์โหลดและติดตั้งจาก [nodejs.org](https://nodejs.org/)

ตรวจสอบการติดตั้ง:
```bash
node --version
npm --version
```

### 2. ติดตั้ง TypeScript

```bash
# ติดตั้งแบบ global
npm install -g typescript

# ตรวจสอบการติดตั้ง
tsc --version
```

### 3. ติดตั้ง ts-node (สำหรับรัน TypeScript โดยตรง)

```bash
npm install -g ts-node
```

## 🏗️ สร้างโปรเจกต์แรก

### 1. สร้างโฟลเดอร์โปรเจกต์

```bash
mkdir my-first-typescript
cd my-first-typescript
```

### 2. สร้างไฟล์ package.json

```bash
npm init -y
```

### 3. ติดตั้ง TypeScript สำหรับโปรเจกต์

```bash
# ติดตั้งเป็น dev dependency
npm install --save-dev typescript
npm install --save-dev @types/node

# ติดตั้ง ts-node สำหรับ development
npm install --save-dev ts-node
```

### 4. สร้างไฟล์ tsconfig.json

```bash
npx tsc --init
```

## ⚙️ การตั้งค่า tsconfig.json

### Basic Configuration

```json
{
  "compilerOptions": {
    // Target JavaScript version
    "target": "ES2020",
    
    // Module system
    "module": "commonjs",
    
    // Output directory
    "outDir": "./dist",
    
    // Source directory
    "rootDir": "./src",
    
    // Enable strict type checking
    "strict": true,
    
    // Enable all strict type checking options
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    
    // Module resolution
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    
    // Source maps for debugging
    "sourceMap": true,
    
    // Skip type checking of declaration files
    "skipLibCheck": true,
    
    // Ensure consistent casing
    "forceConsistentCasingInFileNames": true,
    
    // Include type definitions
    "lib": ["ES2020", "DOM"]
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

### คำอธิบายการตั้งค่าสำคัญ

| Option | คำอธิบาย |
|--------|----------|
| `target` | เวอร์ชัน JavaScript ที่จะ compile ออกมา |
| `module` | ระบบ module ที่ใช้ |
| `outDir` | โฟลเดอร์สำหรับไฟล์ JavaScript ที่ compile แล้ว |
| `rootDir` | โฟลเดอร์ต้นทางของ source code |
| `strict` | เปิดใช้งาน strict mode ทั้งหมด |
| `sourceMap` | สร้างไฟล์ source map สำหรับ debugging |

## 📁 โครงสร้างโปรเจกต์

```
my-first-typescript/
├── src/
│   ├── index.ts
│   └── utils/
│       └── helper.ts
├── dist/          (จะถูกสร้างหลัง compile)
├── node_modules/
├── package.json
└── tsconfig.json
```

## 💻 เขียนโปรแกรมแรก

### 1. สร้างไฟล์ src/index.ts

```typescript
// src/index.ts
interface Person {
    name: string;
    age: number;
}

function greet(person: Person): string {
    return `Hello, ${person.name}! You are ${person.age} years old.`;
}

const user: Person = {
    name: "Alice",
    age: 30
};

console.log(greet(user));

// Type checking in action
// greet({ name: "Bob" }); // Error: Property 'age' is missing
```

### 2. รันโปรแกรม

#### วิธีที่ 1: Compile แล้วรัน
```bash
# Compile TypeScript to JavaScript
npx tsc

# Run JavaScript
node dist/index.js
```

#### วิธีที่ 2: รันโดยตรงด้วย ts-node
```bash
npx ts-node src/index.ts
```

### 3. ตั้งค่า npm scripts ใน package.json

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts",
    "watch": "tsc --watch"
  }
}
```

## 🛠️ การตั้งค่า VS Code

### 1. Extensions ที่แนะนำ

- **TypeScript Importer** - Auto import
- **Prettier** - Code formatting
- **ESLint** - Code linting
- **Bracket Pair Colorizer** - ไฮไลต์ bracket

### 2. การตั้งค่า VS Code (settings.json)

```json
{
  "typescript.preferences.quote": "double",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

### 3. การตั้งค่า Prettier (.prettierrc)

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

## 🔍 การ Debug TypeScript

### 1. การตั้งค่า launch.json ใน VS Code

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch TypeScript",
      "program": "${workspaceFolder}/src/index.ts",
      "runtimeArgs": ["--loader", "ts-node/esm"],
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "sourceMaps": true,
      "console": "integratedTerminal"
    }
  ]
}
```

### 2. การใช้ Breakpoints

```typescript
// src/index.ts
function calculateTotal(items: number[]): number {
    let total = 0;
    
    for (const item of items) {
        total += item; // ใส่ breakpoint ที่นี่
    }
    
    return total;
}

const numbers = [1, 2, 3, 4, 5];
console.log(calculateTotal(numbers));
```

## 📝 แบบฝึกหัด

### 1. Basic Setup
สร้างโปรเจกต์ TypeScript ใหม่ที่มี:
- ไฟล์ tsconfig.json ที่ตั้งค่าอย่างเหมาะสม
- โครงสร้างโฟลเดอร์ที่เป็นระเบียบ
- npm scripts สำหรับ build และ run

### 2. First Program
เขียนโปรแกรมที่:
- รับข้อมูลผู้ใช้ (ชื่อ, อายุ, อีเมล)
- แสดงข้อมูลในรูปแบบที่สวยงาม
- มีการตรวจสอบ type อย่างเหมาะสม

### 3. Configuration
ทดลองเปลี่ยนการตั้งค่าใน tsconfig.json:
- เปลี่ยน target เป็น ES5 แล้วดูผลลัพธ์
- ลองปิด strict mode แล้วดูความแตกต่าง

## 🚨 ปัญหาที่พบบ่อยและการแก้ไข

### 1. "tsc: command not found"
```bash
# แก้ไขด้วยการติดตั้ง TypeScript แบบ global
npm install -g typescript
```

### 2. "Cannot find module '@types/node'"
```bash
# ติดตั้ง type definitions สำหรับ Node.js
npm install --save-dev @types/node
```

### 3. "Module not found" errors
ตรวจสอบการตั้งค่า `moduleResolution` ใน tsconfig.json:
```json
{
  "compilerOptions": {
    "moduleResolution": "node"
  }
}
```

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 1: แนะนำ TypeScript](./01-introduction.md)

**บทต่อไป**: [บทที่ 3: ชนิดข้อมูลพื้นฐาน](./03-basic-types.md)

**กลับไปหน้าหลัก**: [README](./README.md)
