# **可以，而且这样拆分比合成一篇更合理**

这两部分对应的是两个正交问题：

- **工作一：语义与判定可信性**  
    监控器是否在监控正确的对象、正确的约束，并且它给出的判定是否可信？
- **工作二：时间与资源可信性**  
    即使监控器逻辑正确，在 CPU/GPU、控制和感知任务竞争时，它能否在故障不可恢复之前完成执行？

可以将其概括为：

**工作一回答“监控什么、判得对不对”；工作二回答“何时监控、来不来得及”。**

CaM 当前直接让 GPT-4o 根据约束元素生成监控代码，代码接收三维位置、执行算术计算，根据预定义容差返回布尔值和原因。它证明了“代码监控”范式的有效性，但没有解决生成代码的语义验证、感知不确定性传播以及资源竞争下的时间保证。 

这两个缺口足以支撑两篇工作，但需要严格控制论文边界，避免被评价为一项工作的拆分发表。

---

# **一、两篇工作的总体定位**

|**维度**|**工作一：可信监控编译**|**工作二：风险驱动运行时**|
|---|---|---|
|核心问题|生成的监控器是否正确、可信|正确的监控器能否及时执行|
|主要错误来源|对象绑定、部件选择、坐标系、单位、阈值、时序语义、感知噪声|CPU/GPU 竞争、任务突发、阻塞、数据过期、监控频率不足|
|核心抽象|Typed Monitor IR / Monitor Contract|Semantic Deadline / Monitor Task|
|主要方法|类型系统、静态检查、测试生成、反例修复、不确定性传播|风险估计、语义截止时间、混合关键级调度、质量自适应|
|输出|经过检查且带置信语义的监控程序|带时间保证或安全降级的执行结果|
|主要指标|语义正确率、误报漏报、校准覆盖率、修复成功率|deadline miss、检测延迟、预警提前量、数据陈旧度|
|应固定的变量|资源充足、固定监控频率|使用人工或已验证的正确监控器|
|主要投稿方向|CoRL、ICRA、IROS、RA-L、CVPR/ICCV|RTSS、EMSOFT、EuroSys、ASPLOS；次选 ICRA/CoRL|

两篇论文不能分别叫“系统前端”和“系统后端”。更合适的表述是两种独立保证：

\text{End-to-end monitor assurance} = \underbrace{\text{semantic assurance}}_{\text{工作一}} + \underbrace{\text{temporal assurance}}_{\text{工作二}}

---

# **二、工作一：可信且不确定性感知的监控编译**

## **2.1 建议题目**

更稳妥的命名是：

**AssuredCaM: Contract-Guided and Uncertainty-Calibrated Compilation of Robot Monitors**

或者：

**Monitor the Monitor: Assured Visual Program Synthesis for Robotic Runtime Monitoring**

暂时不建议直接使用“Fully Verified”一类过强表述。开放环境中无法形式化证明自然语言意图被完整理解，能够严格保证的是：

1. 生成代码符合结构化监控规范；
2. 对象、单位和坐标系使用一致；
3. 监控程序实现与中间表示一致；
4. 给定感知误差模型时，判定具有统计校准性质。

不能直接保证：

VLM 对用户意图和真实世界语义的理解绝对正确。

因此，“assured”“contract-checked”通常比“fully verified”更严谨。

---

## **2.2 核心研究问题**

CaM 当前隐含了一个不可靠链条：

```text
自然语言约束
  → VLM 选择对象和部件
  → VLM 决定计算关系和阈值
  → 直接生成 Python
  → 根据不确定的三维点返回 True / False
```

这里至少有五类不同错误：

|**错误层**|**示例**|
|---|---|
|约束理解错误|“保持杯子竖直”被理解为“杯子位于桌面上方”|
|实体接地错误|使用锅柄而不是锅面计算水平度|
|几何语义错误|在相机坐标系中与世界坐标系 z 轴比较|
|程序实现错误|角度和弧度混用，向量方向取反|
|感知不确定性错误|遮挡或深度噪声下仍给出确定的 True/False|

工作一应该解决的是：

