---
description: Manage drawers — add, get, list, update, delete verbatim memories and check duplicates.
allowed-tools: Bash, Read
---

# Drawers

Drawers hold verbatim memories. All operations use the Python API.

## Add a drawer

```bash
python -c "
from mempalace.mcp_server import tool_add_drawer
import json
result = tool_add_drawer(
    wing='<wing>',
    room='<room>',
    content='''<verbatim content>''',
    source_file='<optional: origin file>',
    added_by='<optional: default mcp>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `wing`, `room`, `content`. Checks for duplicates automatically before filing.

## Get a drawer by ID

```bash
python -c "
from mempalace.mcp_server import tool_get_drawer
import json
result = tool_get_drawer(drawer_id='<drawer_id>')
print(json.dumps(result, indent=2, default=str))
"
```

## List drawers

```bash
python -c "
from mempalace.mcp_server import tool_list_drawers
import json
result = tool_list_drawers(wing='<optional>', room='<optional>', limit=20, offset=0)
print(json.dumps(result, indent=2, default=str))
"
```

All parameters are optional. Returns IDs, wings, rooms, and content snippets.

## Update a drawer

```bash
python -c "
from mempalace.mcp_server import tool_update_drawer
import json
result = tool_update_drawer(
    drawer_id='<drawer_id>',
    content='<new content>',
    wing='<optional: move to different wing>',
    room='<optional: move to different room>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `drawer_id`. At least one of `content`, `wing`, `room` must be provided.

## Delete a drawer

```bash
python -c "
from mempalace.mcp_server import tool_delete_drawer
import json
result = tool_delete_drawer(drawer_id='<drawer_id>')
print(json.dumps(result, indent=2, default=str))
"
```

**Irreversible.** Required: `drawer_id`.

## Check for duplicates

```bash
python -c "
from mempalace.mcp_server import tool_check_duplicate
import json
result = tool_check_duplicate(content='<content to check>', threshold=0.15)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `content`. Optional: `threshold` (default 0.15 — lower = stricter).
