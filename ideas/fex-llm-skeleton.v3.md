---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-llm-skeleton/
---

- One-sentence summary: 将 FEX+LLM 收窄为可恢复的符号先验: LLM 不替 FEX 写死解的骨架, 而是给 controller 一个保留 fallback support 的 top-k 结构先验, 并比较 operator-set、统计结构先验、subtree、full skeleton、oracle 和 corrupted prior 的粒度边界.
- Problem anchor: "FEX 是我的课题组(Haizhao Yang 组)发明的一种方法, 使用 RL 进行 Symbolic Regression, 我(Youran Sun)作为 Yang 的博后, 应该继续在这个方向上探索"
- Hypothesis: FEX 的瓶颈不是单纯缺少完整骨架, 而是 controller 在离散结构空间里缺少可校准的先验. 正确 full skeleton 可让参数优化接近 oracle, 但错误 top-1 hard skeleton 会把搜索锁进坏结构. 如果 LLM top-k 或统计 prior 给出的是有噪声的结构分布, 软 logit/KL bias 或 mixture restart 应当比 hard template 更稳, 并在结构正确时接近 oracle.
- Expected outcome: 成功信号是在真实 FEX controller 上, soft top-k skeleton prior 在标准 PDE 和 anti-memorization PDE 上以相同 relative-L2 阈值把 median candidate evaluations 或 net wall-clock 降到 from-scratch FEX 和 Bhatnagar-style operator-set prior 的 <=1/2, 且在局部 corrupted top-1 skeleton 下不比 operator-set prior 差超过 10%. 最便宜的反证是: 真实 controller 中 operator-set prior 已匹配 skeleton prior, 或 soft prior 只在有限库 proxy 中有效而不能改善 FEX policy-gradient 搜索.
- Contribution type: method + diagnostic
- Contribution drift note: v2 contribution type 是 `method + diagnostic`; v3 仍是 `method + diagnostic`. 新增 controller proxy 和更强 baseline 要求只是收紧同一贡献, 不把本文扩张成 benchmark、application 或 dataset paper.
- Risk: MEDIUM
- Estimated effort:
  - Compute: 60-100 GPU-hours for real FEX controller sweeps with 3-5 seeds, plus small LLM/API cost; v3 proxy ran locally on RTX 4060 Ti.
  - Data: available; analytic PDE metadata and generated collocation handoff are in `workspace/fex-llm-skeleton/data/MANIFEST.md`, with no annotation or external model download needed at idea stage.
  - Implementation: 2-4 weeks for controller logit/KL bias, Bhatnagar-style operator-set prior, and real FEX sweeps.
- Novelty quick-check: FunctionEvolve, Deliberate Evolution, SR-LLM, STRIDE, KeplerAgent, and LLM-SR already cover "LLM prior + symbolic search" in data-driven equation discovery, so this paper should not claim novelty for LLM skeleton generation. SymPlex and SSDE compete in symbolic PDE solving. The remaining gap is narrower but real: for FEX-style PDE residual solving, no prior work maps how much symbolic structure a controller should trust, how hard that prior should be, and when full skeletons add value beyond operator-set or domain-aware statistical priors.
- Strongest objection: v3's new pilot still uses a finite candidate library where the correct skeleton is placed at rank 2; it tests recoverability of a soft prior, not real FEX controller acceleration or real LLM top-k quality.
- Why we should do this: The result answers a concrete design question for FEX: should outside knowledge enter as allowed operators, reusable subtrees, full skeletons, or a soft controller bias? The null result is also useful if it shows operator-set priors already capture most of the value.
- Review response decisions: I accept the v2 criticism that the soft-prior mechanism was underspecified, so v3 defines the mechanism as top-k mixture restart or controller logit/KL bias with an entropy floor and explicit fallback support. I accept the "corrupted skeleton" criticism, so corruption is now local and operational: spurious linear/cross/sine terms or wrong sine arguments. I accept the evidence criticism: the new pilot is only controller-facing proxy evidence, and the decisive next gate is a real FEX controller sweep against from-scratch and Bhatnagar-style operator-set restriction. I accept the missing baseline criticism by adding Domain-Aware Symbolic Priors, SAGE-Fit-style coefficient fitting, SymPlex/SSDE, and FunctionEvolve/STRIDE as required comparisons or scope boundaries. I push back on treating LLM-SR/KeplerAgent as direct collisions: they solve data-driven equation discovery, while this idea studies closed-form PDE solution search under FEX residual/boundary losses and asks a prior-granularity question rather than claiming a new LLM-SR paradigm.
- Pilot:
  - Setup: On local CUDA, `soft_prior_controller_pilot.py` used the three analytic cases from v2: Poisson quadratic d=5, rotated quadratic d=5, and sine-interaction d=4. Each case used a finite library scored by FEX-style PDE residual + boundary loss. The simulated LLM ranking deliberately put a corrupted skeleton at rank 1 and the correct full skeleton at rank 2. Policies were hard corrupted top-1, soft top-k with support, oracle full skeleton, and unranked operator-set uniform proxy over the same finite support.
  - Metric: rel-L2 < 1e-3 counts as success. A positive proxy signal requires hard top-1 to fail, soft top-k to recover in <=3 evaluations, and unranked operator-set search to need more evaluations or fail more often under the same 8-eval budget.
  - Result: CUDA wall time was 9.5s. Hard corrupted top-1 failed on all three cases, with rel-L2 1.12e-2 to 1.30e-2. Soft top-k recovered in 2 evaluations on all three cases, with best rel-L2 3.01e-7, 3.48e-7, and 1.05e-7. Oracle succeeded in 1 evaluation. Unranked operator-set uniform search succeeded 5/5 on Poisson with median 5 evaluations, but only 1/5 on rotated quadratic and 1/5 on sine interaction within the 8-eval budget.
  - Signal: POSITIVE but bounded; it supports the recoverability mechanism, not yet the real FEX controller speedup claim.

