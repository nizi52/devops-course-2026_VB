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