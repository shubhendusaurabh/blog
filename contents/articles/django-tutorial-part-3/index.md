---
title: "Django Tutorial - Part 3"
author: shubhu
date: 2015-08-01
template: article.jade
---

## Part 3

Welcome to part 3. In this part we will write `templates` and `admin.py` file for the app.

### Templates

In top10songs/settings.py file add the following lines for better templates organization.
```python
# List of callables that know how to import templates from various sources.
TEMPLATE_LOADERS = (
    'django.template.loaders.filesystem.Loader',
    'django.template.loaders.app_directories.Loader',
    'django.template.loaders.eggs.Loader',
)

TEMPLATE_CONTEXT_PROCESSORS = (
    'django.contrib.auth.context_processors.auth',
    'django.core.context_processors.request',

)
```

Next create `templates` folder in root folder of your app. Create new file `base.html`. This will be the layout for all other web pages.
```html
{% load staticfiles %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <link type="application/atom+xml" rel="alternate" href="{% url 'feeds' %}" />
    <title>{% block title %}Home{% endblock %} | Top 10 Songs</title>
    <link href="//netdna.bootstrapcdn.com/bootswatch/3.3.5/readable/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.css">
    {% block stylesheets %}{% endblock %}
</head>

<body>
    <div class="container">

        {% block content %} {% endblock %}

    </div>
    <!--container-->
    {% block scripts %}
    <script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
    <script>
        window.jQuery || document.write('<script src={% static "js/jquery.min.js" %}>')
    </script>
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>
    {% endblock %}
</body>

</html>
```

For `Charts List` create file `templates/chart_list.html`
```html
{% extends 'base.html' %}
{% block title %} All {{ language }} Charts {% endblock %}
{% block content %}
<div class="row text-center">
    <div class="col-md-2"></div>
    <div class="col-md-8">

        <div class="panel panel-info">
            <div class="panel-heading">All Charts List</div>
            <ul class="list-group">
                {% for chart in charts %}
                <li class="list-group-item">Top 10 {{ chart.get_language_display }} Songs in <a href='{{ chart.get_absolute_url }}'> {{ chart.week|date }}</a>
                </li>
                {% empty %}
                <li>No Chart Found</li>
                {% endfor %}
            </ul>
        </div>
        {% if is_paginated %}

        <ul class="pagination">

            {% if page_obj.has_previous %}
            <li><a href="?page={{ page_obj.previous_page_number }}">Prev</a>
            </li>
            {% endif %} {% for i in paginator.page_range %} {% if page_obj.has_next %}
            <li><a href="?page={{i}}">{{i}}</a>
            </li>
            {% endif %} {% endfor %} {% if page_obj.has_next %}
            <li><a href="?page={{ page_obj.next_page_number }}">Next</a>
            </li>
            {% endif %}

        </ul>

        {% endif %}
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
```

For `Chart detail` create file `templates/chart_detail.html`
```html
{% extends 'base.html' %}
{% load humanize %}
{% block title %}{{ object.week }} {{ chart.get_language_display }} Charts{% endblock %}
{% block content %}
<div class="panel panel-info text-center" itemscope itemtype="http://schema.org/MusicGroup">
    <!-- Default panel contents -->
    <div class="panel-heading">
        <h1 itemprop="name">Top 10 {{ chart.get_language_display }} Songs <small>({{ chart.week }})</small></h1>
    </div>
    <!-- List group -->
    <div class="list-group" >
        {% for song in object.get_songs %}
        <article class="list-group-item center-block" itemtype="http://schema.org/MusicRecording" itemscope itemprop="tracks">
            <div class="row">
                <div class="col-md-4">
                    <header>
                        <h3 class="list-group-item-heading"><span class="label label-default">#{{ forloop.counter }}</span></h3>
                        <h2 itemprop="name"><a href="{{ song.get_absolute_url }}" itemprop="url">{{ song.name }}</a></h2>
                    </header>
                </div>
                <div class="col-md-4">
                    <a href="{{ song.get_absolute_url }}" itemprop="audio">
                        <img class="img-thumbnail" alt="{{ song.name }}" src="http://i.ytimg.com/vi/{{ song.youtube_id }}/default.jpg" itemprop="image">
                    </a>
                </div>
                <div class="col-md-4">
                    <div>
                        {% if song.album %}
                        <p>Album: <span itemprop="inAlbum"><strong><a href="{% url 'search' %}?q={{ song.album }}">{{ song.album }}</a></strong></span>
                        </p>
                        {% endif %}
                        {% if song.get_artists %}
                        <p>Artist: <span itemprop="byArtist"><strong>
                            {% for artist in song.get_artists %}
                                <a href="{% url 'search' %}?q={{ artist }}">{{ artist }}</a>
                                {% if not forloop.last %}{% ifequal forloop.revcounter 2 %} and {% else %}, {% endifequal %}{% else %}{% endif %}
                            {% endfor %}
                        </strong></span>
                        </p>
                        {% endif %}
                        {% if song.publisher %}
                        <p>Publisher: <span itemprop="publisher"><strong>{{ song.publisher }}</strong></span>
                        </p>
                        {% endif %}
                    </div>
                </div>
            </div>
        </article>
        {% endfor %}
    </div>
</div>
{% endblock %}
```

