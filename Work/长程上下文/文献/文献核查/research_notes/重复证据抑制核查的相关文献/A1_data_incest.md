# A1 — Double counting of evidence ("data incest", "rumor propagation", "common information") in sensor fusion, distributed estimation and multi-robot systems, and whether it is linked to reduced sensing or verification

Scope note for the report writer. This file covers the estimation, fusion and robotics literature, plus Krishnamurthy's data-incest work on social networks. Economics and psychology of correlation neglect appear only as short pointers, because A2_social_learning covers them. Two 2026 LLM preprints are included in detail because they borrow the data-incest framing and are the closest collisions with Paper 1. B1 and B2 may also cover them.

Verification legend, used on every item:
- [V-abs]: checked against the abstract (OpenAlex, Crossref or arXiv).
- [V-full]: checked against full-text passages.
- [V-meta]: only title, authors, year and venue were checked. Any description of content is marked "content unverified".
- [snippet]: seen only in a search-engine snippet.

Citation counts are OpenAlex counts as of 2026-10-21. They run below Google Scholar counts.

Notation for Paper 1 in the overlap tags:
- "P1" = Paper 1's claim (duplicated or same-source memory evidence in an embodied LLM agent leads to overconfidence, less verification, and errors that persist).
- "Overlap: H/M/L" = high, medium or low overlap with P1.

## KQ1 — What are the foundational and most-cited works on data incest, double counting and rumor propagation, and what do they prove about overconfidence?

### Takeaway
Distributed estimation has known since the 1980s–90s that information reused as if it were independent makes the fused covariance or posterior too small (overconfident). It can also make estimates inconsistent or cause filter divergence. Classic results either quantify the error (Bar-Shalom & Campo 1986) or prove that particular remedies stay consistent (covariance intersection, 1997). Krishnamurthy and colleagues carried the term "data incest" over to Bayesian social learning (2013–2016). There it causes overconfidence and bias, and incest can be removed only under necessary-and-sufficient conditions on the information-flow graph. None of these classic works measures what the overconfidence does to later sensing or verification decisions; they stop at estimation quality.

### Cited Findings

