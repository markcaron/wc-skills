---
name: create-component
description: >
  Create a new Lit web component. Modular workflow: design API, implement,
  create demos, write tests, document. Use when asked to "create", "build",
  "implement", or "add" a new component.
tools: Read, Glob, Grep, Shell
---

# Create Component

Build a new Lit custom element. This skill is modular — each phase can be
invoked independently or run end-to-end.

## Phases

| Phase | Purpose | Entry point |
|-------|---------|-------------|
| 1. Analyze | Understand requirements and reference material | Any spec, design, or reference implementation |
| 2. Design API | Define the web-native public API | Phase 1 output or a description of the component |
| 3. Implement | Write the element class, CSS, and sub-elements | Phase 2 output |
| 4. Demos | Create HTML demo files | Phase 3 output |
| 5. Test | Write unit and a11y tests | Phase 3 output |
| 6. Document | Write README and JSDoc | Phase 3 output |
| 7. Audit | Run review-api and review-a11y | Phases 3–6 complete |

---

## Phase 1: Analyze

Gather requirements from whatever source material is available. This could
be a design mockup, an existing React/Angular/Vue component, a spec
document, or a verbal description.

1. Identify the component's purpose and scope
2. If a reference implementation exists, read it and identify:
   - Props/inputs that map to attributes vs properties vs slots vs CSS
   - Callbacks/outputs that map to DOM events
   - Internal state management
   - Form association needs
   - Sub-components needed
   - Dependencies on other custom elements
3. Identify all visual states and variants
4. Identify relevant [ARIA APG Patterns](https://w3.org/WAI/ARIA/apg/patterns/)

### Reference Translation Rules

When working from an existing framework component:

| Framework concept | Web component equivalent |
|---|---|
| `children` / content projection | Default slot |
| Render props / templates | Named slots |
| `className` / style props | Not needed (shadow DOM scoping) |
| `onChange` callback | `change` event (Event subclass) |
| `isExpanded` boolean prop | `expanded` boolean attribute |
| `variant="primary"` | `variant="primary"` (often 1:1) |
| Component composition | Slot composition |
| Context / providers | CSS custom properties, or `@lit/context` for JS values |
| `useEffect` / lifecycle | `willUpdate` / `updated` |
| Hooks / mixins | Lit reactive controllers |

---

## Phase 2: Design API

Read `ADVICE.md` first — it is the canonical reference for all API
decisions.

**Key principle:** easy to add, hard to remove. Ship less when in doubt.

### Prefer Native Web Platform

- ElementInternals for all ARIA and form association (never raw
  `attachInternals()` — use a controller or wrapper)
- When `aria-label` is indicated (e.g. icon-only button), provide an
  `accessible-label` attribute
- Slots for composition
- Native `<dialog>` instead of JS-managed modals
- Container queries instead of responsive/breakpoint attributes
- `<details>` for expandable content (free a11y)

### Attributes & Properties

- Enum-style `variant` over multiple booleans (exclusive, not stackable)
- Boolean attributes: present = true, absent = false. No `is-` prefix
- Don't reflect booleans to `"true"|"false"` strings
- Don't default reflected boolean attributes to `true`
- Avoid array/object properties; don't reflect them
- Multi-word attributes: dash-case with camelCase class fields
- Don't expose `aria-*` as public API; abstract behind custom attributes
- Attribute/slot pairs for content that could be plain or rich
- All attributes must have sensible defaults

### Slots

- Default slot for primary content
- Semantic names, not positional (`header`, not `top`)
- Don't provide placeholder "lorem ipsum" content; provide default
  content when sensible
- Prescriptive descriptions in docs ("Label text" not "The label slot")
- Hide empty slot containers; be wary of SSR hydration issues

### Events

- Event subclasses, not `CustomEvent` with detail
- Cancelable for destructive/state-changing actions
- Match native event names when wrapping native elements
- Child elements should own their interaction (e.g. click) and dispatch
  a composed, bubbling event (e.g. `activate`). The parent container
  listens on its host for that event and reacts — don't have the parent
  inspect `composedPath()` to identify which child was clicked

### Element CSS

- Native nesting — nest state and pseudo-element selectors inside
  `#container` with `&`:

```css
#container {
  display: flex;
  color: light-dark(#555, #aaa);

  &:hover:not(.disabled) {
    color: light-dark(#222, #ddd);
  }

  &.selected {
    color: blue;

    &::after { /* indicator */ }
  }

  &.disabled {
    color: light-dark(#999, #555);
  }
}
```

- `light-dark()` for colors
- Logical properties instead of directional
- Baseline 2024 features or earlier
- Don't nest `:host([attr])` with `&`
- Don't use `:host:has()` or `:host(:has())`
- Never use `:host-context`
- `pointer-events: none` on disabled hosts
- `:host(:focus-visible) #container` for focus rings (focus lives on
  the host via tabindex, style the container)

### CSS Custom Properties

- `--_` prefix for private custom properties
- Public properties as fallback defaults at use sites, not on `:host`
- Don't duplicate native CSS capabilities

### CSS Parts

- Name after internal element. Only expose with clear use cases
- Question aggressively — hard to remove

### Templates

- Prefer ID selectors in shadow DOM; avoid BEM
- Vertical attribute formatting when >2 attrs
- False case first in ternaries
- Use a `#container` div in the shadow root for all visual styling and
  layout. Keep `:host` minimal (`display: block`, `outline: none`, and
  any inherited CSS custom properties that slotted children need).
  Pass reactive state to the container via `classMap`:

```ts
import { classMap } from 'lit/directives/class-map.js';

render() {
  return html`
    <div id="container" class="${classMap({
      vertical: !!this.vertical,
      selected: !!this.selected,
    })}">
      <slot></slot>
    </div>
  `;
}
```

