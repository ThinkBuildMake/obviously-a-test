# USERS_handoff — Brave Frontier vertical slice in Godot 4

> **Status:** Stage 1 companion to [`USERS.md`](USERS.md). Holds everything the users pass surfaced that does not belong in a users document: the design content that was cut so the users doc stays about users, verification needs traced to use case IDs, scope boundaries with their design reasons, and the open questions the next stages must carry. Split rule applied throughout: constraints that were **given** (by the brief or the spec's locked decisions) live in USERS.md §Constraints; conclusions that were **chosen or derived** in the spec and this pass live here.

## A. Verification needs, traced to use cases

These arrive as acceptance criteria for stage 2 (requirements) and the key metrics doc. Each row is the user-visible behavior the check exists to protect.

| # | Verification need | UC | What the check protects |
|---|---|---|---|
| V1 | Element chart is a data table tested for the full cycle (Fire→Earth→Thunder→Water→Fire; Light⇄Dark both ways) with ×1.5/×0.5 lookups verified and no unbeatable element | UC-1 | Damage numbers meaning what they say |
| V2 | Damage math verified in isolation and in composition under a fixed seed with variance disabled: advantage, resist, spark, crit, guard, buff multipliers each checked; a minimum-damage floor asserted | UC-1, UC-2 | The signature multipliers staying real |
| V3 | BC economy end-to-end: per-hit drop rolls honored under seed, per-attack cap respected, phase-end distribution fills the lowest gauge first, a burst spends and resets the full gauge | UC-2 | The hold-or-spend decision existing at all |
| V4 | A full scripted battle driven from setup to victory replays identically under the same seed (state-hash equality, run twice) | UC-5 | "We didn't break the game" being a checked claim |
| V5 | Quest-loop integration: squad-select → battle → results grants the stage's EXP/Zel/Karma; defeat grants nothing; entry spends energy; energy regenerates against an injected clock (simulate 60 minutes → +N) | UC-1 | The loop closing and paying out |
| V6 | Save round-trip: save → load → identical state; a prior-version save migrates without crashing; a corrupt file yields a fresh state with a logged warning | UC-4 | The squad and progress surviving sessions |
| V7 | The suite runs headless in CI on every push, and merge requires green; local runs use the same command | UC-5 | Verification happening at all |
| V8 | Human playtest (not automatable): one full quest from the menu, judged on whether the loop feels like Brave Frontier's | UC-1 | The demo gate |

## B. Access-control needs

None in the slice. It is single-player with local files and no accounts, network services, or shared state (UC-1, UC-4). Save *integrity* (V6) is the closest need and is a data-integrity concern, not access control. If accounts ever arrive (out of scope in USERS.md), that is a new use case and a new stage, not an extension of these.

## C. Design content cut from the users doc

Chosen in the spec and confirmed nowhere else. If a stage disagrees with one of these, it changes the design, not the users doc.

- **Engine and framework lock:** Godot 4.7.x stable, GDScript (not C#) → stack stage.
- **Pure-logic battle engine:** `BattleEngine` as a scene-independent state machine exposing signals and consuming plain command objects; seeded RNG throughout → architecture stage.
- **Damage constants:** ×1.5 element advantage, ×0.5 resist, ×1.5 spark, ×1.5 crit, ×0.5 guard, ±10% variance (disabled in tests), minimum damage 1 → requirements numbers, architecture constants.
- **Our-rule choices pending the playtest (the spec flags each as design, not recovered original rules):** BC distribution fills the lowest gauge first; spark = same-target-same-phase; REC drives turn-end regeneration; Spirit Ward as the third battle item → requirements stage should carry each as tunable data, not architecture.
- **Persistence shape:** versioned JSON saves in `user://`, crash-tolerant writes (temp file, then rename) → architecture stage.
- **Energy realism:** real-time regeneration via an injected clock so tests can simulate elapsed time → architecture and requirements NFRs.
- **Content set:** 8 original units across all 6 elements (one healer, one tank, one support-burst), area "Elrune Grove," 3 stages × 2 waves, 3 enemy types, 3 battle items → requirements content list.
- **Platform details:** 1280×720 desktop window; headless gdUnit4 in GitHub Actions with a pinned Godot binary → requirements NFRs and CI story.
- **Squad-size rule:** up to 6 units, leader skill applies, friend-guest simplified away (USERS.md lists friend-guest as out of scope) → requirements.

## D. Scope boundaries with their design reasons

- **KO'd units are revive-only.** Healing them back would erase UC-3's triage moment; the boundary is the point (UC-3).
- **Retreat counts as defeat:** entry energy is spent, the stage stays uncleared, nothing is granted. Keeps defeat meaningful without punishing experimentation twice (UC-1).
- **BB only — no SBB/UBB.** A smaller tuning surface is what lets the core be judged before layering gauge economies (UC-2).
- **All 8 units owned from the start.** Acquisition would change the product, not add content (UC-4).
- **Defeat economy.** A lost battle costs the entry energy and nothing else — no partial rewards (UC-1).

## E. Time budgets observed in the workflow walk

- Quest loop, menu to results: 10–20 minutes (UC-1).
- Squad select: ~2 minutes; leader pick inside it (UC-4).
- Per-turn ordering: seconds; the burst/recovery decisions are seconds-long but carry the fight (UC-2, UC-3).
- Energy regen rate: **unspecified** — the spec says real-time regen but sets no rate. Stage 2 must pick a number; it is the loop's pacing dial.

## F. Open questions for the next stages

- **Stage 2 (requirements) must resolve:** the energy regen rate (E above); which of the "our-rule" constants get asserted as fixed requirements vs. shipped as tunable data; NFR thresholds for UI responsiveness (the users doc says seconds-per-turn in user terms — requirements must make it a number with a verification method).
- **Web export:** default out (USERS.md, later iterations). Revisit only after the loop is stable.
- **Audio:** default minimal (menu/attack feedback). A full music pipeline needs an audio story and asset budget.
- **Headless replay log format:** default plain state hash; human-readable replays only if asked for.
- **Licensing/intended use:** open question 3 in USERS.md — the asset license choice downstream depends on it.

## G. What stage 2 must inherit

1. **Use case IDs are fixed** — UC-1 through UC-5 as written in USERS.md; requirements and metrics cite them, never renumber them.
2. **The given/chosen split** (§C header): given constraints stay in USERS.md; everything here is chosen and negotiable by the downstream stages, with the spec's locked decisions as the floor.
3. **The demo gate is non-automatable** (V8) — every other check in §A must be mechanized; this one cannot be, and requirements should not pretend otherwise.
4. **The verification table in the spec** is the source every §A row traces back to; when requirements quantifies them, keep the traceability (check → UC → spec row).
5. **Evidence limitations transfer**: nothing upstream proves demand; stage 2 justifies scope by the use cases, not by a market.
