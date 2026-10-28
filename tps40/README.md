# TPS40 (Daisy / tps40a) Configuration & Firmware Guide

## Files in this folder

| File | Purpose |
| :--- | :--- |
| `tps40a.json` | **VIA v3 Keyboard Definition** — Load this into VIA's **Design** tab so VIA recognizes the keyboard hardware, matrix, and custom Bluetooth/OS keycodes. |
| `tps40a.layout.json` | **Saved VIA Keymap** — Load this into VIA's **Save + Load** tab (under Configure) to restore custom keybindings across all layers. |
| `tps40_vial-daisy.bin` | **Vial-QMK Custom Firmware** (`release/tps40/20250804.2` by `kuriki-sasa`) for the Daisy (staggered) layout. (`sha256: efdb4aa0ea95373d77743b56a11b140303a5955f1cc8643950ed0b8e85a35cae`) |
| `boot0.png` | Reference photo showing the `BOOT0` pads on the back of the PCB to short when entering DFU bootloader mode. |

---

## Using the Stock Firmware with VIA

Because the TPS40's stock firmware was never merged into upstream QMK/VIA, VIA does not auto-detect it until you load the v3 definition:

1. Open [https://usevia.app](https://usevia.app) in a WebHID-supported browser (Chrome / Edge).
2. Go to **Settings** (gear icon) and turn **ON** `Show Design tab`.
3. Go to the **Design** tab (paintbrush icon):
   - Ensure `Use V2 definitions (deprecated)` is turned **OFF**.
   - Click **Load** and select `tps40a.json`.
4. Go to the **Configure** tab, authorize the TPS40 in the browser popup, and edit keys directly.
5. To restore or back up your keymap, use the **Save + Load** section on the **Configure** tab with `tps40a.layout.json`.

---

## Weekend Project: Flashing Vial-QMK Firmware

Source release: [https://github.com/kuriki-sasa/vial-qmk/releases/tag/release/tps40/20250804.2](https://github.com/kuriki-sasa/vial-qmk/releases/tag/release/tps40/20250804.2)

### Why upgrade?
- **Auto-detected without JSON side-loading:** Works out of the box with [https://vial.rocks](https://vial.rocks) or the Vial desktop app.
- **Full QMK / Vial features:** Unlocks Tap Dance, Combos, Key Overrides, Rotary Encoder mapping, and QMK tapping term settings.
- **Stability & LED fixes:** Fixes key drops, battery/charging LED indicators, and layer indicator LEDs.

### Prerequisites
Install `dfu-util` via Homebrew if not already installed:
```sh
brew install dfu-util
```

### Step 1: Enter DFU Bootloader Mode
1. Unplug the USB cable from the TPS40 (and turn off the battery switch if applicable).
2. Disassemble the case to access the back of the PCB.
3. Short-circuit the two `BOOT0` pads highlighted in `boot0.png` (using tweezers or a paperclip) **while plugging in the USB cable**.
4. Verify that `dfu-util` sees the microcontroller (`2e3c:df11`):
   ```sh
   dfu-util -l
   ```

### Step 2: Back up the Stock Factory Firmware (Important!)
Before flashing, save a dump of the original factory firmware in case you ever want to revert:
```sh
dfu-util -a 0 -d 2e3c:df11 -s 0x8000000 -U tps40a_factory_backup.bin
```

### Step 3: Flash the Vial Firmware
With the board still in DFU mode (re-plug while shorting `BOOT0` if it exited DFU after the backup):
```sh
dfu-util -a 0 -d 2e3c:df11 -s 0x8000000:leave -D tps40_vial-daisy.bin
```

### Step 4: Configure in Vial
1. Open [https://vial.rocks](https://vial.rocks) or the Vial app.
2. Recreate your keymap (note: VIA `.layout.json` files are tied to the stock 5x13 matrix and custom keycodes, whereas the Vial firmware uses a 5x12 matrix with proper encoder definitions, so you'll want to set up your layers fresh in Vial and save a `.vil` backup).

### Custom Keycodes in Vial (`User` Tab)
In the Vial firmware, Bluetooth and hardware functions live under the **User** tab:
- **Deep Sleep**: Puts MCU into deep sleep (press the rotary encoder button to wake).
- **Pair 1 / Pair 2 / Pair 3**: Initiate Bluetooth pairing for slot 1–3.
- **BT 1 / BT 2 / BT 3**: Connect to paired Bluetooth device in slot 1–3.
  - *Tip:* On stock firmware, tapping `BT 1` connected and holding `BT 1` paired. In Vial, you can replicate this by setting up a **Tap Dance** (Tap = `BT 1`, Hold = `Pair 1`).
- **Disco. BT**: Disconnect from current Bluetooth device.
- **LED mode**: Toggle between LED Mode 1 and LED Mode 2:
  - **LED Mode 1**: LED 1 = Battery status | LED 2 = Caps Lock | LED 3 = BT status
  - **LED Mode 2**: LED 1 = Battery status | LED 2 = Lower layer | LED 3 = Raise layer (both = Adjust)
- **Reboot**: Reboot the MCU.
