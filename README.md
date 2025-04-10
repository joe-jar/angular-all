# 📘 Angular Course — Component Communication & RxJS Focus

This focused course explores all the communication patterns in Angular, including child-parent, cross-component, and deep RxJS operator and subject mastery. Updated with additional details and insights.

---

## 🔸 Part 1 — Parent ↔ Child Communication

### 1.1 `@Input()` — Pass Data from Parent to Child
Used when the parent component needs to provide data to a child component. The value flows one-way from parent to child.

```ts
@Component({ selector: 'child', standalone: true, template: '{{ title }}' })
export class ChildComponent {
  @Input() title!: string;
}
```
```html
<child [title]="parentTitle"></child>
```

### 1.2 `@Output()` — Emit Data from Child to Parent
Allows the child to emit custom events. The parent listens using standard event binding.

```ts
@Component({ selector: 'child', standalone: true })
export class ChildComponent {
  @Output() clicked = new EventEmitter<string>();
  onClick() { this.clicked.emit('Hello'); }
}
```
```html
<child (clicked)="handleClick($event)"></child>
```

### 1.3 Setters with `@Input()` — React to Input Changes
Use a setter for `@Input()` to process or react to changes.
```ts
@Input() set value(val: string) {
  this.internal = val?.toUpperCase();
}
```
Also combine with `ngOnChanges()` lifecycle hook for deeper control.

### 1.4 ViewChild Access
Parent can access child methods or properties directly using `@ViewChild()`.
```ts
@ViewChild(ChildComponent) child!: ChildComponent;
```

---

## 🔸 Part 2 — Unrelated Component Communication

### 2.1 Shared Service with `Subject`
Ideal for passing values across unrelated components (e.g., sibling components).

```ts
@Injectable({ providedIn: 'root' })
export class MessagingService {
  private subject = new Subject<string>();
  message$ = this.subject.asObservable();
  sendMessage(msg: string) { this.subject.next(msg); }
}
```
```ts
// Sender Component
this.messaging.sendMessage('Hi');

// Receiver Component
this.messaging.message$.subscribe(msg => this.received = msg);
```

### 2.2 `BehaviorSubject` to Share and Cache State
Always returns the most recent value to new subscribers.

```ts
@Injectable({ providedIn: 'root' })
export class AuthService {
  private userSubject = new BehaviorSubject<User | null>(null);
  user$ = this.userSubject.asObservable();
  update(user: User) { this.userSubject.next(user); }
}
```
Useful for:
- Auth status
- User preferences
- Shared config/data

---

## 🔸 Part 3 — RxJS Subject Types Explained

| Type             | Caches Last | Emits to Late Subscribers | When to Use                     |
|------------------|-------------|----------------------------|----------------------------------|
| `Subject`        | ❌          | ❌                         | One-time events (e.g., click)    |
| `BehaviorSubject`| ✅          | ✅                         | Shared reactive state (auth)     |
| `ReplaySubject`  | ✅ (N)      | ✅                         | Chat history, logs               |
| `AsyncSubject`   | ✅ (final)  | ✅ (on complete)           | Final-only emissions (e.g., upload result)

```ts
const s = new Subject<string>();
s.subscribe(val => console.log('Late sub:', val));
s.next('Emitted!');
```

```ts
const b = new BehaviorSubject('initial');
b.subscribe(console.log); // receives 'initial'
b.next('new');
```

---

## 🔸 Part 4 — RxJS Operators Deep Dive

### 4.1 `concatMap` — Sequential Mapping
Maintains order by waiting for the previous observable to complete.
```ts
from([1, 2, 3]).pipe(
  concatMap(id => http.get(`/api/${id}`))
).subscribe(console.log);
```
Use cases:
- Step-by-step processing (form wizard)
- Ordered logging or transactions

### 4.2 `mergeMap` — Parallel Mapping
Does not wait; fires all observables in parallel.
```ts
from([1, 2, 3]).pipe(
  mergeMap(id => http.get(`/api/${id}`))
).subscribe(console.log);
```
Use cases:
- Bulk async operations
- Realtime chat, events, telemetry

### 4.3 `switchMap` — Cancel Previous
Cancels previous if new emission arrives before it completes.
```ts
search$.pipe(
  debounceTime(300),
  switchMap(term => http.get(`/api/search?q=${term}`))
).subscribe(console.log);
```
Use cases:
- Search autocomplete
- Input-based filtering
- Reactive forms

### 4.4 `exhaustMap` — Ignore If Busy
Ignores incoming values while previous one is processing.
```ts
click$.pipe(
  exhaustMap(() => http.post('/api/submit'))
).subscribe(console.log);
```
Use cases:
- Prevent double form submission
- Debounced save buttons

