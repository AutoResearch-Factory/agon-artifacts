---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-synchro-prior/
---

- One-sentence summary: 先用 oracle-frequency FEX 诊断 Multi-Scale FEX 的宽频率失败是否来自 controller 频率搜索, 再把 RHS 或 early-residual 的频谱估计编译成带宽受控的 soft frequency prior.
- Hypothesis: Multi-Scale FEX 在 well-separated oscillatory PDE 上的不稳定, 可能来自 RL controller 同时搜索频率、表达式结构和连续系数。如果 oracle-soft prior 明显改善真实 FEX 搜索, 频率选择就是可干预瓶颈; 如果 RHS FFT、early-residual DFT、NCPSD 或 CWT/SST ridge 能在低开销下接近 oracle, 谱先验可以作为 search-space compiler, 而不是另一个 Fourier-feature solver. 若 oracle 也无效, 本路线应转为连续参数耦合或 reward 诊断.
- Expected outcome: 成功信号是在 d<=3、频率可观测、well-separated 的振荡 PDE 上, oracle-soft 和 estimated-soft prior 让真实 Multi-Scale FEX 以少于标准 FEX `50%` 的候选评估或 net wall-clock 达到同等 relative L2, 且估计器开销计入后仍成立. 主 claim 还要求 selected prior 覆盖不超过 dense frequency grid 的 `25%`; 若需要超过 `35%`, 结果只支持"频谱可估计", 不支持"搜索空间被有效缩小". 最便宜反证是 oracle-soft FEX 不优于标准 FEX.
- Contribution type: method + diagnostic
- Contribution drift note: v3 contribution type 是 `method`; v4 保留 `method` 并加入 supporting `diagnostic`, 因为 reviewer 要求把 frequency-search bottleneck diagnosis 从 gate 提升为主要证据. 没有删除 v3 的 method contribution, 也没有扩张成 benchmark、application 或 dataset paper.
- Risk: MEDIUM
- Estimated effort:
  - Compute: 80-120 GPU-hours, 主要用于真实 Multi-Scale FEX oracle/estimated prior 的 3-5 seed 搜索曲线、estimator overhead 和 robustness sweeps; v4 proxy 已在本机 RTX 4060 Ti 跑完.
  - Data: available; 使用 manufactured oscillatory PDE 和 Multi-Scale FEX 风格基准, 无外部数据或模型权重.
  - Implementation: 2-4 weeks for true FEX integration, prior injection modes, and ablations.
- Novelty quick-check: Multi-Scale FEX (2510.22497) 引入 spectral composition, 但频率仍由 controller 在搜索中发现. MSPINN、RUNNs、IFeF-PINN、PRISMA 和 FRES 把谱信号注入连续神经 PDE solver, 不处理 RL expression-tree search. DSO、SSDE、SymPlex、NetGP、Sym-Q 和 2024 spatio-temporal-reward PDE-SR 都是 RL/Transformer/GP 符号搜索相关 prior, 但它们没有把可观测频谱估计编译成 FEX 频率候选分布. 2026-06-17 四个 targeted S2/arXiv-tool 查询未发现 spectral estimation -> soft frequency prior -> RL symbolic PDE search 的已有工作.
- Strongest objection: v4 pilot 仍是 frequency-dictionary proxy; 真实 FEX 中 Adam/BFGS 系数优化、reward 噪声和 expression-tree credit assignment 可能吞掉频率先验的收益.
- Why we should do this: 这个版本回答一个可证伪的机制问题: FEX 的 wide-frequency failure 到底是不是频率搜索造成的. 正结果给出 diagnostic-gated spectral compiler; 负结果也会把后续工作从谱先验及时转向更可能的瓶颈.
- Review response decisions: 接受 real-FEX gap, 所以 Gate 0 仍是 oracle-soft vs standard Multi-Scale FEX, v4 不把 proxy 当决定性证据. 接受 "oracle gate 应提升为主要贡献", 所以加入 supporting diagnostic type. 接受 estimator mismatch、overhead 和 soft prior 退化批评, 所以加入 estimator selector、net wall-clock accounting 和 `25%/35%` prior-width rule. Pushback PINN/spectral-prior conflict: 这些 prior works 修连续优化, 没有缩小 RL 表达式树的离散频率搜索. Pushback benchmark/application expansion: 多 PDE、多 seed 和非规则采样只服务 method evidence, 不发布 benchmark 或 application artifact.
- Pilot:
  - Setup: 本机 RTX 4060 Ti + CuPy, v4 proxy suite. Uniform grid 测 `single_wide_separated` mode `(4,17)` 和 `two_mode_wide_separated` modes `(4,17),(9,22)`; 对比 RHS-FFT top-k、NCPSD 95% band、soft radius sweep 0-12、dense grid、16 seed random no-prior、4096 个非均匀样本 RHS spectral scan 和 frequency-error sweep 0-3.
  - Metric: proxy 正信号要求 RHS-only estimators 覆盖真频率, 候选集合明显小于 dense grid, estimator overhead 计入后仍有 net proxy gain, random no-prior median relative L2 >0.5, 并且小频率误差下 soft prior 能恢复 hard prior 的失败.
  - Result: CuPy wall `4.99s`. Single mode: RHS-FFT top-k 1/576 candidates relL2 0, net proxy speedup 3.98x; NCPSD 68/576 relL2 9.97e-15, speedup 5.20x. Two mode: RHS-FFT 2/576 relL2 7.97e-17, speedup 10.67x; NCPSD 198/576 relL2 9.42e-15, speedup 2.45x. Irregular samples: spectral scan top-1 relL2 3.47e-16 and 50.8x proxy speedup over dense sampled dictionary. Radius-12 soft prior already covers 320/576 single-mode and 395/576 two-mode candidates, so unbounded widening would erase the method. Error sweep: +1 offset is fixed by radius 1, +2 needs radius 2, +3 is not fixed by radius 2.
  - Signal: POSITIVE but bounded; it supports observable frequency-candidate compilation, overhead sanity, and a prior-width cap, not yet real FEX controller improvement.

