---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-synchro-prior/
---

- One-sentence summary: Spectral-Prior Compiler 先用 oracle-frequency FEX 诊断 Multi-Scale FEX 的宽频率失败是否真由频率搜索造成, 再把 PDE 右端项(RHS)或 early-residual 的频谱估计编译成 FEX controller 的软频率先验.
- Hypothesis: Multi-Scale FEX 在宽频率分离时不稳定, 可能来自频率选择、连续参数优化和 residual reward 三者耦合. 如果 oracle frequency 在真实 FEX 上显著改善搜索, 说明频率选择是可干预瓶颈; 若 RHS/residual FFT、NCPSD 或 synchrosqueezed ridge 能接近 oracle, 软先验应比硬冻结更稳健. 如果 oracle 也无效, 本路线应转为连续参数耦合或 reward 诊断, 不再声称谱先验能修复 FEX.
- Expected outcome: 成功信号是在预注册的 d<=3 well-separated oscillatory PDE 上, 真实 Multi-Scale FEX 的 oracle-soft 和 estimated-soft prior 比标准 FEX 少用 >50% 候选评估或 wall-clock 达到同等 relative L2, 且频率估计开销计入后仍成立. 最便宜的反证是 oracle-frequency FEX 不优于标准 Multi-Scale FEX; 这会否定"频率搜索是主瓶颈"的中心解释.
- Contribution type: method
- Contribution drift note: v2 contribution type 是 `method`; v3 仍是 `method`. 新增诊断 gate 和多设置 evidence 只是约束方法 claim, 不把本文扩张成 benchmark、application 或 dataset paper.
- Risk: MEDIUM
- Estimated effort:
  - Compute: 60-100 GPU-hours, 主要用于真实 Multi-Scale FEX oracle/estimated prior 的 3-5 seed 搜索曲线; v3 proxy 已在本机 RTX 4060 Ti 跑完.
  - Data: available; 使用 manufactured oscillatory PDE 和 Multi-Scale FEX 风格基准, 无外部数据或模型权重.
  - Implementation: 2-4 weeks for true FEX integration and ablations.
- Novelty quick-check: Multi-Scale FEX (2510.22497) 有 symbolic spectral composition 和可学习频率缩放, 但频率仍在表达式搜索中被 controller 发现. Spectral-Prior Guided MSPINNs (2508.17902), RUNNs (2603.12982), IFeF-PINN (2510.19399) 和 PRISMA (2512.01370) 都把 residual 或 spectral signal 注入连续神经 PDE solver, 不处理 RL expression-tree search. 本轮 4 个窄查询没有发现把可观测频谱信号编译进 FEX/RL symbolic PDE search 的工作; 近期 symbolic-prior SR 论文只提供结构先验, 不是 PDE 频率先验.
- Strongest objection: 当前 pilot 虽是正信号, 但仍是 frequency-dictionary proxy, 不是 FEX controller; RHS spectrum 在非线性算子、局部振荡和非规则域上可能与解频率不一致, 而 soft prior 如果过宽会退化为 dense grid.
- Why we should do this: 这个版本把"修 FEX"改成先回答一个因果问题: 宽频率失败到底是不是频率搜索造成的. 正结果给出一个小而清楚的 prior-guided search method; 负结果也能把 FEX 的努力方向从谱先验转向连续优化或 reward 设计.
- Review response decisions: real-FEX gap 成立, 所以 v3 把真实 oracle-frequency FEX 设为 Go/No-Go gate, proxy 只作为先验编译可行性证据. 鲁棒性批评也成立, 因此 pilot 和 Experiments 加入多模、NCPSD 带宽、非均匀采样 scan、频率误差 sweep 和 wall-clock overhead. hard-freeze 批评成立, 所以 method 默认 soft/adaptive prior, hard freeze 只作 brittle control. 对 PINN prior conflict 作 pushback: 2508.17902/RUNNs/IFeF-PINN/PRISMA 覆盖的是连续优化中的谱特征或谱注意力, 没有缩小 RL 表达式树的离散频率搜索空间.
- Pilot:
  - Setup: 本机 RTX 4060 Ti + CuPy, v3 proxy suite. Uniform grid 测 `single_wide_separated` mode `(4,17)` 和 `two_mode_wide_separated` modes `(4,17),(9,22)`; 对比 RHS-FFT top-k、NCPSD 95% band、soft radius-1、dense grid、16 seed random no-prior. 另用 4096 个非均匀样本做 RHS spectral scan, 并扫 hard frequency error offset 0-3.
  - Metric: proxy 正信号要求 RHS-only estimators 覆盖真频率并把候选集合显著小于 dense grid, random no-prior median relative L2 >0.5, 且 +1 频率误差下 hard prior 失败但 soft radius-1 恢复.
  - Result: CuPy run wall 0.624s. Single mode: RHS-FFT 1 candidate relL2 0, NCPSD band 68/576 relL2 9.97e-15, random median relL2 1.0, +1 hard relL2 1.0, soft radius-1 relL2 1.68e-15. Two mode: RHS-FFT 2 candidates relL2 7.97e-17, NCPSD band 198/576 relL2 9.42e-15, random median relL2 1.0 and 0/16 seeds hit both modes. Irregular samples: RHS spectral scan top-1 relL2 3.47e-16, random median relL2 1.015.
  - Signal: POSITIVE but bounded; it supports observable frequency-candidate compilation, not yet real FEX controller improvement.

