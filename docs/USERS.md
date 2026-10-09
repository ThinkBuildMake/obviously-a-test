# USERS — Brave Frontier vertical slice in Godot 4

> **Status:** Stage 1 of the documentation pipeline. Written from the committed spec (*Spec: Brave Frontier vertical slice in Godot 4*, art_zmUVyeNW), which stands in as the brief because the requester, Michael Chuang, was unavailable for an interview. Every claim that only he can confirm lives in **Open questions** with a working default, so downstream stages can proceed without blocking. Reader: the engineering team building this slice. Companion file: [`USERS_handoff.md`](USERS_handoff.md) — the design content that was deliberately kept out of this document.

## Scope

This document covers: who the slice is for, the problem it answers, the moments it enters, the use cases it must survive, the minimum first iteration, and the constraints the project arrived with. It leaves to downstream docs: functional and non-functional requirements (stage 2), the metrics that watch them, the component sizing (high-level design), technology choices (stack), and module contracts (architecture). Nothing here says how the system is built — when a design decision showed up during this pass, it was cut and logged in the handoff file rather than written in.

## Target user

One person at one moment: **the returning Brave Frontier player at the end of an evening, deciding to run one more battle.** The first concrete instance is Michael Chuang — he asked for this recreation, he is its first player, and the spec makes his playtest the demo gate: one full quest from the menu, judged on whether the loop *feels* like Brave Frontier's. The persona is inferred from that gate and from the request ("recreate Brave Frontier"), not from a survey; see Evidence limitations.

**Who is not the target:**

- Players who never played Brave Frontier and want a fresh live-service gacha — there is no gacha, no acquisition loop, no live service here.
- Players who want the full content suite now — no arena, no evolution, no town, no story.
- Mobile players — the demo target is a desktop window, 1280×720, mouse and keyboard.
- Anyone hoping for gumi's characters, art, or story — the project will not ship them, ever.

## The problem in numbers

- Brave Frontier launched in Japan on July 3, 2013 (A-Lim) and worldwide in December 2013 (gumi). All service ended in April 2022: the Japanese servers went down April 25, 2022, and the global service's last day was April 27, 2022 (Wikipedia; Siliconera's January 2022 shutdown announcement; the qoo-app store page now marked "no longer in operation").
- Since April 27, 2022 there has been no legitimate way to play the original: it was server-dependent, and the original is, in a fan-maintained legacy page's words, "no longer playable" (meowdb). Nearly nine years of the system people remember ended in a maintenance window.
- The community did not dissolve. Its wiki and subreddit stayed active after the shutdown, and a fan-maintained wiki reports a teased official revival (*Brave Frontier Origin*, aimed at 2027) with no playable build and no confirmed platform as of this writing (meowdb, page dated July 2026).
- The harm behind these numbers is not "an old game closed." It is that one specific, remembered system — six elements countering each other, Brave Crystals filling Brave Burst gauges, sparks from simultaneous hits — became unreachable except through server-dependent clients or ripped game data. Whether people would play a faithful rebuild is measured nowhere; the claim rests on the community's persistence and on Michael's own request.

## Why these moments

The slice targets the **10–20 minutes of an evening when a player can run one stage**: squad prep, the battle itself, the results beat, and the return to the map. It deliberately does **not** target the commute (no mobile layout in the slice), the long collection-management session (no gacha or evolution to manage), or the story reader (there is no story). Those are sequencing choices, not judgments — each one bolts onto the finished loop later. What the slice is for is the moment where a better decision changes the fight in front of you, and where that change is checkable: a damage number that reflects the element chart, a burst that fires exactly when the gauge is full, a spark that pays out for stacking hits.

## The workflow walk

Michael, Tuesday, 9:06 PM. His desktop is free; he opens the build with eleven minutes to spare. The main menu loads straight into where he left off (UC-1 starts here). He has done this loop twice, so squad select takes ninety seconds (UC-4): eight units, six slots, Ignar leads because the stage ahead is earth-heavy and fire counters it. Stage select — Elrune Grove, stage 1, the energy covers it — and the battle opens on wave one.

