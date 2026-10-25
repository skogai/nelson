# Estimate-Drafter Briefing — Dispatch 1 (Q2 Intent, Q3 Effects)

## Sailing orders (current — v2, amended)

```json
{
  "outcome": "Produce DASHBOARD.md documenting Nelson's mission-progress/reporting model for this fork — what artifacts a mission produces, where they live, and how to read them to know where a mission stands — plus a ranked improvement backlog in the captain's log that explicitly proposes a lightweight dashboard tool/command as a scoped next-mission item (not built this session).",
  "success_metric": "DASHBOARD.md lands at repo root, reviewed, and explains the reporting/state model concretely (using this mission's own artifacts as the worked example); captain's log carries a ranked improvement backlog including a scoped dashboard-tool proposal; stale mission state is reported, not fixed.",
  "deadline": "this session",
  "constraints": [
    "Read-only research — no edits to skill logic, scripts, or hooks.",
    "Do not touch the stale mission (2026-07-06_223339_761d8f37) beyond reporting on it.",
    "Stay documentation-only for this mission's own deliverable — do not build the dashboard tool now, only scope and justify it in the backlog."
  ],
  "out_of_scope": [
    "skogai/experiments/* — explicitly excluded from this mission by the user.",
    "Resolving or reconciling in-flight branches.",
    "Building the stale mission's repo-target feature.",
    "Fixing any bugs found during research.",
    "Building the proposed dashboard tool itself — propose only."
  ],
  "stop_criteria": [
    "DASHBOARD.md written and reviewable at repo root.",
    "Captain's log written with a ranked improvement backlog."
  ]
}
```

v1 of the sailing orders (generic "ONBOARDING.md") is preserved in
`sailing-orders.json`'s `amendments` array for context — v2 above is the
live mission.

## Q1 Reconnaissance (already complete)

Read `.nelson/missions/2026-10-02_181703_3d83347e/estimate.md` section
"## 1. Reconnaissance" plus the "## Addendum — Checkpoint 1 outcome" that
follows it. That addendum is the most important part — it records why the
deliverable changed from a generic onboarding doc to a reporting/dashboard
explainer, and what the user specifically found missing (the mechanical
trace of what happens on disk, in order, when a mission runs).

Key facts from reconnaissance you should build on, not re-derive:
- Mission state lives under `.nelson/missions/{YYYY-MM-DD_HHMMSS}_{8hex}/`:
  `sailing-orders.json`, `battle-plan.json` (written at Step 3),
  `mission-log.json` (append-only event log), `fleet-status.json`
  (current snapshot: phase, progress, budget, squadron, blockers),
  `estimate.md`, `quarterdeck-report.md` (checkpoint snapshots, rotated as
  `quarterdeck-report-N.md`), `damage-reports/{ship}.json`,
  `turnover-briefs/`, and `captains-log.md` (written at stand-down).
- `nelson-phase.py` enforces a linear phase state machine:
  `SAILING_ORDERS -> ESTIMATE -> BATTLE_PLAN -> FORMATION -> PERMISSION ->
  UNDERWAY -> STAND_DOWN`. `fleet-status.json`'s `mission.phase` field
  reflects current phase.
- `nelson-data.py status` prints a one-line digest (phase, fleet
  done/total, budget %, hull tally, blockers) — this is the closest thing
  to an existing "dashboard" today, and it only covers the single mission
  whose `.active-{id}` marker nelson-data.py resolves to context, not a
  cross-mission view.
- There is a genuinely stale, abandoned mission at
  `.nelson/missions/2026-07-06_223339_761d8f37/` (BATTLE_PLAN phase since
  July, never formed/approved/stood down, `.active-761d8f37` marker still
  live) that is a concrete illustration of "a dashboard would have caught
  this" — useful as a worked example of why the proposed tool matters, and
  as a backlog item in its own right (orphaned mission + marker cleanup).
- This very mission (`3d83347e`) is itself a live worked example: by the
  time DASHBOARD.md ships, its own `estimate.md`, `battle-plan.md`,
  `quarterdeck-report.md`, and `captains-log.md` will exist on disk and can
  be cited directly ("see this mission's own `fleet-status.json` for what
  BATTLE_PLAN phase looks like mid-flight," etc.)

## Your task

Produce two sections, appended to
`.nelson/missions/2026-10-02_181703_3d83347e/estimate.md`:

### Q2 — Intent
One paragraph: the commander's intent. What are we really trying to
achieve and why, in the user's terms. This paragraph will be prepended
verbatim to every captain's brief later, so it must stand alone and convey
*why this matters*, not just *what to build*. Ground it in the Checkpoint 1
finding: the gap isn't "no docs," it's "no mechanical trace of mission
state that a human (or future agent) can read cold."

### Q3 — Effects
Decompose the intent into concrete effects. Each effect needs:
- A short outcome-focused name.
- One paragraph: what must change, where it lands, why.
- **Commander's guidance** — how to do it (structure, tone, what to
  include/exclude, use of this mission as worked example).
- **Acceptance criteria** — what must be true when done, each with an
  implied verification method (review, link-check, read-back-and-confirm,
  etc. — these are documentation effects, not code, so criteria should be
  things like "every artifact named in the doc actually exists in this
  mission's directory and the doc's description matches it," "no content
  duplicates what README/CONTRIBUTING/CLAUDE.md/docs/project_structure.md
  already say — those are linked, not restated," "the stale-mission finding
  is reported with path and phase, not resolved.")

Expect roughly 2-4 effects given the scope (e.g. something like: "Write
DASHBOARD.md's state-model explainer," "Write the mechanical
walkthrough/worked-example using this mission," "Produce the ranked
improvement backlog with the dashboard-tool proposal," — but use your own
judgement on the right split; don't force a count).

## Output requirements

- Append two H2 sections to `.nelson/missions/2026-10-02_181703_3d83347e/estimate.md`:
  `## 2. Intent` and `## 3. Effects`. Do not rewrite section 1 or the
  addendum already there.
- Voice and register: read
  `/home/skogix/.claude/plugins/cache/nelson-marketplace/nelson/2.4.0/skills/nelson/references/the-estimate.md`
  (especially "Voice and register" and "Effects, acceptance criteria, and
  commander's guidance") before writing, and match it — a capable officer
  briefing peers, concise but not terse.
- Do not touch any files outside this mission directory's `estimate.md`.
  This is a planning dispatch, not an implementation one — no edits to
  README, DASHBOARD.md (doesn't exist yet), or anything in `skills/`,
  `hooks/`, `scripts/`.
- Return a short confirmation when done (what you wrote, section
  boundaries) — not the full text back in the response, since it's already
  on disk.
