# Loading & Error States

## Table of Contents
- [Loading States with Signals](#loading-states-with-signals)
- [Error Handling with Signals](#error-handling-with-signals)
- [Async Pipe Pattern](#async-pipe-pattern)
- [Combined Loading/Error/Empty States](#combined-loadingerrorempty-states)
- [Reusable Loading Component](#reusable-loading-component)
- [Global Error Handler](#global-error-handler)
- [Best Practices](#best-practices)

---

## Loading States with Signals

```typescript
import { Component, signal } from '@angular/core';

export class UserListComponent {
  isLoading = signal(false);
  users = signal<User[]>([]);

  loadUsers() {
    this.isLoading.set(true);
    this.userService.getUsers().subscribe({
      next: (data) => {
        this.users.set(data);
        this.isLoading.set(false);
      },
      error: () => this.isLoading.set(false)
    });
  }
}
```

**Template:**
```html
@if (isLoading()) {
  <div class="spinner-border">Loading...</div>
} @else {
  <div *ngFor="let user of users()">
    {{ user.name }}
  </div>
}
```

## Error Handling with Signals

```typescript
export class UserListComponent {
  users = signal<User[]>([]);
  error = signal<string | null>(null);
  isLoading = signal(false);

  loadUsers() {
    this.isLoading.set(true);
    this.error.set(null);

    this.userService.getUsers().subscribe({
      next: (data) => {
        this.users.set(data);
        this.isLoading.set(false);
      },
      error: (err: HttpErrorResponse) => {
        this.error.set(err.message || 'Failed to load users');
        this.isLoading.set(false);
      }
    });
  }
}
```

**Template:**
```html
@if (error()) {
  <div class="alert alert-danger">
    {{ error() }}
    <button (click)="loadUsers()">Retry</button>
  </div>
}

@if (isLoading()) {
  <div class="spinner-border"></div>
} @else if (users().length === 0) {
  <p>No users found</p>
} @else {
  <div *ngFor="let user of users()">{{ user.name }}</div>
}
```

## Async Pipe Pattern

```typescript
// Service
users$ = this.http.get<User[]>('/api/users');

// Component
users$ = this.userService.getUsers();
```

**Template:**
```html
<div *ngIf="users$ | async as users; else loading">
  <div *ngFor="let user of users">{{ user.name }}</div>
</div>

<ng-template #loading>
  <div class="spinner-border">Loading...</div>
</ng-template>
```

## Combined Loading/Error/Empty States

```typescript
type LoadingState<T> = {
  loading: boolean;
  data: T | null;
  error: string | null;
};

export class UserListComponent {
  state = signal<LoadingState<User[]>>({
    loading: false,
    data: null,
    error: null
  });

  loadUsers() {
    this.state.set({ loading: true, data: null, error: null });

    this.userService.getUsers().subscribe({
      next: (data) => this.state.set({ loading: false, data, error: null }),
      error: (err) => this.state.set({ loading: false, data: null, error: err.message })
    });
  }
}
```

**Template:**
```html
@if (state().loading) {
  <div class="spinner-border"></div>
} @else if (state().error) {
  <div class="alert alert-danger">{{ state().error }}</div>
} @else if (state().data) {
  <div *ngFor="let user of state().data">{{ user.name }}</div>
} @else {
  <p>No data available</p>
}
```

## Reusable Loading Component

```typescript
// shared/components/loading-spinner/loading-spinner.component.ts
@Component({
  selector: 'app-loading-spinner',
  standalone: true,
  template: `
    <div class="text-center p-4">
      <div class="spinner-border" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      @if (message()) {
        <p class="mt-2">{{ message() }}</p>
      }
    </div>
  `
})
export class LoadingSpinnerComponent {
  message = input<string>();
}

// Usage
<app-loading-spinner [message]="'Loading users...'" />
```

## Global Error Handler

```typescript
// core/interceptors/error.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { catchError, throwError } from 'rxjs';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      let errorMessage = 'An error occurred';

      if (error.error instanceof ErrorEvent) {
        // Client-side error
        errorMessage = error.error.message;
      } else {
        // Server-side error
        errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
      }

      console.error(errorMessage);
      return throwError(() => new Error(errorMessage));
    })
  );
};

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([errorInterceptor])
    )
  ]
};
```

## Best Practices

✅ **Show loading immediately** - Set loading state before async call
✅ **Handle errors gracefully** - Show user-friendly messages
✅ **Provide retry options** - Let users retry failed operations
✅ **Use signals for local state** - Reactive and efficient
✅ **Use async pipe for observables** - Automatic subscription management

❌ **Don't leave users hanging** - Always show loading feedback
❌ **Don't expose technical errors** - Show friendly messages
❌ **Don't forget empty states** - Handle no data scenarios
