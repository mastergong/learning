# บทที่ 10: Angular Material และ Component Dev Kit (CDK)

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- ติดตั้งและใช้งาน Angular Material ในโปรเจกต์
- ปรับแต่ง themes และ typography
- ใช้งาน Material components ที่สำคัญ
- เข้าใจและใช้งาน Component Dev Kit (CDK)
- สร้าง custom UI components ด้วย CDK

## 🛠️ Angular Material คืออะไร?

Angular Material เป็น UI component library อย่างเป็นทางการของ Angular ที่ปฏิบัติตาม Google's Material Design principles

### 📦 การติดตั้ง Angular Material

```bash
# ติดตั้งด้วย Angular CLI
ng add @angular/material
```

ระหว่างการติดตั้ง คุณจะถูกถามคำถามต่อไปนี้:

1. เลือก prebuilt theme (หรือ custom theme):
   - Indigo/Pink
   - Deep Purple/Amber
   - Pink/Blue Grey
   - Purple/Green
   - Custom

2. ตั้งค่า global Angular Material typography styles:
   - Yes
   - No

3. เพิ่ม browser animations:
   - Yes (recommended)
   - Yes, with NoopAnimations (สำหรับ environments ที่ไม่รองรับ animations)
   - No

## 🎨 Theming ใน Angular Material

### การตั้งค่า Custom Theme

```scss
// styles.scss
@use '@angular/material' as mat;

// ประกาศ custom palette
$primary: mat.define-palette(mat.$indigo-palette, 500);
$accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);
$warn: mat.define-palette(mat.$red-palette);

// สร้าง custom theme
$my-theme: mat.define-light-theme((
  color: (
    primary: $primary,
    accent: $accent,
    warn: $warn,
  ),
  typography: mat.define-typography-config(),
  density: 0,
));

// นำ theme ไปใช้
@include mat.all-component-themes($my-theme);

// เพิ่ม dark mode
.dark-theme {
  $dark-theme: mat.define-dark-theme((
    color: (
      primary: mat.define-palette(mat.$blue-grey-palette),
      accent: mat.define-palette(mat.$amber-palette, A200, A100, A400),
      warn: $warn,
    )
  ));

  @include mat.all-component-colors($dark-theme);
}
```

### Typography Customization

```scss
// Define custom typography
$custom-typography: mat.define-typography-config(
  $font-family: 'Roboto, "Helvetica Neue", sans-serif',
  $headline-1: mat.define-typography-level(96px, 96px, 300, $letter-spacing: -0.015em),
  $headline-2: mat.define-typography-level(60px, 60px, 300, $letter-spacing: -0.005em),
  $headline-3: mat.define-typography-level(48px, 48px, 400),
  $headline-4: mat.define-typography-level(34px, 40px, 400),
  $headline-5: mat.define-typography-level(24px, 32px, 400),
  $headline-6: mat.define-typography-level(20px, 32px, 500),
  $subtitle-1: mat.define-typography-level(16px, 28px, 400),
  $subtitle-2: mat.define-typography-level(14px, 24px, 500),
  $body-1: mat.define-typography-level(16px, 24px, 400),
  $body-2: mat.define-typography-level(14px, 20px, 400),
  $caption: mat.define-typography-level(12px, 20px, 400),
  $button: mat.define-typography-level(14px, 14px, 500),
);

// นำ typography ไปใช้
$my-theme: mat.define-light-theme((
  color: (
    primary: $primary,
    accent: $accent,
    warn: $warn,
  ),
  typography: $custom-typography,
  density: 0,
));
```

## 📋 Material Components ที่สำคัญ

### 1. Buttons & Indicators

#### Button Components
```typescript
@Component({
  selector: 'app-button-demo',
  standalone: true,
  imports: [MatButtonModule, MatIconModule],
  template: `
    <h2>Buttons</h2>
    
    <div class="button-row">
      <button mat-button>Basic</button>
      <button mat-raised-button color="primary">Raised</button>
      <button mat-stroked-button color="accent">Stroked</button>
      <button mat-flat-button color="warn">Flat</button>
      
      <button mat-icon-button aria-label="Example icon button">
        <mat-icon>favorite</mat-icon>
      </button>
      
      <button mat-fab color="primary">
        <mat-icon>add</mat-icon>
      </button>
      
      <button mat-mini-fab color="accent">
        <mat-icon>edit</mat-icon>
      </button>
    </div>
    
    <div class="button-row">
      <button mat-raised-button [disabled]="true">Disabled</button>
      <a mat-button routerLink=".">Link</a>
    </div>
  `,
  styles: [`
    .button-row {
      display: flex;
      gap: 8px;
      margin-bottom: 16px;
      flex-wrap: wrap;
      align-items: center;
    }
  `]
})
export class ButtonDemoComponent {}
```

