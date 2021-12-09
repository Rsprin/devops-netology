# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

1. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
    * Нет, не могут, т.к. жесткие ссылки и файл имеют  одинаковую inode

1. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

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

1. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
    * вводим команду `fdisk /dev/sdb`
    * вводим команду `n` - add a new partition
    * выбираем `Partition type` - p   primary
    * оставляем дефолтное значение Partition number (1-4, default 1):
    * оставляем дефолтное значение первого сектора - First sector (2048-5242879, default 2048):
    * вводим требуемый размер, в нашем случае `+2G`, первый раздел готов
    * вводим команду `n` - add a new partition
    * выбираем `Partition type` - p   primary
    * оставляем дефолтное значение Partition number (2-4, default 2):
    * оставляем дефолтное значение первого сектора - First sector (4196352-5242879, default 4196352):
    * так как нам нужно выбрать все оставшееся пространство под раздел, то просто нажимаем enter при выборе последнего сектора(будет выделен весь неразмеченный размер)
    * далее вводим команду `w` чтобы записать изменения на диск и выйти
    
    в итоге получаем 
    ```bash
    Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xf09e753e
    
    Device     Boot   Start     End Sectors  Size Id Type
    /dev/sdb1          2048 4196351 4194304    2G 83 Linux
    /dev/sdb2       4196352 5242879 1046528  511M 83 Linux 
   ```

1. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
    * `sfdisk -d /dev/sdb | sfdisk /dev/sdc`
    
    * в итоге получаем 
    ```bash
    Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xf09e753e
    
    Device     Boot   Start     End Sectors  Size Id Type
    /dev/sdb1          2048 4196351 4194304    2G 83 Linux
    /dev/sdb2       4196352 5242879 1046528  511M 83 Linux
    
    
    Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
    Disk model: VBOX HARDDISK
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xf09e753e
    
    Device     Boot   Start     End Sectors  Size Id Type
    /dev/sdc1          2048 4196351 4194304    2G 83 Linux
    /dev/sdc2       4196352 5242879 1046528  511M 83 Linux 
    ```
    
1. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
    * для этого введем следующую команду:
    * `mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1`
    
1. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
    * для этого введем следующую команду:
    * `mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2`

1. Создайте 2 независимых PV на получившихся md-устройствах.
    * `pvcreate /dev/md1`
    * `pvcreate /dev/md0`

1. Создайте общую volume-group на этих двух PV.
    ```bash
    root@vagrant:/home/vagrant# vgcreate test_vg /dev/md0 /dev/md1
      Volume group "test_vg" successfully created
    root@vagrant:/home/vagrant# 
   ```

1. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
    ```bash
    root@vagrant:/home/vagrant# lvcreate -L100 -n testlv test_vg /dev/md0
      Logical volume "testlv" created.
    root@vagrant:/home/vagrant# 
   ```

1. Создайте `mkfs.ext4` ФС на получившемся LV.
    ```bash
    root@vagrant:/home/vagrant# mkfs -t ext4 /dev/test_vg/testlv
    mke2fs 1.45.5 (07-Jan-2020)
    Creating filesystem with 25600 4k blocks and 25600 inodes
    
    Allocating group tables: done
    Writing inode tables: done
    Creating journal (1024 blocks): done
    Writing superblocks and filesystem accounting information: done
    
    root@vagrant:/home/vagrant# 
   ```

1. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
    ```bash
    root@vagrant:/home/vagrant# mkdir /mymount
    root@vagrant:/home/vagrant# mount /dev/test_vg/testlv /mymount
    root@vagrant:/home/vagrant# df | grep mymount
    /dev/mapper/test_vg-testlv     95088        72     87848   1% /mymount
    root@vagrant:/home/vagrant#
   ```

1. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
    ```bash
    root@vagrant:/mymount# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O test.gz
    ...
    2021-12-09 18:54:24 (3.55 MB/s) - ‘test.gz’ saved [22807871/22807871]
    root@vagrant:/mymount#
   ```

1. Прикрепите вывод `lsblk`.

    ```bash
     root@vagrant:/mymount# lsblk
    NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    sda                    8:0    0   64G  0 disk
    ├─sda1                 8:1    0  512M  0 part  /boot/efi
    ├─sda2                 8:2    0    1K  0 part
    └─sda5                 8:5    0 63.5G  0 part
      ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
      └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
    sdb                    8:16   0  2.5G  0 disk
    ├─sdb1                 8:17   0    2G  0 part
    │ └─md1                9:1    0    2G  0 raid1
    └─sdb2                 8:18   0  511M  0 part
      └─md0                9:0    0 1018M  0 raid0
        └─test_vg-testlv 253:2    0  100M  0 lvm   /mymount
    sdc                    8:32   0  2.5G  0 disk
    ├─sdc1                 8:33   0    2G  0 part
    │ └─md1                9:1    0    2G  0 raid1
    └─sdc2                 8:34   0  511M  0 part
      └─md0                9:0    0 1018M  0 raid0
        └─test_vg-testlv 253:2    0  100M  0 lvm   /mymount
    root@vagrant:/mymount#
   ```

1. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
   
   ```bash
    root@vagrant:/mymount# gzip -t test.gz
    root@vagrant:/mymount# echo $?
    0
   ```

1. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
    ```bash
    root@vagrant:/mymount# pvmove /dev/md0 /dev/md1
      /dev/md0: Moved: 24.00%
      /dev/md0: Moved: 100.00%
    root@vagrant:/mymount#
   ```

1. Сделайте `--fail` на устройство в вашем RAID1 md.
    ```bash
    root@vagrant:/mymount# mdadm /dev/md1 --fail --verbose  /dev/sdc1
    mdadm: set /dev/sdc1 faulty in /dev/md1
    root@vagrant:/mymount#
   ```

1. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
     
   ```bash
   [140029.460455] md/raid1:md1: Disk failure on sdc1, disabling device.
                md/raid1:md1: Operation continuing on 1 devices. 
   ```

1. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
   * Да, файл все также доступен
   ```bash
    root@vagrant:/mymount# gzip -t test.gz
    root@vagrant:/mymount# echo $?
    0
   ```

1. Погасите тестовый хост, `vagrant destroy`.
    * Сделано
 
 ---
