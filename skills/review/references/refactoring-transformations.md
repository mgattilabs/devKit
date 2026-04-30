# Refactoring Transformations (Phase 2a — code with tests)

Use this catalog when the code under review **has working tests** that verify observable behavior. Tests are the safety net that lets you transform aggressively. If you do not know whether tests exist, ask. If there are no tests, switch to `legacy-techniques.md` instead.

This catalog is distilled from Fowler's *Refactoring* (2nd edition). It is language-agnostic — the names and shapes of the transformations are universal even when the syntax differs.

---

## The Mechanics of a Safe Transformation

Every transformation in this catalog follows the same micro-process. Internalize it:

1. **Run the tests.** Confirm they are green *before* you change anything. Refactoring on a red bar is debugging in disguise.
2. **Apply one transformation in the smallest step possible.** Even if the goal is large, the step is small.
3. **Run the tests again.** Green? Continue. Red? Revert and take a smaller step.
4. **Commit.** A good refactor session has many small commits. Each commit is reversible.
5. **Repeat** until the structure has reached the target shape.

The reason for this rhythm: small steps make it impossible to introduce a behavior change without the test suite catching it immediately. Large steps lose this property. A Principal Engineer's refactors are boringly safe — the boringness is the feature.

In a review report you cannot show every micro-step (the user reads the final code, not your edit history). But you must *think* in micro-steps when designing the fix, because that is what guarantees Rule 1.

---

## Catalog

Transformations are organized by intent. Pick the one whose name matches what the finding wants.

### Renaming (the fix for Section 1 findings)

Naming findings from `identification-checklist.md` Section 1 are almost always resolved by one of these three transformations. They look trivial but carry a specific behavior-preservation risk: programs can reach symbols by string (reflection, dependency injection containers, serialization, RPC). A "rename" that only updates source-level references can silently break those.

#### Rename Variable / Field / Parameter
- **When**: a finding from Section 1 (mysterious name, wrong grammatical category, encoded name, noise word, etc.).
- **What**: change the name everywhere it is referenced; nothing else.
- **How safely**:
  1. Use the IDE's rename refactor when available — it updates all in-scope references atomically and surfaces ambiguities.
  2. Without IDE support, search for the exact identifier (whole-word match) across the codebase before replacing. Substring replace is a known cause of breakage.
  3. For public API names: check whether the name is referenced from outside the codebase (consumers, configuration files, persisted data, logs that are parsed). Renaming a public symbol is a breaking change disguised as a rename.
- **Behavior preservation**: identical *if* every reference is updated. The reference points to verify:
  - Direct source references (the easy case)
  - Reflection / introspection by name
  - Serialization keys (JSON, XML, database column names) — often the field name is the wire contract
  - Dependency injection or service-locator registrations keyed by string
  - Test fixtures, mocks, and configuration that reference the symbol by name
- **Risk**: low if scope is local; medium if scope is a public field; high if the field participates in serialization or external contracts (in which case a rename may require a migration step, or the source-level rename should keep a wire-level alias).

#### Rename Function / Method
- **Same mechanics as Rename Variable**, with two extra checks:
  - **Override chains**: in OO languages, renaming a virtual/overridable method must rename every override and every base declaration. The IDE handles this when used correctly; a manual rename rarely does.
  - **Interface contracts**: if the method is part of an interface or protocol, the rename is a contract change. Update the interface, every implementer, and every caller.
- **Special case — splitting a misleading name**: when the original name lies about behavior (e.g., a `getX` that also mutates), the right fix is often *not* a single rename but Separate Query from Modifier (later in this catalog). A rename alone makes the name truthful but leaves a function that does two things.

#### Rename Class / Type
- **Same mechanics**, with the additional concerns:
  - **File names**: in many languages a class name and the file that contains it are coupled. Rename both; verify build configuration that may reference the file path.
  - **External persistence**: ORM mappings, polymorphic serialization tags (`type` discriminator fields), and stored class names in serialized blobs all bind to the class name. A rename without a migration silently breaks them.
  - **Public package surface**: if the class is exported, the rename is a breaking change for consumers.
- **Risk**: medium by default, high when the class is part of a serialization contract or a published API.

#### General rule for all three

