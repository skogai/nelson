# Battle Plan - External Repository Missions

Commander's intent:
Nelson must be able to command work in an external target repository while keeping mission command, records, and recovery in the Nelson control repository. The change should make the target checkout a first-class mission concern, not an implied `cwd`, so captains receive an unambiguous working root and hooks can still find the active mission. For this user's default path, Nelson should create or use a worktree from an already-existing local source repository through the user's `wt` provider, preserving lifecycle scripts and refusing silent clone fallback when the source is absent. Existing single-repository Nelson missions must continue to work unchanged.

Workflow suitability: not selected because the mission has four coordinated implementation fronts with shared terminology and review gates, not a broad repeatable audit or migration that benefits from dynamic workflow scripting.

## Task 1
- Name: Add target repository mission data
- Owner: HMS Daring
- Ship: destroyer
- Crew manifest: Captain implements directly unless implementation reveals a testing split; no initial crew.
- Deliverable: Structured mission and battle-plan data can record a target repository, provider, source repository, branch, and actual working directory while preserving older mission files.
- Dependencies: none
- Station tier: 2
- File ownership:
  - `skills/nelson/scripts/nelson-data.py`
  - `skills/nelson/scripts/nelson_data_lifecycle.py`
  - `skills/nelson/scripts/nelson_data_utils.py`
  - `skills/nelson/scripts/nelson_data_fleet.py`
  - `skills/nelson/scripts/test_nelson_data.py`
  - `skills/nelson/scripts/test_nelson_data_fleet.py`
- Modification targets:
  - `init`, `headless`, and `form` argument/input handling
  - mission-level `target_repository` / `target_repositories` schema
  - battle-plan task target-repo references
  - fleet intelligence denormalization where completed missions are indexed
- Acceptance criteria:
  - A mission can record a target repository distinct from the Nelson control repository.
  - Existing missions without target metadata remain valid and retain current local-repo behavior.
  - Default Nelson-created branch names follow `nelson/<mission-id>`.
  - Multi-repo metadata is possible only by explicit mission configuration, not by accidental inference.
  - Provider resolution can select `wt` when available and configured.
  - Missing source repository paths fail clearly without cloning as a fallback.
  - Existing target checkouts can be used without invoking a creation provider.
- Validation required: focused mission-data tests for new fields, old fixtures, headless input, form input, and default branch naming.
- Rollback note required: yes
- admiralty-action-required: no

## Task 2
- Name: Make hooks mission-root aware
- Owner: HMS Dragon
- Ship: destroyer
- Crew manifest:
  - PWO: implement separated-root mission lookup and hook wiring.
  - MEO: add and run hook tests for control-root and target-root cases.
- Marine capacity: 0
- Deliverable: Hook enforcement can locate the active Nelson mission when the process `cwd` is a target worktree, while preserving current single-root behavior.
- Dependencies: Task 1 field names and transport contract
- Station tier: 3
- File ownership:
  - `hooks/nelson_hooks.py`
  - `hooks/hooks.json`
  - `hooks/test_nelson_hooks.py`
- Modification targets:
  - `_find_mission_dir`
  - `_load_mission_context`
  - `_write_admiral_marker`
  - `cmd_preflight`
  - `cmd_session_init`
  - `cmd_session_check`
  - `cmd_task_complete`
  - `cmd_idle_ship`
- Acceptance criteria:
  - Agent preflight, session checks, task enforcement, completion gates, and idle-ship checks can locate the active mission from a target checkout.
  - Mission artifacts continue to be written under the control repository's `.nelson/missions/...`.
  - Hooks still work for current single-root missions without additional configuration.
  - Runtime failures distinguish "no active mission" from "active mission exists in another control root but was not supplied."
- Validation required: hook tests covering split roots, old cwd-local discovery, missing control-root behavior, and TaskCreate enforcement.
- Rollback note required: yes
- admiralty-action-required: yes
  - action: Human confirmation before merging hook behavior that changes fail-open or reject behavior.
  - timing: after this task completes
  - blocks: stand-down

## Task 3
- Name: Propagate target roots into operational briefs
- Owner: HMS Kent
- Ship: frigate
- Crew manifest: Captain implements directly; request COX review if terminology diverges from the data contract.
- Deliverable: Captain-facing templates clearly distinguish Mission Directory, control repository, source repository, and target working root.
- Dependencies: Task 1 field names
- Station tier: 2
- File ownership:
  - `skills/nelson/references/admiralty-templates/sailing-orders.md`
  - `skills/nelson/references/admiralty-templates/estimate.md`
  - `skills/nelson/references/admiralty-templates/battle-plan.md`
  - `skills/nelson/references/admiralty-templates/ship-manifest.md`
  - `skills/nelson/references/admiralty-templates/crew-briefing.md`
  - `skills/nelson/references/admiralty-templates/marine-deployment-brief.md`
  - `skills/nelson/references/admiralty-templates/quarterdeck-report.md`
  - `skills/nelson/references/admiralty-templates/turnover-brief.md`
  - `skills/nelson/references/admiralty-templates/captains-log.md`
