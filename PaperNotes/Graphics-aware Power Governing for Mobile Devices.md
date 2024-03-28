---
标签:
  - paper
Comments:
---
## Metadata
* Item Type: [[Conference paper]]      
* Authors: [[Yonghun Choi]], [[Seonghoon Park]], [[Hojung Cha]]      
* Date: [[2019-06-12]]   
   
  

## Abstract
## Abstract

Graphics increasingly play a key role in modern mobile devices. The graphics pipeline requires a close relationship between the CPU and the GPU to ensure energy efficiency and the user’s quality of experience (QoE). Our preliminary analysis showed that the current techniques employed to achieve energy efficiency in the Android graphics pipeline are not optimized especially in the frame generation process. In this paper, we aim to improve the energy efficiency of the Android graphics pipeline without degrading the user’s QoE. To achieve this goal, we studied the internals of the Android graphics pipeline and observed the energy inefficiency in the existing governing framework of the CPU and GPU. Based on the findings, we propose three techniques for addressing energy inefficiency: (1) aggressively capping the maximum CPU frequency, (2) lowering the CPU frequency by raising the GPU minimum frequency, and (3) allocating the frame rendering–related threads in the energyefficient CPU cores. These techniques are integrated into a single governing framework, called the GFX Governor, and implemented in the newest Android-based smartphones. Experimental results show that without hampering the user’s QoE the average energy consumption of Nexus 6P, Pixel XL, and Pixel 2 XL is reduced at the device level by 24.2%, 18.6%, and 13.7%, respectively, for the 60 chosen applications. We also analyzed the efficacy of the proposed technique in comparison with the state-of-the-art Energy-Aware Scheduling (EAS) implemented in the latest smartphone.


## 你对此篇文献有何评价？

>此问题的答案在笔记开头的Comments中回答


## 感兴趣的点



## 这篇论文的研究动机




## 这篇论文用到了哪些专业知识？阅读中你遇到了哪些问题？ 

- 知识
	- DVFS
	- UTIL-WALT

## 这篇论文的实验方案是如何设计的？ 




## 有哪些相关的研究？




## 这篇论文的主要贡献是什么？



## 这篇论文在写作表达方面有哪些值得借鉴的地方？





>[!hint] <center>Zotero links</center>
>
>* [Local library](zotero://select/items/1_SDDESKNH)    
>* [Cloud library](http://zotero.org/users/12537825/items/SDDESKNH)  
>* URL: [https://dl.acm.org/doi/10.1145/3307334.3326075](https://dl.acm.org/doi/10.1145/3307334.3326075)  
>* DOI: [10.1145/3307334.3326075](https://doi.org/10.1145/3307334.3326075)    
>* PDF Attachments
	- [Choi 等 - 2019 - Graphics-aware Power Governing for Mobile Devices.pdf](zotero://open-pdf/library/items/XIYNNHJS)  
>* Cite key: undefined