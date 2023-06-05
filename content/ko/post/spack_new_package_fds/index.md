---
title: Spack에서 신규 패키지(FDS) 생성 사례
date: 2023-06-05T07:23:41.994Z
draft: false
featured: false
authors:
  - admin
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
 이 글은 Spack에 오픈소스 CFD SW 중 화재 시뮬레이션에 특화된 [FDS](https://pages.nist.gov/fds-smv/)(Fire Dynamics Simulator)를 등록한 사례에 대한 글입니다. FDS는 [NIST](https://www.nist.gov/)(미국 국립 표준 기술연구소, National Institute of Standards and Technology)에서 개발한 SW입니다. LES(Large Eddy Simulation)을 사용하고 연기(smoke)의 열전달해석이 가능합니다. 후처리는 자체적으로 개발한 Smokeview란 SW를 사용합니다. 2000년에 정식으로 출시되었으며 화재 시뮬레이션 분야에서는 명성이 있는 오픈소스로 보입니다.

## 1. 신규 패키지 생성

Spack에서 패키지 생성하는 방법은 두가지입니다. 하나는 Spack CLI 명령을 이용하는 방법이고 하나는 직접 파이썬 파일을 생성하는 방법입니다. 첫번째 방법은 아래와 같이 `create` 명령으로 생성합니다. 소스코드의 다운로드 주소는 패키지에 맞게 바꿔줘야겠죠? 일단 깃헙의 릴리즈 주소를 이용하였습니다

```bash
spack create https://github.com/firemodels/fds/archive/refs/tags/FDS-6.8.0.tar.gz
```

결과를 더 읽어보면(마지막 줄) spack폴더의 어떤 복잡한 경로 밑에 package.py 파일이 만들어진 것을 알 수 있습니다. 신규 패키지를 생성하는 두번째 방법은 이 package.py 파일을 직접 만드는 것입니다. 패키지 레시피 작성에 익숙해졌다면 create명령을 이용하여 기본 템플릿 파일에서 시작하는것보다 기존에 다른 사람들이 만들었던 레시피 파일을 참고하는 것이 더 유용할 것입니다.

{{% callout info %}}
 ==> Using specified package name: ‘fds’

 ==> Found 1 version of fds:
  
  6.8.0  https://github.com/firemodels/fds/archive/refs/tags/FDS-6.8.0.tar.gz
  
 ==> Fetching <https://github.com/firemodels/fds/archive/refs/tags/FDS-6.8.0.tar.gz>

 ==> Warning: Unable to detect a build system. Using a generic package template.

 ==> Created template for fds package

 ==> Created package file: /root/spack/var/spack/repos/builtin/packages/fds/package.py
{{% /callout %}}

## 2 빌드시스템 선택

먼저 [빌드시스템](https://en.wikipedia.org/wiki/Build_automation)에 대한 이해가 있어야 합니다. 대표적인 빌드시스템으로 Make, CMake, Maven, Meson, Autotools등이 있습니다. 대부분의 큰 오픈소스들은 빌드시스템을 이용하고 있기 때문에 Spack 패키지 레시피를 만들기 위해서는 빌드시스템에 대한 이해가 필수입니다. FDS는 어떤 빌드시스템을 이용하고 있을까요? 앞단락에서 `spack create` 명령으로는 찾을 수 없다고 했습니다. FDS는 깃헙 위키를 운영하고 있어 정보를 얻을 수 있습니다. 이 중 [FDS Compilation](https://github.com/firemodels/fds/wiki/FDS-Compilation)글을 보면 빌드하는 방법을 알 수 있습니다. 아키텍쳐마다 빌드 디렉토리가 따로 있고 자체 bash 스크립트(`make_fds.sh`)를 쓰도록 되어있습니다.

{{% callout info %}}

cd to the directory in the fds repository called **Build**.

cd to the appropriate directory within Build, such as **ompi_intel_linux** for the Intel compiler and Open MPI libraries under linux

Type make_fds.bat or **make_fds.sh**, depending on your OS.

{{% /callout %}}

`make_fds.sh` 파일을 보면 `make` 명령어와 `makefile`등을 이용하는 것을 보아 Makefile 빌드 시스템을 이용하는 것을 알 수 있습니다.

```bash
#!/bin/bash
dir=`pwd`
target=${dir##*/}

echo Building $target
make -j4 VPATH="../../Source" -f ../makefile $target
```

결국 이 `make_fds.sh`파일이 하는 역할은 빌드 시 적절한 변수를 지정해주는 수준의 역할을 하는것으로 판단했습니다. 여기서 선택을 하나 해야 되는데요. `make_fds.sh`파일을 fds에서 자체적으로 만든 빌드 파일로 보고 Spack에서 `make_fds.sh`파일을 실행하게 할 것인지 아니면 `make_fds.sh`파일을 사용하지 않고 Makefile 빌드 시스템을 이용할것인지요. 저는 가급적 Spack의 함수들을 많이 사용하는것이 좋다고 판단했기 때문에 Makefile 빌드 시스템을 이용하기로 했습니다. 즉 `make_fds.sh` 파일이 하는 역할을 spack의 패키지 레시피가 대신하게 됩니다. 참고로 `spack create` 명령으로 빌드 시스템을 못찾았던 이유는 `makefile`이 소스코드의 최상위 폴더에 있지 않았기 때문입니다.

먼저 Spack에서 제공하는 [Makefile 페이지](https://spack.readthedocs.io/en/latest/build_systems/makefilepackage.html)을 읽어보고 진행합니다. Makefile 빌드시스템은 edit, build, install 의 과정을 거치게 됩니다. edit는 makefile을 수정하는 과정, build는 컴파일하는 과정이고 install은 컴파일해서 만들어진 실행파일을 복사/링크 하는것입니다.

### 2.1 edit

edit는 makefile을 수정하거나 필요한 환경변수를 입력하는 과정입니다. 예시로 본 `make_fds.sh` 파일외에 다른 폴더에 있던 `make_fds.sh` 파일을 살펴보면 intel 컴파일러 선택시 `INTEL_IFORT` 변수를 넣어주는 과정이 있습니다. 디폴트 값은 `ifort`값이 들어가 있습니다. 이건 포트란을 알고 있으면 눈치를 챌 수 있는데요. `ifort`는 인텔 포트란의 명령어 이름입니다. 그런데 인텔이 oneapi를 출시하면서 명령어를 `ifx`로 바꾸어버렸습니다. 그래서 이 부분을 변수에서 지정해줘야 합니다.

`makefile.filter` 부분은 makefile을 바꿔주는 부분으로 리눅스 명령어중 `sed`와 사용법이 유사합니다. make_fds.sh 파일을 실행하는것과 makefile을 바로 make하는것과 경로가 차이가 납니다. 그래서 이와 관련된 경로를 맞게 바꿔주는 것입니다.

```python
    def edit(self, spec, prefix):
        env["MKL_ROOT"] = self.spec["mkl"].prefix
        if spec.compiler.name == "oneapi":
            env["INTEL_IFORT"] = "ifx"
        makefile = FileFilter("Build/makefile")
        makefile.filter(r"\.\./Scripts", "./Scripts")
        makefile.filter(r"\.\.\\Scripts", ".\\Scripts")
```

{{% callout warning %}}
`MKL_ROOT` 환경변수를 입력해주는 부분은 검토가 필요합니다. FDS에서 `MKL_ROOT`대신 `MKLROOT` 변수를 쓰는것으로 확인이 됩니다. MKL과 ROOT사이에 ‘_‘가 없어야 합니다. 틀렸던 이유는 위키의 내용과 실제 makefile이 다르기 때문입니다. `MKLROOT`는 spack에서 oneapi 설치 시 환경변수로 들어가게 되는데 특정 옵션이 켜진 상태에서만 들어가게 됩니다. 이 옵션은 디폴트가 켜져있는 상태이긴 하지만 사용자가 끌수도 있으므로 확실한 정의가 필요합니다.
{{% /callout %}}

### 2.2 build

make 명령어는 아래와 같이 옵션과 타겟으로 나누어집니다. 예시로 보여드린 `make_fds.sh`의 make행에서 -j, -f는 옵션이고 `$target`이 `[target]`에 해당합니다.

> make –help Usage: make \[options] \[target] …

\-j4는 빌드 시 4개의 쓰레드를 사용하는 것입니다. Spack에는 쓰레드가 알아서 지정이 되고 CPU가 충분할 시 더 많은 쓰레드로 지정이 됩니다. VPATH는 makefile에 VPATH란 변수를 넣어주는것입니다. -f는 makefile의 경로를 지정해주는 것입니다. makefile이 있는 디렉토리에서 빌드할 것이기 때문에 위치는 ../makefile이 아닌 ./makefile입니다. ../는 상위 폴더, ./는 현재 폴더를 의미합니다. 현재 폴더에서 빌드한다면 굳이 -f 옵션은 필요없습니다. 다음 부분은 target인데 여기에는 OS, Compiler, MPI의 값이 들어가게 됩니다. 기존 FDS 방법은 디렉토리가 타겟값이 됩니다. 저는 기존 FDS 방법 처럼 특정 폴더에 들어가지 않고 빌드를 하니 target값을 spack에서 만들어줘야 합니다. property decorator와 build_targets함수로 아래와 같이 target값을 지정할 수 있습니다.

```python
    @property
    def build_targets(self):
        spec = self.spec
        mpi_mapping = {"openmpi": "ompi", "intel-oneapi-mpi": "impi", "intel-mpi": "impi"}
        compiler_mapping = {"gcc": "gnu", "oneapi": "intel", "intel": "intel"}
        platform_mapping = {"linux": "linux", "darwin": "osx"}
        mpi_prefix = mpi_mapping[spec["mpi"].name]
        compiler_prefix = compiler_mapping[spec.compiler.name]
        platform_prefix = platform_mapping[spec.architecture.platform]
        return ["{}_{}_{}".format(mpi_prefix, compiler_prefix, platform_prefix)]
```

리턴해주는 값은 `["{}_{}_{}".format(mpi_prefix, compiler_prefix, platform_prefix)]` 으로 mpi, 컴파일러, 플랫폼(OS)값을 주게 됩니다. spack의 명칭과 FDS의 명칭이 다르니 mapping해주는 딕셔너리를 만들었습니다.

즉 기존에 Build 디렉토리 밑에 타겟값으로 쓰일 디렉토리로 들어가 make_fds.sh 파일로 빌드 하는 방법을 Build 디렉토리에서 바로 빌드하되 target값은 spack에서 받는 값으로 대체한것입니다.

### 2.3 install

기존 FDS는 Build/ompi_gnu_liunx처럼 빌드한 폴더에 실행파일이 있게되고 그 폴더를 환경변수 PATH에 넣는 방식을 취합니다. 그런데 저는 새로운 빌드 폴더를 쓰기로 했으므로 실행 파일이 설치 될 곳이 필요하고요. 간단히 FDS설치 폴더의 bin파일에 넣으려고 합니다. 아래와 같이 bin 폴더를 만들고 필요한 파일들을 bin폴더에 넣는 install 함수를 만들었습니다.

```python
   def install(self, spec, prefix):
        mkdirp(prefix.bin)
        with working_dir(self.build_directory):
            install("*.mod", prefix.bin)
            install("*.o", prefix.bin)
            install("fds_" + self.build_targets[0], prefix.bin + "/fds")
```

{{% callout warning %}}
GNU 포트란과 open mpi로 컴파일시 모듈 파일(mod)과 오브젝트 파일(o)을 같이 bin 폴더로 설치하는데 이 부분은 확실하지 않습니다. 이 파일들이 필수로 필요한지 다른 컴파일 조건에서도 같은 파일이 생성되는지등을 확인하지 못했으며 향후 검토할 예정입니다.
{{% /callout %}}

## 3. 의존성

의존성은 FDS에 필요한 사전 패키지들을 정의하는 과정입니다. 먼저 가장 먼저 떠오르는것은 mpi와 mkl입니다. mpi와 mkl은 spack에서는 일반 패키지가 아닌 virtual 패키지로 정의합니다. mpi는 OpenMPI, Intel MPI, MPICH등 다 양한 패키지들이 있습니다. 어떤 패키지에서 mpi가 필요하다고 정의하면 MPI 패키지중 어떤 패키지라도 하나만 있으면 충족이 됩니다.\
단 FDS는 아직까지는 다양한 MPI, 컴파일러등에서 테스트 되지는 않은것 같습니다. GNU Fortran, OpenMPI, Intel Fortan, OneAPI MPI에서만 빌드하는 방법을 제공하고 있습니다. 그래서 아래와 같이 mpi와 mkl을 `depend_on`으로 의존 관계를 정의하면서 `requires`에 ‘one of’ policy로 gcc(GNU Fortran), intel, oneapi 만을 컴파일러로 쓰도록 정의했습니다.

```python
    depends_on("mpi")
    depends_on("mkl")

    requires(
        "%gcc",
        "%intel",
        "%oneapi",
        policy="one_of",
        msg="FDS builds only with GNU Fortran or Intel Fortran",
    )
```

`requires`는 아래와 같이 사용할수도 있습니다. gcc를 사용할때는 openmpi만을 사용하도록 하라는 정의입니다. 아직 FDS에서 GNU Fortan+Intel MPI조합의 빌드는 공식적으로 제공하고 있지 않습니다.
```python
    requires(
        "^openmpi",
        when="%gcc platform=linux",
        msg="OpenMPI can only be used with GNU Fortran on Linux platform",
    )
```

## 4. PR

PR을 위한 마무리 작업을 해야 합니다. 먼저 docstring에 들어갈 문장을 만들어줍니다. 보통은 패키지 홈페이지에 있는 소개 문구를 그대로 옮겨줍니다.

또 작성한 코드가 리포지토리에서 정한 스타일 조건에 맞는지 체크합니다. spack에서는 CLI 명령을 제공해줘서 고쳐주는 기능까지 제공합니다.
```bash
spack style --fix
```

이렇게 명령을 주면 내가 작성한 코드도 스타일이 바뀌어집니다.

다음으로 PR에 사용할 설명글입니다. 변경 사항은 무엇이고 어떻게 테스트했다라는 설명을 자세하게 적는것이 좋습니다. 물론 영어로 작성해야 하기에 영어가 익숙하지 않으면 번거로운 과정입니다. 저는 ChatGPT를 사용하여 약간의 번역+글짓기를 같이 작성하게 했습니다.

위의 사항들을 정리하여 [PR](https://github.com/spack/spack/pull/37850)을 신청했습니다. 거의 하루만에 [Merge](https://github.com/spack/spack/commit/bf4fccee15853153c8346bd26328524195b603f9)가 되었습니다. 내용에 비해서는 굉장히 빨리 Merge가 된 사례입니다. 기존 패키지들은 작은 수정도 쉽지 않았는데 신규 패키지라 좀 더 쉽게 신청이 된 느낌도 있습니다. 좀 더 신규 신청건이 많아지면 확실히 이야기할 수 있을듯 합니다.

## 5. 홍보

아직은 FDS 사용자들이 Spack에 대해 잘 모르는것 같기에 알릴 필요가 있는데요. Spack 커뮤니티도 있겠지만 FDS 커뮤니티에 올리는것이 좋겠습니다. FDS는 별도의 [포럼](https://groups.google.com/g/fds-smv)을 운영하고 있습니다. 여기에 [홍보글](https://groups.google.com/g/fds-smv/c/SKyKgzHXDzo/m/WKhecl2rAQAJ)을 올렸습니다. Randy McDermott란 분이 이렇게 답변을 주었네요.

{{% callout info %}}
Hi. Thanks for the information. I was not familiar with Spack. We always seem to have to wrestle with whatever system we are compiling on, as all the HPC machines tend to have their own intricacies. But if this is somehow becoming less chaotic, that is a nice to hear.
{{% /callout %}}
Spack에 대해 잘 몰랐지만 HPC 환경에서 FDS를 컴파일하는데 애를 먹었었고 Spack이 복잡한 것을 줄여주면 좋겠다고 말하네요. FDS는 보통 일년에 두번정도의 릴리즈를 하는것 같고 신규 버전을 spack에도 반영하면서 추가 기능들을 만들어가야겠습니다.

