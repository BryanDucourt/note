---
标签:
  - paper
  -  open
Comments:
---
## Metadata
      
* Authors: [[Michael Ahn]], [[Anthony Brohan]], [[Noah Brown]], [[Yevgen Chebotar]], [[Omar Cortes]], [[Byron David]], [[Chelsea Finn]], [[Chuyuan Fu]], [[Keerthana Gopalakrishnan]], [[Karol Hausman]], [[Alex Herzog]], [[Daniel Ho]], [[Jasmine Hsu]], [[Julian Ibarz]], [[Brian Ichter]], [[Alex Irpan]], [[Eric Jang]], [[Rosario Jauregui Ruano]], [[Kyle Jeffrey]], [[Sally Jesmonth]], [[Nikhil J. Joshi]], [[Ryan Julian]], [[Dmitry Kalashnikov]], [[Yuheng Kuang]], [[Kuang-Huei Lee]], [[Sergey Levine]], [[Yao Lu]], [[Linda Luu]], [[Carolina Parada]], [[Peter Pastor]], [[Jornell Quiambao]], [[Kanishka Rao]], [[Jarek Rettinghouse]], [[Diego Reyes]], [[Pierre Sermanet]], [[Nicolas Sievers]], [[Clayton Tan]], [[Alexander Toshev]], [[Vincent Vanhoucke]], [[Fei Xia]], [[Ted Xiao]], [[Peng Xu]], [[Sichun Xu]], [[Mengyuan Yan]], [[Andy Zeng]]      
* Date: [[2022-08-16]]   
* Topics: [[robot+llm]]   
  

## Abstract

Large language models can encode a wealth of semantic knowledge about the world. Such knowledge could be extremely useful to robots aiming to act upon high-level, temporally extended instructions expressed in natural language. However, a signiﬁcant weakness of language models is that they lack real-world experience, which makes it difﬁcult to leverage them for decision making within a given embodiment. For example, asking a language model to describe how to clean a spill might result in a reasonable narrative, but it may not be applicable to a particular agent, such as a robot, that needs to perform this task in a particular environment. We propose to provide real-world grounding by means of pretrained skills, which are used to constrain the model to propose natural language actions that are both feasible and contextually appropriate. The robot can act as the language model’s “hands and eyes,” while the language model supplies high-level semantic knowledge about the task. We show how low-level skills can be combined with large language models so that the language model provides high-level knowledge about the procedures for performing complex and temporally extended instructions, while value functions associated with these skills provide the grounding necessary to connect this knowledge to a particular physical environment. We evaluate our method on a number of real-world robotic tasks, where we show the need for real-world grounding and that this approach is capable of completing long-horizon, abstract, natural language instructions on a mobile manipulator. The project’s website, the video, and open sourced code in a tabletop domain can be found at say-can.github.io.


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

* [Local library](zotero://select/items/1_CH89JT8Z)    
* [Cloud library](http://zotero.org/users/12537825/items/CH89JT8Z)  
* URL: [http://arxiv.org/abs/2204.01691](http://arxiv.org/abs/2204.01691)  
* DOI: [10.48550/arXiv.2204.01691](https://doi.org/10.48550/arXiv.2204.01691)    
* PDF Attachments
	- [Ahn 等 - 2022 - Do As I Can, Not As I Say Grounding Language in R.pdf](zotero://open-pdf/library/items/SQ7V49UI)  
* Cite key: undefined