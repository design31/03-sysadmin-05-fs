# 03-sysadmin-05-fs
 ## Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

Не знал об этом виде файлов. Было полезно )

---

2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?  

Жесткая ссылка это по сути зеркальная копия объекта, наследующая его права, владельца и группу. Имеет тот же inode что и оригинальный файл.  
Поэтому разных владельцев и разные права оригинал и хардлинк иметь не могут.  

---

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.  
    
    Готово. Проверим диски:  
```
vagrant@vagrant:~$ sudo fdisk -l
Disk /dev/sda: 64 GiB, 68719476736 bytes, 134217728 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3f94c461

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1050623   1048576  512M  b W95 FAT32
/dev/sda2       1052670 134215679 133163010 63.5G  5 Extended
/dev/sda5       1052672 134215679 133163008 63.5G 8e Linux LVM


Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vgvagrant-root: 62.55 GiB, 67150807040 bytes, 131153920 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vgvagrant-swap_1: 980 MiB, 1027604480 bytes, 2007040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

---


4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.  
 ```
 fdisk /dev/sdb

 Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-5242879, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (4196352-5242879, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
Посмотрим что получилось:  
 ```
 fdisk -l
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd12211b0

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux

```

---

5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.  
 
 Прямой команды копирования разделов я не нашёл, использовал опцию -b (backup)  
 с последующим восстановлением в другой диск:
 ```
 sfdisk -b /dev/sdb

Welcome to sfdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Checking that no-one is using this disk right now ... OK

Backup files:
         MBR (offset     0, size   512): sdb_bak-sdb-0x00000000.bak

Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd12211b0

Old situation:

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux

Type 'help' to get more information.

>>> write

New situation:
Disklabel type: dos
Disk identifier: 0xd12211b0

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
``` 
Проверим созданный файл бэкапа:
```
ll
-rw------- 1 root    root     512 Nov 30 15:49 sdb_bak-sdb-0x00000000.bak
```
Восстановим его в диск sdc:
```
dd if=sdb_bak-sdb-0x00000000.bak of=/dev/sdc
1+0 records in
1+0 records out
512 bytes copied, 0.000435306 s, 1.2 MB/s
```
Проверим партиции:
```
fdisk -l /dev/sdc
Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd12211b0

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux
```

---

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
 
 ```
 mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b1,c1}
 mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? yes
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
 ```
 ---

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.  
 
 ```
 mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b2,c2}
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
 ```
Посмотрим на созданные массивы:  

```
lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
```
Добавим инфу о рэйдах в файл mdadm.conf:
```
mdadm --detail --scan --verbose >> /etc/mdadm.conf
```

---

8. Создайте 2 независимых PV на получившихся md-устройствах.  
 
 ```
 pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
```
---

9. Создайте общую volume-group на этих двух PV.
 
 ```
 root@vagrant:/home/vagrant# vgcreate vol_group1 /dev/md0 /dev/md1
  Volume group "vol_group1" successfully created
root@vagrant:/home/vagrant# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               vgvagrant
  PV Size               <63.50 GiB / not usable 0
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              16255
  Free PE               0
  Allocated PE          16255
  PV UUID               Mx3LcA-uMnN-h9yB-gC2w-qm7w-skx0-OsTz9z

  --- Physical volume ---
  PV Name               /dev/md0
  VG Name               vol_group1
  PV Size               <2.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               cbmMat-DAGL-PDyO-Dimz-16fh-HgdV-jqz33q

  --- Physical volume ---
  PV Name               /dev/md1
  VG Name               vol_group1
  PV Size               1018.00 MiB / not usable 2.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              254
  Free PE               254
  Allocated PE          0
  PV UUID               XIPOF8-vYdS-oXJS-aphO-cvGQ-1ASi-8qjaFm
```
И ещё vgdisplay информативно будет:
```
root@vagrant:/home/vagrant# vgdisplay
  --- Volume group ---
  VG Name               vgvagrant
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <63.50 GiB
  PE Size               4.00 MiB
  Total PE              16255
  Alloc PE / Size       16255 / <63.50 GiB
  Free  PE / Size       0 / 0
  VG UUID               PaBfZ0-3I0c-iIdl-uXKt-JL4K-f4tT-kzfcyE

  --- Volume group ---
  VG Name               vol_group1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0
  Free  PE / Size       765 / <2.99 GiB
  VG UUID               14pUvP-frDu-0FWG-lY6p-fxDv-1GEL-ItG2dC

```

---

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.  
 
 Создаю и проверяю:
 ```
 root@vagrant:/home/vagrant# lvcreate -L 100M -n logical_vol1 vol_group1 /dev/md1
  Logical volume "logical_vol1" created.
root@vagrant:/home/vagrant# lvdisplay
    --- Logical volume ---
  LV Path                /dev/vol_group1/logical_vol1
  LV Name                logical_vol1
  VG Name                vol_group1
  LV UUID                VtGdPV-cW0F-83bl-0NQi-aSwu-DLkz-bicbbP
  LV Write Access        read/write
  LV Creation host, time vagrant, 2021-12-01 15:56:33 +0000
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           253:2
```

---


11. Создайте `mkfs.ext4` ФС на получившемся LV.

```
root@vagrant:/home/vagrant# mkfs.ext4 /dev/vol_group1/logical_vol1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

root@vagrant:/home/vagrant# lsblk -o NAME,PATH,SIZE,FSTYPE
NAME                          PATH                                 SIZE FSTYPE
sda                           /dev/sda                              64G
├─sda1                        /dev/sda1                            512M vfat
├─sda2                        /dev/sda2                              1K
└─sda5                        /dev/sda5                           63.5G LVM2_member
  ├─vgvagrant-root            /dev/mapper/vgvagrant-root          62.6G ext4
  └─vgvagrant-swap_1          /dev/mapper/vgvagrant-swap_1         980M swap
sdb                           /dev/sdb                             2.5G
├─sdb1                        /dev/sdb1                              2G linux_raid_member
│ └─md0                       /dev/md0                               2G LVM2_member
└─sdb2                        /dev/sdb2                            511M linux_raid_member
  └─md1                       /dev/md1                            1018M LVM2_member
    └─vol_group1-logical_vol1 /dev/mapper/vol_group1-logical_vol1  100M ext4
sdc                           /dev/sdc                             2.5G
├─sdc1                        /dev/sdc1                              2G linux_raid_member
│ └─md0                       /dev/md0                               2G LVM2_member
└─sdc2                        /dev/sdc2                            511M linux_raid_member
  └─md1                       /dev/md1                            1018M LVM2_member
    └─vol_group1-logical_vol1 /dev/mapper/vol_group1-logical_vol1  100M ext4

```

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
 
 ```
 root@vagrant:/home/vagrant# mkdir /tmp/logical_vol1
 root@vagrant:/home/vagrant# mount /dev/vol_group1/logical_vol1 /tmp/logical_vol1/
 ```

---

13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
 
 ```
 root@vagrant:/home/vagrant# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/logical_vol1/test.gz
--2021-12-01 16:12:37--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22712347 (22M) [application/octet-stream]
Saving to: ‘/tmp/logical_vol1/test.gz’

/tmp/logical_vol1/test.gz                 100%[===================================================================================>]  21.66M  5.81MB/s    in 4.8s

2021-12-01 16:12:42 (4.50 MB/s) - ‘/tmp/logical_vol1/test.gz’ saved [22712347/22712347]
```
---

14. Прикрепите вывод `lsblk`.
 
 ```
 root@vagrant:/home/vagrant# lsblk
NAME                          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                             8:0    0   64G  0 disk
├─sda1                          8:1    0  512M  0 part  /boot/efi
├─sda2                          8:2    0    1K  0 part
└─sda5                          8:5    0 63.5G  0 part
  ├─vgvagrant-root            253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1          253:1    0  980M  0 lvm   [SWAP]
sdb                             8:16   0  2.5G  0 disk
├─sdb1                          8:17   0    2G  0 part
│ └─md0                         9:0    0    2G  0 raid1
└─sdb2                          8:18   0  511M  0 part
  └─md1                         9:1    0 1018M  0 raid0
    └─vol_group1-logical_vol1 253:2    0  100M  0 lvm   /tmp/logical_vol1
sdc                             8:32   0  2.5G  0 disk
├─sdc1                          8:33   0    2G  0 part
│ └─md0                         9:0    0    2G  0 raid1
└─sdc2                          8:34   0  511M  0 part
  └─md1                         9:1    0 1018M  0 raid0
    └─vol_group1-logical_vol1 253:2    0  100M  0 lvm   /tmp/logical_vol1
```

---

15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

1. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

1. Сделайте `--fail` на устройство в вашем RAID1 md.

1. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

1. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

1. Погасите тестовый хост, `vagrant destroy`.

 
 ---