- Claims and Claims matrix: Main claim: when oracle-frequency FEX succeeds, observable spectral estimates can remove a specific frequency-selection burden from RL expression-tree search. POSITIVE: oracle-soft and estimated-soft both improve true Multi-Scale FEX by >50% candidate evaluations or wall-clock on verified wide-frequency PDE; claim limited to d<=3, observable, well-separated modes. NULL-A: oracle helps but estimated prior fails, so the compiler idea is valid but the estimator is the bottleneck. NULL-B: oracle fails, so frequency search is not the bottleneck and the paper should become a negative diagnostic, not a spectral-prior method claim. NEGATIVE: hard freeze fails under small frequency error while soft prior works, supporting only the soft/adaptive design choice.
- Narrative: 这篇文章应当先做因果诊断, 再谈修复. 先问 FEX 是否因为 controller 必须自己发现频率而失败; 只有 oracle gate 通过后, 才把 spectral compilation 作为修复方法来卖点. 这比"加 Fourier features"更窄也更有力, 并且能和 PINN spectral-bias 论文区分开.
- Experiments: Gate 0 true-FEX oracle falsifier: standard Multi-Scale FEX, oracle hard, oracle soft, initialization-only and dense frequency grid on the original wide-separated benchmark, with 3-5 seeds and frequency-resolved errors. Gate 1 injection ablation: candidate pruning, soft sampling prior, reward bonus and initialization-only under equal candidate/wall-clock budgets. Gate 2 estimator comparison: RHS FFT, early-residual DFT, NCPSD and synchrosqueezed/CWT ridge, reporting estimator overhead and final FEX error. Gate 3 robustness: frequency perturbation, two-mode and harmonic cases, local oscillation, nonuniform samples and simple nonrectangular masks; report when soft prior widens enough to lose advantage.
- Assets status: v3 CuPy proxy, results and handoff notes are ready locally; data/model status is recorded in `workspace/fex-synchro-prior/data/MANIFEST.md`.

<review date="2026-06-16">

## Novelty

