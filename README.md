# Django Heroku Deployment Guide 

## This is a step-by-step guide on how to deploy to Heroku with all the configuration details.

1. Create a Heroku project:

```
heroku create <heroku_app_name>
```

2. Connect your git to heroku remote:

```
heroku git:remote -a [heroku_app_name]
```

3. Add `SECRET_KEY` and `DEBUG_VALUE` to `.bash_profile` local file

4. Then, configure and copy the same settings on Heroku server:

```
heroku config:set SECRET_KEY='<your_secret_key>'
heroku config:set DEBUG_VALUE='False'
```

5. Heroku web applications require a `Procfile`. In the `Procfile`, we'll tell Heroku to start a Gunicorn production server and point to that server to our Django project's default WSGI interface. It is located in the root of your project directory. Create a `Procfile` with the following code:  
```
web gunicorn <django_project_name>.wsgi
```

6. Configure Django project `settings.py` file:

    - Update `SECRET_KEY` and `DEBUG`:
        ```
        SECRET_KEY = os.environ.get('SECRET_KEY')
        DEBUG = (os.environ.get('DEBUG_VALUE') == 'True')
        ```
	- Update `STATIC_ROOT`. This is where Heroku will find all your static files to serve from your Django project: 
    ```
    STATIC_ROOT = os.path.join(BASE_DIR, ‘staticfiles’)
    ```
    - Update `ALLOWED_HOSTS` list with your domain you're deploying
    - Install and add `gunicorn` and `django-heroku` packages to `requirements.txt`, as well as other dependencies you've installed. `django-heroku` is a package that automatically and seamlessly configures your Django application to work on Heroku production server, such as database configuration.
    ```
    pip install gunicorn django-heroku
    pip freeze > requirements.txt
    ```
    You should have something basic like this:
    ```
    django==2.2
    gunicorn==20.0.4
    django-heroku==0.3.1
    ...<other_dependencies_you_installed>
    ```
    - Add `django-heroku` code: 
    ```
    import django_heroku // top of settings.py
    ... 
    django_heroku.settings(locals()) // bottom of settings.py
    ```


Your settings.py should now look like this: 

```
import os
import django_heroku


# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ.get('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = (os.environ.get('DEBUG_VALUE') == 'True')

ALLOWED_HOSTS = [
    'localhost',
    '<your_domain>',
]

.
.
.

# collected static files go here: python manage.py collectstatic
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

django_heroku.settings(locals())
```



7.	Save the changes and commit locally:
```
git add .
git commit -m “Updated settings for Heroku production”
```

8. Push to Heroku remote repository:

```
git push heroku master
```

9. Run migration files on Heroku production server, and
create a superuser to access the admin site:
```
heroku run bash
python manage.py migrate 
python manage.py createsuperuser
```

