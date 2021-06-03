---
title: "Cómo desplegar un proyecto de Django con uWSGI + nginx + supervisor"
date: 2017-01-16 17:39:09 -0500
excerpt_separator: <!-- more -->
comments: true
categories:
- Python
---

Suponemos que tenemos [un proyecto](https://github.com/vero4karu/sitp_scraper) de Django y queremos desplegarlo en un servidor con un sistema operativo Linux.

La idea es que nuestro servidor web va a usar una interfaz (WSGI) para "hablar" con nuestra aplicación de Django. Esa interfaz va a correr la aplicación, pasarle las peticiones de usuario y devolver la respuesta. WSGI (Web Server Gateway Interface) es un estándar y uWSGI es una de sus implementaciones que vamos a usar:

    Usuario <-> Servidow web <-> Socket <-> uWSGI <-> Django

<!-- more -->

Decimos que el código del proyecto está en la carpeta `/home/deploy/sitp_scraper`. Primero creamos el entorno virtual e instalamos las librerías de Python proyecto (en éste ejemplo usamos [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)):

    mkvirtualenv stip_scraper
    pip install -r requitements.txt

Instalamos uWSGI:

    pip install uwsgi

Verificamos si podemos correr nuestro proyecto si problema:

    ./manage.py runserver 0.0.0.0:8000

Ahora probamos correrlo usando uWSGI:

    uwsgi --http :8000 --module stip_scraper.wsgi

Instalamos [nginx](https://nginx.org/en/):

    sudo apt-get install nginx
    sudo /etc/init.d/nginx start 

Si al visitar `nuestroservidor.com:80` vemos el mensaje "Welcome to nginx", podemos concluir que nginx funciona de la forma correcta.

Configuramos nginx `sitp_scraper_nginx.conf`:

```nginx
upstream django {
    server unix:///tmp/sitp_scraper.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name  nuestroservidor.com;  # Nuestro dominio o dirección IP
    charset      utf-8;

    client_max_body_size 10M;  # Tamaño máximo de archivos que podrémos subir al servidor

    location /media  {
        # MEDIA_ROOT de nuestro proyecto de Django
        alias /home/deploy/sitp_scraper/media;
    }

    location /static {
        # STATIC_ROOT de nuestro proyecto de Django
        alias /home/deploy/sitp_scraper/static;

    }

    location / {
        uwsgi_pass  django;
        include     /home/deploy/sitp_scraper/uwsgi_params;
    }
}
```

El archivo `uwsgi_params` se puede descargar aquí: [github.com/nginx/nginx/blob/master/conf/uwsgi_params](https://github.com/nginx/nginx/blob/master/conf/uwsgi_params)

Creamos en la carpeta `/etc/nginx/sites-enabled` un enlace hacía nuestra configuración para que nginx pueda verla:

    sudo ln -s ~/path/to/your/sitp_scraper_nginx.conf /etc/nginx/sites-enabled/

Verificamos si nuestra configuración está bien:

    nginx -t

y reiniciamos nginx:

    sudo /etc/init.d/nginx restart

Ahora creamos un archivo `sitp_scraper_uwsgi.ini` para guardar las opciones que vamos a pasar a uWSGI:

```
[uwsgi]
# Django-related settings
chdir           = /home/deploy/sitp_scraper
module          = sitp_scraper.wsgi
home            = /home/deploy/.virtualenvs/sitp_scraper

# Process-related settings
master          = true
processes       = 10

# The socket (use the full path to be safe)
socket          = /tmp/sitp_scraper.sock
chmod-socket    = 664
uid             = www-data
gid             = www-data

# Clear environment on exit
vacuum          = true
```

Agregamos nuestro usuario de Linux al grupo `www-data` (para no tener problemas con permisos) y verificamos que todo funcione bien:

    uwsgi --ini sitp_scraper_uwsgi.ini

Podemos ver que se creó un archvo `/tmp/sitp_scraper.sock`:
    
    ll /tmp/sitp_scraper.sock
    srw-rw-r-- 1 www-data www-data 0 Jan 15 20:46 /tmp/sitp_scraper.sock


Ahora instalamos supervisor:

    apt-get install supervisor
    service supervisor start

y creamos un archivo de configuración `/etc/supervisor/conf.d/sitp_scraper.conf`

```
[program:uwsgi]
user            = www-data
command = /home/deploy/.virtualenvs/sitp_scraper/bin/uwsgi --ini=/home/deploy/sitp_scraper/sitp_scraper_uwsgi.ini
autostart       = true
autorestart     = true
stderr_logfile  = /tmp/uwsgi_err.log
stdout_logfile  = /tmp/uwsgi.log
stopsignal=INT
```

Reiniciamos supervisor:

    $ supervisorctl  
    uwsgi                            RUNNING   pid 2762, uptime 1 day, 2:39:40
    supervisor> restart uwsgi

Voilà: `nuestroservidor.com:80`