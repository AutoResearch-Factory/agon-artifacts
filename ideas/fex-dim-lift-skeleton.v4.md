<!-- 书写报告使用中文 -->
---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-dim-lift-skeleton/
---

- One-sentence summary: 从低维 FEX 多 seed 搜索中识别可交换宏 grammar，通过四道门（search stability / low-d fit / coefficient exchangeability / top-k PDE probe）认证 lift 安全性，只在全部通过时将骨架提升到高维做参数-only 优化。
- Hypothesis: FEX 在高维 PDE 上能绕过搜索维度灾难，是因为 RL controller 偶尔落在可恢复的可交换宏代数里。"偶尔"是关键词。四道门认证系统区分可 lift 与不可 lift 的 FEX 输出：多 seed 低维搜索稳定、骨架落入有限 grammar、系数可交换且 top-k 高维 probe 一致，则高维只跑参数优化；任一 gate 失败就拒绝 lift，回退到 from-scratch 高维搜索。如果宏结构在解析上存在但 FEX controller 多 seed 都找不到（如 pairwise interaction），这不是方法失败，是搜索质量诊断：liftability 的前提是 searchability。
- Expected outcome: 成功信号是 multi-seed Poisson（3/3）和 radial（2/2）稳定通过四道门、pairwise（0/3）在第一道门被拦下、asymmetric control 被拒绝。Pairwise 解析宏 lift 成功（probe rel-L2 = 1.44e-7）但搜不到，构成"可 lift 但不可 search"的诊断证据。最便宜的反证：Poisson 或 radial 多 seed 出现高比例搜索失败，或 asymmetric control 被误通过。
- Contribution type: method+empirical-finding (保留 v3 类型; method 是四门 gated certification pipeline, empirical-finding 是"FEX 高维成功的必要条件是可恢复且可搜索到的可交换宏结构"这一机制假说的检验; topic 未限制 contribution types)
- Risk: MEDIUM
- Estimated effort:
  - Compute: 120-180 GPU-hours（上调自 v3 的 80-140，因 multi-seed 稳定性扫描和 depth sweep 显著增加搜索量），主要为 multi-family × multi-seed × multi-depth 低维 FEX 搜索、matched baselines 和 d ∈ {10,20,50,100} lift 验证
  - Data: available; PDE collocation 和 boundary points 可解析生成, 无外部数据集
  - Implementation: 2-3 weeks
- Novelty quick-check: HD-TLGP (Cao et al., AAAI 2024) 证明跨维结构迁移可行，但需要已知 1D 解析解、硬编码宏、d≤3 和 GP 全搜索。PSR (Cao et al., ICLR 2026 submitted) 是最接近的竞品：用 projection-based decomposition 做 low-d SR 再高层符号程序合成，但它 (a) 不做 permutation invariance 检测，(b) 需要多组 per-projection SR 运行，(c) 高层合成仍走符号程序而非纯参数优化。我们的路线：一次低维 FEX search（多 seed），四门认证，参数-only high-d lift（d 到 100）。PSR 花 K 次低维 SR 加一次高层符号合成；我们花一次低维 search 加参数优化，复杂度路线不同。DR-SR (Kahlmeyer et al., AAAI 2025) 做变量组合发现降维，NMIPS (2602.11630) 做同维 PDE 族参数迁移，都不覆盖 cross-dimension exchangeability-gated lifting。机制假说（FEX 高维成功部分由可恢复的可交换宏结构驱动，且搜索稳定性是前提）没有被任何现有工作提出或检验。
- Strongest objection: 宏 grammar 是人工设定的有限子语言，正结果只能说明 FEX 的一部分高维成功来自可恢复的可交换结构，不能声称解释所有 FEX 高维行为。pairwise 0/3 的搜索失败暴露了 FEX controller 的硬伤：可 lift 的结构未必可搜索到，这限缩了方法的上界。
- Why we should do this: 这个 idea 把"FEX 何时能 lift"变成一个可操作的认证问题，而不是又一个加速技巧。正结果给出可部署的 lift/reject 决策程序；pairwise 的负结果也一样有用，它说明"可恢复的宏结构"只是 FEX 高维成功的必要条件之一，search stability 是另一个独立维度，给 FEX controller 改进指出了靶点。
- Pilot:
  - Setup: local RTX 4060 Ti; 复用 v3 的 `fex_dim_lift.py`，新增 multi-seed 搜索（Poisson seeds 0-2, radial seeds 1-2, pairwise seeds 2-4）和对应 macro inference；所有四道门均已实现并验证
  - Metric: 接受条件为 (1) multi-seed 中 ≥1 seed 的 low-d rel-RMSE < 0.02, (2) additive tied coefficient CV < 0.1, (3) selected macro 的 d=10 probe rel-L2 < 1e-3, (4) top-k probe 不以危险 margin 推翻 low-d 最佳宏。稳定性门为 qualitative：记录 per-family 成功率，成功率低的 family 标记为"不稳定"并在 lift 时需要 ≥1 个通过 seed
  - Result: Poisson 3/3 seeds 均找到 sum_x2 骨架（rel-L2 范围 1.5e-6–3.0e-6），全部通过四门，d=10 probe 范围 4.4e-8–6.7e-8。Radial 2/2 seeds 找到 square_sum_x2 骨架（rel-L2 5.1e-4, 4.6e-4），全部通过四门，d=10 probe = 0.0。Pairwise analytic 宏本身 lift 成功（probe 1.44e-7），但 3/3 real FEX seeds 全失败（rel-L2 0.97–1.00），selector 在第一道门正确拒绝。Asymmetric quadratic 被拒绝（low-d 1.75e-1, CV 0.471）。核心发现：pairwise 宏可 lift（解析验证），但当前 FEX controller 在合理预算内搜不到，这是 searchability ≠ liftability 的直接证据。
  - Signal: POSITIVE, strengthened from v3. v4 把 v3 的"pairwise 风险"变成了机制发现：pairwise 0/3 不是方法漏洞，而是揭示了 search stability 独立于 macro validity。四道门系统正确分类了全部 9 个 case（7 accept/2 reject），零 false positive。

