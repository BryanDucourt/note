# 上下文骨架会审：Gemini 第一轮原文（1028）

conversationId：`0521c925-b3ab-4061-abf1-a4902274815e`，模型 gemini-4-argon-high。问题见 [[1028-上下文骨架会审-问题]]。以下是原文，只把代码链接里的本机路径保留原样，没有其他改动。其中两条说法经核对不成立，见 [[1028-上下文骨架会审-对比]]。

---

基于对代码库（`models.py`、`context.py`、`tools.py`、`check_conditions.py`、`task_planning.py`、`navigation_guard.py`、`planner_projection.py`、`progress.py`、`task_sequence.py`）和离线探针数据的审查，直接给出独立方案与问题诊断。

### 1. 系统提示：包含什么、怎么组织、去留清单

**现状诊断**：
`planner_system`（models.py#L542-L603）采用 7 层字符串 `.replace()` 切片拼接，存在直接的代码级缺陷：
- **死规则残留**：`_without_manipulation_guidance`（models.py#L524-L539）只切掉了 87～88 行，却把 `PLANNER_SYSTEM`（models.py#L89-L92）中 `Only choose vla_interact when interaction_targets...` 等 4 句留在了禁用 VLA 的提示词里。
- **提示与输入冲突**：`paper1_aba_1022`（models.py#L571）硬编码了 `allows at most 15 decisions`，与 THOR 输入里的 `decisions_per_stage: 7` 直接冲突。
- **把代码内部过滤逻辑塞进提示词**：如 `SPLIT_PLANNER_GUIDANCE`（models.py#L490-L514）花大段解释 `condition_filter_notice` 和 `latest_progress` 更新条件，对选动作毫无帮助。

**重写方案（总长压至 1,500 字符内，按模板参数生成，严禁再用 `.replace()` 叠补丁）**：
按以下 4 块顺序组织：
1. **角色与完成判定（置顶）**：说明任务是找 `current_target` 并使其出现在正前方头部图（`head_000`）以达成 `G2`。程序在 `G2=satisfied` 时自动换阶段或结束。明确写出三条决策准则：
   - **准则 A（G1 满足但 G2 未满足）**：目标已在某张 `scan` 环视图中出现，下一步必须直接按该图提示的 `angle_degrees` 调用 `rotate_base` 对准，或对该图上的目标调用 `move_to_point`。
   - **准则 B（目标可见但远/模糊）**：调用 `move_to_point` 靠近。
   - **准则 C（当前未见目标）**：优先查 `memory.object`（是否之前见过）和 `memory.spatial`（候选房间/未探视点），用 `scan` 环视或 `move_to_point` 穿过可见通道。
2. **工具边界与坐标系**：6 个工具的适用边界（`move_to_point` 主力接近但不绕障、`move_base` 仅限 ≤0.5m 微调、`scan` 原地环视）。明确 `point_xy=[x,y]`（右 x 下 y，0～1）与 `rotate_base`（**左正右负**）。
3. **证据与防死锁原则**：浓缩为 3 条：① 区分当前视觉事实与历史记忆/假设，门洞不等于已确认门后房间；② 阶段切换后旧阶段的 G1/G2 不算数，但历史 Object/Spatial 记忆仍有效；③ 连续 2 步无进展必须换方向或换区域，禁止原地小角度来回转或空调 `observe`。
4. **输出格式**：4 字段 JSON 定义及单行示例。

- **必须保留**：G1/G2 语义差及应对动作、6 个工具的物理限制、左正右负与图像坐标约定、防重复探索原则。
- **必须去掉**：① 所有 `vla_interact`/`move_ee`/`interaction_targets`/抓握判定残留；② 硬编码的阶段步数（改由输入读取）；③ `check_conditions` 和 `search_plan` 的填表与过滤规则；④ `evidence_group_ids`、里程计漂移、相机内外参说明；⑤ 旧 `navigation_guard="legacy"` 的 8°/0.3m 规则。

---

### 2. 输入结构：分块、顺序与独立记忆给法

