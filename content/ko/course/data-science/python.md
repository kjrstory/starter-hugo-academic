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
| 답변 목록 | `/api/answer/list/` | post | 질문의 답변 목록을 보여준다. |


`[답변 목록 API 입력 항목]`
- question_id - 보여줄 답변의 질문 id
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
답변 생성 API와 유사하게 question_id가 없다면 에러를 발생하게 해야 한다. 

```python{hl_lines=["3-16"]}
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
API가 완성되었다면 바로 프론트엔드를 작성하기전에 API가 잘 작동하는지 봐야한다.  FastAPI의 doc페이지에서 다음과 같이 답변 목록을 잘 출력해줌을 확인하였다.


## 임시 답변 데이터 생성하기

페이징을 적용하기 위해 임시로 테스트 질문을 생성해보자. 

```python
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
>>> db.commit()
```

## 질문 상세 화면 변경하기

이제 PageDetail.vue파일을 다음과 같이 수정하자.
```html{linenos=table, hl_lines=["5-6", "9-23","47-50","55-57","61-72",76]}
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

## 정렬 로직 추가
여기서부터 질문 목록 API에서 하지 않았던 것을 해야한다. 추천순 정렬과 시간순 정렬등의 기능을 추가하는 것이다. 시간순 정렬은 Answer 모델에 create_time을 저장하고 있기 때문에 하는 방법이 간단하다. 하지만 추천순 정렬은 추천수를 저장하고 있지 않다. subquery를 사용하는 방법도 있을 것 같지만 사용법이 쉽지 않아 일단 voter_count란 속성을 추가하는 방식으로 구현하려고 한다. models.py에 다음과 같이 voter_count 속성을 추가한다.

```python{hl_lines=["4-5"]}
(... 생략 ...)
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
스키마에 voter_count를 추가한다. 초기값은 0이다.
```python{hl_lines=["10"]}
(... 생략 ...)
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
CRUD파일에는 sort_by와 desc 인수를 추가한다. sort_by는 정렬 방법이고 Answer 모델의 속성값이 들어갈 수 있으며며 초기값은 create_date로 한다. desc는 오름차순과 내림차순을 정하는 것으로 true일때가 내림차순, false일때가 오름차순이다.

```python{hl_lines=[4, "7-13"]}
(... 생략 ...)
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

한가지 잊지 않고 추가해줄 것이 있다. 이제 답변에 추천을 하게 되면 voter_count가 증가하게 해야한다.

```python{hl_lines=[4]}
(... 생략 ...)
def vote_answer(db: Session, db_answer: Answer, db_user: User):
    db_answer.voter.append(db_user)
    db_answer.voter_count = db_answer.voter_count + 1
    db.commit()
(... 생략 ...)
```

### 라우터
라우터 파일에는 앞서 추가했던 sort_by와 create_date를 추가한다.

```python{hl_lines=["5-6", 13]}
(... 생략 ...)
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

## 스토어
여기까지 했지만 한가지 또 빠진게 있다. 바로 스토어 기능이다. 질문 목록의 페이지와 유사하게 답변 목록의 페이지, sort_by, desc들도 스토어 변수로 저장해야만 브라우저가 새로 고침을 하더라도 사용할 수 있다. 단 질문 목록에 쓰였던 변수명과 동일하게 사용하면 안되고 별도로 해야 서로 충돌되지 않을것이다. 질문 목록의 page변수와 다르게 answer_page란 변수로 만든다. 질문 목록에서도 나중에 정렬 기능이 필요할 것 같지만 그 때가서 변수명 정리를 하기로 한다.

## 스토어 파일 변경
store/index.js파일을 다음과 같이 수정한다.

```javascript{linenos=table, hl_lines=["13-15","33-41","59-67"]}
import { createStore } from 'vuex'
import persistedstate from 'vuex-persistedstate';


const store = createStore({
  plugins: [persistedstate()],
  state: {
    page: 0,
    access_token: "",
    username: "",
    is_login: false,
    keyword: "",
    answer_page: 0,
    sort_by: 'create_date',
    desc: true,
  },
  mutations: {
    setPage(state, payload) {
      state.page = payload
    },
    setAccessToken(state, token) {
      state.access_token = token
    },
    setUsername(state, username) {
      state.username = username
    },
    setIsLogin(state, is_login) {
      state.is_login = is_login
    },
    setKeyword(state, keyword) {
      state.keyword = keyword
    },
    setAnswerPage(state, answer_page) {
      state.answer_page = answer_page
    },
    setSortBy(state, sort_by) {
      state.sort_by = sort_by
    },
    setDesc(state, desc) {
      state.desc = desc
    },
  },
  actions: {
    setPage(context, payload) {
      context.commit('setPage', payload)
    },
    setAccessToken(context, payload) {
      context.commit('setAccessToken', payload)
    },
    setUsername(context, payload) {
      context.commit('setUsername', payload)
    },
    setIsLogin(context, payload) {
      context.commit('setIsLogin', payload)
    },
    setKeyword(context, payload) {
      context.commit('setKeyword', payload)
    },
    setAnswerPage(context, payload) {
      context.commit('setAnswerPage', payload)
    },
    setSortBy(context, payload) {
      context.commit('setSortBy', payload)
    },
    setDesc(context, payload) {
      context.commit('setDesc', payload)
    },
  },
  getters: {
    getPage: (state) => state.page,
  }
})

export default store
```


