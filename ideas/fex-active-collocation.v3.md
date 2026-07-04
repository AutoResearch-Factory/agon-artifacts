<!-- 书写报告使用中文 -->
---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-active-collocation/
---

- One-sentence summary: 把 FEX 的候选表达式缓存当成 query-by-committee, 用候选间 PDE 残差分歧选择 hard collocation points, 再用这个 probe 判断 FEX 的瓶颈是 reward 估计还是表达式结构搜索.
- Hypothesis: QBC collocation 只会在两个条件同时满足时有用: collocation noise 会改变候选表达式排序, 且 PDE 的误差集中在 uniform sampling 容易漏掉的局部区域. Smooth Poisson 上的证据已经反过来了: naive QBC 没有加速, stronger reward probe 中 QBC-mixed pools 把 fixed-candidate reward-error CV 从 0.535 提到 1.032, rank stability 从 0.333 降到 0.167. 这里的 hard set 主要是在制造 reward non-stationarity.
- Expected outcome: 成功要求 EMA-stabilized QBC 在 Burgers/conservation-law 这类局部特征 PDE 上先降低 fixed-candidate reward variance 或提高 rank stability, 再在 3 seeds full search 中用至少 25% less wall time 找到同等质量表达式, hold-out residual 不变坏. 最便宜的证伪信号是: localized PDE 的 fixed-candidate probe 里, EMA-QBC 仍不优于 uniform 或 residual-only sampling.
- Contribution type: method + diagnostic
- Risk: MEDIUM
- Estimated effort:
  - Compute: 45 GPU-hours for Poisson + two localized PDE families, 3 seeds, and hard-set ablations
  - Data: available
  - Implementation: 1-2 weeks
- Novelty quick-check: RL-PINNs, PINNACLE, Aikawa et al., Deep Collocation Method, variational residual adaptivity, PACMANN, and recent coreset/QR-DEIM PINN papers all study adaptive collocation for neural PDE solvers. DISCOVER, SSDE, SymPlex, and FEX study symbolic/RL PDE expression search. We found no prior work that uses disagreement among symbolic candidate expressions to choose collocation points for an RL expression-tree reward.
- Strongest objection: Current evidence is negative on smooth Poisson, and the stronger probe shows QBC can partly collapse toward residual-heavy points while making candidate rewards less stable; without a localized-PDE variance win, this is a diagnostic result rather than a useful method.
- Why we should do this: The experiment separates two FEX failure modes that are currently conflated: noisy reward estimation over collocation points versus discrete structure exploration. A positive result gives a small sampling module; a negative result tells the FEX line not to spend effort on active collocation for smooth high-dimensional solves.
- Pilot:
  - Setup: GPU Poisson `d=10` search using upstream FEX, comparing uniform sampling with 80/20 QBC hard-set sampling, plus a v3 fixed-candidate reward probe over uniform and QBC-mixed pools.
  - Metric: Full-search metric is first RL step crossing `score > 0.99`; diagnostic metric is whether QBC lowers reward-error CV and preserves candidate rank stability relative to uniform.
  - Result: Standard uniform crossed at step 18; active refresh=50 tied only because it did not refresh before convergence; active refresh=5 crossed at step 23. In the stronger reward probe, QBC/residual overlap was 0.712, QBC/residual score correlation was 0.247, reward-error CV worsened from 0.535 to 1.032, and rank stability worsened from 0.333 to 0.167. Controller entropy stayed near 1.0, so the observed issue is not early policy collapse.
  - Signal: NEGATIVE

- Claims and Claims matrix: Main claim: candidate-disagreement collocation is a direct test of FEX reward geometry, not a generic active-learning trick. POSITIVE: EMA-QBC reduces fixed-candidate reward variance on localized PDEs and yields at least 25% wall-time reduction with unchanged hold-out residual. MIXED: QBC does not reliably accelerate search, but its variance/rank/overlap diagnostics separate smooth PDEs from localized PDEs where sampling matters. NEGATIVE: FEX reward under the tested budgets is robust enough to uniform collocation, and active hard sets mostly add non-stationarity; the main bottleneck is expression topology search.
- Narrative: The paper should frame collocation as a stress test for FEX reward design. FEX already produces a candidate committee; the question is whether the committee identifies points that genuinely distinguish expressions, or whether hard-set updates scramble the reward surface faster than the controller can learn from it.
- Experiments: Gate 0 is a fixed-candidate probe on Poisson and one localized PDE: uniform, residual-only, random hard set, static QBC, and EMA-QBC; report reward-error CV, rank Spearman, top-action flip rate, controller entropy, QBC/residual overlap, and hold-out residual. Gate 1 runs full FEX search only if Gate 0 shows lower variance or better ranking, using 3 seeds and wall-clock normalized metrics. If Gate 0 fails on localized PDEs, stop the method claim and keep the diagnostic claim.
- Assets status: FEX code, Poisson search artifacts, and v3 reward-geometry probes are recorded in `workspace/fex-active-collocation/data/MANIFEST.md`; no external data or model download is needed.

