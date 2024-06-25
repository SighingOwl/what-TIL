# 마크다운

게시판 서비스에 글을 작성할 때 일반 텍스트 형식을 사용하는 것은 가독성이 떨어지고 글이 주목받게 하기 어렵게 한다. 이때 마크다운을 사용하면 글의 형태를 다양하게 바꾸며 가독성을 높인 글을 작성이 가능해진다.

## 마크다운 적용

마크다운은 python에서 기본으로 제공하는 패키지가 아니어서 pip로 설치 후 적용해야한다.

1. markdown 설치
    
    ```bash
    pip install markdown
    ```
    
    - 반드시 django 프로젝트를 진행하는 가상 환경에 진입한 후 설치한다.
2. 템플릿 필터에 마크다운 필터 등록
    
    ```python
    import markdown
    from django.utils.safestring import mark_safe
    ...
    @register.filter
    def mark(value):
        extensions = ["nl2br", "fenced_code"]
        return mark_safe(markdown.markdown(value, extensions=extensions))
    ```
    
    - Markdown으로 작성한 문서를 HTML 문서로 변환하기 위해 마크다운 필터를 템플릿 필터로 등록해야 한다.
    - mark 함수는 markdown 모듈과 mark_safe 함수를 사용해 문자열을 HTML 코드로 변환
    - markdown 모듈에 nl2br, fenced_code 확장 사용
        - nl2br은 줄바꿈 문자를 <br> 태그로 변경한다.
        - fenced_code는 코드블록을 사용하기 위해 적용. 마크다운에서 소스코드를 표현하기위해 백틱(```)기호를 사용한다.
3. 질문 상세 페이지에 마크다운 적용
    
    ```html
    ...
    {% pybo_filter %}
    ...
    <!--질문 내용 영역-->
    <div class="col-11">
        <div class="card">
            <div class="card-body">
                <div class="card-text">
                    {{  question.content|mark }}
                </div>
    ...
    <div class="col-11">
    		<div class="card my-3">
    		    <div class="card-body">
    		        <div class="card-text">
    		            {{ answer.content|mark }}
    ```
    
    - 원래 card-text에는 style 속성으로 white-space: pre-line;이 적용되어 있었지만 이를 제거하고 질문과 답변 내용에 마크다운 템플릿 필터를 적용
4. 마크다운 테스트
    
    ```markdown
    # 마크다운 테스트
    * 이 문서는 마크다운 테스트를 위한 문서입니다.
    * 리스트 1
    * 리스트 2
    * 리스트 3
    ```
    print('Hello World!')
    ```
    ```
    
    <img src="/images/markdown.png" width="70%" height="70%" title="markdown" alt="markdown">   