**现状诊断**：
`ContextStore.packet`（context.py#L463-L525）把“科研审计存储结构”原样喂给了规划器：每张图带 3×3 `intrinsics` 和 4×4 `camera_pose` 矩阵（仅 `navigation_guard.py`（#L28-L46）算射线需要，LLM 根本不会算矩阵乘法）；`proprio_summary`（robot_state.py#L6-L45）塞入双臂位置和四元数；`"Dead reckoning drifts..."` 重复 10 次。探针把输入砍到 13% 导致准确率下降，是因为粗暴删减破坏了上下文连贯性，而非模型需要这 50% 的矩阵和审计哈希。

**重构方案（在 `planner_projection.py` 中做视图投影，审计仍存全量，给模型的 JSON 从 22 个顶层字段压为 5 块，控制在 4,000～6,000 字符）**：

1. **`task_state`（置顶，因为探针证明决策第一依赖完成信号）**：
   - `overall_goal`, `current_stage`（如 `"2/3"`）, `decisions_remaining_in_stage`（消除步数冲突）；
   - `current_target`: `{"id", "name", "aliases"}`；
   - `G1_any_head_image`: `{"status": "satisfied|not_satisfied|...", "matched_refs": [...], "bbox": [...]}`；
   - `G2_front_head_image`: `{"status": "satisfied|not_satisfied|...", "ref": "..."}`；
   - **删除顶层 `current_subgoal`**（原代码把模型上一步自写的 `subgoal` 放在顶层，换阶段还不清空，极易造成跨阶段目标混淆）。
2. **`current_observation`**：
   - `step`, `observation_id`, `odom_xy_yaw_deg`: `[x_m, y_m, yaw_deg]`（保留 1 位小数，删掉弧度与免责声明）；
   - `images`: 仅保留 `{"ref", "is_front_view": true/false, "rotate_base_to_face_deg": -144.5}`。**彻底剥离** `intrinsics`、`camera_pose` 4×4 矩阵、`sha256`、`robot_state` 双臂状态。
3. **`recent_history`（合并 `before`, `recent_tools`, `execution_history_summary`）**：
   - `recent_actions`（最近 3 步）：仅留 `{"step", "tool", "arguments", "status", "stop_reason", "measured_turn_deg", "measured_dist_m", "subgoal"}`；
   - `progress`: 仅留 `steps_without_progress`、`same_tool_streak`、`stationary_observed_headings_deg`。
4. **`memory`（严格维持四库独立、仅共享原始图片 `evidence_refs` 的硬约束）**：
   - **剥离审计外壳**：去掉每条记录里的 `provenance`、`record_id`、`evidence_group_ids`、`supersedes`，这些留给后台审计，不进规划器 prompt。
   - `task`: 仅放任务级未决信息（G1/G2 已提至 `task_state`，不再重复）。
   - `object`: 0～4 条，保留 `{"key", "category", "text", "visibility", "basis", "assessment", "evidence_refs", "last_seen": {"step", "evidence_ref", "bbox_normalized"}}`。在 A→B→A 下，`planner_limits`（input_limits.py#L80-L84）必须优先置顶匹配**当前阶段目标（`current_target`）**的 `last_seen_by_target` 记录，而不是随便置顶任意最新物体！
   - `spatial`: 将原顶层 `spatial_memory` 收拢为 `memory.spatial` 的只读索引子字段（因为 `SpatialMemory.annotate`（spatial_memory.py#L86-L120）本就严格只消费 Spatial 记录），保留 `records`（1～4 条）、`known_rooms`、`retrieved_viewpoints`（仅留 `viewpoint_id`、`step`、`sightings`、`return_offset_xy_m`、`return_turn_deg`，删掉冗长的 `note` 和 `geometric_rejoin_tolerance`）。
   - `experience`: `[]`。
5. **`available_tools`**：仅列当前启用的 6 个工具签名。

---

### 3. 输出字段：顺序与 `search_plan`、`check_conditions` 处理

