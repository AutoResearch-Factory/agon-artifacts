<!-- 书写报告使用中文 -->
---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-dim-lift-skeleton/
---

- One-sentence summary: 将低维 FEX 搜索得到的 skeleton 先做 searchability audit, 再用有限可交换宏 grammar、系数交换性和高维 PDE probe 认证是否可以提升到高维并只优化连续参数。
- Hypothesis: FEX 的一部分高维成功来自两个同时成立的条件: 解落在可交换宏类里, 且 RL controller 在低维能稳定搜到该宏的 skeleton。Poisson 和 radial 这类稳定家族应通过四道门并在 d=100 仍能 parameter-only lift; pairwise 解析宏本身可 lift, 但当前 FEX 搜索若多 seed 找不到它, 应被拒绝而不是被强行 lift。
- Expected outcome: 成功信号是 Poisson/radial 多 seed 通过四道门, d=100 probe 仍低于 1e-3, asymmetric control 被拒绝, pairwise 保持"analytic liftable but FEX-unsearchable"的诊断结果。最便宜的反证是 stable family 的 d=100 probe 失败, 或 asymmetric/pairwise failed-search 输出被 gate 误通过。
- Contribution type: method+empirical-finding (保留 v4 类型; method 是四门 certificate/audit pipeline, empirical-finding 是 searchability 与 liftability 可分离这一机制发现; topic 未限制 contribution types)
- Risk: MEDIUM
- Estimated effort:
  - Compute: 120-180 GPU-hours, 主要用于 5 seeds x 4 families x depth sweep、d in {10,20,50,100} lift 验证、from-scratch FEX/FEX-PG/HD-TLGP/PSR-style matched baselines 和 gate ablation
  - Data: available; PDE collocation 和 boundary points 解析生成, 无外部数据集或模型权重
  - Implementation: 2-3 weeks
- Novelty quick-check: PSR 是最近竞品, 但它走 projection-based decomposition 加 symbolic program synthesis, 不做 exchangeability certificate, 也不做 parameter-only lift/reject 决策。HD-TLGP 需要已知 1D 解析解、硬编码扩展和 GP 全搜索, 只到 d=3。GSR 的 permutation invariance 是表达式内部的交换律表示, 不是跨维度宏推断。NMIPS 是同维 PDE family 参数迁移。现有工作没有把 FEX cross-dimension lifting 写成"低维搜索稳定性 + 宏可交换性 + 高维 probe"的认证问题。
- Strongest objection: 宏 grammar 是手设的有限子语言, 所以本文不能解释所有 FEX 高维行为; 它只能认证一批明确的 exchangeable PDE families, 并把失败归因给搜索稳定性或宏不匹配。
- Why we should do this: 这个项目把 FEX 的无 CoD 近似理论和实际 RL 搜索行为之间的缺口变成可测问题。正结果给出一个可部署的 lift/reject 程序; NULL 结果也能指出 FEX controller 哪些可 lift 结构搜不到。
- Pilot:
  - Setup: local RTX 4060 Ti; 使用真实 FEX depth1 搜索输出做 Poisson/radial macro inference, 用 analytic pairwise/asymmetric controls 做机制对照, 并新增 d=100 probes 与 radial from-scratch FEX smoke baselines
  - Metric: Gate 1 记录 family-level search stability; Gate 2 要求 low-d macro rel-RMSE < 0.02; Gate 3 要求 additive coefficient CV < 0.1; Gate 4 要求 selected macro 的 high-d probe rel-L2 < 1e-3 且 top-k probe 不以 2x margin 推翻低维最佳宏
  - Result: Poisson real FEX seed1 在 d=100 选 `sum_x2`, rel-L2 2.13e-8, wrong top-k macros 为 0.998 和 0.00293。Radial real FEX seed2 在 d=100 选 `square_sum_x2`, rel-L2 0.0, wrong top-k macros 为 0.995 和 0.103。Analytic pairwise d=100 选 `pairwise_xx`, rel-L2 9.23e-7, 但真实 pairwise FEX 仍是 0/3 搜索失败。Asymmetric d=100 被拒绝, low-d rel-RMSE 0.174 且 CV 0.471。Radial d=10 from-scratch FEX 可成功, rel-L2 4.39e-4 in 81.1s; radial d=100 short-budget FEX smoke 失败, rel-L2 0.169 in 92.7s, 而 lifted radial d=100 只需 0.64s probe 且 rel-L2 0.0。
  - Signal: POSITIVE but bounded. v5 支持"通过认证后可到 d=100"和"single d=10 不是主要卖点"; 论文价值应写成 certification、amortization 和 searchability audit, 而不是声称 from-scratch FEX 在所有高维都失败。

- Review handling (v4->v5):
  1. Matched baselines 缺失: ACCEPT。v5 增加 radial d=10 from-scratch FEX 和 d=100 short-budget smoke; 完整 FEX-PG/HD-TLGP/PSR-style 仍是 P0 实验, 不把它写成 benchmark contribution。
  2. d=100 证据不足: ACCEPT。v5 新跑 Poisson/radial real-FEX d=100 probes, analytic pairwise d=100, asymmetric d=100 rejection。
  3. Gate 1 太 qualitative: ACCEPT。正文把 Gate 1 写成 family-level audit, proposal 阶段用 >=2/5 seeds passing 作为 stable family 的操作阈值; idea pilot 只报告当前 3/2/3 seeds。
  4. Pairwise 0/3 可能被看成方法失败: PUSHBACK on framing, ACCEPT on clarity。它确实限制自动 selector recall, 但 analytic pairwise d=100 rel-L2 9.23e-7 证明宏可 lift; 失败来自 controller searchability, 这正是经验发现。
  5. d=10 from-scratch FEX 也能成功: ACCEPT。v5 去掉"single target dimension 加速"暗示, 改成跨维度摊销和 d=100 认证。
  6. Macro grammar 手工设定: PUSHBACK on "must be general symmetry discovery"。本文不做通用 symmetry discovery; 合理 claim 是有限宏类上的 certificate。ACCEPT 范围限制, 在 strongest objection 和 claims 里明确写出。
  7. Weight tying / coefficient exchangeability: ACCEPT。Gate 3 保留为独立 gate; additive 类要求 CV < 0.1, 非 additive 宏用高维 probe 和 family-specific validation 控制。
  8. Contribution drift check: v4->v5 仍是 `method+empirical-finding`, 没有删除 v4 的任何 contribution type; topic 没有 `preferred-contribution-types`。
- Claims and Claims matrix: POSITIVE claim: 对于落在有限 exchangeable macro grammar 且低维 FEX 多 seed 稳定找到 skeleton 的 PDE family, 四门认证后可以安全提升到 d=100 并用 parameter-only optimization 达到低误差, 同时节省重复高维搜索。NULL claim: analytic 宏可 lift 但真实 FEX 多 seed 搜不到, 论文转为 FEX searchability diagnostic。NEGATIVE claim: stable family 的 high-d probe 失败或 asymmetric false positive 出现, method claim 失败。
- Narrative: 论文叙事更适合写成 FEX scalability 的 certificate and audit, 而不是新的高维 PDE solver。四道门分别回答"FEX 是否搜得到"和"搜到的 skeleton 是否该 lift"; pairwise 结果说明这两个问题可以分开。
- Experiments: P0: 5 seeds x Poisson/radial/pairwise/asymmetric, d in {10,20,50,100}, from-scratch FEX/FEX-PG/HD-TLGP/PSR-style matched baselines, four-gate ablation, and amortization break-even. P1: depth-1/2/3 sweep, weight tying vs independent coefficients, heldout-family threshold calibration, and larger finite macro grammar checks. Extra datasets/settings are evidence for method claims, not a benchmark or application contribution.
- Assets status: v5 d=100 probes and radial baseline smokes are complete locally; code/data handoff is in `workspace/fex-dim-lift-skeleton/data/MANIFEST.md`.

## Deep Lit 2026-06-21 (experiment scope)

以下 2 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-21 搜索并精读后收录。本次搜索覆盖 6 axes + B0 web search + 2 轮 B7 反向扩展，基于 179 篇已读 wiki 池高度饱和，仅发现 2 篇新论文。

### SIGS — Grammar-Guided Neuro-Symbolic PDE Solver (2502.01476)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2502.01476.md)

