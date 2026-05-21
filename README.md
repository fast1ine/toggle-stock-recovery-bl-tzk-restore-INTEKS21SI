# Enable BL/TZK Recovery Restore for INTEKS21SI

## Purpose

This flashable ZIP re-enables the vendor BL/TZK recovery restore wrapper on INTEKS21SI.
It is the inverse of `disable-bl-tzk-recovery-restore`.

It renames:

```
/vendor/bin/install-vendor-recovery.sh.bak  →  /vendor/bin/install-vendor-recovery.sh
```

After this ZIP runs, the next boot will execute `install-vendor-recovery.sh` and the
BL / TZK recovery sub-images will be re-applied to `/dev/block/by-name/bl_recovery` and
`/dev/block/by-name/tzk_recovery` via the existing `/vendor/bin/install_recovery` helper.

The restored file is set to permission `0755`, owner `root:root`.
`restorecon` is invoked only if it is available on the device.

> **Note:** This ZIP does **not** touch the Android stock recovery partition
> (`/dev/block/by-name/recovery`). For the stock recovery auto-restore, use the
> separate `enable-stock-recovery-restore` ZIP.

---

## What This ZIP Changes

| Path | Action |
|------|--------|
| `/vendor/bin/install-vendor-recovery.sh.bak` | Renamed back to original |
| `/vendor/bin/install-vendor-recovery.sh` | Restored (the renamed-back original) |

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
- If `/vendor/bin/install_recovery` or the `.subimg` files are missing, the actual
  restore at next boot may fail even though this ZIP completes successfully.
- This ZIP does **not** include busybox. Only standard tools available at `/sbin/sh`
  are used.
- Test in TWRP first and verify the resulting filesystem state after installation.

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
git clone -b enable-bl-tzk-recovery-restore \
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

zip -r ../enable-bl-tzk-recovery-restore-unsigned.zip . \
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
  ../enable-bl-tzk-recovery-restore-unsigned.zip \
  ../enable-bl-tzk-recovery-restore-signed.zip
```

---

## How to Install in TWRP

1. Boot INTEKS21SI into TWRP recovery.
2. Tap **Install**.
3. Navigate to the signed ZIP file.
4. Swipe to confirm flash.
5. Reboot to System — do **not** reboot to Recovery immediately.

---

## Verification Commands After Installation

```bash
adb shell ls -la /vendor/bin/install-vendor-recovery.sh*
adb shell cat /vendor/bin/install-vendor-recovery.sh
adb shell ls -la /vendor/bin/install_recovery
adb shell ls -la /vendor/etc/boot/bl_recovery.subimg
adb shell ls -la /vendor/etc/boot/tzk_recovery.subimg
```

**Expected state:**

| Path | Expected |
|------|----------|
| `/vendor/bin/install-vendor-recovery.sh` | Must exist |
| `/vendor/bin/install-vendor-recovery.sh.bak` | Must **not** exist |
| `/vendor/bin/install_recovery` | Must still exist (untouched) |
| `/vendor/etc/boot/bl_recovery.subimg` | Must still exist (untouched) |
| `/vendor/etc/boot/tzk_recovery.subimg` | Must still exist (untouched) |

---

## Recovery / Revert Procedure

Flash the matching disable ZIP to undo this change:

```bash
# Boot into TWRP, then install:
disable-bl-tzk-recovery-restore-signed.zip
```

This renames `/vendor/bin/install-vendor-recovery.sh` back to
`/vendor/bin/install-vendor-recovery.sh.bak` and stops the auto-restore mechanism.

---

## Dangerous Operations That Must Not Be Performed

- Do **not** delete `/vendor/etc/boot/bl_recovery.subimg` or `tzk_recovery.subimg`.
- Do **not** wipe `/dev/block/by-name/bl_recovery` or `/dev/block/by-name/tzk_recovery` directly.
- Do **not** delete `/vendor/bin/install_recovery`.
- Do **not** flash this ZIP if vbmeta is enforcing on the vendor partition and `/vendor`
  is not writable from TWRP.
- Do **not** run `fastboot oem` commands or `fastboot flash` on these partitions
  while attempting to use this ZIP — they are separate operations.
