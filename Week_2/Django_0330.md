# Django ORM & View - 2

<br>

## HTTP request methods
- `redirect()`
  - 인자에 작성된 주소로 다시 요청을 보냄
  - ```python
    # create view 함수 수정
    # redirect 함수 적용
    from django.shortcuts import render, redirect

    def create(request):
  
      title = request.GET.get('title')
      content = request.GET.get('content')
      article = Article(title=title, content=content)
      article.save()

      return redirect('articles/create.html', article.pk)

    ```

- `HTTP`
  - 네트워크 상에서 데이터를 주고 받기위한 약속

- `HTTP request methods`
  - 데이터(리소스)에 어떤 요청(행동)을 원하는지를 나타내는 것
  - GET & POST

- `GET` Method
  - 특정 리소스를 조회하는 요청
  - GET으로 데이터를 전달하면 Quert String 형식으로 보내짐
  - http://127.0.0.1:8000:articles/create/?title=제목&content=내용
  - 반드시 데이털르 가져올 때만 사용

- `POST` Method
  - 특정 리소스에 변경사항을 만드는 요청
  - POST으로 데이터를 전달하면 HTTP Body 에 담겨 보내짐
  - http://127.0.0.1:8000:articles/create/?title=제목&content=내용

- `HTTP response status code`
  - 특정 HTTP 요청이 성공적으로 완료되었는지 알려줌
  - 5개의 그룹으로 나뉘어짐(1xx, 2xx, 3xx, 4xx, 5xx)

- `403 Forbidden`
  - 서버에 요청이 전달되었지만, 권한 때문에 거절되었다는 것을 의미

- `CSRF(Cross-Site-Request-Forgery)`
  - 사이트 간 요청 위조
  - 사용자가 자신의 의지와 무관하게 공격자가 의도한 행동을 하여 특정 웹 페이지를 보안에 취약하게 하거나 수정, 삭제 등의 작업을 하게 만드는 공격 방법

- `Security Token(CSRF Token)
  - 대표적인 CSRF 방어 방법
  1. 서버는 사용자 입력 데이터에 임의의 난수 값(token)을 부여
  2. 매 요청마다 해당 token을 포함시켜 전송 시키도록 함
  3. 이후 서버에서 요청을 받을 때마다 전달된 token이 유효한지 검증

  - DTL의 `csrf_token 태그`를 사용해 사용자에게 토큰 값을 부여 요청 시 토근 값도 함께 서버로 전송될 수 있도록 함
  - ```html
    {% csrf_token %}
    ```

  - POST Method는 데이터베이스에 대한 변경사항을 만드는 요청이기 때문에 토큰을 사용해 최소한의 신원 확인을 하는 것


## DELETE
- ```python
    # articles/urls.py

    urlpatterns = [
      ...
      ath('<int:pk>/delete/', views,delete, name='delete'),
    ]
  ```

- ```python
  # articles/views.py
  def create(request, pk):
    article = Article.object.get(pk=pk)
    article.delete()

    return redirect('articles:index')
  ```

- ```html
  <!-- articles/index.html -->
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <input type="submit" value='DELETE'>
  </form>
  ```
## UPDATE
- Update 로직을 구현하기 위해 필요한 view 함수
  - `edit`
    - 사용자의 입력을 받는 페이지 렌더링
  - `update`
    - 사용자가 입력한 데이터를 받아 DB에 저장
  
- edit 로직
  - ```python
    # articles/urls.py
      urlpatterns = {
        ...
       path('<int:pk>/edit/', views.edit, name='edit'),
      }
    ```
  - ```python
    # articles/views.py
    def edit(request, pk):
      article = Article.objects.get(pk=pk)
      context = {
          'article': article,
    }
    ```
  - ```html
    <!-- tmplates/articles/edit.html -->
    <h1>EDIT</h1>
    <form action="{% url 'articles:update' article.pk %}" method="POST">
      {% csrf_token %}
      <div>
        <label for="title">제목: </label>
        <input type="text" name="title" id="title" value="{{ article.title }}">
      </div>
      <div>
        <label for="content">내용: </label>
        <textarea name="content" id="content" cols="30" rows="10">{{ article.content }}</textarea>
      </div>
      <input type="submit" value="[UPDATE]">
    </form>
    <a href="{% url 'articles:index' %}">[back]</a>
    ```
- edit 페이지로 이동하기 위한 하이퍼 링크
  - ```html
    <!-- tmplates/articles/detail.html -->
    <a href="{% url 'articles:edit' article.pk %}">[Edit]</a>
    ```

- update 로직
  - ```python
    # articles/urls.py
      urlpatterns = {
        ...
       path('<int:pk>/update/', views.update, name='update'),
      }
    ```
  - ```python
    # articles/views.py
    def update(request, pk):
      article = Article.objects.get(pk=pk)
      article.title = request.POST.get('title')
      article.content = request.POST.get('content')
      article.save()

      return redirect('articles:detail', article.pk)
  - ```html
    <!-- tmplates/articles/edit.html -->
    <form action="{% url 'articles:update' article.pk %}" method="POST">
    ```