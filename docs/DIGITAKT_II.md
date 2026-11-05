# Digitakt II: the plan to support it

A plan for the owner to review, written 5 November 2026 against `main` at `dc5688ba`. It takes the Digitakt II (OS 1.17) from research to a machine with a library, downloads and the Octatrack's stack ([recipe](MACHINE_RECIPE.md)). Nothing here is built yet: each step below ships as its own pull request after the owner agrees to it. Where the machine stands today is in its guide ([sdk/machines/digitakt-ii/README.md](../sdk/machines/digitakt-ii/README.md)); when a step lands, that guide and the recipe's table change with it, and this plan loses the step.

## Where it stands (5 November 2026, after step 1)

| Piece | State | Where |
| --- | --- | --- |
| Firmware format, rebuild, boot, core | `done`, `done`, `started`, `done` in the profile | [machine.json](../sdk/machines/digitakt-ii/machine.json) |
| Site build | Local preview: the kit 0.7.0 worker links Modwerk's base with the selected mods from the owner's OS 1.17; downloads off (`DIGI_DOWNLOADS_ENABLED['digitakt-ii'] === false`) | [machine-build.ts](../src/engine/elekloader/machine-build.ts), [elekloader-digitakt2-perform-v1.0.json](../sdk/imports/elekloader-digitakt2-perform-v1.0.json) |
| Core contract | Interface 1.1: core 1.0's six UI events plus core 1.1's entries (marked `since` 1.1); the machine's core is 1.1 | [core-profile/interface.json](../sdk/digitakt-ii/core-profile/interface.json) |
| Base | Built: core 1.1's source (#221's `0df8409`) by `npm run machine:base`, identity `f41b89cd`; the site builds every Digitakt II set on it | [elekloader/](../sdk/machines/digitakt-ii/elekloader/README.md) |
| Library | Empty. Mute Latch (crisp909, #1160) is the first module under review; the owner asked it to move to `sdk/digitakt-ii/modules/mute-latch/` with a pinned package, and to take the key LEDs from core 1.1's `ev_key_leds` rather than its own patch of the site core 1.1 owns | [modules/](../sdk/digitakt-ii/modules/README.md) |
| USB link, loader, base updates, MCP tools, machine page | Not started | [recipe](MACHINE_RECIPE.md#the-pieces) |
| Emulator | digikit (toonst's, AGPL-3.0-or-later): ColdFire and SHARC, OS 1.17; captures only, never hardware evidence | [digikit](https://github.com/toonst/digikit) |

Upstream: elekloader's `main` (5 November) still ships core-dt2 1.0. Core 1.1 is [elekloader#221](https://github.com/elekloader/elekloader/pull/221) by toonst, rebased on elekloader's `main` on 5 November (`0df8409`) and cut to the Digitakt II and what a mod needs: the frame values, key LEDs (with `fw_led_set`), project and kit data and value rows. Its Digitone II half left the pull request, since elekloader has released a different core-dn2 0.3. This pull request refreshes Modwerk's copy to `0df8409` (`sdk/imports/core-dt2-source-0df8409.json`). Elekloader's 0.9.0 plan (#1129) leaves the Digitakt II out of scope. Since 28 October a Digitakt II hook goes into Modwerk's base and does not wait for elekloader ([decision](decisions/2026-10-28-modwerk-base-every-machine.md)).

## What is decided, and what is open

Decided by the owner and binding on this plan:

- **The base is Modwerk's** and owns every shared site; a change to core-dt2 itself goes to elekloader ([recipe](MACHINE_RECIPE.md#rules)).
- **A module never patches a site the core owns** (owner on #1160, 2 November). It subscribes to the hook, or waits for it.
- **A Digitakt II module goes straight into `sdk/digitakt-ii/modules/<id>/`** with its source-only package built in the pinned toolchain, its import record and its release notes `src/community/module-changelogs/digitakt-ii-<id>.json`, and is live on merge; `_submissions/` takes no new modules (owner on #1160, 3 November; #1170).
- **Hardware results are recorded as the named person reported them**, on the exact build.

Open, for the owner (each is asked again in the pull request that needs the answer):

1. **The source pin.** This pull request copies `0df8409`, #221 as rebased; the same sites and entries as `3d7c2e3`, without the exports no module used. Take the source again from elekloader's release if #221 lands changed; otherwise the base builds from this copy. Waiting for a release costs time and buys nothing the base check below does not cover, since the base is Modwerk's either way.
2. **What the base is checked against.** `machine:base` compares a base with the release it started from (`reference.keeps`). There is no core 1.1 release. *Recommended:* compare with the vendored `core-1.0.elemod`, keeping `id`, `sites`, `exports`, `imports`, `collections` and `requires` but not `version` or `build`. Core 1.1 keeps every core 1.0 site, event and table size, so the check proves the base is 1.0 with additions, as the Rytm's base is core-ar2 0.7 with additions.
3. **The per-kit room.** A Digitakt II kit record has 194 unused bytes after its end mark, which leaves 191 bytes of kit data in all, shared by every module (`core_kitdata_room()`). Macro mode's per-kit setup is exactly 191 bytes. Three ways:
   - (a) **A budget, first come.** The profile gets a `Kit data` budget of 191 bytes, each module declares the bytes it takes (a `kitdata:<tag>` claim and its size), and the planner refuses a set over the budget. Macro mode then excludes every other kit-data module, and says so.
   - (b) **More room from the base.** The pattern record has 308 unused bytes after its last field (`0x15acc`–`0x15c00`, by macro mode's storage notes, digikit-measured, not checked here). Each pattern has its own kit, so a base table there gives a module per-pattern-kit room beside the kit record's. It does not travel with SAVE KIT or the kit library, which the kit record does. It needs its own research and an emulator round trip before anyone relies on it.
   - (c) **A base record per module, small.** The base splits the 191 bytes into fixed records, for example three of 63, and a module fits its block in one. Macro mode would then keep only part of its setup per kit.
   *Recommended:* (a) now, because it is honest about the cost and needs only a budget and a planner check, and (b) as research before a second kit-data module appears. (c) changes a module's design to fit a rule no second module has asked for yet.
4. **Card sectors.** Macro mode keeps its configuration (two values) in two card sectors the stock OS never writes (`0x2df0e00`–`0x2df0e02`). digikit's runtime keeps an 82 MiB store on the same card, in an area it measured as unused. Two owners of raw card sectors will meet sooner or later. *Recommended:* the base owns one small settings record on the card, with a record per module, and modules stop writing raw sectors. Until that exists, a module that writes the card claims its exact sector range (`card:<lo>-<hi>`) and the planner refuses overlaps. A small settings record is cheaper than sector claims, but it is base work and should not hold the first modules back.
5. **The copy hook.** Core 1.1 hooks the firmware's `memcpy` (`0x40136388`) so kit data follows pattern copy and paste, PASTE KIT and undo. That adds a compare and a branch to every copy the OS makes, whether or not a module uses kit data, against the recipe's rule that a hook costs nothing while no module uses it. *Recommended:* two variants, as on the Rytm: **plain**, core 1.1 without the kit-data sites and the copy hook, and **kitdata**, with them, chosen by the selection (`MODWERK_BASES` in `machine-build.ts`). The site picks kitdata only for a set with a module that contributes to `core_kitdata`.

## The steps

Each step is one pull request with its own changelog entry and, where something is built, a verification record. The owner reviews every base, link, loader and page. Before each, claim its items on the [agent board](AGENT_BOARD.md); `pr:ready` claims what a branch adds. #1055's proposed entries (`digitakt-ii/events/6`, `events/7`, `tables/1`, `tables/2`) have not been claimed yet.

### 1. The base builds (core 1.1 ships): done

Done on 5 November 2026, in the pull request that carries this plan:

- **Recipe:** [base.json](../sdk/machines/digitakt-ii/elekloader/base.json) is core-dt2 1.1's source as copied, with the include folder, OS 1.17 and the reviewed toolchain. It has no `add` and no variants yet: open item 5 stays the owner's call.
- **Reference check:** the base is checked against core-dt2 1.0's release (item 2) and keeps every 1.0 site, export, import, table and contribution.
- **Identity:** [base-binding.json](../sdk/machines/digitakt-ii/base-binding.json) records `f41b89cd`. toonst built it from their own OS file with m68k-elf GCC 16.2.0 and binutils 2.47.
- **Package:** published in `sdk/imports/packages/modwerk-base-digitakt-ii/` and pinned in `sdk/imports/elekloader-modwerk-base-digitakt-ii-2026-11-05.json`, as the Rytm's base is.
- **Site:** `machine-build.ts` builds every Digitakt II set on the base (`MODWERK_BASES['digitakt-ii']`). The profile's core is 1.1, so a module that uses a 1.1 entry validates. The core 1.0 record stays, as the release the base is checked against.
- **Record:** [docs/verification/2026-11-05-digitakt-ii-base.md](verification/2026-11-05-digitakt-ii-base.md) has the checks, the links and the emulator run.

Left for later:
- a `Kit data` budget the planner enforces (item 3);
- the variants (item 5);
- a unit boot of the base's own package, reported by whoever runs it, with the stock `.syx` restored after.

### 2. The first modules

Each module ships alone, from its author, after this base:

- **Mute Latch** (crisp909, #1160): already asked to move into `modules/`. With the base in, it can light the armed tracks through `ev_key_leds` (owner on #1160) in a later version.
- **Macro mode** (toonst): eight macro knobs on PAGE + MOD, each moving up to four parameters on chosen tracks, applied per frame on top of the kit through `ev_frame_params`, with its LEDs through `ev_key_leds`, its setup in `core_projdata` and, per item 3, `core_kitdata`. Source: [toonst/digi_macro](https://github.com/toonst/digi_macro) (GPL-2.0-or-later). Ported as a contract v3 module in its own pull request. On core 1.1 (`0df8409`) it passed its digikit scenarios (94 + 32 + 12 checks) and, by its author's report, works on a Digitakt II; its Digitone II build waits for a Digitone II core. That pull request waits for this step.
- **Perform Direct and Perform Levels** (toonst, core 1.0 records today): move into `modules/` as contract v3 modules when their author wants them in the library. Until then they stay builder-only records.

What the first module also needs from the tools:

- `module:doctor` covers Digitakt II modules (#1160 found it ignores them). Fix it in a `Fix:` pull request, never inside a module.
- Captures: digikit's emulator, with its capture record (`media/capture.json`, image identity) as Mute Latch has. Write down in [MODULE_UI_CAPTURES.md](MODULE_UI_CAPTURES.md) how a capture is taken there.

When the first module validates, the profile's `status` moves to `preview` and `boot` to `done`. Downloads stay off.

### 3. Downloads

The owner switches `DIGI_DOWNLOADS_ENABLED['digitakt-ii']` on, as for the Rytm (#1190), once at least the base and one module have run on a unit from the site's own build, and the recovery steps in the profile have been followed once on a unit and work as written: FUNC at power on, TRIG 4, the stock `.syx` over MIDI DIN. The page then offers the `.syx` after the owner confirms they know how to get back to stock. The full OS file sent with Transfer stays the route until step 4 and the fallback after it.

### 4. The USB link and loading without a reflash

Payoff: the recipe's reason for a base. Modules load to RAM for review, trial, Keep or Undo, and a power cycle restores the stock state. Confidence: medium. The Digitakt II offers USB MIDI and USB audio, not a vendor interface, so the link is SysEx over USB MIDI, as planned for the Rytm (#878) and the LXR-02.

- **What exists.** digikit's runtime already has the pieces on this OS: a SysEx upload over USB MIDI (`F0 7D 64 6B`, hooked in the MIDI task), a card store, a boot counter with safe mode, a crash record and a log read over SysEx. Its runtime and upload have run on a Digitakt II unit, by its author's report. It loads at boot only, not live. It is AGPL-3.0-or-later, so its code cannot go into a GPL-2.0-or-later base unless its authors relicense it. Its findings (addresses, site behaviour, what the card leaves free) are facts and can be used.
- **The plan.** Port Modwerk's [runtime loader](../sdk/runtime/loader/README.md#porting-to-another-machine) as base `add` sources: the machine glue it lists (stopped, uncached alias, cache invalidation, patchable ranges, masking, paused tasks), trampolines on the core's four UI events, and a SysEx transport in place of the Octatrack's `ep0.c`. The host side is a SysEx transport behind `upload-session.ts` and `module-set.ts`, not a new protocol. Ask toonst whether digikit's upload and store code may be offered under GPL-2.0-or-later. If it may, start from it; if not, write the transport new, using digikit's findings.
- **Base updates over USB.** Elektron's OS update over USB (what Transfer does) stays the route for the base itself at first. Whether a two-slot base like the Octatrack's ([two-slot base](OCTATRACK_TWO_SLOT_BASE.md)) can live beside the sealed main OS needs its own research; don't promise it.
- **MCP device tools** go through the same bridge as the Octatrack's, with the SysEx transport behind them ([FRONTEND_MCP.md](FRONTEND_MCP.md)).

### 5. The machine page

The Octatrack's hub, generalized with only the pages that fit (status, module sets, guided sync), beta-only behind `hasBetaUi` until the owner releases it.

### 6. Sound

The Digitakt II's audio runs on its SHARC DSP, and core 1.1 has no audio hook: `ev_frame_params` moves parameter values once a frame, before the stock modulators, which is enough for macros, LFO-like modules and performance tools, but not for a new effect or machine. digikit runs its own machines and effects on the SHARC (send and insert effects with an FX page, run-time machines), in its emulator and, for one machine, on a unit. Bringing that into a Modwerk base is the largest step and the least certain. It means a DSP loader in the base, its licence question (as in step 4), and qualification on hardware for every effect. *Recommended:* don't start it until steps 1 to 4 are done and a module author asks for it. Then write it up as its own plan with its payoff and confidence, as [OCTATRACK_PACKAGE_DSP_HOOKS.md](OCTATRACK_PACKAGE_DSP_HOOKS.md) did for the Octatrack.

## Risks

- **Emulator-only claims.** Kit data across PASTE KIT and undo has run nowhere yet, and the base's own build has not run; core 1.1 with macro mode has run on a unit only by its author's report. Each step's record keeps "emulator" and "unit" apart, and no step claims hardware it did not see.
- **Later OS releases.** Every address and guard is OS 1.17's. Elektron's next OS moves them, and the stock bytes guarded in the kit and pattern records may be used by it. Supporting a new OS means re-deriving each site, and the guards refuse a stock file that differs.
- **Sealed firmware.** OS 1.17 images are sealed (HMAC). The kit rebuilds and reseals them, and elekloader's tests check the seal. A build the kit cannot seal is refused, never sent. Modwerk's own writer (`src/engine/elektron/ele3.ts`, used by `scripts/link-image.mjs`) refuses a sealed OS, so a local Digitakt II test image comes from the site's builder path (the kit) or from elekloader until that writer seals too. That fix belongs in a `Fix:` pull request of its own, with a test against elekloader's output.
- **One-module claims.** PAGE + MOD (macro mode), PREP MUTES with PERFORM (Mute Latch) and the perform keys (Perform Direct) are each one module's gesture. Before a second performance page arrives, agree with the owner whether the base offers a list of pages behind one key, as the Rytm base's `core_view_pages` does, rather than a key combination per module ([choose how a module hooks in](module-guides/README.md#choose-how-a-module-hooks-in)).

## Order

1. The owner reviews this plan and answers the open items, or defers them to the pull request named for each.
2. Step 1 (base): done (5 November). Item 2 is settled by it; item 5 stays open.
3. Step 2's tool fixes (`module:doctor`, captures), then the modules, one pull request each, macro mode with item 3 and 4.
4. Step 3 (downloads, the owner's switch).
5. Step 4 (link, loader, MCP), step 5 (page), each with its own plan section updated first.
6. Step 6 only on demand.
