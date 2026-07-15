<!-- 书写报告使用中文 -->
---
topic: topics/0713-scs-benchmark.md
landscape: topics/0713-scs-benchmark-landscape.md
workspace: workspace/scs-env-benchmark/
---

- One-sentence summary: 建立一个以风暴为中心、把**长时窗环境场 (HRRR) + 雷达 (MRMS) + 事件标签 (NOAA Storm Events)** 统一封装成事件包的开放强对流 (SCS) 基准数据集, 填补"现有开放集 (SEVIR/MYRORSS/HKO-7/MeteoNet 等) 无一以环境场时空演变为一等公民"的空白, 并交付从收集到训练 ready 的全流程工作流代码。
- Hypothesis: (benchmark idea, 此处记数据集要支撑的核心可用性主张) 把长时窗 (事件区域时间起止分别往前 15 天、往后 5 天) 环境演变 + 风暴中心 MRMS/原始雷达 + 预警/报告标签统一封装后, 单一数据产品即可同时支撑六类任务 —— 环境诊断 / CI 检测 / 反射率与灾害场 nowcasting / 分割 / 追踪 / 预警 lead-time 评估; 且"环境场时空演变"作为**显式模态**能提供 SEVIR 式纯影像/纯雷达基准给不出的预报增量与过程可诊断性。
- Expected outcome: 成功 = 发布一个带 DOI/版本、provenance 完整可追溯到文件与处理步、license 矩阵清晰、canonical splits (event-level + leave-one-region-cluster-out + leave-one-season/year-out) 与两层 baselines (tier1: 持久/光流/RF/GBT; tier2: ConvLSTM/TrajGRU/PredRNN/Earthformer) 的数据产品, 且 ≥1 个 nowcasting baseline + 1 个预警 lead-time 示范能跑通, 并显示长时窗环境场带来**可测量**增量; 失败/最便宜证伪 = 在最小可行子集 (1 季节 × 1 事件聚类区域) 上, 加入长时窗环境场相对纯 MRMS 影像基线在任一核心任务 (CI 检测或 0–2h 反射率 nowcasting) 上无可测量且方向一致的增益, 或时间对齐/许可/体量任一被证明不可控。
- Contribution type: benchmark+application
- Risk: MEDIUM
- Estimated effort:
  - Compute: 数据整编以 CPU + 大规模存储为主 (下载 / 谐一化 / 时空对齐 / QC); baseline 训练需中等 GPU-hours (ConvLSTM/Earthformer 量级); 全量体量巨大, 须先做最小可行子集再决定是否全量。
  - Data: needs collection (全部公开可得但需大规模下载 + 谐一化 + 对齐 + QC + 逐源许可审计); 事件/预警标签为 link + 不确定性标注 (annotation-lite)。
  - Implementation: 最小可行子集数周; 全量数据产品 + 工作流代码 + 示范分析 数月。
