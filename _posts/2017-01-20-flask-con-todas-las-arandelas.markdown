---
title: 'Flask con todas las arandelas'
date: 2017-01-20 09:18:28 -0500
featured_image: '/images/demo/demo-square.jpg'
comments: true
categories: 
- Flask
- Python
- SQLAlchemy
---

## Para empezar

Hay muchos recursos acerca de Flask en los internetes, 
pero aquí me gustaría mencionar los tres que considero fundamentales.

El primero, por supuesto, es la documentación oficial del framework: 
[flask.pocoo.org/docs](http://flask.pocoo.org/docs/).

Luego, es muy bueno mirar el libro de Miguel Grinberg: 
[flaskbook.com](https://flaskbook.com/), en donde se explica de forma 
bastante detallada cómo hacer una plataforma de publicación de entradas de blog. 
El código, que sirve como ejemplo en el libro, se puede encontrar en 
la página de github del autor. Como material complementario al libro, 
es muy recomendado echar una mirada en el 
[blog personal de Miguel Grinberg](https://blog.miguelgrinberg.com/), 
donde el autor cubre los temas más específicos del desarrollo en Flask.

Y ahora, como dijo el astronauta ruso Yuri Gagarin en el momento del 
despegue de su nave Vostok 1: "¡Poyejali!"" 
(en ruso: Поехали!; se traduce como «¡Vámonos!»).

<iframe src="//slides.com/cansadadeserfeliz/flask-con-todas-las-arandelas/embed?style=light" width="576" height="420" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

## Aplicación básica

Empezamos con instalar **Flask**:

```bash
$ mkvirtualenv pycon2017 -p python3
$ pip install Flask
```

Al momento de ésta presentación la última versión del framework es **0.12**.

Ahora creamos nuestra primera aplicación de Flask que va a ser sólo un archivo de Python:

`app.py`:

```python
from flask import Flask
app = Flask(__name__)


@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

El decorador ``route`` nos ayuda a definir la ruta de URL para la vista 
llamada ``hello`` que simplemente devuelve al navegador la respuesta HTTP 200 
con la frase "Hello World!". 

Corremos la aplicación con el siguiente comando:

```bash
$ python app.py 
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Y ahora lo podemos abrir en nuestro navegador:

![](/images/flask/flask_hello_world.png)

## manage.py

Nosotros, por experiencia con otros frameworks, ya nos acostumbramos a 
la posibilidad de acceder al shell de Python de nuestra aplicación, o con 
poder correr comandos, que pertenecen a la aplicación, desde la consola. 
Para tener todo esto en Flask, instalamos un librería llamada `Flask-Script`.

```bash
$ pip install Flask-Script
```

Ahora separamos nuestra aplicación en dos archivos: en el primero vamos iniciar 
la app de Flask, en la que especificamos la ruta de los archivos estáticos y 
del archivo de configuración (equivalente de `settings.py` en Django)

`run.py`

```python
from flask import Flask

def create_app():
    app = Flask(__name__, 
        static_url_path='/static')
    app.config.from_object('conf.config')
    return app
```

y en el segundo vamos a colocar una instancia de **Manager**, que se encaragará 
de correr la aplicación con `python manage.py`:

`manage.py`

```python
from flask.ext.script import Manager
from run import create_app

app = create_app()
manager = Manager(app)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == '__main__':
    manager.run()
```

Corriendo el siguiente comando en la consola

```bash
$ ./manage.py runserver -h HOST -p PORT
```

podemos ver la misma aplicación, como en el paso anterior, 
pero ahora podemos además acceder a shell usando el comando

```bash
$ ./manage.py shell
In [1]:
```

## Configuración

Miremos con más atención la estructura del proyecto:

```
├── conf
│   ├── config.py
│   ├── __init__.py
│   └── local_config.py
├── manage.py
└── run.py
```

Ahora tenemos una carpeta con los archivos de configuración - `conf`:

`conf/config.py`

```python
import sys

DEBUG = False

try:
    if 'test' in sys.argv:
        from test_config import *
    else:
        from local_config import *
except ImportError:
    pass
```

Más información acerca de la configuración en Flask se puede encontrar 
en la página [flask.pocoo.org/docs/0.12/config/](http://flask.pocoo.org/docs/0.12/config/).

## Blueprints

Cuando nuestro archivo con las vistas crezca, vamos a querer a separar 
la lógica de nuestra aplicación en módulos diferentes 
(así como lo tenemos con apps de Django):

```
├── conf
│   ├── config.py
│   ├── __init__.py
│   └── local_config.py
├── app1
│   ├── templates
│   └── views.py
├── app2
│   ├── templates
│   └── views.py
├── manage.py
└── run.py
```

En este caso nos ayuda un concepto de crear aplicaciones llamado **Blueprints**. 
La información completa acerca de Blueprints se puede encontrar en la documentación 
de Flask: [flask.pocoo.org/docs/0.12/blueprints/](http://flask.pocoo.org/docs/0.12/blueprints/). 

Creamos un Blueprint para nuestra aplicación, donde vamos a guardar las vistas 
relacionadas a nuestros usuarios, por ejemplo:

`users/views.py`

```python
from flask import Blueprint

users = Blueprint(
    'users',
    __name__,
    template_folder='templates')

@users.route('/')
def hello():
    return 'Hello Worlds!'
```

y lo registramos en el archivo `run.py`:

`run.py`

```python
from flask import Flask


def create_app():
    app = Flask(__name__,
        static_url_path='/static')
    app.config.from_object('conf.config')

    from users.views import users
    app.register_blueprint(users, url_prefix='/users')

    return app
```

indicando en el argumento `url_prefix` del método `register_blueprint`, 
que todos los URLs, relacionados con el Blueprint de usuario, van a 
empezar con el prefijo `/users`, por ejemplo: 
[http://localhost:5000/users/](http://localhost:5000/users/).

## URLs

Miremos cómo se definen los URLs en Flask.

Primero, hay formas diferentes de especificar la ruta: usando el decorador de Blueprint `@users.route` o el método `users.add_url_rule`:

```python
@users.route('/user/<name>')
def user_profile(name):
    return '<h1>Hello, {}!</h1>'.format(name)

def users_list():
    pass
users.add_url_rule('/', 'users_list', users_list)
```

En el primer ejemplo podemos ver, que a la vista se pasa el parámetro de URL `name`, 
al que luego accedemos desde el parámetro `name` de la vista `user_profile`.

Flaks permite especificar el tipo de parámetro que esperamos. Por ejemplo, para 
los números enteros, el decorador de escribe así:

```python
@users.route('/user/<int:pk>')
def user_detail(pk):
    return '<h1>Hello, user #{}!</h1>'.format(pk)
```

También se puede explícitamente indicar el método HTTP, que acepta la vista.

```python
@users.route('/user/<int:pk>/create', methods=('POST',))
def user_create(pk):
    pass
```

En este caso al tratar de acceder a la vista con el método GET, obtenemos 
el error 405 (Method not allowed).

Para obtener el URL, que corresponde a la vista usamos el método `url_for`, 
al que pasamos el nombre del Blueprint y el nombre de la vista:

```python
from flask import url_for

url_for('users.user_detail', pk=user_pk)
```

## Vistas

#### Request

En Flask el objeto `request` no se está pasando como parámetro al la vista (como en Django, por ejemplo), sino que es una variable global:

```
GET http://localhost:5000/users/user/1?q=foo
```

`users/views.py`

```python
from flask import Blueprint, request

users = Blueprint('users', __name__, template_folder='templates')


@users.route('/user/<int:pk>')
def user_detail(pk):
    print(request)
    print(request.method)  # GET
    print(request.args)    # ImmutableMultiDict([('q', u'foo')])
    print(request.headers)
    return '<h1>Hello, user #{}!</h1>'.format(pk)
```

#### Decoradores

Aparte de la definición de ruta, la vista en Flask puede tener otros 
decoradores, por ejemplo:

`users/views.py`

```python
@users.route('/user/<int:pk>')
@login_required
def user_detail(pk):
    return '<h1>Hello, user #{}!</h1>'.format(pk)
```

## Plantillas

Ahora vamos a renderizar una plantilla desde nuestra vista. Para eso 
vamos a usar el método `render_template` que recibe como argumentos 
la ruta hasta la plantilla dentro de la carpeta que especificamos en 
el parámetro `template_folder` del Blueprint, y los demás argumentos 
que son los parámetros de contexto.

`users/views.py`

```python
from flask import Blueprint
from flask import render_template

users = Blueprint('users', __name__, template_folder='templates')


@users.route('/user/<int:pk>')
def user_detail(pk):
    return render_template('users/detail.html', user_id=pk)
```

`users/templates/users/detail.html`

```html
<html>
  <body>
    <h1>Hello, user #{{ user_id }}!</h1>
  </body>
</html>
```

### Jinja2

Para renderizar plantillas Flask usa el lenguaje `Jinja2`. 
Vamos a mirar algunas de sus funcionalidades:

#### Variables:

```html{% raw %}
{{ foo.bar }}
{{ foo['bar'] }}
{% endraw %}```

#### Condicionales:

```html{% raw %}
{% if user.address %}
    <p>{{ user.address }}</p>
{% endif %}
{% endraw %}```

##### Bucles:

```html{% raw %}
{% for user in users %}
  <li>{{ user.name }}</li>
{% else %}
  <li>No hay usuarios</li>
{% endfor %}
{% endraw %}```

##### Bloques:

```html{% raw %}
{% block title %}Usuarios{% endblock %}
{% endraw %}```

##### Extender una plantilla base:

```html{% raw %}
{% extends "layouts/main.html" %}
{% endraw %}```

#### Comentario:

```html{% raw %}
{# Texto #}
{% endraw %}```

##### Incluir otra plantilla:

```html{% raw %}
{% with rating=user.rating %}
    {% include 'includes/_rating.html' %}
{% endwith %}
{% endraw %}```

##### Asignar un valor a una variable local:

```html{% raw %}
{% set permissions = user.get_permission() %}
{{ permissions }}
{% endraw %}```

##### Filtros:

```html{% raw %}
{{ user.address|default('N/A') }}
{% endraw %}```

## Procesadores de contexto

De forma predeterminada, el contexto de todas las plantillas ya tiene 
las siguientes variables:

* `config` - Objeto de configuración (`flask.config`)

```html{% raw %}
{{ config.DEBUG }}
{% endraw %}```

* `request` - Objeto de la petición actual (`flask.request`)

```html
{% raw %}
{{ request.path }}
{% endraw %}```

* `session` - Objeto de sesión (`flask.session`)
* `g` - Variables globales

Hay una forma de tener `context_processors` (como en Django) para poder 
pasar variables a todas las plantillas del proyecto:

`run.py`

```python
from flask import Flask

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')

    from users.views import users
    app.register_blueprint(users, url_prefix='/users')

    @app.context_processor
    def constants_processor():
        return {
            'say_hello': 'Hola',
        }

    return app
```

Ahora podemos acceder a la variable `say_hello` desde todas las plantillas 
sin tener que pasarla cada vez explícitamente:

`users/templates/users/detail.html`

```html{% raw %}
<html>
  <body>
    <h1>{{ say_hello }}, user #{{ user_id }}!</h1>
  </body>
</html>
{% endraw %}```

## Modelos

### SQLAlchemy

Hay dos librerías que nos permiten trabajar con modelos y 
hacer peticiones SQL desde Flask.

```bash
$ pip install SQLAlchemy
$ pip install Flask-SQLAlchemy
```

Una es `SQLAlchemy` y la otra es su extensión para Flask - `Flask-SQLAlchemy`, 
que viene con algunas funcionalidades adicionales y útiles en el desarrollo web, 
por ejemplo, el método `first_or_404()` para obtener el primer elemento 
del query o error HTTP 404 si no existe, o el método `paginate()` para realizar 
paginación sobre los objetos de `BaseQuery`.

Ahora incluimos `SQLAlchemy` como una aplicación externa de nuestro proyecto. 
Primero creamos una variable `db` para que el import no se rompa cuando 
la aplicación todavía no se ha inicializado, y en `create_app` cuando inicializamos 
la aplicación: `db.init_app(app)`.

`run.py`

```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')

    db.init_app(app)
    return app
```

En el archivo de configuración colocamos la ruta hacia nuestra base de datos.

`conf/config.py`

```python
SQLALCHEMY_DATABASE_URI = 'sqlite:////tmp/test.db'
```

Y ahora podemos crear los modelos. Aquí en el ejemplo se puede ver 
cómo definir modelos, columnas de tipos diferentes y crear llaves foráneas

```python
from run import db


class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)
    city_id = db.Column(db.ForeignKey(u'cities.id'), nullable=False, index=True)
    created_at = db.Column(db.DateTime, nullable=False, default=db.func.now())
    updated_at = db.Column(db.DateTime, nullable=False, default=db.func.now(),
                        onupdate=db.func.now())

    def __repr__(self):
        return '<Usuario %r>' % self.username


class City(db.Model):
    __tablename__ = 'cities'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255))

    users = db.relationship('User', backref='city', lazy='dynamic')
```

### CRUD

Miramos los métodos de CRUD básicos que nos ofrece el ORM de SQLAlchemy.

##### Crear

```python
user = User(email='john@example.com', username='')
db.session.add(user)
db.session.commit()
```

##### Obtener todos los usuarios

```python
users = User.query.all()  # [<Usuario john'>, <Usuario u'admin'>]
```

##### Obtener el primer usuario

```python
john = User.query.first()  # <Usuario 'john'>
```

##### Filtrar usuarios y ordenar

```python
User.query.filter_by(username='john').all()
User.query.filter(
    User.created_at >= (datetime.datetime.utcnow() - datetime.timedelta(days=3)
).order_by(User.created_at.desc()).limit(10).all()
```

##### Borrar

```python
db.session.delete(user)
db.session.commit()
```

`BaseQuery` no tiene método `save()`, sino todos los cambios que hacemos 
a objetos de modelos se agregan a la sesión. Como la sesión sólo vive durante 
la petición, para que nuestos cambios sean enviados a la base de datos, 
hace falta llamar el método `db.session.commit()`.

## Formularios

Existen varias librerías de Python que nos permiten trabajar con formularios. 
En este ejemplo vamos a mirar una llamada `Flask-WTF`.

```bash
$ pip install Flask-WTF
$ pip install Flask-Bootstrap
```

Vamos a usarla junto con una librería complementaria - `Flask-Bootstrap` - que 
lo único que hace es generar el código de HTML del formulario con la estructura y 
las clases de [Twitter Bootstrap](http://getbootstrap.com/2.3.2/) 
(lo mismo que hace `django-crispy-forms`).

`users/forms.py`

```python
import wtforms
from flask_wtf import Form

class UserForm(Form):
    email = wtforms.StringField(
        validators=[
            wtforms.validators.Email(),
            wtforms.validators.DataRequired(),
        ],
    )
    username = wtforms.StringField(
        validators=[wtforms.validators.DataRequired()],
    )
    submit = wtforms.SubmitField('Save')
```

Ahora podemos renderizar el formulario en nuestra plantilla:

`users/templates/users/form.html`

```html{% raw %}
{% extends "layouts/main_layout.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block content %}
  {{ wtf.quick_form(form) }}
{% endblock %}
{% endraw %}```

Este código nos va a renderizar el formulario completo con todos 
los campos especificados. Si queremos renderizar sólo algunos campos específicos, 
podemos escribirlo de la siguiente forma:

`users/templates/users/form.html`

```html{% raw %}
{{ wtf.form_field(form.email) }}
{{ wtf.form_field(form.submit) }}
{% endraw %}```

Lo único que hace falta es escribir una vista que reciba los datos del 
formulario y los guarde en la base de datos, por ejemplo.

El diccionario con los valores para cada campo se encuentra en 
la variable `request.form`. Si el request tiene método POST, validamos 
el formulario `form.validate()`, y en el caso exitoso, pasamos los valores 
del formulario a nuestro objeto: `form.populate_obj(user)`.

`users/views.py`

```python
@users.route('/update/<int:pk>/', methods=('GET', 'POST'))
def user_update(pk):
    user = User.query.filter_by(id=pk).first_or_404()

    form = UserForm(user, request.form)

    if request.method == 'POST' and form.validate():
        form.populate_obj(user)

        db.session.add(user)
        db.session.commit()

        flash('Usuario fue editado exitosamente', 'success')
        return redirect(url_for('users.user_detail', pk=user.id))

    return render_template(
        'users/form.html',
        form=form,
    )
```

## Autenticación

Para agregar autenticación al proyecto de Flask, normalmente usan las 
dos siguientes librerías:

```bash
$ pip install Flask-Login
$ pip install Flask-OAuth
```

La primera tiene toda la funcionalidad de ingreso, salida de usuario. 
La podemos instalar de la misma forma, que hicimos con SQLAlchemy hace poco:

`run.py`

```python
from flask_login import LoginManager

login_manager = LoginManager()
login_manager.login_view = 'users.login'

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')
    app.permanent_session_lifetime = datetime.timedelta(days=365)

    login_manager.init_app(app)
```

En `login_manager.login_view` especificamos qué vista corresponde al ingreso (login), 
y en `permanent_session_lifetime`  se puede indicar el tiempo en el qué durará 
activa la sesión.

El modelo que vamos a usar para usuarios debe heredar de `UserMixin` de `flask_logins`

`users/models.py`

```python
from flask_login import UserMixin
from run import db

class User(db.Model, UserMixin):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    # ...
```

Y ahora usando la otra librería - `Flask-OAuth` - podemos hacer una vista para que nuestros usuarios puedan ingresar desde su cuenta de Google, por ejemplo:

`users/views.py`

```python
@users.route('/login')
def login():
    callback = url_for('users.authorized', _external=True)
    return google.authorize(callback=callback)
```

Cuando el usuario ya está autenticado, en cualquier punto de nuestra 
aplicación podemos preguntar por el objeto correspondiente: 

```python
from flask_login import current_user
```

En la siguiente dirección se puede encontrar más información acerca de cómo 
configurar los tokens de autenticación Google: 
[pythonhosted.org/Flask-OAuth/](https://pythonhosted.org/Flask-OAuth/).

## Múltiples idiomas

Si queremos tener soporte de múltiples idiomas, tendremos que instalar otra librería:

```bash
$ pip install Flask-Babel
```

Miremos qué comandos nos ofrece:

```bash
# Extraer los textos para traducción 
# (se corre sólo una vez al principio)
$ pybabel extract -F babel.cfg -o messages.pot .


# Generar un catálogo para español
$ pybabel init -i messages.pot -d translations -l es

# Se crea el directorio translations/es
# Por dentro hay otro directorio llamado LC_MESSAGES que tiene  
# un archivo messages.po. 

# Después de traducir los textos y guardarlos en messages.po,
# compilamos el archivo y publicamos los textos:

$ pybabel compile -d translations

# Para actualizar las traducciones a diario:

$ pybabel extract -F babel.cfg -o messages.pot .
$ pybabel update -i messages.pot -d translations
```

Si queremos traducir cadenas de texto dentro del código de Python, 
habrá que usar la siguiente sintaxis:

```python
from flask_babel import gettext as _

_('Invalid authentication token')
```

Y en plantillas se ve muy parecido:

```html{% raw %}
<button class="btn btn-default" title="{{ _('Help') }}">
    <i class="fa fa-question"></i>
</button>
{% endraw %}```

## Mensajes

Los que están familiarizados con el framework Django, seguramente 
recuerdan que Django tiene un procesador de contexto `messages` que 
nos permite mandar mensajes a las plantillas desde el código Python:

```python
from django.contrib import messages

messages.add_message(request, messages.INFO, 'Hello world.')
```

En Flask es muy parecido, sólo que esos mensajes se llaman `flash`:

```python
from flask import flash

flash('Hello world.', 'success')
```

y para consultarlos dentro de la plantilla llamamos el método `get_flashed_messages`:

```html{% raw %}
{% for category, message in get_flashed_messages(with_categories=true) %}
<div class="alert alert-{{ category|replace('message', 'info') }}">
  <button type="button" class="close" data-dismiss="alert">×</button>
  {{ message }}
</div>
{% endfor %}
{% endraw %}```

## Caché

Y para terminar la presentación, vamos a meter todo en caché #comonosgusta:

```bash
$ pip install Flask-Cache
```

Ya conocemos cómo instalar las aplicaciones externas en Flask, pero repasémoslo:

`run.py`

```python
from flask_cache import Cache


cache = Cache()

def create_app():
    app = Flask(__name__, static_url_path='/static')
    app.config.from_object('conf.config')

    cache.init_app(app)
```

Y ahora sí, vamos con toda... ponemos en caché una vista completa:

`users/views.py`

```python
from run import cache

@users.route('/list')
@cache.cached(timeout=60*60, key_prefix='user_list')
def user_list():
    pass
```

y luego un fragmento de html:

```html{% raw %}
{% cache 60*60, 'dashboard_menu_user' + current_user.id|string %}
  {% include 'layouts/_menu.html' %}
{% endcache %}
{% endraw %}```

## pip freeze

Y ahora vamos resumir qué librerías hemos instalado y cuales otras nos podrían ser útiles en el futuro:

```{% raw %}
# Framework
Flask==0.11

# Libs
celery==3.1.20
coverage==4.0.3
factory-boy==2.6.1
SQLAlchemy==1.0.12
SQLAlchemy-Utils==0.31.6

# Flask libs
Flask-And-Redis==0.6
Flask-Babel==0.9               # Múltiples idiomas y zonas horarias
Flask-Bootstrap==3.3.5.7       # ~ django-crispy-forms para Bootstrap
Flask-Cache==0.13.1
Flask-DebugToolbar==0.10.0     # En Django: django-debug-toolbar
Flask-fillin==0.2              # Diligenciar formularios en pruebas
Flask-Login==0.3.2
Flask-Migrate==1.8.0
Flask-Moment==0.5.1            # Integración con moment.js
Flask-OAuth==0.12              # ~ python-social-auth
Flask-Script==2.0.5            # manage.py, shell
Flask-SQLAlchemy==2.1
Flask-WTF==0.12                # Formularios con protección CSRF
WTForms-Alchemy==0.15.0        # Para crear formularios basados en modelos
WTForms-Components==0.10.0     # Campos adicionales para los formularios de Flask
{% endraw %}```

![](/images/flask/nerd-dad.jpg)

Gracias por su atención y a [Tappsi](https://tappsi.co/) por el apoyo.

Estamos contratando jobs@tappsi.co