#  LVM
## Уменьшить том под / до 8G
- подготовка тома
  
  ```bash
  pvcreate /dev/sdb /
  vgcreate vg_root /dev/sdb /
  lvcreate -n lv_root -l +100%FREE /dev/vg_r
  ```
  получаем
  <img width="800" height="134" alt="image" src="https://github.com/user-attachments/assets/5af7fad4-3d5e-4217-89ae-6f3fdfdd46d0" />
- создаем ФС
  ```bash
  mkfs.ext4 /dev/vg_root/lv_root
  ```
<img width="800" height="194" alt="image" src="https://github.com/user-attachments/assets/06e22568-2a0e-4dff-8ca7-9eea00df4a26" />
- монтируем `mount /dev/vg_root/lv_root /mnt`
- копируем `rsync -avxHAX --progress / /mnt/`, ждем завершения. В итоге получим
  ```
  sent 4,125,280,042 bytes  received 2,853,364 bytes  81,745,215.96 bytes/sec
  total size is 4,118,307,974  speedup is 1.00
  ```
- перепроверяем на всякий случай 
  <img width="800" height="470" alt="image" src="https://github.com/user-attachments/assets/69c7334e-94e3-4c4f-bf48-b2a9ee6edc32" />
- подготавливаем окружение 
```bash
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
```
- меняем корень `chroot /mnt/`
- настройка граб `grub-mkconfig -o /boot/grub/grub.cfg`
  <img width="800" height="249" alt="image" src="https://github.com/user-attachments/assets/6a3a2db0-53c3-494f-aca1-7d8f885567ad" />
- Обновим образ initrd `update-initramfs -u`
`update-initramfs: Generating /boot/initrd.img-6.19.0-061900-generic`
- Перезагружаем систему `exit` `shutdown -r now`. Получаем ошибку `Running in chroot, ignoring request.` выходим из chroot и снова пробуем перезагруку получаем 
  <img width="800" height="126" alt="image" src="https://github.com/user-attachments/assets/da909d7f-8865-4b08-aa1a-c473864274a2" />
- Проверяем наши действия `lsblk`
<img width="800" height="175" alt="image" src="https://github.com/user-attachments/assets/8f74fb8e-6463-4c42-b65e-6129ee21362a" />
- т.к. изначально моя система была установлена не на LV создам для имитации раздел на sda2
  1) создаю VG на sda2 `vgcreate bubuntu_vg /dev/sda2`
  2) Создаю LV на весь размер `lvcreate -n bubuntu_vg/bubuntu_lv -l 100%FREE /dev/bubuntu_vg`
  результат 
  <img width="800" height="183" alt="image" src="https://github.com/user-attachments/assets/147881e8-8e46-4767-91e5-3b3075744060" />
  можно вернуться к методичке
  <img width="800" height="208" alt="image" src="https://github.com/user-attachments/assets/003bbb61-4beb-4f49-8d74-c207a300c1de" />
- нужно уменьшить размер рут с 25 гб до 8 гб. Удаляем том
  ```bash
  lvremove /dev/bubuntu_vg/bubuntu_lv
  ```
  создаем новый с ключом `-L` чтоб указать конекртный размер
  ```bash
  lvcreate -n bubuntu_vg/bubuntu_lv -L 8G /dev/bubuntu_vg
  ```
  результат наших действий 
  <img width="800" height="295" alt="image" src="https://github.com/user-attachments/assets/e5e46a2d-a3df-4788-8e2f-0388ca85e894" />
- теперь нужно вернуть всё что мы перенесли обратно повторяем действия с поправкой на расположение 
  1) `mkfs.ext4 /dev/bubuntu_vg/bubuntu_lv`
  2) `mount /dev/bubuntu_vg/bubuntu_lv /mnt`
  3) `rsync -avxHAX --progress / /mnt/`
  <img width="800" height="46" alt="image" src="https://github.com/user-attachments/assets/2d7475b2-9a8d-4991-aa70-9faa2b70533b" />
  4) `for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/root_resize/$i; done`
  5) `chroot /mnt/`
  6) `grub-mkconfig -o /boot/grub/grub.cfg`
  7) `update-initramfs -u`
  результат
  <img width="800" height="304" alt="image" src="https://github.com/user-attachments/assets/57297f32-e00a-446a-9f11-b791755a8aac" />
- по спику заданий нужно сделать том для `/home` но в методичке предлогают сделать сразу том под `/var` в зеркало
## Выделить том под /var в зеркало
- создаем зеркало
  ```bash
  pvcreate /dev/sdc /dev/sdd
  vgcreate vg_var /dev/sdc /dev/sdd
  lvcreate -L 950M -m1 -n lv_var vg_var
  ```
  <img width="800" height="129" alt="image" src="https://github.com/user-attachments/assets/67dda166-761e-4144-91fa-e3f901db517b" />
- создаем фс, монтируем и преносим `/var`
  ```bash
  mkfs.ext4 /dev/vg_var/lv_var
  mount /dev/vg_var/lv_var /mnt
  cp -aR /var/* /mnt/
  ```
  после `cp` получаем что места выделено мало 
  <img width="800" height="212" alt="image" src="https://github.com/user-attachments/assets/5a9818c5-f4df-4b18-942b-bc61a8f5a787" />
  увеличу LV отмотрировав перед этим `lvextend -L 100M /dev/vg_var/lv_var`
  <img width="800" height="115" alt="image" src="https://github.com/user-attachments/assets/c2fce730-db51-48ad-a2de-63177d42205e" />
  вывод без ошибок значит все сделано правильно
- делаем копию `/var` 
  ```bash
  mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
  ```
- перебортируем нужное
  ```bash
  umount /mnt
  mount /dev/vg_var/lv_var /var
  ```
- воспользуюсь скриптом для fstab
  ```bash
  echo "`blkid | grep var: | awk '{print $2}'` \
   /var ext4 defaults 0 0" >> /etc/fstab
  ```
- перезагружаем ОС не забыв выйти из chroot
  ```bash
  exit
  shutdown -r now
  ```




