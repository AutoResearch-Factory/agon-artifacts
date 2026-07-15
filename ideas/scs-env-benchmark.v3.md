<!-- 书写报告使用中文 -->
---
topic: topics/0713-scs-benchmark.md
landscape: topics/0713-scs-benchmark-landscape.md
workspace: workspace/scs-env-benchmark/
---

- One-sentence summary: 建立一个以确认强对流事件为中心的开放基准: 直接对 NOAA Storm Events 表格记录做时空聚类(ST-DBSCAN 式)得到事件区域, 再配对 issue-time 可审计的 HRRR 环境序列、MRMS 雷达和 IEM/NCEI 预警记录, 供环境感知临近预报、预警评估及下游时变因果研究复用。
- Hypothesis: 在严格按业务可用时刻截断输入后, HRRR 环境序列相对雷达单模态的增量应在 0-2h 较小、2-6h 与 6-12h 逐步增大。前置环境窗保留在原始量级(~15 天), 因为它是让数据集支持"事件前后环境如何时空演变"这一探索性分析的**通用能力**, 不是为了匹配某个下游消费者当前固定的管线参数; `causal-scs-indicator` 的 LKIF 用 96h 张量只是众多可能子集方式之一。窗口长度只约束数据供给, 本数据集不计算因果强度, 也不作因果主张。
- Expected outcome: 成功时发布带 DOI/版本的去重资产层、事件索引、机读溯源/许可/QC、confirmed/hard-negative/random 三类样本、无泄漏的数据划分, 以及一个 0-2/2-6/6-12h 提前量 x 历史长度示范。环境增量为零时仍可成立 ESSD 数据产品, 但要删除"提升预报"的主张; 最便宜的下一证伪信号是小规模覆盖审计发现四源交集、事件聚类稳定性或逐源再分发条款不足以支撑公开样本。
- Contribution type: benchmark+application+empirical-finding
- Contribution drift note: 与 v2 完全一致, 未增未删。三者仍全部落在 topic `preferred-contribution-types` 集合内。
- Risk: MEDIUM
- Estimated effort:
  - Compute: 数据整编以 CPU、网络和存储为主; 轻量 baseline 约 100-300 GPU-hours, 全量训练预算在覆盖/体量审计后再定。
  - Data: available but needs collection; 四个主源已做单事件可用性检查, 完整交集、缺测和再分发审计尚未完成。
  - Implementation: 覆盖 + 聚类敏感性审计 2-3 周(比 v2 双算法比较更轻, 只需调 1 个方法的 3 个超参); 发布数据产品和基准约 3-6 个月。
- Novelty quick-check: 58+ 篇 deep-lit 未发现同构产品(MYRORSS/TorNet/HR-Extreme 均非现代事件级开放基准, 详见 landscape)。本轮只换了"事件聚类"这一步的具体算法, 不影响组合层面 novelty 判断。
- Strongest objection: 公开源拼接本身没有新颖性; 若聚类超参对结果高度敏感、四源交集覆盖不足、issue-time-safe 环境输入没有可重复增量, 或预测任务的空间范围仍从含未来报告的完整事件区域截取(泄漏), 本工作就会退化为昂贵但难复用的数据汇编。
- Why we should do this: 现有资源仍要求每个团队重复完成事件、雷达、环境、预警的对齐与泄漏审计。本 topic 是上游数据基础, `causal-scs-indicator` 是明确的下游消费者之一, 但非唯一或必须绑定的消费者。
- Pilot:
  - Setup: (i) 2024-05-06 Oklahoma 四源访问/schema 检查(v2 已跑, 不变); (ii) 新增对该日 114 条真实 Storm Events 记录跑 ST-DBSCAN 式聚类(空间/时间两个独立阈值 eps_km、eps_hr, precomputed-DBSCAN 实现), 检验超参敏感性、与 NOAA `EPISODE_ID` 分组的一致性(ARI)、以及预测视图 vs 回顾视图的 bbox 差异。
  - Metric: 四源 PASS(不变); 聚类侧看 eps 是否真的改变簇数、ARI 是否有信息量、cutoff-vs-full 的 bbox 差异是否可测。
  - Result: 4/4 PASS(不变)。聚类: 簇数随 eps 从 1(粗)到 17(细); 默认组(eps_km=30, eps_hr=1.5h)ARI=0.206 为本轮最高, 且比 NOAA 的 2 个 episode 更细粒度; leakage ablation 显示同一簇 bbox 从 cutoff 时的数十公里量级膨胀到全事件的 215x58km, 差 3 倍以上。数值明细见 `workspace/scs-env-benchmark/data/MANIFEST.md`。
  - Signal: POSITIVE(真实数据; 聚类方法可行、超参敏感性需冻结前审计、泄漏量级已量化)。

