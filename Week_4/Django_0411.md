# Django Many to one relationships - 1

<br>

## 개요
- `Foreign Key`
  - 테이블의 필드 중 다른 테이블의 레코드를 식별할 수 있는 키
  - 각 레코드에서 서로 다른 테이블 간의 '관계'를 만드는 데 사용


## Comment & Article
### 1. 모델 관계 설정
- `Many to one relationships (N:1 or 1:N)`
  - 한 테이블의 0개 이상의 레코드가 다른 테이블의 레코드 한 개와 관련된 관계

- `Comment(N) - Article(1)`
  - 0개 이상의 댓글은 1개의 게시글에 작성 될 수 있다

- `ForeignKey()`
  - django에서 N:1 관계 설정 모델 필드

- Comment 모델 정의
  - ForeignKey() 클래스의 인스턴스 이름은 참조하는 모델 클래스 이름의 단수형(소문자)으로 작성하는 것을 권장
  - ForeignKey() 클래스를 작성하는 위치와 관계없이 필드 마지막에 생성됨
  - ```python
    # articles/models.py
    class Comment(models.Model):
      article = models.ForeignKey(Article, on_delete=models.CASCADE)
      content = models.TextField()
      created_at = models.DateTimeField(auto_now_add=True)
      updated_at = models.DateTimeField(auto_now=True)
    ```

- `ForeignKey(to, on_delete)`
  - to - 참조하는 모델 class 이름
  - on delete - 참조하는 모델 class가 삭제 될 때 연결된 하위 객체의 동작을 결정

- `on_delete`
  - 외래키가 참조하는 객체(1)가 사라졌을 때, 외래키를 가진 객체(N)를 어떻게 처리할 지를 정의하는 설정 (데이터 무결성)
  - "CASCADE" 부모 객체(참조 된 객체)가 삭제 됐을 때 이를 참조하는 객체도 삭제

- migration 진행 후 Comment 테이블 확인
  - article_id 필드 확인
  - 참조하는 클래스 이름의 소문자(단수형)로 작성하는 것이 권장 되었던 이유

- 댓글 생성 연습 - 1
  - shell_plus 실행 및 게시글 작성
  - ```python
    # 게시글 작성
    Article.objects.create(title='title', content='content')
    ```

- 댓글 생성 연습 - 2
  - 댓글 생성
  - ```python
    # Comment 클래스의 인스턴스 comment 생성
    comment = Comment()

    # 인스턴스 변수 저장
    comment.content = 'first comment'

    # DB에 댓글 저장
    comment.save()

    # 에러 발생
    django.db.utils.IntegrityError: NOT NULL constraint failed: articles_comment.article_id
    # articles_comment 테이블의 ForeignKeyId, article_id 값이 저장 시 누락되었기 때문
    ```

- 댓글 생성 연습 - 3
  - 댓글 생성
  - ```python
    # 게시글 조회
    article = Article.objects.get(pk=1)

    # 외래 키 데이터 입력
    comment.article = article
    # 또는 comment.article_id = article.pk 처럼 pk 값을 직접 외래 키 컴럼에 넣어 줄 수 도 있지만 권장하지 않음

    # DB에 댓글 저장 및 확인
    comment.save()
    ```

- 댓글 생성 연습 - 4
  - comment 인스턴스를 통한 article 값 접근
  - ```python
    comment.pk
    => 1

    comment.content
    => 'first comment'

    # 클래스 변수명인 article로 조회 시 해당 참조하는 게시물 객체를 조회할 수 있음
    - comment.article
    => <Article: Article object (1)>

    # article_pk는 존재하지 않는 필드 이기 때문에 사용 불가
    comment.article_id
    => 1
    ```

- 댓글 생성 연습 - 5
  - 댓글 생성
  - ```python
    # 1번 댓글이 작성된 게시물의 pk 조회
    comment.article.pk
    => 1

    # 1번 댓글이 작성된 게시물의 content 조회
    comment.article.content
    => 'first comment'
    ```

- 댓글 생성 연습 - 6
  - 두번째 댓글 작성
  - ```python
    comment = Comment(content='second comment', article=article)
    comment.save()

    comment.pk
    => 2

    comment
    => <Comment: Comment object (2)>

    comment.article.pk
    => 1
    ```

