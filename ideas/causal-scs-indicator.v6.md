<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 把滑窗 conditioner sensitivity 收紧为跨估计器可识别性/失效边界审计：在同一扰动网格中判断 fixed-edge partial correlation、PCMCI+ 与 LKIF 的条件化分歧何时可解释为机制信号，何时只是 estimator artifact；真实强对流仅作 held-out 证伪。
- Hypothesis: 这里的 conditioner sensitivity 指同一目标边改用 confounder proxy 或等维 sham 条件集后，D/J 诊断的差。只有非退化空对照校准通过，而且差值符号跨混合扰动、部分可观测度和估计器保持，它才可能携带机制信息。现有 fixed-edge 结果在 graph-only 角点为正，却在更现实的 mixed+obs=0.5 格消失；LKIF smoke 也不一致。因此主假设是“存在可测的适用边界”，不是“条件化一般有益”。该方向与 anchor regression/minimax 的扰动位置边界仅作定性类比和迁移检验；尚无数学桥接或 Theta(t^2) 定量证据，不能称为理论预测。
- Expected outcome: 支持信号要求修复后的 nominal-null gate 通过，且全规模 PCMCI+/LKIF 在预注册格上给出可重复的同向或稳定异向结果；最便宜的反证是任一非退化空对照失准。当前 v6 null audit 四个主 gate 均因过覆盖失败，故跨估计器与大气 claim 继续冻结。
- Contribution type: empirical-finding+diagnostic
- Contribution drift note: v5 为 `empirical-finding+diagnostic`；v6 两类全部保留，不新增 method/theory/application/benchmark。收紧 headline 是因为现有证据只支持诊断审计，并非删除 empirical finding。
- Risk: HIGH
- Estimated effort:
  - Compute: v6 实测 58.16s L4；修复 null 后的全规模 PCMCI+/LKIF+Causal-CCP 预计 <1 GPU-hour、5-15 CPU-hours；通过后 H2 约 10-30 GPU-hours、500-1500 CPU-hours。
  - Data: needs annotation；HRRR/SPC smoke assets 已有，H2 还需独立雷达/地面边界标注。
  - Implementation: inference 修复与三估计器审计 1-2 周；若过门，H2 标注和 held-out 检验 4-6 周。