Orders go out in seconds per turn (UC-2 fires on turn two): Mera's gauge filled from last wave's crystal pool, and the Thorn Crawler in front of her is the spark target — Ignar and Thal both hit it, the spark pays out, and the crystal pool at phase end refills Vexa's gauge instead. 9:14 PM, wave two: a sweep goes badly, Thal goes down (UC-3). The item tray is the only way back for him — a Revive Light costs Vexa her action, and Michael guards the rest of the turn instead of pushing. The wave breaks, results screen: EXP ticks Lumis to a new level, Zel and Karma accrue, the stage clears (UC-1 closes). He looks at the map, sees the next stage's cost against his energy bar, decides it's tomorrow's fight, and quits — the save already holds the squad, the level, and the cleared-stage mark.

Ten minutes, one full loop. That is the whole product promise: every signature mechanic reachable in one sitting, and the loop closed.

## Use cases

### UC-1 — Run one stage, from menu to results

| | |
|---|---|
| **Trigger** | An evening gap; "one more battle" after opening the build. |
| **Input** | A stage pick from the quest map (energy covers it); orders for every living unit, every turn. |
| **Needs to see** | The stage's waves, each squad unit's HP and burst gauge, per-action results as they resolve, and a results screen paying out EXP, Zel, and Karma. |
| **Does with it** | Judges whether the fight was worth it, bank the rewards, return to the map. |
| **Time they have** | 10–20 minutes for the whole loop. |
| **Must never** | Resolve a player phase while a living unit has no order; grant rewards after a defeat or retreat; strand the player without a way back to the menu. |

**Why this shape of product.** A video playthrough shows this system faithfully, and an archived client plays the real thing — concede both. But the original is unplayable and the teased revival is not out; the archived-client route runs on gumi's content, which this project will not ship; and a rebuild is the only version where the rules are inspectable, tunable data the player's decisions act on directly. What a video cannot do is let the fight be *yours* — and what an emulator cannot do is be legally shippable and extensible. The slice rests there.

### UC-2 — Time the burst

| | |
|---|---|
| **Trigger** | Mid-battle: a burst gauge fills to full while the next wave looms. |
| **Input** | The choice: spend the burst now, or hold it and attack normally. |
| **Needs to see** | The full gauge lit and visually distinct from short ones; what the burst will do; that spending empties the gauge. |
| **Does with it** | Times the burst against the wave — burn it on the boss wave, not the trash. |
| **Time they have** | Seconds per turn, but the decision carries the fight. |
| **Must never** | Let a burst fire on a short gauge; let a full gauge silently reset; hide the burst's effect before it is chosen. |

**Why this shape.** A wiki page can state that bursts exist and a guide can name the optimal time to use one — concede both. The hold-or-spend tension only exists when the spend is the player's, against a gauge they watched fill from crystals their hits dropped. No document, dashboard, or video produces that moment; only a game where the gauge is yours does.

### UC-3 — Recover a battle going wrong

| | |
|---|---|
| **Trigger** | An enemy wave sweep leaves a unit down — or two — with the wave still alive. |
| **Input** | Guard orders, a Heal Potion on the wounded, a Revive Light on the fallen; each item use costs that unit's action. |
| **Needs to see** | Who is KO'd (clearly, persistently), item counts remaining, and that a KO'd unit returns only via revive — never by healing. |
| **Does with it** | Triage: save the healer, guard the rest, accept a loss when the items run out. |
| **Time they have** | Seconds, under pressure. |
| **Must never** | Heal a KO'd unit back; let a KO'd unit act; hide that a total wipe ends the battle in defeat. |

**Why this shape.** A difficulty slider would remove the moment instead of creating it — concede that an easier fight needs no triage. The value of this use case *is* the triage: constrained actions, visible costs, no undo. A report of "you lost" after auto-resolving would not produce it, and neither would a game that quietly revives everyone.

### UC-4 — Set the squad, pick the leader

| | |
|---|---|
| **Trigger** | Before a quest — after a results screen, a level-up, or a new plan for the stage ahead. |
| **Input** | Eight owned units, six slots, one leader pick. |
| **Needs to see** | Elements at a glance, each unit's role (healer, tank, burst support), and what the leader skill grants the squad. |
| **Does with it** | Commits a squad shaped for the stage; the leader's skill applies to everyone. |
| **Time they have** | A couple of minutes. |
| **Must never** | Lose the squad across sessions; apply the leader skill to the wrong units; imply any unit needs acquiring — all eight are owned from the start. |

**Why this shape.** A tier list ranks units; a spreadsheet can even suggest a lineup — concede that. What neither does is make the leader pick a real trade: a fire leader into an earth stage is a decision with a visible payoff in the fight, and it is only a decision because the game commits the player to six slots and one leader. The squad screen earns its place by turning knowledge into commitment.

