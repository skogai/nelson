# Nelson

Nelson is a Claude Code skill for coordinating agent work using Royal Navy terminology. It provides an eight-step operational framework: Sailing Orders, The Estimate, Battle Plan, Form the Squadron, Get Permission to Sail, Quarterdeck Rhythm, Action Stations, and Stand Down.

## Key references

- **[docs/project_structure.md](./docs/project_structure.md)** — full repository layout
- **[README.md](./README.md)** — user-facing overview and quick start

## Maintainability sensors

See `CLAUDE.md` for the full sensor reference. Quick checklist for agent
runs:

- `ruff check` — lint with AI-targeted thresholds (config in `pyproject.toml`).
- `ruff format --check` — formatting compliance.
- Tests, one directory at a time (each has its own `conftest.py`):
  `pytest skills/nelson/scripts/ -v && pytest hooks/ -v && pytest scripts/ -v`.
- `pre-commit run --all-files` — Gitleaks + Ruff + hygiene (install with
  `pre-commit install`).

Suppression convention: `# noqa: <rule> -- <reason>`. Naked noqas are not
acceptable. See `CLAUDE.md` → *Maintainability sensors* for the rationale.
