# The Estimate — Fleet research → ONBOARDING.md

## 1. Reconnaissance

Three Explore ships were sent out: one into the skill's own architecture
(`skills/nelson/`), one into the repo's tooling and CI (`hooks/`, `scripts/`,
`.github/`, lint/test config, plugin manifests), and one into the existing
documentation set (README, CONTRIBUTING, AGENTS.md, CLAUDE.md,
`docs/project_structure.md`, the design-plan and research docs) to find
what's already written down before we add to it.

**The skill itself.** `skills/nelson/SKILL.md` (479 lines) is the operational
core: the eight steps from Sailing Orders through Stand Down, each backed by
reference material under `skills/nelson/references/` — eleven
`admiralty-templates/` (the prose/JSON scaffolds for each step's artifact),
eleven `damage-control/` playbooks (one per recovery scenario — man
overboard, relief on station, scuttle and reform, and so on), and seventeen
`standing-orders/` (named anti-patterns the admiral scans for at every
checkpoint). A dozen further top-level reference files cover doctrine that
doesn't belong to a single step: `model-selection.md`, `squadron-composition.md`,
`crew-roles.md`, `the-estimate.md` itself, and the rest. All the actual data
plumbing lives in `skills/nelson/scripts/` — `nelson-data.py` is the single
CLI for every mission-lifecycle write, `nelson-phase.py` enforces the linear
phase state machine, and a family of `nelson_data_*.py` / `nelson_*scan*.py`
modules back specific subcommands (conflict scanning, circuit breakers,
cross-mission memory, and — notably — a self-amending standing-orders miner
in `nelson_data_patterns.py` that is explicitly restricted to *proposing*
new orders for human review, never loosening existing ones). `agents/nelson.md`
is a thirteen-line shim, not a second copy of the doctrine — it just points
the `nelson` agent persona at the `nelson` skill.

**Repo tooling.** Two directories ship inside the installed plugin:
`hooks/` (five Claude Code hook events wired through `hooks/hooks.json` to
`nelson_hooks.py` — preflight checks on `Agent` spawns, turnover-brief
validation, task-completion gating, idle-ship advisories, session-init) and
`skills/nelson/scripts/`. A second, separate top-level `scripts/` directory
is repo-maintainer-only tooling (`check-references.sh` verifies every
reference cited in SKILL.md actually exists; `count-tokens.py` is a manual
hull-integrity utility) — it is not part of the plugin payload, and the
README says so explicitly. `settings.json` turns on the experimental
agent-teams flag and gates every `Agent`/`TeamCreate`/`TaskCreate` call
through `nelson-phase.py validate-tool`, failing open if the mission scripts
aren't present. Quality is enforced in layers: `.pre-commit-config.yaml`
(ruff, mypy on an explicit allow-list, gitleaks secret scanning, standard
hygiene hooks), `pyproject.toml` (ruff thresholds tuned for AI-agent-shaped
mistakes — `max-args=5`, `max-branches=10`, `max-complexity=10`,
`line-length=120` — each annotated with the failure mode it targets), and
a single CI workflow (`.github/workflows/ci.yml`) with eight parallel jobs
mirroring all of the above plus markdown lint, link checking, and spell
checking. `.claude-plugin/plugin.json` is the single source of truth for
plugin metadata; `.cursor-plugin/plugin.json` is literally a symlink to it.

**Existing documentation — the decisive finding.** Every file checked —
`README.md`, `CONTRIBUTING.md`, `AGENTS.md`, `CLAUDE.md`,
`docs/project_structure.md`, `docs/plan-agent-agnostic.md`,
`docs/plan-the-estimate.md`, and all three `docs/research-*.md` comparative
studies — is **byte-identical to `upstream/main`**. Zero fork-specific
content, confirmed by `git diff upstream/main --stat` returning nothing for
any of them, and zero hits for "skogix" or "skogai" anywhere in that set.
`git remote -v` confirms the chain: `origin` = `skogai/nelson` (this fork),
`upstream` = `Aspegio/nelson`, which is itself a fork of `harrymunro/nelson`
— two hops upstream, not one. The only fork-specific material anywhere in
the tree is `skogai/experiments/` (salvaged fleet-memory skill, persona
index, a design memo — explicitly excluded from this mission by the user),
plus runtime state: `.nelson/missions/`, `.nelson/.active-*` markers, and
`.claude/worktrees/`. None of it is documented anywhere.

**A stale mission, found in passing.** Before sailing orders were drafted,
a check of `.nelson/` turned up an abandoned mission —
`.nelson/missions/2026-07-06_223339_761d8f37/` — stuck at `BATTLE_PLAN`
phase since July, outcome "add external-repository-target support to
Nelson missions," battle plan drafted (`battle-plan.md`, 10.7KB) but never
formed, approved, or stood down. Its `.active-761d8f37` session marker is
still live on disk. This matches two recent commits on `master`
(`c9f9ef0 Add battle-plan for external repository missions`,
`a69e550 battle plan`) — the battle plan was apparently committed to the
repo directly rather than being an artifact of a completed mission. Per
sailing orders this is reported, not fixed.

**Surprise for Checkpoint 1.** Reconnaissance substantially changes what an
"introduction to how we do things around here" can contain. Nearly
everything that could be documented about *using* Nelson already is,
upstream, and duplicating it would drift out of sync immediately. The
genuine gap — the material with no home anywhere — is narrow: the fork's
remote chain and what forking buys this environment, the stale/orphaned
mission as a cautionary pattern, and a couple of local-only conventions
(`.nelson/` runtime hygiene, `settings.json`'s phase-gate hook,
`.claude/worktrees/`). With `skogai/experiments/` explicitly out of scope,
that gap is smaller still. This reframes the mission from "write a broad
onboarding doc" to "write a short, pointed doc that links out to the
upstream docs for everything they already cover, and documents only the
fork-specific sliver that has no other home."

## Addendum — Checkpoint 1 outcome

The user read README's actual Quick Start/Usage section (reproduced
verbatim at Checkpoint 1) and confirmed the suspicion above cuts deeper
than first framed: that section tells a reader *what to type*, not *what
happens on disk, in order* — no trace of mission directories, phase
transitions, or the JSON/markdown artifacts filling up as a mission runs.
The user reframed the deliverable accordingly: drop "ONBOARDING.md" as a
generic intro, and instead produce **`DASHBOARD.md`** — a doc explaining
Nelson's mission-progress/reporting model specifically: what artifacts
exist (`sailing-orders.json`, `battle-plan.json`, `fleet-status.json`,
`mission-log.json`, `quarterdeck-report.md`, damage reports,
`captains-log.md`), where they live, and how to read them to know where a
mission stands — using this very mission as the worked example. A second
deliverable change: the captain's log improvement backlog must explicitly
propose a lightweight dashboard tool/command (e.g. a richer
`nelson-data.py status` or a small renderer over `fleet-status.json`
across missions) as a scoped, justified next-mission item — not built this
session, kept to documentation-only per the amended sailing orders
(v2, recorded in `sailing-orders.json`). Sailing orders v1 is preserved in
that file for context.

