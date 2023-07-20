---
title: "Contributions to Open Source Spack Installation Recipes: OpenFOAM (1)"
date: 2023-05-17T11:52:56+09:00
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

[Spack](https://spack.io) is a tool that assists in the installation of open-source applications used in HPC (High-Performance Computing) environments. Current open-source software packages are not just single packages (libraries), but rather consist of a complex interrelationship of many packages, as depicted in the image below:

![Dependency_Ares](Ares_Spack.png "Dependency graph of one configuration of ARES")[^1]

Additionally, depending on the compiler and CPU architecture used, the compilation options may vary. To manage such complexity systematically, [Spack](https://spack.io) was developed. This article does not aim to serve as a tutorial or user guide for Spack. Instead, it intends to showcase cases where developers contribute to Spack. Spack can install a vast number of open-source software (approximately 7000 packages as of version 0.19.2 in April 2023). In this article, we will focus on one of the open-source CFD (Computational Fluid Dynamics) software, Openfoam.

## Basic Usage of Spack
Although a separate article could be dedicated to explaining how to use Spack, this article will provide a brief explanation. The installation process of Spack is simple. You need to clone the GitHub repository and set up the environment variables.

```bash
git clone -c feature.manyFiles=true https://github.com/spack/spack.git
. spack/share/spack/setup-env.sh
spack version 
```

You can view package descriptions using the spack info command.

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


The OpenFOAM package provides options for versions from 1612 to 2206, along with various dependencies and installation variants.

Before installation, you can use the spack spec command to determine which packages will be installed.

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

For OpenFOAM, packages such as flex, openmpi, boost, cgal, cmake, and scotch will be installed. Typically, when compiling and installing OpenFOAM, some of these packages might have been installed as binary packages (e.g., rpm or deb) if you have prior experience with OpenFOAM installations. However, Spack's default behavior is to compile and install all the required packages from source. Of course, if you already have package files installed using other methods like yum, dnf, or apt, you can utilize those as well.

During the installation configuration, you can choose the settings, options, and compilers for each of the dependent packages. The command provided below demonstrates the installation of OpenFOAM version 2206 with specific options and a selected compiler:

```bash
  spack install openfoam@2206~float32~int64~kahip~knl~metis~mgridgen~paraview+scotch+source~spdp~vtk~zoltan build_system=generic %gcc@9.4.0
```

However, if you prefer the default options, you can omit them and specify only the desired version of OpenFOAM and the compiler:

```bash
  spack install openfoam@2206 %gcc@9.4.0
```

Once you execute the command, all the packages that were previously checked with the spec command will be installed, compiling them from source. The installation time may vary depending on the server environment, taking anywhere from several minutes to several hours.

## Introduction to Spack Package Recipes
The version definitions, dependencies, and variant definitions for OpenFOAM are specified in the [Spack repository](https://github.com/spack/spack), not by the OpenFOAM community. 
You can view the installation details in the recipe code using the `spack edit openfoam` command, which is written in Python and considered a Domain Specific Language (DSL). 
In the Spack community, these recipe codes are referred to as package recipes. 
Below is an example quoted from the Spack documentation:
In summary, Spack can be defined as a tool that provides package recipes, which are self-contained language-specific representations of how to install open-source software packages.

```python
class Openjpeg(CMakePackage):
    """OpenJPEG is an open-source JPEG 2000 codec written in C language"""

    homepage = "https://github.com/uclouvain/openjpeg"
    url = "https://github.com/uclouvain/openjpeg/archive/v2.3.1.tar.gz"

    version("2.4.0", sha256="8702ba68b442657f11aaeb2b338443ca8d5fb95b0d845757968a7be31ef7f16d")

    variant("codec", default=False, description="Build the CODEC executables")
    depends_on("libpng", when="+codec")

    def url_for_version(self, version):
        if version >= Version("2.1.1"):
            return super(Openjpeg, self).url_for_version(version)
        url_fmt = "https://github.com/uclouvain/openjpeg/archive/version.{0}.tar.gz"
        return url_fmt.format(version)

    def cmake_args(self):
        args = [
            self.define_from_variant("BUILD_CODEC", "codec"),
            self.define("BUILD_MJ2", False),
            self.define("BUILD_THIRDPARTY", False),
        ]
        return args
```

Example of package recipe[^2]


The Spack community defines users of Spack as general users, developers, and packagers. Packagers are individuals who develop package recipes, separate from core developers, to distinguish their roles. Contributing to the core of well-known open-source projects often requires a deep understanding of the project's structure and passing rigorous tests and reviews, which might not be suitable for newcomers. In this context, even contributing to small parts or documentation can be considered as valuable contributions to open-source projects, including contributing to Spack's package recipes.

## Introduction to OpenFOAM Distributions and Package Recipes Status
OpenFOAM stands out from other open-source projects in that it is distributed in three main versions, each managed by different entities. The versions are: [OpenFoam Foundation](http://openfoam.org), [OpenFoam Plus](http://openfoam.com) (distributed by ESI Open CFD), and [Foam-Extend](https://sourceforge.net/projects/foam-extend/) (managed by Hrvoje Jasak and available at Foam-Extend SourceForge). In this article, we refer to them as Foundation distribution, OpenCFD distribution, and Extend distribution, respectively. In Spack, each of these distributions is registered as a separate package.

Interestingly, when examining the package recipe code, it is evident that the developer of the ESI distribution has contributed to the code for other distributions as well. However, while the ESI distribution is frequently updated, the other distributions receive updates less frequently. To address this, the focus is primarily on enhancing the package recipe for the Foundation distribution. From now on, I will provide cases where contributions have been made to the code.

## First Contribution (Version Definition)
Below is an excerpt of the package recipe file, specifically the part related to version definitions:

```python
  class OpenfoamOrg(Package):
      ...

      homepage = "https://www.openfoam.org/"
      baseurl = "https://github.com/OpenFOAM"
      url = "https://github.com/OpenFOAM/OpenFOAM-4.x/archive/version-4.1.tar.gz"
      git = "https://github.com/OpenFOAM/OpenFOAM-dev.git"

      version("develop", branch="master")
      version(
          "10",
          sha256="59d712ba798ca44b989b6ac50bcb7c534eeccb82bcf961e10ec19fc8d84000cf",
          url=baseurl + "/OpenFOAM-10/archive/version-10.tar.gz",
      )
      version(
          "9",
          sha256="0c48fb56e2fbb4dd534112811364d3b2dc12106e670a6486b361e4f864b435ee",
          url=baseurl + "/OpenFOAM-9/archive/version-9.tar.gz",
      )
      version(
          "8",
          sha256="94ba11cbaaa12fbb5b356e01758df403ac8832d69da309a5d79f76f42eb008fc",
          url=baseurl + "/OpenFOAM-8/archive/version-8.tar.gz",
      )
      ...
      version(
          "5.0",
          sha256="9057d6a8bb9fa18802881feba215215699065e0b3c5cdd0c0e84cb29c9916c89",
          url=baseurl + "/OpenFOAM-5.x/archive/version-5.0.tar.gz",
      )
      ...
      version(
          "2.3.1",
          sha256="2bbcf4d5932397c2087a9b6d7eeee6d2b1350c8ea4f455415f05e7cd94d9e5ba",
          url="http://downloads.sourceforge.net/foam/OpenFOAM-2.3.1.tgz",
      )
      ...
```

In the official Spack documentation, it is mentioned that Spack can intelligently find the URL if it follows a specific format based on the version. When looking at the code and explanation below, if the URL format is version-specific, Spack automatically detects the appropriate URL for each version. Explicitly specifying URLs for each version, as in the case of OpenFOAM-org, can lead to messy code, so it is necessary to refactor the code to make it more streamlined.

```python
  class Foo(Package):

      url = "http://example.com/foo-1.0.tar.gz"

      version("8.2.1", md5="4136d7b4c04df68b686570afa26988ac")
      version("8.2.0", md5="1c9f62f0778697a09d36121ead88e08e")
      version("8.1.2", md5="d47dd09ed7ae6e7fd6f9a816d7f5fdf6")
```

{{% callout quote %}}

By default, each version’s URL is extrapolated from the url field in the package. For example, Spack is smart enough to download version 8.2.1 of the Foo package above from [http://example.com/foo-8.2.1.tar.gz](http://example.com/foo-8.2.1.tar.gz).

{{% /callout %}}

However, there is one exception to the rule. For versions 5.0 and below, there is a special suffix in the URL, such as "5.x", that needs to be handled differently. To address this, I decided to create a new function to resolve the issue. The modified part is as follows, and it significantly improves the clarity of the code compared to the original version.

```python
  class OpenfoamOrg(Package):
    ...
      homepage = "https://www.openfoam.org/"
      baseurl = "https://github.com/OpenFOAM"
      url = "https://github.com/OpenFOAM/OpenFOAM-6/archive/version-6.tar.gz"
      git = "https://github.com/OpenFOAM/OpenFOAM-dev.git"

      version("develop", branch="master")
      version("10", sha256="59d712ba798ca44b989b6ac50bcb7c534eeccb82bcf961e10ec19fc8d84000cf")
      version("9", sha256="0c48fb56e2fbb4dd534112811364d3b2dc12106e670a6486b361e4f864b435ee")
      ...
    ...

      def url_for_version(self, version):
          """If the version number is 5.0 or lower, the returned URL includes
          the ".x" suffix in the OpenFOAM directory name to reflect
          the old directory naming convention for these versions.

          """
          if version <= Version("5.0"):
              url= "https://github.com/OpenFOAM/OpenFOAM-{0}.x/archive/version-{1}.tar.gz"
              return url.format(version.up_to(-1), version)
    ...
```

By the way, to obtain the sha256 value of the file, you can use the sha256sum utility available in most Linux environments, as shown below:


```bash
sha256sum ./version-6.tar.gz
32a6af4120e691ca2df29c5b9bd7bc7a3e11208947f9bccf6087cfff5492f025  ./version-6.tar.gz
```

Now, I will attempt to create a [PR(Pull Request)](https://github.com/spack/spack/pull/37587). Without providing a detailed explanation of Git in this article, a PR is a request to merge the changes you made into the official repository. Since open-source projects involve collaboration from many people, trying to change each other's code directly can quickly lead to spaghetti code. Therefore, in open-source projects, there are usually main organizations or lead developers who have the final authority to merge changes. Each open-source project may have different rules and philosophies regarding PRs. Merging can be a challenging process for some open-source projects, and some people might be disappointed if they approach open-source with the expectation of complete freedom. I personally believe that even in open-source projects, the initial developers who design the structure and maintain the repository (often referred to as the repository owner) are remarkable, and it is essential to follow their approach.

However, my PR did not receive an immediate approval as I had initially thought. Instead, it received some negative feedback:

{{% callout quote %}}
Hmmm. I confirmed/re-confirmed all of the explicit package versions.

While this works I am hesitant to approve because the url_for_version method is normally "complete" in that it provides a single location to determine the "standard" URLs for the package.
{{% /callout %}}

The reviewer pointed out that using the url_for_version method requires providing all versions, making it not a complete solution with just one if statement.

As a result, I made the following code modifications to address the concern:
```python
    def url_for_version(self, version):
        """If the version number is 5.0 or lower, the returned URL includes
        the ".x" suffix in the OpenFOAM directory name to reflect
        the old directory naming convention for these versions.
        """
        if version == Version("2.3.1"):
            return "http://downloads.sourceforge.net/foam/OpenFOAM-2.3.1.tgz"
        elif version <= Version("5.0"):
            version_prefix = str(version.up_to(-1)) + ".x"
        else:
            version_prefix = version

        url = "https://github.com/OpenFOAM/OpenFOAM-{}/archive/version-{}.tar.gz".format(
            version_prefix, version
        )
        return url
```


I further improved the code by enhancing the conditional statements to cover all cases, making the code more complete and satisfying the reviewer's concern. With these enhancements, the PR was approved and merged. Initially, I might not have fully grasped the review, but after making these adjustments, it was well-received.

Even though the contribution may seem small, it serves as a relevant example of how even minor contributions in open-source projects require going through rigorous processes to be implemented. Beyond version specification, I started to notice other areas that could be improved.

Moving forward, I plan to explore more complex aspects that often require firsthand experience with compiling and installing OpenFOAM. Additionally, I will cover further contributions made, addressing them in Part 2.

To be Continued :pray:

Part 2 Link: (/ko/post/spack_openfoam_contribution2)

[^1]: "Spack: A Flexible Package Manager for HPC Software", [https://computing.llnl.gov/projects/spack-hpc-package-manager](https://computing.llnl.gov/projects/spack-hpc-package-manager)
[^2]: Main Spack Documentation, https://spack.readthedocs.io

