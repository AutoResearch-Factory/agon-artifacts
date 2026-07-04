---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-failure-taxonomy/
---

- One-sentence summary: 提出 Counterfactual Oracle-Gap Attribution, 用可控 oracle 判断一次 FEX/RL 符号回归失败本可由哪种干预救回: 表示容量、连续参数拟合、reward 代理选择还是结构搜索.
- Hypothesis: FEX 失败往往不是单轴标签. 固定 PDE、grammar、树形空间和搜索预算后, 反事实救回才是可操作归因: capacity oracle 成功说明目标可表示; 固定 action 的 multi-start refit 能救回说明连续参数拟合是短板; 同一候选池里 true-error 选择优于 residual 选择说明 reward 代理错排; 扩大随机/穷举小树搜索能救回而标准 controller 不能说明搜索不足. v5 pilot 已确认前两点: 非 Poisson 失败不是表示不足, 而且 6/6 标准失败可由固定 action refit 救回.
- Expected outcome: 强阳性结果是在至少 3 个预注册 stress family 上得到稳定的非互斥 oracle-rescue split: 每个接受的主导轴在 affected seeds 中出现率 >=20% 且在 10/20/30% 阈值下排序不翻转, 非标签 trace 特征的 held-out macro-F1 >=0.70, 并出现至少一个反直觉 bottleneck ranking. 当前最便宜的证伪信号是后续 candidate-pool true-error oracle 和扩大搜索 oracle 都不产生 refit 以外的差异; 若 macro-F1 <=0.40, 结论改为 "FEX failure is coupled rather than stage-separable."
- Contribution type: diagnostic+empirical-finding
- Risk: MEDIUM
- Estimated effort:
  - Compute: 130-190 GPU-hours for 4 stress families, 3-5 PDEs per family, 20-50 seeds where needed, plus candidate-pool true-error, refit, and small-tree/random search oracles
  - Data: available; analytic manufactured PDE cases are generated on the fly, no external dataset or pretrained weights
  - Implementation: 3-4 weeks
- Novelty quick-check: Drissi 2606.01122 gives a per-component diagnostic protocol for neural HJB-PIDE solvers, and Shikhman 2601.11428 stress-tests neural operators, but neither studies RL expression-tree PDE solvers or counterfactual oracle rescue. EGRL-SR and correlated-proxy reward-hacking work motivate reward ambiguity, while SAGE-Fit motivates the parameter-refit oracle; none gives a multi-axis FEX/RL-SR attribution protocol. The 2026-06-17 deep-lit pass found no direct oracle-gap failure taxonomy for FEX or RL-guided symbolic PDE solving.
- Strongest objection: The current positive evidence is refit-dominated; without candidate-pool true-error and enlarged-search oracles, the full taxonomy could collapse to "FEX needs stronger inner optimization" rather than a general failure-attribution protocol.
- Why we should do this: A failed FEX run currently gives no principled next action: enlarge the grammar, search longer, change the reward, or refit constants. Oracle-gap attribution makes that choice measurable and can turn FEX debugging into a reusable diagnostic procedure.
- Pilot:
  - Setup: Ran the existing local CUDA FEX runner on two non-Poisson manufactured PDE gates (`conservation_sine`, `helmholtz_sine`) with the same dim=4 `depth2_sub` tree, controller, operator set, and residual+boundary reward; then added fixed-skeleton capacity and fixed-action multi-start refit oracles.
  - Metric: Positive signal requires a failed standard run to be rescued by at least one non-capacity oracle under rel.L2 < 1e-4; residual <1e-8 is a supporting sanity check, not the rescue definition.
  - Result: Standard FEX failed 6/6 non-Poisson runs, with 56/120 deceptive epochs and standard final rel.L2 2.83e-2 to 7.07e-2. The analytic sine capacity oracle solved both PDE gates at rel.L2 5.01e-8. The new refit oracle kept each standard best action fixed and gave it 800 optimization steps from two random starts; it rescued 6/6 actions and 12/12 refit trials, with best refit rel.L2 1.63e-7 to 5.29e-7 and residual 1.71e-12 to 1.46e-11.
  - Signal: POSITIVE but narrowed. The early non-Poisson failures are not capacity failures and are not yet proof of pure reward/search failure; the first confirmed non-capacity bottleneck is continuous-parameter refit shortfall under the standard budget.

