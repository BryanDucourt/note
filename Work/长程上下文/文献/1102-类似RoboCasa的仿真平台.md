# 和 RoboCasa 类似的仿真平台（1102 检索）

**起因**：导师提议用 ManiSkill 做补充数据集。用户 1102 晚："检索一下有多少跟robocasa类似的数据集可以用"。

**做法**：
- 两个子代理分头检索：一路只查 ManiSkill，一路查其他平台。每条事实都附了出处。
- Claude 抽查了四处：ManiSkill 的场景文档、RMMBench 正文、PARTNR 表 2、MolmoSpaces 的 README。
- 没抽查的，标"（未核对）"。检索日期 2026-11-02。

## 一、ManiSkill（ManiSkill3）

- **版本**：3.0.1（2026-04-21）。代码是 Apache-2.0 许可，资产是 CC BY-NC 4.0。GPU 并行仿真只在 Linux + NVIDIA 上，渲染要 Vulkan 驱动。
- **机器人**：能移动的有 Fetch 和 XLeRobot，没有 PandaOmron。Fetch 有躯干升降、头部相机和手上相机，默认 128×128。
- **相机**：RGB、深度、分割（能分到每个物体和每节连杆）、点云。
- **场景**：自带 RoboCasa、ReplicaCAD、AI2-THOR 三套。文档原话是这些大场景 "there are not any trainable tasks with defined success/fail conditions"（Claude 核对），开发组还在写类似 ReplicaCAD 整理挑战赛的任务代码。
  - RoboCasa（RoboCasaKitchen-v1）：120 个厨房（10 种布局 × 12 种风格），只加载设施，不加载物体，也没有成功判定。门和微波炉按钮的状态代码还在调 robosuite 的接口，这个移植里没有这些接口（未核对）。
  - ReplicaCAD：柜门、抽屉可以动，只能用 Fetch，没有成功判定。
  - AI2-THOR：只注册了 ArchitecTHOR，不加载可动部件，只有 8 类物体能挪动（未核对）。
- **有任务的**：
  - 桌面任务 21 个；
  - 移动操作只有开柜门、开抽屉两个，以及 MS-HAB。
- **MS-HAB**（ICLR 2025）：
  - ReplicaCAD 里 3 个整理任务（TidyHouse、PrepareGroceries、SetTable），拆成拿、放、开、关几种子任务；
  - Fetch，抓取按真实物理；
  - 任务由一个"完美的任务规划器"事先拆好，导航直接瞬移过去，没有语言指令；
  - 基线最高约 20%；示范数据约 490 GB。
- **运动规划**：用 mplib，桌面任务有现成的解法，不支持移动底盘。
- **速度**：GPU 批量仿真很快，但 RoboCasa 这种整间屋子的场景只有约 25 FPS。我们的瓶颈在模型调用，这个长处用不上。

**对我们**：用 ManiSkill 做厨房里的任务，任务、成功判定和技能都要自己写。它能给的是另一个身体（Fetch）和真实的抓取物理。

## 二、其他平台

子代理一共列了 22 个，按和我们的接近程度分组：

| 平台 | 仿真 | 机器人 | 场景和可动设施 | 任务和判定 | 备注 |
|---|---|---|---|---|---|
| BEHAVIOR-1K | OmniGibson，v3.9.4（2026-10-30） | R1Pro：底盘、躯干、腕部相机 | 50 个场景，柜子、冰箱、电器 | 1000 个长任务，有判定；2026 挑战赛 100 个任务 | 已经在用；自带原语不能用（会审第九节） |
| MolmoSpaces（Ai2） | MuJoCo（数据生成和评测只支持 MuJoCo），v0.2.9 | Franka FR3、RB-Y1、Unitree G1 | iTHOR 120 个、ProcTHOR 约 12 万、ProcTHOR-Objaverse 约 11 万、Holodeck 约 11 万个屋子；冰箱、烤箱、门 | 8 个原子任务 | 新的独立平台；组合任务和技能要自己写；RB-Y1 有没有躯干，README 没写 |
| RMMBench | MuJoCo，RoboCasa 和 Objaverse 资产 | Omron 底盘 + Franka；头部、腕部相机 | RoboCasa 厨房 | 正文 90 个场景（摘要写 70 个）：60 个操作、30 个导航加操作；按规则判定 | 设计最接近，见第三节；没写许可，一个月前才发 |
| LW-BenchHub | Isaac Lab | PandaOmron 等 | 100 个厨房配置 | 138 个 RoboCasa 任务 + 130 个 LIBERO 任务 | 把 RoboCasa 搬到 Isaac 上（未核对） |
| Habitat 3.0（HAB、LangR） | Habitat-Sim | Fetch、Spot、Stretch | ReplicaCAD、HSSD | 3 个长整理任务；LangR 有 1000 个语言测试任务 | Meta 在 v0.3.4 以后停止维护（未核对） |
| PARTNR | Habitat 3.0 | Spot + 模拟的人 | 60 个 HSSD 房子 | 10 万训练、1000 测试的多步语言任务 | 评大模型规划器的规程最成熟，见第三节 |
| HomeRobot OVMM | Habitat | Stretch | 60 个 HSSD 房子 | 一种拿放模板 | 2024 年以后没更新（未核对） |
| EMMOE | Habitat 2.0 | Fetch | Replica | 100 个长任务 | 评过 o1、GPT-4o 等（未核对） |
| InternUtopia（原 GRUtopia） | Isaac Sim 4.5 | 人形、四足加臂、Franka | 10 万个场景，开放 100 个 | 导航、社交导航、移动操作 | 大模型做移动操作 0%（未核对） |
| Kitchen-R | Isaac Sim | Mobile ALOHA | 一个数字孪生厨房 | 563 条语言指令，4–8 步 | （未核对） |
| Genie Sim（智元） | Isaac Sim 5.1 | Genie G2 | 家、商店、办公室 | 200 多个任务，由 VLM 判定 | （未核对） |

