# Answer Key

Expected violations per fixture file. Use this to verify the skill
catches what it should when run against the test fixtures.

## material-honesty.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 8 | use-elements-for-their-intended-purpose | Critical | `<div>` as button |
| 9 | use-elements-for-their-intended-purpose | Critical | `<span>` as link |
| 8–9 | unobtrusive-javascript | Warning | Inline `onclick` handlers |
| 12 | links-navigate-buttons-act | Critical | `<button>` used for navigation |
| 13 | links-navigate-buttons-act | Critical | `<a>` used for action (delete) |
| 14 | links-navigate-buttons-act | Critical | `<a>` without `href` |
| 17 | no-div-soup | Warning | `<div class="header">` → `<header>` |
| 19 | no-div-soup | Warning | `<div class="nav">` → `<nav>` |
| 19 | nav-is-a-list | Warning | Navigation is `<a>` tags in a `<div>`, not a `<ul>` |
| 24 | no-div-soup | Warning | `<div class="main">` → `<main>` |
| 25 | no-div-soup | Warning | `<div class="article">` → `<article>` |
| 26 | no-div-soup | Warning | `<div class="bold">` → `<strong>` |
| 27 | no-div-soup | Warning | `<div class="italic">` → `<em>` |
| 30 | no-div-soup | Warning | `<div class="footer">` → `<footer>` |
| 2 | lang-attribute | Critical | `<html>` missing `lang` |

## headings.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 10 | heading-hierarchy | Critical | `<h3>` after `<h1>`, skipped `<h2>` |
| 11 | heading-hierarchy | Critical | `<h5>` after `<h3>`, skipped `<h4>` |
| 14 | one-h1-per-page | Warning | Second `<h1>` on page |
| 17 | dont-style-headings-by-level | Nitpick | `<h4>` likely chosen for visual size |
| 20 | headings-not-divs | Warning | Styled `<div>` used as heading |
| 21 | headings-not-divs | Warning | Styled `<p>` used as heading |

## links.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 11 | no-click-here | Critical | "click here" link text |
| 12 | no-click-here | Warning | "Read more" link text |
| 13 | no-click-here | Warning | "Learn more" link text |
| 16–17 | no-url-as-link-text | Critical | Raw URL as link text |
| 20 | no-verb-link-text | Nitpick | "Click to see" in link text |
| 23–25 | unique-link-text | Warning | Three "Learn more" links to different destinations |
| 28–29 | keep-link-text-brief | Nitpick | Entire sentence hyperlinked |
| 32 | no-forced-new-window | Warning | `target="_blank"` without `rel` or accessible indication |
| 35 | underlines-mean-links | Nitpick | `<u>` used for emphasis, not a link |
| 38 | link-color-reserved | Nitpick | Blue text that isn't a link |
| 41–42 | inline-links-need-underline | Warning | Inline link with underline removed |

## tables.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 11–15 | tables-for-tabular-data-only | Warning | Simple key-value data; use `<dl>` instead |
| 18–29 | tables-need-structure | Critical | Missing `<caption>`, `<thead>`, `<th scope>` |
| 32 | never-change-table-display | Critical | `display: grid` on `<table>` strips semantics |
| 39–44 | no-layout-tables | Critical | Table used for page layout |

## aria-misuse.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 10 | prefer-native-html | Critical | `<div role="button">` instead of `<button>` |
| 11–13 | prefer-native-html | Warning | `<div role="navigation">` instead of `<nav>` |
| 14 | prefer-native-html | Critical | `<span role="link">` instead of `<a>` |
| 14 | unobtrusive-javascript | Warning | Inline `onclick` handler |
| 17 | no-redundant-roles | Nitpick | `role="navigation"` on `<nav>` |
| 18 | no-redundant-roles | Nitpick | `role="list"` on `<ul>` |
| 19 | no-redundant-roles | Nitpick | `role="link"` on `<a>` |
| 22 | no-redundant-roles | Nitpick | `role="main"` on `<main>` |
| 23 | no-redundant-roles | Nitpick | `role="button"` on `<button>` |
| 27–30 | no-aria-for-visibility | Critical | `aria-hidden` with focusable children |
| 33 | no-aria-on-non-interactive | Critical | `role="checkbox"` on `<div>` without behavior |
| 34 | no-aria-on-non-interactive | Critical | `role="slider"` on `<div>` without behavior |
| 37 | dont-override-native-semantics | Critical | `<h2 role="button">` |
| 38 | dont-override-native-semantics | Warning | `<a role="button">` |
| 39–43 | dont-override-native-semantics | Critical | `role="presentation"` on data table |

