# 02 — Desktop and audio

Finish the chroot install (packages, GRUB, services), reboot, then set up PipeWire, PowerDevil, and keyboard.

**Part A** is still as **root inside** `artix-chroot /mnt`.  
**Part B** is after reboot, as your normal user.

Package meanings: [packages.md](packages.md).

---

# Part A — Still in the chroot

## Step 1 — Desktop and core packages

LabWC is included **only** so the LXQt session menu can offer another compositor. Later guides focus on **KWin**.

> ℹ️ Intel GPU/audio firmware packages are in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)** (hardware notes), not here.

```bash
pacman -Syu grub efibootmgr \
  lxqt lxqt-wayland-session \
  labwc kwin xorg-xwayland qt6-wayland \
  kscreen kscreenlocker plasma-desktop systemsettings powerdevil layer-shell-qt \
  gvfs mesa mesa-utils vulkan-tools \
  networkmanager networkmanager-openrc network-manager-applet \
  blueman bluez bluez-openrc bluez-utils \
  alsa-utils pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber wireplumber-openrc pavucontrol-qt
```

**What this does:** Installs the bootloader tools, LXQt, KWin, networking, Bluetooth, and PipeWire audio stack.

**You should now see:** pacman finish without errors.

---

## Step 2 — Install GRUB (UEFI + legacy BIOS)

> ⚠️ **Replace `/dev/sda` if your disk name is different.**

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Artix --removable
```

**What this does:** Installs GRUB for UEFI.

```bash
grub-install --target=i386-pc --recheck /dev/sda
```

**What this does:** Also installs GRUB for old BIOS-style boots (handy for USB portability).

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

**What this does:** Writes the boot menu.

---

## Step 3 — Basic OpenRC services

```bash
rc-update add NetworkManager default
```

**What this does:** Starts NetworkManager every boot.

```bash
rc-update add bluetoothd default
```

**What this does:** Starts Bluetooth every boot.

> Do **not** enable `fprintd` here — that is in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)** (fingerprint guide).

---

## Step 4 — Leave chroot and reboot

```bash
exit
```

**What this does:** Leaves the chroot.

```bash
umount -R /mnt
```

**What this does:** Unmounts the installed system.

```bash
reboot
```

**What this does:** Restarts the computer. Remove the live USB when asked (or boot from the disk).

**You should now see:** A text login (TTY). Log in as the user you created, then start the desktop:

```bash
startlxqtwayland
```

**What this does:** Starts LXQt on Wayland.

(If you installed X11 packages on purpose, you could use `startx` instead.)

---

# Part B — After first login (your normal user)

## Step 5 — Rank mirrors again

```bash
sudo pacman -Syy
```

**What this does:** Refreshes package lists with sudo.

```bash
sudo pacman -S --needed pacman-contrib
```

**What this does:** Ensures `rankmirrors` is installed.

```bash
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist-artix
```

**What this does:** Backs up the mirror list.

```bash
sudo rankmirrors /etc/pacman.d/mirrorlist-artix | sudo tee /etc/pacman.d/mirrorlist
```

**What this does:** Writes a faster mirror list.

---

## Step 6 — Extra packages for a full desktop

> ℹ️ Packages like `fprintd` and `iio-sensor-proxy` are for **[artix-post-install](https://github.com/mrwingkong/artix-post-install)** — install them there when needed.

```bash
sudo pacman -S --needed \
  qt6-tools xdg-desktop-portal xdg-desktop-portal-kde \
  swaybg swaync plasma-keyboard libnotify brightnessctl sxhkd \
  python-pyqt6 \
  libarchive libstatgrab git wget zip unzip xz \
  btrfs-progs ntfs-3g exfatprogs xfsprogs e2fsprogs f2fs-tools dosfstools \
  squashfs-tools sed mujs clang cmake ninja qemu-base libbsd
