# 추천 기능

질문이나 답변에 다른 사람들이 추천하는 기능을 추가하는 것을 시도한다. 이를 위해 Question, Answer 모델에 추천자를 의미하는 필드가 추가되어야 한다. 그리고 자신의 글에는 추천할 수 없도록 기능을 막는 것도 필요하다.

## Question, Answer 모델 수정

게시물 추천은 하나의 글에 여러 명이 추천하거나 한 명이 여러 게시물에 추천할 수 있다. 따라서 이 필드는 다대다 관계에 해당한다. Django에는 다대다 관계를 위한 ManyToManyField 함수를 지원한다.

1. Question 모델 수정
    
    ```python
    ...
    class Question(models.Model):
        ...
        voter = models.ManyToManyField(User)
    ...
    ```
    
2. makemigration 실행
    
    ```bash
    python manage.py makemigrations
    SystemCheckError: System check identified some issues:
    
    ERRORS:
    pybo.Question.author: (fields.E304) Reverse accessor 'User.question_set' for 'pybo.Question.author' clashes with reverse accessor for 'pybo.Question.voter'.
    	HINT: Add or change a related_name argument to the definition for 'pybo.Question.author' or 'pybo.Question.voter'.
    pybo.Question.voter: (fields.E304) Reverse accessor 'User.question_set' for 'pybo.Question.voter' clashes with reverse accessor for 'pybo.Question.author'.
    	HINT: Add or change a related_name argument to the definition for 'pybo.Question.voter' or 'pybo.Question.author'.
    ```
    
    - 변경된 모델을 적용하기 위해 makemigration을 수행하면 위와 같은 오류가 발생한다.
    - Question 모델에서 author와 voter가 모두 User 모델을 참조하는데 User.question_set을 사용하여 Question 모델에 접근할 때 author와 voter 중 어느 것을 기준으로 접근해야 할지 django가 알 수 없어 발생한 오류이다.
    - HINT에서 related_name 인자를 추가하거나 변경하여 문제를 해결하라고 되어 있다.
3. Question 모델에 related_model 추가
    
    ```python
    ...
    class Question(models.Model):
        author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='author_question')
    	  ...
    	  voter = models.ManyToManyField(User, related_name='voter_question')
    ...
    ```
    
    - author, voter에 related_name을 추가
4. 같은 방법으로 Answer 모델 수정
    
    ```python
    ...
    class Answer(models.Model):
        author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='author_answer')
        ...
        voter = models.ManyToManyField(User, related_name='voter_answer')
    ...
    ```
    
5. makemigration, migrate 실행
    
    ```python
    python manage.py makemigrations
    Migrations for 'pybo':
      pybo/migrations/0006_answer_voter_question_voter_alter_answer_author_and_more.py
        - Add field voter to answer
        - Add field voter to question
        - Alter field author on answer
        - Alter field author on question
        
    python manage.py migrate
    Operations to perform:
      Apply all migrations: admin, auth, contenttypes, pybo, sessions
    Running migrations:
      Applying pybo.0006_answer_voter_question_voter_alter_answer_author_and_more... OK
    ```
    

## 질문 추천 기능 생성

1. 질문 상세 페이지 질문 추천 버튼 추가
    
    ```html
    <!--질문 내용 영역-->
        <h2 class="border-bottom py-2">{{ question.subject }}</h2>
        <div class="row my-3">
            <!-- 추천 영역 Start-->
            <div class="col-1">
                <div class="bg-light text-center p-3 border font-weight-bolder mb-1">
                    {{ question.voter.count }}
                </div>
                <a href="#" data-uri="{% url 'pybo:vote_question' question.id %}" class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
            </div>
            <!-- 추천 영역 End -->
            <div class="col-11">
                <div class="card">
                    <div class="card-body">
                        ...
                    </div>
                </div>
            </div>
        </div>
    ```
    
    - 추천 버튼과 기존 질문 내용의 너비를 천제 화면의 1/12, 11/12로 분할하기 위해서 같은 div 안에 먼저 넣고 추천 버튼의 div는 col-1 클래스를 사용, 기존 질문 영역은 col-11 클래스를 사용했다.
2. 추천 확인 창 생성
    
    지난 삭제 확인 창을 생성할 때와 같이 script 영역에 추천 확인 창을 생성하는 script를 작성
    
    ```html
    ...
    {% block script %}
    <script type="text/javascript">
    $(document).ready(function(){
        ...
        $(".recommend").on('click', function() {
            if(confirm("정말로 추천하시겠습니까?")) {
                location.href = $(this).data('uri');
            }
        });
    });
    </script>
    {% endblock %}
    ```
    
    - 추천 버튼에 class=”recommend”가 적용되어 있어 이 엘리멘트를 찾아주는 jQuery 코드 $(.recommend)를 사용
    - 확인 창에서 확인을 누르면 data-uri의 URL이 호출되도록 함
3. pyby/urls.py에 질문 추천 URL 매핑
    
    ```python
    ...
    from .views import base_views, question_views, answer_views, comment_views, vote_views
    ...
    urlpatterns = [
    		...
        # vote_views.py
        path('vote/question/<int:question_id>/', vote_views.vote_question, name='vote_question'),
    ]
    ```
    
