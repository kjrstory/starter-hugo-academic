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
Open CFD 배포판의 variant


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
Openfoam Foundation 배포판의 variant

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
foam-extend 배포판 의 variant

 
역시 OpenCFD 배포판이 압도적으로 많습니다. 실제 배포판의 차이일 수도 있겠지만 제가 알고 있는 바로는 이 정도로 기능의 차이가 나지는 않는 걸로 알고 있고 단순히 레시피를 만드는 차이라고 보면 되겠습니다.

먼저 float32 옵션을 보면 single precision을 사용하는 옵션입니다. 여기서 precision은 시뮬레이션 결과의 소수점 자릿수를 결정하는 옵션으로 결과의 정밀도를 나타냅니다. single, double precision(SP,DP)이 있고 long double precision도 있습니다.

그런데 OpenCFD 배포판에서는 spdp란 variants도 있습니다. 이는 single precision과 double precision을 섞어 쓰는 것입니다. CFD 시뮬레이션도 내부에서는 수많은 수치해석 기법이 쓰이며 정확도가 중요한 부분에서는 double precision을 쓰고 덜 중요한 부분에서는 single을 쓰는 방법입니다. 그런데 이 옵션이 다른 배포판에서는 없네요. 이제 각 배포판의 실제 소스코드를 뒤져봐야 합니다. Openfoam의 설치 설정파일은 여러 개 있는데 가장 대표적인 것은 `[source폴더]/etc/bashrc` 입니다.  이 정보는 사실 Openfoam을 직접 컴파일해서 설치하지 않았다면 알기 어려운 정보이며 CFD에 대한 기본적인 지식도 있어야 합니다.

{{% callout info %}}
```
  # [WM_PRECISION_OPTION] - Floating-point precision:
  # = DP | SP | SPDP
  export WM_PRECISION_OPTION=DP
```
{{% /callout %}}

Precision 옵션은 반드시 3개중에 하나가 선택되어야 하는 옵션인데 레시피 코드를 보면 boolean 두 개로 이것을 처리하고 있었습니다. 사용자가 실수로 +spdp+float32 로 입력을 주면 임의로 SPDP로 정하는 건데 이런 정보를 사용자가 알 수가 없으므로 잘못된 코드라고 판단했습니다. boolean 2개 대신 list안에서 선택하는 변수로 바꿔줘야 합니다.

```python
  if "+spdp" in spec:
      self.precision_option = "SPDP"
  elif "+float32" in spec:
      self.precision_option = "SP"
```

이번에는 Foundation 배포판을 보면 SP, DP외에 LP란 옵션이 추가로 있습니다. 이건 Long double precision을 의미합니다. 
하지만 Spack에는 LP옵션이 반영이 되어있지는 않았습니다. 이것을 반영해야겠습니다.
그런데 LP 옵션을 추가하고 싶은데 과연 모든 버전에서 LP가 있었던 것일까 의심이 됩니다. 이럴 때는 릴리즈 또는 커밋 히스토리를 뒤져보거나 리포지토리의 Blame 기능을 써서 찾을 수 있습니다.

`WM_PRECISION_OPTION = SP | DP | LP`

