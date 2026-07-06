# The Estimate - External Repository Missions

## 1. Reconnaissance

The mission is to let Nelson command work in a target repository that is not the repository where the Nelson mission is being coordinated. The intended path is explicit: take an already-existing local source repository, create or use a mission worktree for it, send ships to that working directory, and keep Nelson's mission artifacts in the control workspace under `.nelson/missions/...`. That split is not represented cleanly today.

Three Explore agents scouted the terrain.

The mission-data scout found that `sailing-orders.json` is the natural authority for mission-level target identity. It already records outcome, metric, deadline, budget, constraints, scope, stop criteria, and handoff artifacts. `battle-plan.json` is the execution authority for tasks and squadron formation, and `fleet-status.json` is only a live snapshot. Task records carry file ownership and modification targets, but there is no field for target repository, clone path, target ref, or per-ship working directory. Composite `form` is the most practical ingestion point for new battle-plan metadata because it already preserves selected top-level planning fields and builds all task records in one path.

The hook scout found the most important hazard. Hooks discover active missions by scanning `cwd/.nelson/.active-*`. That works when ships run inside the same repository that owns the mission artifacts. It fails open when a captain's `cwd` is the target worktree and the active mission remains in the Nelson control workspace. The same cwd assumption affects `admiral.session`, Agent preflight, TaskCreate enforcement, TaskCompleted gates, and idle-ship checks. Existing file-ownership enforcement also checks only duplicate declared ownership at launch; it does not enforce normal Write/Edit paths against a captain's assigned files or against a target repository root.

The documentation scout found that the user-facing doctrine assumes one codebase root. `README.md`, `SKILL.md`, `structured-data.md`, the sailing orders template, the estimate template, the battle-plan template, captain and marine brief templates, quarterdeck reports, turnover briefs, and the captain's log all need the distinction between Mission Directory and Target Repository. The existing words "the codebase" and "file ownership" are currently ambiguous once the code being modified lives outside the mission-control workspace.

Existing prior art helps. `nelson_conflict_scan.py` already accepts a separate `--root`, which is close to the target-repository concept. Recovery already has stronger active-mission lookup than hooks, but it is still rooted at a supplied or default `.nelson/missions` path. The agent-agnostic design notes already name worktree/external isolation as a platform concern. This mission should use those patterns rather than inventing a separate orchestration layer.

The reconnaissance does not suggest a full remote-execution service. The needed change is smaller and sharper: make the target repository first-class mission metadata, propagate the target root into battle plans, captain briefs, runtime status, and handoff artifacts, and make hooks capable of finding the active Nelson mission even when payload `cwd` is the target worktree. Network cloning should remain an explicit, user-approved operation outside the default path; the durable contract should work with an already-existing local source repository and with an already-created target checkout.

### Addendum - Worktree Provider

The user demonstrated their normal worktree creation path:

`/home/skogix/.cargo/bin/wt -C /home/skogix/dash-skogai switch --create mytestworktree-showing-codex-nelson --no-cd`

That command created a branch and worktree at `/home/skogix/dev/dash-skogai/mytestworktree-showing-codex-nelson` and fired the user's `skogcli` worktree lifecycle scripts (`worktree-pre-switch`, `worktree-pre-start`, `worktree-post-switch`, `worktree-post-start`). This is material terrain. Nelson should prefer an explicit checkout provider model over hard-coding `gh repo clone`: `provider: wt` for an already-existing local source repository, and `provider: existing` when the user supplies an already-created target checkout. `gh-clone` should not be a default fallback for this user's setup, because the important input is `foo <PATH-TO-ALREADY-EXISTING-REPO-TO-CREATE-WORKTREE-FROM>`; if the source repository does not exist, the user's worktree lifecycle does not apply.

The default branch name for Nelson-created worktrees should be `nelson/<mission-id>`, where `<mission-id>` is the already-issued Nelson session id. The exact worktree location should be provider-owned rather than Nelson-owned. Confirmed examples:

