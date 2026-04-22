# CHECKLIST:LANGUAGE — Language & Naming Consistency Review

**Step 0 — detect verb form convention**: search 5–10 existing function/method names to determine the dominant verb form (imperative `get/create`, third-person `gets/creates`, or participle `getting/creating`). Note it and flag deviations.

### Spell Check (added/changed lines only)
- [ ] Comments and docstrings: common English typos
- [ ] String literals (user-facing messages, log messages, error text)
- [ ] Identifier names: split on camelCase / snake_case boundaries, check each word
- [ ] Parameter names and local variables
- [ ] Constant names (split UPPER_SNAKE_CASE into words)

Common typos: `teh`, `recieve`, `occured`, `seperate`, `reutrn`, `lenght`, `widht`, `heigth`, `calss`, `fucntion`, `retun`, `paramter`, `defintion`, `initalize`, `occurance`

Do NOT flag: domain abbreviations used consistently (`ctx`, `cfg`, `mgr`, `dto`, `repo`), short-scope loop vars (`i`, `j`, `n`), external library names.

### Verb Form Consistency
- [ ] New function/method names use the dominant verb form detected in Step 0?
- [ ] Boolean-returning functions use consistent prefix (`is`/`has`/`can`/`should`)?
- [ ] Event handlers use consistent prefix (`on`/`handle`)?
- [ ] Async functions follow naming convention (e.g., no `Async` suffix if unused elsewhere)?

**Output**: two sections:

### Spelling Issues
- **[File:Line]** `identifier` — Likely intended: `corrected form`

### Verb Form Issues
- **[File:Line]** `functionName` — Uses {form} but convention is {expected}. Suggested: `correctedName`

If a section has no issues: `No issues found.`
