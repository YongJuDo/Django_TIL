# Django Many to many relationships - 2

<br>

## 개요
- profile 구현
  - 자연스러운 follow 흐름을 위한 프로필 페이지 작성
  - ```python
    # accounts/urls.py
    urlpatterns = [
      ...
      path('profile/<username>/', views.profile, name='profile'),
    ]
    ```
  - ```python
    # accounts/view.py
    from django.contrib.auth import get_user_model

    def profile(request, username):
      User = get_user_model()
      person = User.objects.get(username=username)
      context = {
        'person': person,
      }
      return render(request, 'accounts/profile.html', context)
    ```
  - ```html
    <!-- accounts/profile.html -->
    <h1>{{ person.username }}의 프로필 페이지</h1>

    <hr>

    <h3>{{ person.username  }}가 작성한 모든 게시글</h3>
    {% for article in person.article_set.all %}
      <div>{{ article.title }}</div>
    {% endfor %}

    <hr>

    <h3>{{ person.username }}가 작성한 모든 댓글</h3>
    {% for comment in person.comment_set.all %}
      <div>{{ article.content }}</div>
    {% endfor %}

    <hr>

    <h3>{{ person.username }}가 좋아요를 누른 모든 게시글</h3>
    {% for article in person.like_articles.all %}
      <div>{{ article.title }}</div>
    {% endfor %}
    ```
  - ```html
    <!-- articles/index.html -->
    <a href="{% url 'accounts:profile' user.username %}">내 프로필</a>
    <p>작성자 : 
      <a href="{% url 'accounts:profile' article.user.username %}">{{ article.user }}</a>
    </p>
    ```


## User & User
- `User(M) - User(N)`
  - 유저는 0명 이상의 다른 유저와 관련된다.
  - 유저는 다른 유저로부터 0개 이상의 팔로우를 받을 수 있고, 유저는 0명 이상의 다른 유저들에게 팔로잉 할 수 있다.

- Follow 구현
  - ManyToManyField 작성 및 Migration 진행
  - ```python
    # accounts/models.py
    class User(AbstractUser):
      followings = models.ManyToManyField('self', symmetrical=False, related_name='followers')
    ```
  - ```python
    # accounts/urls.py
    urlpatterns = [
      ...
      path('<int:user_pk>/follow/', views.follow, name='follow'),
    ]
    ```
  - ```python
    # accounts/views.py
    def follow(request, user_pk):
      User = get_user_model()
      you = User.objects.get(pk=user_pk)
      me = request.user

      if you != me:
          if me in you.followers.all() :
              you.followers.remove(me)
              # me.followings.remove(you)
          else:
              you.followers.add(me)
              # me.followings.add(you)
      return redirect('accounts:profile', you.get_username)
    ```
  - ```html
    <!-- accounts/profile.html -->
    <div>
      팔로잉 : {{ person.followings.all|length }} / 팔로워 : {{ person.followers.all|length }}
    </div>

    {% if request.user != person %}
    <div>
      <form action="{% url 'accounts:follow' person.pk %}" method="POST">
        {% csrf_token %}
        {% if request.user in person.followers.all  %}
          <input type="submit" value="언팔로우">
        {% else %}
          <input type="submit" value="팔로우">
        {% endif %}
      </form>
    </div>
    {% endif %}
    ```
