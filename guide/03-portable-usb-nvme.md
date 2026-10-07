# 03 — Portable USB and NVMe

Keep a **full install** on a USB stick now, or an internal NVMe later.  
This is **not** a live ISO with persistence — you update with normal `pacman -Syu`.

Do this **after** [01-disk-and-base.md](01-disk-and-base.md) and [02-desktop-and-audio.md](02-desktop-and-audio.md).

Machine-specific drivers (GPU, SOF audio, fingerprint, tablet) live in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**, not here.

---

## Part A — Removable-root safety

A full install on USB can corrupt if the stick is yanked or lost while the system is still writing (including some standby cases).

### Rule 1 — Prefer shut down before unplug

```bash
sudo poweroff
```

**What this does:** Stops the system cleanly and flushes disks.

**You should now see:** The machine powers off. Then unplug the USB.

### Rule 2 — If you must leave it plugged in overnight

Leave the stick plugged in. Do **not** yank it while the screen is asleep.

### Rule 3 — Emergency sync (only if the desktop is still up)

```bash
sync
```

**What this does:** Asks the kernel to flush pending writes to disk.

Wait a few seconds, then shut down if you can. Sync alone is **not** as safe as a full shutdown.

> ⚠️ **Suspend / sleep on a USB root is risky.** Prefer shut down when you will move the stick between PCs.

---

## Part B — UUID boot check

The base install already writes `/etc/fstab` with **UUIDs** (`fstabgen -U`) and installs GRUB with `--removable` so the same stick can boot on other UEFI machines.

### Step 1 — See your partition UUIDs

```bash
lsblk -f
```

**What this does:** Lists filesystems and UUID values.

**You should now see:** Lines for your EFI (`vfat`) and root/home (`btrfs`) with UUID columns filled.

### Step 2 — Confirm fstab uses UUID=

```bash
cat /etc/fstab
```

**What this does:** Shows how partitions are mounted at boot.

**You should now see:** Lines starting with `UUID=` for `/`, `/home`, and `/boot` (not bare `/dev/sda3` style names).

> ⚠️ If you ever rewrote fstab by hand with `/dev/sdX` names, change them back to `UUID=` from `lsblk -f` before moving the disk to another PC.

### Step 3 — On another PC

1. Plug in the USB (or NVMe via adapter).
2. Open that PC’s firmware boot menu.
3. Pick the Artix / USB EFI entry (or the removable EFI path).

**You should now see:** The same system boots. If drivers for the *new* hardware are missing, continue in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**.

---

## Part C — USB → NVMe clone (same layout)

Goal: copy your running full install from a USB disk onto an internal NVMe so the NVMe boots the same way.  
You can later pull that NVMe and plug it into another machine (direct or USB adapter).

> ⚠️ **This wipes the target NVMe.** Double-check disk names with `lsblk`.  
> ⚠️ **Change `/dev/sda` (source USB) and `/dev/nvme0n1` (target) if `lsblk` shows different names.**

### Step 1 — Boot a live Artix OpenRC USB

Use a **different** live installer stick if your main system USB is the source you will copy from.  
Or boot the live ISO and attach the source USB as a second disk.

### Step 2 — See disks

```bash
lsblk
```

**What this does:** Shows source and target disks so you pick the right ones.

### Step 3 — Partition the NVMe like the USB

Match the same layout as [01-disk-and-base.md](01-disk-and-base.md) (EFI + BIOS boot + root + home).

```bash
cfdisk /dev/nvme0n1
```

**What this does:** Opens the partition editor on the target NVMe.

### Step 4 — Format the NVMe partitions

> ⚠️ **Change partition names if `lsblk` differs** (for example `nvme0n1p1`).

```bash
mkfs.fat -F32 /dev/nvme0n1p1
```

**What this does:** Formats the new EFI partition.

```bash
mkfs.btrfs -f /dev/nvme0n1p3
```

**What this does:** Formats the new root partition.

```bash
mkfs.btrfs -f /dev/nvme0n1p4
```

**What this does:** Formats the new home partition.

### Step 5 — Mount target and source

```bash
mount /dev/nvme0n1p3 /mnt
```

**What this does:** Mounts new root at `/mnt`.

```bash
mkdir -p /mnt/home /mnt/boot
```

**What this does:** Creates home and boot mount points.

```bash
mount /dev/nvme0n1p4 /mnt/home
```

**What this does:** Mounts new home.

```bash
mount /dev/nvme0n1p1 /mnt/boot
```

**What this does:** Mounts new EFI at `/mnt/boot`.

Mount the **source** USB partitions read-only somewhere safe (example: `/mnt/src`):

```bash
mkdir -p /mnt/src /mnt/src/home /mnt/src/boot
```

**What this does:** Creates mount points for the source system.

> ⚠️ **Change `/dev/sda3` / `/dev/sda4` / `/dev/sda1` to your source partitions.**

```bash
mount -o ro /dev/sda3 /mnt/src
```

**What this does:** Mounts source root read-only.

```bash
mount -o ro /dev/sda4 /mnt/src/home
```

**What this does:** Mounts source home read-only.

```bash
mount -o ro /dev/sda1 /mnt/src/boot
```

**What this does:** Mounts source EFI read-only.

### Step 6 — Copy files with rsync

```bash
pacman -Sy --needed rsync
```

**What this does:** Installs `rsync` on the live system if needed.

```bash
rsync -aAXv --info=progress2 /mnt/src/ /mnt/ --exclude=/mnt/src
```

**What this does:** Copies the whole system onto the NVMe (permissions and most attributes kept).

**You should now see:** A progress display, then a finished copy with no error spam.

### Step 7 — Rewrite fstab for the new UUIDs

```bash
bash -c 'fstabgen -U /mnt > /mnt/etc/fstab'
```

**What this does:** Writes UUID mount lines for the **NVMe** partitions.

```bash
cat /mnt/etc/fstab
```

**What this does:** Lets you check the new file.

**You should now see:** `UUID=` lines that match `lsblk -f` for the NVMe, not the old USB.

### Step 8 — Reinstall GRUB on the NVMe

```bash
artix-chroot /mnt
```

**What this does:** Enters the copied system on the NVMe.

> ⚠️ **Change `/dev/nvme0n1` if needed.**

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Artix --removable
```

**What this does:** Installs UEFI GRUB on the NVMe EFI partition.

```bash
grub-install --target=i386-pc --recheck /dev/nvme0n1
```

**What this does:** Also installs BIOS GRUB on the NVMe (handy for adapters / odd firmware).

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

**What this does:** Writes the boot menu.

```bash
exit
```

**What this does:** Leaves the chroot.

### Step 9 — Unmount and reboot from the NVMe

```bash
umount -R /mnt/src
```

**What this does:** Unmounts the source USB copy mounts.

```bash
umount -R /mnt
```

**What this does:** Unmounts the NVMe mounts.

```bash
reboot
```

**What this does:** Restarts so you can pick the NVMe in the firmware boot menu.

**You should now see:** The same desktop and user after login. Updates still use `sudo pacman -Syu`.

---

## Related repos

- Desktop polish + hardware: **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**
- Modular apps: **[artix-portix](https://github.com/mrwingkong/artix-portix)**

---

*Artix OpenRC only — no systemd.*
