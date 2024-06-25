# 스크롤 초기화 문제

질문 상세 화면에서 답변 등록이나 수정 같은 작업 후 원래 페이지로 리다이렉션 될 때 스크롤바가 항상 상단으로 이동되는 것을 확인할 수 있다. 이 문제는 서비스를 사용할 때 답변의 개수가 많아 스크롤을 할 상황이 발생하면 불편함으로 느낄 수 있다. 따라서 해당 페이지에서 어떠한 작업 후 리다이렉션이 되더라도 기존의 위치를 유지하도록 수정을 시도한다.

가장 간단한 방법은 <a> 태그를 사용해 리다이렉션을 특정 id를 가진 엘리먼트로 이동하도록 하는 것이다.

## 답변 등록, 답변 수정 시 스크롤 초기화 문제 해결

1. 질문 상세 페이지의 답변 영역에 앵커 엘리먼트 추가
    
    ```html
    ...
    <!--답변 내용 영역-->
    <h5 class="border-bottom my-3 py-2">
        {{ question.answer_set.count }}개의 답변이 있습니다.
    </h5>
    {% for answer in question.answer_set.all %}
    <a name="answer_{{ answer.id }}"></a>   <!-- 스크롤 초기화를 방지하기 위한 앵커 엘리먼트 -->
    ...
    ```
    
    - 질문에 등록된 모든 답변에 앵커 엘리먼트가 필요하므로 for문 바로 다음에 <a>태그를 작성
    - 각 답변은 고유해야 하므로 name 속성에 답변의 id를 사용
2. pybo/views/answer_views.py에서 앵커 엘리먼트로 이동할 수 있도록 redirect 수정
    
    ```python
    ...
    from django.shortcuts import render, get_object_or_404, redirect, resolve_url
    ...
    @login_required(login_url='common:login')
    def answer_create(request, question_id):
        ...
                return redirect('{}#answer_{}'.format(resolve_url('pybo:detail', question_id=question.id), answer.id))
    ...
    @login_required(login_url='common:login')
    def answer_modify(request, answer_id):
        ...
                return redirect('{}#answer_{}'.format(resolve_url('pybo:detail', question_id=answer.question.id), answer.id))
    ...
    ```
    
3. 스크롤 초기화 문제 해결 확인
    
    ```
    http://localhost:8000/pybo/2/
    -> http://localhost:8000/pybo/2/#answer_3
    ```
    
    - 2번 질문에서 답변을 수정하고 돌아오면 URL에 답변의 id가 추가된 것과 해당 답변으로 자동 스크롤 되는 것을 확인할 수 있다.

## 댓글에 앵커 기능 추가

1. 질문 상세 페이지의 댓글 영역에 앵커 엘리멘트 추가
    
    ```html
    ...
    <!-- 질문 댓글 Start -->
    <!-- 질문에 댓글이 있을 때 질문 영역에 댓글 표시 -->
    {% if question.comment_set.count > 0 %}
    <div class="mt-3">
    {% for comment in question.comment_set.all %}
        <a name="comment_{{ comment.id }}"></a>     <!-- 댓글 수정 후 스크롤 초기화 방지 -->
    ...
    <!-- 답변 댓글 Start -->
    {% if answer.comment_set.count > 0 %}
    <div class="mt-3">
    {% for comment in answer.comment_set.all %}
    	<a name="comment_{{ comment.id }}"></a>     <!-- 댓글 수정 후 스크롤 초기화 방지 -->
    ...
    ```
    
    - 질문과 답변에 모두 댓글이 있으므로 각각의 댓글에 앵커 엘리먼트 추가
2. pybo/views/comment_views.py의 리다이렉션 수정
    
    ```python
    ...
    from django.shortcuts import render, get_object_or_404, redirect, resolve_url
    ...
    @login_required(login_url='common:login')
    def comment_create_question(request, question_id):
    				    ...
                return redirect('{}#comment_{}'.format(resolve_url('pybo:detail', question_id=question.id), comment.id))
    ...
    @login_required(login_url='common:login')
    def comment_modify_question(request, comment_id):
    				    ...
                return redirect('{}#comment_{}'.format(resolve_url('pybo:detail', question_id=comment.question.id), comment.id))
    ...
    @login_required(login_url='common:login')
    def comment_create_answer(request, answer_id):
    				    ...
                return redirect('{}#comment_{}'.format(resolve_url('pybo:detail', question_id=answer.question.id), comment.id))
    ...
    @login_required(login_url='common:login')
    def comment_modify_answer(request, comment_id):
    					  ...
                return redirect('{}#comment_{}'.format(resolve_url('pybo:detail', question_id=comment.answer.question.id), comment.id))
    ...
    ```
    
3. 댓글 등록 및 수정 후 스크롤 초기화 방지 확인
    
    ```
    http://localhost:8000/pybo/2/
    -> http://localhost:8000/pybo/2/#comment_6
    ```
    
    - 댓글을 등록하거나 수정하면 질문 상세 페이지의 URL 뒤에 댓글의 id가 추가되며 스크롤이 해당 댓글로 자동 이동하는 것을 확인 가능