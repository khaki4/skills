# Angular Routing Guide (Angular 21)

## Table of Contents
- [Basic Route Configuration](#basic-route-configuration)
- [Lazy Loading Features](#lazy-loading-features)
- [Route Guards](#route-guards)
- [Route Parameters](#route-parameters)
- [Query Parameters](#query-parameters)
- [Route Resolvers](#route-resolvers)
- [Router Navigation](#router-navigation)
- [Platform Navigation (Experimental)](#platform-navigation-experimental)
- [Route Configuration Options](#route-configuration-options)
- [Best Practices](#best-practices)

---

## Basic Route Configuration

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', loadComponent: () => import('./features/home/home.component').then(m => m.HomeComponent) },
  { path: '**', loadComponent: () => import('./shared/components/not-found.component').then(m => m.NotFoundComponent) }
];
```

## Lazy Loading Features

```typescript
export const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.routes').then(m => m.USERS_ROUTES)
  }
];

// features/users/users.routes.ts
export const USERS_ROUTES: Routes = [
  { path: '', component: UserListComponent },
  { path: ':id', component: UserDetailComponent }
];
```

## Route Guards

```typescript
// guards/auth.guard.ts
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  router.navigate(['/login']);
  return false;
};

// Usage
{
  path: 'admin',
  loadComponent: () => import('./features/admin/admin.component').then(m => m.AdminComponent),
  canActivate: [authGuard]
}
```

## Route Parameters

```typescript
// Component
import { ActivatedRoute } from '@angular/router';

export class UserDetailComponent {
  private route = inject(ActivatedRoute);
  userId = signal<number>(0);

  ngOnInit() {
    // Get route parameter
    this.userId.set(+this.route.snapshot.paramMap.get('id')!);

    // Or subscribe to changes
    this.route.paramMap.subscribe(params => {
      this.userId.set(+params.get('id')!);
    });
  }
}

// Route
{ path: 'users/:id', component: UserDetailComponent }
```

## Query Parameters

```typescript
// Navigate with query params
this.router.navigate(['/users'], {
  queryParams: { page: 1, search: 'john' }
});

// Read query params
this.route.queryParamMap.subscribe(params => {
  const page = params.get('page');
  const search = params.get('search');
});
```

## Route Resolvers

```typescript
// user.resolver.ts
export const userResolver: ResolveFn<User> = (route) => {
  const userService = inject(UserService);
  return userService.getUser(+route.paramMap.get('id')!);
};

// Route config
{
  path: 'users/:id',
  component: UserDetailComponent,
  resolve: { user: userResolver }
}

// Component
export class UserDetailComponent {
  user = this.route.snapshot.data['user'] as User;
}
```

## Router Navigation

```typescript
// Programmatic navigation
private router = inject(Router);

// Navigate to route
this.router.navigate(['/users']);

// Navigate with ID
this.router.navigate(['/users', userId]);

// Navigate relative to current route
this.router.navigate(['../'], { relativeTo: this.route });

// Navigate with query params
this.router.navigate(['/search'], { queryParams: { q: 'angular' } });
```

## Platform Navigation (Experimental)

Angular 21.1 introduces experimental platform navigation feature that enables the router to intercept native browser navigations.

```typescript
// app.config.ts
import { provideRouter, withPlatformNavigation } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withPlatformNavigation() // Enable experimental platform navigation
    )
  ]
};
```

**Platform Navigation Benefits:**
- **Intercepts external navigations** - Converts standard link clicks to SPA navigations
- **Native scroll restoration** - Uses browser's built-in scroll and focus restoration
- **Better accessibility** - Communicates ongoing navigations to the browser
- **Progressive enhancement** - Falls back gracefully if not supported

### Example: Enabling Platform Navigation

```typescript
// main.ts or app.config.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter, withPlatformNavigation } from '@angular/router';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(
      routes,
      withPlatformNavigation({
        scrollPositionRestoration: 'enabled', // Use native restoration
        anchorScrolling: 'enabled'
      })
    )
  ]
});
```

**Note:** Platform navigation is experimental in Angular 21.1. Use with caution in production.

## Route Configuration Options

```typescript
{
  path: 'users',
  component: UserListComponent,
  canActivate: [authGuard],              // Guard before activation
  canDeactivate: [unsavedChangesGuard],  // Guard before leaving
  resolve: { users: usersResolver },     // Prefetch data
  data: { title: 'Users', roles: ['admin'] }, // Custom data
  children: [                            // Nested routes
    { path: ':id', component: UserDetailComponent }
  ]
}
```

## Best Practices

✅ **Lazy load features** - Improves initial load time
✅ **Use route guards** - Protect routes that require auth
✅ **Use resolvers** - Prefetch data before navigation
✅ **Components are standalone by default** - No need to specify in Angular 21
✅ **Consider platform navigation** - For better browser integration (experimental)

❌ **Don't eager load everything** - Hurts performance
❌ **Don't put logic in guards** - Keep them simple
❌ **Don't forget 404 route** - Always have `**` wildcard
