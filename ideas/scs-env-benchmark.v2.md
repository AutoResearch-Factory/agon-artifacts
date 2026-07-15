---
topic: topics/0713-scs-benchmark.md
landscape: topics/0713-scs-benchmark-landscape.md
workspace: workspace/scs-env-benchmark/
---
<!-- 书写报告使用中文 -->
- One-sentence summary: 建立一个以确认强对流事件为中心的开放基准：用 PPF 把 NOAA Storm Events 离散报告聚成连续提取区域，再配对 issue-time 可审计的 HRRR 环境序列、MRMS 雷达和 IEM/NCEI 预警记录，供环境感知临近预报、预警评估及下游时变因果研究复用。
- Hypothesis: 在严格按业务可用时刻截断输入后，HRRR 环境序列相对雷达单模态的增量应在 0–2 h 较小、2–6 h 与 6–12 h 逐步增大；192 h 前置环境窗还能为 LKIF 等时变估计器保留预热数据和多个事件前读数。这里的 192 h 只约束数据供给：本数据集不计算因果强度，也不作因果主张。
- Expected outcome: 成功时发布带 DOI/版本的去重资产层、事件索引、机读溯源/许可/QC、confirmed/hard-negative/random 三类样本、无泄漏的数据划分，以及一个 0–2/2–6/6–12 h 提前量 × 历史长度示范。环境增量为零时仍可成立 ESSD 数据产品，但要删除“提升预报”的主张；最便宜的下一证伪信号是小规模覆盖审计发现四源交集、PPF 区域稳定性或逐源再分发条款不足以支撑公开样本。
- Contribution type: benchmark+application+empirical-finding
- Contribution drift note: v1 的 benchmark 与 application 均保留；新增 empirical-finding，因为“环境增量如何随 lead time、区域和季节变化”现在是预注册结果，而非附带演示。三者均属于 topic 允许集合。
- Risk: MEDIUM
- Estimated effort:
  - Compute: 数据整编以 CPU、网络和存储为主；轻量 baseline 约 100–300 GPU-hours，全量训练预算在覆盖/体量审计后再定。
  - Data: available but needs collection；四个主源已做单事件可用性检查，完整交集、缺测和再分发审计尚未完成。
  - Implementation: 小规模覆盖与区域敏感性审计 2–3 周；可发布数据产品和基准约 3–6 个月。
- Novelty quick-check: 58 篇 deep-lit 未发现同构数据产品。SEVIR/TorNet 缺长历史环境场，HR-Extreme 是预报评测快照；MYRORSS 已发布雷达加近风暴环境，因此推翻了 v1 的绝对空白表述，但它没有现代事件级任务、issue-time 语义、负例协议和标准数据划分。本工作的差异是“确认事件区域 + 长历史环境演变 + 雷达 + 预警 + 按可用时刻约束的评测协议”这一组合，并不声称首次配对雷达与环境。
- Strongest objection: 公开源拼接本身没有新颖性；若 PPF 边界对参数高度敏感、四源交集覆盖不足、issue-time-safe 环境输入没有可重复增量，或发布只能复制大量相同 HRRR/MRMS 网格，本工作就会退化为昂贵但难复用的数据汇编。
- Why we should do this: 现有资源仍要求每个团队重复完成事件、雷达、环境和预警的对齐与泄漏审计。`0713-scs-benchmark` 是上游数据基础，`0710-causal-scs` 是明确的下游消费者；后者需要连续历史场来计算 LKIF 等时变读数，两者不再按“暂各自独立”处理。

- Review decisions:
  1. **Accept warning-source gap**：主处理源采用 IEM VTEC 元数据和多边形下载，NCEI SRRS 作为官方存档与缺口核验层；字段中显式标记 2007-10-01 county-to-polygon 断点。
  2. **Accept negative-sampling gap**：confirmed 为 Storm Events 报告包；hard negative 为有强对流雷达/预警信号但无匹配报告的窗口；random 先按 EWB 的 zero-tornado + `<10` hail 日筛选，再排除局地报告/预警缓冲区。日期先整体按 Julian-day-mod-20 分配，重叠事件包不得跨 split。
  3. **Accept leakage concern, push back on -72 h replacement**：废弃无依据的 -15 d，但标准前置窗取 192 h。公式为 `H=W+A+(K-1)Δ=72+72+4×12=192 h`：72 h 估计器预热，加上要在 `T0-72 h` 得到读数，再向前保留 5 个、间隔 12 h 的趋势点。`T0` 为区域首个确认严重天气报告时刻。事件结束后最多 24 h 仅进入回顾分析视图；预测视图在各自 cutoff 截断。该长度服务下游时变因果分析的历史深度，不是土壤湿度记忆假说。
  4. **Accept issue-time gap**：每个 HRRR 字段保存 `cycle_time/issue_time/valid_time/availability_time/lead_hour/product_type/source_key`；f00 标为 analysis、f01 标为 forecast，任务只读取 cutoff 前已可用的最近 cycle，并单列 analysis-train/forecast-inference shift。
  5. **Accept coarse-pilot gap**：正式示范改为 0–2/2–6/6–12 h lead-time sweep；每档再比较 MRMS-only、当前 HRRR、72 h 与 192 h 历史，避免把“有环境”误当成“长历史有用”。
  6. **Push back on tracker-family expansion; resolve selection**：事件区域只聚合已确认的 Storm Events 点/线报告，不追踪雷达回波。主算法选 AgentCaster 的 PPF（KDE→盘卷积→Poisson），因为输入本来就是稀疏点过程；two-pass enhanced watershed 需要先人为构造连续强度场。MCSMIP 表明算法选择会显著改变统计量，因此 enhanced watershed 只作预注册敏感性对照，不作为并列默认，也不引入 tobac/PyFLEXTRKR 等单体追踪器。

