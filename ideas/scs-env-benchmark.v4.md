<!-- 书写报告使用中文 -->
---
topic: topics/0713-scs-benchmark.md
landscape: topics/0713-scs-benchmark-landscape.md
workspace: workspace/scs-env-benchmark/
---

- One-sentence summary: 发布开放、版本化的强对流环境-雷达事件包和统一数据契约, 同时支持环境演化分析、环境驱动的事件概率预报、以及由 MRMS 回波锚定的环境约束 nowcasting/预警研究。
- Hypothesis: 同一份 issue-time 可审计的 HRRR/MRMS/Storm Events 数据产品可支撑三类一等用途: (1) 比较候选正常背景与事件前、中、后环境演化及其和雷达的关系; (2) 在固定网格或预定义区域日上, 用环境输入预测事后事件标签概率; (3) 用 cutoff 前 MRMS 回波移动作为空间锚点, 检验环境约束对 0-2h/2-6h nowcasting 的增量。事件标签用于训练/验证目标而非模型输入; 0-6h 事件预警标签和 3D 风暴核参考均须实验后才纳入正式任务。
- Expected outcome: 发布 DOI/版本、去重源资产索引、provenance/QC/license、三类样本、canonical splits、Croissant/RAI 元数据和上述三类用例。预测增量为 NULL 仍可成立 ESSD 数据产品; 最便宜的失败信号是 30 日审计中四源交集或可再分发性不足, 或 -15d/+5d 包内找不到足够的无报告且弱/无对流回波时段作为正常态候选。
- Contribution type: benchmark+application+empirical-finding
- Contribution drift note: 与 v3 一致, 且仍是 topic 允许集合的子集。
- Risk: MEDIUM
- Estimated effort:
  - Compute: 整编以 CPU、网络和多 TB 存储为主; 两类轻量 baseline 约 300-800 GPU-hours, 全量 nowcasting 预算待 30 日体量审计。
  - Data: available but needs collection; 单日四源可访问, 覆盖、缺测、正常态候选率和逐源再分发条款待审计。
  - Implementation: 30 日覆盖/聚类/窗口审计 3-4 周; 可发布数据产品、三类示范和论文约 6-12 个月。
- Novelty quick-check: FunnelCloud (2017, DOI:10.1080/17538947.2017.1279235) 已直接把龙卷报告做 ST-DBSCAN 并关联 NARR 与 NEXRAD; MYRORSS 也已发布雷达和近风暴环境。故撤回“未找到同构先例”。差异只主张组合层面: 多灾害、开放 DOI/版本、输入 issue-time 语义、负例、canonical splits/baselines 和现代 ML-ready 封装, 而非新聚类方法。
- Strongest objection: 这可能只是昂贵的公开源拼接; Storm Events 标签有偏且滞后, 长窗成本高, 两类预测任务和正常态对比都尚未用跨年份证据证明可用。
- Why we should do this: 现有团队仍须各自重做环境、雷达和标签的对齐与泄漏审计。一个任务边界清楚的数据契约能同时服务预测研究和环境演化研究, 且不把事后标签误装成实时输入。
- Pilot:
  - Setup: 复用 2024-05-06 Oklahoma 的 114 条真实报告和四源访问检查; 在每个 first/25%/50%/75% cutoff 先过滤全表再重新聚类, 最终簇只用于事后匹配锚点。
  - Metric: 复现簇数/ARI; 比较每个 cutoff 重聚类区域的成员和 bbox, 并把外部有效性、覆盖与预测增量明确留作未测项。
  - Result: 四源 4/4 PASS; 1-17 簇和默认 ARI=0.206 复现。最大簇在 25/50/75% 为 68x25/151x40/151x56km, 完整回顾区域 215x58km; 75% 时 33 个最终成员已出现, 仅 27 个连到当时锚点, 证明旧版“全量聚类后截前缀”不等价。
  - Signal: MIXED(访问、可执行性、敏感性和回顾区域扩张为正; 30 日稳定性、外部有效性、许可覆盖及预测价值未测)。

