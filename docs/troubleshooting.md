---
type: canonical
owner: platform-engineering
last-reviewed: 2026-03-31
---

# Troubleshooting -- provegate

## Common Issues

**Server not found after `claude mcp add`**

Verify the path passed to `claude mcp add` is the absolute path to the
correct `server.py`. Run `claude mcp list` to confirm registration. If
the path contains spaces, quote it.

**`ModuleNotFoundError: fastmcp`**

fastmcp is not installed in the Python interpreter the MCP host is
invoking. Install it in the same environment:

```bash
pip install fastmcp
```

If you are using a virtual environment, ensure the server is launched with
that environment's Python binary (use an absolute path to `python` inside
the venv).

**`ModuleNotFoundError: shared`**

The `shared` package must be on the Python path. Installing the repo in
editable mode resolves this:

```bash
pip install -e .
```

**claude-drift returns no violations despite expected rule match**

Check that the intent source exists and is readable. claude-drift reads
from `CLAUDE.md` in the working directory of the session, then falls back
to `.drift-rules.json`. If neither exists, no intents are loaded. Run
`scan_intents` first to confirm what was parsed.

**Memory mesh returns stale or missing claims**

Decay runs on demand via `run_decay`. Call it manually if claims you
expect to expire are still present. SQLite files persist across server
restarts; if the database is in an unexpected state, delete it and restart
the server to begin clean (all claims are lost).

**proof chain rollback targets wrong checkpoint**

`rollback` targets the most recent checkpoint in the active proof chain.
Call `list_active_proofs` to inspect chain state before rolling back.

## Diagnostic Steps

1. Run `claude mcp list` to confirm server registration.
2. Start the server directly (`python claude-drift/server.py`) and check
   for import errors before re-registering with the MCP host.
3. Run `python scripts/scan_repo.py . --json` to test drift scanning
   without the MCP layer.
4. Run `pytest` to confirm the test suite passes in the local environment.
5. Run `ruff check .` to rule out syntax issues.

## Known Failure Modes

- **SQLite concurrent writes.** Running multiple memory-mesh instances
  against the same database file can corrupt state. Use one server instance
  per agent session.
- **Large repos and scan time.** `scan_repo.py` scans all files; on repos
  with many files this can be slow. Use `--scope` to limit to a
  subdirectory.
- **Intent extraction false positives.** Auto-extraction of constraints
  from natural language can match sentences that are not intended as rules.
  Use explicit `<!-- drift:intent -->` blocks or `.drift-rules.json` for
  precise control.

## FAQ

**Do I need all three servers?**

No. Each server is independent. You can use claude-drift alone for drift
checking, or claude-memory-mesh alone for persistent memory, without the
others.

**Where is the SQLite file stored?**

Each server creates its database in its own directory by default. Check the
server source for the exact path; it is configurable via an environment
variable if the server supports it.

**Is provegate suitable for production or multi-user environments?**

Not currently. The servers have no authentication and are designed for
single-developer local use. See the Constraints section of
`docs/architecture.md`.
