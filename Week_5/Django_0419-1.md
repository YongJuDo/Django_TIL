# Django Improve query

<br>

## Improve query
- `Improve query`
  - 같은 결과에 대한 쿼리 개수를 줄여 조회하기

- 사전 준비
  - migrate 및 fixtures 데이터 load
  - ```python
    python manage.py migrate
    python manage.py loaddata users.json
    articles.json comments.json
    ```
  - 데이터 현황
    - 게시글 10개, 댓글 100개, 유저 5개
  - 모델관계
    - N:1 - Article:User / Comment:Article / Comment:Article
    - N:M - Article:User

- `annotate`
  - SQL의 GROUP BY 절을 활용

- annotate - 1
  - 11 queries including 10 similar
  - 원인 : 각 게시글 별 댓글 개수를 반복 평가
  - ```html
    <!-- index_1.html -->
    <p>댓글개수 : {{ article.comment_set.count }}</p>
    ```

- annotate - 2
  - "11 queries including 10 similar -> 1 query
  - 해결 : annotate를 사용해 첫 조회 시 댓글 개수까지 한번에 조회
  - ```python
    # views.py
    def index_1(request):
      articles = Article.objects.order_by('-pk')
      # articles = Article.objects.annotate(Count('comment')).order_by('-pk')
      context = {
          'articles': articles,
      }
      return render(request, 'articles/index_1.html', context).
    ```
  - ```html
    <!-- index_1.html -->
    <p>댓글개수 : {{ article.comment__count }}</p>
    ```

- `select_related`
  - 1:1 또는 N:1 참조 관계에서 사용
  - SQL의 INNER JOIN 절을 활용

- select_related - 1
  - 11 queries including 10 similar and 8 duplicates
  - 원인 : 각 게시글 출력 후 게시글을 작성한 유저의 이름까지 반복 평가
  - ```html
    <!-- index_1.html -->
    {% for article in articles %}
      <h3>작성자 : {{ article.username }}</h3>
      <p>제목 : {{ article.title}}</p>
      <hr>
    {% endfor %}
    ```
- select_related - 2
  - 11 queries including 10 similar and 8 duplicates -> 1 query
  - 해결 : select_related를 사용해 article을 조회하면서 user까지 한번에 조회
  - ```python
    # views.py
    def index_2(request):
      # articles = Article.objects.order_by('-pk')
      articles = Article.objects.select_related('user').order_by('-pk')
      context = {
          'articles': articles,
      }
      return render(request, 'articles/index_2.html', context)
    ```

- `prefetch_related`
  - M:N 또는 N:1 역참조 관계에서 사용
  - SQL이 아닌 Python을 사용한 JOIN이 진행됨

- prefetch_related - 1
  - 11 queries including 10 similar
  - 원인 : 각 게시글 출력 후 게시글의 댓글 목록을 개별적으로 모두 조회
  - ```html
    <!-- index_1.html -->
    {% for article in articles %}
      <p>제목 : {{ article.title}}</p>
      <p>댓글목록</p>
      {% for comment in articles.comment_set.all %}
        <p>{{ comment.content }}</p>
      {% endfor %}
      <hr>
    {% endfor %}
    ```

- prefetch_related - 2
  - 11 queries including 10 similar
  - 해결 : prefetch_related 사용해 article을 조회하면서 comment까지 한번에 조회
  - ```python
    # views.py
    def index_3(request):
      articles = Article.objects.order_by('-pk')
      # articles = Article.objects.prefetch_related('comment_set').order_by('-pk')
      context = {
          'articles': articles,
      }
      return render(request, 'articles/index_3.html', context)
    ```

- select_related & prefetch_related - 1
  - 111 queries including 110 similar and duplicates
  - 원인 : 각 게시글 출력 후 게시글의 댓글 목록을 개별적으로 모두 조회
  - ```html
    <!-- index_1.html -->
    {% for article in articles %}
      <p>제목 : {{ article.title }}</p>
      <p>댓글 목록</p>
      {% for comment in article.comment_set.all %}
        <p>{{ comment.user.username }} : {{ comment.content }}</p>
      {% endfor %}
      <hr>
    {% endfor %}
    ```

- prefetch_related - 2
  - 111 queries including 110 similar and duplicates
  - 해결 : 게시글 출력 + 각 게시글의 댓글 목록 + 댓글의 작성자를 한번에 조회
  - ```python
    # views.py
    def index_4(request):
      articles = Article.objects.order_by('-pk')
      # articles = Article.objects.prefetch_related('comment_set').order_by('-pk')
      # articles = Article.objects.prefetch_related(
      #     Prefetch('comment_set', queryset=Comment.objects.select_related('user'))
      # ).order_by('-pk')

      context = {
          'articles': articles,
      }
      return render(request, 'articles/index_4.html', context)
    ```