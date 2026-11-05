# Digitakt II base

Modwerk's own base for the Digitakt II on OS 1.17 ([recipe](../../../../docs/MACHINE_RECIPE.md)). Today it is the
source of the Digitakt II core 1.1, copied unchanged: `core-dt2/` (its `mod.json` and eight sources) and `core/`
(the shared `core.s`, `settings.s` and `fast.s`, which `mod.json` builds with the Digitakt II's addresses), and the
header `../include/digitakt-mk2/core.h` that describes its additions to mods. Provenance and hashes:
[sdk/imports/core-dt2-source-0df8409.json](../../../imports/core-dt2-source-0df8409.json). The files are
GPL-2.0-or-later and keep their SPDX lines. The core is irpina's, from elekloader; toonst wrote the 1.1 additions,
proposed to elekloader as [elekloader#221](https://github.com/elekloader/elekloader/pull/221) (commit `0df8409`,
rebased on elekloader's main, no release tag yet). Modwerk's own sources go beside these, so the imported
files stay exact.

Core 1.1 keeps the six events and every site of core 1.0 and adds, in new code after the old:

- `ev_frame_params`: each frame's per-track parameter values, before the stock modulators;
- `ev_key_leds`: the key-LED frame, before the stock views set theirs;
- `core_projdata` and `core_kitdata`: a mod's block in the project file's header gap and in each kit record, saved
  and loaded with the project, carried through the firmware's kit copies, and written only over zeros or core's own
  data;
- `core_addvalue`, a value row for PERSONALIZE or SETTINGS, and `fw_led_set` for `ev_key_leds` handlers.

The entries are in [core-profile/interface.json](../../../digitakt-ii/core-profile/interface.json) (marked `since`
1.1), with each new site's address, length and SHA-256 guard.

## Build it

[base.json](base.json) names the core folder, the SDK headers (`../include`), the OS and the reviewed toolchain
(m68k-elf GCC 16.2.0, binutils 2.47). The SDK is the kit-pinned Elekloader checkout. The build needs your own original
`Digitakt_II_OS1.17.syx`, which stays outside the repository:

```sh
npm run upstream:tools
npm run machine:base -- digitakt-ii --stock /private/Digitakt_II_OS1.17.syx
```

It builds in a temporary folder outside Git and checks two things:

- **Against core-dt2 1.0's release** (`vendor/elekloader/catalog/core-1.0.elemod`, toonst's digitakt2-perform v1.0, the
  core the site built with before): every site, export, import, table and contribution 1.0 has (`reference.keeps`).
  Not its version, and not its sources: 1.0 was built from an older `core.s`, `settings.s` and `personalize.s`.
- **Against [base-binding.json](../base-binding.json):** the SHA-256 of the package (`f41b89cd`, one layout: no
  `add` and no variants yet).

`--out NEW-DIR` (outside Git) keeps the `.elemod` under the name the site's catalogue serves it by,
`modwerk-base-digitakt-ii-<first 8 of its identity>.elemod`; it is published in
`sdk/imports/packages/modwerk-base-digitakt-ii/` and pinned in
`sdk/imports/elekloader-modwerk-base-digitakt-ii-2026-11-05.json`. `--write` records a new identity: use it only for a
deliberate change, and say why in the pull request.

| Build | `.run` bytes | `.bss` bytes | Sites |
|---|---|---|---|
| toonst's release `core-1.0.elemod` | 480 | 20 | 7 |
| m68k-elf GCC 16.2.0, binutils 2.47 (reviewed; `f41b89cd`) | 3,708 | 1,808 | 14 |
| Ubuntu 24.04 m68k-linux-gnu GCC 13.3.0 (not recorded) | 3,200 | 1,808 | 14 |

## What is not here yet

- **Not elekloader's release yet.** `0df8409` is elekloader#221's branch, not merged. If it lands changed,
  elekloader's core-dt2 1.1 and this source differ: take the source again from its merged release and rebuild
  the base. Changes to core-dt2 itself go to elekloader ([contributing](../../../../CONTRIBUTING.md#elekloader-mods-and-cores)).
- **Hardware.** This copy's source (`0df8409`), built by elekloader with macro mode, passed digi_macro's digikit
  emulator scenarios (5 November 2026) and, by toonst's report, works on a Digitakt II on OS 1.17; a unit has run core
  1.0 with a mod on its key events. The base's own package (`f41b89cd`) has not run on a unit yet, and Modwerk has
  qualified none of it on hardware.
