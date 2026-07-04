---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-search-complexity/
---

- One-sentence summary: 建立 FEX discoverability factorization theorem, 把 "表达式存在" 到 "controller 找得到" 之间的难度拆成 quotient-cover、PG geometry 和实值系数可行性三项.
- Problem anchor: "FEX 是我的课题组(Haizhao Yang 组)发明的一种方法, 使用 RL 进行 Symbolic Regression, 我(Youran Sun)作为 Yang 的博后, 应该继续在这个方向上探索"; 本 idea 只研究 FEX/PDE 表达式搜索的复杂度边界, 不漂移成通用 SR benchmark 或新工程系统.
- Hypothesis: FEX 的搜索困难不能由 raw expression-tree count 单独解释. 对某些 PDE 解族, 保守等价类 quotient 已经显著压缩结构空间; 但 blind sampler 仍有 `Omega(Q/K)` 命中下界, PG controller 还需要 polynomial `kappa_PG`, 选中结构后的实值 residual feasibility 仍可能继承 ∃R 困难.
- Expected outcome: 成功版本应能证明几条边界清楚的结果. (i) Quotient-count theorem: 对显式 rewrite set `R_ac+c` (add/mul 局部交换、zero/one leaf 常数族合并、zero/one root 常数输出合并), rooted binary-composition grammar 满足 `q_0=8`, `q_{l+1}=1+7*(q_l^2+q_l*(q_l+1))`; Poisson `depth2_sub` 正是 `l=1`, raw `2187` -> quotient `953`, 且 finite canonicalizer checks 全通过. (ii) Sampler lower bound: 对 `Q` 个 quotient classes 和 `K` 个 epsilon-good classes, blind class-level sampler with replacement 的期望 hit time 是 `Q/K`, no-repeat sampler 在 uniform hidden good set 下是 `(Q+1)/(K+1)`, 所以不使用 reward geometry 的 distribution-free 说法只能给 `Omega(Q/K)`. (iii) Conditional PG theorem: 若 `N_FEX(F,L,eps)=poly(d,1/eps)` 且 softmax/f-softargmax expected reward 满足 `J*-J(theta) <= kappa_PG ||grad J(theta)||^2`、方差有界, 则 structure gap 在 `poly(N_FEX,kappa_PG,1/eps)` 更新内下降. (iv) Feasibility-hardness route: bounded-ETR/ETR-INV 的 `x+y=z`、`xy=z`、box constraints 映射到 FEX residual squares; 未完成 reduction 前只称 route, 不称 ∃R-hardness. 最便宜的证伪信号是: e-graph/semantic quotient 与 FEX residual grammar 不相容, `kappa_PG` 在真实 trace 中随好类稀疏度爆炸, 或 bounded-ETR gadget 不能保持有界变量和小 FEX 子树.
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
  - Setup: Keep v3/v4 Poisson and PG-trace diagnostics; v5 adds a CUDA-run quotient certificate for `R_ac+c`, finite `depth2_sub` canonicalizer checks, rooted grammar recurrence through level 4, and explicit blind-sampler lower-bound examples.
  - Metric: Require exact count agreement with the old probe (`2187 -> 953`), idempotent representatives, zero commutative swap failures, zero subtraction false equalities under nonconstant roots, and CUDA sanity on the local GPU.
  - Result: v5 certificate passes: `depth1` raw `243` -> quotient `136`; `depth2_sub` raw `2187` -> quotient `953`; idempotence true; commutative swap failures `0`; subtraction false equalities under nonconstant roots `0`; representative checksum `b1e3bf9e6ce69d17ec57e6d12cc64daff450334a2d9cc1df6d172185f997ae80`. Rooted recurrence gives quotient counts `q_2=12721598`, `q_3=2265746868481643`, `q_4=71870524208481219124311271083788` for the declared subgrammar; this is not the full FEX depth-3 controller count. CUDA sanity ran on RTX 4060 Ti.
  - Signal: POSITIVE for Layer-1 theorem artifact; WEAK for the full factorization because PG and ETR layers remain conditional.

