---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-failure-taxonomy/
---

- One-sentence summary: 提出 Counterfactual Oracle-Gap Attribution, 用反事实 oracle 判断一次 FEX/RL 符号回归失败本可被哪种干预救回: 表示容量、reward 代理、结构搜索还是连续参数拟合.
- Hypothesis: FEX 失败通常不是单一标签, 而是几个缺口叠在一起. 固定 PDE、grammar、树形空间和预算后, 比较标准 controller 与容量 oracle、true-error/reward oracle、搜索 oracle 和参数 refit oracle, 可以给每次失败分配非互斥归因: 容量 oracle 失败才说明表示不足; 容量 oracle 成功但标准 reward 低 residual 高 rel.L2 说明 reward 代理欺骗; true-error 选择能救回而标准 controller 不能说明 reward misranking; exhaustive/random-controller 能救回而 policy 不能说明搜索不足; 多起点或 SAGE-Fit 风格 refit 能救回说明参数拟合短板.
- Expected outcome: 强阳性结果是在至少 3 个预注册 stress family 上得到稳定的 oracle-rescue split, 每个接受的主导轴在 affected seeds 中出现率 >=20%, 非标签 trace 特征的 held-out macro-F1 >=0.70, 且至少一个 bottleneck ranking 反直觉, 例如 reward deception 比纯搜索不足更常见. 最便宜的证伪信号是接下来的候选池 true-error oracle 和参数 refit oracle 都无法产生救回差异; 若 macro-F1 <=0.40, 结论改为 "FEX failure is coupled rather than stage-separable."
- Contribution type: diagnostic+empirical-finding
- Risk: MEDIUM
- Estimated effort:
  - Compute: 120-180 GPU-hours for 4 stress families, 3-5 PDEs per family, 20-50 seeds where needed, plus candidate-pool oracle and refit controls
  - Data: available; analytic manufactured PDE cases are generated on the fly, no external dataset or pretrained weights
  - Implementation: 3-4 weeks
- Novelty quick-check: Drissi 2606.01122 gives a per-component diagnostic protocol for neural HJB-PIDE solvers, and Shikhman 2601.11428 stress-tests neural operators, but neither studies RL expression-tree PDE solvers or counterfactual oracle rescue. EGRL-SR and correlated-proxy reward-hacking work explain reward ambiguity, while SAGE-Fit explains Good-Structure-Bad-Score parameter failures; they are single-axis priors, not a FEX/RL-SR attribution protocol. The 2026-06-17 deep-lit pass found no direct oracle-gap failure taxonomy for FEX or RL-guided symbolic PDE solving.
- Strongest objection: The fixed analytic skeleton oracle is only a capacity sanity check. The paper needs candidate-pool true-error selection, search oracle, and refit oracle to produce reproducible held-out rescue splits.
- Why we should do this: A failed FEX run now gives no principled next action: enlarge the grammar, run longer search, change reward, or refit constants. Counterfactual oracle attribution turns that debugging choice into something measurable.
- Pilot:
  - Setup: Instrumented the upstream FEX Poisson runner on local CUDA, added manufactured `conservation_sine` and `helmholtz_sine` gates, and added a fixed-action sine capacity oracle while keeping the same controller, tree, operator set, and residual+boundary reward.
  - Metric: Signal is positive if two non-Poisson PDEs reproduce residual-vs-true-error deception under the standard controller, while the fixed FEX skeleton oracle solves them; this rules out pure representation capacity for the early gate.
  - Result: Poisson base sweep succeeded in 10/10 seeds but had 354/600 deceptive search epochs. Poisson stress runs split into 3/3 finetune-shortfall and 3/3 search-underfit failures. Across `conservation_sine` and `helmholtz_sine`, the standard controller failed 6/6 runs, with 5/6 terminal reward-deception and 56/120 deceptive epochs; the analytic sine skeleton oracle solved both non-Poisson cases at rel.L2 5.01e-8 with residual about 2e-13.
  - Signal: POSITIVE, but bounded: current evidence supports capacity-vs-standard-controller separation and reward-deception transfer, not yet full search-vs-reward-vs-parameter attribution.

