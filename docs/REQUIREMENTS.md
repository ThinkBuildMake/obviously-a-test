# REQUIREMENTS.md — Brave Frontier vertical slice in Godot 4

**Status:** draft — stage 2 of the documentation pipeline (versioned when handed to key metrics / architecture).
**Purpose:** turn the stage-1 users document and the committed spec into requirements the next stages can be *chosen against*: what the user does and expects (functional), what the system has to accommodate, as numbers with verification methods (non-functional), and the constraints the project arrived with (givens). This document does not design the system.
**Companion files:** [`USERS.md`](USERS.md) and [`USERS_handoff.md`](USERS_handoff.md) (stage 1) · spec *Brave Frontier vertical slice in Godot 4* (Blueprint art_zmUVyeNW — cited as "spec").

## Scope

**This document decides:** the functional requirements for the minimum product (one group per use case), the non-functional requirements with numbers and verification methods, the given constraints as requirement rows, the resolution of the three gaps stage 2 owns (energy regen rate, fixed-vs-tunable our-rule constants, seconds-per-turn made numeric), and the traceability from every check back through a use case to its spec row.

**It deliberately defers:** component sizing, capacity, and techniques for meeting a budget → high-level design (stage 4); technology choices and tradeoffs → stack (stage 5); modules, schemas, engine contracts, and data flow → architecture (stage 6); which requirements become tracked metrics and dashboards → key metrics (stage 3); the problem's "why" and cut line → USERS.md; flows and personas → USERS.md. Functional rows cover the minimum product only; later iterations (USERS.md §Later iterations) get rows when they enter a build.

## How to read a requirement

- **Tags.** `[src …]` = ground truth from an upstream document, cited to its section. `[prop …]` = a proposed value from this pass — justified inline, collected in §5, open for the playtest to tune. `[tbd …]` = cannot be set until a measurement exists; none survived drafting (see §5).
- **Priorities.** **Gate** = must hold before the demo is shown as "the slice" — it protects the IP boundary, player data, or the claim the whole verification strategy rests on. **Must** = required for the product to be done. **Should** = required unless a documented tradeoff says otherwise. Priority is about need, not the sprint: this document never cuts scope for a demo; if a Gate or Must would be deferred, the story map records when it lands.
- **Trace format.** `UC-x · V# · spec <row>` — use case in USERS.md, verification need in USERS_handoff §A, and the spec verification/mechanics row every check traces back to (handoff §G4).
- **Functional rows** (§2) have four columns — they carry no threshold; numbers live in §3. **Non-functional and given rows** (§1, §3) have five — the Requirement *is* the number, with its tag.

## Where the numbers come from

Ground truth extracted from the inputs before any threshold was written: quest loop menu-to-results **10–20 minutes** (U§workflow, UC-1); squad select **~2 minutes** (U§workflow, H§E); per-turn ordering **"seconds"** (H§E, UC-2/UC-3); squads of **up to 6 units** from **8 owned** (spec, U§minimum product); **3 stages × 2 waves, 3 enemy types, 3 items** (spec content table); damage constants **×1.5 advantage, ×0.5 resist, ×1.5 spark, ×1.5 crit, ×0.5 guard, ±10% variance, damage floor 1** (spec damage function); **energy regenerates in real time — rate unspecified** (spec quest-loop row, H§E); demo window **1280×720, mouse + keyboard** (U§Constraints); **headless suite in CI on every push, merge requires green** (H§V7). The energy *rate*, the UI timing budgets, and every pacing number below are **not** ground truth — they are this pass's proposals and say so.

## Assumptions

1. **USERS.md stands in for the problem statement.** No separate problem doc exists; its problem-in-numbers, why-these-moments, and out-of-scope sections carry the "why" and the cut line, and its exclusions become the scope-boundary requirements (GC-8).
2. **The users doc's working defaults stand** (slice reading, repo home, Michael as the sole demo gate) — Michael was unavailable for interview in stage 1 and has not answered open questions 1–4 since; an answer against a default rewrites scope, not a detail.
3. **Every combat constant is an original design choice in Brave-Frontier shape, not a recovered original rule** — the original's damage formula was never public (spec evidence limitations). Nothing in this document may cite an "original Brave Frontier value" that this session did not retrieve.

## Definitions

