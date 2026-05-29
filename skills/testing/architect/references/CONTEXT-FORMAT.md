# CONTEXT.md Format

## Structure

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Order**:
A request from a customer to purchase one or more products.
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account

## Relationships

- A **Customer** has many **Orders**
- An **Order** has many **Line Items**
- An **Order** produces one **Invoice** upon shipment

## Flagged ambiguities

- "Account" was used to mean both Customer and User — resolved: **Customer** is the billing entity, **User** is the login identity.

## Example dialogue

> **Dev**: "When an Order is placed, does it automatically generate an Invoice?"
> **Domain expert**: "No — the Invoice is created only when the Shipment is dispatched. An Order can sit in 'confirmed' status for days before shipping."
```

## Rules

- **Be opinionated.** One name per concept. List alternatives as _Avoid_.
- **Flag conflicts explicitly.** If a term is used ambiguously, call it out with a clear resolution.
- **Keep definitions tight.** One or two sentences. Define what it IS, not what it does.
- **Show relationships.** Bold term names, express cardinality.
- **Only domain-specific terms.** No general programming concepts (timeout, error, utility).
- **Write the example dialogue.** It forces you to prove the terms work together naturally. It also helps agents understand the domain in context.
- **Zero implementation details.** CONTEXT.md is a glossary. Not a spec, not a scratchpad.

## Single vs multi-context repos

**Single context (most repos):** One `CONTEXT.md` at `docs/CONTEXT.md`.

**Multiple contexts:** A `CONTEXT-MAP.md` at `docs/` lists the contexts and their relationships:

```md
# Context Map

## Contexts

- [Ordering](../src/ordering/CONTEXT.md) — receives and tracks customer orders
- [Billing](../src/billing/CONTEXT.md) — generates invoices and processes payments
- [Fulfillment](../src/fulfillment/CONTEXT.md) — manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched`; Billing generates invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

Create files lazily — only when you have something to write.
