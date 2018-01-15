---
layout: post
title: 'Django fixture json파일 만들고 불러오기'
author: through.kim
date: 2016-03-29 13:49
tags: [python, study, django]
comments: true
---

- app의 post데이터를 fixture로 만들기

```python
>>> python manage.py dumpdata app.Post --indent 4 > post.json
```

`--indent` 옵션을 이용하면 생성된 json파일이 좀 더 깔끔하게 출력된 것을 확인할 수 있다.  
(생략도 가능하다.)

- post.json 픽스쳐를 DB로 불러오기

```python
>>> python manage.py loaddata post.json
```

위 명령어를 사용하면 해당 json파일을 DB로 불러올 수 있다.