---
name: angular-dev-guidelines
description: Frontend development guidelines for Angular 21/TypeScript applications. Modern patterns including zoneless change detection, standalone components, signals, Signal Forms, NgRx SignalStore, file organization with features directory, Bootstrap/Tailwind/Material styling, performance optimization, and TypeScript best practices. Use when creating components, services, features, fetching data, styling, routing, or working with frontend code.
globs:
  - "**/angular.json"
  - "**/*.component.ts"
  - "**/*.component.html"
  - "**/*.service.ts"
  - "**/*.module.ts"
  - "**/*.directive.ts"
  - "**/*.pipe.ts"
  - "**/*.guard.ts"
  - "**/*.resolver.ts"
  - "**/*.interceptor.ts"
  - "**/app.routes.ts"
  - "**/app.config.ts"
  - "**/*.store.ts"
  - "**/package.json"
---

# Angular 21 Development Guidelines (2025 Edition)

> **⚠️ MANDATORY: All patterns and guidelines in this skill MUST be followed strictly. These are not suggestions - they are required standards.**
>
> **📅 Updated**: January 2025 - Based on Rainer Hahnekamp's Angular 2025 recommendations

## Purpose

Comprehensive guide for modern Angular 21 development, emphasizing:
- **Signals-first approach** for reactivity
- **Zoneless change detection** by default
- **Standalone components** (default in Angular 21)
- **NgRx SignalStore** for state management
- **OnPush change detection** for performance
- Proper file organization and TypeScript best practices

## When to Use This Skill

Automatically activates when working on:
- Creating new components or pages
- Building new features
- Implementing services and state management
- Setting up routing with Angular Router
- Styling components with Bootstrap
- Performance optimization
- Organizing frontend code
- TypeScript best practices
- RxJS patterns and operators

## ⚠️ CRITICAL: How to Use This Skill Properly

When this skill activates:
1. **Read this main file first** for overview and core principles
2. **Check the Navigation Guide table below** - it tells you which resource file to read based on your task
3. **ALWAYS read the recommended resource file** - it contains detailed patterns and examples
4. **Apply the patterns from BOTH files** - main file + resource file