#### Progress Indicators
```typescript
@Component({
  selector: 'app-progress-demo',
  standalone: true,
  imports: [MatProgressBarModule, MatProgressSpinnerModule, MatButtonModule],
  template: `
    <h2>Progress Indicators</h2>
    
    <h3>Progress Bar</h3>
    <mat-progress-bar mode="determinate" [value]="progressValue"></mat-progress-bar>
    <mat-progress-bar mode="indeterminate"></mat-progress-bar>
    <mat-progress-bar mode="buffer" [value]="progressValue" [bufferValue]="bufferValue"></mat-progress-bar>
    <mat-progress-bar mode="query"></mat-progress-bar>
    
    <h3>Progress Spinner</h3>
    <div class="spinner-container">
      <mat-spinner [diameter]="50"></mat-spinner>
      <mat-progress-spinner mode="determinate" [value]="progressValue" [diameter]="100"></mat-progress-spinner>
      <button mat-raised-button (click)="incrementProgress()">Increment Progress</button>
    </div>
  `,
  styles: [`
    mat-progress-bar {
      margin-bottom: 16px;
    }
    
    .spinner-container {
      display: flex;
      align-items: center;
      gap: 16px;
      flex-wrap: wrap;
    }
  `]
})
export class ProgressDemoComponent {
  progressValue = 35;
  bufferValue = 75;
  
  incrementProgress() {
    this.progressValue = (this.progressValue + 10) % 100;
    this.bufferValue = Math.min(this.progressValue + 30, 100);
  }
}
```

### 2. Form Controls

#### Input & Form Fields
```typescript
@Component({
  selector: 'app-input-demo',
  standalone: true,
  imports: [
    MatFormFieldModule,
    MatInputModule,
    MatIconModule,
    MatButtonModule,
    ReactiveFormsModule
  ],
  template: `
    <h2>Input & Form Fields</h2>
    
    <form [formGroup]="form">
      <mat-form-field appearance="fill">
        <mat-label>Full name</mat-label>
        <input matInput formControlName="name" placeholder="John Doe">
        <mat-icon matSuffix>person</mat-icon>
        <mat-hint>Enter your full name</mat-hint>
        <mat-error *ngIf="name.errors?.['required']">Name is required</mat-error>
      </mat-form-field>
      
      <mat-form-field appearance="outline">
        <mat-label>Email</mat-label>
        <input matInput formControlName="email" placeholder="example@mail.com" required>
        <mat-error *ngIf="email.errors?.['required']">Email is required</mat-error>
        <mat-error *ngIf="email.errors?.['email']">Please enter a valid email address</mat-error>
      </mat-form-field>
      
      <mat-form-field appearance="fill">
        <mat-label>Password</mat-label>
        <input matInput formControlName="password" [type]="hidePassword ? 'password' : 'text'">
        <button mat-icon-button matSuffix (click)="hidePassword = !hidePassword" [attr.aria-label]="'Hide password'" [attr.aria-pressed]="hidePassword">
          <mat-icon>{{hidePassword ? 'visibility_off' : 'visibility'}}</mat-icon>
        </button>
      </mat-form-field>
      
      <mat-form-field appearance="fill">
        <mat-label>Comments</mat-label>
        <textarea matInput formControlName="comments" placeholder="Leave a comment"></textarea>
      </mat-form-field>
      
      <button mat-raised-button color="primary" [disabled]="form.invalid">Submit</button>
    </form>
  `,
  styles: [`
    form {
      display: flex;
      flex-direction: column;
      max-width: 400px;
      gap: 16px;
    }
  `]
})
export class InputDemoComponent {
  form = new FormGroup({
    name: new FormControl('', [Validators.required]),
    email: new FormControl('', [Validators.required, Validators.email]),
    password: new FormControl('', [Validators.required, Validators.minLength(8)]),
    comments: new FormControl('')
  });
  
  hidePassword = true;
  
  get name() { return this.form.get('name')!; }
  get email() { return this.form.get('email')!; }
}
```

