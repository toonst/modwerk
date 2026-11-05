# Digitakt II SDK

Modwerk's Digitakt II path is experimental and targets stock OS 1.17 only. The site builds every Digitakt II set on Modwerk's own base, core-dt2 1.1's source built by `npm run machine:base`, with elekloader kit 0.7.0. The machine remains in research: builds can be checked locally, but downloads stay disabled until the exact Modwerk packages are qualified and reviewed.

## Modwerk stack

The Digitakt II is getting the Octatrack's stack ([recipe](../../../docs/MACHINE_RECIPE.md)): a Modwerk base that takes the hooks mods propose, then a USB link, the loader's glue, base updates over USB, the MCP's device tools and the machine page.

The base is built (5 November 2026): core-dt2 1.1's source (elekloader#221, `0df8409`), copied unchanged into [elekloader/](elekloader/README.md) with its provenance in `sdk/imports/`, built by `npm run machine:base -- digitakt-ii` with the reviewed toolchain and identified in [base-binding.json](base-binding.json) (`f41b89cd`). It keeps everything core 1.0 (toonst's digitakt2-perform 1.0, the core the site built with before) had, and adds what a performance mode needs: per-frame parameter values, key LEDs, per-project and per-kit data and value rows. The USB link, the loader, base updates, the device tools and the machine page are not started. The steps from here, in order, and the decisions they need are in the [Digitakt II plan](../../../docs/DIGITAKT_II.md).

## Core hooks

The core is Modwerk's base, core-dt2 1.1 (GPL-2.0-or-later; irpina's core with toonst's 1.1 additions), pinned by hash in [sdk/imports/elekloader-modwerk-base-digitakt-ii-2026-11-05.json](../../imports/elekloader-modwerk-base-digitakt-ii-2026-11-05.json); its contract is [core-profile/interface.json](../../digitakt-ii/core-profile/interface.json). It exposes the UI hooks below and nothing of the sequencer, the LFOs or the voices. A mod that needs one of those sites proposes a core hook first and never patches the site itself ([propose a hook](../../../docs/module-guides/README.md#propose-a-hook-on-the-digitakt-digitone-or-digitakt-ii)); the hook goes into the Digitakt II's Modwerk base ([Modwerk stack](#modwerk-stack)).

The core subscribes mods to these ColdFire/UI hooks:

| Hook | Handler | Use |
| --- | --- | --- |
| `ev_tick` | `void f(void *ctrl)` | UI compose check, approximately 30 Hz. Set `ctrl + 0x20` to 1 to request redraw. This is not an audio-rate clock. |
| `ev_draw` | `void f(void *bmp, void *ctrl)` | Draw after the OS composes its 128x64 screen. |
| `ev_key` | `int f(void *brain, void *event)` | Inspect or consume a key event; nonzero consumes it. |
| `ev_enc` | `int f(void *brain, void *event)` | Inspect or consume an encoder event; nonzero consumes it. |
| `ev_settings` | `void f(void *menu)` | Add rows while the SETTINGS menu is built. |
| `ev_personalize` | `void f(void *menu)` | Add rows while PERSONALIZE is built. |

### Added in core 1.1

These are in [core-profile/interface.json](../../digitakt-ii/core-profile/interface.json) marked `since` 1.1. A module that uses one sets `platform.core` to `"1.1"`.

| Entry | What | Handler or record |
| --- | --- | --- |
| `ev_frame_params` | each frame's per-track parameter values, once a frame in the frame interrupt, before the stock modulators add theirs | `void f(int16_t *values, int stride)`; track t's value j at `values[stride * t + j]`, stride 71; add to it, held in the parameter's range |
| `ev_key_leds` | the key-LED frame, every UI frame, after the frame clears the LEDs and before the stock views set theirs | `void f(void *mgr)`; set an LED with `fw_led_set(mgr, logical, 2, value)`; the first setter wins |
| `core_projdata` | a mod's block in the project file's header gap, saved with every project save and loaded back | table of `core_projdata_t {tag[4], size, data, loaded}`, four entries; claim `projdata:<tag>` |
| `core_kitdata` | a mod's block per kit (128), after each kit record's end mark; saved with the project and SAVE KIT and carried through pattern copy and paste, PASTE KIT and undo | table of `core_kitdata_t {tag[4], size, data, changed}`, four entries; claim `kitdata:<tag>` |
| `core_addvalue`, `core_kit_on`, `fw_led_set` | a value row for PERSONALIZE or SETTINGS drawn like LED INTENSITY, the kit on, and the LED setter for `ev_key_leds` | `digitakt-mk2/core.h` |

By the core's documentation, both data areas hold bytes the stock OS writes as zeros and never reads, so a project saved with mods loads on the stock OS, which keeps them when it saves a project it loaded. Core writes them only over zeros or its own data and counts anything else in `core_store_conflicts`. **A kit record has room for 191 bytes of kit data in all** (`core_kitdata_room()`), shared by every mod's block. One mod with a 191-byte block takes all of it and leaves any other kit-data mod out, so per-kit state that several modules will want belongs to the base, as one record per module in a table the base owns ([recipe](../../../docs/MACHINE_RECIPE.md#rules)), not to the first module that asks for the room.

The Digitakt II core does not start DTIM0 and exposes no audio-render hook. Audio is rendered by the separate SHARC DSP; these hooks are not sample-accurate.

## Memory

- Shared DDR: `0x47F00000–0x47F40000` (262,144 bytes for core and linked mods).
- Fast-code SRAM: `0x8000F100–0x80010000` (3,840 bytes). The core copies `.fast` code on its first UI tick.
- The base's footprint: 3,708 bytes `.run` plus 1,808 bytes `.bss` (`f41b89cd`); the linker remains authoritative for combined sizes and overlap.
- Each hook collection and table has four entries. `core_fast` has sixteen copy-table entries.

## Build and recovery

Use the exact owner-supplied `Digitakt_II_OS1.17.syx`; the builder recognizes firmware by hash and keeps the bootstrap and updater sections unchanged. The toolchain is m68k-elf GCC 16.2.0 and binutils 2.47 (`sdk/build/Dockerfile`, or Homebrew's `m68k-elf-gcc` at those versions; `machine:base` refuses any other). Rebuild and check the base with:

```sh
npm run upstream:tools
npm run machine:base -- digitakt-ii --stock /private/Digitakt_II_OS1.17.syx
```

The site's builder path (the vendored kit) builds and seals a set. Modwerk's own writer behind `scripts/link-image.mjs` does not seal a Digitakt II OS yet, so a local test image comes from the site or from elekloader.

A normal OS update uses Elektron Transfer. For recovery, hold FUNC while powering on, choose TRIG 4 (OS UPGRADE), then send the owner's stock `.syx` over MIDI DIN; this recovery mode does not use USB. Local parsing and emulator checks do not guarantee hardware safety. Do not flash an unqualified build.

## Status

The base builds with the reviewed toolchain and passes its check against core 1.0 (`machine:base`). Core 1.1's source with macro mode passed digi_macro's digikit emulator scenarios and, by its author's report, works on a Digitakt II (5 November 2026); the base's own package has not run on a unit yet. No module in this SDK is approved or qualified for publication yet.
