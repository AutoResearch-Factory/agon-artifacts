---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-synchro-prior/
---

- One-sentence summary: Spectral-Prior Compiler 将可观测的 RHS / residual 频谱信号变成 FEX 的频率候选或软约束, 让 RL controller 主要搜索非频率表达式结构, 专门处理 Multi-Scale FEX 在宽频率分离时的搜索不稳定.
- Hypothesis: Multi-Scale FEX 的宽频率分离失败主要来自离散频率选择与表达式结构搜索耦合; 若 oracle 频率先证明这个瓶颈真实存在, 再用非泄漏的 RHS / early-residual FFT、NCPSD 或 synchrosqueezed estimator 提供频率先验, FEX 应在相同搜索预算下更快找到正确振荡结构. 若 oracle 频率也无收益, 根因更可能是连续参数耦合或 residual reward 失真, 该路线应收窄或归档.
- Expected outcome: 成功信号是在预注册的 well-separated oscillatory PDE 上, oracle 或 RHS/residual-estimated prior 让达到同等 relative L2 的候选评估数或 wall-clock 降低 >50%, 且 soft prior 在轻微频率误差下不劣于标准 Multi-Scale FEX; 最便宜的反证是 oracle-frequency FEX 也不优于原 FEX, 这会否定"频率搜索是主瓶颈"的解释.
- Contribution type: method
- Risk: MEDIUM
- Estimated effort:
  - Compute: pilot 已用本机 RTX 4060 Ti 跑完; 完整 FEX ablation 预计 40-80 GPU-hours
  - Data: available (manufactured Multi-Scale FEX-style PDE; 无需外部数据)
  - Implementation: 2-4 weeks
- Novelty quick-check: Multi-Scale FEX (2510.22497) 已有 symbolic spectral composition 和可学习频率缩放, 但频率仍由 RL/连续优化在表达式搜索中发现; Spectral-Prior Guided MSPINNs (2508.17902) 与 RUNNs (2603.12982) 用 residual FFT/PSD/NCPSD 指导 PINN 或 variational NN, 不处理 RL expression-tree search. 本 idea 的差异是把可观测频谱信号变成 FEX controller 的候选裁剪、软采样分布或初始化, 直接减少离散符号搜索负担.
- Strongest objection: RHS/residual 频率估计可能只在低维、规则网格、可分离振荡上可靠; 硬冻结错误频率会把 FEX 锁进错误表达式类.
- Why we should do this: 频谱先验已经在 PINN/variational NN 中显示价值, 但还没有人把它用于 FEX 的离散符号搜索. 先跑 oracle falsifier 可以避免在错误根因上投入过多; 成功时是一个清楚的方法论文, 失败时也能定位 Multi-Scale FEX 频率分离问题的真正瓶颈.
- Pilot:
  - Setup: 本机 RTX 4060 Ti + CuPy, 2D manufactured oscillatory PDE proxy `sin(2*pi*4*x)sin(2*pi*17*y)`, 频率字典 `1..24` 的 576 个 sine-product pair; 比较 oracle prior、RHS-FFT estimated prior、soft 3x3 neighborhood、dense grid 和 16 seed random no-prior search.
  - Metric: 若 estimated prior 在不读取 exact solution 的情况下恢复真频率并把候选评估从 dense grid 的 576 降到 <=9, 同时 random no-prior median relative L2 >0.5, 信号为正.
  - Result: RHS-FFT 和 oracle 都在 1 次评估中恢复 `(4,17)` 且 relative L2 = 0; soft neighborhood 9 次评估 relative L2 = 0; dense grid 576 次评估 relative L2 = 0; random no-prior 64 次/seed 仅 1/16 命中, median relative L2 = 1.0; perturbed frozen `(5,17)` relative L2 = 1.0.
  - Signal: POSITIVE

