# Accessibility — YemenMart

## 1. Overview

YemenMart targets **WCAG 2.1 Level AA** compliance across all platforms. Accessibility is built into the design system and component library from the start.

## 2. WCAG 2.1 AA Requirements

### 2.1 Perceivable

#### 1.1 Text Alternatives

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Images have alt text | `alt` attribute on all `<img>` | Required |
| Decorative images | `alt=""` and `role="presentation"` | Required |
| Icons have labels | `aria-label` on icon buttons | Required |
| Complex images | Longer descriptions via `aria-describedby` | Required |

#### 1.2 Time-Based Media

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Audio controls | Pause/stop/volume controls | Required |
| Captions | Subtitles for video content | Required |
| Audio description | Descriptions of visual content | Required |

#### 1.3 Adaptable

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Info and relationships | Semantic HTML (`<header>`, `<nav>`, `<main>`, `<footer>`) | Required |
| Meaningful sequence | Logical reading order in DOM | Required |
| Sensory characteristics | Not relying solely on color/shape/sound | Required |
| Orientation | Works in portrait and landscape | Required |
| Identify input purpose | `autocomplete` attributes on forms | Required |

#### 1.4 Distinguishable

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Color contrast | 4.5:1 for normal text, 3:1 for large text | Required |
| Resize text | Text resizable to 200% without loss | Required |
| Reflow | Content reflows at 320px width | Required |
| Text spacing | Adjustable without breaking layout | Required |
| Content on hover/focus | Dismissable, hoverable, persistent | Required |

### 2.2 Operable

#### 2.1 Keyboard Accessible

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Keyboard accessible | All functionality via keyboard | Required |
| No keyboard trap | Tab navigation reaches all elements | Required |
| Focus order | Logical tab order (RTL-aware) | Required |
| Focus visible | Visible focus indicator on all interactive elements | Required |

#### 2.2 Timing Adjustable

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Timer adjustable | Session timeout warnings | Required |
| Pause, stop, hide | Auto-scroll can be paused | Required |

#### 2.3 Seizures and Physical Reactions

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| No flashing | Nothing flashes more than 3 times/second | Required |
| Motion from interaction | `prefers-reduced-motion` respected | Required |

#### 2.4 Navigable

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Skip navigation | "Skip to main content" link | Required |
| Page titles | Descriptive `<title>` per page | Required |
| Focus order | Logical sequence | Required |
| Link purpose | Descriptive link text | Required |
| Multiple ways | Search, sitemap, navigation | Required |
| Headings | Proper heading hierarchy (h1 → h2 → h3) | Required |
| Labels | Visible labels for form inputs | Required |
| Focus visible | High-contrast focus rings | Required |

#### 2.5 Input Modalities

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Pointer gestures | Single pointer alternative for gestures | Required |
| Pointer cancellation | `mousedown` → `click` with confirmation | Required |
| Label in name | Accessible name matches visible label | Required |
| Motion actuation | No motion-only functionality | Required |

### 2.3 Understandable

#### 3.1 Readable

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Language of page | `lang="ar"` on `<html>` | Required |
| Language of parts | `lang` attribute on foreign text | Required |

#### 3.2 Predictable

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| On focus | No unexpected context changes on focus | Required |
| On input | No unexpected changes on input | Required |
| Navigation | Consistent navigation across pages | Required |
| Identification | Consistent identification of components | Required |

#### 3.3 Input Assistance

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Error identification | Clear error messages in Arabic | Required |
| Labels or instructions | Visible labels and hints | Required |
| Error suggestion | Suggestions for correction | Required |
| Error prevention | Confirmation before submission | Required |

### 2.4 Robust

#### 4.1 Compatible

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| Parsing | Valid HTML | Required |
| Name, role, value | ARIA attributes on custom components | Required |
| Status messages | `role="status"` for dynamic updates | Required |

## 3. ARIA Implementation

### Component ARIA Patterns

```typescript
// Button with loading state
<button
  aria-busy={loading}
  aria-disabled={disabled}
  aria-label={ariaLabel}
>
  {loading && <Spinner aria-hidden="true" />}
  {children}
</button>

// Modal dialog
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
>
  <h2 id="modal-title">{title}</h2>
  <div id="modal-description">{description}</div>
</div>

// Form field with error
<div>
  <label htmlFor={fieldId}>{label}</label>
  <input
    id={fieldId}
    aria-invalid={!!error}
    aria-describedby={error ? `${fieldId}-error` : undefined}
    aria-required={required}
  />
  {error && <span id={`${fieldId}-error`} role="alert">{error}</span>}
</div>

// Live region for notifications
<div role="status" aria-live="polite" aria-atomic="true">
  {notification}
</div>

// Product rating
<div role="img" aria-label={`Rating: ${rating} out of 5 stars`}>
  {stars.map((star, i) => (
    <Star key={i} filled={star <= rating} aria-hidden="true" />
  ))}
</div>
```