<review date="2026-06-17">

## Novelty

- Score: 6/10
- Closest prior work: RL-PINNs (Song 2025, arXiv:2504.12949) — DQN-driven adaptive collocation for PINN training; Diversity-Aware Adaptive Collocation via Sparse QUBO & Hybrid Coresets (Salloum et al. 2026, arXiv:2603.06761) — coreset collocation with diversity+informativeness dual objective; QR-DEIM (Celaya et al. 2025, arXiv:2501.07700) — residual snapshot SVD + QR pivot collocation; SymPlex (Park & Osher 2026, arXiv:2602.03816) and SSDE (Wei & Yu 2024, ICML 2025) — closest symbolic PDE search competitors but use fixed sampling.
- Key differentiator: 将 Query-by-Committee (QBC) 原则与 FEX RL 表达式树搜索结合, 利用 search buffer 中自然存在的候选 expression committee 按残差分歧选择 collocation 点. 这在 PINN adaptive collocation (已高度饱和) 和 RL/symbolic PDE search (固定采样) 两个领域中都未被探索. 新颖性属增量式交叉组合而非机制突破, 但在 deep-lit (6篇精读 + 24次 B7 反向扩展, 2026-06-16) 和本次补充搜索中均确认无重叠. Pilot NEGATIVE 结果旁证了新颖性: 若此事显而易见或已知有效, 不会至今无人做.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v3 hypothesis 退化为双条件 (collocation noise 改变排序 + PDE 局部集中), 比 v2 更精确. 但因果链仍未拆解: (a) QBC 选点→降低 reward variance→加速搜索 这条链路中, pilot 数据显示 rank stability 从 0.333 恶化到 0.167, 即 QBC 反而让 reward 排序更不稳定, 这意味着存在抵消性机制 (variance 可能降低但 bias 增大); (b) EMA 稳定化 hard set 是合理 proposal, 但从 "EMA loss weighting 在 PINN 中有效" 到 "EMA collocation set 稳定 reward 排序" 之间缺机制论证——EMA 平滑的是 collocation 点分布, 不等于平滑 reward 排序; (c) controller entropy 维持在 ~1.0 被解释为 "排除 policy collapse", 但若 operator 空间较大, entropy=1.0 可能对应的是近乎随机的策略, 需对照 max entropy baseline. |
| Missing evidence signals | 5/10 | v3 已补上 v2 要求的核心诊断信号: reward-error CV (0.535→1.032)、rank Spearman、QBC/residual overlap (0.712)、score correlation (0.247)、controller entropy. 但仍缺失: (a) localized PDE (Burgers/Allen-Cahn) 上的同口径 fixed-candidate probe——这是 Gate 0 的核心判定依据, 目前完全依赖 conjecture; (b) EMA hard-set probe 在任何 PDE 上均未执行, EMA decay rate 的 sensitivity sweep 也未规划; (c) residual-only baseline 的 fixed-candidate variance/rank 报告——QBC overlap=0.712 意味着 ~29% 选点不同于 residual-only, 但未验证这 29% 差异点的独立信息量; (d) 多 seed 的 variance 置信区间——CV=1.032 vs 0.535 是一个点估计, 不清楚是系统性恶化还是单 seed 噪声. |
| Narrative | 7/10 | v3 叙事从 "apply active learning to FEX" 成功转型为 "collocation as stress test for FEX reward design", 这是正确的方向. 明确指出 "这里的 hard set 主要是在制造 reward non-stationarity" 是一种诚实的 self-critique, 增加了 diagnostic framing 的可信度. 当前叙事的主要弱点: (a) 对外部 (非 FEX) 审稿人, "FEX reward design 的 stress test" 的可迁移洞见是什么? 若仅回答 "FEX 的 bottleneck 在 structure search 而非 reward estimation", 这对不使用 FEX 的人价值有限; (b) 叙事中 "QBC 是诊断工具" 与 "EMA-QBC 可能是方法" 两条线仍存在张力——paper 到底是 method paper 还是 diagnostic paper? 当前 body 似乎在 "method + diagnostic" 之间摇摆. |
| Venue contribution | 5/10 | 按 FEX 发表历史 (JMLR 2025, JCP 2025) 推断目标 venue 压力: 作为 diagnostic + 潜在的 method paper, 对 JMLR/JCP 有合理空间. 但对 NeurIPS/ICML 不足——贡献局限在 FEX niche, 未提供可迁移到更广泛 ML/PDE 社区的 insight. 若 localized PDE Gate 0 通过且 EMA-QBC 在 ≥3 PDE families 上显示稳健收益, contribution 可升至 6/10; 若 localized PDE Gate 0 也失败, 仅剩 "FEX reward is robust to collocation" 的 narrow negative result, contribution 降至 4/10. |
| Testability | 9/10 | Gate 0 是出色的 cheapest falsifying signal: localized PDE fixed-candidate probe, 比较 uniform / residual-only / random hard set / static QBC / EMA-QBC, 报告 reward-error CV 和 rank Spearman. 若 EMA-QBC 在 localized PDE 上仍不优于 uniform 或 residual-only, method claim 当场死亡——这个判定明确、成本低 (<5 GPU-hours)、因果链清晰. 唯一 missing: 未定义 localized PDE probe 中 "不优于" 的定量阈值 (统计意义上还是绝对数值?). |
| Outcome realism | 7/10 | POSITIVE 被 Gate 0 加了一道前置滤波 (先降 variance 再跑 full search), 降低了追求不可能目标的概率. 25% wall-time reduction 是合理的阈值 (非 v1 的 30%). Pilot NEGATIVE 提供了诚实的 ground-truth calibration. NULL 和 NEGATIVE 的 claim 有发表价值, 且均有实验支撑路径. 唯一 realistic concern: 若 localized PDE 上 EMA-QBC 仍是 NEGATIVE, "reward non-stationarity dominates" 的 insight 可能过薄——缺乏对 "why reward non-stationarity is inherent to FEX's reward surface" 的机制解释, 而不仅仅是 "we tried QBC and it didn't work". |
| Contribution type compliance | n.a. | topic 未声明 `preferred-contribution-types`, 跳过检查. |
| Overall Quality | 6/10 | v3 是一个诚实的、被 pilot 数据校准的 diagnostic idea. 最大的资产是执行了 pilot 并诚实地报告了 NEGATIVE 信号——这比未经测试的乐观 hypothesis 有价值得多. 最大的负债是贡献深度: 当前版本在 "method" 和 "diagnostic" 之间摇摆, 且 localized PDE Gate 0 完全是 conjecture. 实验优先级的建议与 v2 review 一致: 先跑 localized PDE fixed-candidate probe, 根据结果决定该 idea 是 method (若 Gate 0 通过) 还是 pure diagnostic (若 Gate 0 失败). |

