<!-- 书写报告使用中文 -->
---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-dim-lift-skeleton/
---

- One-sentence summary: 从低维 FEX 表达式里推断可交换的宏变量, 如 `sum_x2`、`(sum_x2)^2`、`pairwise_xx`, 再把该宏提升到高维并只优化少量连续参数。
- Hypothesis: FEX 在某些高维 PDE 上看似逃过搜索维度灾难, 主要是因为解落在低复杂度的置换不变宏代数里。若低维骨架能自动映射到这些宏, 高维阶段不需要重新跑完整 RL 表达式搜索; 若低维骨架不是可交换结构, 正确行为是拒绝提升。
- Expected outcome: 成功信号是自动 macro selector 在 separable、radial、pairwise 三类 PDE 上选中正确宏, 在 d=10-100 通过参数-only 优化达到接近从零 FEX 的误差, 同时在非对称 PDE 上拒绝提升; 最便宜的反证是 selector 在低维 heldout 上无法区分 `sum_x2` 与非可交换二次型, 或强行提升在对称族上也达不到从零 FEX 2x 误差内。
- Contribution type: method+empirical-finding (保留 v1 类型; method 是 macro-inference-and-lift pipeline, empirical-finding 是检验 FEX 高维成功是否由可交换宏结构解释)
- Risk: MEDIUM
- Estimated effort:
  - Compute: 60-100 GPU-hours, dominated by multi-seed FEX searches for 3-5 PDE families and matched from-scratch baselines
  - Data: available; PDE collocation points are generated analytically, no external dataset required
  - Implementation: 2-3 weeks
- Novelty quick-check: HD-TLGP 已证明跨维结构迁移可行, 但它需要已知 1D 解析解, 用硬编码加法/乘法扩展, 只测到 d=3, 高维仍跑完整 GP 搜索。DR-SR (2506.19537) 自动找变量组合做降维, 不是 low-to-high lift。NMIPS (2602.11630) 在同一维度内做 PDE 族参数迁移, 不涉及跨维度。
  - **2026-06-16 deep-lit (idea scope)**: 7 轴 S2/arXiv 检索 + WebSearch 饱和扫描, **未发现直接覆盖**"低维 FEX/RL 骨架 → 自动置换不变宏 → 高维参数-only lift"的工作 (arXiv 2025-2026 窗口内 0 篇新相关论文可精读)。但发现 1 篇高度相关竞品:
    - **Projective Symbolic Regression (PSR)**: Lulu Cao (HD-TLGP 作者), Yinglan Feng, Liang Feng, Ran Cheng, KC Tan. ICLR 2026 投稿, 2025.09 提交. 无 arXiv ID (OpenReview 上). 核心方法: 对高维 PDE 解数据做多组低维投影 (fixing subsets of variables) → 每投影上 SR 提取局部函数分量 → 高层符号程序合成全局表达式 → PDE residual 约束. **与 fex-dim-lift-skeleton 的关键差异**: (a) PSR 用投影分解, 我们用置换不变宏推断; (b) PSR 的合成依赖高层符号程序 (可能用 GP/SR), 我们只跑低维 RL 搜索 + 参数-only 优化; (c) PSR 是通用框架, 不显式检测 exchangeable 结构, 我们专攻 separable/radial/pairwise 三类置换不变族; (d) PSR 的分解策略对变量间关系无先验假设, 我们的 selector 明确拒绝不可交换 PDE (asymmetric control). **撞车评估**: PSR 是最接近的竞品, 但机制不同 (projection-based decomposition ≠ permutation-invariant macro inference). 应作为 primary related work 引用, 强调我们的差异化: 自动检测可交换宏结构 vs. 通用投影分解, RL 骨架学习 vs. 投影 SR, 参数-only 高维优化 vs. 符号程序合成. **风险**: PSR 若被 ICLR 2026 接收 (正等待结果), 将成为本方向最强的 prior work, 需在 intro 中明确差异定位.
- Strongest objection: 仅凭 d=2 骨架推断高维宏存在 identifiability 风险, 因为多个宏在低维上可能拟合得很接近。
- Why we should do this: 这不是单纯加速技巧。它追问 FEX 的高维表现是否来自可恢复的对称结构。正结果给出可操作搜索策略; 负结果也能界定 FEX 无 CoD 近似理论与实际 RL 搜索之间的缺口。
- Pilot:
  - Setup: local RTX 4060 Ti; 复用已有 d=2 FEX Poisson search JSON, 新增 `infer_macro` 模式; low-dim heldout 拟合 + coefficient exchangeability check + d=10 PDE probe
  - Metric: accepted symmetric family 需 low-dim rel-RMSE < 0.02 且 d=10 probe rel-L2 接近机器精度; asymmetric family 应被拒绝, forced tied lift 明显变差
  - Result: Poisson FEX d=2 `final_expr` 自动选 `sum_x2` (rel-RMSE 6.46e-8, coefficient CV 8.4e-8), d=10 probe rel-L2 6.71e-8; analytic radial quartic 选 `(sum_x2)^2` (rel-L2 0.0); analytic pairwise 选 `pairwise_xx` (rel-L2 1.44e-7); asymmetric quadratic 被拒绝 (rel-RMSE 1.75e-1, forced lift rel-L2 7.43e-2)
  - Signal: POSITIVE but bounded; selector logic works, and Poisson uses real FEX output, but radial/pairwise rows are analytic skeleton proxies until full FEX searches are run
