---
layout: post
title: Pyenv + VirtualEnv로 파이썬 가상 개발환경 구축하기
---


>프로젝트를 진행하다보면 같은 파이썬 프로젝트라도 요구되는 파이썬 버전이나 사용하는 프레임워크의 버전이 다를 수 있다. 이런 문제를 해결해줄 수 있는 것이 pyenv와 Vvrtualenv이다. Mac OS X 기준으로 pyenv와 virtualenv를 setup 하고 설정하는 방법에 대해 기록해본다.


- pyenv: 로컬에 다양한 파이썬 버전을 설치하고 사용할 수 있도록 한다. pyenv를 사용함으로써 파이썬 버전에 대한 의존성을 해결할 수 있다.
- virtualenv: 로컬에 다양한 파이썬 환경을 구축하고 사용할 수 있도록 한다. 일반적으로 Python Packages라고 부르는 ( pip install을 통해서 설치하는 ) 패키지들에 대한 의존성을 해결할 수 있다.



##Pyenv 설치 및 실행

```bash
user$ brew update
user$ brew install pyenv
```

HomeBrew를 통해 `pyenv`를 설치한다. HomeBrew는 Mac OS X를 위한 패키지 관리자이다.
HomeBrew가 설치되어있지 않다면 아래 내용을 터미널에 입력하여 HomeBrew설치를 진행한다.

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

```bash
user$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
user$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```

위의 내용을 입력하여 터미널을 시작할 때 `pyenv`를 실행하도록 설정해준다. 

```bash
user$ pyenv version
system (set by /Users/username/.pyenv/version)
user$ pyenv versions
* system
  3.4.0
  3.4.1
```

`pyenv version`을 입력하면 현재 사용중인 Python의 버전이 출력된다. `pyenv versions`를 입력하면 설치되어있는 python의 버전들이 출력된다.

```bash
user$ pyenv install 3.4.1
```
python의 특정 버전을 install하기 위해서 위의 명령어를 입력하면 된다.

```bash
user$ pyenv shell 3.4.1
user$ pyenv version
* 3.4.1
  system
  3.4.0
```
설치된 python의 특정 버전을 사용하기 위해서는 위와 같이 `shell`명령어를 사용하면 된다.

##Virtualenv 설치 및 실행
`virtualenv`를 사용하게 되면, 프로젝트에 사용하게 되는 python package에 대한 의존성도 해결할 수 있다.

```bash
user$ brew install pyenv-virtualenv
```
위의 명령어를 입력하여 `pyenv-virtualenv`를 설치한다.

```bash
user$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
```
위의 명령어를 입력하여 터미널을 시작할 때 `virtualenv`를 실행하도록 한다.

```bash
user$ pyenv virtualenv 3.4.1 my_project_pyenv_3.4.1
user$ pyenv versions
* 3.4.1
  system
  3.4.0
  my_project_pyenv_3.4.1
```
`pyenv virtualenv PYTHON_VERSION VIRTUAL_ENV_NAME`을 입력하여 가상 파이썬 개발환경을 설정할 수 있다.
`pyenv versions`명령어를 통해 설정된 개발환경을 확인할 수 있다.

```bash
user$ pyenv shell my_project_pyenv_3.4.1
pyenv-virtualenv: activate my_project_pyenv_3.4.1
(my_project_pyenv_3.4.1) user$
```

`pyenv shell`명령어를 통해 가상환경을 실행할 수 있다. 성공적으로 실행이 되었다면 입력 라인의 괄호 안에 가상환경의 이름이 입력되게 된다.`(VIRTUAL_ENV_NAME)user$ ` 여기서 부터 `pip install`등의 명령어를 이용해 파이썬 패키지를 설치해주면 개발을 위한 가상환경 setup이 완료되게 된다.



- - -

참고

[pyenv + virtualenv + autoenv 를 통한 Python 개발 환경 구축하기](https://dobest.io/how-to-set-python-dev-env/)  
[GitHub - yyuu/pyenv: Simple Python version management](https://github.com/yyuu/pyenv)  
[python:pyenv [권남] - 권남이 홈페이지](http://kwonnam.pe.kr/wiki/python/pyenv)
