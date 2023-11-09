-
- ### API 명세
-
- 답변 페이징을 하기 위해서 질문 목록 API처럼 API를 별도로 만들 필요가 있다.  다음과 같이 API 명세를 작성해보자.
-
- `[답변 목록 API 명세]`
  
  | API명 | URL | 요청 방법 | 설명 |
  |---|---|---|---|
  | 답변 목록 | `/api/answer/list/` | post | 질문(question_id)에 대한 답변 목록을 보여준다. |
- `[답변 목록 API 입력 항목]`
	- question_id - 보여줄 답변의 질문
	- page - 답변 목록의 페이지
	- size - 한 페이지에 보여줄 목록 크기
- `[답변 목록 API 출력 항목]`
	- total - 답변 목록의 전체 수
	- answer_list - 답변 목록
-
- ### 스키마
- 질문 목록을 최대한 참고하면서 만들어보자.
-
  ```python
  (... 생략 ...)
  
  
  class AnswerList(BaseModel):
      total: int = 0
      answer_list: list[Answer]= []
  ```
- ### CRUD
- 질문 목록과 다른 것은 question_id 값을 필요로 하는 부분이다. filter기능을 사용해서 한 질문에 해당하는 목록만 리턴하게 하였다.
-
-
  ```python
  (... 생략 ...)
  
  def get_answer_list(db: Session, question_id: int,
                      skip: int = 0, limit: int = 10,
                      ):
      _answer_list = db.query(Answer).filter(Answer.question_id == question_id)\
              .order_by(Answer.create_date.desc())
      total = _answer_list.count()
      answer_list = _answer_list.offset(skip).limit(limit).all()
      return total, answer_list
  ```
- ### 라우터
- 생성 API와 유사하게 question_id가 잘못됐다면 에러를 발생하게 해야 한다. 다른 부분은 질문 목록 API와 유사하다.
-
  ```python
  (... 생략 ...)
  
  @router.get("/list", response_model=answer_schema.AnswerList)
  def answer_list(question_id:,
                                 db: Session = Depends(get_db),
                                  page: int = 0, size: int = 10):
      question = question_crud.get_question(db, question_id=question_id)
      if not question:
          raise HTTPException(status_code=404, detail="Question not found")
      total, _answer_list = answer_crud.get_answer_list(
                            db, question_id = question_id,
                            skip=page*size, limit=size)
      return {
              'total': total,
              'answer_list': _answer_list
              }
  ```
-
-
- API가 완성되었다면 바로 프론트엔드를 작성하기전에 API가 잘 작동하는지 봐야한다.  FastAPI의 doc페이지에서 다음과 같이 답변 목옥을 잘 출력  해준다.
-
-
-
- 또 한가지 해야할 것이 페이징을 적용하기 위해 임시로 테스트 질문을 생성해보자.
-
  ```
  >>> from models import Question, Answer
  >>> from datetime import datetime
  >>> from database import SessionLocal
  >>> db = SessionLocal()
  >>> q = Question(subject='프로그래밍 언어 추천', content='어떤 프로그래밍 언어를 배우는 것이 좋을까요?', create_date=datetime.now())
  >>> db.add(q)
  >>> db.commit()
  >>> q = db.query(Question).get(308)
  >>> import random
  >>> programming_languages = ['Python', 'Java', 'C++', 'C#', 'JavaScript']
  >>> for i in range(1, 101):
  >>> for i in range(1, 101):
  ...     language = random.choice(programming_languages)
  ...     a = Answer(question=q, content=f"{i}번째 추천하는 프로그래밍 언어는 {language}입니다.", create_date=datetime.now())
  ...     db.add(a)
  ...     db.commit()
  >>>
  ```
-
-
  ```
   answers = db.query(Answer).all()
  >>> for answer in answers:
  ...     answer.voter_count = len(answer.voter)
  ...
  >>> db.commit()
  ```