- Review handling: Accept the critiques about underspecified macro inference, missing negative control, multi-family evidence, matched baselines, multi-seed stability, tree depth, and break-even cost; v2 adds a concrete selector plus pilot and moves the remaining work into Experiments. Accept the mechanism-finding framing over a pure speedup story. Push back on collapsing the novelty into HD-TLGP because HD-TLGP uses known 1D solutions and hard-coded transfer, while this method learns low-d skeletons and lifts by automatic macro inference plus parameter-only optimization.
- Claims and Claims matrix: The central claim is that a low-dimensional FEX skeleton can become a high-dimensional macro program only when its fitted structure is exchangeable under variable permutations. A secondary claim is that accepted macros turn high-dimensional solving into low-dim search plus small parameter optimization, amortized over a dimension family. If all symmetric families pass and asymmetric controls reject, claim FEX scalability is macro-structure-driven. If manual lifts pass but automatic selection fails, downgrade to a diagnostic result about identifiability. If asymmetric controls also pass, the selector is over-permissive and the central claim fails.
- Narrative: Frame the paper as a mechanism study of FEX scalability. Approximation theory says the function class can avoid CoD, but the RL controller still needs searchable structure. Dimension lifting tests whether that structure is a recoverable invariant macro, not merely a lucky high-dimensional search.
- Experiments: Run FEX low-d searches for separable, radial-quartic, pairwise-interaction, and asymmetric PDE families; compare lifted solve to from-scratch FEX, FEX-PG where applicable, and HD-TLGP on matched compute; use 5 seeds, depth-1/2/3 trees, d in {10,20,50,100}, and a break-even curve showing when low-d search amortizes; include ablations removing heldout macro selection, coefficient exchangeability, and PDE probe rejection.
- Assets status: Code, v1/v2 pilot results, and data handoff are ready locally; see `workspace/fex-dim-lift-skeleton/data/MANIFEST.md`.

<review date="2026-06-16">

## Novelty

