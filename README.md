# Домашнее задание к занятию 1 `«Disaster recovery и Keepalived»` - `Евгений Головенко`

---

### Задание 1

Дана схема для Cisco Packet Tracer, рассматриваемая в лекции.

На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)

Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).

Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.

На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

---

### Решение 1

<img width="1020" height="830" alt="PT" src="https://github.com/user-attachments/assets/3d070da7-550f-48a1-829a-1d3cf57d887c" />

<img width="694" height="691" alt="Router1" src="https://github.com/user-attachments/assets/e7aa0a7e-4b6e-487d-bf36-ba4dc506f23e" />

<img width="700" height="688" alt="Router2" src="https://github.com/user-attachments/assets/f75623db-1736-4f29-9a73-cf9c3a24d945" />

---

### Задание 2

1. Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного файла.
2. Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
3. Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
4. Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
5. На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

---

### Решение 2

1. 4. Развернуты 2 VM Debian (master IP 192.168.1.149, backup IP 192.168.1.47), на каждой VM поднят Keepalived. Keepalived сконфигурирован так, чтобы он запускет некий скрипт каждые 3 секунды (про скрипт будет на следующем шаге) и переносит виртуальный IP (192.168.1.111/24) на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля. Использована для этого секция vrrp_script:
   
`keepalived.conf`

```
vrrp_script chk_web {
    script "/etc/keepalived/health_check.sh"
    interval 3    # every 3 s
    timeout 2
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state backup
    interface enp0s3
    virtual_router_id 15
    priority 200
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass secretpass
    }

    virtual_ipaddress {
        192.168.1.111/24
    }

    track_script {
        chk_web
    }
}
```

2. На каждой VM запущен nginx

```
sudo apt update
sudo apt install nginx -y
echo "<h1>Hello from $(hostname)</h1>" | sudo tee /var/www/html/index.html
systemctl enable nginx --now
```

<img width="671" height="187" alt="nginx_master" src="https://github.com/user-attachments/assets/b8aa5465-170a-4758-8c00-1ee4b8c483b3" />

<img width="673" height="202" alt="nginx_backup" src="https://github.com/user-attachments/assets/ecc550e9-cd08-400e-86df-5ea0e2cfd8a6" />

3. Собственно, сам Bash-скрипт, который проверяет доступность порта веб-сервера и существование файла index.html в root-директории веб-сервера.

```
#!/bin/bash
WEB_PORT=80
WEB_ROOT="/var/www/html"
INDEX_FILE="$WEB_ROOT/index.html"
HOST="127.0.0.1"

# Checking the local web server port
nc -z "$HOST" "$WEB_PORT" >/dev/null 2>&1
if [ $? -ne 0 ]; then
    echo "Port $WEB_PORT unavailable"
    exit 1
fi

# Checking for the presence of index.html
if [ ! -f "$INDEX_FILE" ]; then
    echo "File $INDEX_FILE not found"
    exit 1
fi

# If you got here - OK
echo "All checks passed successfully"
exit 0
```

<img width="571" height="443" alt="Script" src="https://github.com/user-attachments/assets/c51c9279-5fbc-4bec-9205-849926696761" />

5. sdfsgsdg

`Скриншоты с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html`

<img width="995" height="196" alt="VIP_master" src="https://github.com/user-attachments/assets/34de2a87-bb2c-4200-9e8d-5bc82fa735f3" />

<img width="667" height="209" alt="VIP_master_brouser" src="https://github.com/user-attachments/assets/ee606d95-695d-4d24-a220-31de46b047f4" />

<img width="1037" height="176" alt="VIP_backup" src="https://github.com/user-attachments/assets/fe244367-d661-4828-92c9-e0f3b3b4bb4f" />

<img width="675" height="262" alt="VIP_backup_brouser" src="https://github.com/user-attachments/assets/9c34d868-fb1c-4e00-abf0-931c71537549" />

---

