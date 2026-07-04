---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-search-complexity/
---

- One-sentence summary: 建立 FEX search-complexity factorization theorem, 把 FEX 表达式搜索难度写成可分开分析的三项: quotient-cover size、policy-gradient landscape 常数和实值系数可行性.
- Problem anchor: "FEX 是我的课题组(Haizhao Yang 组)发明的一种方法, 使用 RL 进行 Symbolic Regression, 我(Youran Sun)作为 Yang 的博后, 应该继续在这个方向上探索"; 本 idea 只研究 FEX/PDE 表达式搜索的复杂度边界, 不漂移成通用 SR benchmark 或新工程系统.
- Hypothesis: FEX 的维度困难不是单一的表达式树数量爆炸. 对低 FEX cover 的 Fourier、Barron-like 或可分离 PDE 解类, 结构搜索的组合项可以很小; 但 controller 是否快速把概率放到好类上取决于独立的 `κ_PG`, 而选中结构后的 real-coefficient residual feasibility 仍可能继承 ∃R 困难.
- Expected outcome: 成功版本是一组能写进论文的 theorem statements: (i) 定义 `Q_L(R)` 为深度 `L` FEX grammar 在 rewrite set `R` 下的 quotient template classes, 并定义 `N_FEX(F,L,eps)` 为覆盖函数族 `F` 所需的 quotient classes 加参数 cover; (ii) 证明 counting lemma: `Q_L` 可由 canonicalization 精确计算, 且若只有 `K` 个 eps-good classes, distribution-free sampler 在均匀对抗先验下有 `Ω(Q_L/K)` hit-time lower bound; (iii) 给出 conditional PG theorem: 若 `N_FEX=poly(d,1/eps)` 且 expected reward 满足 `J*-J(theta) <= κ_PG ||grad J(theta)||^2`、梯度方差有界, 则 softmax/f-softargmax controller 的结构 gap 在 `poly(N_FEX, κ_PG, 1/eps)` 更新内下降; (iv) 以 bounded ETR/ETR-INV 为来源问题, 把 `x+y=z`、`xy=z` 和边界约束映射到 FEX residual squares, 证明 fixed-structure real-parameter feasibility 的 ∃R-hardness. 最便宜的证伪信号是: depth3/4 quotient 几乎不压缩, 或真实 FEX trace 的 `κ_PG` proxy 随维度/好类稀疏度爆炸到让 conditional theorem 失去解释力, 或 ETR gadget 不能在 FEX grammar 中保持有界实变量约束.
- Contribution type: theory
- Risk: HIGH
- Estimated effort:
  - Compute: 20-80 GPU-hours for diagnostic sweeps with PG trace; theorem work is mostly 0 GPU-hours
  - Data: available
  - Implementation: Layer 1 counting/cover formalization needs weeks; Layer 2 conditional PG proof plus trace diagnostics needs 3-12 months; Layer 3 full ∃R reduction needs months and may become a separate theory note
- Novelty quick-check: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026), partially preempts generic SR tractability through FPT/W-hierarchy, so this idea should not sell raw SR counting as novel. What remains open is FEX-specific: quotient cover for PDE residual expression trees, PG geometry for the FEX controller, and ∃R hardness of real-coefficient residual feasibility; PG convergence papers and ETR compendia provide tools but do not analyze this object.
- Strongest objection: This may still be short of a top-venue theory paper if `κ_PG` remains only an assumption and the ∃R route stops at toy arithmetic gadgets.
- Why we should do this: FEX already has approximation theory, but that only says a compact expression exists. This factorization tells the group where the hard part sits: grammar cover, controller geometry, or coefficient feasibility.
- Pilot:
  - Setup: v3 Poisson sweep and quotient probe were kept; v4 adds controller `pg_trace` logging, one short CUDA trace sanity, a quotient-sized softmax-bandit PL proxy with 64 seeds, and a toy FEX-style residual gadget for `x*y=2, x+y=3`.
  - Metric: Use `kappa_proxy=(J*-J)/||grad J||^2` to test whether cover size alone controls PG geometry; require the toy arithmetic gadget to reach residual `<1e-10`; treat the real-controller trace only as logging sanity, not scaling evidence.
  - Result: Conservative counts stay `depth1` raw 243 -> quotient 136 and `depth2_sub` raw 2187 -> quotient 953. For `depth2_sub`, median kappa proxy is about `9.0e5` with one good class, `2.9e4` with 31 good classes, and `9.6e3` with 95 good classes. The toy residual gadget succeeds in 8/8 CUDA runs; the real trace records 6 steps with median grad norm 0.00876 and median slot entropy 1.922, but no 0.99 hit.
  - Signal: WEAK

