# Django Fixtures

<br>

## 개요
- `fixtures`
  - Django가 데이터베이스로 가져오는 방법을 알고 있는 데이터 모음
  - Django가 직접 만들기 때문에 데이터베이스 구조에 맞추어 작성 되어 있음
  - Django는 fixtures를 사용해 모델에 초기 데이터를 제공


## 초기 데이터 제공하기
- `dumpdate`
  - 데이터베이스의 모든 데이터를 출력
  - 여러 모델을 하나의 json 파일로 만들 수 있음
  - ```pyhton
    python manage.py dumpdata [app_name[.ModelName] [app_name.[.ModelName] ...]] > filename.json
    ```

- fixtures 생성
  - ```pyhton
    python manage.py dumpdata --indent 4 articles.article > articles.json
    ```
  - ```pyhton
    python manage.py dumpdata --indent 4 articles.user > users.json
    python manage.py dumpdata --indent 4 articles.comment > users.comment
    ```

- `loaddata`
  - fixtures 데이터를 데이터베이스로 불러오기

- fixtures 기본 경로
  - app_name/fixtures/
  - Django는 설치된 모든 app의 디렉토리에서 fixtures 폴더 이후의 경로로 fixtures 파일을 찾아 load함

- fixtures 불러오기
  - articles/fixtures/json 파일
  - db.sqlite3 파일 삭제후 migrate 진행
  - ```python
    python manage.py loaddata articles.json users.json comments.json
    ```

- loaddata 순서 주의사항
  - loaddata를 한번에 실행하지 않고 하나씩 실행한다면 모델 관계에 따라 순서가 중요할 수 있음
    - comment는 article에 대한 key 및 user에 대한 key 필요
    - article은 user에 대한 key 필요
  - 즉, 현재 모델 관계에서는 user -> article -> comment 순으로 data를 넣어야 오류가 발생하지 않음
  - ```python
    python manage.py loaddata user.json
    python manage.py loaddata articles.json
    python manage.py loaddata comments.json
    ```

## 참고
- `fixtures는 직접 만드는 것이 아니다.`
  - 반드시 dumpdata를 사용하여 생성하는 것

- loaddata 시 encoding codec 관련 에러가 발생하는 경우
  - 2가지 방법중 택 1
  1. dumpdata시 추가 옵션 작성
  - ```python
    python -Xutf8 manage.py dumpdata [생략]
    ```
  2. 메모장 활용
  - 1 - 메모장으로 json 파일열기
  - 2 - "다른 이름으로 저장" 클릭
  - 3 - 인코딩을 UTF8로 선택 후 저장