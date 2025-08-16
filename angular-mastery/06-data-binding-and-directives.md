# บทที่ 6: Data Binding and Directives

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจและใช้ Data Binding ทุกประเภทได้อย่างเชี่ยวชาญ
- ใช้ Built-in Directives อย่างมีประสิทธิภาพ
- สร้าง Custom Directives สำหรับความต้องการเฉพาะ
- เข้าใจความแตกต่างระหว่าง Structural และ Attribute Directives
- ออกแบบ Interactive UI ด้วย Two-way Data Binding

## 🔗 Data Binding Overview

Data Binding เป็นหัวใจของ Angular ที่เชื่อมต่อ Component Class กับ Template

### **Data Binding Types**
```mermaid
graph LR
    A[Component Class] --> B[Template]
    B --> A
    
    A --> C[Interpolation {{}}]
    A --> D[Property Binding []]
    B --> E[Event Binding ()]
    A <--> F[Two-way Binding [()]]
    
    style A fill:#ff9999
    style B fill:#99ff99
    style F fill:#9999ff
```

## 📊 Interpolation

### **Basic Interpolation**
```typescript
// Component
@Component({
  selector: 'app-interpolation-demo',
  template: `
    <div class="interpolation-demo">
      <h3>Interpolation Examples</h3>
      
      <!-- Basic interpolation -->
      <p>Hello, {{userName}}!</p>
      <p>Today is {{currentDate}}</p>
      
      <!-- Expressions -->
      <p>2 + 3 = {{2 + 3}}</p>
      <p>User age: {{user.age}} years old</p>
      
      <!-- Method calls -->
      <p>Uppercase name: {{getUserNameUpperCase()}}</p>
      <p>Random number: {{getRandomNumber()}}</p>
      
      <!-- Conditional expressions -->
      <p>Status: {{isLoggedIn ? 'Logged In' : 'Logged Out'}}</p>
      <p>Message: {{message || 'No message available'}}</p>
      
      <!-- Complex expressions -->
      <p>Full Name: {{user.firstName + ' ' + user.lastName}}</p>
      <p>Total Items: {{items.length > 0 ? items.length + ' items' : 'No items'}}</p>
      
      <!-- Safe navigation -->
      <p>City: {{user.address?.city || 'City not specified'}}</p>
      <p>Profile Image: {{user.profile?.avatar?.url || 'No image'}}</p>
    </div>
  `,
  styles: [`
    .interpolation-demo {
      padding: 20px;
      background: #f8f9fa;
      border-radius: 8px;
      margin: 20px 0;
    }
    
    .interpolation-demo p {
      margin: 8px 0;
      padding: 5px;
      background: white;
      border-left: 3px solid #007bff;
      padding-left: 15px;
    }
  `]
})
export class InterpolationDemoComponent {
  userName = 'John Doe';
  currentDate = new Date().toLocaleDateString('th-TH');
  isLoggedIn = true;
  message = '';
  
  user = {
    firstName: 'John',
    lastName: 'Doe',
    age: 30,
    address: {
      city: 'Bangkok',
      country: 'Thailand'
    },
    profile: {
      avatar: {
        url: 'https://example.com/avatar.jpg'
      }
    }
  };
  
  items = ['Item 1', 'Item 2', 'Item 3'];
  
  getUserNameUpperCase(): string {
    return this.userName.toUpperCase();
  }
  
  getRandomNumber(): number {
    return Math.floor(Math.random() * 100);
  }
}
```

### **Advanced Interpolation with Pipes**
```typescript
@Component({
  selector: 'app-interpolation-pipes',
  template: `
    <div class="pipes-demo">
      <h3>Interpolation with Pipes</h3>
      
      <!-- Date pipes -->
      <p>Date: {{today | date}}</p>
      <p>Short Date: {{today | date:'short'}}</p>
      <p>Custom Date: {{today | date:'dd/MM/yyyy HH:mm'}}</p>
      <p>Thai Date: {{today | date:'fullDate':'':'th-TH'}}</p>
      
      <!-- Number pipes -->
      <p>Price: {{product.price | currency:'THB':'symbol':'1.2-2'}}</p>
      <p>Percentage: {{user.progress | percent:'1.0-1'}}</p>
      <p>Decimal: {{pi | number:'3.1-5'}}</p>
      
      <!-- String pipes -->
      <p>Uppercase: {{user.name | uppercase}}</p>
      <p>Lowercase: {{user.name | lowercase}}</p>
      <p>Title Case: {{user.fullName | titlecase}}</p>
      
      <!-- JSON pipe for debugging -->
      <pre>User Object: {{user | json}}</pre>
      
      <!-- Slice pipe -->
      <p>Short Description: {{product.description | slice:0:50}}...</p>
      
      <!-- Async pipe preview -->
      <p>Observable Value: {{observableValue | async}}</p>
      
      <!-- Custom pipe -->
      <p>Phone: {{user.phone | phoneFormat}}</p>
    </div>
  `
})
export class InterpolationPipesComponent implements OnInit {
  today = new Date();
  pi = Math.PI;
  
  user = {
    name: 'john doe',
    fullName: 'john smith doe',
    progress: 0.75,
    phone: '0812345678'
  };
  
  product = {
    price: 1299.99,
    description: 'This is a long product description that needs to be truncated for display purposes.'
  };
  
  observableValue = new BehaviorSubject('Initial Value');
  
  ngOnInit(): void {
    // Simulate observable updates
    setTimeout(() => {
      this.observableValue.next('Updated Value');
    }, 2000);
  }
}

// Custom Pipe Example
@Pipe({ name: 'phoneFormat' })
export class PhoneFormatPipe implements PipeTransform {
  transform(value: string): string {
    if (!value) return value;
    
    // Format: 081-234-5678
    return value.replace(/(\d{3})(\d{3})(\d{4})/, '$1-$2-$3');
  }
}
```

