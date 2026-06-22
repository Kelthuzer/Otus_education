# Загрузка системы  
## Включить отображение меню Grub
 - Редактируем Grub
   в ручую отредактируем `nano /etc/default/grub` заменив
   ```bash
   GRUB_TIMEOUT_STYLE=hidden
   GRUB_TIMEOUT=0
   ```  
   на
   ```bash
   #GRUB_TIMEOUT_STYLE=hidden
   GRUB_TIMEOUT=10
   ```
   или воспульзуемся `sed`
   ```bash
   sed -i \
   -e 's/^GRUB_TIMEOUT_STYLE=.*/#GRUB_TIMEOUT_STYLE=hidden/' \
   -e 's/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=10/' \
   /etc/default/grub
   ```
   <img width="733" height="847" alt="image" src="https://github.com/user-attachments/assets/933ccb60-9909-4550-9e9f-144ac369e85b" />
   <img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/9f699a31-6177-4922-8a3b-4787f4a98c7e" />  
   
## Попасть в систему без пароля  
### init=/bin/bash
 - выбираем нужную систму в моем случае Убунту и жмем E 
 редактируем параметры запуска имзенив строку начинаеющуюся с `linux` у брав все начиная с кваейт оставив только следущее
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/68ae73cb-848b-4902-a073-13cf27ccec96" />
жмежм ctrl+x
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/b3d64a29-17e1-4418-a2d8-7e0aa7a521df" />
## Recovery mode
 - прерываем загрузки выбиарем Advanced, потом recovery**
 <img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/4a13a39b-0bb0-4c11-b001-b5b39ea4ce82" />
 <img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/64008657-a95c-4405-920e-5ec8918e12fc" />
далее для работы с дисками понадобиться активировать пунк нетворк
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/f6ff68a6-2a09-4eeb-a8bf-7eff46868b0b" />
и выбирам рут  
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/5562e687-e8d1-45fa-9384-db5179cdc1bd" />
у нас есть рут доступ
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/6e08343c-af5b-4793-84db-01f12a7e8510" />

## Переименовать VG 
посмотрим что имеем `vgs` и переименум `vgrename ubuntu-vg ubuntu-otus` получим
<img width="759" height="261" alt="image" src="https://github.com/user-attachments/assets/3d3d5501-92f5-4124-bb5a-48d9edf81de2" />  
отредактируем через `sed`  
```bash
sed -i 's/ubuntu--vg/ubuntu--otus/g' /boot/grub/grub.cfg
```
до 
<img width="1903" height="1017" alt="image" src="https://github.com/user-attachments/assets/f94c6eb3-2982-424f-8d23-bd028b04762b" />  
после  
<img width="1903" height="1017" alt="image" src="https://github.com/user-attachments/assets/4244e82e-99fc-4db4-88a8-f5ffae6db080" />
результат  
<img width="407" height="53" alt="image" src="https://github.com/user-attachments/assets/e2bd73ad-e03e-40f8-bea4-aa98da3da043" />