- Claims and Claims matrix: Positive claim: "FEX search difficulty factorizes into quotient cover, PG geometry, and real feasibility; with polynomial cover and polynomial `κ_PG`, structure search has polynomial conditional complexity, while fixed-structure real-parameter feasibility is ∃R-hard." Null claim: "Even if `κ_PG` is not polynomial, the theorem identifies the failing factor and rules out the false inference that small cover implies fast PG." Negative retreat: if the ∃R proof fails, keep the counting lemma plus conditional PG theorem and state only that FEX residual gadgets embed bounded arithmetic constraints, not that feasibility is hard.
- Narrative: The paper should start from the gap between FEX approximation and FEX search. "The expression exists" is only one factor; the controller still needs useful reward geometry, and the coefficient solver may still face real-algebraic feasibility.
- Experiments: Keep experiments diagnostic, not benchmark-like: extend quotient counts to depth3/4 with e-graph rewrite sets; rerun Poisson/Fourier/separable/nonseparable PDE families with `pg_trace`; compare softmax and f-softargmax controllers through `κ_PG` proxies; report loose structure-hit thresholds separately from strict residual thresholds; include one hand-checked bounded-ETR gadget before claiming any hardness theorem.
- Assets status: No external data is needed; v4 CUDA trace/probe artifacts are complete, with details in `workspace/fex-search-complexity/data/MANIFEST.md`.
- Review handling: I accept the request to stop adding breadth and now center the idea on a theorem skeleton. I accept the `N_FEX` formalization criticism by separating `Q_L(R)`, function-family cover, and parameter cover. I accept the PG bridge criticism and make the bridge explicitly conditional; the v4 bandit probe supports a pushback against the stronger mistaken inference that cover size alone should imply PG convergence. I accept the ETR criticism only as far as the current evidence goes: v4 validates a residual-gadget interface, but does not claim ∃R-hardness until a bounded-ETR/ETR-INV reduction is written. I accept the logging-gap criticism by adding real-controller `pg_trace`; I do not turn the requested extra PDE/Fourier checks into a benchmark contribution. Contribution drift note: v3 was `theory`; v4 remains `theory`, with no method, benchmark, diagnostic, application, or empirical-finding type added or removed.

<review date="2026-06-17">

## Novelty

