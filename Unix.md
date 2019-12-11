# Теория



## Блочное устройство

Блочное устройство (block device) — вид файла устройств в UNIX/Linux-системах, обеспечивающий интерфейс к устройству, реальному или виртуальному, в виде файла в файловой системе.

С блочным устройством обеспечивается обмен данными блоками данных. Типичные примеры блочных устройств: жёсткий диск, CD-ROM, НГМД.

$ ls -l /dev/fd0 

brw-rw--— 1 root floppy 2, 0 Jan 1 11:17 /dev/fd0 

 

Первый символ в расширенном выводе ls (ls -l) для блочных устройств — буква b (block).

## Ссылки

**Жёсткая ссылка (****hard** **link)** – связывает inode файла с каталогом и дает имя файлу. Файл в UFS представляет собой структуру блоков данных на диске, имеющую уникальный индексный дескриптор (i-node) и набор атрибутов (метаинформацию). У файла может быть несколько жёстких ссылок, ведущих на одну inode: в таком случае он будет фигурировать на диске одновременно в различных каталогах или под различными именами в одном каталоге.

Количество жёстких ссылок файла сохраняется на уровне файловой системы в метаинформации. Файлы с нулевым количеством ссылок перестают существовать для системы и, со временем, будут перезаписаны физически. Все жесткие ссылки одного файла равноправны и неотличимы друг от друга. 

Жёсткая ссылка ссылаются на индексный дескриптор, уникальный в пределах дискового раздела, поэтому создание жёсткой ссылки на файл в каталоге другого раздела невозможно. Для преодоления этого ограничения используются символьные (символические) ссылки.

Создание жесткой ссылки: 

**ln** файл имя_ссылки

Аналогом команды `rm`, является команда *unlink*, удаляющая из каталога жесткую ссылку.

Символическая («мягкая») ссылка («симлинк», Symbolic link) — специальный файл в файловой системе, в котором вместо пользовательских данных содержится путь к файлу, открываемому при обращении к данной ссылке (файлу). Создание:

**ln** -s файл имя_ссылки

Если «файл» не существует, символическая ссылка всё равно будет создана.

## PATH

Просмотр переменной среды `PATH`:

```bash
echo $PATH
```

Отдельные директории разделены символом `:`

```bash
/usr:/usr/bin
```

Команды делятся на:

- встроенные (*built-in utility*) – являются частью оболочки и всегда могут быть найдены. Например, `cd`, `echo`, `kill`.
- исполняемые (*executable*) – сторонняя программа, которую *shell* должен найти.

Порядок поиска команды после ввода в *shell*:

- если *command name* не содержит ни одного `/`, то:
  - совпадает ли *command name* с одной из *built-in utility*
  - поиск *utility* в каталогах, указанных в `PATH`.   
- если *command name* содержит хотя бы один `/`, то исполнить *utility* по этому пути

Т.е. текущий каталог не проверяется и поэтому чтобы запустить *utility* в текущем каталоге необходимо использовать формат `./<command>`.

Можно добавить текущий каталог `.` в `PATH`:

```bash
PATH=$PATH:.
```

однако это не рекомендуется по соображения безопасности. Например, можно случайно вместе *built-in utility* запустить *executable utility* в текущем каталоге. 

## tar

**tar** (*tape archive*) – формат битового потока или файла архива, а также название традиционной для *Unix* программы для работы с такими архивами. Используется для хранения нескольких файлов внутри одного файла, например, для создания архива части файловой системы. 

Преимущество: в архив записывается информация о структуре каталогов, о владельце и группе отдельных файлов, а также временны́е метки файлов.

Самостоятельно не создает сжатые архивы, использует для сжатия внешние утилиты (например, gzip).

<u>Синтаксис:</u>

```
tar [-опции] <имя_архива> [файлы или папки, которые необходимо поместить в архив]
```

<u>Опции:</u>

- `-c, --create` — создать архив;
- `-a, --auto-compress` — дополнительно сжать архив с компрессором который автоматически определить по расширению архива. Если имя архива заканчивается на `*.tar.gz` то с помощью *gzip* и т.д. 
- `-x, --extract, --get` — извлечь файлы из архива;
- `-f, --file` — указать имя архива;
- `-v, --verbose` — выводить список обработанных файлов.

