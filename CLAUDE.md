# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware configuration for the **Eyelash Sofle** split ergonomic keyboard running on nRF52840 MCUs. This is a personal configuration repository, not the ZMK firmware source itself.

## Build Commands

```bash
# Initialize workspace (first time only)
west init -l config
west update

# Build left half
west build -d build/left -b eyelash_sofle_left -- -DSHIELD=eyelash_sofle_left

# Build right half
west build -d build/right -b eyelash_sofle_right -- -DSHIELD=eyelash_sofle_right

# Clean build (if needed)
west build -d build/left -b eyelash_sofle_left -p -- -DSHIELD=eyelash_sofle_left
```

Output: `build/left/zephyr/zmk.uf2` and `build/right/zephyr/zmk.uf2`

## Architecture

### Key Directories
- `config/` - User configuration (keymap, features, west manifest)
- `boards/arm/eyelash_sofle/` - Hardware board definitions (device tree, Kconfig)
- `keymap-drawer/` - Auto-generated SVG keymap visualizations (via GitHub Actions)

### Configuration Files
- `config/eyelash_sofle.keymap` - Primary keymap definition (layers, combos, behaviors)
- `config/eyelash_sofle.conf` - Feature toggles (RGB, display, power management)
- `config/west.yml` - Dependency manifest (ZMK, nice-view-gem, mario peripheral)
- `boards/arm/eyelash_sofle/eyelash_sofle.dtsi` - Shared hardware configuration
- `boards/arm/eyelash_sofle/*_left.dts` / `*_right.dts` - Split-specific hardware

### Layer Structure (in keymap)
- Layer 0: Main QWERTY with mod-tap behaviors
- Layer 1: Symbols, numbers, F-keys
- Layer 2: Bluetooth/system controls, RGB, media
- Layer 3: Gaming mode

### Split Configuration
- Left half = Central (USB + Bluetooth, main logic)
- Right half = Peripheral (Bluetooth only to left half)

## ZMK Device Tree Syntax

Keymaps use ZMK's device tree bindings:
- `&kp KEY` - Standard keypress
- `&mo LAYER` - Momentary layer
- `&lt LAYER KEY` - Layer-tap (hold=layer, tap=key)
- `&mt MOD KEY` - Mod-tap (hold=modifier, tap=key)
- `&trans` - Transparent (pass to lower layer)
- `&none` - No action

Behaviors are defined in `/ { behaviors { ... } }` blocks with custom timing parameters.

## GitHub Actions

- `build.yml` - Compiles firmware on push, uploads UF2 artifacts
- `draw.yml` - Regenerates keymap SVG diagrams when config changes