- Score: 8/10
- Closest prior work: Soubki & Cranmer, *When Is Symbolic Regression Tractable?* (ICML 2026); Jiang et al., *EGG-SR* (2511.05849, ICLR 2026); Virgolin & Pissis, *Symbolic Regression is NP-hard* (TMLR 2022)
- Key differentiator: 该 idea 不声称 generic SR tractability 或 equivalence pruning 为 novel. 防御性新颖性在于 FEX-specific 三因子分解: (a) FEX grammar 的 quotient-cover 形式化与计数——无任何 prior work 对 PDE residual expression-tree grammar 做 canonicalization-based counting; (b) expression-tree action space 上 PG 收敛的 conditional theorem——EGG-DRL 证明了 variance reduction 而非 convergence rate, Kumar/Ju 等 generic PG 理论未 bridge 到 expression-tree grammar 结构, VSR-DPG/GCN-SR/SYMPOL 均为纯 empirical; (c) FEX real-coefficient feasibility 的 ∃R-hardness——Virgolin (NP-hard) 和 Soubki (FPT/W-hierarchy) 均未涉及 ∃R. 三个 claim 均独立搜索并确认为未被覆盖.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v4 将 formalization gap 从 "骨架缺失" 升级为 "骨架有但肌肉待填": theorem skeleton 已落实到代码与 md, `N_FEX` 定义存在, quotient count 已计算, κ_PG proxy 已测量. 剩余三大桥接缺口: (a) canonical quotient class formalization 仍需从 syntactic rewrites 升级到 semantic equivalence (需与 EGG-SR 的 e-graph rewrite rules 对齐); (b) cover size → PG landscape geometry 的桥接引理依然 null——软最大化 bandit proxy 证明了 cover size alone 不控制 κ_PG, 但未给出正向 bridge; (c) bounded-ETR→FEX 仍为 route 而非 reduction. `Ω(Q_L/K)` lower bound 如 codex 所指, 需 sharp specification of sampler class 否则退化为 trivial no-free-lunch lemma. |
| Missing evidence signals | 7/10 | v4 新增四项有效信号: real-controller `pg_trace` (填补 v3 的 logits/grad-norm logging gap), softmax-bandit PL proxy (揭示 κ_PG 随 good class count 降低), toy ETR gadget (8/8 success, validates reduction interface), theorem skeleton formalization in code. Still missing: depth3/depth4 quotient counts (would reveal whether compression collapses at scale), e-graph-assisted quotient comparison (当前仅 conservative commutativity+constant collapse), Fourier/separable/nonseparable PDE families 的跨域对比. 这些缺失主要是 diagnostic breadth, 不影响 proximal 入口门. |
| Narrative | 8/10 | "FEX approximation theory proves existence, but this factorization tells the group where discoverability fails" 的 arc 干净且有力. 三因子被重构为乘积关系 (difficulty = cover × PG × feasibility) 而非并列 decomposition, 使 conditional PG theorem 从 "打折扣的正面" 变成 "factorization 的必要组件." Soubki & Cranmer 被显式引用并区分, 形成 "他们证明了 generic SR 的 FPT/W-hierarchy 结构, 我们证明 FEX-specific 的 discoverability 结构" 的叙事配合, 而非竞争. |
| Venue contribution | 6/10 | Topic 无 `target-venue`, 推断为 ICML/NeurIPS/ICLR theory track. 若三个 layer 中至少一个 unconditional theorem 兑现 (counting lemma 最可行) + conditional PG theorem + ∃R sketch: solid top-venue theory paper. 仅 counting lemma + conditional PG (assumed PL/gradient domination): 可能不够顶会, 除非 FEX-specific structure 能以新方式控制或诊断 κ_PG. 该判断与 codex 一致. |
| Testability | 8/10 | Expected outcome 中 "depth3/4 quotient 几乎不压缩"、"κ_PG proxy 随维度/好类稀疏度爆炸"、"ETR gadget 不能在 FEX grammar 中保持有界实变量约束" 均为合理且便宜的 falsifier. v4 的数据已在三个层面部分测试: depth2_sub 压缩 57% (正面信号), κ_PG proxy 随好类数下降 (支持 factorization framing), toy ETR gadget 8/8 成功 (validate interface 而非 hardness). 每个 falsifier 杀死的是因子而非全 program, 这是正确的 granularity. |
| Outcome realism | 7/10 | Effort split 诚实: Layer 1 counting/canonicalization weeks, Layer 2 conditional PG 3-12 months (conditional version lowers bar), Layer 3 full ∃R reduction months-to-years. Realistic top-venue 版本: counting lemma (unconditional) + conditional PG theorem (diagnostic: cover alone does not imply fast PG, but if κ_PG is poly, convergence is fast) + ∃R reduction sketch (validated 通过 toy gadget + 完整 bounded-ETR mapping). 三个 layer 全部 unconditional 兑现仍需 1-3 年, 但 idea 已清晰退路. |
| Contribution type compliance | n.a. | idea types = {theory}; topic 未声明 `preferred-contribution-types`, 本项跳过, 不计入 Overall Quality. |
| Overall Quality | 7/10 | v4 是 v3 的实质性增量: 从 "定义有、计数有、conditional bridge 有" 升级到 "theorem skeleton 有、diagnostic probes 有、real-controller trace 有." 主要风险已从 "well-framed research agenda" 转移为 "能否把 theorem skeleton 转换成至少一个 non-trivial theorem statement with proof." 这与 codex 的 assessment 一致——next version should stop refining prose and produce one hard artifact. |

## Contribution Drift