## 2. Intent

Right now, knowing where a Nelson mission stands means already knowing Nelson cold. The README's Quick Start tells a reader what to type, not what lands on disk, in what order, or what any of it means once it's there — and that gap isn't theoretical: a mission from July sits abandoned at `BATTLE_PLAN` with its session marker still live on disk, and nothing short of a maintainer manually walking `.nelson/missions/` would ever have surfaced it. The fix is not another onboarding doc — upstream already covers general usage, and restating it would only drift out of sync — it's a short, fork-specific explainer of the mechanical trace a mission actually leaves behind: which artifacts appear at which phase, what each one means, and how to read them cold, proven against this very mission's own files rather than described in the abstract. The improvement backlog that rides alongside it matters for the same reason the doc does: documenting that trace by hand is itself evidence the real fix is a tool that reads it for you, so that proposal needs to be scoped and argued now, ready to pick up as its own mission, even though nobody builds it this session.

## 3. Effects

### Effect: Write the state-model explainer

`DASHBOARD.md`, landing at the repo root, must give a reader the inventory and the map: every artifact a mission can produce (`sailing-orders.json`, `battle-plan.json`, `mission-log.json`, `fleet-status.json`, `estimate.md`, `quarterdeck-report.md` and its rotated siblings, `damage-reports/{ship}.json`, `turnover-briefs/`, `captains-log.md`), where each lives under `.nelson/missions/{YYYY-MM-DD_HHMMSS}_{8hex}/`, and which phase of `nelson-phase.py`'s state machine (`SAILING_ORDERS -> ESTIMATE -> BATTLE_PLAN -> FORMATION -> PERMISSION -> UNDERWAY -> STAND_DOWN`) brings each one into existence or updates it. This is the section that turns "a folder full of JSON and markdown" into a readable state model.

