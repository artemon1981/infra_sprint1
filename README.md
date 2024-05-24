# Kittygram - блог для размещение фотографий котиков.

### Описание проекта:

Проект Kittygram даёт возможность пользователям поделиться и похвастаться фотографиями своих любимымих котиков. Зарегистрированные пользователи могут создавать, просматривать, редактировать и удалять свои записи.

### Технологии и необходимые ниструменты:

- Python 3.x
- Node.js 9.x.x
- Git
- Nginx
- Gunicorn
- Django (backend)
- React (frontend)


### Установка проекта:

#### Установка проекта на локальный компьютер из репозитория 
 - Клонировать репозиторий `git clone <адрес вашего репозитория>`
 - перейти в директорию с клонированным репозиторием
 - установить виртуальное окружение `python3 -m venv venv`
 - установить зависимости `pip install -r requirements.txt`
 - в директории /backend/kittygram_backend/ создать файл .env
 - в файле .env прописать ваш SECRET_KEY в виде: `SECRET_KEY = '<ваш_ключ>' и `ALLOWED_HOSTS = '['localhost']'`

### Деплой проекта на удаленный сервер

#### Подключение сервера к аккаунту на GitHub
- на сервере должен быть установлен Git. Для проверки выполнить `sudo apt update` `git --version`
- если Git не установлен - установить командой `sudo apt install git`
- находясь на сервере сгенерировать пару SSH-ключей командой `ssh-keygen`
- сохранить открытый ключ в вашем аккаунте на GitHub. Для этого вывести ключ в терминал командой `cat .ssh/id_rsa.pub`. Скопировать ключ от символов ssh-rsa, включительно, и до конца. Добавить это ключ к вашему аккаунту на GitHub.
- клонировать проект с GitHub на сервер: `git clone git@github.com:Ваш_аккаунт/<Имя проекта>.git`

#### Запуск backend проекта на сервере
- Установить пакетный менеджер и утилиту для создания виртуального окружения `sudo apt install python3-pip python3-venv -y`
- находясь в директории с проектом создать и активировать виртуальное окружение `python3 -m venv venv`  `source venv/bin/activate` 
- установить зависимости `pip install -r requirements.txt`
- выполнить миграции `python manage.py migrate`
- создать суперюзера `python manage.py createsuperuser`
- отредактировать .env на сервере: в список ALLOWED_HOSTS добавить внешний IP-адрес вашего сервера и адреса `127.0.0.1` и `localhost` . ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']

#### Запуск frontend проекта на сервере
- установить на сервер `Node.js`   командами
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\`
`sudo apt-get install -y nodejs`
- установить зависимости frontend приложения. Из директории `<ваш проект>/frontend/` выполнить команду: `npm i`

#### Установка и запуск Gunicorn
- при активированном виртуальном окружении проекта установить пакет gunicorn `pip install gunicorn==20.1.0`
- открыть файл settings.py проекта и установить для константы `DEBUG` значение `False` `DEBUG = False`
- В директории _/etc/systemd/system/_ создайте файл _gunicorn.service_ `sudo nano /etc/systemd/system/gunicorn.service`  со следующим кодом (без комментариев):

	    [Unit]
    
	    Description=gunicorn daemon
    
	    After=network.target
    
	    [Service]
    
	    User=yc-user
    
	    WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/
    
	    ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    
	    [Install]
    
	    WantedBy=multi-user.target

Чтобы точно узнать путь до Gunicorn можно при активированном виртуальном окружении использовать команду `which gunicorn`

#### Установка и настройка Nginx

 - На сервере из любой директории выполнить команду: `sudo apt install nginx -y`
- Для установки ограничений на открытые порты выполнить по очереди команды: `sudo ufw allow 'Nginx Full'`  `sudo ufw allow OpenSSH`
- включить файервол `sudo ufw enable`
	#### собрать и разместить статику frontend-приложения.
- Перейти в директорию _<имя_проекта>/frontend/_  и выполнить команду `npm run build` Результат сохранится в директории ..._/frontend/build/_.  В системную директорию сервера _/var/www/_ скопировать содержимое папки _/frontend/build/_
- открыть файл конфигурации веб-сервера `sudo nano /etc/nginx/sites-enabled/default` и заменить его содержимое следующим кодом:

    	
        server {
    
	        listen 80;
	        server_name публичный_ip_вашего_удаленного_сервера;
            server_tokens off;
        
	        location / {
            root   /var/www/<имя проекта>;
            index  index.html index.htm;
            try_files $uri /index.html;
	        }
    
	    }
- проверить корректность конфигурации `sudo nginx -t`
- перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`
	#### настроить проксирование запросов
- Открыть файл конфигурации Nginx _/etc/nginx/sites-enabled/default_ и добавить в него ещё один блок `location`

	    server {
    
	        listen 80;
	        server_name публичный_ip_вашего_удаленного_сервера;
            server_tokens off;
    
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

- Сохранить изменения, проверить и перезагрузить конфигурацию веб-сервера:

	    sudo nginx -t
	    sudo systemctl reload nginx

	#### собрать и настроить статику для backend-приложения.
- в файле _settings.py_ прописать настройки 
	

	    STATIC_URL = 'static_backend'
	    STATIC_ROOT = BASE_DIR / 'static_backend'

- активировать виртуальное окружение проекта, перейти в директорию с файлом _manage.py_ и выполнить команду `python manage.py collectstatic`
- в директории_<имя_проекта>/backend/_ будет создана директория _static_backend/_ 
- Скопировать директорию _static_backend/_ в директорию _/var/www/<имя_проекта>/_

#### Добавление доменного имени в настройки Django
- в файле .env добавить в список `ALLOWED_HOSTS` доменное имя: 
	ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен'] 
- сохранить изменения и перезапустить gunicorn `sudo systemctl restart gunicorn`
- внести изменения в конфигурацию Nginx. Открыть конфигурационный файл командой: `sudo nano /etc/nginx/sites-enabled/default`
- Добавьте в строку `server_name` выданное вам доменное имя (через пробел, без `< >`):

		server {
		...
		    server_name <ваш-ip> <ваш-домен>;
		...
		}
- Проверить конфигурацию `sudo nginx -t` и перезагрузить её командой `sudo systemctl reload nginx`, чтобы изменения вступили в силу.

### Автор

- [Дмитрий Бойцев](https://github.com/Dmitriy-boytsev)
