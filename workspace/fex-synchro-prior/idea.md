---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-synchro-prior/
---

- One-sentence summary: 先用 oracle-soft 频率先验诊断 Multi-Scale FEX 的宽频率失败是否真由 controller 频率搜索造成, 再把通过验证的 RHS/early-residual 频谱估计变成有宽度上限的频率动作先验.
- Hypothesis: Multi-Scale FEX 在 well-separated oscillatory PDE 上的不稳定来自三个变量同时搜索: 频率、表达式结构和连续系数。若 oracle-soft 频率先验能缩短真实 FEX 搜索, 频率就是可干预瓶颈; 若 estimated-soft prior 在泄漏/谐波诱饵控制下仍接近 oracle, 频谱信号可以作为 search-space compiler。v5 接受 reviewer 的 RHS 泄漏批评: naive RHS-FFT 只能是候选估计器, 必须通过 candidate validation 和 decoy/nonlinear gates 后才可支撑 method claim.
- Expected outcome: 成功信号是在 d<=3、频率可观测、wide-separated 的振荡 PDE 上, 真实 Multi-Scale FEX 的 oracle-soft prior 比标准 FEX 用少于 `50%` 的候选评估或 net wall-clock 达到同等 relative L2; estimated-soft prior 在 estimator overhead 计入后达到 oracle gain 的主要部分, selected prior 覆盖不超过 dense frequency grid 的 `25%`, 且在 RHS decoy、solution/RHS spectrum mismatch 和 nonuniform samples 上不退化为 blind FFT. 若 oracle-soft 不优于标准 FEX, 最便宜反证成立, 本路线应转为连续参数耦合或 reward 诊断.
- Contribution type: method + diagnostic
- Contribution drift note: v4 contribution type 是 `method + diagnostic`; v5 保持完全相同的类型, 未删除 method 或 diagnostic, 也不新增 benchmark、application 或 dataset paper。Topic `0616-fex.md` 未声明 `preferred-contribution-types`, 因此无 subset 限制; 多 PDE、多 seed 和 decoy 控制只作为 method/diagnostic 证据, 不作为 benchmark 贡献。
- Risk: MEDIUM
- Estimated effort:
  - Compute: 100-160 GPU-hours, 主要用于真实 Multi-Scale FEX standard/oracle/estimated prior 的 3-5 seed 搜索曲线、estimator validation、overhead 计入和泄漏/非线性控制.
  - Data: available; 使用 manufactured oscillatory PDE、Multi-Scale FEX 风格基准和小型真实 FEX-family smoke, 无外部数据或模型权重.
  - Implementation: 2-4 weeks for true Multi-Scale FEX prior injection, controller logging, estimator selector, and leakage gates.
