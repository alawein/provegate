---
type: canonical
source: none
sync: none
sla: none
authority: canonical
audience: [ai-agents, contributors]
last_updated: 2026-05-24
last-verified: 2026-05-24
---

# CLAUDE.md: Provegate

## Workspace identity

`provegate` is an epistemic stack for coding agents: three composable FastMCP
servers that handle drift detection, graph memory, and proof-gated
verification.

Shared voice and workspace prompt:

- <https://github.com/alawein/alawein/blob/main/docs/style/VOICE.md>
- <https://github.com/alawein/alawein/blob/main/prompt-kits/AGENT.md>

## Directory structure

- `claude-drift/`: architectural intent parsing and drift analysis
- `claude-memory-mesh/`: SQLite-backed claim graph
- `claude-proof/`: proof chains, checkpoints, and rollback
- `shared/`: domain model and shared serialization types
- `scripts/`: CLI tools such as `scan_repo.py`
- `tests/`: unit and integration coverage
- `website/`: public-facing documentation surface
- `docs/`: architecture, deployment, prompt, and operations notes

## Governance rules

1. Treat `shared/types.py` as the single source of truth for the domain model.
2. Keep the three server directories co-located as siblings; composition logic
   depends on that layout.
3. Do not weaken proof or memory guarantees without updating tests and docs in
   the same change.
4. Keep path handling cross-platform and explicit.
5. Preserve the distinction between proposed claims, verified claims, and
   expired or contested claims.

## Code conventions

- Python 3.10+
- FastMCP servers with minimal runtime dependencies
- Dataclasses plus explicit `to_dict()` serialization, not Pydantic
- Comments explain composition, verification, persistence, or portability
  constraints
- Keep CLI and MCP behavior deterministic by default

## Build and test commands

```bash
uv pip install -e ".[dev]"
python -m pytest
python -m pytest tests/test_drift.py
ruff check .
python scripts/scan_repo.py . --json
python claude-drift/server.py
python claude-memory-mesh/server.py
python claude-proof/server.py
```
