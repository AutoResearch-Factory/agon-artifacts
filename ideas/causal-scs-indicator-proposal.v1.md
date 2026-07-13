<!-- 书写报告使用中文 -->
---
idea: causal-scs-indicator
title: "因果时序不稳定性特征对原始 4D 强对流预警 CNN 的公平增量审计"
version: 1
date: 2026-07-13
workspace: workspace/causal-scs-indicator/
---

## Problem Anchor（原样保留, 逐版不漂移）

- Bottom-line problem: 跨气压层、多尺度滞后的因果时序不稳定性特征 `(D,J)`，能否为一个在原始 4D（pressure-level x longitude x latitude x time）场上端到端训练的强对流预警模型，提供该模型自身从原始历史中学不到的增量预警信息？
- Must-solve bottleneck: 此前九轮（v1-v9）比较的都不是"合格的原始张量端到端模型"——依次是无 CI 的单 seed 合成筛查、手工聚合特征上的 LR/RF、闭式统计量。v10 首次做到真正的 raw 张量直喂 CNN、容量匹配、含 sham，但两位独立审稿人一致指出仍有两处偏差源未受控：(1) v10b 架构（`4x2x2` 粗池化）是看到 v10（全局池化）训练失败后才选的，属于"看结果选架构"；(2) 维度匹配的 Gaussian sham 只控制参数量，没控制 `(D,J)` 享有的"已知真实滞后/邻接边"特权——容量相同但不知道该往哪看的 CNN 不是公平对照。
- Non-goals: 不主张因果估计器（PCMCI+/LKIF）新颖；不主张融合机制新颖；不识别真实大气因果图；不证明物理分岔机制；不构建新的时空骨干网络。
- Success condition（REAL-4D 阶段）: `>=2` 季节、`>=50` 个独立 storm systems 的 outer storm holdout 上，raw+`(D,J)` 相对同容量/同历史/独立冻结架构的 raw-only，`Delta AUPRC >= 0.02` 且 CI 下界 `> 0.01`；同时优于 sham 与特权匹配非因果对照；Brier/ECE 不退步；跨季节复现同号。

## 数据/计算资产交接状态（本轮核查）

- SPC 逐日报告（9 文件, 205KB, 2024-03-14/05-06/05-25）与 HRRR 单时刻 3km 分析子集（3 文件, 34MB）：均下载完成、未损坏（本轮核对字节数/SHA-256 与 `data/MANIFEST.md` 一致，无残留 `.part`/`.tmp`）。
- 连续多日/多风暴的完整 96h HRRR 序列：**未下载，且非中断或失败下载**——是刻意的 stop gate（v1-v9 合成前置门槛未通过前不启动全量拉取，避免烧预算）。本机当前无任何在跑/残留下载任务（已用 `squeue`/`ps` 核实）。因此不需要"接续中断下载"，而是要把 stop gate 解除条件写清楚——见 Gate 0 与 Compute/Timeline 两节。
- 计算环境：workspace 本地 uv venv（PyTorch 2.9.1+cu128, sklearn 1.9.0）、`tigramite`（PCMCI+/ParCorr）、`LK_Info_Flow`（LKIF）均已装好，直接复用。

## Technical Gap

**失败点**：v10 位置保留 CNN 是十轮里唯一让 raw 分支真正收敛（AUPRC 0.690）的版本，causal-minus-raw 仍是 `+0.065 [-0.013, +0.154]`——CI 跨零。但两位独立审稿人指出这个 NULL 可能仍偏乐观：比较双方不对等，`(D,J)` 在已知真实耦合边（850→250 hPa，3h）与真实滞后（500→850 hPa，24h）的位置上直接计算，raw/sham 分支必须从零学习"该往哪个像素、哪个时刻看"。参数量相同不等于任务难度相同。

**为什么朴素修补不够**：加 seed 只会让被污染的效应量估得更精确，不消除偏差；换更大/更现代的骨干不解决"对照没被告知往哪看"这一问题，任何容量的 raw 模型都吃同样的亏；单纯拿更多真实数据只是把同一偏差在更贵预算下重犯一遍。

**最小充分干预**：不引入新因果发现算法、新融合机制或新骨干，只修两处协议缺陷——(a) 架构选择在触碰任何因果/sham 通道结果之前，用独立合成校准集盲选并冻结；(b) 在 Gaussian sham 之外新增一个"特权匹配非因果对照"：给它和 `(D,J)` 相同的候选滞后/邻接搜索特权，但去掉条件化，只用确定性非因果统计量（边际滞后相关、局部方差、一阶差分幅度）。一个新表示同时回应审稿人列出的"结构化非因果对照"与"oracle 位置/滞后特权"两个缺口，不是两个新模块。