<u>Примеры:</u>

Создание архива `archive.tar` из файла `README.txt` и каталога `src`:

```
tar -cvf archive.tar README.txt src
```

Извлечение содержимого `archive.tar` в текущий каталог:

```
tar -xvf archive.tar
```

Создание сжатого *gzip* архива `archive.tar.gz` из файла `README.txt` и каталога `src`:

```
tar -cavf archive.tar.gz README.txt src
```

Извлечение содержимого `archive.tar.gz` в текущий каталог:

```
tar -xvf archive.tar.gz
```

## Команды

### Работа с консолью

#### echo

echo [OPTION]... [STRING]...

Вывести строку текста в stdout.

**Параметры:**

| -n   | не выводить в  конце символ новой строки                 |
| ---- | -------------------------------------------------------- |
| -e   | включить  интерпретацию управляющих символов (в т.ч. \n) |

**Примеры:**

Вывод строки с управляющим символом в stdout

$ echo -e "123\n345"

 

123

345

Отправка HTTP-запроса на yandex.ru:80

$ echo -en "GET / HTTP/1.1\r\nHost: yandex.ru\r\n\r\n" | nc yandex.ru 80

 

HTTP/1.1 302 Found

Date: Thu, 07 Jun 2018 11:50:35 GMT

...

Можно использовать для создания файла:

$ echo "string" > filename

$ cat filename

string

 

### Пакетный менеджер APT

Advanced Packaging Tool (APT) – набор утилит (apt-get, apt-cache...) для управления программными пакетами в операционных системах основанных на Debian. APT предоставляет дружественную надстройку над упаковочной системой DPKG.

Инструмент «dpkg» формирует базовый упаковочный уровень, APT предоставляет удобные интерфейсы и осуществляют обработку зависимостей. 

#### apt-get

Утилита управления пакетами: установка, обновление, удаление.

Требует привелегий суперпользователя.

##### apt-get update

Обновить информацию о пакетах, содержащихся в репозиториях.

sudo apt-get update

##### apt-get upgrade

Обновление в системе пакетов, для которых в репозитории доступны новые версии.

sudo apt-get upgrade

##### apt-get install

Установить пакет.

sudo apt-get install nginx

##### apt-get remove

Удалить пакеты без удаления его конфигурационных файлов:

sudo apt-get remove имя_пакета

##### apt-get purge

Удалить пакет вместе с его конфигурационными файлами:

sudo apt-get purge имя_пакета

Можно использовать маску с символом «*»:

sudo apt purge php7.1*

##### apt-get autoremove

Автоматическое удаления пакетов, которые были установлены как зависимости других пакетов, но сейчас больше не нужны. 

Стоит запускать после удаления пакетов через apt-get remove и apt-get purge.

sudo apt-get autoremove

Также можно указать имя пакета после команды «autoremove», чтобы удалить пакет и его зависимости.

sudo apt-get autoremove php7.1

#### apt-cache

Утилита для запроса информации в базе данных (кеше) пакетов. 

Не требует привелегий суперпользователя

##### apt-cache search

Поиск пакетов по ключевому слову:

apt-cache search ключевое_слово

#### apt-key

apt-key используется для управления списком ключей, используемых apt для аутентификации пакетов. 

##### adv

Передать дополнительные параметры в gpg. С помощью ключа -recv можно загрузить ключ из серверов ключей непосредственно в доверенный набор ключей.

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com: 80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5

 

#### add-apt-repository

Добавление репозитория

sudo add-apt-repository ppa:ondrej/php

#### dpkg

dpkg - это пакетный менеджер для Debian систем. 

Получение списка пакетов, установленных в системе:

dpkg --list

##### Статусы пакетов

·   r – пакет был отмечен для удаления

·   c – файлы конфигурации в настоящее время присутствуют в системе

·   rc –пакет не полностью удален, файлы конфигурации все еще присутствуют.

 

### Управление сервисом service

#### service start