- Score: 7/10
- Closest prior work: Multi-Scale FEX (2510.22497, Hardwick & Yang 2025); Spectral-Prior Guided MSPINNs (2508.17902, Li et al. 2025); IFeF-PINN (2510.19399); DSO (2505.10762, Hayes et al. 2025)
- Key differentiator: 所有现有 spectral-prior 工作 (MSPINN, RUNNs, IFeF-PINN, PRISMA, FRES) 都将谱信号注入连续神经优化——初始化、特征增强、注意力权重——没有一个涉及离散 RL 表达式树搜索空间缩减。本 idea 的核心差异化在于"谱估计→候选裁剪/软采样分布→RL controller 频率搜索空间缩小"这一编译机制。但此差异化的强度完全依赖于 real-FEX oracle 实验能否证明频率搜索确实是 FEX RL dynamics 的具体瓶颈。DSO (2505.10762) 的 RSPG 和 in-situ constraint 提供了 RL 表达式树搜索的竞争性 baseline，但不涉及谱先验。经本轮 targeted search，FRES (Hou et al. 2025, Neural Networks) 确认是 PINN 连续优化方向，不威胁 novelty。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v3 正确地将 idea 重构为"因果诊断→修复"结构，Go/No-Go gate 设计清晰。剩余 gap：(1) 所有正面证据仍来自 frequency-dictionary proxy，而非真实 Multi-Scale FEX RL controller——proxy 的成功证明了频率候选编译可行，但未证明 FEX controller 的频率搜索是瓶颈；(2) RHS/residual 频谱在非线性算子、局部振荡、非规则域下可能与解频谱不一致，idea 承认但未给出具体缓解策略；(3) 若 soft prior 过宽退化为 dense grid，频率先验的价值消失，这一退化边界未在 proxy 中充分刻画。 |
| Missing evidence signals | 6/10 | v3 pilot 相比 v2 新增多模、NCPSD band、非均匀采样和频率误差 sweep，值得肯定。但决定性证据仍缺失：(1) 真实 Multi-Scale FEX oracle-soft vs 标准 FEX 的 3-5 seed 搜索曲线对比；(2) 频率估计开销 (FFT/SST/NCPSD wall time) 计入后的 net gain；(3) synchrosqueezed/CWT ridge 与 RHS-FFT/NCPSD 的估计精度和最终 FEX 误差对比；(4) 非矩形域/非规则几何下的 estimator 表现。Gate 0 是所有后续工作的前提条件。 |
| Narrative | 8/10 | "先因果诊断，再谈修复"的叙事结构远优于 v1/v2。能与 PINN spectral-bias 文献清晰区隔。当前 Spectral-Prior Compiler 命名稍显技巧化——若 oracle gate 通过，应将 oracle 实验本身提升为核心贡献之一 ("frequency-search bottleneck diagnosis") 而非仅仅 gate，这样即使 estimator 只部分工作论文也有独立价值。 |
| Venue contribution | 6/10 | Topic 未声明 target-venue。按 FEX 文献发表模式 (~JMLR/JCP/AAAI) 和领域惯例推断，目标为 SciML/ML 顶会。当前 proxy pilot 距顶会标准仍有距离。若 real-FEX oracle 成功且 estimated-soft prior 接近 oracle，可冲击 NeurIPS/ICLR；若仅 oracle 成功，可作 diagnostic/empirical 论文投稿较低 venue；若 oracle 失败，应归档。 |
| Testability | 9/10 | Expected outcome 包含明确的最便宜反证：oracle-frequency FEX 不优于标准 Multi-Scale FEX 直接否定"频率搜索是主瓶颈"。POSITIVE / NULL-A / NULL-B / NEGATIVE 四态 claims 界限清晰且预注册。Gate 0 仅需 2-3 天工作量即可给出 Go/No-Go 信号，是出色的实验设计。 |
| Outcome realism | 7/10 | >50% 候选评估或 wall-clock 降低在 d<=3 well-separated oscillatory PDE 上是现实的——如果频率搜索确为主要瓶颈。但需注意：(1) 连续参数优化 (Adam+BFGS) 可能在真实 FEX 中主导 wall-clock，压缩 frequency-search gain；(2) 频率估计开销需从 net wall-clock gain 中扣除；(3) soft prior 在真实 FEX multi-seed RL 搜索中的方差可能大于 proxy 的确定性 dictionary lookup。claims 已适当限定为 d<=3、observable、well-separated modes。 |
| Contribution type compliance | n.a. | Topic (0616-fex.md) 未声明 `preferred-contribution-types`，跳过本检查。idea 声明 `method`，不触发 scope hard cap。 |
| Overall Quality | 7.2/10 | v3 相比 v2 有显著改善：pilot 从单一 mode 扩展到多模+NCPSD+非均匀采样+频率误差 sweep；叙事升级为因果诊断结构；Go/No-Go gate 提供了清晰的下一步决策点。但决定性证据 (真实 Multi-Scale FEX oracle ablation) 仍缺失——一切 method claim 都以此为前提。Claude 与 Codex 在各维度评分高度一致 (最大偏差 0 分)。 |

