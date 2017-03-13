---
author: springloops
comments: true
date: 2016-07-21
layout: post
title: 'Add Volume Disk'
categories:
- Linux/Operate
tags:
- Linux
- Centos
- fdisk
- blkid
- fstab
- mount
---

새로운 볼륨디스크를 

1. 파티션 생성
2. 파일시스템 설정
3. 마운트
4. 영구 마운트

하는 과정을 기록함.


### root 변경

```bash
sudo -i
```


## 파티션 생성


### 연결된 하드디스크 조회

fdisk -l

```bash

[root@min-hadoop-rm ~]# fdisk -l

Disk /dev/vda: 53.7 GB, 53687091200 bytes
43 heads, 32 sectors/track, 76204 cylinders
Units = cylinders of 1376 * 512 = 704512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0002a3c4

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           2       76204    52427328   83  Linux

Disk /dev/vdb: 107.4 GB, 107374182400 bytes
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/vdc: 107.4 GB, 107374182400 bytes
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

```

### 파티션 생성

```bash
fdisk /dev/vdb

[root@min-hadoop-rm ~]# fdisk /dev/vdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x9a8822d2.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

   Command (m for help): 
   
   n 입력

   Command action
   e   extended
   p   primary partition (1-4)

   p 입력

   Partition number (1-4): 1

First cylinder (1-208050, default 1): 
Last cylinder, +cylinders or +size{K,M,G} (1-208050, default 208050): 
```

### 파티션 생성 조회

```bash

fdisk -l
[root@min-hadoop-rm ~]# fdisk -l

Disk /dev/vda: 53.7 GB, 53687091200 bytes
43 heads, 32 sectors/track, 76204 cylinders
Units = cylinders of 1376 * 512 = 704512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0002a3c4

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           2       76204    52427328   83  Linux

Disk /dev/vdb: 107.4 GB, 107374182400 bytes
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x9a8822d2

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1               1      208050   104857168+  83  Linux

Disk /dev/vdc: 107.4 GB, 107374182400 bytes
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

```

## filesystem 생성

```bash
mkfs.ext4 /dev/vdb1


```

## 마운트

```bash
mkdir /data1
mount /dev/vdb1 /data1

```

### 영구 마운트 ( 영구는 없다. )

UUID 확인

```bash

blkid

/dev/vda1: UUID="7e7a79a2-849d-4dc7-9551-2eeefd8a5965" TYPE="ext4"
/dev/vdb1: UUID="ff4960f0-a3ce-4b52-aa62-5c997604fbc3" TYPE="ext4"
/dev/vdc1: UUID="aca31417-bc73-4e67-bf4f-4951bee47a31" TYPE="ext4"

UUID 는 시스템 별로 달라짐.

```

fstab 에 등록

```bash
vi /etc/fstab

UUID=ff4960f0-a3ce-4b52-aa62-5c997604fbc3	/data1		ext4	default 1 2
UUID=aca31417-bc73-4e67-bf4f-4951bee47a31	/data2		ext4	default 1 2

```

### 확인

```bash

shutdown -r now

재시작후 mount 여부 확인

```