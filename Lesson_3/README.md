#  LVM
## Уменьшить том под / до 8G
- подготовка тома
  
  ```bash
  pvcreate /dev/sdb 
  vgcreate vg_root /dev/sdb
  lvcreate -n lv_root -l +100%FREE /dev/vg_root
  ```
  получаем
  <img width="737" height="146" alt="image" src="https://github.com/user-attachments/assets/788170f8-3d8f-48d7-baf9-746f0dcf7cd1" />
- создаем ФС
  
  ```bash
  mkfs.ext4 /dev/vg_root/lv_root
  ```
  <img width="599" height="188" alt="image" src="https://github.com/user-attachments/assets/b7ba26e5-2e33-458c-a081-1570801a573a" />

- монтируем `mount /dev/vg_root/lv_root /mnt`
- копируем `rsync -avxHAX --progress / /mnt/`, ждем завершения. В итоге получим
  ```
  sent 4,125,280,042 bytes  received 2,853,364 bytes  81,745,215.96 bytes/sec
  total size is 4,118,307,974  speedup is 1.00
  ```
- перепроверяем на всякий случай 
  <img width="578" height="498" alt="image" src="https://github.com/user-attachments/assets/d424d558-1ec2-4262-91f9-9200ebba2eda" />
- подготавливаем окружение 
  ```bash
  for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
  ```
- меняем корень `chroot /mnt/`
- настройка граб `grub-mkconfig -o /boot/grub/grub.cfg`
  <img width="893" height="191" alt="image" src="https://github.com/user-attachments/assets/40d75076-3da8-43e7-92a1-7fc1dc1fc512" />

- Обновим образ initrd `update-initramfs -u`
  `update-initramfs: Generating /boot/initrd.img-6.8.0-110-generic`
- Перезагружаем систему `exit` `shutdown -r now`. Получаем ошибку `Running in chroot, ignoring request.` выходим из chroot и снова пробуем перезагруку
- Проверяем наши действия `lsblk`
  <img width="742" height="53" alt="image" src="https://github.com/user-attachments/assets/cafb4535-f0f0-49b2-9cc1-8cd083717e6c" />
- нужно уменьшить размер рут с 25 гб до 8 гб. Удаляем том
  ```bash
  lvremove /dev/bubuntu_vg/bubuntu_lv
  ```
  создаем новый с ключом `-L` чтоб указать конекртный размер
  
  ```bash
  lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
  ```
  результат наших действий 
  <img width="502" height="218" alt="image" src="https://github.com/user-attachments/assets/42f715f3-00c5-4180-b71e-b44ec81c6f93" />

- теперь нужно вернуть всё что мы перенесли обратно повторяем действия с поправкой на расположение 
  1) `mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv`
  2) `mount /dev/ubuntu-vg/ubuntu-lv /mnt`
  3) `rsync -avxHAX --progress / /mnt/`
  4) `for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done`
  5) `chroot /mnt/`
  6) `grub-mkconfig -o /boot/grub/grub.cfg`
  7) `update-initramfs -u` 
  результат
  <img width="888" height="259" alt="image" src="https://github.com/user-attachments/assets/81df8f2b-8e84-4432-9818-459b77655116" />

- по спику заданий нужно сделать том для `/home` но в методичке предлогают сделать сразу том под `/var` в зеркало
## Выделить том под /var в зеркало
- создаем зеркало
  ```bash
  pvcreate /dev/sdc /dev/sdd
  vgcreate vg_var /dev/sdc /dev/sdd
  lvcreate -L 1G -m1 -n lv_var vg_var
  ```
  <img width="712" height="141" alt="image" src="https://github.com/user-attachments/assets/6d90ee56-9127-470a-b330-83371c8b6b20" />

- создаем фс, монтируем и преносим `/var`
  ```bash
  mkfs.ext4 /dev/vg_var/lv_var
  mount /dev/vg_var/lv_var /mnt
  cp -aR /var/* /mnt/
  ```
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
- результат стараний
  <img width="514" height="335" alt="image" src="https://github.com/user-attachments/assets/9dae7b89-68d3-4b4b-8494-7489bef8b4fe" />
- небольшая уборка
  ```bash
  lvremove /dev/vg_root/lv_root
  vgremove /dev/vg_root
  pvremove /dev/sdb
  ```
  результат
  <img width="730" height="448" alt="image" src="https://github.com/user-attachments/assets/dcb15c46-065e-49a5-a5ea-dfb7ec5db2d5" />





