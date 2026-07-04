---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-dim-lift-skeleton/
---

- One-sentence summary: 在低维度 (d=2-3) 学习符号骨架后通过排列不变宏 (sum, product, norm, pairwise interaction) 提升到高维度, 为 FEX 的高维 scalability 提供可操作的搜索策略.
- Hypothesis: 许多高维 PDE 的解具有对称结构 (如 u = f(sum(x_i^2)) 或 u = g(sum(x_i))); 在低维学到的骨架 (如 f(x_1^2 + x_2^2)) 可以通过 sum/product macro 自动推广到任意维度, 从而避免高维从零搜索.
- Expected outcome: 成功: 在 separable/radial/symmetric PDE 家族上, 低维骨架提升到 d=10-100 的精度与从零搜索 FEX 相当, 但搜索时间仅为低维时间 + 参数优化时间 (远低于高维搜索); 失败: 非对称 PDE 的低维骨架无法提升, 说明 FEX 的高维成功依赖于解的对称性.
- Contribution type: method+empirical-finding
- Risk: MEDIUM
- Estimated effort:
  - Compute: 40 GPU-hours on A100
  - Data: available (FEX 标准 benchmark, 设计对称/非对称 PDE)
  - Implementation: 2-3 weeks
- Novelty quick-check: FEX 原始论文在不同维度上分别搜索, 没有跨维度迁移. Parameter grouping (FEX-PG, 2410.00835) 减少参数数量但不做骨架迁移. 无人在 RL-based SR 中做过显式的 dimension lifting.
- Strongest objection: 仅对具有特定对称性的 PDE 有效, 适用范围可能过窄.
- Why we should do this: FEX 的理论证明了无 CoD, 但实际搜索仍然在高维变慢; dimension lifting 提供了一条将理论保证转化为实际搜索策略的路径. 无论成败都揭示 FEX 高维成功的机制.
- Pilot:
  - Setup: d=2 FEX 搜索骨架 (depth-1, 40 epochs) -> 提取骨架 sum(x_i^2) -> 提升到 d=10, 仅 BFGS 优化 alpha, beta
  - Metric: 若提升后精度 < 从零搜索的 2 倍 且时间 < 20%, 信号为正
  - Result (workspace/fex-dim-lift-skeleton/, 见 NOTES.md): d=2 FEX 搜索 (depth1 树, 40 epochs) 正确识别骨架 0.5*sum(x_i^2), rel-L2 1.5e-6, 264s. 将骨架 alpha*sum(x_i^2)+beta 提升到 d=10/20/50/100, 仅 L-BFGS 优化两个参数, 均收敛到 alpha=0.5,beta=0 (解析解, rel-L2 9.6e-9~9e-8), 每维约 1s, 与维度无关. 从零 FEX 搜索 d=10 (同 depth1 树/预算) 也干净成功 (恢复 0.5*sum(x_i^2), 全部系数=0.5000, rel-L2 7.3e-7) 但需 261s. 成功判据大幅满足: lift 误差好 76x, 时间仅 0.4%.
  - Signal: POSITIVE (caveat: lift 消耗的 d=2 骨架本身花 268s, 优势是跨维度族摊销 [d∈{10,20,50,100} sweep: lift 管线~272s vs 从零~1040s], 非单维度更快; 这是最容易的可分离 PDE; sum macro 手工给定. 自动推断置换不变 macro + 非对称 PDE 上验证 lift 应失败, 才是真正贡献, 本 pilot 未测.)

<review date="2026-06-16">

## Novelty