- Novelty quick-check: 尚未做正式检索 (本 idea 由用户提供的一页纸课题笔记 + 一份**未核验**的 deep-research 蓝图直接转写, 跳过了 idea-creator 的 landscape survey / 生成阶段)。用户蓝图初判最近邻: **SEVIR** (多模态风暴影像, 但固定 4h 窗、不以环境场时空演变为核心)、**MYRORSS** (雷达再分析 + 近风暴环境分析, 但非现代基准封装、无标准 ML splits)、**WeatherBench 2** (全球中期预报评估, 非风暴尺度/预警)、**HKO-7 / MeteoNet** (雷达为主 / 地域局部)。差异化卖点 = 事件中心风暴演变 + 长时窗环境上下文 + 预警相关标签三者**一次性开放封装**。⚠️ 上述对照全部未经 arxiv-tools 核验; idea-reviewer 需运行完整 novelty check, 并特别核查 2024–2026 是否已有"环境场为一等公民"的强对流多模态基准 (如 TorNet 及其后续) 抢先占坑, 以及"用环境场提升 nowcasting/warning"是否已有等价数据产品。
- Strongest objection: 数据集贡献的价值不在"把公开数据拼起来"(这一步几乎无新颖性), 而在于证明"长时窗环境场时空演变"能带来纯影像/纯雷达基准给不出的**可测量**预报或诊断增量; 若最小可行子集上加入环境场相对纯 MRMS 基线无增益, 或 license 混用 / 时间对齐泄漏 / 体量任一失控, 则数据产品退化为"又一个强对流数据集", 达不到 ESSD / 顶会 D&B 门槛。
- Why we should do this: 现有强对流开放资源按用途割裂, 无一以环境场时空演变为核心; 一个把 MRMS/原始雷达 + 长时窗环境 + 预警/报告标签统一封装、provenance 与许可清晰、带 canonical tasks/splits/baselines 的数据产品能同时服务过程研究、nowcasting 与预警评估, 并为姊妹 idea `causal-scs-indicator` 提供真实数据底座, 复用价值高。
- Pilot:
  - Setup: 选 1 个季节 × 1 个事件聚类区域, 走通"收集 → (1) 汇集态源数据 → (2) 训练 ready"最小工作流; 构造两组输入 —— 纯 MRMS 影像基线 vs 加入 HRRR 长时窗环境场 —— 训一个轻量 ConvLSTM 做 0–2h 反射率 nowcasting (或 CI 检测)。
  - Metric: 加入环境场相对纯影像基线在 CSI@阈值 / FSS (nowcasting) 或 POD/FAR (CI) 上的增量; 若核心任务上出现可测量且方向一致的增益 (如 CSI 相对提升超过预设阈值), 信号为正。
  - Result: pending
  - Signal: SKIPPED (跳过 idea-creator 阶段, 未运行 pilot; 建议作为 refine / 阶段一实验第一步执行)

<review date="2026-07-13">

## Novelty

