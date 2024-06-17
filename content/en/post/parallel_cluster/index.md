---
title: Exploring AWS Parallel Cluster: A Unique Approach to HPC
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


AWS [Parallel Cluster](https://aws.amazon.com/hpc/parallelcluster/) is one of the services in AWS's HPC (High-Performance Computing) portfolio. I believe this product is highly unique compared to other services, and I intend to discuss the reasons in this post. This post is not a guide on how to use Parallel Cluster; for that, you can refer to the official guide or workshops provided by AWS. Instead, we will explore the development approach of Parallel Cluster.

AWS provides basic information about its services without logging in, but to utilize cloud resources directly, you need to log into the (Management) **Console**. 
Once logged in, you will see various products called **services**. 
If you have used AWS, you are likely familiar with popular services like EC2 and S3. 
Most of the tasks can be managed within the console. For instance, if you are using EC2, you can perform all configurations from creation to detailed settings within the console. 
Nowadays, web terminals are provided, so you can do everything necessary without using SSH connection programs like Putty or MobaXterm. Cloud terminology and processes are well-defined in this way.

However, while the web interface is excellent, it is not suitable for repetitive tasks or creating reproducible interfaces. This is where developer tools like APIs, CLI, and SDKs come in. Additionally, the concept of IaC (Infrastructure as Code) emerged, with HashiCorp's [Terraform](https://www.terraform.io) being the most famous IaC code for the cloud. AWS also offers its own IaC service called [Cloudformation](https://aws.amazon.com/cloudformation/). Tasks previously performed on the web can now be executed via terminal commands or code.

If you have used AWS or similar cloud services, the above explanation might seem tedious. Why am I explaining it in such detail? 
Because Parallel Cluster is a service that is entirely different from the above. 
The most noticeable difference is that Parallel Cluster does not exist in the console! 
Analysts who compare various cloud services based on the service list in the console often miss out on Parallel Cluster, thinking it might not be an official product despite its documentation on the AWS website.

Parallel Cluster is a CLI code developed as a Python library and is open-source, [available on GitHub](https://github.com/aws/aws-parallelcluster). The question arises, why create a separate CLI program when there is already an AWS CLI? To understand this, one needs knowledge of HPC. HPC, often described simply as a computation cluster, actually requires many software and resources to build. These can be individual services in the cloud. Below is the list of services currently used by Parallel Cluster:

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

It utilizes a total of 25 services. 
Not all clusters use all 25 services, but even the minimal use involves around 10 services. 
Hence, HPC in the cloud is a complex product that combines many elements. 
It's like financial products where basic products like savings, stocks, and bonds exist, but combined products like funds and ETFs are also available. 
Parallel Cluster allows customers to combine various products freely.

There is a counterargument that combined products should not be hidden from the console. 
[Lightsail](https://aws.amazon.com/Lightsail/), for example, combines services like EC2, EBS, VPC, Route 53, DynamoDB, and ECS, making it easier to use web services. However, Lightsail is available in the console with a more intuitive web interface. Similarly, services like OpenSearch and Hadoop (EMR) are provided in the console. The complexity and purpose of the combination seem to influence the decision not to show HPC products in the console.


Therefore, AWS initially provided this product only via CLI, without an API or web interface, but now they have started to provide them.

![](ui-image.png "Parallel Cluster UI(from AWS)")

The above screen shows the Parallel Cluster UI, which has a very similar interface to the AWS console. 
However, the way this web interface is provided is quite unique. 
Instead of being offered directly in the console, users need to implement it themselves using serverless web applications. 
All the code and infrastructure for implementing this serverless web application are open-source, [available on GitHub])(https://github.com/aws/aws-parallelcluster-ui), allowing users to set it up with just a few inputs and clicks. The same approach applies to the API; instead of using AWS's default API, users create their own APIs using API Gateway and Lambda.

위 화면은 Parallel Cluster UI의 화면으로 AWS 콘솔과 매우 유사한 UI를 가졌습니다. 
그런데 이 웹 인터페이스를 제공하는 방식이 매우 특이합니다. 
콘솔에서 제공해주는 방식이 아니라 사용자가 직접 서버리스 웹을 구현하는 형태입니다. 
이 서버리스 웹을 구현할 때의 코드와 인프라 코드들을 모두 [오픈소스](https://github.com/aws/aws-parallelcluster-ui) 로 공개하고 있기 때문에 사용자는 거의 입력 몇 번과 클릭 몇 번으로 서버리스 웹을 만들 수 있습니다. 
API도 마찬가지로 AWS의 기본 API를 쓰지 않고 API Gateway및 Lambda란 자신들의 상품을 사용하여 사용자 별로 API를 만드는 형태입니다.
이쯤 되면 AWS의 고집이 느껴집니다.
Parallel Cluster를 절대로 콘솔에 보여주지 않겠다 라는 것입니다. 심지어 웹 화면을 위한 프론트엔드, 백엔드 코드를 모두 만들었음에도 직접 사용자가 그 웹 서비스를 올려서 쓰라는 것입니다.

![](pcm-architecture.png "Parallel Cluster Architecture(from AWS)")

Azure와 Google Cloug의 HPC 서비스도 마찬가지입니다.
Google Cloud는 현재 CLI 툴만 제공하고 있습니다.
Azure는 CLI 툴과 함께 웹을 제공해주는데 자신들의 마켓플레이스에 있습니다.
대부분의 사람들은 마켓플레이스에는 CSP 자사의 상품이 아니라 써드 파티/파트너 회사들의 솔루션 상품을 올리는 것으로 인식을 합니다.
마켓플레이스에 자사의 솔루션을 올리는 것을 이해하지 못하는 사람들도 있습니다.
Paralell Cluster도 일종의 마켓플레이스 형 상품으로 생각해야 할 것 같습니다.
클라우드에서의 HPC를 다들 고성능 CPU, GPU장비, 또는 인피니밴드 같은 하드웨어만을 생각합니다.
하지만 이런 하드웨어들도 HPC의 컴퍼넌트일 뿐 그 자체로 HPC가 되지 않습니다.
결합형 상품임을 인식하여 어떤 형태로 제공해줄 것인가(또는 사용할 것인가)부터가 클라우드 HPC를 시작하는 첫 단계라고 생각합니다.
아쉽게도 제가 만난 사람들은 이런 인식의 전환이 막혀있는 사람들이 많았습니다.
그래서 저는 이런 마켓플레이스 형 상품을 CSP에서 직접 만들고 본인들의 마켓플레이스에 올리는가로 클라우드의 성숙도를 평가할 수 있다고 생각합니다.
기회가 된다면 Parallel Cluster와 Parallel Cluster UI 코드이 기반이 되는 Cloudformation, SDK, CDK, Lambda, API Gateway, Cognito같은 코드/서비스들드도 분석하고 코드에 대해서도 뜯어보는 글을 쓰고 싶네요. 




