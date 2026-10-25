# Battle Plan — Fleet research → DASHBOARD.md

Commander's intent:
Right now, knowing where a Nelson mission stands means already knowing Nelson cold. The README's Quick Start tells a reader what to type, not what lands on disk, in what order, or what any of it means once it's there — and that gap isn't theoretical: a mission from July sits abandoned at `BATTLE_PLAN` with its session marker still live on disk, and nothing short of a maintainer manually walking `.nelson/missions/` would ever have surfaced it. The fix is not another onboarding doc — upstream already covers general usage, and restating it would only drift out of sync — it's a short, fork-specific explainer of the mechanical trace a mission actually leaves behind: which artifacts appear at which phase, what each one means, and how to read them cold, proven against this very mission's own files rather than described in the abstract. The improvement backlog that rides alongside it matters for the same reason the doc does: documenting that trace by hand is itself evidence the real fix is a tool that reads it for you, so that proposal needs to be scoped and argued now, ready to pick up as its own mission, even though nobody builds it this session.

Workflow suitability: not selected because this is a single-file, single-owner documentation task with no fan-out, no repeatable orchestration need, and no cross-checking requirement — a workflow run would add machinery with nothing to parallelize.

## Task 1
- Name: Write DASHBOARD.md (state-model explainer + worked-example walkthrough)
- Owner: assigned at Step 4 — Form the Squadron
- Ship (if crewed): assigned at Step 4 — Form the Squadron
- Crew manifest: none — captain implements directly (atomic task, no sub-task decomposition warrants crew)
- Deliverable: `DASHBOARD.md` at repo root, covering Estimate §3 Effects 1 and 2 (state-model explainer organized by mission phase; mechanical worked-example walkthrough using this mission's own files plus the stale mission as the cautionary half)
- Dependencies: none (Estimate approved; no predecessor task)
- Station tier: 0 — Patrol (documentation-only, no runtime/security/data-integrity risk, not user-visible/coupled work; see Estimate §7)
- File ownership: `DASHBOARD.md` (create). Read-only access to `.nelson/missions/2026-10-02_181703_3d83347e/`, `.nelson/missions/2026-07-06_223339_761d8f37/`, `skills/nelson/scripts/nelson-phase.py`, `README.md`, `CONTRIBUTING.md`, `AGENTS.md`, `CLAUDE.md`, `docs/project_structure.md`, `skills/nelson/references/the-estimate.md`, `skills/nelson/references/action-stations.md`.
- Modification targets: none — greenfield file, no existing code or docs edited.
- Acceptance criteria (inherited from Estimate §3, Effects 1 and 2):
  - Every artifact filename and path named in the state-model section exists somewhere under this mission's own directory, or is explicitly flagged as "produced at a later phase, not yet written" — verification: direct file check against `.nelson/missions/2026-10-02_181703_3d83347e/`.
  - The phase list matches `nelson-phase.py`'s actual state machine verbatim — verification: diff against the script.
  - No paragraph restates content already covered in README, CONTRIBUTING, AGENTS.md, CLAUDE.md, or `docs/project_structure.md`; those are linked, not duplicated — verification: review, spot-check for overlap.
  - Every quoted or described field in the worked-example section traces to a real file this mission produced, re-read at the time the task completes — verification: re-read cited files and confirm match.
  - The stale-mission passage names the path and phase and stops there — no remediation steps, no edits proposed to that mission's files — verification: review against out-of-scope constraint.
  - A reader unfamiliar with Nelson can follow the walkthrough and correctly state, for a hypothetical mission directory, roughly what phase it's in from the files present — verification: read-back check by the admiral.
- Validation required: admiral reads `DASHBOARD.md` against the real `.nelson/` files before Stand Down (ordinary quarterdeck check, no red-cell review — Station 0).
- Rollback note required: no (trivial — unreviewed or unwanted file can be edited or deleted; nothing downstream references it yet).
- admiralty-action-required: no

## Standing Order Check

- **becalmed-fleet.md** — Single-session instead of multi-agent? Considered and rejected in favor of `subagents` mode with exactly one captain: true single-session (per `tool-mapping.md`) means the admiral executes directly, which collides with `admiral-at-the-helm` below. One captain under `subagents` mode gets the same zero-coordination-overhead shape (no team, no shared task list, no peer messaging) while still delegating the actual writing. No squadron is formed — there is exactly one task and no independent branch to parallelize.
- **light-squadron.md** — Task count equal to independent work units? Yes: there is exactly one independent work unit (Effects 1+2 share a file and are sequential by nature; Effect 3 is the admiral's own permitted captain's-log work, not a squadron task), and it is assigned to exactly one task. No under-splitting.
- **split-keel.md** — Exclusive file ownership, no conflicts? Yes, trivially — one task, one file (`DASHBOARD.md`), one owner. Verified by the conflict scan in Step 4.
- **unclassified-engagement.md** — Does every task have a risk tier? Yes — Task 1 is Station 0 (Patrol), reasoned from `action-stations.md`'s decision tree in Estimate §7.
- **all-hands-on-deck.md** — Crewed only with roles the work demands? Yes — zero crew. The task is atomic (one ship reading real files and writing prose); no role (XO, PWO, NO, etc.) has distinct sub-work to own.
- **skeleton-crew.md** — Would any task deploy exactly one crew member for atomic work the captain should handle directly? N/A — zero crew deployed, captain implements directly, which is the correct shape for this atomic task.
- **crew-without-canvas.md** — Every agent justified by actual task scope? Yes — exactly one agent (the captain) is spawned, justified by `admiral-at-the-helm` (the admiral cannot write the file itself). No additional agents are added.
- **captain-at-the-capstan.md** — For tasks with crew, is the captain's role coordination not implementation? N/A — no crew mustered; the captain implements directly, which is correct for a 0-crew task (the restriction is about captains with crew sidestepping coordination, not about solo captains doing the one task that exists).
- **press-ganged-navigator.md** — Red-cell navigator assigned implementation work? N/A — no red-cell navigator exists for this mission (Station 0, no adversarial-review-shaped criteria per Estimate §5).
- **admiral-at-the-helm.md** — Does the battle plan assign implementation work to the admiral? No — Task 1 (writing `DASHBOARD.md`) is assigned to a captain. The admiral's only direct work is the captain's log at Stand Down (Effect 3's ranked backlog), which is explicitly permitted coordination-artifact work, not implementation.
- **wrong-ensign.md** — Do planned coordination tools match the selected mode? Yes — `subagents` mode: the captain is spawned via `Agent` tool, reports synchronously via its return value, uses no `TaskCreate`/`TaskList`/`SendMessage`. The admiral uses `TaskCreate`/`TaskUpdate` only for session-visibility tracking.
- **pulling-the-oar.md** — For dispatched-subagent tasks, is the failure-recovery plan to fix the brief and re-dispatch rather than absorb the work? Yes — if the captain's output falls short of an acceptance criterion, the remedy is to amend the captain's brief and re-dispatch, not for the admiral to rewrite `DASHBOARD.md` directly (that would itself violate `admiral-at-the-helm`).

Workflow suitability: not selected (see line above commander's intent) — no fan-out, no repeatable orchestration, single file.
