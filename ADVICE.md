# Web Component API Design Advice

Distilled from 4800+ pull request reviews across two production Lit-based
design system projects in the
[PatternFly Elements](https://github.com/patternfly/patternfly-elements)
ecosystem. Forked from the
[original source](https://github.com/patternfly/patternfly-elements/blob/main/.claude/ADVICE.md).
Patterns have been generalized; project-specific naming and conventions
removed.

## HTML

### Attributes & Properties

#### Prefer enum-style `variant` attributes over multiple booleans
Consolidate mutually exclusive visual states into a single `variant`
attribute with an enum type rather than multiple boolean attributes.
Variants should be exclusive, not stackable.

#### Distinguish `variant` from theme attributes
`variant` (or `type`) is for functional/structural differences between
component configurations. Theme attributes are for color/visual
differences. Never conflate the two.

#### Boolean attributes for boolean states
When an attribute is yes/no (present/absent), use a boolean attribute,
not a string `"true"/"false"`. Use `@property({ type: Boolean })` with
a default of `false`.

#### Avoid `is-` prefix for boolean attributes
Since boolean attributes are true when present and false when absent,
the `is-` prefix adds noise without value. Pair dash-case attributes to
camelCase properties.

#### Don't default reflected booleans to `true`
A reflecting boolean attribute that defaults to `true` is problematic
because there is no good way to unset it in HTML. Flip the sense of the
attribute (e.g. `no-flip` instead of `flip` defaulting to true).

#### Avoid initializing reflected properties
Don't set defaults on reflected properties. Let absence be the default
state to avoid noisy attributes appearing on every instance when they
connect to the DOM.

#### Don't reflect array or object properties
Avoid accepting and certainly reflecting array or object properties. If
a list is needed, use a converter.

#### Don't sprout attributes
Avoid dynamically adding attributes that weren't present at creation
time.

#### Boolean-or-enum converter for multi-modal attributes
When an attribute has a default boolean behavior but also supports
specific modes, use a boolean-or-enum converter pattern.

#### Attributes should always have sensible defaults
All properties/attributes should have defaults. An element should work
correctly with zero attributes set.

#### Multi-word attributes use dash-case
Multi-word attribute names should be dash-case since HTML attributes are
case-insensitive. Property names should be camelCase.

#### Don't expose internal state as public attributes
Internal implementation details like breakpoint state, color context, or
responsive behavior should not be exposed as attributes. Use controllers
internally.

#### Don't expose `aria-*` as public API
Custom elements should expose their own semantic attributes (e.g.
`disabled`, `accessible-label`) and manage ARIA attributes internally
via ElementInternals.

#### Abstract ARIA behind custom attributes
Provide custom attributes for screen-reader names rather than exposing
ARIA attributes directly. Apply `aria-label` and `role` via
ElementInternals internally.

#### Provide attribute/slot pairs for content
For content that could be plain text or rich markup, provide both an
attribute (for simple use) and a named slot (for rich content). Demos
should primarily show the attribute form.

#### Internalize accessibility wiring via attribute/slot pairs
If content serves as an accessible description (helper text, error
text), make it an attr/slot pair so the component handles
`aria-describedby` automatically, removing that burden from users.

#### Normalize enumerated attribute values
Always normalize enumerated attribute values to lowercase internally, so
`state="neUtRal"` works as expected.

#### Remove dead attributes promptly
If an attribute has no effect (no CSS or JS references it), remove it
with a proper major version bump. Dead attributes mislead users.

#### Don't redeclare native HTML attributes
The `role` attribute already exists on all elements. Don't add a
`@property` for it.

#### Use container queries over manual compact/responsive variants
If a layout change is purely a response to available space, make it
automatic via container queries. Document how users can force the
narrower layout by constraining the container width.

#### Avoid boolean `has-*` attributes
Instead of boolean `has-*` attributes that signal "some CSS property is
set," prefer value-carrying attributes.

#### Prefer opt-in over opt-out for features
Reverse the sense of opt-out booleans so features are opt-in by default.

#### Avoid empty strings as public API values
Empty string values are unclear in a public API. Prefer explicit,
meaningful values.

#### Limit public custom elements per component
Prefer working within a single shadow root when possible. Only ship
multiple elements when accessibility or composition requires it.

### Slots

#### Default slot for primary content
The main content of a component goes in the unnamed (default) slot. Only
name slots for secondary/metadata content areas.

#### Use semantic slot names, not implementation-specific ones
Slot names should reflect semantic purpose. Prefer `actions` over
`action-groups`, `header` over `title`. Prefer brevity.

#### Use the DOM standard term "default slot"
The DOM standard uses the term "default slot", not "anonymous slot".

#### Require semantic HTML in slots
Header slots should accept `<h2>`–`<h6>` tags. Design slot APIs around
standard HTML semantics rather than requiring custom class names.

#### Don't provide placeholder content in slots
End users should never see strings like "Placeholder Description" on
pages where developers forgot to include required content. Issue console
warnings instead.

#### Be conservative about adding named slots
Consider alternatives (attributes, i18n controllers) when the number of
slots could grow unboundedly.

#### Use named slots with defaults for i18n
Allow translation of built-in UI text by providing named slots with
English default content.

#### Hide empty slot containers
When a slot has no slotted content, hide wrapper/container elements in
shadow DOM to avoid empty spacing, visual artifacts, or empty landmarks
in the accessibility tree. `::has-slotted` is not yet widely available;
a DOM-based query may be needed client-side. SSR scenarios require
`ssr-hint-has-slotted` attributes.

#### Don't add slots tightly coupled to a single child component
Prefer composable, general-purpose slots. If the existing API can
support the use case (even with CSS workarounds), prefer that over API
expansion.

#### Keep slot semantics strict
Don't mix unrelated content types in a slot designed for a specific
purpose.

#### Prefer text nodes and attributes over nested child elements
For simple content with metadata, use text nodes for content and
attributes for metadata.

#### Use prescriptive slot descriptions in JSDoc
Slot descriptions should tell users what to put there, not describe the
slot abstractly. e.g. `@slot - Label text` not `@slot - The label slot`.

### Shadow DOM & Templates

#### Minimize wrapper/container elements
If a class or attribute can be applied directly to a semantic element
(like `<article>`), don't add an extra `<div>` wrapper.

#### Prefer `<slot>` directly over wrapping `<div>`
Since slots have `display: contents`, slotted content participates in
the parent's layout. Only add wrapper divs when CSS layout truly
requires them.

#### Set display on slot elements instead of wrapping divs
When a slot is wrapped in a div purely for layout, try setting display
directly on the slot element.

#### Use ID selectors and tag selectors in shadow DOM CSS
Prefer `#id` and `tagname` selectors over `.class` in shadow roots.
Shadow DOM provides encapsulation, so IDs are safe and more semantic.

#### No BEM in shadow DOM
BEM conventions were designed for a global CSS paradigm. Shadow DOM
provides encapsulation, making BEM unnecessary and noisy. Use IDs for
unique elements.

#### No classes required in light DOM
Components should not require users to add specific CSS classes to light
DOM content.

#### Use `::slotted()` styles over light DOM stylesheets
When possible, style slotted content from shadow DOM using `::slotted()`
rather than requiring light DOM stylesheets.

#### Distinguish slotted vs fallback content in CSS
Regular selectors target fallback content; `::slotted()` targets slotted
content. Combine them to cover both cases.

#### No self-closing void elements
Follow the HTML spec strictly: `<col>`, `<br>`, `<img>` should not have
trailing slashes.

#### Static content belongs in HTML, not JavaScript
Static modal/dialog content should be in the HTML template, not created
dynamically in JavaScript.

## CSS

### Custom Properties & Parts

#### Private CSS custom properties use `--_` prefix
Internal/private CSS custom properties use `--_` prefix. Public/
customizable ones use component-namespaced names.

#### Don't create custom properties that duplicate native CSS
If users can achieve the same thing with `width`, `color`, etc., don't
add a component-specific custom property for it.

#### Less is more for CSS custom property APIs
Only expose custom properties when they provide value beyond native CSS.
You can always add them later.

#### Consider CSS custom properties before adding variant attributes
If a visual change is purely stylistic (e.g. transparent backgrounds), a
CSS custom property is lower-cost and more flexible than a new
attribute.

#### Question the need for CSS parts aggressively
Parts create API surface that must be maintained and can allow users to
break component internals. Only expose parts with clear use cases.

#### Public CSS properties should be declared as fallback defaults
Public properties should be provided as fallback defaults at the use
site, not set on `:host`. Setting on `:host` overrides user settings.

```css
/* WRONG: overrides user settings */
:host {
  --my-el-color: blue;
}

/* RIGHT: user settings take precedence */
#label {
  color: var(--my-el-color, blue);
}
```

#### Non-overridable CSS properties go on internal elements
CSS custom properties that are intentionally non-overridable should be
set on internal shadow DOM elements, not the host.

#### Use token names, not raw values, everywhere
In code, reviews, and design specs, always reference design token names
rather than pixel/hex values.

#### CSS docstrings describe token usage within the element
When documenting CSS custom properties, describe how the token is used
in this specific element, not the token's general purpose.

#### Use logical properties in CSS custom property names
Use `border-block-start` not `border-top` in custom property naming.

### Conventions

#### Always use CSS logical properties
Use `inline-start`/`block-start` instead of `left`/`top` to support RTL
and vertical writing modes.

#### Use `::after` double-colon syntax for pseudo-elements
Per CSS3 spec.

#### Animate GPU-friendly properties
Animate `transform`/`translate`/`opacity`, not `padding`/`margin`/
`width`.

#### Shadow DOM animation names don't need prefixes
CSS animation names within shadow DOM are scoped automatically. Keep
them short and readable.

#### Use modern CSS transform properties
Use individual transform properties (`rotate`, `translate`, `scale`)
instead of the combined `transform` function.

#### Use `aspect-ratio` instead of width+height
When width and height should maintain a ratio, use the `aspect-ratio`
property.

#### Use `pointer-events: none` on disabled hosts
Disabled elements should prevent interaction at the CSS level.

```css
:host([disabled]),
:host(:disabled) {
  pointer-events: none;
}
```

#### Use CSS grid over CSS table modes
CSS grid handles spanning (like `colspan`) better than CSS table display
modes.

#### Use native CSS nesting
Prefer native CSS nesting syntax over preprocessor nesting.

#### Don't nest `:host` attribute selectors with `&`
CSS nesting does not support `&([attr])` as a shorthand for
`:host([attr])`. The `&` in nested CSS represents the parent selector
as-is, but `:host` is a pseudo-class function, and `&([attr])` parses
as a function call, not an attribute selector appended to `:host`. Keep
`:host([variant="..."])` blocks flat at the top level. Nesting is valid
for descendant selectors within those blocks.

```css
/* WRONG: &([attr]) is invalid */
:host {
  &([variant="primary"]) { ... }
}

/* RIGHT: flat :host selectors, nested descendants */
:host([variant="primary"]) {
  --_color: blue;
}

:host([variant="primary"]) button {
  &:hover { --_color: darkblue; }
  &:active { --_color: navy; }
}
```

#### Use `display: contents` cautiously
`display: contents` on `:host` can cause issues. Consider
ResizeObserver with a private CSS var instead.

#### Use `light-dark()` for color values
Prefer `light-dark()` for theme-dependent colors. Set `color-scheme`
on `:host` or let it inherit.

#### Baseline 2024 or earlier
Use CSS features that are Baseline 2024 or earlier, but no later.

#### Don't use `:host:has()` or `:host(:has())`
These are not yet reliable cross-browser.

#### Never use `:host-context`
It does not exist in any shipping browser.

## JavaScript

### Events

#### Use custom `Event` subclasses, not `CustomEvent` with detail
Create typed Event subclasses instead of `new CustomEvent('name',
{ detail: {...} })`. Better TypeScript and runtime type safety.

```ts
export class DialogCloseEvent extends Event {
  reason: string;
  constructor(reason: string) {
    super('close', { bubbles: true, cancelable: true });
    this.reason = reason;
  }
}
```

#### Make destructive/state-changing events cancelable
Events for close, delete, copy, and similar actions should be cancelable
via `preventDefault()`. Demonstrate the cancelable pattern in demos.

#### Make action events mutable
Events for user-facing actions (like copy) should allow consumers to
modify the action data, not just cancel.

#### Prefer simple native Event objects when no data is needed
Use `new Event('ready')` rather than `CustomEvent` when the event
carries no data payload.

#### Match native event names when wrapping native elements
When wrapping `<dialog>` or similar, match native event names and
timing.

#### Don't bake analytics into components
Let analytics scripts add their own event listeners.

### Architecture

#### Prefer forking over inheriting from upstream base classes
Don't force inheritance when the downstream component has significantly
different needs. Extend `LitElement` directly for cleaner, simpler code.
Share controllers and helpers without class inheritance.

#### Prefer composition (controllers) over inheritance (base classes)
Controllers allow sharing behavior without class hierarchy.
Configuration is better than inheritance for shared behavior.

#### Elements should have single responsibility
If combining features creates accessibility issues, split them. Let
consuming patterns compose simple primitives.

#### Context propagation must work at arbitrary depth
Test context with grandchild and deeply nested scenarios, not just
parent-child.

#### Lazy-load optional dependencies
When a component optionally uses another component (e.g. an icon),
lazy-load that dependency only when the relevant attribute is set.

#### Don't import heavy token maps in client code
Use CSS custom properties and `getComputedStyle` to access values at
runtime, keeping bundle size small.

#### Move private statics to module scope
Private static methods should be module-scoped functions rather than
static class methods, to prevent users from seeing them on the
constructor.

#### SSR awareness in component code
DOM APIs like `getBoundingClientRect()`, `children`,
`assignedElements()` are not available during server-side rendering.
Guard DOM-dependent code with `isServer` checks.

#### Prefer declarative state over imperative
Prefer attribute-driven state over imperative state set via JS methods.

#### Use MutationObserver when slotchange is insufficient
`slotchange` won't fire when children are added/removed without slot
attributes changing. Use MutationObserver for dynamic content.

#### `slotchange` already bubbles — don't add redundant listeners
Don't manually add `slotchange` listeners on individual slots when you
can listen on the shadow root.

#### Be careful with `hostConnected`
`hostConnected` reinitializes state when elements are moved in the DOM.

#### Use single static event listeners instead of per-instance
For document-level event listeners, add a single static listener that
iterates over a private set of element instances. Per-instance listeners
are memory leaks.

#### State updates should be unidirectional
Don't set the same state in multiple places. State should flow in one
direction.

#### Expose optional protected hooks
When subclasses may need to hook into behavior, expose an optional
protected method rather than making methods protected by default.

#### Modules don't need to wait on `load`
ES modules are always deferred, so waiting for the `load` event is
unnecessary.

#### Components should not own features that belong to external code
If a feature belongs to consuming code, document it with demos but don't
build it into the component.

#### Ensure hard-coded strings are internationalizable
Don't bake locale-specific strings into component logic. Consider i18n
from the start.

#### Mark properties as private to avoid accidental API surface
Public properties are API surface that bumps the minor version. If a
property doesn't need to be public, make it private.

#### Consider whether something should be CSS API or HTML attribute API
Some features may be better expressed as CSS custom properties than HTML
attributes.

## Accessibility

### Manage ARIA internally
Elements should manage their own ARIA attributes internally rather than
requiring users to set them in markup.

### Derive accessible labels from slotted content
Automatically compute accessible labels from headline/heading slots.
Always provide an `accessible-label` attribute escape hatch.

### Compute ARIA state privately via context
ARIA state derived from DOM structure (`aria-posinset`, `aria-setsize`)
should be computed internally via context, not exposed as public
attributes.

### Use ElementInternals for ARIA semantics
Use ElementInternals to set `role`, `aria-label`, and heading levels
rather than forcing users to manage them.

### Use ElementInternals for form association
Use the ElementInternals API for form-associated custom elements rather
than wrapping or slotting native form elements. Call `setValue()` on the
internals object whenever the value changes.

### Avoid landmark pollution
Don't put `<article>`, `<header>`, `<footer>`, `<nav>` landmark
elements in shadow DOMs of components used in multiples. They create
excessive landmarks for screen reader users.

### Lean on the native platform
When adopting native elements like `<dialog>`, question any JavaScript
that reimplements what the platform already provides.

### Provide fallback content for computed-content elements
Elements that render computed content should accept the raw value as
text content for progressive enhancement / no-JS fallback.

### Use `<details>` for expandable content
Styled `<details>` elements give expandable/collapsible accessibility
for free, avoiding manual ARIA management.

### Use `aria-hidden="true"` on decorative shadow icons
Decorative icons in shadow DOM should always be hidden from the
accessibility tree.

### Don't add click listeners to icons
Icons are decorative or informational, not interactive. Click handlers
belong on buttons or other interactive elements.

### Consider cross-root ARIA limitations
When composing elements across shadow boundaries, cross-root ARIA
references don't work declaratively yet. ARIA IDL DOM properties
(Baseline 2025) can resolve some cross-root references imperatively.

### Don't set contradictory ARIA on the same internals
When a role like `role="none"` is set, don't also set contradictory
attributes like `aria-disabled` on the same internals object.

### Leave "why comments" on non-obvious ARIA decisions
Non-obvious accessibility decisions need comments explaining the
reasoning for future maintainers.

## API Design Philosophy

### API Surface

#### Easy to add, hard to remove
Be conservative with API surface. Every slot, part, property, and
attribute is API surface. Adding new features is a minor release;
removing them is a breaking change.

#### Additive API changes are minor versions, not patches
Adding a new public property, attribute, event, or slot is a minor
version bump, not a patch.

#### Don't add escape-hatch slots without concrete use cases
Override/escape-hatch slots that allow replacing entire shadow DOM
sections should only be added when there is a proven need.

#### Distinguish element-level from pattern-level features
Some visual treatments are achieved through composition and content
(pattern-level) rather than built-in attributes (element-level).

#### Composability over configuration
Features that compose multiple behaviors belong at the "pattern" level
rather than as built-in booleans.

#### Don't ship what you can't fully support
Limit scope to what can be properly themed/maintained.

#### Reuse existing components rather than creating new sub-elements
Before creating new sub-elements, check if existing components can serve
the purpose.

#### Backwards-compatible API evolution
When simplifying an element's API, always maintain backwards
compatibility with the existing approach.

#### Deprecation over removal for shipped APIs
Never remove a shipped public attribute without a deprecation period.
Add deprecation warnings (JSDoc and runtime console warnings).

#### Shipped attribute values are part of the public CSS API
Even when deprecating attribute values, keep them functional in CSS to
avoid breaking downstream selectors.

#### Provide deprecated aliases when renaming elements
When renaming elements, always provide backward-compatible aliases.

#### Breaking change communication
Changesets for breaking changes should include: (1) what changed,
(2) BEFORE/AFTER code migration examples, (3) rationale.

### Naming Conventions

#### Use "variant" not "variation"
Consistent terminology across all documentation and code.

#### Use `expanded` over `open` for collapsible semantics
For accordion-like collapsible elements, `expanded` is more conventional
than `open`.

#### Rename parts and public APIs to avoid implementation jargon
CSS parts and public API names should use user-facing terminology, not
internal concepts.

#### Controller names shouldn't reference underlying libraries
Name controllers for what they do, not what library they wrap.

## Documentation & Testing

### Every element needs `@summary` JSDoc
Required for the custom elements manifest.

### All public members need JSDoc
Think of the custom elements manifest as consumable by LLMs and tooling.
JSDoc blocks on base classes inherit to subclasses.

### Use `/** */` JSDoc blocks, not `//` comments, for public API
Public API documentation should use JSDoc blocks. Examples in JSDoc
should use the DOM API (`document.querySelector`), not browser console
(`$0`).

### Use CSS data types in `@cssprop` JSDoc
Use CSS data types where feasible and show direct values in `@default`.

### Don't use unnecessary `@default` JSDoc tags
The CEM analyzer picks up defaults from property initializers
automatically.

### Document all public CSS custom properties with `@cssprop`
Every public CSS custom property needs a descriptive comment.

### Document all CSS parts with `@csspart`
Every exposed CSS part needs a JSDoc `@csspart` annotation.

### Summaries and descriptions belong in inline JSDoc
Not in external data files. Co-locate documentation with code.

### Tests should verify public API, not shadow DOM internals
Shadow DOM is an implementation detail. Test via attributes, properties,
events, computed styles, offsetWidth, and accessibility tree snapshots.

### Use a11ySnapshot for accessibility assertions
Test accessibility by asserting against the accessibility tree, not by
querying shadow DOM structure.

### Separate setup from assertions in tests
Move fixture creation and actions into `beforeEach`. Keep `it` blocks
focused on assertions.

### Don't use arrow functions in Mocha tests
Mocha tests should use `function()` to allow `this.timeout()` and other
Mocha context features.

### Test element computed sizes, not internal CSS property values
Test observable behavior like `offsetWidth` rather than internal CSS
custom property values.

### Test form submission behavior
Wrap form-associated elements in forms, add submit listeners that do
`event.preventDefault()`, and verify values.

### Tests should assert observable behavior, not rendering
A test that only checks whether the element renders is not useful.

### Split demos into separate files per variant
Each demo file should show one variant or feature. Index demo is the
simplest possible usage.

### Inline demo dependencies
Demo files should be self-contained. Inline JavaScript imports.

### Avoid wrapper divs in demo HTML
They add noise and don't represent how users would actually use the
component.

### Make component scope abundantly clear in docs
When a component has a constrained scope, make that limitation explicit
at every documentation entry point.

### Changesets describe what changed for the user
Explain user-facing changes, not the development process. Include code
snippets.

## Lit Conventions

### Omit `type: String` and single-word `attribute` in decorators
`type: String` is the default in `@property()` and should be omitted.
The `attribute` option only needs specification when the attribute name
differs from the property name.

### Don't imperatively update shadow DOM — use reactive state
Don't imperatively add/remove classes or modify shadow DOM. Use reactive
properties and `classMap` in templates.

### State then effects
Methods should update state; side effects (events, class changes) should
be derived from state changes in lifecycle callbacks or render methods.

### Put false case first in template ternaries
Makes templates easier to scan: `${!condition ? '' : html\`content\`}`.

### Controllers for cross-cutting concerns
Use reactive controllers rather than manually managing state.

### Use `static styles = [styles]` array form
Even for single stylesheets, use array syntax so adding additional style
sources later is a minimal diff.

### Prefer `willUpdate` over mutation observers for property changes
Use Lit's reactive property system (`@property` + `willUpdate`) instead
of mutation observers.

### Keep render methods pure — side effects in `willUpdate`
Side-effect logic should move to `willUpdate`. The render method should
be a pure function of state.

### Prefer `firstUpdated()` over async `connectedCallback`
For initial DOM setup, use Lit's `firstUpdated()`.

### Prefer `updated()` over `connectedCallback` for property reactions
Lit controls CE lifecycle callbacks. Use `updated()` to react to
property changes.

### Don't break up templates into private render methods without reason
Keep templates inline unless there's a compelling reason to extract
them.

### Don't explicitly bind instance methods in Lit templates
Lit handles `this` binding for event listeners in templates.

### Use `styleMap` for dynamic inline styles
When dynamic inline styles need to override the CSS cascade, use Lit's
`styleMap` directive.

### Design controllers for `ReactiveControllerHost`, not just elements
Controllers may need to work with non-element hosts (e.g.
`useController` hook in lit/react).

### Ensure controllers are composable
Elements should be able to use multiple controllers together without
conflicts.

## Lit SSR

### Guard browser-only APIs with `isServer`
Code that accesses `window`, `document`, `MutationObserver`,
`IntersectionObserver`, `getComputedStyle()`, `getBoundingClientRect()`,
`children`, `assignedElements()`, or any other browser-only API will
crash during server render.

```ts
import { isServer } from 'lit';

override connectedCallback() {
  super.connectedCallback();
  if (!isServer) {
    this.#normalize();
  }
}
```

### Move DOM logic out of `connectedCallback`
Enabling Lit context SSR support requires components to run
`connectedCallback` on the server. All DOM-related logic must move into
`firstUpdated` or `updated`.

### Avoid hydration mismatches
Mismatches between server-rendered HTML and initial client render can
prevent components from ever updating on the client.

```ts
override updated() {
  if (!isServer && this.hasUpdated) {
    // Safe to run client-side-only logic
  }
}
```

### Replace template ternaries with `?hidden` for SSR
Conditional ternaries in templates can break hydration. Prefer `?hidden`
attribute binding over ternaries for conditional rendering.

### Use scalar Lit context values, not complex objects
When sharing state via Lit context during SSR, use multiple contexts
each carrying a single scalar value rather than a single context with a
complex object.

### Use SSR hint attributes for slot-dependent templates
Lit SSR cannot inspect slotted children. Components that alter their
template based on slotted content need `ssr-hint-has-slotted` attributes
on the host so the server render matches the client render.
