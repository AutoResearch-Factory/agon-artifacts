---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-search-complexity/
---

- One-sentence summary: 建立 FEX discoverability factorization theorem, 把 "表达式存在" 到 "controller 找得到" 之间的难度拆成 quotient-cover、PG geometry 和实值系数可行性三项.
- Problem anchor: "FEX 是我的课题组(Haizhao Yang 组)发明的一种方法, 使用 RL 进行 Symbolic Regression, 我(Youran Sun)作为 Yang 的博后, 应该继续在这个方向上探索"; 本 idea 只研究 FEX/PDE 表达式搜索的复杂度边界, 不漂移成通用 SR benchmark 或新工程系统.
- Hypothesis: FEX 的搜索困难不能由 raw expression-tree count 单独解释. 对某些 PDE 解族, 保守等价类 quotient 已经显著压缩结构空间; 但 blind sampler 仍有 `Omega(Q/K)` 命中下界, PG controller 还需要 polynomial `kappa_PG`, 选中结构后的实值 residual feasibility 仍可能继承 ∃R 困难.
- Expected outcome: 成功版本应能证明几条边界清楚的结果. (i) Quotient-count theorem: 对显式 rewrite set `sound_ac` (add/mul commutativity only; 旧 `R_ac+c` 含 zero/one 常数合并已作废), rooted binary-composition grammar 满足 `q_0=9`, `q_{l+1}=9*(q_l^2+q_l*(q_l+1))`; Poisson `depth2_sub` 正是 `l=1`, raw `2187` -> quotient `1539`, 且 finite canonicalizer checks 全通过. (ii) Sampler lower bound: 对 `Q` 个 quotient classes 和 `K` 个 epsilon-good classes, blind class-level sampler with replacement 的期望 hit time 是 `Q/K`, no-repeat sampler 在 uniform hidden good set 下是 `(Q+1)/(K+1)`, 所以不使用 reward geometry 的 distribution-free 说法只能给 `Omega(Q/K)`. (iii) Conditional PG theorem: 若 `N_FEX(F,L,eps)=poly(d,1/eps)` 且 softmax/f-softargmax expected reward 满足 `J*-J(theta) <= kappa_PG ||grad J(theta)||^2`、方差有界, 则 structure gap 在 `poly(N_FEX,kappa_PG,1/eps)` 更新内下降. (iv) Feasibility-hardness route: bounded-ETR/ETR-INV 的 `x+y=z`、`xy=z`、box constraints 映射到 FEX residual squares; 未完成 reduction 前只称 route, 不称 ∃R-hardness. 最便宜的证伪信号是: e-graph/semantic quotient 与 FEX residual grammar 不相容, `kappa_PG` 在真实 trace 中随好类稀疏度爆炸, 或 bounded-ETR gadget 不能保持有界变量和小 FEX 子树.
- Contribution type: theory
- Risk: HIGH
- Estimated effort:
  - Compute: 20-80 GPU-hours for PG diagnostics; quotient theorem and ETR proof are mostly 0 GPU-hours
  - Data: available
  - Implementation: quotient-count theorem is weeks; semantic/e-graph quotient comparison and PG diagnostics are 1-3 months; full bounded-ETR reduction is months and may split into a theory note
- Novelty quick-check: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026), explains generic SR tractability through FPT/W-hierarchy, and EGG-SR / GSR handle equivalence or permutation-invariant representations for general SR search. They do not analyze FEX/PDE residual grammar, class-level sampler lower bounds for FEX quotient covers, expression-tree PG convergence, or real-coefficient residual feasibility. Virgolin & Pissis prove NP-hardness for SR, not ∃R-hardness of fixed-structure FEX coefficient feasibility.
- Strongest objection: v5 有了一个 proof-facing artifact, 但它仍是保守的 syntactic quotient theorem; 顶会理论稿还需要 semantic/e-graph quotient control、和 FEX controller features 相关的非空 `kappa_PG` 条件, 或完整 bounded-ETR reduction.
- Why we should do this: FEX already has approximation theory; this project explains discoverability. It tells the Yang/FEX line whether failure comes from too many distinguishable structures, bad controller geometry, or coefficient feasibility.
- Pilot:
  - Setup: Keep v3/v4 Poisson and PG-trace diagnostics; v5 adds a CUDA-run quotient certificate for `sound_ac`, finite `depth2_sub` canonicalizer checks, rooted grammar recurrence through level 4, and explicit blind-sampler lower-bound examples.
  - Metric: Require exact count agreement with the canonical certificate (`2187 -> 1539` under `sound_ac`), idempotent representatives, zero commutative swap failures, zero subtraction false equalities under nonconstant roots, and CUDA sanity on the local GPU.
  - Result: v5 certificate passes (under canonical `sound_ac`): `depth1` raw `243` -> quotient `171`; `depth2_sub` raw `2187` -> quotient `1539`. (旧 `R_ac+c` 的 `136/953` 已作废.); idempotence true; commutative swap failures `0`; subtraction false equalities under nonconstant roots `0`; representative checksum `b1e3bf9e6ce69d17ec57e6d12cc64daff450334a2d9cc1df6d172185f997ae80`. Rooted recurrence gives quotient counts `q_2=42647229`, `q_3=32738150928636999`, `q_4=19292157472071881094470124768801009` for the declared subgrammar; this is not the full FEX depth-3 controller count. CUDA sanity ran on RTX 4060 Ti.
  - Signal: POSITIVE for Layer-1 theorem artifact; WEAK for the full factorization because PG and ETR layers remain conditional.

- Claims and Claims matrix: Positive claim: "For the declared conservative FEX subgrammar, quotient counts and blind-sampler lower bounds are exact, and FEX discoverability factorizes into quotient cover, PG geometry, and coefficient feasibility." Conditional claim: "If `kappa_PG` is polynomial, low `N_FEX` gives polynomial structure-search updates." Null claim: "Small quotient cover alone does not imply fast PG; v4 softmax-bandit proxy already shows `kappa_PG` depends on good-class mass." Negative retreat: if semantic quotient or ETR reduction fails, keep the syntactic quotient theorem plus conditional PG diagnostic and explicitly drop ∃R-hardness.
- Narrative: 论文从 "FEX can represent the solution" 和 "FEX can discover the solution" 的差距切入. Quotient theorem 是第一块严格结果; PG 和 real feasibility 分开写, 避免用小 grammar count 掩盖优化困难.
- Experiments: Keep experiments diagnostic: compare `sound_ac` counts with EGG-SR/GSR-style e-graph quotienting; rerun short FEX traces with logged logits/grad norms on Poisson/Fourier/separable/nonseparable PDE families; report `kappa_PG` proxies by good-class mass; include one bounded-ETR gadget family before any hardness claim. Extra PDE families are evidence that theory objects are non-vacuous, not a benchmark contribution.
- Assets status: No external data is needed; v5 CUDA quotient certificate and prior PG/ETR diagnostics are complete, with handoff details in `workspace/fex-search-complexity/data/MANIFEST.md`.
- Review handling: I accept the main v4 criticism and add a Layer-1 proof artifact instead of another framing pass. I accept the warning that `Q_L` was too syntactic by naming the rewrite set and marking semantic/e-graph quotienting as the next step, not as solved. I accept the sampler-lower-bound criticism by specifying the blind class-level sampler and the hidden-good-set model. I accept that cover-to-PG remains unproved, so the PG theorem stays conditional; I push back only against the stronger demand that cover size should imply PG convergence, because v4 already showed the opposite. I accept that bounded-ETR is still a route, not a result, so v5 does not claim ∃R-hardness. I treat missing depth3/4 and PDE-family breadth as an in-scope diagnostic concern: v5 adds recurrence counts for the declared rooted grammar and leaves cross-family traces for proposal-stage diagnostics. Contribution drift note: v4 was `theory`; v5 remains `theory`, with no method, benchmark, diagnostic, application, or empirical-finding type added or removed.

## Deep Lit 2026-06-19 — idea-scope 新收录 (5 篇)

