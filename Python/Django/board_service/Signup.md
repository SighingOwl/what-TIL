# 회원가입

로그인 기능을 구현하였을 때 처음부터 등록되어 있던 admin 계정을 사용해서 로그인했었지만 다른 사람이 로그인하기 위해서는 사용자가 회원가입을 할 수 있는 기능이 추가되어야 한다. Django에는 django.contrib.auth 앱을 사용해서 회원가입 기능을 구현할 수 있다.

## 회원가입 구현

1. 로그인 페이지에 회원가입 링크 추가
    - templates/common/login.html 수정
        
        ```html
        <div class="container my-3">
            <!-- 회원가입 링크 -->
            <div class="row">
                <div class="col-4">
                    <h4>로그인</h4>
                </div>
                <div class="col-8 text-right">
                    <span>
                        또는 <a href="{% url 'common:signup' %}">계정을 만드세요.</a>
                    </span>
                </div>
            </div>
        ```
        
    - URL 매핑 - common/urls.py
        
        ```python
        ...
        from . import views
        ...
        urlpatterns = [
        		...
            path('signup/', views.signup, name='signup'),
        ]
        
        ```
        
2. 회원가입에 사용할 폼 생성 - common/forms.py
    
    ```python
    from django import forms
    from django.contrib.auth.forms import UserCreationForm
    from django.contrib.auth.models import User
    
    class UserForm(UserCreationForm):
        email = forms.EmailField(label='이메일')
    
        class Meta:
            model = User
            fields = ("username", "email")
    ```
    
    - django.contrib.auth.forms 패키지의 UsercreationForm에는 기본적으로 사용자 이름과 패스워드, 패스워드 확인 속성을 가지고 있다. 여기에 사용지의 email 추가한 UserForm을 생성
    - UserCreationForm의 is_vaild 함수는 사용자 이름, 비밀번호, 비밀번호 확인을 모두 입력했는지 확인하고 비밀번호와 비밀번호 확인이 동일한지, 비밀번호가 생성 규칙에 맞는지를 검사.
3. common/views.py에 signup 함수 정의
    
    ```python
    from django.contrib.auth import authenticate, login
    from django.shortcuts import render, redirect
    from common.forms import UserForm
    
    def signup(request):
        # 회원가입
        # POST 요청일 때 입력된 데이터로 사용자 생성
        if request.method == "POST":
            form = UserForm(request.POST)
            if form.is_valid():
                form.save()
                username = form.cleaned_data.get('username')
                raw_password = form.cleaned_data.get('password1')
                user = authenticate(username=username, password=raw_password)
                login(request, user)
                return redirect('index')
        # GET 요청일 때 회원가입 페이지를 표시    
        else:
            form = UserForm()
        return render(request, 'common/signup.html', {'form': form})
    ```
    
    - form.cleaned_data.get() 함수는 각 항목에 입력된 데이터를 얻기 위해 사용
4. 회원가입 템플릿 - templates/common/signup.html
    
    ```html
    {% extends 'base.html' %}
    {% block content %}
    <div class="container my-3">
        <div class="row my-3">
            <div class="col-4">
                <h4>회원가입</h4>
            </div>
            <div class="col-8 text-end">
                <span>또는 <a href="{% url 'common:login' %}">로그인 하세요.</a></span>
            </div>
        </div>
        <form method="post" class="post-form">
            {% csrf_token %}
            {% include "form_errors.html" %}
            <!-- 사용자 이름 입력 -->
            <div class="form-group">
                <label for="username">사용자 이름</label>
                <input type="text" class="form-control" name="username" id="username" value="{{ form.username.value|default_if_none:'' }}">
            </div>
            <!-- 비밀번호 입력 -->>
            <div class="form-group">
                <label for="password1">비밀번호</label>
                <input type="password" class="form-control" name="password1" id="password1" value="{{ form.password1.value|default_if_none:'' }}">
            </div>
            <!-- 비빌먼호 확인 -->
            <div class="form-group">
                <label for="password2">비밀번호 확인</label>
                <input type="password" class="form-control" name="password2" id="password2" value="{{ form.password2.value|default_if_none:'' }}">
            </div>
            <!-- 이메일 입력 -->
            <div class="form-group">
                <label for="email">이메일</label>
                <input type="text" class="form-control" name="email" id="email" value="{{ form.email.value|default_if_none:'' }}">
            </div>
            <!-- 회원가입 버튼 -->
            <button type="submit" class="btn btn-primary">생성하기</button>
        </form>
    </div>
    {% endblock %}
    ```
    
    - form에 내용이 누락되었거나 잘못되었을 때 오류를 표시하기 위해 form_errors.html을 include 함수로 포함
5. 회원가입 링크 및 회원가입 화면 확인
    
    <img src="/images/signup_page_1.png" width="50%" height="50%" title="signup page 1" alt="signup page 1">      
    
    - 로그인 페이지 오른쪽에 회원가입 링크 생성
    
    <img src="/images/signup_page_2.png" width="50%" height="50%" title="signup page 2" alt="signup page 2">      
    
    - 회원가입을 위한 사용자 정보 입력 필드와 로그인 페이지로 이동하는 링크가 표시
    
    <img src="/images/signup_page_3.png" width="50%" height="50%" title="signup page 3" alt="signup page 3">      
    
    - 필드에 값을 채우지 않고 생성하기를 클릭하면 오류 메시지 발생
    
    <img src="/images/signup_page_4.png" width="50%" height="50%" title="signup page 4" alt="signup page 4">      
    
    - django admin에 접속하면 회원가입한 사용자 정보를 확인할 수 있다.