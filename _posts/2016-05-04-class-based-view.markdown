---
layout: post
title: Django Class-based View
---

> Django에는 Class-based view라는 강력한 기능이 있는데, 이를 공부 하기 위해 [Documentation](https://docs.djangoproject.com/ja/1.9/topics/class-based-views/intro/)을 번역해본다.
  
Class-based view는 view를 Python의 function 대신 object의 형태로 상속받을 수 있는 방법을 제시한다. 이것은 function-based view를 대체하는 것이 아니지만, function-based view와 비교했을 때 확실한 차이점과 이점이 존재한다. 

 - 특정 HTTP 메소드(GET,POST,etc)와 관련된 코드들을 조건을 이용해 분기처리 하는 대신 메소드의 형태로 사용할 수 있다.
 - Mixin(다중상속)과 같은 객체지향 테크닉을 활용해 재사용가능한 컴포넌트의 형태로 코드를 짤 수 있다.

---

### Generic views, class-based views, class-based generic view의 관계와 역사  

최초의 view는 function방식 밖에 없었다. Django는 HttpRequest를 당신의 function으로 넘겨주고 HttpResponse를 돌려받기를 기대하며, 이것이 Django가 제공할 수 있는 것의 한계였다.  
  
초기에 view를 개발할 때 공통적인 패턴, 반복적 표현이 존재한다는 것을 발견했고, Function-based generic view가 이러한 패턴을 추상화하고, 공통적인 케이스의 개발을 더 쉽게 하기 위해 소개되었다.  
  
Function-based generic view의 문제점이 있었는데, 바로 간단한 케이스는 잘 커버하지만, 간단한 설정 옵션을 넘어선 customize나 extend가 불가능하다는 것이고, 이는 수많은 real world application에서 그 유용성을 제한하는 요인이 되었다.  
  
Class-based generic view는 function-based generic view와 마찬가지로 view의 개발을 더 쉽게 하기 위한 목적으로 만들어졌다. 그러나 mixin의 활용을 통해 class-based generic view는 function-based에 비해 더 extensible하고 flexible해질 수 있는 방법을 제공하고 있다.  
  
Django가 class-based generic view를 만들기 위해 사용하는 기본적 클래스들과 다중상속 같은 도구들은 flexibility를 최대화하기 위해 만들어졌고, 그것은 다양한 hook을 기본 메소드 상속(implementation)과 속성(attribute)의 형태로 가지고 있다.  

---
### Class-based view 사용하기

Class-based view는 당신이 단일 view function에서 조건 분기처리 코딩을 하는 대신에, 서로 다른 클래스 인스턴스 메소드를 이용해 서로 다른 HTTP request에 응답하도록 해준다.  
  
view function에서 HTTP __GET__을 처리하는 내용은 다음과 같다.  

```python
from django.http import HttpResponse

def my_view(request):
	if request.method == 'GET':
    	# <view logic>
        return HttpResponse('result')
```

Class-based view에서는 다음과 같다.

```python
from django.http import HttpResponse
from django.views.generic inport Vuew

class myView(View):
	def get(self, request):
    # <view logic>
    return HttpResponse('result')
```

Django의 URL resolver는 request와 그와 연관된 인자들을 class가 아닌 호출가능(callable)한 function으로 전송하기를 기대하고 있기 때문에, class-based view는 당신의 클래스로의 entry point인 __as_view()__라는 클래스 메소드를 가지고 있다. __as_view__ entry point는  당신의 class의 인스턴스를 생성하고 __dispatch()__메소드를 호출한다. __dispatch__는 request를 살펴보고 __GET__인지 __POST__인지 혹은 다른 형태인지 결정하고, 결정이 되면 request를 매칭되는 메소드로 전달해주며, 그렇지 않으면 __HttpResponsNotAllowed__ 에러를 발생시킨다.  

```python
# urls.py
from django.conf.urls import url
from myapp.views import MyView

urlpatterns = [
	url(r'^about/', MyView.as_view()),
]
```

당신의 메소드의 리턴값이 function-based view의 리턴값 즉, HttpResponse의 형태와 동일하다면 아무런 가치가 없다. 이것은 class-based view 안에서 http shortcut 혹은 TemplateResponse 객체를 사용할 때 의미가 있다는 것을 의미한다.  
  
최소한의 class-based view는 그 역할 수행을 위해 아무런 클래스 속성(class attribute)을 요구하지 않지만, 클래스 속성은 많은 class-based 디자인에서 유용하며, 클래스 속성을 설정하는 방법은 두가지가 있다.  
  
첫번째 방법은 표준 파이썬 방식으로 서브클래싱과 서브클래스 내에서의 속성과 메소드의 오버라이딩을 하는 방식이다. 만약 당신의 부모 클래스가 greeting이라는 속성을 다음과 같이 가진다면

```python
from django.http import HttpResponse
from django.views.generic import View

class GreetingView(View):
	greeting = "Good Day"
    
    def get(self, request):
    	return HttpResponse(self.greeting)
```

당신은 그것은 서브클래스에서 오버라이드할 수 있다.

```python
class MorningGreetingView(GreetingView):
	greeting = "Morning to ya"
```

두번째 방법은 URLconf내에서 __as_view()__를 호출할 때 클래스 속성을 키워드 인자를 이용해 설정해주는 것이다.

```python
urlpatterns = [
	url(r'^about/', GreetingView.as_view(greeting = "G'day")),
]
```

>__Note__  
>당신의 클래스는 각각의 요청 마다 인스턴스화 되지만, __as_view()__ entry point를 통한 클래스 속성 설정은 당신의 URL들이 import될 때 오직 한번만 설정된다.
  
---

### Mixin 사용하기

Mixin은 복수의 부모 클래스가 포함될 수 있는 행동(behavior)과 속성(attribute)의 다중상속의 형태이다.

예를들어 generic class-based view에는 __TemplateResponseMixin__이라고 불리는 mixin이 있는데, 그것의 주 목적은 __render\_to\_response()__ 메소드를 정의하는 것이다. __View__ 베이스 클래스의 행동(behavior)과 결합되어 있을때, __TemplateView__ 클래스가 request를 매칭 되는 적합한 메소드(__View__ 베이스 클래스에서 정의된 행동)로 request를 보내주며, 그것은 __TemplateResponse__ 객체(__TemplateResponseMixin__에서 정의된 행동)를 리턴하기 위해 __template_name__ 을 사용하는 __render\_to\_response()__ 메소드를 가지고 있다.

Mixin은 다중 클래스간의 코드 재사용을 하기 위한 훌륭한 방법이지만, 다소 비용이 든다. 당신의 코드가 mixin사이에 산재하면 할수록 자식클래스가 정확히 무슨 일을 하는지 알기가 더 어려워지고, 당신이 깊은 상속 트리를 가진 무언가를 서브클래싱 했다면, 어떤 메소드를 오버라이드 했는지 알아내기가 어려워진다.

오직 하나의 generic view에서만 상속받을 수 있다 - 다시말해 오직 하나의 부모 클래스만 View로부터 상속을 받고, 나머지는 mixin이어야만 한다. 하나 이상의 클래스를 View로부터 상속받고자 한다면 예상되로 작동되지 않을 것이다.

---

###Class-based view로 form 핸들링하기

기본적 function-based view는 form을 다음과 같이 핸들링한다.

```python
from django.http import HttpResponseRedirect
from django.shortcuts import render

from .forms import MyForm

def myview(request):
	if request.method == "POST":
    	form = MyFrom(request.POST)
        if form.is_vaild():
        	# <procss form cleaned data>
            return HttpResponseRedirect('/success/')
    else:
    	form = MyForm(initial={'key': 'value'})
        
    return render(request, 'form_template.html', {'form': form})
```

비슷한 class-based view는 다음과 같을 것이다.

```python
from django.http import HttpResponseRedirect
from django.shortcuts import render
from django.views.generic import view

from .forms import MyForm

class MyFormView(View):
	form_class = MyForm
    initial = {'key': 'value'}
    template_name = 'form_template.html'
    
    def get(self, request, *args, **kwargs):
    	form = self.form_class(initial=self.initial)
        return render(reqeust, self.template_name, {'form': form})
        
    def post(self, request, *args, **kwargs):
    	form = self.form_class(request.POST)
        if form.is_valid():
        	# <process form cleaned data>
            return HttpResponseRedirect('/success/')
        
        return render(request, self.template_name, {'form': form})
```

이것은 매우 간단한 케이스이지만, 클래스 속성 오버라이딩을 통해 이 view를 customizing할 수 있는 옵션을 볼 수 있을것이다. 예를들어 __form_class__를 URLconf 설정이나 서브클래싱을 통해서 하나 혹은 그 이상의 메소드를 오버라이딩 할 수 있다.

---

### class-based view 데코레이팅

class-based view의 확장(extension)은 mixin사용에만 국한되지 않는다. 당신은 데코레이터(decorator)도 사용할 수 있다. class-based view가 function이 아니기 때문에, 데코레이딩을 하는 방법은 __as_view()__를 사용하는지 혹은 서브클래스 생성하는지 여부에 따라 상이해진다.

#### URLconf에서 데코레이팅하기

class-based view를 데코레이팅 하는 가장 쉬운 방법은 __as_view()__함수의 결과를 데코레이트하는 것이다. 가장 쉬운 장소는 당신의 view를 배치하는 URLconf이다.

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.views.generic import TemplateView

from .views import VoteView

urlpatterns = [
	url(r'^about/', login_required(TempateView.as_view(template_name="secret.html"))),
    url(r'^vote/', permission_required('polls.can_vote')(VoteView.as_view())),
]
```
이러한 접근방법은 데코레이터를 인스턴스화 될 때 한번만 적용시키게 된다. 만약 당신이 view의 모든 인스턴스마다 데코레이팅을 하고 싶다면 다른 접근방법을 사용해야 할 것이다.

#### Class 데코레이팅 하기

Class-based view의 모든 인스턴스마다 데코레이팅을 하고 싶다면, 당신은 클래스 자체를 데코레이팅할 필요가 있다. 이것을 하기 위해서 당신은 클래스의 __dispatch()__ 메소드에 데코레이터를 적용하면 된다.

클래스의 메소드는 독립적인(standalone) function과 같지는 않다. 그렇기 때문에 당신이 단순히 function 데코레이터를 메소드에 적용시키는 것은 불가능하며, 데코레이터를 메소드 데코레이터로 변환시키는 작업이 필요하다. __method_decorator__는 function 데코레이터를 메소드 데코레이터로 변환시켜주어서 인스턴스 메소드에 사용할 수 있게해준다. 예시는 다음과 같다.

```python
from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator
from django.views.generic import TemplateView

class ProtectedView(TemplateView):
	template_name = 'secret.html'
    
    @method_decorator(login_required)
    def dispath(self, *args, **kwargs):
    	return super(ProtectedView, self).dispatch(*args, **kwargs)
```

혹은 더욱 간결하게, 당신은 클래스를 데코레이팅 하는 대신에 데코레이팅 될 메소드의 이름을 키워드 인자 __name__으로 넘겨주면 된다.

```python
@method_decorator(login_required, name='dispatch')
class ProtectedView(templateView):
	template_name = 'secret.html'
```

만약 당신이 여러 군데에 사용되는 일반적인 데코레이터의 모임을 가지고 있다면, 데코레이터의 리스트 혹은 튜플을 정의하고 __method_decorator()__를 여러번 사용하는 대신에 그것을 불러와서 사용할 수 있다. 다음 두 클래스는 동일하다.

```python
decorators = [never_cache, login_required]

@method_decorator(decorators, name='dispatch')
class ProtectedView(TemplateView):
	template_name = 'secret.html'
    
@method_decorator(never_cache, name='dispatch')
@method_decorator(login_required, name='dispatch')
class ProtectedView(TemplateView):
	template_name = 'secret.html'
```
decorators는 request를 기록 된 순서대로 데코레이터로 넘겨주어 처리할 것이다. 예시에서는 __never_cache()__가 __login_required()__보다 먼저 처리된다. 이 예시에서는 모든 ProtectedView의 인스턴스가 login protection을 가지게 될 것이다.

>__Note__  
>__method_decorator__는 __*args__와 __**kwargs__를 클래스의 데코레이팅 된 메소드에 파라미터로 넘겨준다. 만약 당신의 메소드가 받아들이는 형식이 파라미터와 일치하지 않는다면 __TypeError__를 발생시키게 될 것이다.