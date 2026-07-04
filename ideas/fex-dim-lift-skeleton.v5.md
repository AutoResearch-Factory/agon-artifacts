<!-- 书写报告使用中文 -->
---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-dim-lift-skeleton/
---

- One-sentence summary: 将低维 FEX 搜索得到的 skeleton 先做 searchability audit, 再用有限可交换宏 grammar、系数交换性和高维 PDE probe 认证是否可以提升到高维并只优化连续参数。
- Hypothesis: FEX 的一部分高维成功来自两个同时成立的条件: 解落在可交换宏类里, 且 RL controller 在低维能稳定搜到该宏的 skeleton。Poisson 和 radial 这类稳定家族应通过四道门并在 d=100 仍能 parameter-only lift; pairwise 解析宏本身可 lift, 但当前 FEX 搜索若多 seed 找不到它, 应被拒绝而不是被强行 lift。
- Expected outcome: 成功信号是 Poisson/radial 多 seed 通过四道门, d=100 probe 仍低于 1e-3, asymmetric control 被拒绝, pairwise 保持"analytic liftable but FEX-unsearchable"的诊断结果。最便宜的反证是 stable family 的 d=100 probe 失败, 或 asymmetric/pairwise failed-search 输出被 gate 误通过。
- Contribution type: method+empirical-finding (保留 v4 类型; method 是四门 certificate/audit pipeline, empirical-finding 是 searchability 与 liftability 可分离这一机制发现; topic 未限制 contribution types)
- Risk: MEDIUM
- Estimated effort:
  - Compute: 120-180 GPU-hours, 主要用于 5 seeds x 4 families x depth sweep、d in {10,20,50,100} lift 验证、from-scratch FEX/FEX-PG/HD-TLGP/PSR-style matched baselines 和 gate ablation
  - Data: available; PDE collocation 和 boundary points 解析生成, 无外部数据集或模型权重
  - Implementation: 2-3 weeks
- Novelty quick-check: PSR 是最近竞品, 但它走 projection-based decomposition 加 symbolic program synthesis, 不做 exchangeability certificate, 也不做 parameter-only lift/reject 决策。HD-TLGP 需要已知 1D 解析解、硬编码扩展和 GP 全搜索, 只到 d=3。GSR 的 permutation invariance 是表达式内部的交换律表示, 不是跨维度宏推断。NMIPS 是同维 PDE family 参数迁移。现有工作没有把 FEX cross-dimension lifting 写成"低维搜索稳定性 + 宏可交换性 + 高维 probe"的认证问题。
- Strongest objection: 宏 grammar 是手设的有限子语言, 所以本文不能解释所有 FEX 高维行为; 它只能认证一批明确的 exchangeable PDE families, 并把失败归因给搜索稳定性或宏不匹配。
- Why we should do this: 这个项目把 FEX 的无 CoD 近似理论和实际 RL 搜索行为之间的缺口变成可测问题。正结果给出一个可部署的 lift/reject 程序; NULL 结果也能指出 FEX controller 哪些可 lift 结构搜不到。
- Pilot:
  - Setup: local RTX 4060 Ti; 使用真实 FEX depth1 搜索输出做 Poisson/radial macro inference, 用 analytic pairwise/asymmetric controls 做机制对照, 并新增 d=100 probes 与 radial from-scratch FEX smoke baselines
  - Metric: Gate 1 记录 family-level search stability; Gate 2 要求 low-d macro rel-RMSE < 0.02; Gate 3 要求 additive coefficient CV < 0.1; Gate 4 要求 selected macro 的 high-d probe rel-L2 < 1e-3 且 top-k probe 不以 2x margin 推翻低维最佳宏
  - Result: Poisson real FEX seed1 在 d=100 选 `sum_x2`, rel-L2 2.13e-8, wrong top-k macros 为 0.998 和 0.00293。Radial real FEX seed2 在 d=100 选 `square_sum_x2`, rel-L2 0.0, wrong top-k macros 为 0.995 和 0.103。Analytic pairwise d=100 选 `pairwise_xx`, rel-L2 9.23e-7, 但真实 pairwise FEX 仍是 0/3 搜索失败。Asymmetric d=100 被拒绝, low-d rel-RMSE 0.174 且 CV 0.471。Radial d=10 from-scratch FEX 可成功, rel-L2 4.39e-4 in 81.1s; radial d=100 short-budget FEX smoke 失败, rel-L2 0.169 in 92.7s, 而 lifted radial d=100 只需 0.64s probe 且 rel-L2 0.0。
  - Signal: POSITIVE but bounded. v5 支持"通过认证后可到 d=100"和"single d=10 不是主要卖点"; 论文价值应写成 certification、amortization 和 searchability audit, 而不是声称 from-scratch FEX 在所有高维都失败。

