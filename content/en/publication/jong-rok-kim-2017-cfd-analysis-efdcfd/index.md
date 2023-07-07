---
title: CFD Analysis of EFD-CFD Workshop Case 3 Using Commercial and Open Source CFD
  Codes
date: '2017-01-01'
draft: false
publishDate: '2023-06-05T10:35:56.371447Z'
authors:
- Jong Rok Kim
publication_types:
- '2'
abstract: Computational fluid dynamics analysis was performed for the case 3 of the
  EFD-CFD workshop. Solvers were used for three commercial CFD codes(Star-CCM+, Fluent
  and CFX) and an open source CFD code(SU2). The grid were generated four types depending
  on the total cells using commercial grid generation code(Pointwise). Mach number
  of 0.4 and 0.8, 2 degree angle of attack and Mach number of 0.9, 1 degree angle
  of attack were calculated. Similar pressure coefficient curve and normal force coefficient
  were showed from the coarse grid to fine grid of four codes. But there is a difference
  in the drag coefficient. The position of the shock wave was predicted forward as
  the discretization order increased in calculations using Star-CCM+ and Fluent. The
  computation time to converge, Fluent, Star-CCM +, CFX are in order, and SU2 takes
  much time to converge.
featured: false
publication: '*Journal of the Korean Society for Aeronautical & Space Sciences*'
tags:
- CFD
- CFX
- EFD-CFD
- Fluent
- RAE-A
- Star-CCM+
- SU2
---

Paper Link: https://scienceon.kisti.re.kr/commons/util/originalView.do?cn=JAKO201713842134307&oCn=JAKO201713842134307&dbt=JAKO&journal=NJOU00290662

The EFD-CFD workshop was organized under the leadership of professors and researchers from the Aerospace Research Institute. Considering the participants of the workshop, it is expected that institutions such as the Aerospace Research Institute and the Defense Science Institute will attend. These research institutes have highly skilled research personnel and possess in-house codes. Additionally, graduate students can be expected to participate as well.

In universities, the quality of papers and presentations may vary depending on the research lab. It is also anticipated that they will primarily utilize their in-house codes. The in-house codes of graduate programs have been refined over several years or even decades, so it is likely that their explanations will focus on the advantages of these in-house codes. On the other hand, it is also expected that representatives from commercial CFD software support companies will attend. They will claim that their commercial codes provide accurate interpretations.

With these anticipated participants, what choices did I have to make? Firstly, my company did not possess in-house codes at that time. However, unlike other institutions, we had the advantage of having access to various analysis software. Instead of purchasing only one commercial software for a specific field, our company procured multiple software. Therefore, my goal was to use as many codes as possible for comparison. It was evident that other institutions would interpret their papers using only one code per publication.

In this article, I would like to discuss the stories of the four CFD software packages featured in the papers.


# 1. Ansys Fluent
First and foremost, Ansys [Fluent](https://www.ansys.com/products/fluids/ansys-fluent) was the main software I used, and I was most confident in using it. Ansys acquired Fluent in 2006, and since then, "Ansys Fluent" has been its accurate full name. Fluent has maintained its position as the market leader in the CFD industry for decades. As its reputation suggests, it is capable of analyzing numerous fluid domains and possesses powerful parallel performance. In truth, even in 2017 when I wrote this paper, the solver was powerful, but the user interface (UI) was somewhat inconvenient. However, with yearly updates, the UI improved significantly, and by around 2020, it was not notably inconvenient. Ansys has continued to provide noticeable updates since 2020, and below are some key updates that are relevant and of great interest to me. Although each item could be discussed in detail, my current company does not have a valid license for Fluent, so I cannot perform a proper analysis using it.
![Figure by Ansys](fluent-2023r1-highlight1.webp)


## 1-1. Introduction of Python

In the field of CFD, where computational speed is crucial, most codes are implemented in core languages such as C++ or Fortran. However, if the code is solely developed in languages like C++, it becomes difficult to utilize the code for external tasks such as creating batch processes or generating tables. Therefore, it is common to combine script-based languages like Python or Perl. Typically, individuals who specialize in CFD are familiar with in-house CFD codes, but if they only use commercial software at work, they may lose touch with coding and their programming skills may deteriorate. However, to achieve an advanced level of proficiency with commercial software, one must learn the scripting language supported by the software to use it effectively. They will have to revisit and work on the code they had previously set aside (which may be an unfamiliar programming language), and some interpretation capabilities may decline.

Fluent had some issues with script support. Fluent's scripting language is [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)), which is a dialect of Lisp and a historically significant programming language. However, its current usage is quite low. Furthermore, CFD practitioners are not typically computer science majors, so the chances of them being proficient in functional programming languages like Scheme are close to zero. There is very little available information on the web as well. This was one of the few weaknesses of Fluent. However, with the recent introduction of Python support, Fluent has overcome this limitation. I have been using Python for quite some time and have a strong affinity for it. If I could continue working with Fluent, Python support would have been a game-changer for me. Detailed information on this topic can be found in the Ansys blog post titled  "[Providing Open-Source Access to Ansys Fluent with PyFluent]"
(https://www.ansys.com/blog/open-source-access-to-fluent-with-pyfluent)


## 1-2. GPU Support

In the field of AI/ML, the use of GPUs as accelerators has become commonplace. However, GPU acceleration has been challenging in the CFD field. To put it simply, when discussing CPU specifications, we typically mention architecture generation, clock speed, and core count. Servers used for CFD generally have clock speeds of 3.0 MHz or higher and use servers with 30 or more cores. GPUs are often talked about in terms of architecture generation and memory, but they also have their own cores. Nvidia refers to these cores as CUDA cores. For example, an Nvidia V100 has 5210 CUDA cores. Of course, the cores in CPUs and GPUs don't have the same roles or performance. If we were to make an analogy, CPU cores would be like university students, while GPU cores would be like elementary school students. If they were engaged in a tug-of-war, even if a university student had more strength, they wouldn't be able to beat 100 elementary school students. However, what if it was a calculus problem? Even if 100 or 1000 elementary school students were present, they wouldn't be able to solve it. Let's say that traditional CFD problems were like calculus problems. That's why the cores in GPUs, which correspond to elementary school students, couldn't perform well. To solve this, the problem had to be divided into smaller parts so that even elementary school students could handle it. However, until now, there were very few people who succeeded in modifying existing legacy codes in various ways. So the solution was to rewrite the code from scratch. In reality, the analogy I used earlier with elementary school students, university students, and calculus doesn't perfectly align with actual CFD numerical analysis methods. What I wanted to convey is that in order to achieve proper GPU acceleration, it seems that the CFD code had to be rewritten from the beginning. The detailed code for Fluent's GPU solver is not known, but they probably created a new code, which is why it took time to release it. Detailed information on this topic can be found in the Ansys blog post titled [Unleashing the Power of Multiple GPUs for CFD Simulations](https://www.ansys.com/blog/unleashing-the-power-of-multiple-gpus-for-cfd-simulations)


