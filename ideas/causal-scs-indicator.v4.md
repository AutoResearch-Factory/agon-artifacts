<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 在目标边、条件集维度和统计流程都相同的条件下，审计滑窗因果估计的不稳定性：区分真实边变化与混杂漂移，判断条件化何时提高或损害离散型 D / 突变型 J 的可探测性，以及这些信号能否超过原始变量 EWS。
- Hypothesis: 条件化指在检验目标边时控制第三变量。固定目标边、条件集维度、窗口和推断流程后，它的作用应主要取决于扰动来自真实边还是混杂机制，而不是“事件前/事件后”标签。真实目标边变化时，控制真实混杂应提高 D/J 的灵敏度；只有混杂机制漂移时，条件化可能去掉边际相关中的 D 信号，但这一损失未必适用于其他通道、时段或估计器。任何预警价值仍须独立超过完整的 variance+AC1 原始变量 EWS。
- Expected outcome: 当前只支持一个窄结论：D 稳定检测所测真实边变化，J 只在两个事件前的强变化单元过门槛；最大“仅边变化”单元中，控制真实混杂在事件前后都优于控制等维虚假变量；最大“仅混杂漂移”单元只有事件前 D 出现小幅损失，时段交互均不成立，所有估计器与原始变量 EWS 的配对差值都偏向后者。若要形成顶级 Earth-system/AI4Science empirical/diagnostic 稿件，必须在非退化可交换空对照、非线性/local-independence DGP 和第二估计器上复现这种扰动来源差异，再在去重风暴系统上证明严格提前量增量。最便宜的下一证伪是空对照失准，或第二估计器不能复现真实边变化下的收益；任一发生都继续停止完整 HRRR/radar 下载。
- Contribution type: empirical-finding+diagnostic
- Contribution drift note: v3 是 `empirical-finding+diagnostic`；v4 完全保留这两个类型，不新增 method、benchmark、application、theory 或 dataset，也未删除已有类型。
- Risk: HIGH
- Estimated effort:
  - Compute: v4 gate 实测为 1 个 L4 作业、2.13 秒核心计算；非退化空对照、非线性 DGP 和第二估计器复现预计低于 2 GPU-hours、约 50 CPU-hours。只有再过 gate 才投入真实阶段的 10--30 GPU-hours 特征抽取与 500--1500 CPU-hours 滑窗估计。
  - Data: needs collection；SPC 标签样例和三个 HRRR 3-km 字段子集已就绪，完整小时序列仍被停止门阻塞。
  - Implementation: 1--2 周完成可识别性复现；若通过，真实数据功效分析、storm-system 去重和验证另需 4--6 周。
- Novelty quick-check: RCV-PCMCI/VCDF (2410.19412/2602.21381) 用跨折 Consistency/Variability 过滤不稳定边，并未检验滑窗不稳定性在何种扰动来源下可识别；conditioning-depth graph instability (2606.01214) 通过改变条件集深度诊断隐藏混杂，却没有在等维条件集下比较扰动来源、时段和原始变量 EWS；PCMCI+ coherency score (2502.14719) 复用 CI 日志报告内部矛盾，不做事件判别；tvVAR-EWS (2205.07576) 用预测 VAR 的不确定性预警慢变生态转折。这里没有提出新的不稳定性分数。窄增量在于用真实混杂与虚假变量的等维对照及两阶段/分层推断表明：v3 的大幅损失在匹配后消失，真实边变化反而得到小而稳定的条件化收益，事件前后不构成边界。该发现目前只在一个线性 DGP 上成立。
- Strongest objection: 在线性高斯、固定目标边的场景中，“控制真实混杂有助于检测真实边变化”可能接近教科书结论；空对照又因 event/control 完全相同而机械得到 AUC 0.5。原始变量 EWS 全面占优，且没有真实风暴证据，因此当前结果更像一次严格的反证审计，还不是足以投稿的顶会贡献。
- Why we should do this: v3 的 `PCMCI+ - marginal = -0.222` 曾诱导出“因果条件化丢弃预事件信号”的强叙事，但它混合了父集搜索、显著性筛选、DGP、事件前后和窗口泄漏。v4 以很低的成本排除了这条归因，并说明什么证据出现后才值得投入真实数据。
- Pilot:
  - Setup: 在 NVIDIA L4 上运行预注册的四变量时变 VAR 因子实验：时段 `{pre,post}` × 真实边变化 `{0,0.2,0.4}` × 混杂漂移 `{0,0.25,0.5}`，每格 96 对共享随机创新的 event/control、8 个随机种子；32 个 controls 校准、64 对评估。主对比固定 `x_{t-1}->y_t`，使用相同数量的协变量，分别控制真实混杂 `z` 或独立虚假变量 `u`；400 次 calibration+evaluation bootstrap 后再重采样随机种子。事件前窗口全部在截止点前结束，事件后窗口全部在截止点后开始。
  - Metric: D/J 各自要求 AUROC >= 0.65 且 95% CI 下界 > 0.50；条件化损失/收益分别要求 `Delta=AUC_true-AUC_sham` 的区间完全小于/大于 0；时段 claim 要求至少两个非空单元的 `|Delta_post-Delta_pre|>=0.05` 且区间排除 0；预警 claim 还须以配对区间超过 variance+AC1 原始变量 EWS。
  - Result: 空对照单元为 AUC 0.500、Delta 0。最大“仅边变化”单元：事件前 `Delta_D=0.042 [0.023,0.067]`、`Delta_J=0.035 [0.013,0.054]`，事件后 `0.049 [0.020,0.075]` 与 `0.026 [0.004,0.049]`。最大“仅混杂漂移”单元只有事件前 D 为 `-0.029 [-0.055,-0.003]`；事件前 J、combined 及全部事件后 contrasts 包含 0。所有九个时段交互区间包含 0。在最大“仅边变化”单元中，true-D 仍落后原始变量 EWS：事件前 `-0.048 [-0.085,-0.018]`，事件后 `-0.065 [-0.104,-0.035]`；J 差值为 `-0.271` 与 `-0.302`。
  - Signal: NEGATIVE（generic conditioning penalty、时段边界、双族成功和独立预警价值均失败）+ 局部正诊断信号（条件化有助于 D/J 检测真实边变化）。

