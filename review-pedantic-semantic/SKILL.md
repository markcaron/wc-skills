---
name: review-pedantic-semantic
description: >
  Pedantic review of HTML semantics, material honesty, and web standards
  compliance. Nitpicks element choice, heading order, link text, links
  vs buttons, dropdown patterns, table usage, and UX affordances. Use
  when asked to "check semantics", "review HTML", "audit markup",
  "nitpick", or when reviewing demos, pages, or slotted content for
  standards correctness.
tools: Read, Glob, Grep, Shell
---

# Pedantic Semantic Review

A nitpicky review of HTML for semantic correctness, material honesty,
and web standards compliance by [Mark Caron](https://markcaron.me).
This skill is opinionated — it enforces the principle that **the right
element for the job is the only element for the job**.

Grounded in Jeremy Keith's [Resilient Web Design](https://resilientwebdesign.com/)
principle of material honesty:

> One material should not be used as a substitute for another, otherwise
> the end result is deceptive.

## Workflow

### Phase 1: Gather

1. Read the target HTML — a page, a demo file, a component template, or
   slotted content examples
2. Identify every HTML element, its role, and its context
3. Note any custom elements and check what they render internally,
   including whether they use `ElementInternals` for added semantic
   meaning (role, accessible name, form association, states)

### Phase 2: Audit

Work through every rule section below. Flag violations with the rule
name and a corrected code example. When a section points to a reference
file, read it for the full rule details and examples.

### Phase 3: Report

```markdown
## Pedantic Semantic Review: <target>

### Critical
[Violations that break semantics, a11y, or user expectations]

### Warnings
[Material dishonesty, suboptimal element choices]

### Nitpicks
[Conventions, best practices, refinements]
```

---

## Rules

### Material Honesty

#### Rule: use-elements-for-their-intended-purpose

Every HTML element carries meaning. Using an element for something other
than its intended purpose is materially dishonest — it creates a façade
that crumbles under assistive technology, keyboard navigation, or
unexpected conditions.

**Bad:**

```html
<div class="button" onclick="save()">Save</div>
<span class="link" onclick="location.href='/about'">About</span>
```

**Good:**

```html
<button type="button" onclick="save()">Save</button>
<a href="/about">About</a>
```

#### Rule: links-navigate-buttons-act

Links navigate. Buttons act. Do not interchange them.

| Type | Element | Purpose | Keyboard | Announced as |
|------|---------|---------|----------|--------------|
| **Link** | `<a href>` | Navigate to a URL | Enter | "link" |
| **Button** | `<button type="button">` | Perform an action | Enter + Space | "button" |
| **Submit** | `<button type="submit">` | Submit a form | Enter + Space | "button" |
| **CTA** | `<a href>` (styled) | Navigate — still a link | Enter | "link" |

Links give users affordances buttons cannot: open in new tab, copy URL,
bookmark. CTAs are links, not buttons — style them distinctly (arrow
icon, whitespace) but keep them as `<a>` elements. Submit buttons
should NOT use `cursor: pointer` — reserve the hand cursor for links.

> Consistency is not about making different things the same. It's about
> making the same things the same. — Adam Silver

#### Rule: no-div-soup

Don't use `<div>` or `<span>` when a semantic element exists.

| Instead of | Use |
|------------|-----|
| `<div class="header">` | `<header>` |
| `<div class="nav">` | `<nav>` |
| `<div class="main">` | `<main>` |
| `<div class="footer">` | `<footer>` |
| `<div class="sidebar">` | `<aside>` |
| `<div class="section">` | `<section>` (with a heading) |
| `<div class="article">` | `<article>` |
| `<div class="list">` + `<div>` children | `<ul>` / `<ol>` + `<li>` |
| `<div class="dialog">` | `<dialog>` |
| `<div class="details">` | `<details>` + `<summary>` |

---

### Headings

- **heading-hierarchy** — Never skip heading levels when incrementing.
- **one-h1-per-page** — Exactly one `<h1>` per page.
- **dont-style-headings-by-level** — Choose level by structure, style
  with CSS. Don't pick `<h4>` because the default size "looks right."
- **headings-not-divs** — Don't use styled `<div>` or `<p>` tags as
  headings. Screen reader users navigate by heading.

---

### Links

Rules: **no-click-here**, **no-url-as-link-text**, **no-verb-link-text**,
**unique-link-text**, **keep-link-text-brief**, **no-title-as-link-context**,
**no-forced-new-window**, **visited-link-color**, **underlines-mean-links**,
**link-color-reserved**, **inline-links-need-underline**,
**underline-exceptions**, **link-underline-interaction**,
**focus-styles-on-links**

Key principles:

- Link text must make sense out of context (screen readers list links)
- "Click here", "read more", raw URLs — all failures
- Inline links need underlines (WCAG 1.4.1); nav/CTA exempt
- Don't force `target="_blank"` — external domain isn't reason enough
- Provide `:visited` color and never remove `:focus` styles

For full rules, examples, and rationale, see
[references/links.md](references/links.md).

---

### Tables

- **tables-for-tabular-data-only** — Before using a table, ask: (1)
  common attribute across rows *and* columns? (2) Could a list or `<dl>`
  convey the same meaning? If yes to #2, don't use a table.
- **tables-need-structure** — `<caption>`, `<thead>`/`<tbody>`,
  `<th scope>`, and `headers` for complex tables.
- **never-change-table-display** — `display: grid/flex/block` on table
  elements strips table semantics from the a11y tree.
- **no-layout-tables** — Tables must never be used for layout.
- **responsive-table-awareness** — Factor mobile behavior into the
  "should this be a table?" decision.

---

### "Dropdowns" and APG Patterns

The word "dropdown" doesn't appear in the
[APG](https://www.w3.org/WAI/ARIA/apg/patterns/). Use the pattern name:

| Purpose | APG pattern | Wrong name |
|---|---|---|
| Actions from a button | [Menu Button](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/) | Dropdown |
| Choosing a value | [Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/) (select-only) | Dropdown |
| Typeahead filtering | [Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/) (editable) | Dropdown |
| Always-visible options | [Listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/) | Options list |
| Show/hide content | [Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure-show-hide/) | Dropdown |

If "dropdown" must appear, always prefix with the type: "menu dropdown",
"select dropdown." Never use "dropdown" unqualified.

Rules: **menu-button-for-actions**, **combobox-for-selecting-values**,
**listbox-for-always-visible-lists**, **use-apg-pattern-names**,
**menu-has-no-selected-state**, **dont-mix-links-and-actions-in-menu**,
**no-hover-open-menus**, **no-actions-inside-tablist**,
**disclosure-nav-not-menubar**

Key principles:

- Menu Buttons are for actions, not value selection (no form value)
- Don't put `role="menuitem"` on links — it kills link semantics
- Menus don't have selected values — that's a Combobox
- Site nav should use Disclosure, not Menubar
- Don't nest non-`tab` elements in a `tablist` (children presentational)
- Dropdowns must open by activation, not hover

For full rules, examples, and rationale, see
[references/patterns.md](references/patterns.md).

---

### ARIA: Less Is More

> No ARIA is better than bad ARIA.
> — [W3C WAI](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/)

- **apg-is-not-a-pattern-library** — The APG was created to demonstrate
  ARIA's capabilities, not as a source of truth for implementation. Its
  code examples disproportionately favor ARIA over native HTML.
  Reference the APG for **pattern names** and **keyboard contracts** —
  not for code to copy. The [WebAIM Million](https://webaim.org/projects/million/)
  found that more ARIA correlates with more detected errors.
  ([Bailey](https://ericwbailey.website/published/heres-how-to-instruct-a-llm-to-reference-the-aria-authoring-practices-guide/),
  [Bushell](https://dbushell.com/2026/06/26/aria-anti-patterns-and-you/))
- **prefer-native-html** — Use `<button>`, `<nav>`, `<a>` instead of
  ARIA-decorated `<div>`s. Native elements give you keyboard, focus,
  and a11y for free; ARIA gives you only the tree label.
- **no-redundant-roles** — Don't add `role="navigation"` to `<nav>`.
  (Exception: `role="list"` on `<ul>` with `list-style: none` in Safari.)
- **no-aria-for-visibility** — `aria-hidden` doesn't prevent Tab focus.
  Use `hidden` or `display: none` to hide from everyone.
- **no-aria-on-non-interactive** — `role="checkbox"` on a `<div>`
  without keyboard/focus behavior is worse than no ARIA.
- **dont-override-native-semantics** — Don't make an `<h2>` into
  `role="button"`. Use the right element.

For full rules and code examples, see
[references/aria.md](references/aria.md).

---

### Lists

- **use-list-elements** — Use `<ul>`, `<ol>`, `<dl>` — not `<div>` soup.
- **nav-is-a-list** — Navigation: `<nav>` > `<ul>` > `<li>` > `<a>`.
- **disclosure-nav-not-menubar** — Site nav with expandable sections
  should use Disclosure, not Menubar. See
  [references/patterns.md](references/patterns.md).

---

### Landmarks

- **limited-landmarks** — Use landmarks purposefully. One `<main>`,
  label `<nav>` elements, don't repeat `<header>`/`<footer>` in cards.
- **section-needs-heading** — `<section>` without a heading is a `<div>`.

---

### Forms

- **labels-required** — Every control needs a `<label>`. Prefer real
  `<label>` over `aria-label` (it enlarges the click target).
- **label-is-not-a-group-label** — `<label>` is for one control.
  Group labels are `<fieldset>` + `<legend>`.
- **fieldset-for-groups** — Radio groups, checkbox groups → `<fieldset>`.
- **button-type-explicit** — Always specify `type` on `<button>`.
  Default is `submit` inside forms.

---

### Images and Icons

- **meaningful-images-need-alt** — Write alt based on context/function.
- **decorative-images-hidden** — `alt=""` for `<img>`,
  `aria-hidden="true"` for inline `<svg>`.
- **no-icon-fonts** — Use `<svg>` or `<img>` instead.
- **functional-images-describe-action** — Image in a link? Alt text
  describes *where it goes*, not what it looks like.

---

### Typography, Punctuation, and Emphasis

- **hyphens-en-dashes-em-dashes** — Three characters, three jobs:

| Character | Glyph | Use |
|-----------|-------|-----|
| Hyphen | - | Compound words: "well-known" |
| En dash | – | Ranges: "2020–2024", "Mon–Fri" |
| Em dash | — | Parenthetical asides — like this |

- **no-underline-for-emphasis** — Underlines mean links on the web.
- **no-all-caps-for-emphasis** — Use `<strong>` or `<em>` instead.
- **strong-not-b-em-not-i** — Prefer semantic elements over presentational.

Rules: **smart-quotes**, **proper-ellipsis**, **multiplication-not-x**,
**minus-not-hyphen**, **no-fake-fractions**, **spaces-around-slashes**

For full rules, entity tables, and code examples, see
[references/typography.md](references/typography.md).

---

### Semantic Elements People Forget

- **use-time-element** — Wrap dates/times in `<time datetime="...">`.
  Without it, "November 14, 2016" is just text to machines.
- **use-abbr** — First use of an abbreviation: `<abbr title="Web
  Content Accessibility Guidelines">WCAG</abbr>`.
- **use-blockquote-q-cite** — `<blockquote>` for block quotes, `<q>`
  for inline quotes, `<cite>` for source titles. Attribution goes
  outside `<blockquote>`, not inside.
- **use-dialog** — Use `<dialog>` for modals, not `<div>` + ARIA +
  focus-trap JS. It gives you modal behavior, Escape, focus trapping,
  and backdrop `inert` for free.
- **use-code-kbd-samp-var** — `<code>` for code, `<kbd>` for user
  input, `<samp>` for output, `<var>` for variables. Not
  `<span class="code">`.
- **use-address** — `<address>` for contact info related to the
  nearest `<article>` or `<body>`. Not for arbitrary postal addresses.
- **use-del-ins** — `<del>` and `<ins>` for edited content. Don't rely
  on strikethrough styling alone — screen readers ignore CSS.
- **use-mark** — `<mark>` for highlighted/relevant text, not
  `<span class="highlight">`.

---

### Miscellaneous

- **lang-attribute** — `<html>` must have `lang`. Use `lang` on
  containers for content in a different language.
- **page-title** — Unique `<title>` with unique content first:
  "Pricing | Acme", not "Acme | Pricing".
- **no-tabindex-positive** — Never. Use `0` or `-1` only.
- **no-autoplaying-media** — Must have pause/stop controls.
- **focus-visible-not-removed** — Never remove focus outlines without
  an equivalent indicator.
- **skip-links** — Pages with repeated nav blocks need a "Skip to main
  content" link as the first focusable element.
- **dom-order-matches-visual-order** — CSS reordering (`order`,
  `flex-direction: row-reverse`, grid placement) breaks Tab order.
  DOM sequence must match visual sequence (WCAG 1.3.2).
- **iframe-needs-title** — Every `<iframe>` must have a descriptive
  `title` so screen readers announce its purpose.
- **svg-accessibility** — Meaningful inline SVGs need `role="img"` +
  `<title>` (and optionally `<desc>`). Decorative SVGs get
  `aria-hidden="true"`.
- **hidden-content-patterns** — Know the three patterns: `hidden` /
  `display:none` (hidden from everyone), `aria-hidden="true"` (AT
  only), visually-hidden CSS class (visible to AT, hidden visually).
  Using the wrong one is a common mistake.

---

## Applying to Web Components

When reviewing a Lit component's `render()` template:

- Shadow DOM templates follow all the same semantic rules
- Slots should encourage semantic content (e.g. `<h2>`–`<h6>` in a
  heading slot, not `<div>`)
- Demos must model correct usage — they're copy-pasted as documentation
- Custom elements wrapping native behavior (links, buttons, dialogs)
  must preserve native semantics and affordances
- Prefer `ElementInternals` for role, accessible name, form
  association, and custom states. See
  [references/aria.md](references/aria.md).

## Principles

- **Structural honesty**: use the right element for the job
- **Material honesty**: don't make one thing look like another
- **Progressive enhancement**: content works without CSS or JS
- **Fault tolerance**: HTML and CSS fail silently; lean into this
- **Resilience**: a solid semantic foundation survives redesigns,
  migrations, and assistive tech updates

> Put your users first, and search engines will love your site.

## Sources

- Jeremy Keith, [Resilient Web Design](https://resilientwebdesign.com/)
- Mark Caron, [Don't use "click here"](https://heyoka.medium.com/dont-use-click-here-f32f445d1021)
- Mark Caron, [Let's bring \<table\> to the table, again](https://heyoka.medium.com/lets-bring-table-to-the-table-again-f1ae751159d5)
- Adam Silver, [But sometimes links look like buttons (and buttons look like links)](https://medium.com/simple-human/but-sometimes-links-look-like-buttons-and-buttons-look-like-links-9b371c57b3d2)
- Nathan Curtis, [Buttons in Design Systems](https://medium.com/eightshapes-llc/buttons-in-design-systems-eac3acf7e23)
- [W3C ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) (pattern names and keyboard contracts only — [not code examples](https://ericwbailey.website/published/heres-how-to-instruct-a-llm-to-reference-the-aria-authoring-practices-guide/))
- [Red Hat Design System — Accessibility](https://ux.redhat.com/accessibility/)
- [Red Hat Design System — Links](https://ux.redhat.com/foundations/interactions/links/)
- [Red Hat Design System — Accessible Tables](https://ux.redhat.com/accessibility/content/#accessible-tables)
- [Red Hat Design System — Table Accessibility](https://ux.redhat.com/elements/table/accessibility/)