- Score: 5/10
- Closest prior work: SEVIR (NeurIPS 2020; 事件中心五模态 4h 窗, 已链接 Storm Events, 无环境场); MYRORSS (BAMS 2022; MRMS 框架融合雷达 + 近风暴环境分析, 1998-2011, 非 ML 基准); Coffer et al. 2025 (arXiv:2503.15466; GridRad-Severe 风暴轨迹 + HRRR 环境配对, 分析论文非开放基准); Extreme Weather Bench (arXiv:2605.01126, 2026; 高影响天气评估框架含对流 outbreak cases 与全球 storm reports, 非训练数据集).
- Key differentiator: "长时窗环境**演变**序列 + 风暴中心 MRMS + 事件/预警标签 + canonical splits/baselines" 的一次性现代开放封装经多源检索确无占坑者 (Storm250-L2 arXiv:2510.16031 为最近似的风暴中心封装尝试, 纯雷达, 且已于 2026-06-25 因事件覆盖不足撤稿). 但 idea 的绝对表述 "现有开放集无一以环境场为一等公民" 不成立 — MYRORSS 已捆绑近风暴环境分析. 无方法新颖性 (预期内, benchmark idea); 真正可能新颖的是 "环境增量随 lead time/区域/季节如何变化" 的 empirical finding. 注意 Leinonen et al. 2022 (NHESS 22:577) 已在 0-60 min 雷暴 nowcasting 中测试 NWP 输入并发现其缺失可被观测大体补偿 — 对 pilot 所选的 0-2h 短 lead 增量是直接的先验负面证据.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 | (i) 数据模型仅含 Storm Events (事后报告), 无 NWS warning/CAP 档案源, "预警 lead-time 评估" 任务与数据源不闭合; (ii) 无负例/非事件采样设计, CI 检测、FAR 与预警任务不可识别; (iii) 时窗设计: +5d 环境入预测输入即后视泄漏 (仅可回顾分析), -15d 无物理论证 (topic 引用的理想化湿度实验是小时尺度) 却是主要成本驱动; (iv) HRRR f00/f01 的 issue-time/可用延迟约束未处理; (v) pilot 的 "纯 MRMS vs +环境" 二元对比不能证明"长历史"价值, 至少需 contemporaneous/短历史/长历史三档消融; (vi) "事件聚类区域" 承载事件包与 splits 定义但从未被定义. |
| Missing evidence signals | 3/10 | 缺: 逐年×区域×灾害的事件与负例覆盖矩阵 (Storm250-L2 正因 "insufficient event coverage" 撤稿, 该风险已被实证); MRMS-HRRR-标签完整交集与缺测率; 每事件体量/下载吞吐/总存储审计; 逐源再分发 license 结论; split 邻近泄漏审计. Pilot 标记 SKIPPED, 全部主张目前零证据. |
| Narrative | 6/10 | "现有资源按用途割裂" 的 gap 故事有力, 且组合层面经核验成立; 但六任务 + 双 venue 同时追求摊薄主线, 绝对化 gap 表述会被审稿人用 MYRORSS 一击反驳. |
| Venue contribution | 6/10 | ESSD: in kind 合格, 成败取决于 provenance/QC/license 执行 — 这正是 topic 自己声明的评估维度; NeurIPS D&B (2026 已更名 Evaluations & Datasets, 要求明确 evaluation question): 六任务宣称 + 一个小 pilot 不够, 需环境增量的稳健证明. |
| Testability | 5/10 | 便宜 pilot 具体可执行 (1 季节 × 1 区域, ±环境 ConvLSTM, CSI/FSS/POD/FAR), 值得肯定; 但作为证伪器欠功效且逻辑歧义 — 单区单季 null 无法区分 "数据无信息/融合失败/任务选错", 且把 ESSD 数据产品可行性与 0-2h 环境增量过度耦合 (后者按 Leinonen 2022 先验最可能为 null). 需预注册最小实际效应、独立事件数与历史长度消融. |
| Outcome realism | 4/10 | 单区单季 pilot 数周内现实; 但 "-15d..+5d 全量环境场 + 六任务标签 + 三种 splits + 两层 baselines + DOI/QC/license/provenance" 在数月内整体交付不可信; 按事件复制长窗 3D 场存储失控, 需去重资产层 + derived parameters 才现实. |
| Contribution type compliance | 10/10 | idea types {benchmark, application} ⊆ preferred {benchmark, application, empirical-finding}: yes |
| Overall Quality | 5/10 | 方向有价值且 venue 对口, 但任务-数据源闭合、无泄漏时间语义、负例、覆盖率、体量五个硬缺口未闭合, 当前版本不是 proposal-ready. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A (v1). Hard cap triggered: no.

## Alternative Framing

收窄为一个 availability-aware、leakage-audited 的 storm-environment 数据产品 (ESSD 主线): 核心贡献 = 事件索引 + 覆盖/质量审计 + provenance + 标签不确定性 + 去重资产层; 环境模态首选 HRRR f00/f01 派生探空参数时间序列 (CAPE/CIN/SRH/STP 等, hourly, 物理可辩护的 -72h..+24h 窗, 仅土壤湿度等 land-state 异常保留更长回溯), 而非全 3D 场; 示范分析聚焦一个明确的 evaluation question — "环境增量随 lead time (0-2h vs 2-6h)/区域/季节如何变化" (empirical finding, 在 preferred types 内), 以 CI/pre-convective 分类为旗舰环境敏感任务而非 0-2h 反射率 nowcasting. 该框架同时服务 ESSD 与 NeurIPS E&D, 不引入越界贡献类型.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅可声称: 在预注册任务、模型族、issue-time-safe 输入与 held-out 年份/区域上, HRRR 环境上下文相对雷达-only 产生超过最小实际效应的增量预测信息; 不可声称因果作用、普适提升或六任务全部受益. |
| NULL | 仅可声称: 当前样本量/模型/lead-time 下未检出超阈值增量; 数据产品的 ESSD 价值 (过程诊断/评估底座/复用) 独立成立, 但必须放弃 "环境场带来预报增量" 的卖点表述. 当前 idea 把 null 一律记为失败, 与 ESSD 路径不符, 需解耦. |
| NEGATIVE | 环境输入在多个 held-out split 稳定劣化 → 仅可声称所测环境表示/融合方案不稳健; 覆盖/license/体量失控 → 仅可判定当前发布规格不可行, 不否定研究问题本身. |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: Likelihood = Medium — ESSD 数据描述路径明确且以工程执行为主, 数据源全部公开可得; 但成稿依赖若干条件: 事件/负例覆盖充足 (Storm250-L2 撤稿证明该风险真实)、warning 源补齐、issue-time 泄漏语义闭合、license 矩阵干净、体量收缩到可发布规格; 若走 NeurIPS E&D 还需环境增量成立 (短 lead 有先验负面证据). Impact = High — 最乐观情形下成为被社区采用的环境-雷达-预警统一开放基准, 统一并加速一条明确的强对流 ML 研究线 (env-aware nowcasting/warning), 并为姊妹 idea 提供数据底座; 但不引入方法/理论、不推翻领域共识, 未达 Exceptional.

