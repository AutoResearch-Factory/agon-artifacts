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

<review date="2026-06-17">

## Novelty

- Score: 8/10
- Closest prior work: Multi-Scale FEX (2510.22497, Hardwick & Yang 2025); Domain-Aware Symbolic Priors for SR (2503.09592, Huang & Yang et al. 2025); MSPINN (2508.17902); PRISMA (2512.01370); DSO (2505.10762)
- Key differentiator: 谱估计本身不是新东西。本 idea 的核心差异化在两个层面。第一, 把可观测频谱信号编译成 RL expression-tree controller 的软频率动作先验, 从而缩小离散频率搜索空间——所有已有谱先验工作 (MSPINN/RUNNs/IFeF-PINN/PRISMA/FRES) 注入的是连续神经优化 (初始化/注意力/特征), 无一涉及离散 RL 表达式树搜索空间缩减。Yang 组自己的 Domain-Aware Symbolic Priors (2503.09592) 用领域语料库统计作为符号先验, 不是 PDE 频谱估计。第二, the diagnostic-first architecture: oracle frequency gate 作为 causally isolate frequency-search bottleneck 的工具, 在文献中完全空白。多轮 targeted search (6 组 query, S2/arXiv/Web) 未发现任何 spectral estimation → soft frequency prior → RL symbolic PDE search 的已有工作。若方法最终不新, 可能新的是 finding: FEX 的 wide-frequency failure 是否可被频率搜索瓶颈因果解释。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 8/10 | v5 相比 v4 关闭了 RHS 泄漏 gap: naive RHS-FFT 现在只是候选估计器, 必须通过 decoy/nonlinear gates 后才可支撑 method claim。decoy RHS negative control 和 real-FEX-family smoke 的加入使 proxy 证据链更完整。剩余 gap: (a) 真实 Multi-Scale FEX oracle 实验仍是缺失的——proxy PG 和 dictionary lookup 的确定性成功不等于 RL controller 受频率动作拖累; (b) Adam/BFGS 系数优化、reward 噪声和 expression-tree credit assignment 三者可能非线性耦合, oracle gate 无法完全 disentangle; (c) "candidate validation" 需要固定预算约束, 否则可能 quietly become brute-force search。 |
| Missing evidence signals | 7/10 | v5 新增 risk-seeking PG proxy (64 runs, uniform vs oracle vs estimated vs wrong+3 prior), decoy RHS leakage control (FFT top-1 选中 (11,3) 而漏掉真值 (4,17)), 和 real-FEX-family `helmholtz_sine` smoke (standard 失败 relL2 5.21e-2 vs oracle skeleton 成功 5.01e-8)。proxy 证据质量显著提升。但决定性证据仍然缺失: (a) 真实 Multi-Scale FEX 上 standard vs oracle-soft vs estimated-soft 的 3-5 seed 搜索曲线 (equal candidate/wall-clock budget, frequency-resolved error); (b) 非平凡非线性 PDE 上 RHS 与解频谱真正不同时的 estimator behavior; (c) CWT/SST ridge 在真实 oscillatory PDE 上的实测精度和 overhead。Gate 0 仍是所有 method claim 的前提条件, 当前未执行。 |
| Narrative | 9/10 | "Frequency-Search Bottleneck Diagnosis → Checked Spectral Priors as Search-Space Compilers" 是当前最锐利的 framing。因果先行 (先诊断再修复) 的叙事结构远超加预处理模块的写法。v5 对 RHS 泄漏 concern 的 explicit acceptance (naive RHS-FFT 只能是候选估计器) 展现了 intellectual honesty, 反而加强了叙事可信度。"Compiler" 隐喻的含义在 body 中已限定为 "prior → bounded frequency action distribution → smaller controller search", 不会误导成端到端 PDE compiler。Codex 建议改为 "Causal Bottleneck Audit for Frequency Actions in RL Symbolic PDE Solvers", 这个 framing 强调 diagnosis-first 价值, 值得考虑但不强制。 |
| Venue contribution | 7/10 | Topic 未声明 target-venue, 按 FEX/SciML 顶会标准推断。若真实 FEX oracle 成功且 estimated prior 在非平凡 PDE 上接近 oracle: 强 NeurIPS/ICLR candidate (method+diagnostic 双贡献)。若仅 oracle 成功但 estimated prior 弱: 可作 diagnostic paper 投稿 JMLR/JCP 或 SciML 专题。若 oracle 失败: 应归档或转向连续参数耦合/reward 诊断。当前贡献的顶会分量完全取决于 Gate 0 的结果。 |
| Testability | 10/10 | 最便宜反证清晰: "oracle-soft FEX 不优于标准 FEX" 直接否定"频率搜索是主瓶颈"的中心因果解释。v5 的 POSITIVE/NULL-A/NULL-B/LEAKAGE-FAIL/DEGENERATE 五态 claims 界限清晰可预注册。decoy RHS gate 是出色的 negative control——若正结果只来自 manufactured RHS 直接暴露解频率的设置, claim 降级为 "blind RHS prior is unsafe"。`25%/35%` prior-width rule 防止 "先验变 dense grid" 后仍硬 claim 搜索空间缩小。Gate 0→1→2→3 的级联实验设计逻辑严密。 |
| Outcome realism | 8/10 | >50% candidate evaluation 降低在 d<=3、well-separated、频率可观测的窄 regime 下是现实的——如果频率搜索确为主瓶颈。net wall-clock >50% 更难, 因为 Adam/BFGS 系数优化和 validation overhead 可能在真实 FEX 中主导 wall-clock。v5 对 outcome 的限定比 v4 更严格: 加入 RHS decoy、solution/RHS spectrum mismatch 和 nonuniform samples 不退化为 blind FFT 的条件, 使 success claim 更难满足但更可信。 |
| Contribution type compliance | n.a. | idea types = {method, diagnostic} ⊆ preferred-contribution-types: n.a. (topic 0616-fex.md 未声明 `preferred-contribution-types` 字段, 跳过检查) |
| Overall Quality | 8/10 | v5 是 disciplined improvement: PG proxy + decoy RHS + real-FEX-family smoke 三项新增使 proxy evidence 显著增强, RHS 泄漏 concern 被 explicit 接受并转化为 gate condition。但决定性 evidence gap (真实 Multi-Scale FEX oracle) 未变——这是当前 blocking item, 不是设计缺陷。Codex 总体评分低 1 分 (更保守), 但核心判断一致。 |

