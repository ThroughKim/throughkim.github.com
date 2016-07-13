---
layout: post
title: 카카오톡 자동응답 API로 학식봇 구현
---

> 카카오톡의 옐로아이디 자동응답 API로 동국대학교 학생식당의 메뉴를 알려주는 봇을 구현해보았다. Python3와 Django를 이용했으며, Mac OS X를 기반으로 작성했다.

### 구현된 모습

![완성된 카카오톡 자동응답 학식봇](/images/kakaocomplete.png)

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
(myenv) user$ cd myproject
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

![카카오 API TEST 성공한 모습](/images/kakaoapitest.png)

Required되는 내용은 방금 작성한 /keyboard/요청에 대한 응답이고, Optional에 해당하는 부분은 구현하지 않았기 때문에 오류가 발생한 것을 확인할 수 있다. API테스트의 Required부분을 통과했기 때문에, 하단에 서비스 시작 버튼을 클릭하여 API형 자동응답을 실행할 수 있다.  

![카카오 키보드 구현모습](/images/kakaokeyboard.png)

API형 자동응답 서비스를 시작 뒤에 자신이 등록한 옐로아이디와의 대화창에 들어가보면 위와 같이 등록한 키보드 버튼들이 나타나는 것을 확인할 수 있다. 하지만 아직 버튼을 눌렀을때의 반응에 대해 세팅해주지 않았기 때문에 버튼을 클릭해도 오류만 발생하는 것을 볼 수 있을 것이다.  
  
---

###버튼에 따른 Response구현하기

원하는 결과는 버튼을 클릭했을 때 해당 학생식당의 식단이 출력되는 것이지만, 아직 크롤러를 구현하지 않았기 때문에 단순히 _"'~~'식당의 0월0일 식단입니다."_ 라는 메세지만 우선 출력해보도록 한다.  
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
  
message뒤에 '/'가 붙지 않아야 한다. 이어서 dguhaksik앱에서 views.py파일에 answer라는 함수를 만들어야 하는데, 그에 앞서 [옐로아이디 API Documentation](https://github.com/plusfriend/auto_reply)에서 reponse의 유형을 확인해야 한다.  
  
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
  
문서에 따르면 message부분은 Required이고, Keyboard부분은 Optional이다. 그러나 Keyboard를 설정해주지 않으면, 한 곳의 식당을 확인하면, 버튼들이 더이상 나타나지 않는 문제가 생기므로, keyboard부분에 앞서 작성했던 키보드 버튼 내용들을 작성해주어야 할 것이다. 그리고 message부분에 text형식으로 원하는 내용을 입력해주면 된다.  
  
```python
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
                'text': today_date + '의 교직원식당 중식 메뉴입니다.'
            },
            'keyboard': {
                'type': 'buttons',
                'buttons': ['상록원', '그루터기', '아리수', '기숙사식당', '교직원식당']
            }

        })
```




