# 댓글 기능

질문과 답변에 추가적인 댓글을 남길 수 있도록 댓글 기능을 추가. 댓글도 내용이 저장되어야 하므로 댓글 모델을 생성하고 질문이나 답변과 같이 폼과 함수, 템플릿을 생성하는 순서로 기능을 추가

## 댓글 모델 생성

1. pybo/models.py에 comment 모델 추가
    
    ```python
    ...
    class Comment(models.Model):
        author = models.ForeignKey(User, on_delete=models.CASCADE)
        content = models.TextField()
        create_date = models.DateTimeField()
        modify_date = models.DateTimeField(null=True, blank=True)
        question = models.ForeignKey(Question, null=True, blank=True, on_delete=models.CASCADE)
        answer = models.ForeignKey(Answer, null=True, blank=True, on_delete=models.CASCADE)
    ```
    
    - 작성자, 내용, 작성일시, 수정일시 필드를 포함한다.
    - 댓글은 질문이나 답변에 모두 작성할 수 있으므로 질문과 답변 필드를 외래키로 가져온다.
        - 질문이나 답변 중 하나를 택해 작성하므로 질문 필드 혹은 답변 필드가 채워지지 않을 수 있어 null과 blank를 true로 둔다
        - 작성자 계정이 삭제되면 댓글도 함께 삭제되도록 한다.
2. 모델이 추가되었으므로 makemigrations, migrate 실행
    
    ```bash
    python manage.py makemigrations
    python manage.py migrate
    ```
    

## 질문 댓글 기능 추가

1. 질문 상세 페이지에 댓글 목록과 댓글 입력 링크 추
    
    ```html
    ...
    <div class="card-body">
    ...
    	<!-- 질문 댓글 Start -->
    	        <!-- 질문에 댓글이 있을 때 질문 영역에 댓글 표시 -->
    	        {% if question.comment_set.count > 0 %}
    	        <div class="mt-3">
    	        {% for comment in question.comment_set.all %}
    	            <div class="comment py-2 text-muted">
    	                <span style="white-space: pre-line;">{{ comment.content }}</span>
    	                <!-- 작성일시, 수정 일시 표시-->
    	                <span>
    	                    - {{ comment.author }}, {{ comment.create_date }}
    	                    {% if comment.modify_date %}
    	                    (수정:{{ comment.modify_date }})
    	                    {% endif %}
    	                </span>
    	                <!-- 수정, 삭제 링크 표시 -->
    	                {% if request.user == comment.author %}
    	                <a href="{% url 'pybo:comment_modify_question' comment.id %}" class="small">수정</a>
    	                <a href="%" class="delete small" data-uri="{% url 'pybo:comment_delete_question' comment.id %}">삭제</a>
    	                {% endif %}
    	            </div>
    	        {% endfor %}
    	        </div>
    	        {% endif %}
    	        <!-- 댓글 추가 링크 표시 -->
    	        <div>
    	            <a href="{% url 'pybo:comment_create_question' question.id %}" class="small"><small>댓글 추가 ..</small></a>
    	        </div>
    	        <!-- 질문 댓글 End -->
    ...
    ```
    
    - for 문을 활용해 질문에 등록된 모든 댓글을 표시하도록 함
    - if 문을 활용해 수정, 삭제 요청자와 댓글 작성자가 같을 때 수정, 삭제가 가능하도록 함
    - comment 클래스를 별도의 CSS로 작성해 댓글의 크기를 작게 표시되도록 한다.
2. static/style.css에 comment class CSS 작성
    
    ```css
    .comment {
        border-top:dotted 1px #ddd;
        font-size:0.7em
    }
    ```
    
3. pybo/urls.py에 질문 댓글 URL 매핑 추가
    
    ```python
    ...
    urlpatterns = [
    		...
        path('comment/create/question/<int:question_id>/', views.comment_create_question, name='comment_create_question'),
        path('comment/modify/question/<int:comment_id>/', views.comment_modify_question, name='comment_modify_question'),
        path('comment/delete/question/<int:comment_id>'/, views.comment_delete_question, name='comment_delete_question'),
    ]
    ```
    
    - 질문 등록은 특정된 질문에 댓글을 등록하므로 질문 id를 참조한다.
4. pybo/forms.py에 질문 댓글 폼 작성
    
    ```python
    ...
    class CommentForm(forms.ModelForm):
        class Meta:
            model = Comment
            fields = ['content']
            labels = {
                'content': '댓글 내용'
            }
    ```
    
