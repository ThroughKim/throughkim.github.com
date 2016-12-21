---
layout: post
title: 'Python Facebook SDK를 활용해 페이지에 글 쓰기'
author: through.kim
date: 2016-04-04 23:17
tags: [python, study, facebook]
---

Python과 Facebook API를 사용해 글을 써 주는 프로그램을 제작하고자 한다. 일단은 기초적인 API를 사용해 페이지에 '테스트'라고 적힌 포스팅만 게시하는 프로그램을 만들어 보고, 추후에 원래 목적이었던 학식을 파싱해서 올리는 것으로 완성하려고 한다.

> Python 3.4.1, Mac OS X 기반으로 쓰인 글임

## 페이지 만들기
 - 개인 담벼락이나 그룹이 아닌 페이지에 글을 포스팅하기를 원하기 때문에 페이지를 만들어야 한다. 
 - 임의의 페이지를 만들고 `정보` 창에서 `Facebook 페이지 ID` 를 복사하여 메모해둔다.

## 페이스북 앱 만들기
 - [Facebook Developer](https://developers.facebook.com) 사이트로 이동하여 개발자로 등록을 한다.
 - [Facebook Apps Dashboard](https://developers.facebook.com/apps) 로 이동하여 `Add New App`을 클릭한다.
 - `WWW`를 클릭하고, 임의의 앱 이름을 설정해준 뒤 `Create New Facebook App ID`버튼을 클릭한다.
 - `Category`만 설정해 준 뒤 `Create App ID`버튼을 클릭한다.
 - `Quick Start`창이 뜨는데 무시하고 다시 [Facebook Apps Dashboard](https://developers.facebook.com/apps)로 이동한다.
 - 생성된 앱 클릭 -> `Settings` -> Contact Email을 작성한다.
 - `App Review`탭으로 이동하여 `Make 앱이름 public?`을 yes로 바꿔준다.
 - 위의 작업을 해야 앱을 통해 발행된 포스트를 모든 사람이 볼 수 있다.(그렇지 않으면 자신에게만 보인다고 한다.)
 - 다시 `Settings`탭으로 돌아가 `App ID`와 `App Secret`을 메모해둔다.

## 토큰 발행받기
 - [Graph API Explorer](https://developers.facebook.com/tools/explorer/)로 이동한다.
 - 우측 상단에 `Graph API Explorer`드롭다운을 클릭하고, 방금 만들어준 앱을 선택한다.
 - 밑에 있는 `Get Token` 드롭다운을 클릭하고, `Get Page Access Token`을 클릭하고, Permission을 해준다.
 - 다시 드롭다운메뉴를 클릭하고 포스팅하고자 했던 페이지를 선택한다. 
 - 발행된 토큰을 메모해둔다.

## 파이썬 프로그램 작성하기
 - `Facebook Python SDK`를 사용하기 위해 `pip install facebook-sdk`를 입력해 인스톨해준다.

```python
import facebook

graph = facebook.GraphAPI('페이지 토큰')
graph.put_object("페이지 ID", "feed", message='포스팅하고자하는 말')
```
 - 위와 같은 파이썬 프로그램을 작성하고 실행해준다.
 - 페이지에 들어가 원하는 말이 제대로 포스팅 되었는지 확인한다.


## 앞으로...
기본적으로 작성만 할 수 있는 프로그램을 제작했다. 그런데, 발급 받은 토큰은 임시토큰이고, 금방 만료가 되므로 프로그램을 실행했을 때 토큰을 발행받을 수 있는 코드를 짜는 작업이 필요하다. 그리고 앞선 포스팅에서 BeautifulSoup를 이용해 학식 테이블을 스크래핑 해오는 방법을 익혔으므로, 그 코드를 응용해 요일별로 포스트를 작성해 포스팅하는 것 까지 완성해야 할 것이다.