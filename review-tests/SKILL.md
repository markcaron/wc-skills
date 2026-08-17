---
name: review-tests
description: >
  Review a Lit web component's test suite for coverage, patterns, and
  correctness. Checks public API testing, a11y assertions, keyboard
  navigation, form submission, and Mocha conventions. Use when asked to
  "review tests", "check test coverage", "audit tests", or after
  writing tests for a new component.
tools: Read, Glob, Grep
---

# Test Review

Review a Lit web component's test suite for coverage quality and
correct testing patterns.

## Workflow

### Phase 1: Read

Read:
- All test files
- Element source (to identify the full public API)
- `ADVICE.md` test-related rules

### Phase 2: Coverage Checklist

#### Public API
- [ ] Every public attribute/property tested?
- [ ] Every public method tested?
- [ ] Every dispatched event tested (type, properties, timing)?
- [ ] Child-dispatched events tested (e.g. `activate` from a child
      element)?

#### Accessibility
- [ ] `a11ySnapshot` assertions for roles, labels, and states?
- [ ] Keyboard navigation tested (arrow keys, Home, End, Tab,
      Space/Enter as applicable)?
- [ ] Focus management tested (roving tabindex, focus moves)?

#### States
- [ ] All component states covered (disabled, expanded, selected,
      error, loading, etc.)?
- [ ] Transitions between states tested?
- [ ] Default/initial state asserted?
- [ ] Initial attribute override tested (e.g. `selected` set in HTML)?

#### Form Association (if applicable)
- [ ] Form submission tested with `preventDefault()` + value check?
- [ ] Validation states tested?

### Phase 3: Pattern Checklist

- [ ] Setup in `beforeEach`, assertions in `it` blocks?
- [ ] No arrow functions in `describe`/`it`/`beforeEach` blocks
      (Mocha needs `this` context)?
- [ ] Test observable behavior (attributes, properties, events,
      `offsetWidth`, computed styles, a11y tree), not shadow DOM
      internals?
- [ ] No querying into shadow roots to verify rendering?
- [ ] Cancelable events tested via `preventDefault()` + assert no
      state change?
- [ ] Async state changes properly awaited (`updateComplete`,
      `oneEvent`)?

### Phase 4: Report

```markdown
## Test Review: <element-name>

### Coverage
- Attributes/properties: X/Y tested
- Events: X/Y tested
- States: X/Y covered
- Keyboard: [covered | gaps]
- a11y tree: [covered | missing]

### Pattern Issues
[List any anti-patterns found]

### Missing Tests
[Specific tests that should be added]
```

## Principles

- Tests should verify what the user experiences, not how it's built
- a11ySnapshot is the source of truth for accessibility assertions
- Every dispatched event (public and internal) needs a test
- State coverage matters more than line coverage
