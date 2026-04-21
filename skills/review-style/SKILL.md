---
name: review-style
description: "Checklist for verifying code style, naming conventions, formatting, and consistency in a code diff. Use when reviewing code for style compliance."
user-invocable: false
---

# Style & Conventions Review Checklist

## Step 0 — Detect Project Style Rules

Before reviewing, search for project-level style configuration:
- `.editorconfig`, `eslint.config.*`, `.eslintrc.*`
- `pyproject.toml` (tool.black, tool.ruff, tool.isort), `setup.cfg`, `.flake8`
- `prettier.config.*`, `tsconfig.json`, `.stylelintrc`
- `rubocop.yml`, `checkstyle.xml`, `.scalafmt.conf`
- `.github/instructions/coding-style.instructions.md`

If found, apply those rules. If not found, apply the dominant language standard (PEP 8, ESLint recommended, Google Style, etc.).

## Checklist

### Naming
- [ ] Variable names follow project convention (camelCase / snake_case / etc.)?
- [ ] Function/method names follow convention and are consistent with existing codebase?
- [ ] Class/type names follow convention (PascalCase, etc.)?
- [ ] Constants use the correct style (UPPER_SNAKE_CASE, etc.)?
- [ ] File names follow the established pattern?
- [ ] No single-letter names except conventional loop indices (`i`, `j`, `k`) or math variables?

### Formatting
- [ ] Indentation matches the project standard (tabs vs spaces, width)?
- [ ] Lines within max length limit?
- [ ] No trailing whitespace?
- [ ] Consistent blank line usage between functions/classes?
- [ ] Quote style consistent (single vs double)?
- [ ] Trailing comma usage consistent with project convention?

### Imports
- [ ] Imports ordered per project convention (stdlib → third-party → local)?
- [ ] No unused imports introduced?
- [ ] No wildcard imports (`import *`) unless explicitly allowed?

### Language Idioms
- [ ] Idiomatic constructs used where available (list comprehensions, destructuring, ternaries, etc.)?
- [ ] No verbose patterns where a one-liner is the established convention?
- [ ] No deprecated language features?

### Consistency with Surrounding Code
- [ ] New code matches the style of the file it's in, even if the file predates a newer convention?
- [ ] No mixing of styles within a single block?

## Output Format

Return a bulleted list:

```
- **[File:Line]** `code snippet` — Rule violated and recommended fix.
```

If no issues found: `No style issues found.`
