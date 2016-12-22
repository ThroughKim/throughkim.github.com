---
layout: post
title: 'Python BeautifulSoup 한글 인코딩'
author: through.kim
date: 2016-12-08 14:30
tags: [python]
image: '/files/covers/python-in-ex-machina.jpg'
---

>BeautifulSoup에서 갑자기 한글이 깨지는 오류 발생

오늘 오전 크롤링한 메뉴부터 한글이 갑자기 깨지는 오류가 발생했다.  

urllib에서 html문서를 읽어오는 과정에서 한글이 깨진 것을 확인했고  

다음과 같은 코드를 삽입하여 문제를 해결할 수 있었다.  

```python
html = urlopen(url).read().decode('cp949', 'ignore')
```