**IMPORTANT:** When a reference API does not cleanly map to a web
component API, surface that to the user for discussion.

---

## Phase 3: Implement

1. Create element directory
2. Write main element class:
   - `@customElement('prefix-name')` decorator
   - `static readonly styles` array
   - Reactive properties with `@property`
   - ElementInternals for ARIA and form association (via controller)
   - Template in `render()` method

3. Write CSS:
   - Native CSS nesting inside `#container` for state/pseudo selectors
   - `light-dark()`, logical properties
   - Shadow DOM scoping (IDs not BEM)
   - Token-derived fallback values
   - `box-sizing` reset on `#container`

4. Create sub-elements only if a11y/composition requires them
5. Update package exports if applicable

### Code Patterns

- ECMAScript private fields (`#field`), not TypeScript `private`
- `override` on any method that exists on the parent class — it is a
  TypeScript safety check (catches typos and renames), not a semantic
  statement about overriding behavior. The real distinction is `super`
  calls:
  - Standard CE lifecycle (`connectedCallback`, `disconnectedCallback`)
    and `update()`: always call `super` — Lit has real behavior there
  - Reactive update hooks (`render`, `willUpdate`, `updated`,
    `firstUpdated`, `shouldUpdate`): `super` not needed when extending
    LitElement directly (base implementations are no-ops), but required
    when extending another component that implements them
- `static styles = [styles]` array form
- Controllers for cross-cutting concerns
- `isServer` guards on browser-only APIs
- Quote attribute values in Lit templates `attr="${this.value}"`
- Side effects in `willUpdate`, not `render`

### Multi-Element Event Flow

When a component is split across parent and child elements (e.g. tabs +
tab), use the **child-dispatches / parent-listens** pattern:

1. The child element adds an event listener on itself (e.g. `click`)
2. The child dispatches a composed, bubbling event
   (`new Event('activate', { bubbles: true, composed: true })`)
3. The parent listens on its host via `addEventListener` in the
   constructor — not via template `@event` bindings on shadow DOM
   elements