```

**What this does:** Installs portals, notification/OSD helpers, wallpaper tools, and useful disk/build tools.

---

## Step 7 — PipeWire autostart (system-wide)

```bash
sudo mkdir -p /etc/xdg/autostart
```

**What this does:** Creates the system autostart folder.

```bash
sudo tee /etc/xdg/autostart/pipewire.desktop > /dev/null << 'END'
[Desktop Entry]
Type=Application
Name=PipeWire
Comment=Multimedia server
Exec=/usr/bin/pipewire
Hidden=false
NoDisplay=true
X-GNOME-Autostart-enabled=true
END
```

**What this does:** Starts PipeWire when you log into a desktop.

```bash
sudo tee /etc/xdg/autostart/pipewire-pulse.desktop > /dev/null << 'END'
[Desktop Entry]
Type=Application
Name=PipeWire Pulse
Comment=PulseAudio replacement
Exec=/usr/bin/pipewire-pulse
Hidden=false
NoDisplay=true
X-GNOME-Autostart-enabled=true
END
```

**What this does:** Starts the PulseAudio-compatible PipeWire service at login.

```bash
sudo tee /etc/xdg/autostart/wireplumber.desktop > /dev/null << 'END'
[Desktop Entry]
Type=Application
Name=WirePlumber
Comment=Session/policy manager for PipeWire
Exec=/usr/bin/wireplumber
Hidden=false
NoDisplay=true
X-GNOME-Autostart-enabled=true
END
```

**What this does:** Starts WirePlumber (device routing) at login.

---

## Step 8 — PowerDevil (power management)

```bash
mkdir -p ~/.config/autostart
```

**What this does:** Creates your user autostart folder.

```bash
cat > ~/.config/autostart/powerdevil.desktop << 'END'
[Desktop Entry]
Type=Application
Name=PowerDevil
Exec=/usr/lib/org_kde_powerdevil
X-GNOME-Autostart-enabled=true
END
```

**What this does:** Starts PowerDevil every login (idle dim, lid, suspend).

```bash
/usr/lib/org_kde_powerdevil &
```

**What this does:** Starts PowerDevil for the current session.

```bash
ps aux | grep -i powerdevil | grep -v grep
```

**What this does:** Checks that PowerDevil is running.

**You should now see:** A line with `org_kde_powerdevil`.

Log out and log back in so autostart is clean. Then open settings:

```bash
systemsettings
```

**What this does:** Opens System Settings — set Energy Saving how you like.

### Disable PowerDevil brightness shortcuts (important later)

Custom brightness scripts in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)** will own the brightness keys.

1. Open **System Settings → Shortcuts**
2. Search for `brightness`
3. Set these four actions to **None**:
   - Increase Screen Brightness
   - Decrease Screen Brightness
   - Increase Screen Brightness by 1%
   - Decrease Screen Brightness by 1%

PowerDevil still manages idle, lock, lid and battery — only the brightness keys are released.

---

## Step 9 — Keyboard layout (UK example)

> ⚠️ **UK example.** Change keymap / layout codes for your country.

### Console / TTY

```bash
sudo nano /etc/vconsole.conf
```

**What this does:** Opens the console keyboard file.

Set the file to (UK example):

```text
KEYMAP=uk
```

### Wayland + XWayland

```bash
mkdir -p ~/.config/environment.d
```

**What this does:** Creates the environment drop-in folder.

```bash
cat > ~/.config/environment.d/00-keyboard.conf << 'END'
XKB_DEFAULT_LAYOUT=gb
XKB_DEFAULT_MODEL=pc105
END
```

**What this does:** Sets the graphical keyboard layout (UK example uses `gb`).

### KWin layout

```bash
cat > ~/.config/kxkbrc << 'END'
[Layout]
LayoutList=gb
Use=true
END
```

**What this does:** Tells KWin which layout to use.

Log out and back in (or reboot).

**You should now see:** For UK layout, Shift+2 types `"` and Shift+' types `@`.

---

## Step 10 — Plasma virtual keyboard (optional, helpful on tablets)

```bash
sudo pacman -S --needed plasma-keyboard
```

**What this does:** Installs the on-screen keyboard (if not already installed).

```bash
kwriteconfig6 --file kwinrc --group Wayland --key InputMethod "/usr/share/applications/org.kde.plasma.keyboard.desktop"
```

**What this does:** Points KWin at Plasma Keyboard.

```bash
ls /usr/share/applications/*plasma*keyboard*
```

**What this does:** Checks the desktop file exists.

Log out and back in. In tablet/touch mode, tap a text field — the keyboard should appear.

Optional (only if it never appears on touch):

```bash
echo 'KWIN_IM_SHOW_ALWAYS=1' >> ~/.config/environment.d/plasma-keyboard.conf
```

**What this does:** Forces the input method UI to show more readily.

> ThinkPad tablet-mode tricks for QTerminal are in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**.

---

## Step 11 — Set KWin as default compositor (recommended)

```bash
mkdir -p ~/.config/lxqt
```

**What this does:** Creates the LXQt config folder.

```bash
cat > ~/.config/lxqt/session.conf << 'END'
[General]
compositor=kwin_wayland
END
```

**What this does:** Makes KWin the default compositor for LXQt Wayland.

---

## Step 12 — PATH for local scripts

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

**What this does:** Puts your `~/.local/bin` scripts on the PATH.

```bash
source ~/.bashrc
```

**What this does:** Reloads your shell settings now.

```bash
mkdir -p ~/.local/bin
```

**What this does:** Creates the folder used by later OSD scripts.

---

## Next

1. Optional portable USB / NVMe notes: [03-portable-usb-nvme.md](03-portable-usb-nvme.md)
2. Optional desktop polish + hardware: **[artix-post-install](https://github.com/mrwingkong/artix-post-install)** (notifications, OSDs, theme, extras, ThinkPad notes)
3. Optional modular apps: **[artix-portix](https://github.com/mrwingkong/artix-portix)** (`.xzm` / `pman`)

> Intel SOF **volume-reset** and other machine-specific audio fixes are **not** in this base guide — see **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**.

---

*Artix OpenRC only — no systemd.*
