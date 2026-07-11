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

<review date="2026-07-11">

## Novelty

- Score: 4/10 (Claude 初评 5/10, codex second opinion 3/10, 合并偏严取 4)
- Closest prior work:
  1. Bian et al. (2025), CNM (arXiv:2412.16235) — 确定性因果强度趋零机制.
  2. Yu & Liang (2026), ST-CND (arXiv:2606.17553) — 空间因果网络拓扑+恢复率.
  3. NPG (2026) — 跨方法在 tipping 窗口内失效, 但当作局限而非信号.
  4. Ruiz et al. (2026), Causal-Audit (arXiv:2604.02488) — 假设违背风险打分用于弃权.
  5. **Laitinen & Lahti (2022), "Probabilistic Multivariate Early Warning Signals"** (arXiv:2205.07576, codex second opinion 发现, 已通过 arxiv.org 摘要页核验存在) — 用概率化 VAR 模型的正则化/不确定性处理作为多变量 EWS, 在生态多物种模型上验证. 与本 idea 共享"把模型拟合过程的不确定性当 EWS 信号"这一元思路, 但机制是预测性 VAR (非因果发现)、系统是经典慢变临界转换 (非快变强迫), 也没有因果推断框架或双通道设计.
  6. Ashwin et al. (2026) VAR-eigenvalue EWS (arXiv:2605.28260, idea-reviewer 本轮发现) — 从 VAR 系数反推特征值判断分岔类型 (确定性动力学重构), 而非追踪 VAR 系数本身的滑动窗口不稳定性.
