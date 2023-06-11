---
title: Creating a New Package (FDS) in Spack
date: 2023-06-05T07:23:41.994Z
draft: false
featured: false
tags:
  - spack
  - fds
categories:
  - HPC
image:
  filename: fds_example.png
  focal_point: Smart
  preview_only: false
  caption: "Figure by [fdstutorial](https://fdstutorial.com/what-is-fds)"
---

This article describes a case of registering a new package, [FDS](https://pages.nist.gov/fds-smv/) (Fire Dynamics Simulator), specialized in fire simulation, in [Spack](https://spack.io). FDS is an open-source CFD (Computational Fluid Dynamics) software developed by [NIST](https://www.nist.gov/) (National Institute of Standards and Technology) in the United States. It utilizes LES (Large Eddy Simulation) and enables thermal analysis of smoke. For post-processing, it uses a self-developed software called Smokeview. It was officially released in 2000 and has gained a reputation as a prominent open-source software in the field of fire simulation.

## 1. Creating a new package

Spack provides two methods for creating packages. One is using the Spack CLI commands, and the other is creating a Python file directly. Let's first look at the first method using the create command. You will need to replace the download URL with the appropriate source code for your package. In this example, I used the release URL from GitHub:

```bash
spack create https://github.com/firemodels/fds/archive/refs/tags/FDS-6.8.0.tar.gz
```

The output of this command is as follows

```bashsession
  ==> Using specified package name: ‘fds’
  ==> Found 1 version of fds:
    6.8.0  https://github.com/firemodels/fds/archive/refs/tags/FDS-6.8.0.tar.gz
  ==> Fetching <https://github.com/firemodels/fds/archive/refs/tags/FDS-6.8.0.tar.gz>
  ==> Warning: Unable to detect a build system. Using a generic package template.
  ==> Created template for fds package
  ==> Created package file: /root/spack/var/spack/repos/builtin/packages/fds/package.py
```

If you continue reading the output (the last line), you can see that a package.py file is created under a complex path within the spack folder. The second method of creating a new package is by directly creating this package.py file. If you are familiar with writing package recipes, it would be more useful to refer to existing recipe files created by others, rather than starting from the default template file, when using the create command.


## 2 Selecting a build system

First, you need to have an understanding of the [build system](https://en.wikipedia.org/wiki/Build_automation). Some popular build systems include Make, CMake, Maven, Meson, and Autotools. Since most major open-source projects utilize a build system, it is essential to have an understanding of build systems when creating Spack package recipes. So, what build system does FDS use? As mentioned in the previous paragraph, it is not found using the `spack create` command. However, FDS operates a GitHub wiki where you can find information. In the [FDS Compilation article](https://github.com/firemodels/fds/wiki/FDS-Compilation), you can learn how to build FDS. It mentions separate build directories for different architectures and the use of a custom bash script called `make_fds.sh`.

{{% callout info2 %}}
cd to the directory in the fds repository called **Build**.

cd to the appropriate directory within Build, such as **ompi_intel_linux** for the Intel compiler and Open MPI libraries under linux

Type make_fds.bat or **make_fds.sh**, depending on your OS.
{{% /callout %}}

Based on the contents of the make_fds.sh file, which utilizes the make command and references a Makefile, it can be inferred that FDS uses the Makefile build system.

```bash
#!/bin/bash
dir=`pwd`
target=${dir##*/}

echo Building $target
make -j4 VPATH="../../Source" -f ../makefile $target
```

I have determined that the make_fds.sh file primarily serves the purpose of setting appropriate variables during the build process. Now, a decision needs to be made: whether to consider the make_fds.sh file as a self-made build file in fds and use a general build system in Spack or to utilize the Makefile build system. In my opinion, it is preferable to make use of Spack's functions as much as possible. Therefore, I have decided to utilize the Makefile build system, which means that the role of the make_fds.sh file will be replaced by the Spack package recipe. As a side note, the reason the build system couldn't be found using the spack create command is because the Makefile was not located in the top-level directory of the source code.

First, I suggest reading the Spack documentation page on [the Makefile build system](https://spack.readthedocs.io/en/latest/build_systems/makefilepackage.html) provided by Spack. This will give you an overview of the edit, build, and install steps involved in the Makefile build system. The edit step involves modifying the Makefile, the build step entails compiling the code, and the install step involves copying or linking the compiled executable files.

### 2.1 edit

The edit step involves modifying the Makefile or inputting necessary environment variables. Taking a look at another `make_fds.sh` file located in a different directory as an example, there is a process of including the `INTEL_IFORT` variable when selecting the Intel compiler. The default value for this variable is set to `ifort`, which can be recognized by those familiar with Fortran. `ifort` is the command name for Intel Fortran. However, with the release of Intel's oneAPI, they changed the command name to `ifx`. Therefore, this part needs to be specified in the variable accordingly.

The `makefile.filter` section is responsible for modifying the Makefile. It uses the sed command, which is similar to other Linux commands. There is a difference in the paths when executing the `make_fds.sh` file compared to directly running make on the Makefile. Therefore, the purpose of this section is to adjust the paths accordingly to ensure consistency.

```python
    def edit(self, spec, prefix):
        env["MKL_ROOT"] = self.spec["mkl"].prefix
        if spec.compiler.name == "oneapi":
            env["INTEL_IFORT"] = "ifx"
        makefile = FileFilter("Build/makefile")
        makefile.filter(r"\.\./Scripts", "./Scripts")
        makefile.filter(r"\.\.\\Scripts", ".\\Scripts")
```

{{% callout warning2 %}}
It is necessary to review the part where the MKL_ROOT environment variable is set. It appears that FDS uses the MKLROOT variable instead of MKL_ROOT. There should be no underscore between "MKL" and "ROOT". The reason for the mistake is that the information in the wiki differs from the actual Makefile. MKLROOT is an environment variable that is set by Spack during the installation of oneAPI. However, it is only set if a specific option is enabled. Although this option is enabled by default, users have the ability to disable it, so a clear definition is needed.
{{% /callout %}}

### 2.2 build

The make command is typically divided into options and targets. In the example you provided from make_fds.sh, the -j and -f are options, while $target corresponds to the [target].

```bash
make –help Usage: make \[options] \[target] …
```

The -j4 option indicates that the build process will use four threads. In Spack, the number of threads is automatically determined, and if there are sufficient CPUs, more threads may be utilized.

The -f option specifies the path to the Makefile. Since the build will take place in the directory where the Makefile is located, the path should be ./makefile, not ../makefile. The ../ signifies the parent directory, while ./ represents the current directory. If the build is performed in the current directory, the -f option is not necessary.

The next part pertains to the target. The values for the OS, compiler, and MPI are included in the target. In the original FDS approach, the directory serves as the target value. However, if you want to build without entering a specific folder, the target value needs to be created in Spack. You can specify the target value using the property decorator and the build_targets function as follows.

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

The return value will be `["{}_{}_{}".format(mpi_prefix, compiler_prefix, platform_prefix)]`, which provides the MPI, compiler, and platform (OS) values. Since the naming conventions in Spack and FDS are different, a dictionary mapping was created to establish the correspondence between them.

In other words, instead of entering the directory that serves as the target value under the Build directory and building using the `make_fds.sh` file, the approach has been modified to build directly under the Build directory. However, the target value is replaced with the value received from Spack.

### 2.3 install

I have decided to use a new build directory instead of the existing FDS build structure, where the executable files are placed in a folder built with the format Build/ompi_gnu_linux. As a result, I need a designated location for the installed executable files. I plan to create a bin folder within the FDS installation directory and place the necessary files in that bin folder. Below is the install function I have created for this purpose.

```python
   def install(self, spec, prefix):
        mkdirp(prefix.bin)
        with working_dir(self.build_directory):
            install("*.mod", prefix.bin)
            install("*.o", prefix.bin)
            install("fds_" + self.build_targets[0], prefix.bin + "/fds")
```

{{% callout warning2 %}}
I am not sure about installing module files (mod) and object files (o) into the bin folder when compiling with GNU Fortran and Open MPI. I have not been able to confirm if these files are necessary or if similar files are generated under different compilation conditions. I plan to review this in the future.
{{% /callout %}}

## 3 Dependency

Dependency definition is the process of specifying the prerequisite packages for FDS. The first two that come to mind are MPI and MKL. In spack, MPI and MKL are defined as virtual packages, rather than regular packages. MPI encompasses various packages such as OpenMPI, Intel MPI, and MPICH. If a package specifies a dependency on MPI, any one of these MPI packages satisfies the requirement.

However, it seems that FDS has not been extensively tested with different MPI implementations and compilers. It currently provides build instructions for GNU Fortran, OpenMPI, Intel Fortran, and OneAPI MPI. Therefore, the dependency relationship is defined by using `depend_on` to specify the dependencies on MPI and MKL, while `requires` enforces a 'one of' policy for compilers, allowing only gcc (GNU Fortran), Intel, and OneAPI to be used as compilers.


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

`requires` can also be used as follows: when using gcc, specify that only OpenMPI should be used. It is worth noting that the FDS does not currently provide official support for the combination of GNU Fortran and Intel MPI.

```python
    requires(
        "^openmpi",
        when="%gcc platform=linux",
        msg="OpenMPI can only be used with GNU Fortran on Linux platform",
    )
```

## 4 Pull Request

I need to perform some final tasks for the pull request. Firstly, I will create a statement for the docstring. Typically, this involves copying the introductory statement from the package's homepage.

```python
    """
    Fire Dynamics Simulator (FDS) is a large-eddy simulation (LES) code for low-speed flows,
    with an emphasis on smoke and heat transport from fires.
    FDS and Smokeview are free and open-source software tools provided by the National Institute
    of Standards and Technology (NIST) of the United States Department of Commerce. Pursuant
    to Title 17, Section 105 of the United States Code, this software is not subject to copyright
    protection and is in the public domain. View the full disclaimer for NIST-developed software.
    """
```

Additionally, I will check if the code I have written adheres to the style guidelines specified by the repository. In the case of spack, they provide CLI commands that can help automatically fix certain style issues.

```bash
spack style --fix
```

By issuing such a command, the code I have written will indeed be modified to adhere to the specified style.

Next, I will provide a description for the pull request (PR). It is recommended to provide detailed explanations of the changes made and how they were tested. Since the description needs to be in English, it can be a cumbersome process if you are not familiar with the language. I used ChatGPT to assist with a combination of translation and writing.

I summarized the above points and submitted a PR, which can be found [here](https://github.com/spack/spack/pull/37850). Surprisingly, it was merged in just one day, as seen [here](https://github.com/spack/spack/commit/bf4fccee15853153c8346bd26328524195b603f9). The merge was relatively fast compared to the content of the changes. While making small modifications to existing packages was not easy, it seems that the process for a new package was smoother. If there are more new package submissions in the future, we can definitely make a stronger case for it.

## 5 Promotion

It seems that FDS users are still not familiar with Spack, so it would be helpful to inform them. While there is a Spack community, it would be a good idea to post about it in the FDS community as well. FDS operates a separate [forum](https://groups.google.com/g/fds-smv), where I have posted a promotional [message](https://groups.google.com/g/fds-smv/c/SKyKgzHXDzo/m/WKhecl2rAQAJ). Randy McDermott provided the following response:

{{% callout quote2 %}}
Hi. Thanks for the information. I was not familiar with Spack. We always seem to have to wrestle with whatever system we are compiling on, as all the HPC machines tend to have their own intricacies. But if this is somehow becoming less chaotic, that is a nice to hear.
{{% /callout %}}

They were not familiar with Spack, but they had struggled with compiling FDS in an HPC environment. They mentioned that it would be great if Spack could simplify the complexity of the process. It seems that FDS typically releases updates about twice a year, and they expressed the need to incorporate new versions into Spack while adding additional features.
