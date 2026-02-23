# ANGULAR INTERVIEW NOTES (ENHANCED)

---

## TOPIC: What is Node.js?

- **Definition:** Node.js is a runtime built on Chrome's V8 engine that lets you run JavaScript outside the browser.
- **Key Features:** Non-blocking I/O, event-driven architecture, npm ecosystem for packages.
- **Why It Matters:** Provides tooling (npm, Angular CLI), local dev server, build process for Angular apps.

---

## TOPIC: Why Node.js is required in an Angular project

- Angular CLI depends on Node.js and npm to:
  - Install dependencies (npm/yarn)
  - Run dev server (`ng serve`)
  - Build and bundle (`ng build`)
  - Run tests and linting
- Node.js is **NOT** required at runtime in the browser (Angular runs in the browser), but it is essential for development tooling.

---

## TOPIC: Angular — Overview

- A framework for building client-side Single Page Applications (SPAs) using TypeScript.
- Opinionated structure with powerful tooling (CLI, schematics, testing).

---

## TOPIC: Core Features of Angular

1. **Component-based architecture**
   - UI is composed of reusable components (view + logic).
   - Encourages separation of concerns and reusability.
2. **Two-way data binding**
   - Using `ngModel` (Forms) syncs model ⇄ view.
   - Reduces boilerplate for form inputs.
3. **Dependency Injection (DI)**
   - Services provided via injectors; promotes testability.
4. **Directives**
   - Structural (e.g., `*ngIf`, `*ngFor`) and Attribute (e.g., `[ngClass]`, `[ngStyle]`).
5. **Routing**
   - Client-side navigation using `RouterModule`; supports parameters, guards, lazy loading.
6. **Reactive programming**
   - RxJS Observables, operators, async pipe for async data.
7. **Cross-platform**
   - Works in browsers; supports SSR (Angular Universal) and PWAs.

---

## TOPIC: Single Page Application (SPA)

- **What is it?**
  - An app that loads a single HTML shell and updates the view dynamically using JavaScript.
- **How does it work?**
  - Router intercepts navigation; only parts of the DOM update.
  - API calls fetch data; no full page reloads.
  - Improves perceived performance and UX.

---

## TOPIC: Angular CLI — Common Commands

Angular CLI stands for **Angular Command Line Interface**. It's a powerful tool provided by Angular to create, build, test, and manage Angular applications easily from the terminal.

| Command                               | Description                |
| ------------------------------------- | -------------------------- |
| `ng new first-angular-app`            | Create app                 |
| `ng serve` / `ng serve --open`        | Start dev server           |
| `ng g c my-first-component`           | Create component           |
| `ng g s user`                         | Create service             |
| `ng g m admin --routing`              | Create module with routing |
| `ng build --configuration production` | Build prod                 |

---

## TOPIC: Angular Project Structure — File Meanings

| File/Folder           | Purpose                                                                              |
| --------------------- | ------------------------------------------------------------------------------------ |
| `package.json`        | Project dependencies and scripts                                                     |
| `angular.json`        | Angular workspace configuration (build, serve, test)                                 |
| `tsconfig.json`       | TypeScript compiler options                                                          |
| `src/main.ts`         | Entry point; bootstraps the app (`bootstrapApplication` or `platformBrowserDynamic`) |
| `src/index.html`      | App host page with root element                                                      |
| `src/styles.css/scss` | Global styles                                                                        |
| `src/app/`            | Application source (components, modules, services)                                   |
| `environments/`       | Environment-specific settings (dev/prod)                                             |

---

## TOPIC: How a Component Loads in the Browser

**Bootstrap process:**

1. `main.ts` starts the app (`bootstrapApplication(AppComponent)` or `bootstrapModule(AppModule)`).
2. Root component renders into the root element in `index.html`.
3. Router renders component views into `<router-outlet>`.
4. Change detection updates the DOM when data changes.

---

## TOPIC: Component

**Component** — It is the basic building block of a UI which encapsulates HTML, CSS, and TypeScript template in it.

---

## TOPIC: Standalone Components

Components that manage their own dependencies without needing an `@NgModule`. As of Angular 17/18, this is the default approach. Angular previously required every component to be declared inside an `NgModule`. Standalone components remove this requirement.

---

