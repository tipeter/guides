## Gentoo on RPi 3/B+ 64bit

### Preparing SDCard and SSD

Partitions on SDCard:
```bash
Disk /dev/sda: 29.2 GiB, 31306285056 bytes, 61145088 sectors
Disk model: SD/MMC          
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x589a38aa

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sda1          2048   131071   129024   63M  c W95 FAT32 (LBA)
/dev/sda2        131072  8519680  8388609    4G 82 Linux swap / Solaris
/dev/sda3       8521728 61145087 52623360 25.1G 83 Linux
```

Partitions of the SSD:
```bash
```

Format the partitions:
```bash
# mkfs.vfat -n BOOT /dev/sda1
# mkswap --label="fast-swap" /dev/sda2
# mkfs.ext4 -L backup /dev/sda3 
```

Get the PARTUUID and UUID values of partitions:
```bash
# blkid
/dev/sda1: SEC_TYPE="msdos" LABEL_FATBOOT="BOOT" LABEL="BOOT" UUID="0CCF-3C39" TYPE="vfat" PARTUUID="589a38aa-01"
/dev/sda2: LABEL="fast-swap" UUID="e5ed110f-7084-42ee-be7e-ebc0fa96b606" TYPE="swap" PARTUUID="589a38aa-02"
/dev/sda3: LABEL="backup" UUID="5ea2b921-d5e1-421e-bffb-2fb2da11cc83" TYPE="ext4" PARTUUID="589a38aa-03"
/dev/sdb1: LABEL="rootfs" UUID="7f87925e-ae68-4831-a2af-a690bd5cec82" TYPE="ext4" PARTUUID="e9d76a01-01"
```

List the partition structure of the gentoo-pi.img image!
```bash
# fdisk -lu gentoo-pi.img
Disk gentoo-pi.img: 29.2 GiB, 31306285056 bytes, 61145088 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x7a0c8bb0

Device         Boot  Start      End  Sectors  Size Id Type
gentoo-pi.img1        2048   131071   129024   63M  c W95 FAT32 (LBA)
gentoo-pi.img2      131072 61145087 61014016 29.1G 83 Linux
```

Create two mount points "mnt-src" and "mnt-dest", and mount the partition you want to copy!
```bash
# mkdir mnt-src mnt-dest
# mount -o loop,offset=$((2048 * 512)) gentoo-pi.img ~/mnt-src
# mount -o loop,fooset=$((131072 * 512)) gentoo-pi.img ~/mnt-src
```

Mount the partition you want to use as target!
```bash
# mount /dev/sda1 ~/mnt-dest
# mount /dev/sdb1 ~/mnt-dest
```

Copy the contents of the gentoo-pi.img to the physical partitions!
```bash
# rsync -av ~/mnt-src/ ~/mnt-dest/
```


### Things to do after the first boot:

Setting up timezone:
```bash
# ls /usr/share/zoneinfo

# echo "Europe/Budapest" > /etc/timezone
# emerge --config sys-libs/timezone-data
```

Setting up keyboard layout
```bash
# nano /etc/conf.d/keymaps

# nano /etc/X11/xorg.conf.d/00-keyboard.conf
```

Setting up locale:
```bash
# nano /etc/locale.gen

---
hu_HU ISO-8859-2
hu_HU.UTF-8 UTF-8
---

# locale-gen

# eselect locale list

Available targets for the LANG variable:
  [1] C
  [2] C.utf8
  [3] POSIX
  [4] en_GB
  [5] en_GB.iso88591
  [6] en_GB.utf8
  [7] hu_HU
  [8] hu_HU.iso88592
  [9] hu_HU.utf8
  [ ] (free form)

# eselect locale set 9

# env-update && source /etc/profile
```

Setting up locale for users:
```bash
# nano ~/.bashrc

---
export LANG="hu_HU.UTF-8"
export LC_COLLATE="C"
---
```

```bash
# nano ~/.profile

---
export LANG="hu_HU.UTF-8"
export LC_COLLATE="C"
---
```

Enable hardware clock (RTC):
```bash
rc-update add hwclock boot
rc-update del swclock boot
```

### Installing Rust
[*Source: Manage Rust installations on Gentoo with rustup by Thomas Jespersen*](https://laumann.xyz/gentoo/2018/05/01/gentoo-package-provided.html)

Remove Genoo's own Rust installation:
```bash
# emerge -C virtual/cargo virtual/rust dev-lang/rust
```
Install official Rust and its components (nightly as default, then stable):
```bash
# curl https://sh.rustup.rs -sSf | sh
# source $HOME/.cargo/env
# rustup component add rls-preview rust-analysis rust-src rustfmt-preview
# rustup toolchain add stable-aarch64-unknown-linux-gnu
```

Prevent Gentoo to install Rust:
```bash
# nano /etc/portage/profile/package.provided
---
dev-lang/rust-1.31.1
virtual/rust-1.31.1-r1
virtual/cargo-1.31.1
---

# nano /etc/portage/profile/profile.bashrc
---
export PATH="/home/demouser/.cargo/bin:$PATH"

STABLE=/home/demouser/.rustup/toolchains/stable-aarch64-unknown-linux-gnu
rustup toolchain link build-stable $STABLE &> /dev/null
rustup default build-stable &> /dev/null
---
```

Test installation by emerging "ripgrep" utility

```bash
# nano /etc/portage/package.accept_keywords/ripgrep
---
sys-apps/ripgrep * ~*
---

# emerge --ask sys-apps/ripgrep
```
