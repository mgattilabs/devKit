---
name: mempalace
description: >
  MemPalace — full AI memory system: mine projects, search memories, manage drawers, knowledge graph, diary, tunnels, and navigation.
  Use when asked about mempalace, memory palace, mining memories, searching memories, palace setup, drawers, knowledge graph, diary, or tunnels.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# MemPalace

A searchable memory palace for AI — mine projects and conversations, then search them semantically.
Manage drawers, knowledge graph, diary, tunnels, and palace navigation via CLI and Python API.

## Prerequisites

Ensure `mempalace` is installed:

```bash
mempalace --version
```

If not installed:

```bash
pip install mempalace
```

## Two calling modes

### CLI commands (for ingestion, search, status)

```bash
mempalace <command> [args]
```

Available CLI commands: `init`, `mine`, `search`, `wake-up`, `status`, `compress`, `split`, `repair`.

### Python API (for drawer CRUD, knowledge graph, diary, tunnels, navigation)

Operations that have no CLI equivalent are called directly via the Python API:

```bash
python -c "
from mempalace.mcp_server import <tool_function>
import json
result = <tool_function>(<args>)
print(json.dumps(result, indent=2, default=str))
"
```

All `tool_*` functions live in `mempalace.mcp_server` and return dicts.

## Dynamic instructions

MemPalace provides dynamic instructions via the CLI for core operations:

```bash
mempalace instructions <command>
```

Where `<command>` is one of: `help`, `init`, `mine`, `search`, `status`.

## Reference commands

### Core (CLI)
- [help](references/help.md) — Full help, architecture, and available commands
- [init](references/init.md) — Initialize a new palace
- [mine](references/mine.md) — Mine projects and conversations
- [search](references/search.md) — Semantic search with wing/room filtering
- [status](references/status.md) — Palace overview and stats
- [wake-up](references/wake-up.md) — Load palace context at session start

### Drawers (Python API)
- [drawers](references/drawers.md) — Add, get, list, update, delete drawers and check duplicates

### Knowledge Graph (Python API)
- [knowledge-graph](references/knowledge-graph.md) — Add, query, invalidate facts; stats and timeline

### Diary (Python API)
- [diary](references/diary.md) — Read and write per-agent diary entries

### Tunnels & Navigation (Python API)
- [tunnels](references/tunnels.md) — Create, delete, list, find, and follow cross-wing tunnels
- [navigate](references/navigate.md) — List wings/rooms, traverse graph, get taxonomy
