---
title: Cloud HPC 서비스 분류 및 목록
date: 2023-06-05T07:22:43.471Z
draft: false
featured: false
authors:
  - admin
tags:
  - cloud
  - hpc
categories:
  - Cloud
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---

In this article, we aim to classify Cloud HPC services into four categories. I welcome any feedback or suggestions if there are any errors or additions to be made.

## 1 DIY Cloud

The first category is DIY Cloud for HPC, which allows users to freely configure their HPC environments and computational resources according to their preferences. However, it can be challenging to create a hybrid setup due to a lack of knowledge about cloud technologies or variations in development tools across different cloud providers. Leading companies in this category include major CSPs (Cloud Service Providers) such as AWS. However, recently, non-CSP companies like Cloudy Cluster and Rntier Cloud have also emerged, offering new features and products.

* AWS Parallel Cluster: <https://aws.amazon.com/hpc/parallelcluster/>
* Azure CycleCloud: <https://learn.microsoft.com/en-us/azure/cyclecloud/?view=cyclecloud-8>
* GCP HPC Toolkit: <https://cloud.google.com/hpc-toolkit/docs/overview>
* Alibaba Elastic HPC: <https://www.alibabacloud.com/product/ehpc>
* KT Cloud HPC: <https://cloud.kt.com/product/computing/hpc/>
* SCP HPC Cluster: <https://www.samsungsds.com/en/compute-hpccluster/hpccluster.html>
* Cloudy Cluster: [https://cloudycluster.com](https://cloudycluster.com/)
* Rntier Cloud: <http://www.clunix.com/RNTier_Cloud>
* OpenFlightHPC: [https://www.openflighthpc.org](https://www.openflighthpc.org/)

## 2 ISV Cloud

The second category is ISV Cloud for HPC, which refers to products offered by Independent Software Vendors (ISVs) that provide cloud capabilities for their own software products. These products typically do not have a separate web interface, allowing users to directly use cloud features within the respective software (primarily desktop applications). However, a drawback of this category is that each ISV company offers their own product line, requiring separate configurations when using multiple Computer-Aided Engineering (CAE) applications.

* Ansys Cloud Direct: <https://www.ansys.com/products/cloud/ansys-cloud>
* Simcenter Cloud HPC: <https://plm.sw.siemens.com/en-US/simcenter/integration-solutions/cloud-hpc/>
* Synopsys Cloud: <https://www.synopsys.com/cloud.html>
* Cadence Cloud: <https://www.cadence.com/en_US/home/solutions/cadence-cloud.html>
* Altair One: <https://altair.com/altair-one>
* Simulia Cloud: <https://www.3ds.com/products-services/simulia/products/simulia-cloud/>

## 3 SaaS Cloud

The third category is SaaS Cloud for HPC, which primarily offers web-based solutions that include CAE applications. These products are provided through collaboration between CSP companies and CAE ISVs, with Rescale being one of the most prominent examples of successful collaboration in this regard. Recently, there has been an increase in companies such as Simscale, PACE, CFD FEA SERVICE SRL, specializing in offering tailored solutions based on open-source products.

* Rescale: [https://rescale.com](https://rescale.com/)
* Gompute: <https://www.gompute.com/>
* Uber Cloud: [https://www.theubercloud.com](https://www.theubercloud.com/)
* Scala Computing: [https://www.scalacomputing.com](https://www.scalacomputing.com/)
* TAESUNG Cloud: <https://www.etsne.com/CloudService>
* Simscale: [https://www.simscale.com](https://www.simscale.com/)
* CFD FEA Service SRL: [https://cfdfeaservice.it](https://cfdfeaservice.it/)
* PACE on Cloud: <https://landing.pace-on-cloud.com/>
* Sabalcore: [https://www.sabalcore.com](https://www.sabalcore.com/)

## 4 Job Scheduler형 클라우드

Lastly, It is the Job Scheduler Cloud for HPC. In the traditional on-premises HPC middleware market, job scheduler companies have been dominant. However, with the advent of the cloud era, even traditional commercial schedulers like IBM LSF and Altair PBS are offering cloud capabilities within their products. The products in this category have the advantage of being able to easily configure hybrid setups, such as combinations of multiple clouds or a combination of on-premises and cloud usage.

* Altair Control: <https://altair.com/control>
* IBM LSF Suite: <https://www.ibm.com/products/hpc-workload-management>
* Slurm Cloud: <https://slurm.schedmd.com/elastic_computing.html>
* MS HPC Pack: <https://learn.microsoft.com/en-us/powershell/high-performance-computing/overview?view=hpc19-ps>

