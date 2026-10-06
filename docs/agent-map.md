# Agent Map

The module map (`module-map.md`) describes the code. This describes the agents
that use it. Nelson is the admiral agent. The seven crew roles are peer agents.
Each has the same shape: a persona, a boundary, a home directory, a structured
input and output, and the modules it connects to.

## Agent card format

Every agent is defined by an `agent.md` file in its home directory, with the
same fields:

| Field | Meaning |
|---|---|
| `officer`, `epithet` | Persona. Name and character, from the persona table. |
| `thinks_like` | The heuristic the agent applies. Guides judgement, not tools. |
| `function` | One line: what the agent is for. |
| `boundary` | What it may read, write, and run. Enforced by subagent type until a per-role envelope exists. |
| `home` | Project subfolder the agent owns: `.nelson/agents/<role>/`. |
| `input` | Structured contract. What the agent receives. |
| `output` | Structured contract. What it must return. |
| `state` | What persists in its home across missions. |
| `events` | Event types it emits into `mission-log.json`. |
| `modules` | Code it connects to. See `module-map.md`. |

Nelson's own card is the same shape. Its home is `.nelson/missions/<id>/`, which
it alone writes. Crew homes hold only what that role keeps across missions.

## Contract conventions

- Input and output are JSON, validated at the boundary. Free prose goes in a
  `notes` field, never in place of a structured field.
- Every output carries `role`, `task_id`, `mission_id`, and `produced_at`.
- Every output carries `confidence` (`high` | `medium` | `low`) and `evidence`
  (a list of file paths, URLs, or command outputs). A finding without evidence
  is recorded as a hypothesis.
- A role never writes outside its home except to return output to Nelson.
  Nelson records it in the mission log.

## Roles

### XO: Ashdown

> The ship runs because she says it does. Nothing crosses the Captain's desk
> without passing through me first.

- **function:** integration across crew outputs, and the gate between crew and
  the mission record.
- **boundary:** read all crew outputs. Writes only its own integration notes
  and the decision to accept or return each output.
- **home:** `.nelson/agents/xo/`
- **input:** `{ "task_id", "outputs": [<role output>...] }`
- **output:** `{ "task_id", "decision": "accept" | "return" | "escalate", "reasons": [...], "integrated_result": {...} }`
- **state:** `integration-log.json`, a record of past decisions and the reasons
  for each return.
- **events:** `task_integrated`, `task_returned`
- **modules:** `nelson-phase.py` (`validate-tool`) is the gate. `nelson-data.py`
  `form` and `handoff` are its integration points.

### PWO: Okorie

> He sees the engagement first. Ask questions rather than issue corrections,
> and make the right answer feel inevitable.

- **function:** the default doer. Implements one deliverable.
- **boundary:** write access to files owned by its task in the battle plan.
  No test runs, which belong to MEO.
- **home:** `.nelson/agents/pwo/`
- **input:** `{ "task_id", "name", "deliverable", "deps", "station_tier", "owned_files", "acceptance" }`, taken from the `task` record in `battle-plan.json`.
- **output:** `{ "task_id", "files_changed": [...], "summary", "open_questions": [...] }`
- **state:** `patterns-used.json`, approaches it has used and how they went.
- **events:** `task_started`, `task_completed`, `question_raised`
- **modules:** `nelson_data_lifecycle.py` (`task`, `event`), `nelson_conflict_scan.py`
  (file ownership comes from `owned_files`).

### NO: Marlowe

> She knows where the rocks are. Annotate in pencil, never pen. Everything is
> contingent, revisable.

- **function:** research and exploration. Read-only. Two mission types use this
  role with different contracts (see below).
- **boundary:** read access and search. No writes outside its home.
- **home:** `.nelson/agents/no/`
- **input:** `{ "task_id", "question", "scope", "mission_type": "environment" | "web" }`
- **output:** `{ "task_id", "findings": [{ "claim", "confidence", "evidence": [...], "observed_at" }], "unknowns": [...] }`
  - `mission_type: "environment"`: evidence is command output or file paths.
    `observed_at` is the time of the command. Facts are snapshotted with a hash.
  - `mission_type: "web"`: evidence is URLs. Each source carries `fetched_at`.
    Sources without a fetch date are rejected.
- **state:** `sources.json` for web missions. A ledger of URLs, fetch dates, and
  the claims each one supports. Entries older than a threshold are flagged stale.
