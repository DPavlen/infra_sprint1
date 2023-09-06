# Kittygram
Kittygram — социальная сеть для обмена фотографиями любимых питомцев.
 Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

##Технологии:
***
**Nginx/1.18.0**

**Django==3.2.3**

**djangorestframework==3.12.4**

**djoser==2.1.0**

**webcolors==1.11.1**

**Pillow==9.0.0**

**pytest==6.2.4**

**pytest-django==4.4.0**

**pytest-pythonpath==0.7.3**

**python-dotenv==1.0.0**

**gunicorn==20.1.0**
***
***
## Функционал проекта:
- Позволяет добавлять фотографии своих котиков и просматривать добавленные фото котиков других пользователей;
- При загрузке котиков необходимо заполнить обязательные поля и по желанию поле "Достижения";
- Добавлять и просматривать фотографии могут только зарегистрированные и авторизованные пользователи.

***
## Установка
1. Склонировать репозиторий
```
git clone git@github.com:DPavlen/infra_sprint1.git
```
2. Создать и активировать виртуальное окружение
```
python3 -m venv venv
source venv/bin/activate
```
3. Установить зависимости для Python
```
pip install -r requirements.txt 
```
4. Перейти в папку backend и выполнить миграции
```
cd infra_sprint1/backend/
python manage.py migrate
```
5. Создать суперюзера
```
python manage.py createsuperuser
```
6. В разделе /infra_sprint1/backend  необходимо создать дополнительный файл настройки.

Создайте файл **.env** в папке **backend** (в папке с `manage.py`), затем добавьте строки, содержащиеся в файле **.env.example** и добавьте свои значения.

7. В этом же файле поменять значение переменной DEBUG с True на False
```
DEBUG = False
```

## Настройка фронтенд-приложения
1. Находясь в директории с фронтенд-приложением, установите зависимости для него:
```
npm i
```
2. Из директории с фронтенд-приложением выполните команду:
```
npm run build
```
3. Скопируйте статику фронтенд-приложения в директорию по умолчанию:
 Точка после build важна — будет скопировано содержимое директории.
```
sudo cp -r путь_к_директории_с_фронтенд-приложением/build/. /var/www/имя_проекта/
```

## Установка и настройка WSGI-сервера Gunicorn
1. Подключитесь к удалённому серверу, активируйте виртуальное окружение
бэкенд-приложения и установите пакет gunicorn: 
```
pip install gunicorn==20.1.0
```
2. Перейдите в директорию с файлом manage.py, и запустите Gunicorn:
```
gunicorn --bind 0.0.0.0:8080 backend.wsgi
```
3. Необходимо использовать порт 8080, так порт 8000 уже занят.
Пример кода можно посмотреть в корне проекта в папке /infra/ .
```
sudo nano /etc/systemd/system/gunicorn_kittygram.service
```
4. Необходимо скопировать содержимое файла sudo nano /etc/systemd/system/gunicorn.service
в файл sudo nano /etc/systemd/system/gunicorn_kittygram.service с помощью команды cp,
нужно выполнить следующую команду в командной строке:
```
sudo cp /etc/systemd/system/gunicorn.service /etc/systemd/system/gunicorn_kittygram.service
```
##### Внимание, вы должны изменить USER_NAME на свое имя пользователя в файле!

**Примечание.** Вы можете найти мой пример конфигурации Gunicorn в папке **infra** 
и ознакомиться с ним.
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=USER_NAME
WorkingDirectory=/home/USER_NAME/infra_sprint1/backend/
ExecStart=/home/USER_NAME/infra_sprint1/backend/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target
```
Путь к директории проекта.
WorkingDirectory=/home/имя_пользователя/папка_с_проектом/папка_с_файлом_manage.py/

Запустите процесс gunicorn_kittygram.service:
sudo systemctl start gunicorn_kittygram

Команда sudo systemctl с параметрами start, stop или restart запустит, остановит
или перезапустит Gunicorn. 

Чтобы systemd следил за работой демона Gunicorn, запускал его при старте системы
и при необходимости перезапускал, используйте команду:
```
sudo systemctl enable gunicorn_kittygram
```
## Установка и настройка веб- и прокси-сервера Nginx
1. Если Nginx ещё не установлен на удалённый сервер, установите его:
```
sudo apt install nginx -y
```
2. Запустите Nginx командой:
```
sudo systemctl start nginx
```
3.В существующем файле "default" конфигурации Ngnix добавьте новые настройки
для приложения kittygram:
```
sudo nano /etc/nginx/sites-enabled/default
```
```
server {
    server_name YOUR_IP & YOUR_DOMAIN;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        client_max_body_size 20M;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
        client_max_body_size 20M;
    }

    location /media/ {
        root /var/www/kittygram;
    }

    location / {
        root   /var/www/kittygram;
        index  index.html index.htm;
        try_files $uri /index.html;
    }
}
```
Сохраните изменения в файле, закройте его и проверьте на корректность:
```
sudo nano /etc/nginx/sites-enabled/default
```
Перезагрузите конфигурацию Nginx:
```
sudo systemctl reload nginx
```
**Примечание.** Далее Введите в адресную строку браузера IP-адрес вашего удалённого сервера без указания порта.
Должна открыться страница приветствия от Nginx. Это подтверждает, что программа работает и отвечает.

## Настройка файрвола ufw

Файрвол установит правило, по которому будут закрыты все порты, кроме тех, которые
вы явно укажете.
Настройка файрвола ufw
Получение и настройка SSL-сертификата
1. Активируйте разрешение принимать запросы только на порты 80, 443 и 22:
80, 443: с ними будут работать пользователи, делая запросы к приложению.
```
sudo ufw allow 'Nginx Full'
```
22: нужен, чтобы вы могли подключаться к серверу по SSH.
```
sudo ufw allow OpenSSH
```
2. Включите файрвол:
```
sudo ufw enable
```
В терминале выведется запрос на подтверждение операции. Введите y и нажмите Enter.

3. Проверьте работу файрвола:
```
sudo ufw status
```

## Получение и настройка SSL-сертификата

1. Находясь на сервере, установите certbot, если он ещё не установлен:
Установка пакетного менеджера snap.
```
sudo apt install snapd
```
Установка и обновление зависимостей для пакетного менеджера snap.
```
sudo snap install core; sudo snap refresh core
```
Установка пакета certbot.
```
sudo snap install --classic certbot
```
Обеспечение доступа к пакету для пользователя с правами администратора.
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
2. Запустите certbot и получите SSL-сертификат. Для этого выполните команду:
```
sudo certbot --nginx
```
Далее система попросит вас указать электронную почту и ответить на несколько вопросов. Сделайте это.
Следующим шагом укажите имена, для которых вы хотели бы активировать HTTPS:
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

Введите номер нужного доменного имени и нажмите Enter.

После этого certbot отправит ваши данные на сервер Let's Encrypt и там будет выпущен
сертификат, который автоматически сохранится на вашем сервере в системной директории /etc/ssl/. Также будет автоматически изменена конфигурация Nginx: в файл
/etc/nginx/sites-enbled/default добавятся новые настройки и будут прописаны пути к сертификату.

5. Сохраните изменения, проверьте и перезагрузите конфигурацию веб-сервера Nginx:
```
sudo nginx -t
sudo systemctl reload nginx 
``` 

## Автор проекта:
- [Dmitry Pavlenko](https://github.com/DPavlen)