- Claims and Claims matrix: Main claim: in RL symbolic PDE solving, frequency search can be isolated as a measurable bottleneck, and observable spectra can compile that bottleneck into a bounded soft candidate distribution. POSITIVE: oracle-soft and estimated-soft both improve true Multi-Scale FEX by >50% candidate evaluations or net wall-clock on verified wide-frequency PDE, with selected prior <=25% of dense grid. NULL-A: oracle helps but estimated prior fails, so the compiler interface is valid but current estimators are weak. NULL-B: oracle fails, so frequency search is not the bottleneck and the result becomes a negative diagnostic. DEGENERATE: estimated prior needs >35% of dense grid, so the spectrum is observable but not useful for search-space reduction. NEGATIVE-CONTROL: hard freeze fails under small frequency error while soft prior works, supporting only the soft/adaptive injection choice.
- Narrative: The paper should be framed as "Frequency-Search Bottleneck Diagnosis for RL Symbolic PDE Solvers, with Spectral Compilation as Repair." This is narrower than adding Fourier features: the unit of intervention is the controller's discrete frequency choices, and the repair is allowed only after the oracle diagnostic says that frequency search is the bottleneck.
- Experiments: Gate 0 true-FEX oracle falsifier: standard Multi-Scale FEX, oracle hard, oracle soft, dense frequency grid and initialization-only, with 3-5 seeds, equal candidate/wall-clock budgets and frequency-resolved errors. Gate 1 injection ablation: candidate pruning, soft sampling prior, reward bonus and initialization-only under the same budget. Gate 2 estimator selector: RHS FFT, early-residual DFT, NCPSD, CWT/SST ridge and local-window variants, reporting estimator overhead, prior width and final FEX error. Gate 3 robustness: two-mode/harmonic cases, nonlinear RHS where solution and residual spectra differ, local oscillations, nonuniform samples, nonrectangular masks and frequency perturbation; report when the prior crosses the 35% degeneration line.
- Assets status: v4 CuPy proxy and handoff notes are ready locally; data/model status and rerun instructions are in `workspace/fex-synchro-prior/data/MANIFEST.md`.

<review date="2026-06-17">

## Novelty