如何把 VLM 生成的自由代码，转变成由受限中间表示、可信编译器、静态检查、动态测试和不确定性语义共同约束的监控程序？

---

## **2.3 核心架构**

```text
任务指令 + 场景观测
          │
          ▼
  Candidate Constraint IR
       VLM 生成候选规范
          │
          ▼
  Static Contract Checker
  ├─ 类型检查
  ├─ 单位检查
  ├─ 坐标系检查
  ├─ 对象/部件一致性
  ├─ 传感器可观测性
  └─ 时序约束可监控性
          │
          ▼
 Counterexample Validator
  ├─ 边界状态生成
  ├─ 变形测试
  ├─ 场景扰动测试
  └─ 反例驱动修复
          │
          ▼
 Uncertainty-aware Compiler
  ├─ 感知误差传播
  ├─ 鲁棒性区间
  ├─ 三值监控语义
  └─ 可信代码生成
          │
          ▼
  SAT / VIOLATED / UNKNOWN
```

---

## **2.4 第一项创新：类型化 Monitor IR**

不再让 VLM 直接生成任意 Python，而是生成受限的结构化规范：

```yaml
monitor:
  id: keep_pan_level
  scope: during
  phase: transfer_pan

  entities:
    subject:
      instance: pan
      part: inner_surface
      type: planar_surface

  reference:
    frame: world
    axis: z

  predicate:
    operator: angle_less_than
    lhs: normal(subject)
    rhs: reference.axis
    threshold:
      value: 8
      unit: degree

  sensors:
    - front_rgbd
    - top_rgbd

  severity: critical
  fallback: stop_and_stabilize
```

IR 至少需要包含：

- 实体类型和部件类型；
- 点、线、面、姿态、接触等几何或物理类型；
- 坐标系和变换关系；
- 物理单位；
- during、until、upon-completion 等时序作用域；
- 所需传感器；
- 阈值来源；
- 严重等级；
- 故障响应；
- 输出语义。

随后由可信编译器生成 C++、NumPy 或 ROS 2 monitor。

### **能够静态排除的错误**

```text
angle(distance(A, B), world.z)       # 类型错误
distance(A, B) < 10 degree           # 单位错误
normal(point_element)                # 元素类型错误
compare(camera_frame, world_axis)    # 缺少坐标变换
monitor force without force sensor   # 不可观测
```

这部分与单纯“自然语言转 LTL/STL”不同。RoboGuard 已经将环境化的安全规则转化为时序逻辑约束，SafeManip 也已经使用 LTLf 属性评价操作轨迹；因此不能把“引入时序逻辑”本身作为主创新。新的重点必须是**开放场景视觉接地、几何类型、单位坐标系检查、自动编译和不确定性运行时判定**。 

---

## **2.5 第二项创新：反例驱动的监控器验证**

静态检查不能判断“锅面是否选成了锅柄”，因此还需要动态验证。

可以自动生成以下测试：

### **边界测试**

- 倾角分别为 7.9^\circ、8.0^\circ、8.1^\circ；
- 距离位于阈值上下；
- 对象恰好进入或离开目标区域；
- 抓取状态处于接触边界。

### **变形测试**

对监控程序定义必须保持不变的 metamorphic relations：

- 场景整体平移不应改变相对距离；
- 物体和参考系同时旋转不应改变相对姿态；
- 米转换为厘米后判定不应变化；
- 移动无关物体不应改变目标约束；
- 调换两个目标对象后，监控结果应随绑定关系变化；
- 相机视角变化不应改变世界坐标约束。

### **故障注入**

有计划地注入：

- 错误对象 ID；
- 错误部件；
- 坐标系缺失；
- 单位混用；
- 错误阈值；
- 点、线、面类型退化；
- 遮挡和跟踪漂移；
- 传感器掉线。

发现反例后，不重新生成整段代码，而是将：

```text
原规范 + 失败测试 + 最小反例
```

反馈给 VLM，仅修复对应的 IR 节点。这形成类似 counterexample-guided synthesis 的闭环。

---

## **2.6 第三项创新：不确定性感知的三值语义**

