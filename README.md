# otus-zfs

## Введение

Копируем Vagrantfile и файл setup_zfs.sh в директорию на компьютере.

## Запуск тестового окружения

Открываем консоль, перейдим в директорию с проектом и выполнить `vagrant up`
```shell
cd otus-zfs
vagrant up
```

После успешного завершения у вас будут подняты 2 виртуальных машины:
- `server` - 10.0.0.40
- `client` - 10.0.0.41

## Подключение к серверу

Для подключения к серверу необходимо выполнить
```shell
vagrant ssh server
```

# Задание 1

## Определим диски
```shell
echo disk{1..6} | xargs -n 1 fallocate -l 500M
```

## Создаем pool
```shell
zpool create storage raidz3 $PWD/disk[1-4]
```

Проверяем
```shell
zpool status
```

<details><summary>Пример вывода</summary>
<p>

```log
pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME             STATE     READ WRITE CKSUM
        storage          ONLINE       0     0     0
          raidz3-0       ONLINE       0     0     0
            /pool/disk1  ONLINE       0     0     0
            /pool/disk2  ONLINE       0     0     0
            /pool/disk3  ONLINE       0     0     0
            /pool/disk4  ONLINE       0     0     0

errors: No known data errors
```
</p>
</details>

## Создаем 4 файловые системы
```shell
zfs create storage/data1
zfs create storage/data2
zfs create storage/data3
zfs create storage/data4
```
Проверяем
```shell
zfs list
```
<details><summary>Пример вывода</summary>
<p>

```log
NAME            USED  AVAIL     REFER  MOUNTPOINT
storage         206K   352M       28K  /storage
storage/data1    24K   352M       24K  /storage/data1
storage/data2    24K   352M       24K  /storage/data2
storage/data3    24K   352M       24K  /storage/data3
storage/data4    24K   352M       24K  /storage/data4
```
</p>
</details>

```shell
mount -t zfs | column -t
```

<details><summary>Пример вывода</summary>
<p>

```log
storage        on  /storage        type  zfs  (rw,seclabel,xattr,noacl)
storage/data1  on  /storage/data1  type  zfs  (rw,seclabel,xattr,noacl)
storage/data2  on  /storage/data2  type  zfs  (rw,seclabel,xattr,noacl)
storage/data3  on  /storage/data3  type  zfs  (rw,seclabel,xattr,noacl)
storage/data4  on  /storage/data4  type  zfs  (rw,seclabel,xattr,noacl)
```
</p>
</details>

## Применяем алгоритмы шифрования на каждую файловую систему

## lzjb по умолчанию
```shell
zfs set compression=on storage/data1
```
## gzip - N
```shell
zfs set compression=gzip-9 storage/data2
```
## zle
```shell
zfs set compression=zle storage/data3
```
## lz4
```shell
zfs set compression=lz4 storage/data4
```

```shell
zfs get compression
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server pool]# zfs get compression
NAME           PROPERTY     VALUE     SOURCE
storage        compression  off       default
storage/data1  compression  on        local
storage/data2  compression  gzip-9    local
storage/data3  compression  zle       local
storage/data4  compression  lz4       local
```
</p>
</details>

## Скачиваем файл "Война и мир" и распологаем на файловой системе

```shell
 wget -O War_and_Peace.txt https://all-the-books.ru/download_book/00ae0503fb3889d7e9faf30400aefde3/
```

## Копирум в каждую файловую систему и проверяем, какой алгоритм лучше
```shell
zfs get compression,compressratio
```

```shell
du -h
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server storage]# zfs get compression,compressratio
NAME           PROPERTY       VALUE     SOURCE
storage        compression    off       default
storage        compressratio  1.39x     -
storage/data1  compression    on        local
storage/data1  compressratio  1.60x     -
storage/data2  compression    gzip-9    local
storage/data2  compressratio  2.51x     -
storage/data3  compression    zle       local
storage/data3  compressratio  1.06x     -
storage/data4  compression    lz4       local
storage/data4  compressratio  1.60x     -

[root@server storage]# du -h
476K    ./data4
724K    ./data3
301K    ./data2
476K    ./data1
2.7M    .
```
</p>
</details>

## Вывод: gzip-9 лучше


# Задание 2

## Скачиваем архив
```shell
wget --no-check-certificate -O zfs_task1.tar.gz 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download'
```

```shell
tar -zxvf zfs_task1.tar.gz
```

```shell
tar -zxvf zfs_task1.tar.gz
```

## Импорт пула

```shell
zpool import -d $PWD/zpoolexport/ otus
```