- **专门评多模态智能体的**：EmbodiedBench（THOR、Habitat、CoppeliaSim 上 1128 个任务，有高层技能）、Embodied Agent Interface（符号化，不看图）、VLABench（MuJoCo，固定机械臂，60 个原子任务 + 40 个组合任务）、RoboCerebra（LIBERO 上的长任务）、EmbodiedEval。
- **固定机械臂的**：LIBERO（以及 Plus、PRO）、RLBench（以及 Colosseum、GemBench）、CALVIN、RoboTwin 2.0、ARNOLD、Franka Kitchen、RoboVerse。机器人不能走，"够不够得着"只取决于机械臂的工作空间。
- **真机数据**：Open X-Embodiment、DROID，都不是能交互的仿真。

## 三、和新重心直接相关的两篇（Claude 读了正文的相关部分）

### RMMBench（CoRL 2026，[arXiv 2610.05414](https://arxiv.org/abs/2610.05414)，v2 2026-10-23）

- **设定**：
  - MuJoCo 加 RoboCasa 的厨房；Omron 全向底盘加 Franka 机械臂；头部和腕部相机，480×480。
  - 模型每步选一个带参数的技能（拿、放、前进、转身、开门、开水槽、拧旋钮、观察、回忆、结束），给出 0–1000 的框，再从匿名编号的候选抓取姿态里挑一个；底层用 RRT 和逆运动学执行。
  - 深度和分割只在底层用（找放置点），模型看到的是相机图。
- **结果**（成功率 %）：

| 模型 | 短任务 | 长任务 | 导航加操作，无俯视图 | 加俯视图 |
|---|---:|---:|---:|---:|
| GPT-6-Astra | 88.89 | 74.17 | 40.00 | 48.89 |
| Gemini-3-Flash | 58.97 | — | 21.11 | 30.00 |
| Gemini-3-Pro | — | 71.48 | 17.78 | 25.56 |
| Doubao-SEED-2.0-Pro | — | 73.68 | 11.11 | 21.11 |
| Gemini-2.5-Pro | — | 77.14 | 10.00 | 17.78 |
| Kimi-K3 | — | — | 6.67 | 12.22 |
| Qwen-3.5-Plus | — | — | 5.56 | 8.89 |
| DeepSeek-V4.1-Flash | 22.22 | 31.73 | 3.33 | 7.78 |

  "—"是这次没抄下来的数，要看原表。
- **失败分析**：
  - 目标定位偏了，比如水龙头把手；目标被挡住时决策出错；框不够准；
  - 失败以后常常重复同一个动作，不用腕部相机换个角度看清楚；
  - 没有俯视图时，还没走到台前就开始操作，反复去拿够不着的东西，忘了前几帧看到的障碍。

### PARTNR（Meta，[arXiv 2411.00081](https://arxiv.org/abs/2411.00081)）表 2

Llama3.1-70B 规划器（ReAct，部分可观测）：

| 技能 | 感知 | 成功率 |
|---|---|---:|
| 理想技能 | 真值世界图 | 0.73 |
| 学出来的技能 | 真值世界图 | 0.57 |
| 学出来的技能 | ConceptGraphs（真实感知） | 0.30 |

## 四、对我们意味着什么（只写事实）

1. **ManiSkill 没有现成的厨房任务**：官方说明这些大场景没有带成功判定的任务；有任务的只有 MS-HAB 的 3 个整理任务。
2. **RMMBench 和我们的新重心最近**：用的是 RoboCasa 厨房和类似的身体，评的正是多模态模型做移动操作时的空间定位。
   - 导航加操作，GPT-6-Astra 40.00，其余模型都不超过 21.11；
   - 加俯视图以后每个模型都涨了，但 GPT-6-Astra 涨 8.89，DeepSeek-V4.1-Flash 涨 4.45，Qwen-3.5-Plus 涨 3.33，差距没有缩小；
   - 没做"把深度给模型"的对比。
3. **DeepSeek-V4.1-Flash 在 RMMBench 上几乎每项都最低**。我们用的 deepseek-flash 是不是这个版本，没核对。
4. **PARTNR 里，感知从真值换成真实感知，成功率从 0.57 掉到 0.30**；它也按"给不给真值世界图"分条件报告。
