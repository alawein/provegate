# Provegate

Status:      active
Category:    ventures
Owner:       alawein
Visibility:  public
Purpose:     Agent MCP tooling for drift control, memory, and proof-oriented orchestration.
Next action: continue

Three MCP servers for AI coding agents: drift detection, evidence-backed memory,
and verified change chains.

[![CI](https://github.com/alawein/provegate/actions/workflows/ci.yml/badge.svg)](https://github.com/alawein/provegate/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Abstract

AI coding tools are stateless by session, blind to architectural intent, and
unable to distinguish a tested invariant from a hallucinated guess. Provegate
addresses repeated reasoning, contradictory decisions, and invisible
architectural drift with three composable MCP servers:

```
claude-drift        detects divergence between intent and implementation
claude-memory-mesh  persists what agents believe (with evidence and decay)
claude-proof        verifies changes and gates what enters memory
```

`claude-drift` reads intent from CLAUDE.md, ADRs, or `.drift-rules.json` and
flags violations. `claude-memory-mesh` stores graph-structured claims with
evidence, typed edges, and decay. `claude-proof` wraps edits in verification
chains and promotes verified claims into memory.

## Status

- Lifecycle: `active`
- Visibility: `public`
- Homepage: [provegate.online](https://provegate.online)

## Runtime requirements

- Python 3.10 or newer
- `fastmcp` for MCP server wiring
- SQLite for memory mesh storage (no external DB setup)

```bash
pip install fastmcp

claude mcp add claude-drift -- python /path/to/claude-drift/server.py
claude mcp add claude-memory-mesh -- python /path/to/claude-memory-mesh/server.py
claude mcp add claude-proof -- python /path/to/claude-proof/server.py
```

Mark intents with `<!-- drift:intent -->` blocks in CLAUDE.md or structured
`.drift-rules.json` rules.

## Reproducibility

Development on provegate source:

```bash
pip install -e ".[dev]"
pytest
ruff check .
```

Scan any repo for drift without MCP setup:

```bash
python scripts/scan_repo.py /path/to/repo
python scripts/scan_repo.py https://github.com/org/repo
python scripts/scan_repo.py . --scope src/auth
python scripts/scan_repo.py . --json
```

## Datasets

- `examples/`: worked MCP and drift-rule examples
- `shared/`: shared types and utilities across MCP servers
- SQLite stores created at runtime by memory mesh (not committed)

## Docs map

- [docs/README.md](docs/README.md)
- [SSOT.md](SSOT.md)
- [LESSONS.md](LESSONS.md)