- Review handling (v4->v5):
  1. Matched baselines 缺失: ACCEPT。v5 增加 radial d=10 from-scratch FEX 和 d=100 short-budget smoke; 完整 FEX-PG/HD-TLGP/PSR-style 仍是 P0 实验, 不把它写成 benchmark contribution。
  2. d=100 证据不足: ACCEPT。v5 新跑 Poisson/radial real-FEX d=100 probes, analytic pairwise d=100, asymmetric d=100 rejection。
  3. Gate 1 太 qualitative: ACCEPT。正文把 Gate 1 写成 family-level audit, proposal 阶段用 >=2/5 seeds passing 作为 stable family 的操作阈值; idea pilot 只报告当前 3/2/3 seeds。
  4. Pairwise 0/3 可能被看成方法失败: PUSHBACK on framing, ACCEPT on clarity。它确实限制自动 selector recall, 但 analytic pairwise d=100 rel-L2 9.23e-7 证明宏可 lift; 失败来自 controller searchability, 这正是经验发现。
  5. d=10 from-scratch FEX 也能成功: ACCEPT。v5 去掉"single target dimension 加速"暗示, 改成跨维度摊销和 d=100 认证。
  6. Macro grammar 手工设定: PUSHBACK on "must be general symmetry discovery"。本文不做通用 symmetry discovery; 合理 claim 是有限宏类上的 certificate。ACCEPT 范围限制, 在 strongest objection 和 claims 里明确写出。
  7. Weight tying / coefficient exchangeability: ACCEPT。Gate 3 保留为独立 gate; additive 类要求 CV < 0.1, 非 additive 宏用高维 probe 和 family-specific validation 控制。
  8. Contribution drift check: v4->v5 仍是 `method+empirical-finding`, 没有删除 v4 的任何 contribution type; topic 没有 `preferred-contribution-types`。
- Claims and Claims matrix: POSITIVE claim: 对于落在有限 exchangeable macro grammar 且低维 FEX 多 seed 稳定找到 skeleton 的 PDE family, 四门认证后可以安全提升到 d=100 并用 parameter-only optimization 达到低误差, 同时节省重复高维搜索。NULL claim: analytic 宏可 lift 但真实 FEX 多 seed 搜不到, 论文转为 FEX searchability diagnostic。NEGATIVE claim: stable family 的 high-d probe 失败或 asymmetric false positive 出现, method claim 失败。
- Narrative: 论文叙事更适合写成 FEX scalability 的 certificate and audit, 而不是新的高维 PDE solver。四道门分别回答"FEX 是否搜得到"和"搜到的 skeleton 是否该 lift"; pairwise 结果说明这两个问题可以分开。
- Experiments: P0: 5 seeds x Poisson/radial/pairwise/asymmetric, d in {10,20,50,100}, from-scratch FEX/FEX-PG/HD-TLGP/PSR-style matched baselines, four-gate ablation, and amortization break-even. P1: depth-1/2/3 sweep, weight tying vs independent coefficients, heldout-family threshold calibration, and larger finite macro grammar checks. Extra datasets/settings are evidence for method claims, not a benchmark or application contribution.
- Assets status: v5 d=100 probes and radial baseline smokes are complete locally; code/data handoff is in `workspace/fex-dim-lift-skeleton/data/MANIFEST.md`.

<review date="2026-06-17">

## Novelty

