enable-bl-tzk-recovery-restore
==============================

Purpose
-------
This zip re-enables the vendor-side init service that restores the BL
(bootloader) and TZK recovery sub-images on boot. It is the inverse of
disable-bl-tzk-recovery-restore. It does NOT touch the Android stock
recovery partition.

Mechanism
---------
- init.rc service:
    service vendor.flash_recovery2 /vendor/bin/install-vendor-recovery.sh
- Target file: /vendor/bin/install-vendor-recovery.sh.bak
- Action: rename /vendor/bin/install-vendor-recovery.sh.bak -> /vendor/bin/install-vendor-recovery.sh
- Permission: 0755, owner root:root
- restorecon is invoked only if available on the device.

After this zip runs, the next boot will execute install-vendor-recovery.sh
and the BL / TZK recovery sub-images will be re-applied to
/dev/block/by-name/bl_recovery and /dev/block/by-name/tzk_recovery via
the existing /vendor/bin/install_recovery helper.

Preserved (NOT touched)
-----------------------
- /vendor/bin/install_recovery
- /vendor/etc/boot/tzk_recovery.subimg
- /vendor/etc/boot/bl_recovery.subimg
- /dev/block/by-name/tzk_recovery
- /dev/block/by-name/bl_recovery
- /dev/block/by-name/recovery

Notes
-----
- This is an unsigned zip source directory. signapk was NOT executed.
- tools/busybox is NOT included. Only default /sbin/sh tools are used.
- If /vendor/bin/install_recovery or the .subimg files are missing, the
  actual restore at next boot may fail even though this zip succeeds.
- On devices that enforce vbmeta on the vendor partition, /vendor
  writes may not persist. Verify vbmeta state before applying.
