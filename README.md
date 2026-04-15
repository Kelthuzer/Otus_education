## С чего начинается Linux
- Запустить ВМ c Ubuntu.<br> 
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/c04b0699-ba6d-4192-a1a6-2b77d57fe74c" /><br>
- Проверка устеновленного образа (версия и разрядность системы)<br>
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/7815d981-c531-4bf1-aed1-da93c90151a1" /><br>
- Загрузка павкетов ( я решил обновить до последний возможной версии в ракмках одного релиза глобальной версии именно до 6.19 AMD64)<br>

 ```bash
wget https://kernel.ubuntu.com/mainline/v6.19/amd64/linux-headers-6.19.0-061900-generic_6.19.0-061900.202602082231_amd64.deb
wget https://kernel.ubuntu.com/mainline/v6.19/amd64/linux-headers-6.19.0-061900_6.19.0-061900.202602082231_all.deb
wget https://kernel.ubuntu.com/mainline/v6.19/amd64/linux-image-unsigned-6.19.0-061900-generic_6.19.0-061900.202602082231_amd64.deb
wget https://kernel.ubuntu.com/mainline/v6.19/amd64/linux-modules-6.19.0-061900-generic_6.19.0-061900.202602082231_amd64.deb  
```
<br>
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/ba6cec8e-1572-457d-acdb-254c67db8802" /><br>
- установка пакетов из текущей директории <br>

 ```bash
 sudo dpkg -i *.deb
 ```
 <br><img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/85d87405-4ea8-4fd6-a7b7-a45ee020d588" /><br>
- проверка
```bash
 ls -al /boot
```
Получаем вывод 
```
-rw-r--r--  1 root root   306589 Feb  8 22:31 config-6.19.0-061900-generic
-rw-r--r--  1 root root   287605 Mar 19 11:44 config-6.8.0-110-generic
drwxr-xr-x  5 root root     4096 Apr 15 12:42 grub
lrwxrwxrwx  1 root root       32 Apr 15 12:42 initrd.img -> initrd.img-6.19.0-061900-generic
-rw-r--r--  1 root root 80800394 Apr 15 12:42 initrd.img-6.19.0-061900-generic
-rw-r--r--  1 root root 32872388 Apr 15 10:27 initrd.img-6.8.0-100-generic
-rw-r--r--  1 root root 77183239 Apr 15 10:33 initrd.img-6.8.0-110-generic
lrwxrwxrwx  1 root root       28 Apr 15 10:24 initrd.img.old -> initrd.img-6.8.0-110-generic
-rw-------  1 root root 11765120 Feb  8 22:31 System.map-6.19.0-061900-generic
-rw-------  1 root root  9126224 Mar 19 11:44 System.map-6.8.0-110-generic
lrwxrwxrwx  1 root root       29 Apr 15 12:42 vmlinuz -> vmlinuz-6.19.0-061900-generic
-rw-------  1 root root 17469632 Feb  8 22:31 vmlinuz-6.19.0-061900-generic
-rw-------  1 root root 15042952 Mar 19 13:43 vmlinuz-6.8.0-110-generic
lrwxrwxrwx  1 root root       25 Apr 15 10:24 vmlinuz.old -> vmlinuz-6.8.0-110-generic
```
- обновляем граб/груб/grub (по факту список уже обновлен на моменте установки новых пакетов)
```bash
sudo update-grub
```
```
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.19.0-061900-generic
Found initrd image: /boot/initrd.img-6.19.0-061900-generic
Found linux image: /boot/vmlinuz-6.8.0-110-generic
Found initrd image: /boot/initrd.img-6.8.0-110-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```
- Из получено списка выше мы имеем 4 пункта нам нужен первый но т.к отсчет начинается с 0 выставлев загрузку по умолнчанию в 0 т.е. 
``` 
sudo grub-set-default 0
``` 
перезагзука системы 
```
sudo shutdown -r now
```
првоерка результата <br>
<img width="620" height="453" alt="image" src="https://github.com/user-attachments/assets/94114d2a-a180-4421-84f1-8d92fd1e8cb4" />

