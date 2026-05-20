# Disable BL/TZK Recovery Restore for INTEKS21SI

## Purpose

This flashable ZIP disables the vendor BL/TZK recovery restore wrapper on INTEKS21SI.
It renames the vendor init.rc service script:

```
/vendor/bin/install-vendor-recovery.sh  →  /vendor/bin/install-vendor-recovery.sh.bak
```

The init.rc service `vendor.flash_recovery2` calls `install-vendor-recovery.sh` on every boot.
That script writes `/vendor/etc/boot/tzk_recovery.subimg` to `/dev/block/by-name/tzk_recovery`
and `/vendor/etc/boot/bl_recovery.subimg` to `/dev/block/by-name/bl_recovery` via the
`/vendor/bin/install_recovery` helper. Renaming the script to `.bak` is sufficient to
stop the auto-restore without touching the sub-image files or their partitions.

> **Note:** This is **not** the same as the Android stock recovery auto-restore path for
> `/dev/block/by-name/recovery`. This ZIP targets the BL/TZK recovery restore wrapper
> only. Use it only if you specifically need to disable that mechanism.
> For the stock recovery auto-restore, use the separate `disable-stock-recovery-restore` ZIP.

---

## What This ZIP Changes

| Path | Action |
|------|--------|
| `/vendor/bin/install-vendor-recovery.sh` | Renamed to `.bak` |
| `/vendor/bin/install-vendor-recovery.sh.bak` | Created (the renamed original) |

---

## What This ZIP Does Not Touch

- `/vendor/bin/install_recovery`
- `/vendor/etc/boot/bl_recovery.subimg`
- `/vendor/etc/boot/tzk_recovery.subimg`
- `/dev/block/by-name/bl_recovery`
- `/dev/block/by-name/tzk_recovery`
- `/dev/block/by-name/recovery`

---

## Important Notes

- If vbmeta / dm-verity is enforcing on the vendor partition of your device, changes to
  `/vendor` may not persist after reboot. Verify vbmeta state before applying.
- This ZIP does **not** include busybox. Only standard tools available at `/sbin/sh`
  are used.
- Test in TWRP first and verify the resulting filesystem state after installation.
- To revert, flash the matching `enable-bl-tzk-recovery-restore` ZIP.

---

## Required Packages

```bash
sudo apt update
sudo apt install -y git zip unzip signapk
```

If `signapk` is not available from your package manager, install it separately or
use the `signapk` binary from an AOSP or TWRP build tree that is already in your `PATH`.

---

## Clone Instructions

```bash
git clone -b disable-bl-tzk-recovery-restore \
  https://github.com/fast1ine/toggle-stock-recovery-bl-tzk-restore-INTEKS21SI.git

cd toggle-stock-recovery-bl-tzk-restore-INTEKS21SI
```

---

## How to Create the Unsigned ZIP

Enter the repository directory first. Zip only the **contents** — do not include the
repository directory itself as a top-level entry.

The ZIP root must contain these files directly:

```
META-INF/com/google/android/update-binary
META-INF/com/google/android/updater-script
README.txt
```

`README.md` should **not** be included inside the flashable ZIP.
The ZIP must **not** contain an extra top-level repository or branch directory.

```bash
cd toggle-stock-recovery-bl-tzk-restore-INTEKS21SI

zip -r ../disable-bl-tzk-recovery-restore-unsigned.zip . \
  -x "*.git*" \
  -x "*.md*" \
  -x "*.DS_Store" \
  -x "__MACOSX/*"
```

---

## How to Sign the ZIP with signapk

```bash
signapk \
  --min-sdk-version 11 \
  /path/to/certificate.x509.pem \
  /path/to/privatekey.pk8 \
  ../disable-bl-tzk-recovery-restore-unsigned.zip \
  ../disable-bl-tzk-recovery-restore-signed.zip
```

---

## How to Install in TWRP

1. Boot INTEKS21SI into TWRP recovery.
2. Tap **Install**.
3. Navigate to the signed ZIP file.
4. Swipe to confirm flash.
5. Reboot to System — do **not** reboot to Recovery.

---

## Verification Commands After Installation

```bash
adb shell ls -la /vendor/bin/install-vendor-recovery.sh*
adb shell ls -la /vendor/bin/install_recovery
adb shell ls -la /vendor/etc/boot/bl_recovery.subimg
adb shell ls -la /vendor/etc/boot/tzk_recovery.subimg
```

**Expected state:**

| Path | Expected |
|------|----------|
| `/vendor/bin/install-vendor-recovery.sh` | Must **not** exist |
| `/vendor/bin/install-vendor-recovery.sh.bak` | Must exist |
| `/vendor/bin/install_recovery` | Must still exist (untouched) |
| `/vendor/etc/boot/bl_recovery.subimg` | Must still exist (untouched) |
| `/vendor/etc/boot/tzk_recovery.subimg` | Must still exist (untouched) |

---

## Recovery / Revert Procedure

Flash the matching enable ZIP to undo this change:

```bash
# Boot into TWRP, then install:
enable-bl-tzk-recovery-restore-signed.zip
```

This renames `/vendor/bin/install-vendor-recovery.sh.bak` back to
`/vendor/bin/install-vendor-recovery.sh` and restores the auto-restore mechanism.

---

## Dangerous Operations That Must Not Be Performed

- Do **not** delete `/vendor/etc/boot/bl_recovery.subimg` or `tzk_recovery.subimg`.
- Do **not** wipe `/dev/block/by-name/bl_recovery` or `/dev/block/by-name/tzk_recovery` directly.
- Do **not** delete `/vendor/bin/install_recovery`.
- Do **not** flash this ZIP if vbmeta is enforcing on the vendor partition and `/vendor`
  is not writable from TWRP.
- Do **not** run `fastboot oem` commands or `fastboot flash` on these partitions
  while attempting to use this ZIP — they are separate operations.