CaM 当前最终返回布尔值，但输入来自分割、深度、点云和跟踪，这些都存在误差。 

令约束鲁棒性为：

\rho_i(x_t)

其中：

- \rho_i>0：满足；
- \rho_i<0：违反；
- |\rho_i| 表示距离约束边界的裕量。

由于真实状态不是单点，而是一个置信集合 X_t，可以定义：

\begin{aligned} \text{SAT}, &\quad \inf_{x\in X_t}\rho_i(x)>0\\ \text{VIOLATED}, &\quad \sup_{x\in X_t}\rho_i(x)<0\\ \text{UNKNOWN}, &\quad \text{otherwise} \end{aligned}

例如：

```text
约束：pan tilt < 8°
估计倾角：7.2°
置信区间：[5.9°, 9.3°]
结论：UNKNOWN
原因：侧视相机遮挡，法向量估计区间跨越安全边界
响应：减速并刷新分割
```

SAFE 已经使用 VLA 内部特征输出整体失败分数，并采用 conformal prediction 平衡检测准确率和检测时间；因此你不能只做“给故障检测加 conformal calibration”。差异应当是：

- SAFE：轨迹或策略级失败概率；
- 工作一：**具体约束谓词级的置信区间和可解释判定**；
- SAFE：回答“机器人是否可能失败”；
- 工作一：回答“哪个对象的哪个约束为什么处于不确定或违反状态”。 

---

## **2.7 工作一的实验设计**

### **实验中必须固定资源条件**

为避免和工作二混淆，工作一应当：

- 使用资源充足的平台；
- 固定监控频率；
- 不注入 CPU/GPU 竞争；
- 不提出调度算法；
- 只报告监控器自身开销，不把性能优化作为贡献。

### **研究问题**

| **RQ** | **问题**                         |
| ------ | ------------------------------ |
| RQ1    | CaM 直接生成代码时，存在多少可执行但语义错误的监控程序？ |
| RQ2    | Typed IR 和静态检查能排除多少错误？         |
| RQ3    | 反例测试能否发现静态检查无法发现的接地和阈值错误？      |
| RQ4    | 不确定性语义能否降低遮挡、深度噪声和跟踪漂移造成的误报漏报？ |
| RQ5    | 验证机制是否影响开放任务泛化和运行效率？           |

### **主要基线**

- 原始 CaM 直接生成 Python；
- JSON/DSL 约束生成，但没有验证；
- LLM self-refinement；
- 双 VLM 交叉检查；
- 人工编写的几何或 STL/LTL monitor；
- 只有静态检查；
- 只有不确定性估计；
- 完整方法。

### **指标**

- 代码可执行率；
- 约束语义正确率；
- 实体和部件绑定准确率；
- 坐标系、单位、时序错误检出率；
- 故障事件 Precision、Recall、F1；
- 每小时误报数；
- 主动预警提前量；
- Expected Calibration Error；
- conformal coverage；
- UNKNOWN 比例；
- 反例发现率；
- 自动修复成功率；
- 生成和验证时间；
- 运行时监控开销。

### **建议规模**

至少需要：

- 50–100 类约束实例；
- 5–8 类系统化 monitor fault；
- 3 个模拟环境；
- 1 个真实机器人平台；
- 几何、包含、抓取、铰接、接触等多类约束；
- 未见对象、未见场景和未见约束组合。

---

# **三、工作二：资源竞争下的风险驱动实时监控**

## **3.1 建议题目**

不建议继续使用 CaM 作为标题主体，否则容易被理解成 CaM 的调度优化。

更合适的是：

**Semantic Deadlines: Risk-Adaptive Scheduling for Robot Runtime Monitors**

或者：

**RiskRT: A Mixed-Criticality Runtime for Semantic Robot Monitors**

它应当被定义为一个支持多种监控前端的通用运行时，而不是只支持工作一。

---

## **3.2 核心研究问题**

正确的 monitor 仍可能因为资源竞争而失效：

