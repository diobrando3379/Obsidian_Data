## 模型结构：时空解耦

![[Pasted image 20260112225818.png]]
## 时序模块改进方法

使用transformer代替多尺度并行时间卷积块
可能可以参考 [CrossViVit](https://github.com/gitbooo/CrossViVit) 的时序部分？
![[Pasted image 20260112230102.png]]
需要去尝试这个方法

# fstgan_exp6_8: 可插模块与替换 MSTCN 的缝合方案

## 目标与插入位置（针对你当前 Block 结构）
- **插入位置**：把时序建模模块放在 `Block` 中原本 `MSTCN` 出现的位置（即 `self.tcn/self.tcn2/self.tcn3` 的位置）。这样能确保**Topo 注入 + A‑NHG** 后的特征再做时间建模，从而把拓扑条件传递到时序路径（和你当前“bn0 之后注入 topo_bias”的思路一致）。
- **数据形状**：你当前时序分支输入一般是 `(N, C, T, V)`。时序模块大多期望 `[(B, L, C) 或 (B, C, L)]`，建议**按关节维展开**进行“逐关节的时间建模”，即：
  - `x (N, C, T, V)` → `x.permute(0, 3, 2, 1).reshape(N * V, T, C)`（用于需要 `[B, L, C]` 的模块）。
  - `x (N, C, T, V)` → `x.permute(0, 3, 1, 2).reshape(N * V, C, T)`（用于需要 `[B, C, L]` 的模块）。

## 当前仓库中“可用于替换 MSTCN 的时序模块”
下面列出的模块都在当前仓库内，且天然是**序列/时间建模结构**，适合作为 MSTCN 的替代或混合路径。

1. **BiMamba2_1D**（长序列全局建模）
   - 模块位置：`布尔大学士模块/B站更新（和视频同步更新）/046 BiMamba2_1D.py`。
   - 结构是双向 Mamba2，输入为序列 `(B, L, C)`，适合处理长序列依赖。适合用来替换 `tcn3` 或整个 `tcn/tcn2/tcn3` 堆叠，以增强全局时间建模能力。【F:布尔大学士模块/B站更新（和视频同步更新）/046 BiMamba2_1D.py†L1-L229】

2. **MDM (Multi-Scale Decomposable Mixing Block)**（多尺度时间分解 + 混合）
   - 模块位置：`布尔大学士模块/B站更新（和视频同步更新）/101 Multi-Scale Decomposable Mixing Block（AAAI 2025）.py`。
   - 期望输入为 `[batch, feature_num, seq_len]`，适合做**多尺度时间分解 + 线性混合**。可替换 `tcn2` 作为“中段时间细化”。【F:布尔大学士模块/B站更新（和视频同步更新）/101 Multi-Scale Decomposable Mixing Block（AAAI 2025）.py†L1-L75】

3. **DDI (Dual Dependency Interaction Block)**（时间依赖 + 通道依赖交互）
   - 模块位置：`布尔大学士模块/B站更新（和视频同步更新）/102 Dual Dependency Interaction Block（AAAI 2025）.py`。
   - 同样输入 `[batch, feature_num, seq_len]`，用 patch 聚合做时间依赖交互。适合替换 `tcn2` 或 `tcn3` 作为“跨 patch 的时间依赖”。【F:布尔大学士模块/B站更新（和视频同步更新）/102 Dual Dependency Interaction Block（AAAI 2025）.py†L1-L93】

4. **TSSA (Token Statistics Self-Attention)**（线性复杂度注意力）
   - 模块位置：`布尔大学士模块/B站更新（和视频同步更新）/129 Token Statistics Self-Attention（ICLR 2025）.py`。
   - 期望输入 `[B, L, C]`，计算复杂度线性于序列长度，适合作为**替代 MSTCN 的“注意力型时序块”**。【F:布尔大学士模块/B站更新（和视频同步更新）/129 Token Statistics Self-Attention（ICLR 2025）.py†L1-L90】

## 实验表格：替代 MSTCN 的缝合方式（建议从简单到复杂）
> 下面每行就是一个实验配置。推荐先从 “单一替换” 做 ablation，再考虑混合/级联。

| 实验ID | 替换位置 | 替换模块 | 输入形状适配 | 设计依据 |
|---|---|---|---|---|
| E0 | 无（baseline） | MSTCN×3 | 不变 | 基线对照，确保后续提升可归因。 |
| E1 | `tcn3` | BiMamba2_1D | `(N,C,T,V) → (N*V,T,C)` | 双向 Mamba 处理长程时序依赖，替换末段下采样时序路径，提升全局时序建模。【F:布尔大学士模块/B站更新（和视频同步更新）/046 BiMamba2_1D.py†L193-L229】 |
| E2 | `tcn2` | MDM | `(N,C,T,V) → (N*V,C,T)` | 多尺度时间分解 + 线性混合，适合作为中段细粒度时间增强。【F:布尔大学士模块/B站更新（和视频同步更新）/101 Multi-Scale Decomposable Mixing Block（AAAI 2025）.py†L15-L75】 |
| E3 | `tcn2` | DDI | `(N,C,T,V) → (N*V,C,T)` | patch 级时间依赖 + 通道依赖交互，补充 MSTCN 的固定感受野不足。【F:布尔大学士模块/B站更新（和视频同步更新）/102 Dual Dependency Interaction Block（AAAI 2025）.py†L14-L93】 |
| E4 | `tcn` | TSSA | `(N,C,T,V) → (N*V,T,C)` | 线性复杂度注意力，提供更强的时间 token 交互，适合替换首段时序建模。【F:布尔大学士模块/B站更新（和视频同步更新）/129 Token Statistics Self-Attention（ICLR 2025）.py†L18-L90】 |
| E5 | `tcn2 + tcn3` | MDM + BiMamba2_1D | MDM 用 `(N*V,C,T)`，Mamba 用 `(N*V,T,C)` | 中段多尺度分解 + 末段全局序列建模，兼顾局部与全局时间依赖。【F:布尔大学士模块/B站更新（和视频同步更新）/101 Multi-Scale Decomposable Mixing Block（AAAI 2025）.py†L15-L75】【F:布尔大学士模块/B站更新（和视频同步更新）/046 BiMamba2_1D.py†L193-L229】 |

## 缝合要点（落地时的最小改动思路）
1. **逐关节时间建模（推荐）**：先把 `(N,C,T,V)` reshape 成 `(N*V, T, C)` 或 `(N*V, C, T)`，跑完时序模块后再 reshape 回 `(N,C,T,V)`。这能让模块专注“每个关节的时间动态”。
2. **保留 topo 注入 + A‑NHG 的先后顺序**：你已经在 `unit_san` 的 `bn0` 后注入 topo_bias，再接 A‑NHG。时序模块应当继续接在这一路之后，以保证时序模块“感知拓扑条件”。
3. **优先做单模块替换的消融**：先替换 `tcn3` 或 `tcn2`，确保增益稳定后，再替换更多层，避免一次性引入过多变量。

如果你告诉我你想先跑哪一条实验（比如 E1/E2/E4），我可以给你更具体的替换代码片段（包括 reshape 细节和保持残差的写法）。