- Score: 7/10
- Closest prior work: Multi-Scale FEX (2510.22497, Hardwick & Yang 2025); Spectral-Prior Guided MSPINNs (2508.17902, Li et al. 2025); Domain-Aware Symbolic Priors for SR (2503.09592, Huang & Yang et al. 2025); PRISMA (2512.01370, Sawhney et al. 2025)
- Key differentiator: 谱估计本身不新, RHS/residual spectrum → PINN 特征增强已有 MSPINN/RUNNs/IFeF-PINN/PRISMA/FRES 多条工作线。本 idea 的核心差异化是把可观测频谱信号编译成 RL expression-tree controller 的软频率动作先验, 从而缩小离散频率搜索空间——所有已有谱先验工作注入的是连续神经优化 (初始化/注意力/特征), 无一涉及离散 RL 表达式树搜索空间缩减。Yang 组自己的 2503.09592 用了 domain-aware symbol priors, 但来自领域语料库统计, 不是 PDE 频谱估计。本 reviewer 独立执行了 6 组 targeted web/S2 search (2026-06-17), 覆盖 "frequency prior RL expression tree", "spectral compilation symbolic PDE search", "FFT symbolic regression RL search space" 等 query, 未发现任何已有工作。若方法不新, 可能新的是 finding: FEX 的 wide-frequency failure 是否可被频率搜索瓶颈因果解释——该诊断实验本身在文献中完全空白。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 7/10 | v4 的 "诊断→修复" 因果链清晰: oracle gate 隔离频率搜索瓶颈 → estimator selector 编译先验 → robustness sweep 标定退化边界。剩余 gap: (a) 真实 FEX oracle 实验仍是缺失的——proxy dictionary lookup 的确定性成功不等于 RL controller 受频率动作拖累, 两者在 Adam/BFGS 系数优化、reward 噪声和 expression-tree credit assignment 上存在质的差异; (b) manufactured oscillatory PDE 的 RHS 可能直接暴露解频率, 若正结果只来自这种设置, 审稿人会认为"把答案从 RHS FFT 读出来"而非解决了 FEX 搜索问题 (codex 提出的新 concern, 有价值); (c) 频率搜索、表达式结构搜索和连续系数优化三者可能非线性耦合, oracle gate 无法完全 disentangle。 |
| Missing evidence signals | 7/10 | v4 pilot 相比 v3 新增 radius sweep 0-12、net proxy speedup 核算、prior-width 实测 (single-mode radius-12 已覆盖 320/576 candidates, 占 55.6%→超过 25% 线) 和频率误差 sweep 0-3, 代理证据质量显著提升。但决定性证据仍缺失: (a) 真实 Multi-Scale FEX 上 standard vs oracle-soft vs estimated-soft 的 3-5 seed 搜索曲线 (equal candidate/wall-clock budget, frequency-resolved error); (b) 非平凡非线性/局部振荡/非矩形域 case 上 RHS/residual spectrum 不退化; (c) CWT/SST ridge 在 manufactured oscillatory PDE 上的实测精度和 overhead。Gate 0 仍然是所有 method claim 的前提条件, 当前未执行。 |
| Narrative | 9/10 | "Frequency-Search Bottleneck Diagnosis for RL Symbolic PDE Solvers, with Spectral Compilation as Repair" 是当前最锐利的 framing。因果先行 (先诊断再修复) 的叙事结构远超 v1/v2 的 "加一个预处理模块", 与 PINN spectral-bias 文献的区隔清晰有力。diagnostic 作为 supporting contribution type 的显式化使论文即使 estimator 部分失败也有独立发表价值。轻微的修辞风险: "spectral compiler" 隐喻可能让审稿人期待端到端自动编译 pipeline, 而实际机制更接近 "frequency candidate distribution as prior"。 |
| Venue contribution | 7/10 | Topic 未声明 target-venue, 按 FEX/SciML/ML 顶会标准推断。若真实 FEX oracle 成功且 estimated prior 在非平凡 PDE 上接近 oracle: 强 NeurIPS/ICLR candidate (method+diagnostic 双贡献)。若仅 oracle 成功但 estimated prior 弱: 可作 diagnostic paper 投稿 JMLR/JCP 或 SciML 专题。若 oracle 失败: 应归档或转向连续参数耦合/reward 诊断。当前贡献的顶会分量完全取决于 Gate 0 的结果——proxy evidence 不足以支撑顶会。 |
| Testability | 10/10 | Expected outcome 包含明确的最便宜反证: "oracle-soft FEX 不优于标准 FEX" 直接否定"频率搜索是主瓶颈"的中心因果解释, 仅需 2-3 天 compute。POSITIVE/NULL-A/NULL-B/DEGENERATE/NEGATIVE-CONTROL 五态 claims 界限清晰且可预注册。`25%/35%` prior-width rule 是出色的退化边界设计——防止"先验变 dense grid"后仍硬 claim 搜索空间缩小。Gate 0→1→2→3 的级联实验设计逻辑严密。 |
| Outcome realism | 8/10 | >50% candidate evaluation 降低在 d<=3、well-separated、频率可观测 PDE 上是现实的——如果频率搜索确为主瓶颈。net wall-clock >50% 更难, 因为 Adam/BFGS 系数优化可能在真实 FEX 中主导 wall-clock, 压缩 controller-side gain; reward 噪声和 multi-seed RL 搜索方差也会增加不确定性。v4 对 outcome 的限定合理: (a) 条件限定于 d<=3、well-separated、observable; (b) `25%/35%` prior-width rule 防止 overclaim; (c) estimator overhead 计入 net wall-clock; (d) DEGENERATE outcome 处理"频谱可估计但不可用"的中间情况。 |
| Contribution type compliance | n.a. | idea types = {method, diagnostic} ⊆ preferred-contribution-types: n.a. (topic 0616-fex.md 未声明 `preferred-contribution-types` 字段, 跳过检查) |
| Overall Quality | 8/10 | v4 相比 v3 在叙事锐度、prior-width 退化边界和 pilot evidence 覆盖面上有明显改善, 解决了 v3 review 提出的多项 concern。但决定性 evidence gap (真实 FEX oracle) 未变——这是当前 blocking item, 不是设计缺陷。Codex 在个别维度评分低 1 分 (更保守), 但总体判断高度一致。 |

