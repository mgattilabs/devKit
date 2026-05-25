# CONTEXT.md Format

`CONTEXT.md` records a project's **ubiquitous language** — the terms specific to
its domain, defined once and used consistently. It exists so that the code, the
specs and the conversations all use the same words for the same things.

## Location

A single-context project keeps one `CONTEXT.md` at the repository root. A
multi-context project keeps a `CONTEXT-MAP.md` at the root and one `CONTEXT.md`
per context (e.g. `src/<context>/CONTEXT.md`). Create the file lazily — only
when the first domain term is resolved.

## Structure

```
# {Context name}

{One or two sentences: what this context is and why it exists.}

## Language

**{Term}**:
{One sentence — what it IS, not what it does.}
_Avoid_: {aliases that should not be used for this concept}

## Relationships

- A **{Term}** produces one or more **{Term}**
- A **{Term}** belongs to exactly one **{Term}**

## Example dialogue

> **Dev:** "..."
> **Domain expert:** "..."

## Flagged ambiguities

- "{word}" was used for both **{Term}** and **{Term}** — resolved: {resolution}.
```

## Rules

- **Be opinionated.** When several words exist for one concept, pick the best
  one and list the rest as aliases to avoid.
- **Keep definitions tight.** One sentence. Define what the term *is*, not what
  it does.
- **Flag conflicts explicitly.** A term used ambiguously goes under "Flagged
  ambiguities" with a clear resolution.
- **Show relationships.** Use bold term names and state cardinality where it is
  obvious.
- **Only project-specific terms.** General programming concepts (timeouts,
  error types, utility patterns) do not belong, even if used heavily. Before
  adding a term, ask: is this unique to this domain, or a general concept? Only
  the former qualifies.
- **Group under subheadings** when natural clusters appear; a flat list is fine
  when all terms belong together.
- **Write an example dialogue** between a developer and a domain expert that
  shows the terms interacting and clarifies the boundaries between related ones.

## Single vs multiple contexts

`CONTEXT-MAP.md` lists the contexts, where each one lives, and how they relate:

```
# Context Map

## Contexts

- [Ingestion](./src/ingestion/CONTEXT.md) — imports and indexes raw photo sets
- [Culling](./src/culling/CONTEXT.md) — ranks and filters photos for selection

## Relationships

- **Ingestion -> Culling**: Ingestion emits `SetIndexed`; Culling consumes it
  to start ranking
```

Infer the structure: if `CONTEXT-MAP.md` exists, read it to find the contexts;
if only a root `CONTEXT.md` exists, the project is single-context; if neither
exists, create a root `CONTEXT.md` lazily when the first term is resolved.

## In the arch workflow

`arch` starts every project single-context, with one root `CONTEXT.md` populated
lazily during the functional phase and module grilling. It promotes to
`CONTEXT-MAP.md` only when the decomposition reveals genuinely separate
languages — the same word meaning different things across modules. That
promotion is an architectural decision and is recorded as an ADR. A **context is
not the same as a module**: two modules can share one context, so do not create
one context per module by default.