# บทที่ 2: Environment Setup

## 🎯 จุดประสงค์ของบทเรียน
- ติดตั้ง Node.js และเครื่องมือที่จำเป็น
- เรียนรู้การสร้างโปรเจ็กต์ Next.js
- ตั้งค่า Development Environment
- เข้าใจโครงสร้างโปรเจ็กต์เบื้องต้น

## 💻 ข้อกำหนดของระบบ

### **⚙️ System Requirements**

```bash
# ✅ ข้อกำหนดขั้นต่ำ
Operating System: Windows 10+, macOS 10.13+, Linux
Node.js: 18.17+ หรือใหม่กว่า
RAM: 4GB+ (แนะนำ 8GB+)
Storage: 10GB+ ว่าง
Browser: Chrome, Firefox, Safari, Edge (เวอร์ชันล่าสุด)

# 🔍 ตรวจสอบเวอร์ชัน Node.js
node --version  # ควรเป็น v18.17.0 หรือสูงกว่า
npm --version   # ควรเป็น 9.0.0 หรือสูงกว่า
```

### **📦 Package Managers ที่แนะนำ**

```bash
# npm (มาพร้อม Node.js)
npm --version

# yarn (ติดตั้งเพิ่ม - แนะนำ)
npm install -g yarn
yarn --version

# pnpm (ประหยัด disk space)
npm install -g pnpm
pnpm --version

# bun (เร็วที่สุด - ใหม่)
curl -fsSL https://bun.sh/install | bash
bun --version
```

## 🚀 การติดตั้ง Node.js

### **🌐 วิธีที่ 1: ดาวน์โหลดจากเว็บไซต์อย่างเป็นทางการ**

```bash
# 1. ไปที่ https://nodejs.org
# 2. ดาวน์โหลด LTS version (แนะนำ)
# 3. ติดตั้งตามขั้นตอน
# 4. ตรวจสอบการติดตั้ง

node --version
npm --version
```

### **🔧 วิธีที่ 2: ใช้ Node Version Manager (แนะนำ)**

#### **Windows (nvm-windows):**

```powershell
# 1. ดาวน์โหลด nvm-windows จาก GitHub
# https://github.com/coreybutler/nvm-windows/releases

# 2. ติดตั้ง nvm
nvm --version

# 3. ติดตั้ง Node.js เวอร์ชันล่าสุด
nvm install latest
nvm use latest

# 4. ติดตั้ง Node.js LTS
nvm install lts
nvm use lts

# 5. ดู Node.js versions ที่ติดตั้งแล้ว
nvm list
```

#### **macOS/Linux (nvm):**

```bash
# 1. ติดตั้ง nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 2. Restart terminal หรือ
source ~/.bashrc

# 3. ตรวจสอบ nvm
nvm --version

# 4. ติดตั้ง Node.js LTS
nvm install --lts
nvm use --lts

# 5. ตั้งเป็น default
nvm alias default lts/*

# 6. ดู versions ที่มี
nvm list
```

### **🍺 วิธีที่ 3: ใช้ Package Manager**

#### **macOS (Homebrew):**

```bash
# ติดตั้ง Homebrew ก่อน (ถ้ายังไม่มี)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# ติดตั้ง Node.js
brew install node

# ตรวจสอบ
node --version
npm --version
```

#### **Windows (Chocolatey):**

```powershell
# ติดตั้ง Chocolatey ก่อน (Run as Administrator)
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# ติดตั้ง Node.js
choco install nodejs

# ตรวจสอบ
node --version
npm --version
```

#### **Ubuntu/Debian:**

```bash
# วิธีที่ 1: NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# วิธีที่ 2: Snap
sudo snap install node --classic

# ตรวจสอบ
node --version
npm --version
```

## 🛠️ การติดตั้งเครื่องมือพัฒนา

### **📝 Text Editor / IDE**

#### **Visual Studio Code (แนะนำ):**

```bash
# ดาวน์โหลดจาก https://code.visualstudio.com

# Extensions ที่แนะนำสำหรับ Next.js:
- ES7+ React/Redux/React-Native snippets
- Auto Rename Tag  
- Bracket Pair Colorizer
- GitLens
- Prettier - Code formatter
- ESLint
- Tailwind CSS IntelliSense
- Next.js snippets
- TypeScript Importer
- Path Intellisense
```

