# Legacy Project README

This archived document preserves the previous root README content in English.
It is no longer the active project entry point.

## Project

Personal ZMK firmware configuration for an Eyelash Sofle split ergonomic keyboard.
The configuration targets an Eyelash Sofle build with nice!view-style display support.

## Previous Feature Summary

- Display support for layer and connection status.
- Sleep behavior for long idle periods.
- RGB underglow, backlight, Bluetooth, and media controls configured through ZMK.

## Hardware

- Eyelash Sofle split keyboard.
- nice!nano-compatible controller.
- nice!view-compatible display.

## Previous Keymap Notes

The keymap is organized into multiple layers:

- `Qwerty`: base typing layer with tap-hold keys for escape, shift, alt, command, enter, tab, backspace, and layer access.
- `Hk+Num`: function keys, number pad keys, symbols, and common shortcuts.
- `Sys+Nav`: Bluetooth controls, media controls, navigation keys, brightness, volume, and system shortcuts.

The current authoritative keymap is `config/eyelash_sofle.keymap`.
The rendered layout is `keymap-drawer/eyelash_sofle.svg`.

## Previous File Map

- `build.yaml`: GitHub Actions build matrix.
- `keymap_drawer.config.yaml`: keymap-drawer configuration.
- `boards/`: keyboard hardware definitions.
- `config/`: ZMK configuration and keymap files.
- `keymap-drawer/`: rendered keymap assets.
- `zephyr/`: Zephyr module metadata.

## Previous Build Notes

The previous README recommended using `mise` for local tool management:

```bash
mise install
mise run setup
mise run build-all
```

These notes are archived for reference only. The active root README intentionally shows only the keymap layout.
