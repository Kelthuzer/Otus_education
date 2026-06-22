# Systemd  
## Сервис мониторинга   
- создадим файл конфигурации    
```bash
nano /etc/default/watchlog
```  
наполним его слудющим содрежанием с коментариями для того чтоб потом вспомнить что это и зачем  
```bash  
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit 
WORD="ALERT"
LOG=/var/log/watchlog.log
```
<img width="452" height="123" alt="image" src="https://github.com/user-attachments/assets/568091e0-cc83-42a6-a36b-fbeec0a92c03" />  

создадим файл лога `echo "test line" >> /var/log/watchlog.log && echo "ALERT NO HUGS" >> /var/log/watchlog.log `  
- Создадим скрипт `nano /opt/watchlog.sh`
```bash  
#!/bin/bash

WORD=$1
LOG=$2
DATE=$(date)

if grep "$WORD" "$LOG" &> /dev/null
then
    logger "$DATE: I found precious, Master!"
else
    exit 0
fi
```

<img width="416" height="267" alt="image" src="https://github.com/user-attachments/assets/07457237-1994-489b-9dfc-d6c81d45c184" />  

права на исполнение `chmod +x /opt/watchlog.sh`  
- Создадим юнит 
```bash  
cat > /etc/systemd/system/watchlog.service <<'EOF'
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
EOF
```  
- Создадим дергалку по таймеру  
```bash  
cat > /etc/systemd/system/watchlog.timer <<'EOF'
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
EOF
```
- обновим настройки и запустим наш новый сервис
```bash
systemctl daemon-reload
systemctl start watchlog.service
systemctl start watchlog.timer
systemctl status watchlog.timer
```
- трогаем логи `journalctl -n 100 | grep "I found"`   
<img width="795" height="276" alt="image" src="https://github.com/user-attachments/assets/858a64e0-e6a7-4e60-88fe-6edcf0d63790" />  
  
## spawn-fcgi и создать unit-файл