Orestis Oikonomou, Levi E. Lingsch, Dana Grund, Siddhartha Mishra, Georgios Kissas (ETH Zürich), 2025.02. 1 citation.

用 CFG 文法生成 23,682 个数学原子表达式，GVAE 嵌入 32 维潜在流形，对每个新 PDE 指定 Ansatz 组装方式，两阶段搜索（结构选择 + Adam 系数精炼），仅靠 PDE residual 评分，不需训练数据。在 Burgers/Diffusion/Damped Wave 上达机器精度（~1e-13），比 HD-TLGP 快 100-1000 倍且精度高 13-15 个数量级。首次处理耦合非线性 PDE 系统。局限：依赖人工 Ansatz，未测高维（>3d）。

**与本实验关系**：不撞车（SIGS 是 equation-driven，FEX 是 data-driven RL 搜索）。但 SIGS 直接 benchmark 了 HD-TLGP 和 SSDE，为实验急需的外部 baseline 对照提供性能参考数据：HD-TLGP 在 Poisson 上 rel-L2 4.36e-4，SIGS 达 1e-13。SIGS 的 grammar-guided 方法论（CFG → GVAE → 两阶段搜索）可启发 FEX macro grammar 的未来扩展，但不影响当前实验设计。

### CCM — Discovering Physical Directions in Weight Space (2605.14546)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2605.14546.md)

Pengkai Wang, Pengwei Liu 等, 2026.05. 2 citations.

研究从同一 family anchor 微调的 PDE endpoint experts 在权重空间中编码了可解释的物理方向。Endpoint-anchor 残差可分解为 shared adaptation + signed physical direction。提出 Calibration-Conditioned Merge (CCM)，无需训练即可跨 PDE regime 迁移。在 Reaction-Diffusion/NS/Dam-Break 上 OOD 外推 error 分别降 54.2%/42.8%/13.8%。

**与本实验关系**：不撞车——CCM 是 neural operator 方法，本实验是 symbolic macro grammar。概念上相关：CCM 的 cross-regime physical direction 与我们的 cross-dimension macro lifting 都在解决"PDE family 内结构复用"问题，但技术路线正交（weight-space composition vs symbolic gate audit）。CCM 的 wrong-sign control 实验范式（翻转坐标方向验证方向性）可作为实验设计参考。

## Deep Lit 2026-06-22 (experiment scope, R2)

以下 3 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-22 搜索并精读后收录。本次搜索覆盖 6 axes × 2 轮 + B0 web search (9 组) + 12 次 B7 反向扩展。基于 199+ 篇已读 wiki 池极度饱和，仅发现 3 篇新论文。

### Trustworthy AI in Numerics — A Posteriori Verification for NN PDE Solvers (2509.26122)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2509.26122.md)

Emil Haugen (Oslo), Alexei Stepanenko, Anders C. Hansen (Cambridge), 2025.09. 1 citation.

提出针对 NN PDE 求解器的后验验证算法。核心：利用 PDE 残差边界 + activation bounds（逐层计算 ReLU/RePU 网络导数局部变化上界） + 中点求积，构造可认证的 ε-accuracy 验证框架。如果 NN 不满足精度要求，算法能检测并拒绝；通过验证的 NN 被认证为 ε-accurate。在 1D heat equation 上演示（2 隐层、宽度 100、RePU 3 阶），最高精度 ε=2.5e-5。局限：仅支持 2 隐层、仅 ReLU/RePU、仅 1D 演示、计算复杂度 O(ε^{-d})。

**与本实验关系**：不撞车（NN solver 的 a posteriori verification vs 我们的 symbolic macro gate audit）。方法论参考价值：(1) "通过验证 → 认证；未通过 → 拒绝"的 accept/reject 决策模式与 three-gate audit 逻辑同构；(2) activation bounds 方法可迁移到 feature attribution 的 reliability bound 计算；(3) 论文论证了"a priori error bounds 无法保证计算解精度，必须 a posteriori 验证"——这正是我们 gate audit 存在的理由。反引 2606.12050 (Huseynov et al.) 做 PINN 上下界后验误差估计，方向正在被快速跟进。

### PG-SR — Prior-Guided Symbolic Regression (2602.13021)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2602.13021.md)

Jing Xiao, Xinhai Chen 等 (NUDT), 2026.02. 0 citations. ICML 2026 投稿。

识别并形式化「伪方程陷阱」(Pseudo-Equation Trap)——仅靠经验风险最小化导致方程拟合数据但不符科学原理。提出三阶段管线（Warm-up → Evolution → Refinement），核心创新：(1) 显式编码领域先验为可执行约束检查程序（LLM 辅助提取 + 人类专家验证）；(2) PACE 机制（Prior-Annealing Constrained Evaluation），早期允许违反约束以保持探索多样性，后期逐步收紧。理论上用 Rademacher Complexity 证明约束子空间有更紧泛化界。实验在 LLM-SRBench 5 个跨领域 benchmark 上取得 SOTA，OOD 泛化优势显著。

**与本实验关系**：不撞车（面向一般 SR，非 PDE 求解）。概念参考：(1) pseudo-equation trap 与我们的 "exact-library tautology" 问题有类比——reviewer 质疑 gate audit 只是在做 exact-match lookup 时，论文可引用 PG-SR 的框架说明"先验约束（macro grammar + PDE residual gate）减少假阳性"；(2) PACE 退火机制可参考用于 gate threshold 渐进校准——早期宽松允许更多候选通过，后期收紧到最终阈值。

### PICS — Partition-of-unity Certified PDE Solver (2603.21271)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2603.21271.md)

Z. Tao, Hong Zhou, Han-Yu Liang, Fujun Liu, 2026.03. 0 citations.

闭环神经 PDE 求解器框架，核心创新：(1) gate-structured admissible manifold 用 partition-of-unity 构造解空间，通过流函数潜表示实现不可压缩性为硬约束；(2) restricted jet prolongation 仅保留闭包必需的有限导数坐标；(3) certificate field 从归一化残差切面提取 a posteriori 误差估计，驱动训练测度向未验证高风险区动态传输。在 3 个 2D 合成 benchmark 上评估：Case 1/2 全面领先 PINN/DGM/DRM，但 Case 3（最难设定）被 DGM 全面压制。

**与本实验关系**：不撞车（mesh-free neural PDE solver，与 symbolic macro audit 完全不同）。概念参考有限：(1) "gate-structured" 和 "certificate field" 术语与我们的 gate audit 有表面相似，但技术内核不同（PICS 的 gate 是连续流形约束，我们的 gate 是离散 accept/reject 决策）；(2) 论文的 Case 3 失败是有用教训——当 setting 变难时，sophisticated method 可能不如简单 baseline，提醒我们关注 gate audit 在 harder families 上的表现。

## Deep Lit 2026-06-22 (experiment scope, R3)

以下 4 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-22 搜索并精读后收录。本次搜索覆盖 6 axes × 2 轮 + B0 web search (5 组) + 16 次 B7 反向扩展。基于 210+ 篇已读 wiki 池极度饱和，仅发现 4 篇新论文（全部非撞车，均为概念/方法论参考）。

### BEACONS — Bounded-Error Neural PDE Solvers with Certificates (2602.14853)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2602.14853.md)

Jonathan Gorard, Ammar Hakim, J. Juno (Princeton Plasma Physics Lab), 2026.02. 0 citations.

构造具有可证明误差界的神经 PDE 求解器框架。核心三步：(1) 利用双曲 PDE 特征线法先验预测解的连续阶数，即使在训练域外；(2) 结合 Mhaskar-Poggio 浅层网络逼近理论将光滑性预测转化为 L∞ 误差界 O(N^{-n/d})；(3) 代数可组合性——将间断解分解为光滑函数与间断函数的复合，用光滑函数的低误差压制间断函数的高误差。含 Racket DSL + C 代码生成器 + 自动定理证明器（machine-checkable 证明证书）。在 1D/2D 线性对流、无粘 Burgers、可压缩 Euler 上 L∞ 误差降低 40-80%。

