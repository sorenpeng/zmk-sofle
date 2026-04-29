# CLAUDE.md

## Project Snapshot
- Repo purpose: ZMK configuration for the Eyelash Sofle split keyboard (not the ZMK firmware source).
- Primary editable areas: `config/` and `boards/arm/eyelash_sofle/`.
- Treat local west workspace directories as generated state unless explicitly requested: `build/`, `zmk/`, `modules/`, `tools/`, `.west/`.
- Keep board/shield names stable: `eyelash_sofle_left`, `eyelash_sofle_right`, `nice_view_gem`, `settings_reset`.

## Commands (Authoritative)
- Install toolchain: `mise install`
- First-time setup: `mise run setup`
- Sync dependencies: `mise run sync`
- Build left: `mise run build-left`
- Build right: `mise run build-right`
- Build both halves: `mise run build-all`
- Build Studio (left/central): `mise run build-left-studio`
- Clean local artifacts: `mise run clean`
- Regenerate keymap SVG: `keymap-drawer -c keymap_drawer.config.yaml`

Direct west commands (when needed):
- `west build -s zmk/app -d build/left -b eyelash_sofle_left -- -DSHIELD=eyelash_sofle_left -DZMK_CONFIG=$PWD/config`
- `west build -s zmk/app -d build/right -b eyelash_sofle_right -- -DSHIELD=eyelash_sofle_right -DZMK_CONFIG=$PWD/config`

## Validation
- There is no automated unit test suite.
- Primary verification is successful build of both halves.
- Expected artifacts:
  - `build/left/zephyr/zmk.uf2`
  - `build/right/zephyr/zmk.uf2`
  - `build/left-studio/zephyr/zmk.uf2` (Studio build)
- If keymap changes, regenerate and review `keymap-drawer/eyelash_sofle.svg`.

## Code Style (Repo-Specific)
- Use 4-space indentation in `.keymap` and `.dts/.dtsi`.
- Keep `bindings = < ... >;` columns aligned for readability.
- Behavior and helper identifiers use lower snake case (example: `mt_z_pref`).
- Behavior labels use uppercase (example: `TD_CAPS_WORD`).
- `.conf` files should only contain `CONFIG_...` entries with concise comments.

## Architecture and Boundaries
- `config/eyelash_sofle.keymap`: layers, combos, custom behaviors.
- `config/eyelash_sofle.conf`: shared feature flags (display, RGB, power, HID).
- `config/eyelash_sofle_left.conf`: left/central-only options (for example ZMK Studio).
- `boards/arm/eyelash_sofle/*.dts*`: hardware mapping, matrix, transforms, encoder.
- `config/west.yml`: module and dependency pins.
  - `zmk` is pinned to `v0.3.0`.
  - `nice-view-gem` is pinned to commit `7794ebf` for LVGL v8 compatibility.

## Workflow and Repo Etiquette
- Keep changes minimal and scoped to the requested task.
- Common commit styles in this repo: `[Draw] ...`, `Updated ...`, `Update ...`, `fix(scope): ...`.
- For keymap changes, include the corresponding `keymap-drawer/eyelash_sofle.svg` update in the same change.

## Gotchas
- Left half is central (USB/BLE + Studio); right half is peripheral.
- CI includes fallback logic from `nice_view_gem` to `nice_view`; avoid unnecessary renames of board/shield identifiers.
- Avoid broad refactors across layout/behavior blocks unless explicitly requested.
