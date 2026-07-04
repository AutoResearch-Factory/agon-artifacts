<!-- 书写报告使用中文 -->
---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-dim-lift-skeleton/
---

- One-sentence summary: 从低维 FEX 表达式中识别有限的可交换宏 grammar, 用低维拟合、系数可交换性和 top-k 高维 PDE probe 三道门决定是否把该骨架提升到高维并只优化少量连续参数。
- Hypothesis: FEX 在部分高维 PDE 上看似绕过搜索维度灾难, 是因为 RL 搜索偶尔落在可恢复的可交换宏代数里。若低维 skeleton 能被有限 grammar 表达并通过高维 PDE probe, 高维阶段应只需要参数优化; 若 skeleton 不可交换、低维拟合不稳或 probe 与低维选择冲突, 正确行为是拒绝 lift。
- Expected outcome: 成功信号是 selector 在 separable 和 radial 的真实低维 FEX 输出上接受正确宏, 在 pairwise analytic skeleton 上验证宏本身可 lift, 并在非对称或失败搜索输出上拒绝; 最便宜的反证是 top-k probe 经常推翻低维最佳宏, 或真实 FEX radial/pairwise skeleton 的低维 rel-RMSE 无法低于 0.02。
- Contribution type: method+empirical-finding (保留 v2 类型; method 是 gated macro-inference-and-lift pipeline, empirical-finding 是检验 FEX 高维成功是否依赖可恢复的可交换宏结构; topic 未限制 contribution types)
- Risk: MEDIUM
- Estimated effort:
  - Compute: 80-140 GPU-hours, 主要来自多 seed 低维 FEX 搜索、matched from-scratch FEX/FEX-PG/HD-TLGP 对照和 depth sweep
  - Data: available; PDE collocation 和 boundary points 可解析生成, 无外部数据集
  - Implementation: 2-3 weeks
- Novelty quick-check: HD-TLGP 证明跨维结构迁移可行, 但需要已知 1D 解析解、硬编码加法/乘法扩展, 且只测到 d=3。PSR 是目前最近的 OpenReview 竞品: 它用低维投影、局部 SR 和全局 SR 组合求高维 PDE, 不是从低维 FEX skeleton 推断可交换宏; 2026-06-17 核验时 OpenReview 仍列为 ICLR 2026 submitted, 未显示接收状态。DR-SR 自动找变量组合做降维, NMIPS 在同一维度内迁移 PDE 族参数, 都不覆盖 low-d FEX skeleton -> invariant macro -> high-d parameter-only lift。
- Strongest objection: 宏 grammar 是人工设定的有限子语言, 因此正结果只能说明 FEX 的一部分高维成功来自可恢复的可交换结构, 不能声称解释所有 FEX 高维行为。
- Why we should do this: 这个 idea 把 FEX 的无 CoD 近似理论和实际 RL 搜索之间的缺口变成可测机制问题。正结果给出可操作的搜索压缩策略; 负结果也能定位 FEX 何时没有学到可迁移结构。
- Pilot:
  - Setup: local RTX 4060 Ti; 更新 `fex_dim_lift.py` 支持 family-specific search、任意单 family FEX 输出的 macro inference、top-k 高维 PDE probe; 跑 Poisson、radial、pairwise 和 asymmetric controls
  - Metric: 接受条件为 low-d macro rel-RMSE < 0.02, additive tied coefficient CV < 0.1, selected macro 的 d=10 probe rel-L2 < 1e-3, 且 top-k probe 不以 2x 以上 margin 推翻低维最佳宏
  - Result: Poisson 真实 d=2 FEX 输出选 `sum_x2` (low 6.46e-8, d=10 probe 6.71e-8; 错误 top-k probe 为 3.67e-2 和 1.47e-1)。Radial quartic 真实 d=2 FEX 搜索 seed1 找到两个 tied `x^2` leaves 的乘积, final rel-L2 5.10e-4; selector 选 `square_sum_x2` (low 1.48e-4, d=10 probe 0.0)。Pairwise analytic skeleton 选 `pairwise_xx` (low 8.51e-8, d=10 probe 1.44e-7), 但真实 pairwise FEX 搜索 seed2 未学到该 skeleton (search rel-L2 9.70e-1), selector 对该失败输出拒绝 (low 4.19e-1)。Asymmetric quadratic 被拒绝 (low 1.75e-1, coefficient CV 0.471, forced `sum_x2` probe 7.43e-2)。
  - Signal: POSITIVE but narrowed; Poisson 和 radial 已由真实 FEX 输出支持, pairwise 证明宏可 lift 但暴露了低维 FEX 搜索稳定性风险。