- `/home/skogix/.cargo/bin/wt -C /home/skogix/dash-skogai switch --create nelson/761d8f37 --no-cd` created `/home/skogix/dev/dash-skogai/nelson-761d8f37`.
- `/home/skogix/.cargo/bin/wt -C /home/skogix/dot-skogai switch --create nelson/761d8f37 --no-cd` created `/home/skogix/dev/dot-skogai/nelson/761d8f37`.

The provider can create the folder, but repo-specific path sanitization may differ. Nelson must therefore capture the actual path from the provider result or accept it as an explicit field; it should not assume one fixed layout such as `.nelson/missions/<mission>/repo`. The hard requirement is simpler: each target repository has a checked-out branch in a known folder, and Nelson records that folder in mission data and briefs. Mission state should remain in a single Nelson control repository. Multi-repo missions should be explicit, and the useful rule of thumb is one ship per target repository so file ownership, validation commands, and rollback remain anchored to one working root per captain.

## 2. Intent

Nelson must be able to command work in an external target repository while keeping mission command, records, and recovery in the Nelson control repository. The change should make the target checkout a first-class mission concern, not an implied `cwd`, so captains receive an unambiguous working root and hooks can still find the active mission. For this user's default path, Nelson should create or use a worktree from an already-existing local source repository through the user's `wt` provider, preserving lifecycle scripts and refusing silent clone fallback when the source is absent. Existing single-repository Nelson missions must continue to work unchanged.

## 3. Effects

### Effect: Record target repository metadata as mission authority

Nelson mission data must describe the target repository separately from the mission-control repository, including provider, source repository path, target working root, branch name, mission id, and whether the mission is single-repo or explicitly multi-repo.

**Commander's guidance:** Extend the existing mission-data path rather than introducing a parallel state store. `sailing-orders.json` should remain the mission-level authority, with battle-plan and task records inheriting or referencing that target metadata as needed. Default to one primary target repository per mission; multi-repo missions should be explicit and should preserve the rule of one ship per repository.

**Acceptance criteria:**
- A mission can record a target repository distinct from the Nelson control repository.
- Existing missions without target metadata remain valid and retain current local-repo behavior.
- Default Nelson-created branch names follow `nelson/<mission-id>`.
- Multi-repo metadata is possible only by explicit mission configuration, not by accidental inference.

### Effect: Define the worktree provider resolution and creation contract

Nelson must resolve the target checkout through a provider contract that prefers the user's `wt` tool when available and uses an already-existing local source repository as the normal input.

**Commander's guidance:** Treat `wt` as the preferred provider for this environment because it fires user lifecycle scripts. Use a provider model such as `wt` for worktree creation and `existing` for already-created target checkouts. Do not silently fall back to `gh repo clone` when the configured local source path is missing; network clone remains an explicit, approved operation.

**Acceptance criteria:**
- Provider resolution can select `wt` when available and configured.
- Worktree creation uses the source repository path supplied by the mission, not the Nelson control repository by accident.
- Missing source repository paths fail clearly without cloning as a fallback.
- Created target branches default to `nelson/<mission-id>` unless the mission explicitly overrides the branch.
- Existing target checkouts can be used without invoking a creation provider.

### Effect: Make hook and runtime lookup work across separated roots

Nelson hooks and runtime checks must find the active mission even when a captain runs inside the target repository rather than the control repository.

**Commander's guidance:** Replace plain `cwd/.nelson/.active-*` assumptions with an explicit control-root or mission-dir lookup path that can travel through captain briefs, environment, or structured task data. Keep mission artifacts in the control repository. Hook behavior should remain backward compatible when `cwd` is the control root.

**Acceptance criteria:**
- Agent preflight, session checks, task enforcement, completion gates, and idle-ship checks can locate the active mission from a target checkout.
- Mission artifacts continue to be written under the control repository's `.nelson/missions/...`.
- Hooks still work for current single-root missions without additional configuration.
- Runtime failures distinguish "no active mission" from "active mission exists in another control root but was not supplied."

### Effect: Propagate target roots into captain briefs and task templates

Captain-facing artifacts must state the mission-control location and the target working root separately so ships know where to read, write, test, and report.

