# Ergo Track Split Keyboard

ZMK firmware for a split ergonomic keyboard with PMW3610 trackball and rotary encoders.

## Features

- **Split design** - Left and right halves connected via Bluetooth
- **PMW3610 trackball** on right side with auto-mouse layer activation
- **Rotary encoders**:
  - Left side: 2 encoders (volume, page navigation)
  - Right side: 1 encoder (brightness)
- **84 keys** total (6 rows × 7 columns per side)
- **Battery LED indicator** on right side
- **nice!nano v2** compatible boards

## Hardware

### Pin Assignments

See `PINAssign.pdf` for detailed pin mappings.

Key features:
- **Matrix**: COL2ROW with 6 rows, 7 columns per side
- **SPI**: Single-wire SPI (SDIO) for PMW3610 on right side
- **Battery monitoring**: Via ADC on nice!nano v2

### Components

- 2× nice!nano v2 boards
- 1× PMW3610 trackball sensor module
- 3× EC11 rotary encoders (2 left, 1 right)
- 84× keyboard switches + diodes
- 1× LED for battery indicator
- 2× LiPo batteries

## Firmware Setup

### Prerequisites

1. Install ZMK development environment
2. Clone this repository
3. Initialize west workspace:

```bash
west init -l config/
west update
```

### Building

Build firmware using GitHub Actions (recommended):

1. Fork this repository
2. Enable GitHub Actions
3. Push changes to trigger build
4. Download firmware from Actions artifacts

Or build locally:

```bash
cd zmk/app
west build -d build/left -b nice_nano_v2 -- -DSHIELD=ergo_track_split_left
west build -d build/right -b nice_nano_v2 -- -DSHIELD=ergo_track_split_right
```

### Flashing

1. Put nice!nano into bootloader mode (double-tap reset)
2. Flash the appropriate UF2 file:
   - Left side: `ergo_track_split_left-nice_nano_v2-zmk.uf2`
   - Right side: `ergo_track_split_right-nice_nano_v2-zmk.uf2`

## ZMK Studio - Web-based Keymap Editor

This keyboard supports **ZMK Studio**, allowing you to edit your keymap directly in your browser without reflashing firmware (similar to Remap/VIA).

### Using ZMK Studio

1. **Connect your keyboard** via USB to your computer
2. **Open ZMK Studio** in a Chrome/Edge browser: https://zmk.studio/
3. **Unlock the keyboard**: Press `Fn + Del` (Layer 4 + rightmost key on top row)
4. **Edit your keymap** in the web interface
5. **Save changes** by clicking the "Save" button (changes are stored on the keyboard)

### Important Notes

- **ZMK Studio only works over USB**, not Bluetooth
- Changes are saved to the keyboard's flash memory and persist across power cycles
- You must click "Save" in the top right corner to persist changes
- Once using ZMK Studio, .keymap file changes won't apply unless you "Restore Stock Settings"
- The unlock binding (`&studio_unlock`) is located at: **Layer 4 + Del key**

### Supported Features in ZMK Studio

✅ Change key bindings on all layers
✅ Modify encoder functions
✅ Configure mouse buttons
✅ Bluetooth profile management

⚠️ Advanced features (macros, tap dance, complex behaviors) must still be configured in .keymap file

### Troubleshooting

**Can't connect to ZMK Studio:**
- Make sure you're using USB connection (not Bluetooth)
- Use Chrome or Edge browser (WebUSB required)
- Try unlocking the keyboard with `Fn + Del`

**Changes don't persist:**
- Always click the "Save" button after making changes
- Wait for the save operation to complete (a few seconds)

## Configuration

### Trackball Settings

Adjust trackball behavior in `ergo_track_split_right.overlay`:

```c
trackball: trackball@0 {
    cpi = <600>;              // Cursor speed (400-1600)
    x-invert;                 // Uncomment to invert X axis
    y-invert;                 // Uncomment to invert Y axis
    xy-swap;                  // Uncomment to swap X/Y axes
    
    // Power saving timeouts
    sleep1-timeout-ms = <500>;
    sleep2-timeout-ms = <2000>;
    sleep3-timeout-ms = <10000>;
};
```

### Auto-Mouse Layer

The trackball automatically activates Layer 3 (mouse layer) when moved:

```c
trackball_listener: trackball_listener {
    layers = <3>;              // Mouse layer number
    timeout-ms = <1000>;       // Return to default after 1 second
};
```

### Keymap Customization

Edit `ergo_track_split.keymap` to customize:

- Key assignments
- Layer functions
- Encoder behaviors
- Mouse button mappings

### Battery LED

The battery LED on the right side (P1.13) can be configured for different battery level indicators by modifying the firmware.

## Layers

- **Layer 0**: Default typing layer
- **Layer 1**: Lower (function keys, navigation)
- **Layer 2**: Raise (symbols, numbers)
- **Layer 3**: Mouse (auto-activated by trackball movement)
- **Layer 4**: Function/Settings (Bluetooth, etc.)

## Troubleshooting

### Trackball not working

1. Check SPI connections (SCK, SDIO, nCS)
2. Verify MOTION pin is connected
3. Enable debug logging in `.conf`:
   ```
   CONFIG_ZMK_USB_LOGGING=y
   CONFIG_PMW3610_LOG_LEVEL_DBG=y
   ```

### Split halves not connecting

1. Clear Bluetooth bonds on both sides
2. Re-flash both halves
3. Check that left side is set as central role

### High battery drain

1. Reduce trackball CPI
2. Increase sleep timeouts
3. Disable debug logging

## Credits

- PMW3610 driver: [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)
- Input behavior listener: [badjeff/zmk-input-behavior-listener](https://github.com/badjeff/zmk-input-behavior-listener)
- ZMK Firmware: [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk)

## License

MIT License - See individual files for copyright notices.
