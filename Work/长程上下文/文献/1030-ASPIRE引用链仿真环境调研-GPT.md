# ASPIRE 引用链里的仿真环境（GPT 检索，Claude 核对）

来源：用户 1030 转来的 GPT 检索，原文放在第三节。Claude 当天只核对了和第三平台选择有关的说法，见第一节。和平台会审的关系见 `会审/1030-轻量机器人平台会审.md` 第六节。

## 一、核对

| GPT 的说法 | 核对结果 | 出处 |
|---|---|---|
| RoboCasa365 有 365 个任务、2,500 个厨房场景，其中 300 个是组合任务 | **属实**。65 个原子任务、300 个组合任务，2,500 多个厨房，3,200 多个物体。v1.0 发布于 2026-02-18。场景只有厨房，没有多房间住宅 | [robocasa.ai](https://robocasa.ai/) |
| 2026-10-28 开放 agentic 榜，接受在基础策略外面加规划、记忆、重试、工具调用的系统 | **日期属实**，见官网新闻栏。**GPT 漏了条件**：<br>① 必须运行一个不改权重的基础策略，写明它的 HF 仓库、提交号和许可证，也就是要有一个训练好的策略检查点（VLA 一类）；<br>② 只能用策略本来就有的观测，不能用仿真特权状态，回合中也不能拿到真值成功信号；<br>③ 必须是新模型或新的 agent 框架，"现成模型加现成编程 agent 简单指挥"不算；<br>④ 代码公开，一句话能装好。闭源 API 的规划器要交一份够跑完整评测的 API 额度，官方重跑，以官方的数为准。<br>评测共 50 个任务：18 个原子任务，16 个训练里见过的组合任务，16 个没见过的组合任务。标准榜第一总分 64.4，没见过的组合任务成功率 45.2%。agentic 榜目前还没有条目 | [agentic 榜](https://robocasa.ai/leaderboard/agentic.html)、[提交说明](https://github.com/robocasa-benchmark/leaderboard)、[标准榜](https://robocasa.ai/leaderboard.html) |
| ENPIRE 在 RoboCasa 里调用了视觉、抓取和运动规划工具 | **论文里属实**。附录 D 里，编程 agent 写 Python，调用 SAM3、AnyGrasp、cuRobo 或 PyRoki、GOAT 导航，也能调用 GR00T 策略；运行中不给目标位姿和仿真物体状态。**但这套工具拿不到**：公开仓库 NVlabs/ENPIRE 里，按路径名搜不到任何 RoboCasa 文件 | [ENPIRE 附录 D](https://arxiv.org/html/2606.19980)、[NVlabs/ENPIRE](https://github.com/NVlabs/ENPIRE) |
| ASPIRE 在 BEHAVIOR-1K 上只做了两个任务 | **属实**。同一个场景里拿收音机、拿汽水罐。<br>做法：R1Pro；cuRobo 规划底盘和手臂；SAM3 和 Contact-GraspNet 做感知；只用头部和两只手上相机的 RGB-D，不给 BDDL、物体表和实例分割。<br>结果：README 说 Opus 5.5 在这些任务上 97–100%；behavior.md 的实测记录是收音机 2/3、汽水罐 1/3～2/3，失败都在抓取那一下。<br>环境：要 Isaac Sim 5.1、Python 3.11，cuRobo 要按显卡编译；4090 上占 12 GB 显存 | [behavior.md](https://github.com/NVlabs/ASPIRE/blob/main/docs/behavior.md)、[NVlabs/ASPIRE](https://github.com/NVlabs/ASPIRE) |
| CaP-Gym 把已有环境包成编程 agent 能调用的接口 | **属实**。39 个任务，覆盖 Robosuite、LIBERO-PRO、BEHAVIOR（R1Pro），**不含 RoboCasa**。最高一档 S1 用真值掩码和物体位姿，其余各档用真实感知 | [CaP-X](https://arxiv.org/html/2603.22435)、[capgym/cap-x](https://github.com/capgym/cap-x) |
| GR00T N1 用的是 RoboCasa 的 24 个原子任务 | 没核对 | — |
| 其余环境（MetaWorld、Isaac Gym、VirtualHome、Minecraft 等） | 没逐条核对，和眼下的决定关系不大 | — |

**Claude 补查的两条**

- **RoboCasa 默认机器人 PandaOmron 的身体**：
  - 底盘能前后、左右平移和转向；躯干能升高 0～0.34 m。
  - 左、中、右三个 agentview 相机装在底盘支架上，跟着机器人走，自己不能转；手腕上另有一个相机。
  - 图像默认 128×128，可以打开深度。
  - 出处：robocasa 的 `camera_utils.py`、`env_utils.py`，robosuite 的 `omron_mobile_base.xml`。
- **和 SAM 3 检查有关的一个数**：ENPIRE 附录 D 里，SAM3 在 RoboCasa"台面到柜子"的场景里认物体，256×256 时 20 次对 10 次，提高分辨率后对 14 次。

## 二、和我们的关系（Claude 的看法）

1. **第三平台：RoboCasa 应该比会审里排得靠前。**
   - **身体**：相机装在底盘和手腕上、自己不能转；底盘会被台面挡住；手臂有够得着的范围，站的位置不对就够不着。这几类问题和 BEHAVIOR 的 R1Pro 很像。
   - **轻**：MuJoCo，物理跑在 CPU 上，不用编译 CUDA 算子。在 5090 上，安装风险比 ManiSkill 小（ManiSkill 用 SAPIEN 和 Vulkan，有 5090 段错误的 issue）。
   - **维护**：2026-10-28 还有更新。许可是 GPT 会审时查的：代码 MIT，资产 CC BY 4.0，Claude 没有再核。
   - **不足**：
     - 只有厨房，没有跨房间；
     - 导航、点选、开关拿放这些语义工具还得自己写，因为 ENPIRE 的工具没公开，CaP-Gym 也不含 RoboCasa。
     - Gemini 原来反对 RoboCasa 的这条理由仍然成立。
2. **agentic 榜不能直接当我们现有系统的发表口。**
   - 要上榜，底层必须是一个训练好的策略，执行器也不能再用实例掩码。这等于推翻 1029"不用 VLA"的决定。
   - 底层失败占的比例会很大：没见过的组合任务上，最好的策略也只有 45%。上下文的作用要在这么大的底层噪声上测出来。
   - 任务是官方定的 50 个，不一定需要"早先看到、后来看不见了"的信息。
   - NVIDIA GEAR 这一系（CaP-X、ASPIRE、ENPIRE）已经在 RoboCasa365 上和 GR00T、CaP-X 做过比较，很可能会来投这个榜。
3. **BEHAVIOR 上的抓取有了一条现成的路，但先记下不动。**
   - ASPIRE 和 CaP-X 公开了一套 R1Pro 的感知抓取（SAM3、Contact-GraspNet、cuRobo），不用 VLA。这重新打开了"BEHAVIOR 只做导航和观察"这件事。
   - 但简单拿取的实测通过率只有 1/3～2/3，而且要另装 Isaac Sim 5.1，还要为 5090 编译 cuRobo。
4. **相关工作：CaP-X 多轮那一节值得读。**
   - 它的多轮分档是逐档加东西：先只给执行反馈，再加原图，再加画面变化的描述。比的就是"执行过程中给哪些信息有用"，是这条链里离"上下文值多少"最近的一篇。
   - 区别：它做的是编程 agent 的短操作任务；ASPIRE 的技能库是跨任务的记忆，不是单个任务里的上下文。
5. **GPT 选环境的条件和 THOR 会审的结论一致。**这个条件是：过去看到的信息在当前画面里已经没了，却还会影响后面的动作；做错了会改变世界，逼着它更新旧判断。

## 三、GPT 原文

沿着 **ASPIRE 及其近五年的具身相关引用**查下来，我认为对我们最值得关注的是 **RoboCasa、BEHAVIOR-1K、LIBERO/robosuite 和 RLBench**。这条引用链也覆盖了奖励学习仿真器、家庭语义环境，以及 Minecraft 等游戏环境；它们承担的研究任务差别很大。

这次以 **2021 年 10 月至今的引用论文**为主，追查了实验设置、附录和官方实现。基础仿真器即使发布更早，只要这些论文实际使用了，也纳入介绍。

先分清几个层级：**MuJoCo、Bullet 属于物理引擎；robosuite、OmniGibson 属于仿真框架；LIBERO、BEHAVIOR-1K 属于任务基准。** CaP-X 中的 CaP-Gym 则负责把已有环境包装成 coding agent 能调用的接口。研究高层规划或历史管理，可以在这些物理环境上使用运动规划器、技能程序或学习策略，并不要求采用 VLA。[robosuite 文档](https://robosuite.ai/docs/overview.html)、[CaP-X §2](https://arxiv.org/html/2603.22435)

**先看与实际机器人操作最相关的环境。**

| 环境与底层 | 引用链中的使用者 | 环境内容与研究特点 |
|---|---|---|
| **LIBERO / LIBERO-Pro**；robosuite、MuJoCo | ASPIRE、OpenVLA、AtomVLA；GaP 当前公开实现 | 以机械臂操作为主，包含物体搬运、容器放置、抽屉等任务。LIBERO 按空间关系、物体、目标和较长任务组织；Pro 加入位置、物体和目标扰动。任务、示范和评测协议比较现成，适合研究规划接口、执行失败和分布变化。场景范围相对集中，研究历史记忆时需要进一步挑选存在延迟信息需求的任务。[OpenVLA 附录 E](https://arxiv.org/html/2406.09246v3#A5)、[LIBERO-Pro](https://arxiv.org/abs/2510.03827) |
| **robosuite**；MuJoCo | ASPIRE、CaP-X | 提供可组合的机器人、物体、控制器和任务，包括抓取、堆叠、擦拭、插接、双臂举物和交接。适合把接触、碰撞、抓取姿态、双臂协调等问题单独拿出来研究。若要做复杂家庭长程任务，需要自行扩展任务组织。[官方介绍](https://robosuite.ai/docs/overview.html)、[CaP-X](https://arxiv.org/html/2603.22435) |
| **RoboCasa / RoboCasa365**；robosuite、MuJoCo | GR00T N1、ENPIRE | 面向厨房日常操作，包含柜门、抽屉、水龙头、炉灶、微波炉、咖啡机，以及物体在不同位置之间的搬运。当前 RoboCasa365 包含 **365 个任务、2,500 个厨房场景，其中 300 个是组合任务**。它同时提供具体物理操作和较完整的生活任务，是高层 agent、任务进度维护和失败恢复很值得考虑的平台。[官方项目](https://robocasa.ai/)、[ENPIRE 附录 D](https://arxiv.org/html/2606.19980#A4) |
| **BEHAVIOR-1K / OmniGibson**；Isaac Sim | ASPIRE、CaP-X；引用中也有 BEHAVIOR-1K 本身 | 覆盖整屋家庭活动，机器人需要移动、寻找和操作物体，还可能涉及清洁、加热、切分等状态变化。它对“物体在哪里、经历过什么、下一步需要什么条件”提供了丰富空间。复杂活动同时依赖物理仿真和语义状态转换规则，例如部分切分、烹饪过程通过规则实现；具体任务应分别看其机制。[官方概念说明](https://github.com/StanfordVL/BEHAVIOR-1K/blob/main/docs/getting_started/important_concepts.md)、[状态转换规则](https://behavior.stanford.edu/omnigibson/transition_rules.html) |
| **RLBench**；CoppeliaSim、PyRep | Uni-Skill；VoxPoser 官方演示实现 | 提供大量机械臂操作任务及变体，例如开抽屉、按按钮、搬运、整理和工具操作，并有视觉观测、示范生成和不同控制方式。适合研究技能组合、多阶段操作和执行恢复。Uni-Skill 的实验包含基本技能能够覆盖的任务，以及需要扩展技能的任务。[RLBench 官方仓库](https://github.com/stepjam/RLBench)、[Uni-Skill §IV](https://arxiv.org/abs/2603.02623) |
| **PyBullet 中的 Ravens 风格桌面环境** | Code as Policies、LRLL；RoboGen 公开重实现 | 常见配置是机械臂、夹爪、彩色积木、碗和容器，执行排列、堆叠、分类和多步搬运。场景简洁，任务和物体状态容易控制，适合快速研究程序生成、技能库和历史信息机制。视觉复杂度及家庭状态种类较少。[Code as Policies](https://arxiv.org/abs/2209.07753)、[LRLL §IV-A](https://arxiv.org/abs/2406.18746) |
| **Gazebo + ROS / MoveIt** | RoboCodeX；LRLL 的真机迁移部分 | 属于通用机器人系统仿真，可接入机器人模型、传感器、碰撞、关节控制和运动规划。RoboCodeX 在其中构建物体、柜体与多步操作任务；LRLL 用它衔接双臂真机。适合已有 ROS 系统的研究，但任务集合、成功条件和数据生成通常需要较多自行建设。[RoboCodeX §4、附录 A](https://arxiv.org/abs/2402.16117)、[LRLL §IV-D](https://arxiv.org/abs/2406.18746) |

这里有三个会影响我们判断的具体差别：

- **ASPIRE 在 BEHAVIOR-1K 上实际选了两个任务：导航后拿起汽水罐、导航后拿起收音机。** 它在这个平台上的实验范围，比 BEHAVIOR-1K 全部家庭活动的范围窄得多。ASPIRE 的另一组长程实验来自 LIBERO-Pro Long。[ASPIRE §3.2](https://arxiv.org/html/2607.00272v2)、[官方 BEHAVIOR 配置](https://github.com/NVlabs/ASPIRE/blob/main/docs/behavior.md)
- **GR00T N1 使用的 RoboCasa 是 24 个原子任务**，主要测试基础操作。当前 RoboCasa365 的 300 个组合任务属于后来扩展。GR00T 另外使用了 **DexMimicGen 的 9 个双臂／灵巧操作任务**和 **24 个 GR-1 桌面任务**；前者涉及穿线、装配、倒物、托盘搬运等，后者主要是物体在不同容器或位置之间的转移。这些任务套件也值得参考，但要分别判断其长程程度。[GR00T N1 §4.1](https://arxiv.org/abs/2503.14734)
- **VoxPoser 的主要论文实验是真机；RLBench 是官方开放的演示实现。** 官方仓库还明确说明演示实现使用了 RLBench 的物体掩码，因此复现时应单独考虑感知输入条件。[VoxPoser 官方说明](https://github.com/huangwl18/VoxPoser)

**另一批环境更偏向技能训练、奖励设计和物理控制。**

| 环境 | 哪些论文使用 | 具体介绍与适用问题 |
|---|---|---|
| **ManiSkill2 / SAPIEN** | Text2Reward | 包含插接、装配、搬桶、杂物抓取等任务，覆盖固定／移动底座、单／双臂和刚体／软体操作，提供 RGB-D、点云和示范。它突出物体几何变化和可泛化操作技能。适合研究技能学习、对象变化及接触执行；研究高层历史通常还需要构造任务组合。[ManiSkill2](https://maniskill2.github.io/)、[Text2Reward §3](https://arxiv.org/abs/2309.11489) |
| **MetaWorld / MuJoCo** | Text2Reward | 由 50 类机械臂操作任务构成，例如推拉、开门、按按钮、插接，主要服务多任务学习和元学习。任务相对紧凑，适合比较奖励、学习效率和技能迁移；完整家庭活动和长期状态依赖需要额外设计。[官方文档](https://metaworld.farama.org/) |
| **Isaac Gym / IsaacGymEnvs，以及双手操作任务集** | Eureka、DrEureka、Eurekaverse | 核心优势是 GPU 上的大规模并行物理仿真。Eureka 用它学习多种机器人控制和灵巧手任务；DrEureka 研究四足、灵巧操作及仿真到真机迁移；Eurekaverse 生成逐渐变难的四足跑酷地形。主要适合奖励设计、低层控制和课程学习。[Eureka](https://eureka-research.github.io/)、[DrEureka](https://arxiv.org/abs/2406.01967)、[Eurekaverse 官方实现](https://github.com/eureka-research/eurekaverse) |
| **MuJoCo + MPC，及 Gym 的 MuJoCo 控制任务** | Language to Rewards；Text2Reward | Language to Rewards 把语言要求转成奖励，通过模型预测控制生成机器人行为；Text2Reward 还使用 Hopper、Ant 等运动控制任务。这类环境便于研究“目标如何转成可优化的控制信号”，任务重心落在运动与接触控制。[Language to Rewards](https://language-to-reward.github.io/)、[Text2Reward](https://arxiv.org/abs/2309.11489) |
| **Genesis；RoboGen 公开版本采用 PyBullet** | RoboGen | RoboGen 自动生成任务、场景和学习过程，涉及操作、运动及不同材料对象。**论文使用的是当时内部版 Genesis；公开仓库给出的是 PyBullet 重实现。** 因此，评估其开放代码能支持哪些任务时，应按公开实现的能力计算。它更接近自动生成训练任务的平台，现成家庭长程基准的完备程度需要逐项看。[RoboGen 论文](https://arxiv.org/abs/2311.01455)、[官方仓库说明](https://github.com/Genesis-Embodied-AI/RoboGen) |
| **Gym-PushT** | ENPIRE | 二维平面接触任务：控制一个推杆，把 T 形物体推动到指定位置和朝向。环境简单，但存在推点、摩擦和旋转耦合，适合比较策略搜索、接触反馈和程序修正。ENPIRE 用它与真实 Push-T 实验对照。[ENPIRE §3.1](https://arxiv.org/html/2606.19980)、[环境仓库](https://github.com/huggingface/gym-pusht) |

这里的 **Isaac Gym 与 Isaac Sim 应分别理解**：上述奖励学习论文主要利用前者的并行训练能力；OmniGibson 使用后者组织复杂家庭仿真。

GaP 还有一处实现边界值得记下：论文及项目介绍提到了 Isaac／Newton 等仿真排练，而当前公开文档的可运行路径主要是 **LIBERO/MuJoCo**，其版本说明将部分 Isaac 排练和双臂能力列为后续恢复内容。复现选型应以具体公开版本为准。[GaP 官方文档](https://graph-robots.github.io/graph-as-policy/getting-started/overview.html)

**还有一类是家庭导航、语义交互和游戏环境。它们也出现在这条引用链里。**

| 环境 | 引用链中的工作 | 环境介绍 |
|---|---|---|
| **Habitat / Housekeep** | ELLM：*Guiding Pretraining in RL with LLMs* | Housekeep 要求 agent 探索家庭，识别放错位置的物体，再按常识整理到合适位置。底层使用 Habitat，任务结合导航、寻找和重新摆放。**ELLM 实际使用了简化版本：4 个单房间场景，每个有 5 个放错位置的物体。** 对空间记忆、搜索历史和整理进度很有参考价值。[ELLM §4.2](https://arxiv.org/abs/2302.06692)、[Housekeep 官方实现](https://github.com/yashkant/housekeep) |
| **VirtualHome** | ProgPrompt | 通过动作程序执行家庭活动，例如移动、拿取、开关电器和容器。提供 Unity 可视化仿真，以及直接更新环境图的 Evolving Graph 模式。研究重点通常是动作前置条件、步骤顺序和执行反馈；在常用接口下，抓取轨迹与接触控制已被封装。[官方介绍](https://github.com/xavierpuigf/virtualhome)、[ProgPrompt](https://arxiv.org/abs/2209.11302) |
| **Minecraft** | Voyager、VistaWise | 开放式三维游戏，包含探索、采集、制作、资源依赖和技能积累，天然具有长任务链。**Voyager 使用 MineDojo 相关环境基础与 Mineflayer JavaScript API；VistaWise 则使用原版桌面客户端、截图和鼠标键盘控制。** 两者的信息和动作接口差别很大。它适合研究开放式技能记忆，机械臂操作的物理细节则不在其建模范围内。[Voyager](https://arxiv.org/abs/2305.16291)、[VistaWise §3.4、§4.1](https://aclanthology.org/2025.emnlp-main.1111/) |
| **Crafter** | ELLM | 二维生存与制作游戏，包含采集、工具制作、生存和具有前后依赖的成就。运行成本低，适合研究探索、子目标选择及技能积累。[ELLM](https://arxiv.org/abs/2302.06692)、[Crafter 官方介绍](https://github.com/danijar/crafter) |
| **BabyAI / MiniGrid** | HYVIN：*Hypothesis, Verification, and Induction* | 部分可观测的网格世界，执行找物、开门、取钥匙、搬运和组合语言指令。规则清楚，便于研究指令组合、技能抽象和记忆机制；物理交互高度抽象。[HYVIN](https://ojs.aaai.org/index.php/AAAI/article/view/29376)、[BabyAI 文档](https://minigrid.farama.org/environments/babyai/) |
| **ALFWorld** | CodeAct | 把家庭任务组织成寻找、打开、取放、清洗、加热等交互。ALFWorld 有文字环境与对应具身环境，**CodeAct 使用的是纯文本版本，通过 Python 包装动作 API**。适合研究高层计划和搜索记忆，感知与机械执行问题被大幅简化。[CodeAct §3](https://arxiv.org/html/2402.01030v4)、[ALFWorld 官方仓库](https://github.com/alfworld/alfworld) |
| **ScienceWorld** | CodeAct | 文字式科学实验环境，agent 操作物体、设备并完成实验任务。CodeAct 将其用于跨动作格式的泛化评估。它有长步骤和状态依赖，适合作为抽象 agent 对照环境。[CodeAct §3.2](https://arxiv.org/html/2402.01030v4)、[ScienceWorld](https://github.com/allenai/ScienceWorld) |
| **LunarLander-v3 / Box2D** | Code Evolution for Control | 二维登月器控制，通过发动机动作调整位置、速度和姿态，实现安全着陆。这篇工作使用离散动作版本，适合检验代码策略搜索，对家庭长程操作的直接参考较少。[论文](https://arxiv.org/abs/2601.06845)、[环境文档](https://gymnasium.farama.org/environments/box2d/lunar_lander/) |

另外，RoboCodeX 导航实验中出现的 **HM3D、HSSD 是室内场景数据集**，应与承载它们的仿真平台分开记录。[RoboCodeX §4.2](https://arxiv.org/abs/2402.16117)

**结合我们要研究的“实际操作、长程任务、高层历史管理”，我的选择是：**

1. **新增环境优先考虑 RoboCasa365。** 它提供大量现成组合任务，ENPIRE 又展示了在其中调用视觉、抓取和运动规划工具的路径。更直接的是，官网在 **2026 年 10 月 28 日**开放了 agentic leaderboard，明确接受围绕基础策略加入规划、记忆、重试和工具调用的系统。[官方入口](https://robocasa.ai/leaderboard/agentic.html)
2. **需要整屋搜索、跨位置物体关系和复杂状态变化时，优先 BEHAVIOR-1K。** 我们应从具体任务中找出需要保留、更新和纠正的历史事实，任务选择可以超出 ASPIRE 的两个导航抓取案例。
3. **需要把问题做得可控时，选 LIBERO-Pro、RLBench 或 robosuite。** 它们更适合先研究某类失败、状态更新或技能切换，再扩展到完整家庭任务。

对历史压缩而言，我最看重的环境条件是：**过去获得的信息在当前画面中已经消失，但仍会影响后续动作；错误行动还会改变世界，迫使 agent 更新旧判断。** RoboCasa 和 BEHAVIOR 提供了较丰富的构造空间，最终仍需落实到具体任务和观测接口。
