说明：之前上传的开题报告/论文 PDF 等文件在当前会话中已经过期；下面只基于 **GitHub 仓库** **`5f4f4bc`** **最新提交**、当前 PTS 配置、以及你明确的 M1/M2/M3 motivation 重新规划。

---

## **1. 当前进展检查**

## **1.1 最新提交概况**

你指定的最新提交是：

```text
5f4f4bce89ad55a32deaecd1523d0482c201aba7
message: track generated run artifacts
```

这个提交相对上一版新增了几类内容：两个 collector smoke run、一次 `pts/c-ray` paired run 的 `run_feature_summary.csv`，以及 `tables/c_ray_once_dataset.csv`；同时 `.gitignore` 中对 `results/`、`runs/`、`figures/` 的忽略规则被移除。

  

这说明当前仓库已经从“只有 raw paired run artifact”推进到：

```text
collector window features
→ active-window aggregation
→ run-level feature summary
→ run-level dataset
```

这条链路已经局部跑通。

---

## **1.2 当前已经完成的关键能力**

### **A. PTS pilot matrix 已经存在**

`configs/pts_pilot_matrix.yaml` 已经列出 6 个 PTS pilot scenarios：

```text
apache_concurrency_100
nginx_connections_100
c_ray_1080p
hackbench_thread_count_8
sysbench_cpu
sysbench_ram
```

并明确记录了 PTS 正常 `TimesToRun=3`，`FORCE_TIMES_TO_RUN=1` 只用于 smoke validation。   

### **B. M2 policy metadata 已经更新**

`configs/policies.yaml` 已经升级到 schema v2，并定义了：

```text
baseline
batch_manual
affinity_compact_manual
affinity_spread_manual
fifo_manual_later
rr_manual_later
```

其中非 baseline 策略都标记为 `dangerous: true` 和 `manual_only: true`，符合当前安全边界。 

### **C. Paired-run checker 已经实现**

`analysis/check_paired_run.py` 已经存在，会检查 benchmark 和 collector 的 `run_id`、`policy`、`workload`，检查 `benchmark/run_summary.csv` 和 `collectors/window_features.csv`，并允许 PTS run-level objective 没有 window-level app metrics。 

### **D. Active-window aggregation 已经实现**

`analysis/aggregate_window_features.py` 已经实现，会读取 schema v2 `window_features.csv`，按 `run_id,cgroup` 分组，识别 active windows，并输出 all-window 与 active-window summary。它默认根据 `cpu_util >= threshold` 或 PMU 数值是否存在判定 active window。   

### **E. Run-level dataset builder 已经实现**

`analysis/build_run_level_dataset.py` 已经存在，可以从 paired run 目录中读取：

```text
benchmark/run_summary.csv
collectors/run_feature_summary.csv
benchmark/run_meta.json
collectors/run_meta.json
```

然后输出 run-level rows。 

### 

### 

### **F. 当前**

**`pts-c-ray`**

**run 已经生成 run-level summary 和 dataset**

`run_feature_summary.csv` 显示当前 c-ray run：

```text
total_window_count = 2401
active_window_count = 2124
active_duration_sec = 212.4
active_fraction ≈ 0.8846
active_cpu_util_mean ≈ 7.776
```

`tables/c_ray_once_dataset.csv` 已经把 `pts_score=208.525 Seconds, higher_is_better=false` 与 active feature summaries 合并到一行 dataset 中。 

这一步很重要：它说明 **M2/M3 所需的 run-level 数据结构已经能生成**。

---

## **1.3 当前仍未完成的关键点**

### 

### 

### **A.**

**`run_m2.py`**

**仍不是 paired-run planner**

当前 `runners/run_m2.py` 仍只是 offline analysis planner，只会打印 `analysis/m2_policy_matrix.py` 的调用命令。它还不能根据 `configs/pts_pilot_matrix.yaml` 生成 scenario × policy 的 paired-run 计划。 

### **B. M2 analysis 仍未完全适配 run-level dataset**

