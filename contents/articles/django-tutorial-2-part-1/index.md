---
title: "Django Tutorial 2 - Part 1"
author: shubhu
date: 2015-08-05
template: article.jade
---

Welcome to Second hands on Django Tutorial. In this part we will make a testseries application. We will develop the backend using django and django-rest-framework while for the frontend will be developed in AngularJS.

This series is work in progress, so it make take a while before application is completed.


### Prerequitese
Along with Django this project also requires bower to be installed and uses sqlite as database.

As usual create a new django project in virtualenv using virtualenvwrapper.
```bash
mkvirtualenv testseries
```
In the newly created project install following dependencies.
```
Django==1.8.2
Pillow==2.9.0
argparse==1.2.1
dj-database-url==0.3.0
dj-static==0.0.6
django-debug-toolbar==1.3.2
django-extensions==1.5.5
django-toolbelt==0.0.1
djangorestframework==3.1.3
gunicorn==19.3.0
psycopg2==2.6.1
six==1.9.0
sqlparse==0.1.15
static3==0.6.1
wsgiref==0.1.2
```

Apply the following changes to `testseries/settings.py` file
```python
-SECRET_KEY = 'kw_z-k_7yz$owwel0afiu5vd_q1a@(iub_to_u*&hjjt@4y4yy'
+from django.core.exceptions import ImproperlyConfigured
+
+def get_env_variable(var_name):
+    try:
+        return os.environ[var_name]
+    except KeyError:
+        error_msg = "Set the %s env variable" % var_name
+        raise ImproperlyConfigured(error_msg)
+
+SECRET_KEY = get_env_variable('SECRET_KEY')

-DEBUG = True
+DEBUG = False
+TEMPLATE_DEBUG = False

+STATICFILES_DIRS = (
+    os.path.join(BASE_DIR, "static"),
+)
+
+TEMPLATE_DIRS = (
+    os.path.join(BASE_DIR, 'templates'),
+)
+
+SITE_ID = 1
+
+INTERNAL_IPS = ('127.0.0.1',)
+
+# List of callables that know how to import templates from various sources.
+TEMPLATE_LOADERS = (
+    'django.template.loaders.filesystem.Loader',
+    'django.template.loaders.app_directories.Loader',
+    'django.template.loaders.eggs.Loader',
+)
+
+TEMPLATE_CONTEXT_PROCESSORS = (
+    'django.contrib.auth.context_processors.auth',
+    'django.core.context_processors.request',
+  
+)
+
+if get_env_variable('DEVELOPMENT') == 'True':
+
+    INSTALLED_APPS += ('debug_toolbar', 'django_extensions', )
+
+    MIDDLEWARE_CLASSES += ('debug_toolbar.middleware.DebugToolbarMiddleware', )
+
+    INTERNAL_IPS = ('127.0.0.1', )
+
+    DEBUG = True
+
+    TEMPLATE_DEBUG = True
+
+    DATABASES = {
+        'default': {
+            'ENGINE': 'django.db.backends.sqlite3',
+            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
+        }
+    }
```

Next run `python manage.py migrate` to setup initial tables and then run `python createapp questions` to initailize new app.

In the questions app 