**Example:**
- Task: "Create a user list component that fetches data from API"
- Step 1: ✅ Read this main skill.md (you're doing this)
- Step 2: ✅ Check Navigation Guide → "Component with data fetching" → component-patterns.md + data-fetching.md
- Step 3: ✅ **READ both resource files** before writing code
- Step 4: ✅ Apply component patterns and data fetching patterns from those resources

---

## Project Setup (2025 Recommendations)

### Project Tools: Angular CLI vs Nx

| Situation | Recommendation |
|-----------|----------------|
| New application | **Angular CLI** (simpler, well-integrated) |
| After project scales | Migrate with `nx import` |
| Library development | Start with Nx from beginning |

**Reasoning**: Nx adds abstraction layer and learning curve. Angular CLI integrates well with Schematics. Easy to migrate to Nx later when needed.

### Package Manager

**Recommended**: `pnpm` for better disk space efficiency and faster installs.

### Quick Project Creation

```bash
# Create new Angular 21 project with recommended settings
pnpm create @angular@latest -s -t -S --experimental-zoneless --ssr false --style scss [projectName]

# Add UI library (choose one)
pnpm ng add @angular/material  # Material Design
# OR use Bootstrap/Tailwind based on project needs

# Code quality tools
pnpm ng add @angular-eslint/schematics
pnpm i -D eslint-plugin-unused-imports husky prettier lint-staged @softarc/{sheriff-core,eslint-plugin-sheriff}

# Testing setup
pnpm i -D @testing-library/angular @testing-library/dom @testing-library/user-event
pnpm i -D @playwright/test && pnpm playwright install
```

---

## Quick Start

### New Component Checklist

Creating a component? Follow this checklist:

- [ ] Components are standalone by default (no need to specify `standalone: true`)
- [ ] **Use `ChangeDetectionStrategy.OnPush`** (required for local change detection)
- [ ] Define TypeScript interface for component inputs
- [ ] **Use signals for reactive state** (Signals-first approach)
- [ ] Zoneless change detection is default (no Zone.js needed)
- [ ] Use `inject()` function for dependency injection
- [ ] Lazy load feature modules when appropriate
- [ ] Use Bootstrap/Tailwind/Material classes for styling
- [ ] Use async pipe or signals for observables in templates
- [ ] Follow Angular style guide naming conventions
- [ ] Consider Signal Forms for form handling (experimental)
- [ ] Keep components small (~100 lines), use Container + Presentational pattern

### New Feature Checklist

Creating a feature? Set up this structure:

- [ ] Create `src/app/features/{feature-name}/` directory
- [ ] Create subdirectories: `services/`, `components/`, `models/`, `guards/`
- [ ] Create feature service: `services/{feature}.service.ts`
- [ ] Set up TypeScript interfaces in `models/`
- [ ] Create routes file if feature has routes (`{feature}.routes.ts`)
- [ ] Components are standalone by default
- [ ] Use **NgRx SignalStore** for complex state management
- [ ] Write **feature-based tests** (not file-based) with Angular Testing Library
- [ ] Use **Playwright** for E2E testing
- [ ] Implement proper error handling
- [ ] Export public API from feature

---

## Common Imports Cheatsheet

```typescript
// Angular Core (with OnPush - REQUIRED)
import {
  Component, OnInit, OnDestroy, inject, signal, computed, input, output,
  ChangeDetectionStrategy
} from '@angular/core';

// Angular Router
import { Router, ActivatedRoute } from '@angular/router';

// Forms - Traditional
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

// Signal Forms (Experimental - Angular 21)
import { SignalFormGroup, SignalFormControl } from '@angular/forms/signals';

// NgRx SignalStore (Recommended for state management)
import { signalStore, withState, withComputed, withMethods, patchState } from '@ngrx/signals';
import { withEntities, setEntities, addEntity, updateEntity } from '@ngrx/signals/entities';

// RxJS (use when Signals don't provide clear benefit)
import { Observable, Subject, BehaviorSubject } from 'rxjs';
import { takeUntil, map, filter, switchMap, tap } from 'rxjs/operators';

// HTTP Client
import { HttpClient, HttpErrorResponse, HttpResponse } from '@angular/common/http';

// Resource API (Experimental - Angular 21)
import { resource, rxResource } from '@angular/core';

// Services
import { MyFeatureService } from './services/my-feature.service';

// Models
import { User, UserRole } from './models/user.model';
```

---

## Navigation Guide (⚠️ MANDATORY - Check This Table and Read Relevant Resources)

Use this table to determine which resource file you MUST read based on your task:

| If You Need To... | Then MUST Read... | When to Read It |
|-------------------|-------------------|-----------------|
| **Create components, pages, lifecycle hooks** | **[component-patterns.md](resources/component-patterns.md)** | **When creating ANY component** |
| **Fetch data from API, HTTP calls, state management** | **[data-fetching.md](resources/data-fetching.md)** | **⚡ CRITICAL: When making HTTP requests** |
| **Complex state, NgRx, Effects, enterprise patterns** | **[ngrx-patterns.md](resources/ngrx-patterns.md)** | **When app needs scale or DevTools tracing** |
| Organize project structure, features, folders | [file-organization.md](resources/file-organization.md) | Before starting new features |
| Style components, use Bootstrap utilities | [styling-guide.md](resources/styling-guide.md) | When implementing UI/styling |
| Set up routing, guards, lazy loading | [routing-guide.md](resources/routing-guide.md) | When configuring routes |
| Handle loading states, errors, async data | [loading-and-error-states.md](resources/loading-and-error-states.md) | When implementing data loading |
| Optimize performance, OnPush, trackBy | [performance.md](resources/performance.md) | When optimizing components |
| TypeScript types, interfaces, strict mode | [typescript-standards.md](resources/typescript-standards.md) | When defining models/types |
| Forms, interceptors, guards, pipes | [common-patterns.md](resources/common-patterns.md) | When implementing common features |

**⚡ Most Common Mistakes:**
1. Creating components without reading component-patterns.md (missing OnPush, proper lifecycle, signals)
2. Making HTTP calls without reading data-fetching.md (missing error handling, memory leaks from unsubscribed observables)
3. Not using async pipe or proper cleanup (memory leaks)
4. ⭐ **For styling questions (badges, status indicators, UI patterns), ALWAYS check common-patterns.md FIRST** before searching codebase - ensures consistency across all pages

---

## Topic Guides (Quick Reference - Always Check Navigation Guide Above)

### 🎨 Component Patterns
Modern Angular 21 standalone components with signals, OnPush (required), and proper lifecycle.
**[📖 Complete Guide: resources/component-patterns.md](resources/component-patterns.md)**

### 📊 Data Fetching & State Management
Services, HttpClient, Signals-first approach, `resource`/`rxResource`, error handling.
**[📖 Complete Guide: resources/data-fetching.md](resources/data-fetching.md)**

### 🏗️ NgRx SignalStore & Enterprise Patterns
**NgRx SignalStore (recommended)**, Effects, chain flows, RxJS operator selection.
**[📖 Complete Guide: resources/ngrx-patterns.md](resources/ngrx-patterns.md)**

### 📁 File Organization
Feature-based structure, services/, components/, models/, guards/ organization.
**[📖 Complete Guide: resources/file-organization.md](resources/file-organization.md)**

### 🎨 Styling with Bootstrap
Bootstrap 5 utilities, component styles, scoped styling, ViewEncapsulation.
**[📖 Complete Guide: resources/styling-guide.md](resources/styling-guide.md)**

### 🛣️ Routing
Standalone routes, lazy loading, guards, resolvers, route configuration.
**[📖 Complete Guide: resources/routing-guide.md](resources/routing-guide.md)**

### ⏳ Loading & Error States
Async pipe, loading indicators, error handling, user feedback patterns.
**[📖 Complete Guide: resources/loading-and-error-states.md](resources/loading-and-error-states.md)**

### ⚡ Performance
OnPush strategy, lazy loading, trackBy, pure pipes, signals, unsubscribe patterns.
**[📖 Complete Guide: resources/performance.md](resources/performance.md)**

### 📘 TypeScript Standards
Strict mode, explicit types, interfaces, no `any`, null handling.
**[📖 Complete Guide: resources/typescript-standards.md](resources/typescript-standards.md)**

### 🔧 Common Patterns
Reactive forms, interceptors, guards, directives, pipes, RxJS operators.
**[📖 Complete Guide: resources/common-patterns.md](resources/common-patterns.md)**

---

## Core Principles

### Framework Defaults (Angular 21 / 2025)
1. **Zoneless by Default**: Angular 21 uses zoneless change detection (no Zone.js needed)
2. **Standalone by Default**: Components are standalone by default, no `standalone: true` needed
3. **Signals First**: Start with Signals → Use RxJS only when it provides clear benefit
4. **OnPush Required**: Always use `ChangeDetectionStrategy.OnPush` for local change detection
5. **inject() Function**: Prefer `inject()` over constructor injection

### Reactivity Principle (2025)
```
Signals로 시작 → RxJS가 명확한 이점을 제공하는 경우에만 RxJS 사용
```
- Signal Forms, `httpResource` are still in development
- `effect` is in developer preview
- **Adopt Signals now** - migrating from RxJS to Signals later costs more than starting with Signals

### Architecture Principles (Why We Do Things This Way)
6. **Explicit State Changes**: State must change through explicit, traceable actions - never implicitly through side effects
7. **Unidirectional Data Flow**: Data flows one way: Action → State → View. No two-way binding except for forms
8. **Component-Logic Separation**: Components handle only dispatch and select. Business logic belongs in services/effects
9. **Traceable Flows**: All state changes must be visible in DevTools. If you can't trace it, it's wrong

### Implementation Standards
10. **NgRx SignalStore**: Use for both local and global state management
11. **Features are Organized**: services/, components/, models/ subdirs
12. **Styling Options**: Bootstrap, Tailwind CSS, or Angular Material
13. **Lazy Load Features**: Improve initial load performance
14. **Type Safety**: Strict TypeScript, no `any` types

---

## Prohibited Patterns (What NOT To Do)

> **⚠️ These patterns are FORBIDDEN. Violating these rules creates untraceable bugs and maintenance nightmares.**

### ❌ Two-Way Binding (Except Forms)

Two-way binding hides state changes and makes debugging impossible.

```typescript
// ❌ FORBIDDEN - State changes are invisible
<input [(ngModel)]="user.name">

// ✅ ALLOWED - Reactive Forms (explicit state)
<input [formControl]="form.controls.name">

// ✅ ALLOWED - Explicit event handling
<input [value]="name()" (input)="onNameChange($event)">
```

### ❌ Flow Logic in Components

Components should NOT contain business logic chains. Use Effects instead.

```typescript
// ❌ FORBIDDEN - useEffect-style implicit flows
ngOnInit() {
  this.user$.subscribe(user => {
    if (user.status === 'created') {
      this.loadOrders();      // Hidden side effect
      this.sendNotification(); // Another hidden side effect
    }
  });
}

// ✅ ALLOWED - Explicit Effects (for complex apps)
@Injectable()
export class UserEffects {
  userCreatedFlow$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.createSuccess),
      switchMap(({ user }) => [
        OrderActions.load({ userId: user.id }),
        NotificationActions.send({ type: 'welcome' })
      ])
    )
  );
}

// ✅ ALLOWED - Simple apps can use service methods (see data-fetching.md)
```

### ❌ Direct State Mutation from Components

State changes must go through proper channels (Store dispatch or service methods).

```typescript
// ❌ FORBIDDEN - Direct mutation
this.userService.user.set(newUser);
this.users.push(newUser);
this.data = newData;

// ✅ ALLOWED - Store dispatch (complex apps)
this.store.dispatch(UserActions.update({ user: newUser }));

// ✅ ALLOWED - Service method (simple apps)
this.userService.addUser(newUser); // Service encapsulates the mutation
```

### When to Use Simple vs. Complex Patterns

| App Complexity | State Management | Flow Management |
|----------------|------------------|-----------------|
| Simple CRUD | Signals in services | Service methods |
| Medium complexity | **NgRx SignalStore** | Service methods with clear naming |
| Complex/Enterprise | NgRx SignalStore (or Global Store for eventing) | NgRx Effects (rare cases only) |

**Key Insight**:
- **NgRx SignalStore is the recommended default** for both local and global state
- NgRx Global Store (Redux pattern) is only for rare cases needing event-based systems
- Wait for SignalStore's eventing extension before considering Global Store
- See [ngrx-patterns.md](resources/ngrx-patterns.md) for when and why to scale up

---

## Quick Reference: File Structure

```
src/app/
  features/
    users/
      services/
        user.service.ts
      components/
        user-header.component.ts
      pages/
        user-list.component.ts
        user-detail.component.ts
      models/
        user.model.ts
      guards/
        user.guard.ts
      users.routes.ts

  shared/
    components/
      loading-spinner/
      error-message/
    directives/
      highlight.directive.ts
    pipes/
      custom-date.pipe.ts

  core/
    services/
      auth.service.ts
      api.service.ts
    interceptors/
      auth.interceptor.ts
      error.interceptor.ts
```

---

## Modern Component Template (Quick Copy)

```typescript
import { Component, OnInit, OnDestroy, inject, signal, ChangeDetectionStrategy } from '@angular/core';
import { Subject, takeUntil } from 'rxjs';
import { MyFeatureService } from '../services/my-feature.service';
import { MyData } from '../models/my-data.model';

@Component({
  selector: 'app-my-component',
  // standalone: true is default in Angular 21
  changeDetection: ChangeDetectionStrategy.OnPush,  // REQUIRED for local CD
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.scss']
})
export class MyComponent implements OnInit, OnDestroy {
  private myService = inject(MyFeatureService);  // inject() preferred over constructor
  private destroy$ = new Subject<void>();

  // Signals for reactive state (Signals-first approach)
  data = signal<MyData[]>([]);
  isLoading = signal(false);
  error = signal<string | null>(null);

  ngOnInit(): void {
    this.loadData();
  }

  loadData(): void {
    this.isLoading.set(true);
    this.myService.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (data) => {
          this.data.set(data);
          this.isLoading.set(false);
        },
        error: (err) => {
          this.error.set(err.message);
          this.isLoading.set(false);
        }
      });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### NgRx SignalStore Example (Recommended for State Management)

```typescript
import { signalStore, withState, withComputed, withMethods, patchState } from '@ngrx/signals';
import { computed } from '@angular/core';

interface UserState {
  users: User[];
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  isLoading: false,
  error: null
};

export const UserStore = signalStore(
  withState(initialState),
  withComputed((state) => ({
    activeUsers: computed(() => state.users().filter(u => u.isActive)),
    userCount: computed(() => state.users().length)
  })),
  withMethods((store, userService = inject(UserService)) => ({
    async loadUsers() {
      patchState(store, { isLoading: true, error: null });
      try {
        const users = await userService.getUsers();
        patchState(store, { users, isLoading: false });
      } catch (error) {
        patchState(store, { error: error.message, isLoading: false });
      }
    },
    addUser(user: User) {
      patchState(store, { users: [...store.users(), user] });
    }
  }))
);
```

---

## Anti-Patterns to Avoid

❌ Adding Zone.js when not needed (zoneless is default in Angular 21)
❌ Still using `standalone: true` explicitly (it's the default now)
❌ **Missing `ChangeDetectionStrategy.OnPush`** (required for local change detection!)
❌ Not unsubscribing from observables (memory leaks!)
❌ Using `.subscribe()` in templates (use async pipe instead)
❌ Not handling error states in HTTP calls
❌ Using `any` type instead of proper interfaces
❌ Missing loading indicators for async operations
❌ Not using `track` expression in @for loops
❌ Forgetting to implement ngOnDestroy for cleanup
❌ Subscribing multiple times to the same observable
❌ **Starting with RxJS when Signals would suffice** (Signals-first approach!)
❌ Putting business logic in components instead of services
❌ **Using constructor DI instead of `inject()` function**
❌ Creating file-based tests instead of feature-based tests

---

## Testing (2025 Recommendations)

### Unit/Integration Testing

| Tool | Purpose |
|------|---------|
| **Jasmine + Karma** | Default Angular test framework |
| **Angular Testing Library** | Recommended - user-facing selectors, simplified setup |

**Angular Testing Library Benefits:**
- User-facing selectors (role, label-based) instead of implementation details
- `render` function simplifies TestingModule setup
- Automatic handling of async operations and change detection

```typescript
// Angular Testing Library example
import { render, screen } from '@testing-library/angular';
import { UserComponent } from './user.component';

it('should display user name', async () => {
  await render(UserComponent, {
    componentInputs: { user: { name: 'John' } }
  });
  expect(screen.getByText('John')).toBeInTheDocument();
});
```

### E2E Testing

**Recommended:** Playwright

```bash
pnpm i -D @playwright/test
pnpm playwright install
```

### Testing Philosophy

```
❌ File-based tests (one test file per source file)
✅ Feature-based tests (test features/behaviors, not files)
```

- Disable CLI auto-generation of test files in `angular.json`
- Focus on testing user-facing behavior, not implementation

---

## Code Quality Tools

| Tool | Purpose |
|------|---------|
| **ESLint** (angular-eslint) | Linting + Angular style guide |
| **eslint-plugin-unused-imports** | Auto-remove unused imports |
| **Prettier** | Code formatting |
| **lint-staged** | Run Prettier on commit |
| **Husky** | Git hook management |
| **Sheriff** | Architecture rules (module boundaries, dependencies) |

**Prettier Best Practice:**
- ❌ Don't use as ESLint plugin (formatting errors appear as lint errors)
- ✅ Use lint-staged to run before commits

```json
// package.json
{
  "lint-staged": {
    "*.{ts,html,scss}": ["prettier --write"]
  }
}
```

---

## Schematics Customization

Add to `angular.json` for consistent code generation:

```json
"schematics": {
  "@schematics/angular:component": {
    "inlineTemplate": true,
    "inlineStyle": true,
    "style": "scss",
    "skipTests": true,
    "flat": true,
    "changeDetection": "OnPush"
  },
  "@schematics/angular:class": { "skipTests": true },
  "@schematics/angular:directive": { "skipTests": true },
  "@schematics/angular:guard": { "skipTests": true },
  "@schematics/angular:interceptor": { "skipTests": true },
  "@schematics/angular:pipe": { "skipTests": true },
  "@schematics/angular:resolver": { "skipTests": true },
  "@schematics/angular:service": { "skipTests": true }
}
```

**Design Philosophy:**
- Keep components small (~100 lines)
- Container + Presentational component separation
- Move logic to services
- Prefer inline template/styles for small components

---

## SSR, Hydration & Incremental Hydration

**Use SSR only when:**
- Initial loading performance is critical for internet-facing apps
- SEO is important

**Caution:**
- Increases development/deployment complexity
- **Only use when truly necessary**

```bash
# Enable SSR during project creation
pnpm create @angular@latest --ssr true [projectName]
```

---

## 2025 Emerging Technologies

| Technology | Status | Description |
|------------|--------|-------------|
| **httpResource** | Experimental | New Angular resource API for HTTP |
| **Signal Forms** | Experimental | Form handling with Signals |
| **resource / rxResource** | Experimental | Data fetching primitives |
| **Angular Query** | Stable | TanStack Query for Angular - server state & caching |
| **Analog** | Growing | Angular full-stack meta-framework |

**Recommendation:** Adopt `resource`/`rxResource` in new projects despite experimental status.

---

## Related Skills

- **dotnet-backend-guidelines**: .NET Core backend API patterns that Angular consumes

---

## References

- **Rainer Hahnekamp's Angular 2025 Setup**: https://dev.to/this-is-angular/my-favorite-angular-setup-in-2025-3mbo
- **Starter Repo**: https://github.com/rainerhahnekamp/angular-starter
- **Video**: https://www.youtube.com/watch?v=lbiOP-VLKGI

---

**Skill Status**: Angular 21 + 2025 Best Practices (Updated January 2025)
