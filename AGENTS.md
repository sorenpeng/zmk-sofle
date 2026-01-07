# Repository Guidelines

## Project Structure & Module Organization
This repo holds a ZMK configuration for the Eyelash Sofle split keyboard (not the ZMK firmware source). Key paths:
- `config/`: user configuration (`eyelash_sofle.keymap`, `eyelash_sofle.conf`, `eyelash_sofle_left.conf`, `west.yml`).
- `boards/arm/eyelash_sofle/`: board definitions and DTS/Kconfig for left/right halves.
- `keymap-drawer/`: generated SVG layouts and assets.
- `build.yaml`: GitHub Actions build workflow.
- `zephyr/`: module definitions for Zephyr.

## Build, Test, and Development Commands
- `west init -l config` + `west update`: set up a ZMK workspace (first time).
- `west build -d build/left -b eyelash_sofle_left -- -DSHIELD=eyelash_sofle_left`: build left UF2.
- `west build -d build/right -b eyelash_sofle_right -- -DSHIELD=eyelash_sofle_right`: build right UF2.
- `west build -d build/left -b eyelash_sofle_left -p -- -DSHIELD=eyelash_sofle_left`: clean rebuild.
- `west build -d build/left -b eyelash_sofle_left -S studio-rpc-usb-uart`: Studio build (left/central only).
- `keymap-drawer -c keymap_drawer.config.yaml`: regenerate `keymap-drawer/eyelash_sofle.svg`.

## Coding Style & Naming Conventions
- Use 4-space indentation in `.keymap`/`.dts` device-tree blocks; keep `bindings` aligned for readability.
- Key and behavior names are lower snake case (e.g., `mt_z_pref`), labels are uppercase (e.g., `TD_CAPS_WORD`).
- `.conf` uses `CONFIG_...` Kconfig entries; keep comments short and focused.
- Prefer the `eyelash_sofle` prefix for related files and shields.

## Testing Guidelines
- No automated test suite; treat a successful `west build` for both halves as the primary verification.
- Verify artifacts at `build/left/zephyr/zmk.uf2` and `build/right/zephyr/zmk.uf2`.
- For keymap changes, regenerate the SVG and visually inspect layer legends.

## Commit & Pull Request Guidelines
- Commit messages commonly use `[Draw] ...` for SVG updates, `Update:` / `Updated ...` for general changes, or `feat(scope):` / `fix(scope):` when scoped.
- PRs should describe the layer/behavior changes, mention hardware impact (left/right, display, RGB), and include updated `keymap-drawer/eyelash_sofle.svg` when keymaps change.