- Claims and Claims matrix: Strong claim: on a preregistered FEX/RL-SR stress suite, counterfactual oracle gaps give stable, non-exclusive failure attributions across held-out PDE families, with macro-F1 >=0.70 from traces that exclude oracle labels. Moderate claim supported by current pilot: standard FEX can sample rescuable sine-PDE structures but fail because the inner parameter fit is too weak; capacity is not the bottleneck in these gates. Null claim: if labels collapse or macro-F1 <=0.40, stage-wise debugging is unreliable because failures are coupled. Negative claim: if true-error and search oracles add no signal beyond refit, the paper should narrow to a parameter-refit diagnostic rather than a full taxonomy.
- Narrative: The paper asks: which counterfactual intervention would have saved this failed symbolic-regression run? v5 makes the story more honest: the first decisive oracle says "refit this structure harder" before blaming grammar or search.
- Experiments: Pre-register four stress families: oscillatory/frequency-separated PDEs, operator-set omission, tree-depth or representation mismatch, and collocation/boundary perturbation. For each family, run standard FEX, fixed-skeleton capacity oracle, fixed-action multi-start/SAGE-Fit-style refit, same-candidate-pool true-error selection, and small-tree exhaustive or larger random-controller search where feasible. Define rescue by rel.L2 <1e-4 on held-out points, report residual as an audit, mark every rescuing oracle for multi-label attribution, and report bootstrap confidence intervals plus threshold sensitivity for axis prevalence.
- Assets status: Analytic PDE cases, local FEX code, cross-PDE standard runs, capacity-oracle runs, and the new refit-oracle batch are documented in `workspace/fex-failure-taxonomy/data/MANIFEST.md`; no external asset download is needed.

<review date="2026-06-17">

## Novelty