- Claims and Claims matrix: Strong claim: on a preregistered FEX/RL-SR stress suite, counterfactual oracle gaps give stable, non-exclusive failure attributions across held-out PDE families, with macro-F1 >=0.70 from traces that exclude the oracle labels themselves. Moderate claim: reward proxy deception transfers beyond Poisson and survives a capacity oracle, while search and parameter axes need narrower evidence. Null claim: if oracle labels collapse or macro-F1 <=0.40, the empirical finding is that stage-wise fixes are unreliable because FEX failures are coupled. Negative claim: if candidate-pool true-error and refit oracles do not rescue any failures, the work should be downgraded to a FEX debugging note.
- Narrative: The paper asks one practical question: which counterfactual intervention would have saved this failed symbolic-regression run? The contribution is predictive failure attribution, not a benchmark release or a catalogue of bad runs.
- Experiments: Pre-register four stress families: oscillatory/frequency-separated PDEs, operator-set omission, tree-depth or representation mismatch, and collocation/boundary perturbation. For each family, run standard FEX against capacity oracle, candidate-pool true-error or all-point reward selection, exhaustive small-tree or random/uniform controller where feasible, and multi-start/SAGE-Fit-style refit. Use a non-exclusive decision rule: report every oracle that rescues a failed run, mark multi-oracle rescue as coupled, and attach bootstrap confidence intervals plus a small power analysis for each reported axis.
- Assets status: Analytic PDE cases, local FEX code, Poisson/stress/non-Poisson runs, fixed-skeleton oracle results, and handoff notes are documented in `workspace/fex-failure-taxonomy/data/MANIFEST.md`; no external asset download is needed.

<review date="2026-06-17">

## Novelty

- Score: 7/10
- Closest prior work: A Per-Component Diagnostic Protocol for Neural HJB-PIDE Solvers (Drissi, 2606.01122, 2026); Diagnosing Failure Modes of Neural Operators Across Diverse PDE Families (Shikhman, 2601.11428, 2026); Architecture-Induced Recoverability Bias in Differentiable SR (Gupta & LaGrow, 2604.23256, 2026); EGRL-SR (Sun et al., 2601.14693, 2026); SAGE-Fit (Wang et al., 2605.23272, 2026)
- Key differentiator: 经 arxiv/S2 多源交叉验证，未发现任何直接针对 FEX/RL 表达式树搜索的 oracle-gap 失败归因协议。Drissi 2606.01122 的 per-component HJB-PIDE 诊断在方法学上最接近（分项审计→定位故障点），但针对的是非 RL 神经 HJB 求解器而非 RL 表达式树搜索。Shikhman 2601.11428 对 FNO/DeepONet/CNO 做压力测试，但对象是 neural operator 且框架是 empirical degradation 而非 counterfactual oracle rescue。Gupta 2604.23256 拆开 differentiable SR 的 representability vs recoverability，验证了本 idea 的 representation axis 但仅为单轴研究。EGRL-SR 和 SAGE-Fit 各自覆盖 reward ambiguity 和 parameter fitting 单轴，均非多轴归因协议。RLHF reward hacking 文献（2604.12086, 2403.03185）提供了 proxy-true correlation 的形式化语言但未应用于 SR。多源搜索确认 FEX/RL-SR oracle-gap failure taxonomy 为交叉领域空白。创新度集中在：(a) 反事实 oracle rescue 作为 RL-SR 归因工具；(b) 跨 held-out PDE family 的稳定归因验证；(c) 非互斥多因标签框架。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v4 的非互斥归因框架显著改进了 v3 的因果重叠问题。Oracle 决策链已明确：capacity oracle 失败→表示不足；true-error/all-point reward oracle 能救回→reward 代理失真；exhaustive/random-controller 能救回而 policy 不能→搜索不足；multi-start/SAGE-Fit refit 能救回→参数拟合短板。剩余 gap: (a) 各 oracle 之间的隔离并非完全干净——例如 exhaustive search oracle 隐含改变了 reward 评估方式；(b) "rescue" 的操作化定义仍需精化（residual 阈值还是 rel.L2 阈值？）；(c) >=20% affected seeds 的阈值缺乏敏感性分析。 |
| Missing evidence signals | 7/10 | v4 相较 v3 有实质证据升级：增加 helmholtz_sine gate，形成 2 个非 Poisson PDE 的 stress 证据；analytic sine skeleton oracle 在两个非 Poisson 案例上均成功 rescue（rel.L2 5.01e-8, residual ~2e-13），是首个真实 oracle rescue 信号；6/6 standard controller 全败 + 5/6 terminal reward deception 提供了跨 PDE 的 reward-deception transfer 证据。仍缺失：(a) true-error/搜索/refit oracle 的 rescue 结果（capacity oracle 只是 sanity check）；(b) held-out stress family 的归因稳定性；(c) bootstrap CI 和 power analysis（v4 承诺了但未执行）；(d) 预注册 stress PDE suite 的锁定。 |
| Narrative | 8/10 | "Counterfactual Oracle-Gap Attribution" 直接采纳了 v3 codex 建议，叙事升级完整执行。"Which counterfactual intervention would have saved this failed symbolic-regression run?" 是出色的 paper-level question，将贡献锚定在预测性归因而非被动分类。v4 明确区分"不是 benchmark release 或失败目录"是正确的叙事选择，但需注意：对 ICML/NeurIPS/ICLR 而言，stress suite 和 decision rule 的可复用性仍是审稿人期望的。叙事弱点：若无法配套 public stress suite，仍有被读作 FEX 内部调试笔记的风险。 |
| Venue contribution | 7/10 | topic 未声明 target-venue。按 FEX 系列历史 venue (JMLR/JCP) 及顶会 (ICML/NeurIPS/ICLR) 标准评估。纯 diagnostic+empirical-finding 在顶会处于 borderline。通往 top-venue 的充分条件：(a) 至少一个反直觉 bottleneck ranking（如 reward deception > search insufficiency 是主导失效轴）；(b) 可复用且预注册的 stress-test PDE suite 作为 artifact；(c) 跨 >=3 个 stress family 的 holdout 归因稳定性，macro-F1 >=0.70。若三者全满足，venue contribution 可至 8/10。 |
| Testability | 7/10 | v4 包含明确的 cheapest falsifying signal：若 candidate-pool true-error oracle 和 parameter refit oracle 均无法产生 rescue 差异→降级为 FEX debugging note。POSITIVE (macro-F1>=0.70) 和 NULL (macro-F1<=0.40) 均有量化阈值。改进建议：falsifier 应宽化为"no non-capacity oracle produces differential rescue"，而不仅是 true-error+refit——因为 search oracle 单独 rescue 也是 valid 信号。 |
| Outcome realism | 7/10 | 120-180 GPU-hours 在已有 instrumentation 的前提下可行，但 oracle 对照（exhaustive search、SAGE-Fit refit、true-error 全点评分）可能显著推高成本。exhaustive search 在小树上可行但在实际规模下不可行，需要在 stress PDE 设计时就限定搜索空间。3-4 周实现时间合理。真正的风险不在 compute 而在：non-label trace features 能否在不泄露 oracle 信息的前提下达到 macro-F1 >=0.70；stress PDE 设计是否过度耦合于特定 failure axis。 |
| Contribution type compliance | n.a. | idea types (diagnostic+empirical-finding) ⊆ preferred-contribution-types: n.a. — topic 未声明 `preferred-contribution-types`，跳过检查。不计入 Overall Quality 平均。 |
| Overall Quality | 7/10 | v4 相较 v3 有实质升级：framing 从 Oracle-Gap Decomposition 进化为 Counterfactual Oracle-Gap Attribution（采纳 codex 建议）；pilot 证据从 1 个非 Poisson PDE 扩展到 2 个 + 首次真实 oracle rescue 信号（analytic sine skeleton 在 conservation_sine 和 helmholtz_sine 上均成功，rel.L2 5.01e-8）；非互斥归因框架消除了 v3 的最大逻辑 gap。当前状态是 a credible proposal with genuine positive pilot evidence and a clear next kill test——距 top-venue result 还差决定性实验。 |