#### Select & Autocomplete
```typescript
@Component({
  selector: 'app-select-demo',
  standalone: true,
  imports: [
    MatFormFieldModule,
    MatSelectModule,
    MatAutocompleteModule,
    MatInputModule,
    ReactiveFormsModule
  ],
  template: `
    <h2>Dropdown Controls</h2>
    
    <div class="controls-container">
      <mat-form-field appearance="fill">
        <mat-label>Favorite fruit</mat-label>
        <mat-select [formControl]="fruitControl">
          <mat-option>None</mat-option>
          <mat-option *ngFor="let fruit of fruits" [value]="fruit">{{fruit}}</mat-option>
        </mat-select>
        <mat-hint *ngIf="fruitControl.value">You selected: {{fruitControl.value}}</mat-hint>
      </mat-form-field>
      
      <mat-form-field appearance="fill">
        <mat-label>State</mat-label>
        <input type="text" matInput [formControl]="stateControl" [matAutocomplete]="auto">
        <mat-autocomplete #auto="matAutocomplete">
          <mat-option *ngFor="let state of filteredStates | async" [value]="state">
            {{state}}
          </mat-option>
        </mat-autocomplete>
      </mat-form-field>
    </div>
  `,
  styles: [`
    .controls-container {
      display: flex;
      flex-direction: column;
      max-width: 400px;
      gap: 16px;
    }
  `]
})
export class SelectDemoComponent {
  fruitControl = new FormControl('');
  stateControl = new FormControl('');
  
  fruits: string[] = ['Apple', 'Banana', 'Orange', 'Mango', 'Pineapple'];
  states: string[] = [
    'Alabama', 'Alaska', 'Arizona', 'California', 'Colorado',
    'Florida', 'Georgia', 'Hawaii', 'New York', 'Texas', 'Washington'
  ];
  
  filteredStates: Observable<string[]>;
  
  constructor() {
    this.filteredStates = this.stateControl.valueChanges.pipe(
      startWith(''),
      map(value => this._filterStates(value || ''))
    );
  }
  
  private _filterStates(value: string): string[] {
    const filterValue = value.toLowerCase();
    return this.states.filter(state => state.toLowerCase().includes(filterValue));
  }
}
```

### 3. Navigation Components

#### Toolbar & Sidenav
```typescript
@Component({
  selector: 'app-navigation-demo',
  standalone: true,
  imports: [
    MatToolbarModule,
    MatSidenavModule,
    MatButtonModule,
    MatIconModule,
    MatListModule
  ],
  template: `
    <div class="sidenav-container">
      <mat-toolbar color="primary">
        <button mat-icon-button (click)="drawer.toggle()">
          <mat-icon>menu</mat-icon>
        </button>
        <span>My Application</span>
        <span class="toolbar-spacer"></span>
        <button mat-icon-button aria-label="Example icon-button with share icon">
          <mat-icon>share</mat-icon>
        </button>
        <button mat-icon-button [matMenuTriggerFor]="menu">
          <mat-icon>more_vert</mat-icon>
        </button>
        <mat-menu #menu="matMenu">
          <button mat-menu-item>
            <mat-icon>person</mat-icon>
            <span>Profile</span>
          </button>
          <button mat-menu-item>
            <mat-icon>settings</mat-icon>
            <span>Settings</span>
          </button>
          <button mat-menu-item>
            <mat-icon>logout</mat-icon>
            <span>Logout</span>
          </button>
        </mat-menu>
      </mat-toolbar>
      
      <mat-sidenav-container class="example-container">
        <mat-sidenav #drawer mode="side" opened class="example-sidenav">
          <mat-nav-list>
            <a mat-list-item routerLink="/">
              <mat-icon matListItemIcon>home</mat-icon>
              <span matListItemTitle>Home</span>
            </a>
            <a mat-list-item routerLink="/dashboard">
              <mat-icon matListItemIcon>dashboard</mat-icon>
              <span matListItemTitle>Dashboard</span>
            </a>
            <a mat-list-item routerLink="/profile">
              <mat-icon matListItemIcon>person</mat-icon>
              <span matListItemTitle>Profile</span>
            </a>
            <mat-divider></mat-divider>
            <a mat-list-item routerLink="/settings">
              <mat-icon matListItemIcon>settings</mat-icon>
              <span matListItemTitle>Settings</span>
            </a>
          </mat-nav-list>
        </mat-sidenav>
        <mat-sidenav-content class="example-sidenav-content">
          <div class="content-container">
            <h2>Main Content Area</h2>
            <p>This is where your main content would go.</p>
            <button mat-raised-button color="accent">Some Action</button>
          </div>
        </mat-sidenav-content>
      </mat-sidenav-container>
    </div>
  `,
  styles: [`
    .sidenav-container {
      height: 100vh;
      display: flex;
      flex-direction: column;
    }
    
    .toolbar-spacer {
      flex: 1 1 auto;
    }
    
    .example-container {
      flex: 1;
    }
    
    .example-sidenav {
      width: 250px;
    }
    
    .content-container {
      padding: 20px;
    }
  `]
})
export class NavigationDemoComponent {}
```

