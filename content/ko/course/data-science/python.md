---
title: 답변 페이징과 정렬
date: '2023-11-01'
type: book
weight: 5
---

## 답변 목록 API
### API 명세

답변 페이징을 하기 위해서 질문 목록 API처럼 API를 별도로 만들 필요가 있다. 다음과 같이 API 명세를 작성해보자.

`[답변 목록 API 명세]`
  
| API명 | URL | 요청 방법 | 설명 |
|---|---|---|---|
| 답변 목록 | `/api/answer/list/` | post | 질문(question_id)에 대한 답변 목록을 보여준다. |


`[답변 목록 API 입력 항목]`
- question_id - 보여줄 답변의 질문
- page - 답변 목록의 페이지
- size - 한 페이지에 보여줄 목록 크기

`[답변 목록 API 출력 항목]`
- total - 답변 목록의 전체 수
- answer_list - 답변 목록

### 스키마

질문 목록을 참고하여 작성한다.
```python{hl_lines=["3-5"]}
(... 생략 ...)
   
class AnswerList(BaseModel):
    total: int = 0
    answer_list: list[Answer]= []
```

### CRUD
질문 목록과 다른 것은 question_id 값을 필요로 하는 부분이다. filter기능을 사용해서 한 질문에 해당하는 목록만 리턴하게 하였다.

```python{hl_lines=["3-10"]}
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

### 라우터
생성 API와 유사하게 question_id가 잘못됐다면 에러를 발생하게 해야 한다. 다른 부분은 질문 목록 API와 같다.

```python{hl_lines=["3-"]}
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

### API 확인
API가 완성되었다면 바로 프론트엔드를 작성하기전에 API가 잘 작동하는지 봐야한다.  FastAPI의 doc페이지에서 다음과 같이 답변 목옥을 잘 출력해준다.


## 임시 답변 데이터 생성하기

또 한가지 해야할 것이 페이징을 적용하기 위해 임시로 테스트 질문을 생성해보자. 

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
...     language = random.choice(programming_languages)
...     a = Answer(question=q, content=f"{i}번째 추천하는 프로그래밍 언어는 {language}입니다.", create_date=datetime.now())
...     db.add(a)
... db.commit()
>>>
```

## 질문 상세 화면 변경하기

이제 PageDetail.vue파일을 다음과 같이 수정하자.
```
<template>
(... 생략 ...)

    <!-- 답변 목록 -->
    <h5 class="border-bottom my-3 py-2">{{total}}개의 답변이 있습니다.</h5>
    <div v-for="answer in answerList" :key="answer.id" class="card my-3">



(... 생략 ...)

    <!-- 페이징처리 시작 -->
    <ul class="pagination justify-content-center">
      <li class="page-item" :class="{ disabled: page <= 0 }">
        <button class="page-link" @click="getAnswerList(page - 1)">이전</button>
      </li>
      <template v-for="(value, index) in Array.from({ length: totalPage })" :key="index">
        <li class="page-item" v-if="index >= page - 5 && index <= page + 5" :class="{ active: index === page }">
          <button class="page-link" @click="getAnswerList(index)">{{ index + 1 }}</button>
        </li>
      </template>
      <li class="page-item" :class="{ disabled: page >= totalPage - 1 }">
        <button class="page-link" @click="getAnswerList(page+1)">다음</button>
      </li>
    </ul>
    <!-- 페이징처리 끝 -->

    <!-- 답변 등록 -->
    <form @submit.prevent="postAnswer" class="my-3">
      <div class="mb-3">
        <textarea rows="10" v-model="content" 
               class="form-control"
               :class="{ 'disabled': !is_login }"></textarea>
      </div>
      <input type="submit" value="답변등록" class="btn btn-primary" :class="{ 'disabled': !is_login }">
    </form>
</div>
  
</template>

<script>
(... 생략 ...)

