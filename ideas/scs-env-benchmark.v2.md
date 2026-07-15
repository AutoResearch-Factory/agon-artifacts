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
