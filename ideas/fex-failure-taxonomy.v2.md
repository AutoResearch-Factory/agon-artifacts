---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-failure-taxonomy/
---

- One-sentence summary: 设计一个受控诊断框架, 将 FEX/RL 符号回归搜索失败分解为表示不足、搜索不足、参数拟合短板和 reward 代理失真等可复现实验轴.
- Hypothesis: FEX 的失败不是单一 "RL 不稳定" 问题. 在固定 PDE、固定 grammar 和可控搜索预算下, 结构搜索、连续参数拟合、reward 代理和表示容量会以可分离的方式先后失效. v1 pilot 已给出初步信号: 标准 Poisson 设置 10/10 成功, 但 354/600 个 search epoch 出现低 residual 而高 true relL2 的 deceptive reward; 去掉 search 或 finetune 余量后, 6/6 stress runs 变成两类 terminal failures.
- Expected outcome: 成功时, 至少 4 类失效模式能被预注册 stress PDE 或预算干预稳定触发, 并能用结构命中时间、residual-relL2 gap、controller entropy、参数拟合曲线和 oracle gap 区分; 失败时, 一个只看这些诊断量的 holdout mode classifier macro-F1 <= 0.40, 说明这些模式在操作层面不可分离, 应把 claim 改成 "FEX failure is coupled rather than modular."
- Contribution type: diagnostic+empirical-finding
- Risk: MEDIUM
- Estimated effort:
  - Compute: 80-120 GPU-hours for 4 stress families x 3-5 PDEs x 20-50 seeds, plus oracle and random-controller controls
  - Data: available; analytic PDE stress cases are generated on the fly, no external model weights
  - Implementation: 3-4 weeks
- Novelty quick-check: Multi-Scale FEX reports frequency-separated oscillations as a failure case but does not isolate multiple failure causes. EGRL-SR diagnoses error-based ambiguity in general SR and proposes a GCRL/HER fix, but it is not a PDE residual or FEX pipeline diagnosis. Kronberger et al. quantify GP inefficiency via equality saturation, not RL controller, PDE residual, or continuous FEX fitting. A fresh 2026-06-16 arXiv sanity check found no direct "RL-SR failure taxonomy" paper beyond the already logged deep-lit items.
- Strongest objection: The work could become a Poisson-specific debugging note if the modes only appear under hand-picked stress knobs and fail to predict new PDE failures.
- Why we should do this: FEX papers report solved cases, but method development needs to know which part of the pipeline breaks first when the grammar, reward, budget, or parameter fit is stressed. More RL, bigger grammar, more collocation points, and longer finetune address different failures; this study says which fix is warranted.
- Pilot:
  - Setup: Instrumented the upstream FEX Poisson runner on GPU, logged per-epoch residual, relL2, controller probabilities, entropy, best structure, and finetune outcome; ran a 10-seed base sweep plus two 3-seed stress conditions.
  - Metric: Signal is positive if at least 3 operationally distinct failure mechanisms appear, with at least 2 becoming terminal under controlled interventions.
  - Result: Base d=10 Poisson succeeded in 10/10 seeds, yet 59% of search epochs had residual <= 1e-3 while relL2 > 0.1; stress runs produced 3/3 finetune-shortfall failures and 3/3 search-underfit failures. A v2 CUDA smoke run on seed 99 completed locally in 1.8s.
  - Signal: POSITIVE, with the scope limited to one analytic PDE until the stress suite is broadened.