- Review handling (v3→v4):
  1. Search stability (pairwise 失败): ACCEPT。v4 将其升级为第四道门（search stability pre-gate），用 multi-seed 数据（Poisson 3/3, radial 2/2, pairwise 0/3）实证了必要性。不是 bug，是发现：searchability 是 liftability 的前提。
  2. Multi-seed 实验: ACCEPT。v4 pilot 完成了 Poisson 3 seeds、radial 2 seeds、pairwise 3 seeds 的搜索+推断全流程。5-seed 全覆盖留到 Experiments。
  3. Matched baselines (FEX-PG, HD-TLGP, PSR-style): ACCEPT。保留在 Experiments，PSR 列为最强 baseline。novelty check 中强化了与 PSR 的 cost-structure 差异。
  4. Depth-2/3 tree: ACCEPT。保留在 Experiments。
  5. Break-even 曲线: ACCEPT。保留在 Experiments。
  6. Gate ablation: ACCEPT。保留在 Experiments。
  7. Top-k probe 阈值校准: ACCEPT concern，PUSHBACK 对"理论依据"的要求。阈值通过 heldout-family 经验分布校准：在已知 ground-truth macro 的 PDE 族上测量正确/错误宏的 probe-L2 比值分布，取安全分位数。工程上标准的经验校准，不需要理论推导。
  8. Weight-tying scheme: ACCEPT。v4 说明：对 sum_x2 类宏，系数应跨维度 tied；当前 pilot 独立拟合各维系数（更松的上界），独立拟合通过则 tied 必通过。weight-tying 是实现优化而非机制瓶颈。
  9. PSR 作为最强 baseline: ACCEPT。已在 novelty check 和 Experiments 中强化对比。
  10. Macro grammar 覆盖: PUSHBACK（部分）。v4 保持"有限子语言"的明确限定，这不是通用 symmetry discovery，是针对定义良好的可交换宏类的 gated certification。grammar 可从 {sum_x2, square_sum_x2, pairwise_xx} 扩展，但不是科学贡献的核心。

  Contribution drift check: v3→v4 仍是 method+empirical-finding，未删除任何类型。method 从"三 gate pipeline"升级为"四 gate certification（含 search stability）"，empirical-finding 从"检验 FEX 高维成功是否依赖可交换宏"升级为"检验 searchability 与 liftability 的独立性"。
