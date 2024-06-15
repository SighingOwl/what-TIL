# 질문 등록 기능

질문 목록 페이지에 질문 등록 버튼을 생성하여 질문을 추가할 수 있는 기능을 추가

## 질문 등록 버튼

1. templates/pybo/question_list.html에 버튼 추가
    
    ```html
    ...
        </table>
        <a href="{ url 'pybo:question_create' &}" class="btn btn-primary">
            질문 등록하기
        </a>
    </div>
    ...
    ```
    
2. pybo/urls.py에 질문 등록 버튼에 대한 url 매핑 추가
    
    ```python
    ...
    urlpatterns = [
        ...
        path('question/create/', views.question_create, name='question_create'),
    ]
    ```
    
3. pybo/views.py에 question_create 함수 추가
    
    ```python
    from .forms import QuestionForm
    ...
    def question_create(request):
        # pybo 질문 등록
        form = QuestionForm()
        return render(request, 'pybo/question_form.html', {'form': form})
    ```
    
    - QuestionForm 클래스은 질문 등록을 위한 장고 폼이다.
4. pybo/forms.py에 장고 폼 작성
    
    ```python
    from django import forms
    from pybo.models import Question
    
    class QuestionForm(forms.ModelForm):
        class Meta:
            model = Question
            fields = ['subject', 'content']
    ```
    
    - pybo에 forms.py를 새로 생성한 후 QuestionForm 클래스를 생성
    - django.forms의 ModelForm을 상속받았으므로 QuestionForm은 모델과 연결된 모델 폼이다.
        - 모델 폼은 모델 폼 객체를 저장하면 연결된 모델에 데이터를 저장 가능
        - 모델 폼은 내부 클래스로 Meta가 반드시 필요하며 모델폼이 사용할 모델과 모델의 필드를 입력
5. template/pybo/question_form.html에 장고 폼 적용
    
    ```html
    {% extends 'base.html' %}
    
    {% block content %}
    <div class="container">
        <h5 class="my-3 border-bottom pb-2">질문 등록</h5>
        <form method="post" class="post-form my-3">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="btn btn-primary">저장하기</button>
        </form>
    </div>
    {% endblock %}
    ```
    
    - {{ form.as_p }}에서 form이 question_create에서 전달한 QuestionForm 객체
    - {{ form.as_p }}는 모델 폼과 연결된 입력 항목에 값을 입력할 수 있는 HTML 코드를 자동으로 생성
6. 화면 출력 확인
    
    <img src="/images/regist_question_1.png" width="50%" height="50%" title="question registration 1" alt="queation registration 1">        
    
    - 이 상태에서 데이터를 저장하는 기능을 만들지 않아 내용 입력은 가능하지만 데이터 저장이 되지 않는다.
7. pybo/view.py의 question_create 함수를 수정하여 데이터 저장 기능 추가
    
    ```python
    ...
    def question_create(request):
        # pybo 질문 등록
        if request.method == 'POST':
            form = QuestionForm(request.POST)
            if form.is_valid():
                question = form.save(commit=False)
                question.create_date = timezone.now()
                question.save()
                return redirect('pybo:index')
        else:
            # request.method가 GET인 경우
            form = QuestionForm()
        context = {'form': form}
        return render(request, 'pybo/question_form.html', {'form': form}우
    ```
    
    - URL 요청이 POST, GET을 구분하여 처리
        - GET 요청은 질문 등록을 위한 페이지 요청
        - POST 요청은 질문 등록 페이지에 입력한 데이터를 모델에 저장하는 요청
    - POST 요청에서 form이 유효한지 검사를 form.is_vaild() 함수로 수행
    - form.save(commit=False) 함수는 데이터를 모델에 저장하기 위함 함수이다.
        - commit=False는 임시 저장을 의미한다.
        - 모델을 설계할 때 create_date도 함께 추가되어야 하므로 question에 subject와 content 데이터를 가진 객체를 저장한 후 create_date를 추가
        - create_date을 추가한 후 save 함수로 모델에 저장
8. 질문 등록 확인
    
    <img src="/images/regist_question_2.png" width="50%" height="50%" title="question registration 2" alt="queation registration 2">        
    
    - 질문의 제목과 내용을 입력해 저장
    
    <img src="/images/regist_question_3.png" width="50%" height="50%" title="question registration 3" alt="queation registration 3">        
    
    - 질문 목록에 새로운 질문이 등록된 것을 확인
