# Конфигурация виртуальной машины devops-vm

## 1. Параметры машины
- ОЗУ: 2048 МБ
- Ядра: 2
- Диск: 25 ГБ (динамически расширяемый)

## 2. Сетевые интерфейсы
- Адаптер 1: NAT (10.0.2.15) — доступ в интернет
- Адаптер 2: Host-only (192.168.56.101) — доступ с хоста

## 3. Правило проброса портов
- Порт хоста: 2222
- Порт гостя: 2222(был 22)

## 4. Учётные записи
- zainullin (sudo, пароль)
- devops (sudo, вход по ключу)

## 5. Служба SSH
- Порт: 2222
- Файл конфигурации: /etc/ssh/sshd_config.d/99-hardening.conf
- Изменённые директивы: PermitRootLogin no, PasswordAuthentication no, MaxAuthTries 3, AllowUsers devops

## 6. Правила межсетевого экрана
- Политики: deny incoming, allow outgoing
- Правила: 2222/tcp LIMIT, 80/tcp ALLOW, 443/tcp ALLOW

## 7. Снимки состояния
- 01-clean-install
- 02-keys-configured
- 03-ssh-hardened
- 04-ufw-configured
- 05-dns-configured
- 04-nginx-https
- 05-docker-compose

## 8. Веб-сервер
- Пакет: nginx (из репозитория Ubuntu, версия 1.28.3), служба nginx.service, автозапуск включён
- Процессы: главный — root, рабочие — www-data
- Конфигурация ресурса: /etc/nginx/sites-available/devops-site, ссылка в /etc/nginx/sites-enabled/; стандартный ресурс default отключён (удалена ссылка)
- Блоки server: порт 80 — перенаправление 301 на https://$host$request_uri; порт 443 ssl — обслуживание ресурса, протоколы TLSv1.2 TLSv1.3
- Имя ресурса: devops.local (запись в hosts на хосте, адрес Host-only)
- Каталог ресурса: /var/www/devops-site, владелец devops:devops, права каталога 755, файлов 644
- Страница ошибки: /404.html
- Журналы: /var/log/nginx/devops-site.access.log, /var/log/nginx/devops-site.error.log
- Сертификат: /etc/ssl/certs/devops.crt, root:root, права 644
- Закрытый ключ: /etc/ssl/private/devops.key, root:root, права 600
- Формирование сертификата (самоподписанный, RSA 2048, срок действия 365 дней):
  sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/devops.key -out /etc/ssl/certs/devops.crt -subj "/CN=devops.local" -addext "subjectAltName=DNS:devops.local"
- Доставка содержимого: scripts/deploy.sh (rsync --delete --chmod=D755,F644 из каталога site/ репозитория)

## 9. Контейнеризация
- Docker Engine 29.9.2 (Community) из официального репозитория download.docker.com (пакеты docker-ce, docker-ce-cli, containerd.io, docker-buildx-plugin, docker-compose-plugin); пакет docker.io не используется; служба docker.service, автозапуск включён
- Участники группы docker: devops (имеет sudo; участие в группе равносильно правам root)
- Проект на сервере: /home/devops/app (доставляется из каталога app/ репозитория: rsync -avz --delete --exclude=.env app/ devops:app/)
- Сервисы: app (образ devops-app:1.0, собирается из app/Dockerfile, процесс от пользователя app), db (postgres:16-alpine, проверка готовности pg_isready); restart: unless-stopped
- Файл окружения: /home/devops/app/.env, владелец devops, права 600; создаётся на сервере из .env.example, пароль POSTGRES_PASSWORD формируется командой openssl rand -hex 16; в репозиторий не добавляется (.gitignore, .dockerignore)
- Публикуемые порты: 127.0.0.1:5000 -> app:5000; порт СУБД 5432 не публикуется
- Сеть проекта: app_backend (bridge), СУБД доступна приложению по имени db
- Именованные тома: app_db_data -> /var/lib/postgresql/data (данные PostgreSQL)
- Обратный прокси-сервер: в блоке HTTPS /etc/nginx/sites-available/devops-site location /app/ -> proxy_pass http://127.0.0.1:5000/ с заголовками Host, X-Real-IP, X-Forwarded-For, X-Forwarded-Proto; статический ресурс сохранён
