# Codebase Issues and Bugs

Analysis of the Eyelash Sofle ZMK firmware configuration. Issues are prioritized by severity.

---

## 🔴 Critical Issues

### 1. Missing Encoder Sensor Bindings

**Severity:** Critical
**Files:** `config/eyelash_sofle.keymap`
**Status:** Non-functional feature

The left encoder is enabled in hardware (`boards/arm/eyelash_sofle/eyelash_sofle_left.dts:21-23`) but the keymap contains **no `sensor-bindings`** block. This means the encoder physically rotates with no output.

**Current State:**
- Hardware enabled: ✓
- Sensor bindings: ✗

**Fix:**
Add sensor bindings to each layer in the keymap. Example for layer 0:

```dts
keymap {
    compatible = "zmk,keymap";

    Main0 {
        label = "Qwerty";
        sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN>;
        bindings = <
            // ... existing bindings
        >;
    };

    Sym+Num1 {
        label = "Sym+Num";
        sensor-bindings = <&inc_dec_kp PAGE_UP PAGE_DOWN>;
        bindings = <
            // ... existing bindings
        >;
    };

    // Add for other layers as needed
};
```

---

### 2. JK-to-Escape Combo Uses Wrong Key Positions

**Severity:** Critical
**File:** `config/eyelash_sofle.keymap:17-21`
**Issue:** Combo triggers on wrong keys

The combo is named "jk-to-Escape" but uses key positions `34` and `35`, which correspond to **H and J**, not J and K.

**Current Mapping:**
```
Position 34 → H (Row 2, Col 9)
Position 35 → J (Row 2, Col 10)
Position 36 → K (Row 2, Col 11)
```

**Actual Behavior:**
- Pressing H+J triggers ESC (not intended)
- Pressing J+K does nothing (defeats the combo purpose)

**Fix:**
```dts
jk-to-Escape {
    bindings = <&kp ESC>;
    key-positions = <35 36>;  // Changed from 34 35 to 35 36
    layers = <0>;
};
```

---

## 🟠 High Priority Issues

### 3. Hold-Tap Behaviors Missing `quick-tap-ms`

**Severity:** High
**File:** `config/eyelash_sofle.keymap:32-46`
**Impact:** Unintended modifier activation during fast typing

The `mt_z_pref` and `mt_slash_pref` behaviors lack the `quick-tap-ms` parameter. This causes the behavior to treat rapid keypresses as holds, activating modifiers unintentionally.

**Current Definition:**
```dts
mt_z_pref: mod_tap_z_preferred {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    bindings = <&kp>, <&kp>;
    // Missing: quick-tap-ms = <150>;
};
```

**Problem Scenario:**
- Typing "zzz" quickly while trying to tap Z may register Z as a modifier
- User expects Z, gets unexpected modifier behavior

**Fix:**
Add `quick-tap-ms` to both behaviors:

```dts
mt_z_pref: mod_tap_z_preferred {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    quick-tap-ms = <150>;  // Added
    bindings = <&kp>, <&kp>;
};

mt_slash_pref: mod_tap_slash_preferred {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    quick-tap-ms = <150>;  // Added
    bindings = <&kp>, <&kp>;
};
```

**Note:** Adjust `quick-tap-ms` value (150ms is a suggestion) based on typing speed preferences.

---

### 4. Duplicate Identical Behaviors

**Severity:** High
**File:** `config/eyelash_sofle.keymap:32-46`
**Issue:** Code redundancy

`mt_z_pref` and `mt_slash_pref` are **100% identical** in configuration. They differ only in name but have identical:
- `compatible`
- `#binding-cells`
- `flavor`
- `tapping-term-ms`
- `bindings`

**Current Code:**
```dts
mt_z_pref: mod_tap_z_preferred {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    bindings = <&kp>, <&kp>;
};

mt_slash_pref: mod_tap_slash_preferred {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    bindings = <&kp>, <&kp>;
};
```

**Fix - Option A (Consolidate):**
```dts
mod_tap_preferred: mod_tap_preferred {
    compatible = "zmk,behavior-hold-tap";
    label = "MOD_TAP_PREFERRED";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    bindings = <&kp>, <&kp>;
};
```

Then use in keymap:
```dts
&mod_tap_preferred LEFT_ALT Z
&mod_tap_preferred RIGHT_ALT SLASH
```

**Fix - Option B (Keep separate for now):**
At minimum, add `label` fields to differentiate them for debugging purposes.

---

## 🟡 Medium Priority Issues

### 5. Backlight Never Auto-Turns Off During Idle

