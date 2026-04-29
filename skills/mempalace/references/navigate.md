---
description: Palace navigation — list wings/rooms, traverse the graph, get taxonomy and connectivity stats.
allowed-tools: Bash, Read
---

# Navigate

Explore the palace structure: wings, rooms, taxonomy, and graph traversal.
All operations use the Python API.

## List wings

```bash
python -c "
from mempalace.mcp_server import tool_list_wings
import json
print(json.dumps(tool_list_wings(), indent=2, default=str))
"
```

Returns all wings with drawer counts.

## List rooms

```bash
python -c "
from mempalace.mcp_server import tool_list_rooms
import json
result = tool_list_rooms(wing='<optional: filter by wing>')
print(json.dumps(result, indent=2, default=str))
"
```

## Get taxonomy

```bash
python -c "
from mempalace.mcp_server import tool_get_taxonomy
import json
print(json.dumps(tool_get_taxonomy(), indent=2, default=str))
"
```

Full taxonomy tree: wing → room → drawer count.

## Traverse graph

```bash
python -c "
from mempalace.mcp_server import tool_traverse_graph
import json
result = tool_traverse_graph(start_room='<room>', max_hops=2)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `start_room`. Walk the palace graph to see connected ideas across wings.

## Graph stats

```bash
python -c "
from mempalace.mcp_server import tool_graph_stats
import json
print(json.dumps(tool_graph_stats(), indent=2, default=str))
"
```

Palace graph overview: total rooms, tunnel connections, edges between wings.

## AAAK spec

```bash
python -c "
from mempalace.mcp_server import tool_get_aaak_spec
import json
print(json.dumps(tool_get_aaak_spec(), indent=2, default=str))
"
```

Get the AAAK compressed memory dialect specification.