- Score: 7/10
- Closest prior work: HD-TLGP (Cao et al., AAAI 2024) — genetic programming with 1D→high-D structural transfer for PDE solving. But HD-TLGP: (a) requires known 1D analytical solution, (b) hard-codes multiplication/addition macros per PDE type, (c) only tests to d=3, (d) still runs full 100-generation GP search in high-d. The current idea differs fundamentally: learns skeleton from data via FEX RL (no known solution needed), infers permutation-invariant macros automatically, scales to d=100, and does pure parameter optimization at high-d.
- Key differentiator: The "learn low-d skeleton → infer invariant macro → lift with parameter-only optimization" pipeline is novel. HD-TLGP proves structural transfer across dimensions works, but the core mechanism (automatic macro inference + RL skeleton discovery + parameter-only lifting) has no direct prior. The FEX committor paper (Song et al., SIAM J. Sci. Comput. 2025) goes high→low (discovers intrinsic low-dim structure), not low→high.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | Automatic macro inference from low-d skeleton is unspecified — this is the core method contribution and is untested. Identifiability: d=2 skeleton may not uniquely determine the correct high-d macro (e.g., sum vs product forms can look identical in low-d). Weight-tying scheme for macro parameters is also unspecified. |
| Missing evidence signals | 5/10 | (1) Automatic macro inference algorithm on ≥3 nontrivial symmetric PDE families; (2) asymmetric PDE negative control confirming lift failure; (3) comparison to FEX-PG and HD-TLGP baselines at matched compute; (4) multi-seed runs showing search stability; (5) depth-2/3 trees to test whether deeper skeletons lift correctly; (6) compute budget sweep to quantify amortization break-even. |
| Narrative | 7/10 | Hypothesis is clear and the pilot is well-documented with honest caveats (amortization, easiest case, hand-supplied macro). The explicit acknowledgment that "无论成败都揭示 FEX 高维成功的机制" is excellent falsification-first thinking. The connection to the landscape gap (FEX search complexity theory) could be made stronger. |
| Venue contribution | 6/10 | For inferred NeurIPS/ICML standards: the pilot proves concept viability, but the full contribution (automatic macro inference + multi-family validation) is needed. Current state is a promising diagnostic rather than a complete method. The mechanism-finding framing ("when does FEX exploit symmetry?") is stronger than the scalability-engineering framing. |
| Testability | 8/10 | Clean falsifiable prediction: asymmetric PDEs should fail to lift. Pilot metric (lift error < 2x from-scratch, time < 20%) is well-defined and was exceeded. Cheapest next falsifier: automatic lift fails on 3+ nontrivial symmetric families, or succeeds on asymmetric ones (both informative). |
| Outcome realism | 7/10 | POSITIVE outcome is realistic for separable/radial PDEs (pilot confirmed). The NULL outcome (manual works but auto fails) is also realistic and publishable as a diagnostic. The NEGATIVE outcome (skeletons don't transfer at all) is unlikely given pilot evidence but possible for complex operator trees. |
| Contribution type compliance | n.a. | Topic `0616-fex.md` does not declare `preferred-contribution-types` in frontmatter; check skipped. No hard cap triggered from this dimension. |
| Overall Quality | 6.5/10 | Well-formed idea with strong pilot signal and honest self-assessment. Main open risk is automatic macro inference, which needs a concrete algorithm before this can be considered method-complete. |

## Contribution Drift

N/A (v1 — no previous version to compare)

## Alternative Framing

Frame as: "Automatic discovery of permutation-invariant macro skeletons for cross-dimensional FEX transfer, with success/failure used to identify when FEX's apparent no-curse-of-dimensionality behavior is symmetry-driven." This reframes the contribution as a mechanism finding about FEX scalability (defensible, novel, and interesting regardless of outcome) rather than an engineering speedup (less defensible given HD-TLGP prior art). The current framing leans too heavily on "scalability" when the deeper contribution is the diagnostic question.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | For PDE families whose solutions lie in a permutation-invariant macro algebra (separable, radial, symmetric), low-d FEX skeletons with automatically inferred macros can amortize search across dimensions and match from-scratch FEX accuracy at a fraction of the search cost. |
| NULL | Manual macro lifting works on trivial symmetric examples, but automatic macro inference fails to generalize reliably beyond the easiest cases, revealing that FEX's high-d success depends on conditions stricter than simple symmetry. |
| NEGATIVE | Low-dimensional FEX skeletons do not transfer reliably even with manual macros beyond the simplest separable case; FEX's apparent high-d scalability is not explained by recoverable invariant skeleton reuse. |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: **Likelihood=Medium**: the pilot already demonstrates the core mechanism works for the easiest case, and the path to a top-venue paper is clear (automatic macro inference + 3-5 PDE families + negative controls + baselines). However, success depends on the automatic macro inference algorithm working across diverse PDE families, which is non-trivial. **Impact=High**: if it works, this would (a) provide the first practical search strategy for FEX at very high dimensions, (b) reveal symmetry as the mechanism governing FEX scalability, connecting to the landscape's #1 gap about why FEX scales, and (c) open a new research direction in curriculum/skeleton-based SR search. Claude-codex disagreement: codex assessed Likelihood=Low (concerned about automatic macro inference and novelty vs HD-TLGP). After deep reading HD-TLGP, I assess the novelty delta as substantial (auto vs hard-coded macro, RL vs GP, d=100 vs d=3, no known solution needed), justifying Likelihood=Medium rather than Low.

## Overall

- Priority: High
- Score: 7
- Comments: A well-formed idea with an honest pilot and clear falsifiability. The biggest contribution risk is the automatic macro inference step, which is the core method innovation and is currently unspecified. The mechanism-finding framing (alternative framing above) is recommended over the scalability-engineering framing. HD-TLGP (AAAI 2024) must be cited as prior work establishing the feasibility of cross-dimension structural transfer, with explicit differentiation on the key deltas (auto macro inference, RL vs GP, d=100 vs d=3). Claude and codex disagree on Likelihood by 1 level (Medium vs Low); the discrepancy stems from codex not having read HD-TLGP fully. The deep-lit-reader confirmed HD-TLGP's substantial limitations that differentiate it from this idea.

</review>

<deep-lit date="2026-06-16" scope="idea" rounds="2" papers_read="9">

## Novelty Update (post deep-lit)

9 篇新论文精读后, novelty assessment 不变甚至加强。最相关的先行工作是 **Dimension Reduction for SR (2506.19537)** 和 **Decomposable Neuro SR (2511.04124)**, 但它们与我们的核心创新（低维 RL 骨架学习 → 自动宏推断 → 纯参数优化提升）有本质差异。详见下文。

## 本次新读论文与 idea 的关系

### 直接相关（影响 novelty / 需要引用）

**2506.19537 — Dimension Reduction for Symbolic Regression** (Kahlmeyer, Fischer, Giesen, 2025.06)
- **做什么**: 用功能依赖度量 (Chatterjee coefficient / Codec / KMAc) 检验变量替换有效性, 通过 beam search 迭代降维, 再传给任意 SR 算法。
- **与 idea 的关系**: 这是与我们 idea 最接近的 prior work——同样做"发现变量组合来降维"。但关键差异: (a) DR-SR 做纯粹的统计搜索 (穷举 DAG + beam search), 没有跨维度迁移的概念; (b) DR-SR 的替换是单向降维 (dimension reduction), 不是我们的 low→high lift; (c) DR-SR 不涉及 RL 或 skeleton 学习。我们的 idea 仍然是首个提出 "低维 RL 骨架学习 → 推断置换不变宏 → 高维纯参数优化" 的工作。
- **该引**: 是。作为 SR 中自动发现变量关系的状态-of-the-art baseline。
- **可用组件**: Codec/KMAc 功能依赖度量可作为我们自动宏推断的验证工具。

**2511.04124 — Decomposable Neuro Symbolic Regression** (Morales, Sheppard, 2025.11)
- **做什么**: Multi-Set Transformer 生成多个 univariate symbolic skeletons, GA 筛选, GP cascade 逐步合并为 multivariate expression。
- **与 idea 的关系**: 与我们的 "先学习单维度骨架再组合" 思路高度相似, 但 key differences: (a) 它是 univariate skeleton → multivariate merge (单一维度的变量分解), 我们是 low-d skeleton → high-d lift (跨维度); (b) 它用于解释黑盒模型, 我们用于 PDE 求解; (c) 它的 merge 通过 GP 搜索, 我们通过宏推断+参数优化。
- **该引**: 是。作为 skeleton-based SR 的 prior work, 体现"分解-组合"范式的可行性。
- **撞车风险**: 低, 因为维度提升与变量分解是正交问题。

**2505.12083 — Discovering Symbolic Differential Equations with Symmetry Invariants** (Yang, Bhat, Hu, Yu et al., 2025.05)
- **做什么**: 用已知对称群的微分不变量作为 SR 的原子实体, 确保发现的方程满足指定对称性。
- **与 idea 的关系**: 如果我们的 automatic macro inference 需要从骨架反推对称性, 本文的微分不变量框架是理论基础。但关键差异: 本文需要先知道对称群, 我们的目标是自动从数据中发现对称结构。
- **该引**: 是。如果我们的 macro inference 步骤用到置换不变性检测, 需引用本文的不变量框架。
- **跟进论文**: **DI-SINDy (2505.18798)** (Hu, Li, Lin, 2025.05) — 同样基于微分不变量但更侧重计算效率; **LieNLSD (2510.01855)** — 同组, 发现非线性 Lie 对称性。

### 方法参考（不直接竞争但提供可用组件）

**2502.03367 — SyMANTIC** (Muthyala, Sorourifar et al., 2025, 15 citations)
- 互信息特征选择 + ℓ₀ 稀疏回归做高维 SR, 识别低维 descriptor。
- 可作 baseline: 如果 SyMANTIC 的 MI 筛选能从高维数据中自动识别出哪些变量组合是可降维的, 那它就是自动宏推断的一个统计 baseline。
- 撞车风险: 低 (纯数值 SR vs RL-based skeleton learning)。

**2503.19043 — QDSR: Quality-Diversity + Dimensional Analysis** (Bruneton, 2025, 3 citations)
- MAP-Elites + 量纲分析, 91.6% 精确恢复率 (Feynman-AI)。
- 启示: Quality-Diversity 可用于 macro 空间的多样性探索——不仅推断一个宏, 而是维持一组候选宏, 按维度适用范围分桶。
- 撞车风险: 低 (GP 搜索 vs RL 搜索, 维度分析 vs 对称性推断)。

**2506.19550 — Discovering Symmetries of ODEs by Symbolic Regression** (Kahlmeyer et al., 2025.06)
- 用 SR 搜索 ODE 的 Lie 点对称性生成元。
- 启示: 如果有对称性但未知, 可将本文的对称性发现方法作为宏推断的前置步骤。
- 撞车风险: 低 (ODE 对称性发现 != PDE 解的维度骨架迁移)。

**2511.09416 — Transformer Semantic GP for d-dimensional SR** (Anthes, Sobania, Rothlauf, 2025.11)
- 预训练 transformer (3.8M params) 作为 GP 语义变异算子, 单一模型跨 d=2-5 泛化。
- 启示: Transformer 可以学习跨维度的语义相似性——这为我们的"宏推断"步骤提供了神经网络的替代实现路径。但 TSGP 只做到 d=5, 且需要大量预训练数据。
- 撞车风险: 低 (GP 变异算子 vs skeleton lifting)。

**2504.02630 — GODE: Grammar-based ODE Discovery** (Yu, Chatzi, Kissas, ETH, 2025, 7 citations)
- CFG 语法 → Grammar VAE 嵌入 → CMA-ES 在潜空间搜索 ODE 骨架 → 系数精化。
- 启示: grammar 约束搜索空间是经典范式, 我们的 macro 本质上也是一种 grammar (sum, product, norm 等置换不变算子组成的子语言)。Grammar VAE 的潜空间搜索可参考。
- 撞车风险: 低 (ODE vs PDE, grammar 定义 vs 宏推断)。
- 跟进: **Latent Grammar Flow (2604.16232)** — 同组, FSQ 嵌入 + discrete flow matching, 已收录 landscape。

### 补充发现 (B7 / R2 B1 发现, 未被选读)

- **2510.01855 (LieNLSD)** — Hu, Li, Lin, 2025.10: 首次从数据显式发现非线性 Lie 对称性, 用 SVD 求解系数矩阵。可作为对称性发现工具。
- **2511.09779** — Kreider et al., 2025.11: model-free 数据驱动对称性发现, GMLS 流形学习。
- **2605.11524 (EqOD)** — N'guessan, Kim, 2026: 对称性稳定性选择, DI-SINDy 的跟进工作。
- **2605.21160** — Zhong et al., 2026.05: 反向生成数据+引导 RL 学习守恒量 (first integrals)。

## 更新后的撞车风险矩阵

| 核心 Claim | 撞车状态 | 最接近工作 | Δ 差异 |
|-----------|---------|-----------|--------|
| 低维 RL 骨架学习 | ✅ 唯一 | — | FEX 独有此能力 |
| 自动置换不变宏推断 | 🟢 开放 | DR-SR (2506.19537) 做统计降维, 非宏推断 | DR-SR 是降维→降维, 我们是降维→升维 |
| 跨维度骨架提升 | 🟢 开放 | HD-TLGP (已收录) 做硬编码 1D→3D; Decomposable SR 做单维变量分解 | 我们是自动宏+RL 骨架+任意维度 |
| 对称性驱动的搜索压缩 | ⚠️ 有方法铺垫 | SI-SR (2505.12083), DI-SINDy (2505.18798) | 它们需要已知对称群, 我们目标是自动发现 |
| 参数唯一性/可辨识性 | 🟢 开放 | — | 骨架提升时参数优化的 identifiability 未被研究 |

## 建议的论文叙事位置

1. **Intro / Related Work**: 引用 HD-TLGP (established cross-dim transfer), DR-SR (statistical variable combination discovery), Decomposable Neuro SR (skeleton decomposition paradigm)
2. **Method**: 宏推断步骤引用 SI-SR/DI-SINDy 的不变量框架作为理论背景, 但强调我们的贡献是从数据自动推断而非预设对称群
3. **Discussion**: 提及 QDSR 的 diversity 视角、TSGP 的跨维度语义学习、GODE 的 grammar 约束作为未来工作方向

## 总体结论

本次 deep-lit 确认: **fex-dim-lift-skeleton 的核心 novelty（低维 RL 骨架→自动宏推断→跨维度提升）在 literature 中仍然是空白**。最接近的 DR-SR 做的是统计降维而非跨维度升维, Decomposable SR 做的是单维变量分解而非维度迁移。对称性不变量文献提供了宏推断的理论工具, 但需要"从数据自动发现对称性"这个环节, 这正是我们的贡献所在。Idea 的 novelty 从 7/10 维持或微升至 7.5/10。

</deep-lit>
