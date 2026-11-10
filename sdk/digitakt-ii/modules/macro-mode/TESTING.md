# Macro Mode testing

Version 0.1.0-experimental, for Digitakt II OS 1.17 on the Digitakt II base (core 1.1).

## The build

| What | Identity |
| --- | --- |
| Source | digi_macro `62f0b0a`, as in `src/`; identity `19b03c9d…` ([evidence/source-identity.json](evidence/source-identity.json)) |
| Package | `macro-mode-0.1.0-experimental-os1.17.elemod`, SHA-256 `5791a094…`, compiled from this folder by `scripts/build-elemod-packages.mjs` with m68k-elf GCC 16.2.0 and binutils 2.47 (the versions of `sdk/build/Dockerfile`, built from GNU's release archives) |
| Base | `modwerk-base-digitakt-ii-ce3e5488.elemod`, as `sdk/machines/digitakt-ii/base-binding.json` names it |
| Image | Base and package linked on the contributor's stock `Digitakt_II_OS1.17.syx` (`26c22f66…`) by the site's builder path (`prepareBuild`, `buildWithLock`, the vendored kit 0.7.0), version `MW01`: SHA-256 `6be06fd9…`. Outside Git |

The package's `.run` is 19,793 bytes and its `.bss` 8,275: 28,068 bytes of the shared mod area. It imports `core_kit_on` and `core_store_conflicts`, and patches no stock site.

## Results

| Check | Result |
| --- | --- |
| Image checks | Passed: the site's preflight (every section but the main OS stock, every changed byte inside a declared site) and elekloader's verify (the seal, the stock bootloader, the main image). Elekloader's own linker gives the same main OS, byte for byte |
| Emulator: the project's setup, save and load | Passed, 23 of 23 (`tests/emu/project_only.py`) |
| Emulator: round trips through the stock OS, a conflict | Passed, 13 of 13 (`tests/emu/compat_project.py`) |
| Emulator: the macro page, the setup, naming, frame values, LEDs | 91 of 94 (`tests/emu/scenario.py`); the three are its restart checks, see below |
| Emulator: CREATE NEW | The setup and the knob positions that were on stay |
| Emulator: with Perform Direct and Perform Levels | Builds and passes the same image checks (`5f2ae3ba…`); boots to the main screen and the macro page opens |
| Emulator: frame interrupt load | Counted, below |
| Unit, this image | Not tested |
| Unit, a power cycle without saving the project | Not tested |
| Audio | Not tested: the emulator's runner makes no sound |
| KIT MANAGER, pattern copy and paste | Not tested with this build; it keeps no kit data, so they have nothing of its to carry |

All emulator runs are in digikit's `zero_patched` runner (digikit `3d3a243`, the ColdFire with the DSP side answering zeros, so the frame interrupt runs) on 10 November 2026, with digi_macro's scenarios at `8b67398` and a symbol map from elekloader's link of the same two packages. The summary is in [evidence/emulator-2026-11-10.json](evidence/emulator-2026-11-10.json).

### What the scenarios cover

- **`project_only.py`:** the build has no kit data and no MACRO SETUP row; PAGE + MOD opens the macro page; a turned macro moves its destination's frame value on its tracks while the kit keeps its own; another pattern's kit changes nothing; SAVE PROJECT AS writes the setup and the knobs into the project header's gap and nothing after the kit records or into the card sectors digi_macro's own build uses; LOAD PROJECT brings both back.
- **`compat_project.py`:** the stock OS loads the module's project and saves it again, and the two files differ only in the header's gap, where the stock save keeps the setup; the module then loads the stock save with no conflict; with another tool's bytes in the gap the base counts a conflict, the module takes no setup from them, and a save leaves them as they were.
- **`scenario.py`:** entry and exit (PAGE + MOD, MOD + PAGE, MOD, the page keys), the input map, the setup screen and naming, depths, the per-frame values held in range on the chosen tracks, the LEDs, and macro mode under SETTINGS, TEMPO, the pattern and preset popups and mute mode. Its last three checks restart the emulator on a card with only the two sectors digi_macro's own build keeps its setup in. This build writes none (the project feature), so the restart starts without a setup and the three fail as expected. The emulator cannot restart on a card with a saved project, so whether an unsaved setup survives a power cycle is a question for a unit.

### Frame interrupt load

`tests/emu/frame_load.py` counts instructions between the frame interrupt's first instruction (exception vector 191) and its return, and between the module's hook and its return, over 200 frames each. Instruction counts in the emulator are not time on a unit: they say how the work grows, not how much headroom is left.

| State | Frame interrupt, median | The module's hook |
| --- | --- | --- |
| Stock OS, stopped | 44,355 | — |
| Stock OS, playing | 44,291 | — |
| Base and module, no macro set up | 44,821 | 4 |
| One macro, four destinations, sixteen tracks | 47,161 | 1,831 |
| Eight macros, four destinations each, sixteen tracks | 58,417 | 12,631 |
| The same, playing | 58,413 | 12,631 |

The heaviest setup the module holds (32 destinations on all 16 tracks, every knob turned) adds 12,631 instructions a frame, 28% of the stock interrupt's count. The base without any module adds 462. [evidence/frame-load.json](evidence/frame-load.json) has every figure.

## Earlier results, on other builds

- **digi_macro's own build** (`tools/build.sh`, per-kit setups, core-dt2 1.1 at elekloader `0df8409`, image `98f9d975…`): toonst reports it working on a Digitakt II, OS 1.17, on 5 November 2026, including macro mode under mute mode and the pattern popup, TEMPO, and the knob positions coming back after a restart. That build keeps its setup in kit and project data and two card sectors; this one in project data alone, on Modwerk's base.
- **Host tests** (`tools/check.sh` in digi_macro): mode logic, the macro math and its range limits, the storage format and the screens, passing at `8b67398`.
