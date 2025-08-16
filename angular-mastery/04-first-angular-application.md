# บทที่ 4: First Angular Application

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- สร้างและเข้าใจ Angular application แรก
- เข้าใจ project structure และไฟล์สำคัญ
- ทำงานกับ Angular Components พื้นฐาน
- ใช้ Angular CLI สำหรับการพัฒนา
- Debug และแก้ไขปัญหาเบื้องต้น

## 🚀 สร้าง Angular Application แรก

### **Step 1: สร้างโปรเจกต์ใหม่**
```bash
# สร้างโปรเจกต์ Angular ใหม่
ng new my-first-app

# ตัวเลือกระหว่างการสร้าง
? Would you like to add Angular routing? Yes
? Which stylesheet format would you like to use? CSS

# เข้าไปในโฟลเดอร์โปรเจกต์
cd my-first-app

# รันโปรเจกต์
ng serve

# เปิดเบราว์เซอร์ไปที่ http://localhost:4200
```

### **Step 2: ตรวจสอบการทำงาน**
เมื่อรันสำเร็จ คุณจะเห็น:
- Angular welcome page
- URL: `http://localhost:4200`
- Live reload เมื่อมีการเปลี่ยนแปลงไฟล์

## 📁 Project Structure Overview

### **Root Level Files**
```
my-first-app/
├── src/                    # Source code หลัก
├── node_modules/           # Dependencies
├── angular.json           # Angular CLI configuration
├── package.json           # Project dependencies และ scripts
├── tsconfig.json          # TypeScript configuration
├── karma.conf.js          # Unit testing configuration
├── .editorconfig          # Editor configuration
├── .gitignore            # Git ignore rules
└── README.md             # Project documentation
```

### **Source Directory (src/)**
```
src/
├── app/                   # Application code
│   ├── app.component.css     # Root component styles
│   ├── app.component.html    # Root component template
│   ├── app.component.spec.ts # Root component tests
│   ├── app.component.ts      # Root component class
│   ├── app.module.ts         # Root module
│   └── app-routing.module.ts # Routing configuration
├── assets/                # Static assets (images, fonts, etc.)
├── environments/          # Environment-specific configuration
│   ├── environment.ts        # Development environment
│   └── environment.prod.ts   # Production environment
├── index.html            # Main HTML file
├── main.ts               # Application bootstrap
├── polyfills.ts          # Browser compatibility
├── styles.css            # Global styles
└── test.ts               # Testing configuration
```

## 🔍 ไฟล์สำคัญและหน้าที่

### **1. main.ts - Application Bootstrap**
```typescript
// main.ts
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';
import { environment } from './environments/environment';
import { enableProdMode } from '@angular/core';

// Enable production mode if in production environment
if (environment.production) {
  enableProdMode();
}

// Bootstrap the application
platformBrowserDynamic()
  .bootstrapModule(AppModule)
  .catch(err => console.error(err));
```

### **2. index.html - Main HTML Page**
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>MyFirstApp</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <!-- Angular app จะถูก render ใน element นี้ -->
  <app-root></app-root>
</body>
</html>
```

### **3. app.module.ts - Root Module**
```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent    // Components ที่เป็นของ module นี้
  ],
  imports: [
    BrowserModule,        // Module สำหรับรันในเบราว์เซอร์
    AppRoutingModule      // Routing configuration
  ],
  providers: [],          // Services ที่ provide ในระดับ module
  bootstrap: [AppComponent]  // Component ที่จะ bootstrap เมื่อเริ่มแอป
})
export class AppModule { }
```

### **4. app.component.ts - Root Component**
```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',           // CSS selector สำหรับ component
  templateUrl: './app.component.html',  // HTML template
  styleUrls: ['./app.component.css']    // CSS styles
})
export class AppComponent {
  title = 'my-first-app';        // Component property
  
  // Component methods
  onButtonClick(): void {
    console.log('Button clicked!');
  }
}
```

### **5. app.component.html - Component Template**
```html
<!-- app.component.html -->
<div class="app-container">
  <header>
    <h1>Welcome to {{title}}!</h1>
  </header>
  
  <main>
    <p>This is my first Angular application.</p>
    <button (click)="onButtonClick()">Click Me!</button>
  </main>
  
  <footer>
    <p>&copy; 2024 My First App</p>
  </footer>
</div>

