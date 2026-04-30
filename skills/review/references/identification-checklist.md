# Identification Checklist (Phase 1)

This checklist is the lens you scan the code through during Phase 1. It is distilled from Fowler's *Refactoring* (code smells), Martin's *Clean Code* (naming, function, and module smells), and Feathers' *Working Effectively with Legacy Code* (testability signals), plus a curated list of language-agnostic performance signals.

You do not need to flag every item on this list — only those where the cost is real for *this* code. Junior reviewers flag everything; Principal reviewers flag what matters.

Walk the code section by section. For each finding you spot, record: name, location, cost, severity. Do not propose fixes here — that is Phase 2.

---

## 1. Naming & Communication

Naming carries more cognitive load than any other code quality dimension: every reader pays for a bad name on every visit. Walk this section in three passes — grammatical role, lexical content, scope-fit — and only flag what genuinely costs the reader.

### 1a. Grammatical role (the part-of-speech rule)

A name's grammatical category should match what the named thing *is*. Mismatches force the reader to decode intent.

- **Function / method that performs an action** → must be a **verb or verb phrase** (`calculate...`, `loadFrom...`, `notify...`). A function named as a noun (`emailValidation()`) hides its intent. Flag when a function name has no verb at all and is not a predicate.
- **Function / method that returns a boolean (predicate)** → must be a **predicate phrase** with a modal/state verb (`is...`, `has...`, `can...`, `should...`, `contains...`). A boolean-returning function named without one (`empty()`, `validUser()`) reads ambiguously: is it a query or a command?
- **Class / type / module** → must be a **noun or noun phrase** naming an entity, role, or capability (`Order`, `OrderRepository`, `PaymentGateway`). A class named as a verb (`ProcessOrder`) is almost always a function masquerading as a class — flag and consider whether it should actually be a function.
- **Variable / field / parameter holding a single value** → **noun**. Holding a boolean → **predicate** form, same rules as boolean functions. Holding a collection → **plural noun** (`users`, not `userList` or `user`). Singular names for collections are a recurring source of off-by-one bugs in reading.
- **Constant** → noun or noun phrase, in the language's idiomatic constant convention. A constant named as a verb signals that the value is the *result* of an action — usually it should be a function call instead.
- **Event / message / command type** → follow the team's convention (past-tense for events: `OrderShipped`; imperative for commands: `ShipOrder`). Mixing the two in the same codebase is a finding.

### 1b. Lexical content (does the word actually mean something?)

Even a grammatically correct name can be empty. These are the recurring offenders:

- **Noise words** — `data`, `info`, `manager`, `helper`, `util`, `handler`, `processor`, `service`, `wrapper` used without further qualification. They convey "something happens here" and nothing else. A `UserManager` either does one thing (and should be named for it: `UserAuthenticator`, `UserStore`) or does many (and should be split). Flag noise words; require qualification.
- **Implementation-based names** — `accountList`, `userArray`, `configDict`. The container type is implementation; the reader cares about intent. `accounts` is enough — and stays correct if the implementation changes.
- **Encoded names** — Hungarian notation (`strName`, `iCount`, `m_field`), type-suffix conventions, prefixes the team did not agree to. The IDE and the type system already carry this information.
- **Abbreviations not part of a shared vocabulary** — `usr`, `cfg`, `ctx`, `acc` cost the reader a translation step. Standard domain abbreviations (`HTTP`, `URL`, `ID`) are fine; ad-hoc ones are not.
- **Misleading names** — the function name promises one thing, the body does another. `getUser` that creates a user when none exists. `validate` that also persists. These are bugs in the name; they cause real defects when callers trust the name.
- **Negated predicates** — `isNotEmpty`, `disabled` (vs `enabled`), `shouldNotRetry`. Double negatives at call sites (`if (!isNotEmpty(...))`) are reliably misread. Prefer the positive form.
- **Names that lie about side effects** — a name shaped like a query (`getX`, `findX`) on a function that mutates state. Either rename to reflect the mutation, or split (Command-Query Separation, see Section 2).
- **Inconsistent terms for the same concept** — `user` in one module, `account` in another, `customer` in a third, all referring to the same entity. The reader has to verify they are the same thing every time. Pick one and propagate.

