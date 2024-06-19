# 로그인, 로그아웃

django.contrib.auth 앱을 사용해서 로그인, 로그아웃 기능을 구현

## common 앱

로그인 기능은 일반적으로 하나의 웹 사이트 전체에서 동작하는 기능이다. 예를 들어 네이버에 한번 로그인을 하면 메일 서비스, 블로그 서비스 등 네이버의 서비스를 모두 사용할 수 있으며 서비스를 사용할 때 각 서비스 별로 로그인을 다시 요구하는 경우는 없다. 따라서 로그인 기능을 pybo 앱에 구현하는 것보다 별도의 앱으로 구현하는 것이 바람직하다. 로그인 기능은 다른 앱에서도 공통적으로 동작하는 기능이므로 common 앱에 구현한다.

1. common 앱 생성
    
    ```bash
    (django_projects) sleepyowl ~/Study/django_projects/test_site
    django-admin startapp common
    ```
    
    - pybo 앱을 생성했던 방법과 동일하게 웹 프로젝트 디렉토리에서 앱을 생성
2. settings.py에 앱 등록 및 urls.py에 url 등록
    - config/settings.py 수정
        
        ```python
        ...
        # Application definition
        
        INSTALLED_APPS = [
            ...
            'common.apps.CommonConfig',
        ]
        ...
        ```
        
    - config/urls.py 수정
        
        ```python
        ...
        urlpatterns = [
            path('admin/', admin.site.urls),
            path('pybo/', include('pybo.urls')),
            path('common/', include('common.urls')),
        ]
        ...
        ```
        
        - pybo와 동일하게 url을 앱 namespace에서 관리하기 위해 include 함수를 사용
    - common/urls.py 생성
        
        ```python
        app_name = 'common'
        
        urlpatterns = [
            
        ]
        ```
        

## 로그인 구현

1. 내비게이션 바의 로그인 버튼 경로 수정
    - templates/navbar.html
        
        ```html
        <!-- 로그인 버튼 Start -->
            <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'common:login' %}">로그인</a>
                    </li>
                </ul>
            </div>
            <!-- 로그인 버튼 End -->
        ```
        
        - 버튼 구현만을 위해 경로를 #로 설정해둔 것을 common 앱의 login 경로로 수정
    - common/urls.py에 login url 추가
        
        ```python
        from django.urls import path
        from django.contrib.auth import views as auth_views
        
        app_name = 'common'
        
        urlpatterns = [
            path('login/', auth_views.LoginView.as_view(), name='login'),
        ]
        ```
        
        - pybo 앱에서는 질문 및 답변 조회, 등록을 위한 함수를 views.py에 정의해서 사용했지만 로그인 기능은 django.contrib.auth 앱의 LoginView 클래스를 활용해서 views.py에 로그인에 필요한 함수를 정의하지 않아도 된다.
2. 로그인 탬플릿 생성
    - 지금 상태에서는 로그인 경로와 연결된 탬플릿 파일이 없어서 탬플릿이 존재하지 않는다는 오류 페이지가 출력된다.
    - LoginView는 탬플릿을 찾을 때 registration이라는 디렉토리에서 탬플릿을 찾게되는데 common 앱을 사용해 로그인 기능 구현할 예정이어서 LoginView가 참조할 탬플릿의 경로도 common의 login.html로 변경해야 한다.
    - common/urls.py 수정
        
        ```python
        ...
        urlpatterns = [
            path('login/', auth_views.LoginView.as_view(template_name='common/login.html'), name='login'),
        ]
        ...
        ```
        
        - LoginView에 template_name 속성으로 login할 때 참조할 탬플릿 경로를 지정
    - common/login.html 생성
        
        ```html
        {% extends "base.html" %}
        {% block content %}
        <div class="container my-3">
            <form method="post" class="post-form" action="{% url 'common:login' %}">
                {% csrf_token %}
                {% include "form_errors.html" %}
                <!-- 사용자 ID 입력 -->
                <div class="form-group my-3">
                    <label for="username">사용자ID</label>
                    <input type="text" class="form-control my-1" name="username" id="username" value="{{ form.username.value|default_if_none:'' }}">
                </div>
                <!-- 비밀번호 입력 -->
                <div class="form-group my-3">
                    <label for="password">비밀번호</label>
                    <input type="password" class="form-control my-1" name="password" id="password" value="{{ form.password.value|default_if_none:'' }}">
                </div>
                <!-- 로그인 버튼 -->
                <button type="submit" class="btn btn-primary my-2">로그인</button>
            </form>
        </div>
        {% endblock %}
        ```
        
        - username과 password는 django.contrib.auth에서 요구하는 필수 항목
        - form_errors.html은 로그인 실패시 로그인 실패 원인을 표시하는 페이지
    - templates/form_errors.html 생성
        - 로그인 뿐만 아니라 form의 필드에 적절한 값을 넣지 않는 모든 오류에 대해 원인을 표시하기 위해 프로젝트 디렉토리의 templates 디렉토리에 생성
        
        ```html
        {% if form.errors %}
            {% for field in form %}
                <!-- 필드 오류 출력 -->
                {% for error in field.errors %}
                    <div class="alert alert-danger">
                        <strong>{{ field.label }}</strong>
                        {{ error }}
                    </div>
                {% endfor %}
            {% endfor %}
            <!-- None 필드 오류 출력 -->
            {% for error in form.non_field_errors %}
                <div class="alert alert-danger">
                    <strong>{{ error }}</strong>
                </div>
            {% endfor %}
        {% endif %}
        ```
        
        - 필드에 잘못된 형식의 값이 입력되거나 누락되었을 때 필드오류가 발생
        - 입력값과 관계 없이 발생하는 오류일 때 넌필드 오류 발생
    - 로그인 페이지 확인
        
        <img src="/images/login_page_1.png" width="50%" height="50%" title="login page 1" alt="login page 1">    
        
    - 로그인 페이지 오류 확인
        
        <img src="/images/login_page_2.png" width="50%" height="50%" title="login page 2" alt="login page 2">    
        
        - 필드를 비워둔 채로 로그인하면 필드를 채우라는 경고가 발생
