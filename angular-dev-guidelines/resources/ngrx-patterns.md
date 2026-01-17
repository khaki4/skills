# NgRx & Enterprise State Management Patterns

Complete guide for complex Angular applications requiring scalable state management, traceable flows, and DevTools integration.

## Table of Contents
- [When to Use NgRx](#when-to-use-ngrx)
- [Core Architecture Principles](#core-architecture-principles)
- [Component Pattern](#component-pattern)
- [NgRx Effects Patterns](#ngrx-effects-patterns)
- [Signal Store Pattern](#signal-store-pattern)
- [Reactive Forms Pattern](#reactive-forms-pattern)
- [RxJS Operator Selection Guide](#rxjs-operator-selection-guide)
- [Directory Structure](#directory-structure)
- [Checklist](#checklist)

---

## When to Use NgRx

> **Note**: data-fetching.md patterns (BehaviorSubject, simple services) are intentional simplifications of these principles. Use them for simple apps. Scale up to NgRx when needed.

| Indicator | Simple Pattern (data-fetching.md) | NgRx Pattern (this file) |
|-----------|-----------------------------------|--------------------------|
| Team size | 1-3 developers | 4+ developers |
| State complexity | Single feature state | Cross-feature state sharing |
| Debug needs | Console.log is enough | Need Redux DevTools time-travel |
| Flow complexity | Linear API calls | Multi-step chains (A triggers B triggers C) |
| Offline/Optimistic | Not needed | Required |

**Key Insight**: If you can't explain why you need NgRx, you probably don't. Start simple, scale up when pain appears.

---

## Core Architecture Principles

These principles apply regardless of whether you use simple services or NgRx:

1. **Explicit State Changes**: State changes through defined actions/methods only
2. **Traceable Flows**: Every state change must be visible in DevTools
3. **Unidirectional Data Flow**: Action -> State -> View (never backward)
4. **Component-Logic Separation**: Components only dispatch and select

---

## Component Pattern

Components handle ONLY **dispatch** and **select**. No business logic.

```typescript
@Component({
  selector: 'app-order',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input [formControl]="form.controls.name">
      <button type="submit" [disabled]="loading()">Save</button>
    </form>

    @if (error()) {
      <div class="alert alert-danger">{{ error() }}</div>
    }
  `
})
export class OrderComponent {
  private store = inject(Store);

  // State subscription (select)
  order = this.store.selectSignal(selectCurrentOrder);
  loading = this.store.selectSignal(selectOrderLoading);
  error = this.store.selectSignal(selectOrderError);

  // Form definition
  form = new FormGroup({
    name: new FormControl(''),
    quantity: new FormControl(0)
  });

  // External state -> Form sync (explicit)
  constructor() {
    effect(() => {
      const order = this.order();
      if (order) {
        this.form.patchValue(order);
      }
    });
  }

  // Action dispatch
  onSubmit() {
    this.store.dispatch(OrderActions.create({
      payload: this.form.getRawValue()
    }));
  }
}
```

---

## NgRx Effects Patterns

Effects handle all side effects declaratively. Flows are traceable in DevTools.

### Basic API Call Flow

```typescript
@Injectable()
export class OrderEffects {
  private actions$ = inject(Actions);
  private orderApi = inject(OrderApiService);

  create$ = createEffect(() =>
    this.actions$.pipe(
      ofType(OrderActions.create),
      exhaustMap(({ payload }) =>
        this.orderApi.create(payload).pipe(
          map(order => OrderActions.createSuccess({ order })),
          catchError(error => of(OrderActions.createFailure({ error })))
        )
      )
    )
  );
}
```

### Chain Flows (Multi-Step Actions)

```typescript
// Order success -> Decrease inventory -> Send notification
orderSuccessFlow$ = createEffect(() =>
  this.actions$.pipe(
    ofType(OrderActions.createSuccess),
    switchMap(({ order }) => [
      InventoryActions.decrease({ items: order.items }),
      NotificationActions.send({ type: 'order_created', orderId: order.id })
    ])
  )
);

// Payment failure -> Rollback order -> Log error
paymentFailureFlow$ = createEffect(() =>
  this.actions$.pipe(
    ofType(PaymentActions.failure),
    switchMap(({ orderId, error }) => [
      OrderActions.rollback({ orderId }),
      LogActions.error({ context: 'payment', error })
    ])
  )
);
```

### Reusable API Effect Factory

```typescript
// Common API flow factory function
export function createApiEffect<TPayload, TResult>(
  actions$: Actions,
  triggerAction: ActionCreator<string, (props: { payload: TPayload }) => { payload: TPayload }>,
  apiCall: (payload: TPayload) => Observable<TResult>,
  successAction: ActionCreator<string, (props: { data: TResult }) => { data: TResult }>,
  failureAction: ActionCreator<string, (props: { error: Error }) => { error: Error }>
) {
  return createEffect(() =>
    actions$.pipe(
      ofType(triggerAction),
      exhaustMap(({ payload }) =>
        apiCall(payload).pipe(
          map(data => successAction({ data })),
          catchError(error => of(failureAction({ error })))
        )
      )
    )
  );
}
```

---

## Signal Store Pattern

For service-level state, use Signals with explicit change methods only.

```typescript
@Injectable({ providedIn: 'root' })
export class UiStateService {
  // Private writable signals
  private _sidebarOpen = signal(true);
  private _theme = signal<'light' | 'dark'>('light');

  // Public readonly signals
  sidebarOpen = this._sidebarOpen.asReadonly();
  theme = this._theme.asReadonly();

  // Explicit change methods (the only way to change state)
  toggleSidebar() {
    this._sidebarOpen.update(v => !v);
  }

  setTheme(theme: 'light' | 'dark') {
    this._theme.set(theme);
  }
}
```

**Why This Pattern?**
- External code cannot directly mutate state
- All changes go through named methods (traceable)
- Easy to add logging/analytics to change methods

---

## Reactive Forms Pattern

### Form Factory

```typescript
@Injectable()
export class OrderFormFactory {
  create(initial?: Partial<Order>): FormGroup<OrderForm> {
    return new FormGroup({
      name: new FormControl(initial?.name ?? '', [
        Validators.required,
        Validators.minLength(2)
      ]),
      quantity: new FormControl(initial?.quantity ?? 1, [
        Validators.required,
        Validators.min(1)
      ]),
      items: new FormArray<FormGroup<OrderItemForm>>([])
    });
  }

  addItem(form: FormGroup<OrderForm>, item?: Partial<OrderItem>) {
    form.controls.items.push(new FormGroup({
      productId: new FormControl(item?.productId ?? ''),
      amount: new FormControl(item?.amount ?? 0)
    }));
  }
}
```

### Form Value Extraction

```typescript
// Always use getRawValue() to include disabled fields
onSubmit() {
  if (this.form.invalid) return;

  const payload = this.form.getRawValue();
  this.store.dispatch(OrderActions.create({ payload }));
}
```

---

## RxJS Operator Selection Guide

| Situation | Operator | Reason |
|-----------|----------|--------|
| Form submit (prevent duplicates) | `exhaustMap` | Ignore while in progress |
| Search (need latest only) | `switchMap` | Cancel previous requests |
| Sequential processing | `concatMap` | Guarantee order |
| Parallel processing | `mergeMap` | Execute simultaneously |

### Visual Guide

```
exhaustMap: [Request A] -----> [Response A]
            [Request B ignored while A is in progress]

switchMap:  [Request A] --X    (cancelled)
            [Request B] -----> [Response B]

concatMap:  [Request A] -----> [Response A]
                              [Request B] -----> [Response B]

mergeMap:   [Request A] -----> [Response A]
            [Request B] -----> [Response B]
            (both run simultaneously)
```

---

## Directory Structure

```
src/app/
├── core/
│   ├── services/          # API services
│   └── guards/
├── store/
│   ├── actions/
│   │   ├── order.actions.ts
│   │   └── user.actions.ts
│   ├── reducers/
│   │   ├── order.reducer.ts
│   │   └── user.reducer.ts
│   ├── effects/
│   │   ├── order.effects.ts
│   │   └── user.effects.ts
│   ├── selectors/
│   │   ├── order.selectors.ts
│   │   └── user.selectors.ts
│   └── index.ts
├── features/
│   └── order/
│       ├── components/     # Presentational components
│       ├── containers/     # Smart components (dispatch/select only)
│       ├── forms/          # Form factories
│       └── order.routes.ts
└── shared/
    ├── components/
    └── pipes/
```

---

## Checklist

When developing new features, verify:

- [ ] Component has NO business logic (only dispatch/select)
- [ ] State changes go through action -> reducer path only
- [ ] Flows are separated into Effects
- [ ] No two-way binding (`[(ngModel)]`) used
- [ ] Forms use Reactive Forms pattern
- [ ] Action flow is traceable in Redux DevTools

---

## See Also

- [data-fetching.md](data-fetching.md) - Simpler patterns for basic apps (intentional simplification of these principles)
- [component-patterns.md](component-patterns.md) - Component structure
- [common-patterns.md](common-patterns.md) - Forms, interceptors, guards