**Route 比较**：Route A（采用）——协议层最小修补，因果表示与骨干完全不变，直接对准两处具体偏差源，novelty 落在审计协议而非新模型上。Route B（放弃）——换现代时空骨干（3D ViT/Swin/扩散式）重做整套比较；更"前沿"但不对准瓶颈（瓶颈是协议不公平，不是表达能力不够），且骨干越复杂越难在预算内避免"看结果选设计"。用户已明确方法新颖性非重点，Route B 会把预算从"问对增量问题"移到"把骨干做新"，是研究问题漂移，予以拒绝；仅保留一条附录级 robustness check（见 Modern Primitive Usage）。

**核心技术主张**：消除架构事后选择与 oracle 位置/滞后特权后，`(D,J)` 能否为同历史、同容量、独立冻结架构的 raw-4D CNN，在真实 storm-level HRRR/SPC 数据上提供跨季节可复现的 AUPRC 增量——十轮里从未在去偏协议下问过的问题。

**所需最小证据**：合成 Gate 0 通过；`>=2` 季节 `>=50` storm 的真实 nested holdout 上 `Delta AUPRC` 的 storm-cluster 校准 CI；同时优于两类对照；校准不退步；移除新增控制会改变结论（证明协议非装饰）。

## Method Thesis

- 一句话主张：把审计协议本身修到公平（独立冻结架构 + 特权匹配非因果对照 + 校准 bootstrap），是判断 `(D,J)` 能否为已训练好的原始 4D 强对流 CNN 提供真实增量所需的最小且充分的修补。
- 为什么是最小充分干预：所有新增都是协议层（何时冻结架构、多一个对照臂、如何校准区间），零新学习组件、零新因果估计器、零新融合机制；因果表示、CNN 骨干、Gaussian sham 全部复用 v10 已验证代码。
- 为什么现在及时：不靠生成式前沿组件，而是 anchor regression / IRM / DomainBed 一脉文献（1801.06229, 2007.01434, 2010.16412, 2402.09891）反复证明因果衍生特征优于强 baseline 类主张在严格公平对照下大比例站不住；本项目把这一已成熟的审计标准具体应用到强对流预警上。

## Contribution Focus

- Dominant contribution：消除两处已知偏差源（架构事后选择、oracle 位置/滞后特权）后，对"`(D,J)` 能否为真实 storm-level 4D 数据上端到端训练的强对流预警 CNN 提供增量"给出 storm-holdout 级别、诚实标注统计可判定性的答案（无论方向）。
- Optional supporting contribution：把审计协议（盲选架构 + 特权匹配非因果对照 + graph-null 校准 bootstrap + 统一判据）写成可复用模板，供任何"物理/因果衍生特征 vs. 已训练原始网格模型"类主张审计使用。
- Explicit non-contributions：不提出新因果发现算法；不恢复真实大气 DAG；不提出新融合架构；不构建新时空骨干；不对 D/J 做超出统计定义之外的物理机制解释。

## Proposed Method

### Complexity Budget

- 冻结/复用：合成 Gate 0 沿用固定边局部回归 proxy；真实数据阶段用 PCMCI+（tigramite, ParCorr）与 LKIF（`LK_Info_Flow`，两环境均已装好，直接复用）；`RawTensorCNN` 架构族（`Conv3d→ReLU→MaxPool3d→Conv3d→ReLU→AdaptiveAvgPool3d→Linear`，唯一可变超参是池化粒度）；Gaussian sham（v10 已验证）保留为第一个对照臂；AUPRC 主指标与两阶段 seed/storm-cluster bootstrap（`factorial_helpers.py`/`cnn_fusion_helpers.py` 机制直接复用）。
- 新增：盲选架构预注册规则；特权匹配非因果对照表示；把 v6 的 exchangeable-null 校准审计（`null_audit_helpers.py`）扩展到本轮 CNN+bootstrap 管线（此前只覆盖旧 factorial 管线）；真实 HRRR/SPC storm-level 数据管线（storm-relative/平流匹配坐标 + `>=2` 季节 `>=50` storm 标签）；统一 POSITIVE/NULL/NEGATIVE 判据（去掉"CI 上界 `<0`"与"`<0.01`"两套阈值并存的歧义）；seed 不收敛诊断规则（v10 的 seed `20260901` 卡在 `log(2)` 却被直接平均进汇总，改为预注册排除/双报告）。
- 刻意不做：新因果发现方法；新融合/加权机制（沿用 v10 通道拼接）；新骨干网络（transformer/DiT 仅作附录 robustness check，只在 Gate 0 与主结果均 POSITIVE 时才跑）；P3 归因本轮不做，留待通过后下一轮。