3. 로그인 성공 시 넘어갈 페이지 등록
    - 이 상태에서 로그인을 성공하면 이동할 페이지를 등록하지 않아 로그인에 성공하면 404 오류가 발생한다.
    - django.contrib.auth 앱이 로그인에 성공하면 “/account/profile/” URL로 페이지를 리다이렉션하는데 이 URL을 등록하지 않았기 때문
    - settings.py에 LOGIN_REDIRECT_URL을 추가해 index.html로 이동하도록 수정
        
        ```python
        ...
        # 로그인 성공 시 자동으로 이동할 URL
        LOGIN_REDIRECT_URL = '/'
        ```
        
    - ‘/’에 대한 URL 매핑 - config/urls.py
        
        ```python
        ...
        urlpatterns = [
        		...
            path('', views.index, name='index'),    # / 페이지에 해당하는 urlpatterns
        ]
        ...
        ```
        
4. url 매핑까지 수행하면 로그인이 가능하며 로그인 이후에 index로 넘어간다.

## 로그아웃 구현

로그인 구현 이후 내비게이션 바에 로그인한 계정과 로그아웃 버튼이 활성화되지 않고 그대로 로그인 버튼이 활성화된 문제가 있다. 따라서 내비게이션 바 수정이 필요하다.

1. templates/navbar.html 수정
    
    ```html
    ...
    <!-- 로그아웃 버튼 표시 -->
    {% if user.is_authenticated %}
    <form action="{% url 'common:logout' %}" method="post">
        {% csrf_token %}
        <input type="submit" value="{{ user.username }} (로그아웃)">
    </form>
    <!-- 로그인 버튼 표시 -->
    {% else %}
    <form action="{% url 'common:login' %}" method="post">
        {% csrf_token %}
        <input type="submit" value="로그인">
    </form>
    {% endif %}
    ...
    ```
    
    - if문을 사용해서 user.is_authenticated(로그인 상태)이면 사용자 이름과 로그아웃 표시로 된 로그아웃 버튼을 표시
    - 로그인 상태가 아니면 로그인 버튼을 표시
2. 로그아웃 URL 매핑 - common/urls.py
    
    ```python
    ...
    urlpatterns = [
        path('login/', auth_views.LoginView.as_view(template_name='common/login.html'), name='login'),
        path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    ]
    ...
    ```
    
    - logout은 auth_views.LogoutView 클래스를 사용
3. 로그아웃 성공시 index로 이동하도록 페이지 등록 - config/settings.py
    
    ```python
    ...
    # 로그인, 로그아웃 성공 시 자동으로 이동할 URL
    LOGIN_REDIRECT_URL = '/'
    LOGOUT_REDIRECT_URL = '/'
    ```
4. 로그아웃 링크 출력 확인
    
    <img src="/images/login_page_3.png" width="50%" height="50%" title="login page 3" alt="login page 3">    
    

## 오류 수정

- 테스트 도중 bootstrap.min.css.map과 bootstrap.min.js.map을 찾을 수 없다는 오류가 발생하여 static에 두 파일을 추가
- django 5부터는 LogoutView가 POST 방식으로 동작해서 HTML 문서에 post method를 사용하도록 변경해야 한다.
    
    [장고 5 부터 LogoutView로의 요청은 POST 방식만 허용 | 파이썬 사랑방](https://pyhub.kr/recipe/Y8b3dWNOkN4D5/)
    
    - form을 사용해서 로그아웃 버튼을 만들었지만 디자인이 맘에 들지 않아서 javascript로 a태그의 메소드를 post로 바꾼 코드를 작성하려고 했으나 a태그를 사용한 csrftoken post는 보안상 문제가 있을 것 같아서 다른 방법을 찾는 중