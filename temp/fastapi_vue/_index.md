---
title: 점프 투 FastAPI with Vue
linkTitle: 점프 투 FastAPI with Vue
summary: 점프 투 FastAPI 책의 Svelte 코드를 Vue JS환경으로 변경 및 추가 기능 개발
date: '2023-10-24'
type: book
tags:
  - Python
---

이 코스는 개인적으로 파이썬의 FastAPI를 공부하기 위해 [점프 투 FastAPI](https://wikidocs.net/book/8531)를 읽으면서 공부했던 것을 정리한 것이다. 해당 책은 백엔드로 Svelte를 사용하였다. 다만 개인적인 사유로 Vue를 사용해야 되는 상황이었다. 그래서 기존 책의 내용을 그대로 하되 프론트엔드를 Vue로 바꾼 과정을 설명하기로 한다. 또 점프 투 FastAPI의 저자는 [3-16 도전! 저자 추천 파이보 추가 기능](https://wikidocs.net/177232)에서 독자에게 숙제를 주었다. 이 추가 기능은 FastAPI와 Vue 코드를 각각 변경해야만 한다. 추가 기능을 구현한 것도 이 코스에 담으려고 한다. 정리하면 점프 투 FastAPI의 1장~3-15절까지는 Svelte에서 Vue로 변경한 것, 3-16장은 FastAPI와 Vue로 작성한 것을 코스에 담았다. 그래서 기본적인 환경 구성과 개요는 담지 않는다. 그러므로 반드시 점프 투 FastAPI를 읽고 오길 바라며 가급적 3-15장까지는 실제 실습을 하길 바란다.


작성한 코드는 모두 깃헙(https://github.com/kjrstory/fastapi_vue)에 올려놓았다. 점프 투 FastAPI책의 리포지토리를 참고하여 각 장마다 브랜치를 따로 두었다. 점프투FastAPI의 코드외에 본 코스에서 작성한 코드는 출처만 밝힌다면 사용에 제한 사항은 없다. 다만 점프 투 FastAPI의 코드의 많은 부분이 그대로 사용되기도 하므로 그런 부분은 점프 투 FastAPI의 저자에게 질문을 하기 바란다. 

> 점프 투 FastAPI의 저자 [박응용](https://wikidocs.net/profile/info/book/3)님께 감사의 말을 전합니다.
