<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 在目标边、条件集维度和统计流程都相同的条件下, 审计滑窗因果估计的不稳定性: 区分真实边变化与混杂漂移, 判断条件化何时提高或损害 D/J 的可探测性, 这一模式在部分可观测混杂、非线性机制下能否稳健, 以及在真实强对流环境场中可能揭示的领域新现象。
- Hypothesis: 条件化指在检验目标边时控制第三变量。固定目标边、条件集维度、窗口和推断流程后, 其作用应主要取决于扰动来自真实边还是混杂机制。v4 的混杂 `z` 完全观测且线性, pre/post 只是同一曲线的时间平移, 两处都可能人为保证了结论方向。v5 同时加固三处: 条件化只用含噪代理 `w`、边效应改为饱和非线性 `tanh`、pre/post 改为两套独立基线动力学。若模式在三处加固下同时存活, 说明它不是构造的人为产物; 但方向已被 anchor regression/有限样本 minimax 决策论边界预测 (见 Novelty quick-check), 更像迁移检验而非全新发现。
- Expected outcome: v5 GPU 加固实验 (36886062) 显示: graph-only-max 的条件化收益在部分可观测 (`corr(w,z)` 实测 0.70-0.78, 非标称值)、非线性边、独立 pre/post 动力学下依然存活, 两个 phase 的 Delta CI 均排除 0 (pre 0.029[0.004,0.047]; post 0.035[0.014,0.056]); confound-only-max 惩罚反而更不稳定, full observability 下两个 phase 均不显著。官方 `LK_Info_Flow` 在同一 v4 DGP 上的 smoke-scale 复现 (32 pairs/2 seeds) 方向相反: conditioned_true 未超过 sham/marginal, 与 v3 真实 PCMCI+ 一致，说明"条件化对真实边变化有益"可能是 fixed-edge partial correlation 的特有产物, 需同规模 PCMCI+/LKIF 复现才能下结论。raw EWS 仍全面占优。
- Contribution type: empirical-finding+diagnostic
- Contribution drift note: v4 是 `empirical-finding+diagnostic`; v5 完全保留, 不新增 method/benchmark/application/theory/dataset, 也未删除已有类型。real-data 阶段新增的 storm-mode/空间梯度探索分析臂仍在此框架内 (更多分层证据, 非新 contribution type)。
- Risk: HIGH
- Estimated effort:
  - Compute: v5 加固网关实测 2.48s L4 GPU + 12.5s CPU (LKIF smoke); 全规模 PCMCI+/LKIF 复现按 smoke 计时外推 <1 GPU-hour+<2 CPU-hours。真实阶段 (若通过) 沿用 v4 估计: 10-30 GPU-hours+500-1500 CPU-hours。
  - Data: needs collection；SPC 样例、HRRR smoke subsets 已就绪, 完整序列仍被停止门阻塞; 按 storm mode 分层还需 SPC/NWS 事后分类标注 (待补充)。
  - Implementation: 加固 DGP+LKIF smoke 已完成 (<1 周); 全规模双估计器复现预计 3-5 天; 若通过, 真实数据双臂另需 4-6 周。
- Novelty quick-check: RCV-PCMCI/VCDF (2410.19412/2602.21381) 过滤不稳定边但不检验扰动来源; graph-instability (2606.01214) 变条件集深度但不比较扰动来源; coherency score (2502.14719) 复用 CI 日志不做事件判别。Siddique et al. (2026, submitted ESD, Conditioners-LKIF) 对 LKIF 做混杂强度扫描, 是最近的方法论近邻, 但用固定系数静态 VAR, 无滑窗/转折点框架。核心模式与 anchor regression (1801.06229) + 有限样本 minimax (2606.12680) 的决策论边界同构: 扰动作用于协变量本身 (对应"真实边变化") vs 作用于混杂/隐变量 (对应"混杂漂移") 决定谁被支配。v5 是把这条 population-level/i.i.d. 边界迁移到滑窗时序、部分可观测混杂场景下检验其是否依然成立, 而非提出新边界。
- Strongest objection: 三点。(1) 模式即便在加固 DGP 下存活, 方向已被 anchor regression/minimax 理论预测, "意外性"很低; 真正新颖处只在于检验该边界是否迁移到一个从未被检验过的领域 (滑窗、临近分岔、部分可观测混杂), 而 v5 尚未找到与该预测背离的场景。(2) 官方 LKIF 在同一 DGP 上的 smoke-scale 复现方向相反, 与 v3 真实 PCMCI+ 一致, 说明该模式可能是 fixed-edge partial correlation 的特有产物而非跨估计器现象，比 v4 被指出的问题更严重， 全规模双估计器复现是不可再推迟的下一步。(3) raw EWS 在四轮迭代中无一例外全面占优, 尚无证据支持真实数据上的排序反转。
- Why we should do this: v4 排除了"条件化系统性丢失预事件信号"这一强叙事, 但被指出构造可能保证了结论方向、pre/post 缺乏独立物理意义, 且未回应"跑在真实大气数据上能发现什么"。v5 用最小加固压力测试前两点, 并把真实数据阶段扩展为验证+探索, 给出两个可证伪的领域假设。
- Pilot:
  - Setup: `factorial_helpers_v5.py` 在 v4 冻结骨架上加固三处 (含噪代理混杂、tanh 非线性边、pre/post 独立基线动力学), 在决策性角点 (null/graph-only-max/confound-only-max) x phase x 混杂可观测度 `{1.0,0.5}` 上重跑同规模 (96 pairs/8 seeds/400 次两阶段+分层 bootstrap)。另用官方 `LK_Info_Flow` 包 (github.com/YinengRong/LKIF, 需 numpy<2.0) 在冻结 v4 DGP 三个决策 cell 上做 smoke-scale (32 pairs/2 seeds) 复现。
  - Metric: 同 v4 门槛 (AUC>=0.65 且 CI 下界>0.50), Delta 收益/惩罚要求 CI 完全大于/小于 0; 新增 oracle-proxy gap 与跨估计器方向一致性检验。
  - Result: graph-only-max 收益两个 phase、两档可观测度下 CI 均排除 0 (pre 0.037[0.008,0.069]→obs0.5 时 0.029[0.004,0.047]; post 0.054[0.031,0.082]→0.035[0.014,0.056])。confound-only-max 惩罚仅在 post 且 obs=0.5 时显著 (-0.030[-0.053,-0.007]), 其余三格含 0, 比 v4 更不稳定。oracle-proxy gap 对边检测不显著, 对混杂漂移检测在低可观测度下显著 (0.028[0.008,0.050])。LKIF smoke: 4 个非空 cell 的 conditioned_true 点估计全部 <= sham/marginal, 但 32 pairs 下 CI 大幅重叠, 非决定性。raw EWS 仍全面占优 (0.88-0.99)。
  - Signal: NEGATIVE (预警增量、跨估计器普适性仍不成立) + 局部正诊断信号在加固 DGP 下更稳健存活, 跨估计器一致性从假设变为待全规模复现裁决的开放问题。