- Novelty quick-check: Multi-Scale FEX (2510.22497) 有 spectral composition, 但频率仍由 controller 搜索。MSPINN、RUNNs、IFeF-PINN、PRISMA 和 FRES 把频谱信号注入连续神经 PDE solver, 不是 RL expression-tree controller 的离散频率动作先验。DSO、SSDE、SymPlex、NetGP、Sym-Q 和 spatio-temporal-reward PDE-SR 覆盖 RL/Transformer/GP 符号搜索, 但没有把可观测频谱估计编译成 FEX 频率候选分布。v4 review 的 2026-06-17 targeted search 未发现 spectral estimation -> soft frequency prior -> RL symbolic PDE search 的已有工作. 2026-06-19 deep-lit-tick (idea scope, 163 篇已读) 进一步确认: (a) 2501.09987 综述系统梳理了连续 NN 的频率偏置缓解方法 (MscaleDNN/Fourier features/RFM/HINTS 等), 但 RL 离散表达式树频率动作的因果诊断在整篇综述中完全缺失; (b) 2603.00904 揭示 NN 低频偏好 vs 微分算子高频放大的竞争决定 PINN 训练频率动态, 为 FEX reward (PDE residual) 的频率行为提供新解释框架但不涉及离散搜索; (c) 2506.22365 (PiPRL) 是 "symbolic prior → RL policy" 的最近先例 (navigation, DSL program guiding waypoint), 但机制与本 idea 的 spectral PMF logits 注入完全不同。三轮搜索 + 12 项 B7 反向扩展均未发现 "spectral estimation → soft frequency prior → RL symbolic PDE search" 的已有工作。
- Strongest objection: 真实 Multi-Scale FEX 的 Adam/BFGS 系数优化、reward 噪声和 expression-tree credit assignment 可能吞掉频率先验收益; manufactured RHS 也可能直接暴露解频率, 使 estimated prior 看起来像读答案.
- Why we should do this: 这个 idea 的价值不是再加 Fourier features, 而是把 FEX 的 wide-frequency failure 变成一个可证伪的因果诊断: 频率搜索是否是主瓶颈。正结果给出一个小而可复用的 controller prior; 负结果也能把 Yang 组后续 FEX 工作转向真正瓶颈.
- Review response decisions: 接受 real-FEX gap, 所以 v5 仍把真实 Multi-Scale FEX oracle-soft vs standard 作为 Gate 0, 新增 real-FEX-family smoke 只作辅助证据。接受 RHS 泄漏 concern, 因此 v5 加 decoy-RHS negative control, 并把 estimator selector 改成 "RHS FFT / early residual / NCPSD / CWT-SST + candidate validation", 不允许 blind FFT 支撑主 claim。接受 frequency-structure-coefficient coupling concern, 所以 Gate 0 必须记录 frequency-action entropy、first-hit time、constant-fit loss 和 final relative L2。Pushback PINN/spectral-prior novelty conflict: 连续 PINN feature/initialization prior 不会缩小 RL expression-tree 的离散频率动作空间。Pushback benchmark/application expansion: 多 setting 只服务 evidence sufficiency.
- Pilot:
  - Setup: 本机 RTX 4060 Ti。v5 CuPy proxy 保留 single `(4,17)`、two-mode `(4,17),(9,22)`、NCPSD、soft-radius、frequency-error、非均匀采样; 新增 risk-seeking frequency-action PG proxy 和 RHS decoy `(11,3)` leakage control; 另用 sibling instrumented FEX runner 跑一个真实 FEX-family `helmholtz_sine` smoke.
  - Metric: proxy 正信号要求 estimated soft prior 比 uniform controller sampling 更快命中真频率, prior width 不越过 `25%/35%` 退化线, decoy control 能暴露 blind FFT 失败而不被写成成功; real-FEX-family smoke 只要求 oracle skeleton 显示 capacity rescue.
  - Result: CuPy wall `18.88s`. PG proxy: uniform hit `(4,17)` in `23/64` runs, median first hit `128` evaluations; RHS-FFT soft prior hit `64/64`, median first hit `7`; oracle soft prior hit `64/64`, median `8.5`; wrong +3 prior hit `57/64`, median `106`. Decoy RHS: FFT top-1 selects `(11,3)`, misses true `(4,17)`, relL2 `1.0`, confirming leakage risk. Uniform-grid FFT top-k gives `56.4x/92.9x` proxy net speedup on single/two-mode; NCPSD gives `7.8x/2.69x`; irregular spectral scan recovers the mode but is slower than dense sampled dictionary here (`0.32x`). Real-FEX-family smoke: standard `helmholtz_sine` controller seed 0 fails at relL2 `5.21e-2`, analytic sine skeleton oracle succeeds at `5.01e-8`.
  - Signal: POSITIVE but bounded; supports soft prior for controller-side sampling and validates the leakage concern, but still does not replace true Multi-Scale FEX Gate 0.