## Overall

- Priority: High
- Score: 7
- Comments: 有潜力做成强 ESSD 数据产品的 benchmark 方向, 组合层面 gap 经核验成立; 但 v1 的绝对 gap 主张被 MYRORSS 证伪, 且 warning 标签源缺失、负例设计、issue-time 泄漏、事件覆盖率、存储体量五个硬缺口需在 refine 中闭合. Claude 与 codex 双轴判断一致 (Medium × High → 7), 无 >= 1 level 分歧; 未触发 contribution-scope hard cap. 另注: frontmatter 指向的 workspace/scs-env-benchmark/ 目录尚不存在.

</review>

## Deep-lit 精读关键发现 [dispatcher, topic-scope, 2026-07-14, 累计 54 篇完整整合]

对照 2026-07-13 review 提出的 6 个硬缺口逐条核对本轮 topic-scope deep-lit（54 篇, 详见 `topics/0713-scs-benchmark-landscape.md`）能提供什么, 结论: **3 个缺口有文献级证据/可移植设计可直接闭合, 3 个缺口仍完全开放, refine 时不要误以为已解决**。

### 已获文献支撑、可在 refine 直接采用

- **缺口 (v) pilot 设计缺陷（二元对比不能证明"长历史"价值）**：`2512.08974`（FuXi-Nowcast）给出直接反驳材料——环境模态增量随 lead time 系统性增大（0–2h 小、2h+ 递增；去掉 3D 大气场变体在 dryline CI 个例上完全失败）。这精确定位了 Alternative Framing 引用的 Leinonen et al. 2022 负面先验的适用范围（该先验只在 0–60min 尺度成立）。**具体建议**：把 pilot 从"纯 MRMS vs +环境"单一二元对比改成 **0–2h / 2–6h / 6–12h 三档 lead-time sweep**，预期信号方向是短 lead 增量小、长 lead 增量显著——这既回应了 review 要求的三档消融，又让 pilot 同时具备"证实"和"证伪"两种可能结果，比原设计更具信息量。
- **缺口 (iv) HRRR f00/f01 issue-time/延迟约束未处理**：三篇独立来源给出可直接抄的处理范式——`2512.08974` 定量化了 train-on-analysis/infer-on-forecast 的 shift 幅度（CR@50dBZ 12h 掉 25.4%，且非均匀）并给出"选最近可用 cycle"的业务规则；`2601.17268`（Stormscope）明确把 ERA5(reanalysis)-train vs GFS(forecast)-infer 的切换记为需要在 provenance 显式声明的决策点；`2605.06944`（AIMIP）的 CO₂-as-clock 案例是同类"训练态/推断态不一致导致隐性泄漏"的独立佐证。**具体建议**：事件包 schema 必须给每个环境场字段一个 issue-time/product-type 元数据位（analysis=f00 / short-forecast=f01 / reanalysis），且 canonical task 定义中显式声明训练与评测各用哪一种，不能隐式假设两者一致。
- **缺口 (ii) 无负例/非事件采样设计**：`2605.01126`（EWB）的"zero-tornado + <10-hail 负样本日"判据与 `2401.16437`（TorNet）的三类采样（confirmed / 困难负样本 / random）+ Julian-day-mod-20 防泄漏 split 都是可直接移植的负例设计模板，不需要从零设计。

