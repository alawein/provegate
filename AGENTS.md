---
type: canonical
source: none
sync: none
sla: none
authority: canonical
audience: [agents, contributors, maintainers]
last_updated: 2026-04-16
last-verified: 2026-04-16
---

# AGENTS — Provegate

## Workspace identity

`provegate` is the repo for `claude-drift`, `claude-memory-mesh`, and
`claude-proof`, plus the shared types and CLI tooling that bind them together.

## Directory structure

- `claude-drift/`: intent parsing and drift checks
- `claude-memory-mesh/`: persistence and claim graph operations
- `claude-proof/`: verification chains and rollback
- `shared/`: canonical shared types
- `scripts/`: repo scanners and helpers
- `tests/`: regression and integration coverage
- `docs/`: architecture, deployment, operations, and prompt notes

## Governance rules

1. Changes to shared behavior must update tests.
2. Keep all three MCP servers runnable from repo root.
3. Do not hand-wave proof, memory, or drift semantics in docs or code.
4. Keep persistence, decay, and claim-state transitions explicit.
5. Do not introduce secrets or environment-specific credentials into source.

## Code conventions

- Python 3.10+
- FastMCP for server entrypoints
- Dataclasses and typed enums for the shared model
- Comments explain composition, persistence, or verification constraints
- Conventional commits only

## Build and test commands

```bash
uv pip install -e ".[dev]"
python -m pytest
ruff check .
python scripts/scan_repo.py .
```
