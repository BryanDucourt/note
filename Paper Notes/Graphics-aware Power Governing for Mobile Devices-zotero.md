# Graphics-aware Power Governing for Mobile Devices

## Metadata

* Item Type: [[Conference paper]]
* Authors: [[Yonghun Choi]], [[Seonghoon Park]], [[Hojung Cha]]
* Proceedings Title: [[Proceedings of the 17th Annual International Conference on Mobile Systems, Applications, and Services]]
* Date: [[2019-06-12]]
* Date Added: [[2024-03-26]]
* URL: [https://dl.acm.org/doi/10.1145/3307334.3326075](https://dl.acm.org/doi/10.1145/3307334.3326075)
* DOI: [10.1145/3307334.3326075](https://doi.org/10.1145/3307334.3326075)
, #zotero, #literature-notes, #reference
* PDF Attachments
	- [Choi 等 - 2019 - Graphics-aware Power Governing for Mobile Devices.pdf](zotero://open-pdf/library/items/XIYNNHJS)

## Abstract

Graphics increasingly play a key role in modern mobile devices. The graphics pipeline requires a close relationship between the CPU and the GPU to ensure energy efficiency and the user’s quality of experience (QoE). Our preliminary analysis showed that the current techniques employed to achieve energy efficiency in the Android graphics pipeline are not optimized especially in the frame generation process. In this paper, we aim to improve the energy efficiency of the Android graphics pipeline without degrading the user’s QoE. To achieve this goal, we studied the internals of the Android graphics pipeline and observed the energy inefficiency in the existing governing framework of the CPU and GPU. Based on the findings, we propose three techniques for addressing energy inefficiency: (1) aggressively capping the maximum CPU frequency, (2) lowering the CPU frequency by raising the GPU minimum frequency, and (3) allocating the frame rendering–related threads in the energyefficient CPU cores. These techniques are integrated into a single governing framework, called the GFX Governor, and implemented in the newest Android-based smartphones. Experimental results show that without hampering the user’s QoE the average energy consumption of Nexus 6P, Pixel XL, and Pixel 2 XL is reduced at the device level by 24.2%, 18.6%, and 13.7%, respectively, for the 60 chosen applications. We also analyzed the efficacy of the proposed technique in comparison with the state-of-the-art Energy-Aware Scheduling (EAS) implemented in the latest smartphone.


##  Zotero links
* [Local library](zotero://select/items/1_SDDESKNH)
* [Cloud library](http://zotero.org/users/12537825/items/SDDESKNH)

