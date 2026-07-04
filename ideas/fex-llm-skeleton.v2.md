---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-llm-skeleton/
---

- One-sentence summary: 将 FEX+LLM 从"LLM 直接写完整骨架"改成结构先验粒度研究: 比较 operator-set、子树、完整骨架、oracle 与 corrupted prior, 并把 top-k LLM 骨架作为 FEX controller 的软偏置而非硬模板.
- Hypothesis: FEX 的搜索瓶颈同时来自算子选择、算子排列和子树结构; 正确完整骨架能让参数优化快速收敛, 但稍错的硬骨架会误导搜索, 因此最稳的机制应是 top-k 骨架给 controller 提供可恢复的软 prior.
- Expected outcome: 成功信号是 soft skeleton prior 在标准 FEX PDE 和 anti-memorization PDE 上, 以相同 rel-L2 阈值将 median wall-clock/evaluation budget 相比 from-scratch FEX 和 Bhatnagar-style operator-set prior 至少降到 1/2, 且 corrupted skeleton 下不比 operator-set prior 差超过 10%; 失败信号是 operator-set prior 已经匹配完整骨架, 或 soft prior 在 OOD PDE 上稳定拖累 controller.
- Contribution type: method + diagnostic
- Risk: MEDIUM
- Estimated effort:
  - Compute: 40-80 GPU-hours for FEX controller sweeps plus small LLM/API cost; current pilots used local RTX 4060 Ti.
  - Data: available; analytic PDE cases and generated collocation metadata are in `workspace/fex-llm-skeleton/data/MANIFEST.md`, with no annotation needed.
  - Implementation: 2-3 weeks
- Novelty quick-check: Bhatnagar et al. (2503.09986) predicts operator sets for FEX but does not test richer prior granularity or controller injection. LLM-SR, SR-LLM, FunctionEvolve, STRIDE, and KeplerAgent already cover "LLM prior + symbolic search" for data-driven equation discovery, so this idea should not claim novelty for LLM skeleton generation itself. The clean difference is PDE solving with FEX's residual/boundary objective, plus a controlled study of how hard or soft each prior granularity should be.
- Strongest objection: The phase-map framing may still look diagnostic rather than method-level unless soft controller bias gives a reliable speedup beyond a strong operator-set-only FEX baseline.
- Why we should do this: This answers a concrete FEX design question for the Yang group: should LLMs provide only allowed operators, reusable subtrees, or full candidate skeletons? Even a null result is useful if it shows that full skeletons add no value over cheaper operator priors.
- Pilot:
  - Setup: On local CUDA, fit candidate skeletons with the FEX-style PDE residual plus boundary loss on textbook Poisson d=5, a rotated quadratic d=5, and a sine-interaction d=4 anti-memorization case; compare full skeletons, partial skeletons, corrupted hard skeletons, and a random operator-set-only proxy.
  - Metric: rel-L2 < 1e-3 counts as high-accuracy recovery; the pilot is positive only if full skeletons clear the threshold while corrupted/partial/operator-only proxies expose separable failure modes.
  - Result: Full skeletons reached 6.27e-7, 9.04e-7, and 1.59e-6 on the three cases. Corrupted hard skeletons degraded to 9.76e-3, 9.91e-3, and 1.34e-2; partial skeletons were 2.60e-1, 1.30e-1, and 4.32e-2; the best random operator-set proxy stayed between 4.21e-1 and 4.83e-1. The earlier DeepSeek Poisson pilot also found the correct sum-of-squares skeleton, but that case is too easy and likely memorized.
  - Signal: POSITIVE, but only for the revised prior-granularity question; it is not yet evidence that the real FEX controller accelerates.

