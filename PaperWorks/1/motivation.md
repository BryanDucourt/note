下面给出 **4 个 motivation × 每个 3 种实验方案**。这里的核心不是证明某个具体 workload 有问题，而是为实验一的主张服务：**计算特征可以在线识别应用场景，并指导调度策略参数选择**。这四个 motivation 已经在你们的大纲中定义为：Proxy metric mismatch、Static policy mismatch、Feature-policy correlation、Safety requirement。 

实验对象优先选择公开、认可度较高的 benchmark / 应用程序：TailBench、CloudSuite、DeathStarBench、BenchBase、YCSB、RocksDB `db_bench`、NAS Parallel Benchmarks、PARSEC/SPLASH、cyclictest/rt-tests、stress-ng 等。TailBench 明确面向 latency-critical applications，DeathStarBench 是开源 cloud microservices benchmark，CloudSuite 覆盖 online services 和 analytics workloads，NPB 是 NASA 的并行计算 benchmark，cyclictest 用于测量线程预期唤醒时间与实际唤醒时间的差异，BenchBase 是多 DBMS SQL benchmarking framework。 

---

# **Motivation 1：单一系统指标不足以反映应用目标**

## **要证明的结论**

**CPU utilization、平均 CPU share、平均吞吐等 proxy metric 不能充分解释应用真正关心的 p99 latency、jitter、deadline miss、wakeup latency 等目标。调度决策需要引入更接近应用目标和调度路径的计算特征。**

这个 motivation 可借鉴 eMFS 的论证方式：eMFS 明确指出，很多调度策略依赖 CPU utilization 这类 proxy metric，但它们与 end-to-end latency / throughput / SLO 并不直接对齐；eMFS 因此用 eBPF 采集更接近真实性能的 latency 信号。 

---

## **方案 1.1：Latency-critical 服务 + CPU/内存后台负载**

|**项目**|**设计**|
|---|---|
|**测试对象**|前台：TailBench 中的 `xapian`、`masstree`、`silo` 或 `sphinx`；后台：NPB `EP/CG/MG/FT`、PARSEC `blackscholes/x264/swaptions`，或 stress-ng 的 CPU/cache/vm stressor|
|**为什么合适**|TailBench 专门用于测量 tail latency；NPB/PARSEC/stress-ng 可以稳定制造 CPU-bound、cache-sensitive、memory-sensitive 后台负载|
|**实验问题**|在 CPU utilization 相近的情况下，前台 p99 latency 是否显著不同？|
|**实验做法**|1. 固定前台 TailBench QPS，例如 50%、70%、90% saturation load；2. 叠加不同后台负载：CPU-bound、cache-heavy、memory-heavy；3. 通过 cgroup quota 或调节后台线程数，使总 CPU utilization 保持相近，例如 70±3%、85±3%；4. 每组运行 5–10 次，随机化运行顺序，避免温度和频率漂移|
|**采集指标**|CPU utilization、per-core utilization、p50/p95/p99 latency、wakeup latency、runqueue latency、context switches、migrations、IPC、LLC miss rate|
|**分析/检验**|1. 画 **CPU utilization vs p99 latency 散点图**，观察同一 utilization 区间内 p99 是否大幅分散；2. 计算 CPU utilization 与 p99 latency 的 Pearson/Spearman correlation；3. 用多元回归或随机森林比较：只用 CPU utilization 预测 p99 vs 加入 runqueue latency/LLC miss/context switch 后预测 p99；4. 用 Mann-Whitney U test 或 bootstrap CI 检验同 utilization 下不同后台负载的 p99 差异|
|**预期 Takeaway**|CPU utilization 相近不代表应用性能相近；调度路径特征和微架构特征更能解释尾延迟|
|**推荐图表**|Fig. 1(a) CPU utilization vs p99 latency；Fig. 1(b) runqueue latency timeline；Fig. 1(c) feature importance bar|

**优先级：高。**  
这是最适合作为 motivation 第一张图的方案，因为 TailBench 的目标和“尾延迟不可由 CPU utilization 解释”高度一致。

---

## **方案 1.2：周期实时任务 + 后台干扰负载**

