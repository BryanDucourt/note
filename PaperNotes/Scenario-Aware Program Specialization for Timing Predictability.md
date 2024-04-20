---
标签:
  - paper
  -  open
Comments:
---
## Metadata
* Item Type: [[Article]]      
* Authors: [[Joscha Benz]], [[Oliver Bringmann]]      
* Date: [[2021-12-31]]   
* Topics: [[可编程内核]]   
  

## Abstract

The successful application of static program analysis strongly depends on flow facts of a program such as loop bounds, control-flow constraints, and operating modes. This problem heavily affects the design of real-time systems, since static program analyses are a prerequisite to determine the timing behavior of a program. For example, this becomes obvious in worst-case execution time (WCET) analysis, which is often infeasible without user-annotated flow facts. Moreover, many timing simulation approaches use statically derived timings of partial program paths to reduce simulation overhead. Annotating flow facts on binary or source level is either error-prone and tedious, or requires specialized compilers that can transform source-level annotations along with the program during optimization. To overcome these obstacles, so-called
              scenarios
              can be used. Scenarios are a design-time methodology that describe a set of possible system parameters, such as image resolutions, operating modes, or application-dependent flow facts. The information described by a scenario is unknown in general but known and constant for a specific system. In this article,
              1
              we present a methodology for scenario-aware program specialization to improve timing predictability. Moreover, we provide an implementation of this methodology for embedded software written in C/C++. We show the effectiveness of our approach by evaluating its impact on WCET analysis using almost all of TACLeBench–achieving an average reduction of WCET of 31%. In addition, we provide a thorough qualitative and evaluation-based comparison to closely related work, as well as two case studies.


## 你对此篇文献有何评价？

>此问题的答案在笔记开头的Comments中回答


## 感兴趣的点



## 这篇论文的研究动机




## 这篇论文用到了哪些专业知识？阅读中你遇到了哪些问题？ 
### 问题
### 知识


## 这篇论文的实验方案是如何设计的？ 




## 有哪些相关的研究？




## 这篇论文的主要贡献是什么？



## 这篇论文在写作表达方面有哪些值得借鉴的地方？





Zotero links

* [Local library](zotero://select/items/1_ARIELKTP)    
* [Cloud library](http://zotero.org/users/12537825/items/ARIELKTP)  
* URL: [https://dl.acm.org/doi/10.1145/3473333](https://dl.acm.org/doi/10.1145/3473333)  
* DOI: [10.1145/3473333](https://doi.org/10.1145/3473333)    
* PDF Attachments
	- [Benz 和 Bringmann - 2021 - Scenario-Aware Program Specialization for Timing P.pdf](zotero://open-pdf/library/items/5UQ2B47J)  
