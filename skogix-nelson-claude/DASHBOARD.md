# DASHBOARD

How to tell where a Nelson mission stands, by reading the files it leaves on disk.

This is fork-specific documentation. It does not repeat what Nelson *does*
or how to install/use it — see [README.md](./README.md) for that, and
[docs/project_structure.md](./docs/project_structure.md) for the full
repository layout. It does not repeat the JSON field-by-field schemas
either — those are already documented, exhaustively, in
[`skills/nelson/references/structured-data.md`](./skills/nelson/references/structured-data.md)
(write timing table, event type catalogue, full schema examples for every
artifact). What's missing from all of that is a *reporting model*: given a
mission directory on disk, right now, cold, what phase is it in and what
should (or shouldn't) be there. That's what this document is for.

The worked example throughout is this very mission — the one that produced
this file — at
`.nelson/missions/2026-10-02_181703_3d83347e/`. Every quoted field below
was re-read from that directory immediately before this document was
finalized.

## 1. The state model, by phase

Nelson missions move through a strictly linear phase state machine,
enforced by
[`skills/nelson/scripts/nelson-phase.py`](./skills/nelson/scripts/nelson-phase.py)
(see its module docstring and `PHASES` tuple — quoted verbatim, not
paraphrased):

```
SAILING_ORDERS -> ESTIMATE -> BATTLE_PLAN -> FORMATION -> PERMISSION -> UNDERWAY -> STAND_DOWN
```

The current phase for a given mission directory is a single string field:
`fleet-status.json` → `mission.phase`. Everything below is organized around
that one field, because that's the question a reader actually has
("what phase are we in, what should exist now") before they care what any
individual file contains. Each phase section below names the mission
directory this document can show you the artifact in: `3d83347e` for
"exists now," or "not yet written" when this mission hasn't reached that
phase yet.

All paths are relative to a mission directory,
`.nelson/missions/{YYYY-MM-DD_HHMMSS}_{8hex}/` — see README's
[Mission artifacts](./README.md#mission-artifacts) section for the full
directory-structure diagram; this section layers phase sequencing on top
of that structure rather than re-drawing it.

### SAILING_ORDERS

**Brings into existence:** `sailing-orders.json`, `mission-log.json`,
`fleet-status.json` (plus empty `damage-reports/` and `turnover-briefs/`
subdirectories), and the `.nelson/.active-{8hex}` session marker one level
up, outside the mission directory itself.

**Means:** the mission's outcome, success metric, constraints, and
stop criteria are committed to disk. `sailing-orders.json` is write-once
in the sense that its core fields don't change, but it is **amendable** —
this mission's own `sailing-orders.json` carries an `amendments` array with
one entry (version 2, amended after Checkpoint 1, reframing the deliverable
from a generic `ONBOARDING.md` to this `DASHBOARD.md`). Amendments are
appended, not overwritten; version 1 stays on record.

**Exit criteria** (from `nelson-phase.py`'s `EXIT_CRITERIA_DESC`):
`sailing-orders.json` must exist with a non-empty `outcome` field.

### ESTIMATE

**Brings into existence:** `estimate.md` (the prose Seven-Question
analysis — see
[`skills/nelson/references/the-estimate.md`](./skills/nelson/references/the-estimate.md)
for the template this follows), plus, in this mission's case,
`estimate-briefing-1.md` and `estimate-briefing-2.md` — numbered checkpoint
briefings written during estimate drafting (not a fixed artifact name every
mission produces; this fork's estimate step visibly used two checkpoints).

**Means:** reconnaissance is done and the plan of attack (Intent, Effects,
Terrain, Forces, Coordination, Control) is written down for review before
anyone touches a file.

**Exit criteria:** `estimate.md` must exist, or `sailing-orders.json` must
carry `estimate_skipped: true`.

### BATTLE_PLAN

**Brings into existence:** `battle-plan.md` (prose — commander's intent,
per-task spec, acceptance criteria, Standing Order Check) and
`battle-plan.json` (structured — tasks array, each with `station_tier`,
`file_ownership`, dependencies; a `squadron` section comes later, at
FORMATION).

**Means:** approved Effects have been turned into concrete task
assignments with owners-to-be, file ownership, and risk tiers.

**Exit criteria:** `battle-plan.json` must exist with at least one task,
and every task must have a `station_tier` assigned. This is the exact
criterion the stale mission (§2.2 below) never satisfies — it has
`battle-plan.md` but no `battle-plan.json` on disk at all.

### FORMATION

**Updates:** `battle-plan.json` gains a `squadron` section (admiral,
captains, their ship classes/models, red-cell if any, execution mode).
`fleet-status.json` gets its first real squadron entries.

**Means:** the squadron is formed — who's doing which task, under which
execution mode (`single-session | subagents | agent-team | workflow |
hybrid-workflow`).

**Exit criteria:** `battle-plan.json.squadron` must exist and have an
admiral assigned.

### PERMISSION

**Updates:** `mission-log.json` gains a `permission_granted` event.

**Means:** the plan was presented and the user approved committing
resources to it — the step between planning and spending tokens.

**Exit criteria:** a `permission_granted` event must exist in
`mission-log.json`.

### UNDERWAY

**Updates, continuously:** `mission-log.json` (append-only event stream —
`task_started`, `checkpoint`, `task_completed`, `blocker_raised`, and so
on) and `fleet-status.json` (overwritten at each checkpoint with current
squadron/progress/budget state — see its `last_updated` field). Prose
`quarterdeck-report.md` is updated at every checkpoint (per the
Write Timing table in `structured-data.md`) — **this mission has not
reached a checkpoint yet, so `quarterdeck-report.md` does not exist in
`3d83347e/` at the time of writing.** Per-ship damage reports land in
`damage-reports/{ship}.json` if hull integrity concerns arise; turnover
briefs land in `turnover-briefs/{ship}-{timestamp}.json`/`.md` on relief.
Neither has happened in this mission — both directories exist and are
empty.

**Means:** execution is actually running — tasks moving from pending to
in-progress to completed, checkpoints tracking budget and hull integrity.

**Exit criteria:** every task in `battle-plan.json` has a matching
`task_completed` event in `mission-log.json` (or a `mission_complete`
event is present, covering an early/aborted stop).

### STAND_DOWN

**Brings into existence:** `stand-down.json` (auto-computed duration,
budget, ship/relief counts, violation counts, adopt/avoid patterns) and the
prose `captains-log.md` (final report: delivered artifacts, decisions,
validation evidence, follow-ups — ranked improvement backlog lives here).
Also updates the cross-mission memory store under `.nelson/memory/`
(`patterns.json`, `standing-order-stats.json`), outside any single mission
directory and created automatically on first use — **neither the
`.nelson/memory/` directory nor either file exists anywhere in this repo
yet**, since no mission here has reached STAND_DOWN.

**Means:** terminal phase — the mission is closed out and its outcome is
recorded for future missions' intelligence briefs.

**Neither `stand-down.json` nor `captains-log.md` exists yet in
`3d83347e/` — both are produced at STAND_DOWN, a phase this mission hasn't
reached.** `quarterdeck-report.md` is likewise not yet written (produced
at the first UNDERWAY checkpoint, which also hasn't happened). Don't take
their absence as a problem; take it as confirmation of which phase you're
reading.

### Today's closest thing to a dashboard

`nelson-data.py status --mission-dir <dir>` (or no `--mission-dir`, which
auto-picks the most-recently-modified directory under
`.nelson/missions/`) is the closest thing Nelson has to a dashboard today.
Run against this mission:

```
[nelson-data] Status: underway (checkpoint 0)
Fleet: 0/1 done | Budget: 0.0% (0 tokens) | Hull: 1G 0A 0R 0C | Blockers: 0
```

That's genuinely useful — phase, progress, budget, hull, blockers in one
line, read straight from `fleet-status.json` and `mission-log.json`. What
it does **not** cover, plainly: it reports on **one mission at a time**
(whichever directory you point it at, or whichever sorts newest by mtime).
There is no cross-mission view — no way to ask "which of my missions are
stuck," no equivalent for the stale mission below short of knowing its
path already. `recover`, `brief`, and `analytics` (also in
`nelson-data.py`) each read across missions for a narrower purpose
(session resumption, pre-mission intelligence, win-rate analytics
respectively) but none of them render "what phase is every mission
currently sitting in," which is exactly the gap a maintainer manually
walking `.nelson/missions/` to find the stale mission below had to fill by
hand. That gap is the honest justification for the backlog item the
captain's log will carry (not written here — Effect 3 is the admiral's own
work at stand-down, not this document's).

## 2. Worked example: reading two missions cold

### 2.1 This mission (`3d83347e`) — the healthy case

Reading `.nelson/missions/2026-10-02_181703_3d83347e/` top to bottom, in
the order a reader would actually encounter the evidence:

**`fleet-status.json` → `mission.phase`:** `"UNDERWAY"`. Confirmed by
running the phase engine directly: `nelson-phase.py current --mission-dir
.nelson/missions/2026-10-02_181703_3d83347e/` prints `UNDERWAY`.

**`mission-log.json`'s event stream** tells the same story as a timeline.
Its `phase_transition` events, in order:

```
SAILING_ORDERS -> ESTIMATE   (18:17:24Z)
ESTIMATE       -> BATTLE_PLAN (21:16:27Z)
```

— followed by a `squadron_formed` event (`captain_count: 1, has_red_cell:
false, execution_mode: "subagents"`), a `battle_plan_approved` event
(`task_count: 1, parallel_tracks: 1, critical_path_length: 1`), then two
more transitions, `BATTLE_PLAN -> FORMATION` and `FORMATION -> PERMISSION`,
a `permission_granted` event, and finally `PERMISSION -> UNDERWAY`. Eight
events total, no gaps — a reader can reconstruct the entire mission
history from this file alone without reading any prose.

**`fleet-status.json`'s squadron section**, at the moment captured:

```json
"squadron": [
  {
    "ship_name": "HMS Spey",
    "ship_class": "patrol vessel",
    "role": "captain",
    "hull_integrity_pct": 100,
    "hull_integrity_status": "Green",
    "task_id": 1,
    "task_status": "pending"
  }
]
```

One captain, one task, hull Green, `task_status: "pending"` — matching
`progress: {pending: 1, completed: 0, total: 1}` in the same file. This is
what "a squadron has formed but no ship has started work yet" looks like
on disk.

**`battle-plan.md` and `battle-plan.json` at BATTLE_PLAN/FORMATION:**
`battle-plan.json` names exactly one task (`id: 1`, owner `HMS Spey`,
`station_tier: 0`, `file_ownership: ["DASHBOARD.md"]`) and a `squadron`
section with admiral `HMS Victory` and one captain, mode `subagents` — the
two pieces of evidence `nelson-phase.py`'s BATTLE_PLAN and FORMATION exit
criteria each check for. `battle-plan.md` carries the same task as
readable prose, plus the Standing Order Check (twelve orders considered
line by line, none triggered — matching the `triggered: []` the
structured `squadron_formed`/`battle_plan_approved` events also record)
that doesn't have a structured-JSON line-by-line equivalent of its own.

**What isn't here yet, and why that's correct, not broken:**
`quarterdeck-report.md` and `captains-log.md` don't exist in this mission's
directory. Per §1 above, the first is written at the first UNDERWAY
checkpoint and the second at STAND_DOWN — this mission has only just
entered UNDERWAY (one `recent_events` entry: `"Squadron formed: 1
captains"`) and hasn't reached either milestone. `damage-reports/` and
`turnover-briefs/` exist as directories but are empty — no ship has had a
hull incident or been relieved. Their absence is exactly what a mission at
this specific point in UNDERWAY should show.

### 2.2 The stale mission (`761d8f37`) — the cautionary case

`.nelson/missions/2026-07-06_223339_761d8f37/` is a second, unrelated
mission (outcome: external-repository-target support for Nelson) that was
never finished. Reading it the same way:

**Phase:** `fleet-status.json` → `mission.phase` is `"BATTLE_PLAN"`
(`mission.status: "forming"`), confirmed directly: `nelson-phase.py
current --mission-dir
.nelson/missions/2026-07-06_223339_761d8f37/` prints `BATTLE_PLAN`.

**What's present:** `sailing-orders.json` (with its own `amendments` entry,
version 2, reframing the target mechanism), `estimate.md`,
`estimate-briefing-1.md`, `estimate-briefing-2.md`, and `battle-plan.md`
(10,702 bytes of drafted prose — commander's intent and task breakdown
written out). `mission-log.json` has three events: the two expected
`phase_transition`s (`SAILING_ORDERS -> ESTIMATE`, `ESTIMATE ->
BATTLE_PLAN`), and then a `battle_plan_amended` event
(`"Sailing orders amendment v2 added; original v1 preserved."`) — a note
about the sailing orders, not a battle plan being finalized.

**What's missing:** no `battle-plan.json` anywhere in the directory. Per
§1's BATTLE_PLAN exit criteria, that file — with at least one task, each
carrying a `station_tier` — is exactly what's required to advance to
FORMATION. Without it, there is also no `squadron_formed` or
`battle_plan_approved` event in `mission-log.json`, and `fleet-status.json`
shows `squadron: []`, `progress: {total: 0}` — nobody was ever assigned to
the plan that was drafted. `recent_events` reads only `"Mission
initialized"`, frozen since `last_updated: "2026-07-06T23:52:52Z"`.

**The live marker:** `.nelson/.active-761d8f37` still exists at
`/home/skogix/nelson/.nelson/.active-761d8f37`, pointing at this mission
directory — the session-marker file that normally tells recovery tooling
and hooks which mission is "active." It has not been cleaned up in the
roughly three months since. (This mission's own marker,
`.nelson/.active-3d83347e`, coexists alongside it — `nelson-phase.py`'s
auto-discovery picks whichever `.active-*` file has the newest mtime, so
two live markers don't silently collide, but their simultaneous presence
is itself a visible symptom a reader can spot by listing `.nelson/`.)

**What a healthy mission would show instead, at BATTLE_PLAN:** a
`battle-plan.json` with at least one task carrying a `station_tier`
(§1 BATTLE_PLAN), shortly followed by a `squadron_formed` event and a
`squadron` section in `battle-plan.json` (§1 FORMATION) — the same shape
`3d83347e/` reached within about a minute, per its own timestamps. No
remediation is proposed here for `761d8f37/` — this passage is descriptive
only, per the mission's standing orders; path and phase are the whole
report.

### 2.3 Reading any mission directory cold

Put together, the read-cold procedure is:

1. Open `fleet-status.json`, read `mission.phase` (or run `nelson-phase.py
   current --mission-dir <dir>`, which reads the same field).
2. Check which artifacts from §1's list for that phase, and every phase
   before it, are present. All of them should be; a gap at an
   *earlier* phase than the reported one is the signature of a stuck or
   corrupted mission.
3. Check which artifacts from *later* phases are absent — they should be,
   and that absence is informative, not alarming, as long as it matches
   the reported phase.
4. Read `mission-log.json`'s event stream for the "how did we get here"
   narrative; read `fleet-status.json`'s `squadron`/`progress`/`budget`
   for the "where exactly are we right now" snapshot.
5. Check `.nelson/.active-*` for stray markers pointing at directories
   whose `last_updated` is old and whose phase hasn't advanced — that
   combination (stale mtime + live marker + no forward progress) is what
   an abandoned mission looks like, as demonstrated by `761d8f37/` above.

A reader who has never seen Nelson before, handed a mission directory
containing `sailing-orders.json`, `estimate.md`, `battle-plan.md`, and
`battle-plan.json` with tasks and station tiers, but no `squadron` section
in `battle-plan.json` and no further events past
`battle_plan_approved`... should now be able to say: this mission is at
BATTLE_PLAN, approved but not yet formed into a squadron — the same
diagnosis `761d8f37/` warrants, one step earlier, since in its case even
`battle-plan.json` itself never appeared.