## Contribution Drift (n=4)

- v_3 contribution types: {method}
- v_4 contribution types: {method, diagnostic}
- Status: expanded(+diagnostic)
- Hard cap triggered: no (topic 未声明 preferred-contribution-types; refiner 在 "Contribution drift note" 中明确写了扩张理由: "reviewer 要求把 frequency-search bottleneck diagnosis 从 gate 提升为主要证据", 理由成立)

v3 review 每条 concern 的当前状态:
- "proxy 在手工 sine dictionary 上完成, 非真实 FEX controller" → partially resolved (v4 仍将 real-FEX oracle 设为 Gate 0, 但未执行; proxy 证据质量提升但本质 gap 不变)
- "RHS 频谱与解频谱在非线性算子下可能不一致" → partially resolved (v4 Gate 3 加入 "nonlinear RHS where solution and residual spectra differ" 测试, 但未给出具体缓解策略; codex 额外指出 manufactured PDE 的 RHS 可能直接泄漏答案)
- "soft prior 过宽退化为 dense grid" → resolved (v4 加入 `25%/35%` prior-width rule, radius sweep 0-12, 实测 single-mode radius-12 覆盖 320/576=55.6%→已超 25% 线→证实在 unbounded widening 下退化)
- "真实 Multi-Scale FEX benchmark 上跑 oracle" → planned (Gate 0), 尚未执行
- "multi-seed RL 搜索曲线" → planned (Gate 0), 尚未执行
- "频率估计开销纳入 wall-clock" → resolved (v4 明确要求 "estimator overhead 计入后仍成立", pilot 已报 net proxy speedup)
- "estimator 对比 (NCPSD/synchrosqueezing)" → resolved (v4 Gate 2 加入 CWT/SST ridge 和 local-window variants)
- "非规则域/非均匀采样鲁棒性" → resolved (v3 已加入, v4 维持并加入 nonrectangular masks)
- "频率误差相关性分析" → resolved (v3 已加入 frequency error sweep 0-3, v4 维持)
- "PINN prior conflict" pushback → 接受 (refiner 正确指出 MSPINN/IFeF/PRISMA 覆盖连续优化, 未缩小 RL 表达式树离散频率搜索空间; codex 也认可此 pushback 有据)
- "oracle gate 应提升为主要贡献" → resolved (v4 加入 supporting diagnostic type, 叙事升级为 diagnosis-first)
- v3 Alternative Framing 建议 "Frequency-Search Bottleneck Diagnosis... with Spectral Compilation as Repair" → resolved (v4 全文采纳)

Refiner pushback 评估:
- Benchmark/application expansion pushback: 有据, 接受。多 PDE、多 seed 和非规则采样只服务 method evidence, 不发布 benchmark artifact。
- PINN/spectral-prior conflict pushback: 有据, 接受。已有工作修连续优化, 不修离散 RL 频率搜索。

## Alternative Framing

