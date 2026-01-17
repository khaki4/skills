# Performance Optimization (Angular 21)

## Table of Contents
- [Zoneless Change Detection (Default)](#zoneless-change-detection-default)
- [Lazy Loading](#lazy-loading)
- [Track Expression in @for](#track-expression-in-for)
- [Pure Pipes](#pure-pipes)
- [Signals for Reactivity](#signals-for-reactivity)
- [Unsubscribe from Observables](#unsubscribe-from-observables)
- [Virtual Scrolling](#virtual-scrolling)
- [Debounce User Input](#debounce-user-input)
- [Optimize Images](#optimize-images)
- [Bundle Optimization](#bundle-optimization)
- [Performance Checklist](#performance-checklist)

---

## Zoneless Change Detection (Default)

Angular 21 uses zoneless change detection by default - Zone.js is no longer included.

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-user-list',
  // standalone: true is default in Angular 21
  // Zoneless change detection is automatic
  template: `
    @for (user of users(); track user.id) {
      <div>{{ user.name }}</div>
    }
  `
})
export class UserListComponent {
  // Signals automatically trigger change detection
  users = signal<User[]>([]);
}
```

**Benefits:**
- **No Zone.js overhead** - Smaller bundle, faster execution
- **Signals trigger change detection** - Just use signals
- **Simpler mental model** - No Zone.js magic
- **Better debugging** - Clear change detection triggers

### If You Need Zone.js (Legacy Migration)

```typescript
// app.config.ts - Only for legacy code that needs Zone.js
import { provideZoneChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true })
  ]
};
```

## Lazy Loading

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./features/users/user-list.component')
      .then(m => m.UserListComponent) // ✅ Lazy load
  }
];
```

**Benefits:**
- Smaller initial bundle
- Faster first page load
- Load features on demand

## Track Expression in @for

In Angular 21, use `track` expression in `@for` loops instead of `trackBy` function:

```typescript
// ❌ BAD - No track expression
@for (user of users()) {
  <div>{{ user.name }}</div>
}

// ✅ GOOD - With track expression (required in @for)
@for (user of users(); track user.id) {
  <div>{{ user.name }}</div>
}

// ✅ GOOD - Track by index for simple arrays
@for (item of items(); track $index) {
  <div>{{ item }}</div>
}
```

**Note:** The `track` expression is required in `@for` blocks. It's not optional like `trackBy` was in `*ngFor`.

## Pure Pipes

```typescript
@Pipe({
  name: 'filterUsers',
  standalone: true,
  pure: true // ✅ Default, only recalculates when inputs change
})
export class FilterUsersPipe implements PipeTransform {
  transform(users: User[], searchTerm: string): User[] {
    return users.filter(u => u.name.includes(searchTerm));
  }
}
```

## Signals for Reactivity

```typescript
// ✅ GOOD - Signals are efficient
export class UserListComponent {
  users = signal<User[]>([]);
  searchTerm = signal('');

  // Computed signal - only recalculates when dependencies change
  filteredUsers = computed(() => {
    const term = this.searchTerm().toLowerCase();
    return this.users().filter(u => u.name.toLowerCase().includes(term));
  });
}
```

## Unsubscribe from Observables

```typescript
import { Subject, takeUntil } from 'rxjs';

export class UserListComponent implements OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit() {
    this.userService.getUsers()
      .pipe(takeUntil(this.destroy$)) // ✅ Auto-unsubscribe
      .subscribe(users => this.users.set(users));
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Or use async pipe - auto-unsubscribes
users$ = this.userService.getUsers(); // ✅ Template: users$ | async
```

## Virtual Scrolling

```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  imports: [ScrollingModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let user of users()">
        {{ user.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `
})
export class UserListComponent {
  users = signal<User[]>([]); // Large list
}
```

## Debounce User Input

```typescript
import { debounceTime, distinctUntilChanged } from 'rxjs/operators';

searchControl = new FormControl('');

ngOnInit() {
  this.searchControl.valueChanges
    .pipe(
      debounceTime(300), // Wait 300ms after user stops typing
      distinctUntilChanged(), // Only if value changed
      takeUntil(this.destroy$)
    )
    .subscribe(term => this.search(term));
}
```

## Optimize Images

```html
<!-- Lazy load images -->
<img [src]="user.avatar" loading="lazy" alt="User avatar">

<!-- Use responsive images -->
<img
  [srcset]="user.avatar + ' 1x, ' + user.avatarHd + ' 2x'"
  [src]="user.avatar"
  alt="User avatar">
```

## Bundle Optimization

```typescript
// ❌ BAD - Imports entire lodash
import _ from 'lodash';

// ✅ GOOD - Import only what you need
import { debounce } from 'lodash-es';
```

## Performance Checklist

✅ **Zoneless by default** - No Zone.js overhead in Angular 21
✅ **Lazy load features** via routing
✅ **Track expression** in @for loops (required)
✅ **Pure pipes** for transformations
✅ **Signals** for reactive state (triggers change detection)
✅ **Unsubscribe** from observables (takeUntil or async pipe)
✅ **Virtual scrolling** for long lists (1000+ items)
✅ **Debounce** user input
✅ **Optimize images** (lazy loading, responsive)
✅ **Tree-shakeable imports** (import specific functions)
✅ **Use Vitest** for faster testing (default in Angular 21)

❌ **Don't add Zone.js** unless absolutely needed for legacy code
❌ **Don't eager load all features**
❌ **Don't forget track** in @for loops
❌ **Don't create impure pipes** unnecessarily
❌ **Don't forget to unsubscribe**