A rename in a codebase with full IDE tooling and no string-based access is one of the safest transformations available — it is a Phase 2 default. A rename in a codebase that uses reflection, persistence, or external contracts is one of the riskier ones, because the safety net of the compiler does not extend to those references. Decide which world you are in before proposing the fix.

In the report, when the fix is a rename, the **Behavior-preservation check** line must explicitly confirm: "no reflection or serialization by this name; all source references updated." If you cannot confirm this from the code alone, mark the finding as needing user verification.

---

### Composing Methods (function-shape fixes)

#### Extract Function
- **When**: a function is too long, or a code fragment within it has a meaningful name and could stand alone.
- **What**: take the fragment, turn it into its own function, replace the fragment with a call.
- **Risk**: low if the fragment has clear inputs/outputs; medium if it touches many local variables (consider grouping into a parameter object first).
- **Behavior preservation**: identical if all variables read/written by the fragment are passed in/returned out correctly. Watch for early returns (they change meaning when extracted).

#### Inline Function
- **When**: a function adds no value; its body is as clear as its name; or it is a single-call indirection layer.
- **What**: replace the call with the function body, delete the function.
- **Risk**: low.

#### Extract Variable
- **When**: an expression is hard to read or repeated, and a name would document intent.
- **What**: assign the expression to a well-named local, use the local in place of the expression.
- **Risk**: very low.
- **Note**: prefer Extract Variable over a comment when the goal is to explain *what* the expression computes.

#### Inline Variable
- **When**: a variable adds no information beyond the expression itself.
- **What**: replace the variable with the expression, delete the assignment.
- **Risk**: very low; watch for variables read multiple times where the expression has side effects.

#### Replace Temp with Query
- **When**: a temporary variable holds the result of a computation that could be a method.
- **What**: turn the computation into a method (or property), replace temp reads with method calls.
- **Risk**: low. Watch for performance: the method runs every read; if the temp is read many times in a hot loop and the computation is non-trivial, keep the temp.

#### Split Variable (or "Split Loop")
- **When**: one variable (or one loop) is doing two unrelated jobs.
- **What**: introduce a second variable (or loop) so each does one job.
- **Risk**: low. The duplicated loop may look slower but is usually negligible; if the loop is on a hot path, weigh the tradeoff (Rule 3).

### Moving Features

#### Move Function / Move Field
- **When**: a function/field is used more by another class than its own; or it conceptually belongs elsewhere.
- **What**: move it; update all callers.
- **Risk**: medium — touching many call sites raises merge-conflict risk on a team.

#### Slide Statements
- **When**: related statements are scattered through a function with unrelated statements between them.
- **What**: move them adjacent so they read as a unit.
- **Risk**: low if no data dependencies cross; **invalid** if reordering changes observable side-effect order. Verify dependencies.

### Organizing Data

#### Replace Magic Literal with Named Constant
- **When**: an unexplained number/string appears in code (the canonical `42`, `"admin"`, `0.07`).
- **What**: introduce a named constant at module/class level, replace usages.
- **Risk**: very low.

#### Encapsulate Variable
- **When**: a public/global field is read or written from many places.
- **What**: make it private, expose getter/setter (or property), funnel access through them.
- **Risk**: low. Enables future invariant-checking and logging.

#### Replace Primitive with Object
- **When**: a primitive carries domain meaning (email, currency, postal code) and validation/behavior would belong with it.
- **What**: introduce a small value type, replace usages.
- **Risk**: medium — touches many sites. Worth it when the domain concept is real and recurring.

#### Replace Type Code with Polymorphism (or Strategy)
- **When**: a type tag drives different behavior across many `switch`/`if-else` blocks scattered through the code.
- **What**: introduce a class hierarchy or strategy map, dispatch by type rather than by tag.
- **Risk**: medium — bigger restructuring; warranted when the conditional appears in multiple places (single-site type checks rarely justify polymorphism).

### Simplifying Conditionals

#### Decompose Conditional
- **When**: a conditional has a complex boolean test and complex branches.
- **What**: extract the condition into a named function (`isEligibleForDiscount(...)`), extract each branch into a named function. The reader sees three named pieces instead of one wall.
- **Risk**: very low.

#### Consolidate Conditional Expression
- **When**: several sequential conditions return the same result.
- **What**: combine with `||` / `&&`, optionally extract the combined condition into a named function.
- **Risk**: very low.