#### Roots: information graph, decentralized architectures, common process noise
- **Chong, Mori, Tse, Wishner (1982), "Distributed Estimation in Distributed Sensor Networks", American Control Conference.** An early root of the distributed-estimation and information-graph line. [V-meta; content unverified] — [doi:10.23919/acc.1982.4787968](https://doi.org/10.23919/acc.1982.4787968). Overlap: L (numeric state estimation; no decisions).
- **Chong, Chang, Mori (1986), "Distributed Tracking in Distributed Sensor Networks", ACC.** Same research line. [V-meta; content unverified] — [doi:10.23919/acc.1986.4789229](https://doi.org/10.23919/acc.1986.4789229). Overlap: L.
- **Bar-Shalom & Campo (1986), "The Effect of the Common Process Noise on the Two-Sensor Fused-Track Covariance", IEEE Trans. Aerospace and Electronic Systems (511 cites).**
  - Finding: for an α-β tracker, the fused uncertainty area is about 70% of the single-sensor area once common process noise is modeled. It is 50% if the dependence is ignored.
  - So treating dependent tracks as independent overstates the precision gained. This is the canonical quantified overconfidence result for track-to-track fusion. [V-abs] — [doi:10.1109/TAES.1986.310815](https://doi.org/10.1109/TAES.1986.310815).
  - Overlap: L (Gaussian tracks; no decisions). It is the classic citation for the claim that "ignoring common information overstates certainty".
- **Durrant-Whyte, Rao, Hu (1990), "Toward a fully decentralized architecture for multi-sensor data fusion", ICRA (129 cites).** Proposes a fully decentralized EKF-style network: each node processes locally, any two nodes may communicate, and there is no central processor. This is the architecture whose loops later raised the double-counting problem. [V-abs] — [doi:10.1109/ROBOT.1990.126185](https://doi.org/10.1109/ROBOT.1990.126185). OpenAlex lists the year as 2002; the DOI shows ICRA 1990. Overlap: L.
- **Grime, Durrant-Whyte, Ho (1992), "Communication in Decentralized Data-Fusion Systems", ACC.** [V-meta; content unverified] — [doi:10.23919/acc.1992.4792761](https://doi.org/10.23919/acc.1992.4792761). Overlap: L.
- **Grime & Durrant-Whyte (1994), "Data fusion in decentralized sensor networks", Control Engineering Practice.** [V-meta via Crossref.] — [doi:10.1016/0967-0661(94)90349-2](https://doi.org/10.1016/0967-0661(94)90349-2).
  - It is usually credited as the channel-filter paper: each link keeps a filter of the information already shared, so that it can be subtracted. The usual stated requirement is a tree-connected (no-loop) network.
  - The abstract was not retrieved, so this content is unverified for this specific paper. Channel-filter consistency is confirmed indirectly by Chang et al. 2010 below. Overlap: L.
- **Liggins, Chong, Kadar, Alford, Vannicola, Thomopoulos (1997), "Distributed Fusion Architectures and Algorithms for Target Tracking", Proceedings of the IEEE (353 cites).**
  - Introduces the information graph as a model of information flow in distributed fusion.
  - Derives distributed estimation algorithms based on that graph for arbitrary fusion architectures, plus distributed versions of JPDA and MHT. [V-abs] — [doi:10.1109/JPROC.1997.554211](https://doi.org/10.1109/JPROC.1997.554211).
  - Overlap: L. It is the conceptual ancestor of provenance tracking: you can remove double counting only if you know the graph of where each piece of information came from.
- **Chong, Mori, Barker, Chang (2000), "Architectures and algorithms for track association and fusion", IEEE AES Magazine (241 cites).** Compares sensor-to-sensor and sensor-to-system track fusion. Discusses statistical correlation between track errors, and compares (by theory and Monte Carlo) fusion rules that account for it. [V-abs] — [doi:10.1109/62.821657](https://doi.org/10.1109/62.821657). Overlap: L.

#### Covariance intersection (CI) and its generalizations: proofs of consistency under unknown correlation
- **Julier & Uhlmann (1997), "A non-divergent estimation algorithm in the presence of unknown correlations", ACC (1,026 cites; the most-cited work in this area).**
  - Introduces CI and proves it yields consistent (not overconfident) estimates whatever the actual cross-correlation is.
  - Illustrates it on a decentralized estimation case where a Kalman filter cannot be used consistently. [V-abs] — [doi:10.1109/ACC.1997.609105](https://doi.org/10.1109/ACC.1997.609105).
  - Overlap: L (numeric means and covariances; no decisions). Paper 1 can cite it as the standard "remedy by conservatism".
- **Mahler (2000), "Optimal/robust distributed data fusion: a unified approach", SPIE (200 cites).** Uses Finite-Set Statistics to propose two approaches to fusing outputs of geographically separated fusion systems: an optimal one and a robust one. The robust branch is the root usually credited for "generalized CI" (GCI). The abstract is truncated before the robust rule is described, so the GCI attribution is secondary knowledge. [V-abs, partial] — [doi:10.1117/12.395064](https://doi.org/10.1117/12.395064). Overlap: L.
- **Hurley (2002), "An information theoretic justification for covariance intersection and its generalization", FUSION (179 cites).** Shows CI is equivalent to a log-linear (geometric-mean) combination of two Gaussians, which opens the way to Chernoff-type fusion of general densities. [V-abs] — [doi:10.1109/ICIF.2002.1021196](https://doi.org/10.1109/ICIF.2002.1021196). Overlap: L.
- **Uhlmann (2003), "Covariance consistency methods for fault-tolerant distributed data fusion", Information Fusion (207 cites).** [V-meta; content unverified.] It is commonly cited for CI and covariance union (handling inconsistent estimates), but the abstract was not retrieved. — [doi:10.1016/S1566-2535(03)00036-8](https://doi.org/10.1016/S1566-2535(03)00036-8). Overlap: L.
- **Julier & Uhlmann (2007, online 2006), "Using covariance intersection for SLAM", Robotics and Autonomous Systems (207 cites).** [V-meta; content unverified] — [doi:10.1016/j.robot.2006.06.011](https://doi.org/10.1016/j.robot.2006.06.011). Overlap: L.
- **Bailey, Julier, Agamennoni (2012), "On conservative fusion of information with unknown non-Gaussian dependence", FUSION (122 cites).**
  - Defines consistency and conservativeness for non-Gaussian fusion in terms of "counting of common information".
  - Shows the normalised weighted geometric mean (NWGM) generalizes CI and is conservative. [V-abs]. No DOI found; verified via [OpenAlex record](https://api.openalex.org/works?search=On%20conservative%20fusion%20of%20information%20with%20unknown%20non-Gaussian%20dependence).
  - Overlap: L. It is the clearest formal statement that the core error is counting common information twice, for arbitrary densities.

#### Explicit "data incest" papers in sensor networks
- **McLaughlin, Krishnamurthy, Challa (2003), "Managing data incest in a distributed sensor network", ICASSP (27 cites).**
  - Defines data incest as reusing identical information as though it were independent. Says it usually falsely raises confidence in a biased overall estimate, and that it is fundamental in many-to-many (all-to-all) networks.
  - Gives a fusion strategy that produces incest-free estimates for that structure. [V-abs] — [doi:10.1109/ICASSP.2003.1199920](https://doi.org/10.1109/ICASSP.2003.1199920). OpenAlex lists 2004.
  - Overlap: M on the mechanism (identical information reused, giving false confidence in a biased estimate, which matches P1's "one misperception copied"). L on everything else (numeric tracking; no decisions).
- **McLaughlin, Evans, Krishnamurthy (2003), "Data incest removal in a survivable estimation fusion architecture", FUSION (21 cites).** Treats decentralized fusion as an out-of-sequence-measurement (OOSM) problem in a full-information network with fixed link delays. Nodes communicate fused posteriors whose conditioning set records which individual measurements went in. [V-abs] — [doi:10.1109/ICIF.2003.177451](https://doi.org/10.1109/ICIF.2003.177451). Overlap: L. The 2026 "Epistemic Sybil" LLM preprint cites it (see KQ2).
- **Chang, Chong, Mori (2010), "Analytical and Computational Evaluation of Scalable Distributed Fusion Algorithms", IEEE TAES (77 cites).**
  - Notes that pedigree-based distributed fusion may not scale under limited bandwidth.
  - Evaluates five rules analytically against centralized fusion: channel-filter, naive, Chernoff, Shannon and Bhattacharyya.
  - Finds the channel filter, a first-order approximation of information-graph fusion, is the only consistent one. [V-abs] — [doi:10.1109/TAES.2010.5595611](https://doi.org/10.1109/TAES.2010.5595611).
  - Overlap: L. It is the direct evidence that naive fusion (counting everything as new) is inconsistent.
- **Üney, Clark, Julier (2013), "Distributed Fusion of PHD Filters Via Exponential Mixture Densities", IEEE J. Sel. Topics Signal Processing (232 cites).** Notes that consistent optimal fusion without double counting can be guaranteed only for highly constrained network architectures. Builds a consistent multi-target method from generalized CI (exponential mixture densities) and random finite sets. [V-abs] — [doi:10.1109/JSTSP.2013.2257162](https://doi.org/10.1109/JSTSP.2013.2257162). Overlap: L.
- **Abu Bakr & Lee (2017), "Distributed Multisensor Data Fusion under Unknown Correlation and Data Inconsistency", Sensors (105 cites; review).**
  - Treats data incest, rumor propagation and double counting as one phenomenon: data used several times without the system knowing.
  - Two causes: information recirculating through cyclic paths, or the same information reaching the fusion node by several paths.
  - Consequence: fusing without removing common information underestimates the error covariance. [V-abs + snippet] — [doi:10.3390/s17112472](https://doi.org/10.3390/s17112472).
  - Overlap: L–M. The "several paths from one source" case is structurally the same as P1's one LLM call writing into several memory stores.
- **Julier (UCL eprint 15814, title not confirmed; possibly the FUSION 2009 paper on the degree of independent information).** A search snippet says that failing to account for dependency causes double counting, overconfident estimates and possibly catastrophic filter failure. It also says exact cancellation of common information needs global network knowledge. [snippet only — full text returned HTTP 403; unverified] — [UCL Discovery PDF](https://discovery-pp.ucl.ac.uk/id/eprint/15814/1/15814.pdf). Overlap: L.
- **Fitzgerald (1971), "Divergence of the Kalman filter", IEEE Trans. Automatic Control (426 cites).**
  - Shows mean-square error can grow without bound when the filter model is wrong, distinguishing "true" from "apparent" divergence.
  - Shows divergence may or may not be fixed by assuming more process noise. [V-abs] — [doi:10.1109/TAC.1971.1099836](https://doi.org/10.1109/TAC.1971.1099836).
  - Overlap: L. It is not about double counting. It is the classic citation for the general mechanism: a filter that is too confident stops learning from new measurements.

#### Krishnamurthy and colleagues: data incest in Bayesian social learning
- **Hamdi & Krishnamurthy (2013), "Removal of Data Incest in Multi-agent Social Learning in Social Networks", arXiv:1309.6687.**
  - Model: agents update private beliefs by Bayes' rule, then broadcast a finite action (a rating).
  - Calls the resulting repeated reuse of information "data incest" (misinformation propagation) and describes it as ruinous.
  - Formulates incest management on a directed acyclic graph, derives topology conditions under which incest can be removed, and gives a removal algorithm. [V-abs] — [arXiv:1309.6687](https://arxiv.org/abs/1309.6687). Overlap: M (agents act on the incest-corrupted public belief; no verification or sensing choice).
- **Krishnamurthy & Hoiles (2014), "Online Reputation and Polling Systems: Data Incest, Social Learning, and Revealed Preferences", IEEE Trans. Computational Social Systems (18 cites).**
  - Mechanism: agents mistakenly treat gathered information as independent, which yields overconfidence and bias in state estimates.
  - Theory: necessary and sufficient conditions on the information-exchange graph for mitigating incest, plus removal algorithms.
  - Experiments: human-subject experiments show the effect of social influence and data incest on decision-making, and indicate that social-learning protocols must be designed carefully to handle incest.
  - Application: an expectation-polling example. [V-abs] — [doi:10.1109/TCSS.2014.2377891](https://doi.org/10.1109/TCSS.2014.2377891); [arXiv:1501.00994](https://arxiv.org/abs/1501.00994).
  - Overlap: M. It gives the overconfidence claim and evidence that incest changes decisions, but the decisions are ratings, not verification or sensing, and there is no causal copy-count manipulation of an AI agent.
  - The search tool reported that incest removal lowered MSE at the most informed node from about 0.31 to about 0.14 in a corporate-network example. [snippet; unverified]
- **Krishnamurthy & Poor (2013), "Interactive Sensing in Social Networks", arXiv:1312.7630.** Among other questions, asks how protocols can prevent data incest in online reputation systems, and how a global decision maker can use socially interacting sensors to detect change. [V-abs] — [arXiv:1312.7630](https://arxiv.org/abs/1312.7630). Author conflict: OpenAlex lists Krishnamurthy & Poor; the search tool attributed it to Hamdi & Krishnamurthy. Overlap: L–M.
- **Krishnamurthy, Gharehshiran, Hamdi (2014), "Interactive Sensing and Decision Making in Social Networks", Foundations and Trends in Signal Processing.** Monograph and tutorial on sensing and decision making in social networks. [V-abs via Crossref, partial] — [doi:10.1561/2000000048](https://doi.org/10.1561/2000000048). Overlap: L–M.
- **Krishnamurthy (2016), "Partially Observed Markov Decision Processes", Cambridge Univ. Press, Ch. "Multi-agent sensing: social learning and data incest".** Book version of the same model: incest gives overconfidence and bias, plus graph conditions and removal algorithms. [V-abs] — [doi:10.1017/CBO9781316471104.007](https://doi.org/10.1017/CBO9781316471104.007). Overlap: M.

#### Multi-robot localization and SLAM (double counting as a consistency problem)
- **Fox, Burgard, Kruppa, Thrun (2000), "A Probabilistic Approach to Collaborative Multi-Robot Localization", Autonomous Robots (692 cites).** [V-meta; content unverified.] Often cited for a heuristic that limits reuse of the same evidence between robots; the abstract was not retrieved. — [doi:10.1023/A:1008937911390](https://doi.org/10.1023/A:1008937911390). Overlap: L.
- **Howard, Matarić, Sukhatme (2003), "Putting the 'I' in 'team': an ego-centric approach to cooperative localization", ICRA (157 cites).** Each robot broadcasts relative-pose measurements, and each robot processes them independently into an ego-centric particle-filter estimate. [V-abs]. The abstract does not mention circular updates, so any double-counting claim about this paper is unverified. — [doi:10.1109/ROBOT.2003.1241702](https://doi.org/10.1109/ROBOT.2003.1241702). Overlap: L.
- **Bahr, Walter, Leonard (2009), "Consistent cooperative localization", ICRA.** [V-meta via Crossref; content unverified.] Usually cited for bookkeeping that avoids double counting in AUV cooperative localization. — [doi:10.1109/ROBOT.2009.5152859](https://doi.org/10.1109/ROBOT.2009.5152859). Overlap: L.
- **Leung, Barfoot, Liu (2010), "Decentralized Localization of Sparsely-Communicating Robot Networks: A Centralized-Equivalent Approach", IEEE T-RO (106 cites).** Proves a robot needs only its own knowledge of the network topology to produce the centralized-equivalent estimate whenever that is possible. [V-abs] — [doi:10.1109/TRO.2009.2035741](https://doi.org/10.1109/TRO.2009.2035741). Overlap: L.
- **Indelman, Gurfil, Rivlin, Rotstein (2012), "Graph-based distributed cooperative navigation for a general multi-robot measurement model", IJRR (42 cites).** Each robot keeps a graph of all multi-robot measurement updates and computes cross-correlation terms on demand, to keep fusion consistent. [V-abs] — [doi:10.1177/0278364912446325](https://doi.org/10.1177/0278364912446325). Overlap: L.
- **Cunningham, Indelman, Dellaert (2013), "DDF-SAM 2.0: Consistent distributed smoothing and mapping", ICRA (171 cites).** Introduces "anti-factors" that subtract information already shared, avoiding double counting without the over-conservative scheme of DDF-SAM 1.0. [V-abs] — [doi:10.1109/ICRA.2013.6631323](https://doi.org/10.1109/ICRA.2013.6631323). Overlap: L.
- **Carrillo-Arce, Nerurkar, Gordillo, Roumeliotis (2013), "Decentralized multi-robot cooperative localization using covariance intersection", IROS (185 cites).** CI-based cooperative localization that scales O(N), is provably consistent and handles asynchronous communication. [V-abs] — [doi:10.1109/IROS.2013.6696534](https://doi.org/10.1109/IROS.2013.6696534). Overlap: L.
- **Chang, Chen, Mehta (2021), "Resilient and Consistent Multirobot Cooperative Localization With Covariance Intersection", IEEE T-RO (66 cites).** CI with an explicit communication update. Analyzes covariance boundedness against communication and observation graphs. [V-abs] — [doi:10.1109/TRO.2021.3104965](https://doi.org/10.1109/TRO.2021.3104965). Overlap: L.

#### Recent decentralized-Bayesian work that uses the term "rumor propagation"
- **Langevin, Valtorta, Bloemeke (2010), "Agent-encapsulated Bayesian networks and the rumor problem", AAMAS (extended abstract).** Agents that exchange marginal posteriors on shared variables produce "rumors", i.e. double counting. The authors compensate by passing joint probabilities instead. [V-abs] — [PDF](https://www.cse.sc.edu/~mgv/papers/AAMAS2010.pdf). Overlap: L–M. These are symbolic multi-agent beliefs (closer to LLM agents than covariances), but there are no decisions.
- **Dagan & Ahmed (2022/23), "Exact and Approximate Heterogeneous Bayesian Decentralized Data Fusion", IEEE T-RO (14 cites).** Uses conditional independence to derive exact and approximate factorized fusion rules for agents with overlapping state subsets. Extends the channel filter to conservative heterogeneous fusion. [V-abs] — [doi:10.1109/TRO.2022.3226115](https://doi.org/10.1109/TRO.2022.3226115); [arXiv:2101.11116](https://arxiv.org/abs/2101.11116). Overlap: L.
- **Funk, Dagan, Noack, Ahmed (2023), "Exploiting Structure for Optimal Multi-Agent Bayesian Decentralized Estimation", arXiv:2307.10594.** Calls rumor propagation (double counting) the key challenge: previously sent data circulates back to the sender. CI handles it but is often over-conservative; exploiting independence structure (multiple weights) gives tighter bounds. [V-abs] — [arXiv:2307.10594](https://arxiv.org/abs/2307.10594). Overlap: L.

#### Adjacent pointers (economics and psychology; A2 covers these in depth)
- DeMarzo, Vayanos, Zwiebel (2003), QJE, "persuasion bias" (958 cites). Agents fail to adjust for repetition of information they receive. Beliefs converge to a weighted average whose weights depend on network position. [V-abs] — [doi:10.1162/00335530360698469](https://doi.org/10.1162/00335530360698469).
- Eyster & Rabin (2010), AEJ: Micro, "Naïve Herding in Rich-Information Settings" (205 cites). Naive agents over-weight early signals embedded in later actions. They herd on wrong actions with positive probability even with rich information, and become fully confident even when wrong. [V-abs] — [doi:10.1257/mic.2.4.221](https://doi.org/10.1257/mic.2.4.221).
- Enke & Zimmermann (2019; online 2017), Review of Economic Studies (185 cites). Lab evidence that many people treat correlated signals as independent. The result is overshooting beliefs and excessive confidence swings, which in an asset market create bubbles and crashes. [V-abs] — [doi:10.1093/restud/rdx081](https://doi.org/10.1093/restud/rdx081).
- Weaver, Garcia, Schwarz, Miller (2007), JPSP: one person repeating an opinion makes it seem widely held, even when people know it is one speaker. [V-abs] — [doi:10.1037/0022-3514.92.5.821](https://doi.org/10.1037/0022-3514.92.5.821).
- Yousif, Aboody, Keil (2019), Psychological Science: people are equally confident in a "false consensus" (one primary source) and a true consensus (independent sources). [V-abs] — [doi:10.1177/0956797619856844](https://doi.org/10.1177/0956797619856844).
- Whalen, Griffiths, Buchsbaum (2017/18), Cognitive Science, a counterpoint: people are sensitive to several patterns of dependence in testimony. [V-abs] — [doi:10.1111/cogs.12485](https://doi.org/10.1111/cogs.12485).

### Inferences
- What is "proved" about overconfidence comes in three kinds:
  - quantified covariance underestimation (Bar-Shalom & Campo 1986);
  - proofs that remedies are consistent (CI 1997; NWGM 2012; channel filter consistent in Chang et al. 2010);
  - necessary-and-sufficient graph conditions for removing incest (Krishnamurthy & Hoiles 2014).
- The overconfidence claim at the heart of P1 is therefore textbook-level for estimation. Paper 1 should cite it as background, not claim it as a finding.
- Abu Bakr & Lee's "same information via several paths to the fusion node" is the exact structural analog of P1's "one LLM call writes the same misperception into task, object, spatial and experience memories". Paper 1 can borrow this vocabulary (multi-path double counting vs cyclic recirculation).
- The rumor-problem work on agents exchanging marginal beliefs (Langevin et al.) is the fusion work closest in spirit to LLM agents, because beliefs there are symbolic or discrete rather than covariances.

### Gaps
- The abstracts of Grime & Durrant-Whyte 1994, Uhlmann 2003, Sijs & Lazar 2012, Battistelli & Chisci 2014, Fox et al. 2000, Bahr et al. 2009 and Julier & Uhlmann 2007 were not available through OpenAlex, Semantic Scholar or Crossref. Their content descriptions are standard attributions, not verified here.
- The title of the UCL Julier eprint behind the "catastrophic filter failure" statement could not be confirmed (HTTP 403).
- The full texts of the Krishnamurthy & Hoiles human-subject experiments were not read. What subjects decided, and the effect sizes, are unverified.

## KQ2 — Does any work in sensor management or active sensing show that double-counted or correlated information makes a system under-sample, stop sensing early, or skip verification?

### Takeaway
No classic data-incest or sensor-management paper was found that explicitly shows double counting causes under-sampling or skipped verification. That literature stops at estimation consistency.

The link does exist in neighboring work:
- **Strongest robotics evidence, active perception (Velez et al., JAIR 2012):** a planner that treats correlated repeated observations as independent became overconfident. It over-valued information from nearby observations, took shorter sensing paths, and detected objects significantly worse.
- **Swarm decision-making (Bayes Bots, ICRA 2020):** fewer, less-correlated observations improved go/no-go decision accuracy.
- **2026 multi-robot active-inference preprint:** double counting at the planning stage produces redundant exploration.
- **Closest overall precedent, a 2026 LLM preprint ("Copies or Sources?"):** forwarded copies raise LLM aggregators' commit-instead-of-query rate, and the size of the effect depends on the model.

None of these involves an embodied LLM agent's internal multi-store memory, real re-observation actions, or tracking of error persistence over time.

### Cited Findings

#### Robotics and active sensing
- **Velez, Hemann, Huang, Posner, Roy (2012), "Modelling Observation Correlations for Active Exploration and Robust Object Detection", Journal of Artificial Intelligence Research 44:423–453 (12 cites); arXiv:1401.4612.** [V-full: passages extracted from the arXiv PDF]
  - Method: learns an object-detector sensor model with spatial correlation between successive measurements, and plans viewpoints by forward search.
  - Core argument: typical estimation and planning algorithms wrongly assume measurements are conditionally independent given robot pose. A stationary robot facing a static scene gains no new information after a handful of images.
  - Planning consequence: with the correlation model, the mutual information of further measurements at a visited waypoint goes to zero, so the robot prefers new waypoints to repeated measurements in the same place.
  - Head-to-head comparison: the baseline was the RTBSS online POMDP planner, which could use only the independent part of the sensor model.
    - Its predicted belief updates were overconfident, giving shorter path lengths but worse precision–recall.
    - In the multi-object scenario, the lack of a correlation model made it overconfident in its belief, and it performed significantly worse on precision–recall than the correlation-aware planner.
    - Trajectory-density analysis: the authors write that "RTBSS over-valued the information gained from nearby observations". The correlation-aware planner spread its detections apart.
  - Links: [doi:10.1613/jair.3516](https://doi.org/10.1613/jair.3516); [arXiv:1401.4612](https://arxiv.org/abs/1401.4612).
  - Overlap: M–H on mechanism (correlated evidence treated as independent → overconfident belief → less or poorer further sensing → more errors). Differences from P1: correlation is in repeated sensor readings, not duplicated memory records; the planner is a classical POMDP or forward search, not an LLM; there is no provenance manipulation.
  - This is the best robotics citation for P1's "overconfidence reduces verification" link.
- **Ebert, Gauci, Mallmann-Trenn, Nagpal (2020), "Bayes Bots: Collective Bayesian Decision-Making in Decentralized Robot Swarms", ICRA (49 cites).** [V-abs]
  - Method: robots keep Beta-type beliefs about a binary environmental feature, share information locally, and commit to a go/no-go decision once a credible-interval threshold is crossed.
  - Findings: fewer, less-correlated observations can improve decision accuracy; a good prior and threshold allow fast decisions at a small accuracy cost; bio-inspired positive feedback improved both speed and accuracy.
  - The detailed mechanism (frequent, spatially correlated samples counted as independent, leading to premature, less accurate commitment) comes from the body, which could not be extracted. That detail is unverified.
  - Link: [doi:10.1109/ICRA40945.2020.9196584](https://doi.org/10.1109/ICRA40945.2020.9196584).
  - Overlap: M. Correlated evidence affects a stopping (commit) decision, but these are swarm robots with numeric beliefs and no LLM or memory.
- **Siemensma, Chiu, Ramshanker, Nagpal, Haghighat (2024), "Collective Bayesian Decision-Making in a Swarm of Miniaturized Robots for Surface Inspection", ANTS 2024 (LNCS).**
  - Finding: spatial correlation among observations makes beliefs fluctuate, which lengthens decision times and lowers accuracy. A "soft-feedback" sharing strategy improves convergence.
  - Reported in the search snippet of arXiv v2: up to 20.52% shorter mean decision time at 0.78% accuracy loss.
  - Verification: setup [V-abs, Semantic Scholar, truncated]; correlation claim and numbers [snippet]. Links: [doi:10.1007/978-3-031-70932-6_5](https://doi.org/10.1007/978-3-031-70932-6_5); [arXiv:2404.08390](https://arxiv.org/abs/2404.08390).
  - Overlap: L–M. Note the direction differs: here correlation slows decisions rather than causing early stopping.
- **Wu, Imani, Kamara, Islam, Ghoreishi, Imani (2026, submitted 15 Sep 2026), "Exact Fusion and Coordinated Exploration in Multi-Robot Active Inference", arXiv:2609.17384 (preprint).** [V-abs via arXiv page and OpenAlex]
  - Problem: under conjugate exponential-family beliefs, a team counts the shared belief once per robot in two places:
    - at fusion, the product of local posteriors counts the common prior n times;
    - at planning, every robot scores its plan under the same belief, so the team converges on the same unknown.
  - Fix: add evidence increments (realized at fusion, expected at planning) to the shared natural parameter.
  - Theory: corrected gains sum to the joint gain; the removed redundancy equals the total correlation of the planned observation streams; sequential commitment keeps a 1/2 greedy guarantee.
  - Experiments (cooperative RockSample, foraging, field monitoring): fixing fusion alone leaves exploration redundancy unchanged; anticipated-evidence planning removes it. No numbers were visible on the abstract page.
  - Link: [arXiv:2609.17384](https://arxiv.org/abs/2609.17384).
  - Overlap: M. Double counting demonstrably changes sensing decisions. Differences from P1: the effect is redundant over-sampling of the same target, not under-verification; the beliefs are numeric; there are no LLMs.
  - Useful for P1: correcting the fusion or belief alone did not fix the decision problem. That suggests P1 should measure decisions directly rather than infer them from calibration.
- **Liu, Liao, Ferrari (2020), "Rumor-robust Decentralized Gaussian Process Learning, Fusion, and Planning for Modeling Multiple Moving Targets", IEEE CDC.** Combines rumor-robust GP fusion (weighted exponential product plus Chernoff information) with decentralized information-driven sensing-path planning. The abstract reports no comparison of planning quality under naive (rumor-prone) fusion. [V-abs] — [doi:10.1109/CDC42340.2020.9304365](https://doi.org/10.1109/CDC42340.2020.9304365); [arXiv:2009.06021](https://arxiv.org/abs/2009.06021). Overlap: L–M (rumor robustness is put inside an active-sensing loop, but no effect of rumor on decisions is reported). A 2026 Autonomous Robots follow-up (Dong et al., [doi:10.1007/s10514-026-10249-4](https://doi.org/10.1007/s10514-026-10249-4)) handles target assignment and planning; its abstract does not mention rumor or double counting. [V-abs]

#### Social learning and stopping decisions (Krishnamurthy)
- **Krishnamurthy (2012), "Quickest Detection POMDPs With Social Learning: Interaction of Local and Global Decision Makers", IEEE Trans. Information Theory (57 cites).** [V-abs]
  - Setup: a global decision maker must stop (declare a change) using local agents' socially learned decisions.
  - Finding 1: the optimal cost is always at least that of classical quickest detection (Blackwell dominance).
  - Finding 2: the optimal stopping policy generally has multiple thresholds. Multi-agent sensor management with heterogeneous priors has a similar structure.
  - Link: [doi:10.1109/TIT.2012.2201372](https://doi.org/10.1109/TIT.2012.2201372).
  - Overlap: L–M. This is a formal case of corrupted or herded social information changing a stopping decision, but the mechanism is herding (information loss), not incest (double counting).
- **Krishnamurthy (2022), "Dynamics of Social Networks: Multi-agent Information Fusion, Anticipatory Decision Making and Polling", arXiv:2212.13323 (survey).** Surveys controlled sensing with social learning, including how risk-averse social learning affects quickest change detection. [V-abs] — [arXiv:2212.13323](https://arxiv.org/abs/2212.13323). Overlap: L–M.

#### LLM-era work that imports the data-incest framing (closest collisions with P1)
- **Gao, Li, Yu, Ren, Chen, Wang (2026, submitted 5 Oct 2026), "Copies or Sources? Measuring How LLM Aggregators Count Restated Evidence in Multi-Agent Systems", arXiv:2610.06192 (preprint).** [V-abs + HTML v1 read by an automated reader; re-check numbers against Tables 15–18 before citing]
  - Framing:
    - Calls copy-counting a data-incest violation.
    - Defines a "copy weight" from 0 (count sources) to 1 (count statements).
    - Cites Krishnamurthy & Hoiles (2014), which it says handles incest correction when the topology is known, and Julier & Uhlmann (1997).
  - Testbeds:
    - message logs with an exact Bayesian oracle;
    - web documents with appended copies;
    - logs written by LLM agent teams under four communication protocols (relays, shared boards, discussion rounds).
  - Models: GPT-6 Luna, GPT-6.1 Sol, Gemini 3.8 Flash, DeepSeek V4.1 Flash.
  - How much copies count:
    - Forwarded copies count as 0.06–0.42 of a new reading. Two copies raise reported evidence from 1 reading to 1.12–1.84 even though provenance labels mark them as copies.
    - Removing the forwarded-from labels raises this to 1.20–1.97.
  - Effect of interventions:
    - A one-paragraph declaration of what a copy contributes cuts copy weight to ≤0.08 on controlled logs.
    - A field guide that only explains the label barely helps.
    - A placebo note that draws attention to copies increases counting.
    - A reference-instead-of-restate protocol lowers belief-implied early commitment from 11.2% to 1.1%.
  - Decision analysis:
    - Model: commit pays ±1; a query costs c and buys one more reading of reliability r.
    - The belief-implied commitment boundary is swept from 1 to 3 readings.
    - Emitted-action experiments fix c = 0.05, with r set so that the boundary is two readings.
    - With two copies and no note, emitted commitment reaches 24.2% / 6.6% / 30.4% / 70.4% (Luna / Sol / Gemini / DeepSeek), 5.6–34.6 points above the single-reading condition.
    - The declaration lowers this to 2.4% / 1.0% / 6.8% / 45.6%.
    - Belief-implied early commitment grows with the number of copies (two to eight statements).
  - Scope limits: no embodied agents, robots, planners or memory stores. Agents never re-observe in an environment. "Query" is a recorded choice whose value comes from the cost model.
  - Stated limitations: labels correct, copies exact, full log visible; forged identifiers, altering relays and shared failure modes not modeled.
  - Links: [arXiv:2610.06192](https://arxiv.org/abs/2610.06192); [HTML v1](https://arxiv.org/html/2610.06192v1).
  - **Overlap: H.** It already shows, with causal copy-count and provenance-label manipulation across four LLMs, that copies inflate confidence and raise commit-instead-of-query choices.
  - Differences from P1:
    - The copies come from inter-agent forwarding in text logs, not one LLM call writing to several internal memory stores.
    - Queries are stylized, not real re-observation with real costs.
    - There is no perception, no embodied task, and no measurement of error persistence or spread to progress judgments and later actions.
    - It does not hold images and context length fixed.
- **Bara (2026, submitted 1 Sep 2026), "Epistemic Sybil Resistance: Multiplying AI Agents Without Multiplying Evidence", arXiv:2609.01873 (preprint, single author).** [V-abs + HTML read by an automated reader]
  - Method: Gaussian shared-root model; more than 20,000 calls to one "Haiku-class" LLM; task is estimating a fictional firm's quarterly revenue from memos.
  - Holding one evidence root fixed and raising reports from 1 to 32:
    - naive 95%-interval coverage falls from 0.940 to 0.263;
    - the calibration ratio rises from 0.961 to 5.473;
    - a provenance-aware aggregator stays at 0.850–0.940.
  - At 16 reports, naive coverage is 0.400 with one root and recovers to 0.927 with 16 roots.
  - Report-only deduplication tracks rationale wording rather than true ancestry.
  - No decisions or verification are measured. Cites McLaughlin, Evans & Krishnamurthy (2003); Das Sarma, Dong & Halevy (2011); Enke & Zimmermann (2019).
  - Link: [arXiv:2609.01873](https://arxiv.org/abs/2609.01873).
  - Overlap: H on "duplicates make LLM collective inference overconfident; provenance fixes calibration". L on verification and embodiment.

#### Not found / negative results of targeted searches
- Searches for double counting combined with sensor management, premature exploration termination, occupancy-grid map merging, negative information in decentralized search, or gating-induced track loss found no primary source that explicitly links double counting to early stopping or skipped re-observation. Snippet-level search answers explicitly said no such source was found:
  - [search: double counting / active sensing](https://dmlab.cs.umn.edu/new/papers/pmc09.pdf) (closest hit: duplicate counting of targets in sensor networks, not exploration);
  - [search: decentralized search](https://arxiv.org/pdf/1907.04008) (negative-information fusion in Gaussian-mixture search, no duplicate-counting analysis);
  - [search: map merging](https://pmc.ncbi.nlm.nih.gov/articles/PMC7730201).
- Grocholsky and Durrant-Whyte's information-theoretic decentralized control work (mutual-information-driven control of multiple UAVs, built on decentralized information filters) does not mention double counting in its abstract. [V-abs, snippet level] — [SPIE 2000](https://neurophotonics.spiedigitallibrary.org/conference-proceedings-of-spie/4196/0000/Information-theoretic-approach-to-decentralized-control-of-multiple-autonomous-flight/10.1117/12.403734.full); [Grocholsky thesis](https://ses.library.usyd.edu.au/handle/2123/796).

### Inferences
- Wherever a sensing policy is driven by value of information, by entropy reduction, or by a confidence threshold for stopping, a posterior inflated by double counting mechanically lowers the predicted value of more observations, so the policy stops or moves on earlier.
- Velez et al. 2012 is the cleanest empirical robotics confirmation of this mechanism: shorter paths, over-valued nearby observations, worse precision–recall. Bayes Bots is a weaker swarm-level one.
- Paper 1 can frame its verification result as the LLM-agent instance of a known estimation-and-decision mechanism, rather than as a brand-new phenomenon.
- Novelty against Gao et al. 2026 must be argued explicitly. Candidate differentiators, each confirmed absent from Gao et al. as read:
  1. Duplication is **intra-agent and architectural**: one perception call fans out into several memory stores. This is the "multiple paths from one source" form of incest, not inter-agent forwarding.
  2. Errors are **perceptual**, and verification is a **real embodied re-observation** with environment-defined cost.
  3. The study measures **error persistence and spread** into progress judgments and next actions over long horizons.
  4. Controls hold **images and context length fixed**.
- Gao et al.'s finding that labels alone help little, while a declaration of what a copy contributes helps a lot, is a strong prior for P1's provenance-label manipulation. P1 should expect plain provenance labels to be weak and should include a "copy-contribution declaration" arm.
- Gao et al. also find that most wrong emitted actions were action-stage errors (correct belief, wrong action), about 70–80% without the note. So P1 should measure belief and verification choice separately: overconfidence and under-verification can come apart.
- Wu et al. 2026 found that fixing fusion did not fix exploration redundancy. Together with the Gao et al. point above, this supports measuring decisions directly rather than treating calibration as a proxy.

### Gaps
- The Bayes Bots full text could not be parsed (no PDF tools available). The exact claim about observation intervals and correlation is unverified beyond the abstract.
- Siemensma et al.'s numbers are snippet-level only.
- The numbers for Gao et al. 2026 came from an automated reading of the arXiv HTML. The tables were not checked by eye.
- No primary source was found for the classical statistics claim that a sequential probability ratio test (SPRT) fed positively correlated samples as if independent stops too early and inflates error rates. It is a plausible textbook point; the search tool flagged it as general knowledge, not sourced.
- No source was found that explicitly ties data incest to validation-gate shrinkage and track loss in tracking. It remains an inference.
- Liu et al. 2016 (IEEE TSP), "Sensor Selection for Estimation with Correlated Measurement Noise" ([doi:10.1109/TSP.2016.2550005](https://doi.org/10.1109/TSP.2016.2550005)), is a possible lead on correlation-blind sensor selection. Only metadata was verified.
- The Grocholsky thesis and the Manyika & Durrant-Whyte book "Data Fusion and Sensor Management" were not read. They may discuss how double counting affects decentralized sensor management.

## KQ3 — What remedies exist (provenance or pedigree tracking, conservative fusion), and what do they assume?

### Takeaway
Remedies fall into three families:
1. **Exact bookkeeping of common information** (information graph, channel filters, pedigree, anti-factors, joint-message passing, DAG-based incest removal). Needs knowledge of the information-flow topology or per-item provenance, and often tree topologies or extra communication.
2. **Conservative fusion** (CI and its generalizations: Chernoff/GCI/WEP/NWGM, inverse CI, ellipsoidal intersection, conservative linear estimators). Stays consistent under unknown correlation but needs numeric means and covariances or densities over a shared state, and gives away information.
3. **Sampling and protocol design** (less-correlated sampling, correlation-aware sensor models, reference-instead-of-restate messaging, copy declarations).

For LLM memory, only family 1 (provenance) and family 3 (protocols and declarations) carry over directly. Family 2 assumes numeric uncertainty that textual memories lack.

### Cited Findings

#### Exact bookkeeping (pedigree and provenance)
- Information graph: the theoretical basis for exact distributed estimation in arbitrary architectures. It requires modeling the information flow. [V-abs] — [Liggins et al. 1997](https://doi.org/10.1109/JPROC.1997.554211).
- Pedigree-based fusion may not scale when the communication architecture is not fixed and bandwidth is limited. Among scalable rules, only the channel filter (first-order approximation to information-graph fusion) was consistent. [V-abs] — [Chang, Chong, Mori 2010](https://doi.org/10.1109/TAES.2010.5595611).
- Channel filter: the original 1994 paper's content is unverified here. The tree-topology requirement is a standard attribution. Dagan & Ahmed extend channel filters to heterogeneous (partial-state) conservative fusion. [V-meta / V-abs] — [Grime & Durrant-Whyte 1994](https://doi.org/10.1016/0967-0661(94)90349-2); [Dagan & Ahmed 2022](https://doi.org/10.1109/TRO.2022.3226115).
- Without topology constraints, exact cancellation of common information needs global network knowledge. [snippet; unverified] — [Julier, UCL eprint](https://discovery-pp.ucl.ac.uk/id/eprint/15814/1/15814.pdf). Consistent optimal fusion without double counting can be guaranteed only for highly constrained architectures. [V-abs] — [Üney, Clark, Julier 2013](https://doi.org/10.1109/JSTSP.2013.2257162).
- Exact incest-free fusion in all-to-all networks:
  - by a fusion strategy designed for many-to-many links [V-abs] — [McLaughlin, Krishnamurthy, Challa 2003](https://doi.org/10.1109/ICASSP.2003.1199920);
  - by an OOSM formulation with fixed known link delays and posteriors that record which measurements they contain [V-abs] — [McLaughlin, Evans, Krishnamurthy 2003](https://doi.org/10.1109/ICIF.2003.177451).
- Robotics bookkeeping variants:
  - centralized-equivalent estimation from a robot's own knowledge of network topology [V-abs] — [Leung, Barfoot, Liu 2010](https://doi.org/10.1109/TRO.2009.2035741);
  - locally maintained measurement graphs giving correlation terms on demand [V-abs] — [Indelman et al. 2012](https://doi.org/10.1177/0278364912446325);
  - anti-factors subtracting already-shared information in factor-graph SLAM [V-abs] — [Cunningham, Indelman, Dellaert 2013](https://doi.org/10.1109/ICRA.2013.6631323).
- Symbolic multi-agent beliefs: rumors are compensated by exchanging joint rather than marginal probabilities, at a complexity the authors analyze. [V-abs] — [Langevin, Valtorta, Bloemeke 2010](https://www.cse.sc.edu/~mgv/papers/AAMAS2010.pdf).
- Social learning: incest removal on the information-exchange DAG, with necessary-and-sufficient conditions on the graph. It assumes the exchange topology is known. [V-abs] — [Hamdi & Krishnamurthy 2013](https://arxiv.org/abs/1309.6687); [Krishnamurthy & Hoiles 2014](https://doi.org/10.1109/TCSS.2014.2377891). The 2026 LLM preprint states this requirement too. [V-full, automated read] — [Gao et al. 2026](https://arxiv.org/html/2610.06192v1).
- Conjugate exponential-family teams: realized and expected "evidence increments" on a shared natural parameter give the exact centralized posterior and remove planning redundancy. Assumes conjugacy and a shared parameterization. [V-abs] — [Wu et al. 2026](https://arxiv.org/abs/2609.17384).
- LLM aggregation: provenance-aware aggregation restores calibration. It needs known ancestry and a dependence model (Gaussian, equicorrelated numeric noise), and report text alone cannot reliably recover ancestry. [V-abs + HTML] — [Bara 2026](https://arxiv.org/abs/2609.01873).

#### Conservative fusion
- CI: consistent for any unknown cross-correlation. Needs a mean and covariance (second-order, roughly Gaussian) for each estimate of the same state. [V-abs] — [Julier & Uhlmann 1997](https://doi.org/10.1109/ACC.1997.609105).
- CI equals a log-linear (geometric) combination of Gaussians, which generalizes to Chernoff fusion of arbitrary densities. [V-abs] — [Hurley 2002](https://doi.org/10.1109/ICIF.2002.1021196).
- Normalised weighted geometric mean for non-Gaussian densities, shown to be conservative with respect to counting common information. [V-abs] — [Bailey, Julier, Agamennoni 2012](https://api.openalex.org/works?search=On%20conservative%20fusion%20of%20information%20with%20unknown%20non-Gaussian%20dependence).
- Multi-object versions:
  - FISST optimal and robust distributed fusion [V-abs, partial] — [Mahler 2000](https://doi.org/10.1117/12.395064);
  - generalized CI with exponential mixture densities for PHD filters [V-abs] — [Üney et al. 2013](https://doi.org/10.1109/JSTSP.2013.2257162);
  - generalized CI with labeled random finite sets, which depends on labels being consistent across nodes and degrades badly otherwise [V-abs] — [Li et al. 2017](https://doi.org/10.1109/TSP.2017.2760286);
  - weighted exponential product with Chernoff weights for Gaussian-process fusion [V-abs] — [Liu, Liao, Ferrari 2020](https://doi.org/10.1109/CDC42340.2020.9304365).
- The cost of conservatism: the CI bound is not tight and often over-conservative. Exploiting the problem's independence structure (several weights) gives tighter consistent bounds. [V-abs] — [Funk et al. 2023](https://arxiv.org/abs/2307.10594).
- Less conservative variants:
  - inverse CI, which bounds the largest possible common information and removes it [V-abs] — [Noack, Sijs, Hanebeck 2017 (FUSION)](https://doi.org/10.23919/ICIF.2017.8009694); [Noack et al. 2017 (Automatica), V-meta](https://doi.org/10.1016/j.automatica.2017.01.019);
  - ellipsoidal intersection [V-meta; content unverified] — [Sijs & Lazar 2012](https://doi.org/10.1016/j.automatica.2012.05.077);
  - conservative linear unbiased estimation when covariances are partly known, with fundamental bounds [V-abs] — [Forsling et al. 2022](https://doi.org/10.1109/TSP.2022.3179841);
  - a proof that CI is optimal among conservative unbiased linear rules even for partial-state fusion [V-abs] — [Trumpf, Zamani, Manzie 2025](https://arxiv.org/abs/2507.16216).
- Consensus on probability densities via Kullback–Leibler averaging, with guaranteed stability of distributed estimation. [V-meta; content title-level] — [Battistelli & Chisci 2014](https://doi.org/10.1016/j.automatica.2013.11.042). Robot cooperative localization with CI:
  - linear complexity, provably consistent [V-abs] — [Carrillo-Arce et al. 2013](https://doi.org/10.1109/IROS.2013.6696534);
  - resilient to communication and observation failures [V-abs] — [Chang, Chen, Mehta 2021](https://doi.org/10.1109/TRO.2021.3104965).

#### Sampling and protocol design
- Fewer, less-correlated observations improve swarm go/no-go accuracy. [V-abs] — [Ebert et al. 2020](https://doi.org/10.1109/ICRA40945.2020.9196584).
- Modeling observation correlation (a convex mix of an independent and a history-dependent model) inside the planner makes the robot seek new viewpoints. [V-full] — [Velez et al. 2012](https://doi.org/10.1613/jair.3516).
- For LLM aggregators, Gao et al. 2026 report [V-full, automated read] — [arXiv HTML](https://arxiv.org/html/2610.06192v1):
  - declaring what a copy contributes is strong (copy weight ≤0.08);
  - explaining the label alone is weak;
  - a placebo that draws attention to copies backfires;
  - a reference-not-restate protocol sharply cuts premature commitment.

### Inferences
- The assumptions that block direct transfer of conservative fusion to LLM memory:
  - numeric uncertainty (covariances or densities);
  - a common state definition, with labels consistent across nodes;
  - and, for exact methods, known topology or pedigree metadata.
- An LLM agent's memory stores hold text, not covariances. The transferable idea is pedigree, i.e. a source ID per record carried across stores, plus protocol design. Pedigree is what P1's provenance-label manipulation tests.
- P1 can position its remedy (if any) as "channel-filter-like bookkeeping for multi-store LLM memory". It should cite Chang et al. 2010 and Üney et al. 2013 for the classic scalability limits of pedigree tracking.

### Gaps
- The content of the ellipsoidal-intersection and KL-consensus papers was not verified beyond their titles.
- No source was found that evaluates any remedy's effect on downstream sensing or verification decisions. The exceptions are Wu et al. 2026 (planning redundancy) and Gao et al. 2026 (commit vs query).

## KQ4 — How much does each work overlap with the Paper 1 claim (H/M/L), and what is the key difference?

### Takeaway
- Two 2026 LLM preprints overlap highly:
  - Gao et al. (copies inflate evidence and raise commit-instead-of-query; provenance labels and declarations manipulated; four LLMs);
  - Bara (duplicates destroy calibration; provenance-aware aggregation fixes it).
- Velez et al. 2012 is the medium-to-high robotics precedent for overconfidence from correlated evidence reducing sensing quality.
- The classic fusion literature overlaps lowly; it supplies the mechanism and vocabulary.
- No work found combines: intra-agent multi-store memory duplication by one LLM call; an embodied task with real re-observation; copy-count and provenance interventions with images and context length fixed; verification measured across a cost sweep; and error persistence into progress judgments and next actions.

### Cited Findings
| Work | Overlap | Key difference from P1 |
|---|---|---|
| [Gao et al. 2026, arXiv:2610.06192](https://arxiv.org/abs/2610.06192) | **H** | Copies come from inter-agent forwarding in text logs; "query" is a stylized choice (emitted-action runs at fixed c = 0.05; belief boundary swept 1–3 readings); no embodiment, perception, memory stores, or error persistence. |
| [Bara 2026, arXiv:2609.01873](https://arxiv.org/abs/2609.01873) | **H** (overconfidence) / L (verification) | Numeric revenue estimation with one LLM; calibration only; no decisions or verification. |
| [Velez et al. 2012, JAIR](https://doi.org/10.1613/jair.3516) | **M–H** | Correlation in repeated sensor readings, not duplicated memory; classical POMDP or forward-search planner; shows the overconfidence → poorer sensing → worse detection chain. |
| [Wu et al. 2026, arXiv:2609.17384](https://arxiv.org/abs/2609.17384) | M | Double counting changes exploration, but as redundant over-sampling; conjugate numeric beliefs; robots, not LLMs. |
| [Ebert et al. 2020, Bayes Bots](https://doi.org/10.1109/ICRA40945.2020.9196584) | M | Correlated observations hurt go/no-go stopping accuracy; swarm with numeric beliefs; no provenance manipulation. |
| [Krishnamurthy & Hoiles 2014](https://doi.org/10.1109/TCSS.2014.2377891) | M | Data incest gives overconfidence and bias; human-subject decision effects; ratings, not verification; removal needs known topology. |
| [Hamdi & Krishnamurthy 2013](https://arxiv.org/abs/1309.6687) / [Krishnamurthy 2016 ch.](https://doi.org/10.1017/CBO9781316471104.007) | M | Theory of incest in social learning on DAGs; no sensing or verification choice. |
| [McLaughlin, Krishnamurthy, Challa 2003](https://doi.org/10.1109/ICASSP.2003.1199920) | M (mechanism) / L | Identical information reused, giving false confidence in a biased estimate; numeric tracking; no decisions. |
| [Eyster & Rabin 2010](https://doi.org/10.1257/mic.2.4.221) | M | Theory: naive double counting leads to full confidence in wrong actions; humans; no verification. |
| [Yousif et al. 2019](https://doi.org/10.1177/0956797619856844) | M | Humans equally confident in single-source "consensus"; no verification measure. |
| [Enke & Zimmermann 2019](https://doi.org/10.1093/restud/rdx081) | M–L | Humans treat correlated signals as independent, so beliefs overshoot; no information acquisition. |
| [Krishnamurthy 2012, TIT](https://doi.org/10.1109/TIT.2012.2201372) | L–M | Social learning (herding) changes the optimal stopping policy; not double counting. |
| [Liu, Liao, Ferrari 2020](https://doi.org/10.1109/CDC42340.2020.9304365) | L–M | Rumor-robust fusion inside active sensing; no reported effect of rumors on decisions. |
| [Siemensma et al. 2024](https://arxiv.org/abs/2404.08390) | L–M | Correlation makes beliefs fluctuate and decisions slower; swarm. |
| [Langevin et al. 2010](https://www.cse.sc.edu/~mgv/papers/AAMAS2010.pdf) | L–M | Rumors among agents exchanging symbolic marginal beliefs; no decisions. |
| [Abu Bakr & Lee 2017](https://doi.org/10.3390/s17112472) | L–M | Review; the "same information via several paths" case matches P1's fan-out structurally. |
| [Julier & Uhlmann 1997 (CI)](https://doi.org/10.1109/ACC.1997.609105), [Hurley 2002](https://doi.org/10.1109/ICIF.2002.1021196), [Bailey et al. 2012](https://api.openalex.org/works?search=On%20conservative%20fusion%20of%20information%20with%20unknown%20non-Gaussian%20dependence), [Üney et al. 2013](https://doi.org/10.1109/JSTSP.2013.2257162), [Noack et al. 2017](https://doi.org/10.23919/ICIF.2017.8009694), [Forsling et al. 2022](https://doi.org/10.1109/TSP.2022.3179841), [Funk et al. 2023](https://arxiv.org/abs/2307.10594), [Trumpf et al. 2025](https://arxiv.org/abs/2507.16216) | L | Conservative numeric fusion with consistency proofs; no agents' decisions; needs covariances or densities. |
| [Bar-Shalom & Campo 1986](https://doi.org/10.1109/TAES.1986.310815), [Liggins et al. 1997](https://doi.org/10.1109/JPROC.1997.554211), [Chong et al. 2000](https://doi.org/10.1109/62.821657), [Chang et al. 2010](https://doi.org/10.1109/TAES.2010.5595611), [Grime & Durrant-Whyte 1994](https://doi.org/10.1016/0967-0661(94)90349-2) | L | Classic quantification of, or bookkeeping for, common information in tracking. |
| [Leung et al. 2010](https://doi.org/10.1109/TRO.2009.2035741), [Indelman et al. 2012](https://doi.org/10.1177/0278364912446325), [Cunningham et al. 2013](https://doi.org/10.1109/ICRA.2013.6631323), [Carrillo-Arce et al. 2013](https://doi.org/10.1109/IROS.2013.6696534), [Chang, Chen, Mehta 2021](https://doi.org/10.1109/TRO.2021.3104965), [Bahr et al. 2009](https://doi.org/10.1109/ROBOT.2009.5152859) | L | Multi-robot localization and SLAM consistency; estimation only. |
| [Nakayashiki 2026, "When Stale Constraints Go Unchecked" (Zenodo)](https://doi.org/10.5281/zenodo.22108557) | M (verification budget) / L (duplication) | Agent inherits memory with a superseded constraint and has a scarce verification budget of two records, allocated by the agent or redirected to the provenance path. Staleness, not duplicate counting. [V-abs, truncated]; also listed as [arXiv:2608.25553](https://arxiv.org/pdf/2608.25553). |
| [Wang et al. 2026, MemGuard](https://arxiv.org/abs/2608.21867) | L–M | Memory banks pile up duplicate, stale and conflicting records; verifier signals are persisted as metadata; no causal copy-count test. [V-abs] |
| [Xu et al. 2026, Mandela effect in LLM multi-agent systems](https://arxiv.org/abs/2602.00428) | L–M | Collective false memory through social influence; no verification or embodiment. [V-abs] |
| [Yang et al. 2026, cross-user contamination](https://arxiv.org/abs/2604.01350) | L | Shared-state misapplication across users; not evidence counting. [V-abs] |
| [Li et al. 2026, MATE](https://arxiv.org/abs/2609.35808) | L | Embodied memory adaptation after retrieval; not duplication or verification. [V-abs] |

### Inferences
- **Not novel for P1:**
  - "duplicated or same-source evidence makes the receiver overconfident" (fusion since 1986–1997; social learning since 2003–2014; LLM aggregators in 2026);
  - "provenance-aware aggregation restores calibration" (Bara 2026);
  - "copies raise LLM commit-instead-of-query choices, and copy declarations reduce this" (Gao et al. 2026, text-only).
- **Probably novel, if executed as planned:**
  - copies created inside one agent's architecture (one perception call fanned out to several memory stores);
  - embodied, perception-grounded errors in BEHAVIOR-1K;
  - real re-observation as the verification action, with sweeps of environment-defined verification cost;
  - error persistence and spread to progress judgments and next actions;
  - controls holding images and context length fixed.
- Recommended framing: "multi-path data incest inside an agent's memory architecture". Name Gao et al. 2026 as the closest prior work, and contrast inter-agent message forwarding with intra-agent memory fan-out, and stylized queries with embodied verification.
- Two design ideas carried over from Gao et al.:
  - Measure the belief–action gap separately: their action-stage errors dominate.
  - Add a "copy-contribution declaration" arm, since labels alone may be weak.

### Gaps
- The full LLM landscape is not covered here. B1 (LLM repetition) and B2 (LLM sources/agents) may list further collisions, e.g. RAG duplicate-passage or repetition-induced confidence studies. The report writer should cross-check those files.
- All 2026 LLM items are preprints or Zenodo records, not peer-reviewed. Model names and numbers are as the papers report them.
