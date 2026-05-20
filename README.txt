disable-bl-tzk-recovery-restore
===============================

Purpose
-------
This zip disables the vendor-side init service that restores the BL
(bootloader) and TZK recovery sub-images on boot. It does NOT touch the
Android stock recovery partition. That is handled by a separate zip
(disable-stock-recovery-restore).

Mechanism
---------
- init.rc service:
    service vendor.flash_recovery2 /vendor/bin/install-vendor-recovery.sh
- Target file: /vendor/bin/install-vendor-recovery.sh
- Action: rename /vendor/bin/install-vendor-recovery.sh -> /vendor/bin/install-vendor-recovery.sh.bak

install-vendor-recovery.sh is the shell wrapper that writes
/vendor/etc/boot/tzk_recovery.subimg to /dev/block/by-name/tzk_recovery
and /vendor/etc/boot/bl_recovery.subimg to /dev/block/by-name/bl_recovery
(and likely invokes the /vendor/bin/install_recovery helper). Renaming
just this wrapper to .bak is enough to stop the auto-restore.

This path is independent of /dev/block/by-name/recovery.

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
- On devices that enforce vbmeta on the vendor partition, /vendor
  writes may not persist. Verify vbmeta state before applying.
- To revert, flash the enable-bl-tzk-recovery-restore zip.
