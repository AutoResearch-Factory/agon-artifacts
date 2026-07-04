# Deep Literature Review — fex-dim-lift-skeleton (R11) — 2026-06-28

## 循环统计

| 轮次 | 搜索数 | 新论文 | 选中读全文 | Wiki 写入 | 新关键词 |
|------|--------|--------|-----------|----------|---------|
| R11 | 6 axes arxiv-tool + 5 B0 web | 0 | 0 | 0 | — |

## 已读论文清单

329 篇 wiki（`$ARXIV_WIKI_DIR/*.md` 中含 `## Read by: 0616-fex`）。

R1-R10 已累积收录包括但不限于：SIGS (2502.01476), CCM (2605.14546), Trustworthy AI (2509.26122), PG-SR (2602.13021), PICS (2603.21271), BEACONS (2602.14853), Formal Proofs (2503.13877), COSINE (2604.12806), A Posteriori (2502.20336), AutoNumerics (2602.17607), NOMTO (2501.08086), FEX-Turbulence (2605.10687), NGCG (2603.20474), SES (2606.07152), MOFS (2508.01211), Neural ODE-Transformer (2511.00102), NN PDE Survey (2601.13256), ASYS (2606.20467), 及 landscape 内所有已知竞品（SymPlex, SSDE, PSR, DSO, PROSE 系列, HD-TLGP, NMIPS, EGG-SR 等）。

## 本次新增论文（供上层并入 landscape）

**无。** R11 = R5-R6 后的再次 0-新增轮次。连续 3 轮（R5, R6, R11）0 新增确认文献池硬饱和。

## 撞车风险评估

| Claim | 撞车风险 | 最接近竞品 | 评估 |
|-------|---------|-----------|------|
| A. Gate audit 安全 lift | 低 | — | 无竞品做 FEX skeleton cross-dimension gate audit |
| B. Audit 正确拒绝 | 低 | BEACONS (formal cert), Trustworthy AI (a posteriori) | 方法论参考，非竞品 |
| C. Gate 非 exact-match | 低 | — | 无竞品 |
| D. Conservation 完整链 | 低 | — | 无竞品做相同 PDE family |
| E. Conservation 窗口唯一 | 低 | Exhaustive SR (2507.13033) | 穷举范式参考，非竞品 |
| F. Pipeline 必要性 | 低 | — | 无竞品 |
| G. Gates 必要性 | 低 | — | 无竞品 |
| H. 外部 baseline 优势 | 低 | SIGS, ASYS, PSR | 均为已知，方法正交 |
| I. Gate-2 冗余 | 低 | — | 实验发现，无竞品 |
| J. Claim scope 一致 | 低 | MOFS (2508.01211) | 跨 PDE 泛化困难是领域共性 |

## 可用新工具/方法

无。当前文献池已完整覆盖 symbolic PDE solver / FEX family / SR certification / verification / gate audit 方向的所有已知工作。

## Errors

无。本轮无 B5 dispatch、无 B7 反向扩展调用（B3 产出 0 篇）。

## 对实验的启示（不变）

文献调研已达硬饱和（329 篇 wiki，R5/R6/R11 三轮 0 新增）。实验应聚焦 reviewer V10 指出的 MUST-fix：
- **MUST-001**: sin×depth2_sub 特殊性的 semi-formal proposition
- **MUST-002**: Pipeline "激活维度" d≥50 标注
- **MUST-003**: 12/87 false rejects tie-break 分析
- **MUST-004**: FA 定义命名统一

这些问题的解决方案不在文献中——它们需要从实验内部（已有 624-cell gate ablation 数据、14-combo 穷举分析 JSON）推导和形式化。
