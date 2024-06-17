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
[Lightsail](https://aws.amazon.com/Lightsail/), for example, combines services like EC2, EBS, VPC, Route 53, DynamoDB, and ECS, making it easier to use web services. 
However, Lightsail is available in the console with a more intuitive web interface. Similarly, services like OpenSearch and Hadoop (EMR) are provided in the console. The complexity and purpose of the combination seem to influence the decision not to show HPC products in the console.
Therefore, AWS initially provided this product only via CLI, without an API or web interface, but now they have started to provide them.

![](ui-image.png "Parallel Cluster UI(from AWS)")

The above screen shows the Parallel Cluster UI, which has a UI very similar to the AWS console. The similarity likely stems from the fact that the developers of this code are affiliated with AWS. 
If you look at the [repository](https://github.com/aws/aws-parallelcluster-ui) contributors, you’ll see that the developers are indeed working at AWS, with minimal contributions from outside. 
Although this repository is open-source, it feels more like a restaurant with a transparent kitchen where you can see the cooking process, rather than an open marketplace where everyone can participate.
Parallel Cluster and Parallel Cluster UI repositories are separate, but both codes are exclusively for use with AWS, suggesting a commitment to transparency in their development process.

However, the way this web interface is provided is quite unique. 
Instead of being available directly in the console, users need to implement it themselves using serverless web applications. 
All the frontend, backend, and infrastructure codes needed to implement this serverless web application are open-source, allowing users to set it up with just a few inputs and clicks. 
The same approach applies to the API; instead of using AWS's default API, users create their own APIs using API Gateway and Lambda.

At this point, it feels like AWS is adamant about not showing Parallel Cluster in the console. Even though they have created both frontend and backend code for the web interface, they insist that users host the web service themselves.

![](pcm-architecture.png "Parallel Cluster Architecture(from AWS)")


Azure and Google Cloud's HPC services are similar. Google Cloud currently only provides a CLI tool called [HPC Toolkit](https://github.com/GoogleCloudPlatform/hpc-toolkit). Azure offers a product called [CycleCloud](https://learn.microsoft.com/en-us/azure/cyclecloud/overview?view=cyclecloud-8) that provides both CLI tools and a web interface, but it is also available in their marketplace rather than the console. Using this marketplace product involves running a web server on a virtual machine. Therefore, Azure and Google Cloud also do not have HPC services available directly in the console.

Most people perceive the marketplace as a platform for third-party/partner solutions rather than CSP's own products. Some do not understand why CSP would list their solutions in the marketplace. Parallel Cluster should be considered a marketplace-type product as well.

In the cloud, HPC is often thought of only in terms of high-performance CPUs, GPUs, or InfiniBand hardware. However, these are just components of HPC, not HPC itself. Recognizing it as a combined product and determining how to provide or use it is the first step in cloud HPC. Unfortunately, many people I’ve met are not open to this change in perspective. Therefore, I believe the maturity of a cloud can be evaluated by whether the CSP creates and lists marketplace-type products in their marketplace. If given the chance, I would love to delve into the CloudFormation, SDK, CDK, Lambda, API Gateway, and Cognito codes/services that underpin Parallel Cluster and its UI.
