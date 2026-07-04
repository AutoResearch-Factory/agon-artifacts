---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-search-complexity/
---

- One-sentence summary: 建立 FEX 表达式树搜索的分层复杂度理论, 把离散结构搜索、等价类压缩和实值系数优化分开, 给出受限函数类上的多项式搜索条件与一般 FEX 可行性问题的硬度边界.
- Hypothesis: FEX 的维度难点并不全在离散树搜索本身: 对低有效 FEX 覆盖数的 Barron-like、有限 Fourier 或可分离 PDE 解类, 熵正则 softmax controller 的期望 reward 可能满足带多项式常数的 gradient-domination 条件, 因而结构搜索步数可控; 真正随维度恶化的部分更可能是实值系数优化和 PDE residual 评分.
- Expected outcome: 成功版本包含三层结果: (i) FEX 搜索空间在树深、算子字典和等价类商空间下的计数上下界; (ii) 对一个明确受限的函数类证明 "FEX 覆盖数 + policy-gradient gradient-domination => poly(d, 1/eps) 结构搜索步数"; (iii) 对 full real-parameter FEX residual-feasibility 证明 ∃R-hardness, 或在更弱限制下证明退化 FEX 的 NP-hardness. 不再主张 QBF/PSPACE, 除非能构造真正的量词交替结构.
- Contribution type: theory
- Risk: HIGH
- Estimated effort:
  - Compute: 20-40 GPU-hours for diagnostic sweeps; main theorem work is 0 GPU-hours
  - Data: available
  - Implementation: weeks-months
- Novelty quick-check: Liang & Yang (2206.10121) 证明 FEX 函数空间有避免维度灾难的 approximation guarantee, 但没有分析 RL controller 找到表达式的 step/sample complexity. Virgolin & Pissis (2207.01018) 和 Song et al. (2404.13820) 证明的是 data-fit SR 的 NP-hardness, 不是 PDE residual FEX, 也不处理 real-parameter inner optimization; Mei et al. (2005.06392) 与 Agarwal et al. (1908.00261) 给出 policy-gradient 理论工具, 但没有 expression-tree/FEX 覆盖引理. 近期 VSR-DPG、CADSR、EGG-SR 和 VaSST 改善 SR 搜索效率或连续松弛, 仍未给出 FEX/PDE controller 的复杂度理论.
- Strongest objection: 正面定理可能只能覆盖很窄的函数类, 而负面 ∃R-hardness 若做得太一般, 可能被认为只是把实值非凸可行性搬到 FEX 上.
- Why we should do this: FEX 已有 approximation theory, Yang 脉络里也已有 NN optimization CoD lower bound, 但 FEX 的 RL 搜索复杂度仍是空白. 把搜索、等价类和系数优化拆开后, 这个问题不再是一个模糊的 "RL 会不会找到答案", 也能解释 pilot 中 "结构搜索不慢但高精度优化变难" 的现象.
- Pilot:
  - Setup: patched FEX Poisson controller on `depth2_sub`, dimensions 2/5/10/20/30, three seeds, reward hit thresholds 0.90/0.95/0.99; CUDA sanity run verified the output path fix
  - Metric: if log(hit_time) vs d has slope > 0.5 with R^2 > 0.5, treat as exponential-search evidence; if loose thresholds stay reachable at high d while 0.99 fails, treat as evidence that coefficient optimization is the bottleneck
  - Result: 13/15 runs hit reward 0.95; 11/15 hit reward 0.99, with no d=30 seed reaching 0.99. The primary 0.99 fit has slope 0.067 and R^2 0.14, so hit time is flat/noisy rather than exponential.
  - Signal: WEAK