**Commander's guidance:** Update battle-plan, ship manifest, captain brief, marine brief, quarterdeck, turnover, and log templates only as needed to carry the target repository contract. Preserve the existing Nelson voice and avoid duplicating state by hand where generated structured data can supply it. For multi-repo missions, assign each captain one target repository unless the user explicitly approves otherwise.

**Acceptance criteria:**
- Captain briefs include both mission directory/control root and target working root.
- Task modification targets and file ownership are interpreted relative to the correct target repository.
- Handoff artifacts and turnover reports identify which target repository was changed.
- Existing brief generation remains readable and unchanged in meaning for single-root missions.

### Effect: Cover repository-target behavior with focused tests and docs

The implementation must be verified and documented so future missions can use external targets without losing Nelson state or lifecycle behavior.

**Commander's guidance:** Add focused tests around mission metadata parsing, provider resolution, branch naming, hook mission lookup, and brief/template propagation. Update user-facing and reference docs to define Mission Directory, control repository, source repository, target repository, provider, and target working root. Documentation should describe `wt` as the preferred local worktree path for this setup and make clone/network behavior explicitly opt-in.

**Acceptance criteria:**
- Tests demonstrate that target repository metadata is recorded and consumed without breaking existing mission data.
- Tests cover `wt` provider selection, missing-source failure, existing-checkout behavior, and default branch naming.
- Tests or fixtures demonstrate hook/runtime mission lookup from a separated target root.
- Documentation explains the separated-root model and the default one-primary-repo mission shape.
- Existing test suites relevant to Nelson mission data, hooks, and templates pass after the change.

## 4. Terrain

The first effect lands in the structured mission-data path. The likely primary files are `skills/nelson/scripts/nelson-data.py`, `skills/nelson/scripts/nelson_data_utils.py`, `skills/nelson/scripts/nelson_data_fleet.py`, and the existing tests under `skills/nelson/scripts/test_nelson_data*.py`. `sailing-orders.json` should carry mission-level target repository metadata; `battle-plan.json` and task records should inherit or reference it during `form`. `fleet-status.json` should remain a runtime snapshot, not the source of authority.

The provider contract lands beside mission creation and task formation. `skills/nelson/scripts/nelson-data.py` is the command surface most likely to accept target-repository fields, provider selection, branch naming, and existing-checkout inputs. Shared parsing or validation belongs in `skills/nelson/scripts/nelson_data_utils.py` if it is used by more than one command. Tests should sit near `test_nelson_data.py`, `test_nelson_data_patterns.py`, or a focused new data test if the existing files would become diffuse.

The separated-root runtime work lands in `hooks/nelson_hooks.py`, `hooks/hooks.json`, and `hooks/test_nelson_hooks.py`. The important change is mission discovery: hooks must be able to resolve the control mission directory from explicit context when `cwd` is the target checkout. Backward-compatible `cwd/.nelson/.active-*` discovery remains part of the terrain, but it can no longer be the only path.

The captain-facing propagation lands in the planning references and templates: `skills/nelson/references/admiralty-templates/battle-plan.md`, `ship-manifest.md`, `crew-briefing.md`, `marine-deployment-brief.md`, `quarterdeck-report.md`, `turnover-brief.md`, and `captains-log.md`. The doctrine files `skills/nelson/SKILL.md`, `skills/nelson/references/structured-data.md`, and `skills/nelson/references/the-estimate.md` need only enough language to keep Mission Directory, control repository, source repository, and target working root distinct.

The documentation and verification effect lands in `README.md`, `docs/project_structure.md`, and focused reference material under `skills/nelson/references/`. The strongest test anchors are the mission-data tests, hook tests, and conflict-scan tests. `skills/nelson/scripts/nelson_conflict_scan.py` already has a separate `--root` concept and should be treated as prior art rather than rewritten.

## 5. Forces

Recommended mode: `agent-team`. The work has several separable fronts, but the data contract, hook lookup, templates, and documentation must converge on one vocabulary. Captains benefit from a shared task list and peer visibility more than isolated returns.

Use four captains:

