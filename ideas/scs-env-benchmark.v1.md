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
