# Common Patterns (Angular 21)

## Table of Contents
- [API Request Handling with ApiHandler](#api-request-handling-with-apihandler)
- [Paginated Requests with PaginationState](#paginated-requests-with-paginationstate)
- [Signal Forms (Angular 21 Experimental)](#signal-forms-angular-21-experimental)
- [Reactive Forms with Validation](#reactive-forms-with-validation)
- [HTTP Interceptors](#http-interceptors)
- [Route Guards](#route-guards)
- [Custom Pipes](#custom-pipes)
- [RxJS Patterns](#rxjs-patterns)
- [UI Component Patterns](#ui-component-patterns)
- [Best Practices](#best-practices)

---

## API Request Handling with ApiHandler

```typescript
// Using ApiHandler for standard requests
export class UserService {
  private http = inject(HttpClient);

  getUser(id: number, isLoading: WritableSignal<boolean>) {
    const request$ = this.http.get<APIBaseResponse<User>>(`/api/users/${id}`);

    return ApiHandler.handle(
      request$,
      (loading) => isLoading.set(loading),
      (data) => console.log('User loaded:', data),
      (errorMessage, errorModel) => console.error(errorMessage, errorModel)
    );
  }
}

// Component usage
export class UserDetailComponent {
  private userService = inject(UserService);
  user = signal<User | null>(null);
  isLoading = signal(false);
  error = signal<string | null>(null);

  loadUser(id: number) {
    this.userService.getUser(id, this.isLoading).subscribe({
      next: (response) => this.user.set(response.data),
      error: (err) => this.error.set(err.message)
    });
  }
}
```

## Paginated Requests with PaginationState

```typescript
export class UserListComponent {
  private userService = inject(UserService);

  users = signal<User[]>([]);
  isLoading = signal(false);
  error = signal<string | null>(null);
  paginationState = new PaginationState(10); // 10 items per page

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    const request$ = this.http.get<APIPaginatedResponse<User>>('/api/users', {
      params: {
        pageNumber: this.paginationState.currentPage,
        pageSize: this.paginationState.pageSize
      }
    });

    ApiHandler.handlePaginatedWithState(
      request$,
      this.paginationState,
      (items) => this.users.set(items),
      (loading) => this.isLoading.set(loading),
      (errorMessage) => this.error.set(errorMessage)
    ).subscribe();
  }

  nextPage() {
    if (this.paginationState.goToNextPage()) {
      this.loadUsers();
    }
  }

  previousPage() {
    if (this.paginationState.goToPreviousPage()) {
      this.loadUsers();
    }
  }

  changePageSize(newSize: number) {
    this.paginationState.updatePageSize(newSize);
    this.loadUsers();
  }
}
```

**Template:**
```html
<div class="pagination-controls">
  <button (click)="previousPage()" [disabled]="!paginationState.hasPreviousPage">
    Previous
  </button>
  <span>Page {{ paginationState.currentPage }} of {{ paginationState.totalPages }}</span>
  <button (click)="nextPage()" [disabled]="!paginationState.hasNextPage">
    Next
  </button>
</div>
```

## Signal Forms (Angular 21 Experimental)

Angular 21 introduces Signal Forms - a new signal-based approach to form handling.

### Basic Signal Form

```typescript
import { Component, signal } from '@angular/core';
import { SignalFormGroup, SignalFormControl, Validators } from '@angular/forms/signals';

@Component({
  selector: 'app-user-form',
  // standalone: true is default in Angular 21
  template: `
    <form (ngSubmit)="onSubmit()">
      <div class="mb-3">
        <label class="form-label">Name</label>
        <input [formField]="form.controls.name" class="form-control" />
        @if (form.controls.name.invalid() && form.controls.name.touched()) {
          <small class="text-danger">Name is required (min 3 chars)</small>
        }
      </div>

      <div class="mb-3">
        <label class="form-label">Email</label>
        <input [formField]="form.controls.email" class="form-control" type="email" />
        @if (form.controls.email.invalid() && form.controls.email.touched()) {
          <small class="text-danger">Valid email is required</small>
        }
      </div>

      <button type="submit" class="btn btn-primary" [disabled]="form.invalid()">
        Submit
      </button>
    </form>
  `
})
export class UserFormComponent {
  // Signal-based form
  form = new SignalFormGroup({
    name: new SignalFormControl('', [Validators.required, Validators.minLength(3)]),
    email: new SignalFormControl('', [Validators.required, Validators.email])
  });

  onSubmit() {
    if (form.valid()) {
      const formData = this.form.value(); // Returns signal value
      console.log('Form submitted:', formData);
    }
  }
}
```

### Signal Forms Benefits

- **Better type safety** - Full TypeScript inference
- **Reactive by nature** - Uses signals for all state
- **Simpler validation** - Validation state as signals
- **No Zone.js needed** - Works with zoneless detection
- **Lightweight** - Similar to template-driven forms but powerful like reactive forms

### Signal Forms vs Reactive Forms

| Feature | Signal Forms | Reactive Forms |
|---------|-------------|----------------|
| State management | Signals | Observables |
| Type safety | Better | Good |
| Zone.js dependency | No | Yes (traditional) |
| Bundle size | Smaller | Larger |
| Validation API | Similar | Similar |

**Note:** Signal Forms is experimental in Angular 21. For production apps, you can still use traditional Reactive Forms.

## Reactive Forms with Validation

```typescript
import { FormBuilder, Validators, ReactiveFormsModule } from '@angular/forms';

export class UserFormComponent {
  private fb = inject(FormBuilder);

  userForm = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]]
  });

  onSubmit() {
    if (this.userForm.valid) {
      const user = this.userForm.value;
      this.userService.createUser(user).subscribe();
    }
  }
}
```

**Template:**
```html
<form [formGroup]="userForm" (ngSubmit)="onSubmit()">
  <input formControlName="name" class="form-control">
  @if (userForm.get('name')?.invalid && userForm.get('name')?.touched) {
    <small class="text-danger">Name is required (min 3 chars)</small>
  }
  <button type="submit" [disabled]="userForm.invalid">Submit</button>
</form>
```

## HTTP Interceptors

```typescript
// auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authToken = localStorage.getItem('token');

  if (authToken) {
    const authReq = req.clone({
      setHeaders: { Authorization: `Bearer ${authToken}` }
    });
    return next(authReq);
  }

  return next(req);
};
```

## Route Guards

```typescript
// auth.guard.ts
export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  router.navigate(['/login']);
  return false;
};
```

## Custom Pipes

```typescript
@Pipe({ name: 'timeAgo', standalone: true })
export class TimeAgoPipe implements PipeTransform {
  transform(value: Date | string): string {
    const seconds = Math.floor((new Date().getTime() - new Date(value).getTime()) / 1000);
    if (seconds < 60) return 'just now';
    if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
    return `${Math.floor(seconds / 86400)}d ago`;
  }
}

// Usage: {{ post.createdAt | timeAgo }}
```

## RxJS Patterns

```typescript
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';

// Search with debounce
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.searchService.search(term))
).subscribe(results => this.results.set(results));
```

## UI Component Patterns

### Badge Styling Standards

Use subtle badge styles for softer, modern appearance:

```html
<!-- Active/Success state (green) -->
<span class="badge bg-success-subtle text-success">
  {{ 'LABEL.ACTIVE' | translate }}
</span>

<!-- Inactive/Error state (red) -->
<span class="badge bg-danger-subtle text-danger">
  {{ 'LABEL.INACTIVE' | translate }}
</span>

<!-- Yes/No badges with conditional classes -->
<span class="badge"
      [class.bg-success-subtle]="value"
      [class.text-success]="value"
      [class.bg-danger-subtle]="!value"
      [class.text-danger]="!value">
  {{ value ? ('LABEL.YES' | translate) : ('LABEL.NO' | translate) }}
</span>
```

**Why subtle badges?**
- Softer, more modern appearance
- Better visual hierarchy
- Reduces visual noise
- Light background with colored text

### Activate/Deactivate Actions Standards

Use consistent icons and colors for activation controls:

```html
<!-- Deactivate option (shown when active) -->
<a class="dropdown-item" href="javascript:void(0);" (click)="deactivate()">
  <i class="ri-pause-circle-fill align-bottom me-2 text-warning"></i>
  {{ 'LABEL.DEACTIVATE' | translate }}
</a>

<!-- Activate option (shown when inactive) -->
<a class="dropdown-item" href="javascript:void(0);" (click)="activate()">
  <i class="ri-play-circle-fill align-bottom me-2 text-success"></i>
  {{ 'LABEL.ACTIVATE' | translate }}
</a>
```

**Standard conventions:**
- **Deactivate**: `ri-pause-circle-fill` icon with `text-warning` (orange/yellow)
- **Activate**: `ri-play-circle-fill` icon with `text-success` (green)
- Show deactivate when item is active, show activate when inactive
- Icons are semantically meaningful (pause/play for state changes)

### Dropdown Menu Standards

Use NgBootstrap dropdown with `container="body"` to prevent overflow issues:

```html
<div class="dropdown" ngbDropdown container="body">
  <button class="btn btn-soft-secondary btn-sm dropdown-toggle-no-caret"
          type="button"
          ngbDropdownToggle>
    <i class="ri-more-fill align-middle"></i>
  </button>
  <ul class="dropdown-menu dropdown-menu-end" ngbDropdownMenu>
    <li>
      <a class="dropdown-item" href="javascript:void(0);" (click)="edit()">
        <i class="ri-pencil-fill align-bottom me-2 text-muted"></i>
        {{ 'BUTTON.EDIT' | translate }}
      </a>
    </li>
    <li>
      <a class="dropdown-item text-danger" href="javascript:void(0);" (click)="delete()">
        <i class="ri-delete-bin-fill align-bottom me-2"></i>
        {{ 'BUTTON.DELETE' | translate }}
      </a>
    </li>
  </ul>
</div>
```

**Component SCSS:**
```scss
// Hide dropdown arrow (only show dots icon)
.dropdown-toggle-no-caret::after {
  display: none;
}
```

**Why this pattern?**
- `container="body"` prevents dropdown from being cut off by table overflow
- `dropdown-toggle-no-caret` hides the default arrow, showing only the dots icon
- `dropdown-menu-end` aligns dropdown to the right
- Works perfectly in scrollable tables with few records

**Component imports needed:**
```typescript
import { NgbDropdownModule } from '@ng-bootstrap/ng-bootstrap';

@Component({
  imports: [CommonModule, NgbDropdownModule, /* other imports */]
})
```

## Best Practices

✅ **Use ApiHandler** for consistent request handling
✅ **Use PaginationState** for paginated data
✅ **Signal Forms** for new forms (experimental) or Reactive Forms for production
✅ **Interceptors** for auth tokens
✅ **Guards** for route protection
✅ **Pipes** for data transformation
✅ **Subtle badges** (`bg-*-subtle` + `text-*`) instead of solid colors
✅ **NgBootstrap dropdown** with `container="body"` to prevent table overflow
✅ **Pause/Play icons** with warning/success colors for activate/deactivate actions
✅ **Zoneless detection** is default in Angular 21 - use signals for reactivity
