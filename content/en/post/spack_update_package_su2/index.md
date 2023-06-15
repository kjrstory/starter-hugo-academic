---
title: Updating the SU2 package in Spack
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
  filename: 'SU2_hl_crm_01.png'
  focal_point: Smart
  preview_only: false
  caption: "Figure by [SU2 Homepage](https://su2code.github.io)"
---

 [SU2](https://su2code.github.io) is an open-source CFD (Computational Fluid Dynamics) software developed by the Aerospace Design Lab at Stanford University. I have been interested in SU2 since the early 2010s, and I have used SU2 for analysis in [my paper](/ko/publication/jong-rok-kim-2017-cfd-analysis-efdcfd). I also updated it on Spack two years ago and provided support as a maintainer.
At Spack, [the maintainer system](https://spack.readthedocs.io/en/latest/packaging_guide.html#maintainers) is in place for every package. Once you become a maintainer, you have the authority to approve and review PRs (Pull Requests) or issues submitted to the package recipe file. However, even though I became a maintainer for SU2, I was unable to provide updates during that time. When I looked back at it two years ago, the issues that were difficult to address seemed clearer now, and I was able to make the necessary fixes. In the process, I believe my skills have unknowingly grown.
 
# 1.Pull Request  

 The existing recipe didn't have any variants because I didn't know how to apply them, although SU2 itself does support options. This time, after reading [the SU2 build instructions](https://su2code.github.io/docs_v7/Build-SU2-Linux-MacOS/), I decided to apply variants.

 SU2 supports the following options:

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

I have applied these changes to the package recipe.
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

Then, while continuing to read SU2's build documentation, I set the dependencies.
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
Now, the crucial part is how to apply the content of the variants. To do that, we need to understand the build system of SU2, which is [Meson](https://mesonbuild.com).

Meson is a relatively modern build system written in Python. It works in conjunction with [Ninja](https://ninja-build.org) to build projects. It is known to be easier to use and faster in terms of build speed compared to CMake.

I need to read the explanation of [Meson Build] (https://spack.readthedocs.io/en/latest/build_systems/mesonpackage.html)in Spack.
In the end, the options required for building SU2 are Meson arguments, and these can be applied using the meson_args function. I have written the code as f

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

# 2.Review 1: Meson
 Starting from version 7.4.0 to 7.5.1, SU2 fixed the Meson version to 0.61.1. However, the Meson package in Spack does not have version 0.61.1 registered, so I requested a [pull request](https://github.com/spack/spack/pull/37770) to add it.
I received a negative review from eli-schwartz.
To be honest, receiving a negative review or having a PR rejected can initially be disheartening. However, there are always reasons for such rejections, so it's important to calmly read the review again and respond accordingly.

{{% callout quote %}}
I disagree with this change because it is an old and buggy version of meson that has later patch/bugfix releases within the same minor release series, which spack already packages.

Furthermore, SU2 no longer needs this due to su2code/SU2#1951
{{% /callout %}}

It seems that version 0.61.1 is too low compared to the current version, and SU2 also does not support it. Upon further investigation, I found that the latest version of Meson is 1.1.0, so version 0.61.1 was indeed outdated. I should read the [PR](https://github.com/su2code/SU2/pull/1951) for SU2 (PR) again.
By reading the PR, it appears that the Meson version restriction is changing from fixed at 0.61.1 to a minimum version requirement, and there are also some modifications to the build options.

However, I realized that this PR was for the develop branch of SU2, and eli-schwartz was the reviewer for this PR. It means that the PR has not been incorporated into the released version yet.
After contemplating the situation, I came to the conclusion that there is no need to restrict the Meson version.
Therefore, I have decided to patch the installation process of SU2 by removing the check for Meson version 0.61.1. I have confirmed that the installation works correctly with Meson version 1.1.0, and I have finalized the decision to apply the patch.

I have applied the patch and left my comments.

{{% callout quote %}}
I agree with @eli-schwartz opinion.
However, currently, this commit(su2code/SU2#1951) has not been applied to any released versions of SU2.
As I have tested, the restriction was implemented in a specific range of versions (@7.4.0:7.5.1) of SU2, but I have successfully installed SU2 with a different version of meson(1.1.0).
In light of this, I have created the patch and submitted a pull request (#37767).
I would greatly appreciate your feedback on the meson version included in this PR.
{{% /callout %}}

In the Meson PR, there were a few more comments added. adamjstewart mentioned that applying an older version doesn't seem like a big deal, but eli-schwartz provided more detailed comments in response.

{{% callout quote %}}
I'd be less concerned about adding an old version if it were the latest bugfix release of that old version. :D

Spack doesn't currently package the .5 bugfix release for this old minor release. I'd have no objections to adding that -- it fixes bugs that may be encountered by people who have made the judgment call that their needs are best served by sticking to that old version and avoiding sweeping changes, deprecations, and whatnot, from newer versions.

As an upstream meson maintainer, I want to push people to always upgrade to the latest bugfix release, if they aren't in a position to upgrade to the latest feature release. That's why we have 0.61.x as an LTS in the first place.

But for software that locks itself to an exact bugfix release, the best solution is to backport an upstream change that removes the locked dependency. That's what the other PR does, so this PR won't be needed even for SU2 anymore. :)
{{% /callout %}}


eli-schwartz being a developer of the Meson build system may have contributed to the detailed feedback provided, even for a seemingly simple version constraint change. It's true that one of the valuable aspects of pull requests is the opportunity to learn from and exchange opinions with other open-source developers.

# 3.Review 2: minor requests

After that, alalazo made a few requests:

1. Add simple comments to the patch.
2. Reflect that CMake is a build-only dependency.
3. Reflect that Swig is also a build-only dependency.

Requests number 2 and 3 are similar in nature. We need to distinguish between dependencies required for the build phase and those required for the runtime phase.

For dependencies that are only needed during the build phase, you can simply add the statement type="build" as follows:
```python
   depends_on("swig", type="build", when="+pywrapper")
```

After incorporating all three requested changes, the patch has finally been approved.

# 4. Test

Actually, tests should be conducted before carrying out the PR, and I have performed tests to ensure proper installation. However, after the initial PR, there were a few modifications, so I conducted another test for the purpose of writing an article on this blog.

While it would be ideal to conduct tests across multiple architectures and compilers, the testing conditions have not been fully secured. Therefore, I have performed tests only in the gcc9 and Ubuntu20-Cascadelake environment.

Using the default options, I have selected the following test cases by varying each variant option:


 - (default)
 - +autodiff+directidiff 
 - +cgns
 - +mixedprec
 - +mpp
 - +mkl
 - +openblas
 - +mpi
 - +openmp
 - +pywrapper 

Each installation option was successfully installed without any errors. Lastly, I conducted additional tests by adding several options. The success of the tests was determined by examining the build logs.

 - +autodiff+directdiff+mpi+mpp+openblas+openmp+pywrapper

If you have any specific questions or need further assistance regarding the test results or the installation process, please let me know.

```
SU2 7.5.1 "Blackbird"

  Subprojects
    Mutationpp       : YES

  User defined options
    buildtype        : release
    default_library  : shared
    libdir           : /root/spack/opt/spack/linux-ubuntu20.04-cascadelake/gcc-9.4.0/su2-7.5.1-lukxy4ymotmzhmxt6aertfgltivalpce/lib
    prefix           : /root/spack/opt/spack/linux-ubuntu20.04-cascadelake/gcc-9.4.0/su2-7.5.1-lukxy4ymotmzhmxt6aertfgltivalpce
    strip            : false
    wrap_mode        : nodownload
    enable-autodiff  : True
    enable-cgns      : True
    enable-directdiff: False
    enable-mixedprec : False
    enable-mkl       : False
    enable-mpp       : True
    enable-openblas  : True
    enable-pywrapper : True
    enable-tecio     : True
    with-mpi         : enabled
    with-omp         : True
```
Based on the log, it is evident that the intended variants have been applied successfully. 

# 5. Future Work

I believe this pull request actually had quite a few changes. However, there was a task that I couldn't finish in time, and I thought it would be better to request the PR at the current state, splitting it into smaller pieces rather than applying everything at once. Instead, I have clearly listed the tasks to be done from now on below.

## 5-1. Excluding the Git submodule feature

In the existing method of building SU2 without using Spack, there is a feature that utilizes Git submodules. For example, among the dependencies, there is a package called [codipack](https://github.com/SciCompKL/CoDiPack), which has its own separate repository. In SU2, it downloads the commit corresponding to a specific version from this repository.

However, this approach differs from the installation method used in Spack. In Spack, dependencies also need to be installed using Spack itself. Here are the external packages that utilize Git submodules:

 - codipack
 - medipack
 - opdilib
 - MEL
 - ninja
 - meson
 - coolprop
 - mutationpp

Among these packages, meson and ninja are already available for installation using Spack. However, codipack, medipack, opdilib, MEL, coolprop, and mutationpp are not currently registered in Spack and need to be added. While mutationpp is registered in Spack, there have been some errors when applying it, so it has been incorporated as a submodule instead.

## 5-2. Testing with actual analysis problems

As mentioned in section 4, the determination of test success was based solely on the build log, without actually conducting tests using SU2. While there has been experience using SU2 for various analyses, it does not necessarily cover all the options and configurations specifically tailored for Spack.

To prepare for this, specific analysis files corresponding to each variant are required, making it not easily prepared. For example, AD(reverse) support is used only for specific analyses, so it is necessary to prepare and run an analysis problem using it to confirm if the installation has been properly completed.

## 5-3. Setting environment variables during runtime

There is a part in the build log of SU2 that mentions setting environment variables when using SU2.

```
         Please be sure to add the $SU2_HOME and $SU2_RUN environment variables,
         and update your $PATH (and $PYTHONPATH if applicable) with $SU2_RUN

         Based on the input to this configuration, add these lines to your .bashrc file:

         export SU2_RUN=/root/spack/opt/spack/linux-ubuntu20.04-cascadelake/gcc-9.4.0/su2-7.5.1-lukxy4ymotmzhmxt6aertfgltivalpce/bin
         export SU2_HOME=/tmp/root/spack-stage/spack-stage-su2-7.5.1-lukxy4ymotmzhmxt6aertfgltivalpce/spack-src
         export PATH=$PATH:$SU2_RUN
         export PYTHONPATH=$PYTHONPATH:$SU2_RUN
```

Spack also has a feature for setting environment variables, but at the time of the pull request, I was not familiar with that feature and couldn't apply it. If the feature is applied, instead of directly modifying the .bashrc file as mentioned in the previous statement, simply loading the package with the following command will set the environment variables.


```bash
spack load su2@7.5.1
```
