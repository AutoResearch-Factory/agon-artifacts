---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-failure-taxonomy/
---

- One-sentence summary: 提出 Oracle-Gap Decomposition, 用一组反事实 oracle 将 FEX/RL 符号回归失败归因到表示容量、结构搜索、连续参数拟合和 reward 代理失真四个可操作缺口.
- Hypothesis: FEX 失败不是单一的 "RL 不稳定". 若固定 PDE、grammar、树形空间和搜索预算, 比较标准 FEX 与四类 oracle 上界, 就能判断失败来自哪个缺口: grammar/tree oracle 仍失败是表示不足, true-error 或 all-point reward oracle 能救回是 reward 代理失真, exhaustive/random-controller 对照能救回而 policy 不能是搜索不足, 多起点或 SAGE-Fit 风格 refit 能救回是参数拟合短板.
- Expected outcome: 成功时, 至少 3 个 stress family 在 held-out PDE 上复现可分离 oracle gap, 每个主导轴在 affected seeds 中出现率 >=20%, 非标签诊断特征的 held-out macro-F1 >=0.70; 最廉价证伪信号是前两个非 Poisson PDE 都不复现 residual-true-error gap 或 oracle rescue split, 此时应把 claim 降为 Poisson debugging note. 若 macro-F1 <=0.40, 结论改为 "FEX failure is coupled rather than stage-separable."
- Contribution type: diagnostic+empirical-finding
- Risk: MEDIUM
- Estimated effort:
  - Compute: 100-150 GPU-hours for 4 stress families, 3-5 PDEs per family, 20-50 seeds where needed, plus oracle/refit controls
  - Data: available; analytic manufactured PDE cases are generated on the fly, no external dataset or pretrained weights
  - Implementation: 3-4 weeks
- Novelty quick-check: EGRL-SR diagnoses error-based ambiguity in general SR and proposes GCRL/HER/APSR as a fix, but it does not decompose PDE residual search into representation/search/parameter/reward oracle gaps. SAGE-Fit targets the inner-loop Good-Structure-Bad-Score problem and is a natural parameter oracle, not a full FEX failure attribution protocol. Neural-operator failure diagnostics stress-test FNO/DeepONet/CNO rather than RL expression-tree solvers. A 2026-06-17 arXiv sanity check found no direct FEX/RL-SR oracle-gap failure taxonomy.
- Strongest objection: The study may still look like a lab-specific diagnostic unless oracle gaps predict held-out PDE failures and expose at least one non-obvious bottleneck ranking, such as reward deception dominating search insufficiency on nominally easy tasks.
- Why we should do this: A failed FEX run currently gives no guidance about whether to enlarge the grammar, run longer search, refit constants, or change the reward. Oracle-gap attribution makes that debugging choice testable and points each failure to a concrete next fix.
- Pilot:
  - Setup: Instrumented the upstream FEX Poisson runner on local CUDA, logged residual, rel.L2, controller entropy and structures; added a manufactured `conservation_sine` first-order PDE gate while keeping the same controller, tree, grammar and reward.
  - Metric: Signal is positive if at least two terminal failure mechanisms appear and a non-Poisson early gate reproduces either reward-deception or search-underfit.
  - Result: Poisson base sweep succeeded in 10/10 seeds but had 354/600 deceptive search epochs (residual <=1e-3 while rel.L2 >0.1); Poisson stress runs produced 3/3 finetune-shortfall and 3/3 search-underfit terminal failures. The conservation-sine gate had 0/3 successes, with 2/3 terminal reward-deception runs, 1/3 search-underfit run, and 25/60 deceptive epochs.
  - Signal: POSITIVE, but bounded to two manufactured PDE families until the oracle controls and held-out stress suite are run.

- Claims and Claims matrix: Strong positive claim: FEX/RL-SR failures can be attributed by oracle gaps that remain stable across held-out PDE families, with macro-F1 >=0.70 from non-label traces and each accepted axis recurring in >=20% of affected seeds. Moderate positive claim: residual reward deception and search underfit transfer beyond Poisson, but parameter and representation gaps need narrower claims. Null claim: if oracle-gap labels do not separate (macro-F1 <=0.40), the correct empirical finding is that stage-wise fixes are unreliable because failures are coupled. Negative claim: if non-Poisson gates fail to reproduce the gaps, the project should not be pitched as a general RL-SR diagnostic paper.
- Narrative: The paper asks a concrete question: when RL-guided symbolic regression fails, which oracle would have saved it? That keeps the contribution on predictive failure attribution, not a catalogue of observed bad runs.
- Experiments: Run four stress families: oscillatory/frequency-separated PDEs, operator-set omission, tree-depth or representation mismatch, and collocation/boundary perturbation. For each, compare standard FEX to oracle operator/tree capacity, exhaustive small-tree or large-budget search where feasible, true-error or all-point reward selection, random/uniform controller, and multi-start/SAGE-Fit-style refit. Report both attribution accuracy on held-out PDEs and the bottleneck ranking under equal compute.
- Assets status: Analytic PDE cases, local FEX code, Poisson pilot logs, conservation-sine early-gate logs, and handoff notes are documented in `workspace/fex-failure-taxonomy/data/MANIFEST.md`; no external asset download is needed.

