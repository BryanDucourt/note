# User-driven Online Kernel Fusion for SYCL

## Metadata

* Item Type: [[Article]]
* Authors: [[Víctor Pérez]], [[Lukas Sommer]], [[Victor Lomüller]], [[Kumudha Narasimhan]], [[Mehdi Goli]]
* Date: [[2023-06-30]]
* Date Added: [[2024-04-18]]
* URL: [https://dl.acm.org/doi/10.1145/3571284](https://dl.acm.org/doi/10.1145/3571284)
* DOI: [10.1145/3571284](https://doi.org/10.1145/3571284)
* Topics: [[可编程内核]]
, #zotero, #literature-notes, #reference
* PDF Attachments
	- [Pérez 等 - 2023 - User-driven Online Kernel Fusion for SYCL.pdf](zotero://open-pdf/library/items/FJWGV24F)

## Abstract

Heterogeneous programming models are becoming increasingly popular to support the ever-evolving hardware architectures, especially for new and emerging specialized accelerators optimizing specific tasks. While such programs provide performance portability of the existing applications across various heterogeneous architectures to some extent, short-running device kernels can affect an application performance due to overheads of data transfer, synchronization, and kernel launch. While in applications with one or two short-running kernels the overhead can be negligible, it can be noticeable when these short-running kernels dominate the overall number of kernels in an application, as it is the case in graph-based neural network models, where there are several small memory-bound nodes alongside few large compute-bound nodes.
To reduce the overhead, combining several kernels into a single, more optimized kernel is an active area of research. However, this task can be time-consuming and error-prone given the huge set of potential combinations. This can push programmers to seek a tradeoff between (a) task-specific kernels with low overhead but hard to maintain and (b) smaller modular kernels with higher overhead but easier to maintain. While there are DSL-based approaches, such as those provided for machine learning frameworks, which offer the possibility of such a fusion, they are limited to a particular domain and exploit specific knowledge of that domain and, as a consequence, are hard to port elsewhere. This study explores the feasibility of a user-driven kernel fusion through an extension to the SYCL API to address the automation of kernel fusion. The proposed solution requires programmers to define the subgraph regions that are potentially suitable for fusion without any modification to the kernel code or the function signature. We evaluate the performance benefit of our approach on common neural networks and study the performance improvement in detail.




##  Zotero links
* [Local library](zotero://select/items/1_CAQZFRVS)
* [Cloud library](http://zotero.org/users/12537825/items/CAQZFRVS)

