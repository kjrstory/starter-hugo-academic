---
title: Using OpenFoam with Open Source Package Manager Spack
date: '2023-10-01'
draft: false
publishDate: '2024-02-22T08:00:00Z'
authors:
- ' Jong Rok Kim'
publication_types:
- '1'
featured: false
publication: '*10th OKUCC*'
url_pdf: https://nextfoam.co.kr/proc/DownloadProc.php?fName=231101150741_stiSq0tEeT.pdf&realfName=10thOKUCC_%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4%20%EC%84%A4%EC%B9%98%EB%A7%A4%EB%8B%88%EC%A0%80%20Spack%EC%97%90%EC%84%9C%20OpenFOAM%20%ED%99%9C%EC%9A%A9.pdf
image:
  filename: fig-01.png
  focal_point: Smart
  preview_only: false
---

[OKUCC](https://nextfoam.co.kr/okuc.php) stands for OpenFOAM Korea Users’ Community Conference, an event for the OpenFOAM community in Korea hosted by NextFOAM. I started using OpenFOAM while working at Hanwha. However, I couldn't attend OKUCC during my time at Hanwha and ended up presenting at Samsung SDS, where I couldn't use it as much. Although I didn’t attend OKUCC at Hanwha, I did participate in annual conferences every year, but I couldn’t do so after joining Samsung SDS. After this presentation, I set a goal to participate in external activities at least twice a year.

OKUCC has a more relaxed atmosphere than typical academic conferences, but the content of the presentations is not necessarily simple. As a user community event, it only requires sharing presentation materials, without the need to prepare papers in written form. This is both an advantage and a disadvantage: it saves presenters from the effort of writing a paper, but it also means that attendees don’t get as much detailed information from the presentations.

Therefore, I want to include explanations for each slide in this post. The explanations include both content that was actually discussed during the presentation and some additional points.

#### Slide 2
One of the main barriers to entry for OpenFOAM is its difficult installation process. This is due to the complex interdependencies of the packages required for OpenFOAM. Containers were expected to be a solution, but currently, they cannot replace all applications in the HPC field. However, I am still keenly interested in container technology for HPC. Technologies like Apptainer (Singularity), CharlieCloud, Shifter, Sarus, Podman exist in this realm, and exploring their differences could be very interesting.

#### Slide 3
An important aspect of the installation process is that performance can vary significantly based on compilation options. For example, using different compilation options for the open-source GROMACS on Intel Cascade Lake showed a performance difference of up to 70%. I conducted similar tests with OpenFOAM, but contrary to expectations, there was no significant difference. This seems to stem from differences in software code structure and analyzing the reasons for these performance differences is an important research topic. Therefore, I plan to present on performance testing at OKUCC in 2024, but performance analysis is twice as challenging as the effort put into this presentation, so I might give up on it.

#### Slide 4
The installation issues experienced with OpenFOAM are common across many open-source software. Hence, [Spack](https://spack.io), an open-source package manager, was developed by LLNL in the USA. Spack is undergoing active development and has over 7000 packages registered. It is widely used in several supercomputer centers abroad, but it seems less used in Korea.

#### Slide 5
Installing Spack is straightforward, involving a simple cloning process using Git. Installing OpenFOAM with Spack is very easy, and you can set detailed options using identifiers like '@', '%', '+', '^'. This ease of use is one of the major advantages of Spack.

#### Slides 6-7
Running the command ‘spack info openfoam’ provides a brief description of OpenFOAM, supported versions, and installation options. In the Variants section, you can see various detailed options that can be applied to OpenFOAM. The Dependencies section lists the prerequisite packages required for installing OpenFOAM. Pre-requisite packages like boost, cgal, paraview can also be installed easily through Spack, which is a significant advantage.

#### Slide 8
How does Spack handle all these processes in the background? Spack has a separate recipe file for each package, written in a DSL (Domain Specific Language), using Python. This recipe file allows setting dependencies and installation logic. Writing a Spack recipe file requires consulting Spack's official documentation and understanding its key syntax.

#### Slide 9
Let's take a look at how the recipe file for OpenFOAM is written. There are mainly three distributions of OpenFOAM: the distribution provided by [OpenCFD](https://www.openfoam.com), the one by the [OpenFOAM Foundation](https://www.openfoam.org), and the [Foam-extend](https://sourceforge.net/projects/foam-extend/) distribution led by Hrvoje Jasak. In Spack, each distribution is separated into different packages with their own recipe files. Around 2017, the recipe for other distributions was written based on the OpenCFD distribution. The OpenCFD distribution was well-maintained, but the others were not.

#### Slide 10
Therefore, I tried to improve the recipe for the other OpenFOAM Foundation distribution. There are various ways to contribute, and I chose to make a PR (Pull Request) to contribute directly to the source code. There were a total of five PR cases, as follows:

1. Version URL Function
2. Precision Options
3. Decomposition Method (Zoltan)
4. New Solver
5. Etc.

#### Slides 11-12
I will present a simple case first. In Spack, you write download URLs for each package version. On the left side of the slide, you can see that there is a certain rule for the download URL of OpenFOAM. The only rule that changes is that for versions below 5.0, the rule ‘5.x’ is used. I intended to simplify the code by creating a function instead of writing a URL for each version. Spack has a function named url_for_version that allows setting special rules for a package. Initially, I expected it to pass easily as the changed part was very simple, but it received negative reviews. The first version of the package, 2.3.1, did not use the GitHub URL but used the SourceForge URL, which was not included in the function. Therefore, I added rules where version 2.3.1 uses the SourceForge URL, and versions below 5.0 follow the ‘x’ rule.

#### Slide 13
The second case involves precision. OpenFOAM natively offers single and double precision. Now, methods beyond these two precisions are being developed. One is mixed precision, combining single and double precision, using double precision only where needed for accuracy, thereby saving on file size and computation time. The second is long double precision, which extends accuracy beyond double precision. Different distributions have different settings for these, which I have summarized in a table. The OpenCFD distribution includes mixed precision starting from the 1906 version and is registered in Spack. The Foundation distribution includes long double precision starting from the 6.0 version released in 2018, but it is not registered in Spack. The Foam-extend distribution has long double precision starting from the 3.2 version released in 2015, but again, it is not registered in Spack.

#### Slide 14
Reflecting this in the code is not difficult, but it is more important to find out from which version these options are applicable. Searching the commit history or checking every commit is one way but cumbersome. Using GitHub's Blame feature makes it easier. In OpenFOAM, precision settings are specified in the bashrc file, and using the Blame feature on this file can precisely identify which commit introduced the change. This allows us to know from which version the new precision method is applicable. For example, if you try to apply long double precision to OpenFOAM version 5.0 in the Foundation distribution, you will get an error message before installation.

#### Slide 15
The third case is about the decomposition method. Decomposition methods are used to divide the grid for parallel analysis. Besides commonly used methods like Metis and Scotch, new methods like Kahip and Zoltan are being developed. These are applied differently in each distribution. The OpenCFD distribution has applied the Kahip method and it is reflected in Spack. The Foundation distribution applied the Zoltan method in the version released in 2023, but it is not yet reflected in Spack. It is important to note that Zoltan is also used for renumbering methods, so existing packages might require the Zoltan package as well.

#### Slide 16
The Foundation distribution does not have the Zoltan package registered as a prerequisite, so it needs to be added. One thing to note is that it's not just the package that is needed, but a patch as well. The Zoltan package is required in its shared version (zoltan+shared). However, when installed in shared mode, the extension is a ‘.so’ file. In OpenFOAM, this is arbitrarily changed to an ‘.a’ extension. To align these mismatches, a patch is necessary.

#### Slides 17-18
The fourth case is about a new solver. Oak Ridge National Laboratory wanted to register a new package called [AdditiveFOAM](https://github.com/ExascaleAM/AdditiveFOAM) in Spack. This solver is for additive manufacturing. Their initial commit did not fully utilize Spack's DSL syntax. For additional packages like Additive Foam that use basic OpenFOAM as a pre-requisite package, the OpenCFD distribution created a script called spack-derived-Allwmake. Based on this script, a new build logic was created.

#### Slide 19
I also modified recipe files for other simulation codes. FDS is used for fire simulation and is written in Fortran. FDS was my first contribution to Spack and holds special significance for me. SU2 is a versatile CFD code widely used for aerodynamics analysis. SU2 uses a more modern build method called Meson, and learning about Meson was necessary to contribute to its package recipe. OpenRadioss is a Fortran-written structural analysis code. OpenRadioss has separate pre-processing and solver builds. Unlike OpenFOAM, which has its own pre-processing functionality but does not separate builds, OpenRadioss has separate builds for each. As a result, I decided to register separate packages in Spack for them. Typically, there is one Spack package per repository, but OpenRadioss is a case of two Spack packages in one repository.

#### Slide 20
I am registered as the package maintainer for FDS, SU2, and OpenRadioss in Spack. Now, I have decided to become the package maintainer for the OpenFOAM Foundation distribution. Although the above three codes are also excellent, I deliberated a lot about becoming the maintainer for OpenFOAM because it is a very widely used open-source code. Nevertheless, I decided to contribute because it helps me grow through these contributions.

