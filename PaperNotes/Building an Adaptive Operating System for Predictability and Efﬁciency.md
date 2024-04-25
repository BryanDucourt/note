---
标签:
  - paper
  -  open
Comments:
---
## Metadata
* Item Type: [[Article]]      
* Authors: [[Gage Eads]], [[Juan A Colmenares]], [[Steven Hofmeyr]], [[Sarah Bird]], [[Davide B Bartolini]], [[David Chou]], [[Brian Gluzman]], [[Krste Asanovic]], [[John D Kubiatowicz]]      
   
* Topics: [[可编程内核]]   
  

## Abstract

We address the system-level challenge of supporting a dynamically changing, complex mix of simultaneously running applications with diverse requirements including responsiveness, throughput, and performance guarantees. In our approach, called Adaptive Resource Centric Computing (ARCC), the OS distributes resources to QoS domains called cells, which are explicitly parallel lightweight containers with guaranteed, user-level access to resources. The resource allocations are dynamically adjusted whenever applications change state or the mix of applications changes. This paper gives explicit details about our implementation of ARCC on Tessellation OS, an experimental platform for resource management on multicore-based computers. We discuss the implementation of cells, user-level scheduling, and resource allocation policies. Our running example is a realistic video conferencing scenario that incorporates parallelism, QoS guarantees, and dynamic optimization with two-level scheduling. Our system reduces reserved CPU bandwidth to 69.95% of that of a static allocation, while still meeting performance and QoS targets.


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

* [Local library](zotero://select/items/1_SSEYGLHR)    
* [Cloud library](http://zotero.org/users/12537825/items/SSEYGLHR)  
  
    
* PDF Attachments
	- [Eads 等 - Building an Adaptive Operating System for Predicta.pdf](zotero://open-pdf/library/items/HYZBD8LL)  
