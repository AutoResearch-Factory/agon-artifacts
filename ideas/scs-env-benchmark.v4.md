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

<review date="2026-07-16">

## Novelty

- Score: 4/10
- Closest prior work: **MYRORSS Storm Cluster Dataset for Tornado Detection and Climatology (2005-2011)** (Zenodo DOI:10.5281/zenodo.19644586, 2026-04-18 发布; Cui/Ortega/Williams; 本轮 codex 发现、经我 WebFetch Zenodo 记录页独立核验属实); 其次 FunnelCloud (2017)、TorNet (2024)、HR-Extreme (2024)、GridRad-Severe (MWR 2023)。
- Key differentiator: 新发现的 MYRORSS Storm Cluster Dataset 已发布 "CONUS 风暴对象簇 + NEXRAD 雷达参数 + RUC 环境特征 + Storm Events 多灾害标签 (tornado/hail/wind) + ML-ready 特征集 + CatBoost/RF baselines + DOI/版本 + CC-BY" —— v1-v3 反复依赖的宽松组合主张 ("多灾害雷达+环境+标签+baselines 的现代开放封装无人占坑") 自 2026-04 起**不再成立**, 这正是 v2 review 预警的 "MYRORSS 缺现代 ML 封装" 空缺被别人填掉。仍然成立的差异轴收窄为: (1) **原始时空演化场事件包** (HRRR 3km 逐时全场 + MRMS 2min 序列) vs 对象级标量特征表; (2) 现代 HRRR/MRMS 档案 vs 2006-2011 RUC 时代; (3) issue/valid/availability-time 可审计语义; (4) 长时窗环境演化 + 包内正常态对比这一等用途; (5) 泄漏审计的概率/nowcasting 评测协议与 canonical splits。方法零新颖 (v4 自己已诚实承认, FunnelCloud 已正确引用); 真正可能新颖的仍是 "issue-time-safe 环境增量随 lead time/区域/季节" 的 empirical finding, 尚未跑出。下一版 novelty 段必须显式引用并逐项对表该 Zenodo 产品, 否则按现表述投稿会被一击反驳。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | v3 三大问题两个真正闭合: (a) recluster-at-cutoff 代码经我逐行复读 `cluster_helpers.py:recluster_expansion` 并对照 `cluster_events_pilot_log.txt` 与 workspace commit 634e950 核验——每个 cutoff 先按时间过滤全表再从头重聚类, 全天聚类只用于选回顾锚点, 数字 (68x25/151x40/151x56 vs 215x58; 75% 时 33 现 27 连) 与代码逐行吻合, 修复真实; (b) 发布时滞 pushback 有据——固定网格概率任务与 MRMS 回波锚定 nowcasting 都不再让报告进入输入或空间定位, 报告降为事后标签, 这是回顾式 benchmark 的标准做法 (TorNet/HR-Extreme 同构), v3 "换近实时源" 的建议不再必要, 其合理内核 (输入按 issue time 截断) 已保留在 C1/C3/C4。(c) 窗口问题降级为诚实的设计约定: C2 明确否认 15 天物理记忆并把 H 节饱和证据与 "包内正常态采样" 问题区分开, 且把候选率审计列为最便宜证伪信号——这不再是静默绕行而是可证伪的工程主张; 残余缺口是 -15d 这个具体长度仍只是采样预算约定 (为何不是 -7d 或 -30d 无论证), 以及包内采样相对 "直接采独立正常日" 的边际价值 (同地配对对照) 未量化。**本轮最大的新缺口 (codex 提出, 我认可)**: 概率任务的评测总体未定义——confirmed/hard-negative/random 三类采样是 case-control 构造, 在其上算 Brier/BSS/可靠性并不能解释为真实场景概率, 除非显式给出时空抽样框与纳入概率/权重 (TorNet 对此有明确的 selection-probability 区分); "预定义区域日" 的 "预定义" 必须显式声明与报告无关 (固定 tile/行政区), 否则选择性泄漏从空间定义层回流。 |
| Missing evidence signals | 4/10 | Pilot 证据真实且自我报告诚实 (MIXED 恰当): 四源访问 4/4、超参敏感性 (1-17 簇)、ARI=0.206、真重聚类审计均可独立复现。但仍是单日单州: 30 日覆盖矩阵、逐源再分发条款、正常态候选产率 (本版新增的关键 gate 尚无一个数据点)、跨区域聚类稳定性、任何预测增量全部零证据; 另缺与 MYRORSS-2026 Zenodo 产品的字段级差异表, 以及概率任务的基率/纳入概率论证。 |
| Narrative | 6/10 | "一份数据契约三类一等用途" 比 v3 的双视图更干净, 且与 topic 的环境演化初衷对齐; novelty 表述诚实 (撤回 "未找到同构先例" 并引用 FunnelCloud)。弱点: 三用途仍像共享素材的三个任务而非一个科学问题; "组合空缺" 主线被 MYRORSS-2026 削弱后, 叙事重心需要移到 "特征表 vs 原始演化场 + issue-time 契约" 这条更硬的差异轴上。 |
| Venue contribution | 5/10 | ESSD 数据描述路径完好且 NULL 兼容; NeurIPS D&B (Evaluations & Datasets) 除 Croissant/RAI (已列入交付) 外还要求明确 "该数据集揭示什么评测假设/失效模式", 且现在必须回答 "相对 2026-04 的 MYRORSS Storm Cluster Dataset 增量何在"——概念上有答案 (原始场/现代档案/issue-time), 但须写成显式对表并用 baseline 差异证明。 |
| Testability | 7/10 | 最便宜证伪信号具体且真便宜: 30 日审计的四源交集/可再分发性 + 正常态候选产率, 都在 3-4 周审计内可命中; recluster 审计已给出可复现模板。缺数值门槛: 候选产率多少算 "足够"、预注册最小实际效应未定; 对 "相对 MYRORSS-2026 的贡献是否成立" 尚无便宜测试 (一张字段对比表即可, 应前移)。 |
| Outcome realism | 6/10 | 6-12 个月与 300-800 GPU-hours 比 v2/v3 的 3-6 个月诚实; 先 gate 后下载顺序正确。但多 TB 存储 + 多年四源版本协调 + 三类样本 + 三示范用途 + 许可/QC/DOI 全部 12 个月内完成仍偏乐观; 收缩为 "数据产品 + 一条旗舰评测" 更现实, 三任务齐全更像 12-18 个月。 |
| Contribution type compliance | 10/10 | idea types `{benchmark, application, empirical-finding}` ⊆ preferred-contribution-types `{benchmark, application, empirical-finding}`: yes |
| Overall Quality | 6/10 | 七维平均 6.1。v4 是三轮以来第一个把上版全部硬伤真正闭合或有据顶回的版本 (recluster 代码诚实化、发布时滞 pushback 成立、窗口主张可证伪化、pilot 降级 MIXED、effort 上调), 但 MYRORSS-2026 Zenodo 产品的出现收窄了 novelty 空间, 且概率任务的抽样框/基率问题是新暴露的设计级缺口。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} contribution types: `{benchmark, application, empirical-finding}`
- v_n contribution types: `{benchmark, application, empirical-finding}`
- Status: unchanged
- Hard cap triggered: no (类型集合未变; 无越界扩张; 无 method/theory 删除, topic 亦未声明这两类)

