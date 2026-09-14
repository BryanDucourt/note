---
sci_agent_schema: 1
entity_type: paper
entity_id: 93be4a7f-7c7f-4181-b762-5684aadb985a
project_id: 8f7c11cb-b5a9-4c80-8641-5caddea791d4
revision_id: 9c00b008-d9ce-4ad4-b5d0-77aff175e076
owner: system
---

# Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory

本批摘要介绍 Mem0 及其图记忆增强变体。作者报告在 LOCOMO 上取得回答质量收益，并相对全文上下文方案显著降低延迟与 token 成本。材料提供了主要结果和代码入口，实验配置与实现细节仍待全文补充。

## 研究问题

如何通过可扩展的长期记忆机制，缓解 LLM 固定上下文窗口对多会话对话一致性的限制，并兼顾回答质量、延迟与 token 成本？

## 方法

Mem0 采用以记忆为核心的架构，动态提取、整合和检索持续对话中的重要信息；增强变体利用图记忆表示对话元素间的复杂关系。


## 数据

- LOCOMO：摘要中唯一明确命名的评估基准；未提供数据规模、划分和预处理方式。


## 实验

- 在 LOCOMO 上将提出的方法与六类基线比较；完整基线清单未提供。

- 评估单跳、时间、多跳和开放域四类问题。

- 报告相对 OpenAI 的 LLM-as-a-Judge 指标提升，以及图记忆变体相对基础 Mem0 的总体得分提升。

- 与全文上下文方案比较 p95 延迟和 token 成本。


## 局限

- 当前材料仅为三个摘要片段，来源质量标记为 limited；结果解读限于摘要覆盖范围。

- 未提供记忆提取、整合、检索和图表示的具体算法及实现参数。

- 未提供绝对分数、样本量、统计不确定性、评审模型配置或完整基线设置。

- 未提供延迟测试环境、成本核算范围及不同对话长度下的表现。


## 复现资料

- 摘要提供代码入口：https://mem0.ai/research；当前材料未包含链接内容。

- 复现所需的代码版本、模型版本、提示词、超参数、数据划分和运行环境均未在片段中给出。


## 待研究问题

- 记忆提取、整合与检索各自对质量和效率提升贡献多大？

- 图记忆约 2% 的总体得分提升主要来自哪些问题类别，其额外资源开销如何？

- 在更长对话、不同领域及记忆冲突场景下，报告的收益如何变化？


## 关键论断

- **reported_result** · Mem0 通过动态提取、整合和检索对话中的重要信息，支持长期对话记忆。
  - 条件：摘要对架构的描述；面向持续、多会话对话。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a?evidence=d6e35556-c7d6-4202-96b2-74d81629f937)：We introduce Mem0, a scalable memory-centric architecture that addresses this issue by dynamically extracting, consolidating, and retrieving salient information from ongoing conversations.


- **reported_result** · 作者提出图记忆增强变体，以表示对话元素之间的复杂关系。
  - 条件：摘要中的图记忆变体设计描述。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a?evidence=d6e35556-c7d6-4202-96b2-74d81629f937)：Building on this foundation, we further propose an enhanced variant that leverages graph-based memory representations to capture complex relational structures among conversational elements.


- **reported_result** · 作者在 LOCOMO 上与六类基线比较，并报告其方法在单跳、时间、多跳和开放域四类问题上均优于参与比较的现有记忆系统。
  - 条件：LOCOMO 评估及摘要所述比较范围；具体基线与分类分数未提供。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a?evidence=d6e35556-c7d6-4202-96b2-74d81629f937)：Through comprehensive evaluations on the LOCOMO benchmark, we systematically compare our approaches against six baseline categories.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a?evidence=dcd18b70-fb71-471d-ba52-c5414b55483c)：Empirical results demonstrate that our methods consistently outperform all existing memory systems across four question categories: single-hop, temporal, multi-hop, and open-domain.


- **reported_result** · 作者报告，Mem0 的 LLM-as-a-Judge 指标相对 OpenAI 提升 26%；图记忆变体的总体得分较基础 Mem0 高约 2%。
  - 条件：摘要所述 LOCOMO 实验；OpenAI 比较配置及图变体约 2% 提升的计算口径未说明。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a?evidence=dcd18b70-fb71-471d-ba52-c5414b55483c)：Notably, Mem0 achieves 26% relative improvements in the LLM-as-a-Judge metric over OpenAI, while Mem0 with graph memory achieves around 2% higher overall score than the base Mem0 configuration.


- **reported_result** · 作者报告，Mem0 相对全文上下文方案降低 91% 的 p95 延迟，并节省超过 90% 的 token 成本。
  - 条件：相对于全文上下文方案；测量环境、负载及成本核算范围未提供。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a?evidence=dcd18b70-fb71-471d-ba52-c5414b55483c)：Beyond accuracy gains, we also markedly reduce computational overhead compared to the full-context approach. In particular, Mem0 attains a 91% lower p95 latency and saves more than 90% token cost, thereby offering a compelling balance between advanced reasoning capabilities and practical deployment constraints.



来源覆盖：abstract。

[在研究工作台查看](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/93be4a7f-7c7f-4181-b762-5684aadb985a)

人工补充请写入同目录的 [[Notes.user]]。