**与本实验关系**：不撞车（neural PDE solver 形式验证 vs symbolic macro gate audit）。方法论参考价值：(1) "machine-checkable certificate" 概念为我们的 gate audit 提供更严格的参照系——可在论文中将我们的 three-gate cascade 定位为 "empirical certification"，与 BEACONS 的 "formal certification" 形成明确的 rigor spectrum；(2) 代数可组合性的误差传播公式 ||f∘g - f̃∘g̃|| ≤ e_f + L·e_g 可作为 structured decomposition 的理论参考；(3) 论文前作 2503.13877 也在本轮精读。

### Formal Proofs for Hyperbolic PDE Solvers (2503.13877)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2503.13877.md)

Jonathan Gorard, Ammar Hakim (Princeton), 2025.03. 3 citations.

BEACONS 的前作。在 Racket 中构建形式化验证管线：DSL 描述一阶双曲 PDE 系统 → 自动生成 C 代码 → 浮点感知符号定理证明器（IEEE 754 严格遵循：允许交换律但不允许结合律）→ 自动证明 L²稳定性、通量守恒、Lipschitz 连续性、TVD 等性质。关键发现：定理证明器的"失败"大多是正确的（如等温 Euler 中 ρ>0 前提缺失被正确阻塞），只有 Roe 通量非线性 Euler 是真失败（代数能力不足）。

**与本实验关系**：不撞车。方法论参考：(1) 浮点感知的符号推理——提醒我们 gate audit 的数值阈值判断（rel-RMSE < 0.02, rel-L2 < 1e-3）在浮点精度边界可能有隐含风险；(2) "证明器正确失败"的概念 = 我们 gate 正确拒绝非 exact-library family 的类比。

### COSINE — LLM-Guided Adaptive Symbolic Library (2604.12806)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2604.12806.md)

Xiao Liang, Juyuan Zhang 等 (中科大), 2026.04. 0 citations. ICML 2026 投稿。

联合推断隐式交互图和稀疏符号动力学。内层用 Gumbel-Softmax 可微图学习 + 稀疏线性组合基函数联合优化；外层用 LLM（GPT-OSS 20B）根据内层反馈（loss/系数重要性/残差）**自适应剪枝或扩充基函数库**，best-so-far 策略保证单调改进。18 个 setting 中 17 个 AUC 最优。消融显示静态库在复杂系统显著衰退。

**与本实验关系**：不撞车（交互网络逆问题 vs 我们的 PDE symbolic audit）。直接参考价值：**reviewer 提出的 grammar scaling 问题（"4 claim-eligible macros + 5 distractors 太小，grammar 变大后 FA rate 是否仍为 0？"）可以借鉴 COSINE 的自适应库扩展思路** — 用 inner-loop 数值反馈（PDE residual）引导 outer-loop 库扩展，而非手工增加 macro。COSINE 的 Message-Update 分解范式也可参考设计 structured macro expansion。

### A Posteriori Certification for NN PDE Approximations (2502.20336)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2502.20336.md)

Lewin Ernst, Nikolaos Rekatsinas, K. Urban, 2025.02. 3 citations.

为 NN PDE 解提供严格后验误差上下界。核心：选择嵌入/包络原始域的简单几何域（圆/矩形），通过零延拓构造 Sobolev 空间同构（isometry with constant 1），将残差对偶范数计算转移到简单域上用谱方法+快速 FEM 高效求解。penalized extension 通过罚方法近似正交投影解决 Hahn-Banach 定理非构造性问题。理论覆盖椭圆和抛物线性 PDE，数值实验验证下界尖锐、上界经罚修正后接近真误差。

**与本实验关系**：不撞车（误差认证工具，非 solver）。概念参考：(1) 严格的 upper+lower error bound 框架为我们的 gate threshold 选择提供理论对照——论文证明了后验误差估计可以做到 tight；(2) 我们的 gate audit 在概念上类似其 "certification" 步骤（通过 → 认证，不通过 → 拒绝），但我们是 discrete accept/reject on symbolic macro fit + PDE residual，他们是 continuous error bound on NN output。

## Deep Lit 2026-06-26 (experiment scope, R4)

以下 3 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-26 搜索并精读后收录。本次搜索覆盖 6 axes × 12+ arxiv-tool queries + 5 组 B0 web search + FEX/SIGS 反引检查 + 12 次 B7 反向扩展。基于 322 篇已读 wiki 池极度饱和，仅发现 3 篇新论文（全部非撞车）。

### AutoNumerics — Autonomous Multi-Agent PDE Solving Pipeline (2602.17607)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2602.17607.md)

Jianda Du, Youran Sun, Haizhao Yang (Maryland), 2026.02. 3 citations.

基于 GPT-4.1 的多智能体框架，从自然语言 PDE 描述自动设计、实现、调试和验证经典数值求解器。核心创新：(1) Planner+Selector agent 自动生成 10 个候选数值方案（FD/FEM/Spectral/FV）并过滤不稳定配置；(2) 粗到细执行策略（先低分辨率修逻辑，再高分辨率修数值稳定性）；(3) 残差自验证机制（无解析解时用 PDE 残差范数评估质量）；(4) Fresh Restart 避免坏代码路径。CodePDE 5 题几何平均 nRMSE 9.00e-9（比 CodePDE 低 6 个数量级）。局限：仅规则域、绑定单一 LLM、无代码开源。

**与本实验关系**：不撞车（AutoNumerics 是自动化数值求解工具链，fex-dim-lift-skeleton 是符号表达式跨维 lift 方法）。Yang 组最新方向：从符号 FEX 扩展到 LLM-agent 数值求解。方法论参考：(1) 残差自验证机制与 gate audit 的 PDE residual probe 在概念上同构；(2) 粗到细策略可启发 gate threshold 的渐进校准；(3) 多候选方案评分过滤与 top-k macro probe 的设计模式一致。

### NOMTO — Neural Operator-based Symbolic Model Discovery (2501.08086)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2501.08086.md)

S. Garmaev, Siddhartha Mishra (ETH), Olga Fink, 2025.01. 2 citations.

将预训练神经算子（FNO/CNO/FNN）作为可微原语嵌入 EQL 符号计算图，将方程发现从局部微分算子扩展到非局部空间算子、辅助场耦合和时间记忆积分。核心设计：「算子学习与方程发现分离」——神经算子先独立训练逼近目标物理算子，冻结后在符号发现阶段仅优化图权重，经三阶段优化（全图拟合→L1 稀疏+剪枝→refit）提取紧凑符号方程。在分数扩散、Euler-Poisson、遗传粘弹性三案例上成功恢复正确方程结构。与 SIGS (2502.01476) 同组（Mishra/ETH）。

**与本实验关系**：不撞车（PDE 符号发现 vs 我们的 FEX skeleton 跨维 lift audit）。方法论参考：(1) 分离学习与结构搜索的设计模式——预训练子模块后上层做可微结构搜索，与 FEX macro inference 的"先搜 skeleton 再推 macro"思路类似；(2) 三阶段稀疏优化可参考用于 macro grammar expansion；(3) 代理精度决定上层发现质量（FNN 在 Poisson 算子上的失败是重要教训）。

### FEX-Turbulence — FEX for Turbulent Dynamics with Moment Recovery (2605.10687)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2605.10687.md)

Xingjian Xu, Di Qi, Chunmei Wang, 2026.05. 0 citations.

两阶段数据驱动框架：Stage I 用 FEX（RL controller + 二叉树搜索）从轨迹数据发现确定性动力学闭式表达式；Stage II 用生成模型（TFDM/SRAN/VAE）学习残差随机分布以恢复高阶统计矩（最高五阶）。在随机 triad 模型 5 种 regime 上测试，FEX 在所有 regime 下准确恢复正确表达式和系数（误差 10^-2~10^-5），显著优于 Weak SINDy。理论分析证明 FEX 系数估计量相合性和 TFDM 误差界。

**与本实验关系**：不撞车（FEX 应用而非方法改进）。但展示了 FEX 在新领域的能力，确认 Yang/Wang 组持续扩展 FEX 应用版图（此前：PDE/PIDE/committor/epidemiology/data-driven physical laws，现增 turbulence）。两阶段分解（符号+生成模型）可作为 idea.md 未来迭代的设计参考。

## Deep Lit 2026-06-26 (experiment scope, R5)