<!-- 注意: 不要将 <added-on-refine> 这个 xml tag 写到报告中, 它只是模板标记 -->

- Claims and Claims matrix:
  - C1 数据主张: 发布物能把确认事件、困难负例和随机负例连接到去重的 HRRR/MRMS/预警资产, 并从每个样本追溯到源文件、issue time 与处理版本。
  - C2 实证主张: 只检验 issue-time-safe 环境信息的增量如何随 lead time、历史长度、区域和季节变化; 不把相关增量解释为因果作用。
  - C3 下游定位: 提供~15 天前置的连续小时场作为探索性环境-时空演变分析的通用底座; `causal-scs-indicator` 等下游工作按自己的方法需要从中截取子窗(如 96h), 数据集本身不主张任何特定截取长度是"必需"的。
  - C4 区域主张: 事件区域由对 Storm Events 表格记录的时空聚类产生, 参数须经 30 天样本敏感性审计后冻结; 预测类任务的空间裁剪只能用 cutoff 前已发布报告定义, 不得用事件全部报告(含未来报告)。

  | Outcome | 允许的 claim |
  |---|---|
  | POSITIVE | 在预注册任务、模型和 held-out year/region 上, 环境输入产生超过最小实际效应的增量, 且长 lead 的增量更大。 |
  | NULL | 数据产品与协议仍可发布; 只能说当前设置未检出增量, 不能声称环境无信息。 |
  | NEGATIVE | 覆盖/license/聚类区域稳定性失败则当前发布规格不可行; 融合稳定劣化只否定所测表示与模型。 |

- Narrative: 论文把可用时刻、负例、区域定义(含泄漏审计)和聚类敏感性审计写成可复用的数据契约, 再用分提前量、分历史长度的实验回答环境信息何时真正有用。
- Experiments:
  1. 在多个年份、区域和灾害类型上生成事件数、三类样本数、四源完整率、每事件体量与许可矩阵; 先做 30 日小样本, 再决定全量抓取。
  2. 在 30 天开发样本上对 ST-DBSCAN 式聚类的 eps_km/eps_hr/min_samples 做网格搜索, 用 ARI(对 EPISODE_ID)、簇数稳定性和跨样本重复性冻结默认超参; 记录选型对下游事件计数/区域面积的敏感性(仿 MCSMIP 思路)。
  3. 冻结参数后, 每个事件同时生成"预测视图"(仅用 cutoff 前报告聚类)和"回顾视图"(全部报告), 量化 bbox 差异分布, 确保预测任务只读预测视图。
  4. 去重保存源资产, 事件包只保存时空索引; 审核 `availability_time <= task_cutoff`, 并量化 f00-train/f01-inference shift。
  5. 运行 3 个 lead-time x 4 个历史条件的轻量 baseline; 主指标 CSI/FSS/POD/FAR, 按事件 cluster bootstrap, 并报告 held-out year、region 与 season。
  6. 比较 event-level、leave-region-out、leave-year-out, 并以日期整体的 Julian-day-mod-20 assignment 审计邻近泄漏。
- Assets status: 四源单事件访问和字段检查已通过; 新增对真实 114 条 Storm Events 记录的聚类 pilot 已跑通(POSITIVE); 全量下载等待覆盖与聚类敏感性门槛, 详见 `workspace/scs-env-benchmark/data/MANIFEST.md`。

