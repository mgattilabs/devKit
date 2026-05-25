# ADR Format

An ADR (Architecture Decision Record) records *that* a decision was made and
*why*. Its value is the record itself, not the prose around it.

## Location and naming

ADRs live in `docs/adr/`, named `NNNN-slug.md` with sequential numbering —
`0001-...`, `0002-...`. To get the next number, scan `docs/adr/` for the highest
existing one and add one. Create the `docs/adr/` directory only when the first
ADR is actually needed.

## Template

```
# {Short title of the decision}

{One to three sentences: the context, what was decided, and why.}
```

That is the whole template. An ADR can be a single paragraph. Do not pad it with
sections to look thorough.

## Optional sections

Add these only when they carry real value — most ADRs need none of them:

- **Status** (frontmatter: `proposed | accepted | deprecated | superseded by ADR-NNNN`)
  — when a decision is likely to be revisited.
- **Considered options** — when the rejected alternatives are worth remembering.
- **Consequences** — when there are non-obvious downstream effects to call out.

## When to write an ADR

Write one only when **all three** of these are true:

1. **Hard to reverse** — changing your mind later carries a meaningful cost.
2. **Surprising without context** — a future reader looking at the code would
   ask "why was it done this way?".
3. **The result of a real trade-off** — there were genuine alternatives and one
   was chosen for specific reasons.

If a decision is easy to reverse, skip it — you will just reverse it. If it is
not surprising, nobody will wonder why. If there was no real alternative, there
is nothing to record beyond "we did the obvious thing".

## What qualifies

- **Architectural shape** — "the system is a backend plus a worker", "the write
  model is event-sourced, the read model is a projection".
- **Integration patterns between parts** — "modules A and B communicate by
  events, not synchronous calls".
- **Technology choices with lock-in** — database, message bus, auth provider,
  deployment target. Not every library — only the ones that would take months
  to swap out.
- **Boundary and scope decisions** — who owns which data, what is deliberately
  out of scope. The explicit no's matter as much as the yes's.
- **Deliberate deviations from the obvious path** — "manual SQL instead of an
  ORM, because X". Anything a reasonable reader would assume the opposite of;
  recording it stops the next engineer from "fixing" something deliberate.
- **Constraints not visible in the code** — "no cloud provider X for compliance
  reasons", "responses must stay under 200ms because of a partner contract".
- **Non-obvious rejected alternatives** — if an option was considered and
  dropped for subtle reasons, record it, or it will be re-proposed in six
  months.

## In the arch workflow

`arch` writes ADRs autonomously and announces each one; the user can veto. Two
decisions are ADR-worthy by construction: the **module decomposition** (the
approved module list and its dependencies) and any **promotion of `CONTEXT.md`
to `CONTEXT-MAP.md`**. ADRs are flushed at a module's end-of-grilling
checkpoint, not mid-interview — a decision is only worth recording once it is
stable.