- Claims and Claims matrix: Positive claim: FEX/RL-SR failures can be decomposed into at least four intervention-stable axes, and each axis has a diagnostic signature that predicts held-out stress runs. Null claim: the axes collapse under holdout evaluation, so the correct finding is that FEX failure is coupled and cannot be assigned to individual pipeline stages. Negative claim: the pilot signatures are Poisson artifacts; if they do not transfer to oscillatory, operator-missing, tree-depth, or noisy-collocation PDEs, the work should be downgraded to an internal engineering report.
- Narrative: Frame the paper as failure attribution for RL-guided symbolic regression. When a symbolic PDE solver fails, the diagnosis should say whether the grammar made the solution impossible, the controller failed to sample it, parameter fitting failed, or the reward lied.
- Experiments: Extend the pilot to four stress families: frequency-separated Helmholtz, operator-set omission, tree-depth/representation mismatch, and collocation or boundary perturbation. For each family run matched interventions plus oracle controls: oracle operator set, oracle tree depth, exhaustive small-tree search where feasible, multi-start parameter fitting, random/uniform controller, and a stronger reward or all-point satisfaction variant inspired by EGRL-SR. Report separability with pre-registered thresholds: each induced terminal mode must recur in >=20% of affected seeds, and a diagnostic classifier trained on non-label features should reach macro-F1 >=0.70 on held-out PDEs for a strong positive claim.
- Assets status: Analytic PDE data, upstream FEX code, pilot logs, and handoff notes are documented in `workspace/fex-failure-taxonomy/data/MANIFEST.md`; no external dataset or model-weight download is needed.

<review date="2026-06-16">

## Novelty

- Score: 7/10
- Closest prior work: Multi-Scale Finite Expression Method (Hardwick & Yang, 2025); EGRL-SR (Sun et al., 2026, arXiv:2601.14693); The Inefficiency of Genetic Programming for Symbolic Regression (Kronberger et al., PPSN 2024)
- Key differentiator: 现有工作各自覆盖 RL-SR 失败的单个维度 (Multi-Scale FEX 仅频率分离; EGRL-SR 仅 error ambiguity 并提供 fix; Kronberger 仅 GP 等价类低效), 本 idea 是首个提出受控多轴诊断框架以分离 FEX/RL-SR 的表示不足、搜索不足、参数拟合短板和 reward 代理失真四类失效的工作. 方法学层面借鉴了受控消融→分类的已知范式, 新颖度集中在 domain application 和实验 systematics 层面. 2026-06-16 arXiv 复核未发现直接撞击的 "RL-SR failure taxonomy" 论文; 相近的 "Diagnosing Failure Modes of Neural Operators" (2601.11428) 针对的是 FNO/DeepONet 而非 RL-SR.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v2 将 v1 中模糊的 "频率分离/参数简并/非凸 landscape" 替换为四类操作性轴 (representation/search/parameter/reward), 逻辑结构显著清晰. 但 search insufficiency 与 reward proxy distortion 的边界仍存因果重叠风险: 若 reward 欺骗 controller, 导致的究竟是 search failure 还是 reward failure? 需要在实验设计中显式区分 "有 oracle reward 时搜索是否成功" 和 "标准 reward 下搜索是否被误导". |
| Missing evidence signals | 6/10 | v2 补充了 oracle baselines (oracle operator set, oracle tree depth, exhaustive search, multi-start fitting, random controller) 和预注册阈值, 较 v1 大幅改善. 但 Poisson 单 PDE pilot 过于狭窄; 最关键缺失是 cross-PDE transfer 的证据 —— 在非 Poisson PDE 上同样的 residual-relL2 deception 是否存在、是否同样预示 terminal failure. 置信区间和多 seed 统计力未量化. |
| Narrative | 7/10 | v2 的 "failure attribution for RL-guided symbolic regression" 叙事比 v1 的 "FEX 失效分类学" 更具一般性. "the diagnosis should say whether the grammar made the solution impossible, the controller failed to sample it, parameter fitting failed, or the reward lied" 这一 punchy 定位出色. 但仍缺乏一个 top-venue 级的 hook: 预测性 failure attribution (而非事后分类) 或可作为 sharper 叙事方向. |
| Venue contribution | 6/10 | topic 未声明 target-venue. 按 FEX 系列历史 venue (JMLR/JCP) 标准, 纯诊断研究在缺乏方法论贡献或 benchmark artifact 时较难获得顶级接受. 若成功构建可复用的 stress-test PDE suite 并获得至少一个反直觉发现 (如 "reward deception 是最主导失效而非搜索不足"), 贡献等级可显著提升. 若 target venue 为 NeurIPS D&B track, 需补充 benchmark artifact. |
| Testability | 8/10 | 这是 v2 提升最大的维度. Expected outcome 包含明确的 holdout classifier macro-F1 <= 0.40 作为 NULL 判据, 各终端模式需在 >=20% affected seeds 中复现. POSITIVE 判据 macro-F1 >= 0.70 有量化标准. 但这些并非最廉价的证伪信号 —— 首个非 Poisson PDE 上的 transfer 失败即可更早证伪. 建议在实验中优先跑 cross-PDE 迁移作为 early gate. |
| Outcome realism | 7/10 | 80-120 GPU-hours 和 3-4 周实现时间 realistic. 从 Poisson 单 PDE 扩展到 4 stress families × 3-5 PDEs × 20-50 seeds 的规模合理. 真正的风险不在 compute 而在 stress PDE 设计: 能否构造出特异性触发各 failure axis 的 PDE 而不过度耦合? 这本质上是实验设计艺术, 可能需要多轮迭代. |
| Contribution type compliance | n.a. | idea types ⊆ preferred-contribution-types: n.a. topic 未声明 `preferred-contribution-types`, 跳过检查. 不计入 Overall Quality 平均. |
| Overall Quality | 7/10 | v2 在实验设计、可测试性和逻辑结构上较 v1 有显著提升. 剩余的弱点集中在: (a) 缺乏 cross-PDE transfer 的 pilot 信号; (b) 叙事从 "taxonomy product" 到 "diagnosis methodology" 的转型尚未完成; (c) venue fit 仍依赖 benchmark artifact 或反直觉发现来证明贡献等级. 方向正确且可执行, 适合继续推进. |

