# 작성자 표시

게시판 게시물에 작성자를 표시하는 것을 시도

## 질문 목록 페이지에 작성자 표시

1. template/pybo/question_list.html 수정
    
    ```html
    ...
    <thead class="text-center table-dark">
            <tr>
                <th>번호</th>
                <th style="width:50%">제목</th>
                <th>작성자</th>
                <th>작성일시</th>
    ...
    ```
    
    - 테이블의 헤더에 작성자 항목을 추가
    
    ```html
    ...
    </td>
    <td>{{ question.author.username }}</td>
    <td>{{ question.create_date }}</td>
    ...
    ```
    
    - 테이블 속성 값에 작성자 추가
    - 작성자 표시 확인
        
        <img src="/images/show_author_1.png" width="50%" height="50%" title="show author 1" alt="show auther 1">        
        

## 질문 상세 페이지에 작성자 표시

1. template/pybo/question_detail.html 수정
    - 질문 영역에 작성날짜와 작성자 표시
        
        ```html
        <div class="badge bg-light text-dark text-start p-2">
        	<div class="mb-2">{{ question.author.username }}</div>
        	<div>{{ question.create_date }}</div>
        ```
        
    - 답변 영역에 작성날짜와 작성자 표시
        
        ```html
        <div class="badge bg-light text-dark  text-start p-2">
        	<div class="mb-2">{{ answer.author.username }}</div>
        	<div>{{ answer.create_date }}</div>
        ```
        
    - 작성자 표시 확인
        <img src="/images/show_author_2.png" width="50%" height="50%" title="show author 2" alt="show auther 2">        