以下 5 篇论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-search-complexity` 于 2026-06-19 系统性搜索并精读后收录。搜索覆盖 6 axes（PG convergence for expression tree SR / equivalence class quotient counting / ETR ∃R-hardness / SR tractability-complexity / gradient domination RL convergence / RL discrete search complexity）+ B7 反向扩展 24 次。Round 2 搜索产出 0 篇新候选，文献饱和终止。

### Layer 3: ∃R-Hardness / ETR Feasibility (与 idea 的 feasibility-hardness route 直接相关)

- **PCP for ETR (2605.23517)** [wiki](wiki/2605.23517.md): Jack Stade, 2026.05. 证明 MAX-ETR-INV 存在常数 ε 使得 (1-ε)-近似是 ∃R-hard。ETR-INV 的约束恰好是 x=1, xy=1, x+y=z，变量在 [1/2,2]——与 idea v5 的 bounded-ETR 可行性路径完全对齐。核心技术：midpoint code + continuous assignment tester + constraint shrinking（从一般 C(q) 约束缩到 ETR-INV）。直接影响：(a) 即使 FEX 找到了正确结构，优化系数的"满足所有约束"仍可能是 ∃R-hard；(b) MAX-ETR-INV 的 inapproximability 意味着部分系数满足也无法高效近似；(c) 为 idea 的 bounded-ETR gadget reduction 提供了新的 gap-preserving 工具（midpoint code、assignment tester 可直接移植到 FEX residual 编码）。**对 claim (iv) 的影响：strengthens the feasibility-hardness route — MAX-ETR-INV inapproximability is a stronger result than plain ∃R-hardness, suggesting that even approximate coefficient optimization under fixed structure inherits hardness.**

- **Constrained Nonneg Gram Feasibility ∃R-Complete (2603.19976)** [wiki](wiki/2603.19976.md): A. Majumdar, 2026.03. 从 ETR-AMI 归约证明 rank-2 constrained nonneg Gram factorization 是 ∃R-complete。归约技术：anchor direction + variable-row encoding (x,1) + 内积实现加法乘法约束。**与 idea 的关系：** anchor gadget 和 variable-row encoding 是 ETR-AMI→目标问题归约的通用 pattern，可参考设计 FEX residual feasibility 的 ∃R-hardness reduction。但 FEX 的约束结构（嵌套一元/二元算子的复合函数在 PDE residual 上的平方和最小化）比 Gram 内积约束复杂得多，直接移植不可行。

- **Affine Rank Minimization ∃R-Complete (2602.14037)** [wiki](wiki/2602.14037.md): A. Majumdar, 2026.02. 从 ETR 归约证明 ARM(k) 在 fixed rank k≥3 时是 ∃R-complete。归约：ETR formula → arithmetic circuit (gate-equality normal form) → 矩阵 entry 承载 gate value，线性约束 enforce affine gates，rank-forcing gadget enforce 乘法。**Reader 发现核心 proof 存在薄弱点：** pinned gauge block 不能消除 factor gauge；乘法 gadget 的 4×4 determinant obstruction 依赖未约束的 off-diagonal zeros。**与 idea 的关系：** ETR-to-circuit 编译 + designated carriers + occurrence fan-out 的 framing 可参考，但 rank gadget 需要重证或替换——提醒 idea 在做 bounded-ETR→FEX reduction 时必须仔细处理 gadget soundness。

### Layer 2: PG Convergence Theory (与 idea 的 conditional PG theorem 直接相关)

- **Convergence and Sample Complexity under VGD for Agnostic RL (2507.04406)** [wiki](wiki/2507.04406.md): Uri Sherman, Tomer Koren, Yishay Mansour, 2025.07. 在 agnostic RL setting 下（策略类不包含最优策略），提出 VGD (variational gradient dominance) 条件——严格弱于 completeness 和 coverability——给出 SDPO、CPI (Frank-Wolfe 重释)、PMD 三种算法的收敛和采样复杂度上界。**对 idea claim (iii) 的直接影响：** (a) VGD 条件是 idea 所假设的梯度支配条件 `J*-J(θ) ≤ κ_PG ||∇J||²` 的精确形式化对应物——可直接引用为 conditional PG theorem 的理论基础；(b) agnostic setting 自然对应 FEX 的情况（S_k 不一定包含最优 PDE 解的精确表达式）；(c) 但关键迁移难点是 FEX 的离散树策略、PDE residual reward、内层 Adam/BFGS 参数优化不满足论文的 convex policy class 与 MDP occupancy 结构。**建议：** 用 VGD 作为 conditional PG theorem 的正面 reference template，同时在论文中显式讨论 FEX controller 为何不满足 convex policy class 假设——这是 "cover alone doesn't imply fast PG" 的另一个具体原因。

- **Gradient Dominance and LQR Policy Optimization (2507.10452)** [wiki](wiki/2507.10452.md): E.D. Sontag, 2025.07. 系统梳理 gradient dominance / PLI 在 policy optimization 中的作用：全局 PLI 给指数收敛，但连续时间 LQR 通常只有 saturated/semiglobal PLI，导致全局近线性、局部指数的 mixed convergence；PLI 变体对应 ISS/siISS/iISS 扰动鲁棒性。后半部分讨论线性 feedforward NN 的 imbalance invariant 在某些区域恢复 global PLI。**对 idea 的影响：** (a) saturated PLI 语言可精确描述 FEX controller 的 κ_PG 行为——当好类稀疏时 PLI 退化为 saturated 形式，导致全局收敛减速；(b) ISS 框架可用于分析 FEX controller 的梯度估计误差（采样 PDE residual 引入的方差 = 扰动）对收敛的影响；(c) overparameterized representation 改善 gradient landscape 的分析模板可启发 FEX 的 controller 设计改进。

### Layer 2 补充：Performative RL Gradient Dominance

- **Performatively Optimal Policy (2510.04430)** [wiki](wiki/2510.04430.md): Ziyi Chen, Heng Huang, 2025.10. 在 performative RL（策略改变环境动力学）中证明：负熵正则下，当 regularizer dominance 成立时，value function 满足 gradient dominance → 任何 stationary point 都是 PO。用 zeroth-order Frank-Wolfe 实现 O(ε⁻⁴ log) 收敛。**与 idea 的关系：** 有限、间接。Regularizer dominance → gradient dominance 的分析模板可作为参考，但 performative setting（策略改变转移核）与 FEX 的固定 PDE 环境不同。主要启示：(a) 负熵正则（= FEX softmax controller 的 entropy）可诱导 gradient dominance——但前提是 regularizer 足够强，这与 FEX 的 exploration rate 衰减相矛盾；(b) L0 子空间扰动 + two-point 梯度估计可作为 FEX 搜索的替代优化方案参考。

### 本次调研对 idea claims 的影响总结

| Claim | 本次新发现 | 影响 |
|-------|-----------|------|
| (i) Quotient-count theorem | 无直接新竞品 | unchanged |
| (ii) Sampler lower bound | 无直接新竞品 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **2507.04406 VGD + 2507.10452 saturated PLI 提供了理论基础和分析工具** | strengthened — VGD 是 κ_PG 假设的精确形式化对应物；saturated PLI 解释了 κ_PG 随好类稀疏度退化的机制 |
| (iv) Feasibility-hardness route (∃R) | **2605.23517 PCP for ETR 证明 MAX-ETR-INV inapproximability** | significantly strengthened — inapproximability 比 plain ∃R-hardness 更强；midpoint code + assignment tester 可作 reduction 工具；2603.19976 和 2602.14037 提供归约技术参考 |

### 撞车评估

五篇新读论文均不与 idea 的核心 claims 撞车：它们分别位于 ETR 理论、RL convergence theory、performative RL 领域，没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解。最接近的是 2507.04406 (VGD for agnostic RL)，但它不涉及离散表达式树、PDE residual reward 或 quotient counting。idea 的核心 novelty gap（FEX-specific discoverability factorization）保持完整。

## Deep Lit 2026-06-19 续扫 — Round 2 (5 篇)

以下 5 篇论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-search-complexity` 第二轮于 2026-06-19 系统性搜索并精读后收录。搜索覆盖 6 axes（PG convergence expression tree / SR sample complexity / expression enumeration counting / gradient domination PL / SR search failure / ETR hardness fitting）+ 4 组 web search + B7 反向扩展 20 次（5 papers × 4 types）。Round 2 搜索产出 0 篇新候选，文献饱和终止。

### Layer 2 扩展: PL Inequality 变体与收敛理论

- **PL inequality 变体与梯度系统收敛 (2503.23641)** [wiki](wiki/2503.23641.md): A.C.B.D. Oliveira, Leilei Cui, E.D. Sontag, 2025.03. 是 2507.10452 (已读的 Sontag talk) 的形式化论文伴侣。核心贡献：将 PL 不等式严格分为 global / semi-global / saturated comparison-function 三味，证明 CT-LQR policy optimization **不可能**满足最强 global PL——沿 high-gain curve 代价发散但梯度有界。**对 idea claim (iii) 的影响：** (a) K_SAT 下界给出 saturated PLI 区分远端近线性和局部指数收敛的精确数学工具，可直接用于描述 FEX controller 的 κ_PG 行为；(b) ISS 视角可分析 FEX 的梯度估计误差（PDE residual 采样方差 = 扰动）对收敛的影响；(c) gradient norm / cost gap 分区诊断方法论可移植到 FEX diagnostic protocol。**补充 2507.10452：** 形式化论文提供了 talk 中未包含的完整证明（特别是 CT-LQR 永远不满足 global PLI 的严格论证）。

- **Entropic Mean-Field Neural ODE 的 PL 泛型性 (2507.08486)** [wiki](wiki/2507.08486.md): Samuel Daudin, François Delarue, 2025.07. 把无限深/宽 ResNet 写成熵正则 mean-field relaxed optimal control，证明对开稠密初值集存在唯一稳定全局极小点，且在该极小点的相对熵邻域内满足以 Fisher information 表达的**局部 PL 不等式**。**对 idea 的影响：** (a) 证明了 entropy regularization + mean-field limit → PL 是 **generic** 的（开稠密集），这为 FEX softmax controller 在何种条件下可能满足 local PL 提供了正面理论参照；(b) 但关键限制是该结果仅在 mean-field (无限宽) + 连续层极限成立，FEX controller 是有限参数离散选择器——迁移需要额外假设；(c) relaxed control → Gibbs optimal policy → Fisher info PL 的证明管线可启发 idea 写 conditional PG theorem 时的技术路线。

- **Wasserstein Policy Gradient 全局收敛 (2605.26078)** [wiki](wiki/2605.26078.md): Zhao Zhu, Rui Gao, Shuang Li, 2026.05. 证明 entropy-regularized continuous-action RL 中 WPG 的全局收敛：Bellman residual 承认 statewise KL 表示 → Bellman contraction 关联 residual 到 optimality gap → resolvent identity 连接 value improvement 到 relative Fisher information → 结合 uniform LSI 得到 **distributional PL** → 几何收缩到 O(η) 偏差。**对 idea 的影响：** (a) "Bellman 结构替代 convexity 诱导 PL 几何" 是一个全新视角——FEX 的 RL 搜索也有 Bellman 结构（sequential expression tree generation），但 FEX 是离散 action 空间而非连续；(b) statewise residual-to-gap pipeline 和 resolvent identity 可作为 conditional PG theorem 的技术参考模板；(c) 不直接撞车——WPG 针对连续 action + continuous measure space，FEX controller 是 softmax over discrete tree positions。

### Layer 2 补充: Gradient Domination in Stochastic Control

- **Entropy-Regularized LQ with Multiplicative Noise (2510.02896)** [wiki](wiki/2510.02896.md): Gabriel Diaz, Lucky Li, Wenhao Zhang, 2025.10. 在无限时域离散 LQ + 乘性噪声下证明 RPG (regularized policy gradient) 在 gradient domination + almost-smoothness 下全局线性收敛；提出 zeroth-order SB-RPG 实现 model-free 高概率收敛。**对 idea 的影响：** (a) gradient domination → global convergence 的证明模板可参考，但 LQ 结构（线性动力学 + 二次代价）远比 FEX 的离散树搜索简单；(b) zeroth-order estimator (随机扰动 + 有限 rollout) 可作为 FEX 搜索替代优化方案的技术参考；(c) 不撞车——纯 LQ control theory。

### Layer 4 补充: Coefficient Fitting 优化病态

- **Complex Equation Learner (2605.03841)** [wiki](wiki/2605.03841.md): S. Garmaev, Maurice Gauché, Olga Fink, 2026.05. 把 EQL 权重扩展到复数域以绕过 division/log/sqrt 在实域的梯度相消和定义域问题。核心发现：含 poles 的 rational SR 对 EQL 来说主要是**优化几何病态**而非表达能力不足——复权重让轨迹绕过实轴退化点。在 log/sqrt/rational benchmark 上 1e-5 到 1e-10 精度超越 EQL-div 和 SINDy。**对 idea claim (iv) 的影响：** 从工程角度**验证**了实值 coefficient fitting 存在 fundamental 的优化几何障碍——这正是 idea Layer 4 (coefficient feasibility) 要从理论上刻画的对象。CEQL 提供了一个 "绕过"（复域扩展）而非 "证明困难"（∃R reduction）的角度，两者互补。可在 idea 论文 Layer 4 讨论中引用 CEQL 作为 "实域优化病态" 的具体工程证据。