- Claims and Claims matrix: Claim A: raw FEX tree count is a misleading complexity measure unless quotienting by operator identities and template equivalence is made explicit; supported by a counting theorem alone. Claim B: for a restricted low-cover class, FEX controller structure search is polynomial under two new lemmas, a FEX covering lemma and a softmax-PG gradient-domination lemma with polynomial constants; if either lemma remains conditional, the paper must state a conditional theorem rather than claim unconditional convergence. Claim C: full FEX residual-feasibility with real coefficients is at least as hard as a real-algebraic feasibility problem; if ∃R-hardness fails, the negative claim retreats to NP-hardness for a clearly degenerate integer/affine FEX restriction. Claim D: empirical pilot evidence only supports search/optimization separation on one Poisson family, not a universal scaling law.
- Narrative: FEX 已经解决了高维符号 PDE 求解的 approximation 问题, 但还没有回答 controller 能否找到那个表达式. 这篇文章应该把这个空白讲清楚: 小 FEX cover 可以让结构搜索变得温和, 不受限制的实值参数会把困难转移到连续优化.
- Experiments: Use diagnostics only, not a benchmark contribution: repeat Poisson sweeps with multiple thresholds; add finite Fourier and separable/nonseparable PDE families; ablate tree depth, entropy, equality-graph quotienting, and inner optimizer budget; report hit-time distributions separately for structure threshold and high-accuracy residual threshold.
- Assets status: Pilot code, generated collocation data notes, regenerated analysis, and CUDA sanity status are recorded in `workspace/fex-search-complexity/data/MANIFEST.md`.

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026); Virgolin & Pissis, *Symbolic Regression is NP-hard* (TMLR 2022); Song et al., *Prove Symbolic Regression is NP-hard by Symbol Graph* (2024); PGTS 2506.07054
- Key differentiator: 无人对 FEX 的 RL-controller-on-expression-trees 建立过 policy-gradient 收敛率或针对 PDE residual+FEX 实数参数化的 hardness 归约. Soubki & Cranmer (ICML 2026) 通过 parameterized complexity (FPT/W-hierarchy) 分析了 SR 的 tractability, 部分抢占 Claim A (搜索空间计数/复杂度假说), 但其分析对象是 data-fit SR 且使用 FPT 框架, 不涉及 FEX 的 RL 收敛率、PDE residual 搜索或 ∃R-hardness. PGTS (2506.07054) 明确将 "establishing convergence rates" 列为 open future work. 本 idea 的正面 PG 收敛方向 (Claim B) novelty 仍 HIGH, 但整体 novelty 因 Soubki & Cranmer 的 partial preemption 而从前版 7 降至 6.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | v2 修掉了两处大硬伤 (Uniform-LGI citation mismatch 与 QBF/PSPACE overclaim), 但核心逻辑链仍有显著缺口: (a) "FEX 覆盖数" 的覆盖对象未定义 (是 function-class 的 ε-覆盖? controller policy 空间的覆盖? 表达式等价类商空间的覆盖?); (b) 从 "覆盖数小" 到 "gradient domination" 的推导路径缺失——前者是离散/组合性质, 后者是连续优化 landscape 性质, 两者间的桥梁 (如连续松弛、softmax 参数化如何继承覆盖结构) 完全没有草图; (c) ∃R-hardness 无任何 reduction 构造或 gadget 线索. 这些不是 minor gaps, 是 proof strategy 的骨架缺失. |
| Missing evidence signals | 6/10 | v2 有实质进步: pilot 已跑、三层 framing 已形成、引用已修正. 但仍缺: (i) covering object 的准确定义与最小非平凡计算 (如对 depth=1, operator=3 的小实例给出精确计数/商空间 size); (ii) gradient-domination 常数的维度依赖分析 (对 toy problem 的 numerical check); (iii) ∃R-hardness 的 candidate complete problem (如 ETR-INV, STRICT-INEQUALITY 等) 及其到 FEX 的大致映射. |
| Narrative | 8/10 | "approximation 已解决, controller 能否找到表达式未知; 困难可能从结构搜索转移到系数优化" 的叙事简洁有力. 三层分解 (计数→正面向导→负面 hardness) 使 idea 即使正面定理做不出也有退路. 但需显式承认 Soubki & Cranmer (ICML 2026) 并区分. |
| Venue contribution | 6/10 | 从 NeurIPS/ICML/ICLR theory track 视角: 若三个 layer 全部兑现, 是 transformative paper. 但 realistic 版本是 counting lemma (Layer 1) + ∃R-hardness (Layer 3), 这是 solid theory 贡献, 可投理论会议或 TMLR. 若只有 counting lemma (且需与 Soubki & Cranmer 区分), 贡献降为中等. |
| Testability | 7/10 | Pilot (Poisson d=2..30 hit-time vs d slope=0.067, R^2=0.14) 是合理的 early falsification signal——排除了 "结构搜索指数爆炸" 作为唯一维度假说. 若扩展到 finite Fourier 和 nonseparable PDE families 并测试 entropy 消融 (验证 gradient-domination 机制猜想), 将更强. falsifier 在 Pilot 段而非 Expected outcome 段, 但可接受. |
| Outcome realism | 5/10 | "weeks-months" 对 full three-layer paper 仍严重低估. Layer 1 (combinatorial counting) 几周可行; Layer 3 (∃R-hardness) 数月; Layer 2 (PG convergence + covering lemma) 是 PhD-level 工作, 每条 lemma 可能独立成为 paper. 三层全部兑现需 1-3 年. |
| Contribution type compliance | n.a. | idea types = {theory}; topic 未声明 `preferred-contribution-types`, 本项跳过, 不计入 Overall Quality. |
| Overall Quality | 6/10 | v2 完成了核心修复 (citation、framing、claims boundary), 从一个 "方向对但工具全错" 的 idea 变成了 "方向对、工具对但 proof bridge 仍需构建" 的 idea. 主要风险从 narrative/引用错误转为 theorem 可行性——这是正确的风险转移方向. |