- Score: 7/10
- Closest prior work: A Per-Component Diagnostic Protocol for Neural HJB-PIDE Solvers (Drissi, 2606.01122, 2026); Diagnosing Failure Modes of Neural Operators Across Diverse PDE Families (Shikhman, 2601.11428, 2026); DSO: Good-Structure-Bad-Score phenomenon (Hayes et al., 2505.10762, 2025); EGRL-SR (Sun et al., 2601.14693, 2026); SAGE-Fit (Wang et al., 2605.23272, 2026)
- Key differentiator: 经多源 arxiv/S2 交叉验证及 155+ 篇已读论文的 landscape，未发现任何直接针对 FEX/RL 表达式树 PDE 求解的 counterfactual oracle-gap 失败归因协议。Drissi 2606.01122 的 per-component HJB-PIDE 诊断在方法学上最接近（分项审计→定位故障），但针对 neural HJB-PIDE 求解器而非 RL 表达式树搜索，且诊断哲学是"找 bug"而非"counterfactual rescue"。Shikhman 2601.11428 对 FNO/DeepONet/CNO 做 stress-test，但测量 degradation 而非做 oracle rescue。DSO 的 Good-Structure-Bad-Score 是单轴现象观察，非多轴归因协议。EGRL-SR 和 SAGE-Fit 各自覆盖 reward ambiguity 和 parameter fitting 单轴，均非多轴 orcale 协议。RLHF reward hacking 文献（2604.12086, 2403.03185）提供 proxy-true correlation 语言但未用于 SR。v5 新增的 refit oracle 在方法学上最接近 SAGE-Fit，但将其嵌入多轴 oracle rescue 协议是新增贡献。创新度集中在：(a) 反事实 oracle rescue 作为 RL-SR 归因工具而非被动分类；(b) 非互斥多因标签框架；(c) cross-stress-family 稳定归因验证设计。需注意：若最终 evidence 仅支撑 refit 轴，则 novelty 从"multi-axis taxonomy"退化为"SAGE-Fit 在 FEX/PDE 上的应用"，framing 需相应调整。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v5 的逻辑链清晰：capacity oracle 失败→表示不足；refit oracle 能救回→参数拟合短板；true-error oracle 能救回→reward 代理错排；search oracle 能救回而 controller 不能→搜索不足。剩余 gap：(a) oracle 间并非完全独立——true-error oracle 改变选择目标后参数优化轨迹也不同，无法纯粹隔离 reward misranking；(b) 当多个 oracle 同时 rescue 时，归因是联合的而非分离的，v5 的非互斥框架可处理此但需在实验设计中显式分析 oracle overlap pattern；(c) true-error 和 search oracle 的 rescue 操作化定义仍模糊（需多少搜索预算、true-error 评估需要多少 collocation 点？）。 |
| Missing evidence signals | 6/10 | v5 相比 v4 有实质证据升级：refit oracle 6/6 rescue（rel.L2 1.63e-7 至 5.29e-7）是首次真实非 capacity oracle rescue 信号，证实参数拟合是主导瓶颈。仍缺失：(a) candidate-pool true-error oracle 结果；(b) 扩大/随机 search oracle 结果；(c) held-out stress family 上的归因稳定性；(d) trace-level feature 的 macro-F1 测量；(e) bootstrap CI 和 power analysis（v5 承诺了但未执行）；(f) 10/20/30% 阈值排序稳定性的理由。 |
| Narrative | 8/10 | "Counterfactual Oracle-Gap Attribution" 保持锐利。"Which intervention would have saved this failed run?" 是出色的 paper-level question。v5 叙事比 v4 更诚实："the first decisive oracle says 'refit this structure harder' before blaming grammar or search"——这在叙事层面是正确的谦逊。叙事弱点：headline scope（full taxonomy）与当前 evidence（refit-dominated）之间存在张力，审稿人会注意到。若后续 true-error/search oracle 不产生独立信号，paper scope 需收窄至 refit diagnostic。 |
| Venue contribution | 6/10 | topic 未声明 target-venue。按 FEX 系列历史 venue (JMLR/JCP) 及顶会 (ICML/NeurIPS/ICLR) 标准评估。纯 diagnostic+empirical-finding 在顶会处于 borderline。当前 evidence 仅支撑一个非 capacity 轴（refit），离 top-venue 充分条件（>=2 个可分离非 capacity 轴 + 跨 stress family 稳定 + 一个反直觉 bottleneck ranking）仍有距离。若仅 refit 轴成立，story 退化为"SAGE-Fit for FEX/PDE"，对顶会不够但对 JMLR/JCP 可能足够。 |
| Testability | 8/10 | v5 包含明确的 cheapest falsifying signal：若 true-error 和 search oracle 不产生 refit 以外的差异→降级为 parameter-refit diagnostic。POSITIVE（macro-F1 >= 0.70，>=3 stress families，>=20% affected seeds）和 NULL（macro-F1 <= 0.40）均有量化阈值。改进建议：NULL 的"FEX failures are coupled" 结论可能过强——低 macro-F1 也可能因为 trace feature 太弱或 label 噪声大，不必然证明 fail 是耦合的。建议 NULL 改为"this trace-based attribution protocol does not yield predictive stage-separable labels"。 |
| Outcome realism | 6/10 | 130-190 GPU-hours 在已有 instrumentation 的前提下可行。但风险不在 compute 而在于：(a) macro-F1 >= 0.70 从 non-label trace features 实现是非常 ambitious 的目标——trace features（residual 历史、reward 方差、action entropy 等）未必包含足够的预测信号；(b) stress PDE 设计可能过度耦合于特定 failure axis 而影响归因的 external validity；(c) true-error oracle 的 candidate pool evaluation 成本随候选池大小线性增长。 |
| Contribution type compliance | n.a. | idea types (diagnostic+empirical-finding)。topic 未声明 `preferred-contribution-types`，跳过 compliance 检查。不计入 Overall Quality 平均。 |
| Overall Quality | 7/10 | v5 是 v4→v5 的实质升级：refit oracle pilot 提供首个真实非 capacity oracle rescue 信号（6/6，rel.L2 1e-7 级），叙事变得更诚实（承认 refit dominance），experiment 设计也更细化。当前状态是 well-formed proposal with genuinely positive pilot evidence concentrated on one axis。下一决定性实验不应是 full stress sweep，而应是小而暴力的 kill-test：same fixed stress set，四类 oracle 全部跑通，preregistered decision rule。若仅 refit oracle 产生 signal，pivot 到 narrower paper immediately。 |

