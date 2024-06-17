---
title: AWS Parallel Cluster 코드에 대한 고찰
date: 2024-06-15T08:00:00.000Z
draft: false
featured: false
authors:
  - admin
tags:
  - cloud
  - hpc
  - aws
categories:
  - HPC
---

AWS의 [Parallel Cluster](https://aws.amazon.com/hpc/parallelcluster/)는 AWS의 HPC 상품군 중 하나입니다. 저는 이 제품이 다른 서비스에 비해 매우 독특하다고 생각하고 이 포스트에 그 이유에 대해서 이야기하려고 합니다. 
이 포스트는 Parallel Cluster의 사용법에 대해 설명하는 글이 아닙니다. 
기본적인 사용법은 공식 가이드나 AWS에서 제공하는 워크샵에도 충분한 자료가 있습니다. 
대신 이 포스트에서는 Parallel Cluster가  어떤한 개발 방식을 택했는지 고찰해 보는 시간을 갖도록 합시다.

AWS는 로그인하지 않아도 기본적인 서비스들의 특징을 볼 수 있지만 직접 클라우드의 자원을 활용하려면 로그인해야 하는데 이 로그인을 (매니지먼트) **콘솔**에 로그인한다고 합니다. 
콘솔에 로그인하면 **서비스**라고 부르는 다양한 상품들이 있습니다. 
AWS를 직접 써봤다면 EC2나 S3같은 너무나 유명한 상품들도 보입니다. 
이런 콘솔에서 제공하는 서비스는 대부분의 작업을 콘솔 안에서 사용 가능합니다.
EC2를 쓴다면 생성부터 세세한 설정을 모두 콘솔에서 할 수 있습니다.
요새는 웹 터미널도 제공을 해주기 때문에 Putty나 MobaXterm같은 SSH 접속 프로그램을 쓰지 않고 웹 안에서 모든 필요한 것을 하는 것도 가능합니다. 
클라우드는 이런 용어와 프로세스가 정형화 되어 있습니다.

그런데 웹은 분명 훌륭한 인터페이스이긴 하지만 반복적인 작업을 하거나 재현성이 필요할 때 좋은 인터페이스는 아닙니다.
그래서 API나 CLI, SDK같은 개발자 도구들을 만들어 냈습니다.
또 IaC라 하여 Infra as Structure란 개념도 생겨났습니다. 하시코프사의 [Terraform](https://www.terraform.io)이 가장 유명한 클라우드 IaC코드입니다. 
AWS는 자체적으로 [Cloudformation](https://aws.amazon.com/cloudformation/)이란 IaC 서비스도 갖고 있습니다. 
기존에 웹에서 했던 작업들을 다양한 인터페이스로 터미널에서 명령을 주거나 코드로 만들어내는 것이 가능해졌습니다.

만약 이 글을 읽는 사람이 AWS와 같은 클라우드를 한번이라도 써봤다면 위의 설명은 매우 지루한 설명이었을 것입니다. 
이렇게 길게 설명하는 이유가 무엇일까요? 
그건 Parallel Cluster란 서비스가 위의 설명과는 전혀 다른 서비스이기 때문입니다. 
가장 확실히 차이점을 느낄 수 있는 것은 Parallel Cluster란 서비스는 콘솔에 존재하지 않습니다! 
AWS와 여러 클라우드의 서비스들을 분석하고 비교하는 사람들은 콘솔에서 보여지는 서비스 리스트를 토대로 서비스들을 목록화하고 각 서비스들을 비교하게 되는데 여기에 Parallel Cluster는 없으므로 종종 공식적인 상품은 아니지 않나라는 생각을 갖게 됩니다. 
그래서 AWS 홈페이지내에 공식 문서가 있음에도 불구하고 많은 사람들이 기본 서비스로서 인식을 못하고 있는 듯합니다.

Paralle Cluster는 파이썬 라이브러리로 만들어진 CLI 코드이며 오픈소스로 [공개](https://github.com/aws/aws-parallelcluster)되어 있습니다.
여기서 드는 의문은 이미 AWS  CLI라고 하여서 기본 CLI프로그램이 있는데 왜 별도의 CLI 프로그램을 따로 만들었을까 입니다. 
이것을 알기 위해서는 HPC에 대한 지식이 필요합니다. 
HPC를 쉽게 설명할 때는 아주 간단하게 계산용 클러스터라고 표현하기도 하지만 실제로 그 클러스터를 만들기 위해서는 굉장히 많은 SW와 자원들이 있어야 합니다. 
이런 것들을 클라우드에서는 하나의 작은 서비스가 되기도 합니다. 
아래는 현재 Paralelle Cluster에서 사용되는 서비스들의 목록입니다.

* Amazon API Gateway
* AWS Batch
* AWS CloudFormation
* Amazon CloudWatch
* Amazon CloudWatch Events
* Amazon CloudWatch Logs
* AWS CodeBuild
* Amazon DynamoDB
* Amazon Elastic Block Store
* Amazon Elastic Compute Cloud
* Amazon Elastic Container Registry
* Amazon EFS
* Amazon FSx for Lustre
* Amazon FSx for NetApp ONTAP
* Amazon FSx for OpenZFS
* AWS Identity and Access Management
* AWS Lambda
* Amazon RDS
* Amazon Route 53
* Amazon Simple Notification Service
* Amazon Simple Storage Service
* Amazon VPC
* Elastic Fabric Adapter
* EC2 Image Builder
* NICE DCV

무려 25개의 서비스들을 이용한다고 되어 있습니다. 
물론 모든 클러스터가 항상 25개 전체를 사용하는 것은 아니지만 최소한으로 사용해도 10개 정도는 기본적으로 사용하는 것 같습니다.
클라우드에서 HPC는 단일 상품으로 만드는 것이 어려울 정도로 여러가지를 결합하는 상품입니다.
금융 상품으로 예를 들면 예금, 주식, 채권 같은 기본 상품도 있지만 그 상품들을 결합한 펀드와 ETF등도 있습니다.
일반적인 펀드와 ETF는 고객의 마음대로 투자 비율을 조정하는 것은 불가능합니다.
Parallel Cluster는 고객이 여러 상품들을 결합하여 쓰게 하면서도 자유롭게 조정도 가능한 상품입니다.

결합 상품이라 콘솔에 보여주지 않는다는 것에는 사실 반론이 있습니다. 
이런 여러가지를 결합하는 상품 중에는 [Lightsail](https://aws.amazon.com/Lightsail/)이란 상품도 있습니다. 
이 상품은 웹서비스를 편하게 사용할 수 있는 상품으로 EC2, EBS, VPC, Route53, DynamoDB, ECS등의 상품을 결합하여 기존 콘솔보다 쉽게 사용할 수 있도록 합니다. 
그런데 이 상품은 콘솔에서 제공합니다. 정확히는 콘솔에서 링크를 제공하고 그 링크는 콘솔보다 더 직관적인 인터페이스를 제공하는 웹 인터페이스를  갖고 있습니다. 또 OpenSearch나 Hadoop(EMR)같은 서비스도 클러스터로 분류되는데 이런 서비스들은 콘솔에서 제공해주는 서비스입니다. 정확한 기준은 알 수 없지만 결합의 복잡도와 목적에 의해 HPC 상품은 콘솔에 보여주지 않는 것으로 결정한 것 같습니다.
그래서 AWS는 이 상품을 콘솔에서 별도의 화면으로 제공해주지 않고 오직 CLI로 제공했었습니다. 원래는 API 나 웹 화면은 제공해주지 않았는데 이제는 제공을 해줍니다.

![](ui-image.png "Parallel Cluster UI(from AWS)")

위 화면은 Parallel Cluster UI의 화면으로 AWS 콘솔과 매우 유사한 UI를 가졌습니다. 
유사한 이유는 이 코드의 개발자가 AWS 소속이기 때문일 것입니다.
[리포지토리](https://github.com/aws/aws-parallelcluster-ui)의 컨튜리뷰터를 보면 실제로 이 리포지토리의 개발자들은 AWS에 근무하고 있고 외부에서 기여하는 경우는 아직까지 드뭅니다.
이 리포지토리는 오픈소스이지만 모두가 참여할 수 있는 시장이라기보다는 식당중에서도 주방을 투명하게 공개하여 조리 과정을 모두 볼 수 있는 식당의 느낌이 들었습니다.
Parallel Cluster와 Parallel Cluster UI의 리포지토리가 분리되어 있는데 둘 다 AWS에서만 사용 가능한 코드인데도 불구하고 공개한데는 우리의 개발과정을 모두 투명하게 보여주겠다는 것으로 받아들여집니다.

그런데 이 웹 인터페이스를 제공하는 방식이 매우 특이합니다. 
콘솔에서 제공해주는 방식이 아니라 사용자가 직접 서버리스 웹을 구현하는 형태입니다. 
이 서버리스 웹을 구현할 때의 프론트/백엔드코드와 인프라 코드들을 모두 오픈소스로 공개하고 있기 때문에 사용자는 거의 입력 몇 번과 클릭 몇 번으로 서버리스 웹을 만들 수 있습니다. 
API도 마찬가지로 AWS의 기본 API를 쓰지 않고 API Gateway및 Lambda란 자신들의 상품을 사용하여 사용자 별로 API를 만드는 형태입니다.
이쯤 되면 AWS의 고집이 느껴집니다.
Parallel Cluster를 절대로 콘솔에 보여주지 않겠다 라는 것입니다. 심지어 웹 화면을 위한 프론트엔드, 백엔드 코드를 모두 만들었음에도 직접 사용자가 그 웹 서비스를 올려서 쓰라는 것입니다.

![](pcm-architecture.png "Parallel Cluster Architecture(from AWS)")

Azure와 Google Cloud의 HPC 서비스도 마찬가지입니다.
Google Cloud는 현재 [HPC Toolkit](https://github.com/GoogleCloudPlatform/hpc-toolkit)이란 CLI 툴만 제공하고 있습니다.
Azure는 [CycleCloud](https://learn.microsoft.com/en-us/azure/cyclecloud/overview?view=cyclecloud-8)란 상품을 CLI 툴과 함께 웹을 제공해주는데 역시 콘솔이 아니라 자신들의 마켓플레이스에 있습니다.
이 마켓플레이스의 상품을 사용하면 Virtual 머신에서 웹서버를 띄어주는 형태입니다. 
Azure, Google Cloud 역시 콘솔에서 바로 사용 가능한 HPC 서비스는 없는 것입니다.
대부분의 사람들은 마켓플레이스에는 CSP 자사의 상품이 아니라 써드 파티/파트너 회사들의 솔루션 상품을 올리는 것으로 인식을 합니다.
마켓플레이스에 자사의 솔루션을 올리는 것을 이해하지 못하는 사람들도 있습니다.
Paralell Cluster도 일종의 마켓플레이스 형 상품으로 생각해야 할 것 같습니다.
클라우드에서의 HPC를 다들 고성능 CPU, GPU, 또는 인피니밴드 같은 최고급의 하드웨어만을 생각합니다.
하지만 이런 하드웨어들도 HPC의 컴퍼넌트일 뿐 그 자체로 HPC가 되지 않습니다. 
결합형 상품임을 인식하여 어떤 형태로 제공해줄 것인가(또는 사용할 것인가)부터가 클라우드 HPC를 시작하는 첫 단계라고 생각합니다.
아쉽게도 제가 만난 사람들은 이런 인식의 전환이 막혀있는 사람들이 많았습니다.
그래서 저는 이런 마켓플레이스 형 상품을 CSP에서 직접 만들고 본인들의 마켓플레이스에 올리는가로 클라우드의 성숙도를 평가할 수 있다고 생각합니다.
기회가 된다면 Parallel Cluster와 Parallel Cluster UI 코드이 기반이 되는 Cloudformation, SDK, CDK, Lambda, API Gateway, Cognito같은 코드/서비스들드도 분석하고 코드에 대해서도 뜯어보는 글을 쓰고 싶네요. 