- v_{n-1} contribution types: {theory}
- v_n contribution types: {theory}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`, 且 v4 在 Review handling 段显式声明 "v3 was `theory`; v4 remains `theory`")

## Prior Review Resolution (v3 concerns)

| v3 Concern | Status | Notes |
|------------|--------|-------|
| `N_FEX` formal treatment 仍为 sketch 级 (v3 score 6/10) | **Partially resolved** | v4 将 `Q_L(R)` (pure grammar quotient), function-family cover, parameter cover 三者分离; `N_FEX` 定义在 `theorem_skeleton()` 中 explicit. 但 full formalization (metric, equivalence oracle, parameter-cover structure) 仍未完成. |
| 缺 logits/grad-norm sweep 的 PL proxy (v3 self-recorded gap) | **Resolved** | v4 新增 real-controller `pg_trace` (6 steps, median grad norm 0.00876) + softmax-bandit PL proxy (κ_PG ~9e5→9.6e3 随好类数增加). |
| 缺 finite Fourier / separable/nonseparable PDE families (v3 要求) | **Not resolved — pushback accepted** | Refiner 声明 "I do not turn the requested extra PDE/Fourier checks into a benchmark contribution." Pushback 有据: idea 声明为 theory type, 跨 PDE 对比是 diagnostic 不是 benchmark. 但缺失仍削弱 "theory objects are non-vacuous" 的说服力. |
| 缺 ETR-INV→FEX toy reduction (v3 要求) | **Partially resolved** | v4 的 toy ETR gadget 验证了 residual interface (8/8 success), 但非 hardness reduction. bounded-ETR mapping 仍为 route. |
| cover→gradient domination 桥接引理缺失 (v3 score 6/10) | **Partially resolved** | v4 的 softmax-bandit proxy 证明了 _negative_ 方向: cover size alone 不控制 PG geometry. 正向 bridge (cover + reward structure → κ_PG bound) 仍然为 0. |
| EGG-SR 等价类未整合进 `N_FEX` (v3 要求) | **Not resolved** | v4 body 未讨论 EGG-SR 的 e-graph rewrite rules 与 `Q_L(R)` 的 quotient canonicalization 的关系. Codex 也指出需 "e-graph rewrite comparison." |
| depth3/depth4 quotient counts (v3 隐含要求) | **Not resolved** | v4 仅保留 depth1 (136) 和 depth2_sub (953) 的保守计数. 扩展到更深 tree 的 quotient 行为仍未知. |
| 日志缺口 (缺 logits/grad-norms, v3 self-recorded) | **Resolved** | controller `pg_trace` 已实现并验证. |

## Alternative Framing

当前 "FEX search-complexity factorization theorem" framing 已 sharp. Codex 建议 "FEX discoverability theorem" 作为 reframing: 将中心 claim 从 "difficulty 如何分解" 转为 "FEX approximation theory proves existence, but discoverability requires three separate conditions." 这是一个微调建议, 不改变本质内容, 但让 paper 对 "applied FEX" 读者的吸引力更强 (因为 "我的 FEX 为什么搜不到答案" 比 "complexity 怎么分解" 更 immediate). 两个 framing 可并存: "discoverability theorem" 是 intro hook, "factorization theorem" 是 main technical result.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | "For a function family F with N_FEX(F,L,eps) = poly(d,1/eps) and a softmax/f-softargmax controller whose expected-reward landscape satisfies gradient domination with polynomial κ_PG, the controller finds an ε-good structure in poly(N_FEX, κ_PG, 1/ε) updates. Separately, if the bounded-ETR/ETR-INV reduction is completed, fixed-structure FEX real-parameter feasibility is ∃R-hard." 必须显式标注 conditional 部分, 不可暗示 unconditional. |
| NULL | "Small quotient cover alone does not imply fast PG search — the factorization shows that cover size, PG geometry, and real-coefficient feasibility are three separable factors; the theorem identifies which factor fails." 不可声称已证明 unconditional polynomial convergence. |
| NEGATIVE | "If depth3/4 quotient compression collapses, κ_PG proxy explodes, or bounded-ETR gadgets cannot be embedded in FEX grammar, we retreat to counting lemma + conditional PG diagnostics + toy arithmetic embedding (NOT ∃R-hardness or unconditional tractability), and explicitly distinguish from Virgolin (NP-hard), Song (graph-based NP-hard), and Soubki & Cranmer (FPT/W-hierarchy)." |

## Likelihood-Impact Matrix

- Priority: High (7) = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  - **Likelihood: Medium** — v4 有三条清晰退路 (counting lemma + conditional PG + ∃R sketch), diagnostic evidence 支撑每个 layer. 一条 unconditional theorem 已有 feasibility evidence (exact quotient count + toy gadget validation). 主要不确定性: cover-to-PG bridge lemma 仍未填充; ∃R 从 route 到 reduction 有非平凡 proof gap. 整体有明确成稿路径, 依赖至少一个 unconditional theorem 成立, 非纯投机.
  - **Impact: High** — 若 factorization theorem 成立, 将 fundamentally change 社区对 expression-tree RL 搜索的理解: "维度灾难不在离散搜索自身, 而在连续优化 feasibility." Soubki & Cranmer (ICML 2026) 的存在增强 relevance 和 urgency——社区刚意识到 SR complexity 可分析, FEX-specific discoverability 分析正好赶上窗口. 即使仅 counting + conditional PG + ∃R sketch, 也是 FEX/Yang 脉络中长期缺失的 "search complexity" 拼图.
  - Claude 与 codex 对 Likelihood (均 Medium) 和 Impact (均 High) 无分歧.

## Overall

- Priority: High
- Score: 7
- Comments: v4 是连续三轮的实质性改进——从 v1 的 "方向对但工具全错" 到 v2 的 "方向对、工具对、proof bridge 待建", 到 v3 的 "定义有、计数有、conditional bridge 有", 再到 v4 的 "theorem skeleton 有、diagnostic probes 有、real-controller trace 有." 三因子乘积 framing 和 conditional PG theorem 的 diagnostic 角色转换是 v4 的核心贡献. 当前主要风险不再是 novelty (三个 claim 仍 genuinely novel, 独立搜索确认), 而是 paper 仍只是 "well-characterized research agenda with compelling evidence"——v5 应停止 refining prose, 产出至少一个 hard artifact: either a complete bounded-ETR-to-FEX reduction 或一个 nontrivial formal quotient-cover theorem with proof. Claude 与 codex 独立评估一致收敛到 Medium Likelihood × High Impact = High (7). 无 contribution drift, 无 hard cap.

</review>