<review date="2026-06-17">

## Novelty

- Score: 7/10
- Closest prior work: Per-Component Diagnostic Protocol for Neural HJB-PIDE Solvers (Drissi, 2606.01122, 2026); Diagnosing Failure Modes of Neural Operators Across Diverse PDE Families (Shikhman, 2601.11428, 2026); EGRL-SR (Sun et al., 2601.14693, 2026); SAGE-Fit (Wang et al., 2605.23272, 2026); Architecture-Induced Recoverability Bias in Differentiable SR (Gupta & LaGrow, 2604.23256, 2026)
- Key differentiator: 经 arXiv/S2 多源复核，未发现任何直接针对 FEX/RL-SR 的 oracle-gap 分解或失败 taxonomy 论文。2606.01122 的 per-component HJB-PIDE 诊断协议在方法论上最接近（分项审计→定位 bug），但针对的是非 RL 神经 HJB 求解器而非 RL 表达式树搜索。2601.11428 对 FNO/DeepONet/CNO 做压力测试，方法论可借鉴但对象 (neural operator vs RL expression-tree) 与归因框架 (empirical degradation vs counterfactual oracle rescue) 均不同。2604.23256 拆开 differentiable SR 的 representability vs recoverability，直接验证本 idea 的 representation insufficiency 轴，但它是单一轴实证研究而非多轴 oracle 归因协议。方法学层面的诊断范式已知，新颖度集中在：(a) 反事实 oracle 作为 FEX/RL-SR 归因工具；(b) 跨 held-out PDE family 的稳定归因验证。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v3 的 oracle rescue 逻辑比 v2 显著清晰：grammar/tree oracle 仍失败→表示不足，true-error oracle 能救回→reward 代理失真，exhaustive/random 对照能救回而 policy 不能→搜索不足，多起点/SAGE-Fit refit 能救回→参数拟合短板。但标签间仍存在因果重叠：reward 欺骗可诱发 search 失败，参数失败可伪装为 reward 差。建议引入非互斥多因标签或形式化决策树 (若 oracle-A rescue 成功则归因到 A 的对应 gap; 若多 oracle 同时 rescue，标记为耦合失效)。 |
| Missing evidence signals | 6/10 | v3 新增 conservation-sine early gate (n=3 seeds, 2/3 reward-deception, 1/3 search-underfit)，是跨 PDE transfer 的有益初步证据。但缺失：(a) 任何 oracle 对照的运行结果——目前只知道有 failure，不知道 oracle 能否 rescue；(b) 置信区间和 statistical power analysis；(c) stress PDE 是否预注册而非事后调优的证明；(d) oracle 实现自身的正确性验证。 |
| Narrative | 8/10 | "when RL-guided symbolic regression fails, which oracle would have saved it?" 这一提问是出色的叙事 hook，直接对标预测性归因而非事后分类。从 v2 的 "诊断框架" 到 v3 的 "Oracle-Gap Decomposition"，叙事转型完整执行。弱势：如不配套 public stress suite 或通用 RL-SR 协议，仍有被读作 FEX 内部调试笔记的风险。codex 建议的 "Counterfactual Oracle-Gap Attribution" 进一步强化因果归因味道，值得采纳。 |
| Venue contribution | 6/10 | topic 未声明 target-venue。按 FEX 系列历史 venue (JMLR/JCP) 及科学 ML 顶会标准，纯 diagnostic+empirical-finding 处于 borderline。通往 top-venue 的条件：(a) 至少一个反直觉 bottleneck ranking（如 reward deception > search insufficiency 是主导失效）；(b) 可复用的 stress-test PDE suite 作为 benchmark artifact；(c) 跨至少 3 个 stress family 的 holdout 归因稳定性。若三者全满足，venue contribution 可升至 8/10。 |
| Testability | 8/10 | v3 包含明确的 cheapest falsifying signal：前两个非 Poisson PDE 都不复现 residual-true-error gap 或 oracle rescue split，即降级为 Poisson debugging note。POSITIVE (macro-F1>=0.70) 和 NULL (macro-F1<=0.40) 均有量化阈值。codex 建议将早期 PDE 预注册（在查看结果前固定），进一步增强证伪可信度。较 v2 的改进：非 Poisson early gate 已部分执行 (conservation-sine POSITIVE)，降低了证伪成本。 |
| Outcome realism | 7/10 | 100-150 GPU-hours 预算对于 4 stress families × 3-5 PDEs 规模看起来可行，但 oracle 对照（exhaustive search、SAGE-Fit refit、true-error reward）可能显著推高成本，尤其是 exhaustive search 在小树上可行但在实际规模下不可行。3-4 周实现时间在代码已有 instrumentation 的前提下合理。真正的风险：(a) oracle gap 在真实对照中不干净分离；(b) stress PDE 设计与各 failure axis 耦合，需要多轮迭代。 |
| Contribution type compliance | n.a. | idea types (diagnostic+empirical-finding) ⊆ preferred-contribution-types: n.a. — topic 未声明 `preferred-contribution-types`，跳过检查。不计入 Overall Quality 平均。 |
| Overall Quality | 7/10 | v3 较 v2 有实质性提升：Oracle-Gap Decomposition framing 完全采纳了 v2 的 Alternative Framing 建议，conservation-sine early gate 提供了非 Poisson 跨 PDE transfer 的初步证据，cheapest falsifier 已内嵌。主要剩余风险：(a) oracle controls 尚未跑过，真实归因分离能力未知；(b) stress PDE 设计可能无法干净触发单轴 failure。方向正确、可执行、pilot 信号持续为正，适合继续推进到实验阶段，但需设严格 early gate。 |