#### Tabs
```typescript
@Component({
  selector: 'app-tabs-demo',
  standalone: true,
  imports: [MatTabsModule],
  template: `
    <h2>Tabs</h2>
    
    <mat-tab-group mat-stretch-tabs="false" mat-align-tabs="start">
      <mat-tab label="First">
        <div class="tab-content">
          <h3>First Tab Content</h3>
          <p>Content for the first tab goes here.</p>
        </div>
      </mat-tab>
      <mat-tab label="Second">
        <div class="tab-content">
          <h3>Second Tab Content</h3>
          <p>Content for the second tab goes here.</p>
        </div>
      </mat-tab>
      <mat-tab label="Third">
        <div class="tab-content">
          <h3>Third Tab Content</h3>
          <p>Content for the third tab goes here.</p>
        </div>
      </mat-tab>
      <mat-tab disabled>
        <ng-template mat-tab-label>
          <mat-icon>lock</mat-icon>
          Locked
        </ng-template>
      </mat-tab>
    </mat-tab-group>
  `,
  styles: [`
    .tab-content {
      padding: 20px;
    }
  `]
})
export class TabsDemoComponent {}
```

### 4. Data Table

#### MatTable
```typescript
@Component({
  selector: 'app-table-demo',
  standalone: true,
  imports: [
    MatTableModule,
    MatSortModule,
    MatPaginatorModule,
    MatFormFieldModule,
    MatInputModule
  ],
  template: `
    <h2>Data Table</h2>
    
    <div class="table-container">
      <mat-form-field appearance="standard">
        <mat-label>Filter</mat-label>
        <input matInput (keyup)="applyFilter($event)" placeholder="Search" #input>
      </mat-form-field>
      
      <div class="mat-elevation-z8">
        <table mat-table [dataSource]="dataSource" matSort>
          
          <!-- ID Column -->
          <ng-container matColumnDef="id">
            <th mat-header-cell *matHeaderCellDef mat-sort-header> ID </th>
            <td mat-cell *matCellDef="let element"> {{element.id}} </td>
          </ng-container>
          
          <!-- Name Column -->
          <ng-container matColumnDef="name">
            <th mat-header-cell *matHeaderCellDef mat-sort-header> Name </th>
            <td mat-cell *matCellDef="let element"> {{element.name}} </td>
          </ng-container>
          
          <!-- Email Column -->
          <ng-container matColumnDef="email">
            <th mat-header-cell *matHeaderCellDef mat-sort-header> Email </th>
            <td mat-cell *matCellDef="let element"> {{element.email}} </td>
          </ng-container>
          
          <!-- Role Column -->
          <ng-container matColumnDef="role">
            <th mat-header-cell *matHeaderCellDef mat-sort-header> Role </th>
            <td mat-cell *matCellDef="let element"> {{element.role}} </td>
          </ng-container>
          
          <!-- Status Column -->
          <ng-container matColumnDef="status">
            <th mat-header-cell *matHeaderCellDef mat-sort-header> Status </th>
            <td mat-cell *matCellDef="let element">
              <span [ngClass]="{'active': element.status === 'Active', 'inactive': element.status === 'Inactive'}">
                {{element.status}}
              </span>
            </td>
          </ng-container>
          
          <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
          <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
          
          <!-- Row shown when there is no matching data. -->
          <tr class="mat-row" *matNoDataRow>
            <td class="mat-cell" colspan="5">No data matching the filter "{{input.value}}"</td>
          </tr>
        </table>
        
        <mat-paginator [pageSizeOptions]="[5, 10, 25, 100]" aria-label="Select page of users"></mat-paginator>
      </div>
    </div>
  `,
  styles: [`
    .table-container {
      max-width: 800px;
      margin: 0 auto;
    }
    
    mat-form-field {
      width: 100%;
      margin-bottom: 10px;
    }
    
    .active {
      color: green;
      font-weight: bold;
    }
    
    .inactive {
      color: red;
    }
  `]
})
export class TableDemoComponent implements AfterViewInit {
  displayedColumns: string[] = ['id', 'name', 'email', 'role', 'status'];
  dataSource = new MatTableDataSource<User>(USERS_DATA);
  
  @ViewChild(MatPaginator) paginator!: MatPaginator;
  @ViewChild(MatSort) sort!: MatSort;
  
  ngAfterViewInit() {
    this.dataSource.paginator = this.paginator;
    this.dataSource.sort = this.sort;
  }
  
  applyFilter(event: Event) {
    const filterValue = (event.target as HTMLInputElement).value;
    this.dataSource.filter = filterValue.trim().toLowerCase();
    
    if (this.dataSource.paginator) {
      this.dataSource.paginator.firstPage();
    }
  }
}

export interface User {
  id: number;
  name: string;
  email: string;
  role: string;
  status: string;
}

const USERS_DATA: User[] = [
  {id: 1, name: 'John Doe', email: 'john@example.com', role: 'Admin', status: 'Active'},
  {id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'Editor', status: 'Active'},
  {id: 3, name: 'Bob Johnson', email: 'bob@example.com', role: 'User', status: 'Inactive'},
  // Add more data here...
];
```

