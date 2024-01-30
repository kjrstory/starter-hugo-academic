---
title: "OpenFOAM 설치 with Spack"
date: 2024-01-0708:00:00Z
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


## 사전 설치 

## Spack 설치
```bash
git clone -c feature.manyFiles=true https://github.com/spack/spack.git
. spack/share/spack/setup-env.sh
spack version
```



## 컴파일러 확인



## 패키지 정보 확인
`spack info` 명령으로 패키지에 대한 설명을 볼 수 있습니다.

```bash
spack info openfoam
```

{{< spoiler text="Result of `spack info openfoam`" >}}
```bash
  Package:   openfoam

  Description:
    OpenFOAM is a GPL-opensource C++ CFD-toolbox. This offering is supported
    by OpenCFD Ltd, producer and distributor of the OpenFOAM software via
    www.openfoam.com, and owner of the OPENFOAM trademark. OpenCFD Ltd has
    been developing and releasing OpenFOAM since its debut in 2004.

  Homepage: https://www.openfoam.com/

  Preferred version:
    2206           https://sourceforge.net/projects/openfoam/files/v2206/OpenFOAM-v2206.tgz

  Safe versions:
    develop        [git] https://develop.openfoam.com/Development/openfoam.git on branch develop
    master         [git] https://develop.openfoam.com/Development/openfoam.git on branch master
    2206           https://sourceforge.net/projects/openfoam/files/v2206/OpenFOAM-v2206.tgz
    2112_220610    https://sourceforge.net/projects/openfoam/files/v2112/OpenFOAM-v2112_220610.tgz
    2112           https://sourceforge.net/projects/openfoam/files/v2112/OpenFOAM-v2112.tgz
    2106_220610    https://sourceforge.net/projects/openfoam/files/v2106/OpenFOAM-v2106_220610.tgz
    2106_211215    https://sourceforge.net/projects/openfoam/files/v2106/OpenFOAM-v2106_211215.tgz
    2106           https://sourceforge.net/projects/openfoam/files/v2106/OpenFOAM-v2106.tgz
    2012_220610    https://sourceforge.net/projects/openfoam/files/v2012/OpenFOAM-v2012_220610.tgz
    2012_210414    https://sourceforge.net/projects/openfoam/files/v2012/OpenFOAM-v2012_210414.tgz
    2012           https://sourceforge.net/projects/openfoam/files/v2012/OpenFOAM-v2012.tgz
    2006_220610    https://sourceforge.net/projects/openfoam/files/v2006/OpenFOAM-v2006_220610.tgz
    2006_201012    https://sourceforge.net/projects/openfoam/files/v2006/OpenFOAM-v2006_201012.tgz
    2006           https://sourceforge.net/projects/openfoam/files/v2006/OpenFOAM-v2006.tgz
    1912_220610    https://sourceforge.net/projects/openfoam/files/v1912/OpenFOAM-v1912_220610.tgz
    1912_200506    https://sourceforge.net/projects/openfoam/files/v1912/OpenFOAM-v1912_200506.tgz
    1912_200403    https://sourceforge.net/projects/openfoam/files/v1912/OpenFOAM-v1912_200403.tgz
    1912           https://sourceforge.net/projects/openfoam/files/v1912/OpenFOAM-v1912.tgz
    1906_200312    https://sourceforge.net/projects/openfoam/files/v1906/OpenFOAM-v1906_200312.tgz
    1906_191103    https://sourceforge.net/projects/openfoam/files/v1906/OpenFOAM-v1906_191103.tgz
    1906           https://sourceforge.net/projects/openfoam/files/v1906/OpenFOAM-v1906.tgz
    1812_200312    https://sourceforge.net/projects/openfoam/files/v1812/OpenFOAM-v1812_200312.tgz
    1812_191001    https://sourceforge.net/projects/openfoam/files/v1812/OpenFOAM-v1812_191001.tgz
    1812_190531    https://sourceforge.net/projects/openfoam/files/v1812/OpenFOAM-v1812_190531.tgz
    1812           https://sourceforge.net/projects/openfoam/files/v1812/OpenFOAM-v1812.tgz
    1806           https://sourceforge.net/projects/openfoam/files/v1806/OpenFOAM-v1806.tgz
    1712           https://sourceforge.net/projects/openfoam/files/v1712/OpenFOAM-v1712.tgz
    1706           https://sourceforge.net/projects/openfoam/files/v1706/OpenFOAM-v1706.tgz
    1612           https://sourceforge.net/projects/openfoam/files/v1612+/OpenFOAM-v1612+.tgz

  Deprecated versions:
    None

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

  Build Dependencies:
    adios2  boost  cgal  cmake  fftw-api  flex  kahip  m4  metis  mpi  paraview  parmgridgen  scotch  vtk  zlib  zoltan

  Link Dependencies:
    adios2  boost  cgal  fftw-api  flex  kahip  metis  mpi  paraview  scotch  vtk  zlib  zoltan

  Run Dependencies:
    None
```    
{{< /spoiler >}}


