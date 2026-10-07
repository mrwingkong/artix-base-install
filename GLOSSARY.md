# Glossary (simple word list)

| Word | Plain meaning |
|------|----------------|
| **ISO** | A disk image file you write to a USB stick to boot the installer |
| **Partition** | A slice of your disk (like rooms in a house) |
| **Format** | Erase a partition and prepare it for files |
| **Mount** | Make a partition visible at a folder (e.g. `/mnt`) |
| **chroot** | “Enter” the new system so commands change *it*, not the live USB |
| **pacman** | The Artix/Arch package installer (`pacman -S` installs software) |
| **basestrap** | Artix tool that installs the base system onto `/mnt` |
| **OpenRC** | The service starter (instead of systemd). Uses `rc-update` / `rc-service` |
| **Wayland** | Modern display system (how windows are drawn). These guides prefer Wayland |
| **X11 / Xorg** | Older display system. Only needed if you choose it |
| **Compositor** | Draws windows (here: **KWin**; optional **LabWC** in the login menu) |
| **AUR** | Community package recipes for Arch-like systems. **yay** installs them |
| **sudo** | Run one command as the administrator |
| **wheel** | A group of users allowed to use `sudo` |
| **hostname** | Your computer’s name on the network |
| **locale** | Language and regional formats (e.g. `en_GB.UTF-8`) |
| **OSD** | On-screen display (the volume/brightness popup) |
| **`.xzm` module** | A read-only squashfs app pack (Porteus-style idea) |
| **UUID** | A unique ID for a partition. Prefer `UUID=` in `/etc/fstab` so the disk still mounts when the device name changes |
| **rsync** | A copy tool that keeps permissions; used to clone USB → NVMe |