**现状诊断**：
1. 关闭思考时，先输出 `tool`/`arguments` 再输出 `reason`，迫使自回归模型在未做视觉/角度推理前先盲猜动作参数，导致 THOR 转对率仅 37%。
2. `search_plan` 占了 26 条校验规则（`parse_search_plan`，task_planning.py#L24-L76），思考模式下 9 次拒收全死在这里，且回传为 `last_search_plan` 持续污染上下文。
3. `check_conditions` 80% 都在写空洞的 `visibility_improved`，只为给 `confirmed_progress`（progress.py#L10-L15）提供布尔值，却引入了 6 条校验和判断模块开销。

**具体方案**：
1. **从规划器输出中彻底移除 `search_plan` 和 `check_conditions`**：
   - **替代 `search_plan`**：在 `reason` 之后放一个轻量字符串字段 `target_region`（如 `"kitchen_counter_left"`），保留“先选区域再选动作”的引导，0 复杂嵌套校验。
   - **替代 `check_conditions`**：`confirmed_progress` 改由程序确定性判定（例如：本步 G1/G2 改善、或移动距离 ≥0.2m、或转向到了新的未观测角度、或 Object/Spatial 新增了 `supported` 观察记录）。如果实验架构必须保留判断模块调用，由程序根据动作类型自动生成默认检查条件，不再让规划器手写归一化 `bbox`。
2. **输出固定为 4 个字段，严格按以下顺序**：

```json
{
  "reason": "1) 核对G1/G2与目标所在图片；2) 结合记忆选定目标区域；3) 确定动作与直接引用的角度/坐标",
  "subgoal": "当前一步的具体意图（如：转向正对 head_003 中的目标刀）",
  "tool": "rotate_base",
  "arguments": {"angle_degrees": -144.5}
}
```

把 `reason` 放在第一字段，用 2～3 句话充当受控的轻量级 CoT，既有思考模式的准确率，又避免思考模式 10 倍 token 和 `search_plan` 拒收。

---

### 4. 朝向和转向表示：如何彻底消灭换算错误

**探针败因剖析**：
- BEHAVIOR 把 `215.5°` 算成“左转 144°”：因为给的是 `0～360°`（左正），模型算对了 `360 - 215.5 = 144.5`，但在换算瞬间漏掉了“超过 180° 符号变负（右转）”。
- 0010 探针写了 `"-144.5° (144.5° to the right)"` 仍转 `+144°`：因为 `models.py`（#L310）原图片说明写着 `left positive`，且正数 `144.5°` 与英文 `right` 并列时，无思考的 Flash 模型直接提取了正数 `144.5`！

**具体方案（让模型做“参数复制”，禁止做“几何心算”）**：
1. **全局废除 `0～360°` 和弧度（`rad`）**：输入 JSON 和图片说明中，所有角度统一由程序用 `wrap_angle` 预计算为 **`[-180°, +180°]` 的有符号角度**。删掉所有 `planned_heading_degrees`（0～360°）和 `rotation_rad`。
2. **图片说明直接写成与 `rotate_base` 参数同构的键值形式**（修改 `ChatClient.request`，models.py#L309-L311）：
   - 不要同时出现 `144.5°`（无符号）和 `to the right`！
   - 正前方图（`head_000`）写：`CURRENT step 12, ref .../head_000.png | FRONT VIEW (0.0 deg, used for G2)`
   - 环视图写：`CURRENT step 12, ref .../head_003.png | SCAN VIEW (to face this view: rotate_base {"angle_degrees": -144.5})`
3. **提示词明确引导优先用 `move_to_point`**：若环视图中已看清目标且距离 >0.5m，直接对该环视图 `ref` 调用 `move_to_point(evidence_ref, point_xy)`，由底层代码自动转向对准并前进，完全绕开角度正负号问题。

---

### 5. 验证方案与最大风险

1. **基准决策点选取（40～60 个冻结决策点，分 4 个诊断桶）**：
   - **桶 1（G1=真, G2=假，约 15 点）**：检验环视看到目标后，转向或 `move_to_point` 的方向准确率（核心看那 13 个曾把 `>180°` 转反的点和 0010 点）。
   - **桶 2（A→B→A 换阶段首步及回溯步，约 15 点）**：检验阶段切换时是否还会受旧 subgoal 干扰，以及找回 A 时能否利用 Object/Spatial 记忆直接回去而非盲目重搜。
   - **桶 3（连续无进展/局部卡死点，约 15 点）**：检验能否跳出重复动作。
   - **桶 4（常规探索与 G1=假，约 10 点）**：检验有無退化。