- 댓글 생성 연습 - 7
  - 작성된 댓글을 db.splite에서 확인


### 2. 관계 모델 참조
- `역참조`
  - 나를 참조하는 테이블(나를 외래 키로 지정한)을 참조 하는 것
  - N:1 관계에서는 1이 N을 참조하는 상황
  - 하지만 Article에는 Comment를 참조할 어떠한 필드도 없음

- `article.comment_set.all()`
  - article : 모델 인스턴스
  - comment_set : related manager
  - all() : QuerySey API

- `related manager`
  - N:1 혹은 M:N 관계에서 역참조 시에 사용하는 manager
  - object라는 매니저를 통해 queryset api를 사용했던 것처럼 related manager를 통해 queryset api를 사용할 수 있게 됨

- related manager가 필요한 이유
  - article.comment 형식으로는 댓글 객체를 참조 할 수 없음
  - 실제 Article 클래스에는 Comment와의 어떠한 관계도 작성되어 있지 않기 때문
  - 대신 Django가 역참조 할 수 있는 'comment_set' manager를 자동으로 생성해 article.comment_set 형태로 댓글 객체를 참조할 수 있음
  - N:1 관계에서 생성되는 Related manager의 이름은 참조하는 "모델명_set" 이름 규칙으로 만들어짐

- Related manager 연습 - 1
  - shell_plus 실행 및 1번 게시글 조회
  - ```python
    article = Article.objects.get(pk=1)
    ```

- Related manager 연습 - 2
  - 1번 게시글에 작성된 모든 댓글 조회하기 (역참조)
  - ```python
    article.comment_set.all()
    => <QuerySet [<Comment: Comment object (1)>, <Comment: Comment object (2)>]>
    ```

- Related manager 연습 - 3
  - 1번 게시글에 작성된 모든 댓글 출력하기
  - ```python
    comments = article.comment_set.all()

    for comment in comments:
      print(comment.content)
    ```


### 3. 댓글 기능 구현
- Comment CREATE - 1
  - 사용자로부터 댓글 데이터를 입력 받기 위한 CommentForm 작성
  - ```python
    # articles/forms.py
    from .models import Article, Comment

    class CommentForm(forms.ModelForm):
      class Meta:
        model = Comment
        fields = '__all__'
    ```

- Comment CREATE - 1
  - detail 페이지에서 CommentForm 출력 (View 함수)
  - ```python
    # articles/views.py
    from .models import Article, Comment

    class CommentForm(forms.ModelForm):
      class Meta:
        model = Comment
        fields = '__all__'
    ```

- Comment CREATE - 2
  - 사용자로부터 댓글 데이터를 입력 받기 위한 CommentForm 작성
  - ```python
    # articles/forms.py
    from .forms import ArticleForm, CommentForm

    def detail(request, pk):
      article = Article.objects.get(pk=pk)
      comment_form = CommentForm()
      context = {
          'article': article,
          'comment_form': comment_form,
      }
      return render(request, 'articles/detail.html', context)
    ```

- Comment CREATE - 3
  - detail 페이지에서 CommentForm출력 (템플릿)
  - ```html
    <!-- articles/detail.html -->
    <form action="#" method="POST">
      {% csrf_token %}
      {{ comment_form }}
      <input type="submit">
    </form>
    ```

- Comment CREATE - 4
  - CommentForm 출력확인
  - 외래키 필드는 사용자의 입력을 받는 것이 아니라 view 함수 내에서 받아 처리되어 저장되어야 함

- Comment CREATE - 5
  - ```python
    # articles/forms.py
    from .models import Article, Comment

    class CommentForm(forms.ModelForm):
      class Meta:
          model = Comment
          fields = ('content',)
    ```

- Comment CREATE - 6
  - 출력에서 제외된 외래 키 데이터는 어디서 받아와야 할까?
  - detail 페이지의 url을 살펴보면 path('<int:pk>/', views.detail, name='detail') url에 해당 게시글의 pk 값이 사용 되고 있음
  - 댓글의 외래 키 데이터에 필요한 정보가 바로 게시글의 pk 값

