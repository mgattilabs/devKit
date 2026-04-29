---
description: Show the current state of your memory palace — wings, rooms, drawer counts, and suggestions.
allowed-tools: Bash, Read
---

# Status

Palace overview: total drawers, wing and room counts, protocol, AAAK dialect.

## CLI

```bash
mempalace status
```

For dynamic instructions: `mempalace instructions status`

## Python API

```bash
python -c "
from mempalace.mcp_server import tool_status
import json
print(json.dumps(tool_status(), indent=2, default=str))
"
```

Returns: total_drawers, wings (with counts), rooms (with counts), palace_path, protocol, aaak_dialect.