5. pybo/views.py에 댓글 등록 함수 추가
    
    ```python
    ...
    from .forms import QuestionForm, AnswerForm, CommentForm
    ...
    @login_required(login_url='common:login')
    def comment_create_question(request, question_id):
        # pybo 질문 댓글 등록
        question = get_object_or_404(Question, pk=question_id)
        
        # 요청 메소드가 POST일 때 입력된 댓글 데이터를 comment 모델에 저장
        if request.method == 'POST':
            form = CommentForm(request.POST)
            if form.is_valid():
                comment = form.save(commit=False)
                comment.author = request.user
                comment.create_date = timezone.now()
                comment.question = question
                comment.save()
                return redirect('pybo:detail', question_id=question.id)
        # 요청 메소드가 GET일 때 댓글폼을 로드
        else:
            form = CommentForm()
        context = {'form': form}
        return render(request, 'pybo/comment_form.html', context)
    ```
    
6. templates/pybo/comment_form.html에 질문 댓글 등록 템플릿 작성
    
    ```python
    {% extends 'base.html' %}
    {% block content %}
    <div class="container my-3">
        <h5 class="border-bottom pb-2">댓글등록하기</h5>
        <form method="post" class="post-form my-3">
            {% csrf_token %}
            {% include "form_errors.html" %}
            <!-- 질문 입력 필드 -->
            <div class="form-group my-2">
                <label for="content">댓글내용</label>
                <textarea class="form-control my-2" name="content" id="content" rows="3">{{ form.content.value|default_if_none:'' }}</textarea>
            </div>
            <!-- 질문 등록 버튼 -->
            <button type="submit" class="btn btn-primary my-2">저장하기</button>
        </form>
    </div>
    {% endblock %}
    ```
    
7. pybo/views.py에 댓글 수정 함수 추가
    
    ```python
    ...
    from .models import Question, Answer, Comment
    ...
    @login_required(login_url='common:login')
    def comment_modify_question(request, comment_id):
        # pybo 질문 댓글 수정
        comment = get_object_or_404(Comment, pk=comment_id)
    
        # 수정 요청자와 작성자가 다르면 댓글 수정 거부
        if request.user != comment.author:
            messages.error(request, '댓글수정 권한이 없습니다.')
            return redirect('pybo:detail', question_id=comment.question.id)
        
        # 요청 메소드가 POST일 때 입력된 댓글 데이터로 기존 댓글을 변경
        if request.method == 'POST':
            form = CommentForm(request.POST, instance=comment)
            if form.is_valid():
                comment = form.save(commit=False)
                comment.author = request.user
                comment.modify_date = timezone.now()
                comment.save()
                return redirect('pybo:detail', question_id=comment.question.id)
        # 요청 메소드가 GET일 때 기존 댓글을 포함한 댓글폼을 생성
        else:
            form = CommentForm(instance=comment)
        context = {'form': form}
        return render(request, 'pybo/comment_form.html', context)
    ```
    
8. pybo/views.py에 댓글 삭제 함수 추가
    
    ```python
    @login_required(login_url='common:login')
    def comment_delete_question(request, comment_id):
        # pybo 질문 댓글 삭제
        comment = get_object_or_404(Comment, pk=comment_id)
    
        # 삭제 요청자와 작성자가 다르면 댓글 삭제 거부
        if request.user != comment.author:
            messages.error(request, '댓글삭제 권한이 없습니다.')
            return redirect('pybo:detail', question_id=comment.question.id)
        # 삭제 요청자와 작성자가 같으면 댓글 삭제
        else:
            comment.delete()
        return redirect('pybo:detail', question_id=comment.question.id)
    ```
    
9. 질문 댓글 확인
    1. 질문 작성 폼 화면
        
        <img src="/images/comment_1.png" width="50%" height="50%" title="comment 1" alt="comment 1">        
        
    2. 질문 상세에서 등록된 댓글 확인
        
        <img src="/images/comment_2.png" width="50%" height="50%" title="comment 2" alt="comment 2">        
        

## 답변 댓글 기능 추가

답변 댓글 기능은 질문 댓글 기능과 같은 절차로 추가