Старт сервиса:

**sudo service** serviceName **start**

#### service stop

Остановка сервиса:

**sudo servioe** serviceName **stop**

#### service restart

Перезапуск сервиса

**sudo service** serviceName **restart**

#### service reload

Перечитывание конфигурационных файло без перезапуска сервиса

**sudo servi****сe** serviceName **reload**

### Файловые операции

#### ls

**-****l** Вывод информации из inode (куда ведет ссылка, какие права доступа, кто владелец)

[pparshikov@cup ~]$ ls -l /etc/sphinx/sphinx.conf

lrwxrwxrwx 1 root root 52 Nov 7 16:57 /etc/sphinx/sphinx.conf -> /www/fishki6.cup.ondu.ru/misc/sphinx_fishki_cup.conf

**-****a** показать скрытые файлы

**-****I** номер inode

#### **mkdir**

Создание директории 

**mkdir** dir1

**Создать вложенные друг в друга папки**

**mkdir -p** /tmp/dir1/dir2

#### mv

Перемещение файла и каталога

**mv** id_dsa .ssh/

#### **ln**    

Смотреть Ссылки.

## Системные команды

### id

### sudo

`-u` – позволяет указать *user*, от которого идет запуск команды

```bash
sudo -u<name> <command>
sudo -upavelparshikov php ...
```

### pbcopy

Выполняет сохранение стандартного вывода в буфер обмена:

- Сохранение содержимого файла в буфер обмена:

  ```bash
  pbcopy < {file}
  ```

- Сохранение вывода программы в буфер обмена:

```
{command} | pbcopy
```



###  which 

Выполняет поиск по переменной окружения PATH и возвращает первый найденный результат, а также путь к нему:

$ which wvdial 

/usr/bin/wvdial

### uname 

информацию о системе.

[pparshikov@cup fishki6.cup.ondu.ru]$ uname

Linux

[pparshikov@cup fishki6.cup.ondu.ru]$ uname -a

Linux cup.relax.ru 2.6.32-573.7.1.el6.x86_64 #1 SMP Tue Sep 22 22:00:00 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

Для получения информации от ядра о состоянии используются файловая система **proc** и **sys**. Все поддиректории, файлы и хранящаяся в них информация генерируется ядром на лету, как только вы ее запрашиваете. 

### ps

Информация о процессах. 

Без параметров выдает список процессов в текущей сессии. 

**$ ps**

 **PID TTY     TIME CMD**

 5364 pts/2  00:00:00 bash

 5541 pts/2  00:00:00 ps

Параметры:

**ax** вывести все процессы.

**u** вывести пользователей, запустивших процесс

В выводе команды PID (Process IDentifier) – идентификатор процесса; PPID (Parent Process IDentifier) – идентификатор родительского процесса.

**$ ps -aux**

**USER    PID %CPU %MEM  VSZ  RSS TTY   STAT START  TIME COMMAND**

root     1 0.0 0.4 119848 4496 ?    Ss  17:48  0:03 /sbin/init

root     2 0.0 0.0   0   0 ?    S  17:48  0:00 [kthreadd]

**-****p** **PID** вывести информацию о процессе

**-****T –****p** **PID** вывести информацию о потоках процесса.

SPID – идентификатор потока

~$ ps -T -p 5536

 PID SPID TTY     TIME CMD

 5536 5536 ?    00:00:00 nginx

### top

**top –****H** вывести информацию потоках

**top –****H –****p** **PID** вывести информацию о потоках процесса

### systemctl

Рестарт процесса

systemctl restart serviceName

аналог

service serviceName restart

 

### rm

Удалить файл

\# Удалить файл
 rm ~/file.ext

 \# Удалить директорию dir вместе с файлами и поддиректориями (ключ -R)
 rm -R ~/dir

 \# Удаление без вопросов (ключ -f)
 rm -Rf ~/dir

 \# Запрос на удаление каждого файла (ключ -f)
 rm -Ri ~/dir

### chmod

Установка прав

chmod 700 .ssh

### kill

Послать сигнал

kill -SIGUSR1 <PID>

### ipcs