- Claims and Claims matrix: Main claim: FEX should use symbolic priors as recoverable controller bias, not as hard templates. POSITIVE: in real FEX, soft top-k skeleton prior beats from-scratch and operator-set prior by >=2x candidate evaluations or wall-clock while staying robust to local corruption. NULL-A: operator-set or domain-aware statistical prior matches full skeleton, so FEX only needs coarse priors and the diagnostic phase map is the contribution. NULL-B: full skeleton helps direct fitting but not controller search, so parameter fitting is not the bottleneck and controller injection must be redesigned. NEGATIVE: corrupted priors drag soft search below operator-set prior, so LLM skeletons are too brittle for FEX unless repaired before injection.
- Narrative: The paper should be about calibrating trust in scientific symbolic search. The question is not whether an LLM can guess a textbook PDE solution. It is how much structure FEX can borrow before the prior stops helping and starts trapping the search.
- Experiments: Gate 0 real FEX sweep: from-scratch FEX, Bhatnagar-style operator-set FEX, hard top-1 skeleton, soft logit/KL bias, top-k mixture restart, grammar mask, Domain-Aware statistical prior, and oracle prior on Poisson, conservation law, Schrodinger, rotated/nonseparable, and anti-memorization PDEs. Gate 1 prior-source comparison: LLM top-k, n-gram/Domain-Aware prior, KeplerAgent-style physics-tool prior, random prior, and oracle. Gate 2 fitting fairness: standard Adam+BFGS vs SAGE-Fit-style structure-aware coefficient fitting so "good structure, bad score" does not decide the conclusion. Gate 3 boundary comparisons: SymPlex/SSDE as symbolic PDE solver baselines and FunctionEvolve/STRIDE as general SR boundary comparisons, reported without claiming they instantiate FEX.
- Assets status: v3 CUDA proxy, v2 granularity pilot, analytic metadata, and handoff notes are ready locally; data/model status is recorded in `workspace/fex-llm-skeleton/data/MANIFEST.md`.

<review date="2026-06-16">

## PROSE 系列补全 addendum [deep-lit-tick, 2026-06-24]

本轮 deep-lit-tick 系统性补录了 Hayden Schaeffer (UCLA) 的 PROSE 系列 9 篇论文，此前 landscape 仅一句话提及。核心发现：

### 最关键的 prior: PROSE-PDE Skeleton 模式 (2404.12355)

PROSE-PDE 在 20 族 1D PDE 上训练，设置了一个 "Skeleton" 模式：仅提供方程骨架结构（算子和项的拓扑），不提供系数。结果 Skeleton 模式数据误差 1.06%，与 Known 模式（完整方程已知）的 0.92% 仅差 0.14 个百分点。这直接验证了本 idea 的核心假设——仅符号骨架（不含系数）即可大幅指导求解。

**但关键区别**：PROSE-PDE 用 Skeleton 做 Transformer 算子学习（数值解预测），FEX 用骨架做 RL 搜索解析解。两者的搜索目标不同（数值 vs 解析），所以 PROSE-PDE Skeleton 模式不直接抢本 idea 的 novelty，但它是必须引用和详细区分的 prior。

