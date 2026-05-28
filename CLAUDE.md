# Nelson

Nelson is a Claude Code skill for coordinating agent work using Royal Navy terminology. It provides an eight-step operational framework: Sailing Orders, The Estimate, Battle Plan, Form the Squadron, Get Permission to Sail, Quarterdeck Rhythm, Action Stations, and Stand Down.

## Key references

- **[docs/project_structure.md](./docs/project_structure.md)** — full repository layout
- **[README.md](./README.md)** — user-facing overview and quick start

## Maintainability sensors

This repo has automated sensors that you (the agent) should run and read
carefully before considering a task done. They're how the codebase gives you
fast feedback about whether your changes are maintainable.

### Active sensors

- **Linter** — `ruff check`. Config in `pyproject.toml`. AI-targeted
  thresholds (`max-args=5`, `max-branches=10`, `max-statements=50`,
  `max-complexity=10`, `line-length=120`). Each violation message is
  feedback — read it, don't just look at the line. Many rules accept a
  suppress-with-reason or threshold-bump instead of forcing a fix.
- **Formatter** — `ruff format --check`. Apply with `ruff format` before
  committing.
- **Tests** — run separately per directory because each has its own
  `conftest.py`:
  ```
  pytest skills/nelson/scripts/ -v
  pytest hooks/ -v
  pytest scripts/ -v
  ```
- **Pre-commit** — `pre-commit run --all-files`. Includes secret scanning
  (Gitleaks), Ruff, and standard hygiene. Install once with
  `pre-commit install`. If a hook fails, fix the underlying issue rather
  than passing `--no-verify`.
- **CI** — the same checks re-run on clean infra after push (see
  `.github/workflows/ci.yml`). Green locally + red in CI usually means
  environment drift (Python version, env vars, OS-specific paths).

### Suppressing or bumping a sensor

You may suppress a rule or bump a threshold when the rule is clearly wrong
for the situation. **The reason is mandatory.**

```python
result = subprocess.run(  # noqa: S603 -- args are repo-internal paths, sys.executable is trusted
    [sys.executable, str(_SCRIPT), ...],
    ...,
)
```

Naked suppressions (no `--` reason) are not acceptable — they're noise and
they hide the next regression. If you find existing naked suppressions,
treat them as a mini code-review task: add a reason or remove the
suppression.

If a threshold (`max-complexity`, `max-args`, `line-length`, etc.) needs to
go up project-wide, change it in `pyproject.toml` and add a comment
explaining the trade-off. Don't disable the rule entirely — leaving the
rule active means it will catch the *next* drift.

**Why this matters:** every suppress-with-reason line and every
threshold-bump comment is itself a *review anchor*. With a sensor-aware
workflow, those lines are the durable record of "we considered this, and
here's why we accepted it." Write the reasons accordingly: aim for *the
next reader can decide whether this is still a good idea*, not *I want the
lint to stop yelling*.

### Brownfield complexity backlog

The largest functions carry `# noqa: C901, PLR0912, PLR0915` suppressions
pointing to **beads issue `nelson-e6j`**. When you touch one of those
functions, remove the noqa and verify the limit is met; refactor if not.
Don't bulk-refactor (the over-engineering risk is real) — fix
opportunistically.

### When sensors disagree with you

Sensors are heuristics, not laws. If you genuinely believe a rule is wrong
for a particular file or function, propose the threshold change or
per-directory override explicitly — don't reach for a suppression as a
shortcut. Discussion in a comment / PR description beats a silent disable.
