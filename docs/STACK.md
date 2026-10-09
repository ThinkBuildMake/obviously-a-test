# STACK.md — Brave Frontier vertical slice in Godot 4

**Status:** stage 5 of the documentation pipeline. Written from [`USERS.md`](USERS.md), [`USERS_handoff.md`](USERS_handoff.md), [`REQUIREMENTS.md`](REQUIREMENTS.md), [`KEY_METRICS.md`](KEY_METRICS.md), [`HIGH_LEVEL_DESIGN.md`](HIGH_LEVEL_DESIGN.md), and the committed spec (*Brave Frontier vertical slice in Godot 4*, Blueprint art_zmUVyeNW — cited as "spec"). **Inheritance is binding:** the HLD §8 handoff fixes the module list (the four layers of one client) and rules out any backend, database, or hosting product (KD-6; HLD §5). Reader: architecture (stage 6).

**Michael Chuang was unavailable for the choice review.** Per the pipeline rule, every genuine choice below is recorded as a **working default** — safe to build on, marked for his confirmation in *Open questions*, never a pipeline blocker.

## What this document is

Technology selection, and nothing else. For each module the HLD handoff lets the stack actually choose — engine version pinning, CI runner shape, asset and audio tooling, save-schema convenience — it gives the verified options, their tradeoffs, and the one recommendation, with the deciding requirement cited. Three technology commitments arrive as **given constraints** and are listed as constraint rows, not choices. Schemas, module contracts, and data flow live in `ARCHITECTURE.md` (stage 6); which components exist and their load lives in `HIGH_LEVEL_DESIGN.md` §5.

## How the shortlist was built

Candidates were verified against official sources this session (2026-10-09): Godot's release policy and download archive, the gdUnit4 compatibility table and its GitHub Action's metadata, GitHub's own billing and runner documentation, and each tool's license/pricing page. Sources are listed at the foot of the document. Facts not confirmed against an official page are marked `[verify]`.

## Markers

- **Working default** — the recommended choice, recorded without Michael's confirmation; see *Open questions*.
- **[verify]** — an external fact not confirmed against an official page this session.
- Requirement IDs (`GC-`, `FR-`, `VR-`, `PF-`, `DE-`, `RL-`, `OB-`, `EV-`, `KD-`) point into [`REQUIREMENTS.md`](REQUIREMENTS.md); `HLD §n` points into [`HIGH_LEVEL_DESIGN.md`](HIGH_LEVEL_DESIGN.md); "spec" is the committed Blueprint.

## Summary

| Module | Choice (working default) | Deciding requirement | Runner-up | Cost |
|---|---|---|---|---|
| Engine version pinning | **Godot 4.7.2 stable** (current 4.7 patch), downloaded in CI from the official `godot-builds` release and verified against its published `.sha256` | DE-2 (recorded, reproducible environment) + Godot's release policy (only the newest 4.7 patch is supported) | A Godot version-manager tool | $0 |
| CI runner & test wiring | GitHub-hosted **`ubuntu-latest`** + the gdUnit4 author's **`gdUnit4-action`**, with `godot-version: 4.7.2` and gdUnit4 **v6.2.2** pinned | DE-1, FR-Verify.1 (headless, green-gated, identical command) | Raw `gdUnit4Runner` CLI in the workflow | $0 for a public repo |
| Art tooling | **AI-assisted generation with a mandatory human pass in Krita**; PNGs committed to the repo | GC-5 (agent builders) under GC-3 (zero gumi content) | Hand-drawn in Krita; Aseprite ($19.99 one-time) | $0 |
| Audio feedback tooling | **Kenney CC0 audio packs** for the minimal menu/attack feedback | GC-7 + the spec's minimal-audio default | jsfxr-generated WAVs | $0 |
| Save schema convenience | **No addon** — typed validation inside `SaveManager`; the VR-6 test fixtures are the real validator | VR-6 + dependency minimization | A JSON Schema validation addon | $0 |

**Total: $0/month recurring and $0 one-time** at the chosen defaults. HLD §5's cost check holds — nothing here reaches a paid tier. Aseprite is the one paid candidate in any shortlist; it is not chosen.

## Glossary

### Engine & CI