- Claims and Claims matrix:
  - C1（D 对真实边变化的灵敏度）: 在当前固定目标边的 DGP 中，D 在全部 12 个非零真实边变化单元通过 family gate；最大“仅边变化”单元的条件化收益在事件前后均成立。
  - C2（J 的窄边界）: J 只在两个事件前的强边变化单元通过 family gate；不得用 combined AUC 掩盖其余 J 失败。
  - C3（一般损失与时段边界均不成立）: “仅混杂漂移”条件下只有事件前 D 出现小幅损失；J/combined 及事件后损失不成立，所有时段交互均为 null。
  - C4（无独立预警价值）: 所测估计器不稳定性均未超过完整原始变量 EWS；不得声称强对流预警增量。
  - C5（归因边界）: v3 的大差值不能归因于条件化；本结果不等于真实 DAG 恢复、物理分岔或外部天气有效性。

  | Outcome | 允许的 claim |
  |---|---|
  | REPLICATED SOURCE BOUNDARY | 只有在非退化空对照、非线性 DGP 与第二估计器都复现后，才可声称扰动来源决定条件化对 D/J 的方向；事件前后仍须单独检验。 |
  | CURRENT PARTIAL | 仅声称当前线性、固定目标边的 DGP 中 D 稳定、J 窄、条件化帮助检测真实边变化，而一般损失、时段边界和原始变量增量均不成立。 |
  | ESTIMATOR-SPECIFIC | 若第二估计器不复现，只报告 fixed-edge partial-correlation 的边界，不推广到 PCMCI+/LKIF。 |
  | NULL / ARTIFACT | 若非退化空对照失准或扰动来源差异消失，结论降为测量装置 artifact，停止真实数据阶段。 |

- Narrative: 论文问题不再是“因果不稳定性会不会预警”，而是“比较估计器不稳定性时，何时能把差异归因于条件化”。当前结果给出一个可复现的反证：匹配统计流程后，v3 的一般损失与事件前后不对称消失；条件化对真实边变化有小收益，但原始状态已经包含更多判别信息。
- Experiments: (1) 用独立但同分布的 event/control 构造非退化可交换空对照，校验 type-I、等价界和 calibration coverage；(2) 在非线性/local-independence DGP 上复现同一 fixed-edge D/J estimand，并用固定 parent set 的官方 PCMCI+ 或可匹配的第二估计器复核；(3) 只有前两步通过，才对去重风暴系统做 prospective power、严格 1/3/6/12 h 提前量、cluster bootstrap/BH-FDR，并比较 CAPE/shear、完整原始变量 EWS、资料质量和稳健 causal-change baselines。多 DGP/多事件只提供 empirical/diagnostic 证据，不构建 benchmark。
- Assets status: v4 GPU gate、锁定环境、SPC 样例和 HRRR smoke subsets 已就绪；完整环境场下载因停止门继续暂停，异常与交接只见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