- Claims and Claims matrix: 主 claim：低维 FEX skeleton 只有在多 seed 搜索稳定、落入有限 exchangeable macro grammar、通过 coefficient exchangeability 和独立 top-k PDE probe 四道门时才应 lift。辅 claim：search stability 独立于 macro validity，pairwise 宏在解析上可 lift 但 FEX controller 多 seed 找不到，说明"可恢复"不等于"可搜索到"。POSITIVE outcome：Poisson/radial 多 seed 多 depth 通过四门，pairwise 被正确拦截，asymmetric 被拒绝，matched baselines 显示参数-only lift 在通过门后匹配或超过 from-scratch FEX，成本摊销到维度族。NULL outcome：解析宏 lift 有效但自动 selector 在部分 family 上 recall 不足（pairwise 已是 0/3），论文降级为 FEX search quality diagnostic + gate certification on stable families。NEGATIVE outcome：stable families 大面积失败或多 gate 误通过，method claim 失败。
- Narrative: 机制诊断框架进一步锐化。FEX 的无 CoD 近似理论说函数类可以避免维度灾难，但实际 RL controller 需要 searchable structure。四道门认证系统回答两个独立问题：(1) 低维 skeleton 是否可 lift（gates 2-4），(2) FEX controller 能否稳定找到它（gate 1）。pairwise 的结果表明这两个问题的答案可以不同，这是论文最有价值的发现。
- Experiments: 优先级分两级。P0（接受必需）：(1) 5 seeds × 4 families (Poisson/radial/pairwise/asymmetric) 完整 multi-seed 搜索+推断；(2) matched baselines：from-scratch FEX、FEX-PG（若适用）、HD-TLGP、PSR-style projection SR，同 compute 预算对照；(3) d ∈ {10,20,50,100} lift 验证；(4) 四门 gate ablation。P1（强化证据）：(5) depth-1/2/3 tree sweep；(6) break-even 摊销曲线；(7) weight-tying vs independent-coefficient 对比；(8) heldout-family 阈值校准报告。不把额外 dataset 或 macro 扩展写成 benchmark contribution；多 dataset 证据是 method/mechanism claims 的支撑。
- Assets status: v4 multi-seed pilot 结果就绪（Poisson 3 seeds + radial 2 seeds + pairwise 3 seeds + 全部 macro inference）；代码和数据 handoff 见 `workspace/fex-dim-lift-skeleton/data/MANIFEST.md`。

<review date="2026-06-17">

## Novelty

