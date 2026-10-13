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

- [ ] T1 — `config/west.yml`: add `englmaxi` remote + `zmk-dongle-display`
      project at `revision: v0.3`. Check: YAML parses; manifest has 3 projects.
- [ ] T2 — `boards/shields/charybdis/charybdis_dongle.overlay`: add
      `zephyr,display = &oled;` to chosen and the SH1106 node on
      `&pro_micro_i2c`. Check: node props match Zephyr 3.5 sh1106 binding.
- [ ] T3 — `boards/shields/charybdis/charybdis_dongle.conf`: enable
      `CONFIG_ZMK_DISPLAY` and idle timeout. Check: keys present.
- [ ] T4 — `build.yaml`: dongle shield becomes `charybdis_dongle dongle_display`.
      Check: YAML parses; shield order base-first.
- [ ] T5 — `docs/pinout.md`: document display I2C wiring + module. Check: links
      and pin table consistent with T2.
- [ ] T6 — Work-unit commit on `feat/dongle-sh1106-display`; record hash.

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
- T1..T6: pending.

## Next step

Delegate T1–T5 to one bounded writer, then verify and commit T6.
