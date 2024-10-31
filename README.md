# 1.	Введение в системное администрирование:

СРО: Проведите установку и настройку операционной системы на виртуальной машине (например, Ubuntu Server). Настройте пользователи, группы и простейшие сценарии автоматизации через cron. Подготовьте отчет с командной строкой действий и выводом системных команд.
СРОП: Практическая работа по созданию и настройке виртуальной сети в VirtualBox, установка серверных служб (например, SSH), управление пользователями и настройка прав доступа для них.

## 1. СРО

### 1.1 Установка Ubuntu Server на виртуальной машине

1. Скачивание ISO-образа:
   - ISO-образ Ubuntu Server был скачан с официального сайта [Ubuntu](https://ubuntu.com/download/).

2. Создание новой виртуальной машины в VirtualBox:
   - Открытие VirtualBox и нажатие "Создать".
   - Имя: `edward`
   - Тип: `Linux`
   - Версия: `Ubuntu (64-bit) (или 32-bit в зависимости от вашей системы)`
   - Оперативная память: `сколько не жалко (я выделил 8)`
   - Новый виртуальный жесткий диск (VDI), объем: `по дефолту 25 ГБ, можно оставить`.

3. Настройка параметров виртуальной машины:
   - В разделе "Носители" был выбран ISO-образ Ubuntu Server.
   - В разделе "Сеть" выбрана "Сетевой мост".

4. Запуск виртуальной машины:
   - Следование инструкциям установщика:
     - Выбор языка и раскладки клавиатуры.
     - Выбор "Установить Ubuntu Server".
     - Настройка сетевых параметров (обычно DHCP).
     - Создание учетной записи пользователя и задание пароля.
     - Выбор программного обеспечения (OpenSSH server).

5. Завершение установки:
   - Дождался завершения установки и перезагрузил виртуальную машину.

### 1.2 Настройка пользователей и групп

1. Добавление нового пользователя:
```
root@edward-VM:/home/edward# sudo adduser ed
info: Добавляется пользователь «ed» ...
info: Выбор UID/GID из диапазона от 1000 до 59999 ...
info: Добавляется новая группа «ed» (1001) ...
info: Добавление нового пользователя `ed' (1001) в группу `ed (1001)' ...
info: Создаётся домашний каталог «/home/ed» ...
info: Копирование файлов из «/etc/skel» ...
Новый пароль: 
НЕУДАЧНЫЙ ПАРОЛЬ: Пароль должен содержать не менее 8 символов
Повторите ввод нового пароля: 
passwd: пароль успешно обновлён
Изменение информации о пользователе ed
Введите новое значение или нажмите ENTER для выбора значения по умолчанию
	Полное имя []: 
	Номер комнаты []: 
	Рабочий телефон []: 
	Домашний телефон []: 
	Другое []: 
Данная информация корректна? [Y/n] y
info: Adding new user `ed' to supplemental / extra groups `users' ...
info: Добавляется пользователь «ed» в группу «users» ...
root@edward-VM:/home/edward# 
```

2. Создание новой группы:
```
root@edward-VM:/home/edward# sudo groupadd edgroup
```

3. Добавление пользователя в группу:
```
root@edward-VM:/home/edward# sudo usermod -aG edgroup ed
```

### 1.3 Настройка автоматизации через cron
Для начала подготовим скрипт для тестирования запуска задачи. 
Создадим файл log_datetime.sh. Это простой Bash-скрипта, который записывает текущую дату и время в файл.
Я привык использовать nano:
```
root@edward-VM:/home/edward/Documents# nano log_datetime.sh
```

```                     
#!/bin/bash

# Укажите путь к файлу, в который будет записываться дата и время
LOGFILE="/home/edward/Documents/logfile.txt"

# Запись текущей даты и времени в файл
echo "Текущая дата и время: $(date)" >> "$LOGFILE"
```
Сделаем скрипт исполняемым (выдадим права):
```
chmod +x /home/edward/Documents/log_datetime.sh
```

Создадим текстовый файл в который будут записываться данные:
```
touch /home/edward/Documents/logfile.txt
```
Проверим его права на запись и выдадим в случае необходимости:
```
root@edward-VM:/home/edward/Documents# ls -l /home/edward/Documents/logfile.txt
-rw-r--r-- 1 root root 144 окт 31 21:37 /home/edward/Documents/logfile.txt
root@edward-VM:/home/edward/Documents# chmod 666 /home/edward/Documents/logfile.txt
```

Теперь окройте crontab:
```
crontab -e
```

Добавьте запись:
```
* * * * * /home/edward/Documents/log_datetime.sh
```
Сохраните изменения и закройте редактор.

Проверка работы скрипта: 
```
cat /home/edward/Documents/logfile.txt
```

Вывод:
```
root@edward-VM:/home/edward/Documents# cat /home/edward/Documents/logfile.txt
Текущая дата и время: Чт 31 окт 2024 21:36:01 +05
Текущая дата и время: Чт 31 окт 2024 21:37:01 +05
Текущая дата и время: Чт 31 окт 2024 21:38:02 +05
```

Если вывода нет, можно попробовать перезапустить cron:
```
sudo systemctl restart cron
```

# СРОП

## Создание и настройка виртуальной сети

### 1. Создайте виртуальную сеть в VirtualBox:
- Перейдите в "Файл" -> "Настройки" -> "Сеть".
- Добавьте новую виртуальную сеть (например, "Host-Only").

### 2. Подключите виртуальную машину к созданной сети:
- В настройках виртуальной машины выберите "Сеть".
- Выберите "Сетевой адаптер 2" и установите его в режим "Host-Only Adapter".

### 3. Настройте серверные службы:
- Установите SSH-сервер, если это еще не сделано:
```
sudo apt update
sudo apt install openssh-server
```

- Убедитесь, что SSH работает:
```
root@edward-VM:/home/edward/Documents# systemctl status ssh
○ ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: ena>
     Active: inactive (dead)
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
lines 1-6/6 (END)
```

## Управление пользователями и настройка прав доступа:

### 1. Создайте пользователей и назначьте права:

```
sudo adduser ed
```

```
root@edward-VM:/home/edward/Documents# sudo chown ed:ed /home/edward/Documents/logfile.txt
sudo chmod 770 /home/edward/Documents/logfile.txt
```

```
root@edward-VM:/home/edward/Documents# ls -l /home/edward/Documents
итого 8
-rwxr-xr-x 1 root root 314 окт 31 21:28 log_datetime.sh
-rwxrwx--- 1 ed   ed   936 окт 31 21:48 logfile.txt
```
-----------------------------------------------------------





  
# 2.	Введение в сетевое администрирование:

## СРО: Настройте локальную сеть на двух виртуальных машинах с помощью статической и динамической IP-адресации. Настройте базовый файервол на уровне iptables и обеспечьте маршрутизацию трафика.

## СРОП: Проведите настройку и тестирование сетевого соединения между несколькими виртуальными машинами через VirtualBox, используя VLAN. Представьте отчет с анализом конфигураций и результатами сетевых проверок (ping, traceroute).





-----------------------------------------------------------

# 3.	Протоколы передачи данных и маршрутизация:

## СРО: Изучите и реализуйте маршрутизацию с использованием протоколов RIP и OSPF в Cisco Packet Tracer или GNS3. Проанализируйте различия в маршрутизации данных при добавлении нового узла в сеть.

## СРОП: Постройте сеть в Packet Tracer, реализующую статическую и динамическую маршрутизацию между несколькими узлами. Проверьте корректность маршрутов с помощью команд show ip route и предоставьте результаты.






-----------------------------------------------------------

# 4.	Управление пользователями и правами доступа:

## СРО: Настройте управление пользователями на сервере Linux. Создайте группы, назначьте им права на файлы и директории, ограничьте доступ к конфиденциальным данным с помощью команд chown и chmod.

## СРОП: Создайте несколько учетных записей с разными уровнями доступа (администратор, обычный пользователь) и продемонстрируйте, как правильно управлять их доступом к ресурсам в ОС. Приведите примеры на реальных сценариях использования прав доступа.


## СРО

### 1. Создание пользователей

1. Например, создайте несколько пользователей с префиксом `ed_`:

```
sudo adduser ed_test
sudo adduser ed_test_2
sudo adduser ed_admin
```

2. Создайте две группы: ed_group для обычных пользователей и ed_admin_group для администратора:
```
sudo groupadd ed_group
sudo groupadd ed_admin_group
```

3. Добавьте ed_test и ed_test_2 в группу ed_group, а ed_admin в группу ed_admin_group:
```
sudo usermod -aG ed_group ed_test
sudo usermod -aG ed_group ed_test_2
sudo usermod -aG ed_admin_group ed_admin
```

4. Создайте тестовый файл в директории Documents, который будет использоваться для назначения прав доступа:
```
touch /home/edward/Documents/testfile.txt
echo "Это тестовый файл." > /home/edward/Documents/testfile.txt
```

5. Теперь настройте права доступа для тестового файла:
```
sudo chown ed_admin:ed_admin_group /home/edward/Documents/testfile.txt
```

Установите владельца и группу:
```
sudo chown ed_admin:ed_admin_group /home/edward/Documents/testfile.txt
```

Установите права доступа:
```
sudo chmod 640 /home/edward/Documents/testfile.txt
```

Права 640 означают:
  - Владелец (администратор): чтение и запись (6)
  - Группа (ed_admin_group): чтение (4)
  - Остальные: нет прав (0)

6. Проверьте права доступа к файлу:
```
ls -l /home/edward/Documents/testfile.txt
```

Вывод:
```
edward@edward-VM:~$ ls -l /home/edward/Documents/testfile.txt
-rw-r----- 1 ed_admin ed_admin_group 34 окт 31 22:02 /home/edward/Documents/testfile.txt
```


## CРОП

7. Тестирование доступа
Войдите как ed_admin и проверьте доступ к файлу:
```
ed_admin@edward-VM:~$ su - ed_admin
cat /home/edward/Documents/testfile.txt
Пароль: 
```
Вывод для пользователя ed_admin:
```
ed_admin@edward-VM:~$ cat /home/edward/Documents/testfile.txt
Это тестовый файл.
ed_admin@edward-VM:~$
```

Для остальных пользователей:
```
cat: /home/edward/Documents/testfile.txt: Отказано в доступе
```
-----------------------------------------------------------

# 5.	Файловые системы и управление дисками:

## СРО: Выполните разметку жесткого диска, создайте разделы, установите файловую систему (например, ext4), настройте монтирование разделов в ОС Linux. Проведите анализ файловых систем с помощью утилит df, du, и предоставьте результаты.

## СРОП: Настройка RAID уровня 1 или 5 на виртуальной машине с подробным отчетом о выполненных действиях и тестировании отказоустойчивости системы.



## СРО
### 1. Определение дисков и их текущего состояния

Для начала необходимо определить, какие диски доступны в системе. Используйте команду:

```
sudo fdisk -l
```

```
edward@edward-VM:~$ sudo fdisk -l
[sudo] пароль для edward: 
Диск /dev/loop0: 74,25 MiB, 77852672 байт, 152056 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop1: 4 KiB, 4096 байт, 8 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop2: 11,71 MiB, 12283904 байт, 23992 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop3: 271,98 MiB, 285192192 байт, 557016 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop4: 10,72 MiB, 11239424 байт, 21952 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop5: 505,09 MiB, 529625088 байт, 1034424 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop6: 89,69 MiB, 94044160 байт, 183680 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop7: 13,99 MiB, 14667776 байт, 28648 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/sda: 25 GiB, 26843545600 байт, 52428800 секторов
Disk model: VBOX HARDDISK   
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: gpt
Идентификатор диска: BD35AB50-8203-41D0-B124-C712A9F75B8B

Устр-во    начало    Конец  Секторы Размер Тип
/dev/sda1    2048     4095     2048     1M BIOS boot
/dev/sda2    4096 52426751 52422656    25G Файловая система Linux


Диск /dev/sdb: 2 GiB, 2147483648 байт, 4194304 секторов
Disk model: VBOX HARDDISK   
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop8: 10,67 MiB, 11186176 байт, 21848 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop9: 38,83 MiB, 40714240 байт, 79520 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт


Диск /dev/loop10: 564 KiB, 577536 байт, 1128 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
```

### 2. Разметка жесткого диска

Выберите диск, который вы хотите разметить (для теста был создан новый диск на 2 Гб, /dev/sdb). Запустите fdisk для разметки:
```
sudo fdisk /dev/sdb
```

Далее следуйте этим шагам:
	- Нажмите n для создания нового раздела.
	- Выберите p для создания первичного раздела.
	- Укажите номер раздела (например, 1).
	- Нажмите Enter, чтобы использовать значение по умолчанию для первого сектора.
	- Нажмите Enter, чтобы использовать значение по умолчанию для последнего сектора (для использования всего диска).
	- Нажмите w, чтобы сохранить изменения и выйти.

Выберите диск, который вы хотите разметить (в данном случае /dev/sdb). Запустите fdisk для разметки:

```
edward@edward-VM:~$ sudo fdisk /dev/sdb

Добро пожаловать в fdisk (util-linux 2.40.2).
Изменения останутся только в памяти до тех пор, пока вы не решите записать их.
Будьте внимательны, используя команду write.

Устройство не содержит стандартной таблицы разделов.
Created a new DOS (MBR) disklabel with disk identifier 0xa39d2531.

Команда (m для справки): n
Тип раздела
   p   основной (0 primary, 0 extended, 4 free)
   e   расширенный (контейнер для логических разделов)
Выберите (по умолчанию - p): p
Номер раздела (1-4, по умолчанию 1): 
Первый сектор (2048-4194303, по умолчанию 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-4194303, по умолчанию 4194303): 

Создан новый раздел 1 с типом 'Linux' и размером 2 GiB.

Команда (m для справки): w
Таблица разделов была изменена.
Вызывается ioctl() для перечитывания таблицы разделов.
Синхронизируются диски.
```

### 3. Форматирование раздела

Теперь необходимо отформатировать созданный раздел в файловую систему ext4. Предположим, что раздел будет /dev/sdb1:
```
edward@edward-VM:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.47.1 (20-May-2024)
Creating filesystem with 524032 4k blocks and 131072 inodes
UUID файловой системы: 08152abf-60aa-46e8-9666-07ef25b71a4a
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Распределение групповых таблиц: готово                            
Сохранение таблицы inod'ов: готово                            
Создание журнала (8192 блоков): готово
Writing superblocks and filesystem accounting information: готово
```

### 4. Создание точки монтирования

Создайте директорию, где будет монтироваться раздел:

```
sudo mkdir /mnt/mydata
```

### 5. Монтирование раздела

Теперь смонтируйте созданный раздел в только что созданную директорию:
```
edward@edward-VM:~$ sudo mkdir /mnt/mydata
```

### 6. Добавление в fstab для автоматического монтирования

Чтобы раздел автоматически монтировался при загрузке системы, добавьте его в файл /etc/fstab. Откройте файл для редактирования:
```
sudo nano /etc/fstab
```

Добавьте строку в конец файла:
```
/dev/sdb1 /mnt/mydata ext4 defaults 0 2
```

### 7. Анализ файловых систем с помощью утилит df и du

df: Для отображения информации о свободном и используемом пространстве на файловых системах выполните:
```
edward@edward-VM:~$ df -h
Файл.система   Размер Использовано  Дост Использовано% Cмонтировано в
tmpfs            743M         1,7M  742M            1% /run
/dev/sda2         25G          10G   14G           43% /
tmpfs            3,7G            0  3,7G            0% /dev/shm
tmpfs            5,0M         8,0K  5,0M            1% /run/lock
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-journald.service
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-udev-load-credentials.service
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-tmpfiles-setup-dev-early.service
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-sysctl.service
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-tmpfiles-setup-dev.service
tmpfs            3,7G         3,6M  3,7G            1% /tmp
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-tmpfiles-setup.service
tmpfs            1,0M            0  1,0M            0% /run/credentials/systemd-resolved.service
tmpfs            743M         128K  743M            1% /run/user/1000
```

du: Для анализа использования дискового пространства в конкретной директории (например, /mnt/mydata) выполните:
```
edward@edward-VM:~$ du -sh /mnt/mydata
4,0K	/mnt/mydata
```

# СРОП

## Настройка RAID уровня 1 на виртуальной машине

### 1. Создание виртуальных дисков

Создайте два виртуальных диска (например, 10 ГБ каждый) на вашей виртуальной машине.

### 2. Убедитесь, что диски доступны

Запустите команду для проверки доступных дисков:
```
sudo fdisk -l
```

### 3. Установка необходимых утилит

Для настройки RAID вам понадобится утилита mdadm. Установите её с помощью следующей команды:
```
sudo apt update
sudo apt install mdadm
```
### 4. Создание RAID 1

Выполните команду для создания RAID 1 из двух дисков:





-----------------------------------------------------------

# 6.	Резервное копирование и восстановление данных:

## СРО: Настройте автоматизированную систему резервного копирования с использованием утилит rsync и cron. Выполните полный цикл: создание резервной копии, внесение изменений в оригинальные данные и восстановление резервной копии.

## СРОП: Создайте план резервного копирования и восстановления для серверов, опишите процессы использования инкрементальных и полных бэкапов. Проведите тестирование восстановления данных после повреждения файловой системы.



# 7.	Мониторинг и оптимизация систем:

## СРО: Настройте мониторинг системных ресурсов (процессор, оперативная память, диск) с помощью утилит htop, iotop, df и представьте анализ производительности системы за период наблюдения.

## СРОП: Установите и настройте инструмент мониторинга, такой как Zabbix или Nagios, для отслеживания состояния серверов и сетевых ресурсов. Создайте отчет с графиками и анализом.



# 8.	Виртуализация и контейнеризация:

## СРО: Установите и настройте виртуальные машины с помощью VirtualBox или VMware. Подготовьте отчет по созданию и настройке гипервизора, сетевого окружения и настройки доступа к виртуальным машинам.

## СРОП: Настройте Docker-контейнеры для развертывания веб-сервиса. Включите в отчет подробное описание процесса создания Dockerfile, сборки контейнера и проверки его работоспособности на реальном сервере.