## Contribution Drift (n=5)

- v_{4} contribution types: {diagnostic, empirical-finding}
- v_{5} contribution types: {diagnostic, empirical-finding}
- Status: unchanged
- Silent downgrade/expansion: none. v5 explicitly acknowledges that the current evidence is refit-dominated and that the full taxonomy could collapse, which is honest disclosure, not silent downgrade.
- Substantive note (not a hard cap trigger): v5's evidence center has shifted — v4 had all four axes as open hypotheses, while v5 now has strong evidence only on the refit axis. The headline "Counterfactual Oracle-Gap Attribution" still suggests full taxonomy scope. This is a framing tension, not a contribution type violation.
- Hard cap triggered: no (topic 未声明 preferred-contribution-types)

## Prior Review Concern Resolution (v4 → v5)

| v4 Concern | Status | Notes |
|-----------|--------|-------|
| 缺乏 non-capacity oracle 证据 | Partially resolved | refit oracle 6/6 rescue 是首次真实非 capacity oracle rescue 信号，但仅覆盖一个轴 |
| 需要 true-error reward oracle | Ignored / pending | v5 无 true-error oracle 实验证据，仅作为 planned experiment 提及 |
| 需要 search oracle | Ignored / pending | v5 无 search oracle 实验证据。由于 standard FEX 已 sample 到 rescuable 结构，search 可能不是瓶颈反而被现有 evidence 弱化——v5 应讨论这一推断 |
| capacity oracle 仅 sanity check | Resolved | refit oracle 使 capacity 不再单独背负全文，当前 evidence 重心转移至 refit 轴 |
| rescue 操作化定义模糊 | Mostly resolved | v5 使用 rel.L2 < 1e-4 作为 rescue 定义，residual 作为 audit，比 v4 更精确 |
| 阈值敏感性缺失 | Partially resolved | v5 加入 10/20/30% 排序稳定性测试，但仍无 power justification |
| held-out stress family 稳定性 | Not resolved | v5 仍是 planned，无跨 family 稳定性 evidence |
| CI / power analysis | Not resolved | v5 提及 bootstrap CI 但未执行 |
| artifact/benchmark pushback | Partially resolved | v5 保持 diagnostic framing，但对顶会而言 reusable stress protocol 仍重要 |

Refiner pushback 评估：v5 对 v4 review 中"将贡献与 benchmark/stress-suite 解耦"的 pushback 在当前 evidence 阶段合理——先证明 protocol 本身有效，再讨论 artifact release。但若进入 full stress sweep 阶段后仍拒绝 release stress suite，审稿人会质疑 replicability。Pushback 在当前阶段接受，后续需重新评估。

## Alternative Framing