```text
监控器逻辑正确
      │
      ├─ 相机帧延迟
      ├─ 跟踪任务等待 GPU
      ├─ VLA 推理占满 GPU
      ├─ ROS 2 callback 排队
      ├─ CPU 被规划器和日志任务抢占
      └─ monitor 判定在故障发生后才完成
```

因此，工作二的核心不是普通调度，而是：

如何根据约束的风险演化、剩余安全裕量和最后可恢复时刻，生成 monitor 的 period、deadline、criticality 和 fidelity，并在资源过载时优先保障真正关键的检测？

RED 已经研究了动态机器人环境中的多任务 DNN DAG 调度、子截止时间分配、ROS 2 集成和过载下的低关键级任务丢弃。因此，单纯做“机器人推理任务的 EDF 或自适应调度”创新不足。 

新的差异必须是：

**RED 的 deadline 主要来自应用 DAG 和端到端预算；你的 deadline 来自物理约束的安全裕量、风险变化率和最后可恢复窗口。**

---

## **3.3 核心抽象：Monitor Task**

把每一个监控器建模为：

M_i = (C_i(q),T_i,D_i,L_i,Q_i,A_i,\rho_i)

其中：

|**字段**|**含义**|
|---|---|
|C_i(q)|在质量等级 q 下的执行时间或 WCET|
|T_i|监控周期|
|D_i|语义截止时间|
|L_i|criticality / severity|
|Q_i|可选择的监控质量等级|
|A_i|最大允许数据陈旧度|
|\rho_i|当前约束鲁棒性|

质量等级可以对应：

- 单视角或多视角；
- 稀疏跟踪点或密集跟踪点；
- 低分辨率或高分辨率；
- 使用已有跟踪或重新分割；
- 几何快速检查或调用 VLM；
- CPU 轻量模型或 GPU 完整模型。

---

## **3.4 第一项创新：从约束风险推导语义截止时间**

假设约束安全边界为：

\rho_i(t)=0

当前约束仍满足，即：

\rho_i(t)>0

若约束裕量正在下降，可以粗略估计 time-to-violation：

\hat{\tau}^{vio}_i = \frac{\rho_i(t)} {\max(-\dot{\rho}_i(t),\epsilon)}

但真正需要的不是故障发生时间，而是最后可恢复时间：

\hat{\tau}^{lr}_i = \hat{\tau}^{vio}_i - C_i^{decision} - C_i^{reaction} - C_i^{actuation} - \Delta_i

其中：

- C_i^{decision}：判定和恢复决策时间；
- C_i^{reaction}：系统响应时间；
- C_i^{actuation}：机械动作生效时间；
- \Delta_i：安全余量。

相应地，监控任务的 deadline 可设为：

D_i(t) \leq \hat{\tau}^{lr}_i

例如：

|**状态**|**监控策略**|
|---|---|
|锅稳定且倾角远离阈值|2 Hz、单视角、低质量|
|倾角快速增大|20–30 Hz、多视角|
|距离最后可恢复时刻不足 200 ms|抢占低风险任务|
|无法保证按时完成|减速、暂停或紧急停车|

这使 deadline 不再是人工固定的 20 ms 或 50 ms，而是由任务语义和物理风险在线产生。

---

## **3.5 第二项创新：混合关键级与质量自适应**

可以将监控任务划分为：

|**级别**|**示例**|**资源不足时的行为**|
|---|---|---|
|Safety-critical|人体碰撞、危险力、重物跌落|必须满足 deadline；否则触发 fail-safe|
|Mission-critical|抓取稳定、目标身份、步骤顺序|保留监控，降低其他任务质量|
|Best-effort|姿态优化、效率条件|降频、跳帧或暂时关闭|

优化目标不能只写成最大吞吐量，而应写成类似：

\max_{\pi,q} \sum_i U_i(q_i,t)

满足：

\Pr(R_i \le D_i) \ge 1-\delta_i, \quad i\in\mathcal{M}_{critical}

其中 U_i 是监控带来的风险降低或检测效用。

运行时可以执行：

1. 提高高风险 monitor 的周期和优先级；
2. 降低低风险 monitor 的视角和分辨率；
3. 复用感知中间结果；
4. 延后低优先级 VLM 请求；
5. 避免过期数据继续参与判定；
6. 在任务不可调度时触发保守控制。

