# Django Authentication System - 2

<br>

## 개요
- `User 객체와 CUD`
  - 회원 가입, 회원 탈퇴, 회원정보 수정, 비밀번호 변경

## 회원가입
- `회원가입`
  - User 객체를 Create 하는 것

- UserCreationForm()
  - 회원 가입을 위한 built-in ModelForm

- 회원 가입 페이지 및 로직 작성
  - ```python
    # accounts/urls.py

    app_name = 'accounts'
    urlpatterns = [
      ...,
      path('signup/' views.signup, name='signup')
    ]
    ```
  - ```python
    # accounts/views.py

    from django.contrib.auth.forms import UserCreationForm

    def signup(request):
      if request.method == 'POST':
        if form.is_vaild():
          form.save()
          return redirect('aticles:index')
      else:
        form = UserCreationForm()
      context ={
        'form': form,
      }
      return render(request. 'accounts/signup.html', context)
    ```
  - ```html
    <!-- accounts/signup.html -->

    <h1>회원가입</h1>
    <form action="{% url 'accounts:signup' %}" method="POST">
      {{% csrf_token %}}
      {{% form.as_p %}}
      <input type="sumbit">
    </form>
    ```

- 회원 가입 진행 후 에러 페이지 확인
  - 회원가입에 사용하는 UserCreationForm이 우리가 대체한 커스텀 유저 모델이 아닌 기존 유저 모델로 인해 작성된 클래스이기 때문
  - ```python
    class Meta:
      model = User
      fields = ("username",)
      fields_classes = {"username": UsernameField}
    ```

- 커스텀 유저 모델을 사용하려면 다시 작성해야하는 forms
  - UserCreationForm, UserChangeForm 두 form 모두 class Meta: model = User가 등록된 form 이기 때문

- 커스텀 Form 작성
  - ```pyhton
    from django.contrib.auth import get_user_model
    from django.contrib.auth.forms import UserCreationForm, UserChangeForm

    class CustomUserCreationForm(UserCreationForm):
      class Meta(UserCreationForm.Meta):
        model = get_user_model()
    
    class CustomUserChangeForm(UserChangeForm):
      class Meta(UserChangeForm.Meta):
        model = get_user_model()
    ```

- `get_user_model()`
  - 현재 프로젝트에서 활성화된 사용자 모델을 반환하는 함수

- User 모델을 직접 참조하지 않는 이유
  - User 모델을 get_user_model()을 사용해 참조하면 커스텀 User 모델을 자동으로 반환해주기 때문
  - Django는 User 클래스를 직접 참조하는 대신 get_user_model()을 사용해 참조해야 한다고 강조

- 회원 가입 로직 수정
  - ```python
    def signup(request):
    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('accounts:index')
    else:
        form = CustomUserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
    ```

## 회원 탈퇴
- `회원 탈퇴`
  - User 객체를 Delete 하는 것

- 회원 탈퇴 로직 작성
  - ```python
    # accounts/urls.py

    app_name = 'accounts'
    urlpatterns = [
      ...,
      path('delete/' views.delete, name='delete'),
    ]
    ```
  - ```python
    # accounts/views.py

    def delete(request):
      request.user.delete()
      return redirect('articles:index')
    ```
  - ```html
    <!-- accounts/index.html -->

    <form action="{% url 'accounts:delete' %}" method="POST">
      {{% csrf_token %}}
      <input type="sumbit" value="회원탈퇴">
    </form>
    ```
- 탈퇴 하면서 유저의 세션 정보도 함께 지우고 싶을 경우
  - 탈퇴(1) 후 로그아웃(2)의 순서가 바뀌면 안됨
  - 먼저 로그아웃을 해버리면 해당 요청 객체 정보가 없어지기 때문에 탈퇴에 필요한 유저 정보 또한 없어지기 때문
  - ```python
    def delete(request):
      request.user.delete()
      auth_logout(request)
    ```
  

## 회원정보 수정
- `회원정보 수정`
  - User 객체를 Update 하는 것

- `UserChangeForm()`
  - 회원 가입을 위한 built-in ModelForm

- 회원정보 수정 페이지 및 로직 작성
  - ```python
    # accounts/urls.py

    app_name = 'accounts'
    urlpatterns = [
      ...,
      path('update/', views.update, name='update'),
    ]
    ```
  - ```python
    # accounts/views.py

    def update(request):
      if request.method == 'POST':
          form = CustomUserChangeForm(request.POST, instance=request.uesr)
          if form.is_valid():
              form.save()
              return redirect('accounts:index')
      else:
          form = CustomUserChangeForm(instance=request.uesr)
      context = {
          'form': form,
      }
      return render(request, 'accounts/update.html', context)
    ```
  - ```html
    <!-- accounts/index.html -->

    <h1>회원정보 수정</h1>
    <form action="{% url 'accounts:update' %}" method="POST">
      {% csrf_token %}
      {{ form.as_p }}
      <input type="sumbit"></input>
    </form>
    ```
