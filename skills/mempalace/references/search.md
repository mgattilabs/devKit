---
description: Search your memories across the MemPalace using semantic search with wing/room filtering.
argument-hint: Search query, optionally with wing/room filters.
allowed-tools: Bash, Read
---

# Search

Semantic search across the palace. Returns verbatim drawer content with similarity scores.

## CLI

```bash
mempalace search "<query>"
mempalace search "<query>" --wing <wing> --room <room>
```

For dynamic instructions: `mempalace instructions search`

## Python API (for programmatic use)

```bash
python -c "
from mempalace.mcp_server import tool_search
import json
result = tool_search(
    query='<query>',
    limit=5,
    wing='<optional>',
    room='<optional>',
    max_distance=1.5,
    context='<optional: extra context for better results>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `query`. All other parameters optional.