## Contribution Drift

- v_{n-1} contribution types: {theory}
- v_n contribution types: {theory}
- Status: unchanged
- Hard cap triggered: no

## Prior Review Resolution (v1 concerns)

| v1 Concern | Status | Notes |
|------------|--------|-------|
| Citation mismatch: 2202.10670 Uniform-LGI 误用作 RL 收敛工具 | **Resolved** | 已替换为 Mei 2020 (2005.06392) + Agarwal 2021 (1908.00261) softmax-PG canon, 并显式列出 covering + gradient-domination 两条待证 lemma |
| QBF/PSPACE overclaim 与 FEX ∃-结构不匹配 | **Resolved** | 已改为 ∃R-hardness 或退化 NP-hardness, 并承认 "除非能构造真正的量词交替结构" |
| 三层 framing 建议 | **Resolved** | 已采纳: Claim A (counting) + Claim B (positive PG) + Claim C (negative hardness) + Claim D (empirical support) |
| Pilot 未跑 | **Resolved** | 已跑 15 runs: 13/15 达 0.95, 11/15 达 0.99, d=30 无种子到 0.99; slope=0.067, R^2=0.14 |
| Deep-lit on OpenXOR 2511.11712 | **Partially resolved** | v2 body 提及 OpenXOR 但未确认全文已读; novelty quick-check 节承认 "是强相关引用与 negative 分支的部分先例" |
| Effort estimate 过于乐观 | **Ignored** | v2 仍写 "weeks-months", 对本轮 reviewer 强调的 "每条 lemma 可能是独立 paper 级" 未作回应 |
| EGG-SR 等新发现文献的整合 | **Partially resolved** | deep-lit-update 段收录了 EGG-SR/GENSR/SAGE-Fit, 但 body 未说明 EGG-SR 等价类剪枝如何影响商空间 counting |

## Alternative Framing

