---
sci_agent_schema: 1
entity_type: paper
entity_id: 1667ebc4-beef-4de0-b3b2-166597b2c867
project_id: 8f7c11cb-b5a9-4c80-8641-5caddea791d4
revision_id: bfd03bda-5b5b-43dc-993e-335386ebc843
owner: system
---

# Evaluating Very Long-Term Conversational Memory of LLM Agents

LOCOMO以人格、因果事件图和记忆机制生成超长期对话，经人工整理后构成三类评测。长上下文改善总体问答表现，同时暴露对抗题和复杂推理困难；事实观察检索在问答及多模态生成中表现较好，增加检索量后的收益减弱。GPT-4-turbo在事件摘要中领先，事实覆盖仍有改善空间。结论适用于本文生成数据、所测模型和单次推理设置，真实会话与长期视觉一致性仍待进一步研究。

## 研究问题

如何评估LLM代理在跨数月、多会话、多模态对话中的事实记忆、时间与因果理解及叙事一致性；长上下文和不同RAG记忆表示分别带来哪些收益与代价？

## 方法

先从MSC扩写人格设定，再初始化三个独立事件并迭代生成具有因果连接的时间事件图。虚拟代理结合递进会话摘要、带来源轮次ID的事实观察、检索反思、当前会话和人格生成回复，并注入相邻会话之间发生的事件；图像描述也存入长期记忆。人工修订跨轮次一致性、图像相关性及事件图与对话的对齐。最终通过问答、事件摘要和多模态生成，比较有限上下文、长上下文与不同记忆表示的RAG。


## 数据

- LOCOMO：10段跨数月的对话；平均588.2轮、27.2个会话、16,618.1 tokens，最多32个会话；平均每会话21.6轮、每段91.2张图像。

- 问答基准：共1,986题，其中单跳841、多跳282、时间推理321、开放领域知识96、对抗问题446。

- 事件摘要：每段对话平均35.8个参考事件，参考摘要平均1,042.7 tokens。

- MSC：提供4—5句初始人格描述，由gpt-3.5-turbo配合一个示例扩写。

- 多模态训练数据：另行生成50段未经人工过滤的会话；MiniGPT-5从已有MMDialog检查点初始化，正文与附录分别使用微调和预训练措辞。


## 实验

- 问答比较Base、Long-context和RAG：Base截去超出窗口的早期历史；Long-context扩大可见历史；RAG使用DRAGON检索原始对话、事实观察或会话摘要。批次报告将回答指标记为F1，并记录检索召回率。

- 问答top-5观察与对话日志的总体得分分别为43.3和38.8，差4.5分；正文概述为约5%的改善。该列含义依据批次报告，当前原始表格片段缺少表头。

- 事件摘要比较Base与Long-context，使用前序摘要递进处理会话，并提供一个输入输出示例引导重要事件选择；作者未设置该任务的RAG实验。

- 事件摘要使用ROUGE和FactScore。批次报告标注GPT-4-turbo的ROUGE-L/FactScore F1为21.6/48.9，Llama-3增量摘要为19.2/37.8；原始表格片段保留这些数值但缺少表头。

- 多模态生成比较MiniGPT-5的历史对话、历史加摘要、历史加观察三类训练条件；表7摘要top-k为1/2/5，观察top-k为5/10/25，报告BLEU-1/2、ROUGE-L和MM-R。

- 定性错误分析涉及信息遗漏、幻觉、语用线索、说话者归属与事件重要性判断。


## 局限

- 基准仅包含10段人工整理的生成对话，真实用户、自然交流和更广泛人群上的适用性仍需研究。

- 网络图片缺少个人照片中的长期视觉一致性；问答和事件摘要采用图像描述输入，视觉能力评估范围受此设置约束。

- 生成管线仅为英语开发，并依赖付费闭源模型；跨语言和开源模型适配尚待验证。

- 每模型仅报告一次推理结果，材料未提供重复运行波动范围或显著性检验。

- LLM即使被要求简短作答也会产生冗长答案，给正确性评估带来困难。

- 作者未报告长上下文微调开源模型及滑动窗口模型的结果，比较范围受模型选择限制。

- 记忆集合符号存在内部不一致；检索讨论中的“reduce the signalto-noise (SNR) ratio”方向存在歧义。表7摘要标签有重复与空白，top-k与全局摘要的对应关系需要核对。

