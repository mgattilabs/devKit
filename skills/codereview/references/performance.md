# CHECKLIST:PERFORMANCE — Performance Review

Analyze only the added/changed lines for performance problems.

### Algorithmic Complexity
- [ ] Nested loops over the same data (O(n²) or worse where O(n) is achievable)?
- [ ] Linear search where a map/set lookup would be O(1)?
- [ ] Sorting inside a loop when the sort could be done once outside?
- [ ] Recursive calls without memoization on overlapping subproblems?

### Memory & Allocations
- [ ] Unnecessary object/string/array creation inside hot loops?
- [ ] Large data structures copied instead of passed by reference?
- [ ] Buffers or collections grown incrementally when final size is known upfront?
- [ ] Memory leaks: allocated resources never freed?

### Database / I/O
- [ ] N+1 query pattern: query inside a loop where a batch query would work?
- [ ] Missing `SELECT` column filter (fetching all columns when only some are needed)?
- [ ] Missing pagination on queries returning unbounded result sets?
- [ ] Synchronous / blocking I/O where async alternatives exist?
- [ ] Network calls inside a loop where batching/parallelism is possible?

### Caching & Memoization
- [ ] Expensive computation repeated with identical inputs (no caching)?
- [ ] External calls (API, DB) made on every request for data that rarely changes?
- [ ] Memoization dependency arrays missing or over-broad (React / computed values)?

### Unnecessary Work
- [ ] Computation in a loop that could be hoisted outside?
- [ ] Early-exit / short-circuit missing on expensive checks?
- [ ] Unused return values from expensive operations?
- [ ] DOM/layout thrashing (read-write interleaving in a loop)?

**Output**: bulleted list:
- **[File:Line]** `snippet` — Performance problem and suggested fix.
If nothing found: `No performance issues found.`