- Claims and Claims matrix: 主 claim 是"可观测频谱先验能把 FEX 中最难的频率选择从 RL 符号搜索中分离出来"; 辅助 claim 是"soft/adaptive prior 比硬冻结更稳健". POSITIVE: oracle 与 estimated prior 都在 Multi-Scale FEX 宽频率 benchmark 上提升搜索效率和精度, claim 限定为 d<=3、可观测、well-separated modes. NULL: oracle 有效但 estimated 无效, claim 收窄为 estimator 问题, method 保留 oracle/compiler 上界与更强 estimator 需求. NEGATIVE: oracle 无效, 不再声称频率搜索是主瓶颈, 转向连续参数耦合或 reward 诊断.
- Narrative: 这篇论文不讲"给 FEX 加一个 Fourier 特征", 而是把经典频谱分析当作进入 RL expression-tree search 之前的一步编译: 能从 PDE 中看见的频率, 不再交给 controller 硬搜. 这也解释了为什么 PINN 里的谱先验结果不能直接覆盖 FEX: FEX 的收益来自缩小离散搜索空间, 不是改善连续神经优化.
- Experiments: 第一组做 oracle-frequency FEX falsifier: 标准 Multi-Scale FEX、dense grid、oracle frozen、oracle soft、initialization-only, 用候选评估数、wall-clock、relative L2 和 frequency-resolved error 评价. 第二组做非泄漏 estimator: RHS FFT、early-residual DFT、NCPSD、synchrosqueezing/CWT, 比较估计误差和最终 FEX 误差. 第三组做鲁棒性: 加频率扰动、非线性谐波、局部振荡和 d=2/3 几何变化, 验证 soft prior 何时比 hard freeze 安全.
- Assets status: CuPy GPU pilot 已完成且无外部数据依赖; 交接信息见 `workspace/fex-synchro-prior/data/MANIFEST.md`.

<review date="2026-06-16">

## Novelty