- Key differentiator: 本轮独立复核确认, v2 尚未被上述任何一篇直接覆盖的窄 niche 仍然成立——"因果发现方法 (PCMCI+/LKIF) 在真实假设违背下的滑动窗口时间脆弱性, 经方法内 null 校准为 (D, J) 双通道 profile, 用于事件级证伪式检验快变强对流"这一具体组合无人占据. 但 codex second opinion 与本轮补充检索一致显示: "把估计不确定性/不稳定性当 EWS 信号"本身已是一个反复出现的元技巧 (CNM/ST-CND/NPG/Laitinen-Lahti/Ashwin-VAR-eigenvalue/DAGgr/Bagged-PCMCI+ 频率/graph-instability diagnostic 至少 8 篇), 本 idea 的贡献主要是这些既有部件在新领域 (快变中尺度对流) 和新工具 (PCMCI+/LKIF 而非纯预测 VAR) 上的重组合, 方法本身新颖性有限; 只有在真实跨区域数据上得到稳健、反直觉的增量 empirical finding, 才构成实质新贡献. v2 相对 v1 的实质进步是: 完整排除了 v1 novelty check 遗漏的 self-compatibility 谱系 (2307.09552/2606.00278/2411.05625, 经原文核验其扰动轴是变量而非时间窗口) 这一此前最大的未知撞车风险, 并显式引用/区分了全部四篇最近前作, 使差异化论证从"未讨论"变为"有据可查".

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 未声明 `target-venue` 或 `preferred-contribution-types`, 也无 `## Target venues` / `## Review standards` 小节, 沿用 v1 review 的推断: 按顶级大气科学 / Earth-system methods / AI-for-science 视角评估 (Claude 与 codex 一致).

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 (Claude 6, codex 5, 合并偏严) | 显著改善: 已明确撤销 CSD/分岔机制假设, 用 fast-upstream/slow-downstream 耦合 saddle-node testbed 正面回应快慢尺度错配 (呼应 2506.01981/2509.03996 的建议). 仍存在的缺口: (1) codex 指出"事件特异信息应同时表现为 D 与 J"并非逻辑必然, 遗漏过程也可能只造成稳定偏差而非离散或跳变; (2) 方法内 null-percentile 校准解决了量纲问题, 但未证明不同估计器 (PCMCI+ 的 CI 统计量 vs LKIF 的信息流) 校准后的百分位在语义上可交换/可聚合; (3) 尚未证明因果估计器的不稳定性优于同复杂度的普通预测模型 (non-causal) 的不稳定性, 即"因果"这个前缀是否必要仍未被证伪性地检验; (4) 独立读码发现: `pilot.py` 只对 event 轨迹断言穿越 fold (`assert crossed.any(dim=1).all()`), 对 control 轨迹未做类似的"未穿越"运行时断言, 依赖的是参数设计 (ramp 上限 0.28 < fold ≈0.385) 而非验证; bootstrap CI 是在固定 64 个 calibration control 导出的 null 百分位基础上对 192 个留出对重采样, 未把 null 校准本身的抽样不确定性传播进 CI, 可能低估真实不确定性. |
| Missing evidence signals | 4/10 (Claude 6→4 采纳 codex 后下修, codex 3) | 已有大幅改善: 匹配负对照 (ramp offset 不越 fold)、五类混淆解释 (non-iid/样本量/资料质量/UaDB/surrogate null) 具名列出、baseline 清单扩至 CAPE/shear+raw EWS+RCDyM/TIPMOC+CNM/ST-CND+Causal-Audit+PLaCy+稳健 causal change detector. codex 补充的关键缺口 (Claude 认为成立, 采纳): 缺少"同复杂度非因果模型脆弱性"基线 (若普通预测模型在同一窗口下也一样不稳定, 则因果框架本身不必要); 缺少一个已知真实 DAG 会变化的 positive control regime (用于验证方法真能检测真实结构变化, 而非把一切不稳定都吃进 fragility 桶); 尚无 PCMCI+/LKIF 的官方 gate 复现 (目前仅跑了更便宜的 VAR proxy); 真实阶段缺少事件数与统计功效分析 (workspace 目前仅 3 个 HRRR smoke-test 个例, 跨区域/季节/事件类型分层后独立事件数是否足够支撑 cluster bootstrap + nested blocked CV 尚无估算), 也缺 storm-system 级去重 (同一风暴系统内多个雷达/环境采样点是否算独立单位未说明). |
| Narrative | 7/10 (Claude 7, codex 7, 一致) | 相比 v1 显著收窄且诚实: 明确聚焦"通常被丢弃的因果估计失败是否有可复现预事件时间形态", 并如实报告 Stage-1 负结果而非回避. 残留弱点: (D,J) 双阳性被设为"双族成功"判据的理论依据仍主要是设计偏好而非从假设推出的必然要求; 本轮新发现的 tvVAR-EWS (2205.07576) 与 graph-instability diagnostic (2606.01214) 尚未被纳入 novelty 叙事视野. |
| Venue contribution | 4/10 (Claude 5→4, codex 4) | 目前仍是标准因果发现方法 + 标准离散/变点统计 + 标准校准协议的组合应用于新领域; 若真实数据阶段能在跨区域/多 lead-time 上取得可复现增量并排除全部列名混淆, 可支撑一篇有清晰边界的 Earth-system/AI4Science empirical-finding 论文, 但尚不构成对顶级 ML 主会场有独立方法贡献的申报. |
| Testability | 8/10 (Claude 8, codex 8, 一致) | 本版最大的实证进步: Stage-1 gate 已经真实跑通 (单次 L4 GPU job 仅 2.44 秒), 门槛在跑之前已冻结, 得到干净的 NEGATIVE 且未触发真实数据下载——这正是"cheapest falsifier"纪律的教科书式执行. 遗留的规程缺口: 尚未明确"每个估计器分别过 gate 还是跨方法聚合过 gate"、两个 testbed regime 是否需同时通过, 以及真实阶段多 lead-time 检验的 multiplicity correction 规则. |
| Outcome realism | 4/10 (Claude 5→4 采纳 codex 后下修, codex 3) | codex 给出的具体算术校验很有说服力: 当前 testbed 上 raw EWS AUROC 已达 0.919, 若要求联合 profile 比它高出 ≥0.05, 等价于联合 AUROC 需达到约 0.969 这一近乎完美的判别水平, 换用更强的估计器 (PCMCI+/LKIF) 也难以实现这种跃迁 (虽然这一具体数字是该 testbed 特有的, 未必是所有 regime 的通例, 但足以说明当前冻结门槛在某些 regime 下可能对"raw EWS 极强"的场景不现实). 真实阶段进一步要求在噪声更强、标签更模糊的观测数据上复现跨区域增量, 累积起来构成一个依赖多个乐观条件同时成立的目标. |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致). |
| Overall Quality | 5/10 (Claude 6→5, codex 5, 合并取 5) | v2 是一次执行纪律优秀、claims 边界清晰、真正跑出并诚实报告负结果的高风险研究计划, 相比 v1 (4/10) 有实质提升; 但核心机制桥梁 (因果 vs 通用模型脆弱性的可分性)、跨方法可比性证明与真实数据统计功效仍是尚未合上的缺口, 提升的执行质量还不能抵消当前唯一实证结果 (Stage-1 VAR proxy) 是清楚的负向证据这一事实. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} (v1) contribution types: {empirical-finding, diagnostic}
- v_n (v2) contribution types: {empirical-finding, diagnostic}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`, 且即使假设声明也无越界/降级)

v1 review concern-by-concern 复核 (对照 `workspace/causal-scs-indicator/REFINEMENT_LOG.md` 与 v2 body, Claude 与 codex 独立复核后基本一致):

| v1 concern | Status | Assessment |
|---|---|---|
| CNM/ST-CND/NPG/Causal-Audit 占据宽泛 causal-warning 叙事 | resolved | v2 novelty quick-check 明确限定为 estimator fragility 而非因果网络预警本身. |
| CD-NOD/CDANs 已把 changing modules 当信号 | resolved | 明确区分"恢复变化机制 (信号)"与"估计过程时间敏感性 (本 idea 关注对象)". |
| 慢变 CSD 假设与快变强对流尺度错配 | partially resolved | 已撤销通用 CSD 解释并换用 fast-upstream/slow-downstream testbed 正面回应, 但 synthetic 到真实强对流的外推桥梁仍未建立, 且该 testbed 已给出负结果. |
| PCMCI+/LKIF 量纲不可比 | partially resolved | 方法内 null-percentile 校准思路合理, 但尚未在真实两方法上实测验证, 百分位跨方法聚合的语义等价性也未证明 (codex 独立指出同一问题). |
| 建议收窄为单一 scalar fragility score | 未采纳 (pushback), 有据 | 保留二维 (D,J) profile 是为了检验 C2 互补性 claim, 若合并为标量将无法检验该 claim; 判定为有据的 pushback, 接受. |
| "latent-process activation" 解释信号来源 | 未采纳 (pushback), 有据 | v2 明确将其列为未经验证的解释, claims 表只承诺"事件特异诊断信息", 不识别隐藏过程; 判定为有据的 pushback, 接受. |
| 重叠窗口 + Cohen's d 造成伪重复 | resolved | 已改为以轨迹/事件为统计单位, 配合 paired/cluster bootstrap. |
| 平稳对照使 pilot 设计上保证阳性 | partially resolved | 已换成匹配 ramp-offset 的非 tipping 负对照, 但目前只有一个真实 regime 跑过, 且代码未运行时验证所有 control 确实未穿越 (见 Logical gaps 行). |
| IAAFT 单独不足以排除混淆 | partially resolved | 已把 IAAFT 列为多种 null/control 之一, 但除 pilot 已跑的匹配对照外, 其余 (surrogate null, order-p Markov/moving-block) 均尚未实际执行. |
| 缺 non-iid/UaDB/样本量/资料同化负对照 | partially resolved | 已在 Expected outcome 与 Strongest objection 中具名列出五类混淆渠道, 但尚无各渠道的可操作负对照设计. |
| 缺 CAPE/shear、经典 EWS、稳健 causal baseline | resolved | Experiments 第 3 项已列全 (CAPE/shear、raw EWS、RCDyM/TIPMOC、CNM/ST-CND、Causal-Audit、PLaCy、稳健 causal change detector). |
| 少量真实个例证据不足 | partially resolved | 提出跨区域/季节留出协议, 但尚未冻结具体数据集规模、独立事件数或功效分析 (见 Missing evidence signals 行). |
| Claims 扩张为真实结构/新诊断范式 | resolved | Claims and Claims matrix 表格给出 POSITIVE/FAMILY-SPECIFIC/NULL/NEGATIVE 边界清晰的表述. |
| Self-Compatibility/LOVO 可能已占据 novelty | 未采纳 (pushback), 有据 | idea-scope deep-lit 精读原文确认 2606.00278/2411.05625 扰动轴均为变量/变量子集而非时间窗口; 判定为有据的 pushback, 接受, novelty check 已据此更新. |

三处 pushback (标量化、latent-process 解释、self-compatibility 撞车) 经复核均有充分证据支撑, 判定为合理拒绝, 不需要在本轮重新强调.

## Alternative Framing

Claude 与 codex 提出的框架建议方向互补, 合并为一个主建议 + 一个备选: **(主)** 采纳 codex 建议, 把当前框架从"证明 (D,J) 能预警强对流"收窄为"因果估计脆弱性信号的适用边界图 (applicability boundary map)"——系统地在 forcing timescale × non-iid 强度 × hidden-driver 记忆 × 真实 graph-change 四个维度上刻画 (D,J) 相对 raw EWS 与同复杂度非因果模型脆弱性基线的增量在何处存在、何处消失; 把当前已获得的 Stage-1 负结果作为该边界图的第一个标定点, 而不是需要被绕过的挫折. 这样即使后续在真实强对流上得到 NULL/NEGATIVE, 边界图本身仍是可发表的 empirical-finding+diagnostic 贡献, 不依赖最终必须在真实数据上拿到阳性结果. **(备选, Claude 独立提出)** 若团队更看重理论深度, 可将 Stage-1/1.5 阶段独立成一篇更聚焦的诊断/理论论文, 正面使用 idea-scope deep-lit 已发现的 Rabel & Runge (2511.21537) Impossibility Result B 与 Shah & Peters (1804.07203) No-Free-Lunch 定理, 系统刻画"统计脆弱性 vs 确定性因果强度变化"在弱信号 regime 下的可分性边界, 将真实强对流应用推迟到后续论文. 两种框架都仍属 empirical-finding+diagnostic, 不引入 topic 未声明限制之外的贡献类型.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅当 (D) 与 (J) 分别通过预注册事件级门槛, 且联合 profile 在冻结的方法、数据源、事件类型和 lead-time 范围内, 相对匹配负对照与最强 baseline (CAPE/shear、raw EWS、稳健 causal change detector、Causal-Audit/PLaCy 类风险分) 取得事件级增量, 才可声称该 profile 提供预警诊断信息; 任一单族通过只能称 family-specific finding; 不得声称识别真实因果结构、隐藏过程或物理分岔机制. |
| NULL | 在预注册的方法、变量、分辨率、regime 和事件样本中, 未检测到超越上述 baseline 的增量信息; 不能外推为所有因果估计器或所有强对流类型均无此信号, 只能否定当前测试范围内的具体 profile 定义. |
| NEGATIVE | 当前唯一已获得的证据 (标准化 lag-1 VAR proxy 在 fast-upstream/slow-downstream saddle-node testbed 上) 已支持: 该代理估计器的 (D,J) 不具备预事件增量且显著落后 raw EWS. 若后续 PCMCI+/LKIF 上的信号同样只能被 non-iid、样本量、资料质量、UaDB 时间尺度误判或 matched null 解释, 则应结论为该 pipeline 中的不稳定性是非特异估计伪影, 不能作为独立预警指标; 不应扩张为"因果诊断普遍无用". |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 一致判定 Low, 无分歧。理由: Stage-1 官方门槛尚未用 PCMCI+/LKIF 复现 (目前只有更便宜的 VAR proxy, 已清楚失败且方向反直觉——dispersion AUROC=0.392 < 0.5, 说明离散度在这一 testbed 上于转换前反而降低, 而非预期中的升高); 官方门槛本身在高 raw-EWS regime 下的算术要求 (联合 AUROC 需比 raw EWS 高 0.05) 接近不现实; 真实阶段还需要独立通过跨区域/多 lead-time 验证、排除至少五类具名混淆、且样本量/功效尚未评估。多个高风险条件需要同时成立, 判定为 Low.
  - Impact: Claude 初评时曾倾向 Medium (与 v1 一致, 理由是"在已成形的 CNM/ST-CND/NPG/Causal-Audit 研究线内做差异化, 而非开辟新方向"), 但复核 codex 的论证后修正为 High, 与 codex 一致: 若最乐观情形成立 (跨区域、≥6h lead-time 稳定增量, 排除全部混淆), 其意义不止是"在拥挤 niche 内再添一篇", 而是首次实证证明"因果估计器的统计脆弱性"这一认识论信号可以在多篇 2025-2026 论文 (2506.01981/2605.16128/2603.14944) 明确指出经典 (确定性) CSD/EWS 框架失效的快变强迫 regime 内提供独立预警能力——这恰好回应了该子领域当前被多篇最新论文点名的开放缺口, 而非仅仅是缺口之外的又一增量应用。因此对"若成立"的叙事影响力评级为 High, 而非仅 Medium。
  - 两轴本轮 Claude 与 codex 均一致 (Low, High), 无 disagreement 需要标注; 记录在案的是 Claude 自身复核前后对 Impact 的判断修正 (Medium→High), 供后续版本参考。
- 查表: Low x High = Medium (5).

## Overall

- Priority: Medium
- Score: 5
- Comments: v2 相对 v1 是一次高质量的 refine——REFINEMENT_LOG.md 逐条回应了 v1 review 的几乎全部 concern, 三处 pushback (双通道保留、latent-process 解释降级、self-compatibility 撞车排除) 均有据, 且 idea 真正跑通了自己承诺的 Stage-1 便宜证伪门槛并诚实报告 NEGATIVE 结果 (workspace 代码与 JSON 结果已核实一致, 无数字虚报)。但这份执行纪律尚不能转化为更高的 Likelihood: 唯一已获得的实证信号是负向且方向反直觉, 官方 (PCMCI+/LKIF) 门槛尚待复现, 新一轮独立检索 (codex 发现 arXiv:2205.07576, Claude 发现 arXiv:2605.28260/2604.24345/2604.20949, 均已追加进 landscape) 也把"估计不确定性当 EWS 信号"这一元技巧的既有覆盖范围进一步扩大, 压缩了方法新颖性空间 (Novelty 4/10, 较 v1 的 3/10 仅小幅改善)。Likelihood-Impact 查表结果 Low×High=Medium(5) 与 v1 的最终分数 (5) 数值巧合一致, 但推导路径不同 (v1 为 Low×Medium 之误算 5, 本轮为严格按矩阵表查得的 Low×High=5)。建议下一步: 先完成一次冻结的 PCMCI+/LKIF 双-regime 官方 Stage-1 gate (含至少一个已知真实 graph-change 的 positive control 和一个同复杂度非因果模型脆弱性 baseline), 若再次失败应考虑按 Alternative Framing 转向"适用边界图"收尾, 而非直接进入真实数据长循环。

</review>

## Deep-lit 补充发现 (2026-07-11, topic-scope deep-lit-tick iteration 2)

<!-- 本节由 deep-lit-tick 追加, 不修改上方已定稿 review 的分数. 本轮精读 34 篇 (topics/0710-causal-scs-landscape.md 的 N/O/P/Q 组), 均不构成新的 novelty 撞车. 与本 idea 下一轮 refine 直接相关的发现摘录如下, 完整解读见 landscape. -->

- **novelty 撞车候选降级**: arXiv:2511.04361 ("Causal Regime Detection in Energy Markets") 标题层面与本 idea 高度相似、曾被 B7 反向扩展列为最高优先级候选; 全文精读后发现该文仅 196 行、无 Results/Experiments section、摘要性能声明被作者自己注释掉但结论段落又原样保留 (内部矛盾), R-Score Tier C 下限 (7/25)。**结论: 不构成需要在 novelty quick-check 中对比的真实竞品**, 之前的警觉可以解除。
- **novelty 空间部分恢复**: arXiv:2604.24345 (landscape 此前误署名 "Franzke et al.", 已更正为 Smith/Morr/Schötz/Boers) 经全文精读判定为"诚实误差传播/稳健回归", 并非追踪估计过程不稳定性本身——此前 novelty check 把它计入压缩"估计不确定性当 EWS 信号"新颖性空间的 ≥8 篇之一并不准确, 建议下一版 Novelty 部分剔除此文, 相应恢复一点新颖性空间。
- **novelty 正面佐证增加**: arXiv:2602.02830 (SC3D, 2026-02, 同赛道最新论文) 明确将"显式时变因果图/在线设定"列为 future work, 是独立第三方论文承认"窗口不稳定性量化为信号"仍是空白的直接文本证据; arXiv:2605.08111 (TTCD) 代表"主动消除非平稳性求更准因果图"的主流立场, 与本 idea 方向互为镜像对立, 可作为 related work 中的主流对照方法引用。
- **回应 v2 review Logical gaps #4 (control 轨迹缺少运行时"未穿越"验证)**: arXiv:2401.07712 给出鞍结分岔 overshoot 安全/危险边界的严格解析公式 (区分浅/深越界两种 regime), 可直接替代当前依赖经验参数 (ramp 上限 0.28 < fold≈0.385) 的构造方式, 提供不依赖参数试凑的解析验证证书; 已确认与已入库 arXiv:2304.12786 (Attractors.jl/RAFM) 双向引用。
- **回应 Missing evidence signals 中"缺少已知真实 DAG 会变化的 positive control regime"**: 三个独立来源均可用——arXiv:2511.03168 (UnCLe) 的 TVSEM/ND8 合成基准、arXiv:2602.02830 (SC3D) 独立复用同一 TVSEM/ND8 资产 (二次独立确认可用性)、arXiv:2603.11090 (CausalTimePrior) 的 regime-switching TSCM 生成器 (含开源代码)。三者均可接入 Stage-1 作第二/第三 regime。
- **回应 Missing evidence signals 中"缺少同复杂度非因果模型脆弱性基线"**: arXiv:2601.21135 (TRACE) 的 W(t)/α(t) 双轨消融范式 (K_active 增大时"结构学错"与"分解阶段几何病态"清晰分离) 提供可直接借鉴的诊断设计模板。
- **Pilot NEGATIVE 结果外部效度缺口 (新发现, 未在 v2 review 中出现)**: arXiv:2505.19034 (R-Score S/22) 证明真实观测数据中 λ_AC1/λ_Var 常被缺失值和异常值系统性削弱/偏置, 而本 idea 冻结的 Stage-1 pilot 使用的是无缺失、无异常值的干净模拟; 这可能部分解释了为何 raw-EWS baseline 分数 (AUROC=0.919) 偏高, 建议下一版 refinement 中明确标注此外部效度缺口。
- **候选第六类混淆源**: arXiv:2604.09661 (intermingledness) 提出"分岔诱导 tipping 需低 intermingledness 诊断变量、噪声诱导转变需高 intermingledness"的判据, 为 pilot 负结果提供候选假说 (saddle-node testbed 属分岔诱导型, raw-state 变量可能天然贴近最优判别特征); arXiv:2603.08311 的隐变量结构性不可辨识, 均可补充进现有五类混淆解释之外的候选清单, 供下一版 refine 取舍。

完整撞车矩阵与全部 34 篇解读见 `topics/0710-causal-scs-landscape.md` 的 "[deep-lit-tick --scope topic, 2026-07-11 iteration 2]" 章节 (N/O/P/Q 四组)。

