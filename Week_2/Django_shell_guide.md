# Django shell_plus 가이드

<br>

## 사전 준비 사항
1. [터미널 입력] - 패키지 설치
  - ```python
    pip install ipython
    pip install django_extensions
    ```

2. [파일 조작] - django_extensions App 등록
  - ```python
    # settings.py
    INSTALLED_APPS = [
      'django_extensions', # 추가
    ]
    ```

3. [터미널 입력] - shell 진입
  - ```python
    python manage.py shell_plus
    ```

