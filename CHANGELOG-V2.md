# WoR-Flasher v2 changelog

## v2.0.0

Compatibility/reliability release for Raspberry Pi OS, particularly Raspberry Pi 5.

### Fixes

- Run the graphical interface through XWayland when the desktop session is Wayland.
- Set `GTK_A11Y=none` to avoid the missing accessibility-bus warning.
- Clean an incomplete ISO extraction workspace before re-extracting.
- Copy Windows ISO boot/EFI/WIM files without preserving read-only mode bits.
- Avoid calling `umount` with no partition arguments.
- Run `partprobe` and `udevadm settle` after recreating the partition table.
- Retry partition discovery before calling `mkfs.fat`/`mkfs.exfat`.
- Fail with useful `lsblk` output if partition device nodes still do not appear.

### Tested failure modes addressed

- `cp: cannot create regular file ... Permission denied`
- `umount: bad usage`
- `Partition 1: , Partition 2:`
- `mkfs.fat: unable to open : No such file or directory`
- YAD X11/GDK warnings when Raspberry Pi OS uses Wayland