Отображает состояние всех объектов System V IPC.

------ Очереди сообщений --------

ключ  msqid   владелец права исп. байты сообщения

 

------ Сегменты совм. исп. памяти --------

ключ  shmid   владелец права байты nattch   состояние

0x00000000 753664   pavel   600    524288   2     назначение    

 

------ Массивы семафоров --------

ключ  semid   владелец права nsems   

-q: показывать только очереди сообщений 

-s: показывать только семафоры 

-m: показывать только разделяемую память

### ipcrm

Удалить объект IPC из ядра. 

ipcrm {shm|msg|sem} id

### scp

Копирование файла из локали на сервер через ssh-сессию:

scp path/myfile user@8.8.8.8:/full/path/to/new/location/

с сервера на локаль:

scp user@8.8.8.8:/full/path/to/file /path/to/put/here

**-r** — рекурсивное копирование директорий;

### wget

Консольная программа для загрузки файлов по сети. Поддерживает протоколы HTTP, FTP и HTTPS. Файлы можно скачивать рекурсивно по ссылкам в HTML страницах с определённой глубиной следования по ссылкам. При загрузке по ftp файлы можно скачивать «по маске» имени. Wget поддерживает докачку файла в случае обрыва соединения.

Скачать файл 

wget http://fishki.net

### top

????

Load Average

### touch

### find

Узначать что это такое? Меняли на фишках права на файлы.

find . -type d -exec chmod 775 \{\} \;

find . -type f -exec chmod 660 \{\} \;

### screen

Позволяет управлять несколькими сессиями из одной консоли или окна терминала. Часто используется для сворачивания в фон программ, которые сами этого не умеют, с возможностью последюущего возврата к ним. 

Преимущества:

- к одному сеансу можно подключиться из нескольких мест одновременно
- можно взаимодействовать с несколькими сессиями. 
- при потере связи по `ssh` можно вернуться в свою сессию и выполняемые в момент разрыва операции не прервутся. 

*Screen* создает отдельные объекты, называемые «скринами». Каждый скрин - это что-то вроде окна, которое можно свернуть-развернуть, если проводить аналогию с графическим интрефейсом. 

Команды:

- создание нового скрина

  ```bash
  screen
  ```

- отключиться от скрина – сочетание клавиш `Ctrl + A`, затем `D`

- список активных скринов:

  ```bash
  screen -ls
  ```

- подключиться к скрину с указанным именем

  ```bash
  screen -r <name>
  ```

  

### Фильтрация текста

#### awk

Инструмент фильтрации текста. Часто используется для выбора одного из полей, разделенных пробелами

awk '{print $2}'

#### wc

wc [OPTION]... [FILE]...

wc (сокращение от word count).

Программа считывает файл и выводит количество переводов строк \n (т.е. количество строк, 1 колонка), количество слов (2 колонка) и количество байт (3 колонка). 

$ wc test

2  4  16 test

Если указан список файлов, то выводится статистика по отдельным файлам и общая статистика.

$ wc test1.txt test2.txt 

 2 4 16 test1.txt

 3 6 37 test2.txt

 5 10 53 итого

Аналогично выводится результат, если использована маска:

$ wc *.txt

 2 4 16 test1.txt

 3 6 37 test2.txt

 5 10 53 итого

Если входной файл не задан, или равен ‘-‘, то данные считываются с stdin. В выводе команды отсутствует название файла.

$ **wc**

123

345

(Ctrl+D)

 2 2 8

 

$ echo -en '123\n345 678\n' | wc

 2  3 12

Параметры:

| -l   | вывести  количество переводов строк \n (т.е. количество строк) |
| ---- | ------------------------------------------------------------ |
| -w   | вывести  количество слов                                     |
| -c   | вывести количество байт. Может применяться к  нетекстовым файлам |
| -m   | вывести количество символов                                  |
| -L   | вывести длину самой длинной строки                           |

 

$ wc -l test1.txt

2 test1.txt

 

$ echo -en '123\n345 678\n' | wc -l

2

Примеры:

Найти количество попыток доступа с определенного IP адреса к nginx:

