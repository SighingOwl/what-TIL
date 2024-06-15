# Django 기초

# 앱 생성하기

일반적으로 앱이라고 하면 우리가 스마트폰에서 사용하는 애플리케이션을 생각하지만 웹에서 동작하는 애플리케이션은 사용자가 웹 사이트에서 사용하는 각각의 기능을 의미한다.

## pybo app

1. pybo app 생성하기
    
    django 앱을 생성하기 위해서는 프로젝트 디렉토리에서 django-admin startapp 명령을 사용한다.
    
    ```bash
    django-admin startapp pybo
    cd pybo
    tree
    .
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py
    ```
    
    - 생성한 앱 디렉토리의 내부 구조가 위와 같으면 성공적으로 앱을 생성한 것이다.
2. 개발 서버를 동작하여 앱에 접근
    
    개발서버를 동작시킨 후 localhost:8000/pybo로 접근하면 페이지를 찾을 수 없다는 페이지가 웹 브라우저에 출력되며 개발서버를 실행한 터미널에도 404 Not Found 로그가 출력된다.
    
    - 오류 페이지
        
        <img src='/images/django_app_1.png' width='50%' height='50%' title='django app' alt='django app'>   
        
    - 터미널 오류 로그
        
        ```bash
        [13/Jun/2024 15:49:06] "GET / HTTP/1.1" 200 10629
        Not Found: /pybo
        [13/Jun/2024 15:49:12] "GET /pybo HTTP/1.1" 404 2089
        ```
        
        - 200 코드는 localhost:8000에 접근했을 때 발생한 코드
        - 404 코드는 localhost:8000/pybo으로 pybo 앱에 접근했을 때 코드
    
    이 오류가 발생한 이유는 pybo로 접근할 수 있는 url이 config/urls.py에 매핑이 되어있지 않아 django가 pybo 앱을 찾을 수 없기 때문이다.
    
3. config/urls.py 수정
    
    프로젝트의 config 디렉토리에서 urls.py을 찾을 수 있고 urls.py는 아래의 코드가 작성되어 있다.
    
    - urls.py
        
        ```python
        from django.contrib import admin
        from django.urls import path
        
        urlpatterns = [
            path('admin/', admin.site.urls),
        ]
        ```
        
    
    위 코드에서 url 매핑을 추가하여 django가 pybo 앱에 url로 접근할 수 있게 한다.
    
    - 수정된 urls.py
        
        ```python
        from django.contrib import admin
        from django.urls import path
        from pybo import views
        
        urlpatterns = [
            path('admin/', admin.site.urls),
            path('pybo/', views.index),
        ]
        ```
        
        - 매핑할 url은 urlpatterns 내부에 추가한다.
        - path 함수를 사용해 pybo/ 경로를 view.index에 매핑
        - path에 localhost:8000을 제외한 주소를 작성한 이유는 호스트명과 포트번호가 django 실행환경에 따라서 변할 수 있는 값이기 때문
        - pybo뒤에 ‘/’를 붙인 이유는 ‘/’가 있을 때 사용자가 주소창에 ‘/’를 붙이지 않아도 django가 자동으로 붙여주는 기능을 사용할 수 있기 때문
4. urls.py 수정 후 페이지 접근
    
    처음과 다르게 “사이트에 연결할 수 없음” 메시지가 브라우저 출력되고 터미널에 urls.py에 추가한 view.index가 없다는 오류 메시지가 출력된다.
    
    - 터미널에 출력된 오류 메시지
        
        ```python
        File "/Users/sleepyowl/Study/django_projects/test_site/config/urls.py", line 23, in <module>
            path('pybo/', views.index),
                          ^^^^^^^^^^^
        AttributeError: module 'pybo.views' has no attribute 'index'
        ```
        
5. pybo 앱에 views.py를 작성 후 접근
    - pybo/views.py
        
        ```python
        from django.http import HttpResponse
        
        def index(request):
            return HttpResponse("Hello World!")
        ```
        
    
    views.py를 작성하면 django는 자동으로 소스코드를 인식하여 정상으로 돌아온다. 이때 발생하는 http 응답코드는 301번으로 url이 변경된 주소로 영구 리다이렉팅 되었음을 의미한다.
    
    - 로그
        
        ```bash
        [13/Jun/2024 16:12:45] "GET /pybo HTTP/1.1" 301 0
        [13/Jun/2024 16:12:45] "GET /pybo/ HTTP/1.1" 200 12
        ```
        

## URL 분리

