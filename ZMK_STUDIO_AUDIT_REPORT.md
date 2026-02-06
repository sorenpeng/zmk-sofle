# ZMK Studio Support Audit Report
**Date**: 2026-01-11
**Repository**: zmk-sofle (Eyelash Sofle)
**Audit Objective**: Ensure firmware can connect to zmk.studio via USB and support live remapping

---

## Executive Summary

**Audit Result**: ✅ **ALL ZMK Studio requirements are correctly configured**

This repository fully satisfies all hard requirements for ZMK Studio:
- Physical Layout correctly defined (64 keys)
- Build flags properly configured (`CONFIG_ZMK_STUDIO=y` + `studio-rpc-usb-uart` snippet)
- Unlock mechanism configured (locking disabled for debugging)
- Split keyboard roles correctly assigned (Left = Central + USB, Right = Peripheral)

If Studio connection fails, the issue is NOT in the configuration. See [Troubleshooting](#troubleshooting).

---

## A. Repository Structure Identification

### 1. Build System

| Item | Value |
|------|-------|
| **Build Method** | GitHub Actions |
| **Workflow** | `.github/workflows/build.yml` (uses `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3.0`) |
| **Build Matrix** | `/build.yaml` |
| **ZMK Version** | v0.3.0 (config/west.yml:22) |

### 2. Target Hardware

| Item | Value |
|------|-------|
| **Board** | `eyelash_sofle` (nRF52840-based) |
| **Variants** | `eyelash_sofle_left` (Central/USB), `eyelash_sofle_right` (Peripheral/BLE) |
| **Shield** | `nice_view_gem` (display module) |
| **Architecture** | Split keyboard |

### 3. Key Configuration Files

```
boards/arm/eyelash_sofle/
├── eyelash_sofle.dtsi              # Hardware definition (shared)
├── eyelash_sofle-layouts.dtsi      # Physical Layout definition
├── eyelash_sofle_left.dts          # Left-specific config
├── eyelash_sofle_right.dts         # Right-specific config
├── eyelash_sofle.keymap            # Keymap bindings
├── eyelash_sofle_left_defconfig    # Left build options
└── eyelash_sofle_right_defconfig   # Right build options

config/
├── eyelash_sofle.conf              # Feature flags (shared)
├── eyelash_sofle_left.conf         # Left-specific config
└── west.yml                        # Dependency manifest

build.yaml                          # GitHub Actions build matrix
```

---

## B. ZMK Studio Hard Requirements Audit

### B1. Physical Layout Configuration

#### ✅ Audit PASSED

**File**: `boards/arm/eyelash_sofle/eyelash_sofle-layouts.dtsi`

```devicetree
/ {
    eyelash_sofle_layout: eyelash_sofle_layout {
        compatible = "zmk,physical-layout";  // ✅ Correctly declared
        display-name = "Default Layout";

        keys = <  // ✅ 64 key positions with physical attributes
            &key_physical_attrs <100> <100> <0> <0> <0> <0> <0>
            &key_physical_attrs <100> <100> <100> <0> <0> <0> <0>
            // ... (64 key definitions total)
        >;
    };
};
```

**chosen node configuration** (`eyelash_sofle.dtsi:26`):
```devicetree
chosen {
    zmk,physical-layout = &eyelash_sofle_layout;  // ✅ Correctly referenced
    // No zmk,matrix-transform (deprecated)  ✅ Correct
};
```

**Verification Results**:
- Matrix Transform key count: 64
- Keymap bindings count: 64
- Physical Layout key count: 64
- **Status**: ✅ Perfect match

---

### B2. Build Flags Configuration

#### ✅ Audit PASSED

**File 1**: `config/eyelash_sofle_left.conf:2`
```kconfig
CONFIG_ZMK_STUDIO=y  // ✅ Studio support enabled
```

**File 2**: `build.yaml:9-11`
```yaml
- board: eyelash_sofle_left
  shield: nice_view_gem
  snippet: studio-rpc-usb-uart  // ✅ USB-UART communication enabled
  cmake-args: -DCONFIG_ZMK_STUDIO_LOCKING=n  // ✅ Locking disabled (for debugging)
  artifact-name: eyelash_sofle_studio_left
```

**File 3**: `boards/arm/eyelash_sofle/eyelash_sofle.dtsi:109-111`
```devicetree
&zephyr,cdc_acm_uart {
    cdc_acm_uart: cdc_acm_uart {
        compatible = "zephyr,cdc-acm-uart";  // ✅ USB CDC ACM device
    };
};
```

**File 4**: `boards/arm/eyelash_sofle/eyelash_sofle_left_defconfig:40`
```kconfig
CONFIG_ZMK_USB=y  // ✅ USB support enabled
```

**Verification Checklist**:
- [x] `CONFIG_ZMK_STUDIO=y`
- [x] `snippet: studio-rpc-usb-uart`
- [x] USB CDC ACM device defined
- [x] USB functionality enabled

---

### B3. Unlock Key Configuration

#### ✅ Audit PASSED

**File**: `boards/arm/eyelash_sofle/eyelash_sofle.keymap:39`

**Unlock key location**:
```
Layer 2 (Row 2, Column 1 - TAB physical position):
&studio_unlock  &bt BT_CLR  &bt BT_CLR_ALL  ...
```

**How to access**:
1. Hold Layer 0 Row 5 Column 10 (`&mo 2`, right thumb cluster)
2. Simultaneously press top-left Row 2 Column 1 (TAB position)

**Locking behavior** (`build.yaml:10`):
```yaml
cmake-args: -DCONFIG_ZMK_STUDIO_LOCKING=n
```
- **Current state**: Locking disabled
- **Impact**: Automatically unlocked after connection, no need to press `&studio_unlock`

**Verification Results**:
- [x] `&studio_unlock` exists in keymap
- [x] Key position is accessible (Layer 2, Row 2 Col 1)
- [x] Locking disabled (debugging-friendly)

---

## C. Split Keyboard Special Handling

### ✅ Configuration Correct

#### Central/Peripheral Role Assignment

**Left side** (`boards/arm/eyelash_sofle/Kconfig.defconfig:11-12`):
```kconfig
config ZMK_SPLIT_ROLE_CENTRAL
    default y  // ✅ Left = Central
```

**Right side**:
```kconfig
# No CENTRAL config  // ✅ Right = Peripheral (default)
```

#### USB Configuration

| Side | USB Config | Reason |
|------|------------|--------|
| Left | `CONFIG_ZMK_USB=y` | Central handles USB connection |
| Right | `CONFIG_ZMK_USB=n` | Peripheral only communicates via BLE to Central |

#### Studio Build Configuration

**build.yaml comparison**:
```yaml
# Left (with Studio)
- board: eyelash_sofle_left
  snippet: studio-rpc-usb-uart  # ✅ Only Central has snippet

# Right (regular firmware)
- board: eyelash_sofle_right
  # No snippet  # ✅ Peripheral doesn't need it
```

**Verification Results**:
- [x] Left = Central (ZMK_SPLIT_ROLE_CENTRAL=y)
- [x] Right = Peripheral
- [x] Studio only enabled on Central
- [x] USB only enabled on Central

---

## Troubleshooting

### Issue 1: Studio Cannot Connect to Keyboard

#### Checklist

1. **Confirm you flashed the correct firmware**

After downloading `firmware.zip` from GitHub Actions, you should have multiple `.uf2` files:

```
firmware/
├── eyelash_sofle_studio_left-nice_view_gem-zmk.uf2  ← Flash to LEFT half
├── eyelash_sofle_right-nice_view_gem-zmk.uf2        ← Flash to RIGHT half
└── settings_reset.uf2                               ← For resetting settings
```

**Critical**: You MUST use the firmware with `studio` in the filename.

2. **Confirm USB connection**

- Connect **left half** to computer via USB data cable
- Verify computer recognizes device (should appear as CDC ACM serial device)

Linux/WSL verification:
```bash
ls /dev/ttyACM*
# Should show /dev/ttyACM0 or similar
```

Windows verification:
- Open Device Manager → Ports (COM & LPT)
- Should see "USB Serial Device (COMx)"

3. **Browser compatibility**

- Use Chrome or Edge (supports WebSerial API)
- Firefox does NOT support WebSerial

4. **Access Studio**

Visit https://zmk.studio and click "Connect", then select the corresponding serial port.

---

### Issue 2: Connected but Cannot Remap

#### Possible Causes

1. **Locking enabled** (currently disabled)

If you later modify the config to enable locking, you need to press `&studio_unlock`:
- Hold right thumb cluster `&mo 2`
- Simultaneously press TAB position key

2. **Firmware version mismatch**

Ensure Studio web version is compatible with firmware ZMK version (currently using v0.3.0).

---

### Issue 3: Local Build Fails

#### Local Build Commands (with Studio support)

```bash
# Initialize workspace (first time only)
west init -l config
west update

# Build left half (with Studio)
west build -d build/left -b eyelash_sofle_left -S studio-rpc-usb-uart -- \
  -DSHIELD=nice_view_gem \
  -DCONFIG_ZMK_STUDIO=y \
  -DCONFIG_ZMK_STUDIO_LOCKING=n

# Output: build/left/zephyr/zmk.uf2

# Build right half (regular firmware)
west build -d build/right -b eyelash_sofle_right -- \
  -DSHIELD=nice_view_gem

# Output: build/right/zephyr/zmk.uf2
```

#### Clean Build

```bash
# If encountering strange compilation errors, try cleaning
west build -d build/left -t pristine
```

---

## Configuration Summary Table

| Feature | Requirement | Current Config | Status |
|---------|-------------|----------------|--------|
| Physical Layout | `zmk,physical-layout` | `eyelash_sofle_layout` (64 keys) | ✅ |
| Chosen config | Reference physical-layout | `zmk,physical-layout = &eyelash_sofle_layout` | ✅ |
| Studio enabled | `CONFIG_ZMK_STUDIO=y` | Set (left.conf:2) | ✅ |
| USB-UART comm | `snippet: studio-rpc-usb-uart` | Set (build.yaml:9) | ✅ |
| CDC ACM device | `zephyr,cdc-acm-uart` | Defined (dtsi:109) | ✅ |
| USB functionality | `CONFIG_ZMK_USB=y` | Enabled (left_defconfig:40) | ✅ |
| Unlock key | `&studio_unlock` | Layer 2, Row2 Col1 | ✅ |
| Split roles | Central=Left, Peripheral=Right | Correctly configured | ✅ |
| Studio Central-only | Snippet only on Central build | Correctly configured | ✅ |

---

## Key Code Snippets

### Physical Layout Definition

**File**: `boards/arm/eyelash_sofle/eyelash_sofle-layouts.dtsi`

```devicetree
#include <physical_layouts.dtsi>

/ {
    eyelash_sofle_layout: eyelash_sofle_layout {
        compatible = "zmk,physical-layout";
        display-name = "Default Layout";

        kscan = <&kscan0>;

        keys  // 64 keys, each with 7 params: width height x y rotation rotation_x rotation_y
            = <&key_physical_attrs 100 100   0   0 0 0 0>
            , <&key_physical_attrs 100 100 100   0 0 0 0>
            // ... (64 lines total)
            ;
    };
};
```

### Matrix Transform Definition

**File**: `boards/arm/eyelash_sofle/eyelash_sofle.dtsi:62-68`

```devicetree
default_transform: keymap_transform_0 {
    compatible = "zmk,matrix-transform";
    columns = <16>;
    rows = <5>;
    map = <
        RC(0,0) RC(0,1) RC(0,2) RC(0,3) RC(0,4) RC(0,5)    RC(0,9) RC(0,10) RC(0,11) RC(0,12) RC(0,13) RC(0,14) RC(0,15)
        // ... (64 RC() mappings total)
    >;
};
```

### Studio Build Configuration

**File**: `build.yaml`

```yaml
---
include:
  # Right side - Regular firmware
  - board: eyelash_sofle_right
    shield: nice_view_gem

  # Left side - Studio firmware
  - board: eyelash_sofle_left
    shield: nice_view_gem
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO_LOCKING=n
    artifact-name: eyelash_sofle_studio_left

  # Settings reset firmware
  - board: eyelash_sofle_left
    shield: settings_reset
```

---

## Modification History

| Date | Item | Change | Reason |
|------|------|--------|--------|
| (No changes) | - | - | Configuration already correct, no modifications needed |

---

## Verification Steps

### Automated Verification (Recommended)

Trigger GitHub Actions build and download firmware:

```bash
# 1. Push code to trigger build
git push

# 2. Wait for GitHub Actions to complete
# Visit: https://github.com/[your-username]/zmk-sofle/actions

# 3. Download firmware.zip
# From latest successful workflow run artifacts

# 4. Flash firmware
# - Left half: eyelash_sofle_studio_left-nice_view_gem-zmk.uf2
# - Right half: eyelash_sofle_right-nice_view_gem-zmk.uf2
```

### Manual Verification

1. **Check Physical Layout**
```bash
grep -A 5 "compatible = \"zmk,physical-layout\"" boards/arm/eyelash_sofle/eyelash_sofle-layouts.dtsi
```

2. **Check Studio Config**
```bash
grep "CONFIG_ZMK_STUDIO" config/eyelash_sofle_left.conf
grep "snippet:" build.yaml
```

3. **Check Unlock Key**
```bash
grep "studio_unlock" boards/arm/eyelash_sofle/eyelash_sofle.keymap
```

---

## Conclusion

This ZMK firmware configuration **fully satisfies** all technical requirements for ZMK Studio:

1. ✅ Physical Layout correctly defined with matching key counts
2. ✅ Build system correctly configured for Studio support
3. ✅ USB-UART communication enabled
4. ✅ Unlock mechanism configured (locking disabled)
5. ✅ Split keyboard roles correctly assigned

**If Studio connection fails, the issue is**:
- Wrong firmware flashed (used regular firmware without `studio` in filename)
- USB connection problem (damaged cable, connected to right half, etc.)
- Browser doesn't support WebSerial (use Chrome/Edge)

**No code modifications are needed.**

---

## References

- [ZMK Studio Official Docs](https://zmk.dev/docs/features/studio)
- [Physical Layouts Guide](https://zmk.dev/docs/features/physical-layouts)
- [Split Keyboard Configuration](https://zmk.dev/docs/features/split-keyboards)
- [GitHub Actions Build](https://zmk.dev/docs/user-setup#github-actions)

---

*Generated: 2026-01-11*
*Audit Tool: Claude Code*