### 1c. Scope-fit (is the name the right length for where it lives?)

Name length should be proportional to the scope it spans. A short name is fine in a tight scope; the same name is unreadable across a long one.

- **Single-letter or near-empty names outside a 3–5 line scope** — `i`, `x`, `tmp`, `d` are acceptable in a tiny loop or a one-line lambda. In a 30-line function or as a field, they are a finding.
- **Long names in tight scope** — the inverse failure: `currentlyIteratingUserAccount` as a loop variable in a 3-line block is noise. Loop variables can be short by convention.
- **Public API names that are too short** — exported / public symbols are read by people who do not have the surrounding context. They earn longer, more explicit names than internals.
- **Names that depend on context the reader doesn't have** — `process()` on a `Job` class is fine in context; the same name on a free-floating utility module is empty.

### 1d. Comments

Naming and comments occupy the same niche — when a name is good, the comment becomes redundant; when a name is bad, the comment apologizes for it.

- **Comments that explain *what* instead of *why*** — if the code needs a comment to say what it does, the code is not clear enough. Improve the name first, delete the comment second. Comments that explain *why* (a non-obvious decision, a workaround, an external constraint, a deliberate tradeoff) are valuable and should stay.
- **Outdated or wrong comments** — worse than no comments. They actively misinform. Treat them as a finding equal in severity to a bug-shaped name.
- **Commented-out code** — version control remembers; the codebase should not. Flag.

## 2. Function Shape

- **Long function** — a function that does not fit on one screen, or that requires the reader to scroll to hold its meaning. The threshold is contextual but rarely above 30–40 lines.
- **Multiple levels of abstraction in one function** — a function that mixes high-level orchestration (`processOrder`) with low-level details (`bytes[3] = 0xFF`) in the same block. Each function should operate at one level.
- **Long parameter list** — more than 3–4 parameters is a smell. Often signals that parameters should be grouped into a value object, or that the function does too much.
- **Flag parameters** — a boolean parameter that switches the function between two behaviors. Usually means the function should be split into two.
- **Output parameters** — passing a parameter to be mutated as the "result". The reader expects parameters to be inputs; mutation should be either explicit (return value) or clearly documented.
- **Side-effect surprise** — a function whose name suggests a query but that also mutates state, or vice versa. Command-Query Separation: a function should either return a value *or* change state, not both.
- **Dead code** — unreachable branches, commented-out blocks, unused parameters, unused returns. Version control remembers them; the codebase does not need to.

## 3. Class & Module Shape

- **Large class** — a class doing too many things. Telltale signs: many fields, many public methods, or methods that operate on disjoint subsets of fields.
- **Feature envy** — a method that uses another class's data more than its own. The behavior probably belongs in the other class.
- **Data clump** — the same set of parameters repeatedly traveling together (e.g., `firstName`, `lastName`, `email` always passed together). They want to be a value object.
- **Primitive obsession** — using primitives for domain concepts (a string for an email, a number for a currency amount). Wrapping them gives type safety, validation point, and a place to attach behavior.
- **Divergent change** — one module is modified for many unrelated reasons. Single Responsibility violation.
- **Shotgun surgery** — one logical change requires touching many modules. Cohesion is in the wrong place.
- **Lazy element** — a class or function so trivial it adds no value. Inline it.
- **Middle man** — a class that mostly delegates to another. Inline the delegation.

## 4. Data & State

- **Mutable data passed across boundaries** — when a caller can mutate data after handing it to another function, every reader has to reason about who else might change it. Immutability or defensive copy at the boundary.
- **Global mutable state** — singletons, statics, module-level mutable variables. Makes testing painful and reasoning non-local.
- **Temporary field** — a field that is only set in some scenarios. Usually means an extracted class is hiding inside.
- **Inconsistent state** — multiple fields that must be kept in sync but are updated independently. Either compute one from the other, or encapsulate the joint update.
- **Null/undefined as control flow** — using null returns to signal "not found" or "error". Optional/Maybe/Result types or specific exceptions are clearer.

