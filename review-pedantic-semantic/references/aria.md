# ARIA and ElementInternals

Detailed rules for ARIA usage and the ElementInternals API in
custom elements.

## ARIA: Less Is More

> No ARIA is better than bad ARIA.
> — [W3C WAI](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/)

ARIA is a repair tool, not a feature. It exists to fill gaps in native
HTML semantics — not to replace them, and not to be sprinkled
everywhere "just in case."

### Rule: prefer-native-html

If a native HTML element or attribute provides the semantics you need,
use it. Don't reach for ARIA.

**Bad:**

```html
<div role="button" tabindex="0" aria-pressed="false">Toggle</div>
<div role="navigation" aria-label="Main">...</div>
<span role="link" tabindex="0" onclick="navigate()">About</span>
```

**Good:**

```html
<button type="button" aria-pressed="false">Toggle</button>
<nav aria-label="Main">...</nav>
<a href="/about">About</a>
```

Native elements give you keyboard behavior, focus management, form
participation, and a11y tree semantics for free. ARIA gives you none
of those — only the label in the a11y tree.

### Rule: no-redundant-roles

Don't add a role that matches the element's implicit role.

**Bad:**

```html
<nav role="navigation">...</nav>
<main role="main">...</main>
<button role="button">Save</button>
<a href="/about" role="link">About</a>
<ul role="list">...</ul>
```

**Good:**

```html
<nav>...</nav>
<main>...</main>
<button>Save</button>
<a href="/about">About</a>
<ul>...</ul>
```

These are noise. They add bytes, visual clutter in source, and signal
that the author may not understand what the element already provides.

**Exception:** Safari removes list semantics from `<ul>` / `<ol>` when
`list-style: none` is applied. Adding `role="list"` explicitly is a
valid workaround in that specific case.

### Rule: no-aria-for-visibility

Don't use `aria-hidden="true"` as a substitute for actually hiding
content. If content should be hidden from everyone, use `hidden`,
`display: none`, or `visibility: hidden`. `aria-hidden` only hides
from the a11y tree — sighted users still see it, and keyboard users
can still Tab to focusable elements inside it.

**Bad:**

```html
<div aria-hidden="true">
  <button>This button is still tabbable!</button>
</div>
```

**Good:**

```html
<div hidden>
  <button>Properly hidden from everyone</button>
</div>
```

### Rule: no-aria-on-non-interactive

Don't add interactive ARIA attributes to non-interactive elements
without also providing the full keyboard and focus behavior.

**Bad:**

```html
<div role="checkbox" aria-checked="false">Accept terms</div>
```

This announces "checkbox, not checked" but the user can't Tab to it,
can't press Space to toggle it, and clicking does nothing. It's worse
than no ARIA at all — it creates an expectation it can't fulfill.

**Good:**

```html
<label>
  <input type="checkbox"> Accept terms
</label>
```

### Rule: dont-override-native-semantics

Don't change the role of a native element to something contradictory.

**Bad:**

```html
<h2 role="button">Click me</h2>
<a href="/about" role="button">About</a>
<table role="presentation"><!-- data table! --></table>
```

If you need a button, use `<button>`. If you need a link, use `<a>`.
Overriding semantics creates a Frankenstein element that has one
element's behavior and another's announced role.

---

## ElementInternals

`ElementInternals` is the platform API for giving custom elements
first-class semantics and form participation. Prefer it over ARIA
attribute hacks on the host element.

### Role and accessible name

Set the element's role and accessible properties through internals,
not `this.setAttribute('role', ...)`:

```js
class MyButton extends LitElement {
  static formAssociated = true;
  #internals = this.attachInternals();

  constructor() {
    super();
    this.#internals.role = 'button';
  }

  updated() {
    this.#internals.ariaLabel = this.label;
  }
}
```

**Why:** Host ARIA attributes can be overridden by consumers
(`<my-button role="link">`). Internals-set roles are the element's
default — they can still be overridden, but the default is correct.

### Form association

Custom elements that wrap form controls should use
`static formAssociated = true` and `ElementInternals` for:

- `setFormValue()` — participates in form submission
- `setValidity()` — participates in constraint validation
- `formStateRestoreCallback()` — supports autofill and back/forward
- `labels` — returns associated `<label>` elements

Don't re-implement form semantics with hidden `<input>` elements or
custom event dispatching when the platform provides it natively.

### States (`:--custom-state`)

Use `this.#internals.states` (the `CustomStateSet`) to expose
component states to CSS via `:--state-name` pseudo-classes instead of
reflecting boolean attributes solely for styling:

```js
if (this.pressed) {
  this.#internals.states.add('--pressed');
} else {
  this.#internals.states.delete('--pressed');
}
```

```css
:host(:--pressed) { /* styles */ }
```

### Accessibility tree over ARIA attributes

When setting `ariaLabel`, `ariaDisabled`, `ariaExpanded`, etc., prefer
the `ElementInternals` properties over `this.setAttribute('aria-*')`.
The internals API sets the element's **default** accessible properties
without polluting the DOM with attributes that could conflict with
author-supplied overrides.
