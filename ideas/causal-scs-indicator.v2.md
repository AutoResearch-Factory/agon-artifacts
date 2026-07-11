<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 将滑动窗口中的因果估计失稳表示为两个同等重要的通道：离散型 (D)（滚动方差/标准差/置信区间宽度）与突变型 (J)（归一化一阶差分、PELT/BOCPD/CUSUM 变点、效应符号翻转、相邻窗口 DAG edge 变化），检验这个二维 profile 能否在严格预事件条件下提供超出常规非平稳性与业务指标的强对流预警信息。
- Hypothesis: 强对流发生前，若遗漏过程、regime 演变或估计器假设违背确有事件特异信息，它应以两种不同时间形态出现：(D) 捕捉一段时间内逐渐扩大的估计离散度，(J) 捕捉短时跳变与结构重排。对每个方法、edge、窗口长度和匹配 stratum 分别做 null-percentile 校准后，两族都应在事件级检验中区别于同季节、同区域、同样本量和相近漂移/自相关的非事件窗口；联合使用应有互补增益。这里不假定强对流满足经典临界慢化，也不把阳性信号解释为真实 DAG、隐藏过程或物理分岔。
- Expected outcome: 成功要求 (D) 与 (J) 在跨区域/季节留出事件上各自优于匹配负对照，且联合 profile 在至少两个预注册 lead-time（其中一个不少于 6 h）上相对最强 CAPE/shear、原始变量 EWS 与数据质量基线取得事件级 AUROC 增益 ≥ 0.03，cluster bootstrap 95% CI 下界大于 0；任一单族阳性只算 family-specific finding，不算双族成功。失败包括两族均无增量、只在包含事件后的窗口出现，或被 non-iid、样本量、资料质量、UaDB 时间尺度误判及 surrogate null 解释。最便宜的证伪信号是：PCMCI+ 与 LKIF 在两个 matched synthetic regimes 上不能让两族分别达到 AUROC 0.70、95% CI 下界大于 0.50，或联合 profile 不能比 raw EWS 高 0.05；触发后不启动大规模真实数据实验。
- Contribution type: empirical-finding+diagnostic
- Risk: HIGH
- Estimated effort:
  - Compute: synthetic gate 低于 1 GPU-hour，另需约 100–200 CPU-hours 跑 PCMCI+/LKIF 与 bootstrap；真实阶段预计 10–30 GPU-hours 做格点特征抽取/事件级模型，另需 500–1500 CPU-hours 做滑窗因果估计。
  - Data: needs collection；NOAA/SPC 事件标签样例已可用，ERA5+公开雷达或 HRRR+雷达的环境场仍需按时间分辨率与覆盖范围二选一。
  - Implementation: 6–8 周；先用 1–2 周完成 synthetic stop gate，再决定是否投入真实数据。
- Novelty quick-check: CNM (arXiv:2412.16235) 与 ST-CND (arXiv:2606.17553) 利用确定性因果强度、网络拓扑和恢复率预警，NPG 2026 已观察到因果方法在 tipping 窗口内失效但把它视为局限；本 idea 的可检验差异是“经匹配 null 校准的估计器时间敏感性是否在事件前仍有独立信息”。CD-NOD/CDANs 旨在恢复 changing modules，Self-Compatibility/LOVO 则扰动变量或变量子集，均未覆盖这里的时间窗口扰动、双族判决和事件级增量 lead-time 检验。因此新意不在“因果网络也能预警”或把现成 EWS 搬到强对流，而在对 estimator fragility 的严格、可失败诊断。
- Strongest objection: 负 pilot 已显示线性 VAR 的 (D/J) profile 远逊于 raw EWS；即使换成 PCMCI+/LKIF，所谓不稳定性仍可能只是有限窗口、普遍 non-iid、观测/资料同化变化或 UaDB 几何误判，matched controls 与稳健因果基线一旦加入，事件特异信号可能完全消失。
- Why we should do this: 这条路线已有低成本的 go/no-go 判据，无需先下载整套环境场。若两族在更合适的估计器和两个机制不同的 testbed 上仍失败，就应尽早停止；若都能越过 raw EWS，再到真实事件验证会形成一个结论边界清楚的 empirical finding。
- Pilot:
  - Setup: 在 NVIDIA L4 上生成 256 对 fast-upstream/slow-downstream 耦合 saddle-node 轨迹；每对共享 ramp 幅度、长度、隐藏 AR(1) 驱动与噪声，control 的 ramp offset 保证不越过 fold。用标准化 lag-1 VAR 作最低成本估计器 proxy，64 对 control 校准 null，192 对留出评估；所有窗口至少在下游事件前 40 步结束，统计单位是轨迹而非重叠窗口。
  - Metric: 预先规定 (D) AUROC ≥ 0.70 且 95% CI 下界 > 0.50，(J) 同门槛，等权联合 AUROC ≥ 0.75，并比 raw variance/autocorrelation EWS 高 ≥ 0.05；六项须全部通过。
  - Result: (D=0.392) [0.345, 0.437]，(J=0.517) [0.468, 0.568]，联合 (=0.428) [0.383, 0.473]，raw EWS (=0.919) [0.893, 0.943]，联合差值 (-0.491) AUROC；六项 gate 全部失败。
  - Signal: NEGATIVE；只证伪当前 testbed 上的线性 VAR proxy，不外推到 PCMCI+、LKIF 或真实强对流。下一步受 synthetic stop gate 约束，不直接进入昂贵真实实验。