本次 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-26 执行，覆盖 6 axes × 8 arxiv-tool queries（含 arxiv + S2）+ 5 组 B0 web search + OpenReview 定向搜索 + SIGS/SymPlex/PDEBench/PDEAgent-Bench 验证。

**结果：0 篇新论文。**

基于 322 篇已读 wiki 池（R1-R4 四轮积累）极度饱和。本轮所有搜索返回的候选要么已在 wiki 中（2602.03816 SymPlex、2602.15603 Symbolic PDE Recovery、2512.15920 SR Review），要么与 symbolic macro gate audit 无交集（2605.09636 PDEAgent-Bench 是 LLM FEM 代码生成 benchmark；2506.21275 Embed-Learn-Lift 是 CFD bifurcation analysis；2510.17657 是 manifold learning + SINDy；2510.05178 Logistic-Gated Operators 是 healthcare SR）。web 搜索同样未发现新相关论文（PSR/LLM-SR/SI-SR 等均为已知工作）。

**对实验的启示**：当前文献池已覆盖 symbolic PDE solver / FEX family / SR certification / verification / gate audit 相关方向的所有已知工作。实验下一轮 iteration 应聚焦 reviewer V4 指出的核心问题（CRITICAL-001: 第二 hard PDE target；MAJOR-001: non-perturbative non-exact 证据；MAJOR-002: stronger external baseline），而非继续文献搜索。PSR 代码不可用的 infeasibility receipt 已写；SIGS/NOMTO (ETH Mishra 组) 是当前最接近的符号 PDE baseline，但其方法论（CFG grammar + GVAE）与 FEX macro grammar 正交且未测高维，无法直接用于 matched comparison。

## Deep Lit 2026-06-27 (experiment scope, R6)

本次 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-27 执行，覆盖 6 axes × 13 arxiv-tool queries + 5 组 B0 web search。

**结果：0 篇新论文。**

基于 323 篇已读 wiki 池（R1-R5 五轮积累）极度饱和。本轮所有 arxiv-tool 搜索返回的候选要么已在 wiki 中，要么与 symbolic PDE / FEX dimension lift 完全无关（image SR、gravitational waves、audio codecs 等）。B0 web search 同样未发现新相关论文。

**对实验的启示（不变）**：当前文献调研已达硬饱和——连续两轮 (R5, R6) 0 篇新增。实验应停止文献搜索，聚焦 reviewer V5 指出的核心问题：MUST-001 (第二 hard PDE family)、MUST-002 (Gate-3 ablation)、MUST-003 (stronger executable baseline)、MUST-004 (SR scaling factor 统一)。

## Deep Lit 2026-06-27 (experiment scope, R7)

以下 1 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-27 搜索并精读后收录。本次搜索覆盖 6 axes × 8 arxiv-tool queries + 5 组 B0 web search + 3 组 R2 扩展关键词搜索 + 4 次 B7 反向扩展。基于 324 篇已读 wiki 池极度饱和，仅发现 1 篇新论文。

### ASYS — Agentic Symbolic Search for PDE Characterization (2606.20467)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2606.20467.md)

Zongmin Yu, Liu Yang (NUS), 2026.06. 0 citations.

LLM coding-agent 驱动的先验引导符号搜索框架，用 EvE 演化搜索将 PDE 理论先验转化为可微符号程序，自动发现 PDE 解的显式数学结构。核心创新：(1) 四维评分系统（含首创的 compatibility condition 防止 trivial solution）；(2) 目标分离设计（agent 训练 loss vs evaluator 固定评分）；(3) 纯 equation-residual 驱动（无需观测数据）。在 5 个问题上验证：NLS（恢复呼吸子解 + endpoint correction）、Allen-Cahn 2D（23 参数几何公式描述界面演化）、Keller-Segel（9 参数收缩律超过 49K 参数 SS-PINN, L²=0.188 vs 0.258）、Graveleau（恢复第二类自相似框架）。gCLM 失败（nonlocal Hilbert 变换导致 agent 无法在 10 轮内构建 global reduction），诚实定位了方法边界。

**与本实验关系**：不撞车但高度相关。FEX 用弱先验 grammar-based 搜索，ASYS 用 LLM 强先验搜索——技术路线正交。对实验 MUST-fix 的参考价值：(1) MUST-001 reframe path——ASYS 在 5 个结构性不同 PDE 上测试，为"多 family benchmark"提供参考范本；论文可将 Conservation 定位为 primary case study + 引 ASYS 级别的多问题覆盖作为 scope 对照；(2) MUST-003 baseline 参考——ASYS 是最新的符号 PDE 结构发现方法，其 compatibility condition 和 scale-free relative residual 可参考优化 gate audit 评分设计；(3) FEX 可从 ASYS 直接借用 compatibility condition 评分维度、scale-free relative residual 标准化、五问题 benchmark protocol。

**对实验的启示**：文献池达 324 篇硬饱和（R5/R6 连续 0 新增，R7 仅 1 篇）。ASYS 是最近发表的唯一新增竞品级工作。实验应继续聚焦 V6 reviewer MUST-fix，ASYS 的参考价值在 narrative reframing（MUST-001 path b）和 gate audit 评分改进（MUST-004 FA 定义精化），不改变实验技术路线。

## Deep Lit 2026-06-28 (experiment scope, R8)

以下 1 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 搜索并精读后收录。本次搜索覆盖 6 axes × 2 轮 + B0 web search (5 组) + 4 次 B7 反向扩展。基于 324 篇已读 wiki 池硬饱和，仅发现 1 篇边缘相关论文（conservation law discovery 方向，非 FEX 跨维 lift）。

### NGCG — Neural Discovery of Conservation Laws Without False Positives (2603.20474)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2603.20474.md)

Rahul Ray (BITS Pilani), 2026.03. 0 citations.

四阶段解耦 neural-symbolic pipeline：(1) MLP 独立学习动力学（冻结）；(2) 10-restart variance minimizer 学习近常数 latent representation；(3) 四种系统特定符号提取策略（Polynomial Lasso / Log-Basis Lasso / PDE candidates / PySR fallback）；(4) constancy gate (threshold τ=0.01) + diversity filter (ρ ratio > 10) 消除伪发现。在 9 个系统（4 有守恒律 + 5 无守恒律）上 DR=1.0/FDR=0.0，是唯一在 Lotka-Volterra 上成功的方法。constancy 比最佳基线低 2-3 个数量级。局限：单作者学生论文、代码未公开链接、PDE 降维到 3 个空间矩丢弃细粒度结构。

**与本实验关系**：不撞车——NGCG 做 conservation law discovery（从轨迹数据发现不变量的符号表达式），本实验做 FEX skeleton 跨维 lift + gate audit（从低维 FEX 搜索输出推断 macro 并认证高维 PDE probe）。技术路线正交。方法论参考价值：(1) constancy gate + diversity filter 的 "generator → verifier gate" 架构与 three-gate audit 概念平行——NGCG 的 gate 是 constancy threshold on latent representation，我们的 gate 是 PDE residual threshold on lifted expression；(2) diversity filter 的 ρ 比率（std_i(mean_t(C)) / mean_i(std_t(C)) > 10）是轻量级伪发现检测器，可移植到 gate audit 中辅助区分真实 lift 和 trivial constant；(3) 9-system benchmark 可为 Conservation 以外的 PDE family 测试提供灵感（但需注意这些系统多为 ODE，PDE 仅 Burgers/KS 且经降维处理）；(4) multi-restart 策略（10 次独立初始化 + 选 constancy 最佳者）与我们的 5-seed FEX search 策略在设计原理上一致。

**对实验的启示**：NGCG 不改变实验技术路线或优先级。CRITICAL-001（blind recovery 移除 target-specific macros）和 CRITICAL-002（第二 hard PDE target）仍需实验层面解决，NGCG 未提供直接方案。diversity filter 的 ρ 比率可作为 gate audit 的辅助诊断指标（低成本，无架构变更），建议在下一轮实验中可选集成。

## Deep Lit 2026-06-28 (experiment scope, R9)

以下 1 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 搜索并精读后收录。本次搜索覆盖 6 axes × 2 轮 + B0 web search (5 组) + 4 次 B7 反向扩展。基于 325 篇已读 wiki 池极度饱和，仅发现 1 篇新论文。

### SES — Symbolic Equation Solver: Data-Free Symbolic Regression (2606.07152)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2606.07152.md)

