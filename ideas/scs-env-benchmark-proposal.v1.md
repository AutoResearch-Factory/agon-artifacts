<!-- 书写报告使用中文 -->
---
idea: scs-env-benchmark
title: "SCS-EnvBench：带 issue-time 契约的强对流环境—雷达开放数据产品"
version: 1
date: 2026-07-16
workspace: workspace/scs-env-benchmark/
---

## Technical Gap

现有公开资源分别覆盖雷达影像、全球天气评测、事件报告或对象级特征，但缺少一份把原始环境场演化、雷达序列、事件前正常态和预警/实况的可用时刻语义共同产品化的数据底座。结果是环境条件化预报研究反复自建不可比管线；若仍以事后报告裁预测区域、在事件富集样本上报 Brier 分数，模型即使有效也不能回答真实基率总体上的校准问题。

必须解决的瓶颈不是新模型，而是四个可执行契约：源资产到样本的 issue/valid/availability-time 谱系、分析与预测双视图、代表性概率评测总体、跨重叠事件包无泄漏 split。成功标准是第三方仅凭发布清单即可重建样本，在冻结总体上复现概率与 nowcasting 评测；预测增量为 NULL 时，数据产品仍满足 ESSD 的可追溯、可复用要求。

真实 pilot 已完成而非计划结果：2024-05-06 Oklahoma 四源访问与 schema/hash 检查 4/4 PASS；ST-DBSCAN 在真实 114 条报告上得到 1--17 簇、对 NOAA `EPISODE_ID` 的最高 ARI=0.206，并完成每个 cutoff 先截表再重聚类的审计；84 个真实 MRMS 时刻无缺测，其中 52/84 满足正常态候选条件、32 个位于 -15d..-2d。后两数只作 6 小时采样实测，不外推为小时数或“六倍裕量”。无中断下载，无需本轮扩大下载；30 日审计先于全量采集。

## Method Thesis

- One-sentence thesis: 发布同源、双视图、issue-time 可审计的 HRRR--MRMS--Storm Events--预警数据产品，并用未抽样固定时空总体与重叠组件 split，使环境演化分析、概率预报和雷达锚定 nowcasting 可复现且不混入事后信息。
- Why this is the smallest adequate intervention: 复用 NOAA/IEM 源、FunnelCloud 在先的表格 ST-DBSCAN、成熟气象基线与评分器；新增的是数据契约和评测清单，不发明追踪器或预报架构。
- Why this route is timely in the foundation-model era: 环境条件化模型已证明多源组装与 train-on-analysis/test-on-forecast shift 是瓶颈（FuXi-Nowcast, 2512.08974），统一底座比再加一个专有模型更能改变可比性。

## Contribution Focus

- Dominant contribution: 版本化开放数据产品，含去重资产索引、原始/谐一化层、issue-time 语义、固定与风暴中心双视图、正常态抽样框、canonical tasks/splits/metrics、CF/ACDD/STAC/Croissant/RAI 元数据、代码与 DOI。
- Optional supporting contribution: 在冻结总体和容量匹配下，报告原始环境演化场相对 issue-time-safe 特征统计表的增量如何随 lead time、区域和季节变化；功效充分的 NULL 同样报告。
- Explicit non-contributions: 不主张 ST-DBSCAN、预测模型或“环境+雷达+标签”组合首次；不作 15 天物理记忆、因果、业务部署或 CONUS 外泛化主张。0--6 h 报告预警与 3D 风暴核仅在专项 pilot 后转正。

## Proposed Method

### Complexity Budget

- Frozen / reused backbone: NOAA HRRR/MRMS/Storm Events、IEM/NCEI 预警档案，ST-DBSCAN 表格聚类，逻辑回归、树模型、持久性/光流及小型场编码器。
- New trainable components: 无数据构建必需组件；示范实验只训练容量匹配的表格/场编码器和 MRMS-only/+environment 两臂。
- Tempting additions intentionally not used: 不加入 LLM/VLM、扩散生成器、新追踪算法、卫星/闪电/WoFS 或全分辨率 Level-II。后续版本可加模态，但不改变 v1 契约。