- Review handling: 接受 grammar 覆盖、tie-breaking、negative control、多 family 证据、matched baselines、多 seed 稳定性、tree depth 和 break-even cost 的批评。v3 直接定义有限宏 grammar, 用 top-k PDE probe 做 tie-breaker, 并补上真实 radial FEX 行和 pairwise 失败行。接受机制诊断叙事, 不把论文写成单纯加速技巧。不同意把 HD-TLGP 或 PSR 视为已覆盖本 idea: 它们也做 low-to-high 结构利用, 但没有从 low-d FEX skeleton 出发, 也没有用 invariant-macro gates 决定 parameter-only lift。也不同意把本文要求成通用 symmetry discovery; 合理 claim 是一个定义清楚的宏类上的 gated method。Contribution drift check: v2 -> v3 仍是 `method+empirical-finding`, 没有删除类型。
- Claims and Claims matrix: 主 claim 是: low-dimensional FEX skeleton 只有在能落入有限 exchangeable macro grammar 并通过独立 PDE consistency check 时才应该 lift。辅 claim 是: 这些 PDE family 上的 FEX scalability 取决于 RL search 是否真的找到可恢复宏, 而不是宏在解析上是否存在。若 Poisson/radial/pairwise 的真实 FEX searches 多 seed 通过, 且 asymmetric controls 被拒绝, 可以说 macro-recoverability 解释了 FEX high-d behavior 的一个有用子集。若 analytic macros 通过但真实 FEX 输出失败, 论文降级为 search diagnostic: FEX 有可 lift 的目标, 但 controller 不稳定地找到它。若 asymmetric controls 通过或 top-k probe 经常推翻 low-d selection, selector 过松, method claim 失败。
- Narrative: 把这篇写成 FEX scalability 的反证实验。FEX 在函数类上可以无 CoD 近似高维解, 但这里问的是 RL controller 什么时候会找到一个低维 skeleton, 并且这个 skeleton 能被认证成高维 macro program。
- Experiments: Run 5 seeds for separable, radial quartic, pairwise interaction and asymmetric controls; include depth-1/2/3 trees because liftability may depend on how the controller represents the same algebra; compare accepted lifts with from-scratch FEX, FEX-PG where applicable, HD-TLGP and PSR-style projection SR on matched compute; report d in {10,20,50,100}, top-k probe contradictions, rejection rates and amortization break-even curves. Do not frame extra datasets as a benchmark contribution; they are evidence for the method and mechanism claims.
- Assets status: Code and v3 local GPU pilot results are ready; see `workspace/fex-dim-lift-skeleton/data/MANIFEST.md`.

<review date="2026-06-17">

## Novelty