<!-- Router outlet สำหรับ routing -->
<router-outlet></router-outlet>
```

### **6. app.component.css - Component Styles**
```css
/* app.component.css */
.app-container {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

header {
  text-align: center;
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

h1 {
  color: #333;
  margin: 0;
}

main {
  background-color: #fff;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin-bottom: 20px;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #0056b3;
}

footer {
  text-align: center;
  color: #666;
  font-size: 14px;
}
```

## 🛠️ Customizing Your First App

### **Step 1: แก้ไข Component Properties**
```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'My Awesome Angular App';
  version = '1.0.0';
  author = 'Your Name';
  currentDate = new Date();
  
  // Application features
  features = [
    'TypeScript Support',
    'Component-Based Architecture',
    'Reactive Programming',
    'Powerful CLI',
    'Rich Ecosystem'
  ];
  
  // Counter example
  counter = 0;
  
  // Methods
  incrementCounter(): void {
    this.counter++;
  }
  
  decrementCounter(): void {
    this.counter--;
  }
  
  resetCounter(): void {
    this.counter = 0;
  }
  
  getCurrentTime(): string {
    return new Date().toLocaleTimeString();
  }
}
```

### **Step 2: อัปเดต Template**
```html
<!-- app.component.html -->
<div class="app-container">
  <!-- Header Section -->
  <header class="app-header">
    <h1>🅰️ {{title}}</h1>
    <p>Version: {{version}} | Author: {{author}}</p>
    <p>Current Time: {{getCurrentTime()}}</p>
  </header>
  
  <!-- Main Content -->
  <main class="app-content">
    <!-- Features Section -->
    <section class="features-section">
      <h2>🌟 Angular Features</h2>
      <ul class="features-list">
        <li *ngFor="let feature of features" class="feature-item">
          {{feature}}
        </li>
      </ul>
    </section>
    
    <!-- Counter Section -->
    <section class="counter-section">
      <h2>🔢 Counter Demo</h2>
      <div class="counter-display">
        <span class="counter-value">{{counter}}</span>
      </div>
      <div class="counter-controls">
        <button (click)="decrementCounter()" class="btn btn-danger">-</button>
        <button (click)="resetCounter()" class="btn btn-secondary">Reset</button>
        <button (click)="incrementCounter()" class="btn btn-success">+</button>
      </div>
    </section>
    
    <!-- Info Section -->
    <section class="info-section">
      <h2>📚 About This App</h2>
      <p>This is my first Angular application built with:</p>
      <div class="tech-stack">
        <span class="tech-item">Angular</span>
        <span class="tech-item">TypeScript</span>
        <span class="tech-item">HTML5</span>
        <span class="tech-item">CSS3</span>
      </div>
    </section>
  </main>
  
  <!-- Footer -->
  <footer class="app-footer">
    <p>&copy; {{currentDate | date:'yyyy'}} {{author}}. Built with ❤️ and Angular.</p>
  </footer>
</div>
```

### **Step 3: ปรับปรุง Styles**
```css
/* app.component.css */
.app-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f8f9fa;
  min-height: 100vh;
}

/* Header Styles */
.app-header {
  text-align: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 30px;
  border-radius: 12px;
  margin-bottom: 30px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
  margin: 0 0 10px 0;
  font-size: 2.5rem;
  font-weight: bold;
}

.app-header p {
  margin: 5px 0;
  opacity: 0.9;
}

/* Main Content */
.app-content {
  display: grid;
  gap: 30px;
}

/* Sections */
section {
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

section h2 {
  margin: 0 0 20px 0;
  color: #333;
  font-size: 1.5rem;
}

/* Features Section */
.features-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
}

.feature-item {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  border-left: 4px solid #007bff;
  font-weight: 500;
}

/* Counter Section */
.counter-section {
  text-align: center;
}

.counter-display {
  margin: 20px 0;
}

.counter-value {
  display: inline-block;
  font-size: 4rem;
  font-weight: bold;
  color: #007bff;
  background: #f8f9fa;
  padding: 20px 40px;
  border-radius: 50%;
  border: 3px solid #007bff;
  min-width: 120px;
}

.counter-controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 20px;
}

/* Buttons */
.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 80px;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-success:hover {
  background-color: #218838;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #c82333;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #545b62;
}

/* Tech Stack */
.tech-stack {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 15px;
}

.tech-item {
  background: #007bff;
  color: white;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
}

/* Footer */
.app-footer {
  text-align: center;
  color: #666;
  font-size: 14px;
  margin-top: 30px;
  padding: 20px;
  border-top: 1px solid #ddd;
}

