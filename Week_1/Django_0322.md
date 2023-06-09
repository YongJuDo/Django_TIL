# Django template

<br>

## Template System
- django template system
  - 데이터 표현을 제어하면서, 표현과 로직을 담당

- DTL (django template Language)
  - Template에서 조건, 반복, 변수, 필터 등의 프로그래밍적 기능을 제공하는 시스템
  - DTL Syntax (Variable, Filters, Tags, Comments)
  1. Variable
      - view 함수에서 render 함수의 세번째 인자로 딕셔너리 타입으로 넘겨 받을 수 있음
      - 딕셔너리 key에 해당하는 문자열이 template에서 사용 가능한 변수명이 됨
      - dot(.)를 사용하여 변수 속성에 접근할 수 있음
      - ```python
        {{ variable }}
        ```
  2. Filters
      - 표시할 변수를 수정할 때 사용
      - chained가 가능하며 일부 필터는 인자를 받기도 함
      - 약 60개의 built-in template filters를 제공
      - ```python
        {{ variable|filter }}
        ```
  3. Tags
      - 반복 또는 논리를 수행하여 제어 흐름을 만드는 등 변수보다 복잡한 일들을 수행
      - 일부 태그는 시작과 종료 태그가 필요
      - 약 24개의 built-in template tags를 제공
      - ```python
        {% tag %}
        ```
  4. Comments
      - DTL에서의 주석 표현
      - ```python
        {# name #}
        ```

## 템플릿 상속(Template inheritance)
- 템플릿 상속이란?
  - 페이지의 공통요소를 포함하고, 하위템플릿이 재정의 할수 있는 공간을 정의하는 기본
    'skeleton' 템플릿을 작성하여 상속 구조를 구축하는 것
  - skeleton 역할 템플릿 작성
  - ```html
    <!-- skeleton template -->

    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Document</title>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-rbsA2VBKQhggwzxH7pPCaAqO46MgnOM80zW1RWuH61DGLwZJEdK2Kadq2F9CUG65" crossorigin="anonymous">
      {% block style %}
      {% endblock style %}
    </head>
    <body>
      {% block content %}
      {% endblock content %}
      <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-kenU1KFdBIe4zVF0s0G1M5b4hcpxyD9F7jL+jjXkk+Q2h455rYXK/7HAuoJl+0I4" crossorigin="anonymous"></script>
    </body>
    </html>
    ```
  - ```html
    <!-- {% extends 'articles/base.html' %} -->

    {% block content %}
      <h1>DTL 실습</h1>
      <h3>메뉴판</h3>
      <p>{{ foods }}</p>
      <ul>
        {% for food in foods %}
          <li>{{ food }}</li>
        {% endfor %}
      </ul>

      {% if foods|length == 0 %}
        <p>메뉴가 남아있지 않습니다.</p>
      {% else %}
        <p>메뉴가 아직 남아있습니다.</p>
      {% endif %}
      {% endblock content %}
    ```
  - ```extends``` tag
    - ```pyhton
      {% extends 'path' %}
      ```
    - 자식(하위)템플릿이 부모 템플릿을 확장한다는 것을 알림
    - 반드시 템플릿 최상단에 작성되어야 함 (2개 이상 사용 X)
  
  - ```block``` tag
    - ```pyhton
      {% block name %} {% endblock name %}
      ```
    - 하위 템플릿에서 재정의(overridden)할 수 있는 블록을 정의 (하위 템플릿이 작성할 수 있는 공간을 지정)

## 요청과 응답
  - 데이터를 보내고 가져오기
    - HTML form element를 통해 사용자와 애플리케이션 간의 상호작용 이해하기
    - HTML form은 HTTP 요청을 서버에 보내는 가장 편리한 방법
    - ```html
      <form action="" method="GET">
        <input type="text" name="massage">
        <input type="submit">
      </form>
      ```
  - ```form``` element
    - 사용자로부터 할당된 데이터를 서버로 전송
    - 웹에서 사용자 정보를 입력하는 여러 방식(text, password 등)을 제공
  - ```action``` & ```method```
    - form의 핵심속성 2가지
    - 데이터를 어디(action)로 어떤 방식(method)으로 보낼지

  - action
    - 입력된 데이터가 전송될 URL을 지정(목적지)
    - 만약 이 속성을 지정하지 않으면 데이터는 현재 form이 있는 페이지의 URL로 보내짐

  - method
    - 데이터를 어떤 방식으로 보낼것인지 정의
    - 데이터의 HTTP request menthods (GET, POST)를 지정
  
  - ```input``` element
    - 사용자의 데이터를 입력 받을 수 있는 요소 (type 속성 값에 따라 다양한 유형의 입력 데이터를 받음)

  - ```name``` element
    - input의 핵심 속성
    - 데이터를 제출했을 때 서버는 name 속성에 설정된 값을 통해 사용자가 입력한 데이터에 접근할 수있음

  - Qurey String Prameters
    - 사용자의 입력 데이터를 URL 주소에 파라미터를 통해 넘기는 방법
    - 문자열은 앰퍼샌드(&)로 연결된 key=value 쌍으로 구성되며, 기본 URL과 물음표(?)로 구분
    - EX) ```http://host:port/paht?key=value&key=value```

## 요청과 응답 활용
  - form 데이터는 어디에 들어 있을까?
    - 모든 요청 데이터는 HTTP request 객체에 들어있음 (view 함수의 첫번째 인자)

## DTL 주의사항
  - Python처럼 일부 프로그래밍(if, for 등)를 사용할 수 있지만 명칭을 그렇게 설계 했을 뿐
    Pyhton 코드로 실행되는 것은 아니며 Pyhton과 아무런 관련이 없음
  - 프로그래밍적 로직이 아니라 프레젠테이션을 표현하기 위한 것임을 명심
    - 프로그래밍적 로직은 되도록 view 함수에서 작성 및 처리