## TOPIC: Change Detection Strategy

**Change Detection** is the process Angular uses to keep the view (DOM) in sync with the component's data.

- Angular checks the entire component tree from top to bottom on every browser event (click, timer, HTTP response).
- Angular uses **Zone.js**

---

## TOPIC: Data Binding in Angular

1. **Interpolation (one-way)**
   - Syntax: `{{ expression }}`
   - Binds component data to text nodes.

2. **Property Binding (one-way)**
   - Syntax: `[property]="expression"`
   - Example: `<img [src]="avatarUrl">`
   - **a) Class Binding**
     - Syntax: `[class.active]="isActive"` or `[ngClass]="{active: isActive}"`
   - **b) Style Binding**
     - Syntax: `[style.color]="color"` or `[ngStyle]="{color: color}"`
     - Best practice: Prefer classes (`ngClass`) for multiple styles; use `ngStyle` for dynamic inline values.

3. **Event Binding (one-way from view to component)**
   - Syntax: `(event)="onEvent($event)"`
   - Example: `<button (click)="onClick()">Click</button>`

4. **Two-way Binding**
   - Syntax: `[(ngModel)]="value"` (requires `FormsModule`)
   - Combines property + event binding; ideal for form inputs.

---

## TOPIC: Directive vs Decorator (Definitions)

- **Directive:** Class that adds behavior to elements (e.g., `*ngIf`, `*ngFor`, custom attribute directives). Directives are instructions that modify the DOM.
- **Decorator:** TypeScript metadata (e.g., `@Component`, `@Directive`, `@Injectable`, `@ViewChild`) used by Angular to configure classes and properties.

---

## TOPIC: Angular Directives — Types and Examples

### 1) Component Directive

A directive with a template; every component is a directive. A component directive is a special type of directive in Angular that has its own template and styles. It is created using the `@Component` decorator and is used to define a reusable UI block.

### 2) Structural Directives

Change the DOM structure by adding/removing elements.

- **`*ngIf`:** Conditionally render
  - Old way: `<div *ngIf="isLoggedIn; else guest"></div>`
  - New way: `@If(){<div>}`
- **`*ngFor`:** Loop items — `<li *ngFor="let item of items; index as i">{{i}} {{item}}</li>`
- **`*ngSwitch`:** Conditional rendering —
  ```html
  <div [ngSwitch]="status">
    <p *ngSwitchCase="'ready'">Ready</p>
    <p *ngSwitchDefault>Unknown</p>
  </div>
  ```

### 3) Attribute Directives

Change the appearance or behavior of an element.

- Built-in: `[ngClass]`, `[ngStyle]`
- Custom: `@Directive({selector: '[appHighlight]'})`, manipulate `ElementRef`/`Renderer2`

---

### Cheat: Template Elements

| Element        | Description                                                                                                            |
| -------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `ng-template`  | Defines a fragment, not rendered by default.                                                                           |
| `ng-content`   | Projects parent content into a child component.                                                                        |
| `ng-container` | A logical wrapping element. Does NOT render any extra DOM node. Can host structural directives like `*ngIf`, `*ngFor`. |

---

## TOPIC: Angular Decorators/Annotations

### Types

| Type                    | Description                                                                                     | Example             |
| ----------------------- | ----------------------------------------------------------------------------------------------- | ------------------- |
| **Class decorator**     | Applied to classes. Commonly used to define Angular components, directives, modules, and pipes. | `@Component`        |
| **Method decorator**    | Applied to methods inside a class. Used for lifecycle hooks or custom decorators.               | `@HostListener`     |
| **Property decorator**  | Angular uses these as property (variable) of class, dependency injection and data binding.      | `@Input`, `@Output` |
| **Parameter decorator** | Applied to constructor parameters. Angular uses these for dependency injection.                 | `@Inject`           |

### Common Decorators

