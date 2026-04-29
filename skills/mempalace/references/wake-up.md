---
description: Load palace context at session start — pull recent drawers and active KG entities into context.
allowed-tools: Bash, Read
---

# Wake-up

Load palace context at session start. Pulls L0 (status) and L1 (recent drawers, active KG entities) into context.

## CLI

```bash
mempalace wake-up
```

Scoped to a specific wing:

```bash
mempalace wake-up --wing <wing_name>
```

Output is ~600-900 tokens of context ready to inject.
