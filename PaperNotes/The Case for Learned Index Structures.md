---
标签:
  - paper
  -  open
Comments:
---
## Metadata
      
* Authors: [[Tim Kraska]], [[Alex Beutel]], [[Ed H. Chi]], [[Jeffrey Dean]], [[Neoklis Polyzotis]]      
* Date: [[2018-04-30]]   
* Topics: [[todo]]   
  

## Abstract

Indexes are models: a B-Tree-Index can be seen as a model to map a key to the position of a record within a sorted array, a Hash-Index as a model to map a key to a position of a record within an unsorted array, and a BitMap-Index as a model to indicate if a data record exists or not. In this exploratory research paper, we start from this premise and posit that all existing index structures can be replaced with other types of models, including deep-learning models, which we term learned indexes. The key idea is that a model can learn the sort order or structure of lookup keys and use this signal to effectively predict the position or existence of records. We theoretically analyze under which conditions learned indexes outperform traditional index structures and describe the main challenges in designing learned index structures. Our initial results show, that by using neural nets we are able to outperform cache-optimized B-Trees by up to 70% in speed while saving an order-of-magnitude in memory over several real-world data sets. More importantly though, we believe that the idea of replacing core components of a data management system through learned models has far reaching implications for future systems designs and that this work just provides a glimpse of what might be possible.


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

* [Local library](zotero://select/items/1_GFXFBLR4)    
* [Cloud library](http://zotero.org/users/12537825/items/GFXFBLR4)  
* URL: [http://arxiv.org/abs/1712.01208](http://arxiv.org/abs/1712.01208)  
    
* PDF Attachments
	- [Kraska 等 - 2018 - The Case for Learned Index Structures.pdf](zotero://open-pdf/library/items/7LKUNAZK)  
* Cite key: undefined