`analysis/m2_policy_matrix.py` 已经能读 `run_summary.csv` 或 objective table，但当前接口还是 `--run-summary / --objective-table / --app-metrics`，尚未面向 `tables/m2_dataset.csv` 这种 run-level paired dataset 输出 heatmap/regret/oracle policy 全套产物。 

### **C. M3 run-level analysis 还没有**

当前 `analysis/m3_feature_policy.py` 仍是 window-level feature-objective correlation，不是基于 `run_feature_summary.csv + oracle_policy.csv` 的 run-level feature-scenario / feature-policy analysis。 

### **D. M1 的 p99 latency path 还没有**

M1 当前仍缺：

```text
wrk/hey/siege parser
M1 web latency matrix
M1 run directory analysis
cpu_util_norm 处理
```

---

## **2. 完整实验计划：按先后顺序执行**

下面不按“我执行 / Codex 执行”分组，而是按**执行顺序**列出，每一步标明执行者。

---

## **Step 1：拉取最新仓库并确认当前状态**

**执行者：你**

```bash
cd ~/experiments/featsched-motivation
git pull
git checkout -b complete-m1-m2-m3-experiments
git log --oneline -5
```

确认能看到：

```text
5f4f4bc track generated run artifacts
1c419e5 add pts pilot matrix policies
```

运行测试：

```bash
pytest -q
```

---

## **Step 2：检查当前 c-ray paired run 是否完整**

**执行者：你**

```bash
RUN_ROOT=runs/pts-c-ray-once-paired-20260609-02

python3 analysis/check_benchmark_run.py \
  --run-dir "$RUN_ROOT/benchmark" \
  --allow-benchmark-normalization

python3 analysis/check_run_integrity.py \
  --run-dir "$RUN_ROOT/collectors" \
  --expect-raw all

python3 analysis/check_paired_run.py \
  --run-root "$RUN_ROOT" \
  --allow-workload-alias pts_c_ray_cpu_once=pts_c_ray_cpu_once
```

如果 `--allow-benchmark-normalization` 不存在或失败，优先修 Phoronix benchmark ID 一致性。

---

## **Step 3：修 Phoronix benchmark ID 一致性**

**执行者：Codex**

Prompt：

```text
Fix benchmark ID consistency for Phoronix parsed outputs.

Current issue:
Historical run_summary.csv may contain benchmark "pts/c_ray" while benchmark/run_meta.json uses "pts/c-ray".

Tasks:
1. Inspect analysis/parse_phoronix_result.py.
2. Ensure future parse_phoronix_result.py writes the benchmark argument exactly as provided, e.g. "pts/c-ray".
3. Do not slug or sanitize the benchmark column in run_summary.csv.
4. Keep analysis/check_benchmark_run.py strict by default.
5. Add optional --allow-benchmark-normalization only for historical data.
6. Add regression tests:
   - input benchmark argument "pts/c-ray"
   - output run_summary.csv benchmark must be "pts/c-ray"
   - check_benchmark_run.py passes strict mode on new output.
7. Do not rewrite existing real artifacts automatically.
```

验收：

  

**执行者：你**

```bash
pytest -q

python3 analysis/check_benchmark_run.py \
  --run-dir runs/pts-c-ray-once-paired-20260609-02/benchmark \
  --allow-benchmark-normalization
```

---

## **Step 4：验证当前 c-ray run-level dataset**

**执行者：你**

```bash
RUN_ROOT=runs/pts-c-ray-once-paired-20260609-02

python3 analysis/aggregate_window_features.py \
  --input "$RUN_ROOT/collectors/window_features.csv" \
  --output "$RUN_ROOT/collectors/run_feature_summary.csv" \
  --active-cpu-util-threshold 1.0

python3 analysis/build_run_level_dataset.py \
  --run-root "$RUN_ROOT" \
  --output tables/c_ray_once_dataset.csv

cat tables/c_ray_once_dataset.csv
```

验收标准：