- Score: 7/10
- Closest prior work: Spectral-Prior Guided Multistage PINNs (2508.17902, Li et al. 2025); Multi-Scale Finite Expression Method (2510.22497, Hardwick & Yang 2025)
- Key differentiator: 2508.17902 和 RUNNs (2603.12982) 用残差频谱引导 PINN 或 variational NN 的连续优化，不涉及离散 RL 表达式树搜索。Multi-Scale FEX 已有符号谱合成模块，但频率仍由 RL controller 在表达式搜索中共同发现——没有外部编译步骤。本 idea 的差异化在于：将可观测频谱信号作为进入 RL expression-tree search 之前的编译步骤，通过候选裁剪或软采样分布直接缩小离散搜索空间。这是机制层面的差异（搜索空间缩减 vs 连续优化改善），而非仅"把 PINN 的谱先验搬到 FEX"。但需注意，这一差异化的强度取决于 oracle-to-estimator 实验能否证明频率搜索确实是 FEX 的具体瓶颈。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v2 修复了 v1 最严重的信息泄漏模糊性（明确用 RHS / early-residual 信号），pilot 已完成，oracle falsifier 逻辑清晰。剩余 gap：(1) pilot 在手工 sine dictionary 上完成，非真实 Multi-Scale FEX RL controller——尚未证明 controller 的频率搜索（而非连续 alpha 调参、reward 噪声或候选评分）是瓶颈；(2) RHS 频谱与解频谱在非线性算子、非规则域下可能不一致，pilot 的 manufactured PDE 不能覆盖此情况；(3) soft prior 的鲁棒性声称为初步推测，pilot 仅测了 3x3 neighborhood，未系统扫频率误差。 |
| Missing evidence signals | 5/10 | v2 pilot 是重要进展，但仍是 toy proxy。关键缺失：(1) 在 Multi-Scale FEX 明确失败的 wide-separated-frequency benchmark 上跑真实 oracle-frequency FEX；(2) multi-seed RL 搜索曲线（FEX controller 有随机性）；(3) 频率估计开销（FFT/SST 耗时）纳入 wall-clock 对比；(4) RHS-FFT vs early-residual DFT vs NCPSD vs synchrosqueezing 的估计精度和最终 FEX 误差对比；(5) 非规则域/非均勻采样下的 estimator 鲁棒性；(6) 频率估计误差与最终 PDE 误差的相关性分析。 |
| Narrative | 7/10 | "Spectral-Prior Compiler" 框架远优于 v1 的"缝合 Yang 两条脉络"内部叙事。清晰地把 idea 定位为"不是给 FEX 加 Fourier 特征，而是把经典频谱分析作为 RL expression-tree search 之前的编译步骤"。叙事仍需注意：在 oracle FEX ablation 成功之前，不宜过度声称已确认频率搜索是根因。建议强调"因果诊断"角度。 |
| Venue contribution | 6/10 | 按 FEX 文献发表模式（JMLR/JCP/AAAI）和 topic 无显式 target-venue 推断，目标应为 SciML/ML 顶会。当前 pilot 仅在 toy sine dictionary 上验证，远未达到顶会标准。若完成真实 Multi-Scale FEX 集成 + oracle falsifier + estimator comparison + 系统 benchmark，可达 NeurIPS/ICLR 水平的方法论文。当前阶段更接近 AAAI/ICLR workshop 级别。 |
| Testability | 8/10 | Expected outcome 包含明确的最便宜反证：如果 oracle-frequency FEX 不优于标准 Multi-Scale FEX，直接否定"频率搜索是主瓶颈"的解释。pilot 已在 proxy 上给出 POSITIVE 信号。POSITIVE/NULL/NEGATIVE 三态 claims 界限清晰。建议在进入完整 FEX 实验前，先用真实 FEX codebase 跑一次 oracle falsifier（2-3 天工作量），作为 Go/No-Go gate。 |
| Outcome realism | 7/10 | v2 将 v1 的"精度提升 >1 数量级"收窄为"候选评估数或 wall-clock 降低 >50%"，更现实。pilot 显示 oracle 从 576 降到 1——但这是手工 sine dictionary 的最优情形。在真实 Multi-Scale FEX 中，频率成分多、非整数频率比、连续 alpha 调参等因素会压缩 gain。Wall-clock 降低需扣除频率估计开销。Soft prior 在轻微误差下不劣于标准 FEX 是合理预期。 |
| Contribution type compliance | n.a. | Topic 文件 (0616-fex.md) 未声明 `preferred-contribution-types`，跳过本检查。idea 声明 `method`，不触发 hard cap。 |
| Overall Quality | 6.7/10 | v2 相比 v1 有实质性改善：pilot 完成且 POSITIVE、oracle falsifier 设计到位、叙事升级为 Spectral-Prior Compiler、claims 收窄并分三态。但当前证据主要验证了手工频率字典上的频率查找，尚未触及真实 FEX RL controller 机制。进入实验阶段前必须先跑真实 oracle-FEX falsifier。 |

## Contribution Drift (n=2)

- v_{n-1} contribution types: {method}
- v_n contribution types: {method}
- Status: unchanged
- Hard cap triggered: no

Refiner 对 v1 每条 concern 的处理：
- 信息泄漏/数据源模糊 → resolved（明确 RHS/early-residual）
- Pilot SKIPPED → resolved（完成且 POSITIVE）
- Oracle-frequency ablation missing → resolved（pilot 包含）
- Dense grid baseline → resolved
- Soft vs frozen prior → resolved（pilot 包含 soft 3x3 neighborhood）
- 叙事从"缝合 Yang 脉络"升级 → resolved（采纳 Alternative Framing 中的 Spectral-Prior Compiler）
- Claims discipline → resolved（POSITIVE/NULL/NEGATIVE 三态表）
- 真实 FEX codebase 集成 → partially resolved（pilot 在 CuPy proxy 上而非真实 FEX）
- NCPSD/synchrosqueezing estimator 对比 → partially resolved（pilot 仅测 RHS-FFT）
- 频率误差相关性分析 → ignored

Refiner 未采纳/未完全解决的建议均有合理 pushback（pilot 选择最小可行 proxy 先验证核心假设，estimator 对比和误差分析留给后续实验阶段），接受。

## Alternative Framing

