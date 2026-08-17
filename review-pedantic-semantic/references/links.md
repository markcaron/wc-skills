# Link Text, Underlines, and Color

Detailed rules for hyperlink semantics, link text quality, underline
conventions, and color usage.

## Link Text

### Rule: no-click-here

Never use "click here", "here", "read more", "learn more", "info", or
other vague phrases as link text. This hurts three things at once:

1. **Usability**: Users scan pages in an F-pattern, foraging for
   keywords. Hyperlinks stand out visually and are critical to this
   information foraging. Vague link text has poor "information scent" —
   it forces users to read surrounding text to piece together context,
   increasing cognitive load.
2. **Accessibility**: Screen reader users navigate by pulling up a list
   of all links on a page. With vague links, they hear: *"click here"
   Link. "click here" Link. "Learn more" Link.* — no context at all.
3. **SEO**: Search engines are blind — they index pages much like screen
   readers. Link text keywords create relationships to destination URLs.
   "Click here" provides no keyword signal.

Also, "click" describes a mouse-specific mechanic. It makes no sense on
touch devices.

**Bad:**

```html
For documentation, <a href="/docs">click here</a>.
<a href="/blog/post">Read more</a>
<a href="/pricing">Learn more</a>
```

**Good:**

```html
Read our <a href="/docs">documentation</a>.
<a href="/blog/post">Read more about accessible web components</a>
<a href="/pricing">Pricing and plans</a>
```

Good link text should be:
- Descriptive and concise
- Start with a keyword
- Contain concrete nouns
- Not be a verb phrase
- Meaningful when read out of context

If visible text must be short, use a visually-hidden span:

```html
<a href="/blog/post">
  Read more
  <span class="visually-hidden"> about accessible web components</span>
</a>
```

### Rule: no-url-as-link-text

Don't use raw URLs as link text. They are not human-readable, they
increase cognitive load, and screen readers announce them character by
character: *"h-t-t-p-colon-forward-slash-forward-slash..."*

URLs as link text also provide no keyword signal to search engines.

**Bad:**

```html
Visit <a href="https://example.com/docs/getting-started">
  https://example.com/docs/getting-started</a>
```

**Good:**

```html
Visit the <a href="https://example.com/docs/getting-started">
  getting started guide</a>
```

### Rule: no-verb-link-text

Don't include action verbs ("click", "tap", "press") in link text.
Users know how to activate links. Focus on the destination, not the
mechanic.

### Rule: unique-link-text

Identical link text should go to identical destinations. Different
destinations need different link text. Three "Learn more" links going
to three different places is a failure.

### Rule: keep-link-text-brief

Link text should be concise. Don't hyperlink entire sentences or long
phrases — link the meaningful noun or noun phrase.

**Bad:**

```html
<a href="/docs">Read our documentation to learn about getting started
with the platform and configuring your environment</a>
```

**Good:**

```html
Read our <a href="/docs">documentation</a> to learn about getting
started with the platform.
```

### Rule: no-title-as-link-context

Don't rely on the `title` attribute to provide context for links. It's
not exposed by all browsers accessibly — keyboard and touch users will
never see it.

### Rule: no-forced-new-window

Avoid `target="_blank"` on links. Keep users in control of their own
experience — let them choose whether to open a link in a new tab.

Pointing to an external domain is **not** reason alone to force a new
window. Reserve `target="_blank"` for cases where losing the current
page state would genuinely harm the user (e.g. a link inside an
unsaved form). When you do use it, always pair with
`rel="noopener noreferrer"` and indicate the behavior visually (e.g.
an external-link icon) and accessibly:

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Example <span class="visually-hidden">(opens in a new tab)</span>
</a>
```

### Rule: visited-link-color

Provide a distinct `:visited` link color. Users rely on this to know
where they've already been. Not styling visited links is a
[top-10 usability mistake](https://www.nngroup.com/articles/top-10-mistakes-web-design/).

---

## Underlines and Color

### Rule: underlines-mean-links

On the web, underlined text signifies a link. Don't underline text that
is not a link. Use `<strong>` or `<em>` for emphasis instead.

### Rule: link-color-reserved

Avoid using your site's link color (typically blue) for non-link text.
Users will try to click it.

### Rule: inline-links-need-underline

Links within prose text (paragraphs, list items, headings, definition
lists) should be underlined. Color alone is insufficient to distinguish
links from surrounding text per WCAG 1.4.1 (Use of Color). If
underlines are removed and color is the only differentiator, the
contrast ratio between link color and surrounding text must be at least
3:1, and an underline or other non-color cue must appear on hover/focus.

### Rule: underline-exceptions

The following contexts are exempt from the underline requirement because
their position, visual treatment, or surrounding context already signals
interactivity:

- **Navigation elements** — menus, breadcrumbs, nav bars
- **Links with visual action cues** — e.g. calls to action with an
  arrow icon
- **Links not alongside non-link text** — e.g. a list of links inside
  a card where every item is a link

If in doubt, underline. The exemption applies only when the link's
context makes its interactive nature obvious without an underline.

### Rule: link-underline-interaction

Link underlines should respond to user interaction. At minimum,
underlines should become more prominent on `:hover` and `:focus` — for
example, transitioning from a dashed or subtle underline to a solid one,
or shifting the underline offset. This reinforces the link's
interactivity when the user engages with it.

### Rule: focus-styles-on-links

Never remove or suppress the `:focus` state on links. Keyboard users
depend on it. A focused link should show both the standard focus
indicator (outline/ring) and the same text/underline styles as
`:hover`. Style with `:focus-visible` but never remove focus styles
entirely.