```text
1. active_window_count > 0
2. active_duration_sec > 0
3. metric_value = 208.525
4. higher_is_better = false
5. active_cpu_util_mean、active_ipc_mean、active_cache_miss_rate_mean 等字段存在
```

当前仓库已经有该 dataset，内容显示 active window 数量、active duration、active CPU/PMU/sched summaries 已生成。 

---


## **Step 5：把**

**`run_m2.py`**

**改成 paired-run planner**

**执行者：Codex**

当前 `run_m2.py` 还只是 offline analysis planner。  需要让它读取 `configs/pts_pilot_matrix.yaml` 和 `configs/policies.yaml`，生成完整的 run plan。

Prompt：

```text
Refactor runners/run_m2.py into a PTS paired-run planner.

Inputs:
- configs/pts_pilot_matrix.yaml
- configs/policies.yaml

Required behavior:
1. Generate scenario × policy tuples.
2. Write planned_runs.csv with:
   run_id
   scenario
   workload_config
   pts_profile
   pts_options
   policy
   dangerous
   manual_only
   benchmark_dir
   collectors_dir
   objective_metric_hint
3. For every tuple, print:
   - collector command
   - benchmark command
   - PTS JSON export command
   - parse_phoronix_result command
   - aggregate_window_features command
   - check_paired_run command
   - build_run_level_dataset hint
4. Default is dry-run.
5. Do not execute PTS, sudo, chrt, taskset, cgroup mutation, or eBPF.
6. Support filters:
   --scenarios
   --policies
   --output-dir
7. Include PTS TimesToRun=3 note in planned_runs.csv.
8. If a policy is manual_only, emit the policy command template as instructions only.
9. Add tests verifying:
   - six scenarios are generated from configs/pts_pilot_matrix.yaml
   - selected policy filters work
   - dangerous/manual_only fields are preserved
```

验收：

  

**执行者：你**

```bash
pytest -q

python3 runners/run_m2.py \
  --matrix configs/pts_pilot_matrix.yaml \
  --policies baseline,batch_manual,affinity_compact_manual,affinity_spread_manual \
  --output-dir results/m2_plan \
  --dry-run

head results/m2_plan/planned_runs.csv
```

---

# **Step 6：执行 M2 baseline paired runs**

**执行者：你**

目标 scenarios 来自 `configs/pts_pilot_matrix.yaml`：

```text
apache_concurrency_100
nginx_connections_100
c_ray_1080p
hackbench_thread_count_8
sysbench_cpu
sysbench_ram
```

这些 scenario 已经在配置中定义。 

  

生成 baseline 计划：

```bash
python3 runners/run_m2.py \
  --matrix configs/pts_pilot_matrix.yaml \
  --policies baseline \
  --output-dir results/m2_baseline_plan \
  --dry-run
```

对每一行 planned run，手动执行：

```text
1. collector command
2. benchmark command
3. PTS JSON export command
4. parse_phoronix_result command
5. check_paired_run command
6. aggregate_window_features command
```

每个 run 完成后都要：

```bash
python3 analysis/check_paired_run.py --run-root <RUN_ROOT>

python3 analysis/aggregate_window_features.py \
  --input <RUN_ROOT>/collectors/window_features.csv \
  --output <RUN_ROOT>/collectors/run_feature_summary.csv \
  --active-cpu-util-threshold 1.0
```

---

# **Step 7：执行 M2 非默认策略 paired runs**

**执行者：你**

执行 policy：

```text
batch_manual
affinity_compact_manual
affinity_spread_manual
```

对应 policy metadata 已在 `configs/policies.yaml` 中定义。 

  

生成计划：

```bash
python3 runners/run_m2.py \
  --matrix configs/pts_pilot_matrix.yaml \
  --policies batch_manual,affinity_compact_manual,affinity_spread_manual \
  --output-dir results/m2_static_plan \
  --dry-run
```

执行要求：