当前三层 framing 已是 v1 review 建议的产物, 结构合理. 但存在一个更尖锐的 framing: **把核心 contribution 从 "FEX 搜索复杂度边界" 重新定位为 "FEX 搜索的 complexity decomposition theorem"** ——即不承诺证明 controller 在广泛类上 poly 收敛, 而是证明 "复杂度 = 离散结构覆盖数 (Lemma A) × 连续 PG landscape 难易度 (Lemma B) × 实值可行性难度 (Lemma C)", 三者各自独立且可分别击破. 这个 framing 让 paper 即使 Lemma B (PG convergence) 只证到条件定理, 整体仍是完整的 decomposition 贡献.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | "在显式定义的 FEX-K-cover 函数类 ∩ 已证 poly(d) gradient-domination 条件下, 理想化 softmax-PG controller 在 poly(d, 1/ε) 步内以高概率命中 ε-好表达式结构." 不可声称 unconditionally 适用于所有 PDE 或含 BFGS noisy reward 的实际 FEX 实现. |
| NULL | 若 covering 或 PG lemma 只能条件化: 可声称 "FEX 搜索复杂度下界由商空间等价类数 Ω(Q) 给出, 上界在 gradient-domination 条件下为 poly(Q, 1/ε)." 不可声称已证明无条件 polynomial convergence. |
| NEGATIVE | 若证出 ∃R-hardness: 可声称 "FEX residual-feasibility with real coefficients is ∃R-hard." 若只证退化 NP-hardness: 必须限定为 "integer/affine-restricted degenerate FEX search is NP-hard" 并显式区分 Virgolin (2207.01018) / Song (2404.13820) / Soubki & Cranmer (ICML 2026). 不可暗示 full FEX RL search 必然难. |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  - **Likelihood: Medium** —— v2 有三条清晰退路 (counting lemma / ∃R-hardness / 条件定理), 至少一条可成 top-venue 级理论结果. counting lemma (Layer 1) 几周-3月可成; ∃R-hardness 数月; full PG convergence (Layer 2) 是最高风险项但非必须. 整体有明确成稿路径, 依赖 Layer 1+3 成立. 没有无条件 guarantee 但也不是纯投机.
  - **Impact: High** —— 若正面 PG convergence 定理成立, 将 fundamentally change 社区对 expression-tree RL 搜索的理解: "维度灾难不在离散搜索, 而在实值优化." 即使只有 counting + ∃R-hardness, 也是 FEX/Yang 脉络缺失的 "search complexity" 拼图, 会被大量后续工作引用. Soubki & Cranmer (ICML 2026) 的存在反而增强了这个方向的 relevance 和 urgency.
  - Codex 独立判断给出 Likelihood=Medium, Impact=High, score=7, 与本评一致.

## Overall

- Priority: High
- Score: 7
- Comments: v2 是实质性的好修复, 已从 v1 的 "方向对但工具全错" 进化到 "方向对、工具对、proof bridge 待建." 但 reviewer 发现了一个威胁: Soubki & Cranmer (ICML 2026) 通过 parameterized complexity 大幅 close 了 "SR 搜索复杂性" 这个 gap, 虽不针对 FEX/PDE, 但仍部分抢占本 idea 的核心 novelty 叙事. v3 必须 (1) 显式引用并区分 Soubki & Cranmer; (2) 写出 covering object 的准确定义; (3) 为 ∃R-hardness 提供 candidate complete problem 和至少一个 gadget 草图; (4) 修正 effort estimate 为 "Layer 1: weeks; Layer 3: months; Layer 2: 6-18 months, conditional on Lemma A+B." 代码和收益比仍优秀, 可进入下一轮 refine. Claude 与 codex 对 Likelihood (均 Medium) 和 Impact (均 High) 无分歧.

</review>

## Deep Lit 2026-06-16 Update

本次 deep-lit 续扫发现以下与本 idea 直接相关的新文献:

- **DSO (2505.10762)** [wiki](wiki/2505.10762.md): LLNL DSO 框架综述。RSPG (risk-seeking PG) 是 FEX PG 的最直接竞品——FEX 用 vanilla PG + Adam/BFGS, DSO 用 RSPG + GP seeding + in-situ constraints。若本 idea 要做 FEX PG convergence analysis，DSO 的 RSPG formulation 是必须对比的分析对象。DSO 的 Good-Structure-Bad-Score 现象也提供了 concrete 反例场景。
- **Exhaustive Symbolic Regression (2507.13033)** [wiki](wiki/2507.13033.md): 穷举搜索 + MDL 模型选择。在给定复杂度上限内穷举所有表达式树，为 FEX controller 提供 oracle benchmark——在某复杂度上限内 RL 是否找到最优？对 counting lemma (Layer 1) 提供可计算 ground truth。
- **eggp (2501.17848)** [wiki](wiki/2501.17848.md): GP 中嵌入 e-graph 剪枝等价表达式。与 EGG-SR (2511.05849) 互补（GP side vs MCTS/DRL/LLM side）。等价剪枝效果进一步支持"search space counting 是 SR complexity 核心"的叙事。
- **Data-driven discovery of governing DEs review (2606.09638)** [wiki](wiki/2606.09638.md): REO 抽象框架 + solvability metric。为本 idea 的 claims 提供 community-facing 术语体系来定位 FEX search complexity 贡献。

## Deep Lit 2026-06-16 Update (R2 — idea-specific for fex-search-complexity)

以下 8 篇由 `deep-lit-tick --scope idea 0616-fex --idea fex-search-complexity` 于 2026-06-16 第二轮系统性搜索并精读。本轮聚焦 idea 的三个核心 claim（Claim A 计数、Claim B PG convergence、Claim C ∃R-hardness），覆盖六轴。

### Claim A & B: PG Convergence Theory for Expression-Tree Search

- **2605.24939** [wiki](wiki/2605.24939.md): Ziyue Chen, David Šiška, Lukasz Szpruch (2026). *Global linear convergence of entropy-regularized softmax policy gradient beyond tabular MDPs*. 在连续状态/动作空间下证明 entropy-regularized softmax PG 的全局线性收敛。核心技术: non-uniform Polyak-Łojasiewicz inequality + KL radial unboundedness + Lyapunov 控制。**对 Claim B 的价值**: 提供了 PG convergence 的最先进通用工具链（PŁ + KL unboundedness），但这些工具尚未特化到 FEX 的离散表达式树 action space。若要将 FEX controller 的 convergence 归约为 PŁ 条件，需在表达式树策略空间上建立 covering lemma。

- **2601.12604** [wiki](wiki/2601.12604.md): Safwan Labbi, Daniil Tiapkin, Paul Mangold, Éric Moulines (2026). *Beyond Softmax and Entropy: Convergence Rates of Policy Gradients with f-SoftArgmax Parameterization & Coupled Regularization*. 提出 f-softargmax 替代 softmax，耦合 f-divergence 正则化，证明 Tsallis divergence 下 f-PG 达到多项式 sample complexity（vs softmax 的指数复杂度）。**对 Claim B 的价值**: 首次证明通过更换策略参数化可从指数收敛跳到多项式收敛——直接支持 Claim B 的叙事"FEX 搜索复杂度取决于策略参数化和覆盖结构，而非内在指数难"。

- **2102.11270** [wiki](wiki/2102.11270.md): Gen Li, Yuting Wei, Yuejie Chi, Yuxin Chen (2021). *Softmax policy gradient methods can take exponential time to converge*. 构造 3-action MDP 证明 vanilla softmax PG 需要 (1/η)|S|^{2^{Ω(1/(1-γ))}} 次迭代收敛。**对 Claim B/C 的价值**: softmax PG 指数慢收敛的 definitive negative result，为 Claim B 的正面定理提供必须区分的 baseline。与 2601.12604 形成对偶: softmax → 指数慢, f-softargmax → 多项式快。