- **events:** `findings_recorded`
- **modules:** none yet. This is a gap. The closest existing pieces are
  `nelson_data_memory.py`, which stores patterns but not sources, and
  `nelson_data_fleet.py`, which has no per-claim evidence.

### MEO: Fen

> Fragile is just broken that hasn't happened yet. Rebuild it properly.

- **function:** testing and validation. Owns pass/fail for each task.
- **boundary:** runs tests and reads results. Writes only to its home and
  damage reports.
- **home:** `.nelson/agents/meo/`
- **input:** `{ "task_id", "acceptance", "files_changed" }`
- **output:** `{ "task_id", "result": "pass" | "fail", "failures": [...], "rerun_command": "..." }`
- **state:** `test-history.json`, which tests were flaky and under what conditions.
- **events:** `validation_passed`, `validation_failed`
- **modules:** `damage-reports/` (written on failure), `nelson_circuit_breakers.py`
  (repeated failures trip the hull threshold).

### WEO: Beynon

> She maintains what others merely use. If you can't describe the fault
> specifically, you haven't understood it.

- **function:** config, infrastructure, and the enforcement layer itself.
- **boundary:** write access to config and hook scripts. Changes to the
  enforcement layer require Nelson's approval.
- **home:** `.nelson/agents/weo/`
- **input:** `{ "task_id", "change_spec", "affected_systems": [...] }`
- **output:** `{ "task_id", "changes": [...], "fault_description": "...", "verified_by": "..." }`. `fault_description` must name the specific fault, not "misconfigured".
- **state:** `fault-log.json`, specific faults and their fixes.
- **events:** `config_changed`, `fault_recorded`
- **modules:** `hooks/nelson_hooks.py` and `hooks/hooks.json`. Partial: the
  hooks are the enforcement layer, but WEO has no code-level ownership of them yet.

### LOGO: Haidari

> If it's aboard, Haidari put it there. Keep snapshots of known-good states.
> The larder saves ships.

- **function:** documentation, dependencies, and known-good snapshots.
- **boundary:** write access to docs and to its snapshot store.
- **home:** `.nelson/agents/logo/`
- **input:** `{ "task_id", "snapshot_of": "..." } | { "task_id", "docs_scope" }`
- **output:** `{ "task_id", "snapshot": { "id", "hash", "paths": [...] } }` or `{ "task_id", "docs_changed": [...] }`
- **state:** `snapshots/`, known-good states the other roles can restore from.
- **events:** `snapshot_taken`, `snapshot_restored`
- **modules:** none yet. This is a gap. There is no snapshot primitive in the
  codebase. The existing `partial-rollback.md` and `relief-on-station.md`
  procedures describe the need, but nothing stores a known-good state.

### COX: Reardon

> Walk the ship. Stretch your legs. See what others don't report.

- **function:** standards review and cross-ship observation. Read-only.
- **boundary:** read access to files, git state, and the battle plan. No writes outside its home.
- **home:** `.nelson/agents/cox/`
- **input:** `{ "task_id", "scope", "conventions_source" }`
- **output:** `{ "task_id", "observations": [{ "file", "line", "issue", "severity" }], "unreported": [...] }`. `unreported` is for things seen but not in any role's scope.
- **state:** `walk-notes.json`, recurring observations across missions.
- **events:** `observation_raised`
- **modules:** `nelson_conflict_radar.py` (opt-in, live `git status` check),
  `nelson_conflict_scan.py` (preflight), and the `idle-ship` hook (`idle-tracker.json`).

## Coverage

| Role | Module connected | Status |
|---|---|---|
| XO | `nelson-phase.py`, `nelson-data.py` form/handoff | connected |
| PWO | lifecycle `task`/`event`, conflict scan | connected |
| NO | none | **gap**: no source ledger, no per-claim evidence |
| MEO | damage reports, circuit breakers | connected |
| WEO | hooks | partial: no code-level ownership |
| LOGO | none | **gap**: no snapshot primitive |
| COX | conflict radar/scan, idle hook | connected |

Two roles (NO and LOGO) have no code behind them. Those are the first places
new modules would need to go, and the two mission types for NO are what
determine the shape of its contract.

## Open decisions

- Whether `boundary` becomes enforced per role (a tool envelope per agent card)
  or stays prose plus subagent type.
- Whether crew homes persist across missions (`.nelson/agents/<role>/`) or are
  reset per mission. This doc assumes persistent homes.
- Whether the NO `web` and `environment` contracts should be two roles
  (`NO-web`, `NO-env`) or one role with a `mission_type` switch.
