# Macro Mode testing

## This folder's build

Not built or run yet. It needs the Digitakt II base with core 1.1, which Modwerk does not build yet ([plan](../../../../docs/DIGITAKT_II.md), step 1). Once it does, this folder's package is built in the pinned toolchain and the results below are recorded for that exact build.

| Check | Result |
| --- | --- |
| Source-only package, pinned toolchain (`scripts/build-elemod-isolated.sh`) | Not run |
| Linked with the base on stock OS 1.17 by the site's builder | Not run |
| digikit emulator: boot, the macro page, the setup, a project round trip | Not run |
| Unit: the same build on a Digitakt II, OS 1.17 | Not run |
| Memory and frame-interrupt load | Not measured |

What was checked here (5 November 2026): the sources are digi_macro's at `89abd8e`. With m68k-linux-gnu GCC 13.3 they compile, for the Digitakt II with the kits feature, to 19,517 bytes of text and 33,575 of bss with no undefined symbol but core's. Every object's text, disassembly and relocations are identical to those of the commit before (`ac5c237`), whose manifest passed the firmware locations as compiler flags. That is not the reviewed toolchain, and nothing ran.

Modwerk's own source builder (`scripts/build-elemod-packages.mjs --module digitakt-ii:macro-mode`, the same GCC 13.3, in a local run with the machine's core version set to 1.1 and not committed) compiles the folder for OS 1.17: `.run` 19,517 bytes, `.bss` 33,575, together 53,092 of the 262,144-byte shared mod area. It imports only `core_addvalue`, `core_kit_on` and `core_store_conflicts`, all core 1.1 exports, and patches no stock site. With the profile as committed (core 1.0) the contract refuses it, as it should until the base ships.

## Upstream results, as reported

From digi_macro's [STATUS.md](https://github.com/toonst/digi_macro/blob/89abd8ebf9944a15e58124c50f7a34b892061e38/docs/STATUS.md), on elekloader builds of the same source with core-dt2 1.1 (toonst's elekloader fork, `2b0616e`), not on this folder's build:

- **Host tests** (`tools/check.sh`): mode logic, the macro math and its range limits, the storage format and the screens, 659 checks, all passing (run again here on `89abd8e`).
- **digikit emulator, Digitakt II OS 1.17**: `tests/emu/scenario.py` 94 checks, `tests/emu/project.py` 32 and `tests/emu/compat.py` 12, all passing, by the author's report. They cover entry and exit, the input map, the per-frame values held in range, the LEDs, the setups per project and per kit, pattern copy and paste carrying them, and stock-OS round trips of the project file.
- **Unit**: the author reports the module running on a Digitakt II on OS 1.17, including macro mode as a page under mute mode and the pattern popup, TEMPO, and the knob positions coming back after a restart. That is the author's report on their own build, not this folder's.

Not tested anywhere: KIT MANAGER's PASTE KIT and undo with per-kit setups, audio output in the emulator (its runner makes no sound), and the frame interrupt's load with all eight macros moving.
