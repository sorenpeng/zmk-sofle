# Code Review Notes

## Scope
- Manual review of the ZMK config and board definitions in this repo.

## Findings (Prioritized)
### 1) ZMK Studio key referenced without Studio enabled
- **Where**: `config/eyelash_sofle.keymap` and `boards/arm/eyelash_sofle/eyelash_sofle.keymap`
- **Issue**: `&studio_unlock` is referenced, but standard left/right builds in `build.yaml` do not enable `CONFIG_ZMK_STUDIO`. If Studio is not enabled, the label is typically undefined and can break the build.
- **Impact**: Build failure for non-Studio configurations.
- **Suggestion**: Either enable Studio globally in `config/eyelash_sofle.conf`, or remove/guard the `&studio_unlock` binding for normal builds.

### 2) Two keymap sources can diverge
- **Where**: `config/eyelash_sofle.keymap` and `boards/arm/eyelash_sofle/eyelash_sofle.keymap`
- **Issue**: Two separate keymaps exist. Editing one may not affect the actual build depending on which flow is used.
- **Impact**: Confusion and unintentional mismatch between compiled firmware and expected layout.
- **Suggestion**: Prefer a single authoritative keymap or document which one is used for day-to-day builds.

### 3) Underglow chain length comment mismatch
- **Where**: `boards/arm/eyelash_sofle/eyelash_sofle.dtsi`
- **Issue**: `chain-length = <7>;` but the comment says 6 keys have underglow.
- **Impact**: Potential off-by-one expectations if hardware has 6 LEDs.
- **Suggestion**: Confirm actual LED count and align `chain-length` and comment.

## Assumptions
- Builds are primarily driven by `config/` plus `west` and `build.yaml`.