- **2209.13966** [wiki](wiki/2209.13966.md): Gal Dalal, Shie Mannor 等 (2022). *SoftTreeMax: Policy Gradient with Tree Search*. 将 tree search 直接并入 PG 策略参数化，报告 3 个数量级的梯度方差降低和最高 5x 性能提升。**对 Claim A/B 的价值**: 唯一将 tree search 和 PG 在梯度层面统一的工作——FEX controller 本质上是 tree-structured policy。其方差降低机制为 FEX covering lemma 提供了潜在的统计效率分析框架。

- **2503.06879** [wiki](wiki/2503.06879.md): Ding Lin, Jianhui Wang 等 (2025). *Reinforcement Learning-Based Symbolic Regression for Load Modeling*. 用 Actor-Critic + 二叉表达式树 + risk-seeking PG + 候选池微调做符号回归。**对 Claim A/B 的价值**: 与 FEX 方法学高度相似（表达式树 + RL controller），可作为 cross-domain 对照——对比两者的搜索效率为 FEX covering 分析提供 evidence。

- **2107.09158** [wiki](wiki/2107.09158.md): Mikel Landajuela, Brenden Petersen 等 (LLNL, 2021). *Improving exploration in policy gradient search: Application to symbolic optimization*. 诊断 DSR 的 early commitment 和 initialization bias 失败模式，提出 hierarchical entropy regularizer + soft length prior。**对 Claim A/B 的价值**: early commitment 为 FEX counting lemma 提供 failure-mode motivation；hierarchical entropy 是 Claim B "熵正则化改善 PG landscape"的 empirical instantiation。

### Claim C: ∃R-Completeness & Real-Algebraic Hardness

- **2605.23517** [wiki](wiki/2605.23517.md): Jack Stade (2026). *Probabilistically checkable proofs for the Existential Theory of the Reals*. 证明 ∃R 的 PCP 定理：MAX-ETR-INV 是 ∃R-hard to approximate within constant factor。核心技术: continuous assignment tester + midpoint code。**对 Claim C 的价值**: 为 ∃R-hardness of approximation 提供最新工具——若 FEX residual-feasibility 的 inapproximability 可归约到 MAX-ETR-INV，可直接引用该文。ETR-INV 约束形式 (x=1, xy=1, x+y=z) 与 FEX 算子字典存在表面相似性，可作为 reduction gadget 起点。

- **2407.18006** [wiki](wiki/2407.18006.md): Marcus Schaefer, Jean Cardinal, Tillmann Miltzow (2024). *The Existential Theory of the Reals as a Complexity Class: A Compendium*. ∃R 的 comprehensive survey。收录 ML 中的 ∃R-complete 问题（含 Training FCNNs is ∃R-Complete, 2204.01368）。**对 Claim C 的价值**: 提供完整 ∃R 工具图谱和已知 complete problems 清单，确认 ML-specific ∃R-complete 先例存在，为 FEX → ∃R reduction 提供范式参考。

### 对 idea 核心 claims 的影响总结

| Claim | 本次新证据 | 影响 |
|-------|----------|------|
| Claim A (counting) | 无直接竞争者；SoftTreeMax + 2107.09158 提供 adjacent motivation | 仍开放 |
| Claim B (PG poly convergence) | 2605.24939 (PŁ+KL) + 2601.12604 (f-softargmax→poly) + 2102.11270 (softmax→exp) 提供通用 PG 工具 | 工具链更清晰，但仍需 FEX covering lemma |
| Claim C (∃R-hardness) | 2605.23517 (∃R PCP) + 2407.18006 (compendium) + 2204.01368 (FCNN 先例) | 工具成熟度提升，需构造 FEX-specific reduction |
| Claim D (pilot) | 无外部研究做 FEX controller scaling | unique evidence 维持 |

**撞车风险评估**: 全部 4 个 claims 仍未被已有工作覆盖。最接近的 preemption 仍是 Soubki & Cranmer (ICML 2026) 通过 FPT/W-hierarchy 分析 SR tractability，但不针对 FEX 的 RL convergence、PDE residual 搜索或 ∃R-hardness。