- Novelty quick-check: Causal-CCP (2403.12677) 已在线性/offline/iid 下定义 invariance-based causal change point 与 stability loss，是必须正面对比的最近 baseline。Siddique et al. (2026, Conditioners-LKIF) 不只做静态 VAR；其主体已在真实 land-atmosphere 数据上研究 conditioner divergence 的时空/regime 依赖。这里尚未被覆盖的窄问题是：在滑窗、时间依赖、部分可观测且真实边/混杂可同时漂移时，跨估计器分歧能否被校准并定位失效边界。
- Strongest objection: 当前唯一全规模证据来自一个 AR skeleton 的 fixed-edge partial correlation；mixed+obs=0.5 不显著、LKIF 只有 32 pairs、null CI 过保守、raw EWS 全胜，完全可能只是 estimator/inference artifact。
- Why we should do this: 正负结果都能回答一个有用问题：何时 conditioner sensitivity 可解释，何时必须拒绝物理归因。它比在校准失败时继续堆真实案例更可证伪。
- Pilot:
  - Setup: v5 原始 JSON 实为 `phase x g{0,.4} x h{0,.5} x obs{1,.5}` 全 16 格（96 pairs、8 seeds、400 bootstrap），而非仅角点。v6 先以独立创新但同一 mixed 机制构造可交换 null，预注册 256 外层重复后在 L4 运行；commit `9b00984` 早于 job 36888825，结果 runtime 记录该 commit。
  - Metric: D 是跨窗口离散度，J 是相邻窗口跳变；表中 P/S/R 为 proxy/sham/raw combined AUC，DeltaD/DeltaJ/DeltaC 为 P-S，DeltaC 给 95% CI。v6 主 gate 要求 type-I Wilson 95% CI 含 0.05 且绝对均值偏差 <=0.02。
  - Result:

    | 相 | g | h | obs | P/S | DeltaD | DeltaJ | DeltaC [95% CI] | R |
    |---|---:|---:|---:|---|---:|---:|---|---:|
    | pre | 0 | 0 | 1 | .500/.500 | .000 | .000 | .000 [.000,.000] | .500 |
    | pre | 0 | 0 | .5 | .500/.500 | .000 | .000 | .000 [.000,.000] | .500 |
    | pre | 0 | .5 | 1 | .500/.510 | -.021 | .005 | -.010 [-.044,.027] | .987 |
    | pre | 0 | .5 | .5 | .498/.510 | -.021 | -.001 | -.011 [-.036,.012] | .987 |
    | pre | .4 | 0 | 1 | .728/.691 | .034 | .035 | .037 [.008,.069] | .882 |
    | pre | .4 | 0 | .5 | .719/.691 | .021 | .016 | .029 [.004,.047] | .882 |
    | pre | .4 | .5 | 1 | .663/.624 | .038 | .037 | .039 [.001,.073] | .998 |
    | pre | .4 | .5 | .5 | .644/.624 | .022 | .009 | .020 [-.009,.046] | .998 |
    | post | 0 | 0 | 1 | .500/.500 | .000 | .000 | .000 [.000,.000] | .500 |
    | post | 0 | 0 | .5 | .500/.500 | .000 | .000 | .000 [.000,.000] | .500 |
    | post | 0 | .5 | 1 | .502/.504 | -.004 | -.004 | -.002 [-.035,.030] | .985 |
    | post | 0 | .5 | .5 | .474/.504 | -.026 | -.028 | -.030 [-.053,-.007] | .985 |
    | post | .4 | 0 | 1 | .673/.619 | .056 | .036 | .054 [.031,.082] | .903 |
    | post | .4 | 0 | .5 | .653/.619 | .028 | .029 | .035 [.014,.056] | .903 |
    | post | .4 | .5 | 1 | .596/.537 | .069 | .035 | .059 [.030,.089] | .996 |
    | post | .4 | .5 | .5 | .555/.537 | .024 | .008 | .017 [-.009,.043] | .996 |

    obs=.5 时实测 corr(w,z) 为 pre .705、post .778。mixed 格只在 obs=1 显著；obs=.5 两相均含 0。LKIF 四个非空格中 3/4 与 fixed-edge graph-only 收益方向相反；唯一同向格为 post/graph-only，true=.9331 > sham=.9301、marginal=.9303，32 pairs 下 CI 高度重叠，不能作方向显著性结论。v6 null 的 pre AUC/Delta coverage=.988/.984、post=.984/.992（type-I=.012/.016/.016/.008）；四个 Wilson 区间均低于 .05，均值偏差 <.002，说明 overcoverage 而非 type-I 膨胀。
  - Signal: NEGATIVE；nominal 95% inference 未校准，跨估计器一般性和真实大气价值均未成立。

- Claims and Claims matrix:
  - C1: fixed-edge graph-only 收益在两相和两档 obs 下为正；mixed 收益在 obs=.5 不稳健，不得写成“加固后全面存活”。
  - C2: LKIF 是 3 格反向、1 格同向且 underpowered；PCMCI+/LKIF 全规模前只能称 estimator specificity 未决。
  - C3: 非退化 null 显示当前区间过保守；显著格可能保守，但不能称 nominal-coverage validated。
  - C4: raw EWS 在全部非空格占优；没有真实 SCS、DAG 恢复或独立预警 claim。

  | Outcome | 允许的 claim |
  |---|---|
  | CALIBRATED+AGREE | 可报告跨估计器的扰动来源边界，仍不外推到天气。 |
  | CALIBRATED+DISAGREE | 核心发现是 estimator-specific failure map。 |
  | UNCALIBRATED | 仅报告 instrument audit，暂停效应显著性与 H2。当前属于此格。 |
  | REAL-H2 POSITIVE | 仅在 held-out 增量 CI >0 时称边界诊断有增量。 |