- Review decisions(1/2/4/5 维持 v2 回应不变, 简述; 3/6 为本轮 owner 直接修正, 覆盖 v2 与 review 自身建议; 7 处理 codex 新发现的 leakage):
  1. **warning-source gap(不变)**: IEM VTEC 元数据/多边形为主处理源, NCEI SRRS 为官方存档核验层, 2007-10-01 断点已标记。
  2. **negative-sampling gap(不变)**: confirmed=报告包; hard negative=有信号无报告窗口; random 按 EWB 判据筛选并排除缓冲区; Julian-day-mod-20 split, 事件包不跨 split。
  3. **前置窗口(owner 修正)**: v2 的"H=W+A+(K-1)Delta=192h"公式, 其四个输入(72h warmup/72h 偏移/5 个 12h 趋势点)在下游 `causal-scs-indicator.v10` 的真实设计(固定 96h 张量)或任何文献中都找不到依据, 是算术自洽但输入无据的推理链。Owner 判断: 窗口的真实用途是让分析者探索事件前后环境演变的时空特征, 这是数据集的通用能力, 不是为了匹配某个下游消费者当前结算的管线参数。撤回 192h 公式, 窗口恢复原始量级(~15 天), 精确天数不是重点: 数据集提供足够历史深度供探索性分析, LKIF 只是示例消费者之一, 各下游方法按自己需要从窗口里截取子集, 数据集不必对齐任何单一消费者的具体选择。
  4. **issue-time gap(不变)**: 每个 HRRR 字段存 `cycle_time/issue_time/valid_time/availability_time/lead_hour/product_type/source_key`; f00=analysis、f01=forecast, 只读 cutoff 前最近 cycle。
  5. **coarse-pilot gap(不变)**: 正式示范为 0-2/2-6/6-12h lead-time sweep。
  6. **事件聚类(owner 修正, 推翻 v2 的 PPF 选型)**: v2 选 PPF 优于 watershed 的理由("PPF 不需要连续场")不成立, PPF 首步就是 KDE 构造连续密度场。更根本的问题是两者都不是正确的任务 family: 本任务不是"离散点变连续场再提对象", 而是**直接对 Storm Events 表格记录做数据聚类**, 簇的时空范围就是下载 MRMS/HRRR 的区域。改用 ST-DBSCAN 式(空间/时间两个独立阈值, precomputed-DBSCAN 实现), 已在真实数据上跑通(见 Pilot): 超参敏感性真实存在, 且与 NOAA `EPISODE_ID` 有交叉验证信号。arxiv-tools 检索(S2 反复 429, arXiv fallback 多为无关噪声)未找到直接同构先例, 故不强行引用背书, 依据是数据本身的扫描结果。tobac/PyFLEXTRKR/MCSMIP 仍是错误任务 family(连续网格场逐帧追踪), 继续排除。
  7. **localization leakage(codex 二次意见, 本轮新处理)**: 用事件全部报告(含未来报告)定义区域再当预测任务裁剪会泄漏未来信息。解决: 区分"预测视图"(只读 `task_cutoff` 前报告聚类)与"回顾视图"(全部报告)。Pilot 的 leakage ablation 已量化差异(3 倍以上), 写入 Experiments #3 与 MANIFEST handoff gate。

<review date="2026-07-15">

## Novelty

