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
   
