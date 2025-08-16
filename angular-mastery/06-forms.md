# 06 — Forms (Template-driven & Reactive)

## Template-driven (quick)
```ts
@Component({
  selector: 'contact-form',
  standalone: true,
  imports: [FormsModule],
  template: `
    <form #f="ngForm" (ngSubmit)="save(f.value)">
      <input name="name" [(ngModel)]="model.name" required />
      <input name="email" [(ngModel)]="model.email" email />
      <button [disabled]="f.invalid">Save</button>
    </form>
  `
})
export class ContactFormComponent {
  model = { name: '', email: '' };
  save(v:any){ console.log(v); }
}
```

## Reactive Forms (recommended for complex)
```ts
@Component({
  selector: 'profile-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="submit()">
      <input formControlName="name" />
      <input formControlName="email" />
      <div *ngIf="email.invalid && email.touched">Invalid email</div>
      <button [disabled]="form.invalid">Save</button>
    </form>
  `
})
export class ProfileFormComponent {
  form = this.fb.group({ name: ['', Validators.required], email: ['', [Validators.required, Validators.email]] });
  constructor(private fb: FormBuilder) {}
  get email(){ return this.form.controls.email!; }
  submit(){ console.log(this.form.value); }
}
```

## Custom Validators
```ts
export function domainEmail(domain: string): ValidatorFn {
  return (c: AbstractControl) => c.value?.endsWith('@'+domain) ? null : { domain: true };
}
```