- Claims and Claims matrix:
  - C1 数据: 样本可追溯到源文件、处理版本及 HRRR/MRMS 的 valid/issue/availability time; 报告发布时滞作为标签 provenance 保存。
  - C2 分析: -15d/+5d 是采样范围, 用来寻找包内正常态候选并比较前中后演化, 不是 15 天物理记忆或因果主张; “正常”还须无匹配报告且弱/无对流 MRMS 信号。
  - C3 概率预报: 固定网格/预定义区域日上的环境输入不含报告衍生 crop、特征或索引; Storm Events 只在事后生成目标, 以 Brier/BSS、PR-AUC 和可靠性评估。
  - C4 nowcasting: 空间锚点来自 cutoff 前可观测的 MRMS 回波; 比较 MRMS-only 与 +environment。事件标签预警和 3D 核跟踪仅在 pilot 支持后成为 claim。

  | Outcome | 允许的 claim |
  |---|---|
  | POSITIVE | held-out year/region 上, 相对气候学或 MRMS-only 的预注册增量超过实际效应门槛; 描述性演化跨事件稳健。 |
  | NULL | 数据产品仍可发布; 只称当前任务/模型未检出增量, 不称环境无信息。 |
  | NEGATIVE | 覆盖/license/正常态候选率失败否定当前规格; 可选标签预警或 3D 核失败只删除对应任务。 |

- Narrative: 论文按三类用途组织同一数据契约: 事后报告建立可审计事件档案, 固定空间支持环境概率预报, cutoff 前雷达锚定 nowcasting, 长窗提供同包正常态对比和演化分析。
- Experiments:
  1. 30 日多区域样本审计四源交集、许可、缺测、体量、三类样本及无报告+弱回波正常态候选率。
  2. 网格扫 ST-DBSCAN 参数并真正在各 cutoff 重聚类; 用 ARI、簇数/面积稳定性和人工审计冻结参数。报告簇只作回顾索引/分析。
  3. 做正常态-事件前中后环境合成及与 MRMS 强度、结构、移动的关系分析; 按季节/区域匹配并报告标签不确定性。
  4. 概率任务用固定网格或预定义区域日, 环境-only 对比气候学/逻辑回归, held-out year/region, 禁止任何报告衍生输入。
  5. nowcasting 用 cutoff 前 MRMS 回波/轨迹 + issue-time-safe 环境, 对比 MRMS-only; 另行消融 0-6h 事件目标和 3D 核参考, 失败即删除可选项。
- Assets status: 四源单日检查通过, 真正 cutoff 重聚类 pilot 已重跑且为 MIXED; 全量下载等待 30 日 gates, 详见 `workspace/scs-env-benchmark/data/MANIFEST.md`。

- Review decisions:
  1. ACCEPT FunnelCloud 直接先例并收窄 novelty; ACCEPT ARI/五组参数证据弱, 故 pilot 改为 MIXED。
  2. ACCEPT v3 静默绕过 landscape H 节是缺陷; PUSHBACK -72h 结论。H 节正确回答了土壤/大气物理记忆尺度, 因而 v4 明确禁止 15 天记忆主张; 但窗口回答的是“能否在同包采到正常背景并分析演化”, 不是“影响能记忆多久”, 故保留 -15d/+5d 并用候选率审计证伪。
  3. ACCEPT 旧 cutoff 代码不诚实; 已改成先过滤全表再重聚类并重跑, 用途降为量化回顾区域随报告进展的形成过程。
  4. PUSHBACK Storm Events 75-90 天时滞使研究设计失效、必须换近实时源: 本阶段做历史训练/矫正, 事后观测作验证标签是成立的; 不作业务部署主张。保留其合理内核: 所有模型输入仍按 issue time 截断, 且空间锚点按任务定义。
  5. ACCEPT localization concern 的任务内核: 概率任务用固定空间且报告只作标签; nowcasting 用 cutoff 前 MRMS 回波锚定, 不用未来报告 crop。
  6. ACCEPT effort 与 v2 脱节、覆盖/license/NeurIPS 元数据证据不足; 已上调工期/算力并补 30 日 gate 与 Croissant/RAI 交付。
  7. PUSHBACK Alternative Framing 中换近实时源和收窄 -72h 两项, 理由同 2/4; 其余双视图、claims discipline 与 held-out 要求已按任务重写。