## Contribution Drift (n=2)

- v1 contribution types: {diagnostic, empirical-finding}
- v2 contribution types: {diagnostic, empirical-finding}
- Status: unchanged
- Hard cap triggered: no

## Prior Review Concern Resolution (v1 → v2)

| v1 Concern | Status | Notes |
|-----------|--------|-------|
| 候选失效模式存在因果重叠 | Resolved | v2 从模糊的 5-6 个模式替换为四轴操作性框架 (representation/search/parameter/reward) |
| 缺乏 oracle ablation 设计 | Resolved | v2 加入了 oracle operator set/tree depth, exhaustive search, multi-start fitting, random controller |
| NULL 场景缺乏量化判据 | Resolved | v2 设定了 holdout classifier macro-F1 <= 0.40 |
| 叙事过于 FEX 内部调试 | Partially resolved | v2 的 "failure attribution" 叙事比 v1 更一般化, 但 "taxonomy product → diagnosis methodology" 的转型未完成 |
| 5-8 种失效模式预期不现实 | Resolved | v2 收敛为 4 个 stress families 外加 oracle controls, 设计更严格 |
| Venue contribution 不足 | Partially resolved | v2 补充了 stronger reward variant (EGRL-SR inspired), 但 benchmark artifact 仍缺失 |
| 50 GPU-hours 可能不足 | Resolved | v2 提升为 80-120 GPU-hours, 更现实 |
| Alternative framing 建议未被完全采纳 | Partially resolved | v2 未完全转向 "A Controlled Experimental Framework for Diagnosing Failure Modes in RL-Guided SR", 但在叙事方向上部分对齐 |

Refiner 未对任何 v1 review concern 做 unsupported pushback. 所有未完全采纳的建议均有合理依据 (优先跑 pilot 获取信号, 而非在实验证据不足时过早锁定宏大叙事).

## Alternative Framing

