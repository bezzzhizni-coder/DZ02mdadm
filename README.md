# DZ02mdadm
• Добавить в виртуальную машину несколько дисков  
• Собрать RAID-0/1/5/10 на выбор  
• Сломать и починить RAID  
• Создать GPT таблицу, пять разделов и смонтировать их в системе.  
>"На проверку отправьте:
скрипт для создания рейда,"  

не умею писать скрипты  
можно пример скрипта?

```
gor@testsrv:~$ sudo lsblk
[sudo] password for gor:
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sr0                        11:0    1  3,1G  0 rom

gor@testsrv:~$ sudo lshw -short | grep disk
/0/100/10/0.0.0        /dev/sda    disk       17GB Virtual disk
/0/100/11/1/0.0.0      /dev/cdrom  disk       VMware SATA CD00
/0/100/11/1/0.0.0/0    /dev/cdrom  disk
/0/100/11/2/0.0.0      /dev/sdb    disk       1073MB Virtual disk
/0/100/11/2/0.1.0      /dev/sdc    disk       1073MB Virtual disk

root@testsrv:/home/gor# mdadm --zero-superblock --force /dev/sdb /dev/sdc
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
root@testsrv:/home/gor# mdadm --create /dev/md01 -l 10 -n 2 /dev/sdb /dev/sdc
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md01 started.
mdadm: timeout waiting for /dev/md01

root@testsrv:/home/gor# cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid10 sdc[1] sdb[0]
      1046528 blocks super 1.2 2 near-copies [2/2] [UU]

root@testsrv:/home/gor# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Fri Nov  7 09:19:48 2025
        Raid Level : raid10
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Fri Nov  7 09:19:54 2025
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : testsrv:01  (local to host testsrv)
              UUID : 42fdc2cb:79ded733:2146ab82:cef3466c
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc

root@testsrv:/home/gor# mdadm /dev/md1 --fail /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md1
root@testsrv:/home/gor#  cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid10 sdc[1](F) sdb[0]
      1046528 blocks super 1.2 2 near-copies [2/1] [U_]

root@testsrv:/home/gor# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Fri Nov  7 09:19:48 2025
        Raid Level : raid10
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Fri Nov  7 09:35:56 2025
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : testsrv:01  (local to host testsrv)
              UUID : 42fdc2cb:79ded733:2146ab82:cef3466c
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       -       0        0        1      removed

       1       8       32        -      faulty   /dev/sdc

root@testsrv:/home/gor# echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan
root@testsrv:/home/gor# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part   /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm    /
sdb                         8:16   0    1G  0 disk
└─md1                       9:1    0 1022M  0 raid10
sdc                         8:32   0    1G  0 disk
└─md1                       9:1    0 1022M  0 raid10
sdd                         8:48   0    1G  0 disk
sr0                        11:0    1  3,1G  0 rom

root@testsrv:/home/gor# mdadm /dev/md1 --remove /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md1

root@testsrv:/home/gor# mdadm /dev/md1 --add /dev/sdd
mdadm: added /dev/sdd
root@testsrv:/home/gor# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Fri Nov  7 09:19:48 2025
        Raid Level : raid10
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Fri Nov  7 09:47:37 2025
             State : clean, degraded, recovering
    Active Devices : 1
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 16% complete

              Name : testsrv:01  (local to host testsrv)
              UUID : 42fdc2cb:79ded733:2146ab82:cef3466c
            Events : 26

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       2       8       48        1      spare rebuilding   /dev/sdd

root@testsrv:/home/gor# mdadm --zero-superblock --force /dev/sdc
root@testsrv:/home/gor# mdadm /dev/md1 --manage --add-spare /dev/sdc
mdadm: added /dev/sdc
root@testsrv:/home/gor# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Fri Nov  7 09:19:48 2025
        Raid Level : raid10
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Fri Nov  7 09:51:16 2025
             State : clean
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : testsrv:01  (local to host testsrv)
              UUID : 42fdc2cb:79ded733:2146ab82:cef3466c
            Events : 42

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       2       8       48        1      active sync set-B   /dev/sdd

       3       8       32        -      spare   /dev/sdc

root@testsrv:/home/gor# mkfs.ext4 /dev/md1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 261632 4k blocks and 65408 inodes
Filesystem UUID: 161aa195-083c-470c-a702-a5aa4b80d7dd
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@testsrv:/home/gor# mount /dev/md1 /mnt/md/

root@testsrv:/home/gor# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1,8G  0 part   /boot
└─sda3                      8:3    0 14,2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm    /
sdb                         8:16   0    1G  0 disk
└─md1                       9:1    0 1022M  0 raid10
sdc                         8:32   0    1G  0 disk
└─md1                       9:1    0 1022M  0 raid10
sdd                         8:48   0    1G  0 disk
└─md1                       9:1    0 1022M  0 raid10
sr0                        11:0    1  3,1G  0 rom


root@testsrv:/home/gor# ls -l /mnt/md/
total 16
drwx------ 2 root root 16384 ноя  7 09:53 lost+found
```
