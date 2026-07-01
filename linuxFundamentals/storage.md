# Chapter 15 — Storage Media (The Linux Command Line)

A quick-revision summary of managing disks, partitions, filesystems, and media.

>  **Danger zone:** many commands here (`fdisk`, `mkfs`, `dd`) write directly to devices and can **destroy data instantly** with no undo. Always double-check the device name before pressing Enter.

---

## 1. The Big Idea

Linux treats storage through **one unified filesystem tree** rather than drive letters. A device (hard disk, USB stick, CD) must be **mounted** — attached to a directory (a **mount point**) — before you can use it. Detaching it is **unmounting**.

**Device naming** lives under `/dev`:

| Name | Device |
|------|--------|
| `/dev/sda` | First SATA/SCSI/USB disk (whole device) |
| `/dev/sdb` | Second such disk |
| `/dev/sda1`, `/dev/sda2` | **Partitions** 1, 2 on the first disk |
| `/dev/sr0`, `/dev/cdrom` | Optical drive |
| `/dev/hda` | First PATA/IDE disk (older systems) |

> Whole device = `sda`; a partition on it adds a number = `sda1`.

---

## 2. Mounting and Unmounting

### See what's currently mounted

```bash
mount        # with no arguments, lists all mounted filesystems
```

Each line reads roughly: `device on mount_point type fstype (options)`.

### `/etc/fstab` — the mount table

This file lists devices and where they mount **automatically at boot**. Fields per line:

| Field | Meaning |
|-------|---------|
| Device | e.g. `/dev/sda1`, or a `UUID=`/`LABEL=` |
| Mount point | e.g. `/`, `/home` |
| Filesystem type | e.g. `ext4`, `vfat` |
| Options | e.g. `defaults` |
| Dump | Backup flag (usually `0`) |
| Pass (fsck order) | `0` skip, `1` root, `2` others |

### Mounting manually

```bash
mount -t ext4 /dev/sdb1 /mnt/flash     # mount device at a mount point
mount /dev/sdb1 /mnt/flash             # type often auto-detected
```

### Unmounting

```bash
umount /dev/sdb1        # by device
umount /mnt/flash       # or by mount point
```

> A device is **busy** and won't unmount if any process is using it — including a shell that has `cd`'d into it. Move out first (`cd`), then `umount`.

### Finding a device's name

After plugging in a device, discover what the kernel called it:

```bash
dmesg | tail                # recent kernel messages
less /var/log/messages      # system log (on some distros)
```

---

## 3. Creating New Filesystems

Two steps: **partition** the device, then **put a filesystem** on the partition.

### `fdisk` — edit the partition table

```bash
fdisk /dev/sdb        # operate on the whole device (as root)
```

Interactive sub-commands:

| Key | Action |
|-----|--------|
| `p` | **Print** the partition table |
| `n` | **New** partition |
| `d` | **Delete** a partition |
| `t` | Change a partition's **type** (ID code) |
| `l` | **List** known type codes |
| `w` | **Write** changes to disk and exit (commits everything) |
| `q` | **Quit** with no changes (safe escape) |

Common type IDs: `83` Linux, `82` Linux swap, `b` FAT32.

> Nothing is written until you press `w`. If you make a mistake, `q` walks away cleanly.

### `mkfs` — make a filesystem

```bash
mkfs -t ext4 /dev/sdb1     # format partition as ext4
mkfs -t vfat /dev/sdb1     # format as FAT32 (portable/USB)
```

> This **erases** whatever was on the partition.

---

## 4. Testing and Repairing Filesystems

```bash
fsck /dev/sdb1        # check (and optionally repair) a filesystem
```

- The device should be **unmounted** first.
- `fsck` = "filesystem check." It verifies integrity and can fix errors.
- Recovered but unlinked fragments land in a **`lost+found`** directory on that filesystem.

---

## 5. Moving Data Directly To/From Devices — `dd`

`dd` copies **raw blocks** between files/devices, bypassing the filesystem.

```bash
dd if=INPUT of=OUTPUT [bs=BLOCKSIZE]
```

| Operand | Meaning |
|---------|---------|
| `if=`   | **Input** file/device |
| `of=`   | **Output** file/device |
| `bs=`   | Block size (e.g. `bs=1M`) |

```bash
dd if=/dev/sdc of=/dev/sdd          # clone one device onto another
dd if=/dev/sda of=disk.img          # image a whole disk to a file
```

> ⚠️ `dd` has been nicknamed "**disk destroyer**." Swapping `if=` and `of=` overwrites the wrong device. There is no confirmation and no undo — verify names first.

---

## 6. Creating and Writing Optical (CD/DVD) Images

### Make an ISO image

From an existing disc:

```bash
dd if=/dev/cdrom of=image.iso        # copy a CD to an ISO file
```

From a directory of files:

```bash
genisoimage -o image.iso -R -J directory
```

| Option | Meaning |
|--------|---------|
| `-o`   | Output image filename |
| `-R`   | Rock Ridge extensions (preserve Unix names/permissions) |
| `-J`   | Joliet extensions (Windows-compatible long names) |

### Inspect an image without burning — mount it via loopback

```bash
mount -t iso9660 -o loop image.iso /mnt/iso
```

### Blank a rewritable disc (CD-RW)

```bash
wodim dev=/dev/cdrom blank=fast
```

### Write the image to disc

```bash
wodim dev=/dev/cdrom image.iso
```

> On some systems the tools are named `mkisofs` (≈ `genisoimage`) and `cdrecord` (≈ `wodim`).

---

## 7. Verifying Image Integrity — `md5sum`

Confirm a downloaded or copied image isn't corrupted by comparing checksums:

```bash
md5sum image.iso        # prints a hash to compare against the published one
```

Matching hashes ⇒ the file is intact.

---

## Quick Reference Cheat Sheet

```text
# Mounting
mount                       # list mounted filesystems
mount /dev/sdb1 /mnt/flash  # mount a device
umount /dev/sdb1            # unmount (must not be in use)
/etc/fstab                  # auto-mount table
dmesg | tail                # find a newly plugged device name

# Partitioning + formatting
fdisk /dev/sdb             # edit partition table (p n d t w q)
mkfs -t ext4 /dev/sdb1     # create ext4 filesystem
mkfs -t vfat /dev/sdb1     # create FAT32 filesystem

# Check / repair
fsck /dev/sdb1             # check a filesystem (unmount first)

# Raw copy  (DANGER: verify if=/of=)
dd if=/dev/sdc of=/dev/sdd     # clone device to device
dd if=/dev/cdrom of=image.iso  # rip a CD to an ISO

# Optical images
genisoimage -o img.iso -R -J dir      # build ISO from a folder
mount -o loop img.iso /mnt/iso        # inspect an ISO
wodim dev=/dev/cdrom blank=fast       # blank a CD-RW
wodim dev=/dev/cdrom img.iso          # burn an image

# Integrity
md5sum image.iso           # checksum to verify a copy/download
```

---

### Mental model to remember

> A device isn't usable until it's **mounted** onto the tree, and can't be removed until **unmounted**.
> To prepare fresh media: **`fdisk`** (partition) → **`mkfs`** (filesystem) → mount and use; **`fsck`** repairs later.
> **`dd`** ignores filesystems and copies raw blocks — supremely useful and supremely dangerous, so always check `if=`/`of=` before you hit Enter.
