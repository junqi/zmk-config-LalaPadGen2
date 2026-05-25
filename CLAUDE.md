# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a **ZMK firmware user-config** for the **LalaPadGen2**, a split keyboard with an IQS9151 capacitive trackpad on each half. Builds run in GitHub Actions; there is no local build toolchain assumed. Pushing to the repo triggers a firmware build that produces `.uf2` artifacts to flash onto each half.

- MCU: `seeeduino_xiao_ble` (nRF52840) on both halves
- Trackpad: Azoteq IQS9151 over I2C (one per half), each providing 1F/2F/3F tap, press-hold, scroll, pinch, and inertia
- Wireless: BLE split, with the right half as the central
- ZMK Studio is enabled (`CONFIG_ZMK_STUDIO=y`) so the keymap can be edited live over USB on the right half

## Build / flash

Firmware is built in CI via `.github/workflows/build.yml`, which delegates to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`. The matrix is defined in `build.yaml`:

- `lalapadgen2_right` + `rgbled_adapter` + `studio-rpc-usb-uart` snippet — central, USB/Studio side
- `lalapadgen2_left` + `rgbled_adapter` — peripheral
- `settings_reset` — utility firmware for clearing bonds

To rebuild, push to a branch (or run the workflow manually); download the artifact zip and flash the matching `.uf2` to each half via the Xiao bootloader (double-tap reset). The `settings_reset` firmware is flashed temporarily to wipe stored settings/bonds when re-pairing.

There is no local lint/test command — validation happens through the CI build.

## Manifest & external modules (`config/west.yml`)

The build pulls in non-trivial out-of-tree modules; changes that depend on these need to match the pinned revisions:

- `zmkfirmware/zmk` @ `v0.3.0` — base firmware (note: NOT `main`)
- `ShiniNet/zmk-driver-iqs9151` @ `main` — IQS9151 trackpad driver and gesture pipeline; all `CONFIG_INPUT_IQS9151_*` knobs come from here
- `ShiniNet/zmk-easy-charge-indicator` @ `main` — LED indicator driven by the charge IC's STAT pin
- `caksoylar/zmk-rgbled-widget` @ `v0.3` — battery / layer status on the RGB LED

When debugging a config symbol or DT compatible you can't find in ZMK proper, check these modules first.

## Code layout — what to read where

- `config/lalapadgen2.keymap` — keymap, behaviors, combos, macros, conditional layers. Four layers: `DEFAULT_LAYER (0)`, `SECONDARY_LAYER (1)`, `TERTIARY_LAYER (2)`, `SYSTEM_LAYER (3)`. Layer 3 is reached only via the conditional-layers combo of holding layers 1 + 2 (see `Cond_Syslayer`).
- `config/lalapadgen2.conf` — global Kconfig: BLE TX power, pointing/scroll, sleep timeouts, RGB widget thresholds, ZMK Studio. Trackpad sensor tuning lives **per-shield** in `lalapadgen2_left.conf` / `lalapadgen2_right.conf`, not here.
- `config/lalapadgen2.json` — physical layout metadata for editors / Studio (separate from the runtime `lalapadgen2-layouts.dtsi`).
- `config/boards/shields/lalapadgen2/` — the actual shield definition. The matrix, trackpad bindings, LEDs, and charge indicator are all defined here:
  - `lalapadgen2.dtsi` — shared base: matrix transform, kscan row GPIOs, both trackpad input-splits/listeners, input processors, LEDs, easy-charge-indicator, and the IQS9151 on `xiao_i2c`. Position constants `POS_TP_*_L` / `POS_TP_*_R` (52–67) map trackpad gesture buttons back into matrix positions that the keymap can bind to (`&mkp`, `&kp`, etc. on rows 5/6 of the keymap).
  - `lalapadgen2_left.overlay` / `lalapadgen2_right.overlay` — per-side col GPIOs and which trackpad listener/split is `okay` vs `disabled`. The right overlay sets `col-offset = <6>` on the matrix transform.
  - `lalapadgen2_left.conf` / `lalapadgen2_right.conf` — IQS9151 gesture/inertia tuning per half (currently identical, but kept separate so the halves can diverge).
  - `Kconfig.shield` / `Kconfig.defconfig` — declares the two shield symbols; the **right** half is the split central (`ZMK_SPLIT_ROLE_CENTRAL=y`, peripherals=1, `BT_MAX_CONN=5`).
  - `lalapadgen2-layouts.dtsi` — the `zmk,physical-layout` (key positions/sizes) referenced by `chosen { zmk,physical-layout }`.
- `boards/shields/` — empty placeholder; the real shield lives under `config/boards/shields/` (the `zephyr/module.yml` sets `board_root: .` so both roots are searched).

## Things easy to get wrong

- **Matrix split**: cols 0–5 belong to the left half, cols 6–11 to the right; the right overlay adds `col-offset = <6>`. When editing the keymap, the left columns appear first on each row, then the right columns — see the 12-column layout in `lalapadgen2.keymap`.
- **Trackpad positions (52–67)** are *virtual* matrix slots fed by `tp_to_pos`. Keymap rows 5 and 6 (the last two binding rows in every layer) target those slots and must be present in every layer even if `&trans`.
- **Which half owns which trackpad**: `lalapadgen2_left.overlay` enables `trackpad_split_L` and points it at `&iqs9151`; `lalapadgen2_right.overlay` enables `trackpad_listener_R` (the central side that actually consumes the input). The default `status` in `.dtsi` is `disabled` — overlays flip the correct ones to `okay`.
- **Layer 3 (SYSTEM_LAYER)** is not reachable via `&mo 3`; it's activated by holding layers 1 and 2 simultaneously (`conditional_layers` block). Don't add a direct `&mo 3` binding without understanding that interaction.
- **Custom behaviors** in the keymap: `mt2` is a stricter mod-tap with `require-prior-idle-ms`; `tap_dance_layer_1and2` taps to MO(1)/MO(2); `zip_dyn_scale[_set]` adjusts trackpad scaling at runtime (used on layer 3). Don't replace these with stock `&mt` / `&lt` without checking the call sites.
- **ZMK version pin**: this config tracks ZMK `v0.3.0`, not main. New ZMK features (behaviors, dt-bindings) may not exist here — check the v0.3 branch of `zmkfirmware/zmk` before assuming a symbol is available.

## Conversation language

Per the user's global `~/.claude/CLAUDE.md`, respond in **Japanese**.
