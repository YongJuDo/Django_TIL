# Django 개발 환경 설정 가이드

<br>

1. 가상환경(venv) 생성
    - `python -m venv venv`

2. 가상환경 활성화
    - `source venv/Scripts/activate`
    - Ctrl(command) + Shift + p => Json 검색 => interpreter 검색 => venv 선택 => 터미널 창

3. django 설치
    - 설치 버전 : 3.2.18 (현 LTS)
    - `pip install django==3.2.18`
    - 버전을 입력하지 않을경우 최신버전 설치

4. 의존성 파일 생성 (requirements.txt)
    - `pip freeze > requirements.txt`

5. django 프로젝트 생성
    - `django-admin startproject firstpjt .`
    - (firstpjt라는 이름의 프로젝트 생성)

6. django 서버 실행
    - `python manage.py runserver`
    - manage.py와 동일한 경로에서 명령어 진행
    - http://127.0.0.1:8000/ 접속후 확인

### git 초기화 시 필수 사항
- gitigonre 작성
    - https://www.toptal.com/developers/gitignore/ 활용
    - django, windows, macOS, visualStudioCode, Python
    - macOS의 경우 협업을 위해 필요