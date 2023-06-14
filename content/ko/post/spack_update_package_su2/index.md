---
title: Spack에서 SU2 패키지 업데이트 사례
date: 2023-06-14T07:23:41.994Z
draft: false
featured: false
authors:
  - admin
tags:
  - spack
  - su2
categories:
  - HPC
image:
  filename: ''
  focal_point: Smart
  preview_only: false
---

 [SU2](https://su2code.github.io)는 스탠포드 대학교의 항공우주공학과 연구실에서 개발한 오픈소스 CFD SW입니다. 저는 2010년대 초반부터 SU2에 관심이 많아서 [논문](ko/publication/jong-rok-kim-2017-cfd-analysis-efdcfd)에도 SU2를 이용하여 해석을 했었습니다. Spack에도 2년전에 업데이트를 했었고 Maintainer로 지원을 했습니다. Spack에서는 모든 패키지마다 [maintainer](https://spack.readthedocs.io/en/latest/packaging_guide.html#maintainers) 제도를 운영하고 있습니다. maintainer가 되면 누군가 그 패키지 레시피 파일에 PR이나 Issue를 남길 경우 승인하고 review를 할 수 있습니다. 하지만 SU2의 maintainer가 되고도 그동안 업데이트를 하지 못했습니다. 2년전에 볼 때는 보완하기 어려웠던 것이 다시 보니 어떻게 고쳐야 될지 보이기 시작했고 고칠 수 있었습니다. 그동안 알게 모르게 실력이 성장한 것 같습니다.  
 
# 1.Pull Request 첫 
기존 레시피에는 아무런 variant가 없었습니다. 실제 SU2에 variant가 없었던 것은 아니고 어떻게 적용하는지를 몰랐기 때문입니다. 이번에 다시 [SU2빌드 방법](https://su2code.github.io/docs_v7/Build-SU2-Linux-MacOS/)을 읽어보고 variant를 적용하기로 하였습니다.

SU2에는 다음과 같은 옵셥을 지원합니다.

{{% callout quote %}}
| Option | Default value | Description |
|---| --- | --- |
| `-Denable-autodiff`  | `false`   |   enable AD (reverse) support (needed for discrete adjoint solver)  |
| `-Denable-directdiff` | `false`     |  enable AD (forward) support |
| `-Denable-pywrapper` | `false`      |    enable Python wrapper support|
| `-Dwith-mpi`       | `auto` |   Set dependency mode for MPI (`auto`,`enabled`,`disabled`)  |
| `-Dwith-omp`       | `false` |  enable MPI+Threads support (experimental) |
| `-Denable-cgns`     | `true`    |       enable CGNS support           |        
| `-Denable-tecio`    |  `true`       |    enable TECIO support         |
| `-Denable-mkl`      |  `false`      |    enable Intel MKL support     |
| `-Denable-openblas` |  `false`      |    enable OpenBLAS support      |
| `-Denable-pastix`   |  `false`      |    enable PaStiX support        |
| `-Denable-mpp`   |  `false`      |    enable Mutation++ support        |
| `-Denable-mixedprec` | `false`      |    enable the use of single precision on linear solvers and preconditioners |
{{% /callout %}}

이것을 패키지 레시피에 모두 적용시켰습니다.
```python
    variant("mpi", default=False, description="enable MPI support")
    variant("openmp", default=False, description="enable OpenMP support")
    variant("tecio", default=True, description="enable TECIO support")
    variant("cgns", default=True, description="enable CGNS support")
    variant("autodiff", default=False, description="enable AD(reverse) support")
    variant("directdiff", default=False, description="enable AD(forward) support")
    variant("pywrapper", default=False, description="enable Python wrapper support")
    variant("mkl", default=False, description="enable Intel MKL support")
    variant("openblas", default=False, description="enable OpenBLAS support")
    variant("mpp", default=False, description="enable Mutation++ support")
    variant(
        "mixedprec",
        default=False,
        description="use single precision floating point arithmetic for sparse algebra",
    )

```    

그 후 SU2의 빌드 문서를 계속 읽어보면서 dependency를 설정합니다.
```python
    depends_on("meson@0.61.1:", type=("build"))
    depends_on("pkg-config")
    depends_on("mpi", when="+mpi")
    depends_on("swig", type="build", when="+pywrapper")
    depends_on("py-mpi4py", when="+pywrapper")
    depends_on("intel-oneapi-mkl", when="+mkl")
    depends_on("openblas", when="+openblas ~mkl")
    depends_on("cmake", type="build", when="+mpp")
```
이제 중요한 부분은 Variant의 내용을 어떻게 적용시킬지입니다. 그걸 하기 위해서는 SU2의 빌드시스템인 [Meson](https://mesonbuild.com)에 대해서 알아야 합니다. 
Meson은 비교적 최신의 빌드시스템으로 Python으로 되어있습니다. Meson은 [Ninja](https://ninja-build.org)와 커플되어 빌드하는 도구입니다. 
CMake보다 쉽고 빌드가 빠르다고 하는 빌드시스템입니다.
Spack의 [Meson 빌드](https://spack.readthedocs.io/en/latest/build_systems/mesonpackage.html) 설명도 읽어봐야 합니다.
결국 SU2빌드 시 필요했던 옵션들은 Meson의 argument이고 이것은 meson_args함수로 적용시킬 수 있습니다. 아래와 같이 코드를 작성하였습니다.

```python
    def meson_args(self):
        args = [
            "-Dwith-omp={}".format("+openmp" in self.spec),
            "-Denable-tecio={}".format("+tecio" in self.spec),
            "-Denable-cgns={}".format("+cgns" in self.spec),
            "-Denable-autodiff={}".format("+autodiff" in self.spec),
            "-Denable-directdiff={}".format("+direcdiff" in self.spec),
            "-Denable-pywrapper={}".format("+pywrapper" in self.spec),
            "-Denable-mkl={}".format("+mkl" in self.spec),
            "-Denable-openblas={}".format("+openblas" in self.spec),
            "-Denable-mpp={}".format("+mpp" in self.spec),
            "-Denable-mixedprec={}".format("+midexprec" in self.spec),
        ]

        if "+mkl" in self.spec:
            args.append("-Dmkl_root=" + self.spec["intel-oneapi-mkl"].prefix)

        if "+mpi" in self.spec:
            args.append("-Dwith-mpi=enabled")
        else:
            args.append("-Dwith-mpi=disabled")
            
        return args
```

# 2.테스트



# 3.리뷰1 Meson
SU2의 7.4.0부터 7.5.1버전은 Meson의 버전을 0.61.1로 고정시켰습니다.
그런데 Spack의 Meson패키지는 0.61.1버전이 등록되어 있지 않아 등록하는 [PR](https://github.com/spack/spack/pull/37770)을 요청했습니다.
eli-schwartz님에게 부정적인 리뷰를 받았습니다.
사실 부정적인 리뷰나 PR 거절을 받게 되면 처음에는 기분이 상하는데요. 하지만 이유없는 거절은 없고 다시 차분하게 리뷰를 읽어보고 대응해야 합니다.

{{% callout quote %}}
I disagree with this change because it is an old and buggy version of meson that has later patch/bugfix releases within the same minor release series, which spack already packages.

Furthermore, SU2 no longer needs this due to su2code/SU2#1951
{{% /callout %}}

0.61.1은 현재 버전에 비해 너무 낮고 SU2에서도 지원하지 않을거라고 하네요.
더 찾아보니 Meson의 최신 버전은 1.1.0이라 0.61.1버전이 너무 낮았었고요. 저 SU2의 [PR](https://github.com/su2code/SU2/pull/1951)을 또 읽어봐야 합니다.
PR을 읽어보면 Meson의 버전 제한을 0.61.1 고정에서 이상으로 바뀌고 몇 가지 빌드 옵션들도 손을 보는 것 같습니다.
그런데 이 PR은 SU2의 develop브랜치의 PR이었고 eli-schwartz님이 이 PR의 리뷰어라 알고 있었습니다.
배포된 버전에서는 아직 반영이 안된 PR이었습니다.
여기서 어떻게 할 지 고민을 하였는데 결국 Meson의 버전을 제한할 필요는 없다고 생각했습니다. 
SU2 설치 시 Meson버전이 0.61.1이 아니면 에러가 나게 되어있는데 체크하는 부분을 없애는 패치를 하기로 하였습니다.
다시 Meson 1.1.0버전으로 제대로 테스트 되는지 확인을 하였습니다.
정상적으로 설치가 되는 것을 확인하고 패치를 적용하기로 확정했습니다.

패치를 적용하고 의견을 남겼습니다.
{{% callout quote %}}
I agree with @eli-schwartz opinion.
However, currently, this commit(su2code/SU2#1951) has not been applied to any released versions of SU2.
As I have tested, the restriction was implemented in a specific range of versions (@7.4.0:7.5.1) of SU2, but I have successfully installed SU2 with a different version of meson(1.1.0).
In light of this, I have created the patch and submitted a pull request (#37767).
I would greatly appreciate your feedback on the meson version included in this PR.
{{% /callout %}}

Meson PR에는 몇 가지 코멘트가 더 달렸습니다. adamjstewart님은 옛날 버전을 적용하는 것이 big deal이 아닌것 같다고 하였는데 eli-schwartz님이 다시 자세히 코멘트를 달았습니다. 

{{% callout quote %}}
I'd be less concerned about adding an old version if it were the latest bugfix release of that old version. :D

Spack doesn't currently package the .5 bugfix release for this old minor release. I'd have no objections to adding that -- it fixes bugs that may be encountered by people who have made the judgment call that their needs are best served by sticking to that old version and avoiding sweeping changes, deprecations, and whatnot, from newer versions.

As an upstream meson maintainer, I want to push people to always upgrade to the latest bugfix release, if they aren't in a position to upgrade to the latest feature release. That's why we have 0.61.x as an LTS in the first place.

But for software that locks itself to an exact bugfix release, the best solution is to backport an upstream change that removes the locked dependency. That's what the other PR does, so this PR won't be needed even for SU2 anymore. :)
{{% /callout %}}

eli-schwartz님은 meson빌드시스템도 개발하시는 분이었고 어찌보면 버전 고정의 한 줄이었는데도 이렇게 상세히 피드백을 받을 수 있었습니다. 
여러 오픈소스 개발자에게 배우고 의견 교환할 수 있다는 것이 PR에서 얻는 배움인 것 같습니다. 
