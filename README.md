_рус_

# Kittygram - учебный проект. Минимальная социальная сеть для размещения фотографий котиков.

-------------

## Описание проекта
-------------

Данный проект создавался в рамках учебного курса, однако поддерживает весь необходимый минимальный функционал. Основная задача данного проекта - научиться работать с кодом, как готовой базой для деплоя и публикация его, как готового проекта в промышленном использовании.


## Технологии
-------------

- Python 3.9
- Django==3.2.3
- djangorestframework==3.12.4
- Nginx
- gunicorn


## Установка проекта на локальный компьютер из сетевого репозитория
-------------


- Клонировать репозиторий `git clone <адрес вашего репозитория>`
- Перейти в директорию с клонированным репозиторием
- Установить виртуальное окружение `python3 -m venv venv`
- Установить зависимости `pip install -r requirements.txt`
- В директории /backend/kittygram_backend/ создать файл .env
- В файле .env прописать ваш SECRET_KEY в виде: `SECRET_KEY = '<ваш_ключ>'`


# Деплой проекта на удаленный сервер
-------------

## Подключение сервера к аккаунту на GitHub
-------------

- На сервере должен быть установлен Git. Для проверки выполнить `git --version`
- Если Git не установлен - установить командой `sudo apt install git`
- Находясь на сервере сгенерировать пару SSH-ключей командой `ssh-keygen`
- Сохранить открытый ключ в вашем аккаунте на GitHub. Для этого вывести ключ в терминал командой `cat .ssh/id_rsa.pub`. Скопировать ключ от символов ssh-rsa, включительно, и до конца. Добавить это ключ к вашему аккаунту на GitHub.
- Клонировать проект с GitHub на сервер: `git clone git@github.com:Ваш_аккаунт/<Имя проекта>.git`

## Запуск backend проекта на сервере
-------------

- Установить пакетный менеджер и утилиту для создания виртуального окружения `sudo apt install python3-pip python3-venv -y`
- Находясь в директории с проектом создать и активировать виртуальное окружение `python3 -m venv venv && source venv/bin/activate`
- Установить зависимости `pip install -r requirements.txt`
- Выполнить миграции `python3 manage.py migrate`
- Создать суперюзера `python3 manage.py createsuperuser`
- Отредактировать settings.py на сервере: в список ALLOWED_HOSTS добавить внешний IP-адрес вашего сервера и адреса 127.0.0.1 и localhost . ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']

## Запуск frontend проекта на сервере
-------------

- Установить на сервер `Node.js` командами `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\ sudo apt-get install -y nodejs`
- Установить зависимости frontend приложения. Из директории `<ваш проект>/frontend/` выполнить команду: `npm i`

## Установка и запуск Gunicorn
-------------

- При активированном виртуальном окружении проекта установить пакет gunicorn `pip install gunicorn==20.1.0`

- открыть файл settings.py проекта и установить для константы `DEBUG` значение `False` `DEBUG = False`

- В директории /etc/systemd/system/ создайте файл _gunicorn.service_ `sudo nano /etc/systemd/system/gunicorn.service` со следующим кодом (без комментариев):

```
  [Unit]

  Description=kittygram_gunicorn daemon

  After=network.target

  [Service]

  User=yc-user

  WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/

  ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

  [Install]

  WantedBy=multi-user.target
```

Чтобы точно узнать путь до Gunicorn можно при активированном виртуальном окружении использовать команду `which gunicorn`

## Установка и настройка Nginx
-------------

- На сервере из любой директории выполнить команду: `sudo apt install nginx -y`
- Для установки ограничений на открытые порты выполнить по очереди команды: `sudo ufw allow 'Nginx Full'` `sudo ufw allow OpenSSH`
- Включить файервол `sudo ufw enable`
- Собрать и разместить статику frontend-приложения.
- Перейти в директорию <имя_проекта>/frontend/ и выполнить команду `npm run build` Результат сохранится в директории .../frontend/build/. В системную директорию сервера /var/www/ скопировать содержимое папки /frontend/build/
- Открыть файл конфигурации веб-сервера `sudo nano /etc/nginx/sites-enabled/default` и заменить его содержимое следующим кодом:

```
server {

      listen 80;
      server_name публичный_ip_вашего_удаленного_сервера;
  
      location / {
      root   /var/www/<имя проекта>;
      index  index.html index.htm;
      try_files $uri /index.html;
      }
  }
```


