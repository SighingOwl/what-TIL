# 템플릿 필터

템플릿 필터란 템플릿 테그에서 ‘|’ 파이프 기호 뒤에 사용하는 필터를 의미한다. 앞서 작성란이 비어있을 때 None 대신 공백 문자열을 표시하도록 사용한 default_if_none과 같은 것이 템플릿 필터이다.

[커스텀 템플릿 태그 및 필터를 만드는 방법 | Django 문서](https://docs.djangoproject.com/ko/5.0/howto/custom-template-tags/)

## 게시판 페이지를 넘어갈 때 게시물 번호가 1부터 시작하는 문제 해결

페이지를 이동하면 게시물의 번호가 무조건 1부터 시작하는 문제가 발생한다. 이를 템플릿 필터로 해결하는 것을 시도

게시물은 가장 처음 등록된 것이 1번이고 가장 마지막에 등록된 것이 가장 마지막 번호로 등록이 되어야 한다. 따라서 전체 게시물에서 현재 게시물의 인덱스를 빼주면 된다. 하지만 페이징을 통해 개시물을 나눠서 현재 인덱스만 빼면 지금과 동일하게 페이지마다 같은 숫자만 표시하게 될 것이다. 따라서 페이지마다 시작 인덱스도 함께 빼줄 것이다. 공식은 아래와 같다.

 

$$
게시물 번호 = 전체 게시물 - 시작 인덱스 - 현재 인덱스 + 1
$$

여기서 문제가 되는 점은 django에는 빼기 필터가 없어서 직접 만들어야 한다. “| add:-1”를 사용할 아이디어를 낼 수도 있지만 add 필터는 인수로 숫자만 입력할 수 있어 ‘-’ 기호를 사용할 수 없다.

1. 템플릿 필터 디렉토리 생성
    
    템플릿 필터는 템플릿 필터 파일로 정의해야 한다. 따라서 템플릿 필터 디렉토리를 생성하여 관리하는 것이 필요하다. 템플릿 필터 디렉토리는 반드시 앱 디렉토리 아래에 생성해야 한다.
    
2. pybo/templatetags/pybo_filter.py
    
    ```python
    from django import template
    
    register = template.Library()
    
    @register.filter
    def sub(value, arg):
        return value - arg
    ```
    
    - 템플릿 필터 함수를 만들 때는 @register.filter라는 데코레이터를 적용하면 템플릿에서 필터로 사용한다.
3. 템플릿 필터 사용 - templates/pybo/question_list.py 수정
    
    ```html
    {% extends 'base.html' %}
    {% load pybo_filter % }
    ...
    <tr>
    	<td>{{ qustion_list.paginator.count|sub:question_list.start_index|sub:forloop.counter0|add:1 }}</td>
    ...
    ```
    
    - 기존 게시물 번호를 표시하던 forloop.counter 대신 게시물 번호를 계산하는 공식을 템플릿 필터를 활용해 표시하도록 수정
    - 시작 인덱스는 각 페이지에서 가장 빠른 인덱스로 1페이지에서 1부터, 2페이지 11번…으로 시작한다.
4. 게시물 번호 확인
    
    <img src="/images/template_filter.png" width="50%" height="50%" title="template filter" alt="template filter">      
    
    - 게시물이 등록된 순으로 번호가 붙은 것을 확인