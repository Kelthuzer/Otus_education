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
- установка пакетов из текущей директории и проверка <br>
 ```bash
 sudo dpkg -i *.deb && ls -al /boot
 ```
 
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/85d87405-4ea8-4fd6-a7b7-a45ee020d588" /><br>
