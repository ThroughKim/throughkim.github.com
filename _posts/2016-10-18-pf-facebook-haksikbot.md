---
layout: post
title: '[Project] Facebook 학식봇'
author: through.kim
date: 2016-10-18 16:25
tags: [project]
---

>교내 학생식당의 식단을 편리하게 확인할 수 있도록 Facebook에 자동으로 글을 게시하는 Bot을 제작했습니다.

---

* Language/Framework : Python/Django
* Library : FacebookSDK, BeautifulSoup, PyVirtualDisplay
* Server : nCloud, Ubuntu 14.04.4 LTS
* 기타 : Git, Vim, PyCharm, Crontab
* [소스코드](https://github.com/ThroughKim/haksikbot)

---

![facebook학식봇메인](/assets/images/fbhaksik_main.png)

매일 정해진 시간에 이미지와 같은 학생식당의 식단을 이미지로 페이스북에 업로드하도록 만들었습니다.

![facebook학식봇 이미지](/assets/images/fbhaksik_image.png)

BeautifulSoup로 학식 메뉴 페이지를 파싱해 HTML테이블에 값을 입력한 뒤 PyVirtualDisplay를 이용해 캡쳐한 이미지를 생성하고 Cron을 이용해 매일 정해진 시간에 Facebook에 포스팅하는 방식으로 구현했습니다.