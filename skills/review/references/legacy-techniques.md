# Legacy Techniques (Phase 2b — code without tests)

Use this reference when the code under review **has no tests, or test coverage is unknown**. Working without a safety net changes the rules: you cannot apply Fowler-style transformations aggressively, because there is nothing to catch a behavior change. This section is distilled from Feathers' *Working Effectively with Legacy Code*.

The defining property of legacy code, in Feathers' definition, is the absence of tests. Age, style, and language are irrelevant — code written yesterday with no tests is legacy. Code written in 1995 with thorough tests is not.

---

## The Legacy Code Dilemma

To refactor safely you need tests. To write tests you often need to refactor (because the code is not testable as-is). This circular dependency is the central problem this reference solves.

The resolution is **minimal-risk edits**: techniques that introduce just enough seam — a place where behavior can be altered without editing in place — to allow a test, without applying any aggressive transformation that could change behavior.

Once tests exist, you graduate back to `refactoring-transformations.md` and apply Fowler transformations with the safety net in place.

---

## What to Recommend in the Report

When reviewing legacy code, the report's Proposed Fixes section should normally include:

1. **A characterization test recommendation** — even if you cannot write the test for the user, name what the test should pin down (the inputs, the expected outputs based on current observed behavior).
2. **One or more minimal-risk edits** that introduce seams or sprouts to enable the testing without disturbing existing behavior.
3. **The aggressive refactor** as a *deferred* item — reference it but mark it `[OUT-OF-SCOPE for this review until tests exist]`.

Do not propose the aggressive refactor as the main fix when no tests exist. That is "edit and pray" and a Principal Engineer does not endorse it.

---

## Characterization Tests

A characterization test pins down what the code **currently does**, not what it *should* do. The point is to capture present behavior — bugs and all — so that future refactors cannot silently change it. If the present behavior is wrong, that is a separate finding (`[BUG]`), addressed separately, after the refactor is complete.

The mechanical recipe:

1. Pick an entry point (a function, a method).
2. Call it with a representative input.
3. Write the test asserting an obviously wrong expected output (e.g., "this should return 42", knowing it won't).
4. Run the test. The actual output appears in the failure message.
5. Update the test to assert that actual output. The test now passes and pins behavior.
6. Repeat for edge cases: nulls, empty inputs, maximum values, error conditions, the input that triggered the bug originally.

Over time, accumulated characterization tests form the safety net you needed.

When recommending characterization tests in the report, name the input classes the user should pin: happy path, each documented edge case, observed error paths. Do not write speculative behavior — characterize only what you can see or what the user has confirmed.

---

## Seams: where to alter behavior without editing in place

A **seam** is a place in the code where behavior can be changed without modifying that exact line. Seams are what make hidden dependencies testable. The common seam types, language-agnostic:

- **Object seam** — a virtual/overridable method can be subclassed and overridden. The most common seam in OO languages.
- **Parameter seam** — a dependency passed as a parameter can be substituted with a fake.
- **Function-pointer / callback seam** — a callable held as a value can be swapped.
- **Link/build seam** — at link time or build time, a real implementation can be swapped for a fake. Useful when the dependency is a free function or a static.

The first move in legacy work is usually to **introduce a seam where none exists**. The techniques below are how you do that with minimal risk.

---

## Minimal-Risk Edits to Introduce Seams

### Sprout Method

- **When**: you need to add new behavior to an existing untested method, and you do not want to touch the existing code at all.
- **How**: write the new behavior as a separate, testable method. Test it in isolation. Then add a single call to it from inside the existing method.
- **Risk**: very low — the existing code is untouched except for the new call site.
- **Tradeoff**: the existing method is still untested. You did not improve its coverage; you only avoided making it worse.

### Sprout Class

- **When**: same as Sprout Method, but the new behavior is large enough to deserve its own class, or the existing class is too tangled to extend.
- **How**: build the new class, fully tested. Instantiate and call into it from the existing untested code.
- **Risk**: very low.

### Wrap Method

- **When**: you need to change what happens *around* an existing untested method (logging, additional behavior before/after, monitoring) without touching its body.
- **How**: rename the original method to something internal (`_originalDoStuff`), create a new method with the original name that calls the renamed one and adds the new behavior. The wrapper is testable; the inner is unchanged.
- **Risk**: low. All callers continue to call the original name and get the new wrapped behavior.

### Wrap Class

- **When**: similar to Wrap Method, but at the class level — you need to interpose behavior across many methods.
- **How**: create a new class that holds an instance of the original and forwards calls, adding behavior as needed. Callers depend on the wrapper.
- **Risk**: low–medium depending on how callers obtain the instance.

### Extract and Override Call

- **When**: a method has a problematic call inside it (a static call, a `new` expression, a global access) that you cannot easily replace with a parameter.
- **How**: extract the problematic call into its own protected/virtual method. In tests, subclass the original class and override that method to return a controlled value.
- **Risk**: low. Existing behavior is preserved (the new method does exactly what the call did); you have created a seam.

### Extract and Override Factory Method

- **When**: a constructor or method directly instantiates a hard-to-fake object (`new RealDatabase()`).
- **How**: replace the `new` with a call to a `createDatabase()` factory method on the same class. Override it in test subclasses to return a fake.
- **Risk**: low.

### Extract Interface (when supported by the language)

- **When**: a class depends on another concrete class that you cannot fake without changing types.
- **How**: define an interface with the methods the dependent code uses. Have the concrete class implement the interface. Have the dependent code declare against the interface. Now a test fake can implement the interface.
- **Risk**: low. No behavior change; only type signatures shift.

### Parameterize Constructor / Method

- **When**: a class instantiates its dependencies internally; you want to inject them.
- **How**: add a constructor (or method) overload that accepts the dependency. The original constructor calls the new one with the production default. Existing callers are unaffected; tests use the new constructor.
- **Risk**: low if backwards-compatible defaulting is preserved.

### Encapsulate Global Reference

- **When**: code reads or writes a global/static (`Logger.global`, `DateTime.Now`, `Math.random`).
- **How**: introduce a small class that wraps the global and exposes the operations the code uses. Inject this wrapper. In tests, supply a fake wrapper.
- **Risk**: low. The global call still happens — just funneled through the new class.

### Adapt Parameter

- **When**: a method takes a parameter of a hard-to-fake type (e.g., a heavy framework object), and you cannot or should not extract an interface.
- **How**: introduce a small adapter type that exposes only the operations the method uses. The method takes the adapter; production passes a real-backed adapter, tests pass a fake adapter.
- **Risk**: low.

---

## The Workflow for a Legacy Review

When the report deals with legacy code:

1. **Map the change you actually need to make** — refactor or new feature. State it explicitly in the report.
2. **Identify the dependencies that block testing.** Each blocker becomes a finding.
3. **Pick the minimum seam-introduction technique** for each blocker. Prefer Sprout/Wrap (they touch the existing code least) over Extract Interface or Parameterize (they touch more sites).
4. **Recommend characterization tests** for the existing behavior, listing input classes to pin.
5. **Once the seams exist and tests pin behavior**, the aggressive refactor becomes safe. State this explicitly: *"After the seams in F1, F2 are introduced and characterization tests are written, the refactor described in F4 (out-of-scope here) becomes safe."*

This is slower than "just refactor it" — and it is the only honest path when no tests exist. A review that recommends aggressive refactors on untested code is recommending production incidents.

---

## Anti-patterns specific to legacy work

- **"It's simple, I can refactor it without tests."** This sentence has caused more outages than any other phrase in software. The simplicity of the code is not the issue; the absence of a safety net is.
- **"Let me just write a test that covers the whole module."** Module-level tests on legacy code are slow, brittle, and rarely catch behavior changes precisely. Characterize the small entry point you actually need to change.
- **"I'll fix the bug while I'm in there."** Combining a refactor with a behavior change destroys your ability to attribute any later regression. Refactor first (no behavior change), then fix the bug (deliberate behavior change), in two separate commits.
- **"The tests pass, so the refactor is safe."** Only true if the tests actually exercise the behavior you changed. Coverage is a necessary, not sufficient, condition.