### 5. Dialog & Overlay

#### Dialog
```typescript
@Component({
  selector: 'app-dialog-demo',
  standalone: true,
  imports: [MatButtonModule, MatDialogModule],
  template: `
    <h2>Dialog</h2>
    
    <div class="button-container">
      <button mat-raised-button (click)="openDialog()" color="primary">Open Dialog</button>
    </div>
  `,
  styles: [`
    .button-container {
      margin-top: 20px;
    }
  `]
})
export class DialogDemoComponent {
  constructor(public dialog: MatDialog) {}
  
  openDialog(): void {
    const dialogRef = this.dialog.open(DialogExampleComponent, {
      width: '350px',
      data: { name: 'John Doe', animal: 'Dog' },
    });
    
    dialogRef.afterClosed().subscribe(result => {
      console.log('Dialog closed with result:', result);
    });
  }
}

@Component({
  selector: 'dialog-example',
  standalone: true,
  imports: [MatDialogModule, MatButtonModule, MatFormFieldModule, MatInputModule, FormsModule],
  template: `
    <h1 mat-dialog-title>Hi {{ data.name }}</h1>
    <div mat-dialog-content>
      <p>What's your favorite animal?</p>
      <mat-form-field appearance="fill">
        <mat-label>Favorite Animal</mat-label>
        <input matInput [(ngModel)]="data.animal">
      </mat-form-field>
    </div>
    <div mat-dialog-actions>
      <button mat-button (click)="onCancel()">Cancel</button>
      <button mat-button color="primary" [mat-dialog-close]="data.animal" cdkFocusInitial>Ok</button>
    </div>
  `,
})
export class DialogExampleComponent {
  constructor(
    public dialogRef: MatDialogRef<DialogExampleComponent>,
    @Inject(MAT_DIALOG_DATA) public data: {name: string, animal: string}
  ) {}
  
  onCancel(): void {
    this.dialogRef.close();
  }
}
```

## 🛠️ Component Dev Kit (CDK)

CDK คือชุดเครื่องมือที่ช่วยในการสร้าง custom UI components โดยไม่ต้องผูกติดกับ Material Design

### CDK Categories

1. **Accessibility (a11y)** - การจัดการ focus, live announcer, focus trap
2. **Overlay** - popovers, dropdowns, dialogs
3. **Layout** - flexbox, responsive layouts
4. **Portal** - dynamic content projection
5. **Drag and Drop** - draggable elements
6. **Platform** - browser detection, feature detection
7. **Text Field** - auto-resize, auto-fill monitoring
8. **Virtual Scrolling** - efficient rendering of large lists

### CDK Examples

