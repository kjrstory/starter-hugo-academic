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
  filename: fds_example.png
  focal_point: Smart
  preview_only: false
---


 SU2는 스탠포트 대학교에 시작한 오픈소스 CFD SW입니다. 저는 SU2에 관심이 많아서 이 논문도 썼었고 저논문 도 썼썽썻습니다.
논문에 쓰는것에 그치지 않고 실무에도 적응을 하여록 했는데요. 그 당시에 Low Fidelity 해석에 사용했던것으로 기억합니다.
Spack에서도 SU2ㅡㄴ 이미 등록이 되어있었습니다.

ㅇ무려 제가 운영자로 신청을 했엇습니다.

단 그 동안 시간 부족을 핑계로 업데이트를 하지 못하였습니다.

dlqjs
이번에 다시 업데이트를 하기로 했습니다.
SU2는 특이하게 meson이란 빌드시스템을 사용합니다. meson은 파이썬으로 구성된 빌드시스템으로 주저리 설명

SU2의 빌드 페이지를 보고 make.build 파일을 보면서 많은 부분을 수정햇습니다.

사실 기존의 S2 패키지는 거의 기능이 없다 ㅣ피한것이었습니다.

한지기 걸리는것은 특정 모듈은 다른 깃헙에서 서브모듈기능을 사용합니다. 

깃헙에는 서브모듈 기능이 잉써어서 다른 리포지토리와 연결할 수 있습니다.
Spack에서도 같은 방법으로 다른 리포지토리를 다운받는 방법을 쓸지 고민이었습니다.

일단 기존 방법대로 하되 차후 리포지티로를 바꾸는 것으로 햇습니다.


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


```python
    depends_on("meson@0.61.1:", type=("build"))
    depends_on("python@3:", type=("build", "run"))
    depends_on("zlib")
    depends_on("pkg-config")
    depends_on("mpi", when="+mpi")
    depends_on("swig", type="build", when="+pywrapper")
    depends_on("py-mpi4py", when="+pywrapper")
    depends_on("intel-oneapi-mkl", when="+mkl")
    depends_on("openblas", when="+openblas ~mkl")
    depends_on("cmake", type="build", when="+mpp")
```


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
```


메슨 관련된 이이야기


지적했던 상황