- Claims and Claims matrix: Main claim: frequency search is a measurable bottleneck in RL symbolic PDE solving, and checked spectra can turn that bottleneck into a bounded soft frequency-action prior. POSITIVE: true Multi-Scale FEX oracle-soft and estimated-soft both improve candidate evaluations or net wall-clock by >50%, selected prior <=25% dense grid, and leakage/nonlinear controls pass. NULL-A: oracle helps but estimated prior fails or is too expensive, so the controller interface is valid but current estimators are weak. NULL-B: oracle fails, so frequency search is not the main bottleneck and the paper becomes a negative diagnostic about coefficient coupling, reward deception, or credit assignment. LEAKAGE-FAIL: RHS FFT works only when manufactured RHS directly reveals the solution frequency; claim is downgraded to "blind RHS prior is unsafe." DEGENERATE: prior width exceeds 35% dense grid, so frequency is observable but not useful for search-space reduction.
- Narrative: The paper should be "Frequency-Search Bottleneck Diagnosis for RL Symbolic PDE Solvers, with Checked Spectral Priors as Search-Space Compilers." "Compiler" means prior -> bounded frequency action distribution -> smaller controller search; it must not suggest an end-to-end PDE compiler.
- Experiments: Gate 0 true Multi-Scale FEX oracle falsifier: standard, oracle hard, oracle soft, dense grid, initialization-only, and true-skeleton capacity oracle with 3-5 seeds and equal candidate/wall-clock budgets. Gate 1 injection ablation: candidate pruning, soft sampling prior, reward bonus, initialization-only, and adaptive width cap. Gate 2 estimator selector: RHS FFT, early-residual DFT, NCPSD, CWT/SST ridge and local-window variants, each with overhead and candidate-validation logs. Gate 3 leakage/robustness: RHS decoys, nonlinear RHS where solution and forcing spectra differ, local oscillations, nonuniform samples, nonrectangular masks, and frequency perturbation. Report frequency-action entropy, first true-frequency hit time, constant-fit loss, final relative L2, and prior-width crossing.
- Assets status: v5 CuPy proxy, real-FEX-family smoke, notes and rerun instructions are ready locally; data/model status lives in `workspace/fex-synchro-prior/data/MANIFEST.md`.

## Deep Lit 2026-06-22 (experiment scope, 6 篇)

以下论文由 `deep-lit-tick --scope experiment fex-synchro-prior` 于 2026-06-22 搜索并精读。本轮聚焦 reviewer must-fix 中的 retention 直接证据需求和 RL 探索漂移机制。

### RL Policy Gradient 熵动力学（直接服务 retention 诊断）

- **The Entropy Mechanism of RL (2505.22617)** [wiki](wiki/2505.22617.md): Ganqu Cui 等, 2025.05, 373 citations. 系统研究 PG 训练中策略熵崩塌。核心发现：(a) 熵变 ≈ -η·Cov(log π, π·A)，协方差全程为正→熵单调递减；(b) R = -a·exp(H)+b 精确预测性能天花板；(c) Clip-Cov/KL-Cov 仅干涉 0.01%-0.1% token 即阻止崩塌。**对本实验**：covariance-driven 熵递减机制直接解释 FEX controller 为何在搜索早期找到正确频率后丢失——risk-seeking PG 的高优势+高概率频率动作正好符合熵递减条件。建议在 frequency-action 节点实现 per-epoch entropy tracking 以直接测量此效应。

- **STEER (2510.10150)** [wiki](wiki/2510.10150.md): Zhezheng Hao 等, 2025.10, 33 citations. 推导 token 级熵变一阶近似 Ω(s) 的 4 个控制因子（clip indicator, advantage, token probability, conditional entropy）。提出基于熵变估计的自适应重加权 STEER，比 Clip-Cov 再高 2.2%。**对本实验**：4 因子分解框架可迁移到 FEX frequency-action 节点的熵变诊断——哪个因子主导了频率动作熵的递减？

- **Entropy-Preserving RL (2603.11682)** [wiki](wiki/2603.11682.md): ICLR 2026. 系统分析哪些 PG 变体天然保熵 vs 崩塌。关键发现：(a) PPO clipping bounds 熵变范围；(b) BF16 量化产生 multiplicative bias 等价于非对称 clipping，系统性加速熵崩塌；(c) REPO 通过修改 advantage 引入熵控制（零额外显存）。**对本实验**：FEX 的 risk-seeking PG 是否天然保熵或崩塌？需检查 FEX controller 的 logits precision。REPO 的 advantage 修改思路可直接迁移为 frequency-action 的 entropy regularizer。