**Severity:** Medium
**File:** `config/eyelash_sofle.conf:47`
**Impact:** Battery drain

The backlight is set to never turn off during idle:
```
CONFIG_ZMK_BACKLIGHT_AUTO_OFF_IDLE=n
```

Combined with `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000` (1 hour), the backlight remains **on for 60 minutes** even when keyboard is idle. This drains battery unnecessarily.

**Current Configuration:**
```
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=3600000     # 1 hour until sleep
CONFIG_ZMK_BACKLIGHT_ON_START=y            # Backlight starts on
CONFIG_ZMK_BACKLIGHT_AUTO_OFF_IDLE=n       # Never auto-off during idle
```

**Recommendation:**
Either enable auto-off OR reduce idle timeout:

**Option A (Enable auto-off):**
```
CONFIG_ZMK_BACKLIGHT_AUTO_OFF_IDLE=y
```

**Option B (Reduce idle timeout):**
```
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=300000       # 5 minutes
```

---

### 6. RGB Underglow Never Auto-Turns Off During Idle

**Severity:** Medium
**File:** `config/eyelash_sofle.conf:23`
**Impact:** Battery drain

Same issue as backlight. RGB underglow never turns off during idle:
```
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=n
```

**Current Configuration:**
```
CONFIG_ZMK_RGB_UNDERGLOW_ON_START=n        # Starts off (good)
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=n   # But won't auto-off if turned on
```

**Recommendation:**
Enable auto-off for RGB underglow:

```
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y
```

This ensures RGB turns off when keyboard goes idle, regardless of initial state.

---

## 🔵 Low Priority Issues

### 7. Missing Peripheral Battery Level Fetching Configuration

**Severity:** Low
**File:** `config/eyelash_sofle.conf`
**Impact:** Missing information feature

For split keyboards, the central controller (left half) cannot display the peripheral's (right half) battery level unless explicitly configured:

**Recommended Addition:**
```
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
```

This allows the left side's display to show both halves' battery percentages, useful for monitoring right-side battery without manual checking.

---

### 8. No Positional Hold-Tap Configuration for Homerow Mods

**Severity:** Low
**File:** `config/eyelash_sofle.keymap:57-58`
**Impact:** Occasional mis-triggers on other keys

The keymap uses hold-tap modifiers on several keys:
- `&mt LEFT_SHIFT BACKSPACE` (left pinky)
- `&mt LEFT_CONTROL TAB` (left thumb)
- `&mt LEFT_SHIFT BACKSPACE` (right thumb)

**Current Issue:**
These behaviors can accidentally trigger modifiers when typing other keys in quick succession. Example:
- Type "Shift+letter" quickly → can accidentally hold Shift on nearby keys

**Recommendation:**
Add `hold-trigger-key-positions` to prevent unintended triggering:

```dts
mt_backspace_shift: mt_backspace_shift {
    compatible = "zmk,behavior-hold-tap";
    label = "MT_BACKSPACE_SHIFT";
    #binding-cells = <2>;
    flavor = "balanced";  // or "hold-preferred"
    tapping-term-ms = <200>;
    hold-trigger-key-positions = <0 1 2 3 4 5 6 7 8 9 10 11 12 13>;  // Top row excluded
    bindings = <&kp>, <&kp>;
};
```

This prevents the behavior from triggering when pressing specific key positions.

---

## 📋 Summary Table

| Priority | Issue | Type | Impact |
|----------|-------|------|--------|
| 🔴 Critical | Missing encoder bindings | Non-functional | Encoder does nothing |
| 🔴 Critical | Wrong combo key positions | Mis-mapped | JK doesn't trigger ESC |
| 🟠 High | Missing quick-tap-ms | Usability | Accidental modifiers |
| 🟠 High | Duplicate behaviors | Code quality | Maintainability |
| 🟡 Medium | Backlight always on | Battery | 1hr drain per session |
| 🟡 Medium | RGB always on | Battery | Drain when powered |
| 🔵 Low | No battery sync | Feature | Missing info |
| 🔵 Low | No positional hold-tap | Usability | Occasional mis-triggers |

---

## Recommended Fix Priority

1. **Fix Critical Issues First** - Encoder and combo positions affect core functionality
2. **Fix High Priority** - Quick-tap-ms and consolidate behaviors before adding more features
3. **Address Medium Issues** - Battery drain has cumulative effect over time
4. **Consider Low Priority** - Quality-of-life improvements for future iterations

---

*Analysis Date: January 5, 2026*
*Codebase: ZMK Eyelash Sofle Configuration*
