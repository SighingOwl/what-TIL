# 검색, 정렬 기능

게시판에 작성된 글이 많아질 때 원하는 글을 일일이 찾아보는 것이 힘들어진다. 이때 검색 기능을 사용하면 원하는 내용만을 조회할 수 있게 된다. 보통 게시판에서 글을 검색할 때는 게시물의 제목과 내용, 작성자를 검색하므로 이 기능을 추가한다.

Django에서는 db.models 앱의 Q 함수를 사용해 OR 조건으로 데이터를 조회할 수 있다. 그리고 중복된 데이터 조회를 처리하기 위해 distinct 함수를 반드시 사용해야 한다. 데이터를 조회하기 위한 검색어를 받을 때는 POST 방식보다는 GET 방식 사용을 권장한다. 그 이유는 검색어를 POST로 전달하면 그에 따른 응답도 POST 형식으로 전달해야 하고 POST 방식은 동일한 POST 요청이 발생하면 중복 방지를 위해 오류를 발생시키기 때문에 새로고침이나 뒤로가기를 했을 때 오류가 발생할 수 있다.

## 검색 기능

1. 질문 목록 페이지에 검색창 추가
    
    ```html
    ...
    <div class="container my-3">
        <!-- 검색창 영역 Start -->
        <div class="row justify-content-end my-3">
            <div class="col-4">
                <div class="input-group">
                    <input type="text" class="form-control kw" value="{{ kw|default_if_none:'' }}">
                    <div class="input-group-append">
                        <button class="btn btn-outline-secondary" type="button" id="btn_search">찾기</button>
                    </div>
                </div>
            </div>
        </div>
        <!-- 검색창 영역 End -->
    ...
    ```
    
    - 테이블 오른쪽 상단에 검색창을 위치
    - Javascript에서 검색창에 입력된 값을 읽을 수 있도록 input 엘리먼트의 클래스에 kw 속성을 추가
2. 질문 목록 페이지에 form 엘리먼트 추가
    
    ```html
    <!-- 검색 폼 영역 Start -->
    <form id="searchForm" method="get" action="{% url 'index' %}">
        <input type="hidden" id="kw" name="kw" value="{{ kw|default_if_none:'' }}">
        <input type="hidden" id="page" name="page" value="{{ page }}">
    </form>
    <!-- 검색 폼 영역 End-->
    {% endblock %}
    ```
    
    - page와 keyword를 GET 방식으로 요청할 수 있도록 form 엘리먼트의 method를 get으로 설정
    - page와 kw는 이전 요청의 값을 기억해야 하므로 value 속성에 질문 목록 함수에서 전달받은 값을 대입
    - 질문 목록 URL로 form을 전송해야하므로 action의 값은 index url로 설정
3. 질문 목록 페이지의 페이징 수정
    
    ```html
    ...
    <!-- 페이징 처리 Start -->
    <ul class="pagination justify-content-center">
        <!-- 이전 페이지 -->
        ...
            <a class="page-link" data-page="{{ question_list.previous_page_number }}" href="#">이전</a>
        ...
        <!-- 페이지 리스트 -->
        ...
            <li class="page-item active" aria-current="page">
                <a class="page-link" data-page="{{ page_number }}" href="#">{{ page_number }}</a>
            </li>
            {% else %}
            <li class="page-item">
                <a class="page-link" data-page="{{ page_number }}" href="#">{{ page_number }}</a>
            </li>
        ...
        <!-- 다음 페이지 -->
        {% if question_list.has_next %}
        <li class="page-item">
            <a class="page-link" data-page="{{ question_list.next_page_number }}" href="#">다음</a>
        </li>
        ...
    <!-- 페이징 처리 End -->
    ...
    ```
    
    - 모든 페이지 링크를 href 속성에 직접 입력하는 대신에 data-page 속성으로 값을 읽을 수 있도록 변경