### 仍完全开放, refine 不能假装已解决

- **缺口 (i) NWS warning/CAP 档案源缺失**：本轮 54 篇里没有一篇提供"如何把 NWS CAP 预警档案接入训练数据管线"的现成方案——多篇（2603.20250 watch-to-warning、2508.06859 MeteorPred）都在用预警/watch 相关标签，但均为机构内部数据，不能解决我们的开放数据源问题。这个缺口需要独立的数据可得性调研（NWS CAP 历史档案的公开程度、格式、许可），deep-lit 无法替代。
- **缺口 (iii) 时窗设计缺物理论证（-15d 无依据、+5d 后视泄漏）**：`2004.11636`（idealized sounding physics）证明的是"垂直结构瞬时敏感性"，时间尺度是小时级 CM1 模拟，不能背书 -15 天这个具体数字；`2310.11631`（未来 SCS 环境变化）是气候变化尺度，同样不支撑。**没有一篇论文为"事件前 15 天"这个具体窗口长度提供物理依据**——refine 时要么找到能证明该窗口的文献（如边界层土壤湿度记忆时间尺度的研究），要么把窗口长度改为可辩护的更短值（如 idea review 建议的 -72h），不能沿用 v1 的 -15d 不做修正。
- **缺口 (vi) "事件聚类区域"从未被定义**：没有一篇 deep-lit 论文直接定义我们需要的"事件聚类区域"算法, 但有可移植的候选算法可供选择——`2012.00679` 的两遍 enhanced watershed（大面积阈值+小面积阈值+实心度 QC 迭代）和 `2310.03349`(AgentCaster) 的 PPF (KDE→盘卷积→Poisson) 均可用于把离散 Storm Events 报告聚合成连续空间区域，但选哪个、参数怎么调仍需 refine 阶段自行决定并验证，不是文献自动给出的答案。

**结论**：本轮 deep-lit 把 review 六个硬缺口中的三个（ii/iv/v）从"无方向"推进到"有可执行设计", 但 (i)/(iii)/(vi) 仍需 refine 阶段独立解决, 不能靠更多文献调研自动闭合——建议下一轮 refine 优先处理这三个, 而不是继续开新的 deep-lit round。

## Deep-lit 缺口追读 [dispatcher, idea-scope, 2026-07-14, 2 次 resume 完成]

针对上面标记为"仍完全开放"的三个缺口 (i)(iii)(vi) 做定向追读（非全 topic 扫描）。本轮共精读 4 篇全文（2102.08523、2412.03049 为 resume 1 派发, 2506.13939、2505.10850 为 resume 2 派发, 均已 wiki_written 并完成 dispatcher 级 references+cited+author+title-term 四项反向扩展）, 另有约 10 篇 DOI-only（无 arXiv 镜像）论文经 arxiv-tool search/S2 核验元数据后作引用级证据采纳。结论：**(iii) 已饱和闭合, (i) 已通过非文献渠道解决, (vi) 从零选型依据推进到候选与评测方法论充分, 选型决策留给 refine 阶段**。

### 缺口 (iii): -15 天前置窗口物理论证 — 饱和, 结论: 无支撑证据, 应放弃 15 天设计

4 篇全文精读的时间尺度形成清晰断层, 无一支持 15 天：

