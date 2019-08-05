## Heroku Deployment

1. Is the Python version you're using specified in your Pipfile?

* ONLY DO THIS IF THE PYTHON VERSION YOU'RE USING IS NOT SPECIFIED
* specify Python version in your Pipfile, e.g. `python_version = "3.7"`
* $ pipenv lock ($ indicates a terminal command, you do not type $)

2. Install gunicorn

* gunicorn is a production suitable web server
* $ pipenv install gunicorn==19.9.0

3. Create a Procfile

* update your Procfile
	* web: gunicorn <project_name>.wsgi --log-file -
  * angle brackets indicate required content, you do not type the angle brackets
  * replace <project_name> with the name of your project, e.g. conf
	* server configuration is contained in the wsgi.py file

4. Install whitenoise

	* $ pipenv install whitenoise==3.3.1
	* update settings.py
		* add `'whitenoise.runserver_nostatic'` above `'django.contrib.staticfiles'` in INSTALLED_APPS
		* add `'whitenoise.middleware.WhiteNoiseMiddleware'` to MIDDLEWARE directly after the Django SecurityMiddleware
    
5. Install dj-database-url

  * $ pipenv install dj-database-url
  * updated settings.py
    * add `'import dj_database_url'` directly below `'import os'` at the top of the file
    * update the DATABASES setting
```
if DEBUG:
    DATABASES = {
        'default': dj_database_url.config(default=os.environ['DATABASE_URL']),
    }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }
```
  * `DEBUG = True` is the default setting (look towards the top of the setting.py file)
    * Change `DEBUG` to `DEBUG = False` before pushing to GitHub
    * Change `DEBUG` to `DEBUG = True` to run your app locally
  
6. Add `STATIC_ROOT` and `STATICFILES_STORAGE` to the bottom of your settings.py file

	* `STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')`
	* `STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'`
  
7. Add `REACT_APP_DIR` and `STATICFILES_DIRS` below `STATICFILES_STORAGE` setting.

  * `REACT_APP_DIR = os.path.join(BASE_DIR, 'frontend/static')`
  * `STATICFILES_DIRS = (
      os.path.join(BASE_DIR, 'frontend/static/build/static'),
  )`

6. Install psycopg2 

7. Create Heroku app

	* Add the Heorku domain to the ALLOWED_HOSTS list in settings.py
	* Copy and paste everything except the https://
