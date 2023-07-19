---
title: "오픈소스 Spack의 설치 레시피 기여 사례: Openfoam (2)"
date: 2023-05-24T11:52:56+09:00
draft: false
featured: false
authors:
  - admin
tags:
  - spack
  - openfoam
categories:
  - HPC
---

1편 링크: (/ko/post/spack_openfoam_contribution1)

전편에 이어서 기여 사례에 대해 설명하겠습니다. 사실 버전에 관련된 부분이 레시피에 관한 것 중 가장 쉬운 부분에 해당합니다. version이후에는 variant와 dependency에 대한 설정이 나오고 patch 설정 후 컴파일하게 되는 절차들을 레시피에 작성합니다. 먼저 variants부터 보기로 합니다.

{{% callout info %}}
  ```
  Variants:
    Name [Default]            When    Allowed values    Description
    ======================    ====    ==============    ==================================================

    build_system [generic]    --      generic           Build systems supported by the package
    float32 [off]             --      on, off           Use single-precision
    int64 [off]               --      on, off           With 64-bit labels
    kahip [off]               --      on, off           With kahip decomposition
    knl [off]                 --      on, off           Use KNL compiler settings
    metis [off]               --      on, off           With metis decomposition
    mgridgen [off]            --      on, off           With mgridgen support
    paraview [off]            --      on, off           Build paraview plugins and runtime post-processing
    scotch [on]               --      on, off           With scotch/ptscotch decomposition
    source [on]               --      on, off           Install library/application sources and tutorials
    spdp [off]                --      on, off           Use single/double mixed precision
    vtk [off]                 --      on, off           With VTK runTimePostProcessing
    zoltan [off]              --      on, off           With zoltan renumbering

  ```
{{% /callout %}}
Openfoam.com(OpenCFD)의 variant


{{% callout info %}}
  ```
  Variants:
    Name [Default]            When    Allowed values    Description
    ======================    ====    ==============    =================================================

    build_system [generic]    --      generic           Build systems supported by the package
    float32 [off]             --      on, off           Compile with 32-bit scalar (single-precision)
    int64 [off]               --      on, off           Compile with 64-bit label
    metis [off]               --      on, off           With metis decomposition
    source [on]               --      on, off           Install library/application sources and tutorials
  ```
{{% /callout %}}
Openfoam.org의 variant

{{% callout info %}}
  ```
  Variants:
    Name [Default]            When    Allowed values    Description
    ======================    ====    ==============    =================================================

    build_system [generic]    --      generic           Build systems supported by the package
    float32 [off]             --      on, off           Compile with 32-bit scalar (single-precision)
    metis [on]                --      on, off           With metis for decomposition
    paraview [off]            --      on, off           Build paraview plugins (eg, paraFoam)
    parmetis [on]             --      on, off           With parmetis for decomposition
    parmgridgen [on]          --      on, off           With parmgridgen support
    ptscotch [on]             --      on, off           With ptscotch for decomposition
    scotch [on]               --      on, off           With scotch for decomposition
    source [on]               --      on, off           Install library/application sources and tutorials
  ```
{{% /callout %}}
foam-extend의 variant

 
역시 ESI 배포판이 압도적으로 많습니다. 실제 배포판의 차이일 수도 있겠지만 제가 알고 있는 바에서는 이 정도로 기능의 차이가 나지는 않는 걸로 알고 있고 단순히 레시피를 만드는 차이라고 보면 되겠습니다.
먼저 float32 옵션을 보면 single precision을 사용하는 옵션입니다. 여기서 precision에 대해서 알아야 하는데 Openfoam은 CFD코드이고 CFD를 전공했다면 precision은 기본 개념으로 알고 있을 것입니다. single, double precision(SP,DP)이 있고 long double precision도 있습니다. 쉽게 말해 정확도라고 생각하면 되는데 수학적인 정의 보다는 실제 경험을 생각해보면 사실 거의 double precision으로 해석하면  됩니다. single과 double precision간에 계산 시간은 거의 차이가 나지 않고 메모리나 파일의 용량은 차이가 날 수 있지만 RAM과 스토리지가 CFD쪽에서는 타이트한 자원이 아니다 보니 그냥 precision 고민할 바에 자원을 더 확보하는 것이 낫습니다. 만약 long double precision으로 해석해야 되는 시뮬레이션이었다면 선배나 교수님이  반드시 이야기했을 거고 스스로 어떤 정확도로 풀어야 될지 판단할 수 있는 레벨이 될 때는 이런 팁도 필요 없어지겠죠?

