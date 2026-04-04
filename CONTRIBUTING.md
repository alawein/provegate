---
type: canonical
source: _devkit/templates
sync: propagated
sla: none
---

# Contributing to provegate

Three MCP servers for AI agent drift detection, memory, and proof chains.

This project follows the [alawein org contributing standards](https://github.com/alawein/alawein/blob/main/CONTRIBUTING.md).

## Getting Started

```bash
git clone https://github.com/alawein/provegate.git
cd provegate
pip install -e ".[dev]"
```

## Development Workflow

1. Branch off `main` using prefix: `feat/`, `fix/`, `docs/`, `chore/`, `test/`
2. Make your changes — keep PRs focused on a single concern
3. Run `pytest` to validate your changes before committing
4. Commit using [Conventional Commits](https://www.conventionalcommits.org/) — `type(scope): subject`
5. Open a Pull Request to `main`

## Code Standards

- Python 3.10+, single runtime dependency: `fastmcp>=2.0.0`
- Dataclasses with `to_dict()` -- not Pydantic
- All timestamps UTC ISO 8601
- UUIDs for all entity IDs

## Pull Request Checklist

- [ ] CI passes (no failing checks)
- [ ] Tests added or updated for new functionality
- [ ] `ruff check . && pytest` passes
- [ ] `CHANGELOG.md` updated under `[Unreleased]`
- [ ] No breaking changes without a version bump plan

## Reporting Issues

Open an issue on the [GitHub repository](https://github.com/alawein/provegate/issues) with steps to reproduce and relevant context.

## License

By contributing, you agree that your contributions will be licensed under [MIT](LICENSE).
