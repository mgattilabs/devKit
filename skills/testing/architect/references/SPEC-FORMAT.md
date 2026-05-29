# SPEC Format

Specs live in `docs/spec/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`.

Create `docs/spec/` lazily — only when the first spec is needed.

## Template

```md
# {Feature title}

## Problem

{What problem does this solve? For whom? What happens today without this?
2-3 sentences max. If you can't state the problem clearly, you don't understand it yet.}

## Solution

{What we're building. Describe the behavior, not the implementation.
Use domain language from CONTEXT.md. Reference relevant ADRs by number.}

## Acceptance criteria

Tests that prove the feature works. Each test IS a criterion.

- `should_create_order_with_valid_items` — Given a customer with items in stock, when they place an order, then the order is created with status "confirmed"
- `should_reject_order_when_inventory_insufficient` — Given an item with 0 stock, when a customer tries to order it, then the order is rejected with reason "out of stock"
- `should_emit_order_placed_event` — Given a successfully created order, then an OrderPlaced event is emitted with order ID and customer ID

## Tasks

Atomic, agent-executable units of work. Each maps to one or more acceptance criteria. Tasks describe WHAT to implement and which docs to reference — not HOW to implement it.

- [ ] `should_create_order_with_valid_items` — Implement the happy path for order creation. Ref: CONTEXT.md for Order and Customer definitions.
- [ ] `should_reject_order_when_inventory_insufficient` — Add inventory validation when placing an order. Ref: CONTEXT.md for Inventory context.
- [ ] `should_emit_order_placed_event` — Emit domain event on successful order creation. Ref: ADR-0003 for event bus choice.

## Out of scope

{What this spec deliberately does NOT cover, and why. Each item is a candidate for a future spec or a "no" ADR.}
```

## Rules

- **Problem first.** If the problem section is weak, the spec is weak. Grill harder.
- **Tests are the spec.** The acceptance criteria section is the most important part. If you can't write the test name and the given/when/then, the feature isn't understood.
- **Domain language.** Use terms from CONTEXT.md. If a term isn't in the glossary and you need it, add it to CONTEXT.md as a side-effect.
- **Tasks are for agents.** Each task should be executable by an agent reading only: the task description, the test, and CONTEXT.md. No implicit knowledge.
- **Out of scope is mandatory.** Forces you to state the "no" decisions at the feature level. Anything non-trivial in out-of-scope should reference an ADR or become one.
- **No implementation details.** The spec says WHAT, not HOW. The how lives in the code. Exception: if an ADR constrains the how (e.g., "must use event sourcing"), reference the ADR.

## Numbering

Scan `docs/spec/` for the highest existing number and increment by one.

## Lifecycle

A spec starts as `proposed`. During implementation, tasks get checked off.
When all tasks are done and tests pass, the spec is `done`.
If requirements change, update the spec — don't create a new one unless the scope is fundamentally different.
