# บทที่ 5: Components and Templates

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจ Component Lifecycle อย่างลึกซึ้ง
- สร้าง Template ที่ซับซ้อนด้วย Template Syntax
- ใช้ Component Communication แบบต่างๆ
- จัดการ Component State และ Events
- สร้าง Reusable Components ที่มีประสิทธิภาพ

## 🧬 Component Lifecycle

### **Lifecycle Hooks Overview**
```typescript
import { 
  Component, 
  OnInit, 
  OnDestroy, 
  OnChanges, 
  DoCheck,
  AfterContentInit,
  AfterContentChecked,
  AfterViewInit,
  AfterViewChecked,
  SimpleChanges 
} from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  template: `
    <div class="lifecycle-demo">
      <h3>Lifecycle Demo Component</h3>
      <p>Counter: {{counter}}</p>
      <button (click)="increment()">Increment</button>
    </div>
  `
})
export class LifecycleDemoComponent implements 
  OnInit, OnDestroy, OnChanges, DoCheck,
  AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked {
  
  counter = 0;
  
  constructor() {
    console.log('1. Constructor called');
  }
  
  ngOnChanges(changes: SimpleChanges): void {
    console.log('2. ngOnChanges called', changes);
  }
  
  ngOnInit(): void {
    console.log('3. ngOnInit called');
    // Initialization logic here
  }
  
  ngDoCheck(): void {
    console.log('4. ngDoCheck called');
    // Custom change detection logic
  }
  
  ngAfterContentInit(): void {
    console.log('5. ngAfterContentInit called');
    // After content projection
  }
  
  ngAfterContentChecked(): void {
    console.log('6. ngAfterContentChecked called');
    // After content checked
  }
  
  ngAfterViewInit(): void {
    console.log('7. ngAfterViewInit called');
    // After component view initialized
  }
  
  ngAfterViewChecked(): void {
    console.log('8. ngAfterViewChecked called');
    // After component view checked
  }
  
  ngOnDestroy(): void {
    console.log('9. ngOnDestroy called');
    // Cleanup logic here
  }
  
  increment(): void {
    this.counter++;
  }
}
```