- Comment CREATE - 7
  - ```python
    # articles.urls.py
    urlpatterns = [
      ...,
      path('<int:article_pk>/comments/', views.comment_create, name='comment_create')
    ]
    ```
  - ```python
    # articles.vies.py
    
    def comment_create(request, article_pk):
      # 몇 번 게시글인지 조회
      article = Article.objects.get(pk=article_pk)
      # 댓글 데이터를 받아서
      comment_form = CommentForm(request.POST)
      # 유효성 검증
      if comment_form.is_valid():
          comment = comment_form.save(commit=False)
          comment.article = article
          comment.save()
          return redirect('articles:detail', article_pk)
      context = {
          'article': article,
          'comment_form': comment_form,
      }
      return render(request, 'articles/detail.html', context)
    ```
  - ```html
    <!-- articles/detail.html -->
    <form action="{% url 'articles:comment_create' article.pk %}" method="POST">
      {% csrf_token %}
      {{ comment_form }}
      <input type="submit">
    </form>
    ```

- `save(commit=False)`
  - "Create, but don't save the new instance"
  - DB에 저장하지 않고 인스턴스만 반환

- Comment CREATE - 8
  - 댓글 작성후 db.split3에서 테이블 확인

- Comment READ - 1
  - 전체 댓글 출력(view 함수)
  - ```python
    # articles/views.py
    def detail(request, pk):
      article = Article.objects.get(pk=pk)
      comment_form = CommentForm()
      # 해당 게시글에 작성된 모든 댓글을 조회(역참조)
      comments = article.comment_set.all()
      context = {
          'article': article,
          'comment_form': comment_form,
          'comments': comments,
      }
      return render(request, 'articles/detail.html', context)
    ```

- Comment READ - 2
  - 전체 댓글 출력(템플릿)
  - ```html
    <!-- articles/detail.html -->
    <h4>댓글 목록</h4>
    <ul>
      {% for comment in comments %}
        <li>{{ comment.content }}</li>
      {% endfor %}
    </ul>
    ```

- Comment READ - 3
  - 전체 댓글 출력 확인

- Comment DELETE - 1
  - 댓글 삭제 url 작성
  - ```python
    # articles.urls.py
    urlpatterns = [
      ...,
      path('<int:article_pk>/comments/<int:comment_pk>/delete/', views.comment_delete, name='comment_delete'),
    ]
    ```

- Comment DELETE - 2
  - 댓글 삭제 view 함수 작성
  - ```python
    # articles/views.py
    def comment_delete(request, article_pk, comment_pk):
      # 삭제할 댓글을 조회
      comment = Comment.objects.get(pk=comment_pk)
      # 댓글 삭제
      comment.delete()
      return redirect('articles:detail', article_pk)
    ```

- Comment DELETE - 3
  - 댓글 삭제 버튼 작성
  - ```html
    <!-- articles/detail.html -->
    <form action="{% url 'articles:comment_delete' article.pk comment.pk %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="삭제">
    </form>
    ```


## 참고
- 댓글 개수 출력하기
  - DTL filter - length 사용
    - ```python
      {{ comments|length }}

      {{ article.comments_set.all|length }}
      ```
  - Queryset API - count() 사용
    - ```python
      {{ article.comment_set.count }}
      ```

- 댓글이 없는 경우 대체 컨텐츠 출력
  - DTL tag - for empty 사용
    - ```html
      {% empty %}
        <p>댓글이 없어요..</p>
      ```

- 댓글 수정을 구현하지 않는 이유
  - 일반적으로 댓글 수정은 수정 페이지로 이동 없이 현재 페이지가 유지된 상태로 Form 부분만 변경되어 수정 할 수 있도록 함
  - 페이지의 일부 내용만 업데이트 하는 것은 JavaScript의 영역이기 때문에 JavaScript를 사용해 도전해 볼 수 있음

- admin site 등록
  - 새로 작성한 Comment 모델을 admin site에 등록하기
  - ```python
    from .models import Article, Comment

    admin.site.register(Article)
    admin.site.register(Comment)
    ```