## 上版 concern 逐条判定 (v3 review → v4)

1. FunnelCloud 必须引用 — **resolved** (Novelty quick-check 与 MANIFEST gate 2 均已引用并正确限定)。
2. 15 天窗口静默绕行 H 节饱和证据 — **partially resolved, pushback 有据**: v4 显式引用并反驳 (物理记忆问题 ≠ 正常态采样问题), C2 明确禁止记忆/因果主张, 并给出候选率审计作证伪器; 未采纳 -72h 收窄有据可依, 接受。残余: 15 天具体长度仍无独立性/成本论证。
3. leakage ablation 代码不诚实 — **resolved** (真重聚类, 代码/日志/commit 三方核验一致)。
4. Storm Events 75-90 天时滞使预测视图不成立 — **resolved by redesign + 有据 pushback**: 报告彻底退出输入与空间定位, 回顾训练/验证用事后标签成立; "换近实时源" 不再必要。
5. Pilot 过度记为 POSITIVE — **resolved** (MIXED, 未测项逐条列出)。
6. Effort 与 v2 脱节、D&B 元数据缺失 — **resolved** (6-12 月、300-800 GPU-h、Croissant/RAI 入列)。

## Alternative Framing

保留三用途契约但换差异化主轴: 把 "对象特征表 vs 原始演化场" 作为旗舰对比——显式以 MYRORSS Storm Cluster Dataset (2026) 为特征级 baseline, 证明同一任务上原始 HRRR/MRMS 演化场 + issue-time 契约给出特征表给不出的增量与诊断; 同时给概率任务补一份显式抽样框契约 (全时空格点-日总体 + 纳入概率/权重), 使 Brier/可靠性可解释为真实基率下的概率。仍严格落在 `{benchmark, application, empirical-finding}` 内。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅在预注册的代表性时空 cohort (显式纳入概率)、held-out year/region、最小实际效应门槛下, 才能声称 issue-time-safe 环境输入相对气候学/MRMS-only 有可复现增量; 演化分析仅可作跨事件稳健的描述性结论。不得声称组合层面 "首次" (MYRORSS-2026/FunnelCloud 在先)、15 天物理记忆、因果作用或业务可部署性。 |
| NULL | 数据产品仍可发布 (ESSD 路径); 只可称当前任务/模型/功效未检出增量, 不可称环境无信息; 功效充分的 NULL 是一条克制的 empirical finding。 |
| NEGATIVE | 覆盖/许可/正常态候选率任一失败仅否定当前发布规格; 代表性 cohort 无法构造则概率/校准主张必须撤回而非改用事件富集样本继续报; 可选 0-6h 报告预警或 3D 核失败只删对应任务。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: Likelihood = Medium——ESSD-first 成稿路径清楚, pilot 证据真实, v3 全部硬伤已闭合或有据顶回, 主要风险回到工程审计 (覆盖/许可/产率) 与两个可修复的设计缺口 (抽样框契约、对表 MYRORSS-2026), 均有明确修复路径但需多个条件同时成立, 且 6-12 月工期不短。Impact = High——若交付原始演化场级、issue-time 可审计、基率可解释的现代 HRRR-MRMS 开放基准并坐实环境增量异质性, 会实质影响 env-aware nowcasting/warning 研究线并为姊妹 idea 提供数据底座; 但 MYRORSS-2026/FunnelCloud/TorNet 占据多个组件层面在先位置, 且限于美国, 不达 Exceptional。查表 Medium × High = High (7)。Claude 与 codex 在 Likelihood (Medium)、Impact (High)、Priority (High/7)、hard cap (no) 上完全一致, 无 >= 1 level 分歧; codex 独立发现的 MYRORSS Storm Cluster Dataset (Zenodo) 经我 WebFetch 记录页独立核验属实并已 append 到 landscape。

## Overall

- Priority: High
- Score: 7
- Comments: v4 是本 idea 三轮迭代中质量提升最实的一版: v3 的三个核心指控 (窗口绕行、审计代码造假式截前缀、时滞不兼容) 分别被可证伪化、真实修复、有据顶回, 全部经我独立读代码/日志/commit 核验成立。但 novelty 端出现实质收窄——codex 发现且我核验属实的 MYRORSS Storm Cluster Dataset (Zenodo, 2026-04) 已占据宽松组合主张, 下一版必须显式对表并把差异化重心移到 "原始演化场 + issue-time 契约 + 基率可解释评测" 这条硬轴, 同时补概率任务的抽样框契约。分数维持 7 (Medium × High): 质量修复与 novelty 收窄大体相抵, 方向依然值得推进但尚未 proposal-ready。文件 7.2 KB, 未超限。

</review>
