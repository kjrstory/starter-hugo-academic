---
title: "클라우드 아키텍쳐 그림 그리기 도구로 D2와 Diagrams 비교"
date: 2023-10-31T11:52:56+09:00
draft: false
featured: false
authors:
  - admin
tags:
 - d2
 - cloud
categories:
 - Cloud
image:  
  filename: example3_2.png
  focal_point: Smart
  preview_only: false
  caption: "Cloud Architecture using D2" 
---


## 개요
클라우드의 솔루션 아키텍트는 아키텍처 다이어그램을 자주 그리게 됩니다.
다이어그램을 그리는 도구는 여러가지가 있는데 [Draw.io](https://www.drawio.com)같은 서비스가 대표적입니다.
이런 도구는 GUI로 다이어그램을 그리게 되는데 의외로 많은 시간이 소모됩니다.
또한 아키텍트마다 아키텍처를 그리는 패턴도 달라서 같은 아키텍처도 서로 다른 그림을 그리게 됩니다.
최근에는 이런 점을 개선하고자 Diagram as Code 개념이 생겼습니다.
사실은 오래전부터 있었던 개념이나 일반적인 다이어그램을 그리는데 중심이 되어있었고 클라우드 아키텍처를 그리는데 최적화 된 것은 아니었습니다.
그래서 클라우드 아키텍처를 그리는데 초점이 맞쳐진 코드도 있습니다.
바로 한국인 개발자가 개발한 [Diagrams](https://diagrams.mingrammer.com)코드 입니다. 
이 코드는 오픈소스이고 나름 명성을 가지고 있습니다.
~~하지만 주 개발자가 바쁜지 한동안 업데이트가 안되고 있습니다.
물론 Fork를 해서 이어 개발할수도 있겠지만 공식적으로 아카이빙이 되지 않는 한 포크한 프로젝트도 유지보수가 안 될 가능성이 있습니다.~~
*(글을 작성하는 중에 오랜만에 업데이트가 됐습니다)* 
그래서 대안을 살펴보던 중 D2란 코드를 발견했습니다.
[D2](https://d2lang.com)는 더 많은 사용자를 확보했고 golang기반입니다.
그러나 클라우드 아키텍처에 쓰는 데는 아직까지 불편함이 있습니다. 
그래서 D2로 Diagrams 예제를 똑같이 그려보면서 클라우드 아키텍처 그림 도구로 적합한지 알아보려고 합니다.


## 첫번째 예제
Diagrams의 [Quick Start](https://diagrams.mingrammer.com/docs/getting-started/installation#quick-start)에는 아래 예제가 있습니다.

```python
from diagrams import Diagram
from diagrams.aws.compute import EC2
from diagrams.aws.database import RDS
from diagrams.aws.network import ELB
  
with Diagram("", show=False):
    ELB("lb") >> EC2("web") >> RDS("userdb") 
```

{{<figure src="web_service_diagram.png" caption="web service diagram(from Diagrams)" width="75%" >}}

매우 단순한 코드이나 여기서도 하나 눈여겨볼 것은 \>\>로  인스턴스간의 관계를 나타낸것입니다.
이는 연산자 오버로딩이란 기능으로 인스턴스 객체끼리 정의되어 있는 기존 연산자 기능을 바꾸어 정의하는 것입니다.
파이썬에서 \>\>는 rshift란 연산자입니다.

이 아키텍쳐를 D2에서 그릴려면 아래와 같이 작성하면 됩니다.
```
direction: right
  
lb: "lb" {
  shape: image
  icon: https://icon.icepanel.io/AWS/svg/Networking-Content-Delivery/Elastic-Load-Balancing.svg
  width: 100
  height: 100
}
  
ec2: "web" {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
  width: 100
  height: 100
}
  
db: "db" {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-RDS.svg
  width: 100
  height: 100
}
  
lb -> ec2 -> db 
  
```

{{<figure src="example1.png" caption="web service diagram using D2" width="75%" >}}

간단한 그림이므로 거의 유사하게 만들수 있습니다. 
\>\> 연산자가 -> 으로 변경되었습니다.
다만 Diagrams에서는 각 항목이 인스턴스객체로 쉽게 불러올 수 있으나 D2에서는 직접 그림파일을 이용하여 shape로 지정해야되는 불편함이 있습니다.
D2에 Class기능과 Import기능이 있습니다.
이것들을 이용하여 아래와 같이 바꿔보겠습니다.

먼저 Class역할을 해줄 파일을 만듭니다. 이 파일은 models.d2로 이름 붙이겠습니다.
```
classes:{
lb: {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FNetworking%20&%20Content%20Delivery%2FElastic-Load-Balancing.svg
  width: 100
  height: 100
}

ec2: {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
  width: 100
  height: 100
}

db: {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-RDS.svg
  width: 100
  height: 100
}
}
```
 
그리고 이렇게 models파일을 import해서 작성하면 똑같은 결과를 보여줍니다.
```
direction: right

...@models.d2

lb.class: lb
ec2.class: ec2
ec2.label: web
db.class: db
db.label: events

lb -> ec2 -> db
```

## 두번째 예제
두번째 예제는 Diagrams에 있는 Examples중 첫번째 예제입니다.

[링크](https://diagrams.mingrammer.com/docs/getting-started/examples#grouped-workers-on-aws)

Grouped Workers on AWS
```python
  from diagrams import Diagram
  from diagrams.aws.compute import EC2
  from diagrams.aws.database import RDS
  from diagrams.aws.network import ELB
  
  with Diagram("Grouped Workers", show=False, direction="TB"):
      ELB("lb") >> [EC2("worker1"),
                    EC2("worker2"),
                    EC2("worker3"),
                    EC2("worker4"),
                    EC2("worker5")] >> RDS("events")
```
{{<figure src="grouped_workers_diagram.png" caption="grouped_workers_diagram(from Diagrams)" width="75%" >}}

각 노드가 파이썬의 인스턴스이고 이 인스턴스들을 리스트로 만들어 한번에 연결 명령을 줄 수 있습니다.
이것을 D2에서는 어떻게 할 수 있을까요? 아쉽게도 D2에는 그룹기능은 없습니다. 비슷한 개념으로 Container라는것이 있지만 그룹과는 조금 다른 개념이므로 다음 예제에서 설명하겠습니다.
아래처럼 모든 노드에 대해 각각 연결 관계를 작성해야 합니다.

```
direction: down

...@models2.d2

lb.class: lb
worker1.class: ec2
worker2.class: ec2
worker3.class: ec2
worker4.class: ec2
worker5.class: ec2
db.class: db
db.label: events

lb -> worker1 -> db
lb -> worker2 -> db
lb -> worker3 -> db
lb -> worker4 -> db
lb -> worker5 -> db

```

여기서 하나 더 설명할 기능은 D2의 [Layout](https://d2lang.com/tour/layouts)이란 기능입니다. Layout은 다이어그램을 어떻게 배치할지를 결정하는 엔진입니다. D2에서는 Dagre, ELK, TALA 이렇게 3개의 Layout엔진을 지원합니다. Layout이란 개념이 그래도 잘 모르겠다면 직접 그려보면서 눈으로 확인해보십다.
D2에서 그림을 추출하는 명령은 아래와 같습니다. 
```bash
d2 -s -t 302 -l dagre example.d2 example2new_dagre.png
```
여기서 -s는 스케치 버전으로 그리겠다는 것이고 -t 302는 테마를 지정하는것입니다. 
스케치 버전과 테마는 공식 문서를 참고하시기 바랍니다.
-l 이 layout을 지정하는것으로 darge를 지정한것입니다. 
그리고 d2파일, 출력파일 순으로 작성하면 됩니다.

먼저 [Dagre](https://d2lang.com/tour/dagre)버전으로 그려보았습니다.

{{<figure src="example2new_dagre.png" caption="grouped_workers_diagram: Dagre Layout" width="75%" >}}

[ELK](https://d2lang.com/tour/elk)버전으로 그려보겠습니다.

{{<figure src="example2new_elk.png" caption="grouped_workers_diagram: ELK Layout" width="75%" >}}

앞의 두 레이아웃은 무료로 사용할 수 있지만 [TALA](https://d2lang.com/tour/tala)는 유료로 사용해야 합니다. 단 평가 목적일 때는 무료로 사용 가능하다고 하니 본 블로그 글 목적으로는 사용해도 무방하리라 판단하였습니다.

{{<figure src="example2new_tala.png" caption="grouped_workers_diagram: Tala Layout" width="75%" >}}

세 레이아웃 중 어떤 것이 가장 좋은가요? 이것은 개인의 호불호에 따라 달라질 것 같습니다. 객관적인 평가는 하지 않고 이 예제에서는 제 주관적으로 무료로 사용 가능한 ELK방식이 나은듯 합니다.


## 세번째 예제

세번째 예제는 Diagrams에 있는 Examples중 두번째 예제입니다.

[링크](https://diagrams.mingrammer.com/docs/getting-started/examples#clustered-web-services)


```python
from diagrams import Cluster, Diagram
from diagrams.aws.compute import ECS
from diagrams.aws.database import ElastiCache, RDS
from diagrams.aws.network import ELB
from diagrams.aws.network import Route53

with Diagram("Clustered Web Services", show=False):
    dns = Route53("dns")
    lb = ELB("lb")

    with Cluster("Services"):
        svc_group = [ECS("web1"),
                     ECS("web2"),
                     ECS("web3")]

    with Cluster("DB Cluster"):
        db_primary = RDS("userdb")
        db_primary - [RDS("userdb ro")]

    memcached = ElastiCache("memcached")

    dns >> lb >> svc_group
    svc_group >> db_primary
    svc_group >> memcached
```
{{<figure src="clustered_web_services_diagram.png" caption="Clustered Web Services(from Diagrams)" width="75%" >}}

Diagrams의 Cluster 함수는 여러 노드를 그룹핑해서 하나의 상자안에 그려주는 함수입니다.
이걸 D2로도 그려보겠습니다.
이번엔 그림부터 보여드리겠습니다.

{{<figure src="example3_1.png" caption="Clustered Web Services(D2 Case 1)" width="80%" >}}

이렇게 하면 유사하게 그린 것인데요. 화살표가 너무 지저분하게 있다는 것이 걸립니다. D2에서는 하나의 상자안에 그리는 기능을 Container라고 하는데 Container와 노드를 연결할 수 있습니다. 사실 그림으로 봐야 이해가 갈겁니다.

{{<figure src="example3_2.png" caption="Clustered Web Services(D2 Case 2)" width="80%" >}}

표현하고 싶은 것에 따라 다르겠지만 저 개인적으로는 후자처럼 그리는 것이 더 깔끔해서 좋은 것 같습니다. Diagrams에서는 아직 상자(Cluster)로 연결시켜주는 기능은 없습니다.
작성한 코드는 이렇습니다.

models.d2
```
classes:{
lb: {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FNetworking%20&%20Content%20Delivery%2FElastic-Load-Balancing.svg
  width: 100
  height: 100
}

ec2: {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
  width: 100
  height: 100
}

db: {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-RDS.svg
  width: 100
  height: 100
}

dns: "dns" {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FNetworking%20&%20Content%20Delivery%2FAmazon-Route-53.svg
  width: 100
  height: 100
}

memcached : "memcached" {
  shape: image
  icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-ElastiCache.svg
  width: 100
  height: 100
}

}
```

```
direction: right

...@models.d2

dns.class: dns
lb.class: lb
container1.web1.class: ec2
container1.web2.class: ec2
container1.web3.class: ec2
container2.userdb1.class: db
container2.userdb1.label: userdb
container2.userdb2.class: db
container2.userdb2.label: userdb ro
memcached.class: memcached

container1: "Services" {
  web1
  web2
  web3
}

container2: "DB Cluster" {
  userdb1
  userdb2
}

dns -> lb
lb -> container1
container1 -> container2
container1 -> memcached

container2.userdb1 -- container2.userdb2
```

## 결론

이렇게 Diagrams와 D2를 클라우드 아키텍쳐 그리는 관점으로 비교를 해봤습니다. 사실 아직까지 두 코드 모두 Draw.io같은 그리기 도구를 직접 사용하는 것보다는 예쁘게 그리기는 어렵습니다. 
다만 Diagrams as Code의 필요성이 커진다면 두 코드 모두 유용하게 쓰일 수 있을것입니다.
두 코드 각각 개발이 더 많이 이루어져야 할 것 같은데 D2가 Terrastruct란 회사가 주도하여 개발이 되고 있는 점이 D2의 장점입니다. 
Diagrams의 장점은 파이썬 코드이기 때문에 다른 파이썬 코드와 통합하여 무언가를 추가 개발하기 좋다는 것입니다.
D2도 Golang에서 라이브러리 처럼 쓰는 기능을 일부 지원하지만 아직까지는 많은 것을 지원하지는 않고 있습니다. 
저는 두 코드 모두 미래에 어떻게 발전이 되는지 주시할 것입니다.


* 참고
  - D2 리포지토리: https://github.com/terrastruct/d2
  - D2 공식 사이트: https://d2lang.com
  - Diagrams 리포지토리: https://github.com/mingrammer/diagrams
  - Diagrams 공식 사이트: https://diagrams.mingrammer.com
