# Django란?

Django는 Python으로 웹 프로그래밍을 하기 위한 기능을 제공하는 웹 프레임워크이다. 웹 프레임워크는 웹 애플리케이션이 실행할 때 필요한 쿠키 및 세션 처리, 로그인 처리, DB 처리와 같은 기능을 개발자에게 제공하여 웹 애플리케이션에 필요한 기본적인 기능을 직접 개발하지 않아도 사용할 수 있도록 돕는다. 따라서 개발자는 웹 애플리케이션의 개발 기간을 단축시키고 안정적인 웹 애플리케이션을 개발할 수 있다.

Django를 사용할 때 가장 큰 장점은 보안 기능을 제공한다는 점과 오랜 시간 사용되며 잘 다듬어진 웹 프레임워크라는 점이다. Django는 SQL 인젝션이나 XSS, CSRF와 같은 보안 공격을 방어하는 기능을 사용할 수 있어 개발자가 보안을 위한 코드를 직접 작성할 필요는 없다. 또한 오랫동안 오류 수정과 새로운 기능 추가로 웹 프로그래밍을 위한 대부분의 기능이 준비되어 있어 “바퀴를 새로 발명하지 말라”라는 말을 충실히 따를 수 있다.

- SQL 인젝션: 악의적인 SQL문을 주입 및 실행하여 DB를 비정상적인 조작하며 공격하는 방식
    - https://ko.wikipedia.org/wiki/SQL_삽입
- XSS: Cross-site scripting의 약자로 웹 관리자가 아닌 자가 웹 페이지에 악성 스크립트를 삽입하여 사용자의 정보(쿠키, 세션)을 탈취하거나 자동으로 비정상적인 기능을 수행하도록 하는 공격 방식. 다른 웹 사이트와 정보를 교환하는 방식으로 작동해서 사이트간 스크립팅이라는 이름이 붙여졌다.
    - https://ko.wikipedia.org/wiki/사이트_간_스크립팅
- CSRF: Cross-site request forgery의 약자로 사용자가 자신의 의도와 무관하게 공격자가 의도한 행위(수정, 삭제, 등록)을 특정 웹 사이트에 요청하게 하는 공격 방식. XSS는 사용자가 특정 웹사이트를 신뢰하는 점을 노린 것이라면 CSRF는 특정 웹사이트가 사용자의 웹 브라우저를 신뢰하는 것을 노린 공격. 과거 전자상거래 서비스 옥션에서 발생한 개인정보 유출 사건에서 사용한 공격 방식 중 하나.

## Django 설치

Django를 설치하기 위해서 먼저 Python이 설치되어 있어야 한다. Python은 Python 공식 사이트에서 다운로드 받거나 아나콘다를 사용하여 설치한다. 다음 Django 패키지를 사용할 가상환경을 생성 및 활성화 한 후 Django 설치를 진행한다.

### 설치 순서

1. Django 프로젝트를 진행할 디렉토리에서 가상환경을 생성 및 활성화 → django 루트 디렉토리에서 진행
    
    ```bash
    sleepyowl ~/Study/django_projects
    pipenv --python 3.12
    pipenv shell
    (django_projects)  sleepyowl ~/Study/django_projects
    ```
    
    - pipenv를 사용해서 가상환경 생성 및 가상환경 실행
    - 터미널에서  가상환경을 바꾸면 괄호 안의 이름이 바뀌며 활성화되어 있는 가상환경이 무엇인지 알려준다.
2. pip를 사용해 Django 설치
    
    ```bash
    pip install django
    Collecting django
      Using cached Django-5.0.6-py3-none-any.whl.metadata (4.1 kB)
    Collecting asgiref<4,>=3.7.0 (from django)
      Using cached asgiref-3.8.1-py3-none-any.whl.metadata (9.3 kB)
    Collecting sqlparse>=0.3.1 (from django)
      Using cached sqlparse-0.5.0-py3-none-any.whl.metadata (3.9 kB)
    Using cached Django-5.0.6-py3-none-any.whl (8.2 MB)
    Using cached asgiref-3.8.1-py3-none-any.whl (23 kB)
    Using cached sqlparse-0.5.0-py3-none-any.whl (43 kB)
    Installing collected packages: sqlparse, asgiref, django
    Successfully installed asgiref-3.8.1 django-5.0.6 sqlparse-0.5.0
    ```
    
    - pip를 사용해 최신 버전의 django 설치를 진행
        - pip install django==’version’을 사용하면 특정 버전의 django 설치 가능
    - pip로 패키지를 설치하면 pip가 알아서  django에 필요한 종속성을 함께 설치해준다.

