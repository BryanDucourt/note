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


## 可能的方向

CPU和GPU的联合调度
- 现有的方法只考虑GPU/CPU？；使用的模型比较简单？
- GPU的调度只考虑GPU本身的能效，不考虑任务的整体能效
- GPU负载的预测

考虑不同类型负载的计算特征
- 访存/CPU计算/GPU计算 比例
- 任务是否有周期性/受信号影响vsync、etc

不同的CPU核心
- 同类型的任务在不同核心上的收益/能效


## 研究动机




## 用到了哪些专业知识？遇到了哪些问题？ 
### 知识
- [[DVFS]]
- [[SOC]]
- [[UTIL]]
- [[pipline]]
### 问题
-   实验针对的应用类型
>researchers tried to manage the power issue for the CPU and the GPU, but they mostly focused on game applications, and thus, not optimized for general applications. Game applications generate frames continuously whereas general applications tend to produce frames intermittently depending on user interactions. Moreover, games typically use a different rendering pipeline than general Android applications.

绝大多数为轻负载类型的应用，相邻帧之间的变化不显著
- 使用的假设
	- 任务执行时间与频率成反比
	- 负载恒定
- 没有考虑到不同类型负载的特征
## 实验方案是如何设计的？ 
- 测量帧渲染时间，发现帧生成速度过快
	- 性能冗余，能源效率低下



## 有哪些相关的研究？




## 主要贡献是什么？

- 根据VSync剩余时间调整CPU频率
- 根据帧渲染的工作负载调整GPU频率
- 小核簇上渲染

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