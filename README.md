# Kittygram
***
### Описание:
Kittygram - социальная сеть для обмена фотографиями любимых питомцев, а именно, котиков.

### Стек используемых технологий:
- _Python 3.8_
- _Django REST Framework_
- _Gunicorn_
- _Nginx_
- _Sqlite3_
- _React_

## Клонирование и развертывание проекта
#### Git
- Клонируйте репозиторий, переместитесь в директорию с проектом, настройте окружение, выполните миграции
```shell
git clone git@github.com:ваш_гитхаб_аккаунт/infra_sprint1.git
cd infra_sprint1/backend/
python3 -m venv venv
source venv/bin/activate
cd ..
pip install -r requirements.txt
cd backend
python manage.py migrate
python manage.py createsuperuser
```
***
#### Django
- В settings.py добавьте в список ALLOWED_HOSTS внешний IP сервера и домен:
```python
# Вместо xxx.xxx.xxx.xxx укажите IP сервера, а вместо kittygramanton.ddns.net – ваше доменное имя.
ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'kittygramanton.ddns.net']
```
- Создайте файл .env, содержащий в себе следующие переменные окружения:
  * TOKEN - токен безопасности для файла infra_sprint1/backend/kittygram_backend/settings.py
***
#### Static
- Скопируйте директорию static_backend/ в директорию /var/www/название_проекта/:
```shell
sudo cp -r путь_к_директории_с_бэкендом/static_backend /var/www/название_проекта/
```
- Скопируйте статику фронтенд-приложения в директорию по умолчанию:
```shell
sudo cp -r путь_к_директории_с_фронтенд-приложением/build/. /var/www/название_проекта/
```
***
#### Gunicorn
- Создайте файл конфигурации юнита systemd для Gunicorn в директории
/etc/systemd/system/. Назовите его по шаблону gunicorn_название_проекта.service:
```shell
sudo nano /etc/systemd/system/gunicorn_название_проекта.service
```
- В файл конфигурации добавьте этот код, подставив свои данные:
```shell
[Unit]
Description=gunicorn daemon
After=network.target
[Service]
User=имя_пользователя_в_системе
WorkingDirectory=/home/имя_пользователя/папка_с_проектом/папка_с_файлом_manage.py/
ExecStart=/.../venv/bin/gunicorn --bind 0.0.0.0:8000 kittygram_backend.wsgi:application
[Install]
WantedBy=multi-user.target 
```
- Запустите и активируйте автозапуск демона Gunicorn
```shell
sudo systemctl start gunicorn_название_проекта
sudo systemctl enable gunicorn_название_проекта
```
***
#### Nginx
- Установите и настройте веб- и прокси-сервер Nginx
```shell
sudo apt install nginx -y
sudo systemctl start nginx
sudo nano /etc/nginx/sites-enabled/default 
```
- Очистите содержимое файла и запишите новые настройки:
```text
server {
 listen 80;
 server_name ваш_домен;
 
 location /api/ {
    proxy_pass http://127.0.0.1:8000;
 }

 location /admin/ {
    proxy_pass http://127.0.0.1:8000;
 }
 
 location / {
    root /var/www/имя_проекта;
    index index.html index.htm;
    try_files $uri /index.html;
 }
}
```
- Сохраните изменения и перезагрузите конфигурацию nginx
```shell
sudo systemctl reload nginx
```
***
#### Файрвол и SSL-сертификат (Let's Encrypt)
- Настройте файрвол на порты 80, 443 и 22, а затем включите и проверьте:
```shell
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH

sudo ufw enable
sudo ufw status
```
- Установите certbot
```shell
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
```
- Запустите certbot и укажите номер домена для активации HTTPS:
```shell
sudo certbot --nginx

Account registered.
Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains
in a VirtualHost/server block.
1: <доменное_имя_вашего_проекта_1>
2: <доменное_имя_вашего_проекта_2>
3: <доменное_имя_вашего_проекта_3>
...
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
```

#### Проект готов к использованию!
***
### Автор работы:
**[Антон Земцов](https://github.com/antonata-c)**