| Term | Meaning |
|---|---|
| Point (patch) release | A maintenance update inside a version series — 4.7.2 is the third release of the 4.7 series. Bug fixes only, no new features. |
| Pinned version | An exact version fixed in configuration (not "4.7 or newer"), so every machine and every CI run uses the identical binary. |
| SHA-256 checksum | A fingerprint of a downloaded file. Comparing it against the publisher's published value proves the file arrived uncorrupted and untampered. |
| Headless mode | Running the engine with no window, graphics, or sound — how a test suite runs on a CI server that has no screen. |
| gdUnit4 | The unit-testing framework for Godot this project's test suite is written in; it runs the tests and reports green or red per check. |
| GitHub-hosted runner | A fresh virtual machine GitHub provides for each CI run; `ubuntu-latest` means the newest Ubuntu image. Standard runners are free with unlimited minutes for public repositories. |
| GitHub Action | A reusable workflow step from a shared catalog. `gdUnit4-action` is the step that wires the gdUnit4 test framework into CI. |

### Licensing

| Term | Meaning |
|---|---|
| CC0 | A Creative Commons license that dedicates work to the public domain: anyone may use it for anything, attribution optional. |
| EULA | A proprietary software license agreement. Aseprite's permits shipping art made with the tool in any product, open or commercial. |
| GPLv3 | The GNU General Public License. It governs the Krita program itself, not the images a person draws with it. |

### Saves

| Term | Meaning |
|---|---|
| `user://` | Godot's path prefix for the player's local data directory — the standard place a game writes its save file. |
| Versioned save / migration | The save file records a schema version number, so a newer game build can load an older save by upgrading it in defined steps. |
| JSON Schema validation | A declarative description of what a JSON file may contain, checked by a validator program. This project validates by typed code and tests instead (Module 4). |

## Constraint rows — given, not chosen

These three arrive locked by the spec and REQUIREMENTS.md. This document does not re-litigate them; the modules below only make them mechanically honest.

