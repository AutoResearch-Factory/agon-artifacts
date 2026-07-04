---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-search-complexity/
---

- One-sentence summary: 为 FEX 的 RL controller 在 Barron-like 函数类上建立搜索复杂度理论 -- 正面给出 poly(d, 1/eps) 收敛定理, 或反面给出到 3-SAT/QBF 的归约.
- Hypothesis: FEX 的 RL 搜索在有结构的函数类 (如 Barron 函数、有限 Fourier 展开) 上具有多项式收敛率, 因为这些函数类的表达式空间有效维度远低于一般情形; 若此假设不成立, 则 FEX 搜索在一般情形下具有与 SR 相同的 NP-hard 复杂度.
- Expected outcome: 成功: 在 Barron-like 函数类上证明 FEX 的搜索步数为 poly(d, 1/eps, k), 利用 Liu-Yang-Hayou (2202.10670) 的 Uniform-LGI 机制; 失败: 构造 FEX 搜索到 QBF 的归约, 证明一般情形下 FEX 搜索是 PSPACE-hard. 无论哪个方向都是 seminal 结果.
- Contribution type: theory
- Risk: HIGH
- Estimated effort:
  - Compute: 0 GPU-hours (纯理论)
  - Data: available (使用已有 PDE benchmark 做验证)
  - Implementation: weeks-months (证明推导)
- Novelty quick-check: Na & Yang (2502.05360) 证明了 NN 优化的 CoD, Virgolin (2207.01018) 证明 SR 是 NP-hard, 但无人针对 FEX 特定的 RL-controller-on-expression-trees 结构建立过收敛率或 hardness 结果. FEX 的树结构约束和 policy gradient 更新使其与一般 SR 不同.
- Strongest objection: Barron-like 函数类的证明可能过于受限, 对实际 PDE 求解的指导意义有限.
- Why we should do this: 这是 Yang 整个学术脉络中最大的理论缺口 -- FEX 有 approximation theory, Na & Yang 有 optimization lower bound, 但 FEX 搜索算法本身的复杂度完全空白. 填补此缺口对理解 symbolic PDE solving 的根本能力边界有决定性意义.
- Pilot:
  - Setup: 在 d=2,4,6,8,10 的 Poisson 方程上测量 FEX 搜索步数与 d 的关系, 拟合 log(steps) vs d 判断是否指数增长
  - Metric: 若 log(steps) vs d 的斜率 < 0.5 (即步数增长 < exp(0.5d)), 视为正面信号支持多项式收敛
  - Result: pending
  - Signal: SKIPPED

<review date="2026-06-16">

## Novelty

- Score: 7/10
- Closest prior work:
  - **Approximation theory of FEX**: Liang & Yang, *Finite Expression Method* (arXiv:2206.10121, JMLR 2025) — 只证 S_k 对 Holder 函数 *存在* 无 CoD 表达式 (existence / approximation), 没有任何关于 RL controller *找到* 该表达式的 step / sample 复杂度. FEX-PG (arXiv:2410.00835) 的 "PG" 是 parameter-grouping 不是 policy-gradient theory, 纯数值. 所有 FEX 后续 (2305.08342 / 2306.12268 / 2401.03092 / 2510.22497 等) 均无 controller 复杂度结果. **搜索复杂度缺口确实完全开放.**
  - **SR hardness**: Virgolin & Pissis, *Symbolic Regression is NP-hard* (arXiv:2207.01018, TMLR 2022) — 从 **Unbounded Subset Sum** 归约, 但只覆盖 linear-sum 类 (f = Σ x_j m_j), ε=0 精确插值, 且把 ephemeral constant 强制为 0 (即 **故意绕开实值内层优化**). 第二个独立证明 Song et al. *Prove SR is NP-hard by Symbol Graph* (arXiv:2404.13820, 2024) 覆盖完整 nested 算子空间, 更接近 FEX 树结构, 但仍是 data-fit SR, 非 PDE-residual, 非 RL.
  - **NN 优化 CoD**: Na & Yang (arXiv:2502.05360) 证的是 **2-Wasserstein gradient flow (mean-field)** 下 shallow NN 的 population risk 衰减不快于 t^{-4r/(d-2r)} — 纯 **连续梯度流**, 对 discrete / RL / 树搜索 *只字未提*. 它是同一 (Yang) 学术脉络里 "optimization lower bound" 的 template, 但留下 discrete-search 类比完全空白, 是 idea 最该对照的方法论先例 (连续-NN vs 离散-RL 对比).
  - **Policy-gradient 收敛理论 (正面方向真正的工具)**: Mei-Xiao-Szepesvari-Schuurmans 2020 (arXiv:2005.06392, softmax PG 的 **non-uniform Łojasiewicz** ⇒ O(1/t), 加 entropy ⇒ 线性); Agarwal-Kakade-Lee-Mahajan 2021 (PG 的 gradient domination / PL); Bhandari-Russo 2019 (arXiv:1906.01786). 这些才是 "LGI ⇒ poly-step PG rate" 的正确 canon.
  - **并发风险 (negative 方向)**: Zixi Li, *Reasoning: From Reflection to Solution* (arXiv:2511.11712, 2025-11) 提出 OpenXOR, 已给 **3-SAT 归约 + neural-guided/beam search 在 B ≪ 2^k 时失败** 的 hardness. 但针对 LLM reasoning / XOR-SAT, **不是 FEX / 不是 PDE expression search / 不是 policy-gradient controller**. 是强相关引用与 negative 分支的部分先例, 不构成 scoop, 但下一版必须显式区分.
