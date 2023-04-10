# Django Static files

<br>

## 개요
- `Static Files`
  - 서버 측에서 변경되지 않고 고정적으로 제공되는 파일(이미지, JS, CSS 파일 등)

- 웹 서버와 정적 파일
  - 웹 서버의 기본 동작은
    - 특정 위치(URS)에 있는 자원을 요청(HTTP request) 받아서
    - 응답(HTTP response)을 처리하고 제공(serving)하는것
  - 자원에 접근 가능한 주소가 있다. 라는 의미
  - 웹서버는 요청 받은 URL로 서버에 존재하는 정적자원을 제공
  - 정적 파일을 제공하기 위한 경로(URL)가 있어야 함

## Static files 제공하기
- 경로에 따른 Static file 제공하기
  - 기본 경로
    - app/static/
    - articles/static/articles 경로에 이미지 파일 배치
    - static tag를 사용해 이미지 파일에 대한 url 제공
    - ```html
      <!-- articles/index.html -->
      {%load static %}

      <img src="{% static 'articles/sample-1.png' %}" alt="img">
      ```
  - 추가 경로
    - STATICFILES_DIRS
    - 추가 경로에 이미지 파일 배치
    - ```python
      # settings.py
      STATIC_URL = '/static/'

      STATICFILES_DIRS = [
          BASE_DIR / 'static',
      ]
      ```
    - static tag를 사용해 이미지 파일에 대한 url 제공
    - ```html
      <!-- articles/index.html -->
      <img src="{% static 'sample-2.png' %}" alt="img">
      ```
  - `STATICFILES_DIRS`
    - 정적 파일의 기본 경로 외에 추가적인 경로 목록을 정의하는 리스트

- 정적 파일을 제공하기 위해서는 요청할 URL이 필요하다
    

- `STATIC_URL`
  - 기본 경로 및 추가 경로에 위치한 정적 파일을 참조하기 위한 URL
  - 실제 파일이나 디렉토리가 아니며, URL로만 존재
  - 비어 있지 않은 값을 설정한다면 반드시 '/'로 끝나야함


## Media files
- `Media files`
  - 사용자가 웹에서 업로드하는 정적 파일 (user-uploaded)

- `imageField()`
  - 이미지 업로드에 사용하는 모델 필드
  - 이미지 객체가 직접 저장되는 것이 아닌 '이미지 파일의 경로 문자열'이 DB에 저장

- 미디어 파일을 제공하기 전 준비
  - 1 - settings.py에 MEDIA_ROOT, MEDIA_URL 설정
  - 2 - 작성한 MEDIA_ROOT와 MEDIA_URL에 대한 url 지정

- `MEDIA_ROOT`
  - 미디어 파일들이 위치하는 디렉토리의 절대 경로
  - ```python
    # settings.py
    MEDIA_ROOT = BASE_DIR / 'media'
    ```

- `MEDIA_URL`
  - MEDIA_ROOT에서 제공되는 미디어 파일에 대한 주소를 생성 (STATIC_URL과 동일한 역할)
  - ```python
    # settings.py
    MEDIA_URL = '/media/'
    ```

- MEDIA_ROOT와 MEDIA_URL에 대한 url 지정
  - 업로드 된 파일의 URL == settings.MEDIA_ROOT
  - 위 URL을 통해 참조하는 파일의 실제 위치 == settings.MEDIA_ROOT
  - ```python
    # crud.urls.py
    from django.conf import settings
    from django.conf.urls.static import static

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('articles/', include('articles.urls')),
    ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    ```


## 이미지 업로드 및 제공하기
- 이미지 업로드
  - blank=True 속성을 작성해 빈 문자열이 저장될 수 있도록 설정
    - ```python
      # articles/models.py
      class Article(models.Model):
        title = models.CharField(max_length=10)
        content = models.TextField()
        image = models.ImageField(blank=True)
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
      ```
  - 기존 필드 사이에 작성해도 실제 테이블 생성 시 가장 뒤에 추가됨
  - migration 진행
    - imageField를 사용하려면 반드시 Pillow 라이브러리가 필요함
  - form 요소의 enctype 속성 추가
    - ```html
      <!-- articles/create.html -->
      <h1>Create</h1>
      <form action="{% url 'articles:create' %}" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit">
      </form>
      ```
  - view 함수에서 업로드 파일에 대한 추가 코드 작성
    - ```python
      # articles/view.py
      def create(request):
        if request.method == 'POST':
            form = ArticleForm(request.POST, request.FILES)
      ...
      ```
  - 이미지 업로드 결과 확인
    - db.sqlite3 들어가 imags에 데이터가 들어갔는지 확인

- 업로드 이미지 제공하기
  - url 속성을 통해 업로드 파일의 경로 값을 얻을 수 있음
    - ```html
      <!-- articles/detail.html -->
      <img src="{{ article.image.url }}" alt="img">
      ```
    - article.image.url - 업로드 파일의 경로
    - article.image - 업로드 파일의 파일 이름
  - 업로드 출력 확인 및 MEDIA_URL 확인
  - 이미지를 업로드하지 않은 게시물은 detail 템플릿을 출력할 수 없는 문제 해결
  - 이미지 데이터가 있는 경우만 이미지를 출력할 수 있도록 처리
    - ```html
      <!-- articles/detail.html -->
      {% if article.image %}
      <img src="{{ article.image.url }}" alt="img">
      {% endif %}
      ```

- 업로드 이미지 수정
  - 수정 페이지 form 요소에 enctype 속성 추가
    - ```html
        <!-- articles/update.html -->
        <h1>Update</h1>
        <form action="{% url 'articles:update' article.pk %}" method="POST" enctype="multipart/form-data">
          {% csrf_token %}
          {{ form.as_p }}
          <input type="submit" value="UPDATE">
        </form>
      ```
  - view 함수에서 업로드 파일에 대한 추가 코드 작성
      - ```python
        # articles/view.py
        def update(request, article_pk):
          article = Article.objects.get(pk=article_pk)
          if request.method == 'POST':
              form = ArticleForm(request.POST, request.FILES, instance=article)
        ...
        ```

## 참고
- `'upload_to'  argument`
  - ImageField()의 upload_to 인자를 사용해 미디어 파일 추가 경로 설정
  - ```python
      # 1
      image = models.ImageField(blank=True, upload_to='imags/')
      # 2
      image = models.ImageField(blank=True, upload_to='%Y/%m/%d/')
      # 3
      def articles_image_path(instance, filename):
        return f'images/{instance.user.username}/{filename}'
      image = models.ImageField(blank=True, upload=to=articles_image_path)
    ```

- request.FILES가 두번째 위치 인자인 이유
  - ModelForm 상위 클래스의 생성자 함수 참고