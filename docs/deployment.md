---
type: canonical
owner: platform-engineering
last-reviewed: 2026-03-31
---

# Deployment and Release -- provegate

Provegate servers run as local processes; there is no hosted or
cloud-deployed artifact. Deployment means installing the package and
registering each server with an MCP host (Claude Code, Cursor, or
compatible client).

## Deployment Process

Install from source:

```bash
pip install fastmcp
pip install -e .
```

Register each server with Claude Code (adjust path to where you cloned):

```bash
claude mcp add claude-drift -- python /path/to/claude-drift/server.py
claude mcp add claude-memory-mesh -- python /path/to/claude-memory-mesh/server.py
claude mcp add claude-proof -- python /path/to/claude-proof/server.py
```

After registration, the MCP host starts each server on demand and tools
become available in agent sessions. No port configuration is required; MCP
uses stdio transport by default.

To verify registration:

```bash
claude mcp list
```

## Release Strategy

Releases follow semantic versioning. The current version is tracked in
`pyproject.toml` (`version = "0.1.1"` at time of writing). There is no
separate build artifact; pip install from the repo tag or a local editable
install covers all usage.

To cut a release:

1. Update `version` in `pyproject.toml`.
2. Update `CHANGELOG.md`.
3. Tag the commit (`git tag v<version>`).
4. Push the tag.

There is no PyPI publish configured as of this writing. Distribution is
via direct repo install.

## Rollback Procedures

Rollback to a prior version by checking out the relevant tag and
reinstalling:

```bash
git checkout v<prior-version>
pip install -e .
```

claude-memory-mesh and claude-proof persist state in SQLite files. Those
files are not altered by a rollback. If a schema change between versions is
incompatible, delete the SQLite file to start from a clean state (all
stored claims are lost).

## Environment Configuration

No required environment variables for local use. The servers read intent
sources (CLAUDE.md, ADRs, `.drift-rules.json`) from the working directory
of the calling agent session.

For the optional Notion sync (ops runbook feature), set:

| Variable | Purpose |
|----------|---------|
| `NOTION_TOKEN` | Notion integration secret |
| `NOTION_DB_ID` | Target database ID |

See `docs/operations/RUNBOOK.md` for detail on ops sync configuration.