## Contribution Drift

- v_2 contribution types: {method, diagnostic}
- v_3 contribution types: {method, diagnostic}
- Status: unchanged
- Hard cap triggered: no — topic 未声明 preferred-contribution-types.

### v2 Review Concern Resolution

| v2 Concern | Status | Notes |
|-----------|--------|-------|
| Logical gaps (因果链未拆解) | Partially resolved | Hypothesis 加了双条件约束, 但 QBC disagreement → variance → convergence 的分解验证仍未做; EMA 机制论证仍停留在类比层面. |
| Missing evidence signals (缺 fixed-candidate variance / entropy / QBC-vs-residual overlap) | Partially resolved | v3 已补 Poisson fixed-candidate probe 的全套诊断数据; 但 localized PDE 和 EMA probe 仍白卷, residual-only baseline 的方差/rank 对照缺失. |
| Narrative (QBC framing) | Resolved | v3 维持并深化了 QBC/reward-geometry framing, "stress test" 叙事比 v2 更聚焦. |
| Venue contribution (niche depth) | Partially resolved | Diagnostic framing 增强了可信度, 但核心问题 (贡献是否足够 top venue) 取决于 localized PDE Gate 0 的结果. |
| Testability (单 seed / 单 PDE) | Resolved | Pilot 从 Poisson 扩展到 fixed-candidate probe, 多 seed/多 PDE 已在 experiment plan 中. |
| Outcome realism (aggressive threshold) | Partially resolved | v3 的 POSITIVE 加了 Gate 0 前置条件, 25% 比 v1 的 30% 更务实; NULL/NEGATIVE 边界清晰. |

**Refiner pushback 评估**: v3 基本全面采纳了 v2 review 的 diagnostic 转向建议, 并用更强 Poisson probe 数据支撑了 "reward non-stationarity" 叙事. 未完成项 (localized PDE Gate 0, EMA probe, residual-only baseline variance) 属 pending experiments 而非 refiner 拒绝采纳. Pushback 均有据, 无 silent downgrade.