| **Policy**                | **手工动作**                                                              |
| ------------------------- | --------------------------------------------------------------------- |
| `batch_manual`            | 对 benchmark 主进程执行 `chrt --batch --pid 0 <pid>`                        |
| `affinity_compact_manual` | 对 benchmark 主进程执行 `taskset --cpu-list <compact_cpu_list> --pid <pid>` |
| `affinity_spread_manual`  | 对 benchmark 主进程执行 `taskset --cpu-list <spread_cpu_list> --pid <pid>`  |

每次运行必须把实际手工命令记录到对应 `benchmark/run_meta.json`，至少包括：

```json
"manual_policy_commands": [
  "taskset --cpu-list 2-9 --pid 12345"
]
```

---

# **Step 8：构建完整 M2 dataset**

**执行者：你**

将所有 paired run 组织在一个目录，例如：

```text
runs/
  <run-id-1>/
    benchmark/
    collectors/
  <run-id-2>/
    benchmark/
    collectors/
```

构建 dataset：

```bash
python3 analysis/build_run_level_dataset.py \
  --results-dir runs \
  --output tables/m2_dataset.csv
```

检查：

```bash
python3 - <<'PY'
import pandas as pd
df = pd.read_csv("tables/m2_dataset.csv")
print(df[["run_id","workload","policy","benchmark","metric_name","metric_value","higher_is_better"]].head())
print(df.groupby(["workload","policy"]).size())
PY
```

验收：

```text
至少包含：
6 scenarios × 4 policies = 24 个 run-level rows
如果某个 PTS scenario 产生多个 metric，则会更多 rows
```

---

# **Step 9：完善 M2 analysis**

**执行者：Codex**

当前 `m2_policy_matrix.py` 尚未面向 `tables/m2_dataset.csv` 输出完整 heatmap/regret/oracle 文件。需要重构。

Prompt：

```text
Refactor analysis/m2_policy_matrix.py for run-level paired datasets.

Input:
--run-level-dataset tables/m2_dataset.csv

Requirements:
1. Read run-level dataset rows.
2. Allow selecting metric_name; default to pts_score when present.
3. Compute workload × policy mean metric_value.
4. Respect higher_is_better per metric.
5. normalized_score:
   higher_is_better: value / best_value
   lower_is_better: best_value / value
6. regret:
   higher_is_better: best_value / value - 1
   lower_is_better: value / best_value - 1
7. Output:
   tables/m2/objective_table.csv
   tables/m2/normalized_scores.csv
   tables/m2/oracle_policy.csv
   tables/m2/policy_regret.csv
   figures/m2/policy_heatmap.pdf
   figures/m2/policy_regret.pdf
8. Failed or incomplete runs must be excluded only through an explicit failed_runs.csv.
9. Add tests for:
   - lower-is-better metric
   - higher-is-better metric
   - missing policy
   - multiple workloads
```

验收：

  

**执行者：你**

```bash
pytest -q

python3 analysis/m2_policy_matrix.py \
  --run-level-dataset tables/m2_dataset.csv \
  --metric-name pts_score \
  --out-dir figures/m2 \
  --tables-dir tables/m2

ls tables/m2
ls figures/m2
```

M2 验收标准：

```text
1. 每个 workload/scenario 都有 policy score。
2. 每个 workload 的 best normalized_score = 1。
3. oracle_policy.csv 至少显示两个 workload 的 best policy 不同。
4. policy_regret.csv 中每个 static policy 都存在非零 regret。
```

---

# **Step 10：实现 M3 run-level analysis**

**执行者：Codex**

当前 `m3_feature_policy.py` 是 window-level correlation。  需要新增 run-level M3。

Prompt：

