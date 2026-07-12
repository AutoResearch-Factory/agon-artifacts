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
