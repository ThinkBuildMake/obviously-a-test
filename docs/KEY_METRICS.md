# KEY_METRICS.md — Brave Frontier vertical slice in Godot 4

**Status:** stage 3 of the documentation pipeline. Written from [`USERS.md`](USERS.md), [`USERS_handoff.md`](USERS_handoff.md), [`REQUIREMENTS.md`](REQUIREMENTS.md), and the committed spec (art_zmUVyeNW). Inherits REQUIREMENTS §0 "For stage 3": the mechanized verification set V1–V7 as the tracked set, V8 (the human playtest) as the untracked demo gate.

**Nothing is running yet.** No code exists in this repo. Every number below is a target the build will be measured against — by CI jobs, by test output, and by the playtest record once the stories that produce them land. There are no live dashboards, and this document asks for none.

## Guarantee metrics

What we watch so the slice keeps its promises. All six are read from the automated gdUnit4 suite and CI, run headless on every push (FR-Verify.1).

| # | Metric | Threshold (as required) | Requirement | How it's verified · where it's read |
|---|---|---|---|---|
| 1 | CI green on every push | The full suite passes headless in CI on every push; a merge requires green | FR-Verify.1, DE-1 | GitHub Actions job result, per push |
| 2 | Replay determinism: same seed, same battle | Two runs of the scripted battle under the same seed end at an equal final state hash, every suite execution | VR-4, FR-Verify.3 | The replay test's assertion, in the suite output |
| 3 | Saves never lose or corrupt silently | Save → load round-trips identically; a prior-version save migrates without crashing; a corrupt file loads as a fresh state with a logged warning; a failed write leaves the previous save loadable | VR-6, RL-1, OB-1 | Save suite tests (round-trip, migration, corrupt fixture, failure injection), in the suite output |
| 4 | Energy regen matches the shipped data | +20 energy per simulated 60 minutes against the injected clock — 1 per 3 real minutes (KD-2) | VR-5 | Quest-loop integration test with a fake clock, in the suite output |
| 5 | Input acknowledgment time | ≤ 100 ms from command input to visible acknowledgment, at the 1280×720 desktop target | PF-1 | Headless UI test injecting input events and timing the acknowledgment, in the suite output |
| 6 | Phase compute time | One full phase — six actions resolved, BC dropped and distributed, enemy turn — computes in ≤ 200 ms headless | PF-2 | Timing assertion in the replay test, in the suite output |

Two notes on what these watch:

- **The rest of the verification set rides metric 1.** The element chart (V1), damage math (V2), BC economy (V3), and engine-rejection tests (VR-7) run in the same suite execution; they are pinned checks, not drift-watchers, so they stay requirements enforced by CI rather than rows here. When any of them goes red, metric 1 is the row that catches it.
- **Dashboards watch data, not constants in code** (CH-1, KD-3). The energy rate and every combat constant ship as data values. If the playtest retunes the rate, the row above and the suite's pinned value update in the same edit — metric 4 tracks "regen matches the shipped data value," never a hardcoded constant.

## Success metric

**Success means Michael clears one full quest from the menu to the results screen in a single evening session and writes down that the loop feels like Brave Frontier's.** (USERS.md target user; spec playtest row; V8, EV-2, GC-10.)

The demo gate is one recorded human verdict, and no document dresses it up as a test: a playtest record exists, one full quest is cleared menu-to-results in a single session of 10–20 minutes, and Michael's written verdict on the feel of the loop is in it. The slice passes only on that record. Target: verdict recorded and positive before the slice is called done. Where it's read: the playtest record, not CI.

What it can't tell you: it is one person's judgment (GC-5; USERS.md open question 2) — it says nothing about whether another returning player would feel the same, and nothing upstream measures demand.

## Supporting metrics

The pacing levers behind the verdict — if the feel verdict comes back negative, these say whether pacing was the reason. All three are observed and recorded in the playtest record; none is automatable (KD-4).

| # | Metric | Target | Requirement | Where it's read |
|---|---|---|---|---|
| 1 | Quest-loop time | One menu → results session lands in 10–20 minutes | UC-1 (USERS.md §workflow walk, H§E) | Playtest record |
| 2 | Squad-select time | ~2 minutes, leader pick included | UC-4 (H§E) | Playtest record |
| 3 | Command-phase ordering time | Six orders given in ≤ 30 s (~5 s each) | PF-3 (KD-4) | Playtest record — judged as pacing, not timed |

## Upstream fixes and open questions

- The 30 s ordering budget (PF-3) is a proposal pending the playtest (REQUIREMENTS §5, KD-4); a playtest revision updates it and this row together.
- Energy rate, max 50, and flat-10 entry cost are proposed data values (KD-2); guarantee 4's row updates with any retune.
- USERS.md open question 2: a second playtester would de-risk the single-judge success metric without changing the build.