- Score: 5/10
- Closest prior work: MYRORSS (BAMS 103(3), 2022; 雷达+近风暴环境分析捆绑发布, 已在 v2 review 中确立); **FunnelCloud — a cloud-based system for exploring tornado events** (Int'l J. Digital Earth, 2017, DOI:10.1080/17538947.2017.1279235; 新发现, 见下)。
- Key differentiator: 本轮 codex second opinion 发现且经我用 WebFetch/WebSearch 独立核验(NCEI FAQ 原文 + 多个独立摘要级来源交叉一致, 但 tandfonline.com 与 researchgate.net 均 403 拒绝全文访问, 未能读取完整方法章节, 按摘要级证据谨慎记录, 已 append 到 landscape)成立的一点: **FunnelCloud (2017) 已经对龙卷报告记录本身运行 ST-DBSCAN 聚类, 再关联 NARR 环境再分析场与 NEXRAD 雷达数据**——这与 v3 owner 修正后采用的"直接对 Storm Events 表格记录做 ST-DBSCAN 式聚类"技术选型在方法组合层面几乎同构。这直接推翻了 v3 Review decisions #6 "arxiv-tools 检索未找到直接同构先例, 故不强行引用背书"的表述——先例是有的, 只是不在 arXiv/S2 覆盖范围内(与 landscape 已反复记录的"storm-tracking 算法文献系统性不在 arXiv"元发现完全一致, 属该盲区的一个具体命中实例, 而非检索失误)。好消息是 FunnelCloud 限定单一灾害(龙卷)、是"探索/查询系统"而非发布的 splits/baselines/DOI 版本化现代 ML 基准, 所以"开放、版本化、issue-time-safe、多灾害(hail/wind/tornado/flash-flood)、负例采样、canonical split 的现代 ML 训练就绪基准"这一组合层面差异化主张在 MYRORSS+FunnelCloud 两个最近邻之上仍然成立, 但 idea 必须在下一版明确承认并引用 FunnelCloud, 不能再声称聚类选型"未找到同构先例"。无方法新颖性(预期内, benchmark idea); 真正可能新颖的仍是"issue-time-safe 环境增量随 lead time/区域/季节的严格无泄漏异质性发现", 但该发现本身尚待跑出。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 3/10 | 三处 v2 遗留问题中一处真正 resolved、一处仅表面 resolved 实质未解、一处 partially resolved 但暴露了更深的新缺口。(1) **PPF/watershed 技术错误——resolved**: 我独立读 `cluster_events_pilot.py` 确认距离矩阵 `d(i,j)=max(d_space/eps_km, d_time/eps_hr)` 直接对两两报告的经纬度/时间做 precomputed-DBSCAN, 全程无 KDE、无栅格、无连续场构造, 技术表述现在准确, codex 独立复核一致。(2) **15 天窗口——unresolved, 且回避了本仓库自己的饱和证据**: `ideas/scs-env-benchmark.v1.md` 的"缺口(iii)"追读(4 篇全文精读 + 2 篇权威综述, 跨南非/大平原/西非/印度/全球尺度)与 `topics/0713-scs-benchmark-landscape.md` 的整合结论(2026-07-14, 与本次 review 同一仓库、仅早一天)都已明确给出"15 天在文献中无支撑证据, 应放弃 15 天设计, 改用可辩护的 -72h(或另寻大尺度环流/ENSO 一类完全不同的机制论证)"这一具体建议。v3 的"通用探索能力, 不追求匹配下游"这一新表述完全没有引用、反驳或提及这条本仓库自产的、饱和的、日期上几乎同时的证据链, 只是换了一种说法就把已被明确否定的具体数字重新写回去——这不是"给窗口一个更谦逊的定位", 而是对已有定论的静默绕行。若窗口真的"精确天数不是重点", 更诚实的选择是直接采用已有证据支持的 -72h, 而不是回退到已被证伪的 -15d。(3) **localization leakage——仅 partially resolved, 且揭示了一个更根本的问题**: 我独立复读 `cluster_events_pilot.py` 的 `main()` 确认 codex 的关键发现属实——`labels = st_dbscan(df, ...)` 对全部 114 条记录(含同一簇里时间上更晚的报告)一次性算出簇归属, 随后的"cutoff"消融只是对这个已经用了未来信息定出的簇成员按时间顺序取前缀, 并未真正"只用 cutoff 前可得的报告重新聚类"——不能证明预测视图机制本身可行, 只证明了"同一簇内早期报告的 bbox 比全部报告小", 这是一个较弱、不同的命题。更严重的是: 经我用 WebFetch 核验 NCEI 官方 FAQ 原文, NOAA Storm Events Database 本身是"data month 结束后约 75–90 天"才发布的事后档案产品(FAQ 原句: "the Storm Events Database are updated within 75–90 days after the end of a data month"), 与 HRRR 的小时级 issue-time 语义完全不是一个量级。这意味着在任何真实的 0–2/2–6/6–12h 业务预测时刻, Storm Events 记录本身根本不存在于"已发布"状态——"预测视图=只读 task_cutoff 前的 Storm Events 报告"这句话在业务时间尺度上没有非空的所指, 这不是一个实现细节, 而是这套预测视图设计从数据源层面就不成立, 需要换成真正近实时的源(如 NWS Local Storm Reports 文本产品或雷达/预警信号)才能定义 cutoff-safe 的预测区域。 |
| Missing evidence signals | 4/10 | 我与 codex 各自独立重跑/核对 `cluster_events_pilot_log.txt`, 数字(1–17 clusters、默认组 ARI=0.206、cluster 0 从 68×25 到 215×58 km)与代码逐行吻合, 不是编造。但证据强度有限: 5 组手选超参不是完整 sweep; ARI=0.206 只是"本轮最高", 且 97/114 记录集中在同一 NOAA episode 的基率会让任何合理聚类都偏向"看起来纯", 削弱其独立信息量; leakage 的"前 25%报告"不是真实业务 cutoff 时刻的良好代理(见上); idea 自己的 Pilot 记为单一"Signal: POSITIVE"掩盖了这些局限, 更诚实的记法应是 MIXED——可执行性与超参敏感性证据为正, 区域外部有效性与"无泄漏预测视图确实可行"两条尚未获得支持。覆盖率/许可矩阵/真实 availability 审计仍和 v2 一样是零证据。 |
| Narrative | 4/10 | 换掉 PPF/watershed 一处确实让技术表述更准确, 是真实进步; 但"未找到直接同构先例"被 FunnelCloud 证伪, "窗口是通用能力"回避而非回答"为什么是 15 天"这个具体问题, "预测视图已解决泄漏"在代码和数据源两层都超出了实际证据支持——三条主线索的说服力都被这轮独立核验削弱了, 且窗口与 FunnelCloud 两条本可通过一次文献/仓库自查就避免。 |
| Venue contribution | 5/10 | ESSD 路径概念上仍站得住, 但预测视图机制的数据源缺陷意味着"预警 lead-time 评估"这条任务线需要重新设计事件区域来源, 不是执行层面的收尾工作; NeurIPS D&B(现称 Evaluations & Datasets)对 2026 的要求(持久托管、Croissant/RAI 元数据、可审阅样本)尚未被涉及。`empirical-finding` 合规(见下), 但目前仍是预注册计划。 |
| Testability | 5/10 | 0–2/2–6/6–12h × 历史条件的 sweep 设计具体可执行; 但 idea 自己的"最便宜证伪信号"清单里没有列出真正最便宜、最可能命中的那个测试——"按 Storm Events 的真实发布时滞, 统计有多少比例的预测样本能拿到任何 cutoff-前可用的报告来定义 crop", 这个测试几乎肯定会立刻证伪当前的预测视图设计, 应该被前移为第一道 gate, 而不是等到覆盖矩阵之后才发现。 |
| Outcome realism | 4/10 | 去重资产层、先审计后下载的顺序仍然正确; 但窗口从 v2 的 192h 回退到 ~15 天(未同步调整 Estimated effort 文本, 与 v2 逐字相同)、加上预测视图需要更换数据源重新设计(而非调参), 3–6 个月的数据产品交付估计比 v2 时更加乐观, 而不是更现实。 |
| Contribution type compliance | 10/10 | idea types `{benchmark, application, empirical-finding}` ⊆ preferred-contribution-types `{benchmark, application, empirical-finding}`: yes |
| Overall Quality | 5/10 | 七维平均 5.0。本轮的真实进展(PPF/watershed 技术修正、真实可复现的聚类 pilot)被两处独立核验坐实的新/未愈问题抵消并反超: 窗口问题从"公式无据"退化为"对本仓库自己饱和证据的静默绕行", 且新发现 localization-leakage 修复建立在一个不具备近实时可得性的数据源(Storm Events)之上, 从"部分处理"降级为"整条预测视图机制的前提不成立"。与 v2 相比这不是进步, 是同一深度问题换了一层更隐蔽的包装。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} contribution types: `{benchmark, application, empirical-finding}`
- v_n contribution types: `{benchmark, application, empirical-finding}`
- Status: unchanged
- Hard cap triggered: no(类型集合未变, 无越界扩张, 无 method/theory 静默删除; topic 本身也未声明这两类)