$ cat /logs/nginx/fishki.net_access.log | grep '171.25.171.101' | wc –l

 

17

Вывести суммарное количество строк в файлах, заданных маской:

$ cat *.txt | wc -l

5 

Вывести количество файлов в директории:

$ ls | wc -l

19

Получить размер двоичного файла:

$ wc -c 2.jpg

213418 2.jpg

 

 

#### grep

#### zgrep

Поиск строки в архиве `xxx.tar.gz`:

```bash
zgrep -a <string> file.tar.gz
```



#### sort

#### uniq

#### sed

#### head

#### xargs

#### cut

#### Задачки

Найти и сгруппировать с подсчетом количества (COUNT GROUP BY) строки в логе, в которых на запрос к странице странице .html возвращалось 404.

Здесь 401095 взято от балды, узнать как найти номер строки и выбрать строки начиная с какой-то даты.

**tail** -n 401095 /var/log/nginx/petpop.cc/nginx_access.log | grep ' 404 ' | grep '.html HTTP' | awk '{print $7}' | sort | uniq -c | sort -rn

## Файловая система

Стандарт иерархии файловой системы (*Filesystem Hierarchy Standard*) определяет структуру каталогов в Linux-системах.

Все файлы и каталоги отображаются в корневом каталоге `/` , даже если они хранятся на разных физических или виртуальных устройствах (в отличии от *Windows*, где у каждого диска есть буква диска, обозначающая корень дерева его файловой системы).

Каталоги Unix не содержат *files*. Вместо этого они содержат *file names* в паре с *inode number*. 

### inode

inode (index node, индексный узел, а́йнод или ино́д), индексный дескриптор — это структура данных в *Unix-style file system*. В этой структуре хранится метаинформация о стандартных файлах, каталогах или других объектах файловой системы, кроме непосредственно данных и имени.

Каждый файл имеет свой индексный дескриптор, идентифицируемый по уникальному номеру (часто называемому 'i-номером' или 'инодом'), в файловой системе, в которой располагается сам файл.

Индексные дескрипторы хранят информацию о файлах, такую как принадлежность владельцу (пользователю и группе), режим доступа (чтение, запись, запуск на выполнение) и тип файла. Каталоги в Unix являются списками 'ссылочных' структур, каждая из которых содержит одно имя файла и один номер индексного дескриптора;



| Каталог                      | Описание                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| `/`                          | Корень *первичной иерархии*                                  |
| `/bin`                       | Папка для исполняемых  файлов, содержит ключевые утилиты, например, `ls`, `cp`, `cat` |
| `/etc`                       | Расшифровывается *et cetera* (и так далее), т.к. содержит файлы, который не могут быть помещены в какой-то другой каталог. Содержит конфигурационные файлы системных утилит и программ (например, `/etc/nginx`). Редактирование этих файлов влечёт за собой изменения для всех пользователей системы, поэтому изменять что-либо в данном каталоге может только *root*. |
| `/home` (`/Users` в *macOS*) | Users' home directories                                      |
| `/usr`                       | Корень *вторичной иерархии* for read-only user data. Расшифровывается как Unix System Resources (системные  ресурсы Unix). В этой папке хранятся данные всех установленных пакетов  программ, т.е. большинство приложений. Управление  папкой осуществляется пакетным менеджером, который ведет учет каждого файла в  этой папке. Приложения непосредственно в `/usr` рассматриваются как часть операционной системы (в отличии от `/usr/local`) |
| `/usr/bin`                   | исполняемые  файлы                                           |
| `/usr/sbin`                  | исполняемые  файлы, которые запускаются с правами администратора |
| `/usr/lib`                   | библиотеки для  исполняемых файлов                           |
| `/usr/local`                 | Корень *третичной иерархии* для локальных (*local*) данных. По стандарту `/usr`  должен быть общим для нескольких компьютеров и смонтирован по сети, а `/usr/local`  должен содержать установленные пакеты программы только на локальной машине. В  реальности, в `/usr/local`  размещают сторонние пакеты (*third party packages)* ~~, которые не устанавливаются пакетным  менеджером.~~ |
| `/usr/local/bin`             | исполняемые  файлы сторонних программ                        |
| `/usr/local/etc`             | их конфиги                                                   |
| `/usr/local/sbin`            | исполняемые  файлы посторонних программ, которые запускаются с правами администратора |
| `/var`                       | Переменные (*variable*) файлы - файлы, содержимое которых, как ожидается, будет постоянно изменяться во время нормальной работы системы, такие как *logs*, *databases*. |
| `/var/log`                   | Лог-файлы                                                    |