### 本次调研对 idea claims 的累计影响 (两轮合计 10 篇)

| Claim | 两轮共发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | 无直接新竞品 | unchanged — FEX grammar 闭式递推仍为白地 |
| (ii) Sampler lower bound | 无直接新竞品 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **7 篇 PL/gradient domination 理论 (VGD, saturated PLI, PL variants formal, generic PL, WPG distributional PL, LQ gradient domination, performative RL)** | significantly strengthened — κ_PG 假设有 VGD (形式化对应物) + saturated PLI (退化机制) + generic PL (entropic 条件下可保证) + distributional PL (Bellman 管线) 四重理论支撑；但全部针对连续/LQ/mean-field 场景，FEX 离散树 action space 的特殊性仍是 idea 的独特贡献白地 |
| (iv) Feasibility-hardness route (∃R) | **PCP-for-ETR inapprox + nonneg Gram ∃R + ARM ∃R + CEQL 实域优化病态** | significantly strengthened — ∃R-hardness 工具箱完备 (midpoint code, anchor gadget, ETR-to-circuit); CEQL 从工程侧验证实域 coefficient fitting 有 fundamental 优化障碍 |

### 撞车评估 (两轮合计)

本轮 5 篇新读论文均不与 idea 核心 claims 撞车。它们全部位于 optimization theory (PL variants, gradient domination, Wasserstein PG) 和 gradient-based SR (CEQL) 领域，没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解。两轮累计 10 篇 idea-scope 精读后，idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**。

## Deep Lit 2026-06-19 续扫 — Round 3 & 4 (9 篇)