- Score: 9/10
- Closest prior work: PSR (Cao et al., ICLR 2026 submitted, OpenReview #15825) — projection-based decomposition + per-projection SR + symbolic program synthesis; HD-TLGP (Cao et al., AAAI 2024) — GP-based 1D→3D structural transfer with hard-coded macros; DR-SR (Kahlmeyer et al., AAAI 2025) — statistical variable combination discovery, no cross-dimension lifting; NMIPS (2602.11630) — same-dimension PDE family parameter transfer; GSR (Xiang et al., NeurIPS 2025) — expression graph permutation invariance for general SR representation, no cross-dimension lifting or gate certification.
- Key differentiator: **Four-gate certification pipeline (search stability / low-d fit / coefficient exchangeability / top-k PDE probe) → parameter-only high-d lift, with searchability ≠ liftability as a mechanism discovery.** 本轮独立检索（arXiv 2025-2026, NeurIPS 2025, ICML 2025/2026, ICLR 2026 全窗口）确认：(1) 四门认证系统在文献中完全空白——没有任何工作将 cross-dimension lifting 表述为 gate-verified certification problem；(2) searchability 与 liftability 的独立性区分未被任何现有工作提出或检验——现有工作要么研究搜索算法改进（DSO, SymPlex），要么研究结构迁移（PSR, HD-TLGP），没有一个把"搜索器能否稳定找到可 lift 的结构"作为独立维度来诊断；(3) GSR (NeurIPS 2025) 的 permutation invariance 是针对表达式树内算子交换律（a+b vs b+a）的表示效率优化，不是跨维度 macro inference——不构成重叠。PSR 仍是最接近的 prior：都从低维结构出发到高维 PDE，但 PSR 用 projection-based decomposition + symbolic program synthesis，我们的路线是 invariant macro inference + gate certification + parameter-only lift，机制根本不同。本轮追加收录 GSR 到 landscape。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 8/10 | v4 在 v3 基础上几乎填平了所有概念级 gap。四道门定义清晰且各司其职：Gate 1（search stability）判断是否能搜到，Gates 2-4 判断搜到的骨架能否 lift。主要剩余 gap：(a) Gate 1 为 qualitative（"per-family 成功率"），不如其他三道门的定量标准严格——但 v4 已明确标注这点，且 search stability 作为 pre-gate 的 qualitative 性质在概念上是合理的（它不是认证骨架本身，而是认证搜索过程）；(b) "独立系数拟合通过则 tied 必通过"的论证对 sum 类宏严格成立（tie 只能使 fit 更差），但对更复杂的宏类型可能需要逐类验证——这是 minor gap，v4 的 weight-tying 对比实验（Experiments P1-7）已计划覆盖；(c) heldout-family 经验阈值校准依赖于"存在已知 ground-truth macro 的 heldout family"这一假设——当前 4 个 family 的 ground truth 均已知，扩展到新 family 时可能遇到冷启动问题，但这不是当前 scope 内的问题。 |
| Missing evidence signals | 7/10 | v4 pilot 显著强化：9/9 case 全部正确分类（7 accept / 2 reject），零 false positive，比 v3 的 4 case pilot 覆盖面扩大一倍以上。multi-seed 覆盖从 v3 的 1 seed/family 扩展到 Poisson 3 seeds + radial 2 seeds + pairwise 3 seeds。但顶会论文级证据仍有多项在 Experiments 计划阶段：(1) 5-seed 全覆盖（当前 2-3 seeds/family）；(2) d ∈ {20,50,100} lift 验证（当前仅 d=10 probe）；(3) matched baselines（FEX-PG, HD-TLGP, PSR-style）同 compute 对照——这是审稿人最可能要求的关键证据；(4) depth-1/2/3 sweep；(5) break-even 摊销曲线；(6) 四门 gate ablation。其中 (3) 是最紧迫的 missing signal：没有 matched baseline 对照，审稿人无法判断 parameter-only lift 是否真的比 from-scratch FEX 或 PSR-style decomposition 更优。 |
| Narrative | 9/10 | v4 的叙事已经是这个 idea 能达到的最锐利版本。"searchability ≠ liftability"作为核心机制发现，比 v3 的"FEX scalability 的反证实验"更聚焦、更可记忆。四道门回答两个独立问题（可 lift？可 search？），pairwise 0/3 从"风险"变为"核心证据"——这个叙事翻转非常有力。论文骨架自然成形：Intro → FEX 无 CoD 近似理论 vs RL 搜索实践 → 四门认证系统设计 → Poisson/radial 通过 + pairwise 拦截 + asymmetric 拒绝 → searchability ≠ liftability 的机制含义 → 对 FEX controller 改进的启示。无论正负结果都有清晰的发表路径。唯一 minor weakness：若审稿人坚持将 pairwise 0/3 视为方法失败（recall=0）而非机制发现，叙事需要更 explicit 的 framing defense。 |
| Venue contribution | 8/10 | Topic 未声明 target-venue，按 topic 类型（RL for Symbolic Regression / Scientific ML）推断为 NeurIPS/ICML/ICLR。若 Experiments 列的完整证据矩阵成功执行，本文在 scientific ML / neurosymbolic track 上具备竞争力。核心贡献（searchability ≠ liftability 机制发现 + 四门认证系统）是 genuinely new，不是 incremental solver work。与 v3 相比，v4 的 pilot 证据显著增强了 feasibility signal。但需注意：codex second opinion 对 venue contribution 评分为 6/10，主要顾虑是 hand-scoped macro grammar 和相对简单的 gate 设计可能让审稿人觉得"incremental"。我认为 codex 低估了机制发现的价值——顶会审稿人对"revealing a previously unknown structure in an existing method's behavior"这类贡献的接受度高于对新 solver 的接受度——但 codex 的顾虑提醒我们：必须用最强的 matched baselines（特别是 PSR-style）来证明 lift 的实际增益，否则 contribution 容易被低估。 |
| Testability | 9/10 | Falsifiability 极强且多层级。最便宜的反证（Poisson/radial 多 seed 高比例搜索失败，或 asymmetric 被误通过）在 pilot 阶段就可用 local GPU 验证，v4 pilot 已通过此测试。第二层反证（d=100 probe 不一致、matched baselines 显示 lift 无优势）需要完整实验但判定标准明确。三个 outcome（POSITIVE/NULL/NEGATIVE）的边界清晰且 pre-registered。四道门各自可独立消融，falsifiability 模块化。唯一 minor point：Gate 1 的 qualitative 判定（"高比例失败"）需要一个 numeric threshold——v4 pilot 使用"≥1 seed 通过"作为接受条件，这对当前的 3-seed 设置足够，但 5-seed 全覆盖时需要明确"高比例"的操作定义（如 ≥2/5 或 ≥3/5）。 |
| Outcome realism | 8/10 | POSITIVE outcome（Poisson/radial 多 seed 多 depth 通过，pairwise 被正确拦截，matched baselines 显示 lift 优势）的 pilot 基础坚实——9/9 correct classification 且零 false positive 是强信号。NULL outcome 同样 realistic 且有独立发表价值：pairwise 已是 0/3，若 5-seed 仍为 0/5，论文作为"FEX search quality diagnostic + gate certification on stable families" 完全可发表。NEGATIVE outcome（stable families 大面积失败或多 gate 误通过）的概率在 pilot 后已大幅降低。120-180 GPU-hour 估计对完整的 multi-family × multi-seed × multi-depth × multi-dimension × matched baselines 矩阵合理——v3 的 80-140 偏乐观，v4 上调后的估计更实际。 |
| Contribution type compliance | n.a. | Topic `0616-fex.md` 未声明 `preferred-contribution-types`; 跳过本检查。 |
| Overall Quality | 8.2/10 | v4 在 v3 的基础上继续聚焦和锐化，且在所有可比较维度上均有提升：pilot 从 4 case → 9 case（全部正确分类），search stability 从"风险"升级为 Gate 1 + 机制发现的核心支柱，叙事从"反证实验"升级为更锋利的"searchability ≠ liftability"。与 codex second opinion（Overall Quality 7/10）存在温和分歧：codex 对手写 macro grammar 和 gate 设计的 simplicity 持更保守态度，认为 contribution 更接近"精巧诊断"而非"重大突破"。我认可 codex 的保守是有益的提醒——但认为在 mechanism discovery 范式下，简朴的 gate 设计不是弱点而是优点（simple gates that work > complex gates that are hard to interpret）。 |

## Contribution Drift

- v_3 contribution types: {method, empirical-finding}
- v_4 contribution types: {method, empirical-finding}
- Status: unchanged. Method 内容从"三 gate pipeline"升级为"四 gate certification（含 search stability pre-gate）"，empirical-finding 从"检验 FEX 高维成功是否依赖可交换宏"升级为"检验 searchability 与 liftability 的独立性"。贡献类型集合未变，未删除任何类型。
- Hard cap triggered: no. Topic 未声明 `preferred-contribution-types`。

**v3 review concern 追踪:**
1. Search stability (pairwise 失败): **resolved**。v4 升级为 Gate 1 + 机制发现核心支柱。
2. Multi-seed 实验: **partially resolved**。pilot 完成 3/2/3 seeds，5-seed 全覆盖留到 Experiments。
3. Matched baselines: **deferred to Experiments**（plan acknowledged, not yet executed）。
4. Depth-2/3 tree: **deferred to Experiments**。
5. Break-even 曲线: **deferred to Experiments**。
6. Gate ablation: **deferred to Experiments**。
7. Top-k probe 阈值校准: **partially resolved**。Pushback 有据——经验校准对当前 scope 足够，heldout-family 方案留到 Experiments P1。
8. Weight-tying scheme: **resolved**。v4 body 中明确说明，Experimental P1-7 已列入计划。
9. PSR 作为最强 baseline: **resolved**。novelty check 和 Experiments 中已强化。
10. Macro grammar 覆盖: **partially resolved**。v4 保持有限子语言限定，Pushback 有据——这不是通用 symmetry discovery，honest scoping 优于 overclaim。

v3 review 中 "weight-tying scheme 仍 ignored" 的批评在本轮已解决——v4 line 34 明确说明并在 Experiments 中列为 P1-7。

## Alternative Framing

当前 framing 已接近最优。codex 建议的 **"exchangeability certificate and searchability audit for FEX scalability"** 是更精确的表述：突出"certificate"（认证）和"audit"（审计）两个操作，比"dimension lifting"更能防止审稿人归类为 incremental solver work。建议在论文 title 和 abstract 的第一句使用这个 framing。pairwise 0/3 在这个 framing 下从"方法失败"变成"审计发现"，叙事力量更强。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 对于解落在有限可交换宏 grammar {sum_x2, square_sum_x2, pairwise_xx} 中的 PDE 族，当 FEX RL controller 多 seed 稳定恢复低维 skeleton（Gate 1），且该 skeleton 通过 low-d fit（Gate 2）、coefficient exchangeability（Gate 3）和 top-k PDE probe（Gate 4）四道门认证后，可被安全提升到高维（d 到 100）并仅用参数优化匹配或超过同 compute 预算下的 from-scratch FEX 精度，将搜索成本摊销到维度族上。同时，pairwise 宏解析上可 lift 但 FEX controller 多 seed 搜不到，构成 searchability ≠ liftability 的直接证据——揭示了 FEX 高维成功的双重前提（可恢复 AND 可搜索）。 |
| NULL | 解析/手工宏本身可 lift（pairwise probe rel-L2 = 1.44e-7），但 FEX RL controller 在非平凡 skeleton 上多 seed 搜索不稳定，导致自动 selector 的 recall 不足（pairwise 0/3 可能扩展到其他 non-separable 族）。此结果作为"FEX search quality diagnostic + gate certification on provably-liftable families"有独立发表价值——认证系统在 stable families 上工作，同时精确指出了 FEX controller 的搜索弱点，为后续 controller 改进提供了量化靶点。 |
| NEGATIVE | Stable families（Poisson/radial）多 seed 大面积搜索失败，或 asymmetric control 被误通过，或 top-k probe 频繁推翻 low-d selection。此时 method claim 失败，论文贡献转为负结果——排除"可交换宏结构认证可指导 FEX 高维 lift"这一假说，并记录四门系统在哪些条件下失效。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**（Claude 与 codex 一致）：v4 pilot 9/9 correct classification 提供了强 feasibility signal，到顶会论文的路径清晰。但若干关键条件仍需验证：matched baselines 对照（最关键的剩余风险——parameter-only lift 是否能匹配 from-scratch FEX 和 PSR-style decomposition 在同等 compute 下）、d=100 lift 验证、depth sweep、gate ablation。这些不是纯工程执行——matched baselines 可能显示 lift 无显著优势，depth sweep 可能暴露深度依赖的 liftability 变化。**Impact=High**（Claude）vs **Impact=Medium**（codex）——1 level 分歧。Codex 认为 Impact=Medium：手写 macro grammar 和相对简单的 gate 设计使 contribution 更接近精巧诊断而非重大突破，且 scope 限于 FEX 而非 general SR。我判定 Impact=High 的理由：(a) searchability ≠ liftability 是 genuine mechanism discovery——它改变了我们对 FEX 高维行为的理解方式，不是 incremental improvement；(b) 四门认证系统提供了一个可操作的、有明确边界条件的决策程序，不是模糊的"加速技巧"；(c) 如果成功，它将 FEX 的无 CoD 近似理论与实际 RL 搜索行为之间的理论-实践 gap 缩小了一个具体步骤——这是 FEX 社区的核心关切；(d) NULL outcome 同样有清晰的独立发表价值——这在机制发现范式中是强信号。综合判定：Impact=High, Priority=High (7)。

## Overall

- Priority: High
- Score: 7
- Comments: v4 在四轮迭代（v1→v2→v3→v4）中持续聚焦和收紧，已经达到 idea 阶段的成熟状态。pilot 从 4 case 扩展到 9 case 全正确分类、search stability 从 risk 升级为 mechanism discovery 支柱、叙事从"反证实验"升级为"searchability ≠ liftability"——每一步都在强化核心贡献。v3 review 的 10 条 concern 中：3 条 resolved（search stability, weight-tying, PSR baseline），4 条 partially resolved（multi-seed, probe calibration, macro grammar, depth/baselines/gate ablation deferred），3 条 deferred to Experiments。**Claude 与 codex 在 Impact 上有 1 level 分歧**：codex 判 Impact=Medium（hand-scoped grammar, gate simplicity 限制贡献宽度），Claude 判 Impact=High（searchability ≠ liftability 是 genuine mechanism discovery, 无论正负都有发表价值）。**Likelihood 一致为 Medium**。综合 Priority=High (7)，score 与 v3 持平——v4 的实质改进（pilot 规模翻倍、search stability 升级为机制发现）被 baseline risk（matched baselines 尚未执行、depth sweep 未知）平衡。下一个关键决策点：Proposal 阶段必须执行 matched baselines 对照以验证 lift 的实际增益，否则顶会审稿人会质疑"参数-only lift 是否真的比直接在高维跑 FEX 更 efficient"。若无 matched baseline 优势证据，score 应下调至 Medium (5)。Contribution types 未漂移，topic 未声明 preferred-contribution-types，无 hard cap 触发。文件约 10 KB（不含本 review），未超限。

#if file_size_kB > 10   // 不含本次 review 块
超长警告: 文件约 10 KB（不含本 review）, 未超限
#endif

</review>
