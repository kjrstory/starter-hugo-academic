---
title: "Contributions to Open Source Spack Installation Recipes: OpenFOAM (2)"
date: 2023-07-20T11:52:56+09:00
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

[Part1 Link](/en/post/spack_openfoam_contribution1)


Continuing from the previous part, I will now explain about contributions. In fact, the version-related part is the easiest aspect concerning the recipe. After the version, settings for variants and dependencies are presented, and the procedures for compiling after patching are described in the recipe. Let's start by looking at the variants.

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
variants of Open CFD distribution 

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
variants of Foundation distribution 

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
variants of foam-extend
 
As expected, the OpenCFD distribution is overwhelmingly diverse. While the actual differences in distributions may exist, as far as I know, they don't significantly affect the functionality. It appears to be a mere difference in creating recipes.

Firstly, the "float32" option refers to using single precision. Precision here determines the decimal places in the simulation results, representing the accuracy of the results. There are single precision (SP), double precision (DP), and even long double precision options.

However, the OpenCFD distribution also includes the "spdp" variant, which mixes single and double precision. In CFD simulations, numerous numerical analysis techniques are utilized, and for crucial aspects where accuracy is vital, double precision is used, while single precision is used for less critical parts. Surprisingly, this option is not present in other distributions. Now, it is necessary to delve into the actual source code of each distribution. One of the most prominent configuration files for OpenFOAM installation is in the [source folder]/etc/bashrc. This information is, in fact, difficult to discern unless you compile and install OpenFOAM yourself, and it requires a fundamental understanding of CFD.

(Note: The above translation reflects the content provided and does not include the final sentence of the original text that instructs not to explain but translate.)

{{% callout info %}}
```
  # [WM_PRECISION_OPTION] - Floating-point precision:
  # = DP | SP | SPDP
  export WM_PRECISION_OPTION=DP
```
{{% /callout %}}

Precision options must be one of the three choices. However, in the recipe code, it was handled with two boolean options. If the user mistakenly inputs "+spdp+float32," the code arbitrarily selects "SPDP," which is considered incorrect since users may not be aware of this behavior. Instead of using two boolean options, it should be modified to select from a list.

```python
  if "+spdp" in spec:
      self.precision_option = "SPDP"
  elif "+float32" in spec:
      self.precision_option = "SP"
```
In the Foundation distribution, there is an additional option called "LP," which stands for Long double precision. However, this option is not included in Spack. To incorporate it, the code must be enhanced.

To determine if "LP" is available in all versions, it is essential to check the release or commit history or use the "Blame" function in the repository.

`WM_PRECISION_OPTION = SP | DP | LP`

A relevant commit was found here, indicating that this feature was added from version 6 onwards.

