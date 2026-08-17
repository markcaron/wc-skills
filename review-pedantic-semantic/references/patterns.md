# APG Patterns, Navigation, and Composite Widgets

Detailed rules for dropdown disambiguation, navigation patterns,
and composite widget constraints.

## Dropdown Patterns

### Rule: menu-button-for-actions

A **[Menu Button](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/)**
opens a **[Menu](https://www.w3.org/WAI/ARIA/apg/patterns/menu/)** — a
list of *actions* the user can perform (Save, Delete, Copy, Print). It
does NOT select a value.

- Roles: `button` (trigger) → `menu` → `menuitem` / `menuitemcheckbox` /
  `menuitemradio`
- Keyboard: Enter/Space/Down opens; arrow keys navigate items; Enter
  activates; Escape closes
- `aria-haspopup="menu"` on the trigger

A Menu Button **cannot** be marked `aria-required`, has no form value,
and should never be used to choose from a set of options that
participates in form submission.

### Rule: combobox-for-selecting-values

The APG **[Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)**
pattern covers both "select" and "typeahead" use cases. A combobox is
an input widget with an associated popup from which the user selects a
value.

**Select-only combobox** (no text input — like a native `<select>`):

- APG example: [Select-Only Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/examples/combobox-select-only/)
- Roles: `combobox` (trigger) → `listbox` → `option`
- Keyboard: Enter/Space opens; arrow keys navigate options; Enter
  selects; Escape closes
- `aria-haspopup="listbox"` on the trigger
- Can be marked `aria-required`; has an accessible value
- The trigger displays the currently selected option

**Editable combobox** (text input for typeahead/filtering):

- APG example: [Editable Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/examples/combobox-autocomplete-list/)
- Roles: `combobox` (text input) → `listbox` → `option`
- Focus stays in the text input; `aria-activedescendant` tracks the
  highlighted option
- Use when the option count is large (~25+) or partial-match filtering
  benefits the user

Both variants are **Combobox** in APG terms. The difference is whether
the trigger accepts typed input.

### Rule: listbox-for-always-visible-lists

The APG **[Listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/)**
is a standalone, always-visible list of options — it has no trigger
button or popup. Use it when the list of options is permanently
displayed (e.g. a scrollable option panel). Don't confuse it with the
popup `listbox` inside a Combobox.

### Rule: menu-has-no-selected-state

If your "dropdown" displays a currently selected value on its trigger,
it's a **Combobox** (select-only), not a Menu Button. Menus don't have
selected values. Dynamically changing a Menu Button's label to reflect
a "chosen" item creates a false affordance — it looks like a Combobox
but behaves like a Menu.

### Rule: dont-mix-links-and-actions-in-menu

Do not mix navigation links and action buttons in a single
`role="menu"` container. The ARIA `menu` pattern is for application-
style actions (Save, Delete, Copy) — not navigation. Putting
`role="menuitem"` on an `<a>` overrides the implicit link role:

- Assistive tech announces "menu item" instead of "link"
- The link disappears from the screen reader's links list
- Users lose the cue that activating it will navigate away
- Right-click → open in new tab is no longer advertised

If your dropdown contains links, use the
**[Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure-show-hide/)**
pattern instead. Every link stays announced as a link, appears in the
SR links list, and Tab works normally. The entire JS requirement is
toggling `aria-expanded` and the `hidden` attribute.

**If a top-level item must both navigate AND open a submenu**, separate
the concerns: a link for the destination, plus a small adjacent button
(chevron) that toggles the submenu. Two elements, two roles, no
conflict. The APG has a dedicated example:
[Disclosure Navigation Menu with Top-Level Links](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-navigation-hybrid/).

```html
<li>
  <a href="/products">Products</a>
  <button aria-expanded="false" aria-controls="products-menu">
    <span class="visually-hidden">Products submenu</span>
    <svg aria-hidden="true"><!-- chevron --></svg>
  </button>
  <ul id="products-menu" hidden>
    <li><a href="/products/linux">Enterprise Linux</a></li>
    <li><a href="/products/openshift">OpenShift</a></li>
  </ul>
</li>
```

### Rule: no-hover-open-menus

Dropdown menus must open only by intentional activation — click,
Enter, or Space. Hover must not open, close, reveal, or dismiss
dropdown content. CSS-only `:hover` dropdowns lack state communication
(`aria-expanded`), Escape-to-close, and proper focus management.

---

## Navigation

### Rule: disclosure-nav-not-menubar

Site navigation with expandable sections should use the
**[Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure-show-hide/)**
pattern — not the
**[Menubar](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/)** pattern.

The APG itself warns against this. From the
[Navigation Menubar Example](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/examples/menubar-navigation/):

> The `menubar` pattern requires complex functionality that is
> unnecessary for typical site navigation that is styled to look like a
> menubar with expandable sections or fly outs. A pattern more suited
> for typical site navigation with expandable groups of links is the
> Disclosure Pattern.

**Why Menubar is wrong for site nav:**

- Menubar requires roving `tabindex`, arrow-key navigation between
  items, Home/End, and typeahead — keyboard behavior users don't
  expect from a nav bar
- It strips list semantics (`role="none"` on `<li>` elements) so
  screen readers lose the "list, N items" announcement
- Links become `role="menuitem"`, losing their link affordances
  (right-click, open in new tab may not work as expected)
- It's significantly more JavaScript to implement and test

**The Disclosure pattern** is simpler and keeps native semantics:
a `<button>` toggles visibility of a group of `<a>` links inside a
`<ul>`. No roving tabindex, no arrow-key contracts, no role overrides.

```html
<nav aria-label="Main">
  <ul>
    <li><a href="/">Home</a></li>
    <li>
      <button aria-expanded="false" aria-controls="about-menu">
        About
      </button>
      <ul id="about-menu" hidden>
        <li><a href="/about/overview">Overview</a></li>
        <li><a href="/about/team">Team</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

**Real-world examples of disclosure navigation:**

- [Disclosure Navigation Menu](https://codepen.io/markcaron/pen/ZEgNwoZ) —
  a vanilla implementation of the APG disclosure nav pattern
- [Red Hat Design System `rh-navigation-primary`](https://ux.redhat.com/elements/navigation-primary/) —
  a Web Component that uses the disclosure pattern for site-wide
  navigation with expandable mega-menu panels
- APG: [Example Disclosure Navigation Menu](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure-show-hide/examples/disclosure-navigation/)

Reserve the Menubar pattern for application-style menubars (e.g. a
text editor's File/Edit/View menu) where users expect arrow-key
navigation between top-level items.

---

## Composite Widgets

### Rule: no-actions-inside-tablist

A `tablist` must contain only `tab` elements. Do not put buttons,
links, popovers, or "add new" actions inside a `tablist` or inside a
`tab`. There are two reasons:

1. **`tab` has "children presentational: true"** — browsers strip
   semantics from any descendant elements inside a `tab`. A button
   nested inside a tab won't be exposed as a button in the a11y tree.
   Screen reader users cannot discover or activate it.
2. **`tablist` required owned elements** — axe-core flags non-`tab`
   interactive children as `aria-required-children` violations.

**Bad:**

```html
<div role="tablist">
  <button role="tab" aria-selected="true">Dashboard</button>
  <button role="tab">Settings</button>
  <button onclick="addTab()">+ Add tab</button> <!-- not a tab! -->
</div>
```

**Good:**

```html
<div role="tablist">
  <button role="tab" aria-selected="true" tabindex="0">Dashboard</button>
  <button role="tab" tabindex="-1">Settings</button>
</div>
<div role="tabpanel" tabindex="0">
  <!-- panel content -->
</div>
<button class="add-tab">+ Add tab</button>
```

Place secondary actions **after the entire tabset** (tablist +
tabpanel) in DOM order. Use CSS to visually position the button in
the tab bar if needed.

**Why after, not between?** The APG tabs keyboard contract states that
Tab from the tablist moves focus to "the tabpanel unless the first
element containing meaningful content inside the tabpanel is
focusable." Placing a focusable element between the tablist and
tabpanel breaks this intended flow. Placing it after the tabpanel
preserves the contract: **tablist → tabpanel → action button**.

This is an imperfect solution — the visual-DOM order may not perfectly
match — but it's the least disruptive to the expected keyboard
contract. There is currently no spec-compliant way to add secondary
actions to a tablist without some tradeoff.

**Note:** The W3C is developing an
[`aria-actions` attribute](https://github.com/w3c/aria/issues/1440)
to let composite widget items reference associated action buttons.
The APG has an
[experimental example](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/examples/tabs-actions/)
— but this is still a draft, not production-ready.
