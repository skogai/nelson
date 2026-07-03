# Crew Suggestions — Skill Improvement Proposals

> **Provenance:** Salvaged from the SkogBackup/nelson experiments. These
> proposals were written against the older **six-step** Nelson SKILL.md.
> The current skill is **eight-step** (Sailing Orders, The Estimate, Battle
> Plan, Form the Squadron, Get Permission to Sail, Quarterdeck Rhythm,
> Action Stations, Stand Down). The *core idea* — persona-voice framing over
> rule-lists — still ports cleanly; the specific step references below do
> not map one-to-one. Treat this as a design memo, not a drop-in patch.

Origin: Session reviewing Nelson SKILL.md, prompted by discussion of why persona-casting works better than rule-lists for LLM agents.

Core insight driving all proposals: LLMs embody characters better than they follow instructions. The epithet + "thinks like" pattern in the Fleet Personnel Index already does this perfectly. These proposals extend that logic into parts of SKILL.md that are still written as rules.

---

## 1. Admiralty Doctrine → Vane's Voice

**Current:** Six bullet points about coordination behavior.

**Proposal:** Rewrite as Admiral Vane speaking. Vane asks "what are the 2nd-order consequences?" and "what would make me reverse course?" — those two questions already contain the whole doctrine. A section written in Vane's voice would activate the same character associations that make the persona table so effective, without adding any new instructions.

**Why:** The doctrine section is the one place the skill tells the admiral *how to think*. Vane is already defined as the person who does that. Let Vane do it.

---

## 2. Steps Framed Around Character Modes

**Current:** Six procedural steps, consistent neutral tone throughout.

**Proposal:** Each step is naturally owned by a different character mode. Battle Plan is Rook (one-page clarity). Quarterdeck Rhythm is Holt (name the failure point, proceed consistently). Action Stations is Vaas (list every assertion made without evidence). A single framing line at the top of each step — "Rook's test: does the plan fit one page?" — would prime the right thinking mode before the procedural bullets.

**Why:** The steps are already good. This is a light-touch addition, not a rewrite. The character line does the behavioral work so the bullets can stay lean.

---

## 3. Crew Briefing as Holt's Handoff

**Current:** "Use template X, include these fields, target 500 tokens."

**Proposal:** "Write this as Holt handing off to a junior captain: name the most likely failure point aloud, then give them everything they need to proceed without asking questions." The template stays in the reference file. The instruction in SKILL.md becomes one sentence of character framing.

**Why:** The template already has the right fields. The problem is the *quality* of briefings, not the structure. Holt's framing ("name the failure point aloud") is a behavioral instruction that no field checklist can replicate.

---

## 4. Fleet Memory Integration Note

**Current:** Nelson manages task state via TaskCreate/TaskUpdate. No explicit memory architecture.

**Proposal:** Add a short pointer in the Battle Plan step noting that missions with parallel agents and persistent state should consider the fleet-memory skill for scope isolation and checkpoint management. Not mandatory, not procedural — just "if your mission needs this, there's a skill for it."

**Why:** fleet-memory exists at `.skogai/experiments/fleet-memory` and solves the exact coordination problem that emerges when Nelson missions get complex. The two skills are complementary but currently unaware of each other.

---

## Status

All four are proposals only. None applied to SKILL.md. Revisit when ready to iterate.