## Contribution Drift (n=5)

- v_4 contribution types: {method, diagnostic}
- v_5 contribution types: {method, diagnostic}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 preferred-contribution-types; types 完全一致, 无 expansion/downgrade)

v4 review 每条 concern 的当前状态:
- "proxy 在手工 sine dictionary 上完成, 非真实 FEX controller" → **partially resolved** (v5 新增 PG proxy + real-FEX-family smoke, 但真实 Multi-Scale FEX oracle 未执行)
- "RHS 频谱与解频谱在非线性算子下可能不一致" → **resolved in design** (v5 加入 decoy RHS negative control, 实测证实泄漏风险; naive RHS-FFT 降级为候选估计器, 必须通过 decoy/nonlinear gates)
- "soft prior 过宽退化为 dense grid" → **resolved** (v4 已加入 `25%/35%` rule, v5 维持)
- "真实 Multi-Scale FEX benchmark 上跑 oracle" → **still planned** (Gate 0), 尚未执行
- "multi-seed RL 搜索曲线" → **still planned** (Gate 0), 尚未执行
- "频率估计开销纳入 wall-clock" → **resolved** (v4 已明确, v5 维持)
- "estimator 对比 (NCPSD/synchrosqueezing)" → **resolved** (v4 Gate 2, v5 维持)
- "非规则域/非均匀采样鲁棒性" → **resolved** (v3 已加入, v5 维持)
- "频率误差相关性分析" → **resolved** (v3 已加入, v5 维持)
- "PINN prior conflict" pushback → **accepted** (refiner 正确指出 MSPINN/IFeF/PRISMA 修连续优化, 不修离散 RL 频率搜索; codex 也认可)
- "oracle gate 应提升为主要贡献" → **resolved** (v4 已加入 diagnostic type)
- v4 Alternative Framing 建议 → **resolved** (v4 已全文采纳)
- v4 Codex 提出的 "manufactured PDE RHS 可能泄漏答案" → **resolved in design** (v5 explicit 接受为 gate condition, pilot 实测证实)

Refiner pushback 评估:
- Benchmark/application expansion pushback: 有据, 接受。多 PDE、多 seed 和非规则采样只服务 method evidence。
- PINN/spectral-prior conflict pushback: 有据, 接受。已有工作修连续优化, 不修离散 RL 频率搜索。

## Alternative Framing