### **Practical Lifecycle Usage**
```typescript
import { Component, OnInit, OnDestroy, ViewChild, ElementRef } from '@angular/core';
import { interval, Subscription } from 'rxjs';

@Component({
  selector: 'app-timer',
  template: `
    <div class="timer-container">
      <h3>Timer Component</h3>
      <div class="timer-display" #timerDisplay>
        {{formatTime(elapsedTime)}}
      </div>
      <div class="timer-controls">
        <button (click)="start()" [disabled]="isRunning">Start</button>
        <button (click)="pause()" [disabled]="!isRunning">Pause</button>
        <button (click)="reset()">Reset</button>
      </div>
    </div>
  `,
  styles: [`
    .timer-container {
      text-align: center;
      padding: 20px;
      border: 2px solid #007bff;
      border-radius: 10px;
      margin: 20px;
    }
    
    .timer-display {
      font-size: 3rem;
      font-weight: bold;
      color: #007bff;
      margin: 20px 0;
      padding: 20px;
      background: #f8f9fa;
      border-radius: 10px;
    }
    
    .timer-controls button {
      margin: 0 10px;
      padding: 10px 20px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
  `]
})
export class TimerComponent implements OnInit, OnDestroy {
  @ViewChild('timerDisplay', { static: true }) timerDisplay!: ElementRef;
  
  elapsedTime = 0;
  isRunning = false;
  private subscription?: Subscription;
  
  ngOnInit(): void {
    console.log('Timer component initialized');
    this.updateDisplayColor();
  }
  
  ngOnDestroy(): void {
    console.log('Timer component destroyed');
    this.cleanup();
  }
  
  start(): void {
    if (!this.isRunning) {
      this.isRunning = true;
      this.subscription = interval(1000).subscribe(() => {
        this.elapsedTime++;
        this.updateDisplayColor();
      });
    }
  }
  
  pause(): void {
    this.isRunning = false;
    this.cleanup();
  }
  
  reset(): void {
    this.pause();
    this.elapsedTime = 0;
    this.updateDisplayColor();
  }
  
  formatTime(seconds: number): string {
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = seconds % 60;
    return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
  }
  
  private updateDisplayColor(): void {
    if (this.timerDisplay && this.timerDisplay.nativeElement) {
      const element = this.timerDisplay.nativeElement;
      if (this.elapsedTime > 60) {
        element.style.color = '#dc3545'; // Red after 1 minute
      } else if (this.elapsedTime > 30) {
        element.style.color = '#fd7e14'; // Orange after 30 seconds
      } else {
        element.style.color = '#007bff'; // Blue default
      }
    }
  }
  
  private cleanup(): void {
    if (this.subscription) {
      this.subscription.unsubscribe();
      this.subscription = undefined;
    }
  }
}
```

## 📝 Advanced Template Syntax

### **Template Reference Variables**
```html
<!-- Template Reference Variables -->
<div class="template-refs-demo">
  <h3>Template Reference Variables</h3>
  
  <!-- Basic reference -->
  <input #nameInput type="text" placeholder="Enter your name">
  <button (click)="greet(nameInput.value)">Greet</button>
  
  <!-- Reference to component -->
  <app-timer #timerRef></app-timer>
  <button (click)="timerRef.start()">Start Timer</button>
  <button (click)="timerRef.reset()">Reset Timer</button>
  
  <!-- Reference in loops -->
  <div *ngFor="let item of items; let i = index">
    <input #itemInput [value]="item.name">
    <button (click)="updateItem(i, itemInput.value)">Update</button>
  </div>
</div>
```

### **Safe Navigation Operator**
```html
<!-- Safe Navigation (Elvis Operator) -->
<div class="safe-navigation-demo">
  <!-- Without safe navigation - can cause errors -->
  <!-- <p>{{user.profile.address.street}}</p> -->
  
  <!-- With safe navigation -->
  <p>Street: {{user?.profile?.address?.street}}</p>
  <p>City: {{user?.profile?.address?.city || 'Not specified'}}</p>
  
  <!-- Safe navigation with method calls -->
  <p>Full Name: {{user?.getFullName?.()}}</p>
  
  <!-- Safe navigation in property binding -->
  <img [src]="user?.profile?.avatar || 'assets/default-avatar.png'" 
       [alt]="user?.name + ' avatar'">
</div>
```

### **Template Expressions และ Statements**
```typescript
// Component
export class TemplateExpressionComponent {
  user = {
    name: 'John Doe',
    age: 30,
    isAdmin: true,
    lastLogin: new Date()
  };
  
  products = [
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics' },
    { id: 2, name: 'Book', price: 25, category: 'Education' },
    { id: 3, name: 'Coffee', price: 5, category: 'Food' }
  ];
  
  // Template helper methods
  isExpensive(price: number): boolean {
    return price > 100;
  }
  
  getCategoryColor(category: string): string {
    const colors = {
      'Electronics': '#007bff',
      'Education': '#28a745',
      'Food': '#fd7e14'
    };
    return colors[category as keyof typeof colors] || '#6c757d';
  }
  
  formatCurrency(amount: number): string {
    return new Intl.NumberFormat('th-TH', {
      style: 'currency',
      currency: 'THB'
    }).format(amount);
  }
}
```

```html
<!-- Template Expressions -->
<div class="template-expressions">
  <!-- Simple expressions -->
  <p>User: {{user.name}} ({{user.age}} years old)</p>
  <p>Is Admin: {{user.isAdmin ? 'Yes' : 'No'}}</p>
  
  <!-- Method calls -->
  <p>Last Login: {{user.lastLogin.toDateString()}}</p>
  
  <!-- Complex expressions -->
  <div *ngFor="let product of products" class="product-card">
    <h4 [style.color]="getCategoryColor(product.category)">
      {{product.name}}
    </h4>
    <p>Price: {{formatCurrency(product.price)}}</p>
    <p [class.expensive]="isExpensive(product.price)">
      {{isExpensive(product.price) ? 'Premium' : 'Budget-friendly'}}
    </p>
  </div>
  
  <!-- Arithmetic in templates -->
  <p>Total Products: {{products.length}}</p>
  <p>Average Price: {{formatCurrency(
    products.reduce((sum, p) => sum + p.price, 0) / products.length
  )}}</p>
</div>
```

### **Conditional Rendering**
```html
<!-- Advanced Conditional Rendering -->
<div class="conditional-rendering">
  <!-- ngIf with else -->
  <div *ngIf="user.isAdmin; else userTemplate">
    <h3>Admin Dashboard</h3>
    <button>Manage Users</button>
    <button>System Settings</button>
  </div>
  
  <ng-template #userTemplate>
    <h3>User Dashboard</h3>
    <button>View Profile</button>
    <button>Update Settings</button>
  </ng-template>
  
  <!-- ngIf with then and else -->
  <div *ngIf="products.length > 0; then productList; else emptyState"></div>
  
  <ng-template #productList>
    <h3>Product List ({{products.length}} items)</h3>
    <div *ngFor="let product of products">{{product.name}}</div>
  </ng-template>
  
  <ng-template #emptyState>
    <div class="empty-state">
      <h3>No Products Found</h3>
      <p>Add some products to get started.</p>
      <button>Add Product</button>
    </div>
  </ng-template>
  
  <!-- ngSwitch -->
  <div [ngSwitch]="user.role">
    <div *ngSwitchCase="'admin'">
      <h3>Administrator Panel</h3>
      <p>Full system access</p>
    </div>
    <div *ngSwitchCase="'manager'">
      <h3>Manager Dashboard</h3>
      <p>Department management</p>
    </div>
    <div *ngSwitchCase="'user'">
      <h3>User Profile</h3>
      <p>Personal settings</p>
    </div>
    <div *ngSwitchDefault>
      <h3>Guest Access</h3>
      <p>Limited features</p>
    </div>
  </div>
</div>
```

## 🔄 Component Communication

### **Parent to Child - @Input**
```typescript
// Child Component
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

export interface Product {
  id: number;
  name: string;
  price: number;
  description: string;
  imageUrl: string;
  category: string;
  rating: number;
  inStock: boolean;
}

@Component({
  selector: 'app-product-card',
  template: `
    <div class="product-card" [class.out-of-stock]="!product.inStock">
      <div class="product-image">
        <img [src]="product.imageUrl" [alt]="product.name">
        <div class="product-badge" *ngIf="showBadge">{{badgeText}}</div>
      </div>
      
      <div class="product-info">
        <h3 class="product-name">{{product.name}}</h3>
        <p class="product-description">{{product.description}}</p>
        
        <div class="product-meta">
          <span class="product-price">{{formatPrice(product.price)}}</span>
          <span class="product-rating">
            ⭐ {{product.rating}}/5
          </span>
        </div>
        
        <div class="product-category">
          <span [style.background-color]="categoryColor">
            {{product.category}}
          </span>
        </div>
        
        <div class="product-actions">
          <button 
            [disabled]="!product.inStock"
            (click)="onAddToCart()">
            {{product.inStock ? 'Add to Cart' : 'Out of Stock'}}
          </button>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .product-card {
      border: 1px solid #ddd;
      border-radius: 8px;
      overflow: hidden;
      transition: transform 0.2s, box-shadow 0.2s;
      background: white;
    }
    
    .product-card:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }
    
    .product-card.out-of-stock {
      opacity: 0.6;
    }
    
    .product-image {
      position: relative;
      height: 200px;
      overflow: hidden;
    }
    
    .product-image img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    
    .product-badge {
      position: absolute;
      top: 10px;
      right: 10px;
      background: #dc3545;
      color: white;
      padding: 5px 10px;
      border-radius: 4px;
      font-size: 12px;
      font-weight: bold;
    }
    
    .product-info {
      padding: 15px;
    }
    
    .product-name {
      margin: 0 0 10px 0;
      color: #333;
    }
    
    .product-description {
      color: #666;
      font-size: 14px;
      margin-bottom: 15px;
    }
    
    .product-meta {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
    }
    
    .product-price {
      font-size: 18px;
      font-weight: bold;
      color: #007bff;
    }
    
    .product-category span {
      color: white;
      padding: 4px 8px;
      border-radius: 4px;
      font-size: 12px;
      font-weight: bold;
    }
    
    .product-actions {
      margin-top: 15px;
    }
    
    .product-actions button {
      width: 100%;
      padding: 10px;
      border: none;
      border-radius: 4px;
      background: #007bff;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }
    
    .product-actions button:disabled {
      background: #6c757d;
      cursor: not-allowed;
    }
    
    .product-actions button:not(:disabled):hover {
      background: #0056b3;
    }
  `]
})
export class ProductCardComponent implements OnChanges {
  @Input() product!: Product;
  @Input() showBadge = false;
  @Input() badgeText = 'NEW';
  @Input() categoryColor = '#6c757d';
  
  ngOnChanges(changes: SimpleChanges): void {
    // React to input changes
    if (changes['product']) {
      console.log('Product changed:', changes['product'].currentValue);
    }
  }
  
  formatPrice(price: number): string {
    return new Intl.NumberFormat('th-TH', {
      style: 'currency',
      currency: 'THB'
    }).format(price);
  }
  
  onAddToCart(): void {
    if (this.product.inStock) {
      console.log('Adding to cart:', this.product.name);
      // This will be connected to @Output later
    }
  }
}
```

### **Child to Parent - @Output**
```typescript
// Enhanced Product Card with Output Events
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-product-card',
  template: `
    <!-- Previous template with updated buttons -->
    <div class="product-actions">
      <button 
        [disabled]="!product.inStock"
        (click)="onAddToCart()"
        class="btn-primary">
        {{product.inStock ? 'Add to Cart' : 'Out of Stock'}}
      </button>
      
      <button 
        (click)="onViewDetails()"
        class="btn-secondary">
        View Details
      </button>
      
      <button 
        (click)="onToggleFavorite()"
        class="btn-icon"
        [class.favorited]="isFavorited">
        {{isFavorited ? '❤️' : '🤍'}}
      </button>
    </div>
  `
})
export class ProductCardComponent {
  @Input() product!: Product;
  @Input() isFavorited = false;
  
  // Output events
  @Output() addToCart = new EventEmitter<Product>();
  @Output() viewDetails = new EventEmitter<number>();
  @Output() toggleFavorite = new EventEmitter<{productId: number, isFavorited: boolean}>();
  @Output() ratingChanged = new EventEmitter<{productId: number, rating: number}>();
  
  onAddToCart(): void {
    if (this.product.inStock) {
      this.addToCart.emit(this.product);
    }
  }
  
  onViewDetails(): void {
    this.viewDetails.emit(this.product.id);
  }
  
  onToggleFavorite(): void {
    this.isFavorited = !this.isFavorited;
    this.toggleFavorite.emit({
      productId: this.product.id,
      isFavorited: this.isFavorited
    });
  }
  
  onRatingClick(rating: number): void {
    this.ratingChanged.emit({
      productId: this.product.id,
      rating: rating
    });
  }
}

// Parent Component
@Component({
  selector: 'app-product-list',
  template: `
    <div class="product-list">
      <h2>Product Catalog</h2>
      
      <div class="products-grid">
        <app-product-card
          *ngFor="let product of products"
          [product]="product"
          [isFavorited]="isProductFavorited(product.id)"
          (addToCart)="handleAddToCart($event)"
          (viewDetails)="handleViewDetails($event)"
          (toggleFavorite)="handleToggleFavorite($event)"
          (ratingChanged)="handleRatingChanged($event)">
        </app-product-card>
      </div>
      
      <!-- Cart Summary -->
      <div class="cart-summary" *ngIf="cartItems.length > 0">
        <h3>Cart ({{cartItems.length}} items)</h3>
        <p>Total: {{formatPrice(getTotalPrice())}}</p>
      </div>
    </div>
  `
})
export class ProductListComponent {
  products: Product[] = [
    {
      id: 1,
      name: 'MacBook Pro',
      price: 65000,
      description: 'Powerful laptop for professionals',
      imageUrl: 'assets/macbook.jpg',
      category: 'Electronics',
      rating: 4.8,
      inStock: true
    },
    // ... more products
  ];
  
  cartItems: Product[] = [];
  favoriteProductIds: Set<number> = new Set();
  
  handleAddToCart(product: Product): void {
    console.log('Adding to cart:', product.name);
    this.cartItems.push(product);
    // Show notification, update cart service, etc.
  }
  
  handleViewDetails(productId: number): void {
    console.log('Viewing details for product:', productId);
    // Navigate to product details page
  }
  
  handleToggleFavorite(event: {productId: number, isFavorited: boolean}): void {
    if (event.isFavorited) {
      this.favoriteProductIds.add(event.productId);
    } else {
      this.favoriteProductIds.delete(event.productId);
    }
    console.log('Favorite toggled:', event);
  }
  
  handleRatingChanged(event: {productId: number, rating: number}): void {
    console.log('Rating changed:', event);
    // Update product rating
    const product = this.products.find(p => p.id === event.productId);
    if (product) {
      product.rating = event.rating;
    }
  }
  
  isProductFavorited(productId: number): boolean {
    return this.favoriteProductIds.has(productId);
  }
  
  getTotalPrice(): number {
    return this.cartItems.reduce((total, item) => total + item.price, 0);
  }
  
  formatPrice(price: number): string {
    return new Intl.NumberFormat('th-TH', {
      style: 'currency',
      currency: 'THB'
    }).format(price);
  }
}
```

### **Template-driven vs Reactive Communication**
```typescript
// Parent Component with different communication patterns
@Component({
  selector: 'app-communication-demo',
  template: `
    <div class="communication-demo">
      <!-- Method 1: Template Reference Variables -->
      <h3>Method 1: Template Reference</h3>
      <app-child-component #childRef></app-child-component>
      <button (click)="childRef.doSomething()">Call Child Method</button>
      <p>Child Value: {{childRef.value}}</p>
      
      <!-- Method 2: @Input/@Output -->
      <h3>Method 2: Input/Output</h3>
      <app-child-component
        [inputValue]="parentValue"
        (outputEvent)="handleChildEvent($event)">
      </app-child-component>
      
      <!-- Method 3: ViewChild -->
      <h3>Method 3: ViewChild</h3>
      <app-child-component #viewChildRef></app-child-component>
      <button (click)="accessChildViaViewChild()">Access via ViewChild</button>
    </div>
  `
})
export class CommunicationDemoComponent implements AfterViewInit {
  @ViewChild('viewChildRef') childComponent!: ChildComponent;
  
  parentValue = 'Hello from Parent';
  
  ngAfterViewInit(): void {
    // Access child component after view initialization
    console.log('Child component:', this.childComponent);
  }
  
  handleChildEvent(data: any): void {
    console.log('Received from child:', data);
  }
  
  accessChildViaViewChild(): void {
    if (this.childComponent) {
      this.childComponent.doSomething();
      console.log('Child value:', this.childComponent.value);
    }
  }
}
```

## 🔍 ViewChild และ ViewChildren

### **ViewChild Example**
```typescript
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-viewchild-demo',
  template: `
    <div class="viewchild-demo">
      <h3>ViewChild Demo</h3>
      
      <!-- Element Reference -->
      <input #nameInput type="text" placeholder="Enter name">
      <button (click)="focusInput()">Focus Input</button>
      
      <!-- Component Reference -->
      <app-timer #timerComponent></app-timer>
      <button (click)="controlTimer()">{{timerRunning ? 'Stop' : 'Start'}} Timer</button>
      
      <!-- Canvas Example -->
      <canvas #myCanvas width="300" height="200" style="border: 1px solid #ccc;"></canvas>
      <button (click)="drawOnCanvas()">Draw Circle</button>
    </div>
  `
})
export class ViewChildDemoComponent implements AfterViewInit {
  @ViewChild('nameInput') nameInput!: ElementRef<HTMLInputElement>;
  @ViewChild('timerComponent') timerComponent!: TimerComponent;
  @ViewChild('myCanvas') canvas!: ElementRef<HTMLCanvasElement>;
  
  timerRunning = false;
  
  ngAfterViewInit(): void {
    // ViewChild elements are available here
    console.log('Name input element:', this.nameInput.nativeElement);
    console.log('Timer component:', this.timerComponent);
    console.log('Canvas element:', this.canvas.nativeElement);
  }
  
  focusInput(): void {
    this.nameInput.nativeElement.focus();
    this.nameInput.nativeElement.select();
  }
  
  controlTimer(): void {
    if (this.timerRunning) {
      this.timerComponent.pause();
    } else {
      this.timerComponent.start();
    }
    this.timerRunning = !this.timerRunning;
  }
  
  drawOnCanvas(): void {
    const ctx = this.canvas.nativeElement.getContext('2d');
    if (ctx) {
      ctx.clearRect(0, 0, 300, 200);
      ctx.beginPath();
      ctx.arc(150, 100, 50, 0, 2 * Math.PI);
      ctx.fillStyle = '#007bff';
      ctx.fill();
      ctx.stroke();
    }
  }
}
```

### **ViewChildren Example**
```typescript
import { Component, ViewChildren, QueryList, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-viewchildren-demo',
  template: `
    <div class="viewchildren-demo">
      <h3>ViewChildren Demo</h3>
      
      <div class="item-list">
        <div class="item" #listItem *ngFor="let item of items; let i = index">
          <h4>{{item.title}}</h4>
          <p>{{item.description}}</p>
          <button (click)="highlightItem(i)">Highlight</button>
        </div>
      </div>
      
      <div class="controls">
        <button (click)="highlightAll()">Highlight All</button>
        <button (click)="clearHighlights()">Clear Highlights</button>
        <button (click)="addItem()">Add Item</button>
      </div>
      
      <!-- Multiple Timer Components -->
      <div class="timers">
        <app-timer *ngFor="let timer of timers; let i = index" 
                   [title]="'Timer ' + (i + 1)">
        </app-timer>
      </div>
      
      <div class="timer-controls">
        <button (click)="startAllTimers()">Start All Timers</button>
        <button (click)="resetAllTimers()">Reset All Timers</button>
      </div>
    </div>
  `,
  styles: [`
    .item {
      border: 1px solid #ddd;
      padding: 15px;
      margin: 10px 0;
      border-radius: 5px;
      transition: background-color 0.3s;
    }
    
    .item.highlighted {
      background-color: #fff3cd;
      border-color: #ffeaa7;
    }
    
    .controls, .timer-controls {
      margin: 20px 0;
    }
    
    .controls button, .timer-controls button {
      margin-right: 10px;
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      background: #007bff;
      color: white;
      cursor: pointer;
    }
    
    .timers {
      display: flex;
      flex-wrap: wrap;
      gap: 20px;
      margin: 20px 0;
    }
  `]
})
export class ViewChildrenDemoComponent implements AfterViewInit {
  @ViewChildren('listItem') listItems!: QueryList<ElementRef>;
  @ViewChildren(TimerComponent) timerComponents!: QueryList<TimerComponent>;
  
  items = [
    { title: 'Item 1', description: 'First item description' },
    { title: 'Item 2', description: 'Second item description' },
    { title: 'Item 3', description: 'Third item description' }
  ];
  
  timers = [1, 2, 3]; // Just for creating multiple timer instances
  
  ngAfterViewInit(): void {
    console.log('List items:', this.listItems);
    console.log('Timer components:', this.timerComponents);
    
    // Subscribe to changes in QueryList
    this.listItems.changes.subscribe(() => {
      console.log('List items changed:', this.listItems.length);
    });
    
    this.timerComponents.changes.subscribe(() => {
      console.log('Timer components changed:', this.timerComponents.length);
    });
  }
  
  highlightItem(index: number): void {
    const item = this.listItems.toArray()[index];
    if (item) {
      item.nativeElement.classList.add('highlighted');
      setTimeout(() => {
        item.nativeElement.classList.remove('highlighted');
      }, 2000);
    }
  }
  
  highlightAll(): void {
    this.listItems.forEach(item => {
      item.nativeElement.classList.add('highlighted');
    });
  }
  
  clearHighlights(): void {
    this.listItems.forEach(item => {
      item.nativeElement.classList.remove('highlighted');
    });
  }
  
  addItem(): void {
    const newIndex = this.items.length + 1;
    this.items.push({
      title: `Item ${newIndex}`,
      description: `Description for item ${newIndex}`
    });
  }
  
  startAllTimers(): void {
    this.timerComponents.forEach(timer => {
      timer.start();
    });
  }
  
  resetAllTimers(): void {
    this.timerComponents.forEach(timer => {
      timer.reset();
    });
  }
}
```

## 🎨 Dynamic Component Creation

### **ComponentFactoryResolver (Legacy)**
```typescript
import { 
  Component, 
  ComponentFactoryResolver, 
  ViewChild, 
  ViewContainerRef,
  Type
} from '@angular/core';

// Dynamic component example
@Component({
  selector: 'app-alert',
  template: `
    <div class="alert" [class]="'alert-' + type">
      <strong>{{title}}</strong>
      <p>{{message}}</p>
      <button (click)="close()">Close</button>
    </div>
  `,
  styles: [`
    .alert {
      padding: 15px;
      margin: 10px 0;
      border-radius: 4px;
      border: 1px solid;
    }
    
    .alert-success {
      background: #d4edda;
      border-color: #c3e6cb;
      color: #155724;
    }
    
    .alert-error {
      background: #f8d7da;
      border-color: #f5c6cb;
      color: #721c24;
    }
    
    .alert button {
      float: right;
      background: none;
      border: none;
      cursor: pointer;
    }
  `]
})
export class AlertComponent {
  type: 'success' | 'error' = 'success';
  title = '';
  message = '';
  
  close(): void {
    // Emit close event or remove component
  }
}

@Component({
  selector: 'app-dynamic-component',
  template: `
    <div class="dynamic-component-demo">
      <h3>Dynamic Component Creation</h3>
      
      <div class="controls">
        <button (click)="createSuccessAlert()">Create Success Alert</button>
        <button (click)="createErrorAlert()">Create Error Alert</button>
        <button (click)="clearAlerts()">Clear All Alerts</button>
      </div>
      
      <!-- Container for dynamic components -->
      <div #alertContainer class="alert-container"></div>
    </div>
  `,
  styles: [`
    .controls button {
      margin-right: 10px;
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    
    .alert-container {
      margin-top: 20px;
    }
  `]
})
export class DynamicComponentDemoComponent {
  @ViewChild('alertContainer', { read: ViewContainerRef }) 
  alertContainer!: ViewContainerRef;
  
  constructor(private componentFactoryResolver: ComponentFactoryResolver) {}
  
  createSuccessAlert(): void {
    this.createAlert('success', 'Success!', 'Operation completed successfully.');
  }
  
  createErrorAlert(): void {
    this.createAlert('error', 'Error!', 'Something went wrong.');
  }
  
  private createAlert(type: 'success' | 'error', title: string, message: string): void {
    // Create component factory
    const factory = this.componentFactoryResolver.resolveComponentFactory(AlertComponent);
    
    // Create component instance
    const componentRef = this.alertContainer.createComponent(factory);
    
    // Set component properties
    componentRef.instance.type = type;
    componentRef.instance.title = title;
    componentRef.instance.message = message;
    
    // Auto-remove after 5 seconds
    setTimeout(() => {
      componentRef.destroy();
    }, 5000);
  }
  
  clearAlerts(): void {
    this.alertContainer.clear();
  }
}
```

### **Modern Dynamic Component Creation (Angular 13+)**
```typescript
import { Component, ViewChild, ViewContainerRef, createComponent, EnvironmentInjector } from '@angular/core';

@Component({
  selector: 'app-modern-dynamic',
  template: `
    <div class="modern-dynamic-demo">
      <h3>Modern Dynamic Component Creation</h3>
      
      <div class="controls">
        <button (click)="createComponent()">Create Component</button>
        <button (click)="clearComponents()">Clear All</button>
      </div>
      
      <div #container class="component-container"></div>
    </div>
  `
})
export class ModernDynamicComponentDemo {
  @ViewChild('container', { read: ViewContainerRef }) 
  container!: ViewContainerRef;
  
  constructor(private envInjector: EnvironmentInjector) {}
  
  createComponent(): void {
    const componentRef = createComponent(AlertComponent, {
      environmentInjector: this.envInjector,
      hostElement: this.container.element.nativeElement
    });
    
    // Set properties
    componentRef.instance.type = 'success';
    componentRef.instance.title = 'Modern Alert';
    componentRef.instance.message = 'Created with modern API!';
    
    // Append to DOM
    this.container.insert(componentRef.hostView);
    
    // Auto-destroy
    setTimeout(() => {
      componentRef.destroy();
    }, 3000);
  }
  
  clearComponents(): void {
    this.container.clear();
  }
}
```

## 🧪 แบบฝึกหัด

### **Exercise 1: Advanced Todo Component**
สร้าง Todo component ที่มี:
1. Lifecycle hooks สำหรับ auto-save
2. ViewChild สำหรับ focus input
3. Parent-child communication
4. Dynamic component creation สำหรับ todo items

### **Exercise 2: Dashboard Widget System**
สร้างระบบ Dashboard ที่:
1. สร้าง widgets แบบ dynamic
2. ใช้ ViewChildren เพื่อจัดการ widgets
3. Communication ระหว่าง widgets
4. Save/load widget configurations

### **Exercise 3: Image Gallery Component**
สร้าง Image Gallery ที่มี:
1. Lazy loading สำหรับรูปภาพ
2. Template reference variables สำหรับ modal
3. Output events สำหรับ image selection
4. ViewChild สำหรับ canvas manipulation

## 🧪 Quiz

### **Question 1**
Lifecycle hook ใดที่ถูกเรียกหลังจาก ngOnInit:
- a) ngOnChanges
- b) ngDoCheck
- c) ngAfterContentInit
- d) constructor

