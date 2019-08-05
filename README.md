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
```
ALLOWED_HOSTS = [
    '127.0.0.1',
    'sleepy-eyrie-95680.herokuapp.com',  # add your Heroku domain here, this is an example
]
```

8. Add a database
	
* Open your app in Heroku and navigate to the Resouces tab
* Under Add-ons, search for Heroku Postgres
* Choose Hobby Dev - Free when prompted
* Click provision
* Navigate to the Settings tab
* Click Reveal Config Vars
* Copy the database url and paste it inside the .env file located in the root directory of your Django project
	* DATABASE_URL = 'database_url_goes_here'
		
* Add server-side environment variables to the Config Vars in Heroku.
	* Server-side env variables are stored in an .env file located in the root directory of your Django app
	* the key must match the name of the environment variable inside your .env file
	* e.g. Key = TWILIO_AUTH_TOKEN, Value = '1a2cb345678912c1b2222dd33e44a1f7'
		
* Do not add client-side environment variables to the Config Vars section in Heroku.
	* Client-side env variables are stored in an .env file located in the root directory of your React app
		* React env variables are bundled when you run npm run build	
		* React env variables are exposed in your code
		* React env variable example: REACT_APP_BASE_URL = "https://intense-thicket-80055.herokuapp.com"
		
* If you are using an React env variable for your BASE_URL, you will need to update it depending on whether you are runnning localhost:3000, localhost:8000, or deploying to Heroku. Examples of each are below. The Heroku url is project specific.
		
	* REACT_APP_BASE_URL = "https://intense-thicket-80055.herokuapp.com"
	* REACT_APP_BASE_URL = "http://localhost:8000"
	* REACT_APP_BASE_URL = "http://localhost:3000"
		
9. Deploy your app

* In Heroku, navigate to the Deploy tab
* Connect your app to your GitHub repository
* Deploy branch
* Recommended that you Enable Automatic Deploys
		
## DEPLOYMENT TROUBLESHOOTING

Below are things to check if you have deployment issues:

* Read and resolve all deployment error messages generated by Heroku
* Make sure the frontend app includes:
	* views.py that loads the build directory (see code below)
	* urls.py that renders the IndexView in views.py on the home route, `path('', IndexView.as_view(), name='index'),`
```
import os
import logging

from django.views.generic import View
from django.http import HttpResponse
from django.conf import settings


class IndexView(View):
    """
    Serves the compiled frontend entry point (only works if you have run `npm
    run build`).
    """

    def get(self, request, *args, **kwargs):
        try:
            with open(os.path.join(settings.REACT_APP_DIR, 'build', 'index.html')) as f:
                return HttpResponse(f.read())
        except FileNotFoundError:
            logging.exception('Production build of app not found')
            return HttpResponse(
                """
                This URL is only used when you have built the production
                version of the app. Visit http://localhost:3000/ instead, or
                run `npm run build` to test the production version.
                """,
                status=501,
            )
```
	
* Does your conf urls include your frontned urls
* Did you run npm run build inside the static directory
		
	
## SAMPLE settings.py

Below is a sample settings.py file of a deployed project. Specific settings will vary based on project.
```
"""
Django settings for conf project.

Generated by 'django-admin startproject' using Django 2.2.3.

For more information on this file, see
https://docs.djangoproject.com/en/2.2/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/2.2/ref/settings/
"""

import os
import dj_database_url

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.2/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = '^m&wl-ut1xvoej@#s77b_pbrr@$ujpbrvip_m^@(#kbn0l^mzf'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ["*"]


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.sites',
    'whitenoise.runserver_nostatic',
    'django.contrib.staticfiles',

    'rest_framework',
    'rest_framework.authtoken',
    'rest_auth',
    'allauth',
    'allauth.account',
    'rest_auth.registration',
    'allauth.socialaccount',
    'twilio',

    'frontend.apps.FrontendConfig',
    'accounts.apps.AccountsConfig',
    'api.apps.ApiConfig',
]

REST_AUTH_SERIALIZERS = {
    'REGISTER_SERIALIZER': 'api.serializers.RegisterSerializer',
    'TOKEN_SERIALIZER': 'api.serializers.TokenSerializer',
}



REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        # project level permissions
        'rest_framework.permissions.IsAuthenticated'
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
        # https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication
        #  To use the TokenAuthentication scheme you'll need to configure the authentication classes to include TokenAuthentication
        'rest_framework.authentication.TokenAuthentication',  # new

    ]
}

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'conf.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR, 'frontend/static/build')
        ],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'conf.wsgi.application'


# Database
# https://docs.djangoproject.com/en/2.2/ref/settings/#databases


# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
#     }
# }

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


# Password validation
# https://docs.djangoproject.com/en/2.2/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/2.2/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.2/howto/static-files/

STATIC_URL = '/static/'

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

REACT_APP_DIR = os.path.join(BASE_DIR, 'frontend/static')

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'frontend/static/build/static'),
)

AUTH_USER_MODEL = 'accounts.CustomUser'

SITE_ID = 1

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

TWILIO_API_SID = ['TWILIO_API_SID']
TWILIO_AUTH_TOKEN = ['TWILIO_AUTH_TOKEN']
SERVICE_SID = ['SERVICE_SID']
```
