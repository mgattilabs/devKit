---
name: review-simplifier
description: "Checklist for identifying simplification, clarity, maintainability, and Clean Code design principle violations in recently changed code. Use when reviewing code for readability, reduced complexity, SOLID, DRY, YAGNI, Law of Demeter."
user-invocable: false
---

# Simplification & Clarity Review Checklist

**Scope**: Read the full content of each modified file (not just the diff lines) to understand context, then focus analysis on recently changed sections and their immediate surroundings.

## Checklist

### Control Flow Simplification
- [ ] Deeply nested `if/else` that could be flattened with early returns / guard clauses?
- [ ] Boolean expressions that could use De Morgan's law or be simplified?
- [ ] `if (condition) { return true } else { return false }` → `return condition`?
- [ ] `switch`/`if-else` chains that could be replaced by a lookup table or map?

### Duplication & Extraction
- [ ] Logic repeated two or more times that could be extracted into a helper?
- [ ] Magic numbers or strings that should be named constants?
- [ ] Inline logic that shadows an existing utility already in the codebase?

### Naming Clarity
- [ ] Variables or parameters with misleading, vague, or abbreviated names (`d`, `temp`, `data`, `result`)?
- [ ] Functions named after their implementation rather than their intent?

### Function / Method Size
- [ ] Functions that do more than one logical thing and could be split?
- [ ] Overly long parameter lists (>3–4) that should be grouped into an object/struct?

### Dead Code
- [ ] Commented-out code left in?
- [ ] Variables declared but never used after the change?
- [ ] Unreachable branches introduced?

### Over-engineering
- [ ] Abstractions introduced that serve only this one call site?
- [ ] Design patterns applied where a simple function would suffice?
- [ ] Premature generalization (generic solution for a problem that only exists once)?

### Cognitive Load
- [ ] Long chains of method calls that are hard to mentally parse?
- [ ] Side effects inside expressions (assignments in conditions, etc.)?
- [ ] Implicit assumptions about argument order / types that could be made explicit?

### Clean Code — Design Principles

#### Single Responsibility (SRP)
- [ ] Does the changed class/module do more than one thing? (multiple reasons to change)
- [ ] Does the changed function mix I/O, business logic, and formatting?
- [ ] Are unrelated concerns (e.g., logging + validation + persistence) mixed in the same function?

#### Open/Closed (OCP)
- [ ] Does the change require modifying existing stable code to add new behavior, where extension points could have been used instead?

#### Liskov Substitution (LSP)
- [ ] Does the changed subclass violate the contract of its parent (throws unexpected exceptions, narrows preconditions, weakens postconditions)?

#### Interface Segregation (ISP)
- [ ] Does the changed interface or class expose methods that some callers never use?
- [ ] Are unrelated methods grouped in the same interface?

#### Dependency Inversion (DIP)
- [ ] Does new code depend on concrete implementations where depending on an abstraction would decouple it?
- [ ] Are high-level modules importing low-level detail modules directly?

#### Law of Demeter (LoD)
- [ ] Does new code call methods on objects returned by other calls (`a.getB().getC().doX()` — "train wreck")?
- [ ] Does a function reach into the internals of an object it received as a parameter?

#### DRY / YAGNI / KISS
- [ ] Is the same knowledge (rule, formula, constant) duplicated in multiple places? *(DRY)*
- [ ] Is new code added "just in case" for a use case that doesn't exist yet? *(YAGNI)*
- [ ] Is the simplest solution used, or is unnecessary complexity introduced? *(KISS)*
- [ ] Could a built-in language/library feature replace custom implementation? *(KISS)*
- [ ] Are there multiple layers of indirection (wrappers, adapters, factories) where a direct call would work? *(KISS)*
- [ ] Is a design pattern applied that adds more complexity than the problem it solves? *(KISS + YAGNI)*

#### Comments (Clean Code style)
- [ ] Comments that explain *what* the code does instead of *why*?
- [ ] Commented-out code left in (already in Dead Code section, but flag here too)?
- [ ] Outdated comments that no longer match the code?

## Output Format

Return a bulleted list:

```
- **[File:Line]** `code snippet` — What to simplify and why. Brief example of cleaner form.
```

If no issues found: `No simplification opportunities found.`