## Contribution Drift (n=4)

- v3 contribution types: {diagnostic, empirical-finding}
- v4 contribution types: {diagnostic, empirical-finding}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 preferred-contribution-types; 无 contribution 类型扩张或降级)

## Prior Review Concern Resolution (v3 → v4)

| v3 Concern | Status | Notes |
|-----------|--------|-------|
| Search vs reward 因果重叠 | Partially resolved | v4 的非互斥多因标签框架是正确方向，允许同时归因于多个 oracle；但 oracle 隔离仍需在实验层面验证 |
| 缺乏 cross-PDE transfer 证据 | Partially resolved | helmholtz_sine gate + fixed-skeleton oracle rescue (rel.L2 5.01e-8) 是 v3→v4 的最大证据升级；但仍仅 2 个 manufactured PDE，非 held-out stress suite |
| 叙事缺乏 top-venue hook | **Resolved** | Counterfactual Oracle-Gap Attribution 完全执行了 v3 的 Alternative Framing 建议 |
| Venue contribution 不足 | Partially resolved | stress families 和 preregistration 在 v4 中 explicit；但"not a benchmark release" 立场对顶会而言是 weak pushback——可复用的 evaluation protocol/decision rule 仍应作为 artifact 贡献 |
| 需要 cheapest falsifier | Partially resolved | true-error+refit oracle gate 是正确的廉价证伪路径；建议宽化为"no non-capacity oracle produces differential rescue"以覆盖 search oracle |
| Stress modes 可能耦合 | **Resolved** | v4 的 "non-exclusive decision rule: report every oracle that rescues" 和 "mark multi-oracle rescue as coupled" 直接处理了此问题 |
| 缺失 CI/power analysis | Partially resolved | v4 承诺了 bootstrap CI + small power analysis，但仍是 planned 而非已执行的证据 |