| arxiv_id | 覆盖尺度 | 结论 |
|---|---|---|
| 2102.08523 | 6-32 小时（起报提前量） | 南非 CI 个例：高分辨率土壤湿度初始场效力在此尺度已显著衰减/被降水反馈污染 |
| 2412.03049 | 季节内（JJA）+ 年际/30 年滑动 | GLACE 耦合方差分解：无日-周尺度记忆分析 |
| 2506.13939 | 严格同日（晨→午后-午夜） | Great Plains HDMR 功能分解：晨间土壤湿度解释午后降水方差最高 40%（GLACE 经典估计~16%的两倍以上）,但作者刻意剔除前 24h 降水样本以排除天气尺度混杂, Discussion 明确承认更长期记忆是未验证方向 |

**引用级交叉验证**（经 arxiv-tool S2 references/search 核验元数据, DOI-only 无 arXiv 镜像, 未获全文）：
- Rahmati et al. 2024, *Reviews of Geophysics*, "Soil Moisture Memory: State-of-the-Art and the Way Forward"（DOI:10.1029/2023RG000828, 被引 92）— 该领域现行权威综述, 未见任何两周尺度物理机制表述。
- Findell et al. 2024, *GMD*（DOI:10.5194/gmd-17-1869-2024）— 明确将陆气相互作用按时间尺度分层："minutes (turbulent fluxes), hours (BL growth/decay), **days (soil moisture memory)**, and seasons (behavioral regime variability)"——行业标准文献把土壤湿度记忆明确归类为"天"尺度, 而非两周。
- Maruf & Kumar 2026, *JHM*（DOI:10.1175/jhm-d-25-0062.1）— CLM5 实验测土壤湿度去相关时标敏感性, 量级仍为天到周。
- Tavakoli & Dirmeyer 2026, *Nature Sci. Data*（DOI:10.1038/s41597-026-07519-2）— 2412.03049 reader 建议追读的 Dirmeyer 一脉最新工作（非 2011 年那篇, 是 2026 年新数据集论文）, 全球观测陆气耦合指标数据集含 SM memory 一项。
- Taylor/Klein/Harris 2024 "Multiday Soil Moisture Persistence and Atmospheric Predictability Resulting From Sahelian Mesoscale Convective Systems"（经 2506.13939 引文列表 #3 命中, 标题直击"多日持续性", 但为西非季风区 MCS 而非美国 SCS 环境, 未获全文）; 2308.15196（印度季风核心区个例, 次表层土壤湿度记忆约 2 倍表层, 佐证记忆时标随深度变化但仍非"15 天"量级证据）。

**饱和判定**：4 篇全文精读 + 1 篇权威综述 + 1 篇行业标准时标分层文献, 跨南非/美国大平原/西非萨赫勒/印度季风区/全球尺度, 无一篇支持"事件前 15 天"这一具体窗口长度；现有定量文献的记忆时标要么是小时到天（个例/理论）,要么是季节到年际（诊断）, 中间"1-2 周"区间在 SCS 相关文献中是空白而非有争议或被否定——即找不到反驳"15 天无据"的证据, 也找不到支持"15 天有据"的证据, 与 review 原判断完全一致且现已交叉验证到饱和。

**建议 refine 阶段**：采纳 review 已给出的 Alternative Framing, 把窗口收窄到可辩护的 -72h（土壤湿度/边界层类环境特征）,仅对土壤湿度等 land-state 异常量保留更长回溯（回溯天数需单独论证, 但不再是"15 天"这个具体数字）；若坚持长窗口, 需把论证主体从"土壤湿度记忆"转移到确有多日至季节尺度机制的变量（如大尺度环流型持续性、ENSO 调制——这些有成熟的 1-2 周至季节尺度可预报性文献, 但是完全不同的论证链, 不能与土壤湿度记忆混用）。

### 缺口 (i): NWS warning/CAP 档案可得性/格式/许可 — 已通过 web 搜索解决, 非文献问题

4 组 arxiv-tool 搜索验证了一个负面事实：S2/arXiv 索引里几乎没有专门讨论"NWS CAP 历史档案接入训练管线"的论文, 命中的多为无关领域噪声——这个"零命中"本身有信息量：答案不在论文里, 在数据基础设施文档里。改用 B0 web 搜索直接命中权威答案：