```shell
zpool status
```
<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                    STATE     READ WRITE CKSUM
        otus                    ONLINE       0     0     0
          mirror-0              ONLINE       0     0     0
            /zpoolexport/filea  ONLINE       0     0     0
            /zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```
</p>
</details>

## Размер 500M

```shell
df -h
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         96M     0   96M   0% /dev
tmpfs           112M     0  112M   0% /dev/shm
tmpfs           112M  7.4M  105M   7% /run
tmpfs           112M     0  112M   0% /sys/fs/cgroup
/dev/sda1        10G  7.2G  2.8G  72% /
tmpfs            23M     0   23M   0% /run/user/1000
otus            350M  128K  350M   1% /otus
otus/hometask2  352M  2.0M  350M   1% /otus/hometask2
```
</p>
</details>

```shell
zpool list
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zpool list
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus   480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
```
</p>
</details>

## Тип mirror-0

```shell
zpool status
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                    STATE     READ WRITE CKSUM
        otus                    ONLINE       0     0     0
          mirror-0              ONLINE       0     0     0
            /zpoolexport/filea  ONLINE       0     0     0
            /zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```
</p>
</details>

## Значение recordsize 128K

```shell
zfs get recordsize
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zfs get recordsize
NAME            PROPERTY    VALUE    SOURCE
otus            recordsize  128K     local
otus/hometask2  recordsize  128K     inherited from otus
```
</p>
</details>

## Сжатие zle

```shell
zfs get compression,compressratio
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zfs get compression,compressratio
NAME            PROPERTY       VALUE     SOURCE
otus            compression    zle       local
otus            compressratio  1.00x     -
otus/hometask2  compression    zle       inherited from otus
otus/hometask2  compressratio  1.00x     -
```
</p>
</details>

## Контрольная сумма sha256

```shell
zfs get checksum
```
<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zfs get checksum
NAME            PROPERTY  VALUE      SOURCE
otus            checksum  sha256     local
otus/hometask2  checksum  sha256     inherited from otus
```
</p>
</details>

## Файл настроек

```shell
zfs get all
```
<details><summary>Пример вывода</summary>
<p>

