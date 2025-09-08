# Linux Filesystem Management – Master Reference

This document explains the complete Linux filesystem architecture, hierarchy, and management concepts.

---

## 1. Linux Filesystems: The Foundation of Storage

A Linux filesystem is how the OS organizes and manages data on storage devices such as HDDs, SSDs, or NVMe drives. Unlike Windows, Linux does not use drive letters; everything lives in a single unified directory tree starting at / (root).

### Partition vs Filesystem

* **Partition:** A subsection or "slice" of a storage device. Acts like a fenced area in a city. Example: /dev/sda1 (first partition of first disk).

* **Filesystem:** The method Linux uses to store, organize, and retrieve files inside a partition. Think of it as the street map and building layout within that fenced city. Example: ext4, XFS, Btrfs.

* **Key Relation:** Partition = container → Filesystem = organization inside container.

* **Advanced setups:** one filesystem can span multiple partitions (LVM, RAID).

### Common Linux Filesystem Types

#### Traditional Disk Filesystems (HDD/SSD)

* **ext3** - Older, journaled, reliable.
* **ext4** - Modern default, supports large files & partitions.
* **XFS** - High-performance, excellent for large files.
* **Btrfs** - Copy-on-write, snapshots, checksums, modern features.
* **JFS** - IBM journaled FS, rarely used today.
* **NTFS** - Windows compatibility.
* **vfat/exfat** - Cross-platform, used for USB drives.

#### Flash-Specific Filesystems (SSD/Embedded)

* **ubifs** - NAND flash, wear-leveling.
* **jffs2** - Older flash FS for embedded devices.
* **yaffs** - Embedded devices, optimized for NAND.

#### Special-Purpose / Virtual Filesystems

* **procfs (/proc)** - Info about running processes, kernel stats.
* **sysfs (/sys)** - Hardware and kernel information.
* **tmpfs** - RAM-based FS for temporary data.
* **squashfs** - Compressed, read-only FS, used in live CDs.
* **debugfs** - Kernel debugging info.
* **FUSE** - User-space FS (e.g., sshfs).

### Mounting & Mount Points

* **Mounting:** Attaching a filesystem (from a partition or device) into the Linux directory tree.

* **Command:** mount /dev/sda1 /mnt

* Here, /dev/sda1 becomes accessible at /mnt.

* **Mount Point:** Directory where the filesystem is attached. Examples: /, /home, /boot.

#### Conceptual Map:

* / → root FS, city center
* /boot → airport (kernel & bootloader)
* /home → residential area (user files)
* /var → industrial/logs/data area

### Boot-Time Usage of Filesystems

* Bootloader loads the kernel and initramfs into RAM.
* Kernel initializes hardware and temporarily uses initramfs to mount the real root filesystem (/).
* Once root FS is mounted, Linux gains access to all directories (/usr, /var, /home) and starts user-space processes and daemons.

* **Key Insight:** Mounting is the bridge connecting storage hardware to the OS. Without it, applications cannot access /usr/bin, /var/log, or /home.

### Why Filesystem Knowledge Matters

* Troubleshooting boot issues (/boot missing kernel, initramfs problems).
* Partitioning, resizing, or formatting disks.
* Understanding /etc/fstab and automount behavior.
* Optimizing performance based on storage type (HDD vs SSD vs NVMe).

#### Mind-Connector: Linux filesystem = city map

* / → city center
* /home → residential
* /var → industrial/logs
* /boot → airport
* /tmp → temporary market
* /usr → offices, shops
* /dev → hardware ports

---

## 2. Linux Filesystem Hierarchy Standard (FHS)

FHS is a standard directory layout for Linux, maintained by the Linux Foundation. Think of it as city zoning: each area has a clear purpose, consistent across distributions.

### Core Directories

