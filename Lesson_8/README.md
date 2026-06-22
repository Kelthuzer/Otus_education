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
 - выбираем нужную систму в моем случае Убунту и жмем E 
 редактируем параметры запуска имзенив строку начинаеющуюся с `linux` у брав все начиная с кваейт оставив только следущее
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/68ae73cb-848b-4902-a073-13cf27ccec96" />
жмежм ctrl+x
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/8f64b2b9-f587-4e81-a89a-4ffb361d478c" />
<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/b3d64a29-17e1-4418-a2d8-7e0aa7a521df" />


   
