# Django Handling http requests

<br>

## 개요
- new & create view 함수간 공통점과 차이점
  - 공통점
    - 데이터 생성 로직을 구현하기 위함
  - 차이점
    - new는 GET method 요청만을
    - create는 POST method 요청만을 처리

## view 함수의 변화
- new와 view 함수 결합
  - 기존
  - ```python
    def new(request):
      form = ArticleForm()
      context = {
          'form': form,
      }
      return render(request, 'articles/new.html', context)
    ```
  - ```python
    def create(request):
      form = ArticleForm(request.POST)
      if form.is_valid():
          article = article.save()
          return redirect('articles:detail', article.pk)
      context = {
          'form': form,
      }
      return render(request, 'articles/new.html', context)
    ```
  - 결합
  1. request 객체의 method 값을 사용한 분기
  2. POST일 때는 과거 create 함수의 로직 처리
  3. POST가 아닐 때는 과거 new 함수의 로직 처리
  - ```python
    def create(request):
      # HTTP request method가 POST 라면
      if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = article.save()
            return redirect('articles:detail', article.pk)
      # POST 아니라면
      else:
        form = ArticleForm()
      context = {
          'form': form,
      }
      return render(request, 'articles/new.html', context)

- new url 제거
  - 불필요해진 new url 제거
  - ```python
    app_name = 'articles'
    urlpatterns = [
        path('', views.index, name='index'),
        # path('new/', views.new, name='new'),
        path('create/', views.create, name='create'),
        path('<int:pk>/', views.detail, name='detail'),
        path('<int:article_pk>/edit/', views.edit, name='edit'),
        path('<int:artilce_pk>/delete/', views.delete, name='delete'),
        path('<int:article_pk>/update/', views.update, name='update'),
    ]
      ```

- 기존 new 관련 코드 수정
  - ```html
    <!-- articles/index.html -->
    <h1>Articles</h1>
    <a href="{% url 'articles:create' %}">[NEW]</a>
    {% for article in articles %}
      <p>제목: 
        <a href="{% url 'articles:detail' article.pk %}">{{ article.title }}</a>
      </p>
      <p>내용: {{ article.content }}</p>
      <hr>
    {% endfor %}
    ```
  - ```html
    <!-- articles/create.html -->
    <h1>CREATE</h1>
    <form action="{% url 'articles:create' %}" method="POST">
      {% csrf_token %}
      {{ form.as_p }}
      <input type="submit">
    </form>
    ```
  - ``` python
    def create(request):
      # HTTP request method가 POST 라면
      if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = article.save()
            return redirect('articles:detail', article.pk)
      # POST 아니라면
      else:
        form = ArticleForm()
      context = {
          'form': form,
      }
      return render(request, 'articles/create.html', context)
    ```

- `GET`
- articles/create/
    - 게시글 생성 페이지를 줘!

- `POST`
  - articles/create/
    - 게시글을 생성해줘!

- 새로운 update view 함수
  - ```python
    # ariticles/views.py
    def update(request, article_pk):
    article = Article.objects.get(pk=article_pk)
    if request.mehtod == 'POST':
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            article.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm(instance=article)
    context = {
        'form': form,
        'article': article,
    }
    return render(request, 'articles/update.html', context)
    ```
- edit url 정리
  - 불필요해진 edit url 제거
  - ```python
    app_name = 'articles'
    urlpatterns = [
        path('', views.index, name='index'),
        # path('new/', views.new, name='new'),
        path('create/', views.create, name='create'),
        path('<int:pk>/', views.detail, name='detail'),
        # path('<int:article_pk>/edit/', views.edit, name='edit'),
        path('<int:artilce_pk>/delete/', views.delete, name='delete'),
        path('<int:article_pk>/update/', views.update, name='update'),
    ]
      ```
- 기존 edit 관련 코드 수정
  - ```html
    <!-- articles/detail.html -->
    <h1>Detail</h1>
    <p>글 번호: {{ article.pk }}</p>
    <p>제목: {{ article.title }}</p>
    <p>내용: {{ article.content }}</p>
    <p>작성일: {{ article.created_at }}</p>
    <p>수정일: {{ article.updated_at }}</p>
    <a href="{% url 'articles.update' article.pk %}">UPDATE</a><br>
    <form action="{% url 'articles:delete' article.pk  %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="삭제">
    </form>
    ```
  - ```html
    <h1>New</h1>
      <form action="{% url 'articles:update' article.pk %}" method="POST">
        {% csrf_token %}
        {{ form.as_p }}
    <input type="submit">
    </form>
    ```