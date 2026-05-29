# Modes

## DDD concepts — plain language

Use these concepts as a thinking lens during the grill. When explaining them to the user, use these definitions — not academic DDD jargon.

**Ubiquitous Language**: the shared vocabulary of the project. Every concept gets one name, and everyone — devs, stakeholders, agents — uses that name. If two people use different words for the same thing, bugs happen at the boundary. CONTEXT.md is where this vocabulary lives.

**Bounded Context**: the boundary within which a word has one precise meaning. "Account" in accounting is a different thing from "Account" in login. When a word changes meaning, you've crossed into a different context. The bounded context tells you where your vocabulary ends.

**Aggregate**: the group of things that must stay consistent together. An order with its line items: if you add a line, the total must update. You can't touch a line without the order knowing. The order + its lines is the aggregate — the smallest unit that must remain consistent. If you change the customer's address, the order doesn't need to know. The customer is a different aggregate.

**Domain Event**: something that just happened that other parts of the system care about. "An order was placed" — who needs to know? Shipping? Billing? Notifications? The event is the announcement; listeners decide what to do with it.

In practice: **bounded context** tells you where your vocabulary ends, **aggregate** tells you what must change together, **ubiquitous language** tells you what to call things, and **domain events** tell you how the pieces talk to each other.

---

## New project mode

Trigger: user brings a brief, requirements, stakeholder notes, or an idea.

### Flow

1. **Read everything the user provides.** Brief, notes, transcripts, whatever. Don't ask for more yet.
2. **Open with the domain, not the solution.** First question is always about the problem space, never about technology. "Before we talk about how — what problem does this solve? For whom? What happens today without this system?"
3. **Map the nouns.** Listen for domain terms. Every noun is a candidate for the glossary. Challenge each one: "You said 'customer' and 'user' — same thing or different?"
4. **Map the verbs.** "A customer places an order" — what does 'places' mean? What changes? Who gets notified? What can go wrong?
5. **Find the boundaries.** "Where does this system end and something else begins? What's NOT your problem?" This reveals bounded contexts.
6. **Challenge scope.** For every feature mentioned, ask: "Is this in scope for the first version? Why?" Capture the "no" decisions.
7. **Produce artifacts.** When enough ground is covered (you'll know because new questions circle back to resolved concepts):
   - Write CONTEXT.md with the glossary
   - Write ADRs for the decisions that emerged
   - Write specs for the features in scope, each with task decomposition and test criteria

### Red flags to probe

- User jumps to technology before defining the problem → pull back
- User uses the same word for different things → force a choice
- User says "it's obvious" or "we'll figure it out later" → that's where the bugs live
- User can't describe what happens without the system → the problem isn't understood yet

## Legacy bootstrap mode

Trigger: existing codebase, no CONTEXT.md or ADR.

### Flow

1. **Explore the codebase first.** Read the project structure, entry points, main files. Form a mental map before asking anything.
2. **If Graphify output exists** (`graphify-out/` or similar), read `GRAPH_REPORT.md` first. Use the graph as evidence for your questions.
3. **Open with what you found.** "I see OrderService has 15 dependencies. I see UserAccount is used in 3 different meanings. Here's what looks odd to me — let me ask about each."
4. **The graph is the witness, you are the prosecutor.** Use structural evidence to ask hard questions:
   - God nodes (high connectivity): "Which of these responsibilities genuinely belong here?"
   - Surprising couplings: "Why does ReportGenerator depend on UserSession?"
   - Orphan modules: "Nothing calls this. Dead code, WIP, or missing integration?"
   - Naming conflicts: "Account means 3 different things — which is canonical?"
5. **Distill, don't catalog.** The goal is not to document every class. It's to extract the domain model that's buried in the code. What are the real concepts? What are the real boundaries? What decisions were made (intentionally or by accident)?
6. **Produce artifacts.** Same outputs as new project mode, plus:
   - `docs/FLAGGED_FOR_SPEC.md` — problems discovered that need future specs

### What NOT to do

- Don't try to document everything. Document what's surprising, what's ambiguous, what would confuse a new developer or an agent.
- Don't judge the code quality. You're extracting domain knowledge, not doing a code review.

## Evolution mode

Trigger: existing codebase WITH CONTEXT.md and/or ADRs already present.

### Flow

1. **Read existing docs first.** CONTEXT.md, all ADRs, all specs. This is your baseline.
2. **Read relevant code.** Check if the code matches the docs. Surface contradictions.
3. **Grill against the glossary.** When the user uses a term that conflicts with CONTEXT.md, call it out immediately.
4. **Update inline.** When a term is resolved or a decision is made, update the relevant doc right there. Don't batch updates.

## Spec mode

Trigger: user has a specific feature or change request, docs already exist.

### Flow

1. **Read CONTEXT.md and relevant ADRs.** Understand the current domain model and decisions.
2. **Grill the feature request.** What problem does it solve? What changes in the domain? Does it introduce new terms? Does it contradict existing ADRs?
3. **Write the spec.** Problem, solution, acceptance criteria as test names, task decomposition.
4. **Update CONTEXT.md if needed.** New terms? Changed relationships? Do it now, as a side-effect.
5. **Flag ADR candidates.** If the feature forces a new architectural decision, write the ADR.