```log
[root@server /]# zfs get all
NAME            PROPERTY              VALUE                  SOURCE
otus            type                  filesystem             -
otus            creation              Fri May 15  4:00 2020  -
otus            used                  2.04M                  -
otus            available             350M                   -
otus            referenced            24K                    -
otus            compressratio         1.00x                  -
otus            mounted               yes                    -
otus            quota                 none                   default
otus            reservation           none                   default
otus            recordsize            128K                   local
otus            mountpoint            /otus                  default
otus            sharenfs              off                    default
otus            checksum              sha256                 local
otus            compression           zle                    local
otus            atime                 on                     default
otus            devices               on                     default
otus            exec                  on                     default
otus            setuid                on                     default
otus            readonly              off                    default
otus            zoned                 off                    default
otus            snapdir               hidden                 default
otus            aclinherit            restricted             default
otus            createtxg             1                      -
otus            canmount              on                     default
otus            xattr                 on                     default
otus            copies                1                      default
otus            version               5                      -
otus            utf8only              off                    -
otus            normalization         none                   -
otus            casesensitivity       sensitive              -
otus            vscan                 off                    default
otus            nbmand                off                    default
otus            sharesmb              off                    default
otus            refquota              none                   default
otus            refreservation        none                   default
otus            guid                  14592242904030363272   -
otus            primarycache          all                    default
otus            secondarycache        all                    default
otus            usedbysnapshots       0B                     -
otus            usedbydataset         24K                    -
otus            usedbychildren        2.01M                  -
otus            usedbyrefreservation  0B                     -
otus            logbias               latency                default
otus            objsetid              54                     -
otus            dedup                 off                    default
otus            mlslabel              none                   default
otus            sync                  standard               default
otus            dnodesize             legacy                 default
otus            refcompressratio      1.00x                  -
otus            written               24K                    -
otus            logicalused           1020K                  -
otus            logicalreferenced     12K                    -
otus            volmode               default                default
otus            filesystem_limit      none                   default
otus            snapshot_limit        none                   default
otus            filesystem_count      none                   default
otus            snapshot_count        none                   default
otus            snapdev               hidden                 default
otus            acltype               off                    default
otus            context               none                   default
otus            fscontext             none                   default
otus            defcontext            none                   default
otus            rootcontext           none                   default
otus            relatime              off                    default
otus            redundant_metadata    all                    default
otus            overlay               off                    default
otus            encryption            off                    default
otus            keylocation           none                   default
otus            keyformat             none                   default
otus            pbkdf2iters           0                      default
otus            special_small_blocks  0                      default
otus/hometask2  type                  filesystem             -
otus/hometask2  creation              Fri May 15  4:18 2020  -
otus/hometask2  used                  1.88M                  -
otus/hometask2  available             350M                   -
otus/hometask2  referenced            1.88M                  -
otus/hometask2  compressratio         1.00x                  -
otus/hometask2  mounted               yes                    -
otus/hometask2  quota                 none                   default
otus/hometask2  reservation           none                   default
otus/hometask2  recordsize            128K                   inherited from otus
otus/hometask2  mountpoint            /otus/hometask2        default
otus/hometask2  sharenfs              off                    default
otus/hometask2  checksum              sha256                 inherited from otus
otus/hometask2  compression           zle                    inherited from otus
otus/hometask2  atime                 on                     default
otus/hometask2  devices               on                     default
otus/hometask2  exec                  on                     default
otus/hometask2  setuid                on                     default
otus/hometask2  readonly              off                    default
otus/hometask2  zoned                 off                    default
otus/hometask2  snapdir               hidden                 default
otus/hometask2  aclinherit            restricted             default
otus/hometask2  createtxg             216                    -
otus/hometask2  canmount              on                     default
otus/hometask2  xattr                 on                     default
otus/hometask2  copies                1                      default
otus/hometask2  version               5                      -
otus/hometask2  utf8only              off                    -
otus/hometask2  normalization         none                   -
otus/hometask2  casesensitivity       sensitive              -
otus/hometask2  vscan                 off                    default
otus/hometask2  nbmand                off                    default
otus/hometask2  sharesmb              off                    default
otus/hometask2  refquota              none                   default
otus/hometask2  refreservation        none                   default
otus/hometask2  guid                  3809416093691379248    -
otus/hometask2  primarycache          all                    default
otus/hometask2  secondarycache        all                    default
otus/hometask2  usedbysnapshots       0B                     -
otus/hometask2  usedbydataset         1.88M                  -
otus/hometask2  usedbychildren        0B                     -
otus/hometask2  usedbyrefreservation  0B                     -
otus/hometask2  logbias               latency                default
otus/hometask2  objsetid              81                     -
otus/hometask2  dedup                 off                    default
otus/hometask2  mlslabel              none                   default
otus/hometask2  sync                  standard               default
otus/hometask2  dnodesize             legacy                 default
otus/hometask2  refcompressratio      1.00x                  -
otus/hometask2  written               1.88M                  -
otus/hometask2  logicalused           963K                   -
otus/hometask2  logicalreferenced     963K                   -
otus/hometask2  volmode               default                default
otus/hometask2  filesystem_limit      none                   default
otus/hometask2  snapshot_limit        none                   default
otus/hometask2  filesystem_count      none                   default
otus/hometask2  snapshot_count        none                   default
otus/hometask2  snapdev               hidden                 default
otus/hometask2  acltype               off                    default
otus/hometask2  context               none                   default
otus/hometask2  fscontext             none                   default
otus/hometask2  defcontext            none                   default
otus/hometask2  rootcontext           none                   default
otus/hometask2  relatime              off                    default
otus/hometask2  redundant_metadata    all                    default
otus/hometask2  overlay               off                    default
otus/hometask2  encryption            off                    default
otus/hometask2  keylocation           none                   default
otus/hometask2  keyformat             none                   default
otus/hometask2  pbkdf2iters           0                      default
otus/hometask2  special_small_blocks  0                      default
```
</p>
</details>

# Задание 3

## Скачиваем файл

```shell
wget --no-check-certificate -O otus_task2.file 'https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download'
```

## Заполняем данными
```shell
zfs create otus/storage
```

```shell
cp otus_task2.file otus
```

## Восстанавливаем

```shell
 zfs receive otus/storage < otus_task2.file
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server storage]# ls -l
total 2589
-rw-r--r--. 1 root    root          0 May 15  2020 10M.file
-rw-r--r--. 1 root    root     727040 May 15  2020 cinderella.tar
-rw-r--r--. 1 root    root         65 May 15  2020 for_examaple.txt
-rw-r--r--. 1 root    root          0 May 15  2020 homework4.txt
-rw-r--r--. 1 root    root     309987 May 15  2020 Limbo.txt
-rw-r--r--. 1 root    root     509836 May 15  2020 Moby_Dick.txt
drwxr-xr-x. 3 vagrant vagrant       4 Dec 18  2017 task1
-rw-r--r--. 1 root    root    1209374 May  6  2016 War_and_Peace.txt
-rw-r--r--. 1 root    root     398635 May 15  2020 world.sql
```
</p>
</details>

## Поиск сообщения

```shell
find . -name secret_message
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server storage]# find . -name secret_message
./task1/file_mess/secret_message
```
</p>
</details>

```shell
cat ./task1/file_mess/secret_message
```

<details><summary>Пример вывода</summary>
<p>

```log
[root@server storage]# cat ./task1/file_mess/secret_message
https://github.com/sindresorhus/awesome
```
</p>
</details>


