---
layout: post
title: '[Project] Twenty CMS'
author: through.kim
date: 2016-10-18 16:25
tags: [project]
---

>대학생을 위한 미디어 플랫폼 트웬티(TWENTY)에서 인턴으로 근무하며 콘텐츠 매니징 시스템을 개발했습니다.

---

* Language/Framework : Python/Django
* DB : MySQL, Django ORM
* Server : nCloud, uCloud, Ubuntu 14.04.4 LTS
* Charts : D3.js
* 기타 : TDD, Git, Vim, PyCharm

---

제가 맡은 부분은 서비스 사용 로그를 분석해 통계를 내리는 DASHBOARD부분과 콘텐츠 관리페이지, 개별 콘텐츠 통계분석페이지였습니다.  

![main](/assets/images/twenty_main_page.png)

메인 Dashboard이며, 사용자가 로그인 했을 때 현재 서비스의 상태를 직관적으로 확인할 수 있도록 하고자 만들었습니다.  

![postlist](/assets/images/twenty_postlist.png)

콘텐츠 매니징 시스템의 콘텐츠 리스트 부분입니다.  
트웬티 어플리케이션에 매일 발행되는 뉴스 콘텐츠를 관리할 수 있는 페이지입니다.  
콘텐츠가 많아지면서 어플리케이션의 성능이 떨어지는 문제점이 발생했습니다.  
때문에, 쿼리와 페이지네이터 개선을 통해 성능을 높였던 경험이 있습니다.  

![postview](/assets/images/twenty_postview.png)

개별 콘텐츠의 내용과 통계를 볼 수 있는 페이지입니다.  