- Проверить корректность конфигурации `sudo nginx -t`
- Перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`
- Настроить проксирование запросов
- Открыть файл конфигурации Nginx /etc/nginx/sites-enabled/default и добавить в него ещё один блок `location`

```
server {

      listen 80;
      server_name публичный_ip_вашего_удаленного_сервера;

      location /api/ {
          proxy_pass http://127.0.0.1:8080;
      }
      
      location /admin/ {
  	    proxy_pass http://127.0.0.1:8000;
  		}
  	
      location / {
          root   /var/www/<имя_проекта>;
          index  index.html index.htm;
          try_files $uri /index.html;
      }
  }
```

- Сохранить изменения, проверить и перезагрузить конфигурацию веб-сервера:

```
  sudo nginx -t
  sudo systemctl reload nginx
```

- Собрать и настроить статику для backend-приложения.
- В файле settings.py прописать настройки:

```
 STATIC_URL = 'static_backend'
 STATIC_ROOT = BASE_DIR / 'static_backend'
```

- Активировать виртуальное окружение проекта, перейти в директорию с файлом manage.py и выполнить команду `python3 manage.py collectstatic`
- В директории_<имя_проекта>/backend/_ будет создана директория static_backend/
- Скопировать директорию static_backend/ в директорию /var/www/<имя_проекта>/


## Добавление доменного имени в настройки Django
-------------

- В файле settings.py добавить в список `ALLOWED_HOSTS` доменное имя: ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен']
- Сохранить изменения и перезапустить gunicorn `sudo systemctl restart gunicorn`
- Внести изменения в конфигурацию Nginx. Открыть конфигурационный файл командой: `sudo nano /etc/nginx/sites-enabled/default`
- Добавьте в строку `server_name` выданное вам доменное имя (через пробел, без `< >`):

```
server {
  ...
      server_name <ваш-ip> <ваш-домен>;
  ...
  }
```

- Проверить конфигурацию `sudo nginx -t` и перезагрузить её командой `sudo systemctl reload nginx`, чтобы изменения вступили в силу.

## Получение и настройка SSL-сертификата
-------------

### Установка certbot

- Зайдите на сервер и последовательно выполните команды:

```
 sudo apt install snapd
 sudo snap install core; sudo snap refresh core
 sudo snap install --classic certbot
 sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

- Запустить certbot и получить SSL-сертификат:

```
sudo certbot --nginx
```

- Cертификат автоматически сохранится на вашем сервере в системной директории /etc/ssl/ Также будет автоматически изменена конфигурация Nginx: в файл /etc/nginx/sites-enabled/default добавятся новые настройки и будут прописаны пути к сертификату.

- Перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`

## Автор проекта
-------------

_Богатырев Роман_


> ***_Примечание:_*** _проект использует MIT License._

_____
_____

_eng_

# Kittygram - educational project. Minimal social network for posting photos of cats.

-------------

## Description of the project
-------------

This project was created as part of a training course, but it supports all the necessary minimum functionality. The main objective of this project is to learn how to work with the code as a ready-made base for deployment and publish it as a finished project for industrial use.

## Technologies
-------------

- Python 3.9
- Django==3.2.3
- djangorestframework==3.12.4
- nginx
- gunicorn


## Installing a project on a local computer from a network repository
-------------


- Clone the repository `git clone <address of your repository>`
- Go to the directory with the cloned repository
- Install virtual environment `python3 -m venv venv`
- Install dependencies `pip install -r requirements.txt`
- In the directory /backend/kittygram_backend/ create a .env file
- In the .env file, write your SECRET_KEY in the form: `SECRET_KEY = '<your_key>'`

# Deploy the project to a remote server
-------------

## Connecting the server to a GitHub account
-------------

- Git must be installed on the server. To check, run `git --version`
- If Git is not installed, install with the command `sudo apt install git`
- While on the server, generate a pair of SSH keys with the `ssh-keygen` command
- Save the public key in your GitHub account. To do this, display the key in the terminal with the command `cat .ssh/id_rsa.pub`. Copy the key from the ssh-rsa characters, inclusive, to the end. Add this key to your GitHub account.
- Clone the project from GitHub to the server: `git clone git@github.com:your_account/<project name>.git`

## Run the backend project on the server
-------------

- Install the package manager and virtual environment creation utility `sudo apt install python3-pip python3-venv -y`
- Being in the project directory, create and activate the virtual environment `python3 -m venv venv && source venv/bin/activate`
- Install dependencies `pip install -r requirements.txt`
- Run `python3 manage.py migrate` migrations
- Create superuser `python3 manage.py createsuperuser`
- Edit settings.py on the server: add the external IP address of your server and the addresses 127.0.0.1 and localhost to the ALLOWED_HOSTS list. ALLOWED_HOSTS = ['<your server's external address>', '127.0.0.1', 'localhost']

## Run the frontend project on the server
-------------

- Install `Node.js` on the server with `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\ sudo apt-get install -y nodejs`
- Install frontend application dependencies. From the directory `<your project>/frontend/` run the command: `npm i`

## Installing and running Gunicorn
-------------

- With the project virtual environment activated, install the gunicorn package `pip install gunicorn==20.1.0`

- open the project's settings.py file and set the `DEBUG` constant to `False` `DEBUG = False`

- In the /etc/systemd/system/ directory, create the file _gunicorn.service_ `sudo nano /etc/systemd/system/gunicorn.service` with the following code (without comments):

```
  [Unit]

  Description=kittygram_gunicorn daemon

  After=network.target

  [Service]

  User=yc-user

  WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/

  ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

  [Install]

  WantedBy=multi-user.target