4. The parent identifies the child via `event.target`

```ts
// Child element
constructor() {
  super();
  this.addEventListener('click', this.#onClick);
}

#onClick() {
  if (this.disabled) return;
  this.dispatchEvent(
    new Event('activate', { bubbles: true, composed: true }),
  );
}

// Parent container
constructor() {
  super();
  this.addEventListener('activate', this.#onChildActivate);
}

#onChildActivate(event: Event) {
  const child = event.target as ChildElement;
  const index = this.#children.indexOf(child);
  if (index >= 0) this.#select(index);
}
```

This keeps interaction ownership in the child and avoids brittle
`composedPath()` inspection in the parent.

---

## Phase 4: Demos

Create HTML demo files. If using a demo framework (e.g. `cem serve`,
Storybook), follow its conventions. Otherwise, create standalone HTML
files.

### Demo conventions

- One demo per variant/feature
- Index demo = simplest possible usage
- YAML frontmatter if the demo framework supports it:
  ```html
  ---
  name: Variant examples
  description: Shows primary, secondary, and link styles.
  ---
  ```
- Inline `<script type="module">` with element import
- No wrapper divs; minimal markup
- Demos are partials if the framework wraps them; full HTML otherwise

### Additional demos for WC-specific APIs

- Slot composition patterns
- Form-associated behavior (if applicable)
- CSS custom property theming overrides

---

## Phase 5: Test

Write tests from scratch. Use `@open-wc/testing` conventions or the
project's test framework.

### Test checklist

- [ ] Public API: all attributes, properties, events
- [ ] a11ySnapshot assertions for accessibility tree
- [ ] Keyboard navigation
- [ ] Form submission (if form-associated)
- [ ] Setup in `beforeEach`, assertions in `it` blocks
- [ ] No arrow functions in Mocha test blocks
- [ ] Test observable behavior (offsetWidth, computed styles), not shadow
      DOM internals
- [ ] Cover all component states (disabled, expanded, error, loading, etc.)

### Test patterns

```ts
describe('my-element', function() {
  let element: MyElement;

  beforeEach(async function() {
    element = await fixture(html`<my-element></my-element>`);
  });

  it('should have correct default role', async function() {
    const snapshot = await a11ySnapshot();
    expect(snapshot.role).to.equal('button');
  });

  describe('when disabled', function() {
    beforeEach(async function() {
      element.disabled = true;
      await element.updateComplete;
    });

    it('should reflect disabled in a11y tree', async function() {
      const snapshot = await a11ySnapshot();
      expect(snapshot.disabled).to.be.true;
    });
  });
});
```

---

## Phase 6: Document

### JSDoc

- `@summary` on element class
- `/** */` JSDoc on all public properties/methods
- CSS custom properties documented with `/** */` in CSS, co-located
  with declaration or `var()` reference
- CSS parts documented with HTML comments before the `part` element
- Slots documented with HTML comments before the `<slot>` element
- `@fires` for all dispatched events
- No unnecessary `@default` tags (the analyzer picks up initializers)

### README (optional)

If the element diverges from a reference implementation, document it:

1. **Title and summary** — element tag, one-line description
2. **Usage** — 2–3 HTML snippets showing common patterns
3. **Divergences from reference** — tables for:
   - Not implemented (reference features with no WC equivalent)
   - Changed API (different name, type, or mechanism)
   - Added API (WC-only features not in the reference)

### CEM (optional)

If the project uses Custom Elements Manifest tooling:

```bash
cem health --component my-element --format json
```

Flag any category below 80%.

---

## Phase 7: Audit

Prompt the user to run the companion review skills:

- **review-api** — API surface quality, code patterns, test coverage
- **review-a11y** — accessibility tree verification, keyboard, ARIA

Fix all critical and warning findings.

## Quality Bar

Visually and functionally correct. API should feel native to web
developers. Accessibility should meet or exceed any reference
implementation.