- Claims and Claims matrix: Positive claim: "For the declared conservative FEX subgrammar, quotient counts and blind-sampler lower bounds are exact, and FEX discoverability factorizes into quotient cover, PG geometry, and coefficient feasibility." Conditional claim: "If `kappa_PG` is polynomial, low `N_FEX` gives polynomial structure-search updates." Null claim: "Small quotient cover alone does not imply fast PG; v4 softmax-bandit proxy already shows `kappa_PG` depends on good-class mass." Negative retreat: if semantic quotient or ETR reduction fails, keep the syntactic quotient theorem plus conditional PG diagnostic and explicitly drop ∃R-hardness.
- Narrative: 论文从 "FEX can represent the solution" 和 "FEX can discover the solution" 的差距切入. Quotient theorem 是第一块严格结果; PG 和 real feasibility 分开写, 避免用小 grammar count 掩盖优化困难.
- Experiments: Keep experiments diagnostic: compare `R_ac+c` counts with EGG-SR/GSR-style e-graph quotienting; rerun short FEX traces with logged logits/grad norms on Poisson/Fourier/separable/nonseparable PDE families; report `kappa_PG` proxies by good-class mass; include one bounded-ETR gadget family before any hardness claim. Extra PDE families are evidence that theory objects are non-vacuous, not a benchmark contribution.
- Assets status: No external data is needed; v5 CUDA quotient certificate and prior PG/ETR diagnostics are complete, with handoff details in `workspace/fex-search-complexity/data/MANIFEST.md`.
- Review handling: I accept the main v4 criticism and add a Layer-1 proof artifact instead of another framing pass. I accept the warning that `Q_L` was too syntactic by naming the rewrite set and marking semantic/e-graph quotienting as the next step, not as solved. I accept the sampler-lower-bound criticism by specifying the blind class-level sampler and the hidden-good-set model. I accept that cover-to-PG remains unproved, so the PG theorem stays conditional; I push back only against the stronger demand that cover size should imply PG convergence, because v4 already showed the opposite. I accept that bounded-ETR is still a route, not a result, so v5 does not claim ∃R-hardness. I treat missing depth3/4 and PDE-family breadth as an in-scope diagnostic concern: v5 adds recurrence counts for the declared rooted grammar and leaves cross-family traces for proposal-stage diagnostics. Contribution drift note: v4 was `theory`; v5 remains `theory`, with no method, benchmark, diagnostic, application, or empirical-finding type added or removed.

<review date="2026-06-17">

## Novelty

