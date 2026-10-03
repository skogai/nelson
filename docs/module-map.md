# Module Map

This maps Nelson's actual runtime behavior — the phase state machine, which
files get read/written at each step, and which Python modules are load-bearing
versus optional — straight from the code (`nelson-phase.py`,
`nelson_data_lifecycle.py`, the CLI dispatch table in `nelson-data.py`), not
from prose documentation. Written as a spike check for build-vs-rewrite before
investing further in this fork; see `project_structure.md` for the plain
file-tree listing this complements.

## Phase state machine

`SAILING_ORDERS → ESTIMATE → BATTLE_PLAN → FORMATION → PERMISSION → UNDERWAY → STAND_DOWN`

`nelson-phase.py advance` is the only way forward, and it refuses to advance
until the exit check for the *current* phase passes:

| Phase | Exit criteria (code, not docs) | File(s) it reads | Tools blocked while in this phase |
|---|---|---|---|
| **SAILING_ORDERS** | `sailing-orders.json` exists with an `outcome` | `sailing-orders.json` | Agent, TeamCreate, TaskCreate |
| **ESTIMATE** | `estimate.md` exists, OR `sailing-orders.json.estimate_skipped == true` | `estimate.md`, `sailing-orders.json` | TeamCreate, TaskCreate |
| **BATTLE_PLAN** | `battle-plan.json` has tasks, every task has a `station_tier` | `battle-plan.json` | Agent, TeamCreate, TaskCreate |
| **FORMATION** | `battle-plan.json.squadron` exists with an `admiral` | `battle-plan.json` | Agent, TeamCreate |
| **PERMISSION** | a `permission_granted` event exists | `mission-log.json` | Agent, TeamCreate, TaskCreate |
| **UNDERWAY** | every task has a `task_completed` event, or a `mission_complete` event | `battle-plan.json`, `mission-log.json` | *(none)* |
| **STAND_DOWN** | terminal — can't advance further | — | TeamCreate, TaskCreate |

Full mission-dir artifact set: `sailing-orders.json`, `estimate.md`,
`estimate-outcomes.json`, `battle-plan.json`/`.md`, `mission-log.json`,
`fleet-status.json`, `idle-tracker.json`, `stand-down.json`, plus
`damage-reports/` and `turnover-briefs/` subdirectories. Two files
(`mission-log.json`, `estimate-outcomes.json`) have `.lock` siblings for
concurrent-agent writes.

## CLI commands mapped to the phase they belong to

`nelson-data.py` is one dispatcher over 19 subcommands, implemented across
several modules. Grouped by where they fire in the lifecycle:

| Lifecycle point | Command(s) | Writes to |
|---|---|---|
| Start | `init` | creates mission dir, `sailing-orders.json`, `mission-log.json`, `fleet-status.json`, `.active-{id}` marker |
| ESTIMATE | `skip-estimate`, `record-estimate-outcome` | `sailing-orders.json`, `estimate-outcomes.json` |
| BATTLE_PLAN | `task` (×N), `plan-approved` | `battle-plan.json` |
| FORMATION | `squadron` | `battle-plan.json.squadron` |
| PERMISSION | `admiralty-decision` (implicit `permission_granted` event) | `mission-log.json` |
| UNDERWAY | `event`, `checkpoint`, `handoff` | `mission-log.json`, `fleet-status.json`, turnover-briefs |
| STAND_DOWN | `stand-down` | `stand-down.json` |
| Cross-cutting | `status`, `recover`, `form` (task+squadron+plan composite), `headless` (init+form, no gate) | reads everything / composite writes |
| Not phase-bound | `goal-condition` (`nelson_data_goal.py`) | composes a Claude `/goal` string from sailing orders |

That's the entire **core tier** — everything `SKILL.md`'s walkthrough actually
drives.

## Module tiers

### Core (load-bearing, exercised every mission)

- **`nelson_data_lifecycle.py`** (2,205 LOC) — all the `cmd_*` handlers above:
  init, squadron, task, checkpoint, stand-down, handoff, recover, status.
  This is the one file that's outgrown itself (41 functions, several with
  `noqa: C901` complexity suppressions tracked against issue `nelson-e6j`).
  If anything gets split, split this first.
- **`nelson-phase.py`** (592 LOC) — the state machine table above, standalone,
  no imports from the rest of the project. Clean candidate to lift out on its
  own if the project is ever trimmed.
- **`nelson_data_utils.py`** (385 LOC) — shared I/O/locking, no business logic.
- **`hooks/nelson_hooks.py`** (793 LOC) — SessionStart / PreToolUse /
  PostToolUse / TaskCompleted / TeammateIdle enforcement, imports
  `nelson_circuit_breakers`.
- **`nelson_conflict_scan.py`** — preflight static check (import-graph based
  "split-keel" detection), called explicitly in `SKILL.md` before
  FORMATION → PERMISSION.

### Support library (imported by core, not directly invoked)

- **`nelson_data_memory.py`** (435 LOC) — cross-mission pattern store,
  imported by lifecycle, calibration, fleet, and patterns modules.
- **`nelson_circuit_breakers.py`** (460 LOC) — hull/budget/idle alarm
  thresholds, imported by hooks and lifecycle.

### Optional/analytics tier (reachable via CLI, not part of the mission walkthrough)

- **`nelson_data_fleet.py`** (855 LOC) — `index` / `history` / `brief` /
  `analytics` subcommands, cross-mission reporting.
- **`nelson_data_patterns.py`** (1,242 LOC) — `detect-patterns` /
  `promote-candidate` / `dismiss-candidate`, the standing-order mining
  pipeline mentioned once in `SKILL.md` as a "see references" aside, not a
  required step.
- **`nelson_data_calibration.py`** (596 LOC) — `trust-report`. **Confirmed
  unreferenced anywhere in `SKILL.md` or any reference doc** — only reachable
  by typing the subcommand directly. Maps to still-open upstream issue #88
  (confidence-weighted trust calibration), never finished/integrated before
  upstream went quiet.

### Opt-in, disabled by default

- **`nelson_conflict_radar.py`** (161 LOC) — a *different* check from
  `nelson_conflict_scan.py`: live `git status` vs. ownership during UNDERWAY,
  not a preflight. Its own docstring says it's opt-in and must be manually
  added to `settings.json` hooks — not a duplicate of conflict-scan, just a
  runtime watcher nobody wired up by default.

## Context: why this map exists

Upstream (`Aspegio/nelson`) last pushed 2026-07-03 and has been dormant for
three months as of this writing (2026-10-03), with 2 stale open issues and no
open PRs — this fork is the active branch going forward. The project was
designed as a skill (SKILL.md entrypoint + on-demand reference docs) from its
first commit, not a monolith that grew sprawl over time, so the module count
itself isn't a decay signal. The actual finding worth acting on: roughly half
the Python (fleet intelligence, pattern mining, trust calibration) sits
outside the core mission walkthrough and should be treated as experimental
rather than load-bearing until there's a concrete reason to depend on it.
