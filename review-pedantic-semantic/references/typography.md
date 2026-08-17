# Typography, Punctuation, and Emphasis

Detailed rules for typographic correctness in prose content.

## Emphasis and Formatting

### Rule: no-underline-for-emphasis

On the web, underlines mean links. Use `<strong>` (bold) or `<em>`
(italic) for emphasis.

### Rule: no-all-caps-for-emphasis

Don't use all-caps for emphasis — it's harder to read, especially for
users with dyslexia. Use `<strong>` or `<em>`. Reserve all-caps for
acronyms and abbreviations, and apply via `text-transform: uppercase`
in CSS if a design requires it.

### Rule: strong-not-b-em-not-i

`<strong>` and `<em>` carry semantic meaning (importance, stress
emphasis). `<b>` and `<i>` are presentational. Prefer the semantic
elements.

---

## Punctuation

### Rule: hyphens-en-dashes-em-dashes

These are three different characters with three different jobs. Do not
interchange them.

| Character | Glyph | HTML entity | Use |
|-----------|-------|-------------|-----|
| Hyphen | - | `-` | Compound words, hyphenated adjectives: "well-known", "re-enter" |
| En dash | – | `&ndash;` | Ranges: "2020–2024", "pages 1–10", "Mon–Fri"; scores: "3–1" |
| Em dash | — | `&mdash;` | Parenthetical asides, interjections — like this — in prose |

**Common mistakes:**

```html
<!-- Hyphens used as dashes -->
<p>Open 9:00-5:00, Monday-Friday</p>
<p>The project - which started in 2020 - is complete.</p>

<!-- Correct -->
<p>Open 9:00–5:00, Monday–Friday</p>
<p>The project — which started in 2020 — is complete.</p>
```

**Em dash style:** No spaces around em dashes (Chicago/most style
guides). Some house styles use spaced en dashes instead — pick one
and be consistent.

**In code and UI:** Hyphens are fine in code identifiers, CSS
properties, CLI flags, etc. This rule applies to prose content, labels,
descriptions, and documentation.

### Rule: smart-quotes

Use proper curly quotation marks and apostrophes in prose content, not
straight/typewriter quotes.

| Character | Glyph | HTML entity | Use |
|-----------|-------|-------------|-----|
| Left double quote | " | `&ldquo;` | Opening quote |
| Right double quote | " | `&rdquo;` | Closing quote |
| Left single quote | ' | `&lsquo;` | Opening single quote |
| Right single quote / apostrophe | ' | `&rsquo;` | Closing single quote, apostrophes |

**Common mistakes:**

```html
<!-- Straight quotes -->
<p>She said "it's fine."</p>

<!-- Correct -->
<p>She said &ldquo;it&rsquo;s fine.&rdquo;</p>
```

**In code:** Straight quotes are correct inside `<code>`, `<pre>`, and
code blocks. This rule applies to prose content only.

### Rule: proper-ellipsis

Use the ellipsis character `…` (`&hellip;`), not three periods `...`.
Three periods are three separate characters with incorrect spacing.

### Rule: multiplication-not-x

Use the multiplication sign `×` (`&times;`) for dimensions, not the
letter "x". Example: "1920 × 1080", not "1920 x 1080". The letter "x"
is acceptable in casual/code contexts like "2x" for resolution.

### Rule: minus-not-hyphen

In mathematical or numeric contexts, use the minus sign `−` (`&minus;`)
rather than a hyphen. A hyphen is shorter and sits at a different
vertical position: "5 − 3 = 2", not "5 - 3 = 2".

### Rule: no-fake-fractions

Use proper fraction characters when available (½ ⅓ ¼ ⅔ ¾) or the
`<sup>`/`<sub>` pattern. Don't use "1/2" in prose when "½" is meant.

### Rule: spaces-around-slashes

In prose, use "and/or" (no spaces) for tight constructions, but
"design / development" (spaces) when separating longer terms for
readability. Be consistent.