1. Mission Data Captain, a destroyer, owns structured target repository metadata, provider fields, branch naming, and data tests.
2. Runtime Hooks Captain, a destroyer, owns active mission lookup from separated roots, hook behavior, and hook tests.
3. Briefing and Templates Captain, a frigate, owns battle-plan, captain, marine, quarterdeck, turnover, and log template propagation.
4. Documentation and Integration Captain, a frigate, owns README/reference documentation, terminology alignment, and final cross-surface checks.

Add one red-cell navigator. This is medium-risk coordination work: a small mistake can leave ships writing in the wrong repository or hooks failing open. The navigator should review the target-root contract, fallback behavior, and any place where `cwd` still silently means both control root and target root.

Crew should stay light. The Mission Data and Runtime Hooks captains may each use one tester if their implementation grows beyond a narrow patch. The Template and Documentation captains should implement directly unless they need a reviewer for terminology consistency. No captain needs broad marine support at the outset.

## 6. Coordination

The mission data contract is the keel. It must establish names and structured fields before hooks, templates, and docs settle. Start with the primary target repository schema, provider values, branch default, and compatibility behavior for missions without target metadata.

Runtime hooks can proceed in parallel once the control-root lookup contract is clear. Their work depends on knowing how a captain receives or exposes the mission directory, but not on every template word being final.

Templates and documentation can begin as a terminology pass in parallel, but final wording should wait for the data contract. They should avoid inventing field names the implementation does not use. The rule is simple: Mission Directory and control repository describe Nelson state; source repository and target working root describe the code under command.

Suggested execution order:

1. Define and test the structured target repository contract.
2. Implement separated-root mission lookup in hooks against that contract.
3. Propagate mission-control and target-root fields into briefs, reports, and logs.
4. Update docs and reference doctrine with the final terms and examples.
5. Run integration checks across data, hooks, templates, and docs.

Parallel tracks are safe after step 1. Mission Data and Runtime Hooks should coordinate on field names and environment or briefing transport. Templates and Documentation should coordinate on terminology and examples. Multi-repo behavior should remain explicit and modest: one ship per repository unless the user approves otherwise.

## 7. Control

Quality gates:

- Data gate: mission metadata records provider, source repository, target working root, branch, mission id, and single-repo versus explicit multi-repo status without breaking older missions.
- Provider gate: `wt` creation uses `/home/skogix/.cargo/bin/wt -C <source_repo> switch --create nelson/<mission-id> --no-cd`, captures or records the actual `working_dir`, and fails clearly when the local source repository is missing.
- Runtime gate: hook tests prove active mission lookup works from a target checkout and from the control repository.
- Briefing gate: captain-facing artifacts show both mission-control location and target working root, with file ownership interpreted relative to the target root.
- Documentation gate: README and reference docs explain the separated-root model, preferred `wt` provider, explicit clone/network behavior, and the one-primary-repo default.

Intervention points:

- Stop after the data contract if field names or provider semantics are disputed.
- Stop if `wt` output cannot be parsed or reliably captured; the fallback should be explicit target working root input, not guessed layout.
- Stop if hooks cannot receive the control mission directory without broadening trust or weakening enforcement.
- Stop before enabling multi-repo automation beyond recording explicit metadata.

Action-station tiers:

- Mission Data Captain: Station 2. Shared schema and compatibility behavior are central, but the blast radius is bounded by tests.
- Runtime Hooks Captain: Station 3. Hooks enforce mission discipline and may fail open or block valid work if lookup is wrong.
- Briefing and Templates Captain: Station 2. Template drift can misdirect captains, but changes are reviewable.
- Documentation and Integration Captain: Station 1. Mostly prose and verification, with escalation if integration checks expose contract flaws.

Rollback and cleanup:

Structured metadata changes should be backward-compatible and can be rolled back by ignoring absent target fields. Provider-created worktrees should use branch `nelson/<mission-id>` and be discoverable with `wt list`; cleanup should use `wt remove <branch>` only when the mission explicitly abandons or completes that checkout. Hooks should preserve the current `cwd/.nelson/.active-*` path as the compatibility fallback. If separated-root support proves unsafe during implementation, stand down with docs and tests describing the blocked contract rather than shipping partial runtime enforcement.
