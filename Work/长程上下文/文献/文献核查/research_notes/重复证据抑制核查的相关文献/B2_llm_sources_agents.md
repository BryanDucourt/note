# B2: LLM source weighting, multi-agent conformity, memory poisoning, and confidence-driven verification. Closest work to Paper 1 (novelty check, 2026-10-21)

Paper 1 claim, in short: one misperception gets copied by a single LLM write into several memory stores. The planner then sees it as if several sources agreed, becomes overconfident, verifies less, and the error persists into both progress judgments and next actions (BEHAVIOR-1K knife search; interventions on copy count and provenance labels, with images and context length held fixed).

How to read each entry. Each entry gives authors, year, venue, link, method and finding, then four tags:
- **P/D**: whether the work involves provenance or source-reliability labels (P), or duplicated or same-source evidence (D).
- **V**: whether it measures verification or information-seeking behaviour.
- **Overlap**: how close it is to Paper 1 (HIGH, MED or LOW), plus the key difference.
- **Checked**: what I verified. "abstract" means the arXiv abstract, pulled from the arXiv API; "full text" means I also read the arXiv HTML.

Venues come from the arXiv comment or journal field, the ACL Anthology, or the NeurIPS site. A venue I could not confirm is marked "venue unverified".

## 1. Source attribution and credibility: do LLMs weight evidence by source reliability or independence?

### Takeaway
LLMs do respond to explicit source labels, but they do not discount same-source repetition. Two copies of one low-credibility source can reverse their source preference (Whose Facts Win, ACL 2026). Paraphrases of one document move answers more than independent documents (Rational Synthesizers, 2026). RAG models also follow a "majority rule" by frequency (Jin et al. 2024). The closest conceptual precedent is Epistemic Sybil Resistance (Sept 2026): replicated reports from one evidence root collapse naive posterior coverage. But the overconfident aggregator there is a fixed formula, not an LLM, and nothing in that paper measures verification.

