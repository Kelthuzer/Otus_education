# Работа с NFS
## Стенд   
Тестовый стенд сделан клонированием
* Хост `otus_host` ip:`192.168.88.30`
* Клиент `otus_client` ip:`192.168.88.28`
### Настраиваем сервер NF
  - Установка NFS `apt install nfs-kernel-server` и проверка что сервис слушает порты <br>
  <img width="1049" height="128" alt="image" src="https://github.com/user-attachments/assets/869c0ac6-2c17-4493-ba13-e3d4680a78bf" />
  - Создаём и настраиваем директорию:
    
```bash
mkdir -p /srv/share/upload
chown -R nobody:nogroup /srv/share
chmod 0777 /srv/share/upload
```
  - Cоздаём в файле /etc/exports структуру указываем ip клиента
    
```bash
cat << EOF > /etc/exports 
/srv/share 192.168.88.28/32(rw,sync,root_squash)
EOF
```

  - Экспортируем директорию `exportfs -r`<br>
    <img width="999" height="78" alt="image" src="https://github.com/user-attachments/assets/30079f76-2412-4c40-9bbd-6fee7e2d2b29" />
  - Проверяем `exportfs -s`
    Вывод `/srv/share  192.168.88.28/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)`
    
### Настраиваем клиент NFS

- ставим клиент `apt install nfs-common`
- Добавляем в `/etc/fstab` точку монтирования
  ```bash
   echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
  ```
  - перезегружаем сервисы
  ```bash
  systemctl daemon-reload
  systemctl restart remote-fs.target
  ```
  - проверям созданием файла<br>
  клиент<br>
  <img width="582" height="178" alt="image" src="https://github.com/user-attachments/assets/b4915e3a-9a32-4fe9-91d2-97dadfabed3e" /><br>
  хост <br>
  <img width="489" height="136" alt="image" src="https://github.com/user-attachments/assets/559b2cdc-9061-460e-9ea4-ba23dd2a11f5" /><br>

