# Feature: Dongle 1.3" I2C OLED status display

**Feature ID**: dongle-sh1106-display
**Status**: in progress
**Branch**: feat/dongle-sh1106-display
**Route**: delegated direct (writer) — trigger: 3+ non-trivial files

## Objective

Add a 1.3" I2C OLED status display to the `nice_nano_v2` Charybdis dongle,
using the `englmaxi/zmk-dongle-display` module for the custom status screen
(layer name, modifiers, peripheral battery, output, bongo cat).

## Problem

The dongle currently exposes no display. `charybdis_dongle.overlay` only wires a
mock kscan; there is no `zephyr,display` chosen node, no I2C bus, and
`config/west.yml` does not declare the display module.

## Why

User request: add a 1.3" IIC display module to the dongle based on
https://github.com/englmaxi/zmk-dongle-display.

## Verified facts (from primary sources)

- Repo pins ZMK `v0.3`, which uses Zephyr `v3.5.0+zmk-fixes`; that Zephyr
  ships the SH1106 driver (`drivers/display/ssd1306.c`, binding
  `sinowealth,sh1106`, `Kconfig.ssd1306` `default y` on the DT compat).
- 1.3" 128x64 I2C OLED uses the SH1106 controller and needs
  `segment-offset = <2>` (module README + Zephyr canonical overlay).
- `nice_nano_v2` interconnect exposes `pro_micro_i2c: &i2c0` with
  SDA=P0.17 (D2), SCL=P0.20 (D3) (`arduino_pro_micro_pins.dtsi`,
  `nice_nano-pinctrl.dtsi`).
- The dongle does **not** include `charybdis_3610.dtsi`, so P0.17/P0.20 are
  free (no SPI0 on the central dongle; i2c0/spi0 share the same nRF52840
  peripheral, so this matters).
- `zmk-dongle-display` provides shield `dongle_display`: empty `.overlay`,
  `dongle_display.conf` (`CONFIG_ZMK_DISPLAY=y`, battery fetching, dedicated
  thread stack), and `Kconfig.defconfig` that selects the custom status screen,
  LVGL widgets, `ZMK_WPM`, and `ZMK_HID_INDICATORS`.
- Module has tags `v0.3` and `v0.3.0`.
- Shield `requires:` (e.g. `i2c_oled`) is parsed only by
  `app/scripts/west_commands/metadata.py`; **not** validated by `west build`.

## Scope

In scope:
- Declare the module in `config/west.yml` (`remote: englmaxi`, `revision: v0.3`).
- Add `zephyr,display = &oled` + SH1106 node on `&pro_micro_i2c` to the dongle overlay.
- Enable the display in the dongle `.conf`.
- Add `dongle_display` to the dongle entry in `build.yaml`.
- Update `docs/pinout.md` with the display wiring.

Out of scope:
- No changes to left/right shields, keymap, matrix, or trackball.
- No changes to `config/info.json`.
- No push / PR / merge (user decision).

## Constraints

- ZMK must stay pinned to `v0.3`; the module revision must be `v0.3`.
- Multiple shields (`charybdis_dongle dongle_display`) are combined in one build.
- No local west/Zephyr toolchain available → CI is the compile gate.

## Tasks

- [x] T1 — `config/west.yml`: add `englmaxi` remote + `zmk-dongle-display`
      project at `revision: v0.3`. Check: YAML parses; manifest has 3 projects.
- [x] T2 — `boards/shields/charybdis/charybdis_dongle.overlay`: add
      `zephyr,display = &oled;` to chosen and the SH1106 node on
      `&pro_micro_i2c`. Check: node props match Zephyr 3.5 sh1106 binding.
- [x] T3 — `boards/shields/charybdis/charybdis_dongle.conf`: enable
      `CONFIG_ZMK_DISPLAY` and idle timeout. Check: keys present.
- [x] T4 — `build.yaml`: dongle shield becomes `charybdis_dongle dongle_display`.
      Check: YAML parses; shield order base-first.
- [x] T5 — `docs/pinout.md`: document display I2C wiring + module. Check: links
      and pin table consistent with T2.
- [x] T6 — Work-unit commit on `feat/dongle-sh1106-display`; record hash.

## Acceptance criteria

- Manifest declares the module at a revision compatible with ZMK `v0.3`.
- Dongle overlay defines an SH1106 node at `0x3c` on `&pro_micro_i2c` and pins it
  as `zephyr,display`.
- `build.yaml` dongle target builds `charybdis_dongle dongle_display`.
- Documentation matches the overlay.

## Delivery forecast

~60 authored changed lines, 5 files → single PR, under the ~400-line budget.
Strategy: `ask-on-risk` (default).

## Progress / evidence

- 2026-10-13: Feature created. Research complete (see Verified facts).
- T1..T5: implemented by delegated writer. Structural checks: `YAML OK` for
  `config/west.yml` + `build.yaml`; grep assertions on overlay/conf/build.yaml
  pass; diff limited to the 5 authorized files (`75 insertions(+), 3 deletions(-)`
  before the docs index fix).
- Independent read-only verifier: **PASS-WITH-CONCERNS**. All 10 claims
  CONFIRMED against ZMK v0.3 / Zephyr v3.5.0 / module tag `v0.3`. One real
  documentation defect found and fixed inline: the I2C table used nRF port
  numbers as `pro_micro` indices (`pro_micro 17/20`); corrected to
  `&pro_micro 2` (D2) / `&pro_micro 3` (D3).
- Native risk assessment (`gentle-ai review assess`): **medium**, `review_due:
  false` (`under_budget`). RDD mode is **off** (global).
- T6: work-unit commits on `feat/dongle-sh1106-display`:
  - `3881713` docs(odd): add dongle-sh1106-display feature task
  - `9a940ee` feat(dongle): add 1.3" I2C SH1106 status display

## Outstanding / not verified

- **No local west/Zephyr toolchain**: `west build` was not run. CI
  (`zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`) is the
  compile gate and has not run.
- Module `v0.3` → ZMK `v0.3` Kconfig/LVGL symbol compatibility is unverified
  locally (e.g. `LV_USE_ANIMIMG`, `LV_FONT_UNSCII_8`, `CUSTOM_STATUS_SCREEN`).
- Physical panel orientation and whether `inversion-on` is needed are
  hardware-only checks. If colors appear inverted, remove `inversion-on`.
- Pre-existing doc issue (out of scope, not fixed): `docs/pinout.md` section 1
  labels matrix pins with nRF port numbers as `pro_micro` indices (e.g. COL 0
  `pro_micro 19` claimed as P0.19, but `&pro_micro 19` = P0.02). Header still
  cites branch `master` / commit `cce97e6`.

## Next step

Push `feat/dongle-sh1106-display` (or merge to `main`) and let CI compile the
dongle target; then flash and confirm the panel renders. Push/PR/merge is the
user's decision.
