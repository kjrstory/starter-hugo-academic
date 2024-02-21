---
title: 오픈소스 설치매니저 Spack에서 OpenFOAM 활용
date: '2023-10-01'
draft: false
publishDate: '2024-02-22T08:00:00Z'
authors:
- ' 김종록'
publication_types:
- '1'
featured: false
publication: '*10th OKUCC*'
url_pdf: https://nextfoam.co.kr/proc/DownloadProc.php?fName=231101150741_stiSq0tEeT.pdf&realfName=10thOKUCC_%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4%20%EC%84%A4%EC%B9%98%EB%A7%A4%EB%8B%88%EC%A0%80%20Spack%EC%97%90%EC%84%9C%20OpenFOAM%20%ED%99%9C%EC%9A%A9.pdf
image:
  filename: fig-01.png
  focal_point: Smart
  preview_only: false
---

[OKUCC](https://nextfoam.co.kr/okuc.php)는 OpenFOAM Korea Users’ Community Conference로 넥스트폼에서 주최하는 한국의 오픈폼 커뮤니티 행사입니다.
제가 OpenFOAM을 사용하기 시작한 것은 한화에서 일할 때였습니다. 
하지만 OKUCC를 정작 한화에 다닐 때는 가지 못하였고 많이 사용할 수 없는 삼성 SDS에서 발표를 하게 되었습니다.
한화에 있을 때는 OKUCC는 아니지만 매년 학회에 참석 했었는데 삼성SDS에 와서는 그러지 못하였습니다.
이 발표를 계기로 매년 최소 두 번은 대외 활동을 하기로 목표를 세웠습니다.

OKUCC는 일반적인 학회보다는 가벼운 분위기였지만 그렇다고 발표하는 내용도 쉽지는 않았습니다.
다만 유저 커뮤니티 행사이다 보니 프리젠테이션 형식의 발표자료만 공유하고 논문 형태의 자료는 작성하지 않아도 됩니다.
이것은 장점이자 단점인데 발표하는 입장에서는 논문을 작성하는 고생을 하지 않아도 되지만 듣는 입장에서 프리젠테이션으로는 자세한 내용을 알 수가 없다는 아쉬운 점도 있습니다.

그래서 이 포스트에 각 슬라이드에 대한 설명을 담으려고 합니다.
설명은 실제 발표 시에 이야기한 내용도 있지만 아닌 내용도 담았습니다.

#### 2장
OpenFOAM의 주요 진입장벽 중 하나는 설치가 어렵다는 것입니다. 설치가 어려운 이유는 OpenFOAM에 필요한 패키지들이 매우 복잡하게 얽혀있기 때문입니다.
컨테이너가 해결사가 될 것이라고 기대를 했지만 아직까지는 HPC 분야의 모든 어플리케이션을 컨테이너로 대체할 수 없는 상황입니다.
하지만 저는 컨테이너 for HPC도 관심있게 보고 있습니다. 
HPC용 컨테이너 기술에는 Apptainer(Singularity), CharlieCloud, Shifter, Sarus, Podman 등이 있으며, 이들의 차이점을 탐구하는 것이 매우 흥미울듯 합니다.

#### 3장
설치 과정에서 중요한 것 중 하나는 컴파일 옵션에 따라 성능이 크게 달라진다는 것입니다. 
GROMACS라는 오픈소스를 예로 들면, Intel Cascade Lake에서 다른 컴파일 옵션을 사용했을 때 성능 차이가 70%에 이르렀습니다. 
저는 OpenFOAM에서도 유사한 테스트를 진행했지만, 예상과 달리 큰 차이가 나타나지 않았습니다. 
이는 소프트웨어의 코드 구조 차이에서 기인한 것으로 보이며, 이러한 성능 차이의 원인을 분석하는 것도 매우 중요한 연구 주제입니다.
그래서 가능하다면 2024년에 OKUCC에는 성능에 테스트를 보완하여 성능에 대한 주제로 발표할 계획이 있습니다.
하지만 성능 분석은 이번 발표 때 했던 노력에 비해 두 배는 어려운 일이라서 포기할수도 있습니다.

#### 4장
OpenFOAM이 겪고 있는 설치와 관련된 문제는 다른 많은 오픈소스 소프트웨어에서도 공통적으로 나타나고 있습니다
그래서 [Spack](https://spack.io)이란 오픈소스 패키지 매니저가 미국의 LLNL 연구소에서 개발되었습니다.
Spack은 현재 굉장히 활발한 개발이 이루어지고 있으며 7000개 이상의 패키지가 등록되어있습니다.
해외에서는 여러 유수의 슈퍼컴퓨터 센터에서 사용되고 있지만 한국에서는 많이 사용하고 있지 않은 것 같습니다.

#### 5장
Spack의 설치는 Git을 이용한 간단한 클론 과정을 통해 이루어집니다.
OpenFOAM을 Spack으로 설치하는 것은 매우 간단하며, '@', '%', '+', '^'와 같은 식별자를 사용하여 상세 옵션을 설정할 수 있습니다.
이러한 간편함은 Spack이 제공하는 큰 장점 중 하나입니다.

#### 6~7장
`spack info openfoam` 명령을 실행하면 OpenFOAM에 대한 간략한 설명과 지원하는 버전, 설치 옵션들이 나타납니다. 
Variants 섹션에서는 OpenFOAM에 적용할 수 있는 다양한 세부 옵션들을 볼 수 있습니다.
Dependencies 항목에서는 OpenFOAM 설치에 필요한 사전 패키지들을 확인할 수 있습니다.
boost, cgal, paraview와 같은 사전 패키지들도 Spack을 통해 간편하게 설치할 수 있는 것이 큰 장점입니다.

#### 8장
그렇다면 이런 일련의 과정들을 어떻게 Spack은 백그라운드에서 처리하는걸까요?
Spack은 각 패키지마다 DSL(Domain Specific Language)으로 작성된 별도의 레시피 파일을 가지고 있으며, 이 파일은 파이썬으로 작성됩니다.
이 레시피 파일에서는 패키지의 의존성, 설치 로직 등을 설정할 수 있습니다.
Spack의 레시피 파일을 작성하기 위해서는 Spack의 공식 문서를 참고해야 하며, 이 과정에서 레시피의 주요 문법을 이해하는 것이 중요합니다.

#### 9장
OpenFOAM의 레시피 파일은 어떻게 작성이 되었는지 살펴봅시다.
그 전에 OpenFOA의 배포판은 크게 [OpenCFD](https://www.openfoam.com)에서 제공하는 배포판과 [OpenFOAM Foundation](https://www.openfoam.org)에서 배포하는 배포판, Hrvoje Jasak이 주도하여 배포하는 [Foam-extend](https://sourceforge.net/projects/foam-extend/) 배포판이 있습니다.
Spack에서는 배포판마다 별도의 패키지로 구분하며 각각 레시피 파일이 존재합니다.
2017년경에 OpenCFD 배포판을 기준으로 다른 배포판의 레시피가 작성되었습니다.
OpenCFD배포판은 유지보수가 잘되고 있었지만 다른 배포판은 유지보수가 잘되고 있지 않았습니다.

#### 10장
그래서 저는 다른 OpenFOAM Foundation 배포판의 레시피를 개선 하려고 했습니다.
기여하는 방법에는 여러가지가 있는데 저는 여기서 PR(Pull Request)을 하여 직접 소스코드에 기여하고자 하였습니다.
PR 사례는 총 5가지이고 다음과 같습니다.
1. Version URL Function
2. Precision Options
3. Decomposition Method(Zoltan)
4. New Solver
5. Etc.

#### 11~12장
처음으로 간단한 사례를 보여드리려고 합니다.
Spack에서는 각 패키지 버전에 대한 다운로드 URL을 작성합니다.
슬라이드의 왼쪽을 보면 OpenFOAM의 다운로드 URL에도 일정한 규칙이 있는 것을 알 수 있습니다.
단 한가지 규칙이 달라지는 것은 5.0이하의 버전에는 5.x 와 같이 x라는 규칙을 씁니다. 
버전마다 url을 작성하지 않고 함수로 만들어서 코드를 간단히 하려고 합니다.
Spack에서는 url_for_version이란 함수로 패키지의 특별한 규칙을 정할수 있습니다.
처음에는 바뀐 부분이 매우 단순했기 때문에 매우 쉽게 통과할 것으로 기대했지만 리뷰에서 부정적인 의견을 받았습니다.
이 패키지의 가장 첫번째 버전인 2.3.1은 github의 url을 사용하지 않고 소스포지의 url을 사용하게 되는데 이것이 함수에 담겨있지 않았기 때문입니다.
그래서 2.3.1에서는 소스포지의 url, 5.0이하에서는 x 규칙이 들어가게 규칙을 추가하였습니다.
 

#### 13장

두 번째는 정밀도에 관한 것입니다.
OpenFOAM은 기본적은로 단정밀도(single precision), 배정밀도(double precision)을 제공하고 있습니다.
이제는 두 정밀도외의 방법도 개발이 되고 있습니다. 
하나는 mixed precision이라고 하여 단정밀도와 배정밀도를 결합한 방법입니다.
정밀도를 고려해야하는 부분에만 배정밀도 방법을 쓰고 그렇지 않은 부분에는 단정밀도를 써서 파일 용량과 계산 시간들을 효율적으로 하는 방법입니다.
두 번째는 long double precision입니다. 이는 배정밀도보다 더 정밀도를 확장한 방법입니다.
배포판마다 이런 설정들이 달라서 표로 정리를 해보았습니다.
OpenCFD 배포판에는 mixed precision이 1906버전에 들어가있고 Spack에 등록되어있습니다.
Foundation 배포판에는 long double precision이 2018년도에 배포된 6.0버전부터 들어가 있지만 Spack에는 미등록되어 있습니다.
Foam-extend 배포판에도 long double precision이 2015년도에 배포된 3.2버전부터 들어가 있지만 역시 Spack에는 미등록되어 있습니다. 

#### 14장
이것을 코드에 반영하는 것은 어렵지는 않습니다. 
단 이런 옵션들이 어떤 버전부터 적용되어있는 것을 알아내는 것이 더 중요합니다.
Commit 내역을 검색하거나 일일이 모든 commit을 확인하면서 찾아내는 방법도 있지만 번거롭습니다.
Github의 Blame기능을 쓰면 좀 더 쉽게 알아낼 수 있습니다.
OpenFOAM에서 정밀도 설정은 bashrc파일에 지정하게 되어있고 이 파일에 Blame기능을 쓰면 어떤 커밋에 의해 변경되었는지 정확히 알 수 있습니다.
이 커밋으로 새로운 정밀도 방법이 어느 버전부터 적용되었는지 알 수 있습니다.
만약 예를 들어 Foundation 배포판의 OpenFOAM에서 5.0에 long double precision을 적용하려고 한다면 설치 전에 에러 메세지를 출력하게 됩니다.

#### 15장
세 번째는 decomposition 메소드 사례입니다. 
decomposition 메쏘드는 격자를 병렬 해석하기 위해 여러 파트로 나누는 작업입니다.
많이 사용되는 Metis와 Scotch방법외에도 Kahip, Zoltan같은 새로운 방법들이 개발되고 있습니다.
이것도 배포판마다 적용된 방법이 다릅니다. 
OpenCFD 배포판에는 Kahip 메소드만 적용이 되었고 Spack에 반영이 되어있습니다.
Foundation 배포판에는 Zoltan방법이 2023년에 배포된 버전에 적용이 되었지만 역시 Spack에는 반영이 되어있지 않습니다.
한가지 유의해야할 것은 Zoltan은 renumbering방법에도 사용이 되므로 기존 패키지들도 Zoltan 패키지가 필요할 수 있습니다.

#### 16장

Foundation 배포판에는 Zoltan 패키지가 사전 패키지로 등록되어 있지 않기 때문에 추가해야 합니다.
한가지 유의할 것은 단순히 패키지만 필요로 하는것이 아니라 패치가 필요하다는 것입니다.
일단 Zoltan 패키지가 shared버전(`zoltan+shared`)으로 필요합니다.
그런데 shared버전으로 설치를 하게되면 확장자가 so 파일입니다.
OpenFOAM에서는 이것을 a 확장자로 임의로 바꿔서 정리합니다.
이런 서로 맞지 않는 것을 맞추기 위해 패치가 필요합니다.

#### 17장~18장
4번째 사례는 새로운 솔버에 관한 것입니다.
Oak Ridge 국립 연구소에서는 [AdditiveFOAM](https://github.com/ExascaleAM/AdditiveFOAM)이라고 하여 Spack에 새로운 패키지로 등록하기 원했습니다.
이 솔버는 적층 제조에 관한 솔버입니다.
이들이 처음 제시한 commit은 Spack 레시피의 DSL 문법을 충분히 사용하지 않은 상황이었습니다.
OpenCFD배포판에서는 Additive Foam처럼 기본 OpenFOAM을 사전 패키지로 한 추가 패키지들을 위해 `spack-derived-Allwmake`스크립트도 만들었습니다.
이 스크립트를 기반으로 빌드 로직을 새로 만들었습니다.

#### 19장
OpenFOAM외에도 다른 시뮬레이션 코드들의 레시피 파일도 수정을 했습니다.
FDS는 화재 시뮬레이션에 사용하는 코드로 포트란으로 되어 있습니다.
FDS는 제가 Spack에 처음 기여한 사례로써 의미가 있습니다.
SU2는 범용적인 CFD코드로 주로 공기역학 해석 사례에 많이 사용되고 있습니다.
SU2는 Meson이란 좀 더 현대적인 빌드 방법을 사용하는데 이 패키지 레시피에 기여하기 위해 Meson에 대해 알아볼 수 있었습니다.
OpenRadioss는 포트란으로 작성된 구조해석 코드입니다.
OpenRadioss는 전처리와 솔버가 나누어져있고 빌드도 각각 하게 됩니다.
OpenFOAM도 자체적인 전처리 기능이 있지만 빌드를 따로 하진 않습니다. 
OpenRadioss는 빌드를 각자 하므로 고민 결과 Spack에 패키지를 나누어서 등록하기로 했습니다.
일반적으로 1 리포지토리 1 Spack 패키지인 경우가 많으나 OpenRadioss는 1 리포지토리에 2 Spack 패키지인 경우로 의미가 있겠습니다.

#### 20장

저는 FDS, SU2, OpenRadioss의 Spack 레시피 운영자로 등록되어 있습니다.
이제 OpenFOAM Foundation 배포판의 레시피 운영자가 되기로 했습니다.
위의 3코드도 아주 좋은 코드이지만 OpenFOAM은 사용자가 아주 많은 오픈소스 코드라서 이 코드의 운영자가 되는데 고민을 많이 했습니다.
그럼에도 기여하려고 하는 것은 기여로 인해 저도 성장을 하기 때문입니다.