当前" Spectral-Prior Compiler"框架已经足够锐利。若要加强顶会叙事，可将中心从"编译方法"略微调整为**"因果诊断 + 谱编译"**：将 oracle-frequency FEX 实验提升为核心贡献而非 pilot，论文的核心问题是"Multi-Scale FEX 的频率分离不稳定性根因到底是频率搜索、连续参数耦合还是 reward 失真？" oracle 实验回答这个因果问题，estimated prior 实验展示一种修复方案。这样：(1) 即使 oracle 失败，负面结果本身也是可发表的诊断发现；(2) 正面结果是"诊断 + 修复"的完整故事，比单纯的"加了一个预处理步骤"更有顶会分量。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在 d<=3、频率可观测、well-separated 的振荡 PDE 上，谱先验能将 FEX 的离散频率搜索负担从 RL controller 中分离出来，使达到同等 relative L2 的候选评估数或 wall-clock 降低 >50%。claim 限定于已验证的 PDE 类和频率估计器。 |
| NULL | 若 oracle 有效但 estimated prior 无效，定位于估计器问题而非搜索范式问题——方法保留 oracle/compiler 上界价值，后续工作方向为更强估计器。若 oracle 也无效，不再声称频率搜索是瓶颈，转向连续参数耦合或 reward 诊断。 |
| NEGATIVE | 若冻结错误频率损害结果，可声称硬约束 brittle，soft/adaptive prior 或 initialization-only 注入是必要设计选择。不能声称频谱先验整体无效。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  **Likelihood = Medium**: 有清晰的实验路径（oracle→estimated prior→robustness），pilot 在 proxy 上 POSITIVE，Multi-Scale FEX 论文明确记录了 wide frequency separation 导致 RL dynamics instability。但成稿依赖若干条件成立：(1) oracle prior 在真实 Multi-Scale FEX benchmark 上显著改善搜索——proxy pilot 鼓励但不保证；(2) 非泄漏频率估计在更复杂 PDE（非线性、非规则域）上可靠；(3) estimated prior 效果接近 oracle 并超越 dense grid baseline；(4) soft prior 在频率估计误差下保持鲁棒。条件 (1) 概率中等偏高，条件 (2)(3) 存在真实不确定性。Claude 与 Codex 对 Likelihood 判断一致为 Medium。
  **Impact = High**: 若成功，将：(1) 确立"将经典频谱分析编译为 RL 符号搜索先验"这一此前缺失的设计范式；(2) 直接定位并修复 Multi-Scale FEX 论文明确承认的失效模式；(3) 证明 Yang 两条独立学术脉络的合成价值。虽不改变整个 SciML/ML 领域判断，但在 FEX/RL-symbolic-PDE 这条明确研究线上形成强结果。Claude 与 Codex 对 Impact 判断一致为 High。

## Overall

- Priority: High
- Score: 7
- Comments: v2 相比 v1 有显著提升——pilot 完成且 POSITIVE、oracle falsifier 设计出色、叙事升级为 Spectral-Prior Compiler、claims 三态分明。不触发 contribution-scope hard cap（topic 未声明 preferred-contribution-types；v1→v2 contribution type 不变）。当前最大风险是 proxy pilot 到真实 FEX codebase 的迁移不确定性——建议在投入完整实验前用真实 Multi-Scale FEX 跑一次 oracle-frequency falsifier（预计 2-3 天），作为 Go/No-Go gate。若 oracle 在真实 FEX 上也显著改善，该 idea 具有 High (7) 的顶会潜力；若 oracle 无效，应及时归档或转向连续参数耦合诊断。

</review>

<deep-lit date="2026-06-16" rounds="2" new-papers="10">

## Deep Literature Review: fex-synchro-prior — 2026-06-16

Search axes covered: method (spectral prior + expression tree), application (multi-scale oscillatory PDE), data/benchmark, evaluator/judge, failure mode (RL instability), adversarial framing. B7 reverse expansion: references + cited + author + title-term for all 10 wiki_written.

### Round 1 (7 papers)

#### 1. Zero Collapse: A Failure Mode of Policy Gradient Methods in Discontinuous Reward Environments (2605.30896) [wiki](wiki/2605.30896.md)