## 4. Keyboard Navigation

### Tab Order (RTL)

```
┌─────────────────────────────────────────────────┐
│  Header                                         │
│  [Logo] [Search] [Cart] [Account] [Language]    │
│                                                   │
│  Main Content                                     │
│  [Skip to content]                                │
│                                                   │
│  Product Grid                                     │
│  [Product 1] [Product 2] [Product 3]             │
│  [Product 4] [Product 5] [Product 6]             │
│                                                   │
│  Footer                                           │
│  [Links] [Contact] [Social] [Legal]              │
└─────────────────────────────────────────────────┘
```

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Tab | Move to next interactive element |
| Shift+Tab | Move to previous interactive element |
| Enter/Space | Activate button/link |
| Escape | Close modal/dropdown |
| Arrow keys | Navigate within component |
| Home/End | Move to first/last item in list |

## 5. Focus Management

```typescript
// Focus trap for modals
function trapFocus(modal: HTMLElement) {
  const focusableElements = modal.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const firstElement = focusableElements[0] as HTMLElement;
  const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

  modal.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;

    if (e.shiftKey) {
      if (document.activeElement === firstElement) {
        lastElement.focus();
        e.preventDefault();
      }
    } else {
      if (document.activeElement === lastElement) {
        firstElement.focus();
        e.preventDefault();
      }
    }
  });

  firstElement.focus();
}

// Restore focus when modal closes
function restoreFocus(previousFocus: HTMLElement) {
  previousFocus?.focus();
}
```

## 6. Color Contrast

| Element | Foreground | Background | Ratio | Pass |
|---------|-----------|------------|-------|------|
| Body text | neutral-900 (#111827) | white (#ffffff) | 17.4:1 | AA |
| Secondary text | neutral-500 (#6b7280) | white (#ffffff) | 5.0:1 | AA |
| Primary button | white (#ffffff) | primary-600 (#2563eb) | 4.6:1 | AA |
| Error text | error-600 (#dc2626) | white (#ffffff) | 4.8:1 | AA |
| Link text | primary-600 (#2563eb) | white (#ffffff) | 4.6:1 | AA |
| Placeholder | neutral-400 (#9ca3af) | white (#ffffff) | 3.0:1 | Large only |

## 7. Screen Reader Testing

### Test Checklist

| Screen Reader | Browser | Platform | Status |
|---------------|---------|----------|--------|
| NVDA | Chrome | Windows | Required |
| NVDA | Firefox | Windows | Required |
| JAWS | Chrome | Windows | Required |
| VoiceOver | Safari | macOS | Required |
| VoiceOver | Safari | iOS | Required |
| TalkBack | Chrome | Android | Required |

### Common Patterns

```typescript
// Announce page changes
function announcePageChange(title: string) {
  const announcer = document.getElementById('page-announcer');
  if (announcer) {
    announcer.textContent = title;
  }
}

// Announce dynamic updates
function announceUpdate(message: string, priority: 'polite' | 'assertive' = 'polite') {
  const announcer = document.createElement('div');
  announcer.setAttribute('role', 'status');
  announcer.setAttribute('aria-live', priority);
  announcer.className = 'sr-only';
  announcer.textContent = message;
  document.body.appendChild(announcer);
  setTimeout(() => announcer.remove(), 1000);
}
```

## 8. Testing Tools

| Tool | Purpose | Frequency |
|------|---------|-----------|
| axe-core | Automated accessibility testing | Every PR |
| Lighthouse | Performance and accessibility audit | Weekly |
| WAVE | Visual accessibility evaluation | Monthly |
| Manual testing | Keyboard and screen reader testing | Every sprint |
| Color contrast analyzer | Contrast ratio verification | Design review |

## 9. Related Files

| File | Description |
|------|-------------|
| `design-system.md` | Design system tokens |
| `responsive-design.md` | Responsive patterns |
| `rtl-design.md` | RTL layout implementation |
| `05-frontend/component-library.md` | Accessible components |
| `12-non-functional/README.md` | Non-functional requirements |

---

*Document Version: 1.0 | Last Updated: 2026-09-13 | Classification: Confidential*
