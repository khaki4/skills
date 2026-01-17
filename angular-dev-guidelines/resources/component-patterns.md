# Component Patterns

Modern Angular 21 component architecture emphasizing zoneless change detection, standalone by default, signals, and proper lifecycle management.

## Table of Contents
- [Standalone Component Pattern (PREFERRED)](#standalone-component-pattern-preferred)
- [Signals for Reactive State](#signals-for-reactive-state)
- [Component Structure Template](#component-structure-template)
- [Lazy Loading Pattern](#lazy-loading-pattern)
- [Component Communication](#component-communication)
- [Change Detection Strategies](#change-detection-strategies)
- [Advanced Patterns](#advanced-patterns)
- [Summary](#summary)

---

## Standalone Component Pattern (DEFAULT)

### Why Standalone Components

Angular 21 uses standalone components by default:
- **`standalone: true` is now the default** - No need to specify it
- No NgModule boilerplate required
- Easier to understand and maintain
- Better tree-shaking and bundle size
- Simpler testing
- Zoneless change detection by default

### Basic Pattern

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-my-component',
  // standalone: true is default in Angular 21
  template: `
    <div class="container">
      <h2>{{ title }}</h2>
      <p>User ID: {{ userId }}</p>
    </div>
  `,
  styleUrls: ['./my-component.component.scss']
})
export class MyComponent {
  title = 'My Component';
  userId: number = 1;
}
```

**Key Points:**
- `standalone: true` is **default** - No need to specify
- Zoneless change detection by default - No Zone.js needed
- Typed properties
- Separate template and styles files for complex components

---

## Signals for Reactive State

### When to Use Signals

Use signals for:
- Local component state
- Reactive values that change over time
- Computed values derived from other signals
- Better change detection performance

### Signal Pattern

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  // standalone: true is default in Angular 21
  template: `
    <div class="card">
      <h3>{{ fullName() }}</h3>
      <p>Email: {{ email() }}</p>
      <button (click)="updateName()">Update Name</button>
    </div>
  `
})
export class UserProfileComponent {
  // Signals for reactive state
  firstName = signal('John');
  lastName = signal('Doe');
  email = signal('john.doe@example.com');

  // Computed signal
  fullName = computed(() => `${this.firstName()} ${this.lastName()}`);

  updateName(): void {
    this.firstName.set('Jane');
  }
}
```

**Key Points:**
- `signal()` for writable state
- `computed()` for derived state
- Call signal as function to read value: `firstName()`
- `.set()` to update signal value
- Automatic change detection

---

## Component Structure Template

### Recommended Order

```typescript
import { Component, OnInit, OnDestroy, inject, signal, computed } from '@angular/core';
import { Subject, takeUntil } from 'rxjs';

// Services
import { MyFeatureService } from '../services/my-feature.service';

// Models
import { MyData } from '../models/my-data.model';

// 1. COMPONENT DECORATOR
@Component({
  selector: 'app-my-component',
  // standalone: true is default in Angular 21
  // Zoneless change detection is default - no Zone.js needed
  templateUrl: './my-component.component.html',
  styleUrls: ['./my-component.component.scss']
})
export class MyComponent implements OnInit, OnDestroy {
  // 2. DEPENDENCY INJECTION (using inject function)
  private myService = inject(MyFeatureService);
  private router = inject(Router);

  // 3. SIGNALS (reactive state)
  data = signal<MyData[]>([]);
  selectedItem = signal<MyData | null>(null);
  isLoading = signal(false);
  errorMessage = signal<string | null>(null);

  // 4. COMPUTED SIGNALS
  filteredData = computed(() =>
    this.data().filter(item => item.active)
  );

  totalCount = computed(() => this.data().length);

  // 5. OBSERVABLES & SUBJECTS
  private destroy$ = new Subject<void>();

  // 6. REGULAR PROPERTIES
  title = 'My Component';
  mode: 'view' | 'edit' = 'view';

  // 7. LIFECYCLE HOOKS
  ngOnInit(): void {
    this.loadData();
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  // 8. PUBLIC METHODS
  loadData(): void {
    this.isLoading.set(true);
    this.errorMessage.set(null);

    this.myService.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (data) => {
          this.data.set(data);
          this.isLoading.set(false);
        },
        error: (error) => {
          this.errorMessage.set('Failed to load data');
          this.isLoading.set(false);
          console.error('Error loading data:', error);
        }
      });
  }

  selectItem(item: MyData): void {
    this.selectedItem.set(item);
  }

  // 9. PRIVATE HELPER METHODS
  private formatData(data: MyData[]): MyData[] {
    // Processing logic
    return data;
  }
}
```

---

## Lazy Loading Pattern

### When to Lazy Load

Lazy load features that are:
- Not needed on initial page load
- Route-based feature modules
- Heavy components with large dependencies
- Admin or rarely-used sections

### How to Lazy Load

**Route Configuration:**
```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./features/users/user-list.component')
      .then(m => m.UserListComponent)
  },
  {
    path: 'admin',
    loadChildren: () => import('./features/admin/admin.routes')
      .then(m => m.ADMIN_ROUTES)
  }
];
```

**Component Lazy Loading:**
```typescript
// Parent component template
<div class="container">
  @defer (on viewport) {
    <app-heavy-chart />
  } @placeholder {
    <div class="spinner-border"></div>
  } @loading {
    <p>Loading chart...</p>
  } @error {
    <p>Failed to load chart</p>
  }
</div>
```

**Angular 21 @defer Blocks:**
- `@defer` - Lazy load component
- `on viewport` - Load when visible
- `@placeholder` - Show while not loaded
- `@loading` - Show during loading
- `@error` - Show on error

---

## Component Communication

### Input/Output Pattern

```typescript
// Child component
import { Component, input, output } from '@angular/core';

@Component({
  selector: 'app-child',
  // standalone: true is default in Angular 21
  template: `
    <div>
      <p>{{ data() }}</p>
      <button (click)="handleClick()">Select</button>
    </div>
  `
})
export class ChildComponent {
  // Signals-based inputs
  data = input.required<string>();
  optionalData = input<string>('default');

  // Signal-based outputs
  selected = output<string>();

  handleClick(): void {
    this.selected.emit(this.data());
  }
}

// Parent component
@Component({
  selector: 'app-parent',
  // standalone: true is default in Angular 21
  imports: [ChildComponent],
  template: `
    <app-child
      [data]="myData"
      (selected)="onSelected($event)"
    />
  `
})
export class ParentComponent {
  myData = 'Hello from parent';

  onSelected(value: string): void {
    console.log('Selected:', value);
  }
}
```

---

## Change Detection Strategies

### Zoneless Change Detection (DEFAULT in Angular 21)

Angular 21 uses zoneless change detection by default. Zone.js is no longer included automatically.

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-optimized',
  // standalone: true is default
  // Zoneless change detection is automatic
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <button (click)="increment()">Increment</button>
    </div>
  `
})
export class OptimizedComponent {
  count = signal(0);

  increment(): void {
    this.count.update(n => n + 1);
  }
}
```

**Zoneless Benefits:**
- **No Zone.js overhead** - Better performance
- **Signals trigger change detection** automatically
- **Smaller bundle size** - No Zone.js dependency
- **Simpler mental model** - Just use signals

### If You Need Zone.js (Legacy)

```typescript
// app.config.ts - Only if you need Zone.js for legacy code
import { provideZoneChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }) // Opt-in to Zone.js
  ]
};
```

---

## Advanced Patterns

### Content Projection

```typescript
// Card component
@Component({
  selector: 'app-card',
  // standalone: true is default in Angular 21
  template: `
    <div class="card">
      <div class="card-header">
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="card-body">
        <ng-content></ng-content>
      </div>
      <div class="card-footer">
        <ng-content select="[footer]"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent {}

// Usage
<app-card>
  <h3 header>Card Title</h3>
  <p>Card content goes here</p>
  <button footer>Action</button>
</app-card>
```

### Template References

```typescript
@Component({
  selector: 'app-template-example',
  // standalone: true is default in Angular 21
  template: `
    @if (data$ | async; as data) {
      {{ data }}
    } @else {
      <div class="spinner-border"></div>
    }
  `
})
export class TemplateExampleComponent {
  data$ = this.service.getData();
}
```

---

## Summary

**Modern Angular 21 Component Recipe:**
1. Components are standalone by default (no `standalone: true` needed)
2. Zoneless change detection by default (no Zone.js needed)
3. Signals for reactive state management
4. Signal-based inputs with `input()` and outputs with `output()`
5. Inject function for dependency injection
6. Proper lifecycle hooks (OnInit, OnDestroy)
7. TakeUntil pattern for unsubscribing observables
8. TypeScript strict typing
9. Use Vitest for testing (default in Angular 21)

**See Also:**
- [data-fetching.md](data-fetching.md) - Service and HTTP patterns
- [loading-and-error-states.md](loading-and-error-states.md) - Error handling
- [common-patterns.md](common-patterns.md) - Signal Forms and more