- **Brave Crystal (BC)** — a crystal dropped by a landed hit, pooled during the player phase and distributed to burst gauges at phase end.
- **Burst gauge** — a per-unit meter (0 to full) filled by distributed BC; "full" means the unit's Brave Burst may fire. Spending it empties the gauge.
- **Spark** — the bonus applied when two or more units' attacks land on the same target within the same player phase.
- **Player phase (command phase)** — one turn's span in which every living player unit receives exactly one order, after which the orders resolve in sequence, BC is distributed, and the enemy turn runs.
- **Living unit** — a squad unit not KO'd. **KO'd** — a unit at 0 HP: it cannot act, cannot be healed, and returns only via Revive Light.
- **Wave / stage** — a stage is one quest entry; it plays as sequential waves of enemies. Clearing the last wave wins the stage.
- **Energy** — the resource a stage entry spends, regenerating in real time.
- **State hash** — a digest of the battle engine's full state; identical seeds must produce identical hashes.
- **Injected clock** — a time source handed to the energy system so tests can simulate elapsed time without waiting.
- **The suite** — the automated gdUnit4 test suite, run headless locally and in CI with the identical command.

## 0. Summary

| | |
|---|---|
| **Product** | A playable vertical slice: the Brave-Frontier-shaped battle core (elements, BC → burst gauges, sparks, guard, items, waves) inside a quest → battle → results loop, with squad management and local saves. |
| **Users served** | The returning Brave Frontier player in a 10–20-minute evening gap (UC-1..UC-4), and the engineering team verifying rule changes headless (UC-5). |
| **Requirement counts** | 10 givens (GC), 27 functional (FR), 17 non-functional (VR/RL/PF/DE/OB/EV/CH). 54 rows total. |
| **Stage-2 gaps resolved** | Energy regen: **1 per 3 real minutes** (original design default, tunable data, awaiting the playtest — KD-2). Our-rule constants: **spark detection and BC distribution order asserted as fixed rules; REC turn-end regen and Spirit Ward shipped as tunable data** (KD-3). Seconds-per-turn: **100 ms input acknowledgment (automated), ≤200 ms phase compute (automated), ≤30 s to order a full squad (playtest-judged)** (KD-4). |
| **Non-negotiables** | Zero gumi content (GC-3, Gate); deterministic combat — same seed, identical state hash (VR-4, Gate); a save write can never destroy the previous save (RL-1, Gate); KO'd units return only via revive (FR-Recovery.2, Gate); defeat and retreat grant nothing (FR-Quest.5, Gate). |
| **The demo gate** | The human playtest (V8) is not automatable and is never dressed up as a testable requirement (EV-2); every other verification need V1–V7 is mechanized (EV-1). |
| **For stage 3 (key metrics)** | Inherit: the V1–V7 mechanized checks as the tracked set, V8 as the untracked human gate; the energy rate (+20/hour) and the PF budgets as the pacing numbers to watch; the fixed/tunable split so dashboards watch data, not constants in code. |

## 0a. Key decisions

