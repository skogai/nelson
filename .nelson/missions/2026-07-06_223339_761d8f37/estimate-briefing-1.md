# Estimate Briefing 1 - Intent and Effects

Mission directory: `.nelson/missions/2026-07-06_223339_761d8f37`

Read `.nelson/missions/2026-07-06_223339_761d8f37/sailing-orders.json` and `.nelson/missions/2026-07-06_223339_761d8f37/estimate.md`.

Task: append sections `## 2. Intent` and `## 3. Effects` to the estimate.

Required voice: concise officer's briefing. The output should be clear enough to travel verbatim into captain briefs.

User clarifications to honor:
- Mission state should remain in one Nelson control repository.
- The important target input is an already-existing local source repository path from which Nelson can create a worktree.
- Use the user's `wt` provider when available, not raw `git worktree`, because it triggers user lifecycle scripts.
- If the source repository does not exist, do not silently fall back to clone for this default path.
- Default Nelson-created branch name: `nelson/<mission-id>`, where `<mission-id>` is the existing Nelson session id.
- Default to one primary target repository per mission. Multi-repo missions are explicit.
- "One ship per repo" is a useful rule for multi-repo missions, so each captain has one target working root.

Effects must include commander's guidance and acceptance criteria. Cover at least:
- target repository metadata/schema,
- worktree provider resolution and creation contract,
- hook/runtime lookup across separated control and target roots,
- captain/brief/template propagation,
- tests and docs.

Do not implement code. Do not change files outside the estimate.