- Pilot:
  - Setup: 2024-05-06 Oklahoma 单事件日，仅检查四个真实数据源能否访问、关键字段是否齐全；下载 IEM SV/TO warning CSV+Shapefile、NCEI 2024 Storm Events、一个 MRMS 最低层反射率时次，以及 HRRR f00/f01 的 CAPE/CIN/PWAT byte-range。此阶段不训练模型。
  - Metric: 四源均可重复下载并通过格式/关键字段检查即 PASS；任何一源不可取或 f00/f01 语义无法从 index 区分即 FAIL。
  - Result: 4/4 PASS，共 23,916,994 bytes。IEM 为 87 条 warning；Storm Events 有 114 条 Oklahoma 5 月 6 日记录，其中 107 条为 hail/wind/tornado；HRRR f00/f01 各含 3 个有效 GRIB 消息且 index 分别标为 `anl`/`1 hour fcst`；MRMS gzip 解压后以 `GRIB` 开头。原始文件、URL、hash 与命令见数据清单。
  - Signal: POSITIVE（仅通过数据访问和字段检查；不是环境增量或模型性能证据）

- Claims and Claims matrix:
  - C1 数据主张：发布物能把确认事件、困难负例和随机负例连接到去重的 HRRR/MRMS/预警资产，并从每个样本追溯到源文件、issue time 与处理版本。
  - C2 实证主张：只检验 issue-time-safe 环境信息的增量如何随 lead time、历史长度、区域和季节变化；不把相关增量解释为因果作用。
  - C3 下游定位：为 `0710-causal-scs` 提供至少 192 h 连续小时场；LKIF 等分析、warmup 选择和因果结论均属于下游工作。

  | Outcome | 允许的 claim |
  |---|---|
  | POSITIVE | 在预注册任务、模型和 held-out year/region 上，环境输入产生超过最小实际效应的增量，且长 lead 的增量更大。 |
  | NULL | 数据产品与协议仍可发布；只能说当前设置未检出增量，不能声称环境无信息。 |
  | NEGATIVE | 覆盖/license/PPF 稳定性失败则当前发布规格不可行；融合稳定劣化只否定所测表示与模型。 |

- Narrative: 论文把可用时刻、负例、区域定义和泄漏审计写成可复用的数据契约，再用分提前量、分历史长度的实验回答环境信息何时真正有用。
- Experiments:
  1. 在多个年份、区域和灾害类型上生成事件数、三类样本数、四源完整率、每事件体量与许可矩阵；先做 30 日小样本，再决定全量抓取。
  2. 在开发年固定 PPF bandwidth/radius/threshold；与 enhanced watershed 比较区域 IoU、面积/事件数稳定性、跨参数敏感性及人工事件包审计，冻结后才构建 canonical regions。
  3. 去重保存源资产，事件包只保存时空索引；审核 `availability_time <= task_cutoff`，并量化 f00-train/f01-inference shift。
  4. 运行 3 个 lead-time × 4 个历史条件的轻量 baseline；主指标 CSI/FSS/POD/FAR，按事件 cluster bootstrap，并报告 held-out year、region 与 season。
  5. 比较 event-level、leave-region-out、leave-year-out，并以日期整体的 Julian-day-mod-20 assignment 审计邻近泄漏。
- Assets status: 四源单事件访问和字段检查已通过；全量下载等待覆盖与 PPF 敏感性门槛，详见 `workspace/scs-env-benchmark/data/MANIFEST.md`。

<review date="2026-07-14">

## Novelty

