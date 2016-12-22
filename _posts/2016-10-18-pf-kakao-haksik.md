---
layout: post
title: '[Project] KakaoTalk 학식봇'
author: through.kim
date: 2016-10-18 16:25
tags: [project]
---

>카카오톡 자동응답 API로 학생식당 메뉴를 알려주는 어플리케이션을 개발했습니다.

---

* Language/Framework : Python/Django
* Library : BeautifulSoup
* DB : MySQL, Django ORM
* Server : nCloud, Ubuntu 14.04.4 LTS
* 기타 : Git, Vim, PyCharm, Crontab
* [소스코드](https://github.com/ThroughKim/kakaohaksik)

---

![카톡봇 메인](/assets/images/kakao_main.jpeg)

카카오톡 자동응답 API를 이용해 파싱되어 DB에 저장된 학식 메뉴를 알려주는 어플리케이션 입니다.  
Python BeutifulSoup를 이용해 학생식당 메뉴를 파싱하고, DB에 값을 저장한 뒤 URI를 통해 요청이 오면 DB에 저장된 값을 Return하도록 제작했습니다.

제작과정을 상세하게 기록한 [포스팅](http://throughkim.github.io/2016/07/11/kakao-haksik/)을 작성했습니다.