그런데 ESI 배포판에서는 spdp란 variants도 있습니다. 이는 single precision과 double precision을 섞어 쓰는 것입니다. CFD 시뮬레이션도 내부에서는 수많은 수치해석 메쏘드가 쓰이는 것인데 정확도가 중요한 부분에서는 double precision을 쓰고 덜 중요한 부분에서는 single을 쓰는 방법입니다. 그런데 이 옵션이 다른 배포판에서는 없네요. 이제 각 배포판의 실제 소스코드를 뒤져봐야 합니다. Openfoam의 설치 설정파일은 여러 개 있는데 가장 대표적인 것은 `[source폴더]/etc/bashrc` 입니다.  이 정보는 사실 Openfoam을 직접 컴파일해서 설치하지 않았다면 알기 어려운 정보이며 CFD에 대한 기본적인 지식이 있어야 합니다.

{{% callout note %}}

```
  # [WM_PRECISION_OPTION] - Floating-point precision:
  # = DP | SP | SPDP
  export WM_PRECISION_OPTION=DP
```

{{% /callout %}}

Precision 옵션은 반드시 3개중에 하나가 선택되어야 하는 옵셔인데 레시피 코드를 보면 boolean 두 개로 이것을 처리합니다.

```python
  if "+spdp" in spec:
      self.precision_option = "SPDP"
  elif "+float32" in spec:
      self.precision_option = "SP"
```

사용자가 실수로 +spdp+float32 로 입력을 주면 임의로 SPDP로 정하는 건데 이런 정보를 사용자가 알 수가 없으므로 잘못된 코드라고 할 수 있겠습니다. boolean 2개 대신 list안에서 선택하는 변수로 바꿔줘야 합니다. 단 이건 기존의 옵션을 삭제하는것이라 혼선이 있을 수 있기 때문에 PR전에 Issue부터 올리고 의견을 물어보았습니다.

PR 그림 캡처

이번에는 Foundation 배포판을 보면 SP, DP외에 LP란 옵션이 추가로 있습니다. 이건 Long double precision을 의미합니다. LP 옵션을 추가하고 싶은데 과연 모든 버전에서 LP가 있었던 것일까 의심이 됩니다. 이럴 때는 릴리즈 또는 커밋 히스토리를 뒤져보거나  리포지토리의 Blame 기능을 써서 찾을 수 있습니다.

`WM_PRECISION_OPTION = SP | DP | LP`

아래와 같은 커밋을 찾았습니다. tag가 version6부터 있는 것으로 봐서 6 버전부터 적용이 되었군요.

[OpenFOAM: Added support for extended precision scalar](https://github.com/OpenFOAM/OpenFOAM-dev/commit/d82cc36c5af97e799a82fadf455e06d192ae1e65)
캡처

그런데 다시 돌아가 그럼 ESI 배포판은 옛날 버전도 spdp가 될까요? 보통 sp와 dp는 기본적으로 지원하고 그 외의 옵션은 나중에 개발이 됩니다. ESI 배포판의 깃 리포지토리는 [여기](https://develop.openfoam.com/Development/openfoam)입니다. 깃헙이 아닌 깃랩(Gitlab)으로 구축했네요. 역시 마찬가지로 blame으로 찾아낼 수 있습니다.
[ENH: add primitives support for mixed precision](https://develop.openfoam.com/Development/openfoam/-/blob/46bc808261ef44cb29b512cb0c93acabdc09153a/etc/bashrc)
캡처

태그를 보면 1906버전부터 적용이 된 것을 알 수 있습니다. 이렇게 특정 버전만 적용되는 것이라면 반드시 레시피에 조건문을 달아야 합니다. 그리고 옵션 설명에도 명시를 해서 사용자에게 혼선이 없도록 해야 합니다.

Foam-Extend도 살펴보겠습니다. Foam-Extend의 리포지토리는 소스포지네요. https://sourceforge.net/projects/foam-extend/
[Feature: Single precision and long double precision port](https://sourceforge.net/p/foam-extend/foam-extend-3.2/ci/6b022758d1b15a8d08718a78d3f68879e95bcf90)

`WM_PRECISION_OPTION = LDP | DP | SP`

여기는 LP란 이름 대신 LDP를 쓰고 3.2버전부터 적용이 되었네요.
정리를 하면 3가지 개선 사항이 있겠네요.

1. precision variants 추가
2. Foundation과 Extend버전에는 적용이 안된 precision옵션(LP,LDP) 추가
3. 배포판 별 특정 Precision(SPDP,LP,LDP)에 대해서 특정 버전 이후에만 적용되도록 변경


----------------------------------------------------------------------------
#TODO Write

3개를 별도의 PR로 요청하겠습니다.
먼저 ESI 배포판의 variants부터 수정하는 PR을 요청했습니다.
PR 사례1

Foundation 배포판에서는 1+2+3을 동시에 해결하는 PR을 요청했습니다.
PR 사례2

이런 과정은 사실 노가다에 가깝습니다. 이런 변화들은 새로운 버전이 릴리즈 됐을 때 맞춰서 적용하는 것이 가장 바람직합니다. 그래서 패키지 레시피에도 운영자가 필요한 이유라고 할 수 있겠습니다.