### UC-5 — Verify a combat-rules change (developer use case)

| | |
|---|---|
| **Trigger** | The engineering team touches a combat rule — a constant, a distribution order, an element multiplier. |
| **Input** | The automated test suite, run headless on every push; a scripted battle replay from a fixed seed. |
| **Needs to see** | Green or red, and on red: which rule broke — element chart, damage math, BC economy, or the replayed battle's final state. |
| **Does with it** | Merges only when green; runs the identical command locally before pushing. |
| **Time they have** | Minutes of attention per change, not a replay-everything-by-hand evening. |
| **Must never** | Leave a rule change verified only by memory or one manual click-through; let the same seed produce two different battles. |

**Why this shape.** Manual playtesting catches feel but not silent regressions — concede that it is the only way to judge the demo gate. For everything else, the deterministic core is the point: the same seed replaying the same battle turns "we didn't break the game" from a hope into a checked claim. This is the developer's version of the player's trust that damage numbers mean what they say.

## The minimum product

The set below stays above the floor because one ten-minute session touches every signature mechanic — element counter-picks in the damage numbers, crystals pooling into a hold-or-spend decision, a spark on a stacked hit, an item triage, a results screen that pays out — and closes the loop back to the map with energy ticking. Remove a mechanics row and the demo gate fails on that mechanic's absence; remove a meta row and the battle has nowhere to live.

| Feature | UC | What it does for the user | Acceptable in v1 |
|---|---|---|---|
| Quest map, stage select, energy | UC-1 | Turns "play a battle" into a chosen, priced activity | 1 area × 3 stages; energy spent on entry, regenerates in real time |
| Full battle loop with waves | UC-1 | The core moment: order all units, watch the phase resolve | 2-wave stages, 3 enemy types, victory and defeat both reachable |
| Six-element chart | UC-1, UC-2 | Makes squad composition matter before the fight starts | Correct cyclic chart; advantages and resists visible in damage |
| Brave Crystals → burst gauge | UC-2 | Hits earn the resource that powers the fight's big decision | Drops roll per hit with a per-attack cap; phase-end distribution |
| Brave Burst | UC-2 | The signature spend: attack burst, heal, or support buff | Fires only on a full gauge; empties after use |
| Sparks | UC-1, UC-2 | Rewards stacking hits on one target | Same-target-same-phase detection; +50% base damage, better drop odds |
| Criticals and guard | UC-1, UC-3 | Variance and risk management inside a turn | Per-unit crit chance; guard halves incoming damage |
| Battle items | UC-3 | Recovery is a choice that costs an action | 3 items with limited counts; KO'd units return only via revive |
| Squad select with leader pick | UC-4 | Commitment that shapes the whole fight | Any 6 of 8 units; one leader; leader skill applies to squad |
| Results, EXP, Zel, Karma | UC-1 | The loop pays out and progression accrues | Levels raise stats; defeat grants nothing |
| Save/load | UC-4 | The squad and progress survive the session | Versioned saves; a corrupt file yields a fresh state with a warning |
| Deterministic replay + automated suite | UC-5 | Rule changes are checked, not remembered | Same seed → identical battle; suite runs headless on every push |

## Later iterations

Ordered as the player would want them; each is the plan, not a promise.

| Feature | UC | What it adds | Why not first |
|---|---|---|---|
| Super/Ultimate Bursts (SBB/UBB) | UC-2 | A deeper gauge economy above the burst | Keeps the tuning surface small while the core is judged |
| Status ailments | UC-3 | New ways for a battle to go wrong | Needs a deeper enemy roster before they matter |
| Richer enemy AI | UC-1 | Fights that punish predictable orders | Simple targeting is enough to judge the loop |
| More areas, bosses | UC-1 | Content to spend the loop on | Content multiplies only after the loop is fun |
| Full audio and music | UC-1 | Feel — the part feedback bleeps can't carry | Needs an audio story and asset budget; menu/attack feedback ships |
| Web export | UC-1 | Playable anywhere, no install | Forces asset/UI audits early; cheap to add once the loop is stable |
| Evolution, fusion | UC-4 | Progression depth beyond levels | Out of the slice's sequencing; bolts onto roster data later |
| Controller, mobile | UC-1 | Where and how the loop is played | Demo target is desktop; layout work is real |