#### **Extensions Configuration:**

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "emmet.includeLanguages": {
    "javascript": "javascriptreact"
  },
  "typescript.preferences.importModuleSpecifier": "relative"
}
```

### **🌐 Browser Development Tools**

```bash
# Chrome DevTools Extensions:
- React Developer Tools
- Redux DevTools
- Web Vitals

# Firefox Extensions:
- React Developer Tools
- Redux DevTools
```

## 🎯 สร้างโปรเจ็กต์ Next.js แรก

### **⚡ วิธีที่ 1: ใช้ create-next-app (แนะนำ)**

```bash
# สร้างโปรเจ็กต์ด้วย npm
npx create-next-app@latest my-next-app

# สร้างโปรเจ็กต์ด้วย yarn  
yarn create next-app my-next-app

# สร้างโปรเจ็กต์ด้วย pnpm
pnpm create next-app my-next-app

# สร้างโปรเจ็กต์ด้วย bun
bunx create-next-app my-next-app
```

### **🎛️ Configuration Options**

```bash
# ✨ Interactive setup
npx create-next-app@latest

# คำถามที่จะถูกถาม:
✔ What is your project named? my-app
✔ Would you like to use TypeScript? Yes
✔ Would you like to use ESLint? Yes  
✔ Would you like to use Tailwind CSS? Yes
✔ Would you like to use `src/` directory? Yes
✔ Would you like to use App Router? (recommended) Yes
✔ Would you like to customize the default import alias (@/*)? No
```

### **🚀 Manual Configuration**

```bash
# สร้างโปรเจ็กต์ด้วยการตั้งค่าเอง
npx create-next-app@latest my-app \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"

# เข้าไปในโฟลเดอร์โปรเจ็กต์
cd my-app

# เริ่มต้น development server
npm run dev
# หรือ
yarn dev
# หรือ  
pnpm dev
# หรือ
bun dev
```

### **🌟 Template Options**

```bash
# ใช้ template สำเร็จรูป
npx create-next-app@latest --example with-tailwindcss my-app
npx create-next-app@latest --example with-typescript my-app
npx create-next-app@latest --example with-redux my-app
npx create-next-app@latest --example with-mongodb my-app
npx create-next-app@latest --example with-auth0 my-app

# ดู template ทั้งหมด
# https://github.com/vercel/next.js/tree/canary/examples
```

## 📁 โครงสร้างโปรเจ็กต์

### **🗂️ โครงสร้างพื้นฐาน (Pages Router)**

```
my-next-app/
├── 📁 public/              # Static files
│   ├── favicon.ico
│   ├── vercel.svg
│   └── next.svg
├── 📁 src/                 # Source code (optional)
│   └── 📁 pages/           # Page components
│       ├── api/            # API routes
│       ├── _app.js         # App wrapper
│       ├── _document.js    # Document wrapper
│       └── index.js        # Home page
├── 📁 styles/              # CSS files
│   ├── globals.css
│   └── Home.module.css
├── 📄 package.json         # Dependencies
├── 📄 next.config.js       # Next.js config
├── 📄 tailwind.config.js   # Tailwind config
├── 📄 postcss.config.js    # PostCSS config
└── 📄 README.md            # Documentation
```

### **🗂️ โครงสร้างใหม่ (App Router - Next.js 13+)**

```
my-next-app/
├── 📁 public/              # Static files
├── 📁 src/
│   └── 📁 app/             # App directory
│       ├── globals.css     # Global styles
│       ├── layout.js       # Root layout
│       ├── page.js         # Home page
│       ├── loading.js      # Loading UI
│       ├── error.js        # Error UI
│       ├── not-found.js    # 404 page
│       └── 📁 api/         # API routes
├── 📄 package.json
├── 📄 next.config.js
└── 📄 tailwind.config.js
```

### **📄 ไฟล์สำคัญ**

#### **package.json:**

```json
{
  "name": "my-next-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "react": "^18",
    "react-dom": "^18",
    "next": "14.0.0"
  },
  "devDependencies": {
    "typescript": "^5",
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "autoprefixer": "^10.0.1",
    "postcss": "^8",
    "tailwindcss": "^3.3.0",
    "eslint": "^8",
    "eslint-config-next": "14.0.0"
  }
}
```

#### **next.config.js:**

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Experimental features
  experimental: {
    appDir: true, // Enable App Router
  },
  
  // Image domains (สำหรับ next/image)
  images: {
    domains: ['example.com', 'cdn.example.com'],
  },
  
  // Environment variables
  env: {
    CUSTOM_KEY: 'my-value',
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-page',
        destination: '/new-page',
        permanent: true,
      },
    ];
  },
  
  // Headers
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          {
            key: 'Access-Control-Allow-Origin',
            value: '*',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

#### **tailwind.config.js:**

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [],
};
```

## 🏃‍♂️ เริ่มต้น Development Server

### **🚀 คำสั่ง Scripts**

```bash
# เริ่ม development server
npm run dev
# ➜ Local: http://localhost:3000
# ➜ Network: http://192.168.1.100:3000

# เปลี่ยน port
npm run dev -- -p 4000
# หรือ
PORT=4000 npm run dev

# Build production
npm run build

# Start production server
npm run start

# Lint code
npm run lint

# Type check (TypeScript)
npm run type-check
```

### **🌐 เข้าถึงผ่าน Network**

```bash
# แสดง Network URL
npm run dev

# หรือระบุ host เอง
npm run dev -- -H 0.0.0.0

# เข้าถึงจากเครื่องอื่นใน network
# http://[your-ip]:3000
```

## 🔧 Development Environment Setup

### **⚙️ Environment Variables**

```bash
# .env.local (สำหรับ development)
NEXT_PUBLIC_API_URL=http://localhost:3001/api
DATABASE_URL=postgresql://username:password@localhost:5432/mydb
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:3000

# .env.production (สำหรับ production)
NEXT_PUBLIC_API_URL=https://api.myapp.com
DATABASE_URL=postgresql://username:password@prod-db:5432/mydb
NEXTAUTH_URL=https://myapp.com
```

### **📋 ESLint Configuration**

```json
// .eslintrc.json
{
  "extends": ["next/core-web-vitals"],
  "rules": {
    "react/no-unescaped-entities": "off",
    "@next/next/no-page-custom-font": "off",
    "prefer-const": "error",
    "no-unused-vars": "warn"
  }
}
```

### **🎨 Prettier Configuration**

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
```

### **🚫 .gitignore**

```gitignore
# Dependencies
/node_modules
/.pnp
.pnp.js

# Production build
/.next/
/out/

# Environment variables
.env*.local

# Debug logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# TypeScript
*.tsbuildinfo
next-env.d.ts
```

## 🧪 การทดสอบ Installation

### **✅ ตรวจสอบการติดตั้ง**

```bash
# 1. ตรวจสอบ Node.js และ npm
node --version  # v18.17.0+
npm --version   # 9.0.0+

# 2. สร้างโปรเจ็กต์ทดสอบ
npx create-next-app@latest test-app --typescript --tailwind --eslint --app

# 3. เข้าไปในโปรเจ็กต์
cd test-app

# 4. ติดตั้ง dependencies
npm install

# 5. เริ่ม development server
npm run dev

# 6. เปิด browser ไปที่ http://localhost:3000
# ควรเห็นหน้า Welcome to Next.js
```

### **🔍 ทดสอบ Hot Reload**

```jsx
// src/app/page.js - แก้ไขข้อความ
export default function Home() {
  return (
    <main>
      <h1>Hello Next.js! 🚀</h1>
      <p>This is my first Next.js app</p>
    </main>
  );
}

// บันทึกไฟล์และดูผลลัพธ์ใน browser
// ควรเปลี่ยนแปลงทันทีโดยไม่ต้อง refresh
```

### **🎯 ทดสอบ Routing**

```bash
# สร้างหน้าใหม่
mkdir src/app/about
echo 'export default function About() { 
  return <h1>About Page</h1>; 
}' > src/app/about/page.js

# เปิด http://localhost:3000/about
# ควรเห็นหน้า About Page
```

## 🛠️ Troubleshooting

### **⚠️ ปัญหาที่พบบ่อย**

#### **1. Node.js เวอร์ชันเก่า:**

```bash
# หาก node version < 18.17
Error: You are using Node.js 16.x. Next.js requires Node.js 18.17 or higher.

# แก้ไข: อัพเดต Node.js
nvm install --lts
nvm use --lts
```

#### **2. Port ถูกใช้อยู่:**

```bash
# หาก port 3000 ถูกใช้
Error: listen EADDRINUSE: address already in use :::3000

# แก้ไข: เปลี่ยน port
npm run dev -- -p 3001
```

#### **3. Permission Error:**

```bash
# หาก Permission denied
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) /usr/local/lib/node_modules
```

#### **4. Module Not Found:**

```bash
# หาก Module not found
rm -rf node_modules package-lock.json
npm install
```

#### **5. TypeScript Error:**

```bash
# หาก TypeScript errors
npm run type-check

# หรือปิด type checking ชั่วคราว
// next.config.js
const nextConfig = {
  typescript: {
    ignoreBuildErrors: true,
  },
};
```

### **🔧 Performance Optimization**

```bash
# เพิ่มความเร็วในการ install
npm config set registry https://registry.npmjs.org/
npm config set fund false
npm config set audit false

# ใช้ yarn แทน npm (เร็วกว่า)
npm install -g yarn
yarn install
yarn dev

# ใช้ pnpm (ประหยัด disk space)
npm install -g pnpm
pnpm install
pnpm dev
```

## 📱 Mobile Development Setup

### **📲 การทดสอบบนมือถือ**

```bash
# 1. หา IP address ของเครื่อง
# Windows
ipconfig

# macOS/Linux  
ifconfig

# 2. เริ่ม server ด้วย network access
npm run dev -- -H 0.0.0.0

# 3. เข้าถึงจากมือถือ
# http://[your-ip]:3000
# เช่น http://192.168.1.100:3000
```

### **🔗 QR Code Generator**

```javascript
// สร้าง QR code สำหรับง่ายในการเข้าถึง
npm install qrcode-terminal

// scripts/dev-mobile.js
const qrcode = require('qrcode-terminal');
const os = require('os');

const interfaces = os.networkInterfaces();
const addresses = [];

for (const k in interfaces) {
  for (const k2 in interfaces[k]) {
    const address = interfaces[k][k2];
    if (address.family === 'IPv4' && !address.internal) {
      addresses.push(address.address);
    }
  }
}

if (addresses.length > 0) {
  const url = `http://${addresses[0]}:3000`;
  console.log(`\n📱 Mobile Access:`);
  qrcode.generate(url, { small: true });
  console.log(url);
}
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic Setup**
1. ติดตั้ง Node.js เวอร์ชันล่าสุด
2. สร้างโปรเจ็กต์ Next.js ใหม่ด้วย TypeScript และ Tailwind CSS
3. เปลี่ยนข้อความในหน้าแรกและดูผลลัพธ์
4. สร้างหน้า `/about` ใหม่

### **🎯 แบบฝึกหัดที่ 2: Configuration**  
1. ตั้งค่า environment variables
2. เปลี่ยน default port เป็น 4000
3. เพิ่ม custom domain ใน next.config.js สำหรับ images
4. ติดตั้งและตั้งค่า Prettier

### **🎯 แบบฝึกหัดที่ 3: Mobile Testing**
1. หา IP address ของเครื่อง
2. เข้าถึงเว็บไซต์จากมือถือ
3. ทดสอบ responsive design
4. ตรวจสอบ performance บนมือถือ

### **🎯 แบบฝึกหัดที่ 4: Troubleshooting**
เจตนาสร้างข้อผิดพลาดแล้วแก้ไข:
1. ลบ node_modules แล้วแก้ไข
2. ใช้ port ที่ถูกใช้อยู่แล้วแก้ไข  
3. สร้าง syntax error แล้วแก้ไข
4. ลบ package.json แล้วสร้างใหม่

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 1: Introduction to Next.js](./01-introduction.md)

**บทต่อไป**: [บทที่ 3: Project Structure](./03-project-structure.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [Node.js Official Website](https://nodejs.org)
- [Next.js Installation Guide](https://nextjs.org/docs/getting-started/installation)
- [Visual Studio Code](https://code.visualstudio.com)
- [nvm (Node Version Manager)](https://github.com/nvm-sh/nvm)