- Claims and Claims matrix: Main claim: FEX needs a recoverable structural prior rather than a hard LLM-written formula. Positive outcome supports "soft top-k skeleton bias improves search efficiency over operator-set-only while preserving fallback behavior"; null outcome supports "operator-set priors are the right granularity for FEX"; negative outcome supports "LLM skeletons are too brittle for FEX unless repaired or downgraded to soft masks." None of these outcomes supports a broad claim that LLMs solve symbolic PDE search.
- Narrative: The story centers on trust calibration: how much structure should a PDE symbolic solver borrow from an LLM before the prior becomes a liability?
- Experiments: First, build a prior-granularity sweep with oracle, LLM top-k, subtree, operator-set-only, random, and corrupted priors across Poisson, conservation law, Schrodinger, multi-scale, and anti-memorization analytic PDEs. Second, compare injection mechanisms: hard initialization, grammar mask, KL/controller-logit bias, and top-k mixture restart. Third, use from-scratch FEX, Bhatnagar operator-set FEX, LLM-SR-style skeleton+BFGS, and recent LLM-SR systems as baselines or boundary comparisons; report wall-clock, evaluations to threshold, final rel-L2, prior failure rate, and recovery after corrupted priors.
- Assets status: Data/model assets are available for the idea stage; current analytic metadata, commands, and handoff notes are in `workspace/fex-llm-skeleton/data/MANIFEST.md`, with no exceptions.

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: FunctionEvolve (2606.07704, NeurIPS 2026) — LLM+AST-guided evolution for SR, 82.9% SA@50; SymPlex (2602.03816, ICML 2026) — structure-aware Transformer + RL for symbolic PDE solving; KeplerAgent (2602.12259) — LLM agent + physics tools + SR backend; SR-LLM (Guo et al. 2025, PNAS) — LLM+RAG skeleton + DRL assembly
- Key differentiator: (1) 系统比较 FEX PDE 求解中 prior 粒度的相图 (operator-set / subtree / full skeleton / oracle / corrupted) — 无现有工作做此系统性对照; (2) 面向 PDE 求解 (residual/boundary objective + Adam/BFGS) 而非数据驱动方程发现; (3) top-k LLM 骨架作为 FEX policy-gradient controller 的可恢复软偏置 (KL/controller-logit bias/mixture restart) 而非硬模板. 但核心概念 "LLM prior + 搜索后端" 已被 LLM-SR, FunctionEvolve, KeplerAgent, STRIDE 等充分覆盖; novelty 主要来自诊断设计和 FEX-specific 注入机制, 非全新方法范式. v2 的 reframing 从 "apply LLM skeleton to FEX" 改为 "结构先验粒度相图" 显著提升了 novelty (v1: 5/10 → v2: 6/10), 但 FunctionEvolve (2026.06) 和 Deliberate Evolution (ICML 2026) 的涌现进一步压缩了 "LLM+搜索" 的 novelty 空间.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v2 的逻辑链清晰: FEX 瓶颈 → 结构先验有帮助 → 但错误先验有害 → 需校准粒度 → 软 prior 可恢复. 但软 prior 的具体实现机制 (top-k 骨架如何转为 controller logit bias/KL/grammar mask, 如何保持 support coverage, 超参如何选取) 仍模糊. "corrupted skeleton 下不比 operator-set prior 差超过 10%" 的宣称缺乏对 "corrupted" 的操作化定义. |
| Missing evidence signals | 5/10 | Pilot 仅测试了直接骨架拟合 (非真实 FEX controller 加速), 缺失: (1) 真实 FEX controller sweep 结果, (2) multi-seed 稳定性, (3) LLM top-k 骨架在 anti-memorization PDE 上的质量, (4) 强 operator-set baseline (Bhatnagar-style, 而非 random proxy), (5) SymPlex/SSDE 作为 PDE 符号求解 baseline 对照, (6) wall-clock 时间测量, (7) SAGE-Fit 风格的结构感知参数拟合 (因骨架评分公平性直接影响结论), (8) Yang 组 Domain-Aware Symbolic Priors (2503.09592) 的统计 prior 作为对照粒度. |
| Narrative | 7/10 | "信任校准" 叙事清晰有力: "一个 PDE 符号求解器应该从 LLM 借多少结构, prior 才会从资产变成负债?" 这是 genuine research question. 但需避免听起来像 Yang 组内部 ablation; 应 framing 为 "可恢复符号先验在科学搜索中的粒度规律", 将 FEX 作为实例化平台而非全部贡献范围. |
| Venue contribution | 6/10 | 对 NeurIPS/ICML: 若 phase-map 结果强且反直觉 (如 soft prior 系统性优于 hard, 或 operator-set 在多数 PDE 族上已足够), 有顶会潜力. 但当前证据仅 support 一个很好的诊断论文; 方法贡献 (soft prior injection) 是增量式的. 对 JCP/SISC 等计算物理 venue: 贡献更充分. 关键 risk: SymPlex (ICML 2026) 和 SSDE (ICML 2025) 已占据 "符号 PDE 求解" 空间, 本 idea 需明确 differentiate. |
| Testability | 8/10 | POSITIVE 信号 (soft prior 将 budget 降到 ≤ 1/2, corrupted 下不差于 operator-set >10%) 具体可测. NULL 信号 (operator-set 已匹配完整骨架) 和 NEGATIVE 信号 (soft prior 在 OOD 上拖累 controller) 均有清晰 falsification 条件. Anti-memorization PDE 的纳入是关键改进. Pilot 廉价可复现. |
| Outcome realism | 6/10 | 40-80 GPU-hours 和 2-3 周实施周期合理. 但实现 claimed speedup (1/2 budget) 面临几个风险: (1) pilot 只测了直接骨架拟合, 非 FEX controller 加速, (2) LLM inference 成本和多候选探索 overhead 可能侵蚀 wall-clock 收益, (3) strong operator-set baseline (Bhatnagar-style) 可能已接近 oracle skeleton 性能, 压缩 soft prior 的边际增益. Pilot 的 POSITIVE signal 值得鼓励但尚未构成 controller-level 证据. |
| Contribution type compliance | n.a. | topic 未声明 preferred-contribution-types, 跳过检查. |
| Overall Quality | 6/10 | v2 相比 v1 是实质性改进: claim bounded, falsifiable, 不再是 "apply X to Y". 当前状态是 well-formed diagnostic idea, 有明确实验路径. 顶会发表仍需 controller-level 的意外发现 (如 operator-set 在多数 PDE 族上已足够, 或 soft prior 在特定 PDE 族上有相变式优势). |

