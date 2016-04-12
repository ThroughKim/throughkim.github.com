---
layout: post
title: Facebook Page Permanent Access Token 얻기
---

> Facebook 페이지에 자동으로 게시를 하는 봇을 만들다가, Access Token가 일정 시간 뒤에 만료가 되는 문제점이 생겼다. 구글링을 해보니 페이지 Access Token의 경우 영구 Access Token을 발행받을 수 있고, 이 방법을 상세히 설명해놓은 자료가 있어 한글로 번역, 기록해본다.

## 1. 유저 Short-lived 토큰 얻기

1. [Graph API Explorer](https://developers.facebook.com/tools/explorer)로 이동한다.
2. 토큰을 받고자 하는 어플리케이션을 드롭다운 메뉴에서 선택한다.(Explorer 우측상단에 위치)
3. `Get Access Token`버튼을 클릭한다.
4. 팝업창에서 `Extended Permissions`을 클릭하고, `manage_pages`를 선택한다.
5. `Get Access Token`버튼을 클릭한다.
6. Access Token필드에 나타난 값은 Short-lived토큰이다. 기록해두자.

## 2. Long-lived 토큰 생성하기

```
https://graph.facebook.com/v2.2/oauth/access_token?grant_type=fb_exchange_token&client_id={app_id}&client_secret={app_secret}&fb_exchange_token={short_lived_token}
```

해당 값에 어플리케이션 ID와 Secret 그리고 방금 발행받은 Short-lived 토큰을 넣고 주소창에 입력해 준다. 그러면 결과값이 아래와 같은 형태로 나타난다.

```
access_token=ABC123&expires=5182959
```

_(비슷한 원리로 Long-lived Access Token을 생성해주는 [사이트](http://www.slickremix.com/facebook-60-day-user-access-token-generator/)도 있다.)_

여기서 `ABC123`부분이 Long-lived Access Token에 해당한다. 이 토큰을 [Access Token Debugger](https://developers.facebook.com/tools/debug/)에 복사하여 붙여넣기 하여 토큰의 유효성을 검사해볼 수 있다. Expire부분에 토큰이 만료되기까지 남은 기한이 표시된다.

## 3. User Id 갖고오기

위에서 얻은 Long-lived Access Token을 이용해 다음과 같은 GET 리퀘스트를 보낸다.

```
https://graph.facebook.com/v2.2/me?access_token={long_lived_access_token}
```

여기서 얻은 `ID`필드의 값이 User ID이다. 바로 다음 단계에서 필요할 것이다.

## 4. 영구적인 Page Access Token얻기

아래와 같은 GET 리퀘스트를 보낸다.

```
https://graph.facebook.com/v2.2/{account_id}/accounts?access_token={long_lived_access_token}
```

`account_id`부분에 방금 전에 얻은 User ID를 입력하고, 아까 얻은 Long lived access token도 함께 넘겨준다.
JSON형태의 response가 return되는데, 여기서 원하는 앱의 Access Token부분을 기록해두면 된다. [Access Token Debugger](https://developers.facebook.com/tools/debug/)에 얻어낸 토큰을 입력해보면 Expire부분에 **Never**라고 표시되는 것을 확인할 수 있다.

---

원문: [stackoverflow facebook: permanent access token?](http://stackoverflow.com/questions/17197970/facebook-permanent-page-access-token)

