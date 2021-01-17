---
title: Enlarge an encrypted LUKS partition on LVM
excerpt: Enlarge an encrypted LUKS partition based on LVM.
toc: true
comments: true
author: Eimert Vink
tags:
  - tech
  - linux
---
I recently upgraded my laptop SSD from 512GB to 1TB.
![Kingston ssd](https://tweakers.net/i/dVbxOtqB6T84aIQ2uxdDRSSIelo=/fit-in/x800/filters:strip_icc():strip_exif()/i/2002979434.jpeg?f=imagegallery)

Getting the data over to the new SSD is a three-step process:
1. I made an image using [clonezilla](clonezilla.org/) and an external hdd.
2. Installed the new SSD and restored the clonezilla image.
3. _Increased the LVM and LUKS partition size to make use of all the space on the SSD._

This article describes step 3 in detail.

> :warning: **Warning**: I always recommend taking backups or snapshots. In case something goes wrong, you’re not
permanently burned and you can revert back to the previous setup.

The encrypted partition in this guide is `/dev/nvme0n1p3`, you should adapt the commands for your own system.

## Enlarge an encrypted partition
1. Boot with your (favourite) distro's live CD.
2. Remove and recreate the existing partition using fdisk.

    ```
    sudo fdisk /dev/nvme0n1
    ```
This was my fdisk session:
    ```
    Opdracht (m voor hulp): p
    Disk /dev/nvme0n1: 931,5 GiB, 1000204886016 bytes, 1953525168 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 3B5990BC-BBAF-41A6-9293-76DC9DAF0017

    Apparaat         Start      Einde   Sectoren   Size Type
    /dev/nvme0n1p1    2048    1050623    1048576   512M EFI System
    /dev/nvme0n1p2 1050624    2050047     999424   488M Linux filesystem
    /dev/nvme0n1p3 2050048 1953525134 1951475087 930,5G Linux filesystem

    Command (m for help): d
    Partition number (1-3): 3

    Command (m for help): n
    Partition number (1-3): 3

    Just press enter to use fdisk defaults for partition size, in order to use the full disk (remaning unallocated space)
    ```

3. Reboot (after changing the partition table with fdisk).

4. Boot the live CD.

5. Mount the LUKS partition with cryptsetup:

    ```
    mint ~ # cryptsetup open /dev/nvme0n1p3 cryptdisk
    Enter passphrase for /dev/nvme0n1p3:
    ```

6. Enlarge the (LVM) Physical Volume with pvresize.

    ```
    mint ~ # pvresize /dev/mapper/cryptdisk
      Physical volume "/dev/mapper/cryptdisk" changed
      1 physical volume(s) resized / 0 physical volume(s) not resized
    mint ~ # pvdisplay -m
      --- Physical volume ---
      PV Name               /dev/mapper/cryptdisk
      VG Name               mint-vg
      PV Size               930,53 GiB / not usable 1,69 MiB
      Allocatable           yes
      PE Size               4,00 MiB
      Total PE              238216
      Free PE               116371
      Allocated PE          121845
      PV UUID               JcuPsv-fDo0-zmkX-DN0d-zlg7-hlWY-aerVcg

      --- Physical Segments ---
      Physical extent 0 to 117791:
        Logical volume /dev/mint-vg/root
        Logical extents 0 to 117791
      Physical extent 117792 to 121844:
        Logical volume /dev/mint-vg/swap_1
        Logical extents 0 to 4052
      Physical extent 121845 to 238215:
        FREE
    ```


7. Enlarge the (root) Logical Volume (LV) and file system with lvresize.

    Option `-l +100%FREE` to take up all the free space.
    Include option `-r` to invoke resize2fs for resizing the underlying file system (ext4 in my case).

    ```
    mint ~ # lvresize -l +100%FREE -r /dev/mint-vg/root
    fsck from util-linux 2.27.1
    /dev/mapper/mint--vg-root: 3129546/30154752 bestanden (0.5% niet-aaneengesloten), 113461654/120619008 blokken
      Size of logical volume mint-vg/root changed from 460,12 GiB (117792 extents) to 914,70 GiB (234163 extents).
      Logical volume root successfully resized.
    resize2fs 1.42.13 (17-May-2015)
    Van grootte veranderen van bestandssysteem op /dev/mapper/mint--vg-root naar 239782912 blokken (van 4K).
    Het bestandssysteem op /dev/mapper/mint--vg-root is nu 239782912 blokken (van 4K) groot.

    mint ~ # lvs
      LV     VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root   mint-vg -wi-a----- 914,70g
      swap_1 mint-vg -wi-a-----  15,83g
    ```

8. Reboot to your encrypted hard drive.

Thanks for reading, let me know in the comments how your experience was.

## References
[Ubuntu ResizeEncryptedPartitions](https://help.ubuntu.com/community/ResizeEncryptedPartitions)<br>
[ArchLinux Resizing LVM-on-LUKS](https://wiki.archlinux.org/index.php/Resizing_LVM-on-LUKS)<br>