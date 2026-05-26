# ZFS
## 1) Определение алгоритма с наилучшим сжатием
- Создаем виртуальную машину. Смотрим список всех дисков, которые есть в виртуальной машине `lsblk` <br>
  <img width="373" height="221" alt="image" src="https://github.com/user-attachments/assets/d18b9dc0-03a3-4aeb-94cc-2de686fa0fba" />
- Установим пакет утилит для ZF `apt install zfsutils-linux` <br>
  <img width="1096" height="242" alt="image" src="https://github.com/user-attachments/assets/a56c145d-6fc2-485a-9ac2-bec4db8937b9" />
- Создаём пулы из двух дисков в режиме RAID 1 `zpool create %poolname% mirror %dev1% %dev2%`<br>
  <img width="603" height="66" alt="image" src="https://github.com/user-attachments/assets/2df35738-ff52-482b-8359-eb3a60a10b3f" />
- смотрим список пулов `zpool list`<br>
  <img width="692" height="96" alt="image" src="https://github.com/user-attachments/assets/0df46825-cfc0-411a-9afd-df6e77de5f9c" />
- Добавим разные алгоритмы сжатия в каждую файловую систему
  1)  Алгоритм lzjb: `zfs set compression=lzjb zfs1`
  2)  Алгоритм lz4:  `zfs set compression=lz4 zfs2`
  3)  Алгоритм gzip: `zfs set compression=gzip-9 zfs3`
  4)  Алгоритм zle:  `zfs set compression=zle zfs4`
- Проверим, что все файловые системы имеют сжатияе `zfs get all | grep compression` <br>
  <img width="497" height="86" alt="image" src="https://github.com/user-attachments/assets/8c11c707-da91-459a-bba8-7e633d28a8a1" />
- Скачаем один и тот же текстовый файл во все пулы
  ``` bash
  for i in {1..4}; do wget -P /zfs$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
  ```
- проверим что файл скачан в каждый пул <br>
  <img width="639" height="367" alt="image" src="https://github.com/user-attachments/assets/c444d01d-0146-4dc2-8efc-5dc71d392dd4" />
- проверяем эфективность сжатия `zfs get all | grep compressratio | grep -v ref` <br>
  <img width="369" height="102" alt="image" src="https://github.com/user-attachments/assets/fbc39263-1d10-4c38-bdd4-9b985df9f401" />
  <img width="452" height="70" alt="image" src="https://github.com/user-attachments/assets/a5ce55ac-ef29-4af9-8ebf-9290658cdb94" /> <br>
## 2) Определение настроек пула
- качаем и распоковываем архив <br>
  <img width="461" height="76" alt="image" src="https://github.com/user-attachments/assets/52772535-bb00-477f-857d-f7e0635191ca" />
- проверим импорт `zpool import -d zpoolexport/`<br>
  <img width="642" height="227" alt="image" src="https://github.com/user-attachments/assets/d42a110b-cf9b-47ec-a5b9-f52c19752f75" />
- импорт и проверка `zpool import -d zpoolexport/ otus` <br>
 <img width="634" height="330" alt="image" src="https://github.com/user-attachments/assets/ed5c6d8d-9871-42fa-8447-2c42bb3bf440" />
- Запрос параметров ФС `zfs get all otus` <br>
  <details>
  <summary>Вывод</summary>
```bash
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
otus  aclmode               discard                default
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
otus  relatime              on                     default
otus  redundant_metadata    all                    default
otus  overlay               on                     default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default
```
  </details>

- C помощью команды get можно уточнить конкретный параметр, например
  * `zfs get available otus`
  * `zfs get readonly otus`
  * `zfs get recordsize otus`
  * `zfs get compression otus`
  * `zfs get checksum otus`
  * <br>
    <img width="448" height="258" alt="image" src="https://github.com/user-attachments/assets/9b13c794-5b93-455c-9547-71123d4c7cdf" />

  ## 3) Работа со снапшотом, поиск сообщения от преподавателя<br>
- Восстановим файловую систему из снапшота `zfs receive otus/test@today < otus_task2.file`
- Ищем послание от преподователя `find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message`<br>
<img width="621" height="148" alt="image" src="https://github.com/user-attachments/assets/067a3ec8-4692-444d-a4ad-934217058967" />

