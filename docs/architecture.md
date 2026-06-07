---
type: canonical
owner: platform-engineering
last-reviewed: 2026-03-31
---

# Architecture Overview -- provegate

Provegate is three composable FastMCP servers that give AI coding agents
drift detection, graph-structured memory, and proof-gated verification.
They communicate via the Model Context Protocol and share domain types
defined in `shared/types.py`.

## Components

**claude-drift** (`claude-drift/server.py`) -- reads architectural intent
from `CLAUDE.md`, ADRs, or `.drift-rules.json`; derives enforceable rules
from natural language; scans source files for violations; returns a drift
score and per-violation report. Tools: `scan_intents`, `check_drift`,
`will_this_drift`, `check_drift_for_changes`, `declare_intent`, `export_rules`.

**claude-memory-mesh** (`claude-memory-mesh/server.py`) -- SQLite-backed
claim graph. Claims are revisable beliefs with confidence, evidence, and
scope. Typed edges (supports / contradicts / supersedes) link claims. Each
claim carries bi-temporal timestamps; unused claims decay automatically by
type (observations 30 days, decisions indefinite, failures 60 days). Tools:
`store_claim`, `before_modifying`, `record_decision`, `record_failure`,
`query_claims`, `invalidate_for_file`, `add_relationship`, `run_decay`,
`memory_stats`.

**claude-proof** (`claude-proof/server.py`) -- wraps code modifications in
verification chains. Each chain declares intent, records checkpoints, runs
test steps, and on success promotes verified claims into claude-memory-mesh.
Rollback targets the most recent checkpoint. Tools: `begin_modification`,
`checkpoint`, `verify_step`, `rollback`, `finalize_proof`, `quick_verify`,
`list_active_proofs`.

**shared** (`shared/types.py`) -- domain model used by all three servers.
This is the single source of truth for claim schemas and serialization.

**scripts** (`scripts/scan_repo.py`) -- standalone CLI scanner; runs
claude-drift against any local or remote repo without an MCP host.

## Data Flow

```
Code change
  --> claude-proof: begin_modification / checkpoint / verify_step
  --> on success: finalize_proof outputs verified claims
  --> claude-memory-mesh: store_claim records them in the graph
  --> claude-drift: check_drift reads intents from CLAUDE.md + memory graph
                    and scans changed files for violations
```

An agent calling `before_modifying(file)` before editing receives
active constraints, past failures, and relevant decisions from the graph,
reducing repeated reasoning across sessions.

## Dependencies

Runtime dependency: `fastmcp>=2.0.0`. Python 3.10+. No external services.
Persistence: SQLite (file per server, co-located with server directory).
All three servers are independent processes; composition is through
sequential MCP tool calls from the agent, not inter-server network calls.

## Constraints

- Path handling must be cross-platform. All file references use
  `pathlib.Path` internally.
- Claim validation is intentionally strict: unverified claims do not enter
  shared memory.
- Servers are stateless across restarts except for the SQLite files.
- No authentication layer -- servers are intended for local developer use
  or a single-agent context. Do not expose over a public network.