NONE. v4 已经采用最锐利的 framing: "Frequency-Search Bottleneck Diagnosis for RL Symbolic PDE Solvers, with Spectral Compilation as Repair." 进一步锐化的空间有限; 若一定要挑, 可以把 "Spectral Compilation as Repair" 改成 "Spectral Prior as Search-Space Compiler" 以更精确地传达机制 (prior → candidate distribution → smaller search space) 而非暗示端到端编译。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在 d<=3、频率可观测、well-separated 的振荡 PDE 上, oracle-soft 和 estimated-soft spectral prior 使真实 Multi-Scale FEX 以 <50% candidate evaluations 或 net wall-clock 达到同等 relative L2, 且 selected prior 覆盖 <=25% dense frequency grid, estimator overhead 计入后 net gain 仍为正。 |
| NULL-A | oracle 有效但 estimated prior 无效或需要 >35% dense grid: 编译器接口 (frequency prior → controller) 范式成立, 但当前 estimator 是瓶颈 (NULL-A) 或频谱可估计但不可用于搜索空间缩减 (DEGENERATE)。后续方向为更强估计器或替代瓶颈假说。 |
| NULL-B | oracle-soft 不优于标准 Multi-Scale FEX: 频率搜索不是 FEX 的主要瓶颈, 论文转为负诊断——定位连续参数耦合、residual reward 失真或 expression-tree credit assignment 为真正根因, 不再声称谱先验能修复 FEX。 |
| NEGATIVE-CONTROL | 硬冻结在微小频率误差 (+1) 下失败而 soft prior (radius>=1) 恢复: 仅声称 soft/adaptive 注入是必要设计选择, 不声称谱先验整体解决 FEX 频率问题。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale:
  **Likelihood = Medium**: 成稿路径清晰 (Gate 0 oracle → Gate 1 injection → Gate 2 estimator → Gate 3 robustness), proxy pilot 给出 POSITIVE but bounded 信号, v4 的 falsifier 和退化线设计扎实。但强依赖真实 FEX oracle 成功——proxy dictionary lookup 的确定性成功不等价于 RL controller 受频率动作拖累, Adam/BFGS 系数优化和 reward 噪声可能在真实 FEX 中压缩 oracle gain。此外, manufactured PDE 的 RHS 可能直接暴露解频率, 若正结果只来自这种设置则审稿人会质疑 circularity。估计 oracle gate 通过概率 ~50-60%。Claude 与 Codex 对 Likelihood 判断一致为 Medium, 无分歧。
  **Impact = High**: 若成功, 将 (a) 确立"将经典频谱分析编译为 RL 符号搜索先验"这一此前缺失的设计范式; (b) 直接定位并修复 Multi-Scale FEX 论文明确承认的 wide-frequency-separation 失效模式; (c) 证明 Yang 两条独立学术脉络 (FEX RL search + SynLab spectral estimation) 的合成价值。影响集中在 FEX/RL-symbolic-PDE 研究线, 不会改变全 SciML/ML 领域判断, 但在此明确研究线上形成强 top-venue 结果。Claude 与 Codex 对 Impact 判断一致为 High, 无分歧。

## Overall

- Priority: High
- Score: 7
- Comments: v4 是一次 disciplined improvement: 没有 silent downgrade, 没有越界 benchmark/application 扩张, diagnostic contribution 的显式化有明确 reviewer feedback 依据且理由成立。v3 review 的大部分 concern 已 resolved 或 partially resolved, 剩余的是需要真实 FEX 实验才能关闭的 evidence gap。当前最紧迫的下一步仍然是 Gate 0: 用真实 Multi-Scale FEX codebase 跑 oracle-frequency vs standard FEX 的 3-5 seed 对比 (预计 2-3 天 compute)。若 oracle 通过, 本 idea 具有 High (7) 的顶会潜力; 若 oracle 失败, 应及时归档或转向连续参数耦合/reward 诊断。Claude 与 Codex 在所有关键维度 (Likelihood/Impact/Priority/Score) 上判断一致, 仅在个别 Quality 子维度存在 1 分差异 (Codex 更保守), 不构成实质性分歧。Codex 额外提出的 "manufactured PDE RHS 可能泄漏答案" concern 值得重视, 建议在 Gate 3 robustness 中显式加入 RHS-blind test (例如用 white-noise RHS 或仅从 early-residual 估计频率)。

</review>
