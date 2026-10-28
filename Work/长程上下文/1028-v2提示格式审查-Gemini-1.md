# v2 提示和格式审查：Gemini 第一轮（1028）

conversationId：`aab23ef2-bee9-4b48-a269-07d9d294eaeb`，模型 gemini-4-argon-high。问题见 [[1028-v2提示格式审查-问题]]。原文很长，这里只记要点；完整回答在对话记录里。

## Gemini 提的 9 条（按它排的严重程度）

1. 历史图检索用目标名去匹配 Spatial 记录，几乎匹配不上，退化成取"最近的另一个视点"；系统提示的 Input 段也没提会附一张历史图。
2. `decisions_ago()` 跳过了脚本前缀，带前缀的运行里会出现自相矛盾的"0 decisions ago"。
3. 旧记录原样保留"visibility: visible""visible in the current view"等字样，容易被当成当前情况。
4. 程序视点里混进了当前视点；way_back 为空的重复视点占位置；重排提示里"places where the target was seen"对 Spatial 不成立，因为 Spatial 不记目标的目击。
5. 新阶段开始时 G1/G2 是 unassessed，提示没说目标已经在正前方时该 observe；G1.images 在三种状态下含义不统一。
6. BEHAVIOR 的 move_to_point 说"没有深度时必须给 distance_m"，可模型看不到深度，只能瞎猜；另建议加"受阻后别重复同一动作"。
7. 参数里值为 null 的键会导致拒收。
8. 浮点误差会写出"left 0.0°"；环视的 turned 两个平台口径相反。
9. reason 要求写三步，例子只写了两句；THOR 的 move_to_point 说明可能照搬了 BEHAVIOR 的"No routing"。

## Claude 的核对

| 条 | 结果 | 依据 |
|---|---|---|
| 1 | 基本属实，"几乎恒为 0"说重了 | Spatial 记录里提到目标名或别名的：BEHAVIOR 130/1923（6.8%），THOR 49/1364（3.6%）。样例的 5 张历史图里有 3 张是"2 decisions ago"，即当前和动作前之外最近的那个视点。Input 段确实没提历史图，是说明附录 A 漏写的。 |
| 2 | 属实 | BEHAVIOR 样例 3：obs_000000 和 obs_000001 都写"0 decisions ago"。 |
| 3 | 属实 | 6 个样例里有 4 个，带着 visible 或"current view"字样的 Spatial 记录，最后一次看到是 2～12 次决策之前。 |
| 4 | 属实 | 样例 1 的程序视点里有当前视点 obs_000013（0 decisions ago）；Spatial 角色本来就不看任务，不记目标。 |
| 5 | 前半条有道理 | 是机制说明里少了一句，不算补丁。 |
| 6 | 前半条属实，只涉及 BEHAVIOR | THOR 配置写的是"travel limit; default and maximum 1.5 m"，没有这个问题。后半条是针对具体情形的补丁，不建议加。 |
| 7 | 代码确实如此 | 6 个样例里没出现过；改起来是一行。 |
| 8 | 属实 | 样例里有"left 0.0° of start""turn left 0.0°"；BEHAVIOR 环视记 left 359.5°，THOR 记 0.0°。 |
| 9 | 前半条属实，后半条不成立 | THOR 配置的 move_to_point 写的是网格路线，没有照搬"No routing"。 |
