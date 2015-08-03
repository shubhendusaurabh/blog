---
title: "Django Tutorial - Part 4"
author: shubhu
date: 2015-08-02
template: article.jade
---

Welcome to part 4 of Django Tutorial. In this part we will create feeds and sitemaps.

### RSS Feeds
Create charts/feeds.py file.
Add the following code.

```python
from django.contrib.syndication.views import Feed
from .models import Chart


class LatestChartsFeeds(Feed):
    title = 'Top 10 Songs'
    link = '/charts/'
    description = "Top 10 Songs in English, Hindi. Weekly Updated"

    def items(self):
        return Chart.objects.order_by('-week')[:10]

    def item_title(self, item):
        return "Top 10 " + item.get_language_display() + " Songs in " + item.week.strftime("%B %d, %Y")

    def item_description(self, item):
        songs = item.songs.all()
        content = '<ul>'
        template = '<li><a href="%s">%s</a></li>'
        for song in songs:
            temp = template % (song.get_absolute_url(), song.name)
            content += temp
        return content + '</ul>'
```
Here we are fetching lastest chart in each language and displaying them in format.
Read more about [syndication feeds](https://docs.djangoproject.com/en/1.8/ref/contrib/syndication/) in django.

### Sitemaps

For Sitemaps add `'django.contrib.sitemaps'` to `INSTALLED_APPS` tuple.
```python
from django.contrib.sitemaps import Sitemap
from .models import Chart, Song


class ChartSitemap(Sitemap):
    changefreq = "never"
    priority = 0.5

    def items(self):
        return Chart.objects.all()

    def lastmod(self, obj):
        return obj.created_at


class SongSitemap(Sitemap):
    changefreq = 'never'
    priority = 0.5

    def items(self):
        return Song.objects.all()

    def lastmod(self, obj):
        return obj.updated_at
```
Read more about [sitemaps](https://docs.djangoproject.com/en/1.8/ref/contrib/sitemaps/) in django.

### Django Tutorial Series - Making [Top10Songs](http://www.top10songs.co.in)

* [Django Tutorial Part 1](/articles/django-tutorial-part-1)
* [Django Tutorial Part 2](/articles/django-tutorial-part-2)
* [Django Tutorial Part 3](/articles/django-tutorial-part-3)
* [Django Tutorial Part 4](/articles/django-tutorial-part-4)