|**项目**|**设计**|
|---|---|
|**测试对象**|前台：`cyclictest` 或 ROS 2 real-time benchmarks；后台：stress-ng `cpu/cache/vm/matrix`、NPB `CG/MG/FT`、STREAM|
|**为什么合适**|cyclictest 是实时 Linux 中常用的 wakeup latency / jitter 测试工具；ROS 2 real-time benchmarks 更接近机器人/边缘场景|
|**实验问题**|CPU utilization 不高时，是否仍然可能出现 deadline miss 或 wakeup latency spike？|
|**实验做法**|1. 运行 `cyclictest -p high -i 1000us` 或多个周期线程，设定周期如 1ms/5ms/10ms；2. 后台运行不同类型负载：纯 CPU、cache pressure、memory bandwidth、fork/exec pressure；3. 控制 CPU utilization 在 40%、60%、80% 三档；4. 比较默认调度、SCHED_FIFO、SCHED_DEADLINE、static cgroup、简单 sched_ext 策略|
|**采集指标**|max latency、p99/p99.9 wakeup latency、deadline miss rate、CPU utilization、IRQ/softirq 时间、context switches、runqueue latency、migration rate|
|**分析/检验**|1. 对比同 CPU utilization 下的 max wakeup latency；2. 检验 deadline miss 是否随 CPU utilization 单调增加，还是与调度路径/中断/迁移更相关；3. 用 quantile plot 展示 tail 行为；4. 用 threshold analysis：当 utilization < 70% 但 p99 wakeup latency > deadline 时，标记为 proxy metric failure|
|**预期 Takeaway**|即使 CPU 没有饱和，实时/周期任务也可能因调度延迟、迁移、cache 干扰而错过 deadline|
|**推荐图表**|Fig. 2(a) utilization vs wakeup p99；Fig. 2(b) deadline miss timeline；Fig. 2(c) migration/context switch 与 jitter 的相关性|

Linux Foundation 文档说明 cyclictest 测量线程预期 wake-up time 与实际 wake-up time 之间的差异，因此它适合用来证明“CPU 使用率之外的调度路径延迟”问题。 

**优先级：高。**  
这个方案非常适合你们的终端/边缘/具身智能场景，能把“平均资源利用率”和“实时性目标”区分开。

---

## **方案 1.3：数据库/键值存储前台请求 + 后台批处理**

|**项目**|**设计**|
|---|---|
|**测试对象**|前台：BenchBase TPC-C、YCSB、RocksDB `db_bench readwhilewriting`；后台：TPC-H query loop、RocksDB compaction pressure、NPB/PARSEC|
|**为什么合适**|BenchBase/TPC-C、YCSB、RocksDB 都是系统论文常用的数据库/存储 workload；RocksDB 官方 wiki 说明 `db_bench` 是 RocksDB 的主要 benchmark 工具|
|**实验问题**|前台事务/查询的 p99 latency 是否能被 CPU utilization 解释？|
|**实验做法**|1. 运行 PostgreSQL + BenchBase TPC-C 或 Redis/RocksDB + YCSB；2. 逐步加入后台 TPC-H 查询、RocksDB compaction 或 NPB；3. 用 cgroup quota 控制总 CPU utilization 相同；4. 对比默认调度、static cgroup、foreground boosted、background throttled|
|**采集指标**|tpmC/ops/s、p95/p99 transaction latency、CPU utilization、runqueue latency、lock wait、context switches、LLC miss、IO wait|
|**分析/检验**|1. 在相同 CPU utilization 桶内比较 p99 transaction latency；2. 用 partial correlation 控制 CPU utilization，检验 runqueue latency / lock wait / LLC miss 对 p99 的解释力；3. 对照 UFS 的做法，把前台 CPU-bursty 与后台 CPU-bound 分开分析|
|**预期 Takeaway**|数据库类 workload 中，CPU utilization 不能解释事务尾延迟；调度路径和应用阶段特征更关键|
|**推荐图表**|Fig. 3(a) p99 latency by background type；Fig. 3(b) CPU util vs p99 scatter；Fig. 3(c) runqueue/lock wait breakdown|

