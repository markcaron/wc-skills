---
name: review-api
description: >
  Review a Lit web component's API (HTML, CSS, JS) quality. Checks
  attributes, properties, events, slots, CSS parts, custom properties,
  code patterns, Lit conventions, and test coverage. Use when asked to
  "review api", "review code", "check element quality", "audit api",
  or before opening a PR.
tools: Read, Glob, Grep, Shell
---

# API Review

Review a Lit web component's HTML, CSS, and JS API quality against
ADVICE.md rules and Lit conventions.

## Workflow

### Phase 1: Read Element

Read:
- Element source (`.ts`)
- Element styles (`.css` or inline)
- All tests

Read `ADVICE.md` for reference rules.

### Phase 2: API Surface

#### Attributes & Properties
- [ ] Enum-style `variant` instead of multiple booleans?
- [ ] No `is-` prefix on boolean attributes?
- [ ] No reflected booleans defaulting to `true`?
- [ ] No reflected booleans as `"true"|"false"` strings?
- [ ] No reflected array/object properties?
- [ ] No `aria-*` exposed as public API?
- [ ] `accessible-label` attribute where `aria-label` is indicated?
- [ ] Multi-word attributes dash-case with camelCase fields?
- [ ] All attributes have sensible defaults?
- [ ] No native HTML attributes redeclared?
- [ ] Attribute/slot pairs where content could be plain or rich?
- [ ] No internal state exposed as public attributes?
- [ ] No light DOM classes required?

#### Slots
- [ ] Default slot for primary content?
- [ ] Semantic names, not positional?
- [ ] No placeholder "lorem ipsum" content?
- [ ] Default content provided where sensible?
- [ ] Prescriptive descriptions in JSDoc?
- [ ] Empty slot containers hidden (with SSR awareness)?
- [ ] Note: `:empty` doesn't work on slots; `::has-slotted` not yet
      widely available

#### Events
- [ ] Event subclasses, not `CustomEvent` with detail?
- [ ] Destructive/state-changing events cancelable?
- [ ] Native event names when wrapping native elements?

#### CSS Custom Properties
- [ ] `--_` prefix for private custom properties?
- [ ] Public properties as fallback defaults at use site, not on
      `:host`?
- [ ] No custom properties duplicating native CSS?
- [ ] Component-namespaced names for public properties?

#### CSS Parts
- [ ] Named after internal elements?
- [ ] Only exposed with clear use cases?

#### General
- [ ] Minimal API surface (easy to add, hard to remove)?
- [ ] No sub-elements unless a11y/composition requires?
- [ ] ECMAScript private fields (`#field`)?
- [ ] `override` on all methods that exist on the parent class (type
      safety check)? `super` called in standard CE lifecycle
      (`connectedCallback`, `disconnectedCallback`) and `update()`?
      `super` not needed for reactive hooks (`render`, `willUpdate`,
      `updated`, `firstUpdated`, `shouldUpdate`) when extending
      LitElement directly, but required when extending another
      component?
- [ ] Member ordering follows convention?

### Phase 3: Code Patterns

#### CSS
- [ ] Native nesting?
- [ ] `light-dark()` for color values?
- [ ] Logical properties (no `left`/`top`/`right`/`bottom`)?
- [ ] Baseline 2024 or earlier?
- [ ] No BEM classes? ID selectors for unique shadow elements?
- [ ] `box-sizing` reset?
- [ ] Token-derived fallback values (not guessed)?
- [ ] `pointer-events: none` on disabled host?
- [ ] `aspect-ratio` instead of width+height where applicable?
- [ ] GPU-friendly animations?
- [ ] Individual transform properties (`rotate`, `translate`, `scale`)?
- [ ] Shadow DOM animation names unprefixed?
- [ ] `:host` attribute selectors not nested with `&`?
- [ ] No `:host:has()` or `:host(:has())`?
- [ ] No `:host-context`?

#### Templates
- [ ] No BEM in shadow DOM?
- [ ] Minimal wrapper elements?
- [ ] `<slot>` used directly where possible?
- [ ] Vertical attribute formatting when >2?
- [ ] False case first in ternaries?

#### Lit Patterns
- [ ] No imperative DOM updates?
- [ ] Side effects in `willUpdate`, not `render`?
- [ ] `static styles = [styles]` array form?
- [ ] Controllers for cross-cutting concerns?
- [ ] `isServer` guards on browser-only APIs?
- [ ] Attribute values quoted in Lit templates `attr="${this.value}"`?
- [ ] No unnecessary template extraction into private render methods?
- [ ] No explicit `this` binding in template event listeners?

#### Lint (if available)

```bash
npx eslint <element-dir>/**/*.ts
npx stylelint <element-dir>/**/*.css
```

### Phase 4: Test Review

- [ ] Tests exist?
- [ ] Test public API, not shadow DOM internals?
- [ ] a11ySnapshot assertions present?
- [ ] Keyboard navigation tested?
- [ ] Form submission tested (if form-associated)?
- [ ] Setup in `beforeEach`, assertions in `it`?
- [ ] No arrow functions in test blocks?
- [ ] All component states covered (disabled, expanded, error, etc.)?

### Phase 5: JSDoc and Documentation

- [ ] `@summary` present on element class?
- [ ] All public properties/methods have `/** */` JSDoc (not `//`)?
- [ ] CSS custom properties documented with `/** */` comments in CSS,
      co-located with declaration or `var()` reference?
- [ ] CSS parts documented with HTML comments before the `part` element
      in template?
- [ ] Slots documented with HTML comments before the `<slot>` element
      in template?
- [ ] `@fires` for all dispatched events?
- [ ] No unnecessary `@default` tags (analyzer picks up initializers)?

Prefer HTML comments in template for part and slot descriptions, CSS
comments for `@cssprop`, over JSDoc on the class.

#### CEM health (optional)

If Custom Elements Manifest tooling is available:

```bash
cem health --component <element-name> --format json
```

Flag any category below 80%.

### Phase 6: Report

```markdown
## API Review: <element-name>

### Summary
- API: X issues (Y critical)
- Code: X pattern violations
- Tests: X gaps
- Docs: coverage assessment

### Critical Issues
[Must fix — API surface violations are critical because they're hard
to fix post-release]

### Warnings
[Should fix]

### Suggestions
[Nice to have]

### Intentional Divergences
[Where we chose different API/behavior than reference, and why]
```

## Principles

- Review against ADVICE.md rules, not taste
- API surface violations are critical (hard to fix post-release)
- Departures from a reference API are not necessarily problems, but
  must be surfaced to the user for human review
- Minimal API > comprehensive API
