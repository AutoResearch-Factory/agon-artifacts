---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-search-complexity/
---

- One-sentence summary: 建立 FEX 表达式树搜索的复杂度分解定理, 把等价类覆盖数、policy-gradient 搜索条件和实值系数可行性分开, 说明 FEX 什么时候可搜、什么时候只是把困难转移到连续优化.
- Problem anchor: "FEX 是我的课题组(Haizhao Yang 组)发明的一种方法, 使用 RL 进行 Symbolic Regression, 我(Youran Sun)作为 Yang 的博后, 应该继续在这个方向上探索"; 本 idea 只研究 FEX/PDE 表达式搜索的复杂度边界, 不漂移成通用 SR benchmark 或新工程系统.
- Hypothesis: FEX 的维度困难不是一个单一的 "表达式树数量爆炸" 问题. 对低 FEX-cover 的 Fourier、Barron-like 或可分离 PDE 解类, 离散结构搜索可能由等价类覆盖数控制; 但 full real-coefficient residual feasibility 会继承实代数可行性的困难, 这解释了 pilot 中结构阈值可达而高精度系数优化失败的现象.
- Expected outcome: 成功版本是一条分解定理: (i) 定义 `N_FEX(F,L,eps)` 为函数族 `F` 在深度 `L` FEX grammar 的等价模板类 epsilon-cover size, 其中模板类 quotient 掉加法/乘法交换、常量族和可证明的算子恒等式; (ii) 证明结构搜索步数上界可写为 `poly(N_FEX, 1/eps, kappa_PG)` 或在 softmax/f-softargmax controller 下的条件定理, 其中 `kappa_PG` 是 expected-reward 几何里的 gradient-domination/PL 常数; (iii) 以 ETR-INV 或 bounded ETR 为 candidate complete problem, 构造 FEX residual-feasibility 的 ∃R-hardness. 最便宜的证伪信号是: 小树 quotient 几乎不压缩、或 logits/grad-norm sweep 显示 `kappa_PG` 随维度指数退化.
- Contribution type: theory
- Risk: HIGH
- Estimated effort:
  - Compute: 20-60 GPU-hours for diagnostic sweeps; theorem work is mostly 0 GPU-hours
  - Data: available
  - Implementation: Layer 1 quotient/counting needs weeks; Layer 3 ∃R reduction needs months; Layer 2 PG theorem is 6-18 months unless kept conditional
- Novelty quick-check: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026), partly occupies generic SR tractability through FPT/W-hierarchy results, so v3 no longer claims "SR search counting" as the main novelty. The remaining gap is FEX-specific: PDE residual search, RL controller step/sample complexity, and real-coefficient ∃R feasibility are not covered by Soubki & Cranmer, Virgolin & Pissis (2207.01018), Song et al. (2404.13820), or PGTS (2506.07054), which studies lookahead PG in MDPs and leaves convergence-rate/sample-based extensions open.
- Strongest objection: Layer 2 可能只能做成条件定理, 因为小 quotient cover 本身不推出好的 policy-gradient 几何.
- Why we should do this: FEX already has approximation theory, and Yang-line NN work already studies optimization curse-of-dimensionality; the missing piece is the search layer between existence and continuous optimization. A decomposition theorem is useful even if the positive PG theorem stays conditional, because it tells future FEX work which object must be improved: grammar cover, controller landscape, or coefficient solver.
- Pilot:
  - Setup: Existing CUDA Poisson sweep on `depth2_sub`, dimensions 2/5/10/20/30, three seeds, plus a v3 proof-gap probe that enumerates conservative quotient classes for `depth1` and `depth2_sub`.
  - Metric: Treat loose reward hits (0.90/0.95) as structure-search evidence and strict reward 0.99 as coefficient-polishing evidence; for quotienting, require a nontrivial reduction from raw action templates without claiming full semantic equivalence.
  - Result: Poisson sweep still shows flat/noisy hit-time scaling at 0.99 (slope 0.067, R2 0.14), with 13/15 hits at 0.95 and 11/15 at 0.99. The v3 probe gives `depth1` raw 243 -> quotient 136 and `depth2_sub` raw 2187 -> quotient 953 under commutativity plus constant-family collapse; it also records that current logs cannot estimate a PL constant because logits and gradient norms were not stored.
  - Signal: WEAK

