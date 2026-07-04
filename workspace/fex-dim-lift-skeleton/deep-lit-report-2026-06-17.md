# Deep Literature Review — fex-dim-lift-skeleton (idea scope) — 2026-06-17

## 循环统计

| 轮次 | 搜索数 | 新候选(去重后) | 选中读全文 | Wiki 写入 | 新关键词 |
|------|--------|---------------|-----------|----------|---------|
| 1 | 14 (10 arxiv axes + 4 extra) | 4 | 0 | 0 | — |

**B4 终止**：4 个新候选均不满足 B3 选择标准。146 篇已读论文（topic `0616-fex` wiki 池）已覆盖该 idea 的全部相关维度。

## 搜索覆盖

本轮执行了完整的 6-axis + 额外针对性搜索：

| Axis | 查询组数 | 说明 |
|------|---------|------|
| 方法 (Method) | 2 | cross-dimension transfer, permutation invariant expression tree |
| 应用 (Application) | 2 | high-dim PDE closed-form, dimension reduction SR for PDE |
| 数据 (Data/Benchmark) | 1 | PDE benchmark SR high-dim dataset |
| 评估 (Evaluator) | 1 | symbolic expression verification gatekeeper |
| 失败模式 (Failure) | 2 | RL policy gradient instability, SR overfitting spurious structure |
| 对抗 framing (Competition) | 2 | symmetry discovery PDE permutation, projection decomposition SR |
| 额外针对性 | 4 | ansatz discovery, tensor product decomposition, low-to-high dim transfer, hierarchical decomposition |

所有搜索均使用 `--max 15 --year 2025,2026`。

## B0 Web Search 发现

- **PSR (Projective Symbolic Regression)**: ICLR 2026 submitted (OpenReview #15825)，已在 landscape §15 中记载。是 closest prior——用低维 projection + 局部 SR + 全局 symbolic program synthesis 做高维 PDE。机制与我们的 macro inference 路径不同。
- 无其他 arxiv ID 产出。

## 本次新发现但未深读的论文

以下论文不在 146 篇已读列表中，但经 info 验证后判定与 fex-dim-lift-skeleton idea 相关性不足，未派 reader：

| arxiv_id | Title | Year | 为何不选 |
|----------|-------|------|---------|
| 2501.08086 | NOMTO: Neural Operator-based symbolic Model approximaTion and discOvery | 2025.01 | Neural Operator 扩展 SR 算子集，不涉及 cross-dimension lifting 或 invariant macro inference |
| 2603.10131 | Invariant Reduction for PDEs. IV: Symmetries that Rescale Geometric Structures | 2026.03 | 经典 Lie 对称性理论，无 ML/SR 组件，与 idea 机制无交集 |
| 2503.00645 | Computer Assisted Discovery of Integrability via SILO | 2025.03 | 面向可积系统的 Lax pair 发现，与 cross-dimension structure lifting 无关 |
| 2512.23410 | High-Dimensional Search, Low-Dimensional Solution: Decoupling Optimization from Representation | 2025.12 | 神经网络模型压缩，非 SR/PDE 领域 |

## 已读论文清单（与本 idea 直接相关者，选自 146 篇 topic wiki 池）

以下论文已在 landscape 中记载，是本 idea 的核心 related work：

| arxiv_id | Title | Year | 与本 idea 的关系 |
|----------|-------|------|-----------------|
| 2602.11630 | NMIPS (Neuro-assisted Multitasking Symbolic PDE Solver) | 2026.02 | Same-dimension 参数迁移——证明跨实例结构共享可行，但不做 cross-dimension lifting |
| — | HD-TLGP (AAAI 2024) | 2024 | 1D→3D 硬编码结构迁移，需已知解析解，d≤3。与我们自动发现+d=100 本质不同 |
| — | PSR (ICLR 2026 submitted) | 2026 | Projection decomposition + 局部 SR + 全局合成。最接近的竞品但机制根本不同 |
| 2506.19537 | DR-SR | 2025.06 | 统计变量组合发现做降维，不做 cross-dimension lifting |
| 2604.22208 | FEX+TranNet | 2026.04 | TransNet 初始化 FEX functional pool，不做 cross-dimension lifting |
| 2510.22497 | Multi-Scale FEX | 2025.10 | FEX 处理多尺度振荡，坦承频率分离导致 RL 不稳定——为 lift 稳定性提供 motivation |
| 2602.03816 | SymPlex | 2026.02 | RL+Transformer 做符号 PDE 求解，竞争方法但无 cross-dimension 组件 |
| 2405.14620 | SSDE | 2024.05 | RL-based symbolic ODE/PDE solver，FEX baseline，无 dimension lifting |
| 2505.10762 | DSO | 2025.05 | LLNL 深度符号优化框架，RSPG+GP seeding，为 FEX warm-start 提供范本 |
| 2506.20607 | H-FEX | 2025.06 | FEX 族内 Hamiltonian 扩展，interaction node 设计模式可参考 |
| 2509.07303 | FIND | 2025.09 | 物理约束搜索空间缩减，量纲不变性+两阶段分解，方法路线正交 |

## 本次新增论文（供上层并入 landscape）

**无。** 本轮搜索未发现需要深读的新论文。146 篇已读论文已饱和覆盖该 idea 相关的全部文献维度。

## 撞车风险评估

| Claim | 状态 | 证据 |
|-------|------|------|
| Low-d FEX skeleton → invariant macro inference → high-d parameter-only lift | 🟢 完全空白 | 无任何现有工作做此 pipeline。PSR 用 projection decomposition（非 macro inference），HD-TLGP 用硬编码宏（非自动发现），NMIPS 做 same-dim 迁移 |
| 三道门 gate (low-d fit / coefficient exchangeability / top-k probe) | 🟢 完全空白 | 无现有工作提出类似的多门验证机制 |
| Macro grammar {sum_x2, square_sum_x2, pairwise_xx} 的有限性 | 🟢 空白 | 无工作从 FEX skeleton 推断此类宏 |
| FEX RL controller 搜索稳定性对 lift 的影响 | 🟢 空白 | Multi-Scale FEX 承认频率分离导致 RL 不稳定，但未系统研究搜索稳定性对结构可迁移性的影响 |

**总体**：所有核心 claims 均未被覆盖。文献调研通过。

## 可用新工具/方法

- MDBench (2509.20529): PDE discovery benchmark，可用于评测 lifted expressions
- ERBench (2606.09276): Equation Recovery Benchmark，强调 OOD 鲁棒性
- EGG-SR (2511.05849) + eggp (2501.17848): 等价性剪枝模块，可作为 FEX plugin 提升搜索效率

## Errors

无。B4 正常终止，无 reader dispatch，无 wiki 写入，无 B7 调用。
