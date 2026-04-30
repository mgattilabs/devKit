# Copilot Instructions — MemPalace Memory System

You have access to MemPalace, a persistent memory system.
Use it as durable memory and retrieval, not as optional note-taking.

Raw chat capture is handled by hooks. Do not treat the model as the primary verbatim recorder. Use MemPalace to retrieve prior context, keep project/wing structure consistent, and file additional durable context when structured storage is useful.

## Mandatory Setup Gate

At the start or resumption of ANY session, complete these steps in order:

1. Read this instruction file fully and treat it as an execution gate, not background guidance.
2. Bootstrap the chat session by loading palace context using the MemPalace status and discovery tools exposed in the current environment.
3. Identify the current wing and keep it fixed for the session.
4. Inspect the available MemPalace tools before attempting memory reads or writes.

If any required setup step cannot be completed, do not continue as though setup succeeded.
State which step is blocked, why it is blocked, and what viable fallback or next action is available.

In the first progress update of a task, explicitly confirm whether setup is complete.

Note: tool names may be exposed with MCP namespace prefixes in some environments. Use the MemPalace status, wing listing, search, KG, drawer, duplicate-check, and diary tools that are actually available instead of assuming an exact literal tool name.

## Memory Protocol

1. ON WAKE-UP / SESSION START / SESSION RESUME: call the MemPalace status tool once to bootstrap active palace context, then call the wing-listing tool. Do not call status on every message. Re-check status only when resuming after interruption, after a configuration change, or when palace state is genuinely uncertain.
2. BEFORE RESPONDING about any person, project, past event, prior decision, or user preference: call MemPalace search, or KG query, FIRST. Never guess. Unless the fact is directly available in the current session context.
3. IF UNSURE about a fact: say "let me check" and query the palace.
4. WHEN FACTS CHANGE: invalidate the old fact if needed, then add the new fact.
5. TO SAVE MEMORIES: use drawer filing for durable context that should remain queryable as a first-class memory item. Prefer verbatim content when storing decisions, preferences, key repro steps, or exact user wording that matters later. Summaries and diary entries are supplementary and must not replace drawer filing when durable structured recall is needed.

## Wing Discipline

The palace uses wings to separate projects. Every save must include the correct wing.
To determine the wing for this session:

1. Call the wing-listing tool first and prefer an existing matching wing if one clearly corresponds to the current project.
2. If there is no established wing, derive one from the workspace folder name using a stable lowercase slug.
3. Use this wing consistently for all saves in the session.
4. Never mix wings within one task unless the user is explicitly working across projects.

## Clean Chat Rule

Do not inform the user in every response that you are 'checking mempalace storage' or similar. Just do it. The user can see the commands that are run as part of the chat environment and will know. Similarly, after reading instructions, do not state 'I read the instructions', or similar, in every response. Efficient transparency is good, but excessive transparency is noise. Use your judgment to strike the right balance.
The one explicit exception is the bootstrap step at the start of the session where you confirm whether setup is complete.

## Save Strategy

Hook-exported transcripts are mined by MemPalace outside the agent. Use MemPalace saves for durable structured recall, not as a duplicate dump of everything.

Good candidates for drawer filing:

- exact user preferences that will matter later
- key repro steps or observations worth retrieving independently of the full transcript
- confirmed decisions, constraints, and resolved ambiguities
- facts whose old values may later need invalidation

Do not skip a useful durable save merely because duplication is possible, but also do not create redundant manual saves for context that the hook-captured transcript already preserves unless separate retrieval value is clear.

## Plans (Structured Plan Memory)

MemPalace should be used to store structured plans (project plans, implementation plans, runbooks) as first-class memory items when they have long-term retrieval value.

Guiding rules
- Always save plans into the `plans` drawer for the current wing. Use the drawer filing tool and include structured metadata in YAML frontmatter so plans are both human-readable and machine-queryable.
- Before filing, run the duplicate-check tool with the plan's `title`, `tags`, `wing`, `repo` and a short content `hash` to detect highly similar existing plans. If duplicates are found, surface the top matches to the user and offer to link, supersede, or create a new plan.
- Default plan persistence is `session` (draft) until the user explicitly confirms persistent storage. Never persist a plan containing PII or secrets without explicit user confirmation and redaction.