- Score: 8/10
- Closest prior work: HD-TLGP (Cao et al., AAAI 2024) — GP-based 1D→3D structural transfer with hard-coded macros and known analytical solutions; PSR (Cao et al., ICLR 2026 submitted, OpenReview #15825) — projection-based decomposition with per-projection SR + symbolic program synthesis; DR-SR (Kahlmeyer et al., 2506.19537, AAAI 2025) — statistical variable combination discovery for dimension reduction, no cross-dimension lifting; FEX+TranNet (Huynh et al., 2604.22208) — TransNet initialization for FEX functional pool, no cross-dimension lifting or macro discovery.
- Key differentiator: **FEX RL skeleton → permutation-invariant exchangeability check → top-k PDE probe gate → parameter-only high-d lift.** 三步骤+三道门的组合在文献中仍然是空白。PSR 是最接近的竞品（都做 low-d 结构→high-d PDE），但机制根本不同：PSR 用 projection-based decomposition（固定变量子集做投影），不做 permutation invariance 检测；PSR 的高层合成走 symbolic program synthesis，我们只需参数优化。本轮独立检索确认 GenSR (ICLR 2026)、FEX+TranNet (2604.22208) 均不涉及 cross-dimension macro inference。NMIPS (2602.11630) 是 same-dimension 参数迁移，不威胁。HD-TLGP 需已知 1D 解析解、硬编码宏、d=3 上限、GP 全搜索，与我们的自动发现+RL 骨架+d=100+纯参数优化有本质差异。机制假说 claim（FEX 高维成功部分由可恢复的可交换宏结构驱动）亦未被任何现有工作提出或检验。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 8/10 | v3 实质性填补了 v2 的核心 gap: 定义了有限宏 grammar {sum_x2, square_sum_x2, pairwise_xx}, 用 top-k PDE probe 做 tie-breaker, pilot 覆盖了全部 4 个 family（Poisson 真实 FEX / radial 真实 FEX seed1 / pairwise analytic+真实失败 / asymmetric 拒绝）。剩余 gap：(a) pairwise 真实 FEX seed2 失败暴露了 FEX RL controller 的搜索稳定性风险——这是核心贡献能否成立的关键瓶颈而非小工程问题；(b) top-k probe 推翻 low-d winner 时决策规则（"2x margin" 阈值）的校准缺乏理论依据；(c) depth-2/3 tree 行为完全未知，而 liftability 可能依赖于 controller 如何以不同深度表达同一代数结构；(d) macro grammar 覆盖仅在 3 种宏上验证，更复杂的置换不变结构（sum_i sin(x_i)、高阶矩）不在 grammar 中，但 v3 已明确承认这是有限子语言而非通用 symmetry discovery。 |
| Missing evidence signals | 7/10 | v3 pilot 新增了 2 个关键证据：真实 radial FEX seed1 行（square_sum_x2 被正确选中，final rel-L2 5.10e-4）和 pairwise FEX 失败行（seed2 rel-L2 9.70e-1，selector 正确拒绝）。但以下证据仍在计划阶段：(1) multi-seed FEX 搜索稳定性（5 seeds per family）；(2) d ∈ {20, 50, 100} 真实 FEX 升维实验（当前仅 d=10 probe）；(3) matched baselines（FEX-PG、HD-TLGP、PSR-style projection SR）在同等 compute 预算下对照；(4) depth-2/3 tree 升维验证；(5) break-even 摊销曲线；(6) gate ablation 各门的边际贡献。其中 (1) 和 (3) 是论文能否被接收的最关键剩余证据——pairwise 行已证明搜索稳定性不是可以忽略的小风险。 |
| Narrative | 8/10 | 机制诊断框架已完全落实："FEX scalability 的反证实验"——追问 FEX 的 RL controller 什么时候会找到一个低维 skeleton，并且这个 skeleton 能被认证成高维 macro program。这个叙事比"加速工具"强得多，无论正负结果都有发表价值。v3 在 one-sentence summary 和 Hypothesis 中已将核心思想表述为必要条件式（"只有当..."），falsifiability 在语言层面也更鲜明。 |
| Venue contribution | 7/10 | Topic 未声明 target-venue，按 topic 类型（RL for Symbolic Regression / Scientific ML）推断为 NeurIPS/ICML/ICLR。若 Experiments 节列出的完整证据矩阵成功执行（多 family 多 seed、matched baselines、深度 sweep、摊销曲线），机制发现型论文在 top venue 的 scientific ML / neurosymbolic track 具备竞争力。当前 pilot 证据不构成完整 paper，但到完整 paper 的路径清晰可执行。 |
| Testability | 9/10 | Falsifiability 极强且便宜。最便宜的证伪信号：top-k probe 以显著 margin 频繁推翻 low-d 最佳宏（pairwise 行已经展示了 selector 在真实 FEX 失败输出上正确拒绝的能力），或真实 FEX radial/pairwise skeleton 的 low-d rel-RMSE 多 seed 无法稳定低于 0.02。三个 outcome 都有清晰判定标准。三道门（low-d fit / coefficient exchangeability / top-k probe）各自独立可消融，falsifiability 模块化程度高。 |
| Outcome realism | 8/10 | POSITIVE outcome：Poisson 和 radial 已由真实 FEX 输出支持（各 1 seed），pairwise analytic 宏可 lift 但真实 FEX seed2 搜索失败暴露了核心风险——FEX RL controller 能否稳定找到可恢复的 skeleton 是决定性的不确定因素。这使 pairwise 成为最关键的 stress-test family。NULL outcome（解析宏 lift 有效但自动推断在多 seed 上不稳定）高度 realistic 且有独立发表价值。80-140 GPU-hour 估计对 4 families × 5 seeds × 4 dimensions × depth sweep + matched baselines 可能偏乐观，建议按 120-180 GPU-hours 预算。 |
| Contribution type compliance | n.a. | Topic `0616-fex.md` 未声明 `preferred-contribution-types`; 跳过本检查。 |
| Overall Quality | 7.8/10 | v3 相比 v2 在所有可比较维度上均有提升：macro grammar 从"未定义"到"有限但明确定义"，tie-breaking 从"缺失"到"top-k PDE probe"，pilot 从 3 个 analytic proxy 行扩展到 4 family 全覆盖（含 2 个真实 FEX 行和 1 个失败行）。"机制诊断"叙事稳固。剩余风险（FEX 搜索稳定性、multi-seed、matched baselines）是实验执行风险而非概念缺陷，但 pairwise 真实失败警告这些风险不可轻视。 |

## Contribution Drift

- v_2 contribution types: {method, empirical-finding}
- v_3 contribution types: {method, empirical-finding}
- Status: unchanged
- Hard cap triggered: no

## Alternative Framing

当前 framing 已是最锐利版本。可进一步收紧：将论文主 claim 表述为 **"macro-recoverability certificate for FEX scalability"**——不是"又一个高维 PDE solver"而是"FEX 何时可以被认证为 liftable 的决策程序"。这比当前的"dimension lifting method"更能防止审稿人将其归类为 incremental solver work。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 对于解落在有限可交换宏 grammar {sum_x2, square_sum_x2, pairwise_xx} 中的 PDE 族，低维 FEX RL controller 多 seed 稳定恢复的 skeleton 在通过 low-d fit、coefficient exchangeability 和 top-k PDE probe 三道门后，可被安全提升到高维并仅用参数优化匹配从零 FEX 精度，将搜索成本摊销到维度族上。 |
| NULL | 解析/手工宏本身可 lift，但真实 FEX RL controller 在非平凡 skeleton 上多 seed 不稳定，导致自动 selector 的召回率不足；此结果作为"FEX search quality diagnostic"有独立发表价值——无论宏结构在解析上是否存在，FEX 能否可靠找到它才是 lift 的前提。 |
| NEGATIVE | 真实 FEX skeleton 多 seed 在 separable/radial/pairwise 族上大面积失败，或 asymmetric control 被误通过，或 top-k probe 频繁推翻 low-d selection；此时 method claim 失败，论文贡献转为负结果——排除"可交换宏结构解释 FEX high-d 成功"这一假说。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**: v3 pilot 在 Poisson（真实 FEX）和 radial quartic（真实 FEX seed1）上取得正面信号，到 top-venue 论文的路径清晰。但 pairwise 真实 FEX seed2 搜索失败暴露了核心瓶颈——FEX RL controller 能否稳定找到可恢复的 skeleton 是决定性的不确定因素，这使 Likelihood 无法达到 High。**Impact=High**: 如果成功，本工作将 (a) 提供首个面向 FEX 高维搜索的可操作 lift/reject 策略，(b) 揭示 FEX CoD 回避的结构性机制——将近似理论的无 CoD 保证与实际 RL 搜索的成功条件连接起来，(c) 开辟"search quality diagnostic"这一 FEX 子方向。即使 NULL 结果也有清晰发表价值。**Claude 与 codex 无分歧**：均判定 Likelihood=Medium, Impact=High → Priority=High (7)。

## Overall

- Priority: High
- Score: 7
- Comments: v3 在三轮迭代中持续聚焦和收紧：v1 建立了概念和 pilot，v2 具象化了 selector 和叙事，v3 定义了有限宏 grammar、引入 top-k gate 并补充了真实 FEX 证据行和失败行。"机制诊断"叙事稳固且无论正负结果都有发表价值。核心不确定因素已从"宏是否能被推断"转移到"FEX RL controller 能否稳定找到可恢复的 skeleton"——pairwise 真实 FEX 失败是这一风险的早期预警，必须在下一步中优先以 multi-seed 实验检验。v2 review 的大部分 concern 已 resolved 或 partially resolved。weight-tying scheme 仍 ignored（v2 和本轮均未在 body 中出现），建议在 proposal 阶段必须明确。PSR 应被当作最强的 related work + baseline，而非轻描淡写为"机制不同"。Contribution type 未漂移，topic 未声明 preferred-contribution-types，无 hard cap 触发。Claude 与 codex 在 Likelihood 和 Impact 上均无分歧。

#if file_size_kB > 10   // 不含本次 review 块
超长警告: 文件约 5 KB (含本 review), 未超限
#endif

</review>

<deep-lit date="2026-06-17" scope="idea" topic="0616-fex">

## 搜索记录

针对 fex-dim-lift-skeleton 的核心机制（cross-dimension skeleton lifting / invariant macro inference / three-gate verification），于 2026-06-17 执行了 6-axis + 4 组额外针对性搜索，共 14 组 query，覆盖：

- **方法轴**: cross-dimension structure transfer, permutation invariant expression tree
- **应用轴**: high-dim PDE closed-form symbolic solution, dimension reduction SR for PDE
- **数据轴**: PDE benchmark SR high-dim dataset
- **评估轴**: symbolic expression verification gatekeeper
- **失败模式轴**: RL policy gradient instability in expression tree search, SR overfitting
- **竞争轴**: symmetry discovery PDE permutation invariance, projection-based SR high-dim composition
- **额外**: ansatz discovery, tensor product decomposition, low-to-high dim transfer, hierarchical decomposition

**结果**: 所有搜索返回的 2025-2026 年论文几乎全部已在 146 篇 topic wiki 池中。发现 4 篇未读候选（2501.08086 NOMTO, 2603.10131 Invariant Reduction for PDEs IV, 2503.00645 SILO, 2512.23410 High-D Search Low-D Solution），经 info 验证后判定均与本 idea 核心机制无直接交集，未派 reader。

**饱和结论**: 文献调研饱和。本 idea 的核心 claims——low-d FEX skeleton → invariant macro inference → gate-verified high-d parameter-only lift——在现有文献中完全空白。最接近的竞品（PSR projection decomposition, HD-TLGP hard-coded macros, NMIPS same-dim transfer）与我们的机制路径有本质差异。

**已确认的竞品无变化**: PSR (ICLR 2026 submitted) 仍是最接近的 prior，但其 projection-based decomposition + symbolic program synthesis 与我们 invariant macro inference + parameter-only lift 机制根本不同。其余竞品格局无变化。

</deep-lit>