Sergei Garmaev, Vinay Sharma, Olga Fink (ETH/EPFL), 2026.06. 0 citations.

将方程求解形式化为对可微符号模型（EQL 架构）的残差优化——仅从控制方程和初始/边界条件构造损失函数，在配点上最小化方程残差，无需任何配对输入输出数据。三阶段训练：50k epochs 残差优化 → 200k epochs L1 稀疏化+剪枝（每轮剪 25% 最低贡献权重，至少保留 5 个）→ 20k epochs 精调。算子库仅 6 种 {id, const, (.)², exp, tanh, ×}，网络深度固定为 2 层。在 6 类方程上验证（线性方程组、超越方程、非线性 ODE、输运方程、Poisson 方程两种 BC），均恢复与解析解匹配的符号表达式。SES 处于符号回归、PINN、方程求解三岔口：比传统 SR 省去 paired data 依赖，比 PINN 多出显式符号表达。局限：仅概念验证、算子库和网络深度有限、无标准 benchmark 对比、无代码开源承诺、未与 PINN/数值求解器做定量精度比较。

**与本实验关系**：不撞车——SES 是 data-free 范式（仅需方程残差 + BC），本实验是 data-driven FEX RL 搜索 + gate audit。技术路线正交（EQL 梯度优化 vs FEX RL policy gradient）。方法论参考价值：(1) SES 的 "方程残差作为唯一目标函数" 与 gate audit 的 "PDE residual probe" 在概念上同构——两者都利用 PDE 残差作为独立验证信号，不依赖 ground truth 解数据；(2) 三阶段稀疏化训练（残差优化→L1 剪枝→精调）可参考用于 gate audit 的渐进阈值校准——早期宽松允许候选通过，后期收紧到最终阈值；(3) EQL 可微符号架构提供了不同于 FEX RL 搜索的替代符号表达式生成路径——如果 FEX controller 在特定 PDE 上 searchability 不足，EQL 梯度优化可能作为 fallback search 策略；(4) 6 种算子库的极小设计（远小于 FEX 的 7 种 unary + 4 种 binary）说明了 sparse operator set 在方程求解中的可行性，间接支持我们 finite macro grammar 的设计哲学。

**对实验的启示**：SES 不直接解决 CRITICAL-001（blind recovery）或 CRITICAL-002（第二 hard PDE target），但为 gate audit 提供了可微符号架构的替代参照系。三阶段稀疏化策略可启发 gate threshold 的渐进校准设计。SES 的纯方程残差驱动范式佐证了 gate audit 使用 PDE residual 作为独立验证信号的合理性。

## Deep Lit 2026-06-28 (experiment scope, R10)

以下 3 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 搜索并精读后收录。本次搜索覆盖 6 axes × 6 arxiv-tool queries + 5 组 B0 web search + 12 次 B7 反向扩展（3 篇 wiki_written × 4）。基于 326 篇已读 wiki 池极度饱和，仅发现 3 篇边缘相关新论文（均非撞车，均为概念/方法论参考）。

### MOFS — Multi-Operator Few-Shot Learning for PDE Families (2508.01211)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2508.01211.md)

Yile Li, Shandian Zhe (Utah), 2025.08. 1 citation. AAAI 2026 投稿。

多模态框架，目标是用极少示例（J=4）泛化到未见过的 PDE 算子。三阶段：(1) 多任务自监督预训练 FNO 编码器（masked reconstruction + frequency spectrum prediction）；(2) Prompt-Conditioned Few-Shot Supervised Learning（BERT 文本嵌入 + ResNet-18 视觉特征 + 门控融合 + 梯度交叉注意力 + 记忆缓冲区检索）；(3) End-to-End Contrastive Fine-Tuning。在 PDEBench 的 11 个算子上做 leave-one-out 评测，Darcy Flow 全面碾压 FNO/DeepONet/UNet，但 NavierStokes 上 FNO 仍最强。局限：未与 Meta-PDE/LeMON 对比、数据极小（每算子 10 样本）、仅 2D 规则网格、无代码开源。

**与本实验关系**：不撞车——MOFS 是 neural operator（FNO）路线，本实验是 symbolic FEX macro grammar 路线。概念参考价值：(1) MOFS 的 "跨 PDE family 泛化" 目标与本实验的 "跨维度 lift" 在高层问题上有类比——两者都试图复用低维/少样本知识到新设定；(2) MOFS 的实验结果（即便在 neural operator 空间，跨 family 泛化也非平凡——NavierStokes 上 FNO 仍最强）间接支持我们的 "FEX 跨维窗口极窄" 发现——跨 PDE family/dimension 泛化本身就是困难问题，不是我们方法的 weakness；(3) 记忆增强 retrieval + 渐进对比学习 curriculum 可参考用于 macro library 的扩展策略。

**对实验的启示**：MOFS 为 CRITICAL-001（single hard target → narrow finding framing）提供外部参照——跨 PDE 泛化困难是领域共性，非本实验特有。论文可引用 MOFS 作为 "cross-PDE generalization is an open challenge" 的证据，将 Conservation 的窄窗口定位为这一广泛挑战在 FEX 领域的具体表现。不改变实验技术路线。

### Neural ODE-Transformer for Conservation Law Discovery (2511.00102)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2511.00102.md)

Vivan Doshi, 2025.11. 1 citation. NeurIPS 2025 MATH-AI Workshop。

三模块混合框架：(1) Neural ODE 从含噪声轨迹数据学习连续向量场；(2) Transformer + PPO 强化学习在学到的向量场上生成符号候选不变量；(3) 符号-数值验证器在 10,000 grid points 上检查梯度点积，认证候选者。在 harmonic oscillator/pendulum/Kepler 上显著超越 PySR 和端到端 Transformer。局限：仅 toy 系统、单作者 workshop paper、代码未公开。

**与本实验关系**：不撞车——守恒律发现（从轨迹数据发现不变量）vs 本实验的 FEX skeleton 跨维 lift。方法论参考：(1) "先学 NN proxy 再符号搜索" 的 denoising 范式可引入 FEX 处理噪声数据场景；(2) 数值验证器的 grid-based verification 与 gate audit 的 PDE residual probe 概念同构；(3) PPO reward 中的非退化惩罚项可改进 FEX 候选多样性。NGCG (2603.20474) 是同一方向的更强后续工作（已在 R8 收录）。

**对实验的启示**：不改变实验优先级。作为 "verifier-style symbolic methods are gaining traction" 的领域趋势证据，论文可在 related work 中简要引用。

### Deep NN for High-Dimensional Parabolic PDEs — Survey (2601.13256)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2601.13256.md)

Wenzhong Zhang, Zhenyuan Hu, Wei Cai, George Karniadakis, 2026.01. 0 citations。

教程/综述论文，将高维抛物型 PDE 的神经网络求解方法组织为三大范式：(A) PDE 残差方法（PINN/SDGD/HTE/STDE）；(B) 随机方法（Deep BSDE/FBSNN/DeepMartNet）；(C) 混合随机差分方法（Shotgun/DRDM）。在 HJB 和 Black-Scholes 方程 d=100/1000 上 benchmark，误差 10⁻²~10⁻³。代码开源。

**与本实验关系**：不撞车——纯综述，无新方法。参考价值：(1) 三大策略分类框架是很好的 method positioning 参考坐标系——FEX 可定位为 "不依赖自动微分的符号替代方案"，与三大范式互补；(2) SDGD/HTE 框架的 "随机维度梯度下降" 思路可降低高维 FEX probe 的自动微分成本（如果未来扩展到 d>100）；(3) Karniadakis（PINN 发明人）的权威综述可作为高维 PDE 求解 landscape 的权威引用。

**对实验的启示**：不改变实验优先级。论文中可作为 "high-dimensional PDE solving landscape" 的权威 survey 引用，帮助定位 FEX gate audit 在 broader ecosystem 中的位置。

## Deep Lit 2026-06-28 (experiment scope, R11)

本次 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 执行，覆盖 6 axes × 6 arxiv-tool queries + 5 组 B0 web search。

**结果：0 篇新论文。**

