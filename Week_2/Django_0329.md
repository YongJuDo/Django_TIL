# Django ORM & View

<br>

## ORM UPDATE
- 데이터 수정
  - ```python
    # 수정할 인스턴스 조회
    article = Article.objects.get(pk=1)
    
    # 인스턴스 변수를 변경
    article.title = 'byebye'

    # 저장
    article.save()

    # 정상적으로 변경된 것을 확인
    article.title
    ```

## ORM DELETE
- 데이터 삭제
  - ```python
    # 삭제할 인스턴스 조회
    article = Article.objects.get(pk=1)
    
    # delete 메서드 호출 (삭제 된 객체가 반환)
    article.delete()

    # 삭제한 데이터는 더이상 조회 불가능
    Article.objects.get(pk=1)
    ```

## 사전 준비
- app URLs 분할 및 연결
  - ```python
    # articles/urls.py
    from django.urls import path

    app_name = 'articles'
    urlpatterns = [
    ]
    ```
  - ```python
    # crud/urls.py
    from django.urls import path, include
    form django.contrib import admin

    urlpatterns = [
      path('admin/', admin.site.urls),
      path('articles/', include('articles.urls')),
    ]
    ```

- index 페이지 작성
  - ```python
    # articles/urls.py
    from django.urls import path
    from . import views

    app_name = 'articles'
    urlpatterns = [
      path('', view.index, name='index')
    ]
    ```
  - ```python
    # articles/view.py
    
    def index(request):
      retrun render(request, 'articles/index.html')
    ```
  - ```html
    <!-- articles/index.html -->
    <!-- body 태그 생략 -->
    <h1>Articles</h1>
    ```

## READ
  - 전체 게시글 조회
    - ```python
      # articles/views.py
      from .models import Article

      def index(request):
        articles = Article.objects.all()
        context = {
          'articles': articles,
        }
        return render(request, 'articles/index.html', context)
      ```
    - ```html
      <!-- articles/index.html -->
      <h1>Articles</h1>
      <hr>
      {% for article in articles %}
        <p>글 번호: {{ article.pk }}</p>
        <p>글 제목: {{ article.title }}</p>
        <p>글 내용: {{ article.content }}</p>
        <hr>
      {% endfor %}
      ```

  - 단일 게시글 조회
    - ```python
      # articles/urls.py
      
      urlpatterns = {
        ...
        path('<int:pk>/', views.detail, name='detail'),
      }
      ```
    - ```python
      # articles/views.py
      from .models import Article

      def detail(request):
        articles = Article.objects.get(pk=pk)
        context = {
          'articles': articles,
        }
        return render(request, 'articles/detail.html', context)
      ```
    - ```html
      <!-- tmplates/articles/detail.html -->
      <h2>DETAIL</h1>
      <h3>{{ article.pk }} 번째 글</h3>
      <p>제목: {{ article.title }}</p>
      <p>내용: {{ article.content }}</p>
      <p>작성 시각: {{ article.created_at }}</p>
      <p>수정 시각: {{ article.updated_at }}</p>
      <hr>
      <a href="{% url 'articles:index' %}">[back]</a>
      ```
  - 제목을 누르면 해당 글의 상세 페이지로 이동
    - ```html
      <!-- tmplates/articles/detail.html -->
      <h1>Articlese</h1>
      <hr>
      {% for article in articles %}
      <p>글 번호: {{ article.pk }}</p>
      <a href="{% url 'articles:detail' article.pk %}">
        <p>글 제목: {{ article.title }}</p>
      </a>
      <p>글 내용: {{ article.content }}</p>
      <hr>
      {% endfor %}
      ```


## CREATE
  - Create 로직을 구현하기 위해 필요한 view 함수
    - `new`
      - 사용자의 입력을 받는 페이지 렌더링
    - `create`
      - 사용자가 입력한 데이터를 받아 DB에 저장
  
  - new 로직 작성
    - ```python
      # articles/urls.py
      
      urlpatterns = {
        ...
        path('new/', views.new, name='new'),
      }
      ```
    - ```python
      # articles/views.py
      from .models import Article

      def new(request):
        return render(request, 'articles/new.html')
      ```
    - ```html
      <!-- tmplates/articles/new.html -->
      <h1>NEW</h1>
      <form action="#" method="GET">
        <div>
          <label for="title">Title: </label>
          <input type="text" name="title" id="title">
        </div>
        <div>
          <label for="content">Content: </label>
          <textarea name="content" id="content"></textarea>
        </div>
        <input type="submit">
      </form>
      <hr>
      <a href="{% url 'articles:index' %}">[back]</a>
      ```
  - new 로직 작성
    - new 페이지로 이동할 수 있는 하이퍼링크 작성
    - ```html
      <!-- tmplates/articles/index.html -->
      <h1>Articles</h1>
      <a href="{% url 'articles:new' %}">NEW</a>
      <hr>
      ```

  - create 로직 작성
    - ```python
      # articles/urls.py
      
      urlpatterns = {
        ...
        path('create/', views.create, name='create'),
      }
      ```
    - ```python
      # articles/views.py
      from .models import Article

      def create(request):
        title = request.GET.get('title')
        content = request.GET.get('content')

        # 1.
        # aricle = Article()
        # article.title = title
        # article.content = content
        # article.save()

        # 2.
        # 저장 전에 유효성 검사와 같은 추가 작업을  위해 2번 방법을 택함
        article = Article(title=title, content=content)
        article.save()

        # 3.
        # Article.objects.create(title=title, content=content)


        return render(request, 'articles/create.html')
      ```
    - ```html
      <!-- tmplates/articles/create.html -->
      <h1>NEW</h1>
      <form action="{% url 'articles:create' %}" method="GET">
        <div>
          <label for="title">Title: </label>
          <input type="text" name="title" id="title">
        </div>
        <div>
          <label for="content">Content: </label>
          <textarea name="content" id="content"></textarea>
        </div>
        <input type="submit">
      </form>
      <hr>
      <a href="{% url 'articles:index' %}">[back]</a>
      ```