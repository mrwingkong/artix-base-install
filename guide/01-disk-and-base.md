# 01 — Disk and base system

Install Artix OpenRC onto your disk.  
After this file you will reboot into a text login, then continue with [02-desktop-and-audio.md](02-desktop-and-audio.md).

**You are:** root on the live Artix OpenRC USB (unless a step says otherwise).

> ⚠️ **Change `/dev/sda` if `lsblk` shows a different disk** (for example `/dev/nvme0n1`).  
> Wrong disk = wipe the wrong drive.

---

## Step 1 — See your disks

```bash
lsblk
```

**What this does:** Lists disks and partitions so you know the correct name.

**You should now see:** Something like `sda` or `nvme0n1` with sizes.

---

## Step 2 — Wipe the chosen disk

> ⚠️ This erases the whole disk. Double-check the name.

```bash
wipefs -a /dev/sda
```

**What this does:** Clears old partition signatures on that disk.

---

## Step 3 — Make new partitions

```bash
cfdisk /dev/sda
```

**What this does:** Opens a partition editor.

Choose partition type **gpt**, then create:

| Partition | Size | Type |
|-----------|------|------|
| sda1 | 512M | EFI System |
| sda2 | 2M (min) | BIOS boot |
| sda3 | 128G | Linux filesystem (root) |
| sda4 | rest | Linux filesystem (home) |

Write changes and quit.

**You should now see:** Four new partitions in `lsblk`.

---

## Step 4 — Format partitions

```bash
mkfs.fat -F32 /dev/sda1
```

**What this does:** Formats the EFI partition as FAT32.

```bash
mkfs.btrfs -f /dev/sda3
```

**What this does:** Formats the root partition as btrfs.

```bash
mkfs.btrfs -f /dev/sda4
```

**What this does:** Formats the home partition as btrfs.

---

## Step 5 — Mount partitions

```bash
mount /dev/sda3 /mnt
```

**What this does:** Mounts root at `/mnt`.

```bash
mkdir /mnt/home
```

**What this does:** Creates the home mount point.

```bash
mount /dev/sda4 /mnt/home
```

**What this does:** Mounts home.

```bash
mkdir -p /mnt/boot
```

**What this does:** Creates the boot mount point.

```bash
mount /dev/sda1 /mnt/boot
```

**What this does:** Mounts the EFI partition at `/boot`.

**You should now see:** `lsblk` shows `/mnt`, `/mnt/home`, and `/mnt/boot` mounted.

---

## Step 6 — Connect to the network (live USB)

The live ISO uses **ConnMan**.

```bash
connmanctl
```

**What this does:** Opens the network tool.

Inside `connmanctl` type these one by one:

```text
enable wifi
scan wifi
services
agent on
connect wifi_xxxxxxxxxx
quit
```

> ⚠️ Replace `wifi_xxxxxxxxxx` with the real service name from `services`.

**Optional — set the clock:**

```bash
rc-service ntpd start
```

**What this does:** Starts time sync on the live system (if available).

---

## Step 7 — Rank Artix mirrors (faster downloads)

```bash
pacman -Syy
```

**What this does:** Refreshes package lists.

```bash
pacman -S pacman-contrib
```

**What this does:** Installs `rankmirrors`.

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist-artix
```

**What this does:** Saves a backup of the mirror list.

```bash
rankmirrors /etc/pacman.d/mirrorlist-artix > /etc/pacman.d/mirrorlist
```

**What this does:** Puts the fastest mirrors first.

---

## Step 8 — Install the base system

> ℹ️ **Intel ThinkPad extras** (`sof-firmware`, `intel-ucode`, `intel-media-driver`, …) are **not** here.  
> Add them later in **[artix-post-install](https://github.com/mrwingkong/artix-post-install)** (ThinkPad / Intel notes) if that is your machine.

```bash
basestrap /mnt base base-devel openrc elogind-openrc linux linux-headers linux-firmware nano
```

**What this does:** Installs the core Artix OpenRC system onto `/mnt`.

```bash
bash -c 'fstabgen -U /mnt > /mnt/etc/fstab'
```

**What this does:** Writes mount entries so the new system finds its disks after reboot.

**You should now see:** A `/mnt/etc/fstab` file with UUID lines for your partitions.

---

## Step 9 — Enter the new system (chroot)

```bash
artix-chroot /mnt
```

**What this does:** Switches you into the installed system so the next commands edit *it*.

---

## Step 10 — Timezone

> ⚠️ **UK example.** Change `Europe/London` to your timezone (look under `/usr/share/zoneinfo/`).

```bash
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

**What this does:** Sets the system timezone.

```bash
hwclock --systohc
```

**What this does:** Saves the time to the hardware clock.

---

## Step 11 — Language (locale)

```bash
nano /etc/locale.gen
```

**What this does:** Opens the locale list so you can uncomment your language.

Uncomment (example for UK English):

```text
en_GB.UTF-8 UTF-8
```

> ⚠️ Change to your locale (for example `en_US.UTF-8 UTF-8`) if you are not in the UK.

```bash
locale-gen
```

**What this does:** Builds the locale you uncommented.

```bash
echo "LANG=en_GB.UTF-8" > /etc/locale.conf
```

**What this does:** Sets the default language.

> ⚠️ Match `LANG=` to the locale you enabled.

---

## Step 12 — Name your computer

> ⚠️ **Change `myhostname` to your chosen hostname** (simple letters/numbers).

```bash
echo "myhostname" > /etc/hostname
```

**What this does:** Saves the computer’s name.

> ⚠️ **Change `myhostname` to your chosen hostname**

```bash
echo "hostname=\"myhostname\"" > /etc/conf.d/hostname
```

**What this does:** Tells OpenRC the same name.

---

## Step 13 — Hosts file

```bash
nano /etc/hosts
```

**What this does:** Opens the hosts file for editing.

Make sure it has lines like:

> ⚠️ **Change `myhostname` to your chosen hostname** (same name as above).

```text
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
```

**You should now see:** Those three kinds of lines in `/etc/hosts`.

---

## Step 14 — Root password

```bash
passwd
```

**What this does:** Sets the administrator (`root`) password. Type it twice.

---

## Step 15 — Create your user

> ⚠️ **Change `myname` to your own username**

```bash
useradd -m -G wheel myname
```

**What this does:** Creates your user and adds them to the `wheel` group (for sudo later).

> ⚠️ **Change `myname` to your own username**

```bash
passwd myname
```

**What this does:** Sets your user’s password.

---

## Step 16 — Allow sudo for wheel

```bash
EDITOR=nano visudo
```

**What this does:** Opens the sudo rules file safely.

Uncomment this line (remove the `#` at the start):

```text
%wheel ALL=(ALL:ALL) ALL
```

**You should now see:** That line active (no `#` in front).

---

## Step 17 — Finish this file

Desktop packages, GRUB, and services continue in the next file **while you are still in the chroot** — open:

→ **[02-desktop-and-audio.md](02-desktop-and-audio.md)** (steps marked “still in chroot”)

Or if you already left the chroot by mistake, run `artix-chroot /mnt` again first.

---

*Artix OpenRC only — no systemd.*