4. 질문 목록 페이지의 페이징과 검색을 위한 자바스크립트 코드 추가
    
    ```html
    ...
    {% block script %}
    <script type="text/javascript">
    $(document).ready(function() {
        // 페이징 처리
        $(".page-link").on('click', function() {
            $("#page").val($(this).data("page"));
            $("#searchForm").submit();
        })
    
        // 검색 처리
        $("#btn_search").on('click', function() {
            $("#kw").val($(".kw").val());
            $("#page").val(1);   // 1 페이지부터 데이터를 조회
            $("#searchForm").submit();
        })
    });
    </script>
    {% endblock %}
    ```
    
    - class 속성이 page-link인 링크를 클릭하면 data-page 속성값을 읽어 searchForm의 page 필드에 값을 넣어 form을 요청
    - 검색 버튼을 누르면 searchForm의 kw 필드에 값을 넣어 form을 요청
    - 검색 요청을 매번 새로운 요청이므로 1번 페이지부터 데이터를 조회하도록 page 필드에 항상 1을 설정하여 form을 요청
5. pybo/views/base_views.py의 index 함수 수정
    
    ```python
    ...
    from django.db.models import Q
    ...
    def index(request):
        # pybo 목록 출력
    
        # 입력 인자
        page = request.GET.get('page', '1') # 페이지
        kw = request.GET.get('kw', '')      # 검색어
    
        # 조회
            # 만약 kw에 값이 있을 때는 kw를 포함하는 데이터를 조회
        question_list = Question.objects.order_by('-create_date')
        if kw:
            question_list = question_list.filter(
                Q(subject__icontains=kw) |                  # 제목 검색
            Q(content__icontains=kw) |                      # 내용 검색
                Q(author__username__icontains=kw) |         # 질문 글쓴이 검색
                Q(answer__author__username__icontains=kw)   # 답변 글쓴이 검색
            ).distinct()
        
                                     
        # 페이징 처리
        paginator = Paginator(question_list, 10) # 페이지 당 질문 10개씩 표시
        page_obj = paginator.get_page(page)
        
        context = {'question_list' : page_obj, 'page': page, 'kw': kw}
        return render(request, 'pybo/question_list.html', context)
    ```
    
    - Q 함수에서 “’필드’__icontains=’값’”는 필드에 속한 데이터 중 값을 포함하고 있는 여부를 확인한다.
    - filter 함수에서 모델 필드에 접근하려면 “__”를 사용
6. 검색 기능 확인
    
    <img src="/images/search_sort_1.png" width="50%" height="50%" title="search and sort 1" alt="search and sort 1">        
    

## 정렬 기능

정렬 기능은 최신순, 추천순, 인기순을 기준으로 정렬할 수 있도록 한다.

| 정렬 기준 | 설명 |
| --- | --- |
| 최신순 | 최근 등록된 질문을 먼저 보여주는 방식 |
| 추천순 | 추천을 많이 받은 질문을 먼저 보여주는 방식 |
| 인기순 | 답변이 많이 등록된 질문을 먼저 보여주는 방식 |

정렬 기준에 해당하는 파라미터도 검색 기능과 동일하게 GET 방식으로 요청

1. 질문 목록 페이지에 정렬 조건 추가
    
    ```html
    ...
    <!-- 검색창, 정렬 영역 Start -->
    <div class="row justify-content-between my-3">
        <div class="col-2">
            <select class="form-control so">
                <option value="recent" {% if so == 'recent' %}selected{% endif %}>최신순</option>
                <option value="recommend" {% if so == 'recommend' %}selected{% endif %}>추천순</option>
                <option value="popular" {% if so == 'popular' %}selected{% endif %}>인기순</option>
            </select>
        </div>
    ...
    ```
    
    - div 엘리먼트를 양쪽 정렬로 변경
    - 왼쪽에는 정렬 조건, 오른쪽에는 검색 조건을 추가
    - 현재 선택된 정렬 기준을 읽을 수 있도록 select 엘리먼트의 class를 so로 지정
