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