```text
Implement analysis/m3_run_level_analysis.py.

Input:
--dataset tables/m2_dataset.csv
--oracle tables/m2/oracle_policy.csv optional

Feature groups:
1. cpu_util_only:
   active_cpu_util_mean, active_cpu_util_p95, active_cpu_util_p99
2. system:
   active_cpu_util_*, active_cpu_pressure_some_*, active_cpu_pressure_full_*, active_fraction
3. sched:
   active_rq_lat_p95_us_*, active_rq_lat_p99_us_*,
   active_ctx_switch_rate_*, active_invol_switch_rate_*,
   active_migration_rate_*, active_wakeups_per_sec_*
4. pmu:
   active_ipc_*, active_cache_miss_rate_*, active_branch_miss_rate_*
5. all:
   all available active feature columns

Tasks:
1. Derive workload_class from workload/scenario names or an optional config map.
2. If oracle_policy.csv is provided, attach oracle_best_policy and near_best_policy_set.
3. Produce PCA by workload_class.
4. Produce feature group ablation for workload_class classification.
5. If oracle policy diversity is sufficient, produce best-policy / near-best policy prediction metrics.
6. Use grouped split by workload/scenario.
7. No random per-window split.
8. Print an exploratory warning if sample count is small.

Outputs:
- tables/m3/dataset_used.csv
- tables/m3/metrics.csv
- tables/m3/feature_importance.csv
- figures/m3/pca_by_workload_class.pdf
- figures/m3/feature_group_ablation.pdf
- figures/m3/decision_tree.pdf
- figures/m3/policy_prediction_regret.pdf if oracle is available
- docs/m3_rules.txt

Do not claim online adaptation.
```

验收：

  

**执行者：你**

```bash
pytest -q

python3 analysis/m3_run_level_analysis.py \
  --dataset tables/m2_dataset.csv \
  --oracle tables/m2/oracle_policy.csv \
  --out-dir figures/m3 \
  --tables-dir tables/m3

ls tables/m3
ls figures/m3
cat docs/m3_rules.txt
```

M3 验收标准：

```text
1. workload_class 可分。
2. all-features 优于 cpu_util_only。
3. decision tree 规则可解释。
4. 如果 M2 best policy 有多样性，则输出 near-best policy prediction。
```

---

# **Step 11：实现 M1 wrk parser**

**执行者：Codex**

M1 需要 p99 latency，不能使用 PTS apache/nginx 的 RPS 直接替代。当前 M1 配置已经以 `app_p99_us` 为目标。 

Prompt：

```text
Add M1 external latency support.

Create analysis/parse_wrk_output.py.

Input:
--input stdout.txt
--run-id
--ts-start-ns
--ts-end-ns
--cgroup foreground
--output app_metrics.csv

Parse wrk --latency output:
- Requests/sec -> app_throughput
- Latency Distribution 50%, 75%, 90%, 99% if present
- p50, p95 if available; if p95 absent, leave NaN
- p99 is required unless --allow-no-p99 is passed

Output schema:
run_id,ts_start_ns,ts_end_ns,cgroup,app_p50_us,app_p95_us,app_p99_us,app_throughput

Add tests with synthetic wrk output.
Do not run wrk automatically.
```

验收：

  

**执行者：你**

```bash
pytest -q

python3 analysis/parse_wrk_output.py --help
```

---

# **Step 12：新增 M1 web latency matrix**

**执行者：Codex**

Prompt：

```text
Add configs/m1_web_latency_matrix.yaml.

Scenarios:
- nginx_wrk_c20_none
- nginx_wrk_c100_none
- nginx_wrk_c500_none
- nginx_wrk_c100_c_ray_bg
- nginx_wrk_c100_hackbench_bg
- nginx_wrk_c100_sysbench_ram_bg
- nginx_wrk_c100_stress_cache_bg

Fields:
- foreground_service
- wrk_threads
- wrk_connections
- wrk_duration_sec
- background_workload
- collector_window_ms
- objective: app_p99_us
- policy: baseline

Notes:
- PTS apache/nginx RPS is not app_p99_us.
- app_p99_us must come from wrk/hey/siege or TailBench++.
- Use cpu_util_norm = cpu_util / allocated_cpu_count.
```

---

# **Step 13：执行 M1 web latency runs**

**执行者：你**

安装 wrk：

```bash
sudo apt install -y wrk
```

准备 nginx：

```bash
sudo apt install -y nginx
echo "ok" | sudo tee /var/www/html/test.html
curl http://127.0.0.1/test.html
```

单次 run 模板：

