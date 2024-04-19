---
标签:
  - paper
Comments:
---
## Metadata
* Item Type: [[Article]]      
* Authors: [[Víctor Pérez]], [[Lukas Sommer]], [[Victor Lomüller]], [[Kumudha Narasimhan]], [[Mehdi Goli]]      
* Date: [[2023-06-30]]   
* Topics: [[可编程内核]]   
  

## Abstract
- 原文
	Heterogeneous programming models are becoming increasingly popular to support the ever-evolving hardware architectures, especially for new and emerging specialized accelerators optimizing specific tasks. While such programs provide performance portability of the existing applications across various heterogeneous architectures to some extent, short-running device kernels can affect an application performance due to overheads of data transfer, synchronization, and kernel launch. While in applications with one or two short-running kernels the overhead can be negligible, it can be noticeable when these short-running kernels dominate the overall number of kernels in an application, as it is the case in graph-based neural network models, where there are several small memory-bound nodes alongside few large compute-bound nodes. To reduce the overhead, combining several kernels into a single, more optimized kernel is an active area of research. However, this task can be time-consuming and error-prone given the huge set of potential combinations. This can push programmers to seek a tradeoff between (a) task-specific kernels with low overhead but hard to maintain and (b) smaller modular kernels with higher overhead but easier to maintain. While there are DSL-based approaches, such as those provided for machine learning frameworks, which offer the possibility of such a fusion, they are limited to a particular domain and exploit specific knowledge of that domain and, as a consequence, are hard to port elsewhere. This study explores the feasibility of a user-driven kernel fusion through an extension to the SYCL API to address the automation of kernel fusion. The proposed solution requires programmers to define the subgraph regions that are potentially suitable for fusion without any modification to the kernel code or the function signature. We evaluate the performance benefit of our approach on common neural networks and study the performance improvement in detail.
- 翻译
	异构编程模型随着硬件架构的不断演进而越来越受到重视，特别是在针对特定任务优化的新兴专用加速器方面。尽管这类程序在一定程度上能够实现现有应用在不同异构架构间的性能可移植，但数据传输、同步以及内核启动的开销可能会削弱短运行设备内核的应用性能。在只涉及一两个短运行内核的应用中，这种开销可能微不足道，但如果这类内核在应用中占多数，如在基于图的神经网络模型中常见，其中包含多个小型内存受限节点和少数大型计算受限节点，那么这种开销就会变得相当明显。为了降低这种影响，将多个内核合并为一个更高效的单一内核，已成为研究的热点。但是，由于潜在组合众多，这项工作既耗时又容易出错，这使得程序员不得不在（a）特定任务的低开销但难以维护的内核与（b）模块化程度高、开销较大但易于维护的小型内核之间寻求平衡。虽然有基于 DSL 的方法，如机器学习框架所提供的，可以实现这种内核的融合，但这些方法通常限于特定领域，并且依赖于领域特定知识，不易迁移到其他领域。本研究通过扩展 SYCL API 探索了用户驱动的内核融合的可能性，旨在实现内核融合的自动化。我们提出的方案无需更改内核代码或函数签名，只需程序员指定可能适合融合的子图区域。我们还在常见神经网络上测试了该方法的性能提升，并详细分析了性能改善情况。 ^add5e3

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






* [Local library](zotero://select/items/1_CAQZFRVS)    
* [Cloud library](http://zotero.org/users/12537825/items/CAQZFRVS)  
* URL: [https://dl.acm.org/doi/10.1145/3571284](https://dl.acm.org/doi/10.1145/3571284)  
* DOI: [10.1145/3571284](https://doi.org/10.1145/3571284)    
* PDF Attachments
	- [Pérez 等 - 2023 - User-driven Online Kernel Fusion for SYCL.pdf](zotero://open-pdf/library/items/FJWGV24F)  