- Narrative: 采纳更聚焦的 framing，因为完整网格、LKIF 和 null 三项证据都把“估计器是否可信”置于“天气上发现什么”之前。真实数据降为 held-out falsifier。H1 按 storm mode 分层有下游选择/circularity 风险，降为探索项；除非另有独立冷池/大尺度强迫标签，不作机制归因。
- Experiments: P0（本轮完成）非退化 null 失败，先诊断两层 percentile bootstrap 的 overcoverage并另行预注册重跑。P1（must-run）在同一 16 格以 96 pairs/8 seeds/full bootstrap 跑 PCMCI+、LKIF，并用官方 Causal-CCP：`Y_t` 对 `(x_{t-1},y_{t-1},w_{t-1})`，`s/alpha` 只由 calibration controls 选，比较 AUROC、type-I 与变点定位误差；同时冻结 window/stride/J-quantile 敏感性和跨格 multiplicity 规则。P2（推迟，因当前只有定性类比）在满足 linear-SCM 假设的独立 slice 连续扫 t，以 `Delta(t)-Delta(0)` 为 estimand，预注册 quadratic 对 linear 的 held-out 误差和 log-log slope 是否含 2；不在 nonlinear v5 格上强套 Theta(t^2)。P3 仅在 P0-P2 过门后运行 H2：定义 `S_DJ=(D_proxy-D_sham,J_proxy-J_sham)`；由两名不看 HRRR predictor 的标注者用雷达 fine-line/地面分析独立画 outflow/dryline 边界；主 estimand 为 held-out storm systems、60-min lead、6-km tolerance 下 `DeltaF1 = F1(|grad S_DJ|)-max(F1湿度/θe梯度, F1风向突变/辐合, F1开发集拟合的原始场融合)`，阈值只在开发风暴选，按风暴 cluster bootstrap，95% CI 下界 >0 才算增量；0/30/90-min 和定位距离为次要结果。H1 只在独立机制标签可得时探索。
- Assets status: v6 null 结果与 smoke 数据已就绪，完整天气序列仍由失败 gate 暂停；单一交接源见 `workspace/causal-scs-indicator/data/MANIFEST.md`。

<review date="2026-07-12">

## Novelty

- Score: 4/10 (Claude 4, codex 4,无分歧)
- Closest prior work:
  1. **Huang, Peters, Pfister (2024), Causal-CCP** (arXiv:2403.12677) — 仍是最直接的 baseline，线性/offline/iid 下已占据"因果变点/stability loss"这一位置。
  2. **Rothenhäusler et al., Anchor regression** (1801.06229) + **有限样本 minimax** (2606.12680) — population-level 决策论边界，本 idea 是其向滑窗时序/部分可观测混杂场景的迁移检验，非新边界。
  3. **Siddique et al. (2026, submitted ESD), Conditioners-LKIF** — 最近的方法论近邻；v6 的 Novelty quick-check 措辞已修正 (见下方专项核查 1)，不再把该文误描述为"只有静态 VAR"。