### RL 遗忘/保持机制（解释 frequency retention vs drift）

- **Retaining by Doing (2510.18874)** [wiki](wiki/2510.18874.md): Howard Chen, Danqi Chen 等, 2025.10. 通过高斯混合模型证明：on-policy RL 的 mode-seeking (reverse KL) 可保持 old mode 不变而仅移动 new mode，比 SFT (forward KL) 遗忘更少。消融确认 on-policy data（非 KL regularization/advantage estimation）是核心因素。**对本实验**：FEX risk-seeking PG 是 on-policy 的，但仍然丢失频率——说明瓶颈不在 on-policy vs off-policy，而在 action space structure（频率动作的 mode 数太多、reward noise 太高，on-policy 的 mode-seeking 保护不够）。为 "retention 是主瓶颈" 叙事提供更精确的理论定位。

- **RL's Razor (2509.04259)** [wiki](wiki/2509.04259.md): Shenfeld, Agrawal 等, 2025.09, 118 citations. 发现遗忘程度 ∝ 微调模型与基础模型的 KL 散度（R²=0.96）。On-policy RL 隐式偏向 KL-minimal 解（"RL's Razor"），理论等价于 EM 算法收敛到 KL 最小最优策略。**对本实验**：可用 KL 散度度量 standard vs oracle_soft 的 policy 偏移量，作为 retention 的量化证据——如果 oracle_soft 的 KL 更小，说明频率先验同时稳定了搜索和保持了策略接近初始分布。

### RL 优化稳定性（Entropy-Clip 诊断）

- **BAPO (2510.18927)** [wiki](wiki/2510.18927.md): Zhiheng Xi 等, 2025.10, 39 citations. 识别 PPO/GRPO 的 Entropy-Clip Rule：固定对称裁剪系统性阻挡能增加熵的低概率正样本→策略熵持续下降。自适应动态裁剪上下界修复。**对本实验**：FEX 的 risk-seeking PG 不用 PPO clipping，但其 top-quantile reward thresholding 可能产生类似效应——只保留高 reward 样本等价于只保留高概率频率动作，低概率的"新发现"频率被过滤掉。

## Deep Lit 2026-06-24 第二轮 (experiment scope, 6 篇)

以下论文由 `deep-lit-tick --scope experiment fex-synchro-prior` 于 2026-06-24 搜索并精读。本轮聚焦 reviewer must-fix #1（替换 Gate 2 validation gate 的 true_sol 基准）和 #5（retention TFP 跨 PDE 复制的理论支撑）。

### PDE 残差验证与误差认证（直接服务 must-fix #1: 替换 true_sol）

- **Reliable Error Estimation for PINNs (2606.12050)** [wiki](wiki/2606.12050.md): Huseynov, Ahmadova, Bashirov, 2026.06. 推导 PINN 求解 ODE 时的可计算 a posteriori 双边误差界，**不需要访问真解**。核心：(a) localized strong monotonicity 条件下的误差下界；(b) one-sided Lipschitz 条件下的误差上界（比全局 Lipschitz 紧致最多 10^9 倍）；(c) 线性系统的显式谱公式；(d) signed-residual finite-probe 下界证书。**对本实验**：为 Gate 2 validation gate 替换 true_sol 提供直接理论工具——可用 PDE residual + local monotonicity/Lipschitz 常数构造不依赖 true_sol 的误差证书。one-sided Lipschitz 在旋转/振荡系统中远紧于全局 Lipschitz，直接适用于 Helmholtz 类振荡 PDE。

- **Rigorous Error Certification for Neural PDE Solvers (2603.19165)** [wiki](wiki/2603.19165.md): Mukherjee, Fitzsimmons, Fernández, Liu, 2026.03. 首次建立从训练残差到解空间误差的严格理论桥梁。核心：当近似解在紧子集中时，残差→0 保证收敛到真解。给出泛化界：解误差 ≤ 稳定常数 × (最大采样残差 + 紧集连续模)。用 dReal/autoLiRPA/∂-CROWN 做形式化验证。**对本实验**：确认残差控制是 PDE 解质量的有效代理——为 Gate 2 用 PDE residual 替代 true_sol 的合理性提供理论依据。关键限制：认证界通常保守 2-100 倍，stiffness 增加时指数恶化。