| Decorator          | Purpose                                                              |
| ------------------ | -------------------------------------------------------------------- |
| `@Component`       | Define component (selector, template, styles)                        |
| `@Directive`       | Define custom directive                                              |
| `@Pipe`            | Define transformation pipe                                           |
| `@NgModule`        | Define Angular module (declarations, imports)                        |
| `@Injectable`      | Make class injectable via DI                                         |
| `@Input`           | Receive data from parent                                             |
| `@Output`          | Emit events to parent                                                |
| `@ViewChild`       | Query single child in view (component's template)                    |
| `@ViewChildren`    | Query multiple children in view                                      |
| `@ContentChild`    | Query single projected content (via `<ng-content>`)                  |
| `@ContentChildren` | Query multiple projected contents                                    |
| `@HostListener`    | Listen to DOM events on the host element of a directive or component |
| `@Inject`          | Inject by token                                                      |
| `@Optional`        | Dependency can be null                                               |
| `@Self`            | Resolve in current injector only                                     |
| `@SkipSelf`        | Resolve in parent injector                                           |
| `@Host`            | Resolve from host element injector                                   |
| `@Attribute`       | Read attribute value from host                                       |

---

## TOPIC: Component Relationship — Input & Output

- `@Input()` : Parent → Child data binding
- `@Output()` : Child → Parent event communication via `EventEmitter`

**Example:**

```html
<!-- Parent template -->
<child [user]="selectedUser" (saved)="onSaved($event)"></child>
```

```typescript
// Child
@Input() user;
@Output() saved = new EventEmitter<string>();
```

---

## TOPIC: Template Reference Variables (#var)

- **Definition:** A local handle to a DOM element/component inside template.
- **Syntax:** `<input #username>` — usable in template: `{{username.value}}`
- **With @ViewChild:** `@ViewChild('username') inputRef: ElementRef;`

---

## TOPIC: Angular Pipes

**Pipe** — A Pipe in Angular is a function that takes in data and returns a transformed version of that data, typically used in the template with the `|` (pipe) operator.

- **Built-in:** `date`, `currency`, `percent`, `uppercase`, `lowercase`, `slice`, `json`, `async`
- **Usage:** `{{ total | currency:'INR':'symbol' }}`
- **Custom Pipe:**
  ```typescript
  @Pipe({name: 'capitalize'})
  transform(value: string): string { ... }
  ```
- **Command:** `ng g p pipe_name`

---

## TOPIC: Component Initialization — Constructor vs ngOnInit

|                        | Constructor            | ngOnInit                        |
| ---------------------- | ---------------------- | ------------------------------- |
| **When**               | Class instantiation    | After inputs are set            |
| **Use for**            | DI and simple defaults | Fetching data, complex setup    |
| **@Input values**      | NOT set yet            | ✅ Available                    |
| **Child view/content** | NOT ready              | NOT ready (use ngAfterViewInit) |

- Access `@ViewChild` in `ngAfterViewInit`.
- Access `@ContentChild` in `ngAfterContentInit`.

---

## TOPIC: Angular Lifecycle Hooks — Overview

**Component lifecycle:** Create → Render → Update → Destroy

**Definition:** They are predefined methods in Angular that get called at specific stages of a component's lifecycle. Lifecycle Hooks are special methods that allow you to tap into key moments in the life of a component or directive.

> **Note:** Hooks are listed in calling order.

| Hook                    | When Called                                               | Common Use                                                                                                                                  |
| ----------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `ngOnChanges`           | On every `@Input` change                                  | React to input changes — `ngOnChanges(changes: SimpleChanges): void { ... }` (has map-like object passed as parameter to access all inputs) |
| `ngOnInit`              | Once after inputs are set                                 | Fetching data from APIs, initializing variables                                                                                             |
| `ngDoCheck`             | Every change detection cycle                              | Custom change detection (invoked even if no property changed)                                                                               |
| `ngAfterContentInit`    | After `<ng-content>` is projected                         | Access `@ContentChild`                                                                                                                      |
| `ngAfterContentChecked` | After every check of projected content                    | Post-content check logic                                                                                                                    |
| `ngAfterViewInit`       | After component's view and child views are initialized    | Access `@ViewChild`, DOM manipulation                                                                                                       |
| `ngAfterViewChecked`    | After every check of the component's view and child views | Post-view check logic                                                                                                                       |
| `ngOnDestroy`           | Just before Angular destroys the component                | Cleanup (unsubscribe, detach listeners)                                                                                                     |

### Lifecycle Order

```
Constructor → ngOnChanges → ngOnInit → ngDoCheck →
ngAfterContentInit → ngAfterContentChecked →
ngAfterViewInit → ngAfterViewChecked → ngOnDestroy
```

### Best Practices

- Use `ngOnInit` for init and API calls
- Use `ngOnChanges` to react to `@Input` changes
- Use `ngAfterViewInit` for DOM/`@ViewChild` access
- Use `ngAfterContentInit` for `@ContentChild` access
- Use `ngOnDestroy` for cleanup (unsubscribe, remove listeners)

---

## Programmatic Rendering

| API                 | Description                                                                         |
| ------------------- | ----------------------------------------------------------------------------------- |
| `ngComponentOutlet` | Render a component dynamically in template.                                         |
| `ViewContainerRef`  | Create/insert component dynamically in code (`createComponent`, `insert`, `clear`). |

---

## TOPIC: Services

- A **Service** is a piece of reusable code with a focused purpose. Code that you will use in many components across your application.
- Services are easier to test. The task of getting data from the back-end server must be delegated to some other class. We call such a class a **Service class**.
- They are easier to debug.
- We can reuse the service at many places.

---

## TOPIC: Dependency Injection (DI)

A design pattern where a class receives its dependencies from an external source instead of creating them itself.

- **Loose coupling:** Components don't create services directly. (`this.productService = new ProductService();`)

### Core Concepts

| Concept        | Description                                     |
| -------------- | ----------------------------------------------- |
| **SERVICE**    | The Logic class (marked with `@Injectable`)     |
| **PROVIDER**   | The Map/Recipe (Tells Angular HOW to create it) |
| **INJECTOR**   | The Container (Stores and manages instances)    |
| **DEPENDENCY** | The Result (The actual object delivered to you) |

### Injector Levels

- **Root level** — `providedIn: 'root'`
- **Environment level** — `providers: [ChatRoomService]` in routes
- **Element level** — `providers: [ChatRoomService]` in `@Component` decorator

### Injection Types

- **Constructor Injection**
- **Property Injection** — `@Inject()`
- **Method Injection** — Dependencies are passed as method parameters.

### Hierarchical DI

Angular's DI system is hierarchical — child injectors inherit from parent injectors, and each level can override providers.

---

## TOPIC: Angular Routing and Navigation — Steps & Concepts

Routing in Angular allows navigation between different views or pages in a Single Page Application (SPA) without reloading the entire page.

### Steps

1. Define routes: `const routes = [{path: 'home', component: HomeComponent}];`
2. Add `<router-outlet>` in a host template
3. Navigate: `<a routerLink="/home">Home</a>` or `this.router.navigate(['home'])`

### Routing Terms

| Term               | Description                                                                                                                                                 |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `router`           | Angular Router service and module                                                                                                                           |
| `route`            | A single route config (path + component)                                                                                                                    |
| `routes`           | Array of route configs                                                                                                                                      |
| `redirectTo`       | Redirect one path to another (useful for default routes)                                                                                                    |
| `routerOutlet`     | Placeholder that renders matched route's component                                                                                                          |
| `routerLink`       | Template directive to navigate                                                                                                                              |
| `RouterLinkActive` | Adds CSS class when link is active                                                                                                                          |
| `ActivatedRoute`   | Access route params, query params, fragments                                                                                                                |
| `RouterState`      | Snapshot/tree of current router state                                                                                                                       |
| Route param var    | Read via `route.paramMap.get('id')`                                                                                                                         |
| Nested routes      | Let you render child components inside a parent component's view. The parent template holds a `<router-outlet>` where the matched child route is displayed. |
| Lazy loading       | Loading feature modules on demand instead of at app startup. Improves performance by reducing initial bundle size.                                          |
| `loadComponent`    | Lazy load a standalone component (Angular standalone APIs)                                                                                                  |
| `loadChildren`     | Lazy-load an entire feature module when a route is activated                                                                                                |
| Wildcard routes    | `{ path: '**', component: NotFoundComponent }`                                                                                                              |

### Route Guards

Route Guards in Angular are special interfaces that allow you to control navigation in your application.

| Guard           | Description                                                 |
| --------------- | ----------------------------------------------------------- |
| `CanActivate`   | Most common guard — determines if a user can enter a route. |
| `CanDeactivate` | Checks if a user is allowed to leave a route.               |
| `Resolve`       | Pre-fetch data before route is activated.                   |
| `CanLoad`       | Prevents lazy-loaded module from loading.                   |
| `CanMatch`      | Determines if a route can be matched.                       |

### Route Parameters

Dynamic values embedded in the URL path. Used for identifying resources like `id`, `slug`, etc.

```
products/:id
route.paramMap.get('id')
```

### Query Parameters

Key-value pairs appended to the URL after `?`. Used for filtering, sorting, pagination, etc.

```
/products?page=2&sort=desc
route.queryParamMap.get('page')
```

---

## TOPIC: Angular Forms

### Template-Driven Forms

- Quick setup; use `ngModel`; less boilerplate; good for simple forms.
- Module: `FormsModule`; Validation via HTML attributes and directives.

### Reactive Forms

- `FormGroup`, `FormControl`, `FormBuilder`; explicit, scalable, testable.
- Module: `ReactiveFormsModule`; Validation via `Validators`.
- The `FormGroup`, `FormControl` & `FormArray` are the three building blocks of Angular Forms.

```typescript
const form = this.fb.group({ name: ["", Validators.required] });
```

### Form Building Blocks

| Class         | Description                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------------- |
| `FormControl` | Represents the value and validation state of a single input field.                                  |
| `FormGroup`   | A collection of `FormControl`s (and/or nested `FormGroup`s/`FormArray`s), tracked as a single unit. |
| `FormBuilder` | A helper service to create `FormControl`s and `FormGroup`s more concisely (less boilerplate).       |

---

## TOPIC: RxJS in Angular — Observables

### Observables vs Promise

|             | Observable                | Promise      |
| ----------- | ------------------------- | ------------ |
| Values      | Multiple values over time | Single value |
| Cancellable | ✅ Yes                    | ❌ No        |
| Operators   | Rich (RxJS operators)     | Minimal      |

---

### Observable

**Observable** is a function that converts the ordinary data stream into an observable one. You can think of Observable as a wrapper around the ordinary data stream. Unlike a standard variable (which has one value) or a Promise (which gives you one value and then finishes), an Observable can emit multiple values over a period of time.

An Observable communicates using three types of signals:

- **Next:** "Here is a new piece of data." (Can happen 0 to infinite times).
- **Error:** "Something went wrong." (The stream stops).
- **Complete:** "I am finished, no more data is coming." (The stream stops).

### The Lifecycle

1. **Create:** Define the stream.
2. **Subscribe:** Start listening (Crucial! No sub = No data).
3. **Execute:** `Next()`, `Error()`, or `Complete()` signals sent.
4. **Unsubscribe:** Stop listening (to prevent memory leaks).

---

### `.pipe()` — The Assembly Line

A method used to chain **Operators** together to transform Observable data before it reaches the subscriber.

```
Observable -> .pipe( op1, op2, op3 ) -> .subscribe()
```

**Common Operators:**

| Operator     | Purpose                                           |
| ------------ | ------------------------------------------------- |
| `map`        | Transform data (e.g., lowercase a string).        |
| `filter`     | Select specific data (e.g., `x > 10`).            |
| `tap`        | Side effects (e.g., `console.log` for debugging). |
| `catchError` | Handle API failures gracefully.                   |
| `take(n)`    | Only grab the first `n` values then stop.         |

---

### `of()` vs `from()`

|             | `of()`                                               | `from()`                                            |
| ----------- | ---------------------------------------------------- | --------------------------------------------------- |
| **Name**    | THE WRAPPER                                          | THE EXTRACTOR                                       |
| **Syntax**  | `of(value1, value2, ...)`                            | `from(array \| promise \| iterable)`                |
| **Does**    | Emits the arguments exactly as they are.             | Breaks the input down and emits items individually. |
| **Example** | `of([1, 2, 3])` → emits `[1, 2, 3]` (the whole list) | `from([1, 2, 3])` → emits `1`, then `2`, then `3`   |
| **Pro Tip** | —                                                    | Use `from()` to convert a Promise to an Observable. |

---

### Observable APIs in Angular

| Source     | Usage                                              |
| ---------- | -------------------------------------------------- |
| HTTP       | `this.http.get<T>(url).pipe(...)`                  |
| Forms      | `formControl.valueChanges`                         |
| Router     | `route.paramMap`                                   |
| State      | `BehaviorSubject`/`ReplaySubject` for shared state |
| Async pipe | `{{ data$ \| async }}` auto-subscribe/unsubscribe  |

---

## TOPIC: Subject and BehaviorSubject

### What is a Subject?

A **Subject** is a special type of Observable that can multicast values to multiple subscribers.

- Does **NOT** store any previous value.
- When you subscribe, you only get **future values** (values emitted after you subscribe).
- If something was emitted before you subscribed, you won't get it.
- It is also an **Observer**, meaning you can call `.next()`, `.error()`, `.complete()` on it to emit values.

### What is a BehaviorSubject?

A **BehaviorSubject** is a type of Subject that stores the latest value and emits it immediately to new subscribers.

- Stores the **latest value only** (not multiple previous values).
- When you subscribe, you immediately get the **most recent value** (or the initial value if nothing was emitted yet).
- Does NOT keep a history of two or more previous values — just the latest one.

---

## Common Operators — Quick Guide

| Category           | Operators                                                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| **Creation**       | `of()`, `from()`, `interval()`, `timer()`                                                                                  |
| **Transformation** | `map`, `switchMap` (latest wins), `mergeMap` (concurrent), `concatMap` (in order), `exhaustMap` (ignore new while running) |
| **Filtering**      | `filter`, `take`, `takeUntil`, `first`, `last`, `debounceTime`, `distinctUntilChanged`                                     |
| **Combination**    | `combineLatest`, `forkJoin`, `zip`, `withLatestFrom`                                                                       |
| **Error Handling** | `catchError`, `retry`, `retryWhen`                                                                                         |
| **Utility**        | `tap`, `finalize`, `shareReplay`                                                                                           |

---

## TOPIC: State — Types & Approaches

### Types of State

| Type                 | Description                                          |
| -------------------- | ---------------------------------------------------- |
| **Local state**      | Component-only (inputs, internal variables)          |
| **Shared state**     | Across components (services + RxJS)                  |
| **Global state**     | App-wide (stores like NgRx/Akita/NGXS)               |
| **URL/router state** | Route params, query params, fragments                |
| **Server state**     | Data on backend (cached via services, `shareReplay`) |

### Approaches

- **Local (Inputs/Outputs, Signals)**
  - Signals: `signal<T>()`, `set()`, `update()`; reactive primitives for local state.
- **Shared (Services + RxJS)**
  - `BehaviorSubject` for current value; expose as Observable.
- **Global (Stores)**
  - NgRx (Redux-style), Akita, NGXS for structured global state.

---

### Signals

**Signals** are reactive states introduced in Angular 16 to manage state in a more predictable and efficient way. They allow you to store a value and automatically notify consumers when that value changes.

| API          | Description                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------- |
| `signal()`   | Creates a reactive value.                                                                                      |
| `computed()` | Creates a derived value based on other signals.                                                                |
| `effect()`   | Runs a side effect whenever any signal it depends on changes. Automatically re-runs when signal value changes. |

```typescript
import { signal } from "@angular/core";

const count = signal(0);
console.log(count()); // read value
count.set(5); // update value
count.update((v) => v + 1); // increment
```

---

## TOPIC: HttpClient

`HttpClient` is Angular's built-in HTTP API for making HTTP requests (`GET`/`POST`/`PUT`/`PATCH`/`DELETE`) and handling JSON by default. It's RxJS-based, so responses are Observables, enabling powerful composition.

---

## TOPIC: NgRx — Quick Overview

| Piece         | Description                                                                                                                                                 |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Store**     | Holds the app state as one immutable tree.                                                                                                                  |
| **Actions**   | Plain objects describing what happened (`type` + optional payload).                                                                                         |
| **Reducers**  | Pure functions that take the current state and an action to produce a new state. They never modify the old state (immutability).                            |
| **Selectors** | Functions used to slice and dice the store data for components. They are memoized (cached) for performance.                                                 |
| **Effects**   | Used for asynchronous tasks like API calls. They listen for an action, do some work (like an HTTP request), and then dispatch a new action with the result. |

- **Core pieces:** Actions (events), Reducers (state updates), Selectors (read state), Effects (handle side effects like HTTP).
- **Benefits:** Predictable state, time-travel debugging, scalability.

---

_END OF NOTES_
