---
title: LVM上のbtrfsのRAID1のサイズを拡張する
tags: Linux btrfs lvm
author: takaki@github
slide: false
---
LinuxのLVMでbtrfsのRAID1を作成していたときにLVMのボリュームを拡張してbtrfs自体を拡張する実験をした。

まずはbtrfsでRAID1を作成する。

```
/mnt# lvcreate -L 1G -n v0 vg4t  Using default stripesize 64.00 KiB.
  Logical volume "v0" created.
/mnt# lvcreate -L 1G -n v1 vg4t
  Logical volume "v1" created.
/mnt# mkfs.btrfs /dev/vg4t/v0
btrfs-progs v4.7
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               51c63065-a84e-4aa5-85b6-9597e5f9a943
Node size:          16384
Sector size:        4096
Filesystem size:    1.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP              51.19MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     1.00GiB  /dev/vg4t/v0

/mnt# mount /dev/vg4t/v0 testfs
/mnt# btrfs fi df testfs
Data, single: total=8.00MiB, used=0.00B
System, DUP: total=8.00MiB, used=16.00KiB
Metadata, DUP: total=51.19MiB, used=112.00KiB
GlobalReserve, single: total=16.00MiB, used=0.00B
/mnt# btrfs device add /dev/vg4t/v1 testfs
/mnt# btrfs fi df testfs
Data, single: total=8.00MiB, used=0.00B
System, DUP: total=8.00MiB, used=16.00KiB
Metadata, DUP: total=51.19MiB, used=112.00KiB
GlobalReserve, single: total=16.00MiB, used=0.00B
/mnt# btrfs balance start -dconvert=raid1 -mconvert=raid1 testfs
Done, had to relocate 3 out of 3 chunks
/mnt# btrfs fi df testfs
Data, RAID1: total=208.00MiB, used=128.00KiB
Data, single: total=208.00MiB, used=0.00B
System, RAID1: total=32.00MiB, used=16.00KiB
Metadata, RAID1: total=208.00MiB, used=112.00KiB
GlobalReserve, single: total=16.00MiB, used=0.00B
```
mkfs.btrfsで一発で作成をする方法もあるが今回は後からデバイスを追加する手順でやってみた。

次にLVMのボリュームを拡張する。

```
/mnt# lvextend -L+1G /dev/vg4t/v0
  Size of logical volume vg4t/v0 changed from 1.00 GiB (256 extents) to 2.00 GiB (512 extents).
  Logical volume vg4t/v0 successfully resized.
/mnt# lvextend -L+1G /dev/vg4t/v1
  Size of logical volume vg4t/v1 changed from 1.00 GiB (256 extents) to 2.00 GiB (512 extents).
  Logical volume vg4t/v1 successfully resized.
/mnt# df /mnt/testfs
ファイルシス        1K-ブロック  使用 使用可 使用% マウント位置
/dev/mapper/vg4t-v0     1048576 16640 695168    3% /mnt/testfs
```
この段階では当然拡張はされない。

```
/mnt# btrfs filesystem  resize max testfs
Resize 'testfs' of 'max'
/mnt# df /mnt/testfs
ファイルシス        1K-ブロック  使用 使用可 使用% マウント位置
/dev/mapper/vg4t-v0     1572864 16640 695168    3% /mnt/testfs
```
残念ながらこれでは拡張されていない。

```
/mnt# btrfs filesystem show testfs
Label: none  uuid: 51c63065-a84e-4aa5-85b6-9597e5f9a943
	Total devices 2 FS bytes used 256.00KiB
	devid    1 size 2.00GiB used 448.00MiB path /dev/mapper/vg4t-v0
	devid    2 size 1.00GiB used 656.00MiB path /dev/mapper/vg4t-v1
```
devid 2が拡張されてない。

次のように明示的にdevidを指定して拡張する。

```
/mnt# btrfs filesystem  resize 2:max testfs
Resize 'testfs' of '2:max'
/mnt# df /mnt/testfs
ファイルシス        1K-ブロック  使用  使用可 使用% マウント位置
/dev/mapper/vg4t-v0     2097152 16640 1743744    1% /mnt/testfs
/mnt# btrfs filesystem show testfs
Label: none  uuid: 51c63065-a84e-4aa5-85b6-9597e5f9a943
	Total devices 2 FS bytes used 256.00KiB
	devid    1 size 2.00GiB used 448.00MiB path /dev/mapper/vg4t-v0
	devid    2 size 2.00GiB used 656.00MiB path /dev/mapper/vg4t-v1
```
容量が増えた。