**Commander's guidance:** Organize by phase, not by file type — a reader tracking a live mission thinks "what phase are we in, what should exist now" before they think "what does `fleet-status.json` contain." Name `nelson-data.py status` as the closest thing to an existing dashboard today, and say plainly what it doesn't cover (single active mission only, no cross-mission view) — that honesty is what justifies the backlog effect below. Link to `docs/project_structure.md`, the README, and `skills/nelson/references/the-estimate.md` rather than re-explaining what they already cover; this section earns its place only where no other doc does.

**Acceptance criteria:**
- Every artifact filename and path named in this section exists somewhere under this mission's own directory, or is named as "produced at a later phase, not yet written" where it genuinely isn't there yet — reviewer checks each against `.nelson/missions/2026-10-02_181703_3d83347e/` directly.
- The phase list matches `nelson-phase.py`'s actual state machine verbatim — reviewer diffs against the script.
- No paragraph restates content already covered in README, CONTRIBUTING, AGENTS.md, CLAUDE.md, or `docs/project_structure.md`; those are linked, not duplicated — reviewer spot-checks for overlap.

### Effect: Write the mechanical worked-example walkthrough

A second section of `DASHBOARD.md` walks a reader through reading an actual mission's state cold, using this mission (`3d83347e`) as the specimen: what `fleet-status.json` showed at `ESTIMATE` phase, what `mission-log.json`'s phase-transition event records, and — once they exist by the time this doc ships — what `battle-plan.md`, `quarterdeck-report.md`, and `captains-log.md` add at their respective phases. The stale mission at `.nelson/missions/2026-07-06_223339_761d8f37/` belongs here too, as the cautionary half of the same walkthrough: a `BATTLE_PLAN`-phase mission with a battle plan drafted, never formed or stood down, its `.active-761d8f37` marker still live — exactly the kind of state a reader should now be able to diagnose at a glance.

**Commander's guidance:** Quote or excerpt real file contents from this mission's directory rather than inventing illustrative JSON — the worked example's whole value is that it's checkable. Keep the stale-mission passage descriptive, not corrective: name the path, the phase, the marker, and what a healthy mission would show instead at that phase; do not propose fixing it here, only in the backlog effect's cleanup item.

**Acceptance criteria:**
- Every quoted or described field traces to a real file this mission produced, re-read at the time `DASHBOARD.md` is finalized — reviewer re-reads the cited files and confirms the match.
- The stale-mission passage names the path and phase and stops there — no remediation steps, no edits proposed to that mission's files — reviewer confirms by reading the passage against the out-of-scope constraint.
- A reader unfamiliar with Nelson can follow the walkthrough and correctly state, for a hypothetical mission directory, roughly what phase it's in from the files present — reviewer read-back check.

### Effect: Draft the ranked improvement backlog with the dashboard-tool proposal

The captain's log (written at stand-down, per the normal lifecycle) must carry a ranked backlog of improvements surfaced by this mission's research, with the dashboard-tool proposal as a named, scoped entry: something like a richer `nelson-data.py status` or a small renderer over `fleet-status.json` across missions, pitched as a future mission's outcome, not built now. The orphaned-mission-and-marker cleanup belongs in this same backlog as its own ranked item, distinct from the tool proposal — the stale mission is both evidence for the tool (effect above) and a standalone cleanup task in its own right.

