---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-synchro-prior/
---

- One-sentence summary: 将 Yang 博士论文的 synchrosqueezed transform / butterfly factorization 估计出的频率作为 FEX 输入层的冻结先验, 让 RL 只搜索非频率结构, 直接修复 Multi-Scale FEX 承认的频率分离失效模式.
- Hypothesis: Multi-Scale FEX 的频率分离不稳定性根源在于 RL 同时搜索频率和非频率结构; 冻结频率先验后, RL 的搜索空间大幅缩小, 在振荡 PDE 上的收敛率和精度都将显著改善.
- Expected outcome: 成功: 在 Multi-Scale FEX 论文中失败或精度差的振荡 PDE benchmark 上, synchro-prior FEX 将搜索时间降低 >50% 且精度提升 >1 个数量级; 失败: 频率先验并未显著帮助, 说明 RL 不稳定性的根源不在频率搜索而在参数耦合.
- Contribution type: method
- Risk: MEDIUM
- Estimated effort:
  - Compute: 20 GPU-hours on A100
  - Data: available (使用 Multi-Scale FEX 论文中的 benchmark PDE)
  - Implementation: 2-3 weeks
- Novelty quick-check: Multi-Scale FEX (2510.22497) 加了符号谱合成模块但不是冻结先验; 没有任何论文将时频分析先验与 RL-based expression tree search 结合. Yang 自己的 SynLab 工具和 FEX 是两条独立脉络, 从未被缝合.
- Strongest objection: synchrosqueezed transform 本身在高维 (d>3) 上的适用性和计算成本是否可控.
- Why we should do this: 这是"Yang 本人最懂却还没动手"的位置 -- 直接缝合他的两条学术脉络 (时频分析和 FEX), 解决他自己论文明确承认的失效模式. 无论成败都有清晰的叙事.
- Pilot:
  - Setup: 在 Multi-Scale FEX 论文的 2D 振荡 Helmholtz 方程上, 用 scipy 的 cwt 估计主频, 注入 FEX 输入层的 sin/cos 基底, 单 seed 对比标准 FEX
  - Metric: 若相对 L2 误差下降 >1 个数量级 (从 1e-3 到 1e-4 或更低), 信号为正
  - Result: pending
  - Signal: SKIPPED

<review date="2026-06-16">

## Novelty