## Alternative Framing

收紧为双视图数据契约, 且把"预测"与"回顾"两个视图建立在**不同的数据源**上而不只是同一数据源的不同读取时刻: 回顾视图沿用 ST-DBSCAN 对 Storm Events 表格记录聚类, 专供事件索引/QC/climatology 用途, 明确不参与任何预测任务; 预测视图改用真正近实时可得的源(NWS Local Storm Reports 文本产品、雷达/预警信号, 或固定网格锚点)定义 cutoff-safe 的空间 crop。前置环境窗按本仓库自己的 landscape 结论收窄到有证据支持的 -72h 作为默认, 更长历史(如涉及土壤湿度以外确有多日至季节尺度机制的变量, 例如大尺度环流持续性)需要单独论证, 不与"通用探索能力"这一无需论证的说法混用。旗舰问题仍是 issue-time-safe 环境增量随 lead time/区域/季节如何变化, 不引入 preferred 集合外的贡献类型。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅当预测任务使用真正 cutoff-safe(非 Storm-Events-衍生)的空间锚点、预注册最小实际效应、held-out year/region 且功效充分时, 才能声称 HRRR 环境输入相对 MRMS-only 产生增量; 只有 lead-time 交互项本身显著且跨 held-out 稳健才能声称"增量随 lead time 增大"。不得声称 ST-DBSCAN 应用于风暴报告是新方法(FunnelCloud 2017 已有直接先例)、15 天窗口有物理依据, 或当前"预测视图"设计已解决泄漏。 |
| NULL | provenance/QC/coverage 合格的数据产品仍可发布; 只能说当前设置未检出环境或长历史增量, 不能声称环境无信息; 功效充分的 NULL 本身可构成一条克制的 empirical finding。 |
| NEGATIVE | 覆盖/许可/聚类区域稳定性任一失败, 仅否定当前发布规格; 若 Storm Events 的真实发布时滞证明"预测视图"在业务时间尺度上无法用该数据源构造, 这仅否定"用 Storm Events 定义预测 crop"这一具体设计, 不否定回顾性事件索引、固定网格预测任务, 也不否定环境信息本身或下游因果研究的可行性。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: Likelihood = Medium——四源单事件访问与聚类 pilot 均为真实、可独立复现的证据, ESSD-first 成稿路径依然清楚; 但本轮新增/坐实的两处问题(15 天窗口对本仓库饱和证据的绕行、预测视图机制建立在无近实时可得性的 Storm Events 之上)都是有明确修复路径的工程/设计问题(收窄窗口或另寻机制论证; 换用近实时报告源定义预测 crop), 不构成不可逾越的障碍, 故维持 Medium 而非降为 Low。Impact = High——若最终交付一个真正 issue-time-safe、覆盖多灾害、有 splits/baselines 的开放强对流环境-雷达-预警基准, 仍会明显推进 env-aware nowcasting/warning 这条研究线并坐实姊妹 idea 的数据前提; 但 MYRORSS 与新发现的 FunnelCloud 已分别占据"雷达+环境"和"报告聚类+环境+雷达"两个组件层面的在先系统, 方法本身不构成突破, 未达 Exceptional。Claude 独立评估与 codex second opinion 在 Likelihood(Medium)、Impact(High)、Priority(High)、numeric score(7)、hard cap(no)上完全一致, 无 >= 1 level 分歧; codex 独立发现的 FunnelCloud 先例与 Storm Events 发布时滞两点, 经我用 WebFetch/WebSearch 分别独立核验后确认属实, 已采纳入本报告并 append 到 landscape。