### System Overview

```mermaid
flowchart LR
    A["HRRR 与 MRMS"] --> E["去重资产与时间谱系"]
    B["Storm Events"] --> E
    C["预警档案"] --> E
    E --> F["固定网格点日视图"]
    E --> G["回顾事件包视图"]
    F --> H["未抽样概率评测"]
    G --> I["环境演化分析"]
    E --> J["cutoff 前雷达锚点"]
    J --> K["环境约束 nowcasting"]
```

### Core Mechanism

#### 1. 数据与时间契约

每个资产记录源 URL/对象键、字节范围、SHA-256、处理 commit、变量/层位、issue time、valid time、实际或可复算 availability time、插值窗口、QC 与许可状态。预测视图在 cutoff `t` 只解析 `availability_time <= t` 的资产；HRRR f00 分析与 f01+ 预报分组保存，不以 valid time 相同为由互换。Storm Events 及其约 75--90 日发布时滞只作事后 target/provenance，不进入输入或区域定义；研究阶段等待最终标签不损害任务，业务可用性声明则明确禁止。

固定视图使用报告无关的 CONUS 0.5° NCEP Grid 4 网格；事件包视图用整日 Storm Events 表做回顾 ST-DBSCAN 索引，区域随报告簇形成但绝不用于预测裁剪。每个 cutoff 重跑“先按时间截表、再聚类”，量化最终 bbox 的后视偏差。nowcasting 空间锚仅来自 cutoff 前 MRMS 回波。

环境窗固定为事件锚点 -15d/+5d，用于正常态预算、环境演化和姊妹课题估计器预热，不解释为物理记忆。landscape H 节只支持小时至数日并建议 -72 h；因此物理 precursor 分析明确限于 -72h..0，-15d..-72h 仅作背景/预热池。正常候选仍须同时满足 bbox 内 ±3 h 无匹配报告且 MRMS 最大反射率 <35 dBZ，并发布到最近“事件信号”的距离：事件信号为同 bbox 报告时刻或 MRMS 首次/再次达到 35 dBZ 的时刻，分箱 `<6 h`、`6--24 h`、`24--72 h`、`>72 h`。主要正常合成只用 `>=24 h`；近事件候选保留但不混称静稳背景。

#### 2. 冻结的 canonical 概率总体

选择 C3 的“未抽样固定时空框架”而非纳入概率加权。发布总体

`U = {(g,d): g 为预先发布的 CONUS 陆地 0.5° 网格点，d 为 2018--2024 每个 eligible 1200--1200 UTC 有效日}`。

每个 `(g,d)` 的 cutoff 为有效日开始前 12 h 的 0000 UTC，输入仅含当时已可用的 HRRR 环境轨迹，目标 `Y=1` 当且仅当随后 12--36 h 有龙卷、冰雹 >=1 in 或对流大风 >=50 kt 的最终 Storm Events 报告落在 `g` 的 40 km 邻域；否则为 0。40 km/24 h 定义复用 Hill et al.（2208.02383），并与 Flora et al.（2603.20250）发现 36 km 有技巧而 9/18 km 近乎无技巧的尺度证据一致。静态 land/coverage mask 与因源缺测形成的 eligible mask 在读取标签前冻结并逐项发布；不得按事件密度删 test 单元。

确认性 test 包含其全部 eligible 网格点日，不抽 confirmed/hard/random 子集。面积权重 `w(g,d)=cos(latitude_g)`；例如 35°N 与 45°N 单元的未归一化权重分别为 0.819 与 0.707。Brier、BSS 和可靠性分箱均用同一权重，BSS 参照只用 train 年估计的逐日/逐时气候学（±15 日、120 km 平滑），逻辑回归为第二强基线。三类样本仍作为训练、困难负例诊断和独立 stress-test 发布，可任意重采样；其分数不得替代 `U` 上的主结果，也无需估计纳入概率。