- Claims and Claims matrix: Claim A: The counted object is `N_FEX(F,L,eps)`, a cover by quotient classes of FEX templates plus optimizable real parameters, not the raw number of action strings; the first exact sanity count is 2187 raw `depth2_sub` templates collapsing to 953 conservative quotient classes. Claim B: A positive search theorem must say "cover + PG geometry", not "cover implies PG convergence"; if a polynomial `kappa_PG` is proved, the controller gets a polynomial step bound, and if it is only assumed, the theorem is conditional. Claim C: The negative result should target FEX residual-feasibility with real coefficients via ETR-INV/bounded-ETR gadgets: variables become real leaf/affine parameters, `x+y=z` and `xy=z` constraints become small FEX subtrees with squared residual penalties, and inequalities use slack or bounded-domain gadgets; until this is formal, ∃R-hardness is a candidate route, not a claimed theorem. Claim D: Empirical evidence only supports the search/optimization separation on one Poisson family, not a universal scaling law.
- Narrative: Prior theory says compact FEX expressions exist; the missing question is whether the controller can find them. v3 tells the story as a three-factor decomposition: grammar quotient controls which structures are distinguishable, PG geometry controls how fast the controller puts mass on them, and real algebraic feasibility controls whether the chosen structure can be polished.
- Experiments: Keep experiments diagnostic. Extend the Poisson sweep only enough to log controller logits, sampled actions, losses, and grad norms for a PL/gradient-domination proxy; add finite Fourier and separable/nonseparable PDE families; compare raw template count, conservative quotient count, and e-graph-assisted quotient count; report structure-hit and strict-residual thresholds separately. Do not present this as a benchmark contribution.
- Assets status: Pilot sweep and v3 proof-gap probe completed locally with CUDA sanity; no external data is required, see `workspace/fex-search-complexity/data/MANIFEST.md`.
- Review handling: I accept the Soubki & Cranmer novelty warning, so generic SR tractability is no longer the main claim; I do not treat it as a scoop because it does not analyze FEX/PDE residuals, RL controller convergence, or ∃R real-coefficient feasibility. I accept the undefined-cover criticism by defining `N_FEX(F,L,eps)` and adding the first exact quotient count. I accept the missing PG bridge by making the positive theorem conditional on a polynomial landscape constant unless that constant is proved. I accept the ∃R criticism by naming ETR-INV/bounded ETR and sketching the first gadgets, while refusing to claim QBF/PSPACE. I accept the effort criticism and split effort by layer. Contribution drift note: v2 was `theory`; v3 remains `theory`, with no method, benchmark, diagnostic, application, or empirical-finding type added or removed.

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026); Virgolin & Pissis, *Symbolic Regression is NP-hard* (TMLR 2022); PGTS 2506.07054 (lookahead PG in MDPs); DSO/RSPG 2505.10762 (LLNL framework)
- Key differentiator: The idea does NOT claim "SR search space counting" as novel — Soubki & Cranmer partially occupies that through FPT/W-hierarchy. The defensible novelty is FEX-specific: (a) quotient-cover decomposition of FEX grammar with explicit counting (first-ever: 2187→953 for depth2_sub); (b) policy-gradient convergence analysis specialized to expression-tree action spaces — no prior work bridges the PG-theory tools (PL inequality, gradient domination, f-softargmax) to FEX's discrete expression tree grammar; (c) ∃R-completeness of real-coefficient FEX residual-feasibility — no prior work connects symbolic regression or PDE solution discovery to the existential theory of reals. Claims B and C are genuinely novel mechanisms; Claim A is partially preempted in theme but not in FEX-specific instantiation.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v3 修掉了 v2 的两大硬伤 (Uniform-LGI mismatch 与 QBF/PSPACE overclaim), 且 `N_FEX` 有了定义和第一个非平凡计数. 剩余缺口: (a) `N_FEX` 的 formal treatment 仍为 sketch 级——metric、equivalence oracle、parameter-cover 结构尚未 formalize; (b) "cover + gradient domination → poly convergence" 的桥接引理仍然缺失——`kappa_PG` 是 landscape 性质, 不能直接从 cover size 推出; (c) ETR-INV→FEX 的 gadget 已有命名但尚无完整 reduction sketch. 这些不是 minor gaps, 但 v3 已将它们从 "骨架缺失" 升级为 "骨架有但肌肉待填". |
| Missing evidence signals | 7/10 | v3 有实质新证据: exact quotient count (243→136, 2187→953)、ETR-INV candidate complete problem 被命名、effort 按 layer 拆分. 仍缺: (i) logits/grad-norm sweep 的 PL proxy measurement (v3 自己记录了 "logs cannot estimate PL constant because logits and gradient norms were not stored"); (ii) finite Fourier 和 separable/nonseparable PDE families 的对比数据; (iii) 至少一个最小的 ETR-INV→FEX toy gadget 的手工构造. |
| Narrative | 8/10 | 三因子分解 (quotient cover → PG geometry → real feasibility) 是 v3 相较于 v2 最重要的叙事升级. "approximation 已解决, 但 controller 能否找到表达式未知; 困难从结构搜索转移到系数优化" 的 arc 简洁有力. Soubki & Cranmer 被显式承认并区分, 不做虚假 novelty claim. |
| Venue contribution | 6/10 | Topic 无 `target-venue` 字段, 按 topic 类型推断为 ICML/NeurIPS/ICLR theory track. 若三个 layer 全部兑现 (至少一个 unconditional theorem + 一个 conditional theorem), 是 transformative top-venue paper. Realistic 版本 (counting lemma + ∃R-hardness + conditional PG): solid theory 贡献, 可投顶会 theory track 或 TMLR. 若仅 counting lemma (且需与 Soubki & Cranmer 区分): 中等贡献. |
| Testability | 8/10 | Expected outcome 中 "小树 quotient 几乎不压缩" 和 "`kappa_PG` 随维度指数退化" 是合理且便宜的 falsifier. v3 的 pilot 已跑 quotient 压缩实验 (2187→953, ~43% 压缩), 若扩展到 depth3/4 且压缩率坍塌, 正面叙事将受重大打击. 日志缺口 (缺 logits/grad-norms) 是明确且可修复的诊断学 debt. |
| Outcome realism | 7/10 | v3 的 effort split (Layer 1: weeks; Layer 3: months; Layer 2: 6-18 months unless conditional) 比 v2 的 "weeks-months" 诚实得多. 一个 realistic top-venue 版本大概率是 counting lemma (unconditional) + conditional PG theorem + ∃R reduction sketch (无需完整 hardness proof). 三个 layer 全部 unconditional 兑现仍需 1-3 年. |
| Contribution type compliance | n.a. | idea types = {theory}; topic 未声明 `preferred-contribution-types`, 本项跳过, 不计入 Overall Quality. |
| Overall Quality | 7/10 | v3 是又一次实质性进步, 从 v2 的 "方向对、工具对、proof bridge 待建" 升级到 "定义有、计数有、candidate problem 有、bridge 的 conditional 形式有." 主要风险从 "骨架缺失" 转移到 "能否把一个好 decomposition story 转换成至少一个 non-trivial theorem." 这是正确的风险转移方向. |

