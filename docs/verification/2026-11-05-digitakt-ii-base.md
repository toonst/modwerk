# Digitakt II: Modwerk's base on core-dt2 1.1 — 5 November 2026

The site builds every Digitakt II set on Modwerk's own base: core-dt2 1.1's source as elekloader#221 has it (`0df8409`, `sdk/imports/core-dt2-source-0df8409.json`), with no Modwerk sources added yet. It replaces toonst's core 1.0 record as the core a set links with; that record stays as the release the base is checked against ([plan](../DIGITAKT_II.md), step 1).

- **Build:** toonst ran `npm run machine:base -- digitakt-ii --stock … --write` on their Mac, with m68k-elf GCC 16.2.0 and binutils 2.47 (the reviewed versions), the kit-pinned Elekloader `3b31ef3` and their own `Digitakt_II_OS1.17.syx` (`26c22f66…`).
- **Identity:** `f41b89cd…fe88` ([base-binding.json](../../sdk/machines/digitakt-ii/base-binding.json)). It has `.boot` 50, `.run` 3,708 and `.bss` 1,808 bytes, and 14 sites.
- **Package:** `modwerk-base-digitakt-ii-f41b89cd.elemod` is committed at `95e3d674` and pinned in `sdk/imports/elekloader-modwerk-base-digitakt-ii-2026-11-05.json`. Its SHA-256 is the binding's identity, and `npm run elekloader:check` passes with it in the catalogue.
- **Reference check:** `machine:base` checks the base against `vendor/elekloader/catalog/core-1.0.elemod` (`95303655…`), and the base keeps every site, export, import, table and contribution of it. A trial build in a scratch copy with Ubuntu's m68k-linux-gnu GCC 13.3.0 (not recorded) passed the same check, at `.run` 3,200.
- **Lint:** `elekloader.lint` (elekloader `0df8409`) passes on the package with the owner's OS. Macro mode links on it (58,712 of 262,144 bytes of RAM), and so do macro mode with Perform Direct and Perform Levels (58,936).
- **The site's builder path:** `prepareBuild` and `buildWithLock` on the vendored kit 0.7.0, with the owner's OS, plan the base and build four sets:
  - the base alone;
  - macro mode, packaged from a local GCC 13.3 build of its Modwerk folder;
  - macro mode with both perform mods;
  - the two perform mods.

  Each image passes elekloader's `syx.verify`: the seal, the stock bootloader and the main image.
- **Emulator:** elekloader's link of the base with macro mode (digi_macro `bc2295e`) has the same main OS, byte for byte, as the site builder's. That image ran in digikit's `zero_patched` runner (digikit `3d3a243`), and digi_macro's scenarios passed with no failures:

  | Scenario | Checks |
  |---|---|
  | `tests/emu/scenario.py` | 94 |
  | `tests/emu/project.py` | 32 |
  | `tests/emu/compat.py` | 12 |

  They cover the per-frame values, the key LEDs, the value row, project and kit data through save, load, pattern copy and paste, and stock-OS round trips. The runner makes no sound.
- **On a unit:** core-dt2 1.1 from elekloader's own build of the same source, with macro mode, works on a Digitakt II on OS 1.17, as toonst reported the same day. The base's package itself has not been flashed. Neither the stock OS restore after a base build, nor PASTE KIT and undo with kit data, has been tried.
- **Not here:** the variants (plain and kitdata) and a `Kit data` budget, which are open items 5 and 3 of the plan. Downloads stay off.
