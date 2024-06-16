# 페이징

게시판에 내용이 많아질 경우 한 페이지에 모든 내용을 출력하는 것은 가독성에 좋지 않다. 따라서 게시판의 내용을 일정 개수로 나누어 표시하기 위해 페이징 기능을 추가한다.

## 페이징 기능 구현

1. pybo/views.py에서 index 함수에 구현
    
    ```python
    ...
    from django.core.paginator import Paginator
    
    def index(request):
        # pybo 목록 출력
    
        # 입력 인자
        page = request.GET.get('page', '1') # 페이지
    
        # 조회
        question_list = Question.objects.order_by('-create_date')
                                     
        # 페이징 처리
        paginator = Paginator(question_list, 10) # 페이지 당 질문 10개씩 표
        page_obj = paginator.get_page(page)
        
        context = {'question_list' : page_obj}
        return render(request, 'pybo/question_list.html', context)
    ...
    ```
    
    - page를 처음 요청할 때 1번 페이지를 가져오도록 request.GET.get(’page’, ‘1’) 함수를 사용
        - pybo에 별도의 page 파라미터가 없는 URL에 기본값을 지정해기 위해 get(’page’, ‘1’)을 사용
    - Paginator는 페이징 기능을 구현하기 위한 django 클래스
2. 페이징 기능 확인
    
    <img src="/images/paging_1.png" width="50%" height="50%" title="paging 1" alt="paging 1">       
    
    - 1번 페이지에 10개의 질문이 표시됨
    - 아직 페이지 이동 기능을 추가하지 않아 이전 페이지로 이동할 수는 없다.

## 페이징 적용

질문 목록에 페이징을 적용해야 하므로 question_list.html에 페이징 기능을 적용

1. templates/pybo/question_list.html 수정
    
    ```html
    ...
    </table>
    <!-- 페이징 처리 Start -->
    <ul class="pagination justify-content-center">
        <!-- 이전 페이지 -->
        {% if question.list.has_previous %}
        <li class="page-item">
            <a class="page-link" href="?page={{ question_list.previous_page_number }}">이전</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">이전</a>
        </li>
        {% endif %}
        <!-- 페이지 리스트 -->
        {% for page_number in question_list.paginator.page_range %}
            {% if page_number == question_list.number %}
            <li class="page-item active" aria-current="page">
                <a class="page-link" href="?page={{ page_number }}">{{ page_number }}</a>
            </li>
            {% else %}
            <li class="page-item">
                <a class="page-link" href="?page={{ page_number }}">{{ page_number }}</a>
            </li>
            {% endif %}
        {% endfor %}
        <!-- 다음 페이지 -->
        {% if question_list.has_next %}
        <li class="page-item">
            <a class="page-link" href="?page={{ question_list.next_page_number }}">다음</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">다음</a>
        </li>
        {% endif %}
    </ul>
    <!-- 페이징 처리 End -->
    ...
    ```
    
    - 테이블 아래에 페이징 기능을 추가
    - {{ question_list }}가 views.py의 page_obj가 된다.
    - if문을 사용해서 이전 페이지가 있을 때 이전 링크가 활성화 된다. 다음 페이지도 동일하게 적용
    - 이전과 다음 사이에는 이동할 수 있는 페이지의 링크를 표시
2. 페이지 기능이 적용된 화면 확인
    
    <img src="/images/paging_2.png" width="50%" height="50%" title="paging 2" alt="paging 2">       
    - 페이지가 표시되긴 하지만 모든 페이지가 표시되어서 가독성이 좋지 않다.

## 페이지 표시 제한

1. templates/pybo/question_list.html 수정
    
    ```html
    ...
    <!-- 페이지 리스트 -->
    {% for page_number in question_list.paginator.page_range %}
    {% if page_number >= question_list.number|add:-4 and page_number <= question_list.number|add:4 %}
    ...
    {% endif %}
    {% endfor %}
    <!-- 다음 페이지 -->
    ...
    ```
    
    - if문을 활용해서 현재 페이지를 기준으로 이전 4페이지, 이후 4페이지만 표시하도록 수정
2. 질문 목록 페이지 확인
    
    <img src="/images/paging_3.png" width="50%" height="50%" title="paging 3" alt="paging 3">       