---

## **3.6 第三项创新：资源不可调度时的安全降级协议**

真正的运行时保证不应只覆盖“资源足够时怎么调度”，还要覆盖：

资源不够时系统做什么。

可以定义三类结果：

```text
SCHEDULABLE
  所有关键 monitor 能在 deadline 前完成

DEGRADED
  关键 monitor 可保证，低关键级 monitor 降频或降质量

UNSAFE-TO-CONTINUE
  无法保证关键 monitor，必须减速、暂停或执行 fail-safe
```

这比直接丢弃任务更符合机器人安全语义。

需要注意，**sched_ext 本身更适合软实时和策略实验，不宜直接宣称硬实时保证**。如果论文要强调形式化 deadline guarantee，建议采用分层执行：

```text
控制与紧急安全路径
    → PREEMPT_RT / SCHED_FIFO / SCHED_DEADLINE

关键监控任务
    → SCHED_DEADLINE 或受约束的实时 executor

普通监控和后台任务
    → sched_ext 自适应调度

GPU 推理任务
    → CUDA stream priority + admission control + quality selection
```

如果只使用 sched_ext，则论文应使用：

- firm/soft real-time；
- probabilistic SLO；
- empirical bounded latency；

而不是 hard real-time guarantee。

---

## **3.7 工作二必须支持多种监控器**

这是保证它独立于工作一的关键。

至少接入三类监控前端：

1. **CaM 类几何代码监控器**；
2. **人工编写或 STL/LTL 类 monitor**；
3. **学习式故障检测器**，例如 VLA latent failure score；
4. 可选：力、触觉或普通异常检测器。

AgentChord 已经通过预先构建带恢复分支的任务图和低延迟 monitor 实现快速恢复，因此工作二不要把“检测后切换到预编译恢复动作”作为主创新。你的贡献应限于**监控任务的资源分配、语义截止时间和不可调度时的安全处理**。 

---

## **3.8 工作二的实验设计**

### **实验中必须固定 monitor 正确性**

核心实验应使用：

- 人工构造的 ground-truth monitor；
- 或经过离线验证的 monitor；
- 或回放固定 monitor trace。

这样可以避免出现：

检测失败到底是代码写错了，还是调度晚了？

工作一生成的 monitor 可以作为额外 case study，但不能成为唯一输入。

### **研究问题**

|**RQ**|**问题**|
|---|---|
|RQ1|资源竞争会在多大程度上延迟机器人 monitor？|
|RQ2|固定频率和普通 EDF 是否会错过最后可恢复时刻？|
|RQ3|语义风险生成的 deadline 是否优于固定 deadline？|
|RQ4|质量自适应能否在相同资源下保留更多关键检测？|
|RQ5|过载时的安全降级是否能改善最终任务成功与安全表现？|
|RQ6|运行时能否跨硬件和监控类型工作？|

### **基线**

- 固定周期、固定质量；
- ROS 2 SingleThreadedExecutor；
- ROS 2 MultiThreadedExecutor；
- Linux CFS；
- 静态实时优先级；
- Rate Monotonic；
- EDF；
- cgroup 权重或资源配额；
- RED-like DAG scheduler；
- 只有风险优先级、没有 deadline；
- 只有 deadline、没有 fidelity adaptation；
- 完整方法。

### **干扰矩阵**

|**资源**|**干扰方式**|
|---|---|
|CPU|stress-ng、规划任务、日志和序列化任务|
|GPU|VLA 推理、分割、视觉检测并发|
|内存|带宽压力、缓存竞争|
|ROS 2|高频 callback、多 DAG 并发|
|传感器|相机掉帧、时间戳漂移|
|网络|远程 VLM 延迟与抖动|
|任务结构|monitor 动态加入和删除|

### **指标**

- critical monitor deadline miss ratio；
- end-to-end detection latency；
- observation staleness；
- last-recoverable-point miss rate；
- proactive lead time；
- 故障发生前完成报警的比例；
- safety-critical recall；
- 最终任务成功率；
- fail-safe 触发次数；
- CPU/GPU 利用率；
- monitor fidelity；
- 能耗；
- 调度器开销；
- 对控制循环 jitter 的影响。