Openfoam 패키지에 대한 옵션들이 기술되어 있습니다. 버전 1612부터 2206까지 선택할 수 있고 다양한 의존 패키지(Dependencies) 및 설치 옵션(Variants)들이 기술되어 있습니다. 

설치 하기 전 `spack spec`명령을 통해 실제 어떤 패키지들이 설치될지 알 수 있습니다.

```bash
spack spec openfoam
```

{{< spoiler text="Result of `spack spec openfoam`" >}}
```bash
  Input spec
  --------------------------------
  openfoam

  Concretized
  --------------------------------
  openfoam@=2206%gcc@=9.4.0~float32~int64~kahip~knl~metis~mgridgen~paraview+scotch+source~spdp~vtk~zoltan build_system=generic arch=linux-ubuntu20.04-cascadelake
      ^adios2@=2.9.0%gcc@=9.4.0+blosc+bzip2~cuda~dataspaces~fortran~hdf5~ipo~libpressio+mpi~pic+png~python+ssc+sst+sz+zfp build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
          ^bzip2@=1.0.8%gcc@=9.4.0~debug~pic+shared build_system=generic arch=linux-ubuntu20.04-cascadelake
          ^c-blosc@=1.21.2%gcc@=9.4.0+avx2~ipo build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
              ^lz4@=1.9.4%gcc@=9.4.0 build_system=makefile libs=shared,static arch=linux-ubuntu20.04-cascadelake
              ^snappy@=1.1.10%gcc@=9.4.0~ipo+pic+shared build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
          ^gmake@=4.4.1%gcc@=9.4.0~guile build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^libfabric@=1.18.0%gcc@=9.4.0~debug~kdreg build_system=autotools fabrics=sockets,tcp,udp arch=linux-ubuntu20.04-cascadelake
          ^libffi@=3.4.4%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^libpng@=1.6.39%gcc@=9.4.0~ipo build_system=cmake build_type=Release generator=make libs=shared,static arch=linux-ubuntu20.04-cascadelake
          ^pkgconf@=1.9.5%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^sz@=2.1.12.2%gcc@=9.4.0~fortran~hdf5~ipo~netcdf~pastri~python~random_access+shared~stats~time_compression build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
          ^zfp@=0.5.5%gcc@=9.4.0~aligned~c~cuda~fasthash~fortran~ipo~openmp~profile~python+shared~strided~twoway+utilities bsws=64 build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
      ^boost@=1.82.0%gcc@=9.4.0+atomic+chrono~clanglibcpp~container~context~contract~coroutine+date_time~debug+exception~fiber+filesystem+graph~graph_parallel~icu+iostreams~json+locale+log+math~mpi+multithreaded~nowide~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded~stacktrace+system~taggedlayout+test+thread+timer~type_erasure~versionedlayout+wave build_system=generic cxxstd=98 patches=a440f96,a7c807f visibility=hidden arch=linux-ubuntu20.04-cascadelake
          ^xz@=5.4.1%gcc@=9.4.0~pic build_system=autotools libs=shared,static arch=linux-ubuntu20.04-cascadelake
          ^zstd@=1.5.5%gcc@=9.4.0+programs build_system=makefile compression=none libs=shared,static arch=linux-ubuntu20.04-cascadelake
      ^cgal@=4.13%gcc@=9.4.0~core~demos+eigen~header_only~imageio~ipo+shared build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
          ^eigen@=3.4.0%gcc@=9.4.0~ipo build_system=cmake build_type=RelWithDebInfo generator=make arch=linux-ubuntu20.04-cascadelake
          ^gmp@=6.2.1%gcc@=9.4.0+cxx build_system=autotools libs=shared,static patches=69ad2e2 arch=linux-ubuntu20.04-cascadelake
          ^mpfr@=4.2.0%gcc@=9.4.0 build_system=autotools libs=shared,static arch=linux-ubuntu20.04-cascadelake
              ^autoconf-archive@=2023.02.20%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^texinfo@=7.0.3%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
      ^cmake@=3.26.3%gcc@=9.4.0~doc+ncurses+ownlibs~qt build_system=generic build_type=Release arch=linux-ubuntu20.04-cascadelake
          ^ncurses@=6.4%gcc@=9.4.0~symlinks+termlib abi=none build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^openssl@=1.1.1t%gcc@=9.4.0~docs~shared build_system=generic certs=mozilla arch=linux-ubuntu20.04-cascadelake
              ^ca-certificates-mozilla@=2023-01-10%gcc@=9.4.0 build_system=generic arch=linux-ubuntu20.04-cascadelake
      ^fftw@=3.3.10%gcc@=9.4.0+mpi~openmp~pfft_patches build_system=autotools precision=double,float arch=linux-ubuntu20.04-cascadelake
      ^flex@=2.6.4%gcc@=9.4.0+lex~nls build_system=autotools patches=f8b85a0 arch=linux-ubuntu20.04-cascadelake
          ^autoconf@=2.69%gcc@=9.4.0 build_system=autotools patches=35c4492,7793209,a49dd5b arch=linux-ubuntu20.04-cascadelake
          ^automake@=1.16.5%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^bison@=3.8.2%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^diffutils@=3.9%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^libiconv@=1.17%gcc@=9.4.0 build_system=autotools libs=shared,static arch=linux-ubuntu20.04-cascadelake
          ^findutils@=4.9.0%gcc@=9.4.0 build_system=autotools patches=440b954 arch=linux-ubuntu20.04-cascadelake
          ^gettext@=0.21.1%gcc@=9.4.0+bzip2+curses+git~libunistring+libxml2+tar+xz build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^libxml2@=2.10.3%gcc@=9.4.0~python build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^tar@=1.34%gcc@=9.4.0 build_system=autotools zip=pigz arch=linux-ubuntu20.04-cascadelake
                  ^pigz@=2.7%gcc@=9.4.0 build_system=makefile arch=linux-ubuntu20.04-cascadelake
          ^help2man@=1.49.3%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^libtool@=2.4.7%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
      ^m4@=1.4.19%gcc@=9.4.0+sigsegv build_system=autotools patches=9dc5fbd,bfdffa7 arch=linux-ubuntu20.04-cascadelake
          ^libsigsegv@=2.14%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
      ^openmpi@=4.1.5%gcc@=9.4.0~atomics~cuda~cxx~cxx_exceptions~gpfs~internal-hwloc~java~legacylaunchers~lustre~memchecker~orterunprefix+romio+rsh~singularity+static+vt+wrapper-rpath build_system=autotools fabrics=none schedulers=none arch=linux-ubuntu20.04-cascadelake
          ^hwloc@=2.9.1%gcc@=9.4.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml~oneapi-level-zero~opencl+pci~rocm build_system=autotools libs=shared,static arch=linux-ubuntu20.04-cascadelake
              ^libpciaccess@=0.17%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
                  ^util-macros@=1.19.3%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^numactl@=2.0.14%gcc@=9.4.0 build_system=autotools patches=4e1d78c,62fc8a8,ff37630 arch=linux-ubuntu20.04-cascadelake
          ^openssh@=9.3p1%gcc@=9.4.0+gssapi build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^krb5@=1.20.1%gcc@=9.4.0+shared build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^libedit@=3.1-20210216%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^libxcrypt@=4.4.33%gcc@=9.4.0~obsolete_api build_system=autotools arch=linux-ubuntu20.04-cascadelake
          ^perl@=5.36.0%gcc@=9.4.0+cpanm+open+shared+threads build_system=generic arch=linux-ubuntu20.04-cascadelake
              ^berkeley-db@=18.1.40%gcc@=9.4.0+cxx~docs+stl build_system=autotools patches=26090f4,b231fcc arch=linux-ubuntu20.04-cascadelake
              ^gdbm@=1.23%gcc@=9.4.0 build_system=autotools arch=linux-ubuntu20.04-cascadelake
                  ^readline@=8.2%gcc@=9.4.0 build_system=autotools patches=bbf97f1 arch=linux-ubuntu20.04-cascadelake
          ^pmix@=4.2.3%gcc@=9.4.0~docs+pmi_backwards_compatibility~python~restful build_system=autotools arch=linux-ubuntu20.04-cascadelake
              ^libevent@=2.1.12%gcc@=9.4.0+openssl build_system=autotools arch=linux-ubuntu20.04-cascadelake
      ^scotch@=7.0.3%gcc@=9.4.0+compression~esmumps~int64~ipo~metis+mpi+shared build_system=cmake build_type=Release generator=make arch=linux-ubuntu20.04-cascadelake
      ^zlib@=1.2.13%gcc@=9.4.0+optimize+pic+shared build_system=makefile arch=linux-ubuntu20.04-cascadelake
```
{{< /spoiler >}}



## OpeoFOAM 설치


설치 설정 시 각 의존 패키지들의 설정과 옵션 및 컴파일러를 선택할 수 있습니다. 아래 제공된 명령어는 OpenFOAM 버전 2206을 특정 옵션과 선택한 컴파일러로 설치하는 예시를 보여줍니다.

```bash
  spack install openfoam@2206~float32~int64~kahip~knl~metis~mgridgen~paraview+scotch+source~spdp~vtk~zoltan build_system=generic %gcc@9.4.0
```

이 명령은 모든 옵션을 쓴 것이라 복잡해 보이는데 defalut옵션과 동일하다면 그 옵션을 생략할 수 있고 보통 설치할 패키지의 버전과 컴파일러를 선택하여 설치하게 됩니다.

```bash
  spack install openfoam@2206 %gcc@9.4.0
```




## OpenFOAM 실행