当前 "Counterfactual Oracle-Gap Attribution" 是最锐利的 framing，保持。但建议在 paper 中将"taxonomy"语言改为"intervention ranking"语言——"rank which counterfactual intervention rescues failures under matched budget" 比 "assign failure to a category" 更准确且更易辩护。Codex 提出的 "Sampled but Unfit: Fixed-Action Refit Oracles Reveal Inner-Optimization Failure in FEX" 是更窄但更诚实的当前 evidence framing，适合作为 moderate claim 而非 headline。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在预注册的 FEX/RL-SR stress-test PDE suite 上，counterfactual oracle gaps 可在 >=3 个 stress family 上产生稳定、非互斥的 failure attribution，每轴在 >=20% affected seeds 中复现，基于非标签诊断特征的 held-out macro-F1 >= 0.70，且至少一个 bottleneck ranking 反直觉。**注意：此 claim 当前无 evidence 支撑；仅 refit 轴有 pilot evidence。** |
| NULL | 若 macro-F1 <= 0.40，则基于 trace features 的 oracle-rescue 预测不可靠——这不必然证明 FEX failures 本身是耦合的，可能仅说明 trace features 信息量不足或 label noise 过大。建议 NULL 精化为"this trace-based attribution protocol does not yield predictive stage-separable labels"。 |
| NEGATIVE | 若 true-error 和 search oracles 不产生 refit 以外的 differential rescue 信号，工作应窄化为 parameter-refit diagnostic：FEX 在 non-Poisson manufactured PDE 上常 sample 到正确结构但因内层参数优化不足而失败。 |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**: pilot 证据持续为正——v4 的 capacity oracle (rel.L2 5.01e-8) + v5 的 refit oracle (6/6 rescue, rel.L2 1e-7 级) 证实了两类 oracle 的 rescue 可行性。实验路径清晰，compute 预算可行。但 top-venue 成稿依赖多个未验证条件：(a) true-error 和 search oracles 需产生独立且 nontrivial 的 rescue 信号——当前 evidence 反而暗示 search 可能不是瓶颈（standard FEX 已 sample 到 rescuable 结构），若 true-error oracle 也不产生新 signal，paper 将为 refit-only；(b) non-label trace features 需在不泄露 oracle 信息的前提下达到 macro-F1 >= 0.70——这极度 ambitious；(c) 归因需在 held-out stress families 上稳定。**Impact=High**: Claude 与 codex 一致判定为 High。若成功，本工作将为 RL-SR/FEX 社区提供首个可操作的 counterfactual 归因协议——不再是对失败的模糊诊断，而是直接回答"哪个反事实干预能救回这个失败？"并据此指导方法改进方向。counterfactual oracle rescue 方法论可作为 RL-for-scientific-discovery 管线诊断的范本。Impact 不到 Exceptional 因为 scope 限定在 RL-SR/FEX niche，不构成 broad ML paradigm shift。**Codex 与 Claude 分歧**: 无 >=1 level 分歧。双方一致 Likelihood=Medium, Impact=High, Priority=High(7)。Codex 额外指出若 evidence 仅支撑 refit 轴则 Impact 降为 Medium，此判断合理但属于 condition-dependent 而非当前分歧。

## Overall

- Priority: High
- Score: 7
- Comments: v5 是 v4→v5 的实质升级——refit oracle 6/6 rescue (rel.L2 ~1e-7) 是首个真实非 capacity oracle rescue 信号，证明参数拟合是 FEX 在 non-Poisson PDE 上的主导瓶颈。叙事更诚实："先 refit 好再看 grammar/search"。但 headline "Counterfactual Oracle-Gap Attribution" 的全分类学 scope 与当前 refit-dominated evidence 之间存在张力——这是 v5 最诚实的自指："The current positive evidence is refit-dominated; without candidate-pool true-error and enlarged-search oracles, the full taxonomy could collapse to 'FEX needs stronger inner optimization'." Claude 与 codex 均建议下一实验应小而暴力：相同 stress set + 四类 oracle 全跑 + preregistered decision rule。若仅 refit oracle 产生 signal，pivot 到 narrower "good structure, bad fit in FEX" paper immediately。当前不做 full stress sweep。

</review>