#### 1. Drag and Drop
```typescript
@Component({
  selector: 'app-dnd-demo',
  standalone: true,
  imports: [CdkDragDrop, CdkDropList, CommonModule],
  template: `
    <h2>Drag and Drop</h2>
    
    <div class="drag-drop-container">
      <h3>Todo List</h3>
      <div 
        cdkDropList 
        #todoList="cdkDropList" 
        [cdkDropListData]="todos"
        [cdkDropListConnectedTo]="[doneList]" 
        class="list-container"
        (cdkDropListDropped)="drop($event)">
        <div class="list-item" *ngFor="let item of todos" cdkDrag>
          <div class="list-item-content">
            {{item}}
            <div class="list-item-handle" cdkDragHandle>
              <svg width="24px" fill="currentColor" viewBox="0 0 24 24">
                <path d="M10 9h4V6h3l-5-5-5 5h3v3zm-1 1H6V7l-5 5 5 5v-3h3v-4zm14 2l-5-5v3h-3v4h3v3l5-5zm-9 3h-4v3H7l5 5 5-5h-3v-3z"></path>
              </svg>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="drag-drop-container">
      <h3>Done List</h3>
      <div 
        cdkDropList 
        #doneList="cdkDropList"
        [cdkDropListData]="done" 
        [cdkDropListConnectedTo]="[todoList]" 
        class="list-container"
        (cdkDropListDropped)="drop($event)">
        <div class="list-item" *ngFor="let item of done" cdkDrag>
          <div class="list-item-content">
            {{item}}
            <div class="list-item-handle" cdkDragHandle>
              <svg width="24px" fill="currentColor" viewBox="0 0 24 24">
                <path d="M10 9h4V6h3l-5-5-5 5h3v3zm-1 1H6V7l-5 5 5 5v-3h3v-4zm14 2l-5-5v3h-3v4h3v3l5-5zm-9 3h-4v3H7l5 5 5-5h-3v-3z"></path>
              </svg>
            </div>
          </div>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .drag-drop-container {
      width: 400px;
      max-width: 100%;
      margin-bottom: 25px;
      display: inline-block;
      vertical-align: top;
      margin-right: 25px;
    }
    
    .list-container {
      min-height: 60px;
      border-radius: 4px;
      overflow: hidden;
      background: #f5f5f5;
    }
    
    .list-item {
      padding: 20px 10px;
      border-bottom: solid 1px #ccc;
      color: rgba(0, 0, 0, 0.87);
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: space-between;
      box-sizing: border-box;
      cursor: move;
      background: white;
    }
    
    .list-item-content {
      display: flex;
      flex-grow: 1;
      justify-content: space-between;
    }
    
    .list-item-handle {
      color: #ccc;
      cursor: move;
      width: 24px;
      height: 24px;
    }
    
    .cdk-drag-preview {
      box-sizing: border-box;
      border-radius: 4px;
      box-shadow: 0 5px 5px -3px rgba(0, 0, 0, 0.2),
                  0 8px 10px 1px rgba(0, 0, 0, 0.14),
                  0 3px 14px 2px rgba(0, 0, 0, 0.12);
    }
    
    .cdk-drag-placeholder {
      opacity: 0;
    }
    
    .cdk-drag-animating {
      transition: transform 250ms cubic-bezier(0, 0, 0.2, 1);
    }
    
    .list-item:last-child {
      border: none;
    }
    
    .list-container.cdk-drop-list-dragging .list-item:not(.cdk-drag-placeholder) {
      transition: transform 250ms cubic-bezier(0, 0, 0.2, 1);
    }
  `]
})
export class DragDropDemoComponent {
  todos = [
    'Get to work',
    'Pick up groceries',
    'Go home',
    'Fall asleep'
  ];
  
  done = [
    'Get up',
    'Brush teeth',
    'Take a shower',
    'Check e-mail'
  ];
  
  drop(event: CdkDragDrop<string[]>) {
    if (event.previousContainer === event.container) {
      moveItemInArray(event.container.data, event.previousIndex, event.currentIndex);
    } else {
      transferArrayItem(
        event.previousContainer.data,
        event.container.data,
        event.previousIndex,
        event.currentIndex,
      );
    }
  }
}
```

#### 2. Virtual Scrolling
```typescript
@Component({
  selector: 'app-virtual-scroll-demo',
  standalone: true,
  imports: [ScrollingModule],
  template: `
    <h2>Virtual Scrolling</h2>
    
    <div class="virtual-scroll-container">
      <h3>5000 Items</h3>
      <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
        <div *cdkVirtualFor="let item of items; let i = index" class="item">
          Item #{{i}}: {{item}}
        </div>
      </cdk-virtual-scroll-viewport>
    </div>
  `,
  styles: [`
    .virtual-scroll-container {
      width: 500px;
      max-width: 100%;
    }
    
    .viewport {
      height: 400px;
      width: 100%;
      border: 1px solid black;
    }
    
    .item {
      height: 50px;
      padding: 10px;
      box-sizing: border-box;
      border-bottom: 1px solid #ccc;
      display: flex;
      align-items: center;
    }
    
    .item:nth-child(odd) {
      background-color: #f5f5f5;
    }
  `]
})
export class VirtualScrollDemoComponent {
  items = Array.from({length: 5000}).map((_, i) => `Item #${i}`);
}
```

#### 3. Overlay - Custom Tooltip
```typescript
@Component({
  selector: 'app-custom-tooltip',
  standalone: true,
  imports: [OverlayModule, PortalModule],
  template: `
    <div
      class="tooltip-trigger"
      (mouseenter)="show()"
      (mouseleave)="hide()">
      Hover me for custom tooltip
    </div>
    
    <ng-template #tooltipTemplate>
      <div class="tooltip-content">
        <h3>Custom Tooltip</h3>
        <p>This is a custom tooltip built with CDK Overlay</p>
      </div>
    </ng-template>
  `,
  styles: [`
    .tooltip-trigger {
      padding: 10px;
      background: #f0f0f0;
      border: 1px solid #ccc;
      display: inline-block;
      cursor: pointer;
    }
    
    .tooltip-content {
      padding: 10px;
      background: black;
      color: white;
      border-radius: 4px;
      max-width: 200px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    }
  `]
})
export class CustomTooltipComponent {
  @ViewChild('tooltipTemplate') tooltipTemplate!: TemplateRef<any>;
  private overlayRef: OverlayRef | null = null;
  
  constructor(private overlay: Overlay, private viewContainerRef: ViewContainerRef) {}
  
