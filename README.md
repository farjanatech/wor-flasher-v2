# WoR-Flasher v2

**Unofficial Raspberry Pi OS / Wayland compatibility fork of [Botspot/wor-flasher](https://github.com/Botspot/wor-flasher).**

WoR-Flasher v2 keeps the original WoR-Flasher workflow while adding reliability fixes for newer Raspberry Pi OS releases, with particular attention to **Raspberry Pi 5**, **Wayland/XWayland**, ISO extraction, and SD/eMMC partition detection.

> This is an unofficial fork maintained by **farjanatech**. It is not an official release from Botspot or the Windows on Raspberry project.

## What v2 fixes

WoR-Flasher v2 includes fixes for problems seen on current Raspberry Pi OS installations:

- Native **Wayland-compatible GUI startup**. Raspberry Pi OS stays on Wayland; WoR/YAD uses XWayland where necessary.
- Suppresses the harmless GTK accessibility-bus warning with `GTK_A11Y=none`.
- Cleans stale/incomplete ISO extraction files before retrying.
- Copies Windows ISO files without preserving read-only mode bits.
- Fixes repeated errors such as:
  - `cp: cannot create regular file ... Permission denied`
- Avoids:
  - `umount: bad usage`
- Runs `partprobe` and `udevadm settle` after partition creation.
- Waits for `/dev/mmcblk0p1`, `/dev/mmcblk0p2`, `/dev/sdX1`, etc. before formatting.
- Fixes:
  - `Partition 1: , Partition 2:`
  - `mkfs.fat: unable to open : No such file or directory`
- Installs the additional `parted` and `xwayland` packages required by the v2 compatibility fixes.

---

# Installation tutorial

## Requirements

Recommended:

- Raspberry Pi 5, Pi 4/400, or another model supported by WoR-Flasher
- Raspberry Pi OS
- Internet connection
- A destination SD card, USB drive, or other supported target
- At least **25 GB** for a self-installing Windows drive
- For a custom ISO: a **Windows 10/11 ARM64 ISO**

> **Warning:** WoR-Flasher repartitions and formats the target drive. Everything on the selected target device will be erased.

## 1. Install Git

Open a Terminal on Raspberry Pi OS:

```bash
sudo apt update
sudo apt install -y git
```

## 2. Download WoR-Flasher v2

```bash
cd ~
git clone https://github.com/farjanatech/wor-flasher-v2.git
cd wor-flasher-v2
```

Make the scripts executable:

```bash
chmod +x install-wor.sh install-wor-gui.sh terminal-run
```

## 3. Start the graphical installer

Run:

```bash
cd ~/wor-flasher-v2
./install-wor-gui.sh
```

The first run automatically installs the required Linux packages.

### Raspberry Pi OS Wayland

You do **not** need to switch Raspberry Pi OS to an X11 desktop session.

WoR-Flasher v2 detects Wayland and runs the GTK/YAD interface through XWayland while leaving the rest of the desktop on Wayland.

You can check your desktop session with:

```bash
echo "$XDG_SESSION_TYPE"
```

Seeing:

```text
wayland
```

is fine.

---

# Flash Windows 11 ARM on Raspberry Pi 5

## 1. Choose Windows and Pi model

In the first WoR-Flasher window choose:

- **Windows 11**
- **Pi5**

Then continue.

## 2. Choose the Windows source

You can let WoR-Flasher obtain Windows automatically, or use your own ARM64 ISO.

To use your own ISO, choose:

**More options → Use a Windows ISO file**

Then enter the full path, for example:

```text
/home/pi/Desktop/uupdump/Windows11_ARM64.iso
```

The ISO must be an **ARM64/AArch64** Windows image. An ordinary x64/AMD64 Windows ISO will not work for Raspberry Pi.

## 3. Choose language

Select your Windows language.

Example:

```text
en-us
```

## 4. Select the destination drive

Carefully choose the SD card/USB drive that Windows will be flashed to.

Typical Raspberry Pi storage names include:

```text
/dev/mmcblk0
/dev/sda
/dev/sdb
```

**Double-check this selection. The chosen device will be repartitioned and erased.**

If the destination is large enough, WoR-Flasher can create a drive capable of installing Windows onto itself.

## 5. Start flashing

Review the configuration and click **Flash**.

When using an ISO, a correctly running v2 installation should show:

```text
Preparing ISO extraction workspace
Mounting ...
Copying files from ISO file ...
```

Later, after the target partition table is created, you should see:

```text
Generating partitions
Waiting for partition devices
Generating filesystems
```

For an SD-card style device, a successful detection looks similar to:

```text
Partition 1: /dev/mmcblk0p1, Partition 2: /dev/mmcblk0p2
```

The exact device name depends on the storage you selected.

---

# Using an existing Windows ARM64 ISO

WoR-Flasher stores extracted Windows files under:

```text
~/wor-flasher-files
```

v2 automatically removes incomplete generated ISO extraction output when starting a new ISO extraction. Your original ISO is not deleted.

For example, if an interrupted run previously left:

```text
~/wor-flasher-files/winfiles_from_iso_.../bootpart
```

v2 rebuilds that generated working directory rather than attempting to overwrite stale read-only files.

This addresses errors such as:

```text
cp: cannot create regular file '.../bootpart/boot/fonts/...': Permission denied
```

---

# Running WoR-Flasher v2 again

Use:

```bash
cd ~/wor-flasher-v2
./install-wor-gui.sh
```

For the terminal interface:

```bash
cd ~/wor-flasher-v2
./install-wor.sh
```

---

# Updating WoR-Flasher v2

To manually update this fork:

```bash
cd ~/wor-flasher-v2
git pull
```

Then run:

```bash
./install-wor-gui.sh
```

The v2 auto-update check follows this fork rather than replacing the compatibility code with the upstream repository.

---

# Troubleshooting

## Confirm you are using v2

Run:

```bash
grep -nE 'Preparing ISO extraction workspace|Waiting for partition devices|WOR_PARTITIONS' ~/wor-flasher-v2/install-wor.sh
```

You should see matches for all three v2 fixes.

For the GUI Wayland fix:

```bash
grep -n 'WoR-Flasher v2 Wayland compatibility' ~/wor-flasher-v2/install-wor-gui.sh
```

## ISO copy still reports Permission denied

Check that you cloned this repository rather than the original upstream repository:

```bash
cd ~/wor-flasher-v2
git remote -v
```

The `origin` URL should point to:

```text
https://github.com/farjanatech/wor-flasher-v2
```

## Partition names are empty

A successful v2 run should print:

```text
Waiting for partition devices
```

before:

```text
Generating filesystems
```

If necessary, inspect the device manually:

```bash
sudo partprobe /dev/mmcblk0
sudo udevadm settle
lsblk -o NAME,TYPE,SIZE,FSTYPE /dev/mmcblk0
```

Replace `/dev/mmcblk0` with the actual target device.

---

# v2 technical changes

See:

- [`CHANGELOG-V2.md`](CHANGELOG-V2.md)
- [`V2-NOTICE.md`](V2-NOTICE.md)
- [`VERSION`](VERSION)

The main compatibility changes are in:

- `install-wor.sh`
- `install-wor-gui.sh`

---

# Upstream project

WoR-Flasher v2 is based on the original project by **Botspot**:

https://github.com/Botspot/wor-flasher

The original project history and attribution are preserved through GitHub's fork relationship.

For Windows on Raspberry project information and drivers, see the upstream resources referenced by the original WoR-Flasher project.

## Disclaimer

This software performs destructive disk operations. Carefully verify the selected destination device before flashing.

Windows licensing and activation are separate from this tool. This repository does not distribute a Windows license.