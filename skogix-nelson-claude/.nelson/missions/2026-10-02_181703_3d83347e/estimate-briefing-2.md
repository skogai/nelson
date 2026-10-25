# Estimate-Planner Briefing — Dispatch 2 (Q4 Terrain, Q5 Forces, Q6 Coordination, Q7 Control)

Read `.nelson/missions/2026-10-02_181703_3d83347e/estimate.md` in full first —
sections 1-3 (Reconnaissance, the Checkpoint 1 addendum, Intent, and the three
approved Effects) are all there and all approved by the user. Do not revise
them. Then read
`/home/skogix/.claude/plugins/cache/nelson-marketplace/nelson/2.4.0/skills/nelson/references/the-estimate.md`
for voice/register and the Q4-Q7 thought process.

Working directory: `/home/skogix/nelson`.

## Context you need that isn't in the estimate yet

Two standing orders constrain how this mission must be staffed, and you
should reason from them explicitly in Forces/Coordination/Control rather
than defaulting to a generic plan:

- **`admiral-at-the-helm.md`**: the admiral must not perform implementation
  work (writing `DASHBOARD.md`'s content counts as implementation). The
  admiral IS explicitly permitted to write the captain's log directly at
  Stand Down. So: Effects 1 and 2 (both land on `DASHBOARD.md`) must be
  delegated to a captain. Effect 3 (the ranked backlog, which lives inside
  the captain's log) is the admiral's own permitted work at Stand Down, not
  a squadron task.
- **`becalmed-fleet.md`**: do not form a multi-captain team for linear,
  sequential work with no independent parallel branches. Effects 1 and 2
  both write to the same file (`DASHBOARD.md`) and are sequential by
  nature (inventory section before worked-example section) — this is a
  single-owner, single-file job. One captain, not a squadron.

Given those two, the shape should be: **one captain, one task** (covering
Effects 1 and 2 together, since they share a file and ownership), mode
`subagents` (a single ship reporting synchronously to the admiral — not
`agent-team`, there's nothing to coordinate peer-to-peer). No red-cell
navigator needed (documentation-only, low risk, the acceptance criteria
are reviewer-checkable file comparisons, not adversarial-review-shaped).
No marines (the task doesn't decompose into distinct specialist sub-tasks
— one ship reading real files and writing prose from them is atomic
enough for a captain to do directly, no crew).

Feel free to disagree with this shape if your own reasoning from the
standing orders leads somewhere else — but if you depart from it, say why
in Q5/Q6 explicitly, the same way the Estimate always shows its reasoning
rather than asserting conclusions.

## Your task

Append two more sections to
`.nelson/missions/2026-10-02_181703_3d83347e/estimate.md`:

### Q4 — Terrain
Where this lands: `DASHBOARD.md` (new file, repo root) is the only file
being created or edited. No existing files are modified. No test suites
are affected (documentation-only mission, per sailing orders v2). Blast
radius is effectively zero — a markdown file that doesn't exist yet,
reviewed before stand-down.

### Q5 — Forces
Captain count, ship class/model, crew (if any), red-cell navigator
(if any) — reasoned from the two standing orders above. State explicitly
why no crew/marines/red-cell are needed for this task, don't just omit
them.

### Q6 — Coordination
Dependency graph (should be trivial — one task, no predecessors blocking
it once sailing orders are settled), parallel tracks (none expected —
say so and why, given becalmed-fleet), and the one coordination surface
that does exist: the captain needs read access to this mission's own
directory (`.nelson/missions/2026-10-02_181703_3d83347e/`) and to the
stale mission's directory (`.nelson/missions/2026-07-06_223339_761d8f37/`)
as source material, and write access to `DASHBOARD.md` only.

### Q7 — Control
Action-station tier for the DASHBOARD.md task (reason it out — this is
read-only-sourced documentation work with reviewer-checkable acceptance
criteria, not code with runtime risk; consult
`/home/skogix/.claude/plugins/cache/nelson-marketplace/nelson/2.4.0/skills/nelson/references/action-stations.md`
for the actual tier definitions and pick the one that fits rather than
guessing a number). Intervention points (where the user/admiral should
look before accepting the deliverable — almost certainly: read `DASHBOARD.md`
and sanity-check it against the real `.nelson/` files before stand-down).
Rollback plan (trivial: an unreviewed or unwanted `DASHBOARD.md` can simply
be edited or deleted; nothing downstream depends on it yet).

## Output requirements

- Append `## 4. Terrain`, `## 5. Forces`, `## 6. Coordination`, `## 7. Control`
  as H2 sections to `.nelson/missions/2026-10-02_181703_3d83347e/estimate.md`,
  after the existing `## 3. Effects` section. Do not rewrite anything above
  that.
- Same voice/register as the existing sections — a capable officer
  briefing peers, prose over bullets where the content flows.
- Do not touch any file outside this mission's `estimate.md`. This is
  planning, not execution — no `DASHBOARD.md`, no battle-plan.md yet.
- Return a short confirmation when done — what you wrote, not the full text
  back (it's on disk).
