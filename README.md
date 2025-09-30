Clone git repository : https://github.com/slim-elabed/empty-django-project.git

Remove/Add needed python libraries in requirements/base.txt

Run commands :

- docker-compose.exe -f docker-compose-dev.yml build
- docker-compose.exe -f docker-compose-dev.yml up
    - backend, celery and celer-beat containers will crash : needed code is not yet added, ignore it for the moment

## Initialise DB:

connect to container postgres and launch these commands :

- su - postgres
- psql
```sql
CREATE DATABASE myapp;
CREATE USER myapp_user WITH ENCRYPTED PASSWORD 'mypass';
GRANT ALL PRIVILEGES ON DATABASE myapp TO myapp_user;

\c myapp;
GRANT USAGE, CREATE ON SCHEMA public TO myapp_user;
```    
modify backend command in docker-compose-dev.yml : comment sleep command
run docker-compose.exe -f docker-compose-dev.yml up
connect to backend container and run this command :

```bash
django-admin startproject myapp backend
```
Edit settings.py : modify Database config
```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "<db_name>",
        "USER": "<django_dbuser>",
        "PASSWORD": "<django_dbpasswd>",
        "HOST": "Database",  # service name in docker-compose
        "PORT": 5432,  # default postgres port
    }
}
```
connect to container backend and launch these commands :

```bash
cd <path-to-django-project>

python manage.py migrate

python manage.py createsuperuser

python manage.py runserver 0.0.0.0:8000

```

Go to your browser and connect to [localhost:8000](http://localhost:8000) and [localhost:8000/admin](http://localhost:8000/admin) to verify the result.

modify docker_compose_dev.yml, backend command : comment sleep line and uncomment runserver line.

## Adding Apps to the project:

reconnect to backend container and create needed apps, example :

- python manage.py startapp <my_app_name>
- modify your model file, view, urls …
- add app in site configfile (settings.py)
- python manage.py makemigrations  <my_app_name>
- python manage.py migrate <my_app_name>

## Add support of Celery and Celery Beat:

create file celery.py in project folder (like settings.py)

```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myapp.settings')

app = Celery('myapp') # this is the folder name where you put your file celery.py
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

```

modify __init__.sh in project folder, add these lines :

```python
from .celery import app as celery_app

__all__ = ('celery_app',)

```

add this lines to settings.py

```python
CELERY_BROKER_URL = 'redis://redis:6379/0'
CELERY_RESULT_BACKEND = 'redis://redis:6379/0'

```

add "django_celery_beat" to installed path (settings.py):

```python
INSTALLED_APPS = [
    # tes apps Django natives
    "myapp",              # ton projet principal
    "django_celery_beat", # gestion des tâches périodiques avec Celery Beat
    # autres apps
]

```

apply migration of celery beat (to create default tables in DB), connect to backend container and run this command : 

```bash
python [manage.py](http://manage.py/) migrate django_celery_beat

```

Restart all containers :

```bash
docker-compose.exe -f docker-compose-dev.yml down
docker-compose.exe -f docker-compose-dev.yml up
```

check containers states and logs
