---
layout: post
title: '카카오톡 자동응답 API로 학식봇 구현'
author: through.kim
date: 2016-07-11 19:08
tags: [python, django, study, kakao]
---

> 카카오톡의 옐로아이디 자동응답 API로 동국대학교 학생식당의 메뉴를 알려주는 봇을 구현해보았다. Python3와 Django를 이용했으며, Mac OS X를 기반으로 작성했다.

### 구현된 모습

![완성된 카카오톡 자동응답 학식봇](/assets/images/kakaocomplete.png)

위의 사진과 같이 동국대학식봇을 친구로 추가하고, 대화창에서 해당 식당의 버튼을 클릭하면 메뉴를 출력해주는 방식으로 구현했다.  

---


### 시작 및 키보드 구현하기

우선 시작을 하기 위해선 옐로아이디 등록과 카카오 앱 등록이 필요한데, 이는 다른 블로그에도 많이 나와있으므로 생략한다.  
  
카카오 옐로아이디 앱을 등록하고, API형 자동응답을 하기 위해서는 일단 기본적으로 카카오 API중에서 `https://your_server_url/keyboard/`요청에 알맞게 반응하도록 해야한다.  
  
이제부터 Python과 Django를 이용해 해당 API 요청에 제대로 응답할 수 있도록 만들어주면 된다. 처음에는 REST Framework를 이용해 구현해보려 했지만, 단순 JSON Response만 해주면 되기 때문에 Django만을 이용해 구현했다.  
  