## Contribution Drift (n=3)

- v2 contribution types: {diagnostic, empirical-finding}
- v3 contribution types: {diagnostic, empirical-finding}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 preferred-contribution-types; 无 contribution 类型扩张或降级)

## Prior Review Concern Resolution (v2 → v3)

| v2 Concern | Status | Notes |
|-----------|--------|-------|
| Search vs reward 因果重叠 | Partially resolved | v3 的 true-error/all-point reward oracle 设计提供了分离机制，但标签仍需形式化决策树或允许多因归因 |
| 缺乏 cross-PDE transfer 证据 | Partially resolved | conservation-sine gate (0/3 success, 2/3 reward-deception) 是真实改进，但 n=3 且无 oracle 对照仍然单薄 |
| 叙事缺乏 top-venue hook | **Resolved** | Oracle-Gap Decomposition 直接采纳了 v2 的 Alternative Framing，叙事完成升级 |
| Venue contribution 不足 | Partially resolved | bottleneck ranking 被提及，但 public stress suite 和 benchmark artifact 仍未 explicit |
| 需要 cheapest falsifier | **Resolved** | 前两个非 Poisson PDE 失败即降级——这是正确且廉价的 early gate |
| Stress modes 可能耦合 | Partially resolved | oracle 设计部分缓解，但耦合 failure 的可能性应在实验设计中预做计划而非视为 nuisance |
| Alternative framing 未完全采纳 | **Resolved** | v3 明确采用 Oracle-Gap Decomposition 作为核心 framing |

Refiner 无 unsupported pushback。未完全采纳的建议（public benchmark release、confidence intervals 量化、预注册 stress PDE）均有合理优先级考量（先跑 oracle 对照 gate 再决定投入规模）。

## Alternative Framing