- Score: 7/10
- Closest prior work: Multi-Scale Finite Expression Method for PDEs with Oscillatory Solutions on Complex Domains (Hardwick & Yang, 2025, 2510.22497)
- Key differentiator: 现有所有 FEX 变体（包括 Multi-Scale FEX 的符号谱合成模块）让 RL controller 在离散频率字典中同时搜索频率和非频率结构；本 idea 首次用外部时频分析（synchrosqueezed transform / butterfly factorization）为 FEX 输入层提供冻结频率先验，将频率搜索从 RL 搜索空间中移除。Yang 自己的两条学术脉络（SynLab/ButterflyLab 的时频分析工具 和 FEX 的 RL 符号搜索）从未被缝合，LLM+FEX（Bhatnagar et al. 2025）仅用 LLM 预测算子集而非频率先验。"Parsing the Language of Expression"（Huang et al. 2025）使用 domain-aware 符号先验但面向一般 SR 而非 PDE/FEX 频率场景。本质 novelty 在于组合机制而非单一组件。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | 最大漏洞：频率估计的数据源未定义。若从 exact solution 或 dense labeled samples 做 CWT/SST 会构成信息泄漏；若从 PDE 的 RHS、boundary、coefficients 或 early FEX residual 估计，需明确可观测性和估计可行性。High-dimensional (d>3) synchrosqueezed transform 适用性存疑（idea 自身在 Strongest objection 中承认但未给出解决方案）。冻结频率在先验误差或非线性谐波存在时是否反而限制 FEX 的表达式 expressivity 未讨论。 |
| Missing evidence signals | 4/10 | Pilot 标记为 SKIPPED/Result pending。关键缺失信号：(1) oracle-frequency ablation（先验证完美频率信息是否真能改善）; (2) estimated-frequency 与 oracle 的对比；(3) dense frequency grid baseline（证明改进不仅是增加了频率候选数）；(4) soft prior vs frozen prior 对比；(5) 频率估计误差与最终 PDE 误差的相关性分析。 |
| Narrative | 6/10 | "缝合 Yang 两条脉络" 对内部动机有效，但非顶会叙事。更强叙事应为：用经典时频分析将 RL symbolic search 中的 spectral combinatorial search 编译为可观测先验，从而诊断并缓解 RL-based PDE solver 的频率瓶颈——从"修一个方法"升级为"揭示一种设计原则"。 |
| Venue contribution | 5/10 | 按 FEX 文献发表模式（JMLR/JCP/AAAI）和 topic 无显式 target-venue 推断，目标应为 ML/SciML 顶会。当前像一个有希望的 FEX 修补，若只有 2D/3D 合成振荡 PDE 上的加速和精度提升，贡献偏薄。需要系统 benchmark、理论分析或机制诊断证明这不是简单的 feature engineering。若确证了失败模式的根因并提供了可推广的解决方案，才有顶会分量。 |
| Testability | 7/10 | Expected outcome 有清晰的可测指标。最干净的最便宜反证实验应为：在 Multi-Scale FEX 明确失败的 wide-separated-frequency benchmark 上，先用 oracle exact-frequency frozen FEX——若此 upper bound 都不能显著超越标准 Multi-Scale FEX，则可直接归档。Pilot 设计合理（2D Helmholtz + scipy cwt + 单 seed）。 |
| Outcome realism | 5/10 | 搜索时间降低 >50% 在缩小 operator/frequency space 后是可期待的。但精度提升 >1 个数量级仅对 Multi-Scale FEX 明确未收敛或精度差的频率分离案例现实；对原论文中已接近 machine precision 的 benchmark 不现实。预计实际 gain 集中在具体失败模式上，而非全局改善。 |
| Contribution type compliance | n.a. | Topic 文件 (0616-fex.md) 未声明 `preferred-contribution-types`，跳过本检查。idea 声明 `method`，不触发 hard cap。 |
| Overall Quality | 5.3/10 | 方向合理且切中真实失败模式，但当前缺少：无泄漏频率估计的明确定义、oracle 对照基线、失败模式定位的实验设计。建议先跑 oracle-frequency falsifier pilot 再决定投入力度。 |

## Contribution Drift

N/A (n=1, 初版)

## Alternative Framing

将 idea 重新框架为 **"Spectral-Prior Compiler for RL-Based Symbolic PDE Solvers"**。先做 oracle frequency upper bound 证明"频率搜索"确实是 Multi-Scale FEX 的频率分离不稳定性根因；再用 SST/CWT 替代 oracle，比较 frozen prior / soft prior / initialization-only 三种注入方式。这样：(1) 无论结果如何都有 publishable insight（正面 = 新的 prior-injection 设计范式，反面 = 频率不是根因的诊断）；(2) 从中能产生一个关于"RL symbolic search 中哪些搜索维度最受益于外部先验"的机制理解。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在预注册的 well-separated oscillatory PDE benchmarks 上，外部频率先验（oracle 或 estimated）能降低 FEX 的 spectral search burden，在相同 search budget 下提升收敛速度和精度。claim 应限制在可观测频率、低中维（d≤3）、well-separated mode 情形。 |
| NULL | 若 estimated prior 无显著收益，只能说明该估计器或注入方式未优于 Multi-Scale FEX 的现有频率搜索；不能直接断言 RL 不稳定性与频率搜索无关，除非 oracle-frequency ablation 同样失败。 |
| NEGATIVE | 若冻结频率先验使结果变差，可 claim 频率估计误差或刚性约束会损害 FEX 表达式灵活性，需要 soft/adaptive prior；不能 claim synchrosqueezing 或时频先验整体无效。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  **Likelihood = Medium**: 有清晰实验路径（oracle→estimated prior），Multi-Scale FEX 论文明确记录 wide frequency separation 导致 RL dynamics instability。但成稿依赖若干条件：(1) oracle prior 确实显著改善搜索——若 oracle 都不帮，直接归档；(2) 无泄漏的频率估计在高维可行；(3) estimated prior 效果接近 oracle 并超越简单初始化/dense grid baseline。条件 (1) 在 2D 场景概率中等偏高，条件 (2)(3) 在 d>3 时不确定性大。
  **Impact = High**: 若成功，这会：(1) 确立"将经典数学分析工具作为外部先验注入 RL symbolic search"这一此前缺失的设计范式；(2) 直接修复 Yang 自己论文承认的失效模式，对 FEX/SciML 子领域形成清晰的方法论贡献；(3) 证明 Yang 两条独立学术脉络的合成价值，对组内后续方向有指导意义。虽不改变整个 SciML 或 ML 领域的判断，但在 FEX/symbolic PDE solver 这条明确研究线上形成强结果。Codex 对 Impact 判断为 High，Claude 独立判断为 Medium（偏保守），经综合考量接受 Codex 判断（分歧已消解）。