## Contribution Drift

- v_{n-1} contribution types: {theory}
- v_n contribution types: {theory}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`, 且 v3 在 Review handling 段显式声明 "v2 was `theory`; v3 remains `theory`, with no method, benchmark, diagnostic, application, or empirical-finding type added or removed.")

## Prior Review Resolution (v2 concerns)

| v2 Concern | Status | Notes |
|------------|--------|-------|
| Soubki & Cranmer novelty warning — 需显式引用并区分 | **Resolved** | Novelty quick-check 段和 Review handling 段均显式处理, 不再声称 generic SR counting 为 main novelty |
| Undefined cover object — 需准确定义 + 最小非平凡计算 | **Resolved** | `N_FEX(F,L,eps)` 已定义 (though formal treatment 仍为 sketch 级), 并有 exact quotient count (2187→953) |
| Missing PG bridge — 需 cover→gradient domination 推导草图 | **Partially resolved** | 正面向导已改为 conditional, `kappa_PG` 被命名; 但从 cover 到 landscape constant 的桥接引理仍无 sketch |
| ∃R criticism — 需 candidate complete problem + gadget 线索 | **Partially resolved** | ETR-INV/bounded ETR 已命名, 基本 gadget (leaf params, subtree constraints, squared residual penalties) 已 sketch; 但尚无完整 reduction, 哪怕对一个 toy instance |
| Effort estimate 过于乐观 (v2 "weeks-months") | **Resolved** | 已按 layer 拆分: Layer 1 weeks, Layer 3 months, Layer 2 6-18 months (unless conditional) |
| EGG-SR 等价类剪枝未整合进 body | **Partially resolved** | Experiments 段提到 "e-graph-assisted quotient count" 但未说明 EGG-SR 的 equivalence class 如何与 `N_FEX` 的 quotient class 交互 |
| Deep-lit 新文献的整合 | **Resolved** | v3 body 末尾的 Deep Lit 段收录了 DSO、Exhaustive SR、eggp、PG convergence theory papers 等, 并给出了 claim-by-claim 的影响评估 |

## Alternative Framing

当前三因子分解 framing 已是 v2 review 建议的产物, 结构合理且自洽. 一个微调建议: 把 "decomposition theorem" 重新定位为 **FEX search-complexity factorization theorem**, 强调三因子是乘积关系 (difficulty = cover size × PG geometry × real feasibility), 而非简单的并列分解. 乘积 framing 的优点是: 即使某个因子未被独立证明 (如 `kappa_PG` 仅假设), 只要另两个因子被严格控制, 乘积公式仍能提供有用的 search difficulty upper/lower bound. 这个 framing 不改变 idea 的本质内容, 但让 conditional PG theorem 在 paper 中的角色从 "打折扣的正面向导" 变为 "factorization 的一个必要组件."

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | "For a function family F with N_FEX(F,L,eps) = poly(d,1/eps) and a softmax-PG controller whose expected-reward landscape satisfies a gradient-domination condition with poly(d) constant kappa_PG, the controller finds an eps-good expression structure in poly(d, 1/eps) steps." 必须显式声明两个条件 (low cover + poly kappa_PG), 不可暗示 unconditional. |
| NULL | "We establish a factorization theorem: FEX search difficulty decomposes into quotient-cover size × PG landscape condition × real-coefficient feasibility. Even without proving kappa_PG is polynomial, this decomposition enables targeted diagnosis of which component limits search." 不可声称已证明无条件 polynomial convergence. |
| NEGATIVE | "FEX residual-feasibility with real coefficients, restricted to the set of expression trees whose structure has already been selected, is ∃R-hard via reduction from ETR-INV." 必须限定 hardness 的作用域为 real-coefficient feasibility (the " polishing" phase after structure selection), 不可暗示 FEX RL controller 本身的 search process 是 ∃R-hard. 若 ∃R reduction 失败, 可 retreat 到 "integer/affine-restricted degenerate FEX is NP-hard" 但需显式区分 Virgolin (2207.01018) / Song (2404.13820) / Soubki & Cranmer (ICML 2026). |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  - **Likelihood: Medium** — v3 有三条清晰退路 (quotient counting / ∃R candidate route / conditional PG theorem), 至少一条可成 top-venue 级理论结果. counting lemma (Layer 1) 几周至 3 月可成, 已有 exact quotient count 作 feasibility evidence; ∃R reduction (Layer 3) 数月, ETR-INV gadget 路线可行但需实际构造; PG convergence (Layer 2) 是最高风险项, 但 conditional 版本可大幅降低兑现门槛. 整体有明确成稿路径, 依赖至少一个 unconditional theorem 成立. 没有 guarantee 但不是纯投机.
  - **Impact: High** — 若 factorization theorem 成立, 将 fundamentally change 社区对 expression-tree RL 搜索的理解: "维度灾难不在离散搜索自身, 而在连续优化 feasibility." 即使只有 counting + ∃R-hardness, 也是 FEX/Yang 脉络中长期缺失的 "search complexity" 拼图, 会被后续 FEX/RL-SR 工作广泛引用. Soubki & Cranmer (ICML 2026) 的存在反而增强了这个方向的 relevance 和 urgency——社区刚意识到 SR complexity 是可分析的, FEX-specific 的分析正好赶上窗口.
  - Claude 与 codex 对 Likelihood (均 Medium) 和 Impact (均 High) 无分歧.

## Overall

- Priority: High
- Score: 7
- Comments: v3 是连续两轮的实质性改进——从 v1 的 "方向对但工具全错" 到 v2 的 "方向对、工具对、proof bridge 待建", 再到 v3 的 "定义有、计数有、conditional bridge 有、candidate complete problem 有." 三因子分解叙事是 v3 最重要的贡献, 让 idea 即使正面 PG theorem 只做到 conditional 也是完整的 decomposition 贡献. 当前最大风险不是 novelty (Claims B 和 C 仍 genuinely novel), 而是 paper 仍停留在 "well-framed research agenda" 层面——v4 应该停止加 breadth, 写出至少一个 concrete theorem skeleton: 要么 formalize `N_FEX` 并证明一个 non-trivial bound, 要么构造一个最小的 ETR-INV→FEX toy reduction. 整体代码和收益比仍优秀, 建议进入下一轮 refine, 但下次审查应要求 concrete theorem statement 而非 further framing refinement.

</review>

## Deep Lit 2026-06-17 (per-idea scope, 1 round)

本次 `deep-lit-tick --scope idea 0616-fex --idea fex-search-complexity` 围绕三因子分解框架搜索 PG 收敛理论 + ∃R 复杂度文献。首轮 7 轴搜索 → 8 篇选中 → 6 篇 wiki_written + 2 篇 tex_download_failed。B7 反向扩展（6×4=24 次调用）产出 9 篇新候选但均为同组增量扩展，无新增 ETR/∃R 或表达式树 PG 方向内容，文献饱和。

### 本次新读论文（供上层并入 landscape）

| arxiv_id | title | year | 一句话解读 | 与哪个 claim 相关 |
|----------|-------|------|-----------|------------------|
| 2505.03155 | Rethinking the Global Convergence of Softmax PG with Linear FA | 2025 | 证明近似误差不是 PG 收敛的正确度量，feature 排序保持才是充要条件；O(1/T) rate | Claim B (PG geometry) |
| 2506.05953 | Learning Deterministic Policies with PG in CMDP | 2025 | 统一 primal-dual PG 框架，梯度支配假设下 dimension-free last-iterate 收敛 | Claim B (gradient domination) |
| 2503.00229 | Armijo Line-search Can Make GD Provably Faster | 2025 | (L0,L1) 非均匀光滑下 Armijo LS 在 softmax PG 上从次线性加速到线性收敛 | Claim B (PG convergence diagnosis) |
| 2503.17644 | Sample Complexity Bounds in Bilevel RL | 2025 | 利用 PL 条件建立双层 RL 首个 O(ε⁻³) 样本复杂度界；Hessian-free 一阶算法 | Claim B (PL condition methodology) |
| 2512.06244 | Auto-exploration for Online RL | 2025 | 无参数自探索 RL，SPMD + Tsallis mirror map + TORGB 梯度估计，O(ε⁻²) 样本复杂度 | Claim B (exploration automation) |
| 2509.25424 | Polychromic Objectives for RL | 2025 | Set-level 目标 + vine sampling 防止 RL 微调中探索坍缩；PPO 适配 | Claims B/D (exploration collapse) |

### 对三因子的影响评估

**Claim A (N_FEX quotient cover)**: 0 篇覆盖。PG 理论文献不涉及表达式语法等价类计数。Soubki & Cranmer (ICML 2026) 仍然是唯一接近的 prior。N_FEX 的 formal treatment 仍然是 FEX-specific gap。

**Claim B (PG convergence with κ_PG)**: 6 篇提供增量背景但无一篇分析 expression-tree action space。
- 2505.03155 + 2504.02130（未读）建立 feature representation conditions 框架，可适配到 "FEX controller representation" 分析但需 substantial translation
- 2503.00229 的 (L0,L1) 非均匀光滑框架可用来分析 FEX controller loss landscape
- 2512.06244 的 auto-exploration 框架可作为 FEX controller 不需要手动调探索参数的论据
- 关键 gap 不变：从 expression-tree cover size 到 PG landscape constant 的桥接引理无人做过

**Claim C (∃R-hardness)**: 0 篇覆盖。本轮搜索未发现任何新 ∃R/ETR 复杂度论文。ETR-INV→FEX reduction 仍是完全开放问题。

**Claim D (empirical diagnostics)**: 2509.25424 的 polychromic objective 提供了探索多样性度量的方法论，但面向 LLM RL 而非表达式树。FEX 的 logits/grad-norm logging 缺口仍未填补。

### Errors
- 2504.02130: tex_download_failed（arxiv-tool 下载 tex source 失败）
- 2511.08241: tex_download_failed（同上）

### Verdict
所有核心 claims 未被已有文献覆盖。PG 收敛理论方向文献丰富但关键的 "表达式树 PG 收敛桥接定理" 完全空白。∃R-hardness 方向完全空白。**文献调研通过，建议继续推进。**