## 🎛️ Property Binding

### **Basic Property Binding**
```typescript
@Component({
  selector: 'app-property-binding',
  template: `
    <div class="property-binding-demo">
      <h3>Property Binding Examples</h3>
      
      <!-- Element properties -->
      <img [src]="imageUrl" 
           [alt]="imageAlt"
           [width]="imageWidth"
           [height]="imageHeight">
      
      <!-- Disabled state -->
      <button [disabled]="isButtonDisabled">
        {{isButtonDisabled ? 'Disabled Button' : 'Active Button'}}
      </button>
      
      <!-- Hidden attribute -->
      <div [hidden]="!showMessage">
        <p>This message can be hidden</p>
      </div>
      
      <!-- Class binding -->
      <div [class]="dynamicClass">Dynamic Class</div>
      <div [class.active]="isActive">Conditional Class</div>
      <div [ngClass]="getClassObject()">Multiple Classes</div>
      
      <!-- Style binding -->
      <div [style.color]="textColor">Colored Text</div>
      <div [style.font-size.px]="fontSize">Sized Text</div>
      <div [ngStyle]="getStyleObject()">Multiple Styles</div>
      
      <!-- Attribute binding -->
      <div [attr.data-id]="user.id"
           [attr.title]="user.tooltip"
           [attr.aria-label]="user.ariaLabel">
        User Info
      </div>
      
      <!-- Form elements -->
      <input [value]="inputValue"
             [placeholder]="inputPlaceholder"
             [readonly]="isReadonly">
      
      <select [selectedIndex]="selectedOptionIndex">
        <option *ngFor="let option of options" [value]="option.value">
          {{option.label}}
        </option>
      </select>
    </div>
  `,
  styles: [`
    .property-binding-demo {
      padding: 20px;
    }
    
    .property-binding-demo > * {
      margin: 15px 0;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    .active {
      background-color: #d4edda;
      border-color: #c3e6cb;
    }
    
    button:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }
  `]
})
export class PropertyBindingComponent {
  // Image properties
  imageUrl = 'https://via.placeholder.com/200x150';
  imageAlt = 'Placeholder Image';
  imageWidth = 200;
  imageHeight = 150;
  
  // Button state
  isButtonDisabled = false;
  
  // Visibility
  showMessage = true;
  
  // Class and style properties
  dynamicClass = 'highlight-box';
  isActive = true;
  textColor = '#007bff';
  fontSize = 18;
  
  // User data
  user = {
    id: 123,
    tooltip: 'User information tooltip',
    ariaLabel: 'User details section'
  };
  
  // Form properties
  inputValue = 'Default Value';
  inputPlaceholder = 'Enter text here...';
  isReadonly = false;
  selectedOptionIndex = 1;
  
  options = [
    { value: 'option1', label: 'Option 1' },
    { value: 'option2', label: 'Option 2' },
    { value: 'option3', label: 'Option 3' }
  ];
  
  getClassObject(): object {
    return {
      'text-primary': this.isActive,
      'text-muted': !this.isActive,
      'font-weight-bold': true,
      'border-rounded': this.isActive
    };
  }
  
  getStyleObject(): object {
    return {
      'background-color': this.isActive ? '#e3f2fd' : '#f5f5f5',
      'padding': '15px',
      'border-radius': '8px',
      'transition': 'all 0.3s ease'
    };
  }
}
```

### **Advanced Property Binding**
```typescript
@Component({
  selector: 'app-advanced-property-binding',
  template: `
    <div class="advanced-property-binding">
      <h3>Advanced Property Binding</h3>
      
      <!-- Component property binding -->
      <app-user-card 
        [user]="selectedUser"
        [showActions]="canEditUser"
        [theme]="currentTheme"
        [config]="cardConfig">
      </app-user-card>
      
      <!-- Dynamic component properties -->
      <div [innerHTML]="sanitizedHtml"></div>
      
      <!-- Complex object binding -->
      <app-chart 
        [data]="chartData"
        [options]="chartOptions"
        [style.height.px]="chartHeight">
      </app-chart>
      
      <!-- Conditional property binding -->
      <input 
        [type]="isPasswordVisible ? 'text' : 'password'"
        [class.form-control]="true"
        [class.is-invalid]="hasValidationError"
        [attr.autocomplete]="isPasswordField ? 'current-password' : 'off'">
      
      <!-- Array and object property binding -->
      <app-data-table
        [columns]="tableColumns"
        [data]="tableData"
        [sortBy]="sortConfiguration"
        [pagination]="paginationSettings">
      </app-data-table>
    </div>
  `
})
export class AdvancedPropertyBindingComponent {
  selectedUser = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    role: 'Admin'
  };
  
  canEditUser = true;
  currentTheme = 'dark';
  
  cardConfig = {
    showAvatar: true,
    avatarSize: 'large',
    showBadge: true,
    animationEnabled: true
  };
  
  // Sanitized HTML content
  sanitizedHtml = `
    <div class="content-box">
      <h4>Dynamic Content</h4>
      <p>This content is <strong>dynamically</strong> inserted.</p>
    </div>
  `;
  
  // Chart configuration
  chartData = [
    { label: 'January', value: 30 },
    { label: 'February', value: 45 },
    { label: 'March', value: 28 }
  ];
  
  chartOptions = {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      y: { beginAtZero: true }
    }
  };
  
  chartHeight = 300;
  
  // Form state
  isPasswordVisible = false;
  hasValidationError = false;
  isPasswordField = true;
  
  // Table configuration
  tableColumns = [
    { key: 'id', label: 'ID', sortable: true },
    { key: 'name', label: 'Name', sortable: true },
    { key: 'email', label: 'Email', sortable: false },
    { key: 'createdAt', label: 'Created', sortable: true }
  ];
  
  tableData = [
    { id: 1, name: 'John', email: 'john@example.com', createdAt: new Date() },
    { id: 2, name: 'Jane', email: 'jane@example.com', createdAt: new Date() }
  ];
  
  sortConfiguration = {
    column: 'name',
    direction: 'asc'
  };
  
  paginationSettings = {
    currentPage: 1,
    pageSize: 10,
    totalItems: 100
  };
}
```