- **关键发现**: 识别了 policy gradient 在 threshold/discontinuous reward 环境中的 "zero collapse" 失效模式——stochastic exploration + finite step size 导致 policy 过冲到零奖励平坦区, 梯度信号退化后难以恢复。Actor-critic 特别脆弱 (critic smoothing bias 加速 collapse)。
- **与 fex-synchro-prior 的关系**: 不直接是符号 PDE/FEX 论文, 但为理解 Multi-Scale FEX 的 RL frequency search failure 提供了精确的诊断框架。FEX 的频率搜索 reward landscape 可能具有类似的不连续性——只有命中正确频率对才能获得低 residual, 错过则 residual 平坦。Zero collapse 机制可解释为什么 Multi-Scale FEX 在 wide frequency separation 时 RL dynamics 变得不稳定: controller 可能在频率空间中 "过冲" 到零信号区。
- **可用工具**: update norm/KL 控制、informed initialization、baseline shaping、reward histogram/entropy diagnostics。
- **撞车风险**: 低（非 FEX/PDE 论文, 但对其 RL 失效分析高度可引用）。

#### 2. Overcoming Spectral Bias via Cross-Attention (2512.18586) [wiki](wiki/2512.18586.md)

- **关键发现**: 提出 RFF-CA/NN-CA 架构: multiscale random Fourier feature bank + cross-attention 做输入相关的频率选择; Adaptive Frequency Enhancement (AFE) 用 DFT 从中间预测抽取 dominant modes 逐步注入 attention bank; PDE 部分用高低频分解 (u_h + α u_l)。
- **与 fex-synchro-prior 的关系**: AFE 的 "DFT→dominant modes→feature bank" 管线是本 idea "residual FFT→frequency prior→FEX controller" 的最接近现有工作, 但 AFE target 是 continuous NN optimization, 不是 RL expression-tree search。这确认了差异化的正确方向: 本 idea 的核心差异在于把频谱信号编译进离散符号搜索空间, 而非连续特征增强。
- **可用工具**: RFF bank 构造、mode-wise error、高低频分解。
- **撞车风险**: 低-中（若声称 attention-based spectral token routing 则撞车, 但我们做的是 RL tree search 编译）。

#### 3. Hierarchical Physics-Embedded Learning for Spatiotemporal Dynamical Systems (2510.25306) [wiki](wiki/2510.25306.md)

- **关键发现**: 两级 AFNO: 第一层学习/硬编码基础物理表达式, 第二层学习组合规则。在稀疏噪声观测下做 forward prediction + inverse PDE discovery。用 deep symbolic regression 从 learned operator 抽取显式方程。
- **与 fex-synchro-prior 的关系**: 分层解耦 (physics terms → combination rules) 与本 idea 的 "频率选择与表达式结构解耦" 在范式层面高度类似——两篇论文都在论证 "把某类知识从搜索中分离出来" 的价值。差异: HPE-AFNO 解耦的是 physics terms vs. 组合规则, 我们解耦的是频率 vs. 表达式结构。可作为 parallel evidence 引用。
- **可用工具**: hierarchical feature design、partial physics 外推、符号回归前的平滑抽取。
- **撞车风险**: 低（范式相似但 domain 不同）。

#### 4. IFeF-PINN: Iterative Training of PINNs with Fourier-enhanced Features (2510.19399) [wiki](wiki/2510.19399.md)

- **关键发现**: hidden-feature RFF + two-stage training: (i) estimate basis in feature space, (ii) regression/QP 确定系数。迭代更新上层 basis。高频 convection relative L2 从 ~1.0 降到 ~0.0025。
- **与 fex-synchro-prior 的关系**: 高度撞车 hidden-feature RFF + two-stage training 架构, 但 target 是 PINN continuous optimization。直接确认了 "在特征空间加 RFF 有效" 的先例, 同时确认了 "这一点还没人在 RL expression-tree search 中做过"。
- **撞车风险**: 高（对 two-stage Fourier feature 增强方法）。
- **差异化**: 本 idea 的编译步骤 target 离散搜索空间缩减, 不是连续特征增强; IFeF-PINN 没有 RL controller 或表达式树。