当前 framing 已较锐利, 但可进一步提升. 建议: **"Oracle-Gap Decomposition for RL-Guided Symbolic Regression: Closing the gap between what the grammar can express, what the controller finds, what parameter fitting achieves, and what the reward signals"** — 将贡献从 "失败分类学" 转变为 "oracle-gap 归因法", 使 paper 不仅报告失败模式, 更提供一套可迁移到其他 RL-SR 管线的归因协议. 配合 public stress-test PDE suite, 这一 framing 显著增强 top-venue 可行性.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在设计的 FEX/RL-SR stress-test PDE suite 上, 失效可按 oracle-gap 分解为四类干预稳定的操作性轴 (表示不足/搜索不足/参数拟合短板/reward 失真), 每轴在 >=20% affected seeds 可复现, 且基于非标签特征的 holdout classifier macro-F1 >= 0.70 跨 PDE family 成立. |
| NULL | 在当前诊断协议下, 失效模式无法被 holdout classifier (macro-F1 <= 0.40) 分离, 表明 FEX 失效在操作层面是耦合的, 分治 fix 策略在当前实验条件下不成立. |
| NEGATIVE | Poisson pilot 中观察到的 deceptive reward 和 terminal failure 信号仅限 Poisson PDE, 在 oscillatory/operator-missing/tree-depth/collocation-perturbed PDE 上不迁移; 论文应降级为 internal engineering report. |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale: **Likelihood=Medium**: v2 的 pilot 提供了 POITIVE 信号 (deceptive reward 59% epochs, 两类 terminal failure 各 3/3 seeds), 实验路径清晰且成本可控. 但 top-venue 级成稿依赖多个条件成立: (a) stress PDE 设计能 cleanly isolate 各 failure axis; (b) non-Poisson transfer 成功; (c) holdout classifier 达到有意义的 macro-F1. 这些条件在首次尝试中未必全部满足. **Impact=Medium**: 成功的 failure attribution 框架将显著指导 FEX/RL-SR 方法改进路线, 对 RL 驱动 SR 社区有参考价值. 但作为纯诊断+实证贡献, 不会改变领域范式或开启新研究路线. 若附带 public stress-test PDE suite 作为 benchmark artifact 并产出一个反直觉发现 (例如 reward deception > search insufficiency 是主导失效), Impact 可升至 High.

## Overall

- Priority: Medium
- Score: 5
- Comments: v2 较 v1 在实验设计、可测试性和逻辑结构上有显著进步, pilot 信号为正. Claude 与 codex 在 Impact 维度存在 1 级分歧 (Claude: Medium, codex: High). codex 认为成功的 failure attribution protocol 将影响 RL-SR/FEX 研究线 (Impact=High); Claude 认为纯诊断贡献即使成功也只是局部推进 (Impact=Medium). 考虑到 idea 自身声明的 contribution type 为 diagnostic+empirical-finding 且无方法学创新, Claude 维持 Medium 判断. 建议: (1) 优先在 1-2 个非 Poisson PDE 上测试 cross-PDE transfer 作为 cheap early gate; (2) 若 transfer 成功, 按 Alternative Framing 方向将叙事从 taxonomy 升级为 oracle-gap decomposition methodology; (3) 考虑将 stress-test PDE suite 打包为 public benchmark artifact 以增强 venue contribution.

#if file_size_kB > 10
超长警告: 文件 ~5 KB (不含本 review 块 ~3 KB), 未超上限
#endif

</review>

## Deep Lit 2026-06-16 Update

本次 deep-lit 续扫发现以下与本 idea 直接相关的新文献:

- **ANN-PYSR (2506.17908)** [wiki](wiki/2506.17908.md): noise-robust PDE discovery pipeline (SG 平滑 → attention NN 导数估计 → PySR)。直接对应 FEX gap #7（噪声鲁棒性）——为 failure taxonomy 的 "noise sensitivity" axis 提供具体失效模式和修复基线。
- **TIRMOO (2501.01905)** [wiki](wiki/2501.01905.md): TIR 符号回归多目标过拟合防治。核心发现：Pareto-based 选择（非纯精度最大化）才能缓解小数据过拟合。为 taxonomy 的 "overfitting" axis 提供 Pareto 视角检测/修复方法。
- **MDL-GP (2605.22374)** [wiki](wiki/2605.22374.md): MDL + 多目标 GP 模型选择。五类准则比较：先 Pareto 后 MDL 后处理 > 直接 MDL 单目标。为 taxonomy 的 "model selection failure" axis 提供实证基线。
- **MDBench (2509.20529)** [wiki](wiki/2509.20529.md): 引入 equation fidelity metric 超越 NMSE——直接验证 taxonomy 核心假设：低 NMSE 可掩盖错误方程结构。为 stress-test PDE suite 设计提供 benchmark 范本。
- **DSO (2505.10762)** [wiki](wiki/2505.10762.md): Good-Structure-Bad-Score (GSBS) 现象——结构对但系数差的表达式被 PG 低估。为 taxonomy 新增候选 failure axis: "reward-deception via coefficient sensitivity"。

