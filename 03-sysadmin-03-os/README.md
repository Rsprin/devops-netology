# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.
    * `chdir("/tmp")  = 0`
1. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.

    В моем случае искал последовательно в следующих местах:
    * `/home/vagrant/.magic.mgc`
    * `/etc/magic`
    * `usr/share/misc/magic.mgc`
1. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
    * можно усечь файл, например вот такой командой - 
    ```
        vagrant@vagrant:~$ : > /proc/9490/fd/3
    ```
1. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
    * Нет, не занимают. 
1. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
    * Не совсем понял что необходимо сделать, утилиту установил, воспользовался, вот пример работы:
    ```bash
        vagrant@vagrant:~$ sudo /usr/sbin/opensnoop-bpfcc -T -d 10
        TIME(s)       PID    COMM               FD ERR PATH
        0.000000000   614    irqbalance          6   0 /proc/interrupts
        0.000098000   614    irqbalance          6   0 /proc/stat
        0.000143000   614    irqbalance          6   0 /proc/irq/20/smp_affinity
        0.000160000   614    irqbalance          6   0 /proc/irq/0/smp_affinity
        0.000176000   614    irqbalance          6   0 /proc/irq/1/smp_affinity
        0.000190000   614    irqbalance          6   0 /proc/irq/8/smp_affinity
        0.000205000   614    irqbalance          6   0 /proc/irq/12/smp_affinity
        0.000219000   614    irqbalance          6   0 /proc/irq/14/smp_affinity
        0.000234000   614    irqbalance          6   0 /proc/irq/15/smp_affinity
        2.466461000   806    vminfo              4   0 /var/run/utmp
        2.466623000   607    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
        2.466649000   607    dbus-daemon        18   0 /usr/share/dbus-1/system-services
        2.466717000   607    dbus-daemon        -1   2 /lib/dbus-1/system-services
        2.466729000   607    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
        7.483841000   806    vminfo              4   0 /var/run/utmp
        7.484166000   607    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
        7.484305000   607    dbus-daemon        18   0 /usr/share/dbus-1/system-services
        7.484521000   607    dbus-daemon        -1   2 /lib/dbus-1/system-services
        7.484615000   607    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
        9.989480000   614    irqbalance          6   0 /proc/interrupts
        9.989598000   614    irqbalance          6   0 /proc/stat
        9.989652000   614    irqbalance          6   0 /proc/irq/20/smp_affinity
        9.989673000   614    irqbalance          6   0 /proc/irq/0/smp_affinity
        9.989691000   614    irqbalance          6   0 /proc/irq/1/smp_affinity
        9.989709000   614    irqbalance          6   0 /proc/irq/8/smp_affinity
        9.989728000   614    irqbalance          6   0 /proc/irq/12/smp_affinity
        9.989746000   614    irqbalance          6   0 /proc/irq/14/smp_affinity
        9.989828000   614    irqbalance          6   0 /proc/irq/15/smp_affinity 
   ```
1. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
    * `uname -a` использует системный вызов `uname()`
    * > Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
1. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
   
    * последовательность команд через `;` - все команды выполнятся, вне зависимости от их exit code
    * последовательность команд через `&&` - вторая команда выполнится лишь в том случае, если exit code первой команды равен нулю(успешное выполнение)
    
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
    
    * Смысла использовать `&&` с примененным `set -e` вероятно нет, т.к. результат будет одинаков(выполнение завершится при первом не нулевом exit code)
    * c примененным `set -e` можно использовать `;` в последовательностях команд, тогда при первом ненулевом exit code выполнение последовательности команд прекратится(т.е. с помощью `set -e` можно как бы переопределить работу `;`)
    
1. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
    * опция `-e` - Прерывает работу сценария при появлении первой же ошибки (когда команда возвращает ненулевой код завершения)
    * опция `-u` - при попытке обращения к неопределенным переменным, будет выдано сообщение об ошибке и прервана работа сценария
    * опция `-x` - выводит на stdout команды перед их исполнением, с подставленными значениями
    * опция `-o pipefail` - позволяет сделать так, чтобы код завершения всего пайплайна был самым последним из не нулевых в пайплайне или нулевым, если все команды в пайплайне завершились успешно(с нулевым кодом завершения). Без указания этой опции код завершения всего пайплайна это код завершения последней его команды.
    * Все эти опции на мой взгляд делаю скрипт более "предсказуемым" и ошибки будут обнаружены как можно скорее.У скрипта не будет "неожиданного" поведения. А опция `-x` поможет при отладке скриптов.
1. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
    * самые часто встречающие статусы у процессов это
        * S  -  interruptible sleep (waiting for an event to complete)
        * R  -  running or runnable (on run queue)
        