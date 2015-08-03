---
title: "Django Tutorial - Part 1"
author: shubhu
date: 2015-07-30
template: article.jade
---

This blog series is about how [Top 10 Songs](http://www.top10songs.co.in) was written in [Django](https://www.djangoproject.com/). Mostly intended for beginners looking forward to learning [Django](https://www.djangoproject.com/).
Hope it helps.

## The Beginning
This article assumes that you have already installed django and configured it. If not use [Django docs](https://docs.djangoproject.com/en/1.8/) for basic setup process.

Next using your cli interface cd into directory where you want to keep your code and then run the following command:
`django-admin startproject top10songs`

It will create pretty basic django app boilerplate code.

Next in `top10songs/settings.py` django will have `SECRET_KEY` auto generated for you but remove it before deploying the project to public and move the reference to environment variables.

We can set django to use environment variables instead of hardcoding by using following function.

```python
from django.core.exceptions import ImproperlyConfigured


def get_env_variable(var_name):
    try:
        return os.environ[var_name]
    except KeyError:
        error_msg = "Set the %s env variable" % var_name
        raise ImproperlyConfigured(error_msg)

SECRET_KEY = get_env_variable('SECRET_KEY')
```
As you can read from the code it tries to get particular environment variable otherwise it raises Error and django won't start.

So you will need to set the environment variables by running
```bash
export SECRET_KEY='<random string>' # for Linux
set SECRET_KEY='<random string>' # for Windows
```

Next change database, timezone, language settings as per requirements.

Also for convenience install django plugins like
`Fabric`
`django-extensions`
`django-debug-toolbar`
`pip-tools`
`django-htmlmin`
and add them to `INSTALLED_APPS` tuple.

Next run `python manage.py migrate` for initial database setup
After that you can run
```bash
python manage.py runserver
```
If everything goes right you will be greeted with welcome page.

Next we begin to start creating our app.
To do that run
```bash
python manage.py startapp charts
```
## [The Models](https://github.com/shubhendusaurabh/top10songs/blob/master/charts/models.py)

In `charts/models.py` file
Create Models `Chart`, `Song` and `Ranking`

For `Chart` model will have following attributes
* `language` (what language is the chart for)
* `songs` (more on this later)
* `week` Datefield (which week chart is for)
* `created` (for storing when was the chart created)

Methods
* `get_songs`()
    because we will be using many to many field using a through table and want to preserve the position of the songs
* `get_absolute_url`()
    for quick link to the chart object based on its id
* `__unicode__`()
    for displaying in console or admin app

For `Song` Model attributes are
* `name` (Song name)
* `slug` (autoslug field imported from django-extensions)
* `artist` (song artist)
* `album` (song album)
* `youtube_id` (song youtube video url if available)
* `publisher` (song publisher)
* `created` and `updated` fields for log keeping

Methods:
* `__unicode__`:
    for displaying object in admin or console when printing
* `get_absolute_url`:
    reverse song_detail based on primary key
* `get_artists`:
    this method just splits artists name from strings like & or words like Featuring, With etc

Model `Ranking`

This is a through table because we need to store position of the songs in the charts.
So it will have 3 fields
* `song` (Foreignkey to song)
* `chart` (Foreignkey to chart)
* `position` (ordering will be based on position)

With models complete we will look at `Views` and `Urls` in the next part.

### Django Tutorial Series - Making [Top10Songs](http://www.top10songs.co.in)

* [Django Tutorial Part 1](/articles/django-tutorial-part-1)
* [Django Tutorial Part 2](/articles/django-tutorial-part-2)
* [Django Tutorial Part 3](/articles/django-tutorial-part-3)
* [Django Tutorial Part 4](/articles/django-tutorial-part-4)