  show() {
    if (this.overlayRef) {
      return;
    }
    
    const positionStrategy = this.overlay
      .position()
      .flexibleConnectedTo(this.viewContainerRef.element.nativeElement)
      .withPositions([
        { originX: 'center', originY: 'bottom', overlayX: 'center', overlayY: 'top', offsetY: 5 },
        { originX: 'center', originY: 'top', overlayX: 'center', overlayY: 'bottom', offsetY: -5 },
      ]);
    
    this.overlayRef = this.overlay.create({
      positionStrategy,
      scrollStrategy: this.overlay.scrollStrategies.close()
    });
    
    const portal = new TemplatePortal(
      this.tooltipTemplate,
      this.viewContainerRef
    );
    
    this.overlayRef.attach(portal);
  }
  
  hide() {
    if (this.overlayRef) {
      this.overlayRef.detach();
      this.overlayRef.dispose();
      this.overlayRef = null;
    }
  }
}
```

## 🧩 รวม Components ทั้งหมด (Demo App)

```typescript
@Component({
  selector: 'app-material-demo',
  standalone: true,
  imports: [
    // Components
    ButtonDemoComponent,
    ProgressDemoComponent,
    InputDemoComponent,
    SelectDemoComponent,
    NavigationDemoComponent,
    TabsDemoComponent,
    TableDemoComponent,
    DialogDemoComponent,
    DragDropDemoComponent,
    VirtualScrollDemoComponent,
    CustomTooltipComponent,
    // Modules
    MatTabsModule,
    MatDividerModule,
    MatExpansionModule
  ],
  template: `
    <mat-toolbar color="primary">
      <span>Angular Material & CDK Demo</span>
    </mat-toolbar>
    
    <div class="demo-container">
      <mat-tab-group>
        <mat-tab label="Form Controls">
          <div class="tab-content">
            <app-input-demo></app-input-demo>
            <mat-divider></mat-divider>
            <app-select-demo></app-select-demo>
          </div>
        </mat-tab>
        
        <mat-tab label="Navigation">
          <div class="tab-content">
            <app-navigation-demo></app-navigation-demo>
            <mat-divider></mat-divider>
            <app-tabs-demo></app-tabs-demo>
          </div>
        </mat-tab>
        
        <mat-tab label="Layout">
          <div class="tab-content">
            <app-button-demo></app-button-demo>
            <mat-divider></mat-divider>
            <app-progress-demo></app-progress-demo>
          </div>
        </mat-tab>
        
        <mat-tab label="Data Table">
          <div class="tab-content">
            <app-table-demo></app-table-demo>
          </div>
        </mat-tab>
        
        <mat-tab label="Popups">
          <div class="tab-content">
            <app-dialog-demo></app-dialog-demo>
            <mat-divider></mat-divider>
            <app-custom-tooltip></app-custom-tooltip>
          </div>
        </mat-tab>
        
        <mat-tab label="CDK">
          <div class="tab-content">
            <mat-accordion>
              <mat-expansion-panel>
                <mat-expansion-panel-header>
                  <mat-panel-title>Drag & Drop</mat-panel-title>
                </mat-expansion-panel-header>
                <app-dnd-demo></app-dnd-demo>
              </mat-expansion-panel>
              
              <mat-expansion-panel>
                <mat-expansion-panel-header>
                  <mat-panel-title>Virtual Scroll</mat-panel-title>
                </mat-expansion-panel-header>
                <app-virtual-scroll-demo></app-virtual-scroll-demo>
              </mat-expansion-panel>
            </mat-accordion>
          </div>
        </mat-tab>
      </mat-tab-group>
    </div>
  `,
  styles: [`
    .demo-container {
      padding: 20px;
      max-width: 1200px;
      margin: 0 auto;
    }
    
    .tab-content {
      padding: 20px 0;
    }
    
    mat-divider {
      margin: 30px 0;
    }
  `]
})
export class MaterialDemoComponent {}
```

## 🛠️ Best Practices

### 1. Optimize Performance
- ใช้ `OnPush` change detection strategy
- ลด CSS custom styles และใช้ theming system
- แยก import เฉพาะ modules ที่ใช้งาน

```typescript
// BAD
import { MaterialModule } from '@angular/material'; // Deprecated and not recommended