### System Overview

```mermaid
graph TB
    A["HRRR 96h history, 4 pressure levels x lon x lat<br/>plus SPC event labels"] --> B["Storm-relative resample<br/>advection-matched coordinates"]
    B --> C["Raw tensor branch<br/>standardized on train split only"]
    B --> D["Causal estimator, frozen<br/>real PCMCI-plus and LKIF"]
    D --> E["D and J channels<br/>dispersion and jump at discovered lags"]
    B --> F["Privilege-matched non-causal control<br/>same candidate lags and neighbors, no conditioning"]
    B --> G["Gaussian sham<br/>dimension matched only"]
    C --> H["RawTensorCNN<br/>blind pre-registered pooling shape"]
    E --> H
    F --> H
    G --> H
    H --> I["Held-out storm AUPRC and calibration"]
    I --> J["Graph-null calibrated storm-cluster bootstrap"]
    J --> K["Decision: POSITIVE, NULL, or NEGATIVE"]
```

### Core Mechanism

- Input / output：四通道标准化原始场（1000/850/500/250 hPa，96h，lon x lat）+ 一个辅助通道臂（zero / Gaussian-sham / 特权匹配非因果 / `(D,J)`），输出 2-6h 内目标邻域发生强对流灾害的概率。
- Architecture：`RawTensorCNN` 池化形状从 `{(1,1,1), (2,2,2), (4,2,2), (4,4,4)}` 中，用从未出现在 Gate 0 决定性运行或真实数据中的独立合成校准种子池，按"raw-only 验证 AUPRC 最高"规则盲选并冻结，再生成/比较其余三条件。
- 特权匹配非因果对照：合成场景下把与 `(D,J)` 相同的真实边位置/滞后（3h/24h）告诉一个非因果统计量（边际滞后相关或局部方差），即字面 oracle 对照；真实数据场景没有"真实边"，改为让该统计量使用与 PCMCI+/LKIF 相同的候选滞后/邻接搜索网格与同款筛选步骤但跳过条件化——分离"被告知该找哪些候选"（causal 与该对照共享）与"条件化本身"（只有 causal 有）。
- Training signal / loss：`BCEWithLogitsLoss`，Adam（`lr=0.003, weight_decay=0.01`），120 epoch（合成阶段沿用 v10 设置，真实阶段按数据量调整）。
- 主要新颖点：不是审计对象新，而是审计标准补齐了两处此前未被覆盖的偏差源——同一个"NULL"在 v10 里可能偏乐观，在本协议下才可信。

### Optional Supporting Component

- 协议模板可迁移性：本审计协议（盲选架构 + 特权匹配非因果对照 + graph-null 校准 bootstrap + 统一判据）只依赖"衍生表示是原始历史的确定性函数"这一前提，可直接套用到任何"物理诊断量/因果衍生特征 vs. 已训练原始网格模型"审计（如 CAPE/shear 衍生指数）。
- 为什么不发散：本轮只在本表示上实例化并报告，可迁移性只是 Discussion 段一句话论证，不构成第二个实验轴。

### Modern Primitive Usage

- 不以 LLM/VLM/Diffusion/RL 为核心机制；PCMCI+/LKIF 是经典因果发现方法，非前沿生成式组件。瓶颈是比较协议公平性而非表征能力，故不硬凑前沿组件（按 experiment-planning 规则明说并跳过 frontier block）。
- 唯一涉及"现代"骨干之处：把 transformer/DiT 式时空模型列为附录级 robustness check（非必需）——仅当 Gate 0 与主结果都 POSITIVE 时，才追加跑一次同容量预算下的现代骨干，检验结论是否只是 CNN 特定归纳偏置。

### Integration into Base Generator / Downstream Pipeline

- Gate 0（合成，协议冻结）→ 通过则 P1 真实 mini（管线验证，3 个既有日期 + 少量新日期）→ P1.5（storm-relative/平流对照 + graph-null 校准，P2 前置硬门）→ P2（`>=2` 季节 `>=50` storm confirmatory）。任何一步不通过即停止向下游花预算，转入诚实边界报告（见 Failure Modes）。
- 架构一旦在 Gate 0 冻结，P1/P1.5/P2 全程复用同一份权重初始化协议与超参，不再重新搜索。

