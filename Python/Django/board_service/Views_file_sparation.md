# views.py 개선

게시판의 기능이 하나씩 추가되면서 views.py의 내용이 방대해져 관리가 어려졌다. 따라서 views.py를 용도에 따라 분리해야한다.

## views.py 파일 분리

1. views.py을 분리하여 관리하기 위해 pybo 앱 디렉토리에 views 디렉토리를 생성하고 분리한 views.py를 관리
    
    
    | 파일명 | 기능 | 함수 |
    | --- | --- | --- |
    | base_views.py | 기본 관리 | index, detail |
    | question_views.py | 질문 관리 | question_create, question_modify, question_delete |
    | answer_views.py | 답변 관리 | answer_create, answer_modify, answer_delete |
    | comment_views.py | 댓글 관리 | comment_create_question, comment_modify_question, comment_delete_question, comment_create_answer, comment_modify_answer, comment_delete_answer |
2. views 디렉토리에 생성한 각각의 views.py 파일에 해당하는 함수를 원본 views.py에서 복사한다.
    - base_views.py
        
        ```python
        from django.core.paginator import Paginator
        from django.shortcuts import render, get_object_or_404
        from ..models import Question
        
        def index(request):
            # pybo 목록 출력
        
            # 입력 인자
            page = request.GET.get('page', '1') # 페이지
        
            # 조회
            question_list = Question.objects.order_by('-create_date')
                                         
            # 페이징 처리
            paginator = Paginator(question_list, 10) # 페이지 당 질문 10개씩 표시
            page_obj = paginator.get_page(page)
            
            context = {'question_list' : page_obj}
            return render(request, 'pybo/question_list.html', context)
        
        def detail(request, question_id):
            # pybo 질문 내용 출력
            question = get_object_or_404(Question, pk=question_id)
            context = {'question': question}
            return render(request, 'pybo/question_detail.html', context)
        ```
        
    - question_views.py
        
        ```python
        from django.contrib import messages
        from django.contrib.auth.decorators import login_required
        from django.shortcuts import render, get_object_or_404, redirect
        from django.utils import timezone
        from ..models import Question
        from ..forms import QuestionForm
        
        @login_required(login_url='common:login')
        def question_create(request):
            # pybo 질문 등록
            if request.method == 'POST':
                form = QuestionForm(request.POST)
                if form.is_valid():
                    question = form.save(commit=False)
                    question.author = request.user
                    question.create_date = timezone.now()
                    question.save()
                    return redirect('pybo:index')
            else:
                # request.method가 GET인 경우
                form = QuestionForm()
        
            context = {'form': form}
            return render(request, 'pybo/question_form.html', {'form': form})
        
        @login_required(login_url='common:login')
        def question_modify(request, question_id):
            # pybo 질문 수정
        
            question = get_object_or_404(Question, pk=question_id)
            
            # 요청자와 작성자가 다를 때 권한 없음을 알리고 질문 수정을 요청한 질문 페이지로 돌아간다.
            if request.user != question.author:
                messages.error(request, '수정권한이 없습니다.')
                return redirect('pybo:detail', question_id=question_id)
            
            # 요청이 POST일 때
                # 폼에 입력된 값이 유효할 때 내용과 작성자, 수정일시를 모델에 저장하고 질문 상세 페이지로 돌아간다.
            if request.method == "POST":
                form = QuestionForm(request.POST, instance=question)
                if form.is_valid():
                    question = form.save(commit=False)
                    question.author = request.user
                    question.modify_date = timezone.now()
                    question.save()
                    return redirect('pybo:detail', question_id=question_id)
            # 요청이 GET 일 때 질문 작성 폽을 form 변수에 저장
            else:
                form = QuestionForm(instance=question)
            context = {'form': form}
            return render(request, 'pybo/question_form.html', context)
        
        @login_required(login_url='common:login')
        def question_delete(request, question_id):
            # pybo 질문 삭제
            
            question = get_object_or_404(Question, pk=question_id)
            if request.user != question.author:
                messages.error(request, '삭제 권한이 없습니다.')
                return redirect('pybo:detail', question_id=question_id)
            question.delete()
            return redirect('pybo:index')
        
        ```
        
    - answer_views.py
        
        ```python
        from django.contrib import messages
        from django.contrib.auth.decorators import login_required
        from django.shortcuts import render, get_object_or_404, redirect
        from django.utils import timezone
        from ..models import Question, Answer
        from ..forms import AnswerForm
        
        @login_required(login_url='common:login')
        def answer_create(request, question_id):
            # pybo 답변 등록
            question = get_object_or_404(Question, pk=question_id)
            if request.method == "POST":
                form = AnswerForm(request.POST)
                if form.is_valid():
                    answer = form.save(commit=False)
                    answer.auther = request.user
                    answer.create_date = timezone.now()
                    answer.question = question
                    answer.save()
                    return redirect('pybo:detail', question_id=question.id)
            else:
                # request.method가 GET인 경우
                form = AnswerForm()
        
            context = {'question': question, 'form':form}
            return render(request, 'pybo/question_detail.html', context)
        
        @login_required(login_url='common:login')
        def answer_modify(request, answer_id):
            # pybo 답변 수정
        
            answer = get_object_or_404(Answer, pk=answer_id)
            # 요청자와 답변 작성자가 다를 때 수정 거부
            if request.user != answer.author:
                messages.error(request, '수정 권한이 없습니다.')
                return redirect('pybo:detail', question_id=answer.question.id)
            
            # 요청 메소드가 POST일 때 입력된 수정 답변을 answer 모델에 저장
            if request.method == 'POST':
                form = AnswerForm(request.POST, instance=answer)
                if form.is_valid():
                    answer = form.save(commit=False)
                    answer.author = request.user
                    answer.modify_date = timezone.now()
                    answer.save()
                    return redirect('pybo:detail', question_id=answer.question.id)
            # 요청 메소드가 GET일 때 기존의 답변 데이터를 가져와 폼을 생성
            else:
                form = AnswerForm(instance=answer)
            context = {'answer': answer, 'form': form}
            return render(request, 'pybo/answer_form.html', context)
        
        @login_required(login_url='common:login')
        def answer_delete(request, answer_id):
            # pybo 답변 삭제
            answer = get_object_or_404(Answer, pk=answer_id)
            
            # 삭제 요청자와 답변 작성자가 다를 때 삭제 거부
            if request.user != answer.author:
                messages.error(request, '삭제 권한이 없습니다.')
            # 삭제 요청자와 답변 작성자가 같을 때 삭제
            else:
                answer.delete()
            return redirect('pybo:detail', question_id=answer.question_id)
        ```
        
    - comment_views.py
        
        ```python
        from django.contrib import messages
        from django.contrib.auth.decorators import login_required
        from django.shortcuts import render, get_object_or_404, redirect
        from django.utils import timezone
        from ..models import Question, Answer, Comment
        from ..forms import CommentForm
        
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
        
3. pybo/views 디렉토리에 __init__.py 파일을 생성
    
    ```python
    from .base_views import *
    from .question_views import *
    from .answer_views import *
    from .comment_views import *
    ```
    
    - __init__.py 파일은 해당 디렉토리가 패키지의 일부임을 알려주는 역할을 수행
    - Python 3.3 이후에는 없어도 패키지로 인식하지만 하위 버전 호환을 위해 사용하는 것이 안전하다.
    - __init__.py 파일에 초기화를 위한 코드가 포함되기도 한다.
    - __init__.py 파일로 views에 있는 모든 함수를 임포트하면 views.py 파일을 사용할 때와 동일한 방법으로 views 모듈을 사용가능하며 소스코드 수정이 필요가 없다.
4. 원본 views.py 파일 삭제 및 pybo 앱 기능 테스트

## 유지보수에 유리한 구조로 수정

이렇게 분리한 파일은 __init__.py 파일에서 모두 import 하므로 마치 view.py에서 import 하는 것과 같은 효과를 볼 수 있지만 문제는 어떤 오류가 발생하여 urls.py에서부터 오류를 추적할 때 함수가 어느 파일에 있는지 알 수 없다는 것이다. 따라서 __init__.py를 사용하는 대신 각각의 파일을 urls.py에 import 해서 디버깅에 용이하도록 수정한다.

- __init__.py 파일 삭제
- pybo/urls.py 수정
    
    ```python
    from django.urls import path
    from .views import base_views, question_views, answer_views, comment_views
    
    app_name = 'pybo'
    
    urlpatterns = [
        # base_views.py
        path('', base_views.index, name='index'),
        path('<int:question_id>/', base_views.detail, name='detail'),
    
        # question_views.py
        path('question/create/', question_views.question_create, name='question_create'),
        path('question/modify/<int:question_id>/', question_views.question_modify, name='question_modify'),
        path('question/delete/<int:question_id>/', question_views.question_delete, name='question_delete'),
    
        # answer_views.py
        path('answer/create/<int:question_id>/', answer_views.answer_create, name='answer_create'),
        path('answer/modify/<int:answer_id>/', answer_views.answer_modify, name='answer_modify'),
        path('answer/delete/<int:answer_id>/', answer_views.answer_delete, name='answer_delete'),
    
        # comment_views.py
        path('comment/create/question/<int:question_id>/', comment_views.comment_create_question, name='comment_create_question'),
        path('comment/modify/question/<int:comment_id>/', comment_views.comment_modify_question, name='comment_modify_question'),
        path('comment/delete/question/<int:comment_id>/', comment_views.comment_delete_question, name='comment_delete_question'),
        path('comment/create/answer/<int:answer_id>/', comment_views.comment_create_answer, name='comment_create_answer'),
        path('comment/modify/answer/<int:comment_id>/', comment_views.comment_modify_answer, name='comment_modify_answer'),
        path('comment/delete/answer/<int:comment_id>/', comment_views.comment_delete_answer, name='comment_delete_answer'),
    ]
    ```
    
    - 각각의 views.py 파일을 import 하고 함수의 기능별로 분류하여 모듈을 연결
- config/urls.py 수정
    
    ```python
    ...
    from pybo.views import base_views
    ...
    urlpatterns = [
        ...
        path('', base_views.index, name='index'),   # '/'에 해당되는 path
    ]
    ```
    
    - config/urls.py 파일의 index 함수에 해당하는 URL 매핑을 base_view 모듈의 index 함수를 사용하도록 매핑