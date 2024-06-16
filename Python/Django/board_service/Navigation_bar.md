# 내비게이션

## 내비게이션 바 추가

내비게이션 바는 모든 페이지에서 공통적으로 보여하는 컴포넌트이다. 따라서 base.html에 내비게이션 바 영역을 추가해야한다.

1. templates/pybo/base.html에 내비게이션바 영역 추가
    
    ```html
    ...
    <body>
    <!-- 내비게이션 바 Start -->
    <nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
        <a class="navbar-brand" href="{% url 'pybo:index' %}">Pybo</a>
        <button class="navbar-toggler ml-auto" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
            <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link" href="#">로그인</a>
                </li>
            </ul>
        </div>
    </nav>
    <!-- 내비게이션 바 End -->
    ...
    ```
    
    - pybo 로고를 내비게이션 바 가장 왼쪽에 배치
    - 로그인 링크를 로고 오른쪽에 추가
    
    > 점프 투 장고에서는 bootstrap 4 버전을 사용해서 nav에서 콜랩스를 사용할 때 data-toggle, data-target을 사용했지만 나는 bootstrap 5 버전을 사용하여 data-bs-toggle, data-bs-target으로 바꿔 사용했다.
    > 
2. 내비게이션 생성 확인
    
    <img src="/images/navigation_1.png" width="50%" height="50%" title="navigation bar 1" alt="navigation bar 1">   
    
3. 햄머거 메뉴 버튼
    
    <img src="/images/navigation_2.png" width="50%" height="50%" title="navigation bar 2" alt="navigation bar 2">   
    
    - 웹 브라우저 창 크기를 일정 수준 이하로 줄이면 로그인 버튼이 사라지고 햄버거 버튼이 대신 나타나는 것을 확인할 수 있다.
    - base.html에 bootstrap 자바스크립트 파일이 포함되어 있지 않아 햄버거 버튼이 아직 실질적인 역할을 수행하지 못한다.
4. base.html에 bootstrap 자바스크립트 파일 추가
    - bootstrap.min.js 파일을 static 디렉토리에 추가
    - base.html의 head 영역에 bootstrap.min.js 파일을 추가
        
        ```html
        ...
        <!-- 기본 템플릿 안에 삽입될 내용 End -->
        <!-- Bootstrap JS -->
        <script src="{% static 'bootstrap.min.js' %}"></script>
        ...
        ```
        
        > 점프 투 장고에서는 bootstrap 4를 사용해서 jQuery 의존성이 남아있지만 bootstrap 5부터는 jQuery 의존성을 완전히 없애 jQuery를 추가하지 않아도 된다.
        [내비게이션 바](https://getbootstrap.kr/docs/5.1/components/navbar/)
5. 햄버거 메뉴 확인
    
    <img src="/images/navigation_3.png" width="50%" height="50%" title="navigation bar 3" alt="navigation bar 3">   
    

## Include 기능으로 내비게이션 바 추가

django의 기능 중 템플릿의 특정 위치에 템플릿 파일을 삽입하는 include 기능을 사용해 내비게이션 바를 base.html에 추가한다. include 기능은 여러 템플릿에서 반복되는 코드를 하나의 템플릿 파일로 관리하여 유지, 보수에 유리한 장점이 있다.

1. templates/navbar.html 생성 및 내비게이션 바 코드 작성
    
    ```html
    <!-- 내비게이션 바 -->
    <nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
        <a class="navbar-brand" href="{% url 'pybo:index' %}">Pybo</a>
        <button class="navbar-toggler ml-auto" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
            <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link" href="#">로그인</a>
                </li>
            </ul>
        </div>
    </nav>
    ```
    
    - bass.html에 작성했던 내비게이션 바 코드와 동일한 코드
2. templates/base.html에 include 적용
    
    ```html
    ...
    <body>
    {% include "navbar.html" %}
    ...
    ```
    
    - 기존 내비게이션 바 코드를 모두 삭제 후 include로 navbar.html를 포함시킨다.