## Deep Lit 2026-06-16 — fex-failure-taxonomy 专项扫 (9 篇新 wiki)

以下论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-failure-taxonomy` 于 2026-06-16 专项搜索并精读后收录。

### 直接相关：诊断协议与 failure mode 框架

- **2606.01122** [wiki](wiki/2606.01122.md): R. Drissi, 2026.06. Per-Component Diagnostic Protocol for Neural HJB-PIDE Solvers. 提出五步诊断协议将神经 PDE solver 失败分解为 Hamiltonian 各分量（drift, diffusion, compensator, nonlocal integral），逐项与独立 reference 比较。核心发现：headline scalar diagnostic 通过而 operator 级错误仍存在（missing 1/2-mixture factor）。对 taxonomy 的核心价值：(a) 提供 per-component diagnostic protocol 的方法论范本——FEX failure taxonomy 可直接借鉴其 "分项审计" 范式；(b) 其 "pointwise agreement ≠ component-level correctness" 的洞察直接验证 taxonomy 的核心理念。

- **2601.11428** [wiki](wiki/2601.11428.md): Lennon Shikhman, 2026.01. Diagnosing Failure Modes of Neural Operators Across Diverse PDE Families. 在 FNO/DeepONet/CNO 三种架构 × 五类 PDE 家族上训练 750 个模型，系统评估参数/边界/分辨率/rollout/输入扰动 shift 下的鲁棒性。核心发现：strong in-distribution accuracy 不可靠地预测鲁棒性；failure patterns 依赖于架构和 PDE 家族的联合作用。对 taxonomy 的启发：(a) 其 stress-testing 方法论可直接迁移到 FEX stress-test PDE suite 设计；(b) "in-distribution accuracy ≠ robustness" 范式对应 taxonomy 中 "低 residual ≠ 低 relL2" 的 deceptive reward 现象。

### 表示不足轴 (Representation Insufficiency)

- **2604.23256** [wiki](wiki/2604.23256.md): Chakshu Gupta, Theodore J. LaGrow, 2026.04. Architecture-Induced Recoverability Bias in Differentiable Symbolic Regression. 核心发现：同一 depth-3 grammar 和 operator 下，仅改变变量 routing architecture，可表示目标的恢复率从 0/64 变到 64/64；balanced two-subtree 架构在所有配置下都失败（0/3,776 次）。直接验证 taxonomy 的 representation insuffficiency axis——grammar 可表达不等于架构可恢复，fixed architecture 引入隐式 bias 导致系统性失败。为 taxonomy 提供 "architecture as hidden failure variable" 的实证范本。

- **2508.21484** [wiki](wiki/2508.21484.md): Clémence Métayer 等, 2025.08. Data-driven discovery of digital twins in biomedical research. 系统评估 177 个方法对抗 8 维挑战（噪声/缺失/多条件/先验/隐变量/高维/导数估计/不确定性量化）的分类框架。对 taxonomy 的价值：其 8 维 challenge taxonomy 可直接适配为 FEX failure taxonomy 的结构范本。

### 参数拟合轴 (Parameter Fitting)

- **2605.23272** [wiki](wiki/2605.23272.md): Boxiao Wang 等, 2026.05, AAAI 2026. SAGE-Fit: When Good Equations Get Bad Scores. 指出 SR 的 Good-Structure-Bad-Score 瓶颈：BFGS 等局部优化器在非线性算子下陷入 poor local minima 导致正确结构被错误丢弃（Material Science 领域 lost rate 高达 65%）。三模块：Tree-Directed Variable Projection + Function-Space Farthest-Point Sampling + Projected Gauss-Newton。对 taxonomy：直接对应 parameter fitting 轴——GSBS 现象是 "参数拟合短板" 的精确刻画，其 lost rate 量化方法可融入 FEX 诊断管线。

### Reward 代理失真轴 (Reward Proxy Distortion)

- **2506.03835** [wiki](wiki/2506.03835.md): Jianyuan Yin, Qianxiao Li, 2025.06. Learning task-specific predictive models for scientific computing. 证明 MSE minimization 在训练分布上最优 ≠ 下游算法结果最优；提出 algorithm support 概念和 task-specific reweighting。对 taxonomy 的价值：为 "reward proxy distortion" 轴提供理论基础——FEX 的 PDE residual 是真实目标（找到 PDE 解）的代理，代理最优 ≠ 目标最优的 gap 是该文的数学刻画。

### 搜索不足轴相关 (Search Insufficiency)

- **2602.12846** [wiki](wiki/2602.12846.md): Zesheng Hong 等, 2026.02. Amortized Reasoning Tree Search (ARTS). 识别 RLVR 中 "Normalization Squeeze" 病理：mode-seeking policy gradient + 有限采样 → 低概率但正确的 reasoning path 被系统性压制。对 taxonomy：Normalization Squeeze 可能解释 FEX 的 search insufficiency——RL controller 可能压制了低概率但结构正确的表达式路径，导致 search 收敛到次优解。

### PDE 发现方法论补充

- **2603.22951** [wiki](wiki/2603.22951.md): Xinxin Li 等, 2026.03. Weak-PDE-Net: 可微符号网络 + NAS + 弱形式积分实现开放式 PDE 发现。对 taxonomy：其 "open-form discovery" 范式消除了 closed grammar 的表示边界——为 taxonomy 的 oracle operator set 对照提供参考。

- **2603.22380** [wiki](wiki/2603.22380.md): Xingyu Chen 等, 2026.03. Symbolic Graph Networks (SGN): 图消息传递 + PySR 蒸馏实现噪声鲁棒 PDE 发现。对 taxonomy：graph-based non-local representation + SR 两阶段管线为 FEX 的 noise sensitivity axis 提供替代方案。

- **2602.15603** [wiki](wiki/2602.15603.md): Erion Morina 等, 2026.02. Symbolic recovery of PDEs from measurement data. 用 rational symbolic networks 从 noisy/incomplete measurement 中恢复 PDE 物理律。核心理论结论：若真实律可由所选 architecture 表示，在完整无噪极限下可唯一恢复。对 taxonomy：提供 "representability → recoverability" 的理论保证框架——为 taxonomy 的 "grammar expressiveness" vs "search recoverability" 界限提供数学语言。

### 撞车风险评估 (fex-failure-taxonomy 专项)

| 该 idea 的 claim / axis | 2026-06-16 状态 | 最接近的已有工作 |
|---|---|
| ≥4 可分离 failure axes (POSITIVE) | 🟢 完全空白 | 2606.01122 (HJB-PIDE 分项诊断) + 2601.11428 (NeuralOp 压力测试) 提供方法论参考但不重叠 |
| Holdout classifier macro-F1 ≥ 0.70 | 🟢 完全空白 | 无直接竞品 |
| Representation insufficiency | 🟢 有新基线 | 2604.23256 (architecture bias 在 diff-SR 中的系统证据) |
| Search insufficiency | 🟢 有新观察 | 2602.12846 (Normalization Squeeze 可能解释) |
| Parameter fitting | 🟡 有新基线 | 2605.23272 (SAGE-Fit / GSBS lost rate 量化) |
| Reward proxy distortion | 🟢 有新理论 | 2506.03835 (task-specific supervised learning 框架) |
| Cross-PDE transfer | 🟢 完全空白 | 无 |
| NULL claim (failure is coupled) | 🟢 完全空白 | 2606.01122 发现 per-component 错误可独立隔离——间接支持 separable 假说 |