아래와 같은 커밋을 찾았습니다. tag가 version6부터 있는 것으로 봐서 6 버전부터 적용이 되었군요.
[OpenFOAM: Added support for extended precision scalar](https://github.com/OpenFOAM/OpenFOAM-dev/commit/d82cc36c5af97e799a82fadf455e06d192ae1e65)


그런데 다시 돌아가 그럼 OpenCFD 배포판은 옛날 버전도 spdp가 될까요? 보통 sp와 dp는 기본적으로 지원하고 그 외의 옵션은 나중에 개발이 됩니다. ESI 배포판의 깃 리포지토리는 [여기](https://develop.openfoam.com/Development/openfoam)입니다. 깃헙이 아닌 깃랩(Gitlab)으로 구축했네요. 역시 마찬가지로 blame으로 찾아낼 수 있습니다.

[ENH: add primitives support for mixed precision](https://develop.openfoam.com/Development/openfoam/-/blob/46bc808261ef44cb29b512cb0c93acabdc09153a/etc/bashrc)

태그를 보면 1906버전부터 적용이 된 것을 알 수 있습니다. 이렇게 특정 버전만 적용되는 것이라면 반드시 레시피에 조건문을 달아야 합니다. 
그리고 옵션 설명에도 명시를 해서 사용자에게 혼선이 없도록 해야 합니다.

Foam-Extend도 살펴보겠습니다. Foam-Extend의 리포지토리는 소스포지네요. 

[Feature: Single precision and long double precision port](https://sourceforge.net/p/foam-extend/foam-extend-3.2/ci/6b022758d1b15a8d08718a78d3f68879e95bcf90)

`WM_PRECISION_OPTION = LDP | DP | SP`

여기는 LP란 이름 대신 LDP를 쓰고 3.2버전부터 적용이 되었네요.
정리를 하면 3가지 개선 사항이 있겠네요.

1. precision variants 추가
2. Foundation과 Extend버전에 적용이 안된 precision옵션(LP,LDP) 추가
3. 배포판 별 특정 Precision(SPDP,LP,LDP)에 대해서 특정 버전 이후에만 적용되도록 변경

## OpenCFD 배포판 Precision Variant 변경
먼저 OpenCFD 배포판부터 변경하기로 하였습니다.
아래 옵션들은 지우기로 합니다.
```python
    variant("float32", default=False, description="Use single-precision")
    variant("spdp", default=False, description="Use single/double mixed precision")
```

대신 precision이란 더 명확한 variant를 추가하기로 합니다. 디폴트 값은 dp이며 spdp는 1906이상 버전부터 가능하고, 중복이 허용이 안되게(multi=False) 설정하였습니다.
```python
    variant(
        "precision",
        default="dp",
        description="Precision option",
        values=("sp", "dp", conditional("spdp", when="@1906:")),
        multi=False,
    )
```

그리고 실제 이 variants에 따라 컴파일(또는 빌드)옵션이 달라지는 곳을 찾아 바꿔줍니다.

```python
        self.precision_option = "DP"  # <- precision= sp | dp | spdp
        ...
        # WM_PRECISION_OPTION
        if "precision=sp" in spec:
            self.precision_option = "SP"
        elif "precision=spdp" in spec:
            self.precision_option = "SPDP"
```

이렇게 레시피 파일을 변경한 후 아래 명령어로 테스트를 수행하였습니다.

그 후 PR을 하였습니다. 이 패키지에는 maintainer가 있었는데 다행히 큰 의견 없이 승인이 되었습니다. 보통 maintainer가 있는 패키지들은 spack의 주 개발자들도 그 사람들을 믿고 반영하는 듯 합니다.

## Foundation 배포판 Precision Variant 변경
다음으로 Open Foundation 배포판도 변경을 하였습니다.
역시 float32란 variants는 지우고 

```python
    variant(
        "precision",
        default="dp",
        description="Precision option",
        values=("sp", "dp", conditional("lp", when="@6:")),
        multi=False,
    )
```

그 후 이 옵션이 들어가는 부분을 빌드 옵션에서 바꿔저야 합니다. 이 부분이 OpenCFD하고 약간의 차이가 있습니다.
1편에서 설명을 했듯이 OpenCFD 배포판의 패키지레시피를 기반으로 Foundation 배포판의 패키지 레시피 파일이 만들어져있습니다.
Foundation 배포판의 레시피에는 OpenCFD 배포판의 클래스와 함수들을 그대로 갖고와서 사용하는 경우가 많습니다.
클래스끼리는 상속을 받아 사용하게 되는데 이 개념은 객체지향에 나오는 개념으로 관련 지식이 필요합니다.
상속을 받되 달라지는 부분만 바꿔서 사용하는데 객체지향개념과 파이썬에 대해 잘 모른다면 어려울수도 있는 부분입니다.
아래는 OpenfoamOrgArch 클래스는 OpenfoamArch 클래스(OpenCFD버전에서 사용되는 클래스)를 상속받고 있습니다.
생성자(`__init__`)에서는 먼저, 부모 클래스인 OpenfoamArch의 생성자를 호출하여 초기화합니다. 이를 위해 `super().__init__(spec, **kwargs)`를 사용합니다.
그리고 OpenfoamArch와 다른 precision옵션을 써주면 됩니다.

```python
class OpenfoamOrgArch(OpenfoamArch):
    """An openfoam-org variant of OpenfoamArch"""

    def __init__(self, spec, **kwargs):
        super().__init__(spec, **kwargs)
        if "precision=lp" in spec:
            self.precision_option = "LP"
        elif "precision=sp" in spec:
            self.precision_option = "SP"
        self.update_options()
```

역시 아래 명령어로 variants가 성공적으로 적용된것을 확인합니다.

[PR](https://github.com/spack/spack/pull/38746)을 하였고 커밋메세지에 OpenCFD의 PR도 첨부하였습니다. 이미 비슷한 PR 사례가 있다보니 이번에는 특별한 리뷰없이 바로 승인이 되었습니다.

---
Foam-Extend 배포판도 수정하고 싶었지만 이 버전은 지금 제 환경에서는 기본 설치조차 안됩니다. 레시피 파일을 계속 검토했는데도 방법을 찾지 못하여 이슈로 남기고 다른 개선사항을 찾아 보려고 합니다. 다른 기여가 있다면 또 다른 글을 작성하도록 하겠습니다.