- 批次报告记录了Gemini版本、QA表Observation重复top-5行及百分比口径的差异；对应完整表头和行在当前原始片段中不齐全。


## 复现资料

- 可直接核实的设置：2024年5月的OpenAI、Gemini、Claude API及Huggingface；temperature=0、top-p=1；作者报告使用A6000服务器和FP32，每模型单次推理。

- DRAGON用于检索；MiniGPT-5采用MMDialog已有检查点，在50段自动生成会话上训练10个epoch，单张A6000约30小时，超参数沿用原代码库推荐值。

- 人格扩展使用gpt-3.5-turbo及单个上下文示例；原始片段将人格、事件图、摘要和观察提示分别定位到图5、6、8、9，事件图示例定位到图7。

- 原始片段明确作者计划公开生成管线代码。所附批次报告记录项目入口https://snap-research.github.io/locomo及CC BY-NC 4.0 DEED许可；当前原始片段未附相应地址和许可正文，发布状态未核验。

- 完整复现仍需确认具体API快照、依赖版本、随机种子、训练数据划分、完整提示及指标实现，并解决记忆符号、表格标签和百分比口径差异。


## 待研究问题

- 如何选择和组织事实观察，使检索增加时仍保持有效信息密度，并提高多跳及时间推理表现？

- 摘要压缩损失了哪些事件、时间、因果和说话者信息，这些损失如何影响高召回条件下的回答质量？

- 如何同时改善长上下文模型的总体问答和对抗题表现，减少幻觉及说话者归属错误？

- 递进摘要、事实观察、反思、事件注入与人工修订分别贡献了多少收益？

- 在真实长期会话、连续个人照片、多语言和开源模型上，现有配置排序是否保持？

- 扩大数据规模并重复运行后，多模态top-5观察的优势是否稳定；事实指标对时间和因果关系的覆盖是否充分？


## 关键论断

- **reported_result** · LOCOMO包含10段超长期对话，平均每段约600轮、27.2个会话、16,618.1 tokens，最多32个会话；配套问答基准包含1,986题。
  - 条件：适用于LOCOMO发布基准的统计；引言中的600轮和16K tokens为近似表述。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=cc5f7d35-4100-42e9-9ee7-e134e2d70aff)：With this pipeline, we create LOCOMO, a dataset of 10 very long-term dialogues, each consisting of 600 turns and 16K tokens on avg., over up to 32 sessions

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=979c36a2-a5b8-4265-b318-ac8c77464786)：| Avg. # sessions k . in conversation h                   | 27.2        |

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=979c36a2-a5b8-4265-b318-ac8c77464786)：| Avg. # tokens. conversation h                           | 16,618.1    |

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=cf237c3f-f709-46aa-828b-f1bbe0f28ad9)：| Total. # questions.                                     | 1,986       |


- **reported_result** · 生成流程以人格设定和迭代扩展的因果事件图驱动对话；短期记忆采用递进会话摘要，长期记忆保存说话者事实观察，观察关联来源轮次ID以追踪证据。
  - 条件：描述生成代理及其记忆表示；正文记忆集合符号存在不一致，功能解释采用附录的明确描述。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=080e123c-d5fb-4957-b19f-9d768aaf8e68)：Then, the base prompt is combined with the prompt for the iterative generation of events to continue generating events that are caused by one or more of the events that are already present in the graph.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=cbfe5399-1449-4aa6-8cd6-d01c62f8fa78)：The short-term memory is a summary of a session that is conditioned on the summary from a previous session.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=cbfe5399-1449-4aa6-8cd6-d01c62f8fa78)：The long-term memory is a database of observations about each speaker, that are essentially assertive statements about the speaker's persona and life.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=cbfe5399-1449-4aa6-8cd6-d01c62f8fa78)：In practice, the conversation is annotated with turn IDs for each turn, and the model is also instructed to indicate the turn IDs that directly contribute to each observation.