/* Responsive Design */
@media (max-width: 768px) {
  .app-container {
    padding: 15px;
  }
  
  .app-header h1 {
    font-size: 2rem;
  }
  
  .counter-value {
    font-size: 3rem;
    padding: 15px 25px;
  }
  
  .counter-controls {
    flex-direction: column;
    align-items: center;
  }
  
  .features-list {
    grid-template-columns: 1fr;
  }
  
  .tech-stack {
    justify-content: center;
  }
}
```

## 🧩 Adding New Components

### **Step 1: สร้าง Component ใหม่**
```bash
# สร้าง header component
ng generate component components/header
# หรือ
ng g c components/header

# สร้าง footer component
ng g c components/footer

# สร้าง user-card component
ng g c components/user-card
```

### **Step 2: Header Component**
```typescript
// components/header/header.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html',
  styleUrls: ['./header.component.css']
})
export class HeaderComponent {
  @Input() title: string = 'Default Title';
  @Input() subtitle: string = '';
  
  currentTime: string = '';
  
  constructor() {
    this.updateTime();
    // Update time every second
    setInterval(() => {
      this.updateTime();
    }, 1000);
  }
  
  private updateTime(): void {
    this.currentTime = new Date().toLocaleTimeString();
  }
}
```

```html
<!-- components/header/header.component.html -->
<header class="app-header">
  <div class="header-content">
    <h1>🅰️ {{title}}</h1>
    <p *ngIf="subtitle" class="subtitle">{{subtitle}}</p>
    <div class="time-display">
      <span>🕐 {{currentTime}}</span>
    </div>
  </div>
</header>
```

```css
/* components/header/header.component.css */
.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.header-content {
  text-align: center;
  max-width: 800px;
  margin: 0 auto;
}

.header-content h1 {
  margin: 0 0 10px 0;
  font-size: 2.5rem;
  font-weight: bold;
}

.subtitle {
  font-size: 1.1rem;
  margin: 10px 0;
  opacity: 0.9;
}

.time-display {
  font-family: 'Courier New', monospace;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.2);
  padding: 5px 15px;
  border-radius: 20px;
  display: inline-block;
  margin-top: 10px;
}
```

### **Step 3: User Card Component**
```typescript
// components/user-card/user-card.component.ts
import { Component, Input } from '@angular/core';

export interface User {
  id: number;
  name: string;
  email: string;
  role: string;
  avatar?: string;
  isActive: boolean;
}

@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.css']
})
export class UserCardComponent {
  @Input() user!: User;
  
  getRoleColor(): string {
    switch (this.user?.role?.toLowerCase()) {
      case 'admin': return '#dc3545';
      case 'manager': return '#fd7e14';
      case 'user': return '#28a745';
      default: return '#6c757d';
    }
  }
  
  getStatusText(): string {
    return this.user?.isActive ? 'Active' : 'Inactive';
  }
  
  getAvatarUrl(): string {
    return this.user?.avatar || `https://ui-avatars.com/api/?name=${this.user?.name}&background=007bff&color=fff`;
  }
}
```

```html
<!-- components/user-card/user-card.component.html -->
<div class="user-card" [class.inactive]="!user.isActive">
  <div class="user-avatar">
    <img [src]="getAvatarUrl()" [alt]="user.name + ' avatar'">
  </div>
  
  <div class="user-info">
    <h3 class="user-name">{{user.name}}</h3>
    <p class="user-email">{{user.email}}</p>
    
    <div class="user-meta">
      <span class="user-role" [style.background-color]="getRoleColor()">
        {{user.role}}
      </span>
      <span class="user-status" [class.active]="user.isActive">
        {{getStatusText()}}
      </span>
    </div>
  </div>
</div>
```

```css
/* components/user-card/user-card.component.css */
.user-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  border: 1px solid #e0e0e0;
}

.user-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.user-card.inactive {
  opacity: 0.6;
  background: #f8f9fa;
}

.user-avatar {
  text-align: center;
  margin-bottom: 15px;
}

.user-avatar img {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 3px solid #007bff;
}

.user-info {
  text-align: center;
}

.user-name {
  margin: 0 0 5px 0;
  font-size: 1.2rem;
  color: #333;
}

.user-email {
  margin: 0 0 15px 0;
  color: #666;
  font-size: 0.9rem;
}

.user-meta {
  display: flex;
  justify-content: center;
  gap: 10px;
  flex-wrap: wrap;
}

.user-role {
  color: white;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 0.8rem;
  font-weight: 600;
  text-transform: uppercase;
}

.user-status {
  background: #6c757d;
  color: white;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 0.8rem;
  font-weight: 600;
}