- **Iowa Environmental Mesonet (IEM)**, `mesonet.agron.iastate.edu`（Iowa State University 运营）：提供 NWS VTEC-enabled watch/warning/advisory 事件的处理后 metadata + polygon 几何图形下载接口, 格式含 shapefile+CSV（zip）、KML、Excel。覆盖范围：多数 WWA 类型回溯至 2008 或 2005 年, Flash Flood Warning 至约 2002 年, **Tornado/Severe Thunderstorm Warning 回溯至 1986 年**；自 2005-11-12 起无已知缺口。Storm-based warning polygon 2002 年试验性引入, **2007-10-01 起对部分 VTEC 类型转为正式官方警报**——这个日期是重要的数据集设计断点, 事件包 schema 需显式处理 2007 前后 warning 产品形态不同。
- **许可**：NWS 网页信息属公共领域, 可免费用于任何合法目的, 限制仅为不可声称己有/不可暗示 NOAA 背书/不可修改后包装成官方材料——与 landscape 已有 NOAA/NASA 许可结论一致, 不引入新许可复杂度。
- **权威源**：NCEI 的 Service Records Retention System (SRRS) 存档 NWS 观测/预报/警报/公告, 国会要求至少保留 5 年——可作为 IEM 处理版之外的第二数据源, 用于交叉验证或补 IEM 缺口。

**结论**：缺口 (i) 已解决, 不需要再分配文献搜索预算。数据模型可加 `warning_ids`/`report_ids` 字段直接对接 IEM 的 VTEC 下载接口；时窗设计需注意 2007-10-01 polygon 官方化断点；provenance 文档需同时记录 IEM（处理层）与 NCEI SRRS（官方存档层）两个源。

### 缺口 (vi): "事件聚类区域"算法选型依据 — 候选从 2 个推进到 6+ 个, 选型留给 refine

已知两候选（enhanced watershed 2012.00679、AgentCaster PPF 2310.03349）彼此独立、无引用关系。本轮新增：

| 来源 | 类型 | 关键信息 |
|---|---|---|
| 2505.10850（全文精读） | 拓扑追踪（merge tree + partial optimal transport） | 第三种独立候选方法；更重要的贡献是**评测方法论**——无 ground truth 场景下用轨迹时长分布、沿轨迹物理量标准差、轨迹线性度损失、锚点匹配距离诊断这套代理指标比较追踪器, 可直接套用于比较 watershed vs PPF vs 其他候选；但其"轨迹线性度高=好"假设与强对流离散传播、快速非线性增强特征相反, 不建议直接迁移方法本体 |
| tobac v1.5（Sokolowsky/Freeman/Jones 2024, *GMD*, DOI-only 无 arXiv） | 社区标准工具 | Python 开源包, 40+ 同行评审论文使用, 支持任意变量/任意网格追踪（雷达/卫星/模式输出均可）, v1.5 新增 3D 追踪与 split/merge 支持——不想自研算法时最有社区背书的现成选项 |
| PyFLEXTRKR（Feng et al. 2023, *GMD*, DOI:10.5194/gmd-16-2753-2023, 被引 90） | 专为对流云设计的工具 | 显式处理 merge/split, 已用于 LES（~100m）到中尺度（~10km）多分辨率对流云追踪——比 tobac 更贴近"强对流风暴对象"场景, 是 2505.10850 reader 建议的下一步候选 |
| **MCSMIP**（Feng/Prein/Kukulies et al. 2025, *JGR*, DOI:10.1029/2024JD042204, 被引 26） | **追踪器互比基准** | 对 DYAMOND 模拟应用 **10 种不同 feature tracker**, 发现观测 MCS 频率上不同追踪器间差异达 2-3 倍——选型依据最直接的证据来源：追踪器选择对下游统计结果影响巨大, 不能假设"随便选一个都差不多" |
| Zan et al. 2019（*Atmos. Res.*, DOI:10.1016/j.atmosres.2018.12.007, 被引 20） | 雷达专用对比研究 | "Solving the storm split-merge problem"——测试对比多种既有风暴识别追踪方法后提出 CSIT 算法, 30dBZ 阈值+连通域标记, 6 年暖季雷达数据验证 |
| Muñoz et al. 2018（*Atmos. Res.*, DOI:10.1016/j.atmosres.2017.10.027, 被引 50） | 对流雨暴专用 | "Enhanced object-based tracking algorithm for convective rain storms and cells"——标题直接匹配我们场景, 是 enhanced watershed 之外另一个"enhanced"系族候选 |

