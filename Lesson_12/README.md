# Задача
Вариант 2. Реализация аналога lsof
Создайте скрипт, который определяет открытые файлы процессов через /proc.
Реализуйте вывод минимум следующих данных: PID процесса, имя процесса, путь к открытому файлу
Проверьте работу скрипта на запущенной системе.
Зафиксируйте пример результата работы.
 Ожидаемый результат:
рабочий скрипт, отображающий открытые файлы процессов.

## Скрипт с коментариями

```bash
#!/bin/bash

printf "%-8s %-8s %s\n" "PID" "PPID" "PROCESS"
printf "%-8s %-8s %s\n" "--------" "--------" "--------------------"

# Получаем список процессов из /proc
for proc_dir in /proc/[0-9]*; do
    [ -r "$proc_dir/status" ] && [ -r "$proc_dir/comm" ] || continue

    # PID берём из имени каталога /proc/PID
    pid=$(printf '%s\n' "$proc_dir" | sed 's|.*/||')

    # Имя процесса и PID родителя
    process_name=$(sed -n '1p' "$proc_dir/comm" 2>/dev/null)
    ppid=$(grep -m1 '^PPid:' "$proc_dir/status" 2>/dev/null |
        awk '{print $2}')

    [ -n "$process_name" ] || continue

    printf "%-8s %-8s %s\n" \
        "$pid" \
        "${ppid:-?}" \
        "$process_name"
done | sort -n

echo
read -rp "Введите PID процесса: " selected_pid

# Проверяем формат PID
if ! printf '%s\n' "$selected_pid" | grep -Eq '^[0-9]+$'; then
    echo "Ошибка: PID должен состоять из цифр."
    exit 1
fi

if [ ! -d "/proc/$selected_pid" ]; then
    echo "Ошибка: процесс с PID $selected_pid не найден."
    exit 1
fi

fd_dir="/proc/$selected_pid/fd"

if [ ! -r "$fd_dir" ] || [ ! -x "$fd_dir" ]; then
    echo "Ошибка: нет доступа к дескрипторам. Запустите через sudo."
    exit 1
fi

# Получаем актуальные данные выбранного процесса
process_name=$(sed -n '1p' "/proc/$selected_pid/comm" 2>/dev/null)
ppid=$(grep -m1 '^PPid:' "/proc/$selected_pid/status" 2>/dev/null |
    awk '{print $2}')

printf "\nПроцесс: %s\n" "$process_name"
printf "PID:     %s\n" "$selected_pid"
printf "PPID:    %s\n\n" "${ppid:-?}"

printf "%-8s %s\n" "FD" "OPEN FILE"
printf "%-8s %s\n" "--------" "----------------------------------------"

found=0

# Читаем символические ссылки из /proc/PID/fd
for fd in "$fd_dir"/*; do
    [ -L "$fd" ] || continue

    file_path=$(readlink "$fd" 2>/dev/null)
    [ -n "$file_path" ] || continue

    printf "%-8s %s\n" "${fd##*/}" "$file_path"
    found=1
done

[ "$found" -eq 0 ] &&
    echo "Открытые дескрипторы не найдены или недоступны."
```

## Демонстарицая работы
<img width="1240" height="1024" alt="image" src="https://github.com/user-attachments/assets/33095a0a-6829-4d0a-9875-acca93d308e8" />
<img width="1240" height="1024" alt="image" src="https://github.com/user-attachments/assets/231f3011-def1-49b5-b636-e62466ce7866" />