当前 "Oracle-Gap Decomposition" 已较锐利。codex 建议的微调：**"Counterfactual Oracle-Gap Attribution for RL-Guided Symbolic Regression"**——将 "decomposition" 升级为 "counterfactual attribution"，更贴近因果推断语言，强调 "which counterfactual intervention would have rescued this run?" 而非被动分解。配合 public stress-test PDE suite，这一 framing 可将贡献从 FEX-specific debugging 升级为 reusable evaluation methodology。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在预注册的 FEX/RL-SR stress-test PDE suite 上，失效可按反事实 oracle gap 归因至至少 3 个 stress family，每轴在 >=20% affected seeds 中复现，且基于非标签诊断特征的 holdout macro-F1 >= 0.70。Moderate positive: residual reward deception 和 search underfit 跨 PDE transfer 已确认，但 parameter 和 representation gap 需更窄 claim。 |
| NULL | 若 oracle-gap label 无法分离（macro-F1 <= 0.40），正确的实证结论是 FEX 失效在当前诊断协议下是耦合的，分治 fix 策略不可靠。 |
| NEGATIVE | 若前两个预注册的非 Poisson PDE 都无法复现 residual-true-error gap 或 oracle rescue split，工作应降级为 Poisson/FEX debugging note，不作为通用 RL-SR 诊断结果发布。 |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**: v3 pilot 持续为正（Poisson deceptive reward 354/600 epochs, conservation-sine gate 复现 reward-deception, 两类 terminal failure 各 3/3 seeds），实验路径清晰，compute 预算可行。但 top-venue 成稿依赖多个条件同时成立：(a) oracle 对照能干净分离 failure axes；(b) holdout classifier 跨 PDE family 稳定；(c) stress PDE 设计不过度耦合；(d) 至少一个反直觉 bottleneck ranking。**Impact=High**: Claude 与 codex 最终达成一致。v3 的 Oracle-Gap Decomposition 是 transferable diagnostic methodology——成功后将直接指导 FEX/RL-SR 方法学改进路线（扩大 grammar？延长搜索？换 reward？refit 参数？），成为 RL-SR 调试的标准范式。配合至少一个非平凡发现（如 reward deception 而非 search insufficiency 是主导失效），对 RL-SR 研究线有明确牵引作用。

## Overall

- Priority: High
- Score: 7
- Comments: v3 在 v2 基础上完成了核心叙事升级（Oracle-Gap Decomposition），并通过 conservation-sine gate 提供了跨 PDE transfer 的首个 POSITIVE 信号。与 codex 在 Likelihood 维度一致（Medium），在 Impact 维度经讨论后收敛（均认为可达 High）。关键建议：(1) 下一 decisisive experiment 不应是全量 stress sweep，而是 2 个预注册非 Poisson PDE + 真实 oracle rescue 对照；若 gate 失败则降级或停止；(2) 若 gate 通过，将 stress suite + attribution decision rule 打包为贡献主体而非仅报告观察结果；(3) 考虑 codex 建议的 "Counterfactual Oracle-Gap Attribution" 微调叙事。

#if file_size_kB > 10
超长警告: 文件 ~5 KB (不含本 review 块 ~4 KB), 未超上限
#endif

</review>

<deep-lit date="2026-06-17" scope="idea" rounds="1">

## Deep Lit 结果：FEX/RL-SR Oracle-Gap Failure Taxonomy

### 搜索概览

- 搜索 axes: method, application, data, evaluator, failure-mode, adversarial-framing（6 axes, 14 queries）
- 首轮产出: 5 篇跨域相邻论文精读 + wiki 写入
- B7 反向扩展: 20 次调用（references + cited + author + title-term × 5 papers）
- 结论: **FEX/RL-SR oracle-gap failure taxonomy 交叉领域确认为空白** — B7 候选全部落入不相交的相邻域（LLM RLHF reward hacking、PINN failure、制造工程 RL）

### 已读论文与发现

| arXiv ID | Title | Year | 域 | 对本 idea 的价值 |
|----------|-------|------|-----|-----------------|
| 2605.30910 | PINNs Failure Modes are Overfitting | 2026.05 | PINN 失败诊断 | **方法论灵感**: train/test residual divergence 作为 failure mode 判别标准；subtractive experiment 方法；'过拟合 vs 优化困难'的失败分类维度和 residual 可视化 |
| 2605.25063 | Proxy-FEA Diagnostic Framework for RL Reward Diagnosis | 2026.05 | 制造工程 RL | **方法论灵感**: bilevel evaluator（cheap proxy + sparse high-fidelity verifier）设计模式；proxy-verifier pairwise ranking agreement 作为 reward 可靠性诊断 |
| 2604.12086 | Robust Optimization for Mitigating Reward Hacking with Correlated Proxies | 2026.04 | RL reward hacking | **概念对接**: r-correlated proxy 形式化 → FEX residual vs true-error 的 proxy correlation 可类比定义；max-min robust formulation 可启发 FEX worst-case failure bound |
| 2403.03185 | Correlated Proxies: Definition and Mitigation for Reward Hacking | 2024.03 | RL reward hacking | **核心 prior**: proxy-true reward correlation 的正式定义+ORPO；若 FEX failure taxonomy 用到 reference-policy 上的 proxy-true 相关性则必须引用 |
| 2505.08783 | CodePDE: LLM-driven PDE Solver Generation | 2025.05 | LLM + PDE | **边际相关**: 系统性 failure mode 分析的工程范式（self-debugging loop、错误分类），但与 RL 表达式树搜索方法学正交 |