[OpenFOAM: Added support for extended precision scalar](https://github.com/OpenFOAM/OpenFOAM-dev/commit/d82cc36c5af97e799a82fadf455e06d192ae1e65)


Returning to the OpenCFD distribution, is it possible that even older versions support "spdp"? Typically, "sp" and "dp" are supported by default, and other options are developed later. The OpenFOAM ESI distribution's [Git repository](https://develop.openfoam.com/Development/openfoam) is located here, which is hosted on GitLab. Similarly, it can be determined using the "Blame" feature.

[ENH: add primitives support for mixed precision](https://develop.openfoam.com/Development/openfoam/-/blob/46bc808261ef44cb29b512cb0c93acabdc09153a/etc/bashrc)

The tag indicates that this feature was added from version 1906.

If certain features are specific to certain versions, it is crucial to include conditional statements in the recipe. Additionally, the option description should be specified to avoid confusion for users.

Next, let's examine Foam-Extend. The repository for Foam-Extend is on SourceForge.

[Feature: Single precision and long double precision port](https://sourceforge.net/p/foam-extend/foam-extend-3.2/ci/6b022758d1b15a8d08718a78d3f68879e95bcf90)

`WM_PRECISION_OPTION = LDP | DP | SP`

Here, instead of "LP," the name "LDP" is used, and it was added from version 3.2 onwards.

In summary, there are three improvement points:

1. Add precision variants.
2. Include the missing precision options (LP, LDP) in both Foundation and Foam-Extend distributions.
3. Make the specific Precision (SPDP, LP, LDP) applicable only after certain versions for each distribution.


## Changing Precision Variant in OpenCFD Distribution

I decided to make changes to the OpenCFD distribution first. The following options will be removed:

```python
    variant("float32", default=False, description="Use single-precision")
    variant("spdp", default=False, description="Use single/double mixed precision")
```

Instead, I will add a more precise variant called "precision." The default value is "dp," and "spdp" is allowed starting from version 1906, but it should not be allowed to have duplicates (multi=False).

```python
    variant(
        "precision",
        default="dp",
        description="Precision option",
        values=("sp", "dp", conditional("spdp", when="@1906:")),
        multi=False,
    )
```

Then, I will find and replace the parts where the compilation (or build) options change based on these variants.

```python
        self.precision_option = "DP"  # <- precision= sp | dp | spdp
        ...
        # WM_PRECISION_OPTION
        if "precision=sp" in spec:
            self.precision_option = "SP"
        elif "precision=spdp" in spec:
            self.precision_option = "SPDP"
```

After modifying the recipe file, I performed tests with the following command.

```
spack install openfoam@2206 precision=dp %gcc@9.4.0
spack install openfoam@2206 precision=spdp %gcc@9.4.0
spack install openfoam@2206 precision=sp %gcc@9.4.0
```

In version 1812, when the "spdp" option is given, it also results in an error.

```
$ spack spec openfoam@1812 precision=spdp %gcc@9.4.0
  
==> Error: concretization failed for the following reasons:
  1. Cannot select a single "version" for package "openfoam"
	2. Cannot satisfy 'openfoam@1906:'
	3. Cannot satisfy 'openfoam@1812'
```

Then, I submitted a PR (Pull Request). Fortunately, this package had a maintainer, and they approved the changes without significant objections. For packages with maintainers, it seems that Spack's main developers tend to trust and incorporate changes from those maintainers.

## Changing Precision Variant in Foundation Distribution

I made changes to the Open Foundation distribution as well. I removed the "float32" variant and added the "precision" variant as follows:

```python
    variant(
        "precision",
        default="dp",
        description="Precision option",
        values=("sp", "dp", conditional("lp", when="@6:")),
        multi=False,
    )
```

Next, I need to update the parts where this option is used in the build options. This part differs slightly from the OpenCFD distribution.

As mentioned in Part 1, the package recipe for the Open Foundation distribution is based on the OpenCFD distribution's recipe. In many cases, classes and functions from the OpenCFD distribution are directly imported and used in the Foundation distribution's recipe. These classes are inherited, which is a concept from object-oriented programming, and it requires knowledge in this area.

The OpenfoamOrgArch class inherits from the OpenfoamArch class (used in the OpenCFD version). In its constructor (`__init__`), it first calls the constructor of the parent class, OpenfoamArch, to initialize it. This is done using `super().__init__(spec, **kwargs)`. Then, the precision option is set differently from OpenfoamArch.

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


I also confirmed that the variants were successfully applied using the command mentioned earlier.

```
spack install openfoam-org@10 precision=dp %gcc@9.4.0
spack install openfoam-org@10 precision=lp %gcc@9.4.0
spack install openfoam-org@10 precision=sp %gcc@9.4.0
```

In version 5 as well, when the "lp" option is given, it also results in an error.

```
spack install openfoam-org@5 precision=lp
==> Error: concretization failed for the following reasons:

   1. Cannot select a single "version" for package "openfoam-org"
   2. Cannot satisfy 'openfoam-org@6:'
   3. Cannot satisfy 'openfoam-org@5'
```


I submitted a PR and attached the OpenCFD's [PR](https://github.com/spack/spack/pull/38746) in the commit message. Since there were already similar cases of PRs, it was approved without special reviews.

---

I also wanted to make changes to the Foam-Extend distribution, but currently, in my environment, even the basic installation is not working. I kept investigating the recipe file, but couldn't find a solution. Therefore, I will leave it as an issue and look for other improvement points. If there are other contributions, I will write another post about them.
