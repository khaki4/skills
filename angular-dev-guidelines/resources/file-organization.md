# File Organization

## Project Structure

```
src/app/
├── features/              # Feature modules (domain-specific)
│   ├── users/
│   │   ├── services/      # User-related services
│   │   ├── components/    # Feature-specific components
│   │   ├── pages/         # Routable page components
│   │   ├── models/        # TypeScript interfaces
│   │   ├── guards/        # Route guards
│   │   └── users.routes.ts
│   └── products/
│       └── ...
│
├── shared/                # Reusable across features
│   ├── components/        # Generic components
│   │   ├── loading-spinner/
│   │   └── error-message/
│   ├── directives/        # Custom directives
│   ├── pipes/             # Custom pipes
│   └── models/            # Shared interfaces
│
├── core/                  # Singleton services
│   ├── services/
│   │   ├── auth.service.ts
│   │   └── api.service.ts
│   ├── interceptors/
│   │   ├── auth.interceptor.ts
│   │   └── error.interceptor.ts
│   └── guards/
│       └── auth.guard.ts
│
└── app.routes.ts          # Root routes
```

## Naming Conventions

**Components:**
- `user-list.component.ts` (kebab-case)
- Selector: `app-user-list`

**Services:**
- `user.service.ts`
- Class: `UserService`

**Models:**
- `user.model.ts`
- Interface: `User`, `CreateUserDto`

**Routes:**
- `users.routes.ts`

## Features vs Shared vs Core

### Features Directory
**When to use:** Domain-specific functionality

```typescript
// features/users/services/user.service.ts
export class UserService {
  // User-specific business logic
}
```

### Shared Directory
**When to use:** Truly reusable across multiple features

```typescript
// shared/components/loading-spinner/loading-spinner.component.ts
export class LoadingSpinnerComponent {
  // Used by many features
}
```

### Core Directory
**When to use:** App-wide singleton services

```typescript
// core/services/auth.service.ts
@Injectable({ providedIn: 'root' })
export class AuthService {
  // Single instance across app
}
```

## Feature Structure Example

```
features/products/
├── services/
│   └── product.service.ts          # API calls
├── components/
│   ├── product-card.component.ts   # Presentational
│   └── product-filter.component.ts
├── pages/
│   ├── product-list.component.ts   # Route component
│   └── product-detail.component.ts
├── models/
│   ├── product.model.ts            # Interfaces
│   └── product-filter.model.ts
└── products.routes.ts              # Feature routes
```

## Best Practices

✅ **Feature-based organization** - Group by feature, not by file type
✅ **Flat structure when possible** - Avoid deep nesting
✅ **Consistent naming** - Follow Angular style guide
✅ **Index files** - Export public API from features

❌ **Don't organize by file type** - No separate folders for all components
❌ **Don't mix concerns** - Keep features isolated
❌ **Avoid circular dependencies** - Features shouldn't import each other