For `Song list` create `templates/song_list.html`
```html
{% extends 'base.html' %}
{% block title %}All Songs{% endblock %}
{% block content %}
<div class="row text-center">
    <div class="col-md-2"></div>
    <div class="col-md-8">

    <div class="panel panel-info">
        <div class="panel-heading">All Songs List</div>
        <ul class="list-group">
            {% for song in object_list %}
            <li class="list-group-item"><a href='{{ song.get_absolute_url }}'> {{ song.name }} by {{ song.artist|default:'N/A' }}</a>
            </li>
            {% empty %}
            <li>No Songs Found</li>
            {% endfor %}
        </ul>
    </div>
    {% if is_paginated %}

    <ul class="pagination">

        {% if page_obj.has_previous %}
        <li><a href="?page={{ page_obj.previous_page_number }}">Prev</a>
        </li>
        {% endif %} {% for i in paginator.page_range %} {% if page_obj.has_next %}
        <li><a href="?page={{i}}">{{i}}</a>
        </li>
        {% endif %} {% endfor %} {% if page_obj.has_next %}
        <li><a href="?page={{ page_obj.next_page_number }}">Next</a>
        </li>
        {% endif %}

    </ul>

    {% endif %}
   </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
```html

For `Song detail` create file `templates/song_detail.html`
```html
{% extends 'base.html' %}
{% block title %}
{{ song.name }} {% if song.artist %}by {{ song.artist }} {% endif %} {% if song.album %} from {{ song.album }} {% endif %}
{% endblock %}
{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <article itemtype="http://schema.org/MusicRecording" itemscope itemprop="track" class="col-md-8">
        <header>
            <h1 itemprop="name">{{ song.name }}</h1>
        </header>
        <div>
            {% if song.album %}
            <p>Album:
                <span itemprop="inAlbum">
                    <strong>
                        <a href="{% url 'search' %}?q={{ song.album }}">{{ song.album }}</a>
                    </strong>
                </span>
            </p>
            {% endif %} {% if song.get_artists %}
            <p>Artist:
                <span itemprop="byArtist">
                    <strong>
                        {% for artist in song.get_artists %}
                        <a href="{% url 'search' %}?q={{ artist }}">{{ artist }}</a>
                        {% if not forloop.last %}{% ifequal forloop.revcounter 2 %} and {% else %}, {% endifequal %}{% else %}{% endif %}
                        {% endfor %}
                    </strong>
                </span>
            </p>
            {% endif %} {% if song.publisher %}
            <p>Publisher: <span itemprop="publisher"><strong>{{ song.publisher }}</strong></span>
            </p>
            {% endif %}
            <meta itemprop="datePublished" content={{ song.created_at }} />
            <meta itemprop="duration" content="PT5M0S" />
        </div>
        <div itemprop="video" itemscope itemtype="http://schema.org/VideoObject">

            <p class="muted">Youtube Link: <a href="http://youtu.be/{{ song.youtube_id }}" itemprop="audio">{{ song.name }}</a>
            </p>
            <section class="row">
                <div class="embed-responsive embed-responsive-16by9">
                    <iframe src="//www.youtube-nocookie.com/embed/{{ song.youtube_id }}" class="embed-responsive-item" allowfullscreen></iframe>
                </div>
            </section>
        </div>
        <div class="row">
            <h3>Chart Date <span class="pull-right">Position</span></h3>
              <div class="list-group">
              {% for ranking in song.song.select_related %}
              <a class="list-group-item" href="{{ ranking.chart.get_absolute_url }}">
                      <span class="badge">{{ ranking.position }}</span>
                      Top 10 {{ ranking.chart.get_language_display}} Songs in {{ ranking.chart.week|date }}
                  </a>
              {% endfor %}
              </div>
        </div>
    </article>
    <div class="col-md-2"></div>
</div>
{% endblock %}
```
The homepage template is similar to chart detail template except that it has more than one template.
And the archive templates are similar to chart list templates.
### Admin
Edit your `charts/admin.py` file and import all the models created in `charts/models.py` file.
```python
class SongInline(admin.StackedInline):
    model = Chart.songs.through
    extra = 10

class ChartAdmin(admin.ModelAdmin):
    inlines = [SongInline]
```
We are adding songs as inline form for the admin panel. And as songs are added using a through table, hence the model name is `Chart.songs.through`.


### Django Tutorial Series - Making [Top10Songs](http://www.top10songs.co.in)

* [Django Tutorial Part 1](/articles/django-tutorial-part-1)
* [Django Tutorial Part 2](/articles/django-tutorial-part-2)
* [Django Tutorial Part 3](/articles/django-tutorial-part-3)
* [Django Tutorial Part 4](/articles/django-tutorial-part-4)
