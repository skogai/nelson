# Estimate Briefing 2 - Terrain, Forces, Coordination, Control

Mission directory: `.nelson/missions/2026-07-06_223339_761d8f37`

Read `.nelson/missions/2026-07-06_223339_761d8f37/estimate.md`.

Task: append sections `## 4. Terrain`, `## 5. Forces`, `## 6. Coordination`, and `## 7. Control` to the estimate.

Required voice: concise officer's briefing. This is execution planning, not implementation.

Approved mission framing:
- Nelson keeps all mission state in one control repo under `.nelson/missions/...`.
- Ships may work in other repositories via target worktrees.
- Default target input is an already-existing local source repository.
- Preferred provider is `/home/skogix/.cargo/bin/wt`.
- Creation command shape: `/home/skogix/.cargo/bin/wt -C <source_repo> switch --create nelson/<mission-id> --no-cd`.
- `wt` creates the target folder and runs user lifecycle scripts.
- Nelson must capture or record the actual `working_dir`; do not assume one fixed path.
- `wt list`, `wt step diff`, and `wt remove <branch>` cover discovery, diff, and cleanup.
- Branch naming: `nelson/<mission-id>`.
- Multi-repo missions are explicit; use one ship per repo as the normal rule.

Planning requirements:
- Q4 Terrain should map each effect to concrete files/modules likely touched.
- Q5 Forces should recommend mode, captains, crew, and red-cell need.
- Q6 Coordination should state dependencies and parallel tracks.
- Q7 Control should name quality gates, intervention points, action-station tiers, and rollback/cleanup.

Do not implement code. Do not change files outside this mission estimate/briefing area.
