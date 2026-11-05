# Macro Mode

A Digitakt II module by @toonst. Version 0.1.0-experimental, for OS 1.17 on the Digitakt II base (core 1.1).

## Overview

Hold PAGE and press MOD, and encoders A–H become eight performance macros. Each macro moves up to four sound parameters, chosen from a list like the stock modulation setup's, each with its own depth and on the tracks you choose. The macros ride on top of the sound the way the stock modulators do: the module adds its offsets to every track's parameter values once a frame, before the LFOs add theirs, held in each parameter's range. The kit itself never changes, so a macro cannot be recorded or locked, and turning it back to the centre leaves the sound exactly as saved. This is especially useful when the Digitakt II plays as an MPE sound module or live, with one knob sweeping several parameters on several tracks at once.

The setup and the knobs' positions are saved inside the project, either once for the whole project or once per pattern's kit, and travel with pattern and kit copies.

## Controls

**Macro page** (hold PAGE and press MOD, or MOD and then PAGE; MOD lights blue):

| Do this | And |
| --- | --- |
| Turn encoders A–H | Move that macro, −64 … +63. A macro with no parameter yet shows the MIDI pages' mark for a free knob |
| Turn LEVEL/DATA | Move all eight macros together |
| FUNC + NO, or push LEVEL/DATA | All macros back to the centre |
| FUNC + push an encoder | Open that macro's setup |
| FUNC + push and hold an encoder | Name the macro |
| MOD, or another page key | Leave macro mode |

**Setup** (FUNC + push an encoder on the macro page):

| Do this | And |
| --- | --- |
| Encoders A–D | Choose what parameters 1–4 control, from a list like the stock modulation setup's |
| Encoders E–H | How far each one moves (its depth, positive or negative, in the stock depth format) |
| LEFT / RIGHT | Select one of the four parameters |
| Trig keys | Choose the tracks the selected parameter acts on (lit red) |
| NO or MOD | Back to the macro page |

**SETTINGS > PERSONALIZE > MACRO SETUP**: PROJECT (default) keeps one setup and one set of knob positions for the whole project. KIT gives each pattern's kit its own, which comes on with the pattern (while playing, when the new pattern starts). A kit without a setup starts as a copy of the current one; a kit whose macros you have not turned yet has its knobs at the centre. LEFT and RIGHT change the row. Switching between the two loses nothing: the project and every kit keep their own.

## Usage

Set up one macro, then play it. The screens are under [Screens and audio](#screens-and-audio).

### Quick tutorial

1. Hold PAGE and press MOD: the macro page opens and MOD lights blue.
2. Hold FUNC and push encoder A: the setup of macro A opens.
3. Turn encoder A to choose FLTR FREQ, and encoder E to set its depth, for example +64.00.
4. Press the trig keys of the tracks it should act on; they light red.
5. Press NO, then turn encoder A: the filter opens and closes on those tracks, and the kit stays as saved.

Macro mode works as a page: the keys it does not use keep their stock function, and menus, popups and mute mode come over the macro page while the mode stays on. TRK + trig keeps the mode.

## Compatibility and limitations

- **Needs the Digitakt II base with core 1.1**: `ev_frame_params` for the per-frame values, `ev_key_leds` for its LEDs, `ev_personalize` and `core_addvalue` for the MACRO SETUP row, and `core_projdata` and `core_kitdata` for the setups. It does not build on core 1.0, so it waits for the base ([plan](../../../../docs/DIGITAKT_II.md)).
- **Takes PAGE + MOD and the macro page** (`ui:PAGE + MOD`, `ui:macro mode`) and the PERSONALIZE row MACRO SETUP.
- **Kit data.** With MACRO SETUP on KIT, each kit's setup takes all 191 bytes of kit data a Digitakt II kit record has (claim `kitdata:MMK2`). No other module that keeps kit data fits beside it. The project's setup takes 192 of the 484 bytes of the project header's gap (`projdata:MMP2`).
- **Card.** Its two settings (the setup on, the MACRO SETUP choice) are kept in two card sectors the stock OS does not write (`card:0x2df0e00-0x2df0e02`).
- **Projects stay safe.** A project saved with macro mode still opens on the stock OS, without macros, and the stock OS keeps the setups when it saves it. A project from the stock OS opens with its kits without setups. If something else has already put data in those bytes, macro mode leaves it alone and shows MACRO SETUP CONFLICT.
- **Not offered as destinations yet:** machine-specific parameters and the LFOs.
- **Stock flows changed:** PAGE + MOD opens the macro page, where stock does nothing with that combination. Everything else reaches the stock OS as before.
- Projects saved with digi_macro builds before 0.1 open, but their macro setups are not read.

## Tests and measurements

See [TESTING.md](TESTING.md). In short: this folder's build, linked on core-dt2 1.1 through the site's builder path (a local build, not the pinned toolchain), passed digi_macro's digikit emulator scenarios on OS 1.17: 94, 32 and 12 checks. Its author reports the module running on a Digitakt II with an earlier core. It has not run on a unit from this folder.

## Authorship and licences

Macro mode is toonst's, from [toonst/digi_macro](https://github.com/toonst/digi_macro) at `bc2295e`, under GPL-2.0-or-later ([LICENSE](LICENSE)); its own README is in [upstream/](upstream/README.md). It runs on the Digitakt II base, which starts from irpina's core-dt2 (elekloader, GPL-2.0-or-later) with toonst's 1.1 additions. The firmware locations in `src/mm_os.h` are addresses only. This folder holds no firmware bytes.

## Screens and audio

![The macro page](media/macro-page.png)

![The setup of macro C](media/macro-setup.png)

Both are captures of the build TESTING.md describes, in digikit's emulator ([capture record](media/capture.json)). The module makes no sound of its own: it moves the parameters of the stock sound.
