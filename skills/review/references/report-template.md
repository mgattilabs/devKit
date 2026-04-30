# Report Template

This is the exact structure for every Principal Engineer Review. Do not deviate. Consistency across reviews makes them scannable, comparable, and trustworthy.

Tags used in findings (apply where relevant):

- `[BEHAVIOR-PRESERVING]` — pure refactor, no behavior or performance change
- `[PERF]` — primary motivation is execution speed
- `[READABILITY]` — primary motivation is readability/maintainability
- `[READABILITY TRADEOFF]` — gain in performance at a cost in readability (Rule 3 case — must be justified by hot-path evidence)
- `[BUG]` — finding is an actual bug; **not fixed** in this review (Rule 1); reported only
- `[OUT-OF-SCOPE]` — finding requires a behavior or API change; reported only, decision deferred to user

---

## Template (use this exact structure)

```markdown
# Principal Engineer Review

## Summary
- **Code reviewed**: <function/class/file name>, ~<N> lines
- **Tests detected**: <yes / no / unknown>
- **Findings**: <total> (<refactoring count> refactoring, <perf count> performance, <bug count> bugs reported, <out-of-scope count> out-of-scope)
- **Risk of applying this review**: <low / medium / high> — <one-sentence reason>
- **Hot-path assumption**: <stated by user / inferred from code role / unknown — assumed cold>

## Findings

### F1 — <short finding name> [TAG]
- **Location**: <function name, lines X–Y>
- **What**: <one sentence describing the smell or signal>
- **Cost**: <readability / maintainability / O(...) / allocations per call / etc.>
- **Severity**: <low / medium / high>

### F2 — <short finding name> [TAG]
- **Location**: ...
- **What**: ...
- **Cost**: ...
- **Severity**: ...

<...repeat for every finding...>

## Proposed Fixes

### F1 — <same finding name>
- **Technique**: <name from the refactoring catalog, e.g. "Extract Function", "Replace Loop with Built-in Aggregation", "Sprout Method">
- **Rationale**: <one or two sentences — why this technique fits this finding>
- **Risk**: <low / medium / high — what could go wrong, what mitigates it>
- **Behavior-preservation check**: <which edge cases were mentally verified — happy path, nulls, empty inputs, exceptions, etc.>

### F2 — <same finding name>
- **Technique**: ...
- **Rationale**: ...
- **Risk**: ...
- **Behavior-preservation check**: ...

<...repeat for every finding that has a fix...>

## Final Code

<the complete refactored code, in a single fenced code block, in the original language>

## Behavior Preservation Statement

I confirm the proposed code produces identical observable outputs to the original for: <list each input class verified — happy path, edge cases, error paths>. No business logic was altered. No public API was changed.

<If any [BUG] or [OUT-OF-SCOPE] findings exist, end with this section:>

## Deferred Items (your decision)

- **<finding name> [BUG]**: <description and recommended action — but not applied>
- **<finding name> [OUT-OF-SCOPE]**: <description and why this needs your call before changing>
```

---

## Rules for filling the template

**Summary section.** Keep it dense. The summary is the only thing some readers will read. Every bullet must convey real information; do not pad.

**Findings section.** One finding = one issue. Do not bundle two issues into one finding "for brevity" — the user can't act on bundled findings independently. If two issues share a fix, that's fine: write two findings, then in Proposed Fixes write one fix that resolves both, referencing both finding IDs.

**Proposed Fixes section.** The "Behavior-preservation check" line is mandatory and concrete. "Verified edge cases" is not enough — list which ones. If the original code threw on null input, the proposed code must too, and you must say so explicitly.

**Final Code section.** One block, complete, ready to paste. Do not show diffs unless the original code is so long that a full rewrite would obscure the changes (>200 lines). In that case, show the changed regions clearly delimited with comments like `// === changed: F1, F3 ===`.

**Behavior Preservation Statement.** This is a contract. Sign it explicitly. If you cannot make this statement honestly — for instance, because you found a bug and the user might want it fixed in this review — stop and ask the user before producing the final code.

**Deferred Items section.** Only include this section if there are `[BUG]` or `[OUT-OF-SCOPE]` findings. Do not include an empty "Deferred Items: none" line — the absence of the section is the signal.

---

## Length targets (typical)

- Summary: 5–7 bullets
- Each finding entry: 4 lines
- Each fix entry: 5–7 lines
- Behavior Preservation Statement: 2–4 sentences

If a review balloons past these, your findings are probably too granular — consolidate. A Principal Engineer review of a 100-line function rarely produces more than 5–8 findings; if you have 15, you are nitpicking.