### PI-MFM 的 zero-shot physics fine-tuning (2512.23056)

PI-MFM 展示了从 physics-informed 预训练模型出发，仅用 PDE 残差和 IC/BC（无标注解数据）即可适配到未见 PDE。这为 "预训练模型 → FEX" 整合提供了最直接的实验范本。PI-MFM 的 PDE 残差自动计算机制（从 Polish notation 解析表达式树→向量化计算残差）与 FEX 的 RL 奖励函数异曲同工。

### 对 fex-llm-skeleton 实验设计的影响

1. Gate 0 必须加入 PROSE-PDE Skeleton 模式作为 baseline 或详细 differentiate
2. 本 idea 的 "先验粒度相图" 应将 PROSE Skeleton 作为 "full skeleton + Transformer 数值求解" 端点，与 FEX 的 "soft prior + RL 解析求解" 做系统比较
3. PI-MFM 的自动 PDE 残差计算可作为 FEX controller 奖励信号的替代实现参考

### 补录论文列表

| arxiv_id | 标题 | 与本 idea 的关系 |
|----------|------|----------------|
| 2309.16816 | PROSE (original) | 符号编码方式（Polish notation）与 FEX 二叉树同构 |
| 2404.12355 | PROSE-PDE | **Skeleton 模式是最直接的 prior** |
| 2409.09811 | PROSE-FD | 2D 扩展，展示 scale-up 能力 |
| 2502.06026 | PROSE + Text | 文本替代符号的多模态方向 |
| 2512.23056 | PI-MFM | zero-shot physics fine-tuning，残差自动计算 |
| 2408.16168 | LeMON | meta-learning 跨 PDE 族 warm-start |
| 2409.11609 | PROSE + SymPy | SymPy 标准化 + 粒子滤波系数精炼 |
| 2501.18972 | BCAT | block causal transformer，放弃符号模态 |
| 2510.25379 | MNO/MONet | 多算子学习理论框架，scaling law |

## Novelty

