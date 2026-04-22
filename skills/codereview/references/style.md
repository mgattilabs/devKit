# CHECKLIST:STYLE — Style & Conventions Review

**Step 0 — detect project style config** before reviewing: look for `.editorconfig`, `eslint.config.*`, `.eslintrc.*`, `pyproject.toml`, `prettier.config.*`, `tsconfig.json`, `.stylelintrc`, `rubocop.yml`, `.github/instructions/coding-style.instructions.md`. Apply those rules if found; otherwise apply the dominant language standard.

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

**Output**: bulleted list:
- **[File:Line]** `snippet` — Rule violated and recommended fix.
If nothing found: `No style issues found.`