- Key differentiator: 经两路独立 web 核验 (arxiv / Scholar / S2 / OpenReview, 含最近 6 个月), **没有任何论文对 FEX RL controller (或一般 RL/MCTS expression-tree search) 在结构化函数类上给出 step/sample 复杂度或针对 FEX-PDE-search 的 hardness 归约**. 甚至通用 tree-search policy-gradient 论文 (arXiv:2506.07054 PGTS) 自己把 "establishing convergence rates" 列为 open future work, 旁证缺口真实. **目标问题 (gap) novelty HIGH**; 但两个分支各有 novelty 折损 (见下), 故综合 7。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Problem fidelity (gap 真实性) | 9/10 | 缺口经独立双路核验确为开放, 且是 Yang 脉络 (approximation theory 有 / optimization lower bound 有 / **search complexity 空白**) 最自然的下一块拼图. topic 文件对 [6][7] 的引用 (Na-Yang CoD, Hardwick-Yang "computational CoD" 原话) 精准, problem 锚定扎实. |
| 正面方向 proof-bridge 健全性 | **3/10** | **最严重问题.** idea 把正面证明寄托于 "利用 Liu-Yang-Hayou 2202.10670 的 Uniform-LGI 机制" — 但 2202.10670 是 **supervised gradient-flow → generalization** 理论, 作用于 *连续 NN 参数*, 全文无 RL / policy gradient / REINFORCE / combinatorial 内容. 这是 **citation mismatch / 近乎 category error**: (a) LGI 只控 "给定 landscape 的优化速度", 而 FEX 搜索是 *离散树结构空间*, 那里没有梯度; 唯一能套 LGI 的对象是 PG 的 *expected-reward J(Φ)* 连续松弛, 但 idea 没说这一步; (b) 即便对 J(Φ), 需要为 FEX **非标准 top-quantile 目标** + **内层 BFGS 噪声 reward** 证一条 *新的* gradient-domination 引理且常数 poly(d), 现有 softmax-PG 结果 (Mei 2020 的 non-uniform LGI 常数会退化、初值依赖) 不能直接搬; (c) 关键: "需枚举多少离散结构才命中 ε-好表达式" 是 **covering / effective-dimension** 命题, LGI *根本不提供*, idea 把它混入 "LGI 机制" 一词。**修复方向**: 把 2202.10670 换成 Mei 2020 + Agarwal 2021 做承重工具, 并把 (i) FEX-quantile-目标的 gradient-domination 引理 + (ii) Barron 类的 covering 引理 显式列为两条 *待证* 新 lemma。 |
| 负面方向 reduction 可行性 | **4/10** | QBF / PSPACE 主张 **概念上站不稳**: FEX 搜索是 *存在型* (∃ 算子序列 ∃ 参数 s.t. residual 小), 是 ∃-shaped (NP- 或 ∃R-flavored), **不是 ∀∃ 交替** — 没有产生 quantifier alternation 的对手结构, QBF/PSPACE 不自然落地. 且实值 BFGS 内层使 *干净离散 verifier* 更难: Virgolin-Pissis 之所以能归约, 正是把空间限制成整数 linear-sum + ε=0 + 强制常数为零, **恰好扔掉了让 FEX 成为 FEX 的实值内层优化**. 可防御的负面结果是 (i) Virgolin 式 NP-hardness (但只证 *退化* FEX, 且与两篇已有 SR-hardness 重叠, novelty 仅 MEDIUM/LOW), 或 (ii) **∃R-hardness** (保留实值内层, 继承 existential-theory-of-reals 的 hardness). PSPACE/QBF 应放弃或需给出明确交替 gadget 论证. |
| Contribution 聚焦 | 7/10 | "正面 poly 定理 OR 负面 hardness" 的 dichotomy framing 干净有力, 单一 dominant 贡献 (FEX 搜索的复杂度边界). 但当前两个分支的承重工具都需更换/补强, framing 强而 *弹药未到位*. |
| Frontier leverage | 7/10 | 用 FEX (RL-on-expression-trees) 这一 foundation-era primitive 做复杂度理论是合时宜的; 但真正该 leverage 的 frontier 工具是 PG-convergence canon (Mei/Agarwal), idea 误引了 generalization 论文。 |
| Feasibility (理论可交付性) | 5/10 | 0 GPU 成本属实, 但 "weeks-months 证明推导" **低估**了正面方向的真实难度: 要同时拿下 (a) 非标准 RL 目标的 gradient-domination + poly(d) 常数, (b) Barron 类 covering, (c) 把 per-iter rate 映成 poly(d,1/ε) search-step count — 任一条都可能是独立 paper 级工作. 负面方向若退到 ∃R-hardness 相对可控. HIGH risk 评级诚实, 但 effort 估计偏乐观。 |
| Validation (pilot 设计) | 6/10 | pilot (d=2..10 Poisson, log(steps) vs d 斜率) 思路对路 — 用经验指数增长率作为 "正面 poly 不太可能" 的早期信号, 但当前 **Signal: SKIPPED / Result: pending**, 没真跑. 对纯理论 idea, 这个 pilot 价值在于 *证伪正面分支* (若 log-steps 明显线性增长就该转负面), 建议真做一次廉价扫描以决定主攻方向。 |
| Venue readiness | 7/10 | 任一方向成立都是 seminal (顶会理论 track / TMLR / 应用数学顶刊). 但 venue 价值高度依赖能否真正 *交付定理*, 而非仅指出缺口; 当前 proof-bridge 缺陷使 "能否做出来" 存疑。 |

