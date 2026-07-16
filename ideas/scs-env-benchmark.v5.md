<!-- 书写报告使用中文 -->
---
topic: topics/0713-scs-benchmark.md
landscape: topics/0713-scs-benchmark-landscape.md
workspace: workspace/scs-env-benchmark/
---

- One-sentence summary: 发布开放、版本化的强对流环境-雷达事件包数据产品, 差异轴是三条经 15 近邻逐行审计确认的唯一属性: 事件对齐长窗的原始 HRRR/MRMS 演化场序列、作为交付物的事件前正常态时段、预警记录与实况标签分离的 issue-time 可审计语义; 同一契约支撑环境演化分析、环境驱动概率预报和雷达锚定 nowcasting。
- Hypothesis: 同一份 issue-time 可审计的 HRRR/MRMS/Storm Events/预警数据产品可支撑三类一等用途: (1) 比较包内正常背景与事件前中后环境演化及其与雷达的关系; (2) 在固定网格或与报告无关的预定义区域日上, 用环境输入预测事后事件标签概率; (3) 用 cutoff 前 MRMS 回波作空间锚, 检验环境约束对 0-2h/2-6h nowcasting 的增量。事件标签只作训练/验证目标; 0-6h 预警任务和 3D 风暴核参考须 pilot 支持后才纳入正式任务。
- Expected outcome: 交付 DOI/版本、去重源资产索引、provenance/QC/license、三类样本及其纳入概率/抽样权重、canonical splits、Croissant/RAI 元数据和三类示范用例; 预测增量为 NULL 时 ESSD 数据产品仍成立。最便宜的证伪信号: 30 日审计中四源交集或可再分发性不足, 或正常态候选产率不过数值 gate (<80% 的包达到 ≥48 候选小时)。
- Contribution type: benchmark+application+empirical-finding
- Contribution drift note: 与 v4 一致, 无增删, 仍是 topic preferred 集合的子集。
- Risk: MEDIUM
- Estimated effort:
  - Compute: 整编以 CPU、网络和多 TB 存储为主; 轻量 baseline 与数据种类消融约 300-800 GPU-hours; 全量 nowcasting 预算待 30 日审计。
  - Data: available but needs collection; 单日四源可访问, 覆盖/缺测/再分发条款待 30 日审计。
  - Implementation: 30 日审计 3-4 周; 交付分两期 — 数据产品 + 概率预报旗舰评测 + 演化分析约 6-12 个月, nowcasting 全量在第二期 (三任务全部完成约 12-18 个月)。
- Novelty quick-check: 15 个近邻数据集的交付物逐行审计见 landscape "对照数据集" 审计节 (2026-07-16), 三条属性在全表唯一: (a) 原始环境场的事件对齐长窗序列 — 近邻要么交付派生标量特征统计, 要么短窗/单时刻快照, 要么没有环境; (b) 事件前正常态时段作为交付物 — 最近先例 HR-Extreme (2409.18885) 只在论文内做过 ±15d 随机正常时刻对比, 未随数据发布; (c) issue-time 可审计语义, 预警与实况分离交付 — MeteorPred (2508.06859) 把预警当真值, 是反例。以下共享轴各有 ≥1 近邻占位, 不得单独作 novelty: 多灾害确认标签、风暴中心事件封装、HRRR 时代、MRMS 级融合雷达、"ML-ready+baseline+DOI" 组合。最近邻按序: HR-Extreme > MYRORSS Storm Cluster 2026 (Zenodo DOI:10.5281/zenodo.19644586) > GridRad-Severe; 聚类步骤以 FunnelCloud (2017, DOI:10.1080/17538947.2017.1279235) 为直接先例, 不主张聚类方法新颖。MYRORSS-2026 经变量清单下载核验, 交付的是 ~1060 列按风暴对象汇总的派生特征统计量 (雷达量与 RUC NSE 标量指数的 mean/max/std/百分位), 无格点场/廓线/环境时间序列/事件前窗口/正常态时段; 与它的区分是数据种类之别 (特征统计 vs 原始演化场), RUC/HRRR 时代差只是叠加项。
- Strongest objection: 三条唯一轴都是交付形态与语义层面的差异, 不是新任务; 若拿不出 "原始演化场给出特征统计表给不出的增量" 的实验证据, 它仍可能被读成昂贵的公开源拼接 (由实验 6 正面回应)。
- Why we should do this: 现在每个团队都各自重做环境、雷达、标签的对齐与泄漏审计; 一份任务边界清楚、正常态与预警语义显式的数据契约能同时服务预测研究和演化研究, 且不把事后标签误装成实时输入。
- Pilot:
  - Setup: 2024-05-06 Oklahoma 114 条真实报告: 四源访问检查、每个 cutoff 先过滤全表再重聚类的审计、以及 -15d/+5d 窗内 6 小时步长的正常态候选产率 smoke test (最大簇 bbox, 84 个 MRMS 时刻)。
  - Metric: 复现簇数/ARI; 各 cutoff 重聚类区域的成员与 bbox; 正常态候选 = bbox 内无 ±3h 匹配报告且最大反射率 <35 dBZ。
  - Result: 四源 4/4 PASS; 1-17 簇, ARI=0.206; 最大簇 25/50/75% 时 68x25/151x40/151x56 km, 全天 215x58 km, 75% 时 33 个最终成员仅 27 个连锚。正常态 smoke: 52/84 时点 (62%) 为候选, 其中 32 个落在 -15d..-2d, 折合约 312/504 小时, 单包超 gate 约 6 倍; 事件日 00-18Z 也读为正常 (对流 21Z 后才起)。
  - Signal: MIXED (访问、可执行性、参数敏感性、单包产率为正; 30 日稳定性、外部有效性、许可覆盖和一切预测价值未测)。

