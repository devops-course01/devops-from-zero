# Домашнее задание 4

## Цель
Написать скрипт для проверки доступности ресурса по адресу (например `http://app.local`) и возвращать код ошибки при недоступности. Также настроить локальный Nginx-сервер и доменную запись `app.local`.

---

## 1️⃣ Установка Nginx и настройка локальной доменной записи

Установка Nginx (Ubuntu/Debian):
```bash
sudo apt update
sudo apt install -y nginx
```
Создать локальную доменную запись app.local:
   
Добавить в файл /etc/hosts строку:
```bash
lua
127.0.0.1   app.local
```
Настроить HTTPS-сертификат для app.local:

Создать self-signed сертификат:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/app.local.key \
-out /etc/ssl/certs/app.local.crt
```

Настроить Nginx для использования HTTPS:

```bash
server {
    listen 443 ssl;
    server_name app.local;

    ssl_certificate /etc/ssl/certs/app.local.crt;
    ssl_certificate_key /etc/ssl/private/app.local.key;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```
Скрипт проверки доступности ресурса
Bash-версия (check_resource.sh)
```bash
#!/usr/bin/env bash

URL=$1

if [[ -z "$URL" ]]; then
    echo "Usage: $0 <URL>"
    exit 1
fi

# Проверка доступности через curl
if curl --head --silent --fail "$URL" >/dev/null; then
    echo "Resource $URL is available."
    exit 0
else
    echo "Resource $URL is NOT available."
    exit 1
fi
```
Сделать скрипт исполняемым:

```bash
chmod +x check_resource.sh
```
Пример запуска:

```bash
./check_resource.sh http://app.local
```
Python-версия (check_resource.py)
```python
#!/usr/bin/env python3
import sys
import requests

if len(sys.argv) < 2:
    print("Usage: python check_resource.py <URL>")
    sys.exit(1)

url = sys.argv[1]

try:
    response = requests.head(url, timeout=5)
    if response.status_code < 400:
        print(f"Resource {url} is available.")
        sys.exit(0)
    else:
        print(f"Resource {url} returned status {response.status_code}.")
        sys.exit(1)
except requests.RequestException:
    print(f"Resource {url} is NOT available.")
    sys.exit(1)
```
Запуск Python-скрипта:

```bash
python3 check_resource.py http://app.local
```