#### 3. 无泄漏 split 与重叠包规则

canonical 时间块固定为 train=2018--2022、validation=2023、test=2024。所有同一 UTC 日的网格单元先整体入同一 split，故同日相邻区域不会跨 split。所有任务样本仅在完整输入/标签支撑落入对应时间块时进入 manifest；事件包按最长的 -15d/+5d 支撑统一执行各块首 15 日、末 5 日 embargo。跨界包仍在档案层发布，但不进入 canonical 训练/评测。

随后按实际谐一化支撑建无向图：两个包若共享任一 `(source, variable, spatial_chunk, valid_hour)`，或其 bbox 加一格 halo 后相交且事件核心相距 <=24 h，则连边；连通分量即 `leakage_component_id`，正常候选引用也继承该 ID。一个分量只能属于一个 split；若审计发现跨块分量则从任务 manifest 整体 purge，不拆包、不复制为不同标签。地域泛化附加采用 leave-one-NCEI-climate-region-out：跨界分量归 test，训练再去掉一格空间 halo。显著性以该分量为 block 做 2000 次 bootstrap，分量内日期/区域共同移动。

TorNet 的 Julian-day-mod-20 规则不作 canonical 切分：21 天环境窗会穿透逐日交错。为可比性，仅在完成上述分量合并后记录 `j20 = earliest_anchor_day mod 20` 辅助字段及 `<17 / >=17` 标签；它只用于 event-core stress-test，不支持主 claim。

#### 4. 雷达锚定 nowcasting 契约

输入为 cutoff 前 MRMS 历史与 issue-time-safe HRRR，比较 MRMS-only 与 +environment；输出未来 MRMS 组合反射率，分别报告 0--2 h 与 2--6 h。主判据为 5 km 邻域最大池化后的 40 dBZ CSI，30/50 dBZ 为预注册次指标，并同时报告 POD/FAR/frequency bias/FSS。确定性场直接用同一物理阈值；若输出超阈概率，只在 validation 的 `q=0.05,0.10,...,0.95` 中选择使 `|log(FB)|` 最小者，平局取 CSI 高者再取较高 `q`，按 lead/反射率阈值冻结后一次性测试，并发布完整 performance diagram。`Delta CSI = CSI(+environment)-CSI(MRMS-only) >=+0.02` 只在对应 lead 与 5 km/40 dBZ 层声明，不能跨尺度汇总掩盖 NULL。

### Optional Supporting Component

- Only include if truly necessary: 环境演化分析是一等数据用途，不是预测附录；以 `>=24 h` 正常层为参照，合成 -72h..+事件衰减期的 CAPE/CIN/PWAT/shear/湿度廓线与 MRMS 强度、结构、移动关系，按区域、季节和标签置信度分层。
- Input / output: 输出带不确定性的合成曲线、事件级效应分布与可复算样本清单；不输出因果效应。
- Training signal / loss: 无训练；事件/`leakage_component_id` 为重采样单位。
- Why it does not create contribution sprawl: 它直接验证长窗和正常态交付物的科学可用性，仍服务同一数据产品。

### Modern Primitive Usage

- Which LLM / VLM / Diffusion / RL-era primitive is used: 无；此数据 proposal 不需要前沿生成组件。
- Exact role in the pipeline: 现代天气基础模型仅是未来消费者，可在固定输入/输出契约上另报结果。
- Why it is more natural than an old-school alternative: 不适用；简单基线更能暴露数据契约本身的价值。

### Integration into Base Generator / Downstream Pipeline

源文件只存一次，固定视图与事件包通过索引引用同一资产；开放层发布可再分发数据、标签、元数据与 derived cubes，受限或条款未覆盖的资产只发布 hash 清单和重建 recipe。数据 DOI 与软件 DOI 分离、版本不可变；landing page 提供英文说明、license matrix、datasheet、field dictionary、split manifests、读取示例和独立数值复核样本。

### Training Plan