## Out of scope

| The project won't | Why |
|---|---|
| Gacha or any acquisition loop | All 8 units are owned from the start; adding acquisition changes the product, not just the content |
| gumi's characters, art, story, or branding | IP boundary, committed in the spec: original everything, in the repo and in the game |
| Cloud saves, accounts | Local versioned saves cover the demo; accounts are infrastructure without a user moment yet |
| Monetization, analytics | No intended use for either has been stated; nothing measures a player the product doesn't have |
| Localization beyond English | English-only is the demo's language |
| Friend-guest system | The spec simplifies it away; squads are self-contained |

## Constraints the project arrived with

All of these arrive via the spec as the standing brief; none were re-confirmed with Michael this session.

| Constraint | Value |
|---|---|
| Deadline | None given. Default: milestone-sequenced through the build pipeline. |
| What drops first if half done | Not stated. Default from the spec's shape: content breadth drops, the loop and battle rules hold. |
| Team | Agent builders plus Michael as reviewer and demo gate. |
| Spend | No budget stated. |
| Services / vendors | None mandated except the engine lock: Godot 4.7.x stable, GDScript. |
| Platform (demo target) | Desktop window, 1280×720, mouse + keyboard. |
| Intended use | A free, playable project; nothing in the brief authorizes selling it. Commercialization unconfirmed — see open questions. |
| IP / legal | Zero gumi content: no ripped art, no character names, no story, no in-game "Brave Frontier" branding. |
| Language | English only. |

## Open questions (only Michael can answer)

Working defaults below let downstream stages proceed; an answer against a default rewrites scope, not just a detail.

1. **★ Is the vertical-slice reading of "recreate Brave Frontier" confirmed, and is `ThinkBuildMake/obviously-a-test` the project home?** Default: yes to both — the spec's recommended option, and the only connected greenfield repo. An answer the other way rewrites the entire build or re-points the sandbox.
2. **★ Whose judgment defines "feels like Brave Frontier" — and is Michael's playtest the only demo gate?** Default: yes. That is a single point of failure: a second playtester would de-risk the gate without changing the build.
3. **What is the intended use and licensing posture?** Default: free, public repo; original assets only; nothing commercial until stated. This decides how original assets are licensed and whether anything about the project could ever ship beyond this workspace.
4. **Is there a deadline or a spend cap the pipeline should sequence against?** Default: none; milestone-ordered, content drops first.

## Evidence limitations

- No study measures demand for a Brave-Frontier-like rebuild. Nothing in this document is evidence of market size; the audience claim rests on the game's nine-year run, the shutdown dates, the community's post-shutdown activity, and one firsthand request.
- The shutdown dates come from press and store snapshots (Siliconera, qoo-app, Wikipedia) that agree with each other; the "no longer playable" line and the *Brave Frontier Origin* teaser come from a fan-maintained wiki (meowdb), not from gumi or Alim.
- What "feels like Brave Frontier" means comes from the spec's retrieved mechanics summary (en.namu.wiki, combat and elemental sections, via Perplexity, 2026-10-09) — not from a player survey. The original's exact damage formula was never public, so every constant in the slice is a design choice awaiting the playtest.
- None of these sources measure whether this slice satisfies anyone. The playtest is the only check, and it is not automatable.

## References

- Spec: *Brave Frontier vertical slice in Godot 4* (committed Blueprint, art_zmUVyeNW) — scope, mechanics table, verification table, locked decisions.
- Brave Frontier — Wikipedia: https://en.wikipedia.org/wiki/Brave_Frontier (launch dates, service history).
- Siliconera, "All Brave Frontier Mobile Games Will Shut Down in Japan in April 2022" (Jan 31, 2022): https://www.siliconera.com/all-brave-frontier-mobile-games-will-shut-down-in-japan-in-april-2022/
- qoo-app store page, Brave Frontier (Japanese): https://m-apps.qoo-app.com/en-US/app/1254 ("server has been shut down on 25 April 2022").
- NiaMeowDB, "The Brave Frontier Legacy": https://meowdb.com/db/brave-frontier-origin/mechanics/the-brave-frontier-legacy (April 27, 2022 global end; "no longer playable"; *Brave Frontier Origin* 2027 teaser).
- Mechanics summary used by the spec: en.namu.wiki, "brave frontier" (combat and elemental sections), retrieved via Perplexity, 2026-10-09.