```

To find out exactly the path to Gunicorn, you can use the `which gunicorn` command when the virtual environment is activated.

## Installing and configuring Nginx
-------------

- On the server from any directory, run the command: `sudo apt install nginx -y`
- To set restrictions on open ports, execute the commands in turn: `sudo ufw allow 'Nginx Full'` `sudo ufw allow OpenSSH`
- Enable firewall `sudo ufw enable`
- Collect and host frontend application statics.
- Go to the <project_name>/frontend/ directory and execute the command `npm run build` The result will be saved in the .../frontend/build/ directory. Copy the contents of the /frontend/build/ folder to the server system directory /var/www/
- Open the web server configuration file `sudo nano /etc/nginx/sites-enabled/default` and replace its contents with the following code:

```
server {

      listen 80;
      server_name публичный_ip_вашего_удаленного_сервера;
  
      location / {
      root   /var/www/<имя проекта>;
      index  index.html index.htm;
      try_files $uri /index.html;
      }
  }
```

- Check if `sudo nginx -t` configuration is correct
- Reload Nginx configuration `sudo systemctl reload nginx`
- Configure request proxying
- Open the Nginx configuration file /etc/nginx/sites-enabled/default and add another `location` block to it

```
server {

      listen 80;
      server_name публичный_ip_вашего_удаленного_сервера;

      location /api/ {
          proxy_pass http://127.0.0.1:8080;
      }
      
      location /admin/ {
  	    proxy_pass http://127.0.0.1:8000;
  		}
  	
      location / {
          root   /var/www/<имя_проекта>;
          index  index.html index.htm;
          try_files $uri /index.html;
      }
  }
```

- Save changes, check and reload web server configuration:

```
   sudo nginx -t
   sudo systemctl reload nginx
```

- Collect and configure statics for the backend application.
- In the settings.py file, write the settings:

```
  STATIC_URL = 'static_backend'
  STATIC_ROOT = BASE_DIR / 'static_backend'
```

- Activate the virtual environment of the project, go to the directory with the manage.py file and run the command `python3 manage.py collectstatic`
- The static_backend/ directory will be created in the _<project_name>/backend/_ directory
- Copy the static_backend/ directory to the /var/www/<project_name>/ directory


## Adding a domain name to Django settings
-------------

- In the settings.py file, add a domain name to the `ALLOWED_HOSTS` list: ALLOWED_HOSTS = ['ip_your_server_address', '127.0.0.1', 'localhost', 'your-domain']
- Save changes and restart gunicorn `sudo systemctl restart gunicorn`
- Make changes to the Nginx configuration. Open the configuration file with the command: `sudo nano /etc/nginx/sites-enabled/default`
- Add the given domain name to the `server_name` line (separated by a space, without `< >`):

```
server {
   ...
       server_name <your-ip> <your-domain>;
   ...
   }
```

- Check the `sudo nginx -t` configuration and reload it with `sudo systemctl reload nginx` for the changes to take effect.

## Obtaining and configuring an SSL certificate
-------------

### Install certbot

- Log in to the server and execute the following commands in sequence:

```
  sudo apt install snapd
  sudo snap install core; sudo snap refresh core
  sudo snap install --classic certbot
  sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

- Run certbot and get an SSL certificate:

```
sudo certbot --nginx
```

- The certificate will be automatically saved on your server in the /etc/ssl/ system directory. Also, the Nginx configuration will be automatically changed: new settings will be added to the /etc/nginx/sites-enabled/default file and paths to the certificate will be added.

- Reload Nginx configuration `sudo systemctl reload nginx`

## Project author
-------------

_Bogatyrev Roman_


> ***_Note:_*** _The project uses the MIT License._