- Key differentiator: 方法本身不新；尚未被直接覆盖的窄组合是"滑窗时间依赖 + 真实边与混杂同时漂移 + 部分可观测"条件下，对 conditioner-vs-sham sensitivity 做跨估计器校准。只有"校准后仍出现可重复、可解释的 estimator-specific failure boundary"这一 finding 可能新颖 (landscape 第 R 组 VCDF 作者自己承认从未测试过这一问题)；仅把 PCMCI+/LKIF 套到新网格上不构成顶会创新。Causal-CCP 仍未被正面对比，novelty 论证在此之前不完整。

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 未声明 `target-venue`/`preferred-contribution-types` (frontmatter 与正文均确认为空)，沿用 v1-v5 推断，按顶级 Earth-system methods / AI4Science / 因果推断方法学视角评估 (Claude 与 codex 一致)。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 (Claude 5, codex 5, 一致) | v5 遗留的三处报告精度问题 (16 格漏报、LKIF"4/4 反向"、anchor-regression"理论预测") 经 Claude 独立重算 `results/stress_pilot_results.json` 与 `results/lkif_smoke_results.json` 全部数字后确认已如实修正 (见下方专项核查)。但新版headline"跨估计器可识别性"仍用词过强 (codex 独立提出, Claude 认同): partial correlation、PCMCI+、LKIF 三者的目标泛函本不相同, 跨估计器"同号"没有理论上必然共享的因果含义, 稳定"异号"也不能单独判定谁是 artifact——这是比 v5 更深一层的措辞问题, 而不只是数据披露问题。Claude 独立发现一处 codex 未提及的补充逻辑缺口: 本轮 null audit 只在最难的混合格 (g=0.4,h=0.5,obs=0.5) 上校准, 而 stress grid 里真正撑起"正结果"的 graph-only-max 格 (g=0.4,h=0,obs∈{1,.5}) 从未被独立做过 null 校准; 过覆盖在高难度混合格成立不能自动推广到信噪比完全不同的 graph-only 格, 该格的显著区间究竟是保守还是反而 anti-conservative 仍是未知数。 |
| Missing evidence signals | 3/10 (Claude 3, codex 3, 一致) | Concern-by-concern 见下表。非退化 null "缺实验"这一 v4/v5 遗留项本轮真正执行且失败, 制造了新的推断有效性 blocker (先修 bootstrap 再谈全规模); H2 标注协议、estimand、raw-gradient baseline 从"无设计"变为"有完整协议"但仍零数据零功效分析; 全规模 PCMCI+/LKIF、Causal-CCP、window/stride/J-quantile 敏感性、multiplicity 校正仍是 ignored/deferred (虽然本轮首次给出了可执行的协议文字, 但零执行); local-independence DGP 仍未触及; raw EWS 连续第 6 轮全面占优仍未缓解。上述"先修校准再做 P1-P3"的推迟顺序有据, 可接受, 但不等于证据缺口已缩小。 |
| Narrative | 6/10 (Claude 6, codex 6, 一致, 较 v5 5/10 上调) | framing 明显更诚实、更集中: 完整披露此前被掩盖的混合格, 采纳了 v5 review 自己建议的"失效边界审计"reframe, 把真实天气正式降级为 held-out falsifier, H1 因下游选择偏差风险被明确降级为探索项。但"identifiability"这一头衔词、跨估计器叙事、Theta(t^2) 类比和 H2 仍构成多条并行支线, 且"正负结果都有用"不等于任意负结果 (如本轮的过覆盖) 自动具备顶会分量。 |
| Venue contribution | 3/10 (Claude 3, codex 3, 一致, 较 v5 3/10 持平) | 仍是单一 AR skeleton 上的 fixed-edge partial-correlation 全规模结果 + 32-pair LKIF smoke + 零全规模 PCMCI+ + 一次失败的 nominal-CI calibration + 零真实强对流证据。null overcoverage 是有价值的 instrument audit, 但不足以构成顶会主贡献; 本轮未新增任何跨估计器或真实数据证据。 |
| Testability | 8/10 (Claude 8, codex 8, 一致, 较 v5 6/10 上调) | Expected outcome 给出的最便宜 falsifier (非退化空对照失准) 本轮被真实执行且确实触发 FAIL, 这是纪律层面的实质加分; 256 次外层重复、四个主 gate、偏差阈值、claims freeze 规则全部预注册且被 Claude 独立用原始 JSON 复算验证无误; v4 式"预注册先于结果"证据链本轮也已恢复 (见下方专项核查 4)。扣分来自 P1 之后的 sensitivity/multiplicity 仍缺完整决策表。 |
| Outcome realism | 4/10 (Claude 4, codex 5, 差距在正常评审波动范围内, 不构成 Likelihood/Impact 意义上的分歧) | raw EWS 连续 6 轮 (v2/v3/v3.5/v4/v5/v6) 全面占优, 无一例外; 顶会级成功需要依次修复覆盖率、取得跨 DGP/估计器可重复边界、超越 Causal-CCP 等基线、并在 held-out storm systems 上相对既有梯度方法产生真实增量, 条件连乘后成功概率偏低。codex 的 5/10 认为"当前只能报告未校准 instrument audit"这一诚实表态本身具有现实性加分, Claude 认为这一诚实表态不能抵消条件连乘带来的整体折扣, 两者差距小, 不影响下方 Likelihood 判断。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致)。 |
| Overall Quality | 5/10 (Claude 29/6≈4.83 取整 5; codex 30/6=5.0; 一致, 较 v5 4/10 上调) | 报告纪律、可证伪性执行、H2 操作化三方面确有真实改善, 使 Overall Quality 相对 v5 回升一档; 但核心科学证据仍未跨出既有单一 synthetic fixed-edge 结果, 且本轮新增的"identifiability"措辞过强问题部分抵消了纪律层面的加分。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} (v5) contribution types: {empirical-finding, diagnostic}
- v_n (v6) contribution types: {empirical-finding, diagnostic}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`；Contribution drift note 与 REFINEMENT_LOG.md 均确认无新增/删除类型；Claude 与 codex 独立核验一致)

v5 `<review>` concern-by-concern 复核 (Claude 与 codex 独立复核后合并):

| v5 concern / 专项核查项 | Status | Assessment |
|---|---|---|
| 混合扰动格未被披露 (16 格中隐藏了最不利的一格) | resolved | Claude 独立用 Python 重算 `results/stress_pilot_results.json` 全部 16 行, 与 idea 正文 Pilot Result 表逐格核对完全一致 (含此前被掩盖的 g=0.4/h=0.5/obs=0.5 两格, dC 分别为 0.020[-0.009,0.046] 和 0.017[-0.009,0.043], 均含零)。全量披露且措辞诚实。 |
| LKIF "全部四格方向相反" 过度概括 | resolved | Claude 独立读取 `results/lkif_smoke_results.json` 确认: post/graph_only_max 格 true=0.9331>sham=0.9301>marginal=0.9303 (与被掩盖格完全吻合), 其余 3 格 true<sham/marginal。v6 正文与 `EXPERIMENT_LOG.md` 均已改写为"3/4 反向、1/4 同向 (post/graph-only-max)", 与原始数据精确匹配。 |
| anchor-regression"已被预测"措辞偏强 | resolved | v6 Hypothesis 段已改为"该方向与 anchor regression/minimax 的扰动位置边界仅作定性类比和迁移检验；尚无数学桥接或 Theta(t^2) 定量证据，不能称为理论预测"，措辞降级到位。 |
| 预注册时间证据链断裂 (v5 计划/代码/结果同一提交, 无法独立核验先后) | resolved | Claude 独立 `git log` 核验: 预注册提交 `9b00984` (2026-07-12 01:06:13) 早于结果提交 `d56c213` (01:12:11), `results/null_audit_results.json` 的 `runtime.git_commit` 精确记录 `9b0098410af680cea285e303524f4ec6aeb2b47f`, 与 v4 式独立可核验证据链一致恢复。 |
| 非退化可交换空对照 type-I/coverage (v3→v4→v5 连续 3 轮 ignored) | 执行但 FAIL, 转化为新 blocker | 本轮真正预注册并运行 (256 重复, `PILOT_PLAN_V6.md`)。Claude 独立重算 `results/null_audit_results.json`: 四个主 gate (pre/post × AUC/Delta) 的 Wilson 95% 区间分别为 [.004,.034]/[.006,.039]/[.006,.039]/[.002,.028], 均值偏差 <.002, 均不含名义 0.05 且均在其下方——精确对应"over-coverage 而非 type-I 膨胀"的正文表述, 无夸大或淡化。这是本轮唯一真正执行的新实验, 但结果是 FAIL, 未推进跨估计器/真实数据证据本身。 |
| Causal-CCP 正面 baseline (v4→v5 连续 2 轮 ignored) | partially resolved | 本轮首次写出可执行协议 (`Y_t` 对 `(x_{t-1},y_{t-1},w_{t-1})`, s/alpha 仅由 calibration controls 选, 比较 AUROC/type-I/变点定位误差), 列为 P1 must-run, 但零执行。 |
| 连续扰动强度/Theta(t^2) 标度律检验 (v4 review 提出, v5 ignored) | partially resolved, 有据推迟 | P2 给出协议 (`Delta(t)-Delta(0)` estimand, quadratic-vs-linear held-out 误差与 log-log slope), 并给出明确理由推迟 (需满足 linear-SCM 假设的独立 slice, 不能强套在非线性 v5 网格上)——有据, 接受推迟。 |
| J 的 window/stride/quantile 敏感性 (v2→v5 连续 4 轮 ignored) | ignored/deferred | 仍未执行, 仅在 P1 中承诺"冻结 window/stride/J-quantile 敏感性"规则, 无新证据。 |
| bootstrap 分辨率与 multiplicity 校正 (v2→v5 连续 4 轮 ignored) | ignored/deferred | 仅在 P1 中承诺"跨格 multiplicity 规则", 无新证据。 |
| H1 storm-mode 分层的下游选择/circularity 风险 (v5 review 专项核查指出) | resolved | H1 已明确降级为"只在独立机制标签可得时探索", 不再作为主线科学假设, 直接回应了 circularity 批评。 |
| H2 相对既有边界探测方法的增量基线缺失 (v5 review 专项核查指出) | resolved (协议层面) | P3 完整协议已写入: 独立标注者盲于 HRRR predictor、held-out storm systems、60-min lead、6-km tolerance、`DeltaF1` 相对既有湿度/θe梯度和风场突变基线的增量要求、阈值仅在开发集选、风暴 cluster bootstrap、95% CI 下界>0 才算增量。协议扎实可执行, 但零数据零执行, 不能计入证据层面的 resolved。 |
| raw EWS 全面占优, 需真实数据排序反转 (连续 5 轮诚实保留) | unresolved, 诚实承认 | v6 continues to report 0.88-1.00 dominance, 无缓解证据, 第 6 轮仍无一例外。 |
| Contribution scope 保持 empirical-finding+diagnostic | resolved | 无新增/删除类型, 正文 drift note 与 REFINEMENT_LOG.md 均属实。 |

未采纳建议的 pushback 复核 (Claude 与 codex 一致): "P0 (null 修复优先) 先于 P1-P3 全规模执行"这一顺序有据, 接受; "P2 Theta(t^2) 推迟到满足 linear-SCM 假设的独立 slice"同样有据, 接受。但 window/stride/J-quantile 敏感性和 bootstrap multiplicity 校正连续 4 轮只以"计划中"形式存在而无任何试点执行 (即便是低成本的敏感性扫描), idea md 未给出不能低成本试跑的理由, 本轮继续重新强调为 must-fix。

## 专项核查 (dispatcher 本轮指定)

**1. 16 格未全披露 (v5 review 问题) 是否已如实改正**: **是, 完全改正**。Claude 独立用 Python 重新读取并计算 `results/stress_pilot_results.json` 的全部 16 个 cell (`phase×graph_strength×confound_shift×confound_observability` 完整 2x2x2x2 交叉), 逐格核对 P/S/DeltaD/DeltaJ/DeltaC/CI/R 七列数字与 idea 正文 Pilot Result 表, 完全一致, 无一处四舍五入外的偏差。此前被 v5 掩盖的"混合扰动"格 (g=0.4,h=0.5) 在 obs=0.5 时两个 phase 的 DeltaC CI 均含零 (pre: 0.020[-0.009,0.046]; post: 0.017[-0.009,0.043]), 现已在正文与 EXPERIMENT_LOG.md 中明确写出。

**2. LKIF 方向被误报为 "4/4" 实为 "3/4" 是否已改正**: **是, 完全改正**。Claude 独立读取 `results/lkif_smoke_results.json` 的 4 个非空 cell: pre/graph_only_max (true=.9423<sham=.9503<marginal=.9641)、pre/confound_only_max (true=.8889<sham=.9373)、post/graph_only_max (true=.9331>sham=.9301, marginal=.9303 ——同向)、post/confound_only_max (true=.8423<sham=.9129)。确认恰好 3 反向 1 同向, 且同向的一格 (post/graph-only-max) 恰是 fixed-edge partial-correlation 家族里效应量最大的一格 (Δ=0.054)——v6 正文与 codex 独立核验均准确复述这一细节, 未淡化也未夸大。

**3. anchor regression/minimax 措辞是否已从"预测"降级为恰当类比**: **是**。v6 Hypothesis 段明确写"仅作定性类比和迁移检验；尚无数学桥接或 Theta(t^2) 定量证据，不能称为理论预测", 与 Novelty quick-check 段口径一致, 无过度声称残留。

**4. 非退化空对照 null audit 的 NEGATIVE 结果是否被准确呈现 ("全部因 over-coverage 而非 type-I 膨胀失败")**: **是, 且这是本轮诚实纪律的亮点**。Claude 独立重算 `results/null_audit_results.json` 的 `primary_gate`/`summaries` 字段: 四个主 estimand (pre-AUC, pre-Delta, post-AUC, post-Delta) 的 type-I rate 分别为 .012/.016/.016/.008, Wilson 95% 上界分别为 .034/.039/.039/.028, 全部低于名义 0.05——这精确对应"过覆盖" (真实 type-I 率显著低于名义值), 而非"膨胀" (若是膨胀, Wilson 下界应超过 .05)。均值偏差 (.001/.0009/.0013/.0016) 全部 <.002, 与正文一致。诚实且精确, 未发现夸大或淡化。**但 Claude 独立发现一处该校验未覆盖的残留问题** (codex 未单独提出): 该 null audit 只在 stress grid 里"最难"的混合格 (g=0.4,h=0.5,obs=0.5) 上运行, 而真正撑起"graph-only-max 收益显著"这一核心正结果的格 (g=0.4,h=0,obs∈{1,.5}) 从未独立做过 exchangeable-null 校准。混合格的过覆盖是否代表同一 bootstrap 机制在所有信噪比 regime 下都保守 (即 graph-only-max 格的显著区间只会更可信), 还是不同信噪比下校准方向可能相反 (即 graph-only-max 格反而 anti-conservative), idea 目前没有证据可以排除后者, 这是"先修复推断再做 P1"这一决策逻辑里一个未被讨论的假设。建议下一版明确, 或在同一 null-audit 框架下补做 graph-only-max 格的校准检验。

**5. H2 操作化协议 (雷达/地面独立标注+ΔF1 estimand+既有梯度方法基线) 是否足够扎实、可执行**: **基本扎实**。协议包含: (a) 双标注者独立标注、且明确"不看 HRRR predictor"以避免标签泄漏; (b) held-out storm systems 而非 held-out windows, 降低同一风暴内部泄漏风险; (c) 60-min lead + 6-km tolerance 给出具体、可复现的容差定义; (d) `DeltaF1` 相对三个具体既有基线 (湿度/θe梯度、风向突变/辐合、开发集拟合的原始场融合) 的增量要求, 直接回应 v5 review 专项核查 2 指出的"缺相对既有方法的增量基线"; (e) 阈值只在开发风暴选、按风暴 cluster bootstrap、95% CI 下界>0 才算增量, 三者合起来构成了合理的反循环论证 (anti-circularity) 设计。**尚未闭合的缺口**: 协议未给出双标注者一致性 (inter-annotator agreement) 的量化标准或最低门槛, 也未说明标注分歧时的仲裁规则; 未给出功效分析 (需要多少 held-out storm systems 才能在给定效应量下达到合理功效); "开发集拟合的原始场融合"基线本身的具体建模选择 (线性 vs 非线性融合、特征集) 也未固定, 存在事后调参空间。这些是协议成熟到可直接执行前仍需补的具体条目, 但不影响协议整体方向的合理性。

## Alternative Framing

Claude 与 codex 收敛到同一诊断, 但表述略有差异 (非 Likelihood/Impact 意义上的分歧, 是同一个建议的两种措辞): 当前"跨估计器可识别性"headline 里的"可识别性 (identifiability)"一词仍隐含"跨估计器方向一致=共同机制、跨估计器方向不一致=可以判定谁是 artifact"这一未经证明的假设, 而 partial correlation、PCMCI+、LKIF 三者的目标泛函本不相同。codex 建议的更锋利表述: 把 headline 进一步收紧为"conditioner-sensitivity 的校准压力测试与 estimator-specific failure map"——只报告各估计器**内部**同定义的 proxy-vs-sham contrast 是否校准、可重复, 不把跨估计器同号/异号直接解释为"是否存在共同机制"或"谁是 artifact"。这一收紧不需要引入 preferred-contribution-types 之外的类型 (topic 未声明该字段), 也更契合当前"零跨估计器全规模证据"的实际证据水平, 但在全规模证据出现前不会改变下方 Likelihood-Impact 的象限判断。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 当前状态是 UNCALIBRATED, 不能把 stress-grid 里显著的区间称为"已验证的显著效应"; 只能描述"graph-only-max 点估计为正、mixed+obs=.5 区间含零"这一既成事实。修复 bootstrap 校准并重新预注册复现后, 最多可声称该失效边界在所测的单一 AR-skeleton DGP、给定窗口/滞后设置、以及已测的扰动/可观测度范围内成立, 不得声称"跨估计器可识别性"已被建立 (三者目标泛函不同), 不得外推到一般因果机制或真实大气过程。 |
| NULL | mixed+obs=.5 格与 LKIF smoke 的重叠区间只能称为"未取得决定性证据", 不能称"效应为零"或"方向已被推翻"; 当前的过覆盖尤其放大了"未拒绝不等于无效应"这一风险, 需要在校准修复后重新评估, 不能直接沿用当前区间下结论。 |
| NEGATIVE | 可确凿声称: 该两层 percentile/hierarchical bootstrap 在本轮指定的 exchangeable mixed null 下明显过覆盖, 四个预注册主 gate 全部失败; raw EWS 在全部非空 stress cell 上占优 (连续 6 轮无例外); LKIF smoke 未能给出 phase-consistent 的 graph-only 收益复现 (3 反向 1 同向, underpowered)。不得据此判定"所有 conditioner sensitivity 均不存在"、"所有因果发现估计器均不可用"或对真实强对流应用价值下结论——这些都超出当前证据范围。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 一致判定 Low, 无分歧。理由: 校准 (最基础的推断有效性前提) 本轮才刚刚开始被认真检验, 且首次真正检验就失败, 这把"全规模跨估计器复现"往后又推了一步, 而不是往前推进了一步; 全规模 PCMCI+/LKIF、Causal-CCP 正面基线、window/stride/quantile 敏感性、multiplicity 校正、H1/H2 真实数据均仍是零执行; 连续 6 轮 raw EWS 全面占优, 无缓解迹象。这些条件需要依次全部成立才能到达 top-venue 级结果, Likelihood 维持 Low, 与 v2-v5 一致。
  - Impact: Claude 与 codex 一致判定 High, 无分歧。理由不变于 v4/v5: 若最乐观情形——跨 DGP、跨估计器 (partial correlation/PCMCI+/LKIF)、跨真实强对流数据都稳定复现一个可解释、可复现的"conditioner-sensitivity 何时可信"的失效边界, 且 H2 在严格 held-out 条件下相对既有边界探测方法产生真实增量——会为"非平稳滑窗场景下因果诊断何时可信"这条研究线提供一个此前从未被清晰刻画过的边界, 具备顶会发表价值; 但工具 (PCMCI+/LKIF/Causal-CCP) 与理论支撑 (anchor regression/minimax) 均已存在于相邻文献, 问题域相对窄, 不到 Exceptional。
  - 查表: Low x High = Medium (5)。topic 无 `preferred-contribution-types` 声明, hard cap 不适用, numeric score 不截断。与 v2-v5 review 的 Low×High=5 结论完全一致——本轮新证据 (null audit 的真实失败) 没有改变象限, 反而进一步印证了 Low 的判断依据。

## Overall

- Priority: Medium
- Score: 5
- Comments: v6 是连续第 5 轮 refine (v2→v6)。Claude 与 codex 独立核验一致确认: v5 review 指出的三处报告精度问题 (16 格未披露、LKIF"4/4"误报、anchor-regression"预测"措辞) 全部经原始 JSON 复核证实已如实、精确改正, 预注册时间证据链也已恢复, 是真实的纪律进步。本轮唯一真正新执行的实验——非退化空对照 null audit——诚实、准确地报告为 FAIL (过覆盖而非膨胀), 这是本轮唯一的新证据, 但它是一个新的推断有效性 blocker, 而非向"跨估计器一般性"或"真实大气价值"迈进的正面证据。除此之外, v6 的其余产出 (H1 降级、H2 完整协议、Causal-CCP/Theta(t^2) 协议设计) 都是对未来实验的规划, 而非已执行的证据。Overall Quality 因此从 v5 的 4/10 回升到 5/10, 但 Likelihood-Impact 象限保持 Low×High=Medium(5) 不变, 与 v2-v5 完全一致, Claude 与 codex 本轮在 Likelihood/Impact 两个轴上均无分歧。

**收敛判断 (dispatcher 特别要求)**: 这个 idea 已经连续 5 轮 (v2→v6) 处于"诚实但边际递减"的状态, 本轮尤其明显——除了修 bug 式的报告精度纠正外, 唯一新执行的实验是一次失败的校准审计, 而"必须做"的全规模 PCMCI+/LKIF/Causal-CCP 复现从 v3/v4 就被列为"最高优先级下一步", 到 v6 仍是零执行, 且理由 (先修好推断工具) 站得住脚但也意味着这个理由本身可以在任何一轮被重新祭出以推迟执行。idea 文件本身在六要素 (one-line/hypothesis/expected outcome/contribution type/risk/effort) 上已经完备、claims matrix 清晰、H2 协议具体到可执行细节, 继续在 idea 层面打磨文字和协议设计, 边际收益已经很低——下一轮能新增的诚实性修正已经不多, 而真正决定 Likelihood 走向的信息 (全规模 PCMCI+/LKIF 是否真的跨估计器一致、H2 在真实雷达数据上是否真的相对既有梯度方法有增量) 只能来自执行, 不能来自继续 refine。因此本审查建议: 不再请求下一轮 idea-refine (v7), 而是直接以 v6 为收敛版本, 把"诊断并修复两层 bootstrap 的 overcoverage, 然后在同一 16 格上跑全规模 PCMCI+/LKIF (以及可能的 Causal-CCP) 复现"作为下一步动作——这本质上是 P1 experiment 执行, 不是 proposal 写作, 也不是继续在合成设计上打磨。若该全规模复现结果稳定支持跨估计器一致的失效边界, 再考虑升级为正式 proposal (并在 proposal 阶段固化 H2 的标注协议细节); 若结果显示估计器之间根本不收敛或效应在全规模下消失, 则应在此处 PIVOT 而非继续 refine idea 文字。换言之: 用户提出的"直接跑 P1 全规模复现再决定"这一替代策略, 是本审查认为优于"继续在 idea 层面打磨"的选项。

</review>
