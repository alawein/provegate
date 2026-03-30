---
type: canonical
source: none
sync: none
sla: none
---

<!-- Template: python-ml-server v1.0.0 -->
<!-- Generated from workspace governance; project-specific sections are authoritative. -->
---
type: normative
authority: canonical
audience: [agents, contributors, maintainers]
last-verified: 2026-03-30
---

# AGENTS — epistemic-stack

> **Status: Normative.** Do not modify this file without maintainer review.

This repository contains three related MCP servers and shared prompting research assets.

## Repository Scope

| Directory | Purpose | Governance level |
|-----------|---------|------------------|
| `claude-drift/` | Drift detection engine and rule generation | Primary |
| `claude-memory-mesh/` | Persistent graph memory and indexing | Primary |
| `claude-proof/` | Proof/verification workflows and checkpoints | Primary |
| `shared/` | Shared tooling and cross-service contracts | Stable |
| `examples/` | Example integrations and docs snippets | Secondary |
| `tests/` | Unit and integration tests | Primary |
| `scripts/` | CLI entrypoints and orchestration helpers | Stable |

## Invariants (Must Always Hold)

1. Changes to core behavior must include or update tests in `tests/`.
2. All scripts and MCP entrypoints must be executable and importable from repo root.
3. No secrets or environment credentials checked into source.
4. Keep dependency/runtime assumptions explicit in CLI entrypoints.
5. Python versions must remain compatible with `>=3.10`.
6. New protocol or schema changes must stay backward-compatible where practical.

## Agent Rules

- Read `AGENTS.md` and `CLAUDE.md` before editing.
- Do not modify `.venv/` or local cache directories unless requested.
- Run `pytest` before proposing behavior-changing edits.
- Run `python -m ruff check .` for touched Python paths.
- Keep imports minimal and avoid broad `*` imports.
- Keep public interfaces documented when behavior changes (README + any touched route docs).
- Any change to prompt/decision logic must preserve deterministic defaults and include regression tests.

## Test Requirements

- Validation baseline: `pytest`
- Optional type check: `mypy scripts` (as available)
- Lint baseline: `ruff check .`

## Repo-Specific Notes

- `README.md` is the canonical onboarding entrypoint; update if CLI flags or package exports change.
- This repository has no `.Codex/AGENTS.md`; this `AGENTS.md` is the authoritative source.
- If adding new dependencies, update `pyproject.toml` and lock/pin constraints consistently.
