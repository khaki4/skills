# Styling with Bootstrap 5

## Table of Contents
- [Bootstrap Utility Classes](#bootstrap-utility-classes)
- [Component Styling Patterns](#component-styling-patterns)
- [Scoped Styles](#scoped-styles)
- [View Encapsulation](#view-encapsulation)
- [CSS Custom Properties](#css-custom-properties)
- [Responsive Design](#responsive-design)
- [Common UI Patterns](#common-ui-patterns)
- [Best Practices](#best-practices)

---

## Bootstrap Utility Classes

### Layout & Spacing

```html
<!-- Container -->
<div class="container">...</div>
<div class="container-fluid">...</div>

<!-- Grid -->
<div class="row">
  <div class="col-md-6">Half width on medium+</div>
  <div class="col-md-6">Half width on medium+</div>
</div>

<!-- Flexbox -->
<div class="d-flex justify-content-between align-items-center">
  <span>Left</span>
  <span>Right</span>
</div>

<!-- Spacing (m = margin, p = padding, t/b/s/e = top/bottom/start/end) -->
<div class="mt-3 mb-4 px-2">
  Margin top 3, margin bottom 4, padding horizontal 2
</div>
```

### Typography

```html
<!-- Headings -->
<h1 class="h1">Heading 1</h1>
<h2 class="h2">Heading 2</h2>

<!-- Text utilities -->
<p class="text-primary">Primary text</p>
<p class="text-muted">Muted text</p>
<p class="text-center">Centered text</p>
<p class="fw-bold">Bold text</p>
<p class="fst-italic">Italic text</p>

<!-- Font sizes -->
<p class="fs-1">Font size 1 (largest)</p>
<p class="fs-6">Font size 6 (smallest)</p>
```

### Colors

```html
<!-- Background colors -->
<div class="bg-primary text-white">Primary background</div>
<div class="bg-secondary text-white">Secondary background</div>
<div class="bg-success text-white">Success background</div>
<div class="bg-danger text-white">Danger background</div>
<div class="bg-warning text-dark">Warning background</div>
<div class="bg-info text-dark">Info background</div>
<div class="bg-light text-dark">Light background</div>
<div class="bg-dark text-white">Dark background</div>

<!-- Text colors -->
<span class="text-primary">Primary</span>
<span class="text-success">Success</span>
<span class="text-danger">Danger</span>
```

### Buttons

```html
<!-- Button variants -->
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-success">Success</button>
<button class="btn btn-danger">Danger</button>

<!-- Button sizes -->
<button class="btn btn-primary btn-sm">Small</button>
<button class="btn btn-primary">Normal</button>
<button class="btn btn-primary btn-lg">Large</button>

<!-- Outline buttons -->
<button class="btn btn-outline-primary">Outline Primary</button>

<!-- Button groups -->
<div class="btn-group">
  <button class="btn btn-primary">Left</button>
  <button class="btn btn-primary">Middle</button>
  <button class="btn btn-primary">Right</button>
</div>
```

---

## Component Styling Patterns

### Using Bootstrap in Components

```typescript
// component.ts
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="card">
      <div class="card-header">
        <h5 class="card-title mb-0">{{ user().name }}</h5>
      </div>
      <div class="card-body">
        <p class="card-text">{{ user().email }}</p>
        <button class="btn btn-primary btn-sm">View Profile</button>
      </div>
    </div>
  `,
  styleUrls: ['./user-card.component.scss']
})
export class UserCardComponent {
  user = input.required<User>();
}
```

### Component-Specific Styles

```scss
// user-card.component.scss

// Component-specific overrides
.card {
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
}

.card-header {
  background-color: #f8f9fa;
  border-bottom: 1px solid #dee2e6;
}

// Custom utility classes specific to this component
.user-status {
  &.active {
    color: var(--bs-success);
  }

  &.inactive {
    color: var(--bs-secondary);
  }
}
```

---

## Scoped Styles

### ViewEncapsulation Options

```typescript
import { ViewEncapsulation } from '@angular/core';

// Default - Emulated (styles scoped to component)
@Component({
  selector: 'app-my-component',
  styleUrls: ['./my-component.component.scss'],
  encapsulation: ViewEncapsulation.Emulated // Default
})

// None - Global styles (use sparingly)
@Component({
  selector: 'app-global-component',
  styleUrls: ['./global-component.component.scss'],
  encapsulation: ViewEncapsulation.None // Styles apply globally
})

// ShadowDom - True Shadow DOM (rarely used)
@Component({
  selector: 'app-shadow-component',
  styleUrls: ['./shadow-component.component.scss'],
  encapsulation: ViewEncapsulation.ShadowDom
})
```

### Host Styles

```scss
// Style the component host element
:host {
  display: block;
  padding: 1rem;
  background: white;
  border-radius: 8px;
}

// Conditional host styles
:host(.highlighted) {
  border: 2px solid var(--bs-primary);
}

:host-context(.dark-mode) {
  background: #1e1e1e;
  color: white;
}
```

---

## CSS Custom Properties

### Using Bootstrap CSS Variables

```scss
// Use Bootstrap's CSS custom properties
.custom-button {
  background-color: var(--bs-primary);
  color: var(--bs-light);
  border-radius: var(--bs-border-radius);
  padding: var(--bs-btn-padding-y) var(--bs-btn-padding-x);
}

// Override Bootstrap variables
:root {
  --bs-primary: #0066cc;
  --bs-primary-rgb: 0, 102, 204;
}
```

### Component-Level Variables

```scss
// Define component-specific variables
:host {
  --card-padding: 1.5rem;
  --card-border-color: #dee2e6;
  --card-hover-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.card {
  padding: var(--card-padding);
  border: 1px solid var(--card-border-color);

  &:hover {
    box-shadow: var(--card-hover-shadow);
  }
}
```

---

## Responsive Design

### Bootstrap Breakpoints

```html
<!-- Responsive classes: sm, md, lg, xl, xxl -->

<!-- Columns -->
<div class="row">
  <div class="col-12 col-md-6 col-lg-4">
    Full on mobile, half on tablet, third on desktop
  </div>
</div>

<!-- Display utilities -->
<div class="d-none d-md-block">Hidden on mobile, visible on tablet+</div>
<div class="d-block d-md-none">Visible on mobile, hidden on tablet+</div>

<!-- Text alignment -->
<p class="text-center text-md-start">Centered on mobile, left on tablet+</p>
```

### Custom Media Queries

```scss
// Use Bootstrap's breakpoints
@import 'bootstrap/scss/functions';
@import 'bootstrap/scss/variables';
@import 'bootstrap/scss/mixins';

.custom-component {
  padding: 1rem;

  @include media-breakpoint-up(md) {
    padding: 2rem;
  }

  @include media-breakpoint-up(lg) {
    padding: 3rem;
  }
}
```

---

## Common UI Patterns

### Cards

```html
<div class="card shadow-sm">
  <div class="card-header bg-primary text-white">
    <h5 class="mb-0">Card Title</h5>
  </div>
  <div class="card-body">
    <p class="card-text">Card content goes here.</p>
    <a href="#" class="btn btn-primary">Action</a>
  </div>
  <div class="card-footer text-muted">
    Last updated 3 mins ago
  </div>
</div>
```

### Badges & Pills

```html
<!-- Status badges -->
<span class="badge bg-success">Active</span>
<span class="badge bg-danger">Inactive</span>
<span class="badge bg-warning text-dark">Pending</span>

<!-- Pill badges -->
<span class="badge rounded-pill bg-primary">10</span>
<span class="badge rounded-pill bg-secondary">New</span>
```

### Alerts

```html
<!-- Alert variants -->
<div class="alert alert-success" role="alert">
  Operation completed successfully!
</div>

<div class="alert alert-danger" role="alert">
  An error occurred. Please try again.
</div>

<!-- Dismissible alert -->
<div class="alert alert-warning alert-dismissible fade show" role="alert">
  <strong>Warning!</strong> Please review your changes.
  <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>
```

### Modals

```html
<!-- Modal trigger -->
<button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#myModal">
  Open Modal
</button>

<!-- Modal -->
<div class="modal fade" id="myModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Modal Title</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        <p>Modal content...</p>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div>
  </div>
</div>
```

### Forms

```html
<!-- Form controls -->
<form>
  <div class="mb-3">
    <label for="email" class="form-label">Email address</label>
    <input type="email" class="form-control" id="email" placeholder="name@example.com">
  </div>

  <div class="mb-3">
    <label for="select" class="form-label">Select option</label>
    <select class="form-select" id="select">
      <option selected>Choose...</option>
      <option value="1">One</option>
      <option value="2">Two</option>
    </select>
  </div>

  <div class="mb-3 form-check">
    <input type="checkbox" class="form-check-input" id="check">
    <label class="form-check-label" for="check">
      Check me out
    </label>
  </div>

  <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

### Tables

```html
<table class="table table-striped table-hover">
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>John Doe</td>
      <td>john@example.com</td>
      <td><span class="badge bg-success">Active</span></td>
    </tr>
  </tbody>
</table>
```

---

## Best Practices

### ✅ DO

1. **Use Bootstrap utilities first** - Before writing custom CSS
2. **Keep styles scoped** - Use ViewEncapsulation.Emulated (default)
3. **Use SCSS variables** - Leverage Bootstrap's Sass variables
4. **Mobile-first approach** - Design for mobile, enhance for desktop
5. **Use semantic HTML** - Proper heading hierarchy, semantic tags
6. **Consistent spacing** - Use Bootstrap's spacing scale (0-5)
7. **Accessibility** - Use proper ARIA labels, color contrast

```html
<!-- ✅ GOOD -->
<div class="d-flex justify-content-between align-items-center mb-3">
  <h2 class="h4 mb-0">Users</h2>
  <button class="btn btn-primary btn-sm">Add User</button>
</div>
```

### ❌ DON'T

1. **Don't inline styles** - Use classes instead
2. **Don't use !important** - Indicates design system issues
3. **Don't duplicate Bootstrap** - Reuse existing utilities
4. **Don't break encapsulation** - Avoid ViewEncapsulation.None unless necessary
5. **Don't use IDs for styling** - Use classes
6. **Don't over-nest selectors** - Keep specificity low

```html
<!-- ❌ BAD -->
<div style="display: flex; margin-bottom: 20px;">
  <h2 id="title" style="color: #333;">Users</h2>
  <button style="background: blue; color: white;">Add User</button>
</div>
```

---

## See Also

- [component-patterns.md](component-patterns.md) - Component architecture
- [common-patterns.md](common-patterns.md) - UI component patterns
- [complete-examples.md](complete-examples.md) - Full feature examples