先跑 30 日 gate，再全量索引/谐一化，最后冻结 manifest 后训练基线。旗舰“特征表 vs 原始演化场”只用 canonical 概率任务：两臂共享网格、cutoff、变量、-72h..0 可用窗口、训练样本、loss、优化器、更新数、5 个 seed 与 12 次 validation 调参预算；表格特征只由固定 40 km 邻域内 cutoff-safe 场计算 mean/max/std 与分位数，不用事后对象位置。两编码器均输出 256 维、共享分类头，trainable parameters 匹配在 ±5% 内；主表同时报告参数量、训练 FLOPs 与峰值显存。这样比较的是信息形态，不是未来定位、样本或容量差异；MYRORSS-2026 只作对象级特征统计近邻，不跨 2006--2011 RUC 与现代 HRRR 直接比数。

### Failure Modes and Diagnostics

- 四源覆盖、再分发或体量失控: 30 日矩阵逐源给出 cadence、缺测、字节与 license；未知条款即 FAIL 或转 recipe-only，不启动全量下载。
- 聚类不稳: 报告参数网格、簇数/面积/噪声与 cutoff 重聚类；若相邻参数使簇统计跨 2 倍或无法冻结，删除报告簇任务，不拿最终区域替代预测区域。
- 正常态不足或近事件污染: 执行 80%/48 h/24 h gate与 distance-to-event 分层；失败即否定当前窗/包规格。
- 标签偏差: 保存源版本、空间/时间不确定性及报告类型；按人口密度/时代/区域分层，不把“无报告”称为无灾真值。
- 环境增量为 NULL: 数据照发；不得再以预测增量论证长窗，长窗价值限演化分析和预热用途。
- 托管成本过高: DOI 层优先发布谐一化子集、资产索引和可复算 recipe；不承诺镜像所有上游原始文件。

### Novelty and Elegance Argument

15 个近邻审计中仅三轴同时空缺：事件对齐的原始环境长序列、正常态时段作为交付物、预警与实况分离的 issue-time 契约。HR-Extreme（2409.18885）只在论文内抽 ±15d 正常时刻；MYRORSS Storm Cluster 2026（DOI:10.5281/zenodo.19644586）交付约 1060 列对象级派生统计而非场/廓线/演化序列；MeteorPred（2508.06859）把预警当标签。FunnelCloud 已在 2017 年使用报告 ST-DBSCAN 关联环境与雷达，故聚类及宽松组合不主张新颖。方案的简洁性在于一个时间/空间/标签契约同时约束三种消费者，而不是为每个任务再造一套数据。

## Claim-Driven Validation Sketch

### Claim 1: v1 发布规格可被真实数据支撑

- Minimal experiment: 30 个分层抽取 UTC 日覆盖至少 3 个气候区，审计四源交集、许可、小时缺测/体量、ST-DBSCAN 参数与 cutoff 重聚类、正常态产率。
- Baselines / ablations: 35 dBZ 扫 30/40 dBZ，报告缓冲 ±3 h 扫 ±1/±6 h；聚类参数各上下一个网格点。
- Metric: HRRR 所需场与小时 MRMS 审计覆盖均 >=95%，四源状态无 unknown license；相邻聚类设置的簇数/中位面积不得跨 2 倍；>=80% 包含 >=48 正常候选小时且 >=24 小时位于 -15d..-2d。
- Expected evidence: 全部通过才启动全量下载；任一失败即停在审计报告并修改或撤回对应发布规格，不以 pilot 52/84 代替。

### Claim 2: 数据产品支持基率可解释的概率评测，且原始演化场的信息增量可被公平检验