.user-status.active {
  background: #28a745;
}
```

### **Step 4: อัปเดต App Component เพื่อใช้ Components ใหม่**
```typescript
// app.component.ts
import { Component } from '@angular/core';
import { User } from './components/user-card/user-card.component';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'My First Angular App';
  subtitle = 'Learning Angular step by step';
  
  // Sample users data
  users: User[] = [
    {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      role: 'Admin',
      isActive: true
    },
    {
      id: 2,
      name: 'Jane Smith',
      email: 'jane@example.com',
      role: 'Manager',
      isActive: true
    },
    {
      id: 3,
      name: 'Bob Johnson',
      email: 'bob@example.com',
      role: 'User',
      isActive: false
    },
    {
      id: 4,
      name: 'Alice Brown',
      email: 'alice@example.com',
      role: 'User',
      isActive: true
    }
  ];
  
  // Counter functionality
  counter = 0;
  
  incrementCounter(): void {
    this.counter++;
  }
  
  decrementCounter(): void {
    this.counter--;
  }
  
  resetCounter(): void {
    this.counter = 0;
  }
}
```

```html
<!-- app.component.html -->
<div class="app-container">
  <!-- Using Header Component -->
  <app-header [title]="title" [subtitle]="subtitle"></app-header>
  
  <main class="app-content">
    <!-- Counter Section -->
    <section class="counter-section">
      <h2>🔢 Counter Demo</h2>
      <div class="counter-display">
        <span class="counter-value">{{counter}}</span>
      </div>
      <div class="counter-controls">
        <button (click)="decrementCounter()" class="btn btn-danger">-</button>
        <button (click)="resetCounter()" class="btn btn-secondary">Reset</button>
        <button (click)="incrementCounter()" class="btn btn-success">+</button>
      </div>
    </section>
    
    <!-- Users Section -->
    <section class="users-section">
      <h2>👥 Team Members</h2>
      <div class="users-grid">
        <app-user-card 
          *ngFor="let user of users" 
          [user]="user">
        </app-user-card>
      </div>
    </section>
  </main>
</div>
```

```css
/* app.component.css */
.app-container {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f8f9fa;
  min-height: 100vh;
}

.app-content {
  display: grid;
  gap: 30px;
}

section {
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

section h2 {
  margin: 0 0 20px 0;
  color: #333;
  font-size: 1.5rem;
}

/* Counter Section */
.counter-section {
  text-align: center;
}

.counter-display {
  margin: 20px 0;
}

.counter-value {
  display: inline-block;
  font-size: 4rem;
  font-weight: bold;
  color: #007bff;
  background: #f8f9fa;
  padding: 20px 40px;
  border-radius: 50%;
  border: 3px solid #007bff;
  min-width: 120px;
}

.counter-controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 20px;
}

/* Buttons */
.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 80px;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-success:hover {
  background-color: #218838;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #c82333;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #545b62;
}

/* Users Section */
.users-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

/* Responsive Design */
@media (max-width: 768px) {
  .app-container {
    padding: 15px;
  }
  
  .counter-value {
    font-size: 3rem;
    padding: 15px 25px;
  }
  
  .counter-controls {
    flex-direction: column;
    align-items: center;
  }
  
  .users-grid {
    grid-template-columns: 1fr;
  }
}
```

## 🧪 Adding Interactive Features

### **Adding Form Input**
```typescript
// app.component.ts - เพิ่ม import
import { Component } from '@angular/core';
import { User } from './components/user-card/user-card.component';

export class AppComponent {
  // ... existing code ...
  
  // Form data
  newUserName = '';
  newUserEmail = '';
  
  // Add new user
  addUser(): void {
    if (this.newUserName.trim() && this.newUserEmail.trim()) {
      const newUser: User = {
        id: this.users.length + 1,
        name: this.newUserName.trim(),
        email: this.newUserEmail.trim(),
        role: 'User',
        isActive: true
      };
      
      this.users.push(newUser);
      
      // Reset form
      this.newUserName = '';
      this.newUserEmail = '';
    }
  }
  