## Contribution Drift (n=3)

- v_2 contribution types: {method}
- v_3 contribution types: {method}
- Status: unchanged
- Hard cap triggered: no

v2 review 每条 concern 的当前状态：
- "pilot 在手工 sine dictionary 上完成，非真实 FEX controller" → partially resolved (v3 将 real-FEX oracle 设为 Go/No-Go gate，但尚未执行)
- "RHS 频谱与解频谱在非线性算子下可能不一致" → partially resolved (idea 已承认此风险，但未给出具体缓解策略)
- "soft prior 鲁棒性仅测了 3x3 neighborhood" → resolved (v3 加入系统频率误差 sweep 0-3)
- "真实 Multi-Scale FEX benchmark 上跑 oracle" → planned (Gate 0)，尚未执行
- "multi-seed RL 搜索曲线" → planned (Gate 0)，尚未执行
- "频率估计开销纳入 wall-clock" → planned (Gate 2)，尚未执行
- "estimator 对比 (NCPSD/synchrosqueezing)" → partially resolved (NCPSD added; synchrosqueezed/CWT 和 early-residual DFT 仍为 future work)
- "非规则域/非均匀采样鲁棒性" → resolved (v3 pilot 加入 4096 非均匀样本 RHS spectral scan)
- "频率误差相关性分析" → resolved (v3 加入频率误差 sweep)
- "PINN prior conflict" pushback on refiner side → 接受 (refiner 正确指出 MSPINN/IFeF/PRISMA 覆盖连续优化，未缩小 RL 表达式树离散频率搜索空间)

## Alternative Framing

当前 "causal diagnosis + spectral compiler" 框架已足够锐利。若进一步强化，可将论文中心从 "Spectral-Prior Compiler 方法" 调整为 **"Frequency-Search Bottleneck Diagnosis for RL Symbolic PDE Solvers, with Spectral Compilation as Repair"**——即把 oracle-FEX 实验从 gate 提升为核心贡献之一。这样 (a) 即使 estimated prior 仅部分接近 oracle，论文仍有无可争议的诊断贡献；(b) 正面结果成为 "诊断+修复" 的完整双贡献故事，比单纯的 "加了一个预处理步骤" 更有顶会分量。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在 d<=3、频率可观测、well-separated 的振荡 PDE 上，oracle-soft 和 estimated-soft spectral prior 使 Multi-Scale FEX 达到同等 relative L2 的候选评估数或 wall-clock 降低 >50%，且频率估计开销计入后 net gain 仍为正。 |
| NULL-A | oracle 有效但 estimated prior 无效：编译器范式成立但当前 estimator 是瓶颈，后续工作方向为更强估计器。 |
| NULL-B | oracle 无效：频率搜索不是 FEX 的主要瓶颈，论文转为负诊断——定位连续参数耦合或 residual reward 失真为真正根因。 |
| NEGATIVE | 硬冻结在微小频率误差下失败而 soft prior 恢复：仅声称 soft/adaptive 注入是必要设计选择，不声称谱先验整体解决 FEX 频率问题。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  **Likelihood = Medium**: 成稿路径清晰（oracle gate → estimator comparison → robustness sweep），proxy pilot 给出 POSITIVE 信号，Multi-Scale FEX 论文明确记录了 wide frequency separation 导致 RL dynamics instability。但依赖若干条件成立：(a) real-FEX oracle 显著改善搜索——proxy 鼓励但不保证，因为 proxy 将 RL controller 替换为确定性 frequency-dictionary lookup，而真实 FEX 的连续参数优化和 reward 噪声可能压缩 oracle gain；(b) 非泄漏频率估计在更复杂 PDE 上可靠；(c) estimated prior 接近 oracle 并超越 dense grid。条件 (a) 概率中等偏高 (~60-70%)，条件 (b)(c) 存在真实不确定性。Claude 与 Codex 对 Likelihood 判断一致为 Medium。
  **Impact = High**: 若成功，将 (a) 确立"将经典频谱分析编译为 RL 符号搜索先验"这一此前缺失的设计范式；(b) 直接定位并修复 Multi-Scale FEX 论文明确承认的失效模式；(c) 证明 Yang 两条独立学术脉络 (FEX RL search + SynLab spectral estimation) 的合成价值。虽不改变整个 SciML/ML 领域判断，但在 FEX/RL-symbolic-PDE 这条明确研究线上形成强结果。Claude 与 Codex 对 Impact 判断一致为 High。

