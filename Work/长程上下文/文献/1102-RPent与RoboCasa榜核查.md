# RPent 在 RoboCasa365 上的情况（1102 核查）

**用途**：顶层重新设计四步里的第 2 步（用户 1102："按这四步走"）。查 Harness VLA 的官方实现 RPent 对 RoboCasa365 的支持。

**来源**：
- RPent 仓库 [RLinf/RPent](https://github.com/RLinf/RPent)，main 分支 `0e406bb`，2026-10-30；
- 文档 `docs/source-en/rst_source/simulators/robocasa.rst`；代码 `robots/robocasa/`；榜单数据 `docs/_static/leaderboard/results.json`；
- 模型卡 [RLWRLD/RLDX-1-FT-RC365](https://huggingface.co/RLWRLD/RLDX-1-FT-RC365)；
- RoboCasa 的 [提交说明](https://github.com/robocasa-benchmark/leaderboard)；
- 1019 存下来的论文全文（`outputs/1019-harness-vla-next-round/text/`）。

## 一、是什么

- **RPent**：Harness VLA 的官方实现，Apache-2.0 许可。RoboCasa365 的接入在 `robots/robocasa/`，命令行写 `--robot robocasa`。
- **机器人和底层**：
  - 机器人是 PandaOmron；
  - 底层动作模型是 RLDX-1-FT-RC365：6.9B 参数，Qwen3-VL-8B 骨干，在 RoboCasa365 上微调过；模型卡说在 5090 上每步 43.7 ms。
  - 这个模型的许可是 RLWRLD Model License v1.0：非商用，要署名，衍生要按相同方式共享。
- **规划器**：支持三种接法，`api`、`claude_code`、`codex`。文档里的例子有 Codex 加 GPT-5.5（xhigh）、Claude Code 加 claude-opus-4-8，还有 GPT-6 Astra。
- **任务**（Target50）：

| 组 | 任务数 | 每个任务的种子 | 单次上限 | 次数 |
|---|---:|---|---|---:|
| 原子 | 18 | 1–10 | 30 分钟 | 180 |
| 见过的组合 | 16 | 1–5 | 60 分钟 | 80 |
| 没见过的组合 | 16 | 1–5 | 60 分钟 | 80 |
| 合计 | 50 | | | 340 |

  - 每次最多 100 轮规划。评测期间不重置环境。
  - 论文 v5 里，每个任务用 5 个种子。
- **记忆**：
  - 放在 HF 数据集 RLinf/RPent-memory 里，按规划器分版本，比如 `robocasa/GPT_5.5_xhigh/`；
  - 分两层：task-specific，是种子 0 上探索出来的成功记录和"配方"；global，是通用规则；
  - 探索模式：最多 3 个会话，每个会话最多 5 次尝试，探索完把记忆存下来。
- **运行环境**：
  - Linux、NVIDIA GPU、Python 3.10、CUDA 和 EGL，Torch ≥ 2.7；厨房资产约 10 GB。
  - VLA 可以单独起一个服务。

## 二、规划器看得到什么（关键）

- **观测**：相机图、深度和世界坐标，底盘、末端和夹爪的状态。
- **动作**：
  - 调 `rldx_skill`，也就是 VLA；
  - 运动原语：move_to、move_delta、rotate_pitch、set_gripper、release、scripted_grasp、move_base、navigate_to。
- **成功信号，规划器在回合中都看得到**：
  - `view_env_state` 返回 `success` 和 `robocasa_terminated`；
  - `task_progress`：在线执行一次官方的 `_check_success()`，把里面每一项的当前值交给规划器，包括子条件的真假、距离和计数（比如洗了多少秒）；
  - `dump_success_criteria`：把官方 `_check_success()` 的源码交给规划器。
- **评测提示的原话**："Check task_progress after every command to see if you are making progress." "Stop when state.success == True. Call finish(status='success', ...)."

**所以**：RPent 在 RoboCasa 上的分数，是在规划器看得到真值成功信号和子条件进度的情况下得到的。
- RoboCasa agentic 榜不允许这样："No ground-truth success signal during an episode"。
- 我们的规矩也不允许：仿真真值和分数不进任何模型输入。
- 1019 读论文时已经看到这一点（论文附录 A 表 8、附录 E 表 16），当时就计划"隐藏评分信号"。

## 三、公布的分数

来源：RPent 文档里的榜单，数据出自论文表 4 和贡献者的更正。

| 系统 | 总分 | 原子 | 见过的组合 | 没见过的组合 |
|---|---:|---:|---:|---:|
| RPent + GPT-6 Astra（high） | 59.2 | 87.8 | 43.8 | 42.5 |
| RPent + GPT-5.5（xhigh） | 57.1 | 92.0 | 61.0 | 13.8 |
| RPent + Opus-4.7（max） | 48.6 | 79.4 | 47.5 | 15.0 |
| Xiaomi-Robotics-1（官方口径） | 57.4 | 80.2 | 57.1 | 32.1 |
| WorldDreamer（只有策略） | 35.3 | 66.3 | 26.7 | 9.0 |
| RLDX-1（只有策略） | 30.0 | 60.0 | 21.3 | 5.0 |
| π0.5（只有策略） | 16.9 | 39.6 | 7.1 | 1.2 |
| π0（只有策略） | 14.8 | 34.6 | 6.1 | 1.1 |

- RPent 三行各是 340 次。
- 用时和花费，文档和论文里都没找到。

## 四、对我们意味着什么（只写事实）

1. **能直接拿来用**：代码是 Apache-2.0，RoboCasa365 的接入、任务、种子和评测脚本都有。
2. **要改了才能用**：按我们的规矩和 agentic 榜的规则，success、terminated、task_progress、success_criteria 都要藏起来。藏起来以后分数掉多少，没有人公布过。
3. **算力**：
   - RLDX-1 约 7B 参数，BF16 的权重约 14 GB；
   - 开发机的 5090 上，BEHAVIOR 常驻进程占约 17 GB（1030 的数），剩下的很紧；
   - VLA 可以单独起服务，放到共享的 A100 上。
4. **时间和成本**：一轮评测 340 次，单次最长 30–60 分钟；规划器用的是强模型加高推理档。要花多少时间和钱，要先小规模实测。
5. **许可**：RLDX-1 非商用；做研究可以，发布改过的权重受限。