1. templates/pybo/question_detail.html에 답변 댓글 목록과 등록 링크 추가
    
    ```python
    ...
    <div class="card-body">
    ...
    	<!-- 답변 댓글 Start -->
    	{% if answer.comment_set.count > 0 %}
    	<div class="mt-3">
    	{% for comment in answer.comment_set.all %}
    	<div class="comment py-2 text-muted">
    	  <span style="white-space: pre-line">{{ comment.content }}</span>
    	  <span>
    	      - {{ comment.author }}, {{ comment.create_date }}
    	      {% if comment.modify_date %}
    	      (수정:{{ comment.modify_date }})
    	      {% endif %}
    	  </span>
    	  {% if request.user == comment.author %}
    	  <a href="{% url 'pybo:comment_modify_answer' comment.id}" class="small">수정</a>
    	  <a href="#" class="delete small" data-uri="{% url 'pybo:comment_delete_answer' comment.id %}">삭제</a>
    	  {% endif %}
    	</div>
    	{% endfor %}
    	</div>
    	{% endif %}
    	<div>
    	<a href="{% url 'pybo:comment_create_answer' answer.id %}" class="small"><small>댓글 추가 ...</small></a>
    	</div>
    	<!-- 답변 댓글 End -->
    ...
    ```
    
2. pybo/urls.py에 답변 댓글 URL 매핑
    
    ```python
    urlpatterns = [
    		...
        path('comment/create/answer/<int:answer_id>/', views.comment_create_answer, name='comment_create_answer'),
        path('comment/modify/answer/<int:comment_id>/', views.comment_modify_answer, name='comment_modify_answer'),
        path('comment/delete/answer/<int:comment_id>/', views.comment_delete_answer, name='comment_delete_answer').
    ]
    ```
    
3. pybo/views.py에 답변 등록, 수정, 삭제 함수 추가
    
    ```python
    ...
    @login_required(login_url='common:login')
    def comment_create_answer(request, answer_id):
        # pybo 답변 댓글 등록
        answer = get_object_or_404(Answer, pk=answer_id)
        
        # 요청 메소드가 POST일 때 입력된 댓글 데이터를 comment 모델에 저장
        if request.method == 'POST':
            form = CommentForm(request.POST)
            if form.is_valid():
                comment = form.save(commit=False)
                comment.author = request.user
                comment.create_date = timezone.now()
                comment.answer = answer
                comment.save()
                return redirect('pybo:detail', question_id=comment.answer.question.id)
        # 요청 메소드가 GET일 때 댓글폼을 로드
        else:
            form = CommentForm()
        context = {'form': form}
        return render(request, 'pybo/comment_form.html', context)
    
    @login_required(login_url='common:login')
    def comment_modify_answer(request, comment_id):
        # pybo 답변 댓글 수정
        comment = get_object_or_404(Comment, pk=comment_id)
    
        # 수정 요청자와 작성자가 다르면 댓글 수정 거부
        if request.user != comment.author:
            messages.error(request, '댓글수정 권한이 없습니다.')
            return redirect('pybo:detail', question_id=comment.answer.question.id)
        
        # 요청 메소드가 POST일 때 입력된 댓글 데이터로 기존 댓글을 변경
        if request.method == 'POST':
            form = CommentForm(request.POST, instance=comment)
            if form.is_valid():
                comment = form.save(commit=False)
                comment.author = request.user
                comment.modify_date = timezone.now()
                comment.save()
                return redirect('pybo:detail', question_id=comment.answer.question.id)
        # 요청 메소드가 GET일 때 기존 댓글을 포함한 댓글폼을 생성
        else:
            form = CommentForm(instance=comment)
        context = {'form': form}
        return render(request, 'pybo/comment_form.html', context)
    
    @login_required(login_url='common:login')
    def comment_delete_answer(request, comment_id):
        # pybo 답변 댓글 삭제
        comment = get_object_or_404(Comment, pk=comment_id)
    
        # 삭제 요청자와 작성자가 다르면 댓글 삭제 거부
        if request.user != comment.author:
            messages.error(request, '댓글삭제 권한이 없습니다.')
            return redirect('pybo:detail', question_id=comment.answer.question.id)
        # 삭제 요청자와 작성자가 같으면 댓글 삭제
        else:
            comment.delete()
        return redirect('pybo:detail', question_id=comment.answer.question.id)
    ```
    
4. 답변 댓글 확인
    - 답변 댓글 추가 링크 확인
        
        <img src="/images/comment_3.png" width="50%" height="50%" title="comment 3" alt="comment 3">        
        
    - 답변 댓글폼 확인
        
        <img src="/images/comment_4.png" width="50%" height="50%" title="comment 4" alt="comment 4">        
        
    - 답변 댓글 등록 확인
        
        <img src="/images/comment_5.png" width="50%" height="50%" title="comment 5" alt="comment 5">        