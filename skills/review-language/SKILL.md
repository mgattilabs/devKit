---
name: review-language
description: "Checklist for detecting spelling errors and verb form inconsistencies in identifiers, comments, and strings in a code diff. Use when reviewing code for language quality."
user-invocable: false
---

# Language & Naming Consistency Checklist

## Step 0 — Detect Verb Form Convention

Before reviewing, search 5–10 existing function/method names in the codebase to determine the dominant verb form:
- **Imperative**: `get`, `create`, `update`, `delete`, `parse`, `render`
- **Third-person singular**: `gets`, `creates`, `updates`
- **Present participle**: `getting`, `creating`

Note the dominant form and flag deviations in the diff.

## Spell Check Checklist

Check ALL of the following in added/changed lines:

- [ ] Comments and docstrings: look for common English typos
- [ ] String literals (user-facing messages, log messages, error text)
- [ ] Identifier names: split on camelCase / snake_case word boundaries, check each word
- [ ] Parameter names and local variables
- [ ] Constant names (split UPPER_SNAKE_CASE into words)

**Common typos to watch for**: `teh`, `recieve`, `occured`, `seperate`, `reutrn`, `lenght`, `widht`, `heigth`, `calss`, `fucntion`, `retun`, `paramter`, `defintion`, `initalize`, `occurance`

**Do NOT flag**:
- Domain-specific abbreviations already used consistently in the codebase (`ctx`, `cfg`, `mgr`, `dto`, `repo`)
- Intentional abbreviations in short-scope variables (`i`, `j`, `n`, `e`)
- External library names, SQL keywords, protocol names

## Verb Form Checklist

- [ ] New function/method names use the same verb form as the dominant convention detected in Step 0?
- [ ] Boolean-returning functions use consistent prefix (`is`/`has`/`can`/`should` vs verb-only)?
- [ ] Event handlers use consistent prefix (`on`/`handle` vs none)?
- [ ] Async functions follow naming convention (e.g., no `Async` suffix if not used elsewhere)?

## Output Format

Return two sections:

```
### Spelling Issues
- **[File:Line]** `identifier or text` — Likely intended: `corrected form`

### Verb Form Issues
- **[File:Line]** `functionName` — Uses `{form}` but project convention is `{expected}`. Suggested: `correctedName`
```

If a section has no issues: `No issues found.`