- Minimal experiment: 在冻结 `U` 与 year split 上跑气候学、逻辑回归、容量匹配特征表、原始场四臂，并做正常态—事件演化分析。
- Baselines / ablations: 特征表 vs 原始场为决定性消融；单时刻场只作诊断，不扩成第三贡献。
- Metric: area-weighted BS/BSS、reliability、AUROC/NAUPDC；按 `leakage_component_id` bootstrap。分别冻结 `Delta BSS_base=BSS_raw-max(0,BSS_logistic)` 与 `Delta BSS_repr=BSS_raw-BSS_table`；对应 claim 仅在差值 >=+0.02 且 95% CI 不含 0 时成立。该门槛只适用于 0.5°/40 km/12--36 h，等价于两臂 BS 差至少达到气候学 BS 的 2%，避免海量网格点把微小差异做成“显著”；40 km 尺度有 2208.02383 与 2603.20250 的直接协议依据。
- Expected evidence: 预期完整场在部分区域/季节有增量，但不预设正结果；NULL 支持“统计表已足够”的克制结论。

### Claim 3: 环境约束的价值随 nowcasting lead time 改变

- Minimal experiment: 同 split、同容量和训练预算比较 MRMS-only 与 +environment，分 0--2 h/2--6 h 报告。
- Baselines / ablations: 持久性、光流、MRMS-only；+environment 是唯一主消融。
- Metric: 5 km/40 dBZ CSI 主指标，30/50 dBZ、POD/FAR/FB/FSS 辅助；每个 lead 的 `Delta CSI >=+0.02` 且 component-bootstrap 95% CI 不含 0 才作增量 claim。5 km/30--50 dBZ 协议复用 FuXi-Nowcast；短 lead 环境可被观测替代、较长 lead 增量上升的先例使分 lead 判定优于 pooled gate。
- Expected evidence: 允许 0--2 h 为 NULL、2--6 h 为正，也允许全 NULL；0--6 h 报告预警和 3D 核失败只删除可选项。

## Paper Outline

- Section 1: 数据产品空白、三条审计差异轴与 ESSD/NeurIPS D&B 双轨定位。
- Section 2: 四源、时间语义、双视图、正常态与许可/provenance。
- Section 3: 固定总体、标签、重叠组件 splits、tasks/metrics。
- Section 4: 30 日 QC、覆盖与代表性；Section 5: 演化分析和最小基线/旗舰消融；Section 6: 局限、版本与访问。
- Key figures: 资产到双视图的谱系图；固定总体与 split/embargo 图；覆盖/QC 地图；正常态—事件演化；特征表/原始场及 MRMS-only/+environment 的 lead-time 图。

## Compute and Timeline Estimate

- Estimated GPU-hours: 整编以 CPU/网络/存储为主；概率基线、容量匹配消融与初步 nowcasting 共 300--800 GPU-hours；全量 nowcasting 预算在 30 日 gate 后冻结。
- Data / annotation cost: 无新增人工标注；预计多 TB，先核算 dedup 后字节、出口与 DOI 托管费用。当前本地 handoff 为 23,916,994 bytes 源 pilot 加 573 MB、84 个 git-excluded MRMS smoke 文件。
- Timeline: 30 日 gate 3--4 周；数据产品、概率旗舰与演化分析 6--12 个月，形成 ESSD 主稿；nowcasting 第二期，总计 12--18 个月，NeurIPS D&B 仅在 canonical 结果与托管完成后投稿。