  // Remove user
  removeUser(userId: number): void {
    this.users = this.users.filter(user => user.id !== userId);
  }
}
```

```html
<!-- app.component.html - เพิ่ม form section -->
<main class="app-content">
  <!-- Add User Form -->
  <section class="form-section">
    <h2>➕ Add New User</h2>
    <form class="user-form" (submit)="addUser(); $event.preventDefault()">
      <div class="form-group">
        <label for="userName">Name:</label>
        <input 
          id="userName"
          type="text" 
          [(ngModel)]="newUserName" 
          placeholder="Enter user name"
          required>
      </div>
      
      <div class="form-group">
        <label for="userEmail">Email:</label>
        <input 
          id="userEmail"
          type="email" 
          [(ngModel)]="newUserEmail" 
          placeholder="Enter user email"
          required>
      </div>
      
      <button 
        type="submit" 
        class="btn btn-primary"
        [disabled]="!newUserName.trim() || !newUserEmail.trim()">
        Add User
      </button>
    </form>
  </section>
  
  <!-- ... existing sections ... -->
</main>
```

**อย่าลืม import FormsModule ใน app.module.ts:**
```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';  // เพิ่มบรรทัดนี้

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HeaderComponent } from './components/header/header.component';
import { UserCardComponent } from './components/user-card/user-card.component';

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    UserCardComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    FormsModule        // เพิ่มบรรทัดนี้
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## 🐛 Debugging และ Development Tools

### **1. Browser DevTools**
```typescript
// เพิ่ม console.log สำหรับ debugging
export class AppComponent {
  incrementCounter(): void {
    this.counter++;
    console.log('Counter incremented:', this.counter);
  }
  
  addUser(): void {
    console.log('Adding user:', this.newUserName, this.newUserEmail);
    // ... rest of the method
  }
}
```

### **2. Angular DevTools**
- ติดตั้ง Angular DevTools browser extension
- ใช้ตรวจสอบ component state และ properties
- ดู component tree และ change detection

### **3. VS Code Debugging**
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Angular",
      "url": "http://localhost:4200",
      "webRoot": "${workspaceFolder}/src",
      "sourceMaps": true
    }
  ]
}
```

## 🧪 แบบฝึกหัด

### **Exercise 1: Todo List Component**
สร้าง Todo List component ที่มีฟีเจอร์:
1. เพิ่ม todo item ใหม่
2. ทำเครื่องหมายเป็น completed
3. ลบ todo item
4. แสดงจำนวน pending และ completed

```bash
ng g c components/todo-list
```

### **Exercise 2: Product Card**
สร้าง Product Card component ที่แสดง:
1. รูปภาพสินค้า
2. ชื่อและราคา
3. คำอธิบาย
4. ปุ่ม Add to Cart

### **Exercise 3: Calculator Component**
สร้าง Calculator component ที่สามารถ:
1. บวก ลบ คูณ หาร
2. แสดงประวัติการคำนวณ
3. Clear และ Reset

## 🧪 Quiz

### **Question 1**
ไฟล์ใดที่เป็น entry point ของ Angular application:
- a) app.component.ts
- b) main.ts
- c) index.html
- d) app.module.ts

### **Question 2**
@Component decorator ประกอบด้วย property ใดบ้าง:
- a) selector เท่านั้น
- b) selector และ template เท่านั้น
- c) selector, template/templateUrl, และ styleUrls
- d) เฉพาะ templateUrl

### **Question 3**
การสร้าง component ใหม่ด้วย Angular CLI ใช้คำสั่ง:
- a) `ng create component`
- b) `ng generate component`
- c) `ng new component`
- d) `ng add component`

**คำตอบ: 1-b, 2-c, 3-b**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🏗️ Project Structure**
1. **Angular project structure** และไฟล์สำคัญ
2. **Component architecture** - template, class, styles
3. **Module system** - organizing application code
4. **Bootstrap process** - จาก main.ts ไปยัง AppComponent

### **🧩 Component Development**
1. **Creating components** ด้วย Angular CLI
2. **Component communication** ด้วย @Input properties
3. **Event handling** และ data binding
4. **Reusable components** design

### **🎨 Styling และ UI**
1. **Component-scoped styles** 
2. **Responsive design** principles
3. **CSS Grid และ Flexbox** integration
4. **Interactive UI elements**

### **🛠️ Development Workflow**
1. **Angular CLI commands** สำหรับ development
2. **Live reload** และ hot module replacement
3. **Debugging techniques** และ dev tools
4. **Code organization** best practices

### **🎯 Next Steps**
ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ Components และ Templates อย่างละเอียด รวมถึง:
- Component lifecycle
- Advanced template syntax
- Component interaction patterns
- Performance optimization

---

**🎉 ยินดีด้วย! คุณได้สร้าง Angular application แรกเรียบร้อยแล้ว**

**[⬅️ บทที่ 3: TypeScript Fundamentals](./03-typescript-fundamentals.md) | [➡️ บทที่ 5: Components and Templates](./05-components-and-templates.md)**
