---
type: canonical
last_updated: 2026-06-29
---

# Repository topology

**Archetype:** `python-agent-service`

Three composable FastMCP servers for drift detection, graph memory, and
proof-gated verification. Each server is a top-level package with a single
`server.py` entry; shared types live in `shared/`.

## On-disk layout

```text
provegate/
├── claude-drift/
│   └── server.py
├── claude-memory-mesh/
│   └── server.py
├── claude-proof/
│   └── server.py
├── shared/
│   ├── __init__.py
│   └── types.py
├── docs/
├── examples/
├── scripts/
├── tests/
├── website/
├── pyproject.toml
└── vercel.json
```

## Role boundaries

- `claude-drift/` scans source against architectural intent from CLAUDE.md, ADRs, or `.drift-rules.json`.
- `claude-memory-mesh/` owns the SQLite-backed claim graph and bi-temporal decay.
- `claude-proof/` wraps modifications in verification chains and promotes verified claims.
- `shared/types.py` is the single schema source for claim serialization across servers.
- `scripts/` holds standalone CLIs (e.g. repo scanner) that run without an MCP host.
- `website/` is the static marketing surface; `examples/` and `tests/` stay outside server packages.

## Related

- [architecture.md](../architecture.md): MCP tools, data flow, and deployment notes.
