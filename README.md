#linux_webserver Ubuntu 26.04.1 LTS
    Настройка Linux-сервера для веб-приложения

    Нижеперечисленные команды необходимо запускать от root, войдя в него через sudo su

1. Создание пользователя
    1.1 Создание пользователя:
        adduser *name*
    1.2 Проверка пользователя:
        id *name*
    1.3 Предоставить пользователю права sudo:
        usermod -aG sudo *name*
    1.4 Зайти от пользователя *name*:
        su - *name*
    1.5 Проверить текущего пользователя:
        whoami

2. SSH подключение (перед началом необходимо посмотреть ip адрес, что получил сервер, а так же установлен ли там openssh)
    2.1 Создание ключа на ПК
        ssh-keygen -t ed25519 -C "linux_server_deploy"
    2.2 C ПК отправить ключ на сервер
        ssh-copy-id -i ~/.ssh/id_ed25519.pub *name*@ip адрес сервера
    2.3 Проверка подключения
        ssh -i ~/.ssh/id_ed25519 *name*@ip адрес сервера 
    2.4 Запретить подключаться через root
        2.4.1 Открыть конфигурацию SSH
            sudo nano /etc/ssh/sshd_config
        2.4.2 Найти параметр PermitRootLogin, раскомментить и заменить на 
            PermitRootLogin no
        2.4.3 Сохранить и применить изменения
            sudo systemctl reload ssh

3. Установка и настройка Firewall
    3.1 Проверка наличия UFW
        sudo ufw status verbose (в случае если Status: inactive - значит установлен, но просто отключен.  Если ufw: command.... - значит нужно установить sudo apt install ufw)
    3.2 Разрешаем нужные порты
        sudo ufw allow 22/tcp #ssh
        sudo ufw allow 80/tcp #http
        sudo ufw allow 443/tcp #https
    3.3 Установить политику
        sudo ufw default deny incoming  - входящие подключения запрещены, если их не разрешает отдельное правило
        sudo ufw default allow outgoing - исходящие подключения разрешены
    3.4 Включить и проверить firewall
        sudo ufw enable
        sudo ufw status verbose

4. Установка и настройка Nginx
    4.1 Установка
        sudo apt install nginx
    4.2 Запустить и настроить автозапуск
        sudo systemctl enable --now nginx
    4.3 Проверить, слушает ли nginx 80 порт
        sudo ss -tlnp |grep '80'

5. Развернуть html страницу через Nginx
    5.1 Создать директорию с файлом html
    5.2 Возможность читать html и директорию
        sudo chmod 755 *путь к папке*
        sudo chmod 644 *путь к файлу*
    5.3 Cоздание конфигурации сайта Nginx
        sudo nano /etc/nginx/sites-available/kai-app
        
                                                    server {
                                                listen 80;
                                                listen [::]:80;

                                                server_name _;

                                                root /var/www/kai-app;
                                                index index.html;

                                                location / {
                                                    try_files $uri $uri/ =404;
                                                }
                                            }
    5.4 Проверка конфигурации
        sudo nginx -t
    5.5 Запуск и проверка страницы
        sudo systemctl reload nginx
        curl -i http://127.0.0.1/

6. Реализация health-check /health
    6.1 Открыть и изменить ранее созданую конфигурация Nginx
        sudo nano /etc/nginx/sites-available/kai-app
                                                        server {
                                                            listen 80;
                                                            listen [::]:80;

                                                            server_name _;

                                                            root /var/www/kai-app;
                                                            index index.html;

                                                            location = /health {
                                                                default_type text/plain;
                                                                return 200 "OK\n";
                                                            }

                                                            location / {
                                                                try_files $uri $uri/ =404;
                                                            }
                                                        }
    6.2 Проверить и применить изменеия
        sudo nginx -t
        sudo systemctl reload nginx
        curl -i http://127.0.0.1/health