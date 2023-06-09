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

본 글에서는 Cloud HPC 서비스를 4가지 유형으로 분류하려고 합니다. 개인적으로 정리한 것을 공유하는 것이니 혹시 틀렸거나 추가해야 될 것이 있어 의견 주시면 환영입니다.

## 1 DIY형 클라우드

첫 번째는 DIY형 HPC Cloud로, 이는 사용자가 원하는 대로 HPC 환경과 계산 자원을 자유롭게 구성할 수 있는 상품입니다. 하지만 클라우드에 대한 지식이 부족하거나 클라우드마다 개발 도구가 다르기 때문에 하이브리드 형태로 만들기 어려울 수 있습니다. 이 분류에서는 AWS를 포함한 메이저 CSP 순위에 해당하는 회사들이 선두를 달리고 있 습니다. 단 최근에는 Cloudy Cluster나 Rntier Cloud처럼 CSP가 아닌 회사에서도 다른 특징을 내세워 새로운 상품을 만들고 있습니다.

* AWS Parallel Cluster: <https://aws.amazon.com/hpc/parallelcluster/>
* Azure CycleCloud: <https://learn.microsoft.com/en-us/azure/cyclecloud/?view=cyclecloud-8>
* GCP HPC Toolkit: <https://cloud.google.com/hpc-toolkit/docs/overview>
* Alibaba Elastic HPC: <https://www.alibabacloud.com/product/ehpc>
* KT Cloud HPC: <https://cloud.kt.com/product/computing/hpc/>
* SCP HPC Cluster: <https://www.samsungsds.com/en/compute-hpccluster/hpccluster.html>
* Cloudy Cluster: <https://cloudycluster.com/>
* Rntier Cloud: <http://www.clunix.com/RNTier_Cloud>
* OpenFlightHPC: <https://www.openflighthpc.org/>

## 2 ISV형 클라우드

두 번째는 ISV형 HPC Cloud로, 이는 ISV(Independent Software Vendor) 회사가 자신들의 제품에 클라우드 기능을 제공하는 제품입니다. 별도의 웹 인터페이스가 없어도 해당 제품(주로 Desktop application)에서 바로 클라우드 기능을 사용할 수 있으나, 각 ISV 회사마다 자신들의 제품군만 제공하므로 여러 CAE Application을 사용할 경우 각각 설정을 해줘야 하는 단점이 있습니다.

* Ansys Cloud Direct: <https://www.ansys.com/products/cloud/ansys-cloud>
* Simcenter Cloud HPC: <https://plm.sw.siemens.com/en-US/simcenter/integration-solutions/cloud-hpc/>
* Synopsys Cloud: <https://www.synopsys.com/cloud.html>
* Cadence Cloud: <https://www.cadence.com/en_US/home/solutions/cadence-cloud.html>
* Altair One: <https://altair.com/altair-one>
* Simulia Cloud: <https://www.3ds.com/products-services/simulia/products/simulia-cloud/>

## 3 SaaS형 클라우드

세 번째는 SaaS형 HPC Cloud로, 이는 주로 웹 형태로 CAE Application까지 같이 제공하는 상품입니다. 이 분류의 상품들은 CSP 회사와 CAE ISV들과 협력을 통해 제공되며, 이 협력 관계를 가장 잘 구축한 회사는 Rescale입니다. 최근에는 Simscale이나 PACE, CFD FEA SERVICE SRL 등 Open Source 상품들을 특성화하여 제공하는 경우가 늘어나고 있습니다.

* Rescale: <https://rescale.com/>
* Gompute: <https://www.gompute.com/>
* Uber Cloud: <https://www.theubercloud.com/>
* Scala Computing: <https://www.scalacomputing.com/>
* TAESUNG Cloud: <https://www.etsne.com/CloudService>
* Simscale: <https://www.simscale.com/>
* CFD FEA Service SRL: <https://cfdfeaservice.it/>
* PACE on Cloud: <https://landing.pace-on-cloud.com/>
* Sabalcore: <https://www.sabalcore.com/>

## 4 Job Scheduler형 클라우드

마지막으로, Job Scheduler형 HPC Cloud입니다. 기존의 온프레미스 HPC 미들웨어 시장은 Job Scheduler 회사가 지배적이었습니다. Cloud 시대가 오면서 IBM LSF와 Altair PBS 등의 전통적인 상용 스케쥴러도 본인들의 상품에 Cloud 기능을 제공하고 있습니다. 이 분류의 상품들은 하이브리드 형태(여러 클라우드를 쓰는 조합 또는 온프레미스와 클라우드를 같이 사용하는 조합)를 구성하기 쉽다는 장점이 있습니다.

* Altair Control: <https://altair.com/control>
* IBM LSF Suite: <https://www.ibm.com/products/hpc-workload-management>
* Slurm Cloud: <https://slurm.schedmd.com/elastic_computing.html>
* MS HPC Pack: <https://learn.microsoft.com/en-us/powershell/high-performance-computing/overview?view=hpc19-ps>