UFS 的实验就采用了 BenchBase TPC-C 作为 CPU-bursty workload，并用 TPC-H/UDF 模拟 CPU-bound 后台任务；它还专门比较了 SOLO、MIN:MAX、50:50 三类 workload mix，证明同一组 CPU 资源下不同 workload 组合会暴露不同调度问题。  RocksDB 官方文档说明 `db_bench` 是用于 benchmark RocksDB 性能的主要工具。 

**优先级：中高。**  
如果你们后续想强调“应用场景识别”而不是纯 OS benchmark，这个方案很有说服力。

---

# **Motivation 2：不同场景需要不同调度策略**

## **要证明的结论**

**不存在一个静态调度策略在所有计算场景下都最优。不同 workload 的最优策略不同，因此需要在线识别场景并切换策略参数。**

这个 motivation 可以借鉴 UFS 的 failure-mode 组织方式：UFS 不是笼统地说 Linux scheduler 不好，而是区分 EEVDF、IDLE、FIFO、RR 在不同 workload mix 下的失败模式。  ALPS 也显示，CFS、FIFO、RR、SFS、SRPT 在不同函数分布和不同分位数下表现不同，不能简单认为某一个策略全局最优。 

---

## **方案 2.1：公开 benchmark 分类矩阵实验**

|**项目**|**设计**|
|---|---|
|**测试对象**|CPU-bound：NPB `EP/BT/SP`、PARSEC `blackscholes/swaptions`；memory/cache-sensitive：NPB `CG/MG/FT`、PARSEC `canneal/dedup`、SPLASH-3 `ocean/raytrace`；latency-sensitive：TailBench `xapian/masstree/silo`；real-time：cyclictest|
|**为什么合适**|覆盖吞吐型、内存敏感型、低延迟型、周期实时型场景|
|**实验问题**|不同 benchmark 类别下，哪个调度策略最好？winner 是否随场景变化？|
|**实验做法**|1. 每个 workload 在相同硬件、相同 core set 下运行；2. 比较 Linux default EEVDF/CFS、SCHED_FIFO、SCHED_RR、SCHED_DEADLINE、static cgroup、sched_ext-low-latency、sched_ext-throughput、sched_ext-affinity；3. 每个策略运行 5–10 次；4. 对每个 workload 定义主目标：completion time、p99 latency、deadline miss、throughput|
|**采集指标**|normalized performance score、runtime、throughput、p99 latency、deadline miss、context switch、migration、LLC miss|
|**分析/检验**|1. 画 **workload × scheduler heatmap**；2. 统计每个策略成为 best/near-best 的次数；3. 使用 Friedman test / Nemenyi post-hoc 检验多个策略 rank 是否有显著差异；4. 计算 worst-case regret：某静态策略相对每场景最优策略的性能损失|
|**预期 Takeaway**|每类 workload 的最佳策略不同；静态策略在部分场景表现好，但在其他场景有明显 regret|
|**推荐图表**|Fig. 4 workload-scheduler heatmap；Table 1 failure mode summary；Fig. 5 best-policy distribution|

NPB 是 NASA 定义的并行 benchmark，用于评估并行超级计算机性能；CloudSuite 和 TailBench 适合作为在线服务与尾延迟 workload，SPLASH-3 是基于 SPLASH-2 的并行 benchmark suite。 

**优先级：高。**  
这是最直接证明 “static policy mismatch” 的实验。

---

## **方案 2.2：前台低延迟服务 + 后台吞吐任务的 Pareto 实验**