### /Proc

В этой папке каждому процессу системы соответствует папка с PID.

**/PROC/VERSION** Содержит версию ядра, компилятора, и в некоторых случаях, даже дистрибутива:

[pparshikov@cup fishki6.cup.ondu.ru]$ cat /proc/version

Linux version 2.6.32-573.7.1.el6.x86_64 (mockbuild@c6b8.bsys.dev.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC) ) #1 SMP Tue Sep 22 22:00:00 UTC 2015



## SSH

Генерация ключей выполняется командой

ssh-keygen -t rsa -b 4096

**-****t** тип ключа

**-****b** длина ключа в битах

В результате выполнения команды получаем в папке ~/.ssh файлы id_rsa.pub — открытый ключ, id_rsa — закрытый ключ.

Открытая часть ключа кладётся в домашний каталог пользователя, «которым» заходят на сервер (пользователь на сервере). Необходимо дописать в файл ~/.ssh/authorized_keys.

Закрытая — в домашний каталог пользователя, который идёт на удалённый сервер (пользователь на клиенте). В папку **~/.****ssh**, имя файла в зависимости от типа ключа для dsa - **id_dsa**, для RSA - **id_rsa**, для ECDSA - **id_ecdsa**. Ключ нельзя «украсть», взломав сервер, т.к. он на сервере не находится. 

Обязательно права на директорию **.ssh** - 700, права на файл с ключом - 600. Владелец и группа - сам пользователь. Если кто-то кроме пользователя имеет доступ к ключу- ключ считается небезопасным и не работает.

Подключение по ssh:

ssh [pparshikov@cup.ondu.ru](mailto:pparshikov@cup.ondu.ru)

Файл **~/.ssh/config** позволяет задать параметры подключения для пользователя, в том числе специальные для серверов, что самое важное, для каждого сервера своё.

Host *

 ForwardAgent yes      //пробросить ключ на удаленную машину

 ServerAliveInterval 300

 ServerAliveCountMax 120

 ControlMaster auto

 ControlPersist 60

 ControlPath /tmp/ssh-%r@%h:%p

 Compression yes

 PubkeyAcceptedKeyTypes=+ssh-dss

 User pparshikov

 

Host cup

Hostname cup.ondu.ru

 

Host dynamic1

Hostname 88.212.233.236

Можно задать настройки подключения для всех пользователей в /etc/ssh/ssh_config

Тогда можно будет входить по символьному имени:

ssh cup

**ssh-agent** –демон для кэширования дешифрованных ключей. Необходимо добавить ключ в кэш ssh-agent: 

\# ssh-add ~/.ssh/identity

Выполнить команду на удалённом сервере и тут же закрыть соединение:

ssh user@server ls /etc/

Чтение удаленного файла с передачей на обработку локальному приложению:

ssh user@server cat /some/file | awk '{print $2}' | local_app

Вывод удаленной команды в локальный файл:

ssh user@8.8.8.8 command > local_file

Проверка отпечатка

SSH клиент при подключении к хосту осуществляет проверку подлинности его ключа. Если SSH клиент не узнает отпечаток, он попросит вас подтвердить его добавление набрав «yes» или «no».

The authenticity of host ***** can't be established.

RSA key fingerprint is *****.

Are you sure you want to continue connecting (yes/no)?

Если вы ответите утвердительно, SSH клиент продолжит подключение, сохранив ключ сервера локально в файле ~/.ssh/known_hosts. Если впоследствии нужно изменить ключ, то нужно удалить этот файл.

## Утилиты

### Telnet, netcat

Подключение к серверу:

telnet localhost 80

nc localhost 80