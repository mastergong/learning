# 10 — Angular Material & CDK

## Install
```powershell
ng add @angular/material
```

## Theming & Components
```ts
@Component({
  selector: 'mat-demo',
  standalone: true,
  imports: [MatButtonModule, MatToolbarModule],
  template: `
    <mat-toolbar color="primary">Toolbar</mat-toolbar>
    <button mat-raised-button color="accent">Click</button>
  `
})
export class MatDemoComponent {}
```

## CDK Essentials
- Overlay, Portal, DragDrop, A11y
