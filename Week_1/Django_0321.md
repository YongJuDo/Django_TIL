# Django design pattern

<br>

## Django App 생성 & 등록 과정

1. 앱 생성
  - `pyhton manage.py startapp articles`
  - 앱의 이름은 '복수형'으로 지정하는 것을 권장

2. 앱 등록
    - ```python
      INSTALLED_APPS = [
      'articles', # articles 앱 등록
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      ]
      ```
    - 반드시 앱을 생성한 후에 등록
    - 반대로 등록 후 생성은 불가능

## Django 디자인 패턴

1. 디자인 패턴
    - 소프트웨어 설계에서 발생하는 문제를 해결하기 위한 일반적인 해결책

2. MVC 디자인 패턴
    - (Modle-View-Controller)
    - 애플리케이션을 구조화하는 대표적인 패턴
    - 시각적 요소와 뒤에서 실행되는 로직을 서로 영향없이, 독립적이고 쉽게 유지보수할 수 있는
      애플리 케이션을 만들기 위해 사용

    - 2.1 Model
      - 데이터와 관련된 로직을 관리
      - 응용프로그램의 데이터 구조를 정의하고 데이터베이스의 기록을 관리

    - 2.2 View
      - Model & Template과 관련된 로직을 처리해서 응답을 반환
      - 클라이언트의 요청에 대해 처리를 분기하는 역할
    
    - 2.3 Template
      - 레이아웃과 화면을 처리
      - 화면상의 사용자 인터페이스 구조와 레이아웃을 정의

3. 프로젝트 구조
    - ```settings.py```
      - 프로젝트 모든 설정 관리
    - ```urls.py```
      - URL과 이에 해당하는 적절한 views를 연결
    - ```__init__.py```
      - 해당 폴더를 패키지로 인식하도록 설정
    - ```asgi.py```
      - 비동기식 웹 서버와의 연결 관련 설정
    - ```wsgi.py```
      - 웹 서버와의 연결 설정 관리
    - ```manage.py```
      - Django 프로젝트와 다양한 방법으로 상호 작용 하는 커맨드라인 유틸리티

4. 앱구조
    - ```admin.py```
      - 관리자용 페이지 설정
    - ```models.py```
      - DB와 관련된 model을 정의
      - MTV 패턴의 M
    - ```views.py```
      - HTTP 요청을 처리하고 해당 요청에 대한 응답을 반환(URL, Model, template과 연계)
      - MTV 패턴의 V
    - ```apps.py```
      - 앱의 정보가 작성된 곳
    - ```tests.py```
      - 프로젝트 테스트 코드를 작성하는 곳


## 요청과 응답

1. URLs
    - http://128.0.0.1:8000/articles/ 로 요청이 왔을때 view모듈의 index를 호출
    - ```python
      from django.contrib import admin
      from django.urls import path
      # urls.py 입장에서는 articles라는 패키지에서 view라는 모듈을 가져와야 함
      from articles import views

      urlpatterns = [
          path('admin/', admin.site.urls),
          path('articles/', views.index),
      ]
      ```

2. View
    - 특정 경로에 있는 template과 request 객체를 결합, 응답 객체를 반환하는 index view
      함수를 정의
    - 모든 view 함수는 첫 번째 인자로 요청 객체를 필수적으로 받음
    - ```python
        from django.shortcuts import render

      # Create your views here.

      def index(request):
          return render(request, 'articles/index.html')
      ```

3. Template
    - articles 앱 폴더안에 templates 폴더 생성
    - templates 폴더 안에서 템플렛 페이지 생성 및 작성

### 데이터 흐름에 따른 코드 작성
  -  URLs => View => Template


## render 함수
- 주어진 템플릿을 주어진 context 데이터와 결합하고 렌더링된 텍스트와 함께
  HttpResponse(응답) 객체를 반환하는 함수
- ```python
  render(requset, template_name, context)
  ```

1. requset
    - 응답을 생성하는 데 사용되는 요청 객체
2. template_naem
    - 템플릿 이름 경로
3. context
   - 템플릿에서 사용할 데이터(딕셔너리 타입으로 작성)