### Vue 파일 변경
다시 PageDetail.vue파일을 수정해야 한다. 익숙하다면 처음에 로컬 변수를 사용하여 작성하지 않고 스토어 변수를 사용하여 작성할 수 있다. 단 본 글의 목적은 교육에 있으므로 단계별로 작성하는 것을 보여주는 것이다.

```html{linenos=table, hl_lines=["4-22", "30-31","34-35","38-39",52,"62-64","77-85","89-102","104-114","117-120"]}
<template>
(... 생략 ...)
    <!-- 답변 목록 -->
    <div class="row">
     <div class="col-6">
      <h5 class="border-bottom my-3 py-2 mb-0">{{total}}개의 답변이 있습니다.</h5>
     </div>
     <div class="col-6 d-flex justify-content-end align-items-center">
      <button class="btn mr-2 btn-outline-primary" 
              :class="{ active: sort_by === 'voter_count'}"
              @click="this.$store.dispatch('setSortBy', 'voter_count');
                      this.$store.dispatch('setDesc', true)">추천순</button>
      <button class="btn btn-outline-primary" 
              :class="{ active: sort_by === 'create_date' && desc === true}"
              @click="this.$store.dispatch('setSortBy', 'create_date');
                      this.$store.dispatch('setDesc', true)">최신순</button>
      <button class="btn btn-outline-primary" 
              :class="{ active: sort_by === 'create_date' && desc === false}"
              @click="this.$store.dispatch('setSortBy', 'create_date');
                      this.$store.dispatch('setDesc', false)">오래된순</button>
     </div>
    </div>
    <div v-for="answer in answerList" :key="answer.id" class="card my-3">
        <div class="card-body">
            <div class="card-text" v-html="markContent(answer.content)"></div>
(... 생략 ...)

    <!-- 페이징처리 시작 -->
    <ul class="pagination justify-content-center">
      <li class="page-item" :class="{ disabled: answer_page <= 0 }">
        <button class="page-link" @click="this.$store.dispatch('setAnswerPage', answer_page - 1)">이전</button>
      </li>
      <template v-for="(value, index) in Array.from({ length: totalPage })" :key="index">
        <li class="page-item" v-if="index >= answer_page - 5 && index <= answer_page + 5" :class="{ active: index === answer_page }">
          <button class="page-link" @click="this.$store.dispatch('setAnswerPage', index)">{{ index + 1 }}</button>
        </li>
      </template>
      <li class="page-item" :class="{ disabled: answer_page >= totalPage - 1 }">
        <button class="page-link" @click="this.$store.dispatch('setAnswerPage', answer_page + 1)">다음</button>
      </li>
    </ul>
    <!-- 페이징처리 끝 -->

(... 생략 ...)
  
</template>

<script>
import fastapi from '../lib/api';
import moment from 'moment';
import 'moment/locale/ko';
import { parse } from 'marked';

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
    };
  },
  computed: {
      is_login() {
        return this.$store.state.is_login;
      },
      markContent() {
        return (content) => parse(content);
      },
      totalPage() {
        return Math.ceil(this.total / this.size);
      },
      answer_page() { 
          return this.$store.state.answer_page 
      },
      sort_by() { 
          return this.$store.state.sort_by 
      },
      desc() {
           return this.$store.state.desc 
      },
  },
  methods: {
(... 생략 ...)
    getAnswerList() {
      let url = "/api/answer/list"
      let params = { 
          question_id: this.question_id,
          page: this.answer_page,
          size: this.size,
          sort_by: this.sort_by,
          desc: this.desc
      }
      fastapi('get', url, params, (json) => {
        this.answerList = json.answer_list;
        this.total = json.total;
      });
    },
  },  
  watch: {
    answer_page() {
        this.getAnswerList();
    },
    sort_by() {
        this.getAnswerList();
    },
    desc() {
        this.getAnswerList();
    }
  },
  created() {
    this.getQuestion();
    this.$store.dispatch('setAnswerPage', 0)
    this.$store.dispatch('setSortBy', 'create_date');
    this.$store.dispatch('setDesc', true)
    this.getAnswerList();
  }
}
```

역시 변경 부분이 많다. 일단 정렬 방법은 추천순, 최신순, 오래된순으로 하여 버튼으로 만들고 버튼을 누르면 버튼이 강조되고 sort_by와 desc 변수가 변경되도록 하였다. 이 변수들은 스토어변수로 스토어에 저장된 값을 사용한다. watch섹션에서는 이 변수가 변경되는것을 지켜보면서 변경되면 답변 목록 함수(API)를 호출한다. created섹션에서는 각 변수의 초기값을 지정해준다. 이러면 한 질문의 상세 페이지를 보다가 다른 화면으로 가고 다시 다른 질문의 상세 페이지를 보면 초기값으로 된다. 단 한 질문 상세 페이지에서는 이전에 바꿨던 것이 스토어에 남으므로 새로 고침을 하더라도 초기값으로 바뀌지 않고 스토어에 있는 값을 사용한다. 