|**项目**|**设计**|
|---|---|
|**测试对象**|前台：TailBench `xapian/masstree` 或 DeathStarBench `socialNetwork/hotelReservation`；后台：NPB、PARSEC、CloudSuite data analytics、RocksDB compaction|
|**为什么合适**|直接构造“低延迟目标 vs 后台吞吐目标”的冲突|
|**实验问题**|同一静态策略是否能同时在低延迟和高吞吐两个目标上接近最优？|
|**实验做法**|1. 设定前台服务目标，例如 p99 latency < X ms；2. 后台任务持续运行；3. 分别运行默认调度、低延迟策略、吞吐策略、static cgroup、affinity 策略、oracle best-of-N；4. 通过改变前台 QPS 和后台线程数形成不同压力区间|
|**采集指标**|前台 p95/p99 latency、SLO violation rate、后台 job completion time、background throughput、CPU share、context switch、migration|
|**分析/检验**|1. 画 **foreground p99 latency vs background throughput Pareto frontier**；2. 标出每种静态策略在不同 load 下的位置；3. 计算 weighted objective，例如 `score = latency_violation_penalty + normalized_completion_time`；4. 检验某一静态策略是否在所有 load 下 dominated|
|**预期 Takeaway**|低延迟策略保护前台但牺牲后台；吞吐策略提升后台但伤害前台；需要按场景动态选择策略参数|
|**推荐图表**|Fig. 6 Pareto frontier；Fig. 7 SLO violation vs background throughput；Table 2 policy tradeoff|

DeathStarBench 是开源微服务 benchmark，包含 cloud microservices end-to-end services；CloudSuite 是面向 first-party cloud services 的 benchmark suite，并包含 online services 与 analytics workloads。 

**优先级：高。**  
这组图很适合放在 motivation 里，直观展示静态策略不可兼顾多目标。

---

## **方案 2.3：数据库混合场景的 SOLO / MIN:MAX / 50:50 实验**

|**项目**|**设计**|
|---|---|
|**测试对象**|PostgreSQL + BenchBase TPC-C；后台 TPC-H query loop / PostgreSQL UDF；可扩展到 MADlib logistic regression 或 RocksDB/YCSB|
|**为什么合适**|借鉴 UFS 的 workload mix，但目标不是复现 UFS，而是证明“策略适用场景不同”|
|**实验问题**|高低优先级混合、同优先级混合、单独运行这三种场景下，最优策略是否不同？|
|**实验做法**|1. SOLO：只运行 TPC-C 或只运行 TPC-H；2. MIN:MAX：TPC-C 高优先级，TPC-H/UDF 低优先级；3. 50:50：TPC-C 和 TPC-H 都高优先级；4. 比较 EEVDF/CFS、FIFO、RR、IDLE、static cgroup、sched_ext priority、sched_ext fair；5. 记录 transaction latency 和 background throughput|
|**采集指标**|TPC-C throughput、TPC-C mean/p95/p99 latency、TPC-H iteration throughput、CPU distribution、runqueue latency、CPU share|
|**分析/检验**|1. 每个场景画 throughput/latency bar；2. 做 failure mode table：哪个策略在哪个场景失败，失败原因是 placement、head-of-line blocking、缺少 virtual runtime、过度公平等；3. 计算每个策略 across scenarios 的最大 regret|
|**预期 Takeaway**|强优先级策略适合 MIN:MAX，但可能在 50:50 中破坏公平；公平策略适合 50:50，但可能不能保护前台；动态场景识别有必要|
|**推荐图表**|Fig. 8 SOLO/MIN:MAX/50:50 throughput；Table 3 scheduler failure modes；Fig. 9 latency percentile|

UFS 正是通过 SOLO、MIN:MAX、50:50 三类 workload mix 分析 Linux 调度器在 mixed database workloads 下的不同失败模式，这种“分场景找失败模式”的组织方式值得借鉴。 

**优先级：中高。**  
如果实验平台能方便部署 PostgreSQL 和 BenchBase，这个方案非常有系统论文味道。

---

# **Motivation 3：计算特征可以预测“该用哪类策略”**

## **要证明的结论**

**微架构特征、调度路径特征和应用目标特征联合起来，可以形成可区分的场景簇，并能预测某个时间窗口内更适合的调度策略。**

这个 motivation 对应你们实验一的核心：不是只证明某个策略更快，而是证明 **feature → scenario → policy** 的映射是存在的、可解释的、可学习的。你们大纲中也明确把 Feature Data Plane 设计为采集 PMU、sched tracepoint、cgroup stats 和 application metrics，并通过滑动窗口转换成 feature vector。 

---

## **方案 3.1：多 benchmark 特征聚类与 best-policy 标注实验**