**Commander's guidance:** Rank by a stated axis (impact, effort, or both) rather than listing in discovery order — a backlog a future captain can actually triage, not a stream of consciousness. The dashboard-tool entry needs enough shape to be picked up cold later: what it would read, what it would surface, what today's `nelson-data.py status` already does that it would extend rather than replace. Keep every entry honest about what's proposal versus what's already partially true (e.g. `nelson-data.py status` existing today is a foundation, not a gap).

**Acceptance criteria:**
- The backlog is ranked, with the ranking axis stated explicitly, not left implicit — reviewer reads the list and confirms a stated axis is present and applied consistently.
- The dashboard-tool entry names what it would read, what gap in today's `nelson-data.py status` it closes, and that it is a scoped future mission, not this session's work — reviewer checks the entry against the out-of-scope constraint barring the tool's construction now.
- The stale-mission cleanup appears as its own ranked entry (path and phase named) separate from the tool proposal — reviewer confirms both entries exist and are distinct.

## 4. Terrain

One file is touched by this mission, and it does not exist yet: `DASHBOARD.md` at the repo root. Both effects write into it — the artifact inventory organized by phase, then the worked-example walkthrough — and nothing else on disk changes. No skill logic, script, or hook is edited; that matches the standing constraint carried over from sailing orders v1 and restated for v2, "stay documentation-only for this mission's own deliverable."

Everything else this mission touches is read, not written. The captain reads this mission's own directory, `.nelson/missions/2026-10-02_181703_3d83347e/` — `estimate.md`, `sailing-orders.json`, `fleet-status.json`, `mission-log.json` — as the live worked example, and the stale mission's directory, `.nelson/missions/2026-07-06_223339_761d8f37/`, as the cautionary one. It reads `skills/nelson/scripts/nelson-phase.py` to quote the state machine verbatim rather than paraphrase it, and `README.md`, `CONTRIBUTING.md`, `docs/project_structure.md`, and `skills/nelson/references/the-estimate.md` to link out rather than duplicate, per the Reconnaissance finding that upstream already owns that ground. None of these are modified; the sailing-orders constraint against touching the stale mission "beyond reporting on it" is satisfied by construction, since nothing is written there at all.

Blast radius is about as small as a mission gets: a markdown file nobody references yet, with no importer, no CI step, no downstream mission pointed at it. The only test suites in this repo that could notice `DASHBOARD.md` existing are the documentation-hygiene ones — markdown lint, link checking, spell checking in `.github/workflows/ci.yml` — and those are satisfied by writing well-formed, correctly-linked prose, not by any functional change.

## 5. Forces

One captain, no crew, no red-cell navigator, `subagents` mode.

Start from `admiral-at-the-helm`: writing `DASHBOARD.md`'s content is implementation work, so the admiral cannot draft it directly, no matter how small the file. At least one ship must be spawned. Then `becalmed-fleet`: Effects 1 and 2 share the same file and have no independent branch between them — the inventory section and the worked-example section are sequential by nature, not parallel work that benefits from two sets of hands. A squadron here would be two captains negotiating edits to one file for a task that doesn't decompose, which is the anti-pattern by name. One captain, covering both effects as a single task, is what the two orders jointly require.

That leaves the mode name worth being precise about, because `becalmed-fleet`'s own remedy text says "use `single-session` mode," and `single-session` is defined elsewhere (`references/tool-mapping.md`) as "no spawning — the admiral executes all work directly." Taken literally, that collides head-on with `admiral-at-the-helm`. The two standing orders aren't actually in tension, though — `becalmed-fleet`'s real concern is coordination overhead with no parallel throughput gain, and `single-session` is just the mode that happens to carry zero coordination overhead when the admiral is allowed to do the work itself. Here the admiral isn't allowed to, so the mode that satisfies both orders is `subagents`: one captain spawned directly via `Agent` with `subagent_type`, reporting only to the admiral, no `TeamCreate`, no shared task list, no peer messaging. It has exactly the coordination overhead of `single-session` — none — while still delegating the actual writing, which is the one thing `single-session` can't do here.