<review date="2026-07-16">
## Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| Problem Fidelity | 9/10 | v5 idea review 指名的唯一 proposal-blocking 项被正面交付: C3 两案择一冻结为未抽样固定总体 `U`(CONUS 陆地 0.5° NCEP Grid 4 网格点日, 2018--2024, 1200--1200 UTC), cos(lat) 面积权重数值示例正确 (0.819/0.707), 年块+embargo+重叠组件 purge 规则成文。40 km/0.5°/24 h 协议直接复用 Hill et al. (2208.02383) 的 SPC 生态标准 (经 wiki 全文核验属实), 0.5° 相对 3 km 原生分辨率的选择有据——原生场仍在数据产品里, 只有 canonical 概率任务放在报告可解释的尺度上。全部 owner-settled 决定 (-15d/+5d 窗+正常态对比理由、双任务类、演化分析一等、Storm Events 时滞 pushback、ST-DBSCAN 表格聚类、MYRORSS-2026 种类区分) 逐项得到尊重, precursor 分析自限 -72h..0 与 landscape H 节证据一致。codex 同评 9/10。 |
| Method Specificity | 7/10 | 冻结件本身可执行: 总体定义、cutoff、标签阈值 (龙卷/雹>=1in/风>=50kt, 与 Hill/Flora 同源)、40 km 邻域、权重、年块、首15日/末5日 embargo、组件图边规则 (共享资产 OR bbox+halo 相交且事件核 <=24h)、整分量 purge、j20 辅助字段的处理均可由第三方复现。残余实现歧义见 Weaknesses: 旗舰输入张量的 valid-time/issue-time 语义、spatial_chunk 未定义、eligible mask 构造规则未成文、`U` 上非事件格点日的 bootstrap 单元未覆盖。codex 给 6/10, 主因相同。 |
| Contribution Quality | 8/10 | 主贡献 (契约化数据产品) 唯一且清晰, 特征表 vs 原始场增量作为从属 empirical finding 定位正确; Complexity Budget 与 Explicit non-contributions 诚实克制 (不加 LLM/新追踪器/新模态, 0-6h 预警与 3D 核 gated)。三任务并列继承自已关双闸门的 idea v5 (owner-settled), 两期交付实质缓解 omnibus 风险; codex 给 6/10 但其修复建议 (演化分析降级、Claim 3 整体移除) 属于重新审理 owner 决定, 不予采纳——本轮只采纳"outline 与两期时间线对齐"的一致性修复。 |
| Frontier Leverage | 8/10 | 对数据 proposal 而言克制正确: 不硬塞 LLM/扩散, 以 FuXi-Nowcast (2512.08974) 的 issue-time/analysis-vs-forecast shift 证据为契约动机, STAC/Croissant/RAI 是 D&B 当前标准。经本轮 web 检索确认无更新竞品出现 (HR-Extreme/AgentCaster 之外无新占位)。可小幅现代化: 派生 cube 的版本化云原生存储 (Zarr v3/Icechunk) 与具名仓储组合未落地。codex 给 9/10。 |
| Validation Focus | 7/10 | 三条 claim 各配最小实验, 30 日审计正确设为全量下载前 go/no-go (失败即停/改规格, 不以 pilot 52/84 顶替), ΔBSS/ΔCSI 门槛有尺度限定与"气候学 BS 的 2%"业务论证, 容量匹配协议 (±5% 参数、共享 head、12 次调参、5 seeds) 落地。缺口: 自设的核心成功标准"第三方仅凭清单重建样本"没有对应实验; lead×区域×季节扫描无多重比较约束; `U` 上主检验的 bootstrap 块定义不完整。codex 给 6/10, 主因相同。 |
| Paper Story and Claims Calibration | 7/10 | 章节-图表骨架完整, NULL discipline 好 (NULL 时数据照发且长窗退守演化用途), pilot 数字经本轮对 workspace 日志逐项复核完全一致 (4/4 PASS、1--17 簇、ARI=0.206、84/0 缺测、52/84、32 个 -15d..-2d、23,916,994 bytes、573 MB), 且"不外推为六倍裕量"的降温措辞兑现了 v5 review 要求。缺口: key figures 含 nowcasting lead-time 主图, 与 Timeline"nowcasting 第二期"自相矛盾; NEGATIVE 行未区分"发布规格失败"与"raw 显著劣于 table / +env 劣于 MRMS-only"两种性质不同的负结果; ESSD 要求的具名永久仓储未点名; S1 缺 15 近邻对照表这一必要故事件。codex 给 7/10。 |
| Overall | 7.4/10 | Claude 六维平均 7.7; codex 独立评审 7.2 (REVISE), 分歧 0.5 (<2, 无需标注分歧); 最终 = (7.7+7.2)/2 = 7.4。 |

## Verdict
REVISE (codex: REVISE; Claude 按小分规则亦为 REVISE。方向与冻结件质量均好, 距 READY 是一轮 spec 补全, 不是方向问题。)

## Weaknesses (dimensions < 7 含 codex 判 <7 的维度)

