# Packages — base install

Only packages used in this **base** guide (`guide/01`–`02` and portable notes).  
Desktop polish and ThinkPad packages live in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**.  
Module / `.xzm` tools live in **[artix-portix](https://github.com/mrwingkong/artix-portix)**.

## Required — core desktop

| Package | What it is | Why needed | Required? |
|---------|------------|------------|-----------|
| `base` | Core system set | Bare OS pieces | required |
| `base-devel` | Build tools | Build AUR apps / themes | required |
| `openrc` | Init system | Starts services (not systemd) | required |
| `elogind-openrc` | Login/session helper | Desktop login without systemd | required |
| `linux` | The kernel | Runs the computer | required |
| `linux-headers` | Kernel headers | Building drivers/modules | required |
| `linux-firmware` | Device firmware | Wi-Fi and other hardware | required |
| `nano` | Text editor | Edit config files in the guides | required |
| `pacman-contrib` | Extra pacman tools | `rankmirrors` for faster downloads | required |
| `grub` | Bootloader | Starts Artix after power-on | required |
| `efibootmgr` | UEFI boot tool | Install GRUB on UEFI | required |
| `lxqt` | LXQt desktop | The desktop you use | required |
| `lxqt-wayland-session` | LXQt on Wayland | Wayland session | required |
| `kwin` | KDE compositor | Draws windows (main path) | required |
| `xorg-xwayland` | X apps on Wayland | Run older X11 apps | required |
| `qt6-wayland` | Qt Wayland support | Qt apps on Wayland | required |
| `kscreen` | Display settings | Screens / brightness tools | required |
| `kscreenlocker` | Lock screen | Lock the session | required |
| `plasma-desktop` | Plasma desktop pieces | Supports PowerDevil stack used here | required |
| `systemsettings` | Settings app | Power, shortcuts, decorations | required |
| `powerdevil` | Power manager | Dim, sleep, lid close | required |
| `layer-shell-qt` | Wayland layer-shell for Qt | Panels / overlays | required |
| `gvfs` | Virtual filesystems | Trash, devices, network shares | required |
| `mesa` | Common GPU drivers | Draw the desktop | required |
| `mesa-utils` | GL test tools | Quick graphics check | required |
| `networkmanager` | Network service | Wi-Fi / Ethernet | required |
| `networkmanager-openrc` | OpenRC unit for NM | Start NM on boot | required |
| `network-manager-applet` | Network tray icon | Click to connect | required |
| `blueman` | Bluetooth app | Manage Bluetooth | required |
| `bluez` | Bluetooth stack | Bluetooth support | required |
| `bluez-openrc` | OpenRC Bluetooth | Start Bluetooth on boot | required |
| `bluez-utils` | Bluetooth tools | Pairing / debug | required |
| `alsa-utils` | Sound mixer tools | `amixer` / ALSA controls | required |
| `pipewire` | Audio/video server | Sound on the desktop | required |
| `pipewire-alsa` | ALSA bridge | App compatibility | required |
| `pipewire-pulse` | PulseAudio bridge | App compatibility | required |
| `pipewire-jack` | JACK bridge | Pro-audio compatibility | required |
| `wireplumber` | PipeWire manager | Routes devices | required |
| `wireplumber-openrc` | OpenRC WirePlumber bits | Service integration | required |
| `pavucontrol-qt` | Volume mixer GUI | Choose devices / levels | required |

## Optional — used in base steps / portable notes

| Package | What it is | Why needed | Required? |
|---------|------------|------------|-----------|
| `labwc` | Light compositor | Optional choice in LXQt session menu | optional |
| `vulkan-tools` | Vulkan tests | Check Vulkan | optional |
| `qt6-tools` | Qt tools (`qdbus6`) | Helpers for settings | optional |
| `xdg-desktop-portal` | Desktop portals | App dialogs / screenshare | optional |
| `xdg-desktop-portal-kde` | KDE portal backend | Works with KWin | optional |
| `plasma-keyboard` | On-screen keyboard | Touch / tablet typing | optional |
| `git` | Version control | Clone related repos | optional |
| `wget` | Downloader | Fetch files | optional |
| `rsync` | File copy tool | USB → NVMe clone steps | optional† |
| `btrfs-progs` | btrfs tools | Guide uses btrfs for root/home | optional† |
| `dosfstools` | FAT tools | EFI partition | optional† |
| `ntfs-3g` | NTFS support | Windows disks | optional |
| `exfatprogs` | exFAT support | USB sticks | optional |
| `e2fsprogs` | ext tools | Common Linux disks | optional |

†Needed if you follow this guide’s FAT + btrfs layout, or the portable clone steps.