2. 질문 목록 페이지의 searchForm 수정
    
    ```html
    ...
    <!-- 검색, 정렬 폼 영역 Start -->
    <form id="searchForm" method="get" action="{% url 'index' %}">
        <input type="hidden" id="kw" name="kw" value="{{ kw|default_if_none:'' }}">
        <input type="hidden" id="page" name="page" value="{{ page }}">
        <input type="hidden" id="so" name="so" value="{{ so }}">
    </form>
    <!-- 검색, 정렬 폼 영역 End-->
    ...
    ```
    
    - 검색 폼을 작성한 form 엘리먼트에 정렬 기준을 입력할 수 있도록 id가 so인 input 엘리먼트를 추가
3. 질문 목록 페이지의 javascript 코드 수정
    
    ```html
    ...
    {% block script %}
    <script type="text/javascript">
    $(document).ready(function() {
        ...
    
        // 정렬 처리
        $(".so").on("change", function() {
            $("#so").val($(this).val());
            $("#page").val(1);
            $("#searchForm").submit();
        });
    });
    </script>
    {% endblock %}
    ```
    
    - class가 so인 엘리먼트의 값이 변경되면 변경된 값을 searchForm의 so 필드에 저장하여 searchForm을 요청
4. index 함수 수정
    
    ```python
    ...
    from django.db.models import Q, Count
    ...
    def index(request):
    		...
        # 입력 인자
        page = request.GET.get('page', '1')     # 페이지
        kw = request.GET.get('kw', '')          # 검색어
        so = request.GET.get('so', 'recent')    # 정렬 기준
    
        # 정렬
        if so == 'recommend':
            question_list = Question.objects.annotate(num_voter = Count('voter')).order_by('-num_voter', '-create_date')
        elif so == 'popular':
            question_list = Question.objects.annotate(num_answer = Count('answer')).order_by('-num_answer', '-create_date')
        else:    # recent
            question_list = Question.objects.order_by('-create_date')
    ...
    context = {'question_list' : page_obj, 'page': page, 'kw': kw, 'so': so}
    ...
    ```
    
    - 정렬 기준이 recommend일 때 추천 수가 큰 것부터 정렬
        - 추천 수는 Question 모델의 voter의 수를 세야하므로 annotate함수와 Count 함수를 사용해 구함
        - annotate는 모델의 기존 필드에 임시 필드를 생성하는 함수
        - 임시 필드를 사용해 order_by로 정렬이 가능하다.
            - 내림차순으로 정렬해야 하므로 정렬 기준이 되는 값에 음수를 취한다.
            - 값이 같을 경우 최신순으로 정렬하기 위해 create_date에 음수를 취해 2번째 기준으로 사용
    - 정렬 기준이 popular일 때 답변 수가 큰 것부터 정렬
        - 추천 수는 Question 모델의 answer의 수를 세야하므로 annotate함수와 Count 함수를 사용해 구함
    - 추천순, 인기순이 아닐 때는 등록일시를 기준으로 정렬한다.
        - 처음 조회할 때 동일한 조건을 사용하므로 조회에서 해당 코드를 삭제
5. 정렬 기능 확인
    - 최신순
        
        <img src="/images/search_sort_2.png" width="50%" height="50%" title="search and sort 2" alt="search and sort 2">        
        
        - 게시판에 처음 접속하거나 최신순으로 정렬했을 때 가장 최근 등록된 질문부터 표시된다.
            - 게시판에 처음 접속했을 때와 최신순으로 정렬했을 때 표시되는 페이지는 같지만 URL은 다르다.
            
            ```
            # 처음 접속했을 때 URL
            http://localhost:8000/pybo/
            
            # 최신순으로 정렬했을 때 URL
            http://localhost:8000/?kw=&page=1&so=recent
            ```
            
    - 추천순
        
        <img src="/images/search_sort_3.png" width="50%" height="50%" title="search and sort 3" alt="search and sort 3">        
        
        - 추천 수가 높은 순으로 정렬
    - 인기순
        
        <img src="/images/search_sort_4.png" width="50%" height="50%" title="search and sort 4" alt="search and sort 4">        
        
        - 답변 수가 많은 순으로 정렬