### Method Specificity (Claude 7, codex 6)
- Weakness 1 (codex 提出, 有效): 旗舰概率任务的输入张量不唯一。总体定义处写"输入仅含当时已可用的 HRRR 环境轨迹" (availability_time <= t, 允许早前 cycle 的 f12+ 预报覆盖目标窗), Training Plan 又写"-72h..0 可用窗口" (读作 valid time 只到 cutoff, 即纯历史轨迹)。两种读法对应两个难度完全不同的任务, 第三方无法确定实现哪个。
- Suggested fix: 冻结一张 normative sample schema: 逐源 product/cycle/lead 集合、变量/层位清单、valid/issue/availability 规则、重网格算子、tensor shape、缺测判定; 显式声明旗舰输入是"valid time 在 [cutoff-72h, cutoff] 的 issue-time-safe 分析/短前时轨迹"(若这就是设计意图), 并把"允许早前 cycle 预报轨迹"列为独立诊断臂或明确排除。
- Priority: CRITICAL (纯 spec 工作, 0 GPU)
- Weakness 2: 组件图边规则中 `(source, variable, spatial_chunk, valid_hour)` 的 spatial_chunk 粒度未定义, 而共享边的连通结构对 chunk 大小定性敏感 (CONUS 级 chunk 会把活跃季全部包链成超大分量); eligible mask 的构造规则 (何种源缺测使 (g,d) 出局) 只承诺"冻结并发布", 规则本身未成文。
- Suggested fix: 把 chunk 定义为发布的谐一化存储 tile (给出具体度数), 随 manifest 冻结; 写出 eligible 判据 (例如: cutoff 前 72h 内 HRRR 缺失 cycle 数 <= k 且标签窗内 Storm Events 源文件完整)。
- Priority: IMPORTANT
- Weakness 3: `U` 是网格点日总体, 多数负例不属于任何事件包, "按 leakage_component_id bootstrap" 对它们无定义。
- Suggested fix: 任务分治——`U` 的主检验用完整 UTC 日 (或 日×NCEI 气候区) 为 block 的 bootstrap; 组件 block 只用于事件包视图 (演化/nowcasting) 的推断。codex 同议。
- Priority: IMPORTANT
- Weakness 4 (codex 提出, 措辞级): Flora et al. (2603.20250) 的 9/18 km 无技巧结果在原文中标为 "not shown" 的早期实验, 只能作为协议先例引用, 不能写成本任务尺度选择的直接证据。
- Priority: MINOR

### Validation Focus (Claude 7, codex 6)
- Weakness: Technical Gap 自设的成功标准是"第三方仅凭发布清单即可重建样本、复现评测", 但 Claim 1--3 没有任何实验验证它; Integration 节的"独立数值复核样本"无协议。
- Suggested fix: 在 Claim 1 下加一个零额外下载的重建审计: 独立进程 (最好独立实现者) 仅凭 manifest 重建分层抽取的样本, 核对源 hash、资产选择、张量数值容差与 cutoff 泄漏为零, 结果作为发布 gate 之一。另: lead×区域×季节扫描声明为 exploratory 或预注册层级/多重比较规则。
- Priority: IMPORTANT (第一项接近 CRITICAL——它直接检验主贡献的核心 claim, 且成本极低)

### Paper Story and Claims Calibration (Claude 7, codex 7)
- Weakness: (a) key figures 列出 MRMS-only/+environment lead-time 图, 但 Timeline 把全量 nowcasting 放在第二期——outline 与交付计划自相矛盾; (b) NEGATIVE 结果只覆盖 gate 失败, 未界定"raw 显著劣于 table"或"+environment 显著劣于 MRMS-only"时可说什么; (c) ESSD 仓储标准 (具名可信仓库、DOI、英文 landing page, 经 ESSD repository criteria 页核验) 只有策略无落点, 多 TB 体量 Zenodo 单家装不下; (d) S1 无 15 近邻对照表。
- Suggested fix: (a) v1 主稿 Section 5 收为"概率旗舰 + 演化分析", nowcasting 保留契约、任务规格与一个 smoke baseline, lead-time 主图移到第二期 D&B 论文 (与既定两期计划一致, 不删任务); (b) NEGATIVE 行拆成两支: 规格失败→修改/撤回发布规格; 增量为负→只可声称"更简单输入在该协议下更优", 不得推及环境信息普遍无用; (c) 点名仓储组合 (如谐一化大体量层走 AWS Open Data/NODD 类对象存储 + Zenodo/PANGAEA 存 manifest/标签/代码并铸 DOI) 并给出费用量级; (d) S1 增加近邻对照表 (landscape 审计表已有现成内容)。
- Priority: IMPORTANT