프로젝트 디렉토리 구조를 살펴보면 urls.py는 config 디렉토리에만 있고 앱 디렉토리에는 없는 것을 확인할 수 있다. 이것은 앱에 접근하기 위한 경로가 추가될 때마다 urls.py에 추가해야 함을 의미한다. 하지만 이러한 방식은 프로젝트 구성에 비효율적인 방법이어서 URL 분리가 필요한 것이다.

URL 분리는 애플리케이션 별로 관련된 URL 매핑 정보를 구성하는 것을 의미한다. 아래 코드는 urls.py를 수정하여 pybo 앱과 관련된 URL을 분리한 코드이다.

- URL 분리한 urls.py 코드
    
    ```python
    from django.contrib import admin
    from django.urls import path, include
    from pybo import views
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('pybo/', include('pybo.urls')),
    ]
    ```
    
    - include를 함수를 사용해 ‘pybo/’로 시작되는 페이지 요청은 모두 pybo 디렉토리에 있는 urls.py를 참조하라는 의미
    - 이를 참조하기 위해 앱의 디렉토리에 별도의 urls.py 파일이 필요하다.

### pybo/urls.py 생성

- pybo/urls.py
    
    ```python
    from django.urls import path
    
    import views
    
    urlpatterns = [
        path('', views.index)
    ]
    ```
    
    - pybo/urls.py를 생성 후에 localhost:8000/pybo에 접근하면 URL 분리하기 이전과 동일하게 접근할 수 있다.

URL을 분리했을 때 django가 URL 매핑을 찾는 과정은 DNS로 도메인 이름에 해당하는 ip 주소를 찾는 방법과 유사하다. pybo/에 접근할 때 django는 가장 먼저 config/urls.py에서 pybo/를 찾는다. 그 다음 pybo/로 시작되는 주소는 pybo/urls.py에서 참조하여 URL 매핑을 찾게 된다. DNS도 동일하게 루트 DNS 서버부터 차상위 도메인 서버까지 계층적으로 탐색하며 ip 주소를 찾는 점이 유사하다.

# Model

일반적으로 웹에서 데이터를 관리할 때 SQL 쿼리문을 사용한다. 하지만 django에서는 모델을 사용해서 데이터 관리를 수행할 수 있다. 그 이유는 django가 ORM - Object Relational Mapping 기능이 있기 때문이다. ORM은 객체를 사용해 연결한다는 의미로 애플리케이션과 DB를 SQL 언어가 아닌 애플리케이션 개발언어로 DB에 접근할 수 있게 하는 기능이다. 

ORM은 SQL을 사용할 때 발생하는 3가지 단점을 보완한다.

1. 같은 동작을 수행하는 쿼리를 개발자마다 다르게 작성하는 것을 방지
2. 잘못된 쿼리문으로 시스템 성능이 저하되는 것을 방지
3. DB 엔진에 대한 의존성을 없애 유지보수에 용이

## Migrate

프로젝트 디렉토리에 db.sqlite3라는 것이 있는데 db.sqlite3는 config/settings.py에서 DATABASES 확인했을 때 db 엔진으로 사용하고 있음을 알 수 있다.

