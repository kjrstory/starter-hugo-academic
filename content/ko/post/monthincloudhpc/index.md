---
title: 2024년 2월 Cloud HPC 블로그 기사 리뷰
date: 2024-02-10T08:00:00.000Z
draft: false
featured: false
authors:
  - admin
categories:
  - Cloud
  - HPC
---



### Choosing the Right Azure Spot VM for Rendering Workloads Using MoonRay ![Link](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)

[Link](https://techcommunity.microsoft.com/t5/azure-high-performance-computing/choosing-the-right-azure-spot-vm-for-rendering-workloads-using/ba-p/4056551)

* 일자: 2/14
* [MoonRay](https://github.com/dreamworksanimation/openmoonray)(드림웍스 애니메이션에서 개발한 몬테카를로 레이 트레이싱 렌더러)를 Azure H-시리즈에서 사용할 때의 성능과 가성비를 평가
* 성능(렌더링 속도)은 가장 최신 세대인 HBv4(AMD EPYC Genoa-X)가 가장 좋음. 이는 CPU의 발전으로 인한 당연한 결과
* 단 가성비는 이전 세대인 HBv3(AMD EPYC Milan-X)가 가장 우수하며 HBv4에 비해 4~6배의 큰 차이가 있음. 이는 Spot 가격으로 인한 것으로 HBv4는 물량에 제한이 있어 Spot 가격이 HBv3보다 상대적으로 비싸기 때문
* Spot 가격은 시기마다 변동이 됨을 인지하고 사용자의 시뮬레이션이 Spot 인스턴스의 정책(시작 시점 불분명하고 갑자기 종료 될 수 있는 특징)에 맞게 변경 가능해야함

### Accelerating OpenFOAM Integration with Azure CycleCloud

[Link](https://techcommunity.microsoft.com/t5/azure-high-performance-computing/accelerating-openfoam-integration-with-azure-cyclecloud/ba-p/4055616)

* 일자: 2/13
* 오픈소스 CFD 코드인 [OpenFOAM](https://www.openfoam.com)을 Azure에서 설치하고 간단한 예제를 수행하는 스크립트를 제공

### Learn how to power your AI transformation with the Microsoft Cloud at NVIDIA GTC

[Link](https://techcommunity.microsoft.com/t5/azure-high-performance-computing/learn-how-to-power-your-ai-transformation-with-the-microsoft/ba-p/4043868)

* 일자 : 2/09
* [Nvidia GTC](https://www.nvidia.com/gtc/)(3/18~21)에 Microsoft 세션 소개

### Dynamic HPC budget control using a core-limit approach with AWS ParallelCluster

[Link](https://aws.amazon.com/ko/blogs/hpc/dynamic-hpc-budget-control-using-a-core-limit-approach-with-aws-parallelcluster/)

* 일자: 2/08
* HPC클러스터에서 매주 동적으로 예산 관리하는 방법 소개
* AWS의 Cost Explorer 기능과 Slurm의 License management method를 이용
* [Dynamic EC2 budget control](https://github.com/aws-samples/dynamic-ec2-budget-control)에 샘플 코드를 공개
