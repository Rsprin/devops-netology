# Домашнее задание к занятию "3.7. Компьютерные сети, лекция 2"

1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?
    * в Linux - `ip link`
    ```bash
    vagrant@vagrant:~$ ip link
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:73:60:cf brd ff:ff:ff:ff:ff:ff
    ```
    * в Windows - 'netsh lan show interfaces'
2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?
    * протокол NDP (Neighbor Discovery Protocol)
    * Используется пакет `libndp-tools`
    * Можно использовать команду `ndptool`
3. Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.
    * Технология `VLAN`
    * есть команда `vconfig` из пакета `vlan`
    * пример конфига, который  указываются в файле /etc/network/interfaces
    ```
    auto vlan1400
    iface vlan1400 inet static
            address 192.168.1.1
            netmask 255.255.255.0
            vlan_raw_device eth0
    ```
4. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.
    * mode=0 (balance-rr) 
    * mode=1 (active-backup)
    * mode=2 (balance-xor)
    * mode=3 (broadcast)
    * mode=4 (802.3ad)
    * mode=5 (balance-tlb)
    * mode=6 (balance-alb)
    * Описывать тут их наверное нет смысла, читал, например, тут - https://bogachev.biz/2015/02/19/tipi-agregatsii-interfeisov-v-linux/
    
    Для балансировки нагрузки можно использовать режим `mode = 2 (balance-xor)` или `mode = 6 (balance-alb)` или `mode=0 (balance-rr)`, который рекомендован по умолчанию
    
    пример конфига /etc/network/interfaces для объединения четырех интерфейсов
    ```
    auto bond0
    iface bond0 inet static
    address 10.10.250.2
    netmask 255.255.255.0
    network 10.10.250.0
    bond_mode balance-rr
    bond_miimon 100
    bond_downdelay 200
    bond_updelay 200
    slaves eth1 eth2 eth3 eth4
   ```
5. Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.
    * в сети с маской /29 `8` IP адресов 
    * из сети с маской /24 можно получить  `32` /29 подсетей
    * `10.10.10.224/29`, `10.10.10.192/29`, `10.10.10.16/29` 
6. Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.
    * Допустимо взять из подсети `100.64.0.0`,  например `100.64.0.0/26`
7. Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?
    * Как проверить ARP таблицу в Linux, Windows? Например команда - `arp -a` или в Linux еще есть - `ip neighbour`
    * Как очистить ARP кеш полностью? - `ip neigh flush all`
    * Как из ARP таблицы удалить только один нужный IP? - `arp -d 192.168.100.25`
    

