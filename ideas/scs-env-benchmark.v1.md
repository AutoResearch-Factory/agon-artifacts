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