## Prior Review Follow-up (v1 → v2)

- "排列组合是主要搜索瓶颈缺乏证据": **resolved** — v2 放弃该 claim, 改为 prior 粒度研究.
- "缺少 oracle/operator-set-only/random/corrupted/OOD 对照": **partially resolved** — v2 列出全部对照并扩展 pilot, 但多数仍为 planned 状态, pilot 未含真实 controller sweep.
- "工程延伸叙事": **resolved** — 采纳 Alternative Framing, 改为 "trust calibration / prior 粒度相图".
- "Poisson pilot 易被记忆": **partially resolved** — 新增 rotated quadratic 和 sine-interaction anti-memorization case, 但 LLM 生成的 top-k 骨架测试仍缺失.
- "claims 太宽": **resolved** — v2 claims matrix 严格限定 POSITIVE/NULL/NEGATIVE 边界.
- v1 reviewer 建议的 SAGE-Fit 和 SymPlex 对照未被纳入实验设计: SAGE-Fit 的参数拟合公平性对此 idea 的骨架评分可靠性至关重要, 建议在实验中作为 ablation 纳入.

## Contribution Drift

- v_{n-1} contribution types: {method}
- v_n contribution types: {method, diagnostic}
- Status: expanded(+diagnostic)
- Hard cap triggered: no (topic 未声明 preferred-contribution-types, 跳过所有 hard cap 检查)

## Alternative Framing

当前 framing 已接近最优. 更锋利的版本:

**Recoverable symbolic priors for PDE solution search: when should an LLM provide operators, subtrees, full skeletons, or only a soft bias?**

