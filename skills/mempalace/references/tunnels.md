---
description: Cross-wing tunnels — create, delete, list, find, and follow connections between wings.
allowed-tools: Bash, Read
---

# Tunnels

Tunnels are explicit cross-wing connections linking related content in different projects.
All operations use the Python API.

## Create a tunnel

```bash
python -c "
from mempalace.mcp_server import tool_create_tunnel
import json
result = tool_create_tunnel(
    source_wing='<wing>',
    source_room='<room>',
    target_wing='<wing>',
    target_room='<room>',
    label='<optional: description>',
    source_drawer_id='<optional>',
    target_drawer_id='<optional>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `source_wing`, `source_room`, `target_wing`, `target_room`.

## Delete a tunnel

```bash
python -c "
from mempalace.mcp_server import tool_delete_tunnel
import json
result = tool_delete_tunnel(tunnel_id='<tunnel_id>')
print(json.dumps(result, indent=2, default=str))
"
```

Required: `tunnel_id`.

## List tunnels

```bash
python -c "
from mempalace.mcp_server import tool_list_tunnels
import json
result = tool_list_tunnels(wing='<optional: filter by wing>')
print(json.dumps(result, indent=2, default=str))
"
```

## Find tunnels between wings

```bash
python -c "
from mempalace.mcp_server import tool_find_tunnels
import json
result = tool_find_tunnels(wing_a='<optional>', wing_b='<optional>')
print(json.dumps(result, indent=2, default=str))
"
```

Find rooms that bridge two wings.

## Follow tunnels from a room

```bash
python -c "
from mempalace.mcp_server import tool_follow_tunnels
import json
result = tool_follow_tunnels(wing='<wing>', room='<room>')
print(json.dumps(result, indent=2, default=str))
"
```

Required: `wing`, `room`. Returns connected rooms in other wings.