## Overall

- Priority: High
- Score: 7
- Comments: v3 相对 v2 有一处真实、可核验的技术修正(PPF/watershed 表述改正, 且换用的 ST-DBSCAN 有真实 pilot 数据支撑其超参敏感性), 但另外两处修复都不如自我报告的扎实: 15 天窗口是对本仓库自己饱和文献结论的静默绕行(未引用、未反驳, 只是换了措辞), localization-leakage 的"预测视图"修复经代码复核和 NCEI FAQ 核验后发现建立在一个发布时滞 75–90 天、在业务时间尺度上不存在"cutoff 前可得报告"的数据源之上, 问题比 v2 review 提出时更深。novelty 侧新发现 FunnelCloud(2017, ST-DBSCAN 直接作用于龙卷报告+NARR+NEXRAD)是本轮 codex 独立找到、经我独立 WebFetch/WebSearch 核验属实的先例, 推翻了 v3 "未找到直接同构先例" 的表述, 但不推翻组合层面(开放、版本化、多灾害、issue-time-safe、canonical-split 现代 ML 基准)的整体差异化。分数维持 v1/v2 的 7 分不变: Likelihood/Impact 两轴的判断本身未变(仍是 Medium × High), 新发现的问题都有清楚的工程修复路径(收窄窗口或换机制论证; 换用近实时报告源定义预测 crop; 补充引用 FunnelCloud), 不构成 Low likelihood 或 Impact 降级的理由。未触发 contribution-scope hard cap。Claude 与 codex 在本轮全部维度上高度收敛, 无 >= 1 level 分歧。

</review>
