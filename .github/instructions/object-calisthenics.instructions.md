---
applyTo: '**/*.{cs,ts,java}'
description: Enforces Object Calisthenics principles for business domain code to ensure clean, maintainable, and robust code
---
# Object Calisthenics Rules

> ⚠️ Exactly 9 rules — none added, removed, or replaced.

## Scope
- **Primary**: Business domain classes (aggregates, entities, value objects, domain services)
- **Secondary**: Application layer services and use case handlers
- **Exemptions**: DTOs, API models, configuration classes, simple data containers, infrastructure code

## The 9 Rules

1. **One level of indentation per method** — extract loop bodies and complex conditions into dedicated methods
2. **No `else`** — use early returns and guard clauses always; apply Fail Fast principle
3. **Wrap all primitives and strings** — every domain primitive becomes a Value Object with validation
4. **First class collections** — a class with a collection attribute must have no other attributes; encapsulate collection behavior
5. **One dot per line** (Law of Demeter) — never chain nested property accesses
6. **Don't abbreviate** — use full, descriptive names for classes, methods, variables
7. **Keep entities small** — max 50 lines/class; max 10 methods/class; max 10 classes/namespace
8. **No more than 2 instance variables** per class (logger excluded)
9. **No getters/setters in domain classes** — use private constructors, static factory methods, behavioral methods

## Implementation Guidelines

- **Domain classes**: Apply all 9 rules strictly. Private constructors, factory methods, no public setters.
- **Application layer**: Apply all 9 rules. Single responsibility and clean abstractions.
- **DTOs**: Rules 3, 8, 9 may be relaxed. Public properties with getters/setters are acceptable.
- **Tests**: Relaxed rules for readability; validate behavior not state.
- **Code reviews**: Enforce strictly for domain/application code; be pragmatic for infrastructure/DTOs.