---

## ✅ Summary
- Use `@Input()` and `@Output()` for parent-child communication.
- Use shared services + `Subject`/`BehaviorSubject` for cross-component.
- Understand behavior differences between RxJS subjects.
- Choose mapping operator based on task behavior:
  - `switchMap` for replaceable
  - `concatMap` for ordered
  - `mergeMap` for parallel
  - `exhaustMap` for busy lockout

Bonus: Combine with `tap`, `finalize`, `retry`, and `takeUntil()` for smart cleanup.


# 📘 Angular Routing — Standalone vs Traditional with Lazy Loading

A complete guide comparing modern standalone routing and legacy NgModule routing approaches in Angular. Focused on lazy loading patterns.

---

## ✅ Standalone Lazy Loaded Routing (Angular 15+)

**Simplified approach using `provideRouter()` and `loadComponent()`.**

```ts
// app.routes.ts
export const appRoutes: Routes = [
  {
    path: '',
    loadComponent: () => import('./features/home/home.component')
      .then(m => m.HomeComponent)
  },
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.component')
      .then(m => m.DashboardComponent)
  }
];
```

```ts
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';
import { appRoutes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [provideRouter(appRoutes)]
});
```

---

## 🧱 Traditional NgModule Lazy Loading (Pre-Standalone)

**Module-based routing using `loadChildren()` and `AppRoutingModule`.**

```ts
// app-routing.module.ts
const routes: Routes = [
  {
    path: '',
    loadChildren: () => import('./features/home/home.module')
      .then(m => m.HomeModule)
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./features/dashboard/dashboard.module')
      .then(m => m.DashboardModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

```ts
// app.module.ts
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, AppRoutingModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## 🔄 Comparison Table

| Feature               | Standalone Approach         | Traditional NgModule Approach   |
|-----------------------|-----------------------------|----------------------------------|
| Bootstrapping         | `bootstrapApplication()`    | `NgModule.bootstrap`            |
| Routing setup         | `provideRouter()`           | `RouterModule.forRoot()`        |
| Lazy loading          | `loadComponent()`           | `loadChildren()`                |
| Complexity            | ✅ Lower                     | ❌ Higher (more boilerplate)    |
| IDE Autocomplete      | ✅ Better                    | ⚠️ Depends on declarations       |
| Tree shaking          | ✅ More efficient            | ⚠️ Less optimized                |
| Preferred in Angular  | ✅ Angular 15+ recommended   | ⚠️ Legacy-compatible             |

---

## ✅ Best Practices

- Prefer `loadComponent()` and standalone setup for modern apps (v15+)
- Keep route declarations in one file (`app.routes.ts`)
- Lazy load major feature views for performance
- Use guards and `canMatch` for protected routes
- Structure routes cleanly with child paths if needed

---

Say the word if you'd like to add **guards**, **route params**, or **named outlets** next.

# 📘 Angular Forms — Template-Driven vs Reactive Forms

A detailed comparison between the two main form-handling approaches in Angular: **Template-Driven Forms** and **Reactive Forms**. This guide covers syntax, structure, validation, and best practices.

---

## ✅ Overview

| Feature                  | Template-Driven Forms             | Reactive Forms                    |
|--------------------------|-----------------------------------|------------------------------------|
| Setup                    | Minimal, uses HTML + `ngModel`    | Programmatic, uses `FormGroup`     |
| Ideal Use Case           | Simple forms, quick dev           | Complex forms, dynamic fields      |
| Code Location            | Mostly in the template            | Mostly in the component (.ts)      |
| Validation               | Template-based                    | API-driven (Validators API)        |
| Testability              | ⚠️ Less testable                  | ✅ Highly testable                 |
| Form State Handling      | Automatically tracked             | Explicit control                   |
| Async Validators         | ❌ Hard to manage                  | ✅ Built-in support                |

---

## 🔹 Template-Driven Forms

### 1. Setup
Import `FormsModule` in your `AppModule` (or component’s standalone `imports`):
```ts
import { FormsModule } from '@angular/forms';
```

### 2.1 Form HTML
```html
<form #form="ngForm" (ngSubmit)="submitForm(form)">
  <input name="email" [(ngModel)]="email" required email />
  <button type="submit">Submit</button>
</form>
```

### 3. Features
- `ngModel` for two-way binding
- `#form="ngForm"` gives access to form state
- Validation through attributes (`required`, `email`, etc.)
- Useful for smaller, static forms

### 4. Accessing Form State
```ts
submitForm(form: NgForm) {
  if (form.valid) {
    console.log(form.value);
  }
}
```