### Training Plan

- 数据来源：真实阶段复用 `data/fetch_hrrr_subset.py`（NOAA 字节范围提取，已验证）与 `data/fetch_spc_reports.py`（SPC 逐日报告，已验证），泛化到多日期、多风暴窗口。
- 监督：事件标签来自 SPC 报告（tornado/wind/hail），负样本取同季节、时空匹配但未触发报告的窗口。
- 校准：nested storm holdout，任何模型选择（池化粒度、早停）只用内层折，外层折只在最终评估用一次。
- Curriculum：合成 Gate 0（先盲选架构，再比较四条件）→ 真实管线冒烟（沿用既有 3 日期资产）→ 真实 confirmatory（`>=2` 季节 `>=50` storm，同时报告 IID storm holdout 与季节/regime 迁移 holdout）。后者按 anchor-regression/CausalKinetiX 文献（1801.06229, 1810.11776, 1707.06422）的理论预期，是因果衍生表示最可能显现真实优势之处——若 `(D,J)` 有真实价值，跨 regime 泛化上的 `Delta` 应大于 IID holdout，这是文献综述给出的可操作、可证伪的具体预测，而非泛泛的"跨季节复现"要求。
- Losses/weighting：单一 BCE，不引入多任务花活；若类别极不平衡，允许类别加权但需同时应用于全部四条件，保持公平。

### Failure Modes and Diagnostics

- 架构预注册被污染（校准种子池与决定性池隐性相关）：用独立种子基生成校准池，Gate 0 报告中明确两池零重叠。
- 特权匹配对照过难/过易：检查该对照与 `(D,J)` 的单变量 AUROC 是否可比（类似既有 P0 诊断），差距过大先调整构造再进入真实数据阶段。
- 平流混淆：storm-relative/advection-matched 坐标变换是 P1.5 硬门，不通过不得进入 P2。
- seed 不收敛（如 v10 的 `20260901`）：raw 分支训练 loss 高于阈值（如 `0.65`，接近 `log(2)`）标记为不收敛，双报告含/不含该 seed 的汇总，不直接平均进主表。
- 真实数据获取超预算：允许缩小 N 并标注功效受限，不得静默降阈值凑 POSITIVE。
- 多重比较膨胀：对照臂 x holdout 切分，统一 Benjamini-Hochberg 校正后再报判据。
- 十轮九轮 NULL/NEGATIVE 的先验：诚实边界是同样可发表的结局，不为凑 POSITIVE 无限加码协议或样本；Gate 0 若仍是宽区间 NULL/NEGATIVE，直接停止 P2 预算，转写失败边界论文。

### Novelty and Elegance Argument

最近邻 Ganesh, Beucler, DeMaria, Runge（2025, SHIPS+, arXiv:2510.02050）：因果发现选出的预测因子增补进标准 21 个 SHIPS predictors，在真实台风业务数据上验证"因果衍生特征增强既有强 baseline"这一路线在邻域可行。Flora, Varga, Potvin, Lang（2026, arXiv:2603.20250）证明 U-Net/HGBT 可在 WoFS 网格化输出（经全文核验为 63-channel 时间聚合 2D 张量，非完整 96h 原始历史）上做 2-6h 强对流预警的真实业务先例。两者共同确立"因果特征可能有用"与"网格化 ML 可用于该窗口"两个前提，但都没做本提案要做的事：同历史、同容量、消除 oracle 特权之后，把 `(D,J)` 与完整时间序列原始张量端到端 CNN 做条件增量审计。Markov blanket 预测最优性文献（2002.09414）给出理论落点：足够表达、训练在完整历史上的模型，其预测能力上界覆盖该历史的任何确定性函数（含 `(D,J)`），真实增益只能来自有限样本归纳偏置，不能来自"额外信息"——这也是为什么"apply 因果特征 to 强对流预警"本身不构成贡献：贡献在于用去偏协议诚实回答归纳偏置是否存在、多大、在哪类 regime 下存在。协议只多了两处必要修补，不是模块堆叠。

## Claim-Driven Validation Sketch

### Claim 1（主锚点，直接对应 Bottom-line problem）