此 framing 将贡献从 FEX-specific 提升为 "科学搜索中可恢复符号先验的粒度规律", FEX 作为实例化平台. 核心结果应为 prior 粒度 × 注入硬度 × PDE 族 × corruption level 的 phase map. 这使 idea 更不易被 "LLM-SR already generates skeletons" 或 "SymPlex already solves PDEs symbolically" 轻易毙掉.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在限定 FEX PDE benchmark 和 anti-memorization 设置下, soft top-k skeleton prior (controller logit bias / KL / mixture restart) 相比 from-scratch FEX 和 Bhatnagar-style operator-set prior 将 median budget 降到 ≤ 1/2, 且 corrupted skeleton 下不比 operator-set prior 差超过 10%. **不能声称** LLM skeleton 普遍优于其他 prior 形式, 或结论推广到所有 SR 方法. |
| NULL | 若 operator-set prior 已匹配或超越 skeleton prior: 只能声称 "在测试 PDE 族和 FEX 框架下, 完整骨架先验未提供超越算子集先验的额外搜索效率增益". **不能声称** LLM 科学先验对 PDE 求解无价值, 或否定 LLM+SR 总路线. |
| NEGATIVE | 若 hard skeleton 系统性误导 FEX controller 且 soft prior 在 OOD 上拖累性能: 可声称 "硬骨架先验对 FEX policy-gradient 搜索过于脆弱; FEX 更适合 operator-set 级先验或仅在 controller 层面使用极软偏置". **不能声称** LLM 不适合 PDE 求解 — 可能只是注入机制需要改进. |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood (Medium): 有明确成稿路径 (prior 粒度 sweep → controller 注入机制 comparison → phase map 产出). Yang 组有 FEX 基础设施和 Bhatnagar et al. 作为直接前驱; pilot 已初步验证不同粒度先验在直接拟合下有可分离的失效模式. 但做成 top-venue 级结果依赖若干条件: (1) 真实 FEX controller sweep 显示 soft prior 有超越 strong operator-set baseline 的显著加速, (2) LLM top-k 骨架在 anti-memorization PDE 上保持质量, (3) phase map 揭示反直觉规律而非仅确认直觉. 若任一条件不成立, 成稿仍是可能的 (作为诊断论文), 但顶会概率显著下降.
  - Impact (Medium): 如果成功 (揭示清晰的 prior 粒度相图 + soft prior 系统性优势), 会对 FEX 设计空间和 LLM-guided symbolic PDE solving 产生清楚影响, 并为更广泛的 "科学搜索中 LLM 先验校准" 提供实证规律. 但不太可能改变整个 SR 或 LLM-SR 领域判断 — FEX 本身是 niche 方法, 且 FunctionEvolve/KeplerAgent/SymPlex 等竞品在各自路线上快速推进.
  - Claude 与 codex 无分歧 (双方一致: Likelihood=Medium, Impact=Medium).

## Deep Lit 2026-06-16 Update

本次 deep-lit 续扫 (11 篇新 wiki) 发现以下与本 idea 直接相关的新文献:

- **NeuroSym-BO (2601.00088)** [wiki](wiki/2601.00088.md): LLM 生成候选 PDE → Bayesian Optimization 动态选择指令。为 LLM+FEX 骨架生成提供替代范式：不直接让 LLM 输出表达式树，而是用 BO 驱动 LLM prompt 选择。当前仅在 1D PDE 且 pipeline 脆弱，但思路对 soft prior 注入机制的设计有启发。
- **DSO (2505.10762)** [wiki](wiki/2505.10762.md): Wikipedia + Set Transformer 预训练 → 为 LLM 先验注入 FEX controller 提供了非 LLM 的替代方案 (pretrained embedding as prior)。DSO 的 GP seeding 也展示了 "外部知识 warm-start RL 搜索" 的有效性。
- **Data-driven discovery of governing DEs review (2606.09638)** [wiki](wiki/2606.09638.md): 提出 LLM equation discovery 的 evaluation protocol（反记忆测试、cross-PDE transfer）。为本 idea 的 LLM 骨架质量评估提供方法论参考。
- **FunctionEvolve (2606.07704)** 和 **Deliberate Evolution (2606.04360)** 已在 landscape §6 中收录，继续压缩 "LLM+搜索" novelty 空间，但均面向 data-driven SR 而非 PDE 求解。

## Deep Lit 2026-06-16 Update II (idea-scope: fex-llm-skeleton)

本次由 `deep-lit-tick --scope idea 0616-fex --idea fex-llm-skeleton` 运行 2 轮搜索+精读，R2 B7 产出 0 新候选后饱和。共新读 5 篇（+ reader 交叉引用产出 5 篇辅助 wiki）。

### 直接相关：Symbolic Skeleton Prediction 工作线

- **Univariate Skeleton Prediction (2406.17834)** [wiki](wiki/2406.17834.md): Giorgio Morales & John Sheppard (MSU), 2024. **首个提出 Multi-Set Symbolic Skeleton Prediction 的论文**。为每个变量生成 counterfactual 点集 → Multi-Set Transformer → 单变量符号骨架。13 个合成方程上骨架基本恢复。对本 idea 的影响：若做 per-variable skeleton explanation 则强撞车；若做完整多变量 PDE 解表达式(skeleton + RL controller + Adam/BFGS)，差异化明显。可复用 ISAB/PMA 多集合编码器、avoidNaNs 表达式生成和 GA skeleton equivalence metric。
- **Decomposable Neuro SR (2511.04124)** [wiki](wiki/2511.04124.md): Morales & Sheppard, 2025. 2406.17834 的后续工作，加入 GP-based cascade 合并单变量 skeleton 为多变量表达式。已在 landscape 先前的 deep-lit 中收录，本次确认其 skeleton 生成 pipeline 完整性。
- **Neuro SR for N-Response Curves (2605.31276)** [wiki](wiki/2605.31276.md): Morales & Sheppard, 2026. 将 skeleton prediction 应用到精准农业。加入 ASPINN 自适应采样降低 epistemic uncertainty，SeTGAP 在所有管理区拟合误差低于传统模型。对本 idea 的启发：(a) 自适应采样可作 FEX collocation point selection 的参考，(b) skeleton 跨子域共享验证了 "共享结构骨架" 的思路。**不撞车**：应用是农业，验证依赖 NN 曲线而非 PDE residual，无公开代码。