## forms-and-images.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 11 | labels-required | Critical | Input with no label (placeholder only) |
| 14–16 | label-is-not-a-group-label | Critical | `<label>` used as group heading for radios |
| 19–23 | fieldset-for-groups | Warning | `<div>` + `<h2>` instead of `<fieldset>` + `<legend>` |
| 27–28 | button-type-explicit | Warning | `<button>` without `type` in a `<form>` |
| 32 | meaningful-images-need-alt | Critical | `<img>` missing `alt` attribute |
| 35 | decorative-images-hidden | Nitpick | Decorative image has descriptive alt text; should be `alt=""` |
| 38–39 | no-icon-fonts | Warning | Icon fonts instead of `<svg>` |
| 42 | functional-images-describe-action | Nitpick | Logo link alt says "Company logo" not "Homepage" |

## typography.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 11 | hyphens-en-dashes-em-dashes | Nitpick | Hyphens for ranges (should be en dashes) |
| 12 | hyphens-en-dashes-em-dashes | Nitpick | Hyphens for parenthetical (should be em dashes) |
| 13 | hyphens-en-dashes-em-dashes | Nitpick | Hyphen for page range |
| 16–17 | smart-quotes | Nitpick | Straight quotes in prose |
| 20–21 | proper-ellipsis | Nitpick | Three periods instead of `…` |
| 24–25 | multiplication-not-x | Nitpick | Letter "x" for dimensions |
| 28 | minus-not-hyphen | Nitpick | Hyphen in math expression |
| 31–32 | no-fake-fractions | Nitpick | "1/2" and "3/4" instead of ½ and ¾ |
| 35 | no-underline-for-emphasis | Nitpick | `<u>` for emphasis |
| 38 | no-all-caps-for-emphasis | Nitpick | All-caps via CSS for emphasis |
| 41 | strong-not-b-em-not-i | Nitpick | `<b>` and `<i>` instead of `<strong>` and `<em>` |

## miscellaneous.html

| Line | Rule | Severity | What's wrong |
|------|------|----------|-------------|
| 3 | lang-attribute | Critical | `<html>` missing `lang` |
| 6 | page-title | Warning | Vague title "Page" |
| 13–15 | no-tabindex-positive | Critical | Positive `tabindex` values |
| — | skip-links | Warning | No skip navigation link |
| 20–24 | dom-order-matches-visual-order | Warning | `flex-direction: row-reverse` reverses visual order |
| 27 | iframe-needs-title | Critical | `<iframe>` without `title` |
| 30–32 | svg-accessibility | Warning | Meaningful SVG without `role="img"` or `<title>` |
| 35–37 | hidden-content-patterns | Critical | `aria-hidden` with focusable button inside |
| 40 | use-time-element | Nitpick | Date without `<time>` element |
| 43–46 | use-blockquote-q-cite | Nitpick | `<div class="quote">` instead of `<blockquote>` |
| 45 | use-blockquote-q-cite | Nitpick | Attribution inside the quote block |
| 49 | use-dialog | Warning | Custom modal with `<div role="dialog">` instead of `<dialog>` |
| 57 | unobtrusive-javascript | Warning | Inline `onclick` handler |
| 58 | unobtrusive-javascript | Warning | Inline `onmouseover` handler |
| 63 | content-presentation-behavior | Nitpick | JS setting `display: none` when `hidden` attribute works |