## Overall

- Priority: High
- Score: 7
- Comments: v3 相比 v2 有实质性改进——pilot 从单模扩展到多模+NCPSD+非均匀采样+频率误差 sweep，叙事升级为"因果诊断→修复"结构，Go/No-Go gate 设计出色。不触发 contribution-scope hard cap (topic 未声明 preferred-contribution-types；v2→v3 contribution type 不变)。Claude 与 Codex 在所有维度上完全一致。当前最紧迫的下一步是 Gate 0：用真实 Multi-Scale FEX codebase 跑一次 oracle-frequency FEX vs 标准 FEX 的 3-5 seed 对比 (预计 2-3 天)。若 oracle 在真实 FEX 上也显著改善搜索，该 idea 具有 High (7) 的顶会潜力；若 oracle 无效，应及时归档或转向连续参数耦合/reward 诊断。

</review>

## Deep Lit 2026-06-17 addenda (idea scope: fex-synchro-prior)

以下 3 篇论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-synchro-prior` 于 2026-06-17 系统性搜索并精读后收录。22 组搜索（6 axes × 2-3 queries + supplementary + B7 反向扩展）后确认文献饱和：频谱估计+RL 表达式树搜索交叉领域无任何已有工作。

### 新增论文解读

- **Shallow NN PDE spectral bias (2510.27658)** [wiki](wiki/2510.27658.md): Roy Y. He, Hongkai Zhao 等, 2025.10. 用 Gram/K奇异值分析量化浅层 NN（PINN/DRM）求解椭圆 PDE 时的频率偏置和数值病态。核心发现：ReLU^p 全局基的 Gram 矩阵谱按 k^{-(2p+2)} 衰减，导致高频分量在直接求解和梯度训练中都难以恢复。**对 fex-synchro-prior 的价值**：(a) Gram 谱诊断和 relative spectral loss 可复用于 Gate 0 的 FEX controller 频率偏置诊断；(b) boundary regularization decomposition 实验设计可启发 FEX RL 搜索中边界处理与频率搜索的 disentanglement；(c) activation scaling sweep 提供系统扫参方法论。**不威胁 novelty**：面向 PINN 连续优化，不涉及 RL 表达式树搜索。同组后续工作 2603.00904 (Operator-Aware Preconditioners) 将诊断提升为算子感知预处理，同样面向连续 NN 优化。

- **NeuSA (2509.04966)** [wiki](wiki/2509.04966.md): Arthur Bizzi 等, 2025.09. 将 PINN 解投影到 Fourier/sine/cosine 谱基，用 Neural ODE 推进谱系数时间演化。核心创新：谱基提供解析空间导数（不依赖深层 autograd）、NODE 提供因果时间推进、Fourier multiplier 线性初始化加速收敛。在 wave/Burgers/sine-Gordon 上以更少训练步超越 PINN/QRes/FLS/PINNsFormer。**对 fex-synchro-prior 的价值**：(a) Fourier multiplier 线性初始化和 residual vector field (`M*hat u + epsilon F_theta`) 为 FEX controller 的 spectral compilation 初始化提供概念参考——即先用线性谱估计初始化 controller 的 operator embedding，再用 RL 搜索学残差；(b) dimension-wise spectral layers 可启发 FEX controller 的 per-dimension frequency prior encoding；(c) Burgers [0,1]→[0,2] 外推评测可复用于评估 FEX+spectral prior 的时间外推稳定性。**不威胁 novelty**：连续 NODE 优化，非离散 RL 表达式树搜索；依赖预选谱基类型（Fourier vs cosine vs sine），与 FEX 的 operator-tree discovery 正交。

- **Integral Bayesian SR (2511.14388)** [wiki](wiki/2511.14388.md): Oriol Cabanas-Tirapu, Guimerà, Sales-Pardo 等, 2025.11. 用数值积分评估候选微分方程轨迹，直接拟合原始时间序列，避免有限差分或平滑导数估计引入偏差。MCMC + parallel tempering 在符号表达式空间采样，用 Bayesian description length 做模型选择。在 Logistic/Lotka-Volterra 上比 FD-BMS/SD-BMS/ESINDy/W-ESINDy 更稳。**对 fex-synchro-prior 的价值**：(a) 积分验证器可复用于 FEX 的 PDE residual evaluation——若 FEX 搜索出的表达式在积分形式下不满足 PDE，说明 residual 形式存在数值问题；(b) learnability/RMSE-σ 评测可为 FEX 的 noise-robustness 扩展提供基线；(c) 本组同期综述 2507.19540 (Bayesian SR overview) 为 probabilistic SR 方法学提供地图。**不威胁 novelty**：面向 ODE 发现+少量噪声数据，使用 MCMC 搜索而非 RL policy gradient；不涉及 PDE 求解或频率估计。

### 撞车风险更新

| Claim | 状态 | 本轮新证据 |
|-------|------|-----------|
| spectral estimation → frequency prior → RL expression tree search | 🟢 未被覆盖 | 22 组搜索、B7 扩展、"frequency prior compile RL expression tree" 精确查询均 0 结果 |
| frequency search is FEX's main bottleneck | 🟢 未被覆盖 | 2510.27658 做 PINN 频率偏置诊断但非 FEX/RL；无人做此因果实验 |
| soft prior more robust than hard freeze | 🟢 未被覆盖 | 无相关比较 |

### 方法论清单（可复用于 Gate 0 诊断设计）

| 来源 | 方法 | 复用方式 |
|------|------|---------|
| 2510.27658 | Gram 谱诊断 | 对 FEX controller 的 policy/value 网络做谱分析，定量测频率偏置 |
| 2510.27658 | relative spectral loss | 按频率 bin 分解 FEX residual，定位哪个频段 controller 最弱 |
| 2509.04966 | Fourier multiplier 初始化 | FEX controller operator embedding 的 spectral warm-start |
| 2509.04966 | residual vector field | RL 搜索学 operator 残差而非全量，先验引导缩小搜索空间 |
| 2511.14388 | 积分验证器 | PDE residual 的积分形式检查，防数值微分放大噪声 |
| 2511.14388 | learnability/RMSE-σ 评测 | FEX noise-robustness 扩展的基线 |

**关键结论**：本轮深度文献调研确认 fex-synchro-prior 的核心差异化——"将经典频谱分析编译为 RL 表达式树搜索先验"——在文献中完全空白。Gate 0（oracle-frequency FEX 真实实验）是当前唯一阻塞项。