### 接近但未深读的 B7 候选

以下论文由 B7 反向扩展发现，与 "failure taxonomy / reward hacking diagnosis" 精神相近，但全部在 **LLM RLHF** 域而非 SR/PDE solver：

- **2606.03238** — "When RLHF Fails: A Mechanistic Taxonomy of Reward Hacking, Collapse, and Evaluator Gaming" (Abahana, 2026.06). RLHF 失败分类学，方法学最接近本 idea 的 taxonomy 精神，但对象是 LLM 而非 FEX。
- **2606.09711** — PRIME: Proxy Reward Internalization and Mechanistic Exploitation (Beigi et al., 2026.06). Early-warning signal for reward hacking。proxy-internalization 概念可类比 FEX 的 deceptive reward detection。
- **2605.20744** — Hack-Verifiable Environments (Roth et al., 2026.05). Reward hacking benchmark 设计范式。
- **2602.01750** — ARA: Adversarial Reward Auditing (Beigi et al., 2026.02). Reward hacking 检测而非归因。
- **2505.10949** — "FP64 is All You Need" (Xu et al., 2025.05). PINN 精度与失败关系。
- **2502.00803** — ProPINN: Demystifying Propagation Failures in PINNs (Wu et al., 2025.02). PINN 传播失败诊断。

### 撞车风险评估（2026-06-17）

| FEX Failure Taxonomy Claim | 状态 | 最接近 prior | 说明 |
|---------------------------|------|-------------|------|
| Oracle-gap decomposition (4 gaps) | ✅ 无直接竞品 | 无 | 反向扩展确认：反事实 oracle 归因在 SR 域完全空白 |
| Reward proxy distortion diagnosis | 🟡 邻域有概念对应 | 2403.03185, 2604.12086 | RLHF reward hacking 文献提供了 proxy-true correlation 形式化工具，但未应用于 SR |
| Search insufficiency attribution | ✅ 无直接竞品 | 无 | exhaustive/random-controller oracle 对照在 SR 域未见 |
| Representation insufficiency | 🟡 邻域有概念对应 | 2604.23256 (Architecture-Induced Recoverability Bias) | 仅覆盖 differentiable SR 的可表示性 vs 可恢复性，非 oracle-gap 框架 |
| Parameter fitting shortfall | ✅ 无直接竞品 | SAGE-Fit (2605.23272) | GSBS 问题是参数 oracle 的 natural prior，但非归因协议 |
| Cross-PDE generalization of failure labels | ✅ 无直接竞品 | 无 | 跨 held-out PDE family 的归因稳定性验证未见 |
| Overall: FEX failure taxonomy as diagnostic protocol | ✅ **Novel 确认** | Per-Component HJB-PIDE Diagnostic (2606.01122) 方法学最接近但域不同 | 最大差异化：(a) 反事实 oracle 而非逐组件审计；(b) RL 表达式树而非神经 HJB 求解器 |

### 对 idea 的建议

1. **Novelty 安全边际充足**。即使是最接近的 2606.01122（Per-Component HJB-PIDE Diagnostic），其方法论（逐组件替换+误差对比）和对象（神经 HJB 求解器）与本 idea（反事实 oracle rescue + RL 表达式树搜索）都不同。应引用以示区分。
2. **RLHF reward hacking 文献是方法论富矿**。2403.03185 的 correlated proxy 定义、2604.12086 的 max-min robust formulation、2606.09711 的 early-warning signal design，都可以作为 FEX reward proxy distortion axis 的理论/方法学参照系——建议在 related work 中专门设一段 "Lessons from RLHF Reward Hacking"。
3. **PINN failure mode 论文提供诊断方法论模板**。2605.30910 的 subtractive experiment（减少配点→暴露过拟合）、train/test divergence 监控、residual visualization，可直接移植到 FEX failure 诊断的 experiment design。
4. **2605.25063 的 bilevel proxy-FEA evaluator 是 oracle-gap 框架的原型**。cheap proxy + sparse high-fidelity verifier 的 pairwise ranking agreement 模式，与本 idea 中 "residual reward vs true-error oracle" 的对照逻辑高度一致——可作为方法论 prior 引用。
5. **B7 饱和确认无需第二轮**。6 轴搜索 + 5 篇精读 + 20 次反向扩展的候选池中，没有任何论文直接覆盖 FEX/RL-SR oracle-gap failure taxonomy。

</deep-lit>