## 🖱️ Event Binding

### **Basic Event Binding**
```typescript
@Component({
  selector: 'app-event-binding',
  template: `
    <div class="event-binding-demo">
      <h3>Event Binding Examples</h3>
      
      <!-- Basic click events -->
      <button (click)="onButtonClick()">Basic Click</button>
      <button (click)="onButtonClickWithParam('Hello')">Click with Parameter</button>
      
      <!-- Event object -->
      <button (click)="onButtonClickWithEvent($event)">Click with Event</button>
      
      <!-- Form events -->
      <input (focus)="onInputFocus()" 
             (blur)="onInputBlur()"
             (input)="onInputChange($event)"
             (keyup)="onKeyUp($event)"
             (keydown)="onKeyDown($event)"
             [value]="inputValue">
      
      <!-- Mouse events -->
      <div class="mouse-area"
           (mouseenter)="onMouseEnter()"
           (mouseleave)="onMouseLeave()"
           (mousemove)="onMouseMove($event)"
           (click)="onAreaClick($event)">
        Mouse Area - Hover and Click Me!
        <p>Mouse Position: {{mousePosition.x}}, {{mousePosition.y}}</p>
      </div>
      
      <!-- Custom events -->
      <app-custom-component 
        (customEvent)="onCustomEvent($event)"
        (dataChanged)="onDataChanged($event)">
      </app-custom-component>
      
      <!-- Form submission -->
      <form (submit)="onFormSubmit($event)" (reset)="onFormReset()">
        <input name="username" [(ngModel)]="formData.username">
        <button type="submit">Submit</button>
        <button type="reset">Reset</button>
      </form>
      
      <!-- Event Log -->
      <div class="event-log">
        <h4>Event Log:</h4>
        <ul>
          <li *ngFor="let event of eventLog; let i = index">
            {{i + 1}}. {{event}}
          </li>
        </ul>
        <button (click)="clearEventLog()">Clear Log</button>
      </div>
    </div>
  `,
  styles: [`
    .event-binding-demo {
      padding: 20px;
    }
    
    .mouse-area {
      width: 300px;
      height: 150px;
      background: #f8f9fa;
      border: 2px dashed #007bff;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      margin: 20px 0;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    
    .mouse-area:hover {
      background: #e3f2fd;
    }
    
    .event-log {
      background: #f8f9fa;
      padding: 15px;
      border-radius: 4px;
      margin-top: 20px;
      max-height: 200px;
      overflow-y: auto;
    }
    
    .event-log ul {
      list-style-type: none;
      padding: 0;
      margin: 0;
    }
    
    .event-log li {
      padding: 2px 0;
      font-family: monospace;
      font-size: 12px;
    }
    
    button, input {
      margin: 5px;
      padding: 8px 12px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    button {
      background: #007bff;
      color: white;
      cursor: pointer;
    }
    
    button:hover {
      background: #0056b3;
    }
  `]
})
export class EventBindingComponent {
  inputValue = '';
  mousePosition = { x: 0, y: 0 };
  eventLog: string[] = [];
  
  formData = {
    username: ''
  };
  
  // Basic click handlers
  onButtonClick(): void {
    this.logEvent('Button clicked');
  }
  
  onButtonClickWithParam(message: string): void {
    this.logEvent(`Button clicked with parameter: ${message}`);
  }
  
  onButtonClickWithEvent(event: Event): void {
    const target = event.target as HTMLButtonElement;
    this.logEvent(`Button clicked - Target: ${target.textContent}`);
  }
  
  // Input event handlers
  onInputFocus(): void {
    this.logEvent('Input focused');
  }
  
  onInputBlur(): void {
    this.logEvent('Input blurred');
  }
  
  onInputChange(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.inputValue = target.value;
    this.logEvent(`Input changed: ${target.value}`);
  }
  
  onKeyUp(event: KeyboardEvent): void {
    this.logEvent(`Key up: ${event.key}`);
  }
  
  onKeyDown(event: KeyboardEvent): void {
    if (event.key === 'Enter') {
      this.logEvent('Enter key pressed');
      event.preventDefault();
    }
  }
  
  // Mouse event handlers
  onMouseEnter(): void {
    this.logEvent('Mouse entered area');
  }
  
  onMouseLeave(): void {
    this.logEvent('Mouse left area');
  }
  
  onMouseMove(event: MouseEvent): void {
    this.mousePosition = {
      x: event.offsetX,
      y: event.offsetY
    };
  }
  
  onAreaClick(event: MouseEvent): void {
    this.logEvent(`Area clicked at (${event.offsetX}, ${event.offsetY})`);
  }
  
  // Custom event handlers
  onCustomEvent(data: any): void {
    this.logEvent(`Custom event received: ${JSON.stringify(data)}`);
  }
  
  onDataChanged(data: any): void {
    this.logEvent(`Data changed: ${JSON.stringify(data)}`);
  }
  
  // Form event handlers
  onFormSubmit(event: Event): void {
    event.preventDefault();
    this.logEvent(`Form submitted with username: ${this.formData.username}`);
  }
  
  onFormReset(): void {
    this.formData.username = '';
    this.logEvent('Form reset');
  }
  
  // Utility methods
  logEvent(message: string): void {
    const timestamp = new Date().toLocaleTimeString();
    this.eventLog.unshift(`[${timestamp}] ${message}`);
    
    // Keep only last 20 events
    if (this.eventLog.length > 20) {
      this.eventLog = this.eventLog.slice(0, 20);
    }
  }
  
  clearEventLog(): void {
    this.eventLog = [];
  }
}
```

### **Advanced Event Handling**
```typescript
@Component({
  selector: 'app-advanced-events',
  template: `
    <div class="advanced-events">
      <h3>Advanced Event Handling</h3>
      
      <!-- Event modifiers -->
      <div class="event-modifiers">
        <h4>Event Modifiers</h4>
        
        <!-- Prevent default -->
        <a href="https://google.com" 
           (click)="onLinkClick($event)">
          Link with preventDefault
        </a>
        
        <!-- Stop propagation -->
        <div class="outer" (click)="onOuterClick()">
          Outer Div
          <div class="inner" (click)="onInnerClick($event)">
            Inner Div (Click to test event propagation)
          </div>
        </div>
        
        <!-- Key event modifiers -->
        <input placeholder="Try Ctrl+Enter, Shift+A, etc."
               (keydown.enter)="onEnterKey()"
               (keydown.escape)="onEscapeKey()"
               (keydown.ctrl.enter)="onCtrlEnter()"
               (keydown.shift.a)="onShiftA()">
      </div>
      
      <!-- Debounced events -->
      <div class="debounced-events">
        <h4>Debounced Search</h4>
        <input placeholder="Search..." 
               (input)="onSearchInput($event)"
               [value]="searchTerm">
        <p>Debounced search term: {{debouncedSearchTerm}}</p>
      </div>
      
      <!-- Multiple event listeners -->
      <div class="multi-events">
        <h4>Multiple Event Listeners</h4>
        <button (click)="onClick()" 
                (mousedown)="onMouseDown()"
                (mouseup)="onMouseUp()"
                (dblclick)="onDoubleClick()">
          Multi-Event Button
        </button>
      </div>
      
      <!-- Conditional event binding -->
      <div class="conditional-events">
        <h4>Conditional Event Binding</h4>
        <button [attr.disabled]="isDisabled ? '' : null"
                (click)="!isDisabled && onConditionalClick()">
          Conditional Click
        </button>
        <label>
          <input type="checkbox" [(ngModel)]="isDisabled">
          Disable button
        </label>
      </div>
    </div>
  `,
  styles: [`
    .advanced-events {
      padding: 20px;
    }
    
    .advanced-events > div {
      margin: 20px 0;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    .outer {
      background: #e3f2fd;
      padding: 20px;
      cursor: pointer;
    }
    
    .inner {
      background: #bbdefb;
      padding: 20px;
      margin: 10px;
    }
    
    input, button {
      margin: 5px;
      padding: 8px 12px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    button {
      background: #007bff;
      color: white;
      cursor: pointer;
    }
    
    button:disabled {
      background: #6c757d;
      cursor: not-allowed;
    }
  `]
})
export class AdvancedEventsComponent implements OnDestroy {
  searchTerm = '';
  debouncedSearchTerm = '';
  isDisabled = false;
  
  private searchSubject = new Subject<string>();
  private subscription: Subscription;
  
  constructor() {
    // Setup debounced search
    this.subscription = this.searchSubject.pipe(
      debounceTime(300),
      distinctUntilChanged()
    ).subscribe(term => {
      this.debouncedSearchTerm = term;
    });
  }
  
  ngOnDestroy(): void {
    this.subscription.unsubscribe();
  }
  
  // Link click with preventDefault
  onLinkClick(event: Event): void {
    event.preventDefault();
    console.log('Link clicked but navigation prevented');
  }
  
  // Event propagation
  onOuterClick(): void {
    console.log('Outer div clicked');
  }
  
  onInnerClick(event: Event): void {
    console.log('Inner div clicked');
    event.stopPropagation(); // Prevent event bubbling
  }
  
  // Key events
  onEnterKey(): void {
    console.log('Enter key pressed');
  }
  
  onEscapeKey(): void {
    console.log('Escape key pressed');
  }
  
  onCtrlEnter(): void {
    console.log('Ctrl+Enter pressed');
  }
  
  onShiftA(): void {
    console.log('Shift+A pressed');
  }
  
  // Debounced search
  onSearchInput(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.searchTerm = target.value;
    this.searchSubject.next(target.value);
  }
  
  // Multiple events
  onClick(): void {
    console.log('Click event');
  }
  
  onMouseDown(): void {
    console.log('Mouse down event');
  }
  
  onMouseUp(): void {
    console.log('Mouse up event');
  }
  
  onDoubleClick(): void {
    console.log('Double click event');
  }
  
  // Conditional event
  onConditionalClick(): void {
    console.log('Conditional click executed');
  }
}
```

## ↔️ Two-way Data Binding

### **NgModel และ Banana-in-a-Box Syntax**
```typescript
@Component({
  selector: 'app-two-way-binding',
  template: `
    <div class="two-way-binding-demo">
      <h3>Two-way Data Binding</h3>
      
      <!-- Basic ngModel -->
      <div class="form-group">
        <label>Name:</label>
        <input [(ngModel)]="user.name" placeholder="Enter name">
        <p>Hello, {{user.name}}!</p>
      </div>
      
      <!-- Text area -->
      <div class="form-group">
        <label>Description:</label>
        <textarea [(ngModel)]="user.description" 
                  rows="3" 
                  placeholder="Enter description"></textarea>
        <p>Character count: {{user.description.length}}</p>
      </div>
      
      <!-- Select dropdown -->
      <div class="form-group">
        <label>Country:</label>
        <select [(ngModel)]="user.country">
          <option value="">Select Country</option>
          <option *ngFor="let country of countries" [value]="country.code">
            {{country.name}}
          </option>
        </select>
        <p>Selected: {{getCountryName(user.country)}}</p>
      </div>
      
      <!-- Radio buttons -->
      <div class="form-group">
        <label>Gender:</label>
        <div class="radio-group">
          <label *ngFor="let gender of genders">
            <input type="radio" 
                   [(ngModel)]="user.gender" 
                   [value]="gender.value">
            {{gender.label}}
          </label>
        </div>
        <p>Selected gender: {{user.gender}}</p>
      </div>
      
      <!-- Checkboxes -->
      <div class="form-group">
        <label>Interests:</label>
        <div class="checkbox-group">
          <label *ngFor="let interest of interests">
            <input type="checkbox" 
                   [(ngModel)]="interest.selected">
            {{interest.name}}
          </label>
        </div>
        <p>Selected interests: {{getSelectedInterests()}}</p>
      </div>
      
      <!-- Range slider -->
      <div class="form-group">
        <label>Age: {{user.age}}</label>
        <input type="range" 
               [(ngModel)]="user.age"
               min="18" 
               max="100" 
               step="1">
      </div>
      
      <!-- Date input -->
      <div class="form-group">
        <label>Birth Date:</label>
        <input type="date" [(ngModel)]="user.birthDate">
        <p>Birth date: {{user.birthDate | date:'fullDate'}}</p>
      </div>
      
      <!-- Custom two-way binding component -->
      <div class="form-group">
        <label>Rating:</label>
        <app-star-rating [(rating)]="user.rating"
                         [maxStars]="5">
        </app-star-rating>
        <p>Current rating: {{user.rating}}/5</p>
      </div>
      
      <!-- User object display -->
      <div class="user-preview">
        <h4>User Data:</h4>
        <pre>{{user | json}}</pre>
      </div>
    </div>
  `,
  styles: [`
    .two-way-binding-demo {
      max-width: 600px;
      margin: 0 auto;
      padding: 20px;
    }
    
    .form-group {
      margin-bottom: 20px;
    }
    
    .form-group label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    
    .form-group input, 
    .form-group textarea, 
    .form-group select {
      width: 100%;
      padding: 8px 12px;
      border: 1px solid #ddd;
      border-radius: 4px;
      font-size: 14px;
    }
    
    .radio-group label,
    .checkbox-group label {
      display: inline-block;
      margin-right: 15px;
      font-weight: normal;
      cursor: pointer;
    }
    
    .radio-group input,
    .checkbox-group input {
      width: auto;
      margin-right: 5px;
    }
    
    .user-preview {
      background: #f8f9fa;
      padding: 15px;
      border-radius: 4px;
      margin-top: 20px;
    }
    
    .user-preview pre {
      margin: 0;
      white-space: pre-wrap;
    }
    
    input[type="range"] {
      width: 100%;
    }
  `]
})
export class TwoWayBindingComponent {
  user = {
    name: '',
    description: '',
    country: '',
    gender: '',
    age: 25,
    birthDate: '',
    rating: 0
  };
  
  countries = [
    { code: 'TH', name: 'Thailand' },
    { code: 'US', name: 'United States' },
    { code: 'JP', name: 'Japan' },
    { code: 'KR', name: 'South Korea' }
  ];
  
  genders = [
    { value: 'male', label: 'Male' },
    { value: 'female', label: 'Female' },
    { value: 'other', label: 'Other' }
  ];
  
  interests = [
    { name: 'Programming', selected: false },
    { name: 'Reading', selected: false },
    { name: 'Sports', selected: false },
    { name: 'Music', selected: false },
    { name: 'Travel', selected: false }
  ];
  
  getCountryName(code: string): string {
    const country = this.countries.find(c => c.code === code);
    return country ? country.name : 'None selected';
  }
  
  getSelectedInterests(): string {
    const selected = this.interests
      .filter(interest => interest.selected)
      .map(interest => interest.name);
    
    return selected.length > 0 ? selected.join(', ') : 'None selected';
  }
}
```

### **Custom Two-way Binding Component**
```typescript
// Star Rating Component with Two-way Binding
@Component({
  selector: 'app-star-rating',
  template: `
    <div class="star-rating">
      <span *ngFor="let star of stars; let i = index"
            class="star"
            [class.filled]="i < rating"
            [class.hoverable]="hoverable"
            (click)="onStarClick(i + 1)"
            (mouseenter)="onStarHover(i + 1)"
            (mouseleave)="onMouseLeave()">
        ★
      </span>
    </div>
  `,
  styles: [`
    .star-rating {
      display: inline-flex;
      gap: 2px;
    }
    
    .star {
      font-size: 24px;
      color: #ddd;
      cursor: pointer;
      transition: color 0.2s ease;
      user-select: none;
    }
    
    .star.filled {
      color: #ffc107;
    }
    
    .star.hoverable:hover {
      color: #ffc107;
      transform: scale(1.1);
    }
  `],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => StarRatingComponent),
      multi: true
    }
  ]
})
export class StarRatingComponent implements ControlValueAccessor {
  @Input() maxStars = 5;
  @Input() hoverable = true;
  @Output() ratingChange = new EventEmitter<number>();
  
  rating = 0;
  stars: number[] = [];
  hoverRating = 0;
  
  // ControlValueAccessor implementation
  private onChange = (rating: number) => {};
  private onTouched = () => {};
  
  ngOnInit(): void {
    this.stars = Array(this.maxStars).fill(0).map((_, i) => i + 1);
  }
  
  writeValue(rating: number): void {
    this.rating = rating || 0;
  }
  
  registerOnChange(fn: (rating: number) => void): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: () => void): void {
    this.onTouched = fn;
  }
  
  onStarClick(rating: number): void {
    this.rating = rating;
    this.ratingChange.emit(rating);
    this.onChange(rating);
    this.onTouched();
  }
  
  onStarHover(rating: number): void {
    if (this.hoverable) {
      this.hoverRating = rating;
    }
  }
  
  onMouseLeave(): void {
    this.hoverRating = 0;
  }
  
  // Custom getter for template to show hover effect
  get displayRating(): number {
    return this.hoverRating || this.rating;
  }
}
```

## 🧭 Built-in Directives

### **Structural Directives**
```typescript
@Component({
  selector: 'app-structural-directives',
  template: `
    <div class="structural-directives">
      <h3>Structural Directives</h3>
      
      <!-- ngIf -->
      <div class="directive-example">
        <h4>*ngIf Examples</h4>
        
        <button (click)="toggleVisibility()">
          {{isVisible ? 'Hide' : 'Show'}} Content
        </button>
        
        <div *ngIf="isVisible" class="content-box">
          <p>This content is conditionally displayed!</p>
        </div>
        
        <!-- ngIf with else -->
        <div *ngIf="user.isLoggedIn; else loginPrompt">
          <h5>Welcome, {{user.name}}!</h5>
          <button (click)="logout()">Logout</button>
        </div>
        
        <ng-template #loginPrompt>
          <div class="login-prompt">
            <h5>Please log in</h5>
            <button (click)="login()">Login</button>
          </div>
        </ng-template>
        
        <!-- ngIf with then and else -->
        <div *ngIf="loadingState; then loadingTemplate; else contentTemplate"></div>
        
        <ng-template #loadingTemplate>
          <div class="loading">Loading...</div>
        </ng-template>
        
        <ng-template #contentTemplate>
          <div class="loaded-content">Content loaded successfully!</div>
        </ng-template>
      </div>
      
      <!-- ngFor -->
      <div class="directive-example">
        <h4>*ngFor Examples</h4>
        
        <!-- Basic ngFor -->
        <ul>
          <li *ngFor="let item of items">{{item}}</li>
        </ul>
        
        <!-- ngFor with index -->
        <ol>
          <li *ngFor="let item of items; let i = index">
            {{i + 1}}. {{item}}
          </li>
        </ol>
        
        <!-- ngFor with tracking -->
        <div class="user-list">
          <div *ngFor="let user of users; trackBy: trackByUserId" 
               class="user-item">
            <img [src]="user.avatar" [alt]="user.name + ' avatar'">
            <div class="user-info">
              <h5>{{user.name}}</h5>
              <p>{{user.email}}</p>
            </div>
            <button (click)="removeUser(user.id)">Remove</button>
          </div>
        </div>
        
        <!-- ngFor with multiple variables -->
        <div class="advanced-list">
          <div *ngFor="let item of items; 
                       let i = index; 
                       let first = first; 
                       let last = last; 
                       let even = even; 
                       let odd = odd"
               [class.first]="first"
               [class.last]="last"
               [class.even]="even"
               [class.odd]="odd">
            {{i + 1}}. {{item}} 
            <span *ngIf="first">(First)</span>
            <span *ngIf="last">(Last)</span>
          </div>
        </div>
        
        <button (click)="addRandomItem()">Add Random Item</button>
        <button (click)="shuffleItems()">Shuffle Items</button>
      </div>
      
      <!-- ngSwitch -->
      <div class="directive-example">
        <h4>*ngSwitch Examples</h4>
        
        <div class="switch-controls">
          <button *ngFor="let mode of viewModes" 
                  (click)="currentViewMode = mode"
                  [class.active]="currentViewMode === mode">
            {{mode}}
          </button>
        </div>
        
        <div [ngSwitch]="currentViewMode" class="switch-content">
          <div *ngSwitchCase="'list'" class="view-mode">
            <h5>List View</h5>
            <ul>
              <li *ngFor="let item of items">{{item}}</li>
            </ul>
          </div>
          
          <div *ngSwitchCase="'grid'" class="view-mode">
            <h5>Grid View</h5>
            <div class="grid">
              <div *ngFor="let item of items" class="grid-item">
                {{item}}
              </div>
            </div>
          </div>
          
          <div *ngSwitchCase="'table'" class="view-mode">
            <h5>Table View</h5>
            <table>
              <tr *ngFor="let item of items; let i = index">
                <td>{{i + 1}}</td>
                <td>{{item}}</td>
              </tr>
            </table>
          </div>
          
          <div *ngSwitchDefault class="view-mode">
            <h5>Default View</h5>
            <p>Please select a view mode.</p>
          </div>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .structural-directives {
      padding: 20px;
    }
    
    .directive-example {
      margin: 30px 0;
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
    
    .content-box {
      background: #e3f2fd;
      padding: 15px;
      border-radius: 4px;
      margin: 10px 0;
    }
    
    .login-prompt {
      background: #fff3cd;
      padding: 15px;
      border-radius: 4px;
      margin: 10px 0;
    }
    
    .loading {
      text-align: center;
      padding: 20px;
      font-style: italic;
    }
    
    .user-list {
      margin: 20px 0;
    }
    
    .user-item {
      display: flex;
      align-items: center;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      margin: 5px 0;
    }
    
    .user-item img {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      margin-right: 10px;
    }
    
    .user-info {
      flex: 1;
    }
    
    .user-info h5 {
      margin: 0 0 5px 0;
    }
    
    .user-info p {
      margin: 0;
      color: #666;
      font-size: 14px;
    }
    
    .advanced-list div {
      padding: 8px;
      margin: 2px 0;
      border-radius: 4px;
    }
    
    .advanced-list .first {
      background: #d4edda;
    }
    
    .advanced-list .last {
      background: #f8d7da;
    }
    
    .advanced-list .even {
      background: #e2e3e5;
    }
    
    .advanced-list .odd {
      background: #f8f9fa;
    }
    
    .switch-controls button {
      margin-right: 10px;
      padding: 8px 16px;
      border: 1px solid #007bff;
      background: white;
      color: #007bff;
      border-radius: 4px;
      cursor: pointer;
    }
    
    .switch-controls button.active {
      background: #007bff;
      color: white;
    }
    
    .switch-content {
      margin-top: 20px;
      min-height: 200px;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
      gap: 10px;
    }
    
    .grid-item {
      background: #f8f9fa;
      padding: 20px;
      text-align: center;
      border-radius: 4px;
      border: 1px solid #ddd;
    }
    
    table {
      width: 100%;
      border-collapse: collapse;
    }
    
    table td {
      padding: 8px;
      border: 1px solid #ddd;
    }
    
    button {
      margin: 5px;
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      background: #007bff;
      color: white;
      cursor: pointer;
    }
    
    button:hover {
      background: #0056b3;
    }
  `]
})
export class StructuralDirectivesComponent {
  isVisible = true;
  loadingState = false;
  currentViewMode = 'list';
  
  user = {
    name: 'John Doe',
    isLoggedIn: false
  };
  
  items = ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry'];
  
  users = [
    {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      avatar: 'https://ui-avatars.com/api/?name=John+Doe&background=007bff&color=fff'
    },
    {
      id: 2,
      name: 'Jane Smith',
      email: 'jane@example.com',
      avatar: 'https://ui-avatars.com/api/?name=Jane+Smith&background=28a745&color=fff'
    },
    {
      id: 3,
      name: 'Bob Johnson',
      email: 'bob@example.com',
      avatar: 'https://ui-avatars.com/api/?name=Bob+Johnson&background=dc3545&color=fff'
    }
  ];
  
  viewModes = ['list', 'grid', 'table'];
  
  toggleVisibility(): void {
    this.isVisible = !this.isVisible;
  }
  
  login(): void {
    this.user.isLoggedIn = true;
  }
  
  logout(): void {
    this.user.isLoggedIn = false;
  }
  
  trackByUserId(index: number, user: any): number {
    return user.id;
  }
  
  removeUser(userId: number): void {
    this.users = this.users.filter(user => user.id !== userId);
  }
  
  addRandomItem(): void {
    const fruits = ['Mango', 'Orange', 'Grape', 'Kiwi', 'Pineapple', 'Strawberry'];
    const randomFruit = fruits[Math.floor(Math.random() * fruits.length)];
    if (!this.items.includes(randomFruit)) {
      this.items.push(randomFruit);
    }
  }
  
  shuffleItems(): void {
    this.items = [...this.items].sort(() => Math.random() - 0.5);
  }
}
```

## 🎯 Custom Directives

### **Custom Attribute Directive**
```typescript
// Highlight Directive
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective implements OnInit, OnDestroy {
  @Input() appHighlight = '#ffeb3b';
  @Input() highlightDuration = 300;
  
  private originalBackground = '';
  
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  
  ngOnInit(): void {
    this.originalBackground = this.el.nativeElement.style.backgroundColor;
  }
  
  ngOnDestroy(): void {
    this.resetBackground();
  }
  
  @HostListener('mouseenter') onMouseEnter(): void {
    this.highlight(this.appHighlight);
  }
  
  @HostListener('mouseleave') onMouseLeave(): void {
    this.resetBackground();
  }
  
  @HostListener('click') onClick(): void {
    this.flashHighlight();
  }
  
  private highlight(color: string): void {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
    this.renderer.setStyle(this.el.nativeElement, 'transition', 'background-color 0.3s ease');
  }
  
  private resetBackground(): void {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', this.originalBackground);
  }
  
  private flashHighlight(): void {
    this.highlight('#ff5722');
    setTimeout(() => {
      this.resetBackground();
    }, this.highlightDuration);
  }
}

// Usage
@Component({
  template: `
    <div class="custom-directives-demo">
      <h3>Custom Directives</h3>
      
      <!-- Highlight directive -->
      <p appHighlight="#e3f2fd">
        Hover over this text to see highlight effect
      </p>
      
      <p appHighlight="#f3e5f5" [highlightDuration]="1000">
        This has a longer highlight duration
      </p>
      
      <button appHighlight="#c8e6c9">
        Highlighted Button
      </button>
    </div>
  `
})
export class CustomDirectivesComponent {}
```

### **Custom Structural Directive**
```typescript
// Unless Directive (opposite of ngIf)
@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
  
  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}

// Advanced Repeat Directive
@Directive({
  selector: '[appRepeat]'
})
export class RepeatDirective implements OnInit {
  @Input() appRepeat = 1;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
  
  ngOnInit(): void {
    this.updateView();
  }
  
  @Input() set appRepeatOf(count: number) {
    this.appRepeat = count;
    this.updateView();
  }
  
  private updateView(): void {
    this.viewContainer.clear();
    
    for (let i = 0; i < this.appRepeat; i++) {
      this.viewContainer.createEmbeddedView(this.templateRef, {
        $implicit: i,
        index: i,
        first: i === 0,
        last: i === this.appRepeat - 1,
        even: i % 2 === 0,
        odd: i % 2 === 1
      });
    }
  }
}

// Usage
@Component({
  template: `
    <div class="structural-directives-demo">
      <!-- Unless directive -->
      <div *appUnless="isHidden">
        This content shows when isHidden is false
      </div>
      
      <!-- Repeat directive -->
      <div *appRepeat="3; let i = index">
        Repeated item {{i + 1}}
      </div>
      
      <div *appRepeat="let item of repeatCount; let first = first; let last = last">
        <span [class.first]="first" [class.last]="last">
          Item {{item + 1}}
        </span>
      </div>
    </div>
  `
})
export class StructuralDirectivesDemo {
  isHidden = false;
  repeatCount = 5;
}
```

## 🧪 แบบฝึกหัด

### **Exercise 1: Interactive Form**
สร้างฟอร์มที่มี:
1. Two-way binding สำหรับทุก input
2. Real-time validation feedback
3. Custom directive สำหรับ password strength
4. Dynamic form fields

### **Exercise 2: Todo List with Custom Directives**
สร้าง Todo List ที่ใช้:
1. Custom highlight directive
2. Custom structural directive สำหรับ filtering
3. Two-way binding สำหรับ todo status
4. Event binding สำหรับ CRUD operations

### **Exercise 3: Data Visualization Dashboard**
สร้าง Dashboard ที่มี:
1. Dynamic chart updates ด้วย property binding
2. Interactive filters ด้วย two-way binding
3. Custom directives สำหรับ chart styling
4. Event binding สำหรับ user interactions

## 🧪 Quiz

### **Question 1**
Two-way data binding ใน Angular ใช้ syntax:
- a) `{{property}}`
- b) `[property]`
- c) `(event)`
- d) `[(ngModel)]`

### **Question 2**
Structural directive ใน Angular เริ่มต้นด้วย:
- a) `@`
- b) `*`
- c) `#`
- d) `[]`

