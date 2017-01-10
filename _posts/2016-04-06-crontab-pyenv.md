---
layout: post
title: 'Python Script를 Crontab으로 실행하기'
author: through.kim
date: 2016-04-06 09:45
tags: [python, study, linux]
---

> 학식을 스크래핑해서 페이스북에 포스팅하는 봇을 제작중인데, 서버에 올려놓고 Crontab을 활용해 자동으로 실행되게 하고자 한다. (우분투, Python3.4, Pyenv-virtualenv환경을 기준으로 작성됨)

## Python 파일 세팅

우선 실행하고자 하는 파이썬 파일 제일 윗 칸에 파이썬 인터프리터의 경로를 입력해야 한다.  
우분투 기본 환경의 파이썬을 사용하고 있다면 `#!/usr/bin/python`을 입력하면 된다. 나의 경우는 pyenv-virtualenv환경에서 python3.4.1을 사용하고 있으므로 경로를 찾아서 스크립트 제일 위에 입력해주면 된다.

```python
python
>>> import sys
>>> sys.path
['', '/root/.pyenv/versions/3.4.1/lib/python34.zip', '/root/.pyenv/versions/3.4.1/lib/python3.4' ...]
>>> exit()
```

bash창에서 `python`을 입력하면 파이썬 스크립트 창으로 들어가진다. (만약 파이썬 가상환경의 path를 얻고자 한다면 가상환경을 실행하고 `python` 스크립트 창으로 들어가야 한다.) `sys`를 `import`하고 `sys.path`를 입력해보면 경로들이 나타난다. 공통적으로 `/root/.pyenv/versions/3.4.1/~`을 가리키고 있다는 것을 볼 수 있다. 실제로 그 경로로 찾아가보면 내가 찾던 `python3.4.1`인터프리터는 `/root/.pyenv/versions/3.4.1/bin/python3.4`에 위치한다는 것을 찾아낼 수 있다. 이 값을 실행하고자하는 파이썬 파일의 가장 윗 줄에 다음과 같이 입력해주면 된다.  

```python
#!/root/.pyenv/versions/3.4.1/bin/python3.4

import datetime
...

```

## Crontab 세팅

우분투의 crontab은 `crontab -e`를 통해 세팅해줄 수 있다.

```bash
crontab -e

SHELL =  /bin/bash
PATH = /sbin:/bin:/usr/sbin:/usr/bin
HOME = /

0 11 * * * /path/to/python/python3.4 path/to/project/python_to_execute.py
```

crontab은 실행될 때 별도의 환경변수를 가져오지 않으므로 위와 같이 기본적인 내용을 설정해줘야 한다.
실행 될 내용은 순서대로 `분 | 시 | 일 | 월 | 요일 | 시행할 작업`이다. 시행할 작업에서 파이썬 인터프리터를 실행해 주어야 파이썬 스크립트를 제대로 실행해줄 수 있다.
