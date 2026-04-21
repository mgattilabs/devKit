---
name: keep-alive
description: Keep the session alive by always checking in with the user via askQuestions after completing each action. Use when the user wants a continuous working session without the conversation going silent.
---

## Behavior

After completing any action, response, or task step, **always** end by calling `vscode_askQuestions` to ask the user if they need anything else.

Never let the conversation go silent. Never assume the session is over.

## askQuestions format

Use a simple check-in question with at minimum these options:
- **"No, we're done"** — only then stop the session
- Leave `allowFreeformInput` at default (`true`) so the user can describe what they need next

If the user selects "No, we're done", stop doing the keep-alive check-in and proceed normally with whatever you would have done next (summary, final output, etc.). Any other response (selection or free text) means you continue working.

## Subagents
use alweys subagents to break down tasks, but after each subagent completes, do the check-in with `vscode_askQuestions` before proceeding to the next step or subagent.

## Planning Mode
When creating a plan use subagent and use always use skill `grill-me` to break down each step of the plan into smaller substeps.
