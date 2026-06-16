# Resize Proxmox VM

Reference: https://pve.proxmox.com/wiki/Resize_disks

## My Process

- In Admin Dashboard, jump into a shell in the host OS and run

```shell
# qm disk resize <vmid> <disk> <size>
qm resize 104 scsi0 5G
```

- ssh into the VM and list the partition tables (note sda1)

```shell
sudo fdisk -l /dev/sda
```

output

```
Disk /dev/sda: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: ABFBF234-6FBD-4B22-B3CA-0C5A6169F802

Device       Start     End Sectors  Size Type
/dev/sda1  2099200 7339998 5240799  2.5G Linux filesystem
/dev/sda14    2048   10239    8192    4M BIOS boot
/dev/sda15   10240  227327  217088  106M EFI System
/dev/sda16  227328 2097152 1869825  913M Linux extended boot

Partition table entries are not in disk order.
```

- next print the partitions using fdisk

```shell
sudo parted /dev/sda

> print
```

output

```
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
15      5243kB  116MB   111MB   fat32              boot, esp
16      116MB   1074MB  957MB   ext4               bls_boot
 1      1075MB  3758MB  2683MB  ext4
```

- then resize the partition that corresponds to the filesystem (should be the end of disk)

```
> resizepart 1 100%
> print
```

output

```
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
15      5243kB  116MB   111MB   fat32              boot, esp
16      116MB   1074MB  957MB   ext4               bls_boot
 1      1075MB  5369MB  4294MB  ext4
```

- Resize the filesystem

```
sudo resize2fs /dev/sda1
```
