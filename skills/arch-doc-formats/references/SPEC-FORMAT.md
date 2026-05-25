# Spec Format

A spec is the implementation-ready description of one module. It is the handoff:
an implementation agent should be able to build the module from its spec alone,
without re-deriving the design.

A spec describes the module's **interface and its contract** — never its
implementation. No code, no algorithms, no internal file layout. The test: if it
could be written before deciding *how* to build the module, it belongs in the
spec; if not, it does not.

## Location and naming

Specs live in `docs/specs/`, one file per module, named `<module-slug>.md`.
Create the `docs/specs/` directory when the first spec is written.

## When it is written

A spec is written at a module's end-of-grilling checkpoint — when its interface
has been fully resolved. Every module gets a spec; unlike an ADR, a spec is
never conditional.

## Structure

```
# {Module name}

{One sentence: what this module is responsible for.}

## Interface

The single surface this module presents to callers and to tests.

- **Inputs** — what callers provide, and the shape of it.
- **Outputs** — what the module returns, and the shape of it.
- **Errors** — the failure modes a caller must handle.

## Dependencies

- **{Other module}** — which part of its interface this module calls, and why.

(Omit the section entirely if the module depends on nothing.)

## Constraints

- {Constraint} — see [ADR-NNNN](../adr/NNNN-slug.md).

Reference ADRs; do not copy their content. Also include constraints carried in
from the constraint registry that did not warrant their own ADR.

## Verification

How a reader would know the module works — observable behaviour and acceptance
criteria, at the level of *what* to check, not test code.
```

## Rules

- **The Interface section is the spine.** If it cannot be written completely,
  the module is not finished being grilled — do not write the spec yet.
- **No implementation.** Inputs, outputs, errors and behaviour — not how they
  are produced.
- **Reference, do not duplicate.** Link to ADRs and to other modules' specs;
  never paste their content. A spec stays correct when the things it references
  change.
- **One module, one interface, one spec.** If a spec describes two surfaces, it
  is two modules.
- **Terse and opinionated.** Include a section only when it carries value — a
  module with no dependencies has no Dependencies section.
- **No open questions.** A finished spec has none. Anything still unresolved
  means the grilling is not done.

## In the arch workflow

`arch` writes one spec per module, in topological order, at each module's
checkpoint. When every module is specced, `arch` presents all specs together for
a single approval. After approval the specs are the contract for a separate
implementation step — `arch` does not implement.