- Modification targets:
  - template fields that currently imply one codebase root
  - JSON schema notes in `battle-plan.md`
  - captain and relief briefing working-directory fields
- Acceptance criteria:
  - Captain briefs include both mission directory/control root and target working root.
  - Task modification targets and file ownership are interpreted relative to the correct target repository.
  - Handoff artifacts and turnover reports identify which target repository was changed.
  - Existing brief generation remains readable and unchanged in meaning for single-root missions.
- Validation required: review all updated templates for consistent field names and no contradictory path guidance.
- Rollback note required: yes
- admiralty-action-required: no

## Task 4
- Name: Update doctrine, docs, and integration checks
- Owner: HMS Richmond
- Ship: frigate
- Crew manifest: Captain implements directly.
- Deliverable: User-facing and reference docs describe the separated-root model, `wt` provider default, explicit multi-repo behavior, and relevant verification commands.
- Dependencies: Tasks 1, 2, and 3 for final terminology and behavior
- Station tier: 1
- File ownership:
  - `README.md`
  - `docs/project_structure.md`
  - `skills/nelson/SKILL.md`
  - `skills/nelson/references/structured-data.md`
  - `skills/nelson/references/the-estimate.md`
  - `skills/nelson/references/squadron-composition.md`
  - `skills/nelson/references/standing-orders/split-keel.md`
- Modification targets:
  - usage examples
  - structured data schema examples
  - Q1 reconnaissance guidance
  - mode/ship guidance for one ship per repository
  - file ownership wording relative to target roots
- Acceptance criteria:
  - Documentation explains the separated-root model and the default one-primary-repo mission shape.
  - Documentation describes `wt` as the preferred local worktree provider for this setup.
  - Clone/network behavior is explicitly opt-in and not the default fallback.
  - `wt list`, `wt step diff`, and `wt remove <branch>` are documented as discovery, diff, and cleanup operations where appropriate.
  - Existing test suites relevant to Nelson mission data, hooks, and templates pass after the change.
- Validation required: docs/reference review plus final test command summary from all captains.
- Rollback note required: yes
- admiralty-action-required: no

## Squadron Formation Proposal

Mode: agent-team
Captain count: 4
Red-cell navigator: HMS Astute

Ships:
- HMS Daring - destroyer - target repository mission data and provider contract
  - Crew: Captain implements directly.
- HMS Dragon - destroyer - separated-root hook enforcement
  - Crew: PWO and MEO.
- HMS Kent - frigate - operational brief and template propagation
  - Crew: Captain implements directly.
- HMS Richmond - frigate - user-facing docs and integration checks
  - Crew: Captain implements directly.
- HMS Astute - submarine - red-cell navigator
  - Role: review target-root contract, fail-open behavior, rollback readiness, and final integration.

ADMIRALTY ACTION LIST - Actions required from Admiralty

1. Make hooks mission-root aware
   action: Confirm hook behavior before merge if the change alters fail-open or rejection semantics.
   timing: after task completes
   unblocks: stand-down

## Standing Order Check

- becalmed-fleet: No. The mission has at least three parallelizable work fronts after the data contract is named: hooks, templates, and docs. Single-session would serialize independent work.
- light-squadron: No under-splitting. Four tasks match the independent work units: data, hooks, templates, docs/integration.
- split-keel: No declared file conflicts. Each task owns an exclusive file set; Task 4 may review outputs but must not edit Task 1-3 files.
- unclassified-engagement: Satisfied. Every task has a station tier.
- all-hands-on-deck: Satisfied. Only Task 2 is crewed because hooks need implementation plus validation; other captains implement directly.
- skeleton-crew: Satisfied. No atomic task has a single unnecessary crew member.
- crew-without-canvas: Satisfied. Each captain shortens a different branch of the critical path.
- captain-at-the-capstan: Satisfied by plan. Task 2 captain coordinates PWO and MEO rather than implementing directly while crew are active.
- press-ganged-navigator: Satisfied. Red-cell navigator performs review only.
- admiral-at-the-helm: Satisfied. The admiral coordinates, checkpoints, and writes mission artifacts only; implementation is delegated.
- wrong-ensign: Satisfied. Selected mode is `agent-team`; captains will be spawned as team members, not subagents. Task visibility and peer coordination will use team/task tools.
- pulling-the-oar: Satisfied. Failed subordinate work will be re-briefed and re-dispatched; the admiral will not absorb implementation work.

## Verification Contract

Each captain must report changed files, acceptance-criterion outcomes, verification commands, rollback notes, and any unresolved risk. HMS Astute must review Task 1 and Task 2 before stand-down, with special attention to source-repo existence checks, provider path capture, hook mission lookup, and old single-root compatibility.