export default {
(... 생략 ...)
  data() {
    return {
      question: { answers: [], voter:[], content:''  },
      content: "",
      answerList: [],
      size: 5,
      total: 0,
      page: 0
    };
  },
  computed: {
(... 생략 ...)
      totalPage() {
        return Math.ceil(this.total / this.size);
      },
  },
  methods: {
(... 생략 ...)
    getAnswerList(_page) {
      let url = "/api/answer/list"
      let params = { 
          question_id: this.question_id,
          page: _page,
          size: this.size}
      fastapi('get', url, params, (json) => {
        this.answerList = json.answer_list;
        this.page = _page,
        this.total = json.total;
      });
    },
  },  
  created() {
    this.getQuestion();
    this.getAnswerList(0);
  }
}
</script>
```

변경점이 많은데 하나씩 보면 질문 목록에서 했던 것과 유사한 것을 알 수 있다. getAnswerList란 함수를 사용하기 위해 page, total, size등의 변수를 만들고 페이징을 질문 목록(PageHome)과 유사하게 템플릿을 작성하였다. 

## 정렬 방법
여기서부터 질문 목록 API에서 하지 않았던 것이 나온다. 추천순 정렬과 시간순 정렬등의 기능을 추가하는 것이다. 시간순 정렬은 Answer 모델에 create_time을 저장하고 있기 때문에 하는 방법이 간단하다. 하지만 추천순 정렬은 추천수를 저장하고 있지 않다. subquery를 사용하는 방법도 있을 것 같은데 일단 voter_count란 속성을 추가하는 방식으로 구현하려고 한다. models.py를 다음과 같이 voter_count속성을 추가한다.

```python
class Answer(Base):
(... 생략 ...)
    voter = relationship('User', secondary=answer_voter, backref='answer_voters')
    voter_count = Column(Integer, default=0)
```
모델이 변경되었으므로 DB 업데이트를 해야한다. 다음 두 명령을 파이썬 가상환경에서 수행한다.
```bash
alembic revision --autogenerate
alembic upgrade head
```

속성을 추가하였지만 기존에 있었던 추천이 voter_count에 반영되어있지 않으므로 수동으로 추가하여 준다.

```python
>>> from models import Question, Answer
>>> from datetime import datetime
>>> from database import SessionLocal
>>> db = SessionLocal()
>>> answers = db.query(Answer).all()
>>> for answer in answers:
...     answer.voter_count = len(answer.voter)
...
>>> db.commit()
```

### 스키마
```python
class Answer(BaseModel):
    id: int
    content: str
    create_date: datetime.datetime
    user: Optional[User] = None
    question_id: int
    modify_date: Optional[datetime.datetime] = None
    voter: list[User] = []
    voter_count: int = 0
```

### CRUD
```python
def get_answer_list(db: Session, question_id: int,
                    skip: int = 0, limit: int = 10,
                    sort_by: str = 'create_date',
                    desc: bool = True,
                    ):
    sort_column = getattr(Answer, sort_by)
    if desc:
        sort_column = sort_column.desc()
    else:
        sort_column = sort_column.asc()
    _answer_list = db.query(Answer).filter(Answer.question_id == question_id)\
            .order_by(sort_column)
    total = _answer_list.count()
    answer_list = _answer_list.offset(skip).limit(limit).all()
    return total, answer_list
```

### 라우터
```python{linenos=table,hl_lines=[8,"15-17"]}
@router.get("/list", response_model=answer_schema.AnswerList)
def answer_list(question_id: int,
                db: Session = Depends(get_db),
                sort_by: str = 'create_date',
                desc: bool = True,
                page: int = 0, size: int = 10):
    question = question_crud.get_question(db, question_id=question_id)
    if not question:
        raise HTTPException(status_code=404, detail="Question not found")
    total, _answer_list = answer_crud.get_answer_list(
                          db, question_id = question_id,
                          sort_by=sort_by, desc=desc,
                          skip=page*size, limit=size)
    return {
            'total': total,
            'answer_list': _answer_list
            }
```