- Score: 9/10
- Closest prior work: PSR (Cao et al., ICLR 2026 submitted, OpenReview #15825) — projection-based decomposition + per-projection SR + symbolic program synthesis; HD-TLGP (Cao et al., AAAI 2024) — GP-based 1D→3D structural transfer with hard-coded macros, requires known 1D analytic solution; GSR (Xiang et al., NeurIPS 2025) — within-expression commutativity normalization, not cross-dimension macro inference; DSO (Hayes et al., LLNL 2025) — deep symbolic optimization framework with RSPG, no cross-dimension lifting; NMIPS (2602.11630) — same-dimension PDE family parameter transfer. 本轮独立检索（arXiv 2025-2026, ICLR 2026, ICML 2026, NeurIPS 2025 全窗口）确认：(1) 四门认证系统在文献中完全空白——没有任何工作将 cross-dimension lifting 表述为 gate-verified certification problem；(2) searchability 与 liftability 的独立性区分未被任何现有工作提出或检验；(3) parameter-only high-d lift 的路线（一次低维 search + gate audit + 参数优化 vs PSR 的 K 次 per-projection SR + 高层符号合成）在 cost structure 上根本不同。PSR 仍是最接近的 prior，但机制完全不同：PSR 做 projection decomposition + symbolic composition，我们做 invariant macro inference + gate certification + parameter-only lift。
- Key differentiator: **Four-gate certification pipeline (search stability / low-d fit / coefficient exchangeability / top-k PDE probe) → parameter-only high-d lift, with searchability ≠ liftability as a mechanism discovery.** 该发现在 v5 中被 d=100 probes 和 pairwise 0/3 analytic 对照进一步强化：pairwise macro analytic d=100 rel-L2 9.23e-7 证明结构本身可 lift，但真实 FEX controller 三 seed 均搜不到——searchability 和 liftability 是可分离的两个维度。无新增 threatening prior work。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | 四门逻辑结构清晰且各司其职。主要 gap：(a) "certificate"本质上是经验 gate audit 而非形式化保证——每道门的阈值（rel-RMSE < 0.02, CV < 0.1, rel-L2 < 1e-3）是 experience-driven 而非 theoretically derived；(b) searchability ≠ liftability 目前仅有 pairwise 一个 counterexample 支撑，作为"机制发现"需要更多 family 证据来建立一般性；(c) 非 additive 类宏的 coefficient exchangeability 逻辑仍偏 heuristic——Gate 3 当前对 additive 类有清晰 CV criterion，但对更复杂宏（如含 cross-term 的宏）缺乏相应的 exchangeability 判据；(d) top-k probe 的 "2x margin" 推翻阈值依赖 heuristic 而非经验校准（heldout-family calibration 仍是 P1 计划）。这些 gap 不会推翻核心逻辑，但会限制顶会审稿人对"certification"严格性的认可。 |
| Missing evidence signals | 7/10 | v5 pilot 相比 v4 有实质进步：Poisson/radial/pairwise/asymmetric d=100 probes 全部完成，radial d=10 from-scratch FEX 和 d=100 short-budget smoke baselines 已补。但顶会论文级关键证据仍有多项在计划阶段：(1) fair matched baselines（FEX-PG、HD-TLGP、PSR-style）同预算对照——这是审稿人最可能要求的关键证据，d=100 smoke failure 是 useful signal 但不能替代 full comparison；(2) 5-seed 全覆盖（当前 1-3 seeds/family）；(3) d={20, 50} 中间维度 scaling；(4) depth-1/2/3 sweep；(5) gate ablation（区分各 gate 的独立贡献）；(6) amortization break-even 曲线；(7) heldout-family threshold calibration。其中 (1) 是最紧迫的 missing signal：没有 fair baseline comparison，审稿人无法判断 parameter-only lift 是否真的比 from-scratch FEX 或 PSR-style decomposition 更 efficient（而不仅是更 elegant）。 |
| Narrative | 8/10 | v5 叙事延续 v4 的"certificate and audit"框架并进一步加强。pairwise 0/3 从失败转为诊断证据的叙事翻转依然有力。d=100 证据使 narrative 从"d=10 加速"升级为"d=100 认证"，消除了 v4 的一个脆弱点。minor weakness：(a) pairwise 0/3 既是诊断发现也是 automatic selector recall 的硬限制——论文叙事不能完全回避"方法对非 separable 族 recall=0"这一弱点；(b) "certificate" 一词可能给审稿人造成"形式化保证"的期待，实际是经验 audit，需要在 intro 中明确 scope；(c) 从"FEX scalability"到"certificate and audit"的过渡需要更平顺的 narrative bridge。 |
| Venue contribution | 7/10 | Topic 未声明 target-venue，按 topic 类型（RL for Symbolic Regression / Scientific ML）推断为 NeurIPS/ICML/ICLR。核心贡献（searchability ≠ liftability 机制发现 + 四门认证系统）是 genuinely new，不是 incremental solver work。但需注意：(a) method 本身（四道经验 gate）偏简单——真正 novelty 是机制发现而非新 solver，审稿人可能认为 technical contribution 不够厚重；(b) 有限 macro grammar（3 个宏）限制 contribution 宽度——与声称"通用 symmetry discovery"的工作（如 GSR）相比范围较窄，但 honest scoping 优于 overclaim；(c) 若 fair baseline comparison 显示 lift 无显著 cost advantage，contribution 会大幅降级。在当前证据状态下，idea 有 top-venue 路径但还不足以直接投稿：需要 fair baselines 和更宽 family/depth 证据把 feasibility signal 转化为 conclusive evidence。 |
| Testability | 9/10 | Falsifiability 极强且多层级。最便宜的反证（stable family d=100 probe 失败，或 asymmetric false positive）判定标准明确（rel-L2 > 1e-3, CV < 0.1 等），pilot 已通过此测试。四道门各自可独立消融，falsifiability 模块化。三个 outcome（POSITIVE/NULL/NEGATIVE）边界清晰且 pre-registered。minor point：Gate 1 的 ">=2/5 seeds" 阈值在 proposal 阶段才最终确定，当前 pilot 的 3/2/3 seeds 分布尚未统一到 5-seed 判定。 |
| Outcome realism | 8/10 | POSITIVE outcome（Poisson/radial 多 seed 通过四门，pairwise 被正确拦截，asymmetric 被拒绝）的 pilot 基础坚实——v5 新增 d=100 证据（Poisson rel-L2 2.13e-8, radial 0.0, pairwise analytic 9.23e-7）是强 positive signal。NULL outcome 同样 realistic：pairwise 已是 0/3，若 5-seed 仍为 0/5 则自动 selector recall=0 的可发表性更高。NEGATIVE outcome 概率在 pilot 后已很低。主要不确定性：matched baselines 可能显示 lift 无显著成本优势——这是最可能导致 contribution 降级的风险。120-180 GPU-hour 估计对完整实验矩阵合理。 |
| Contribution type compliance | n.a. | Topic `0616-fex.md` 未声明 `preferred-contribution-types`; 跳过本检查。 |
| Overall Quality | 7.5/10 | v5 在 v4 基础上继续强化证据（d=100 probes, radial baselines），维持了叙事锐度和机制发现的原创性。与 codex second opinion（Overall Quality 7.3/10）基本一致：idea 成熟且 well-formed，但其 value proposition 仍依赖于后续 fair baseline comparison 和更宽 family/depth 证据来验证。当前处于"强诊断 proposal"向"可投稿顶会 claim"的过渡阶段——pilot 证据已排除最大风险（d=100 probe 失败），但尚未证明 lift 的实际增益优于 from-scratch FEX 或 PSR-style alternatives。 |

## Contribution Drift

- v_4 contribution types: {method, empirical-finding}
- v_5 contribution types: {method, empirical-finding}
- Status: unchanged. Method 内容从"四 gate certification（含 search stability pre-gate）"维持为"四门 certificate/audit pipeline"，empirical-finding 从"检验 searchability 与 liftability 的独立性"进一步强化为"用 d=100 probes 和 from-scratch FEX baselines 证明两者可分离"。贡献类型集合未变，未删除任何类型。
- Hard cap triggered: no. Topic 未声明 `preferred-contribution-types`。

**v4 review concern 追踪:**

1. Gate 1 qualitative（v4 concern a）: **partially resolved**。v5 写入 proposal 阶段 `>=2/5 seeds passing` 操作阈值，但 pilot 仍未完成统一 5-seed 判定。
2. Weight-tying argument 适用范围有限（v4 concern b）: **partially resolved**。Gate 3 保留 CV < 0.1 threshold 并明确 additive 类适用，但 tied vs independent coefficient ablation（P1-7）仍未执行。
3. Heldout-family threshold calibration cold-start（v4 concern c）: **deferred**。v5 在 P1 实验中提及但未给出具体方案。
4. Matched baselines 缺失（v4 missing evidence #3）: **partially resolved**。v5 新增 radial d=10 from-scratch FEX（rel-L2 4.39e-4）和 d=100 short-budget smoke（rel-L2 0.169），但 FEX-PG/HD-TLGP/PSR-style fair comparison 仍为 P0 计划。
5. d=100 证据缺失（v4 missing evidence #2）: **resolved**。v5 完成 Poisson/radial/pairwise/asymmetric d=100 probes，rel-L2 分别为 2.13e-8、0.0、9.23e-7 和被拒绝。
6. Pairwise 0/3 可能被看作方法失败（v4 narrative concern）: **partially resolved**。v5 framing 更清晰，但审稿人仍会将其视为 automatic selector recall 的硬限制——论文需要明确"这既是机制发现，也是方法边界"。
7. d=10 from-scratch FEX 也能成功（v4 narrative concern）: **resolved**。v5 去掉 single-d acceleration claim，改为跨维摊销和 d=100 certification。
8. Macro grammar 手工设定（v4 narrative concern）: **partially resolved**。范围限制已写清楚，pushback 有据（不做通用 symmetry discovery），但顶会 contribution 宽度仍受限于 3-macro grammar。
9. Depth sweep（v4 deferred）: **deferred**。仍为 P1 计划。
10. Gate ablation（v4 deferred）: **deferred**。仍为 P1 计划。
11. Break-even 摊销曲线（v4 deferred）: **deferred**。仍为 P1 计划。
12. Top-k probe 阈值校准（v4 missing evidence #7）: **deferred**。v5 仍用 heuristic "2x margin"，heldout-family calibration 仅计划。

**v5 新增关注点:**
- searchability ≠ liftability 目前仅 pairwise 一个 counterexample 支撑，需要更多 non-separable family（如 triplewise、cross-term 类）来建立一般性。
- "certificate" 一词可能造成形式化保证的期待——建议在论文 intro 中明确写为 "empirical gate audit" 以避免误导。
- 非 additive 宏的 coefficient exchangeability 逻辑需要 family-specific validation（v5 已提及但未给出具体方案）。

## Alternative Framing

当前 framing（"exchangeability certificate and searchability audit for FEX scalability"）已接近最优。建议进一步锐化为：**"Certifying FEX Cross-Dimension Generalization: An Exchangeability Audit with Searchability Diagnostics"**——突出"audit"（审计）+ "diagnostics"（诊断）两个操作，比"certificate"更准确（因为是经验 gate 而非形式化保证），同时将 pairwise 0/3 天然融入"diagnostics"叙事。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 对于解落在有限 exchangeable macro grammar {sum_x2, square_sum_x2, pairwise_xx} 中的 PDE 族，当 FEX RL controller 在低维多 seed 稳定恢复 skeleton（Gate 1, >=2/5 seeds），且该 skeleton 通过 low-d fit（Gate 2, rel-RMSE < 0.02）、coefficient exchangeability（Gate 3, additive CV < 0.1）和 top-k PDE probe（Gate 4, high-d rel-L2 < 1e-3 且 top-k 不以 2x margin 推翻）四道门 audit 后，可被安全提升到高维（d 到 100）并仅用参数优化达到低误差。同时，pairwise 宏解析上可 lift（d=100 rel-L2 9.23e-7）但 FEX controller 多 seed 搜不到，构成 searchability 与 liftability 可分离的直接证据。不可支持：声称所有 FEX 高维行为都可被解释，或自动发现任意 symmetry。 |
| NULL | Analytic 宏本身可 lift（pairwise d=100 rel-L2 9.23e-7），但 FEX RL controller 在非 separable 族上多 seed 搜索不稳定（pairwise 0/3 可能扩展到其他 non-separable 族）。此结果作为"FEX search quality diagnostic + gate audit on provably-liftable families"有独立发表价值——认证系统在 stable families 上工作，同时精确指出了 FEX controller 的搜索弱点，为后续 controller 改进提供量化靶点。不可支持：声称 pairwise 0/3 证明所有 non-separable 族都不可 search（单一 counterexample 不足以建立一般性）。 |
| NEGATIVE | Stable families（Poisson/radial）多 seed 大面积搜索失败，或 asymmetric control 被误通过（false positive），或 top-k probe 频繁推翻 low-d selection。此时 method claim 失败，论文贡献转为负结果——排除"可交换宏结构认证可指导 FEX 高维 lift"这一假说，并记录四门 audit 系统在哪些条件下失效。不可支持：在 NEGATIVE outcome 下仍声称 four-gate framework 有实际价值（除非能证明 gate 失败本身具有诊断意义）。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium（Claude 与 codex 一致）**：v5 pilot 提供了 d=100 层面的强 feasibility signal（Poisson 2.13e-8, radial 0.0），到顶会论文的路径清晰。但若干关键条件仍需验证：fair matched baselines 对照（最关键的剩余风险——parameter-only lift 是否真的比 from-scratch FEX 和 PSR-style decomposition 在同等 compute 下更优）、depth sweep（可能暴露深度依赖的 liftability 变化）、gate calibration 在更宽 family 上的泛化性。这些不是纯工程执行——matched baselines 可能显示 lift 无显著优势，depth sweep 可能发现 liftability 随树深退化。**Impact=High（Claude 与 codex 一致，无 >=1 level 分歧）**：codex 与 Claude 均判 Impact=High。若成功，(a) searchability ≠ liftability 是 genuine mechanism discovery——改变了对 FEX 高维行为的理解方式；(b) 四门 audit 提供可操作的、边界明确的决策程序；(c) 缩小了 FEX 无 CoD 近似理论与实际 RL 搜索行为之间的 gap；(d) NULL outcome 同样有清晰发表路径。Impact 限于 FEX/SR-for-PDE 路线而非 general ML，故不达 Exceptional。综合 Priority=High (7)，score 与 v4 持平——v5 的实质改进（d=100 evidence, radial baselines, refined framing）被 baseline risk（fair comparison 尚未执行）和 evidence breadth（单一 counterexample for searchability ≠ liftability）平衡。

## Overall

- Priority: High
- Score: 7
- Comments: v5 在 v4 基础上增强了两个关键信号（d=100 probes 和 from-scratch FEX baselines），维持了叙事锐度和机制发现的原创性。Claude 与 codex 在 Likelihood（Medium）和 Impact（High）上无 >=1 level 分歧，score 一致为 7。v4 review 的 12 条 concern 中：2 条 resolved（d=100 evidence, d=10 reframing），6 条 partially resolved（Gate 1 qualitative, weight-tying, matched baselines, pairwise framing, macro grammar scope, fair baseline signal），4 条 deferred（depth sweep, gate ablation, break-even, threshold calibration）。v5 新增关注：searchability ≠ liftability 需要更多 family 验证一般性；"certificate" 措辞可能造成形式化保证期待。**下一个关键决策点：Proposal 阶段必须执行 fair matched baselines 对照（特别是 PSR-style 和 from-scratch FEX）以验证 lift 的实际 cost advantage，否则顶会审稿人会质疑"parameter-only lift 是否真的比直接高维搜索更 efficient"。若无 matched baseline 优势证据，score 应下调至 Medium (5)。** Contribution types 未漂移，topic 未声明 preferred-contribution-types，无 hard cap 触发。文件约 7 KB（不含本 review），未超限。

#if file_size_kB > 10   // 不含本次 review 块
超长警告: 文件约 7 KB（不含本 review）, 未超限
#endif

</review>

## Deep Lit 2026-06-19 — idea scope (5 篇)

以下 5 篇论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-dim-lift-skeleton` 于 2026-06-19 系统性搜索并精读后收录。搜索覆盖 method/application/data/evaluator/failure-mode/competition 6 axes + B7 反向扩展（20 调用）。Round 1 后 B7 产出 0 篇新候选，文献饱和终止。

### NMIPS (2602.11630) [wiki](wiki/2602.11630.md)
Yipeng Huang, Dejun Xu, Zexin Lin, Zhenzhong Wang, Min Jiang, 2026.02. 用 multifactorial optimization + C-ADF/Karva 编码 + affine population transfer 同时求同一 PDE family 内多参数实例的解析解。覆盖 1D-3D Advection/Burgers/Advection-Diffusion/Taylor-Green NS 等 6 类 PDE。**与本 idea 的关键 delta**：NMIPS 做固定维度内的参数族迁移，本 idea 做跨维度 skeleton lifting + gate certification。NMIPS 用 GP+MFO 全搜索，本 idea 用 FEX RL + parameter-only lift。NMIPS 的 affine transfer 可作为 amortization baseline。**撞车风险: 低**——机制路线正交（same-dim parametric transfer vs cross-dim skeleton lift/reject），但应在 Novelty 段明确 delta。

### M-Tensor Format (2602.08509) [wiki](wiki/2602.08509.md)
Rémi Cloarec, S. Rodriguez, Xavier Kestelyn, F. Chinesta, 2026.02. 用行级张量积和 m-product 把可分变量特征展开为 m×m 核/Gram 系统，在稀缺数据下做高维非线性回归。覆盖 Rosenbrock 20-300 维、Lorenz、Kuramoto。**与本 idea 的关系**：提供了 separated-variable 结构在高维回归中 scale 的理论语言——FEX 的 exchangeable macros (sum_x2, square_sum_x2) 本质上就是利用加性/乘性可分结构。ALI regularization 可启发 collocation 选择。**撞车风险: 无**——不做 RL 表达式树搜索或 PDE residual。

### KAPI: Meta-Learned Basis Adaptation (2604.09289) [wiki](wiki/2604.09289.md)
Vikas Dwivedi, Monica Sigovan, Bruno Sixou, 2026.04. 参数条件网络把 PDE 参数映射为 Gaussian RBF basis geometry，再交给 PIELM 风格 least-squares corrector。覆盖 diffusion/advection-diffusion/variable-speed transport 四类 linear PDE family。核心发现：corrector 常把误差降 1-3 个数量级，但窄脉冲外推变差。**与本 idea 的关系**：predictor→corrector 的两阶段管线类似 low-d FEX search→parameter-only lift，且 "inherited/refinement/background basis" 三层资源分配思路可启发 Gate 4 的 top-k probe 设计。**撞车风险: 无**——不做符号表达式/有限宏 grammar/gate certification。

### SCaSML (2504.16172) [wiki](wiki/2504.16172.md)
Zexi Fan, Yan Sun, Shihao Yang 等, 2025.04. Defect correction 框架：把预训练 PDE surrogate 的误差定义为 defect，推导保结构的缺陷方程，推理时用 MLMC/Picard 校正。实验覆盖 linear convection-diffusion, viscous Burgers, LQG/HJB, diffusion-reaction，维度最高 160，相对 L2 误差常下降 20-80%。**与本 idea 的关系**：展示了 neural surrogate 在 160-dim 的可校准性——FEX 的 parameter-only lift 若要声称高维优势，需要与此类 defect correction 方法对比 cost/accuracy。可作为高维 PDE 校准 baseline。**撞车风险: 无**——不做符号搜索。

### DMCI (2606.09930) [wiki](wiki/2606.09930.md)
Lucas Sheneman, 2026.06. 把 self-hosting Scheme 解释器编译为 PyTorch 可微模块，LLM 生成的 Scheme 程序进入同一解释器后连续参数通过 autograd 精确校准。覆盖 battery capacity-fade, ENSO Kalman-NLL 等案例。**与本 idea 的关系**：概念上平行于 FEX 的 "skeleton search + parameter optimization"——DMCI 是 "program proposal + gradient calibration"。但 DMCI 操作一般程序而非 PDE 表达式树。可作为 program+parameter 分离的 design pattern 引用。**撞车风险: 无**——不涉及 PDE residual/gate certification/cross-dimension lifting。

### 撞车风险更新

本轮搜索进一步确认：没有任何现有工作将 FEX cross-dimension lifting 表述为 gate-verified certification problem。NMIPS（最近邻）做固定维度内的参数族 affine transfer，方向正交。本 idea 核心 claims 全部 unchallenged。

