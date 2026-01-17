# Data Fetching & State Management

Complete guide to data fetching patterns in Angular 21 using HttpClient, RxJS, signals, and the new HTTP responseType property.

> **Architecture Note**: These patterns are intentional simplifications of the full architecture principles (see [ngrx-patterns.md](ngrx-patterns.md)). They follow the same core principles:
> - **Explicit State Changes**: BehaviorSubject/signals with encapsulated update methods
> - **Unidirectional Data Flow**: Service -> Observable -> Component
> - **Component-Logic Separation**: HTTP logic stays in services, not components
>
> For complex apps with cross-feature state, chain flows, or DevTools requirements, see [ngrx-patterns.md](ngrx-patterns.md).

## Table of Contents
- [Service Layer Pattern](#service-layer-pattern)
- [Component Data Fetching Patterns](#component-data-fetching-patterns)
- [RxJS Operators Patterns](#rxjs-operators-patterns)
- [State Management with BehaviorSubject](#state-management-with-behaviorsubject)
- [HTTP Interceptors](#http-interceptors)
- [Summary](#summary)

---

## Service Layer Pattern

### Creating a Feature Service

```typescript
// services/user.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError, BehaviorSubject } from 'rxjs';
import { catchError, map, tap } from 'rxjs/operators';

// Models
import { User, CreateUserDto } from '../models/user.model';

@Injectable({
  providedIn: 'root' // Singleton service
})
export class UserService {
  private http = inject(HttpClient);
  private readonly apiUrl = '/api/users';

  // State management with BehaviorSubject
  private usersSubject = new BehaviorSubject<User[]>([]);
  public users$ = this.usersSubject.asObservable();

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl).pipe(
      tap(users => this.usersSubject.next(users)),
      catchError(this.handleError)
    );
  }

  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }

  createUser(dto: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.apiUrl, dto).pipe(
      tap(user => {
        const current = this.usersSubject.value;
        this.usersSubject.next([...current, user]);
      }),
      catchError(this.handleError)
    );
  }

  updateUser(id: number, dto: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, dto).pipe(
      tap(updated => {
        const current = this.usersSubject.value;
        const index = current.findIndex(u => u.id === id);
        if (index !== -1) {
          const updated = [...current];
          updated[index] = { ...current[index], ...dto };
          this.usersSubject.next(updated);
        }
      }),
      catchError(this.handleError)
    );
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      tap(() => {
        const current = this.usersSubject.value;
        this.usersSubject.next(current.filter(u => u.id !== id));
      }),
      catchError(this.handleError)
    );
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = 'An error occurred';

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Error: ${error.error.message}`;
    } else {
      // Server-side error
      errorMessage = `Server Error: ${error.status} - ${error.message}`;

      // Angular 21: Use responseType for CORS debugging
      // error.responseType can be: 'basic', 'cors', 'opaque', 'opaqueredirect'
      if (error.status === 0) {
        console.error('CORS or network error. Response type:', (error as any).responseType);
      }
    }

    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

---

## Component Data Fetching Patterns

### Pattern 1: Async Pipe (RECOMMENDED)

```typescript
import { Component, OnInit, inject } from '@angular/core';
import { Observable } from 'rxjs';
import { UserService } from '../services/user.service';
import { User } from '../models/user.model';

@Component({
  selector: 'app-user-list',
  // standalone: true is default in Angular 21
  template: `
    <div class="container mt-4">
      @if (users$ | async; as users) {
        <div class="row">
          @for (user of users; track user.id) {
            <div class="col-md-4 mb-3">
              <div class="card">
                <div class="card-body">
                  <h5 class="card-title">{{ user.name }}</h5>
                  <p class="card-text">{{ user.email }}</p>
                </div>
              </div>
            </div>
          }
        </div>
      } @else {
        <div class="spinner-border"></div>
      }
    </div>
  `
})
export class UserListComponent implements OnInit {
  private userService = inject(UserService);

  users$!: Observable<User[]>;

  ngOnInit(): void {
    this.users$ = this.userService.getUsers();
  }
}
```

**Benefits:**
- Automatic subscription/unsubscription
- No memory leaks
- Clean template syntax
- Works with Angular's async pipe

---

### Pattern 2: Signals with Manual Subscription

```typescript
import { Component, OnInit, OnDestroy, inject, signal } from '@angular/core';
import { Subject, takeUntil } from 'rxjs';
import { UserService } from '../services/user.service';
import { User } from '../models/user.model';

@Component({
  selector: 'app-user-list',
  // standalone: true is default in Angular 21
  template: `
    <div class="container mt-4">
      @if (isLoading()) {
        <div class="spinner-border"></div>
      } @else if (error()) {
        <div class="alert alert-danger">{{ error() }}</div>
      } @else {
        @for (user of users(); track user.id) {
          <div class="card mb-2">
            <div class="card-body">{{ user.name }}</div>
          </div>
        }
      }
    </div>
  `
})
export class UserListComponent implements OnInit, OnDestroy {
  private userService = inject(UserService);
  private destroy$ = new Subject<void>();

  // Signals for state
  users = signal<User[]>([]);
  isLoading = signal(false);
  error = signal<string | null>(null);

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading.set(true);
    this.error.set(null);

    this.userService.getUsers()
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (users) => {
          this.users.set(users);
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

**Benefits:**
- Fine-grained control over state
- Easy to add loading/error states
- Better for complex state management
- Signals provide reactivity

---

## RxJS Operators Patterns

### Common Operators

```typescript
import { Component, inject, signal } from '@angular/core';
import { FormControl } from '@angular/forms';
import { debounceTime, distinctUntilChanged, switchMap, catchError, tap } from 'rxjs/operators';
import { of } from 'rxjs';

@Component({
  selector: 'app-user-search',
  // standalone: true is default in Angular 21
  template: `
    <input
      [formControl]="searchControl"
      type="text"
      class="form-control"
      placeholder="Search users..."
    />

    @if (isSearching()) {
      <div class="spinner-border spinner-border-sm"></div>
    }

    @for (user of searchResults(); track user.id) {
      <div class="list-group-item">{{ user.name }}</div>
    }
  `
})
export class UserSearchComponent {
  private userService = inject(UserService);

  searchControl = new FormControl('');
  searchResults = signal<User[]>([]);
  isSearching = signal(false);

  ngOnInit(): void {
    this.searchControl.valueChanges.pipe(
      debounceTime(300), // Wait 300ms after user stops typing
      distinctUntilChanged(), // Only if value actually changed
      tap(() => this.isSearching.set(true)),
      switchMap(query =>
        query
          ? this.userService.searchUsers(query).pipe(
              catchError(() => {
                this.isSearching.set(false);
                return of([]);
              })
            )
          : of([])
      )
    ).subscribe(results => {
      this.searchResults.set(results);
      this.isSearching.set(false);
    });
  }
}
```

**Key Operators:**
- `debounceTime` - Delay emissions
- `distinctUntilChanged` - Only emit when value changes
- `switchMap` - Cancel previous requests
- `catchError` - Handle errors gracefully
- `tap` - Side effects without modifying stream
- `map` - Transform data
- `filter` - Filter emissions

---

## State Management with BehaviorSubject

### Shared State Service

```typescript
// services/cart.service.ts
import { Injectable, inject } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Injectable({
  providedIn: 'root'
})
export class CartService {
  private cartSubject = new BehaviorSubject<CartItem[]>([]);

  // Public observables
  cart$ = this.cartSubject.asObservable();
  totalItems$ = this.cart$.pipe(
    map(items => items.reduce((sum, item) => sum + item.quantity, 0))
  );
  totalPrice$ = this.cart$.pipe(
    map(items => items.reduce((sum, item) => sum + (item.price * item.quantity), 0))
  );

  addItem(item: CartItem): void {
    const currentCart = this.cartSubject.value;
    const existingItem = currentCart.find(i => i.id === item.id);

    if (existingItem) {
      // Update quantity
      const updatedCart = currentCart.map(i =>
        i.id === item.id
          ? { ...i, quantity: i.quantity + item.quantity }
          : i
      );
      this.cartSubject.next(updatedCart);
    } else {
      // Add new item
      this.cartSubject.next([...currentCart, item]);
    }
  }

  removeItem(id: number): void {
    const currentCart = this.cartSubject.value;
    this.cartSubject.next(currentCart.filter(item => item.id !== id));
  }

  clearCart(): void {
    this.cartSubject.next([]);
  }

  getCartValue(): CartItem[] {
    return this.cartSubject.value;
  }
}
```

### Using Shared State in Components

```typescript
@Component({
  selector: 'app-cart-summary',
  // standalone: true is default in Angular 21
  template: `
    <div class="card">
      <div class="card-body">
        <h5>Cart Summary</h5>
        <p>Items: {{ totalItems$ | async }}</p>
        <p>Total: {{ totalPrice$ | async | currency }}</p>
      </div>
    </div>
  `
})
export class CartSummaryComponent {
  private cartService = inject(CartService);

  totalItems$ = this.cartService.totalItems$;
  totalPrice$ = this.cartService.totalPrice$;
}
```

---

## HTTP Interceptors

### Authentication Interceptor

```typescript
// interceptors/auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    const cloned = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
    return next(cloned);
  }

  return next(req);
};
```

### Error Interceptor

```typescript
// interceptors/error.interceptor.ts
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { catchError, throwError } from 'rxjs';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        // Unauthorized - redirect to login
        router.navigate(['/login']);
      } else if (error.status === 403) {
        // Forbidden
        console.error('Access forbidden');
      } else if (error.status === 500) {
        // Server error
        console.error('Server error occurred');
      }

      return throwError(() => error);
    })
  );
};
```

### Registering Interceptors

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './interceptors/auth.interceptor';
import { errorInterceptor } from './interceptors/error.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    )
  ]
};
```

---

## Summary

**Data Fetching Best Practices (Angular 21):**
1. **Services** - Create services for all HTTP operations
2. **Async Pipe** - Use for automatic subscription management
3. **Signals** - Use for component state with fine-grained control
4. **RxJS Operators** - Use appropriate operators (debounceTime, switchMap, etc.)
5. **BehaviorSubject** - Use for shared state across components
6. **Error Handling** - Always use catchError operator
7. **TakeUntil** - Unsubscribe in ngOnDestroy
8. **Interceptors** - Use for cross-cutting concerns (auth, errors)
9. **HTTP responseType** - Use for CORS debugging (Angular 21)

**See Also:**
- [ngrx-patterns.md](ngrx-patterns.md) - Scale up to NgRx for complex apps
- [component-patterns.md](component-patterns.md) - Component structure
- [loading-and-error-states.md](loading-and-error-states.md) - Error handling patterns
- [common-patterns.md](common-patterns.md) - Signal Forms and more