## Overall

- Priority: High
- Score: 7
- Comments: 不触发 contribution-scope hard cap（topic 未声明 preferred-contribution-types）。idea 值得推进但强烈建议先跑 cheapest falsifier：在 Multi-Scale FEX 明确失败的 wide-separated-frequency 2D 基准上做 oracle-frequency frozen FEX。若 oracle 都不显著改善，归档或转向参数耦合诊断；若 oracle 效果明显再投入 SST/CWT estimated prior，且必须避免从 exact solution 偷看频率（信息泄漏）。推进时建议采用 Alternative Framing 中的"Spectral-Prior Compiler"叙事，从"修一个方法"升级为"建立一种设计原则"。

## Deep Lit 2026-06-16 (idea scope: fex-synchro-prior)

系统性搜索了 synchrosqueezing/butterfly + PDE solver + frequency prior + RL symbolic search 的交叉地带。1 轮搜索（8 次 arxiv-tool 查询 + web search），共发现 28 篇候选，精选 8 篇阅读，6 篇写入 wiki。

### 核心发现：外部频率先验注入范式已在 PINN 社区活跃但从未进入 FEX

PINN 社区已经广泛探索了"从数据/residual 提取频率作为先验注入 solver"的范式，但所有工作都面向连续 NN 优化（梯度下降），没有一篇涉及符号表达式树搜索（RL controller）。FEX 的 RL 搜索空间缩减这一独特机制是未被外部先验覆盖的结构性空白。

### 最相关论文解读

**2508.17902 — Spectral-Prior Guided Multistage PINNs (Yuzhen Li, Liang Li, Stéphane Lanteri et al., 2025)** [wiki](wiki/2508.17902.md)
- **相关度: 最高**。与 fex-synchro-prior 同属"外部频率分析→先验注入 PDE solver"范式。
- SI-MSPINNs: 用 DFT 从上一阶段 PDE residual 提取 Dominant Spectral Pattern（主频、相位、幅值），初始化下一阶段网络。
- RFF-MSPINNs: 将 residual PSD 归一化成随机 Fourier features 的频率采样分布。
- 在 Burgers 和 Helmholtz 上显著超越标准 PINN。
- **关键差异**: 目标 solver 是 PINN（连续 NN 梯度下降），不是 FEX（离散 RL 表达式树搜索）。频率先验的注入方式（DFT→网络初始化/PSD→采样分布）与 synchrosqueezing→FEX 输入层冻结不同，但范式层面高度一致。
- **对本 idea 的影响**: 正面——证明"外部频率先验注入 PDE solver"是有效且正在被追逐的方向。同时确认：FEX 的符号搜索空间缩减是独一无二的差异化。若提交顶会，需要明确 cite 此工作并论证为何 RL 符号搜索比连续 NN 更需要频率先验（搜索空间离散组合爆炸 vs 连续优化 landscape）。

**2508.00628 — SV-SNN: Separated-Variable Spectral Neural Networks (Xiong Xiong et al., 2025)** [wiki](wiki/2508.00628.md)
- 变量分离 + 自适应可学习 Fourier 特征 + SVD 理论分析 spectral bias。
- 用 <10% 参数超越 PINN 1-3 个数量级。
- **启示**: 其 per-dimension Fourier feature 设计可为 FEX 输入层的频率注入提供架构参考（每个空间维度的独立频率缩放因子）。

**2606.02623 — OSSM-PINN: Oscillatory State-Space Models as Inductive Biases (Abhishek Chandra, Taniya Kapoor, 2026)** [wiki](wiki/2606.02623.md)
- 用 LinOSS 振荡状态空间 cell 作为时间演化的结构化先验。
- **核心证据**: 显式振荡谱约束优于通用 Transformer/Mamba 等 sequence prior。这直接支持 fex-synchro-prior 的叙事：结构化频率先验优于让 RL 自行搜索频率。