## 5. Control Flow

- **Deep nesting** — more than 2–3 levels of nested conditionals or loops. Apply guard clauses, early returns, or extract.
- **Nested conditional with parallel structure** — repeated `if (foo) { ... } else if (bar) { ... }` chains across the codebase. Polymorphism or a dispatch map often replaces them.
- **Switch on type** — checking a type tag to dispatch behavior. Polymorphism is the canonical replacement.
- **Loop with multiple responsibilities** — a single loop that filters, transforms, and aggregates in one pass. The pass might be efficient, but readability suffers; split into a pipeline of built-in operations unless the loop is on a hot path (Rule 3).
- **Exception used for control flow** — throwing and catching as a `goto`. Exceptions are for exceptional conditions; expected outcomes should be ordinary return values.

## 6. Performance Signals

These are signals that a piece of code *might* be a performance problem. Whether it actually is depends on whether the code is hot — see Rule 3. Flag the signal regardless; let the user weigh the tradeoff.

- **Hand-rolled loop where a built-in primitive exists** (Rule 2 territory) — manual copies, manual filters, manual sums where the standard library has a single-call equivalent. Built-ins are usually faster *and* clearer.
- **Quadratic over a collection** — nested iteration where the inner loop searches the same collection repeatedly. Often replaceable by building a hash/set once outside the loop.
- **Repeated computation inside a loop** — the same expression (regex compilation, function lookup, attribute access chain) recomputed each iteration when it could be hoisted.
- **String concatenation in a loop** — in many languages this is O(n²) due to string immutability. Use the language's optimized string-builder primitive.
- **Allocation in a hot loop** — creating new objects, arrays, or closures inside an inner loop. Allocation pressure shows up as GC time.
- **Boxing/unboxing in hot paths** — converting between primitive and object representations repeatedly. Languages with this distinction (e.g., .NET, Java) pay a real cost.
- **Synchronous I/O in async context** — blocking the event loop or thread when the surrounding code expects non-blocking behavior. Hard to spot from snippets; flag it when surrounding context makes it visible.
- **N+1 access pattern** — issuing one query/fetch per item in a loop, when a batch operation is available. Classic in DB code, also appears with file/network access.
- **Eager work that could be lazy (or vice versa)** — computing the entire result set when only the first few items will be consumed; or, conversely, lazy chains where the materialization is paid for many times.
- **Wrong data structure** — linear scans where a hash lookup would do, or a hash where insertion order matters and a list/queue would suffice.

## 7. Testability Signals (legacy code radar)

These signals tell you whether the code can be safely refactored at all. If several apply, treat the code as legacy and shift to `legacy-techniques.md` for Phase 2.

- **Hidden dependencies** — `new ConcreteThing()` inside a method, hardcoded singletons, direct `DateTime.Now`/`Math.random` calls, direct file/network/db access. Untestable until a seam is introduced.
- **Long constructor** — significant logic in the constructor (database calls, file reads, network requests). Cannot be instantiated for testing without the real dependencies.
- **Static method dependencies** — calling a static utility that itself has hidden dependencies. The static call is a hard link.
- **No tests visible, or tests that don't actually verify behavior** — green tests that exercise no real assertion give false confidence. Treat them as no tests.
- **Behavior depends on global state** — same input, different output depending on a static or singleton value somewhere else. Hard to characterize.

---

## How to use this list

1. Scan the code once through Sections 1–5 (design smells). Note candidates.
2. Scan again through Section 6 (performance signals). Note candidates.
3. Scan once more through Section 7 (testability signals) — this scan determines which Phase 2 reference file you load.
4. From your candidate list, **drop the ones whose cost is small for this specific code**. A Long Function in a 30-line script is not a finding; the same function in a 5000-line module is.
5. The survivors are your findings. Move to Phase 2.