```bash
RUN_ID="m1-nginx-wrk-c100-none-$(date +%s)"
RUN_ROOT="results/m1/$RUN_ID"
COLLECT_OUT="$RUN_ROOT/collectors"
APP_OUT="$RUN_ROOT/app"

mkdir -p "$COLLECT_OUT" "$APP_OUT"
```

启动 collector：

```bash
scripts/run_collectors_concurrent.sh \
  --execute \
  --run-id "$RUN_ID" \
  --workload-label nginx_wrk_c100 \
  --policy-label baseline \
  --notes "M1 wrk latency run" \
  --pmu-scope system \
  --duration 120 \
  --window-ms 100 \
  --cgroup-path /sys/fs/cgroup/featsched/fg \
  --output-dir "$COLLECT_OUT"
```

运行 wrk：

```bash
START_NS=$(python3 - <<'PY'
import time
print(time.monotonic_ns())
PY
)

wrk --latency -t4 -c100 -d90s http://127.0.0.1/test.html \
  | tee "$APP_OUT/wrk_stdout.txt"

END_NS=$(python3 - <<'PY'
import time
print(time.monotonic_ns())
PY
)

python3 analysis/parse_wrk_output.py \
  --input "$APP_OUT/wrk_stdout.txt" \
  --run-id "$RUN_ID" \
  --ts-start-ns "$START_NS" \
  --ts-end-ns "$END_NS" \
  --cgroup /sys/fs/cgroup/featsched/fg \
  --output "$APP_OUT/app_metrics.csv"
```

对所有 M1 scenarios 重复。

---

# **Step 14：完善 M1 analysis**

**执行者：Codex**

当前 `m1_proxy_metric.py` 是 window-level analysis，但需要支持 results-dir、app interval、cpu_util_norm 和图表输出。

Prompt：

```text
Refactor analysis/m1_proxy_metric.py for real M1 run directories.

Requirements:
1. Support --results-dir containing multiple M1 run roots.
2. For each run root, load:
   collectors/window_features.csv
   app/app_metrics.csv
3. Support app_metrics covering a larger interval than 100ms windows:
   assign app_p99_us to windows whose ts_start_ns..ts_end_ns overlap app interval,
   or aggregate features over the app interval.
4. Add derived cpu_util_norm:
   cpu_util / allocated_cpu_count.
   allocated_cpu_count is passed as --allocated-cpus or inferred from run_meta if possible.
5. Use cpu_util_norm for CPU-utilization bucket analysis.
6. Outputs:
   tables/m1/correlation.csv
   tables/m1/bucket_summary.csv
   tables/m1/prediction_error.csv
   figures/m1/cpu_util_norm_vs_app_p99.pdf
   figures/m1/bucket_app_p99_boxplot.pdf
7. Fail if app_p99_us is missing or all NaN.
8. Do not use collector smoke runs.
9. Add tests with two synthetic M1 run roots.
```

验收：

  

**执行者：你**

```bash
pytest -q

python3 analysis/m1_proxy_metric.py \
  --results-dir results/m1 \
  --out-dir figures/m1 \
  --tables-dir tables/m1 \
  --allocated-cpus 8
```

M1 验收标准：

```text
1. app_p99_us 非 NaN。
2. 使用 cpu_util_norm，而不是 raw cpu_util。
3. 同 CPU utilization bucket 内，不同 background 的 app_p99_us 有明显差异。
4. all-features prediction error 低于 cpu-util-only。
```

---

# **Step 15：生成最终实验 manifest**

**执行者：Codex**

Prompt：

```text
Add scripts/make_manifest.py.

It should:
1. Recursively list files under results, runs, figures, tables, configs, docs.
2. Record file size and sha256.
3. Record git commit hash.
4. Record environment probe path if logs/env_probe.json exists.
5. Output artifacts/manifest.json.
6. Do not modify raw data.
```

验收：

  

**执行者：你**

```bash
mkdir -p artifacts

python3 scripts/make_manifest.py \
  --results results \
  --runs runs \
  --figures figures \
  --tables tables \
  --configs configs \
  --out artifacts/manifest.json

cat artifacts/manifest.json | head
```