- Score: 8/10
- Closest prior work: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026); Jiang et al., *EGG-SR* (2511.05849, ICLR 2026); Virgolin & Pissis, *Symbolic Regression is NP-hard* (TMLR 2022)
- Key differentiator: v5 的四个 core claim 经多源搜索确认均未被预占. (a) 对 FEX PDE residual rooted binary-composition grammar 在显式 rewrite set `R_ac+c` 下的 quotient-count theorem — EGG-SR/eggp/IsalSR/ESR 均做 equivalence reduction 但不针对 FEX 语法、不证明计数定理; (b) blind class-level sampler 的 `Ω(Q/K)` lower bound — 是 coupon-collector 类分析的 FEX 特化, 无 prior work 对 FEX quotient cover 做 sampler lower bound; (c) conditional PG convergence theorem under gradient domination for expression-tree action space — EGG-DRL 证明 variance reduction 非 convergence rate, 现有 PG 理论未 bridge 到 FEX controller 的 expression-tree grammar 结构; (d) bounded-ETR/ETR-INV 到 FEX residual feasibility 的 ∃R-hardness route — Virgolin (NP-hard) 和 Soubki (FPT/W-hierarchy) 均未涉及 ∃R, 且无任何工作将 ETR 映射到 fixed-structure FEX 系数可行性. 本轮扫描未发现 2026 年新预占性工作; IsalSR (2603.21836) 做 canonical form 但不计数、不针对 FEX grammar.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v5 从 v4 的 "骨架有但肌肉待填" 升级为 "Layer-1 有硬证明 artifact": 显式 `R_ac+c` rewrite set、CUDA quotient certificate、rooted grammar recurrence 到 level 4、finite canonicalizer checks 全通过. 相比 v4 的 6/10 提升 1 分. 剩余三大缺口: (a) syntactic `R_ac+c` quotient 与 semantic/e-graph quotient 的对齐/包含关系仍未建立 (EGG-SR 的 e-graph 覆盖了 `log(ab)↔log a+log b` 等语义等价, `R_ac+c` 仅是 commutative+constant 的保守压缩); (b) cover→PG convergence 的正向 bridge 仍为零, conditional PG theorem 完全依靠 assumed gradient domination; (c) bounded-ETR→FEX 仍为 route, 无 reduction. |
| Missing evidence signals | 8/10 | v5 新增的证据信号均有效且可核查: CUDA quotient certificate (idempotence/commutative swap/subtraction false equality checks 全通过, checksum `b1e3bf9e...`)、rooted recurrence counts 到 level 4、blind-sampler lower-bound examples. 相比 v4 的 7/10 提升 1 分. 仍缺: e-graph quotient 与 `R_ac+c` 的比较; 真实 FEX controller 上 `kappa_PG` 随 architecture/reward features 的变化; depth3/4 full controller counts (当前 recurrence 是 declared subgrammar, 非 FEX controller 的完整 action-space 计数). 但这些缺失主要是 diagnostic breadth, 不影响 proximal entrance gate —— v5 已经有可证伪的硬 artifact. |
| Narrative | 8/10 | "FEX approximation theory proves existence, but discoverability requires three separate conditions" 的 framing 比 v4 的 "search-complexity factorization" 更锐利, 更低过度承诺风险. 三因子从 v4 的乘积 framing 演进为 v5 的 "existence vs discoverability" gap framing, 使 conditional PG 和 feasibility route 从 "discount" 变成 "necessary analysis objects." Soubki & Cranmer 被正确引述并区分. 微小叙事风险: "discoverability factorizes" 这句话在尚未证明三因子独立或完备时过度暗示, 建议改为 "proposed factorization framework" 或 "discoverability diagnostic theorem" (与 codex 的 alternative framing 建议一致). |
| Venue contribution | 7/10 | Topic 无 `target-venue`, 推断为 ICML/NeurIPS/ICLR theory track. v5 从 v4 的 "well-characterized research agenda" 升级为 "partial theorem with Layer-1 proof artifact." 若至少一个因子变成 unconditional theorem (counting lemma 最可行) + semantic/e-graph quotient comparison + conditional PG diagnostics: 可成为合格的 top-venue theory paper. 当前 syntactic quotient theorem alone 会不会被认为过窄 (仅 conservative commutative+constant rewrite) 是主要风险. 若能将 `R_ac+c` 的 quotient 与 EGG-SR 的 e-graph rewrite rules 建立包含/间隙关系, venue contribution 会再升 1 分. |
| Testability | 8/10 | Expected outcome 中的三个 falsifier 均合理且便宜: e-graph/semantic quotient 与 `R_ac+c` 不相容; `kappa_PG` 在真实 trace 中随好类稀疏度爆炸; bounded-ETR gadget 不能保持 bounded variables/small FEX subtrees. 更尖锐的便宜 falsifier (codex 建议) 是: quotient compression 的贡献主要由 trivial constant-output class 驱动, 与实际 reward-good structures 几乎无关 —— 这可以直接用 v5 已有数据测试. |
| Outcome realism | 7/10 | Effort split 比 v4 更诚实: quotient-count theorem 已部分兑现 (weeks), conditional PG 3-12 months, full bounded-ETR reduction 明确标注 "may split into a theory note." v5 不再声称 ∃R-hardness, 比 v4 的 "prove ∃R-hardness" 承诺更现实. Realistic top-venue 版本: syntactic quotient theorem + semantic/e-graph comparison + negative PG diagnostic (cover alone doesn't imply fast PG) + bounded-ETR gadget family. 三个 layer 全部硬兑现仍需 1-3 年. |
| Contribution type compliance | n.a. | idea types = {theory}; topic 0616-fex.md 未声明 `preferred-contribution-types` 字段, 本检查跳过, 不计入 Overall Quality 平均. |
| Overall Quality | 7/10 | v5 是 v4 的实质性增量: 从 "theorem skeleton" 升级到 "Layer-1 proof artifact with CUDA certificate." 主要风险已从 "缺乏硬产物" 转移到 "syntactic quotient theorem 是否足够非平凡以支撑顶会理论贡献." 关键下一步: 将 `R_ac+c` 与 EGG-SR 的 e-graph rewrite rules 建立包含/间隙关系, 或证明 quotient compression 的主力不是 trivial constant-output collapse. |

## Contribution Drift

- v_{n-1} contribution types: {theory}
- v_n contribution types: {theory}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`; v5 在 Review handling 段显式声明 "v4 was `theory`; v5 remains `theory`"; ∃R 从 claimed theorem 降为 route 有明确理由, 属 claim correction 非 silent downgrade; 无越界 expansion)

## Prior Review Resolution (v4 concerns)

| v4 Concern | Status | Notes |
|------------|--------|-------|
| 需要至少一个 hard artifact (v4 Overall Comment) | **Partially resolved** | v5 产出 syntactic quotient theorem + CUDA certificate + rooted grammar recurrence. 但这只是 Layer-1 syntactic artifact; 顶会 reviewer 可能认为单独 syntactic counting 不够非平凡. |
| canonical quotient 需从 syntactic 升级到 semantic/e-graph (v4 Logical gaps a) | **Partially resolved in claim boundary, not technically resolved** | v5 在 body 中显式标记 "marking semantic/e-graph quotienting as the next step, not as solved," 诚实但未推进实际 bridge. |
| cover size → PG landscape geometry 的正向 bridge 缺失 (v4 Logical gaps b) | **Not resolved** | v5 保持 conditional PG theorem. Pushback "cover size alone should not imply PG convergence" 有据 (v4 bandit proxy 证明), 但仍未给正向 FEX-specific bridge. |
| bounded-ETR→FEX 仍为 route 非 reduction (v4 Logical gaps c) | **Partially resolved by downgrading claim** | v5 不再声称 ∃R-hardness, 降为 "feasibility-hardness route." 技术上未推进, 但 claims discipline 改善. |
| depth3/depth4 quotient counts (v4 Missing evidence) | **Partially resolved** | v5 有 declared rooted grammar recurrence 到 level 4 (`q_2=12721598`, `q_3≈2.27e15`, `q_4≈7.19e31`), 但不是 full FEX depth-3/4 controller count. |
| Fourier/separable/nonseparable PDE families (v4 Missing evidence) | **Deferred with acceptable pushback** | v5 声明 leaves cross-family traces for proposal-stage diagnostics. Pushback 有据 (theory type, 跨 PDE 对比是 diagnostic 不是 benchmark). 但缺失仍削弱 "theory objects are non-vacuous" 的说服力. |
| EGG-SR 等价类未整合进 quotient formalization (v4) | **Not resolved** | v5 提到 EGG-SR/GSR 但未建立 `R_ac+c` 与 e-graph rewrite rules 的形式关系. |
| Sampler lower bound 需 sharp specification (v4) | **Resolved** | v5 显式指定 blind class-level sampler, with-replacement vs no-repeat, hidden-good-set model. |
| Real-controller `pg_trace` logging gap (v4) | **Resolved** | v4 已实现, v5 继承. |

## Alternative Framing

当前 "existence vs discoverability" framing 已比 v4 的 "search-complexity factorization" 更锐利. Codex 建议进一步收窄为 **FEX discoverability diagnostic theorem**: 核心 claim 从 "discoverability factorizes into three components" 改为 "approximation theory only controls existence; search success additionally requires quotient relevance, reward geometry, and coefficient feasibility — we prove a Layer-1 quotient theorem and show the other two layers are separable and individually necessary." 这降低了过度承诺风险, 且与当前已兑现的证据 (syntactic quotient count + negative PG diagnostic) 更好对齐. 建议 v6 在 body 中用 "diagnostic theorem" 替换 "factorization theorem."

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | "For the declared conservative FEX subgrammar with rewrite set R_ac+c, exact quotient counts are 136 (depth1), 953 (depth2_sub), 12721598 (depth2), with idempotent canonicalization and zero commutativity-induced false equalities. Under a hidden-good-set model, a blind class-level sampler has expected hit time Ω(Q/K)." 不可声称 semantic equivalence 或 full FEX grammar coverage. |
| NULL | "Small quotient cover alone does not imply fast PG search — v4 bandit proxy shows κ_PG depends on good-class mass independent of cover size. The factorization identifies which factor fails, ruling out the false inference that small cover implies fast PG." 不可声称已证明 unconditional polynomial convergence. |
| NEGATIVE | "If semantic/e-graph quotient is incompatible with R_ac+c, or κ_PG proxy explodes with good-class sparsity, or bounded-ETR gadgets cannot be embedded in FEX grammar, we retreat to syntactic quotient theorem + negative PG diagnostic + toy arithmetic embedding (NOT ∃R-hardness or unconditional tractability)." |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  - **Likelihood: Medium** — v5 有一条明确的成稿路径: syntactic quotient theorem (已部分兑现) + semantic/e-graph comparison + negative PG diagnostic + bounded-ETR gadget family. Layer-1 artifact 降低了从零证明的风险; 主要不确定性在: (a) syntactic quotient 是否足够非平凡让顶会 reviewer 接受, (b) cover-to-PG 正向 bridge 能否在不需要完整 landscape analysis 的前提下给出 FEX-specific condition, (c) bounded-ETR reduction 的 proof gap 仍然非平凡. 整体有清晰退路: 即使 PG bridge 和 ETR reduction 均失败, syntactic quotient + diagnostic 仍然构成可发表的 "FEX discoverability diagnostic" 论文.
  - **Impact: High** — 若 factorization/diagnostic theorem 成立, 将 fundamentally change 社区对 expression-tree RL 搜索的理解: "维度灾难不在离散搜索自身, 而在连续优化 feasibility 和 controller geometry." FEX 从 approximation theory (存在性) 到 searchable method (可发现性) 的理论缺口被填补, 直接服务于 Yang 组的 FEX 研究线. 但不到 Exceptional, 因为目前没有迹象会推翻 SR/RL 搜索领域的强共识或打开全新研究路线.
  - Claude 与 codex 对 Likelihood (均 Medium) 和 Impact (均 High) 无分歧.

## Overall

- Priority: High
- Score: 7
- Comments: v5 是连续四轮迭代中首次产出硬证明 artifact 的版本, 从 "well-characterized research agenda" 实质性升级为 "partial theorem." CUDA quotient certificate 质量可接受 (`2187→953`, idempotence/commutativity checks 全通过, rooted recurrence 到 level 4). 与 codex 独立评估完全一致: Medium Likelihood × High Impact = High (7). 主要剩余风险: syntactic quotient alone 可能被顶会 reviewer 视为过窄; 建议 v6 在 (a) 建立 `R_ac+c` 与 EGG-SR e-graph rewrite rules 的包含/间隙关系, 或 (b) 证明 quotient compression 的贡献主力非 trivial constant-output collapse 这两条中择一突破. 无 contribution drift, 无 hard cap.

</review>

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

- **Exhaustive Symbolic Integration (2605.04978)** [wiki](wiki/2605.04978.md): Harry Desmond (Portsmouth), 2026.05. 穷举给定 operator basis 和 complexity 上限内的所有表达式，计算 integrability fraction ρ(k)。核心发现：log 显著提高 bounded grammar 的积分闭合性 (~3x)；operator basis 对 integrability landscape 影响极大。**对 idea claim (i) 的影响：** (a) ESI 的穷举方法提供了 FEX quotient-count theorem 的经验对照——可以在相同 operator basis 下，比较 ESI 的 raw expression count 与 R_ac+c quotient count，验证 quotient compression ratio；(b) integrability fraction 概念可类比 FEX 的 "discoverability fraction"——给定 operator basis 和 depth，多大比例的 PDE 解可被 S_k 表示且 controller 可发现？(c) numerical fingerprinting 技术可用于 FEX 的 semantic equivalence testing，补充 R_ac+c 的 syntactic quotient。但 ESI 面向 symbolic integration 而非 PDE solving，方法论距离较远。

### 本次调研对 idea claims 的累计影响 (三轮合计 19 篇)

| Claim | 三轮共发现 | 累计影响 |
|-------|-----------|---------|
| (i) Quotient-count theorem | ESI (2605.04978) 提供穷举 counting 的经验对照和 numerical fingerprinting 工具 | marginally strengthened — 可用于 empirical validation，但不改变理论贡献 |
| (ii) Sampler lower bound | 无直接新竞品 | unchanged |
| (iii) Conditional PG theorem (κ_PG) | **+4 篇 AC convergence 理论：O(ε⁻³) AC (2410.08868)、O(ε⁻²) optimal AC (2605.13639)、entropy-reg AC strong variance reduction (2605.24357)、operator-theoretic PG (2603.17875)** | further strengthened — 现在有 tabular AC 从 O(ε⁻⁴) → O(ε⁻³) → O(ε⁻²) 的完整 sample complexity 进化链，minimal assumptions 版本，以及 strong variance reduction 在最优附近的正面结果；但全部针对 tabular/finite MDP，FEX 离散树 action space 的特殊性仍是 idea 的独特贡献白地 |
| (iv) Feasibility-hardness route (∃R) | **+4 篇 ∃R 理论：四次凸性 co-∃R-complete (2510.19704)、tensor degeneracy ∃R-complete 纯代数归约 (2604.17061)、continuous clustering ∃R-complete (2604.26972)、ETR+integer powers decidability (2502.02220)** | significantly strengthened — ∃R 工具箱大幅扩充：(a) landscape property verification (凸性/稳定性) 本身是 co-∃R-hard 的新发现; (b) 纯代数归约 (bilinearization) 提供 gadget-free 替代路径; (c) 超越函数系数的 decidability 边界已划定 (EXPSPACE/3EXP) |

### 撞车评估 (三轮合计)

本轮 9 篇新读论文均不与 idea 核心 claims 撞车。它们分别位于 ∃R-hardness 理论（4 篇）、PG/AC convergence theory（4 篇）和函数穷举 counting（1 篇）领域。没有任何一篇研究 FEX/PDE 表达式搜索的复杂度分解、quotient-count theorem、或 FEX-specific PG convergence。三轮累计 19 篇 idea-scope 精读后（加上之前 topic-scope 读的 ~160 篇），idea 的核心 novelty gap（FEX-specific discoverability factorization: quotient cover × PG geometry × coefficient feasibility）**完整保持**，且工具箱（∃R reduction techniques + PG convergence templates + exhaustive counting）已充分完备。