- Score: 7.5/10
- Closest prior work: HD-TLGP (Cao et al., AAAI 2024) — GP-based 1D→3D structural transfer with hard-coded macros and known analytical solutions; DR-SR (Kahlmeyer et al., 2506.19537, AAAI 2025) — statistical variable combination discovery for dimension reduction, no cross-dimension lifting; NMIPS (Huang et al., 2602.11630, 2026.02) — symbolic skeleton transfer across same-dimension PDE parameter instances via GP+MFO, not cross-dimension.
- Key differentiator: 低维 FEX RL 骨架学习 → 自动置换不变宏推断 (基于 coefficient exchangeability + heldout PDE probe) → 高维纯参数优化。三步骤组合在文献中仍然是空白。HD-TLGP 验证了跨维结构迁移的可行性, 但其使用硬编码宏、已知解析解、d=3 上限和完整 GP 搜索, 与本工作的自动发现+RL 骨架+d=100+纯参数优化有本质差异。本轮独立检索确认 NMIPS (2602.11630) 是同一维度内 PDE 族参数迁移 (非跨维度), 不威胁本 idea 的新颖性, 但应作为 related work 引用以佐证符号骨架迁移范式的有效性。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v2 通过 concrete selector (coefficient exchangeability check + PDE probe) 和 pilot 结果实质性填补了 v1 的宏推断空白。剩余 gap: (a) 候选宏 grammar 的穷举性和覆盖范围未定义——当前仅测试了 sum_x2、(sum_x2)^2、pairwise_xx 三种宏, 更复杂的置换不变结构 (如 sum_i sin(x_i)、高阶矩) 是否在 grammar 内? (b) 多个宏在低维上等价时 (如 d=2 时 sum_x2 和 pairwise_xx 在某些 PDE 上可能拟合接近) 的 tie-breaking 策略未说明; (c) deeper FEX trees (depth-2/3) 的宏推断行为完全未知。 |
| Missing evidence signals | 6/10 | v2 pilot 新增了 4 个关键信号: Poisson FEX 真实输出→sum_x2 自动选择、analytic radial quartic→(sum_x2)^2、analytic pairwise→pairwise_xx、asymmetric quadratic→拒绝。但多项 v1 指出的证据缺口仍在"计划实验"阶段: (1) 完整 FEX 搜索 radial/pairwise PDE 族 (当前仅 analytic proxy); (2) matched FEX-PG 和 HD-TLGP baseline; (3) 5-seed 稳定性; (4) depth-2/3 tree lifting; (5) break-even 摊销曲线; (6) ablation 移除 heldout macro selection 和 PDE probe。 |
| Narrative | 8/10 | 机制发现框架已完全采纳并明确表述 ("Frame the paper as a mechanism study of FEX scalability")。问题从"能否加速 FEX"升级为"FEX 的高维成功是否等价于其解落在可恢复的置换不变宏代数里", 这在科学上更有趣, 且无论正负结果都有发表价值。HD-TLGP 的差异化表述准确, pushback 有据。 |
| Venue contribution | 7/10 | Topic 未声明 target-venue, 按 topic 类型 (RL for Symbolic Regression / Scientific ML) 推断为 NeurIPS/ICML/ICLR。当前 v2 的贡献仍不构成完整 top-venue paper——宏推断方法虽已具体化但尚未在完整 FEX 多族搜索上验证。但若 Experiments 节列出的完整证据矩阵成功执行, 机制发现型论文在 NeurIPS/ICML 具备竞争力。 |
| Testability | 9/10 | falsifiability 极强。最便宜的证伪信号明确: selector 在低维 heldout 上无法区分 sum_x2 与非可交换二次型 (pilot 已测并通过), 或强行提升在对称族上达不到从零 FEX 2x 误差内。PDE probe 拒绝测试为 asymmetric PDE 提供了清晰的 NULL/NEGATIVE 边界。三个 outcome (POSITIVE/NULL/NEGATIVE) 都有清晰且可行的判定标准。 |
| Outcome realism | 7/10 | POSITIVE outcome (三类对称 PDE 族全部通过) realistic but non-trivial——pilot 已证实 Poisson (真实 FEX 输出) 和 analytic radial/pairwise proxy 可行, 但完整 FEX 搜索的非平凡骨架是否能被正确推断仍不确定。NULL outcome (手工 macro 工作但自动推断不稳定) 是高度 real 的中间结果, 有独立发表价值。60-100 GPU-hour 估计对 5-seed × 4 族 × 4 维度的完整矩阵可能偏乐观 (实际可能需要 120-150 GPU-hours)。 |
| Contribution type compliance | n.a. | Topic `0616-fex.md` 未声明 `preferred-contribution-types`; 跳过本检查。 |
| Overall Quality | 7.3/10 | v2 相比 v1 在所有可比较维度上均有实质性提升: 宏推断从"未指定"变为有 pilot 验证的 concrete selector, 叙事从加速工具升级为机制发现, falsifiability 进一步强化。剩余风险 (完整 FEX 多族搜索、depth-2/3 tree、宏 grammar 覆盖) 是实验执行风险而非概念缺陷。 |

## Contribution Drift

- v_1 contribution types: {method, empirical-finding}
- v_2 contribution types: {method, empirical-finding}
- Status: unchanged
- Hard cap triggered: no

## Alternative Framing