#### 5. PEM-UDE: Scientific ML of Chaotic Systems Discovers Governing Equations (2507.03631) [wiki](wiki/2507.03631.md)

- **关键发现**: prediction-error method 注入 UDE 训练平滑混沌优化景观, 再用 SINDy/STLSQ 抽取显式方程。在 Rössler、5× 噪声电路、稀疏 Izhikevich 神经群体上成功。发现 connection density 与 oscillation frequency/synchrony 的 emergent relationship。
- **与 fex-synchro-prior 的关系**: 不直接相关（target 是 chaotic ODE discovery, 不是 oscillatory PDE solving）, 但提供了 "先连续去噪再符号提取" 的成功先例和 "频率在方程发现中确实重要" 的实证支撑。PEM 稳定化机制可启发 FEX 在 noisy residual 场景下的改进。
- **撞车风险**: 低（方法论不重叠）。

#### 6. Symmetry-Informed NN and Symbolic Regression via Characteristic Curves (2601.21720) [wiki](wiki/2601.21720.md)

- **关键发现**: 把方程发现约束在 characteristic curves (CC) 结构骨架内, 加入奇对称约束和 post-symbolic regression。先验越强, CC RMSE 和 forward validation 越稳定。
- **与 fex-synchro-prior 的关系**: 核心范式 "先验知识→约束搜索空间→改善发现" 与本 idea 完全一致, 但用的是 symmetry + CC skeleton, 我们用的是 spectral prior + frequency compiler。这确认了 "prior-guided search space reduction" 是有效范式, 且 spectral prior 是此前未探索的先验类型。
- **可用工具**: symmetry loss、forcing residual training、post-PySR。
- **撞车风险**: 低-中（范式层面类比, 可作为 supportive citation）。

#### 7. PRISMA: PDE Residuals as Spectral Attention in Diffusion Neural Operators (2512.01370) [wiki](wiki/2512.01370.md)

- **关键发现**: 把 PDE residual 从外部 loss guidance 改造为 diffusion neural operator 内部的谱注意力 (phase-aware attention + spectral gain + residual gate)。noisy PDE 下 20-50 steps 收敛, 比 DiffusionPDE 快 15-250 倍。
- **与 fex-synchro-prior 的关系**: "PDE residual → spectral domain → 注入模型" 的管线与本 idea "residual FFT → frequency prior → FEX controller" 在信号流向上相似, 但 target 是 diffusion model (gradient-free inference), 不是 RL expression-tree search。
- **关键提示**: 要求 fex-synchro-prior 避免声称首次使用 residual spectrum; 差异应放在 "把谱信号编译进 RL expression-tree search"。
- **撞车风险**: 低（不同模型范式）。

### Round 2 (3 papers, from B7 reverse expansion)

#### 8. Policy Gradient with Tree Search: Avoiding Local Optima through Lookahead (2506.07054) [wiki](wiki/2506.07054.md)

- **关键发现**: PGTS 把标准 PG 替换为 m-step lookahead, 理论证明 stationary policy set 随深度单调收缩 (Π^m ⊆ Π^{m-1}), 无限深度只剩全局最优。在高直径、稀疏 reward 环境中 PG 短视/卡住, PGTS 靠前瞻接受短期 return 下降换取长期最优。
- **与 fex-synchro-prior 的关系**: **这是 10 篇中最直接服务于 idea 理论基础的论文。** FEX 的 RL controller 本质上就是 PG over expression trees——如果频率选择是导致 local optima 的原因, PGTS 的 lookahead 理论为 "把频率从搜索空间中分离出来" 提供了形式化 justification: 频率先验等价于缩小 MDP diameter, 减少 lookahead 需求。
- **可用工具**: T^m Q^π 前瞻打分、visited-state-only update、Ladder/Tightrope 测试环境。
- **撞车风险**: 中（若 claim PG/agent lookahead 逃离局部最优, 本文是直接 prior; 但我们 claim 的是 prior-based search space reduction, 不同机制）。

#### 9. When is a System Discoverable from Data? Discovery Requires Chaos (2511.08860) [wiki](wiki/2511.08860.md)

