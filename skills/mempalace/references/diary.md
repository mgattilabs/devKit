---
description: Per-agent diary — write observations and read back past entries in AAAK format.
allowed-tools: Bash, Read
---

# Diary

Per-agent diary entries stored in AAAK compressed format.
All operations use the Python API.

## Write a diary entry

```bash
python -c "
from mempalace.mcp_server import tool_diary_write
import json
result = tool_diary_write(
    agent_name='<agent identifier>',
    entry='<diary text>',
    topic='<optional: topic tag>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `agent_name`, `entry`. Use to record observations, learnings, and session summaries.

## Read diary entries

```bash
python -c "
from mempalace.mcp_server import tool_diary_read
import json
result = tool_diary_read(
    agent_name='<agent identifier>',
    last_n=10
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `agent_name`. Optional: `last_n` (default 10).