Metadata schema (recommended)
- `id`: stable unique identifier (UUID or slug)
- `title`: short descriptive title
- `wing`: wing slug (required)
- `repo`: repository name or workspace folder
- `project_slug`: optional project identifier
- `status`: one of `draft`, `active`, `completed`, `archived`
- `tags`: list of strings (include `plan`)
- `created_by`: agent or user id
- `created_at`: ISO8601 timestamp
- `updated_at`: ISO8601 timestamp
- `owners`: list of owners/persons responsible
- `related_files`: list of repo paths or file references
- `tasks`: ordered list of steps (short titles or structured items)
- `hash`: short content hash for duplicate detection
- `supersedes`: id of plan this one replaces (optional)
- `persistence`: `permanent` or `session`

Template example
```yaml
---
id: plan-2026-04-30-migration
title: "Database migration plan for X"
wing: backoffice
repo: remo-backoffice-api
project_slug: db-migration
status: draft
tags: [plan, migration, db]
created_by: MPlanAgent
created_at: 2026-04-30T12:00:00Z
owners: [team-db]
related_files: [src/migrations/README.md]
tasks:
	- "Export current schema"
	- "Create migration script"
	- "Run tests"
hash: "sha1:abcd1234"
persistence: session
---

TL;DR: ... (short summary)

1. Step one
2. Step two
3. Step three
```

Filing procedure
- Use the drawer filing tool (the environment's drawer API) to create a new item in room `plans` under the current wing. Include the YAML frontmatter above as the file's metadata block and the plan body as the drawer content.
- Run duplicate-check before filing. If similar plans exist, present them to the user with options: `link`, `supersede`, `create new`.
- On persistent save (persistence: permanent), require explicit user confirmation and confirm any redaction of PII.

Fallback
- If MemPalace is temporarily unavailable, write the plan to a session fallback path `/memories/session/plans/<slug>.md` and mark `persistence: session` in the metadata. Notify the user that the plan is in session-only storage and provide an action to promote it to permanent storage when MemPalace becomes available.

Search and retrieval
- Tag plans with `plan` plus contextual tags (`wing`, `repo`, `project_slug`) to make semantic search effective.
- For agent workflows, prefer semantic queries augmented by tag filters (e.g., `tags:plan wing:backoffice repo:remo-backoffice-api`).

Verification and lifecycle
- When a plan is updated and replaces a previous version, set `supersedes` to the older plan id and optionally archive the prior item (`status: archived`).
- Use `created_at`/`updated_at` for TTL/retention policies; review archival rules with data governance when retention/PII rules apply.

## Transcript Mining Rule

Hook-captured chat transcripts are exported and filed outside the agent.
The hook exports the verbatim session transcript as plain text, files one explicit long-form transcript drawer for that session under a separate `transcript.full.raw` source identity, and then runs normal MemPalace conversation mining on `transcript.txt`.

Do not create a parallel manual chat-summary workflow for normal transcript ingest.
For hook-captured chat history, prefer retrieving from:

- the explicit long-form transcript drawer in `chat_transcript_full` when exact history matters
- the mined conversation records when you want upstream MemPalace's normal chunked representation
- the closet layers when compact retrieval is enough

## Diary Rule

Diary entries are allowed as compressed session logs, and should be considered parallel to any other memories made through your environment.
Use diary writes for session-level recap only.

## Pre-Final Check

Before sending a final answer, verify all of the following:

1. I completed the mandatory setup gate or clearly reported the blocking step.
2. I queried MemPalace before stating facts about prior context, people, projects, or past decisions or the info is directly available in the current session context.
3. If I created a durable save, it used the correct wing and a retrieval-appropriate room.

## Key Tools

- status tool — palace overview, protocol, active storage path
- wing listing tool — discover existing project wings
- search tool — semantic retrieval
- KG query tool — entity relationships and stored facts
- drawer filing tool — durable verbatim memory storage
- duplicate-check tool — reduce duplicate filing
- diary tool — compressed session recap only