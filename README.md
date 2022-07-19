# ohw05-zfs
Первым делом поднимаем ВМ
https://github.com/nussnk/ohw05-zfs
В Vagrantfile прописано создание ВМ с 8 дисквами для работы с zfs
А также в провижинге расписана установка и подключение модуля ZFS

заходим через vagrant ssh 
сразу делаем sudo -i

================================= 1. Определить алгоритм с наилучшим сжатием. =================================
Смотрим какие диски есть

[root@zfs ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk
sdc      8:32   0  512M  0 disk
sdd      8:48   0  512M  0 disk
sde      8:64   0  512M  0 disk
sdf      8:80   0  512M  0 disk
sdg      8:96   0  512M  0 disk
sdh      8:112  0  512M  0 disk
sdi      8:128  0  512M  0 disk

создаем 4 пула
[root@zfs ~]# zpool create otus1 mirror /dev/sdb /dev/sdc
[root@zfs ~]# zpool create otus2 mirror /dev/sdd /dev/sde
[root@zfs ~]# zpool create otus3 mirror /dev/sdf /dev/sdg
[root@zfs ~]# zpool create otus4 mirror /dev/sdh /dev/sdi

смотрим что получилось
[root@zfs ~]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
[root@zfs ~]# zpool status
  pool: otus1
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus1       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus2       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdd     ONLINE       0     0     0
            sde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus3       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdf     ONLINE       0     0     0
            sdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus4       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdh     ONLINE       0     0     0
            sdi     ONLINE       0     0     0

errors: No known data errors

смотрим текущие параметры компрессии
[root@zfs ~]# zfs get all | grep compression
otus1  compression           off                    default
otus2  compression           off                    default
otus3  compression           off                    default
otus4  compression           off                    default
видим. что никакой копрессии нет

задаем 4 разных варианта для каждого пула
[root@zfs ~]# zfs set compression=lzjb otus1
[root@zfs ~]# zfs set compression=lz4 otus2
[root@zfs ~]# zfs set compression=gzip-9 otus3
[root@zfs ~]# zfs set compression=zle otus4

проверяем
[root@zfs ~]# zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local

закачиваем файл в каждый из пулов
[root@zfs ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2022-07-10 20:30:28--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40841120 (39M) [text/plain]
Saving to: ‘/otus1/pg2600.converter.log’

100%[=====================================================================================================================================================================>] 40,841,120  6.19MB/s   in 6.9s

2022-07-10 20:30:36 (5.67 MB/s) - ‘/otus1/pg2600.converter.log’ saved [40841120/40841120]

--2022-07-10 20:30:36--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40841120 (39M) [text/plain]
Saving to: ‘/otus2/pg2600.converter.log’

100%[=====================================================================================================================================================================>] 40,841,120  8.32MB/s   in 5.7s

2022-07-10 20:30:42 (6.82 MB/s) - ‘/otus2/pg2600.converter.log’ saved [40841120/40841120]

--2022-07-10 20:30:42--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40841120 (39M) [text/plain]
Saving to: ‘/otus3/pg2600.converter.log’

100%[=====================================================================================================================================================================>] 40,841,120  52.7KB/s   in 14m 16s

2022-07-10 20:44:59 (46.6 KB/s) - ‘/otus3/pg2600.converter.log’ saved [40841120/40841120]

--2022-07-10 20:44:59--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40841120 (39M) [text/plain]
Saving to: ‘/otus4/pg2600.converter.log’

100%[=====================================================================================================================================================================>] 40,841,120  60.5KB/s   in 13m 46s

2022-07-10 20:58:45 (48.3 KB/s) - ‘/otus4/pg2600.converter.log’ saved [40841120/40841120]

проверим, что все закачалось как надо и сравниваем размер ужатых файлов в строках total
[root@zfs ~]# ls -l /otus*
/otus:
total 0

/otus1:
total 22025
-rw-r--r--. 1 root root 40841120 Jul  2 08:39 pg2600.converter.log

/otus2:
total 17975
-rw-r--r--. 1 root root 40841120 Jul  2 08:39 pg2600.converter.log

/otus3:
total 10952
-rw-r--r--. 1 root root 40841120 Jul  2 08:39 pg2600.converter.log

/otus4:
total 39914
-rw-r--r--. 1 root root 40841120 Jul  2 08:39 pg2600.converter.log
видим, что лучшее сжатие показывает третий пул (gzip-9)

перепроверяем через вывод информации о каждом пуле
[root@zfs ~]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.6M   330M     21.5M  /otus1
otus2  17.6M   334M     17.6M  /otus2
otus3  11.0M   341M     10.7M  /otus3
otus4  39.3M   313M     39.0M  /otus4
[root@zfs ~]# zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.81x                  -
otus2  compressratio         2.22x                  -
otus3  compressratio         3.62x                  -
otus4  compressratio         1.00x                  -

================================= 2. Определяем настройки pool-а =================================

закачиваем файл 
[root@zfs ~]# wget -O archive.tar.gz --no-check-certificate 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download'
--2022-07-19 18:40:04--  https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download
Resolving drive.google.com (drive.google.com)... 142.251.37.206, 2a00:1450:4006:812::200e
Connecting to drive.google.com (drive.google.com)|142.251.37.206|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download [following]
--2022-07-19 18:40:05--  https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download
Reusing existing connection to drive.google.com:443.
HTTP request sent, awaiting response... 303 See Other
Location: https://doc-0c-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/8raiippljvvmkk5vqav4vm5i4debdgtu/1658256000000/16189157874053420687/*/1KRBNW33QWqbvbVHa3hLJivOAt60yukkg?e=download&uuid=2163f858-95c6-40c6-af00-f098bef5bcf4 [following]
Warning: wildcards not supported in HTTP.
--2022-07-19 18:40:10--  https://doc-0c-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/8raiippljvvmkk5vqav4vm5i4debdgtu/1658256000000/16189157874053420687/*/1KRBNW33QWqbvbVHa3hLJivOAt60yukkg?e=download&uuid=2163f858-95c6-40c6-af00-f098bef5bcf4
Resolving doc-0c-bo-docs.googleusercontent.com (doc-0c-bo-docs.googleusercontent.com)... 172.217.18.33, 2a00:1450:4006:801::2001
Connecting to doc-0c-bo-docs.googleusercontent.com (doc-0c-bo-docs.googleusercontent.com)|172.217.18.33|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6.9M) [application/x-gzip]
Saving to: ‘archive.tar.gz’

100%[=====================================================================================================================================================================>] 7,275,140   9.44MB/s   in 0.7s

2022-07-19 18:40:12 (9.44 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]


распаковываем
[root@zfs ~]# tar xvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb

проверяем возможность экспорта
[root@zfs ~]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE


импортируем
[root@zfs ~]# zpool import -d zpoolexport/ otus
[root@zfs ~]# ls /otus
hometask2

смотрим информацию о пуле
[root@zfs ~]# zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupditto                     0                              default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.11M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      1611752275898483751            -
otus  autotrim                       off                            default
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@bookmark_v2            enabled                        local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local

смотрим информацию о файловой системе
[root@zfs ~]# zfs get all otus
NAME  PROPERTY              VALUE                  SOURCE
otus  type                  filesystem             -
otus  creation              Fri May 15  4:00 2020  -
otus  used                  2.04M                  -
otus  available             350M                   -
otus  referenced            24K                    -
otus  compressratio         1.00x                  -
otus  mounted               yes                    -
otus  quota                 none                   default
otus  reservation           none                   default
otus  recordsize            128K                   local
otus  mountpoint            /otus                  default
otus  sharenfs              off                    default
otus  checksum              sha256                 local
otus  compression           zle                    local
otus  atime                 on                     default
otus  devices               on                     default
otus  exec                  on                     default
otus  setuid                on                     default
otus  readonly              off                    default
otus  zoned                 off                    default
otus  snapdir               hidden                 default
otus  aclinherit            restricted             default
otus  createtxg             1                      -
otus  canmount              on                     default
otus  xattr                 on                     default
otus  copies                1                      default
otus  version               5                      -
otus  utf8only              off                    -
otus  normalization         none                   -
otus  casesensitivity       sensitive              -
otus  vscan                 off                    default
otus  nbmand                off                    default
otus  sharesmb              off                    default
otus  refquota              none                   default
otus  refreservation        none                   default
otus  guid                  14592242904030363272   -
otus  primarycache          all                    default
otus  secondarycache        all                    default
otus  usedbysnapshots       0B                     -
otus  usedbydataset         24K                    -
otus  usedbychildren        2.01M                  -
otus  usedbyrefreservation  0B                     -
otus  logbias               latency                default
otus  objsetid              54                     -
otus  dedup                 off                    default
otus  mlslabel              none                   default
otus  sync                  standard               default
otus  dnodesize             legacy                 default
otus  refcompressratio      1.00x                  -
otus  written               24K                    -
otus  logicalused           1020K                  -
otus  logicalreferenced     12K                    -
otus  volmode               default                default
otus  filesystem_limit      none                   default
otus  snapshot_limit        none                   default
otus  filesystem_count      none                   default
otus  snapshot_count        none                   default
otus  snapdev               hidden                 default
otus  acltype               off                    default
otus  context               none                   default
otus  fscontext             none                   default
otus  defcontext            none                   default
otus  rootcontext           none                   default
otus  relatime              off                    default
otus  redundant_metadata    all                    default
otus  overlay               off                    default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default

теперь выведем определенные параметры

размер хранилища;
пул у нас на 480МБ
[root@zfs ~]# zpool get size otus
NAME  PROPERTY  VALUE  SOURCE
otus  size      480M   -
А ФС на 350 МБ
[root@zfs ~]# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -

тип pool; (Вот тут не совсем понимаю вопрос. Проверяем что есть возможность записи?)
[root@zfs ~]# zpool get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     -
[root@zfs ~]# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default

значение recordsize
[root@zfs ~]# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local

какое сжатие используется;
[root@zfs ~]# zfs get compression otus
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local

какая контрольная сумма используется
[root@zfs ~]# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local


================================= 3. Работа со снапшотом, поиск сообщения от преподавателя =================================

качаем файл
[root@zfs ~]# wget -O otus_task2.file --no-check-certificate 'https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download'
--2022-07-19 19:00:16--  https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download
Resolving drive.google.com (drive.google.com)... 142.251.37.206, 2a00:1450:4006:812::200e
Connecting to drive.google.com (drive.google.com)|142.251.37.206|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://drive.google.com/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download [following]
--2022-07-19 19:00:16--  https://drive.google.com/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download
Reusing existing connection to drive.google.com:443.
HTTP request sent, awaiting response... 303 See Other
Location: https://doc-00-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/48ga18v8tq4is8vodbiode11cg48tptd/1658257200000/16189157874053420687/*/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG?e=download&uuid=4a2ccc4d-0602-40df-a9bf-b21dd0c178dd [following]
Warning: wildcards not supported in HTTP.
--2022-07-19 19:00:21--  https://doc-00-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/48ga18v8tq4is8vodbiode11cg48tptd/1658257200000/16189157874053420687/*/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG?e=download&uuid=4a2ccc4d-0602-40df-a9bf-b21dd0c178dd
Resolving doc-00-bo-docs.googleusercontent.com (doc-00-bo-docs.googleusercontent.com)... 172.217.18.33, 2a00:1450:4006:801::2001
Connecting to doc-00-bo-docs.googleusercontent.com (doc-00-bo-docs.googleusercontent.com)|172.217.18.33|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5432736 (5.2M) [application/octet-stream]
Saving to: ‘otus_task2.file’

100%[=====================================================================================================================================================================>] 5,432,736   10.6MB/s   in 0.5s

2022-07-19 19:00:22 (10.6 MB/s) - ‘otus_task2.file’ saved [5432736/5432736]

восстанавливаем снапшот
[root@zfs ~]# zfs receive otus/test < otus_task2.file
[root@zfs ~]# ls /otus/test/
10M.file  cinderella.tar  for_examaple.txt  homework4.txt  Limbo.txt  Moby_Dick.txt  task1  War_and_Peace.txt  world.sql

ищем секретный файл
[root@zfs ~]# find /otus/test/ -name secret*
/otus/test/task1/file_mess/secret_message

читаем
[root@zfs ~]# cat /otus/test/task1/file_mess/secret_message
https://github.com/sindresorhus/awesome

по ссылке попадаем в целевой репозиторий