|**项目**|**设计**|
|---|---|
|**测试对象**|NPB、PARSEC/SPLASH-3、TailBench、cyclictest、BenchBase/YCSB/RocksDB|
|**为什么合适**|覆盖 CPU-bound、memory/cache-sensitive、latency-sensitive、periodic、database/storage 场景|
|**实验问题**|不同 workload 的特征窗口是否能自然聚类？这些特征是否能预测 best policy？|
|**实验做法**|1. 每个 workload 在每个 baseline policy 下运行；2. 每 100ms/500ms/1s 采集一个 feature vector；3. feature 包含 IPC、LLC miss、branch miss、runqueue latency、wakeup latency、context switch rate、migration rate、CPU utilization、p99 latency、throughput；4. 对每个窗口离线标注 best policy：在该 workload/phase 中 normalized score 最好的策略|
|**分析/检验**|1. PCA/UMAP/t-SNE 可视化，看 CPU-bound、memory-sensitive、latency-sensitive 是否分开；2. 训练 decision tree / random forest / multinomial logistic regression 预测 best policy；3. 用 leave-one-workload-out 验证泛化；4. 报告 top-1/top-2 policy accuracy、macro-F1、regret vs oracle；5. 输出 decision tree 规则，检查是否符合直觉|
|**预期 Takeaway**|计算特征不仅能区分 workload 类型，还能预测策略选择；可解释模型足以支撑第一版系统|
|**推荐图表**|Fig. 10 UMAP feature clusters；Fig. 11 decision tree；Fig. 12 regret vs oracle；Table 4 feature importance|

**优先级：最高。**  
这是 Motivation 3 的主实验，也是你们论文区别于“又一个调度器”的关键证据。

---

## **方案 3.2：Trace-driven FaaS / 函数工作负载特征预测实验**

|**项目**|**设计**|
|---|---|
|**测试对象**|Azure Functions trace、Huawei Cloud Functions trace；运行平台可用 OpenLambda、OpenFaaS、SeBS / ServerlessBench；函数应用可选 ALPS 使用过的 Fibonacci、BFS、MST、PageRank、Feature Extraction、Float Operation|
|**为什么合适**|公开云函数 trace 具有 burstiness、duration variance、hot/cold skew，适合生成 phase-changing workload|
|**实验问题**|在真实 trace replay 下，过去窗口特征是否能预测下一窗口适合的策略？|
|**实验做法**|1. 按 trace replay 函数请求；2. 将函数分为 short、medium、long、variable 四类；3. 对每个时间窗口运行不同策略或先离线生成策略标签；4. 收集 feature vector；5. 训练轻量模型预测下一窗口 best policy 或场景标签|
|**采集指标**|per-window IPC、LLC miss、runqueue latency、arrival rate、burstiness、function duration distribution、context switch、tail latency|
|**分析/检验**|1. 用过去 k 个窗口预测下一窗口 best policy；2. 比较三种输入：只用 CPU utilization、只用应用统计、使用完整特征；3. 报告 prediction accuracy、policy regret、SLO violation reduction potential；4. 对 burst window 单独分析，看特征是否提前预警策略变化|
|**预期 Takeaway**|在 trace-driven 动态负载中，多源特征比单一 CPU utilization 更能预测策略选择|
|**推荐图表**|Fig. 13 per-window feature timeline；Fig. 14 prediction accuracy by feature group；Fig. 15 regret CDF|

ALPS 的实验使用 Huawei 和 Azure FaaS traces，并将函数按执行时间分为四组；它还采用 sliding window 收集过去函数 trace，并在用户态训练策略，再通过 eBPF map 传给内核态执行。这个流程可作为我们“窗口特征 → 策略标签”的参考，但不必继承其 SRPT 问题。 

**优先级：中高。**  
如果你们要强调 workload phase change 和 trace realism，这个方案很合适。

---

## **方案 3.3：数据库/微服务真实应用的阶段识别实验**