---

# **Step 16：生成 Motivation 初稿**

**执行者：Codex**

Prompt：

```text
Draft the Motivation section from real outputs only.

Inputs:
- tables/m1, figures/m1
- tables/m2, figures/m2
- tables/m3, figures/m3

Structure:
1. M1: proxy metric mismatch
2. M2: static policy mismatch
3. M3: feature-policy correlation

Rules:
- Do not invent numbers.
- Do not use collector smoke runs as evidence.
- Do not claim online adaptive scheduling.
- Explicitly state:
  PTS run-level results are used for M2/M3.
  M1 p99 latency comes from wrk/hey/siege or TailBench++.
- Include limitations.
```

---

# **3. 最终实验矩阵**

## **M1 矩阵**

|**Scenario**|**Foreground**|**Load**|**Background**|**Objective**|
|---|---|---|---|---|
|nginx_wrk_c20_none|nginx|wrk c=20|none|p99 latency|
|nginx_wrk_c100_none|nginx|wrk c=100|none|p99 latency|
|nginx_wrk_c500_none|nginx|wrk c=500|none|p99 latency|
|nginx_wrk_c100_c_ray_bg|nginx|wrk c=100|c-ray|p99 latency|
|nginx_wrk_c100_hackbench_bg|nginx|wrk c=100|hackbench|p99 latency|
|nginx_wrk_c100_sysbench_ram_bg|nginx|wrk c=100|sysbench RAM|p99 latency|
|nginx_wrk_c100_stress_cache_bg|nginx|wrk c=100|stress-ng cache|p99 latency|

---

## **M2 矩阵**

|**Scenario**|**baseline**|**batch_manual**|**compact affinity**|**spread affinity**|
|---|---|---|---|---|
|apache_concurrency_100|✓|✓|✓|✓|
|nginx_connections_100|✓|✓|✓|✓|
|c_ray_1080p|✓|✓|✓|✓|
|hackbench_thread_count_8|✓|✓|✓|✓|
|sysbench_cpu|✓|✓|✓|✓|
|sysbench_ram|✓|✓|✓|✓|

---

## **M3 数据来源**

```text
M3 = M2 run-level dataset + M2 oracle policy
```

特征组：

```text
cpu_util_only
system
pmu
sched
all
```

标签：

```text
workload_class
oracle_best_policy
near_best_policy_set
```

---

# **4. 最终验收标准**

## **M1**

```text
app_p99_us 真实存在；
cpu_util_norm 用于 proxy 分析；
同 CPU bucket 下 app_p99_us 仍有差异；
all-features 比 cpu-util-only 更能解释 p99。
```

## **M2**

```text
至少 6 个 PTS scenarios；
至少 4 个 policies；
normalized score 计算方向正确；
oracle_policy.csv 显示不同 workload 的 best policy 不完全相同；
policy_regret.csv 显示 static policy regret。
```

## **M3**

```text
active-window features 被使用；
workload_class 可分；
all-features 优于 cpu_util_only；
如果 M2 oracle 有策略多样性，输出 near-best policy prediction；
不声称 online scheduling。
```

---

# **5. 现在应该继续的具体下一步**

你现在最直接执行：

```bash
git pull
git checkout -b complete-m1-m2-m3-experiments
pytest -q
```

然后按顺序推进：

```text
Step 2：检查当前 c-ray paired run
Step 3：Codex 修 benchmark ID 一致性
Step 5：Codex 改造 run_m2.py 为 paired-run planner
Step 6–8：你跑完整 M2 paired-run matrix
Step 9–10：Codex/你完成 M2 analysis
Step 11–14：Codex/你完成 M1 wrk latency path
Step 15–16：manifest 和 motivation 初稿
```

这套计划对应三条 motivation 的完整闭环：

```text
M1：单一 proxy metric 不足以解释 p99
M2：静态策略没有全局最优
M3：多层特征能区分场景并支撑策略选择
```