- **reported_result** · 人工编辑修改了近15%的对话轮次，删除或替换约19%的图像；编辑规范同时要求对话与事件一致，并从事件图删除未在对话出现的事件。
  - 条件：适用于基准数据的人工校验阶段；另行生成的50段多模态训练对话未经人工过滤。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=1459ed4d-c5c5-442f-bd10-bd89dd9d2911)：annotators edited nearly 15% of the dialog turns and removed or substituted approx. 19% images present in the LLM-generated dataset.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=dc8bb090-3b92-4d49-90fc-1eb8fc731dc4)：Edit the dialog to ensure that the details in the conversation are consistent with those given in the event for the session.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=cb92934a-ab96-42a7-a703-90676af73a67)：Remove any events from the event graph if they do not appear in the conversation.


- **reported_result** · 评测覆盖问答、事件摘要和多模态对话生成；问答进一步划分为单跳、多跳、时间、常识或世界知识、对抗五类。
  - 条件：问答与事件摘要使用文本及图像描述，多模态生成直接使用图像。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=7f96effa-c621-4dc3-816f-9acde9d89d7f)：We propose three tasks: question answering, event summarization and multimodal dialog generation to evaluate models' comprehension in very long-term dialogues.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=8b58e0f3-ae69-4c81-a84a-9d6464dfabac)：We classify questions into five distinct reasoning types to evaluate memory from multiple perspectives: single-hop, multi-hop, temporal, commonsense or world knowledge, and adversarial.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=79ffb3ac-1898-4b3e-8b97-fac008405278)：For the question-answering and event summarization tasks, we replace images in LOCOMO with their captions

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=79ffb3ac-1898-4b3e-8b97-fac008405278)：We use images directly for the multimodal dialog generation task only.


- **reported_result** · GPT-4-turbo问答总体得分51.6，人类基准为87.9；其对抗题得分为15.7%。GPT-3.5-turbo扩大上下文后总体表现提高，同时对抗题表现明显下降。
  - 条件：适用于文中所测模型和上下文配置的LOCOMO问答；模型结果来自单次推理。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=73ee2156-8ff0-41bd-9b95-afea64b85807)：Despite gpt-4-turbo emerging as the top-performing model with an overall score of 51.6, it notably lags behind the human benchmark of 87.9;

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=73ee2156-8ff0-41bd-9b95-afea64b85807)：gpt-4-turbo outperforms other approaches on overall performance, but its performance on adversarial questions drops to a mere 15.7%, as compared to 34.8% using gpt-3.5-turbo and 80.0% using llama-3-chat-70B with 4K context lengths.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=56b26596-f28a-43c4-bb87-48315c82bb78)：The overall performance of gpt-3.5-turbo increases with context length, mainly due to large improvements in single-hop and multi-hop scenarios, yet the performance on adversarial questions drops dramatically.


- **synthesis** · RAG收益取决于记忆表示与检索数量：top-5事实观察优于原始对话日志，增加观察数量后收益减弱；会话摘要的高召回与有限回答收益并存。
  - 条件：基于DRAGON检索和GPT-3.5-turbo实验；正文将top-5观察的收益概述为5%，摘要召回按相关会话粒度计算。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=56b26596-f28a-43c4-bb87-48315c82bb78)：There is a noticeable 5% improvement with gpt-3.5-turbo when the input is top 5 relevant observations instead of pure conversation logs.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=66e8b302-e63e-4249-b9fb-011271432042)：This improvement falters with an increase in the number of retrieved observations

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=66e8b302-e63e-4249-b9fb-011271432042)：Conversely, using session summaries as context does not significantly improve the performance despite high recall accuracies

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=b5347537-514a-4e6b-b5e0-29399cf36fde)：For summary-based RAG models, the recall accuracy is based on retrieving the summary of the relevant session(s).


- **reported_result** · GPT-4-turbo在事件摘要的ROUGE和FactScore上领先；Llama-3-70B-Instruct的4K增量摘要在ROUGE-L上的差距较小，在事实覆盖指标上的差距更大。
  - 条件：事件摘要实验；正文使用百分比表述，批次报告按表4数值计算的ROUGE-L与FactScore F1差距分别为2.4和11.1分。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=b93903af-c0c6-4c0d-9455-aadce6bca462)：gpt-4-turbo leads to the highest scores in terms of ROUGE and FactScore metrics, followed by gemini-1.0-pro and claude-3-sonnet .

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=b93903af-c0c6-4c0d-9455-aadce6bca462)：The use of incremental summarization with Llama-3-70B-Instruct (4K context window) performs reasonably well compared to the long-context models, demonstrating only a 2.4% drop in ROUGE-L score. However, there is a nearly 10% drop in performance on the FactScore metric