Ship class: a captain implementing directly, no crew. Per the role guide, a captain implements directly only when the task is atomic, and this one is — one ship reads real files under two mission directories and writes two sections of prose into one new file. There's no sub-task boundary worth a crew role: no code to build (ruling out MEO/WEO), no test harness to run (ruling out a tester), no logistics or navigation concern distinct from the writing itself. A red-cell navigator is likewise not warranted; `squadron-composition.md` pairs red-cell with high threat or high blast radius, and Q4 already establishes this as close to zero blast radius, with acceptance criteria that are reviewer-checkable file comparisons rather than adversarial-review-shaped judgment calls.

Model: the sailing orders carry no cost-savings language — no token budget, no "keep costs low" — so the default-weight threshold table in `model-selection.md` never engages. The captain inherits the admiral's model; no `model: "haiku"` override, no haiku briefing enhancements.

## 6. Coordination

The dependency graph is as flat as this mission gets: one task, no predecessor beyond the mission's own phase gate (Battle Plan approved, squadron formed, permission granted). Internally, the captain writes the inventory section before the worked-example section — Commander's Guidance on Effect 1 and 2 implies that ordering, since the walkthrough reads more naturally once the reader already has the phase-by-phase map — but that's an ordering within one ship's own pass over one file, not a dependency between two tasks that needs tracking.

No parallel tracks are expected, and that's a direct consequence of `becalmed-fleet` rather than an oversight: splitting Effect 1 (inventory) and Effect 2 (worked example) across two captains would hand them the same file to edit, which only manufactures a merge step that buys nothing — there is exactly one independently executable work unit here, not two, so the team-sizing rule in `squadron-composition.md` ("map the dependency graph and count how many tasks can run concurrently with zero shared state") returns one.

The one coordination surface that does exist is access, not sequencing. The captain needs read access to this mission's own directory, `.nelson/missions/2026-10-02_181703_3d83347e/`, as its live worked-example source, and to the stale mission's directory, `.nelson/missions/2026-07-06_223339_761d8f37/`, as the cautionary source — both read-only, consistent with the sailing-orders constraint against touching the stale mission beyond reporting on it. Write access is scoped to `DASHBOARD.md` and nothing else; the captain has no call to touch `skills/`, `hooks/`, or `scripts/`, matching the read-only-research constraint from sailing orders v1 that v2 carries forward. In `subagents` mode the captain reports completion straight back to the admiral through the `Agent` return value — there's no shared task list to post to and no second ship to coordinate with.

## 7. Control

Running the Station decision tree: the action is not irreversible or regulated — a markdown file, trivially edited or deleted, nothing backed by it elsewhere. It carries no security, privacy, or data-integrity implication — no auth, no PII, no financial surface. It isn't user-visible application behavior and it isn't coupled to other in-flight tasks, per the Coordination section above. None of Questions 1 through 3 fire, which lands this squarely at **Station 0: Patrol**.

Patrol's required controls are light by design, and both are already built into the Effects: basic validation evidence is the acceptance criteria themselves — every artifact path and filename checked against the real mission directories, the phase list diffed against `nelson-phase.py` verbatim, every quoted field re-read from its source file at the time the doc is finalized. A recorded rollback step is equally simple to state because it's genuinely simple: `DASHBOARD.md` is new and unreferenced by anything else in the tree, so an unreviewed or unwanted version can be edited or deleted outright, with no downstream mission, test, or doc pointed at it yet. Plan Mode, which `action-stations.md` reserves for Station 2 and 3, isn't called for — the captain executes directly.

The intervention point is the ordinary quarterdeck one, not a red-cell gate: before Stand Down, the admiral (and the user, at final review) reads `DASHBOARD.md` and checks it against the real `.nelson/` files it claims to describe — the same check the acceptance criteria already specify, just performed by a human eye before the mission closes rather than left to the reviewer's trust alone.