## Alternative Framing

当前 framing 已是 v2→v3 改进后版本, 方向正确. 可进一步锐化为: **"Candidate-disagreement collocation as a falsification test: When should we stop optimizing FEX's reward estimation?"** 核心 claim 不是 "QBC 加速 FEX", 而是 "QBC 实验告诉我们 FEX 的 reward design 已经够了, 不必在 sampling 上投入 effort". 这个 framing 将 diagnostic contribution 升格为主贡献, method contribution 降为 diagnostic 的自然推论 (仅当 Gate 0 通过时才存在), 解决了当前 "method vs diagnostic" 的叙事张力.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | EMA-QBC 在至少一个 localized PDE (Burgers 或 Allen-Cahn) 的 fixed-candidate probe 中降低 reward-error CV 或提高 rank Spearman; 在 equal-compute/equal-overhead 口径下, 3 seeds full search 用 ≥25% less wall time 找到同等质量表达式, hold-out residual 不退化. 优势局限于 localized PDE + EMA 组合, 不能 claim "active collocation generally improves FEX". |
| NULL | EMA-QBC 在所有测试 PDE 上均无稳定 wall-time 加速, 但 variance/rank/overlap diagnostics 能区分 smooth PDE (reward robust) 与 localized PDE (reward sensitive) 的采样需求差异, 为 FEX community 提供 collocation budget 分配的经验准则. |
| NEGATIVE | 若 localized PDE + EMA + residual-only 对照下 fixed-candidate probe 证实 reward-error CV 和 rank stability 无改善, 可 claim: 在 tested FEX configuration/budget (5000 collocation points, d≤10) 下, reward estimation 不是主瓶颈; active hard set 主要通过制造 reward non-stationarity 损害 controller 学习; FEX 社区应将 effort 优先投入 structure exploration 而非 collocation engineering. 这个 NEGATIVE claim 必须显式限定 scope (tested PDEs + budget), 不能 claim "FEX universally bottlenecked by structure search". |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium × Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - **Likelihood = Medium**: 实验路径清晰 (FEX 代码可用, Gate 0 成本 <5 GPU-hours, full search 3 PDE × 3 seeds ≈ 45 GPU-hours). Pilot 提供了 honest baseline calibration. 但做成 top-venue 级结果依赖三个未验证条件: (a) localized PDE 上 EMA-QBC 显示 variance/rank 改善——当前证据全部在 Poisson 上且均负面, localized PDE conjecture 纯理论推理; (b) diagnostic depth 足以支撑独立发表——若所有 PDE 均 NEGATIVE, "reward non-stationarity dominates" 的 insight 需有机制解释而非仅 "it didn't work"; (c) EMA collocation set 机制不同于 EMA loss weighting, 对 reward 排序的稳定性未经测试. 三个条件合情但不确定. Claude 与 codex 均判定 Medium, 无分歧.
  - **Impact = Medium**: 最乐观情形 (EMA-QBC 成功): 为 FEX 提供一个可插拔采样模块, 验证 QBC 在 expression tree search 中的适用性. NEGATIVE 情形: 证明 FEX reward 对 collocation robust, 将 FEX community 的 effort 从 sampling 重新导向 structure exploration (NEGATIVE 的 impact 可能高于 POSITIVE). 无论方向, impact 局限在 FEX/SR niche, 不改变更广泛 ML/PDE 社区的研究路线. Claude 与 codex 均判定 Medium, 无分歧.

## Overall

- Priority: Medium
- Score: 5
- Comments: v3 是经过三轮 refine 和 pilot 校准后的成熟 diagnostic idea. 核心进步: (1) pilot 执行并诚实报告 NEGATIVE, 将 idea 从 wishful thinking 拉到 evidence-grounded level; (2) narrative 从 "apply X to Y" 升级为 QBC 作为 reward-design stress test; (3) Gate 0 提供清晰的 cheapest falsifying signal. 核心风险: 若 localized PDE Gate 0 也失败 (这是合理的可能), 剩下的是 "FEX reward is robust to collocation noise in tested settings" 的 narrow diagnostic, 其发表价值取决于能否提供机制解释而非仅实证否定. 建议: 下一轮实验必须优先跑 localized PDE fixed-candidate probe (Gate 0), 包括 residual-only baseline 的同口径 variance/rank 对照. 若 Gate 0 证实 reward-error CV 和 rank stability 在 localized PDE 上仍无改善, 应果断放弃 method claim, 将 idea 重构为 pure diagnostic paper (alternative framing 建议的 "falsification test" 叙事), 并评估该 diagnostic 的期刊发表可行性.

</review>
