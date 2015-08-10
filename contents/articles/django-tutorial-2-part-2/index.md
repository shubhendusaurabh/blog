---
title: "Django Tutorial 2 - Part 2"
author: shubhu
date: 2015-08-08
template: article.jade
---

### Questions app

In previous part we initialized django project with database settings and various configurations.
So lets begin with creating models and views for the app.

### Models
Open `questions/models.py` file and add the following code.
```python
from __future__ import unicode_literals
from django.utils.encoding import python_2_unicode_compatible

from django.db import models

@python_2_unicode_compatible
class Exam(models.Model):
	name = models.CharField(max_length=100)
	abbr = models.CharField(max_length=10)
	description = models.TextField(blank=True)
	created_at = models.DateTimeField(auto_now_add=True)
	updated_at = models.DateTimeField(auto_now=True)

	def __str__(self):
		return self.name

@python_2_unicode_compatible
class Subject(models.Model):
	name = models.CharField(max_length=50)

	def __str__(self):
		return self.name

@python_2_unicode_compatible
class Unit(models.Model):
	name = models.CharField(max_length=100)
	subject = models.ForeignKey(Subject)

	def __str__(self):
		return self.name

@python_2_unicode_compatible
class Chapter(models.Model):
	name = models.CharField(max_length=100)
	unit = models.ForeignKey(Unit)

	def __str__(self):
		return self.name

@python_2_unicode_compatible
class Question(models.Model):
	question_text = models.TextField()
	answer = models.CharField(max_length=2)
	hint = models.TextField(null=True, blank=True)
	chapter = models.ForeignKey(Chapter)
	image = models.ImageField(upload_to='static/img/questions', null=True, blank=True)
	created_at = models.DateTimeField(auto_now_add=True)
	updated_at = models.DateTimeField(auto_now=True)

	def __str__(self):
		return self.question_text

@python_2_unicode_compatible
class Choice(models.Model):
	question = models.ForeignKey(Question, related_name='choices')
	choice_text = models.CharField(max_length=200)

	def __str__(self):
		return self.choice_text
```
In order to make our code both python 2 and python 3 compatible we are importing `unicode_literals` and `python_2_unicode_compatible`. Read more about it at [django docs](https://docs.djangoproject.com/en/1.8/topics/python3/#str-and-unicode-methods).
Sample value for above model would be like following.
```json
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "pk": 4,
            "question_text": "This is my 1st Question ?",
            "answer": "1",
            "hint": "Hint for question one !",
            "chapter": {
                "url": "http://powerful-gorge-4516.herokuapp.com/api/chapters/1/",
                "name": "Group D",
                "unit": {
                    "url": "http://powerful-gorge-4516.herokuapp.com/api/units/1/",
                    "name": "2",
                    "subject": {
                        "url": "http://powerful-gorge-4516.herokuapp.com/api/subjects/1/",
                        "name": "Chemistry "
                    }
                }
            },
            "image": null,
            "choices": [
                "D. None of the above is correct.",
                "C. Choice C for Q 1",
                "B. Choice B for Q 1",
                "A. Choice A for Q1"
            ]
        },
        {
            "pk": 5,
            "question_text": "What is the work done by a man in carrying a suitcase weighing 30 kg over his head, when he travels 10 m in horizontal direction.",
            "answer": "1",
            "hint": "This is pretty basic question. No help required?",
            "chapter": {
                "url": "http://powerful-gorge-4516.herokuapp.com/api/chapters/3/",
                "name": "Organic",
                "unit": {
                    "url": "http://powerful-gorge-4516.herokuapp.com/api/units/2/",
                    "name": "Polymers",
                    "subject": {
                        "url": "http://powerful-gorge-4516.herokuapp.com/api/subjects/1/",
                        "name": "Chemistry "
                    }
                }
            },
            "image": null,
            "choices": [
                "30 kJ",
                "0",
                "(\\ a \\ne 0 \\)",
                "When $$a \\ne 0$$, there are two solutions to \\(ax^2 + bx + c = 0\\) and they are $$x = {-b \\pm \\sqrt{b^2-4ac} \\over 2a}.$$"
            ]
        }
    ]
}
```

### Serializers
Create a new file `questions/serializers.py` and add the following code.
```python
from rest_framework import serializers

from .models import Question, Choice, Exam, Subject, Unit, Chapter

class QuestionSerializer(serializers.HyperlinkedModelSerializer):
    choices = serializers.StringRelatedField(many=True)
    class Meta:
        model = Question
        fields = ('pk', 'question_text', 'answer', 'hint', 'chapter', 'image', 'choices')
        depth = 3

class ChoiceSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Choice
        fields = ('choice_text', 'question')

class ExamSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Exam
        fields = ('name', 'abbr', 'description')

class SubjectSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Subject
        fields = ('name', )

class UnitSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Unit
        fields = ('name', 'subject')

class ChapterSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Chapter
        fields = ('name', 'unit')
```
In the above example we are creating json serializers for all the models we just created. They are subclass of `HyperlinkedModelSerializer` which need a model and its fields which we want to display. `QuestionSerializer` needs an extra field `depth` because it has parent foreign key i.e. `chapter`.

### Views
Edit your `questions/views.py` file to add the following code.
```python
from datetime import datetime, timedelta

from django.shortcuts import render

from django.db.models import Count
from django.views.generic.detail import DetailView
from django.views.generic.edit import UpdateView, CreateView, DeleteView
from django.views.generic.list import ListView

from rest_framework.viewsets import ModelViewSet
from rest_framework.renderers import JSONRenderer
from rest_framework.response import Response
from rest_framework.views import APIView

from .serializers import QuestionSerializer, ChoiceSerializer, ExamSerializer, UnitSerializer, ChapterSerializer, SubjectSerializer

from .models import Question, Choice, Exam, Unit, Chapter, Subject

class QuestionDetail(DetailView):
    model = Question
question_detail = QuestionDetail.as_view()

class QuestionList(ListView):
    model = Question
question_list = QuestionList.as_view()

class QuestionViewSet(ModelViewSet):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer

class ChoiceViewSet(ModelViewSet):
    queryset = Choice.objects.all()
    serializer_class = ChoiceSerializer

class ExamViewSet(ModelViewSet):
    queryset = Exam.objects.all()
    serializer_class = ExamSerializer

class SubjectViewSet(ModelViewSet):
    queryset = Subject.objects.all()
    serializer_class = SubjectSerializer

class UnitViewSet(ModelViewSet):
    queryset = Unit.objects.all()
    serializer_class = UnitSerializer

class ChapterViewSet(ModelViewSet):
    queryset = Chapter.objects.all()
    serializer_class = ChapterSerializer
```
In the above code we have imported Django's generic class views like `ListView`, `DetailView` and also from `django rest framework`. We are creating a `viewset` for each model and its `serializer`.


That's all for this part in the next part we will create templates and get started on the frontend.