NONE. v5 当前的 "Frequency-Search Bottleneck Diagnosis for RL Symbolic PDE Solvers, with Checked Spectral Priors as Search-Space Compilers" 已经是该 idea 最锐利的 framing。Codex 建议的 "Causal Bottleneck Audit for Frequency Actions in RL Symbolic PDE Solvers" 强调 diagnosis-first 价值, 语义上等价但用词更精确 ("audit" > "diagnosis" 在某些圈子), 属风格选择而非实质性改进。当前 framing 无需修改。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在 d<=3、频率可观测、well-separated 的振荡 PDE 上, oracle-soft 和 estimated-soft spectral prior 使真实 Multi-Scale FEX 以 <50% candidate evaluations 或 net wall-clock 达到同等 relative L2, selected prior 覆盖 <=25% dense frequency grid, estimator overhead 计入后 net gain 仍为正, 且 RHS decoy 和 solution/RHS spectrum mismatch 控制通过——即正结果不是从 manufactured RHS 读答案。 |
| NULL-A | oracle 有效但 estimated prior 无效或需要 >35% dense grid: 编译器接口 (frequency prior → controller) 范式成立, 但当前 estimator 是瓶颈 (NULL-A) 或频谱可估计但不可用于搜索空间缩减 (DEGENERATE)。 |
| NULL-B | oracle-soft 不优于标准 Multi-Scale FEX: 频率搜索不是 FEX 的主要瓶颈, 论文转为负诊断——定位连续参数耦合、residual reward 失真或 expression-tree credit assignment 为真正根因。 |
| LEAKAGE-FAIL | RHS FFT 仅当 manufactured RHS 直接暴露解频率时有效; decoy RHS control 暴露盲 FFT 失败后, claim 降级为 "blind RHS prior is unsafe", 仅 early-residual 或 validated 估计器可支撑方法声称。 |
| NEGATIVE-CONTROL | 硬冻结在微小频率误差 (+1) 下失败而 soft prior (radius>=1) 恢复: 仅声称 soft/adaptive 注入是必要设计选择, 不声称谱先验整体解决 FEX 频率问题。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  **Likelihood = Medium**: 成稿路径清晰 (Gate 0 oracle → Gate 1 injection → Gate 2 estimator → Gate 3 robustness), proxy pilot (v5 PG proxy + decoy RHS + real-FEX-family smoke) 给出 POSITIVE but bounded 信号, falsifier 和退化线设计扎实。但强依赖真实 Multi-Scale FEX oracle 成功——proxy PG/dictionary lookup 的确定性成功不等价于 RL controller 受频率动作拖累。Adam/BFGS 系数优化、reward 噪声和 expression-tree credit assignment 可能在真实 FEX 中压缩 oracle gain。"candidate validation" 若无固定预算约束可能 quietly become brute-force search。估计 oracle gate 通过概率 ~50-60%。Claude 与 Codex 对 Likelihood 判断一致为 Medium, **无分歧**。
  **Impact = High**: 若成功, 将 (a) 确立"将经典频谱分析编译为 RL 符号搜索先验"这一此前缺失的设计范式; (b) 直接定位并修复 Multi-Scale FEX 论文明确承认的 wide-frequency-separation 失效模式; (c) 证明 Yang 两条独立学术脉络 (FEX RL search + SynLab spectral estimation) 的合成价值。影响集中在 FEX/RL-symbolic-PDE 研究线, 不会改变全 SciML/ML 领域判断, 但在此明确研究线上形成强 top-venue 结果。Claude 与 Codex 对 Impact 判断一致为 High, **无分歧**。

## Overall

- Priority: High
- Score: 7
- Comments: v5 是一次 disciplined improvement: 没有 silent downgrade, 没有越界扩张, 三项新增 proxy evidence (PG proxy, decoy RHS, real-FEX-family smoke) 显著增强了 pilot 的可信度。RHS 泄漏 concern 被 explicit 接受并转化为 gate condition, 展现了 intellectual honesty。当前最紧迫的下一步仍然是 Gate 0: 用真实 Multi-Scale FEX codebase 跑 oracle-frequency vs standard FEX 的 3-5 seed 对比 (预计 2-3 天 compute)。若 oracle 通过, 本 idea 具有 High (7) 的顶会潜力; 若 oracle 失败, 应及时归档或转向连续参数耦合/reward 诊断。Claude 与 Codex 在 Likelihood/Impact/Priority/Score 上完全一致 (均为 Medium×High=7), 仅在个别 Quality 子维度存在 1 分差异 (Codex 更保守), 不构成实质性分歧。Codex 的 "candidate validation 需要固定预算约束" concern 值得在实验设计中注意。

</review>
