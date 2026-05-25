---
name: impl-coding
description: >
  Behavioral principles and workflow rules for writing implementation code from
  arch specs. Use this skill whenever the impl agent (or any agent following the
  impl workflow) is about to write, edit, or review production code derived from
  an arch spec. Also trigger when the user asks to "implement a spec",
  "code a module", "turn this spec into code", or references impl-coding
  directly. This skill does NOT define stack-specific conventions — those are
  decided per-project before implementation begins.
---

# impl-coding — How impl writes code

This skill defines *how* the impl agent writes code. It is stack-agnostic:
language-specific naming, formatting, and idioms follow the conventions of
whatever stack the spec declares. What this skill governs is the behavioral
discipline around writing code from specs.

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing anything:

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations of the spec exist, present them — don't pick
  silently.
- If a simpler approach exists than what the spec implies, say so.
- If something in the spec is ambiguous or contradictory, stop. Name the
  problem. Ask.

## 2. Simplicity First

Minimum code that solves the spec. Nothing speculative.

- No features beyond what the spec defines.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that the spec didn't ask for.
- No error handling for impossible scenarios.
- If you wrote 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes,
simplify.

Simplicity connects to depth (Ousterhout): a good module has a small interface
hiding a rich implementation. If you're adding parameters, options, or config
to an interface without proportionally more behaviour behind it, you're making
the module shallower, not simpler. Consult `improve-codebase-architecture`
when in doubt.

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken — unless they violate the spec's
  module structure (see section 6). Structural alignment is not optional.
- Match existing style for naming and formatting (see section 7).
- If you notice unrelated issues, mention them — don't fix them.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the spec.

## 4. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Implement module X" → "Satisfy all acceptance criteria in the spec"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

## 5. Spec Is the Contract

The arch spec is the source of truth. Implementation decisions not covered by
the spec are yours to make — but call them out so the user knows.

- **Acceptance criteria** in the spec are non-negotiable. Every one must be
  satisfied and verifiable.
- **Boundaries** (what's in scope, what's out) are respected. Don't implement
  what's explicitly out of scope.
- **Dependencies** between modules are honored. If module A depends on module
  B's interface, code to that interface.
- **Open questions** in a spec mean the spec isn't ready. Don't guess — ask.

## 6. Project Structure — Spec Wins

The spec's module decomposition is the structural truth. The code layout must
mirror it. If the existing codebase has a different structure (e.g., layer-based
folders, entities scattered in a shared Data/ folder, services floating
outside their feature), **align to the spec, not to the existing mess**.

Organize code so each module in the spec maps to a cohesive folder containing
everything that module owns — entities, handlers, services, DTOs, tests:

```
src/
├── invoicing/              # maps to spec: invoicing
│   ├── Invoice.cs          # entity lives HERE, not in Data/Entities/
│   ├── InvoiceLine.cs
│   ├── CreateInvoice.cs    # handler/command
│   ├── GetInvoices.cs      # handler/query
│   ├── PdfService.cs       # service lives HERE, not in Services/
│   └── InvoicingTests/
├── registry/               # maps to spec: registry
│   ├── Client.cs
│   ├── Project.cs
│   └── ...
├── time-log/               # maps to spec: time-log
│   ├── TimeEntry.cs
│   └── ...
└── shared/                 # cross-module concerns ONLY when justified
    ├── BaseEntity.cs
    ├── PagedResult.cs
    └── AppDbContext.cs
```

Rules:

- A module in the spec = a folder in the code.
- Everything a module owns is co-located inside that folder.
- Don't create horizontal layer folders (Data/, Services/, Models/) that spread
  a module's internals across the tree. Those break cohesion.
- `shared/` exists only for genuinely cross-cutting concerns (base classes,
  DB context, pagination utilities). If something is used by one module only,
  it belongs in that module's folder.
- Don't invent modules that don't exist in the specs.
- Don't merge modules that the specs keep separate.
- If the existing codebase violates this structure, refactor the affected parts
  to align with the spec. Flag the structural changes to the user at checkpoint.

## 7. Stack Conventions

This skill does NOT prescribe naming, formatting, test frameworks, or
language idioms. Those are determined per-project through a priority chain:

1. **Language-specific skill** — if a skill exists for the stack in use (e.g.,
   `impl-dotnet`, `impl-angular`, `impl-python`), it overrides everything
   below for that stack's conventions.
2. **From the user** — if the user declares conventions before implementation,
   those apply.
3. **From the spec** — if the arch spec declares a stack, follow that stack's
   community conventions by default.
4. **From the codebase** — if adding to an existing project, match naming,
   formatting, and idioms that are already there.

**Important distinction:** this priority applies to *style* (naming, formatting,
idioms, test frameworks). It does NOT apply to *structure* (folder layout,
co-location, module boundaries). Structure always comes from the spec — see
section 6.

When starting a new project with no existing conventions and no language skill,
ask the user before making opinionated choices (e.g., test framework, linter
config, folder casing).

## 8. Commits

Delegate to `caveman-commit`. Conventional commits, subject ≤50 chars, why
over what. One logical change per commit — don't bundle unrelated changes.

## 9. Tests

- Write tests for acceptance criteria first — they're the spec's verification.
- Test behavior, not implementation details.
- Test names describe the scenario, not the method under test.
- Don't write tests for code you didn't write or change.

## 10. When Things Don't Fit

If during implementation you discover that:

- The spec is missing something critical → stop, describe the gap, ask.
- The spec's approach won't work technically → stop, explain why, propose an
  alternative, ask.
- Two specs contradict each other → stop, name the contradiction, ask.

Never silently deviate from the spec. The user approved those specs for a
reason.