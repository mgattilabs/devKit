---
description: Knowledge graph — add, query, invalidate temporal facts; view stats and timeline.
allowed-tools: Bash, Read
---

# Knowledge Graph

The knowledge graph stores temporal facts as subject → predicate → object triples.
All operations use the Python API.

## Add a fact

```bash
python -c "
from mempalace.mcp_server import tool_kg_add
import json
result = tool_kg_add(
    subject='<entity>',
    predicate='<relationship>',
    object='<value>',
    valid_from='<optional: ISO date>',
    source_closet='<optional: closet reference>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `subject`, `predicate`, `object`.

## Query an entity

```bash
python -c "
from mempalace.mcp_server import tool_kg_query
import json
result = tool_kg_query(
    entity='<entity name>',
    as_of='<optional: ISO date>',
    direction='<optional: in|out|both>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `entity`. Returns typed facts with temporal validity.

## Invalidate a fact

```bash
python -c "
from mempalace.mcp_server import tool_kg_invalidate
import json
result = tool_kg_invalidate(
    subject='<entity>',
    predicate='<relationship>',
    object='<value>',
    ended='<optional: ISO date>'
)
print(json.dumps(result, indent=2, default=str))
"
```

Required: `subject`, `predicate`, `object`. Marks a fact as no longer true.

## Stats

```bash
python -c "
from mempalace.mcp_server import tool_kg_stats
import json
print(json.dumps(tool_kg_stats(), indent=2, default=str))
"
```

Returns: entities, triples, current vs expired facts, relation types.

## Timeline

```bash
python -c "
from mempalace.mcp_server import tool_kg_timeline
import json
result = tool_kg_timeline(entity='<optional: entity name>')
print(json.dumps(result, indent=2, default=str))
"
```

Chronological view of facts. Omit `entity` for full timeline.
