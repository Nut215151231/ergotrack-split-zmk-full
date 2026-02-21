# ErgoTrack Split Keyboard

ZMK firmware for a split ergonomic keyboard with PMW3610 trackball and rotary encoders.

## Features

- **Split design** — Left and right halves connected via Bluetooth
- **PMW3610 trackball** on right side
- **Rotary encoders** — Left side: 2 encoders, Right side: 1 encoder
- **82 keys** total (6 rows × 7 columns per side, row 3 has 13 keys)
- **ZMK Studio support** — Edit keymap in browser without reflashing
- **nice!nano v2** compatible

## Hardware

### Components

- 2× nice!nano v2
- 1× PMW3610 trackball sensor
- 3× EC11 rotary encoders (2 left, 1 right)
- 82× switches + diodes
- 2× LiPo batteries

### Pin Assignments

| Function | Left GPIO | Right GPIO |
|---|---|---|
| Col 0–6 | P1.13, P1.11, P0.10, P0.09, P1.06, P1.04, P0.11 | P1.00, P0.11, P1.04, P1.06, P0.09, P0.10, P1.11 |
| Row 0–5 | P0.06, P0.08, P0.31, P0.29, P0.17, P0.20 | P0.06, P0.08, P0.17, P0.20, P0.31, P0.29 |
| Encoder 1 A/B | P0.22 / P0.02 | — |
| Encoder 2 A/B | P1.15 / P0.24 | — |
| Encoder R A/B | — | P0.02 / P0.22 |
| SPI SCK/MOSI/MISO | — | P1.01 / P1.02 / P1.02 |
| Trackball CS | — | P1.07 |
| Trackball MOTION | — | P0.24 |
| LED | P1.00 | P1.13 |

## Repository Structure

```
config/
├── west.yml                          # ZMK + PMW3610 driver modules
├── build.yaml                        # Build targets (left, right, settings_reset)
└── boards/shields/ergo_track_split/
    ├── ergo_track_split.dtsi         # Common DTS (matrix transform, battery)
    ├── ergo_track_split-layouts.dtsi # Physical layout for ZMK Studio
    ├── ergo_track_split.keymap       # Keymap definition
    ├── ergo_track_split.json         # ZMK Studio layout metadata
    ├── ergo_track_split_left.overlay # Left hardware definition
    ├── ergo_track_split_left.conf    # Left config (PERIPHERAL, ZMK Studio)
    ├── ergo_track_split_right.overlay# Right hardware definition + trackball
    ├── ergo_track_split_right.conf   # Right config (CENTRAL, PMW3610)
    ├── Kconfig.shield
    └── Kconfig.defconfig
```

## Build

### GitHub Actions (recommended)

Push to `main` branch to trigger automatic build. Download firmware from the Actions artifacts.

Three artifacts are produced:
- `ergo_track_split_left-nice_nano-zmk.uf2`
- `ergo_track_split_right-nice_nano-zmk.uf2`
- `settings_reset-nice_nano-zmk.uf2`

### Local build

```bash
west init -l config/
west update
west build -d build/left -b nice_nano -- -DSHIELD=ergo_track_split_left -DSNIPPET=studio-rpc-usb-uart
west build -d build/right -b nice_nano -- -DSHIELD=ergo_track_split_right
```

## Flashing

1. Double-tap reset on nice!nano to enter bootloader
2. Drag and drop the `.uf2` file onto the mounted drive
3. Flash left side first, then right side

## ZMK Studio

Edit your keymap in the browser without reflashing.

1. Connect keyboard via USB
2. Open [zmk.studio](https://zmk.studio/) in Chrome or Edge
3. Select the keyboard and start editing

> ZMK Studio works over USB only, not Bluetooth.

## Trackball

- Sensor: PMW3610, 1600 CPI
- Driver: [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)

To adjust CPI, edit `ergo_track_split_right.overlay`:

```c
trackball: trackball@0 {
    res-cpi = <1600>;  // 400, 800, 1200, or 1600
};
```

## What Changed to Fix the Build

The following issues were causing `devicetree error: lacks #sensor-binding-cells`:

### 1. Added `#sensor-binding-cells = <0>` to encoder nodes

All EC11 encoder nodes in both overlays were missing this required property.

```c
// Before — missing property caused DTS parser to misidentify
//          uart0_default as a sensor node
left_encoder_1: encoder_left_1 {
    compatible = "alps,ec11";
    ...
};

// After — property added to all encoder nodes
left_encoder_1: encoder_left_1 {
    compatible = "alps,ec11";
    ...
    #sensor-binding-cells = <0>;
};
```

### 2. Removed keymap headers from right overlay

`ergo_track_split_right.overlay` had unnecessary includes intended for keymap files, which confused the DTS parser:

```c
// Removed from right overlay:
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/mouse.h>
#include <dt-bindings/zmk/outputs.h>
```

### 3. Added PMW3610 driver to `west.yml`

The PMW3610 driver is not included in ZMK core. Without this, the right side build fails with a missing driver error.

```yaml
# Added to west.yml
- name: zmk-pmw3610-driver
  remote: badjeff
  revision: main
```

## Credits

- PMW3610 driver: [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)
- ZMK Firmware: [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk)

## License

MIT
