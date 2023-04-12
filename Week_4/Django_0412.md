# Django Many to one relationships - 2

<br>

## 개요
- `Article(N) - User(1)`
  - 0개 이상의 게시글은 1개의 회원에 의해 작성 될 수있음

- `Comment(N) - User(1)`
  - 0개 이상의 댓글은 1개의 회원에 의해 작성 될 수 있음


## Article & User
### 1. 모델 관계 설정
- User 외래 키 정의
  - ```python
    # articles/models.py
    from django.conf import settings

    class Article(models.Model):
        user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
        ...
    ```
- User 모델을 참조하는 2가지 방법
  - `get_user_model()`
    - 반환 값: 'User Object' (객체)
    - models.py가 아닌 다른 모든 곳에서 참조할 때 사용
  - `settings.AUTH_USER_MODEL`
    - 반환 값: 'accounts.User' (문자열)
    - models.py가 모델 필드에서 참조할 때 사용

- Migration 진행 - 1
  - 기본적으로 모든 컬럼은 NOT NULL 제약조건이 있기 때문에 데이터가 없이는 새로 추가되는 외래 키 필드 user_id가 생성되지 않음
  - 그래서 기본값을 어떻게 작성할 것인지 선택해야 함
  - 1을 입력하고 Enter 진행

- Migration 진행 - 2
  - article의 user_id에 어떤 데이터를 넣을 것인지 직접 입력해야 함
  - 마찬가지로 1 입력하고 Enter 진행
  - 그러면 기존에 작성된 게시글이 있다면 모두 1번 회원이 작성한 것으로 처리됨

- Migration 진행 - 3
  - migrations 파일 생성 후 migrate 진행

- Migration 진행 - 4
  - article 테이블 user_id 필드 확인


### 2. CRUD 구현
- Article CREATE - 1
  - ArticleForm 출력 확인

- Article CREATE - 2
  - ArticleForm 출력 필드 수정
  - ```python
    # articles/forms.py
    class ArticleForm(forms.ModelForm):
      class Meta:
        model = Article
        fields = ('title', 'content',)
    ```

- Article CREATE - 3
  - 게시글 작성시 user_id 필드 데이터가 누락되어 에러 발생

- Article CREATE - 4
  - 게시글 작성시 작성자 정보가 함께 저장될 수 있도록 save의 commit 옵션 활용
  - ```python
    def create(request):
      if request.method == 'POST':
          form = ArticleForm(request.POST)
          if form.is_valid():
              article = form.save(commit=False)
              article.user = request.user
              article.save()
              return redirect('articles:detail', article.pk)
      else:
      ...
    ```

- Article CREATE - 5
  - 게시글 작성 후 테이블 확인

- Article READ
  - Index 템플릿과 detail 템플릿에서 각 게시글의 작성자 출력 및 확인

- Article UPDATE - 1 
  - 수정을 요청하려는 사람과 게시글을 작성한 사람을 비교하여 본인의 게시글만 수정할 수 있도록 함
  - ```python
    # articles/vies.py
    def update(request, article_pk):
      article = Article.objects.get(pk=article_pk)
      if request.method ==  article.user:
          form = ArticleForm(request.POST, instance=article)
          if form.is_valid():
              form.save()
              return redirect('articles:detail', article.pk)
          else:
              form = ArticleForm(instance=article)
      else:
          return redirect('articles:index')
      ...
    ```

- Article UPDATE - 2
  - 해당 게시글의 작성자가 아니라면, 수정/삭제 버튼을 출력하지 않도록 함
  - ```html
    <!-- articles/detail.html -->
    {% if request.user == article.user %}
      <form action="{% url 'articles:delete' article.pk  %}" method="POST">
        {% csrf_token %}
        <input type="submit" value="삭제">
      </form>
      <a href="{% url 'articles:update' article.pk %}">[UPDATE]</a>
    {% endif %}
    ```

- Article DELETE
  - 삭제를 요청하려는 사람과 게시글을 작성한 사람을 비교하여 본인의 게시글만 삭제할 수 있도록 함
  - ```pyhton
    # articles/vies.py
    def delete(request, artilce_pk):
      article = Article.objects.get(pk=artilce_pk)
      if request.user == article.user:
          article.delete()
      return redirect('articles:index')
    ```

## Comment & User
### 1. 모델 관계 설정
- User 외래 키 정의
  - ```python
    # articles/models.py

    class Comment(models.Model):
        user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
        ...
    ```

- Migration 진행
  - 이전 Article와 User 모델 관계 설정 때와 마찬가지로 기존에 존재하던 테이블에 새로운 컬럼이 추가되어야 하는 상황이기 때문에 migrations 파일이 곧바로 만들어지지 않고 일련의 과정이 필요
  - comment 테이블 user_id 필드 확인


### 2. CRD 구현
- Comment CREATE - 1
  - 댓글 작성시 user_id 필드 데이터가 누락되어 에러 발생
  
- Comment CREATE - 2
  - 댓글 작성시 작성자 정보가 함께 저장될 수 있도록 save의 commit 옵션 활용
  - ```python
    # articles/views.py
    def comment_create(request, article_pk):
      article = Article.objects.get(pk=article_pk)
      comment_form = CommentForm(request.POST)
      if comment_form.is_valid():
          comment = comment_form.save(commit=False)
          comment.article = article
          comment.user = request.user
          comment_form.save()
          return redirect('articles:detail', article_pk)
      ...
    ```
- Comment CREATE - 3
  - 댓글 작성 후 테이블 확인

- Comment READ
  - detail 템플릿에서 각 댓글의 작성자 출력 및 확인

- Comment DELETE - 1
  - 삭제를 요청하려는 사람과 댓글을 작성한 사람을 비교하여 본인의 댓글만 삭제 할수 있도록 함
  - ```python
    # articles/views.py
    def comment_delete(request, article_pk, comment_pk):
    comment = Comment.objects.get(pk=comment_pk)
      if request.user == comment.user:
          comment.delete()
      return redirect('articles:detail', article_pk)
    ```

- Comment DELETE - 2
  - 해당 댓글의 작성자가 아니라면, 댓글 삭제 버튼을 출력하지 않도록 함
  - ```html
    <!-- articles/detail.html -->
    <ul>
    {% for comment in comments %}
      <li>
        {{ comment.user }} - {{ comment.content }}
        {% if request.user == comment.user %}
          <form action="{% url 'articles:comment_delete' article.pk comment.pk %}" method="POST">
            {% csrf_token %}  
            <input type="submit" value="삭제">
          </form>
      </li>
      {% endif %}
    {% endfor %}
    </ul>
    ```


## 참고
- 인증된 사용자인 경우만 댓글 작성 및 삭제하기
  - ```python
    # articles/views.py
    @login_required
    def comments_create(request, pk):
      pass
    
    @login_required
    def comments_delete(request, article_pk, comment_pk):
      pass
    ```