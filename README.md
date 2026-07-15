# ZMK Configuration Template
Wireless Cyboard keyboard configuration repository template for using ZMK firmware. [Instructions for use are located on our documentation site](https://docs.cyboard.digital/user-manual/quick-start/configure-layout).

## ZMK version
This template pins the latest **stable** ZMK release (currently `v0.3.0`), which matches the firmware stack behind [studio.cyboard.digital](https://studio.cyboard.digital). If you want to track current ZMK `main` (Zephyr 4.1) instead, edit `config/west.yml`: set the `zmk` revision to `main` and the `zmk-keyboards` revision to `zephyr-4.1`.

## ZMK Studio support

The left-half firmware built from this template has [ZMK Studio](https://studio.cyboard.digital) enabled (see the `imprint_left` entry in `build.yaml`), so you can edit your keymap live over USB without reflashing. To unlock the keyboard for Studio, **hold the A- and F-position keys (left home row) for 3 seconds**. The unlock combo is tied to the physical key locations, so it keeps working no matter how you remap your keymap.

Keymap changes made in Studio are stored in the keyboard's flash, separately from the `.keymap` file in this repo: they survive reflashes of firmware built from this repo, and the `.keymap` file only provides the defaults Studio starts from (or falls back to after a settings reset).

## Selecting your keyboard model

The keymap selects your keyboard variant with a chosen **physical layout** node, e.g.:

```dts
chosen { zmk,physical-layout = &physical_layout_imprint_number_row; };
```

The available Imprint layouts are defined in
[`zmk-keyboards`](https://github.com/Cyboard-DigitalTailor/zmk-keyboards/blob/main/boards/shields/imprint/imprint-layouts.dtsi);
the `config/default keymaps/` folders contain a matching keymap for each. The
Dactyl and legacy single-arc keymaps still use the older
`zmk,matrix-transform` chosen node, as those models predate the physical
layout definitions.

## Updating a config repo created before July 2026

Older configs track ZMK `main` and select the keyboard model with a
`zmk,matrix-transform` chosen node. To move one onto the current stable stack:

1. In `config/west.yml`, set the `zmk` revision from `main` to `v0.3.0`
   (leave `zmk-keyboards` on `main`).
2. In `.github/workflows/build.yml`, change `@main` to `@v0.3.0`.
3. In your `config/imprint.keymap`, replace the chosen node

   ```dts
   chosen { zmk,matrix-transform = &imprint_<your model>; };
   ```

   with

   ```dts
   chosen { zmk,physical-layout = &physical_layout_imprint_<your model>; };
   ```

   Step 3 matters: with a chosen `zmk,matrix-transform`, ZMK ignores the
   physical layouts that the current `zmk-keyboards` shields are built
   around, and the firmware is not compatible with ZMK Studio.
4. (Optional, for ZMK Studio) In `build.yaml`, add the Studio snippet and
   config to the `imprint_left` entry, as in this template's `build.yaml`:

   ```yaml
   - board: assimilator-bt
     shield: imprint_left
     snippet: studio-rpc-usb-uart
     cmake-args: -DCONFIG_ZMK_STUDIO=y
   ```