| # | Decision | Alternative rejected | Governs |
|---|---|---|---|
| KD-1 | Scope is the vertical slice, per the spec's locked decision and USERS.md open question 1's default | Battle-only demo (no loop to hang later systems on) or full recreation (months of content, drifts toward gumi's IP) | All FR-Content, GC-8 |
| KD-2 | Energy regenerates **1 per 3 real minutes**; max 50 and a flat 10 stage entry ship as tunable data. Pacing rationale: at 1/3 min, one spent battle's cost (10) returns in ~30 minutes and a drained bar in 2.5 hours — "one more battle later tonight" stays possible, while back-to-back stage runs are capped and a full evening session (10–20 min, one or two battles) is never blocked. 1/5 min starves evening replays harder than UC-1's moment needs; 1/min erases the dial. **Original design default awaiting the playtest — never a recovered Brave Frontier rule.** | 1 per 5 min (slower; full refill 4h10m) or 1 per min (no visible dial) | FR-Quest.1, VR-5, CH-1 |
| KD-3 | Of the four our-rule constants: **spark = same-target-same-phase** and **BC distribution fills the lowest gauge first** are asserted as *fixed rules* — they are the slice's combat identity, they are what UC-2's decision and the V2/V3 tests name, and their *shape* changes only with a spec revision. **REC-driven turn-end regen** and **Spirit Ward** ship as *tunable data* — the playtest may retune or drop them without a spec revision. All four live as data values either way. | Asserting all four as fixed (freezes pacing choices the playtest exists to judge) or all four as tunable (lets the spark/BC identity silently change) | FR-Burst.1, FR-Quest.8, FR-Recovery.1, VR-3, CH-1 |
| KD-4 | "Seconds per turn" (H§E) becomes three numbers: per-order input acknowledgment **≤ 100 ms** and headless phase compute **≤ 200 ms** (both automated), and a full six-unit command phase orderable in **≤ 30 s** (judged in the human playtest — pacing is feel, not a clock) | One blanket "the UI feels fast" claim (unfalsifiable) | PF-1, PF-2, PF-3 |
| KD-5 | V8 stays non-automatable: every other verification need (V1–V7) is mechanized in the suite; the demo gate is a recorded human verdict, never a test | Writing the playtest as an automatable check (pretends the one thing that can't be verified was) | EV-1, EV-2, GC-10 |
| KD-6 | No scalability requirements: single-player, local, desktop — no quality-of-scale changes what gets built | Boilerplate load rows that would make the architect build for nobody | (absence) |

## 1. Givens: constraints

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| GC-1 | The game is built with Godot 4.7.x stable and GDScript, not C# [src U§Constraints; spec locked decisions]. | CI builds against the pinned Godot binary; the project's engine version is checked in the pipeline. | spec stack · H§C | Must |
| GC-2 | The demo target is a desktop window at 1280×720 with mouse and keyboard; no mobile or controller layout ships in v1 [src U§Constraints]. | Window size asserted in project settings; observed at the playtest. | U§Constraints · spec platform row | Must |
| GC-3 | Zero gumi content: no ripped art, no original character names, no story text, no in-game "Brave Frontier" branding anywhere in the repo or the build [src U§Constraints; spec locked decisions]. | Content review at the playtest against an original-name checklist — manual by nature; automated IP detection is not attempted and none is claimed. | U§Out of scope · spec locked decisions | Gate |
| GC-4 | English only; no localization ships in v1 [src U§Constraints]. | Build inspection at the playtest. | U§Constraints | Should |
| GC-5 | The team is agent builders with Michael Chuang as reviewer and the sole demo gate [src U§Constraints]. | A playtest record with a written verdict exists before the demo is called passed (EV-2). | U§Constraints · U§OQ2 | Must |
| GC-6 | No deadline and no spend cap were given; build order is milestone-sequenced, and if work is half done, content breadth drops while the loop and battle rules hold [src U§Constraints]. | The stage-4 build plan records the ordering against this row. | U§Constraints | Must |
| GC-7 | Intended use is a free, public project; nothing is sold or monetized until stated otherwise [src U§OQ3 working default]. | Repo license/readme review at handoff. | U§OQ3 | Should |
| GC-8 | Out-of-scope features stay out of v1: gacha or any acquisition loop, cloud saves or accounts, monetization, analytics, localization, the friend-guest system [src U§Out of scope]. | Feature-absence check at the playtest demo. | U§Out of scope · spec scope table | Must |
| GC-9 | Use case IDs UC-1 through UC-5 are fixed and never renumbered; this document and every downstream doc cite them as written [src H§G1]. | Traceability-matrix review (§4). | H§G1 | Must |
| GC-10 | The demo gate is the human playtest and nothing may stand in for it; every automated check covers a different need [src H§G3]. | EV-2's record review. | H§G3 · spec playtest row | Must |

## 2. Functional requirements (minimum product only)

### FR-Quest — run one stage, from menu to results (UC-1)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-Quest.1 | From the main menu the player opens the quest map, sees Elrune Grove's three stages with their entry costs against the current energy, and enters a stage whose cost the energy covers; entering spends the cost and starts wave 1. | UC-1 · V5 · spec quest-loop row | Must |
| FR-Quest.2 | A stage plays as sequential enemy waves — two per stage; clearing a wave starts the next, and clearing the last ends the battle in victory. | UC-1 · V4 · spec turn-structure row | Must |
| FR-Quest.3 | Each turn the player gives every living unit exactly one order — Attack, Brave Burst, Guard, or Item — and the phase cannot resolve while any living unit lacks one; the engine refuses to resolve an incomplete set. | UC-1 · spec empty-command path | Must |
| FR-Quest.4 | On victory the results screen grants the stage's unit EXP, Zel, and Karma; EXP raises unit levels, and levels raise stats. | UC-1 · V5 · spec quest-loop row | Must |
| FR-Quest.5 | Defeat and retreat grant no rewards and leave the stage uncleared; the entry energy stays spent. Retreat is offered from the pause menu and counts as a defeat. | UC-1 · spec defeat economy | Gate |
| FR-Quest.6 | The player can always reach the menu from the map, the battle, and the results; no screen strands them. | UC-1 · U§UC-1 must-never | Must |
| FR-Quest.7 | Damage numbers reflect the six-element chart — advantage ×1.5, resist ×0.5, Light⇄Dark strong both ways — so a counter-picked squad visibly hits harder, and no element is unbeatable. | UC-1, UC-2 · V1, V2 · spec elements row | Must |
| FR-Quest.8 | Living units regenerate a small amount of HP at the end of each turn, driven by their REC stat; the amount is tunable data (CH-1). | UC-1 · spec stats row (REC→regen is ours) | Should |
| FR-Quest.9 | The player can order Guard, and a guarding unit takes half damage during the coming enemy phase. | UC-1, UC-3 · spec guard row | Must |
| FR-Quest.10 | Each unit's attacks can critically hit, multiplying that hit's damage ×1.5 independently of element and spark, at the unit's own crit chance. | UC-1, UC-2 · spec crit row | Must |

### FR-Burst — time the burst (UC-2)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-Burst.1 | Landed hits drop Brave Crystals — a per-hit chance, capped per attack — and at the end of the player phase the pool is distributed to burst gauges, lowest gauge first. | UC-2 · V3 · spec BC row | Must |
| FR-Burst.2 | A unit may fire its Brave Burst only when its gauge is full, and firing empties the gauge; a short gauge can never fire and a full gauge never silently resets. | UC-2 · V3 · spec BB row | Must |
| FR-Burst.3 | A Brave Burst is an attack burst, a heal, or a support buff, and its effect is shown on the unit before the player commits to it. | UC-2 · spec BB row | Must |
| FR-Burst.4 | During the command phase the player can see every unit's gauge fill, and a full gauge is visibly distinct before the phase resolves. | UC-2 · spec battle screen | Must |

### FR-Recovery — recover a battle going wrong (UC-3)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-Recovery.1 | Three battle items with limited counts — Heal Potion (restores HP), Revive Light (revives at 50% HP), Spirit Ward (DEF buff; tunable data, CH-1) — and using one is that unit's action for the turn. | UC-3 · spec items row | Must |
| FR-Recovery.2 | A KO'd unit cannot act and cannot be healed back; it stays visible in the squad list, and only a Revive Light returns it to the fight. | UC-3 · spec KO path | Gate |
| FR-Recovery.3 | The player sees remaining item counts during battle, and a depleted item cannot be chosen. | UC-3 · spec battle screen | Must |
| FR-Recovery.4 | When every player unit is KO'd, the battle ends immediately in defeat. | UC-3 · spec wipe path | Must |

### FR-Squad — set the squad, pick the leader (UC-4)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-Squad.1 | Squad select lets the player commit any 6 of the 8 owned units and pick one leader; the leader's skill applies to all six squad units and to nothing else. | UC-4 · spec squad row | Must |
| FR-Squad.2 | The squad, leader pick, unit levels and EXP, cleared stages, currency balances, and energy persist across sessions in a local save. | UC-4 · V6 · spec saves row | Must |
| FR-Squad.3 | The select screen shows each unit's element and role (healer, tank, support-burst) and what the leader skill grants the squad. | UC-4 · spec squad row | Must |
| FR-Squad.4 | All 8 units are owned from the start; no screen implies acquisition is possible or needed. | UC-4 · U§UC-4 must-never | Must |

### FR-Verify — verify a combat-rules change (UC-5, developer use case)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-Verify.1 | The test suite runs headless in CI on every push, a merge requires green, and local runs use the identical command. | UC-5 · V7 · spec CI row | Must |
| FR-Verify.2 | When a check fails, the failure names the broken rule area — element chart, damage math, BC economy, or the replayed battle's final state. | UC-5 · V7 · spec verification table | Must |
| FR-Verify.3 | A scripted battle driven from setup to victory under a fixed seed is a committed test that asserts the final state hash across two runs. | UC-5 · V4 · spec replay row | Must |

### FR-Content — the slice's content set

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-Content.1 | Eight original units cover all six elements and include at least one healer, one tank, and one support-burst. | UC-4 · spec content table | Must |
| FR-Content.2 | Elrune Grove ships as one area of three stages, each of two waves, drawn from three enemy types. | UC-1 · spec content table | Must |

## 3. Non-functional requirements

### VR — verification and correctness

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| VR-1 | The element chart is a data table whose full cycle tests correct — Fire→Earth→Thunder→Water→Fire and Light⇄Dark both ways — with every ×1.5 advantage and ×0.5 resist lookup asserted for all six elements and no element unbeatable [src H§V1]. | Chart unit test covering every directed element pair. | UC-1 → V1 → spec elements row | Must |
| VR-2 | Damage math passes fixed-seed tests with variance disabled that verify advantage, resist, spark, crit, guard, and atk-buff multipliers in isolation and in composition, at the spec's constants (×1.5, ×0.5, ×1.5, ×1.5, ×0.5, ×(1+buff), ±10% when enabled) with a minimum damage floor of 1 [src S§damage function; H§V2]. | DamageCalculator unit tests, one per multiplier plus composition cases. | UC-1, UC-2 → V2 → spec damage row | Must |
| VR-3 | The BC economy passes end-to-end tests: per-hit drop rolls are honored under seed, the per-attack cap is respected, phase-end distribution fills the lowest gauge first, and a burst spends and resets a full gauge [src H§V3]. | BC-rule unit tests driven by seeded RNG. | UC-2 → V3 → spec BC row | Must |
| VR-4 | Identical seeds produce identical battles: a scripted run's final state hash equals a second run's under the same seed, every suite execution [src H§V4]. | The replay test (FR-Verify.3) run twice per suite execution, hashes asserted equal. | UC-5 → V4 → spec replay row | Gate |
| VR-5 | The quest loop passes an integration test: squad-select → battle → results grants the stage's EXP/Zel/Karma on victory and nothing on defeat; entry spends energy; and against an injected clock, regen yields +20 energy per simulated 60 minutes (1 per 3 minutes — KD-2) [src H§V5; rate prop KD-2]. | Quest-loop integration test with a fake clock (simulate 60 minutes → +20). | UC-1 → V5 → spec quest-loop row | Must |
| VR-6 | Saves round-trip identically (save → load → same state); a prior-version save migrates without crashing; a corrupt file loads as a fresh state with a logged warning, never a crash [src H§V6]. | SaveManager unit tests with v0-shaped and corrupt fixtures. | UC-4 → V6 → spec saves row | Must |
| VR-7 | The engine rejects invalid commands so the UI cannot corrupt state: orders for KO'd units, bursts on short gauges, and resolution of an incomplete command set are refused [src S§engine seam]. | Engine rejection unit tests, one per invalid-command class. | UC-1, UC-2 → spec engine seam | Must |

### RL — reliability

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| RL-1 | A save write that fails partway leaves the previous save loadable — no interrupted write can produce a lost or unreadable save [src S§saves crash-tolerant writes]. | Failure-injection test in the save suite (write through a deliberately failing final step, assert the old save still loads). | UC-4 → V6 → spec saves row | Gate |

### PF — performance (the seconds-per-turn budget, KD-4)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| PF-1 | From a command input to its visible acknowledgment, at most 100 ms elapses at the 1280×720 desktop target [prop KD-4]. | Headless UI test injecting input events and timing the acknowledgment. | UC-2 → H§E → spec battle screen | Must |
| PF-2 | One full phase — six player actions resolved, BC dropped and distributed, enemy turn — computes in at most 200 ms headless, so machine time never eats the per-turn seconds budget [prop KD-4]. | Timing assertion in the replay test. | UC-1, UC-2 → H§E → spec engine | Must |
| PF-3 | A player can order a full six-unit squad within 30 s of the command phase opening — about 5 s per order [prop KD-4]. Held constant meanwhile: the UI never gates ordering beyond the rules. | Observed in the human playtest (V8) — pacing is judged, not automated; this row does not pretend otherwise. | UC-2 → H§E → spec battle screen | Should |

### DE — dependencies and environment

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| DE-1 | CI runs headless gdUnit4 on Ubuntu against a pinned Godot 4.7.x binary, and the identical command runs locally [src H§V7; spec CI row]. | CI workflow review plus a green run on the first push of every story. | UC-5 → V7 → spec CI row | Must |
| DE-2 | A fresh clone builds and runs the game on a clean desktop install; the environment (engine version, OS) is recorded per test run [src skill DE guidance]. | CI's clean-checkout build doubles as the reproducibility check. | UC-5 → V7 | Should |

### OB — observability

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| OB-1 | A corrupt or failed save load is never silent: the fresh-state fallback logs a warning, and no discarded data goes unlogged [src H§V6]. | Log assertion in the save tests. | UC-4 → V6 → spec saves row | Must |

### EV — evaluation

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| EV-1 | Every mechanized verification need V1–V7 exists in the suite and traces check → UC → spec row through this document's numbering [src H§G4]. | Traceability audit against §4. | UC-1..UC-5 → H§A | Must |
| EV-2 | The demo gate (V8) is recorded as a human playtest with a written verdict from Michael; no document or test claims it was automated [src H§G3]. | The playtest record exists before the demo is called passed. | UC-1 → V8 → spec playtest row | Must |

### CH — change

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CH-1 | The four our-rule constants ship as data values the playtest can retune without code changes; for the two fixed rules (KD-3) the rule *shape* changes only with a spec revision, while their numeric parameters stay data, and the tunable two (REC turn-end regen, Spirit Ward) may be retuned or dropped at the playtest without a spec revision [src H§C, resolved KD-3]. | The constants live in data files; the suite pins current values so a data change that breaks a test is visible. | UC-2, UC-3 → H§C, H§F | Must |

## 4. Traceability matrix

| Requirement group | Use cases | Verification needs | Spec source |
|---|---|---|---|
| GC (givens) | all | — | spec locked decisions, scope table, platform row |
| FR-Quest | UC-1, UC-2, UC-3 | V4, V5 | quest loop, turn structure, elements, guard, crit, defeat economy, empty-command path |
| FR-Burst | UC-2 | V2, V3 | BC row, BB row, battle screen |
| FR-Recovery | UC-3 | V5 (defeat grants nothing) | items row, KO path, wipe path |
| FR-Squad | UC-4 | V6 | squad row, saves row, content table |
| FR-Verify | UC-5 | V4, V7 | CI row, replay row, verification table |
| FR-Content | UC-1, UC-4 | — | content table |
| VR-1/VR-2 | UC-1, UC-2, UC-5 | V1, V2 | elements row, damage function |
| VR-3 | UC-2 | V3 | BC row |
| VR-4, FR-Verify.3 | UC-5 | V4 | replay row |
| VR-5 | UC-1 | V5 | quest-loop row |
| VR-6, RL-1, OB-1 | UC-4 | V6 | saves row |
| VR-7 | UC-1, UC-2 | — | engine seam |
| PF-1..3 | UC-1, UC-2 | V8 (PF-3 only) | battle screen, H§E |
| DE-1, DE-2 | UC-5 | V7 | CI row |
| EV-1, EV-2 | UC-1..UC-5 | V1–V8 | verification table, playtest row |
| CH-1 | UC-2, UC-3 | V2, V3 | mechanics table (our-rule rows), H§C |

## 5. Proposed thresholds

Every `[prop]` in the document, collected. None are `[tbd]`: each number below is set, justified, and open for the playtest to tune — a playtest revision updates the row and its dependents in the same edit. **Values not commented on by Michael are treated as accepted** — silence is a choice, and this line says so.

| Threshold | Value | Reasoning | Status |
|---|---|---|---|
| Energy regen rate | 1 per 3 real minutes (+20 per simulated hour) | One spent battle returns in ~30 min, a drained bar in 2.5 h — "one more battle later tonight" survives, stage spam stays capped, and a 10–20 min session is never blocked. Original design default awaiting the playtest; not a recovered Brave Frontier rule. | Proposed — playtest tunes (KD-2) |
| Energy max | 50 | The spec save example's working value; makes a full bar 5 stage entries. | Proposed — tunable data |
| Stage entry cost | 10, flat | Priced but not punitive; keeps stage choice about squad shape, not penny-counting, at three stages. | Proposed — tunable data |
| Input acknowledgment | ≤ 100 ms | Standard instant-feedback budget for a turn-based UI; measurable headless. | Proposed — automated check |
| Phase compute (headless) | ≤ 200 ms | Logic-layer budget so machine time never consumes the seconds-per-turn budget. | Proposed — automated check |
| Command-phase ordering | ≤ 30 s for six orders (~5 s each) | Converts H§E's "seconds per turn" to a pacing number; judged by feel at the playtest, not by a clock. | Proposed — pending playtest (PF-3) |

## Open items (only a running system or Michael can answer)

1. **USERS.md open questions 1–4** — slice reading + repo home, playtest-gate ownership (a second playtester would de-risk without changing the build), licensing posture, deadline/spend. Each answer rewrites scope, not a detail (Assumption 2).
2. **Energy values at the playtest** — the rate is the loop's pacing dial (KD-2); the playtest is where it moves, and §5's rows update with it.
3. **Licensing → asset license choice** — downstream of USERS.md open question 3; blocks nothing until original assets are commissioned.
4. **Carried unchanged from H§F** — web export (out), audio (minimal feedback only), headless replay log format (plain state hash) — revisit only after the loop is stable.