**结论（候选已充分, 选型是 refine 阶段工程决策而非文献缺口）**：候选从 2 个扩展到至少 6 个独立方法/工具, 且拿到了可直接套用的评测方法论（2505.10850 代理指标）和选型依据的实证基础（MCSMIP 证明追踪器选择对结果有 2-3 倍量级影响）。这个缺口不能再靠更多文献调研自动闭合。Refine 阶段建议：(a) 若选现成工具, PyFLEXTRKR 因专为对流云/多分辨率设计, 是比 tobac 更贴近场景的默认选项；(b) 无论选哪个, 都应参照 MCSMIP 思路记录追踪器选择对下游任务（nowcasting/CI 检测/事件计数）的敏感性, 而非隐式假设算法无关；(c) tobac/PyFLEXTRKR/Zan2019/Muñoz2018 均为 DOI-only 无 arXiv 镜像（Elsevier *Atmos. Res.*／Copernicus *GMD*／AGU *JGR*）, 后续如需更多细节需通过期刊官网/机构访问获取全文, arxiv-tool 无法进一步深读。

### 元发现：severe-storm 算法与数据基础设施文献系统性不在 arXiv

gap (i) 和 gap (vi) 的搜索反复观察到同一模式：S2 429 限流触发 arXiv fallback 后, 返回结果几乎全部是无关领域噪声（医学影像、天文归档、空间天气、生物追踪算法等）,真正相关的结果几乎全部来自 S2 索引的 Elsevier *Atmospheric Research*、Copernicus *GMD*、AGU *JGR*/*Reviews of Geophysics* 等期刊, 且多数无 arXiv 镜像。这与 gap (iii) 的模式一致（Rahmati 2024 综述、Findell 2024 等也是 DOI-only）。**后续启示**：本 topic 若还要在"数据源工程细节"或"陆气耦合物理机制"这两类问题上做文献调研, 应预期 arXiv 覆盖率低, 优先用 B0 web 搜索或直接查 S2/期刊数据库, 不要依赖 arxiv-tool 的 arXiv fallback。

**新增论文清单（供上层 dispatcher 并入 topics/0713-scs-benchmark-landscape.md, 本 idea-scope 未直接写 landscape）**：2506.13939、2505.10850（均全文精读, wiki_written）；引用级/DOI-only（未获全文, 仅元数据核验）：Rahmati et al. 2024 (DOI:10.1029/2023RG000828)、Findell et al. 2024 (DOI:10.5194/gmd-17-1869-2024)、Maruf & Kumar 2026 (DOI:10.1175/jhm-d-25-0062.1)、Tavakoli & Dirmeyer 2026 (DOI:10.1038/s41597-026-07519-2)、tobac v1.5 (Sokolowsky/Freeman/Jones 2024, GMD)、PyFLEXTRKR (Feng et al. 2023, DOI:10.5194/gmd-16-2753-2023)、MCSMIP (Feng/Prein/Kukulies 2025, DOI:10.1029/2024JD042204)、Zan et al. 2019 (DOI:10.1016/j.atmosres.2018.12.007)、Muñoz et al. 2018 (DOI:10.1016/j.atmosres.2017.10.027)。resume-1 已读的 2102.08523、2412.03049 若上一轮尚未并入 landscape 的 54 篇清单, 亦请一并核对补入。