- Claims and Claims matrix:
  - C1 数据: 每个样本可追溯到源文件、处理版本及 HRRR/MRMS 的 valid/issue/availability time; 报告发布时滞作为标签 provenance 保存。
  - C2 分析: -15d/+5d 是采样窗, 不是 15 天物理记忆或因果主张; 长度由正常态采样预算和姊妹 topic 0710-causal-scs 的估计器预热深度反推, 并受产率 gate 证伪约束; "正常" 须同时满足无匹配报告和弱/无对流回波。
  - C3 概率预报: 环境输入不含任何报告衍生量; "预定义区域日" 只由固定地理/气候学定义 (固定 tile 或 CWA), 定义过程不接触 Storm Events; 评测要么在未抽样的固定时空框架 (held-out year/region 的全部格点-日) 上进行, 要么随三类样本发布纳入概率/抽样权重, 使 Brier 与可靠性图可解释 (先例: TorNet 的三类采样文档)。
  - C4 nowcasting: 空间锚点来自 cutoff 前可观测的 MRMS 回波; 比较 MRMS-only 与 +environment; 标签预警和 3D 核跟踪仅在 pilot 支持后成为 claim。
  - 预注册数值 gate: (i) 产率 gate — 30 日审计中 ≥80% 的包须含 ≥48 个候选正常小时, 其中 ≥24 个落在 -15d..-2d, 不过则当前窗口规格证伪; (ii) 最小实际效应 — 概率任务对气候学/逻辑回归 ΔBSS ≥ +0.02, nowcasting 对 MRMS-only ΔCSI ≥ +0.02, 且按日 block-bootstrap 的 95% CI 不含 0; 低于门槛只报告数字, 不作增量 claim。

  | Outcome | 允许的 claim |
  |---|---|
  | POSITIVE | held-out year/region 上超过预注册最小实际效应; 演化结论跨事件稳健。不得主张组合层面 "首次" (MYRORSS-2026/FunnelCloud 在先)、15 天记忆、因果作用或业务可部署性。 |
  | NULL | 数据产品仍发布 (ESSD 路径); 只称当前任务/模型/功效未检出增量; 功效充分的 NULL 是一条克制的 empirical finding。 |
  | NEGATIVE | 覆盖/许可/产率 gate 失败否定当前发布规格; 代表性时空框架构造不出则撤回概率/校准主张, 不改用事件富集样本继续报; 可选任务失败只删对应任务。 |

- Narrative: 论文主线是 "同一数据契约的三类一等用途, 外加特征统计与原始演化场的数据种类对比": 事后报告建立可审计事件档案, 固定空间支撑基率可解释的概率预报, cutoff 前雷达锚定 nowcasting, 长窗提供包内正常态对比和演化分析; 旗舰消融负责证明场序列本身的价值。
- Experiments:
  1. 30 日多区域审计: 四源交集、许可、缺测、体量、三类样本, 并按小时粒度对照产率 gate (含 35 dBZ 阈值与 ±3h 缓冲的敏感性扫描)。
  2. ST-DBSCAN 参数网格 + 各 cutoff 重聚类审计后冻结参数; 报告簇只作回顾索引和分析。
  3. 正常态-事件前中后环境合成及其与 MRMS 强度、结构、移动的关系; 按季节/区域匹配并报告标签不确定性。
  4. 概率任务: 固定网格或报告无关区域日, 环境-only 对比气候学/逻辑回归, held-out year/region, 按 C3 抽样契约评测; lead time × 区域 × 季节扫描按 ΔBSS gate 判定。
  5. nowcasting: cutoff 前 MRMS 回波/轨迹 + issue-time-safe 环境, 对比 MRMS-only; 0-6h 事件目标与 3D 核为 gated 选项, 失败即删。
  6. 数据种类旗舰消融: 同一任务上把包内 HRRR 变量压缩成 MYRORSS-2026 式对象级统计特征表, 对比原始演化场序列输入, 用增量差异支撑唯一轴 (a); 不跨时代直接对比 2006-2011 RUC 对象数据。
- Assets status: 四源单日检查、cutoff 重聚类审计、正常态产率 smoke 均已完成 (后者单包过 gate), 全量下载等 30 日 gates; 细节见 `workspace/scs-env-benchmark/data/MANIFEST.md`。

- Review decisions (v4 review → v5):
  1. ACCEPT: MYRORSS-2026 须显式对表 — landscape 审计节已完成 15 行对照, novelty 段重写为三条经审计唯一轴。并修正 v4 review 的 "RUC environment features" 表述: 经变量清单核验, 它交付的是对象级派生特征统计, 不是场意义的环境数据。
  2. ACCEPT: 概率任务抽样框缺口 — C3 增加抽样契约: 未抽样固定框架评测, 或发布纳入概率/权重。
  3. ACCEPT: "预定义区域日" 须与报告无关 — 已在 C3 显式声明。
  4. ACCEPT: 缺数值门槛 — 预注册产率 gate (80%/48h/24h) 和最小实际效应 (ΔBSS/ΔCSI ≥ +0.02); smoke test 已验证该度量可算且单包不失败。
  5. PARTIAL (-15d 长度论证): 窗口保留; 长度依据补为正常态采样预算 + 0710-causal-scs 估计器预热深度反推, 受产率 gate 约束; 物理记忆与因果主张继续禁止。
  6. ACCEPT (工期现实性): 交付分两期, 数据产品 + 概率旗舰评测 + 演化分析先行, nowcasting 全量第二期; 三任务均保留, 不删。
  7. ACCEPT (Alternative Framing 的可用内核): "特征表 vs 原始场" 对比以包内消融纳入 (实验 6); RUC 2006-2011 与 HRRR 不可比, 故不跨时代直接对表其数据。