- Score: 6/10
- Closest prior work: MYRORSS — Comprehensive Radar Data for the Contiguous United States (BAMS 103(3), 2022; NOAA/NSSL); 次近邻 TorNet (arXiv:2401.16437)、HR-Extreme (arXiv:2409.18885)、Storm250-L2 (arXiv:2510.16031, 已撤稿)。
- Key differentiator: 差异不在"首次配对雷达与环境"(MYRORSS 已做), 而在"确认事件索引 + 长历史 HRRR (issue-time/available-time 元数据) + MRMS + IEM/NCEI 预警记录 + confirmed/hard-negative/random 三类样本 + 标准 splits"这一现代开放基准组合。我独立重读 wiki `2510.03349.md`(AgentCaster)、`2012.00679.md`(enhanced watershed)、`2512.08974.md`(FuXi-Nowcast) 全文笔记, 核实 58+4 篇 deep-lit 未见同构产品, 该组合层面 gap 成立。这是数据整编新颖性, 不是方法新颖性; codex 独立评估给出相同分数与相同最近邻判断。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 | 五处 v1 硬缺口中四处 (i/ii/iv/v) 基本闭合, 但 refine 引入两处新的、经我独立核验成立的逻辑问题: (1) **192h 公式与下游 sibling 实际设计不符**——我读取 `ideas/causal-scs-indicator.v10.md` 与 `ideas/causal-scs-indicator-proposal.v3.md` 全文, 确认该下游 idea 历经 v1–v10、proposal v1–v3 十余轮迭代后固定使用 **96h** 历史窗 (`[B,24,96,64,64]` raw tensor, 4 个 72h 子窗终点 `{78,84,90,96}h`, `tau_max=24`), 全篇未出现"72h 估计器预热"、"T0-72h 读数"或"5 个 12h 间隔趋势点"这一结构, 且当前是 fixed-edge local regression proxy 而非真正 LKIF。`H=W+A+(K-1)Δ=192h` 算术自洽, 但 W/A/K/Δ 四个输入参数均无法在 sibling idea 的真实设计或任何引用文献中找到依据——这是"数字精确但输入无据"的推理链, 用"服务下游需求"包装了一个未经验证的假设; (2) **PPF vs enhanced-watershed 的技术对比不准确**：我读 `2510.03349.md` 确认 PPF 本身第一步就是 KDE (σ≈120km) 把离散点变成连续密度场, 再盘卷积+Poisson 变换+栅格转矢量——PPF 同样"构造连续场", 并不像 idea 正文所写"不需要人为构造连续强度场"; `2012.00679.md` 确认 enhanced watershed 也是作用于连续场 (WoFS 集合概率场) 的对象提取算法。两者实为流水线的不同阶段 (点→连续场→离散对象), 不是互斥的替代方案, 该技术表述需要改写为"PPF 是已发表的、专为确认报告设计的点→场转换配方, watershed 缺少这一步的标准做法"才准确。tracker-family 拒绝 (tobac/PyFLEXTRKR/MCSMIP) 本身经核验成立——这些工具面向连续格点场的逐帧对象追踪, 与"把确认报告聚类成事件区域"是不同任务, 该 pushback 有据。codex 独立评审额外指出一个我认可的新问题：用同一事件的**全部**确认报告 (含事件晚期/未来时刻的报告) 定义 PPF 区域, 再把该区域当作预测任务的空间 crop, 存在"用未来信息定义预测边界"的 localization leakage, 与已解决的 issue-time 环境泄漏是不同的泄漏向量, 当前设计未处理。 |
| Missing evidence signals | 4/10 | Pilot 真实且可复核 (见下节), 但仅证明单日四源可访问与 schema 正确, 不证明跨年份/跨区域覆盖率、真实 availability latency、许可矩阵、PPF 参数稳定性或任何预测增量。30 日覆盖矩阵、逐源再分发条款、报告漏报敏感性 (1606.06973 已证 Storm Events 本身缺失过半损失字段) 均待 handoff gate 通过后执行, 目前是计划而非证据。 |
| Narrative | 6/10 | 从 v1 的绝对空白声明收窄为组合层面差异化, 叙事纪律明显提升; 但"192h 服务下游需求"被写成既定事实支撑窗口设计, 而非待验证假设, 是当前最大的叙事风险点——一旦 proposal 阶段被指出与 sibling 实际设计不符, 整个时窗论证的说服力会塌陷。 |
| Venue contribution | 6/10 | ESSD 路径扎实, 且允许环境增量 NULL 结果仍成立数据产品; NeurIPS D&B 路径仍依赖完整覆盖、任务定义和 baseline 尚未跑通。`empirical-finding` 贡献类型合规 (见下), 但目前是预注册计划, 不是被证据挣得的既成贡献。 |
| Testability | 6/10 | 0–2/2–6/6–12h × 4 历史条件的 sweep 设计具体可执行, 比 v1 单一二元对比更有信息量; 但"覆盖不足"/"PPF 不稳定"/"最小实际效应"均未给出可提前判定的数值门槛, 经验证伪目前只能定性执行。 |
| Outcome realism | 6/10 | 去重资产层、先冻结区域再全量下载、分阶段 handoff gate (MANIFEST 已写明 5 条) 显著提升可行性可信度, 相对 v1 的"数月内整体交付"更现实; 但全国多年四源交集 + 许可 + QC + DOI + baseline 仍在 3–6 个月内完成偏乐观, 取决于尚未执行的覆盖/敏感性审计结果。 |
| Contribution type compliance | 10/10 | idea types `{benchmark, application, empirical-finding}` ⊆ preferred-contribution-types `{benchmark, application, empirical-finding}`: yes |
| Overall Quality | 6/10 | v2 是对 v1 六个硬缺口的诚实、大部分成功的回应 (i/ii/iv/v 闭合良好), 但 refine 过程中新引入的 192h 推理链与其自称的下游依据实际矛盾, 且新发现一处 localization leakage 尚未处理, 使当前版本仍未达到 proposal-ready。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} contribution types: `{benchmark, application}`
- v_n contribution types: `{benchmark, application, empirical-finding}`
- Status: expanded(+empirical-finding)
- Hard cap triggered: no（`empirical-finding` 在 topic `preferred-contribution-types` 集合内, 非越界扩张；且无 method/theory 被静默移除, topic 本身也未声明这两类）