- UserChangeForm 사용 시 문제점
  - 일반 사용자가 접근해서는 안될 정보들까지 모두 수정이 가능해짐
  - admin 인터페이스에서 사용되는 ModelForm이기 때문
  - CustomUserChangeForm에서 접근 가능한 필드를 조정해야 함

- CustomUserChangeForm fields 재정의
  - User Model의 필드는 AbstractUser 클래스를 참고
  - ```python
    class CustomUserChangeForm(UserChangeForm):
      class Meta(UserChangeForm.Meta):
          model = get_user_model()
          fields = ('email', 'first_name', 'last_name')
    ```


## 비밀번호 변경
- 비밀번호 변경 페이지
  - django는 비밀번호 변경 페이지를 회원정보 수정 form에서 별도 주소로 안내
    - /accounts/password/

- `PasswordChangeForm()`
  - 비밀번호 변경을 위한 built-in-Form

- 비밀번호 변경 페이지 및 로직 작성
  - ```python
    # accounts/urls.py

    app_name = 'accounts'
    urlpatterns = [
      ...,
      path('password/', views.password, name='password'),
    ]
    ```
  - ```python
    # accounts/views.py

    def change_password(request):
      if request.method == 'POST':
          form = PasswordChangeForm(request.user, request.POST)
          if form.is_valid():
              form.save()
              return redirect('accounts:index')
      else:
          form = PasswordChangeForm(request.user)
      context = {
          'form': form,
      }
      return render(request, 'accounts/change_password.html', context)
    ```
  - ```html
    <!-- accounts/index.html -->

    <h1>비밀번호 변경</h1>
    <form action="{% url 'accounts:change_password' %}" method="POST">
      {% csrf_token %}
      {{ form.as_p }}
      <input type="sumbit"></input>
    </form>
    ```

- 암호 변경시 세션 무효화
  - 비밀번호가 변경되면 기존 세션과의 회원 인증 정보가 일치하지 않게 되어 버려 로그인 상태가 유지되지 못함
  - 비밀번호는 잘 변경되었으나 비밀번호가 변경 되면서 기존 세션과의 회원 인증 정보가 일치하지 않기 때문

- `update_session_auth_hash(request, user)`
  - 암호 변경 시 세션 무효화 방지
  - 암호가 변경되어도 로그아웃 되지 않도록 새로운 password의 session data로 기존 session을 업데이트

- update_session_auth_hash 적용
  - ```python
    # accounts/views.py
    from django.contrib.auth import update_session_auth_hash
    if form.is_valid():
      user = form.save()
      update_session_auth_hash(request, user)
      return redirect('accounts:index')
    ```

## 로그인 사용자에 대한 접근 제한
- 로그인 사용자에 대해 접근을 제한하는 2가지 방법
  1) is_authenticated 속성
  2) login_required 데코레이터

- `is_authenticated`
  - 사용자가 인증 되었는지 여부를 알 수 있는 User model의 속성(attributes)
  - 모든 User 인스턴스에 대해 항상 True인 읽기 전용 속성이며, AnonymousUser에 대해서는 항상 False
  - 권한과는 관련이 없으며, 사용자가 활성화 상태이거나 유효한 세션을 가지고 있는지도 확인하지 않음
  - ```html
    <!-- articles/index.html -->
    {% if user.is_authenticated %}
    {% else %}
    {% endif %}
    ```
  - ```python
    # accounts/views.py
    def login(request):
      if request.user.is_authenticated:
        return redirect('articles.index')
    ```

- `login_required`
  - 인증된 사용자에 대해서만 view 함수를 실행시키는 데코레이터
  - 로그인 하지 않은 사용자의 경우 /accounts/login/ 주소로 redirect 시킴
  - ```python
    # articles/views.py
    from django.contrib.auth.decorators import login_required

    @login_required
    def create(request):
      pass
    
    @login_required
    def delete(request, article_pk):
      pass
    
    @login_required
    def update(request, article_pk):
      pass
    ```
    - ```python
    # accounts/views.py
    from django.contrib.auth.decorators import login_required

    @login_required
    def create(request):
      pass
    
    @login_required
    def delete(request):
      pass
    
    @login_required
    def update(request):
      pass
    
    @login_required
    def change_password(request):
      pass
    ```

## 참고
- 데코레이터(Decorator)
  - 기존에 작성된 함수에 기능을 추가하고 싶을 때, 해당 함수를 수정하지 않고 기능만을 추가 해주는 함수
  - ```python
    def hello(func):
      def wrapper():
        print('HIHI')
        func()
        print('HIHI')

      return wrapper
    
    @hello
    def bye():
      print('byebye')

    bye()
    ```
  - ```python
    # 출력
    HIHI
    byebye
    HIHI
    ```