Refiner pushback 评估：唯一 questionable pushback 是将贡献刻意与 benchmark/stress-suite 解耦——对顶会审稿人而言，可复用的 evaluation protocol 是 diagnostic paper 的核心贡献载体。pushback 本身的论证（"contribution is predictive failure attribution, not a benchmark release"）在叙事层面成立，但 artifacts 维度仍应保留。

## Alternative Framing

当前 "Counterfactual Oracle-Gap Attribution" 已非常锐利。codex 本次建议 "Intervention-ranking oracle audit" 将重点从 "what would have saved it" 转向 "rank interventions by efficacy"——这是一个可考虑的方向但并非明显更优，因为 counterfactual 因果语言比 intervention-ranking 工程语言在 ML 顶会中更具识别度。建议保持当前 framing，仅在实验设计层面显式加入 intervention ranking（按 rescue rate 排序各 oracle 的有效性）。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在预注册的 FEX/RL-SR stress-test PDE suite 上，反事实 oracle gap 可在 >=3 个 stress family 上产生稳定、非互斥的失败归因，每轴在 >=20% affected seeds 中复现，基于非标签诊断特征的 held-out macro-F1 >= 0.70，且至少一个 bottleneck ranking 反直觉（如 reward deception 而非 search insufficiency 是主导失效）。Moderate positive：若仅 capacity+reward 两轴稳定分离，parameter 和 search 轴需窄化 claim。 |
| NULL | 若 oracle-gap labels 无法分离（macro-F1 <= 0.40），当前诊断协议下 FEX 失败是耦合的，分治 fix 策略不可靠——这是一个有意义的 negative result。 |
| NEGATIVE | 若 candidate-pool true-error、search 和 refit oracles 均不产生 beyond-capacity 的 differential rescue 信号，工作应降级为 FEX 调试笔记（reward/true-error decoupling in manufactured PDEs），不作为通用 RL-SR 诊断论文投稿。 |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**: pilot 证据持续为正且强度递增——v4 的 fixed-skeleton oracle rescue (rel.L2 5.01e-8) 是首个真实 counterfactual rescue 信号，证实 capacity-vs-controller 分离可行；reward deception 在 2 个非 Poisson PDE 上稳定 transfer（5/6 terminal deception）。实验路径清晰，compute 预算可行。但 top-venue 成稿依赖多个未验证条件：(a) true-error/search/refit oracles 需产生干净 differential rescue 信号；(b) non-label trace features 需在不泄露 oracle 信息的前提下达到 macro-F1 >=0.70；(c) 归因需在 held-out stress families 上稳定；(d) 至少一个反直觉 bottleneck ranking。**Impact=High**: Claude 与 codex 一致判定为 High。若成功，本工作将为 RL-SR/FEX 社区提供首个可操作的 counterfactual 归因协议——不再是对失败的模糊诊断，而是直接回答"哪个 oracle 能救回这个失败？"并据此指导方法改进方向（扩大 grammar？延长搜索？换 reward？refit 参数？）。counterfactual oracle rescue 方法论可作为 RL-for-scientific-discovery 管线诊断的范本。Impact 不到 Exceptional 因为 scope 限定在 RL-SR/FEX niche，不构成 broad ML paradigm shift。Claude 与 codex 在 Likelihood 和 Impact 上均无分歧 (>=1 level)。

## Overall

- Priority: High
- Score: 7
- Comments: v4 是 v3→v4 迭代中的实质性升级：Counterfactual Oracle-Gap Attribution 命名升级、第二个非 Poisson PDE gate (helmholtz_sine)、首个真实 oracle rescue 信号 (rel.L2 5.01e-8)、非互斥归因框架——每一项都是 reviewer 可见的改进。当前状态是 well-formed proposal with genuinely positive pilot evidence。下一决定性实验不应是全量 stress sweep，而是 **2 个预注册非 Poisson PDE + 4 类 oracle (true-error, search, refit, capacity) 的真实对照**：若仅 capacity oracle 产生 rescue 而其余均无 differential signal，则降级或 pivot；若 >=2 类 oracle 产生可分离 rescue，则进入 full stress sweep。这是成本最低但判别力最强的下一步。

</review>