当前 framing 已是最锐利的版本: "FEX 避免了高维搜索, 当且仅当低维解落在可恢复的可交换宏代数中"——这是一个可证伪的机制诊断命题。建议在论文 intro 中进一步将主 claim 表述为必要条件式 ("FEX avoids high-d search *only if* the solution is macro-recoverable"), 使其 falsifiability 在语言层面更加鲜明。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 对于解落在可交换宏代数中的 PDE 族 (separable/radial/pairwise), 低维 FEX 骨架经自动宏选择和 PDE probe 验证后可提升到高维, 仅需参数优化即可匹配从零 FEX 精度, 并将搜索成本摊销到维度族上。 |
| NULL | 手工/解析宏提升在对称族上有效, 但真实 FEX 输出的自动宏推断在非平凡 skeleton 上不稳定或族特定; 此结果作为 identifiability 诊断有独立发表价值, 揭示了 FEX 高维成功对低维骨架"可辨识性"的依赖性。 |
| NEGATIVE | symmetric family 的自动提升失败或 asymmetric control 被通过, 则选定的 selector 不是 FEX scalability 的可靠解释; 此时论文贡献转为"排除宏结构假说"的负结果, 需显著降低 claim scope。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**: pilot 已证实核心机制在 Poisson (真实 FEX) + analytic proxy 上可行, 到 top-venue 论文的路径清晰 (执行 Experiments 节列出的完整证据矩阵)。但成功依赖于自动宏推断在非平凡 FEX skeleton (depth-2/3, 非 trivial PDE 族) 上的泛化, 这并非纯工程执行风险——宏推断可能在复杂 FEX 输出上遇到 identifiability 瓶颈。**Impact=High**: 如果成功, 本工作将 (a) 提供首个面向 FEX 高维搜索的可操作策略, (b) 揭示 FEX CoD 回避的结构性机制 (直连 landscape gap #1), (c) 开辟 skeleton-based SR 搜索的新研究方向。即使 NULL 结果, 作为"排除宏结构假说"的诊断也有清晰的发表价值。Claude 与 codex 在 Likelihood 和 Impact 上均无分歧 (均判定 Medium × High), 与 v1 时 codex 的 Likelihood=Low 评估相比, v2 的 pilot 证据显著提升了可行性判断的一致性。

## Overall

- Priority: High
- Score: 7
- Comments: v2 成功解决了 v1 的核心弱点——宏推断从概念变为有 pilot 验证的 concrete selector, 叙事从不明确的加速工具升级为清晰的机制发现框架。贡献类型未漂移, 无 hard cap 触发。剩余风险 (完整多族 FEX 搜索、depth-2/3 tree、宏 grammar 覆盖、matched baselines) 是合理范围内的实验执行风险。v1 review 的大部分 concern 已 resolved 或 partially resolved; weight-tying scheme 仍 ignored (未在 v2 body 中出现), 建议在下一版或 proposal 阶段明确。NMIPS (2602.11630) 已收录到 landscape, 建议在论文中作为 related work 引用。HD-TLGP 和 DR-SR 的差异化 pushback 有据, 但论文必须显式引用并清晰区分。

#if file_size_kB > 10   // 不含本次 review 块
超长警告: 文件 ~3 KB, 未超限
#endif

</review>

<deep-lit date="2026-06-16" scope="idea" rounds="1" saturation="B4 at R1">

## Deep-Lit 检索记录

### 搜索覆盖
| Axis | Query | 结果 |
|------|-------|------|
| 方法-1 | symbolic regression cross dimension structure lifting transfer PDE | 15 papers (S2), 0 相关 |
| 方法-2 | permutation invariant macro variable symbolic regression PDE | 11 papers (S2), 0 相关 |
| 应用 | high dimensional PDE symbolic closed form solution solver | 15 papers (S2), 2 相关但已在 landscape (StruSR 2510.06635) |
| 数据 | PDE discovery benchmark cross dimension generalization symbolic | 2 papers (S2), 0 相关 |
| 评估 | expression fidelity verification symbolic PDE discovery soundness | 8 papers (S2), 已有 landscape 论文为主 |
| 失败模式 | symbolic regression extrapolation failure higher dimension identifiability | 2 papers (S2), 0 相关 |
| 竞争-1 | automatic macro discovery structure transfer PDE solving | 2 papers (S2), 0 相关 |
| 竞争-2 | operator decomposition low dimensional projection equation discovery | 10 papers (S2), 0 相关 |
| B0 Web | "dimension lifting" + "permutation invariant" + "cross-dimension" + OpenReview/GitHub | 发现 PSR (见下) |

### 新发现竞品

**Projective Symbolic Regression (PSR)** — ICLR 2026 投稿, 无 arXiv ID
- 作者: Lulu Cao, Yinglan Feng, Liang Feng, Ran Cheng, KC Tan (Lulu Cao 为 HD-TLGP 一作)
- 方法: 多组低维投影 (fix subsets of variables) → 每投影 SR → 高层符号程序合成全局表达式 → PDE residual 约束
- 与 fex-dim-lift-skeleton 差异: PSR 用 projection-based decomposition, 我们用 permutation-invariant macro inference; PSR 需高层符号程序, 我们只需参数-only 优化; PSR 不检测交换性, 我们显式拒绝不可交换 PDE
- 撞车风险评估: **中** — 最接近的 prior work, 但机制完全不同。PSR 验证了"低维结构→高维解"范式的可行性, 可作为 positive prior。若被 ICLR 2026 接收将成为重要 reference
- 行动: 在论文 intro/related work 中明确引用并差异化; 关注 ICLR 2026 结果

### 终止结论
R1 结束后 B4 终止: 2025-2026 arXiv 窗口内无可精读的新论文。0616-fex landscape 已有 121 篇 wiki entries 覆盖 FEX/SR/PDE 空间, 本 idea 的特定方向 (cross-dimension skeleton lifting + permutation-invariant macro) 文献已饱和。PSR 虽有高度相关但不在 arXiv 上, 已手动记录。

### Verdict
- **Claim 1** (低维 FEX 骨架 → 自动置换不变宏 → 高维参数-only lift): **未被覆盖**. 最接近的 PSR 使用不同的机制 (投影分解 vs 宏推断)
- **Claim 2** (FEX 高维成功由可交换宏结构解释): **未被覆盖**. 无现有工作探讨此机制假说
- **总体**: ≥1 个 claim 未被覆盖 → **文献调研通过**, 实验可继续

</deep-lit>