> **关键判断**：Morales/Sheppard 的 skeleton prediction 工作线(2406.17834 → 2511.04124 → 2605.31276)使用 Multi-Set Transformer 生成符号骨架 + GA 拟合参数，这与本 idea 的 "LLM skeleton" 有表面相似但本质不同：(1) 他们从黑盒 NN 曲线蒸馏 skeleton 而非从 LLM 获取科学先验，(2) 没有 PDE residual/boundary objective，(3) 没有 RL controller 搜索，(4) 没有 prior granularity 系统比较。**不构成直接撞车，但 skeleton 概念重叠需要在论文中明确 differentiate。**

### 直接相关：SR 中的先验与模型选择

- **Priors for Symbolic Regression (2304.06333)** [wiki](wiki/2304.06333.md): Deaglan Bartlett, Harry Desmond, Pedro Ferreira (Oxford), 2023. **将 n-gram 语言模型 (Katz back-off) 作为 SR 的函数结构先验**，结合 Fractional Bayes Factor (FBF) 做 Bayesian model selection。在 Pantheon+ 宇宙学数据上，语言模型先验压下物理上不自然的 x^x 表达式，LambdaCDM 进入 top-4。对本 idea 的核心价值：(a) 提供了 "如何定义和量化符号表达式结构先验" 的 baseline——本 idea 的 LLM skeleton prior 可以与之对比 (n-gram prior vs LLM prior)，(b) FBF 可为 skeleton 质量评估提供 principled scoring，(c) Bayesian evidence 框架可为 prior granularity phase map 提供理论语言。**不撞车**：本文做的是模型选择 ranking 而非搜索过程中的 prior injection；不做 PDE 求解。

> **对本 idea 的启发**：可以构建一个 "prior 强度谱"：n-gram LM prior (Bartlett 2023) → domain-aware symbol prior (Huang 2503.09592) → LLM operator-set prior (Bhatnagar 2503.09986) → LLM skeleton prior (本 idea) → oracle prior。这使 idea 的 "prior granularity" 叙事有了清晰的理论参照系。

### 工具层：Benchmark 与评测

- **SURFACEBENCH (2511.10833)** [wiki](wiki/2511.10833.md): Kabra, Shojaee, Reddy (Virginia Tech), 2025. 首个 3D 曲面符号发现 geometry-aware benchmark：183 个解析构造曲面、15 个类别、3 种表示范式。核心发现：现有方法远未解决该任务(LLM-based exact recovery 仅 4%)，LLM 有结构先验但参数校准、多方程耦合和 OOD/噪声鲁棒性弱。对本 idea：(a) 可作为额外的 OOD 评测平台验证 LLM skeleton 在非 PDE 符号任务上的质量，(b) failure taxonomy (参数校准弱、多方程耦合差) 与本 idea 的 "corrupted skeleton" 实验设计共鸣，(c) Chamfer/Hausdorff 几何指标可作为 skeleton structural equivalence 的评估参考。**不撞车**：它是 benchmark 而非方法。

- **UQ in SR Survey (2606.06567)** [wiki](wiki/2606.06567.md): Reuter & Franca, 2026. 符号回归中不确定性量化的系统综述：整理 18 篇 SR-UQ 工作(Bayesian 11 篇 + non-Bayesian 4 篇 + 模型选择 3 篇)。对本 idea 的价值：(a) 提供 skeleton 质量的 UQ 评估方法论(profile likelihood / conformal prediction / Bayesian posterior / MDL)，(b) 如果 FEX 从干净 PDE 求解扩展到 noisy physical law discovery，UQ 管线必需，(c) 结构不确定性量化可直接支撑 "soft prior 何时 degrade 为 hard prior 的判定准则"。**不撞车**：综述，无方法竞争。

### 辅助：交叉引用发现的补充文献

