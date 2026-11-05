# Macro Mode testing

## This folder's build

Not built or run yet. It needs the Digitakt II base with core 1.1, which Modwerk does not build yet ([plan](../../../../docs/DIGITAKT_II.md), step 1), and the base's header `digitakt-mk2/core.h`, which the plan's pull request brings (core-dt2 1.1 as elekloader#221 now has it, `0df8409`). Once it does, this folder's package is built in the pinned toolchain and the results below are recorded for that exact build.

| Check | Result |
| --- | --- |
| Source-only package, pinned toolchain (`scripts/build-elemod-isolated.sh`) | Not run |
| Linked with the base on stock OS 1.17 by the site's builder | Not run |
| digikit emulator: boot, the macro page, the setup, a project round trip | Not run |
| Unit: the same build on a Digitakt II, OS 1.17 | Not run |
| Memory and frame-interrupt load | Not measured |

What was checked here (5 November 2026): the sources are digi_macro's at `bc2295e`. With m68k-linux-gnu GCC 13.3 they compile, for the Digitakt II with the kits feature, to 19,517 bytes of text and 33,575 of bss with no undefined symbol but core's. Every object's text, disassembly and relocations are identical to those of the commit before (`ac5c237`), whose manifest passed the firmware locations as compiler flags. That is not the reviewed toolchain, and nothing ran.

Modwerk's own source builder (`scripts/build-elemod-packages.mjs --module digitakt-ii:macro-mode`, the same GCC 13.3, in a local run with the machine's core version set to 1.1 and the base's `digitakt-mk2/core.h` from the plan's branch, neither committed) compiles the folder for OS 1.17: `.run` 19,517 bytes, `.bss` 33,575, together 53,092 of the 262,144-byte shared mod area. It imports only `core_addvalue`, `core_kit_on` and `core_store_conflicts`, all core 1.1 exports, and patches no stock site. With the profile as committed (core 1.0) the contract refuses it, as it should until the base ships.

## Upstream results, as reported

From digi_macro's [STATUS.md](https://github.com/toonst/digi_macro/blob/bc2295e057e6dae17fa44d3ed3a2575c8e848935/docs/STATUS.md), on elekloader builds of this source before bc2295e's header change, with the earlier core-dt2 1.1 (`2b0616e`, before elekloader#221 was rebased and cut to `0df8409`); not on this folder's build or on `0df8409`:

- **Host tests** (`tools/check.sh`): mode logic, the macro math and its range limits, the storage format and the screens, 659 checks, all passing (run again here on `bc2295e`).
- **digikit emulator, Digitakt II OS 1.17**: `tests/emu/scenario.py` 94 checks, `tests/emu/project.py` 32 and `tests/emu/compat.py` 12, all passing, by the author's report. They cover entry and exit, the input map, the per-frame values held in range, the LEDs, the setups per project and per kit, pattern copy and paste carrying them, and stock-OS round trips of the project file.
- **Unit**: the author reports the module running on a Digitakt II on OS 1.17, including macro mode as a page under mute mode and the pattern popup, TEMPO, and the knob positions coming back after a restart. That is the author's report on their own build, not this folder's.

Not tested anywhere: KIT MANAGER's PASTE KIT and undo with per-kit setups, audio output in the emulator (its runner makes no sound), and the frame interrupt's load with all eight macros moving.