- **PhysicsCorrect (2507.02227)** [wiki](wiki/2507.02227.md): Huang, Perdikaris (UPenn), 2025.07, AAAI 2026. 训练无关的 PDE 残差校正框架：线性化 Predictor-Corrector 管道，通过 PDE 残差的 Jacobian 线性系统校正预测。关键创新：半隐式离散化使 Jacobian 常数化→离线预计算伪逆，推理校正退化为矩阵乘法（加速 163×，开销 <5%）。**对本实验**：(a) PDE 残差定义的准确性比 Jacobian 精度更关键——启发 Gate 2 validation gate 的设计哲学（用准确的 PDE residual 而非精确的 true_sol）；(b) 残差最小化可用于 FEX 候选表达式的 post-hoc 精化；(c) 核实在混沌系统（KS 方程）中近似常 Jacobian 仍有效，为 FEX 中使用简化的 validation 提供信心。

- **Variationally Correct Operator Learning (2512.21319)** [wiki](wiki/2512.21319.md): Qiu, Dahmen, Chen, 2025.12. 用 FOSLS (一阶系统最小二乘) 构造残差损失，使损失值与真解误差在 PDE 诱导范数下严格等价（小残差⟺小误差）。提出 Reduced Basis Neural Operator (RBNO)：CNN+MLP 预测低维 RB 系数→线性解码器重构→计算 RB 残差损失。残差损失与真解误差比值接近 1。**对本实验**：为 Gate 2 validation gate 的替代方案提供最强理论保障——如果能为 FEX 的 PDE 构造变分正确的残差度量，残差本身即为误差的忠实估计，完全不需要 true_sol。FOSLS 框架直接适用于椭圆型 PDE（包括 Helmholtz）。

### RL 遗忘/保持机制续篇（服务 must-fix #5: retention 理论支撑）

- **Mechanistic Origins of Catastrophic Forgetting (2605.28860)** [wiki](wiki/2605.28860.md): Rojas Nunez 等, 2026.05. 从机制可解释性角度研究 RL vs SFT 的遗忘差异。用 Differential Binary Masking 在 attention head 层面分析电路保留。核心发现：RL 保留 72.5% 基础电路，SFT 仅 59.0%；RL 是"分布式适配器"而 SFT 是"压缩器"；输出空间 KL 散度不能预测内部遗忘（RL 的输出 KL 更大但电路保留更好）。**对本实验**：(a) 为 "频率保持是主瓶颈" 叙事提供理论深化——FEX 的 risk-seeking PG 作为 RL 方法应天然保留更多搜索电路，但频率动作空间的特殊结构（多模态、稀疏 reward）可能削弱这一保护；(b) "输出 KL 不能预测内部遗忘" 提示 FEX 的 KL_from_init 可能低估了频率动作层面的真实遗忘程度，需要 action-level 电路分析。

- **Expected Return Causes Mode Collapse (2601.21669)** [wiki](wiki/2601.21669.md): Sinha, Elango, Liu, 2026.01. 证明 outcome-level mode collapse 是 expected-return maximization 的**结构性后果**，而非探索不足。在 gradient flow 下推导出概率比例由频率-优势乘积驱动的"富者愈富"正反馈，必然导致崩塌。提出 Inverse Probability Scaling (IPS)：除以 outcome 概率消除频率偏置，证明稳态解为 reward-proportional distribution。IPS-GRPO 在分子生成/推理任务上显著超越 GRPO。**对本实验**：直接解释 FEX standard 的频率丢失机制——risk-seeking PG 的 expected return maximization 结构性驱动 controller 崩塌到高概率频率子集。Oracle soft prior 的 logit bias 恰好对抗了这一结构性崩塌（相当于人工 IPS）。这为 "retention 是主瓶颈" 提供了最精确的理论框架：不是探索不足，是目标函数本身的结构性缺陷。
