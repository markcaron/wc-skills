---
name: review-a11y
description: >
  Accessibility audit for Lit web components. Verifies computed
  accessibility tree across element states using automated scans,
  APG pattern conformance, keyboard navigation, and cross-root ARIA.
  Use when asked to "review accessibility", "check a11y",
  "audit accessibility", or mentions "WCAG", "screen reader",
  "keyboard navigation", or "ARIA".
tools: Read, Glob, Grep, Shell
---

# Accessibility Review

Review Lit web components for accessibility compliance. Three-layer
audit: automated scan, APG pattern conformance, and component-specific
verification of the computed accessibility tree.

## Prerequisites

- Dev server running with the component available
- Browser automation (Chrome MCP, Playwright, etc.) for ax tree
  inspection — if unavailable, guide the user through manual checks

## Workflow

### Phase 1: Gather Context

1. Read the element source (`.ts` and `.css`)
2. Read demos
3. Identify all element states: default, hover, focus, active, disabled,
   expanded, checked, loading, error, etc. — including sub-element states
4. Identify relevant [ARIA APG Patterns](https://w3.org/WAI/ARIA/apg/patterns/)

### Phase 2: Automated Scan

Run axe-core against the rendered component to catch machine-detectable
violations before manual review.

#### Browser-based

```js
import AxeBuilder from '@axe-core/playwright'; // or @axe-core/puppeteer

const results = await new AxeBuilder({ page })
  .include('my-element')
  .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'best-practice'])
  .analyze();
```

#### CLI quick-check

```bash
npx @axe-core/cli <url> --tags wcag2a,wcag2aa,best-practice
```

#### Severity guide

| Severity | Action |
|----------|--------|
| critical | Must fix — blocks assistive tech entirely |
| serious  | Must fix — significantly degrades UX |
| moderate | Should fix — noticeable impact |
| minor    | Nice to fix — edge-case or cosmetic |

### Phase 3: Check Internal ARIA

- ElementInternals used for role, aria-label, aria-checked, etc.?
- No `aria-*` attributes recommended for public API (abstracted behind
  custom attrs)?
- No contradictory ARIA on same internals (e.g. `role="none"` +
  `aria-disabled`)?

### Phase 4: Check Keyboard

Using browser automation or manual testing:

- Interactive elements keyboard-focusable?
- Logical tab order?
- Keyboard shortcuts match native equivalents?
- Focus trapped correctly in modals/dialogs?
- `:focus-visible` styles present (not `:focus`)?

### Phase 5: Check Semantics

- Landmark roles only where appropriate (no pollution from repeated
  instances)?
- Heading hierarchy preserved through slots?
- Form association via ElementInternals?
- Labels derived from slotted content with attribute escape hatch?

### Phase 6: Check Color Contrast

#### WCAG AA Minimums

| Content | Minimum ratio |
|---------|---------------|
| Normal text | 4.5:1 |
| Large text (18px+ or 14px+ bold) | 3:1 |
| UI components and icons | 3:1 |

#### Checking Contrast

Calculate relative luminance for each color, then:

```
ratio = (lighter + 0.05) / (darker + 0.05)
```

Check BOTH light and dark modes when using `light-dark()`.

### Phase 7: Check Demos

- All demos demonstrate accessible patterns (no bad advice)?
- No redundant ARIA roles on elements with implicit roles?
- No `<div>` used as a button or other interactive element?
- Color contrast passes in all demos?

### Phase 8: APG Pattern Conformance

For each interactive widget, identify the matching APG pattern and
verify **roles**, **states/properties**, and **keyboard interaction**.

| Pattern | Key roles / attrs | Required keyboard |
|---------|-------------------|-------------------|
| Accordion | `heading` + `button`; `aria-expanded`, `aria-controls` | Enter/Space toggle |
| Alert | `role="alert"` (implies `aria-live="assertive"`) | None |
| Breadcrumb | `<nav aria-label="Breadcrumb">`, `<ol>`, `aria-current="page"` | Standard links |
| Button | `<button>` or `role="button"` | Enter/Space activate |
| Checkbox | `role="checkbox"` + `aria-checked` | Space toggle |
| Combobox | `role="combobox"` + `aria-expanded` + `aria-controls` → listbox | Down opens; Up/Down navigate; Enter select; Escape close |
| Dialog | `role="dialog"` + `aria-modal="true"` + `aria-labelledby` | Tab cycles within; Escape closes |
| Disclosure | `<button aria-expanded>` controlling sibling | Enter/Space toggle |
| Grid | `role="grid"` > `row` > `gridcell` | Arrow keys move focus |
| Listbox | `role="listbox"` > `option`; `aria-selected` | Up/Down navigate |
| Menu | `role="menu"` > `menuitem` | Arrow keys navigate; Enter activate; Escape close |
| Menu Button | `<button aria-haspopup="true" aria-expanded>` | Enter/Space/Down open; Escape close |
| Radio Group | `role="radiogroup"` > `radio` + `aria-checked` | Arrow keys move selection |
| Slider | `role="slider"` + value attrs | Left/Right change; Home/End jump |
| Switch | `role="switch"` + `aria-checked` | Enter/Space toggle |
| Tabs | `tablist` > `tab` + `aria-selected` + `aria-controls`; `tabpanel` | Left/Right switch; Tab into panel |
| Toolbar | `role="toolbar"` + `aria-label` | Left/Right between items |
| Tooltip | `role="tooltip"` via `aria-describedby` | Escape dismiss |
| Tree View | `role="tree"` > `treeitem`; `aria-expanded` | Up/Down navigate; Right expand; Left collapse |

### Phase 9: Verify Accessibility Tree

**CRITICAL**: Verify the computed accessibility tree in at least one
browser. If two browsers are available (Chrome + Firefox), verify both —
browser engines compute the ax tree differently for shadow DOM and
ElementInternals.

For each demo, in each state the element supports:

1. Open demo in browser
2. Capture a11y snapshot (verbose/full tree)
3. Verify roles, names, states, and values are correct
4. For interactive elements, interact to transition states, then
   snapshot again

#### States to verify

For each state the element supports, verify the **ax tree** reflects it.
Don't check for specific ARIA attributes in the DOM — the implementation
may use ElementInternals, native semantics, or ARIA attributes. What
matters is the computed ax tree result.

| State | Ax tree expectation |
|-------|---------------------|
| Default | Correct role, accessible name, description |
| Disabled | Node shows disabled state, not focusable |
| Focused | Focus indicator visible, node shows focused |
| Expanded | Node shows expanded, controlled content visible in tree |
| Collapsed | Node shows collapsed, controlled content hidden from tree |
| Checked/unchecked | Node reflects checked/unchecked state |
| Selected | Node reflects selected state |
| Loading | Live region announces, or node shows busy state |
| Error/invalid | Node shows invalid, error description associated |
| Read-only | Node reflects read-only state |

Skip states that don't apply to the element.

#### Composite widget relationships

Composite widgets have internal relationships between child nodes.
Verify in the **ax tree**, not DOM attributes.

| Pattern | Expected roles | Expected relationships |
|---------|---------------|------------------------|
| Combobox/Select | combobox, listbox, option | Combobox owns listbox; focused option tracked; expanded toggles |
| Menu/Dropdown | button, menu, menuitem | Trigger controls menu; menu appears/disappears on toggle |
| Tabs | tablist, tab, tabpanel | Tab controls panel; active tab selected; inactive panels hidden |
| Accordion | heading, button, region | Button controls region; expanded toggles |
| Tree | tree, treeitem, group | Parents show expanded; items have level, setsize, posinset |
| Dialog | dialog | Modal state conveyed; focus contained; name from heading |
| Form controls | varies | Description associated; label associated |

For each composite element:
1. Snapshot collapsed/closed — verify hidden content absent from tree
2. Expand/open via interaction
3. Snapshot expanded/open — verify relationships resolved
4. Interact with children (select option, switch tab, etc.)
5. Snapshot post-interaction — verify state updates propagated
6. Close/collapse, snapshot — verify clean return

#### Cross-browser differences to flag

- Role computed differently
- Name computation differs (common with shadow DOM slotted content)
- State not reflected in one browser's ax tree
- Focus behavior differs
- ElementInternals not reflected in one browser
- Ownership/control relationships not resolved
- Native element semantics computed differently

### Phase 10: Report

```markdown
## Accessibility Review: <element-name>

### Automated Scan
| Severity | Rule ID | Element(s) | Issue | Fix |
|----------|---------|------------|-------|-----|
| ... | ... | ... | ... | ... |

### Accessibility Tree Verification
| State | Browser A | Browser B | Match? |
|-------|-----------|-----------|--------|
| Default | role=button, name="Save" | role=button, name="Save" | yes |

### APG Pattern Issues
| Widget | Expected pattern | Issue | Fix |
|--------|-----------------|-------|-----|
| ... | ... | ... | ... |

### Critical Issues
[Failures that break accessibility, linked to WCAG success criterion]

### Cross-Browser Differences
[Ax tree differences between browsers]

### Warnings
[Issues that may cause problems for some users]

### Recommendations
[Specific fixes with corrected code]
```

## Cross-Root ARIA Element Reflection

Reference material for reviewing cross-shadow-boundary ARIA
relationships.

### The Problem

Declarative ARIA ID references (`aria-describedby="some-id"`) only
resolve within a single DOM tree. Shadow DOM boundaries prevent
cross-root ID resolution.

### ARIA IDL Element Reflection

The ARIA IDL properties (`ariaDescribedByElements`,
`ariaLabelledByElements`, `ariaControlsElements`, etc.) accept Element
references instead of ID strings, bypassing the ID-resolution problem.
However, cross-root references are validated by a **shadow-including
ancestors** algorithm that restricts which directions work.

#### Which directions work

| Direction | Status |
|-----------|--------|
| Same tree (light → light) | Works |
| Shadow-to-light (child → parent) | Works |
| Light-to-own-shadow (parent → child) | Permitted by spec, unreliable in practice |
| Sibling shadow roots | Does NOT work |
| Arbitrary cross-root | Does NOT work |

#### Workarounds

| Relationship | Workaround |
|---|---|
| `describedby` | Static `role="status"` live-region announcer in light DOM |
| `labelledby` | Set `aria-label` directly on the trigger element |
| `controls` | Verify ax tree via native semantics or ElementInternals; document gaps |

### Reference Target Proposal

The **Reference Target** proposal is the long-term solution for
cross-root ARIA. It enables external elements to reference a shadow host
by ID, with the shadow root designating which internal element receives
the reference.

**Status (mid-2026):** WHATWG HTML Stage 3 ("Committed"). Chrome has
experimental support behind a flag. Firefox and Safari have prototypes
in progress. Not yet shipping in any stable browser.

#### What Reference Target solves

- Outside → host → internal target references
- Recursive resolution through nested shadow roots

#### What Reference Target does NOT solve

- Arbitrary sibling-to-sibling cross-root references
- Direct parent-to-specific-child references bypassing the host
- The tooltip case where the trigger is in light DOM and content is in
  the host's own shadow DOM (reverse direction)

### Reviewing elements with cross-root ARIA

1. **Identify all ARIA relationships** the element establishes
2. **Map each relationship's direction**: which element sets the
   property, which is referenced, which shadow roots are involved
3. **Check the ax tree** to see if the relationship actually resolves
4. **Verify the fallback mechanism** works
5. **Keep progressive enhancement code**: set IDL properties even when
   they silently fail today, with comments linking to spec issues
6. **Document the gap** in the review report

## Principles

- Verify the element's internal ARIA management
- Cross-root ARIA: document limitations, use IDL properties as
  progressive enhancement
- Leave "why comments" on non-obvious ARIA decisions
- Test every state, not just default
- Don't check DOM attributes — verify the computed ax tree
- Trust native element semantics (`<dialog>`, `<details>`) but verify