<!-- 注意: 不要将 <added-on-refine> 这个 xml tag 写到报告中, 它只是模板标记, 用于指示 refine 阶段新增的字段 -->

- Claims and Claims matrix:
  - C1 (D 对真实边变化的稳健性): D 通过 graph-only-max family gate, 收益在部分可观测混杂、非线性边、独立 pre/post 动力学下依然显著, 不是 v4 DGP 构造的人为产物。
  - C2 (J 的窄边界): J 仍只在部分强边变化 cell 通过 family gate, 不得用 combined AUC 掩盖。
  - C3 (混杂漂移惩罚更加脆弱): 加固后惩罚仅在一个 phase x 可观测度组合显著, 比 v4 更不稳定, 不得声称一般性惩罚。
  - C4 (无独立预警价值): 所测估计器均未超过完整原始变量 EWS。
  - C5 (归因边界): 不等于真实 DAG 恢复、物理分岔或外部天气有效性。
  - C6 (估计器普适性未决): graph-only 收益在官方 LKIF smoke-scale 复现中方向相反, 与 v3 真实 PCMCI+ 一致; 全规模复现前不得声称跨估计器一般性。

  | Outcome | 允许的 claim |
  |---|---|
  | HARDENING-ROBUST | 仅限 partial-correlation 家族: 三处加固后 graph-only 收益依然显著, 不依赖清洁混杂/线性/同曲线平移三个构造假设; 不得称跨估计器已确认。 |
  | ESTIMATOR-SPECIFIC | 若全规模 LKIF/PCMCI+ 复现仍方向相反, 只报告 fixed-edge partial-correlation 边界, 不推广到 LKIF/PCMCI+。 |
  | ESTIMATOR-CONFIRMED | 若全规模复现方向一致, 可声称该模式跨两个真实公开估计器成立; 尚无证据支持。 |
  | NULL/ARTIFACT | 若空对照失准或加固后模式消失, 降为测量装置 artifact, 停止真实数据阶段。 |
- Narrative: 论文问题仍是"比较估计器不稳定性时, 何时能把差异归因于条件化", 但 v5 把"结论是否只是 DGP 人为构造"这一反对意见转成三处同时加固的压力测试, 并把真实数据阶段从单纯验证扩展为验证+探索。若这一模式在真实 HRRR 4D 场上重现, 其价值不止于预警, 还能检验两个具体、可证伪的领域假设: (H1) 若某类对流环境中 CAPE-shear 协同变化主要由天气尺度强迫 (如高空急流) 决定, 条件化收益应接近 confound-only-max 的窄边界 (小/不显著); 若主要来自边界层/冷池反馈等直接因果通路 (如 MCS 冷池增强下游切变), 收益应接近 graph-only-max (显著为正)。"条件化收益大小"由此变成可按已知风暴模态 (孤立超级单体/MCS-QLCS/脉冲对流) 分层检验的新诊断量, 而非只是预警特征。(H2) 若在 HRRR 格点场上逐格点计算 (D,J), 收益/惩罚的空间梯度是否勾勒出已知中尺度边界 (出流边界、干线，静态 CAPE/shear 场未必直接捕捉), 这会把因果条件化不稳定性变成独立于预警价值的边界探测工具。两个假设都是关于大气科学本身的可证伪陈述, 不依赖诊断能否跑赢 raw EWS。
- Experiments: (1) 非退化可交换空对照的 type-I/coverage 校验 (仍待办)。(2) 加固 DGP 已完成决策性角点; 若资源允许补齐 v4 原始 3x3 网格内部点位。(3) 全规模 PCMCI+/LKIF 复现 (96 pairs/8 seeds/完整 bootstrap), 现在是最高优先级、不可再推迟的下一步，当前只有 LKIF smoke-scale (32 pairs/2 seeds, 方向相反但非决定性) 结果。(4) 只有 (1)-(3) 都通过, 才对去重风暴系统做验证臂 (prospective power、严格提前量、cluster bootstrap/BH-FDR, 比较 CAPE/shear 与原始变量 EWS, 同 v4) 与探索臂并重: 按已知 storm mode 分层比较条件化收益是否对应 H1, 逐格点 D/J 空间梯度是否对应已知中尺度边界 (H2)。多 DGP/多 storm-mode 只提供 empirical/diagnostic 证据, 不构建 benchmark。
- Assets status: v5 加固 GPU 网关、官方 LKIF smoke 复现均已跑通; 完整环境场下载仍被停止门阻塞, 见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
