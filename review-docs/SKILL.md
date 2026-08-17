---
name: review-docs
description: >
  Review a Lit web component's JSDoc, CEM documentation, and inline
  comments for completeness and quality. Checks @summary, @fires,
  @cssprop, @csspart, @slot, and slot/part HTML comments. Use when
  asked to "review docs", "check documentation", "audit jsdoc", or
  before publishing a component.
tools: Read, Glob, Grep, Shell
---

# Documentation Review

Review a Lit web component's documentation for completeness and
quality against CEM (Custom Elements Manifest) conventions.

## Workflow

### Phase 1: Read

Read:
- Element source (`.ts`)
- Element styles (inline or `.css`)
- README (if present)

### Phase 2: JSDoc Checklist

#### Class-Level
- [ ] `@summary` present on element class?
- [ ] Summary is one sentence, describes what the element is?

#### Properties and Methods
- [ ] All public properties have `/** */` JSDoc (not `//`)?
- [ ] All public methods have `/** */` JSDoc?
- [ ] No unnecessary `@default` tags (the CEM analyzer picks up
      defaults from property initializers)?

#### Events
- [ ] `@fires` for every event the element dispatches?
- [ ] `@fires` includes event class name when using Event subclasses?
- [ ] Description explains when/why the event fires?

#### CSS
- [ ] `@cssprop` for every public CSS custom property?
- [ ] CSS custom properties documented with `/** */` in CSS,
      co-located with declaration or `var()` reference?
- [ ] Description explains how the property is used in this element,
      not the token's general purpose?

#### Slots
- [ ] `@slot` for every named slot?
- [ ] `@slot` (no name) for the default slot?
- [ ] Prescriptive descriptions ("Label text" not "The label slot")?
- [ ] HTML comments before each `<slot>` element in the template?

#### CSS Parts
- [ ] `@csspart` for every exposed part?
- [ ] HTML comments before the `part` element in the template?

### Phase 3: Inline Documentation

- [ ] Prefer HTML comments in template for part and slot descriptions
      over JSDoc on the class?
- [ ] Prefer CSS comments for `@cssprop` over JSDoc on the class?
- [ ] No narrating comments ("increment the counter", "return the
      result")?
- [ ] "Why" comments on non-obvious ARIA or accessibility decisions?

### Phase 4: CEM Health (optional)

If Custom Elements Manifest tooling is available:

```bash
cem health --component <element-name> --format json
```

Flag any category below 80%.

### Phase 5: Report

```markdown
## Documentation Review: <element-name>

### Coverage
- JSDoc: X/Y public members documented
- Events: X/Y @fires tags
- CSS: X/Y @cssprop tags
- Slots: X/Y documented (JSDoc + HTML comments)
- Parts: X/Y documented (JSDoc + HTML comments)
- CEM health: [score or N/A]

### Missing Documentation
[Specific items that need docs]

### Quality Issues
[Descriptions that are vague, narrating, or wrong]
```

## Principles

- The CEM is consumable by LLMs and tooling — treat it as a first-class
  API surface
- Co-locate documentation with what it describes (CSS comments with CSS
  properties, HTML comments with slots/parts)
- Descriptions should tell users what to do, not describe the thing
  abstractly
