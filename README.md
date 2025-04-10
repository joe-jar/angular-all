Angular Masterclass — Chapters 1 to 12 (Expert Level, Expanded)

Chapter 1: TypeScript Essentials
Angular relies heavily on TypeScript's capabilities to ensure type safety, maintainability, and developer tooling support.
- **Classes & Constructors**: Used to define components, services, and models.
- **Interfaces**: Promote clear data shapes for models and HTTP response contracts.
- **Access Modifiers**: Enhance encapsulation in services and components (e.g., private service methods).
- **Decorators**: Fundamental in Angular (`@Component`, `@Injectable`, `@Input`, `@Output`). Allow metadata-based design.
- **Optional Chaining**: Prevent null access errors: `user?.address?.street`
- **Nullish Coalescing**: Default fallback: `username ?? 'Guest'`
- **Generics**: Ensure reusable and safe APIs: `Observable<T>`, `FormControl<T>`, etc.
- **Enums**: Represent state and categories cleanly (e.g., form status, roles).

Extended usage example:
```ts
export interface Hero {
  id: number;
  name: string;
  power?: string;
  rating: number;
}

@Injectable({ providedIn: 'root' })
export class HeroService {
  private heroes: Hero[] = [
    { id: 1, name: 'Windstorm', rating: 3 },
    { id: 2, name: 'Magneta', power: 'Magnetism', rating: 5 }
  ];

  getHeroById(id: number): Hero | undefined {
    return this.heroes.find(h => h.id === id);
  }

  filterHeroes(minRating: number): Hero[] {
    return this.heroes.filter(h => h.rating >= minRating);
  }
}
```

Chapter 2: Project Setup (Standalone vs NgModule)
- **Standalone Setup**: Introduced in Angular 14+, full support in v17+. It simplifies configuration.
  - No NgModule needed
  - Components marked with `standalone: true`
  - Use `provideRouter`, `importProvidersFrom`
- **Traditional Setup**: Uses `AppModule`, `RouterModule.forRoot()`

Routing example:
```ts
export const routes: Routes = [
  { path: '', loadComponent: () => import('./home/home.component').then(m => m.HomeComponent) },
  { path: 'admin', loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent), canActivate: [AdminGuard] }
];
```

Folder structure best practices:
```
/src
  /core
    auth.service.ts
    http.interceptor.ts
  /shared
    button.component.ts
    pipe.module.ts
  /features
    /dashboard
    /settings
```

Chapter 3: Components, Templates & Binding
- **@Component**: Declares selector, template, styles, and component metadata.
- **Interpolation**: Embed string expressions into HTML
- **Property Binding**: One-way data flow to child
- **Event Binding**: Respond to user interactions
- **Two-way Binding**: `[(ngModel)]` (FormsModule)
- **@ViewChild**: Access child components or elements
- **Standalone Components**: Declared as `standalone: true` and used directly in `imports`

Chapter 4: Parent ↔ Child Communication
- `@Input()`: Receive values from parent
- `@Output()`: Emit values with `EventEmitter`
- Also possible: use shared services + `BehaviorSubject`

Pattern:
```ts
@Input() userId: number;
@Output() delete = new EventEmitter<number>();
```

RxJS-powered `@Input()`:
```ts
@Input() set searchTerm(value: string) {
  this.term$.next(value);
}
```

Chapter 5: Directives
- Built-in Structural: `*ngIf`, `*ngFor`, `*ngSwitch`
- Attribute: `[ngClass]`, `[ngStyle]`
- Logical structure: `<ng-container>` = no extra DOM
- Template control: `<ng-template>` + `TemplateRef`
- Custom Directives: Use `@Directive` to change appearance or behavior

Custom directive example:
```ts
@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {
  @HostListener('mouseenter') onEnter() {...}
}
```

Chapter 6: Services & Dependency Injection
- Angular uses a hierarchical injector.
- Services can be scoped (root/component level)
- Provide dependencies in root (singleton), or in `providers` array
- Advanced: use `InjectionToken`, multi-provider arrays

Common service use cases:
- API clients
- State management
- MessageBus (Subjects)
- Authentication

Chapter 7: Routing & Navigation
- Define lazy-loaded routes via `loadChildren` or `loadComponent`
- Apply guards with `canActivate`, `canLoad`, `resolve`
- Use `ActivatedRoute` for params
- Use `RouterLinkActive` for active tab logic
- Redirects: `{ path: '', redirectTo: 'home', pathMatch: 'full' }`

Navigation example:
```ts
this.router.navigate(['/user', userId], { queryParams: { edit: true } });
```

Chapter 8: RxJS Essentials
- Create observable with `of`, `interval`, `fromEvent`, `new Observable()`
- `subscribe` and manage `unsubscribe` via `Subscription`
- Combine observables with `combineLatest`, `withLatestFrom`
- Handle side effects with `tap()`
- Use `takeUntil()` for cleanup

Best practices:
- Unsubscribe on destroy (`takeUntil`, `ngOnDestroy`)
- Never subscribe inside service (let component handle it)
- Prefer pipeable operators

Chapter 9: RxJS Subject Types
Comparison table:
| Type            | Remembers Value | Emits to Late Subscribers | Example Use                      |
|------------------|------------------|----------------------------|----------------------------------|
| Subject          | ❌ No            | ❌ No                      | Button click events              |
| BehaviorSubject  | ✅ Yes           | ✅ Latest only            | Current user / theme             |
| ReplaySubject(2) | ✅ Last 2        | ✅ Yes                    | Chat history                     |
| AsyncSubject     | ✅ Final only    | ✅ On complete            | Load-once operations             |

Chapter 10: RxJS Operators
Operator logic:
- `switchMap`: cancel prior async
- `concatMap`: sequence async
- `mergeMap`: fire all at once
- `exhaustMap`: ignore if busy

Advanced:
- `scan`: like reduce
- `shareReplay`: cache observable
- `debounceTime`: for autocomplete
- `finalize`: cleanup
- `distinctUntilChanged`: prevent re-triggers

Chapter 11: Angular Forms
Reactive advantages:
- Dynamic forms with array of controls
- Use `FormArray`, `FormBuilder`
- Listen with `.valueChanges`
- Conditionally disable/enable
- Strong validation handling

Validation Example:
```ts
form = fb.group({
  email: ['', [Validators.required, Validators.email]],
  age: [null, [Validators.min(18)]]
});
```

Async validator:
```ts
checkUsername(control: AbstractControl): Observable<ValidationErrors | null> {
  return timer(500).pipe(
    switchMap(() => this.api.check(control.value)),
    map(exists => exists ? { taken: true } : null)
  );
}
```

Chapter 12: HttpClient
Advanced HttpClient techniques:
- Set global headers with `HttpInterceptor`
- Automatically retry on failure
- Cancel pending HTTP requests
- Use `HttpParams` to build query params
- Chain requests using `switchMap`

HttpOptions:
```ts
const options = {
  headers: new HttpHeaders({ Authorization: 'Bearer ' + token }),
  params: new HttpParams().set('sort', 'desc')
};
```

Interceptors:
```ts
intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
  const authReq = req.clone({ setHeaders: { Authorization: 'Bearer XYZ' } });
  return next.handle(authReq);
}
```