// GOOD
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
```

### 2. Theming Consistency
- สร้าง custom theme และ reuse
- กำหนด typography และ color scheme ที่คงเส้นคงวา
- ใช้ SCSS mixins และ variables

### 3. Accessibility
- ใช้ built-in accessibility features
- ทดสอบกับ screen readers
- ใช้ `aria` attributes เมื่อจำเป็น
- เพิ่ม keyboard navigation

### 4. Mobile Responsiveness
- ใช้ Flex Layout
- ทดสอบบนหลายขนาดหน้าจอ
- ปรับ density options สำหรับหน้าจอขนาดเล็ก

## 🎯 แบบฝึกหัด

### Exercise 1: สร้าง Login Form ด้วย Angular Material
```typescript
@Component({
  selector: 'app-login-form',
  standalone: true,
  imports: [
    MatCardModule,
    MatFormFieldModule,
    MatInputModule,
    MatButtonModule,
    MatCheckboxModule,
    ReactiveFormsModule
  ],
  template: `
    <mat-card class="login-card">
      <mat-card-header>
        <mat-card-title>Login</mat-card-title>
      </mat-card-header>
      
      <mat-card-content>
        <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
          <mat-form-field appearance="outline">
            <mat-label>Username</mat-label>
            <input matInput formControlName="username" required>
            <mat-error *ngIf="loginForm.get('username')?.hasError('required')">
              Username is required
            </mat-error>
          </mat-form-field>
          
          <mat-form-field appearance="outline">
            <mat-label>Password</mat-label>
            <input matInput formControlName="password" [type]="hidePassword ? 'password' : 'text'" required>
            <button mat-icon-button matSuffix (click)="hidePassword = !hidePassword" type="button">
              <mat-icon>{{hidePassword ? 'visibility_off' : 'visibility'}}</mat-icon>
            </button>
            <mat-error *ngIf="loginForm.get('password')?.hasError('required')">
              Password is required
            </mat-error>
          </mat-form-field>
          
          <div class="form-row">
            <mat-checkbox formControlName="rememberMe">Remember me</mat-checkbox>
          </div>
          
          <div class="form-actions">
            <button mat-raised-button color="primary" type="submit" [disabled]="loginForm.invalid">
              Login
            </button>
            <button mat-button type="button">Forgot Password</button>
          </div>
        </form>
      </mat-card-content>
    </mat-card>
  `,
  styles: [`
    .login-card {
      max-width: 400px;
      margin: 0 auto;
      margin-top: 50px;
    }
    
    mat-form-field {
      width: 100%;
      margin-bottom: 15px;
    }
    
    .form-row {
      margin-bottom: 20px;
    }
    
    .form-actions {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
  `]
})
export class LoginFormComponent {
  loginForm = new FormGroup({
    username: new FormControl('', [Validators.required]),
    password: new FormControl('', [Validators.required]),
    rememberMe: new FormControl(false)
  });
  
  hidePassword = true;
  
  onSubmit() {
    if (this.loginForm.valid) {
      console.log('Login:', this.loginForm.value);
    }
  }
}
```

### Exercise 2: สร้าง Data Dashboard ด้วย CDK
```typescript
// Challenge: Create a dashboard with:
// 1. Card layout using FlexModule
// 2. Data table with sorting and filtering
// 3. Charts (you can use a library like ng2-charts)
// 4. Responsive design for mobile/desktop
```

## 🧪 Quiz

### Question 1
Angular Material ทำงานได้โดยไม่ต้องติดตั้ง:
- a) Angular CDK
- b) RxJS
- c) Bootstrap
- d) jQuery

### Question 2
สิ่งใดไม่ใช่ความสามารถของ Angular CDK:
- a) Virtual Scrolling
- b) Drag and Drop
- c) Material-design Components
- d) Overlay

### Question 3
Angular Material theme ประกอบด้วย:
- a) Primary, Accent และ Warn colors
- b) Background และ Foreground colors
- c) Typography
- d) ทั้งหมดที่กล่าวมา

**คำตอบ: 1-b, 2-c, 3-d**

## 📝 สรุป

ในบทนี้เราได้เรียนรู้:

### **🛠️ Angular Material**
1. **การติดตั้งและตั้งค่า** Angular Material
2. **การปรับแต่ง themes** และ typography
3. **Material Components** ที่สำคัญ เช่น buttons, forms, navigation, tables, dialogs

### **📦 Component Dev Kit (CDK)**
1. **Building blocks** สำหรับสร้าง custom components
2. **Advanced functionality** เช่น drag-drop, virtual scrolling, overlay
3. **Platform-agnostic tools** สำหรับ accessibility และ UX

### **💎 Best Practices**
1. **Performance optimization** ด้วย selective imports และ OnPush strategy
2. **Responsive design** สำหรับ mobile และ desktop
3. **Accessibility** เพื่อรองรับผู้ใช้ทุกกลุ่ม

ใน**บทถัดไป** เราจะเรียนรู้เกี่ยวกับการปรับแต่ง performance ใน Angular applications

---

**[⬅️ บทที่ 9: State Management](./09-state-management.md) | [➡️ บทที่ 11: Advanced Performance](./11-advanced-performance.md)**
