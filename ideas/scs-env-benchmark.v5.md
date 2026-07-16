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

<review date="2026-07-16">

## Novelty

- Score: 5/10
- Closest prior work: HR-Extreme (2409.18885, 2024); 最近的已发布数据产品对照: MYRORSS Storm Cluster Dataset (Zenodo DOI:10.5281/zenodo.19644586, 2026-04)
- Key differentiator: 三条审计唯一轴经本轮独立核验全部成立且表述忠实于审计表: (a) HR-Extreme 的 ±15d 正常时刻对比确为论文内验证 (随机 50 时间戳、距事件 ≥24h), 未随数据交付 — wiki/2409.18885.md 全文笔记核验; (b) MYRORSS-2026 经我 WebFetch Zenodo 记录页复核, 交付确是 ~1060 列对象级派生特征统计 (2006-2011, 无格点场/廓线/事件前窗/正常态/预警), v5 的 "特征统计 vs 原始演化场" 种类区分成立; (c) MeteorPred 以 CMA 预警为标签、零事件验证的反例成立 — wiki/2508.06859.md 核验。我的三组独立搜索 (severe convective benchmark / near-storm environment dataset / null-event negative-sample dataset) 未发现审计表之外的新竞品; UniExtreme (2508.01426) 已在 landscape, 2509.09195 为卫星亮温方法论文不相关。但需保持清醒: 三条轴全部是交付形态/整编语义层差异, 方法零新颖 (idea 已诚实声明, FunnelCloud 在先); HR-Extreme 的生成接口本可产任意窗长, 长窗轴严格说是 "严谨产品化" 而非新能力; "唯一" 必须永远限定为 "在已审计 15 近邻中" (v5 目前措辞已如此限定, 投稿时不得放宽)。真正可能的新颖性在尚未跑出的 empirical finding: 原始演化场相对 issue-time-safe 特征表的增量及其随 lead time/区域/季节的异质性 (实验 6+4)。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v4 的三个设计级缺口真正闭合: 抽样框契约 (C3 双轨)、区域日报告无关声明、预注册数值 gate 均落地且 gate 度量机器经真数据验证可跑。残余缺口 (均 proposal 级可修): (i) C3 仍是 "未抽样固定框架 或 发布权重" 两案并存, canonical 任务未冻结——总体、时空单元、lead time、纳入概率算法都待定 (codex 提出, 我认可); (ii) 重叠事件包共享原始资产与正常时段, 仅按 year/region 切分不必然防泄漏, split 规则需按包重叠显式定义 (codex 新发现, 有效); (iii) 实验 6 需锁定 issue-time-safe 特征基线与匹配容量/信息预算, 否则 "原始场胜特征表" 可能混杂容量/分辨率/事后对象定位; (iv) "正常态" 定义未区分静稳背景与临发对流时段——smoke 中事件日 00-18Z 全部计为正常候选 (18Z 距 21Z 起对流仅 3h), 对比 HR-Extreme 用 ≥24h 距事件间隔, 正常态合成若混入临发环境会稀释 axis (b) 的分析价值, 建议交付时加 distance-to-nearest-event 分层字段; (v) -15d 长度仍只有采样预算+姊妹预热的反推, 无窗长-产率-存储成本曲线。 |
| Missing evidence signals | 5/10 | Smoke test 经我全链路独立核验为真: 84 个真实 MRMS 文件在盘 (573MB); 反射率计算 5/5 复算吻合 (含临界值 34.8 dBZ); 报告匹配 6/6 复算吻合 (93 条 in-bbox、±3h 计数、CZ_TIMEZONE 修正对 OK 全为 CST-6 成立); log 数字与 idea 数字一致; 限制 (单包/6h 步长/最低仰角反射率) 在 MANIFEST 诚实记载。一处措辞需降温: "折合约 312/504 小时、超 gate 约 6 倍" 是 6 小时抽样比例外推, 不是逐小时实测, 直接可比数字只有 52/84; 外推当作裕量证据不成立 (codex 提出, 成立)。仍为零证据: 跨季节/区域 30 日矩阵、逐资产再分发条款 (尤其 IEM 几何层)、存储/托管成本、标签偏差量化、任何预测增量或消融结果。 |
| Narrative | 6/10 | "同一契约三类一等用途 + 数据种类旗舰消融" 比 v4 前进——实验 6 给了 "为什么要原始场" 一个可检验的脊柱。弱点: 三任务并列仍显 omnibus, 最强单主线其实是 "可审计环境轨迹产品是否改变相对特征表的评估结论", 演化分析与 nowcasting 更适合作为消费者/示范而非并列旗舰 (codex 与我一致)。 |
| Venue contribution | 6/10 | ESSD 路径完好且 NULL 兼容, 两期交付把重实验留给 D&B 路径的分工基本对齐 ESSD "数据描述为主" 的范围要求; 15 行审计表正是 D&B 审稿人要的对表。缺口: ESSD 级的开放仓储/永久托管/独立数值验证方案未落一字; NeurIPS Evaluations & Datasets 需要实验 6 的实际结果才能回答 "该资源如何改变评估实践"。 |
| Testability | 7/10 | 最便宜证伪信号真实且便宜: 30 日审计的四源交集/再分发/产率 gate, 数值门槛明确 (80%/48h/24h), 度量机器已被单包真数据验证可执行, 敏感性扫描 (35dBZ、±3h) 已列入 TODO。局限: 这些 gate 证伪的是发布规格而非核心价值主张 (后者要等实验 6); ΔBSS/ΔCSI ≥ +0.02 数值本身未按 lead time/空间尺度/业务意义论证, CSI 的阈值选择协议未定; 逐日 block-bootstrap 对重叠包与天气过程自相关的覆盖存疑。 |
| Outcome realism | 6/10 | 两期制 (数据产品+概率旗舰+演化 6-12 月, nowcasting 全量 12-18 月) 回应了 v4 的工期批评, 先 gate 后下载顺序正确。风险仍偏乐观处: 长窗原始场按 -15d/+5d × 多区域可能达数十 TB, 开放托管、出口带宽、长期版本维护未计价; 三任务全保留而非收缩为一条旗舰评测, 团队容量证据缺失 (codex 强调, 部分成立——两期制已部分缓解)。 |
| Contribution type compliance | 10/10 | idea types {benchmark, application, empirical-finding} ⊆ preferred-contribution-types {benchmark, application, empirical-finding}: yes |
| Overall Quality | 6/10 | 六维平均 6.0 (compliance 不计入)。v5 把 v4 提出的全部四个 ACCEPT 项真正落地, smoke test 是本系列第一个经第三方全链路复算验证的 pilot 证据; 剩余缺口集中在 canonical 任务冻结与 ESSD 工程配套, 属 proposal 阶段工作。 |

