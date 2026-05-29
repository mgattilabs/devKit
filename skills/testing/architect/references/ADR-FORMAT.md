# ADR Format

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`.

Create `docs/adr/` lazily — only when the first ADR is needed.

## Template

```md
# {Short title of the decision}

{1-3 sentences: what's the context, what did we decide, and why.}
```

That's it. An ADR can be a single paragraph. The value is in recording THAT a decision was made and WHY.

## Optional sections

Only include when they add genuine value. Most ADRs won't need them.

- **Status** (`proposed | accepted | deprecated | superseded by ADR-NNNN`)
- **Considered Options** — only when rejected alternatives are worth remembering
- **Consequences** — only when non-obvious downstream effects need calling out

## "No" decisions

Decisions NOT to do something are ADRs when they meet the criteria. Format:

```md
# No multi-currency support

The client operates exclusively in EUR. We use a simple decimal for money with no currency wrapper. If the client expands to other markets, the Money value object must become currency-aware — this would touch every module that handles pricing.
```

The "no" ADR prevents the same debate from resurfacing without context.

## When to write an ADR

All three must be true:

1. **Hard to reverse** — changing your mind later is costly
2. **Surprising without context** — a future reader will wonder "why?"
3. **Result of a real trade-off** — genuine alternatives existed

If any is missing, skip. Easy to reverse → just reverse it. Not surprising → nobody will wonder. No real alternative → nothing to record.

## What qualifies

- Architectural shape ("monorepo", "event-sourced write model")
- Integration patterns between contexts ("domain events, not synchronous HTTP")
- Technology choices that carry lock-in (database, message bus, auth provider)
- Boundary and scope decisions ("Customer data owned by Customer context only")
- Deliberate deviations from the obvious path ("manual SQL instead of ORM because X")
- Constraints not visible in code ("can't use AWS due to compliance")
- Rejected alternatives when rejection is non-obvious
- Deliberate scope exclusions with reasoning (the "no" decisions)

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.
