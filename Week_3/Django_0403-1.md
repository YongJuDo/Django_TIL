# Django Form

<br>

## 개요
- `HTML form`
  - 사용자로부터 form요소를 통해 데이터를 받고 있으나 비정상적 혹은 악의적인 요청을 확인하지 않고 모두 수용중
  - 우리가 원하는 데이터 형식이 맞는지에 대한 '유효성 검증' 필요

- `유효성 검사`
  - 수집한 데이터가 정확하고 유효한지 확인하는 과정
  - 유효성 검증에는 입력 값, 형식, 범위, 보안 등 부가적인 많은 것들을 고려해야 함
  - 이런 과정과 가능을 제공해주는 도구가 필요


## Django Form
- `Django Form`
  - 사용자 입력 데이터를 수집하고, 처리 및 유효성 검증을 수행하기 위한 도구
  - 유효성 검사를 단순화하고 자동화 할 수 있는 기능을 제공

- Form class 선언
  - ```python
    from django import forms

    class ArticlesForm(forms.Form):
      title = forms.CharField(max_length=10)
      content = forms.CharField()
    ```

- Form class를 적용한 new 로직
  - ```python
    # articles/views.py
    from .forms import ArticlesForm

    def new(request):
      form = ArticlesForm()
      context = {
          'form': form,
      }
      return render(request, 'articles/new.html', context)
    ```
  - ```html
    <!-- articles/new.html -->
    
    <h1>New</h1>
    <form action="{% url 'articles:create' %}" method="POST">
      {% csrf_token %}
      {{ form }}
      <input type="submit">
    </form>
    ```

- Form rendering options
  - title과 content를 p태그로 분리
  - ```html
    <!-- articles/new.html -->
    
    <h1>New</h1>
    <form action="{% url 'articles:create' %}" method="POST">
      {% csrf_token %}
      {{ form.as_p }}
      <input type="submit">
    </form>
    ```


## Widgets
- `Widgets`
  - HTML 'input' element의 표현을 담당

- Widget은 단순히 input요소의 속성 및 출력되는 부분을 변경하는 것
  - ```python
    from django import forms

    class ArticlesForm(forms.Form):
      title = forms.CharField(max_length=10)
      content = forms.CharField(widigt=forms.Textarea)
    ```
  - https://docs.djangoproject.com/ko/3.2/ref/forms/widgets/#built-in-widgets


## Django ModelForm
- `Form`
  - 사용자 입력 데이터를 DB에 저장하지 않을때
  - ex. 로그인
- `ModelForm`
  - 사용자 입력 데이터를 DB에 저장해야 할 때
  - ex. 회원가입

- ModelForm class 선언
  - ```python
    # articles/forms.py
    from django import forms
    from .models import Article

    class ArticlesForm(forms.ModelForm):
      class Meta:
        model = Article
        fields = '__all__'
    ```

- `Meta class`
  - ModelForm의 정보를 작성하는 곳

- Fields 및 exclude 속성
  - exclude 속성을 사용하여 모델에서 포함하지 않을 필드를 지정할 수도 있음
  - ```python
    # articles/forms.py

    class ArticlesForm(forms.ModelForm):
      class Meta:
        model = Article
        exclude = '__all__'
    ```

- ModelForm을 적용한 create 로직
  - ```python
    # articles/views.py
    from .forms import ArticleForm

    def create(request):
      form = ArticlesForm(request.POST)
      if form.is_valid():
          article = article.save()
          return redirect('articles:detail', article.pk)
      context = {
          'form': form,
      }
      return render(request, 'articles/new.html', context)
    ```
  - 제목 input에 공백 값을 입력 후 에러메시지 확인 (유효성 검사 결과)

- `is_valid()`
  - 여러 유효성 검사를 실행하고, 데이터가 유효한지 여부를 boolean으로 반환

- ModelForm을 적용한 edit 로직
  - ```python
    # articles/views.py
    def edit(request, article_pk):
      article = Article.objects.get(pk=article_pk)
      form = ArticleForm(instance=article)
      context = {
          'article': article,
          'form': form,
      }

      return render(request, 'articles/edit.html', context)
    ```
  - ```html
    <h1>Edit</h1>
    <form action="{% url 'articles:update' article.pk %}" method="POST">
      {% csrf_token %}
      {{ form.as_p }}
      <input type="submit">
    </form> 
    ```

- ModelForm을 적용한 update 로직
  - ```python
    # articles/views.py
    def update(request, article_pk):
      article = Article.objects.get(pk=article_pk)
      form = ArticleForm(request.POST, instance=article)
      if form.is_valid():
        article.save()
        return redirect('articles:detail', article.pk)
      context = {
        'form': form,
      }
      return render(request, 'articles/edit.html', context)
    ```

- `save()`
  - 데이터베이스 객체를 만들고 저장
  - 키워드 인자 instance 여부를 통해 생성할지, 수정할 지를 결정
  - ```python
    # CREATE
    form = ArticleForm(request.POST)
    form.save()

    # UPDATE
    form = ArticleForm(request.POST, instance=article)
    form.save()
    ```

## 참고
- ModelForm 키워드 인자 data와 instance 살펴보기
  - ```python
    class BaseModelForm(BaseForm, AltersData):
      def __init__(
          self, data=None, files=None, auto_id="id_%s", prefix=None, initial=None, error_class=ErrorList, label_suffix=None, empty_permitted=False, instance=None, use_required_attribute=None, renderer=None,
      ):
    ```

- Meta class
  - 클래스 안에 클래스..? 파이썬에서는 Inener class 혹은 Nested class 라고 하는데
  - 파이썬의 문법적 개념으로 접근하지 말것
  - 단순히 모델 정보를 Meta라는 이름의 내부 클래스로 작성되도록 ModelForm의 설계가 이렇게 되어있을 뿐 우리는 ModelForm의 역할과 사용법을 숙지하는데 집중할 것