* **/** - Root - City center, all roads branch from here
* **/bin** - Essential binaries - Commands needed for all users & during boot (ls, cp, cat)
* **/sbin** - System binaries - Admin commands (ifconfig, fsck), mostly root
* **/boot** - Boot files - Kernel images, GRUB configs, initramfs
* **/dev** - Device files - Hardware appears as files: /dev/sda, /dev/tty
* **/etc** - Config files - System-wide configs, startup scripts (fstab, passwd)
* **/home** - User directories - /home/username, personal files
* **/lib** - Libraries - Needed by /bin & /sbin for running programs
* **/opt** - Optional software - Big packages, commercial apps
* **/tmp** - Temporary files - Cleared on reboot
* **/usr** - User programs/data - Secondary hierarchy: apps, libraries, manuals
* **/var** - Variable data - Logs, caches, databases
* **/media** - Removable media - Auto-mounted USBs/CDs (/media/username/USB1)
* **/mnt** - Temporary mount point - For manual mounting of partitions
* **/proc** - Kernel/process info - Virtual FS, live CPU/memory stats
* **/sys** - System info - Hardware info from kernel, virtual FS
* **/run** - Runtime files - Active system info: PIDs, sockets, locks

### Root vs /usr

* **/** = essential files for boot and repair
* **/usr** = secondary programs, not critical for boot

### /var – Variable Data

* Logs (/var/log)
* Databases (/var/lib)
* Cache (/var/cache)
* Spools (/var/spool)

### /etc – System Configuration

* /etc/fstab → defines partitions & mount points
* /etc/hosts → local hostname mappings
* /etc/passwd → user info

### Virtual Filesystems

* /dev → devices as files
* /proc → live kernel/process info
* /sys → hardware/kernel module info

### Temporary/Runtime FS

* /tmp → cleared at reboot
* /run → runtime files (PIDs, sockets)

### /boot – Kernel Access Point

* vmlinuz-... → kernel image
* initrd.img-... → initramfs image
* grub/grub.cfg → bootloader menu

---

## 3. Partitions and Mounting in Linux

### What is a Partition?

* Logical subsection of a disk (HDD, SSD, NVMe)
* **Purpose:** separate OS/data, multiple OS, optimize performance, security

#### Examples:

* /dev/sda1 → first SATA disk partition
* /dev/nvme0n1p2 → second NVMe partition

### What is a Filesystem?

* Stores & organizes files inside a partition
* **Analogy:** streets, buildings, addresses inside a fenced city area
* **Linux filesystems:** ext4, XFS, Btrfs, vfat, exfat, UBIFS, tmpfs
* **Reason for FS per partition:** flexibility, recovery, performance

### Mounting – Making Partitions Usable

* **Mounting** = telling Linux "attach this partition here"
* **Mount point** = directory where FS appears

#### Command:

    sudo mount /dev/sda2 /mnt/data

* /dev/sda2 → partition
* /mnt/data → mount point
* Files become accessible via /mnt/data

#### Automatic Mounting: /etc/fstab

* Defines partitions to mount at boot

#### Line example:

    /dev/sda1 / ext4 defaults 1 1
    /dev/sda2 /home ext4 defaults 1 2

### Partition Types

* **Primary:** bootable, max 4 on MBR
* **Extended:** container for logical partitions
* **Logical:** inside extended, flexible

#### GPT vs MBR:

* **MBR:** 4 primary, 2TB limit
* **GPT:** modern, huge disks, 128+ partitions

### SSD / NVMe / HDD Differences

* **SSD/NVMe** - Fast, low latency - Random access → fast boot/mount, TRIM support
* **HDD** - Slower - Sequential reads preferred
* **NVMe PCIe** - Ultra-fast - Kernel + initramfs + root FS loads extremely fast

### Virtual vs Physical Partitions

* **Physical:** HDD, SSD, NVMe
* **Virtual:** tmpfs, /proc, /sys (in-memory, volatile)

### Mounting Flow During Boot

* BIOS/UEFI → POST → check devices
* Bootloader → load kernel + initramfs into RAM
* Kernel → initialize hardware, RAM, storage
* initramfs → locate root FS partition
* Kernel → mount root FS (/)
* init → start user-space processes, services, login

* **Key Insight:** Mounting bridges storage and user-space applications. Without it, Linux cannot access /usr/bin, /home, /var/log, etc.

#### Mind Map:

    Power ON → BIOS/UEFI → Bootloader → Kernel → initramfs → Root FS Mounted → init → Services/User Login