---

## 🔸 Reactive Forms

### 1. Setup
Import `ReactiveFormsModule` in your `AppModule` or standalone imports:
```ts
import { ReactiveFormsModule } from '@angular/forms';
```

### 2.2 With FormBuilder
A cleaner and more scalable way to build forms.

```ts
constructor(private fb: FormBuilder) {}

form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(6)]]
});
```

### 3. Features
- Control form structure and validation in code
- Reactive `valueChanges` observable
- Easy to test and extend
- Dynamic form generation possible with `FormArray`

### 3.1 Accessing and Validating
```ts
submitForm() {
  if (this.form.valid) {
    console.log(this.form.value);
  }
}
```

### 3.2 Async Validators
```ts
emailControl = new FormControl('', null, [myAsyncValidator]);
```

---

## 🔍 When to Use Which?

### Use Template-Driven When:
- You need a quick form with basic fields
- Validation is simple and declarative
- Form is not deeply tested or reused

### Use Reactive When:
- You need programmatic control of validation
- Complex state, custom + async validation
- You want dynamic forms (add/remove controls)
- Testability and maintainability matter

---

# 📘 Angular — Components, Lifecycle Hooks, Directives, Pipes & Modules

A deep dive into the foundational building blocks of Angular, including modern standalone patterns introduced since Angular v15+.

---

## ✅ Components

### What is a Component?
An Angular component:
- Controls a portion of the UI
- Includes a template (HTML), class (TypeScript), and optional styles (CSS/SCSS)
- Is declared using the `@Component()` decorator

### Classic Component Example (Standalone)
```ts
@Component({
  selector: 'app-welcome',
  standalone: true,
  template: `<h1>Welcome, {{ name }}</h1>`,
  imports: [CommonModule]
})
export class WelcomeComponent {
  name = 'Angular';
}
```

### Lifecycle Hooks
These are special methods Angular calls at specific moments:

| Hook               | When it runs                            |
|--------------------|------------------------------------------|
| `ngOnInit()`       | After inputs are set                     |
| `ngOnChanges()`    | On any input property change             |
| `ngAfterViewInit()`| After the component’s view is initialized|
| `ngOnDestroy()`    | Just before the component is destroyed   |

#### Example:
```ts
export class LifecycleDemoComponent implements OnInit, OnDestroy {
  ngOnInit() {
    console.log('Component initialized');
  }

  ngOnDestroy() {
    console.log('Component destroyed');
  }
}
```

---

## ✅ Directives
Directives are instructions in the DOM:

### Built-In Structural
- `*ngIf`: Conditionally includes a template
- `*ngFor`: Iterates over collections
- `*ngSwitch`: Multi-conditional rendering

### Attribute Directives
Modify the appearance or behavior of an element:
```html
<div [ngClass]="{active: isActive}"></div>
<input [disabled]="isDisabled" />
```

### Custom Directive Example (Standalone)
```ts
@Directive({ selector: '[appBorderHighlight]', standalone: true })
export class BorderHighlightDirective {
  @HostListener('mouseenter') onMouseEnter() {
    this.el.nativeElement.style.border = '2px solid blue';
  }
  @HostListener('mouseleave') onMouseLeave() {
    this.el.nativeElement.style.border = '';
  }
  constructor(private el: ElementRef) {}
}
```

---

## ✅ Pipes
Transform output in templates.

### Built-In Pipes
- `date`, `uppercase`, `lowercase`, `currency`, `percent`

### Custom Pipe Example (Standalone)
```ts
@Pipe({ name: 'initials', standalone: true })
export class InitialsPipe implements PipeTransform {
  transform(name: string): string {
    return name.split(' ').map(n => n[0]).join('').toUpperCase();
  }
}
```
```html
<p>{{ 'John Doe' | initials }}</p> <!-- Output: JD -->
```

---

## ✅ Modules vs Standalone

### Traditional NgModule Approach
```ts
@NgModule({
  declarations: [AppComponent, MyDirective, MyPipe],
  imports: [BrowserModule, FormsModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### Standalone Component Approach
Angular v15+ lets you:
- Skip NgModules
- Use `standalone: true` in components, pipes, and directives
- Bootstrap with `bootstrapApplication()`

#### Standalone Bootstrap Example
```ts
bootstrapApplication(AppComponent, {
  providers: [provideRouter(appRoutes)]
});
```

### Example of Composing Standalone Imports
```ts
@Component({
  selector: 'app-root',
  standalone: true,
  template: '<app-welcome></app-welcome>',
  imports: [WelcomeComponent, BorderHighlightDirective, InitialsPipe]
})
export class AppComponent {}
```

---