### **Question 2**
ViewChild จะมีค่าเมื่อไหร่:
- a) ใน constructor
- b) ใน ngOnInit
- c) ใน ngAfterViewInit
- d) ใน ngOnDestroy

### **Question 3**
การส่ง data จาก child ไป parent ใช้:
- a) @Input
- b) @Output
- c) ViewChild
- d) Service

**คำตอบ: 1-b, 2-c, 3-b**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🧬 Component Lifecycle**
1. **Lifecycle hooks** - constructor, ngOnInit, ngOnDestroy, etc.
2. **Practical usage** - initialization, cleanup, change detection
3. **Best practices** - เมื่อไหร่ใช้ hook ไหน

### **📝 Advanced Templates**
1. **Template syntax** - expressions, statements, operators
2. **Conditional rendering** - ngIf, ngSwitch, ng-template
3. **Template references** - accessing DOM elements และ components

### **🔄 Component Communication**
1. **@Input/@Output** - parent-child communication
2. **ViewChild/ViewChildren** - accessing child components
3. **Template references** - direct component access
4. **Event emitters** - custom events

### **🎨 Dynamic Components**
1. **ComponentFactoryResolver** (legacy approach)
2. **Modern createComponent API** (Angular 13+)
3. **ViewContainerRef** - dynamic content injection
4. **Use cases** - modals, alerts, dynamic forms

### **🎯 Next Steps**
ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ Data Binding และ Directives:
- Two-way data binding
- Built-in directives
- Custom directives
- Attribute vs Structural directives

---

**🎉 ยินดีด้วย! ตอนนี้คุณเข้าใจ Components และ Templates อย่างลึกซึ้งแล้ว**

**[⬅️ บทที่ 4: First Angular Application](./04-first-angular-application.md) | [➡️ บทที่ 6: Data Binding and Directives](./06-data-binding-and-directives.md)**
