---
# 目标 venue: 数据描述类期刊为主, 兼顾 ML 基准 track。
target-venue: [ESSD (Earth System Science Data), "NeurIPS Datasets and Benchmarks"]

# 这是一个数据集 / 基准 idea, 不是方法新颖性 idea。评估按数据贡献维度 (填补空白 / 整编严谨性 / 可复现 / 可用性) 判断。
preferred-contribution-types: [benchmark, application, empirical-finding]
---
<!-- 书写报告使用中文 -->

# 强对流环境–雷达多模态开放基准数据集 (SCS Environment Benchmark)

本 topic 要建立一个用于**强对流 (severe convective storm, SCS) 环境分析、事件预警与临近预报 (nowcasting)** 的开放基准数据库。核心判断: 现有开放数据集虽有 SEVIR、Grad-Severe、HKO-7、MeteoNet、MYRORSS 等较好资源, 但它们按用途割裂 —— **没有一个真正把"环境数据 (environmental fields) 及其时空演变"作为一等公民**。而环境场及其时空变化对强对流系统的分析与可预报性有决定性作用 (理想化探空实验表明, 仅低层湿度或自由对流层湿度的微小改变即可把风暴从"1 小时耗散"翻转为"长寿命超级单体")。这就是本数据集要填补的空白: 把**长时窗环境场 + 雷达观测 + 事件标签**统一封装成以风暴为中心的事件包。

**数据模型 (用户设定的目标形态)**: 年份 → 事件聚类区域 → {雷达 (MRMS, 时间窗从区域降水信号起止各往前后外推到基本无信号) / 事件标签 / 环境场 (区域时间起止分别往前 15 天、往后 5 天) / 静态特征}。原始数据来源: **NOAA Storm Events**(事件记录)、**HRRR**(环境场, f00 与 f01; 个别无法算 f00 的场如 updraft 用 f01)、**MRMS**(雷达, 供 nowcasting 模型)。交付物三层: (1) 汇集态源数据; (2) 模型训练 ready 数据; (3) 从收集到 (1)、从 (1) 到 (2) 的工作流代码。这套"年份 → 事件聚类 → 多模态"结构本身也是一个可迁移到其它雷达/环境数据的工作框架。

**发表角度**: 最强的角度不是"发布一堆拼好的文件", 而是发布一个整编严谨、可版本化、开放记录的**研究数据产品** —— 谐一化的事件包、机读元数据、可追溯到文件与处理步骤的 provenance、显式的不确定性与质量标记、带 DOI/版本的开放仓库, 以及能证明数据集支持气候学/过程研究/nowcasting/预警评估的示范分析。这契合 ESSD 这类数据描述期刊, 也可走 NeurIPS Datasets & Benchmarks track (需配 canonical tasks/splits/baselines/metrics)。

**主要风险不在科学新颖性, 而在工程与整编**: license 混用 (NOAA/NASA/USGS/Copernicus/MADIS 各源再分发条款不一)、标签噪声 (Storm Events 的采集口径随时间/事件类型变化)、时间对齐泄漏 (环境场逐小时、探空 00/12 UTC、雷达数分钟、卫星中尺度 30s, 谐一化步的插值窗口若不显式记录会造成"后视泄漏")、数据体量 (原始雷达矩 + 长时窗多模态存储巨大)、事件定义歧义。把这些干净解决, 数据集就能与现有开放资源明显区分并适配 ESSD 式数据描述论文。

## 关联材料

- 源笔记 (用户提供, 本 topic 由其直接转写, 跳过 idea-creator): `/blue/yixin.wen/weikangqian/GitRepo/myproject/SCSDataset_benckmark.md` (idea 一页纸) 与 `SCSDataset_benckmark_report.md` (一份 deep-research 生成的 landscape 蓝图, 已转入本 topic 的 landscape 文件; 其内联 `citeturn…` 标记来自 web-search 工具, 论文引用未经 arxiv-tools 核验, 待 deep-lit 阶段验证)。
- 同课题组姊妹 topic `0710-causal-scs` (因果效应不稳定性作为强对流预警指标): 与本数据集互补 —— 本基准若建成, 正好为那条 idea 提供长时窗 3km 环境场 + 事件记录的真实数据底座。两者范围不同, 暂各自独立。

## References

<!-- 论文引用统一维护在 topics/0713-scs-benchmark-landscape.md, 且必须经 arxiv-tools 核验后落笔。 -->