#### Replace Nested Conditional with Guard Clauses
- **When**: deep `if-else` nesting where some branches handle exceptional cases that should bail out early.
- **What**: turn the bail-out branches into early returns at the top of the function; the remaining body becomes flat.
- **Risk**: very low. Reduces nesting dramatically.

#### Introduce Special Case (Null Object)
- **When**: many call sites check for the same special value (often null) and respond similarly.
- **What**: introduce an object that represents the special case and behaves trivially. Callers no longer need the check.
- **Risk**: medium — adds a class. Worth it when the null check pollutes the code.

### Refactoring APIs

#### Separate Query from Modifier
- **When**: a function returns a value *and* mutates state.
- **What**: split into two: a pure query, a pure modifier. Callers compose.
- **Risk**: low–medium depending on call site count.

#### Remove Flag Argument
- **When**: a function takes a boolean (or enum) that switches between distinct behaviors.
- **What**: split the function into two named functions, one per behavior. Callers pick the right one.
- **Risk**: low–medium depending on call site count.

#### Preserve Whole Object
- **When**: a caller pulls several fields out of an object and passes them individually to a function.
- **What**: pass the whole object instead. Reduces parameter list, future-proofs against new fields.
- **Risk**: low. Watch for unintended coupling — if the function only ever needs two fields and the object grows, the function now sees more than it should.

### Performance Transformations (Rule 2 territory)

#### Replace Loop with Built-in Pipeline
- **When**: an imperative loop performs filter/map/reduce/aggregate work that the language's standard library expresses in one or two calls.
- **What**: replace the loop with the equivalent built-in pipeline (`filter`, `map`, `reduce`, `groupBy`, `sum`, etc., in whatever form the language provides).
- **Behavior preservation**: built-ins generally iterate in the same order; verify if order matters. Built-ins generally short-circuit identically (`any`, `all`, `find`); verify if the loop had a `break`.
- **Performance**: usually faster (built-ins are JIT/native-optimized), always more readable.

#### Hoist Loop Invariant
- **When**: a loop computes the same value on every iteration.
- **What**: compute it once before the loop, reference the cached result inside.
- **Behavior preservation**: identical, provided the expression has no side effects and does not depend on loop-mutated state.

#### Build Lookup Map Outside Loop
- **When**: an inner loop scans the same collection repeatedly to find matching elements (the O(n²) signal).
- **What**: before the outer loop, build a hash map keyed on the lookup field. Inside the inner loop, do an O(1) lookup.
- **Behavior preservation**: identical for unique keys; if duplicate keys exist, decide explicitly which one wins (last-write or first-write) and document.
- **Performance**: O(n²) → O(n) is the canonical big win.

#### Replace Concatenation with String Builder
- **When**: a loop concatenates strings using `+` or `+=`, especially over many iterations.
- **What**: use the language's optimized string-building primitive (StringBuilder, list-then-join, io.StringIO, etc.).
- **Behavior preservation**: identical output; watch for trailing separators if you use a join-based approach.
- **Performance**: O(n²) → O(n) in immutable-string languages.

#### Replace Linear Scan with Hash Lookup
- **When**: a function repeatedly searches a list for membership or by key.
- **What**: store membership in a hash set or hash map at construction time; query in O(1).
- **Behavior preservation**: identical, except the order of insertion is no longer iterable from a hash set — verify this isn't relied on. If insertion order matters, use an ordered set/map equivalent.

---

## Choosing between transformations

Some findings can be addressed by more than one transformation. A heuristic ordering, when in doubt:

1. The **smallest** transformation that resolves the cost. Prefer Extract Variable to Extract Function. Prefer Extract Function to Move Function. Prefer renames to restructurings.
2. The transformation that **leaves the API stable**. Internal refactors are cheap; API changes are expensive (Rule 1's spirit applies even when the rule itself doesn't formally constrain you — public API changes break callers).
3. The transformation whose **mechanics you can describe in one sentence**. If you cannot describe how to apply it cleanly, you cannot trust it.

When two transformations are equally valid, choose the one that opens the door to the *next* improvement you would have made anyway. Refactoring is a sequence; pick the move that sets up the next move.
