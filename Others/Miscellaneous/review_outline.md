# 复习提纲

## 1. 机器学习的统计学基础

- **最小错误率判别准则**：在所有可能的分类决策函数中，选择使总体错误率（概率错误分类率）最小的决策函数。
- **最小错误率贝叶斯判别准则**：对于每个样本 `x`，根据后验概率 `P(ω_i | x)` 选择使错误率最小的类别：
  $$
  决策 ω_k if k = argmax_i P(ω_i | x)
  $$
- **最大似然估计 (MLE)**：给定观测数据 `{x_n}` 和参数 `θ`，选择 `θ` 最大化似然函数：
  $$
  L(θ) = ∏_n p(x_n | θ)
  θ̂_MLE = argmax_θ L(θ)
  $$
- **最大后验估计 (MAP)**：在 MLE 基础上加入先验 `p(θ)`，最大化后验分布：
  $$
  θ̂_MAP = argmax_θ p(θ | {x_n}) = argmax_θ [p(θ) ∏_n p(x_n | θ)]
  $$
- **贝叶斯估计**：利用完整的后验分布进行预测或决策，常见形式包括后验均值（最小平方误差）和后验中位数（最小绝对误差）。
- **差别**：
  - MLE 忽略先验，仅依赖数据。
  - MAP 考虑先验，但仍为点估计。
  - 贝叶斯估计利用完整后验，更能量化不确定性。
- **离散特征的朴素贝叶斯分类器（PPT I‑1 例 1.1）**：
  $$
  P(ω_i | x) ∝ P(ω_i) ∏_j P(x_j | ω_i)
  $$
  参数由频率估计或加平滑的 MAP 估计得到。

## 2. 线性分类与线性回归

- **线性回归模型**：
  $$
  y = w^T x + b + ε
  $$
  其中 `ε` 为噪声。
- **参数学习的损失函数（均方误差）**：
  $$
  J(w,b) = (1/N) ∑_n (y_n - (w^T x_n + b))^2
  $$
- **逻辑回归模型**：
  $$
  P(y=1 | x) = σ(w^T x + b),  σ(z)=1/(1+e^{-z})
  $$
- **参数学习的损失函数（交叉熵）**：
  $$
  J(w,b) = - (1/N) ∑_n [y_n log p̂_n + (1-y_n) log (1-p̂_n)]
  $$

## 3. 支持向量机

- **最优超平面**：在特征空间中最大化两类样本的间隔。
- **原始优化问题**：
  $$
  min_{w,b}  (1/2) ||w||^2
  s.t.  y_n(w^T x_n + b) ≥ 1, ∀n
  $$
- **对偶优化问题**：
  $$
  max_α ∑_n α_n - (1/2) ∑_{n,m} α_n α_m y_n y_m x_n^T x_m
  s.t. α_n ≥ 0, ∑_n α_n y_n = 0
  $$
- **Hinge Loss（软间隔 SVM）**：
  $$
  J(w,b) = (1/2)||w||^2 + C ∑_n max(0, 1 - y_n(w^T x_n + b))
  $$
- **对偶参数 α 含义**：仅 α\_n > 0 的样本（支持向量）影响分类面，其余 α\_n = 0。
- **由对偶解计算 w, b**：
  $$
  w = ∑_n α_n y_n x_n,
  b = y_k - w^T x_k  （任一支持向量 x_k）
  $$

## 4. 神经网络和深度神经网络

- **前馈计算**：逐层运算
  $$
  z^(l) = W^(l) a^(l−1) + b^(l)
  a^(l) = φ(z^(l))
  $$
  （参见 PPT I‑4 第6–7页，I‑5 例5.2）
- **网络设计与学习方法**：
  1. 结构：层数、神经元数、激活函数
  2. 损失：回归用 MSE，分类用交叉熵
  3. 优化：SGD 及其变种（Adam、RMSProp）
  4. 预测：前馈后取输出或 argmax

## 5. 无监督学习

- **监督 vs 无监督 vs 半监督**：
  - 监督：有标签。
  - 无监督：无标签。
  - 半监督：标签与无标签混合。
- **k-means 算法（PPT I‑8 例8.1）**：
  1. 初始化 K 个质心。
  2. 赋类：按距离最近分配。
  3. 更新：质心设为簇内均值。
  4. 迭代直至收敛。
- **谱聚类**：
  1. 构造相似度矩阵 W。
  2. 计算度矩阵 D、拉普拉斯矩阵 L = D−W。
  3. 提取 L 的前 K 个最小特征向量，构成 U。
  4. 对 U 的行向量做 k-means。

## 6. 流形学习

- **流形假设**：高维数据位于低维流形上。
- **PCA（PPT I‑7 例7.1）**：
  1. 数据中心化。
  2. 计算协方差矩阵或 SVD。
  3. 取前 d 个特征向量构成映射矩阵 U。
  4. 降维： Z = X̃ U。
- **谱嵌入框架**：基于图拉普拉斯的低维表示（Laplacian Eigenmaps）。

## 7. 集成学习与决策树

- **决策树（CART）**：
  1. 计算每个特征分裂的信息增益或基尼系数减少。
  2. 选择最优分裂，递归继续。
- **Bagging**：
  1. 有放回抽取 B 个子样本集。
  2. 在每个子集上训练基学习器。
  3. 投票或平均。
- **随机森林**：在 Bagging 基础上，每次分裂随机选特征子集。

## 8. 生成模型

- **Transformer 结构**：多层自注意力 + 前馈网络，带残差连接与层归一化。
- **Encoder-only vs Decoder-only**：
  - Encoder-only（如 BERT）：理解任务。
  - Decoder-only（如 GPT）：生成任务。
- **图像生成模型**：
  1. **GAN**：生成器对抗判别器。
  2. **VAE**：变分推断 + 重构 + KL 正则。
  3. **Flow-based**：可逆变换 + 精确似然。
  4. **Diffusion**：正向加噪 ↔ 逆向去噪。