|**项目**|**设计**|
|---|---|
|**测试对象**|DeathStarBench `socialNetwork` / `hotelReservation`，CloudSuite web-serving/data-serving，BenchBase TPC-C/TPC-H，YCSB + Redis/RocksDB|
|**为什么合适**|这些应用有明显阶段：front-end request burst、cache hit/miss、DB contention、background analytics、compaction、GC 等|
|**实验问题**|应用阶段变化是否能从系统级特征中识别出来？识别出的阶段是否对应不同策略？|
|**实验做法**|1. 对应用施加有标注的阶段：warm-up、read-heavy、write-heavy、burst、background analytics、cache pressure；2. 每个阶段运行 60–120 秒；3. 每个阶段下比较 static cgroup、low-latency、throughput、affinity、default；4. 收集窗口特征和 best-policy label|
|**采集指标**|request latency、throughput、cache hit ratio、DB lock wait、context switches、runqueue latency、IPC、LLC miss、IO wait|
|**分析/检验**|1. 用监督分类预测阶段标签；2. 用无监督聚类看系统特征能否复原人工阶段；3. 分析“阶段标签”和“best policy label”是否一致；4. 计算 mutual information 或 Cramér’s V，检验 feature cluster 与 best policy 的关联强度|
|**预期 Takeaway**|应用场景不是只能靠应用显式标注，系统级特征本身能反映场景变化；这支持 eBPF/PMU 特征采集的价值|
|**推荐图表**|Fig. 16 phase clustering；Fig. 17 phase-policy transition matrix；Table 5 feature-to-policy rules|

YCSB 是 Yahoo! Cloud Serving Benchmark 的开源实现；BenchBase 是多 DBMS SQL benchmarking framework；DeathStarBench 和 CloudSuite 都提供云服务类应用，适合构造真实应用阶段。 

**优先级：中。**  
该方案工程量略大，但最能体现“应用场景识别”的泛化性。

---

# **Motivation 4：在线策略切换需要安全机制**

## **要证明的结论**

**动态策略如果没有 safety guard，可能因为特征噪声、短期波动、误判、负载突变而产生震荡、过度迁移、上下文切换增加或短期性能回退。因此需要 cooldown、hysteresis、health check 和 fallback。**

这部分可借鉴 ALPS 的 fine-tuning 思路：ALPS 明确考虑了函数执行时间不确定性和 workload shift，并通过 conservative adjustment 缓解 long-function starvation 与 burst 场景下的尾延迟问题。  但我们的目标不是复现 ALPS，而是证明**任何在线策略切换系统都需要安全约束**。

---

## **方案 4.1：公开 benchmark 组合的 phase-changing workload**

|**项目**|**设计**|
|---|---|
|**测试对象**|前台：TailBench `xapian/masstree` 或 DeathStarBench；后台：NPB/PARSEC；实时：cyclictest；内存压力：STREAM/stress-ng cache/vm|
|**为什么合适**|用公开 benchmark 组合构造明确阶段，每个阶段有不同最优策略|
|**实验问题**|策略在线切换时，如果没有 cooldown/hysteresis/fallback，是否出现震荡和性能尖峰？|
|**实验做法**|构造 5 个阶段：1. 前台轻负载；2. 前台 burst；3. 后台 CPU-bound 增强；4. memory/cache pressure 加入；5. 周期实时任务加入。每个阶段 60–120 秒。比较：default、static best-for-phase-1、adaptive-no-guard、adaptive-with-guard、oracle best-of-phase|
|**采集指标**|policy switch count、switch interval、p99 latency、deadline miss、background throughput、migration rate、context switch rate、rollback count、recovery time|
|**分析/检验**|1. 画 timeline：phase、scenario label、policy、p99 latency、migration rate；2. 比较 no-guard 与 with-guard 的 policy switch count；3. 统计 latency spike 次数和最大 spike；4. 定义 oscillation：在 N 个窗口内 A↔B 来回切换超过阈值；5. 检验 guard 是否显著降低 oscillation 和 tail spike|
|**预期 Takeaway**|动态策略本身不够；必须加入 safety guard 才能稳定部署|
|**推荐图表**|Fig. 18 adaptation timeline；Fig. 19 no-guard vs guard switch count；Fig. 20 p99 spike CDF|

**优先级：最高。**  
这是 Motivation 4 最清晰、最可控的实验。

---

## **方案 4.2：真实 trace replay 下的 burst / non-stationary workload**

