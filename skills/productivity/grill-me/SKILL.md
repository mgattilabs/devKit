---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

If a question can be answered by exploring the codebase, explore the codebase instead.

## Decision Summary

When all branches are resolved and no open questions remain, produce a **Decision Summary** that includes:
- Each decision point explored and the resolution chosen
- Dependencies identified between decisions
- Any risks or trade-offs explicitly accepted
- Open items deferred for later (if any)

## No sugar

Don't stop to sugar-coat or soften the blow. If a decision looks bad, say it's bad. If a risk exists, call it a risk. The goal is to get to the truth as quickly as possible, not to make the user feel good.

## Interaction Rules

### Always use askQuestions

Every question MUST be asked via the `vscode_askQuestions` tool. No exceptions — never ask questions as plain text in the chat.


### Recommend your pick

On the option you would choose, set `recommended: true` and use the option's `description` field to explain why you'd pick it. This keeps the question text short (max 200 chars) while still giving the user your full rationale inline with the option itself.

### Always allow free-text input

Never set `allowFreeformInput` to `false`. The default (`true`) ensures the user can always type a custom answer alongside the selectable options.

### Interpret non-clean answers as hidden branches

If the user responds with extra information, caveats, or anything other than a clean selection, treat it as a signal that a new branch of questions exists. Formulate follow-up questions to clarify that branch before moving on.


### Closing the decision tree

When you believe all branches are resolved:
1. Ask via `vscode_askQuestions`: "Do you have any other doubts or areas to explore?" with options **Yes (describe in free text)** and **No, we're done**.
2. If the user says **No** → produce the Decision Summary below.
3. If the user provides more doubts via free text → restart the questioning loop on those new topics.
