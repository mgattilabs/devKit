---
name: 'MemPalace memory rules'
description: 'Persistent memory via MemPalace CLI and Python API. Search before answering, save after deciding, operate silently.'
applyTo: '**'
---

# MemPalace — operating rules

You have CLI and Python API access to MemPalace: a local, verbatim memory store
organized as **wings** (projects / people), **rooms** (sessions / topics), and
**drawers** (exact text). It also exposes a temporal **knowledge graph** and
per-agent **diaries**.

Use it on every turn. Memory is not optional.

## Calling conventions

- **CLI** for ingestion, search, status, and wake-up:
  `mempalace search "query"`, `mempalace wake-up --wing <wing>`, `mempalace status`
- **Python API** for drawer CRUD, knowledge graph, diary, tunnels, and navigation:
  `python -c "from mempalace.mcp_server import <tool_function>; import json; print(json.dumps(<tool_function>(<args>), indent=2, default=str))"`

See the mempalace skill reference files for complete calling patterns.

## At session start

Run a wake-up scoped to the active wing (infer it from the workspace folder
name). Pull recent drawers and active knowledge-graph entities into context
before the first reply:

```bash
mempalace wake-up --wing <wing_name>
```

If no matching wing exists, do a cross-wing search on the workspace name
and the open file's top-level identifiers:

```bash
mempalace search "<workspace name>"
```

## Before any non-trivial answer

1. Search the palace for the topic of the user's request. This is mandatory
   whenever the user uses "we", "our", "before", "remember", "the project",
   or anything that implies prior context:
   ```bash
   mempalace search "<topic>"
   ```
2. Use cross-wing search when the topic spans projects (shared skills,
   recurring patterns, infrastructure decisions).
3. Treat retrieved drawers as authoritative. Quote the user's exact words
   when relevant. Never paraphrase past statements back to them.

## After any decision, design choice, or non-obvious finding

Append a new drawer in the appropriate wing/room:

```bash
python -c "
from mempalace.mcp_server import tool_add_drawer
import json
result = tool_add_drawer(wing='<wing>', room='<room>', content='''<verbatim content>''')
print(json.dumps(result, indent=2, default=str))
"
```

Store verbatim — never rewrite the user's words. Prepend a short structured
header (date, topic, related entities).

Update the knowledge graph when a person, project, library, or external
entity is introduced, changed, or invalidated:

```bash
python -c "
from mempalace.mcp_server import tool_kg_add
import json
result = tool_kg_add(subject='<entity>', predicate='<relationship>', object='<value>')
print(json.dumps(result, indent=2, default=str))
"
```

## Operate silently

- Do not narrate tool calls. No "let me check the palace", no "I'll save
  this to memory", no "based on what I found".
- Do not list the tools used or the drawers retrieved unless the user
  explicitly asks what you remember.
- Memory must feel native: context appears, decisions persist, no
  scaffolding shown.

## Never

- Never paraphrase or summarize when writing to a drawer.
- Never overwrite or delete a drawer — append-only is a hard rule of the
  system.
- Never fabricate context. If a search returns nothing, say so plainly and
  proceed without inventing prior history.
- Never expose palace internals (raw drawer IDs, wing names the user did
  not introduce) unless asked.