[옐로아이디 API Documentation](https://github.com/plusfriend/auto_reply)에 따르면  
  
```bash
curl -XGET 'https://:your_server_url/keyboard/'
```
  
으로 요청을 했을 때 아래와 같이 응답을 하면 된다고 명기되어있다.  
  
```python
{
"type" : "buttons",
"buttons" : ["선택 1", "선택 2", "선택 3"]
}
```
  
Django를 이용해 `https://서버주소/keyboard/`로 요청이 오면 위와 같은 JSON형태로 response해줄 수 있도록 세팅해주면 된다.  
  
우선 [Python 환경세팅](https://throughkim.github.io/2016/03/31/pyenv-virtualenv.html)을 해준다. 세팅이 완료되면  
  
```bash
(myenv) user$ pip install Django
```
  
을 입력하여 Djnago를 설치해준다.  
  
```bash
(myenv) user$ django-admin startproject myproject .
(myenv) user$ python manage.py startapp dguhaksik
```
  
임의의 프로젝트 이름(myproject)으로 장고 프로젝트를 생성해준다(뒤에 . 을 반드시 붙여야 함)  
그리고 프로젝트 폴더에 들어가 실질적으로 동국대학교 학식봇을 구현할 dguhaksik이라는 이름의 앱을 만들어준다.  
  
이어서 장고의 settings.py파일에 dguhaksik이라는 앱을 추가해준다.  
  
```python
~/myproject/settings.py

~~

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'dguhaksik',
]

~~

```
  
그리고 urls.py에 요청받을 주소를 만들어준다. 여기서는 `https://서버주소/keyboard/`가 된다.  
  
```python
~/myproject/urls.py

from django.conf.urls import url

urlpatterns = [
    url(r'^keyboard/', 'dguhaksik.views.keyboard'),
]
```
  
`~/keyboard/` 주소로 요청이 오면 dguhaksik앱의 view파일의 keyboard와 연결을 하겠다는 의미로 보면 된다.  
이제 실제로 dguhaksik앱의 views.py를 편집하여 알맞은 값을 return하도록 코딩해주면 된다.  
  
```python
~/dguhaksik/views.py

from django.http import JsonResponse

def keyboard(request):

    return JsonResponse({
        'type' : 'buttons',
        'buttons' : ['상록원', '그루터기', '아리수', '기숙사식당', '교직원식당']
    })
```
  
[옐로아이디 API Documentation](https://github.com/plusfriend/auto_reply)에 나와있는 양식에 맞춰 자신이 원하는 이름의 키보드 버튼들을 생성해주면 된다.
  
이제 터미널 창에서 테스트 해보면,  
  
```bash
user$ curl -XGET 'https://:your_server_url/keyboard/'
{"buttons": ["\uc0c1\ub85d\uc6d0", "\uadf8\ub8e8\ud130\uae30", "\uc544\ub9ac\uc218", "\uae30\uc219\uc0ac\uc2dd\ub2f9", "\uad50\uc9c1\uc6d0\uc2dd\ub2f9"], "type": "buttons"}
```
  
이렇게 키보드 값이 리턴되는 것을 볼 수 있다.  
이 상태에서 [옐로아이디 페이지](https://yellowid.kakao.com/)에 접속하여 `API TEST`버튼을 클릭하면 다음과 같이 나타난다.

![카카오 API TEST 성공한 모습](/assets/images/kakaoapitest.png)

Required되는 내용은 방금 작성한 /keyboard/요청에 대한 응답이고, Optional에 해당하는 부분은 구현하지 않았기 때문에 오류가 발생한 것을 확인할 수 있다. API테스트의 Required부분을 통과했기 때문에, 하단에 서비스 시작 버튼을 클릭하여 API형 자동응답을 실행할 수 있다.  

![카카오 키보드 구현모습](/assets/images/kakaokeyboard.png)

API형 자동응답 서비스를 시작 뒤에 자신이 등록한 옐로아이디와의 대화창에 들어가보면 위와 같이 등록한 키보드 버튼들이 나타나는 것을 확인할 수 있다. 하지만 아직 버튼을 눌렀을때의 반응에 대해 세팅해주지 않았기 때문에 버튼을 클릭해도 오류만 발생하는 것을 볼 수 있을 것이다.  
  
---

###버튼에 따른 Response구현하기

원하는 결과는 버튼을 클릭했을 때 해당 학생식당의 식단이 출력되는 것이지만, 아직 크롤러를 구현하지 않았기 때문에 단순히 _"'\~\~'식당의 0월0일 식단입니다."_ 라는 메세지만 우선 출력해보도록 한다.  
[옐로아이디 API Documentation](https://github.com/plusfriend/auto_reply)에 따르면 사용자가 버튼을 클릭하면 POST방식으로 요청내용을 JSON에 담아서 보내는 것을 볼 수 있다.  
  
```bash
curl -XPOST 'https://:your_server_url/message' -d '{
  "user_key": "encryptedUserKey",
  "type": "text",
  "content": "상록원"
}'
```

필요로 하는 내용은 `content`에 해당하는 부분이며 내용에 따라 분기처리를 하여 답변을 할 수 있도록 해주면 된다. 우선 요청이 `https://:your_server_url/message`로 오기 때문에 urls.py를 수정해주도록 하자.  
  
```python
~/myproject/urls.py

from django.conf.urls import url

urlpatterns = [
    url(r'^keyboard/', 'dguhaksik.views.keyboard'),
    url(r'^message', 'dguhaksik.views.answer'),
]
```
  
message뒤에 '/'가 붙지 않아야 한다. 이어서 dguhaksik앱에서 views.py파일에 answer라는 함수를 만들어야 하는데, 그에 앞서 [옐로아이디 API Documentation](https://github.com/plusfriend/auto_reply)에서 response의 유형을 확인해야 한다.  
  
```bash
{
  "message": {
    "text": "귀하의 차량이 성공적으로 등록되었습니다. 축하합니다!",
    "photo": {
      "url": "https://photo.src",
      "width": 640,
      "height": 480
    },
    "message_button": {
      "label": "주유 쿠폰받기",
      "url": "https://coupon/url"
    }
  },
  "keyboard": {
    "type": "buttons",
    "buttons": [
      "처음으로",
      "다시 등록하기",
      "취소하기"
    ]
  }
}
```
  
문서에 따르면 message부분은 Required이고, Keyboard부분은 Optional이다. 그러나 Keyboard를 설정해주지 않으면, 한 곳의 식당을 확인한 뒤에 버튼들이 더이상 나타나지 않는 문제가 생기므로, keyboard부분에 앞서 작성했던 키보드 버튼 내용들을 작성해주어야 할 것이다. 그리고 message부분에 text형식으로 원하는 내용을 입력해주면 된다.  
이제 dguhaksik 앱의 views.py파일을 수정한다.
  
```python
~/dguhaksik/views.py

from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse
import json, datetime

def keyboard(request):
    .....

@csrf_exempt
def answer(request):
    json_str = ((request.body).decode('utf-8'))
    received_json_data = json.loads(json_str)
    cafeteria_name = received_json_data['content']
    today_date = datetime.date.today().strftime("%m월 %d일")
    
    return JsonResponse({
            'message': {
                'text': today_date + '의 ' + cafeteria_name + ' 중식 메뉴입니다.'
            },
            'keyboard': {
                'type': 'buttons',
                'buttons': ['상록원', '그루터기', '아리수', '기숙사식당', '교직원식당']
            }

        })
```
  
POST 방식을 사용하기 때문에 Django에서는 CSRF Token 에러가 발생하며, `@csrf_exempt`를 이용해 에러가 발생하지 않도록 해야 한다. 이어서, 요청에서 JSON 객체를 추출하여 원하는 값을 얻어내야 한다. python의 json모듈을 사용하면 쉽게 할 수 있다. `request.body`를 utf-8로 디코딩 해주고, json을 로딩한 뒤에 content에 해당하는 내용을 따로 분리하면, 사용자가 클릭한 버튼의 내용을 얻어낼 수 있다. 날짜를 나타내는 부분은 datetime모듈을 이용해 쉽게 구현할 수 있다.  
  
```bash
user$ curl -XPOST 'http://your_server_url/message' -d '{"user_key": "encryptedUserKey", "type": "text", "content": "상록원"}'
{"message": {"text": "07\uc6d4 13\uc77c\uc758 \uc0c1\ub85d\uc6d0 \uc911\uc2dd \uba54\ub274\uc785\ub2c8\ub2e4."}, "keyboard": {"type": "buttons", "buttons": ["\uc0c1\ub85d\uc6d0", "\uadf8\ub8e8\ud130\uae30", "\uc544\ub9ac\uc218", "\uae30\uc219\uc0ac\uc2dd\ub2f9", "\uad50\uc9c1\uc6d0\uc2dd\ub2f9"]}}
```
  
저장해 준뒤에 터미널 창에서 CURL을 이용해 요청을 보내면 위와같은 결과가 나타날 것이다.  

![response 완성된 모습](/assets/images/kakaoresponse.jpg)

카카오톡에서 테스트해보면 위 스크린샷과 같은 결과가 나타난다.
  
---
  
### 크롤러 구현 및 완성하기

이제 남은 내용은 식당 이름에 맞추어 식단을 출력해주는 것이다. 그러기위해서 식단표를 크롤링하고 DB에 저장한 뒤, 사용자의 요청이 오면 해당식당의 DB내용을 출력해주는 방식으로 구현했다. 크롤러를 만드는 방식은 다음 참고 문서를 확인해보면 될 것이다. ([크롤링 참고](http://throughkim.kr/2016/04/01/beautifulsoup/))  
크롤링해서 나온 결과값을 DB에 저장해야 하는데, 나는 Django의 기본 DB인 Sqlite를 사용했다. 우선 dguhaksik앱의 models.py파일을 수정하여 DB를 생성해주자.  
  
```python
~/dguhaksik/models.py

from django.db import models

class Menu(models.Model):
    id = models.AutoField(primary_key=True)
    cafe_name = models.CharField(max_length=30, default="")
    time = models.CharField(max_length=30, default="")
    menu = models.CharField(max_length=100, default="")
```
  
단순히 id와 식당이름, 중식인지 석식인지 여부를 판단할 time, 그리고 메뉴이름과 가격을 저장할 menu필드를 생성했다. 매번 크롤링 될때마다 DB를 Flush하고 최신값만 저장하도록 설정할 것이므로, 시간에 관련된 필드는 만들지 않았다. 모델을 수정했으므로, Django에 적용시켜줘야 한다.  
  
```bash
(myenv) user$ python manage.py makemigrations
(myenv) user$ python manage.py migrate
```
  
이제 크롤러를 매일 정해진 시간에 자동으로 작동하도록 해야하는데, 다양한 방법이 있겠지만 나의 경우는 Django의 특정 url로 요청이 오면 크롤러가 작동하도록 하고, 리눅스의 Cron기능을 이용해 매일 정해진 시간에 해당 url을 호출하도록 했다._(별로 좋지 않은 방식인 것 같다 ㅜㅜ)_ 따라서 장고의 urls.py를 수정해주도록 한다.  
  
```python
~/myproject/urls.py

from django.conf.urls import url

urlpatterns = [
    url(r'^keyboard/', 'dguhaksik.views.keyboard'),
    url(r'^message', 'dguhaksik.views.answer'),
    url(r'^crawl/', 'dguhaksik.views.crawl'),
]
```
  
이제 url호출에 맞춰서 크롤러를 작동시킬 수 있도록 views.py를 수정해주도록 한다.  
  
```python
~/dguhaksik/views.py

from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse
from dguhaksik.models import Menu
from bs4 import BeautifulSoup
from urllib.request import urlopen
import json, datetime

def keyboard(request):
    .....

@csrf_exempt
def answer(request):
    .....
    
def crawl(request):
    menu_db = Menu.objects.all()
    menu_db.delete()
    
    (크롤러 구현 부분 - 너무 길어 생략)
    
def create_menu_db(cafe_name, time, menu):
    Menu.objects.create(
        cafe_name=cafe_name,
        time=time,
        menu=menu
        )
```
  
url을 통해 `crawl()`을 호출하면 우선 Menu모델을 불러와 안에 내용을 모두 지운다(항상 최신상태만 유지하기 위해). 이어서 크롤러를 구현하고, 해당 식당의 이름, 중식 or 석식, 메뉴와 가격을 표시하는 String을 `create_menu_db()`라는 메소드에 인자로 넘겨주고, 해당 메소드에서 DB에 저장하도록 했다.  
  
마지막으로 DB에 저장된 식단을 사용자 요청에 맞춰 출력해주도록 세팅해주면 된다. 계속해서 views.py를 수정하자.  
  
```python
~/dguhaksik/views.py

from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse
from dguhaksik.models import Menu
from bs4 import BeautifulSoup
from urllib.request import urlopen
import json, datetime

def keyboard(request):
    .....

@csrf_exempt
def answer(request):
    json_str = ((request.body).decode('utf-8'))
    received_json_data = json.loads(json_str)
    cafeteria_name = received_json_data['content']
    today_date = datetime.date.today().strftime("%m월 %d일")
    
    return JsonResponse({
            'message': {
                'text': today_date + '의 ' + cafeteria_name + ' 중식 메뉴입니다. \n \n' + get_menu(cafeteria_name)
            },
            'keyboard': {
                'type': 'buttons',
                'buttons': ['상록원', '그루터기', '아리수', '기숙사식당', '교직원식당']
            }

        })
    
def crawl(request):
    .....
    
def create_menu_db(cafe_name, time, menu):
    .....
```
  
answer부분에서 리턴하는 값에 `get_menu()`라는 메소드를 호출하도록 하고, 학생식당의 이름을 인자로 넘겨 해당 식당의 메뉴를 가져올 수 있도록 했다. 계속해서 views.py에 `get_menu()`라는 메소드를 추가해준다.  
  
```python
~/dguhaksik/views.py

.....

def get_menu(cafeteria_name):
    if cafeteria_name == '상록원':
        sang_bek = Menu.objects.get(cafe_name='백반코너').menu
        sang_ill = Menu.objects.get(cafe_name='일품코너').menu
        sang_yang = Menu.objects.get(cafe_name='양식코너').menu
        sang_dduk = Menu.objects.get(cafe_name='뚝배기코너').menu

        return "------------\n" +  "백반코너 \n" + sang_bek \
               + "------------\n" + "일품코너 \n" + sang_ill \
               + "------------\n" + "양식코너 \n" + sang_yang \
               + "------------\n" + "뚝배기코너 \n" + sang_dduk

    elif cafeteria_name == '아리수':
        .....

```
  
이런식으로 학생식당의 이름에 맞게 분기처리를 하여 식당의 메뉴를 불러올 수 있도록 해주면 된다. 저장해준 뒤 카카오톡에서 실행 해보면 아래 사진과 같이 식당의 메뉴들이 잘 출력되는 것을 확인할 수 있다.

![완성](/assets/images/kakaocomplete.png)
