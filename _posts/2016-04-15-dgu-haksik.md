---
layout: post
title: '동국대학교 학식 알림이봇'
author: through.kim
date: 2016-04-15 13:56
tags: [facebook, python]
comments: true
---

> 스크래핑과 페이스북 API를 공부할 겸 동국대학교 학식을 스크랩 해와서 페이스북에 포스팅하는 프로그램을 만들었다.  
  

![학식알림이 스크린샷](/assets/images/haksikbot.png)  
  
![학식알림이 테이블](/assets/images/haksiktable.png)

처음엔 텍스트 형식으로 스크래핑 해온 값을 포스팅하려고 했지만, 단순 텍스트만 나오니 너무 단조로워서 테이블을 HTML형식으로 만들어서 출력하고, 그것을 서버상의 가상 디스플레이로 열어 스크린샷을 캡쳐하고, 그 스크린샷을 업로드 하는 방식으로 구동했다.

만든지 일주일 정도 되었는데 좋아요수가 130개 남짓 된다. 홍보를 하면 더 늘어날 것 같다.