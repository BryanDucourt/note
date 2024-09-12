---
标签:
  - paper
  -  open
Comments:
---
## Metadata
* Item Type: [[Conference paper]]      
* Authors: [[J. Humphries]], [[Neel Natu]], [[Kostis Kaffes]], [[Stanko Novakovi'c]], [[Paul Turner]], [[Hank Levy]], [[David E. Culler]], [[Christos Kozyrakis]]      
* Date: [[30 August 2024]]   
* Topics: [[可编程内核]], [[todo]]   
  

## Abstract

The end of Moore's Law and the tightening performance requirements in today's clouds make re-architecting the software stack a necessity. To address this, cloud providers and vendors offload the virtualization control plane and data plane, along with the host OS data plane, to IPUs (SmartNICs), recovering scarce host resources that are then used by applications. However, the host OS control plane--encompassing kernel thread scheduling, memory management, the network stack, file systems, and more--is left on the host CPU and degrades workload performance. This paper presents Wave, a split OS architecture that moves OS subsystem policies to the IPU while keeping OS mechanisms on the host CPU. Wave not only frees host CPU resources, but it reduces host workload interference and leverages network insights on the IPU to improve policy decisions. Wave makes OS control plane offloading practical despite high host-IPU communication latency, lack of a coherent interconnect, and operation across two system images. We present Wave's design and implementation, and implement several OS subsystems in Wave, including kernel thread scheduling, the control plane for a network stack, and memory management. We then evaluate the Wave subsystems on Stubby (scheduling and network), our GCE VM service (scheduling), and RocksDB (memory management and scheduling). We demonstrate that Wave subsystems are competitive with and often superior to on-host subsystems, saving 8 host CPUs for Stubby, 16 host CPUs for database memory management, and improving VM performance by up to 11.2%.


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

* [Local library](zotero://select/items/1_N3LQNKBY)    
* [Cloud library](http://zotero.org/users/12537825/items/N3LQNKBY)  
* URL: [https://www.semanticscholar.org/paper/Wave%3A-A-Split-OS-Architecture-for-Application-Humphries-Natu/182b6647c9479f4572540f39a1fbe5a87f3f759d?utm_source=alert_email&utm_content=LibraryFolder&utm_campaign=AlertEmails_WEEKLY&utm_term=LibraryFolder&email_index=1-1-6&utm_medium=40272136](https://www.semanticscholar.org/paper/Wave%3A-A-Split-OS-Architecture-for-Application-Humphries-Natu/182b6647c9479f4572540f39a1fbe5a87f3f759d?utm_source=alert_email&utm_content=LibraryFolder&utm_campaign=AlertEmails_WEEKLY&utm_term=LibraryFolder&email_index=1-1-6&utm_medium=40272136)  
    
* PDF Attachments
	- [Full Text PDF](zotero://open-pdf/library/items/DNP27TV2)  
* Cite key: undefined