以下 9 篇论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-search-complexity` 第三轮于 2026-06-19 系统性搜索并精读后收录。搜索覆盖 6 axes（PG convergence expression tree SR / FEX PDE solver search complexity / SR benchmark enumeration equivalence / gradient domination PL RL / SR search failure mode / ETR hardness coefficient fitting）+ 4 组 web search + 2 组补充搜索（recent ∃R results + canonical form grammar counting）。Round 1 读 6 篇，Round 2 读 4 篇（1 篇 tex 下载失败），B7 反向扩展共 36 次调用（6×4 + 3×4）。Round 3 B7 扩展产出 0 篇高相关新候选，文献饱和终止。

### Layer 3 扩展: ∃R-Hardness 新结果

- **Hilbert's Nullstellensatz ≡ ∃R: 优化与代数问题完备性 (2510.19704)** [wiki](wiki/2510.19704.md): Markus Bläser, Sagnik Dutta, Gorav Jindal (Saarland/MPI/Warsaw), 2025.10. 证明 Affine Polynomial Projection 对任意域 HN-hard，对 R/C HN-complete；SparseShift 对无限域 HN-hard。**关键结果（对 idea claim (iv) 最直接）：** biquadratic form 非负性、四次多项式凸性、四次 hyperbolicity、四次实稳定性均为 co-∃R-complete（universal theory of the reals 的完备问题）。**对 idea 的影响：** (a) 四次凸性是 UTR-complete 意味着判定 FEX coefficient optimization landscape 是否凸在理论上已是 co-∃R-hard——即使 FEX 找到正确结构，验证系数优化景观的凸性本身就 computationally intractable；(b) 从 real stability 的 co-∃R-completeness 可推出 FEX 内层 Adam/BFGS 优化器的收敛性质取决于 ∃R-hard 条件；(c) 与 PCP-for-ETR (2605.23517) 互补——2605.23517 给 approximate feasibility 的 inapproximability，本文给 landscape property verification 的 hardness。

- **∃R-Completeness of Tensor Degeneracy (2604.17061)** [wiki](wiki/2604.17061.md): Angshul Majumdar, 2026.04. 证明 real 3-tensor degeneracy 是 ∃R-complete。纯代数归约链：homogeneous quadratic feasibility → projective bilinear feasibility → singular bilinear pencil feasibility → tensor degeneracy。**无组合 gadget**——与 2602.14037 (ARM) 和 2603.19976 (nonneg Gram) 的 gadget-heavy 归约形成方法论对比。**对 idea 的影响：** (a) 纯代数归约的 bilinearization 技术可作为 FEX residual feasibility reduction 的替代路径——如果 FEX 的嵌套一元/二元算子可以先双线性化，则归约可能比 ETR-to-circuit 更直接；(b) boundary-format lifting + completion polynomial 的框架可参考设计 FEX ∃R 证明的 soundness certification；(c) 但 tensor degeneracy 的约束结构（三线性形式的退化）与 FEX residual（嵌套函数复合）差异较大，直接移植需要额外的结构桥接。

- **Continuous Clustering ∃R-Complete (2604.26972)** [wiki](wiki/2604.26972.md): Angshul Majumdar, 2026.04. 证明 polynomial density 上的 CMRC（mutual reachability clustering）和 VSC（valley-separated clustering）是 ∃R-complete。归约从 RealFeas 构造 two-sheet / circle-product 密度函数。**对 idea 的影响：** (a) polynomial density level set 的连通分支计数和洞检测至少 ∃R-hard（completeness open）——如果 FEX 的搜索空间可以编码为 polynomial density landscape 上的 clustering 问题，则搜索的 tractability 问题继承 ∃R-hardness；(b) 但 idea 的 quotient-count theorem 是 combinatorial counting（离散等价类），不是 semi-algebraic level set 的拓扑问题，所以不直接冲突——两者互补地覆盖 "counting" 在不同数学域的 hardness。

- **ETR Enriched with Integer Powers (2502.02220)** [wiki](wiki/2502.02220.md): Jorge Gallego-Hernández, Alessio Mansutti (IMDEA), 2025.02. 研究 ∃R(r^Z) 的可判定性和复杂度：自然数基底 NEXPTIME，代数基底 EXPSPACE，π/e 等超越基底 3EXP。**对 idea 的影响：** (a) FEX 的算子集包含 exp/log/sin 等超越函数，coefficient feasibility 问题涉及 real transcendental arithmetic——本文的结果划定了当系数取值涉及 exp/log 等超越函数时，ETR-based reasoning 的 decidability 边界（EXPSPACE/3EXP）；(b) polynomial root barrier 概念可用于分析 FEX 系数优化中何时能高效判定符号判断（sign evaluation）——这是 Adam/BFGS 收敛分析的底层问题；(c) 去除 Schanuel 猜想的 decidability 证明是技术进步，但与 FEX 的具体归约路径距离较远。

### Layer 2 扩展: Actor-Critic Sample Complexity 最优结果

- **Single-Timescale Actor-Critic O(ε⁻³) (2410.08868)** [wiki](wiki/2410.08868.md): Navdeep Kumar, Priyank Agrawal, Giorgia Ramponi, Kfir Y. Levy, Shie Mannor (Technion/Columbia/Zurich), 2024.10. 证明 tabular softmax vanilla AC 在 η_k,β_k=O(k^{-2/3}) 下达到 J*-EJ^π_k=O(k^{-1/3})，即 O(ε⁻³) global sample complexity。核心技术：actor sub-optimality 表为 critic bias + critic variance + actor optimization error，每项在 coupled recursion 下用 Lyapunov 方法递推衰减。**对 idea claim (iii) 的影响：** (a) 提供了 gradient domination lemma 在 tabular softmax PG 中的具体应用范本——vanilla AC 本身可对应 FEX 的 controller+evaluator 双循环；(b) O(k^{-2/3}) step size 偏离标准 O(k^{-1/2}) 是非平凡发现——如果 FEX controller 也用 softmax + PDE residual critic，类似的 step size 调整可能是必需的；(c) 但 FEX 的 action space 是离散表达式树的 sequential construction，不是 tabular softmax over finite actions——迁移需要处理 variable-length action sequence 和 tree-structured reward。

- **O(ε⁻²) Single-Loop Actor-Critic under Minimal Assumptions (2605.13639)** [wiki](wiki/2605.13639.md): Ishaq Hamza (IISc), Zaiwei Chen (Purdue), 2026.05. 在单循环、单时间尺度、off-policy AC 框架下，under minimal assumptions（仅需存在一个诱导 irreducible Markov chain 的策略），证明 IS critic 和 ETD critic 均达到 O(ε⁻²) last-iterate sample complexity。核心技术：coupled Lyapunov drift framework，actor 几何收敛 + critic O(1/T) 收敛 + cross-domination property。**对 idea claim (iii) 的影响：** (a) O(ε⁻²) 是 tabular AC 的**最优** sample complexity，直接改进 2410.08868 的 O(ε⁻³)——如果 FEX controller 可映射到 tabular AC，则搜索的 sample complexity 下界已确定；(b) minimal assumptions（不需 uniform mixing/exploration）更接近 FEX 的实际情况——FEX controller 可能不满足 uniform exploration 条件；(c) 但关键差距仍在：FEX 的 sequential tree construction 不是 stationary MDP，且 PDE residual reward 的评估本身有计算开销。

- **Refined Analysis of Entropy-Regularized AC (2605.24357)** [wiki](wiki/2605.24357.md): Safwan Labbi, Paul Mangold, Daniil Tiapkin, Eric Moulines (CMAP/Ecole Polytechnique/Google DeepMind/MBZUAI), 2026.05, ICML 2026. 分析 entropy-regularized tabular AC 中 critic 的作用。核心发现：exact critic 下，actor 的随机梯度方差可由确定性梯度范数控制（"strong variance reduction"），接近最优时噪声消失 → O(log(1/ε)) 复杂度；online critic 下，残余误差主要来自 critic bias/variance。**对 idea claim (iii) 的影响：** (a) strong variance reduction 结果意味着当 FEX controller 接近最优策略时，PG 估计自然变得更精确——这为 κ_PG 在好类附近的行为提供了正面理论支持（κ_PG 在最优附近可能很小）；(b) 但 FEX 使用 Adam/BFGS 作为 "critic"（评估 PDE residual），这不是标准的 TD learning——"strong variance reduction" 是否在 FEX 的 inner-loop 优化中成立是一个开放问题；(c) 桥接了 2601.12604 (f-softargmax) 和 2410.08868 (vanilla AC) 的分析框架，提供了更统一的理论视角。

### Layer 2 补充: Operator-Theoretic PG Framework

- **Operator-Theoretic PG for General MDPs (2603.17875)** [wiki](wiki/2603.17875.md): Abhishek Gupta, Aditya Mahajan (Ohio State/McGill), 2026.03. 将 general MDP with unbounded costs 写成 transition kernel 与 policy operator 在 weighted function spaces 上的优化。核心贡献：(a) 新的最优策略存在性定理（基于 compact sublevel sets on weighted function spaces，不同于经典 Feinberg-Shwartz 条件）；(b) general MDP 的 policy difference lemma 和 Gateaux derivative；(c) IPM (integral probability metric) 上界 → majorization-minimization PG 算法；(d) MMD 作为 IPM 的 MM-RKHS 算法在 finite MDP 上优于 PPO。**对 idea claim (iii) 的影响：** (a) perturbation theory of linear operators 可为 FEX controller 的策略变化提供 operator-norm bound——当 controller 参数微调时，transition kernel 的变化被 operator norm 控制；(b) IPM-based policy difference lemma 提供了比 standard KL-based 方法更一般的 trust region——如果 FEX 搜索使用 non-softmax parameterization，IPM 方法可能更自然；(c) 但 general MDP 框架的代价是具体常数不如 tabular 结果锐利。

### Layer 1 补充: 函数空间穷举与 Counting

- **Exhaustive Symbolic Integration (2605.04978)** [wiki](wiki/2605.04978.md): Harry Desmond (Portsmouth), 2026.05. 穷举给定 operator basis 和 complexity 上限内的所有表达式，计算 integrability fraction ρ(k)。核心发现：log 显著提高 bounded grammar 的积分闭合性 (~3x)；operator basis 对 integrability landscape 影响极大。**对 idea claim (i) 的影响：** (a) ESI 的穷举方法提供了 FEX quotient-count theorem 的经验对照——可以在相同 operator basis 下，比较 ESI 的 raw expression count 与 sound_ac quotient count，验证 quotient compression ratio；(b) integrability fraction 概念可类比 FEX 的 "discoverability fraction"——给定 operator basis 和 depth，多大比例的 PDE 解可被 S_k 表示且 controller 可发现？(c) numerical fingerprinting 技术可用于 FEX 的 semantic equivalence testing，补充 sound_ac 的 syntactic quotient。但 ESI 面向 symbolic integration 而非 PDE solving，方法论距离较远。

### 本次调研对 idea claims 的累计影响 (三轮合计 19 篇)

| Claim | 三轮共发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | ESI (2605.04978) 提供穷举 counting 的经验对照和 numerical fingerprinting 工具 | marginally strengthened — 可用于 empirical validation，但不改变理论贡献 |
| (ii) Sampler lower bound | 无直接新竞品 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+4 篇 AC convergence 理论：O(ε⁻³) AC (2410.08868)、O(ε⁻²) optimal AC (2605.13639)、entropy-reg AC strong variance reduction (2605.24357)、operator-theoretic PG (2603.17875)** | further strengthened — 现在有 tabular AC 从 O(ε⁻⁴) → O(ε⁻³) → O(ε⁻²) 的完整 sample complexity 进化链，minimal assumptions 版本，以及 strong variance reduction 在最优附近的正面结果；但全部针对 tabular/finite MDP，FEX 离散树 action space 的特殊性仍是 idea 的独特贡献白地 |
| (iv) Feasibility-hardness route (∃R) | **+4 篇 ∃R 理论：四次凸性 co-∃R-complete (2510.19704)、tensor degeneracy ∃R-complete 纯代数归约 (2604.17061)、continuous clustering ∃R-complete (2604.26972)、ETR+integer powers decidability (2502.02220)** | significantly strengthened — ∃R 工具箱大幅扩充：(a) landscape property verification (凸性/稳定性) 本身是 co-∃R-hard 的新发现; (b) 纯代数归约 (bilinearization) 提供 gadget-free 替代路径; (c) 超越函数系数的 decidability 边界已划定 (EXPSPACE/3EXP) |

### 撞车评估 (三轮合计)

本轮 9 篇新读论文均不与 idea 核心 claims 撞车。它们分别位于 ∃R-hardness 理论（4 篇）、PG/AC convergence theory（4 篇）和函数穷举 counting（1 篇）领域。没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解、quotient-count theorem、或 FEX-specific PG convergence。三轮累计 19 篇 idea-scope 精读后（加上之前 topic-scope 读的 ~160 篇），idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**，且工具箱（∃R reduction techniques + PG convergence templates + exhaustive counting）已充分完备。

## Deep Lit 2026-06-21 实验 scope — reviewer 反馈后文献补给 (7 篇)

以下 7 篇论文由 `deep-lit-tick --scope experiment fex-search-complexity` 于 2026-06-21 系统性搜索并精读后收录。搜索由 reviewer 的 6 个 must-fix 项驱动（score 4.5/10, NOT READY），聚焦 G3 bridge 形式化 (must-fix #2)、e-graph containment 形式化 (must-fix #4)、diagnostic-to-theory bridge (must-fix #6)。6 axes 搜索 + 5 组 web search + B7 反向扩展 28 次（7 papers × 4 types）。Round 2 搜索产出 0 篇新候选，文献饱和终止。

### G3 Bridge 形式化工具箱 (5 篇)

- **Softmax Policy Mirror Ascent (2411.12042)** [wiki](wiki/2411.12042.md): Asad, Babanezhad, Laradji, Le Roux, Vaswani, 2024.11. 提出 SPMA：在 logits 对偶空间做 mirror ascent（log-sum-exp mirror map），避免 NPG 的动作空间归一化。Tabular 下用常数步长达到**线性收敛**，匹配 NPG 且优于加速 softmax PG。扩展到 log-linear FA 不需要 compatible FA（NPG 需要）。MuJoCo/Atari 上与 PPO/TRPO/MDPO 持平或更优。**对 claim (iii) 的影响：** (a) SPMA 的 logit-space mirror map 可直接类比 FEX controller 的 softmax 参数化——FEX 的 κ_PG 估计可在 logit space 而非 probability space 做，可能改善几何；(b) 不需 compatible FA 意味着 FEX 的非标准 controller 结构也有理论适用的可能；(c) 但 SPMA 假设 tabular/log-linear，FEX 是 sequential tree action，迁移仍有 gap。**可直接写进 conditional PG theorem 的替代理论模板。**

- **CVaR-VaR Policy Gradient (2601.22100)** [wiki](wiki/2601.22100.md): Luo, Delage, 2026.01. 提出将 VaR (quantile) 策略梯度与 CVaR 策略梯度融合以提升 risk-averse RL 样本效率。核心洞察：CVaR = 尾部 VaR 的期望，VaR-PG 作为辅助项自然融入。推导了 VaR Bellman 最优算子并证明 contraction。**对 claim (iii) 的影响：** FEX 使用 risk-seeking（top-quantile）PG，本文提供了 quantile-based PG 的 Bellman 理论基础——如果 FEX 的 conditional PG theorem 采用 quantile formulation（而非 expected reward），CVaR-VaR 的 contraction 证明模板可直接参考。但 FEX 是 risk-seeking（top quantile）而本文是 risk-averse（bottom quantile），direction 相反。

- **Risk-Sensitive Distributional RL Policy Gradient (2405.14749)** [wiki](wiki/2405.14749.md): Xiao, Yu, Ying, 2024.05. 提出首个分布策略梯度定理（Distributional PG Theorem）：将经典 PG 从标量期望推广到概率测度空间，给出概率测度梯度的解析形式。设计 CDPG 算法：categorical 分布近似 + projected distributional Bellman (√γ-contraction in Cramér distance)。有限支持最优性 N=O(ε⁻²) + 有限时间局部收敛。**对 claim (iii) 的影响：** 误差传播分析模板（distribution estimation error → gradient error → convergence）可作为 conditional PG theorem 的技术写作参考，但 problem domain 差异大，不建议作 main reference。

- **PPO-Clip 非渐进全局收敛 (2512.16565)** [wiki](wiki/2512.16565.md): Liu, Dai, Zhang 等, 2025.12. 首次为 actor-only PPO-Clip 在 f-divergence 正则化下建立非渐进全局收敛理论。核心贡献：(a) f-divergence 正则化值函数满足 non-uniform Łojasiewicz inequality；(b) forward KL 正则化下全局线性收敛；(c) reverse KL 下稳态收敛 + 局部线性收敛。**对 claim (iii) 的影响：** (a) f-divergence 框架是 idea 的 conditional PG theorem 最接近的理论外壳——FEX 的 softmax controller 自带 entropy 正则化（= forward KL）；(b) Łojasiewicz inequality 是 idea 的 κ_PG 梯度支配假设的精确数学工具；(c) 但 PPO-Clip 的双层 clipping 机制不存在于 FEX 的 vanilla PG controller 中。**reviewer must-fix #2 的核心工具之一。**

- **Reusing Trajectories in PG, O(ε⁻¹) (2506.06178)** [wiki](wiki/2506.06178.md): Montenegro, Mansutti, Mussi, Papini, Metelli, 2025.06. 提出 RT-PG：用 Power Mean 修正的 Multiple Importance Weighting 复用历史轨迹。证明 sample complexity Õ(ε⁻²ω⁻¹)，复用全部历史时达 Õ(ε⁻¹)——文献最优。**对 claim (iii) 的影响：** 如果 FEX controller 搜索也能利用历史轨迹（已探索表达式树的 reward 信息），RT-PG 的理论框架可给出 sample complexity 的改进上界。但 FEX 的 action space 是 tree-structured sequential construction，off-policy correction 的 importance weight 计算可能不可行。主要作为 PG sample complexity 最新 baseline 引用。

### E-graph 形式理论 (1 篇)

- **Semantic Foundations of Equality Saturation (2501.02413)** [wiki](wiki/2501.02413.md): Suciu, Wang, Zhang, 2025.01, ICDT. E-graph 的首个形式语义基础。核心贡献：(a) E-graph 定义为可达确定性树自动机，语义为 Partial Congruence Relation；(b) Immediate Consequence Operator 将 equality saturation 重述为 fixpoint computation；(c) 证明 equality saturation ≡ chase（Skolem chase）的对应关系；(d) 三层终止复杂度：single-instance PSPACE-hard, all-term-instance 2EXPTIME-complete, all-E-graph-instance NP-hard；(e) 基于无环性的终止充分条件。**对 reviewer must-fix #4 的直接影响：** (a) tree automata → PCR 的对应为 idea 的 containment/gap theorem 提供了精确的形式语言——FEX 的 sound_ac rewrite 可编码为 TRS，其 e-graph 的 PCR 对应 containment，gap 对应 automata 的不可达 classes；(b) 终止条件的无环性判据可直接应用于 sound_ac（仅 add/mul commutativity，无递归规则）；(c) 但本文面向程序优化的 e-graph（任意 TRS），FEX 的 e-graph 是受限的表达式树 grammar——需要利用 FEX grammar 的有限性简化证明。**reviewer must-fix #4 的核心工具。**

### Diagnostic-to-Theory Bridge (1 篇)

- **K-PL 条件与随机梯度鲁棒性 (2509.24277)** [wiki](wiki/2509.24277.md): Cui, Jiang, Sontag, 2025.09. 提出 small-covariance NSS（噪声到状态稳定性）和广义 K-PL 条件。核心发现：K-PL + globally Lipschitz gradient → stochastic gradient dynamics 收敛到最优的邻域（邻域大小由噪声协方差决定）。K∞-PL 给 NSS，positive-definite-PL 给 integral NSS。应用到 LQR policy optimization 和 logistic regression。**对 reviewer must-fix #2 + #6 的影响：** (a) K-PL 条件是 idea 的 κ_PG 假设的最一般化形式——比 standard PL 弱，允许 κ_PG 依赖 state 或 cost gap，精确匹配 FEX controller 的 κ_PG 行为（远离最优时 κ_PG 大，接近时小）；(b) NSS 框架精确刻画 FEX 的 diagnostic-to-theory bridge：ĥat{kappa} 的 stochastic estimation noise = perturbation，K-PL 保证即使 noise 不消失也能收敛到邻域；(c) 是 2503.23641 (PLI variants formal) 和 2507.10452 (saturated PLI talk) 的自然延伸，完成了 Sontag 组 PLI→ISS→NSS 的三角理论链。**reviewer must-fix #6 的核心工具。**

### 本次调研对 idea claims 的累计影响 (实验 scope 7 篇，总计 26 篇)

| Claim | 本次新发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | 无 | unchanged |
| (ii) Sampler lower bound | 无 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+5 篇：SPMA logit-space mirror (2411.12042)、CVaR-VaR quantile PG (2601.22100)、distributional PG theorem (2405.14749)、PPO-Clip PL (2512.16565)、RT-PG O(ε⁻¹) (2506.06178)** | further strengthened — 现有 (a) logit-space mirror descent 线性收敛（无需 compatible FA）；(b) f-divergence→PL 框架（PPO-Clip）；(c) quantile-based PG 的 Bellman 理论；(d) 文献最优 O(ε⁻¹) 样本复杂度。全部针对 tabular/log-linear/continuous MDP，FEX 离散树 action space 仍是唯一白地。 |
| (iv) Feasibility-hardness route (∃R) | 无 | unchanged |
| (v) Diagnostic bridge (κ_PG 估计→收敛保证) | **+2 篇：semantic foundations EqSat (2501.02413)、K-PL NSS (2509.24277)** | new axis — K-PL + NSS 精确刻画 ĥat{kappa} 估计→收敛的 formal bridge；e-graph tree automata 提供 containment/gap 形式化工具 |

### 撞车评估

本次 7 篇新读论文均不与 idea 核心 claims 撞车。它们分别位于 PG convergence theory（5 篇）、e-graph formal semantics（1 篇）和 stochastic gradient robustness（1 篇）领域。没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解。四轮累计 26 篇（19 idea-scope + 7 experiment-scope）精读后，idea 的核心 novelty gap（FEX-specific discoverability factorization）**完整保持**。

## Deep Lit 2026-06-21 实验 scope — reviewer 5.6 后 G3 FEX-specificity 文献补给 (13 篇)

以下 13 篇论文由 `deep-lit-tick --scope experiment fex-search-complexity` 于 2026-06-21 系统性搜索并精读后收录。搜索由 reviewer score 5.6 must-fix #1 (GAP-1.3 G3 FEX-specificity) 驱动，聚焦 softmax PG convergence theory（bandit/MDP/general spaces）、representation-dependent convergence conditions、undiscounted γ=1 PG theory、PG failure modes in discrete action spaces。8 axes 搜索 + 7 组 web search + B7 反向扩展 52 次（13 papers × 4 types）。Round 3 搜索产出 0 篇新候选，文献饱和终止。

### Softmax PG 收敛理论核心工具箱 (6 篇)

- **Global linear convergence of entropy-regularized softmax PG beyond tabular (2605.24939)** [wiki](wiki/2605.24939.md): Ziyue Chen, Šivška, Szpruch, 2026.06. 在连续状态/动作空间中，log-linear softmax PG 在 entropy regularization 下实现全局线性收敛。核心假设 Q^π_τ-realizability；关键技术是 non-uniform PL inequality + KL regularizer 作为 Lyapunov function 保证 coercivity。两种 feature regime（full-affine-span / simplex-valued）下 Fisher information 最小特征值保持正下界。**对 G3 的影响：** 直接提供了 softmax PG 在非 tabular setting 下的 PL inequality——如果 FEX controller 的特征表示满足 full-affine-span 或 simplex-valued 条件，G3 的 κ_PG 假设有精确理论支撑。(已读，非本轮新 wiki)

- **Convergence of actor-critic gradient flow for entropy regularised MDPs in general spaces (2510.14898)** [wiki](wiki/2510.14898.md): Denis Zorba, Šivška, Szpruch, 2025.10. 证明连续时间 actor-critic 梯度流在 Polish 状态-动作空间中的稳定性和全局收敛。核心发现：timescale separation 是稳定性和收敛的关键。相对熵正则化器在一般动作空间中无界，需先证明不会 finite-time blow-up。**对 G3 的影响：** 为 FEX 的 controller+evaluator 双循环提供了 actor-critic 收敛框架。

- **Mirror descent actor-critic for entropy regularised MDPs in general spaces (2602.10838)** [wiki](wiki/2602.10838.md): Zorba, Šivška, Szpruch, 2026.02. 将 2510.14898 扩展到离散时间：single-loop AC 需要满足特定稳定性条件；multi-TD-step 变体通过增加 critic 更新次数放宽条件，给出 TD steps 下界。Sub-linear convergence (TD steps ~ log) 和 linear convergence (TD steps ~ linear + concentrability)。**对 G3 的影响：** 给出了实际可实现的离散时间 AC 收敛速率，可作为 FEX controller 收敛分析的技术模板。

- **Ordering-based Conditions for Global Convergence of PG (2504.02130)** [wiki](wiki/2504.02130.md): Jincheng Mei, Bo Dai, Alekh Agarwal (Google DeepMind), NeurIPS 2023, 2025.04. 证明 softmax PG 全局收敛不取决于 approximation error，而取决于 representation 是否保持 reward ranking（non-domination condition + reward order preservation）。NPG 全局收敛 iff reward projection 保持最优动作 rank。5 个 counter-examples 展示相同近似误差下收敛/发散的差异。**对 G3 的直接影响：** (a) 为 G3 提供了精确的"为什么 FEX controller 的 softmax PG 可能失败"的理论工具——不是因为表达式空间太大，而是因为 representation 的 ordering properties；(b) 可直接用于诊断 FEX 的 controller feature 是否满足全局收敛条件。

- **A Lyapunov Analysis of Softmax PG for Stochastic Bandits (2603.26547)** [wiki](wiki/2603.26547.md): Tor Lattimore (Google DeepMind), 2026.03. 将连续时间 softmax PG 的 Lyapunov 分析适配到离散时间 k-armed bandit。核心结果：学习率 η ≤ Δ_min²/(120 Δ_max log(nk)) 时，regret O(k log(k) log(n) / η)。gap-dependent 学习率选择至关重要。**对 G3 的直接影响：** (a) 2-arm softmax witness 的 κ_PG ~ 1/Δ 行为可直接从 Lattimore 的 Lyapunov 分析推导——当 Δ→0 时学习率必须 O(Δ²) 才能保证收敛，regret 爆炸；(b) 为 G3 separation 提供了严格的 gap-dependent convergence 工具。

- **Small steps no more: Global convergence of stochastic gradient bandits for any learning rate (2502.07141)** [wiki](wiki/2502.07141.md): Mei, Dai, Agarwal, Vaswani, Raj, Szepesvári, Schuurmans (Google DeepMind), NeurIPS 2024, 2025.02. 证明 softmax PG bandit 用任意常数学习率 η>0 几乎必然收敛到全局最优。核心技术：progress-noise decomposition + Borel-Cantelli exploration guarantee。**对 G3 的影响：** 建立了 softmax PG 在 bandit 层面的 strong baseline——收敛不是问题，问题是收敛速率，而速率取决于 gradient geometry (κ_PG)。

### Undiscounted γ=1 PG Theory (2 篇)

- **Why PG Algorithms Work for Undiscounted Total-Reward MDPs (2510.18340)** [wiki](wiki/2510.18340.md): Jongmin Lee, Ernest K. Ryu (UCLA), 2025.10. 首次建立 γ=1 undiscounted total-reward MDP 的 PG 收敛理论。核心创新：recurrent-transient 状态分类对 Π+（严格正策略）不变 + transient visitation measure 替代 ill-defined 的 state visitation measure。Softmax PG 在该 setting 下实现线性收敛。**对 G3 的直接影响：** FEX controller 使用 γ=1 undiscounted reward——此前所有 PG 收敛理论都假设 γ<1，本文填补了这个空白。G3 的 conditional PG theorem 现在可以在 undiscounted 框架下精确陈述。

- **PG Algorithms in Average-Reward Multichain MDPs (2602.18003)** [wiki](wiki/2602.18003.md): Lee, Ryu, 2026.02. 扩展 2510.18340 到 average-reward multichain MDP（多个 recurrent class + transient states）。α-clipped policy mirror ascent 达到 ε-optimal。**对 G3 的影响：** FEX 的 sequential tree construction 可能产生 multichain 结构（某些表达式前缀是 absorbing），此理论为该场景提供收敛保证。

### Softmax PG Failure Modes & Dynamics (3 篇)

- **Overcoming Valid Action Suppression in Unmasked PG (2603.09090)** [wiki](wiki/2603.09090.md): Zabounidis, Siegelmann 等 (UMass), 2026.03. 揭示 softmax PG 的一个关键失败模式：当某 action 在已访问 state 无效时，gradient 通过 shared prefinal features 传播到未访问 state 使该 action 在那里也被 suppress——即使它在那里是 valid 的。证明了 exponential decay bound: π(a|s*) ≤ softmax(...) → exponentially small。**对 G3 的直接影响：** FEX 的 sequential tree construction 中，不同 depth 的 slot 共享 controller network features。Valid action suppression 可能是 FEX controller PG difficulty 的一个具体机制——某个在 depth-1 无效的算子（因 grammar constraint）通过 feature sharing 在 depth-2 也被 suppress，尽管它在那里是 valid 的。

- **Logit Dynamics in Softmax PG Methods (2506.12912)** [wiki](wiki/2506.12912.md): Yingru Li, 2025.06. 推导 softmax PG logit update 的 L2 范数解析公式：‖Δz‖₂ ∝ η|A|√(1-2P_c + C(P))，其中 C(P)=ΣP_a² 是碰撞概率。揭示自调控机制：高置信度 (P_c≈1) 更新极小，低置信度更新大。**对 G3 的影响：** 碰撞概率公式可直接用于分析 FEX controller 在不同训练阶段的 gradient magnitude——当 policy 锁定到少数候选表达式时（Q/K 大），C(P)≈1/~400，更新强度有明确的量化表达。

- **Vanishing L2 regularization for softmax MAB (2605.03752)** [wiki](wiki/2605.03752.md): Aniţa, Turinici, 2026.05. 分析 L2 正则化 softmax PG 在正则化参数 γ→0 时的收敛——这是收敛到真正最优解的必要条件。证明 gradient norm 的 liminf=0 + 引入 Hyp-ργ 假设连接学习率与正则化衰减。**对 G3 的影响：** FEX 的 entropy annealing（逐步降低 exploration）对应正则化衰减——本文为该过程的收敛性提供了理论工具。

### Risk-Sensitive & Structured MDP (2 篇)

- **Softmax gradient policy for variance minimization MAB (2604.00241)** [wiki](wiki/2604.00241.md): Gabriel Turinici, 2026.04. 用 softmax PG 求解方差最小化 MAB（而非 reward 最大化），用两个独立采样构造无偏方差估计。证明几乎必然收敛。**对 G3 的影响：** 方差目标的 softmax PG 分析模板可参考——FEX 的 hat{kappa} 估计涉及 reward variance，本文的 dual-sample 技术可用于诊断。

- **Lyapunov-Based Sample Complexity for Weakly-Coupled MDPs (2606.14095)** [wiki](wiki/2606.14095.md): Wu, Zurek, Wang (CMU/Penn State), 2026.06. 弱耦合 MDP 和 Restless Bandits 在 generative model 下的样本复杂度 polynomial in N（臂数）。Lyapunov drift transfer technique + LP relaxation perturbation analysis。**对 G3 的影响：** 如果 FEX 的 4-slot sequential tree construction 可建模为 weakly-coupled MDP（各 slot 弱耦合），则 polynomial sample complexity 成立——这是 G3 从 bandit 扩展到 structured MDP 的可能路径。

- **Risk-sensitive RL Based on Convex Scoring Functions (2505.04553)** [wiki](wiki/2505.04553.md): Han, Liu, Yu, 2025.05. 用凸评分函数统一多种风险度量（Expected Shortfall、entropic VaR、mean-risk utility），通过增广状态空间解决时间不一致问题。customized Actor-Critic 有理论近似保证，不要求 MDP 连续。**对 G3 的影响：** 如果 FEX 的 risk-seeking PG (top-quantile) 可表为凸评分函数的 dual 形式，本文的 Actor-Critic 框架可作为 conditional PG theorem 的替代证明模板。

### 本次调研对 claims 的累计影响 (五轮合计 39 篇)

| Claim | 本次新发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | 无 | unchanged |
| (ii) Sampler lower bound | 无 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+13 篇 softmax PG 收敛理论：PL beyond tabular (2605.24939)、AC general spaces (2510.14898, 2602.10838)、ordering conditions (2504.02130)、Lyapunov softmax bandit (2603.26547)、any-LR convergence (2502.07141)、undiscounted γ=1 (2510.18340, 2602.18003)、valid action suppression failure (2603.09090)、logit dynamics (2506.12912)、L2 reg decay (2605.03752)、variance MAB (2604.00241)、weakly-coupled MDP (2606.14095)、risk-sensitive scoring (2505.04553)** | **massively strengthened** — G3 理论工具箱从 7 篇 PG convergence papers 扩展到 20 篇。新增四个关键维度：(a) undiscounted γ=1 PG theory（FEX 实际 setting）；(b) ordering/representation-dependent convergence conditions（不是近似误差决定收敛）；(c) softmax PG 的具体 failure modes（valid action suppression via feature sharing）；(d) gap-dependent Lyapunov analysis（精确刻画 κ_PG ~ 1/Δ 行为）。但全部针对 tabular/log-linear/bandit/continuous MDP，FEX 的 4-slot sequential tree action space 仍是理论白地。 |
| (iv) Feasibility-hardness route (∃R) | 无 | unchanged |
| (v) Diagnostic bridge (κ_PG→convergence) | **+2 篇：logit dynamics 碰撞概率公式 (2506.12912)、valid action suppression 指数衰减 bound (2603.09090)** | strengthened — 具体的 softmax PG 诊断工具，可直接用于 FEX controller trace analysis |

### 撞车评估

本次 13 篇新读论文均不与 idea 核心 claims 撞车。它们全部位于 PG convergence theory 和 RL optimization theory 领域，没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解。五轮累计 39 篇（19 idea-scope + 7+13 experiment-scope）精读后，idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**。

## Deep Lit 2026-06-22 实验 scope — V2 reviewer 5.5 must-fix 文献补给 (6 篇)

以下 6 篇论文由 `deep-lit-tick --scope experiment fex-search-complexity` 于 2026-06-22 系统性搜索并精读后收录。搜索由 reviewer V2 score 5.5 的 5 个 must-fix 项驱动：R20 gradient estimator confound、R21 proof gap (κ_full ≥ κ_restricted)、hat{kappa} estimator 统一、G3 reframing、R22 统计功效。10 组 axis 搜索 + 5 组 web search + B7 反向扩展 24 次（6 papers × 4 types）。Round 2 搜索产出 0 篇新候选，文献饱和终止。

### PG Estimator 理论 & Factored Policy (3 篇，直接服务 R20/R21 must-fix)

- **Alternate PG Estimator for Softmax Policies (2112.11622)** [wiki](wiki/2112.11622.md): Garg, Tosatto, Pan 等 (Alberta/TU Darmstadt), AISTATS 2022. 提出替代梯度估计器解决 softmax 策略次优饱和（sub-optimal saturation）时梯度消失问题。核心改动：去掉 estimator 中的 -π 项，变成只更新被采样动作的偏好 ĝ^ALT = (R-b)·e_A。三个好处：(1) 饱和时方差非零可做随机游走逃离；(2) 乐观 critic baseline 产生有偏更新推动策略走向均匀分布；(3) 计算复杂度可降到 O(log|A|)。**对 R20 must-fix 的影响：** 直接解释了为什么 R20 中 factored_slot (joint_lp REINFORCE) 和 joint_tabular (log_softmax + temperature/tanh) 两种估计器行为不同——regular estimator 在高置信度时梯度消失，而 alternate-like 估计器（如 joint_tabular 的 normalized log-softmax）保留更新信号。R20 的 confound 可以从 estimator 理论角度精确刻画。

- **Factored Policy Gradients (2102.10362)** [wiki](wiki/2102.10362.md): Pina, Mahajan, Oliehoek (Delft/McGill), NeurIPS 2021. 提出 FPG 框架：Influence Network（action-target 二分图）+ Factor Baseline（零开销 control variate）+ Minimum Factorisation（最优 biclique vertex cover）。证明 FPG 无偏、VPG/COMA/DRPG 均为特例、方差分解公式。**对 R21 must-fix 的影响：** (a) FEX 的 4-slot product-policy 自然对应 FPG 的 factored action space——每个 slot 是一个 action dimension，slot 间的 reward coupling 可用 influence network 建模；(b) factor baseline 的方差分解公式 (Proposition 5) 精确刻画了 cross-slot gradient interaction 对 ||∇J||² 的影响——这是 R21 Proposition 2 (κ_full ≥ κ_restricted) 证明 gap 的核心问题；(c) minimum factorisation 理论给出了最优的 slot 分组方式，可用于分析 FEX 的 4-slot 结构是否是最优 factorisation。

- **RPG: KL-Regularized PG Design for LLM Reasoning (2505.17508)** [wiki](wiki/2505.17508.md): Zhang, Liu, Yuan 等, 2025. 统一了 KL 正则化 PG 的设计空间：forward/reverse KL × normalized/unnormalized × k₁/k₂/k₃ estimator。核心发现：(1) k₃ estimator = unnormalized KL；(2) REINFORCE-style + stop-gradient 与 fully-differentiable surrogate gradient-equivalent 的条件；(3) GRPO 的 KL 项 off-policy importance weight 缺失。**对 R20 must-fix 的影响：** RPG 框架精确描述了不同 REINFORCE estimator 变体（如 R20 中 joint_lp vs log_softmax）的数学关系——两者可能对应不同的 KL 方向或正则化形式，导致优化目标本身不同，不仅仅是实现上的 confound。

### PL 常数理论 (1 篇，服务 hat{kappa} must-fix)

- **Ballistic limit of log-Sobolev = PL constant (2411.11415)** [wiki](wiki/2411.11415.md): Chewi, Stromme (CMU), 2024. 证明 lim_{t→0⁺} C_LS(μ_t)/t = C_PL(f)，即低温极限下 log-Sobolev 常数的 ballistic limit 精确等于 PL 常数。同时 ballistic Poincaré 常数 = 1/λ_min(∇²f(x*))。**对 hat{kappa} must-fix 的影响：** (a) 提供了 PL 常数的精确数学刻画——hat{kappa} 是 PL 常数的估计量，本文给出了 PL 常数与采样理论（log-Sobolev）之间的桥梁；(b) 非渐近界 C_LS ≤ C_PL·t + O(t^{6/5}) 可用于分析 hat{kappa} 估计的误差行为；(c) 但关键限制：假设唯一全局最小值点 + C² + Laplacian 增长条件，FEX 的 reward landscape 可能不满足。

### Beyond-Softmax 参数化 (1 篇，服务 G3 reframing must-fix)

- **Beyond Softmax: Gradient Bandits via GNL (2510.03979)** [wiki](wiki/2510.03979.md): Melo, Müller (Indiana/Bonn), 2025. 将 discrete choice 理论（GNL = Generalized Nested Logit）引入 gradient bandits。核心贡献：(1) GEV surplus function 的 GBPA 框架给统一遗憾界；(2) GNL 满足 differential consistency 保证亚线性遗憾；(3) Generalized Gradient Bandit 用 GNL 替代 softmax 实现 action 间相关性建模——同 nest 内共享信息。**对 G3 reframing must-fix 的影响：** 提供了 "architecture matters for G3" 的理论支撑——不同的 action selection 参数化（softmax vs GNL）导致根本不同的 exploration/convergence 行为。FEX 的 MLP controller 可视为一种隐式的 nest 结构（shared features = nest），MLP resolves G3 因为 MLP 的参数共享实质上引入了 action 相关性结构。

### Factored Policy 应用 (1 篇，服务 R21 must-fix)

- **MiniMax Factored Stochastic Policies from Conjoint Data (2504.19043)** [wiki](wiki/2504.19043.md): Jerzak, Chandra, Hazra (UT Austin), 2025. 学习 product-of-Categoricals 策略：线性概率近似 + L2 正则 → 闭式解。Minimax 扩展到零和博弈。Delta method 传播不确定性。**对 R21 must-fix 的影响：** (a) product-of-Categoricals 是 FEX product-policy 的精确数学对应物——每个 slot 的 Categorical 独立出 action，product 是 joint policy；(b) Delta method UQ pipeline 可参考设计 FEX PG 理论中 κ_restricted → κ_full 的不确定性传播；(c) 但关键差距：conjoint 是 offline/batch 设定，FEX 是 online sequential 搜索。

### 本次调研对 claims 的累计影响 (六轮合计 45 篇)

| Claim | 本次新发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | 无 | unchanged |
| (ii) Sampler lower bound | 无 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+4 篇：alternate estimator (2112.11622)、factored PG variance (2102.10362)、RPG REINFORCE design (2505.17508)、GNL gradient bandit (2510.03979)** | further strengthened — 新增三个关键维度：(a) estimator choice 对饱和行为的影响 (alternate vs regular)；(b) factored action space 的 variance 分解和 cross-slot gradient 公式（FPG Proposition 5）；(c) beyond-softmax 参数化的探索/收敛差异（GNL nest-based information sharing）。工具箱从 20 篇扩展到 24 篇。 |
| (iv) Feasibility-hardness route (∃R) | 无 | unchanged |
| (v) Diagnostic bridge (κ_PG→convergence) | **+1 篇：PL constant = ballistic log-Sobolev (2411.11415)** | strengthened — PL 常数有精确的采样理论对应物，hat{kappa} 的数学意义更清晰 |
| (vi) R20 confound resolution | **+3 篇：2112.11622、2505.17508、2102.10362** | new axis — estimator difference theory (alternate vs regular vs RPG variants) + variance decomposition formula for cross-slot gradients |

### 撞车评估

本次 6 篇新读论文均不与 idea 核心 claims 撞车。它们分别位于 PG estimator theory（2 篇）、factored RL（2 篇）、PL constant theory（1 篇）和 discrete choice bandits（1 篇）领域。六轮累计 45 篇（19 idea-scope + 7+13+6 experiment-scope）精读后，idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**。

## Deep Lit 2026-06-24 实验 scope — V3 reviewer 5.8 must-fix 文献补给 (5 篇)

以下 5 篇论文由 `deep-lit-tick --scope experiment fex-search-complexity` 于 2026-06-24 系统性搜索并精读后收录。搜索由 reviewer V3 score 5.8 的 5 个 must-fix 项驱动：G3 bridge 形式命题 (MF#1)、stronger FEX-anchor G3 experiment (MF#2)、pre-register magnitude-aware conflict metric (MF#3)、cross-PDE G3 scaling (MF#4)、bounded-ETR preservation (MF#5)。8 组 axis 搜索 + 7 组 web search + B7 反向扩展 20 次（R1: 4×4 + R2: 1×4）。Round 3 搜索产出 0 篇新候选，文献饱和终止。

### G3 Bridge 工具箱 (3 篇，服务 MF#1 G3 bridge formal proposition)

- **Delightful Gradients Accelerate Corner Escape (2605.11908)** [wiki](wiki/2605.11908.md): Jincheng Mei, Ian Osband (Google DeepMind), 2026.05. 揭示 softmax PG 在次优角落的自陷（self-trapping）机制——负 advantage action 强化角落策略并把最优 action 推回去——并提出 DG (Delightful Gradient)，用 advantage × surprisal 的 sigmoid 门控替代等权更新。关键结果：(a) 零温极限下最优 arm 的 logit 增长无条件超过角落 arm，sector gap ≥ Δ_{1j}·ε/(4K)；(b) 首出逃逸时间 O((K/Δ)·log(π₀(j)/π₀(1)))，相比标准 PG 的指数级逃逸是质的飞跃；(c) 所有比角落 arm 更好的 arm 都是 ally（贡献非负），不会互相竞争；(d) 共享函数近似下的 exact counterexample——两状态 MDP 中 DG 收敛到次优 interior fixed point。**对 MF#1 G3 bridge 的影响：** (a) DG 的 ally 结构 ("every action better than the corner action is an ally") 为 FEX controller 的 G3 bridge 提供了精确的数学语言——如果 FEX 的好 template classes 满足 ally 条件，则 restricted witness 的 corner escape 可推广到 full policy；(b) 共享 FA 反例精确标记了理论的边界——FEX MLP controller 的参数共享可能导致 DG/PG 理论的 tabular→FA 推广失效；(c) escape time 公式中的 1/Δ 依赖直接呼应 R24 的 small_gap 边界。

- **Policy Optimization in Hybrid Discrete-Continuous Action Spaces via Mixed Gradients (2605.14297)** [wiki](wiki/2605.14297.md): Matias Alvo, Daniel Russo, Yash Kanoria (Columbia), 2026.05. FEX 恰好是 hybrid discrete-continuous action space——离散选算子 + 连续优化系数。HPO 在 exogenous MDP 下混合 pathwise (PW) 和 score-function (SF) 梯度，保持无偏。核心理论贡献：∇_κ J = PW term + Cross term，Cross term 在离散策略接近 best response 时消失（上界正比于 ε-suboptimality）。实验显示 SF estimator 的 alignment 在高维时降至 ~0 而 mixed estimator 保持 >0.8。HPONoCross（丢弃 cross term）与 full HPO 表现相似。**对 MF#1 G3 bridge 的影响：** (a) Cross term 消失定理为 FEX 的 restricted→full bridge 提供了新路径——如果 FEX 的 slot-restricted 策略已接近 best response，则 cross-slot gradient 可忽略，restricted κ_PG ≈ full κ_PG；(b) SF alignment 随 action 维度崩塌的诊断与 R20 的 per-param gnsq 方向反转一致；(c) gradient quality metrics (alignment/RMSE/signal ratio) 可作为 R24 的 SC1-SC3 的替代/补充。

- **Federated Softmax PG under Heterogeneous Environments (2505.23459)** [wiki](wiki/2505.23459.md): Safwan Labbi, Paul Mangold, Daniil Tiapkin, Eric Moulines (École Polytechnique/MBZUAI), 2025.05, AISTATS 2026. 首次证明异构 FedPG 全局收敛。核心发现：单智能体的 Łojasiewicz 不等式在联邦目标上不成立——最优联邦策略可能必须是 stochastic 的。通过局部正则性 → 全局 quasi-PL 条件的转化绕过了全局 PL 失效。**对 MF#1 G3 bridge 的影响：** (a) FEX 的 4-slot controller 可类比为 4 个 heterogeneous "agent"——每个 slot 的局部最优不等于全局最优，PL 条件在 product-policy objective 上可能不成立；(b) 局部 PL → 全局 quasi-PL 的分析技术是 G3 bridge 的直接数学工具——可用于证明 "slot-restricted κ_PG bounded → full product-policy κ_PG bounded under quasi-PL"。

### Magnitude-Aware Gradient Conflict Metric (2 篇，服务 MF#3 pre-register conflict metric)

- **Gradient Alignment in PINNs: Second-Order Optimization (2502.00604)** [wiki](wiki/2502.00604.md): Sifan Wang, Bhartari, Li, Perdikaris (Penn), 2025.02, 65 citations. 提出 gradient alignment score 推广 cosine similarity 到多个向量。理论证明 small initialization 下所有优化器的 intra-step alignment 收敛到二元随机变量（方向不可预测）。准二阶优化器 (SOAP) 通过 preconditioning 隐式促进梯度对齐。10 个 PDE benchmark 上 SOAP 超 Adam 2-10x。**对 MF#3 的影响：** gradient alignment score 是 R24 sign-only conflict fraction 的直接 magnitude-aware 替代——可在 pre-register 时定义为 "multi-gradient alignment score ≥ threshold τ" 而非 "sign-only cf < 0.10"。这解决了 R24 SC2 的 formal FAIL 问题：jointnorm medium_gap 的 cf=0.333 (sign-only FAIL) 可能在 alignment score 下 PASS（因负 cosine 幅度极小）。

- **Conflict-Aware Harmonized Rotational Gradient for Multiscale Kinetic Regimes (2604.24745)** [wiki](wiki/2604.24745.md): Zhangyong Liang, 2026.04. 提出 HRGrad 用 SO(2) 等距旋转替代欧几里得投影解决梯度冲突，避免 PCGrad/ConFIG 的能量不可逆裁剪。核心公式：gradient alignment metric 保证 final update 与每个 loss-specific gradient 的内积 > 0，同时动态调整 gradient magnitudes based on conflict levels。有凸和非凸收敛证明。**对 MF#3 的影响：** HRGrad 的 alignment metric 是另一个 magnitude-aware conflict metric——它不仅看方向还看模长比。等距旋转 vs 正交投影的区别精确对应 R24 的问题：sign-only projection (PCGrad-style) 可能裁掉有用信号（jointnorm medium_gap 的微小负 cosine），而 HRGrad 的旋转保持信号能量。

### 本次调研对 claims 的累计影响 (七轮合计 50 篇)

| Claim | 本次新发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | 无 | unchanged |
| (ii) Sampler lower bound | 无 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+3 篇：DG corner escape (2605.11908)、hybrid HPO mixed gradient (2605.14297)、federated softmax PG (2505.23459)** | further strengthened — 新增三个关键维度：(a) corner escape 的 ally 结构和对数级逃逸时间 (DG)；(b) hybrid discrete-continuous action 的 cross-term 消失定理（near best response 时 restricted ≈ full）；(c) multi-slot product-policy 的 PL 失效与 quasi-PL 修复。工具箱从 24 篇扩展到 27 篇。 |
| (iv) Feasibility-hardness route (∃R) | 无 | unchanged |
| (v) Diagnostic bridge (κ_PG→convergence) | **+2 篇：gradient alignment score (2502.00604)、HRGrad alignment metric (2604.24745)** | significantly strengthened — 现有两个 magnitude-aware gradient conflict metric 可直接替代 R24 的 sign-only conflict fraction；gradient alignment score (多向量推广 cosine) 和 HRGrad metric (等距旋转保能) 是 MF#3 的 pre-register 基础 |

### 撞车评估

本次 5 篇新读论文均不与 idea 核心 claims 撞车。它们分别位于 softmax PG 优化理论（2 篇）、hybrid RL（1 篇）和 multi-task gradient 方法论（2 篇）领域。七轮累计 50 篇（19 idea-scope + 7+13+6+5 experiment-scope）精读后，idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**。

## Deep Lit 2026-06-25 实验 scope — V4 reviewer 5.6 后 scaling / K-PL / architecture 文献补给 (6 篇)

以下 6 篇论文由 `deep-lit-tick --scope experiment fex-search-complexity` 于 2026-06-25 系统性搜索并精读后收录。搜索由 reviewer V4 score 5.6 的 3 个 CRITICAL must-fix 驱动：#1 theorem is taxonomy（需 lower bound/impossibility）、#2 B4 K-PL gap（需 K-PL for product policies）、#3 all evidence on depth2_sub only（需 scaling analysis）。9 组 B1 axis 搜索 + 4 组 B0 web search + 6 组 Round 2 搜索 + B7 反向扩展 24 次（6 papers × 4 types）。Round 2 搜索产出 0 篇新候选，文献饱和终止。

### PG Convergence Theory — Dimension-Free 结果 (1 篇，服务 CRITICAL #3 scaling)

- **PPO in the Fisher-Rao Geometry (2506.03757)** [wiki](wiki/2506.03757.md): Lascu, Šiška, Szpruch (RIKEN/Edinburgh/Turing), 2025.06. 提出 FR-PPO：用 Fisher-Rao (Hellinger) 几何惩罚替换 PPO 的 clipped surrogate。核心理论：(1) 推导更紧的 surrogate 下界（integrated TV² 而非 max-over-states）；(2) FR-PPO 在 direct parameterization 下达到 O(1/n) sub-linear 收敛且**不依赖 action 或 state space 维度**；(3) parametrized policy 下 sub-linear up to compatible FA error。Proj-NPG 等价性。**对 CRITICAL #3 的影响：** (a) FR-PPO 的 dimension-free 收敛率意味着从 depth2_sub (Q=1539) 扩展到 depth2 (Q≈42.6M) 不会改变收敛速率的理论保证——如果 FEX controller 采用 FR 几何替代 softmax，scaling 问题在理论层面消失；(b) 这为 reviewer 的 "如何处理 Q 增长" 提供了一个 positive answer：不是证明 softmax PG 在大 Q 下收敛（它确实 can take exponential time），而是证明存在 PG 变体（FR-PPO）在大 Q 下收敛速率不变。**对 MF #5 的影响：** FR-PPO 是 product/jointnorm 之外的第三种 controller 架构——理论上解决了 architecture sensitivity 问题。

### SR Search Scaling (2 篇，服务 CRITICAL #3 depth scaling)

- **Scaling Up Unbiased Search-based SR (2506.19626)** [wiki](wiki/2506.19626.md): Kahlmeyer, Giesen, Habeck, Voigt (Jena), IJCAI 2024. 提出 UDFS（Unbiased DAG Frame Search）：用 expression DAG 替代 tree 表示，共享公共子表达式（CSE）显著压缩搜索空间。Variable augmentation 技术通过先在小空间搜 augmentation features 再在增强空间搜 full expressions 实现两阶段扩展。SRBench 上 recovery rate 55.8%，超越所有 SOTA（GP 44.3%、Transformer 系 41.4%）。**对 CRITICAL #3 的影响：** (a) DAG 表示自然对应 quotient compression——CSE sharing 与 sound_ac 的等价类合并在精神上一致，但 DAG 走更远（语义 sharing 而非仅语法交换律）；(b) UDFS 证明 systematic enumeration 可 scale 到 depth-4（"up to 10^6 DAG frames per depth level"），为 FEX 从 depth2_sub 扩展到 depth2/depth3 提供方法论参考；(c) variable augmentation 的两阶段策略可启发 FEX 的 "先搜 restricted template 再扩展到 full action space" 实验设计。

- **GoodRegressor: Hierarchical Inductive Bias (2510.18325)** [wiki](wiki/2510.18325.md): S.-H. Jang (Tohoku U.), 2025.10. 提出深度控制的词典序符号回归：通过 swap（变量替换）+ transit（109 种标量变换）+ pick（减少活性变量以增加交互深度）三步枚举，在最高 ~10^28 种候选结构中系统搜索。核心发现：**interaction-depth evolution 揭示系统依赖的最优窗口**——氧离子导体在 depth-3 饱和，NASICON 在 depth-4 饱和，超导体需要 depth-5+。这是 depth 作为结构复杂性诊断轴的经验分类学。**对 CRITICAL #3 的影响：** (a) 直接回答 reviewer "depth2_sub 之外的证据"：GoodRegressor 展示了 depth 控制如何在 10^28 规模搜索中保持 tractability——通过词典序枚举 + MPI 并行而非穷举；(b) depth 饱和窗口概念可迁移到 FEX：如果 FEX 的 G3 (policy-concentration gate) 也存在 depth-dependent 饱和行为，则可在 depth-2/depth-3 做预测并验证；(c) 但 GoodRegressor 是纯组合枚举（无 RL controller），与 FEX 的 PG 搜索机制不同——迁移主要在 framing 而非方法层面。

### PL Theory — Non-Standard Objectives (1 篇，服务 CRITICAL #2 K-PL gap)

- **PL Inequality for Quadratically Regularized Optimal Transport (2605.27175)** [wiki](wiki/2605.27175.md): González-Sanz, Nutz, Riveros (Columbia), 2026.05. 证明 QOT 对偶目标（含 (t)+ 正部函数，破坏强凹性）满足**局部 PL 不等式**，常数仅依赖 problem primitives。关键技术：(1) 沿插值路径 f_t 分析 Hessian；(2) integral operator 的最小特征值估计（1D: 核 K_f 的积分算子；高维: 块结构 tensor 分析）；(3) 从 error bound 推 PL（error bound → KL growth → PL）。三种算法（gradient ascent / coordinate ascent / coordinate gradient ascent）均证明线性收敛。**对 CRITICAL #2 (B4 K-PL) 的影响：** (a) QOT 的 (t)+ 函数与 FEX reward landscape 中的 threshold 行为（表达式 "命中/未命中" PDE 解）有结构相似性——FEX 的 value function 也存在非光滑区域（reward 在 "好/坏" 结构间跳变）；(b) interpolation path 分析技术可为 FEX controller 的 κ_PG 建立 local PL：沿参数路径 θ_t = (1-t)θ_* + tθ 分析 Hessian，在 θ_* 邻域建立 PL；(c) error bound → PL 的推导链可作为 B4 K-PL 证明的技术路线。

### Architecture Sensitivity (1 篇，服务 MF #5)

- **Distributions as Actions: A Unified Framework (2506.16608)** [wiki](wiki/2506.16608.md): He, Mahmood, White (Alberta/Amii), 2025.06, ICLR 2026 submitted. 将策略的参数化分布参数重定义为 RL agent 的连续动作空间。DA-PG 估计器是 LR 和 RP 估计量的条件期望，因此**严格具有更低方差**。ICL（Interpolated Critic Learning）在策略分布参数和采样动作间插值做 TD 学习，使 critic 学习更平滑的 value landscape。DA-AC 在 40+ 环境（离散/连续/混合/高维离散）取得竞争性性能。**对 MF #5 的影响：** (a) 如果 FEX controller 用 DA-AC 框架（输出每个 slot 的分布参数作为连续动作），则 action space 统一为连续→可直接应用 FR-PPO 等 dimension-free 收敛理论；(b) 方差降低意味着 κ_PG 估计更精确——R20/R24 的 per-param gnsq 方向反转可能在 DA-PG 下消失。

### Gradient Conflict Resolution at Scale (1 篇，服务 G3)

- **Scalable Multi-Objective Robot RL via Gradient Conflict Resolution (2509.14816)** [wiki](wiki/2509.14816.md): Munn, Tidd, Böhm, Gallagher, Howard (UQ/CSIRO), 2025.09. GCR-PPO：multi-head critic 为每个 reward component 独立估计 advantage → 每个 component 产生独立的 clipped PPO gradient → PCGrad 投影解决冲突（优先级排序）。IsaacLab manipulation/locomotion benchmark 上平均提升 9.5%，high-conflict tasks 提升更大。关键发现：**conflict 主要发生在 task reward 和 regularization reward 之间**，而非 task objectives 之间。**对 G3 的影响：** (a) GCR-PPO 的 multi-head critic + per-component gradient 与 FEX 的 4-slot 结构直接对应——每个 slot 可视为一个 "objective"（slot 间梯度冲突 = FEX R24/R28 的 thresholded conflict）；(b) "conflict 主要在 task vs regularizer" 的发现可解释 FEX 的 restricted-to-fullspace transition：restricted (2-arm) 时 slot 间无冲突（只看 task reward），fullspace 时 slot 间冲突激增（更多 slot = 更多 implicit regularizer 效应）；(c) 优先级投影是 PCGrad 的有序版本——可比较 FEX 实验中使用 PCGrad-style 冲突解决后 NDMS 是否降到 0.

### 本次调研对 claims 的累计影响 (八轮合计 56 篇)

| Claim | 本次新发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | UDFS (2506.19626) DAG 压缩与 quotient compression 精神一致 | marginally strengthened — 独立的工程验证表明结构化枚举确实有效 |
| (ii) Sampler lower bound | 无 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+3 篇：FR-PPO dimension-free (2506.03757)、DA-AC 低方差 (2506.16608)、GCR-PPO 冲突解决 (2509.14816)** | further strengthened — 新增两个关键维度：(a) FR 几何下收敛率不依赖 action space 维度（dimension-free guarantee）；(b) distributions-as-actions 统一框架使离散→连续→可用 dimension-free 理论。工具箱从 27 篇扩展到 30 篇。 |
| (iv) Feasibility-hardness route (∃R) | 无 | unchanged |
| (v) Diagnostic bridge (κ_PG→convergence) | **+2 篇：GCR-PPO 多目标冲突分解 (2509.14816)、QOT PL 技术路线 (2605.27175)** | strengthened — (a) multi-head critic + per-component gradient 是 FEX 4-slot conflict 分析的直接工具；(b) interpolation path + error bound → PL 是 B4 K-PL 证明的候选技术路线 |
| (vi) Scaling beyond depth2_sub | **+3 篇：FR-PPO dimension-free (2506.03757)、UDFS depth-4 (2506.19626)、GoodRegressor depth-5+ at 10^28 (2510.18325)** | new axis — 理论（FR-PPO 无维度依赖）+ 方法论（UDFS DAG / GoodRegressor 词典序）双重支持 scaling 可行性；depth 饱和窗口概念可迁移 |

### 撞车评估

本次 6 篇新读论文均不与 idea 核心 claims 撞车。它们分别位于 PG convergence theory（FR-PPO 1 篇）、SR search methodology（UDFS + GoodRegressor 2 篇）、RL action space design（DA-AC 1 篇）、multi-objective RL（GCR-PPO 1 篇）和 OT optimization theory（QOT PL 1 篇）领域。没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解。八轮累计 56 篇（19 idea-scope + 7+13+6+5+6 experiment-scope）精读后，idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**。
