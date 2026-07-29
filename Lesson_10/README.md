# Написать bash-скрипт, который ежечасно формирует и отправляет на email отчёт о работе веб-сервера;

```bash
#!/bin/bash

ACCESS_LOG="/var/log/nginx/access.log"
ERROR_LOG="/var/log/nginxerror.log"

EMAIL="test@mail.ru"
LOCK_FILE="/tmp/report_nginx.lock"
REPORT="/tmp/report_nginx.txt"

START_TIME=$(date -d "1 hour ago" "+%d/%b/%Y:%H")
END_TIME=$(date "+%d/%b/%Y:%H")

cleanup() {
    rm -f "$LOCK_FILE"
}

trap cleanup EXIT INT TERM

if [ -f "$LOCK_FILE" ]; then
    echo "Скрипт уже запущен"
    exit 1
fi

touch "$LOCK_FILE"

create_report() {
    echo "Отчёт веб-сервера" > "$REPORT"
    echo "Период: $START_TIME - $END_TIME" >> "$REPORT"
    echo >> "$REPORT"

    echo "Топ IP-адресов:" >> "$REPORT"
    grep "$START_TIME" "$ACCESS_LOG" |
        awk '{print $1}' |
        sort |
        uniq -c |
        sort -nr |
        head -10 >> "$REPORT"

    echo >> "$REPORT"
    echo "Топ URL:" >> "$REPORT"
    grep "$START_TIME" "$ACCESS_LOG" |
        awk '{print $7}' |
        sed 's/?[^ ]*//' |
        sort |
        uniq -c |
        sort -nr |
        head -10 >> "$REPORT"

    echo >> "$REPORT"
    echo "HTTP-коды:" >> "$REPORT"
    grep "$START_TIME" "$ACCESS_LOG" |
        awk '{print $9}' |
        sort |
        uniq -c |
        sort -nr >> "$REPORT"

    echo >> "$REPORT"
    echo "Ошибки:" >> "$REPORT"

    find "$LOG_DIR" -name "error.log" -type f -exec tail -n 20 {} \; \
        >> "$REPORT"
}

send_report() {
    mail -s "Web server report" "$EMAIL" < "$REPORT"
}

create_report
send_report
```
Cron
`5 * * * * /usr/local/sbin/web_report.sh >> /var/log/web_report_cron.log 2>&1`
