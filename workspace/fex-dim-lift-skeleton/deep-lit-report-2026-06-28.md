# Deep Literature Review — fex-dim-lift-skeleton — 2026-06-28

## 循环统计

| 轮次 | 搜索数 | 新论文 | 选中读全文 | Wiki 写入 | 新关键词 |
|------|--------|--------|-----------|----------|---------|
| R8 | 6 axes + 5 web | 6 candidates | 1 (NGCG) | 1 | constancy gate, diversity filter |
| R9 | 5 compensated | 0 | 0 | 0 | — |

## 已读论文清单

| arxiv_id | title | year | 撞车风险 | 关键发现 |
|----------|-------|------|---------|---------|
| 2603.20474 | NGCG: Neural Discovery of Conservation Laws Without False Positives | 2026.03 | 无（conservation law discovery，非 PDE 符号求解跨维 lift） | constancy gate + diversity filter 架构与 three-gate audit 平行，9-system benchmark 可作新 PDE 测试案例参考 |

## 本次新增论文（供上层并入 landscape）

| arxiv_id | title | year | 一句话解读 | 与哪个 claim/baseline 相关 |
|----------|-------|------|-----------|--------------------------|
| 2603.20474 | From Data to Laws: Neural Discovery of Conservation Laws Without False Positives | 2026.03 | NGCG 用四阶段解耦 pipeline（neural dynamics → multi-restart variance minimizer → symbolic extraction → constancy gate + diversity filter）在 9 个系统上实现 DR=1.0/FDR=0.0，gate 架构与 three-gate audit 概念平行 | 方法论参考（gate verification 设计模式），可为 CRITICAL-002 提供新 benchmark 系统灵感 |

## 撞车风险评估

| Claim | 状态 | 证据 |
|-------|------|------|
| A. Gate audit 安全 lift | 未被覆盖 | NGCG 做 conservation law discovery，不做 PDE 符号求解跨维 lift |
| B. 正确拒绝 0 lift-quality-FA | 未被覆盖 | 同上 |
| C. Gate 非 exact-match | 未被覆盖 | 同上 |
| D. Conservation 完整链 | 未被覆盖 | 同上 |
| E. Conservation 窗口唯一 | 未被覆盖 | 同上 |
| F. Pipeline 必要性 | 未被覆盖 | 同上 |
| G. Gates 必要性 | 未被覆盖 | NGCG 的 constancy gate 是独立概念，与我们的 Gate-3 PDE quality gate 不同 |
| H. 外部 baseline 优势 | 未被覆盖 | 同上 |
| I. Gate-2 冗余 | 未被覆盖 | 同上 |

**NGCG 不构成对任何核心 claim 的撞车风险。**

## 可用新工具/方法

- **NGCG 9-system benchmark**：含 4 个有守恒律系统（mass-spring, Lotka-Volterra, coupled springs, Hénon-Heiles）+ 5 个无守恒律系统（double pendulum, Lorenz, three-body, Burgers, KS），可作为 conservation-law PDE 场景的标准化测试集
- **Diversity Filter (ρ 比率)**：轻量级伪发现检测器，`ρ = std_i(mean_t(C)) / mean_i(std_t(C)) > 10`，可独立应用于 gate audit 中区分真实 lift 和 trivial constant
- **Multi-restart variance minimizer**：10 次独立随机初始化 + 选择验证集 constancy 最低的网络，策略可复用于我们的 low-d FEX multi-seed search（已有 5 seeds，但可增加 restart 维度）

## Errors

无。B7 四项调用均成功执行（references=20, cited=0, author chase=2, title-term chase=1）。wiki 已写入且 ≥50 行（150 行）。

## 文献池饱和度

- 已读论文总数（R8 后）：325 篇
- 连续无新增轮数：R5=0, R6=0, R7=1 (ASYS), R8=1 (NGCG)
- 状态：**硬饱和**。所有核心搜索轴（FEX 跨维 lift、gate audit、symbolic PDE verification、blind recovery、grammar misspecification）均返回 0 新论文。NGCG 是 conservation law discovery 方向的边缘相关论文，不构成竞争或方法替代。

---

<verdict>
核心 claims 逐一评估：
- Claim A-I：全部未被覆盖。NGCG (2603.20474) 专注于从轨迹数据发现动力学系统的守恒律，不做 PDE 符号求解跨维 lift、不做 gate audit for symbolic expression lift、不做 macro grammar inference。其 constancy gate 与我们的 PDE quality gate 是不同的独立概念。
总体：所有 9 个核心 claim 均未被覆盖。文献调研通过。
</verdict>