---

# **四、两篇工作之间应如何共享接口**

可以共享一个很薄的 **Monitor Contract**：

```yaml
monitor_id: keep_pan_level

semantic:
  predicate_id: pan_tilt
  temporal_scope: during_transfer
  severity: critical
  fallback: stop_and_stabilize

runtime:
  robustness: 2.4
  uncertainty: 0.7
  max_data_age_ms: 40
  estimated_time_to_violation_ms: 280

fidelity_levels:
  - id: low
    wcet_ms: 3
    expected_error: 1.8
  - id: medium
    wcet_ms: 9
    expected_error: 0.8
  - id: high
    wcet_ms: 25
    expected_error: 0.3
```

职责划分：

|**接口字段**|**主要归属**|
|---|---|
|predicate、entity、part、frame、unit|工作一|
|uncertainty、robustness interval|工作一产生，工作二可选使用|
|severity、fallback|共同契约|
|WCET、quality、resource demand|工作二|
|period、deadline、max data age|工作二|
|schedulability、overload action|工作二|

最重要的一条原则是：

工作二必须允许其他前端直接填充这个接口，而不能要求必须先运行工作一的完整 verifier。

这样，即使工作一不存在，工作二依然是一篇完整论文。

---

# **五、怎样避免被认为是“切香肠”**

## **5.1 独立性检查**

两篇论文分别满足以下条件才算合理拆分：

|**检查项**|**工作一**|**工作二**|
|---|---|---|
|独立问题陈述|生成 monitor 不可信|正确 monitor 可能来不及执行|
|独立算法核心|编译、验证和不确定性|调度、准入和质量自适应|
|独立基线|CaM、LLM、DSL、formal monitor|CFS、EDF、RM、ROS 2、RED|
|独立指标|正确率、校准、误报漏报|deadline、延迟、资源和安全窗口|
|独立实验变量|monitor 生成与噪声|资源竞争与调度策略|
|可单独成立|固定资源即可评估|使用人工 monitor 即可评估|

一个直接判断标准是：

删除工作一后，工作二仍可使用人工 monitor 完成所有核心实验；删除工作二后，工作一仍可在资源充足条件下证明监控可信性。

满足这一条，通常就不是简单拆分。

---

## **5.2 不要这样拆**

以下拆法风险较高：

### **错误拆法 A**

- 第一篇：设计 DSL；
- 第二篇：为 DSL 增加 verifier。

这更像一篇工作的两个模块。

### **错误拆法 B**

- 第一篇：生成可信 monitor；
- 第二篇：让这些 monitor 跑得更快。

“跑得更快”缺少独立科学问题。

### **错误拆法 C**

- 第一篇：CaM + uncertainty；
- 第二篇：CaM + priority scheduling。

如果第二篇只根据 uncertainty 调整 Linux nice 或 cgroup weight，工作量和理论深度不足。

正确拆法必须是：

- 第一篇建立**语义保证体系**；
- 第二篇建立**基于物理风险和可恢复窗口的实时任务模型及运行时保证**。

---

# **六、推荐的实施顺序**

## **阶段 0：共同可行性验证，4–6 周**

先不要完整实现两套系统，而是建立两项 motivation。

### **可信度 motivation**

收集约 100–300 个 CaM monitor generation 样本，统计：

- 不能执行的代码；
- 能执行但对象绑定错误；
- 坐标系和单位错误；
- 阈值不合理；
- 时序阶段错误；
- 遮挡下的过度确定判定。

需要证明：

“可执行”与“语义正确”之间存在显著差距。

### **资源竞争 motivation**

选择 5–10 个监控器，在以下条件下运行：

- 无干扰；
- CPU 干扰；
- GPU 干扰；
- CPU+GPU 联合干扰；
- VLA/分割/控制并发。

测量：

- 监控周期抖动；
- 数据陈旧度；
- 检测延迟；
- 是否错过最后可恢复时刻。