## Contribution Drift (n = 5)

- v4 contribution types: {benchmark, application, empirical-finding}
- v5 contribution types: {benchmark, application, empirical-finding}
- Status: unchanged
- Hard cap triggered: no (无越界扩张; 无 method/theory 删除; 类型集合与 topic preferred 完全一致)

## 上版 concern 逐条判定 (v4 review → v5)

1. MYRORSS-2026 显式对表 — **resolved**: landscape 15 行审计 + novelty 段重写为三条界定清楚的唯一轴, 且把 v4 review "RUC environment features" 的宽泛表述修正为经下载核验的 "对象级特征统计" (我经 Zenodo 记录页独立复核属实)。
2. 概率任务抽样框契约 — **partially resolved**: C3 原则正确 (未抽样固定框架 或 纳入概率/权重, TorNet 先例), 但两案未择一冻结, 无总体/权重实例; 属设计承诺而非完成。
3. 区域日与报告无关 — **resolved** (规格层面): 固定 tile/CWA、定义过程不接触 Storm Events 已显式写入 C3。
4. 预注册数值门槛 — **resolved**: 产率 gate (80%/48h/24h) + 最小实际效应 (ΔBSS/ΔCSI +0.02, block-bootstrap CI) 落地, 且 gate 度量已由真数据 smoke 验证可算; 新遗留: 阈值的业务论证与 "6 倍裕量" 的外推措辞。
5. -15d 长度独立论证 — **partially resolved, 遗留有据声明**: v5 自己标注 PARTIAL, 补了采样预算+预热深度反推并受产率 gate 约束; 仍缺窗长-成本曲线, 但作为声明诚实的设计约定可接受。
6. 工期收缩为 "数据产品+一条旗舰评测" — **partially resolved, pushback 部分有据**: 两期制采纳了分期内核且 12-18 月总期更诚实; 三任务全保留的容量证据仍缺, 但 nowcasting 后置已实质降低第一期风险, 接受。
7. Alternative framing (特征表 vs 原始场旗舰对比) — **resolved**: 实验 6 纳入, 且正确拒绝跨时代直接对比 RUC 数据。