### **Question 3**
Custom directive ที่ต้องการเข้าถึง DOM element ใช้:
- a) ViewChild
- b) ElementRef
- c) TemplateRef
- d) ViewContainerRef

**คำตอบ: 1-d, 2-b, 3-b**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🔗 Data Binding**
1. **Interpolation** - แสดงข้อมูลใน template
2. **Property Binding** - bind properties และ attributes
3. **Event Binding** - handle user interactions
4. **Two-way Binding** - synchronize data bidirectionally

### **🧭 Built-in Directives**
1. **Structural Directives** - *ngIf, *ngFor, *ngSwitch
2. **Attribute Directives** - ngClass, ngStyle
3. **Best Practices** - performance และ usage patterns

### **🎯 Custom Directives**
1. **Attribute Directives** - เปลี่ยนแปลง DOM elements
2. **Structural Directives** - เพิ่ม/ลบ DOM elements
3. **HostListener และ HostBinding** - event handling
4. **Directive Communication** - input/output properties

### **⚡ Advanced Concepts**
1. **Event Modifiers** - preventDefault, stopPropagation
2. **Template Reference Variables** - direct element access
3. **Safe Navigation** - null-safe property access
4. **Performance Optimization** - trackBy functions

### **🎯 Next Steps**
ในบทต่อไป เราจะเรียนรู้เกี่ยวกับ Services และ Dependency Injection:
- Creating and using services
- Dependency injection patterns
- Service lifecycle management
- Inter-component communication via services

---

**🎉 ยินดีด้วย! ตอนนี้คุณเข้าใจ Data Binding และ Directives อย่างครบถ้วนแล้ว**

**[⬅️ บทที่ 5: Components and Templates](./05-components-and-templates.md) | [➡️ บทที่ 7: Services and Dependency Injection](./07-services-and-dependency-injection.md)**
