# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Выражение `c = a + b` вызовет ошибку - `TypeError: unsupported operand type(s) for +: 'int' and 'str'`  |
| Как получить для переменной `c` значение 12?  | `c = str(a) + b` или `c = f"{a}{b}"`  |
| Как получить для переменной `c` значение 3?  | `c = a + int(b)`  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3
import os

repo_path = "~/Desktop/netology/sysadm-homeworks"
bash_command = ["cd ~/Desktop/netology/sysadm-homeworks", "git status"]

result_os = os.popen(' && '.join(bash_command)).read()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(f"{repo_path}/{prepare_result}")

```

### Вывод скрипта при запуске при тестировании:
```
user@machine Desktop % ./test_py.py        
~/Desktop/netology/sysadm-homeworks/second_file
~/Desktop/netology/sysadm-homeworks/test_dir/third_file
~/Desktop/netology/sysadm-homeworks/test_file1.txt
user@machine Desktop %  
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import os
import sys

# проверяем, передан ли скрипту параметр
try:
    repo_path = sys.argv[1]
except IndexError:
    print("Вы не передали скрипту путь до репозитория в виде параметра")
    exit()

# перенаправление stderr в stdout добавлено специально, для того чтобы обрабатывать ситуации,
# когда заданной директории не существует или заданная директория не является репозиторием
bash_command = [f"cd {repo_path} 2>&1", "git status 2>&1"]

result_os = os.popen(' && '.join(bash_command)).read()

if 'not a git repository' in result_os:
    print("Заданный Вами путь не является репозиторием!")
    exit()

if 'No such file or directory' in result_os:
    print("Не удалось найти заданный Вами путь, проверьте корректность")
    exit()

for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(f"{repo_path}/{prepare_result}")


```

### Вывод скрипта при запуске при тестировании:
```
user@machine Desktop % ./test_py.py ~/Desktop/netology/sysadm-homeworks
/Users/user/Desktop/netology/sysadm-homeworks/second_file
/Users/user/Desktop/netology/sysadm-homeworks/test_dir/third_file
/Users/user/Desktop/netology/sysadm-homeworks/test_file1.txt
user@machine Desktop % ./test_py.py not_a_git_repo                              
Заданный Вами путь не является репозиторием!
user@machine Desktop % ./test_py.py folder_doesnt_exist
Не удалось найти заданный Вами путь, проверьте корректность
user@machine Desktop % ./test_py.py        
Вы не передали скрипту путь до репозитория в виде параметра
user@machine Desktop % 
```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import time
import socket

# список сервисов, которые нужно проверять
service_names = ['drive.google.com', 'mail.google.com', 'google.com']
# словарь, в котром будем хранить информацию о имени сервиса и его ip адресе
dns_ip = {}

# единожды сохраняем в словарь значения IP адреса для каждого сервиса, чтобы было с чем сравнивать далее
for service_name in service_names:
        ip = socket.gethostbyname(service_name)
        dns_ip[service_name] = ip

while True:
    # перебираем все сервисы
    for service_name in service_names:
        # определяем текущий ip адрес
        ip = socket.gethostbyname(service_name)

        # проверяем, изменился ли ip адрес с прошлой проверки
        if dns_ip[service_name] != ip:
            # если изменился, печатаем текст с ошибкой 
            print(f"[ERROR] {service_name} IP mismatch: {dns_ip[service_name]} {ip}")
            # сохраняем новое значение ip адреса сервиса в словарь
            dns_ip[service_name] = ip
        else:
            # ip адрес не изменился, печатаем инфо
            print(f"{service_name} - {ip}")
    # задержка в 5 секунд чтобы слишком часто не отправлять запросы
    time.sleep(5)



```

### Вывод скрипта при запуске при тестировании:
```
user@machine Desktop % ./test.py
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.18
google.com - 209.85.233.138
drive.google.com - 74.125.205.194
[ERROR] mail.google.com IP mismatch: 142.251.1.18 142.251.1.17
[ERROR] google.com IP mismatch: 209.85.233.138 209.85.233.100
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.17
google.com - 209.85.233.100
drive.google.com - 74.125.205.194
mail.google.com - 142.251.1.17
google.com - 209.85.233.100
^CTraceback (most recent call last):
  File "./test.py", line 31, in <module>
    time.sleep(5)
KeyboardInterrupt

```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 

### Ваш скрипт:
```python
???
```

### Вывод скрипта при запуске при тестировании:
```
???
```