# CHECKLIST:BUGS — Bug & Logic Review

Analyze the diff for defects, logic errors, and correctness issues. Read surrounding context as needed.

### Null / Undefined Safety
- [ ] Are all new pointer/reference dereferences guarded?
- [ ] Are optional values checked before use?
- [ ] Are collection accesses bounds-checked or guarded?

### Control Flow
- [ ] Off-by-one errors in loops or array indexing?
- [ ] Are all branches of conditionals handled (including `else` / `default`)?
- [ ] Inverted conditions (`!` in wrong place, `>=` vs `>`)?
- [ ] Unreachable code or logic dead-ends?

### Error Handling
- [ ] Are all exceptions/errors caught at the right level?
- [ ] Are errors silently swallowed (empty catch block)?
- [ ] Are error messages meaningful and not leaking internal details?
- [ ] Are resources (files, connections, locks) released even on error paths?

### Concurrency
- [ ] Race conditions: shared state accessed without synchronization?
- [ ] Deadlocks: locks acquired in inconsistent order?
- [ ] Thread-unsafe data structures used in multi-threaded context?

### Data Integrity
- [ ] Are inputs validated before use (type, range, format)?
- [ ] Are mutations on shared/reference data intentional vs accidental?
- [ ] Are return values from called functions checked?

### Security-Relevant Logic
- [ ] Authentication checks present before protected operations?
- [ ] Authorization checks not bypassable through input manipulation?
- [ ] No SQL/command injection via string concatenation?
- [ ] No sensitive data logged or returned in error responses?

### API Usage
- [ ] Library methods called with correct argument order and types?
- [ ] Deprecated or removed APIs used?
- [ ] Side effects of called methods understood?

**Output**: bulleted list ordered by severity (Critical > High > Medium > Low):
- **[Severity] [File:Line]** `snippet` — Description and suggested fix.
If nothing found: `No bugs or logic issues found.`