- config/settings.py
    
    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }
    ```
    

SQLite는 파일 기반의 소규모 데이터베이스이며 개발 단계에서 간단하게 db를 구현하기 위해 사용한다. 실제 서비스를 운영할 때 사용하지 않고 상용 DB 엔진으로 바꾼다.

### Migrate 명령으로 DB 테이블 생성

1. manage.py로 migrate 실행
    
    ```bash
    python manage.py migrate
    ```
    
    - migrate는 admin, auth, contenttypes, sessions 앱이 사용하는 테이블이 생성된다.

## 모델 생성

1. 모델 설계
    
    여느 DB가 그렇듯 DB에 삽입할 데이터의 형태를 설계해야 한다.
    
    - 질문 모델
        
        
        | 속성명 | 설명 |
        | --- | --- |
        | subject | 질문의 제목 |
        | content | 질문의 내용 |
        | create_date | 질문을 작성한 일 |
    - 답변 모델
        
        
        | 속성명 | 설명 |
        | --- | --- |
        | question | 질문 → 질문에 대한 답변이므로 답변 모델에 질문과 답변이 모두 포함되어야 함 |
        | content | 답변의 내용 |
        | create_date | 답변을 작성한 일시 |
2. pybo 디렉토리에 모델 작성
    
    ```python
    from django.db import models
    
    # 질문 모델
    class Question(models.Model):
        subject = models.CharField(max_length=200)
        content = models.TextField()
        create_date = models.DateTimeField()
    
    # 답변 모델
        # 질문에 대한 답변이 필요하므로 question은 Question 모델에서 외래키로 가져온다.
        # question은 삭제되면 질문도 함께 삭제되도록 삭제 옵션은 CASCADE로 설정
    class Answer(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        content = models.TextField()
        create_date = models.DateTimeField()
    ```
    
3. 테이블 생성을 위해 setting.py에 pybo 앱 등록
    
    ```python
    INSTALLED_APPS = [
        ...
        'pybo.apps.PyboConfig',
    ]
    ```
    
    - pybo.apps.PyboConfig는 pybo 앱을 생성할 때 자동으로 생성된 클래스
    - 이 클래스가 setting.py에 등록이 되어야 django가 pybo 앱을 인식할 수 있다.
    - DB은 앱에 종속적이므로 앱이 DB 작업을 수행하기 위해서 반드시 setting.py에 앱을 등록해야 한다.
4. migrate로 DB 생성
    
    ```bash
    python manage.py migrate
    Operations to perform:
      Apply all migrations: admin, auth, contenttypes, sessions
    Running migrations:
      No migrations to apply.
      Your models in app(s): 'pybo' have changes that are not yet reflected in a migration, and so won't be applied.
      Run 'manage.py makemigrations' to make new migrations, and then re-run 'manage.py migrate' to apply them.
    ```
    
    - migrate를 실행했을 때 pybo의 모델이 적용되지 않는 문제가 발생
    - 모델이 생성 혹은 변경되었을 때 migrate 명령을 수행하기 위해 테이블 작업 파일이 필요
    - 따라서 makemigrations 명령을 먼저 수행해야 한다.
        
        
    
    ```bash
    python manage.py makemigrations
    Migrations for 'pybo':
      pybo/migrations/0001_initial.py
        - Create model Question
        - Create model Answer
    ```
    
    - makemigrations 명령을 테이블 생성 명령이 아닌 테이블 생성을 위한 파일을 생성하는 것이므로 migrate와 혼동할 수 있다.
    - makemigrations 명령을 수행하면 모델이 작성된 앱의 migrations 디렉토리에 DB 초기화를 위한 파일이 생성된 것을 확인할 수 있다.
        
        <img src='/images/django_model_1.png' width='50%' height='50%' title='django model' alt='django model'>   
        
    
    ```bash
    python manage.py migrate
    Operations to perform:
      Apply all migrations: admin, auth, contenttypes, pybo, sessions
    Running migrations:
      Applying pybo.0001_initial... OK
    ```
    
    - 다시 migrate 명령을 실행하면 pybo 앱을 위한 테이블을 생성

## 데이터 생성 및 저장, 조회

모델에서 데이터를 관리하는 방법은 django shell을 사용하는 방법이 있다. 터미널에서 사용하는 shell과 다른 점은 django에 필요한 환경이 자동으로 설정되어 추가 설정없이 django 기능을 사용할 수 있다.

1. django shell 실행
    
    ```bash
    python manage.py shell
    Python 3.12.4 (v3.12.4:8e8a4baf65, Jun  6 2024, 17:33:18) [Clang 13.0.0 (clang-1300.0.29.30)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>>
    ```
    
    - django shell을 실행하면 명령를 입력할 수 있는 shell이 실행이 된다.
2. 생성한 모델 임포트
    
    ```bash
    from pybo.models import Question, Answer
    ```
    
    - 모델 임포트는 python에서 임포트하는 방법과 동일
3. 모델 데이터 생성
    
    ```bash
    from django.utils import timezone
    q = Question(subject='pybo가 무엇인가요?', content='pybo에 대해서 알고 싶습니다.', create_date=timezone.now())
    q.save()
    ```
    
    - 모델 데이터 생성은 python에서 객체를 생성할 때와 동일한 방법으로 생성 가능
    - 모델 데이터 객체를 생성한 다음 .save()까지 수행해야 데이터가 DB에 저장된다.
4. 모델 데이터 id값 확인
    
    ```bash
    q.id
    1
    ```
    
    - django는 데이터 생성시 id를 자동으로 넣어준다. → 1, 2, 3 … 순차적으로
    - id는 데이터의 유일한 값이며 primary key라고 부르기도 한다.
5. 모델 데이터 모두 조회
    
    ```bash
    Question.objects.all()
    <QuerySet [<Question: Question object (1)>, <Question: Question object (2)>]>
    ```
    
    - 데이터를 조회하고자 하는 모델의 모든 데이터를 조회
    - 데이터의 유형과 데이터 id만 확인 가능
6. 모델 데이터 조회 결과에 속성값 포함
    
    데이터의 유형과 id만으로는 데이터가 무엇을 의미하는지 정확히 파악하기 힘들다. 그래서 모델 클래스에 __str__ 메서드를 추가해서 속성값이 출력되도록 한다.
    
    ```python
    class Question(models.Model):
        subject = models.CharField(max_length=200)
        content = models.TextField()
        create_date = models.DateTimeField()
    
        def __str__(self):
            return self.subject
    ```
    
7. 속성값이 포함된 데이터 조회
    
    모델 클래스이 변경되면 이를 적용하기 위해 django shell을 다시 시작한 후 조회
    
    ```bash
    from pybo.models import Question, Answer
    Question.objects.all()
    <QuerySet [<Question: pybo가 무엇인가요?>, <Question: 장고 모델 질문입니다>]>
    ```
    
    - django shell을 다시 시작하면 모델을 다시 임포트를 해야한다.
    - 데이터를 조회하면 속성값이 표시된다.
8. 모델 데이터를 조건문을 사용해 조회
    
    filter 명령을 사용하면 조건에 맞는 데이터만 조회 가능
    
    ```bash
    Question.objects.filter(id=1)
    <QuerySet [<Question: pybo가 무엇인가요?>]>
    ```
    
9. 모델 데이터를 하나만 조회
    
    get 명령을 사용하면 하나의 데이터만 조회 가능
    
    ```bash
    Question.objects.get(id=1)
    <Question: pybo가 무엇인가요?>
    ```
    
    - filter와 차이점은 filter는 조건에 맞는 여러 데이터를 반환하여 Queryset으로 반환하지만 get은 하나의 데이터만 반환하므로 데이터 객체를 반환
10. 조건에 맞지 않는 데이터 조회시 get과 filter의 차이
    - get
        
        ```bash
        Question.objects.get(id=3)
        Traceback (most recent call last):
          File "<console>", line 1, in <module>
          File "/Users/sleepyowl/.local/share/virtualenvs/django_projects-SV9ktjei/lib/python3.12/site-packages/django/db/models/manager.py", line 87, in manager_method
            return getattr(self.get_queryset(), name)(*args, **kwargs)
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
          File "/Users/sleepyowl/.local/share/virtualenvs/django_projects-SV9ktjei/lib/python3.12/site-packages/django/db/models/query.py", line 649, in get
            raise self.model.DoesNotExist(
        pybo.models.Question.DoesNotExist: Question matching query does not exist.
        ```
        
        - get 명령은 조건에 맞지 않는 데이터를 조회하면 맞는 데이터가 없어서 오류가 발생
        - get은 무조건 1개의 데이터를 반환해야하므로 발생한 오류
    - filter
        
        ```bash
        Question.objects.filter(id=3)
        <QuerySet []>
        ```
        
        - filter는 조건에 맞지 않는 데이터를 조회하면 빈 QuerySet을 반환한다.
11. 문자열이 포함된 데이터 조회
    
    ```bash
    Question.objects.filter(subject__contains='장고')
    <QuerySet [<Question: 장고 모델 질문입니다>]>
    ```
    
    - 속성값에 문자열이 포함된 속성에 언더스코어 2개를 붙인 다음 조건을 입력

## 데이터 수정

1. 수정할 데이터를 변수로 불러온다.
    
    ```bash
    q = Question.objects.get(id=2)
    q
    <Question: 장고 모델 질문입니다>
    ```
    
    - 프로그래밍을 할 때 당연할 수 있지만 데이터를 변수로 불러와 데이터에 접근해야한다.
2. 객체의 속성값을 수정
    
    ```bash
    q.subject = 'Django Model Question'
    q.save()
    Question.objects.get(id=2)
    <Question: Django Model Question>
    ```
    
    - 객체를 불러와 속성값을 수정하는 것으로 간단하게 데이터 수정이 가능

## 데이터 삭제

데이터 수정과 비슷하게 먼저 삭제할 데이터를 변수에 불러온 후 삭제를 진행

1. 데이터 조회 후 삭제
    
    ```bash
    q = Question.objects.get(id=1)
    q.delete()
    (1, {'pybo.Question': 1})
    ```
    
    - delete() 함수로 데이터 삭제 가능
    - 삭제 후 데이터의 id와 삭제된 모델 데이터 개수가 출력
        
        ```bash
        {id, {'삭제된 데이터의 모델': 개수}}
        ```
        

## 연결된 데이터

모델을 생성할 때 외래키를 사용해서 데이터를 연결했었다. 따라서 외래키가 포함된 모델의 데이터를 생성할 때는 외래키의 데이터를 가져온 후 생성해야 한다.

1. 외래키 데이터 조회 및 데이터 생성
    
    ```bash
    q = Question.objects.get(id=2)
    q
    <Question: Django Model Question>
    from django.utils import timezone
    a = Answer(question=q, content='네 자동으로 생성됩니다.', create_date=timezone.now())
    a.save()
    ```
    
    - 변수에 외래키 데이터를 가져온 후 생성할 모델 데이터의 속성값에 사용
2. 데이터 조회
    
    ```bash
    a = Answer.objects.get(id=1)
    a
    <Answer: Answer object (1)>
    a.question
    <Question: Django Model Question>
    ```
    
    - 생성한 데이터를 불러와 외래키가 포함된 속성값을 출력하면 연결된 데이터를 조회 가능
3. 외래키 데이터로 데이터 조회
    
    외래키가 포함된 모델의 데이터로 데이터를 조회가 가능하지만 반대의 경우도 가능하다.
    
    ```bash
    q = Question.objects.get(id=2)
    q.answer_set.all()
    <QuerySet [<Answer: Answer object (1)>]
    ```
    
    - 연결모델명_set 명령을 사용하면 2번과 반대 방향으로 데이터를 조회할 수 있다.

# Django Admin

Django는 관리자 권한을 가진 사용자가 웹 서버의 컨텐츠를 관리할 수 있도록 관리자 전용 페이지를 제공하는데 그것이 Django Admin이다.

Django Admin을 사용하기 위해서는 관리자 권한이 필요하므로 슈퍼유저를 생성해야 한다.

## Django Admin 사용

1. 슈퍼 유저 생성
    
    ```bash
    python manage.py createsuperuser
    Username (leave blank to use 'sleepyowl'): admin
    Email address: admin@example.com
    Password:
    Password (again):
    The password is too similar to the username.
    This password is too short. It must contain at least 8 characters.
    This password is too common.
    Bypass password validation and create user anyway? [y/N]: y
    Superuser created successfully.
    ```
    
    - manage.py에서 createsuperuser를 사용해 슈퍼 유저 생성 가능
    - 슈퍼 유저의 사용자 이름과 이메일, 패스워드를 설정
    - 비밀번호 검증 우회는 필요에 따라 설정
2. Django Admin 접속
    - localhost:8000/admin으로 접속하면 관리자 페이지에 접속 가능
        
        <img src='/images/django_admin_1.png' width='50%' height='50%' title='django admin 1' alt='django admin 1'>   
        
        - 생성한 슈퍼 유저로 로그인 진행
    - 로그인 후 초기 화면
        
        <img src='/images/django_admin_2.png' width='50%' height='50%' title='django admin 2' alt='django admin 2'>   
        

### Django Admin에서 모델 관리

Django Admin에서 모델에 데이터를 삽입, 수정, 삭제가 가능

1. 모델을 생성한 앱의 admin.py 코드를 수정하여 모델을 admin에 등록
    
    ```python
    from django.contrib import admin
    from .models import Question
    
    admin.site.register(Question)
    ```
    
2. Django Admin 페이지 새로고침
    
    <img src='/images/django_admin_3.png' width='50%' height='50%' title='django admin 3' alt='django admin 3'>   
    
    - 모델을 admin에 등록하면 admin 페이지에 모델이 속한 앱과 모델이 추가 됨
3. 데이터 추가
    
    <img src='/images/django_admin_4.png' width='50%' height='50%' title='django admin 4' alt='django admin 4'>   
    
    - 데이터를 추가하고자 하는 모델에서 Add를 누르면 위와 같은 화면이 출력된다.
    - 모델의 데이터에 추가할 내용을 입력 후 Save
    
    <img src='/images/django_admin_5.png' width='50%' height='50%' title='django admin 5' alt='django admin 5'>   
    
    - 저장하면 모델 데이터를 확인 가능
4. Django Admin에 데이터 검색 기능 추가
    
    ```python
    from django.contrib import admin
    from .models import Question
    
    class QuestionAdmin(admin.ModelAdmin):
        search_fields = ['subject']
    
    admin.site.register(Question, QuestionAdmin)
    ```
    
    - admin.py에 필드 검색을 위한 search_fields 함수가 포함된 클래스를 생성
5. Django Admin 새로고침 후 검색 기능 확인
    
    <img src='/images/django_admin_6.png' width='50%' height='50%' title='django admin 6' alt='django admin 6'>   
    
    - 특정 문자열이 포함된 데이터 검색 가능