基于 329 篇已读 wiki 池（R1-R10 十轮积累）极度饱和。本轮所有 arxiv-tool 搜索返回的候选要么已在 wiki 中（如 2604.17402 GenBounds SR、FEX 全系列、SIGS/PROSE/ASYS 等），要么与 symbolic PDE / FEX dimension lift 完全无关。B0 web search 同样未发现新相关论文（PCSRL 来自非 arXiv 期刊且无法经 arxiv-tool 验证）。

**对实验的启示（不变）**：文献调研已达硬饱和——R5/R6/R11 三轮 0 篇新增。实验应停止文献搜索，聚焦 V10 reviewer MUST-fix：
- MUST-001: sin×depth2_sub 特殊性的 semi-formal proposition（从已有 14-combo 穷举 JSON 形式化推导）
- MUST-002: Pipeline "激活维度" d≥50 标注
- MUST-003: 12/87 false rejects tie-break 分析
- MUST-004: FA 定义命名统一

这些问题的解决方案不在文献中，需从实验内部数据推导和形式化。

## Deep Lit 2026-06-28 (experiment scope, R12)

以下 3 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 搜索并精读后收录。本次搜索覆盖 6 axes × 3 轮 + B0 web search (5 组) + 12 次 B7 反向扩展（3 篇 wiki_written × 4）。基于 329 篇已读 wiki 池极度饱和，仅发现 3 篇新论文（均非撞车，均为概念/方法论参考）。

### LawMind — Law-Driven Symbolic Discovery of Analytical PDE Solutions (2603.14353)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2603.14353.md)

Min-Yi Zheng, Shengqi Zhang, Liancheng Wu, Jinghui Zhong, Shiyi Chen, Y. Ong, 2026.03. 0 citations.

从 PDE 控制方程出发，不依赖任何数据，通过结构化符号探索 + 物理约束评估自主构造闭式解析解。在 100 个 benchmark PDE（两本权威手册中收录）上全部成功恢复闭式解析解，并发现若干此前未知的线性和非线性 PDE 解析解。核心机制：(1) 符号探索引擎——在预定义原语（多项式、三角函数、指数函数等）构成的表达空间中系统搜索；(2) 物理约束评估器——用 PDE 残差 + 边界条件验证候选解；(3) 渐进组装——从简单组件开始逐步构造完整解。

**与本实验关系**：不撞车（LawMind 是 law-driven data-free 范式，本实验是 data-driven FEX RL 搜索 + gate audit）。但高度相关：(1) LawMind 的 100-benchmark PDE 覆盖展示了符号 PDE 解析解空间的广度，间接支持单 hard target 的"窄窗口是发现不是局限"叙事——即便专门的符号搜索框架也需要 100 个 benchmark 来验证，在 FEX 算子集中找到 1 个 hard target 是合理的；(2) 为 V11 MUST-003 amortization 提供参照系：LawMind 每个 PDE 独立搜索，无跨维复用机制，而 FEX pipeline 的低维搜索→高维参数优化本质是 amortized search——可引用为"无 amortization 的全符号搜索成本"对比点；(3) 100 benchmark PDE 列表可作为 Conservation 以外潜在 hard target 的候选池——如果扩展 FEX 算子集后测试这些 PDE 中的难例。

### Adaptive Correction — Conservation Laws in Neural Operators (2505.24579)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2505.24579.md)

Chaoyu Liu, Yangming Li, Zhongying Deng, Chris J. Budd, C. Schonlieb (Cambridge DAMTP), 2025.05. 3 citations. ICML 2026 投稿。

提出 plug-and-play 可学习自适应修正算子，为 FNO/UNet/GTNO 神经算子输出强制执行线性和二次守恒律。理论证明修正不损害表达力且可能降低重构损失。跨 6 PDE、3 架构验证，守恒误差达机器精度 (0.00)，一致提升预测精度。剑桥应用数学组工作。

**与本实验关系**：不撞车（神经算子格点修正 vs FEX 符号回归）。但跨范式共鸣：(1) 引言明确指出神经算子"often fail to ensure conservation"——与 FEX 发现 scratch FEX d≥50 全 fail 形成跨范式呼应；(2) "plug-and-play"设计哲学（不改架构、加一层保证）与 Two-Gate audit（不改 FEX 核心、加 certifying layer）方法论共鸣；(3) 为 V11 MUST-002 broader impact 论证提供权威引用——conservation enforcement 是 data-driven PDE solver 的普遍挑战，非 FEX 特有弱点。

### FISolver — Learning First Integrals via Backward-Generated Data and GRPO (2605.21160)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2605.21160.md)

Jingfeng Zhong, Zheng Liu, Zhijie Wang, Shuai Li, 2026.05. 0 citations.

基于 LLM 的首次积分自动发现系统。核心："Backward Generation"——从随机采样的首次积分出发，反向求解兼容微分方程，构建大规模 (DE, first integral) 训练对。用 Qwen2.5-Math-1.5B 做 LoRA SFT + GRPO RL 微调（Levenshtein Distance 塑形奖励）。1.5B 模型在 Hard 测试集上 63.7% 准确率，远超 Mathematica (23.3%) 和 DeepSeek-V3.2-Exp (42.8%, 685B 参数)。Data Synthesis + Dataset Blending 策略支持 domain adaptation 到特定 ODE 族。

**与本实验关系**：不撞车（ODE 首次积分发现 vs PDE 跨维 lift）。方法论参考：(1) Backward Generation 范式——"从答案反推问题"的数据构造策略——可迁移到 FEX 场景：从已知 macro 反推 FEX 应搜到的表达式分布，验证 searchability；(2) Predict-and-Verify 架构（beam search 生成候选 + 符号验证 dV/dt=0）与 gate audit 的"生成 + 验证"模式同构；(3) LD reward shaping（连续信号替代稀疏二元奖励）可启发 FEX RL controller 的奖励设计改进；(4) Dataset Blending 的 domain adaptation 策略可参考用于 FEX 跨 PDE family 的 controller 迁移。

**对实验的启示（不变）**：文献调研已达硬饱和——R5/R6/R11/R12 四轮仅发现边缘相关论文。实验应聚焦 V11 reviewer MUST-fix（方法贡献/形式化/broader impact/amortization），这些问题的解决方案不在文献中。R12 新增 3 篇仅提供 narration 支持和跨范式 context citation，不改变实验技术路线或优先级。

## Deep Lit 2026-06-28 (experiment scope, R13)

以下 10 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 搜索并精读后收录。本次搜索覆盖 6 axes × 12+ arxiv-tool queries + 5 组 B0 web search + 40 次 B7 反向扩展（10 wiki_written × 4）。基于 332 篇已读 wiki 池极度饱和，经 2 轮循环发现 10 篇新论文（均非撞车，均为概念/方法论/benchmark 参考）。

### ERBench — Equation Recovery Benchmark (2606.09276)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2606.09276.md)

Paul Kahlmeyer, Henrik Voigt, Michael Habeck, Joachim Giesen (Jena), 2026.06. 0 citations.

面向方程发现的 SR benchmark，核心创新是评估从 in-domain 预测精度转向公式恢复（equation recovery）。包含 10,000 公开公式 + 1,000 秘密评估公式（Feynman/OEIS/Wikipedia + SynEq 合成），评测 6 种算法，仅 PySR 达 29% 恢复率。引入 Jaccard Index 和树编辑距离作为新指标，诊断表明表达式复杂度是恢复率的主要瓶颈。

**与本实验关系**：不撞车（纯 benchmark）。参考价值：(1) ERBench 的 equation recovery 评估框架可迁移到 gate audit 的 macro correctness 指标——我们的 macro correctness FA 本质就是"macro recovery"问题；(2) 秘密测试集设计防止过拟合，可参考用于 gate threshold 的 held-out calibration；(3) 树编辑距离作为符号结构距离度量，可参考用于 gate audit 的 macro ambiguity 定量化。

### LLM-SRBench — Scientific Equation Discovery Benchmark with LLMs (2504.10415)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2504.10415.md)

P. Shojaee, Chandan K. Reddy 等 (Virginia Tech/CMU/VinUniversity), 2025.04. 47 citations. ICML 2025.

239 道题的方程发现 benchmark，分 LSR-Transform（111 题，对 Feynman 方程做变量变换防 LLM 背诵）和 LSR-Synth（128 题，跨 4 领域的合成新项）。最佳系统仅 31.5% 符号准确率。核心主张：现有 benchmark 依赖已知方程导致 LLM 背诵而非推理。

