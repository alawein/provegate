---
**claude-drift: ** `scan_intents`, `declare_intent`, `check_drift`, `check_drift_for_changes`, `will_this_drift`, `export_rules`
**claude-memory-mesh: ** `store_claim`, `before_modifying`, `record_decision`, `record_failure`, `query_claims`, `invalidate_for_file`, `add_relationship`, `run_decay`, `memory_stats`
**claude-proof: ** `begin_modification`, `checkpoint`, `verify_step`, `rollback`, `finalize_proof`, `promote_claims`, `quick_verify`, `list_active_proofs`
**shared/types.py is the single source of truth for the domain model.** All three servers import from it. Key types: `Claim`, `Evidence`, `ClaimEdge`, `ArchitecturalIntent`, `DriftViolation`, `ProofArtifact`, `VerificationStep`. Each has a `.to_dict()` method for JSON serialization.
- **Claim lifecycle**: proposed -> verified/contested/expired/rejected. Claims need evidence to pass verification thresholds (defined in `THRESHOLDS` dict in memory-mesh).
- **Co-location requirement**: `promote_claims` falls back to loading `../claude-memory-mesh/server.py` relative to its own file. All three server directories must remain siblings under the same parent.
- **Cross-platform paths**: `_rel_posix()` normalizes file paths to forward slashes so analyzers work on Windows.
- **Decay rules**: Observations expire in 30 days, failures in 60, invariants in 90. Decisions never decay by time. Defined in the `DECAY` dict.
- **Git dependency**: claude-proof requires `git` on PATH. The `_git()` helper silently returns `(False, error)` on failure -- proof chains will appear to work but checkpoints won't persist.
- **Import detection**: `_extract_import()` handles ES module destructured imports (`from "x"`), CommonJS (`require("x")`), Python (`from x import` / `import x`), and Go. All analyzers share this function.
- **Intent sources**: CLAUDE.md, ADRs, `.drift-rules.json`, human declarations. Drift server parses natural language constraints via regex (not LLM) into typed rules: `import_boundary`, `layer_enforcement`, `prohibition`, `unstructured`.
- **Proof chains use git**: `begin_modification` creates a baseline commit, `checkpoint` creates tagged commits, `rollback` does `git reset --hard`.
- **Proof-to-memory bridge**: `promote_claims` in claude-proof imports the memory-mesh module via `sys.modules` (reuses the already-loaded instance) or loads it fresh via `importlib`. This makes the composition work both in-process (tests, CLI) and when both MCP servers are registered separately.
- **sys.path hack**: Each server does `sys.path.insert(0, parent.parent)` to import `shared.types`. This means servers must be run from the repo root or with the repo on PYTHONPATH.
Each server is a standalone FastMCP application with in-process state: 
They compose: `claude-proof` produces verified claims that promote into `claude-memory-mesh`; `claude-drift` checks intents sourced from CLAUDE.md files and the memory graph.
Three composable MCP servers (FastMCP) that give AI coding agents justified, revisable beliefs about a codebase: 
claude-drift/server.py   # MCP server: intent parsing + drift analysis
claude-memory-mesh/server.py  # MCP server: SQLite-backed claim graph
claude-proof/server.py   # MCP server: verification chains + checkpoints
scripts/scan_repo.py     # CLI: scan local/remote repos for drift
sla: none
source: none
sync: none
type: canonical
| claude-memory-mesh | SQLite via `_db()` | `~/.claude/memory-mesh.db` (override: `MEMORY_MESH_DB` env var) |
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


## Quick Start (First 5 Minutes)

```bash
pip install -e ".[dev]"     # Install in development mode
python -m pytest            # Run tests
python -m pytest tests/test_core.py -v  # Run specific test
```

---

## Build and Run

```bash
# Install dependencies (fastmcp is the only runtime dep)
uv pip install -e ".[dev]"

# Run any server standalone
python claude-drift/server.py
python claude-memory-mesh/server.py
python claude-proof/server.py

# Register as MCP servers in Claude Code (run from repo root)
claude mcp add claude-drift -- python "$(pwd)/claude-drift/server.py"
claude mcp add claude-memory-mesh -- python "$(pwd)/claude-memory-mesh/server.py"
claude mcp add claude-proof -- python "$(pwd)/claude-proof/server.py"
```

```bash
# Tests and linting
python -m pytest                                              # all 149 tests
python -m pytest tests/test_drift.py                          # one file
python -m pytest tests/test_drift.py::TestImportBoundary -v   # one class
ruff check .
```

```bash
# CLI scanner (no MCP needed)
python scripts/scan_repo.py /path/to/repo                     # local repo
python scripts/scan_repo.py https://github.com/org/repo       # clone + scan
python scripts/scan_repo.py . --scope src/auth                # scope filter
python scripts/scan_repo.py . --json                          # JSON output
```

Python >= 3.10 required. Single runtime dependency: `fastmcp>=2.0.0`.
On Windows with multiple Python versions, use `py -3.12 -m venv .venv` (3.14 has compilation issues with C extensions).

### Environment variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `MEMORY_MESH_DB` | `~/.claude/memory-mesh.db` | SQLite database path for claude-memory-mesh |


## Conventions

- All timestamps are UTC ISO 8601 via `datetime.now(timezone.utc).isoformat()`
- UUIDs for all entity IDs (claims, intents, violations, proofs)
- Enums are `str, Enum` for JSON-friendly `.value` access
- Dataclasses with `to_dict()` -- not Pydantic
- `examples/CLAUDE.md` shows `<!-- drift:intent -->` block syntax for marking constraints
- `examples/.drift-rules.json` shows the structured rule format
- Drift server skips: `node_modules`, `.git`, `__pycache__`, `dist`, `build`, `.next`, `venv`, `.venv`, `vendor`
- Drift server scans: `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`
- Ruff config ignores E401/E701/E702/E722 to match the codebase's compact style
- FastMCP v3: use `instructions=` not `description=` in `FastMCP()` constructor

### Testing gotchas

- **Hyphenated dirs**: `conftest.py` uses `importlib.util` to load `claude-drift/server.py` as `claude_drift_server` (and likewise for the other two). Import from these names in tests.
- **DB isolation**: Patch `mms.DB_PATH` via `patch.object(mms, "DB_PATH", memory_db)`, not `os.environ`. The `_db()` default parameter was captured at import time before `patch.dict` could reach it; the function now reads the module-level `DB_PATH` at call time.
- **Drift state**: `_intents` is a module-level dict. Call `_intents.clear()` between tests that use `scan_intents` or `declare_intent` to avoid cross-test leakage.
- **Path assertions in tests**: Use `"x" in path.parts` (exact component match), not `"x" in str(path)`. Pytest temp dirs include test names (e.g., `test_skips_node_modules0`), causing substring false positives.
- **scan_repo.py tests**: Must `sys.path.insert(0, ROOT / "scripts")` before importing, then `# noqa: E402` on the import line.
