---
layout: post
title: 'Django를 이용한 Rest API 서버 구축'
author: through.kim
date: 2018-05-29 16:28
tags: [python, django, study]
image: '/files/covers/python-in-ex-machina.jpg'
comments: true
---

> Django를 이용해 Restful한 API 서버를 구축하는 방법을 익혀본다.

 - Ubuntu 16.04-64-server
 - Python 3.5.2
 - Django 2.0.5
 - django-rest-swagger 2.2.0
 
## 환경 세팅

파이썬 버전을 확인해 준다. 파이썬 3.x 버전이 없다면 설치해준다.

```bash
$ python --version
$ python3 --version
```

이어서 가상환경을 세팅해주기 위해 virtualenv를 설치해준다.

```bash
$ apt-get install virtualenv
```

virtualenv 설치가 완료되었다면 학습을 위한 가상환경을 생성한다.

```bash
$ virtualenv -p python3 rest_env
```

가상환경이 생성완료 되면, 가상환경을 activate 시켜준다.
정상적으로 가동되었다면 터미널 입력창 앞에 `(rest_env)`가 붙어있을 것이다.

```bash
$ source ~/rest_env/bin/activate
```

이어서 필요한 Django 및 파이썬 라이브러리들을 설치해준다.

```bash
$ pip install django
$ pip install djangorestframework
$ pip install django-rest-swagger
```

## Django 세팅

이제 startproject 명령어를 이용해 장고 프로젝트를 생성해 준다.

```bash
$ cd ~/
$ django-admin startproject rest_server
$ cd rest_server
```

이어서 rest_server/settings.py 파일을 열어서 다음과 같이 내용을 추가해준다.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    'rest_framework',
    'rest_framework_swagger'
]
```

마지막으로 사용할 앱을 생성해준다. 나는 회원정보를 담고있는 member 라는 앱을 만들었다.

```bash
$ python manage.py startapp member
```

그리고 rest_server/settings.py 파일에 Installed App에 방금 생성한 앱을 추가해준다.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework_swagger',
    'member'
]
```

## DB 세팅

이제 API를 이용해 접근할 수 있는 DB를 만들어준다.
member/models.py 파일을 수정해준다.

```python
from django.db import models

class Member(models.Model):
    name = models.CharField(max_length=200)
    mail = models.CharField(max_length=200)
    age = models.IntegerField(default=0)
```

모델을 수정한 경우 Django에서는 makemigration과 migrate를 해주어야 한다.
다음 명령어로 실행할 수 있다.

```bash
$ python manage.py makemigrations member
$ python manage.py migrate member
```

이어서 생성한 모델에 값을 입력해준다.
장고는 터미널 상에서 manage.py shell 기능으로 편하게 DB를 수정할 수 있다.

```bash
$ python manage.py shell
>>> from member.models import Member
>>> Member.objects.create(name='tester', mail='test@test.com', age=20)
<Member: Member object (1)>
>>> Member.objects.create(name='tester2', mail='test2@test.com', age=22)
<Member: Member object (2)>
```

## API 뷰 만들기

이제 API요청에 따라 응답을 해주는 뷰를 작성해야 한다.
우선 member 폴더 하단에 api.py를 만들어주고 다음 내용을 작성한다.

Seriallizer는 API를 통한 요청에 대한 응답의 형태를을 결정해주는 클래스이다.
ViewSet은 요청을 처리하여 응답을 해주는 클래스이다. 추후에 이곳에 get, post, put, patch, delete에 대한 액션을 지정해주는 것이 가능하다.

```python
from .models import Member
from rest_framework import serializers, viewsets

class MemberSerializer(serializers.ModelSerializer):

    class Meta:
        model = Member
        fields = '__all__'

class MemberViewSet(viewsets.ModelViewSet):
    queryset = Member.objects.all()
    serializer_class = MemberSerializer
```


그리고 rest_server/urls.py 파일을 열어 api에 접근할 수 있도록 수정해준다.

```python
from django.conf.urls import url, include
from django.contrib import admin
from rest_framework import routers
from rest_framework_swagger.views import get_swagger_view

import member.api

app_name='member'

router = routers.DefaultRouter()
router.register('members', member.api.MemberViewSet)

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api/doc', get_swagger_view(title='Rest API Document')),
    url(r'^api/v1/', include((router.urls, 'member'), namespace='api')),
]
```

## 결과 확인
이제 장고를 실행하고 API요청을 테스트해본다.
다음 명령어로 장고를 실행해볼 수 있다.

```bash
$ python manage.py runserver 0.0.0.0:8000
```

이제 브라우저를 켜고 `http://localhost:8000/api/v1/members/`주소로 접속해보면 다음과 같은 응답을 얻을 수 있다.

![API members 요청 결과](/files/django_images/api_res.png)

`http://localhost:8000/api/doc/`로 접속하면 Swagger로 자동 작성된 API 문서를 확인할 수 있다.

![API 문서](/files/django_images/api_doc.png)