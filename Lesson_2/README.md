# Задание
## Добавить в виртуальную машину несколько дисков
- Создаю виртуальные диски
<details>
<summary>Скриншоты VB</summary>
<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/a7182f3f-f4bf-4f24-9f27-ae4f71243193" /> <br>
<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/8e7fac7a-800e-4555-845e-dde6b16e8549" /> <br>
</details> 
- Првоеряю что диски корректно отображаються в системе `fdisk -l`
<details>
<summary>Вывод </summary>

```bash
Disk /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 19CE4A44-2ECD-4805-B783-FF6CACA28F71

Device     Start      End  Sectors Size Type
/dev/sda1   2048     4095     2048   1M BIOS boot
/dev/sda2   4096 52426751 52422656  25G Linux filesystem


Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sde: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

```
</details> 
<br>
- по методичке 
> Занулим на всякий случай суперблоки
- выполняю команду ,названия устройств у меня совпадают кроме f, я собираютсь делать рейд 10 поэтому диска 4. 
<details>
<summary>mdadm --zero-superblock --force /dev/sd{b,c,d,e}</summary>
  
```
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
```
</details> <br>
- ошибка потому что они не размечены (нет ни MBR ни GPT разметки) на "всякий" случай затру диски ` wipefs -a /dev/sd{b,c,d,e} `. Вывод каомнды пустой
- создавать рейд следующей командой 

```
mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f} 
```
Отредактирую команду под себя 

```
mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}
```
и проверим что у получаеться командой `mdadm -D /dev/md0`
<details>
<summary>получаем при проверке командой</summary>
<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/fb49e06d-5c0a-4b66-b8db-0ddc7349c2f3" />
</details> 

## Сломать и починить RAID 
- принудительно помечаю диск как сломанный `mdadm /dev/md0 --fail /dev/sdc`

```
mdadm: set /dev/sdc faulty in /dev/md0
```
- првоеряю как это видит система ` mdadm -D /dev/md0 `
  
```
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       -       0        0        1      removed
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

       1       8       32        -      faulty   /dev/sdc
```
- демонтируем сломанный диск `mdadm /dev/md0 --remove /dev/sdc`
  
  ```
  hot removed /dev/sdc from /dev/md0
  ```
- "установка нового вместо сломанного" командой `mdadm /dev/md0 --add /dev/sdс`
<img width="529" height="31" alt="image" src="https://github.com/user-attachments/assets/24e5f192-14de-4a6b-9642-5bae05b42ebe" /><br>
размер дисков мальникй поэтому прогресс командой `cat /proc/mdstat` отследить не получилось результат выглядит так:<br>
<img width="599" height="97" alt="image" src="https://github.com/user-attachments/assets/eb908efb-e8c5-44ef-ad66-d1a36814b19d" /><br>
<img width="553" height="88" alt="image" src="https://github.com/user-attachments/assets/9fb2b9f2-da80-4cd5-ac43-673d495064d9" />

## Создать GPT таблицу, пять разделов и смонтировать их в системе
- создаем GPT
  
```
parted -s /dev/md0 mklabel gpt
```
вывода у команды нет но можено проверить через `fdisk -f` что получилось:
<img width="460" height="104" alt="image" src="https://github.com/user-attachments/assets/0ff14c72-5e15-48aa-b8ca-4c8ab6b98d9c" /><br>
я разобью рейд на 4 части для практики<br>
<img width="650" height="183" alt="image" src="https://github.com/user-attachments/assets/4c1baaaa-ddd1-446b-b8cb-a486d9c4a550" /><br>
- далее нужно создать файловые системы на созданых нами разделах делаем простым скриптом
  
```
for i in $(seq 1 4); do sudo mkfs.ext4 /dev/md0p$i; done
```
где i выступапает в роли переменной и частью имени раздела.
<details>
<summary>Получаем</summary>
<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/bafed308-8940-4f14-93b6-fc9b3fed6119" />
</details> 
- создаем каталоги и монтируем новые диски

``` 
mkdir -p /mnt/raid/part{1,2,3,4}
```
затем подключаем каждый диск к свему каталогу скнова воспользуемся предложиним скриптом 
```
for i in $(seq 1 4); do mount /dev/md0p$i /mnt/raid/part$i; done
```
посмотрим что получилось ` ls /mnt/raid/ `
<img width="394" height="131" alt="image" src="https://github.com/user-attachments/assets/8b5bc7d3-1e61-43ff-83b2-6274fd167115" /><br>
на этом задания второго урока выполены
