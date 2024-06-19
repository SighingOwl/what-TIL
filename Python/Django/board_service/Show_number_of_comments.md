# 게시물 답변 개수 표시

웹에서 찾을 수 있는 게시판을 살펴보면 답변이나 댓글이 등록되었을 때 게시물 제목 옆에 개수가 표시되는 것을 확인할 수 있다. 이 부분을 구현하는 것을 시도

## 질문 목록 템플릿에 기능 추가

- templates/pybo/question_list.html 수정
    
    ```html
    ...
    <td>
      <a href="{% url 'pybo:detail' question.id %}">
          {{ question.subject }}
          <!-- 게시물에 등록된 답변 개수 표시 Start -->
          {% if question.answer_set.count > 0 %}
          <span class="text-danger small ml-2">
              [{{ question.answer_set.count }}]
          </span>
          {% endif %}
          <!-- 게시물에 등록된 답변 개수 표시 End -->
    ...
    ```
    
    - if 문으로 답변의 존재 여부를 먼저 확인하고 있으면 답변의 개수를 게시물 제목 옆에 표시
- 답변 개수 표시 확인
    
    <img src="/images/show_number_of_comments.png" width="50%" height="50%" title="show number of comments" alt="show number of comments">      