- Score: 6/10
- Closest prior work: Bhatnagar et al. (2503.09986, LLM+FEX operator-set prediction); FunctionEvolve (2606.07704, NeurIPS 2026, LLM+AST-guided evolution for SR); SymPlex (2602.03816, ICML 2026, structure-aware Transformer+RL for symbolic PDE solving); SSDE (2405.14620, ICML 2025, RL symbolic closed-form PDE/ODE solver); LLM-SR (Shojaee et al. 2024, LLM skeleton for data-driven SR); SR-LLM (Guo et al. 2025, PNAS, LLM+RAG skeleton+DRL assembly); DecAEvolve (NeurIPS 2025/ICML 2026, symbolic term decomposition+RL fine-tuning)
- Key differentiator: (1) 系统比较 FEX PDE 求解中 prior 粒度的相图 (operator-set / 统计 prior / subtree / full skeleton / oracle / corrupted) — 无现有工作做此系统性对照; (2) soft recoverable prior 注入机制 (top-k mixture restart / controller logit-KL bias with entropy floor + fallback support) 而非硬模板; (3) 面向 PDE 残差/边界求解而非数据驱动方程发现. 但核心概念 "LLM prior + 符号搜索" 已被大量工作充分覆盖; novelty 主要来自诊断设计和 FEX-specific 注入机制, 非全新方法范式.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v3 逻辑链清晰且显著改进: FEX controller 瓶颈 → 需要结构先验 → 错误先验有害 → 软可恢复 prior 可解决. 但仍有 gap: (1) 有限库 proxy 中正确答案 guarantee 在 top-k, 真实 FEX controller 无此保证, 从 proxy 到 controller 的推广仅是假设; (2) soft logit/KL bias 在 FEX policy-gradient controller 上的具体实现 (如何 bias action distribution 而不破坏 PG, entropy floor 如何 calibrate) 仍抽象. |
| Missing evidence signals | 6/10 | v3 pilot 比 v2 有实质性提升 (finite-library soft-prior recoverability 已测试), 但关键证据仍缺失: (1) 真实 FEX controller sweep 结果 (Gate 0), (2) LLM top-k 骨架在 anti-memorization PDE 上的真实质量 (当前为模拟 LLM ranking), (3) multi-seed 稳定性, (4) Bhatnagar-style operator-set 强 baseline 对照, (5) Domain-Aware 统计 prior 对照, (6) SAGE-Fit 风格参数拟合公平性尚未测试, (7) SymPlex/SSDE 边界比较未做. |
| Narrative | 7/10 | "信任校准" (calibrating trust in scientific symbolic search) 叙事清晰有力. v3 强调 "问题不在于 LLM 能否猜出教科书 PDE 解, 而在于 FEX 能借多少结构才不至于 prior 从资产变负债" — 这是 genuine research question. 但 idea 仍偏 Yang-group specific, 未充分采用 v2 alternative framing 建议 (将 FEX 作为 "可恢复符号先验在科学搜索中的粒度规律" 的实例化平台, 而非全部贡献范围). |
| Venue contribution | 6/10 | 对 NeurIPS/ICML: 若 real controller sweep 揭示反直觉 phase map (如 operator-set 在多数 PDE 族上已足够, 或 soft prior 在特定 corruption regime 有相变式优势), 有顶会潜力. 但当前证据仅 support 诊断论文; 方法贡献 (soft prior injection) 是增量式的. 对 JCP/SISC 等计算物理 venue: 贡献更充分. SymPlex (ICML 2026) 和 SSDE (ICML 2025) 已占据 "符号 PDE 求解" 空间, 需明确 differentiate. |
| Testability | 8/10 | v3 的 falsification 条件具体可测: "真实 controller 中 operator-set prior 已匹配 skeleton prior" (廉价), "soft prior 只在有限库 proxy 中有效而不能改善 FEX policy-gradient 搜索" (稍贵但明确). POSITIVE/NULL/NEGATIVE 信号均有清晰 metric 和阈值. Anti-memorization PDE 的纳入增强可信度. Pilot 廉价可复现 (9.5s). |
| Outcome realism | 6/10 | 60-100 GPU-hours 和 2-4 周实施周期合理. 但预期 speedup (>=2x over from-scratch 和 operator-set prior) 面临风险: (1) Bhatnagar-style operator-set 限制已大幅缩减搜索空间, 全骨架的边际增益可能很小; (2) LLM inference 成本和多候选探索 overhead 侵蚀 wall-clock 收益; (3) pilot 的 simulated LLM ranking (正确答案在 rank 2) 可能过于乐观 — 真实 LLM top-k 在 anti-memorization PDE 上质量未知. |
| Contribution type compliance | n.a. | topic 未声明 preferred-contribution-types, 跳过检查. |
| Overall Quality | 7/10 | v3 相比 v2 是实质性改进: soft prior 机制已 define (logit/KL bias, mixture restart, entropy floor, fallback support), corruption 已操作化, 实验设计已分层 (Gate 0/1/2/3), pilot 从直接骨架拟合升级为 controller proxy. 当前状态是 well-formed diagnostic+method idea, 有明确实验路径. 顶会发表仍需 real controller-level 的反直觉发现. |

## Prior Review Follow-up (v2 → v3)

- v2 concern: "soft prior 机制 underspecified": **partially resolved** — v3 定义了 logit/KL bias, mixture restart, entropy floor, fallback support, 但具体实现和 calibration 仍抽象.
- v2 concern: "corrupted skeleton 定义不操作化": **resolved** — v3 定义 local corruption: spurious linear/cross/sine terms 或 wrong sine arguments.
- v2 concern: "pilot 只测了直接骨架拟合, 非 controller 加速": **partially resolved** — v3 pilot 升级为 finite-library soft-prior controller proxy, 测试了 recoverability, 但仍非真实 FEX controller sweep.
- v2 concern: "缺少强 baseline 对照": **partially resolved** — v3 列出了 Bhatnagar-style, Domain-Aware, SAGE-Fit, SymPlex/SSDE, FunctionEvolve/STRIDE 为 planned comparisons, 但多数未执行.
- v2 concern: "LLM-SR/KeplerAgent 碰撞": refiner pushback **accepted** — PDE residual/FEX 设定确实不同, pushback 有据.
- v2 suggestion: "SAGE-Fit / SymPlex 对照未纳入实验": **resolved** — v3 Gate 2/3 显式列出.
- v2 suggestion: "Alternative framing: FEX 作为实例化平台, 而非全部贡献范围": **partially adopted** — v3 narrative 走向正确方向但未 fully embrace 此 framing; 本 review 重新强调.

## Contribution Drift

