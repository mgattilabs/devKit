---
name: extract-coding-style
description: "Use when: creating a coding style guide, documenting project conventions, capturing style rules, generating style instructions for the code-review agent. Reads the codebase to infer existing conventions, interviews the user with grill-me to fill gaps, then produces a coding-style.instructions.md file."
argument-hint: "Optionally specify a sub-folder or language to focus on"
---

# Extract Coding Style

Produce a `.github/instructions/coding-style.instructions.md` for this project by inferring conventions from the codebase and interviewing the user.

## When to Use

- Before using the `code-review` agent for the first time, to give the style specialist accurate rules
- When joining a new project and wanting to document its implicit conventions
- When you want to enforce consistent style across a team

## Procedure

### Step 1 — Explore the Codebase

Use an **Explore subagent** (thoroughness: medium) to gather:

1. **Existing style configs**: `.editorconfig`, `eslint.config.*`, `.eslintrc.*`, `pyproject.toml`, `setup.cfg`, `.flake8`, `prettier.config.*`, `tsconfig.json`, `.stylelint*`, `rubocop.yml`, `checkstyle.xml`
2. **Language(s) used**: detect from file extensions in `src/`, `lib/`, root
3. **Naming patterns**: sample 5-10 source files and extract:
   - Variable naming convention (camelCase, snake_case, etc.)
   - Function/method naming convention and dominant **verb form** (imperative `get`, third-person `gets`, etc.)
   - Class/type/interface naming convention (PascalCase, prefixes like `I`, `T`, etc.)
   - File naming convention (kebab-case, PascalCase, snake_case, etc.)
   - Constant naming convention (UPPER_SNAKE_CASE, camelCase, etc.)
   - Boolean-returning function prefixes (`is`, `has`, `can`, `should`, or none)
   - Event handler naming pattern (e.g., `on` prefix, `handle` prefix)
   - Private/protected member convention (underscore prefix, `#`, no convention)
4. **Formatting patterns**: indentation (tabs vs spaces, width), max line length, trailing comma usage, quote style (single vs double)
5. **Import/module conventions**: ordering (std lib → third party → local), grouping, aliasing patterns
6. **Comment style**: doc comment format (JSDoc, docstrings, Javadoc), inline comment conventions
7. **Test conventions**: file naming, describe/test patterns, assertion style

### Step 2 — Grill the User

After exploring, apply **grill-me** mode: interview the user about every gap and ambiguity found in Step 1.

For each inferred convention, ask the user to confirm or correct. For every gap where the codebase is inconsistent or silent:
- Present your best guess as the recommended answer
- Ask explicitly if the guess is correct, or what the real rule is
- Do NOT skip any dimension — naming, formatting, imports, comments, tests

Cover at minimum:
- Is there a language/framework standard this project deviates from intentionally?
- What should happen with TODOs/FIXMEs in committed code?
- Are there forbidden patterns or libraries?
- What is the max acceptable function/method length?
- Is there a preferred error-handling pattern?
- Confirm the dominant verb form for function names (imperative, third-person, etc.)
- Are there naming exceptions for specific domains (e.g., DTOs, services, controllers)?
- Is abbreviation usage encouraged or discouraged? Any approved abbreviation list?

### Step 3 — Generate the Style Guide

Produce `.github/instructions/coding-style.instructions.md` with:

```markdown
---
applyTo: "**"
---
# Coding Style Guide

## Language & Framework
...

## Naming Conventions

### Variables
...

### Functions & Methods
- Verb form: {imperative | third-person | ...}
- Boolean prefix: {is/has/can/should | none}
- Event handler prefix: {on | handle | none}
...

### Classes & Types
...

### Files
...

### Constants
...

### Private/Protected Members
...

## Formatting
...

## Imports & Modules
...

## Comments & Documentation
...

## Error Handling
...

## Tests
...

## Forbidden Patterns
...
```

Rules must be **concrete and actionable** — avoid vague statements like "write clean code". Every rule should be something a code reviewer could check mechanically.

After saving the file, tell the user:
- What was inferred vs what they confirmed
- How to activate it for the `code-review-style` agent (it auto-loads via `applyTo: "**"`)
- How to refine it later