2. **对比基线与消融节奏（遵守“每次只改一处”，同批交错跑 × 每点 5 次）**：
   - `Step 1`：仅改角度表示为 `[-180, +180]` + 同构图片说明（验证问题 4，预期解决 BEHAVIOR 理由对但方向反的问题）；
   - `Step 2`：去掉 `search_plan`/`check_conditions`，输出改为 4 字段且 `reason` 置顶（验证问题 3，预期解决 THOR 转对率低和拒收问题）；
   - `Step 3`：上线输入 JSON 瘦身投影 + 新系统提示词（验证问题 1 & 2）；
   - `Step 4（核心研究消融）`：在 Step 3 上分别关掉 `memory.object`、`memory.spatial`，直接量化独立记忆对决策的净贡献。
3. **什么算更好**：
   - 桶 1 方向正确率 ≥95%（BEHAVIOR 与 THOR 双平台达标）；
   - JSON 解析与校验一次通过率 ≥99%（0 拒收）；
   - 同点 5 次采样的一致率显著提升（消除摇摆）；输入 token 降幅 >80%。
4. **最大风险**：
   - **离线冻结上下文与在线闭环的状态失配**：离线回放点里的 `recent_tools` 和 `memory` 是旧版生成的；且砍掉 `check_conditions` 后，若在线环境的进展计数逻辑未同步替换，在线跑长程任务时防死锁触发时机可能改变。因此每步离线探针过关后，必须跑小规模（如 5 个 episode）在线闭环冒烟测试。

---

### 6. 最大的问题与可能没看到的盲点

**最大的问题**：
**把“防作弊审计数据库（Audit Store）”直接当成了“规划器的工作记忆（Working Prompt）”**，再用 118 句补丁提示词和 56 条校验规则去强迫一个不带思考的 Flash 模型在 5.5 万字符的矩阵、哈希和免责声明里做空间几何心算与复杂填表。

**你们可能没看到的三个盲点**：
1. **A→B→A 换阶段时的状态污染（代码级 Bug）**：
   查看 `ContextStore.record_call`（context.py#L348）和 `TaskSequence.assess`（task_sequence.py#L60-L75）：当阶段 1（找 A）满足 G2 自动切到阶段 2（找 B）时，`planner_projection.py`（#L45-L47）虽然清空了 `goal_condition`，但**顶层的 `current_subgoal`、`last_search_plan` 以及 `execution_summary`（progress.py#L46-L56）的连续无进展计数完全没有按阶段重置**！在阶段 2 的第 1 步，规划器看到的 `current_subgoal` 依然是阶段 1 最后一步写的“找 A”。
2. **研究归因混淆：程序的 `spatial_memory` 遮蔽了模型生成的 `memory.spatial`**：
   你们的核心研究问题是“系统自己生成的记忆兑现了多少”。但在 `context.py`（#L510-L517）中，只要开启 `spatial`，就会捆绑注入 `SpatialMemory.context`（spatial_memory.py#L177-L273）生成的里程计回溯路由（`return_route`）和历史图片（`historical_observations`）。如果不把“LLM 写的语义记录”和“程序算的里程计路径/历史图检索”拆开消融，审稿人很容易质疑：所谓的空间记忆价值，到底来自 LLM 记忆模块，还是来自程序的里程计计算？
3. **为什么 `temperature=0` 下同一请求还会在几个方向间来回变？**
   GPU 浮点非确定性只有在**两个候选 token 的 logit 极其接近（模型处于困惑的决策边界）**时才会翻转输出。是什么让模型处于边界？正是因为输入里同时存在互相打架的信号（如图片说明里既有 `-144.5` 又有 `144.5 to the right` 又有 `left positive`；提示词说 15 步而 JSON 说 7 步；5.5 万字符稀释了注意力权重）。消除一词多表和指令冲突，是收敛 `temperature=0` 采样方差的根本手段。