- Claims and Claims matrix:
  - C1（事件特异性）: 经方法内校准后，(D) 与 (J) 各自包含不能由匹配非事件非平稳性、有限样本或数据质量解释的预事件信息。
  - C2（互补性）: 在事件级 blocked evaluation 中，(D+J) 比任一单族及最强业务/raw-EWS baseline 提供稳定增益；不能用窗口级显著性替代事件级证据。
  - C3（边界）: profile 衡量估计过程的时间敏感性，不等于真实因果结构变化、隐藏过程激活或物理 tipping。

  | Outcome | 允许的 claim |
  |---|---|
  | POSITIVE | 仅在评估的数据源、事件类型、方法与 lead-time 内，双族 profile 对匹配对照和最强 baseline 有事件级增量信息；不得声称识别真实机制。 |
  | FAMILY-SPECIFIC | 只报告通过的 (D) 或 (J) 及另一族的 null；不得删除失败族后把结果改写为“双族不稳定性成功”，而应把它作为诊断边界。 |
  | NULL | 在预注册范围内没有检测到两族的增量信息；不能推广为所有因果估计器或强对流类型都无信号。 |
  | NEGATIVE / ARTIFACT | 若信号只在事件后出现或可被 matched/null/data-quality control 解释，则该 pipeline 的不稳定性是非特异伪影，不能作为独立预警指标。 |

- Narrative: 论文围绕一个具体问题展开：通常被丢弃的因果估计失败，是否有可复现的预事件时间形态？核心证据来自双族、方法内校准、事件级证伪协议，并明确报告它相对已有因果预警、稳健 change detector 和业务 diagnostics 的增量或失败边界。
- Experiments:
  1. Synthetic mechanism gate：在耦合双稳与另一类已知真值动力系统中，分别加入 matched non-tipping drift、non-iid/自回归、样本量/缺测变化、观测跳变、UaDB 时间尺度误判与真实 graph change；用 PCMCI+/J-PCMCI+、LPCMCI/Bagged-PCMCI+ 和 LKIF 重跑冻结的 (D/J) 定义。两套 regime 均不过 gate 即停止。
  2. Real-event test：若 gate 通过，使用公开事件标签与环境场，按 region-season-event type 匹配非事件段；窗口严格截断于 1/3/6/12 h lead-time，采用 event-level nested blocked CV、cluster bootstrap，并跨区域/季节留出验证 AUROC、AUPRC、校准与 lead-time skill。
  3. Falsifiers and baselines：比较 CAPE/shear、原始变量 drift/variance/AC1、资料质量、RCDyM/TIPMOC、CNM/ST-CND、Causal-Audit、PLaCy 与稳健 causal change detector；报告 (D)-only、(J)-only、(D+J)，以及 PELT/BOCPD/CUSUM、edge threshold、窗口长度、IAAFT 与 order-p Markov/moving-block null 的敏感性。
- Assets status: synthetic 代码、GPU 负结果、NOAA/SPC 标签及三个 HRRR 3-km 字段子集已就绪；负 stop gate 暂停了完整小时序列下载，完整状态与交接仅见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