- v_{n-1} contribution types: {method, diagnostic}
- v_n contribution types: {method, diagnostic}
- Status: unchanged (v3 显式声明: "v2 contribution type 是 `method + diagnostic`; v3 仍是 `method + diagnostic`. 新增 controller proxy 和更强 baseline 要求只是收紧同一贡献")
- Hard cap triggered: no (topic 未声明 preferred-contribution-types, 跳过所有 hard cap 检查)

## Alternative Framing

当前 framing 已接近最优, 但可更锋利:

**Recoverable symbolic priors for PDE residual search: when should scientific search trust operators, subtrees, full skeletons, or only a soft bias?**

此 framing 将 FEX 作为 "可恢复符号先验在科学搜索中粒度规律" 的实例化平台, 核心产出为 prior 粒度 × 注入硬度 × PDE 族 × corruption level 的 phase map. 这使得 idea 更不易被 "LLM-SR already generates skeletons" 或 "SymPlex already solves PDEs symbolically" 轻易毙掉, 也更容易吸引 FEX 之外的读者.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在限定 FEX PDE benchmark 和 anti-memorization 设置下, soft top-k skeleton prior (controller logit/KL bias / mixture restart with entropy floor) 相比 from-scratch FEX 和 Bhatnagar-style operator-set prior 将 median candidate evaluations 或 wall-clock 降到 ≤ 1/2, 且局部 corrupted top-1 skeleton 下不比 operator-set prior 差超过 10%. **不能声称** LLM skeleton 普遍优于其他 prior, 或结论推广到所有 SR 方法. |
| NULL-A | operator-set 或 Domain-Aware 统计 prior 已匹配完整 skeleton: 只能声称 "在测试 PDE 族和 FEX 框架下, 全骨架先验未提供超越粗粒度算子和统计先验的额外搜索效率增益; FEX 只需粗先验, 诊断 phase map 为主要贡献". |
| NULL-B | 全骨架帮助参数拟合但不帮助 controller 搜索: 只能声称 "参数拟合非瓶颈, controller injection 需重新设计". **不能声称** LLM 科学先验对 PDE 求解无价值. |
| NEGATIVE | corrupted priors 拖累 soft search 至 operator-set prior 以下: 可声称 "LLM 骨架对 FEX 过于脆弱, 需修复或降级为 operator-set 级 prior 才可安全使用". **不能声称** LLM 不适合 PDE 求解 — 可能只是注入机制需改进. |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood (Medium): 有明确成稿路径 (Gate 0 → Gate 1 → Gate 2 → Gate 3 实验设计清晰). Yang 组有 FEX 基础设施和 Bhatnagar et al. 作为直接前驱; v3 pilot 已初步验证 soft prior 在有限库 proxy 下的 recoverability. 但做成 top-venue 级结果依赖若干条件: (1) 真实 FEX controller sweep 显示 soft prior 对强 operator-set baseline 有超越性加速, (2) LLM top-k 骨架在 anti-memorization PDE 上保持质量, (3) phase map 揭示反直觉规律而非仅确认直觉. 若任一不成立, 成稿仍是可能的 (诊断论文), 但顶会概率显著下降.
  - Impact (Medium): 若成功 (揭示清晰的 prior 粒度相图 + soft prior 系统性优势), 会对 FEX 设计空间和 LLM-guided symbolic PDE solving 产生清楚影响, 并为 "科学搜索中 LLM 先验校准" 提供实证规律. 但不太可能改变整个 SR 或 LLM-SR 领域判断 — FEX 本身是 niche 方法, SymPlex/FunctionEvolve/DecAEvolve 等竞品在各路线快速推进.
  - Claude 与 codex 无分歧: 双方均给 Likelihood=Medium, Impact=Medium. 无需在 Comments 中标注.

## Overall

- Priority: Medium
- Score: 5
- Comments: v3 是 v2 的实质性改进 — soft prior 机制已 define, corruption 已操作化, 实验已分层 (Gate 0/1/2/3), pilot 已升级为 controller proxy. 核心 novelty 是 recoverable prior granularity 的系统研究, 而非 LLM skeleton generation 本身. 决定性的下一步是真实 FEX controller sweep (Gate 0) 对照 Bhatnagar-style operator-set, Domain-Aware 统计 prior, hard skeleton, soft skeleton, oracle, 和 corrupted prior. 在此证据出现前, 本 idea 是一个 well-framed diagnostic+method idea, 有明确实验路径但尚未达到 top-venue 所需的反直觉发现门槛. 建议下轮 refine 前先跑 Gate 0 获取 controller-level signal.

</review>
