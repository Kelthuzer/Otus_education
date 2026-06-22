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
права на исполнение `chmod +x /opt/watchlog.sh`