| Constraint | Value | Source |
|---|---|---|
| Engine | Godot 4.7.x stable, GDScript (not C#) | GC-1; spec locked decisions |
| Test framework & CI | gdUnit4, headless in GitHub Actions, green gates every merge, identical command locally | DE-1; FR-Verify.1; spec CI row |
| Save format | Versioned JSON in `user://`, crash-tolerant atomic writes | spec saves row; RL-1 |

Also decided by absence of a requirement, listed so the absence reads as a decision rather than an oversight: **no auth** (GC-8 — no accounts exist), **no observability product** (OB-1 is met by Godot's own logging), **no hosting product** (GC-2 desktop demo; web export stays out by default), **no backend or database of any kind** (KD-6; HLD §5).

## Module 1 — Engine version pinning

**Why it exists:** GC-1 locks the 4.7 series but not the patch, and DE-1 requires CI to build against a *pinned* Godot binary. This module picks the exact point release and the mechanics that keep the pin honest.

Verified today (2026-10-09): the current stable in the 4.7 series is **4.7.2** (released August 18, 2026), after 4.7.1 (July 14, 2026) and 4.7.0 (June 18, 2026); 4.8 exists only as dev builds. Godot's release policy supports only the newest patch in a series, and official release assets publish a per-file `.sha256` checksum asset.

| Option | Pros | Cons |
|---|---|---|
| ✅ **Pin 4.7.2 — direct download + checksum** — CI (and each dev machine's install script) fetches the Linux x86_64 editor build from the official `godotengine/godot-builds` 4.7.2 release and runs `sha256sum --check` against the published `.sha256` asset; the version string lives in exactly one place | Official checksums make DE-2's "environment recorded per run" mechanical instead of aspirational; no third-party tool in the trust chain; an upgrade is one constant plus one checksum | The CI story writes ~10 lines of download/verify boilerplate |
| Floating `4.7` via a setup action's version input | Zero pin code to write | Defeats DE-2's reproducibility and VR-4's determinism story — a patch bump could change behavior mid-milestone |
| A Godot version-manager tool (e.g. `godotenv`-style) `[verify]` | Nicer local multi-version UX | One more tool CI must trust, for a guarantee the checksum already provides |

**Choice (working default):** Godot **4.7.2 stable**, downloaded from the official `godot-builds` release and `.sha256`-verified in CI. **Deciding:** DE-2 (reproducible, recorded environment) plus Godot's release policy — only the newest 4.7 patch is supported, so anything older is a version in decay.

**Known issue, flagged in this row:** the gdUnit4 compatibility table retrieved this session names Godot **4.7 and 4.7.1** for gdUnit4 v6.2.x; it does not yet explicitly name 4.7.2. The default still pins 4.7.2 (newest supported patch), and the first CI story verifies the pairing — falling back to 4.7.1 if an incompatibility surfaces. The exact Linux asset filename is confirmed against the release page by the same story `[verify]`.

**Needs from the design:** nothing beyond what the spec already commits — CI installs a pinned binary; the pin is a single constant the CI story consumes.

## Module 2 — CI runner & test wiring

| Option | Pros | Cons |
|---|---|---|
| ✅ **`ubuntu-latest` + `MikeSchulze/gdUnit4-action`, both versions pinned** | Maintained by gdUnit4's author; automates the headless invocation Godot's docs prescribe for GPU-less CI; `godot-version` and gdUnit4 `version` inputs pin both tools in the workflow file | A third-party action sits in the trust chain — mitigated by pinning it by tag |
| Raw `gdUnit4Runner` CLI in the workflow (Godot downloaded per Module 1) | Maximum control, and DE-1's "identical command locally" becomes literally the same string | More workflow code to write and maintain for no number it improves |
| Self-hosted runner | Full control of the environment | Real maintenance burden with zero requirement behind it — the hosted runner meets every number |

**Does a GitHub-hosted runner meet every number? Yes, today — with the numbers.** Standard hosted runners are **free with unlimited minutes for public repositories**; if the repo stays private, GitHub Free's plan-included **2,000 minutes/month** still covers hundreds of runs at this suite's size. The public-repo `ubuntu-latest` machine is **4 vCPU, 16 GB RAM, 14 GB SSD** against a workload whose heaviest automated check (PF-2) needs **≤200 ms** of headless compute and whose whole run is minutes-scale — orders of magnitude of headroom on every axis.

**Caching:** skip it in v1. `actions/cache` can cache the Godot download (10 GB per-repo limit, entries evicted after 7 days idle), but a minutes-scale suite doesn't need the optimization until the download is measured to dominate run time. Adding it later is one workflow step.

**Choice (working default):** `ubuntu-latest` + `gdUnit4-action` with `godot-version: 4.7.2` and gdUnit4 `v6.2.2` pinned. **Deciding:** DE-1 / FR-Verify.1 — headless on Ubuntu, green gates every merge, and the same runner CLI locally and in CI.

**Needs from the design:** none — the engine/suite seam (HLD §4: the suite drives the engine's operations and `state_hash()` headless) is exactly the surface the action invokes.

**License:** gdUnit4 is MIT — free to use in a free public project (GC-7).

## Module 3 — Original asset tooling (art and audio feedback)

**Why it exists:** the slice ships eight original units, three enemy types, five UI screens, and minimal feedback audio — all original (GC-3) for a free public project (GC-7). This module picks what *creates* those files; the content itself is the content table's business (HLD §5, FR-Content).

### Art

| Option | Pros | Cons |
|---|---|---|
| ✅ **AI-assisted generation + mandatory human pass in Krita; PNGs committed to the repo** | Fastest path for an agent-built pipeline (GC-5); the human pass in a free editor restores authorship, fixes artifacts, and enforces one consistent palette; output is original-by-construction, not ripped | Provenance must be documented per asset batch; AI-output copyright varies by jurisdiction — the human pass is what makes authorship clean; style drifts unless prompts are pinned |
| Hand-drawn in Krita (free, GPLv3) | Cleanest authorship story; zero tool cost; forever free | Slowest; quality rides entirely on whoever holds the mouse |
| Aseprite ($19.99 one-time, EULA) | Best-in-class pixel-art editor; its EULA explicitly permits shipping its output in any product, open or commercial | Paid; and a GUI editor an agent team cannot drive — it buys nothing this pipeline needs |

**Choice (working default):** AI-assisted generation with a **mandatory human pass in Krita**; PNGs committed to the repo, one provenance note per asset batch. **Deciding:** GC-5 (the team is agent builders) working under GC-3 (original everything) — with the human pass as the guardrail that keeps authorship and quality human-owned. **This is the row that most needs Michael's confirmation**, because it touches USERS.md OQ3's licensing posture; hand-drawn Krita is the fallback that sidesteps the question entirely.

### Audio feedback (spec default: minimal feedback only — no music pipeline)

| Option | Pros | Cons |
|---|---|---|
| ✅ **Kenney CC0 packs** (interface / RPG audio) | CC0 public-domain — zero license friction for GC-7, no attribution required; free; production-ready blips for menu and attack feedback | A fixed catalog — not every imagined sound exists in it |
| jsfxr / sfxr-generated WAVs | Procedural and endlessly tweakable; free | Tool licenses vary by fork — the exact project's license must be verified before its output ships |
| Record or commission | Exact control | Out of all proportion to a feedback-only default |

**Choice (working default):** Kenney CC0 packs for the menu/attack feedback; jsfxr as the gap-filler when a needed sound isn't in the packs. **Deciding:** GC-7 plus the spec's minimal-audio default — CC0 needs no attribution and no negotiation.

**Needs from the design:** already covered by the spec's own rules — assets are files committed to the repo and loaded as Resources; nothing loads from outside the repo. The repo **LICENSE** itself (GC-7's handoff review) is Michael's OQ3 call and is deliberately not chosen here.

## Module 4 — Save schema convenience

| Option | Pros | Cons |
|---|---|---|
| ✅ **Hand-rolled validation in `SaveManager`** — Godot's built-in JSON class, typed parsing, per-key checks, a schema-version constant, and migration functions | Zero dependencies; VR-6's fixtures (round-trip, v0-shaped migration, corrupt file) already own every failure mode the addon would cover; the file is <10 KB with ~8 known keys | The validation is written by hand (dozens of lines) |
| JSON Schema + validator addon | A declarative schema file | A third-party addon for a problem the test suite already owns — dependency and trust for no new guarantee |
| Godot `ConfigFile` instead of raw JSON | Built-in | Not JSON — contradicts the given save format outright |

**Choice (working default):** no schema addon — validation lives in `SaveManager` as typed, versioned parsing; the VR-6 suite is the validator. **Deciding:** VR-6 (the tests are the real validator) and dependency minimization on a file this small.

**Needs from the design:** `SaveManager` owns the schema-version constant, the migration chain, and the corrupt-file fallback (VR-6, OB-1, RL-1). How it structures that is architecture's call (stage 6), not this document's.

## Go-live prerequisites

| Prerequisite | Owner | Lead time |
|---|---|---|
| Godot 4.7.2 + gdUnit4 v6.2.2 installed on dev machines; the same versions pinned in CI | First build story | None — free downloads |
| First CI story proves the workflow end to end (the spec's own risk note: CI is unproven until the first story runs it) | First story | One story |
| Repo LICENSE chosen **before any asset is committed** (GC-7's license/readme review) | Michael (USERS.md OQ3) | None; blocks nothing until the first asset lands |
| Kenney pack downloaded and committed; the provenance note started | First asset story | None |

## Open questions

1. **All four working defaults await Michael's confirmation** (skill Step 4, adapted — he is away; the pipeline does not block). An answer the other way changes tooling, not the design.
2. **Art tooling is the row to confirm.** AI-assisted + human pass touches USERS.md OQ3's licensing posture; if the answer is "no AI-generated assets," hand-drawn Krita replaces it with no other change to this document.
3. **The gdUnit4 ↔ Godot 4.7.2 pairing** — the compatibility table names 4.7 and 4.7.1 for v6.2.x; the first CI story verifies 4.7.2 or falls back to 4.7.1 (Module 1's flag).
4. **Carried, not chosen here:** repo LICENSE (OQ3), web export (out by default), full audio pipeline (out by default) — none adds a tool to the stack.

## Handoff — what stage 6 (architecture) inherits

The four client layers keep HLD §8's module list unchanged. The architecture doc adds three things this document commits: the single-place version constants (Godot 4.7.2, gdUnit4 v6.2.2) the CI story consumes; `SaveManager` owning schema version, migrations, and the corrupt-file fallback as designed in Module 4; and the asset rule that only repo-committed PNG/WAV files ship, with provenance noted per batch.

## Sources (all verified 2026-10-09)

- Godot release policy — https://docs.godotengine.org/en/latest/about/release_policy.html
- Godot download archive (4.7.2 stable, 18 Aug 2026) — https://godotengine.org/download/archive/index.html
- Official builds with per-asset `.sha256` files — https://github.com/godotengine/godot-builds/releases
- Godot 4.7 command-line tutorial (`--headless` on GPU-less CI) — https://docs.godotengine.org/en/4.7/tutorials/editor/command_line_tutorial.html
- gdUnit4 compatibility table (v6.2.x ↔ Godot 4.5–4.7.1) — https://github.com/godot-gdunit-labs/gdUnit4
- gdUnit4 v6.2.2 release (7 Oct 2026) — https://github.com/godot-gdunit-labs/gdUnit4/releases
- gdUnit4 CI docs (`gdUnit4-action`) — https://mikeschulze.github.io/gdUnit4/faq/ci/
- gdUnit4-action inputs — https://github.com/godot-gdunit-labs/gdUnit4-action/blob/master/action.yml
- GitHub Actions billing (public repos free) — https://docs.github.com/en/billing/concepts/product-billing/github-actions
- GitHub-hosted runner specs — https://docs.github.com/en/actions/reference/runners/github-hosted-runners
- Dependency caching limits — https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching
- Aseprite FAQ (price, output rights) — https://www.aseprite.org/faq/
- Krita license (GPL) — https://krita.org/en/about/license/
- Kenney asset license (CC0) — https://kenney.nl/support