## Alternative Framing

现框架已接近最锐 (v4 的替代框架已被采纳为实验 6)。进一步收敛方向 (不改变贡献类型): 把主贡献表述为 "带 availability-time 契约的环境轨迹数据产品 + 一条信息层级旗舰评测" (issue-time-safe 特征表 → 单时刻场 → 完整演化场 → +雷达, 在冻结总体上随 lead time/区域/季节比较), 演化分析与 nowcasting 降为消费者示范。这是叙事收敛而非重构, 可留给 proposal 定夺。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅在冻结的代表性总体、无重叠泄漏 split、availability-time 截断、容量匹配基线、预注册效应门槛全部满足时, 才可声称原始演化场输入相对特征表/气候学/MRMS-only 的可复现增量; 演化结论限跨事件稳健的描述性陈述。不得声称组合层面 "首次"、审计范围外的无条件唯一性、15 天记忆、因果作用或业务可部署性。v5 的 POSITIVE 行已含大部分限定, 达标。 |
| NULL | 数据产品照发 (ESSD); 只称当前任务/模型/功效未检出增量; 功效充分的 NULL 可反向支持 "特征统计已足够" 这一克制 empirical finding——此时不得再用预测增量论证长窗必要性, 长窗价值须退守演化分析用途。v5 的 NULL 行达标。 |
| NEGATIVE | 覆盖/许可/产率 gate 失败仅否定当前发布规格; 代表性框架构造不出则撤回概率/校准主张且不换事件富集样本续报; 可选任务失败只删对应任务; 实验 6 失败削弱 D&B 路径但不否定合格的 ESSD 产品。v5 的 NEGATIVE 行达标。 |

## Likelihood-Impact Matrix

- Priority: High = Likelihood: Medium x Impact: High
- Numeric score for ideas.xml: 7
- Rationale: Likelihood = Medium——ESSD-first 成稿路径明确且 NULL 兼容, v4 全部设计级缺口已闭合, gate 机器经真数据验证; 但成功仍同时依赖多区域覆盖/许可/产率审计通过、canonical 总体冻结、无泄漏 split 构造和长期托管落地等多个条件, 未达 "主要风险仅剩工程执行" 的 High 标准。Impact = High——若交付原始演化场级、issue-time 可审计、基率可解释的现代 HRRR-MRMS 开放基准并坐实 (或功效充分地否证) 环境增量, 会实质影响 env-aware nowcasting/warning 研究线并为姊妹 topic 提供数据底座; 组件层面先例众多、限于 CONUS, 不达 Exceptional。查表 Medium × High = High (7)。Claude 与 codex 在 Novelty (5/5)、Likelihood (Medium/Medium)、Impact (High/High)、Priority (High/7)、hard cap (no) 上完全一致, 无 >= 1 level 分歧。

## Overall

- Priority: High
- Score: 7
- Comments: v5 是本系列收敛质量最高的一版: 三条唯一轴忠实于审计表且经我独立复核, smoke test 经全链路第三方复算 (radar 5/5, 报告匹配 6/6) 证实真实诚实, v4 四个 ACCEPT 项全部落地, 无 contribution drift、无 hard cap。尚差 proposal-ready 一步之遥, 单一最重要遗留 blocker: 冻结概率任务的 canonical 评测总体 (C3 两案择一, 给出总体/权重实例) 并连带定义重叠包无泄漏 split 规则; 次级事项: 正常态候选加 distance-to-event 分层、实验 6 的容量匹配协议、"6 倍裕量" 措辞降温为 52/84 实测值。这些均为 proposal 首章工作量, 不构成方向风险, 建议进入 proposal 阶段并在 proposal 中把 30 日审计设为 go/no-go。
超长警告: 文件 10.0 KB (10001 bytes, 不含本次 review 块), 超过 10 KB 上限 0.01% — 临界值, proposal 阶段请勿再增重 idea 文件。

</review>