**与本实验关系**：不撞车（LLM benchmark）。参考价值：(1) "LLM 背诵而非推理"的批判逻辑可移植到 gate audit——reviewer 质疑 gate audit 只是在做 exact-library lookup 时，可引用 LLM-SRBench 的防背诵方法论论证 macro grammar 不是 lookup table；(2) Transform/Synth 二分设计可参考用于 gate audit 的 non-exact 证据设计——将已知 PDE family 变换后测试 gate 是否仍正确分类。

### Anant-Net — Breaking Curse of Dimensionality (2505.03595)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2505.03595.md)

S. S. Menon, Ameya D. Jagtap (Brown), 2025.05. 14 citations. CMAME accepted.


**与本实验关系**：不撞车（神经 PDE solver vs symbolic macro gate audit）。但高度相关：(1) Anant-Net 在 d=300 的成功是 pipeline 当前无法企及的维度——我们的 pipeline d=100 是上限；(2) active dimension sampling 策略可参考用于高维 PDE probe 的效率优化——当前 probe 在全维度配点上评估 PDE residual，可考虑重要性采样；(3) Anant-Net 的 benchmark results 可作为外部高维 PDE solver baseline 的性能参考点。

### The Need for Verification in AI-Driven Scientific Discovery (2509.01398)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2509.01398.md)

Cristina Cornelio, Takuya Ito, Ryan Cory-Wright, L. Horesh (Samsung AI/IBM Research/Imperial), 2025.09. 4 citations.

综述/立场文章：AI 可高速生成科学假设，但验证管道跟不上，导致未验证假设堆积反而阻碍科学进步。沿 Data-Driven/Knowledge-Aware/Derivable Models 三维组织方法图谱。核心论点：验证必须成为 AI 辅助科学发现的基石。

**与本实验关系**：不撞车（综述）。但为 broader impact narrative 提供最权威引用：(1) 论文的"verification as cornerstone"论点直接支持 gate audit 的核心贡献叙事——我们的 gate audit 就是 AI-driven PDE discovery 的 verification layer；(2) 将 gate audit 定位在论文的 Derivable Models 象限（generator-verifier paradigm）；(3) 作者的 IBM/Samsung 背景增加引用权威性；(4) 论文总结了当前 verification 方法的不足，gate audit 可定位为填补 symbolic SR verification 空白的方案。

### Learn and Verify — Rigorous Verification of PINNs (2601.19818)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2601.19818.md)

Kazuaki Tanaka, Kohei Yatabe (TUAT, Japan), 2026.01. 1 citation.

"Learn and Verify" 框架：Learn 阶段用 DSM loss 训练 sub/super-solution（上下界），Verify 阶段用区间算术（INTLAB）严格验证微分不等式。理论上证明分片 C1 sub/super-solution 的比较原理和 global-in-time 延拓。在非线性 ODE 上构造严格 enclosure，包括有限时间 blow-up 问题。

**与本实验关系**：不撞车（PINN 形式验证 vs symbolic gate audit）。方法论参考：(1) "Learn and Verify" 两阶段范式与 gate audit 的 "search + verify" 在概念上同构——gate-1/2 是 search quality verify，gate-3 是 PDE quality verify；(2) 区间算术的严格误差界为 C3 条件的形式化提供数学工具参照——可考虑用区间算术形式化 FEX searchability 的数值边界；(3) sub/super-solution 上下界构造可启发 gate threshold 的双侧 bound 设计（当前仅 upper bound rel_l2 < 1e-3）。

### Bias Inheritance in Neural-Symbolic Discovery (2604.01335)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2604.01335.md)

Han-Yu Liang, Z. Tao, Fujun Liu, 2026.04. 0 citations.

三阶段 neural-symbolic 框架：数值代理学习 → 符号压缩 → 前向重仿真验证。核心发现是 Bias Inheritance：符号压缩不会修复神经网络代理的本构偏差，BIR 在所有设定下均接近 1。matched-library 下多项式 baseline 极强。

**与本实验关系**：不撞车（reaction-diffusion 本构闭包 vs FEX PDE skeleton）。但发现与 gate audit 有深层共鸣：(1) Bias Inheritance 与我们的 "searchability bottleneck" 平行——BIR~1 意味着符号压缩不修复上游错误，正如 gate-1 搜索失败不能被 gate-3 probe 修复；(2) 论文的最终建议"forward validation is essential, not residual minimization alone"直接支持 gate-3 PDE probe 的必要性（仅靠 low-d fit 不够）；(3) 三阶段设计与 three-gate cascade 在结构上高度同构（数据→代理→符号→验证 vs 搜索→macro→fit→probe）。

### No-Harm Physics-Informed Inverse Learning (2606.07153)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2606.07153.md)

Ronald Katende (Kabale University, Uganda), 2026.06. 0 citations.

"No-harm" 认证-选择框架：学到的逆问题重建只有在残差校准认证半径不差于基线时才能替换基线，否则退回基线。认证半径由四个残差分量（数据/PDE/边界/优化）组合而成。在 Poisson 源恢复、逆热传导、有限角 CT、椭圆系数识别上验证。保守性在强不适定区域自动增加。

**与本实验关系**：不撞车（逆问题认证 vs 符号表达式跨维认证）。但框架设计哲学高度共鸣：(1) "No-Harm" = 我们的 "0 lift-quality-FA"——两者都是安全优先的认证设计；(2) 四个残差分量的组合认证与 three-gate cascade 的多维度评估逻辑同构；(3) "学到的候选只有通过认证才替换 baseline"与 gate audit 的 "lifted expression 只有通过 gate-3 PDE probe 才接受" 对应；(4) 保守性自动增加机制可启发 gate threshold 的自适应校准——对 harder families 自动收紧阈值。

### CHONKNORIS — Operator Learning at Machine Precision (2511.19980)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2511.19980.md)

Aras Bacho, H. Owhadi 等 (Caltech), 2025.11. 4 citations.


**与本实验关系**：不撞车（神经算子 vs 符号回归）。但"machine precision"叙事共鸣：(1) CHONKNORIS 的 "machine precision via numerical analysis structure" 与 pipeline 的 "d=100 1e-6 via macro grammar structure" 共享"结构先验达高精度"的设计哲学；(2) Cholesky 因子学习（学习更简单的线性椭圆算子而非复杂非线性映射）与 macro inference（推断简单 macro structure 而非搜索完整高维表达式）类比；(3) FONKNORIS 的 cross-PDE 泛化（3 个 PDE 训练→未见 PDE）与 pipeline 的 cross-dimension lift（d=3 搜索→d=100 probe）在 "amortized knowledge transfer" 层面平行。

### SRBench 2.0 — Call for Action (2505.03977)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2505.03977.md)

Aldeia, Cranmer, La Cava, de França 等 (SR 社区核心作者群), 2025.05. 13 citations.

SRBench 社区驱动的 living benchmark 更新：方法从 14 扩充至 25 种，双轨道数据集（black-box PMLB + first-principles 物理关系），Dolan-More performance profile 替代聚合指标。核心发现：无单一算法占优，参数优化是区分 top performer 的关键，超参数调优未必优于默认值。

**与本实验关系**：不撞车（benchmark 论文）。但为 CRITICAL-001（SR random d=100）提供关键参考：(1) SRBench 2.0 的 benchmark protocol（30 次独立运行、performance profile、Pareto front）可作为 SR random d=100 实验的设计模板；(2) 论文包含 SR 算法的维度 scaling 特征——哪些算法在高维衰退、哪些保持稳定——可预判 SR random d=100 的可行性；(3) 社区驱动的 "living benchmark" 理念可启发 gate audit 的标准化评测协议设计；(4) 作者群包含 Cranmer (PySR)、La Cava (SRBench)、de França (IT/eggp) 等 SR 领域核心人物，增加引用权威性。

### AgenticSciML — Multi-Agent Scientific ML Discovery (2511.07262)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2511.07262.md)

Qile Jiang, George Karniadakis (Brown University), 2025.11. 12 citations.