4. pybo/views/vote_views.py에 질문 추천 함수 추가
    
    ```python
    from django.contrib import messages
    from django.contrib.auth.decorators import login_required
    from django.shortcuts import get_object_or_404, redirect
    from ..models import Question
    
    @login_required(login_url='common:login')
    def vote_question(request, question_id):
        # pybo 질문 추천
        question = get_object_or_404(Question, pk=question_id)
        
        # 추천 요청자가 작성자와 같은 경우 추천 거부
        if request.user == question.author:
            messages.error(request, '본인이 작성한 글은 추천할 수 없습니다.')
        # 추천 요청자가 작성자와 다른 경우 추천
        else:
            question.voter.add(request.user)
        return redirect('pybo:detail', question_id=question.id)
    ```
    
5. 질문 상세 페이지에 자신의 글 추천시 오류 메시지 표시 기능 추가
    
    ```python
    ...
    <div class="container my-3">
        <!-- 사용자 요류 표시 Start-->
        {% if messages %}
        <div class="alert alert-danger my-3" role="alert">
        {% for message in messages %}
            <strong>{{ message.tags }}</strong>
            <ul><li>{{ message.message }}</li></ul>
        {% endfor %}
        </div>
        {% endif %}
        <!-- 사용자 오류 표시 End -->
    ...
    ```
    
6. 질문 추천 기능 확인
    - 추천 버튼 확인
        
        <img src="/images/suggestion_1.png" width="50%" height="50%" title="post suggestion 1" alt="post suggestion 1">      
        
    - 추천 확인 창 확인
        
        <img src="/images/suggestion_2.png" width="50%" height="50%" title="post suggestion 2" alt="post suggestion 2">      
        
    - 추천 오류 메시지 확인
        
        <img src="/images/suggestion_3.png" width="50%" height="50%" title="post suggestion 3" alt="post suggestion ">      
        
        - 질문 작성자와 추천 사용자가 같은 경우 오류 메시지 표시
    - 추천 수 확인
        
        <img src="/images/suggestion_4.png" width="50%" height="50%" title="post suggestion 4" alt="post suggestion 4">      
        
        - 질문 작성자와 다른 사용자가 추천할 때 추천 수 1 증가

## 답변 추천 기능

1. 질문 상세 페이지에 답변 추천 버튼 생성
    
    ```html
    <!--답변 내용 영역-->
    <h5 class="border-bottom my-3 py-2">
        {{ question.answer_set.count }}개의 답변이 있습니다.
    </h5>
    {% for answer in question.answer_set.all %}
    <div class="row my-3">
        <!-- 추천 영역 Start -->
        <div class="col-1">
            <div class="bg-light text-center p-3 border font-weight-bolder mb-1">
                {{ answer.voter.count }}
            </div>
            <a href="#" data-uri="{% url 'pybo:vote_answer' answer.id %}" class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
        </div>
        <!-- 추천 영역 End -->
        <div class="col-11">
            <div class="card my-3">
                ...
            </div>
        </div>
    </div>
    ```
    
2. pybo/urls.py에 답변 추천 URL 매핑
    
    ```python
    ...
    urlpatterns = [
    	  ...
        # vote_views.py
        path('vote/question/<int:question_id>/', vote_views.vote_question, name='vote_question'),
        path('vote/answer/<int:answer_id>/', vote_views.vote_answer, name='vote_answer'),
    ]
    ```
    
3. pybo/views/vote_views.py에 답변 추천 함수 작성
    
    ```python
    ...
    from ..models import Question, Answer
    ...
    @login_required(login_url='common:login')
    def vote_answer(request, answer_id):
        # pybo 답변 추천
        answer = get_object_or_404(Answer, pk=answer_id)
        
        # 추천 요청자가 작성자와 같은 경우 추천 거부
        if request.user == answer.author:
            messages.error(request, '본인이 작성한 글은 추천할 수 없습니다.')
        # 추천 요청자가 작성자와 다른 경우 추천
        else:
            answer.voter.add(request.user)
        return redirect('pybo:detail', question_id=answer.question.id)
    ```
    
4. 답변 추천 기능 확인
    - 추천 버튼 확인
        
        <img src="/images/suggestion_5.png" width="50%" height="50%" title="post suggestion 5" alt="post suggestion 5">      
        
    - 추천 확인 창 확인
        
        <img src="/images/suggestion_6.png" width="50%" height="50%" title="post suggestion 6" alt="post suggestion 6">      
        
    - 추천 오류 메시지 확인
        
        <img src="/images/suggestion_7.png" width="50%" height="50%" title="post suggestion 7" alt="post suggestion 7">      
        
        - 답변 작성자와 추천 사용자가 같은 경우 오류 메시지 표시
    - 추천 수 확인
        
        <img src="/images/suggestion_8.png" width="50%" height="50%" title="post suggestion 8" alt="post suggestion 8">      
        
        - 답변 작성자와 다른 사용자가 추천할 때 추천 수 1 증가

## 질문 목록 페이지에 추천 수 표시

1. 질문 목록 페이지 수정
    
    ```python
    ...
     <thead class="text-center table-dark">
    <tr>
        ...
        <th>추천</th>
        ...
    </tr>
    </thead>
    <tbody>
    ...
    <tr class="text-center">
        ...
        <!-- 게시물 추천 수 표시 Start -->
        <td>
            {% if question.voter.all.count >= 0 %}
            <span class="badge bg-warning ps-2 py-1">
                {{ question.voter.all.count }}
            </span>
            {% endif %}
        </td>
        <!-- 게시물 추천 수 표시 End -->
    ...
    ```
    
2. 질문 목록 페이지에서 추천 확인
    
    ![suggestion_9.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/4c72d76d-4e1b-4401-8601-6e8f57d474c3/28fcd228-8d3e-4aa0-bb0a-925f433c537f/suggestion_9.png)