- **reported_result** · 多模态生成中，top-5观察配置取得表7最高BLEU-1/2、ROUGE-L和MM-R：58.7/32.2、12.6、55.8；增加观察数量至10和25时指标逐步下降。作者还报告RAG部分缓解了历史增长导致的MM-Relevance下降。
  - 条件：MiniGPT-5使用50段未经人工过滤的生成对话训练；限定于表7配置及单次推理结果。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=8a5c672c-52db-40d2-9b88-640b02f2537b)：| Base                | -        | 56.4 / 31.8 |      11.6 |   54.2 |

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=8a5c672c-52db-40d2-9b88-640b02f2537b)：| + observation       | 5        | 58.7 / 32.2 |      12.6 |   55.8 |
  | + observation       | 10       | 58.1 / 32.1 |      12.0 |   55.1 |
  | + observation       | 25       | 57.8 / 31.6 |      11.8 |   54.9 |

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=a9355b49-05e1-4f1f-b597-982d900c6f1d)：Also, we observe that the MM-Relevance score drops with an increase in the length of dialog history (see Figure 4B). Retrieval-augmented generation alleviates the drop in MM-Relevance to some extent.


- **reported_result** · 事件摘要错误包括关键信息遗漏、幻觉或事件细节混合、对话线索误解、说话者归属错误，以及将次要互动判断为重要事件。
  - 条件：定性错误分类与示例；材料未给出各类错误发生频率。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=f8e44436-2a46-49be-a7a1-61372d0e1adf)：Key details about event are omitted because the model fails to make causal and temporal connections over a long conversation.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=0b14281f-86a3-46e4-9568-3a092b068843)：Non-existent details or details from a different event are padded onto an event

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=0b14281f-86a3-46e4-9568-3a092b068843)：e.g., model confuses a light-hearted statement from a speaker as a serious statement

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=9de683a6-f129-4360-aa13-37c84bcb8758)：Event is attributed to the wrong speaker

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=3d775b99-585d-4f3f-a3ec-1fbb2cfcf1b8)：Unimportant interactions in the conversation are considered significant by model


- **reported_result** · 作者指出生成对话对真实交流细节的覆盖有限，网络图片缺少人物与环境的长期视觉一致性；除OCR场景外，图片通常可用描述替代且信息损失较小。
  - 条件：适用于以生成文本和网络图片构建的LOCOMO。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=40e3cfea-feb2-4334-94c5-397b87bb9dd9)：However, we acknowledge that this dataset may not fully reflect the nuances of real-world online conversations.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=c4a47431-f159-48d7-ab5e-5b06ba1d4180)：Since the images in our dataset are sourced from the web, they do not demonstrate the visual long-term consistencies that are usually exhibited in personal photos (e.g., appearance, home environment, people and pets, etc.).

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=c4a47431-f159-48d7-ab5e-5b06ba1d4180)：Consequently, we find that the images in our dataset can be replaced with their captions without much loss of information, except for cases where OCR is required.


- **reported_result** · 评估设置为temperature=0、top-p=1，每模型报告单次推理结果；MiniGPT-5沿用原代码库推荐超参数，训练10个epoch，单张A6000约耗时30小时。
  - 条件：作者报告的2024年5月实验环境；置信度针对设置记录。

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=ccb37a30-fb3f-4986-ad84-83fe286dbd4c)：We use OpenAI API, Gemini API, Claude API and Huggingface (Wolf et al., 2020), as of May 2024, with specific settings of temperature set to 0 and top p set to 1 for evaluation of the LOCOMO benchmark.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=ccb37a30-fb3f-4986-ad84-83fe286dbd4c)：We report results from a single inference run for each model in our experiments.

  - [原文证据](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867?evidence=ccb37a30-fb3f-4986-ad84-83fe286dbd4c)：For MiniGPT-5, we used the hyperparameters recommended in the original codebase and trained our models for 10 epochs, which took approximately 30 hours on a single A6000 GPU.



来源覆盖：fulltext。

[在研究工作台查看](http://127.0.0.1:8765/projects/8f7c11cb-b5a9-4c80-8641-5caddea791d4/literature/1667ebc4-beef-4de0-b3b2-166597b2c867)

人工补充请写入同目录的 [[Notes.user]]。
