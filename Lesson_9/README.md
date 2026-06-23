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
- ставим пакеты  
```bash
apt update
apt install -y spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid
```
потрогаем для проверки 
```bash
which spawn-fcgi
which php-cgi
```
<img width="400" height="70" alt="image" src="https://github.com/user-attachments/assets/1b5f7f03-9d62-41de-90be-fb4c6b29a69e" />  

- файл с настройками
```bash
mkdir -p /etc/spawn-fcgi
cat > /etc/spawn-fcgi/fcgi.conf <<'EOF'
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
EOF
```
- создадим юнит  
```bash 
cat > /etc/systemd/system/spawn-fcgi.service <<'EOF'
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
EOF
```
<img width="705" height="433" alt="image" src="https://github.com/user-attachments/assets/0330f14a-c9aa-4cee-83e9-e0cfcb406315" />  

- запускаем/првоеряем
```bash
systemctl start spawn-fcgi
systemctl status spawn-fcgi
```   
<img width="790" height="231" alt="image" src="https://github.com/user-attachments/assets/7d8999cc-0567-49c8-b34a-aa9afb9234f4" />  

## Nginx с разными конфигурационными файлами одновременно  
- остановим и отключим апач2 чтоб небыло конфликтов
```bash
systemctl stop apache2
systemctl disable apache2
```  
- установим Nginx и остановим его сервис чтоб небыло проблем в процессе
```bash
apt install nginx -y
systemctl stop nginx
```
- Для запуска нескольких экземпляров сервиса модифицируем исходный service для использования различной конфигурации, а также PID-файлов. Для этого создадим новый Unit для работы с шаблонами (/etc/systemd/system/nginx@.service)  
```bash
cat > /etc/systemd/system/nginx@.service <<'EOF'

# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
EOF
```
- создадим конфиги для nginx `nginx-first.conf` и `nginx-second.conf`  
```bash
cat > /etc/nginx/nginx-first.conf <<'EOF'
user www-data;
worker_processes auto;
pid /run/nginx-first.pid;

events {
    worker_connections 768;
}

http {
    access_log /var/log/nginx/access-first.log;
    error_log /var/log/nginx/error-first.log;

    server {
        listen 9001;
        server_name localhost;

        location / {
            return 200 "nginx first\n";
        }
    }
}
EOF

cat > /etc/nginx/nginx-second.conf <<'EOF'
user www-data;
worker_processes auto;
pid /run/nginx-second.pid;

events {
    worker_connections 768;
}

http {
    access_log /var/log/nginx/access-second.log;
    error_log /var/log/nginx/error-second.log;

    server {
        listen 9002;
        server_name localhost;

        location / {
            return 200 "nginx second\n";
        }
    }
}
EOF
```
- запустим наши новые сервисы и проверим их самочувствие
```bash
systemctl start nginx@first
systemctl start nginx@second
systemctl status nginx@first
systemctl status nginx@second
```
<img width="1214" height="612" alt="image" src="https://github.com/user-attachments/assets/64caec58-7de6-4126-baa2-acf50a6093b8" />  
- проверка по методичке
<img width="1238" height="64" alt="image" src="https://github.com/user-attachments/assets/cc955e8c-2703-4fe8-9c44-993d3ea7ee4f" />


