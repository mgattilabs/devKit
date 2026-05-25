---
name: arch-doc-formats
description: File-format definitions for the design artifacts the `arch` agent produces — ADRs, CONTEXT.md / CONTEXT-MAP.md, and module spec files. Consult this skill whenever arch, or any agent following the arch workflow, is about to write or update an ADR, a ubiquitous-language CONTEXT file, or a module spec, so the file matches the agreed format. Use it even for a quick single-file write — the formats are deliberately terse and easy to get subtly wrong.
---

# arch-doc-formats

Format definitions for the three kinds of file the `arch` agent writes.

`arch` itself defines the *workflow* — when and why each file is produced. This
skill defines the *shape* of each file. Read the relevant reference before
writing the corresponding file:

- Writing an ADR → `references/ADR-FORMAT.md`
- Writing or updating `CONTEXT.md` / `CONTEXT-MAP.md` → `references/CONTEXT-FORMAT.md`
- Writing a module spec → `references/SPEC-FORMAT.md`

Each reference is self-contained: it states where the file lives, when to create
it, the template, and the rules for what belongs in it.

## Quick reference

| File          | Lives in                     | One per   | Written when                                                  |
|---------------|-------------------------------|-----------|---------------------------------------------------------------|
| ADR           | `docs/adr/NNNN-slug.md`       | decision  | a cross-module / global decision passes the three-condition test |
| CONTEXT.md    | repo root (or `src/<context>/`) | context | the first domain term is resolved                            |
| Spec          | `docs/specs/<module-slug>.md` | module    | a module's interface is fully grilled                         |

All three directories are created lazily — only when the first file of that kind
is needed.

## Shared principles

These hold across all three formats:

- **Lazy creation.** Never create a file or directory preemptively. Create it
  when the first real piece of content for it exists.
- **Terse and opinionated.** Include a section only when it carries value. An
  empty or padded section is worse than no section.
- **Reference, never duplicate.** Specs link to ADRs; `CONTEXT-MAP.md` links to
  `CONTEXT.md` files. Copied content goes stale — links do not.