需要证明：

即使 monitor 完全正确，资源竞争仍会使检测失去实际作用。

只有两条 motivation 都成立，再正式并行推进。

---

## **阶段 1：优先完成工作一的可发表闭环**

推荐先完成工作一，原因是：

1. 它与 CaM 的逻辑缺口最直接；
2. 问题边界更容易讲清楚；
3. 不依赖复杂内核和 GPU 调度；
4. 可以先构建 Monitor IR，后续成为工作二的一种前端；
5. 能先产出约束、鲁棒性和故障标签，为工作二提供真实 workload。

但工程上可以同时开始工作二的 profiling 和 runtime harness。

---

## **阶段 2：将工作二扩展为通用运行时**

工作二不要只读取工作一生成的 monitor。应逐步接入：

```text
CaM adapter
Manual/STL adapter
Learned detector adapter
Force/tactile adapter
```

随后做：

- CPU/ROS 2 调度；
- GPU 质量选择；
- 风险驱动 deadline；
- admission control；
- overload fail-safe；
- 多硬件评估。

---

# **七、两篇论文的最低创新门槛**

## **工作一的 go/no-go 条件**

建议至少达到以下效果中的大部分：

- 显著降低可执行但语义错误的 monitor；
- 自动发现大部分单位、坐标系和对象绑定错误；
- 反例驱动修复明显优于直接重新生成；
- 不确定性语义显著降低临界状态下的误报和漏报；
- 校准覆盖率接近目标置信水平；
- 在未见任务上不明显损失泛化能力；
- 运行时开销仍满足在线监控要求。

如果直接 CaM 生成的 monitor 几乎不出现错误，那么这条论文 motivation 会比较弱；需要扩大任务复杂度和系统性扰动，而不能人为构造不现实错误。

## **工作二的 go/no-go 条件**

需要至少证明：

- 实际监控负载在竞争下确实会错过可恢复窗口；
- 普通优先级、EDF 或静态频率无法解决；
- 风险驱动 deadline 比固定 deadline 更有效；
- 质量自适应能保留高风险检测，而不只是降低总体延迟；
- 不可调度检测和 fail-safe 能避免继续盲目执行；
- 调度器不会明显破坏控制循环；
- 效果能跨平台或跨 monitor 类型复现。

如果普通 EDF 已经解决绝大多数问题，那么工作二应进一步引入：

- GPU 非抢占；
- 多精度监控；
- 动态 monitor 加入；
- 数据新鲜度；
- 不确定的 time-to-violation；
- 混合关键级过载场景。

---

# **八、最终建议的博士课题叙事**

两篇工作可以形成非常连贯的主线：

## **工作一**

**从开放式任务语义编译出可信的运行时监控器。**

解决：

- 监控规范生成；
- 视觉和几何接地；
- 代码一致性；
- 不确定性判定。

## **工作二**

**根据监控器产生的任务风险，在资源受限系统中提供及时的运行时监控。**

解决：

- 语义风险映射；
- 实时任务生成；
- CPU/GPU 协同调度；
- 安全降级。

## **合并后的系统故事**

```text
自然语言任务
    │
    ▼
可信 Monitor Compiler            ← 工作一
    │
    ├─ verified predicate
    ├─ robustness
    ├─ uncertainty
    └─ severity
    │
    ▼
Risk-aware Monitor Runtime       ← 工作二
    │
    ├─ semantic deadline
    ├─ fidelity selection
    ├─ CPU/GPU scheduling
    └─ overload fail-safe
    │
    ▼
可解释、可信、及时的机器人故障监控
```

因此，**可以拆，而且建议拆**。较稳妥的边界是：

- 第一篇做到 **contract-relative correctness + calibrated uncertainty**；
- 第二篇做到 **semantic deadline + mixed-criticality execution + safe degradation**；
- 不把“完整自然语言语义绝对正确”作为第一篇的保证；
- 不把“简单提高优先级、降低延迟”作为第二篇的贡献。

最终两篇论文之间共享 Monitor Contract 和实验基础设施，但各自具有独立的问题、方法、基线和评价闭环。