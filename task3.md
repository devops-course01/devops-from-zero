# Домашнее задание 3

## Цель
Создать скрипт, который каждые 17 секунд записывает случайную строку длиной до 20 символов в файл `/opt/app/log.txt`, с возможностью автозапуска и ротации логов.

---

## Скрипт `/opt/app/log_writer.sh`

```bash
#!/usr/bin/env bash

DIR="/opt/app"
FILE="$DIR/log.txt"

mkdir -p "$DIR"
touch "$FILE"

random_string() {
    local length=$((RANDOM % 20 + 1))
    tr -dc 'a-zA-Z0-9' </dev/urandom | head -c "$length"
}

while true; do
    random_string >> "$FILE"
    echo >> "$FILE"
    sleep 17
done
```

## Сделать скрипт исполняемым:

```bash
chmod +x /opt/app/log_writer.sh
```

### Автозапуск через systemd

Создать unit-файл /etc/systemd/system/log_writer.service:
```bash
[Unit]
Description=Log Writer Service
After=network.target

[Service]
ExecStart=/opt/app/log_writer.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target


Включить и запустить сервис:

systemctl daemon-reload
systemctl enable log_writer.service
systemctl start log_writer.service
systemctl status log_writer.service
```

### Ротация логов через logrotate

Создать файл /etc/logrotate.d/log_writer:
```bash
/opt/app/log.txt {
    daily
    rotate 7
    compress
    missingok
    notifempty
    copytruncate
}
```

Проверка конфигурации:
```bash
logrotate -d /etc/logrotate.d/log_writer
```