## Django 프로젝트 생성

Django의 프로젝트는 쉽게 말해 하나의 웹 사이트를 의미한다. 이 프로젝트 안에 웹 사이트에서 동작할 기능이나 애플리케이션을 개발하여 웹 사이트를 구성한다. 

1. 루트 디렉토리 생성
    - Django 프로젝트를 관리할 루트 디렉토리가 필요하다.
    - 프로젝트 관리가 용이하고 루트 디렉토리에서 가상환경을 활성화시키면 하위 디렉토리도 같은 가상환경에서 개발을 진행할 수 있다.
2. Django 프로젝트를 진행할 디렉토리를 루트 디렉토리 하위에 생성 및 이동
    
    ```bash
    mkdir test_site
    cd test_site
    ```
    
3. 프로젝트 디렉토리에서 django 프로젝트 생성
    
    ```bash
    django-admin startproject config .
    ```
    
4. 생성된 프로젝트 구조 확인
    
    ```bash
    tree
    .
    ├── config
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── manage.py
    ```
    
    - 위와 같은 구조의 디렉토리가 생성되었으면 성공적으로 django 프로젝트를 생성한 것

## Django 개발 서버 구동

Django 디렉토리에는 manage.py라는 파일이 있다. 이 파일을 살펴보면 아래와 같은 간단한 코드로 작성되어 있다.

```bash
#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys

def main():
    """Run administrative tasks."""
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)

if __name__ == '__main__':
    main()
```

이 파일을 python [manage.py](http://manage.py) runserver 명령으로 실행하면 runserver가 execute_from_command_line함수의 인자로 전달되고 execute_from_command_line을 실행한다. 이 함수의 자세한 동작과정은 아래 링크에서 설명하고 있다.

https://velog.io/@kitoram/python-manage.py-runserver는-어떻게-동작하는가1

1. 개발 서버 구동
    
    ```bash
    python manage.py runserver
    ```
    
    - 개발 서버가 구동되면 [localhost:8000](http://localhost:8000) 혹은 127.0.0.1:8000으로 접속 할 수 있다.
    - SIGINT(ctrl + c)를 입력하면 개발서버를 종료 가능
2. 개발 서버 접속
    
    <img src="/images/Django_devserver.png" width="75%" height="75%" title="django devserver" alt="django devserver">   
    
    - 개발 서버가 성공적으로 실행되면 웹 브라우저에서 localhost:8000로 접속했을 때 위와 같은 페이지가 표시된다.

## IDE 설정 - VSCode

VSCode에 Django 프로젝트 설정하는 것은 간단하다. VSCode로 프로젝트 디렉토리를 연 다음에 python 인터프리터를 생성한 가상 환경의 python을 설정하면 된다.

1. VScode로 django 프로젝트 열기
    
    <img src="/images/VSCode_django_1.png" width="75%" height="75%" title="vscode django 1" alt="vscode django 1">   
    
2. Python 인터프리터 변경
    - VSCode 상단에서 “> python 인터프리터 선택” 입력
        
        <img src="/images/VSCode_django_2.png" width="75%" height="75%" title="vscode django 2" alt="vscode django 2">   
        
    - 가상환경의 python 인터프리터를 선택
        
        <img src="/images/VSCode_django_3.png" width="75%" height="75%" title="vscode django 3" alt="vscode django 3">   
        
3. 테스트 
    
    <img src="/images/VSCode_django_4.png" width="75%" height="75%" title="vscode django 4" alt="vscode django 4">   
    
    - Setting.py에서 TIME_ZONE을 Asia/Seoul로 변경 후 manage.py 실행
    - VSCode 터미널에서 실행했을 때 iterms에서 실행한 것과 동일하게 개발 서버가 동작하면 성공