## Simplification Opportunities
- 单一 canonical manifest 作为真源, CF/ACDD/STAC/Croissant/RAI 全部由它自动导出, 不并行维护五套元数据规范 (codex 提出, 采纳)。
- `U` 的 split 与推断完全脱钩于事件包组件图 (年块+完整支撑规则已足够), 组件机器只服务事件视图——同时消掉 Weakness 3。
- v1 主稿的 nowcasting 部分收为契约+smoke baseline (上文 Paper Story fix a), 全量训练与 lead-time 结论留第二期; 这是既定两期计划的对齐, 非任务删除。

## Modernization Opportunities
- 派生 cube 采用 Zarr v3 + Icechunk (或等价) 做版本化云原生存储, 与"版本不可变"承诺天然匹配 (经 web 核实为当前标准做法)。
- 仓储落点具名化 (AWS Open Data Program/NODD 对象存储 + DOI 铸造仓库组合), 满足 ESSD repository criteria。
- 建模侧 NONE: 不加生成式/LLM 组件是正确决定, 简单强基线 (Flora 2026 中 HGBT 仍略优于 U-Net) 未过时。

## Drift Warning
NONE。proposal 仍解决 idea v5 锚定的问题 (同一 issue-time 契约支撑三类一等用途), 且恰好交付了 v5 review 指名的唯一 blocker。反向说明: codex 建议把演化分析降为"数据表征验证"并整体移除 Claim 3, 属于重新审理 owner-settled 决定, 本综合评审不采纳; 采纳的只是 outline 与两期时间线的一致性修复。

## Results-to-Claims Mapping

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 数据 gate、(补上的) 独立重建审计、cutoff 泄漏审计与冻结 `U` 全部通过后, 可声称该契约支持真实基率下可复现的概率评测; `Delta BSS_repr >= +0.02` 且 component/日 block CI 不含 0 时, 可声称原始演化场在 0.5°/40 km/12--36 h 协议上优于 issue-time-safe 特征表; nowcasting 增量只在对应 lead 与 5 km/40 dBZ 层声明。不得推及因果、15 天物理记忆、业务部署、CONUS 外或其他尺度。 |
| NULL | 数据产品与可复现性 claim 照常成立 (ESSD 路径); 功效充分时可支持"cutoff-safe 统计表对该任务已足够"的克制 finding; 长窗价值退守演化分析与预热用途, 不得再以预测增量论证长窗。 |
| NEGATIVE | 覆盖/许可/产率/重建 gate 失败→否定当前发布规格, 修改或撤回, 不以事件富集样本续报; raw 显著劣于 table 或 +environment 劣于 MRMS-only→只可声称更简单输入在该协议下更优; 可选任务失败只删对应任务, 不伤合格的 ESSD 产品。 |

## Paper Outline Check
骨架正确 (空白→契约→总体/split→QC→示范→局限), 关键图基本齐。需要补的故事件: S1 的 15 近邻对照表 (审计表现成)、S2 的逐源 issue/valid/availability 时间轴泄漏路径图、S4 的独立重建一致性表; 需要移走的: lead-time nowcasting 主图 (第二期)。S5 以"概率旗舰 reliability/ΔBSS 图 + 正常态—事件演化合成"为双主图即可支撑核心 claim。文件 17.2 KB, 未超 20 KB 上限。

</review>