- **关键发现**: 混沌帮助可发现性——chaotic attractor 的 Hausdorff 维数超 d-1 时可解析发现; 存在非平凡 first integral 的稳定/守恒系统不可仅靠轨迹发现。benchmark 的混沌偏差可能掩盖了非混沌系统的根本不可发现性。
- **与 fex-synchro-prior 的关系**: **直接为本 idea 提供理论动机。** 如果非混沌 PDE 系统（如低维振荡 PDE）本质上更难从纯数据中发现, 那么注入谱先验就不是可选的 heuristic, 而是必要的 identifiability 条件。这升级了 idea 的叙事: 从 "频谱先验加速搜索" 变成 "频谱先验是确保非混沌 PDE 在有限表达式空间内可发现的必要条件"。
- **可用工具**: set-of-uniqueness 框架、conservation-law Hessian 条件、benchmark 混沌偏差分析。
- **撞车风险**: 高理论价值, 零方法论撞车。

#### 10. Action Collapse in Policy Gradient (2509.02737) [wiki](wiki/2509.02737.md)

- **关键发现**: 理想离散 RL 中 policy DNN 的最后层特征和动作头形成 simplex ETF 结构; 固定 ETF action head (ACPG) 可加速收敛。覆盖 REINFORCE、PPO、TRPO 和 CartPole/Atari。
- **与 fex-synchro-prior 的关系**: 与 Zero Collapse (2605.30896) 互补——Action Collapse 是 representation 层面的退化, Zero Collapse 是 reward landscape 层面的退化。两者共同构成 FEX RL controller 失效分类学的两个维度。
- **可用工具**: ETF baseline、AC 指标、探索消融。
- **撞车风险**: 低。

### 撞车风险汇总

| 论文 | 撞车风险 | 撞车维度 |
|------|---------|---------|
| IFeF-PINN (2510.19399) | ⚠️ 高 | two-stage Fourier feature 增强, 若未声明差异 |
| PGTS (2506.07054) | 🟡 中 | PG + tree search lookahead 理论 |
| Cross-Attention (2512.18586) | 🟡 中 | DFT→feature bank 管线 |
| HPE-AFNO (2510.25306) | 🟢 低 | 分层解耦范式类似但 domain 不同 |
| PRISMA (2512.01370) | 🟢 低 | residual→spectral 管线类似但模型范式不同 |
| CC-Symmetry (2601.21720) | 🟢 低 | prior-guided search 范式类比 |
| Zero Collapse (2605.30896) | 🟢 低 | 诊断工具, 非竞品 |
| PEM-UDE (2507.03631) | 🟢 低 | 方法论不重叠 |
| Discoverability (2511.08860) | 🟢 低 | 理论支撑, 非竞品 |
| Action Collapse (2509.02737) | 🟢 低 | 辅助诊断 |

### 对 idea 的影响

**正面强化**:
1. **理论地基大幅加强**: Zero Collapse + Action Collapse 提供了 RL controller 失效的双维度诊断框架; PGTS 提供了 formal justification; Discoverability Requires Chaos 把谱先验从 acceleration heuristic 升级为 identifiability condition。
2. **差异化确认**: 10 篇中没有任何一篇把频谱信号编译进 RL expression-tree search。最近的竞争是 IFeF-PINN 和 Cross-Attention AFE, 但都 target continuous NN optimization。
3. **范式类比充足**: CC-symmetry、HPE-AFNO 都验证了 "prior-guided search space reduction" 的有效性, 可作为 supportive citation。

**需要注意**:
1. **IFeF-PINN 撞车风险**: 必须明确声明本 idea 的差异在于 discrete search space reduction, 不是 continuous feature enhancement。在 related work 中需显式对比。
2. **Discoverability 理论**: 如果 claims 涉及 "纯数据驱动发现非混沌 PDE", 2511.08860 构成理论反例。应将 claims 限定为 "在谱先验辅助下"。
3. **PRISMA 优先权**: residual→spectral 管线已有先例, 应引用并声明差异。

</deep-lit>