## Alternative Framing

收紧为 ESSD-first 的 availability-aware severe-convection data contract：PPF 生成的区域只作回顾性资产索引/QC 用途, 预测类任务改用 issue-time 已知的固定网格域或在线锚点作为空间 crop, 避免用同一事件的未来报告定义预测边界 (回应上面新发现的 localization leakage)；历史契约按下游 sibling idea 实际结算的"每次 issue time 需 96h"推导, 192h 可作为面向未来更重估计器的可选冗余供给, 而非包装成下游"必需"的既定事实。此框架仍完全落在 `{benchmark, application, empirical-finding}` 内, 不引入越界贡献类型。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在预注册 target、模型预算、held-out year/region 与最小实际效应门槛下, issue-time-safe HRRR 相对 MRMS-only 产生增量; 只有当 lead-time 交互项本身显著且跨 held-out 稳健时才能声称"增量随 lead time 增大"。不得声称因果作用、普适提升, 或"192h 是下游必需"。 |
| NULL | 数据产品 (provenance/QC/splits 合格) 仍可发布; 只能说当前任务/模型/功效下未检出环境或长历史增量, 不能声称环境无信息; 功效充分的 NULL 本身可构成一条克制的 empirical finding。 |
| NEGATIVE | 覆盖/许可/PPF 区域稳定性任一失败, 仅否定当前发布规格; 若发现 localization leakage 或融合稳定劣化, 仅否定所测表示/任务定义, 不否定环境信息本身或下游因果研究的可行性。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: Likelihood = Medium——四源单日访问已真实验证 (非合成), ESSD 成稿路径与分阶段 gate 清楚, 但仍需覆盖/许可矩阵、无 localization-leakage 的任务定义、事件区域算法冻结、以及 lead-time×history ablation 本身成立等多个条件连乘; Storm250-L2 因覆盖不足撤稿是同类风险的真实前车之鉴。Impact = High——若最终成为被采用的 issue-time-safe 环境–雷达–预警统一开放基准, 将明显推进并统一强对流 env-aware ML 这条研究线, 并把姊妹 idea 的数据前提坐实; 但不含方法/理论突破, 多个组件 (PPF、issue-time schema、负例设计) 均借用已发表先例组合而成, 未达 Exceptional。Claude 与 codex 独立评估在 Likelihood/Impact 两轴上完全一致, 无 >= 1 level 分歧。

## Overall

- Priority: High
- Score: 7
- Comments: v2 相对 v1 有实质性进步 (五/六个硬缺口基本闭合, pilot 从 SKIPPED 变为真实可核验的 4/4 PASS), 但 refine 过程中的两处 domain-owner override 需要区别对待——tracker-family 拒绝 (#6) 有据成立, 而 192h 时窗公式 (#3) 的具体参数经我独立核验与下游 sibling idea 的实际结算设计 (96h) 不符, 且 PPF/watershed 的技术对比表述不准确, 二者共同构成本轮最需要在下一版修正的逻辑缺口, 外加 codex 新发现的 localization leakage。分数维持 v1 的 7 分不变——工程执行层面的真实进展与新暴露的推理缺口大致抵消, 尚未达到 proposal-ready 门槛。Claude 初评与 codex second opinion 在 novelty (6)、quality 各维度、drift 判定、hard cap (no)、priority (High/7) 上高度收敛, 无 >= 1 level 分歧。

</review>