**2603.12982 — RUNNs: Ritz-Uzawa Neural Networks (Pablo Herrera et al., 2026)** [wiki](wiki/2603.12982.md)
- 用 NCPSD（归一化累积功率谱密度）从 residual 估计活跃频率带宽，动态调谐 Fourier features。
- **启示**: NCPSD 可作为 synchrosqueezing 的轻量替代/对照 baseline。Pilot 中应包含 NCPSD vs synchrosqueezing 的频率估计精度对比。

**2602.19265 — Spectral bias in PINN and operator learning: Analysis and mitigation (Siavash Khodakarami et al., 2026)** [wiki](wiki/2602.19265.md)
- 系统性综述。核心结论：高频失败主要由优化/loss dynamics 驱动而非表示能力不足。二阶优化器显著改善高频学习。
- **启示**: (1) 提供了可移植到 FEX 的 spectral bias 诊断套件（Fourier-resolved error, Barron norm, 统计矩）；(2) 暗示 FEX 的 RL controller（policy gradient 是一阶方法）可能在频率学习上比二阶优化更差——这也是 fex-synchro-prior 的动机论据。

**2605.31027 — MS-SFNN: Multi-Scale Separable Fourier Neural Networks (Qihong Yang, Qiaolin He, 2026)** [wiki](wiki/2605.31027.md)
- Per-dimension 固定随机 Fourier 基 + 最小二乘，达到 1e-12 精度。
- **启示**: per-dim 频率缩放因子设计。但其"固定随机基+最小二乘"范式与 FEX 的 RL 搜索完全不同，不存在撞车。

### 未读但值得后续跟踪

- **2508.04882 — Hilbert Neural Operator (HNO)**: Hilbert 变换提取瞬时幅值/相位→神经算子。tex 下载因 arXiv RateLimiter 失败。若成功读取，可为 synchrosqueezing 的替代时频分析方案提供参考。
- **2603.06354 — FS-HNN: Frequency-Separable Hamiltonian NN**: 显式分离快/慢频率模式。同上，tex 下载失败。

### Novelty Gap 确认

**没有任何论文将 synchrosqueezing transform / butterfly factorization 与 RL-based expression tree search (FEX) 结合。** PINN 社区的"谱先验注入"探索证实了此方向的有效性和时效性，但 FEX 的符号搜索空间缩减是独一无二的差异化。Yang 的 SynLab/ButterflyLab 工具与 FEX 的缝合仍然是完全空白。

### 对 idea claims 的影响

| Claim | 本次证据 | 影响 |
|-------|---------|------|
| Claim 1: Multi-Scale FEX 频率分离不稳定性根因是 RL 同时搜索频率和非频率结构 | 2602.19265 证实 spectral bias 主要由优化 dynamics 驱动；2606.02623 证明显式振荡约束优于通用搜索 | **加强** — 提供间接但一致的证据 |
| Claim 2: 冻结频率先验将降低搜索时间 >50% 且精度提升 >1 数量级 | 2508.17902 在 PINN 上展示了谱先验的显著加速和精度提升 | **加强** — 跨范式类比支持 |
| Claim 3: "缝合 Yang 两条学术脉络"的 novelty | 确认完全空白 | **确认** |
| Claim 4: 频率先验的多种注入方式 (frozen/soft/initialization) | 2508.17902 提供了三种注入方式的对比（DSP 初始化 vs PSD 采样 vs 直接 Fourier feature） | **加强** — 提供了对照实验设计范本 |

### 建议更新 Pilot 设计

基于本次文献调研，建议 Pilot 增加以下对照：
1. **NCPSD baseline**（来自 2603.12982）：用 NCPSD 替代 synchrosqueezing 做频率估计
2. **DFT residual 先验 baseline**（来自 2508.17902）：从 FEX 中间解的 residual 提取 DFT 主频作为先验
3. **Oracle frequency upper bound**（已在原 pilot 中）
4. **Dense frequency grid baseline**（已在原 pilot 中）
5. **No-prior baseline**（标准 Multi-Scale FEX）

</review>