- Minimal experiment：真实 HRRR/SPC storm-level 数据上的公平增量审计。`>=2` 季节、`>=50` 个独立 storm systems，outer storm holdout；每个 storm 提供 96h 历史窗口 + 2-6h 提前量的灾害标签；同时报告 IID storm holdout 与季节/regime 迁移 holdout 两种切分。
- Baselines/ablations：raw-only（Gate 0 冻结架构）、raw+Gaussian-sham、raw+特权匹配非因果对照、raw+`(D,J)`（真实 PCMCI+/LKIF），四臂同历史同容量。
- Metric：paired `Delta AUPRC`（causal − raw）storm-cluster bootstrap 95% CI 为决定性指标；Brier/ECE 为次要指标。
- Expected evidence：POSITIVE 需 CI 下界 `>0.01` 且同时优于两个对照臂、两种 holdout 同号；否则按统一判据报 NULL（跨越 `[0, 0.01]`）或 NEGATIVE（CI 上界 `<0.01`，不与旧的"CI 上界 `<0`"混用）。若存在真实信号，regime 迁移 holdout 上的 `Delta` 理论上应大于 IID holdout。

### Claim 2（Gate 0 必要性 / stop-go 前置门）

- Minimal experiment：合成 oracle-favorable DGP 上，对比"v10 旧协议"（事后选架构 + 仅 Gaussian sham）与"新协议"（盲选架构 + 特权匹配对照 + 校准 bootstrap + 统一判据）在同一批种子上的判定是否一致。
- Baselines/ablations：v10 冻结代码作为未充分控制的协议本身；新协议在相同 DGP 上重跑。
- Metric：两套协议判据是否翻转；特权匹配对照相对 causal 的 CI 是否与 Gaussian sham 相对 causal 的 CI 有实质差异。
- Expected evidence：判据相同则说明新控制此处良性但非必要（仍需真实数据阶段重验）；判据不同（旧协议因未控制特权而偏乐观）则证明新控制确有必要，回应"加码复杂度"的质疑。Gate 0 同时充当 stop/go：新协议下非判定性 NEGATIVE 才授权 P2 预算。

## Paper Outline

- S1 Introduction：静态 diagnostics 局限 + 因果衍生特征可信度问题（十轮进展一句话总结）+ 贡献（去偏审计协议 + storm-level 诚实答案）。
- S2 Related Work：(a) 因果特征增强强 baseline 先例（SHIPS+）；(b) 网格化 ML 强对流预警先例（Flora et al., Sha et al. 2310.06045）；(c) Markov blanket / anchor regression / IRM 解释"因果条件化为何丢弃预测信息"及复现失败文献（DomainBed, 2402.09891）。
- S3 Method：架构盲选预注册、特权匹配非因果对照的双重定义（合成 oracle / 真实候选匹配）、graph-null 校准 storm-cluster bootstrap、统一判据。
- S4 Experiments：Gate 0 结果（Claim 2）；真实 storm-level 主结果（Claim 1），含 IID vs regime-迁移两种 holdout。
- S5 Analysis：校准诊断；storm-relative/平流对照是否改变结论；seed 不收敛诊断的影响。
- S6 Conclusion：诚实标注判据边界；P3 归因与现代骨干 robustness check 留作未来工作。
- Key figures：Fig 1（hero）四臂 `Delta AUPRC` 森林图（Gate 0 vs 真实数据 x IID vs regime-迁移），直接对应 Claim 1+2；Fig 2 系统图（mermaid 定稿版）；Fig 3 校准可靠性图；Fig 4（附录）seed 收敛诊断与协议必要性对比表。

## Compute and Timeline Estimate

- Estimated GPU-hours：Gate 0（架构扫描 + 真实 PCMCI+/LKIF + 更大校准 seed 池）约 5-10 L4 GPU-hour（较 v10 的 0.016 上调，仍是廉价筛查）；P1 真实 mini 约 2-5 GPU-hour；P1.5（CPU 坐标变换为主）GPU 开销很小；P2 confirmatory 约 20-50 GPU-hour + 1000-3000 CPU-hour（沿用 idea 阶段既有估计）。
- Data / annotation cost：SPC 报告体量小（KB 级/日），`fetch_spc_reports.py` 直接泛化到更多日期。HRRR 完整历史复用 `fetch_hrrr_subset.py` 已验证的字节范围提取，扩展到约 7-8 个候选变量 x 4 气压层 x 每风暴 96-150h 窗口 x `>=50` storm，量级预计数十到约百 GB；批量拉取前先做精确字节核算，不整档下载完整 GRIB。
- Timeline：Gate 0 约 1 周；P1 真实 mini + P1.5 约 2-3 周；P2 confirmatory 约 4-6 周（均与 idea 阶段既有估计一致）。Gate 0 若判 NEGATIVE 或功效不足，直接跳过 P2、转写失败边界论文，节省全部真实数据预算——这正是硬门的意义。