|**项目**|**设计**|
|---|---|
|**测试对象**|Azure Functions trace、Huawei Cloud Functions trace；或 DeathStarBench/YCSB 的真实到达率变化 replay|
|**为什么合适**|trace replay 能自然产生 burst、phase shift、hotspot 变化，比手工 phase 更接近实际|
|**实验问题**|在真实到达模式下，在线策略是否会因为短期波动频繁误切换？guard 是否能降低误切换代价？|
|**实验做法**|1. 用 trace replay 生成请求；2. 将 trace 分为 normal、burst、hot function、long-tail-heavy 几类窗口；3. 自适应策略每 100ms/500ms/1s 更新；4. no-guard 版本每次预测变化即切换；with-guard 版本要求连续 K 个窗口满足条件才切换，并设置 cooldown|
|**采集指标**|per-window predicted scene、actual best policy、policy switches、latency/SLO violation、regret、false switch rate|
|**分析/检验**|1. 计算 false switch rate：切换后性能没有改善或更差的比例；2. 计算 regret：当前策略 vs 离线 oracle best policy；3. 比较 no-guard 和 with-guard 在 burst windows 中的 p99 latency 和 SLO violation；4. 分析 window size / cooldown 对性能和反应速度的 tradeoff|
|**预期 Takeaway**|trace-driven 动态负载下，过度敏感的在线策略会被短期噪声误导；guard 能降低 regret，但过强 guard 会降低响应速度|
|**推荐图表**|Fig. 21 regret over time；Fig. 22 false switch rate；Fig. 23 cooldown sensitivity|

ALPS 的 evaluation 中专门做了 sliding window size、learning method 和参数敏感性分析，并指出较短窗口更能反映近期行为但也可能更敏感；这类 sensitivity 设计可直接借鉴到我们的 guard 参数实验。 

**优先级：中高。**  
适合放在主实验之后，证明系统面对真实 trace 时不是只靠人工阶段调好。

---

## **方案 4.3：故障注入 / 错误特征 / 错误策略的 safety microbenchmark**

|**项目**|**设计**|
|---|---|
|**测试对象**|前台：TailBench、cyclictest、BenchBase TPC-C；后台：stress-ng、NPB、TPC-H loop|
|**为什么合适**|公开 benchmark 本身不提供“错误策略/错误 hint”，所以 fault injection 需要人工构造；但被测应用仍然是公开 benchmark|
|**实验问题**|当特征采集出错、场景误判、策略 map 更新延迟或策略参数异常时，系统是否能及时回退？|
|**实验做法**|1. 注入 feature noise：随机扰动 IPC/LLC miss/runqueue latency；2. 注入 missing feature：关闭 PMU 或丢弃部分 eBPF event；3. 注入 wrong label：把 latency-sensitive 场景误判为 throughput 场景；4. 注入 policy delay：延迟写 eBPF map；5. 注入 aggressive policy：过短/过长 time quantum、过高 migration、过度抢占。比较 no-fallback 与 fallback|
|**采集指标**|fallback trigger latency、fallback completion latency、maximum p99 regression、deadline miss recovery time、policy rollback count、event loss rate、system crash/hang|
|**分析/检验**|1. 定义安全目标：任何策略错误导致的性能回退不得超过 X 秒或 Y%；2. 比较 no-fallback 与 fallback 的最大损失；3. 检查 fallback 是否把系统恢复到 default/static-safe baseline；4. 统计 injected fault 下的成功恢复率；5. 使用 survival analysis 或 failure rate table 展示系统鲁棒性|
|**预期 Takeaway**|在线可编程调度存在误判风险；safety guard 和 fallback 不是工程细节，而是设计必要条件|
|**推荐图表**|Fig. 24 fault injection timeline；Table 6 recovery success rate；Fig. 25 max regression bar|

UFS 也使用了一个机制性 micro-experiment 来证明 application hinting 能处理 priority inversion，这类“正常 workload 不一定触发，但机制必须证明”的实验组织方式适合我们借鉴；区别是我们这里测试的是策略安全与回退机制。 

**优先级：中。**  
这是论文后半部分的 safety evaluation，但在 motivation 里可以先放一个小规模 teaser。

---

# **推荐优先执行组合**

