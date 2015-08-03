---
title: "Django Tutorial - Part 2"
author: shubhu
date: 2015-07-31
template: article.jade
---

Welcome to part 2 of Django Tutorial. In this part we will write `Views` and `Urls` files.

## `Views`
In the `charts\views.py` file import datetime, `django.views` modules and models created in `models.py` file.

You can create archive views based on year, month, week, day, today or any particular date by using queryset on `Chart` model.
Archive views need `queryset` or `model name`, `date field` in order to work. You can also define various date formats as your choice, just remember to update `urls.py` file accordingly.

With [Django Class Based Views](https://docs.djangoproject.com/en/1.8/topics/class-based-views/) its very easy to create List View or Detail View for any model.

We are interested in `ListView` and `DetailView` for `Chart` and `Song` model.

For `DetailView` for `Chart` use the method `prefetch_related('songs')` as it has many to many relationship on songs and for `Song` use `select_related` to minimize number of database queries.

For `ListView` for charts we are using filter by language for any particular language otherwise displaying all charts.

We will also need to create `HomePage` view with parent class `TemplateView`. Give it template name and query for latest charts for each language.


## Urls

In the `urls.py` import all the views created above. Next we create regex based patterns for urls.

For charts we will use primary key as lookup key in the url while for songs we will have primary key as well as slug created from song name in urls for Songs in detail view.
`url(r'^chart/(?P<pk>\d+)/$', ChartDetailView.as_view(), name='chart_detail'),`

`url(r'^song/(?P<pk>\d+)/(?P<slug>[\w-]+)/$', SongDetailView.as_view(), name='song_detail')`

For list views we can simply have plural form as urls.
`url(r'^songs/$', SongListView.as_view(), name='song_list')`

`url(r'^charts/$', ChartListView.as_view(), name='chart_list')`

We will also need to create url for HomePage.
`url(r'^$', HomePageView.as_view(), name='homepage')`

### Django Tutorial Series - Making [Top10Songs](http://www.top10songs.co.in)

* [Django Tutorial Part 1](/articles/django-tutorial-part-1)
* [Django Tutorial Part 2](/articles/django-tutorial-part-2)
* [Django Tutorial Part 3](/articles/django-tutorial-part-3)
* [Django Tutorial Part 4](/articles/django-tutorial-part-4)