9. form에 bootstrap 적용 및 항목 이름을 한글로 수정
    
    {{ form.as_p }} 태그를 사용하면 form에 bootstrap을 사용할 수 없는 문제가 생긴다. 이러한 문제는 forms.py에서 widget 속성을 추가하여 적용할 수 있다.
    
    - pybo/forms.py
        
        ```python
        ...
        class QuestionForm(forms.ModelForm):
            class Meta:
                model = Question
                fields = ['subject', 'content']
                widgets = {
                    'subject': forms.TextInput(attrs={'class':'form-control'}),
                    'content': forms.Textarea(attrs={'class':'form-control', 'rows': 10}),
                }
                labels = {
                    'subject': '제목',
                    'content': '내용',
                }
        ```
        
        - Bootstrap을 적용할 form마다 bootstrap 클래스를 설정
        - label 속성을 활용해 영어로 된 항목 이름을 한글로 변경
    - 적용 확인
        
        <img src="/images/regist_question_4.png" width="50%" height="50%" title="question registration 4" alt="queation registration 4">        
        
10. {{ % form.as_p %}} 대신 HTML 코드를 직접 작성해서 form 작성
    - pybo/form.py에서 widgets 속성 삭제 → {{ % form.as_p %}}에 적용하는 속성이라 이를 사용하지 않으면 필요없기 때문
    - templates/pybo/question_form.html 수정
        
        ```html
        {% extends 'base.html' %}
        
        {% block content %}
        <div class="container">
            <h5 class="my-3 border-bottom pb-2">질문 등록</h5>
            <form method="post" class="post-form my-3">
                {% csrf_token %}
                <!-- 오류 표시 Start -->
                {% if form.errors %}
                    <div class="alert alert-danger" role="alert">
                    {% for field in form %}
                        {% if field.errors %}
                        <strong>{{ field.label }}</strong>
                        {% endif %}
                    {% endfor %}
                    </div>
                {% endif %}
                <!-- 오류 표시 End -->
                 <div class="form-group">
                    <label for="subject">제목</label>
                    <input type="text" class="form-control" name="subject" id="subject" value="{{ form.subject.value|default_if_none:'' }}">
                 </div>
                 <div class="form-group">
                    <label for="content">내용</label>
                    <textarea class="form-control" name="content" id="content" rows="10">{{ form.content.value|default_if_none:'' }}</textarea>
                 </div>
                <button type="submit" class="btn btn-primary">저장하기</button>
            </form>
        </div>
        {% endblock %}
        ```
        
        - 오류 표시는 form.is_vaild()가 실패했을 때 오류를 표시하는 역할
            - |default_if_none:’ ’은 값이 없을 때 None이 표시되는데 이를 공백으로 표시하기 위해 사용
    - 오류 확인
        
        <img src="/images/regist_question_5.png" width="50%" height="50%" title="question registration 5" alt="queation registration 5">        
        

## 답변 등록 기능에 장고 폼 적용

1. pybo/forms.py에 AnswerForm 클래스 추가 및 pybo/views.py의 answer_create 함수 수정
    - pybo/forms.py
        
        ```python
        ...
        from pybo.models import Question, Answer
        ...
        class AnswerForm(forms.ModelForm):
            class Meta:
                model = Answer
                fields = ['content']
                labels = {
                    'content': '답변내용',
                }
        ```
        
    - pybo/views.py
        
        ```python
        ...
        from .forms import QuestionForm, AnswerForm
        ...
        def answer_create(request, question_id):
            # pybo 답변 등록
            question = get_object_or_404(Question, pk=question_id)
            if request.method == "POST":
                form = AnswerForm(request.POST)
                if form.is_valid():
                    answer = form.save(commit=False)
                    answer.create_date = timezone.now()
                    answer.question = question
                    answer.save()
                    return redirect('pybo:detail', question_id=question.id)
            else:
                # request.method가 GET인 경우
                form = AnswerForm()
        
            context = {'question': question, 'form':form}
            return render(request, 'pybo/question_detail.html', context)
        ```
        
2. 질문 상세 템플릿에 오류 영역 추가
    - templates/pybo/question_detail.py
        
        ```html
        ...
        <!--답변 입력 영역-->
            <form action="{% url 'pybo:answer_create' question.id %}" method="post" class="my-3">
                {% csrf_token %}
                <!-- 오류 표시 영역 Start -->
                {% if form.errors %}
                <div class="alert alert-danger" role="alert">
                {% for field in form %}
                    {% if field.errors %}
                    <strong>{{ field.label }}</strong>
                    {{ field.errors }}
                    {% endif %}
                {% endfor %}
                </div>
                {% endif %}
                <!-- 오류 표시 영역 End -->
        ...
        ```
        
3. 오류 확인
    
    <img src="/images/regist_question_6.png" width="50%" height="50%" title="question registration 6" alt="queation registration 6">        
    
    - 답변 내용이 비어 있을 때 필드를 채우라는 경고가 발생
    - 3번째 답변이 비어있는 것은 오류 처리 코드를 추가하기 전에 빈 답변을 등록했기 때문