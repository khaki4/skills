# TypeScript Standards & Best Practices

## Table of Contents
- [Strict Mode Configuration](#strict-mode-configuration)
- [Type Annotations](#type-annotations)
- [Interfaces vs Types](#interfaces-vs-types)
- [Generics](#generics)
- [Utility Types](#utility-types)
- [Null Safety](#null-safety)
- [Type Guards](#type-guards)
- [Enums](#enums)
- [Function Typing](#function-typing)
- [Best Practices](#best-practices)

---

## Strict Mode Configuration

### tsconfig.json Setup

```json
{
  "compilerOptions": {
    "strict": true,                           // Enable all strict checks
    "noImplicitAny": true,                   // Error on implicit 'any'
    "strictNullChecks": true,                // Null/undefined handling
    "strictFunctionTypes": true,             // Function parameter checks
    "strictBindCallApply": true,             // Strict bind/call/apply
    "strictPropertyInitialization": true,    // Class property initialization
    "noImplicitThis": true,                  // Error on 'this' expressions
    "alwaysStrict": true,                    // Parse in strict mode
    "noUnusedLocals": true,                  // Error on unused locals
    "noUnusedParameters": true,              // Error on unused parameters
    "noImplicitReturns": true,               // All code paths return value
    "noFallthroughCasesInSwitch": true       // Switch statement fallthrough
  }
}
```

---

## Type Annotations

### Explicit Type Annotations

```typescript
// ✅ ALWAYS: Explicit types for function parameters and returns
function getUserById(id: number): User | null {
  return this.users.find(u => u.id === id) ?? null;
}

// ✅ ALWAYS: Explicit types for public class properties
export class UserService {
  private users: User[] = [];
  public currentUser: User | null = null;
}

// ✅ ALWAYS: Explicit types for complex variables
const userData: APIResponse<User> = await this.fetchUser();

// ✅ OK: Type inference for simple primitives
const count = 10; // number inferred
const name = 'John'; // string inferred
const isActive = true; // boolean inferred
```

### Avoid `any` Type

```typescript
// ❌ NEVER: Using 'any' loses type safety
function processData(data: any): any {
  return data.value;
}

// ✅ ALWAYS: Use proper types or generics
function processData<T extends { value: unknown }>(data: T): T['value'] {
  return data.value;
}

// ✅ ALTERNATIVE: Use 'unknown' if type is truly unknown
function processData(data: unknown): string {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return String(data.value);
  }
  throw new Error('Invalid data');
}
```

---

## Interfaces vs Types

### When to Use Interfaces

```typescript
// ✅ Use interfaces for object shapes
interface User {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}

// ✅ Interfaces can be extended
interface Employee extends User {
  department: string;
  salary: number;
}

// ✅ Interfaces can be merged (declaration merging)
interface Window {
  customProperty: string;
}
```

### When to Use Types

```typescript
// ✅ Use types for unions
type Status = 'pending' | 'approved' | 'rejected';

// ✅ Use types for intersections
type UserWithTimestamps = User & Timestamps;

// ✅ Use types for mapped types
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

// ✅ Use types for tuple types
type Point = [number, number];

// ✅ Use types for function signatures
type Validator = (value: string) => boolean;
```

### Guideline

- **Interfaces** for object shapes and OOP patterns
- **Types** for unions, intersections, and complex types
- Be consistent within the same codebase

---

## Generics

### Basic Generic Functions

```typescript
// Generic function
function identity<T>(value: T): T {
  return value;
}

// Generic with constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Multiple type parameters
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}
```

### Generic Classes

```typescript
// Generic service class
export class DataService<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  getAll(): T[] {
    return this.items;
  }

  findById<K extends keyof T>(id: T[K]): T | undefined {
    return this.items.find(item => item === id);
  }
}

// Usage
const userService = new DataService<User>();
const bookService = new DataService<Book>();
```

### Generic Constraints

```typescript
// Constraint: must have 'id' property
interface HasId {
  id: number;
}

function findById<T extends HasId>(items: T[], id: number): T | undefined {
  return items.find(item => item.id === id);
}

// Constraint: must extend base class
class BaseEntity {
  id: number;
  createdAt: Date;
}

function saveEntity<T extends BaseEntity>(entity: T): Promise<T> {
  // Implementation
  return Promise.resolve(entity);
}
```

---

## Utility Types

### Built-in Utility Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial - Make all properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; password?: string; }

// Required - Make all properties required
type RequiredUser = Required<Partial<User>>;

// Readonly - Make all properties readonly
type ReadonlyUser = Readonly<User>;

// Pick - Select specific properties
type UserSummary = Pick<User, 'id' | 'name'>;
// { id: number; name: string; }

// Omit - Exclude specific properties
type UserWithoutPassword = Omit<User, 'password'>;
// { id: number; name: string; email: string; }

// Record - Create type with specific keys
type UserRoles = Record<'admin' | 'user' | 'guest', User[]>;
// { admin: User[]; user: User[]; guest: User[]; }

// ReturnType - Extract return type of function
type ApiResponse = ReturnType<typeof fetchUsers>;

// Parameters - Extract parameter types
type FetchUsersParams = Parameters<typeof fetchUsers>;
```

### Custom Utility Types

```typescript
// Make specific properties optional
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type CreateUserDto = Optional<User, 'id' | 'createdAt'>;

// Make specific properties required
type Required<T, K extends keyof T> = T & { [P in K]-?: T[P] };

// Deep partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
```

---

## Null Safety

### Strict Null Checks

```typescript
// ✅ Handle null/undefined explicitly
function getUserName(user: User | null): string {
  if (user === null) {
    return 'Guest';
  }
  return user.name;
}

// ✅ Use optional chaining
const userName = user?.profile?.name ?? 'Unknown';

// ✅ Nullish coalescing
const count = data.count ?? 0; // Only replaces null/undefined, not 0 or ''

// ❌ NEVER: Assume value exists
function getUserName(user: User): string {
  return user!.name; // ❌ Non-null assertion - dangerous!
}
```

### Optional Properties

```typescript
interface User {
  id: number;
  name: string;
  email?: string; // Optional property
  phone?: string;
}

// ✅ Check before using
function sendEmail(user: User): void {
  if (user.email) {
    // Type narrowed to string
    this.emailService.send(user.email);
  }
}
```

---

## Type Guards

### Built-in Type Guards

```typescript
// typeof type guard
function printValue(value: string | number): void {
  if (typeof value === 'string') {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}

// instanceof type guard
function handleError(error: Error | string): void {
  if (error instanceof Error) {
    console.error(error.message, error.stack);
  } else {
    console.error(error);
  }
}

// in operator
function processResponse(response: SuccessResponse | ErrorResponse): void {
  if ('data' in response) {
    // SuccessResponse
    console.log(response.data);
  } else {
    // ErrorResponse
    console.error(response.error);
  }
}
```

### Custom Type Guards

```typescript
// User-defined type guard
interface User {
  id: number;
  name: string;
}

interface Admin extends User {
  permissions: string[];
}

function isAdmin(user: User): user is Admin {
  return 'permissions' in user;
}

// Usage
function grantAccess(user: User): void {
  if (isAdmin(user)) {
    // Type narrowed to Admin
    console.log(user.permissions);
  }
}
```

---

## Enums

### Numeric Enums

```typescript
enum UserRole {
  Guest = 0,
  User = 1,
  Admin = 2,
  SuperAdmin = 3
}

// Usage
const role: UserRole = UserRole.Admin;
```

### String Enums (Preferred)

```typescript
// ✅ PREFERRED: String enums are more readable
enum BookState {
  Draft = 'Draft',
  UnderApproval = 'UnderApproval',
  Sent = 'Sent',
  Rejected = 'Rejected'
}

// Usage
const state: BookState = BookState.Draft;
```

### Const Enums (Performance)

```typescript
// Inlined at compile time - no runtime object
const enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT'
}

// Compiled to: if (direction === "UP")
if (direction === Direction.Up) {
  // ...
}
```

### Union Types as Alternative

```typescript
// ✅ ALTERNATIVE: Union types (simpler, more flexible)
type BookState = 'Draft' | 'UnderApproval' | 'Sent' | 'Rejected';

const state: BookState = 'Draft';
```

---

## Function Typing

### Function Signatures

```typescript
// Function type
type Validator = (value: string) => boolean;

// Function with optional parameters
function createUser(name: string, email?: string): User {
  // Implementation
}

// Function with default parameters
function fetchUsers(page: number = 1, limit: number = 10): Promise<User[]> {
  // Implementation
}

// Function with rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0);
}
```

### Async Functions

```typescript
// ✅ ALWAYS: Explicitly type async functions
async function fetchUser(id: number): Promise<User> {
  const response = await this.http.get<APIResponse<User>>(`/users/${id}`);
  return response.data;
}

// ✅ Handle errors with proper types
async function fetchUser(id: number): Promise<User | null> {
  try {
    const response = await this.http.get<APIResponse<User>>(`/users/${id}`);
    return response.data;
  } catch (error) {
    if (error instanceof HttpErrorResponse) {
      console.error('API Error:', error.message);
    }
    return null;
  }
}
```

---

## Best Practices

### ✅ DO

1. **Enable strict mode** in tsconfig.json
2. **Avoid `any`** - use `unknown` or proper types
3. **Use interfaces** for object shapes
4. **Use type guards** for runtime checks
5. **Handle null/undefined** explicitly
6. **Use readonly** for immutable data
7. **Prefer string enums** over numeric enums
8. **Use utility types** (Partial, Pick, Omit)
9. **Type function returns** explicitly
10. **Use generics** for reusable code

### ❌ DON'T

1. **Don't use `any`** - defeats TypeScript's purpose
2. **Don't use non-null assertion `!`** unless absolutely necessary
3. **Don't ignore compiler errors** - fix them properly
4. **Don't disable strict checks** - they prevent bugs
5. **Don't use `as` casting** unnecessarily
6. **Don't mix interfaces and types** randomly
7. **Don't create overly complex types** - keep it simple

---

## See Also

- [component-patterns.md](component-patterns.md) - Component typing
- [data-fetching.md](data-fetching.md) - Service typing patterns
- [complete-examples.md](complete-examples.md) - Typed examples
