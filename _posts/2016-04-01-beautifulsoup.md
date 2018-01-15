---
layout: post
title: 'Beautifulsoup로 학식 테이블 스크래핑하기'
author: through.kim
date: 2016-04-01 21:53
tags: [python, study]
comments: true
---

> 처음 파이썬에 관심을 가지게 된 이유는 크롤러와 스크래퍼 때문이었다. 이번에는 BeutifulSoup를 활용해 동국대학교 학식 식단표를 스크래핑해본다.

_(Mac OS X, Python 3.4.1 환경에서 실행된 내용임)_


#### BeatifulSoup 설치
```bash
user$ pip install beautifulsoup4
```
위의 명령어를 입력하여 `BeutifulSoup`를 설치한다.

- - -


#### 스크래핑할 페이지 갖고오기

```python
from bs4 import BeautifulSoup
from urllib.request import urlopen

html = urlopen('http://dgucoop.dongguk.edu/store/store.php?w=4&l=2&j=0')
source = html.read()
html.close()

print(source)
```
우선 `BeautifulSoup`을 import하고 홈페이지를 얻어오기 위해 `urllib`의 `urlopen`을 import해온다. 그리고 [동국대학교 학식 식단표 페이지](http://dgucoop.dongguk.edu/store/store.php?w=4&l=2&j=0)의 주소를 갖고 오도록 `urlopen`을 사용한 뒤 가져온 내용을 출력해보는 코드를 짠다. 위와같이 임의의 파이썬 파일을 생성하고 실행해보면 복잡한 문자들이 나열되는 것을 볼 수 있다.

> 페이지 주소 뒤에 w,l,j 의 값을 바꿔 입력해보면 w와 j는 메뉴에 따른 request값이고, j는 0이면 이번주, 1이면 다음주 식단이 나타나는 것을 확인할 수 있다. 지난주 식단은 볼 수 없다(?)

```python
import..

html = urlopen('http://dgucoop.dongguk.edu/store/store.php?w=4&l=2&j=0')
source = html.read()
html.close()

soup = BeautifulSoup(source, "lxml")

print(soup)
```
아까 프린트해보았던 `source`를 `BeautifulSoup`에 넣고 그 내용을 출력해보면, 이전보다는 정제된 모습을 출력해주는 것을 확인할 수 있다. `lxml`을 인자로 넘겨주면, 해당 페이지 HTML 파싱을 `lxml parser`로 한다는 것을 의미한다.
(`pip install lxml` 필요)

- - -


#### 식단표 테이블 가져오기

```html
<div class="detail_board">
   <div id="sdetail">
        <table width="100%">
            <tr>
                ...
```
동국대학교 식단표 홈페이지의 소스를 확인해보면 위와 같이 식단표 테이블을 `sdetail`이라는 `id`를 가진 `div`가 감싸고 있음을 확인할 수 있다. 내게 필요한 내용은 식단표 내용밖에 없으니, 이 부분만 불러오도록 한다.

```python
import..

html = urlopen('http://dgucoop.dongguk.edu/store/store.php?w=4&l=2&j=0')
source = html.read()
html.close()

soup = BeautifulSoup(source, "lxml")
table_div = soup.find(id="sdetail")
tables = table_div.find_all("table")
menu_table = tables[1]
trs = menu_table.find_all('tr')

print(trs[2])
```
`BeautifulSoup`의 `find()`메소드는 인자로 넘겨준 값의 첫번째 결과값을 반환한다. 페이지 내에 `table`이 많다면 많은 테이블 중에 첫번째 테이블의 내용만 반환하는 것이다. `find_all()`메소드는 인자로 넘겨준 값의 모든 결과값을 리스트로 반환한다.  

따라서 위와 같은 코드를 입력후 파일을 실행해보면 `div`안의 모든 `table`이 출력됨을 볼 수 있다. 그런데, 식단표 페이지의 소스를 잘 보면 `div#sdetail`안에는 `table`이 두개 들어있다. 식단표 테이블은 두번째 테이블이므로  `tables[1]`에 해당한다.  

식단표 테이블 안에서 각 줄별로 정보를 가져와 `trs`라는 이름의 `list`에 저장한다. 이 파일을 실행해 보면 해당 행의 식단이 표시되는 것을 볼 수 있다.

```python
import..

html = urlopen('http://dgucoop.dongguk.edu/store/store.php?w=4&l=2&j=0')
source = html.read()
html.close()

soup = BeautifulSoup(source, "lxml")
table_div = soup.find(id="sdetail")
tables = table_div.find_all("table")
menu_table = tables[1]
trs = menu_table.find_all('tr')
sangrok_bakban_lunch = trs[8]
tds = sangrok_bakban_lunch.find_all('td')
mon = tds[3].span.string
tue = tds[4].span.string
wed = tds[5].span.string
thu = tds[6].span.string
fri = tds[7].span.string

print(mon, tue, wed, thu, fri)
```
이제 상록원 백반코너의 점심을 가져오는 코드를 짜본다. HTML로 돌아가보면 상록원이 들어있는 `<tr>`은 9번째에 위치한다. 따라서 `sangrok_bakban_lunch`라는 변수에 `trs[8]`을 지정해준다. 마찬가지로 각 요일별 메뉴가 몇번째 `<td>`에 위치하는지 확인하여 해당 값을 저장해주면 된다. 이렇게 저장된 값을 JSON으로 저장할 수도 있고, DB에 바로 저장해 활용해줄 수 있다.

