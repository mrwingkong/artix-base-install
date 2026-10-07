# artix-base-install

Simple step-by-step guides to install **Artix Linux (OpenRC)** with an **LXQt + KWin** desktop on Wayland. No systemd. Built to run from a USB stick or an internal NVMe and stay movable between machines.

This is the **base install** repo. Related projects:

| Repo | What it is |
|------|------------|
| **This one** — [artix-base-install](https://github.com/mrwingkong/artix-base-install) | Disk, base system, desktop, audio, portable USB/NVMe notes |
| [artix-post-install](https://github.com/mrwingkong/artix-post-install) | Optional desktop polish + ThinkPad / your hardware |
| [artix-portix](https://github.com/mrwingkong/artix-portix) | Optional Porteus-style `.xzm` modules and `pman` |

> Draft: these GitHub links are the planned names. Publish only after you say go.

## Who is this for?

Anyone who wants a full Artix OpenRC install (not a live ISO with persistence) that can move between PCs with minimal fuss.

## Before you start

- Boot a recent **Artix base OpenRC** ISO.
- Back up anything important. The install **wipes a whole disk**.
- Find your disk name with `lsblk` (for example `/dev/sda` or `/dev/nvme0n1`).

> ⚠️ **Change `myname` to your own username** whenever you see it.  
> ⚠️ **Change `myhostname` to your chosen hostname.**  
> ⚠️ **Change `/dev/sda` if your disk is different.**  
> ⚠️ **UK locale and London timezone are examples. Change them for your country.**

## Get the files

```bash
cd ~
git clone https://github.com/mrwingkong/artix-base-install.git
```

**What this does:** puts the base guides in `~/artix-base-install`.

## The guide

Start here: **[`guide/01-disk-and-base.md`](guide/01-disk-and-base.md)**

| Step | File | What it sets up |
|-----:|------|-----------------|
| 1 | [01-disk-and-base.md](guide/01-disk-and-base.md) | Disk, base system, user |
| 2 | [02-desktop-and-audio.md](guide/02-desktop-and-audio.md) | LXQt + KWin, GRUB, sound, power, keyboard |
| 3 | [03-portable-usb-nvme.md](guide/03-portable-usb-nvme.md) | Removable-root safety, UUID checks, USB → NVMe clone |

Not sure what a package does? See [`guide/packages.md`](guide/packages.md). New words? See [`GLOSSARY.md`](GLOSSARY.md).

## After the base

1. Optional polish + hardware: **[artix-post-install](https://github.com/mrwingkong/artix-post-install)**
2. Optional modular apps: **[artix-portix](https://github.com/mrwingkong/artix-portix)**

## Licence

[GPL-3.0](LICENSE)