## Alternative Framing

**把 idea 从 "二选一 dichotomy" 重构为 "三层 complexity-landscape of FEX search", 并显式拆成可证的子引理**:

1. **Search-space lower bound (最稳, 先拿)**: 沿 Vertical-SR (arXiv:2312.11955) 的 hypothesis-space 计数思路, 给 FEX 在树深 L、算子字典大小 o 下 *必须区分的等价类数* 的下界 — 这是纯组合计数, 不依赖 RL 动力学, 风险最低, 可作为 paper 的 backbone.
2. **正面分支 (改承重工具)**: 不提 Uniform-LGI; 改为 "在 Barron-like 类上, 若 controller 的 expected-reward J(Φ) 满足 gradient-domination (待证 Lemma A, 仿 Mei 2020 softmax-PG) 且该类的 ε-覆盖数 poly(d) (待证 Lemma B), 则 FEX 搜索 poly(d,1/ε) 收敛". 把两条 lemma 作为论文真正的技术贡献明示。
3. **负面分支 (换 hardness 等级)**: 放弃 QBF/PSPACE, 改证 **∃R-hardness** (保留实值内层 BFGS verifier) 或 Virgolin 式 NP-hardness (并显式说明这是 *退化* FEX 且区分 arXiv:2207.01018 / 2404.13820 / 2511.11712).

这个 framing 让 idea 即使正面定理做不出, 也有 (1) 的下界 + (3) 的 hardness 兜底, 把 "all-or-nothing 高风险赌注" 降为 "分层递进、必有产出"。

## Results-to-Claims Mapping

| Outcome | Supportable claim |
|---------|------------------|
| 正面成立 (poly 收敛定理) | "在 Barron-like / 有限 Fourier 类上, FEX 的 policy-gradient controller 以 poly(d,1/ε,k) 步收敛到 ε-最优" — **必须** 同时给出 (a) FEX-quantile 目标的 gradient-domination 引理 (常数对 d 的依赖明示), (b) 该类的 covering/effective-dimension 引理; 不能只引 LGI. 适用范围严格限 idealized controller (exact gradient / 已知 reward), 不能外推到 BFGS 噪声 reward 的实际实现, 除非额外处理。 |
| 负面成立 (hardness) | 若退到 **∃R-hardness**: "判定是否存在使 PDE-residual ≤ ε 的算子树+参数 是 ∃R-hard" — 干净且保留实值内层, 是最强可防御负面结果. 若退到 NP-hardness (Virgolin 式整数/affine 限制): 只能声称 "*退化的* FEX 搜索 NP-hard", 必须显式承认未覆盖实值内层, 且 novelty 因 arXiv:2207.01018 / 2404.13820 而仅 MEDIUM. QBF/PSPACE 除非给出交替 gadget, 否则 **不可声称**。 |
| 两个分支都卡住 (最现实的高风险结局) | 退守 Alternative Framing 的 (1): "FEX 搜索空间的等价类计数下界为 X", 纯组合, 仍是文献空白可发表的小一档结果。pilot (log-steps vs d) 若呈线性增长, 本身是 "正面 poly 不成立" 的经验证据, 可写成 diagnostic。 |
| 并发被 scoop (OpenXOR 2511.11712 路线扩到 SR) | 若他人先把 3-SAT→neural-search hardness 落到 expression-tree, 负面分支 novelty 归零; 此时 idea 价值转向正面分支与 FEX-specific 的 ∃R / 实值内层处理。**下一版务必读 2511.11712 全文确认其 scope。** |