如果时间有限，不建议 12 个方案全部做。建议按“最小但完整”的方式选择下面 4 个主方案：

|**Motivation**|**推荐主方案**|**原因**|
|---|---|---|
|M1 Proxy metric mismatch|**方案 1.1 TailBench + NPB/PARSEC/stress-ng**|最直接证明 CPU utilization 不解释 p99 latency|
|M2 Static policy mismatch|**方案 2.1 多 benchmark × 多 scheduler heatmap**|最直接证明不存在全局最优静态策略|
|M3 Feature-policy correlation|**方案 3.1 多 benchmark 特征聚类与 best-policy 标注**|最核心，支撑“计算特征 → 策略”的主张|
|M4 Safety requirement|**方案 4.1 phase-changing workload + no-guard/with-guard**|最直接证明安全窗和回退机制必要|

这四个主方案可以形成一条完整 motivation 证据链：

```text
M1: CPU utilization 不够
    ↓
M2: 静态策略没有全局最优
    ↓
M3: 多源计算特征可以预测场景和策略
    ↓
M4: 在线切换策略需要 safety guard
```

---

# **建议统一实验配置**

## **Baselines**

|**类型**|**Baseline**|
|---|---|
|Linux 默认|CFS 或 EEVDF，取决于 kernel 版本|
|实时类|SCHED_FIFO、SCHED_RR|
|周期任务|SCHED_DEADLINE|
|资源控制|static cgroup weight / cpu.max / cpuset|
|可编程策略|sched_ext-low-latency、sched_ext-throughput、sched_ext-affinity|
|上界|offline oracle best-of-N|
|消融|adaptive-no-guard、adaptive-no-PMU、adaptive-no-sched-feature|

Linux 官方文档说明 sched_ext 的行为可由 BPF programs 定义，并且可以动态打开和关闭；这支持我们把不同调度策略参数作为可编程执行器来实验。 

## **统一采集指标**

|**层次**|**指标**|
|---|---|
|应用目标|p50/p95/p99 latency、deadline miss、jitter、throughput、completion time|
|调度路径|runqueue latency、wakeup latency、context switch rate、migration rate|
|微架构|IPC、LLC miss rate、branch miss rate、cycles、instructions|
|系统资源|CPU utilization、per-core utilization、cgroup CPU share、CPU pressure|
|策略行为|policy switch count、fallback count、oscillation count、recovery time|

`perf sched latency` 可以报告每个任务的调度延迟和调度属性，适合作为 eBPF 采集结果的交叉验证工具。 

## **统一分析方法**

|**分析目标**|**方法**|
|---|---|
|证明 proxy metric 不足|correlation、partial correlation、同 CPU utilization 桶内 p99 差异|
|证明静态策略无全局最优|normalized heatmap、rank analysis、regret vs oracle|
|证明特征可预测策略|PCA/UMAP、decision tree、feature importance、top-k policy accuracy|
|证明 safety 必要|no-guard vs with-guard、oscillation count、max regression、recovery time|
|证明结果可靠|5–10 次重复、bootstrap 95% CI、随机实验顺序、固定 CPU frequency|

---

# **最终 motivation 图表建议**

|**图编号**|**对应 motivation**|**图表内容**|
|---|---|---|
|Fig. 1|M1|CPU utilization 相近但 p99 latency / deadline miss 显著不同|
|Fig. 2|M1|runqueue latency、context switch、LLC miss 对 p99 的解释力|
|Fig. 3|M2|workload × scheduler normalized performance heatmap|
|Fig. 4|M2|不同策略的 Pareto frontier：foreground latency vs background throughput|
|Fig. 5|M3|feature space clustering：不同场景形成可区分簇|
|Fig. 6|M3|decision tree / rule list：特征到策略的可解释映射|
|Fig. 7|M4|phase-changing timeline：policy switch、p99 latency、migration rate|
|Fig. 8|M4|no-guard vs with-guard：oscillation、latency spike、fallback recovery|

这套 motivation 实验不会变成 eMFS/ALPS/UFS 的延续问题，而是服务于我们自己的问题：**为什么需要一个基于 eBPF/PMU/sched_ext 的计算特征驱动、自适应、可回退的 CPU 调度框架。**