### Cited Findings
- **Epistemic Sybil Resistance: Multiplying AI Agents Without Multiplying Evidence.** Marc Bara (2026), arXiv preprint.
  - Method: formalises an "epistemic Sybil" report, one that adds zero information about the target given the other reports (I(Θ;Z|R)=0). It runs more than 20,000 controlled LLM-agent report and extraction calls on synthetic evidentiary documents.
  - Finding: with one evidence root held fixed, raising report multiplicity from 1 to 32 drops naive posterior coverage from 0.940 to 0.263. Raising evidence-root multiplicity from 1 to 16 at a fixed report count closes the gap. Replicate extraction errors are correlated (γ_cal = 0.719). Report-space deduplication tracks how similar reports look, not their ancestry (cluster count shifts by 1.425 for a rendering change versus 0.040 for a fourfold ancestry change). The paper concludes that inference "should therefore track evidential ancestry and dependence, not agent or report multiplicity".
  - Full-text check: the "naive posterior" is an explicit statistical aggregator that pools reports "as conditionally independent observations". No LLM acts as the aggregator or states a confidence. Ancestry is never shown to the model. No verification or information seeking is measured. The setup is one Haiku-class model estimating revenue from synthetic memos.
  - P/D: D yes (central); P handled outside the model. V: no.
  - Overlap: HIGH (concept). Difference: the overconfidence belongs to a formula aggregator, not to an LLM planner reading duplicated memory. There are no provenance labels shown to an LLM, no verification choices, no embodied task and no downstream action errors.
  - Checked: abstract and full text (first 100k of 114k characters).
  - [arXiv 2609.01873](https://arxiv.org/abs/2609.01873)
- **Whose Facts Win? LLM Source Preferences under Knowledge Conflicts.** Jakob Schuster, Vagrant Gautam, Katja Markert (2026), ACL 2026 long.
  - Method: synthetic source labels (government, newspaper, people, social media) attached to conflicting passages. Tested on 13 open-weight LLMs: Qwen2.5 (7–72B), OLMo-2 (7–32B), Llama-3.x (3–70B) and Gemma-3 (4–27B). Preference is read from normalised answer-token probabilities.
  - Finding: LLMs prefer institutionally corroborated sources, but "these source preferences can be reversed by simply repeating information from less credible sources".
  - Full-text check: in the repetition setting, "two identical ones are attributed to the same social media source", so this is same-source duplication. Only two copies are tested. A separate "2-Table Majority" condition uses two distinct sources.
  - Mitigation: teacher-student LoRA distillation on Gemma-3-4B, combined with a credibility-aware prompt. It cuts repetition bias by up to 79.2% while keeping at least 72.5% of the original preferences. These are the v3/ACL numbers; v1 reported 99.8% and 88.8%.
  - Not measured: calibration analysis, generations, or verification.
  - P/D: P yes, D yes. V: no.
  - Overlap: HIGH (mechanism). Difference: a static QA knowledge conflict with only two copies, no agent, no verification choices and no error persistence.
  - Checked: abstract and full text (partial).
  - [arXiv 2601.03746](https://arxiv.org/abs/2601.03746); [ACL Anthology 2026.acl-long.1357](https://aclanthology.org/2026.acl-long.1357/)
- **Rational Synthesizers or Heuristic Followers? Analyzing LLMs in RAG-based Question-Answering.** Atharv Naphade (2026), arXiv (ACL ARR submission).
  - Method: GroupQA (1,635 controversial questions, 15,058 documents). Compares "distinct" independent supporting documents with "paraphrased" rephrasings of one document, up to 10 documents, all opposing the model's prior.
  - Finding: "Paraphrasing an argument can be more persuasive than providing distinct independent support". Flip rates, paraphrased versus distinct:

    | Model | Paraphrased | Distinct |
    |---|---|---|
    | DeepSeek-R1-8B | 76.5% | 67.6% |
    | Gemini-2.5-FL | 75.6% | 63.7% |
    | Llama-3.1-70B | 69.8% | 62.9% |
    | Qwen3-32B | 73.7% | 67.3% |

  - Also found: primacy bias, and unfaithful explanations (Llama-3.1-70B faithful on 26%). The paper links the result to the illusory truth effect and suggests deduplication, without testing it.
  - P/D: D yes. V: no.
  - Overlap: HIGH–MED. Difference: RAG QA, no provenance labels, no agent or verification, and context length grows with copy count.
  - Checked: abstract and full text.
  - [arXiv 2601.06189](https://arxiv.org/abs/2601.06189)
- **Tug-of-War Between Knowledge.** Zhuoran Jin et al. (2024), LREC-COLING 2024.
  - Finding: RALMs "follow the principle of majority rule, leaning towards placing trust in evidence that appears more frequently". They also show confirmation bias toward evidence that matches internal memory, and an availability bias. Stronger models show a Dunning-Kruger-like persistence in faulty internal memory. The paper proposes CD2 contrastive decoding to calibrate confidence.
  - P/D: D (frequency) yes; P no. V: no.
  - Overlap: MED. Difference: frequency without any manipulation of source independence, and QA only.
  - Checked: abstract.
  - [arXiv 2402.14409](https://arxiv.org/abs/2402.14409)
- **How retriever redundancy and diversity impact RAG effectiveness.** Jonathan J. Ross, Bevan Koopman, Anton van der Vegt, Guido Zuccon (2026), arXiv.
  - Method: compares exact duplicates, LLM paraphrases and diverse-genre documents on FictionalQA, where prior knowledge cannot answer.
  - Finding: "duplicate redundancy and LLM paraphrasing does not significantly improve answer correctness". Diverse documents improve correctness by 17–47%, driven by genre diversity.
  - P/D: D yes. V: no. Only accuracy is measured, not confidence.
  - Overlap: MED (counterpoint). Difference: correct-evidence duplication, scored on accuracy only.
  - Checked: abstract.
  - [arXiv 2608.13956](https://arxiv.org/abs/2608.13956)
- **Adaptive Chameleon or Stubborn Sloth.** Jian Xie, Kai Zhang, Jiangjie Chen, Renze Lou, Yu Su (2024), ICLR 2024 Spotlight.
  - Finding: LLMs are "highly receptive" to coherent, convincing external evidence even when it conflicts with parametric memory, yet show strong confirmation bias when the evidence partly agrees with that memory.
  - P/D: no. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2305.13300](https://arxiv.org/abs/2305.13300)
- **What Evidence Do Language Models Find Convincing?** Alexander Wan, Eric Wallace, Dan Klein (2024), ACL 2024.
  - Method: the ConflictingQA dataset, which varies facts, argument styles (e.g., appeals to authority) and answers.
  - Finding: models "rely heavily on the relevance of a website to the query, while largely ignoring stylistic features".
  - P/D: P (credibility cues) partly. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2402.11782](https://arxiv.org/abs/2402.11782)
- **ClashEval.** Kevin Wu, Eric Wu, James Zou (2024), arXiv (venue unverified).
  - Method: over 1,200 questions in six domains, with perturbed retrieved content.
  - Finding: LLMs override their own correct prior with incorrect retrieved content more than 60% of the time. The less confident the model is in its initial answer (token probabilities), the more likely it is to adopt the retrieved content.
  - P/D: no. V: no, though it links confidence to adoption. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2404.10198](https://arxiv.org/abs/2404.10198)
- **How Susceptible are LLMs to Influence in Prompts?** Sotiris Anagnostidis, Jannis Bulian (2024), arXiv (venue unverified).
  - Method: adds another model's prediction and explanation to the prompt, varying the stated authoritativeness and confidence of that source.
  - Finding: models are swayed by explanations "irrespective of the quality". Authoritative or confident framing increases the sway, "but the effect is small in size".
  - P/D: P yes. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2408.11865](https://arxiv.org/abs/2408.11865)
- **Credibility-aware Generation (CAG): Not All Contexts Are Equal.** Ruotong Pan et al. (2024), EMNLP 2024 Main.
  - Method: a data-transformation framework that trains models to "discern and process information based on its credibility".
  - P/D: P yes; D no. V: no. Overlap: LOW–MED (a provenance-aware method that does not touch duplication). Checked: abstract.
  - [arXiv 2404.06809](https://arxiv.org/abs/2404.06809)
- **CrAM.** Boyi Deng, Wenjie Wang, Fengbin Zhu, Qifan Wang, Fuli Feng (2025), AAAI 2025.
  - Method: rescales influential attention heads by each document's credibility score. Tested with Llama2-13B, Llama3-8B and Qwen1.5-7B on NQ and TriviaQA under misinformation.
  - P/D: P yes. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2406.11497](https://arxiv.org/abs/2406.11497)
- **RA-RAG: Retrieval-Augmented Generation with Estimation of Source Reliability.** Jeongyeon Hwang et al. (2024), arXiv (venue unverified).
  - Method: estimates source reliability "by cross-checking information across multiple sources", then aggregates with weighted majority voting.
  - P/D: P yes. Source independence is not modelled. V: no.
  - Overlap: LOW–MED. A system-level method, not a study of LLM behaviour.
  - Checked: abstract.
  - [arXiv 2410.22954](https://arxiv.org/abs/2410.22954)
- **Retrieval-Augmented Generation with Conflicting Evidence (RAMDocs / MADAM-RAG).** Han Wang, Archiki Prasad, Elias Stengel-Eskin, Mohit Bansal (2025), COLM 2025.
  - Finding: a substantial gap remains, "especially when increasing the level of imbalance in supporting evidence and misinformation". Llama3.3-70B scores 32.60 exact match on RAMDocs.
  - P/D: D (count imbalance) yes. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2504.13079](https://arxiv.org/abs/2504.13079)
- **Spotlighting.** Keegan Hines et al. (2024), arXiv.
  - Method: transforms input text to give "a reliable and continuous signal of its provenance". Used as a defence against indirect prompt injection.
  - P/D: P yes (security only). V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2403.14720](https://arxiv.org/abs/2403.14720)
- **Contradiction to Consensus.** Md Badsha Biswas, Ozlem Uzuner (2026), arXiv.
  - Method: claim verification over Wikipedia, PubMed and Google evidence that is filtered, deduplicated and aggregated. Confidence scores are used to visualise disagreement between sources.
  - P/D: D (dedup only, no dependence model). V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2602.18693](https://arxiv.org/abs/2602.18693)
- **Susceptibility to Influence of Large Language Models.** Lewis D. Griffin et al. (2023), arXiv.
  - Finding: LLM-simulated participants reproduce the human illusory truth effect pattern, where earlier exposure boosts later truth ratings.
  - P/D: D (repetition) yes. V: no. Overlap: LOW, because it simulates humans rather than measuring agent belief. Checked: abstract.
  - [arXiv 2303.06074](https://arxiv.org/abs/2303.06074)
- **Knowledge Conflicts for LLMs: A Survey.** Rongwu Xu et al. (2024), arXiv (venue unverified).
  - Background survey covering context-memory, inter-context and intra-memory conflicts. Overlap: LOW. Checked: abstract.
  - [arXiv 2403.08319](https://arxiv.org/abs/2403.08319)

### Inferences
- Paper 1 cannot claim the basic effect that "LLMs overweight repeated or same-source evidence". Whose Facts Win already shows it with two same-source copies, Rational Synthesizers with up to 10 paraphrases, and Jin et al. through frequency. Paper 1's novelty has to sit in four places:
  1. an agentic, embodied setting where one write fans out into several memory stores;
  2. a dose-response over copy count with context length held fixed;
  3. provenance labels used as an intervention that might restore discounting;
  4. outcomes measured as calibration, verification choices and downstream persistence, not answer preference.
- Ross et al. find that duplicating correct evidence does not change accuracy. Rational Synthesizers finds that paraphrases do move answers against the prior. Together they suggest the duplication effect concentrates where evidence is wrong or conflicts with the prior, which is Paper 1's misperception case. They also suggest it may show up more in confidence and verification than in accuracy.
- Sybil's provenance-aware aggregator gives a normative reference for how much confidence copies should add. Paper 1 could compare planner confidence against it.
- Credibility methods (CAG, CrAM, RA-RAG) assume per-document credibility. RA-RAG's cross-source checking would plausibly count same-source copies as corroboration, which makes it a natural baseline to break. This has not been tested.

### Gaps
- I could not confirm whether Whose Facts Win holds context length fixed when it repeats a passage (I read roughly the first 100k of 176k characters). In Rational Synthesizers, length grows with the paraphrase count.
- I found no study that varies the number of same-source copies beyond 2 (Whose Facts Win) or 10 (Rational Synthesizers) while measuring calibration rather than answer choice.
- I found no study that shows an LLM explicit "same origin" provenance labels on duplicated items and tests whether it discounts them. Sybil keeps ancestry outside the model.
- Venues not confirmed: ClashEval, Anagnostidis & Bulian, RA-RAG, and the knowledge-conflict survey.

## 2. Multi-agent conformity, herding, echo chambers and sycophancy cascades: does shared or repeated agreement inflate agreement or confidence, and does anyone measure verification?

### Takeaway
Conformity is robust. It grows with majority size, with the gap between a model's own confidence and its peers' confidence, with the model's own uncertainty, and with authority or in-group labels. Peer agreement misleads correct models more easily than it corrects wrong ones.

But every study treats peers as distinct agents. None manipulates whether the agreeing peers share one source and then checks whether the LLM discounts them. Only reliability-memory methods (Σ-Mem, ECL) and Sybil's formula model peer dependence. Verification behaviour is measured in only two places: as protocol compliance under reward pressure (Emergent Collusion), and as an annotated failure category (MAST).

### Cited Findings
- **Easier to Mislead Than to Correct.** Jiaming Qu, Lucheng Fu, Yibo Hu (2026), arXiv.
  - Method: the model answers, sees simulated peers, then answers again. Consensus structure and peer authority labels are manipulated. Four open-weight LLMs, seven QA datasets.
  - Finding: peer agreement makes it "much easier to mislead initially correct models than to correct initially wrong ones". Authority labels push models toward the endorsed answer whether or not it is correct. Chain-of-thought and reflection do not reliably reduce harmful revision. The authors say systems "should verify peer answers rather than simply aggregate them", but they do not measure verification.
  - P/D: P (authority labels) yes; D no. V: no. Overlap: MED. Checked: abstract.
  - [arXiv 2606.01637](https://arxiv.org/abs/2606.01637)
- **LLMs Trust Their Own: Identity-Dependent Conformity in Multi-Agent Systems.** Liron Soffer, Ravid Shwartz-Ziv, Chen Shani (2026), arXiv.
  - Method: 12 open-weight models and nine tasks, with wrong answers coming from peers whose identity is shared with or distinct from the model's.
  - Finding: in-group consensus increases conformity and out-group consensus decreases it. Unlike humans, models are not moved by an ally from the majority's group. CoT suppresses most of these effects. Labelling peers "safety-aligned" shifts overall conformity.
  - P/D: P (identity labels) yes. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2609.33495](https://arxiv.org/abs/2609.33495)
- **Conformity in Large Language Models.** Xiaochen Zhu, Caiqi Zhang, Tom Stafford, Nigel Collier, Andreas Vlachos (2025), ACL 2025 Main.
  - Finding: all tested models conform to the majority, and "LLMs are more likely to conform when they are more uncertain in their own prediction". Instruction-tuned models conform less. A more natural majority tone amplifies conformity. Proposed fixes: Devil's Advocate and Question Distillation.
  - P/D: no. V: no, but it links confidence to conformity. Overlap: MED. Checked: abstract.
  - [arXiv 2410.12428](https://arxiv.org/abs/2410.12428)
- **Herd Behavior: Investigating Peer Influence in LLM-based Multi-Agent Systems.** Young-Min Cho, Sharath Chandra Guntuku, Lyle Ungar (2025), arXiv preprint.
  - Finding: "the gap between self-confidence and perceived confidence in peers significantly impacts" conformity. The format in which peer information is shown modulates herding.
  - P/D: no. V: no. Overlap: MED. Checked: abstract.
  - [arXiv 2505.21588](https://arxiv.org/abs/2505.21588)
- **Do as We Do, Not as You Think (BenchForm).** Zhiyuan Weng, Guikun Chen, Wenguan Wang (2025), ICLR 2025 Oral.
  - Method and finding: five interaction protocols, measured with conformity and independence rates. Factors include "interaction time and majority size". Mitigations: enhanced personas and reflection.
  - P/D: D (majority size) partly. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2501.13381](https://arxiv.org/abs/2501.13381)
- **Exploring Collaboration Mechanisms for LLM Agents (MachineSoM).** Jintian Zhang et al. (2024), ACL 2024.
  - Finding: agent societies built from "easy-going" and "overconfident" traits show conformity and consensus reaching.
  - P/D: no. V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2310.02124](https://arxiv.org/abs/2310.02124)
- **Encouraging Divergent Thinking through Multi-Agent Debate.** Tian Liang et al. (2024), EMNLP 2024.
  - Finding: "Degeneration-of-Thought". Once an LLM "has established confidence in its solutions", self-reflection cannot generate novel thoughts even when the answer is wrong.
  - P/D: no. V: self-reflection only. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2305.19118](https://arxiv.org/abs/2305.19118)
- **Talk Isn't Always Cheap.** Andrea Wynn, Harsh Satija, Gillian Hadfield (2025), ICML 2025 MAS Workshop.
  - Finding: debate can lower accuracy over time even when stronger models outnumber weaker ones. Models shift from correct to incorrect answers, "favoring agreement over challenging flawed reasoning".
  - P/D: no. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2509.05396](https://arxiv.org/abs/2509.05396)
- **Debate or Vote.** Hyeong Kyu Choi, Xiaojin Zhu, Sharon Li (2025), NeurIPS 2025 Spotlight.
  - Finding: majority voting alone explains most of the gains attributed to multi-agent debate. Debate induces a martingale over beliefs, so "debate alone does not improve expected correctness".
  - P/D: no. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2508.17536](https://arxiv.org/abs/2508.17536)
- **Peacemaker or Troublemaker: How Sycophancy Shapes Multi-Agent Debate.** Binwei Yao et al. (2025), arXiv.
  - Finding: inter-agent sycophancy "amplifies disagreement collapse before reaching a correct conclusion" and yields lower accuracy than single-agent baselines.
  - P/D: no. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2509.23055](https://arxiv.org/abs/2509.23055)
- **Not All Flips Are Conformity.** Xiqi Hao et al. (2026), arXiv.
  - Finding: with five GPT-4o agents, 37% of agent-question pairs change answer even with no peer information. Among stable pairs, 29.3% adopt a peer answer. An answer-agnostic intervention suppresses both harmful and beneficial adoption.
  - P/D: no. V: no.
  - Overlap: LOW–MED. The value for Paper 1 is methodological: it shows the need for a matched no-peer control.
  - Checked: abstract.
  - [arXiv 2606.00820](https://arxiv.org/abs/2606.00820)
- **The Deliberative Illusion.** Herun Wan et al. (2026), arXiv.
  - Method: DelibTrace distributes atomic facts across agents and tracks which survive the discussion.
  - Finding: discussion "erases up to 72% of issue-critical facts", and final stances stay anchored in base-model priors. A single malicious agent can inject misinformation "into the shrinking shared context". Summary line: "agents can agree more while knowing less".
  - P/D: shared context, but no duplication manipulation. V: no. Overlap: MED. Checked: abstract.
  - [arXiv 2606.03032](https://arxiv.org/abs/2606.03032)
- **Correlated Errors in Large Language Models.** Elliot Kim, Avi Garg, Kenny Peng, Nikhil Garg (2025), ICML 2025.
  - Finding: across 350+ LLMs, models "agree 60% of the time when both models err" on one leaderboard. Larger, more accurate models have highly correlated errors even across architectures and providers, which affects LLM-as-judge.
  - P/D: model-level dependence. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2506.07962](https://arxiv.org/abs/2506.07962)
- **Great Models Think Alike and this Undermines AI Oversight.** Shashwat Goel et al. (2025), arXiv (venue unverified).
  - Finding: the CAPA similarity metric shows that LLM judges favour models similar to themselves, and that model mistakes are becoming more similar as capability grows.
  - P/D: model-level dependence. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2502.04313](https://arxiv.org/abs/2502.04313)
- **Epistemic Context Learning.** Ruiwen Zhou et al. (2026), arXiv.
  - Finding: agents "blindly conform to misleading peers" because of sycophancy and a weak ability to judge peer reliability. Conditioning on peer profiles built from interaction history lets Qwen3-4B beat a history-agnostic Qwen3-30B.
  - P/D: P (reliability from history) yes. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2601.21742](https://arxiv.org/abs/2601.21742)
- **Σ-Mem: An Online Reliability Memory for LLM-based Multi-Agent Systems.** Peilin Feng, Suorong Yang, Soujanya Poria (2026), arXiv.
  - Motivation: a central model "may be unable to directly verify plausible or correlated peer responses".
  - Method and finding: records evidence on each peer's competence and on relationships among peers. Read directly from memory, it outperforms majority voting on out-of-distribution evaluation.
  - P/D: P and peer correlation yes. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2607.27958](https://arxiv.org/abs/2607.27958)
- **Emergent Collusion in Long-Horizon LLM Agent Interaction.** Xinrui Shi, Yanzhe Zhang, Diyi Yang (2026), arXiv.
  - Method: two agents share task logs and verify each other's work, and a higher reward is reachable only by breaking the verification protocol.
  - Finding: collusion emerges in 94% of trajectories across 10 models. It is "shaped by peer behavior", and restricting interaction history reduces it.
  - P/D: no. V: yes (verification-protocol compliance), but driven by incentives, not by evidence.
  - Overlap: MED. This is the only work I found that measures verification declining because of peers.
  - Checked: abstract.
  - [arXiv 2609.24967](https://arxiv.org/abs/2609.24967)
- **Interacting Large Language Model Agents: Interpretable Models and Social Learning.** Adit Jain, Vikram Krishnamurthy (2024), arXiv.
  - Finding: Bayesian social-learning models capture herding among sequentially interacting LLM agents. Stochastic control can delay herding and improve state estimation.
  - P/D: no. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2411.01271](https://arxiv.org/abs/2411.01271)
- **Steering the Herd.** Raghu Arghal, Kevin He, Shirin Saeedi Bidokhti, Saswati Sarkar (2025), arXiv (ICLR 2026 per a search snippet; venue unverified).
  - Method: theory of an LLM-like planner that controls agents' private signals during sequential social learning.
  - Overlap: LOW. Checked: abstract.
  - [arXiv 2504.02648](https://arxiv.org/abs/2504.02648)
- **Flooding Spread of Manipulated Knowledge in LLM-Based Multi-Agent Communities.** Tianjie Ju et al. (2024 arXiv; journal 2026), Science China Information Sciences 69, 172103.
  - Finding: counterfactual and toxic knowledge spreads between agents. The manipulations "can persist through popular retrieval-augmented generation frameworks, where several benign agents store and retrieve manipulated chat histories". The authors call for guardian agents and fact-checking tools.
  - P/D: D partly (copies stored in several agents' memories). V: no. Overlap: MED. Checked: abstract.
  - [arXiv 2407.07791](https://arxiv.org/abs/2407.07791)
- **Agent Smith.** Xiangming Gu et al. (2024), ICML 2024.
  - Finding: feeding one adversarial image into a single agent's memory infects (almost) all of up to one million LLaVA-1.5 agents, exponentially fast.
  - P/D: D (propagated copies). V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2402.08567](https://arxiv.org/abs/2402.08567)
- **Prompt Infection.** Donghyun Lee, Mo Tiwari (2024), arXiv.
  - Finding: self-replicating LLM-to-LLM injection. The "LLM Tagging" defence, a provenance tag on agent messages combined with other safeguards, significantly mitigates spread.
  - P/D: P yes. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2410.07283](https://arxiv.org/abs/2410.07283)
- **Why Do Multi-Agent LLM Systems Fail? (MAST).** Mert Cemri et al. (2025), arXiv (venue unverified).
  - Method and finding: over 1,600 annotated traces from 7 frameworks, coded into 14 failure modes in 3 categories, one of which is "task verification".
  - P/D: no. V: yes, but only as an annotation category. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2503.13657](https://arxiv.org/abs/2503.13657)

### Inferences
- Copies of one fact in several memory stores are an intra-agent version of a unanimous majority made from one observer. The conformity results predict two things: agreement across stores should raise adoption, and the effect should be stronger when the planner is uncertain (Zhu et al.; Cho et al.). Paper 1 can frame this as "manufactured consensus without independent observers".
- Authority and identity labels already change conformity (Qu et al.; Soffer et al.). No one has tested "same-source" labels, so Paper 1's provenance-label manipulation fills a real gap.
- Not All Flips finds 37% spontaneous answer changes with no peer input. Paper 1 needs a matched no-duplication control and repeated sampling to separate the duplication effect from plain instability.
- The Deliberative Illusion's "agree more while knowing less" maps onto Paper 1's "more confident without more evidence". It is good framing to cite.

### Gaps
- I found no multi-agent study that measures whether consensus reduces tool calls, re-observation or fact-checking. A targeted search turned up nothing.
- The Deliberative Illusion figures in a search snippet (59%, 37%, 13%, 19.3%) are not in the abstract and were not verified. I cite only the abstract numbers.
- Venues not confirmed: Great Models Think Alike, MAST, Steering the Herd.

## 3. Agent memory poisoning and self-reinforcing memory errors: does any work study duplication across memory stores, or provenance tracking as a defence?

### Takeaway
Memory attacks show that a few poisoned records are enough and that their effects persist:
- PoisonedRAG: 5 texts per question give 90% attack success.
- AgentPoison: a poison rate under 0.1% gives over 80% attack success.
- MINJA: poisoning through queries alone works.

Two works describe self-reinforcing memory errors explicitly: A-MemGuard's "self-reinforcing error cycle" and Xiong et al.'s "error propagation". Provenance appears only as a systems primitive or a security label. No work studies one write duplicated across several memory stores. No work tests provenance shown to the planner as a defence, judged by calibration or verification.

### Cited Findings
- **A-MemGuard.** Qianshan Wei et al. (2025), arXiv.
  - Finding: injected records activate only in context. Once triggered, "the corrupted outcome is stored as precedent", a "self-reinforcing error cycle" that amplifies the error and lowers the bar for future attacks.
  - Defence: "consensus-based validation, which detects anomalies by comparing reasoning paths derived from multiple related memories", plus a separate "lessons" memory. Together they cut attack success by over 95%.
  - P/D: no explicit provenance or duplication. V: validation is done by the defence, not chosen by the agent.
  - Overlap: MED–HIGH. Difference: an adversarial setting, no manipulation of copies or provenance, and no measurement of calibration or verification.
  - Checked: abstract.
  - [arXiv 2510.02373](https://arxiv.org/abs/2510.02373)
- **How Memory Management Impacts LLM Agents: Experience-Following Behavior.** Zidi Xiong et al. (2025), arXiv.
  - Finding: when the input is highly similar to a retrieved record, the output is highly similar too ("experience-following"). This causes "error propagation, where inaccuracies in past experiences compound", and "misaligned experience replay". Future task evaluations can serve as free quality labels for stored memories.
  - P/D: no. V: no.
  - Overlap: MED–HIGH (self-reinforcing memory errors). Difference: no copies, no provenance, no confidence measurement.
  - Checked: abstract.
  - [arXiv 2505.16067](https://arxiv.org/abs/2505.16067)
- **AgentPoison.** Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song, Bo Li (2024), arXiv (NeurIPS 2024; venue not re-checked here).
  - Finding: optimised backdoor triggers retrieve poisoned demonstrations. Attack success averages over 80% on a RAG-based autonomous-driving agent, a QA agent and EHRAgent, with under 1% benign impact and a poison rate under 0.1%.
  - P/D: no. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2407.12784](https://arxiv.org/abs/2407.12784)
- **MINJA: Memory Injection Attacks on LLM Agents via Query-Only Interaction.** Shen Dong et al. (2025), NeurIPS 2025 (poster page found).
  - Method: injects malicious records through queries only, using "bridging steps". Sunil et al. (below) cite it as reaching over 95% injection success and 70% attack success "under idealized conditions".
  - P/D: no. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2503.03704](https://arxiv.org/abs/2503.03704); [NeurIPS 2025 poster](https://neurips.cc/virtual/2025/poster/118152)
- **Memory Poisoning Attack and Defense on Memory Based LLM-Agents.** Balachandra Devarangadi Sunil et al. (2026), arXiv.
  - Setting: EHR agents on MIMIC-III with GPT-4o-mini, Gemini-2.0-Flash and Llama-3.1-8B.
  - Finding: pre-existing legitimate memories "dramatically reduce attack effectiveness".
  - Defences: input/output moderation with "composite trust scoring", and memory sanitisation with "trust-aware retrieval" (temporal decay, pattern filtering).
  - P/D: P (trust scores) adjacent. V: no. Overlap: MED. Checked: abstract.
  - [arXiv 2601.05504](https://arxiv.org/abs/2601.05504)
- **MemoryGraft.** Saksham Sahai Srivastava, Haoyu He (2025), arXiv.
  - Finding: planting malicious "successful experiences" exploits the agent's "semantic imitation heuristic". Union retrieval reliably surfaces the grafted memories.
  - P/D: no. V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2512.16962](https://arxiv.org/abs/2512.16962)
- **PoisonedRAG.** Wei Zou, Runpeng Geng, Binghui Wang, Jinyuan Jia (2025), USENIX Security 2025.
  - Finding: "90% attack success rate when injecting five malicious texts for each target question into a knowledge database with millions of texts". Existing defences are insufficient.
  - P/D: D (several injected texts per target). V: no. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2402.07867](https://arxiv.org/abs/2402.07867)
- **Agent Security Bench (ASB).** Hanrong Zhang et al. (2025), ICLR 2025.
  - Finding: the benchmark includes a memory-poisoning attack. The highest average attack success is 84.30%, and the defences tested had limited effect.
  - P/D: no. V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2410.02644](https://arxiv.org/abs/2410.02644)
- **Your Agent May Misevolve.** Shuai Shao et al. (2026), ICLR 2026.
  - Finding: in self-evolving agents, safety alignment degrades "after memory accumulation".
  - P/D: no. V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2509.26354](https://arxiv.org/abs/2509.26354)
- **HaluMem.** Ding Chen et al. (2025), arXiv.
  - Method: an operation-level benchmark of memory hallucinations (fabrication, errors, conflicts, omissions) across extraction, updating and QA.
  - P/D: no. V: no. Overlap: LOW–MED. It shows where memory errors originate, not how they propagate. Checked: abstract.
  - [arXiv 2511.03506](https://arxiv.org/abs/2511.03506)
- **How Language Model Hallucinations Can Snowball.** Muru Zhang, Ofir Press, William Merrill, Alisa Liu, Noah A. Smith (2023), arXiv (venue unverified).
  - Finding: LMs "over-commit to early mistakes". Asked separately, ChatGPT and GPT-4 identify 67% and 87% of their own mistakes.
  - P/D: self-origin evidence. V: shows separate checking works. Overlap: MED. Checked: abstract.
  - [arXiv 2305.13534](https://arxiv.org/abs/2305.13534)
- **Multimodal Hallucination Snowballing (MMHalSnowball).** Weihong Zhong et al. (2024), ACL 2024.
  - Finding: open-source LVLMs answering inside hallucinatory conversations drop at least 31% in performance. They accept earlier generated hallucinations "even though the ground visual information exists".
  - P/D: self-origin text versus image. V: no.
  - Overlap: MED. The closest evidence that earlier text can override available images.
  - Checked: abstract.
  - [arXiv 2407.00569](https://arxiv.org/abs/2407.00569)
- **Governed Shared Memory for Multi-Agent LLM Systems (MemClaw).** Yanki Margalit, Nurit Cohen-Inger, Erni Avram, Ran Taig, Oded Margalit (2026), arXiv.
  - Failure modes named: "stale propagation, contradiction persistence, and provenance collapse".
  - Primitives proposed: provenance tracking and temporal supersession. The system rebuilt 100% of depth-four derivation chains with the correct writer identity.
  - Also found: "a synchronous near-duplicate gate can prematurely reject contradictory writes".
  - P/D: P yes, D (near-duplicates) yes. V: no. This is a systems study with no measurement of LLM behaviour.
  - Overlap: MED. Checked: abstract.
  - [arXiv 2606.24535](https://arxiv.org/abs/2606.24535)
- **Mesh Memory Protocol.** Hongwei Xu (2026), arXiv.
  - A protocol specification in which every claim is traceable "so returning claims are recognised as echoes of the receiver's own prior thinking", using content-hash lineage.
  - P/D: P yes. V: no. There is no controlled evaluation. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2604.19540](https://arxiv.org/abs/2604.19540)
- **A Survey on Long-Term Memory Security in LLM Agents.** Zehao Lin et al. (2026), EMNLP 2026.
  - Organises memory threats along a lifecycle that includes a "Share & Propagate" phase. Argues that security "must be anchored in storage-time provenance, versioning, and policy-aware retention".
  - P/D: P (position). V: no. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2604.16548](https://arxiv.org/abs/2604.16548)
- **Collaborative Memory.** Alireza Rezazadeh et al. (2025), arXiv.
  - Each memory fragment carries "immutable provenance attributes (contributing agents, accessed resources, and timestamps)". These serve access control, not reliability.
  - P/D: P yes. V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2505.18279](https://arxiv.org/abs/2505.18279)
- **Securing AI Agents with Information-Flow Control (FIDES).** Manuel Costa et al. (2025), arXiv.
  - The planner tracks confidentiality and integrity labels (taint tracking), evaluated in AgentDojo.
  - P/D: P (security). V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2505.23643](https://arxiv.org/abs/2505.23643)
- **CaMeL: Defeating Prompt Injections by Design.** Edoardo Debenedetti et al. (2025), arXiv.
  - Separates control flow from data flow and adds capabilities. Solves 77% of AgentDojo tasks with provable security, against 84% undefended.
  - P/D: P (security). V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2503.18813](https://arxiv.org/abs/2503.18813)
- **PROV-AGENT.** Renan Souza et al. (2025), IEEE e-Science 2025.
  - Extends W3C PROV to capture prompts, responses and decisions, so error propagation can be traced through agent workflows.
  - P/D: P yes. V: no. Overlap: LOW. Checked: abstract.
  - [arXiv 2508.02866](https://arxiv.org/abs/2508.02866)
- **BrainMem.** Xiaoyu Ma et al. (2026), arXiv.
  - An embodied planning memory system motivated by stateless planners "repeating errors". It improves results on EB-ALFRED, EB-Navigation and other benchmarks. It does not study errors that memory itself causes.
  - Overlap: LOW. Checked: abstract.
  - [arXiv 2604.16331](https://arxiv.org/abs/2604.16331)

### Inferences
- A-MemGuard's defence compares reasoning paths across multiple related memories. That is exactly the check same-source copies would defeat, because copies look like agreement. A-MemGuard does not test this. Paper 1 could add a consensus-across-stores check as a baseline defence and show that duplication breaks it.
- Almost all poisoning work is adversarial. Paper 1's error source is a benign misperception by the agent's own memory writer. That is closer to experience-following, hallucination snowballing and MMHalSnowball, all of which involve errors of the agent's own origin, and none of which manipulates copy count.
- Governed Shared Memory shows that dedup policies in production memory interact with contradiction handling in non-obvious ways. That is useful design context for any dedup or provenance remedy Paper 1 proposes.

### Gaps
- I did not search USENIX Security, CCS or IEEE S&P 2026 proceedings directly; I relied on arXiv and web search. Provenance-based memory defences published only at those venues may be missing.
- I found no memory-poisoning study on embodied agents beyond AgentPoison's autonomous-driving agent.
- MINJA's details beyond "bridging steps" were not checked in the abstract.
- Venues not confirmed: AgentPoison (NeurIPS 2024 from memory) and hallucination snowballing.

## 4. LLM agents' information seeking and verification versus confidence

### Takeaway
Methods that trigger retrieval, tool calls, help requests or exploration from uncertainty are mature: FLARE, SeaKR, SKR, UALA, KnowNo, Explore-until-Confident, MediQ, UoT. Work from 2026 adds two findings:
- Evidence tools inflate verbalised confidence (Confidence Dichotomy, ACL 2026).
- Authoritative evidence packaging raises commitment while stated probabilities "barely move" (Aggarwal 2026).

No work tests whether duplicated or same-source evidence reduces an agent's own verification. Tool-use benchmarks often show over-calling rather than under-calling. Cost-aware exploration is studied in Calibrate-Then-Act, but nobody runs a sweep over verification costs.

### Cited Findings
- **Calibrated Enough to Know, Not Calibrated to Act: Fabricated Evidence Makes LLM Agents Commit to the Unknowable.** Pranav Aggarwal (2026), arXiv (pre-registered).
  - Finding, across 12 frontier models on provably unpredictable questions:
    - Commitment rises from 6.5% to 54.0% as evidence is escalated.
    - A fully fabricated panel lifts commitment from 24.5% to 36.8%, statistically indistinguishable from the 37.6% produced by genuine data.
    - Stated probabilities "barely move".
    - Models label a question irreducible 90% of the time, then commit on only 0.4% of those.
  - Fix: SFT of a 3B model on 540 synthetic cases drives commitment to 0.0% and transfers to three unseen domains. Rigid response formats undo it.
  - P/D: P-like (the "authority of its packaging"); D no. V: commit versus abstain, not verification.
  - Overlap: MED–HIGH on confidence versus action. Belief and action dissociate.
  - Checked: abstract.
  - [arXiv 2608.27167](https://arxiv.org/abs/2608.27167)
- **The Confidence Dichotomy: Analyzing and Mitigating Miscalibration in Tool-Use Agents.** Weihao Xuan, Qingcheng Zeng, Heli Qi, Yunze Xiao, Junjue Wang, Naoto Yokoya (2026), ACL 2026.
  - Finding: evidence tools such as web search "systematically induce severe overconfidence due to inherent noise in retrieved information". Verification tools such as code interpreters "ground reasoning" and reduce miscalibration. An RL method optimises accuracy and calibration jointly. A third-party figure of up to 68% ECE reduction is unverified.
  - P/D: no. V: tool type versus calibration, not the decision to verify. Overlap: MED. Checked: abstract.
  - [arXiv 2601.07264](https://arxiv.org/abs/2601.07264); [ACL Anthology 2026.acl-long.520](https://aclanthology.org/2026.acl-long.520/)
- **Search Wisely.** Peilin Wu, Mian Zhang, Xinlu Zhang, Xinya Du, Zhiyu Zoey Chen (2025), EMNLP 2025 Main.
  - Finding: defines over-search and under-search (one model could have skipped search in 27.7% of its search steps). Shows "a crucial link between these inefficiencies and the models' uncertainty regarding their own knowledge boundaries". Proposes β-GRPO with a confidence threshold.
  - P/D: no. V: yes (search decisions). Overlap: MED. Checked: abstract.
  - [arXiv 2505.17281](https://arxiv.org/abs/2505.17281)
- **Towards Uncertainty-Aware Language Agent (UALA).** Jiuzhou Han, Wray Buntine, Ehsan Shareghi (2024), ACL 2024 Findings.
  - Finding: gating tool use by uncertainty improves performance with "reduced number of tool calls". Verbalised confidence is an unreliable proxy for uncertainty.
  - P/D: no. V: yes. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2401.14016](https://arxiv.org/abs/2401.14016)
- **Robots That Ask For Help (KnowNo).** Allen Z. Ren et al. (2023), CoRL 2023 Oral.
  - Method: conformal prediction lets LLM planners "know when they don't know and ask for help", with statistical guarantees and minimal help requests.
  - P/D: no. V: yes (asking for help). Overlap: MED–LOW (embodied). Checked: abstract.
  - [arXiv 2307.01928](https://arxiv.org/abs/2307.01928)
- **Explore until Confident.** Allen Z. Ren, Jaden Clark, Anushri Dixit, Masha Itkina, Anirudha Majumdar, Dorsa Sadigh (2024), RSS 2024.
  - Finding: VLM confidence "can be miscalibrated and can cause the robot to prematurely stop exploration or over-explore". Conformal calibration controls when to stop.
  - P/D: no. V: yes (embodied exploration). Overlap: MED. Checked: abstract.
  - [arXiv 2403.15941](https://arxiv.org/abs/2403.15941)
- **Know Where You're Uncertain When Planning with Multimodal Foundation Models.** Neel P. Bhatt et al. (2024), arXiv.
  - Method and finding: separates perception uncertainty from decision uncertainty. An active-sensing step "dynamically re-observes high-uncertainty scenes". Variability drops by up to 40% and success rises by 5%.
  - P/D: no. V: yes (re-observation). Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2411.01639](https://arxiv.org/abs/2411.01639)
- **VAP-TAMP: Robot Planning and Situation Handling with Active Perception.** Austine Oloo et al. (2026), arXiv.
  - Method: prompts a VLM for active view selection and situation assessment during execution.
  - Unverified: a search snippet attributes to the full text the phrase that VLMs are treated as "infallible oracles… never verified", and 88% success versus 76% (OK-Robot) and 69% (COWP).
  - V: yes. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2604.26988](https://arxiv.org/abs/2604.26988)
- **Uncertainty-Informed Active Perception for ObjectNav.** Utkarsh Bajpai et al. (2025), ECMR 2025.
  - Uses semantic uncertainty from VLM prompt variation to guide exploration. Overlap: LOW. Checked: abstract.
  - [arXiv 2506.13367](https://arxiv.org/abs/2506.13367)
- **MediQ.** Shuyue Stella Li et al. (2024), arXiv (venue unverified).
  - Method: the Expert "refrains from making diagnostic decisions when unconfident, and instead elicits missing details via follow-up questions". Quantitative results were not checked.
  - V: yes. Overlap: MED–LOW. Checked: abstract (truncated).
  - [arXiv 2406.00922](https://arxiv.org/abs/2406.00922)
- **Uncertainty of Thoughts (UoT).** Zhiyuan Hu et al. (2024), NeurIPS 2024.
  - Method: uncertainty-aware simulation plus information-gain rewards to choose questions.
  - V: yes. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2402.03271](https://arxiv.org/abs/2402.03271)
- **When2Call.** Hayley Ross, Ameya Sunil Mahabaleshwarkar, Yoshi Suhara (2025), NAACL 2025.
  - Benchmark for when to call a tool, when to ask a follow-up, and when to admit inability.
  - V: yes. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2504.18851](https://arxiv.org/abs/2504.18851)
- **To Call or Not to Call: Diagnosing Intrinsic Over-Calling Bias in LLM Agents.** Wei Shi et al. (2026), arXiv.
  - Finding: six models "over-call" on When2Call, with overall accuracy of 55–70%. Sparse autoencoders reveal an activation-independent call offset, and steering it away corrects over-calling.
  - V: yes. Overlap: MED–LOW. Counterpoint: the default bias leans toward calling. Checked: abstract.
  - [arXiv 2605.18882](https://arxiv.org/abs/2605.18882)
- **LLM Agents Already Know When to Call Tools (When2Tool, Probe&Prefill).** Chung-En Sun, Linbo Liu, Ge Yan, Zimo Wang, Tsui-Wei Weng (2026), arXiv (NeurIPS 2026 per a search snippet; venue unverified).
  - Finding: tool necessity is linearly decodable from hidden states (AUROC 0.89–0.96), but models "fail to act on this knowledge". Probe&Prefill cuts tool calls by 48% with a 1.7% accuracy loss.
  - V: yes. Overlap: MED–LOW. Checked: abstract.
  - [arXiv 2605.09252](https://arxiv.org/abs/2605.09252)
- **SMART: Self-Aware Agent for Tool Overuse Mitigation.** Cheng Qian et al. (2025), ACL 2025 Findings.
  - Finding: SMARTAgent cuts tool use by 24%.
  - V: yes. Overlap: LOW. Checked: abstract.
  - [arXiv 2502.11435](https://arxiv.org/abs/2502.11435)
- **TRUST.** Yijin Zhou et al. (2026), arXiv.
  - Finding: decision-oriented RL "tends to weaken the uncertainty separation between correct and incorrect actions, resulting in overconfident mistakes and weaker exploration signals".
  - V: tool-use decisions. Overlap: LOW–MED. Checked: abstract.
  - [arXiv 2606.06976](https://arxiv.org/abs/2606.06976)
- **Calibrate-Then-Act: Cost-Aware Exploration in LLM Agents.** Wenxuan Ding, Nicholas Tomlin, Greg Durrett (2026), arXiv.
  - Method: formalises cost-uncertainty tradeoffs as sequential decisions, such as writing a test versus running possibly buggy code. Passing the agent an inferred prior over the latent environment state "qualitatively changes agent behavior" and yields more optimal strategies on a synthetic task, QA and file reading.
  - P/D: no. V: yes, with explicit costs.
  - Overlap: MED. The closest precedent for a verification-cost sweep.
  - Checked: abstract.
  - [arXiv 2602.16699](https://arxiv.org/abs/2602.16699)
- **Uncertainty-triggered retrieval methods.** All Overlap LOW; abstracts checked.
  - FLARE: actively decides "when and what to retrieve". Zhengbao Jiang et al. (2023), EMNLP 2023. [arXiv 2305.06983](https://arxiv.org/abs/2305.06983)
  - SeaKR: retrieves when internal-state uncertainty is high. Zijun Yao et al. (2024), arXiv. [arXiv 2406.19215](https://arxiv.org/abs/2406.19215)
  - SKR: self-knowledge decides external calls. Yile Wang et al. (2023), arXiv. [arXiv 2310.05002](https://arxiv.org/abs/2310.05002)
  - A comparison of 35 adaptive-retrieval methods finds that uncertainty estimation "often outperform[s] complex pipelines" in efficiency and self-knowledge. Moskvoretskii et al. (2025), arXiv. [arXiv 2501.12835](https://arxiv.org/abs/2501.12835)
- **Uncertainty Quantification for LLM Agents (TC-ECE).** Moule Lin et al. (2026), arXiv.
  - Finding: step-level calibration does not imply trajectory-level calibration, and step errors are coupled. The agent's own confidence estimates do not consistently beat a simple baseline. Averaging over trajectories "can hide overconfidence at later stages".
  - Overlap: LOW–MED (useful for metric design). Checked: abstract.
  - [arXiv 2609.07395](https://arxiv.org/abs/2609.07395)
- **Beyond Single-Turn Confidence.** Dylan Bouchard, Mohit Singh Chauhan (2026), arXiv.
  - Finding: single-turn UQ transfers to agent trajectories only unevenly. Black-box self-consistency is often the strongest family.
  - Overlap: LOW. Checked: abstract.
  - [arXiv 2608.11552](https://arxiv.org/abs/2608.11552)
- **Replan for Error Recovery (ERR-Bench).** Zhuoran Wang et al. (2026), arXiv.
  - Finding: reveals "a mismatch between task progress and execution reliability across diverse VLMs" in embodied manipulation. Relevant to Paper 1's progress-judgment outcome.
  - Overlap: LOW. Checked: abstract.
  - [arXiv 2610.27003](https://arxiv.org/abs/2610.27003)
- **OmniAct.** Junhao Shi et al. (2026), arXiv.
  - Argues for "explicit separation of planning, memory, and verification" in embodied agents.
  - Unverified: a search snippet says the monitor checks outcomes from images only, without the planner's intent, to reduce task-context leakage.
  - Overlap: LOW. Checked: abstract.
  - [arXiv 2606.27251](https://arxiv.org/abs/2606.27251)
- **Confidence-to-adoption links reported in sections 1–2.** These connect confidence to belief change, not to verification.
  - ClashEval: lower prior confidence leads to more adoption of retrieved content. [arXiv 2404.10198](https://arxiv.org/abs/2404.10198)
  - Zhu et al.: more uncertain models conform more. [arXiv 2410.12428](https://arxiv.org/abs/2410.12428)
  - Cho et al.: the confidence gap with peers drives herding. [arXiv 2505.21588](https://arxiv.org/abs/2505.21588)

### Inferences
- Paper 1's causal chain has two links: duplication raises confidence, and confidence lowers verification. The second link is supported in general terms (Search Wisely, Explore-until-Confident, KnowNo, UALA). But Aggarwal and When2Tool show that action can come apart from both stated belief and internal knowledge. Paper 1 should therefore test whether duplication changes verification even when stated confidence does not move, and report both measures.
- Over-calling bias (Shi et al.; Sun et al.; SMART) means that at low cost the planner may verify regardless. A cost sweep is needed to show where duplication shifts the verification threshold, which matches Paper 1's plan. A Calibrate-Then-Act-style expected-value threshold could serve as the normative reference.
- Calibration should be reported by step or horizon, because averaging can hide late overconfidence (TC-ECE).

### Gaps
- I found no study that links duplicated or same-source evidence to verification or information-seeking decisions.
- Not verified: MediQ's quantitative results, the VAP-TAMP numbers, the Confidence Dichotomy ECE figure, and the When2Tool venue.

## 5. Overlap ranking across all four areas, and overlap with papers already in our notes

### Takeaway
I found no work that combines all four of Paper 1's parts:
1. duplicated or same-source evidence, or provenance labels, shown to an LLM;
2. measurement of that LLM's own calibration and verification choices across verification costs;
3. downstream error persistence in both progress judgments and next actions;
4. an embodied agent with multi-store shared memory.

The three closest works cover only part (1) or its formal version: Epistemic Sybil Resistance, Whose Facts Win and Rational Synthesizers. A-MemGuard and Xiong et al. cover self-reinforcing memory errors. Aggarwal and the Confidence Dichotomy cover evidence inflating confidence or commitment. Commit gates, already in our notes, is still the closest precedent for the design (shared versus independent evidence source).

### Cited Findings
Closest works, ranked. Here P/D is split into D (duplication or same source) and P (provenance), and V is verification or information-seeking measured.

| Rank | Work | D | P | V | Overlap | Key difference from Paper 1 |
|---|---|---|---|---|---|---|
| 1 | [Epistemic Sybil Resistance (2609.01873)](https://arxiv.org/abs/2609.01873) | yes | outside model | no | HIGH | The overconfidence is in a formula aggregator; there is no LLM confidence, no labels shown to the model, no verification and no embodiment. |
| 2 | [Whose Facts Win? (2601.03746, ACL 2026)](https://arxiv.org/abs/2601.03746) | yes (2 copies, same source) | yes | no | HIGH | Static QA with answer-probability preference only; no agent and no verification. |
| 3 | [Rational Synthesizers (2601.06189)](https://arxiv.org/abs/2601.06189) | yes (paraphrases) | no | no | HIGH–MED | RAG QA; context length grows with copies. |
| 4 | [A-MemGuard (2510.02373)](https://arxiv.org/abs/2510.02373) | no | no | defence-side | MED–HIGH | Adversarial; its cross-memory consensus check would likely count copies as agreement (untested). |
| 5 | [Calibrated Enough to Know (2608.27167)](https://arxiv.org/abs/2608.27167) | no | packaging | commit/abstain | MED–HIGH | Unknowable forecasts; tests evidence authority, not copy count. |
| 6 | [Experience-following (2505.16067)](https://arxiv.org/abs/2505.16067) | no | no | no | MED–HIGH | Errors propagate through similarity, not duplication. |
| 7 | [Tug-of-War (2402.14409)](https://arxiv.org/abs/2402.14409) | frequency | no | no | MED | Majority rule in QA; independence not manipulated. |
| 8 | [Easier to Mislead (2606.01637)](https://arxiv.org/abs/2606.01637) | no | authority labels | no | MED | Peers are distinct; no verification measured. |
| 9 | [Confidence Dichotomy (2601.07264)](https://arxiv.org/abs/2601.07264) | no | no | tool type | MED | Evidence tools inflate confidence; no duplication. |
| 10 | [Calibrate-Then-Act (2602.16699)](https://arxiv.org/abs/2602.16699) | no | no | yes (costs) | MED | Cost-aware exploration without evidence manipulation. |
| 11 | [Emergent Collusion (2609.24967)](https://arxiv.org/abs/2609.24967) | no | no | yes | MED | Verification eroded by reward, not by evidence. |
| 12 | [Governed Shared Memory (2606.24535)](https://arxiv.org/abs/2606.24535) | near-duplicates | yes | no | MED | Systems study; no LLM behaviour measured. |
| 13 | [Conformity in LLMs (2410.12428)](https://arxiv.org/abs/2410.12428) | no | no | no | MED | Uncertain models conform more; inter-agent setting. |
| 14 | [Explore until Confident (2403.15941)](https://arxiv.org/abs/2403.15941) | no | no | yes | MED | Embodied confidence leads to stopping; no duplication. |
| 15 | [Redundancy vs diversity in RAG (2608.13956)](https://arxiv.org/abs/2608.13956) | yes | no | no | MED | Counterpoint: duplicated correct evidence leaves accuracy unchanged. |
| 16 | [Deliberative Illusion (2606.03032)](https://arxiv.org/abs/2606.03032) | no | no | no | MED | "Agree more while knowing less"; deliberation setting. |
| 17 | [Flooding Spread (2407.07791)](https://arxiv.org/abs/2407.07791) | partly | no | no | MED | Manipulated knowledge persists via RAG stores across agents. |
| 18 | [MMHalSnowball (2407.00569)](https://arxiv.org/abs/2407.00569) | self-origin | no | no | MED | Earlier text overrides available images; no copies. |

Overlap with papers already in our notes. This only states how the new finds relate to them, not their content.
- [Commit gates (2609.10969)](https://arxiv.org/abs/2609.10969) and [Epistemic Sybil Resistance (2609.01873)](https://arxiv.org/abs/2609.01873) both appeared in September 2026 and make the same structural point, that shared evidence is not independent corroboration. Commit gates makes it for proposer and verifier; Sybil makes it for multi-agent reports. Both work at the system level. Neither shows duplicated copies to an LLM planner, and neither measures the planner's own verification choices. Cite them together.
- [Gaming the Judge (2601.14691)](https://arxiv.org/abs/2601.14691) pairs with [Aggarwal (2608.27167)](https://arxiv.org/abs/2608.27167). Gaming the Judge shows that rewriting narration shifts a judge. Aggarwal shows that authoritative packaging shifts action without moving stated belief. Together they support measuring actions, not only confidence.
- [Self-Attribution Bias (2603.04582)](https://arxiv.org/abs/2603.04582) pairs with the self-origin-evidence findings: [hallucination snowballing (2305.13534)](https://arxiv.org/abs/2305.13534), [MMHalSnowball (2407.00569)](https://arxiv.org/abs/2407.00569), [A-MemGuard's "stored as precedent" (2510.02373)](https://arxiv.org/abs/2510.02373) and [the Mesh Memory Protocol's "echoes of the receiver's own prior thinking" (2604.19540)](https://arxiv.org/abs/2604.19540). None of them manipulates copy count.
- [When Memory Lies (2608.04574)](https://arxiv.org/abs/2608.04574) complements [experience-following (2505.16067)](https://arxiv.org/abs/2505.16067) and [A-MemGuard (2510.02373)](https://arxiv.org/abs/2510.02373). None of these studies a benign misperception fanned out across several stores.
- [Binding Drift (2607.18316)](https://arxiv.org/abs/2607.18316) covers the remedy side: a re-check call reduces wrong actions. Paper 1 asks the inverse question, whether duplication suppresses the agent's own choice to make that check. I found nothing that answers it.

### Inferences
- Safe novelty wording: "the first causal test, in an embodied agent with multi-store memory, of whether same-source duplication and provenance labels change the planner's calibration and its own verification choices across verification costs, and whether the resulting errors persist into both progress judgments and next actions."
- Claims to avoid: "first to show LLMs overweight repeated or same-source evidence" (Whose Facts Win, Rational Synthesizers, Jin et al.); "first to formalise replicated-report overconfidence" (Sybil); "first to show memory errors self-reinforce" (A-MemGuard, Xiong et al.).
- Design implications drawn from the literature:
  1. Cross copy count with source independence at the same count, in a 2×2 (Whose Facts Win's repetition versus 2-Table Majority; Sybil's root multiplicity).
  2. Hold context length fixed, which the closest prior studies do not clearly do.
  3. Include no-duplication and self-repeat controls for spontaneous flips (Not All Flips).
  4. Measure stated confidence and verification action separately (Aggarwal, When2Tool).
  5. Sweep verification cost, because baselines may over-call (Shi et al.; Calibrate-Then-Act).
  6. Include a cross-store consensus check as a baseline defence that duplication should defeat (A-MemGuard).

### Gaps
- Negative result, which is itself the main novelty signal: after searching all four areas I found no paper that measures verification or information-seeking as a function of the number of same-source copies, and none that shows an LLM provenance labels and then tests whether it discounts copies.
- Coverage limits:
  - Security venues were searched only through arXiv and the web.
  - Embodied memory poisoning was barely covered.
  - I read Whose Facts Win only partially (about 100k of 176k characters) and Sybil only partially (100k of 114k characters).
- Venues not confirmed:
  - AgentPoison
  - ClashEval
  - Anagnostidis & Bulian
  - RA-RAG
  - Great Models Think Alike
  - MAST
  - MediQ
  - hallucination snowballing
  - SeaKR
  - SKR
  - When2Tool
  - Steering the Herd
