# Django 가이드

<br>

## 사전 준비 사항
- VSCode extension 설치
  - `Django` 설치
  - `SQLite Viewer` 설치

- VSCode extension 관련 설정
  1. Ctrl(command) + Shift + p => Json 검색 =>  Preferences: Open User Settings (JSON) 선택

  2. 설정 코드 작성
  - ```python
    // settings.json

    {
      ... 생략 ...,

      // python
      "files.associations": {
        "**/*.html": "html",
          "**/templates/**/*.html": "django-html",
        "**/templates/**/*": "django-txt",
        "**/requirements{/**,*}.{txt,in}": "pip-requirements"
      },
      "emmet.includeLanguages": {
        "django-html": "html"
      }
    }
    ```