# Salvaged experiments

Material recovered from `SkogBackup/nelson` (a March-2026 snapshot forked
from an older upstream Nelson) that is **not present** in the current
skill and was judged worth preserving. Everything else in that backup was
either a duplicate or already superseded by the current version — the
`royal-marines`, `commendations`, `red-cell-review`, and
`marine-deployment-brief` concepts, for example, already shipped.

These files are **incubating**, not wired into the shipped `nelson` skill.
Nothing here is loaded at runtime. Promote deliberately.

## Contents

### `fleet-memory/` — standalone skill (v1.0.0)

Multi-agent memory coordination for filesystem-based agent systems: memory
tiers (Policy / State / Entity / Knowledge), scope isolation, the
single-writer rule, checkpoint-based state, and consolidation. Self-contained
(`SKILL.md` + five references). Complements Nelson's coordination model —
Nelson orchestrates the work, fleet-memory governs the shared state.

**To promote:** move to `skills/fleet-memory/` and register it in the plugin
manifest, or publish as its own plugin. Note the current repo already has
in-flight branches (`feat/cross-mission-memory`,
`feat/fleet-intelligence-cross-mission-memory`) — reconcile against those
before promoting so the two efforts don't diverge.

### `personas/` — Fleet Personnel Index

A character-voice layer the current eight-step SKILL.md has no trace of, built
on the observation that LLMs embody named characters more reliably than they
follow rule-lists.

- `PERSONAS.md` — full officer profiles (Vane, Holt, Rook, Vaas, …).
- `personas-index.md` — the load-during-planning quick index (epithet +
  "thinks like…" heuristic per role).

**To promote (non-invasive):** keep as reference docs and have the SKILL point
at them during planning. **To promote (invasive):** wire the index into the
planning steps so the personas are actually loaded — a deliberate edit to the
live SKILL.md, best done as its own reviewed change.

### `crew-suggestions.md` — design memo

Proposals for wiring the personas into the skill's voice. Written against the
older six-step SKILL; the ideas port, the step references do not. See the
provenance note at the top of the file.