- **HierBOSSS (2509.19710)** [wiki](wiki/2509.19710.md): Roy, Pati, Mallick (TAMU), 2025. 全贝叶斯符号回归树集成，Metropolis-within-partially-collapsed Gibbs 采样，Occam's window 模型选择。提供了 Bayesian tree prior (operator/feature Dirichlet + depth-dependent split probability) 的完整示例，可作为本 idea "soft prior 注入 FEX controller" 的 Bayesian 参照。
- **Curriculum in Transformer Tree-Reasoning (2511.07372)** [wiki](wiki/2511.07372.md): Bu, Huang, Suzuki 等, 2025. **证明 curriculum post-training 将树推理采样复杂度从指数降至多项式**。这对本 idea 的 "skeleton 复杂度 curriculum" 设计有直接理论支撑：如果 LLM skeleton 按深度/复杂度分层注入，理论上可以显著降低 FEX controller 的采样预算。
- **SR+RL for Interatomic Potentials (2511.20506)** [wiki](wiki/2511.20506.md): Varughese et al. (UIC/Argonne), 2025. EqNN + continuous-action MCTS 学习 Cu 原子间势函数。展示了 RL (MCTS) + SR 在物理科学中的另一个应用场景，与本 idea 的 RL+SR 范式平行但不重叠(PDE 求解 vs 材料势函数)。
- **Guided Residual Search (2602.22964)** [wiki](wiki/2602.22964.md): Floren & Swevers (KU Leuven), 2026. 非线性状态空间辨识中的 guided residual search (GRS) + multiple shooting。与 FEX 共享 "residual-guided search" 精神但方法学正交(state-space LFR vs 符号表达式树)。
- **VaSST (2602.23561)** [wiki](wiki/2602.23561.md): 已在 landscape §8 中收录，此处确认其在 Bayesian UQ 与软符号树方面的互补价值。

### 更新后的 novelty 评估

| Dimension | 更新前 (v2 review) | 更新后 (本轮 deep-lit 后) |
|-----------|-------------------|------------------------|
| Skeleton 概念的已有工作 | 仅知道 LLM-SR/SR-LLM/FunctionEvolve 有 skeleton 概念 | 新增 Morales/Sheppard 完整工作线 (3 篇)，skeleton prediction 是独立方向但面向 data-driven SR + 农业 |
| 结构先验的形式化 | n-gram prior (Bartlett 2023) 未收录 | Bartlett 2023 提供 "prior 强度谱" 的语言和学习基线 |
| Prior 粒度系统比较 | 无前人做 | 确认仍无前人做——这是 idea 最干净的 novelty 点 |
| 理论支撑 | RL convergence theory 空白 | Curriculum tree-reasoning (2511.07372) 提供 curriculum 降低采样复杂度的理论，可支撑 skeleton 复杂度 curriculum |
| Benchmark 与评测 | 仅知 MDBench/ERBench | SURFACEBENCH 提供 3D 几何评测维度，UQ survey 提供 skeleton 质量 UQ 框架 |

**总体判断**：本轮 deep-lit 未发现直接撞车者。Morales/Sheppard skeleton prediction 是最接近的工作线，但面向完全不同的 setting (NN 蒸馏 + 农业数据 vs LLM prior + PDE residual + RL controller)。Bartlett (2023) 的结构先验为本 idea 提供了理论参照系而非竞争。SURFACEBENCH 和 UQ survey 提供了评测工具。idea 保持差异化空间。

## Overall

- Priority: Medium
- Score: 5
- Comments: v2 相比 v1 有实质性改进 — reframing 成功将 idea 从弱方法贡献升级为有发表价值的机制诊断. Pilot evidence 是 POSITIVE 但仅覆盖直接骨架拟合, 尚未到达 controller-level. 核心风险: (1) FunctionEvolve (2026.06) 和 Deliberate Evolution (ICML 2026) 进一步压缩 "LLM+搜索" 的 novelty 空间, (2) strong operator-set baseline 可能已接近 oracle skeleton 性能, (3) soft prior 注入机制的具体设计仍是 underspecified. 建议下一步: 在真实 FEX controller sweep 上跑 multi-granularity comparison, 加入 SymPlex 作为 PDE 符号求解 baseline, 并纳入 SAGE-Fit 风格参数拟合以避免 "Good Structure, Bad Score" 问题污染结论. Claude 与 codex 在 Likelihood 和 Impact 上均一致 (Medium/Medium).

</review>