10+ 专业化 LLM agent 协作发现 SciML 策略：Proposer/Critic/Engineer/Retriever/Evaluator 通过结构化辩论 + 70 条 Knowledge Base + 进化树搜索迭代生成方案。在 6 benchmark 上超越单 agent 10x-11,169x。发现策略包括 adaptive mixture-of-experts、decomposition-based PINNs。

**与本实验关系**：不撞车（multi-agent SciML automation vs symbolic gate audit）。但提供领域方向参照：(1) Karniadakis（PINN 发明人）的最新方向从 PINN 转向 agent-based SciML——暗示 SciML 领域正在从手工设计转向自动化发现，FEX gate audit 可定位为 "symbolic SciML 的自动化验证层"；(2) AgenticSciML 的 structured debate + evolutionary tree search 架构可参考用于 FEX controller 的搜索策略改进；(3) Knowledge Base retrieval 机制可启发 macro library 的动态扩展——根据 PDE 特征检索相关 macro 而非固定 library。

**对实验的启示**：本轮新增 10 篇论文全部不撞车—均属于 benchmark/verification/conceptual-reference 类别。文献调研在 332→342 wiki 池的基础上继续高度饱和。核心启示：
- **CRITICAL-001 (SR d=100)**：SRBench 2.0 (2505.03977) 和 ERBench (2606.09276) 提供 benchmark protocol 参考，但 SR random d=100 的可行性仍需实验验证——benchmark 论文本身不包含 d=100 数据。
- **MAJOR-002 (C3 形式化)**：Learn and Verify (2601.19818) 的区间算术严格验证和 No-Harm (2606.07153) 的残差校准认证提供形式化工具参照，但 C3 的数学 formalization 仍需从 FEX RL dynamics 内部推导——外部验证框架仅提供方法论类比。
- **MAJOR-003 (broader impact)**：Need for Verification (2509.01398) 是最权威的引用来源——IBM/Samsung 团队的立场文章论证"verification as cornerstone"，可直接支撑 broader impact 叙事。AgenticSciML (2511.07262) 显示 Karniadakis 组转向 agent-based SciML，为领域趋势提供 context。
- **MINOR-004 (cross-domain)**：Anant-Net (2505.03595) 和 CHONKNORIS (2511.19980) 提供外部高维 PDE solver baseline 参考，但不改变实验技术路线。

**实验应继续聚焦 V12 reviewer MUST-fix，这些问题的解决方案主要在实验内部而非文献中。**

## Deep Lit 2026-06-28 (experiment scope, R14)

以下 1 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-06-28 搜索并精读后收录。本次搜索覆盖 6 axes × 15+ arxiv-tool queries + 5 组 B0 web search + Yang/Liang/Wang author chase + SIGS/FEX cited-by + 4 次 B7 反向扩展。基于 342 篇已读 wiki 池硬饱和，仅发现 1 篇边际相关论文（SINDy 参数感知发现方向，非 FEX 跨维 lift）。

### Parameter-Aware Ensemble SINDy (2508.14085)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2508.14085.md)


提出参数感知集成 SINDy 框架，从多参数仿真数据中自动发现可解释 PDE 和亚格子应力（SGS）闭合模型。四个核心增强：(1) 参数感知候选库——将物理参数作为符号变量纳入回归；(2) 量纲相似性滤波器（DSF）——利用维度向量筛选候选 term，典型缩减库规模 1-2 个数量级；(3) Gram 矩阵增量累积——内存从 O(np) 降至 O(p²)；(4) 集成共识与系数稳定性分析——双阈值（consensus frequency + CV）筛选确保鲁棒识别。在 1D Heat/Burgers/KdV-Burgers 上验证精确 PDE 恢复，系数标准差 10⁻⁹~10⁻⁸。关键发现：集成方法必须配合 DSF 才能鲁棒工作。

**与本实验关系**：不撞车——SINDy 稀疏回归 vs FEX RL 树搜索，技术路线正交。方法论参考价值有限：(1) "集成共识 + 系数稳定性"机制与 Gate-1 的多 seed 搜索稳定性在概念上平行——两者都使用跨运行一致性作为可靠性判据；(2) 量纲相似性滤波器（DSF）的维度分析策略可参考用于 macro grammar 的物理约束扩展——但 SINDy 的线性项筛选与 FEX 的非线性表达式树推理机制不同；(3) Gram 增量累积的内存优化技巧可参考用于大规模 gate sweep 的效率改进。

**对实验的启示**：不改变实验技术路线或优先级。2508.14085 是 SINDy/turbulence 方向的工程改进，与当前 V14 reviewer must-fix（C3 overclaim 措辞修复、第二 hard PDE target、Claims 速查修正）无直接交集。文献调研在 342→343 wiki 池基础上继续硬饱和。实验应聚焦 reviewer 文本级修复，这些问题的解决方案不在文献中。

## Deep Lit 2026-07-01 (experiment scope, R15)

以下 1 篇论文由 `deep-lit-tick --scope experiment fex-dim-lift-skeleton` 于 2026-07-01 搜索并精读后收录。本次搜索覆盖 6 axes × 10+ arxiv-tool queries + 5 组 B0 web search + 4 次 B7 反向扩展（1 wiki_written × 4）。基于 343 篇已读 wiki 池极度饱和，仅发现 1 篇新论文。**核心发现：learnable lowd-highd transfer for symbolic expressions 是真正未被探索的方向——文献空白对实验是好消息。**

### UniSymNet — Unified Symbolic Network with Sparse Encoding and Bi-level Optimization (2505.06091)

[wiki](/home/youran/AutoResearch/agent-factory-data/wiki/2505.06091.md)

Xinxin Li (ECNU), Juan Zhang (Beihang), Da Li (IAPCM) 等, 2025.05. 0 citations.

提出 UniSymNet，核心创新是 Ψ 变换——用嵌套 unary 算子 (ln, exp) 统一表示二元非线性算子 (×, ÷, pow)，将二元算子扩展为多元算子，降低表示复杂度并增强表达能力。理论证明 UniSymNet 在特定条件下比 EQL-type 网络深度和节点数更低。Bi-level 优化框架：外层预训练 Transformer（encoder-decoder 架构 + sparse label encoding）学习从数据到网络结构的映射；内层三种参数优化策略（SFO: Gumbel-Softmax + BFGS + risk-seeking policy gradient，符号解率最高 33.5%；DNO-NP: 梯度下降无剪枝，拟合 R² 最高；DNO-P: 有剪枝，复杂度最低）。在 SRBench (Feynman + Strogatz，d 2-10) 上符号解率 56.9%（噪声=0），超所有 14 种对比方法。在 low-dim Standard Benchmarks 上符号解率均为最高。SFO 展现最强噪声鲁棒性。无公开代码。

**与本实验关系**：不撞车——UniSymNet 做同维符号回归/表达式发现，非跨维 lift。但方法论参考价值极高——直接对应 2026-07-01 人类决策的三个方案层级：(1) **大脚豆（leaf mixing）对应项**：UniSymNet 的 Ψ 变换本质是将复杂算子操作转化为更简单的原语组合，与 leaf mixing 的"用线性组合替换变量"思路在"用更简单原语表达复杂结构"这一哲学层面相同；(2) **屁股（tree insertion）对应项**：UniSymNet 的 bi-level 优化（structure search → parameter optimization）提供了在表达式树不同层级引入自由度的模板——可以类比为：外层搜索 transfer 候选结构，内层优化 transfer 参数；(3) **有脑子（learnable functor）对应项**：UniSymNet 的 pre-trained Transformer 学习"从数据到结构"的映射，可类比为学习"从低维树到高维树"的映射。具体可复用技术：(a) sparse label encoding 可用于紧凑表示 transfer candidate space；(b) Gumbel-Softmax + BFGS + policy gradient 组合可用于 transfer candidate 的离散-连续混合优化；(c) beam search 结构候选生成可启发 transfer candidate 的 top-k 筛选。

**关键局限**：UniSymNet 不处理维度变化——输入维度固定，不涉及跨维度结构迁移。所有实验在 d≤10 范围。**对实验的启示**：确认 learnable lowd-highd transfer 是文献空白。UniSymNet 是最近的 methodological neighbor，可从其 Ψ 表示、bi-level 优化、sparse encoding 三个模块分别汲取灵感，但不能直接解决 lowd-highd transfer 问题。