## Overall

- Score: 6/10
- Comments: **缺口真实且重要 (Yang 脉络最大理论空白, 经双路独立核验开放), problem fidelity 极高, 这是一个值得做的方向**。但 v1 的两个分支承重工具都有硬伤: 正面方向误把 supervised-generalization 论文 (2202.10670 Uniform-LGI) 当 RL 收敛工具, 是 citation mismatch / 近 category error — 真正的工具是 softmax-PG gradient-domination canon (Mei 2020 / Agarwal 2021), 且仍需为 FEX 非标准 quantile 目标 + Barron 类 covering 各补一条新引理; 负面方向 QBF/PSPACE 与 FEX 的 *存在型* 搜索结构不匹配, 应改为 ∃R-hardness 或 Virgolin 式 NP-hardness 并区分两篇已有 SR-hardness + 并发 OpenXOR (2511.11712)。**最有价值的下一步**: 按 Alternative Framing 把 idea 拆成 (1) 搜索空间计数下界 (低风险 backbone) + (2) 正面 PG-收敛 (两条 lemma 明示) + (3) ∃R-hardness, 并真跑一次 pilot (d=2..10 log-steps) 决定主攻方向。effort 估计对正面分支偏乐观 (每条 lemma 可能是独立 paper 级)。HIGH risk 评级诚实。审稿信心: 本轮 novelty/feasibility 由两个独立 general-purpose research agent 核验 (含对 2202.10670 / 2207.01018 全文核读), 非跨模型, 故对 "并发 scoop" 与 "∃R-hardness 可行性" 两点仍建议下一版补全文核实。

</review>

<deep-lit-update date="2026-06-16">

## 新发现的相关文献 (deep-lit-tick 0616-fex, 2026-06-16)

### 搜索效率相关的竞品分析

1. **EGG-SR (2511.05849)**: Nan Jiang, Yexiang Xue (Purdue) 等, 2025.11, ICLR 2026. E-graph 等价剪枝可收紧 MCTS regret bound 并降低 DRL 梯度估计方差 (Rao-Blackwellization). 核心数学: DRL 代理的等效类 reward = E[G_t | equivalence class] 替代 per-node G_t, 减少方差. 对本 idea 的启示: (a) EGG-SR 的 regret bound 收紧技术为 FEX 搜索收敛分析提供了新的参照系; (b) 等价类剪枝本身可能提供 FEX 搜索空间 size 的 tighter upper bound, 对本 idea "搜索空间计数下界" 分支有工具价值. 代码开源: github.com/jiangnanhugo/egg-sr.

2. **GENSR (2602.20557)**: CVAE 潜空间 + CMA-ES 替代离散树搜索. 从 Bayesian 视角重新定义 SR 为 maximize p(Equ|Num). 潜空间连续性提供自然的方向信号. 对本 idea 的启示: GENSR 的结构重参数化思路为 "FEX 搜索能否通过连续松弛规避组合困难" 提供了可对照的范式——若连续化可行, 则搜索复杂度的理论下限可能不适用.

3. **SAGE-Fit (2605.23272)**: 系统诊断 "Good Structure, Bad Score" —— FEX 的 BFGS 内层优化可能因非凸性产生错误评分. 这对本 idea 的"搜索迭代次数"度量有直接冲击: 若 failure 来源于参数优化而非结构搜索, 则单纯分析 RL controller 的收敛率可能偏离实际瓶颈.

### 对 idea 的影响

- **Gap 仍然真实**: 所有竞品 (EGG-SR, GENSR, SymPlex) 均未证明 RL controller for expression tree search 的收敛率或采样复杂度. 本 idea 的理论 gap 仍然开放.
- **新参照系**: EGG-SR 的 Rao-Blackwellized gradient estimator 和 regret bound 为正面收敛分析提供了新的比对技术, reviewer 可能期待与此对照.
- **竞争风险**: 若 GENSR 证明连续潜空间可完全取代离散搜索, 则本 idea 的 "离散树搜索复杂度下限" 应用场景可能被压缩.

</deep-lit-update>
